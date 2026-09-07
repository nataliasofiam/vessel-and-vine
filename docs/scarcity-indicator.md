# Scarcity indicator on grid product cards

Working notes / handoff. Written 2026-08-31. Branch `scarcity-indicator-grid-items`.

**Goal:** the scarcity indicator that already runs on the product page also renders
inside grid product cards, driven by a merchant setting rather than hardcoded, and
without leaking into the five other sections that share the card snippet.

---

## What the indicator is

`snippets/scarcityindicator.liquid` renders a single `<p class="scarcity scarcity--STATE">`
holding a coloured dot and a line of text. Four states, decided in the `{%- liquid -%}`
block at the top of the snippet:

| State | Condition | Text |
|-------|-----------|------|
| `sold` | `variant.available == false` | Sold out |
| `ok` | inventory not tracked, or above threshold | Available / `N of M available` |
| `low` | `qty <= low_threshold` (half the edition) | `N of M available` |
| `critical` | `qty <= 1` | `N of M available` |

Edition size resolves in this order: the `custom.edition_size` metafield on the product,
then the `edition_fallback` argument, then `5`.

Colours live in `assets/component-scarcity.css` as four custom properties on `.scarcity`.

---

## Core design decision (already made)

The snippet takes three optional arguments, and **each one is a fact the caller knows
that the snippet cannot work out for itself.** Deciding which of them deserved a
merchant-facing setting was the main call on this branch:

- `block` — present only when rendering from a theme-editor block, so the snippet can
  emit `shopify_attributes` and be selectable in the editor. Omitted from cards; there
  is no block there.
- `edition_fallback` — a section setting on the product page. Omitted from cards.
- `dynamic_content` — whether this instance's text can change after page load, which
  decides whether it gets `aria-live="polite"`. **Deliberately not a merchant setting.**
  A shop owner has no way to answer "should this be a live region?", and a wrong answer
  silently degrades accessibility. It is passed as a literal `true` from the product
  page (variant switching changes the count) and omitted on cards (nothing changes).

`show_scarcity`, added in Stage 2, is the opposite case and *is* a setting — whether to
show remaining stock on collection cards is a merchandising choice, and the merchant is
the one qualified to make it.

All of this works because `{% render %}` is scope-isolated: the snippet sees only what
it is explicitly passed, so omitting an argument reliably yields `nil` rather than
picking up a same-named variable from the calling section.

---

## Status

### Done — snippet parameterisation

`snippets/scarcityindicator.liquid`

- Header comment documents all four arguments under `Accepts:`, with types and defaults.
- `edition` resolves through the metafield / `edition_fallback` / `5` chain.
- The `<p>` on line 33 guards both `aria-live` and `block.shopify_attributes` behind
  their own conditionals.

> Fixed along the way: the guards were originally written with whitespace-stripping
> markers on both sides of every gap (`{%- if x -%}`), which deleted the spaces
> *between* the attributes and emitted `class="…"data-shopify-editor-block="…"` with
> no separator — an HTML parse error browsers happen to recover from. Dropping the
> dashes restored the separators. The cost is a couple of harmless stray spaces before
> the `>`.

### Done — product page wiring

`sections/main-product.liquid:96` passes `block`, `product`, `edition_fallback` and
`dynamic_content: true`.

### Done — Stage 1: cards render the indicator (`ea867c04`)

`snippets/card-product.liquid`

- Line 28: `component-scarcity.css` loaded inside the existing `{%- unless skip_styles -%}`
  block, so it emits once per section rather than once per card.
- Line 211: `{% render 'scarcityindicator', product: card_product %}`, directly below the
  price render. Note the rename at the boundary — the snippet's parameter is `product`,
  the card's variable is `card_product`.

At this point still **unconditional**, so it showed in all six sections that render cards
(seven render sites — `featured-collection` has two). That is what Stage 2 fixed.

---

## Done — Stage 2: the setting (uncommitted)

Modelled throughout on the existing `show_rating` parameter, which appears in the same
four places. Line numbers below are post-edit.

**`snippets/card-product.liquid`**

- Line 11: `show_scarcity` added to the `Accepts:` list, directly after `show_rating` so
  the `show_*` toggles stay grouped. Note the list is internally inconsistent — most
  entries carry an `(optional)` marker but `show_vendor` and `show_rating` do not. The new
  entry follows its neighbours rather than the majority.
- Lines 213–215: the render guarded behind `show_scarcity`, matching the whitespace-marker
  style of the `show_vendor` block at 167–170 — `{%- if -%}` / `{%- endif -%}`, stripping
  both ends, guarded line indented one level in. This is indented block markup, where the
  surrounding whitespace is only newlines between elements, so aggressive stripping is
  correct. That is the *opposite* call from the `<p>` inside the snippet, where stripping
  ate the separators between HTML attributes.
- Lines 29–31: the stylesheet guarded by the same condition, **nested inside** the existing
  `unless skip_styles` rather than wrapped around it. Without the guard, all six
  card-rendering sections download the CSS even with the feature off.

> **The nesting direction is the trap in this stage.** Wrapping the whole `unless` block in
> `if show_scarcity` would suppress all six stylesheets in it — rating, volume pricing,
> price, quick-order-list, quantity-popover — whenever scarcity was off. The page still
> renders and still looks broadly right, so nothing announces the mistake. Guard the one
> line, never the block.
>
> Also fixed here: the first attempt added the guarded copy but left the original
> unguarded `stylesheet_tag` in place above it, so the CSS shipped unconditionally and the
> checkpoint looked like a nesting failure when it was a duplication one.

