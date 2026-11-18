# ADR-0008: Sequelize ORM for Relational Models & Transactions

**Status:** Accepted  
**Date:** 2026-11-10  
**Deciders:** Warehouse Engineering Team  

## Context

Managing raw SQL queries across 20+ tables, handling migrations, managing foreign key relationships, and ensuring SQL injection safety requires an Object-Relational Mapper (ORM).

## Decision

We adopt **Sequelize ORM** (v6) with the `mysql2` dialect:
1. **Model Definition:** Define ES module models in `backend/app/models/` inheriting from central `sequelizeInstance.js`.
2. **Associations:** Configure explicit associations (`hasMany`, `belongsTo`) in `models/index.js` mirroring the ERD.
3. **Transactions:** Use managed transactions (`sequelize.transaction(async (t) => { ... })`) for multi-entity operations (e.g., dock receiving receipt line insertion + purchase order status update + staging balance update).
4. **Default Scopes:** Sensitive fields (e.g., worker password hashes) are omitted by default.

## Consequences

### Positive
- Type and constraint validation before SQL execution.
- Automated parameterized queries protect against SQL injection vulnerabilities.
- Declarative relation queries via `include` reduce boilerplate JOIN queries.

### Negative / Tradeoffs
- Complex reporting queries (e.g. multi-dimensional inventory valuation) may occasionally require raw SQL queries (`sequelize.query`) for performance.

## Related Artifacts
- [ADR-0003: MySQL Relational Database](./0003-mysql-relational-database.md)
- [`schema.sql`](../../schema.sql)

