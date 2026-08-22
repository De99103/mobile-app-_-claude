# Handoff: Rack — wardrobe index (mobile)

## Overview
Rack is a mobile app for cataloguing the clothes you own: photograph a piece, record where you bought it and what you paid, log each time you wear it, and assemble saved outfits from the pieces. It also accepts a shop product link and lets you route the result either into the closet (already owned) or onto a shopping list (not bought yet).

## About the design files
The files in `reference/` are **design references created in HTML** — a working prototype of the intended look and behaviour, not production code to copy. The task is to **recreate these screens in the target codebase's own environment** (React Native, SwiftUI, Flutter, Compose, etc.) using its established patterns, navigation, and component library. If no environment exists yet, pick the framework appropriate for the project and implement the designs there.

`reference/Rack v2.dc.html` is the whole app in one file. It runs in a design-tool runtime (`support.js`) that renders a template plus a logic class; treat the template as markup structure and the logic class as the state model. Ignore the iPhone bezel (`ios-frame.jsx`) — it is presentation chrome for the prototype only, not part of the app.

## Fidelity
**High-fidelity.** Colours, type, spacing, copy and interaction states are final and come from the Industry design system (`reference/design-system/`). Recreate the UI faithfully, substituting the target codebase's equivalents where a native pattern is clearly better (e.g. a native bottom tab bar, native modal sheet, native date picker).

One deliberate exception: all garment photography is represented by **drag-and-drop image placeholders** (`image-slot.js`), because the prototype has no real photos. In the real app these are camera / photo-library slots.

## Design language (Industry)
Industry is a technical-wireframe style: steel blue on a light ground, condensed headings, visible grid, and framed "blueprint" objects.

- **Blueprint frame** — the system's signature. A square-cornered, 1px-bordered, **transparent** box with a small `+` registration mark at each of the four corners. Every card, figure, photo frame and dialog wears it. In HTML this is `class="blueprint"` plus four `<i class="corner tl|tr|bl|br">` children. Recreate as a reusable container component.
- Cards and figures are **line drawings** — never a filled surface, never rounded.
- The **one solid object** on any screen is the primary button: an accent fill, still square-cornered.
- Photographs are duotoned into the accent (`.duotone`).
- Icons: Lucide, stroke-width 1.5, no thicker.

## Design tokens
Read the authoritative values from `reference/design-system/styles.css` (`:root`). The ones this app uses:

| Token | Value | Used for |
| --- | --- | --- |
| `--color-bg` | `#f2f2f3` | app background, sheets, tab bar |
| `--color-text` | `#1d1f20` | all body and heading text |
| `--color-accent` | `#5980a6` | corner marks, primary fill, arrows, active states |
| `--color-accent-700` | ramp step | accent-coloured small text (kickers, cost-per-wear) — accent itself fails contrast at body size |
| `--color-accent-900` | ramp step | toast background (text reversed to `--color-bg`) |
| `--color-divider` | ramp step | hairline rules, photo-frame borders |
| `--font-heading` | Barlow Condensed | screen titles, card titles, item names, tab labels |
| `--font-body` | Barlow | body copy, form labels, table cells |

Alpha variants used inline on top of `--color-text`: `cc` (~80%) for lead paragraphs, `aa` (~67%) for secondary/help text, `99` (~60%) for monospace meta.

**Monospace meta.** A recurring small-print style, class `.mono`: `10px` system UI monospace, `--color-text` at 60%. Used for counts, dates, "3 pieces · $412 in", shop city lines.

**Kicker.** Class `.kick`: `10px` monospace, letter-spacing `.12em`, uppercase, `--color-accent-700`. Sits above every screen title.

Spacing: use the `--space-*` scale (density 0.85×). Radius: `4px` from `--radius-*`, but note the blueprint objects are deliberately square — do not apply radius to cards, figures or buttons.

