# Consistent grid — state and next steps

Handoff notes. **Parts 1 through 6, and all of Part 7, are done.** The hero
panels line up with the product columns at every breakpoint, driven by three
settings; glass and distortion share one definition of "outer"; the gutter is
zeroed theme-wide. The hairlines are one page-wide overlay in
`snippets/vv-grid-lines.liquid`, which also owns the line colour for the whole
theme. Glass blur, tint and the horizontal text inset live in
`snippets/vv-tokens.liquid`.

**Part 7 was built in the order 7c → 7b → 7d → 7a**, decided 2026-08-14, with
7a finished 2026-08-28. `featured-collection` is now a horizontal scroller
whose columns rest on the hairlines at desktop, and the section title, both
slider arrows and the "View all" action all sit *inside* the row rather than
above and below it.

**Next is tablet and mobile snapping** — the one part of the grid that has
never worked. Below 990px the columns do not line up with the overlay at all,
and not by accident: three Dawn rules describe a different design. See "Next —
tablet and mobile snapping".

**Two corrections to earlier notes in this file:**

- The warning that the two mask declarations in `featured-collection.liquid`
  disagree — `mask-image` at 60% against `-webkit-mask-image` at 50% — was
  **already false when it was written**. Both read `60%`, and `git log -p`
  shows they were committed that way in `5438ec41`. Nothing was fixed because
  nothing was broken.
- 7a was planned as a heading row *above* the products, mirroring the
  reference's `home_collection_info`. It was not built that way, and the plan
  text further down still describes the version that was not built. See "Done
  — Part 7a".

**Knowingly incomplete:** the footer and header are still opaque so the lines
stop at both ends of the page, and tablet/mobile columns do not line up with
the overlay. See "Still open after Part 7a".

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
| `a552663a` | hairlines become a page-wide overlay — **Part 7c** |
| `10683765` | record the Part 7c commit hash |
| `79eb7f8f` | featured collection becomes a scroller — **Part 7b**, desktop only |
| `e3501c0d` | scroll settling lands on whole pixels — **Part 7b fix** |
| `5438ec41` | glass tokens move to `vv-tokens.liquid`; product text moves onto the image — **token layer + Part 7d** |
| `483cb9da` | record the token layer and Part 7d commit hash |
| *uncommitted* | title, arrows and "View all" move into the row — **Part 7a**. Record the hash here once committed |

Branch `consistent-grid`. Nothing is broken. **Parts 1–6 and all of Part 7 are
complete.**

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
- **`scrollLeft` is quantized to whole pixels.** Writing `944.5` reads back
  `945`. Any easing loop that steps by a *fraction* of the remaining distance
  therefore stalls once that fraction drops below 1px: the write becomes a
  no-op, the position never changes, and a `< 0.5` exit condition is never met,
  so the `requestAnimationFrame` loop spins forever on a scroll that cannot
  move. It fails silently — nothing throws, the row just rests a few pixels
  short. The stall threshold falls out of the easing constant: progress stops
  below `0.5 / EASING`, which at `EASING = 0.14` is ~3.57px — and the two bad
  readings were off by 3 and 3.5. **Retuning `EASING` moves that boundary.**
  Two rules follow: snap targets must be whole pixels, and the per-frame step
  must be floored at one whole pixel, sign preserved.
- **`--vv-gutter` is already taken** by `sections/vv-header.liquid:9`, where it
  means "header shell inset". Pick different names. In use so far:
  `--vv-rule-color` (featured-collection), `--vv-glass` / `--vv-glass-bg`
  (hero), `--vv-hairline` (`:root`, declared in `snippets/vv-grid-lines.liquid`
  — the only theme-wide one), `--vv-shell` (vv-header).
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
- **A stray space kills a function call, and the symptom depends on where it
  hides.** `repeat (2, 1fr)` — one space before the paren — is not a function,
  so the value is invalid and *the whole declaration is dropped*. `display: grid`
  still applied, leaving a grid with no declared column track: one implicit
  column, and every child became a row. Same root cause as the
  `brightness (1.05)` entry below, but that one hid inside a custom property and
  failed a layer later at substitution, while this one failed immediately.
  The tell is the same in both cases: an effect that is *structurally* wrong
  rather than absent.
- **An unclosed rule breaks its neighbours, and that is the diagnosis.** A
  missing `}` on `.vv-hero__panel + .vv-hero__panel` didn't just lose the
  divider — CSS error recovery consumes tokens until it finds a matching brace,
  so the `@media (max-width: 749px)` block that followed was swallowed into the
  rule and discarded, taking the hero's `grid-template-columns` with it.
  **A value error breaks one thing; a structural error breaks the things
  around it.** If a change to one declaration also disturbs its neighbours,
  stop reading values and count braces. Third brace-depth failure on this
  branch, and each one a different shape: rules nested too deep (Part 4a),
  rules escaped *out* of a media query (7c, the `display: block` pair), and now
  a block left hanging open.
- **`.color-scheme-N` paints a background, not just variables.**
  `layout/theme.liquid:121` renders `body, .color-scheme-1, .color-scheme-2, …`
  from a Liquid-built selector list and sets `background-color` on all of them.
  So a section can have *two* independent opaque backgrounds — one from
  `.gradient`, one from its colour-scheme class — and removing the obvious one
  changes nothing. Worse, the selector doesn't exist until Liquid renders it,
  so grepping the CSS for `.color-scheme-1` finds nothing. **If an element is
  still painted after you removed its background, look for a second rule before
  doubting the first edit.**
- **An opaque background is a contract with everything drawn on top of it.**
  Removing one is never only a background change. Here it revealed a near-black
  page shell, and dark text and dark hairlines went invisible in the same
  instant — two unrelated-looking symptoms from one edit. Before deleting a
  background, ask what colour the thing *behind* it is, and what was chosen
  against the old one.
- **`elementsFromPoint` cannot see `pointer-events: none` elements**, because
  it does hit testing, not paint inspection. An overlay built to be
  click-through will never appear in its results no matter how correct it is.
  It also answers only for points inside the viewport and returns `[]` for
  anything below the fold — `getBoundingClientRect()` is viewport-relative, so
  a section further down the page yields a `y` that is simply off-screen. Both
  of these produced confidently empty output that meant nothing.
- **Custom properties are inherited, so where you declare one decides who can
  read it.** A variable on `.vv-grid-lines` is visible only to its own
  descendants. Sharing a value across sections means declaring it on `:root`.
  And unlike every specificity lesson on this branch, *position in the document
  does not matter* — a `:root` rule in a `<style>` rendered near `</body>`
  still reaches a section at the top of the page, because custom properties
  resolve at computed-style time rather than by source order.
- **Delete a schema setting and the template keeps the key.** Removing
  `divider_opacity` from `vv-hero`'s schema left `"divider_opacity": 60` in
  `templates/index.json`. Shopify ignores keys with no matching setting, so
  nothing errors — the dead value just sits there looking live. Same half-done
  migration as Part 4a Stage C, and the same fix: change the schema and the
  template together, or not at all.
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
- **Two dashes make it a custom property, and a custom property is inert.**
  `--scroll-snap-type: x proximity` is perfectly valid CSS that does absolutely
  nothing: it declares a variable nobody reads. The real property has no dashes.
  This came straight after three legitimate `--` shadowing rules in the same
  style block — the pattern was copied one rule too far. The distinction: you
  shadow a variable when the behaviour you want is already *parameterised* by
  one; `scroll-snap-type` is the property doing the work, so you set it. This is
  the third variant of "declaration fine, effect zero" on this branch, and the
  only one with no substitution to inspect — there is no consumer anywhere.
