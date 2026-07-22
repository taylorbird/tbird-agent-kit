---
name: session-checkpoint
description: Use before ending a session, taking a break, or when the user wants to save session work-state — updates current.md, log.md, questions.md, learnings, and candidate preferences via a single Sonnet subagent. Triggers on "checkpoint", "save session state", "before I go", "/session-checkpoint".
---

Checkpoint the session by handing one dense brief to a **single Sonnet
subagent** (the Agent tool, `model: sonnet`) that updates every work-state file
sequentially. One agent, one copy of the brief — the file targets are disjoint
and small, so parallelism would only buy a few seconds of wall-clock at the cost
of duplicating the brief across several agent contexts.

## 1. Compose the session brief (main thread — you have the context, the agent doesn't)

Write one dense brief covering this session: what was worked on, decisions
made (with rationale), actions taken, findings, anything future sessions need,
new/resolved open questions, and any durable technical learnings discovered
(API behaviors, integration patterns, gotchas, things that work). Also note any
candidate standing preferences or observations about how the user likes to work
that you picked up this session (distinct from technical learnings) — for the
preferences file. Include the current date/time (run `date "+%Y-%m-%d %H:%M %Z"`).

The agent receives ONLY what you put in its prompt — every fact it needs must
be in the brief. Do not assume it can see the conversation.

## 2. Build the per-file instructions

Include instructions for each file that needs updating (tell the agent to skip
any file with nothing to change):

- **`.claude/work/current.md`**: Read the file first. Keep Project/Objective
  (update only if refined). Preserve the Constraints section — add any new hard
  rule / guardrail surfaced this session; never silently drop one (create the
  section if the file predates it and a constraint exists). Rewrite Current
  Focus to what we were just working on, set Last Checkpoint to the provided
  date/time, list 2–5 concrete Next Actions in priority order (carry forward
  still-relevant ones).
- **`.claude/work/log.md`**: Prepend (never overwrite or modify existing
  entries) an entry:

  ```
  ## {YYYY-MM-DD HH:MM}

  **Session Summary**: One paragraph of what we did

  **Decisions Made**:
  - {decision}: {rationale} (or "None")

  **Actions Taken**:
  - {what was done}

  **Context/Thoughts**:
  - {anything important for future sessions}
  ```
- **`.claude/work/questions.md`**: Read the file first. Add new open
  questions, move resolved ones to the Resolved section, note blockers.
- **`.claude/learnings/{topic}.md`**: Only if the brief contains durable
  technical learnings. Create or update one file per topic; check for an
  existing file on the topic before creating a new one.
- **`.claude/work/preferences.md`**: Only if you noticed candidate standing
  preferences or observations about how the user likes to work this session
  (NOT technical learnings — those go in `.claude/learnings/`). Append dated
  entries below the marker line, newest first; never overwrite existing
  entries. Do NOT ask the user to verify them and do NOT promote them to
  confirmed rules — this file is the user's to review.

Capture ALL learnings AND candidate preferences you identified yourself; do NOT
ask the user whether there are any to capture, and do NOT ask them to verify the
ones you captured (they review the preferences file on their own).

## 3. Run the checkpoint agent

**Before launching, announce the model on its own line so the user can catch
it:** `> Checkpoint agent running on model: sonnet`. This is informational only —
do not change or remove the model pin; just surface it every run.

Launch **one** subagent with the Agent tool, `model: sonnet`. Its prompt =
the full session brief + the per-file instructions from step 2 + the absolute
path of each file to update. Instruct it to:

- Read each target file before editing it.
- Update the files sequentially (they're disjoint — order doesn't matter).
- End by reporting, per file, what it changed (or that it skipped it).

## 4. Confirm

When the agent returns, check its per-file report (fix or re-run any file it
failed to update yourself), then say:
"Checkpointed. Next session: {first next action}"

If you appended anything to `.claude/work/preferences.md` this run, list those
additions in the closing summary (one line each) so the user sees what was
captured without having to open the file.
