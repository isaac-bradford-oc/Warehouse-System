# Feature: Maintain Customers

**Feature ID:** 8
**Branch pattern:** `feature/8-maintain-customers`
**Status:** Ready
**Created:** 2026-09-19
**Input:** Keep a record of current company customers and their information.
**Depends on:** — [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md)

---

## User Stories

### US-8.1: Add customer
**As a** office manager
**I want to** add a customer
**So that** the company can order items from that customer

**Priority:** P1
**Independent test:** Add a customer and view its information in a database table
**Acceptance scenarios:** see ### US-8.1 under Acceptance Criteria

### US-8.2: Edit customer
**As a** office manager
**I want to** edit a customer
**So that** customer information is kept up-to-date

**Priority:** P1
**Independent test:** Edit a customer and view its information in a database table
**Acceptance scenarios:** see ### US-8.2 under Acceptance Criteria

### US-8.3: Delete customer
**As a** office manager
**I want to** delete a customer
**So that** the company doesn't order from the wrong customers

**Priority:** P1
**Independent test:** Delete a customer and see that its information is not in the customers database table
**Acceptance scenarios:** see ### US-8.3 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers to manage individual customers.
- **FR-002**: System MUST validate user form input before saving to database.
- **FR-003**: System MUST provide the user with visual confirmation of action taken after attempting to save input. 

---

## Key Entities

- **Customer**: a company customer with Company, Name, Address, Customer Number, P.O. Number, and Terms.

---

## Data Model Requirements

### customers

| Column | Notes |
|--------|-------|
| id | PK |
| company | nvarchar(100), required |
| customerName | nvarchar(100), required |
| customerAddress | nvarchar(255), required |
| customerNumber | int, required |
| postOfficeNumber | int, required |
| customerTerms | int, required |

---

## Acceptance Criteria (Gherkin)

### US-8.1 — Add customer

#### Scenario: User adds customer successfully
* **Given** an office manager fills in the "add customer" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "add customer" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-8.2 — Edit customer

#### Scenario: User edits customer successfully
* **Given** an office manager fills in the "edit customer" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "edit customer" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-8.3 — Delete customer

#### Scenario: User deletes customer successfully
* **Given** an office manager clicks the "delete customer" button
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager clicks the "delete customer" button
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation