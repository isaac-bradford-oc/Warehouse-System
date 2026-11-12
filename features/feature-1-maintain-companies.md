# Feature 1: Maintain Companies

**Feature ID:** 1  
**Branch pattern:** `feature/1-maintain-companies`  
**Status:** Ready  
**Created:** 2026-09-19  
**Last Updated:** 2026-11-10  
**Input:** Keep an accurate master record of registered tenant companies.  
**Depends on:** — None  

---

## User Stories

### US-1.1: Add Company
**As a** system admin  
**I want to** register a new company with its primary contact details  
**So that** tenant accounts are established in the warehouse management system  

* **Priority:** P1  
* **Independent test:** Submit a valid company registration form and verify the new company record exists in the system database with all required fields.  
* **Acceptance scenarios:** see [US-1.1](#us-11--add-company) under Acceptance Criteria.

### US-1.2: Edit Company
**As a** system admin  
**I want to** update existing company information (name, address, phone number)  
**So that** company profile data remains accurate over time  

* **Priority:** P1  
* **Independent test:** Update company details on an existing record and verify the updated values persist and are visible upon reload.  
* **Acceptance scenarios:** see [US-1.2](#us-12--edit-company) under Acceptance Criteria.

### US-1.3: Delete / Deactivate Company
**As a** system admin  
**I want to** delete or deactivate a company that no longer uses the platform  
**So that** inactive company records do not appear in active operational lists  

* **Priority:** P1  
* **Independent test:** Select a company without active child records and confirm deletion; verify the record is removed or marked inactive and no longer accessible in active company lists.  
* **Acceptance scenarios:** see [US-1.3](#us-13--delete--deactivate-company) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow system administrators to view, create, update, and delete/deactivate company records.
- **FR-002**: System MUST enforce unique company names across all active company records (case-insensitive).
- **FR-003**: System MUST validate all input fields against business rules before persisting to the database.
- **FR-004**: System MUST reject deletion of a company if child warehouses, users, or active transactions exist, returning an explanatory error message.
- **FR-005**: System MUST provide immediate feedback with clear confirmation or field-specific validation error messages upon every submission.

---

## Data Ownership & Isolation

Companies are the top-level tenancy boundary in the system.

| Rule | Requirement |
|---|---|
| **Read scope** | System Administrators (`admin`) can view all companies. Tenant users see only their own company profile. |
| **Write scope** | Only System Administrators (`admin`) can create, edit, or delete/deactivate company records. |
| **Create scope** | System Administrator generates new tenant root record. |
| **Cross-user access** | Any unauthorized request to `/warehouse/companies/:id` returns `404 Not Found` (never `403`). |
| **UI scope** | Tenant workers never see the tenant administration navigation; only System Admin sees the Companies view. |
| **Implementation** | Enforced by `requireRole(['admin'])` middleware in `backend/app/authorization/`. |

---

## API Requirements

| Method | Endpoint | Auth | Role | Purpose |
|---|---|---|---|---|
| `GET` | `/warehouse/companies` | Yes | `admin` | List all registered companies |
| `GET` | `/warehouse/companies/:id` | Yes | `admin` | Fetch single company record by ID |
| `POST` | `/warehouse/companies` | Yes | `admin` | Register a new tenant company |
| `PUT` | `/warehouse/companies/:id` | Yes | `admin` | Update company profile details |
| `DELETE` | `/warehouse/companies/:id` | Yes | `admin` | Delete or deactivate company |

**Create Request Body (`POST /warehouse/companies`):**
```json
{
  "company_name": "Apex Logistics Corp",
  "company_address": "123 Supply Chain Blvd, Suite 400, Dallas, TX 75201",
  "company_phone_number": "+1-214-555-0199"
}
```

**Success Response (`201 Created` / `200 OK`):**
```json
{
  "id": 1,
  "company_name": "Apex Logistics Corp",
  "company_address": "123 Supply Chain Blvd, Suite 400, Dallas, TX 75201",
  "company_phone_number": "+1-214-555-0199",
  "status": "ACTIVE",
  "created_at": "2026-11-10T12:00:00.000Z",
  "updated_at": "2026-11-10T12:00:00.000Z"
}
```

**Error Response (`400 Bad Request` / `404 Not Found` / `409 Conflict`):**
```json
{
  "message": "Company name is required (2–100 characters)"
}
```

---

## Screen Requirements

### [View: Companies Administration] — route name `companies`
- **Heading:** **Registered Companies**
- **Primary action:** **+ Register Company** button using class `oc-cta` opening `<v-dialog>` with:
  - `company_name` text input
  - `company_address` text input
  - `company_phone_number` text input
  - **Save Company** (`oc-cta`) and **Cancel** buttons
- **Data Table:** `<v-data-table>` displaying columns:
  - ID, Company Name, Address, Phone Number, Status (`<v-chip color="green">Active</v-chip>`)
  - Row Actions:
    - **Edit** icon (`aria-label="Edit company"`) opens prefilled edit dialog
    - **Delete** icon (`aria-label="Delete company"`) opens confirmation dialog
- **Empty state:** **"No companies registered yet. Register your first tenant company."**
- **Loading state:** Skeleton progress loader during API fetch
- **Error state:** `<v-alert type="error">` displaying API error message

---

## Key Entities

- **Company**: Tenant entity with unique identification, legal company name, headquarters address, primary contact phone number, and account status (`ACTIVE` | `INACTIVE`).

---

## Data Model Requirements

### `companies`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique identifier for the company |
| `company_name` | VARCHAR(100) | NOT NULL, UNIQUE | Legal name of company (2–100 chars) |
| `company_address` | VARCHAR(255) | NOT NULL | Headquarters street, city, state, zip (5–255 chars) |
| `company_phone_number` | VARCHAR(20) | NOT NULL | Formatted telephone number (e.g., `+1-555-123-4567`) |
| `status` | VARCHAR(20) | NOT NULL, DEFAULT `'ACTIVE'` | Enum: `'ACTIVE'`, `'INACTIVE'` |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp when record was created |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp when record was last updated |

---

## Acceptance Criteria (Gherkin)

### US-1.1 — Add company

#### Scenario: Successfully add a new company
* **Given** a system administrator is on the "Add Company" screen
* **When** the administrator enters:
  | Field | Value |
  | `company_name` | "Apex Logistics Corp" |
  | `company_address` | "123 Supply Chain Blvd, Suite 400, Dallas, TX 75201" |
  | `company_phone_number` | "+1-214-555-0199" |
* **And** the administrator submits the form
* **Then** the system creates the company record with status `"ACTIVE"`
* **And** displays a success message "Company Apex Logistics Corp created successfully"
* **And** the new company appears in the company listing

#### Scenario: Attempt to add company with missing required fields
* **Given** a system administrator is on the "Add Company" screen
* **When** the administrator submits the form with empty `company_name`
* **Then** the system rejects the submission without saving to the database
* **And** displays an inline validation error "Company name is required (2–100 characters)"

#### Scenario: Attempt to add company with duplicate name
* **Given** a company already exists with `company_name` "Apex Logistics Corp"
* **When** a system administrator attempts to create a company with `company_name` "apex logistics corp"
* **Then** the system rejects the submission
* **And** displays a validation error "A company with this name already exists"

#### Scenario: Attempt to add company with invalid phone number format
* **Given** a system administrator enters an invalid phone number "123-ABC"
* **When** the administrator submits the form
* **Then** the system rejects the submission
* **And** displays an inline validation error "Phone number must be a valid format between 10 and 20 digits"

---

### US-1.2 — Edit company

#### Scenario: Successfully update company details
* **Given** an existing company "Apex Logistics Corp" with ID `1`
* **When** the system administrator changes `company_phone_number` to "+1-214-555-9988"
* **And** submits the updated form
* **Then** the system updates the record in the database
* **And** updates `updated_at` timestamp
* **And** displays a confirmation message "Company updated successfully"

#### Scenario: Attempt to update company name to an existing duplicate name
* **Given** company A named "Apex Logistics Corp" and company B named "Summit Freight"
* **When** the administrator edits company B and changes the name to "Apex Logistics Corp"
* **And** submits the form
* **Then** the system rejects the update
* **And** displays a validation error "A company with this name already exists"
* **And** company B's name remains unchanged in the database

---

### US-1.3 — Delete / Deactivate company

#### Scenario: Successfully delete a company with no dependent records
* **Given** a company "Old Warehouse LLC" exists and has zero associated warehouses, items, or orders
* **When** the system administrator selects "Delete" on "Old Warehouse LLC"
* **And** confirms the action in the confirmation prompt
* **Then** the system removes the company record from the database
* **And** displays a confirmation message "Company Old Warehouse LLC deleted successfully"

#### Scenario: Deletion blocked when dependent warehouses exist
* **Given** a company "Apex Logistics Corp" has 2 active warehouses associated with it
* **When** the system administrator attempts to delete "Apex Logistics Corp"
* **Then** the system refuses deletion
* **And** displays an error message "Cannot delete company: 2 active warehouses are currently linked. Please reassign or delete dependent warehouses first"
* **And** the company record remains intact

---

## Definition of Done

* [ ] Backend and frontend implemented per this spec (**FR-001** – **FR-005** satisfied)
* [ ] Database table `companies` created with unique constraints and status check
* [ ] All API endpoints (`/warehouse/companies`) secured with `admin` authorization guard
* [ ] All Gherkin acceptance criteria scenarios verified via automated tests
* [ ] Error responses return standard `{ "message": "..." }` and 404 on out-of-scope access

---

## Out of Scope

* Tenant billing, subscription tiers, and payment processing
* Warehouse creation and management ([Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md))
* Company worker and user provisioning ([Feature 5 — Maintain Workers](feature-5-maintain-workers.md))
