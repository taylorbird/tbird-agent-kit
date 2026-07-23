---
name: code-review-team
description: Use when the user wants a coordinated code-review team on an open PR before merging — launches independent senior-engineer reviewers plus a CodeRabbit watcher, gates merge on unanimous agreement, reconciles disagreements once, and escalates to the user when unresolved. Triggers on "code review team", "review this PR", "spin up a review team", "is this ready to merge", "/code-review-team".
---

Run a coordinated code-review team on the current branch's open PR. You (the main
thread) are the **coordinator** — only you can hand the user the PR link, ask
questions, and clear them to merge. The reviewers are independent subagents; they
report to you, not to each other and not to the user.

## 1. Precondition — find the open PR (do NOT open one)

Find the open PR for the current branch:

```
gh pr view --json number,url,title,baseRefName,headRefName
```

- **No open PR found → STOP.** Tell the user there's no open PR for this branch
  and ask what they want to do (open one? point at a different branch?). Do
  **not** open a PR yourself — that's outside this skill's job.
- **PR found → print the PR URL to the user immediately**, then continue.

## 2. Compose the review brief (main thread — you have the context)

Write one dense brief the reviewers will each receive (except the cold reviewer,
step 3 #4): what this work was supposed to accomplish (the requirement/goal),
relevant constraints, and any context a reviewer needs to judge whether the diff
meets the goal. Reviewers are headless — they know ONLY what you put in their
prompt. If work-state exists, pull the goal from `.claude/work/current.md`.

Get the diff for reviewers to examine:

```
gh pr diff <number>
```

Reviewers may also read specific files in the repo for depth beyond the diff.

## 3. Launch the 5 reviewers

Launch reviewers 1–4 as **independent subagents in parallel** (multiple Agent
tool calls in one message so they run concurrently). Start the CodeRabbit watch
(#5) concurrently per step 4. **Round 1 has no cross-talk** — independence is the
whole point.

Every reviewer is a **senior engineer** — experienced but practical. They differ
by lens and by what context they get. Each must return the exact schema in step 5.

1. **Requirements** (brief + diff): Does the diff actually meet the stated goal /
   requirement? Missing pieces, scope gaps, requirement not satisfied.
2. **General quality & correctness** (brief + diff): Open-ended senior review —
   correctness bugs, unhandled edge cases, error handling, race conditions, data
   flow, design, maintainability. The broad "is this good, correct software?" pass.
3. **Simplicity / anti-over-engineering** (brief + diff): Flag over-complexity and
   "overly-AI" patterns — needless abstraction, cleverness, reflection/metaprogramming
   where plain code would do, non-idiomatic constructs. Expect a **standard Node**
   style; call out anything that isn't boring and obvious where it could be.
4. **Cold sanity-check** (**diff ONLY — do NOT give this one the brief**):
   A senior engineer seeing the code cold, with no idea what it's "supposed" to
   do. Purely: "does anything here look crazy, surprising, or wrong on its face?"
   This lens exists to catch what knowing-the-goal blinds the others to.

## 4. CodeRabbit watcher (blocking peer)

CodeRabbit reviews the PR asynchronously — usually minutes after open, sometimes
not at all. Poll for its review, up to a **15-minute timeout** (tunable):

```
gh api repos/{owner}/{repo}/pulls/<number>/reviews
gh pr view <number> --comments
```

Look for the `coderabbitai` bot's review/summary. Poll on an interval (e.g. every
~45–60s) — use the Monitor tool or a background poll loop; do not busy-wait.

- **CodeRabbit posts → treat its findings as a 5th reviewer verdict** (map its
  blocking comments to `blockers`, suggestions to `nits`; `not-ready` if it raised
  anything it considers blocking, else `ready`).
- **Timeout with no CodeRabbit review → STOP and ask the user** how to proceed
  (wait longer? proceed without it? it's not configured on this repo?). Do not
  clear the merge without resolving this.

## 5. Collect verdicts (per-reviewer schema)

Each reviewer (including the CodeRabbit mapping) yields:

```json
{
  "lens": "requirements | quality | simplicity | cold | coderabbit",
  "verdict": "ready | not-ready",
  "blockers": ["specific issue that must be fixed before merge", "..."],
  "nits": ["non-blocking suggestion", "..."],
  "notes": "one-line summary of this reviewer's take"
}
```

Wait for all five. If a reviewer subagent **crashes or returns nothing, report
that lens as failed** — never silently treat a missing reviewer as a pass.

## 6. Coordinate — dedup, then take one of three exits

**Dedup first:** quality, simplicity, and CodeRabbit will overlap. Merge findings
that are the same issue so the user sees each problem once.

- **All five `ready`** → **clear the user to merge.** Report the deduped nits (if
  any) as optional, print the PR URL and the merge command, and state plainly that
  all reviewers agree it's ready. **Do not merge yourself — merging is the user's
  action.**
- **All agree `not-ready`** (consensus there are real problems) → report the
  deduped **blockers** grouped sensibly, no merge. Tell the user to fix and re-run
  the skill.
- **They disagree** (at least one `not-ready` while another passes the same area,
  or a blocker one reviewer raised that another explicitly considered fine) → go
  to step 7.

## 7. Reconciliation — one bounded round, then escalate

For each genuine disagreement, run **one** reconciliation pass: give the
disagreeing reviewers each other's specific reasoning **and** the exact code in
question, and ask each to either hold their position (with justification) or
concede (with justification). Re-judge **once** — do not loop.

Then adjudicate:

- **Reconciled** (they converge) → apply the resolved verdict and return to the
  matching exit in step 6 (clear to merge, or report agreed blockers).
- **Still split after one round** → **escalate to the user.** Present the specific
  disagreement, each reviewer's position in their own words, and your
  recommendation as coordinator. The user decides. Merge remains their action.

## Coordinator principles

- The reviewers are independent in round 1 — never let them see each other's
  findings before they've each judged.
- Every exit ends with the user, never with an automatic merge.
- When you stop to ask the user (no PR, CodeRabbit timeout, unresolved split),
  be specific about what you need from them.
