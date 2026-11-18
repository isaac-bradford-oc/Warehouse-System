# ADR-0003: MySQL Relational Database for Warehouse Transactions

**Status:** Accepted  
**Date:** 2026-11-10  
**Deciders:** Warehouse Engineering Team  

## Context

Warehouse operations depend fundamentally on transactional ACID guarantees:
- Incrementing bin inventory during putaway must atomically decrease dock staging and update total on-hand quantity.
- Decrementing bin inventory during picking must atomically clear order reservations.
- Duplicate SKU codes, barcode UPCs, and order numbers must be physically blocked at the database storage engine level.
- Multi-table relationships (Companies $\rightarrow$ Warehouses $\rightarrow$ Slots $\rightarrow$ Bins $\rightarrow$ Items) require strict foreign key constraints.

## Decision

We select **MySQL 8.0+** using the **InnoDB storage engine** as the primary relational database:
1. Enforces strict foreign key constraints (`ON DELETE RESTRICT` for master entities).
2. Supports transactional semantics (`BEGIN`, `COMMIT`, `ROLLBACK`) across multi-step inventory adjustments.
3. Provides compound unique indexes (`company_id, sku`, `warehouse_id, slot_code`, `slot_id, bin_code`).
4. Aligns with the OC Specs Kit standard environment (XAMPP / MySQL).

## Consequences

### Positive
- Zero risk of orphan records or partial stock balance updates during network disconnects.
- Strong schema enforcement prevents corrupt data entry.
- Direct alignment with course infrastructure and tooling.

### Negative / Tradeoffs
- Schema changes require structured migration scripts rather than dynamic schema mutations.
- Database connections must be pooled and managed efficiently in the Node.js backend.

## Related Artifacts
- [ADR-0008: Sequelize ORM](./0008-sequelize-orm.md)
- [`schema.sql`](../../schema.sql)

