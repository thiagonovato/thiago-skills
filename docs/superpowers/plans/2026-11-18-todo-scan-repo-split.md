# todo-scan Repo Split Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Extrair o CLI `todo-scan` para `/Users/thiagonovato/www/todo-scan` (repo GitHub private novo) e deixar `thiago-skills` só com skills + scaffolding Cursor, com ambos sincronizados no GitHub.

**Architecture:** Snapshot limpo do pacote no worktree `feat/todo-scan` + docs do contrato em um diretório irmão com `git init` novo; `thiago-skills` remove docs/worktree do CLI e ganha `.cursor/rules/`, `mcp/` e README. Push final nos dois remotes.

**Tech Stack:** Node ≥20, TypeScript 5.6, npm, git, `gh` (conta `thiagonovato`)

## Global Constraints

- Nome da pasta/repo: `todo-scan` (nunca `todo-scam`)
- `thiago-skills` final: só `skills/`, `.cursor/rules/`, `mcp/`, `README.md`, `.gitignore`
- Sem scaffolding Cursor dentro de `todo-scan`
- Histórico do CLI: repo limpo (não filtrar commits de `feat/todo-scan`)
- GitHub: `thiagonovato/todo-scan` **private**; `thiago-skills` no remote `git@github.com:thiagonovato/thiago-skills.git`
- Sync: push dos dois repos; devolver os dois links
- Não alterar comportamento/API/testes do CLI; não publicar npm; não preencher rules/MCP
- Conta `gh` ativa: `thiagonovato` (não `thiagonovato-enolytics`)
- Paths absolutos base: `/Users/thiagonovato/www/thiago-skills` e `/Users/thiagonovato/www/todo-scan`

## File Structure

| Path | Responsibility |
|---|---|
| `/Users/thiagonovato/www/todo-scan/src/*` | Código do CLI (cópia inalterada) |
| `/Users/thiagonovato/www/todo-scan/test/*` | Testes do CLI (cópia inalterada) |
| `/Users/thiagonovato/www/todo-scan/package.json` | bin + scripts build/test |
| `/Users/thiagonovato/www/todo-scan/docs/superpowers/**` | Specs/plans do CLI e deste split |
| `/Users/thiagonovato/www/todo-scan/README.md` | Uso do CLI |
| `/Users/thiagonovato/www/thiago-skills/skills/**` | Skills Cursor (inalteradas) |
| `/Users/thiagonovato/www/thiago-skills/.cursor/rules/` | Scaffolding vazio (`.gitkeep`) |
| `/Users/thiagonovato/www/thiago-skills/mcp/` | Scaffolding vazio (`.gitkeep`) |
| `/Users/thiagonovato/www/thiago-skills/README.md` | Descrição do repo de skills |

---

### Task 1: Criar snapshot do pacote todo-scan

**Files:**
- Create: `/Users/thiagonovato/www/todo-scan/` (diretório + cópias)
- Create: `/Users/thiagonovato/www/todo-scan/docs/superpowers/specs/`
- Create: `/Users/thiagonovato/www/todo-scan/docs/superpowers/plans/`
- Create: `/Users/thiagonovato/www/todo-scan/README.md`

**Interfaces:**
- Consumes: snapshot em `/Users/thiagonovato/www/thiago-skills/.worktrees/todo-scan/todo-scan/`
- Consumes: docs em `/Users/thiagonovato/www/thiago-skills/docs/superpowers/`
- Produces: árvore completa de `/Users/thiagonovato/www/todo-scan` pronta para `npm install`

- [ ] **Step 1: Criar diretório e copiar o pacote (sem artefatos)**

```bash
mkdir -p /Users/thiagonovato/www/todo-scan
SRC=/Users/thiagonovato/www/thiago-skills/.worktrees/todo-scan/todo-scan
DEST=/Users/thiagonovato/www/todo-scan
cp -R "$SRC/src" "$SRC/test" "$DEST/"
cp "$SRC/package.json" "$SRC/package-lock.json" "$SRC/tsconfig.json" "$SRC/tsconfig.test.json" "$SRC/.gitignore" "$DEST/"
```

Expected: `ls "$DEST"` mostra `src test package.json package-lock.json tsconfig.json tsconfig.test.json .gitignore`

- [ ] **Step 2: Copiar docs do contrato e do split**

```bash
DEST=/Users/thiagonovato/www/todo-scan
DOCS=/Users/thiagonovato/www/thiago-skills/docs/superpowers
mkdir -p "$DEST/docs/superpowers/specs" "$DEST/docs/superpowers/plans"
cp "$DOCS/specs/2026-11-17-todo-scan-design.md" "$DEST/docs/superpowers/specs/"
cp "$DOCS/specs/2026-11-18-todo-scan-repo-split-design.md" "$DEST/docs/superpowers/specs/"
cp "$DOCS/plans/2026-11-17-todo-scan.md" "$DEST/docs/superpowers/plans/"
cp "$DOCS/plans/2026-11-18-todo-scan-repo-split.md" "$DEST/docs/superpowers/plans/"
```

