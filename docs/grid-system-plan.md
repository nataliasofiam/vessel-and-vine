# Consistent grid — state and next steps

Handoff notes. Parts 1–3 are **done and committed**. Part 4a is **in progress**:
Stages A and B are both committed and the hero renders correctly at all three
breakpoints. Stage C — Steps 4, 5, 6 — is the next thing to type, and it has an
open question to settle first. Read "Where we are" before touching anything.

## How I want to work on this

See the root [`CLAUDE.md`](../CLAUDE.md) — teacher mode, and the copy / tweak /
create paradigm. It applies to the whole project, not just this doc.

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
| `5a4b1afd` | fix missing schema for column breakpoints — Part 4a **Stage A** |
| `c9a5673b` | hero panel columns per breakpoint — Part 4a **Stage B** |

Working tree clean. Nothing is broken right now.

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

In `sections/vv-hero.liquid`, committed as `5a4b1afd`. A `"Columns"` header plus
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

### Done — Part 4a Stage B, the panel CSS

Committed as `c9a5673b`. Three mutually exclusive media ranges in `<style>` —
`max-width: 749`, `750–989`, `min-width: 990` — each setting
`grid-template-columns: repeat(N, 1fr)` from its own `cols_*` variable, hiding
the surplus with `:nth-child(n + N+1)`, and applying glass by position via the
`--vv-glass` / `--vv-glass-bg` aliases on `.vv-hero`. The
`.vv-hero__panel--glass` class rule is gone; the markup still emits the class
(line 166) but nothing matches it until Step 4.

There is **no base `grid-template-columns`** — the three ranges are exhaustive,
so a base value would be dead. The mobile height override moved inside the 749
block, so the hero and the product grid now step on the same pixel.

**Glass is not symmetric at mobile.** The original rule was "first and last
visible column" at every breakpoint. At 2 columns that is *every* column, and
`backdrop-filter` applied everywhere reads as nothing at all — there is no
unfiltered image left to compare against. So the mobile block glasses only
`:nth-child({{ cols_mobile }})`, the right-hand column. Tablet and desktop keep
the first-and-last pair. This has a knock-on for Part 4b — see there.

### Still stale — one `cols` reference left

**Line 140** — `data-cols="{{ cols }}"`, referencing the variable Step 1
deleted. Liquid renders an undefined variable as empty string rather than
erroring, so this emits `data-cols=""` → `parseInt('', 10)` is `NaN` → `|| 1` →
the shader draws a single column. Visible as a weaker cursor lens and no static
edge refraction. **Step 4 fixes it** — it is a known, expected regression until
then, not something to chase.

(An earlier version of this doc claimed Stage B resolved both `cols`
references. It resolved only the CSS one — Stage B is entirely inside
`<style>`.)

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
- **Custom properties are validated at *substitution*, not declaration.** A `--*`
  property accepts almost any token stream, so `brightness (1.05)` — one stray
  space, which stops it being a function call — sits in devtools looking
  perfectly healthy. It fails one layer later, when `var(--vv-glass)` is
  substituted into `backdrop-filter` and that declaration gets dropped. Same
  shape as the Liquid empty-string trap: no error anywhere, just a missing
  effect. If a `var()` does nothing, suspect the *declaration*, not the use.
- **A misspelled selector is indistinguishable from a cascade problem, until you
  look.** `.vv-hero_panel` (one underscore) instead of `.vv-hero__panel` cost a
  round: the rules were syntactically perfect and simply matched no element.
  Tell them apart in devtools — a rule losing a specificity fight still *appears*
  in the Styles pane, struck through. A rule that never appears at all isn't
  being overridden, it isn't matching. Check the selector before the cascade.
- **"First and last" collapses at two columns.** Any rule phrased as outer-edge
  styling needs a sanity check at the smallest breakpoint, where first and last
  can be the same element or the entire set. `backdrop-filter` applied to every
  column is invisible — the effect needs unfiltered pixels beside it to read
  against. This is why mobile glasses one column and the shader's `isOuter` now
  disagrees with the CSS below 750px.

