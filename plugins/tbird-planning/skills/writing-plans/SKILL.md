---
name: writing-plans
description: Use when you have a spec or agreed design for a multi-step task, before touching code — produces an implementation plan a zero-context engineer could execute. Triggers on "write the plan", "implementation plan", "/write-plan".
---

# Writing Plans

Write an implementation plan assuming the executor is skilled but has zero
context for this codebase and its domain. Document which files each task
touches, the code itself, how to test it, and any docs to check. Bite-sized
tasks; DRY; YAGNI; test-first; frequent commits. (Adapted from
obra/superpowers, MIT.)

**Save plans to:** `docs/plans/YYYY-MM-DD-<feature-name>.md`

## Plan header

```markdown
# [Feature Name] Implementation Plan

**Goal:** [one sentence]

**Architecture:** [2–3 sentences]

**Tech Stack:** [key technologies/libraries]
```

## Task structure

Per task: exact file paths (create / modify with line ranges / test), the
actual code (not "add validation"), the exact test commands with expected
output, and a commit step. A task should be a small, independently
verifiable unit — write the failing test, make it pass, commit.

## Execution handoff

After saving the plan, ask how to execute:

1. **Directly in this session** — work through tasks in order, or fan
   independent tasks out to parallel subagents (cheap models for mechanical
   tasks; background agents so nothing blocks).
2. **Separate session/worktree** — for large plans or when this session's
   context is heavy; the plan file is the handoff, so it must stand alone.

Don't add per-task review-agent gates by default — 5-gen models self-verify;
reserve independent review for production-bound or high-risk changes, or when
the user asks.
