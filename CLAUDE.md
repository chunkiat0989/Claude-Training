# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Kacific Inventory Dashboard — an internal warehouse stock dashboard carrying Kacific Broadband
Satellites branding. The entire application is one file, [index.html](index.html) (~1640 lines):
markup, CSS and JavaScript inline, plus the Kacific logo embedded as a base64 `data:` URI.

## Commands

```powershell
Invoke-Item .\index.html      # run it — opens in the default browser
```

Node **is** installed on this machine, so the inline script can be parse-checked without a browser:

```bash
sed -n '/^<script>$/,/^<\/script>$/p' index.html | sed '1d;$d' > /tmp/check.js
node --check /tmp/check.js
```

Playwright's browser tools refuse `file:` URLs, so to drive the page in a real browser serve it
first (`python -m http.server 8731 --bind 127.0.0.1`) and navigate to `http://127.0.0.1:8731/`.

Regression scan for the hard constraints below (must return no matches):

```powershell
$c = [System.IO.File]::ReadAllText("$PWD\index.html", [System.Text.Encoding]::UTF8)
[regex]::IsMatch($c, 'localStorage|sessionStorage|indexedDB|onclick=|<script src|<link[^>]*href=')
```

## Hard constraints (from the original spec — do not relax without being asked)

- One file. No frameworks, no build step, no external libraries, no CDN links, no web fonts.
  The page makes **zero network requests** — the logo is inlined as a `data:` URI, and Montserrat is
  named in the font stack but never fetched, so it renders only where the viewer already has it.
- **No storage APIs.** State is in-memory only; reloading deliberately restores the seed data.
- No inline `onclick` attributes — everything wires up through `addEventListener` in `init()`.
- No `alert()` / `confirm()`. Confirmations go to the `aria-live` region (`announce()`).

## Architecture

