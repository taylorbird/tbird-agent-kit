---
name: project-init
description: Use when starting a new project or initializing project work-state — scaffolds the .claude/ tree if missing, then sets Project/Objective/Constraints and starts the work log. Triggers on "start a new project", "init this project", "set up work state", "/project-init".
---

Starting a new project. Set up the work state.

## 0. Scaffold `.claude/` if missing

Before initializing, make sure the work-state tree exists. Create any directory
or seed file that is **absent**. If a file already **exists**, do NOT overwrite
it — show the user the path and ask before replacing it (they may have real work
in it). Create the following:

Directories: `.claude/standards/`, `.claude/work/`, `.claude/learnings/`, `.claude/archive/`.

**`.claude/CLAUDE.md`**
```markdown
# Project Instructions

## Session Management
Active work state: @.claude/work/current.md

On session start, read .claude/work/current.md to understand current state.
Use /session-checkpoint before ending sessions. Use /session-resume to continue work.

## Standards
@.claude/standards/code-style.md
@.claude/standards/security.md
@.claude/standards/testing.md
@.claude/standards/infrastructure.md

## Learnings
Project-specific technical knowledge: @.claude/learnings/
Capture durable insights here as you discover them (API quirks, integration gotchas, patterns that work).

## Project-Specific Context
<!-- Add project-specific instructions below -->
```

**`.claude/work/current.md`**
```markdown
# Current State

## Project
(none active)

## Objective
(not set)

## Current Focus
(none)

## Last Checkpoint
(none)

## Next Actions
(none)
```

**`.claude/work/log.md`**
```markdown
# Work Log

<!-- Entries prepended, newest first -->
```

**`.claude/work/questions.md`**
```markdown
# Open Questions

<!-- Things uncertain, need revisiting, or blocked on -->
```

**`.claude/learnings/README.md`**
```markdown
# Learnings

Durable, reusable technical knowledge discovered during this project.

## How to use
- Create one file per topic: `{topic}.md` (e.g., `pverify-api.md`, `stytch-m2m.md`)
- Capture things you'd want to know if starting fresh: API quirks, gotchas, patterns that work, things that don't
- Keep entries concise and actionable — not a journal, but a reference
- Updated during /session-checkpoint when new learnings are identified
- Consolidated into archive on /project-close

## Format per file
```markdown
# {Topic}

## Key Facts
- {fact}

## Gotchas
- {gotcha}

## Patterns That Work
- {pattern}
```
```

**`.claude/standards/code-style.md`**
```markdown
# Code Style Standards

<!-- Customize for your organization -->

## General
- Keep functions small and focused
- Prefer explicit over implicit

## JavaScript/TypeScript
- 2-space indentation
- Single quotes
- Semicolons required
- Prefer const, then let, never var
- Use async/await over raw promises

## Naming
- camelCase: variables, functions
- PascalCase: classes, components, types
- SCREAMING_SNAKE: constants
- kebab-case: file names

## Comments
- Explain WHY, not WHAT
- JSDoc for public APIs
- TODO format: `// TODO(name): description`
```

**`.claude/standards/security.md`**
```markdown
# Security Standards

<!-- Customize for your organization -->

## Secrets Management
- Never commit secrets
- Use environment variables or secret managers
- Never log sensitive data
- Rotate credentials regularly

## Input Validation
- Validate all external input
- Use schema validation (Zod, Joi)
- Sanitize before database operations

## Authentication
- Use established libraries (don't roll your own)
- Implement proper session management
- Use secure token storage

## Dependencies
- Review new dependencies before adding
- Keep dependencies updated
- Monitor for vulnerabilities
```

**`.claude/standards/testing.md`**
```markdown
# Testing Standards

<!-- Customize for your organization -->

## Coverage
- Aim for meaningful coverage, not arbitrary percentages
- Critical paths must be tested

## Unit Tests
- Test one thing per test
- Use descriptive test names
- Arrange-Act-Assert pattern

## Integration Tests
- Test real interactions where practical
- Use test databases, not mocks, for data layer

## File Naming
- `*.test.ts` or `*.spec.ts`
- Co-locate with source or in `__tests__/`

## Test Naming
- describe: component or function name
- it: "should {expected} when {condition}"
```

**`.claude/standards/infrastructure.md`**
```markdown
# Infrastructure Standards

<!-- Customize for your organization -->

## General
- Infrastructure as code (always)
- Document manual steps if unavoidable
- Use consistent naming conventions

## AWS
- Tag all resources
- Use least-privilege IAM
- Enable CloudTrail

## Pulumi/IaC
- Stack naming: {env}-{service}-{region}
- Use typed configuration
- Keep stacks focused (don't monolith)

## Logging & Observability
- Structured logging (JSON)
- Include correlation IDs
- Log at appropriate levels
```

**`.gitignore`** — keep this step. If `.gitignore` exists and does not already
contain `.claude/work/`, append:
```
# Claude Code work state (optional - remove if you want to track)
# .claude/work/
```
If `.gitignore` does not exist, create it with those two lines.

Do **not** create `preferences.md` — session-checkpoint creates it on first write.

## 1. Check for existing work

Read `.claude/work/current.md`. If there's an active project (Project field is not "(none active)"):
- Ask if I want to archive it first with /project-close
- Or confirm I want to abandon it

## 2. Initialize work files

Update `.claude/work/current.md`:
```markdown
# Current State

## Project
{project name from argument}

## Objective
{extract or ask for 1-2 sentence objective}

## Constraints
{any hard rules / guardrails stated for this project — e.g. "no git", "never touch prod", "never leak PHI to clients". One bullet each. If none stated, write "None yet."}

## Current Focus
Initial planning and setup

## Last Checkpoint
{current date and time} - Project initialized

## Next Actions
1. {suggest 2-3 initial actions based on the project description}
```

## 3. Start the log

Prepend to `.claude/work/log.md`:
```
## {YYYY-MM-DD HH:MM} - Project Started

**Project**: {name}
**Objective**: {objective}

**Initial Context**:
- {any context provided}

**Planned Approach**:
- {high-level approach if discernible}
```

## 4. Clear questions

Reset `.claude/work/questions.md` to empty (keep header).

## 5. Confirm

Tell me: "Project '{name}' initialized. Ready to start with: {first next action}"
