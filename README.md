# autographed.app

A single-file browser app for managing an autograph and memorabilia collection. Data is stored locally in the browser by default, with optional cloud backup via a free account.

> **Note:** This README documents `dashboard.html` (the app) only. It does not cover `index.html` (the marketing page) or the `/blog/` section.

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
15. [Public Collections](#public-collections)
16. [Publicly Shared Collections Page](#publicly-shared-collections-page)
17. [View Mode](#view-mode)
18. [Accounts & Cloud Sync](#accounts--cloud-sync)
19. [Light & Dark Mode](#light--dark-mode)
20. [Footer](#footer)
21. [Mobile Behavior](#mobile-behavior)
22. [Data Storage](#data-storage)
23. [Item Schema](#item-schema)
24. [Technical Notes](#technical-notes)

---

## Overview

autographed.app is a **single HTML file** (`dashboard.html`). There is no server, no database, no external JavaScript libraries, and no external CSS files. Everything — markup, styles, and logic — lives inside one self-contained file.

Key characteristics:

- **Persistent local storage** — item metadata, preferences, and cached exchange rates are stored in the browser's `localStorage`; photos are stored in **IndexedDB** so large collections aren't capped by the small `localStorage` quota.
- **No account required** — open the page and start adding items immediately.
- **Optional cloud backup** — create a free account to sync your collection to the cloud and restore it on any device. Without an account, your data lives only in your browser and will be lost if you clear browser data.
- **Designed for 8×10 autographed photos and similar memorabilia**, but works for any signed collectible.

---

## Getting Started

Open `dashboard.html` in any modern browser, or visit the hosted version at [autographed.app](https://autographed.app).

The app is blank on first launch. The empty state shows two buttons: **+ Add Item** to begin building your own collection, and **Load Demo Collection** to instantly populate the gallery with a sample set of items so you can explore the app before adding your own data.

---

## Header

The header is sticky — it stays at the top of the viewport while you scroll. It contains:

### Logo

**autographed**.app — "autographed" is styled in gold, ".app" in the default text color.

### Header Controls (right side, left to right)

| Control | Description |
|---|---|
| 👁 Eye (privacy) | Visible only when Privacy Mode is enabled. Clicking it turns Privacy Mode off and hides the button. |
| 👤 Account | Opens the **Account Modal** for sign-in, account creation, cloud sync, and password management. When signed in, the person icon is replaced by the first letter of your email in gold. |
| ⚙️ Settings | Opens the **Settings Modal** for theme, presentation mode, import/export, and currency options |
| **+ Add Item** | Opens the Add Item modal |

The Settings button is a 36×36px square icon button with rounded corners. On hover it brightens slightly.

The **+ Add Item** button is styled in gold with black text to make it the primary call to action. On mobile (≤480px), the text label is hidden and only the **+** icon is shown to save header space.

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
| **Cert Type** | Authentication types used across the collection |
| **Acquisition Method** | All acquisition methods in the collection |
| **Signing Event** | All signing events or venues in the collection |
| **Location** | All physical storage locations in the collection |
| **Detail 2** | All Detail 2 values in the collection (film/show titles, team names, album titles, etc.) |
| **Signer** | All signer names in the collection |

Each option is a toggleable chip. Click a chip to select it (turns gold); click again to deselect. **Multiple chips can be selected simultaneously.**

**Filter logic:**
- Within a category — **OR**: an item matches if it has *any* of the selected values (e.g. tag "sci-fi" OR tag "horror")
- Across categories — **AND**: an item must satisfy *all* active categories (e.g. tag "sci-fi" AND cert type "PSA/DNA Authenticated")

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
   - **Cert** label + a gold clickable link to the authentication company's verification page (if a cert type is selected). Clicking the link **also copies the cert number to the clipboard** and shows a toast. The link click does **not** open the detail modal.
   - **Paid** label + value (or `—` if not set)
   - **Est. Value** label + value; if a Search Term is set, the value is a clickable gold link to matching eBay listings in a new tab. The link click does **not** open the detail modal.
   - **ROI** label + percentage value, green or red. Only shown when both Paid and Est. Value are set.

3. **Clicking the card** (anywhere except the links) opens the **Item Detail Modal**.

Cards animate on hover: they lift up 3px and show a deeper shadow.

---

## Table View

An alternative view available on desktop. Items are shown as rows in a sortable table with the following columns:

| Column | Content |
|---|---|
| (thumbnail) | 40px-tall thumbnail of the photo (width scales with the Photo Format aspect ratio), or a placeholder icon |
| **Signer** | Signer name as a clickable link (Wikipedia or IMDb per the Info Links setting), or **"Multiple Signers"** for multi-signer items |
| **Detail 1** | First contextual field value (e.g. character name, player name, artist, author) — plain text |
| **Detail 2** | Second contextual field value (e.g. film/show, team, album, title) — clickable link (Wikipedia or IMDb per the Info Links setting) |
| **Paid** | Amount paid in display currency |
| **Est. Value** | Estimated value in display currency, colored green (positive) or red (negative) relative to paid. If a Search Term is set, the value is a clickable link to matching eBay listings that opens in a new tab. |
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
| **Item Type** | Dropdown | Optional. The kind of memorabilia. Choose from a predefined grouped list (see [Item Types](#item-types) below). Defaults to **Photo** for new items. Filterable. A hint below the label reads "If you can't find a suitable Item Type, select 'Other'." |
| **Detail 1** | Text input | Optional. First contextual field — label and placeholder change based on Item Type (e.g. "Character" for film/TV items, "Player" for sports items, "Artist" for music items). |
| **Detail 2** | Text input | Optional. Second contextual field — label and placeholder change based on Item Type (e.g. "Film / Show", "Team", "Album"). The value is used as a Wikipedia or IMDb link in the table and detail views. |
| **Signer 1** | Text input | Required. The first (or only) autograph signer's real name. |
| **+ button** | Button | Adds a Signer 2 field. Each additional signer row has a − button to remove it. There is no upper limit on the number of signers. |
| **Cert #** | Text input | Optional. The authentication certificate number. |
| **Cert Type** | Dropdown | Controls the verification URL linked from the cert number. See [Certification URL Derivation](#certification-url-derivation) for the full list of options and their URLs. |
| **Paid** | Number input | Optional. What you paid, in your **local currency**. |
| **Est. Value** | Number input | Optional. Current estimated market value, in your **local currency**. |
| **Search Term** | Text input | Optional. A search term used to look up the item's value on eBay. When set, the Est. Value becomes a clickable link pointing to matching eBay listings (sold or active, depending on the **Est. Value Search** setting). See [Est. Value Search](#est-value-search). |
| **Notes** | Textarea | Optional. Free-form notes. Resizable vertically. |
| **Tags** | Tag input | Optional. Type a tag and press **Enter** or **,** to add it as a chip. Press **Backspace** on an empty input to remove the last tag. Click **×** on a chip to remove it. Multiple tags per item are supported. As you type, a dropdown suggests matching tags already used elsewhere in your collection — click a suggestion or navigate with **↑ ↓** and press **Enter** to select, **Escape** to dismiss. |

### Advanced Fields

Below the Tags field is an **▶ Advanced Fields** expander (`<details>` element). Clicking it reveals five additional optional fields:

| Field | Type | Notes |
|---|---|---|
| **Signing Event / Venue** | Text input | Optional. The event or venue where the item was signed (e.g. "Fan Expo 2024", "Private signing"). Filterable. |
| **Signing Date** | Date picker | Optional. The date the item was signed. Shown in the detail modal as a formatted date (e.g. "June 15, 2024"). Not filterable. |
| **Acquisition Method** | Dropdown | Optional. How the item was obtained. Options: In-Person, Mail-In / TTM, Convention, Purchased Online, Purchased at Show, Auction, Gift, Trade, Gallery / Dealer, Other. Filterable. |
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
| **Film & TV** | Photo², Poster², Prop, Script, Funko Pop, Lithograph, Program, Trading Card¹ | Character | Film / Show |
| **Sports** | Photo², Poster², Jersey, Baseball, Bat, Helmet, Mini Helmet, Football, Basketball, Hockey Puck, Trading Card¹, Glove, Shoe / Cleat | Player | Team |
| **Music** | Photo², Poster², Album / Record, CD, Vinyl | Artist | Album |
| **Music** | Instrument | Artist | Band / Group |
| **Music** | Setlist | Artist | Tour / Show |
| **Books & Literature** | Book, Comic | Author | Title |
| **Books & Literature** | Magazine | Subject | Publication |
| **Other** | Other | Detail 1 | Detail 2 |

> **Default:** New items default to **Photo** (Film & TV), showing the Character and Film / Show labels.

> ¹ Types that share a display name across categories use distinct internal stored values to preserve the correct Detail 1 / Detail 2 field labels per category. In the filter modal they merge into a single chip. Specifically: the Film & TV **Trading Card** is stored as `"Film / TV Card"`, the Sports **Trading Card** as `"Trading Card"`; Sports **Photo/Poster** as `"Sports Photo"` / `"Sports Poster"`; Music **Photo/Poster** as `"Music Photo"` / `"Music Poster"`. Film & TV Photo and Poster use the plain `"Photo"` / `"Poster"` stored values.

> ² Shared display name — see note ¹ above.

The Detail 1 and Detail 2 labels also update instantly when you switch languages — they are always displayed in the current language.

### Photo Upload

Items support **multiple photos**. The **Photos** section of the form shows a row of thumbnails for all uploaded photos, followed by an **Add photo** tile.

- **Click "Add photo"** to open the system file picker (accepts `image/*`; you can select multiple files at once)
- Each selected file is compressed client-side and added to the thumbnail row

Each thumbnail has an **✕** button to remove that individual photo. When an item has more than one photo, thumbnails can be **dragged and dropped** to reorder them — grab a thumbnail and drop it onto another position to rearrange. The first photo in the row is the **cover image** shown on the card and in the table view. There is no limit on the number of photos per item beyond IndexedDB capacity.

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
  - Cert # — clickable link to the cert type's verification page. Clicking **also copies the cert number to the clipboard** and shows a toast confirmation.
  - Paid (in display currency)
  - Est. Value (in display currency, with eBay link if a Search Term is set)
  - ROI — percentage, colored green or red. Only shown when both Paid and Est. Value are set.
  - Signing Date — formatted as "Month D, YYYY" (only shown if set)
  - Signing Event / Venue — plain text (only shown if set)
  - Acquisition Method — plain text (only shown if set)
  - Physical Location — plain text (only shown if set)
  - Tags — clickable chips. Clicking a tag closes the modal and filters the gallery to items with that tag.
- **Notes** — displayed under a "NOTES" section label, in a lightly styled box, after the table. Only shown if notes are set.
- **Description** — an auto-generated text block, separated by a divider. See [Description](#description) below.
- **Date Added** — shown below the Description section.

Fields with no value are omitted (not shown as blank rows).

### Description

A **Description** section appears at the bottom of every detail modal. It auto-generates a plain-text description from the item's fields, formatted for pasting into a listing or elsewhere.

**Format:**

> *[Signer(s)] ([Detail 1] in [Detail 2]) signed [item type]. Professionally authenticated by [Cert Type] (Certificate #[Cert #]). The [item type] is in [condition] condition. The [item type] was signed at [Signing Event] on [Date Signed].*

Rules:
- The signer name and Detail 1 / Detail 2 parenthetical open the description. For multi-signer items the parenthetical is omitted.
- Item type is always lowercase (e.g. "photo", not "Photo").
- The cert sentence is included only if a cert number is set. The condition sentence is included only if condition is set. The signing sentence is included only if a signing event or signing date is set — it uses whichever of the two are available.
- If Notes are set **and** the **Include notes in auto-description** setting is enabled (Settings → General → Auto-Description), they are appended at the end. Notes are excluded by default.
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

Click **Start Slideshow** in **Settings → General → Presentation Mode** to launch a full-screen photo slideshow — designed to display the collection on a TV or monitor.

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
| **General** | Language, Appearance, Presentation Mode, Photo Format, Image Fit, Info Links, Est. Value Search, Privacy Mode, Auto-Description, Grid View Visibility |
| **Data** | Backup, Cloud Sync, and Public Collection (normal mode) or Save as My Collection (view mode) |
| **Currency** | Local Currency, Display Currency, Exchange Rate (Local Currency hidden in view mode) |

The modal always opens on the **General** tab.

### Tooltip Buttons

Every setting has a small italic **i** circle button at the right end of its control row (dropdown or action button). Clicking or tapping it toggles an explanation panel directly below. Only one tip is open at a time — opening a new one closes the previous one.

---

### General Tab

#### Language

A dropdown to set the display language for the entire app. Currently supported:

| Value | Language |
|---|---|
| **English** *(default)* | English |
| **Deutsch** | German |
| **Español** | Spanish (neutral — suitable for all Spanish-speaking regions) |
| **Français** | French |
| **繁體中文** | Traditional Chinese (Taiwan) |

**Language priority:**

1. If the user has previously changed the language in Settings, that choice is stored in `localStorage` (`ag_lang`) and used on every visit.
2. If no preference is saved, the app checks the browser's language preference (`navigator.languages`). If the browser language matches a supported language, it is used automatically. Any `de-*` locale (e.g. `de-DE`, `de-AT`, `de-CH`) maps to German; any `es-*` locale (e.g. `es-MX`, `es-AR`, `es-ES`) maps to the neutral Spanish translation; any `fr-*` locale (e.g. `fr-FR`, `fr-CA`, `fr-BE`) maps to French.
3. If the browser language is not supported, the app falls back to English.

This priority means first-time visitors and shared-link viewers always see the app in their own browser language (if supported), without any setting from the original creator overriding it. Shared collections never carry language preferences that would force a language on the viewer.

All translations live inside `dashboard.html` in a `TRANSLATIONS` object. The `data-i18n` attribute on static HTML elements is updated by `applyI18n()` on startup and whenever the language changes. Dynamic content uses the `t(key)` helper function.

**Dropdown option translation** — item types, condition grades, and acquisition methods are stored in English as raw values in the JSON data (`itemType`, `condition`, `acquiredHow`). This keeps exports and imports language-neutral. Display-only translation is layered on top via three lookup maps and corresponding helper functions:

| Map | Helper function | Used for |
|---|---|---|
| `ITEM_TYPE_KEYS` | `displayItemType(storedValue)` | Item type dropdown, filter chips, detail modal, eBay description |
| `CONDITION_KEYS` | `displayCondition(storedValue)` | Condition dropdown, filter chips, detail modal, eBay description |
| `ACQUIRED_KEYS` | `displayAcquiredHow(storedValue)` | Acquisition method dropdown, filter chips, detail modal |

Each map translates a stored English value to a `TRANSLATIONS` key (e.g. `"Near Mint"` → `"cond_near_mint"` → `"接近完美"` in Traditional Chinese). If a stored value has no entry in the map, the raw stored string is displayed as a fallback.

**Filter chip deduplication** — the filter set for item types, conditions, and acquisition methods is built from display names (via the helper functions) rather than raw stored values. This means types that share a display name across categories — such as `"Film / TV Card"` and `"Trading Card"` both displaying as "Trading Card" — merge into a single filter chip that matches all of them. Filter matching likewise uses the display helpers so stored English values never need to change.

**`applyI18n()` extension** — in addition to updating `textContent` (via `data-i18n`) and `innerHTML` (via `data-i18n-html`), `applyI18n()` also updates `<optgroup>` `label` attributes (via `data-i18n-label`) so grouped dropdown headers translate correctly alongside their option text.

#### Appearance

A row button that toggles between light and dark mode. The label updates to reflect the opposite of the current theme ("Switch to Dark Mode" / "Switch to Light Mode"), and the icon switches between a sun and moon accordingly. See [Light & Dark Mode](#light--dark-mode) for full details.

#### Presentation Mode

A row button labelled **Start Slideshow**. Clicking it closes the Settings modal and immediately launches the full-screen photo slideshow. See [Presentation Mode](#presentation-mode) for full details.

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

#### Est. Value Search

A dropdown that controls which eBay listing type the Est. Value links to when an item has a **Search Term** set.

| Option | Behaviour | URL pattern |
|---|---|---|
| **eBay Sold Listings** *(default)* | Links to completed sold listings — useful for checking real market prices | `ebay.com/sch/i.html?_nkw=TERM&LH_Sold=1` |
| **eBay Active Listings** | Links to currently active listings | `ebay.com/sch/i.html?_nkw=TERM` |

The Search Term is URL-encoded and inserted as the `_nkw` query parameter. It is recommended to append `(bgs,bas,beckett,psa,dna,jsa,swau)` to search terms so results are filtered to authenticated items only — e.g. `Hayden Christensen autograph (bgs,bas,beckett,psa,dna,jsa,swau)`.

**Legacy compatibility** — if an item's `valueUrl` field already contains a full URL (starting with `http://` or `https://`), that URL is used as-is regardless of this setting. Only plain search terms (non-URL text) are passed through the eBay URL builder.

- Saved to `localStorage` (`ag_value_search`). Takes effect immediately — no re-render needed.

#### Auto-Description

A checkbox that controls whether item notes are appended to the auto-generated description in the detail modal.

| Checkbox | Default | Behaviour |
|---|---|---|
| **Include notes in auto-description** | Unchecked | When checked, the item's Notes field is appended as an extra paragraph at the end of the auto-generated description text |

- Saved to `localStorage` (`ag_desc_notes`). Value is `"1"` (include) or `"0"` / absent (exclude).
- Included in Export/Import and shared collections.
- Notes are always visible in the detail modal itself regardless of this setting — this only controls whether they appear in the copyable description text.

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

Row buttons, each with its own `(i)` tooltip on the right:

- **Export Collection** — downloads the full collection as a `.json` file. See [Import & Export](#import--export).
- **Export reminder** — a toggle switch (off by default). When enabled, the browser shows a confirmation dialog if you try to leave or close the page after making changes to your collection since the last export. The reminder is automatically cleared after each successful export, so it only fires when there are genuinely unsaved changes.
- **Import Collection** — opens a file picker to load a previously exported `.json` file, **replacing** the current collection. See [Import & Export](#import--export).
- **Merge Import** — opens a file picker to load a `.json` file and **add** its items to the existing collection without replacing anything. Useful for combining collections. See [Import & Export](#import--export).
- **Batch Import Photos** — opens a multi-file picker to select multiple images at once. Creates one stub item per image, using the filename (minus extension) as the signer name, with all other fields blank to fill in later. You can also drag image files directly onto the page anywhere outside the form to trigger the same flow. See [Import & Export](#import--export).
- **Download CSV** — downloads all item metadata as a `.csv` spreadsheet (photos are not included). Columns cover every field: signers, **Detail 1**, **Detail 2**, item type, cert number, cert type, condition, paid, estimated value, search term, tags, signing event, signing date, acquired how, location, notes, and date added. Paid and Est. Value column headers include the local currency code. Values containing commas or quotes are properly escaped.
- **Print Collection** — generates and opens a print-ready A4 layout of the collection. See [Print Layout](#print-layout).
- **Sync to cloud** — uploads your collection to your account's cloud storage. Disabled (greyed out) when not signed in. See [Accounts & Cloud Sync](#accounts--cloud-sync).
- **Restore from cloud** — downloads your last cloud backup and replaces your local collection. Disabled (greyed out) when not signed in. Shows a confirmation dialog before overwriting. See [Accounts & Cloud Sync](#accounts--cloud-sync).
- **Reset Collection** — permanently deletes your entire collection (items and photos) from both localStorage and IndexedDB, and returns the gallery to the empty state. A confirmation dialog is shown first. Separated from the other buttons by a divider and labelled in red to signal it is destructive. Export a backup first if you want to keep your data.

> ⚠️ In [View Mode](#view-mode) the Data tab shows a single **Save as My Collection** button instead of the backup and share buttons.

---

### Currency Tab

#### Two-Currency System

autographed.app separates **where you record values** from **how you display them**:

| Currency | Purpose |
|---|---|
| **Local Currency** | The currency in which you enter **Paid** and **Est. Value** amounts. Stored with the data. |
| **Display Currency** | The currency shown throughout the UI — stats bar, cards, table, detail modal. |

If local and display currencies differ, a live exchange rate is fetched from the internet and used to convert all displayed monetary values.

#### Currency Options

Both dropdowns offer 152 currencies covering all national currencies, organized into two optgroup sections:

**Popular** — the 11 most commonly used currencies, shown at the top:

`USD`, `GBP`, `EUR`, `TWD`, `JPY`, `CNY`, `AUD`, `CAD`, `HKD`, `SGD`, `KRW`

**Other** — all remaining national currencies in alphabetical order by ISO 4217 code, shown with their full name for identification (e.g. `BRL — R$ (Brazilian Real)`, `INR — ₹ (Indian Rupee)`).

The optgroup labels ("Popular" / "Other") are translated along with the rest of the UI when the language is changed.

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
| Same currency | "Same currency — no conversion needed"; Refresh button hidden |
| Loading | "Fetching rate…"; Refresh button disabled |
| Loaded | "1 $USD = 31.6335 NT$TWD" with a timestamp ("Updated just now" / "Updated Xm ago") |
| Error | "Could not fetch rate — check your connection" |

A **Refresh** button allows manually re-fetching the rate at any time (bypasses the 1-hour cache). All status strings and the Refresh button label are fully translated (English, Spanish, French, German, and Traditional Chinese).

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

- **autographed.app** logo (left) with the same gold/dark two-tone styling as the app header
- Summary statistics (right): total item count, total paid, estimated value, and overall Return — in the display currency. Monetary stats are omitted if Privacy Mode is on.
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

## Public Collections

Signed-in users can make their collection publicly accessible via a permanent URL. Anyone who visits the URL sees the collection in read-only [View Mode](#view-mode) — no account required.

### Enabling Public Access

1. Open the **Account modal** (person icon in the header)
2. Under the sync buttons, click **Make collection public**
3. A warning panel expands explaining that your entire collection — including photos, paid amounts, and certificate numbers — will be publicly accessible
4. Click **Enable** to confirm

On first enable, a random 10-character username (e.g. `k7m2xp9qr4`) is generated and stored in the `public_profiles` database table. This username never changes, so your public URL remains stable even if you disable and re-enable sharing.

### Your Public URL

Once enabled, the Account modal shows:

> *Anyone with the following URL can see your collection:*

```
https://autographed.app/dashboard.html?shared=<username>
```

Click **Copy link** to copy it to your clipboard. Send it to anyone — they open it in a browser and see your collection immediately.

### Listing Your Collection in the Public Directory

When your public URL is enabled, a checkbox appears in the Account modal:

> ☐ **List my collection on [Publicly Shared Collections](shared.html)**

When checked, your collection is added to the public directory at `shared.html` (see [Publicly Shared Collections Page](#publicly-shared-collections-page)). Unchecking it removes your collection from the directory immediately — your public URL stays active, but it is no longer discoverable via the listing. This setting defaults to unchecked and is stored in `public_profiles.listed`.

### What Visitors See

The URL triggers [View Mode](#view-mode): the visitor sees your full collection (all items, photos, and values) in read-only mode with the same gallery UI, search, sort, and filters. No account or login is required to view it.

The collection served is always the **most recent cloud sync** — whatever you last uploaded via **↑ Sync to cloud**. If you have never synced, the public URL will not load successfully.

### Disabling Public Access

Click **Disable** in the Account modal. Your collection immediately becomes inaccessible at the public URL — the sharing flag is set to `false` in the database and the storage access policy denies reads instantly. Your username is preserved, so re-enabling restores the same URL. Disabling also removes your collection from the public directory if it was listed.

### Privacy Warning

> ⚠️ Your **entire** cloud-synced collection is publicly accessible — all item details, photos, paid amounts, estimated values, and certificate numbers. Anyone who knows the URL can read all of this data. Do not enable public access if your collection contains sensitive financial or personal information.

---

## Publicly Shared Collections Page

`shared.html` is a separate static page (same visual style as the rest of the site) that lists all collections where the owner has both enabled public access **and** checked the "List my collection" option.

The page heading is **Publicly Shared Collections** with the subheading *"Browse autograph collections shared by users."* A gold-tinted warning box below the subheading reads:

> ⚠️ Publicly shared collections may not have yet been reviewed by autographed.app admins, and may contain content not suitable for all audiences.

### What It Shows

A table with one row per listed collection:

| Column | Content |
|---|---|
| **Collection** | The collection's random username (e.g. `k7m2xp9qr4`) |
| **Items** | Number of items in the collection, from `collection_meta.item_count` |
| **Last updated** | Date of the owner's most recent cloud sync, from `collection_meta.last_modified` |
| **View →** | Link to open the collection in [View Mode](#view-mode) |

Rows are sorted by most recently updated first. On mobile, the **Last updated** column is hidden.

### Empty & Error States

- **No collections listed yet** — shown left-aligned below the heading when no collections have `listed = true`
- **Error** — shown with a warning icon if the Supabase query fails (e.g. network issue)
- **Loading** — a spinner is shown while the data is being fetched

### How It Works

On page load, `shared.html` lazy-loads the Supabase SDK and runs two queries using the public anon key:

1. `SELECT username, user_id FROM public_profiles WHERE enabled = true AND listed = true`
2. `SELECT user_id, item_count, last_modified FROM collection_meta WHERE user_id IN (...)`

The second query is permitted by an RLS policy that allows anonymous reads of `collection_meta` rows where the owner has `enabled = true AND listed = true` in `public_profiles`. Results are joined client-side and sorted by `last_modified` descending.

### Required Supabase SQL

The following SQL must be run once in the Supabase SQL editor to add the `listed` column and the `collection_meta` read policy:

```sql
ALTER TABLE public_profiles ADD COLUMN IF NOT EXISTS listed boolean NOT NULL DEFAULT false;

CREATE POLICY "Anyone can read meta of listed collections"
ON collection_meta FOR SELECT TO public
USING (EXISTS (
  SELECT 1 FROM public_profiles
  WHERE public_profiles.user_id = collection_meta.user_id
    AND public_profiles.enabled = true
    AND public_profiles.listed = true
));
```

Until this migration is run, the listing checkbox has no effect (the column doesn't exist, so `listed` is treated as `false` everywhere). The dashboard falls back gracefully — if the `listed` column is missing from the query response, it retries without that column so the rest of public sharing continues to work.

---

## View Mode

View mode is a read-only state that activates when opening a public collection URL. It lets you browse someone else's collection without affecting your own data.

### Entering View Mode

View mode is entered automatically when a `?shared=<username>` query parameter is detected on page load. It is never entered manually. See [Public Collections](#public-collections) for how a public URL is created.

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
| **Settings → Data** section (Export, Import) | Hidden |
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
| **Settings → Data → Save as My Collection** | ✅ See below |
| **Settings → Display Currency** | ✅ Change how values are shown |
| **Settings → Appearance** (theme toggle) | ✅ Light/dark mode still works |
| Exchange rate conversion | ✅ Works against the shared collection's local currency |

### Saving a Shared Collection as Your Own

While in view mode, the Settings modal's **Data** tab shows a **Save as My Collection** button instead of the normal export/import/share buttons. This lets you adopt the shared collection as your own, replacing whatever you currently have saved.

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

Your own **collection** (items) is never touched in view mode — it remains in storage exactly as it was. View mode never calls `persist()`, so neither your `localStorage` metadata nor your IndexedDB photos are modified while viewing. The owner's settings (theme, sort, columns, etc.) are applied temporarily to `localStorage` while viewing so you see the collection as they had it configured. When you click **← Back to my collection**, your original preferences are restored from the snapshot taken at the moment you opened the public URL, and your own items are reloaded.

### Exiting View Mode

Click **← Back to my collection** in the gold banner. This:
1. Clears the view mode state
2. Reloads your own items from storage (metadata from `localStorage`, photos from IndexedDB)
3. Restores the Add Item button and all edit/delete controls
4. Hides the banner
5. Re-fetches the exchange rate for your own currency pair

---

## Accounts & Cloud Sync

An account is **optional** but recommended. Without one, your collection lives only in your browser — if you clear browser data or switch devices, it is gone. With an account, you can back up your collection to the cloud and restore it on any device.

Cloud sync is powered by **[Supabase](https://supabase.com)** (authentication + object storage + database).

### Account Modal

Click the **person icon** in the header to open the Account modal. Once signed in, the icon is replaced by the first letter of your email address in gold.

The modal has three states:

| State | Contents |
|---|---|
| **Signed out** | Sign-in and Create account tabs with email + password forms |
| **Signed in** | Your email address, last sync status, Sync to cloud / Restore from cloud buttons, public collection toggle (with URL when enabled), and Change password / Sign out buttons |
| **Loading** | Shown briefly while connecting to Supabase |

### Creating an Account

1. Open the Account modal (person icon in the header)
2. Click **Create account**
3. Enter your email address and a password (minimum 8 characters)
4. Click **Create account** — Supabase sends a confirmation email
5. Click the link in the email to verify your address
6. Return to the app and sign in

### Signing In

1. Open the Account modal
2. Enter your email and password
3. Click **Sign in**

Your session is persisted by Supabase's SDK — you remain signed in across page reloads and browser restarts until you explicitly sign out.

### Syncing to the Cloud

Click **↑ Sync to cloud** in the Account modal (or **Sync to cloud** in Settings → Data).

This uploads a complete snapshot of your collection — all item metadata, all photos, and current settings — to your private Supabase Storage bucket as `{uid}/collection.json`. A lightweight metadata record (`last_modified` timestamp and `item_count`) is also written to the `collection_meta` database table for fast conflict checking.

- Only the signed-in user can read or write their own storage path (enforced by Row Level Security policies).
- Each sync **overwrites** the previous cloud backup — there is no version history.
- The sync hint in the Account modal updates to show the last synced time and item count after a successful push.

### Restoring from the Cloud

Click **↓ Restore from cloud** in the Account modal (or **Restore from cloud** in Settings → Data).

Before downloading, the app checks `collection_meta` for your cloud backup's metadata and shows a **conflict dialog** comparing:

- **Cloud**: how many items are in the cloud backup and when it was last saved
- **Local**: how many items are in your current local collection and when it was last changed

You can then choose to **Restore** (replacing your local collection with the cloud version) or **Cancel** (keeping your local collection). Restore downloads the full collection JSON from Supabase Storage and applies it exactly as if you had imported a `.json` file — all items, photos, and settings are replaced.

> ⚠️ Restoring overwrites your current local collection. Export a local backup first if you want to keep it.

### Sync Buttons in Settings

The **Sync to cloud** and **Restore from cloud** buttons also appear in the **Settings → Data** tab. They behave identically to the buttons in the Account modal but are greyed out and non-interactive when no user is signed in.

### Data Model

| Resource | Details |
|---|---|
| **Supabase Auth** | Email/password authentication. Session tokens are managed by the Supabase JS SDK and stored in the browser. |
| **Storage bucket: `collections`** | Private bucket. Each user's collection is stored at `{uid}/collection.json`. The file format is identical to the local export format: `{ currency, settings, items }` including base64-encoded photos. Normally private; an additional RLS policy allows anonymous reads for users who have enabled public sharing. |
| **Table: `collection_meta`** | Stores `{ user_id, last_modified, item_count, app_version }`. Used for the conflict dialog without downloading the full file. `user_id` has `ON DELETE CASCADE` to `auth.users`. |
| **Table: `public_profiles`** | Stores `{ user_id, username, enabled, listed }`. `username` is a randomly generated 10-character alphanumeric string assigned on first enable. `listed` is a boolean (default `false`) controlling whether the collection appears on `shared.html`. Authenticated users can manage their own row; the `enabled = true` rows are publicly readable (by both authenticated and anonymous clients) so the public URL can resolve `username → user_id`. |

### Security

The Supabase **publishable key** (prefixed `sb_publishable_`) is embedded client-side in `dashboard.html`. This key is safe to expose — it is scoped to the project and cannot bypass Row Level Security (RLS) policies. Security is enforced entirely by RLS:

- **Storage (private reads)**: `auth.uid()::text = (storage.foldername(name))[1]` — users can only read and write files under their own UID path.
- **Storage (public reads)**: an additional policy allows any client (including unauthenticated) to read `collections/{uid}/collection.json` when the owner's `public_profiles.enabled = true`.
- **collection_meta (own row)**: `auth.uid() = user_id` — users can only read and write their own metadata row.
- **collection_meta (public read for listed profiles)**: any client can SELECT rows where the owner has `public_profiles.enabled = true AND public_profiles.listed = true` — this powers the item count and last-updated columns on `shared.html`. SQL: `CREATE POLICY "Anyone can read meta of listed collections" ON collection_meta FOR SELECT TO public USING (EXISTS (SELECT 1 FROM public_profiles WHERE public_profiles.user_id = collection_meta.user_id AND public_profiles.enabled = true AND public_profiles.listed = true));`
- **public_profiles (own row)**: `auth.uid() = user_id` — users can INSERT/UPDATE/SELECT their own profile, including when `enabled = false`.
- **public_profiles (public read)**: any client can SELECT rows where `enabled = true` — this is how the public URL resolves `username → user_id`.

Never use the Supabase `service_role` key in client-side code — it bypasses all RLS policies.

### Changing Your Password

Click **Change password** in the signed-in panel of the Account modal. This reveals an inline form with two fields — **New password** and **Confirm new password** (both require a minimum of 8 characters). The Change password and Sign out buttons are hidden while the form is open.

- Click **Update password** to submit. The new password is applied immediately via Supabase Auth.
- If the two fields do not match, a red error message is shown and no request is sent.
- On success, a green confirmation message is shown and both fields are cleared.
- Click **Cancel** to discard the form and return to the normal signed-in view.

### Public Collection Sharing

See [Public Collections](#public-collections) for full details. The toggle in the signed-in panel of the Account modal controls whether your collection is accessible at a public URL.

- **Off (default)** — collection is private; only you can read it via cloud sync
- **On** — collection is readable by anyone who knows your URL; the Account modal shows the URL, a **Copy link** button, a **Disable** button, and a checkbox to opt into the [Publicly Shared Collections](#publicly-shared-collections-page) directory

The public URL is tied to a randomly generated username, not your email address.

### Signing Out

Click **Sign out** in the Account modal. The session is cleared from the browser and the header icon reverts to the default person icon. Your local collection is not affected.

---

## Light & Dark Mode

The theme toggle lives inside the **Settings Modal** under the Appearance section. It is a full-width row button that switches between light and dark mode. The button label reads "Switch to Dark Mode" when light mode is active, and "Switch to Light Mode" when dark mode is active. The icon changes between a sun (☀️) and moon (🌙) to match.

**Default:** Light mode.

**How it works:** A `data-theme` attribute is always set on the `<html>` element — `data-theme="light"` for light mode and `data-theme="dark"` for dark mode. CSS custom properties (variables) defined in `:root` apply the dark theme, and a `[data-theme="light"]` block overrides them for light mode.

**Persistence:** The chosen theme is saved as `ag_theme` in `localStorage` and restored on next visit.

### Color Palette

| Variable | Dark mode | Light mode |
|---|---|---|
| `--bg` | `#0c0c10` | `#f0f0f5` |
| `--surface` | `#16161e` | `#ffffff` |
| `--card` | `#1d1d27` | `#ffffff` |
| `--gold` | `#c9a435` | `#c9a435` |
| `--text` | `#e6e6f0` | `#18182a` |
| `--sub` | `#9090a8` | `#70708a` |
| `--green` | `#4caf84` | `#1e7a52` |
| `--red` | `#e05555` | `#c03030` |

---

## Import & Export

### Export

Click **Export Collection** in the Settings Modal (⚙️ → Data). The browser downloads a `.json` file named with today's date (e.g. `autographed-2024-11-15.json`).

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
      "valueUrl": "Harrison Ford autograph (bgs,bas,beckett,psa,dna,jsa,swau)",
      "notes": "Signed at Fan Expo 2023",
      "tags": ["sci-fi"],
      "signingEvent": "Fan Expo 2023",
      "signingDate": "2023-08-26",
      "acquiredHow": "In-Person",
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

This is also the format used by cloud sync — the file uploaded to Supabase Storage is identical to a local export.

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

## Footer

A footer is fixed to the bottom of the page and contains four navigation links: **Home** (the marketing page), **My Collection** (the app, `dashboard.html`), **Publicly Shared Collections** (`shared.html`), and **Blog**. It appears on all pages of the site.

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
- **Header buttons are smaller** — on viewports ≤480px, icon buttons shrink to 32×32px with a 4px gap to accommodate the added Account button without overflowing.

---

## Data Storage

autographed.app uses a **hybrid storage model**: lightweight item metadata and preferences live in `localStorage`, while the bulky photo data lives in **IndexedDB**. This keeps the collection from being capped by the small `localStorage` quota once you add lots of images. An optional third layer — **Supabase cloud storage** — provides cross-device backup.

### localStorage (metadata & preferences)

| Key | Type | Contents |
|---|---|---|
| `autographed_v2` | JSON string | Array of all item objects, **without** the `imgs` photo data (photos are stored separately in IndexedDB) |
| `ag_currency` | String | Local currency code (e.g. `"TWD"`) |
| `ag_display_currency` | String | Display currency code (e.g. `"USD"`) |
| `ag_rate_cache` | JSON string | Cached exchange rate: `{from, to, rate, ts}` |
| `ag_last_modified` | String | Unix timestamp (ms) of the last time `persist()` was called. Used by cloud sync to show "last changed" in the restore conflict dialog. |
| `ag_cols` | String | Saved columns per row (e.g. `"5"`) |
| `ag_view` | String | Saved view mode: `"grid"` or `"table"` |
| `ag_theme` | String | Saved theme: `"light"` or `"dark"` |
| `ag_photo_format` | String | Saved photo format key (e.g. `"8x10L"`) |
| `ag_img_fit` | String | Saved image fit mode: `"cover"`, `"contain"`, or `"blurbg"` |
| `ag_info_link` | String | Saved info link target: `"wiki"` or `"imdb"` |
| `ag_value_search` | String | Est. Value Search mode: `"ebay_sold"` (default) or `"ebay_active"` |
| `ag_privacy` | String | Saved privacy mode: `"off"` or `"on"` |
| `ag_sort` | String | Saved sort mode (e.g. `"date-desc"`, `"custom"`) |
| `ag_custom_order` | JSON string | Array of item IDs representing the user-defined custom sort sequence |
| `ag_lang` | String | Saved language preference (e.g. `"en"`, `"de"`, `"es"`, `"fr"`, `"zh-TW"`). Only present when the user has manually selected a language — absence means auto-detect from browser. |
| `ag_grid_image` | String | Grid card image visibility: `"1"` (visible) or `"0"` (hidden). Absent = visible. |
| `ag_grid_signer` | String | Grid card signer name visibility: `"1"` or `"0"`. Absent = visible. |
| `ag_grid_detail1` | String | Grid card Detail 1 field visibility: `"1"` or `"0"`. Absent = visible. |
| `ag_grid_detail2` | String | Grid card Detail 2 field visibility: `"1"` or `"0"`. Absent = visible. |
| `ag_grid_cert` | String | Grid card Cert # row visibility: `"1"` or `"0"`. Absent = visible. |
| `ag_grid_paid` | String | Grid card Paid row visibility: `"1"` or `"0"`. Absent = visible. |
| `ag_grid_value` | String | Grid card Est. Value row visibility: `"1"` or `"0"`. Absent = visible. |
| `ag_grid_roi` | String | Grid card ROI row visibility: `"1"` or `"0"`. Absent = visible. |
| `ag_desc_notes` | String | Whether notes are included in the auto-generated description: `"1"` (include) or `"0"` / absent (exclude, default). |
| `ag_export_reminder` | String | Whether the export reminder is enabled: `"1"` (on) or `"0"` / absent (off, default). |

### IndexedDB (photos)

| Database | Object store | Key | Value |
|---|---|---|---|
| `autographed` | `images` | item `id` | The item's `imgs` array (base64 JPEG data URLs) |

At runtime, each item still carries its `imgs` array in memory, so all rendering, export, import, and share logic is unchanged. Photos only cross the storage boundary in two places: `load()` reads them from IndexedDB and reattaches them to each item, and `persist()` writes metadata to `localStorage` and syncs photos to IndexedDB (deleting records for items that no longer exist).

### Supabase Cloud (optional)

When the user is signed in and clicks **Sync to cloud**, the full in-memory collection (metadata + photos) is serialized and uploaded as `{uid}/collection.json` to a private Supabase Storage bucket. A row in the `collection_meta` table records the `last_modified` timestamp and `item_count` for fast conflict checking without downloading the full file.

The cloud JSON format is identical to the local export format, so a cloud backup can also be downloaded and imported manually if needed.

### Automatic migration

On first load after upgrading from a localStorage-only version, autographed.app detects photos stored inline in `localStorage`, moves them into IndexedDB, and rewrites the `localStorage` item array without the image data. This happens transparently — no user action is needed, and nothing is lost.

autographed.app also automatically migrates items that used the older `character` and `film` field names (from before the Item Type redesign) to the new `detail1` and `detail2` names. This runs on first load and on import — the renamed fields are written back to storage immediately. No data is lost in the process.

### Storage limits

- **IndexedDB** typically allows hundreds of MB up to several GB per origin (browsers permit roughly 50–60% of free disk space), so photo-heavy collections are no longer constrained the way they were under `localStorage`.
- Item photos are still stored as base64-encoded JPEGs (compressed to max 900px at 0.82 quality), roughly 100–400 KB each.
- **Supabase Storage** (free tier / Spark plan) includes 1 GB of storage — sufficient for large collections. Each sync overwrites the previous backup so storage usage is bounded by the size of one collection snapshot.
- **Fallback:** if IndexedDB is unavailable (e.g. some private-browsing modes or very old browsers), autographed.app automatically falls back to the original behavior of storing photos inline in `localStorage`, and shows a notice. In that mode the older 5–10 MB `localStorage` limit applies.

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
| `certCompany` | string | Stored code for the cert type (e.g. `"beckett"`, `"psa-witnessed"`, `"mlb"`). See [Certification URL Derivation](#certification-url-derivation) for all valid values. Empty string means none. |
| `paid` | number \| null | Amount paid, in the local currency |
| `value` | number \| null | Estimated current value, in the local currency |
| `valueUrl` | string | eBay search term used to look up the item's estimated value (optional, may be empty string). Legacy items may contain a full URL — both forms are handled transparently by `buildValueUrl()`. |
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
- Supabase JS SDK (loaded lazily from CDN only when the Account modal is first opened — zero cost for anonymous users)

### Certification URL Derivation

Cert type codes map to display labels and verification URLs as follows:

| Code | Display label | Verification URL |
|---|---|---|
| `beckett` | Beckett Authenticated | `https://www.beckett-authentication.com/verify-certificate` |
| `beckett-witnessed` | Beckett Witnessed | `https://www.beckett-authentication.com/verify-certificate` |
| `psa` | PSA/DNA Authenticated | `https://www.psacard.com/cert` |
| `psa-witnessed` | PSA/DNA Witnessed | `https://www.psacard.com/cert` |
| `jsa` | JSA Authenticated | `https://www.spenceloa.com/verify-authenticity` |
| `jsa-witnessed` | JSA Witnessed | `https://www.spenceloa.com/verify-authenticity` |
| `gai` | GAI Authenticated | *(no public cert lookup — cert # shown as plain text)* |
| `gai-witnessed` | GAI Witnessed | *(no public cert lookup — cert # shown as plain text)* |
| `sgc` | SGC Authenticated | *(no public cert lookup — cert # shown as plain text)* |
| `aga` | AGA Authenticated | *(no public cert lookup — cert # shown as plain text)* |
| `swau` | SWAU Witnessed | `https://auth.swau.com/pages/verify-hologram` |
| `fanatics` | Fanatics Authentic | *(no public cert lookup — cert # shown as plain text)* |
| `leaf` | Leaf Authentic | *(no public cert lookup — cert # shown as plain text)* |
| `mlb` | MLB Authentication | `https://www.mlb.com/authentication` |
| `panini` | Panini Authentic | *(no public cert lookup — cert # shown as plain text)* |
| `steiner` | Steiner Sports | *(no public cert lookup — cert # shown as plain text)* |
| `tristar` | TriStar | *(no public cert lookup — cert # shown as plain text)* |
| `uda` | Upper Deck Authenticated | *(no public cert lookup — cert # shown as plain text)* |

The URL and display label are derived at render time from `item.certCompany` via `CERT_URLS` and `CERT_LABELS`. No URL or label is stored in the item data — only the code. When no URL exists for a cert type, the cert number is displayed as plain text rather than a clickable link.

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

### Supabase SDK Loading

The Supabase JS SDK (`@supabase/supabase-js`) is **not bundled** — it is loaded lazily from the jsDelivr CDN via a dynamically inserted `<script>` tag the first time the Account modal is opened. The load promise is cached in `_sbSdkPromise` so subsequent modal opens reuse the already-loaded script. Users who never open the Account modal incur zero SDK overhead.

On page load, `sbInit()` is called silently after the collection loads. This initializes the Supabase client and fires `onAuthStateChange`, which updates the header icon to show the user's initial if a session is already stored in the browser — without requiring the user to open the Account modal.

### Modal Behavior

Modals use a two-class system:
- `.overlay` — the full-screen backdrop (always in DOM, `opacity: 0`, `pointer-events: none`)
- `.overlay.open` — adds `opacity: 1` and `pointer-events: all`, animates the inner `.modal` from a slightly translated/scaled state to its natural position

Closing can happen via: the ✕ button, a Cancel button, a backdrop click (on the overlay itself, not the modal), or Delete confirmation.

> **Note:** The account modal and conflict dialog are placed in the HTML after the main `<script>` block. `applyI18n()` is therefore called a second time after `await load()` — at which point the full DOM is parsed — to ensure their `data-i18n` elements are translated correctly on page load.

### Sorting Implementation

Sort is applied after search filtering. Items without a value (`null`) are sorted to the bottom when sorting by value or paid. The selected sort is saved to `localStorage` (`ag_sort`) and restored on page load.

**Custom Order** is a special sort mode that allows manual sequencing. Moving an item via the +/− buttons updates the `ag_custom_order` array in `localStorage` and re-renders the gallery. The merge logic preserves the positions of items that are filtered out — only visible items shift when a move is performed. Items not yet present in `ag_custom_order` (e.g. newly added items) sort to the end.

### Click Propagation

Links inside cards and table rows (Est. Value link, Cert link) call `event.stopPropagation()` so that clicking them does not bubble up to the card/row click handler that would otherwise open the detail modal.

### Public Collection URL Detection

On page load, `checkPublicURL()` checks for a `?shared=<username>` query parameter. If found:

1. The query parameter is stripped from the URL immediately via `history.replaceState`
2. A full-page loading spinner appears ("Loading shared collection…")
3. The Supabase anon client queries `public_profiles WHERE username = ? AND enabled = true` to resolve the username to a `user_id`
4. The collection is downloaded from `collections/{user_id}/collection.json` in Supabase Storage (permitted by the public-read RLS policy for enabled profiles)
5. The JSON is parsed and the gallery enters [View Mode](#view-mode)

The Supabase SDK is initialized lazily if not already loaded (same `sbInit()` path as the Account modal). No authentication is required for the visitor.

### View Mode State

View mode is controlled by two module-level variables:

```javascript
let viewMode = false;          // true when viewing a public collection
let viewLocalCurrency = null;  // the collection owner's local currency, used for conversion
```

`getLocalCurrency()` returns `viewLocalCurrency` when `viewMode` is true, transparently making the exchange rate system use the owner's currency without touching `localStorage`. When `exitViewMode()` is called, both variables are reset and the now-async `load()` is awaited to restore your own items (metadata from `localStorage`, photos from IndexedDB).
