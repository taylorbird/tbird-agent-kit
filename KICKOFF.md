# KICKOFF — where we left off

Planning is **done**. This repo is ready to build. Full design:
`docs/plans/2026-07-20-tbird-agent-kit-design.md` — read it first.

## What this is

A private Claude Code plugin marketplace. One plugin (`tbird-claudetools`) bundling
4 skills that replace Taylor's `~/.claude/commands/*.md` and the `setup-claude` bash
script.

## Decisions already made (don't relitigate)

- Monorepo: this repo is both the marketplace and the plugin host.
- Marketplace name: `tbird-agent-kit`. Plugin name: `tbird-claudetools`.
- 4 skills: `project-init`, `session-checkpoint`, `session-resume`, `session-status`.
- `setup-claude` scaffolding is **merged into `project-init`** as step 0
  (scaffold `.claude/` if missing, **ask before overwriting** existing files,
  keep the `.gitignore` step, do NOT create `preferences.md`).
- `project-close` is **dropped** for now (re-addable later).
- Skills are self-contained `SKILL.md` files; descriptions in trigger language.

## Build checklist (in order)

- [ ] `.claude-plugin/marketplace.json` (schema in design doc)
- [ ] `plugins/tbird-claudetools/.claude-plugin/plugin.json`
- [ ] `plugins/tbird-claudetools/skills/project-init/SKILL.md`
      (migrate `~/.claude/commands/project-init.md` + prepend scaffold step 0 from
      `~/.local/bin/setup-claude` heredocs, minus preferences.md)
- [ ] `plugins/tbird-claudetools/skills/session-checkpoint/SKILL.md`
      (migrate `~/.claude/commands/session-checkpoint.md`)
- [ ] `plugins/tbird-claudetools/skills/session-resume/SKILL.md`
      (migrate `~/.claude/commands/session-resume.md`)
- [ ] `plugins/tbird-claudetools/skills/session-status/SKILL.md`
      (migrate `~/.claude/commands/session-status.md`)
- [ ] Commit + push
- [ ] Install & test. NOTE: default `github.com` SSH on this machine resolves to
      the WORK account (`taylorbirdbumphealth`), which can't read this personal
      repo. Use the personal SSH alias URL, not the `owner/repo` shorthand:
      `/plugin marketplace add git@github.com-personal:taylorbird/tbird-agent-kit.git`
      then `/plugin install tbird-claudetools@tbird-agent-kit`
      (This repo's git remote is already set to `git@github.com-personal:...`.)
- [ ] Verify each skill works, then retire old `~/.claude/commands/*.md` + `setup-claude`

## Verify during build

- Short-alias invocation vs. `/tbird-claudetools:project-init` namespacing.

## Then (separate thread)

Return to the original reason `~/bump/dev/planning/dme_metadata` was opened:
brainstorm the **dme_metadata feature** (not yet started). Run the new
`project-init` there once the plugin is installed.