- **`--color-foreground` holds a bare RGB triplet, not a colour.** Every
  consumer wraps it as `rgba(var(--color-foreground), 0.75)`, so `18, 18, 18` is
  the whole value. Setting it to `rgba(255, 255, 255, 100%)` produces
  `rgba(rgba(255,255,255,100%), 0.75)` — nonsense, declaration dropped, and the
  variable reads back perfectly in the console. **When a `var()` does nothing,
  paste its value into the declaration that reads it and read the result as
  CSS.** The error is at the join, not at either end. Splitting colour from
  alpha is also *why* one override fixed four states at once: resting 0.75,
  hover 1.0, disabled 0.3, and the counter inheriting.
- **C# habits are SyntaxErrors, and a SyntaxError kills the whole `<script>`.**
  `bool atRight = …` and `max - 1px` both got typed. JS has no type keywords and
  no units — `scrollLeft` and friends are plain numbers counting CSS pixels.
  Neither is a subtle bug: nothing in the block runs, including the parts that
  worked a minute earlier. Console first.
- **Assigning to a DOM property that does not exist is silent.**
  `track.scrollRight += delta` invents an expando on the element and stores a
  number in it. No error, no warning, no effect. There is no `scrollRight`;
  horizontal position is `scrollLeft` and nothing else.
- **`event.deltaY *= 33` appears to work and is still wrong.** `deltaY` is
  read-only on `WheelEvent`, so outside strict mode the write is discarded while
  the expression still evaluates to the product. Right answer, wrong mechanism,
  and it throws the moment the code lands anywhere strict.
- **`offsetLeft` rounds to whole pixels; `getBoundingClientRect()` does not.**
  The column pitch here is a quarter of the track — `423.75` at 1695px — so
  measuring it with `offsetLeft` drifts by a couple of pixels across the row.
  For anything being compared against a fixed overlay, measure with rects.
- **`wheel` listeners are passive by default on `window` but not on an
  element.** Moving the listener from the `<ul>` to `window` silently turns
  `preventDefault()` into a no-op unless `{ passive: false }` is passed. The
  same code, the same call, a different target, and the feature quietly stops
  working. Also: the boolean third argument means `capture`, never `passive`.
- **`innerWidth` includes the scrollbar; the layout does not.** A diagnostic
  comparing item widths against `innerWidth / 4` was off by ~4px for a whole
  round. `document.documentElement.clientWidth` is the number the layout
  actually divides. Exactly the `100vw` trap from the Part 1 ruler, reproduced
  in the tool built to check for it.
- **Custom properties are not functions.** A `var()` inside a custom property is
  substituted where that property is **declared**, and what inherits down is the
  resolved string. So this does *not* give the card a 12px blur:

  ```css
  :root          { --blur: 4px; --glass: blur(var(--blur)) saturate(1.15); }
  .card__content { --blur: 12px; }
  ```

  Overriding an input below the declaration is too late. The fix is to keep
  **inputs** in the shared layer and do the **composition in a real property, on
  the element being matched** — which is the entire reason
  `snippets/vv-tokens.liquid` exists.
- **Custom properties accept almost anything, and defer the consequences.** A
  custom property's value is a token stream; validity is only checked when it is
  substituted into a real property. So `--vv-glass-blur: 4` (no unit),
  `--vv-glass-tint: 30 | divided_by: 1000.0` (a Liquid filter written as raw CSS
  text) and a `var()` naming a property that does not exist are all *well-formed
  declarations*. They parse, they show in devtools, and they fail one layer
  later — in a different file from the one just edited. An undefined `var()`
  makes the property "invalid at computed-value time", which computes to `unset`
  and therefore, for a non-inherited property like `background`, to `initial`.
- **CSS almost never tells you that you missed.** Five silent failures in one
  session, all the same shape — something well-formed that quietly does the
  wrong thing: an invalid unit in a custom property; a `var()` naming nothing; a
  selector matching no element (`.card_information` for `.card__information`);
  and colour stops in descending order, twice. None errored. **"No error" tells
  you nothing.** The only reliable checks are the Computed tab and the technique
  below.
- **When a subtle effect does not appear, replace it with a grotesque one.** A
  3% white frost that is invisible and a frost that is not rendering at all look
  identical, and no amount of tuning distinguishes them. Swap the colour for
  fully opaque red and reload: either a red band appears — the CSS is fine and
  this is a visibility problem — or it does not, and the declaration is not
  reaching the element. One reload, two completely different fixes. Make sure
  the probe does not itself depend on the value under test: probing with
  `rgba(255, 0, 0, var(--vv-glass-tint))` paints 3% red and proves nothing.
- **Colour stop positions must ascend, and CSS silently clamps them if they do
  not.** A stop positioned before its predecessor is dragged up to the
  predecessor's position. So `linear-gradient(to right, #000 100%, transparent)`
  puts both stops at 100%, the transition occupies zero width, and the result is
  a flat fill with no fade at all. Same for `(tint 100%, transparent 0%)`. The
  position is not a property of the colour: **the stop list is read in order
  along the axis**, so reversing the colours means leaving the positions where
  they are. With two stops, omitting both positions gives 0% and 100%, which is
  usually what is wanted.
- **A prefixed and unprefixed pair are one declaration for two browsers, not two
  settings.** `mask-image` at 60% and `-webkit-mask-image` at 50% is a real
  cross-browser bug that is invisible locally — Chrome and Firefox read the
  first, Safari the second. Same discipline applies to `backdrop-filter`. And
  the converse: a browser that does not recognise the prefixed alias flags it as
  an improper value in devtools, which is expected and not worth chasing.
- **`rgba()` alpha is 0–1, or a percentage *with* the sign.** A bare `100` is out
  of range and clamps to `1`. It happens to give full opacity, so it looks
  correct — but `50` also gives `1`, with no error.
- **A mask reads only the alpha channel; colour is ignored.** So
  `mask-image: linear-gradient(to right, #000 60%, transparent)` is fine, and
  `#000` is convention rather than meaning. This is the exact opposite of a
  *background* gradient, where fading to the `transparent` keyword is a trap
  because it means transparent **black** — `rgba(0, 0, 0, 0)` — and can drag a
  grey haze through the middle of a white fade. Fade backgrounds to the same
  colour at zero alpha; fade masks to whatever.
- **A positioned pseudo-element paints above non-positioned in-flow siblings.**
  So a `::before` carrying a background covers the element's own text. The fix
  is `position: relative` on the children, putting them in the same painting
  group where DOM order decides — and `::before` is by definition first. The
  tempting `z-index: -1` on the pseudo fails here: `.card__content` is positioned
  but has `z-index: auto`, so it creates **no stacking context**, and the
  negative index escapes the subtree entirely and slides behind the product
  image. Also: `content: ''` is mandatory, or the pseudo-element is never
  generated at all.
