# Feature: Maintain Items

**Feature ID:** 3
**Branch pattern:** `feature/3-maintain-items`
**Status:** Ready
**Created:** 2026-09-19
**Input:** Keep a record of current inventory items.
**Depends on:** — [Feature 11 — Maintain Reports](feature-11-maintain-reports.md)

---

## User Stories

### US-3.1: Add item to inventory
**As a** office manager
**I want to** add inventory items
**So that** the inventory database is kept up-to-date

**Priority:** P1
**Independent test:** Add an item to inventory and view it in the database table
**Acceptance scenarios:** see ### US-3.1 under Acceptance Criteria

### US-3.2: Edit inventory item
**As a** office manager
**I want to** edit inventory items
**So that** the inventory database is kept up-to-date

**Priority:** P1
**Independent test:** Edit an item in the inventory and view it in the database table
**Acceptance scenarios:** see ### US-3.2 under Acceptance Criteria

### US-3.3: Delete inventory item
**As a** office manager
**I want to** delete inventory items
**So that** the inventory database is kept up-to-date

**Priority:** P1
**Independent test:** Delete an item from inventory and see that it is not in the database table
**Acceptance scenarios:** see ### US-3.3 under Acceptance Criteria

### US-3.4: Report out of stock items
**As a** automated process
**I want** to create reports for out of stock items
**So that** office managers know that there is a problem with the supplier or item min/max values

**Priority:** P1
**Independent test:** Item quantity hits zero and system creates an "item out of stock" report
**Acceptance scenarios:** see ### US-3.4 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers to manage individual items.
- **FR-002**: System MUST validate user input before saving to database.

---

## Key Entities

- **Item**: a product the company orders with SKU, Item UPC, Case UPC, Supplier, Description, Bin/Slot location, Case Quantity, Case Cost, Price, On-hand, On-order, Min, and Max. 

---

## Data Model Requirements

### items

| Column | Notes |
|--------|-------|
| id | PK |
| sku | int, unique, required |
| itemUpc | int, unique, required |
| caseUpc | int, unique, required |
| supplier | nvarchar(100), required |
| description | nvarchar(255), required |
| bin | nvarchar(8), required |
| slot | nvarchar(8), required |
| caseQuantity | int, required |
| caseCost | float, required |
| price | float, required |
| onHand | int, required |
| onOrder | int, required |
| minQuantity | int, required |
| maxQuantity | int, required |

---

## Acceptance Criteria (Gherkin)

### US-3.1 — Add item to inventory

#### Scenario: User adds item successfully
* **Given** an office manager fills in the "add item" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "add item" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-3.2 — Edit item

#### Scenario: User edits item successfully
* **Given** an office manager fills in the "edit item" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "edit item" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-3.3 — Delete item

#### Scenario: User deletes item successfully
* **Given** an office manager clicks the "delete item" button
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager clicks the "delete item" button
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-3.4 — Report out of stock items

#### Scenario: System creates report successfully
* **Given** a report has not already been created
* **When** an item quantity hits zero 
* **Then** the system creates an "item out of stock" report

#### Scenario: Picker logs more outgoing items than needed
* **Given** a picker logs more outgoing items than items that were actually picked
* **When** an item quantity hits zero 
* **Then** the system creates an "item out of stock" report