The load-bearing idea is a **one-way data flow with no DOM reads**. `inventory`
([index.html:1036](index.html#L1036)) is the single source of truth; the DOM is a pure projection of
it. Nothing anywhere reads a value back out of a table cell or recomputes state from markup.

```
                      ┌─▶ getFilteredRecords() ─▶ compareRecords() ─▶ renderTable()
                      │    (searchTerm,            (sort.key/dir)
inventory[] ──────────┤     categoryFilter)
                      │
                      │   computeKpis()  ─┬─▶ renderKpis()       (4 stat tiles)
                      └─▶ (whole array,   └─▶ renderStatusMix()  (stacked bar + legend)
                           never filtered)
                          renderWarehouseValue()                 (ranked bars)
```

Every mutation calls `refresh()` ([index.html:1434](index.html#L1434)), which computes the KPIs once
and rebuilds the tbody, the four tiles, the status bar and the warehouse bars. There is no
incremental DOM patching — if you add a feature, add it to the render path rather than poking at
elements.

The script is banner-commented into four sections: **1. STATE** (1027), **2. RENDER FUNCTIONS**
(1067), **3. EVENT HANDLERS** (1444), **4. INITIALISATION** (1604). Keep new code in the matching one.

### Invariants worth knowing before editing

- **Everything derived is derived on every render, never stored.** `statusOf()`
  ([index.html:1115](index.html#L1115)) computes In / Low / Out from `qty` vs `reorder`;
  `valueOf()` ([index.html:1122](index.html#L1122)) computes stock value from `qty × cost`;
  `shortfallOf()` ([index.html:1127](index.html#L1127)) computes units-to-reorder-level. Records
  carry none of these as fields, and adding one would create a second source of truth.
- **KPIs read the whole array; the table reads the filtered copy.** `computeKpis()`
  ([index.html:1263](index.html#L1263)) walks `inventory` directly so the management figures stay
  steady while a coordinator searches. Only `renderTable()` goes through `getFilteredRecords()` —
  which is why `handleSearch` / `handleCategoryFilter` call `renderTable()` alone, not `refresh()`.
- **Two kinds of state, kept apart.** `inventory` is data; `searchTerm` / `categoryFilter` / `sort`
  are view state. Filtering and sorting never mutate `inventory` — `getFilteredRecords()` returns a
  new array, and only that copy is sorted.
- **`CATEGORIES` / `WAREHOUSES` are the single option source.** `init()` populates the table's
  category filter *and* both form selects from them. Never hand-write `<option>` elements — the
  lists would drift apart. The warehouse panel, by contrast, derives its rows from the records
  actually present, so an unused warehouse simply does not appear.
- **Sort keys are column-typed, not uniform.** `compareRecords()`
  ([index.html:1150](index.html#L1150)) branches: numeric subtraction for `qty`/`reorder`/`cost`,
  `valueOf()` for the derived `value` column, ISO string compare for `updated`, `STATUS_RANK`
  (Out → Low → In, i.e. urgency, not alphabetical) for `status`, `localeCompare` for the rest. A new
  column needs its branch added there — and the `colSpan` on the empty-state cell (currently **12**)
  bumped to match.
- **`els.thead` is `document.querySelector("thead")`** — the *first* thead in the document. The
  inventory table must therefore stay ahead of the "How each KPI is calculated" table in DOM order.
- **Delete is delegated**, one listener on `<tbody>` ([index.html:1633](index.html#L1633)), matching
  `.btn--delete` and resolving the record by `data-sku`. SKU is the de facto primary key: uniqueness
  is enforced case-insensitively on submit, and lookups depend on it.
- **The new-row flash is one-shot.** `justAddedSku` is set on insert and cleared inside
  `renderTable()`, so the highlight survives exactly one render.
- **Cells are built with `textContent`**, never `innerHTML`, so supplier/product strings cannot
  inject markup. The panel renderers follow the same rule — they build elements and set
  `style.width`, never markup strings. Keep it that way when adding columns.
- **Form validation is hand-rolled** — the `<form>` carries `novalidate`. `handleSubmit()`
  ([index.html:1524](index.html#L1524)) collects *all* field errors before returning, then focuses
  the first offending input. Errors render into `<p class="field-error">` elements bound by
  `aria-describedby`, paired with `aria-invalid` on the control; `clearErrors()` must stay in sync
  with both.

### The KPIs

Four tiles. They are also documented in the page's own "How each KPI is calculated" table — keep
that table and `computeKpis()` in agreement, or the page contradicts itself.

| Tile | Formula |
| --- | --- |
| Inventory value (hero) | `Σ qty × cost` |
| Stock availability | lines with `qty > reorder`, ÷ total lines |
| Lines needing action | count of `qty ≤ reorder`, split into out (`qty === 0`) and low |
| Replenishment cost | `Σ max(reorder − qty, 0) × cost` over flagged lines |

Display numerals use `Intl.NumberFormat` **compact** notation (`$783.6K`); the exact figure always
appears in the caption beneath, so nothing is ever shown only rounded.

## Branding & visual design

The palette, type and motifs come from the `kacific-branding` skill
(`.claude/skills/kacific-branding/`), which extracted them from the live kacific.com theme.

- `#034EA2` brand blue carries headings, the app bar, primary buttons and the single-hue bars.
- Montserrat is the brand face, named in `--font` with a full fallback stack. **It is deliberately
  not loaded** — a web font would break the no-network constraint.
- The **orbit ellipse** motif appears once, as two faint `--k-cyan` hairline ellipses behind the app
  bar (`.appbar::before` / `::after`). Do not repeat it elsewhere; the skill is explicit that
  ellipses everywhere become noise.
- Buttons use the house pill radius (38px), uppercase, weight 500.
- The footer is the brand's two stacked blue bands (`#07529E` over `#0071BC`).
- The logo is `logo-kacific-white.png` from the skill's `assets/`, base64-inlined into the `<img>`
  in the app bar. To swap it, re-encode and replace the `data:image/png;base64,…` payload — do not
  add a file reference.

### Chart / status colour rules (from the `dataviz` skill)

- Semantic status fills (`--ok-fill` `#1E8E4F`, `--warn-fill` `#E0A008`, `--bad-fill` `#C1362F`) are
  **reserved** — they were validated for CVD separation against white and must never be reused as a
  series colour. `--warn-fill` sits below 3:1 on white, which is why every status segment carries a
  visible label or a legend entry; meaning is never colour-alone.
- The warehouse panel is a **single-hue** ranked bar list with direct value labels, so it needs no
  legend and no categorical palette.
- The availability meter's fill carries only the severity band (≥90 ok, ≥70 warn, else bad); the
  numeral beside it always states the real value.

### CSS

Colours live as `--k-*` custom properties on `:root`. Semantic pill/status colours are deliberately
separate from the brand blue — do not collapse them. The table card owns the scroll (`.table-wrap`
has `overflow-x/y: auto` with a sticky `thead`), so the page body never scrolls sideways; the layout
collapses to one column at 900px and the KPI grid steps 4 → 2 → 1 at 1120px and 560px.

## Published artifact

A mirror of this page is published at
https://claude.ai/code/artifact/12b58142-7a1c-4163-8846-31169bc427d5

It carries the **same markup and the byte-identical script**, with three deliberate differences,
because an artifact is rendered by the runtime rather than opened as a file:

1. It omits the `<!doctype>/<html>/<head>/<body>` skeleton the artifact runtime injects.
2. Every component colour is a semantic token (`--surface`, `--band`, `--thead-bg`, `--bar`, …),
   with the complete light palette on bare `:root` and a dark set redefined twice — under
   `@media (prefers-color-scheme: dark)` guarded by `:root:not([data-theme="light"])`, and again
   under `:root[data-theme="dark"]` — because artifacts render in the viewer's theme. The dark
   palette follows the branding skill's guidance: a `#003C72`-to-near-black ground with `#358CCB`
   promoted to the accent role. `index.html` stays light-only and keeps its literal hexes.
3. It loads Montserrat from Google Fonts (the one font host the artifact CSP admits), so the brand
   face actually renders. `index.html` cannot — a web font would break its no-network constraint.

If you change behaviour in `index.html`, port it to the artifact copy and republish by
passing that URL as `url` — publishing without it creates a second artifact instead of updating this
one.

## Windows gotchas

Do not edit `index.html` with PowerShell string operations. `Get-Content -Raw` in Windows PowerShell
5.1 reads a BOM-less UTF-8 file as ANSI, which silently mangles the non-ASCII characters in the
source — the typographic dashes and arrows. Use the Edit/Write tools or a Bash heredoc instead. If
mojibake appears (`Ã`, `â€"`), the file was round-tripped through PowerShell and needs rewriting.

This shell also rejects Bash commands over roughly 8 KB, and the failure surfaces as an unhelpful
`unexpected EOF while looking for matching quote`. Write large files as several appended heredocs
rather than one.
