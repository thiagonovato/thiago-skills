# Split: todo-scan fora de thiago-skills

Separar o CLI `todo-scan` do repositório de skills Cursor. Dois diretórios irmãos sob `/Users/thiagonovato/www/`, cada um com um propósito.

## Decisões

| Decisão | Escolha |
|---|---|
| Nome da pasta/repo | `todo-scan` (não `todo-scam`) |
| Conteúdo de `thiago-skills` | Só `skills/`, `.cursor/rules/`, `mcp/`, `README.md` (+ `.gitignore`) |
| Scaffolding Cursor no CLI | Não — só em `thiago-skills` |
| Histórico git do CLI | Repo limpo (sem histórico de `feat/todo-scan`) |
| GitHub | `thiagonovato/todo-scan`, **private** |
| Código do CLI | Snapshot inalterado; sem mudança de comportamento |

## Estrutura alvo

```
/Users/thiagonovato/www/
├── thiago-skills/
│   ├── README.md
│   ├── .gitignore
│   ├── skills/                 # reviewing-typescript-prs
│   ├── .cursor/rules/          # vazio
│   └── mcp/                    # vazio
│
└── todo-scan/
    ├── README.md
    ├── package.json
    ├── tsconfig.json
    ├── tsconfig.test.json
    ├── .gitignore
    ├── src/
    ├── test/
    └── docs/superpowers/
        ├── specs/2026-11-17-todo-scan-design.md
        ├── specs/2026-11-18-todo-scan-repo-split-design.md
        └── plans/2026-11-17-todo-scan.md
```

## Fontes

- **Pacote:** `.worktrees/todo-scan/todo-scan/` — copiar `src/`, `test/`, `package.json`, `package-lock.json`, `tsconfig.json`, `tsconfig.test.json`, `.gitignore`. Não copiar `node_modules/`, `dist/`, `dist-test/`.
- **Docs do contrato:** `docs/superpowers/specs/2026-11-17-todo-scan-design.md` e `docs/superpowers/plans/2026-11-17-todo-scan.md`.
- **Este spec:** acompanha o move para `todo-scan/docs/superpowers/specs/`.

## READMEs

**thiago-skills:** descreve o repo como coleção de skills Cursor; explica `skills/`, `.cursor/rules/`, `mcp/`; aponta que o CLI vive em `todo-scan`.

**todo-scan:** descreve o comando, Node ≥20, `npm i` / `npm run build` / `npm test`, uso `todo-scan [caminho]`, link para a spec do contrato.

## Git e GitHub

1. Criar `/Users/thiagonovato/www/todo-scan` com o snapshot + docs + README.
2. `npm install` e `npm test` para validar o snapshot.
3. `git init`, commit inicial, `gh repo create thiagonovato/todo-scan --private --source=. --remote=origin --push` com a conta `thiagonovato`.
4. Em `thiago-skills`: criar `.cursor/rules/` e `mcp/`, escrever README, remover `docs/` e o worktree do CLI (`git worktree remove`), ajustar `.gitignore` se preciso, commit de limpeza. Push de `thiago-skills` só se pedido.

## Fora de escopo

- Alterar comportamento, API ou testes do CLI
- Publicar no npm
- Push automático de `thiago-skills`
- Preencher rules ou MCP configs
