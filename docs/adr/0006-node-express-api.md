# ADR-0006: Node.js and Express API Runtime

**Status:** Accepted  
**Date:** 2026-11-10  
**Deciders:** Warehouse Engineering Team  

## Context

The backend service coordinates HTTP requests, data validation, database transactions, and background triggers (e.g. min-max replenishment). We need a lightweight, standard runtime with robust ecosystem support matching the OC curriculum.

## Decision

We adopt **Node.js** (LTS) with **Express 4+** using modern **ECMAScript Modules (`"type": "module"`)**:
1. All routes are mounted under the standardized application prefix: `/warehouse/`.
2. Router files are organized by domain resource (e.g., `company.routes.js`, `order.routes.js`, `inventory.routes.js`).
3. Centralized error handling and structured logging via Winston + Morgan.
4. Export `app` for automated integration testing using Supertest.

## Consequences

### Positive
- Unified JavaScript/ES syntax across full stack.
- Highly extensible middleware pipeline for authentication, tenant scoping, and role authorization.
- Fast execution and low resource footprint.

### Negative / Tradeoffs
- Single-threaded event loop requires careful handling of heavy queries (e.g., aggregating annual inventory reports).

## Related Artifacts
- [ADR-0007: REST JSON API](./0007-rest-json-api.md)
- [ADR-0008: Sequelize ORM](./0008-sequelize-orm.md)

