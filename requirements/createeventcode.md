# AMS Event Management — Requirements Document

**Version:** 2.0
**Date:** May 24, 2026
**Status:** Approved — HTML prototype is the solid reference
**Target Users:** Accounting Staff (Internal Bank)
**Prototype:** `prototypes/ams-event-management.html`

---

## Executive Summary

The AMS Event Management module allows accounting staff to create, configure, and manage accounting events. Each event maps an event code received from upstream systems to a GL Debit and GL Credit account, with optional reference fields and algorithm mappings for attribute derivation.

---

## System Overview

### Purpose
- Define accounting events (e.g., BPAY001 — รับชำระจากลูกค้า-เงินสด)
- Assign GL Debit and GL Credit accounts via smart search
- Configure algorithm mappings per GL side for attribute derivation
- Manage reference fields passed from upstream systems
- View, filter, and manage all events in a paginated table

### Key Users
- **Accounting Staff** — Create and configure events
- **Accounting Managers** — Review and activate/deactivate events

---

## Page Layout

### App Header (sticky, 56px)
- AMS logo (left) + "Accounting Management System" brand text
- User name + Log out button (right)
- Background: blue gradient `#003D99 → #0C447C`

### Page Header
- Title: "Event Management"
- Subtitle: "Configure event codes with GL accounts and algorithm mappings"
- No action button in page header — Create button is in the toolbar

### Toolbar
```
[ 🔍 Search events...        ]   [ Active ▼ ]  [ + Create New Event ]
```
- Search input (flex:1, max 420px) — filters by company code, event code, event name, GL code, GL name in real time
- Status filter dropdown: **Active** (default) / Inactive / All Status
- Create New Event button (primary blue, right)

### Table
- Sticky header — table body scrolls independently (`max-height: calc(100vh - 220px)`)
- Columns: Company Code | Event Code | Event Name | GL Debit Code | GL Debit Name | GL Credit Code | GL Credit Name | Status | Actions
- All data text in black (`#1A1D23`) — no colored data text
- Company Code: pill badge, neutral background
- Event Code: monospace, black
- GL codes: monospace, black
- Status: Active = green pill, Inactive = gray pill
- Inactive rows rendered at `opacity: 0.7`
- Actions column: **Edit** button only (no delete, no status toggle in table)
- Empty state message when no results

---

## Create / Edit Modal

Single modal used for both create and edit. Title changes accordingly.

### Modal Structure
- Header: blue gradient, title + subtitle + close (✕) button
- Body: 3 bordered layer cards (scrollable)
- Footer: Cancel + Save Event buttons
- Validation errors appear inside modal body (above layer cards), never behind backdrop

### Layer 1 — Event Card

**Layout (Create mode):**
```
[ Company Code ▼ ]
[ Event Code        ]  [ Event Name           ]
Reference Fields section
[ ] Require Internal Reference Number for this event
```

**Layout (Edit mode — Status toggle appears far right):**
```
[ Company Code ▼ ]                    Status  [ toggle ]
[ Event Code        ]  [ Event Name           ]
Reference Fields section
[ ] Require Internal Reference Number for this event
```

**Fields:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| Company Code | Dropdown | Yes | Options: 1000, 1010, 1020, 1030 |
| Event Code | Text (monospace) | Yes | Max 10 chars, auto-uppercase |
| Event Name | Text | Yes | Thai language |
| Require Internal Reference Number | Checkbox | No | Default unchecked |
| Status toggle | Toggle switch | Edit only | Active/Inactive; hidden in create mode; new events always start Active |

**Reference Fields (inside Layer 1):**
- Add/remove dynamically, unlimited entries
- Each row: Reference Field (dropdown, Reference1–Reference6) + Remark (text) + Mandatory/Optional (dropdown)
- Mandatory/Optional dropdown: **Mandatory** (default) / Optional → maps to `isMandatory: true/false`
- Empty state: "No reference fields added yet."

### Layer 2 — Debit Card
- Left border: blue (`#378ADD`), title "DEBIT" bold
- GL Account smart search: text input with SVG search icon, real-time dropdown filtering by code/name/type
- Selected GL displays as single line: `1211100100 - Cash and Cash Equivalents` (monospace, black, light blue background)
- ✕ button to clear selection
- Algorithm Mappings sub-section:
  - Add/remove dynamically, optional (zero or more)
  - Each row: Reference Field + Algorithm + Target Attribute + Default Value (text)
  - Summary line below each row showing selected labels
  - Empty state: "No algorithm mappings added yet."

