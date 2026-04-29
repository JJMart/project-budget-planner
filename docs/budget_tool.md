# Budget Tool — Component Reference

> **Status:** Active development  
> **Source file(s):** [`budget_planner.html`](../budget_planner.html)
> **Last reviewed:** 2026-04-28

## Purpose

The budget tool is a self-contained, single-page HTML application that lets a project
manager build a multi-category cost estimate entirely in the browser. It covers four
budget categories — staff hours, travel, materials, and subcontracts — and applies
configurable burden (overhead) rates to each category to produce a burdened grand total.

The tool **deliberately does not**:

- Connect to any server or external API.
- Require a build step, bundler, or package manager.
- Import or parse Excel files at runtime (the original `.xlsx` is a design reference
  only).
- Handle multi-user collaboration or access control.

## Dependencies

| Dependency | Why it is needed |
|------------|-----------------|
| None | The tool is 100 % vanilla HTML/CSS/JS with zero external dependencies. |

## Public Interface

This is a browser application, not a library. There is no programmatic API.  
User-facing controls are described in the [README](../README.md).

## Internal Design

### State Object

All application data lives in a single `state` object persisted to `localStorage`
under the key `projectBudgetPlanner`. The shape is:

| Property | Type | Description |
|----------|------|-------------|
| `projectName` | `string` | Free-text project name shown in header and exports |
| `fiscalYear` | `string` | Free-text fiscal year label |
| `burdenStaff` | `number` | Multiplier applied to total unburdened staff cost |
| `burdenMaterials` | `number` | Multiplier applied to total unburdened materials cost |
| `burdenTravel` | `number` | Multiplier applied to total unburdened travel cost |
| `burdenSubcontracts` | `number` | Multiplier applied to total unburdened subcontracts cost |
| `staff` | `Array<StaffMember>` | List of staff members (id, name, role, rate) |
| `tasks` | `Array<Task>` | List of tasks (id, category, name, notes) |
| `hours` | `Record<string, number>` | Sparse map keyed `"taskId-staffId"` → hours worked |
| `travel` | `Array<TravelRow>` | Travel line items |
| `materials` | `Array<MaterialRow>` | Material/supply line items |
| `subcontracts` | `Array<SubcontractRow>` | Subcontract line items |

A global `nextId` counter provides unique IDs via `genId()`.

#### StaffMember

```
{ id: number, name: string, role: string, rate: number }
```

`role` is one of: `"Management"`, `"Lab/Field Operation"`, `"Project Support"`,
`"Student Employee"`, `"Post Doc"`, `"Other"`.

#### Task

```
{ id: number, category: string, name: string, notes: string }
```

`category` is a free-text grouping label (e.g. "Deployment", "Data analysis").

#### TravelRow

```
{ id, category, destination, trips, people, daysPerTrip,
  perDiem, airfareRate, milesPerTrip, fuelRate, vehicleRate, notes }
```

Computed columns (not stored):

| Computed | Formula |
|----------|---------|
| Total Days | `trips × people × daysPerTrip` |
| Per Diem Total | `totalDays × perDiem` |
| Airfare Total | `trips × people × airfareRate` |
| Fuel Total | `trips × milesPerTrip × fuelRate` |
| Vehicle Total | `trips × daysPerTrip × vehicleRate` |
| Trip Total | sum of the four totals above |

#### MaterialRow

```
{ id: number, item: string, task: string, cost: number, notes: string }
```

`task` is the name of an associated task (optional, for cross-reference).

#### SubcontractRow

```
{ id: number, vendor: string, description: string, cost: number, notes: string }
```

### Key Flows

**Rendering pipeline**

1. Any user input triggers `save()` (sync state → localStorage).
2. `recalcAll()` is called, which:
   a. Reads burden rates from DOM inputs.
   b. Iterates each category to compute unburdened totals.
   c. Multiplies each by its burden rate for burdened totals.
   d. Updates the four summary cards and grand total.
   e. Calls `renderStaffTable()`, `renderTravelTable()`, `renderMaterialsTable()`,
      `renderSubcontractsTable()` — each rebuilds its table's innerHTML from `state`.

**Adding a staff member or task**

1. User clicks "+ Add Staff" or "+ Add Task" → `openModal(type)`.
2. Modal form is populated with fields.
3. On submit, a new object with `genId()` is pushed to the appropriate array.
4. `closeModal()` → `save()` → `recalcAll()`.

**Entering hours**

1. The staff table renders an `<input type="number">` at each task×staff intersection.
2. `onchange` calls `setHours(taskId, staffId, value)`.
3. If value > 0, `state.hours["taskId-staffId"]` is set; if 0, the key is deleted
   (sparse storage).
4. `save()` → `recalcAll()`.

**Import / Export**

- **Save JSON:** Serialises `{ state, nextId }` as indented JSON → triggers download.
- **Load JSON:** Reads a `.json` file, parses it, replaces `state` and `nextId`,
  then `load()` → `recalcAll()`.
- **Export CSV:** Builds a multi-section CSV string (summary, staff, travel, materials,
  subcontracts) → triggers download. Values are quoted to handle commas.

### Burden Rate Logic

The original spreadsheet applied different overhead multipliers to different cost
categories. The tool generalises this with four configurable burden rates displayed
in the project info bar:

| Rate | Default | Applied to |
|------|---------|-----------|
| Staff | 1.0 | Sum of (hours × hourly rate) across all tasks and staff |
| Materials | 1.3 | Sum of material line item costs |
| Travel | 1.2 | Sum of computed trip totals |
| Subcontracts | 1.2 | Sum of subcontract line item costs |

The summary cards show **burdened** totals. The table footers show **unburdened**
totals. This matches common DOE/lab budgeting conventions where burden is applied at
the category level, not per line item.

### FTE Calculation

The staff table footer includes an FTE row computed as `totalHours / 1840` per staff
member, where 1840 is the standard person-year hours (matching the original
spreadsheet).

## Known Constraints and Gotchas

- **Single-file architecture:** All HTML, CSS, and JS live in one file. This is
  intentional for maximum portability (email, USB, SharePoint). Do not split into
  separate files without explicit user request.
- **localStorage only:** Data is scoped to the browser origin. Clearing browser data
  deletes the budget. The JSON save/load feature is the portability mechanism.
- **No undo:** There is no undo stack. The double-confirm on "Reset All" is the only
  safety net.
- **Inline `onclick` handlers:** Used for simplicity in a single-file app. These
  reference `state` array indices which are rebuilt on every render, so they stay
  correct.
- **Hours stored as sparse map:** Keys are `"taskId-staffId"`. Deleting a staff
  member or task also cleans up orphan keys. If IDs are ever reused (they shouldn't
  be — `nextId` only increments), stale data could appear.

## Rejected Approaches

- **Multi-file SPA (HTML + JS + CSS separate files):** Rejected because the primary
  distribution method is sending a single file via email or SharePoint. A multi-file
  approach would require zipping or a web server.
- **React / Vue / framework-based:** Rejected for the same portability reason and
  because it would require a build step.
- **Excel parsing at runtime (SheetJS):** Considered for importing the original `.xlsx`
  directly. Rejected because it adds a 500 KB+ dependency and the spreadsheet
  structure is too messy/specific to parse generically. Better to start fresh with
  clean data entry.
- **IndexedDB storage:** Rejected as overkill for the data volume. localStorage is
  sufficient and simpler to debug.

## Open Issues / To-Do

- Print-friendly CSS for generating PDF reports.
- Optional per-task subtotals in the summary cards.
- Drag-and-drop row reordering for tasks and travel.
- Data validation warnings (e.g., hours exceeding FTE limits).
