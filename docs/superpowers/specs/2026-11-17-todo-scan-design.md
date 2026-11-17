# todo-scan

CLI em Node.js e TypeScript que percorre um diretório, encontra marcadores `TODO:` e imprime as tarefas no stdout, agrupadas por arquivo. Não grava arquivo.

O pacote fica em `todo-scan/`, separado da skill `reviewing-typescript-prs`. Não tem dependência de runtime. TypeScript só compila. Os testes usam `node:test`.

## Comando

```
todo-scan [diretório]
```

Sem argumento, a raiz é `process.cwd()`: o diretório atual do terminal, não a pasta do pacote. Um argumento relativo é resolvido a partir de `process.cwd()`.

O campo `bin` aponta `todo-scan` para `dist/cli.js`. Exige Node 20 ou mais novo. O pacote é ESM.

## Peças

| Peça | Faz | Depende de |
|---|---|---|
| `cli` | Lê o argumento, chama a varredura, escreve stdout/stderr, define o código de saída | `scan`, `format` |
| `scan` | Percorre a árvore e devolve só arquivos com pelo menos uma tarefa | `node:fs`, `node:path` |
| `format` | Transforma essa lista na string agrupada | nada |

`scan` devolve uma lista de `{ file, todos }`. `file` é o caminho relativo à raiz varrida, com o separador do sistema. Cada tarefa é `{ line, text }`. `line` começa em 1. `text` é o trecho depois de `TODO:`, sem espaço nas pontas. `format` ordena os arquivos pelo caminho relativo; a ordem da varredura não importa.

## Saída

Arquivos em ordem lexicográfica do caminho relativo. Tarefas na ordem das linhas. Uma linha em branco entre arquivos. Sem linha em branco depois do último arquivo. Se a lista não é vazia, a string termina com `\n`. Lista vazia é string vazia: nada no stdout e código 0.

```
src/app.ts
  12  ajustar timeout
  40  extrair validação

README.md
  3  documentar o comando
```

Cada tarefa é dois espaços, o número da linha sem zeros à esquerda, dois espaços, o texto. Número mais longo não alinha as colunas. Texto vazio deixa a linha terminar depois dos dois espaços que seguem o número.

## O que conta como tarefa

Uma linha gera no máximo uma tarefa: a primeira ocorrência dos caracteres `TODO:` até o fim da linha. A busca é sensível a maiúsculas. `todo:` e `FIXME:` não contam. `MYTODO:` conta, porque contém `TODO:`.

O texto é o restante da linha, com espaço nas pontas removido. `// TODO: ajustar timeout` produz `ajustar timeout`. `TODO:` sem texto produz texto vazio e a linha entra na lista. `foo TODO: bar TODO: baz` produz uma tarefa, `bar TODO: baz`.

Quebras `\n` e `\r\n` separam linhas.

## O que a varredura ignora

A raiz em si pode ser um link simbólico; `cli` a resolve com `stat`. Durante a varredura, `scan` não segue link simbólico. Só lê arquivo regular. Desce em diretório cujo nome não seja exatamente um destes: `node_modules`, `.git`, `dist`, `build`, `coverage`. O nome é o da pasta, em qualquer profundidade. Outros ocultos, como `.env`, entram.

Antes de decodificar, lê o arquivo inteiro como buffer. Se os primeiros 8192 bytes contêm um byte nulo, o arquivo inteiro fica de fora. Senão, decodifica o buffer como UTF-8 e aplica a regra da tarefa.

## Fluxo

1. `cli` resolve a raiz: `process.argv[2]` ou `process.cwd()`.
2. `scan` devolve os arquivos com tarefa.
3. `format` monta o bloco.
4. `cli` escreve o bloco no stdout.

## Erros

Código 1 e uma linha no stderr, sem lista no stdout:

| Situação | stderr |
|---|---|
| Caminho não existe | `todo-scan: diretório não encontrado: <caminho>` |
| Caminho existe e não é diretório | `todo-scan: não é um diretório: <caminho>` |
| Mais de um argumento (`process.argv.length > 3`) | `uso: todo-scan [diretório]` |
| A raiz existe, é diretório, e não pode ser lida | `todo-scan: não foi possível ler <caminho>: <mensagem>` |

Arquivo ou pasta ilegível no meio da árvore: pula o item, escreve `todo-scan: não foi possível ler <caminho>: <mensagem>` no stderr e continua. Raiz válida termina com código 0, com ou sem tarefas. Achar tarefas não é falha.

## Testes

Sem dependência de runtime. `typescript` compila para `dist/`. Testes em `todo-scan/test/` com `node:test`.

- Casamento da linha, sem disco: `TODO:` com texto, texto vazio, espaço removido, uma tarefa por linha, `todo:` e `FIXME:` de fora, `MYTODO:` dentro.
- Formatação, sem disco: grupo, linha em branco entre arquivos, ordem dos caminhos, lista vazia vira string vazia, texto vazio.
- Varredura em diretório temporário: `TODO:` aninhado, ignora `node_modules`, `.git`, `dist`, `build` e `coverage`, ignora arquivo com byte nulo nos primeiros 8192 bytes, não segue link simbólico.
- CLI em subprocesso: raiz ausente, raiz que é arquivo, raiz ilegível, argumento a mais, sem argumento usa o cwd do processo, arquivo ilegível no meio da árvore gera aviso no stderr, código 0 e as outras tarefas no stdout.

## Fora deste programa

Não grava relatório. Não emite JSON. Não colore. Não resume contagem. Não trata `FIXME`, `XXX` ou `HACK`. Não lê `.gitignore`. Não exige que `TODO:` esteja dentro de comentário. Não falha o processo porque encontrou tarefas.
