# Consistent grid — state and next steps

Handoff notes. **Parts 1 through 6 are committed and the grid system works.**
The hero panels line up with the product columns at every breakpoint, driven by
three settings; glass and distortion share one definition of "outer"; the gutter
is zeroed theme-wide. The scaffolding (ruler, empty stubs) is deleted.

**Next is Part 7** — reshaping `featured-collection` into a full-width
horizontal scroller with no heading, to match the reference. Note that 7a
supersedes a Part 6 decision.

**Knowingly incomplete:** the hairlines are still scoped to
`featured-collection`, so collection and search pages have the zero gutter
without the rules. See "Still open after Part 6". Read "Where we are" first.

**Part 5 was moved ahead of Part 4b**, against the original order. Two reasons,
both worth keeping: Part 5 fixed a visible defect while 4b is a cleanup with
almost no visible effect, and 4b's open question was about `isOuter` at
`u_cols` = 2 — a state the shader could never reach until Part 5 let `u_cols`
change at all. Doing 5 first turned an abstract decision into something
observable.

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
| `d1cbeb80` | hero breakpoint driven by settings, not blocks — Part 4a **Stage C** |
| `b026c793` | record Part 4a complete in grid doc |
| `8df0beb5` | shader column count follows the breakpoint — **Part 5** |
| `a50db993` | stop overscroll exposing white above the hero — header, not grid |
| `80e203fb` | document Part 5 and the overscroll fix |
| `3e783c4d` | "Update vv-hero.liquid" — **Part 4b steps 1–2**, the `isOuter` redefinition and the rewiring of `staticOffset` / `amp` |
| `4e391e27` | shader diet — **Part 4b step 3**, deleting the dead glass path |

Branch `consistent-grid`. Nothing is broken. **Parts 1–6 are all complete.**

**Part 4b was done last, after Part 5**, reversing the original order — see the
note at the top of this file for why.

### Done — Part 1, column ruler — *deleted in Part 6*

**This no longer exists.** Kept as a record of what it did and why, because the
technique is worth reusing. `?grid` does nothing now.

`snippets/vv-grid-ruler.liquid`, rendered from `layout/theme.liquid` just before
`</body>`. Appending `?grid` to any URL showed it. Four full-bleed stripes at
`position: fixed; inset: 0` (**not** `100vw` — that includes the scrollbar and
invents a ~15px offset), stepping 4 → 3 → 2 at 990 and 750.

The `inset: 0` detail is the reusable part: `100vw` includes the scrollbar, so
any full-bleed overlay measured that way sits ~15px off from the content it is
supposed to be checking — which makes the ruler lie in exactly the situation you
built it for.

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
`.vv-hero__panel--glass` class rule is gone; the markup kept emitting the class
until Stage C removed it.

There is **no base `grid-template-columns`** — the three ranges are exhaustive,
so a base value would be dead. The mobile height override moved inside the 749
block, so the hero and the product grid now step on the same pixel.

**Glass is not symmetric at mobile.** The original rule was "first and last
visible column" at every breakpoint. At 2 columns that is *every* column, and
`backdrop-filter` applied everywhere reads as nothing at all — there is no
unfiltered image left to compare against. So the mobile block glasses only
`:nth-child({{ cols_mobile }})`, the right-hand column. Tablet and desktop keep
the first-and-last pair. This has a knock-on for Part 4b — see there.

### Done — Part 4a Stage C, blocks out

Committed as `d1cbeb80`. Three files' worth of change landing together:

- **Markup.** The panel loop reads `{%- for i in (1..cols_max) -%}` — a Liquid
  range literal, `i` discarded, only the count matters — instead of iterating
  `section.blocks`. `shopify_attributes` and the `--glass` class emission are
  gone with it. No `section.blocks` or `block.*` reference survives anywhere in
  the file.
- **Attributes.** `data-cols` now reads `cols_desktop` (the last stale `{{ cols }}`
  is gone), joined by `data-cols-tablet` and `data-cols-mobile`. `data-glass` is
  deleted.
- **Schema.** The `panel` block type is gone and the preset is `[{ "name": "Hero" }]`.
- **`templates/index.json`.** `vv_hero_MdexzK` lost its four `panel_*` blocks and
  `block_order`, and gained `cols_desktop: 4` / `cols_tablet: 3` /
  `cols_mobile: 2`. The template is now the record of the live column counts
  rather than the schema defaults — which is what Part 6 asks for.

