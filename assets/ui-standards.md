# AMS Design System — Master UI Standards

**Version:** 1.0
**Date:** May 24, 2026
**Source of truth:** `prototypes/ams-event-management.html`

All prototypes must follow these standards. Do not hardcode one-off values.

---

## Colors

```css
--primary:        #378ADD   /* buttons, links, focus rings */
--primary-dark:   #0C447C   /* hover states, debit label */
--primary-darker: #003D99   /* header gradient start */
--primary-light:  #E8F2FF   /* selected GL background, badge bg */
--success:        #639922
--success-light:  #F0F7E6
--danger:         #E24B4A   /* credit label, error states */
--danger-light:   #FEF2F2
--warning:        #D97706
--warning-light:  #FFFBEB
--bg-primary:     #FFFFFF
--bg-secondary:   #F8F9FB   /* page background, table header */
--border:         #E0E3E8
--text-primary:   #1A1D23   /* all data text — always black */
--text-secondary: #6B7280   /* labels, placeholders, hints */
```

---

## Typography

```css
font-family: 'Sarabun', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
/* Import: @import url('https://fonts.googleapis.com/css2?family=Sarabun:wght@400;500;600;700;800&display=swap'); */
```

| Usage | Size | Weight |
|-------|------|--------|
| Page title | 22px | 600 |
| Section / layer title | 11px, uppercase, letter-spacing 0.6px | 700 |
| Form labels | 13px | 600 |
| Body / table data | 13–14px | 400 |
| Small hints | 11–12px | 400 |
| GL codes / event codes | Courier New, monospace, 12–13px | 600 |

---

## App Header

- Height: 56px, sticky, `z-index: 100`
- Background: `linear-gradient(135deg, #003D99 0%, #0C447C 100%)`
- Shadow: `0 2px 8px rgba(0,0,0,0.2)`
- Left: AMS logo (32×32) + "Accounting Management System" (15px, white, weight 600)
- Right: user name (13px, white 80% opacity) + Log Out button

### AMS Logo

Use the PNG logo embedded as base64 — **do not use an SVG placeholder**. Source file: `assets/AMS_Logo.png`.

```html
<div class="brand-icon">
  <img src="data:image/png;base64,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" alt="AMS Logo" style="width:32px;height:32px;object-fit:contain;" />
</div>
```

---

## Buttons

| Class | Style | Usage |
|-------|-------|-------|
| `.btn-primary` | Blue bg `#378ADD`, white text | Primary action |
| `.btn-secondary` | `#F8F9FB` bg, border, dark text | Cancel / secondary |
| `.btn-edit` | `#F8F9FB` bg, blue text, border | Table row Edit |
| `.btn-add-inline` | Transparent, dashed blue border | Add rows inline |
| `.btn-danger-ghost` | Transparent, red text | Remove rows |
| `.btn-logout` | White 12% bg, white border | Header logout |

- Border-radius: 8px
- Font-size: 13px, weight 600
- Padding large: `0.625rem 1.25rem`
- Padding small (`.btn-sm`): `0.375rem 0.75rem`, font 12px weight 500

---

## Form Elements

- Border-radius: 8px
- Border: `1px solid #E0E3E8`
- Focus: `border-color: #378ADD` + `box-shadow: 0 0 0 3px rgba(55,138,221,0.15)`
- Font: inherit Sarabun
- Labels: 13px, weight 600, `color: #1A1D23`
- Required asterisk: `<span class="req">*</span>` in red

---

## Table

- `border-collapse: collapse`, full width
- Header: `background: #F8F9FB`, `border-bottom: 2px solid #E0E3E8`
- Header cells: 11px, uppercase, letter-spacing 0.5px, weight 700, color `#6B7280`
- Rows: `border-bottom: 1px solid #E0E3E8`, hover `#FAFBFD`
- Cell padding: `13px 16px`
- Sticky header: `thead th { position: sticky; top: 0; z-index: 1; }`
- Scrollable body: `max-height: calc(100vh - 220px)`, `overflow-y: auto`
- All data text: black (`#1A1D23`) — no colored data

