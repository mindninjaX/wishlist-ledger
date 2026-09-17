# Wishlist Ledger

A ranked list of 21 non-essential purchases, gadgets, a content-creator kit (camera, mic, gimbal),
a music setup (guitar, drum kit, turntable, speaker), and a run of home appliances (TV, fridge,
AC, coffee machine) added one request at a time in the same session. One self-contained HTML
file, no build step. Full item list lives in the `ITEMS` array in `index.html`, not repeated here
since it kept growing mid-build.

**Local:** `python3 -m http.server 4331` from this directory, then http://localhost:4331

## Why it exists

A wishlist that only shows price does not answer the actual question, which of these is worth
buying first. Every item gets 4 scores and 2 derived numbers, sorted any way, in either a List
or a Grid view.

## Views

**List** is the default: one row per item, expandable for the full metric breakdown.
**Grid** is a 3-column (2 on mobile) image-forward view, one product photo per cell with the
worth-it ring as a corner badge, name and price beneath, no expand. Both views share the same
sort control and the same underlying rows, just restyled by CSS off a `view-list` / `view-grid`
class on the list element, so sorting, the fallback icons and the live data are identical in
both. Principle borrowed from a Mobbin reference he sent (a closet app's photo grid): square
image, bold name, one line beneath, tight hairline divisions. Not a layout copy, its rounded
pill nav and light theme stayed out, only the grid-cell shape carried over into this page's own
dark ledger look.

## The model

Base scores, each 0 to 100, judgment calls not measurements:

- **Daily use**: how often it would actually get touched
- **Flex**: pure status value, would a stranger notice
- **Longevity**: realistic years before replacement, capped at 6
- **Joy**: fun per use, independent of frequency

```
Worth it = 0.40 * daily use + 0.25 * (longevity / 6 * 100) + 0.20 * joy + 0.15 * flex
Cost per year = price / longevity
Cost per month, week, day = cost per year / 12, 52, 365
Per joy point = price / joy
```

Daily use is weighted highest on purpose: a cheaper thing used constantly beats an expensive
thing used rarely, which is the actual argument against buying most of these on impulse.

## Budget reality check

Three more numbers sit above the list. An editable monthly income turns the total price into
years and months of income to buy everything outright. A steady-state monthly cost (the sum of
every item's own cost-per-month, not the sticker total) feeds a required-income figure on the
standard 50/30/20 budgeting rule: if owning and replacing all of this is meant to be 30% of a
budget rather than the whole thing, that monthly cost implies an income, and the same split shows
what the other 50% and 20% would need to cover. This is a real named rule, not a made-up score
like the rest of the page, and the page says so; it is illustration, not advice.

The income field defaults to a generic ₹50,000, not his real number, on purpose: this page
publishes under his name, and `apps/money-dashboard` already got corrected once for shipping an
actual income figure in public. Typing a real number in stays local to the browser
(`localStorage`), same as the theme choice.

## Data

Prices are real, checked against Apple India, Sony India and Flipkart on 17 Sep 2026, and quoted
at the official MRP, not a same-day discounted price, so they stay comparable to each other.
Sony's price was corrected same day from an earlier ₹39,990 to the real ₹49,990 MRP after two
independent Flipkart listings and the struck-through MRP on a live listing disagreed with the
original figure. Scores are opinion, written into the data array in `index.html`, editable by
hand.

Product photos are hotlinked from Flipkart's own image CDN, chosen for matching the exact current
model, not a similar-looking older one: the first pass used a Wikipedia photo of the original
2020 PlayStation 5 for a Slim-era price, and an iPad Pro M4 photo for an M5-priced row, both
visually correct-looking but the wrong generation, replaced once the mismatch was caught.
`onerror` swaps any image that fails to load for a matching line icon, so a dead link never shows
a broken-image box.

## Theme

Dark by default, a light theme is one click away (top right, the moon or sun button), remembered
in `localStorage` and restored before the page first paints so there is no flash of the wrong
theme on reload. Every colour is a CSS custom property, so the toggle is a full second palette,
not an inversion filter: light mode deepens the score-ring colours and the brass accent so they
still hold contrast on a pale background, not the same hex values just moved to a lighter page.

## Verification

Served locally and driven headlessly over CDP (real Chrome, synthetic mouse coordinates and key
events, not `element.click()`): all 21 rows render, 0 image failures, 6 fall back to their icon
by design (no clean current-generation photo found for those), no console errors. Confirmed by
direct interaction: row expand
and collapse by mouse click and by Space key, sort dropdown re-sorts correctly on every metric,
the direction toggle reverses it, layout reflows to the two-row mobile grid at 390px wide. Enter
key on a focused row was not exercised past a synthetic-input quirk in headless CDP (Space
confirmed native button activation is wired correctly; Enter needs a real keypress to test, which
this harness cannot send).

Grid view checked the same way: switching to it via a real click sets the 3-column layout (2 at
390px wide), sorting still reorders correctly with the score ring and price intact per cell, no
console errors. One false alarm on the way there worth recording: the first pass at this check
reused `const sel` across two separate `Runtime.evaluate` calls in the same page context, which
CDP treats as one shared top-level scope, so the second declaration threw `SyntaxError: Identifier
'sel' has already been declared` and silently skipped the re-sort, which read like a page bug
until the fix (wrap each snippet in its own `(() => {...})()`) confirmed it was the test script,
not `index.html`.

Theme toggle checked the same way: a real click flips `data-theme`, persists it to `localStorage`,
and the ring stroke colour reads back as the light-mode `--sage` value (not the dark-mode one),
confirming the ring is repainted on toggle rather than left with a stale colour. A reload
immediately after showed the theme attribute already set on the very first read, confirming the
pre-paint script beats the flash it exists to prevent.
