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
[ Ref1 ] [ Ref2 ] [ Ref3 ] [ Ref4 ] [ Ref5 ]    (chip toggles)
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

- Six chips displayed: **Ref1 Ref2 Ref3 Ref4 Ref5** (Ref6 is always implicitly active but not shown as a chip)
- **Ref1** is always active (blue, non-deselectable) and always mandatory
- **Ref2–Ref5** are toggleable by user — click to activate/deactivate
- Each active reference chip shows a config row below with:
  - Remark (text input)
  - Mandatory / Optional dropdown — default **Mandatory**
- Ref1's mandatory dropdown is disabled (locked to Mandatory)
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

After saving, a green banner appears on the page (auto-dismisses after 6 seconds):

- Create: `Event "[EventName]" created successfully!` + GL codes
- Edit: `Event "[EventName]" updated successfully!` + GL codes

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
  "autoPostToSAP": true,
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
Users can bulk-create events by uploading a tab-separated file (TSV). This is designed for new company onboarding scenarios where many events need to be created at once.

### File Format
Tab-separated values (TSV). First row is the header. Compatible with Google Sheets copy-paste.

### Column Definitions

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| `company_code` | String | Yes | One of: 1000, 1010, 1020, 1030 |
| `event_code` | String | Yes | Max 10 chars, uppercase |
| `event_name` | String | Yes | Event description |
| `require_internal_number` | Boolean | Yes | TRUE / FALSE |
| `auto_post_to_sap` | Boolean | Yes | TRUE / FALSE |
| `gl_debit_code` | String (10-digit) | Yes | GL account code for debit side |
| `gl_credit_code` | String (10-digit) | Yes | GL account code for credit side |
| `ref1_active` | Boolean | Yes | Always TRUE (Ref1 is mandatory) |
| `ref1_mandatory` | Boolean | Yes | Always TRUE (Ref1 is always mandatory) |
| `ref1_remark` | String | Yes | Description/label for Ref1 field |
| `ref2_active` | Boolean | Yes | TRUE to activate Ref2 |
| `ref2_mandatory` | Boolean | Yes | TRUE = Mandatory, FALSE = Optional |
| `ref2_remark` | String | No | Description/label for Ref2 field |
| `ref3_active` | Boolean | Yes | TRUE to activate Ref3 |
| `ref3_mandatory` | Boolean | Yes | TRUE = Mandatory, FALSE = Optional |
| `ref3_remark` | String | No | Description/label for Ref3 field |
| `ref4_active` | Boolean | Yes | TRUE to activate Ref4 |
| `ref4_mandatory` | Boolean | Yes | TRUE = Mandatory, FALSE = Optional |
| `ref4_remark` | String | No | Description/label for Ref4 field |
| `ref5_active` | Boolean | Yes | TRUE to activate Ref5 |
| `ref5_mandatory` | Boolean | Yes | TRUE = Mandatory, FALSE = Optional |
| `ref5_remark` | String | No | Description/label for Ref5 field |
| `profit_center_algorithm` | String | No | See Algorithm Values table below |
| `profit_center_lookup_field` | String | No | `Ref1`–`Ref5` (only when algorithm is Look Up) |
| `profit_center_fixed_value` | String | No | Fixed value string (only when algorithm = Fixed Value) |
| `cost_center_algorithm` | String | No | See Algorithm Values table below |
| `cost_center_lookup_field` | String | No | `Ref1`–`Ref5` (only when algorithm is Look Up) |
| `cost_center_fixed_value` | String | No | Fixed value string (only when algorithm = Fixed Value) |
| `customer_algorithm` | String | No | `Fixed Value` only |
| `customer_lookup_field` | String | No | Not applicable for Customer |
| `customer_fixed_value` | String | No | Fixed value string |
| `vendor_algorithm` | String | No | `Fixed Value` only |
| `vendor_lookup_field` | String | No | Not applicable for Vendor |
| `vendor_fixed_value` | String | No | Fixed value string |

### Algorithm Values (upload file strings → system algorithm IDs)

| File Value | Algorithm ID | Description |
|------------|-------------|-------------|
| `Fixed Value` | 1 | Use a hardcoded fixed value |
| `Lookup PC Branch` | 2 | Look Up Profit Center by Branch Code |
| `Lookup CC Branch` | 3 | Look Up Cost Center by Branch Code |
| `Lookup PC Contract` | 4 | Look Up Profit Center by Contract Number |
| `Lookup CC Contract` | 5 | Look Up Cost Center by Contract Number |

### Example Upload Data (5 rows)

```
company_code	event_code	event_name	require_internal_number	auto_post_to_sap	gl_debit_code	gl_credit_code	ref1_active	ref1_mandatory	ref1_remark	ref2_active	ref2_mandatory	ref2_remark	ref3_active	ref3_mandatory	ref3_remark	ref4_active	ref4_mandatory	ref4_remark	ref5_active	ref5_mandatory	ref5_remark	profit_center_algorithm	profit_center_lookup_field	profit_center_fixed_value	cost_center_algorithm	cost_center_lookup_field	cost_center_fixed_value	customer_algorithm	customer_lookup_field	customer_fixed_value	vendor_algorithm	vendor_lookup_field	vendor_fixed_value
1000	BPAY001	Cash Payment from Customer	TRUE	TRUE	1211100100	2212000100	TRUE	TRUE	Transaction ID from lending system	TRUE	TRUE	Loan contract ref	FALSE	FALSE		FALSE	FALSE		FALSE	FALSE		Lookup PC Branch	Ref2					Fixed Value		1020			
1000	INT001	Interest Income from Deposit	FALSE	TRUE	1211200100	4111000100	TRUE	TRUE	Deposit account ref	FALSE	FALSE		FALSE	FALSE		FALSE	FALSE		FALSE	FALSE		Lookup PC Branch	Ref1								Fixed Value		1000
1000	PAY001	Payroll Payment	TRUE	FALSE	5110000100	1211200100	TRUE	TRUE	Payroll batch ref	TRUE	FALSE	HR payroll batch	FALSE	FALSE		FALSE	FALSE		FALSE	FALSE					Lookup CC Branch	Ref1					Fixed Value		201100
1000	TRF001	Internal Account Transfer	TRUE	TRUE	1211200100	1211200100	TRUE	TRUE	Transfer voucher ref	TRUE	TRUE	Source account ref	TRUE	TRUE	Destination account ref	FALSE	FALSE		FALSE	FALSE		Lookup PC Contract	Ref2					Fixed Value		1000			
1000	FEE002	Bank Charge Payment	FALSE	FALSE	5130000100	1211200100	TRUE	TRUE	Bank charge ref	FALSE	FALSE		FALSE	FALSE		FALSE	FALSE		FALSE	FALSE		Fixed Value		1003000000	Fixed Value		1002000				Fixed Value		3100
```

### Upload Validation Rules
- `company_code` must be one of the allowed values
- `event_code` must be unique per `company_code`
- Both `gl_debit_code` and `gl_credit_code` must exist in the GL master
- `ref1_active` and `ref1_mandatory` must always be TRUE
- If `profit_center_algorithm` is a Look Up type, `profit_center_lookup_field` is required
- If `profit_center_algorithm` = `Fixed Value`, `profit_center_fixed_value` is required
- Same rules apply for `cost_center_*`, `customer_*`, `vendor_*`

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
