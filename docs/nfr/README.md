# Non-Functional Requirements (NFR) & Quality Attributes

Living repository of system quality characteristics (“ilities”) for the Warehouse Management System (WMS): security, performance, data integrity, availability, observability, usability, and maintainability.

| Artifact | Question |
|----------|----------|
| [Feature specs](../../features/) | What must the product do, and how is each feature designed? |
| **This folder** (`docs/nfr/`) | What quality bars apply across the entire warehouse system? |
| [ADRs](../adr/README.md) | Why did we choose a particular architectural approach for a quality? |

---

## Files

| File | Contents |
|------|----------|
| [quality-attributes.md](./quality-attributes.md) | Quality attributes table with Status, Targets, Approaches, Verification, and Links |

---

## Status Meanings

- **Accepted:** Active architectural law; must not regress in automated tests or production builds.
- **Accepted (minimal):** Baseline enforcement in place.
- **Deferred:** Documented target for future optimization / load-testing gates.
- **Out of scope:** Explicit non-goals for this system version.

