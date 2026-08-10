# Consistent grid — state and next steps

Handoff notes. Parts 1–3 are **done and committed**. Part 4a is **in progress**:
Stage A is typed but uncommitted, Stage B is the next thing to type. The hero is
**visibly broken until Stage B lands** — read "Where we are" before touching
anything.

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

## The grid, settled

| | Value |
| --- | --- |
| Container | None — full-bleed, 100% of viewport |
| Gutter | 0 |
| Separation | 1px hairline |
| Columns | **4** desktop · **3** tablet · **2** mobile |
| Breakpoints | **990px** and **750px** (Dawn's own) |

---

## Where we are

Branch `consistent-grid`. Commits so far:

| Commit | What |
| --- | --- |
| `3666cdca` | ruler guides |
| `dd3ec596` | featured collection grid breakpoints and hairlines |
| `6530a5b1` | start hero work to adhere to same breakpoints — Part 4a Step 1 |

Uncommitted on top: `sections/vv-hero.liquid`, Stage A only (the three `cols_*`
range settings in the schema).

### Done — Part 1, column ruler

`snippets/vv-grid-ruler.liquid`, rendered from `layout/theme.liquid` just before
`</body>`. Append `?grid` to any URL to show it. Four full-bleed stripes at
`position: fixed; inset: 0` (**not** `100vw` — that includes the scrollbar and
invents a ~15px offset), stepping 4 → 3 → 2 at 990 and 750.

Delete the snippet and its `{% render %}` when the grid work lands.

### Done — Parts 2 and 3, product grid full-bleed

All in `sections/featured-collection.liquid`.

- Dropped `page-width` / `page-width-desktop` from the `<slider-component>`
  class list. Removing `page-width-desktop` also activates
  `template-collection.css:87`, which zeroes the padding at ≥990 on its own.
- Gutter zeroed by **overriding Dawn's spacing custom properties** on the
  section's `.grid`, not by rewriting widths. Every Dawn column width is
  `calc(25% - var(--grid-desktop-horizontal-spacing) * 3 / 4)`, so shadowing the
  variable collapses them all to clean percentages. Scoped to this section, so
  collection/search pages keep their 8px.
- Column counts come from three of Dawn's shipped classes —
  `grid--2-col grid--3-col-tablet grid--4-col-desktop` — replacing
  `grid--N-col-tablet-down`. They cascade correctly on **source order alone**
  (base.css 956 → 961 → 969 → 1017), no specificity overrides.
- New `columns_tablet` range setting, default 3, recorded in `templates/index.json`.
- Hairlines via `box-shadow` (draws outside the box, invisible to layout — a
  `border-left` would have widened the items and wrapped the row), with
  `:not(:nth-child(Nn + 1))` per breakpoint.
- Row gap zeroed too; `padding: 2rem 0` on the items gives the horizontal rules
  breathing room. Vertical padding is safe on content-box items; horizontal
  would not be.

### Done — Part 4a Stage A, the three column settings

In `sections/vv-hero.liquid`, uncommitted. A `"Columns"` header plus
`cols_desktop` / `cols_tablet` / `cols_mobile` ranges (1–8, defaults 4/3/2),
inserted above the `"Distortion"` header. Schema validates; the theme editor
shows three sliders.

**Stage A was moved ahead of the CSS on purpose.** The doc originally had the
schema at Step 5, after the panel CSS. That ordering doesn't work: the CSS reads
`section.settings.cols_desktop`, and until the schema *declares* that setting the
lookup returns nil — so you'd type the whole of Step 3 and reproduce the exact
same empty-`repeat()` bug one layer down, with nothing on screen to show for it.
With the settings declared, Shopify supplies the schema `default` even though
`templates/index.json` has no `cols_*` keys yet, so the Liquid variables resolve
immediately and Stage B has something real to render.

### Broken right now — Part 4a, Stage B not typed

Two places still reference the `cols` that Step 1 deleted:

- **line 56** — `grid-template-columns: repeat({{ cols }}, 1fr);`
- **line 85** — `data-cols="{{ cols }}"`

**What you actually see: the panels stack as rows, one per panel.** Liquid
renders an undefined variable as empty string rather than erroring, so the
browser gets `grid-template-columns: repeat(, 1fr)`, drops that one declaration
as invalid, and keeps the rest of the rule. `.vv-hero__panels` is still
`display: grid`, and a grid with no declared columns falls back to one implicit
column with auto rows. (An earlier version of this doc said "shows one panel" —
it's one *column*, N rows.)

Line 85 fails the same way: `data-cols=""` → `parseInt('', 10)` is `NaN` →
`|| 1` → the shader draws a single column.

Stage B resolves both. Escape hatch: `git checkout sections/vv-hero.liquid`
reverts **only Stage A** now — Step 1 is committed in `6530a5b1`, so that
command lands you back in this broken state, not before it.

---

## Things learned the hard way

- **`0px`, not `0`.** `calc(25% - 0 * 3 / 4)` is a parse error and the whole
  declaration gets dropped, silently reverting the width. Unitless zero is fine
  almost everywhere else in CSS.
- **Dawn has no global `box-sizing: border-box`** — only scattered
  per-component ones. Items are content-box, so horizontal padding adds to
  percentage widths and wraps rows. Vertical padding is always safe.
- **Nested media queries AND together.** A whole stage was lost to rules pasted
  inside the section's existing `@media (min-width: 750px)` block: desktop and
  tablet worked, mobile silently got nothing, and a nested
  `@media (max-width: 749px)` was unreachable code. Check the brace depth.
- **Read `getComputedStyle`, don't trust the eye.** A 4px gutter at mobile
  looked "clean" against the ruler for two whole stages. This one-liner in the
  console settles it:

  ```js
  (() => {
    const g = document.querySelector('.product-grid');
    const li = g.children;
    return [
      'cols=' + getComputedStyle(g).columnGap,
      'rows=' + getComputedStyle(g).rowGap,
      'shadow=' + getComputedStyle(li[1]).boxShadow,
      'gapPx=' + (li[1].getBoundingClientRect().left - li[0].getBoundingClientRect().right),
      'width=' + innerWidth,
    ].join(' | ');
  })()
  ```

  `console.log()` returns `undefined`, which is what the console echoes — return
  a string from an IIFE instead so the result line *is* the answer.
- **Prettier 3 dropped plugin auto-discovery.** `@shopify/prettier-plugin-liquid`
  has to be listed in `.prettierrc.json`'s `plugins` array or you get
  "No parser could be inferred". Now installed and wired up; `node_modules` is
  gitignored. Always pass a path — a bare `--write .` reformats all of Dawn.
- **`--vv-gutter` is already taken** by `sections/vv-header.liquid:9`, where it
  means "header shell inset". Pick different names. In use so far:
  `--vv-rule-color` (featured-collection), `--vv-glass` / `--vv-glass-bg` (hero,
  pending).
- **Three mutually exclusive media ranges beat cascade layering.** Used for both
  the hairlines and the hero panels: `max-width: 749`, `750–989`, `min-width: 990`.
  Exactly one matches, so there is no override reasoning to get wrong.
- **Liquid renders an undefined variable as empty string — it does not error.**
  A renamed variable leaves no trace at the Liquid layer; the damage shows up as
  malformed CSS the browser silently drops, or an empty data attribute that
  `parseInt` turns into `NaN`. When a rename is half-done, grep the old name
  across the whole file before pushing. This cost a full debugging round.
- **A mistyped JSON *key* passes every syntax check.** Typing `"type:": "range"`
  (colon inside the quotes) is valid JSON — the document parses fine. It failed
  one layer later, at Shopify's schema validation: `Invalid schema: setting with
  id="cols_desktop" type is required`. Validation stops at the **first** failure,
  so an error naming one setting can mean all three are wrong. Read such messages
  as "the key is missing *or misspelled*," and fix every occurrence at once.
- **Shopify CLI can push a section group before the section it references.**
  `Failed to upload sections/header-group.json … Section type 'vv-header' does
  not refer to an existing section file` — the file existed locally *and* on the
  remote. The CLI uploads in concurrent batches, so the JSON can reach the server
  before the `.liquid` does ([shopify-cli#2450](https://github.com/Shopify/shopify-cli/issues/2450)).
  Which section it blames varies run to run. **Fix: push again** — the `.liquid`
  landed on the first pass, so the second validates. Expect this on any fresh dev
  theme where the `vv-*` sections don't exist remotely yet. Tell-tale signature:
  the `.liquid` is on the remote and byte-identical, but the group JSON is stale.

---

## Next: finish Part 4a — hero panel count as settings

Blocks out, three number settings in. Render `cols_max` panels; per breakpoint
set `repeat(N, 1fr)` and hide the surplus. **Glass is derived from position** —
first and last visible column — rather than a per-block checkbox.

Current line numbers in `sections/vv-hero.liquid`:

| Line | What |
| --- | --- |
| 17 | `.vv-hero` rule |
| 49 | `.vv-hero__panels` |
| 56 | `grid-template-columns` — **stale `{{ cols }}`** |
| 60 | `.vv-hero__panel` |
| 64 | `.vv-hero__panel--glass` |
| 70 | `.vv-hero__panel + .vv-hero__panel` — 1px divider |
| 74 | `@media (max-width: 750px)` — off by one, see below |
| 85 | `data-cols` — **stale `{{ cols }}`** |
| 86 | `data-glass` |
| 108 | panel render loop |
| 444 | `"Columns"` header + the three `cols_*` ranges — **Stage A, done** |
| 536 | block schema |
| 550 | preset blocks |

Schema line numbers shifted +31 when Stage A went in; everything inside
`<style>` (17–79) is unmoved.

## Stage B — next thing to type

Steps 2 and 3 below, both inside `<style>`. This is the stage where it works:
columns come back, dividers land on the ruler lines and on the product grid
hairlines, glass on the first and last visible panel at every breakpoint.

### Step 2 — glass as value aliases

Add to the existing `.vv-hero` rule (line 17):

```liquid
    --vv-glass: blur({{ section.settings.glass_blur }}px) saturate(1.15) brightness(1.05);
    --vv-glass-bg: rgba(255, 255, 255, {{ section.settings.glass_tint | divided_by: 1000.0 }});
```

Glass has to be applied in three separate media blocks, since which columns are
"outer" changes per breakpoint. These aliases keep the values in one place.

`divided_by: 1000.0` — the `.0` is load-bearing. Liquid does **integer** division
when both operands are integers, so `30 | divided_by: 1000` is `0` and the tint
vanishes. Same trap as the `0px` one above: silently wrong, not an error.

### Step 3 — the panel CSS

Replace lines 49–78 (`.vv-hero__panels` through the close of the old mobile
height query) with:

```liquid
  .vv-hero__panels {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    display: grid;
    pointer-events: none;
  }

  .vv-hero__panel {
    height: 100%;
  }

  .vv-hero__panel + .vv-hero__panel {
    border-left: 1px solid rgba(255, 255, 255, {{ section.settings.divider_opacity | divided_by: 100.0 }});
  }

  @media screen and (max-width: 749px) {
    .vv-hero {
      height: {{ section.settings.hero_height_mobile }}vh;
    }

    .vv-hero__panels {
      grid-template-columns: repeat({{ cols_mobile }}, 1fr);
    }

    .vv-hero__panel:nth-child(n + {{ cols_mobile | plus: 1 }}) {
      display: none;
    }

    .vv-hero__panel:nth-child(1),
    .vv-hero__panel:nth-child({{ cols_mobile }}) {
      backdrop-filter: var(--vv-glass);
      -webkit-backdrop-filter: var(--vv-glass);
      background: var(--vv-glass-bg);
    }
  }

  @media screen and (min-width: 750px) and (max-width: 989px) {
    .vv-hero__panels {
      grid-template-columns: repeat({{ cols_tablet }}, 1fr);
    }

    .vv-hero__panel:nth-child(n + {{ cols_tablet | plus: 1 }}) {
      display: none;
    }

    .vv-hero__panel:nth-child(1),
    .vv-hero__panel:nth-child({{ cols_tablet }}) {
      backdrop-filter: var(--vv-glass);
      -webkit-backdrop-filter: var(--vv-glass);
      background: var(--vv-glass-bg);
    }
  }

  @media screen and (min-width: 990px) {
    .vv-hero__panels {
      grid-template-columns: repeat({{ cols_desktop }}, 1fr);
    }

    .vv-hero__panel:nth-child(n + {{ cols_desktop | plus: 1 }}) {
      display: none;
    }

    .vv-hero__panel:nth-child(1),
    .vv-hero__panel:nth-child({{ cols_desktop }}) {
      backdrop-filter: var(--vv-glass);
      -webkit-backdrop-filter: var(--vv-glass);
      background: var(--vv-glass-bg);
    }
  }
```

Notes:

- No base `grid-template-columns` — the three ranges are exhaustive, so a base
  value would be dead.
- `{{ cols_mobile | plus: 1 }}` — CSS can't do arithmetic inside `:nth-child()`,
  so Liquid does it at render time. With 2 it compiles to `:nth-child(n + 3)`.
- The old `@media (max-width: 750px)` overlapped Dawn's `min-width: 750px` at
  exactly 750. Folding the mobile height into the `749px` block fixes it, so
  hero and grid step on the same pixel.
- The divider needs no changes. Hidden panels are always at the end, so the
  visible run stays contiguous and `+` still targets exactly panels 2..N.

### Checkpoint B

Load with `?grid`. Expect 4 / 3 / 2 columns across the two breakpoints, dividers
on the ruler lines and on the product grid hairlines, hero shortening to
`hero_height_mobile` at ≤749, glass on first and last visible panel (at tablet
that's 1 and 3).

Two things that will look wrong and shouldn't be chased: the cursor lens is
weaker and the static edge refraction is gone (shader still reads `data-cols=""`
→ 1; Part 4b fixes it), and the panel count may read 3 rather than 4 — see the
open question below.

---

## Stage C — Steps 4, 5, 6

These three move together and must land in one go: Step 5 stops the schema
declaring the `panel` block type, which orphans the saved blocks in
`templates/index.json` (Step 6) and would leave the Step 4 loop with nothing to
iterate. Don't checkpoint between them.

### Step 4 — markup

Panels loop (line 108):

```liquid
  <div class="vv-hero__panels">
    {%- for i in (1..cols_max) -%}
      <div class="vv-hero__panel">&nbsp;</div>
    {%- endfor -%}
  </div>
```

`(1..cols_max)` is a Liquid range literal; `i` is unused, only the count matters.

In the `<div class="vv-hero">` attributes: **delete `data-glass`** (line 86) and
make the columns explicit:

```liquid
  data-cols="{{ cols_desktop }}"
  data-cols-tablet="{{ cols_tablet }}"
  data-cols-mobile="{{ cols_mobile }}"
```

`data-cols` stays until 4b retires it.

### Step 5 — schema

The three `cols_*` ranges are **already in** — that was Stage A, line 444.
(`max: 8` mirrors the shader's `MAX_COLS`.)

What's left: delete the whole `"blocks"` array (line 536) and reduce the preset
(line 550) to `[{ "name": "Hero" }]`.

### Step 6 — index.json

Not optional. `vv_hero_MdexzK` in `templates/index.json` still holds four saved
`panel` blocks plus `block_order`. Once the schema stops declaring that type
they're orphans and the section fails to render. Delete both, and add to its
`"settings"`:

```json
        "cols_desktop": 4,
        "cols_tablet": 3,
        "cols_mobile": 2,
```

The file header says auto-generated — true, but hand-editing is the normal way
to do a schema migration. Don't have the theme editor open on this template
while saving.

**Open question, resolve before typing this step.** Local `index.json` records
four `panel` blocks, but the hero appeared to render **three** rows. If Checkpoint
B confirms three, the theme editor has saved over this template and the local file
is stale — pull it first (`shopify theme pull --only templates/index.json`) so the
migration edits the version that's actually live, or the delete will miss a block.
Checkpoint B is what settles it, which is why Stage C comes after.

### Checkpoint 4a

- 4 panels desktop, 3 tablet, 2 mobile; dividers on the ruler lines **and** on
  the product grid hairlines
- Glass on first and last panel at every breakpoint — at tablet that's 1 and 3
- Theme editor shows three column sliders and no panel blocks

**Expected regression until 4b:** the cursor lens looks weaker and the static
edge refraction disappears, because the shader still reads the now-deleted
`data-glass` and gets all zeros.

---

## Then: Part 4b — shader diet

`glass` and `isOuter` are now the same value computed twice, so the whole glass
uniform path can go. The shader gets shorter.

```glsl
float isOuter = max(1.0 - step(0.5, colIndex), step(u_cols - 1.5, colIndex));
```

In the `fragSrc` array:

- delete `'uniform float u_glass[' + MAX_COLS + '];'` (line 171)
- delete the `glassAt()` function (lines 176–182)
- delete `'  float glass = glassAt(colIndex);'` (line 201)
- `staticOffset` (line 207): `glass * …` → `isOuter * …`
- `amp`: `(1.0 + glass) * isOuter` → `(1.0 + isOuter) * isOuter`

In the JS: drop the `glassFlags` parsing, the `uGlass` location lookup (line
270), and the `gl.uniform1fv(uGlass, …)` upload. Keep `MAX_COLS` — still used to
clamp `cols`.

Checkpoint: glass panels regain their edge refraction and the stronger cursor
lens, and the effect stays confined to the outer columns.

---

## Then: Part 5 — `u_cols` on breakpoint change

`u_cols` is uploaded once at init from `data-cols`, so after 4a the shader keeps
drawing 4 columns even when the CSS has stepped to 3 or 2. The distortion seams
drift off the panel dividers.

Add a `matchMedia` listener that recomputes from the three data attributes and
re-uploads:

- `(min-width: 990px)` → `data-cols`
- `(min-width: 750px)` → `data-cols-tablet`
- else → `data-cols-mobile`

Call it once at init (replacing the current `gl.uniform1f(uCols, cols)`) and on
each `change` event. `matchMedia(...).addEventListener('change', …)` is the
modern form. Once this works, rename `data-cols` to `data-cols-desktop` for
symmetry.

Checkpoint: resize across both breakpoints with the cursor in the hero — the
refraction seams should stay glued to the dividers.

---

## Then: Part 6 — guardrails

- Record the four numbers (4/3/2, breakpoints 990/750) somewhere durable —
  `columns_tablet` and the three `cols_*` settings are already in
  `templates/index.json`; this doc is the prose copy.
- **Decide the two 0-byte stubs**: `sections/vv-product-row.liquid` and
  `sections/vv-editorial-band.liquid` are committed empty. Build or delete —
  don't leave them.
- Remove the ruler: delete `snippets/vv-grid-ruler.liquid` and its `{% render %}`
  in `layout/theme.liquid`.
- Open question deferred from Part 2: the section heading still carries
  `page-width` (`featured-collection.liquid:75`), so it's contained at 1200
  while the grid below it is full-bleed. Decide whether the heading should align
  to the grid's outer edge.
- Open question: the gutter change is scoped to `featured-collection`. Collection
  and search pages still have the 8px contained grid, so the homepage and
  collection pages no longer match. Look at both, then decide whether to go
  global via `spacing_grid_horizontal`.

## Watch out for

- **The shader's columns are viewport-relative.** `colWidth = 1.0 / u_cols` over
  `uv.x` spanning the full canvas. Correct only because the hero stays
  full-bleed. If the panels ever move onto a contained grid, the shader needs
  the content inset and gutter as uniforms.
- **Zeroing the gutter globally** would touch every grid on the site.
