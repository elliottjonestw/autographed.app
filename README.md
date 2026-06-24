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
10. [Fullscreen Photo Viewer](#fullscreen-photo-viewer)
11. [Presentation Mode](#presentation-mode)
12. [Settings Modal](#settings-modal)
13. [Import & Export](#import--export)
14. [Print Layout](#print-layout)
15. [Sharing a Collection](#sharing-a-collection)
16. [View Mode](#view-mode)
17. [Light & Dark Mode](#light--dark-mode)
18. [Mobile Behavior](#mobile-behavior)
19. [Data Storage](#data-storage)
20. [Item Schema](#item-schema)
21. [Technical Notes](#technical-notes)

---

## Overview

AutoGallery is a **single HTML file** (`index.html`). There is no server, no database, no external JavaScript libraries, and no external CSS files. Everything — markup, styles, and logic — lives inside one self-contained file.

Key characteristics:

- **Zero dependencies** — runs directly from the filesystem or any static file server (e.g. `python3 -m http.server 4200`).
- **Persistent storage** — item metadata, preferences, and cached exchange rates are stored in the browser's `localStorage`; photos are stored in **IndexedDB** so large collections aren't capped by the small `localStorage` quota.
- **No account required** — open the file, start adding items.
- **Designed for 8×10 autographed photos and similar memorabilia**, but works for any signed collectible.

---

## Getting Started

Open `index.html` in any modern browser. If you need URL-based navigation or want to load images from disk, serve it locally:

```bash
python3 -m http.server 4200
# then open http://localhost:4200
```

The app is blank on first launch. The empty state shows two buttons: **+ Add Item** to begin building your own collection, and **Load Demo Collection** to instantly populate the gallery with a sample set of items so you can explore the app before adding your own data.

---

## Header

The header is sticky — it stays at the top of the viewport while you scroll. It contains:

### Logo

**Auto**Gallery — "Auto" is styled in gold, "Gallery" in the default text color.

### Header Controls (right side, left to right)

| Control | Description |
|---|---|
| 👁 Eye (privacy) | Visible only when Privacy Mode is enabled. Clicking it turns Privacy Mode off and hides the button. |
| ▶ Play | Opens **Presentation Mode** — a full-screen photo slideshow for displaying the collection on a TV or monitor |
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
- When **Privacy Mode** is on, the Total Paid and Est. Value stats are blurred. The Return (ROI) stat is always visible.

---

## Search & Controls Bar

Below the stats bar is a row of controls for filtering and viewing the collection.

### Search

A text input with a magnifying-glass icon. Filters items in real time (on every keystroke) across:

- Signer name(s) — all signers are searched for multi-signer items
- Detail 1 (e.g. Character, Player, Artist, Author — depends on item type)
- Detail 2 (e.g. Film/Show, Team, Album, Title — depends on item type)
- Tags
- Notes

The search is **case-insensitive**. Clearing the input restores the full list.

### Filter

A **Filter** button (funnel icon) opens the Filter modal for structured, multi-select filtering. When one or more filters are active, the button turns gold and shows a count badge indicating how many filters are applied.

#### Filter Modal

The modal shows sections populated from values that actually exist in the current collection (empty sections are hidden):

| Section | Description |
|---|---|
| **Tags** | All tags used across the collection |
| **Item Type** | All item types in the collection |
| **Condition** | Condition grades, sorted from best to worst (Mint → Near Mint → Excellent → Very Good → Good → Fair → Poor) |
| **Cert Company** | Authentication companies used (Beckett, PSA, JSA, SWAU) |
| **Acquisition Method** | All acquisition methods in the collection |
| **Signing Event** | All signing events or venues in the collection |
| **Location** | All physical storage locations in the collection |
| **Detail 2** | All Detail 2 values in the collection (film/show titles, team names, album titles, etc.) |
| **Signer** | All signer names in the collection |

Each option is a toggleable chip. Click a chip to select it (turns gold); click again to deselect. **Multiple chips can be selected simultaneously.**

**Filter logic:**
- Within a category — **OR**: an item matches if it has *any* of the selected values (e.g. tag "sci-fi" OR tag "horror")
- Across categories — **AND**: an item must satisfy *all* active categories (e.g. tag "sci-fi" AND cert company "Beckett")

**Buttons:**
- **Clear All** — removes all active filters and re-renders the full list
- **Done** — closes the modal (filters remain active)

Filters and search work together — both are applied simultaneously.

### Sort

A dropdown to sort the visible items. Options:

| Option | Sort order |
|---|---|
| Newest first | Date added descending |
| Oldest first | Date added ascending |
| Name A–Z | Signer name alphabetically |
| Value: High–Low | Estimated value descending *(default)* |
| Paid: High–Low | Amount paid descending |
| Best ROI | ROI percentage descending |
| Custom Order | User-defined order (see below) |

Items without a value are sorted to the end when sorting by value or paid.

The selected sort is saved to `localStorage` (`ag_sort`) and restored on page reload.

#### Custom Order

When **Custom Order** is selected, items are displayed in a manually arranged sequence. While in this mode, hovering over any card reveals two small reorder buttons in the top-right corner of the card:

- **+** — move this item one position earlier (towards the front)
- **−** — move this item one position later (towards the back)

The custom sequence is saved automatically to `localStorage` (`ag_custom_order`) after every move. Switching away from Custom Order and back restores the last saved arrangement. New items added to the collection appear at the end of the custom order by default.

> Custom Order is **not available on mobile** — the option is hidden and disabled on viewports ≤ 640px wide. If Custom Order was previously saved as the active sort, it falls back to Newest First on mobile.

### Columns (desktop only)

A dropdown to choose how many cards appear per row in grid view. Options: **3, 4, 5, 6, 7, 8**. Default is **5**. This setting is saved to `localStorage` (`ag_cols`) and restored on next visit.

### View Toggle (desktop only)

Two buttons side by side — a grid icon and a table/list icon — to switch between **Grid View** and **Table View**. The active mode is highlighted. The last-used view is saved to `localStorage` (`ag_view`). Both the columns selector and view toggle are **hidden on mobile**.

---

## Grid View

The default view. Items are displayed as cards in a responsive grid. The number of columns is controlled by the **Columns** dropdown (3–8 per row on desktop; 2 columns fixed on mobile).

### Card Layout

Each card from top to bottom:

1. **Image area** — 5:4 aspect ratio. The first (cover) photo fills this area with `object-fit: cover`. If no photos exist, a placeholder icon is shown. On hover, the image subtly scales up (1.03×). If the item has more than one photo, a small badge in the bottom-right corner shows the total count (e.g. `3`).

2. **Card body** (below the image):
   - **Signer name** — bold, 14px. Plain text (not a link) in the grid view. For items with multiple signers, this shows **"Multiple Signers"** instead of individual names.
   - **Detail 1** (e.g. character name, player name) — smaller, muted color, on its own line. Plain text.
   - **Detail 2** (e.g. film/show title, team name) — smaller, muted color, on its own line below Detail 1. Plain text.
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
| **Detail 1** | First contextual field value (e.g. character name, player name, artist, author) — plain text |
| **Detail 2** | Second contextual field value (e.g. film/show, team, album, title) — clickable link (Wikipedia or IMDb per the Info Links setting) |
| **Paid** | Amount paid in display currency |
| **Est. Value** | Estimated value in display currency, colored green (positive) or red (negative) relative to paid. If a value URL is set, the value is a clickable link that opens in a new tab. |
| **ROI** | Whole-number percentage, green or red pill |
| **Cert** | Clickable gold link to the authentication company's verification page. Clicking also copies the cert number to the clipboard. |
| **Added** | Date the item was added, formatted as `MMM D, YYYY` |

- Clicking any **row** opens the **Item Detail Modal** for that item.
- Clicking the **Est. Value link**, **Cert link**, **Signer link**, or **Detail 2 link** opens the URL in a new tab without opening the detail modal (click propagation is stopped).
- Rows have a hover background highlight.
- When the collection is empty or a search/filter returns no results, the table body shows the same empty state as the grid view: an icon, a title, a subtitle, and (on a completely empty collection) the **+ Add Item** and **Load Demo Collection** buttons.

---

## Add / Edit Item Modal

Opened by clicking **+ Add Item** (new item) or **Edit** inside the detail modal (existing item). The modal slides in with a slight scale animation and a blurred backdrop.

### Form Fields

| Field | Type | Notes |
|---|---|---|
| **Photos** | File upload / drag-and-drop | Optional. See [Photo Upload](#photo-upload) below. |
| **Item Type** | Dropdown | Optional. The kind of memorabilia. Choose from a predefined grouped list (see [Item Types](#item-types) below). Defaults to **Photo** for new items. Filterable. |
| **Detail 1** | Text input | Optional. First contextual field — label and placeholder change based on Item Type (e.g. "Character" for film/TV items, "Player" for sports items, "Artist" for music items). |
| **Detail 2** | Text input | Optional. Second contextual field — label and placeholder change based on Item Type (e.g. "Film / Show", "Team", "Album"). The value is used as a Wikipedia or IMDb link in the table and detail views. |
| **Signer 1** | Text input | Required. The first (or only) autograph signer's real name. |
| **+ button** | Button | Adds a Signer 2 field. Each additional signer row has a − button to remove it. There is no upper limit on the number of signers. |
| **Cert #** | Text input | Optional. The authentication certificate number. |
| **Cert Company** | Dropdown | Options: *(none)*, Beckett, PSA, JSA, SWAU. Controls the verification URL. |
| **Paid** | Number input | Optional. What you paid, in your **local currency**. |
| **Est. Value** | Number input | Optional. Current estimated market value, in your **local currency**. |
| **Value Source URL** | Text input | Optional. A URL linking to a source for the estimated value (e.g. eBay sold listing). |
| **Notes** | Textarea | Optional. Free-form notes. Resizable vertically. |
| **Tags** | Tag input | Optional. Type a tag and press **Enter** or **,** to add it as a chip. Press **Backspace** on an empty input to remove the last tag. Click **×** on a chip to remove it. Multiple tags per item are supported. As you type, a dropdown suggests matching tags already used elsewhere in your collection — click a suggestion or navigate with **↑ ↓** and press **Enter** to select, **Escape** to dismiss. |

### Advanced Fields

Below the Tags field is an **▶ Advanced Fields** expander (`<details>` element). Clicking it reveals five additional optional fields:

| Field | Type | Notes |
|---|---|---|
| **Signing Event / Venue** | Text input | Optional. The event or venue where the item was signed (e.g. "Fan Expo 2024", "Private signing"). Filterable. |
| **Signing Date** | Date picker | Optional. The date the item was signed. Shown in the detail modal as a formatted date (e.g. "June 15, 2024"). Not filterable. |
| **Acquisition Method** | Text input | Optional. How you obtained the item (e.g. "In person", "Mail-in", "eBay"). Filterable. |
| **Condition** | Dropdown | Optional. Grades: *(none)*, Mint, Near Mint, Excellent, Very Good, Good, Fair, Poor. Filterable; conditions are displayed in grade order (best to worst) in the filter modal. |
| **Physical Location** | Text input | Optional. Where the item is physically stored (e.g. "Display case", "Storage box #3"). Filterable. |

When any advanced field has a value and you open the **Edit** modal, the Advanced Fields section automatically expands to show those values. Otherwise it starts collapsed.

All monetary inputs (**Paid**, **Est. Value**) are stored in the **local currency** (configured in Settings). They are converted to the display currency at render time using the live exchange rate.

### Multiple Signers

The form always shows a **Signer 1** field. Click the **+** button to the right of any signer field to add the next signer. Additional signer rows (Signer 2, Signer 3, …) each have a **−** button to remove them. There is no limit to the number of signers.

**How multi-signer items are displayed:**

- **Grid and table views** — show **"Multiple Signers"** in place of a name when an item has two or more signers.
- **Detail modal** — the title shows **"Multiple Signers"**, and each signer appears as its own **Signer 1 / Signer 2 / …** row in the detail table.
- **Search** — searches across all signer names simultaneously.
- **Sorting by name** — sorts by the first signer's name.

Single-signer items (Signer 1 only) behave identically to the original behavior — the signer's name is shown normally everywhere.

### Item Types

The **Item Type** dropdown at the top of the form contains a predefined grouped list. Selecting a type automatically relabels the **Detail 1** and **Detail 2** fields with labels and placeholder examples appropriate for that type. Leaving Item Type blank shows no Detail 1 / Detail 2 fields by default.

| Category | Item Types | Detail 1 label | Detail 2 label |
|---|---|---|---|
| **Film & TV** | Photo, Poster, Prop, Script, Funko Pop, Lithograph, Program, Trading Card¹ | Character | Film / Show |
| **Sports** | Jersey, Baseball, Bat, Helmet, Mini Helmet, Football, Basketball, Hockey Puck, Trading Card, Glove, Shoe / Cleat | Player | Team |
| **Music** | Album / Record, CD, Vinyl | Artist | Album |
| **Music** | Instrument | Artist | Band / Group |
| **Music** | Setlist | Artist | Tour / Show |
| **Books & Literature** | Book, Comic | Author | Title |
| **Other** | Magazine | Subject | Publication |
| **Other** | Canvas | Subject | Title |
| **Other** | Other | Detail 1 | Detail 2 |

> **Default:** New items default to **Photo** (Film & TV), showing the Character and Film / Show labels.

> ¹ The Film & TV **Trading Card** type is stored internally as `"Film / TV Card"` to distinguish it from the Sports **Trading Card** (stored as `"Trading Card"`). In the UI it is always displayed as "Trading Card" (English) or "交換卡" (Traditional Chinese). In the filter modal, both types share a single **Trading Card** chip — selecting it matches items of either stored value.

The Detail 1 and Detail 2 labels also update instantly when you switch languages — they are always displayed in the current language.

### Photo Upload

Items support **multiple photos**. The **Photos** section of the form shows a row of thumbnails for all uploaded photos, followed by an **Add photo** tile.

- **Click "Add photo"** to open the system file picker (accepts `image/*`; you can select multiple files at once)
- Each selected file is compressed client-side and added to the thumbnail row

Each thumbnail has an **✕** button to remove that individual photo. The first photo in the row is the **cover image** shown on the card and in the table view. There is no limit on the number of photos per item beyond IndexedDB capacity.

**Compression per image:**

- Maximum dimension: **900px** (width or height, whichever is larger)
- Output format: **JPEG at 0.82 quality**
- Stored as a **base64 data URL** in the item's `imgs` array, persisted to **IndexedDB** (see [Data Storage](#data-storage))

### Validation

Only **Signer 1** is required. If left blank, the field is highlighted in red and saving is blocked. Empty additional signer fields are ignored.

### Buttons

- **Cancel** — closes the modal without saving
- **Save** (Add Item modal) or **Save Changes** (edit modal) — validates and persists the item (metadata to `localStorage`, photos to IndexedDB), then re-renders the gallery

---

## Item Detail Modal

Clicking any card (grid) or row (table) opens a read-only detail view for that item.

### Contents

- **Signer name** (or **"Multiple Signers"**) as the modal title
- **Main photo** (full width, if available). **Click the photo to open it fullscreen** — see [Fullscreen Photo Viewer](#fullscreen-photo-viewer).
- **Photo thumbnail strip** — if the item has more than one photo, a row of 52×52 thumbnail chips appears directly below the main photo. Clicking a thumbnail swaps it into the main photo position. The active thumbnail is highlighted with a gold border.
- All fields displayed as label/value rows in a table:
  - **Signer** — clickable link (Wikipedia or IMDb). For multi-signer items, individual **Signer 1 / Signer 2 / …** rows, each a link.
  - **Detail 1** — plain text; label reflects the item type (e.g. "Character", "Player", "Artist", "Author")
  - **Detail 2** — clickable link (Wikipedia or IMDb); label reflects the item type (e.g. "Film / Show", "Team", "Album")
  - Item Type — plain text (only shown if set)
  - Condition — plain text (only shown if set)
  - Cert # — clickable link to the cert company's verification page. Clicking **also copies the cert number to the clipboard** and shows a toast confirmation.
  - Paid (in display currency)
  - Est. Value (in display currency, with link if URL is set)
  - ROI — percentage, colored green or red. Only shown when both Paid and Est. Value are set.
  - Signing Date — formatted as "Month D, YYYY" (only shown if set)
  - Signing Event / Venue — plain text (only shown if set)
  - Acquisition Method — plain text (only shown if set)
  - Physical Location — plain text (only shown if set)
  - Tags — clickable chips. Clicking a tag closes the modal and filters the gallery to items with that tag.
  - **Notes** — displayed under a "NOTES" section label, in a lightly styled box. Only shown if notes are set.
  - Date Added
- **Description** — an auto-generated text block, separated by a divider. See [Description](#description) below.
- **Date Added** — shown below the Description section.

Fields with no value are omitted (not shown as blank rows).

### Description

A **Description** section appears at the bottom of every detail modal. It auto-generates a plain-text description from the item's fields, formatted for pasting into a listing or elsewhere.

**Format:**

> *[Signer(s)] ([Detail 1] in [Detail 2]) signed [item type]. Professionally authenticated by [Cert Company] (Certificate #[Cert #]). The [item type] is in [condition] condition. The [item type] was signed at [Signing Event] on [Date Signed].*

Rules:
- The signer name and Detail 1 / Detail 2 parenthetical open the description. For multi-signer items the parenthetical is omitted.
- Item type is always lowercase (e.g. "photo", not "Photo").
- The cert sentence is included only if a cert number is set. The condition sentence is included only if condition is set. The signing sentence is included only if a signing event or signing date is set — it uses whichever of the two are available.
- If Notes are set they are appended at the end.
- Paid, Est. Value, ROI, and tags are never included.

The text is editable directly in the textarea before copying — changes are local to the open modal and not saved to the item. Click **Copy** to copy the current text to the clipboard; the button briefly reads "Copied!" to confirm.

### Buttons

- **Edit** — opens the Edit modal pre-populated with all current field values
- **Delete** — shows a browser `confirm()` dialog. If confirmed, removes the item from storage (metadata from `localStorage`, photos from IndexedDB) and re-renders. If cancelled, nothing happens.
- **✕** (top-right) — closes the modal
- Clicking the **backdrop** (outside the modal) also closes it

---

## Fullscreen Photo Viewer

Clicking the main photo in the detail modal opens a fullscreen overlay showing the image at maximum size against a dark background.

### Navigation

If the item has multiple photos, **‹** and **›** buttons appear on the left and right edges. A **"2 / 4"** counter at the bottom center shows the current position. The thumbnail strip in the detail modal stays in sync as you navigate.

**Keyboard shortcuts:**

| Key | Action |
|---|---|
| `←` | Previous photo |
| `→` | Next photo |
| `Escape` | Close viewer |

### Pan & Zoom

- **Scroll wheel** — zoom in/out, centered on the cursor position
- **Click and drag** — pan when zoomed in
- Navigating to a different photo resets zoom and position to 1× centered

---

## Presentation Mode

Clicking the **▶ play button** in the header launches a full-screen photo slideshow — designed to display the collection on a TV or monitor.

### What it shows

Only items that have at least one photo are included. The **first (cover) photo** of each item is shown — additional photos for an item are skipped. Items are shown in the same order as the current gallery view (respecting any active sort or filters).

### Layout

Each slide fills the entire screen against a black background (`object-fit: contain` — no cropping). A caption overlay at the bottom shows:

- **Signer name** — large, bold white text. Shows "Multiple Signers" for items with more than one signer.
- **Detail 1 · Detail 2** — smaller, dimmed text below the name (e.g. "Han Solo · Star Wars" or "Derek Jeter · New York Yankees"). Omitted if neither field is set.

A thin **gold progress bar** runs along the very bottom edge of the screen and animates from left to right over the 5-second display duration, then resets for the next slide.

### Timing & transitions

- Each slide is displayed for **5 seconds**.
- Slides transition with a **crossfade** — the current image fades out over 0.6 seconds before the next fades in.
- After the last item, the slideshow loops back to the first.

### Exiting

| Action | Result |
|---|---|
| Click anywhere | Closes presentation |
| `Escape` | Closes presentation |
| `Space` | Closes presentation |

If no items in the collection have photos, a toast message appears instead of entering presentation mode.

---

## Settings Modal

Opened by clicking the **⚙️ gear icon** in the header. Settings are organized into three tabs:

| Tab | Contents |
|---|---|
| **Display** | Language, Appearance, Photo Format, Image Fit, Info Links, Privacy Mode |
| **Data** | Backup & Share (normal mode) or Save as My Collection (view mode) |
| **Currency** | Local Currency, Display Currency, Exchange Rate (Local Currency hidden in view mode) |

The modal always opens on the **Display** tab.

### Tooltip Buttons

Every setting has a small italic **i** circle button at the right end of its control row (dropdown or action button). Clicking or tapping it toggles an explanation panel directly below. Only one tip is open at a time — opening a new one closes the previous one.

---

### Display Tab

#### Language

A dropdown to set the display language for the entire app. Currently supported:

| Value | Language |
|---|---|
| **English** *(default)* | English |
| **繁體中文** | Traditional Chinese (Taiwan) |

**Language priority:**

1. If the user has previously changed the language in Settings, that choice is stored in `localStorage` (`ag_lang`) and used on every visit.
2. If no preference is saved, the app checks the browser's language preference (`navigator.languages`). If the browser language matches a supported language, it is used automatically.
3. If the browser language is not supported, the app falls back to English.

This priority means first-time visitors and shared-link viewers always see the app in their own browser language (if supported), without any setting from the original creator overriding it. Shared collections never carry language preferences that would force a language on the viewer.

All translations live inside `index.html` in a `TRANSLATIONS` object. The `data-i18n` attribute on static HTML elements is updated by `applyI18n()` on startup and whenever the language changes. Dynamic content uses the `t(key)` helper function.

Item type display names that differ from their stored value are also translated — for example, the Film & TV Trading Card type (stored as `"Film / TV Card"`) uses the `type_film_tv_card` key, which maps to "Trading Card" in English and "交換卡" in Traditional Chinese. The `displayItemType()` helper applies this mapping wherever item types are shown as text (detail modal, filter chips, eBay description builder). The filter set is also built from display names rather than raw stored values, so the Film & TV and Sports Trading Card types merge into a single filter chip that matches both.

#### Appearance

A row button that toggles between light and dark mode. The label updates to reflect the opposite of the current theme ("Switch to Dark Mode" / "Switch to Light Mode"), and the icon switches between a sun and moon accordingly. See [Light & Dark Mode](#light--dark-mode) for full details.

#### Photo Format

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

#### Image Fit

A dropdown that controls how photos are displayed when the uploaded image doesn't match the selected Photo Format:

| Option | Behavior |
|---|---|
| **Crop to fill** *(default)* | Image is cropped to fill the frame (`object-fit: cover`) — no gaps, edges may be trimmed |
| **Fit with black bars** | Full image always visible; gaps filled with a black background (`object-fit: contain`) |
| **Fit with blurred background** | Full image always visible; gaps filled with a blurred, scaled-up copy of the same photo |

- Saved to `localStorage` (`ag_img_fit`). Takes effect instantly — no re-render needed.
- The blurred background option has no effect on cards without a photo.
- Table thumbnails use `contain` (no blur) in the blurred background mode, since the thumbnail size is too small for the effect to be useful.

#### Privacy Mode

A dropdown to enable or disable privacy mode:

| Option | Behavior |
|---|---|
| **Off** *(default)* | All monetary values displayed normally |
| **On — blur monetary values** | Paid amounts and estimated values are blurred wherever they appear (stats bar, grid cards, table, detail modal) |

When privacy mode is on, an **eye button** appears in the header (to the left of the gear icon). Clicking it turns privacy mode off and hides the button. ROI/Return % is never blurred regardless of this setting.

- Saved to `localStorage` (`ag_privacy`). Takes effect instantly via a CSS `data-privacy` attribute on `<html>`.

#### Info Links

A dropdown to choose which site signer names and Detail 2 values link to in the table view and detail modal:

| Option | Signer links to | Detail 2 links to |
|---|---|---|
| **Wikipedia** *(default)* | `en.wikipedia.org/wiki/[Name]` | `en.wikipedia.org/wiki/[Value]` |
| **IMDb** | IMDb name search (`&s=nm`) | IMDb title search (`&s=tt`) |

- Saved to `localStorage` (`ag_info_link`). Takes effect immediately (gallery re-renders on change).
- Links appear in **table view** (Signer column, Film / Show column) and **detail modal** (Signer row, Film / Show row). Grid card text is plain — no links.

#### Grid View Visibility

A set of checkboxes that control which elements are shown on each card in the grid view. All are **checked (visible) by default**. Changes take effect immediately without closing Settings.

| Checkbox | Controls |
|---|---|
| **Image** | The photo area (including the "no photo" placeholder when no image is uploaded) |
| **Signer's Name** | The signer name in bold at the top of the card body |
| **Detail 1** | The first contextual field (e.g. character name, player name) |
| **Detail 2** | The second contextual field (e.g. film/show, team, album) |
| **Cert #** | The certificate number row (only shown when a cert number is set) |
| **Paid** | The amount paid row |
| **Est. Value** | The estimated value row |
| **ROI** | The return on investment percentage row |

- Each setting is saved individually to `localStorage` (`ag_grid_image`, `ag_grid_signer`, `ag_grid_detail1`, `ag_grid_detail2`, `ag_grid_cert`, `ag_grid_paid`, `ag_grid_value`, `ag_grid_roi`). Values are `"1"` (visible) or `"0"` (hidden); absence of a key means visible (default).
- These settings are included in Export/Import and shared collections.
- Hiding a field only affects the grid card display — the field still exists on the item and appears in the detail modal, table view, and exports.

---

### Data Tab

Eight row buttons, each with its own `(i)` tooltip on the right:

- **Export Collection** — downloads the full collection as a `.json` file. See [Import & Export](#import--export).
- **Import Collection** — opens a file picker to load a previously exported `.json` file, **replacing** the current collection. See [Import & Export](#import--export).
- **Merge Import** — opens a file picker to load a `.json` file and **add** its items to the existing collection without replacing anything. Useful for combining collections. See [Import & Export](#import--export).
- **Batch Import Photos** — opens a multi-file picker to select multiple images at once. Creates one stub item per image, using the filename (minus extension) as the signer name, with all other fields blank to fill in later. You can also drag image files directly onto the page anywhere outside the form to trigger the same flow. See [Import & Export](#import--export).
- **Copy Share Link** — uploads the collection to dpaste.com and copies a short shareable URL to the clipboard. See [Sharing a Collection](#sharing-a-collection).
- **Download CSV** — downloads all item metadata as a `.csv` spreadsheet (photos are not included). Columns cover every field: signers, **Detail 1**, **Detail 2**, item type, cert number, cert company, condition, paid, estimated value, value URL, tags, signing event, signing date, acquired how, location, notes, and date added. Paid and Est. Value column headers include the local currency code. Values containing commas or quotes are properly escaped.
- **Print Collection** — generates and opens a print-ready A4 layout of the collection. See [Print Layout](#print-layout).
- **Reset Collection** — permanently deletes your entire collection (items and photos) from both localStorage and IndexedDB, and returns the gallery to the empty state. A confirmation dialog is shown first. Separated from the other buttons by a divider and labelled in red to signal it is destructive. Export a backup first if you want to keep your data.

> ⚠️ In [View Mode](#view-mode) the Data tab shows a single **Save as My Collection** button instead of the backup and share buttons.

---

### Currency Tab

#### Two-Currency System

AutoGallery separates **where you record values** from **how you display them**:

| Currency | Purpose |
|---|---|
| **Local Currency** | The currency in which you enter **Paid** and **Est. Value** amounts. Stored with the data. |
| **Display Currency** | The currency shown throughout the UI — stats bar, cards, table, detail modal. |

If local and display currencies differ, a live exchange rate is fetched from the internet and used to convert all displayed monetary values.

#### Currency Options

Both dropdowns offer the same list of currencies:

`USD`, `GBP`, `EUR`, `TWD`, `JPY`, `CNY`, `AUD`, `CAD`, `HKD`, `SGD`, `KRW`

Each is shown with its symbol in the UI: `$`, `£`, `€`, `NT$`, `¥`, `¥`, `A$`, `CA$`, `HK$`, `S$`, `₩`.

#### Exchange Rate

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

## Print Layout

Click **Print Collection** in the Settings Modal (⚙️ → Data) to generate a print-ready A4 layout of your collection and open the browser's print dialog.

### Layout

The page is formatted for **A4 paper** (`@page { size: A4; margin: 12mm 14mm }`).

**Header** (top of the page):

- **AutoGallery** logo (left) with the same gold/dark two-tone styling as the app header
- Summary statistics (right): total item count, total paid, estimated value, and overall ROI — in the display currency. Monetary stats are omitted if Privacy Mode is on.
- **Generated [date]** — a timestamp line below the header rule, right-aligned.

**Card grid:**

- **3 columns**, regardless of the current screen column setting.
- Items are sorted **by estimated value, high to low** (independent of the active gallery sort).
- Each card mirrors the grid view layout:
  - Photo at the top, using the current **Photo Format** aspect ratio and **Image Fit** mode from Display settings.
  - Signer name in bold, followed by Detail 1 (e.g. character name) and Detail 2 (e.g. film/show title) on separate lines.
  - A thin divider, then rows for Cert #, Paid, Est. Value, and ROI — each label on the left, value on the right. Monetary rows are omitted if Privacy Mode is on.
- Cards with no photo show a "No photo" placeholder in the image area.
- Cards use `break-inside: avoid` so a single card is never split across two pages.

### Notes

- The print layout is generated fresh each time — it reflects the current collection and settings at the moment you click the button.
- Privacy Mode is respected: if enabled, Paid, Est. Value, and ROI are omitted from both the summary header and individual cards.
- The print CSS applies only inside `@media print` and has no effect on the normal browser view.

---

## Sharing a Collection

AutoGallery can generate a short URL that lets anyone view your collection in their own browser — no account, no app install required.

### How to Share

1. Open the **Settings Modal** (⚙️ in the header)
2. Under **Data**, click **Copy Share Link**
3. A warning dialog appears explaining that your collection will be uploaded publicly. Read it carefully — if your gallery contains financial details or personal information you wouldn't want others to see, click **Cancel** and use **Export Collection** to share the file privately instead.
4. Click **Share publicly** to proceed. The button label changes to "Uploading…" while the collection is sent to dpaste.com.
5. Once uploaded, a short URL is copied to your clipboard automatically — e.g.:
   ```
   http://localhost:4200/index.html#blob=G9GW3KB53
   ```
5. Send that URL to anyone. When they open it, AutoGallery loads and immediately displays your collection in read-only [View Mode](#view-mode)

### What Gets Uploaded

The **entire collection** is included — all item fields, all photos, and all user preferences (`settings`). The JSON is uploaded as a single paste to [dpaste.com](https://dpaste.com), a free public paste service.

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
| Saving items to localStorage | Never happens |
| Shared settings applied to localStorage | Temporarily (restored on exit) |

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
   - The shared items are saved as your own (metadata to `localStorage`, photos to IndexedDB)
   - The sharer's local currency is set as your new local currency
   - All shared preferences (theme, sort, columns, photo format, etc.) are applied permanently
   - View mode exits and the full editing UI is restored (Add Item button, Edit/Delete in detail modal, all Settings sections)
   - The exchange rate is re-fetched for your currency pair
   - A toast confirms: *"Saved N items as your collection"*
3. If cancelled, nothing changes.

> ⚠️ This permanently overwrites your existing collection. There is no undo. Export your current collection first if you want to keep it.

### Currency in View Mode

The shared collection was recorded in the sharer's local currency (stored in the JSON payload). When viewing, that currency is used as the local currency for exchange-rate conversion — so if the sharer used TWD and you switch your display currency to USD, the values convert correctly. Your own local currency setting in localStorage is not changed.

### Your Own Data is Untouched

Your own **collection** (items) is never touched in view mode — it remains in storage exactly as it was. View mode never calls `persist()`, so neither your `localStorage` metadata nor your IndexedDB photos are modified while viewing. The shared settings (theme, sort, columns, etc.) are applied temporarily to `localStorage` while viewing so you see the collection as the sharer had it configured. When you click **← Back to my collection**, your original preferences are restored from the snapshot taken at the moment you opened the share link, and your own items are reloaded.

### Exiting View Mode

Click **← Back to my collection** in the gold banner. This:
1. Clears the view mode state
2. Reloads your own items from storage (metadata from `localStorage`, photos from IndexedDB)
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
  "settings": {
    "ag_display_currency": "USD",
    "ag_theme": "dark",
    "ag_cols": "5",
    "ag_view": "grid",
    "ag_photo_format": "8x10L",
    "ag_img_fit": "cover",
    "ag_info_link": "wiki",
    "ag_privacy": "off",
    "ag_sort": "date-desc",
    "ag_custom_order": "[\"abc123\"]"
  },
  "items": [
    {
      "id": "abc123",
      "added": "2024-11-15T08:30:00.000Z",
      "name": "Harrison Ford",
      "signers": ["Harrison Ford"],
      "detail1": "Han Solo",
      "detail2": "Star Wars",
      "itemType": "Photo",
      "certNum": "A1234567",
      "certCompany": "beckett",
      "paid": 15000,
      "value": 22000,
      "valueUrl": "https://www.ebay.com/...",
      "notes": "Signed at Fan Expo 2023",
      "tags": ["sci-fi"],
      "signingEvent": "Fan Expo 2023",
      "signingDate": "2023-08-26",
      "acquiredHow": "In person",
      "condition": "Near Mint",
      "location": "Display case",
      "imgs": ["data:image/jpeg;base64,/9j/...", "data:image/jpeg;base64,/9j/..."]
    }
  ]
}
```

- `currency` is the **local currency** at time of export.
- All monetary values in `items` are in that local currency.
- `settings` contains all user preferences at time of export (only keys that were explicitly set are included).
- `imgs` is an array of base64-encoded JPEG strings. An empty array `[]` means no photos. The first element is the cover image shown on cards and in the table view.

### Import

Click **Import Collection** in the Settings Modal (⚙️ → Data). A file picker opens. Select a previously exported `.json` file.

**Import behavior:**

- The imported `currency` field is set as the new **local currency**.
- All items are loaded and persisted (metadata to `localStorage`, photos to IndexedDB), **replacing** the current collection entirely.
- All preferences in the `settings` field are applied immediately — theme, sort, columns, photo format, etc.
- The gallery re-renders immediately with the imported data.
- If the file is malformed or cannot be parsed, a toast error is shown.

> ⚠️ Import **overwrites** your current data. Export a backup before importing if you want to preserve your existing collection.

### Merge Import

Click **Merge Import** in the Settings Modal (⚙️ → Data). A file picker opens. Select a `.json` file in the same format as an exported collection.

**Merge Import behavior:**

- Items from the file are **appended** to the current collection — nothing is replaced or deleted.
- The imported `currency` and `settings` fields are ignored; your existing preferences are kept.
- A toast confirms how many items were added (e.g. "Added 12 items").
- If the file is malformed or contains no items, a toast error is shown.

Merge Import is useful for combining collections exported from different devices or sessions.

### Batch Import Photos

Click **Batch Import Photos** in the Settings Modal (⚙️ → Data), or drag one or more image files directly onto the page (outside the Add/Edit form).

**Batch import behavior:**

- One stub item is created per image file. The filename (minus extension) is used as the signer name; all other fields (detail1, detail2, cert, paid, value, tags, etc.) are left blank for you to fill in later.
- Images are run through the same compression pipeline as manual photo uploads — resized to a maximum of 900px on the longest edge, re-encoded as JPEG at 0.82 quality, and stripped of EXIF metadata.
- Items land at the top of the collection (newest first when sorted by date added). The sort order depends on the active sort setting.
- A toast confirms how many stubs were created (e.g. "Created 5 stub items — fill in details on each card"). Settings close automatically after import.

**Drag-and-drop:**

Dragging image files over the browser window (when the Add/Edit form is not open) triggers a full-screen drop overlay: a dark backdrop with a dashed gold box reading "Drop photos to import / One stub item will be created per image." Dropping the files runs the same import flow. Non-image files in the drop are silently ignored; if none of the dropped files are images, a toast error is shown.

---

## Mobile Behavior

The app is designed to work on small screens. On viewports **640px wide or narrower**:

- **Grid/Table toggle is hidden** — table view is not available on mobile.
- **Columns selector is hidden** — the column count is not adjustable on mobile.
- **Grid is forced** — even if `ag_view` is set to `"table"` in localStorage, the app renders grid view. This is enforced both in CSS (with `!important` overrides hiding `.table-wrap` and showing `.gallery`) and in JavaScript (a `isMobile` check in the render function).
- **Grid is 2 columns** — a CSS media query sets `--cols: 2` on mobile, overriding any saved column preference.
- **Cards go full width** — the text (signer name, Detail 1, Detail 2) spans the full card width rather than being cramped next to a fixed-width image column.
- **Horizontal scroll is prevented** — all layout uses `box-sizing: border-box` and the gallery padding is reduced to `12px` on mobile to prevent content from overflowing the viewport.
- **Sort and Filter share a row** — the Sort dropdown and Filter button appear side by side on a second row below the search bar, each taking half the available width.

---

## Data Storage

AutoGallery uses a **hybrid storage model**: lightweight item metadata and preferences live in `localStorage`, while the bulky photo data lives in **IndexedDB**. This keeps the collection from being capped by the small `localStorage` quota once you add lots of images.

### localStorage (metadata & preferences)

| Key | Type | Contents |
|---|---|---|
| `autogallery_v2` | JSON string | Array of all item objects, **without** the `imgs` photo data (photos are stored separately in IndexedDB) |
| `ag_currency` | String | Local currency code (e.g. `"TWD"`) |
| `ag_display_currency` | String | Display currency code (e.g. `"USD"`) |
| `ag_rate_cache` | JSON string | Cached exchange rate: `{from, to, rate, ts}` |
| `ag_cols` | String | Saved columns per row (e.g. `"5"`) |
| `ag_view` | String | Saved view mode: `"grid"` or `"table"` |
| `ag_theme` | String | Saved theme: `"light"` or `"dark"` |
| `ag_photo_format` | String | Saved photo format key (e.g. `"8x10L"`) |
| `ag_img_fit` | String | Saved image fit mode: `"cover"`, `"contain"`, or `"blurbg"` |
| `ag_info_link` | String | Saved info link target: `"wiki"` or `"imdb"` |
| `ag_privacy` | String | Saved privacy mode: `"off"` or `"on"` |
| `ag_sort` | String | Saved sort mode (e.g. `"date-desc"`, `"custom"`) |
| `ag_custom_order` | JSON string | Array of item IDs representing the user-defined custom sort sequence |
| `ag_lang` | String | Saved language preference (e.g. `"en"`, `"zh-TW"`). Only present when the user has manually selected a language — absence means auto-detect from browser. |
| `ag_grid_image` | String | Grid card image visibility: `"1"` (visible) or `"0"` (hidden). Absent = visible. |
| `ag_grid_signer` | String | Grid card signer name visibility: `"1"` or `"0"`. Absent = visible. |
| `ag_grid_detail1` | String | Grid card Detail 1 field visibility: `"1"` or `"0"`. Absent = visible. |
| `ag_grid_detail2` | String | Grid card Detail 2 field visibility: `"1"` or `"0"`. Absent = visible. |
| `ag_grid_cert` | String | Grid card Cert # row visibility: `"1"` or `"0"`. Absent = visible. |
| `ag_grid_paid` | String | Grid card Paid row visibility: `"1"` or `"0"`. Absent = visible. |
| `ag_grid_value` | String | Grid card Est. Value row visibility: `"1"` or `"0"`. Absent = visible. |
| `ag_grid_roi` | String | Grid card ROI row visibility: `"1"` or `"0"`. Absent = visible. |

### IndexedDB (photos)

| Database | Object store | Key | Value |
|---|---|---|---|
| `autogallery` | `images` | item `id` | The item's `imgs` array (base64 JPEG data URLs) |

At runtime, each item still carries its `imgs` array in memory, so all rendering, export, import, and share logic is unchanged. Photos only cross the storage boundary in two places: `load()` reads them from IndexedDB and reattaches them to each item, and `persist()` writes metadata to `localStorage` and syncs photos to IndexedDB (deleting records for items that no longer exist).

### Automatic migration

On first load after upgrading from a localStorage-only version, AutoGallery detects photos stored inline in `localStorage`, moves them into IndexedDB, and rewrites the `localStorage` item array without the image data. This happens transparently — no user action is needed, and nothing is lost.

AutoGallery also automatically migrates items that used the older `character` and `film` field names (from before the Item Type redesign) to the new `detail1` and `detail2` names. This runs on first load and on import — the renamed fields are written back to storage immediately. No data is lost in the process.

### Storage limits

- **IndexedDB** typically allows hundreds of MB up to several GB per origin (browsers permit roughly 50–60% of free disk space), so photo-heavy collections are no longer constrained the way they were under `localStorage`.
- Item photos are still stored as base64-encoded JPEGs (compressed to max 900px at 0.82 quality), roughly 100–400 KB each.
- **Fallback:** if IndexedDB is unavailable (e.g. some private-browsing modes or very old browsers), AutoGallery automatically falls back to the original behavior of storing photos inline in `localStorage`, and shows a notice. In that mode the older 5–10 MB `localStorage` limit applies.

---

## Item Schema

Each item is a JavaScript object with the following fields:

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique identifier generated with `Date.now() + Math.random()` at creation time |
| `added` | string | ISO 8601 timestamp of when the item was first saved (e.g. `"2024-11-15T08:30:00.000Z"`) |
| `name` | string | First (or only) signer's name. Always set — used as the sort key. |
| `signers` | string[] | Array of all signer names. Always present; single-signer items have `["Name"]`. Multi-signer items have two or more entries. |
| `detail1` | string | First contextual field value — label depends on item type (e.g. character name, player name, artist, author). Optional, may be empty string. |
| `detail2` | string | Second contextual field value — label depends on item type (e.g. film/show title, team name, album, title). Used as a Wikipedia/IMDb link target. Optional, may be empty string. |
| `certNum` | string | Certificate number (optional, may be empty string) |
| `certCompany` | string | One of: `"beckett"`, `"psa"`, `"jsa"`, `"swau"`, or `""` (none) |
| `paid` | number \| null | Amount paid, in the local currency |
| `value` | number \| null | Estimated current value, in the local currency |
| `valueUrl` | string | URL to a value source (optional, may be empty string) |
| `notes` | string | Free-form notes (optional, may be empty string) |
| `tags` | string[] | Array of tag strings (optional, defaults to `[]`) |
| `itemType` | string | Type of memorabilia (e.g. `"Photo"`, `"Jersey"`). Optional, may be empty string. |
| `signingEvent` | string | Event or venue where the item was signed. Optional, may be empty string. |
| `signingDate` | string | Date the item was signed in `YYYY-MM-DD` format. Optional, may be empty string. |
| `acquiredHow` | string | How the item was acquired (e.g. `"In person"`, `"Mail-in"`). Optional, may be empty string. |
| `condition` | string | Grade: one of `"Mint"`, `"Near Mint"`, `"Excellent"`, `"Very Good"`, `"Good"`, `"Fair"`, `"Poor"`, or `""`. Optional. |
| `location` | string | Physical storage location. Optional, may be empty string. |
| `imgs` | string[] | Array of base64 JPEG data URLs. Empty array means no photos. First element is the cover image. Held in memory on each item, but persisted to **IndexedDB** rather than `localStorage` (see [Data Storage](#data-storage)). |

---

## Technical Notes

### No External Dependencies

The app uses only:
- Native browser APIs (`localStorage`, `IndexedDB`, `fetch`, `FileReader`, `HTMLCanvasElement`)
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

Each uploaded file goes through the same pipeline before being appended to the item's `imgs` array:

```
User picks file(s)
  → FileReader reads each as Data URL
  → Image loaded into <img> element
  → Canvas resized to max 900px on longest side
  → Canvas.toDataURL('image/jpeg', 0.82) produces compressed base64
  → Appended to item.imgs[]
```

Multiple files selected in one picker dialog are processed in parallel and appended in the order they finish compressing. Old data exported with a single `img` field is automatically migrated to `imgs: [img]` on load. Old data with `character`/`film` fields is automatically migrated to `detail1`/`detail2` (see [Automatic migration](#automatic-migration)).

### Modal Behavior

Modals use a two-class system:
- `.overlay` — the full-screen backdrop (always in DOM, `opacity: 0`, `pointer-events: none`)
- `.overlay.open` — adds `opacity: 1` and `pointer-events: all`, animates the inner `.modal` from a slightly translated/scaled state to its natural position

Closing can happen via: the ✕ button, a Cancel button, a backdrop click (on the overlay itself, not the modal), or Delete confirmation.

### Sorting Implementation

Sort is applied after search filtering. Items without a value (`null`) are sorted to the bottom when sorting by value or paid. The selected sort is saved to `localStorage` (`ag_sort`) and restored on page load.

**Custom Order** is a special sort mode that allows manual sequencing. Moving an item via the +/− buttons updates the `ag_custom_order` array in `localStorage` and re-renders the gallery. The merge logic preserves the positions of items that are filtered out — only visible items shift when a move is performed. Items not yet present in `ag_custom_order` (e.g. newly added items) sort to the end.

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

`getLocalCurrency()` returns `viewLocalCurrency` when `viewMode` is true, transparently making the exchange rate system use the sharer's currency without touching `localStorage`. When `exitViewMode()` is called, both variables are reset and the now-async `load()` is awaited to restore your own items (metadata from `localStorage`, photos from IndexedDB).

### hashchange Listener

```javascript
window.addEventListener('hashchange', checkShareURL);
```

This single listener means share URLs work whether the page is freshly loaded or already open. When the user pastes a share URL into the address bar while AutoGallery is already running, the browser fires `hashchange` and `checkShareURL` handles it identically to the initial page load case. The hash is immediately cleared via `history.replaceState` so the loaded state is not tied to the URL.
