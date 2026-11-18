# Feature 11: Maintain Reports

**Feature ID:** 11  
**Branch pattern:** `feature/11-maintain-reports`  
**Status:** Ready  
**Created:** 2026-09-19  
**Last Updated:** 2026-11-10  
**Input:** Track, review, and resolve warehouse exception reports and operational audit summaries.  
**Depends on:** [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md)  

---

## User Stories

### US-11.1: Generate / Record Exception Report
**As an** automated process or office manager  
**I want to** record an operational exception report when discrepancies occur (stock-outs, receiving shortages, damaged goods, transit exceptions)  
**So that** management has an auditable log of warehouse anomalies requiring investigation  

* **Priority:** P1  
* **Independent test:** Trigger an exception event (e.g., dock damage logged); verify an exception report is created with snapshot data and notified to supervisors.  
* **Acceptance scenarios:** see [US-11.1](#us-111--generate--record-exception-report) under Acceptance Criteria.

### US-11.2: View and Filter Reports
**As an** office manager  
**I want to** filter and review warehouse reports by facility, report type, date range, and resolution status  
**So that** I can prioritize unresolved discrepancies and monitor facility health  

* **Priority:** P1  
* **Independent test:** Apply filters by report type and date range; verify only matching report records are displayed.  
* **Acceptance scenarios:** see [US-11.2](#us-112--view-and-filter-reports) under Acceptance Criteria.

### US-11.3: Acknowledge and Resolve Exception Report
**As an** office manager  
**I want to** record investigation notes and mark an exception report as resolved  
**So that** the audit log documents what corrective action was taken and who resolved the incident  

* **Priority:** P1  
* **Independent test:** Update an open report with investigation notes and transition status to `"RESOLVED"`; verify audit fields persist.  
* **Acceptance scenarios:** see [US-11.3](#us-113--acknowledge-and-resolve-exception-report) under Acceptance Criteria.

### US-11.4: Archive Historical Reports
**As an** office manager  
**I want to** archive resolved reports older than the regulatory retention period  
**So that** active reporting views remain clean while preserving historical audit data  

* **Priority:** P2  
* **Independent test:** Archive a resolved report; verify it transitions to `"ARCHIVED"` status and is removed from the active operational dashboard.  
* **Acceptance scenarios:** see [US-11.4](#us-114--archive-historical-reports) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST support creating reports both via automated triggers (e.g., receiving damages, stockouts) and on-demand manual generation.
- **FR-002**: System MUST categorize reports into standardized enum types: `ITEM_OUT_OF_STOCK`, `RECEIVING_DISCREPANCY`, `DELIVERY_DISCREPANCY`, `INVENTORY_VALUATION`.
- **FR-003**: System MUST store structured snapshot details (JSON payload) containing associated orders, items, workers, and quantities.
- **FR-004**: System MUST notify active office managers immediately whenever a high-priority exception report is generated.
- **FR-005**: System MUST require resolution notes before transitioning any report from `OPEN` to `RESOLVED`.
- **FR-006**: System MUST prevent hard deletion of active or unresolved exception reports.

---

## Data Ownership & Isolation

Reports belong to the tenant company and origin warehouse facility.

| Rule | Requirement |
|---|---|
| **Read scope** | `GET /warehouse/reports` returns reports where `company_id = req.user.companyId` and `warehouse_id = req.user.warehouseId`. |
| **Write scope** | Resolving or archiving reports requires `company_id = req.user.companyId` and role `OFFICE_MANAGER` or `COMPANY_ADMIN`. |
| **Create scope** | System triggers and manual generation associate with caller's company and warehouse. |
| **Cross-user access** | Accessing reports belonging to another company returns `404 Not Found` (never `403`). |
| **UI scope** | Managers only see reports relevant to their active warehouse facility. |
| **Implementation** | Centralized query helper `getAccessibleReportOrNull(req, reportId)`. |

---

## API Requirements

| Method | Endpoint | Auth | Role | Purpose |
|---|---|---|---|---|
| `GET` | `/warehouse/reports` | Yes | `office manager`, `company admin` | List & filter reports |
| `GET` | `/warehouse/reports/:id` | Yes | `office manager`, `company admin` | Get report and JSON payload |
| `POST` | `/warehouse/reports` | Yes | `office manager`, `company admin` | Generate on-demand report |
| `POST` | `/warehouse/reports/:id/resolve` | Yes | `office manager`, `company admin` | Record notes & resolve |
| `POST` | `/warehouse/reports/:id/archive` | Yes | `office manager`, `company admin` | Archive resolved report |

**Resolve Report Request Body (`POST /warehouse/reports/42/resolve`):**
```json
{
  "resolution_notes": "Contacted supplier Acme Beverage; credit memo #CM-881 issued for 5 damaged cases."
}
```

**Success Response (`200 OK`):**
```json
{
  "id": 42,
  "report_code": "RPT-202611-0042",
  "company_id": 1,
  "warehouse_id": 2,
  "report_type": "RECEIVING_DISCREPANCY",
  "triggered_by": "AUTOMATED_SYSTEM",
  "summary": "Shortage of 5 cases recorded on PO-202611-0001",
  "status": "RESOLVED",
  "resolved_by_worker_id": 4,
  "resolution_notes": "Contacted supplier Acme Beverage; credit memo #CM-881 issued for 5 damaged cases.",
  "created_at": "2026-11-10T11:00:00.000Z",
  "resolved_at": "2026-11-10T12:30:00.000Z"
}
```

**Error Response (`400 Bad Request` / `404 Not Found`):**
```json
{
  "message": "Resolution notes are required before resolving an exception report"
}
```

---

## Screen Requirements

### [View: Operational Reports & Exceptions] — route name `reports`
- **Heading:** **Operational Reports & Exception Audits**
- **Filter Bar:**
  - Facility selector `<v-select>`
  - Report type filter chips: `All`, `Out of Stock`, `Receiving Discrepancy`, `Delivery Discrepancy`, `Inventory Valuation`
  - Status filter: `OPEN`, `RESOLVED`, `ARCHIVED`
  - Date range picker (start and end date)
- **Primary action:** **+ Run On-Demand Report** (`oc-cta`) opens generation dialog
- **Data Table:** `<v-data-table>` displaying columns:
  - Report Code, Type, Summary, Created At, Status (`<v-chip color="red">Open</v-chip>`)
  - Row Actions:
    - **View Details** icon (`aria-label="View report details"`)
    - **Resolve** icon (`aria-label="Resolve report"`, disabled if already RESOLVED)
    - **Archive** icon (`aria-label="Archive report"`)
- **Resolution Dialog:** `<v-dialog>` with textarea for `resolution_notes`, **Submit Resolution** (`oc-cta`), and **Cancel**
- **Empty state:** **"No exception reports match your criteria. Facility operating normally."**
- **Loading state:** Skeleton rows
- **Error state:** `<v-alert type="error">` displaying API error message

---

## Key Entities

- **Report**: An operational record or discrepancy alert defined by report number, warehouse, report type, creation trigger, summary, structured event payload, resolution status (`OPEN` | `UNDER_REVIEW` | `RESOLVED` | `ARCHIVED`), and resolving manager notes.

---

## Data Model Requirements

### `reports`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique internal report ID |
| `report_code` | VARCHAR(30) | NOT NULL, UNIQUE | Human-readable code (e.g., `RPT-202611-0042`) |
| `company_id` | INT / BIGINT | FK -> `companies(id)`, NOT NULL | Owning tenant company |
| `warehouse_id` | INT / BIGINT | FK -> `warehouses(id)`, NOT NULL | Origin facility |
| `report_type` | VARCHAR(40) | NOT NULL | Enum: `'ITEM_OUT_OF_STOCK'`, `'RECEIVING_DISCREPANCY'`, `'DELIVERY_DISCREPANCY'`, `'INVENTORY_VALUATION'` |
| `triggered_by` | VARCHAR(30) | NOT NULL | Enum: `'AUTOMATED_SYSTEM'`, `'OFFICE_MANAGER'` |
| `summary` | VARCHAR(255) | NOT NULL | Brief summary of discrepancy |
| `details_json` | TEXT / JSON | NOT NULL | Structured snapshot payload |
| `status` | VARCHAR(20) | NOT NULL, DEFAULT `'OPEN'` | Enum: `'OPEN'`, `'UNDER_REVIEW'`, `'RESOLVED'`, `'ARCHIVED'` |
| `resolved_by_worker_id`| INT / BIGINT | FK -> `workers(id)`, NULLABLE | Manager who investigated and resolved report |
| `resolution_notes` | TEXT | NULLABLE | Action taken to resolve issue |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp created |
| `resolved_at` | TIMESTAMP | NULLABLE | Timestamp resolved |

---

## Acceptance Criteria (Gherkin)

### US-11.1 — Generate / Record exception report

#### Scenario: Automated generation of receiving discrepancy report
* **Given** receiving clerk logs a shipment shortage of 10 cases on PO "PO-202611-0001"
* **When** the receiving transaction is submitted
* **Then** the system automatically creates a report with:
  * `report_type` = `"RECEIVING_DISCREPANCY"`
  * `warehouse_id` matching the destination facility
  * `status` = `"OPEN"`
  * `summary` = "Shortage of 10 cases recorded on PO-202611-0001 from Acme Beverage Supply"
* **And** triggers a high-priority dashboard notification to facility office managers

#### Scenario: On-demand generation of inventory valuation report
* **Given** an office manager is on the Reports dashboard
* **When** the manager selects "Generate Report", chooses type `"INVENTORY_VALUATION"` for warehouse "DAL-01", and submits
* **Then** the system aggregates current on-hand quantities multiplied by item unit costs
* **And** saves the report record with `triggered_by` = `"OFFICE_MANAGER"`
* **And** displays the completed report summary on screen

---

### US-11.2 — View and filter reports

#### Scenario: Filter reports by status and type
* **Given** multiple reports exist across various types and statuses
* **When** the office manager filters by `report_type` = `"RECEIVING_DISCREPANCY"` and `status` = `"OPEN"`
* **Then** the system returns only matching open receiving discrepancy reports
* **And** displays count and summary of filtered records

#### Scenario: Filter reports with inverted date range
* **Given** an office manager is on the reports filter panel
* **When** the manager enters a start date of "2026-11-15" and an end date of "2026-11-01"
* **Then** the system rejects the filter query
* **And** displays an inline error message "Start date cannot be after end date"
* **And** does not execute the report query

---

### US-11.3 — Acknowledge and resolve exception report

#### Scenario: Successfully resolve an open exception report
* **Given** an open report "RPT-202611-0042" regarding damaged cans
* **When** the office manager enters resolution notes: "Contacted supplier Acme Beverage; credit memo #CM-881 issued for 5 damaged cases."
* **And** clicks "Mark as Resolved"
* **Then** the system updates report `status` to `"RESOLVED"`
* **And** sets `resolved_by_worker_id` to the logged-in manager's ID
* **And** sets `resolved_at` to the current timestamp
* **And** displays confirmation "Report RPT-202611-0042 marked as Resolved"

#### Scenario: Prevent resolving report without resolution notes
* **Given** an open report "RPT-202611-0042"
* **When** the office manager attempts to click "Mark as Resolved" with empty resolution notes
* **Then** the system rejects the status transition
* **And** displays an inline validation error "Resolution notes are required before resolving an exception report"
* **And** the report remains in `"OPEN"` status

---

### US-11.4 — Archive historical reports

#### Scenario: Successfully archive a resolved report
* **Given** a report "RPT-202609-0010" has status `"RESOLVED"` and was resolved more than 90 days ago
* **When** the office manager selects "Archive Report" and confirms
* **Then** the system updates report status to `"ARCHIVED"`
* **And** removes the report from the active open reports view
* **And** displays confirmation "Report RPT-202609-0010 archived successfully"

#### Scenario: Prevent archiving an unresolved open report
* **Given** report "RPT-202611-0042" is in status `"OPEN"`
* **When** the office manager attempts to archive report "RPT-202611-0042"
* **Then** the system blocks the archive action
* **And** displays an error message "Cannot archive report in OPEN status. Reports must be marked RESOLVED before archiving"
* **And** the report remains in `"OPEN"` status

---

## Definition of Done

* [ ] Backend and frontend implemented per this spec (**FR-001** – **FR-006** satisfied)
* [ ] Database table `reports` created with report type enums and structured JSON storage
* [ ] API endpoints `/warehouse/reports` enforce facility scoping and validation rules
* [ ] Gherkin acceptance criteria verified by automated integration tests
* [ ] Resolution notes strictly required for status transition to `RESOLVED`

---

## Out of Scope

* Automated supplier claim filing over external vendor portals
* PDF export of complex multi-page financial ledger audits
* Scheduled email digest dispatching
