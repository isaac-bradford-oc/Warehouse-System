# Feature 9: Maintain Customer Orders

**Feature ID:** 9  
**Branch pattern:** `feature/9-maintain-customer-orders`  
**Status:** Ready  
**Created:** 2026-09-19  
**Last Updated:** 2026-11-10  
**Input:** Manage customer sales orders, line items, and fulfillment release status.  
**Depends on:** [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md), [Feature 3 — Maintain Items](feature-3-maintain-items.md), [Feature 8 — Maintain Customers](feature-8-maintain-customers.md)  

---

## User Stories

### US-9.1: Create Customer Order
**As an** office manager  
**I want to** enter a customer sales order with requested items, quantities, and delivery dates  
**So that** warehouse operations can plan inventory allocation and fulfillment  

* **Priority:** P1  
* **Independent test:** Create a customer sales order with multiple line items; verify order header and line items persist with correct prices and totals in `"PENDING"` status.  
* **Acceptance scenarios:** see [US-9.1](#us-91--create-customer-order) under Acceptance Criteria.

### US-9.2: Edit Customer Order
**As an** office manager  
**I want to** update line items, requested quantities, or shipping address on a pending order  
**So that** customer-requested amendments are captured prior to warehouse fulfillment  

* **Priority:** P1  
* **Independent test:** Update line item quantities on a pending order; verify total price recalculates and updates in the database.  
* **Acceptance scenarios:** see [US-9.2](#us-92--edit-customer-order) under Acceptance Criteria.

### US-9.3: Release Order for Picking
**As an** office manager  
**I want to** release an order for warehouse picking once inventory availability is verified  
**So that** order picking workflows can begin on the warehouse floor  

* **Priority:** P1  
* **Independent test:** Release a pending order with sufficient inventory; verify status transitions to `"RELEASED_FOR_PICKING"` and becomes available to pickers.  
* **Acceptance scenarios:** see [US-9.3](#us-93--release-order-for-picking) under Acceptance Criteria.

### US-9.4: Cancel Customer Order
**As an** office manager  
**I want to** cancel an order if requested by the customer before shipping  
**So that** unpicked items are released from reservation and no unauthorized deliveries occur  

* **Priority:** P1  
* **Independent test:** Cancel a pending or released order; verify status changes to `"CANCELLED"` and inventory reservations are freed.  
* **Acceptance scenarios:** see [US-9.4](#us-94--cancel-customer-order) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers to create, view, update, release, and cancel customer sales orders.
- **FR-002**: Every order MUST be associated with a valid `company_id`, fulfillment `warehouse_id`, `customer_id`, and authorizer (`worker_id`).
- **FR-003**: Customer orders MUST follow a defined state lifecycle: `PENDING` -> `RELEASED_FOR_PICKING` -> `PICKED` -> `SHIPPED` -> `DELIVERED` (or `CANCELLED`).
- **FR-004**: Orders in status `PICKED`, `SHIPPED`, or `DELIVERED` MUST NOT permit line item modifications.
- **FR-005**: System MUST validate that customer credit limit is not exceeded by the total order value when creating an order.
- **FR-006**: Releasing an order (`RELEASED_FOR_PICKING`) MUST verify that on-hand unreserved inventory is sufficient for all line items.
- **FR-007**: System MUST generate a unique, non-repeating `order_number` (e.g., `SO-202611-0001`).
- **FR-008**: System MUST display unambiguous confirmation and actionable validation errors.

---

## Data Ownership & Isolation

Customer sales orders belong to the tenant company and fulfillment warehouse.

| Rule | Requirement |
|---|---|
| **Read scope** | `GET /warehouse/customer-orders` returns orders where `company_id = req.user.companyId` and `warehouse_id = req.user.warehouseId`. |
| **Write scope** | Editing is permitted only for orders matching `req.user.companyId` and currently in `PENDING` status. |
| **Create scope** | New orders automatically associate with `req.user.companyId` and authorizer `req.user.id`. |
| **Cross-user access** | Accessing an order belonging to another tenant or warehouse returns `404 Not Found`. |
| **UI scope** | Users only see customer orders targeted for their authorized facility. |
| **Implementation** | Centralized query helper `getAccessibleCustomerOrderOrNull(req, orderId)`. |

---

## API Requirements

| Method | Endpoint | Auth | Role | Purpose |
|---|---|---|---|---|
| `GET` | `/warehouse/customer-orders` | Yes | `office manager`, `company admin` | List customer sales orders |
| `GET` | `/warehouse/customer-orders/:id` | Yes | `office manager`, `company admin` | Get order and line items |
| `POST` | `/warehouse/customer-orders` | Yes | `office manager`, `company admin` | Create pending sales order |
| `PUT` | `/warehouse/customer-orders/:id` | Yes | `office manager`, `company admin` | Update line items on pending order |
| `POST` | `/warehouse/customer-orders/:id/release` | Yes | `office manager`, `company admin` | Release order for floor picking |
| `POST` | `/warehouse/customer-orders/:id/cancel` | Yes | `office manager`, `company admin` | Cancel order & release reservations |

**Create Request Body (`POST /warehouse/customer-orders`):**
```json
{
  "warehouse_id": 2,
  "customer_id": 5,
  "requested_delivery_date": "2026-11-18",
  "items": [
    {
      "item_id": 101,
      "quantity_ordered": 400,
      "unit_price": 1.25
    }
  ]
}
```

**Success Response (`201 Created` / `200 OK`):**
```json
{
  "id": 22,
  "order_number": "SO-202611-0001",
  "company_id": 1,
  "warehouse_id": 2,
  "customer_id": 5,
  "authorizer_worker_id": 4,
  "order_date": "2026-11-10",
  "requested_delivery_date": "2026-11-18",
  "total_amount": 500.00,
  "status": "PENDING",
  "items": [
    {
      "id": 110,
      "item_id": 101,
      "quantity_ordered": 400,
      "quantity_picked": 0,
      "quantity_shipped": 0,
      "unit_price": 1.25
    }
  ],
  "created_at": "2026-11-10T12:00:00.000Z"
}
```

**Error Response (`400 Bad Request` / `404 Not Found` / `409 Conflict`):**
```json
{
  "message": "Cannot create order: Order total ($500.00) exceeds customer available credit ($200.00)"
}
```

---

## Screen Requirements

### [View: Customer Sales Orders] — route name `customer-orders`
- **Heading:** **Customer Sales Orders**
- **Warehouse & Status Filters:** `<v-select>` facility selector; filter chips for `PENDING`, `RELEASED_FOR_PICKING`, `PICKED`, `SHIPPED`, `DELIVERED`, `CANCELLED`
- **Primary action:** **+ Create Sales Order** (`oc-cta`) opens order modal with:
  - Customer selector (`customer_id`), fulfillment warehouse, requested delivery date
  - Line items dynamic table: SKU autocomplete, ordered unit count, unit price, line total
  - **Save Order** (`oc-cta`) and **Cancel** buttons
- **Data Table:** `<v-data-table>` displaying columns:
  - SO Number, Customer Name, Order Date, Delivery Target, Total Amount ($), Status Chip
  - Row Actions:
    - **View Details** icon (`aria-label="View order"`)
    - **Release for Picking** button / icon (`aria-label="Release order"`, active only if PENDING)
    - **Cancel** icon (`aria-label="Cancel order"`)
- **Empty state:** **"No customer orders found for this warehouse. Create your first sales order."**
- **Loading state:** Skeleton rows
- **Error state:** `<v-alert type="error">` displaying API error message

---

## Key Entities

- **Customer Order**: Sales order header tracking order number, customer, fulfillment warehouse, order date, requested delivery date, total price, and status (`PENDING` | `RELEASED_FOR_PICKING` | `PICKED` | `SHIPPED` | `DELIVERED` | `CANCELLED`).
- **Customer Order Item**: Line item defining ordered SKU, quantity ordered, unit selling price, and fulfillment quantities (picked, shipped).

---

## Data Model Requirements

### `customer_orders`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique internal order ID |
| `order_number` | VARCHAR(30) | NOT NULL, UNIQUE | Human-readable sales order number |
| `company_id` | INT / BIGINT | FK -> `companies(id)`, NOT NULL | Owning tenant company |
| `warehouse_id` | INT / BIGINT | FK -> `warehouses(id)`, NOT NULL | Origin fulfillment warehouse |
| `customer_id` | INT / BIGINT | FK -> `customers(id)`, NOT NULL | Purchasing customer account |
| `authorizer_worker_id`| INT / BIGINT | FK -> `workers(id)`, NOT NULL | Staff member entering order |
| `order_date` | DATE | NOT NULL | Date order was placed |
| `requested_delivery_date` | DATE | NOT NULL | Customer target delivery date |
| `total_amount` | DECIMAL(12,2) | NOT NULL, DEFAULT 0.00 | Total extended order value |
| `status` | VARCHAR(30) | NOT NULL, DEFAULT `'PENDING'` | Enum: `'PENDING'`, `'RELEASED_FOR_PICKING'`, `'PICKED'`, `'SHIPPED'`, `'DELIVERED'`, `'CANCELLED'` |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp created |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp updated |

### `customer_order_items`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique line item ID |
| `order_id` | INT / BIGINT | FK -> `customer_orders(id)`, NOT NULL | Parent sales order |
| `item_id` | INT / BIGINT | FK -> `items(id)`, NOT NULL | Product SKU |
| `quantity_ordered` | INT | NOT NULL, CHECK (quantity_ordered > 0) | Units ordered by customer |
| `quantity_picked` | INT | NOT NULL, DEFAULT 0, CHECK (quantity_picked >= 0) | Units physically picked |
| `quantity_shipped`| INT | NOT NULL, DEFAULT 0, CHECK (quantity_shipped >= 0) | Units loaded and shipped |
| `unit_price` | DECIMAL(10,2) | NOT NULL, CHECK (unit_price >= 0) | Selling price per unit |

* **Unique Index:** `(order_id, item_id)`

---

## Acceptance Criteria (Gherkin)

### US-9.1 — Create customer order

#### Scenario: Successfully create a customer sales order
* **Given** customer "Lone Star Markets" (ID `5`) has available credit of $10,000.00
* **And** item "DRK-COLA-01" (ID `101`) has unit price $1.25
* **When** the office manager creates a new customer order for warehouse "DAL-01"
* **And** adds line item: Item ID `101`, Quantity `400` units ($500.00 total)
* **And** submits the order
* **Then** the system creates `customer_orders` record in status `"PENDING"` with `total_amount` = 500.00
* **And** displays confirmation "Customer Order SO-202611-0001 created successfully"

#### Scenario: Attempt to create order exceeding customer credit limit
* **Given** customer "Lone Star Markets" has remaining available credit of $200.00
* **When** the office manager creates an order totaling $500.00
* **Then** the system rejects order creation
* **And** displays an error message "Cannot create order: Order total ($500.00) exceeds customer available credit ($200.00)"

---

### US-9.2 — Edit customer order

#### Scenario: Successfully edit line quantities on a pending order
* **Given** order "SO-202611-0001" is in status `"PENDING"` with quantity 400
* **When** the office manager changes quantity to 300
* **And** saves changes
* **Then** the system updates `quantity_ordered` to 300
* **And** recalculates `total_amount` to $375.00
* **And** displays confirmation "Order SO-202611-0001 updated successfully"

#### Scenario: Prevent editing an order that has already been picked
* **Given** order "SO-202611-0001" is in status `"PICKED"`
* **When** the office manager attempts to edit line items
* **Then** the system locks the order against modification
* **And** displays an error message "Cannot edit order: Order has already been picked by warehouse operations"

---

### US-9.3 — Release order for picking

#### Scenario: Successfully release order when stock is available
* **Given** order "SO-202611-0001" is in status `"PENDING"` for 300 units of "DRK-COLA-01"
* **And** warehouse "DAL-01" has 500 unreserved units of "DRK-COLA-01" on hand
* **When** the office manager clicks "Release for Picking"
* **Then** the system reserves 300 units for this order
* **And** transitions order status to `"RELEASED_FOR_PICKING"`
* **And** displays confirmation "Order SO-202611-0001 released for picking"

#### Scenario: Prevent release when inventory is insufficient
* **Given** order "SO-202611-0001" requires 300 units of "DRK-COLA-01"
* **And** warehouse "DAL-01" only has 100 unreserved units on hand
* **When** the office manager attempts to release the order
* **Then** the system rejects the release
* **And** displays an error message "Cannot release order: Insufficient unreserved stock for DRK-COLA-01 (Required: 300, Available: 100)"
* **And** the order remains in `"PENDING"` status

---

### US-9.4 — Cancel customer order

#### Scenario: Successfully cancel a released order and release stock reservation
* **Given** order "SO-202611-0001" is in status `"RELEASED_FOR_PICKING"` with 300 reserved units
* **When** the office manager clicks "Cancel Order" with reason "Customer cancellation request"
* **Then** the system changes order status to `"CANCELLED"`
* **And** releases the 300 reserved units back to available stock
* **And** displays confirmation "Order SO-202611-0001 has been cancelled and inventory reservations released"

#### Scenario: Prevent cancelling an order that has already shipped
* **Given** order "SO-202611-0001" is in status `"SHIPPED"` and loaded on a delivery vehicle
* **When** the office manager attempts to cancel order "SO-202611-0001"
* **Then** the system rejects the cancellation
* **And** displays an error message "Cannot cancel order: Shipment is currently in transit. Please process as a customer return upon delivery"
* **And** the order remains in `"SHIPPED"` status

---

## Definition of Done

* [ ] Backend and frontend implemented per this spec (**FR-001** – **FR-008** satisfied)
* [ ] Database tables `customer_orders` and `customer_order_items` created with state constraints
* [ ] API endpoints `/warehouse/customer-orders` enforce tenant scoping and credit validation
* [ ] Gherkin acceptance criteria verified by automated integration tests
* [ ] Cross-tenant access returns `404 Not Found`

---

## Out of Scope

* Floor picking execution and bin barcode scanning ([Feature 15 — Pick Customer Orders](feature-15-pick-customer-orders.md))
* Bill of Lading generation and truck dispatch ([Feature 16 — Ship and Deliver Customer Orders](feature-16-ship-and-deliver-customer-orders.md))
* Customer invoice generation and payment remittance
