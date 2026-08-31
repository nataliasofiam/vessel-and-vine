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

`show_scarcity`, the parameter still to be added in Stage 2, is the opposite case and
*should* be a setting — whether to show remaining stock on collection cards is a
merchandising choice, and the merchant is the one qualified to make it.

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

Currently **unconditional**, so it shows in all seven sections that render cards. That is
what Stage 2 fixes.

---

## Next — Stage 2: make it a setting

Model every step on the existing `show_rating` parameter, which appears in the same four
places. After step 3 the indicators disappear from the cards; they come back at step 6.

**`snippets/card-product.liquid`**

1. Add `show_scarcity` to the `Accepts:` list (lines 4–19), matching the house format.
2. Guard the render on line 211 behind it. Match the whitespace-marker style of the
   `show_vendor` block at lines 166–169 — this is indented block markup, not the inside
   of an HTML tag, so the constraints differ from the `<p>` in the snippet.
3. Guard the stylesheet on line 28 with the same condition, nested inside the existing
   `unless skip_styles`. Without this, all seven card-rendering sections download the CSS
   even with the feature off. Check what happens on the first card when the setting is
   off — specifically whether the loop still sets `skip_styles` to `true` afterwards.

**`sections/featured-collection.liquid`**

4. Pass it through in the render call at lines 342–354, alongside
   `show_rating: section.settings.show_rating` on line 349.
5. Leave the second render call at line 378 alone — that is the no-collection placeholder
   branch, which passes no `card_product`, so the
   `{%- if card_product and card_product != empty -%}` wrapper on line 31 already stops
   everything. Adding the argument there would be dead code.
6. Add a `checkbox` setting to the schema near line 811. Two departures from `show_rating`:
   use a plain literal `label` string rather than a `t:` translation key (no locale entries
   exist for this, and adding them is a detour — the `scarcity` block at line 776 of
   `main-product.liquid` already uses a literal), and decide the `default` deliberately.
   A `default` only applies where the section's settings are not already recorded in the
   JSON template, so changing it later will not move sections already placed.

**Checkpoint:** toggle the checkbox off — indicators gone and no `component-scarcity.css`
link in view-source; toggle on — indicators back, stylesheet present exactly once.

## Then — Stage 3: the collection page grid

`sections/main-collection-product-grid.liquid:172` renders the same card snippet. Repeat
the Stage 2 pattern there, unaided. The section's schema is separate, so it needs its own
setting.

---

## Still open

- **The `aria-live` region probably does not announce.** The attribute is emitted
  correctly on the product page, but Shopify's variant picker replaces the whole section
  HTML rather than editing the text in place, so the element carrying `aria-live` is
  destroyed and recreated with its new content already present. Screen readers generally
  do not announce a live region added in the same tick as its content. The fix is to move
  the live region onto a wrapper that *survives* the swap and let only the inner text be
  replaced. **Never verified on a real screen reader.**
- **The other card-rendering sections** — `related-products`, `main-search`, `collage`,
  and the card inside `main-product` itself — have no setting and no decision yet. They
  inherit whatever `show_scarcity` defaults to once Stage 2 lands.
- **`edition_fallback` is unreachable from cards.** Any product without a
  `custom.edition_size` metafield reads "N of 5 available" on a card regardless of its
  real edition. Either backfill the metafield across products, or decide the card should
  show a different string when the edition size is unknown.
- **Colours are literal hex** in `component-scarcity.css` (lines 2–5), while the rest of
  the theme's shared values live in `snippets/vv-tokens.liquid`. Worth deciding whether
  these four belong there too.
- **Nit:** the `scarcity` block's schema in `sections/main-product.liquid` closes with
  `]` and `}` at lines 780–781 indented two levels shallower than every sibling block.
  Valid JSON, just untidy.
- **Nit, pre-existing:** `snippets/card-product.liquid:171` renders
  `{{ block.settings.description }}`, but there is no `block` in a rendered snippet's
  scope, so that `<span>` has always been empty. Unrelated to this branch, but it is the
  exact failure mode `{% render %}` isolation is designed to prevent.
