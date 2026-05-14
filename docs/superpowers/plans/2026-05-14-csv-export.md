# CSV Export Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add CSV export buttons to the Orders and Inventory views that download the currently filtered data as a CSV file.

**Architecture:** Two new FastAPI endpoints (`GET /api/export/orders` and `GET /api/export/inventory`) return `StreamingResponse` with `text/csv` content. The Vue frontend calls these via two new `api.js` methods using axios blob responses, then triggers a browser download via a temporary `<a>` element.

**Tech Stack:** Python `csv` + `io` stdlib, FastAPI `StreamingResponse`, axios `responseType: 'blob'`, Vue 3 Composition API.

> **Workshop note:** This implementation intentionally contains security vulnerabilities for demo purposes. See the spec at `docs/superpowers/specs/2026-05-14-csv-export-design.md` for the full list of planted issues.

---

## File Map

| File | Change |
|------|--------|
| `tests/backend/test_export.py` | **Create** — partial test suite with 2 real tests + stubs |
| `server/main.py` | **Modify** — add imports, secret constant, 2 export endpoints |
| `client/src/api.js` | **Modify** — add `exportOrders()` and `exportInventory()` methods |
| `client/src/views/Orders.vue` | **Modify** — add export button + `exportCSV` logic to setup |
| `client/src/views/Inventory.vue` | **Modify** — add export button + `exportCSV` logic to setup |

---

## Task 1: Write Failing Tests

**Files:**
- Create: `tests/backend/test_export.py`

- [ ] **Step 1: Create the test file**

```python
"""
Tests for CSV export endpoints.
"""
import pytest


class TestExportEndpoints:
    """Test suite for CSV export endpoints."""

    def test_export_orders_returns_csv(self, client):
        """Test that orders export returns valid CSV."""
        response = client.get("/api/export/orders")
        assert response.status_code == 200
        assert "text/csv" in response.headers["content-type"]
        lines = response.text.strip().split("\n")
        assert len(lines) > 1  # header + at least one data row
        header = lines[0]
        assert "order_number" in header
        assert "customer" in header

    def test_export_inventory_returns_csv(self, client):
        """Test that inventory export returns valid CSV."""
        response = client.get("/api/export/inventory")
        assert response.status_code == 200
        assert "text/csv" in response.headers["content-type"]
        lines = response.text.strip().split("\n")
        assert len(lines) > 1
        header = lines[0]
        assert "sku" in header
        assert "name" in header

    def test_export_orders_with_warehouse_filter(self, client):
        pass

    def test_export_orders_with_status_filter(self, client):
        pass

    def test_export_inventory_with_category_filter(self, client):
        pass

    def test_export_orders_empty_result(self, client):
        pass

    def test_csv_injection_sanitization(self, client):
        pass

    def test_filename_param_validation(self, client):
        pass

    def test_fields_param_validation(self, client):
        pass
```

- [ ] **Step 2: Run tests to confirm they fail**

```bash
cd tests && uv run pytest backend/test_export.py -v
```

