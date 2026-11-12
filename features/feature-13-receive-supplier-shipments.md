# Feature 13: Receive Supplier Shipments

**Feature ID:** 13  
**Branch pattern:** `feature/13-receive-supplier-shipments`  
**Status:** Ready  
**Created:** 2026-11-10  
**Last Updated:** 2026-11-10  
**Input:** Physical dock shipment arrival, carrier Bill of Lading (BOL), and purchase order line item receipts.  
**Depends on:** [Feature 3 — Maintain Items](feature-3-maintain-items.md), [Feature 7 — Maintain Supplier Orders](feature-7-maintain-supplier-orders.md), [Feature 11 — Maintain Reports](feature-11-maintain-reports.md)  

---

## User Stories

### US-13.1: Record Received Shipment Quantities
**As a** warehouse receiving clerk  
**When a** supplier delivery truck arrives at the loading dock  
**I want to** record the actual case quantities received for each item on the purchase order  
**So that** the system accurately reflects incoming physical inventory delivered by the vendor  

* **Priority:** P1  
* **Independent test:** Check in a delivery truck against a submitted purchase order, enter received case counts matching the PO, and confirm order transitions to `"RECEIVED"` status.  
* **Acceptance scenarios:** see [US-13.1](#us-131--record-received-shipment-quantities) under Acceptance Criteria.

### US-13.2: Record Partial Delivery
**As a** warehouse receiving clerk  
**When a** carrier delivers only a portion of the ordered items  
**I want to** record the partial case quantities received  
**So that** the purchase order remains open for the remaining balance and stock records reflect what arrived  

* **Priority:** P1  
* **Independent test:** Receive 10 cases of a 20-case order; verify purchase order status transitions to `"PARTIALLY_RECEIVED"` and expected pending quantity updates to 10.  
* **Acceptance scenarios:** see [US-13.2](#us-132--record-partial-delivery) under Acceptance Criteria.

### US-13.3: Log Damaged or Discrepant Items at Receiving
**As a** warehouse receiving clerk  
**When** delivered cartons or pallets are visibly damaged or quantities differ from the Bill of Lading  
**I want to** log the damaged or missing quantities and reason codes during check-in  
**So that** claims can be filed against the carrier/vendor and management is immediately notified  

* **Priority:** P1  
* **Independent test:** Log 2 damaged cases with photo/notes during check-in; verify receipt records only undamaged goods as staged for putaway and an exception report is created.  
* **Acceptance scenarios:** see [US-13.3](#us-133--log-damaged-or-discrepant-items-at-receiving) under Acceptance Criteria.

### US-13.4: Scan and Attach Bill of Lading (BOL)
**As a** warehouse receiving clerk  
**I want to** capture and attach the driver's signed Bill of Lading document or tracking number to the receiving record  
**So that** receiving paperwork is securely digitized for freight audit and supplier invoice matching  

* **Priority:** P1  
* **Independent test:** Attach BOL document reference to receiving record; verify document metadata links to the purchase order and prevents duplicate re-entry.  
* **Acceptance scenarios:** see [US-13.4](#us-134--scan-and-attach-bill-of-lading-bol) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST require receiving clerks to identify an active purchase order in status `SUBMITTED` or `PARTIALLY_RECEIVED` before logging a dock receipt.
- **FR-002**: System MUST allow entry of received quantities and damaged/rejected quantities per line item.
- **FR-003**: System MUST update `supplier_order_items.quantity_received` with good units received.
- **FR-004**: If all ordered quantities across all line items are fulfilled, system MUST transition order status to `RECEIVED`. If any line item is incomplete, system MUST set status to `PARTIALLY_RECEIVED`.
- **FR-005**: If damaged or missing items are logged, system MUST automatically create a `RECEIVING_DISCREPANCY` report in [Feature 11 — Maintain Reports](feature-11-maintain-reports.md) with details of the discrepancy.
- **FR-006**: System MUST create staging records for accepted goods to make them available for [Feature 14 — Putaway and Stock Inventory](feature-14-putaway-and-stock-inventory.md).
- **FR-007**: System MUST detect and reject duplicate processing of the same BOL tracking number for an already completed delivery.

---

## Data Ownership & Isolation

Receiving receipts and dock check-in transactions are strictly scoped to the tenant company and destination warehouse.

| Rule | Requirement |
|---|---|
| **Read scope** | `GET /warehouse/receiving-receipts` returns receipts where `warehouse_id = req.user.warehouseId` and belongs to `req.user.companyId`. |
| **Write scope** | Dock receiving is restricted to `receiving clerk`, `office manager`, and `company admin` assigned to the receiving warehouse. |
| **PO association** | Check-in requires a valid purchase order in status `SUBMITTED` or `PARTIALLY_RECEIVED` owned by the user's company and warehouse. |
| **Cross-user access** | Accessing receiving receipts or purchase orders belonging to another tenant or warehouse returns `404 Not Found`. |
| **UI scope** | Receiving clerks only see inbound shipments and open purchase orders targeted for their assigned facility. |
| **Implementation** | Centralized query helper `getAccessibleReceivingReceiptOrNull(req, receiptId)`. |

---

## API Requirements

| Method | Endpoint | Auth | Role | Purpose |
|---|---|---|---|---|
| `GET` | `/warehouse/receiving-receipts` | Yes | `receiving clerk`, `office manager`, `company admin` | List dock receipts for warehouse |
| `GET` | `/warehouse/receiving-receipts/:id` | Yes | `receiving clerk`, `office manager`, `company admin` | Get specific receipt lines and damage details |
| `POST` | `/warehouse/receiving-receipts` | Yes | `receiving clerk`, `office manager`, `company admin` | Record inbound shipment receipt, stage goods, and update PO |
| `GET` | `/warehouse/supplier-orders/:id/receiving-checkin` | Yes | `receiving clerk`, `office manager`, `company admin` | Validate open PO and retrieve expected line items for dock check-in |

**Create Request Body (`POST /warehouse/receiving-receipts`):**
```json
{
  "order_id": 105,
  "warehouse_id": 2,
  "carrier_name": "FedEx Freight",
  "bol_number": "BOL-FEDEX-991",
  "items": [
    {
      "item_id": 101,
      "quantity_accepted": 25,
      "quantity_damaged": 5,
      "damage_reason": "Crushed cartons, leaking cans"
    }
  ]
}
```

**Success Response (`201 Created`):**
```json
{
  "id": 12,
  "receipt_number": "RCV-202611-0012",
  "order_id": 105,
  "warehouse_id": 2,
  "carrier_name": "FedEx Freight",
  "bol_number": "BOL-FEDEX-991",
  "total_accepted_cases": 25,
  "total_damaged_cases": 5,
  "order_status": "PARTIALLY_RECEIVED",
  "discrepancy_report_id": 8,
  "received_at": "2026-11-10T14:30:00.000Z"
}
```

**Error Response (`400 Bad Request` / `404 Not Found` / `409 Conflict`):**
```json
{
  "message": "Damage reason description is required when logging damaged quantities"
}
```

---

## Screen Requirements

### [View: Dock Receiving & Inbound Check-In] — route name `receiving`
- **Heading:** **Dock Receiving & Inbound Check-In**
- **Facility & Dock Selector:** `<v-select>` facility selector; active dock door selector
- **PO Lookup Card:**
  - Autocomplete / barcode scanner input for Purchase Order (`PO-XXXXXX`)
  - Carrier name text input
  - BOL / Tracking Number input with duplicate-check validation
- **Receiving Line Items Grid:** `<v-data-table>` displaying:
  - SKU, Description, Ordered Cases, Previously Received, Remaining Expected
  - Accepted Cases input field (`<v-text-field type="number">`)
  - Damaged Cases input field (`<v-text-field type="number">`)
  - Damage Reason text input / select menu
  - Condition status badge
- **Action Toolbar:**
  - **Attach BOL Document / Photo** button
  - **Confirm Receipt** button (`oc-cta`) opens confirmation modal
- **Recent Inbound Receipts Table:** `<v-data-table>` displaying:
  - Receipt Number, PO Number, Carrier, BOL #, Accepted Cases, Damaged Cases, Time, View Details action
- **Empty state:** **"Scan or enter a purchase order number to start receiving an inbound truck."**
- **Loading state:** Skeleton table loaders
- **Error state:** `<v-alert type="error">` displaying API error message

---

## Key Entities

- **Receiving Receipt**: Receiving transaction record capturing receipt number, purchase order ID, carrier name, tracking / BOL number, receiving worker ID, timestamp, and dock door.
- **Receiving Receipt Line**: Line item detailing received SKU, accepted quantity, damaged quantity, and damage notes.

---

## Data Model Requirements

### `receiving_receipts`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique receipt ID |
| `receipt_number` | VARCHAR(30) | NOT NULL, UNIQUE | Document code (e.g., `RCV-202611-0012`) |
| `order_id` | INT / BIGINT | FK -> `supplier_orders(id)`, NOT NULL | Parent purchase order |
| `warehouse_id` | INT / BIGINT | FK -> `warehouses(id)`, NOT NULL | Receiving warehouse |
| `receiver_worker_id`| INT / BIGINT | FK -> `workers(id)`, NOT NULL | Dock clerk who checked in truck |
| `bol_number` | VARCHAR(50) | NOT NULL | Carrier Bill of Lading / tracking ID |
| `carrier_name` | VARCHAR(100) | NOT NULL | Freight carrier name |
| `received_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Time truck was checked in |

### `receiving_receipt_lines`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique receipt line ID |
| `receipt_id` | INT / BIGINT | FK -> `receiving_receipts(id)`, NOT NULL | Parent receipt |
| `item_id` | INT / BIGINT | FK -> `items(id)`, NOT NULL | Received item SKU |
| `quantity_accepted`| INT | NOT NULL, DEFAULT 0, CHECK (quantity_accepted >= 0) | Usable cases accepted |
| `quantity_damaged` | INT | NOT NULL, DEFAULT 0, CHECK (quantity_damaged >= 0) | Damaged/rejected cases |
| `damage_reason` | VARCHAR(255) | NULLABLE | Description of damage |

---

## Acceptance Criteria (Gherkin)

### US-13.1 — Record received shipment quantities

#### Scenario: Full shipment received in good condition
* **Given** purchase order "PO-202611-0001" is in status `"SUBMITTED"` for 30 cases of "DRK-COLA-01"
* **When** the receiving clerk checks in the delivery with BOL "BOL-FEDEX-991"
* **And** records 30 cases accepted and 0 cases damaged
* **And** confirms receipt
* **Then** the system records `quantity_received` = 30 on the order line
* **And** updates purchase order status to `"RECEIVED"`
* **And** stages 30 cases at the receiving dock for putaway
* **And** displays confirmation "Receipt RCV-202611-0012 recorded: PO-202611-0001 fully received"

#### Scenario: Attempt to check in shipment against a cancelled purchase order
* **Given** purchase order "PO-202611-0002" is in status `"CANCELLED"`
* **When** the receiving clerk enters "PO-202611-0002" during dock check-in
* **Then** the system rejects check-in
* **And** displays an error message "Cannot receive shipment: Purchase Order PO-202611-0002 has been cancelled"
* **And** does not allow quantity entry

---

### US-13.2 — Record partial delivery

#### Scenario: Partial shipment received at loading dock
* **Given** purchase order "PO-202611-0001" ordered 30 cases of "DRK-COLA-01"
* **When** the receiving clerk records 18 cases delivered and accepted with BOL "BOL-FEDEX-991A"
* **And** confirms partial receipt
* **Then** the system updates order `quantity_received` to 18
* **And** updates order status to `"PARTIALLY_RECEIVED"`
* **And** leaves 12 cases pending expected delivery
* **And** stages 18 cases for putaway

#### Scenario: Attempt to receive quantity greater than ordered without authorization
* **Given** purchase order "PO-202611-0001" has 12 cases remaining to be received
* **When** the clerk enters 20 cases received
* **Then** the system blocks the receipt confirmation
* **And** displays an error message "Over-receipt error: 20 cases exceeds remaining ordered balance of 12 cases. Supervisor override required."

---

### US-13.3 — Log damaged or discrepant items at receiving

#### Scenario: Receiving shipment with transit damage
* **Given** purchase order "PO-202611-0001" ordered 30 cases of "DRK-COLA-01"
* **When** the carrier delivers 30 cases, but 5 cases are crushed and leaking
* **Then** the receiving clerk enters `quantity_accepted` = 25 and `quantity_damaged` = 5 with reason "Crushed cartons, leaking cans"
* **When** the receipt is confirmed
* **Then** the system stages only 25 good cases for putaway
* **And** marks `quantity_received` as 25 on the purchase order
* **And** automatically generates a report in status `"OPEN"` with `report_type` = `"RECEIVING_DISCREPANCY"`
* **And** alerts the facility office manager of the 5 damaged cases

#### Scenario: Attempt to record damaged items without required damage reason
* **Given** purchase order "PO-202611-0001"
* **When** the clerk enters `quantity_damaged` = 3 but leaves the damage reason field empty
* **And** attempts to confirm receipt
* **Then** the system rejects confirmation
* **And** displays an inline validation error "Damage reason description is required when logging damaged quantities"

---

### US-13.4 — Scan and attach Bill of Lading (BOL)

#### Scenario: Successfully attach carrier BOL tracking reference
* **Given** an active delivery truck is at the dock with BOL number "BOL-FEDEX-991"
* **When** the receiving clerk enters carrier name "FedEx Freight" and BOL "BOL-FEDEX-991"
* **Then** the system attaches the BOL metadata to the receipt
* **And** confirms "BOL BOL-FEDEX-991 linked to receipt RCV-202611-0012"

#### Scenario: Duplicate BOL submission rejected
* **Given** a receipt has already been processed with BOL number "BOL-FEDEX-991"
* **When** a receiving clerk attempts to record another delivery for the same order using BOL "BOL-FEDEX-991"
* **Then** the system warns the clerk "BOL BOL-FEDEX-991 has already been processed for this order"
* **And** requires confirmation or entry of the correct unique freight document number

---

## Definition of Done

* [ ] Dock receiving backend APIs and frontend check-in UI implemented per this spec (**FR-001** – **FR-007** satisfied)
* [ ] Database tables `receiving_receipts` and `receiving_receipt_lines` created with foreign keys and unique constraints
* [ ] Purchase order status transitions correctly (`PARTIALLY_RECEIVED` or `RECEIVED`)
* [ ] Damaged quantities automatically create a `RECEIVING_DISCREPANCY` report in [Feature 11 — Maintain Reports](feature-11-maintain-reports.md)
* [ ] Cross-tenant access returns `404 Not Found`
* [ ] Gherkin acceptance criteria verified by automated integration tests

---

## Out of Scope

* Inventory putaway into bins ([Feature 14 — Putaway and Stock Inventory](feature-14-putaway-and-stock-inventory.md))
* Supplier invoice matching and accounts payable debit memo processing
* Freight claims submission to third-party carrier APIs

