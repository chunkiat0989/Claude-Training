# LogiTrack Inventory

A warehouse stock dashboard for tracking SKUs across a regional distribution network — search,
filter, sort, and maintain inventory records, with stock status derived automatically from
quantity against reorder level. The entire application is a single `index.html`: markup, CSS and
JavaScript inline, no frameworks and no build step. Open the file and it runs.

## Live demo

**https://chunkiat0989.github.io/Claude-Training/**

Deployed from `main` by GitHub Actions on every push.

## Running it locally

Clone the repo and open `index.html` in any modern browser. That is the whole setup — there is
nothing to install, compile, or serve.

```
git clone https://github.com/chunkiat0989/Claude-Training.git
cd Claude-Training
```

Then double-click `index.html`, or on Windows:

```powershell
Invoke-Item .\index.html
```

## Features

- **Summary strip** — total SKUs, units in stock, count below reorder level, and warehouses in use,
  all recomputed on every change.
- **Search** across SKU, product name and supplier.
- **Category filter**, populated from a single source list so the options can never drift.
- **Typed column sorting** on all ten columns. Sorting is per-column-type rather than uniform:
  numeric for quantity, reorder level and unit cost; date order for last-updated; urgency order
  (Out → Low → In) for status; locale-aware text compare for the rest.
- **Derived stock status** — In Stock / Low / Out is computed from quantity versus reorder level on
  every render. It is never stored on a record, so it cannot fall out of sync.
- **Add and delete records**, with newly added rows briefly highlighted.
- **Accessible form validation** — hand-rolled, collecting every field error at once rather than
  stopping at the first, then focusing the earliest offending input. Errors are bound to their
  inputs via `aria-describedby` and `aria-invalid`; confirmations go to an `aria-live` region
  instead of `alert()`. Column headers expose `aria-sort`.

## Design constraints

These were deliberate and are worth preserving if you fork it:

- **One file.** No frameworks, no bundler, no external libraries, no CDN links, no web fonts. The
  page makes no network requests of any kind.
- **No storage APIs.** State lives in memory only — reloading the page deliberately restores the
  seed data. There is no `localStorage`, no backend, and nothing persists.
- **One-way data flow.** A single `inventory` array is the source of truth and the DOM is a pure
  projection of it; no code reads state back out of the markup. Every mutation re-renders.
- **No inline event attributes.** Everything is wired through `addEventListener`.
- Cells are built with `textContent`, never `innerHTML`.

## Data note

The inventory rows are **sample data — invented for this demo**. The SKUs, quantities, costs,
supplier names and warehouse locations do not correspond to real stock, real vendors, or any real
system. Any resemblance to an actual company name is coincidental.

## Repository contents

| Path | What it is |
| --- | --- |
| `index.html` | The entire application. |
| `CLAUDE.md` | Architecture notes and invariants, for working on the code with Claude Code. |
| `.claude/commands/` | A reusable slash command for publishing this project to GitHub. |

## Status

No tests, no CI, and no license file — this is a demo project. If you want a license added, say so
and one can be included.
