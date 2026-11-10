# Feature 8: Maintain Customers

**Feature ID:** 8  
**Branch pattern:** `feature/8-maintain-customers`  
**Status:** Ready  
**Created:** 2026-09-19  
**Last Updated:** 2026-11-10  
**Input:** Manage commercial customer client records, delivery addresses, and credit terms.  
**Depends on:** [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md)  

---

## User Stories

### US-8.1: Add Customer
**As an** office manager  
**I want to** register a new commercial customer account with delivery addresses and billing terms  
**So that** sales orders can be created, billed, and dispatched to their delivery destinations  

* **Priority:** P1  
* **Independent test:** Submit a valid customer registration form; verify account details, delivery address, and terms persist in the database and appear in customer search.  
* **Acceptance scenarios:** see [US-8.1](#us-81--add-customer) under Acceptance Criteria.

### US-8.2: Edit Customer Details
**As an** office manager  
**I want to** update customer delivery address, contact information, credit limits, or payment terms  
**So that** deliveries are routed to current addresses and invoicing follows agreed terms  

* **Priority:** P1  
* **Independent test:** Update a customer's delivery shipping address and credit limit; verify changes persist and update route assignment lists.  
* **Acceptance scenarios:** see [US-8.2](#us-82--edit-customer-details) under Acceptance Criteria.

### US-8.3: Delete / Deactivate Customer
**As an** office manager  
**I want to** deactivate or remove a customer account that is no longer active  
**So that** retired accounts cannot have new sales orders placed against them  

* **Priority:** P1  
* **Independent test:** Attempt to delete a customer with open orders (blocked); deactivate an inactive customer (succeeds).  
* **Acceptance scenarios:** see [US-8.3](#us-83--delete--deactivate-customer) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers to create, view, update, and deactivate customer accounts.
- **FR-002**: Every customer MUST link to a valid `company_id`.
- **FR-003**: System MUST enforce that `customer_account_number` is unique per company.
- **FR-004**: System MUST enforce that `customer_name` is unique per company (case-insensitive).
- **FR-005**: System MUST validate that `payment_terms_days` >= 0 and `credit_limit` >= 0.00.
- **FR-006**: System MUST prevent hard deletion of any customer associated with existing customer orders; system MUST support deactivation (`status = 'INACTIVE'`).
- **FR-007**: System MUST provide immediate user feedback with specific validation error messages.

---

## Key Entities

- **Customer**: A commercial buyer purchasing goods from the company, defined by account number, client name, billing address, shipping/delivery address, contact email/phone, credit limit, payment terms in days (e.g., Net 30), and account status (`ACTIVE` | `INACTIVE`).

---

## Data Model Requirements

### `customers`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique internal customer ID |
| `company_id` | INT / BIGINT | FK -> `companies(id)`, NOT NULL | Owning tenant company |
| `customer_account_number` | VARCHAR(30) | NOT NULL | Unique customer account number |
| `customer_name` | VARCHAR(100) | NOT NULL | Commercial customer business name |
| `billing_address` | VARCHAR(255) | NOT NULL | Invoice mailing address |
| `shipping_address` | VARCHAR(255) | NOT NULL | Physical delivery dock/store address |
| `contact_email` | VARCHAR(100) | NOT NULL | Billing/dispatch notification email |
| `contact_phone` | VARCHAR(20) | NOT NULL | Customer telephone contact |
| `payment_terms_days` | INT | NOT NULL, DEFAULT 30, CHECK (payment_terms_days >= 0) | Invoice payment terms in days |
| `credit_limit` | DECIMAL(12,2) | NOT NULL, DEFAULT 5000.00, CHECK (credit_limit >= 0) | Maximum allowable outstanding balance |
| `status` | VARCHAR(20) | NOT NULL, DEFAULT `'ACTIVE'` | Enum: `'ACTIVE'`, `'INACTIVE'` |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp created |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp updated |

* **Unique Index:** `(company_id, customer_account_number)`  
* **Unique Index:** `(company_id, customer_name)`

---

## Acceptance Criteria (Gherkin)

### US-8.1 — Add customer

#### Scenario: Successfully add a new customer
* **Given** an office manager for company ID `1` is on the "Add Customer" page
* **When** the manager enters:
  | Field | Value |
  | `customer_account_number` | "CUST-TEX-101" |
  | `customer_name` | "Lone Star Markets" |
  | `billing_address` | "100 Main St, Suite 200, Austin, TX 78701" |
  | `shipping_address` | "550 Warehouse Rd, Dock B, Austin, TX 78702" |
  | `contact_email` | "receiving@lonestarmarkets.com" |
  | `contact_phone` | "+1-512-555-0133" |
  | `payment_terms_days` | 30 |
  | `credit_limit` | 10000.00 |
* **And** submits the form
* **Then** the system creates the customer record with status `"ACTIVE"`
* **And** displays confirmation "Customer Lone Star Markets (CUST-TEX-101) registered successfully"

#### Scenario: Attempt to add customer with duplicate account number
* **Given** a customer with account number "CUST-TEX-101" already exists for company ID `1`
* **When** the office manager attempts to create another customer with account number "CUST-TEX-101"
* **Then** the system rejects the submission
* **And** displays an error message "Customer account number CUST-TEX-101 already exists in your company"

#### Scenario: Attempt to add customer with negative credit limit
* **Given** the office manager enters `credit_limit` = -500.00
* **When** the manager submits the form
* **Then** the system rejects the submission
* **And** displays an inline validation error "Credit limit must be greater than or equal to $0.00"

---

### US-8.2 — Edit customer details

#### Scenario: Successfully update customer shipping address
* **Given** an existing customer "Lone Star Markets"
* **When** the office manager updates `shipping_address` to "700 Distribution Blvd, Dock 4, Austin, TX 78704"
* **And** submits the edit form
* **Then** the system saves the updated shipping address
* **And** displays confirmation "Customer Lone Star Markets updated successfully"

#### Scenario: Attempt to update customer with negative payment terms
* **Given** an existing customer "Lone Star Markets"
* **When** the office manager updates `payment_terms_days` to -15
* **And** submits the edit form
* **Then** the system rejects the update
* **And** displays an inline validation error "Payment terms days must be greater than or equal to 0"
* **And** customer payment terms remain unchanged

---

### US-8.3 — Delete / Deactivate customer

#### Scenario: Successfully deactivate an existing customer
* **Given** customer "Lone Star Markets" has past completed sales orders
* **When** the office manager selects "Deactivate Customer" and confirms
* **Then** the system sets status to `"INACTIVE"`
* **And** displays confirmation "Customer Lone Star Markets has been deactivated"
* **And** blocks creating new sales orders for this customer

#### Scenario: Block deletion when customer has open sales orders
* **Given** customer "Lone Star Markets" has an active order in status `"PENDING"`
* **When** the office manager attempts to delete the customer record
* **Then** the system rejects the deletion
* **And** displays an error message "Cannot delete customer: active customer order exists. Please complete or cancel open orders first"
