# Claude Configs

[![Claude Code](https://img.shields.io/badge/Claude_Code-compatible-D97757?style=flat-square)](https://claude.com/claude-code)
[![Cursor](https://img.shields.io/badge/Cursor-compatible-1a1a1a?style=flat-square)](https://cursor.com)
[![claude.ai](https://img.shields.io/badge/claude.ai-pre--prompt-D97757?style=flat-square)](https://claude.ai)
[![last commit](https://img.shields.io/github/last-commit/Palanx/Claude-Configs?style=flat-square&color=555)](https://github.com/Palanx/Claude-Configs/commits/main)
[![license](https://img.shields.io/github/license/Palanx/Claude-Configs?style=flat-square&color=555)](LICENSE)

Personal configuration for Claude: rules that shape every session, skills that load on
demand, and the pre-prompt for the surface that reads neither.

Nothing here is project-specific. Anything that only makes sense inside one codebase
belongs in that codebase.

## Layout

```
rules/    → ~/.claude/rules/        loaded every session, all projects
skills/   → ~/.claude/skills/       loaded on demand, by description match
gui/      → copy-paste              claude.ai account settings
```

## What reads what

The surfaces differ, and the differences are not obvious:

| Surface | `rules/` | `skills/` | Account pre-prompt |
|---|---|---|---|
| Claude Code CLI / desktop | yes | yes | no |
| Cowork (desktop) | yes | **no** — uses account skills | no |
| claude.ai chat | no | no — uses account skills | yes |
| claude.ai/code (web) | project-level only | no | no |

Two consequences worth knowing:

- **Cowork reads the rules but not the local skills.** A rule that points at a skill —
  `interaction-protocol` sends plans to `planning` — dangles there unless the same skill
  is also enabled on the account.
- **Skills are one file, two deployments.** The same `SKILL.md` is copied to
  `~/.claude/skills/` for the CLI and enabled on claude.ai for Cowork and chat. No forked
  GUI variant: skills that assume a filesystem say what to do without one instead.

## Install

```bash
cp -r rules  ~/.claude/
cp -r skills ~/.claude/
```

Copy, don't symlink: Cowork sessions skip a symlinked `~/.claude/CLAUDE.md`, and a
`~/.claude/rules/` symlink pointing outside the working directory.

For claude.ai, paste `gui/pre-prompt.md` into Settings → Profile and enable the skills
you want from Customize in the desktop sidebar.

## Rules

| File | What it governs |
|---|---|
| `interaction-protocol.md` | When files may be written at all: explicit order only, otherwise ask whether to implement, explain, or plan. Plus the protocol for consulting a project's tech debt log. |
| `response-protocol.md` | Language, register, and epistemics: correct wrong premises first, distinguish knowledge from inference, read the source or search the web before asserting, never restate. |

## Skills

| Skill | Loads when |
|---|---|
| `planning` | A plan is asked for before implementing — bugfix, refactor, feature. Produces a document a person executes step by step, each step independently verifiable. |
| `tech-debt-log` | Writing or updating a project's triaged debt log: the entry schema and the maintenance rules. The log itself is per-project data and lives in that project. |
| `architecture-evaluation` | Designing or reviewing a system architecture. Scope assessment, then a tradeoff table across candidates. |
| `gamedev-client-architecture` | Structuring game client or simulation code. Centred on the engine boundary: what may depend on Unity/Unreal and what must not. |
| `global-engineering-guidelines` | Any coding task. Clean code, SOLID, layering, commit discipline, error handling, testing. |
| `unity-editor-gotchas` | Investigating a rendering, lighting, or reimport symptom reported from the Unity Editor — environment behaviour that looks like a code bug. |

## Conventions

- Skills are self-contained. A skill that depends on a tool, a repository layout, or
  another skill says so and says what to do when it is absent.
- No project identifiers, ticket ids, internal URLs, or employer-specific type names.
  Examples are invented, not copied from real work.
- Rules stay short. They are loaded into every session, so length is a per-session cost;
  anything procedural belongs in a skill instead.
