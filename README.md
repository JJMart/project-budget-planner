# Project Budget Planner

A single-file, browser-based budgeting tool for building multi-category project cost
estimates with timeline awareness. Replaces fragile Excel spreadsheets with a dynamic,
portable HTML application that runs offline — no server, no install.

---

## Getting Started

### Prerequisites

- A modern web browser (Chrome, Firefox, Edge, Safari).
- Internet connection only needed for **📊 Export Excel** (loads SheetJS from CDN).

### Opening the Tool

1. Double-click **`budget_planner.html`** (or drag it into your browser).
2. The tool opens with default charge rates and task categories pre-populated.

---

## Core Concepts

| Concept | Description |
|---------|-------------|
| **Staff** | Team members linked to a charge rate category. |
| **Tasks** | Work items with a category, start/end dates, and a burn rate. Defined in ⚙ Tasks tab. |
| **Hours** | The intersection of a task and a staff member in the Labor tab. |
| **Charge Rates** | Hourly rate categories (e.g., Scientist/Engineer 3 @ $216.32/hr). Staff select a category. |
| **Locations** | Travel destinations with default per diem, airfare, vehicle, and fuel costs. |
| **Task Categories** | Grouping labels (e.g., Field — Deployment, Data Analysis). |
| **Burn Rate** | How a task's cost distributes over time: linear, front-heavy, back-heavy, or bell-curve. |
| **Burden Rate** | Overhead multiplier applied per cost category (labor, travel, materials, subcontracts). |
| **One-time vs Ongoing** | Materials can be a single purchase (one date) or recurring monthly (start/end dates). |

---

## Walkthrough — Building a Budget

### 1. Set Project Info

Fill in **Project Name**, **Fiscal Year**, **Start Date**, and **Duration (months)**.
Adjust **Burden Rates** as needed:

| Category | Default |
|----------|---------|
| Labor | 1.0× |
| Materials | 1.3× |
| Travel | 1.2× |
| Subcontracts | 1.2× |

### 2. Configure Setup Tabs (⚙)

**Task Categories** — 16 defaults included. Add, rename, reorder, or remove.

**Charge Rates** — 14 defaults with actual rates. Import from CSV/JSON or edit inline.

**Locations** — Define travel destinations with per diem ($/day/person), airfare,
vehicle ($/day), fuel ($/mile), and distance.

**Tasks** — Define work items with category, name, start date, end date, burn rate,
and notes. These populate the Labor tab.

### 3. Add Staff (Labor Tab)

Click **+ Add Staff** and select a charge rate category. Edit staff with the ✎ button
(e.g., after a promotion).

### 4. Enter Hours (Labor Tab)

Type hours in the task × staff grid. Costs auto-calculate. Footer shows totals,
costs, and FTE (hours ÷ 1840).

### 5. Add Travel

Select a location (auto-fills rates) and a task. Set trips, people, days. For
multi-trip entries, set start/end dates — trips space evenly across the range.
Override specific rates via the "Override" button.

### 6. Add Materials

Choose **One-time** (single purchase on a date) or **Ongoing ($/mo)** (recurring
with start/end dates, total = rate × months).

### 7. Add Subcontracts

Enter vendor, description, cost, and optional start/end dates.

### 8. Review

- **Summary cards** show burdened totals and grand total.
- **Monthly Spending chart** shows stacked bar chart of burdened costs over time.
- **Labor by Category pie chart** shows where labor dollars are allocated.

---

## Saving and Loading

| Action | How |
|--------|-----|
| **Auto-save** | Data saved to localStorage on every edit. |
| **Save JSON** | 💾 Downloads a timestamped `.json` file with all budget data. |
| **Load JSON** | 📂 Imports a `.json` file, replacing current budget. |
| **Export Excel** | 📊 Downloads a formatted `.xlsx` with 8 sheets (requires internet). |
| **Reset All** | 🗑 Clears everything (double confirmation required). |

---

## Things That Are Easy to Forget

| Gotcha | Detail |
|--------|--------|
| Data is browser-local | Renaming/moving the file or clearing browser data resets it. Use JSON save. |
| Burden rates are per-category | They multiply the entire category total, not individual items. |
| Per Diem is $/day/person | `totalDays = trips × people × daysPerTrip` already includes people. |
| Ongoing materials | Total = monthly cost × number of months in date range. |
| Burn rates need task dates | Without start/end dates on tasks, costs can't distribute on the chart. |
| Excel export needs internet | SheetJS loaded from CDN. All other features work offline. |
| Removing staff/tasks deletes hours | Permanent — use JSON save as backup. |

---

## Quick Reference

| Control | Location | What it does |
|---------|----------|-------------|
| Project info fields | Top bar | Name, fiscal year, start date, duration, burden rates |
| Summary cards | Below top bar | Burdened totals per category and grand total |
| Monthly Spending chart | Below cards | Stacked bars showing monthly burdened costs |
| Labor by Category pie | Next to bar chart | Pie chart of labor costs by task category |
| Cost tabs | Labor, Travel, Materials, Subcontracts | Enter cost data |
| Setup tabs (⚙) | Tasks, Categories, Rates, Locations | Configure reference data |
| + Add buttons | Tab headers | Add new items |
| ✎ / ✕ buttons | Table rows | Edit or remove items |
| Override button | Travel detail rows | Override location defaults per trip |
| 💾 Save JSON | Header | Download budget as timestamped JSON |
| 📂 Load JSON | Header | Upload a JSON budget file |
| 📊 Export Excel | Header | Download formatted multi-sheet Excel |
| 🗑 Reset All | Header | Clear all data (double confirmation) |

---

## Appendix — Developer & Maintainer Notes

### Architecture

Single HTML file (`budget_planner.html`) with embedded CSS and JavaScript. One external
dependency: SheetJS from CDN for Excel export. Everything else works offline.

See [`docs/budget_tool.md`](docs/budget_tool.md) for the full component reference.

### AI Assistant Instructions

This project uses the documentation strategy described in
[`Github_Copilot_Project_Documentation_Guide.md`](Github_Copilot_Project_Documentation_Guide.md).
AI assistants should read [`.roo/rules.md`](.roo/rules.md) at the start of each session.

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
├── charge_rates_template.csv             ← Importable charge rates template
├── README.md                             ← This file
└── Github_Copilot_Project_Documentation_Guide.md  ← Documentation strategy guide
```
