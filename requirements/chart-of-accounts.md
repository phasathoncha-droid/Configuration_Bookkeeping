# AMS Chart of Accounts — Requirements Document

**Version:** 2.1
**Date:** May 24, 2026
**Status:** Approved — HTML prototype is the solid reference
**Target Users:** Accounting Staff (Internal Bank)
**Prototype:** `prototypes/ams-chart-of-accounts.html`

---

## Executive Summary

The AMS Chart of Accounts module allows accounting staff to configure and manage the financial reporting structure of the organization. The module is accessed via the "Chart of Account Configuration" sub-menu under "Admin Configuration" in the left navigation. It contains two tabs: **Chart of Account** (default) and **FS Group**.

This document covers:
- **Phase 1 (complete):** FS Group configuration
- **Phase 2 (this document):** Chart of Account configuration — GL account setup linked to FS Groups

---

## System Overview

### Purpose
- Define Financial Statement Groups (FS Groups) that classify GL accounts into reporting categories
- Assign each FS Group a unique code, a display name, and a Financial Statement Type
- View, filter, search, and manage all FS Groups in a paginated table

### Key Users
- **Accounting Staff** — Create and configure FS Groups
- **Accounting Managers** — Review and activate/deactivate FS Groups

---

## Page Layout

### App Header (sticky, 56px)
- AMS logo (left) + "Accounting Management System" brand text
- User name + Log out button (right)
- Background: blue gradient `#003D99 → #0C447C`

### Page Header
- Title: "Chart of Account Configuration"
- Subtitle: "Manage FS Groups and Chart of Accounts for financial reporting"

### Tab Bar (below page header)
```
[ Chart of Account ]  [ FS Group ]
```
- Two tabs rendered as horizontal tab buttons below the page header
- **Chart of Account** tab: first, active by default — shows a "Coming Soon" placeholder content area
- **FS Group** tab: second, shows the FS Group table and toolbar
- Active tab: blue bottom border (`#378ADD`), bold text, primary color
- Inactive tab: gray text, no border, hover shows subtle blue underline

### Toolbar (inside FS Group tab)
```
[ 🔍 Search FS groups...     ]   [ Active ▼ ]  [ + Create New FS Group ]
```
- Search input (flex:1, max 420px) — filters by FS No., FS Name, FS Type in real time
- Status filter dropdown: **Active** (default) / Inactive / All Status
- Create New FS Group button (primary blue, right)

### Table (inside FS Group tab)
- Sticky header — table body scrolls independently (`max-height: calc(100vh - 280px)`)
- Columns: FS No. | FS Name | FS Type | Status | Actions
- All data text in black (`#1A1D23`) — no colored data text
- FS No.: monospace, black
- FS Name: regular text, black
- FS Type: text badge showing full type name (Asset / Liabilities / Equity / Profit and Loss)
- Status: Active = green pill, Inactive = red pill
- Inactive rows rendered at `opacity: 0.7`
- Actions column: **Edit** button only (no delete, no status toggle in table)
- Empty state message when no results: "No FS groups found."

---

## Create / Edit Modal

Single modal used for both create and edit. Title changes accordingly.

### Modal Structure
- Header: blue gradient, title ("Create New FS Group" / "Edit FS Group") + subtitle + close (✕) button
- Body: 1 layer card containing all fields
- Footer: Cancel + Save FS Group buttons
- Validation errors appear in `#modalAlertArea` inside modal body, never page-level

### Layer Card — FS Group Details

**Layout (Create mode):**
```
[ FS No.          ]  [ FS Type ▼         ]
[ FS Name                                ]
```

**Layout (Edit mode — Status toggle appears far right in card header):**
```
FS GROUP DETAILS          Status  [ toggle ]
[ FS No.          ]  [ FS Type ▼         ]
[ FS Name                                ]
```

**Fields:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| FS No. | Text (monospace) | Yes | Free text, auto-uppercase, max 10 chars |
| FS Type | Dropdown | Yes | See FS Type options below |
| FS Name | Text | Yes | Free text, supports Thai |
| Status toggle | Toggle switch | Edit only | Active/Inactive; hidden in create mode; new records always start Active |

