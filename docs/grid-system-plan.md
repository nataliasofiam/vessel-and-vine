# Consistent grid — findings and plan

Handoff notes from a planning session. Nothing has been implemented yet; the
working tree is untouched apart from this file.

## How I want to work on this

- **Teacher mode.** I'm learning by doing. Do not write or edit code for me.
  Give precise, staged instructions and let me type them.
- I'm not fluent in Liquid/JS — explain syntax as it comes up, don't assume.
- Build in stages with a checkpoint after each, so I can see something work
  before moving on.

## Goal

Make the hero panels in `vv-hero` line up exactly with the product columns in
`featured-collection`, at every breakpoint, with the column count stepping down
as the viewport narrows.

Design reference: <https://www.sotf.com/en>

## Repo facts

- Shopify **Dawn** theme. Remote `nataliasofiam/vessel-and-vine`, branch `main`.
- Custom sections are prefixed `vv-`:
  - `sections/vv-header.liquid` — notch header (466 lines)
  - `sections/vv-hero.liquid` — hero with glass panels + WebGL shader (524 lines)
  - `sections/vv-product-row.liquid` — **empty file, committed at 0 bytes**
  - `sections/vv-editorial-band.liquid` — **empty file, committed at 0 bytes**
- Homepage (`templates/index.json`) order: `image-banner` (disabled) →
  `vv-hero` → `featured-collection`.
- `.theme-check.yml` and `.prettierrc.json` are present — expect linting.

### Relevant settings (`config/settings_data.json`)

| Setting | Value |
| --- | --- |
| `page_width` | 1200 |
| `spacing_grid_horizontal` | 8 → 8px desktop gutter, 4px mobile |

### Relevant `featured-collection` settings (`templates/index.json`)

`columns_desktop: 4` · `columns_mobile: "2"` · `full_width: false` ·
both sliders off.

### Key line numbers

| File | Line | What |
| --- | --- | --- |
| `assets/base.css` | 84 | `.page-width` — `max-width: var(--page-width)` |
| `assets/base.css` | 98 | `.page-width-desktop` — overrides padding; declared later so it wins |
| `assets/base.css` | 894 | `.grid` — flexbox, not CSS grid |
| `assets/base.css` | 915 | `.grid__item` — percentage widths |
| `assets/base.css` | ~969 | `.grid--3-col-tablet` — **exists but is never emitted** |
| `sections/featured-collection.liquid` | 95 | container classes |
| `sections/featured-collection.liquid` | 99 | grid classes |
| `sections/vv-hero.liquid` | 42 | `.vv-hero__panels` — absolute, full-bleed |
| `sections/vv-hero.liquid` | 49 | `grid-template-columns: repeat({{ cols }}, 1fr)` |
| `sections/vv-hero.liquid` | 63 | `.vv-hero__panel + .vv-hero__panel` — 1px divider |
| `sections/vv-hero.liquid` | 101 | panel render loop |
| `sections/vv-hero.liquid` | 130 | `MAX_COLS = 8` |
| `sections/vv-hero.liquid` | 188–215 | fragment shader main() |
| `sections/vv-hero.liquid` | 498–521 | block schema + preset |
| `layout/theme.liquid` | 149 | `--page-width: 120rem` |
| `layout/theme.liquid` | 206 | `--grid-desktop-horizontal-spacing` |

## Diagnosis

The mismatch is **categorical, not a rounding error** — the two elements are
measured in different coordinate systems.

| | Container | Columns | Gutter |
| --- | --- | --- | --- |
| Hero panels | none; `width: 100%`, full viewport bleed | `repeat(4, 1fr)` of the **viewport** | 0 (1px dividers) |
| Product grid | `page-width` + `page-width-desktop` → max **1200px**, 50px padding at ≥990 | 4 cols of the **1100px content box** | **8px** |

At a 1440px viewport, in absolute page coordinates:

- Hero panel edges: `0 · 360 · 720 · 1080 · 1440`
- Product column edges: `170·439 / 447·716 / 724·993 / 1001·1270`

First internal boundary is **83px** out; outer edges are **170px** out.

Two things already correct:

- `columns_mobile` is already `"2"`.
- Below 990px, `.page-width-desktop` zeroes the padding, so the product grid is
  **already full-bleed** and already agrees with the hero. Only desktop is broken.

## Reference measurements (sotf.com)

Measured in-browser; the site is Bootstrap-based.

