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