## Layout frame
- Design canvas: **402 × 874** (iPhone 14/15 logical size). Nothing is pixel-pinned; everything is a vertical flow.
- Structure: a single scrolling content column above a fixed bottom tab bar.
- Screen padding: `58px` top (clears the status bar), `16px` sides, `22px` bottom. Onboarding uses `64px / 26px / 30px`.
- Closet grid: 2 columns, `24px` row gap, `18px` column gap.
- Photo aspect ratios: grid tiles `3:4`, item hero `1:1`, onboarding hero `4:3`, form photo `4:3`, builder slot thumb `56 × 72`.
- Tab bar: `8px 6px 30px` padding (the `30px` is the home-indicator inset), 1px top divider, five equal-flex items, each a 19px Lucide icon above a 10.5px uppercase condensed label with `.08em` tracking. Minimum 44px hit target.

## Navigation
Bottom tab bar, five destinations. Sub-screens are pushed inside a tab and keep that tab highlighted.

| Tab | Screens it owns |
| --- | --- |
| Closet | Closet, Item detail, Add/Edit piece, Add from a link, Shopping list, Shops |
| Outfits | Outfits, Outfit detail |
| Build | Outfit builder |
| Log | Wear log |
| You | Profile, Nudges |

Onboarding runs before the tab bar exists (no chrome). Modal layers, above everything: the add-choice sheet, the slot picker sheet, the confirm dialog, and the toast.

## Screens

### 1. Onboarding (3 steps)
Full-bleed, no tab bar. A header row on every step: a 9px accent square, "RACK" in condensed uppercase with `.14em` tracking, and `Step n / 3` in mono on the right.

- **Step 1 — welcome.** 4:3 blueprint photo frame (closet photo). H1 `38px`, line-height 1.05: "Everything you own, / on one page." Lead paragraph `15px`/1.55 at 80% text. Bottom: primary block button `46px` "Start my closet", ghost block button "Look around first" (skips straight to the closet).
- **Step 2 — first piece.** H2 `30px` "Add your first piece", help line, then a blueprint form card holding: Name, Where you bought it, What you paid, and a Kind segmented control (Tops / Bottoms / Layers / Shoes / Extra). Primary button "Add it". Validation: empty name → toast "Give it a name first."
- **Step 3 — confirmation.** 1:1 blueprint photo frame. H2 reads "The <name> is in." when a name was entered, otherwise "That's one down." Body explains the sample closet is preloaded and that changes persist on the device. Primary "Go to my closet".

### 2. Closet (home)
- Header row: kicker "Closet index", H2 `32px` "My closet". Right side, two 44px icon buttons — secondary (storefront icon) → Shops, primary (plus) → opens the **add-choice sheet**.
- Mono stat line: `12 pieces · 284 wears · $1,224 in`. The spend segment hides when prices are off.
- Search input, placeholder "Search a piece, a shop, a colour". Matches name, shop, colour, city, case-insensitive.
- Category segmented control: All / Tops / Bottoms / Layers / Shoes / Extra.
- Sort row: mono label "Sort" + segmented Recent / Worn / Value. Recent = purchase date descending; Worn = wear count descending; Value = cost per wear ascending.
- **Dormant nudge** (conditional): a blueprint card with a 3px accent left border, title "N pieces haven't been worn in three months.", sub "Tap to see which, before you buy anything new." Tapping goes to Nudges. Shows only when the Wear reminders setting is on and at least one piece qualifies.
- **Shopping list row** (conditional, when the list is non-empty): a single hairline rule row — "Shopping list", mono count, accent arrow.
- Grid of pieces. Each tile: a 3:4 blueprint photo frame, then a tappable caption block — item name in condensed `15px`, mono meta `shop · $price · N wears`, and a small accent `→`. Only the caption navigates; the photo frame is the drop target.
- Empty state when filters match nothing: blueprint card, "Nothing matches that." / "Try a shop name, or clear the filter."

### 3. Item detail
- Top row: secondary "← Closet", ghost "Edit details" right-aligned.
- 1:1 blueprint photo frame.
- H2 `30px` item name. Tag row: accent tag (category), neutral tag (colour), outline tag (`N wears`).
- Section "Where it came from" — blueprint-framed two-column table: Shop, City, Bought (`Mar 2024`), Paid, Worn (`24 times`), Last worn (`3 Jun 2026 (2mo ago)`), Product page (link, only if the piece came from a URL), Per wear (`$2.83 a wear`, in `--color-accent-700`). Paid and Per wear hide when prices are off; Per wear reads "no wears yet" at zero wears.
- Section "Wear log" — up to 6 rows, each a hairline rule with the formatted date left and relative age right. Zero wears shows "Not worn yet. It's been waiting since <month>."
- Section "In outfits" — outline tags linking to each outfit, or "Not in any outfit yet."
- Actions: primary `44px` "Wore it today" (flex), secondary "To outfit", then a ghost block "Remove from closet" → confirm dialog.