### FS Type Options

| ID | Display Label |
|----|---------------|
| 1 | Asset |
| 2 | Liabilities |
| 3 | Equity |
| 4 | Profit and Loss |

---

## Form Validation

Validation errors shown as red banner inside modal body (`#modalAlertArea`).

| Rule | Error Message |
|------|---------------|
| FS No. is empty | "Please fill in all required fields." |
| FS Type is not selected | "Please fill in all required fields." |
| FS Name is empty | "Please fill in all required fields." |

All three fields are validated together — a single error message covers all missing required fields.

---

## Status Management

- **Create:** No status toggle shown. New FS Groups always saved as `isActive: true`.
- **Edit:** Status toggle (Active/Inactive) shown in layer card header, far right. Toggle state saved on Save.
- **Table filter:** Defaults to Active. User can switch to Inactive or All Status.
- **No delete** — FS Groups are deactivated via status toggle in edit modal only.

---

## Success Messages

Toast notification appears top-right after saving (auto-dismisses after 4 seconds):

- Create: `Create successfully`
- Edit: `Edit successfully`

---

## API Payload

Logged to browser console on every save with prefix `=== FS GROUP API PAYLOAD ===`.

**Create payload:**
```json
{
  "groupCode": "A101",
  "fsGroupTypeId": 1,
  "groupName": "เงินสดและรายการเทียบเท่าเงินสด"
}
```

**Edit payload (includes isActive):**
```json
{
  "groupCode": "L207",
  "fsGroupTypeId": 2,
  "groupName": "หนี้สินไม่หมุนเวียนอื่น",
  "isActive": true
}
```

**Key conventions:**
- `groupCode` is a free-text string (uppercase)
- `fsGroupTypeId` is a numeric integer (1=Asset, 2=Liabilities, 3=Equity, 4=Profit and Loss)
- `groupName` is a free-text string (supports Thai)
- `isActive` is a boolean, included in edit payload only

---

## FS Type Reference

| ID | Code Prefix (example) | Display Label |
|----|----------------------|---------------|
| 1 | A | Asset |
| 2 | L | Liabilities |
| 3 | E | Equity |
| 4 | PL | Profit and Loss |

### Sample Seed Data

