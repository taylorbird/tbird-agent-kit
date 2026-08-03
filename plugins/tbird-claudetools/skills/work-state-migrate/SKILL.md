---
name: work-state-migrate
description: Use when a repo's .claude/ work-state predates the Claude 5 restructure — an oversized @-imported current.md, boilerplate standards files, or a preferences.md — and should be migrated to the current shape (small current.md + constraints.md + path-scoped rules + auto-memory). Triggers on "migrate work state", "modernize the claude files", "restructure .claude", "/work-state-migrate".
---

Migrate this repo's `.claude/` work-state to the current shape. The goal:
`current.md` under ~60 lines, durable detail split into `constraints.md` /
`.claude/rules/` / learnings, boilerplate gone, preferences owned by
auto-memory. Do the triage on the main thread — it requires judgment about
what's still true.

## 1. Inventory first

Measure before touching anything: `wc -w` every file `@`-imported by
`.claude/CLAUDE.md` plus everything in `.claude/work/` and `.claude/standards/`.
Report the totals to the user so the before/after is visible at the end.

## 2. Archive the originals

Copy `current.md` (and `preferences.md` if present) to `.claude/archive/`
with a dated name (e.g. `current-YYYY-MM-full.md`) before rewriting. Nothing
gets deleted that isn't preserved there or in git history.

## 3. Split current.md

Triage every constraint / focus block / next action for what it is NOW —
verify anything that looks stale against the code or the live system rather
than copying it forward:

- **File-scoped rules** (constraints about specific source files or flows) →
  `.claude/rules/<topic>.md` with `paths:` YAML frontmatter listing the
  matching globs, so they load only when those files are touched.
- **Durable, non-path-scoped decisions** → `.claude/work/constraints.md`,
  organized by topic, each entry keeping its date and rationale. Superseded
  entries stay only in the archive.
- **History blocks and completed work** → they're already in the archive copy;
  log.md is their living record. Do not carry them forward.
- **Rewrite `current.md` under ~60 lines**: Project / Objective / Constraints
  (one-liners pointing at the ledger, rules, and learnings) / Current Focus
  (latest state only) / Last Checkpoint / Next Actions (live items only, with
  a pointer to the backlog's home).

## 4. Remove boilerplate standards

If `.claude/standards/` files are unedited scaffolding (look for the
"Customize for your organization" marker and generic content), delete the
directory and the `@.claude/standards/` imports from `.claude/CLAUDE.md`.
If a file has real, project-specific edits, keep those lines — move them into
`constraints.md` or a rule file and delete the rest.

## 5. Retire preferences.md

If `.claude/work/preferences.md` exists: save the durable entries to Claude
Code auto-memory (one memory file per preference, with why + how to apply;
update MEMORY.md's index), then delete the file — the archive copy from step 2
keeps the full history. Skip entries auto-memory already covers.

## 6. Update .claude/CLAUDE.md

Keep the `@.claude/work/current.md` import (it's small now). Replace any
standards section with a "Durable knowledge" section pointing at
`constraints.md`, `.claude/rules/`, and `.claude/learnings/`.

## 7. Report

Show before/after word counts for the always-loaded set, list every file
created/rewritten/deleted, and flag anything you couldn't confidently triage
for the user to decide. Leave committing to the user.