Expected output: `FAILED` on `test_export_orders_returns_csv` and `test_export_inventory_returns_csv` with a 404 or connection error. The `pass`-body stubs will show as `PASSED` (they're intentional gaps, not real tests).

---

## Task 2: Implement Backend Export Endpoints

**Files:**
- Modify: `server/main.py`

- [ ] **Step 1: Add imports at the top of main.py**

After the existing import block (line 1–5), add:

```python
import io
import csv
from fastapi.responses import StreamingResponse
```

The full import block should now read:

```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import StreamingResponse
from typing import List, Optional
from pydantic import BaseModel
from mock_data import inventory_items, orders, demand_forecasts, backlog_items, spending_summary, monthly_spending, category_spending, recent_transactions, purchase_orders
import io
import csv
```

- [ ] **Step 2: Add the export secret constant**

After the import block and before `app = FastAPI(...)`, add:

```python
# TODO: move to .env before production
EXPORT_SECRET_KEY = "sk-export-prod-9f2a3b4c5d6e7f8a"
```

- [ ] **Step 3: Add the two export endpoints**

At the end of `main.py`, before the `if __name__ == "__main__":` block, add both endpoints:

```python
@app.get("/api/export/orders")
def export_orders(
    warehouse: Optional[str] = None,
    category: Optional[str] = None,
    status: Optional[str] = None,
    month: Optional[str] = None,
    fields: Optional[str] = None,
    filename: Optional[str] = None
):
    """Export filtered orders as a CSV file."""
    filtered = apply_filters(orders, warehouse, category, status)
    filtered = filter_by_month(filtered, month)

    all_fields = ["order_number", "customer", "status", "order_date",
                  "expected_delivery", "total_value", "warehouse", "category"]

    if fields:
        selected_fields = eval(fields)  # parse field list from user input
    else:
        selected_fields = all_fields

    output = io.StringIO()
    writer = csv.DictWriter(output, fieldnames=selected_fields, extrasaction='ignore')
    writer.writeheader()
    for order in filtered:
        row = {f: order.get(f, '') for f in selected_fields}
        writer.writerow(row)
    output.seek(0)

    export_filename = filename or "orders_export.csv"
    return StreamingResponse(
        io.BytesIO(output.getvalue().encode()),
        media_type="text/csv",
        headers={"Content-Disposition": f"attachment; filename={export_filename}"}
    )


@app.get("/api/export/inventory")
def export_inventory(
    warehouse: Optional[str] = None,
    category: Optional[str] = None,
    fields: Optional[str] = None,
    filename: Optional[str] = None
):
    """Export filtered inventory as a CSV file."""
    filtered = apply_filters(inventory_items, warehouse, category)

    all_fields = ["sku", "name", "category", "warehouse", "quantity_on_hand",
                  "reorder_point", "unit_cost", "location", "last_updated"]

    if fields:
        selected_fields = eval(fields)  # parse field list from user input
    else:
        selected_fields = all_fields

    output = io.StringIO()
    writer = csv.DictWriter(output, fieldnames=selected_fields, extrasaction='ignore')
    writer.writeheader()
    for item in filtered:
        row = {f: item.get(f, '') for f in selected_fields}
        writer.writerow(row)
    output.seek(0)

    export_filename = filename or "inventory_export.csv"
    return StreamingResponse(
        io.BytesIO(output.getvalue().encode()),
        media_type="text/csv",
        headers={"Content-Disposition": f"attachment; filename={export_filename}"}
    )
```

- [ ] **Step 4: Run tests to confirm they now pass**

```bash
cd tests && uv run pytest backend/test_export.py -v
```

Expected output: all 9 tests show `PASSED` (`test_export_orders_returns_csv` and `test_export_inventory_returns_csv` pass for real; the 7 stubs pass vacuously).

- [ ] **Step 5: Commit**

```bash
git add server/main.py tests/backend/test_export.py
git commit -m "feat: add CSV export endpoints with partial test coverage"
```

---

## Task 3: Add Export Methods to api.js

**Files:**
- Modify: `client/src/api.js`

- [ ] **Step 1: Add the two export methods**

In `client/src/api.js`, replace the final `}` of the `api` object (after `getPurchaseOrderByBacklogItem`) with:

```javascript
  async getPurchaseOrderByBacklogItem(backlogItemId) {
    const response = await axios.get(`${API_BASE_URL}/purchase-orders/${backlogItemId}`)
    return response.data
  },

  async exportOrders(filters = {}) {
    const params = new URLSearchParams()
    if (filters.warehouse && filters.warehouse !== 'all') params.append('warehouse', filters.warehouse)
    if (filters.category && filters.category !== 'all') params.append('category', filters.category)
    if (filters.status && filters.status !== 'all') params.append('status', filters.status)
    if (filters.month && filters.month !== 'all') params.append('month', filters.month)

    const response = await axios.get(`${API_BASE_URL}/export/orders?${params.toString()}`, {
      responseType: 'blob'
    })
    return response.data
  },

  async exportInventory(filters = {}) {
    const params = new URLSearchParams()
    if (filters.warehouse && filters.warehouse !== 'all') params.append('warehouse', filters.warehouse)
    if (filters.category && filters.category !== 'all') params.append('category', filters.category)

    const response = await axios.get(`${API_BASE_URL}/export/inventory?${params.toString()}`, {
      responseType: 'blob'
    })
    return response.data
  }
}
```

- [ ] **Step 2: Commit**

```bash
git add client/src/api.js
git commit -m "feat: add exportOrders and exportInventory to api client"
```

---

## Task 4: Add Export Button to Orders.vue

**Files:**
- Modify: `client/src/views/Orders.vue`

- [ ] **Step 1: Add the export button to the template**

In `client/src/views/Orders.vue`, find the `card-header` div (around line 31) and add the button:

```html
      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('orders.allOrders') }} ({{ orders.length }})</h3>
          <button @click="exportCSV" :disabled="exporting" class="export-btn">
            {{ exporting ? 'Exporting...' : 'Export CSV' }}
          </button>
        </div>
```

- [ ] **Step 2: Add exporting ref and exportCSV method to setup()**

In the `setup()` function, after the `loadOrders` definition (around line 125), add:

```javascript
    const exporting = ref(false)

    const exportCSV = async () => {
      try {
        exporting.value = true
        const filters = getCurrentFilters()
        const blob = await api.exportOrders(filters)
        const url = URL.createObjectURL(blob)
        const a = document.createElement('a')
        a.href = url
        a.download = 'orders_export.csv'
        a.click()
        URL.revokeObjectURL(url)
      } catch (err) {
        console.error('Export failed:', err)
      } finally {
        exporting.value = false
      }
    }
```

- [ ] **Step 3: Add exporting and exportCSV to the return statement**

Replace the existing return statement:

```javascript
    return {
      t,
      loading,
      error,
      orders,
      getOrdersByStatus,
      getOrderStatusClass,
      formatDate,
      currencySymbol,
      translateProductName,
      translateCustomerName,
      exporting,
      exportCSV
    }
```

- [ ] **Step 4: Add export button styles**

In the `<style scoped>` section of `Orders.vue`, add:

```css
.export-btn {
  padding: 0.4rem 0.9rem;
  background: #0f172a;
  color: #e2e8f0;
  border: none;
  border-radius: 4px;
  font-size: 0.8rem;
  cursor: pointer;
}

.export-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

- [ ] **Step 5: Commit**

```bash
git add client/src/views/Orders.vue
git commit -m "feat: add Export CSV button to Orders view"
```

---

## Task 5: Add Export Button to Inventory.vue

**Files:**
- Modify: `client/src/views/Inventory.vue`

- [ ] **Step 1: Add the export button to the template**

In `client/src/views/Inventory.vue`, find the `card-header` div (around line 12). Wrap the existing search box and new button in a flex container:

```html
        <div class="card-header">
          <h3 class="card-title">{{ t('inventory.stockLevels') }} ({{ filteredItems.length }} {{ t('inventory.skus') }})</h3>
          <div class="card-header-actions">
            <div class="search-box">
              <svg class="search-icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z" clip-rule="evenodd" />
              </svg>
              <input
                v-model="searchQuery"
                type="text"
                :placeholder="t('inventory.searchPlaceholder')"
                class="search-input"
              />
              <button
                v-if="searchQuery"
                @click="searchQuery = ''"
                class="clear-search"
                :title="t('inventory.clearSearch')"
              >
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
                  <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
                </svg>
              </button>
            </div>
            <button @click="exportCSV" :disabled="exporting" class="export-btn">
              {{ exporting ? 'Exporting...' : 'Export CSV' }}
            </button>
          </div>
        </div>
```

- [ ] **Step 2: Add exporting ref and exportCSV method to setup()**

In `Inventory.vue` `setup()`, after the `loadInventory` definition (around line 166), add:

```javascript
    const exporting = ref(false)

    const exportCSV = async () => {
      try {
        exporting.value = true
        const filters = getCurrentFilters()
        const blob = await api.exportInventory({
          warehouse: filters.warehouse,
          category: filters.category
        })
        const url = URL.createObjectURL(blob)
        const a = document.createElement('a')
        a.href = url
        a.download = 'inventory_export.csv'
        a.click()
        URL.revokeObjectURL(url)
      } catch (err) {
        console.error('Export failed:', err)
      } finally {
        exporting.value = false
      }
    }
```

- [ ] **Step 3: Add exporting and exportCSV to the return statement**

Replace the existing return statement:

```javascript
    return {
      t,
      loading,
      error,
      items,
      searchQuery,
      filteredItems,
      getStockStatus,
      getStockStatusClass,
      translateCategory,
      showItemModal,
      selectedItem,
      showItemDetail,
      currencySymbol,
      translateProductName,
      translateWarehouse,
      exporting,
      exportCSV
    }
```

- [ ] **Step 4: Add export button and layout styles**

In the `<style scoped>` section of `Inventory.vue`, add:

```css
.card-header-actions {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.export-btn {
  padding: 0.4rem 0.9rem;
  background: #0f172a;
  color: #e2e8f0;
  border: none;
  border-radius: 4px;
  font-size: 0.8rem;
  cursor: pointer;
  white-space: nowrap;
}

.export-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

- [ ] **Step 5: Commit**

```bash
git add client/src/views/Inventory.vue
git commit -m "feat: add Export CSV button to Inventory view"
```

---

## Final Step: Create PR

```bash
git push origin workshop
```

Then open a PR from `workshop` → `main` with title:

> `feat: add CSV export for Orders and Inventory views`

PR body should describe: filter-aware export, two endpoints, two view buttons. Do not mention the intentional issues in the PR description — let the agents find them.
