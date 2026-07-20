# tbird-agent-kit — Private Plugin Marketplace Design

**Date:** 2026-07-20
**Author:** Taylor Bird
**Status:** Design complete — ready for implementation
**Repo:** https://github.com/taylorbird/tbird-agent-kit

## Goal

Replace the loose collection of slash commands in `~/.claude/commands/` and the
`setup-claude` bash script with a **private, maintainable, installable Claude Code
plugin marketplace**. One git repo Taylor owns; skills he can update and reinstall
across all his machines/projects.

## End-state structure (monorepo = marketplace + plugin)

```
tbird-agent-kit/                          ← private GitHub repo, IS the marketplace
├── .claude-plugin/
│   └── marketplace.json                  ← lists the plugins in this repo
└── plugins/
    └── tbird-claudetools/                ← the (first) plugin
        ├── .claude-plugin/
        │   └── plugin.json               ← name, description, version
        └── skills/
            ├── project-init/SKILL.md
            ├── session-checkpoint/SKILL.md
            ├── session-resume/SKILL.md
            └── session-status/SKILL.md
```

- **Monorepo** chosen for maintenance: one repo to clone, edit, commit, push.
  Docs confirm a single repo can be both marketplace and plugin host, with plugin
  `source` given as a relative path (`./plugins/tbird-claudetools`).
- **One plugin** (`tbird-claudetools`) bundles the cohesive workflow. Future
  unrelated tools become *additional* plugins in the same marketplace.

## Manifests

**`.claude-plugin/marketplace.json`**
```json
{
  "name": "tbird-agent-kit",
  "owner": { "name": "Taylor Bird", "email": "taylor.bird@bumpboxes.com" },
  "plugins": [
    {
      "name": "tbird-claudetools",
      "source": "./plugins/tbird-claudetools",
      "description": "Project work-state and session-continuity skills"
    }
  ]
}
```

**`plugins/tbird-claudetools/.claude-plugin/plugin.json`**
```json
{
  "name": "tbird-claudetools",
  "description": "Scaffold, checkpoint, resume, and status skills for project work-state",
  "version": "0.1.0"
}
```

## Skills (4)

Down from the original six: `setup-claude` (bash) is **merged into** `project-init`;
`project-close` is **dropped** (Taylor runs ~one project per repo; can be re-added
later as a 5th skill if end-of-project archive docs are wanted).

Each skill is a self-contained `SKILL.md` (frontmatter + prose, seed content
embedded inline). Frontmatter `description` is written in **trigger language** so
the model can auto-invoke appropriately.

### 1. `project-init`
Absorbs the old scaffolder as step 0.
- **Step 0 — scaffold if missing:** create `.claude/` tree (`work/`, `learnings/`,
  `standards/`, `archive/`) and seed files (`CLAUDE.md`, `work/{current,log,questions}.md`,
  `learnings/README.md`, `standards/{code-style,security,testing,infrastructure}.md`).
  **Ask before replacing** any file that already exists (create missing silently).
  Keep the `.gitignore` step (append commented `.claude/work/` note, create if absent).
  Do **not** create `preferences.md` (checkpoint makes it on first write).
- **Step 1+ — init the project:** existing `project-init` behavior — set
  Project/Objective/Constraints in `current.md`, first `log.md` entry, reset
  `questions.md`, confirm.

### 2. `session-checkpoint`
Existing behavior — Sonnet agent team updates `current.md`, prepends `log.md`,
updates `questions.md`, captures learnings + candidate preferences. Unchanged apart
from frontmatter (add `name`, trigger-style `description`).

### 3. `session-resume`
Existing behavior — read work-state and brief back up to speed.

### 4. `session-status`
Existing behavior — 3-line status summary.

## Invocation change (noted)

As plugin skills, these namespace as `/tbird-claudetools:project-init` etc., not the
current bare `/project-init`. Muscle-memory change accepted. (Open: whether a short
alias is possible — verify during build.)

## Build / install / update workflow

1. **Clone** https://github.com/taylorbird/tbird-agent-kit locally, add manifests + skills, commit.
2. **Push** to the repo.
3. **Install:** `/plugin marketplace add taylorbird/tbird-agent-kit` (SSH by default for
   private repos — needs key in `ssh-agent`), then
   `/plugin install tbird-claudetools@tbird-agent-kit`.
4. **Update loop:** edit a `SKILL.md` → bump `version` in `plugin.json` → commit →
   push → `/plugin marketplace update` + `/plugin update`. Background auto-update
   works with SSH key in `ssh-agent`.
5. **Retire** the old `~/.claude/commands/*.md` files + `setup-claude` once the
   plugin is installed and verified.

## Migration of existing content

Source files (verbatim prose, reformatted into SKILL.md):
- `project-init`   ← `~/.claude/commands/project-init.md`
- `session-checkpoint` ← `~/.claude/commands/session-checkpoint.md`
- `session-resume` ← `~/.claude/commands/session-resume.md`
- `session-status` ← `~/.claude/commands/session-status.md`
- `project-init` step 0 seed content ← `~/.local/bin/setup-claude` heredocs
  (verbatim, minus `preferences.md`).

## Open items to verify during build

- Short-alias invocation for plugin skills (vs. `/plugin:skill` namespacing).
- Whether `SKILL.md` at plugin root vs. `skills/<name>/SKILL.md` matters for a
  multi-skill plugin (design uses `skills/<name>/` — the multi-skill form).

## Out of scope (YAGNI, for now)

- `project-close` and its archive-doc generation.
- `standards/*.md` customization (kept as generic templates, as today).
- Additional plugins in the marketplace (added later).
