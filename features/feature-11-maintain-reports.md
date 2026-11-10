# Feature: Maintain Reports

**Feature ID:** 11
**Branch pattern:** `feature/11-maintain-reports`
**Status:** Ready
**Created:** 2026-09-19
**Input:** Keep a record of current reports.
**Depends on:** — [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md)

---

## User Stories

### US-11.1: Add report
**As a** automated process
**I want to** add reports
**So that** office managers know when exceptions arise

**Priority:** P1
**Independent test:** Activity throws exception and the system makes a report. View report on reports webpage
**Acceptance scenarios:** see ### US-11.1 under Acceptance Criteria

### US-11.2: Edit report
**As a** office manager
**I want to** edit reports
**So that** reports are kept up-to-date

**Priority:** P1
**Independent test:** Edit and save a report. View edited report on reports webpage
**Acceptance scenarios:** see ### US-11.2 under Acceptance Criteria

### US-11.3: Delete report
**As a** office manager
**I want to** delete reports
**So that** the reports webpage is kept up-to-date

**Priority:** P1
**Independent test:** Delete a report and see that it is no longer on the webpage
**Acceptance scenarios:** see ### US-11.3 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers to manage individual reports.
- **FR-002**: System MUST validate user input before saving to database.
- **FR-003**: System MUST notify office managers when a new report is created.

---

## Key Entities

- **Report**: a report to management of exceptions in warehouse activities with Warehouse, Report Number, Report Name, and Report Type.

---

## Data Model Requirements

### reports

| Column | Notes |
|--------|-------|
| id | PK |
| warehouse | nvarchar(100), required |
| reportNumber | int, unique, required |
| reportName | nvarchar(100), required |
| reportType | enum: `ITEM_OUT_OF_STOCK` \| `ITEMS_NOT_RECEIVED` \| `ITEMS_NOT_DELIVERED` \| `ITEMS_SHIPPED`, required |

---

## Acceptance Criteria (Gherkin)

### US-11.1 — Add report

#### Scenario: System adds report successfully
* **Given** an exception is thrown in a warehouse activity
* **When** the system creates a new report
* **Then** the system notifies office managers of the new report

#### Scenario: Exception is thrown due to user error
* **Given** an exception is thrown in a warehouse activity due to user input error (exception should not have been thrown)
* **When** the system creates a new report
* **Then** the system notifies office managers of the new report

### US-11.2 — Edit report

#### Scenario: User edits report successfully
* **Given** an office manager fills in the "edit report" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager fills in the "edit report" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-11.3 — Delete report

#### Scenario: User deletes report successfully
* **Given** an office manager clicks the "delete report" button
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an office manager clicks the "delete report" button
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation