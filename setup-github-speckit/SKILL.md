---
name: speckit-setup
description: >
  Sets up spec-kit (github/spec-kit) in any project with full Claude Code integration.
  Installs all 9 speckit slash-command skills (/speckit-constitution, /speckit-specify,
  /speckit-plan, /speckit-clarify, /speckit-analyze, /speckit-tasks, /speckit-taskstoissues,
  /speckit-implement, /speckit-checklist), scaffolds the .specify/ directory tree, and
  wires CLAUDE.md. Use this skill whenever the user wants to: set up spec-kit, add
  spec-driven development, install speckit, initialize speckit, add /speckit-* commands,
  or set up the SDD workflow in a project. Trigger even if the user just says "set up
  speckit" or "add spec-driven development here" without mentioning Claude Code specifically.
argument-hint: "Optional: path to the project root (defaults to current directory)"
user-invocable: true
disable-model-invocation: false
---

## What this skill does

Sets up [spec-kit](https://github.com/github/spec-kit) in a project so that all
`/speckit-*` slash commands become available in Claude Code. Works even on systems
where `specify-cli` cannot be installed directly (e.g. Python < 3.11 environments)
by falling back to a bundled scaffolding script.

## Steps

### 1 — Identify the project root

Check the user's message for an explicit path. If none, use the workspace folder
that is currently open (the folder shown in the file tools). Confirm with the user
if ambiguous.

### 2 — Check whether spec-kit is already set up

Look for `.claude/skills/speckit-constitution/SKILL.md` in the project root.
If it exists, tell the user spec-kit is already installed and ask if they want
to upgrade to the latest version. If they say no, stop here.

### 3 — Get the latest spec-kit version tag

Run:
```bash
git ls-remote --tags https://github.com/github/spec-kit.git \
  | grep -o 'refs/tags/v[0-9]*\.[0-9]*\.[0-9]*$' \
  | sort -V | tail -1 | sed 's|refs/tags/||'
```

If this fails (no network), fall back to `v0.8.11`.

### 4 — Try the official install path

Run:
```bash
uv tool install specify-cli \
  --from git+https://github.com/github/spec-kit.git@<VERSION>
```

If this succeeds, run:
```bash
cd <PROJECT_ROOT> && specify init . --integration claude --force
```

Then skip to Step 6.

### 5 — Fallback: bundled scaffolding script

If Step 4 failed (most commonly because the system Python is < 3.11, which
`specify-cli` requires), use the bundled script instead.

First, clone spec-kit at the target version into a temp directory:
```bash
git clone --depth 1 --branch <VERSION> \
  https://github.com/github/spec-kit.git /tmp/spec-kit-setup
```

Then run the bundled script (it is Python 3.10-compatible):
```bash
python3 <SKILL_DIR>/scripts/setup_speckit.py \
  --spec-kit-src /tmp/spec-kit-setup \
  --project-root <PROJECT_ROOT>
```

The script creates:
- `.claude/skills/speckit-*/SKILL.md` — all 9 command skills
- `.specify/templates/` — spec, plan, tasks, constitution, checklist templates
- `.specify/templates/commands/` — raw command templates
- `.specify/scripts/bash/` — prerequisite-check and feature-setup scripts
- `.specify/memory/` — empty; populated by `/speckit-constitution`
- `.specify/specs/` — empty; populated by `/speckit-specify`

### 6 — Update CLAUDE.md

Check whether `CLAUDE.md` exists in the project root.

**If it already exists and has content:**
Append the following block (between the markers) only if the markers are not
already present:

```
<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
<!-- SPECKIT END -->
```

**If CLAUDE.md does not exist:**
Create it using the template below — replacing `<PROJECT_NAME>` with the
directory name of the project root.

```markdown
# <PROJECT_NAME> — Claude Code Instructions

## Spec-Driven Development (spec-kit)

This project uses spec-kit for structured feature development.
Run `/speckit-constitution` first to establish project principles,
then follow the workflow:

| Skill | Purpose |
|---|---|
| `/speckit-constitution` | Create or update project governing principles |
| `/speckit-specify` | Define what you want to build |
| `/speckit-clarify` | Clarify underspecified areas before planning |
| `/speckit-plan` | Create a technical implementation plan |
| `/speckit-analyze` | Cross-artifact consistency & coverage check |
| `/speckit-tasks` | Generate an actionable task list from the plan |
| `/speckit-taskstoissues` | Push generated tasks to GitHub as issues |
| `/speckit-implement` | Execute all tasks and build the feature |
| `/speckit-checklist` | Generate a quality checklist for requirements |

Specs live in `.specify/specs/<feature>/`.
The project constitution lives in `.specify/memory/constitution.md`.

<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
<!-- SPECKIT END -->
```

### 7 — Confirm and guide next steps

Tell the user:

1. How many skills were installed and where (`.claude/skills/speckit-*/`)
2. What version of spec-kit was used
3. The recommended first command to run:

> **Start here:** Run `/speckit-constitution` with a short description of your
> project's principles (e.g. coding standards, testing requirements, deployment
> constraints). This creates `.specify/memory/constitution.md`, which every
> subsequent speckit command will reference.

4. The typical feature workflow:
   `/speckit-specify` → `/speckit-clarify` → `/speckit-plan` →
   `/speckit-analyze` → `/speckit-tasks` → `/speckit-implement`
