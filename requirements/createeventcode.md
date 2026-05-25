# AMS Event Management — Requirements Document

**Version:** 3.0
**Date:** May 25, 2026
**Status:** Approved — HTML prototype is the solid reference
**Target Users:** Accounting Staff (Internal Bank)
**Prototype:** `prototypes/ams-event-management.html`

---

## Executive Summary

The AMS Event Management module allows accounting staff to create, configure, and manage accounting events. Each event maps an event code received from upstream systems to a GL Debit and GL Credit account, with optional reference fields and algorithm mappings for attribute derivation. Events can be created individually via modal or in bulk via file upload.

---

## System Overview

### Purpose
- Define accounting events (e.g., BPAY001 — รับชำระจากลูกค้า-เงินสด)
- Assign GL Debit and GL Credit accounts via smart search
- Configure algorithm mappings per target attribute type (Profit Center, Cost Center, Customer, Vendor) — shown only when the selected GL is configured for that attribute
- Manage reference fields (Ref1–Ref6) passed from upstream systems
- View, filter, and manage all events in a paginated table
- Bulk-create events via file upload (TSV/CSV)

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

### Toolbar
```
[ 🔍 Search events...        ]   [ Active ▼ ]  [ Upload ]  [ + Create New Event ]
```
- Search input (flex:1, max 420px) — filters by company code, event code, event name, GL code, GL name in real time
- Status filter dropdown: **Active** (default) / Inactive / All Status
- Upload button (secondary) — opens bulk upload modal
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
[ Event Code        ]  [ Event Name                          ]
─── Reference Fields ───────────────────────────────────────
[ Reference1 ] [ Reference2 ] [ Reference3 ] [ Reference4 ] [ Reference5 ] [ Reference6 ]  (chip toggles)
─── config rows for active chips ───────────────────────────
┌─────────────────────────────────────────────────────────┐
│  Require Internal Reference Number     [ toggle OFF ]   │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│  Auto Post to SAP                      [ toggle ON  ]   │
└─────────────────────────────────────────────────────────┘
```

**Layout (Edit mode — Status toggle appears far right):**
```
[ Company Code ▼ ]                          Status  [ toggle ]
[ Event Code        ]  [ Event Name                          ]
─── Reference Fields ────────────────────────────────────────
...same as create...
```

**Fields:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| Company Code | Dropdown | Yes | Options: 1000, 1010, 1020, 1030 |
| Event Code | Text (monospace) | Yes | Max 10 chars, auto-uppercase |
| Event Name | Text | Yes | Thai/English |
| Require Internal Reference Number | Toggle card | No | Default OFF |
| Auto Post to SAP | Toggle card | No | Default **ON** |
| Status toggle | Toggle switch | Edit only | Active/Inactive; hidden in create mode; new events always start Active |

**Reference Fields (chip toggles inside Layer 1):**

- Six chips displayed: **Reference1 Reference2 Reference3 Reference4 Reference5 Reference6**
- **Reference1** and **Reference6** are always active (blue, non-deselectable) and always mandatory
- **Reference2–Reference5** are toggleable by user — click to activate/deactivate
- Each active reference chip shows a config row below with:
  - Remark (text input)
  - Mandatory / Optional dropdown — default **Mandatory**
- Reference1 and Reference6 mandatory dropdowns are disabled (locked to Mandatory)
- Empty state when no optional refs activated: config area shows nothing extra

### Layer 2 — Debit Card
- Left border: blue (`#378ADD`), title "DEBIT" bold
- GL Account smart search: text input with SVG search icon, real-time dropdown filtering by code/name/type
- Selected GL displays as single line: `1211100100 - Short-term Receivables` (monospace, black, light blue background chip with ✕)
- **Target Attribute Configuration** — rendered dynamically based on selected GL:
  - Only rows matching the GL's `trackingDimensionId` and `subLedgerTypeId` are shown
  - `trackingDimensionId = 1` → show **Profit Center** row
  - `trackingDimensionId = 2` → show **Cost Center** row
  - `subLedgerTypeId = 1` → show **Customer** row
  - `subLedgerTypeId = 2` → show **Vendor** row
  - `subLedgerTypeId = 3` → no subledger row shown
  - If no GL selected: placeholder "Select a GL account to see target attribute configuration"
  - Each row has: Algorithm dropdown + conditional Reference Field dropdown (Look Up algorithms) or Fixed Value input (Fixed Value algorithm)

