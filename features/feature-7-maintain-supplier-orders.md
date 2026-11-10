# Feature: Maintain Supplier Orders

**Feature ID:** 7
**Branch pattern:** `feature/7-maintain-supplier-orders`
**Status:** Ready
**Created:** 2026-09-19
**Input:** Keep a record of supplier orders. Send and receive those orders.
**Depends on:** — [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md), [Feature 6 — Maintain Suppliers](feature-6-maintain-suppliers.md), [Feature 11 — Maintain Reports](feature-11-maintain-reports.md)

---

## User Stories

### US-7.1: Add order
**As a** office manager
**I want** to make a new order
**So that** the company can order low quantity items for warehouses 

**Priority:** P1
**Independent test:** Create a new order and preview the order on a webpage
**Acceptance scenarios:** see ### US-7.1 under Acceptance Criteria

### US-7.2: Edit order
**As a** office manager
**I want** to edit an order
**So that** it is possible to make revisions to an order before it is sent

**Priority:** P1
**Independent test:** Edit an existing order and preview the order on a webpage
**Acceptance scenarios:** see ### US-7.2 under Acceptance Criteria

### US-7.3: Delete order
**As a** office manager
**I want** to delete an order
**So that** incorrect orders are not sent by accident

**Priority:** P1
**Independent test:** Remove an order and see that it is not on the orders webpage
**Acceptance scenarios:** see ### US-7.3 under Acceptance Criteria

### US-7.4: Send order
**As a** office manager
**I want** to send an order to a supplier
**So that** needed items are ordered

**Priority:** P1
**Independent test:** Send an order and see that the order form is sent to the supplier
**Acceptance scenarios:** see ### US-7.4 under Acceptance Criteria

### US-7.5: Receieve Bill of Lading
**As a** stocker
**I want** to scan Bill of Lading into the system
**So that** the corresponding order is updated to reflect the received items

**Priority:** P1
**Independent test:** Scan Bill of Lading and view it on the corresponding order's webpage
**Acceptance scenarios:** see ### US-7.5 under Acceptance Criteria

### US-7.6: Log damaged/missing items
**As a** stocker
**I want** to log damaged and/or missing items
**So that** office managers and the system know to order more of the damaged/missing product
**And** that the supplier is notified

**Priority:** P1
**Independent test:** Stocker marks item on the order as damaged/missing and office manager sees the updated order form 
**Acceptance scenarios:** see ### US-7.6 under Acceptance Criteria

### US-7.7: Auto-order low items
**As a** automated process
**I want** order items when their quantity is below a calculated threshold
**So that** the inventory is kept adequately stocked for orders

**Priority:** P1
**Independent test:** Item quantity goes below minimum target and system auto-orders that item up to its maximum target quantity
**Acceptance scenarios:** see ### US-7.7 under Acceptance Criteria

### US-7.8: Stock order
**As a** driver
**I want** to mark an order as ready to be stocked
**So that** stockers are notified

**Priority:** P1
**Independent test:** Mark an order as ready for stocking and view its status on the orders webpage
**Acceptance scenarios:** see ### US-7.8 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST receive and parse Bill of Lading images.
- **FR-002**: System MUST automatically order items up to their calculated maximum quantity when their current quantitiy goes below their calculated minimum quantity. 
- **FR-003**: System MUST validate user form input before saving to database.
- **FR-004**: System MUST provide the user with visual confirmation of action taken after attempting to save input.

---

## Key Entities

- **Supplier Order**: supplier order with Warehouse, Supplier, Order Date, and Authorizor.

---

## Data Model Requirements

### supplierOrders

| Column | Notes |
|--------|-------|
| id | PK |
| warehouse | nvarchar(100), required |
| supplier | nvarchar(100), required |
| orderDate | date, required |
| authorizer | nvarchar(100), required |

---

## Acceptance Criteria (Gherkin)

### US-7.1 — Add order

#### Scenario: User adds order successfully
* **Given** an office manager fills in the "add order" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "add order" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-7.2 — Edit order

#### Scenario: User edits order successfully
* **Given** an office manager fills in the "edit order" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "edit order" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-7.3 — Delete order

#### Scenario: User deletes order successfully
* **Given** an office manager clicks the "delete order" button
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager clicks the "delete order" button
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-7.4 — Send order

#### Scenario: User sends order successfully
* **Given** the order exists and has necessary information
* **When** an office manager clicks the "send order" button
* **Then** the system sends the order to the supplier
* **And** the user receives visual confirmation

#### Scenario: User sends duplicate order
* **Given** the order exists and has necessary information and has already been sent
* **When** an office manager clicks the "send order" button
* **Then** the system sends a duplicate order to the supplier
* **And** the user receives visual confirmation

### US-7.5 — Receive Bill of Lading

#### Scenario: User scans Bill of Lading successfully
* **Given** the Bill of Lading has necessary information
* **When** a stocker scans the Bill of Lading with a mobile device's camera
* **Then** the system receives the Bill of Lading
* **And** the system updates the order with Bill of Lading information
* **And** the user receives visual confirmation

#### Scenario: User scans duplicate Bill of Lading
* **Given** the Bill of Lading has necessary information
* **When** a stocker scans the Bill of Lading with a mobile device's camera for a second time
* **Then** the system receives the Bill of Lading
* **And** the system updates the order with duplicate Bill of Lading information
* **And** the user receives visual confirmation

### US-7.6 — Log damaged/missing items

#### Scenario: User logs items successfully
* **Given** the corresponding order exists
* **When** a stocker marks an order item as damaged or missing
* **Then** the system updates the order
* **And** management is sent a report
* **And** the user receives visual confirmation

#### Scenario: User logs duplicate items
* **Given** the corresponding order exists
* **When** a stocker marks an order item as damaged or missing for a second time
* **Then** the system updates the order with duplicate information
* **And** management is sent a report
* **And** the user receives visual confirmation

### US-7.7 — Auto-order low items

#### Scenario: System auto-orders items successfully
* **Given** the specified items are below their minimum target quantity
* **When** the system creates an auto-order
* **Then** the system sends the order to the supplier

#### Scenario: Item quantity is inaccurate
* **Given** the specified items are reported below their minimum target quantity, but their actual quantity is higher
* **When** the system creates an auto-order
* **Then** the system sends the order to the supplier for too much of the specified items

### US-7.8 — Stock order

#### Scenario: User marks order as ready for stocking successfully
* **Given** the corresponding order exists with Bill of Lading
* **When** a driver marks an order item as ready for stocking
* **Then** the system updates the order in the database/website
* **Then** the system notifies stockers
* **And** the user receives visual confirmation

#### Scenario: User marks order as ready for stocking a second time
* **Given** the corresponding order exists and is marked as ready for stocking
* **When** a driver marks an order item as ready for stocking for a second time
* **Then** the system updates the order in the database/website
* **Then** the system notifies stockers again
* **And** the user receives visual confirmation