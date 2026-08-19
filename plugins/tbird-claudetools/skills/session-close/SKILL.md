---
name: session-close
description: Use when the user is about to quit Claude Code and wants everything wound down safely — stops background agents, tasks, and processes (dev servers etc.), offers a checkpoint if the last one is stale or the session is heavy, then gives the all-clear to exit. Triggers on "close out", "shut everything down", "wind down", "I'm done for now", "before I quit", "/session-close".
---

Wind the session down so the user can exit Claude Code cleanly. Ask all
questions as plain text in the conversation — never via AskUserQuestion.

## 1. Inventory what's still running

Build one list from all of these sources:
- Background subagents and tasks you launched this session that haven't
  completed (agents, workflows, background Bash shells).
- Long-lived processes started during this session: dev servers, watchers,
  tunnels. Check what you remember launching, then verify with `ps`/`lsof`
  (e.g. `lsof -nP -iTCP -sTCP:LISTEN` for servers you started).
- Scheduled/recurring things created this session (loops, monitors, crons) —
  list them but do NOT cancel crons without explicit instruction; they may be
  intentionally persistent.

If nothing is running, say so and skip to step 3.

## 2. Confirm, then stop

Show the list — one line each: what it is, what it's doing, and what stopping
it means (e.g. "vite dev server on :5173 — page goes down", "review agent
mid-task — its findings are lost"). Ask in plain text: "Stop all of these?
Anything you want left running?"

On confirmation, stop everything approved: TaskStop for tracked tasks/agents,
`kill` for PIDs you started. Leave anything the user excluded, and anything
you didn't start (never kill processes from outside this session — if in
doubt, list it as "not mine, leaving alone"). Re-check afterwards and report
anything that survived.

## 3. Checkpoint gate

Read `.claude/work/current.md` → Last Checkpoint. Offer to run
/session-checkpoint (plain-text yes/no) if ANY of:
- No work-state files or no Last Checkpoint value.
- Last Checkpoint is more than 30 minutes ago.
- The session did substantial work after the last checkpoint (large context,
  many edits/decisions) — judge this yourself; when in doubt, offer.

If the checkpoint is fresh and the session added nothing since, skip the
offer and say so ("checkpointed 10 min ago, nothing new since").

If the user accepts, run the checkpoint and WAIT for its subagent to complete
before proceeding — the all-clear must not race the checkpoint write.

## 4. All-clear

Confirm in one short block: what was stopped, what was left running (and
why), checkpoint status. End with: "All clear — safe to exit."

(You cannot exit Claude Code yourself; the user quits with /exit or Ctrl+C.)
