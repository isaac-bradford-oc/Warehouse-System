# Feature 14: Putaway and Stock Inventory

**Feature ID:** 14  
**Branch pattern:** `feature/14-putaway-and-stock-inventory`  
**Status:** Ready  
**Created:** 2026-11-10  
**Last Updated:** 2026-11-10  
**Input:** Dock-staged received items, warehouse slot/bin mapping, and physical stocking scans.  
**Depends on:** [Feature 3 — Maintain Items](feature-3-maintain-items.md), [Feature 4 — Maintain Inventory Locations](feature-4-maintain-inventory.md), [Feature 13 — Receive Supplier Shipments](feature-13-receive-supplier-shipments.md)  

---

## User Stories

### US-14.1: Directed Putaway Task Suggestion
**As a** warehouse stocker  
**When** received shipments are staged at the dock  
**I want** the system to suggest optimal slot and bin locations with available capacity  
**So that** I know exactly where to transport and store incoming products without searching the warehouse  

* **Priority:** P1  
* **Independent test:** View open putaway tasks for staged items; verify system recommends assigned primary bin or empty compatible bin with sufficient capacity.  
* **Acceptance scenarios:** see [US-14.1](#us-141--directed-putaway-task-suggestion) under Acceptance Criteria.

### US-14.2: Scan and Confirm Putaway
**As a** warehouse stocker  
**When** I place goods into a warehouse storage bin  
**I want to** scan the bin barcode and item UPC barcode to confirm stocking  
**So that** bin inventory balances and facility on-hand stock counts are immediately updated in real time  

* **Priority:** P1  
* **Independent test:** Scan matching bin barcode and case UPC, confirm 10 cases stocked; verify bin `current_quantity` increases and staging balance decreases.  
* **Acceptance scenarios:** see [US-14.2](#us-142--scan-and-confirm-putaway) under Acceptance Criteria.

### US-14.3: Prevent Misplacement via Barcode Verification
**As a** warehouse stocker  
**When** I accidentally scan the wrong bin barcode or attempt to place items into an incompatible/full bin  
**I want** the system to reject the scan and alert me immediately  
**So that** inventory is not misallocated or physically lost in incorrect locations  

* **Priority:** P1  
* **Independent test:** Scan a bin assigned to another SKU or an invalid location; verify system rejects scan, displays mismatch error, and prevents quantity update.  
* **Acceptance scenarios:** see [US-14.3](#us-143--prevent-misplacement-via-barcode-verification) under Acceptance Criteria.

### US-14.4: Internal Bin-to-Bin Stock Transfer
**As a** warehouse stocker  
**I want to** move inventory from an overstocked or bulk reserve bin into a designated forward picking bin  
**So that** pick faces remain replenished for order pickers  

* **Priority:** P2  
* **Independent test:** Execute transfer of 5 cases from reserve bin to pick bin; verify source bin decrements and destination bin increments.  
* **Acceptance scenarios:** see [US-14.4](#us-144--internal-bin-to-bin-stock-transfer) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST create putaway tasks for all accepted items from [Feature 13 — Receive Supplier Shipments](feature-13-receive-supplier-shipments.md).
- **FR-002**: System MUST validate that the scanned bin barcode exists in [Feature 4 — Maintain Inventory Locations](feature-4-maintain-inventory.md) and belongs to the active warehouse.
- **FR-003**: System MUST verify that the scanned product barcode matches the item being put away.
- **FR-004**: System MUST ensure that adding the putaway quantity does not exceed the bin's `max_capacity_units`.
- **FR-005**: Upon valid putaway confirmation, system MUST:
  * Increment `bins.current_quantity`.
  * Increment total item unreserved on-hand balance.
  * Decrement dock staging balance.
- **FR-006**: If a barcode mismatch occurs, system MUST reject the transaction, emit an audible alert, and display an explicit error message stating expected vs scanned values.
- **FR-007**: System MUST record worker ID, timestamp, source location, and destination bin for complete traceability.

---

## Key Entities

- **Putaway Task**: Work assignment directing a warehouse stocker to transport accepted dock goods to an assigned slot and bin.
- **Inventory Movement**: Historical audit record tracking source location, destination bin, SKU, quantity moved, worker, and timestamp.

---

## Data Model Requirements

### `putaway_tasks`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique putaway task identifier |
| `receipt_line_id` | INT / BIGINT | FK -> `receiving_receipt_lines(id)`, NOT NULL | Associated dock receipt line |
| `item_id` | INT / BIGINT | FK -> `items(id)`, NOT NULL | Item to be stored |
| `suggested_bin_id` | INT / BIGINT | FK -> `bins(id)`, NOT NULL | System-suggested target bin |
| `confirmed_bin_id` | INT / BIGINT | FK -> `bins(id)`, NULLABLE | Actual bin where items were placed |
| `assigned_worker_id`| INT / BIGINT | FK -> `workers(id)`, NULLABLE | Stocker performing putaway |
| `quantity_cases` | INT | NOT NULL, CHECK (quantity_cases > 0) | Number of cases to stock |
| `status` | VARCHAR(20) | NOT NULL, DEFAULT `'PENDING'` | Enum: `'PENDING'`, `'IN_PROGRESS'`, `'COMPLETED'` |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Task creation timestamp |
| `completed_at` | TIMESTAMP | NULLABLE | Putaway completion timestamp |

---

## Acceptance Criteria (Gherkin)

### US-14.1 — Directed putaway task suggestion

#### Scenario: System recommends primary assigned bin for received item
* **Given** 25 cases of item "DRK-COLA-01" are staged at the dock
* **And** bin "BIN-A01-S01-B01" is assigned to "DRK-COLA-01" with available capacity for 30 cases
* **When** the stocker opens the putaway task for "DRK-COLA-01"
* **Then** the system directs the stocker to location "Aisle A01, Shelf 01, Bin B01"

#### Scenario: Primary assigned bin is full and system suggests overflow bin
* **Given** 25 cases of item "DRK-COLA-01" are staged at the dock
* **And** primary bin "BIN-A01-S01-B01" is currently at 100% capacity
* **When** the stocker opens the putaway task
* **Then** the system suggests secondary overflow bin "BIN-RES-01" with available capacity
* **And** displays notice "Primary bin full. Directed to reserve location BIN-RES-01"

---

### US-14.2 — Scan and confirm putaway

#### Scenario: Successful scan and putaway execution
* **Given** a stocker is at suggested bin "BIN-A01-S01-B01" with 25 cases of "DRK-COLA-01" (Case UPC "00012345678905")
* **When** the stocker scans the bin barcode "BIN-DAL01-A01-S01-B01"
* **And** scans the case UPC "00012345678905"
* **And** confirms storing 25 cases
* **Then** the system increments `current_quantity` of "BIN-A01-S01-B01" by 25
* **And** marks dock staging quantity as 0
* **And** marks the putaway task as `"COMPLETED"`
* **And** displays confirmation "25 cases of DRK-COLA-01 successfully stocked in BIN-A01-S01-B01"

#### Scenario: Stocker attempts to confirm putaway with zero or negative quantity
* **Given** a stocker scans valid bin and item barcodes
* **When** the stocker enters putaway quantity = 0
* **And** attempts to confirm
* **Then** the system rejects confirmation
* **And** displays an inline error message "Putaway quantity must be a positive number greater than 0"
* **And** does not change inventory balances

---

### US-14.3 — Prevent misplacement via barcode verification

#### Scenario: Stocker scans wrong bin barcode
* **Given** a putaway task expects bin "BIN-DAL01-A01-S01-B01"
* **When** the stocker scans adjacent bin barcode "BIN-DAL01-A01-S01-B02"
* **Then** the system rejects the scan with an error alert
* **And** displays an error message "Scanned location BIN-02 does not match assigned destination BIN-01. Please move to assigned location."
* **And** does NOT update any inventory quantities

#### Scenario: Attempting to put away quantity exceeding bin maximum capacity
* **Given** bin "BIN-A01-S01-B01" has `current_quantity` = 80 and `max_capacity_units` = 100
* **When** the stocker attempts to put away 30 cases (totaling 110)
* **Then** the system blocks the confirmation
* **And** displays an error message "Cannot stock 30 cases: exceeds bin remaining capacity (Available space: 20 cases)"

---

### US-14.4 — Internal bin-to-bin stock transfer

#### Scenario: Successfully transfer stock between bins
* **Given** reserve bin "BIN-RES-01" contains 50 cases of "DRK-COLA-01"
* **And** pick bin "BIN-PCK-01" contains 5 cases of "DRK-COLA-01" (capacity: 30)
* **When** the stocker initiates a bin transfer of 20 cases from "BIN-RES-01" to "BIN-PCK-01"
* **And** scans both source and destination bin barcodes
* **Then** the system decrements "BIN-RES-01" to 30 cases
* **And** increments "BIN-PCK-01" to 25 cases
* **And** total warehouse on-hand stock remains unchanged at 55 cases
* **And** displays confirmation "Transferred 20 cases from BIN-RES-01 to BIN-PCK-01"

#### Scenario: Attempt to transfer more units than available in source bin
* **Given** reserve bin "BIN-RES-01" contains 10 cases of "DRK-COLA-01"
* **When** the stocker attempts to transfer 15 cases to "BIN-PCK-01"
* **Then** the system rejects the transfer
* **And** displays an error message "Cannot transfer 15 cases: source bin BIN-RES-01 only contains 10 cases"
* **And** bin balances remain unchanged

