---
name: systematic-debugging
description: Use when hitting a bug, test failure, or unexpected behavior and the cause isn't already established — find the root cause before proposing fixes. Triggers on "debug this", "why is this failing", "/debug".
---

# Systematic Debugging

Random fixes waste time and create new bugs. Find the root cause before
attempting fixes — symptom patches are failure. This holds most under time
pressure, when a "quick fix" seems obvious, or when previous fixes didn't
stick. (Adapted from obra/superpowers, MIT.)

## Phase 1 — Root cause investigation (before any fix)

- Read the error output completely: full stack traces, line numbers, codes.
- Reproduce reliably; if you can't, gather more data rather than guessing.
- Check what changed: git diff, recent commits, dependencies, config, environment.
- In multi-component systems, instrument each boundary (log what enters and
  exits each layer, verify env/config propagation) and run once to see WHERE
  it breaks before theorizing about why.
- Trace bad values backward to their origin; fix at the source, not where the
  symptom surfaced.

## Phase 2 — Pattern analysis

Find similar working code in the same codebase and list every difference from
the broken path, however small. If implementing a reference pattern, read the
reference completely before adapting it.

## Phase 3 — Hypothesis and test

State one specific hypothesis ("X is the root cause because Y"), test it with
the smallest possible change, one variable at a time. Didn't hold? Form a new
hypothesis — don't stack more changes on top. If you don't understand
something, say so and investigate rather than pretending.

## Phase 4 — Implement

Create the simplest failing reproduction (automated test if possible) before
fixing, then make one fix addressing the root cause — no bundled refactoring
or "while I'm here" changes — and verify the reproduction passes and nothing
else broke.

**Escalation rule:** after ~3 failed fix attempts, stop treating it as a bug.
Each fix revealing a new problem elsewhere signals an architectural issue —
question the pattern with the user instead of attempting fix #4.

If investigation genuinely ends at an environmental/timing/external cause,
document what was ruled out and implement appropriate handling (retry,
timeout, monitoring) — but most "no root cause" conclusions are incomplete
investigation.