**Resolved: can the snippet affect when `skip_styles` flips?** No. `skip_card_product_styles`
exists only in `featured-collection.liquid` — initialised `false` at 325, passed in at 350,
set `true` at 356, after the render call and inside the section's own loop. The snippet
never sees that variable, only `skip_styles`, a copy of its value under a different name.
`{% render %}` scope isolation means an `assign` inside the snippet is local and discarded
on exit; there is no path back out to the caller. So the flip is unconditional and entirely
the section's business, and the guard changes only *which* stylesheets card 1 emits, never
*whether* card 1 is the one to emit them.

**`sections/featured-collection.liquid`**

- Line 350: `show_scarcity: section.settings.show_scarcity,` passed through, mirroring
  `show_rating` on 349.
- Line 378 deliberately untouched. That is the no-collection placeholder branch, which
  passes no `card_product`, so the `{%- if card_product and card_product != empty -%}`
  wrapper on line 33 of the snippet already stops every card body inside it. Passing the
  argument would be dead code that implies to the next reader that the branch supports the
  feature.
- Lines 819–824: a `checkbox` setting, with two departures from `show_rating`. The `label`
  is a plain literal string rather than a `t:` translation key — no locale entries exist
  for this, adding them is a detour, and a `t:` key with no matching entry renders as the
  raw key text in the editor. The `scarcity` block in `main-product.liquid` already sets
  that precedent.

### The `default: true` decision

Every other toggle in this schema defaults `false`. This one does not, deliberately:
showing remaining stock is the point of the feature, and a merchant who does not want it
can untick it.

Two consequences worth having written down:

- **It reached the section already on the homepage.** A schema `default` applies only to
  keys *absent* from the section's recorded settings in the JSON template. The
  `featured_collection` section in `templates/index.json` has `show_vendor` and
  `show_rating` recorded at lines 90–91 but no `show_scarcity` key, so the default applied
  and the indicators returned without touching the editor. Once that section is opened and
  saved in the editor, all current values get written into the JSON and later changes to
  this default will no longer move it.
- **"Always on" means "on wherever it is wired" — currently two sections of six.** A schema
  default is per-section. `collage`, `main-product`, `main-search` and `related-products`
  pass no `show_scarcity` at all, so they get `nil`, so off, whatever this default says.

The alternative considered and rejected: make the *snippet* treat an omitted `show_scarcity`
as on, which would reach all six at once. It breaks the `show_*` convention every other
toggle in the file follows (omitted = off) and makes the stylesheet guard meaningless.
Sections get wired deliberately; omission does not mean yes.

**Checkpoint (passed):** indicators back on the cards with no editor visit;
`component-scarcity.css` present exactly once in view-source; the checkbox appears in the
theme editor already ticked; unticking removes both the indicators and the stylesheet
link; the product page is unchanged throughout — it passes `block`, `edition_fallback` and
`dynamic_content` and knows nothing about `show_scarcity`.

## Then — Stage 3: the collection page grid

`sections/main-collection-product-grid.liquid:172` renders the same card snippet. Repeat
the Stage 2 pattern there, unaided. The section's schema is separate, so it needs its own
setting. The card-snippet half of the work is already done and shared; only the
pass-through and the schema entry are new.

## Still open

- **The `aria-live` region probably does not announce.** The attribute is emitted
  correctly on the product page, but Shopify's variant picker replaces the whole section
  HTML rather than editing the text in place, so the element carrying `aria-live` is
  destroyed and recreated with its new content already present. Screen readers generally
  do not announce a live region added in the same tick as its content. The fix is to move
  the live region onto a wrapper that *survives* the swap and let only the inner text be
  replaced. **Never verified on a real screen reader.**
- **The other card-rendering sections** — `related-products`, `main-search`, `collage`,
  and the card inside `main-product` itself — have no setting and no decision yet. Stage 2
  settled what happens to them in the meantime: they pass no `show_scarcity`, so they get
  `nil`, so the indicator is **off** in all four regardless of the `default: true` on
  `featured-collection`. Each needs its own pass-through and schema entry to turn on.
- **`edition_fallback` is unreachable from cards.** Any product without a
  `custom.edition_size` metafield reads "N of 5 available" on a card regardless of its
  real edition. Either backfill the metafield across products, or decide the card should
  show a different string when the edition size is unknown.
- **Colours are literal hex** in `component-scarcity.css` (lines 2–5), while the rest of
  the theme's shared values live in `snippets/vv-tokens.liquid`. Worth deciding whether
  these four belong there too.
- **The `featured-collection` checkbox label is still first-draft.**
  `"Scarcity Indicator, i.e. X of Y available"` does three jobs in one string. The
  neighbouring `t:` keys resolve to bare sentence-case noun phrases — `show_vendor` →
  "Vendor", `show_rating` → "Product rating" — because the checkbox itself supplies the
  verb, and Shopify puts any explanation in a separate `info` key rendered as grey helper
  text. The current label is Title Case, leads with the developer phrase "scarcity
  indicator" rather than what the merchant sees, and its `i.e.` claims a single text form
  when the snippet also renders "Sold out" and "Available". Reshape as a short noun phrase
  plus an optional `info`.
- **Nit:** the `scarcity` block's schema in `sections/main-product.liquid` closes with
  `]` and `}` at lines 780–781 indented two levels shallower than every sibling block.
  Valid JSON, just untidy.
- **Nit, pre-existing:** `snippets/card-product.liquid:171` renders
  `{{ block.settings.description }}`, but there is no `block` in a rendered snippet's
  scope, so that `<span>` has always been empty. Unrelated to this branch, but it is the
  exact failure mode `{% render %}` isolation is designed to prevent.