| | Finding |
| --- | --- |
| Container | **None.** Everything is `row g-0`, content at `left: 0`, full document width. |
| Gutters | **Zero** (`g-0` everywhere) |
| Separation | 1px hairlines (they have a `borderBottom1pxWhite` utility) |
| Hero | `row g-0 home_cover` → full-bleed, internally split |
| Product columns | Slick carousels with **three** column counts: 4 / 3 / 2 |

Slick's breakpoint semantics depend on a `mobileFirst` flag that wasn't
captured, so the exact width→count map is unconfirmed. The load-bearing fact is
that there are three steps.

**This is why the plan reversed direction.** `vv-hero` — full-bleed, zero
gutter, 1px dividers — already matches the reference idiom. Dawn's contained
1200px `featured-collection` is the element that's off-system.

## Decisions taken

1. **Full-bleed the product grid**, don't contain the hero. Gutter 8px → 0,
   1px hairlines between cards. Hero geometry stays as-is.
2. **Hero panel count becomes numeric settings** (desktop/tablet/mobile),
   replacing the block list. **Glass is derived from position** — first and last
   column — rather than a per-block checkbox.

Decision 2 matters because the shader's existing `isOuter` term already encodes
"outer columns are special":

```glsl
float isOuter = max(1.0 - step(0.5, colIndex), step(u_cols - 1.5, colIndex));
```

With glass derived, `glass` and `isOuter` become the same value computed twice,
so `u_glass[]`, `glassAt()` and the `data-glass` attribute can all be deleted.
The shader gets shorter.

## The grid, settled

| | Value |
| --- | --- |
| Container | None — full-bleed, 100% of viewport |
| Gutter | 0 |
| Separation | 1px hairline |
| Columns | **4** desktop · **3** tablet · **2** mobile |
| Breakpoints | **990px** and **750px** |

Breakpoints are **Dawn's own** (`grid--*-col-desktop` at min-width 990,
`grid--*-col-tablet` at min-width 750), not SOTF's ~1025/1200 — adopting those
would plant a third breakpoint system in a theme that already has one. Both the
hero and the product grid must switch at the same two widths.

## Plan

| Part | What |
| --- | --- |
| 1 | **Column ruler** — dev-only overlay, `?grid` to toggle. Four full-bleed stripes at viewport quarters, stepping to 3 and 2 at the breakpoints. Build this first; you can't fix a grid you can't see. |
| 2 | **Product grid → full-bleed.** Drop the `page-width` classes; gutter to zero. Hairlines via `gap: 1px` + background colour on the container, with opaque card backgrounds — not `border-left` on wrapping flex items. |
| 3 | **Add the tablet step.** Dawn ships `.grid--3-col-tablet`; `featured-collection.liquid:99` never emits it (jumps from `columns_mobile` straight to `columns_desktop` via `grid--{{ columns_mobile }}-col-tablet-down`). Class-list edit + new setting. |
| 4 | **Hero panel count.** Blocks out, three number settings in. Render 4 panels; per breakpoint set `repeat(N, 1fr)` and hide the surplus. Glass = three small media blocks: `:nth-child(1),(4)` desktop · `(1),(3)` tablet · `(1),(2)` mobile. |
| 5 | **Shader diet.** Delete `u_glass` / `glassAt()` / `data-glass`, drive glass from `isOuter`. Add a `matchMedia` listener to update `u_cols` on breakpoint change — it's currently uploaded once at init from `data-cols`. |
| 6 | **Guardrails.** Record the four numbers. Decide on the two empty stub sections — build or delete, don't leave 0-byte files committed. |

## Watch out for

- **Zeroing the gutter globally.** `spacing_grid_horizontal: 8` spaces every
  grid on the site — collection pages, search, related products. Scope the
  change to `featured-collection` first, look at both pages, then decide whether
  to go global. Scoping keeps the blast radius small but means the homepage and
  collection-page grids stop matching.
- **`--vv-gutter` is already taken.** Defined on `:root` in
  `sections/vv-header.liquid:9`, it means "header shell inset", not "grid
  gutter". Pick a different name for any grid token.
- **The shader's columns are viewport-relative.** `colWidth = 1.0 / u_cols` over
  `uv.x` spanning the full canvas. That stays correct under the chosen plan —
  but only because the hero stays full-bleed. If the panels ever move onto a
  contained grid, the shader needs the content inset and gutter as uniforms.
- **Panel dividers survive hiding.** `.vv-hero__panel + .vv-hero__panel` stays
  correct when surplus panels are `display: none`, because they're always at the
  end and the visible ones stay contiguous from the left.
- **Ruler must not use `100vw`.** Use `inset: 0`. `100vw` includes the scrollbar
  and page content doesn't, which produces a phantom ~15px offset — exactly the
  bug the ruler exists to find.
