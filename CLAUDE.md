# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

LogiTrack Inventory — an internal warehouse stock dashboard. The entire application is one file,
[index.html](index.html) (~865 lines): markup, CSS and JavaScript inline. There is no package
manager, no bundler, no test runner, and no git repository.

## Commands

```powershell
Invoke-Item .\index.html      # run it — opens in the default browser
```

That is the only command. Node is **not installed on this machine**, so there is no way to lint or
parse-check the JavaScript locally; changes to the `<script>` block have to be reviewed by reading
and then confirmed in a browser.

Regression scan for the hard constraints below (must return no matches):

```powershell
$c = [System.IO.File]::ReadAllText("$PWD\index.html", [System.Text.Encoding]::UTF8)
[regex]::IsMatch($c, 'localStorage|sessionStorage|indexedDB|onclick=|<script src|<link[^>]*href=')
```

## Hard constraints (from the original spec — do not relax without being asked)

- One file. No frameworks, no build step, no external libraries, no CDN links, no web fonts.
- **No storage APIs.** State is in-memory only; reloading deliberately restores the seed data.
- No inline `onclick` attributes — everything wires up through `addEventListener` in `init()`.
- No `alert()` / `confirm()`. Confirmations go to the `aria-live` region (`announce()`).

## Architecture

The load-bearing idea is a **one-way data flow with no DOM reads**. `inventory`
([index.html:582](index.html#L582)) is the single source of truth; the DOM is a pure projection of
it. Nothing anywhere reads a value back out of a table cell or recomputes state from markup.

```
inventory[]  ──filter──▶ getFilteredRecords()  ──sort──▶ compareRecords()  ──▶ renderTable()
     │                     (searchTerm,                   (sort.key/dir)
     │                      categoryFilter)
     └────────────────────────────────────────────────────────────────────▶ renderSummary()
```

Every mutation calls `refresh()` ([index.html:791](index.html#L791)), which rebuilds the whole tbody
and the whole summary strip. There is no incremental DOM patching — if you add a feature, add it to
the render path rather than poking at elements.

The script is banner-commented into four sections: **1. STATE** (line 573), **2. RENDER FUNCTIONS**
(613), **3. EVENT HANDLERS** (798), **4. INITIALISATION** (958). Keep new code in the matching one.

### Invariants worth knowing before editing

- **Status is derived, never stored.** `statusOf()` ([index.html:641](index.html#L641)) computes
  In / Low / Out from `qty` vs `reorder` on every render. Records have no `status` field, and adding
  one would create a second source of truth.
- **Two kinds of state, kept apart.** `inventory` is data; `searchTerm` / `categoryFilter` / `sort`
  are view state. Filtering and sorting never mutate `inventory` — `getFilteredRecords()` returns a
  new array, and only that copy is sorted.
- **`CATEGORIES` / `WAREHOUSES` are the single option source.** `init()` populates the table's
  category filter *and* both form selects from them. Never hand-write `<option>` elements — the
  lists would drift apart.
- **Sort keys are column-typed, not uniform.** `compareRecords()` branches: numeric subtraction for
  `qty`/`reorder`/`cost`, ISO string compare for `updated`, `STATUS_RANK` (Out → Low → In, i.e.
  urgency, not alphabetical) for `status`, `localeCompare` for the rest. A new column needs its
  branch added there.
- **Delete is delegated**, one listener on `<tbody>` ([index.html:837](index.html#L837)), matching
  `.btn--delete` and resolving the record by `data-sku`. SKU is the de facto primary key: uniqueness
  is enforced case-insensitively on submit, and lookups depend on it.
- **The new-row flash is one-shot.** `justAddedSku` is set on insert and cleared inside
  `renderTable()`, so the highlight survives exactly one render.
- **Cells are built with `textContent`**, never `innerHTML`, so supplier/product strings can't inject
  markup. Keep it that way when adding columns.
- **Form validation is hand-rolled** — the `<form>` carries `novalidate`. `handleSubmit()`
  ([index.html:878](index.html#L878)) collects *all* field errors before returning, then focuses the
  first offending input. Errors render into `<p class="field-error">` elements bound by
  `aria-describedby`, paired with `aria-invalid` on the control; `clearErrors()` must stay in sync
  with both.

### CSS

Colours live as custom properties on `:root`. Semantic pill colours (`--ok-*`, `--warn-*`, `--bad-*`)
are deliberately separate from the blue accent — don't collapse them. The table card owns the scroll
(`.table-wrap` has `overflow-x/y: auto` with a sticky `thead`), so the page body never scrolls
sideways; the layout collapses to one column at 900px.

## Published artifact

A mirror of this page is published at
https://claude.ai/code/artifact/12b58142-7a1c-4163-8846-31169bc427d5

It is the **same markup and script** with one deliberate difference: it adds a dark-theme token set
(`@media (prefers-color-scheme: dark)` guarded by `:root:not([data-theme="light"])`, plus
`:root[data-theme="dark"]`) because artifacts render in the viewer's theme, and it omits the
`<!doctype>/<html>/<head>/<body>` skeleton the artifact runtime injects. `index.html` stays
light-only. If you change behaviour in `index.html`, port it to the artifact copy and republish by
passing that URL as `url` — publishing without it creates a second artifact instead of updating this
one.

## Windows gotcha

Do not edit this file with PowerShell string operations. `Get-Content -Raw` in Windows PowerShell 5.1
reads a BOM-less UTF-8 file as ANSI, which silently mangles the non-ASCII characters in the source —
the `▲`/`▼` sort arrows and any typographic dashes. Use the Edit/Write tools instead. If mojibake
appears (`Ã`, `â€"`), the file was round-tripped through PowerShell and needs rewriting.
