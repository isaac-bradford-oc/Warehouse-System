# Feature: Maintain Companies

**Feature ID:** 1
**Branch pattern:** `feature/1-maintain-companies`
**Status:** Ready
**Created:** 2026-09-19
**Input:** Keep a record of all registered companies and their current information.
**Depends on:** — 

---

## User Stories

### US-1.1: Add company
**As a** admin
**I want to** add a company
**So that** we (creators of the software solution) can add new companies to our system

**Priority:** P1
**Independent test:** Add a company and view its information in a database table
**Acceptance scenarios:** see ### US-1.1 under Acceptance Criteria

### US-1.2: Edit company
**As a** admin
**I want to** edit a company
**So that** company information is kept up-to-date

**Priority:** P1
**Independent test:** Edit a company and view its information in a database table
**Acceptance scenarios:** see ### US-1.2 under Acceptance Criteria

### US-1.3: Delete company
**As a** admin
**I want to** delete a company
**So that** we can delete data for old customers

**Priority:** P1
**Independent test:** Delete a company and see that its information is not in the companies database table
**Acceptance scenarios:** see ### US-1.3 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow admins to manage individual companies.
- **FR-002**: System MUST validate user form input before saving to database.
- **FR-003**: System MUST provide the user with visual confirmation of action taken after attempting to save input. 

---

## Key Entities

- **Company**: a company with Name, Address, and Phone Number.

---

## Data Model Requirements

### companies

| Column | Notes |
|--------|-------|
| id | PK |
| companyName | nvarchar(100), required |
| companyAddress | nvarchar(255), required |
| companyPhoneNumber | int, required |

---

## Acceptance Criteria (Gherkin)

### US-1.1 — Add company

#### Scenario: User adds company successfully
* **Given** an admin fills in the "add company" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an admin fills in the "add company" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-1.2 — Edit company

#### Scenario: User edits company successfully
* **Given** an admin fills in the "edit company" form with acceptable information
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an admin fills in the "edit company" form with invalid information
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation

### US-1.3 — Delete company

#### Scenario: User deletes company successfully
* **Given** an admin clicks the "delete company" button
* **When** the user saves the form
* **Then** the system saves the form to the database
* **And** the user receives visual confirmation

#### Scenario: User inputs invalid information into the form
* **Given** an admin clicks the "delete company" button
* **When** the user saves the form
* **Then** the system throws an exception
* **And** the user receives visual confirmation