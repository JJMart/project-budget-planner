# Project Budget Planner

A single-file, browser-based budgeting tool for building multi-category project cost
estimates. Replaces fragile Excel spreadsheets with a dynamic, portable HTML
application that runs entirely offline — no server, no install, no dependencies.

---

## Getting Started

### Prerequisites

- A modern web browser (Chrome, Firefox, Edge, Safari).
- That's it. No installation, no build step, no internet connection required.

### Opening the Tool

1. Double-click **`budget_planner.html`** (or drag it into your browser).
2. The tool opens with an empty budget. Start adding staff and tasks.

---

## Core Concepts

| Concept | Description |
|---------|-------------|
| **Staff** | Team members with an hourly rate. Added via the "+ Add Staff" button. |
| **Tasks** | Work items grouped by category. Added via the "+ Add Task" button. |
| **Hours** | The intersection of a task and a staff member. Enter hours directly in the grid. |
| **Burden rate** | An overhead multiplier applied to each cost category (staff, travel, materials, subcontracts). Configurable in the project info bar. |
| **Unburdened cost** | The raw cost before overhead is applied (shown in table footers). |
| **Burdened cost** | The final cost after multiplying by the burden rate (shown in summary cards). |

---

## Walkthrough — Building a Budget

### 1. Set Project Info

Fill in the **Project Name**, **Fiscal Year**, and adjust the **Burden Rates** at the
top of the page. Default burden rates are:

| Category | Default |
|----------|---------|
| Staff | 1.0× |
| Materials | 1.3× |
| Travel | 1.2× |
| Subcontracts | 1.2× |

### 2. Add Staff Members

1. Click the **Staff Hours** tab.
2. Click **+ Add Staff**.
3. Enter the person's name, role, and fully-burdened hourly rate.
4. Repeat for each team member.

### 3. Add Tasks

1. Click **+ Add Task**.
2. Enter a **Category** (grouping label, e.g. "Deployment") and a **Task Name**.
3. Optionally add notes.

### 4. Enter Hours

In the Staff Hours grid, type the number of hours each person will spend on each task.
Costs are calculated automatically:

- **Task Cost** = sum of (hours × rate) across all staff for that task.
- **Total Hours** and **Total Cost** appear in the footer.
- **FTE** is calculated as total hours ÷ 1840 (standard person-year).

### 5. Add Travel

1. Click the **Travel** tab.
2. Click **+ Add Trip**.
3. Fill in destination, number of trips, people, days, per diem rate, airfare, mileage,
   and vehicle rental rate.
4. Totals are computed automatically.

### 6. Add Materials

1. Click the **Materials** tab.
2. Click **+ Add Item**.
3. Enter item name, optionally link it to a task, and enter the cost.

### 7. Add Subcontracts

1. Click the **Subcontracts** tab.
2. Click **+ Add Subcontract**.
3. Enter vendor, description, and cost.

### 8. Review Summary

The **summary cards** at the top show burdened totals for each category and the
**Grand Total**.

---

## Saving and Loading

| Action | How |
|--------|-----|
| **Auto-save** | Data is saved to your browser's localStorage on every edit. Closing and reopening the file in the same browser restores your data. |
| **Save JSON** | Click **💾 Save JSON** to download a `.json` file containing the full budget. Use this to back up your work or transfer it to another computer. |
| **Load JSON** | Click **📂 Load JSON** and select a previously saved `.json` file. This replaces the current budget. |
| **Export CSV** | Click **📄 Export CSV** to download a `.csv` file suitable for Excel, Google Sheets, or reporting. |
| **Reset All** | Click **🗑 Reset All** to clear everything (requires double confirmation). |

---

## Things That Are Easy to Forget

| Gotcha | Detail |
|--------|--------|
| Data is browser-local | If you clear browser data or open the file from a different location, your auto-saved data won't be there. Use **Save JSON** to preserve your work. |
| Burden rates apply at the category level | They multiply the entire category total, not individual line items. |
| Hourly rates are per-person | Enter the fully loaded rate for each staff member. The tool does not add overhead to individual rates. |
| Removing staff/tasks deletes hours | Removing a staff member or task permanently deletes all associated hour entries. |
| No undo | There is no undo feature. Use **Save JSON** before making large changes. |

---

## Quick Reference

| Control | Location | What it does |
|---------|----------|-------------|
| Project Name / Fiscal Year | Top bar | Labels for the budget (included in exports) |
| Burden Rate fields | Top bar | Overhead multipliers per category |
| Tab buttons | Below summary cards | Switch between Staff, Travel, Materials, Subcontracts |
| + Add Staff / + Add Task | Staff tab header | Open modal to add a new column / row |
| + Add Trip | Travel tab header | Add a new travel line item |
| + Add Item | Materials tab header | Add a new material line item |
| + Add Subcontract | Subcontracts tab header | Add a new subcontract line item |
| ✕ buttons | Row/column ends | Remove that item (with confirmation) |
| 💾 Save JSON | Header | Download budget as JSON |
| 📂 Load JSON | Header | Upload a JSON budget file |
| 📄 Export CSV | Header | Download budget as CSV |
| 🗑 Reset All | Header | Clear all data (double confirmation) |

---

## Appendix — Developer & Maintainer Notes

### Architecture

The entire application is a single HTML file (`budget_planner.html`) with embedded CSS and
JavaScript. There are no external dependencies, no build step, and no server
requirement. This is intentional for maximum portability.

See [`docs/budget_tool.md`](docs/budget_tool.md) for the full component reference
including data structures, rendering pipeline, and design decisions.

### AI Assistant Instructions

This project uses the documentation strategy described in
[`Github_Copilot_Project_Documentation_Guide.md`](Github_Copilot_Project_Documentation_Guide.md).
AI assistants should read [`.roo/rules.md`](.roo/rules.md) at the start of each
session.

### File Structure

```
project-budget-planner/
├── .roo/
│   └── rules.md                          ← AI reads this first, every session
├── .github/
│   └── copilot-instructions.md           ← GitHub Copilot version of rules.md
├── docs/
│   ├── budget_tool.md                    ← Component reference for budget_planner.html
│   └── CHANGELOG_budget_planner_html.md  ← Change history for budget_planner.html
├── budget_planner.html                   ← The application (open in browser)
├── README.md                             ← This file
├── Github_Copilot_Project_Documentation_Guide.md  ← Documentation strategy guide
└── LMN_DamSystem_Budget_11162022.xlsx    ← Original spreadsheet (reference only)
```
