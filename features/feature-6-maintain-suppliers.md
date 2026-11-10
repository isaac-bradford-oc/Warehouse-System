# Feature 6: Maintain Suppliers

**Feature ID:** 6  
**Branch pattern:** `feature/6-maintain-suppliers`  
**Status:** Ready  
**Created:** 2026-09-19  
**Last Updated:** 2026-11-10  
**Input:** Manage supplier vendor master records, payment terms, and lead times.  
**Depends on:** [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md)  

---

## User Stories

### US-6.1: Add Supplier
**As an** office manager  
**I want to** register a new commercial vendor/supplier with purchasing terms and lead times  
**So that** the company can place purchase orders and receive inventory from that vendor  

* **Priority:** P1  
* **Independent test:** Submit a valid supplier entry form; verify vendor details, lead times, and terms are saved and accessible in purchasing menus.  
* **Acceptance scenarios:** see [US-6.1](#us-61--add-supplier) under Acceptance Criteria.

### US-6.2: Edit Supplier Details
**As an** office manager  
**I want to** update supplier contact info, lead time, minimum order amount, and payment terms  
**So that** automated replenishment and purchase orders reflect updated contract terms  

* **Priority:** P1  
* **Independent test:** Update a supplier's minimum order amount and shipping lead time; verify the new parameters are saved and used in replenishment calculations.  
* **Acceptance scenarios:** see [US-6.2](#us-62--edit-supplier-details) under Acceptance Criteria.

### US-6.3: Delete / Deactivate Supplier
**As an** office manager  
**I want to** deactivate or remove a supplier vendor profile  
**So that** discontinued vendors cannot be selected on new purchase orders  

* **Priority:** P1  
* **Independent test:** Attempt deletion of a supplier with open purchase orders (blocked); deactivate a vendor with historical orders (succeeds).  
* **Acceptance scenarios:** see [US-6.3](#us-63--delete--deactivate-supplier) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers to create, view, update, and deactivate supplier profiles.
- **FR-002**: Every supplier MUST be associated with a valid `company_id`.
- **FR-003**: System MUST enforce that `supplier_name` is unique within each tenant company.
- **FR-004**: System MUST validate that `lead_time_days` >= 0 and `payment_terms_days` >= 0.
- **FR-005**: System MUST validate that `min_order_amount` is a non-negative currency value (`DECIMAL(10,2)` >= 0.00).
- **FR-006**: System MUST prevent hard deletion of any supplier associated with existing items or purchase orders; system MUST allow deactivation (`status = 'INACTIVE'`).
- **FR-007**: System MUST provide immediate feedback with clear confirmation or inline validation error messages.

---

## Key Entities

- **Supplier**: A commercial vendor selling products to the company, defined by vendor code, legal name, address, contact email/phone, our customer account number with them, standard shipping lead time in days, payment terms in days (e.g., Net 30), minimum order spend threshold, and vendor status (`ACTIVE` | `INACTIVE`).

---

## Data Model Requirements

### `suppliers`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique supplier ID |
| `company_id` | INT / BIGINT | FK -> `companies(id)`, NOT NULL | Owning tenant company |
| `supplier_code` | VARCHAR(30) | NOT NULL | Vendor code (e.g., `SUPP-BEV-01`) |
| `supplier_name` | VARCHAR(100) | NOT NULL | Commercial vendor name (2–100 chars) |
| `supplier_address` | VARCHAR(255) | NOT NULL | Vendor billing/remittance address |
| `contact_email` | VARCHAR(100) | NOT NULL | Orders/dispatch email address |
| `contact_phone` | VARCHAR(20) | NOT NULL | Vendor contact telephone number |
| `our_customer_number` | VARCHAR(50) | NOT NULL | Our account ID with this supplier |
| `lead_time_days` | INT | NOT NULL, DEFAULT 3, CHECK (lead_time_days >= 0) | Expected delivery transit time in days |
| `payment_terms_days` | INT | NOT NULL, DEFAULT 30, CHECK (payment_terms_days >= 0) | Payment terms in days (e.g., 30 for Net 30) |
| `min_order_amount` | DECIMAL(10,2) | NOT NULL, DEFAULT 0.00, CHECK (min_order_amount >= 0) | Minimum order spend threshold |
| `status` | VARCHAR(20) | NOT NULL, DEFAULT `'ACTIVE'` | Enum: `'ACTIVE'`, `'INACTIVE'` |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp created |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp updated |

* **Unique Index:** `(company_id, supplier_code)`  
* **Unique Index:** `(company_id, supplier_name)`

---

## Acceptance Criteria (Gherkin)

### US-6.1 — Add supplier

#### Scenario: Successfully add a new supplier
* **Given** an office manager for company ID `1` is on the "Add Supplier" page
* **When** the manager enters:
  | Field | Value |
  | `supplier_code` | "SUPP-ACME" |
  | `supplier_name` | "Acme Beverage Supply" |
  | `supplier_address` | "88 Industrial Way, Atlanta, GA 30301" |
  | `contact_email` | "orders@acmebev.com" |
  | `contact_phone` | "+1-404-555-0188" |
  | `our_customer_number` | "CUST-9921" |
  | `lead_time_days` | 5 |
  | `payment_terms_days` | 30 |
  | `min_order_amount` | 250.00 |
* **And** submits the form
* **Then** the system saves the supplier with status `"ACTIVE"`
* **And** displays confirmation "Supplier Acme Beverage Supply (SUPP-ACME) created successfully"

#### Scenario: Attempt to add supplier with duplicate name
* **Given** a supplier named "Acme Beverage Supply" already exists for company ID `1`
* **When** the office manager attempts to create another supplier with `supplier_name` "Acme Beverage Supply"
* **Then** the system rejects the submission
* **And** displays an error message "A supplier with this name already exists in your company"

#### Scenario: Attempt to add supplier with negative lead time
* **Given** the office manager enters `lead_time_days` = -2
* **When** the manager submits the form
* **Then** the system rejects the submission
* **And** displays an inline validation error "Lead time days must be greater than or equal to 0"

---

### US-6.2 — Edit supplier details

#### Scenario: Successfully update supplier lead time and minimum order
* **Given** an existing supplier "Acme Beverage Supply"
* **When** the office manager updates `lead_time_days` to 7 and `min_order_amount` to 500.00
* **And** submits the edit form
* **Then** the system updates the record in the database
* **And** displays confirmation "Supplier Acme Beverage Supply updated successfully"

#### Scenario: Attempt to update supplier with negative minimum order amount
* **Given** an existing supplier "Acme Beverage Supply"
* **When** the office manager updates `min_order_amount` to -100.00
* **And** submits the edit form
* **Then** the system rejects the update
* **And** displays an inline validation error "Minimum order amount must be greater than or equal to $0.00"
* **And** the minimum order amount remains unchanged in the database

---

### US-6.3 — Delete / Deactivate supplier

#### Scenario: Successfully deactivate a supplier
* **Given** supplier "Acme Beverage Supply" has historical purchase orders
* **When** the office manager selects "Deactivate Supplier" and confirms
* **Then** the system sets status to `"INACTIVE"`
* **And** displays confirmation "Supplier Acme Beverage Supply has been deactivated"
* **And** the supplier can no longer be selected on new purchase orders

#### Scenario: Block deletion when supplier has open purchase orders
* **Given** supplier "Acme Beverage Supply" has 1 purchase order in status `"SUBMITTED"`
* **When** the office manager attempts to delete the supplier
* **Then** the system rejects the deletion
* **And** displays an error message "Cannot delete supplier: 1 open purchase order exists. Cancel or receive all open orders first"