- **Liquid and CSS are two languages in two runtimes, bridged one way only.**
  Liquid runs on the server and emits text; CSS custom properties exist in the
  browser afterwards. Nothing wrapped in `{{ }}` is not Liquid — it is
  characters. And a snippet is **not a module**: `vv-tokens.glass-alpha` cannot
  work in either language, because after rendering the browser receives one
  document of `<style>` blocks with no memory of which file each came from. The
  property name is the entire interface. Corollary: `section.settings.*` is
  empty in a snippet rendered from `layout/theme.liquid`, which is why the token
  layer holds literals and the conversion from editor units lives in whichever
  section owns the slider.
- **Dawn uses three near-identical class names one level apart.**
  `.card__information` (BEM element, holds title and price), `.card-information`
  (a different wrapper *inside* it, `card-product.liquid:165`), and
  `.card__content` — which appears **twice** per card, once inside `.card__inner`
  for the badges and once as a direct child of `.card` for the text. A selector
  matching nothing is not an error, so getting one of these wrong costs a
  debugging round with no feedback at all.

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

- ~~**The hairlines are still scoped to `featured-collection`.**~~ **Fixed in
  Part 7c** — they are one page-wide overlay now, global by construction.
- **The breakpoints are still unrecorded.** 990 and 750 live only in this doc
  and in two `<style>` blocks. The column counts are in `templates/index.json`
  now, but changing one section's media queries without the other still
  desynchronises them silently.

---

## Done — Part 7c, hairlines become a page-wide overlay

Built 2026-08-14, **out of the planned order** — 7c before 7b, because the
per-item rules it deletes are exactly the ones that break in 7b's scroller.

### What exists now

**`snippets/vv-grid-lines.liquid`** — a permanent version of the deleted ruler.
`position: fixed; inset: 0; z-index: -1; pointer-events: none`, a 2/3/4 column
grid at the same 750/990 breakpoints, four spans each with a `border-left`,
the first suppressed and the surplus hidden by `:nth-child(n + 3)`. Rendered
from `layout/theme.liquid:329`, **after** the skip-to-content link so that link
stays the first element in the document for keyboard users.

**It owns `--vv-hairline` for the whole theme**, declared on `:root` inside the
snippet. Three things read it: the overlay's own spans,
`featured-collection.liquid:54` (`--vv-rule-color`, the horizontal rules), and
`vv-hero.liquid:66` (the panel dividers). One value, three drawers.

Declaring a document-level variable inside a body-rendered snippet looks wrong
and isn't: custom properties resolve at computed-style time, not by source
order, so a `:root` rule in a `<style>` near `</body>` still reaches a section
earlier in the document. This is the opposite of how specificity and source
order have behaved everywhere else in this branch.

**`featured-collection.liquid`** lost all three `:not(:nth-child(Nn + 1))`
media blocks — every vertical hairline and all the column arithmetic. What
survives is one rule, `box-shadow: 0 -1px 0 var(--vv-rule-color)`, drawing the
*horizontal* row rules. An overlay can only ever draw verticals; it has no idea
where a row starts. `padding: 2rem 0` went too, closing the black bands between
rows.

**`vv-hero`'s `divider_opacity` setting is deleted** from the schema, since the
border now reads the shared variable. The stale key in `templates/index.json`
is still there — see below.

### The two things that made this hard

**The section had two opaque backgrounds, not one.** Removing `gradient` from
the wrapper was not enough: `layout/theme.liquid:121` renders
`body, .color-scheme-1, .color-scheme-2, … { background-color: rgb(var(--color-background)); }`
from a Liquid-built selector list, so `.color-scheme-1` paints a background all
by itself, through a rule that has nothing to do with `.gradient`. Grepping the
CSS files for `.color-scheme-1` finds nothing — the selector only exists after
Liquid renders it.

Fixed with a `vv-no-bg` class on the wrapper and
`div.vv-no-bg { background-color: transparent; }` in the section's `{%- style -%}`.
The selector is deliberately `div.vv-no-bg` at `(0,1,1)` rather than
`.vv-no-bg` at `(0,1,0)`: a bare class ties with `.color-scheme-1` and would
have been decided by source order, which is true today and fragile forever.

**The page shell is near-black.** `vv-header.liquid:43` sets
`body.gradient { background: var(--vv-shell); }` with `#0a0a0c` from
`header-group.json:52`. The featured collection had been a white island on it,
and the only thing making it white was the background just removed. So the
moment it went transparent, both the dark text (`#121212`) and the dark
hairlines (`rgba(18, 18, 18, 0.15)`) became invisible at once — two symptoms,
one cause. Hairlines went white to match the hero and the reference; the text
is left for 7d, which moves it anyway.

### Why the overlay stays *behind* the page

It was tempting to float it on top and have a single system draw everything,
which would let `vv-hero` drop its own dividers entirely. Rejected, and the
reason is worth keeping: the product photographs are near-white, so a white
line at low alpha vanishes on top of them. The overlay only ever draws against
the near-black shell, which is the one background its colour is tuned for. The
hero draws its own because it needs a line that survives photography.

So the lines are visible in every band of shell — padding, text areas, above
and below sections — and interrupted wherever an image sits. That is not a
limitation being worked around; it is what lets one white value work at all,
and it is why the reference indents its product *text* (7d) instead of trying
to draw across its images.

### Still open after Part 7c

- ~~**`templates/index.json:61` still has `"divider_opacity": 60`**~~ **Fixed.**
  The key is gone — a grep of the whole repo for `divider_opacity` finds
  nothing. Schema and template agree again.
- **The footer and header are opaque**, so the lines stop at both ends of the
  page. `footer.liquid:30` and the header both carry `color-… gradient`. The
  same `vv-no-bg` treatment would extend the lines; whether they *should* run
  edge to edge was not decided.
- **The horizontal rules may not survive Part 7.** They are the last thing in
  `featured-collection` still drawn per item. Once 7b makes the grid a single
  scrolling row there are no rows to separate, so decide their fate there
  rather than now.
- **`--vv-hairline` is currently `rgba(255, 255, 255, .8)`** — chosen quickly
  while debugging, not designed. Worth a deliberate look against the reference.

---

## Done — Part 7b, the product grid becomes a scroller

Built 2026-08-19, settling fixed and alignment verified 2026-08-20.
**Desktop only** — see "Verified — Part 7b alignment" for the readings, and
"Still open after Part 7b" for what tablet and mobile still don't do.

### Turning it on cost two booleans

`enable_desktop_slider` and `swipe_on_mobile` are both `true` in
`templates/index.json`. Nothing else was needed to get a scroller. The Liquid
gates at `featured-collection.liquid:78-91` only require
`products_to_display > columns` — 8 against 4 — and they pick the
`slider slider--desktop` classes on the `<ul>`; `component-slider.css` does the
rest. The settings are inert data, exactly like the `data-cols-*` attributes:
Liquid chooses classes, CSS responds to the viewport.

Everything after that was undoing Dawn's assumption that a slider lives inside
a 1200px container.

### Four scoped overrides, all the same move

