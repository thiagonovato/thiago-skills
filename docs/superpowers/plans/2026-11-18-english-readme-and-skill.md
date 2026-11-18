# English README and Skill Polish Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ship an English README (repo purpose, layout, skill install) and remove Portuguese from the skill description.

**Architecture:** Two file edits only — replace `README.md` with the approved English copy; trim Portuguese phrases from `skills/reviewing-typescript-prs/SKILL.md` frontmatter. No layout or scaffolding changes.

**Tech Stack:** Markdown, Cursor Agent Skills paths (`~/.cursor/skills/`, `.cursor/skills/`)

## Global Constraints

- English only in new/changed user-facing text
- Recommended install: personal `~/.cursor/skills/`
- Also document optional per-project `.cursor/skills/`
- Drop Portuguese triggers from skill `description`
- Do not rewrite skill body; do not fill `.cursor/rules` or `mcp`
- Do not change `todo-scan`
- Git author for commits: `Thiago Novato <thiago@thiagonovato.dev>` (not `thiagonovato-enolytics`)

## File Structure

| Path | Responsibility |
|---|---|
| `README.md` | Repo purpose, layout, install instructions |
| `skills/reviewing-typescript-prs/SKILL.md` | Skill; only `description` frontmatter changes |

---

### Task 1: Rewrite README in English

**Files:**
- Modify: `README.md`

**Interfaces:**
- Consumes: design sections for README order
- Produces: English README covering what / layout / install / related

- [ ] **Step 1: Replace README.md with this exact content**

````markdown
# thiago-skills

Personal collection of [Cursor Agent Skills](https://cursor.com/docs) plus empty scaffolding for Cursor rules and MCP configs. This is not an application — each skill is a folder under `skills/` with a `SKILL.md`.

## Layout

```
thiago-skills/
├── skills/                 # Installable Agent Skills
│   └── reviewing-typescript-prs/
├── .cursor/rules/          # Cursor project rules (empty for now)
└── mcp/                    # MCP configs (empty for now)
```

## Install a skill (recommended: personal)

Use this so the skill is available across all your projects.

1. Clone or download this repository.
2. Symlink or copy the skill folder into your personal skills directory:

```bash
# from the thiago-skills repo root — symlink (recommended)
mkdir -p ~/.cursor/skills
ln -s "$(pwd)/skills/reviewing-typescript-prs" ~/.cursor/skills/reviewing-typescript-prs

# or copy instead of symlink
# cp -R skills/reviewing-typescript-prs ~/.cursor/skills/reviewing-typescript-prs
```

3. Restart Cursor or start a new agent chat so the skill is picked up.

## Install per project (optional)

To share the skill only inside one repo, put it under that repo’s `.cursor/skills/`:

```bash
# from the consuming project root
mkdir -p .cursor/skills
ln -s /path/to/thiago-skills/skills/reviewing-typescript-prs .cursor/skills/reviewing-typescript-prs
```

Replace `/path/to/thiago-skills` with the real path (or copy the folder instead of symlinking).

## Related

The `todo-scan` CLI lives in a sibling repository: [thiagonovato/todo-scan](https://github.com/thiagonovato/todo-scan).
````

- [ ] **Step 2: Verify no Portuguese remains in README**

```bash
rg -n '[áàâãéêíóôõúçÁÀÂÃÉÊÍÓÔÕÚÇ]|Coleção|pasta|instalar|repositório' README.md || true
```

Expected: no matches (or only inside URLs/paths if any).

- [ ] **Step 3: Commit**

```bash
git add README.md
git -c user.name="Thiago Novato" -c user.email="thiago@thiagonovato.dev" \
  commit --author="Thiago Novato <thiago@thiagonovato.dev>" -m "$(cat <<'EOF'
Rewrite the README in English with skill install steps.

EOF
)"
```

---

### Task 2: English-only skill description

**Files:**
- Modify: `skills/reviewing-typescript-prs/SKILL.md` (frontmatter `description` only)

**Interfaces:**
- Consumes: current description string
- Produces: same description without Portuguese trigger clauses

- [ ] **Step 1: Replace the description line**

Change the YAML `description` from the current value to exactly:

```yaml
description: Use when reviewing a TypeScript pull request and tsc is green, tests pass, coverage went up, or someone says not to block on type nits. Use when the diff has any, as, as unknown as, a type predicate (x is T), @ts-ignore, @ts-expect-error, or a test that mocks the unit under test. Use when tempted to approve and ask for a follow-up after ship. Use when the user asks to review a PR or to focus on types and tests.
```

Do not change the body below the frontmatter.

- [ ] **Step 2: Verify Portuguese phrases are gone**

```bash
rg -n 'revisar|pede foco' skills/reviewing-typescript-prs/SKILL.md || true
```

Expected: no matches.

- [ ] **Step 3: Commit**

```bash
git add skills/reviewing-typescript-prs/SKILL.md
git -c user.name="Thiago Novato" -c user.email="thiago@thiagonovato.dev" \
  commit --author="Thiago Novato <thiago@thiagonovato.dev>" -m "$(cat <<'EOF'
Drop Portuguese triggers from the TypeScript PR skill description.

EOF
)"
```

---

### Task 3: Sync GitHub

**Files:** none (push only)

- [ ] **Step 1: Push main**

```bash
git push origin HEAD
git status -sb
```

Expected: `main...origin/main` with no ahead/behind.

- [ ] **Step 2: Confirm author on latest commits**

```bash
git log -3 --format='%h %an <%ae> %s'
```

Expected: authors are `Thiago Novato <thiago@thiagonovato.dev>`, not `thiagonovato-enolytics`.
