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
rules/          → ~/.claude/rules/          loaded every session, all projects
skills/         → ~/.claude/skills/         loaded on demand, by description match
settings.json   → ~/.claude/settings.json   merge; see below
gui/            → copy-paste                claude.ai account settings
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
cp settings.json ~/.claude/       # fresh machine only — merge by hand otherwise
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

## Settings

`settings.json` carries the parts of `~/.claude/settings.json` that mean the same thing on
every machine: the model, the theme, the plugins and their marketplace, and a
`permissions.ask` list covering the git and `gh` commands that should never run unattended.

The two plugin keys do less than they look like they do, and the difference is worth
knowing before trusting them. `extraKnownMarketplaces` **is** applied automatically: once
you trust the folder, Claude Code adds the marketplace without asking. `enabledPlugins`
installs nothing — a plugin from an external source stays reported as not installed, and
Claude Code shows you the `claude plugin install` command to run. So a fresh machine does
not arrive with the plugins; it arrives knowing which ones are missing and how to get
them.

That is the whole value of those two keys: a checklist that announces itself instead of
one you have to remember. The `ask` list is different in kind — a missing permission rule
announces itself only after something has already run.

**`statusLine` is deliberately absent.** Its command is an absolute path into the plugin
cache, carrying both the username and the plugin's version number
(`/Users/<you>/.claude/plugins/cache/ponytail/ponytail/<version>/...`). It breaks on any
other machine and rots on the next plugin update, and it fails silently — an empty status
line, no error saying why. Keep that key machine-local.

So on a machine that already has a `settings.json`, merge the keys rather than copying the
file over; a plain copy would drop the local `statusLine` along with anything else set
there.

## Conventions

- Skills are self-contained. A skill that depends on a tool, a repository layout, or
  another skill says so and says what to do when it is absent.
- No project identifiers, ticket ids, internal URLs, or employer-specific type names.
  Examples are invented, not copied from real work.
- Rules stay short. They are loaded into every session, so length is a per-session cost;
  anything procedural belongs in a skill instead.
