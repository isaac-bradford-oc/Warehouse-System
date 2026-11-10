# Feature: Maintain Workers

**Feature ID:** 5
**Branch pattern:** `feature/5-maintain-workers`
**Status:** Ready
**Created:** 2026-09-19
**Input:** Keep current workers' information updated.
**Depends on:** — [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md)

---

## User Stories

### US-5.1: Add worker
**As a** office manager
**I want to** add a worker to the database
**So that** the table of workers remains up-to-date

**Priority:** P1
**Independent test:** Add a worker to the database and view it in a database table
**Acceptance scenarios:** see ### US-5.1 under Acceptance Criteria

### US-5.2: Edit worker
**As a** office manager
**I want to** edit a worker in the database
**So that** worker information remains up-to-date

**Priority:** P1
**Independent test:** Edit a worker in the database and view it in a database table
**Acceptance scenarios:** see ### US-5.2 under Acceptance Criteria

### US-5.3: Delete worker
**As a** office manager
**I want to** delete a worker from the database
**So that** the table of workers remains up-to-date

**Priority:** P1
**Independent test:** Remove a worker from the database and see that it is not in a database table
**Acceptance scenarios:** see ### US-5.3 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers manage individual routes.
- **FR-002**: System MUST validate user form input before saving to database.
- **FR-003**: System MUST provide the user with visual confirmation of action taken after attempting to save input.

---

## Key Entities

- **Worker**: a warehouse worker with name, warehouse, and role (`picker` | `stocker` | `driver`)

---

## Data Model Requirements

### users

| Column | Notes |
|--------|-------|
| id | PK |
| workerName | nvarchar(100), unique, required |
| warehouse | nvarchar(100), required |
| role | enum: `PICKER` \| `STOCKER` \| `DRIVER`, required |

---

## Acceptance Criteria (Gherkin)

### US-5.1 — Add worker

#### Scenario: User adds worker successfully
* **Given** an office manager fills in the "add worker" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "add worker" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-5.2 — Edit worker

#### Scenario: User edits worker successfully
* **Given** an office manager fills in the "edit worker" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "edit worker" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-5.3 — Delete worker

#### Scenario: User deletes worker successfully
* **Given** an office manager clicks the "delete worker" button
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager clicks the "delete worker" button
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation