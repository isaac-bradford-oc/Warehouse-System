# Feature: Maintain Suppliers

**Feature ID:** 6
**Branch pattern:** `feature/6-maintain-suppliers`
**Status:** Ready
**Created:** 2026-09-19
**Input:** Keep a record of current suppliers and their information.
**Depends on:** — [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md)

---

## User Stories

### US-6.1: Add supplier
**As a** office manager
**I want to** add a supplier
**So that** the company can order items from that supplier

**Priority:** P1
**Independent test:** Add a supplier and view its information in a database table
**Acceptance scenarios:** see ### US-6.1 under Acceptance Criteria

### US-6.2: Edit supplier
**As a** office manager
**I want to** edit a supplier
**So that** supplier information is kept up-to-date

**Priority:** P1
**Independent test:** Edit a supplier and view its information in a database table
**Acceptance scenarios:** see ### US-6.2 under Acceptance Criteria

### US-6.3: Delete supplier
**As a** office manager
**I want to** delete a supplier
**So that** the company doesn't order from the wrong suppliers

**Priority:** P1
**Independent test:** Delete a supplier and see that its information is not in the suppliers database table
**Acceptance scenarios:** see ### US-6.3 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers to manage individual suppliers.
- **FR-002**: System MUST validate user form input before saving to database.
- **FR-003**: System MUST provide the user with visual confirmation of action taken after attempting to save input. 

---

## Key Entities

- **Supplier**: a company supplier with Company, Name, Address, Customer Number, P.O. Number, Ship Days, Terms, and Min Order.

---

## Data Model Requirements

### suppliers

| Column | Notes |
|--------|-------|
| id | PK |
| company | nvarchar(100), required |
| supplierName | nvarchar(100), required |
| supplierAddress | nvarchar(255), required |
| customerNumber | int, required |
| postOfficeNumber | int, required |
| supplierShipDays | int, required |
| supplierTerms | int, required |
| supplierMinOrder | int, required |

---

## Acceptance Criteria (Gherkin)

### US-6.1 — Add supplier

#### Scenario: User adds supplier successfully
* **Given** an office manager fills in the "add supplier" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "add supplier" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-6.2 — Edit supplier

#### Scenario: User edits supplier successfully
* **Given** an office manager fills in the "edit supplier" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "edit supplier" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-6.3 — Delete supplier

#### Scenario: User deletes supplier successfully
* **Given** an office manager clicks the "delete supplier" button
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager clicks the "delete supplier" button
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation