# AMS Organization Unit — Requirements Document

**Version:** 1.1
**Date:** 2026-05-24
**Status:** Approved — HTML prototype is the solid reference
**Target Users:** Accounting Staff (Internal Bank)
**Prototype:** `prototypes/ams-organization-unit.html`

---

## Executive Summary

The AMS Organization Unit module allows accounting staff to create, configure, and manage organizational units (Branches, Departments, and Employees). Each unit is assigned a Profit Center code (10-digit) and a Cost Center code (8-digit) to support GL attribute derivation. Staff can view, filter, create, and edit units from a single page, and activate or deactivate units via the edit modal.

---

## System Overview

### Purpose

- Define organizational units across the bank (branches, departments, individual employees)
- Assign Profit Center and Cost Center codes to each unit for downstream GL processing
- Manage unit lifecycle (active / inactive) without permanent deletion
- Provide a clean lookup table for center codes used by algorithm mappings in other modules

### Key Users

- **Accounting Staff** — Create and configure organizational units
- **Accounting Managers** — Review, activate, and deactivate units

---

## Page Layout

### App Header (sticky, 56px)

- AMS logo (`assets/AMS_Logo.png`, embedded as base64, 32×32) + "Accounting Management System" brand text
- User name + Log Out button (right)
- Background: blue gradient `#003D99 → #0C447C`

### Page Header

- Title: "Organization Unit Configuration"
- No subtitle
- No action button in page header — Create button is in the toolbar

### Toolbar

```
[ 🔍 Search units...   ]   [ Active ▼ ]  [ + Create New Unit ]
```

- Search input (flex:1, max 420px) — filters by Unit Code, Unit Name, Unit Type label in real time
- Status filter dropdown: **Active** (default) / Inactive / All Status
- Create New Unit button (primary blue, right)

### Table

- Sticky header — table body scrolls independently (`max-height: calc(100vh - 280px)`)
- Columns: Unit Code | Unit Name | Unit Type | Profit Center | Cost Center | Status | Actions
- All data text in black (`#1A1D23`) — no colored data text
- Unit Code: monospace (Courier New), black
- Unit Name: standard body text, black
- Unit Type: plain text label — "Branch", "Department", or "Employee"
- Profit Center: monospace, black; blank cell if not configured
- Cost Center: monospace, black; blank cell if not configured
- Status: Active = green pill, Inactive = gray pill (dot prefix, 11px, weight 600)
- Inactive rows rendered at `opacity: 0.7`
- Actions column: **Edit** button only (no delete, no inline status toggle)
- Empty state: centered message "No records found." when no results match filters

---

## Create / Edit Modal

Single modal reused for both create and edit modes. Title and behavior change accordingly.

### Modal Structure

- Header: blue gradient, title + subtitle + close (✕) button; backdrop click also closes
- Body: one bordered layer card ("Unit Details"), validation alert area at top
- Footer: Cancel + Save Unit buttons (right-aligned)
- Validation errors appear inside the modal body (above the layer card), never at page level

### Modal Titles

| Mode | Title | Subtitle |
|------|-------|---------|
| Create | "Create New Organization Unit" | "Add a new unit to the system" |
| Edit | "Edit Organization Unit" | "Update unit details" |

### Layer Card — Unit Details

**Create mode layout:**
```
Unit Details
─────────────────────────────────────────────────
[ Unit Code                                      ]
[ Unit Name                ] [ Unit Type ▼       ]
[ Profit Center            ] [ Cost Center       ]
```

**Edit mode layout (Status toggle in layer card header, far right):**
```
Unit Details                        Status [ toggle ]
─────────────────────────────────────────────────────
[ Unit Code                                          ]
[ Unit Name                ] [ Unit Type ▼           ]
[ Profit Center            ] [ Cost Center           ]
```

### Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| Unit Code | Text (monospace) | Yes | Max 20 chars; auto-uppercase on input |
| Unit Name | Text | Yes | Supports Thai characters |
| Unit Type | Dropdown | Yes | Options in order: Branch (1) / Department (3) / Employee (2) |
| Profit Center | Text (monospace) | Yes | Exactly 10 digits; placeholder "10-digit code"; hint "Exactly 10 digits"; maxlength 10; inputmode numeric |
| Cost Center | Text (monospace) | Yes | Exactly 8 digits; placeholder "8-digit code"; hint "Exactly 8 digits"; maxlength 8; inputmode numeric |
| Status toggle | Custom CSS toggle | Edit only | Active / Inactive; hidden in create mode; new units always start Active |

---

## Form Validation

Validation errors shown as a red banner inside `#ouModalAlertArea` at top of modal body (above the layer card).
Errors are checked in order; only the first failing error is displayed.
The banner has a ✕ dismiss button.

| # | Rule | Error Message |
|---|------|---------------|
| 1 | Unit Code, Unit Name, or Unit Type missing or empty | "Please fill in all required fields." |
| 2 | Profit Center is not exactly 10 digits | "Profit Center code must be exactly 10 digits." |
| 3 | Cost Center is not exactly 8 digits | "Cost Center code must be exactly 8 digits." |

---

## Status Management

- **Create:** No status toggle shown. New units always saved as `isActive: true`.
- **Edit:** Status toggle (Active / Inactive) shown in the layer card header, far right. Toggle state is saved on Save Unit.
- **Table filter:** Defaults to Active. User can switch to Inactive or All Status.
- **No delete button** — units are deactivated via the status toggle in the edit modal only.

---

## Success Messages

After saving, a green banner appears on the page (auto-dismisses after 6 seconds):

- Create: `Unit "[UnitName]" created successfully!`
- Edit: `Unit "[UnitName]" updated successfully!`

---

## API Payload

Logged to browser console on every save with prefix `=== ORGANIZATION UNIT API PAYLOAD ===`.

**Create:**
```json
{
  "unitCode": "NBI016",
  "unitName": "บ้านกล้วย",
  "unitTypeId": 1,
  "isActive": true,
  "unitCenterDetail": [
    { "centerTypeId": 1, "centerCode": "1005005100" },
    { "centerTypeId": 2, "centerCode": "10050051" }
  ]
}
```

**Edit (deactivated unit):**
```json
{
  "unitCode": "EMP001",
  "unitName": "นายสมชาย ใจดี",
  "unitTypeId": 2,
  "isActive": false,
  "unitCenterDetail": [
    { "centerTypeId": 1, "centerCode": "3000000001" },
    { "centerTypeId": 2, "centerCode": "30000001" }
  ]
}
```

**Key conventions:**

- `unitTypeId`: 1 = Branch, 2 = Employee, 3 = Department (integer)
- `centerTypeId`: 1 = Profit Center, 2 = Cost Center (integer)
- `centerCode`: string — Profit Center is exactly 10 digits; Cost Center is exactly 8 digits
- `isActive`: boolean (`true` / `false`)
- IDs are numeric integers
- `unitCenterDetail` always contains exactly 2 entries: Profit Center first, then Cost Center

---

## Business Rules

1. Unit Code is unique within the system (enforced by accounting staff convention; no duplicate check in prototype)
2. Every unit must have both a Profit Center and a Cost Center assigned
3. Profit Center code must be exactly 10 numeric digits
4. Cost Center code must be exactly 8 numeric digits
5. Unit Type is one of: Branch (1), Employee (2), Department (3)
6. No delete — use Set Inactive via the edit modal status toggle instead
7. New units are always created as Active (`isActive: true`)
8. Status can only be changed via the Edit modal

---

## Future Enhancements

1. Batch import of units via CSV upload
2. Audit trail (created by / modified by / timestamp)
3. Duplicate Unit Code prevention with server-side validation
4. Hierarchical unit structure (parent branch → sub-branch relationships)
5. Integration with GL algorithm mappings to auto-suggest center codes
6. Export to Excel / PDF
