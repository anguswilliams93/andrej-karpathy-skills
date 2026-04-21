# Portability Rules

Rules the `init-agent-customization` skill must follow so output works in **GitHub Copilot**, **Claude Code**, and **OpenAI Codex** without modification.

## Frontmatter Compatibility

| Key | Copilot | Claude | Codex | Notes |
|---|---|---|---|---|
| `name` | honoured | honoured | ignored | Always include |
| `description` | honoured (used for on-demand discovery) | honoured | ignored | Must contain trigger keywords |
| `applyTo` | honoured (glob) | ignored | ignored | Use only in instruction files |
| `tools` | honoured (restricts tools) | honoured | ignored | Keep narrow |
| `mode` | honoured (e.g. `'agent'`) | ignored | ignored | Only for prompt files |
| `model` | honoured | ignored | ignored | Avoid — not portable |

**Rule**: use only `name`, `description`, `applyTo` (instructions only), and `tools` (agents/prompts only). Any tool that does not recognise a field ignores it silently — no errors.

## File Placement

| Artefact | Canonical location | Why |
|---|---|---|
| Main rules doc | `.github/copilot-instructions.md` | Copilot requires this exact path |
| Root stub for Codex | `AGENTS.md` | Codex only reads root-level `AGENTS.md` |
| Root stub for Claude | `CLAUDE.md` | Claude Code convention |
| Instructions | `.github/instructions/*.instructions.md` | Copilot auto-applies by `applyTo` |
| Skills | `.github/skills/{name}/SKILL.md` | Discoverable by all three |
| Prompts | `.github/prompts/*.prompt.md` | Copilot slash commands |
| Subagents | `.github/agents/*.agent.md` | Copilot + Claude (Claude also reads `.claude/agents/`) |

**Rule**: pick `.github/` as the canonical root. Root stubs (`AGENTS.md`, `CLAUDE.md`) must contain only a link to `.github/copilot-instructions.md` and pointers to sub-folders — never duplicate content.

## Path Conventions

- Always **workspace-relative** markdown links: `[.github/skills/foo/SKILL.md](.github/skills/foo/SKILL.md)`
- **Never** `file://`, `vscode://`, or absolute paths
- Use forward slashes even on Windows
- When one customisation file links to a peer, use a relative path: `[peer](./peer.md)` or `[../agents/x.agent.md](../agents/x.agent.md)`

## Naming Conventions

- Skill folders: kebab-case (`init-agent-customization`, not `InitAgentCustomization`)
- Agent files: `{name}.agent.md`
- Instruction files: `{topic}.instructions.md`
- Prompt files: `{verb}.prompt.md`

## Description Quality

The `description:` field is **the** discoverability signal. For each customisation file:
- Start with a verb or noun phrase stating what it produces
- List 3+ trigger phrases the user might actually type ("create a CR", "write a change request", "generate deployment docs")
- Mention constraints that rule it out ("only for Python", "does not write code")

Bad: `"Helps with tests."`
Good: `"Generates pytest test classes for Python modules. Use when the user asks to 'write tests', 'add test coverage', or 'generate a test file for {module}'. Does not modify the module under test."`

## Tool-specific Gotchas

### GitHub Copilot
- `.github/copilot-instructions.md` is the **only** file auto-loaded into every turn. Keep it short; link heavily.
- Instruction files `applyTo` pattern is matched against workspace-relative paths.
- Prompt files must have `mode: 'agent'` frontmatter to allow tool use.

### Claude Code
- Reads `CLAUDE.md` from project root. Also reads `.claude/` sub-tree if present.
- Sub-agents in `.claude/agents/*.md` use a different frontmatter dialect — avoid writing there; keep canonical copies in `.github/agents/` and reference them from `CLAUDE.md`.

### OpenAI Codex
- Reads `AGENTS.md` from any directory on the path to a file it is editing. Nested `AGENTS.md` files are honoured (deepest wins).
- Does not parse frontmatter — keep information in the markdown body too.
- Ignores skills / prompts / agents folders entirely; relies solely on `AGENTS.md` content.

## Merge vs Overwrite Policy

When initialising on a repo that already has customisation files:

| Found | Action |
|---|---|
| `.github/copilot-instructions.md` | **Merge** — preserve custom content, update sections, report removed duplication |
| `CLAUDE.md` with content | **Replace with stub** only if content duplicates the canonical; otherwise merge into canonical |
| `AGENTS.md` with content | Same as `CLAUDE.md` |
| `.cursorrules` / `.windsurfrules` | **Read and incorporate** into canonical; leave original in place (don't delete other tools' configs) |

## Self-validation

Before declaring the scaffold done, the init skill must:
1. Follow every link in the canonical document and confirm the target exists
2. Confirm `AGENTS.md` and `CLAUDE.md` both resolve to the canonical file
3. Confirm every skill/prompt/agent file has `name` + `description` frontmatter
4. Confirm every instruction file has an `applyTo` glob
5. Report a diff of files changed — no silent writes
