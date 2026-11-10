# Feature: Maintain Inventory

**Feature ID:** 4
**Branch pattern:** `feature/4-maintain-inventory`
**Status:** Ready
**Created:** 2026-09-19
**Input:** Keep record of active inventory locations, including what kind and of how many items are in those locations. 
**Depends on:** — [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md)

---

## User Stories

### US-4.1: Add slot
**As a** office manager
**I want to** add inventory slots
**So that** inventory locations are kept up-to-date

**Priority:** P1
**Independent test:** Add an inventory slot and view it in the database table
**Acceptance scenarios:** see ### US-4.1 under Acceptance Criteria

### US-4.2: Edit slot
**As a** office manager
**I want to** edit inventory slots
**So that** inventory locations are kept up-to-date

**Priority:** P1
**Independent test:** Edit an inventory slot and view it in the database table
**Acceptance scenarios:** see ### US-4.2 under Acceptance Criteria

### US-4.3: Delete slot
**As a** office manager
**I want to** delete inventory slots
**So that** inventory locations are kept up-to-date

**Priority:** P1
**Independent test:** Delete an inventory slot and see that it is not in the database table
**Acceptance scenarios:** see ### US-4.3 under Acceptance Criteria

### US-4.4: Add bin
**As a** office manager
**I want to** add inventory bins
**So that** inventory locations are kept up-to-date

**Priority:** P1
**Independent test:** Add an inventory bin and view it in the database table
**Acceptance scenarios:** see ### US-4.4 under Acceptance Criteria

### US-4.5: Edit bin
**As a** office manager
**I want to** edit inventory bins
**So that** inventory locations are kept up-to-date

**Priority:** P1
**Independent test:** Edit an inventory bin and view it in the database table
**Acceptance scenarios:** see ### US-4.5 under Acceptance Criteria

### US-4.6: Delete bin
**As a** office manager
**I want to** delete inventory bins
**So that** inventory locations are kept up-to-date

**Priority:** P1
**Independent test:** Delete an inventory bin and see that it is not in the database table
**Acceptance scenarios:** see ### US-4.6 under Acceptance Criteria

### US-4.7: Log outgoing items
**As a** picker
**I want to** log outgoing items as I pick them
**So that** the recorded quantity of items in slot bins and slots are kept up-to-date

**Priority:** P1
**Independent test:** Scan the slot's QR code and log outgoing slots. Inventory database reflects changes
**Acceptance scenarios:** see ### US-4.7 under Acceptance Criteria

### US-4.8: Log incoming items
**As a** stocker
**I want** to log incoming items as I stock them
**So that** the recorded quantity of items in slot bins and slots are kept up-to-date

**Priority:** P1
**Independent test:** Scan the slot's QR code and log incoming items. Inventory database reflects changes
**Acceptance scenarios:** see ### US-4.8 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers to manage individual inventory slots/bins.
- **FR-002**: System MUST increment and decrement item quantities as logged by pickers and stockers.
- **FR-003**: System MUST validate user form input before saving to database.
- **FR-004**: System MUST provide the user with visual confirmation of action taken after attempting to save input.
- **FR-005**: System MUST create out of stock item reports the moment a given item quantity hits zero.

---

## Key Entities

- **Slot**: inventory slot with Warehouse, Slot Number, Active Bins, and Total Bins.
- **Bin**: slot bin with Bin Number and Item Name.

---

## Data Model Requirements

### slots

| Column | Notes |
|--------|-------|
| id | PK |
| warehouse | nvarchar(100), required |
| slotNumber | int, unique, required |
| activeBins | nvarchar(100), required, bin numbers separated by commas |
| totalBins | int, required |

### bins

| Column | Notes |
|--------|-------|
| id | PK |
| binNumber | int, unique, required |
| itemName | nvarchar(100), required |

---

## Acceptance Criteria (Gherkin)

### US-4.1 — Add slot

#### Scenario: User adds slot successfully
* **Given** an office manager fills in the "add slot" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "add slot" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-4.2 — Edit slot

#### Scenario: User edits slot successfully
* **Given** an office manager fills in the "edit slot" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "edit slot" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-4.3 — Delete slot

#### Scenario: User deletes slot successfully
* **Given** an office manager clicks the "delete slot" button
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager clicks the "delete slot" button
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-4.4 — Add bin

#### Scenario: User adds bin successfully
* **Given** an office manager fills in the "add bin" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "add bin" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-4.5 — Edit bin

#### Scenario: User edits bin successfully
* **Given** an office manager fills in the "edit bin" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "edit bin" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-4.6 — Delete bin

#### Scenario: User deletes bin successfully
* **Given** an office manager clicks the "delete bin" button
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager clicks the "delete bin" button
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-4.7 — Log outgoing items

#### Scenario: User logs outgoing items successfully
* **Given** a picker has the correct slot and bin selected
* **When** the user decrements items
* **Then** the system saves the updated item quantity to the database
* **And** the user receives visual confirmation

#### Scenario: User scans the wrong slot QR code
* **Given** a picker has the incorrect slot and bin selected
* **When** the user decrements items
* **Then** the system updates the quantity of the wrong item
* **And** the user receives visual confirmation of this

### US-4.8 — Log incoming items

#### Scenario: User logs incoming items successfully
* **Given** a stocker has the correct slot and bin selected
* **When** the user increments items
* **Then** the system saves the updated item quantity to the database
* **And** the user receives visual confirmation

#### Scenario: User scans the wrong slot QR code
* **Given** a stocker has the incorrect slot and bin selected
* **When** the user increments items
* **Then** the system updates the quantity of the wrong item
* **And** the user receives visual confirmation of this