# Feature: Maintain Customer Orders

**Feature ID:** 9
**Branch pattern:** `feature/9-maintain-customer-orders`
**Status:** Ready
**Created:** 2026-09-19
**Input:** Keep a record of customer orders. Send and receive those orders.
**Depends on:** — [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md), [Feature 9 — Maintain Customers](feature-9-maintain-customers.md), [Feature 11 — Maintain Reports](feature-11-maintain-reports.md)

---

## User Stories

### US-9.1: Add order
**As a** office manager
**I want** to create a new customer order
**So that** the company can send customers the items they need

**Priority:** P1
**Independent test:** Create a new order and preview the order on a webpage
**Acceptance scenarios:** see ### US-9.1 under Acceptance Criteria

### US-9.2: Edit order
**As a** office manager
**I want** to edit an order
**So that** it is possible to make revisions to an order before it is picked or shipped

**Priority:** P1
**Independent test:** Edit an existing order and preview the order on a webpage
**Acceptance scenarios:** see ### US-9.2 under Acceptance Criteria

### US-9.3: Delete order
**As a** office manager
**I want** to delete an order
**So that** incorrect or completed orders are not picked or shipped by accident

**Priority:** P1
**Independent test:** Remove an order and see that it is not on the orders webpage
**Acceptance scenarios:** see ### US-9.3 under Acceptance Criteria

### US-9.4: Pick order
**As a** office manager
**I want** to mark a customer order as ready to be picked
**So that** pickers know to pick the order

**Priority:** P1
**Independent test:** Mark an order as ready for picking and view its status on the orders webpage
**Acceptance scenarios:** see ### US-9.4 under Acceptance Criteria

### US-9.5: Ship order
**As a** picker
**I want** to mark a customer order shipped
**So that** drivers knows which orders are ready to be delivered

**Priority:** P1
**Independent test:** Mark an order as shipped and view its status on the orders webpage
**Acceptance scenarios:** see ### US-9.5 under Acceptance Criteria

### US-9.6: Create Bill of Lading
**As a** automated process
**I want** create Bill of Lading after order is marked as ready 
**So that** the customer knows which items the company shipped

**Priority:** P1
**Independent test:** Mark as shipped and view Bill of Lading corresponding order's webpage
**Acceptance scenarios:** see ### US-9.6 under Acceptance Criteria

### US-9.7: Deliver order
**As a** driver
**I want** to mark a customer order as delivered
**So that** company knows which orders were delivered

**Priority:** P1
**Independent test:** Mark an order as delivered and view its status on the orders webpage
**Acceptance scenarios:** see ### US-9.7 under Acceptance Criteria

### US-9.8: Log damaged/missing items
**As a** driver
**I want** to log damaged and/or missing items
**So that** office managers know to audit processes to see where the incident may take place

**Priority:** P1
**Independent test:** Driver marks item on the order as damaged/missing and office manager sees the updated order form 
**Acceptance scenarios:** see ### US-9-8 under Acceptance Criteria



---

## Requirements

### Functional Requirements

- **FR-001**: System MUST generate Bills of Lading.
- **FR-002**: System MUST modify Bills of Lading with damaged/missing items. 
- **FR-003**: System MUST validate user form input before saving to database.
- **FR-004**: System MUST provide the user with visual confirmation of action taken after attempting to save input.
- **FR-005**: System MUST notify customers when an order is on its way. 

---

## Key Entities

- **Customer Order**: customer order with Warehouse, Customer, Order Date, and Authorizer.

---

## Data Model Requirements

### customerOrders

| Column | Notes |
|--------|-------|
| id | PK |
| warehouse | nvarchar(100), required |
| customer | nvarchar(100), required |
| orderDate | date, required |
| authorizer | nvarchar(100), required |

---

## Acceptance Criteria (Gherkin)

### US-9.1 — Add order

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

### US-9.2 — Edit order

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

### US-9.3 — Delete order

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

### US-9.4 — Pick order

#### Scenario: User marks order for picking successfully
* **Given** the order exists and has the necessary information
* **When** an office manager clicks the "ready for picking" button
* **Then** the system notifies pickers and updates the database accordingly
* **And** the user receives visual confirmation

#### Scenario: User marks order for picking a second time
* **Given** the order exists and has the necessary information
* **When** an office manager clicks the "ready for picking" button for a second time
* **Then** the system notifies pickers again and updates the database/website
* **And** the user receives visual confirmation

### US-9.5 — Ship order

#### Scenario: User marks order as shipped successfully
* **Given** the order exists and is marked as ready for picking
* **When** a picker clicks the "ship order" button
* **Then** the system notifies drivers
* **And** the system updates the database/website
* **And** the user receives visual confirmation

#### Scenario: User marks order as shipped a second time
* **Given** the order exists and is marked as shipped
* **When** a picker clicks the "ship order" button
* **Then** the system notifies drivers again
* **And** the system updates the database/website
* **And** the user receives visual confirmation

### US-9.6 — Create Bill of Lading

#### Scenario: System creates Bill of Lading successfully
* **Given** the corresponding order exists
* **When** a picker marks an order as shipped
* **Then** the system creates a Bill of Lading
* **And** the system sends the Bill of Lading to appropriate drivers

#### Scenario: Picker marks order as shipped before finished picking
* **Given** the corresponding order exists and the picking process is not completed
* **When** a picker marks an order as shipped
* **Then** the system creates a Bill of Lading without all of the needed items
* **And** the system sends the Bill of Lading to appropriate drivers

### US-9.7 — Deliver order

#### Scenario: User marks order as delivered successfully
* **Given** the order is delivered to customer
* **When** the driver marks order as delivered
* **Then** the system updates the database/website
* **And** the user receives visual confirmation

#### Scenario: User marks order as delivered before finished delivering
* **Given** the order is not delivered to customer
* **When** the driver marks order as delivered
* **Then** the system updates the database/website
* **And** the user receives visual confirmation

### US-9.8 — Log damaged/missing items

#### Scenario: User logs items successfully
* **Given** the corresponding order exists and is marked as shipped
* **When** a driver marks an order item as damaged or missing
* **Then** the system updates the order
* **And** management is sent a report
* **And** the user receives visual confirmation

#### Scenario: User logs duplicate items
* **Given** the corresponding order exists
* **When** a driver marks an order item as damaged or missing for a second time
* **Then** the system updates the order with duplicate information
* **And** management is sent a report
* **And** the user receives visual confirmation