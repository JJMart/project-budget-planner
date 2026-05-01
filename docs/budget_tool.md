# Budget Tool — Component Reference

> **Status:** Active development
> **Source file(s):** [`budget_planner.html`](../budget_planner.html)
> **Last reviewed:** 2026-05-01

## Purpose

The budget tool is a self-contained, single-page HTML application that lets a project
manager build a multi-category cost estimate with timeline awareness entirely in the
browser. It covers four budget categories — labor, travel, materials, and subcontracts —
applies configurable burden (overhead) rates, distributes costs across a project
timeline using burn rates, and visualizes spending with a monthly bar chart and a
labor-by-category pie chart.

The tool **deliberately does not**:

- Connect to any server or external API (except SheetJS CDN for Excel export).
- Require a build step, bundler, or package manager.
- Import or parse Excel files at runtime.
- Handle multi-user collaboration or access control.

## Dependencies

| Dependency | Why it is needed |
|------------|-----------------|
| [SheetJS (xlsx)](https://cdn.sheetjs.com/xlsx-0.20.3/package/dist/xlsx.full.min.js) | Excel export (`exportExcel()`). Loaded from CDN. Only needed for the "📊 Export Excel" button. |

All other functionality (data entry, calculations, chart rendering, localStorage,
JSON save/load) works offline with zero dependencies.

## Internal Design

### State Object

All data lives in a single `state` object persisted to `localStorage` under key
`projectBudgetPlanner`.

| Property | Type | Description |
|----------|------|-------------|
| `projectName` | `string` | Free-text project name |
| `fiscalYear` | `string` | Free-text fiscal year label |
| `projectStart` | `string` | ISO date string (YYYY-MM-DD) for project start |
| `projectDuration` | `number` | Project length in months |
| `burdenStaff` | `number` | Multiplier for labor costs |
| `burdenMaterials` | `number` | Multiplier for materials costs |
| `burdenTravel` | `number` | Multiplier for travel costs |
| `burdenSubcontracts` | `number` | Multiplier for subcontracts costs |
| `taskCategories` | `Array<{id, name, sortOrder}>` | Managed list of task categories |
| `chargeRates` | `Array<{id, category, rate}>` | Managed list of charge rate categories |
| `locations` | `Array<Location>` | Travel locations with cost defaults |
| `staff` | `Array<{id, name, chargeRateId}>` | Staff roster linked to charge rates |
| `tasks` | `Array<Task>` | Task definitions with dates and burn rates |
| `hours` | `Record<string, number>` | Sparse map `"taskId-staffId"` → hours |
| `travel` | `Array<TravelRow>` | Travel line items with dates |
| `materials` | `Array<MaterialRow>` | Materials with one-time or ongoing cost type |
| `subcontracts` | `Array<SubcontractRow>` | Subcontract line items with dates |

#### Task

```
{ id, categoryId, name, startDate, endDate, burnType, notes, sortOrder }
```

`burnType` is one of: `"linear"`, `"front-heavy"`, `"back-heavy"`, `"bell-curve"`.

`sortOrder` is a non-negative integer controlling display order within a category.
Up/down buttons in the ⚙ Tasks tab reorder tasks; `sortOrder` values are normalized
to `0, 1, 2, …` after each swap.

#### Location

```
{ id, name, distance, airfare, perDiem, vehiclePerDay, fuelPerMile }
```

Note: `perDiem` is $/day/person. Default values when a new location is added:
`perDiem: 200`, `vehiclePerDay: 75`, `fuelPerMile: 0.40`, all others `0`.

#### TravelRow

```
{ id, locationId, taskId, trips, people, daysPerTrip, startDate, endDate,
  perDiemOv, airfareOv, milesOv, fuelOv, vehicleOv, notes }
```

Override fields (`*Ov`) are null to use location defaults, or a number to override.

#### MaterialRow

```
{ id, item, taskId, costType, cost, startDate, endDate, notes }
```

`costType` is `"one-time"` or `"ongoing"` (monthly recurring).

#### SubcontractRow

```
{ id, vendor, description, cost, startDate, endDate, notes }
```

### Key Flows

**Rendering pipeline**

1. Any user input triggers `save()` (sync state → localStorage).
2. `recalcAll()` computes totals, updates summary cards, then calls all
   `render*()` functions which rebuild innerHTML from state.
3. `renderChart()` draws the monthly spending stacked bar chart.
4. `renderPieChart()` draws the labor-by-category pie chart on a canvas.
5. `renderGantt()` builds the Gantt table with Federal fiscal-quarter boundary lines.

**Monthly spending distribution (`calcMonthlySpending()`)**

- **Labor:** Each task's total cost is distributed across its start→end months
  using the task's burn rate weights.
- **Travel:** Single trips go to start month. Multi-trip entries distribute
  evenly across start→end range.
- **Materials (one-time):** Assigned to start date month.
- **Materials (ongoing):** Monthly cost repeated for each month in start→end range.
- **Subcontracts:** Spread evenly across start→end if both dates set.

**Burn rate weights (`burnWeights(type, n)`)**

| Type | Distribution |
|------|-------------|
| `linear` | Even: `1/n` per month |
| `front-heavy` | Decreasing: `n, n-1, ..., 1` (normalized) |
| `back-heavy` | Increasing: `1, 2, ..., n` (normalized) |
| `bell-curve` | Gaussian centered on midpoint (normalized) |

**Excel export (`exportExcel()`)**

Uses SheetJS to generate a multi-sheet `.xlsx` with:
- Summary (project info, cost breakdown, grand total)
- Monthly Spending (month-by-month with cumulative total)
- Labor (task × staff hours grid)
- Staff (roster with rates)
- Tasks (definitions with dates and burn rates)
- Travel (trip details with cost breakdowns)
- Materials (items with type and total cost)
- Subcontracts

Sheets have: merged title rows, auto-filter on headers, currency number formatting,
auto-sized columns, and sheet protection (read-only).

### Tab Structure

| Tab | Type | Purpose |
|-----|------|---------|
| Labor | Cost | Staff × Task hours grid with auto-calculated costs |
| Travel | Cost | Trip entries linked to locations and tasks |
| Materials | Cost | One-time and ongoing material costs with dates |
| Subcontracts | Cost | Vendor line items with dates |
| 📅 Gantt | View | Timeline bar chart of tasks with Federal fiscal-quarter boundary lines |
| ⚙ Tasks | Setup | Define tasks with categories, dates, burn rates, and sort order |
| ⚙ Task Categories | Setup | Managed category list (16 defaults) |
| ⚙ Charge Rates | Setup | Managed rate list (14 defaults with actual rates) |
| ⚙ Locations | Setup | Travel destinations with cost defaults |

### Default Data

On first load, the tool seeds:
- 16 task categories (Planning & Design through Other)
- 14 charge rate categories (Student Intern through Specialist 5) with rates

### Charts

- **Monthly Spending (bar chart):** CSS-based stacked bars, color-coded by category,
  with hover tooltips. Height scales to max month.
- **Labor by Category (pie chart):** HTML5 Canvas-based, with color legend showing
  amounts and percentages. Groups all task costs by their category.

## Known Constraints and Gotchas

- **SheetJS CDN dependency:** Excel export requires internet. All other features
  work offline.
- **localStorage only:** Data scoped to browser origin (file path). Renaming or
  moving the file resets data. Use JSON save/load for portability.
- **No undo:** Double-confirm on Reset All is the only safety net.
- **Per Diem is $/day/person:** The `totalDays = trips × people × daysPerTrip`
  already includes the people multiplier.
- **Ongoing materials:** Total = monthly cost × number of months in range.
  Changing project dates affects totals.

## Rejected Approaches

- **Multi-file SPA:** Rejected for portability (single file distribution).
- **React/Vue/framework:** Rejected — requires build step.
- **Excel parsing at runtime (SheetJS import):** Original spreadsheet too messy.
- **IndexedDB:** Overkill for data volume.
- **Embedding SheetJS inline:** Would add ~500KB to file size. CDN approach chosen.
- **`flat` burn type:** Removed — identical to `linear`, caused confusion.

## Open Issues / To-Do

- Print-friendly CSS for PDF reports.
- Drag-and-drop row reordering.
- Data validation warnings (hours exceeding FTE limits, dates outside project range).
- Cumulative spending line overlay on bar chart.