**The open question is settled.** The panel count was confirmed at **four** via
`document.querySelectorAll('.vv-hero__panel').length`, matching the local
`index.json`. The theme editor had *not* saved over the template; no pull was
needed, and the migration deleted the right set.

**Known regression, expected, do not chase:** no static edge refraction and a
weaker cursor lens. The shader still reads the now-deleted `data-glass`, so
`glassFlags` fills with zeros and `staticOffset` goes to zero. Part 4b is where
that comes back.

**Two loose ends left in `d1cbeb80`:**

1. **Line 593 is `"blocks": [],`** — the key was emptied rather than deleted.
   Valid JSON, valid schema, renders fine, but it declares that the section takes
   blocks and then names no types. Delete the key; mind the `],` closing
   `"settings"` above it.
2. **Indentation inside `<style>` is misleading.** The desktop block reads as
   though it is nested inside the tablet block. It is not — all three media
   queries are siblings at depth 0 and the braces balance. Prettier will not fix
   this; see "Things learned the hard way".

### Done — Part 5, `u_cols` follows the breakpoint

Committed as `8df0beb5`.

`u_cols` was uploaded once at init from `data-cols`, and **that attribute is
rendered by Liquid on the server** — it cannot know the viewport, so it always
reported `cols_desktop`. The shader divided the hero into quarters at every
width: below 990 the refraction seams sat where no divider was, and the mouse
lens was confined to the wrong regions. This is the clean statement of the
whole part: CSS responds to the viewport, Liquid cannot. The three
`data-cols-*` attributes exist so the server can hand the browser every value
and let client-side code choose.

- `colsForViewport()` (line 326) picks from the three attributes via two hoisted
  `MediaQueryList` objects. Widest test first — 1200px matches both `min-width`
  queries, so early return is what makes them exclusive.
- `applyCols()` (line 336) uploads it, clamped to `MAX_COLS`.
- Called once at init (line 344), then from a `change` listener on each query
  (lines 347–348). `matchMedia` rather than the existing `resize` handler:
  `resize` fires continuously through a drag, these fire only when a boundary is
  actually crossed. No redraw needed — the rAF loop draws every frame, so a
  changed uniform lands on the next one.
- `data-cols` renamed `data-cols-desktop`; the `cols` variable it fed is gone,
  dead once `applyCols` took over the init upload.
- The empty `"blocks": []` left over from Stage C is deleted.


### Done — Part 4b, shader diet

Two commits: `3e783c4d` (the redefinition and the rewiring) and `4e391e27` (the
deletion). Note that `4e391e27`'s message describes all three steps, but only
the deletion is actually in it — the other two had already landed in
`3e783c4d`.

**`isOuter` redefined** so it means what the CSS means — the last column always,
the first column only when there are at least three:

```glsl
float isFirst = (1.0 - step(0.5, colIndex)) * step(2.5, u_cols);
float isLast  = step(u_cols - 1.5, colIndex);
float isOuter = max(isFirst, isLast);
```

`step(2.5, u_cols)` is the gate — "are there at least 3 columns?" — and
multiplying by it collapses `isFirst` to zero at 2 columns, leaving the
right-hand column alone, which is exactly the mobile glass rule. At 3 and 4 it
is 1.0 and the result is identical to the old single-line expression. The
midpoint thresholds (`2.5`, `u_cols - 1.5`) rather than integers are the file's
existing idiom for sidestepping float equality. No branching: GPUs run fragments
in lockstep, so `step` plus arithmetic beats an `if`.

**Why this inverted the plan.** The original entry treated the mobile mismatch
as damage to tolerate, and offered "accept it and note it as intentional" as an
option. That was backwards. Redefining `isOuter` to match the CSS makes the two
identical at *every* breakpoint — which restores 4b's premise rather than
complicating it. Glass and `isOuter` are once again the same value computed
twice, so deleting the `u_glass` path became a genuine simplification instead of
a loss of expressiveness. The mobile exception stopped being a special case and
became part of the definition of "outer".

It only became visible after Part 5. With `u_cols` pinned at 4 the degeneracy
could not occur; once the shader followed the breakpoint, mobile showed
distortion across both columns against glass on one. That is the argument for
having reordered the two parts.