In the section's `{%- style -%}` block. Each one shadows a custom property Dawn
already parameterised rather than rewriting the rules that read it — the same
technique as the `--grid-*-spacing` overrides in Parts 2–3.

| Rule | What it undoes |
| --- | --- |
| `slider-component { --desktop-margin-left-first-item: 0px }` | Dawn indents the first slide by `max(5rem, (100vw - page-width + 10rem - gutter) / 2)`, so a full-bleed slider *starts* where a contained one would. At 1920px that was 410px — and it narrowed every column too, because the widths are `calc((100% - that) / 4 - …)`. |
| `.slider { scroll-snap-type: x proximity }` | `mandatory` cannot be scrolled by less than half a column: it drags every small scroll straight back to where it started, so the row looked completely stuck. |
| `.slider-buttons { --color-foreground: 255, 255, 255 }` | The arrows and the `n / total` counter were `#121212` on the near-black shell — rendered, laid out, invisible. The same 7c consequence as the dark text and dark hairlines, noticed later because nobody looks for controls that were never visible. |
| `.slider--desktop::after { padding-left: 0 }` | The trailing twin of the first: a 5rem pad on the end of the track whose `margin-left: calc(-1 * var(--grid-desktop-horizontal-spacing))` clawback resolves to zero now the gutter does. It held the last four columns 50px left of the hairlines at maximum scroll. |

Measured at 1920px, before and after: first item at `410` → `0`; item width
`373.75` → `476.25`, which is exactly `documentElement.clientWidth / 4`.

**Why the `--color-foreground` override is scoped to `.slider-buttons` and not
the section.** The product titles and prices read the same variable and are
still dark on the shell. Widening that scope would fix them by accident and
pre-empt 7d, which moves and restyles them deliberately.

### The wheel handler

`sections/featured-collection.liquid:274-401`, an IIFE at the top level of the
file. Requested because Shift+wheel is the only native way to drive a horizontal
track and nobody discovers it.

What it does, in order, on every `wheel` event:

1. **Bails if `prefers-reduced-motion: reduce`.** Scroll hijacking is a motion
   effect; the arrows still work.
2. **Bails unless the row overlaps the middle half of the viewport.** The
   listener is on `window` — the request was that pointer position not matter —
   so without this gate a section below the fold would swallow page scrolling
   from the top of the document and you could never reach it.
3. **Normalises `deltaMode`.** Chrome reports pixels (`mode 0`, ~100 a notch),
   Firefox reports lines (`mode 1`, ~3 a notch). Unscaled, Firefox would scroll
   three pixels per notch and look broken rather than mis-tuned.
4. **Passes the event through, untouched, when the row has no room left in that
   direction** — no `preventDefault`, no scroll. This is what stops the section
   trapping the page. Both ends are tested separately, each paired with its own
   sign of delta; a single combined condition silently loses the left end.
5. **Accumulates into a `target`** and eases toward it at 14% of the remaining
   distance per `requestAnimationFrame`. Ticks queue against the target rather
   than the live `scrollLeft`, so fast wheeling is not dropped mid-animation.
6. **Suspends snapping for the duration of the gesture**, then 120ms after the
   last tick rounds the target to a whole number of columns and glides there.

**Point 6 is the part that matters.** It was also the part that was wrong —
see "Verified — Part 7b alignment" for what it took to make it land. Snapping
is done in JS rather than left to `scroll-snap-type: proximity` because
proximity only settles when it happens to land inside its own
implementation-defined threshold, which left the columns a few pixels off the
hairlines. The pitch is measured live from the first two items with
`getBoundingClientRect()`, so it follows the breakpoint without knowing the
column count.

**Two things that fight this loop and are handled, not removed:**

- `scroll-behavior: smooth` on the track is what makes Dawn's arrow buttons
  glide (`global.js:821` calls `scrollTo({ left })` with no `behavior`, which
  defers to the CSS). The rAF loop therefore passes `behavior: 'instant'` on
  every frame — opting one caller out instead of flattening the property and
  silently turning both arrows into hard jumps.
- `scroll-snap-type` is set to `none` inline during a gesture and restored by
  clearing the inline value, which lets the stylesheet apply again. Snapping is
  handed back only once no settle is still pending, or the stylesheet would
  animate against the settle.

**Tuning knobs**, all named constants at the top of the IIFE: `EASING` (0.14),
`LINE_HEIGHT` (33), `SETTLE_DELAY` (120).

### Verified — Part 7b alignment

Confirmed 2026-08-20, at a 1343px viewport: pitch 335.75, 8 items,
`scrollWidth` 2686, `maxScroll` 1343. Four readings, each exercising a
different code path:

| Reading | `scrollLeft` | What it proves |
| --- | --- | --- |
| At load | 0 | edges == hairlines exactly — the **layout** |
| Far right | 1343 (= `maxScroll`) | edges == hairlines exactly — the **clamp** |
| Mid-row, arrived from the left | 672 (= 671.5 rounded) | 0.5px uniform offset — the **rounding** |
| Mid-row, arrived from the right | 672 | identical — the **sign** |

Geometry held throughout: `pitch × count` == `scrollWidth` == 2686 and
`maxScroll / pitch` == 4, so the full-bleed overrides are sound — no phantom
`::after` pad, no leading inset.

**The 0.5px at mid-row is expected and is not a defect.** A column boundary at
`335.75 × 2 = 671.5` is not a position `scrollLeft` can hold, so 672 is the
closest reachable pixel. It is uniform across every item and invisible against
a 1px hairline. Do not reopen this on a future measurement.

**What was wrong, and the fix.** Both live in the wheel handler:

- `settle()` computed `Math.round(target / pitch) * pitch`, which is
  *fractional* — 944.5 at the width it was first tested. Unreachable by
  construction. It now rounds that result to a whole pixel.
- `step()` moved by `diff * EASING` each frame. Below ~3.5px that step rounds
  away to nothing and the write becomes a no-op, so `scrollLeft` never changes,
  `diff` never shrinks, `Math.abs(diff) < 0.5` is never satisfied, and the rAF
  loop re-queues forever. The step magnitude is now floored at one whole pixel,
  with the sign taken from the *signed* value — `Math.sign` of an `Math.abs`
  result is always 1, which does not stall but runs away in the wrong
  direction, a worse failure that only shows when scrolling left.

Because `target` is now an integer and `scrollLeft` always is, `diff` is a
whole number, so `Math.abs(diff) < 0.5` means exactly `diff === 0` — the loop
exits on arrival, and every frame strictly decreases the distance. Termination
is provable rather than hopeful. The two fixes are coupled: flooring the step
at 1px against a fractional target would oscillate 671 / 672 forever.

The stalled loop also explains a symptom noticed separately — an inline
`scroll-snap-type: none` still set on the track at rest. `restoreSnap()` is
only reached inside the exit branch, so snapping was never handed back. One
root cause, three symptoms.

### Still open after Part 7b

- ~~UNVERIFIED: do the columns rest on the hairlines?~~ **Verified 2026-08-20
  at desktop** — see "Verified — Part 7b alignment" above. Tablet and mobile
  are still unmeasured and, per the next bullet, still misaligned by
  construction.
