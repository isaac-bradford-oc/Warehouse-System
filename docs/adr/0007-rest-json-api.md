# ADR-0007: RESTful API Conventions & Flat JSON Structure

**Status:** Accepted  
**Date:** 2026-11-10  
**Deciders:** Warehouse Engineering Team  

## Context

Frontend services and automated test suites require predictable HTTP semantics, endpoint paths, status codes, and payload formats. Arbitrary response wrappers or inconsistent HTTP verbs cause frontend bugs and complicate verification.

## Decision

We establish strict **RESTful and JSON Conventions** across all endpoints under `/warehouse/`:
1. **HTTP Verbs:**
   - `GET`: Retrieve resource(s).
   - `POST`: Create a new resource or execute a state transition action.
   - `PUT`: Idempotently update an existing resource.
   - `DELETE`: Remove or deactivate a resource.
2. **Flat JSON Payloads:** Return raw entity JSON objects or arrays. Do NOT wrap in `{ success: true, data: { ... } }` envelopes.
3. **HTTP Status Codes:**
   - `200 OK`: Successful read or update.
   - `201 Created`: Successful creation.
   - `204 No Content`: Successful deletion with empty response.
   - `400 Bad Request`: Input validation failure, malformed payload, or domain constraint violation.
   - `401 Unauthorized`: Missing or invalid session/auth token.
   - `404 Not Found`: Resource does not exist, belongs to another tenant, or is outside worker's warehouse scope.
   - `409 Conflict`: Unique constraint violation (e.g. duplicate SKU or code).
4. **Standard Error Body:** `{ "message": "Descriptive human-readable explanation." }`.

## Consequences

### Positive
- Client code can inspect native HTTP response status codes directly without parsing body envelopes.
- Consistency across all 16 features.
- Seamless compatibility with standard testing tools (Supertest, Axios interceptors).

### Negative / Tradeoffs
- Custom error metadata must be structured cleanly within the message string or supplementary fields when needed.

## Related Artifacts
- [ADR-0002: Security & Data Isolation](./0002-security-and-data-isolation.md)
- [ADR-0006: Node.js and Express API Runtime](./0006-node-express-api.md)