**Then the deletion.** `staticOffset` and `amp` now read `isOuter`, which
restored the static edge refraction and the stronger cursor lens — both absent
since Stage C removed `data-glass` and left the uniform feeding zeros. With
nothing left reading it, the whole per-panel array path went: `glassFlags`
parsing, the `u_glass[]` declaration, `glassAt()` and its per-fragment loop, the
`float glass` local, the `uGlass` lookup, and the `uniform1fv` upload. Fourteen
lines, including a loop that ran for every fragment.

**Kept, and easy to delete by mistake:** `u_glassStrength` / `glassStrength` /
`data-glass-strength` / the `glass_strength` setting are all live — they drive
`staticOffset`, and the slider still works. The names sit one character apart
from the dead ones; `u_glassStrength` is declared on the line directly above
where `u_glass` was. `MAX_COLS` also stays, down to a single use clamping in
`applyCols`.

**Note for the next reader:** `(1.0 + isOuter) * isOuter` is kept in `amp`, but
`isOuter` is strictly 0 or 1 — `isFirst` and `isLast` are `step` results — so
that expression can only ever be `0.0` or `2.0`, making it exactly
`2.0 * isOuter`. The longer form is preserved for continuity with the original
line. Do not read it as implying `isOuter` might be fractional.

### Current line numbers in `sections/vv-hero.liquid`

As of `4e391e27`, with the grid work complete.

| Line | What |
| --- | --- |
| 17 | `.vv-hero` rule — also holds `--vv-glass` / `--vv-glass-bg` (22–23) |
| 51 | `.vv-hero__panels` — no base `grid-template-columns`, by design |
| 65 | `.vv-hero__panel + .vv-hero__panel` — 1px divider |
| 70 / 94 / 115 | the three media blocks — mobile / tablet / desktop |
| 134 | `</style>` |
| 140–142 | `data-cols-desktop`, `data-cols-tablet`, `data-cols-mobile` |
| 165 | panel loop — `{%- for i in (1..cols_max) -%}` |
| 190 | `MAX_COLS = 8` — one use left, the clamp on 319 |
| 236–238 | `isFirst` / `isLast` / `isOuter` |
| 243 / 250 | `staticOffset` / `amp` — both read `isOuter` |
| 306–307 | the two hoisted `MediaQueryList` objects |
| 308 / 318 | `colsForViewport()` / `applyCols()` |
| 328–329 | the two `change` listeners |
| 502 | `"Columns"` schema header + the three `cols_*` ranges |

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
- **Prettier does not format CSS that contains Liquid.** Running it on
  `sections/vv-hero.liquid` rewrote 343 lines of JS and markup but left the
  `<style>` block's indentation exactly as typed — the plugin won't parse a
  stylesheet with `{{ }}` in it, so the whole block passes through untouched.
  The practical cost: misleading indentation inside `<style>` never gets
  corrected for you, in the one file where "check the brace depth" is already a
  rule. Verify nesting by counting braces, not by reading the indentation.
  Correctly nested and correctly indented are different properties here.
- **The half-done rename actually happened.** The warning above was written
  before it fired, and it fired anyway during Part 5: `data-cols` was renamed in
  the markup but `root.dataset.cols` was left in `colsForViewport`, so desktop
  silently fell to one column — no console error, nothing visibly broken until
  you looked at the right breakpoint. Knowing the trap is not the same as
  running the grep. Run the grep.
- **A `ReferenceError` inside the IIFE kills everything after it.** A typo'd
  variable name (`tabletMath` for `tabletMatch`) threw at init, so every
  statement below it — the texture load, the resize listener, the rAF loop —
  never ran, and the canvas never got `is-ready`. The tell: the effect vanished
  *entirely* rather than misbehaving. A shader that is genuinely wrong still
  draws, just wrongly. An effect that is completely absent is nearly always JS
  that died before reaching the draw call, and the console names the line.
  Reading an undeclared variable throws; it does not quietly give `undefined`.
- **Specificity beats source order, always.** `sections/vv-header.liquid` had
  `body { background-color: var(--vv-shell); }` which had never once applied:
  Dawn puts `class="gradient"` on the body and `base.css:2949` styles
  `.gradient`. A class (0,1,0) beats an element selector (0,0,1) no matter how
  much later your `<style>` block appears in the document. Being in an inline
  `<style>` at the bottom of the page buys nothing against a more specific
  selector. This is the mirror image of the misspelled-selector lesson above,
  and the same devtools check separates them: a rule that loses on specificity
  still appears in the Styles pane, struck through.