| FS No. | FS Name | FS Type |
|--------|---------|---------|
| A101 | เงินสดและรายการเทียบเท่าเงินสด | Asset |
| A102 | เงินลงทุนชั่วคราว | Asset |
| A103 | ลูกหนี้การค้าและลูกหนี้อื่น | Asset |
| A104 | ลูกหนี้เงินให้สินเชื่อ - ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | Asset |
| A104A | ลูกหนี้เงินให้สินเชื่อ - ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี - ค่าเผื่อฯ | Asset |
| A105 | ลูกหนี้ตามสัญญาเช่าซื้อ - ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | Asset |
| A105A | ลูกหนี้ตามสัญญาเช่าซื้อ - ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี - ค่าเผื่อฯ | Asset |
| A106 | ลูกหนี้ขายฝาก - ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | Asset |
| A106A | ลูกหนี้ขายฝาก - ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี - ค่าเผื่อฯ | Asset |
| A107 | ทรัพย์สินรอการขาย - สุทธิ | Asset |
| A108 | เงินให้กู้ยืมระยะสั้น | Asset |
| A109 | เงินให้กู้ยืมระยะยาว-ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | Asset |
| A110 | สินทรัพย์หมุนเวียนอื่น | Asset |
| A111 | สินค้าคงเหลือ | Asset |
| A112 | ลูกหนี้ประกันผ่อนชำระ | Asset |
| A112A | ลูกหนี้ประกันผ่อนชำระ - ค่าเผื่อฯ | Asset |
| A201 | เงินฝากธนาคารที่มีภาระค้ำประกัน | Asset |
| A202 | เงินลงทุนระยะยาว - เงินฝากประจำ | Asset |
| A203 | เงินลงทุนในบริษัทย่อย | Asset |
| A204 | เงินลงทุนเผื่อขาย | Asset |
| A205 | เงินลงทุนระยะยาวอื่น | Asset |
| A206 | ลูกหนี้เงินให้สินเชื่อ - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | Asset |
| A206A | ลูกหนี้เงินให้สินเชื่อ - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี - ค่าเผื่อฯ | Asset |
| A207 | ลูกหนี้ตามสัญญาเช่าซื้อ - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | Asset |
| A207A | ลูกหนี้ตามสัญญาเช่าซื้อ - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี - ค่าเผื่อฯ | Asset |
| A208 | ลูกหนี้ขายฝาก - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | Asset |
| A208A | ลูกหนี้ขายฝาก - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี - ค่าเผื่อฯ | Asset |
| A209 | เงินให้กู้ยืมระยะยาว-สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | Asset |
| A210 | อาคารและอุปกรณ์ | Asset |
| A211 | อสังหาริมทรัพย์เพื่อการลงทุน | Asset |
| A212 | สินทรัพย์ไม่มีตัวตน | Asset |
| A212A | สินทรัพย์ไม่มีตัวตน - เพื่อขาย | Asset |
| A213 | สินทรัพย์ภาษีเงินได้รอการตัดบัญชี | Asset |
| A214 | สินทรัพย์ไม่หมุนเวียนอื่น | Asset |
| A215 | สิทธิการใช้สินทรัพย์ | Asset |
| A216 | สินทรัพย์ตราสารอนุพันธ์ | Asset |
| L101 | เงินกู้ยืมระยะสั้นจากสถาบันการเงิน | Liabilities |
| L102 | เงินกู้ยืมระยะสั้น | Liabilities |
| L103 | หนี้สินตราสารอนุพันธ์ | Liabilities |
| L104 | เจ้าหนี้การค้าและเจ้าหนี้อื่น | Liabilities |
| L105 | ส่วนของเงินกู้ยืมระยะยาวที่ถึงกำหนดชำระภายในหนึ่งปี | Liabilities |
| L106 | ส่วนของหุ้นกู้ระยะยาวที่ถึงกำหนดชำระภายในหนึ่งปี | Liabilities |
| L107 | ส่วนของหนี้สินตามสัญญาเช่าซื้อและเช่าการเงินที่ถึงกำหนดชำระภายในหนึ่งปี | Liabilities |
| L108 | ภาษีเงินได้ค้างจ่าย | Liabilities |
| L109 | หนี้สินหมุนเวียนอื่น | Liabilities |
| L109A | ประมาณการค่าชดเชยผลเสียหายจากการค้ำประกัน | Liabilities |
| L110 | ส่วนของหนี้สินตามสัญญาเช่าที่ถึงกำหนดชำระภายในหนึ่งปี | Liabilities |
| L201 | เงินกู้ยืมระยะยาว - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | Liabilities |
| L202 | หุ้นกู้ระยะยาว - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | Liabilities |
| L203 | หนี้สินตามสัญญาเช่าซื้อและเช่าการเงิน - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | Liabilities |
| L205 | ประมาณการหนี้สิน - สำรองผลประโยชน์ระยะยาวของพนักงาน | Liabilities |
| L207 | หนี้สินไม่หมุนเวียนอื่น | Liabilities |
| L208 | หนี้สินตามสัญญาเช่า - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | Liabilities |
| SE101 | ทุนจดทะเบียน ออกจำหน่ายและชำระเต็มมูลค่าแล้ว | Equity |
| SE102 | ส่วนเกินมูลค่าหุ้นสามัญ | Equity |
| SE103 | ใบสำคัญแสดงสิทธิที่จะซื้อหุ้น | Equity |
| SE104 | กำไรสะสมจัดสรรแล้ว - สำรองตามกฎหมาย | Equity |
| SE105 | กำไรสะสมยังไม่ได้จัดสรร | Equity |
| SE106 | องค์ประกอบอื่นของส่วนของผู้ถือหุ้น | Equity |
| SEXXX | None Group | Equity |
| PL101 | รายได้ดอกเบี้ยและค่าธรรมเนียมในการให้บริการสินเชื่อ (Interest and fee income) | Profit and Loss |
| PL101A | รายได้ค่าบริการด้านไอที | Profit and Loss |
| PL102 | รายได้ค่าธรรมเนียม (Guarantee) | Profit and Loss |
| PL102A | รายได้ปรับปรุงจาก STL to EIR (Guarantee) | Profit and Loss |
| PL103 | ค่าปรับชำระล่วงหน้า (Prepayment Fee) | Profit and Loss |
| PL104A | ค่าปรับล่าช้า (Late Penalty Fee) | Profit and Loss |
| PL104B | ค่าติดตามทวงถาม (Collection Fee) | Profit and Loss |
| PL105 | รายได้ค่านายหน้า (Commission income) | Profit and Loss |
| PL106 | รายได้ค่าธรรมเนียมและค่าบริการอื่น (Other fee & Service income) | Profit and Loss |
| PL107 | รายได้อื่น (Other Income) | Profit and Loss |
| PL108 | รายได้ค่าบริหารจัดการ (Management fee income) | Profit and Loss |
| PL109 | รายได้เงินปันผล (Dividend Income) | Profit and Loss |
| PL110 | รายได้จากการขาย (Sales) | Profit and Loss |
| PL111 | ต้นทุนขาย (Cost of Good Sold) | Profit and Loss |
| PL112 | รายได้ค่าอบรมสัมนา (Training income) | Profit and Loss |
| PL113 | ต้นทุนการให้บริการ (Cost of Service) | Profit and Loss |
| PL201 | ค่าใช้จ่ายเกี่ยวกับพนักงาน (Personnel expenses) | Profit and Loss |
| PL202 | ค่าภาษีอื่น ๆ (Tax expenses) | Profit and Loss |
| PL203 | หนี้สูญและหนี้สงสัยจะสูญ (Doubtful expenses) | Profit and Loss |
| PL203A | หนี้สูญรับคืน (Recovery from write off) | Profit and Loss |
| PL203B | หนี้สูญรับคืน (Recovery from write off) - Insurance | Profit and Loss |
| PL204 | ขาดทุนจากการปรับโครงสร้างหนี้ (Loss on debt restructuring) | Profit and Loss |
| PL205 | ค่าเสื่อมราคาและค่าตัดจำหน่าย (Depreciation and amortisation expenses) | Profit and Loss |
| PL206 | ค่าเดินทาง (Travelling and transportation expenses) | Profit and Loss |
| PL207 | ค่าเช่าและค่าบริการ (Rental and service expenses) | Profit and Loss |
| PL207A | ค่าบริการอื่น (Other service expenses) | Profit and Loss |
| PL208 | ค่าอรรถประโยชน์ (Utilities expenses) | Profit and Loss |
| PL209 | ค่าธรรมเนียมวิชาชีพ (Professional fee) | Profit and Loss |
| PL210A | ค่าโฆษณาและค่าใช้จ่ายการตลาด (Advertising Online) | Profit and Loss |
| PL210B | ค่าโฆษณาและค่าใช้จ่ายการตลาด (Advertising Offline) | Profit and Loss |
| PL210C | ค่าโฆษณาและค่าใช้จ่ายการตลาด (Marketing event) | Profit and Loss |
| PL211 | ค่าบำรุงรักษา (Maintenance expenses) | Profit and Loss |
| PL212 | ค่าประกัน (Insurance expenses) | Profit and Loss |
| PL213 | ขาดทุนจากการด้อยค่าทรัพย์สิน (Loss on impairment of assets) | Profit and Loss |
| PL214 | ขาดทุนจากการด้อยค่าทรัพย์สินรอจำหน่าย-รถยึด (Loss on impairment of properties foreclosed) | Profit and Loss |
| PL217 | ขาดทุนจากการขายทรัพย์สินรอจำหน่าย-รถยึด (Loss on disposals of assets foreclosed) | Profit and Loss |
| PL218 | ขาดทุนจากการยึดทรัพย์สินรอการขาย (Loss on Asset Reposess) | Profit and Loss |
| PL219 | ค่าใช้จ่ายอื่น (Other expenses) | Profit and Loss |
| PL219A | ค่าใช้จ่ายบริการคลาวด์และเทคโนโลยี (Cloud and technology service expenses) | Profit and Loss |
| PL219B | ค่าใช้จ่ายอุปกรณ์สำนักงานและวัสดุสิ้นเปลือง (Office equipment & Supply) | Profit and Loss |
| PL219C | ค่าธรรมเนียมศาล (Court fee) | Profit and Loss |
| PL219D | ค่าใช้จ่ายสำหรับผู้จัดการพื้นที่ (Expenses relating to AM) | Profit and Loss |
| PL219E | ขาดทุนจากการขายและตัดจำหน่ายทรัพย์สิน (Loss from disposal and write-off fixed assets) | Profit and Loss |
| PL220 | ขาดทุนจากค่าเผื่อการปรับมูลค่าสินค้าและสินค้าล้าสมัย | Profit and Loss |
| PL221 | Loss Claim-NTB | Profit and Loss |
| PL222 | ค่าใช้จ่ายค่าบริหารจัดการ (Management fee expenses) | Profit and Loss |
| PL223 | เงินปันผลจ่าย (Dividend expenses) | Profit and Loss |
| PL301 | ต้นทุนทางการเงิน (Financial cost) | Profit and Loss |
| PL401 | ค่าใช้จ่ายภาษีเงินได้ (Corporate income tax) | Profit and Loss |
| PL401A | ส่วนเปลี่ยนแปลงจากภาษีเงินได้รอตัดฯ (Change in deferred tax) | Profit and Loss |
| OCI101 | Gain(loss) on revaluation/FMV | Profit and Loss |
| OCI102 | Income tax relating to OCI | Profit and Loss |
| OCI201 | Actuarial Gain/Loss | Profit and Loss |