- **Tablet and mobile are not aligned at all.** The `grid--peek` path has the
  same leading/trailing inset pair as desktop did, plus a column width that
  ignores the column count entirely: `base.css:1066` (`min-width: 35%`),
  `:1070` (a `1.5rem` first-item margin) and `:1075` (the peek `:after`). The
  35% is Dawn deliberately showing a sliver of the next card, which is a
  different design from ours. Measured at a 194px viewport: first item at 15px,
  item width 62.64 — both exactly those two rules.
- **The wheel handler pauses the page** at the section until the row is
  exhausted. Inherent to the pattern, and more noticeable now the listener is on
  `window`. Accepted, not yet judged against a real page length.
- **The horizontal row rules survived after all.** `box-shadow: 0 -1px 0
  var(--vv-rule-color)` on `.grid__item` is still there, now drawing along a
  single scrolling row rather than between rows. Part 7c predicted this would
  need deciding here; it was not decided.
- **The arrows are legible but not designed.** White at Dawn's alpha ramp,
  chosen to be visible. They sit next to `--vv-hairline`, which the doc already
  flags as chosen quickly — the two now share a background and should be looked
  at together.

---

## Done — the `vv-` token layer

Built 2026-08-21, in the middle of 7d and not originally planned. Prompted by
the question "shouldn't the glass blur and tint live in a stylesheet?", which
was the right instinct with the wrong destination.

**There is no `theme.css`.** Dawn's global tokens live in a `:root` block at
`layout/theme.liquid:126`, built from `settings.*` and defined in
`config/settings_schema.json`. The `assets/*.css` files are **consumers only**:
they are served as static files and no Liquid runs in them, so they can read
`var(--x)` but can never define a value that came from a setting. That is the
hard constraint behind the whole question — "put it in a CSS file" and "keep it
editable in the theme editor" are mutually exclusive for the same value.

**`snippets/vv-tokens.liquid`** is the new home: a `<style>`-only snippet with
no markup, rendered from `layout/theme.liquid:329` next to `vv-grid-lines`. It
declares `--vv-glass-blur` and `--vv-glass-tint` on `:root` as **finished CSS
values** — `4px` and `0.03` — never editor units. Sections that expose a slider
do their own conversion and override these names locally.

### Why it holds inputs, not the composed value

The old `vv-hero.liquid:22-23` baked six values into two strings: a blur radius,
a `saturate` multiplier, a `brightness` multiplier, the white `255,255,255`, an
alpha, and the arithmetic that produced it. Four of those six are constants;
only the blur radius and the alpha ever varied. Sealing them into one composed
string is what made the value unreusable — and the reason is not style, it is
mechanics.

**A `var()` inside a custom property resolves where the custom property is
*declared*, not where it is used.** `--vv-glass` declared on `.vv-hero` was
already substituted by the time it inherited anywhere else, so overriding an
input further down would have done nothing at all. Custom properties are not
functions; what inherits is a resolved string.

So the composition has to happen **in a real property, on the element being
matched**. `vv-hero` now declares the two inputs on `.vv-hero` from its settings
and composes in each of the three `:nth-child` glass blocks (`:86-88`,
`:107-109`, `:128-130`). The same composition is written three times; the blocks
select different panels per breakpoint, so a utility class cannot collapse them
without changing how the hero picks panels. Left as is.

`vv-hero`'s two schema settings at `:563` and `:573` are untouched — the hero
keeps its theme-editor controls, they just feed shared names now.

### What was deliberately not done

- **A `.vv-glass` utility class.** It would need the class in markup, and the
  only place to add it is `snippets/card-product.liquid:148`, which *every*
  product card in the theme renders. Wrong blast radius for a section-scoped
  design. Each consumer composes in its own scoped rules instead — which is also
  how Dawn works, `theme.liquid` defining `--media-*` and the `component-*.css`
  files composing them.
- **Real theme settings in `settings_schema.json`.** More "correct" Shopify, and
  it would give merchant-facing controls, but there is no need for merchant
  control of glass yet.
- **Migrating `--vv-hairline` and `--vv-text-inset`.** Both still live where they
  were — `vv-grid-lines.liquid:15` and inside a rule in
  `featured-collection.liquid`. They belong in the token layer; left alone so the
  step stayed one idea.

### Open

- **`glass_tint`'s schema is a 0–100 slider divided by 1000**, so its full range
  only reaches 10% opacity and the top three-quarters of the control are nearly
  indistinguishable. Pre-existing; preserved exactly, because this was a pure
  refactor. Worth revisiting now the cards read the same token.

---

## Done — Part 7d, the product text moves onto the image

Built 2026-08-21. **Changed shape mid-build**: the original plan was to indent
the text block so it cleared the hairlines while sitting *below* the image. The
request became "bring the descriptions inside the cards, in the lower portion of
the product images", which is a different structural problem — the text has to
leave normal flow.

### What exists now

Four rules in `featured-collection.liquid`'s style block, all scoped with
`#collection-{{ section.id }}`:

| Rule | What it does |
| --- | --- |
| `.card__information` | `--vv-text-inset: 2rem` as horizontal padding, and `--color-foreground: 0, 0, 0` |
| `.card > .card__content` | `position: absolute`, pinned bottom/left/right, horizontal padding zeroed, `--vv-glass-tint: 1` |
| `.card > .card__content::before` | the frost — `backdrop-filter`, a white background gradient, and a mask |
| `.card > .card__content > *` | `position: relative`, to lift the text above the frost |

**The child combinator matters.** `card-product.liquid` has **two**
`.card__content` elements: one inside `.card__inner` holding the badges
(`:111`), one a direct child of `.card` holding the title and price (`:148`).
They share a class and only their depth tells them apart.

**Why the text lands on the image.** Absolute positioning takes
`.card__content` out of flow, so it stops contributing height. `.card` is a flex
column whose only remaining in-flow child is `.card__inner` — the image box — so
the card's height collapses to the image height and `bottom: 0` is the bottom
edge of the photograph. `.card-wrapper` was already `position: relative`
(`component-card.css:1`), so the containing block came for free.

**The text is black, not white.** Chosen deliberately: 7c established that this
catalogue's photographs are near-white, which is why the hairlines are
interrupted by images. Black is the value that agrees with that finding. It is
right *contingently* — one dark product shot breaks it — which is what the frost
exists to fix.

### The frost, and why it took four attempts

A flat 3% white tint looked fine and did nothing. Over near-white photographs a
white wash is invisible at *any* alpha, so "looks good" and "works" were not the
same thing, and neither 0.03 nor 1.0 could be judged against the live catalogue.
The dark case is the only test that discriminates — `filter: brightness(0.2)` on
one card's `.card__media` in devtools.

The fade is **horizontal**, opaque at the left, because
`config/settings_data.json:105` sets `card_text_alignment: "left"`, so the text
really does live on the left of the card.

**Why the frost sits on a `::before` and not on `.card__content` itself.**
`backdrop-filter` applies uniformly across an element's whole box and is
completely independent of `background-image`, so fading the background does not
fade the blur — the fade dies onto a floor of blurred, 5%-brightened backdrop
with a hard edge on all four sides. The only thing that fades a backdrop filter
is `mask-image`, which fades the element's **entire rendering, children
included** — and the children here are the title and price. Isolating the frost
on a pseudo-element lets the mask fade the blur while the text stays sharp.