### Status Badges
```css
/* Active */  background: #ECFDF5; color: #065F46; dot: #10B981
/* Inactive */ background: #F3F4F6; color: #6B7280; dot: #9CA3AF
```
Pill shape, dot prefix, 11px, weight 600.

---

## Toolbar Pattern

```
[ 🔍 Search...  (flex:1, max 420px) ]   [ Status ▼ ]  [ + Primary Action ]
```
- Search: left-side, search SVG icon (transparent, no background), clear ✕ button
- Status filter dropdown: default Active
- Primary action button: far right

---

## Modal

- Backdrop: `rgba(0,0,0,0.45)`, `z-index: 200`
- Modal: `z-index: 201`, `border-radius: 14px`, max-width 860px
- Header: blue gradient, white title (18px weight 700) + subtitle (12px) + ✕ close
- Body: `padding: 1.5rem`, scrollable
- Footer: `background: #F8F9FB`, border-top, Cancel + Save buttons right-aligned
- Validation errors: red banner **inside modal body** (id `modalAlertArea`), never page-level

---

## Layer Cards (Modal Sections)

Used to visually separate major form sections inside a modal.

```css
.layer-card {
  border: 1px solid #E0E3E8;
  border-radius: 10px;
  padding: 1.25rem;
  margin-bottom: 1rem;
  background: #FFFFFF;
}
.layer-card-header {
  display: flex; justify-content: space-between; align-items: center;
  border-bottom: 2px solid #E0E3E8;
  padding-bottom: 8px; margin-bottom: 1rem;
}
.layer-card-title {
  font-size: 11px; font-weight: 700;
  text-transform: uppercase; letter-spacing: 0.6px;
}
/* Debit card */
.layer-card.debit-card  { border-left: 3px solid #378ADD; }
.layer-card.debit-card  .layer-card-title { color: #0C447C; font-size: 13px; font-weight: 800; }
/* Credit card */
.layer-card.credit-card { border-left: 3px solid #E24B4A; }
.layer-card.credit-card .layer-card-title { color: #E24B4A; font-size: 13px; font-weight: 800; }
```

---

## GL Smart Search

- Text input with transparent SVG search icon (left)
- Real-time filtering on code, name, type
- Dropdown: white bg, border, shadow, max-height 220px scrollable
- Each option: `CODE — Name` + type badge
- Selected state: single line `CODE - Name`, monospace, black text, light blue bg (`#E8F2FF`), blue border, ✕ clear button

---

## Status Toggle (Edit Modal)

Custom CSS toggle switch — no native checkbox appearance.

- Track: 44×24px, border-radius 12px
- Active: track `#378ADD`, thumb at `left: 23px`
- Inactive: track `#9CA3AF`, thumb at `left: 3px`
- Thumb: 18×18px white circle, shadow
- Label: "Active" / "Inactive" text beside toggle
- Position: inside layer-card-header, far right — same row as Company Code field

---

## Alerts

### Page-level (success after modal close)
- Green banner, auto-dismiss after 6 seconds
- Position: below page header, above toolbar

### Modal-level (validation errors)
- Red banner inside `#modalAlertArea` at top of modal body
- Has ✕ dismiss button
- Cleared on `resetForm()`

---

## Search Icon

SVG magnifier, transparent background, stroke `#378ADD`:

```html
<svg width="13" height="13" viewBox="0 0 13 13" fill="none">
  <circle cx="5.5" cy="5.5" r="4" stroke="#378ADD" stroke-width="1.5"/>
  <line x1="8.7" y1="8.7" x2="11.5" y2="11.5" stroke="#378ADD" stroke-width="1.5" stroke-linecap="round"/>
</svg>
```

---

## Spacing & Layout

- Page container: `max-width: 1400px`, `padding: 2rem`
- Section gap: `1rem` between layer cards
- Form grid gaps: `1rem`
- Dividers: `1px solid #E0E3E8`
