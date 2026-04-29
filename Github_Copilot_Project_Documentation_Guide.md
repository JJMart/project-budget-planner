# AI Assistant Project Documentation Guide

This guide describes a documentation strategy for software projects that use AI coding
assistants (Roo Code, GitHub Copilot, Cursor, Claude, etc.). The goal is to give the AI
enough context to act correctly on the first attempt — avoiding reversions of deliberate
design decisions, re-introduction of fixed bugs, and silent deviation from project
conventions.

Drop this file, and the files it describes, into any repository. Point the AI to this
guide at the start of a session.

---

## Contents

1. [Core Idea](#1-core-idea)
2. [File Structure](#2-file-structure)
3. [AI Instructions File](#3-ai-instructions-file)
4. [Component Reference Files](#4-component-reference-files)
5. [Changelog Files](#5-changelog-files)
6. [User Guide](#6-user-guide)
7. [Workflow Rules — What the AI Must Follow](#7-workflow-rules--what-the-ai-must-follow)
8. [Template: AI Instructions File](#8-template-ai-instructions-file)
9. [Template: Component Reference File](#9-template-component-reference-file)
10. [Template: Changelog Entry](#10-template-changelog-entry)
11. [Maintenance Checklist](#11-maintenance-checklist)
12. [GitHub Copilot Compatibility](#12-github-copilot-compatibility)

---

## 1. Core Idea

Git commit history records *what* changed. It does not record *why*, and it does not
capture dead ends, rationale, or constraints that are invisible in the final code.

An AI assistant starting a new session has access to the current source files but not to
the reasoning behind them. Without that reasoning, it will:

- Silently revert a deliberate workaround that you already tried and rejected.
- Re-implement a utility function that already exists under a different name.
- Violate a naming or architectural convention that exists for a good reason.
- Undo a bug fix whose root cause is not obvious from the surrounding code alone.

This strategy closes that gap by maintaining three layers of prose documentation
alongside the code:

| Layer | Granularity | What it captures | When the AI reads it |
|-------|------------|-----------------|----------------------|
| **Project instructions** (`.roo/rules.md`) | Whole project | Scope, conventions, file map, toolchain constraints | Every session, automatically |
| **Component reference files** (`component_name.md`) | One major system or subsystem (may span many source files) | Full design intent, API, data model, rejected approaches | Before touching that system |
| **Changelog files** (`CHANGELOG_sourcefile.md`) | One individual source file | Chronological record of decisions and dead ends for that file | Before modifying the corresponding source file |

Together, these let an AI reconstruct enough project context to act correctly without
reading every source file from scratch.

---

## 2. File Structure

```
your-project/
├── .roo/
│   └── rules.md                     ← AI reads this first, every session (Roo Code)
├── docs/                            ← optional; all human-authored markdown
│   ├── component_a.md               ← reference for a major system or subsystem
│   ├── component_b.md               ← one .md per system/subsystem, not per source file
│   ├── CHANGELOG_source_file_1.md   ← one changelog per individual source file
│   ├── CHANGELOG_source_file_2.md   ← created the first time that source file is modified
│   └── CHANGELOG_source_file_3.md
├── README.md                        ← end-user documentation and project homepage
├── src/
│   └── ...                          ← your code
└── AI_Assistant_Project_Documentation_Guide.md   ← this file
```

You can keep the markdown files at the repository root instead of in `docs/` if you
prefer. The important thing is consistency: wherever you put them, `.roo/rules.md`
must tell the AI exactly where they are.

> **GitHub Copilot users:** see [§12](#12-github-copilot-compatibility) for how to
> adapt this structure for Copilot's `.github/copilot-instructions.md` file.

---

## 3. AI Instructions File

**Roo Code:** `.roo/rules.md`

This is the single most important file. Roo Code automatically loads `.roo/rules.md`
at the start of every session in the workspace. Other AI tools can be instructed to
read it at the start of a session.

**It must contain:**

1. **What the project is** — one paragraph. What problem does it solve? Who uses it?
2. **Read-first file table** — a table of your component reference files with a
   one-line description of what each covers. The AI reads only what is relevant to the
   current task; this table tells it which file that is. Changelog files are handled in
   a separate section and are discovered on demand, not enumerated here.
3. **Coding conventions** — language version, toolchain, architectural patterns, naming
   rules, things you explicitly do *not* do. Be brief but specific.
4. **Documentation maintenance rules** — when to update the reference files and
   changelogs, and what triggers an update.
5. **Files / folders to ignore** — generated output directories, cache folders, backup
   files. The AI should not create, assume, or reference these.

**Keep it short.** Two to four pages. If it grows beyond that, move detail into the
component reference files and link to them here.

See [§8](#8-template-ai-instructions-file) for a fill-in-the-blanks template.

---

## 4. Component Reference Files

One markdown file per major system or subsystem — not one per individual source file.
A component reference file typically covers an entire coherent feature area that may
span several source files (e.g., the whole GUI, the data processing pipeline, the
authentication system). This is the authoritative source of truth for design intent
for that system.

**A component reference file covers:**

- **Purpose and scope** — what this component does and what it deliberately does *not* do.
- **Public interface** / API — function signatures, input/output types, events emitted,
  side effects.
- **Internal design** — key data structures, state variables, the flow through major
  operations. Enough detail that the AI can make a targeted edit without having to read
  every line of source.
- **Dependencies** — which other modules, libraries, or external services it relies on.
- **Known constraints and gotchas** — things that look wrong but are intentional, platform
  quirks, performance considerations, and past bugs worth remembering.
- **Out-of-scope / rejected approaches** — brief notes on approaches that were considered
  and rejected, so they are not re-proposed.

**Naming:** `<system_name>.md` (e.g., `auth_service.md`, `data_pipeline.md`,
`main_window.md`). Use broad, descriptive names that reflect a feature area, not
one-to-one mappings to source file names.

**Who updates it:** The human developer (or the AI, with human confirmation). See §7.

See [§9](#9-template-component-reference-file) for a template.

---

## 5. Changelog Files

One `CHANGELOG_<source_file_name>.md` per **individual source code file** — named
directly after the file it tracks (e.g., `CHANGELOG_auth_controller.md` for
`auth_controller.py`). Do not create changelogs in advance for every file in the
project; for existing codebases, create a changelog the **first time you modify that
file**, and populate it with the change being made. New projects can adopt the same
approach: the changelog comes into existence when the file first changes, not before.

Entries are added chronologically, newest first.

**What a changelog entry captures:**

- **Date and version/commit** (optional but useful).
- **What changed** — a brief description of the modification.
- **Why** — the reasoning behind the decision. This is the most valuable part. If there
  were alternatives, briefly note why they were not chosen.
- **Dead ends** — if you tried something that didn't work before landing on this
  solution, record it. This prevents the AI (or another contributor) from trying the
  same failed approach.
- **Side effects** — anything else in the codebase that was affected.

**The critical rule:** The AI reads the relevant changelog *before* modifying a file,
and updates it *only after the user confirms the change works*. This prevents the
changelog from accumulating entries for changes that were later reverted.

See [§10](#10-template-changelog-entry) for a template entry.

---

## 6. User Guide

If your project has an end-user-facing interface (GUI, CLI, API, web app), maintain a
`README.md` that describes what the tool does and how to use it — written for
someone who has never seen the code. Placing user-facing content in `README.md` means
it renders automatically as the project homepage on GitHub or GitLab.

The AI must keep this in sync with the code: any user-visible behaviour change (new
button, renamed command, changed output file, new workflow) triggers a `README.md`
update alongside the code change.

Keep developer and maintainer notes (architecture details, AI assistant instructions,
how to run headlessly, etc.) in an **Appendix** section at the bottom of `README.md`
so end users see clean documentation first and contributors can still find the internals.

**Useful sections to include:**

- Getting started / prerequisites
- Core concepts and terminology
- Step-by-step workflows for the most common tasks
- Reference tables for controls, commands, or settings
- Output files: what is created, where, and what it contains
- A "Things That Are Easy to Forget" table for common gotchas
- A quick-reference section for users who already know the tool

---

## 7. Workflow Rules — What the AI Must Follow

These rules are what make the documentation strategy effective. Include them verbatim
(or adapted) in your `.roo/rules.md`.

### Before modifying any file

1. **Check whether a `CHANGELOG_<name>.md` exists** for that file and read it if so.
   Do not read every changelog upfront — only the one(s) for files you are about to
   change.
2. **Check whether a component reference file exists** for the system or subsystem and
   read the relevant sections.
3. If either contradicts the current source, flag the discrepancy rather than silently
   following one or the other.

### During the change

4. **Implement, do not suggest.** Make changes using tools rather than producing
   code blocks for the human to copy.
5. **Reuse what exists.** Prefer extending existing utilities over re-implementing them.
   Check the reference file for what already exists before writing new code.

### After the change is confirmed

6. **Update the component reference file** — remove any "planned / design only" sections
   that are now implemented; fold new behaviour into the appropriate reference sections.
7. **Add a changelog entry** to `CHANGELOG_<name>.md` (create it if it does not exist).
   Capture *why*, not just *what*.
8. **Update `README.md`** for any user-visible behaviour change.

### What never triggers a documentation update

9. Do **not** update reference files, changelogs, or the user guide until the user
   explicitly confirms the change works. This keeps the documentation in sync with
   verified, working code rather than speculative changes.

---

## 8. Template: AI Instructions File

Copy this into `.roo/rules.md` and fill in the bracketed sections.

```markdown
# Roo Code — Project Instructions

## What this project is

[One paragraph: what problem does this project solve, who uses it, what are its main
subsystems or components?]

## Read these files before touching any code

| File | What it documents |
|------|-------------------|
| `docs/component_a.md` | [One sentence describing what this covers] |
| `docs/component_b.md` | [One sentence] |

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

- [Language version and compiler/interpreter requirements]
- [Architectural pattern: e.g., "all state lives in a single top-level class"]
- [Naming conventions: e.g., "utility functions are prefixed with `util_`"]
- [Toolchain / dependency constraints: e.g., "only use stdlib; no third-party packages"]
- [Things you explicitly do NOT do: e.g., "no global variables", "no dynamic SQL"]

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

- `[output directory]` — generated files
- `[cache directory]` — cached data
- `*.[bak/tmp/auto-save extension]` — backup and temporary files
```

---

## 9. Template: Component Reference File

```markdown
# [Component Name]

> **Status:** [Active development / Stable / Deprecated]  
> **Source file(s):** [`src/component_name.ext`](../src/component_name.ext)  
> **Last reviewed:** [YYYY-MM-DD]

## Purpose

[One paragraph: what does this component do? What is its responsibility boundary?
What does it explicitly *not* do?]

## Dependencies

| Dependency | Why it is needed |
|------------|-----------------|
| [Library or module name] | [One-line reason] |

## Public Interface

### `functionOrMethodName(param1, param2)`

**Purpose:** [What it does]  
**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `param1` | `type` | [Description] |
| `param2` | `type` | [Description] |

**Returns:** `type` — [Description]  
**Side effects:** [File I/O, state mutation, events emitted, etc., or "none"]

[Repeat for each public function / method / endpoint.]

## Internal Design

### Data Structures

| Variable / Property | Type | Description |
|--------------------|------|-------------|
| `internalVar` | `type` | [What it holds and why] |

### Key Flows

**[Flow name — e.g., "Initialisation", "Processing a request", "Saving to disk"]**

1. [Step 1]
2. [Step 2]
3. [Step 3]

[Repeat for each significant flow.]

## Known Constraints and Gotchas

- [Anything that looks wrong but is intentional, or non-obvious behaviour to preserve]
- [Platform quirks, performance constraints, or known edge cases]

## Rejected Approaches

- **[Approach name]** — tried [date or version]; abandoned because [reason]. Do not
  re-introduce.

## Open Issues / To-Do

- [Known limitations that are accepted for now]
```

---

## 10. Template: Changelog Entry

Add new entries at the **top** of the file (newest first).

```markdown
# CHANGELOG — [Source File Name]

---

## [YYYY-MM-DD] — [Short title of the change]

**What changed:**  
[1–3 sentences describing the modification.]

**Why:**  
[The reasoning. Why was this approach chosen? What problem did it solve?
If there were alternatives, briefly note why they were not chosen.]

**Dead ends (if any):**  
- Tried [approach X] first — did not work because [reason].
- Considered [approach Y] — rejected because [reason].

**Side effects:**  
[Other files or behaviours affected, or "none".]

---

## [YYYY-MM-DD] — [Previous entry title]

...
```

---

## 11. Maintenance Checklist

Use this after every confirmed code change.

### Per change

- [ ] Changelog entry added to `CHANGELOG_<modified_file>.md` (create if absent)
- [ ] Component reference file updated for any new/changed interface, data structure,
      or design decision
- [ ] `README.md` updated if any user-visible behaviour changed
- [ ] `.roo/rules.md` updated if new files were added that the AI should read, or if
      conventions changed

### Periodically (e.g., each milestone or release)

- [ ] Reference files reviewed for accuracy — code may have drifted from prose
- [ ] "Open Issues / To-Do" sections reviewed and pruned or promoted to issues
- [ ] `.roo/rules.md` still accurately describes the project scope and conventions
- [ ] `.roo/rules.md` changelog section updated if new source files have been added to
      the project (so the AI knows changelogs may exist for them)

---

## 12. GitHub Copilot Compatibility

This project's AI instructions are written for **Roo Code**, which reads `.roo/rules.md`
automatically. If you are sharing this repository with collaborators who use
**GitHub Copilot**, you can make the same instructions available to Copilot with a
one-time setup step.

### How GitHub Copilot loads project instructions

GitHub Copilot Chat automatically includes the contents of
`.github/copilot-instructions.md` in every chat session for the repository. This is
Copilot's equivalent of Roo Code's `.roo/rules.md`.

### Option A — Copy the file (simplest)

Copy `.roo/rules.md` to `.github/copilot-instructions.md`:

```bash
mkdir -p .github
cp .roo/rules.md .github/copilot-instructions.md
```

Commit both files. Keep them in sync manually when the instructions change.

### Option B — Symlink (Unix/macOS only)

```bash
mkdir -p .github
ln -s ../.roo/rules.md .github/copilot-instructions.md
```

This keeps a single source of truth. Note that symlinks may not resolve correctly on
Windows or in some Git hosting environments.

### Option C — Reference from `copilot-instructions.md`

Create `.github/copilot-instructions.md` with a pointer to the Roo Code file:

```markdown
# GitHub Copilot — Project Instructions

The full project instructions are maintained in `.roo/rules.md`.
Please read that file before proceeding.

[Paste the full contents of .roo/rules.md here, or instruct Copilot to read it.]
```

### Differences between Roo Code and GitHub Copilot instruction files

| Feature | Roo Code (`.roo/rules.md`) | GitHub Copilot (`.github/copilot-instructions.md`) |
|---------|---------------------------|---------------------------------------------------|
| Loaded automatically | ✓ Every session | ✓ Every Copilot Chat session |
| File location | `.roo/rules.md` | `.github/copilot-instructions.md` |
| Supports multiple rule files | ✓ (`.roo/rules-*.md` pattern) | ✗ Single file only |
| Mode-specific rules | ✓ (per Roo mode) | ✗ |
| Markdown format | ✓ | ✓ |

### Keeping both files in sync

If you maintain both files, add a note at the top of each:

In `.roo/rules.md`:
```markdown
> **Note:** A copy of these instructions for GitHub Copilot is maintained at
> `.github/copilot-instructions.md`. Keep both files in sync when updating.
```

In `.github/copilot-instructions.md`:
```markdown
> **Note:** These instructions are mirrored from `.roo/rules.md` (Roo Code).
> Keep both files in sync when updating.
```

---

## Why This Works

The strategy is effective because it separates three kinds of knowledge:

| Kind | Where it lives | How it ages |
|------|---------------|-------------|
| Current code state | Source files | Always current |
| Current design intent | Reference `.md` files | Updated on confirmation |
| Historical decisions and dead ends | `CHANGELOG_*.md` files | Append-only, never deleted |

An AI working within this system can answer "what should this function do?", "what has
already been tried?", and "what are the conventions?" without reading the full codebase.
It knows when to act and when to flag a discrepancy. And because documentation is only
updated after confirmation, it stays accurate rather than accumulating speculative content.

The overall cost of maintenance is low: a changelog entry takes two to five minutes to
write. The payoff — not spending thirty minutes walking a new AI session back from a
confidently wrong change — outweighs that cost quickly.
