# Feature 12: Auto-Generate Supplier Orders

**Feature ID:** 12  
**Branch pattern:** `feature/12-auto-generate-supplier-orders`  
**Status:** Ready  
**Created:** 2026-11-10  
**Last Updated:** 2026-11-10  
**Input:** Inventory stock levels, item min/max thresholds, and supplier purchasing rules.  
**Depends on:** [Feature 3 — Maintain Items](feature-3-maintain-items.md), [Feature 4 — Maintain Inventory Locations](feature-4-maintain-inventory.md), [Feature 6 — Maintain Suppliers](feature-6-maintain-suppliers.md), [Feature 7 — Maintain Supplier Orders](feature-7-maintain-supplier-orders.md)  

---

## User Stories

### US-12.1: Automated Min-Max Replenishment Calculation
**As an** automated process  
**I want to** evaluate inventory balances against item minimum thresholds on a scheduled basis  
**So that** replenishment needs are detected immediately before a warehouse runs out of stock  

* **Priority:** P1  
* **Independent test:** Reduce on-hand inventory below minimum threshold; trigger replenishment job and verify system identifies SKU and calculates required case quantities to reach maximum.  
* **Acceptance scenarios:** see [US-12.1](#us-121--automated-min-max-replenishment-calculation) under Acceptance Criteria.

### US-12.2: Auto-Generate Purchase Order Drafts
**As an** office manager  
**I want** the system to automatically group low-stock items by supplier and generate draft purchase orders  
**So that** I can review and approve replenishment orders without manual inventory counting and calculation  

* **Priority:** P1  
* **Independent test:** Run replenishment generator for items across 2 suppliers; verify 2 separate draft purchase orders are created with correct case pack quantities and vendor minimums.  
* **Acceptance scenarios:** see [US-12.2](#us-122--auto-generate-purchase-order-drafts) under Acceptance Criteria.

### US-12.3: Suppress Duplicate In-Flight Reorders
**As an** office manager  
**I want** the replenishment algorithm to account for open purchase orders already in-flight (`on_order`)  
**So that** the system does not create duplicate purchase orders for items that have already been ordered but not yet delivered  

* **Priority:** P1  
* **Independent test:** Trigger replenishment for an item below minimum that already has an open purchase order bringing it above minimum; verify no new reorder is created.  
* **Acceptance scenarios:** see [US-12.3](#us-123--suppress-duplicate-in-flight-reorders) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST compute net available stock using the formula:  
  `Effective Stock = (On-Hand Units + In-Flight On-Order Units) - Reserved Customer Units`.
- **FR-002**: Replenishment MUST trigger when `Effective Stock < item.min_quantity`.
- **FR-003**: System MUST calculate reorder units to reach `item.max_quantity`:  
  `Units Needed = item.max_quantity - Effective Stock`.
- **FR-004**: System MUST convert units needed to integer master cases, rounding up to the nearest whole case:  
  `Cases to Order = CEIL(Units Needed / item.case_quantity)`.
- **FR-005**: System MUST group all triggered items by `supplier_id` and originating `warehouse_id`.
- **FR-006**: If the generated order total is below the supplier's `min_order_amount`, system MUST create the order in `"DRAFT"` status and flag it with a warning notification for manager review.
- **FR-007**: System MUST log replenishment audit runs and notify office managers of newly drafted orders.

---

## Data Ownership & Isolation

Automated replenishment calculations and generated orders are strictly scoped to the tenant company and target warehouse.

| Rule | Requirement |
|---|---|
| **Read scope** | `GET /warehouse/replenishment/logs` returns runs and evaluations matching `company_id = req.user.companyId` and `warehouse_id = req.user.warehouseId`. |
| **Execution scope** | On-demand or scheduled replenishment execution evaluates only items and inventory belonging to `req.user.companyId` and the selected warehouse. |
| **Generated orders scope** | All auto-generated purchase orders are created with `company_id = req.user.companyId`, `warehouse_id = target_warehouse_id`, and status `DRAFT`. |
| **Cross-user access** | Accessing replenishment runs or attempting calculation for another company or warehouse returns `404 Not Found`. |
| **UI scope** | Office managers only see replenishment simulations, candidate items, and historical logs for their assigned facility. |
| **Implementation** | Centralized query helper `getAccessibleWarehouseOrNull(req, warehouseId)`. |

---

## API Requirements

| Method | Endpoint | Auth | Role | Purpose |
|---|---|---|---|---|
| `GET` | `/warehouse/replenishment/logs` | Yes | `office manager`, `company admin` | List historical replenishment execution logs |
| `GET` | `/warehouse/replenishment/logs/:id` | Yes | `office manager`, `company admin` | Get specific replenishment run details and generated POs |
| `POST` | `/warehouse/replenishment/calculate` | Yes | `office manager`, `company admin` | Preview replenishment requirements without creating orders (dry run) |
| `POST` | `/warehouse/replenishment/generate` | Yes | `office manager`, `company admin` | Run calculation and generate draft purchase orders |

**Generate Request Body (`POST /warehouse/replenishment/generate`):**
```json
{
  "warehouse_id": 2
}
```

**Success Response (`200 OK` / `201 Created`):**
```json
{
  "log_id": 42,
  "warehouse_id": 2,
  "items_evaluated_count": 184,
  "items_triggered_count": 3,
  "orders_created_count": 2,
  "generated_order_ids": [105, 106],
  "warnings": [
    "PO-202611-0006 for Nut Supply Co ($120.00) is below vendor minimum ($300.00)"
  ],
  "executed_at": "2026-11-10T06:00:00.000Z"
}
```

**Error Response (`400 Bad Request` / `404 Not Found`):**
```json
{
  "message": "Invalid warehouse ID or warehouse not found"
}
```

---

## Screen Requirements

### [View: Automated Replenishment Planning] — route name `replenishment`
- **Heading:** **Automated Replenishment Planning**
- **Warehouse Filter:** `<v-select>` facility selector
- **Summary KPI Cards:**
  - Items Below Minimum
  - Triggered Reorders
  - Pending Generated Draft POs
  - Last Automated Run Timestamp
- **Action Toolbar:**
  - **Simulate Replenishment** button (runs preview calculation without writing PO records)
  - **Generate Replenishment Orders** button (`oc-cta`) opens confirmation modal summarizing items, total spend, and draft PO count
- **Replenishment Candidate Table:** `<v-data-table>` displaying columns:
  - SKU, Item Name, Category, On-Hand, On-Order, Reserved, Effective Stock, Min Threshold, Max Threshold, Case Size, Calculated Cases, Supplier Name, Estimated Line Total, Status Badge
- **Recent Execution Runs Table:** `<v-data-table>` displaying:
  - Run Date/Time, Items Evaluated, Orders Generated, Status, Details Action
- **Empty state:** **"All inventory levels are currently above reorder thresholds for this warehouse."**
- **Loading state:** Skeleton table loaders
- **Error state:** `<v-alert type="error">` displaying calculation or permission error

---

## Key Entities

- **Replenishment Run**: Execution log recording timestamp, warehouse evaluated, items assessed, orders generated, and summary metrics.

---

## Data Model Requirements

### `replenishment_logs`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique execution log identifier |
| `warehouse_id` | INT / BIGINT | FK -> `warehouses(id)`, NOT NULL | Facility evaluated |
| `executed_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp execution completed |
| `items_evaluated_count`| INT | NOT NULL, DEFAULT 0 | Total SKUs evaluated against min threshold |
| `items_triggered_count`| INT | NOT NULL, DEFAULT 0 | Number of items meeting reorder trigger |
| `orders_created_count` | INT | NOT NULL, DEFAULT 0 | Count of draft POs generated |
| `execution_summary` | TEXT / JSON | NOT NULL | Summary details of generated POs and alerts |


---

## Acceptance Criteria (Gherkin)

### US-12.1 — Automated Min-Max replenishment calculation

#### Scenario: Item drops below minimum threshold and calculates exact case quantity
* **Given** item "DRK-COLA-01" in warehouse "DAL-01" has:
  | Attribute | Value |
  | `min_quantity` | 100 units |
  | `max_quantity` | 500 units |
  | `case_quantity` | 24 units per case |
  | `on_hand` | 80 units |
  | `on_order` | 0 units |
  | `reserved` | 0 units |
* **When** the scheduled replenishment process executes
* **Then** the system calculates `Effective Stock` = 80 units (below min 100)
* **And** calculates `Units Needed` = 500 - 80 = 420 units
* **And** rounds up to cases: `420 / 24` = 17.5 -> 18 cases (432 units)
* **And** identifies item for purchase order generation

#### Scenario: Item inventory already above minimum threshold is skipped
* **Given** item "DRK-COLA-01" has `min_quantity` = 100 and `on_hand` = 250 units
* **When** the replenishment process executes
* **Then** the system determines `Effective Stock` (250) is greater than `min_quantity` (100)
* **And** excludes the item from purchase order generation

---

### US-12.2 — Auto-generate purchase order drafts

#### Scenario: Successfully create draft purchase order grouped by supplier
* **Given** items "DRK-COLA-01" (needs 18 cases at $12.50 = $225.00) and "DRK-LEMON-02" (needs 10 cases at $14.00 = $140.00) belong to supplier "Acme Beverage Supply"
* **When** the auto-replenishment process generates orders
* **Then** the system creates a single purchase order for "Acme Beverage Supply" containing both line items
* **And** sets total order value to $365.00 (meeting supplier min $250.00)
* **And** sets order status to `"DRAFT"`
* **And** sends an in-app alert to warehouse office managers: "New replenishment PO draft PO-202611-0005 generated for Acme Beverage Supply ($365.00)"

#### Scenario: Auto-generated order below supplier minimum flagged with alert
* **Given** only item "SNK-ALMOND-01" triggers replenishment for supplier "Nut Supply Co" (min order $300.00)
* **And** the calculated order value is $120.00
* **When** the auto-replenishment process runs
* **Then** the system creates the order draft in `"DRAFT"` status
* **And** attaches a flag "BELOW_VENDOR_MINIMUM"
* **And** notifies the office manager: "PO-202611-0006 ($120.00) is below vendor minimum ($300.00). Manual review required."

---

### US-12.3 — Suppress duplicate in-flight reorders

#### Scenario: Item below minimum ignored when pending purchase order already covers deficit
* **Given** item "DRK-COLA-01" has `min_quantity` = 100, `max_quantity` = 500
* **And** `on_hand` = 60 units
* **And** an existing purchase order "PO-202611-0001" is in status `"SUBMITTED"` with 18 cases (432 units) `on_order`
* **When** the scheduled replenishment job runs
* **Then** the system calculates `Effective Stock` = 60 + 432 = 492 units
* **And** determines `Effective Stock` (492) is greater than `min_quantity` (100)
* **And** does NOT create any duplicate purchase order

#### Scenario: In-flight order cancelled and subsequent replenishment run re-triggers item
* **Given** purchase order "PO-202611-0001" is cancelled before arrival
* **And** item "DRK-COLA-01" has `on_hand` = 60 units and `on_order` = 0 units
* **When** the next replenishment job runs
* **Then** the system calculates `Effective Stock` = 60 units
* **And** recognizes the deficit below `min_quantity` (100)
* **And** generates a new replenishment draft PO for 18 cases

---

## Definition of Done

* [ ] Replenishment calculation engine and planning UI implemented per this spec (**FR-001** – **FR-007** satisfied)
* [ ] Database table `replenishment_logs` created with constraints and foreign keys
* [ ] Calculation correctly implements formula: `(On-Hand + On-Order - Reserved)` vs `min_quantity`
* [ ] Cases rounded up to nearest whole master case: `CEIL((max - effective) / case_quantity)`
* [ ] Orders grouped by supplier with `BELOW_VENDOR_MINIMUM` warnings attached when applicable
* [ ] Cross-tenant access returns `404 Not Found`
* [ ] Gherkin acceptance criteria verified by automated integration tests

---

## Out of Scope

* Direct supplier EDI auto-transmission without manager review (all auto-generated orders start in `DRAFT` status per [Feature 7 — Maintain Supplier Orders](feature-7-maintain-supplier-orders.md))
* Dock truck check-in and receiving ([Feature 13 — Receive Supplier Shipments](feature-13-receive-supplier-shipments.md))
* Predictive demand forecasting based on historical sales trends or seasonal adjustments

