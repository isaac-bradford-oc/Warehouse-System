# Feature 7: Maintain Supplier Orders

**Feature ID:** 7  
**Branch pattern:** `feature/7-maintain-supplier-orders`  
**Status:** Ready  
**Created:** 2026-09-19  
**Last Updated:** 2026-11-10  
**Input:** Manage the lifecycle, line items, and electronic transmission of supplier purchase orders.  
**Depends on:** [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md), [Feature 3 — Maintain Items](feature-3-maintain-items.md), [Feature 6 — Maintain Suppliers](feature-6-maintain-suppliers.md)  

---

## User Stories

### US-7.1: Create Supplier Purchase Order (Draft)
**As an** office manager  
**I want to** create a new purchase order draft for a supplier with specified items and case quantities  
**So that** the warehouse can formally request inventory replenishment from vendors  

* **Priority:** P1  
* **Independent test:** Create a purchase order in draft status with multiple line items; verify header and line items persist with calculated totals.  
* **Acceptance scenarios:** see [US-7.1](#us-71--create-supplier-purchase-order-draft) under Acceptance Criteria.

### US-7.2: Edit Supplier Order
**As an** office manager  
**I want to** modify order line items, quantities, or expected delivery date on a draft order  
**So that** order details can be revised prior to official submission to the supplier  

* **Priority:** P1  
* **Independent test:** Change line item quantities on a draft order; verify recalculated order totals persist.  
* **Acceptance scenarios:** see [US-7.2](#us-72--edit-supplier-order) under Acceptance Criteria.

### US-7.3: Submit / Send Order to Supplier
**As an** office manager  
**I want to** submit and electronically dispatch an approved purchase order to the vendor  
**So that** the supplier initiates manufacturing and shipping of the requested items  

* **Priority:** P1  
* **Independent test:** Submit a valid draft order meeting supplier minimum order thresholds; verify status transitions to `"SUBMITTED"` and electronic notification/EDI payload is dispatched.  
* **Acceptance scenarios:** see [US-7.3](#us-73--submit--send-order-to-supplier) under Acceptance Criteria.

### US-7.4: Cancel / Delete Supplier Order
**As an** office manager  
**I want to** cancel an active purchase order or delete an unsubmitted draft  
**So that** erroneous orders are cancelled with the supplier and warehouse expectations are updated  

* **Priority:** P1  
* **Independent test:** Cancel an open submitted order; verify status transitions to `"CANCELLED"` and items are removed from warehouse expected incoming balances.  
* **Acceptance scenarios:** see [US-7.4](#us-74--cancel--delete-supplier-order) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers to create, view, update, submit, and cancel purchase orders.
- **FR-002**: Every order MUST be associated with a valid `company_id`, target `warehouse_id`, `supplier_id`, and authorizer (`worker_id`).
- **FR-003**: Orders MUST follow a strict lifecycle state machine: `DRAFT` -> `SUBMITTED` -> `PARTIALLY_RECEIVED` -> `RECEIVED` -> `CLOSED` (or `CANCELLED`).
- **FR-004**: Orders in status `SUBMITTED`, `PARTIALLY_RECEIVED`, `RECEIVED`, or `CLOSED` MUST NOT allow line item addition or deletion.
- **FR-005**: System MUST validate that an order contains at least one line item with `quantity_ordered > 0` before submission.
- **FR-006**: System MUST enforce that the order total meets or exceeds the supplier's `min_order_amount` before allowing submission.
- **FR-007**: System MUST generate a unique, non-repeating `order_number` (e.g., `PO-202611-0001`).
- **FR-008**: System MUST display unambiguous confirmation and actionable validation errors.

---

## Data Ownership & Isolation

Supplier purchase orders belong to the tenant company and destination warehouse.

| Rule | Requirement |
|---|---|
| **Read scope** | `GET /warehouse/supplier-orders` returns orders where `company_id = req.user.companyId` and `warehouse_id = req.user.warehouseId`. |
| **Write scope** | Editing is permitted only for orders matching `req.user.companyId` and currently in `DRAFT` status. |
| **Create scope** | New orders automatically associate with `req.user.companyId` and authorizer `req.user.id`. |
| **Cross-user access** | Accessing a purchase order belonging to another tenant or warehouse returns `404 Not Found`. |
| **UI scope** | Users only see purchase orders targeted for their authorized facility. |
| **Implementation** | Centralized query helper `getAccessibleSupplierOrderOrNull(req, orderId)`. |

---

## API Requirements

| Method | Endpoint | Auth | Role | Purpose |
|---|---|---|---|---|
| `GET` | `/warehouse/supplier-orders` | Yes | `office manager`, `company admin` | List purchase orders |
| `GET` | `/warehouse/supplier-orders/:id` | Yes | `office manager`, `company admin` | Get order and line items |
| `POST` | `/warehouse/supplier-orders` | Yes | `office manager`, `company admin` | Create draft purchase order |
| `PUT` | `/warehouse/supplier-orders/:id` | Yes | `office manager`, `company admin` | Update line items on draft |
| `POST` | `/warehouse/supplier-orders/:id/submit` | Yes | `office manager`, `company admin` | Submit PO to vendor |
| `POST` | `/warehouse/supplier-orders/:id/cancel` | Yes | `office manager`, `company admin` | Cancel purchase order |

**Create Request Body (`POST /warehouse/supplier-orders`):**
```json
{
  "warehouse_id": 2,
  "supplier_id": 10,
  "expected_date": "2026-11-20",
  "items": [
    {
      "item_id": 101,
      "quantity_ordered": 25,
      "case_cost": 12.50
    }
  ]
}
```

**Success Response (`201 Created` / `200 OK`):**
```json
{
  "id": 15,
  "order_number": "PO-202611-0001",
  "company_id": 1,
  "warehouse_id": 2,
  "supplier_id": 10,
  "authorizer_worker_id": 4,
  "order_date": "2026-11-10",
  "expected_date": "2026-11-20",
  "total_amount": 312.50,
  "status": "DRAFT",
  "items": [
    {
      "id": 88,
      "item_id": 101,
      "quantity_ordered": 25,
      "quantity_received": 0,
      "case_cost": 12.50
    }
  ],
  "created_at": "2026-11-10T12:00:00.000Z"
}
```

**Error Response (`400 Bad Request` / `404 Not Found` / `409 Conflict`):**
```json
{
  "message": "Order total ($125.00) is below supplier minimum order threshold of $250.00"
}
```

---

## Screen Requirements

### [View: Supplier Purchase Orders] — route name `supplier-orders`
- **Heading:** **Supplier Purchase Orders**
- **Warehouse & Status Filters:** `<v-select>` facility selector; filter chips for `DRAFT`, `SUBMITTED`, `PARTIALLY_RECEIVED`, `RECEIVED`, `CANCELLED`
- **Primary action:** **+ New Purchase Order** (`oc-cta`) opens order modal with:
  - Vendor selector (`supplier_id`), destination warehouse, expected delivery date
  - Line items dynamic table: SKU autocomplete, ordered case count, case cost, line total
  - **Save as Draft** and **Submit to Vendor** (`oc-cta`) buttons
- **Data Table:** `<v-data-table>` displaying columns:
  - PO Number, Supplier, Order Date, Expected Date, Total Amount, Status Chip
  - Row Actions:
    - **View Details** icon (`aria-label="View order"`)
    - **Edit** icon (`aria-label="Edit draft order"`, hidden if not DRAFT)
    - **Cancel** icon (`aria-label="Cancel order"`)
- **Empty state:** **"No purchase orders found for this warehouse. Create your first replenishment PO."**
- **Loading state:** Skeleton rows
- **Error state:** `<v-alert type="error">` displaying API error message

---

## Key Entities

- **Supplier Order**: Purchase order header tracking PO number, destination warehouse, supplier, creation date, expected delivery date, authorizer, total cost, and status (`DRAFT` | `SUBMITTED` | `PARTIALLY_RECEIVED` | `RECEIVED` | `CLOSED` | `CANCELLED`).
- **Supplier Order Item**: Individual line item associating an item SKU, ordered case count, case cost, and received quantity count.

---

## Data Model Requirements

### `supplier_orders`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique internal order ID |
| `order_number` | VARCHAR(30) | NOT NULL, UNIQUE | Human-readable PO number |
| `company_id` | INT / BIGINT | FK -> `companies(id)`, NOT NULL | Owning tenant company |
| `warehouse_id` | INT / BIGINT | FK -> `warehouses(id)`, NOT NULL | Receiving destination facility |
| `supplier_id` | INT / BIGINT | FK -> `suppliers(id)`, NOT NULL | Vendor fulfilling order |
| `authorizer_worker_id`| INT / BIGINT | FK -> `workers(id)`, NOT NULL | Office manager approving PO |
| `order_date` | DATE | NOT NULL | Date PO was generated |
| `expected_date` | DATE | NOT NULL | Estimated arrival date |
| `total_amount` | DECIMAL(10,2) | NOT NULL, DEFAULT 0.00 | Sum of line item extended costs |
| `status` | VARCHAR(30) | NOT NULL, DEFAULT `'DRAFT'` | Enum: `'DRAFT'`, `'SUBMITTED'`, `'PARTIALLY_RECEIVED'`, `'RECEIVED'`, `'CLOSED'`, `'CANCELLED'` |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp created |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp updated |

### `supplier_order_items`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique line item ID |
| `order_id` | INT / BIGINT | FK -> `supplier_orders(id)`, NOT NULL | Parent purchase order |
| `item_id` | INT / BIGINT | FK -> `items(id)`, NOT NULL | Ordered catalog item SKU |
| `quantity_ordered` | INT | NOT NULL, CHECK (quantity_ordered > 0) | Quantity in cases ordered |
| `quantity_received`| INT | NOT NULL, DEFAULT 0, CHECK (quantity_received >= 0) | Cumulative cases received at dock |
| `case_cost` | DECIMAL(10,2) | NOT NULL, CHECK (case_cost >= 0) | Cost per case locked at order creation |

* **Unique Index:** `(order_id, item_id)`

---

## Acceptance Criteria (Gherkin)

### US-7.1 — Create supplier purchase order (Draft)

#### Scenario: Successfully create a draft purchase order with line items
* **Given** supplier "Acme Beverage Supply" (ID `10`) exists with minimum order $250.00
* **And** item "DRK-COLA-01" (ID `101`, case cost $12.50) is supplied by vendor `10`
* **When** the office manager creates a new supplier order for warehouse "DAL-01"
* **And** adds line item: Item ID `101`, Quantity `25` cases ($312.50 total)
* **And** saves the order as draft
* **Then** the system creates `supplier_orders` record in status `"DRAFT"` with `total_amount` = 312.50
* **And** displays confirmation "Purchase Order PO-202611-0001 saved as Draft"

#### Scenario: Attempt to create order with items from a different supplier
* **Given** item "SNK-CHIP-05" is supplied by vendor "Frito Snack Co" (ID `20`)
* **When** the office manager attempts to add "SNK-CHIP-05" to an order for "Acme Beverage Supply" (ID `10`)
* **Then** the system rejects the line item
* **And** displays an error message "Item SNK-CHIP-05 does not belong to supplier Acme Beverage Supply"

---

### US-7.2 — Edit supplier order

#### Scenario: Successfully update quantities on a draft order
* **Given** order "PO-202611-0001" is in status `"DRAFT"` with 25 cases of item "DRK-COLA-01"
* **When** the office manager changes quantity to 30 cases
* **And** saves the update
* **Then** the system updates `quantity_ordered` to 30
* **And** recalculates `total_amount` to $375.00
* **And** displays confirmation "Order updated successfully"

#### Scenario: Prevent editing an order that has already been submitted
* **Given** order "PO-202611-0001" is in status `"SUBMITTED"`
* **When** the office manager attempts to edit line item quantities
* **Then** the system locks the fields against editing
* **And** displays an error message "Cannot edit purchase order: Order has already been submitted to vendor"

---

### US-7.3 — Submit / Send order to supplier

#### Scenario: Successfully submit an order meeting vendor requirements
* **Given** draft order "PO-202611-0001" has a total value of $375.00 (exceeding vendor min order $250.00)
* **When** the office manager clicks "Submit Order to Vendor"
* **And** confirms submission
* **Then** the system updates order status to `"SUBMITTED"`
* **And** transmits purchase order EDI/email to vendor "orders@acmebev.com"
* **And** displays confirmation "Order PO-202611-0001 submitted to Acme Beverage Supply"

#### Scenario: Prevent submitting order below vendor minimum spend threshold
* **Given** supplier "Acme Beverage Supply" requires a minimum order of $250.00
* **And** draft order "PO-202611-0002" contains 10 cases totaling $125.00
* **When** the office manager attempts to submit the order
* **Then** the system rejects the submission
* **And** displays an error message "Order total ($125.00) is below supplier minimum order threshold of $250.00"
* **And** the order remains in `"DRAFT"` status

---

### US-7.4 — Cancel / Delete supplier order

#### Scenario: Successfully cancel a submitted order
* **Given** order "PO-202611-0001" is in status `"SUBMITTED"` with zero received items
* **When** the office manager selects "Cancel Order" with reason "Vendor out of stock"
* **And** confirms cancellation
* **Then** the system updates order status to `"CANCELLED"`
* **And** sends cancellation notice to the vendor
* **And** displays confirmation "Order PO-202611-0001 has been cancelled"

#### Scenario: Prevent cancelling order that has already been partially received at dock
* **Given** order "PO-202611-0001" is in status `"PARTIALLY_RECEIVED"`
* **When** the office manager attempts to cancel the order
* **Then** the system rejects the cancellation
* **And** displays an error message "Cannot cancel order: shipments have already been received at the loading dock. Process dock returns instead"

---

## Definition of Done

* [ ] Backend and frontend implemented per this spec (**FR-001** – **FR-008** satisfied)
* [ ] Database tables `supplier_orders` and `supplier_order_items` created with state constraints
* [ ] API endpoints enforce lifecycle transitions and vendor minimum order checks
* [ ] Gherkin acceptance criteria verified by automated integration tests
* [ ] Cross-tenant access returns `404 Not Found`

---

## Out of Scope

* Automated replenishment PO calculation ([Feature 12 — Auto-Generate Supplier Orders](feature-12-auto-generate-supplier-orders.md))
* Dock truck check-in and receiving ([Feature 13 — Receive Supplier Shipments](feature-13-receive-supplier-shipments.md))
* Inventory putaway into bins ([Feature 14 — Putaway and Stock Inventory](feature-14-putaway-and-stock-inventory.md))
