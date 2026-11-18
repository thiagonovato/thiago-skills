# English README and skill polish

Make `thiago-skills` English-first: rewrite the README (what the repo is, folder layout, how to install skills) and remove remaining Portuguese from the skill description. New repo text defaults to English.

## Decisions

| Decision | Choice |
|---|---|
| Scope | README + skill `description` + English default for future repo text |
| Install docs | Both personal and per-project; **recommended = personal** (`~/.cursor/skills/`) |
| Portuguese triggers in skill description | Remove (English only) |
| Product layout | Keep `skills/`, `.cursor/rules/`, `mcp/`; do not fill rules/MCP |
| Process docs | `docs/superpowers/` holds English design/plan artifacts for this repo (agent workflow); not the primary user-facing story |
| Related CLI link | Keep one-liner to `thiagonovato/todo-scan` |
| Out of scope | Skill body rewrite; rules/MCP content; `todo-scan` changes; rewriting old commit authors |

## README (English)

Order of sections:

1. **What this is** — Personal collection of Cursor Agent Skills plus empty scaffolding for Cursor rules and MCP. Not an application. Each skill is `skills/<name>/SKILL.md`.
2. **Layout** — `skills/`, `.cursor/rules/`, `mcp/` (and optionally note `docs/superpowers/` only if we keep it after this work; primary bullets stay the three above).
3. **Install (recommended: personal)** — Clone or download the repo; symlink or copy `skills/reviewing-typescript-prs` to `~/.cursor/skills/reviewing-typescript-prs`.
4. **Install (optional: per project)** — Same folder into a consuming repo’s `.cursor/skills/`.
5. **Related** — Link to [thiagonovato/todo-scan](https://github.com/thiagonovato/todo-scan).

No Portuguese. No long Cursor tutorial beyond install.

## Skill change

File: `skills/reviewing-typescript-prs/SKILL.md`

- Keep the English body unchanged.
- Edit YAML `description`: drop `revisar PR`, `revisar pull request`, and `pede foco em tipos e testes`. Keep English when-to-use language only.

## Language default

All new user-facing and process text in this repository is English (README, future rules, MCP notes, specs/plans).

## Success criteria

- README answers: what is this repo, what are the folders, how do I install the skill (personal recommended + project optional).
- Skill frontmatter has no Portuguese.
- Repo remains skills-focused; no rules/MCP content added.
