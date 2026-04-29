# CHANGELOG — budget_planner.html

---

## 2026-04-29 — Labor by category pie chart

**What changed:**  
Added a canvas-based pie chart next to the monthly spending bar chart showing labor
costs broken down by task category with a color legend.

**Why:**  
Visual breakdown of where labor dollars are allocated helps quickly identify which
categories dominate the budget.

**Side effects:** None.

---

## 2026-04-29 — Excel export with formatting

**What changed:**  
Replaced CSV export with multi-sheet Excel export using SheetJS (CDN). Sheets include
Summary, Monthly Spending, Labor, Staff, Tasks, Travel, Materials, Subcontracts.
Added currency number formatting, merged title rows, auto-filters, column auto-sizing,
sheet protection, and cumulative totals. Filenames now include timestamps.

**Why:**  
Excel is more useful for stakeholder review than CSV. Formatting makes the output
professional without manual cleanup. Timestamps prevent overwriting previous exports.

**Dead ends:**  
- Cell styling (bold, colors, borders) requires SheetJS Pro (paid). Free version
  supports number formats, merges, auto-filters, and column widths only.

**Side effects:** Added SheetJS CDN dependency (internet required for Excel export).

---

## 2026-04-29 — Timeline, burn rates, Tasks tab, material/travel dates

**What changed:**  
Major expansion: project start date and duration fields; Tasks setup tab with
start/end dates and burn rate (linear, front-heavy, back-heavy, bell-curve);
materials with one-time vs ongoing ($/mo) cost types and dates; travel with start/end
dates and evenly-spaced multi-trip distribution; subcontracts with dates; monthly
spending stacked bar chart with hover tooltips.

**Why:**  
The original tool had no time dimension. Budget reviews need to know not just total
cost but when money will be spent. Burn rates allow modeling different spending
patterns per task.

**Dead ends:**  
- `flat` burn type was initially included but removed — identical to `linear`.

**Side effects:** Tasks are now defined in the ⚙ Tasks tab, not inline in the Labor
tab. Labor tab is now hours-only entry.

---

## 2026-04-29 — Charge rate defaults with actual rates, import, staff edit

**What changed:**  
Populated default charge rates with actual dollar values from template. Added CSV/JSON
import for charge rates. Added ✎ edit button for staff members to change charge rate
category.

**Why:**  
Eliminates manual data entry for common rate categories. Edit button needed for
promotions/rate changes without deleting and re-adding staff.

**Side effects:** None.

---

## 2026-04-29 — Managed task categories, tab reorder, rename to Labor

**What changed:**  
Added managed task categories (16 defaults) as a setup tab. Tasks now select category
from dropdown. Tab order changed: cost tabs (Labor, Travel, Materials, Subcontracts)
first, then setup tabs (⚙) with dashed border styling. "Staff Hours" renamed to
"Labor".

**Why:**  
Consistent categories across tasks. Visual separation of cost-entry tabs from
configuration tabs.

**Side effects:** Task `category` field migrated from string to `categoryId` reference.

---

## 2026-04-29 — Locations tab, Travel refactor, Charge Rates tab

**What changed:**  
Added Locations tab (distance, airfare, per diem, vehicle, fuel per location). Travel
entries now select a location and task via dropdowns with auto-filled rates and an
override modal. Added Charge Rates tab where staff rates are defined by category.
Staff members select a rate category instead of entering rates manually.

**Why:**  
Eliminates redundant data entry — define a location once, reuse across trips. Charge
rate categories ensure consistency across staff.

**Dead ends:**  
- Considered free-text with autocomplete for categories — rejected in favor of
  managed list for consistency.

**Side effects:** Staff `rate` field replaced with `chargeRateId` reference.

---

## 2026-04-28 — Initial implementation

**What changed:**  
Created the complete single-page budgeting tool from scratch, replacing the original
`LMN_DamSystem_Budget_11162022.xlsx` spreadsheet with a generalized HTML application.

**Why:**  
The original Excel spreadsheet was fragile — formulas broke when rows or columns were
added. A browser-based tool with dynamic management eliminates structural fragility.

**Dead ends:**  
- Considered parsing `.xlsx` at runtime — rejected (too messy/specific).
- Considered multi-file architecture — rejected for portability.

**Side effects:** None — initial creation.

---