**The ordering fix is `position: relative` on the children, not `z-index: -1` on
the pseudo.** A positioned pseudo-element paints above non-positioned in-flow
content, so the frost would cover the title. Making the children positioned puts
them in the same painting group, where DOM order decides, and `::before` is by
definition first. The obvious alternative fails: `.card__content` is positioned
but has `z-index: auto`, so it creates **no stacking context**, and a negative
`z-index` would escape the subtree entirely and slide behind the product image.

`> *` catches both children — `.card__information` and the `.card__badge` at
`card-product.liquid:553`.

### Open after 7d

- **The two masks disagree.** `mask-image` fades from 60%, `-webkit-mask-image`
  from 50%. Chrome and Firefox read the first, Safari the second. A prefixed and
  unprefixed pair are the same declaration for different browsers, not two
  settings — they have to carry the same value. **Known at commit time; fix this
  first.**
- **Two fades multiply.** The background gradient fades across the full width
  *and* the mask fades from 60%, so the effective opacity is the product of the
  two and falls off much faster than either suggests — at 60% across, the
  background is already down to 0.4. Tuning either number moves the result
  non-linearly. The fix, if tuning gets frustrating, is to flatten the background
  to a solid tint and let the mask own the shape.
- **`--vv-glass-tint: 1`** — fully opaque white at the left edge, which erases
  that part of every photograph. It is the strongest possible setting, so there
  is no headroom if the dark case needs more.
- **Title length versus a fixed fade point.** A vertical fade would be uniform
  along the axis the text varies in; a horizontal one is not. The fade sits at a
  fixed fraction of the card's width and titles run to whatever length they run
  to, so a long title's tail crosses into the transparent end — and it will be
  the longest product name, the one never tested with, that breaks first. Three
  ways out, none taken: push the fade point past the longest title (fragile),
  hold full opacity across most of the width (then it is a flat fill with a soft
  edge), or put a `max-width` on the text block so titles wrap before they reach
  the fade.
- **The horizontal row rules are still undecided.** `box-shadow: 0 -1px 0
  var(--vv-rule-color)` on `.grid__item` survives from Parts 2–3. 7c predicted 7b
  would decide it; 7b did not; neither did 7d. There is one row now.

---

## Done — Part 7a, the heading, arrows and action move into the row

Built 2026-08-26 to 2026-08-28. **This is not what the plan below describes.**
The plan called for a heading row *above* the products, mirroring the
reference's `home_collection_info`. What was built follows the design decision
recorded 2026-08-19 instead: the section title, the "View all" action, the
product names *and* the slider arrows all sit **inside** the row. 7d had
already moved the product names; 7a did the other three.

### What exists now

Four overlays on one row, all scoped with `#collection-{{ section.id }}`:

| Element | Position | Inset |
| --- | --- | --- |
| `.collection__title` | top-left, `z-index: 1` | `--vv-text-inset` on top and left |
| `.slider-buttons` | stretched to all four edges of the row, `pointer-events: none` | none |
| `.slider-button` | pushed to both ends by `justify-content: space-between`, `pointer-events: auto` | flush |
| `.collection__view-all` | bottom-right, no `z-index` | `8px` |

`.slider-counter` — Dawn's `n / total` readout — is `display: none`. Safe to
hide or delete outright: `assets/global.js:774` guards its updates with a null
check, so the slider JS does not care whether it exists.

**The height collapse is 7d's trick, applied twice more.** With the title, the
buttons and the action all out of flow, the only in-flow child left under
`#collection-{{ section.id }}` is `slider-component`, and the only in-flow
child under that is the `<ul>`. So the section's height *is* the row's height,
which is what makes `top: 0` and `bottom: 0` mean the row's top and bottom
edges rather than some outer box's.

**Order mattered inside 7a.** The arrows had to leave flow before "View all"
could be pinned to `bottom: 0` — until they did, the bottom of the section was
the bottom of the buttons strip, not the bottom of the row. Same shape of
dependency that put 7c ahead of 7b.

**Two `z-index` answers, in opposite directions.** Positioned siblings with
`z-index: auto` paint in DOM order. `.collection__title` comes *before*
`slider-component`, so without an explicit `z-index` the product images bury
it. `.collection__view-all` comes *after*, so it paints on top for free and
declares nothing. Same rule; DOM order decides which way it cuts. Note this is
the mirror of 7d, where the fix was to make the children positioned rather than
to reach for `z-index` at all.

**`pointer-events` is load-bearing.** `.slider-buttons` is stretched to the full
width *and* height of the row, which makes it an invisible sheet over every
product link. It carries `pointer-events: none`; `.slider-button` carries
`pointer-events: auto` to get itself back. Both halves are required, because
`pointer-events` **is an inherited property** — the buttons inherit `none` from
their container. Two traps worth recording: the SVG-only values (`painted`,
`visiblePainted`, …) are treated as `auto` on HTML elements, so they silently
do nothing; and `none` suppresses **pointer hit-testing only**, leaving the
buttons tabbable and operable by keyboard either way.

### Five things fixed along the way

None of these were part of 7a. All of them were exposed by it.

**1. `--vv-text-inset` moved to the token layer.** It had been declared inside
the `.card__information` rule, where only that element's subtree could read it.
The heading is not in that subtree, so no selector could have reached it — this
was a blocker, not a tidy-up. It now sits on `:root` in
`snippets/vv-tokens.liquid` beside the glass tokens, which closes half of the
migration the token-layer section left open. `--vv-hairline` still lives in
`vv-grid-lines.liquid`.

**2. The `<ul>`'s user-agent margin.** `.grid` (`assets/base.css:894-902`) sets
`margin-bottom` and `padding` but never `margin-top`, so the browser's own
default for `<ul>` — `margin-block-start: 1em` — survives untouched. That is
**16px**, which is why the number matched nothing in a theme whose rem is 10px.
It had always been there, hidden inside `.section-…-padding`'s 44px, because
padding on a parent blocks margin collapsing. Zeroing that padding let it
escape: it collapsed up through `slider-component`, `.collection` and
`div.color-scheme-1`, and stopped at `#shopify-section-…`, which is a **flex
item** — `.content-for-layout` is a flex column (`assets/base.css:154-158`) —
and flex items do not collapse margins with their children. Both ends are now
zeroed on `#collection-… .product-grid`.

**3. `--focus-outline-padding`, not `padding-top`.** Dawn reserves vertical room
inside each slide so focus rings and card shadows are not clipped by the scroll
track: `max(var(--focus-outline-padding), var(--shadow-padding-top))` at
`assets/component-slider.css:107`. With `card_shadow_opacity` at `0` the shadow
arm resolves to `0`, so the 5px came entirely from the focus-outline arm. **The
fix is to redeclare the variable, not the padding.** One declaration reaches
all four rules that read it — `:107`, `:108`, `:112`, `:204` — at every
breakpoint. Overriding `padding-top` directly fixed ≤989 and did nothing at
desktop, where Dawn reserves its space through `padding-bottom` instead; that
asymmetry is what made the bug look breakpoint-specific. The cost is focus-ring
clearance, accepted because 7d had already shrunk the link box to
`.card__information`, well inside the card.

