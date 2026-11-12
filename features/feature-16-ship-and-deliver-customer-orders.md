# Feature 16: Ship and Deliver Customer Orders

**Feature ID:** 16  
**Branch pattern:** `feature/16-ship-and-deliver-customer-orders`  
**Status:** Ready  
**Created:** 2026-11-10  
**Last Updated:** 2026-11-10  
**Input:** Staged picked customer orders, route stops, driver manifests, and delivery confirmations.  
**Depends on:** [Feature 8 — Maintain Customers](feature-8-maintain-customers.md), [Feature 9 — Maintain Customer Orders](feature-9-maintain-customer-orders.md), [Feature 10 — Maintain Delivery Routes](feature-10-maintain-routes.md), [Feature 11 — Maintain Reports](feature-11-maintain-reports.md), [Feature 15 — Pick Customer Orders](feature-15-pick-customer-orders.md)  

---

## User Stories

### US-16.1: Generate Bill of Lading and Shipping Manifest
**As an** office manager or shipping coordinator  
**When** customer orders are picked and assigned to a delivery route  
**I want** the system to generate an official Bill of Lading (BOL) and sequenced driver manifest  
**So that** drivers have the legal transportation documents and customer stop instructions required for delivery  

* **Priority:** P1  
* **Independent test:** Assign 2 picked orders to Route RTE-AUS-01; trigger BOL generation and verify manifest lists customer stops, package counts, and legal BOL numbers.  
* **Acceptance scenarios:** see [US-16.1](#us-161--generate-bill-of-lading-and-shipping-manifest) under Acceptance Criteria.

### US-16.2: Dispatch Truck and Confirm Shipment
**As a** warehouse shipping clerk or delivery driver  
**When** goods are verified and loaded onto the delivery truck  
**I want to** mark the assigned orders as dispatched / shipped  
**So that** order status updates to `"SHIPPED"`, customers receive dispatch notifications, and inventory in-transit is tracked  

* **Priority:** P1  
* **Independent test:** Confirm truck departure for a delivery manifest; verify all included customer orders transition from `"PICKED"` to `"SHIPPED"`.  
* **Acceptance scenarios:** see [US-16.2](#us-162--dispatch-truck-and-confirm-shipment) under Acceptance Criteria.

### US-16.3: Confirm Delivery with Proof of Delivery (POD)
**As a** delivery driver  
**When** I arrive at a customer delivery location and unload the shipment  
**I want to** record customer acceptance with an electronic signature or delivery photo  
**So that** the order is marked as `"DELIVERED"` and proof of delivery is securely archived for invoicing  

* **Priority:** P1  
* **Independent test:** Record delivery confirmation with recipient signature at customer stop; verify order transitions to `"DELIVERED"` status with timestamp and signature stored.  
* **Acceptance scenarios:** see [US-16.3](#us-163--confirm-delivery-with-proof-of-delivery-pod) under Acceptance Criteria.

### US-16.4: Log Delivery Damage, Shortage, or Customer Rejection
**As a** delivery driver  
**When** a customer refuses items due to transit damage or records a package count discrepancy  
**I want to** log the specific items rejected, damaged quantities, and reason codes at the time of delivery  
**So that** customer invoices are automatically adjusted and an exception report is created for office manager investigation  

* **Priority:** P1  
* **Independent test:** Record 2 damaged/rejected cases during delivery; verify delivery is recorded with exceptions, billable quantity is adjusted, and a `"DELIVERY_DISCREPANCY"` report is created in Feature 11.  
* **Acceptance scenarios:** see [US-16.4](#us-164--log-delivery-damage-shortage-or-customer-rejection) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow generation of a Bill of Lading (BOL) only for customer orders in status `PICKED`.
- **FR-002**: Every BOL MUST include unique BOL number, carrier/route identifier, driver name, shipper address, consignee (customer) delivery address, item descriptions, and total package/carton count.
- **FR-003**: Confirming truck dispatch MUST transition all associated customer orders from `PICKED` to `SHIPPED`.
- **FR-004**: System MUST allow delivery drivers to record delivery confirmations on mobile devices, capturing recipient name, timestamp, and GPS coordinates / digital signature.
- **FR-005**: Successful full delivery MUST transition order status to `DELIVERED`.
- **FR-006**: If items are damaged, missing, or rejected at delivery:
  * System MUST record `quantity_shipped` vs `quantity_delivered`.
  * System MUST create a `DELIVERY_DISCREPANCY` report in [Feature 11 — Maintain Reports](feature-11-maintain-reports.md).
  * System MUST adjust the final billable order amount to reflect only accepted items.
- **FR-007**: System MUST prevent duplicate delivery confirmation for an order already marked `DELIVERED`.

---

## Data Ownership & Isolation

Bills of Lading, shipping dispatch records, and proof of delivery confirmations are strictly scoped to the tenant company and origin warehouse.

| Rule | Requirement |
|---|---|
| **Read scope** | `GET /warehouse/shipping/manifests` and `GET /warehouse/bills-of-lading` return shipping documents matching `company_id = req.user.companyId` and `warehouse_id = req.user.warehouseId`. Drivers access stop manifests assigned to their worker ID. |
| **Write scope** | Generating BOLs and dispatching routes is restricted to `shipping clerk`, `office manager`, and `company admin`. Recording POD confirmations is restricted to the assigned `driver`, `office manager`, or `company admin`. |
| **Order & Route isolation** | BOLs and manifests link only orders and routes belonging to the same tenant company and origin facility. |
| **Cross-user access** | Accessing shipping documents, routes, or orders belonging to another tenant or warehouse returns `404 Not Found`. |
| **UI scope** | Shipping clerks only see orders staged at their facility; drivers only see stops assigned to their active route run. |
| **Implementation** | Centralized query helpers `getAccessibleBolOrNull(req, bolId)` and `getAccessibleRouteOrNull(req, routeId)`. |

---

## API Requirements

| Method | Endpoint | Auth | Role | Purpose |
|---|---|---|---|---|
| `POST` | `/warehouse/customer-orders/:id/generate-bol` | Yes | `shipping clerk`, `office manager`, `company admin` | Generate legal BOL document and carton manifest for picked order |
| `GET` | `/warehouse/bills-of-lading/:id` | Yes | `shipping clerk`, `driver`, `office manager`, `company admin` | View BOL document details and printable payload |
| `POST` | `/warehouse/routes/:id/dispatch` | Yes | `shipping clerk`, `office manager`, `company admin` | Confirm truck departure and transition orders to `SHIPPED` |
| `GET` | `/warehouse/driver/manifest` | Yes | `driver` | Get sequenced stop manifest for authenticated driver |
| `POST` | `/warehouse/customer-orders/:id/delivery-confirm` | Yes | `driver`, `office manager`, `company admin` | Record recipient signature, delivered quantities, or damage rejection |

**Delivery Confirmation Request Body (`POST /warehouse/customer-orders/:id/delivery-confirm`):**
```json
{
  "recipient_name": "Dave Miller",
  "signature_data": "data:image/png;base64,iVBORw0KGgo...",
  "items": [
    {
      "item_id": 101,
      "quantity_delivered": 10,
      "quantity_rejected": 2,
      "rejection_reason": "2 cartons water damaged in transit"
    }
  ]
}
```

**Success Response (`200 OK`):**
```json
{
  "order_id": 201,
  "status": "DELIVERED",
  "pod_id": 88,
  "recipient_name": "Dave Miller",
  "delivered_at": "2026-11-10T16:45:00.000Z",
  "discrepancy_report_id": 14,
  "adjusted_invoice_total": 240.00
}
```

**Error Response (`400 Bad Request` / `404 Not Found` / `409 Conflict`):**
```json
{
  "message": "Cannot confirm delivery: Order SO-202611-0001 has already been marked DELIVERED"
}
```

---

## Screen Requirements

### [View: Shipping Staging & Route Dispatch] — route name `shipping`
- **Heading:** **Shipping Staging & Route Dispatch**
- **Facility & Date Filters:** `<v-select>` facility selector; route delivery date picker
- **Staged Orders Table:** `<v-data-table>` displaying:
  - Order Number, Customer Name, Outbound Staging Lane, Carton Count, Assigned Route, BOL Number, Order Status
  - Row Actions:
    - **Generate BOL** button (`oc-cta`, active if status is `PICKED` and BOL not yet generated)
    - **Print BOL** icon button (`aria-label="Print Bill of Lading"`)
- **Route Dispatch Card:**
  - Route code, Driver selector, Vehicle ID, Total Assigned Orders, Total Cartons
  - **Dispatch Truck** button (`oc-cta`) opens confirmation modal to transition all route orders to `SHIPPED`

### [View: Driver Mobile Delivery Manifest] — route name `driver-delivery`
- **Heading:** **Driver Delivery Manifest**
- **Driver Header Bar:** Driver name, Route ID, current stop progress indicator (`Stop 2 of 5`)
- **Sequential Stops List:** `<v-card>` list displaying:
  - Customer Name, Delivery Address, Contact Phone, Carton Count, Order Status Badge
  - **Arrive at Stop** action button
- **Proof of Delivery (POD) Dialog:**
  - Recipient Name input: `<v-text-field label="Recipient Name" required>`
  - Delivered vs Rejected line items list with numeric counter steppers
  - Rejection reason input (mandatory when `quantity_rejected > 0`)
  - Electronic Signature Pad (`<canvas>` component with **Clear** and **Accept Signature** controls)
  - **Confirm Delivery** button (`oc-cta`)
- **Empty state:** **"No active delivery stops assigned to your route today."**
- **Loading state:** Skeleton cards
- **Error state:** `<v-alert type="error">` displaying API error message
- **Responsive design:** Optimized for mobile phones and handheld driver terminals

---

## Key Entities

- **Bill of Lading (BOL)**: Legal freight shipping document linking route run, customer sales orders, freight weights, carton counts, and shipper/carrier signatures.
- **Delivery Confirmation (POD)**: Proof of delivery record tracking delivery timestamp, receiving agent signature, delivered quantities, and exception notes.

---

## Data Model Requirements

### `bills_of_lading`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique BOL internal ID |
| `bol_number` | VARCHAR(30) | NOT NULL, UNIQUE | Document identifier (e.g., `BOL-202611-0089`) |
| `route_id` | INT / BIGINT | FK -> `routes(id)`, NOT NULL | Assigned delivery route |
| `order_id` | INT / BIGINT | FK -> `customer_orders(id)`, NOT NULL | Customer sales order |
| `carrier_worker_id` | INT / BIGINT | FK -> `workers(id)`, NOT NULL | Assigned delivery driver |
| `total_cartons` | INT | NOT NULL, CHECK (total_cartons > 0) | Carton/case count |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Generation timestamp |

### `delivery_confirmations`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique POD ID |
| `order_id` | INT / BIGINT | FK -> `customer_orders(id)`, NOT NULL, UNIQUE | Customer sales order |
| `bol_id` | INT / BIGINT | FK -> `bills_of_lading(id)`, NOT NULL | Associated BOL |
| `delivered_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Actual delivery timestamp |
| `recipient_name` | VARCHAR(100) | NOT NULL | Person signing at customer dock |
| `signature_image_url`| VARCHAR(255) | NULLABLE | Stored signature capture image |
| `has_discrepancy` | BOOLEAN | NOT NULL, DEFAULT FALSE | True if damages or rejections occurred |
| `notes` | TEXT | NULLABLE | Driver delivery comments |

---

## Acceptance Criteria (Gherkin)

### US-16.1 — Generate Bill of Lading and shipping manifest

#### Scenario: Successfully generate BOL for picked customer order
* **Given** customer order "SO-202611-0001" is in status `"PICKED"` with 20 total cartons
* **And** the order is assigned to Route "RTE-AUS-01" with driver "Dave Miller"
* **When** shipping operations triggers manifest generation
* **Then** the system creates a Bill of Lading "BOL-202611-0089" linking order "SO-202611-0001"
* **And** sets `total_cartons` = 20
* **And** generates a printable/downloadable PDF manifest showing customer delivery address and stop instructions

#### Scenario: Attempt to generate BOL for unpicked order
* **Given** customer order "SO-202611-0002" is in status `"RELEASED_FOR_PICKING"`
* **When** shipping operations attempts to generate a Bill of Lading
* **Then** the system blocks BOL creation
* **And** displays an error message "Cannot generate Bill of Lading: Order SO-202611-0002 has not completed picking"

---

### US-16.2 — Dispatch truck and confirm shipment

#### Scenario: Dispatch truck with loaded customer orders
* **Given** order "SO-202611-0001" has generated BOL "BOL-202611-0089" and is loaded on truck
* **When** driver "Dave Miller" confirms truck departure for route "RTE-AUS-01"
* **Then** the system updates order status to `"SHIPPED"`
* **And** sets `quantity_shipped` on order line items to match picked quantities
* **And** displays confirmation "Order SO-202611-0001 marked as In Transit"

#### Scenario: Attempt to dispatch delivery truck with no assigned driver
* **Given** route "RTE-AUS-02" has loaded orders but no driver assigned
* **When** shipping clerk attempts to confirm truck departure
* **Then** the system rejects dispatch
* **And** displays an error message "Cannot dispatch route: No active driver assigned to Route RTE-AUS-02"

---

### US-16.3 — Confirm delivery with Proof of Delivery (POD)

#### Scenario: Driver completes full delivery with customer signature
* **Given** order "SO-202611-0001" is in status `"SHIPPED"`
* **When** driver "Dave Miller" arrives at customer "Lone Star Markets"
* **And** unloads all 20 cartons in good condition
* **And** captures recipient signature "Sarah Jenkins" and confirms delivery
* **Then** the system creates a delivery confirmation record with `has_discrepancy` = FALSE
* **And** updates order status to `"DELIVERED"`
* **And** displays confirmation "Delivery confirmed for Order SO-202611-0001"

#### Scenario: Prevent duplicate delivery confirmation
* **Given** order "SO-202611-0001" is already in status `"DELIVERED"`
* **When** a driver attempts to submit another delivery confirmation for the same order
* **Then** the system rejects the submission
* **And** displays an error message "Order SO-202611-0001 has already been marked DELIVERED"

---

### US-16.4 — Log delivery damage, shortage, or customer rejection

#### Scenario: Customer rejects damaged cartons upon delivery
* **Given** order "SO-202611-0001" contained 20 cases of "DRK-COLA-01" (unit price $12.50/case)
* **When** the driver delivers the shipment, but 3 cases are damaged and rejected by the customer
* **Then** the driver enters 17 cases accepted and 3 cases rejected with reason "Transit crush damage"
* **And** captures recipient signature "Sarah Jenkins" with notes
* **Then** the system records delivery confirmation with `has_discrepancy` = TRUE
* **And** updates order status to `"DELIVERED"` with billing adjusted for 17 cases ($212.50 instead of $250.00)
* **And** automatically creates a report in Feature 11 with type `"DELIVERY_DISCREPANCY"`
* **And** notifies the office manager: "Delivery exception logged for SO-202611-0001: 3 cases rejected due to damage"

#### Scenario: Driver attempts to log rejected items without entering required discrepancy reason
* **Given** order "SO-202611-0001"
* **When** the driver enters 2 cases rejected but leaves the discrepancy reason blank
* **And** attempts to confirm delivery
* **Then** the system rejects confirmation
* **And** displays an inline error message "Discrepancy explanation is required when logging rejected or missing items"

---

## Definition of Done

* [ ] Shipping dispatch and mobile driver delivery UI implemented per this spec (**FR-001** – **FR-007** satisfied)
* [ ] Database tables `bills_of_lading` and `delivery_confirmations` created with foreign keys and unique constraints
* [ ] BOL generation permitted only for orders in `PICKED` status
* [ ] Dispatching route atomically transitions assigned orders to `SHIPPED`
* [ ] Proof of delivery captures recipient name and electronic signature
* [ ] Delivery exceptions create `DELIVERY_DISCREPANCY` report in [Feature 11 — Maintain Reports](feature-11-maintain-reports.md) and adjust billable invoice amount
* [ ] Cross-tenant access returns `404 Not Found`
* [ ] Gherkin acceptance criteria verified by automated integration tests

---

## Out of Scope

* Third-party parcel carrier API integration (FedEx / UPS Web Services)
* Real-time GPS telematics hardware integration and live vehicle tracking maps
* Driver turn-by-turn navigation engine

