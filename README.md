# AutoGallery

A single-file, no-backend browser app for managing an autograph and memorabilia collection. All data is stored locally in the browser — nothing is ever sent to a server.

---

## Table of Contents

1. [Overview](#overview)
2. [Getting Started](#getting-started)
3. [Header](#header)
4. [Stats Bar](#stats-bar)
5. [Search & Controls Bar](#search--controls-bar)
6. [Grid View](#grid-view)
7. [Table View](#table-view)
8. [Add / Edit Item Modal](#add--edit-item-modal)
9. [Item Detail Modal](#item-detail-modal)
10. [Settings Modal](#settings-modal)
11. [Import & Export](#import--export)
12. [Sharing a Collection](#sharing-a-collection)
13. [View Mode](#view-mode)
14. [Light & Dark Mode](#light--dark-mode)
15. [Mobile Behavior](#mobile-behavior)
16. [Data Storage](#data-storage)
17. [Item Schema](#item-schema)
18. [Technical Notes](#technical-notes)

---

## Overview

AutoGallery is a **single HTML file** (`index.html`). There is no server, no database, no external JavaScript libraries, and no external CSS files. Everything — markup, styles, and logic — lives inside one self-contained file.

Key characteristics:

- **Zero dependencies** — runs directly from the filesystem or any static file server (e.g. `python3 -m http.server 4200`).
- **Persistent storage** — all collection data, preferences, and cached exchange rates are stored in the browser's `localStorage`.
- **No account required** — open the file, start adding items.
- **Designed for 8×10 autographed photos and similar memorabilia**, but works for any signed collectible.

---

## Getting Started

Open `index.html` in any modern browser. If you need URL-based navigation or want to load images from disk, serve it locally:

```bash
python3 -m http.server 4200
# then open http://localhost:4200
```

The app is blank on first launch. Click **+ Add Item** in the header to add your first piece.

---

## Header

The header is sticky — it stays at the top of the viewport while you scroll. It contains:

### Logo

**Auto**Gallery — "Auto" is styled in gold, "Gallery" in the default text color.

### Header Controls (right side, left to right)

| Control | Description |
|---|---|
| ⚙️ Settings | Opens the **Settings Modal** for theme, import/export, and currency options |
| **+ Add Item** | Opens the Add Item modal |

The Settings button is a 36×36px square icon button with rounded corners. On hover it brightens slightly.

The **+ Add Item** button is styled in gold with black text to make it the primary call to action.

---

## Stats Bar

Directly below the header, the stats bar shows four live aggregate statistics across the **entire collection** (not just the filtered view):

| Stat | Description |
|---|---|
| **Items** | Total number of items in the collection |
| **Total Paid** | Sum of all "Paid" values |
| **Est. Value** | Sum of all "Est. Value" values |
| **Total ROI** | Overall return on investment: `(total value − total paid) / total paid × 100`, displayed as a whole-number percentage |

- All monetary values are converted to the **display currency** using the live exchange rate (see [Settings Modal](#settings-modal)).
- If no items have both a paid amount and an estimated value, ROI shows `—`.
- Values use the display currency symbol and `toLocaleString()` formatting (e.g. commas for thousands).
- Stats are separated by vertical dividers and use gold for the value text.

---

## Search & Controls Bar

Below the stats bar is a row of controls for filtering and viewing the collection.

### Search

A text input with a magnifying-glass icon. Filters items in real time (on every keystroke) across:

- Signer name(s) — all signers are searched for multi-signer items
- Character name
- Film/show title
- Notes

The search is **case-insensitive**. Clearing the input restores the full list.

### Sort

A dropdown to sort the visible items. Options:

| Option | Sort order |
|---|---|
| Value: High → Low | Estimated value descending *(default)* |
| Value: Low → High | Estimated value ascending |
| Paid: High → Low | Amount paid descending |
| Paid: Low → High | Amount paid ascending |
| Newest First | Date added descending |
| Oldest First | Date added ascending |
| Name A → Z | Signer name alphabetically |
| Name Z → A | Signer name reverse alphabetically |

Items without a value are sorted to the end when sorting by value or paid.

### Columns (desktop only)

A dropdown to choose how many cards appear per row in grid view. Options: **3, 4, 5, 6, 7, 8**. Default is **5**. This setting is saved to `localStorage` (`ag_cols`) and restored on next visit.

### View Toggle (desktop only)

Two buttons side by side — a grid icon and a table/list icon — to switch between **Grid View** and **Table View**. The active mode is highlighted. The last-used view is saved to `localStorage` (`ag_view`). Both the columns selector and view toggle are **hidden on mobile**.

---

## Grid View

The default view. Items are displayed as cards in a responsive grid. The number of columns is controlled by the **Columns** dropdown (3–8 per row on desktop; 2 columns fixed on mobile).

### Card Layout

Each card from top to bottom:

1. **Image area** — 5:4 aspect ratio. If a photo was uploaded it fills this area with `object-fit: cover`. If no photo exists, a placeholder icon is shown. On hover, the image subtly scales up (1.03×).

2. **Card body** (below the image):
   - **Signer name** — bold, 14px. Plain text (not a link) in the grid view. For items with multiple signers, this shows **"Multiple Signers"** instead of individual names.
   - **Character name** — smaller, muted color, on its own line. Plain text.
   - **Film/show title** — smaller, muted color, on its own line below character name. Plain text.
   - A thin horizontal divider
   - **Cert** label + a gold clickable link to the authentication company's verification page (if a cert company is selected). Clicking the link **also copies the cert number to the clipboard** and shows a toast. The link click does **not** open the detail modal.
   - **Paid** label + value (or `—` if not set)
   - **Est. Value** label + value; if a value URL is set, the value is a clickable gold link that opens the source in a new tab. The link click does **not** open the detail modal.
   - **ROI** label + percentage value, green or red. Only shown when both Paid and Est. Value are set.

3. **Clicking the card** (anywhere except the links) opens the **Item Detail Modal**.

Cards animate on hover: they lift up 3px and show a deeper shadow.

---

## Table View

An alternative view available on desktop. Items are shown as rows in a sortable table with the following columns:

| Column | Content |
|---|---|
| (thumbnail) | 52×40px cropped thumbnail of the photo, or a placeholder icon |
| **Signer** | Signer name as a clickable link (Wikipedia or IMDb per the Info Links setting), or **"Multiple Signers"** for multi-signer items |
| **Character** | Character name, plain text |
| **Film / Show** | Film or show title as a clickable link (Wikipedia or IMDb per the Info Links setting) |
| **Paid** | Amount paid in display currency |
| **Est. Value** | Estimated value in display currency, colored green (positive) or red (negative) relative to paid. If a value URL is set, the value is a clickable link that opens in a new tab. |
| **ROI** | Whole-number percentage, green or red pill |
| **Cert** | Clickable gold link to the authentication company's verification page. Clicking also copies the cert number to the clipboard. |
| **Added** | Date the item was added, formatted as `MMM D, YYYY` |

- Clicking any **row** opens the **Item Detail Modal** for that item.
- Clicking the **Est. Value link**, **Cert link**, **Signer link**, or **Film link** opens the URL in a new tab without opening the detail modal (click propagation is stopped).
- Rows have a hover background highlight.

---

## Add / Edit Item Modal

Opened by clicking **+ Add Item** (new item) or **Edit** inside the detail modal (existing item). The modal slides in with a slight scale animation and a blurred backdrop.

### Form Fields

| Field | Type | Notes |
|---|---|---|
| **Signer 1** | Text input | Required. The first (or only) autograph signer's real name. |
| **+ button** | Button | Adds a Signer 2 field. Each additional signer row has a − button to remove it. There is no upper limit on the number of signers. |
| **Character** | Text input | Optional. Character name (e.g. "Darth Vader"). Applies to the item, not per signer. |
| **Film / Show** | Text input | Optional. The title of the film or show. |
| **Cert #** | Text input | Optional. The authentication certificate number. |
| **Cert Company** | Dropdown | Options: *(none)*, Beckett, PSA, JSA, SWAU. Controls the verification URL. |
| **Paid** | Number input | Optional. What you paid, in your **local currency**. |
| **Est. Value** | Number input | Optional. Current estimated market value, in your **local currency**. |
| **Value Source URL** | Text input | Optional. A URL linking to a source for the estimated value (e.g. eBay sold listing). |
| **Notes** | Textarea | Optional. Free-form notes. Resizable vertically. |
| **Photo** | File upload / drag-and-drop | Optional. See below. |

All monetary inputs (**Paid**, **Est. Value**) are stored in the **local currency** (configured in Settings). They are converted to the display currency at render time using the live exchange rate.

### Multiple Signers

The form always shows a **Signer 1** field. Click the **+** button to the right of any signer field to add the next signer. Additional signer rows (Signer 2, Signer 3, …) each have a **−** button to remove them. There is no limit to the number of signers.

**How multi-signer items are displayed:**

- **Grid and table views** — show **"Multiple Signers"** in place of a name when an item has two or more signers.
- **Detail modal** — the title shows **"Multiple Signers"**, and each signer appears as its own **Signer 1 / Signer 2 / …** row in the detail table.
- **Search** — searches across all signer names simultaneously.
- **Sorting by name** — sorts by the first signer's name.

Single-signer items (Signer 1 only) behave identically to the original behavior — the signer's name is shown normally everywhere.

### Photo Upload

The photo area is a dashed drop zone. You can:

- **Click** to open the system file picker (accepts `image/*`)
- **Drag and drop** an image file onto the zone

After an image is selected it is compressed client-side using an HTML5 Canvas:

- Maximum dimension: **900px** (width or height, whichever is larger)
- Output format: **JPEG at 0.82 quality**
- Stored as a **base64 data URL** in `localStorage`

A preview of the selected image is shown in the drop zone. A **Remove** button appears to clear the photo.

### Validation

Only **Signer 1** is required. If left blank, the field is highlighted in red and saving is blocked. Empty additional signer fields are ignored.

### Buttons

- **Cancel** — closes the modal without saving
- **Save** (Add Item modal) or **Save Changes** (edit modal) — validates and writes to `localStorage`, then re-renders the gallery

---

## Item Detail Modal

Clicking any card (grid) or row (table) opens a read-only detail view for that item.

### Contents

- **Signer name** (or **"Multiple Signers"**) as the modal title
- Photo (full width, if available). **Click the photo to open it fullscreen** — see [Fullscreen Photo Viewer](#fullscreen-photo-viewer).
- All fields displayed as label/value rows in a table:
  - **Signer** — clickable link (Wikipedia or IMDb). For multi-signer items, individual **Signer 1 / Signer 2 / …** rows, each a link.
  - Character — plain text
  - Film / Show — clickable link (Wikipedia or IMDb)
  - Cert # — clickable link to the cert company's verification page. Clicking **also copies the cert number to the clipboard** and shows a toast confirmation.
  - Paid (in display currency)
  - Est. Value (in display currency, with link if URL is set)
  - ROI — percentage, colored green or red. Only shown when both Paid and Est. Value are set.
  - Notes
  - Date Added

Fields with no value are omitted (not shown as blank rows).

### Buttons

- **Edit** — opens the Edit modal pre-populated with all current field values
- **Delete** — shows a browser `confirm()` dialog. If confirmed, removes the item from `localStorage` and re-renders. If cancelled, nothing happens.
- **✕** (top-right) — closes the modal
- Clicking the **backdrop** (outside the modal) also closes it

---

## Settings Modal

Opened by clicking the **⚙️ gear icon** in the header. The modal is organized into sections that vary depending on whether you are in normal mode or [View Mode](#view-mode).

**Normal mode sections:** Appearance, Photo Format, Image Fit, Info Links, Data, Local Currency, Display Currency, Exchange Rate

**View mode sections:** Appearance, Photo Format, Image Fit, Info Links, This Collection, Display Currency, Exchange Rate

### Appearance

A row button that toggles between light and dark mode. The label updates to reflect the opposite of the current theme ("Switch to Dark Mode" / "Switch to Light Mode"), and the icon switches between a sun and moon accordingly. See [Light & Dark Mode](#light--dark-mode) for full details.

### Photo Format

A dropdown that sets the image aspect ratio used for grid cards and table thumbnails. Choose the format that matches your memorabilia:

| Option | Aspect Ratio | Typical use |
|---|---|---|
| **8×10 Landscape** *(default)* | 5:4 | Most sports & entertainment photos |
| **8×10 Portrait** | 4:5 | Portrait-oriented photos |
| **5×7 Landscape** | 7:5 | Smaller landscape photos |
| **5×7 Portrait** | 5:7 | Smaller portrait photos |
| **4×6 Landscape** | 3:2 | Standard print landscape |
| **4×6 Portrait** | 2:3 | Standard print portrait |
| **Index Card (3×5)** | 5:3 | Signed index cards |
| **Trading Card (2.5×3.5)** | 5:7 | Sports/entertainment trading cards |
| **Square (1×1)** | 1:1 | Square-format items |

- Saved to `localStorage` (`ag_photo_format`). Takes effect instantly via a CSS custom property (`--card-img-ratio`) — no re-render needed.
- Available in both normal mode and view mode.

### Image Fit

A dropdown that controls how photos are displayed when the uploaded image doesn't match the selected Photo Format:

| Option | Behavior |
|---|---|
| **Crop to fill** *(default)* | Image is cropped to fill the frame (`object-fit: cover`) — no gaps, edges may be trimmed |
| **Fit with black bars** | Full image always visible; gaps filled with a black background (`object-fit: contain`) |
| **Fit with blurred background** | Full image always visible; gaps filled with a blurred, scaled-up copy of the same photo |

- Saved to `localStorage` (`ag_img_fit`). Takes effect instantly — no re-render needed.
- Available in both normal mode and view mode.
- The blurred background option has no effect on cards without a photo.
- Table thumbnails use `contain` (no blur) in the blurred background mode, since the thumbnail size is too small for the effect to be useful.

### Info Links

A dropdown to choose which site signer names and film/show titles link to in the table view and detail modal:

| Option | Signer links to | Film / Show links to |
|---|---|---|
| **Wikipedia** *(default)* | `en.wikipedia.org/wiki/[Name]` | `en.wikipedia.org/wiki/[Title]` |
| **IMDb** | IMDb name search (`&s=nm`) | IMDb title search (`&s=tt`) |

- The setting is saved to `localStorage` (`ag_info_link`) and takes effect immediately (gallery re-renders on change).
- Available in both normal mode and view mode.
- Links appear in **table view** (Signer column, Film / Show column) and **detail modal** (Signer row, Film / Show row). Grid card text is plain — no links.

### Data

Three row buttons for backup, restore, and sharing:

- **Export Collection** — downloads the full collection as a `.json` file. See [Import & Export](#import--export).
- **Import Collection** — opens a file picker to load a previously exported `.json` file, replacing the current collection. See [Import & Export](#import--export).
- **Copy Share Link** — uploads the collection to dpaste.com and copies a short shareable URL to the clipboard. See [Sharing a Collection](#sharing-a-collection).

> ⚠️ The Data section is hidden entirely when viewing a shared collection in [View Mode](#view-mode). It is replaced by the **This Collection** section, which contains the **Save as My Collection** button.

### Currency

#### Two-Currency System

AutoGallery separates **where you record values** from **how you display them**:

| Currency | Purpose |
|---|---|
| **Local Currency** | The currency in which you enter **Paid** and **Est. Value** amounts. Stored with the data. |
| **Display Currency** | The currency shown throughout the UI — stats bar, cards, table, detail modal. |

If local and display currencies differ, a live exchange rate is fetched from the internet and used to convert all displayed monetary values.

### Currency Options

Both dropdowns offer the same list of currencies:

`USD`, `GBP`, `EUR`, `TWD`, `JPY`, `CNY`, `AUD`, `CAD`, `HKD`, `SGD`, `KRW`

Each is shown with its symbol in the UI: `$`, `£`, `€`, `NT$`, `¥`, `¥`, `A$`, `CA$`, `HK$`, `S$`, `₩`.

### Exchange Rate

When local ≠ display currency, the app fetches a live exchange rate from:

```
https://cdn.jsdelivr.net/npm/@fawazahmed0/currency-api@latest/v1/currencies/{base}.json
```

This is a free, public API that requires no authentication.

**Caching:** The exchange rate is cached in `localStorage` (`ag_rate_cache`) for **1 hour**. This prevents excessive API calls on every page load. The cache stores: the `from` currency, the `to` currency, the `rate`, and a Unix timestamp. If the cache is older than 1 hour, or if either currency has changed since it was cached, a fresh fetch is triggered.

**Rate Status Box:** The settings modal shows a small status box indicating the current exchange rate and its state:

| State | Display |
|---|---|
| Same currency | No rate box shown (rate = 1) |
| Loading | "Fetching rate…" with a spinner |
| Loaded | "1 TWD = 0.0312 USD · updated just now" (example) |
| Error | "Could not fetch rate" with a Retry button |

A **Refresh** button allows manually re-fetching the rate at any time (bypasses the 1-hour cache).

**Behavior when currencies match:** If local and display currencies are set to the same value, no exchange rate is fetched, and `exchangeRate` is set to `1` internally. All displayed values are identical to stored values.

### Changing Currencies

Changing either currency in the settings modal:
1. Saves the selection to `localStorage` immediately
2. Clears any cached exchange rate (to prevent stale rates when the currency pair changes)
3. Triggers a fresh `fetchExchangeRate()` call
4. Re-renders the gallery with the new rate once it arrives

---

## Sharing a Collection

AutoGallery can generate a short URL that lets anyone view your collection in their own browser — no account, no app install required.

### How to Share

1. Open the **Settings Modal** (⚙️ in the header)
2. Under **Data**, click **Copy Share Link**
3. The button label changes to "Uploading…" while the collection is sent to dpaste.com
4. Once uploaded, a short URL is copied to your clipboard automatically — e.g.:
   ```
   http://localhost:4200/index.html#blob=G9GW3KB53
   ```
5. Send that URL to anyone. When they open it, AutoGallery loads and immediately displays your collection in read-only [View Mode](#view-mode)

### What Gets Uploaded

The **entire collection** is included — all item fields and all photos. The JSON is uploaded as a single paste to [dpaste.com](https://dpaste.com), a free public paste service.

> ⚠️ **Your collection data is publicly accessible.** Anyone who has or guesses the URL can read all item data including photos, values, cert numbers, and notes. Do not share if your collection data is sensitive.

### How the URL Works

The URL contains only a short blob ID in the hash fragment (`#blob=<id>`). The hash is never sent to the server hosting AutoGallery — it is processed entirely in the browser. When someone opens the URL:

1. The app detects `#blob=<id>` in the URL hash on page load
2. It immediately shows a full-page loading spinner ("Loading shared collection…")
3. It fetches the paste from `https://dpaste.com/<id>.txt`
4. The JSON is parsed and the collection is displayed in [View Mode](#view-mode)
5. The hash is stripped from the URL (`history.replaceState`) so refreshing the page does not re-trigger the load

### Loading in the Same Tab

If AutoGallery is already open and you paste a share URL into the browser's address bar (changing only the hash), the app detects the `hashchange` event and loads the shared collection without a full page reload.

### Paste Expiry

dpaste.com pastes created by AutoGallery are set to expire after **365 days**. After that, the share URL will stop working. There is no way to renew a paste — generate a new share link if needed.

### Error Handling

- If the upload fails (e.g. no internet connection), a toast appears: "⚠️ Upload failed — check your connection". The button returns to its normal state and nothing is copied to the clipboard.
- If a share URL is opened but the paste has expired or the ID is invalid, a toast appears: "⚠️ Could not load shared collection — link may be invalid".

---

## View Mode

View mode is a read-only state that activates when opening a share link. It lets you browse someone else's collection without affecting your own data.

### Entering View Mode

View mode is entered automatically when a `#blob=<id>` URL hash is detected — either on page load or when the hash changes in the current tab. It is never entered manually.

### Visual Indicator

A gold banner appears directly below the header:

> 👁 Viewing a shared collection — your own collection is untouched

The banner contains a **← Back to my collection** button on the right side.

### What Is Disabled in View Mode

| Feature | Status |
|---|---|
| **+ Add Item** button | Hidden |
| **Edit** button in detail modal | Hidden |
| **Delete** button in detail modal | Hidden |
| **Settings → Data** section (Export, Import, Share) | Hidden |
| **Settings → Local Currency** | Hidden |
| Saving to localStorage | Never happens |

### What Remains Available

| Feature | Status |
|---|---|
| Search and sort | ✅ Fully functional |
| Grid / Table view toggle | ✅ Fully functional |
| Columns per row selector | ✅ Fully functional |
| Item detail modal (read-only) | ✅ Opens, no edit/delete |
| **Settings → This Collection → Save as My Collection** | ✅ See below |
| **Settings → Display Currency** | ✅ Change how values are shown |
| **Settings → Appearance** (theme toggle) | ✅ Light/dark mode still works |
| Exchange rate conversion | ✅ Works against the shared collection's local currency |

### Saving a Shared Collection as Your Own

While in view mode, the Settings modal shows a **This Collection** section with a **Save as My Collection** button. This lets you adopt the shared collection as your own, replacing whatever you currently have saved.

**What happens when you click it:**

1. A confirmation dialog appears: *"Replace your own collection with these N items? This cannot be undone."*
2. If confirmed:
   - The shared items (including all photos) are written to `localStorage`
   - The sharer's local currency is set as your new local currency
   - View mode exits and the full editing UI is restored (Add Item button, Edit/Delete in detail modal, all Settings sections)
   - The exchange rate is re-fetched for your currency pair
   - A toast confirms: *"Saved N items as your collection"*
3. If cancelled, nothing changes.

> ⚠️ This permanently overwrites your existing collection. There is no undo. Export your current collection first if you want to keep it.

### Currency in View Mode

The shared collection was recorded in the sharer's local currency (stored in the JSON payload). When viewing, that currency is used as the local currency for exchange-rate conversion — so if the sharer used TWD and you switch your display currency to USD, the values convert correctly. Your own local currency setting in localStorage is not changed.

### Your Own Data is Untouched

View mode never writes to `localStorage`. Your own collection remains exactly as it was. Switching back via **← Back to my collection** immediately reloads your own data from localStorage and restores the full editing UI.

### Exiting View Mode

Click **← Back to my collection** in the gold banner. This:
1. Clears the view mode state
2. Reloads your own items from `localStorage`
3. Restores the Add Item button and all edit/delete controls
4. Hides the banner
5. Re-fetches the exchange rate for your own currency pair

---

## Light & Dark Mode

The theme toggle lives inside the **Settings Modal** under the Appearance section. It is a full-width row button that switches between light and dark mode. The button label reads "Switch to Dark Mode" when light mode is active, and "Switch to Light Mode" when dark mode is active. The icon changes between a sun (☀️) and moon (🌙) to match.

**Default:** Light mode.

**How it works:** A `data-theme="light"` attribute is set on the `<html>` element for light mode. When dark mode is active, the attribute is removed. CSS custom properties (variables) defined in `:root` apply the dark theme, and a `[data-theme="light"]` block overrides them for light mode.

**Persistence:** The chosen theme is saved as `ag_theme` in `localStorage` and restored on next visit.

### Color Palette

| Variable | Dark mode | Light mode |
|---|---|---|
| `--bg` | `#0c0c10` | `#f0f0f5` |
| `--surface` | `#16161e` | `#ffffff` |
| `--card` | `#1d1d27` | `#ffffff` |
| `--gold` | `#c9a435` | `#a07810` |
| `--text` | `#e6e6f0` | `#18182a` |
| `--sub` | `#9090a8` | `#70708a` |
| `--green` | `#4caf84` | `#1e7a52` |
| `--red` | `#e05555` | `#c03030` |

---

## Import & Export

### Export

Click **Export Collection** in the Settings Modal (⚙️ → Data). The browser downloads a `.json` file named with today's date (e.g. `autogallery-2024-11-15.json`).

**File format:**

```json
{
  "currency": "TWD",
  "items": [
    {
      "id": "abc123",
      "added": "2024-11-15T08:30:00.000Z",
      "name": "Harrison Ford",
      "signers": ["Harrison Ford"],
      "character": "Han Solo",
      "film": "Star Wars",
      "certNum": "A1234567",
      "certCompany": "beckett",
      "paid": 15000,
      "value": 22000,
      "valueUrl": "https://www.ebay.com/...",
      "notes": "Signed at Fan Expo 2023",
      "img": "data:image/jpeg;base64,/9j/..."
    }
  ]
}
```

- `currency` is the **local currency** at time of export.
- All monetary values in `items` are in that local currency.
- `img` is the full base64-encoded JPEG string, or `null` if no photo.

### Import

Click **Import Collection** in the Settings Modal (⚙️ → Data). A file picker opens. Select a previously exported `.json` file.

**Import behavior:**

- The imported `currency` field is set as the new **local currency**.
- All items are loaded into `localStorage`, **replacing** the current collection entirely.
- The gallery re-renders immediately with the imported data.
- If the file is malformed or cannot be parsed, a browser `alert()` is shown.

> ⚠️ Import **overwrites** your current data. Export a backup before importing if you want to preserve your existing collection.

---

## Mobile Behavior

The app is designed to work on small screens. On viewports **640px wide or narrower**:

- **Grid/Table toggle is hidden** — table view is not available on mobile.
- **Columns selector is hidden** — the column count is not adjustable on mobile.
- **Grid is forced** — even if `ag_view` is set to `"table"` in localStorage, the app renders grid view. This is enforced both in CSS (with `!important` overrides hiding `.table-wrap` and showing `.gallery`) and in JavaScript (a `isMobile` check in the render function).
- **Grid is 2 columns** — a CSS media query sets `--cols: 2` on mobile, overriding any saved column preference.
- **Cards go full width** — the text (signer name, character, film) spans the full card width rather than being cramped next to a fixed-width image column.
- **Horizontal scroll is prevented** — all layout uses `box-sizing: border-box` and the gallery padding is reduced to `12px` on mobile to prevent content from overflowing the viewport.

---

## Data Storage

All data is stored in the browser's `localStorage`. The following keys are used:

| Key | Type | Contents |
|---|---|---|
| `autogallery_v2` | JSON string | Array of all item objects |
| `ag_currency` | String | Local currency code (e.g. `"TWD"`) |
| `ag_display_currency` | String | Display currency code (e.g. `"USD"`) |
| `ag_rate_cache` | JSON string | Cached exchange rate: `{from, to, rate, ts}` |
| `ag_cols` | String | Saved columns per row (e.g. `"5"`) |
| `ag_view` | String | Saved view mode: `"grid"` or `"table"` |
| `ag_theme` | String | Saved theme: `"light"` or `"dark"` |

**Storage limits:** `localStorage` is typically limited to **5–10 MB** per origin. Because item photos are stored as base64-encoded JPEGs (compressed to max 900px at 0.82 quality), each photo takes roughly 100–400 KB. A collection of 20–50 items with photos is well within typical limits, but very large collections with many high-detail photos could approach the limit. If storage is full, the browser will throw an error and the item will not be saved.

---

## Item Schema

Each item is a JavaScript object with the following fields:

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique identifier generated with `Date.now() + Math.random()` at creation time |
| `added` | string | ISO 8601 timestamp of when the item was first saved (e.g. `"2024-11-15T08:30:00.000Z"`) |
| `name` | string | First (or only) signer's name. Always set — used as the sort key. |
| `signers` | string[] | Array of all signer names. Always present; single-signer items have `["Name"]`. Multi-signer items have two or more entries. |
| `character` | string | Character name (optional, may be empty string) |
| `film` | string | Film or show title (optional, may be empty string) |
| `certNum` | string | Certificate number (optional, may be empty string) |
| `certCompany` | string | One of: `"beckett"`, `"psa"`, `"jsa"`, `"swau"`, or `""` (none) |
| `paid` | number \| null | Amount paid, in the local currency |
| `value` | number \| null | Estimated current value, in the local currency |
| `valueUrl` | string | URL to a value source (optional, may be empty string) |
| `notes` | string | Free-form notes (optional, may be empty string) |
| `img` | string \| null | Base64 JPEG data URL, or `null` if no photo |

---

## Technical Notes

### No External Dependencies

The app uses only:
- Native browser APIs (`localStorage`, `fetch`, `FileReader`, `HTMLCanvasElement`)
- Vanilla JavaScript (ES2017+, using `async/await`)
- Inline CSS with custom properties

### Certification URL Derivation

Cert company codes map to verification URLs as follows:

| Code | Company | Verification URL |
|---|---|---|
| `beckett` | Beckett | `https://www.beckett-authentication.com/verify-certificate` |
| `psa` | PSA | `https://www.psacard.com/cert` |
| `jsa` | JSA | `https://www.spenceloa.com/verify-authenticity` |
| `swau` | SWAU | `https://auth.swau.com/pages/verify-hologram` |

The URL is derived at render time from `item.certCompany`. No URL is stored in the item data — only the company code.

### ROI Calculation

```
ROI = Math.round((value - paid) / paid * 100)
```

Result is a whole number (no decimal places). Displayed as `+47%` or `-12%`. If either `paid` or `value` is missing, ROI is not shown.

### Currency Conversion

All monetary values are stored in the local currency. At render time, every value is multiplied by `exchangeRate` (a module-level variable):

```javascript
const converted = Math.round(Number(n) * exchangeRate);
```

The result is formatted with `toLocaleString()` for thousands separators and prefixed with the display currency symbol.

### Image Compression Pipeline

```
User picks file
  → FileReader reads as Data URL
  → Image loaded into <img> element
  → Canvas resized to max 900px on longest side
  → Canvas.toDataURL('image/jpeg', 0.82) produces compressed base64
  → Stored in item.img
```

### Modal Behavior

Modals use a two-class system:
- `.overlay` — the full-screen backdrop (always in DOM, `opacity: 0`, `pointer-events: none`)
- `.overlay.open` — adds `opacity: 1` and `pointer-events: all`, animates the inner `.modal` from a slightly translated/scaled state to its natural position

Closing can happen via: the ✕ button, a Cancel button, a backdrop click (on the overlay itself, not the modal), or Delete confirmation.

### Sorting Implementation

Sort is applied after search filtering. Items without a value (`null`) are sorted to the bottom when sorting by value or paid. Sort state is not persisted — it resets to "Value: High → Low" on page load.

### Click Propagation

Links inside cards and table rows (Est. Value link, Cert link) call `event.stopPropagation()` so that clicking them does not bubble up to the card/row click handler that would otherwise open the detail modal.

### Share Link Upload (dpaste.com)

Sharing uses the dpaste.com API:

```
POST https://dpaste.com/api/v2/
Content-Type: multipart/form-data
Body: content=<JSON>, syntax=text, expiry_days=365

Response 201: https://dpaste.com/<id>\n  (plain text)
```

The ID is extracted from the response URL (`url.trim().split('/').pop()`). The share URL is then built as `<page-url>#blob=<id>`.

When loading a share URL, the raw content is fetched via:

```
GET https://dpaste.com/<id>.txt
```

The `.txt` suffix returns raw plain text (the original JSON), bypassing dpaste's HTML wrapper page. dpaste.com sets `Access-Control-Allow-Origin: *` on both endpoints, making both POST and GET work from any browser origin without a CORS proxy.

### View Mode State

View mode is controlled by two module-level variables:

```javascript
let viewMode = false;          // true when viewing a shared collection
let viewLocalCurrency = null;  // the sharer's local currency, used for conversion
```

`getLocalCurrency()` returns `viewLocalCurrency` when `viewMode` is true, transparently making the exchange rate system use the sharer's currency without touching `localStorage`. When `exitViewMode()` is called, both variables are reset and `load()` restores items from localStorage.

### hashchange Listener

```javascript
window.addEventListener('hashchange', checkShareURL);
```

This single listener means share URLs work whether the page is freshly loaded or already open. When the user pastes a share URL into the address bar while AutoGallery is already running, the browser fires `hashchange` and `checkShareURL` handles it identically to the initial page load case. The hash is immediately cleared via `history.replaceState` so the loaded state is not tied to the URL.
