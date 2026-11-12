# Architecture Decision Records (ADRs)

Durable **why** decisions for cross-cutting architectural choices that outlive any single feature specification in the Warehouse Management System (WMS).

| Artifact | Question |
|----------|----------|
| **ADRs** (`docs/adr/`) | Why did we choose this approach across the system? |
| **Feature specs** (`features/`) | What must the product do, and how is each feature designed? |
| **Quality attributes** (`docs/nfr/`) | What app-wide NFR and quality bars apply? |

---

## When to write an ADR

- Significant stack or architecture choice
- Security, multi-tenant data isolation, or authorization models
- Database engine, persistence, or transactional consistency patterns
- Frontend, UI styling, and mobile scanner interaction patterns

---

## Index

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| [0001](./0001-client-server-multi-tenant-architecture.md) | Client–Server Multi-Tenant Architecture | Accepted | 2026-11-10 |
| [0002](./0002-security-and-data-isolation.md) | Layered Security & Multi-Tenant Data Isolation | Accepted | 2026-11-10 |
| [0003](./0003-mysql-relational-database.md) | MySQL Relational Database for Warehouse Transactions | Accepted | 2026-11-10 |
| [0004](./0004-vue-frontend-framework.md) | Vue 3 as Frontend SPA Framework | Accepted | 2026-11-10 |
| [0005](./0005-vuetify-ui-library.md) | Vuetify Component Library & Warehouse Mobile UI | Accepted | 2026-11-10 |
| [0006](./0006-node-express-api.md) | Node.js and Express API Runtime | Accepted | 2026-11-10 |
| [0007](./0007-rest-json-api.md) | RESTful API Conventions & Flat JSON Structure | Accepted | 2026-11-10 |
| [0008](./0008-sequelize-orm.md) | Sequelize ORM for Relational Models & Transactions | Accepted | 2026-11-10 |

