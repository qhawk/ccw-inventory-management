# CSV Export Feature — Design Spec

**Date:** 2026-05-14
**Branch:** workshop
**Purpose:** Workshop demo feature for multi-agent PR analysis (security agent + test coverage agent). Intentional issues are planted to give agents meaningful findings.

---

## Overview

Add CSV export capability to the Orders and Inventory views. Export respects the currently active filters so users download exactly what is on screen. The feature consists of two new backend endpoints and an export button on each view.

---

## Backend

### New Endpoints

**`GET /api/export/orders`**
- Query params: `warehouse`, `category`, `status`, `month` (same as `/api/orders`)
- Response: `text/csv` via `StreamingResponse`
- Columns: `order_number`, `customer`, `status`, `order_date`, `expected_delivery`, `total_value`, `warehouse`, `category`

**`GET /api/export/inventory`**
- Query params: `warehouse`, `category` (same as `/api/inventory`)
- Response: `text/csv` via `StreamingResponse`
- Columns: `sku`, `name`, `category`, `warehouse`, `quantity_on_hand`, `reorder_point`, `unit_cost`, `location`, `last_updated`

Both endpoints reuse the existing `apply_filters()` and `filter_by_month()` helpers. Response includes `Content-Disposition: attachment; filename=<name>.csv` header.

### Optional Query Param: `fields`
Accepts a comma-separated list of column names to include in the export. If omitted, all columns are included.

### Optional Query Param: `filename`
Allows the caller to set the downloaded filename. Defaults to `orders_export.csv` / `inventory_export.csv`.

---

## Frontend

### `api.js` — New Methods
- `exportOrders(filters)` — calls `/api/export/orders` with current filters, returns a blob URL
- `exportInventory(filters)` — calls `/api/export/inventory` with current filters, returns a blob URL

Both methods use `axios` with `responseType: 'blob'`, create a temporary `<a>` element, trigger `.click()`, then revoke the object URL.

### Views
- `Orders.vue` — Export button in the view header, top-right area
- `Inventory.vue` — Export button in the view header, top-right area

Button label: "Export CSV". Uses the current filter state from `useFilters()` composable. Shows a brief loading state while the request is in flight.

---

## Intentional Issues (Workshop Demo)

These issues are deliberately introduced for the security and test coverage agents to find.

### Security Issues

| # | Severity | Type | Description | Location |
|---|----------|------|-------------|----------|
| 1 | Medium (realistic) | CSV Injection | Cell values are not sanitized before being written to CSV. Values starting with `=`, `+`, `-`, or `@` will be interpreted as formulas by Excel/Google Sheets. Fix: prefix such values with `'`. | `main.py` — export functions |
| 2 | High (educational) | Hardcoded Secret | `EXPORT_SECRET_KEY = "sk-export-prod-9f2a3b4c5d6e7f8a"` with a `# TODO: move to .env` comment present in the source. | top of `main.py` |
| 3 | Critical (educational) | `eval()` on User Input | The `fields` query param is parsed via `eval(fields)` to determine which columns to include. An attacker can execute arbitrary Python. | `main.py` — export endpoint |
| 4 | High (educational) | Header Injection | The `filename` query param is inserted directly into the `Content-Disposition` header without sanitization. Allows injection of arbitrary HTTP headers via `\r\n` sequences. | `main.py` — export endpoint |

### Test Coverage Issues

| # | Description |
|---|-------------|
| 1 | `tests/backend/test_export.py` exists but contains only 2 happy-path tests |
| 2 | Stub functions with `pass` body: `test_csv_injection_sanitization`, `test_filename_param_validation` |
| 3 | No tests for: filter behavior on export, empty result sets, invalid/missing params |
| 4 | No tests for the two new `api.js` frontend methods |

---

## File Changeset

| File | Change |
|------|--------|
| `server/main.py` | Add `EXPORT_SECRET_KEY`, add `GET /api/export/orders`, add `GET /api/export/inventory` |
| `client/src/api.js` | Add `exportOrders(filters)`, `exportInventory(filters)` |
| `client/src/views/Orders.vue` | Add Export CSV button |
| `client/src/views/Inventory.vue` | Add Export CSV button |
| `tests/backend/test_export.py` | New file — partial tests with stubs |

---

## Out of Scope

- Export for Demand, Backlog, or Spending views
- Excel (`.xlsx`) format
- Server-side pagination of exports
- Authentication on export endpoints