### Layer 3 — Credit Card
- Left border: red (`#E24B4A`), title "CREDIT" bold
- Identical structure to Debit card, independent data

---

## Form Validation

Validation errors shown as red banner inside modal body (above layer cards).

| Rule | Error Message |
|------|---------------|
| Company Code, Event Code, or Event Name empty | "Please enter Event Code, select Company Code, and enter Event Name." |
| Debit or Credit GL not selected | "Please select both Debit and Credit GL accounts." |

Algorithm mappings are **not validated** — zero mappings is allowed.

---

## Status Management

- **Create:** No status toggle shown. New events always saved as `isActive: true`.
- **Edit:** Status toggle (Active/Inactive) shown in Layer 1 header, far right. Toggle state is saved on Save Event.
- **Table filter:** Defaults to Active. User can switch to Inactive or All Status.
- **No delete button** — events are deactivated via status toggle in edit modal only.

---

## Success Messages

After saving, a green banner appears on the page (auto-dismisses after 6 seconds):

- Create: `Event "[EventName]" created successfully!` + GL codes + algorithm counts
- Edit: `Event "[EventName]" updated successfully!` + GL codes + algorithm counts

---

## API Payload

Logged to browser console on every save with prefix `=== EVENT API PAYLOAD ===`.

```json
{
  "entityId": 1,
  "companyCode": "1000",
  "eventCode": "BPAY001",
  "eventName": "รับชำระจากลูกค้า-เงินสด",
  "isRequiredInternalNo": false,
  "isActive": true,
  "eventReferenceDetail": [
    {
      "referenceFieldId": 1,
      "isMandatory": true,
      "remark": "Required from lending system"
    }
  ],
  "eventDebitDetail": {
    "accountId": "1211100100",
    "attributeDetail": [
      {
        "referenceFieldId": 1,
        "algorithmId": 1,
        "defaultValue": "1002000000",
        "targetAttributeId": 1
      }
    ]
  },
  "eventCreditDetail": {
    "accountId": "2212000100",
    "attributeDetail": []
  }
}
```

**Key conventions:**
- IDs are numeric integers
- `isActive` included in payload (true/false)
- `isMandatory` in reference detail (true/false)
- `attributeDetail` is empty array `[]` when no mappings added
- GL code is 10-digit string (e.g., `1211100100`)

---

## Reference Data

### GL Accounts (10-digit codes)
| Code | Name | Type |
|------|------|------|
| 1211100100 | Short-term Receivables | Asset |
| 1211100200 | Trade Receivables | Asset |
| 1211200100 | Cash and Cash Equivalents | Asset |
| 1212000100 | Inventory | Asset |
| 2211100100 | Accounts Payable | Liability |
| 2212000100 | Long-term Loans | Liability |
| 2212100100 | Accrued Expenses | Liability |
| 4110000100 | Service Revenue | Revenue |
| 4111000100 | Interest Income | Revenue |
| 4120000100 | Fee Income | Revenue |
| 5110000100 | Operating Expenses | Expense |
| 5120000100 | Interest Expense | Expense |
| 5130000100 | Administrative Expenses | Expense |

### Reference Fields
Reference1 (id:1) through Reference6 (id:6)

### Algorithm Options
| ID | Label |
|----|-------|
| 1 | Fixed Value |
| 2 | Look Up Cost Center |
| 3 | Look Up Profit Center |

### Target Attribute Options
| ID | Label |
|----|-------|
| 1 | Profit Center |
| 2 | Cost Center |
| 3 | Customer |
| 4 | Vendor |

---

## Business Rules

1. Every event must belong to exactly one company code
2. Every event must have both a Debit GL and a Credit GL
3. Algorithm mappings are optional (zero or more per GL side)
4. Reference fields are optional
5. No delete — use Set Inactive via edit modal instead
6. New events are always created as Active
7. Status can only be changed via the Edit modal

---

## Future Enhancements

1. Batch import via CSV
2. Audit trail (created by / modified by / timestamp)
3. Prevent duplicate event codes per company
4. GL hierarchy browser
5. PDF export of event configuration