- **`SyntaxError` and `ReferenceError` fail at different times, and the
  difference is diagnostic.** A missing comma between two strings in the
  `fragSrc` array is a `SyntaxError` — JS has no implicit string concatenation,
  so the *entire* `<script>` block is rejected at parse time and not one
  statement runs. A `ReferenceError` is a run-time throw: everything above it
  executes, everything below is skipped. Both present identically — the effect
  is simply absent — and the console distinguishes them instantly. **Effect
  completely gone → open the console before reading any code.** Both of these
  cost a round in this branch.

---

## Done — Part 6, housekeeping and the two design questions

Commits `3cce708b` (removals) and `8ae850ac` (the design answers).

**Ruler gone.** `snippets/vv-grid-ruler.liquid` and its `{% render %}` in
`layout/theme.liquid` are deleted. `?grid` no longer does anything, so any
instruction elsewhere in these notes to "load with `?grid`" is historical.

**Stubs gone.** `sections/vv-product-row.liquid` and
`sections/vv-editorial-band.liquid` were 0 bytes and referenced by nothing —
no template, no section group. Deleted rather than built; git has them.

**The gutter is global now.** `spacing_grid_horizontal` and
`spacing_grid_vertical` go from 8 to 0 in `config/settings_data.json`, so
`layout/theme.liquid:205–208` emits `0px` for all four `--grid-*-spacing`
properties and **every product grid in the theme is full-bleed** — collection
and search pages included. The four scoped overrides in `featured-collection`
are redundant and removed; the tablet `calc()` still reads
`--grid-desktop-horizontal-spacing` and now resolves it from `theme.liquid`.
Dawn writes the `px` suffix itself, so the unitless-zero `calc()` trap does not
apply here.

**The heading aligns to the grid.** `page-width` is off `.collection__title`.
It had contained the heading to 1200 while the grid below was full-bleed, so it
read as a separate row rather than part of the grid. It now starts on the
grid's outer edge, flush with the first column's content — which has no
horizontal padding either, so the two agree.

### Still open after Part 6

- **The hairlines are still scoped to `featured-collection`.** Collection and
  search pages now have a zero gutter with *no* rules between cells, which is
  the one place the theme does not yet look like the reference. Making the grid
  read as global end-to-end means lifting the `box-shadow` rules and
  `--vv-rule-color` out of this section into somewhere shared. That is a real
  change, not a move: the `:not(:nth-child(Nn + 1))` selectors are written
  against this section's 4/3/2, and collection pages have their own column
  settings.
- **The breakpoints are still unrecorded.** 990 and 750 live only in this doc
  and in two `<style>` blocks. The column counts are in `templates/index.json`
  now, but changing one section's media queries without the other still
  desynchronises them silently.

---

## Next: Part 7 — featured collection as a full-width scroller

Three changes, decided 2026-08-12, moving `featured-collection` closer to the
reference at <https://www.sotf.com/en>.

> **The reference was not inspected while writing this.** `sotf.com` returns
> HTTP 403 to automated fetches, so everything below is built from the stated
> requirements and from what the code can support. Every visual judgement —
> how much padding, whether the arrows stay, how the row ends — needs a human
> eye on the reference. Do not treat the numbers here as measured.

### 7a — remove the section heading

Supersedes the Part 6 decision. Part 6 aligned the heading to the grid's outer
edge by dropping `page-width`; the heading is now going away entirely, so that
change is moot rather than wrong.

Two levels, and the difference matters:

- Blanking `title` in `templates/index.json` hides the `<h2>` — line 123 wraps
  it in `{%- if section.settings.title != blank -%}`.
- **But the wrapper `<div class="collection__title title-wrapper …">` on line
  122 renders unconditionally**, outside that `if`. An empty wrapper still
  carries `title-wrapper` margins, so the gap above the row stays. Removing the
  heading properly means removing the wrapper, not just emptying the setting.

Check before deleting the wrapper: it also carries
`collection__title--desktop-slider` when a desktop slider is on. The slider
*buttons* are safely elsewhere — `.slider-buttons` is inside
`<slider-component>` at line 215, not in the title wrapper — but confirm what
that modifier class does in `component-slider.css` before assuming the wrapper
is inert.

### 7b — full-width rows, scrollable left to right

**Dawn already ships this; it is turned off.** In `templates/index.json`,
`featured_collection` has `enable_desktop_slider: false` and
`swipe_on_mobile: false`. Turning them on gives `.slider--desktop` —
`overflow-x: auto`, `scroll-snap-type: x mandatory`, `scroll-behavior: smooth`
(`component-slider.css:140`). The gate at line 103 needs
`products_to_display > columns_desktop`; `products_to_show` is 8 against 4
columns, so it passes.

