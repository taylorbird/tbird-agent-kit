---
name: session-resume
description: Use when resuming work, continuing from a previous session, or getting back up to speed on a project — reads the work-state files and briefs the user. Triggers on "resume", "where were we", "catch me up", "continue where I left off", "/session-resume".
---

Read the work state and get me back up to speed:

1. Read .claude/work/current.md
2. Read the most recent 1-2 entries in .claude/work/log.md
3. Check .claude/work/questions.md for blockers
4. List the topic files in .claude/learnings/ (if the dir exists), and read its README.md if present

Tell me:
- **Project**: {name} — {objective}
- **Constraints**: {the Constraints section from current.md, verbatim — omit this line only if there is no Constraints section}
- **Last checkpoint**: {date/time}
- **Where we left off**: {current focus and last actions}
- **Next up**: {next action from list}
- **Open questions**: {any blockers or questions, or "None"}
- **Learnings on file**: {comma-separated .claude/learnings topic names, or "None"}

Then ask: "Continue with '{next action}', or something else?"
