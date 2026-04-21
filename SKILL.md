---
name: init-agent-customization
description: "Bootstrap a portable AI-agent customisation tree (instructions, skills, prompts, subagents) for any codebase, working across GitHub Copilot, Claude Code, and OpenAI Codex. Use when the user asks to 'initialise copilot instructions', 'set up AGENTS.md', 'scaffold a .github/copilot tree', 'create CLAUDE.md', 'add agent customisations to this repo', or when onboarding an AI coding agent to an unfamiliar codebase."
---

# Init Agent Customization

## What this skill produces

A portable customisation tree that any of the three major coding agents can read:

| Agent | Root file it reads | Sub-tree it recognises |
|---|---|---|
| **GitHub Copilot** | `.github/copilot-instructions.md` | `.github/{instructions,skills,prompts,agents,hooks}/` |
| **Claude Code** | `CLAUDE.md` (root) | `.claude/{agents,commands,skills}/` |
| **OpenAI Codex** | `AGENTS.md` (root) | honoured recursively from any directory |

The skill writes **one canonical source** (`.github/`) and creates thin stub files at the root (`AGENTS.md`, `CLAUDE.md`) that point to it — so all three tools stay in sync with no duplication.

## When to Use

- User asks to initialise / bootstrap / scaffold agent customisations
- User has an existing codebase but no `copilot-instructions.md`, `AGENTS.md`, or `CLAUDE.md`
- User wants to migrate between Copilot / Claude Code / Codex without rewriting rules
- User wants the same rule set honoured by multiple agents simultaneously

## Workflow

Follow these phases in order. Ask the user to confirm before moving between phases.

### Phase 1 — Discover (read-only)
Detect what already exists. If any of these are present, load them and **merge — do not overwrite**:
- `.github/copilot-instructions.md`, `.github/instructions/**`, `.github/prompts/**`, `.github/skills/**`, `.github/agents/**`
- `CLAUDE.md`, `.claude/**`
- `AGENTS.md`, `AGENT.md`
- `.cursorrules`, `.windsurfrules`, `.clinerules`, `.cursor/rules/**`

Inventory the codebase to identify:
- **Languages / frameworks** (file extensions, lockfiles, project files)
- **Build & test commands** (`package.json` scripts, `Makefile`, `*.sln`, `pyproject.toml`, `*.sqlproj`)
- **Directory shape** (top-level folders — 1-2 sentence purpose each)
- **Existing conventions** (README, CONTRIBUTING.md, naming patterns in top 20 files)
- **Testing framework** (tSQLt, pytest, jest, xunit, etc.)
- **Any domain rules** visible in existing instructions/rules files

Dispatch a recon subagent (e.g. `db-explorer`, `Explore`) in parallel if the codebase is large. Do not read more than needed.

### Phase 2 — Propose (confirm with user)
Present a **table** of what will be created, what will be merged, and what will be left alone. Do **not** write files yet. Ask the user to approve or adjust.

For each proposed file state:
- Path
- One-line purpose
- Whether it is **new**, **merged**, or **untouched**

### Phase 3 — Scaffold (create folders + root files)
Create the minimum viable tree:

```
.github/
  copilot-instructions.md      # canonical rule set — the ONLY long document
  instructions/                # language/domain standards (applyTo globs)
  skills/                      # on-demand templates and workflows
  prompts/                     # slash-command workflows
  agents/                      # specialist subagents
  hooks/                       # validation scripts (optional)
AGENTS.md                      # stub → .github/copilot-instructions.md
CLAUDE.md                      # stub → .github/copilot-instructions.md
.claude/
  agents/    -> symlink OR stub that references .github/agents/
  commands/  -> symlink OR stub that references .github/prompts/
  skills/    -> symlink OR stub that references .github/skills/
```

On Windows, if symlinks are not available, write short stub files that use relative links to the `.github/` equivalents — see [templates.md](./templates.md).

### Phase 4 — Generate content
Populate `.github/copilot-instructions.md` using the standard section order. See [templates.md](./templates.md) for the skeleton. Must include:

1. **Four Principles** — Think / Simplify / Surgical / Goal-Driven (verbatim — they are agent-neutral)
2. **Workflow phases** — Understand → Plan → Implement → Verify, each tagged with the priority guidelines it enforces
3. **Priority Guidelines table** — PG1..PGn, sourced from the discovery phase
4. **Technology Stack** — only what discovery found, with exact versions
5. **Project Architecture** — directory-by-directory map
6. **Customisation Files table** — links to every file in `.github/` sub-folders
7. **Subagents table** — one row per agent with dispatch trigger
8. **Version Control** — commit prefix / branch rules (read from CONTRIBUTING.md if present)

Then generate the seed sub-files — start with the **minimum set** (Simplicity First):
- `instructions/` — one file per primary language (`applyTo: **/*.{ext}`)
- `skills/` — one skill per recurring multi-step task found in discovery
- `prompts/` — one slash command per high-frequency verb (review, migrate, test)
- `agents/` — one subagent only if a task is clearly repetitive and read-heavy

Do **not** pre-create empty folders or placeholder files.

### Phase 5 — Verify
- Follow every link in `copilot-instructions.md` and confirm each target exists
- Run any existing `validate-customizations` hook
- Open `AGENTS.md` and `CLAUDE.md` and confirm they resolve to the canonical file
- Report a summary table: files created, lines per file, broken links (should be zero)

## Assets

| File | Purpose |
|---|---|
| [templates.md](./templates.md) | Skeletons for `copilot-instructions.md`, `AGENTS.md`, `CLAUDE.md`, instruction file, skill file, prompt file, agent file |
| [portability-rules.md](./portability-rules.md) | Cross-tool compatibility rules — frontmatter fields, path conventions, tool-specific gotchas |

## Design Principles (non-negotiable)

1. **One canonical document** — never duplicate rules across `copilot-instructions.md`, `AGENTS.md`, `CLAUDE.md`. Stubs link to the canonical file.
2. **Link, don't embed** — README/CONTRIBUTING already exist; link to them.
3. **Minimal by default** — every folder and file must justify its existence from discovery. No speculative scaffolding.
4. **Portable frontmatter only** — skill/prompt/agent frontmatter must use fields all three tools tolerate: `name`, `description`, optional `tools`, optional `applyTo` (Copilot only — Claude/Codex ignore unknown keys safely).
5. **No tool-specific absolute paths** — always workspace-relative.
6. **Ask before overwriting** — merge existing customisations, never destroy.

## Constraints

- Do NOT write application code
- Do NOT invent conventions the codebase does not already use
- Do NOT create subagents unless the codebase has a clear repetitive task that benefits from parallelism
- Do NOT exceed ~200 lines in `copilot-instructions.md` — if longer, split into linked instruction files
