# AMS Bookkeeping — Project Guide for Claude

## What This Project Is

Internal bookkeeping system for a bank. The system receives **event codes** and **references** from upstream systems, then maps each event to GL Debit/Credit accounts based on configuration managed by accounting staff.

This repository is a **UI prototype workspace**. The deliverables are:
1. A `.md` requirement document per feature (in `requirements/`)
2. A self-contained `.html` prototype per feature (in `prototypes/`)

---

## Folder Structure

```
Bookkeeping/
├── CLAUDE.md                        ← this file
├── requirements/                    ← one .md file per feature
│   └── createeventcode.md           ← Event Code config (done)
├── prototypes/                      ← one .html file per feature
│   └── ams-event-management.html
└── assets/
    ├── ui-standards.md              ← Master UI/Design System (SOURCE OF TRUTH)
    └── AMS_Logo.png                 ← AMS logo (embedded as base64 in prototypes)
```

---

## How We Work

### When the user gives a new feature request:

1. **Ask clarifying questions first** — do not start drafting until ambiguities are resolved:
   - Who are the users and what is their goal?
   - What are the required vs optional fields?
   - Is there an API payload structure?
   - Any business rules or constraints?

2. **Write the requirement doc** (`requirements/<featurename>.md`) using the same structure as `createeventcode.md`:
   - Executive Summary
   - System Overview (Purpose + Key Users)
   - Page Layout & UI Requirements (sections, fields, layout)
   - Form Validation (required fields, error messages, success message)
   - API Payload Structure (JSON example)
   - Business Rules & Constraints
   - Future Enhancements
   - *(Do NOT include a UI/UX Design Standards section — that lives in `assets/ui-standards.md`)*

3. **Build the HTML prototype** (`prototypes/<featurename>.html`):
   - Single self-contained `.html` file (no external dependencies except Google Fonts)
   - **MUST follow `assets/ui-standards.md` for all colors, typography, components, and patterns — do not hardcode one-off values**
   - All CRUD operations work in-memory (no backend needed)
   - Log API payload to browser console on save (`=== FEATURE API PAYLOAD ===`)
   - Open in Google Chrome after creation: `open -a "Google Chrome" prototypes/<file>.html`

---

## Design System

> **The full design system is in `assets/ui-standards.md`. Always read it before building any prototype.**

Quick reference only — see `ui-standards.md` for complete specs:

### Colors
```
--primary:        #378ADD
--primary-dark:   #0C447C
--primary-darker: #003D99
--primary-light:  #E8F2FF
--success:        #639922
--danger:         #E24B4A
--bg-primary:     #FFFFFF
--bg-secondary:   #F8F9FB
--border:         #E0E3E8
--text-primary:   #1A1D23
--text-secondary: #6B7280
```

### Typography
- Font: `'Sarabun', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif`
- Import: `@import url('https://fonts.googleapis.com/css2?family=Sarabun:wght@400;500;600;700;800&display=swap');`
- Monospace (GL codes, event codes): `Courier New, monospace`

### Standard UI Patterns (reuse across all prototypes)
- **Table toolbar**: search input (left, flex:1) + status filter dropdown + action button (right)
- **Status badges**: Active = green pill, Inactive = gray pill
- **Status filter**: dropdown (Active / Inactive / All Status), default Active
- **GL smart search**: SVG search icon (transparent), live dropdown, selected = single line `CODE - Name` in light blue chip
- **Modal**: backdrop z-index 200, modal z-index 201, blue gradient header, 3 layer cards in body
- **Layer cards**: bordered sections inside modal — use for logical grouping (Event / Debit / Credit)
- **Validation errors**: red banner inside `#modalAlertArea` at top of modal body — never page-level
- **Success alerts**: green banner in page alert area, auto-dismiss after 6s
- **Form inputs**: border-radius 8px, focus = primary blue border + shadow
- **No icons in modal section titles or field labels**
- **All data text is black** (`--text-primary`) — no colored text on table data
- **Status toggle**: custom CSS switch, edit modal only — shown far right on same row as Company Code

---

## API Conventions

- IDs are numeric integers
- All reference field options: `Reference1` through `Reference6` (id 1–6)
- Algorithm options: `Fixed Value` (1), `Look Up Profit Center by Unit Code` (2), `Look Up Cost Center by Unit Code` (3), `Look Up Profit Center by Contract Number` (4), `Look Up Cost Center by Contract Number` (5)
- Target attribute options: `Profit Center` (1), `Cost Center` (2), `Customer` (3), `Vendor` (4)
- Company codes: `1000`, `1010`, `1020`, `1030`
- GL codes are 10-digit strings (e.g., `1211100100`)
- Boolean flags: `isRequiredInternalNo`, `isMandatory`, `isActive`

---

## Completed Features

| Feature | Requirement | Prototype |
|---------|-------------|-----------|
| Event Code Config | `requirements/createeventcode.md` | `prototypes/ams-event-management.html` |

---

## Key Decisions & Rules (from past sessions)

- Algorithm mappings are **optional** — do not validate that at least one exists
- Validation errors must appear **inside the modal**, not in the page-level alert area
- **No delete button** on the event list — Set Inactive / Set Active is done via edit modal only
- Status toggle is **hidden in create mode** — new events always start as Active (`isActive: true`)
- Save captures GL references into local variables **before** calling `closeModal()` to avoid null reference crash
- `tableResultCount` element was removed — do not reference it in JS
- All icons removed from modal section titles and field labels
- Prototype data is stored in JS memory (session only) — no localStorage, no backend
- Mandatory/Optional in reference fields is a **dropdown** (not a checkbox)
- Reference fields section lives **inside the Event layer card**, not as a separate section
