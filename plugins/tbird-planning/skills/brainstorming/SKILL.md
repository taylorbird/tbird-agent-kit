---
name: brainstorming
description: Use when starting feature or design work and the requirements aren't fully pinned down — turns an idea into a validated design through one-question-at-a-time dialogue before any implementation. Triggers on "let's design", "brainstorm", "new feature", "/brainstorm".
---

# Brainstorming Ideas Into Designs

Turn an idea into a fully formed design through natural collaborative
dialogue. (Adapted from obra/superpowers, MIT.)

**Understanding the idea:**
- Check the current project state first (files, docs, recent commits, work state).
- Ask questions **one at a time** to refine the idea — multiple choice when
  possible, open-ended when not. If a topic needs more exploration, break it
  into several questions across turns.
- Focus on purpose, constraints, and success criteria.

**Exploring approaches:**
- Propose 2–3 approaches with trade-offs, conversationally.
- Lead with your recommendation and the grounded reasons for it.

**Presenting the design:**
- Present in sections of 200–300 words, checking after each whether it looks
  right so far. Cover architecture, components, data flow, error handling,
  testing.
- YAGNI ruthlessly; go back and clarify whenever something doesn't fit.

**Afterwards:**
- Write the validated design to `docs/plans/YYYY-MM-DD-<topic>-design.md` and
  offer to commit it.
- If continuing to implementation, use the `writing-plans` skill (and a
  worktree if the work needs isolation from the current workspace).