**4. `image_ratio` is `portrait`, not `adapt`.** Under `adapt` each card's media
box takes its own image's aspect ratio — and since 7d made `.card__content`
absolute, a card's height *is* its image's height. The `<li>`s stretch to the
tallest card; the `.card-wrapper` inside them does not. Seven of eight cards
sat 3px short and leaked page background beneath them. A fixed ratio makes
every card the same height structurally instead of by luck of the catalogue,
which matters more here than in stock Dawn because the whole design rests on
the row reading as one band.

**5. Subpixel column seams.** At four columns each `.grid__item` is 25% of a
viewport that is rarely divisible by four — measured at `432.95px` — so
adjacent edges round to different device pixels and some pairs leave a
one-pixel gap. The boxes tile exactly (each `left` is the previous `left` plus
the width, to within float error); this is **rasterization, not layout**, and
no width fixes it. `#collection-… .product-grid` now carries a background so a
seam reveals that instead of the page. It should be `--vv-hairline` rather than
a literal colour: the seams fall precisely on column boundaries, which is where
this design draws lines anyway. **This deliberately contradicts `vv-no-bg`** —
the section otherwise paints nothing so the overlay shows through — and it is
safe only because the track is completely covered by cards except at those
seams, and 7c already established that product images interrupt the hairlines.
It needs a comment in the code saying so, or it reads as a mistake.

### The legibility decision

**Stage E was closed by decision rather than by code, on 2026-08-28.** All four
overlays are black type on bare photography. That is legible today only because
this catalogue's product shots are near-white — the same contingency 7d
recorded about the product names. The answer chosen is a **content constraint:
product photographs must have white backgrounds.**

Two consequences, both accepted deliberately:

- **Nothing in the theme enforces it.** The failure mode is silent — a
  lifestyle shot uploaded two years from now makes four overlays vanish at
  once, with nothing in the code pointing at why. This paragraph is the
  enforcement.
- **7d's frost is now insurance, not structure.** It exists precisely to remove
  this dependency. Under the constraint it renders nothing visible and costs a
  `backdrop-filter` per card on a scrolling row. Kept — eight cards is cheap
  and it is already written — but it is no longer load-bearing, and the three
  tuning questions left open after 7d are moot unless the constraint is
  dropped.

### Still open after Part 7a

- **Tablet and mobile do not align.** Unchanged since 7b, and now the next
  piece of work. See below.
- **The three insets disagree.** Title 20px, "View all" a hardcoded 8px, arrows
  0. Three overlays on one grid, three numbers, in a theme whose rem is 10px
  and which has a token for exactly this. Any of them could be right; none of
  them was decided.
- **7d shrank the clickable area of every card.** `.card__heading a::after`
  (`assets/component-card.css:352-358`) is `position: absolute` with `inset: 0`
  and is meant to resolve against `.card-wrapper`, making the whole card a link
  target. 7d gave `.card > .card__content > *` `position: relative`, and
  `.card__information` matches it — so the overlay now resolves against the
  text block, and only the text strip is clickable, not the photograph. Found
  while checking a `z-index` during 7a; not investigated or fixed.
- **The caret size is capped by its wrapper.** `.slider-buttons .icon` is set to
  `1.5rem`, but `.svg-wrapper` is 20px wide (`assets/base.css:652-658`) and the
  glyph's `viewBox` is `0 0 10 6`, so `preserveAspectRatio` scales it to 20 × 12
  and centres it in a 15px box. Raising the height alone does nothing further;
  the wrapper is the next knob.
- **The horizontal row rules are still undecided.** `box-shadow: 0 -1px 0
  var(--vv-rule-color)` on `.grid__item` survives from Parts 2–3. 7c predicted
  7b would decide it; 7b, 7d and 7a all did not. There is still one row.
- **The arrows are still not designed** — only made legible, and now black
  rather than white.

---

## Next — tablet and mobile snapping

The one part of the grid that has never worked. Desktop columns rest on the
hairlines exactly, verified in 7b. Below 990px they do not line up at all, and
not by accident: three Dawn rules describe a different design.

**What is in the way**, all in `assets/base.css` under the `grid--peek` path:

- `:1066` — `min-width: 35%` on the item. Dawn is deliberately showing a sliver
  of the next card; that is the "peek" the class is named for, and it ignores
  the column count entirely.
- `:1070` — a `1.5rem` left margin on the first item.
- `:1075` — the trailing `:after` pad.

Measured at a 194px viewport during 7b: first item at 15px, item width 62.64 —
both exactly those two rules, not a rounding artefact.

**The desktop fix is the model.** 7b solved the same three problems at ≥990 with
four scoped overrides: `--desktop-margin-left-first-item: 0px`, zeroing
`.slider--desktop::after`'s padding, and the two width calcs. The tablet and
mobile paths need the same treatment aimed at `grid--peek`'s numbers instead.

**Decide before starting: keep `grid--peek` at all?** `swipe_on_mobile` is
`true` in `templates/index.json`, which switches on a design — the peek sliver
— that this grid does not want. Turning it off changes which Dawn rules apply
and may be less work than overriding each of them one by one; it also gives up
the affordance that tells a mobile visitor the row scrolls. That trade is the
first decision, not an implementation detail.

**Also unresolved at these widths:** the section renders `slider--tablet` at
*all* widths — the markup applies it whenever `show_mobile_slider` is true —
and `slider--desktop` above 990. Several of the rules fixed in 7a turned out to
be breakpoint-specific for exactly this reason. Expect the same shape of
surprise here.

---

## Part 7 as originally planned — kept for the record

> **All four sub-parts are done.** This section is the plan as written on
> 2026-08-12, left unedited so the reasoning survives. Where it disagrees with
> what was built — most sharply on 7a — the "Done" sections above are
> authoritative.

Three changes requested 2026-08-12, moving `featured-collection` toward the
reference at <https://www.sotf.com/en>.

> **Sourcing.** `sotf.com` returns HTTP 403 to automated fetches, so the live
> site could not be read. This section is written from a saved copy of the
> homepage (`~/Downloads/SOTF _ Official online shop.html`, 12 Aug). **Only the
> HTML was saved** — the stylesheets are remote and still blocked. So markup
> structure and JavaScript config below are firsthand and quotable; every
> dimension, colour and spacing value is *not*, and needs a human eye.

### What the reference actually does

**1. The hairlines are one fixed page-wide overlay, not per-item borders.**
First thing inside `.main-wrapper`, before the header:

```html
<div class="row g-0 overlay_grid_wrapper">
  <div class="d-none d-tablet-block col col-xl-2 overlay_grid_border"></div>
  <div class="col overlay_grid_border"></div>
  <div class="col overlay_grid_border"></div>
  <div class="d-none d-tablet-block col overlay_grid_border"></div>
</div>
```

Four empty divs, drawn once, with the page rendered over them. `d-none
d-tablet-block` on the first and last means **4 columns at tablet and up, 2
below** — and `col-xl-2` makes the first column narrower at xl, so the grid is
deliberately *not* equal-width at the largest size.