---

## Business Rules

1. FS No. must be unique across all FS Groups (regardless of FS Type)
2. All three fields (FS No., FS Type, FS Name) are required
3. FS No. is stored in uppercase
4. No delete — use Set Inactive via edit modal instead
5. New FS Groups are always created as Active
6. Status can only be changed via the Edit modal

---

## Future Enhancements (FS Group)

1. Batch import via CSV
2. Audit trail (created by / modified by / timestamp)
3. Prevent duplicate FS No. with inline validation feedback
4. PDF export of FS Group configuration

---
---

# Chart of Account Configuration — Phase 2

---

## Executive Summary

The Chart of Account tab allows accounting staff to create and manage GL accounts. Each GL account is assigned an account number, Thai and English names, a linked FS Group, a sub-ledger type, a tracking dimension, optional master position topics, and an optional Config JV Process setting.

---

## System Overview

### Purpose
- Define GL accounts (e.g., `1151100100 — ลูกหนี้สินเชื่อ`)
- Link each GL account to an FS Group for financial statement reporting
- Configure sub-ledger type, tracking dimension, and master position topics per account
- Optionally enable Config JV Process to record this GL to SAP FI line by line instead of grouping
- View, filter, search, and manage all GL accounts in a scrollable table

### Key Users
- **Accounting Staff** — Create and configure GL accounts
- **Accounting Managers** — Review and activate/deactivate GL accounts