---

## Next: finish Part 4a — hero panel count as settings

Blocks out, three number settings in. Render `cols_max` panels; per breakpoint
set `repeat(N, 1fr)` and hide the surplus. **Glass is derived from position**
rather than a per-block checkbox — first and last visible column at tablet and
desktop, right-hand column only at mobile.

Stages A and B are done. What's left is Stage C: the markup, the schema, and the
`templates/index.json` migration, which land together.

Current line numbers in `sections/vv-hero.liquid`, as of `c9a5673b`:

| Line | What |
| --- | --- |
| 17 | `.vv-hero` rule — also holds `--vv-glass` / `--vv-glass-bg` (22–23) |
| 51 | `.vv-hero__panels` — no base `grid-template-columns`, by design |
| 61 | `.vv-hero__panel` |
| 65 | `.vv-hero__panel + .vv-hero__panel` — 1px divider |
| 70 | `@media (max-width: 749px)` — mobile: columns, hide, glass, hero height |
| 94 | `@media (min-width: 750px) and (max-width: 989px)` — tablet |
| 115 | `@media (min-width: 990px)` — desktop |
| 140 | `data-cols` — **stale `{{ cols }}`**, Step 4 fixes |
| 141 | `data-glass` — Step 4 deletes |
| 163 | `.vv-hero__panels` wrapper |
| 164 | panel render loop — Step 4 rewrites |
| 501 | `"Columns"` header + the three `cols_*` ranges — **Stage A, done** |
| 591 | block schema — Step 5 deletes |
| 605 | presets — Step 5 reduces |

Stage B added ~55 lines inside `<style>`, so everything below it shifted:
markup +55, schema +57. The `.vv-hero` rule at 17 is unmoved.

## Stage C — Steps 4, 5, 6

These three move together and must land in one go: Step 5 stops the schema
declaring the `panel` block type, which orphans the saved blocks in
`templates/index.json` (Step 6) and would leave the Step 4 loop with nothing to
iterate. Don't checkpoint between them.

### Step 4 — markup

Panels loop (lines 163–169):

```liquid
  <div class="vv-hero__panels">
    {%- for i in (1..cols_max) -%}
      <div class="vv-hero__panel">&nbsp;</div>
    {%- endfor -%}
  </div>
```

`(1..cols_max)` is a Liquid range literal; `i` is unused, only the count matters.

This also retires `.vv-hero__panel--glass` from the markup — Stage B already
deleted the rule it pointed at, so the class has been inert since `c9a5673b`.

In the `<div class="vv-hero">` attributes: **delete `data-glass`** (line 141),
fix the stale `data-cols` (line 140), and make the columns explicit:

```liquid
  data-cols="{{ cols_desktop }}"
  data-cols-tablet="{{ cols_tablet }}"
  data-cols-mobile="{{ cols_mobile }}"
```

`data-cols` stays until 4b retires it.

### Step 5 — schema

