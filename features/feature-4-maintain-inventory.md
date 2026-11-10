# Feature 4: Maintain Inventory Locations

**Feature ID:** 4  
**Branch pattern:** `feature/4-maintain-inventory`  
**Status:** Ready  
**Created:** 2026-09-19  
**Last Updated:** 2026-11-10  
**Input:** Manage physical warehouse inventory storage infrastructure, including aisles, slots, and bin locations.  
**Depends on:** [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md), [Feature 3 — Maintain Items](feature-3-maintain-items.md)  

---

## User Stories

### US-4.1: Add Inventory Slot
**As an** office manager  
**I want to** define a new storage slot within a warehouse aisle and shelf  
**So that** the facility's physical storage layout is accurately mapped in the system  

* **Priority:** P1  
* **Independent test:** Create a slot with aisle, shelf, and slot code in a designated warehouse; verify it appears in the warehouse location hierarchy.  
* **Acceptance scenarios:** see [US-4.1](#us-41--add-inventory-slot) under Acceptance Criteria.

### US-4.2: Edit Inventory Slot
**As an** office manager  
**I want to** modify a slot's physical description, shelf level, or maximum bin capacity  
**So that** changes to warehouse racking and layout are kept up-to-date  

* **Priority:** P1  
* **Independent test:** Update a slot's capacity and notes; confirm changes persist in the database.  
* **Acceptance scenarios:** see [US-4.2](#us-42--edit-inventory-slot) under Acceptance Criteria.

### US-4.3: Delete / Deactivate Inventory Slot
**As an** office manager  
**I want to** remove or deactivate an empty storage slot  
**So that** decommissioned racking or reconfigured aisles are removed from active operations  

* **Priority:** P1  
* **Independent test:** Attempt to delete an occupied slot (blocked); delete an empty slot with zero bins/stock (succeeds).  
* **Acceptance scenarios:** see [US-4.3](#us-43--delete--deactivate-inventory-slot) under Acceptance Criteria.

### US-4.4: Add Bin to Slot
**As an** office manager  
**I want to** add a designated bin location inside an existing slot and assign an optional primary item  
**So that** inventory can be physically organized and scanned at bin-level precision  

* **Priority:** P1  
* **Independent test:** Create a bin within a slot; verify it is associated with the slot and visible for stocking.  
* **Acceptance scenarios:** see [US-4.4](#us-44--add-bin-to-slot) under Acceptance Criteria.

### US-4.5: Edit Bin Configuration
**As an** office manager  
**I want to** update a bin's assigned item, maximum unit capacity, or barcode label  
**So that** reslotting and bin capacity limits are maintained accurately  

* **Priority:** P1  
* **Independent test:** Reassign an empty bin to a new SKU and save; verify the bin reflects the new assigned item.  
* **Acceptance scenarios:** see [US-4.5](#us-45--edit-bin-configuration) under Acceptance Criteria.

### US-4.6: Delete / Deactivate Bin
**As an** office manager  
**I want to** remove or retire an empty bin from a slot  
**So that** removed containers are no longer suggested for putaway  

* **Priority:** P1  
* **Independent test:** Attempt deletion of a bin containing inventory (blocked); delete an empty bin (succeeds).  
* **Acceptance scenarios:** see [US-4.6](#us-46--delete--deactivate-bin) under Acceptance Criteria.

*(Note: Real-time operational stock movements, picking, and receiving putaway have been extracted into dedicated operational features: [Feature 14 — Putaway and Stock Inventory](feature-14-putaway-and-stock-inventory.md) and [Feature 15 — Pick Customer Orders](feature-15-pick-customer-orders.md)).*

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers to create, view, update, and delete/deactivate slots and bins.
- **FR-002**: Every slot MUST link to a valid `warehouse_id`.
- **FR-003**: System MUST enforce that `slot_code` is unique within each warehouse (e.g., `A-01-S01`).
- **FR-004**: Every bin MUST link to a valid `slot_id`.
- **FR-005**: System MUST enforce that `bin_code` is unique within each slot (e.g., `BIN-A1`, `BIN-A2`).
- **FR-006**: System MUST prevent deletion of any slot or bin that currently has positive `current_quantity` of inventory.
- **FR-007**: System MUST validate that `max_capacity_units` is greater than zero and `current_quantity` cannot exceed `max_capacity_units`.
- **FR-008**: System MUST return descriptive confirmation or explicit validation error messages for all form operations.

---

## Key Entities

- **Slot**: A discrete physical racking section within a warehouse, defined by aisle, shelf, unique slot code, and maximum bin capacity.
- **Bin**: A specific compartment or container within a slot where items are placed, defined by unique bin code, barcode string, optional assigned SKU, maximum unit capacity, and current unit count.

---

## Data Model Requirements

### `slots`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique slot identifier |
| `warehouse_id` | INT / BIGINT | FK -> `warehouses(id)`, NOT NULL | Owning warehouse |
| `slot_code` | VARCHAR(30) | NOT NULL | Human-readable rack/slot code (e.g., `A01-R02-S03`) |
| `aisle` | VARCHAR(10) | NOT NULL | Aisle identifier (e.g., `A01`) |
| `shelf` | VARCHAR(10) | NOT NULL | Shelf level identifier (e.g., `R02`) |
| `total_bin_capacity` | INT | NOT NULL, CHECK (total_bin_capacity > 0) | Maximum number of bins this slot can hold |
| `status` | VARCHAR(20) | NOT NULL, DEFAULT `'ACTIVE'` | Enum: `'ACTIVE'`, `'INACTIVE'` |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp created |

* **Unique Index:** `(warehouse_id, slot_code)`

### `bins`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique bin identifier |
| `slot_id` | INT / BIGINT | FK -> `slots(id)`, NOT NULL | Parent slot location |
| `bin_code` | VARCHAR(30) | NOT NULL | Local bin identifier (e.g., `BIN-01`) |
| `barcode` | VARCHAR(50) | NOT NULL, UNIQUE | Scannable QR/barcode on the bin label |
| `assigned_item_id` | INT / BIGINT | FK -> `items(id)`, NULLABLE | Optional dedicated product SKU |
| `current_quantity` | INT | NOT NULL, DEFAULT 0, CHECK (current_quantity >= 0) | Current units physically inside bin |
| `max_capacity_units`| INT | NOT NULL, CHECK (max_capacity_units > 0) | Maximum units this bin can accommodate |
| `status` | VARCHAR(20) | NOT NULL, DEFAULT `'ACTIVE'` | Enum: `'ACTIVE'`, `'INACTIVE'` |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp created |

* **Unique Index:** `(slot_id, bin_code)`

---

## Acceptance Criteria (Gherkin)

### US-4.1 — Add inventory slot

#### Scenario: Successfully add a warehouse slot
* **Given** warehouse "DAL-01" exists with ID `2`
* **When** an office manager submits the "Add Slot" form with:
  | Field | Value |
  | `slot_code` | "A01-S01" |
  | `aisle` | "A01" |
  | `shelf` | "01" |
  | `total_bin_capacity` | 4 |
* **Then** the system creates the slot under warehouse `2` with status `"ACTIVE"`
* **And** displays confirmation "Slot A01-S01 created successfully"

#### Scenario: Attempt to add duplicate slot in same warehouse
* **Given** a slot "A01-S01" already exists in warehouse ID `2`
* **When** the office manager attempts to create another slot with code "A01-S01" in warehouse `2`
* **Then** the system rejects the submission
* **And** displays an error message "Slot code A01-S01 already exists in this warehouse"

---

### US-4.2 — Edit inventory slot

#### Scenario: Successfully update slot capacity
* **Given** an existing slot "A01-S01" with total bin capacity 4
* **When** the office manager updates `total_bin_capacity` to 6
* **And** submits the form
* **Then** the system updates the slot record
* **And** displays confirmation "Slot A01-S01 updated successfully"

#### Scenario: Attempt to reduce slot capacity below current active bin count
* **Given** slot "A01-S01" currently contains 3 active bins
* **When** the office manager attempts to change `total_bin_capacity` to 2
* **Then** the system rejects the update
* **And** displays an error message "Cannot set capacity to 2: slot currently contains 3 active bins"
* **And** the slot capacity remains 4

---

### US-4.3 — Delete / Deactivate inventory slot

#### Scenario: Successfully delete an empty slot
* **Given** slot "A01-S01" contains 0 bins
* **When** the office manager clicks "Delete Slot" and confirms
* **Then** the system removes the slot from the database
* **And** displays confirmation "Slot A01-S01 deleted successfully"

#### Scenario: Prevent deletion of slot with active bins
* **Given** slot "A01-S01" contains 2 active bins
* **When** the office manager attempts to delete slot "A01-S01"
* **Then** the system rejects the deletion
* **And** displays an error message "Cannot delete slot: 2 active bins are assigned to this slot. Delete or reassign all bins first"

---

### US-4.4 — Add bin to slot

#### Scenario: Successfully add a bin to an existing slot
* **Given** slot "A01-S01" exists and currently contains 1 bin out of capacity 4
* **When** the office manager submits the "Add Bin" form with:
  | Field | Value |
  | `bin_code` | "BIN-02" |
  | `barcode` | "BIN-DAL01-A01-S01-B02" |
  | `max_capacity_units` | 100 |
* **Then** the system creates the bin associated with slot "A01-S01" with `current_quantity` = 0
* **And** displays confirmation "Bin BIN-02 created successfully"

#### Scenario: Prevent adding bin when slot capacity is reached
* **Given** slot "A01-S01" has `total_bin_capacity` = 2 and currently has 2 active bins
* **When** the office manager attempts to add a third bin
* **Then** the system rejects the creation
* **And** displays an error message "Cannot add bin: Slot A01-S01 has reached its maximum bin capacity of 2"

---

### US-4.5 — Edit bin configuration

#### Scenario: Successfully update assigned item on empty bin
* **Given** an empty bin "BIN-02" has no assigned item
* **When** the office manager assigns item "DRK-COLA-01" (ID `101`) to "BIN-02"
* **And** saves changes
* **Then** the system updates the bin with `assigned_item_id` = 101
* **And** displays confirmation "Bin BIN-02 updated successfully"

#### Scenario: Attempt to reduce bin capacity below current inventory quantity
* **Given** bin "BIN-01" currently contains 45 units of inventory
* **When** the office manager attempts to edit `max_capacity_units` to 30
* **Then** the system rejects the update
* **And** displays an error message "Cannot set max capacity to 30: bin currently holds 45 units"
* **And** the bin capacity remains unchanged

---

### US-4.6 — Delete / Deactivate bin

#### Scenario: Successfully delete an empty bin
* **Given** bin "BIN-02" has `current_quantity` = 0
* **When** the office manager clicks "Delete Bin" and confirms the action
* **Then** the system removes the bin from the database
* **And** displays confirmation "Bin BIN-02 deleted successfully"

#### Scenario: Block deletion when bin contains inventory
* **Given** bin "BIN-01" contains 45 units of item "DRK-COLA-01"
* **When** the office manager attempts to delete bin "BIN-01"
* **Then** the system rejects the deletion
* **And** displays an error message "Cannot delete bin: bin contains 45 units of inventory. Move or pick all items before deleting"
* **And** the bin remains active in the database