---

## Page Layout

### Tab
- **Chart of Account** tab (first tab, default active)
- Shows toolbar + table when active

### Toolbar (inside Chart of Account tab)
```
[ 🔍 Search accounts...      ]   [ Active ▼ ]  [ + Create New ]
```
- Search input (flex:1, max 420px) — filters by account number, account name TH, account name EN, FS No., FS Name in real time
- Status filter dropdown: **Active** (default) / Inactive / All Status
- Create New button (primary blue, right)

### Table (inside Chart of Account tab)
- Sticky header — table body scrolls independently (`max-height: calc(100vh - 280px)`)
- Columns: Account Number | Account Name (TH) | Account Name (EN) | FS Group | Status | Actions
- All data text in black (`#1A1D23`) — no colored data text
- Account Number: monospace, black, 10-digit
- Account Name TH: regular text, black
- Account Name EN: regular text, black
- FS Group: combined `FS No. — FS Name` (e.g., `A103 — ลูกหนี้การค้าและลูกหนี้อื่น`), font-size 12px
- Status: Active = green pill, Inactive = gray pill
- Inactive rows rendered at `opacity: 0.7`
- Actions column: **Edit** button only (no delete)
- Empty state message when no results: "No accounts found."

---

## Create / Edit Modal

Single modal used for both create and edit. Title changes accordingly.