### 4. Add / Edit a piece (manual)
Secondary "← Cancel". Kicker "New piece" / "Editing", title "Add a piece" / "Edit details". 4:3 photo frame, then a blueprint form card: Name, Kind (segmented), Colour, Shop, City, Bought (month picker), Paid. Primary block button "Add to my closet" / "Save changes". Empty name blocks the save with a toast. Saving lands on the new item's detail screen.

### 5. Add from a link ← the routed flow
Kicker "From a shop", H2 "Paste the link", help "The product page from wherever you found it. We'll read what we can and you fix the rest."

- URL field, then a secondary block button "Read the link". A bad or empty URL toasts "That doesn't look like a link yet."
- **Parsing rules** (client-side URL parsing only — the prototype does not fetch the page):
  - Host: strip a leading `www`, `www2`, `m`, `shop`, `store`, `us`, `uk`.
  - Shop name: first host label, hyphens/underscores to spaces, title-cased — but upper-cased if 3 letters or fewer (so `hm` → `HM`).
  - Name: walk the URL path segments from last to first; split each on non-alphanumerics; drop stop-words (`productpage`, `product`, `item`, `dp`, `p`, `html`, locale and gender codes) and pure numbers; accept the first segment leaving 3+ letters. Some shops (H&M, Zara) put only an ID in the path — then the name comes back empty, which is expected.
  - Item reference: the first run of 6+ digits anywhere in the URL.
  - Category guess from the name: shoe/sneaker/boot/loafer/sandal/trainer → Shoes; jean/trouser/chino/skirt/short/pant/slack → Bottoms; coat/jacket/blazer/cardigan/trench/knit/sweater/parka → Layers; bag/tote/belt/scarf/cap/hat/sock → Extras; otherwise Tops.
  - City defaults to "Online"; Bought defaults to the current month.
- **"What we got"** — a blueprint card: 88 × 110 photo slot on the left; on the right the cleaned host in uppercase mono, the parsed name in condensed `18px` (or "Name it below"), and a mono status line. The status line is the honest part: with a name it reads `Item 1367068001 · check the name and price`; with no name, `Item 1367068001 — this shop hides the name in the link. Type it in.` Below that, editable Name / Shop / Price / Kind.
- **"Where does it go"** — two blueprint option cards with radio marks (`●` / `○`):
  - *Straight into my closet* — "You already own it — it starts counting wears today."
  - *Shopping list* — "Park it. The link stays so you can come back to it."
- Primary button label follows the choice: "Add to my closet" / "Save to shopping list". Closet lands on the new item's detail screen; the list lands on the Shopping list screen.

**In the real app** this should become a proper metadata fetch (Open Graph / oEmbed / JSON-LD, or a share-sheet extension) so the name, price and product image arrive filled in. The URL-parsing fallback above is what to do when that fails — and the honest status copy should stay.

### 6. Shopping list
Kicker "Not bought yet", H2 "Shopping list", primary plus icon button → Add from a link. Mono line: `2 saved · $114 if you bought it all` (total hides when prices are off).

