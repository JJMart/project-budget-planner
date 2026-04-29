# Roo Code — Project Instructions

> **Note:** A copy of these instructions for GitHub Copilot is maintained at
> `.github/copilot-instructions.md`. Keep both files in sync when updating.

## What this project is

A single-page HTML budgeting tool that replaces a fragile Excel spreadsheet used for
planning multi-category project budgets (staff hours, travel, materials, subcontracts).
The tool runs entirely in the browser with no server or build step — just open
`index.html`. It persists data to `localStorage` and supports JSON save/load for
portability. The target users are project managers and PIs who need to build and share
cost estimates without wrestling with spreadsheet formula breakage.

## Read these files before touching any code

| File | What it documents |
|------|-------------------|
| `docs/budget_tool.md` | Full design reference for `index.html` — state model, rendering pipeline, table structures, import/export, burden-rate logic |

These are the single source of truth for design intent. If they contradict the code,
flag it — don't silently follow either one.

## Changelog files — read before modifying, create if absent

Many source files have a companion `CHANGELOG_<name>.md` in `docs/`.

- **Before modifying any file**, check whether a changelog exists for it and read it.
  This prevents accidentally reverting decisions that were already tried and abandoned.
- **Do not read every changelog upfront.** Only read the ones relevant to files you are
  about to change.
- **If no changelog exists** for a file you are modifying, create one and add the first
  entry after the user confirms the change works.

## Coding conventions

- **Language:** HTML5, vanilla CSS, vanilla ES6+ JavaScript — all in a single file
  (`index.html`). No frameworks, no build tools, no external dependencies.
- **Architecture:** All application state lives in a single `state` object. All
  rendering is done by `render*()` functions that rebuild their respective table
  innerHTML from `state`. Every user input triggers `save()` → `recalcAll()` which
  re-renders everything and persists to `localStorage`.
- **Naming:** Functions use camelCase. CSS uses BEM-lite with kebab-case class names.
  State keys match the domain model (staff, tasks, hours, travel, materials,
  subcontracts).
- **No external dependencies.** The tool must remain a single self-contained HTML file
  that works offline.
- **No dynamic SQL, no eval, no inline `onclick` string generation** beyond simple
  calls with literal numeric IDs.

## Documentation maintenance rules

- **Do not update** reference files, changelogs, or `README.md` until the user
  confirms the feature works as expected.
- After confirmation, update the relevant reference file and add a changelog entry.
- Changelog entries should capture *why* decisions were made, not just *what* changed.
- Commit messages should include a short "why" alongside the "what".

## README.md — always update for user-facing changes

After a user confirms a feature works, update `README.md` for any change that
affects how someone *operates* the tool. This includes new controls, changed workflows,
new output files, and renamed commands.

## Ignored in Git (do not create or assume these exist)

- No build output directories — there is no build step.
- `*.bak`, `*.tmp` — backup and temporary files.
- The `LMN_DamSystem_Budget_11162022.xlsx` file is the original reference spreadsheet
  and is not used at runtime.
