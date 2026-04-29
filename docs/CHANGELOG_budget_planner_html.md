# CHANGELOG — budget_planner.html

---

## 2026-04-28 — Initial implementation

**What changed:**  
Created the complete single-page budgeting tool from scratch, replacing the original
`LMN_DamSystem_Budget_11162022.xlsx` spreadsheet with a generalized HTML application.
The tool has four tabs (Staff Hours, Travel, Materials, Subcontracts), configurable
burden rates, auto-calculated totals, localStorage persistence, and JSON/CSV
import/export.

**Why:**  
The original Excel spreadsheet was fragile — formulas broke when rows or columns were
added, and it was tightly coupled to a specific project (LMN Dam System). A
browser-based tool with dynamic row/column management eliminates the structural
fragility while making the tool reusable across any project.

**Dead ends (if any):**  
- Considered parsing the `.xlsx` at runtime using SheetJS — rejected because the
  spreadsheet structure was too messy and project-specific to parse generically. A
  clean data-entry interface is more maintainable.
- Considered splitting into multiple files (HTML, CSS, JS) — rejected for portability;
  the tool needs to work as a single file sent via email or SharePoint.

**Side effects:**  
None — this is the initial creation of the file.

---