Expected: quatro arquivos markdown sob `docs/superpowers/`

- [ ] **Step 3: Escrever README do CLI**

Create `/Users/thiagonovato/www/todo-scan/README.md` with exactly:

````markdown
# todo-scan

CLI em Node.js e TypeScript que encontra marcadores `TODO:` e imprime as tarefas no stdout, agrupadas por arquivo. Não grava arquivo.

Requer Node.js 20 ou mais novo.

## Install

```bash
npm install
npm run build
```

## Uso

```bash
todo-scan [caminho]
```

Sem argumento, a raiz é o diretório atual do terminal (`process.cwd()`).

## Test

```bash
npm test
```

## Docs

Contrato: [`docs/superpowers/specs/2026-11-17-todo-scan-design.md`](docs/superpowers/specs/2026-11-17-todo-scan-design.md)
````

- [ ] **Step 4: Verificar árvore (sem node_modules/dist)**

```bash
find /Users/thiagonovato/www/todo-scan -maxdepth 3 \
  \( -name node_modules -o -name dist -o -name dist-test \) -prune -o -print | sort
```

Expected: inclui `src/cli.ts`, `test/cli.test.ts`, docs listados, `README.md`; **não** inclui `node_modules`, `dist`, ou `dist-test`

- [ ] **Step 5: Commit local ainda não — validação vem na Task 2**

Nenhum `git init` nesta task. Pare aqui se a árvore estiver incompleta.

---

### Task 2: Validar o snapshot com npm test

**Files:**
- Create: `/Users/thiagonovato/www/todo-scan/node_modules/` (via npm)
- Create: `/Users/thiagonovato/www/todo-scan/dist/` e `dist-test/` (via build/test; ficam no `.gitignore`)

**Interfaces:**
- Consumes: árvore da Task 1
- Produces: confirmação de que o snapshot passa a suíte existente

- [ ] **Step 1: Instalar dependências**

```bash
cd /Users/thiagonovato/www/todo-scan && npm install
```

Expected: exit 0; `node_modules/typescript` e `node_modules/@types/node` presentes

- [ ] **Step 2: Rodar a suíte**

```bash
cd /Users/thiagonovato/www/todo-scan && npm test
```

Expected: exit 0; todos os testes `node:test` PASS (cli, format, match, scan)

- [ ] **Step 3: Confirmar .gitignore cobre artefatos**

```bash
cd /Users/thiagonovato/www/todo-scan && cat .gitignore
```

Expected: contém `node_modules/`, `dist/`, `dist-test/`

---

### Task 3: Publicar repo GitHub private todo-scan

**Files:**
- Create: `/Users/thiagonovato/www/todo-scan/.git/`
- Remote: `git@github.com:thiagonovato/todo-scan.git`

**Interfaces:**
- Consumes: snapshot validado da Task 2
- Produces: repo private no GitHub com commit inicial e push

- [ ] **Step 1: Confirmar conta gh**

```bash
gh auth status
gh api user --jq .login
```

Expected: login `thiagonovato`. Se a conta ativa for outra, rodar `gh auth switch --user thiagonovato` (ou `gh auth login`) antes de continuar.

- [ ] **Step 2: git init e commit inicial**

```bash
cd /Users/thiagonovato/www/todo-scan
git init -b main
git add src test package.json package-lock.json tsconfig.json tsconfig.test.json .gitignore README.md docs
git status
git commit -m "$(cat <<'EOF'
Initial import of the todo-scan CLI.

EOF
)"
```

Expected: `git status` limpo; staged/commit **não** inclui `node_modules`, `dist`, `dist-test`

- [ ] **Step 3: Criar repo private e push**

```bash
cd /Users/thiagonovato/www/todo-scan
gh repo create thiagonovato/todo-scan --private --source=. --remote=origin --push
```

Expected: exit 0; remote `origin` aponta para `thiagonovato/todo-scan`

- [ ] **Step 4: Confirmar URL**

```bash
gh repo view thiagonovato/todo-scan --json url,isPrivate,nameWithOwner
```

Expected: `"isPrivate": true`, `"nameWithOwner": "thiagonovato/todo-scan"`, URL `https://github.com/thiagonovato/todo-scan`

---

### Task 4: Slim thiago-skills (skills + scaffolding + README)

**Files:**
- Create: `/Users/thiagonovato/www/thiago-skills/.cursor/rules/.gitkeep`
- Create: `/Users/thiagonovato/www/thiago-skills/mcp/.gitkeep`
- Create: `/Users/thiagonovato/www/thiago-skills/README.md`
- Delete: `/Users/thiagonovato/www/thiago-skills/docs/` (árvore inteira, após cópia na Task 1)
- Remove worktree: `/Users/thiagonovato/www/thiago-skills/.worktrees/todo-scan`
- Modify: `/Users/thiagonovato/www/thiago-skills/.gitignore` (manter ignores de tooling; garantir que `.cursor/` e `mcp/` **não** estejam ignorados)

**Interfaces:**
- Consumes: docs já copiados para `todo-scan` (Task 1)
- Produces: working tree de `thiago-skills` só com skills + scaffolding

