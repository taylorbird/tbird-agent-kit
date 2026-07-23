# tbird-codereview — PR Code-Review Team Plugin Design

**Date:** 2026-07-22
**Author:** Taylor Bird
**Status:** Design complete — ready for implementation
**Repo:** https://github.com/taylorbird/tbird-agent-kit (same marketplace as `tbird-claudetools`)

## Goal / origin

Formalize a prompt Taylor has been pasting into many projects: spin up a
coordinated code-review team on an open PR, gate merge on unanimous agreement,
and watch for a CodeRabbit review. Turning it into a skill preserves the
structure (reviewer personas, async CodeRabbit wait, unanimous merge gate) so it
doesn't get re-typed or accidentally varied per project.

Existing ecosystem review tools (`superpowers:requesting-code-review`,
`superpowers:code-reviewer`, `code-review:code-review`, the local `push-review`)
cover single-reviewer passes but none do the multi-persona team + CodeRabbit
watch + unanimous-gate + human-escalation combination. That combination is why
this warrants its own skill.

## Placement

New plugin **`tbird-codereview`** in the **same** `tbird-agent-kit` monorepo /
marketplace — not a 5th skill in `tbird-claudetools` (which stays focused on
session-continuity), and not a separate repo.

```
tbird-agent-kit/
├── .claude-plugin/
│   └── marketplace.json          ← now lists BOTH plugins
└── plugins/
    ├── tbird-claudetools/        ← existing (session skills)
    └── tbird-codereview/         ← new
        ├── .claude-plugin/plugin.json
        └── skills/code-review-team/SKILL.md
```

Install: `/plugin install tbird-codereview@tbird-agent-kit`.
Invocation (namespaced, auto-invocation also available):
`/tbird-codereview:code-review-team`.

## The review team (5 reviewers)

All five are framed as **senior engineers** — seniority is the baseline; they
differ by *lens* and by *what context they receive*. Run as independent
subagents in round 1 (no cross-talk — independence is the value).

1. **Requirements** — full context (brief + diff). Did the diff meet the goal /
   the stated requirement?
2. **General quality & correctness** — full context. Open-ended senior review:
   bugs, edge cases, error handling, design, maintainability. (Fills the gap the
   narrow lenses miss; CodeRabbit covers some mechanically but without human
   design judgment.)
3. **Simplicity / anti-over-engineering** — full context. Flags over-complexity
   and "overly-AI" patterns (needless reflection, cleverness, non-idiomatic
   constructs); expects a standard Node style.
4. **Cold sanity-check** — **diff only, NO brief.** Deliberately context-starved
   so it reacts to the code cold: "does anything look crazy?" Pairs with #2
   (one senior who knows the goal, one who deliberately doesn't).
5. **CodeRabbit watcher** — polls the PR until CodeRabbit's review lands.
   Blocking peer (see policy below).

## Coordinator = the main thread (not a 5th persona subagent)

The orchestrating brain is the **main thread**, because only the main thread can
hand Taylor the PR link, ask him questions, and clear him to merge — headless
subagents return output up to the orchestrator and cannot reach the user. A
dedicated synthesis subagent was considered but adds parts without removing the
main-thread dependency (its escalations would still have to be relayed).

The main thread: composes the brief, launches the reviewers, collects verdicts,
dedups findings, runs the one reconciliation round on any conflict, and
escalates to Taylor when unresolved.

## Control flow

1. **Precondition.** Find the open PR for the current branch (`gh pr view`).
   **No PR found → STOP and ask Taylor** what to do (open one? different
   branch?). The skill never opens a PR on its own. PR found → print the link.
2. **Compose the review brief** (main thread) — a dense statement of what the
   work was supposed to accomplish, plus how to obtain the diff. Reviewers are
   headless; they only know what's in their prompt.
3. **Launch the 5 reviewers in parallel.**
   - Reviewers 1–3 receive the brief **and** the diff.
   - Reviewer 4 (cold) receives the **diff only** — no brief.
   - Reviewer 5 (CodeRabbit watcher) begins polling.
4. **Collect verdicts.** Each returns a structured result: `ready | not-ready`,
   a list of **blockers**, and a list of non-blocking **nits**. Wait for all
   five (CodeRabbit within its timeout).
5. **Decide — three exits:**
   - **All `ready`** → clear Taylor to merge (print link + merge command).
     *Merge is always Taylor's action; the skill does not merge.*
   - **All agree `not-ready`** (consensus on real issues) → report the combined,
     deduped blockers; no merge. Taylor fixes and re-runs the skill.
   - **Disagreement** (one blocks what another passes) → **one reconciliation
     round**: show the disagreeing reviewers each other's reasoning + the
     specific code; each re-judges once; the coordinator adjudicates.
6. **Still split after reconciliation** → escalate to Taylor with both positions
   and a recommendation. Taylor decides.

## CodeRabbit watcher policy

- **Blocking peer.** Its findings must be resolved like any reviewer's; the gate
  does not clear without it.
- **Timeout → escalate.** If CodeRabbit does not post within a timeout
  (default ~15 min, tunable), the coordinator asks Taylor how to proceed rather
  than merging blind.
- **Mechanism (implementation detail):** poll the PR via `gh` on an interval for
  CodeRabbit's review/comments until present or timeout.

## Reviewer I/O

- **Diff sourcing:** main thread obtains the diff (`gh pr diff`, or `git diff`
  against the PR base) and passes it into each reviewer's prompt; reviewers may
  read specific files if they need more than the diff shows.
- **Output schema (per reviewer):** `{ verdict: "ready" | "not-ready",
  blockers: [...], nits: [...], notes: "..." }`.
- **Dedup:** the coordinator merges overlapping findings across reviewers
  (quality + simplicity + CodeRabbit will overlap) before reporting.

## Edge / error handling

- **No PR** → stop and ask (see flow step 1).
- **A reviewer subagent crashes** → coordinator reports that lens failed; does
  NOT silently treat it as a pass.
- **CodeRabbit timeout** → escalate to Taylor.
- **Large diff** → reviewers get the diff and can read specific files for depth.

## Open implementation questions (resolve during writing-plans / build)

- Exact CodeRabbit polling mechanism (background loop vs. Monitor tool) and how
  the watcher subagent (or main thread) sleeps between polls.
- Whether the parallel review round uses the Workflow tool or direct Agent-tool
  fan-out. (Coordination/escalation stays on the main thread regardless.)
- Default CodeRabbit timeout value and whether it's surfaced as a tunable.

## Out of scope (YAGNI, for now)

- Auto-merge (Taylor always merges himself).
- Opening PRs from the skill.
- Inter-agent live debate (round 1 is independent; reconciliation is a single
  bounded round).
- Configurable reviewer roster / adding-removing lenses via config.