### Modal Structure
- Header: blue gradient, title ("Create New Account" / "Edit Account") + subtitle + close (✕) button
- Body: 1 layer card containing all fields (max-width 680px)
- Footer: Cancel + Save Account buttons
- Validation errors appear in `#coaModalAlertArea` inside modal body, never page-level

### Layer Card — Account Details

**Layout (Create mode):**
```
ACCOUNT DETAILS
[ Account Number          ]  [ FS Group (search)                    ]
[ Account Name (TH)                                                  ]
[ Account Name (EN)                                                  ]
[ Sub-Ledger Type ▼       ]  [ Tracking Dimension ▼                 ]
[ Master Position Topics (optional multi-select checkboxes)         ]
[ Config JV Process                               toggle Off / On   ]
[ (expanded when On) Reference on JV  ○ Event Code  ○ Reference 1  ]
```

**Layout (Edit mode — Status toggle appears far right in card header):**
```
ACCOUNT DETAILS                                   Status  [ toggle ]
[ Account Number          ]  [ FS Group (search)                    ]
[ Account Name (TH)                                                  ]
[ Account Name (EN)                                                  ]
[ Sub-Ledger Type ▼       ]  [ Tracking Dimension ▼                 ]
[ Master Position Topics (optional multi-select checkboxes)         ]
[ Config JV Process                               toggle Off / On   ]
[ (expanded when On) Reference on JV  ○ Event Code  ○ Reference 1  ]
```

### Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| Account Number | Text (monospace) | Yes | Numeric only, exactly 10 digits |
| FS Group | Smart search | Yes | Search by FS No. or FS Name; displays `FS No. — FS Name`; selected state shows chip with ✕ clear; only active FS Groups shown |
| Account Name (TH) | Text | Yes | Thai language, free text |
| Account Name (EN) | Text | Yes | English, free text |
| Sub-Ledger Type | Dropdown | Yes | See options below |
| Tracking Dimension | Dropdown | Yes | See options below |
| Master Position Topics | Checkbox group | No | Optional; zero or more may be selected; see options below |
| Config JV Process | Toggle (Off/On) | No | Default Off; when On, expands Reference on JV section |
| Reference on JV | Radio buttons | Conditional | Required only when Config JV Process is On; see options below |
| Status toggle | Toggle switch | Edit only | Active/Inactive; hidden in create mode; new records always start Active |

### Sub-Ledger Type Options

| ID | Display Label |
|----|---------------|
| 1 | Customer |
| 2 | Vendor |
| 3 | No Subledger |

### Tracking Dimension Options

| ID | Display Label |
|----|---------------|
| 1 | Profit Center |
| 2 | Cost Center |

### Master Position Topic Options

| ID | Display Label |
|----|---------------|
| 1 | Suspense |
| 2 | RPT |
| 3 | AR Balance |

### Reference on JV Options (visible only when Config JV Process is On)

| ID | Display Label |
|----|---------------|
| 1 | Event Code |
| 2 | Reference 1 |

### Config JV Process Behaviour
- Default state: Off (toggle shows gray track, label "Off")
- When turned On: toggle shows blue track, label "On"; Reference on JV section expands below the toggle bar, styled with blue border and light blue background, visually attached (border-top removed, bottom border-radius only)
- When turned Off: Reference on JV section collapses; any radio selection is cleared
- Purpose: when enabled, the GL is sent to SAP FI record-by-record instead of being grouped

---

## Form Validation

Validation errors shown as red banner inside `#coaModalAlertArea` at top of modal body.

