# ADR-0002: Layered Security & Multi-Tenant Data Isolation

**Status:** Accepted  
**Date:** 2026-11-10  
**Deciders:** Warehouse Engineering Team  

## Context

The WMS is a multi-tenant platform where multiple companies maintain proprietary supplier contracts, customer lists, item catalogs, and stock counts. Within each company, workers have distinct operational roles (`admin`, `company admin`, `office manager`, `receiving clerk`, `stocker`, `picker`, `driver`).

We must guarantee:
1. Complete tenant isolation (Company A must never view or tamper with Company B's data).
2. Facility-level scoping (Workers at Warehouse 1 only execute operations relevant to Warehouse 1 unless company-wide administrative scope is granted).
3. Role-based privilege enforcement.
4. Information disclosure prevention (an unauthorized attempt to view an out-of-scope resource must not reveal the existence of that resource).

## Decision

We implement **Layered Multi-Tenant Security & Isolation**:
1. **Authentication:** Bearer token / session authentication via JWT tokens containing authenticated `userId`, `companyId`, `warehouseId`, and `role`.
2. **Tenant Scoping:** All queries automatically filter by `company_id = req.user.companyId` and, where applicable, `warehouse_id = req.user.warehouseId`.
3. **Cross-Tenant Concealment (404 Not 403):** If a user attempts to access a resource with an ID that belongs to another company or out-of-scope warehouse, the API returns `404 Not Found` (never `403 Forbidden`), preventing resource enumeration attacks.
4. **Role Guards:** Route-level middleware (`requireRole(['OFFICE_MANAGER', 'COMPANY_ADMIN'])`) restricts administrative functions.

## Consequences

### Positive
- Strict data privacy between competing companies using the shared system.
- Zero reliance on client-side filtering; server enforces 100% of data isolation boundaries.
- Consistent standard error responses across all controllers.

### Negative / Tradeoffs
- Every controller query must consistently include company/warehouse scoping clauses or use an authorization helper.
- System Administrators (`admin`) require a specialized bypass to manage global company tenant records.

## Related Artifacts
- [ADR-0001: Client-Server Multi-Tenant Architecture](./0001-client-server-multi-tenant-architecture.md)
- [ADR-0007: REST JSON API](./0007-rest-json-api.md)

