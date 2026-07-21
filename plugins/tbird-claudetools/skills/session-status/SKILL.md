---
name: session-status
description: Use when the user wants a quick status check or a short summary of the current project state — gives a 3-line (project/focus/next) summary from current.md. Triggers on "status", "where am I", "quick status", "what's the state", "/session-status".
---

Read .claude/work/current.md and give me a 3-line summary:
1. **Project**: {name} — {objective}
2. **Focus**: {current focus}
3. **Next**: {first next action}

If current.md has a Constraints section, add one more line:
4. **Constraints**: {the hard rules, condensed to one line}

No elaboration unless asked.
