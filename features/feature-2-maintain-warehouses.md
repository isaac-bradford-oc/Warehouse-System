# Feature 2: Maintain Warehouses

**Feature ID:** 2  
**Branch pattern:** `feature/2-maintain-warehouses`  
**Status:** Ready  
**Created:** 2026-09-19  
**Last Updated:** 2026-11-10  
**Input:** Keep an accurate record of physical warehouse locations belonging to each tenant company.  
**Depends on:** [Feature 1 — Maintain Companies](feature-1-maintain-companies.md)  

---

## User Stories

### US-2.1: Add Warehouse
**As a** company admin  
**I want to** add a physical warehouse to my company profile  
**So that** the system can track inventory, workers, and orders designated to that physical facility  

* **Priority:** P1  
* **Independent test:** Submit a valid warehouse registration form associated with an existing company and verify the warehouse appears in the company's warehouse list.  
* **Acceptance scenarios:** see [US-2.1](#us-21--add-warehouse) under Acceptance Criteria.

### US-2.2: Edit Warehouse
**As a** company admin  
**I want to** edit warehouse details (facility name, address, operational status)  
**So that** physical warehouse facility information remains up-to-date  

* **Priority:** P1  
* **Independent test:** Modify an existing warehouse's address and status, save changes, and confirm the updated data is stored and displayed.  
* **Acceptance scenarios:** see [US-2.2](#us-22--edit-warehouse) under Acceptance Criteria.

### US-2.3: Delete / Deactivate Warehouse
**As a** company admin  
**I want to** delete or deactivate an empty warehouse facility  
**So that** decommissioned locations are not accidentally selected for receiving, stocking, or shipping  

* **Priority:** P1  
* **Independent test:** Attempt deletion of a warehouse with no active stock or orders; verify deletion succeeds. Attempt deletion of a warehouse with active inventory and verify deletion is prevented.  
* **Acceptance scenarios:** see [US-2.3](#us-23--delete--deactivate-warehouse) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow company administrators to create, view, update, and deactivate/delete warehouse facilities for their tenant company.
- **FR-002**: Every warehouse MUST be linked to an existing, active `company_id`.
- **FR-003**: System MUST enforce that `warehouse_code` is unique within each tenant company.
- **FR-004**: System MUST validate all warehouse input fields (code format, name length, valid address) before persisting.
- **FR-005**: System MUST prevent deletion of any warehouse containing non-zero inventory, active worker assignments, or incomplete orders. Deactivation (`status = 'INACTIVE'`) must be supported as a safe alternative.
- **FR-006**: System MUST return descriptive success notifications and clear inline validation error messages.

---

## Data Ownership & Isolation

Warehouses belong to a specific tenant company (`company_id`).

| Rule | Requirement |
|---|---|
| **Read scope** | `GET /warehouse/warehouses` returns only warehouses where `company_id = req.user.companyId`. |
| **Write scope** | Updates/deletions apply only if the warehouse belongs to `req.user.companyId`. |
| **Create scope** | New warehouse rows automatically inherit `company_id = req.user.companyId`. |
| **Cross-user access** | Accessing a warehouse belonging to another tenant returns `404 Not Found` (never `403`). |
| **UI scope** | Only Company Admins and Office Managers belonging to the tenant see the facility in dropdowns. |
| **Implementation** | Centralized query helper `getAccessibleWarehouseOrNull(req, warehouseId)`. |

---

## API Requirements

| Method | Endpoint | Auth | Role | Purpose |
|---|---|---|---|---|
| `GET` | `/warehouse/warehouses` | Yes | `company admin`, `office manager` | List warehouses for authenticated company |
| `GET` | `/warehouse/warehouses/:id` | Yes | `company admin`, `office manager` | Get single warehouse details |
| `POST` | `/warehouse/warehouses` | Yes | `company admin` | Create a new warehouse facility |
| `PUT` | `/warehouse/warehouses/:id` | Yes | `company admin` | Update warehouse details |
| `DELETE` | `/warehouse/warehouses/:id` | Yes | `company admin` | Delete empty warehouse or deactivate |

**Create Request Body (`POST /warehouse/warehouses`):**
```json
{
  "warehouse_code": "DAL-01",
  "warehouse_name": "Dallas North Hub",
  "warehouse_address": "450 Industrial Parkway, Dallas, TX 75001"
}
```

**Success Response (`201 Created` / `200 OK`):**
```json
{
  "id": 2,
  "company_id": 1,
  "warehouse_code": "DAL-01",
  "warehouse_name": "Dallas North Hub",
  "warehouse_address": "450 Industrial Parkway, Dallas, TX 75001",
  "status": "ACTIVE",
  "created_at": "2026-11-10T12:00:00.000Z",
  "updated_at": "2026-11-10T12:00:00.000Z"
}
```

**Error Response (`400 Bad Request` / `404 Not Found` / `409 Conflict`):**
```json
{
  "message": "Warehouse code DAL-01 is already in use for your company"
}
```

---

## Screen Requirements

### [View: Warehouses Management] — route name `warehouses`
- **Heading:** **Company Warehouses**
- **Primary action:** **+ Add Warehouse** (`oc-cta`) opens `<v-dialog>` with:
  - `warehouse_code` text input (e.g. `DAL-01`)
  - `warehouse_name` text input
  - `warehouse_address` text input
  - **Save Warehouse** (`oc-cta`) and **Cancel** buttons
- **Data Table:** `<v-data-table>` displaying columns:
  - Code, Name, Address, Status (`<v-chip color="green">Active</v-chip>`)
  - Row Actions:
    - **Edit** icon (`aria-label="Edit warehouse"`)
    - **Delete** icon (`aria-label="Delete warehouse"`)
- **Empty state:** **"No warehouses configured yet. Add your first warehouse facility."**
- **Loading state:** Skeleton loader while fetching records
- **Error state:** `<v-alert type="error">` displaying API error message

---

## Key Entities

- **Warehouse**: Physical distribution center or facility tied to a parent company, having a unique code within the company, facility name, address, and operational status (`ACTIVE` | `INACTIVE`).

---

## Data Model Requirements

### `warehouses`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique warehouse identifier |
| `company_id` | INT / BIGINT | FK -> `companies(id)`, NOT NULL | Reference to owning company |
| `warehouse_code` | VARCHAR(20) | NOT NULL | Alphanumeric identifier unique per company (e.g., `WH-DAL-01`) |
| `warehouse_name` | VARCHAR(100) | NOT NULL | Human-readable facility name (e.g., `Dallas Central Distribution`) |
| `warehouse_address` | VARCHAR(255) | NOT NULL | Street address, city, state, postal code |
| `status` | VARCHAR(20) | NOT NULL, DEFAULT `'ACTIVE'` | Enum: `'ACTIVE'`, `'INACTIVE'` |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Record creation timestamp |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Record last updated timestamp |

* **Unique Index:** `(company_id, warehouse_code)`  
* **Unique Index:** `(company_id, warehouse_name)`

---

## Acceptance Criteria (Gherkin)

### US-2.1 — Add warehouse

#### Scenario: Successfully create a warehouse
* **Given** an authenticated company admin for company ID `1` ("Apex Logistics Corp")
* **When** the admin submits the "Add Warehouse" form with:
  | Field | Value |
  | `warehouse_code` | "DAL-01" |
  | `warehouse_name` | "Dallas North Hub" |
  | `warehouse_address` | "450 Industrial Parkway, Dallas, TX 75001" |
* **Then** the system creates the warehouse record linked to `company_id` = 1 with status `"ACTIVE"`
* **And** displays confirmation "Warehouse Dallas North Hub (DAL-01) created successfully"

#### Scenario: Attempt to create warehouse with duplicate code
* **Given** a warehouse with code "DAL-01" already exists for company ID `1`
* **When** the company admin submits a new warehouse with code "DAL-01"
* **Then** the system rejects the creation
* **And** displays an error message "Warehouse code DAL-01 is already in use for your company"

#### Scenario: Attempt to create warehouse with missing required fields
* **Given** the company admin is on the "Add Warehouse" page
* **When** the admin submits the form with empty `warehouse_name` and empty `warehouse_address`
* **Then** the system rejects the submission
* **And** displays inline error messages:
  * "Warehouse name is required (2–100 characters)"
  * "Warehouse address is required (5–255 characters)"

---

### US-2.2 — Edit warehouse

#### Scenario: Successfully update warehouse information
* **Given** an existing warehouse "Dallas North Hub" with code "DAL-01"
* **When** the company admin changes `warehouse_address` to "500 Logistics Way, Dallas, TX 75001"
* **And** submits the edit form
* **Then** the system saves the updated address
* **And** updates `updated_at` to the current timestamp
* **And** displays confirmation "Warehouse details updated successfully"

#### Scenario: Attempt to change warehouse code to one that conflicts
* **Given** warehouse "DAL-01" and warehouse "DAL-02" exist under the same company
* **When** the admin edits warehouse "DAL-02" and changes code to "DAL-01"
* **And** submits the form
* **Then** the system rejects the change
* **And** displays an error message "Warehouse code DAL-01 is already in use"
* **And** the record for "DAL-02" remains unchanged

---

### US-2.3 — Delete / Deactivate warehouse

#### Scenario: Successfully delete an empty warehouse
* **Given** warehouse "DAL-01" has zero inventory items, zero assigned workers, and zero historical orders
* **When** the company admin clicks "Delete Warehouse" and confirms the action
* **Then** the system removes the warehouse record from the database
* **And** displays confirmation "Warehouse DAL-01 deleted successfully"

#### Scenario: Prevent deletion when warehouse has active inventory or orders
* **Given** warehouse "DAL-01" contains 450 units of inventory on hand
* **When** the company admin attempts to delete warehouse "DAL-01"
* **Then** the system rejects the deletion
* **And** displays an error message "Cannot delete warehouse: active inventory exists. Please transfer stock or deactivate the facility instead"
* **And** the warehouse record remains active

---

## Definition of Done

* [ ] Backend and frontend implemented per this spec (**FR-001** – **FR-006** satisfied)
* [ ] Database table `warehouses` created with compound unique index `(company_id, warehouse_code)`
* [ ] API endpoints `/warehouse/warehouses` enforce tenant scoping and role guards
* [ ] Gherkin acceptance criteria verified by automated tests
* [ ] Out-of-tenant access attempts return `404 Not Found`

---

## Out of Scope

* Aisles, racks, slots, and bin layout design ([Feature 4 — Maintain Inventory Locations](feature-4-maintain-inventory.md))
* Warehouse worker assignment ([Feature 5 — Maintain Workers](feature-5-maintain-workers.md))
* Physical GPS geolocation and geofencing
