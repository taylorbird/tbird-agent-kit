---
name: session-checkpoint
description: Use before ending a session, taking a break, or when the user wants to save session work-state — updates current.md, constraints.md, log.md, questions.md, and learnings via a single subagent. Triggers on "checkpoint", "save session state", "before I go", "/session-checkpoint".
---

Checkpoint the session by handing one dense brief to a single subagent
(Agent tool, `model: haiku`). The brief is composed on the main thread — the
agent only executes file edits against it, so a small model is enough.

## 1. Compose the session brief (main thread — you have the context, the agent doesn't)

One dense brief covering: what was worked on, decisions made (with rationale),
actions taken, findings, new/resolved open questions, new durable constraints,
and durable technical learnings (API behaviors, integration gotchas, patterns
that work). Include the current date/time (`date "+%Y-%m-%d %H:%M %Z"`). The
agent sees only this brief, not the conversation — every fact it needs must be
in it.

Standing user preferences and observations about how the user likes to work are
not part of this checkpoint — Claude Code's built-in auto-memory owns those;
save them yourself from the main thread.

## 1b. First checkpoint in a repo? Scaffold what's missing

If `.claude/work/current.md` doesn't exist, create the tree before running the
agent (never overwrite anything that exists):

- Directories: `.claude/work/`, `.claude/rules/`, `.claude/learnings/`, `.claude/archive/`
- `.claude/work/current.md` — section skeleton: Project / Objective /
  Constraints (start with "Full ledger: .claude/work/constraints.md. Path-scoped
  rules: .claude/rules/.") / Current Focus / Last Checkpoint / Next Actions.
  The first checkpoint fills Project and Objective from what this session
  established.
- `.claude/work/constraints.md` — header: "# Constraints Ledger" + one line
  noting entries carry date + rationale and one-liners live in current.md
- `.claude/work/log.md` — "# Work Log" + "<!-- Entries prepended, newest first -->"
- `.claude/work/questions.md` — "# Open Questions"
- `.claude/CLAUDE.md` — if absent, create:

  ```markdown
  # Project Instructions

  ## Session Management
  Active work state: @.claude/work/current.md

  On session start, read .claude/work/current.md to understand current state.
  Use /session-checkpoint before ending sessions. Use /session-resume to continue work.

  ## Durable knowledge
  - .claude/work/constraints.md — durable decisions and hard rules (full ledger)
  - .claude/rules/ — path-scoped rules, auto-loaded when matching files are touched
  - .claude/learnings/ — technical gotchas by topic

  ## Project-Specific Context
  <!-- Project-specific instructions only. No generic best practices. -->
  ```
- If `.gitignore` exists without a `.claude/work/` mention, append the
  commented opt-out (`# .claude/work/` under a "Claude Code work state" note)
  so the user can choose whether to track work state.

## 2. Per-file instructions (tell the agent to skip any file with nothing to change)

**Size contract: `.claude/work/current.md` stays under ~60 lines.** This skill
exists for fast session resume, and that dies if current.md bloats. Nothing
"retained as history" accumulates there — history belongs in log.md, durable
detail in constraints.md, `.claude/rules/`, or learnings.

- **`.claude/work/current.md`**: rewrite in place. Keep Project/Objective
  (update only if refined). Constraints stay a short list of one-liners
  pointing at `.claude/work/constraints.md`, `.claude/rules/`, or a learnings
  file — a new hard rule gets a one-liner here plus its full entry in
  constraints.md; never silently drop one. Rewrite Current Focus to this
  session only. Set Last Checkpoint to the provided date/time. List 2–5 Next
  Actions in priority order (carry forward the still-relevant; drop completed
  ones — log.md records what was done).
- **`.claude/work/constraints.md`**: full entries (date + rationale) for new
  durable constraints, organized by topic; amend or mark superseded entries
  rather than deleting them. Create the file if missing.
- **`.claude/work/log.md`**: prepend (never modify existing entries):

  ```
  ## {YYYY-MM-DD HH:MM}

  **Session Summary**: one paragraph of what we did

  **Decisions Made**:
  - {decision}: {rationale} (or "None")

  **Actions Taken**:
  - {what was done}

  **Context/Thoughts**:
  - {anything future sessions need}
  ```
- **`.claude/work/questions.md`**: add new open questions, move resolved ones
  to the Resolved section, note blockers.
- **`.claude/learnings/{topic}.md`**: durable technical learnings only, one
  file per topic; extend an existing topic file before creating a new one.

## 3. Run the checkpoint agent

Launch one Agent-tool subagent with `model: haiku`. Its prompt = the session
brief + the per-file instructions + the absolute path of each file. Have it
read each file before editing, honor the size contract, and end with a
per-file report of what changed (or that it skipped the file).

## 4. Confirm

Check the per-file report — fix anything it got wrong yourself, including the
current.md size contract — then say:
"Checkpointed. Next session: {first next action}"