| Rule | Error Message |
|------|---------------|
| Account Number empty or not exactly 10 digits | "Please fill in all required fields." |
| FS Group not selected | "Please fill in all required fields." |
| Account Name (TH) empty | "Please fill in all required fields." |
| Account Name (EN) empty | "Please fill in all required fields." |
| Sub-Ledger Type not selected | "Please fill in all required fields." |
| Tracking Dimension not selected | "Please fill in all required fields." |
| Config JV Process is On but no Reference on JV selected | "Please select a Reference Source for Config JV Process." |

All required fields except the Config JV check are validated together with a single message. Master Position Topics are not validated (optional).

---

## Status Management

- **Create:** No status toggle shown. New accounts always saved as `isActive: true`.
- **Edit:** Status toggle (Active/Inactive) shown in layer card header, far right. Toggle state saved on Save.
- **Table filter:** Defaults to Active. User can switch to Inactive or All Status.
- **No delete** — accounts are deactivated via status toggle in edit modal only.

---

## Success Messages

Toast notification appears top-right after saving (auto-dismisses after 4 seconds):

- Create: `Create successfully`
- Edit: `Edit successfully`

---

## API Payload

Logged to browser console on every save with prefix `=== CHART OF ACCOUNT API PAYLOAD ===`.

**Create payload (`isActive` omitted — always Active on create):**
```json
{
  "accountNumber": "1151100100",
  "accountNameTh": "ลูกหนี้สินเชื่อ",
  "accountNameEn": "Trade Accounts Receivable-AR Loan",
  "fsGroupId": 3,
  "subLedgerTypeId": 1,
  "trackingDimensionId": 1,
  "masterPositionTopicIds": [3],
  "isConfigJvProcess": true,
  "referenceSourceId": 1
}
```

**Create payload — Config JV Process Off (`referenceSourceId` is null):**
```json
{
  "accountNumber": "1101000100",
  "accountNameTh": "เงินสดในมือ",
  "accountNameEn": "Cash on Hand",
  "fsGroupId": 1,
  "subLedgerTypeId": 3,
  "trackingDimensionId": 1,
  "masterPositionTopicIds": [],
  "isConfigJvProcess": false,
  "referenceSourceId": null
}
```

**Edit payload (includes `isActive`):**
```json
{
  "accountNumber": "1151100100",
  "accountNameTh": "ลูกหนี้สินเชื่อ",
  "accountNameEn": "Trade Accounts Receivable-AR Loan",
  "fsGroupId": 3,
  "isActive": true,
  "subLedgerTypeId": 1,
  "trackingDimensionId": 1,
  "masterPositionTopicIds": [3],
  "isConfigJvProcess": true,
  "referenceSourceId": 1
}
```

**Key conventions:**
- `accountNumber` is a 10-digit numeric string
- `accountNameTh` and `accountNameEn` are free-text strings
- `fsGroupId` is a numeric integer (references FS Group record)
- `isActive` is a boolean, included in edit payload only
- `subLedgerTypeId` is a numeric integer (1=Customer, 2=Vendor, 3=No Subledger)
- `trackingDimensionId` is a numeric integer (1=Profit Center, 2=Cost Center)
- `masterPositionTopicIds` is an array of numeric integers; empty array `[]` when none selected
- `isConfigJvProcess` is a boolean (true/false)
- `referenceSourceId` is a numeric integer when Config JV is On (1=Event Code, 2=Reference 1), or `null` when Off

---

## Business Rules

1. Account Number must be exactly 10 numeric digits
2. Account Number, FS Group, Account Name (TH), Account Name (EN), Sub-Ledger Type, and Tracking Dimension are all required
3. Master Position Topics are optional — zero or more may be selected
4. Config JV Process is optional (default Off); if On, Reference on JV must be selected
5. Account Number should be unique across all GL accounts
6. No delete — use Set Inactive via edit modal instead
7. New accounts are always created as Active
8. Status can only be changed via the Edit modal
9. FS Group smart search shows only active FS Groups

---

## Future Enhancements

1. Batch import via CSV
2. Audit trail (created by / modified by / timestamp)
3. Prevent duplicate Account Number with inline validation
4. GL hierarchy browser (parent/child account structure)
5. PDF export of chart of accounts
6. Confirm actual API field names for `isConfigJvProcess` and `referenceSourceId` (placeholder names pending backend confirmation)
