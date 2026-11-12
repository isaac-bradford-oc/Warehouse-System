# Feature 10: Maintain Delivery Routes

**Feature ID:** 10  
**Branch pattern:** `feature/10-maintain-routes`  
**Status:** Ready  
**Created:** 2026-09-19  
**Last Updated:** 2026-11-10  
**Input:** Maintain master delivery routes, customer stop sequences, and assigned delivery drivers.  
**Depends on:** [Feature 1 — Maintain Companies](feature-1-maintain-companies.md), [Feature 2 — Maintain Warehouses](feature-2-maintain-warehouses.md), [Feature 5 — Maintain Workers](feature-5-maintain-workers.md), [Feature 8 — Maintain Customers](feature-8-maintain-customers.md)  

---

## User Stories

### US-10.1: Add Delivery Route
**As an** office manager  
**I want to** configure a new delivery route with sequenced customer stops and assigned drivers  
**So that** customer shipments can be efficiently batched and dispatched on designated delivery runs  

* **Priority:** P1  
* **Independent test:** Create a route with assigned primary driver and sequenced customer stops; verify route and stop sequences persist in the database.  
* **Acceptance scenarios:** see [US-10.1](#us-101--add-delivery-route) under Acceptance Criteria.

### US-10.2: Edit Delivery Route
**As an** office manager  
**I want to** resequence customer stops, add/remove stops, or change assigned drivers on an existing route  
**So that** routes adapt to changes in customer schedules or driver availability  

* **Priority:** P1  
* **Independent test:** Update stop sequence and assign a backup driver; verify changes persist and update dispatch rosters.  
* **Acceptance scenarios:** see [US-10.2](#us-102--edit-delivery-route) under Acceptance Criteria.

### US-10.3: Delete / Deactivate Delivery Route
**As an** office manager  
**I want to** delete or deactivate an obsolete delivery route  
**So that** decommissioned routes are not assigned to upcoming shipments  

* **Priority:** P1  
* **Independent test:** Attempt deletion of a route with an active in-transit delivery (blocked); deactivate an idle route (succeeds).  
* **Acceptance scenarios:** see [US-10.3](#us-103--delete--deactivate-delivery-route) under Acceptance Criteria.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow office managers to create, view, update, and deactivate delivery routes.
- **FR-002**: Every route MUST be linked to a valid `warehouse_id`.
- **FR-003**: System MUST enforce that `route_code` is unique within each warehouse.
- **FR-004**: System MUST validate that `primary_driver_id` and optional `secondary_driver_id` reference active workers with role `DRIVER`.
- **FR-005**: System MUST maintain normalized customer stops in `route_stops`, enforcing unique stop sequences within a route.
- **FR-006**: System MUST prevent deletion of any route currently assigned to an active, in-transit shipment run.
- **FR-007**: System MUST provide immediate feedback with clear confirmation or inline validation error messages.

---

## Data Ownership & Isolation

Routes and route stops belong to the tenant company and origin warehouse.

| Rule | Requirement |
|---|---|
| **Read scope** | `GET /warehouse/routes` returns routes where `company_id = req.user.companyId` and `warehouse_id = req.user.warehouseId`. |
| **Write scope** | Editing is permitted only for routes matching `req.user.companyId`. |
| **Create scope** | New routes automatically associate with `req.user.companyId` and origin `warehouse_id`. |
| **Cross-user access** | Accessing a route outside the user's tenant/facility returns `404 Not Found`. |
| **UI scope** | Dispatch rosters and driver mobile views only show routes belonging to the active warehouse. |
| **Implementation** | Centralized query helper `getAccessibleRouteOrNull(req, routeId)`. |

---

## API Requirements

| Method | Endpoint | Auth | Role | Purpose |
|---|---|---|---|---|
| `GET` | `/warehouse/routes` | Yes | `office manager`, `company admin`, `driver` | List routes for facility |
| `GET` | `/warehouse/routes/:id` | Yes | `office manager`, `company admin`, `driver` | Get route details and stop sequence |
| `POST` | `/warehouse/routes` | Yes | `office manager`, `company admin` | Create delivery route with stops |
| `PUT` | `/warehouse/routes/:id` | Yes | `office manager`, `company admin` | Update route code, drivers, or stops |
| `DELETE` | `/warehouse/routes/:id` | Yes | `office manager`, `company admin` | Deactivate or delete route |

**Create Request Body (`POST /warehouse/routes`):**
```json
{
  "warehouse_id": 2,
  "route_code": "RTE-AUS-01",
  "route_name": "Austin Metro Morning",
  "primary_driver_id": 50,
  "secondary_driver_id": null,
  "stops": [
    { "customer_id": 5, "stop_sequence": 1, "estimated_transit_minutes": 25 },
    { "customer_id": 8, "stop_sequence": 2, "estimated_transit_minutes": 15 }
  ]
}
```

**Success Response (`201 Created` / `200 OK`):**
```json
{
  "id": 12,
  "company_id": 1,
  "warehouse_id": 2,
  "route_code": "RTE-AUS-01",
  "route_name": "Austin Metro Morning",
  "primary_driver_id": 50,
  "secondary_driver_id": null,
  "status": "ACTIVE",
  "stops": [
    { "id": 1, "customer_id": 5, "stop_sequence": 1, "estimated_transit_minutes": 25 },
    { "id": 2, "customer_id": 8, "stop_sequence": 2, "estimated_transit_minutes": 15 }
  ],
  "created_at": "2026-11-10T12:00:00.000Z"
}
```

**Error Response (`400 Bad Request` / `404 Not Found` / `409 Conflict`):**
```json
{
  "message": "Assigned worker Marcus Johnson does not have the DRIVER role"
}
```

---

## Screen Requirements

### [View: Delivery Routes] — route name `routes`
- **Heading:** **Delivery Routes & Sequences**
- **Warehouse Filter:** `<v-select>` facility selector
- **Primary action:** **+ Create Route** (`oc-cta`) opens dialog with:
  - `route_code`, `route_name`, `primary_driver_id` (filtered by DRIVER role), `secondary_driver_id`
  - Stops Sequencer: Add customer, reorder stops up/down, estimated drive time
  - **Save Route** (`oc-cta`) and **Cancel** buttons
- **Data Table:** `<v-data-table>` displaying columns:
  - Route Code, Name, Primary Driver, Stop Count, Status Chip
  - Row Actions:
    - **View Route** icon (`aria-label="View route"`)
    - **Edit Route** icon (`aria-label="Edit route"`)
    - **Deactivate** icon (`aria-label="Deactivate route"`)
- **Empty state:** **"No delivery routes configured for this warehouse. Create your first delivery route."**
- **Loading state:** Skeleton loader
- **Error state:** `<v-alert type="error">` displaying API error message

---

## Key Entities

- **Route**: Master delivery route definition with route code, route name, originating warehouse, primary driver, optional backup driver, and operational status (`ACTIVE` | `INACTIVE`).
- **Route Stop**: Ordered waypoint on a route linking a specific customer delivery location with a stop sequence number.

---

## Data Model Requirements

### `routes`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique internal route ID |
| `company_id` | INT / BIGINT | FK -> `companies(id)`, NOT NULL | Owning tenant company |
| `warehouse_id` | INT / BIGINT | FK -> `warehouses(id)`, NOT NULL | Origin warehouse facility |
| `route_code` | VARCHAR(30) | NOT NULL | Identifier (e.g., `RTE-NORTH-01`) |
| `route_name` | VARCHAR(100) | NOT NULL | Route description (e.g., `North Austin Retail Circuit`) |
| `primary_driver_id` | INT / BIGINT | FK -> `workers(id)`, NOT NULL | Assigned primary driver |
| `secondary_driver_id`| INT / BIGINT | FK -> `workers(id)`, NULLABLE | Optional backup driver |
| `status` | VARCHAR(20) | NOT NULL, DEFAULT `'ACTIVE'` | Enum: `'ACTIVE'`, `'INACTIVE'` |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp created |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Timestamp updated |

* **Unique Index:** `(warehouse_id, route_code)`

### `route_stops`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INT / BIGINT | PK, AUTO_INCREMENT | Unique stop ID |
| `route_id` | INT / BIGINT | FK -> `routes(id)`, NOT NULL | Parent delivery route |
| `customer_id` | INT / BIGINT | FK -> `customers(id)`, NOT NULL | Destination customer |
| `stop_sequence` | INT | NOT NULL, CHECK (stop_sequence > 0) | Sequential stop order (1, 2, 3...) |
| `estimated_transit_minutes` | INT | NOT NULL, DEFAULT 30, CHECK (estimated_transit_minutes >= 0) | Estimated drive time from prior stop |

* **Unique Index:** `(route_id, stop_sequence)`  
* **Unique Index:** `(route_id, customer_id)`

---

## Acceptance Criteria (Gherkin)

### US-10.1 — Add delivery route

#### Scenario: Successfully create a new delivery route with stops
* **Given** warehouse "DAL-01" exists
* **And** worker "Dave Miller" (ID `50`) exists with role `DRIVER`
* **And** customer "Lone Star Markets" (ID `5`) and customer "Barton Creek Grocers" (ID `8`) exist
* **When** an office manager creates a new route with:
  | Field | Value |
  | `route_code` | "RTE-AUS-01" |
  | `route_name` | "Austin Metro Morning" |
  | `primary_driver_id` | 50 |
* **And** adds Stop 1: Customer ID `5`
* **And** adds Stop 2: Customer ID `8`
* **And** submits the form
* **Then** the system creates the route and 2 sequenced route stops
* **And** displays confirmation "Route RTE-AUS-01 created successfully with 2 stops"

#### Scenario: Attempt to assign worker without DRIVER role as primary driver
* **Given** worker "Marcus Johnson" has role `PICKER`
* **When** the office manager attempts to create a route assigning Marcus Johnson as `primary_driver_id`
* **Then** the system rejects the submission
* **And** displays an error message "Assigned worker Marcus Johnson does not have the DRIVER role"

#### Scenario: Attempt to create route with duplicate route code in same warehouse
* **Given** route "RTE-AUS-01" already exists for warehouse "DAL-01"
* **When** the office manager submits another route with code "RTE-AUS-01" for warehouse "DAL-01"
* **Then** the system rejects the submission
* **And** displays an error message "Route code RTE-AUS-01 already exists in this warehouse"

---

### US-10.2 — Edit delivery route

#### Scenario: Successfully reorder stop sequences on an existing route
* **Given** route "RTE-AUS-01" has Stop 1 (Customer 5) and Stop 2 (Customer 8)
* **When** the office manager changes Customer 8 to Stop 1 and Customer 5 to Stop 2
* **And** saves changes
* **Then** the system updates `stop_sequence` values in the database
* **And** displays confirmation "Route RTE-AUS-01 stop sequence updated successfully"

#### Scenario: Attempt to update route assigning an inactive driver
* **Given** worker "John Doe" (ID `55`) is in status `"INACTIVE"`
* **When** the office manager edits route "RTE-AUS-01" and changes primary driver to ID `55`
* **And** submits the form
* **Then** the system rejects the update
* **And** displays an error message "Cannot assign driver: Worker John Doe is inactive"
* **And** the primary driver assignment remains unchanged

---

### US-10.3 — Delete / Deactivate delivery route

#### Scenario: Successfully deactivate an active route
* **Given** route "RTE-AUS-01" has no shipments currently out for delivery
* **When** the office manager selects "Deactivate Route" and confirms
* **Then** the system sets status to `"INACTIVE"`
* **And** displays confirmation "Route RTE-AUS-01 has been deactivated"
* **And** the route is excluded from new dispatch schedules

#### Scenario: Prevent deletion when route has active in-transit shipments
* **Given** route "RTE-AUS-01" has 3 customer orders currently in status `"SHIPPED"` / out for delivery
* **When** the office manager attempts to delete route "RTE-AUS-01"
* **Then** the system rejects the deletion
* **And** displays an error message "Cannot delete route: 3 shipments are currently in transit on this route"

---

## Definition of Done

* [ ] Backend and frontend implemented per this spec (**FR-001** – **FR-007** satisfied)
* [ ] Database tables `routes` and `route_stops` created with driver role verification
* [ ] API endpoints `/warehouse/routes` enforce tenant scoping and sequence uniqueness
* [ ] Gherkin acceptance criteria verified by automated integration tests
* [ ] Cross-tenant access returns `404 Not Found`

---

## Out of Scope

* Real-time GPS truck telematics tracking
* Turn-by-turn navigation mapping integrations (Google Maps / Mapbox)
* Bill of Lading generation and customer delivery confirmation ([Feature 16 — Ship and Deliver Customer Orders](feature-16-ship-and-deliver-customer-orders.md))
