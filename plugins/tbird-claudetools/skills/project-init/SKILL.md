---
name: project-init
description: Use when starting a new project or initializing project work-state — scaffolds the .claude/ tree if missing, then sets Project/Objective/Constraints and starts the work log. Triggers on "start a new project", "init this project", "set up work state", "/project-init".
---

Starting a new project. Set up the work state.

## 0. Scaffold `.claude/` if missing

Create any directory or seed file that is **absent**. If a file already
**exists**, don't overwrite it — show the user the path and ask first (it may
hold real work).

Directories: `.claude/work/`, `.claude/rules/`, `.claude/learnings/`, `.claude/archive/`.

**`.claude/CLAUDE.md`**
```markdown
# Project Instructions

## Session Management
Active work state: @.claude/work/current.md

On session start, read .claude/work/current.md to understand current state.
Use /session-checkpoint before ending sessions, /session-resume to continue work.

## Durable knowledge
- .claude/work/constraints.md — durable decisions and hard rules (full ledger)
- .claude/rules/ — path-scoped rules, auto-loaded when matching files are touched
- .claude/learnings/ — technical gotchas by topic (API quirks, integration gotchas, patterns that work)

## Project-Specific Context
<!-- Project-specific instructions only. No generic best practices — they cost
     context and the model already knows them. -->
```

**`.claude/work/current.md`** — keep this file under ~60 lines at all times;
detail lives in constraints.md / rules / learnings, history in log.md.
```markdown
# Current State

## Project
(none active)

## Objective
(not set)

## Constraints
Full ledger: .claude/work/constraints.md. Path-scoped rules: .claude/rules/.

(one-liners for the always-on hard rules go here)

## Current Focus
(none)

## Last Checkpoint
(none)

## Next Actions
(none)
```

**`.claude/work/constraints.md`**
```markdown
# Constraints Ledger

Durable decisions and hard rules, organized by topic, each with date and
rationale. One-line summaries of the always-on rules live in current.md;
path-scoped rules live in .claude/rules/ and are not duplicated here.
```

**`.claude/work/log.md`**
```markdown
# Work Log

<!-- Entries prepended, newest first -->
```

**`.claude/work/questions.md`**
```markdown
# Open Questions

<!-- Things uncertain, need revisiting, or blocked on -->
```

**`.claude/learnings/README.md`**
```markdown
# Learnings

Durable, reusable technical knowledge discovered during this project — one
file per topic (e.g. `stytch-m2m.md`). Capture what you'd want to know
starting fresh: API quirks, gotchas, patterns that work. Concise reference,
not a journal. Updated during /session-checkpoint; consolidated on
/project-close.
```

**`.gitignore`** — if it exists and does not already contain `.claude/work/`, append:
```
# Claude Code work state (optional - remove if you want to track)
# .claude/work/
```
If it does not exist, create it with those two lines.

## 1. Check for existing work

Read `.claude/work/current.md`. If there's an active project (Project field is
not "(none active)"), ask whether to archive it first with /project-close or
abandon it.

## 2. Initialize work files

Update `.claude/work/current.md`: set Project (from argument), Objective
(extract or ask, 1–2 sentences), Constraints (one bullet per hard rule stated
for this project, or "None yet."), Current Focus ("Initial planning and
setup"), Last Checkpoint (current date/time + "Project initialized"), and 2–3
suggested Next Actions based on the project description.

## 3. Start the log

Prepend to `.claude/work/log.md`:
```
## {YYYY-MM-DD HH:MM} - Project Started

**Project**: {name}
**Objective**: {objective}

**Initial Context**:
- {any context provided}

**Planned Approach**:
- {high-level approach if discernible}
```

## 4. Clear questions

Reset `.claude/work/questions.md` to empty (keep header).

## 5. Confirm

Tell me: "Project '{name}' initialized. Ready to start with: {first next action}"