This is the `vv-grid-ruler` technique made permanent rather than kept as a
debug tool. **It also answers the "hairlines are still scoped to
featured-collection" question left open after Part 6**: an overlay is global by
construction — no per-section CSS, no `nth-child` arithmetic, no per-breakpoint
rule sets, and it cannot desynchronise from the sections because nothing in the
sections draws it. The deleted snippet is in git at `3cce708b^` and is the
obvious starting point.

**2. There is a heading, and it sits on the grid.** `home_collection_info` is a
row built from the same column structure as the overlay: an empty spacer cell,
the title, another empty cell, then a "View all" link. The title therefore lands
in column 2 and the action in column 4, both flush to overlay lines.

**This contradicts "the header is unnecessary."** The reference has one — it
just doesn't look like Dawn's centred `title-wrapper`, it looks like two cells
of the grid. Worth deciding deliberately rather than by default.

**3. The carousel is Slick.** Config, verbatim from the page:

```js
{ dots: false, infinite: true, speed: 500, arrows: true,
  slidesToShow: 3, slidesToScroll: 1, swipeToSlide: true,
  responsive: [ { breakpoint: 1199.98, settings: { slidesToShow: 4, slidesToScroll: 1 } },
                { breakpoint: 1024.98, settings: { slidesToShow: 2, slidesToScroll: 1 } } ] }
```

Slick `breakpoint` values are **max-widths**, so this reads: above 1199.98 show
**3**, at or below 1199.98 show **4**, at or below 1024.98 show **2**. Note the
inversion — the *widest* viewport shows the *fewest*, largest items. That is a
deliberate choice and it is not what our 4/3/2 does.

**4. Item structure — image and text are siblings.**

```
.home_collection_item
  .home_collection_item_img       <a><picture>
  .home_collection_item_info
    .home_collection_item_info_title
      .home_collection_item_info_brand
      .home_collection_item_info_article_title
    .home_collection_item_info_price
```

The image sits directly in the item; the text lives in its own wrapper
alongside it. So the image can stay flush to the column edge while only the
text block is inset — which is what "names shouldn't touch the hairlines"
asks for.

### The work

**7a — the heading. DONE, but NOT as planned here — see "Done — Part 7a"
above.** This paragraph decided on 2026-08-14 to rebuild the heading as a row
above the products, title in one cell and "View all" in another. The design
decision of 2026-08-19 overrode it: the title and the action went *inside* the
row instead, along with the arrows. The two readings below are kept for the
reasoning, but neither is what shipped:

- *Remove it*, as requested. Note that blanking `title` in
  `templates/index.json` only hides the `<h2>` — line 123 wraps that in
  `{%- if section.settings.title != blank -%}`, but the wrapper `<div
  class="collection__title title-wrapper …">` on line 122 renders
  unconditionally and keeps its margins. The wrapper has to go too.
- *Rebuild it on the grid*, as the reference does — title in one cell, "View
  all" in another, both aligned to the hairlines. Dawn already has the second
  half of this: `show_view_all` renders a `collection__view-all` block at line
  248.

Supersedes the Part 6 decision either way: aligning the heading to the grid's
outer edge by dropping `page-width` is moot if it is removed, and insufficient
if it is rebuilt.

**7b — the scroller. DONE — see "Done — Part 7b" above.** The plan below is
left as written for the record; both gaps it flags at the end (no `infinite`,
and arrows needing deliberate styling) are still gaps.

`enable_desktop_slider` and `swipe_on_mobile` are both `false` in
`templates/index.json`. Turning them on gives `.slider--desktop` with
`overflow-x: auto`, `scroll-snap-type: x mandatory` and `scroll-behavior:
smooth` (`component-slider.css:140`). The gate at line 103 needs
`products_to_display > columns_desktop` — 8 against 4, so it passes.

Two gaps against the reference to decide on, not to assume:

- **`infinite: true`.** Dawn's slider is a finite scroll-snap track; it does not
  wrap around. Matching that behaviour is a real piece of work, not a setting.
- **`arrows: true` plus a counter.** Dawn renders `.slider-buttons` with prev,
  next and an `n / total` counter at line 215. The reference has arrows but no
  dots. Close, but style it deliberately.

Also note the two rules tuned for a gutter that now resolve to zero:
`.slider--desktop:after` (line 152) offsets a 5rem trailing pad by
`calc(-1 * var(--grid-desktop-horizontal-spacing))`, and `:first-child` uses
`--desktop-margin-left-first-item` for the leading inset.

**7c — hairlines become an overlay. DONE — see "Done — Part 7c" above.** The
plan below is left as written for the record; the consequence it flags in its
last paragraph is exactly what happened, and cost most of the session.

This supersedes the per-item `box-shadow`
approach from Parts 2–3, and it is what makes 7b safe. The current rules use
`:not(:nth-child(4n + 1))`, written for a *wrapping* grid where every 4th item
begins a new row. A horizontal scroller is one continuous row of N, so `4n + 1`
would strip the rule from items 5, 9, 13 mid-row. Rather than rewrite those
selectors to `:not(:first-child)`, drawing the lines as a fixed overlay removes
the coupling between the rules and the column count altogether — which is the
reference's whole reason for doing it that way.

Consequence to check: an overlay sits behind everything, so it will show
through any section that does not paint its own background, not just this one.
That is the intent, but look at the hero and the footer before committing.

**7d — indent the item names. DONE — see "Done — Part 7d" above**, though the
approach changed mid-build: the text moved *onto* the image rather than staying
below it, so the indent stopped being the point. The structural advice below
still held — padding goes on the text wrapper, never on `.grid__item` — and
`.card__information` was indeed the right element. The plan is left as written
for the record.

**7d — indent the item names.** Not on `.grid__item`: Dawn's items are
content-box, so horizontal padding adds to the percentage width and wraps the
row. That is why the existing padding is `2rem 0`, vertical-only, and it is
already recorded as a lesson above.

Put it on the text wrapper instead — `.card__information` in
`snippets/card-product.liquid` (line 112 / 149) is the structural equivalent of
the reference's `.home_collection_item_info`. That leaves the image flush to the
column edge while the text clears the line, which is what the reference does.

### Checkpoint 7

- ~~Hairlines continuous down the whole page~~ — **done in 7c**, with the
  caveat that they stop at the header and footer, which are still opaque, and
  are interrupted by product images by design
- ~~One product row, full width, scrolling horizontally, no scrollbar on
  `<body>`~~ — **done in 7b, and alignment verified at desktop.** Tablet and
  mobile scroll but do not align
- ~~Whatever was decided in 7a, aligned to the same lines as everything else~~
  — **done in 7a**, though "aligned to the same lines" stopped describing the
  goal once the title, arrows and action moved inside the row. The title is
  inset from the first column edge by `--vv-text-inset`; the arrows sit flush
  to both row edges; "View all" is inset 8px from the bottom-right. The three
  insets do not agree with each other yet
- ~~Product names clear of the hairlines; images still flush~~ — **superseded
  by 7d.** The names moved onto the images, so "clear of the hairlines" no
  longer describes the goal. The image is still flush to the column edge; the
  text is inset `--vv-text-inset` from it and backed by the frost. **The
  dark-image case was closed by decision in 7a, not by code** — product
  photographs are now constrained to white backgrounds, which makes the frost
  insurance rather than structure
- The hero still lines up with the overlay columns

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
