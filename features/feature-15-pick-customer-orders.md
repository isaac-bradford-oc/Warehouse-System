# Feature 15: Pick Customer Orders

**Feature ID:** 15  
**Branch pattern:** `feature/15-pick-customer-orders`  
**Status:** Ready  
**Created:** 2026-11-10  
**Last Updated:** 2026-11-10  
**Input:** Released customer orders, warehouse bin locations, and mobile barcode scanning.  
**Depends on:** [Feature 3 — Maintain Items](feature-3-maintain-items.md), [Feature 4 — Maintain Inventory Locations](feature-4-maintain-inventory.md), [Feature 9 — Maintain Customer Orders](feature-9-maintain-customer-orders.md), [Feature 11 — Maintain Reports](feature-11-maintain-reports.md)  

---

## User Stories

### US-15.1: Generate Optimized Pick List
**As an** order picker  
**When** customer orders are released for fulfillment  
**I want** the system to generate an optimized pick path ordered by warehouse aisle and bin  
**So that** I can pick required items in the shortest physical travel time without backtracking  

* **Priority:** P1  
* **Independent test:** Release an order with items in aisles A01, A03, and A02; verify pick list sequences stops in logical traversal order (A01 -> A02 -> A03).  
* **Acceptance scenarios:** see [US-15.1](#us-151--generate-optimized-pick-list) under Acceptance Criteria.

### US-15.2: Scan and Pick Item from Bin
**As an** order picker  
**When** I arrive at the designated storage bin  
**I want to** scan the bin barcode and item barcode, and enter the quantity picked  
**So that** inventory is accurately deducted from the bin and recorded against the customer order  

* **Priority:** P1  
* **Independent test:** Scan matching bin and item barcodes for 10 units; verify bin `current_quantity` decrements by 10, line item `quantity_picked` increments by 10, and reserved inventory balance is cleared.  
* **Acceptance scenarios:** see [US-15.2](#us-152--scan-and-pick-item-from-bin) under Acceptance Criteria.

### US-15.3: Prevent Incorrect Item Pick via Barcode Validation
**As an** order picker  
**When** I inadvertently scan an incorrect item or grab a product from the wrong bin  
**I want** the system to reject the scan and alert me immediately  
**So that** the wrong product is not shipped to the customer  

* **Priority:** P1  
* **Independent test:** Scan an item barcode that does not match the pick task; verify system rejects the pick, produces an audible/visual error, and prevents quantity confirmation.  
* **Acceptance scenarios:** see [US-15.3](#us-153--prevent-incorrect-item-pick-via-barcode-validation) under Acceptance Criteria.

### US-15.4: Record Pick Shortage / Bin Discrepancy
**As an** order picker  
**When** a bin physically contains fewer items than indicated by the system  
**I want to** record a short-pick quantity and flag the bin for inventory recount  
**So that** order fulfillment proceeds with available stock while an inventory discrepancy report is raised  

* **Priority:** P1  
* **Independent test:** Record a pick shortage of 2 units; verify line item records partial pick, bin balance is corrected, and an exception report is generated in Feature 11.  
* **Acceptance scenarios:** see [US-15.4](#us-154--record-pick-shortage--bin-discrepancy) under Acceptance Criteria.

### US-15.5: Stage Picked Order for Packing and Shipping
**As an** order picker  
**When** all items for a customer order have been picked  
**I want to** deposit the picked cart into an assigned outbound staging lane and mark the pick task complete  
**So that** the order transitions to `"PICKED"` status and is ready for packing, Bill of Lading creation, and truck loading  

* **Priority:** P1  
* **Independent test:** Complete all pick tasks for an order and scan outbound staging lane barcode; verify order transitions to `"PICKED"` status and notifies shipping operations.  
* **Acceptance scenarios:** see [US-15.5](#us-155--stage-picked-order-for-packing-and-shipping) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST generate pick tasks automatically whenever an order transitions to `RELEASED_FOR_PICKING` in [Feature 9 — Maintain Customer Orders](feature-9-maintain-customer-orders.md).
- **FR-002**: System MUST sort pick tasks by aisle, rack, shelf, and bin to optimize picker travel time.
- **FR-003**: System MUST validate scanned bin barcode and item barcode before permitting quantity entry.
- **FR-004**: Upon successful pick, system MUST:
  * Decrement `bins.current_quantity`.
  * Increment `customer_order_items.quantity_picked`.
  * Decrement total reserved inventory.
- **FR-005**: System MUST prevent over-picking (entering a quantity greater than requested).
- **FR-006**: When a pick shortage is logged, system MUST:
  * Adjust `bins.current_quantity` to match physical count.
  * Generate an `ITEM_OUT_OF_STOCK` or `INVENTORY_DISCREPANCY` report in [Feature 11 — Maintain Reports](feature-11-maintain-reports.md).
  * Prompt manager review for reorder or order splitting.
- **FR-007**: When all line items for an order are fully picked, system MUST transition order status to `PICKED` and assign an outbound staging location.

---

## Key Entities

- **Pick Task**: Individual picking directive instructing an order picker to retrieve a specific SKU and quantity from a designated bin.
- **Outbound Staging**: Designated warehouse floor location where completed pick carts/pallets are parked for packing and loading.

---

## Data Model Requirements

### `pick_tasks`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique pick task ID |
| `order_item_id` | INT / BIGINT | FK -> `customer_order_items(id)`, NOT NULL | Parent customer order line |
| `bin_id` | INT / BIGINT | FK -> `bins(id)`, NOT NULL | Source picking bin location |
| `picker_worker_id`| INT / BIGINT | FK -> `workers(id)`, NULLABLE | Assigned picker |
| `quantity_requested`| INT | NOT NULL, CHECK (quantity_requested > 0) | Units required |
| `quantity_picked` | INT | NOT NULL, DEFAULT 0, CHECK (quantity_picked >= 0) | Units confirmed picked |
| `status` | VARCHAR(20) | NOT NULL, DEFAULT `'PENDING'` | Enum: `'PENDING'`, `'IN_PROGRESS'`, `'COMPLETED'`, `'SHORT_PICK'` |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Task creation timestamp |
| `completed_at` | TIMESTAMP | NULLABLE | Task completion timestamp |

---

## Acceptance Criteria (Gherkin)

### US-15.1 — Generate optimized pick list

#### Scenario: Generate sequenced pick list for released order
* **Given** customer order "SO-202611-0001" is in status `"RELEASED_FOR_PICKING"`
* **And** line item 1 is stored in bin "BIN-A01-S01-B01" (Aisle A01)
* **And** line item 2 is stored in bin "BIN-A02-S03-B02" (Aisle A02)
* **When** picker "Marcus Johnson" accepts the pick assignment
* **Then** the system displays the pick tasks sequenced starting at Aisle A01, then proceeding to Aisle A02

#### Scenario: Release order with items in unassigned bins
* **Given** customer order "SO-202611-0002" contains an item with no active bin location in warehouse "DAL-01"
* **When** the system generates pick tasks
* **Then** the system flags the task with status "UNSLOTTED_EXCEPTION"
* **And** notifies the office manager to assign a storage bin before picking can proceed

---

### US-15.2 — Scan and pick item from bin

#### Scenario: Successful item pick confirmation
* **Given** a pick task requires 12 cans of "DRK-COLA-01" (UPC "012345678905") from bin "BIN-A01-S01-B01"
* **When** the picker scans bin barcode "BIN-DAL01-A01-S01-B01"
* **And** scans item barcode "012345678905"
* **And** enters quantity picked = 12
* **Then** the system decrements bin `current_quantity` by 12
* **And** sets line item `quantity_picked` to 12
* **And** marks the pick task as completed

#### Scenario: Picker attempts to over-pick item quantity
* **Given** a pick task requires 12 cans of "DRK-COLA-01"
* **When** the picker scans valid barcodes but enters quantity picked = 15
* **Then** the system rejects the entry
* **And** displays an error message "Cannot pick 15 units: exceeds requested quantity of 12"
* **And** does not update inventory balances

---

### US-15.3 — Prevent incorrect item pick via barcode validation

#### Scenario: Picker scans matching item barcode successfully
* **Given** a pick task expects "DRK-COLA-01" (UPC "012345678905")
* **When** the picker scans barcode "012345678905"
* **Then** the system accepts the barcode with visual green confirmation
* **And** unlocks the quantity entry field

#### Scenario: Picker scans wrong item barcode
* **Given** a pick task expects "DRK-COLA-01" (UPC "012345678905")
* **When** the picker scans barcode "098765432109" (Item "DRK-DIET-02")
* **Then** the system rejects the scan
* **And** emits an alert tone
* **And** displays an error message "Barcode mismatch! Scanned: DRK-DIET-02, Expected: DRK-COLA-01. Please return item to shelf."
* **And** does NOT decrement any inventory

---

### US-15.4 — Record pick shortage / bin discrepancy

#### Scenario: Picker encounters inventory shortage in bin
* **Given** a pick task expects 20 units of "DRK-COLA-01" from bin "BIN-A01-S01-B01"
* **And** the bin physically contains only 14 units
* **When** the picker scans the bin and item
* **And** records picked quantity = 14 and logs reason "SHORT_PICK: Bin empty, missing 6 units"
* **Then** the system records `quantity_picked` = 14 on the order line
* **And** adjusts bin `current_quantity` to 0
* **And** automatically generates an exception report in Feature 11 with type `"ITEM_OUT_OF_STOCK"`
* **And** alerts the office manager of the 6-unit shortage for SO-202611-0001

#### Scenario: Picker finds zero units in designated bin
* **Given** a pick task expects 10 units of "DRK-COLA-01" from bin "BIN-A01-S01-B01"
* **And** the bin is completely empty
* **When** the picker logs zero units found with reason "BIN_COMPLETELY_EMPTY"
* **Then** the system sets bin `current_quantity` to 0
* **And** generates an immediate `"ITEM_OUT_OF_STOCK"` exception report
* **And** prompts the picker to proceed to the next item

---

### US-15.5 — Stage picked order for packing and shipping

#### Scenario: Complete picking for all order lines and stage outbound
* **Given** all line items for order "SO-202611-0001" have been picked
* **When** the picker delivers the cart to outbound staging lane "STAGE-LANE-03"
* **And** scans staging lane barcode "STG-LN-03"
* **Then** the system updates order status to `"PICKED"`
* **And** records staging location as "STAGE-LANE-03"
* **And** makes the order available for Bill of Lading generation and route dispatch
* **And** displays confirmation "Order SO-202611-0001 fully picked and staged in STAGE-LANE-03"

#### Scenario: Attempt to stage order before picking is complete
* **Given** order "SO-202611-0001" has 1 completed pick line and 1 remaining pending pick line
* **When** the picker attempts to scan outbound staging lane "STG-LN-03"
* **Then** the system blocks staging completion
* **And** displays an error message "Cannot stage order: 1 line item is still pending pick"
* **And** the order status remains `"RELEASED_FOR_PICKING"`