The three `cols_*` ranges are **already in** — that was Stage A, line 501.
(`max: 8` mirrors the shader's `MAX_COLS`.)

What's left: delete the whole `"blocks"` array (line 591) and reduce the preset
(line 605) to `[{ "name": "Hero" }]`.

### Step 6 — index.json

Not optional. `vv_hero_MdexzK` in `templates/index.json` still holds four saved
`panel` blocks plus `block_order` — verified 2026-08-12:

```
panel_C4az3T  panel_kHqYXc  panel_kL6HaK  panel_Hqfc6b
```

and **no `cols_*` keys in its `"settings"`**, which is why the schema defaults
have been supplying 4/3/2 since Stage A. Once the schema stops declaring the
`panel` type those four blocks are orphans and the section fails to render.
Delete `blocks` and `block_order`, and add to its `"settings"`:

```json
        "cols_desktop": 4,
        "cols_tablet": 3,
        "cols_mobile": 2,
```

The file header says auto-generated — true, but hand-editing is the normal way
to do a schema migration. Don't have the theme editor open on this template
while saving.

**Open question — STILL OPEN, resolve before typing this step.** Local
`index.json` records four `panel` blocks, but the hero appeared to render
**three** rows. Checkpoint B passed on layout, but the panel count at desktop was
never actually counted, so this is unsettled. Count it before starting Stage C:
with `cols_desktop` at 4, four panels should be visible. If you see three, the
theme editor has saved over this template and the local file is stale — pull it
first (`shopify theme pull --only templates/index.json`) so the migration edits
the version that's actually live, or the delete will miss a block.

Cheap way to settle it without counting by eye:

```js
document.querySelectorAll('.vv-hero__panel').length
```

That counts rendered panels, including any hidden by `display: none`, so it
reports what the loop produced rather than what the breakpoint shows. Four means
the local file is accurate.

### Checkpoint 4a

- 4 panels desktop, 3 tablet, 2 mobile; dividers on the ruler lines **and** on
  the product grid hairlines
- Glass on the first and last visible panel at desktop and tablet — at tablet
  that's 1 and 3 — and on the right-hand panel only at mobile
- Theme editor shows three column sliders and no panel blocks

**Expected regression until 4b:** the cursor lens looks weaker and the static
edge refraction disappears, because the shader still reads the now-deleted
`data-glass` and gets all zeros.

---

## Then: Part 4b — shader diet

`glass` and `isOuter` are the same value computed twice, so the whole glass
uniform path can go. The shader gets shorter.

```glsl
float isOuter = max(1.0 - step(0.5, colIndex), step(u_cols - 1.5, colIndex));
```

In the `fragSrc` array:

- delete `'uniform float u_glass[' + MAX_COLS + '];'` (line 226)
- delete the `glassAt()` function (lines 231–237)
- delete `'  float glass = glassAt(colIndex);'` (line 256)
- `staticOffset` (line 262): `glass * …` → `isOuter * …`
- `amp` (line 269): `(1.0 + glass) * isOuter` → `(1.0 + isOuter) * isOuter`

In the JS: drop the `glassFlags` parsing (lines 195–199), the `uGlass` location
lookup (line 325), and the `gl.uniform1fv(uGlass, …)` upload (line 333). Keep
`MAX_COLS` — still used to clamp `cols`.

**Decide this before typing: `isOuter` and the CSS disagree at mobile.** The
premise above — glass and `isOuter` are the same value — holds at 3 and 4
columns only. With `u_cols` = 2 the GLSL returns `1.0` for *both* columns
(`colIndex` 0 matches the first term, `colIndex` 1 matches `step(0.5, 1)`), which
is exactly the degenerate case Stage B rejected in CSS by glassing only the
right-hand column. So after this change the refraction would appear on a mobile
panel that has no glass on it. Two ways out:

1. Teach the shader the same exception — at `u_cols` <= 2, treat only the last
   column as outer. One extra term in the `isOuter` expression.
2. Accept the mismatch below 750px and note it here as intentional.

Not urgent — the shader reads `data-cols=""` → 1 column until Step 4 lands — but
it must be settled before the `u_glass` path is deleted, because deleting it is
what removes the ability to express "these specific panels are glass."

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

- Record the four numbers (4/3/2, breakpoints 990/750) somewhere durable.
  `featured_collection` already has `columns_desktop` / `columns_tablet` /
  `columns_mobile` saved in `templates/index.json` (4 / 3 / "2" — note the
  string, that's Dawn's own type, not a mistake). The hero's three `cols_*` are
  **not** there yet; Step 6 adds them. Until then the hero runs on schema
  defaults, so the two sections agree by coincidence rather than by record.
  This doc is the prose copy.
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
