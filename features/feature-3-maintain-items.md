# Feature 3: Maintain Items

**Feature ID:** 3  
**Branch pattern:** `feature/3-maintain-items`  
**Status:** Ready  
**Created:** 2026-09-19  
**Last Updated:** 2026-11-10  
**Input:** Maintain master product item catalog data including identifiers, pricing, case quantities, and min/max replenishment parameters.  
**Depends on:** [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 6 — Maintain Suppliers](feature-6-maintain-suppliers.md)  

---

## User Stories

### US-3.1: Add Item to Product Catalog
**As an** office manager  
**I want to** register a new product item in the company's master catalog  
**So that** the item is available for purchasing, slotting, and customer order fulfillment  

* **Priority:** P1  
* **Independent test:** Submit a valid item entry form; verify the new SKU exists in the database and catalog list with correct attributes.  
* **Acceptance scenarios:** see [US-3.1](#us-31--add-item-to-product-catalog) under Acceptance Criteria.

### US-3.2: Edit Item Details
**As an** office manager  
**I want to** update item attributes (description, case pack, cost, selling price, min/max thresholds, primary supplier)  
**So that** inventory planning, costing, and order pricing reflect current commercial terms  

* **Priority:** P1  
* **Independent test:** Edit an item's unit price and min/max thresholds; confirm database records updated values without altering inventory balances.  
* **Acceptance scenarios:** see [US-3.2](#us-32--edit-item-details) under Acceptance Criteria.

### US-3.3: Delete / Deactivate Item
**As an** office manager  
**I want to** remove or mark an item as inactive when it is discontinued  
**So that** obsolete products cannot be reordered from suppliers or placed on new customer orders  

* **Priority:** P1  
* **Independent test:** Attempt to delete an item with positive stock on hand (should be blocked); attempt to deactivate an item with zero stock (should succeed).  
* **Acceptance scenarios:** see [US-3.3](#us-33--delete--deactivate-item) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers to create, view, update, and deactivate product items within their company.
- **FR-002**: System MUST enforce that `sku` is unique per company (alphanumeric, 2–50 characters).
- **FR-003**: System MUST enforce that `item_upc` and `case_upc` are valid 12- or 14-digit numeric barcode strings and unique across the catalog.
- **FR-004**: System MUST validate that `case_quantity` is a positive integer (>= 1).
- **FR-005**: System MUST validate that `case_cost` and `unit_price` are valid positive currency amounts (`DECIMAL(10,2)` >= 0.00).
- **FR-006**: System MUST validate that `min_quantity` >= 0 and `max_quantity` >= `min_quantity`.
- **FR-007**: System MUST link every item to a valid `supplier_id` belonging to the same company.
- **FR-008**: System MUST prevent deletion of items that have active inventory on hand or historical order line records; system MUST allow deactivation (`status = 'DISCONTINUED'`) in those cases.

---

## Key Entities

- **Item**: Master catalog entity defining SKU, barcode UPCs, description, supplier reference, case pack size, financial costs/pricing, replenishment min/max thresholds, and catalog status (`ACTIVE` | `DISCONTINUED`).

---

## Data Model Requirements

### `items`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique internal item ID |
| `company_id` | INT / BIGINT | FK -> `companies(id)`, NOT NULL | Owning tenant company |
| `supplier_id` | INT / BIGINT | FK -> `suppliers(id)`, NOT NULL | Primary supplier for reordering |
| `sku` | VARCHAR(50) | NOT NULL | Stock Keeping Unit code (unique per company) |
| `item_upc` | VARCHAR(14) | NOT NULL, UNIQUE | 12-digit UPC-A or 14-digit GTIN unit barcode |
| `case_upc` | VARCHAR(14) | NOT NULL, UNIQUE | Barcode on master case package |
| `description` | VARCHAR(255) | NOT NULL | Detailed item description |
| `case_quantity` | INT | NOT NULL, CHECK (case_quantity > 0) | Number of units per case |
| `case_cost` | DECIMAL(10,2) | NOT NULL, CHECK (case_cost >= 0) | Purchase cost per case from supplier |
| `unit_price` | DECIMAL(10,2) | NOT NULL, CHECK (unit_price >= 0) | Selling price per individual unit to customers |
| `min_quantity` | INT | NOT NULL, CHECK (min_quantity >= 0) | Minimum stock threshold before reorder |
| `max_quantity` | INT | NOT NULL | Target maximum stock level (>= min_quantity) |
| `status` | VARCHAR(20) | NOT NULL, DEFAULT `'ACTIVE'` | Enum: `'ACTIVE'`, `'DISCONTINUED'` |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp created |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp updated |

* **Unique Index:** `(company_id, sku)`  
* **Unique Index:** `item_upc`  
* **Unique Index:** `case_upc`  
* **Check Constraint:** `max_quantity >= min_quantity`

---

## Acceptance Criteria (Gherkin)

### US-3.1 — Add item to product catalog

#### Scenario: Successfully add a valid product item
* **Given** an authenticated office manager for company ID `1`
* **And** a valid supplier "Acme Beverage Supply" exists with ID `10`
* **When** the office manager submits the "Add Item" form with:
  | Field | Value |
  | `sku` | "DRK-COLA-01" |
  | `item_upc` | "012345678905" |
  | `case_upc` | "00012345678905" |
  | `supplier_id` | 10 |
  | `description` | "Sparkling Cola 12oz Can - 24pk" |
  | `case_quantity` | 24 |
  | `case_cost` | 12.50 |
  | `unit_price` | 1.25 |
  | `min_quantity` | 100 |
  | `max_quantity` | 500 |
* **Then** the system creates the item record with status `"ACTIVE"`
* **And** displays confirmation "Item DRK-COLA-01 added successfully"

#### Scenario: Attempt to add item with duplicate SKU
* **Given** an item with SKU "DRK-COLA-01" already exists for company ID `1`
* **When** the office manager submits another item with SKU "DRK-COLA-01"
* **Then** the system rejects the submission
* **And** displays an error message "SKU DRK-COLA-01 already exists in the company catalog"

#### Scenario: Attempt to add item with min_quantity greater than max_quantity
* **Given** the office manager enters `min_quantity` = 500 and `max_quantity` = 200
* **When** the manager submits the form
* **Then** the system rejects the submission
* **And** displays an inline validation error "Maximum quantity (200) cannot be less than minimum quantity (500)"

#### Scenario: Attempt to add item with invalid UPC length
* **Given** the office manager enters `item_upc` = "12345" (5 digits)
* **When** the manager submits the form
* **Then** the system rejects the submission
* **And** displays an inline validation error "Item UPC must be a valid 12- or 14-digit numeric code"

---

### US-3.2 — Edit item details

#### Scenario: Successfully update pricing and thresholds
* **Given** an existing item "DRK-COLA-01" with unit price 1.25
* **When** the office manager changes `unit_price` to 1.49 and `max_quantity` to 600
* **And** submits the edit form
* **Then** the system updates the record in the database
* **And** displays confirmation "Item DRK-COLA-01 updated successfully"

#### Scenario: Attempt to edit item with negative cost
* **Given** an existing item "DRK-COLA-01"
* **When** the office manager enters `case_cost` = -5.00
* **And** submits the form
* **Then** the system rejects the update
* **And** displays an error message "Case cost must be greater than or equal to $0.00"

---

### US-3.3 — Delete / Deactivate item

#### Scenario: Successfully deactivate an item with no active inventory
* **Given** an item "DISC-SODA-99" has 0 units in stock across all warehouses
* **When** the office manager selects "Deactivate Item"
* **And** confirms the action in the prompt
* **Then** the system sets the item status to `"DISCONTINUED"`
* **And** displays confirmation "Item DISC-SODA-99 has been discontinued"
* **And** the item cannot be added to new purchase or sales orders

#### Scenario: Block deactivation or deletion when item has inventory on hand
* **Given** an item "DRK-COLA-01" has 150 units in stock in warehouse "DAL-01"
* **When** the office manager attempts to delete or discontinue the item
* **Then** the system rejects the request
* **And** displays an error message "Cannot discontinue item: 150 units remain in stock across warehouse facilities. Deplete inventory first."
