# Feature: Maintain Warehouses

**Feature ID:** 2
**Branch pattern:** `feature/2-maintain-warehouses`
**Status:** Ready
**Created:** 2026-09-19
**Input:** Keep a record of current company warehouses and their information.
**Depends on:** — [Feature 1 — Maintain Companies](feature-1-maintain-companies.md)

---

## User Stories

### US-2.1: Add warehouse
**As a** company admin
**I want to** add a warehouse
**So that** the company can use another physical warehouse

**Priority:** P1
**Independent test:** Add a warehouse and view its information in a database table
**Acceptance scenarios:** see ### US-2.1 under Acceptance Criteria

### US-2.2: Edit warehouse
**As a** company admin
**I want to** edit a warehouse
**So that** warehouse information is kept up-to-date

**Priority:** P1
**Independent test:** Edit a warehouse and view its information in a database table
**Acceptance scenarios:** see ### US-2.2 under Acceptance Criteria

### US-2.3: Delete warehouse
**As a** company admin
**I want to** delete a warehouse
**So that** old warehouse data isn't used by accident

**Priority:** P1
**Independent test:** Delete a warehouse and see that its information is not in the warehouses database table
**Acceptance scenarios:** see ### US-2.3 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow company admins to manage individual warehouses.
- **FR-002**: System MUST validate user form input before saving to database.
- **FR-003**: System MUST provide the user with visual confirmation of action taken after attempting to save input. 

---

## Key Entities

- **Warehouse**: a company warehouse with Company, Name, and Address.

---

## Data Model Requirements

### warehouses

| Column | Notes |
|--------|-------|
| id | PK |
| company | nvarchar(100), required |
| warehouseName | nvarchar(100), required |
| warehouseAddress | nvarchar(255), required |

---

## Acceptance Criteria (Gherkin)

### US-2.1 — Add warehouse

#### Scenario: User adds warehouse successfully
* **Given** an company admin fills in the "add warehouse" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an company admin fills in the "add warehouse" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-2.2 — Edit warehouse

#### Scenario: User edits warehouse successfully
* **Given** an company admin fills in the "edit warehouse" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an company admin fills in the "edit warehouse" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-2.3 — Delete warehouse

#### Scenario: User deletes warehouse successfully
* **Given** a company admin clicks the "delete warehouse" button
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** a company admin clicks the "delete warehouse" button
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation