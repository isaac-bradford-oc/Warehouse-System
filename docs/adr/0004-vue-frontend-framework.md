# ADR-0004: Vue 3 as Frontend SPA Framework

**Status:** Accepted  
**Date:** 2026-11-10  
**Deciders:** Warehouse Engineering Team  

## Context

The warehouse frontend requires high responsiveness for high-frequency actions: barcode scanning at receiving dock, pick cart item verification, and real-time form validations. A traditional multi-page application (MPA) would incur disruptive full-page reloads that slow down handheld operations.

## Decision

We adopt **Vue 3** (using the Composition API `<script setup>` syntax) bundled with **Vite**:
1. Single Page Application (SPA) with client-side routing via `vue-router`.
2. Fast local compilation and Hot Module Replacement (HMR) via Vite.
3. Centralized HTTP API services using Axios (`*Services.js` pattern).
4. Full architectural alignment with the OC Specs Kit frontend conventions.

## Consequences

### Positive
- Sub-second UI updates upon barcode scan confirmation without page refresh.
- Declarative component-based architecture for warehouse tables, inventory counters, and modal dialogs.
- Clean separation between presentation components and API services.

### Negative / Tradeoffs
- Client must securely manage authentication tokens in storage.
- Initial JavaScript bundle download required upon first load.

## Related Artifacts
- [ADR-0005: Vuetify UI Library](./0005-vuetify-ui-library.md)
- [ADR-0001: Client-Server Multi-Tenant Architecture](./0001-client-server-multi-tenant-architecture.md)

