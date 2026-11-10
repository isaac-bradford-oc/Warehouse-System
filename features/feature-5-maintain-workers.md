# Feature 5: Maintain Workers

**Feature ID:** 5  
**Branch pattern:** `feature/5-maintain-workers`  
**Status:** Ready  
**Created:** 2026-09-19  
**Last Updated:** 2026-11-10  
**Input:** Manage records and warehouse assignments for operational personnel.  
**Depends on:** [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md)  

---

## User Stories

### US-5.1: Add Worker
**As a** company admin or office manager  
**I want to** register a new warehouse worker with an assigned facility and operational role  
**So that** the worker can log in, access role-specific workflows (picking, stocking, delivery), and be assigned operational tasks  

* **Priority:** P1  
* **Independent test:** Submit a valid worker registration form; verify the worker record is stored in the database with correct role and warehouse affiliation.  
* **Acceptance scenarios:** see [US-5.1](#us-51--add-worker) under Acceptance Criteria.

### US-5.2: Edit Worker
**As an** office manager  
**I want to** update a worker's role, contact details, or assigned warehouse  
**So that** employee records accurately reflect promotions, reassignments, or updated contact info  

* **Priority:** P1  
* **Independent test:** Update a worker's assigned role and contact number; verify updated values persist and take effect immediately.  
* **Acceptance scenarios:** see [US-5.2](#us-52--edit-worker) under Acceptance Criteria.

### US-5.3: Delete / Deactivate Worker
**As an** office manager  
**I want to** deactivate or remove a worker who has left the company  
**So that** former employees can no longer log in or be assigned to active routes and pick lists  

* **Priority:** P1  
* **Independent test:** Deactivate a worker with active route history; verify account is marked inactive and blocked from login without corrupting audit history.  
* **Acceptance scenarios:** see [US-5.3](#us-53--delete--deactivate-worker) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow company admins and office managers to manage individual warehouse worker profiles.
- **FR-002**: System MUST associate each worker with a valid `company_id` and an assigned primary `warehouse_id`.
- **FR-003**: System MUST require each worker to have a unique `employee_id` and unique `email` within the system.
- **FR-004**: System MUST validate that the worker's `role` belongs to the defined set of operational roles: `PICKER`, `STOCKER`, `RECEIVING_CLERK`, `DRIVER`, `OFFICE_MANAGER`.
- **FR-005**: System MUST prevent hard deletion of any worker who has historical transaction records (picks, receiving logs, or routes); system MUST support deactivation (`status = 'INACTIVE'`) to preserve audit trails.
- **FR-006**: System MUST return descriptive confirmation or explicit validation error messages for all form operations.

---

## Key Entities

- **Worker**: Operational personnel profile with employee ID, full name, email, phone, role (`PICKER` | `STOCKER` | `RECEIVING_CLERK` | `DRIVER` | `OFFICE_MANAGER`), assigned warehouse facility, and employment status (`ACTIVE` | `INACTIVE`).

---

## Data Model Requirements

### `workers`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique internal worker ID |
| `company_id` | INT / BIGINT | FK -> `companies(id)`, NOT NULL | Owning tenant company |
| `warehouse_id` | INT / BIGINT | FK -> `warehouses(id)`, NOT NULL | Primary assigned warehouse |
| `employee_id` | VARCHAR(30) | NOT NULL, UNIQUE | Company badge / employee identifier |
| `worker_name` | VARCHAR(100) | NOT NULL | Full employee name (2–100 chars) |
| `email` | VARCHAR(100) | NOT NULL, UNIQUE | Corporate or operational login email |
| `phone_number` | VARCHAR(20) | NOT NULL | Contact telephone number |
| `role` | VARCHAR(30) | NOT NULL | Enum: `'PICKER'`, `'STOCKER'`, `'RECEIVING_CLERK'`, `'DRIVER'`, `'OFFICE_MANAGER'` |
| `status` | VARCHAR(20) | NOT NULL, DEFAULT `'ACTIVE'` | Enum: `'ACTIVE'`, `'INACTIVE'` |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp created |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp updated |

---

## Acceptance Criteria (Gherkin)

### US-5.1 — Add worker

#### Scenario: Successfully register a new warehouse worker
* **Given** warehouse "DAL-01" exists with ID `2` under company ID `1`
* **When** an office manager submits the "Add Worker" form with:
  | Field | Value |
  | `employee_id` | "EMP-1042" |
  | `worker_name` | "Marcus Johnson" |
  | `email` | "m.johnson@apexlogistics.com" |
  | `phone_number` | "+1-214-555-0144" |
  | `warehouse_id` | 2 |
  | `role` | "PICKER" |
* **Then** the system creates the worker record with status `"ACTIVE"`
* **And** displays confirmation "Worker Marcus Johnson (EMP-1042) registered successfully"

#### Scenario: Attempt to register worker with existing email
* **Given** a worker with email "m.johnson@apexlogistics.com" already exists
* **When** the office manager submits a new worker with email "m.johnson@apexlogistics.com"
* **Then** the system rejects the submission
* **And** displays an error message "Email address m.johnson@apexlogistics.com is already registered"

#### Scenario: Attempt to register worker with invalid role
* **Given** the office manager submits the form with role "SUPERVISOR" (not in allowed enum)
* **When** the form is submitted
* **Then** the system rejects the submission
* **And** displays an inline error message "Role must be one of: PICKER, STOCKER, RECEIVING_CLERK, DRIVER, OFFICE_MANAGER"

---

### US-5.2 — Edit worker

#### Scenario: Successfully update worker role and warehouse
* **Given** an existing worker "Marcus Johnson" with role "PICKER"
* **When** the office manager updates `role` to "STOCKER" and `phone_number` to "+1-214-555-9900"
* **And** submits the edit form
* **Then** the system saves the updated role and phone number
* **And** displays confirmation "Worker details updated successfully"

#### Scenario: Attempt to update worker email to another worker's registered email
* **Given** worker A has email "m.johnson@apexlogistics.com" and worker B has email "j.smith@apexlogistics.com"
* **When** the office manager edits worker B and changes email to "m.johnson@apexlogistics.com"
* **And** submits the form
* **Then** the system rejects the update
* **And** displays an error message "Email address m.johnson@apexlogistics.com is already registered to another worker"
* **And** worker B's email remains unchanged

---

### US-5.3 — Delete / Deactivate worker

#### Scenario: Successfully deactivate a worker
* **Given** an active worker "Marcus Johnson" who is currently assigned to past pick orders
* **When** the office manager selects "Deactivate Worker"
* **And** confirms the deactivation in the modal dialog
* **Then** the system sets the worker's status to `"INACTIVE"`
* **And** revokes the worker's active session
* **And** prevents the worker from appearing in dispatch assignment lists
* **And** displays confirmation "Worker Marcus Johnson has been deactivated"

#### Scenario: Hard delete permitted only when no historical records exist
* **Given** a newly registered worker "Jane Doe" with zero assigned orders, routes, or scans
* **When** the office manager clicks "Delete Worker" and confirms
* **Then** the system removes the record from the database
* **And** displays confirmation "Worker Jane Doe deleted successfully"
