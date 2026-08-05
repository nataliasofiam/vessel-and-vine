# Keeping the notch header expanded while the cart has items

Working notes / handoff. Written 2026-08-04.

**Goal:** when the cart contains at least one item, `sections/vv-header.liquid` stays
expanded instead of collapsing back to the narrow notch. When the cart empties, it
collapses again.

---

## Core design decision (already made)

Do **not** reuse the existing `data-open` attribute. `data-open` is the *transient*
state — the logo button toggles it, and three separate listeners force it back to
`false` (outside click, Escape, scroll). Anything written there gets stomped within
seconds.

Instead there is a second, independent attribute, `data-cart-locked`, meaning
"the cart has items, stay wide." The two states coexist and the CSS honours either one.

---

## Status

### Done — Step 1: server-rendered state

`sections/vv-header.liquid`

- Lines 7–12: inside the existing `{%- liquid -%}` block, an `if cart.item_count > 0`
  sets `cart_locked` to `true` / `false`.
- Line 233: the `.vv-header` div outputs it as `data-cart-locked`.

This makes the header correct on any full page load — first visit, refresh, navigation.

> Nit to clean up: line 233 currently renders the attribute value **unquoted**
> (`data-cart-locked = {{cart_locked}}`). Legal HTML, but inconsistent with every other
> attribute in the file and fragile if the value ever contains a space. Wrap it in
> quotes and drop the spaces around the `=`.

### Done — Step 2: CSS

Rather than adding new rules (which would have created a specificity fight with the
existing `[data-open='true']` and `:hover` rules that all set `width`), the locked
selector was **added to the existing selector lists**. Four places:

| Line | What it drives |
|------|----------------|
| 165  | desktop bar `width` |
| 169  | nav `opacity` |
| 214  | mobile nav `max-height` |
| 219  | mobile `nav--right` padding |

Verified working: load a page with a non-empty cart and the header renders expanded.

### Not started — Step 3: keeping it in sync without a page reload

This is where work stopped.

**The problem it solves:** Liquid runs once on the server. After the HTML reaches the
browser the attribute is a frozen snapshot. Deleting an item from the cart happens over
AJAX with no reload, so nothing tells the attribute the cart changed — it goes stale.

**Current state of the code** (inside the IIFE in `vv-header.liquid`, ~lines 291–336):

- `setCartLocked(count)` is written and correct — takes a count, sets
  `root.dataset.cartLocked` accordingly.
- A `subscribe(PUB_SUB_EVENTS.cartUpdate, ...)` call is wired up with a `console.log`
  placeholder as the callback.
- It is wrapped in a `DOMContentLoaded` listener, which is **required** — `pubsub.js`
  loads with `defer` (`layout/theme.liquid` line 53), so `subscribe` does not exist yet
  when this inline script runs during parsing. Deferred scripts always finish before
  `DOMContentLoaded` fires.
- Minor: there is still a stray trailing comma after the callback argument. Legal, untidy.

**What still needs doing:**

1. Confirm the log fires (see blocker below — test via the cart page, not add-to-cart).
2. Replace the log with a `fetch` of `window.routes.cart_url + '.js'`
   (`window.routes` is defined in `layout/theme.liquid` line 358). That endpoint always
   returns the full current cart.
3. Chain **two** `.then` calls — the first returns `response.json()`, the second receives
   the parsed cart. Read `item_count` off the second one and pass it to `setCartLocked`.
4. Delete the `console.log`s when it works.

**Why fetch instead of reading the event payload:** the payload is inconsistent.
On a delete or quantity change (`assets/cart.js` line 256) `cartData` is the full cart and
has `item_count`. On an add (`assets/product-form.js` line 80) `cartData` is the
`/cart/add.js` response — the line item that was added — with **no** `item_count`.
Since `undefined > 0` is `false`, reading it directly would collapse the header on every
add: the exact opposite of the goal, and an annoying bug to trace.

**Reference implementation already in the repo:** `assets/cart.js` line 38 is a real
`subscribe(PUB_SUB_EVENTS.cartUpdate, ...)` call. It uses arrow syntax rather than
`function`, but the structure is identical. Worth reading before writing this.

---

## ⚠️ Blocker found — add-to-cart never publishes the event

Do not debug the JS against add-to-cart. **The event is not being sent.** Chain of cause:

1. `assets/product-form.js` line 11:
   `this.cart = document.querySelector('cart-notification') || document.querySelector('cart-drawer');`
2. `config/settings_data.json` line 183 sets `"cart_type": "notification"`, so
   `cart-drawer` is not rendered (`layout/theme.liquid` line 329 only renders it in
   `drawer` mode).
3. `<cart-notification>` is not on the page either. Stock Dawn renders it from
   `sections/header.liquid` line 309 — but `header-group.json` uses **`vv-header`**
   instead. Replacing the header dropped the cart notification, and `cart-notification.js`
   (`sections/header.liquid` line 103) with it.
