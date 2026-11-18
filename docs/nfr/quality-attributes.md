# Quality Attributes

App-wide non-functional requirements and quality targets for the Warehouse Management System (WMS).

## Quality Attributes Table

| Attribute | Target | Approach | How we verify | Status | Links |
|---|---|---|---|---|---|
| **Security** | **100%** of warehouse routes require authentication; **0** cross-tenant data leaks; access to other companies' or out-of-scope facilities returns **404** (not 403) | Multi-tenant scoping helper in middleware; Bearer token authentication; role guards | Automated Jest integration tests with cross-tenant fixtures | Accepted | [ADR-0002](../adr/0002-security-and-data-isolation.md) |
| **Data Integrity** | **100%** of inventory balance mutations are atomic; **0** orphaned child rows; constraint violations rejected at DB engine | MySQL InnoDB transactions; explicit foreign key constraints; Sequelize transactions | Automated Supertest database integration tests; strict foreign key tests | Accepted | [ADR-0003](../adr/0003-mysql-relational-database.md), [ADR-0008](../adr/0008-sequelize-orm.md) |
| **Performance** | p95 API response time **< 200 ms** for standard CRUD; barcode scan verification **< 150 ms**; dashboard initial load **< 2.0 s** | Indexed foreign keys & codes; lean flat JSON payloads; lightweight Vue 3 SPA | Load test scripts (k6) & browser devtools network audit | Deferred | [ADR-0006](../adr/0006-node-express-api.md), [ADR-0007](../adr/0007-rest-json-api.md) |
| **Reliability** | Write operation success rate **≥ 99.5%** during active operations; failed transactions roll back completely | Managed Sequelize transactions; validation prior to persistence | Automated integration tests testing rollback on failure | Accepted | [ADR-0008](../adr/0008-sequelize-orm.md) |
| **Availability** | System uptime goal **≥ 99.0%** during active warehouse shift hours | Standard Node process monitoring; restart-on-failure supervisor | Process supervisor health-check endpoint (`GET /warehouse/health`) | Deferred | [ADR-0001](../adr/0001-client-server-multi-tenant-architecture.md) |
| **Observability** | **100%** of unhandled server exceptions logged with stack trace; operational discrepancy alerts logged | Winston logger with daily file rotation + console stream; audit log tables | Automated inspection of log files and exception report tables | Accepted (minimal) | [Feature 11 — Maintain Reports](../../features/feature-11-maintain-reports.md) |
| **Usability & Ergonomics** | Mobile floor screens touch-friendly (buttons $\ge$ 48px); barcode input auto-focused; **100%** match on CTA labels | Vuetify 4 components; `oc-cta` styling; mobile-responsive cards; clear sound/color alerts on barcode mismatch | Manual handheld testing; Vitest component rendering tests | Accepted | [ADR-0005](../adr/0005-vuetify-ui-library.md) |
| **Accessibility (a11y)** | Keyboard navigable desktop admin tables; **0** unlabeled icon buttons | Standard HTML semantics; `aria-label`s on all action icons | Automated Vitest / axe-core audits | Deferred | [ADR-0005](../adr/0005-vuetify-ui-library.md) |
| **Maintainability** | **100%** of feature behaviors traceable to feature design specs; clean directory structure | Spec-driven development; decoupled Vue frontend and Express backend | Definition of Done checklists on every feature branch | Accepted | Feature Specs & Definition of Done |

---

## Feature-Local NFRs

For requirements specific to a single feature (such as min-max reorder calculation accuracy, dock check-in speed, or delivery signature capture), requirements are specified directly in that feature specification under **Functional Requirements (`**FR-00N**`)**, **Data Model**, and **Gherkin Acceptance Criteria**.

