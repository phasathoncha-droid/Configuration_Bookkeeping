# AMS Account Contract Configuration — Requirements Document

**Version:** 1.1
**Date:** 2026-05-24
**Status:** Approved — HTML prototype is the solid reference
**Target Users:** Accounting Staff (Internal Bank)
**Prototype:** `prototypes/ams-account-contract.html`

---

## Executive Summary

The AMS Account Contract Configuration module allows accounting staff to look up loan contracts and reassign each contract's branch unit code. Contracts originate from upstream systems and cannot be created or deleted through this module. Staff search for a contract by contract number or unit code, open the detail modal to view the current assignment and full assignment history, and save a new unit code assignment when a branch transfer is required.

---

## System Overview

### Purpose

- Look up loan contracts (approximately 1.2 million) by contract number or unit code
- Reassign a contract's branch unit code when a borrower's servicing branch changes
- Maintain a full, append-only audit trail of all unit code assignments per contract
- Activate or deactivate individual contracts via the edit modal

### Key Users

- **Accounting Staff** — Search for contracts and reassign branch unit codes
- **Accounting Managers** — Review assignment history and manage contract status

---

## Page Layout

### App Header (sticky, 56px)

- AMS logo (`assets/AMS_Logo.png`, embedded as base64, 32×32) + "Accounting Management System" brand text
- User name (`phasathon.cha@turbo.co.th`) + Log Out button (right)
- Background: blue gradient `#003D99 → #0C447C`

### Page Header

- Title: "Account Contract Configuration"
- No subtitle
- No action button in page header — there is no create button (contracts come from upstream)

### Toolbar

```
[ Contract Number ▼ ][ Enter contract number...        ][ 🔍 Search ]
```

- Search type dropdown (left, attached to input): **Contract Number** (default) / Unit Code
- Text input (flex, attached to dropdown): placeholder changes based on selected type — "Enter contract number..." or "Enter unit code..."
- Search button (primary blue with white search SVG icon + "Search" label, right, attached to input)
- No status filter dropdown
- No Clear button
- No create button

### Search Behaviour

- On load: table body shows a centered search prompt with a blue search icon — "Select a search type and enter a value, then click Search."
- Changing the search type dropdown: clears the input and returns to the search prompt
- Search fires on Search button click **or** pressing Enter in the text input
- If the input is empty when search fires: returns to the search prompt (does not query)
- Results always show **active records only** — inactive contracts are never shown regardless of search
- Matching is case-insensitive substring match on `contractNumber` (when type = Contract Number) or `unitCode` (when type = Unit Code)

### Table

- Sticky header — table body scrolls independently (`max-height: calc(100vh - 280px)`)
- Columns: Contract Number | Current Unit Code | Status | Actions

| Column | Notes |
|--------|-------|
| Contract Number | Monospace (Courier New), black |
| Current Unit Code | Monospace (Courier New), black |
| Status | Active = green pill only (inactive contracts are never shown) |
| Actions | Single **View** button (btn-edit style) — no delete button |

- All data text in black (`#1A1D23`) — no colored data text
- Empty state (no search entered): centered prompt with search icon — "Select a search type and enter a value, then click Search."
- Empty state (search yields no results): centered message — "No active records found."

---

## Detail / Edit Modal

Single modal for viewing and editing contract assignment. No create mode exists.

### Modal Structure

- Header: blue gradient, title "Contract Details" + subtitle showing the contract number (e.g. "Contract #100000100") + ✕ close button; backdrop click also closes
- Body: two bordered layer cards, validation alert area at top (`#acModalAlertArea`)
- Footer: Cancel + Save buttons (right-aligned)
- Validation errors appear inside the modal body (above the layer cards, inside `#acModalAlertArea`), never at page level

### Modal Titles

| Field | Value |
|-------|-------|
| Title | "Contract Details" |
| Subtitle | "Contract #\<contractNumber\>" (e.g. "Contract #100000100") |

### Layer Card 1 — Current Assignment

**Layout:**
```
CURRENT ASSIGNMENT                             Status [ toggle ]
────────────────────────────────────────────────────────────────
Unit Code (smart search — Branch units only)
```

- Status toggle always visible (edit-only module — no create mode)
- Unit code smart search field (required to save)