**Algorithm options per target attribute type:**

| Target Attribute | Available Algorithms |
|-----------------|---------------------|
| Profit Center | Fixed Value, Look Up Profit Center by Branch Code, Look Up Profit Center by Contract Number |
| Cost Center | Fixed Value, Look Up Cost Center by Branch Code, Look Up Cost Center by Contract Number |
| Customer | Fixed Value |
| Vendor | Fixed Value |

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

Target attribute configurations and reference fields are **not validated** — zero configs is allowed.

---

## Status Management

- **Create:** No status toggle shown. New events always saved as `isActive: true`.
- **Edit:** Status toggle (Active/Inactive) shown in Layer 1 header, far right. Toggle state saved on Save Event.
- **Table filter:** Defaults to Active. User can switch to Inactive or All Status.
- **No delete button** — events are deactivated via status toggle in edit modal only.

---

## Success Messages

After saving, a green toast appears on the page (auto-dismisses after 4 seconds):

- Create: `Create successfully`
- Edit: `Edit successfully`

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
  "isAutoPostSAP": true,
  "isActive": true,
  "eventReferenceDetail": [
    {
      "referenceFieldId": 1,
      "isMandatory": true,
      "remark": "Transaction ID from lending system"
    },
    {
      "referenceFieldId": 2,
      "isMandatory": true,
      "remark": "Loan contract ref"
    }
  ],
  "eventDebitDetail": {
    "accountId": "1211100100",
    "attributeDetail": [
      {
        "targetAttributeId": 1,
        "algorithmId": 2,
        "referenceFieldId": 2,
        "fixedValue": null
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
- `isActive` and `autoPostToSAP` are boolean (true/false)
- `isMandatory` in reference detail is boolean
- `attributeDetail` is empty array `[]` when no mappings configured
- For Fixed Value algorithm: `referenceFieldId: null`, `fixedValue: "value"`
- For Look Up algorithms: `referenceFieldId: <id>`, `fixedValue: null`
- GL code is 10-digit string

---

## Bulk Upload

### Overview
Users can bulk-create events by uploading a tab-separated (TSV) or comma-separated (CSV) file. This is designed for new company onboarding scenarios where many events need to be created at once.

### File Format
- Tab-separated (TSV) or comma-separated (CSV)
- First row = header row
- All column headers are case-insensitive
- Compatible with Google Sheets copy-paste

### Column Definitions

| No | Column Header | Type | Required | Value | Notes |
|----|--------------|------|----------|-------|-------|
| 1 | `Event code` | String | Yes | Text, max 10 chars, uppercase | |
| 2 | `Entity` | String | Yes | `1000` / `1010` / `1020` / `1030` | Company code |
| 3 | `Event Name` | String | Yes | Free text (Thai / English) | |
| 4 | `Dr Account Number` | String (10-digit) | Yes | e.g. `1211100100` | Must exist in GL master |
| 5 | `Cr Account Number` | String (10-digit) | Yes | e.g. `2212000100` | Must exist in GL master |
| 6 | `Logic Dr.Profit center / cost center` | String | No | `Fixed value` / `Look up from Branch Code` / `Look up from Contract` | Leave empty if no mapping |
| 7 | `Dr. Profit center / cost center value` | String | Conditional | Fixed: any string / Look up: `Ref1`–`Ref6` | Required if column 6 is filled |
| 8 | `Dr. Sub ledger Value` | String | No | Any string | Used when GL has Customer/Vendor subledger |
| 9 | `Logic Cr.Profit center / cost center` | String | No | `Fixed value` / `Look up from Branch Code` / `Look up from Contract` | Leave empty if no mapping |
| 10 | `Cr. Profit center / cost center value` | String | Conditional | Fixed: any string / Look up: `Ref1`–`Ref6` | Required if column 9 is filled |
| 11 | `Cr. Sub ledger Value` | String | No | Any string | Used when GL has Customer/Vendor subledger |
| 12 | `ref1 remark` | String | Yes | Free text | Ref1 is always active and mandatory |
| 13 | `Reference2` | String | No | `Required` / `Optional` / *(empty)* | Empty = Ref2 not configured |
| 14 | `ref2 remark` | String | Conditional | Free text | Required if `Reference2` is `Required` or `Optional` |
| 15 | `Reference3` | String | No | `Required` / `Optional` / *(empty)* | Empty = Ref3 not configured |
| 16 | `ref3 remark` | String | Conditional | Free text | Required if `Reference3` is `Required` or `Optional` |
| 17 | `Reference4` | String | No | `Required` / `Optional` / *(empty)* | Empty = Ref4 not configured |
| 18 | `ref4 remark` | String | Conditional | Free text | Required if `Reference4` is `Required` or `Optional` |
| 19 | `Reference5` | String | No | `Required` / `Optional` / *(empty)* | Empty = Ref5 not configured |
| 20 | `ref5 remark` | String | Conditional | Free text | Required if `Reference5` is `Required` or `Optional` |
| 21 | `is_auto_post` | Boolean | Yes | `TRUE` / `FALSE` | |
| 22 | `require_internal_no` | Boolean | Yes | `TRUE` / `FALSE` | |

### Logic Value Reference

| Value in file | Meaning |
|---------------|---------|
| `Fixed value` | Use a hardcoded fixed value — fill the value column with any string |
| `Look up from Branch Code` | Derive from branch code lookup — fill the value column with `Ref1`–`Ref6` |
| `Look up from Contract` | Derive from contract number lookup — fill the value column with `Ref1`–`Ref6` |

### Example Upload Data

```
Event code	Entity	Event Name	Dr Account Number	Cr Account Number	Logic Dr.Profit center / cost center	Dr. Profit center / cost center value	Dr. Sub ledger Value	Logic Cr.Profit center / cost center	Cr. Profit center / cost center value	Cr. Sub ledger Value	ref1 remark	Reference2	ref2 remark	Reference3	ref3 remark	Reference4	ref4 remark	Reference5	ref5 remark	is_auto_post	require_internal_no
BWVO001	1000	รับเงินจากลูกค้า	1211100100	2212000100	Look up from Branch Code	Ref1		Look up from Contract	Ref2		รหัสสาขา	Required	Contract no.	Optional	ช่องทางจ่ายเงิน				TRUE	TRUE
BWVO002	1000	ส่งเงินต่อให้	1211100100	2212000100	Look up from Branch Code	Ref2		Look up from Contract	Ref1	1030	Contract no.							TRUE	TRUE
BWVO003	1000	Fixed value example	1211200100	2212000100	Fixed value	103000000		Fixed value	1030		รหัสอ้างอิง							FALSE	FALSE
BWVO004	1000	Credit look up only	1211200100	2212000100			Look up from Branch Code	Ref1		รหัสอ้างอิง							TRUE	FALSE
```

### Validation Rules

| Rule | Error Message |
|------|---------------|
| Any required column header is missing | `Missing column: <column name>` |
| `Entity` not in allowed list | `Entity "<value>" is invalid (must be 1000/1010/1020/1030)` |
| `Event code` is empty | `Event code is required` |
| `Event code` exceeds 10 characters | `Event code max 10 chars (got <n>)` |
| `Event Name` is empty | `Event Name is required` |
| `Dr Account Number` is empty | `Dr Account Number is required` |
| `Dr Account Number` not found in GL master | `Dr Account Number "<value>" not found in GL master` |
| `Cr Account Number` is empty | `Cr Account Number is required` |
| `Cr Account Number` not found in GL master | `Cr Account Number "<value>" not found in GL master` |
| Logic Dr/Cr is not one of the 3 allowed values | `Logic DR/CR "<value>" is invalid (use: Fixed value / Look up from Branch Code / Look up from Contract)` |
| Logic Dr/Cr = `Fixed value` but value column is empty | `DR/CR value is required when Logic is "Fixed value"` |
| Logic Dr/Cr = `Look up from Branch Code` or `Look up from Contract` but value column is empty | `DR/CR value (reference field) is required when Logic is "<value>"` |
| Logic Dr/Cr = `Look up from Branch Code` or `Look up from Contract` but value is not `Ref1`–`Ref6` | `DR/CR value "<value>" is invalid (use Ref1–Ref6)` |
| `ref1 remark` is empty | `ref1 remark is required` |
| `Reference2`–`Reference5` is not `Required` / `Optional` / empty | `Reference<n> "<value>" is invalid (use Required/Optional or leave empty)` |
| `Reference2`–`Reference5` = `Required` or `Optional` but remark is empty | `ref<n> remark is required when Reference<n> is "<value>"` |
| `is_auto_post` is empty | `is_auto_post is required (TRUE or FALSE)` |
| `is_auto_post` is not `TRUE` or `FALSE` | `is_auto_post "<value>" is invalid (use TRUE or FALSE)` |
| `require_internal_no` is empty | `require_internal_no is required (TRUE or FALSE)` |
| `require_internal_no` is not `TRUE` or `FALSE` | `require_internal_no "<value>" is invalid (use TRUE or FALSE)` |
| Duplicate `Event code` + `Entity` within the same file | `Duplicate event code "<value>" for entity <entity> in this file` |
| `Event code` + `Entity` already exists in the system | `Event code "<value>" already exists for entity <entity>` |

---

## Reference Data

### GL Accounts (10-digit codes)

| Code | Name | Type | Tracking Dimension | Sub-Ledger Type |
|------|------|------|--------------------|----------------|
| 1211100100 | Short-term Receivables | Asset | Profit Center | Customer |
| 1211100200 | Trade Receivables | Asset | Profit Center | Customer |
| 1211200100 | Cash and Cash Equivalents | Asset | Profit Center | No Subledger |
| 1212000100 | Inventory | Asset | Cost Center | No Subledger |
| 2211100100 | Accounts Payable | Liability | Profit Center | Vendor |
| 2212000100 | Long-term Loans | Liability | Profit Center | No Subledger |
| 2212100100 | Accrued Expenses | Liability | Cost Center | No Subledger |
| 4110000100 | Service Revenue | Revenue | Profit Center | No Subledger |
| 4111000100 | Interest Income | Revenue | Profit Center | No Subledger |
| 4120000100 | Fee Income | Revenue | Profit Center | No Subledger |
| 5110000100 | Operating Expenses | Expense | Cost Center | No Subledger |
| 5120000100 | Interest Expense | Expense | Cost Center | No Subledger |
| 5130000100 | Administrative Expenses | Expense | Cost Center | No Subledger |

### Reference Fields
Reference1 (id:1) through Reference5 (id:5) — shown as chips in modal

### Algorithm Options

| ID | Label | Available For |
|----|-------|--------------|
| 1 | Fixed Value | Profit Center, Cost Center, Customer, Vendor |
| 2 | Look Up Profit Center by Branch Code | Profit Center |
| 3 | Look Up Cost Center by Branch Code | Cost Center |
| 4 | Look Up Profit Center by Contract Number | Profit Center |
| 5 | Look Up Cost Center by Contract Number | Cost Center |

### Target Attribute Options

| ID | Label | GL Condition |
|----|-------|-------------|
| 1 | Profit Center | `trackingDimensionId = 1` |
| 2 | Cost Center | `trackingDimensionId = 2` |
| 3 | Customer | `subLedgerTypeId = 1` |
| 4 | Vendor | `subLedgerTypeId = 2` |

---

## Business Rules

1. Every event must belong to exactly one company code
2. Every event must have both a Debit GL and a Credit GL
3. Target attribute configuration rows are shown only when the selected GL is configured for that attribute (driven by Chart of Accounts data)
4. Algorithm mappings are optional (zero or more per target attribute row)
5. **Ref1 is always active and always mandatory** — cannot be deactivated
6. Ref2–Ref5 are optional toggles — user activates as needed
7. No delete — use Set Inactive via edit modal instead
8. New events are always created as Active
9. Status can only be changed via the Edit modal
10. Auto Post to SAP defaults to ON for new events
11. Bulk upload rows with duplicate `event_code` per `company_code` are rejected

---

## Future Enhancements

1. Audit trail (created by / modified by / timestamp)
2. Prevent duplicate event codes per company (currently not validated in prototype)
3. GL hierarchy browser
4. PDF export of event configuration
5. Bulk upload error report (row-by-row validation result file)
6. Ref6 support if needed by upstream systems
