# ADR-0001: Client–Server Multi-Tenant Architecture

**Status:** Accepted  
**Date:** 2026-11-10  
**Deciders:** Warehouse Engineering Team  

## Context

The Warehouse Management System (WMS) supports multiple tenant companies operating physical warehouse facilities, managing stock across aisles/slots/bins, fulfilling supplier and customer orders, and dispatching delivery routes. The system must support administrative desktops (office managers, system admins) and mobile/tablet devices on the warehouse floor and in delivery vehicles (dock receiving clerks, stockers, pickers, drivers).

We need an architectural model that cleanly separates presentation from business logic, ensures data integrity, and scales across multiple tenant organizations without server-side template tight coupling.

## Decision

We adopt a **decoupled Client–Server Single-Page Application (SPA) architecture**:
1. **Frontend:** A standalone Vue 3 SPA communicating exclusively via HTTP/HTTPS REST APIs.
2. **Backend:** A stateless Node.js / Express service providing centralized business logic, transaction handling, barcode scan validation, and relational persistence.
3. **Multi-Tenancy:** Tenant isolation is enforced at the software layer (`company_id` on all tenant-owned entities; `warehouse_id` on all facility-scoped operations).

## Consequences

### Positive
- Strict separation of concerns: Backend is the single authoritative source of truth for inventory balances and order state transitions.
- Client flexibility: The same REST API powers desktop administration, handheld barcode scanner views, and driver mobile web interfaces.
- Independent testability: Backend endpoints are testable via Supertest without DOM dependency; frontend components are testable via Vitest and mocked HTTP responses.

### Negative / Tradeoffs
- Requires CORS management and synchronized API contract management between frontend and backend.
- Network latency over warehouse Wi-Fi/cellular must be handled gracefully with responsive loading states and offline error alerts.

## Related Artifacts
- [ADR-0002: Security & Data Isolation](./0002-security-and-data-isolation.md)
- [ADR-0006: Node.js and Express API Runtime](./0006-node-express-api.md)
- [ADR-0007: REST JSON API](./0007-rest-json-api.md)

