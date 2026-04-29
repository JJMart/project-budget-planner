# GitHub Copilot — Project Instructions

> **Note:** These instructions are mirrored from `.roo/rules.md` (Roo Code).
> Keep both files in sync when updating.

## What this project is

A single-page HTML budgeting tool that replaces fragile Excel spreadsheets for
planning multi-category project budgets (labor, travel, materials, subcontracts)
with timeline awareness. Open `budget_planner.html` in any browser — no server or
build step needed. Persists data to `localStorage`; supports JSON save/load for
portability and Excel export via SheetJS CDN. Target users are project managers and
PIs building cost estimates.

## Read these files before touching any code

| File | What it documents |
|------|-------------------|
| `docs/budget_tool.md` | Full design reference for `budget_planner.html` — state model, rendering pipeline, table structures, timeline logic, burn rates, charts, import/export |

These are the single source of truth for design intent. If they contradict the code,
flag it — don't silently follow either one.

## Changelog files — read before modifying, create if absent

Source files have a companion `CHANGELOG_<name>.md` in `docs/`.

- **Before modifying any file**, check whether a changelog exists for it and read it.
- **Do not read every changelog upfront.** Only the ones relevant to files you change.
- **If no changelog exists**, create one after the user confirms the change works.

## Coding conventions

- **Language:** HTML5, vanilla CSS, vanilla ES6+ JavaScript — all in a single file
  (`budget_planner.html`). No frameworks, no build tools.
- **External dependency:** SheetJS loaded from CDN for Excel export only. Everything
  else works offline.
- **Architecture:** All state in a single `state` object. All rendering via
  `render*()` functions that rebuild innerHTML. Every input triggers
  `save()` → `recalcAll()` → re-render all + charts.
- **Naming:** Functions camelCase. CSS kebab-case. State keys match domain model.
- **Charts:** Monthly spending uses CSS-based stacked bars. Labor pie chart uses
  HTML5 Canvas. No charting libraries.
- **No eval, no dynamic SQL.** Inline `onclick` handlers with literal IDs only.

## Documentation maintenance rules

- **Do not update** reference files, changelogs, or `README.md` until the user
  confirms the feature works as expected.
- After confirmation, update the relevant reference file and add a changelog entry.
- Changelog entries should capture *why* decisions were made, not just *what* changed.

## README.md — always update for user-facing changes

After a user confirms a feature works, update `README.md` for any change that
affects how someone *operates* the tool.

## Ignored in Git (do not create or assume these exist)

- `LMN_DamSystem_Budget_11162022.xlsx` — original reference spreadsheet
- `*_budget.json`, `*_budget.xlsx`, `*_budget.csv` — user exports
- `SF_Study_budget.*` — test data
- `*.bak`, `*.tmp` — backup and temporary files
