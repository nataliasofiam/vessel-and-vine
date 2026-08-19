# Consistent grid — state and next steps

Handoff notes. **Parts 1 through 6 and 7c are committed and the grid system
works.** The hero panels line up with the product columns at every breakpoint,
driven by three settings; glass and distortion share one definition of "outer";
the gutter is zeroed theme-wide. The scaffolding (ruler, empty stubs) is
deleted. The hairlines are now one page-wide overlay in
`snippets/vv-grid-lines.liquid`, which also owns the line colour for the whole
theme.

**Part 7 is being built in the order 7c → 7b → 7d → 7a**, decided 2026-08-14.
7c first because the per-item `:nth-child` hairline rules would break in 7b's
scroller, so removing that coupling had to come first. **7a is settled: the
heading gets rebuilt on the grid** — title in one cell, "View all" in another —
not removed. That supersedes both the original request and the Part 6 decision.

**7b is built but not verified** — the product grid is a horizontal scroller
now, driven by a custom wheel handler. See "Done — Part 7b". The one thing
still unconfirmed is the thing this whole branch is about: whether the columns
come to rest exactly on the hairlines. Read "Where we are" first.

**Next is 7d, then 7a.** A design decision added 2026-08-19 reaches across
both: the section title, the "View all" action, the product names *and* the
slider arrows should all sit **inside** the row, on the grid, rather than above
and below it. 7a and 7d already cover the first three. The arrows are new work
with no reference to copy — the reference puts its arrows outside the row.

**Knowingly incomplete:** the footer and header are still opaque so the lines
stop at both ends of the page, and tablet/mobile columns do not line up with
the overlay at all yet. See "Still open after Part 7b".

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
| *(recorded in the follow-up commit)* | featured collection becomes a scroller — **Part 7b**, desktop only |

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

Built 2026-08-19. **Desktop only, and the alignment is not verified** — see
"Still open after Part 7b" before trusting any of it.

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

**Point 6 is the part that matters and the part that is unverified.** Snapping
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

### Still open after Part 7b

- **UNVERIFIED: do the columns actually rest on the hairlines?** This is the
  branch's entire premise and it has not been confirmed since the JS settling
  replaced the CSS snapping. Check at rest, at several scroll positions, and
  specifically at the far-right end. The far right is only a valid column
  position if the `::after` pad is genuinely gone — confirm `scrollWidth` equals
  `itemWidth × 8` and that `(scrollWidth - clientWidth) / itemWidth` is a whole
  number. If it is not, the settle clamps to a limit that is not on the grid and
  the last screen will always be off.
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

## Next: Part 7 — featured collection as a full-width scroller

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

**7a — the heading. DECIDED 2026-08-14: rebuild it on the grid**, the second
reading below. Title in one cell, "View all" in another, both aligned to the
overlay lines. Do this last, after 7b and 7d. The two readings, kept for the
reasoning:

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
  `<body>`~~ — **done in 7b at desktop only.** Tablet and mobile scroll but
  do not align, and whether the columns rest on the hairlines is unverified
- Whatever was decided in 7a, aligned to the same lines as everything else
- Product names clear of the hairlines; images still flush
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