Two things that were tuned for a gutter and now resolve to zero, which is
probably what you want but should be looked at:

- `.slider--desktop:after` (line 152) adds a 5rem trailing pad offset by
  `calc(-1 * var(--grid-desktop-horizontal-spacing))` — now `0px`.
- `.slider--desktop .slider__slide:first-child` uses
  `--desktop-margin-left-first-item` for the leading inset.

**The hairlines will be wrong, and this is the real work in 7b.** The
`box-shadow` rules use `:not(:nth-child(4n + 1))` — written for a *wrapping*
grid, where every 4th item starts a new row and must not carry a left rule. A
horizontal scroller is **one row of N items**, so `4n + 1` would strip the rule
from items 5, 9, 13… in the middle of a continuous row. In a scroller the rule
is wanted on every item except the first, which is `:not(:first-child)` — and
the three per-breakpoint blocks collapse into one, since column count no longer
determines where rows break. Decide whether the scroller replaces the wrapping
grid at all breakpoints or only some; the answer determines whether those
`nth-child` rules survive at all.

### 7c — left padding on the item names

**Do not put the padding on `.grid__item`.** That is the trap already recorded
above: Dawn's items are content-box, so horizontal padding adds to the
percentage width and wraps the row. `padding: 2rem 0` is vertical-only for
exactly this reason.

The padding belongs on an inner element, where it costs the layout nothing —
`.card__information` or `.card__heading` in `snippets/card-product.liquid`
(lines 112 and 151). Scope it to this section via the existing
`#collection-{{ section.id }}` prefix unless the intent is theme-wide.

Worth deciding at the same time: the hairline is drawn by `box-shadow` on the
*item*, so padding an inner element moves the text away from the rule without
moving the rule. If the image should stay flush to the hairline while only the
text indents, target the text container alone. If everything should inset, the
card wrapper is the place.

### Checkpoint 7

- No heading, and no empty gap where it was
- One row, spanning the full viewport, scrolling horizontally without a
  horizontal scrollbar on `<body>`
- A hairline before every item except the first, continuous across the scroll
- Product names clear of the hairlines by a deliberate amount
- The hero above still lines up with whatever columns remain visible

---

## Reference — settings that carry the grid

- ~~Record the four numbers (4/3/2) durably.~~ **Done in Stage C.** Both
  sections now carry their counts in `templates/index.json`:
  `featured_collection` as `columns_desktop` / `columns_tablet` /
  `columns_mobile` (4 / 3 / `"2"` — the string is Dawn's own type, not a
  mistake), and `vv_hero_MdexzK` as `cols_desktop` / `cols_tablet` /
  `cols_mobile` (4 / 3 / 2). They agree by record now, not by coincidence.
  What is **not** recorded anywhere but this doc and two `<style>` blocks: the
  breakpoints themselves, 990 and 750. Changing one section's media queries
  without the other still silently desynchronises them.
- **Gutter**, since Part 6: `spacing_grid_horizontal` and
  `spacing_grid_vertical` are both `0` in `config/settings_data.json`, under
  `presets.Dawn` — `current` is the string `"Dawn"`, so the preset *is* the live
  value. Note that the theme editor rewrites `current` into an object the first
  time a setting is changed there; after that, edit `current`, not the preset.
- **`config/settings_schema.json` had to change too.** Dawn ships both of those
  ranges with `"min": 4`, so a saved value of `0` fails validation outright:
  *"spacing_grid_horizontal can't be less than 4"*. The floor is lowered to `0`
  on both. `step` stays `4`, so the legal values are now 0, 4, 8, … and the
  slider still reaches zero cleanly. This is an edit to a stock Dawn config
  file — expect a conflict here if the theme is ever updated from upstream.
  The alternative was to leave the setting at `4` and zero the properties in
  CSS instead, which was rejected: the theme-editor slider would then display a
  gutter the storefront does not have.

## Watch out for

- **The shader's columns are viewport-relative.** `colWidth = 1.0 / u_cols` over
  `uv.x` spanning the full canvas. Correct only because the hero stays
  full-bleed. If the panels ever move onto a contained grid, the shader needs
  the content inset and gutter as uniforms.
- **Zeroing the gutter globally** would touch every grid on the site.
