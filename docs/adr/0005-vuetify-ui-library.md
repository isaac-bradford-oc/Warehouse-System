# ADR-0005: Vuetify Component Library & Warehouse Mobile UI

**Status:** Accepted  
**Date:** 2026-11-10  
**Deciders:** Warehouse Engineering Team  

## Context

WMS user interfaces span two distinct form factors:
1. **Desktop Administration:** Dense data tables, complex search filters, master data CRUD dialogs, and reports (office managers and system administrators).
2. **Mobile / Rugged Tablet Floor Views:** Large touch targets, barcode scanner inputs, step-by-step pick cards, and delivery confirmation dialogs (stockers, pickers, drivers).

Building custom UI widgets from scratch for both form factors is error-prone and time-consuming.

## Decision

We adopt **Vuetify** as the primary component library:
1. **Component Suite:** Use `<v-data-table>`, `<v-dialog>`, `<v-card>`, `<v-btn>`, `<v-text-field>`, `<v-alert>`, and `<v-chip>`.
2. **Styling Rules:** Adopt the `oc-cta` CSS utility class for all primary actionable call-to-actions.
3. **Accessibility & Feedback:** All icon-only actions must provide explicit `aria-label`s. Error states render via `<v-alert type="error">` and empty states provide descriptive textual copy.

## Consequences

### Positive
- Accelerated frontend development with pre-built accessible components.
- Responsive grid system handles both wide monitors and mobile touch devices.
- Consistent theming, alert styling, and form validation across all 16 features.

### Negative / Tradeoffs
- Must adhere strictly to Vuetify layout rules (e.g., avoiding labeled action buttons stuffed inside card title slots).

## Related Artifacts
- [ADR-0004: Vue 3 Frontend Framework](./0004-vue-frontend-framework.md)