4. So `this.cart` is `null`, and `product-form.js` lines 70–73 bail out to
   `window.location = window.routes.cart_url` — a **full page redirect**. The
   `publish(PUB_SUB_EVENTS.cartUpdate, ...)` on line 80 sits below that `return` and is
   never reached.

The navigation also clears the console by default, so even a working log would vanish.

### Testing around it

- **Cart page removals still publish.** Go to `/cart` and remove an item or change a
  quantity — that path goes through `assets/cart.js` line 256 and does fire `cartUpdate`.
  Use this to develop and test step 3 today, no fix required.
- **Or trigger it by hand:** call `publish` (`assets/pubsub.js` line 17) from the console
  with the event name and an empty object. Isolates "my handler works" from
  "the trigger works."
- Turn on **Preserve log** in the DevTools console settings (gear icon) so navigation
  stops wiping the output.

### Fixing the add path — separate task, design decision

Three options:

1. **Switch `cart_type` to `drawer`** in theme settings. `layout/theme.liquid` renders the
   drawer independently of the header (line 329) and already loads `cart-drawer.js`
   (line 402). **Zero code**, restores AJAX adds immediately. Only question is whether a
   slide-out drawer fits the design.
2. **Render the `cart-notification` snippet from `vv-header.liquid`** and load
   `cart-notification.js`, mirroring `sections/header.liquid`. Keeps the current cart
   style; more work, and the notification's styling may fight the notch.
3. **Leave the redirect.** Every add reloads the page, so Liquid re-renders and steps 1–2
   already handle adds correctly. Step 3 would then only cover removals and quantity
   changes — still a real gap, so the JS work is worth finishing regardless.

Treat this as its own task. It's a design call, not a bug fix.

---

## Still open after step 3

**Step 4 — the toggle button.** Two loose ends:

- `aria-expanded` on the logo button (line ~253) is only updated by `setOpen`. When the
  cart locks the bar open, the menu is visibly expanded but the button still announces
  `false` to screen readers. Either have the lock function update it too, or have
  `setOpen` compute it as "open OR locked."
- Clicking the logo while locked flips `data-open` but nothing moves — a dead-feeling
  button. Decide: make the click a no-op while locked (simpler), or let an explicit
  `data-open='false'` override the lock, making it "default open" rather than a hard lock
  (friendlier). Pick one and be consistent.

**Step 5 — the cart count text.** Line 283 renders `({{ cart.item_count }})` from Liquid,
so after an AJAX change it shows a stale number even when the bar behaves correctly.
The count is already being fetched in step 3 — add a `data-` hook to that span and write
the number into it in the same callback.

---

## Test checklist

1. Empty cart → collapsed; hover still expands.
2. Add an item → stays expanded (blocked until the add path is fixed; currently the page
   reloads, which also happens to produce the right result).
3. With items in the cart: scroll, click the page background, press Escape → **still
   expanded.** This is the test that proves `data-open` and `data-cart-locked` stayed
   independent.
4. Refresh, then navigate to another page → still expanded.
5. Remove the last item on `/cart` → collapses without a reload.
6. Repeat at a mobile width (< 990px), where the expanded state is `max-height`,
   not `width`.
7. Confirm the cart count in the header matches reality after each change.

---

## Debugging lessons worth keeping

- **One bad selector kills the whole rule.** A `..vv-header__nav` typo in a
  comma-separated selector list invalidated the entire block, which also disabled the
  working `[data-open='true']` rule beside it. Symptom was "menu links never appear,
  even on click." In DevTools, an invalid rule doesn't show in the Styles panel at all;
  an overridden one shows struck through. Different symptoms, different causes.
- **SyntaxError vs ReferenceError.** A syntax error means *nothing* in that `<script>`
  runs — like a build error. A runtime error runs until it throws, then stops, silently
  killing every line below it in the same block. Read the error *type*, not just the
  message.
- **Browser line numbers ≠ file line numbers.** Shopify inlines the section into the page,
  so `cart:919` refers to the rendered page. Match on the message and surrounding code.
- **"My handler didn't fire" has two causes:** the handler is broken, or nothing was sent.
  Check the second one first — a lot of time went into debugging correct code here.
- **Replacing a stock section silently removes everything it rendered.** Dawn's
  `header.liquid` also pulled in the cart notification markup and its script. Nothing
  errors; a feature just stops existing.
- **Three naming conventions are in play, for the same concept:** Liquid variables use
  `snake_case`, HTML data attributes use `kebab-case`, and JS `dataset` uses `camelCase`
  (`cart_locked` → `data-cart-locked` → `dataset.cartLocked`).
- **Liquid and JavaScript never share variables.** Liquid runs once on the server and
  produces text; JS runs later in the browser. `cart.item_count` does not exist in JS.
  The only way data crosses is if Liquid *prints* it into the HTML — which is exactly what
  `data-cart-locked` does.
- **Build in layers and test each one**, since JS has no compiler to catch a mistyped
  property — it just returns `undefined` and carries on.
