# Templates

Copy-and-adapt skeletons. Keep the exact frontmatter keys — they are the cross-tool contract.

---

## 1. `.github/copilot-instructions.md` (canonical)

```markdown
# {Project Name} — AI Agent Instructions

{One-sentence description of the project.}

## The Four Principles

These govern every response. When in conflict, the earlier principle wins.

1. **Think Before Coding** — state assumptions; ask on ambiguity; push back on complexity.
2. **Simplicity First** — minimum code that solves the problem; no speculative features.
3. **Surgical Changes** — every changed line traces to the request; no drive-by refactors.
4. **Goal-Driven Execution** — state success criteria; verify before reporting done.

## Workflow

### Phase 1 — Understand
1. Scan for existing patterns — **PG1**
2. Identify the target area of the codebase
3. Confirm version / environment compatibility — **PG2**
4. Surface ambiguity before writing code

### Phase 2 — Plan
5. State success criteria as verifiable checks
6. Sketch a numbered plan with a verify step per item

### Phase 3 — Implement
7. Apply language standards from [.github/instructions/](./instructions/)
8. Load a skill from [.github/skills/](./skills/) only when creating from scratch
9. Follow naming conventions — **PG3**
10. Apply error handling, security, performance rules — **PG4-PG7**

### Phase 4 — Verify
11. Write / update tests — **PG8**
12. Run matching validation
13. Confirm success criteria

## Priority Guidelines

| # | Guideline | Phase |
|---|---|---|
| PG1 | Codebase Consistency | 1 |
| PG2 | Version Compatibility | 1 |
| PG3 | Naming Conventions | 3 |
| ... | ... | ... |

## Technology Stack

- {Language}: {version}
- {Framework}: {version}
- Testing: {framework}

## Project Architecture

- [{folder}/](./{folder}/) — {one-line purpose}
- ...

## Customisation Files

| Location | Purpose |
|---|---|
| [.github/instructions/](./instructions/) | Language/domain standards, auto-applied by file type |
| [.github/skills/](./skills/) | Reusable templates and workflows |
| [.github/prompts/](./prompts/) | Slash-command workflows |
| [.github/agents/](./agents/) | Specialist subagents |

## Version Control

- Commit prefixes: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`
```

---

## 2. `AGENTS.md` (root stub — for Codex)

```markdown
# Agents

This repository's AI-agent rules live in [.github/copilot-instructions.md](.github/copilot-instructions.md).

All Codex / Copilot / Claude sessions must read that file first.
```

---

## 3. `CLAUDE.md` (root stub — for Claude Code)

```markdown
# Claude Code Instructions

See [.github/copilot-instructions.md](.github/copilot-instructions.md) for the canonical rule set.

Subagents:   [.github/agents/](.github/agents/)
Skills:      [.github/skills/](.github/skills/)
Commands:    [.github/prompts/](.github/prompts/)
```

---

## 4. Instruction file (`.github/instructions/{lang}.instructions.md`)

```markdown
---
description: "{Language} development standards"
applyTo: "**/*.{ext}"
---

# {Language} Standards

## Naming
- {rule}

## Structure
- {rule}

## Error Handling
- {rule}

## Testing
- {rule}
```

---

## 5. Skill file (`.github/skills/{name}/SKILL.md`)

```markdown
---
name: {name}
description: "{One sentence with trigger keywords. Use when the user asks to 'X', 'Y', or 'Z'.}"
---

# {Name} Skill

## When to Use
- {trigger 1}
- {trigger 2}

## Inputs
1. {required input}

## Workflow
1. {step} → verify: {check}
2. {step} → verify: {check}

## Output
{what this produces}

## Constraints
- {rule}
```

---

## 6. Prompt file (`.github/prompts/{verb}.prompt.md`)

```markdown
---
mode: 'agent'
tools: ['codebase', 'search', 'editFiles']
description: '{What this slash-command does}'
---

# /{verb}

## Target
{What the command operates on — usually ${file} or ${selection}}

## Procedure
1. {step}
2. {step}

## Output
{format of the report or artefact}
```

---

## 7. Agent file (`.github/agents/{name}.agent.md`)

```markdown
---
name: "{name}"
description: "{One sentence with trigger keywords. Specify thoroughness / scope so it is easy to invoke.}"
tools: [read, search, edit]
---

# {Name}

## Purpose
{One paragraph — what this agent is for and what it refuses to do.}

## Inputs (ask if missing)
- {input}

## Procedure
1. {step}

## Output Format
{structured output so the main agent can consume it}

## Constraints
- Do NOT {thing}
- Delegate {X} to {other agent}
```
