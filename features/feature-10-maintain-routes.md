# Feature: Maintain Routes

**Feature ID:** 10
**Branch pattern:** `feature/10-maintain-routes`
**Status:** Ready
**Created:** 2026-09-19
**Input:** Keep a record of current delivery routes and corresponding drivers.
**Depends on:** — [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md), [Feature 5 — Maintain Workers](feature-5-maintain-workers.md), [Feature 9 — Maintain Customers](feature-9-maintain-customers.md)

---

## User Stories

### US-10.1: Add route
**As a** office manager
**I want to** add a route
**So that** drivers know where to deliver shipments

**Priority:** P1
**Independent test:** Add a route and view its information in a database table
**Acceptance scenarios:** see ### US-10.1 under Acceptance Criteria

### US-10.2: Edit route
**As a** office manager
**I want to** edit a route
**So that** route information is kept up-to-date

**Priority:** P1
**Independent test:** Edit a route and view its information in a database table
**Acceptance scenarios:** see ### US-10.2 under Acceptance Criteria

### US-10.3: Delete route
**As a** office manager
**I want to** delete a route
**So that** drivers don't take old routes

**Priority:** P1
**Independent test:** Delete a route and see that its information is not in the routes database table
**Acceptance scenarios:** see ### US-10.3 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers manage individual routes.
- **FR-002**: System MUST validate user form input before saving to database.
- **FR-003**: System MUST provide the user with visual confirmation of action taken after attempting to save input.
- **FR-004**: System MUST generate a route directions map with most efficient directions for any given time.

---

## Key Entities

- **Route**: a delivery route with Name, Warehouse, Customers, Primary & Secondary Drivers.

---

## Data Model Requirements

### routes

| Column | Notes |
|--------|-------|
| id | PK |
| routeName | nvarchar(100), required |
| warehouse | nvarchar(100), required |
| customers | nvarchar(255), required, customer names separated by commas |
| driverPrimary | nvarchar(100), required |
| driverSecondary | nvarchar(100), required |

---

## Acceptance Criteria (Gherkin)

### US-6.1 — Add route

#### Scenario: User adds route successfully
* **Given** an office manager fills in the "add route" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "add route" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-6.2 — Edit route

#### Scenario: User edits route successfully
* **Given** an office manager fills in the "edit route" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "edit route" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-6.3 — Delete route

#### Scenario: User deletes route successfully
* **Given** an office manager clicks the "delete route" button
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager clicks the "delete route" button
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation