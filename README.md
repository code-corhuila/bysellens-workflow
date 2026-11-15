# bysellens-workflow

> Business process orchestration (saga)

Part of the **LMS Library** distributed system — team `lms-library`, Grupo 2.
Governance and documentation live in [`library-docs`](https://github.com/code-corhuila/library-docs).

## Branching

Three permanent branches. **None of them accepts a direct commit** — you enter through a child
branch and leave through a Pull Request.

```
develop  <--PR--  feat/... fix/... chore/...
qa       <--PR--  qa/...
main     <--PR--  release/...  hotfix/...
```

Promotion happens **by re-application** (`git cherry-pick -x`), never by merging one permanent
branch into another: `merge develop -> qa` and `merge qa -> main` do not exist in this model.

`main` requires **1 approval from `ariel5253`**. On `develop` and `qa` the team sets its own review
rule.

Full policy: `00-governance/branching-policy.md` in `library-docs`.

## BY_SELLENS Workflow Components

### Purpose

This repository includes the existing BY_SELLENS sales and inventory service logic that
coordinates a sale with inventory reservation. It is source code from the backend, not a
standalone workflow engine or deployable application.

### Source Code

The Java files are copied from the BY_SELLENS backend with their original package declarations
and contents:

- `sales-service/src/main/java/com/bysellens/sales/SalesService.java`
- `inventory-service/src/main/java/com/bysellens/inventory/InventoryService.java`

### Included Components

- `SalesService` looks up and validates customer and product data, creates idempotent sales,
  coordinates inventory reservation, records confirmation events in the outbox when needed,
  and periodically attempts recovery of pending sales.
- `InventoryService` initializes and adjusts stock, reserves inventory, reports reservation
  status, confirms or cancels reservations, and returns inventory views and movement history.

### Sales and Inventory Coordination

`SalesService` uses the existing `Remote` client and configured customer, product, and inventory
URLs. It checks customer and product availability, calculates sale details and totals, then asks
inventory to reserve the sale's items. A confirmed reservation confirms the sale directly; a
reserved response results in a confirmation event being inserted into `rabbit_outbox`. The
service also handles selected inventory conflict/not-found responses and schedules recovery of
pending sales every 30 seconds after an initial 30-second delay.

### Stock Reservation and Confirmation

`InventoryService` sorts reservation lines, rejects duplicate products, locks the reservation
and stock records, checks available stock, and records reservation movements. A transition from
`RESERVADA` to `CONFIRMADA` releases the reserved quantity and deducts it from stock; a transition
to `CANCELADA` releases the reservation without deducting stock. Repeated transitions to the
current target state are idempotent.

### Dependencies

The source imports Spring service, transaction, scheduling, and JDBC APIs, Jackson, and Java
standard-library classes. It also depends on the original application's `Remote` client, common
domain types, controllers, repositories, database schema, and configured service URLs, which
are not included here.

### Execution Status

This repository contains source components only. No standalone application execution, deployment,
or tests are claimed by this README.

### Important Notes

The Java logic and package declarations are preserved from the BY_SELLENS backend. These files
are not independent microservices. Message publication and processing require the existing
backend infrastructure and the corresponding worker components. The existing LMS Library and
branching information above is retained.