Each entry is a blueprint card: 52 × 66 placeholder thumb, name in condensed `17px`, mono `shop · $price · saved 6d ago`, and the host as an external link with a `↗`. Two actions: primary "I bought it" (moves the entry into the closet with today's month as the purchase date, zero wears, and navigates to its detail screen) and secondary "Remove".

Empty state: "Nothing saved yet." / "Paste a shop link and park it here until you decide."

### 7. Outfit builder
Kicker "Assembly", H2 "Build an outfit", help "Fill the slots you care about. Empty ones are fine."

Five fixed slots — Layer, Top, Bottom, Shoes, Extra — each a blueprint row: 56 × 72 thumb, then a tappable text block (uppercase mono slot label, condensed `17px` piece name or "Nothing yet", `11.5px` sub of `shop · month` or "Tap to pick from <category>"), then a trailing accent glyph. The glyph is `+` when empty (opens the picker) and `×` when filled (clears the slot). Tapping the text block always opens the picker.

Then a mono "Outfit value" total (prices only), a "Call it something" name field, a primary block "Save this outfit (N pieces)" — reading "Add a piece to save" and refusing when nothing is picked — and a ghost "Clear the slots". Saving prepends the outfit and navigates to Outfits.

**Slot picker** — a bottom sheet, max 74% height, scrolling. Title "Pick a <slot>", ghost Close, then a hairline list of every piece in that category: 42 × 54 thumb, condensed name, mono `shop · $price`, accent `+`. Choosing fills the slot and closes. Empty category shows "Nothing in this category yet." Backdrop `#1d1f20` at 40%; tapping it closes.

### 8. Outfits
Kicker "Saved sets", H2 "Outfits". A list of blueprint cards, each with the outfit name in condensed `19px`, mono `4 pieces · $265`, and a row of equal-flex 3:4 placeholder thumbs beneath. Whole card taps through to the detail. Empty state, then a secondary block "Build another".

### 9. Outfit detail
Secondary "← Outfits", kicker with the piece count, H2 outfit name. A hairline list of its pieces (44 × 56 thumb, condensed name, mono meta, accent arrow) each linking to Item detail. Mono "Outfit value" row. Then: primary `46px` "Wearing this today" (logs today's wear for every piece in the set, skipping any already logged today, and toasts "Logged N pieces for today."), secondary "Open in the builder" (loads the pieces back into their slots and pre-fills the name), ghost "Delete this outfit" → confirm.

### 10. Wear log
Kicker "What you actually wore", H2 "Log", mono `284 wears logged · since 14 Sep 2025`. Then reverse-chronological days, most recent 40. Each day is a hairline block: formatted date in condensed `16px` with relative age in mono on the right, and beneath it a wrapping row of neutral tags — one per piece worn — each linking to its item.

### 11. Shops
Secondary "← Closet", kicker "Where it all came from", H2 "Shops". Pieces grouped by shop, ordered by piece count descending. Each group is a blueprint card: shop name in condensed `19px`, city in mono, and right-aligned mono `3 pieces` / `$412 total`. Beneath, a hairline row per piece — name left, `Mar 2024 · 24 wears` right — linking to Item detail.

### 12. Nudges
Kicker "Quiet nudges", H2 "Worth knowing". A hairline list; each row is a 7px accent square, then a condensed `17px` title and a `13px` body at 67%, with a mono age on the right. Rows link to the piece they are about. The four generated nudges:

1. Longest-dormant piece — "The camel trench hasn't been out in a while." / "Last worn 4 Mar 2026. Still one of the good ones." (or "Never worn since you added it.")
2. Best value — "Your white leather sneakers is down to $1.75 a wear." / "63 wears logged. Best value in the closet."
3. Dormant count, when more than one — "5 pieces are sitting unworn." / "Worth a look before you buy anything new this month."
4. Worst value — "The camel trench is your most expensive habit." / "$19.33 every time you wear it. A few more outings fixes that."

### 13. Profile ("You")
78 × 78 blueprint avatar slot beside H2 "My wardrobe" and a mono line `12 pieces · 3 outfits · 6 shops`. Then three blueprint stat cells in a row — Pieces, Wears, and either Avg / wear (prices on) or Unworn (prices off) — each a condensed `25px` number over a `9px` uppercase mono label.

"Elsewhere" section: hairline rows to Shopping list (with count), Shops & spend, Full wear log. "Settings" section: three rows, each a label, a `11.5px` sub-line, and a toggle:

- **Wear reminders** — "A nudge when something sits unworn for 90 days". Off hides the closet nudge card.
- **Show what I paid** — "Prices and cost per wear across the app". Off hides every price, spend total and cost-per-wear across all screens.
- **Weekly recap** — "Sunday summary of what you actually wore". Off by default; no prototype behaviour.

Finally a ghost "Reset to the sample closet" → confirm.

## Shared components

**Toast.** Absolutely positioned, `16px` from each side, `110px` from the bottom (clears the tab bar), z above everything. `--color-accent-900` fill, `--color-bg` text, `13.5px`, `12px 14px` padding, square. Auto-dismisses after 2.4s; a new toast replaces the current one.

**Confirm dialog.** The design system's `.dialog-backdrop` + `.dialog`, wearing the blueprint frame. Title, body, and two actions — secondary "Keep it" and primary "Delete" / "Reset". Three cases:

| Trigger | Title | Body | CTA |
| --- | --- | --- | --- |
| Remove a piece | Remove this piece? | Its wear history goes with it. This can't be undone. | Delete |
| Delete an outfit | Delete this outfit? | The pieces stay in your closet — only the set goes. | Delete |
| Reset | Start over? | Your pieces, outfits and wear log go back to the sample closet. Photos you dropped stay. | Reset |

Deleting a piece also removes it from every outfit that contained it.

**Add-choice sheet.** Bottom sheet from the closet's plus button. Title "Add a piece", ghost Close, and two blueprint option cards: "From a shop link" / "Paste the product page. Send it to your closet or park it on the shopping list." and "By hand" / "A photo and whatever details you remember."

## Data model

```
Item     { id, name, cat, colour, store, city, bought:"YYYY-MM", price:Number,
           url?:String, wears:["YYYY-MM-DD", …] }   // newest first
Outfit   { id, name, ids:[itemId] }
WishItem { id, name, cat, colour, store, city, price, url, added:"YYYY-MM-DD" }
Setting  { label, sub, on:Boolean }
```

Categories are exactly `Tops · Bottoms · Layers · Shoes · Extras` ("Extras" renders as "Extra" in controls). Builder slots map one-to-one onto those categories in the order Layer, Top, Bottom, Shoes, Extra; a piece sent from Item detail to the builder lands in the slot matching its category, falling back to Extra.

**Derived values**
- Cost per wear = `price / wears.length`, or `price` when never worn.
- Dormant = zero wears, or the most recent wear older than 90 days.
- Total spend = sum of prices; average per wear = total spend / total wears.
- Wear log days = every wear date across all items, grouped by date, newest first.

**Date formatting** — `3 Jun 2026` (`en-GB`, numeric day, short month, numeric year); months as `Mar 2024`. Relative age: `today`, `yesterday`, `<30d` → `Nd ago`, `<24mo` → `Nmo ago`, else `Ny ago`.

**Logging a wear** appends today's date, unless it's already there — logging twice in one day is a no-op that toasts "Already logged today."

**Persistence.** The prototype writes `{items, outfits, wishlist, settings, screen}` to `localStorage` under `rack.v2` on every state change, restores on mount, and never persists the onboarding or link screen as the entry point. In the real app this is local device storage (or a synced store); dropped photos persist separately, per slot id, which is why the reset dialog says photos stay.

## Interaction notes
- No page transitions or animation in the prototype. Use the platform's standard push/present transitions.
- Every list row, tag and card that navigates is a full-width tap target; keep 44px minimums.
- Interactive states come from the design system: hover tint one accent step past base, a 2px `--color-accent` focus ring at `2px` offset, and 45% opacity when disabled. Don't leave platform-default focus rings.
- Search and filters are live — no submit.
- Validation is a toast, never inline red text. The only rule is a non-empty name.

## Assets
None shipped. Every image in the prototype is a placeholder awaiting real content:

| Slot id | Where | Shape |
| --- | --- | --- |
| `rack-hero` | Onboarding step 1 | 4:3 |
| `rack-first` | Onboarding step 3 | 1:1 |
| `photo-<itemId>` | Closet tiles, item hero | 3:4 / 1:1 |
| `photo-new` / `photo-<editId>` | Add & edit form | 4:3 |
| `rack-avatar` | Profile | 78 × 78 |

Icons are Lucide at stroke-width 1.5: grid (Closet), lines (Outfits), plus (Build), clock (Log), user (You), storefront (Shops), plus (add).

## Files in this bundle

```
reference/
  Rack v2.dc.html            the whole app — read the template for structure, the class for state
  support.js                 the prototype runtime (context only; do not port)
  ios-frame.jsx              iPhone bezel used for presentation only
  image-slot.js              the drag-and-drop photo placeholder
  industry.css               copy of the design-system stylesheet
  design-system/
    styles.css               authoritative tokens and component classes
    readme.md                the Industry design system guide
    _ds_bundle.js            the design system's component bundle
```
