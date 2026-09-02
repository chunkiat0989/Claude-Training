---
name: kacific-branding
description: |
  Applies Kacific Broadband Satellites' visual identity — the #034EA2 brand blue, Montserrat
  typography, the orbit-ellipse motif, pill buttons, and the official logo files.
  Use when: building or restyling anything Kacific-facing — dashboards, internal tools, decks,
  reports, landing pages, email templates, artifacts, diagrams — or when asked to make something
  "on brand" / "look like Kacific" / "use our colours".
  Provides: copy-paste CSS tokens, logo assets and usage rules, type scale, component recipes,
  and the contrast limits that constrain which colours may carry text.
---

# Kacific Branding

Visual identity for **Kacific Broadband Satellites Ltd** — a Singapore-headquartered satellite
broadband operator serving the Asia-Pacific and Pacific Islands. Tagline: *"the heart of broadband"*.

Everything here was extracted from the live kacific.com theme on 2026-09-02 (see
[Provenance](#provenance)) — these are the colours and fonts the site actually renders, not an
approximation.

## Quick start

Copy [reference/tokens.css](reference/tokens.css) into the project and use the custom properties.
The three decisions that make something read as Kacific:

1. **`#034EA2` blue** carries the brand — headings, nav, primary surfaces.
2. **Montserrat**, with body text at **weight 300** (Light). The light body weight against heavy
   blue headings is the signature typographic contrast.
3. **The orbit ellipse** — imagery cropped into a large tilted ellipse, echoing the logo's swoosh.

## Colour

### Core

| Token | Hex | Role |
|---|---|---|
| `--k-blue` | `#034EA2` | **Primary brand blue.** Headings, nav bar, primary buttons, logo wordmark. |
| `--k-blue-deep` | `#003C72` | Deep navy — hover states, dense text on light, footer depth. |
| `--k-blue-mid` | `#0071BC` | Mid blue — secondary surfaces, lower footer band. |
| `--k-sky` | `#358CCB` | Sky blue — accents, highlight bands, data-viz second series. |
| `--k-sky-light` | `#52A4CA` | Light sky — tooltips, subtle callouts. |
| `--k-cyan` | `#5DC1D4` | Cyan accent — sparingly, for orbital line-work and highlights. |
| `--k-grey` | `#A7A9AC` | **Logo grey** (the orbit swoosh + tagline). Pantone Cool Gray 7 territory. |
| `--k-white` | `#FFFFFF` | Ground. The site is overwhelmingly white. |

### Supporting

| Token | Hex | Role |
|---|---|---|
| `--k-ink` | `#464A4C` | Body copy on light backgrounds. |
| `--k-ink-soft` | `#616161` | Secondary copy. |
| `--k-grey-text` | `#929292` | Muted labels — **decorative/large only**, see contrast below. |
| `--k-blue-pale` | `#C5D3E9` | Text and dividers *on* blue backgrounds. |
| `--k-tint` | `#F1F4FA` | Pale blue-grey panel fill — the standard alternating section band. |
| `--k-hairline` | `#DCE4EC` | Borders, rules, table dividers. |

### Contrast — read before assigning text colours

Computed WCAG ratios. The brand's own site violates several of these; new work should not.

| Pair | Ratio | Verdict |
|---|---|---|
| `#034EA2` on white | **8.0:1** | Passes AA + AAA. Safe for any text. |
| `#003C72` on white | **11.1:1** | Passes AAA. |
| `#464A4C` on white | **9.0:1** | Passes AAA. Default body colour. |
| White on `#034EA2` | **8.0:1** | Safe — the standard reversed treatment. |
| White on `#0071BC` | **5.1:1** | Passes AA normal text. |
| `#C5D3E9` on `#034EA2` | **5.3:1** | Passes AA — the correct muted-on-blue pair. |
| `#358CCB` on white | **3.6:1** | ✗ Fails AA normal text. Large text (≥24px) and UI borders only. |
| `#929292` on white | **3.1:1** | ✗ Fails AA normal text. Use `--k-ink` for body copy instead. |
| `#A7A9AC` on white | **2.4:1** | ✗ Decorative only — never text. |

## Typography

**Montserrat** throughout, loaded in weights 300 / 400 / 500 / 600 / 700.

```css
font-family: 'Montserrat', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
```

| Role | Weight | Notes |
|---|---|---|
| Body | **300** (Light) | The distinctive choice — keep it. Don't default to 400. |
| Emphasis / UI labels | 400–500 | Buttons sit at 500. |
| Section headings (h2) | 500–600 | Always `--k-blue`. |
| Hero / statement (h1) | 600–700 | White on imagery, or `--k-blue` on white. |
| Big numbers / stats | 700 | Large `--k-blue` numerals over a light caption. |

Line height runs tight — roughly **1.1–1.25 for headings**, **1.5–1.6 for body**. Letter-spacing
stays at normal; the only uppercase is on buttons.

> **Caveat:** the theme still ships **DIN Next LT Pro** webfont files (`kacific-fonts.min.css`) from
> an earlier identity, but every `@font-face` for it reports `unloaded` — nothing on the site renders
> in it. Treat DIN as legacy. If print/marketing collateral turns up set in DIN, that's the older
> brand; digital is Montserrat. It is also a licensed Linotype face — don't self-host it into a new
> project without checking the licence.

## Logo

Four files in [assets/](assets/):

| File | Size | Use |
|---|---|---|
| `logo-kacific-full.png` | 350×126 | **Primary.** Wordmark + orbit swoosh + tagline. Light backgrounds. |
| `logo-kacific-small.png` | small | Compact header lockup, tight spaces. |
| `logo-kacific-white.png` | reversed | All-white on transparent. **Only** for dark/blue/photo backgrounds. |
| `favicon-kacific-192.png` | 192×192 | Favicon / app icon. |

**The mark:** the word *Kacific* in heavy `#034EA2`, with a grey `#A7A9AC` ellipse arc sweeping
around the left of the "K" — an orbital path. The tagline *"the heart of broadband"* sits in grey
beneath, right-aligned to the wordmark.

Rules:

- **Clear space:** at least the height of the "K" on every side. The orbit arc is part of the logo —
  never crop it.
- **Minimum width:** ~120px before the tagline stops being legible; below that use the compact mark.
- **Never** recolour the wordmark, add effects, stretch it, place the full-colour version on a dark
  or busy background (use the white variant), or set *Kacific* in a substitute typeface.
- On photography, put the white logo over a sufficiently dark region, or an overlay scrim.

## The orbit ellipse — the signature motif

The strongest visual device on the site, and the thing that carries the identity beyond colour:
**imagery is masked into a large, tilted ellipse** that echoes the logo's swoosh. The hero is a
single wide ellipse bleeding off both edges; content cards below repeat it at smaller scale.
Faint concentric cyan hairline arcs sit in the background behind and below it.

```css
/* Hero: wide tilted ellipse, bleeding past the viewport edges */
.k-hero-media {
  border-radius: 50%;
  transform: rotate(-4deg) scale(1.08);
  overflow: hidden;
}

/* Card image: softer ellipse crop */
.k-card-media {
  border-radius: 50% 50% 50% 50% / 22% 22% 22% 22%;
  overflow: hidden;
}

/* Background orbital hairlines */
.k-orbit-field::before {
  content: '';
  position: absolute;
  inset: -20% -10% auto -10%;
  height: 140%;
  border: 1px solid var(--k-cyan);
  border-radius: 50%;
  opacity: .18;
  pointer-events: none;
}
```

Use it deliberately — one hero ellipse plus a repeating card crop. Ellipses everywhere becomes noise.

## Components

**Pill button** — the house CTA. Fully rounded, uppercase, weight 500. On blue sections it inverts
to white-on-blue-text; on white it's blue-filled.

```css
.k-btn {
  display: inline-block;
  border-radius: 38px;          /* fully rounded, not a small radius */
  padding: 14px 28px 10px;      /* note the optical bottom trim */
  font: 500 14px/1 'Montserrat', sans-serif;
  text-transform: uppercase;
  border: 0;
  background: var(--k-blue);
  color: #fff;
}
.k-btn--invert { background: #fff; color: var(--k-blue); }
```

**Header** — white, sticky, logo hard left, utilities (search, menu) right, sitting under a thin
`--k-blue` announcement bar carrying scrolling news items.

**Section bands** — alternate white and `--k-tint` (`#F1F4FA`). Headings in `--k-blue` at 500–600.

**Footer** — two stacked blue bands, `#07529E` over `#0071BC`, white text, pale-blue links.

**Stat blocks** — an oversized `--k-blue` 700-weight numeral above a small light-weight grey caption.

## Applying this to non-web output

- **Decks / documents:** white ground, `#034EA2` headings, `#464A4C` body at Montserrat Light,
  `#F1F4FA` for callout panels, logo top-left with full clear space.
- **Charts:** sequence `#034EA2` → `#358CCB` → `#5DC1D4` → `#A7A9AC` → `#003C72`. Blue is always the
  primary series. Never put `#929292` labels on white — use `#464A4C`.
- **Artifacts / dark mode:** the brand has no official dark palette. Invert to a `#003C72`-to-near-black
  ground, promote `#358CCB` to the accent role (it has the headroom on dark that it lacks on white),
  and switch to `logo-kacific-white.png`.

## Provenance

Extracted 2026-09-02 from the live https://kacific.com WordPress theme by loading the page in a real
browser (Cloudflare blocks plain HTTP clients) and:

- tallying every hex in ~179KB of theme CSS across 7 stylesheets — `#034EA2` led at 132 occurrences,
  `#358CCB` at 62;
- reading computed styles on every rendered element for the in-use type and colour roles;
- pixel-sampling `logo-kacific.png` on a canvas, which returned exactly `#034EA2` (13% of pixels) and
  `#A7A9AC` (6.5%) — independently confirming the CSS primary.

The theme exposes **no** CSS custom properties of its own; the tokens in `reference/tokens.css` are
this skill's naming, mapped onto the hardcoded values the site uses.

To re-derive after a site redesign, repeat the above with the Playwright browser tools —
`curl` gets a 403 from the Cloudflare challenge.