- [ ] **Step 1: Remover o worktree do CLI**

```bash
cd /Users/thiagonovato/www/thiago-skills
git worktree list
git worktree remove /Users/thiagonovato/www/thiago-skills/.worktrees/todo-scan
# Se a branch feat/todo-scan sobrar só local e não for mais necessária:
git branch -D feat/todo-scan || true
```

Expected: `git worktree list` mostra só o worktree principal; path `.worktrees/todo-scan` ausente

- [ ] **Step 2: Criar scaffolding Cursor**

```bash
mkdir -p /Users/thiagonovato/www/thiago-skills/.cursor/rules
mkdir -p /Users/thiagonovato/www/thiago-skills/mcp
touch /Users/thiagonovato/www/thiago-skills/.cursor/rules/.gitkeep
touch /Users/thiagonovato/www/thiago-skills/mcp/.gitkeep
```

- [ ] **Step 3: Escrever README de thiago-skills**

Create `/Users/thiagonovato/www/thiago-skills/README.md` with exactly:

````markdown
# thiago-skills

Coleção de Agent Skills e scaffolding Cursor.

## Layout

- `skills/` — skills instaláveis (ex.: `reviewing-typescript-prs`)
- `.cursor/rules/` — regras Cursor do projeto (vazio por agora)
- `mcp/` — configs / servidores MCP do projeto (vazio por agora)

## CLI relacionado

O CLI `todo-scan` vive no repositório irmão: [thiagonovato/todo-scan](https://github.com/thiagonovato/todo-scan).
````

- [ ] **Step 4: Remover docs do CLI deste repo**

```bash
cd /Users/thiagonovato/www/thiago-skills
rm -rf docs
```

Expected: `test ! -d docs` → true

- [ ] **Step 5: Conferir .gitignore**

`/Users/thiagonovato/www/thiago-skills/.gitignore` deve continuar ignorando `.superpowers/`, `.worktrees/`, `.DS_Store`. Não deve ignorar `skills/`, `.cursor/`, `mcp/`, ou `README.md`.

Se `.cursor` estiver ignorado, remova essa linha. Conteúdo alvo:

```
.superpowers/
.worktrees/
.DS_Store
```

- [ ] **Step 6: Verificar layout final local**

```bash
cd /Users/thiagonovato/www/thiago-skills
find . -maxdepth 3 \
  \( -name .git -o -name .superpowers -o -name .worktrees \) -prune -o -print | sort
```

Expected: `./README.md`, `./.gitignore`, `./skills/...`, `./.cursor/rules/.gitkeep`, `./mcp/.gitkeep`; **sem** `./docs`

- [ ] **Step 7: Commit da limpeza**

```bash
cd /Users/thiagonovato/www/thiago-skills
git add -A
git status
git commit -m "$(cat <<'EOF'
Keep only skills and Cursor scaffolding after splitting out todo-scan.

EOF
)"
```

Expected: commit inclui remoção de `docs/`, adição de README + `.gitkeep`s; working tree limpa (exceto ignores)

---

### Task 5: Sync GitHub nos dois repos e reportar links

**Files:**
- Remote push: `thiagonovato/todo-scan`
- Remote push: `thiagonovato/thiago-skills`

**Interfaces:**
- Consumes: Task 3 (todo-scan já pushed no create) + Task 4 commit
- Produces: ambos remotes alinhados com o disco; URLs devolvidas ao usuário

- [ ] **Step 1: Confirmar todo-scan no remote**

```bash
cd /Users/thiagonovato/www/todo-scan
git status -sb
git push -u origin main
gh repo view thiagonovato/todo-scan --json url,isPrivate -q .
```

Expected: branch limpa / already up to date; `isPrivate: true`

- [ ] **Step 2: Push thiago-skills**

```bash
cd /Users/thiagonovato/www/thiago-skills
git status -sb
git push origin HEAD
gh repo view thiagonovato/thiago-skills --json url -q .url
```

Expected: exit 0; `main` (ou branch atual) alinhada com `origin`

- [ ] **Step 3: Sanity check remoto vs local (thiago-skills)**

```bash
cd /Users/thiagonovato/www/thiago-skills
git ls-tree -r --name-only HEAD | sort
```

Expected: contém `README.md`, `skills/reviewing-typescript-prs/SKILL.md`, `.cursor/rules/.gitkeep`, `mcp/.gitkeep`; **não** contém paths sob `docs/`

- [ ] **Step 4: Sanity check remoto vs local (todo-scan)**

```bash
cd /Users/thiagonovato/www/todo-scan
git ls-tree -r --name-only HEAD | sort
```

Expected: contém `src/cli.ts`, `test/cli.test.ts`, `README.md`, specs/plans listados na Task 1; **não** contém `node_modules` nem `dist`

- [ ] **Step 5: Devolver os links ao usuário**

Reportar exatamente:

- Skills: `https://github.com/thiagonovato/thiago-skills`
- CLI: `https://github.com/thiagonovato/todo-scan`