#### Unit Code Smart Search

| State | Appearance |
|-------|-----------|
| Empty / typing | Text input with SVG search icon (left); placeholder "Type to search branch unit code or name..."; real-time dropdown filtered by code or name |
| Dropdown option | `CODE — Unit Name` one per row |
| Selected | Light blue chip (`#E8F2FF` bg, blue border) with `CODE — Unit Name` in monospace and a ✕ clear button |

- Dropdown displays Branch-type active units only
- Clicking outside the dropdown closes it
- ✕ on chip clears selection and restores the input
- When modal opens, the contract's current unit code is pre-selected

### Layer Card 2 — Assignment History (read-only)

**Layout:**
```
ASSIGNMENT HISTORY
────────────────────────────────────────────────────────────────
Table: Unit Code | Unit Name | Effective Date | Created By
```

- No action buttons in this card
- History rows sorted by Effective Date ascending
- Compact rows (13px font)
- Unit Code column: monospace
- Effective Date: formatted "DD MMM YYYY" (e.g. "15 Jan 2024")
- The most recent (last) history row displays a small "Current" badge (light blue pill, inline after Unit Name)
- Table has a scrollable wrapper (`max-height: ~220px`, `overflow-y: auto`)
- Empty state when no history: centered message "No history available."

### Modal Footer

| Button | Style | Action |
|--------|-------|--------|
| Cancel | btn-secondary | Close modal without saving |
| Save | btn-primary | Validate and persist assignment |

---

## Form Validation

Validation errors shown as a red banner inside `#acModalAlertArea` at the top of the modal body (above the layer cards). Only the first failing rule is displayed. The banner has a ✕ dismiss button.

| # | Rule | Error Message |
|---|------|---------------|
| 1 | No unit code selected | "Please select a Unit Code." |

---

## Status Management

- Status toggle is always shown in the modal (edit-only module — no create mode)
- Toggle state (Active / Inactive) is saved on Save
- No delete button — contracts are deactivated via the status toggle in the modal only
- No status filter on the page — the table always shows active records only

---

## Success Messages

After saving, a toast notification appears fixed at the top-right of the screen (auto-dismisses after 4 seconds, with slide-in/slide-out animation and a manual ✕ dismiss button):

- Edit: `Contract "[ContractNumber]" updated successfully!`

Toast structure: green checkmark icon circle + message body + ✕ close button.

---

## API Payload

Logged to browser console on every save with prefix `=== ACCOUNT CONTRACT API PAYLOAD ===`.

```json
{
  "contractNumber": "100000100",
  "unitCode": "NBI002",
  "isActive": true,
  "effectiveDate": "2026-05-24"
}
```

**Key conventions:**

- `contractNumber`: string — the loan contract identifier from upstream
- `unitCode`: string — the newly assigned branch unit code
- `isActive`: boolean (`true` / `false`)
- `effectiveDate`: ISO date string (`YYYY-MM-DD`), computed as today's date at save time

---

## Business Rules

1. Contracts cannot be created or deleted through this module — they originate from upstream systems
2. Users may only reassign the unit code (branch); no other contract attributes are editable here
3. Every save appends a new row to the assignment history — existing history rows are never modified or deleted
4. The unit code smart search lists only Branch-type active units from the Organization Unit module
5. A unit code selection is required before saving; attempting to save without one triggers a validation error
6. `effectiveDate` is always today's date at the moment the user saves — it is not user-editable
7. `createdBy` is always the logged-in user (`phasathon.cha`) — it is set server-side and logged in the prototype as a constant
8. The table shows no data on load; the user must select a search type, enter a value, and click Search before results appear
9. The table always shows active records only — inactive contracts are never displayed regardless of search input
10. Status can only be changed via the modal toggle; no inline status controls exist on the table

---

## Future Enhancements

1. Server-side pagination and search for the full 1.2-million-contract dataset
2. Bulk reassignment — select multiple contracts and reassign to a single unit code in one operation
3. Export assignment history to Excel / PDF for audit purposes
4. Integration with the Organization Unit module to automatically reflect unit code renames in history display
5. Date-effective scheduling — allow users to set a future effective date for a planned branch transfer
6. Advanced search filters — by effective date range, created-by user, or unit type
