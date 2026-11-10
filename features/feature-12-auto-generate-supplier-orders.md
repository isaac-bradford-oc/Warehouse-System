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

