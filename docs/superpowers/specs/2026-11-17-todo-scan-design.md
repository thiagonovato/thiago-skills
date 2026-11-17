# todo-scan

CLI em Node.js e TypeScript que encontra marcadores `TODO:` e imprime as tarefas no stdout, agrupadas por arquivo. Não grava arquivo.

O pacote fica em `todo-scan/`, separado da skill `reviewing-typescript-prs`. Não tem dependência de runtime. TypeScript só compila. Os testes usam `node:test`.

## Comando

```
todo-scan [caminho]
```

Sem argumento, a raiz é `process.cwd()`: o diretório atual do terminal, não a pasta do pacote. Um argumento relativo é resolvido a partir de `process.cwd()`. `cli` classifica o caminho com `stat`, então um link simbólico na raiz vale o alvo.

O campo `bin` aponta `todo-scan` para `dist/cli.js`. Exige Node 20 ou mais novo. O pacote é ESM.

## Peças

| Peça | Faz | Depende de |
|---|---|---|
| `cli` | Lê o argumento, chama a varredura, decide a decoração, escreve stdout/stderr, define o código de saída | `scan`, `format`, `process.stdout.isTTY`, `process.env.NO_COLOR` |
| `scan` | Lê um arquivo ou percorre uma árvore e devolve só arquivos com pelo menos uma tarefa | `node:fs`, `node:path` |
| `format` | Transforma essa lista na string agrupada, com ou sem cor e link | nada |

`scan` devolve uma lista de `{ file, absolute, todos }`. `file` é o caminho visível, com o separador do sistema. `absolute` é o caminho absoluto usado no link. Cada tarefa é `{ line, text }`. `line` começa em 1. `text` é o trecho depois de `TODO:`, sem espaço nas pontas. `format` ordena pelo `file`; a ordem da varredura não importa.

Num diretório, `file` é o caminho relativo à raiz varrida e `absolute` é esse caminho resolvido a partir da raiz. Num arquivo passado direto, `file` é o caminho relativo a `process.cwd()` quando `path.relative` não começa com `..`. Caso contrário, `file` é o absoluto.

## Saída

Arquivos em ordem lexicográfica de `file`. Tarefas na ordem das linhas. Uma linha em branco entre arquivos. Sem linha em branco depois do último arquivo. Se a lista não é vazia, a string termina com `\n`. Lista vazia é string vazia: nada no stdout e código 0.

Sem decoração:

```
● src/app.ts
  ▸ 12  ajustar timeout
  ▸ 40  extrair validação

● README.md
  ▸ 3  documentar o comando
```

A linha do arquivo é `●`, um espaço e `file`. A linha da tarefa é dois espaços, `▸`, um espaço, o número da linha sem zeros à esquerda, dois espaços e o texto. Número mais longo não alinha as colunas. Texto vazio deixa a linha terminar depois dos dois espaços que seguem o número.

Decoração liga quando `process.stdout.isTTY === true` e `NO_COLOR` está ausente ou é string vazia. Aí a linha do arquivo inteira é ciano (`\x1b[36m` … `\x1b[0m`) e é um hyperlink OSC 8 para a linha 1. O `▸` é amarelo (`\x1b[33m` … `\x1b[0m`). O número e o texto ficam na cor padrão. A linha da tarefa inteira, inclusive os dois espaços iniciais, é um hyperlink para a linha dela.

O hyperlink é `\x1b]8;;<uri>\x1b\\` antes do texto visível e `\x1b]8;;\x1b\\` depois. A cor fica dentro do hyperlink. Na URI, o separador do `absolute` vira `/`. Cada byte que não seja `/` nem caractere não reservado (`A-Z`, `a-z`, `0-9`, `-`, `.`, `_`, `~`) vai em percent-encoding. Depois vem `:` e o número da linha. Exemplo: `/Users/a b/src/app.ts` na linha 12 vira `file:///Users/a%20b/src/app.ts:12`.

Sem decoração, os mesmos `●` e `▸` aparecem, sem ANSI e sem `file://`.

## O que conta como tarefa

Uma linha gera no máximo uma tarefa: a primeira ocorrência dos caracteres `TODO:` até o fim da linha. A busca é sensível a maiúsculas. `todo:` e `FIXME:` não contam. `MYTODO:` conta, porque contém `TODO:`.

O texto é o restante da linha, com espaço nas pontas removido. `// TODO: ajustar timeout` produz `ajustar timeout`. `TODO:` sem texto produz texto vazio e a linha entra na lista. `foo TODO: bar TODO: baz` produz uma tarefa, `bar TODO: baz`.

Quebras `\n` e `\r\n` separam linhas.

## O que a varredura ignora

Diretório: `scan` não segue link simbólico. Só lê arquivo regular. Desce em diretório cujo nome não seja exatamente um destes: `node_modules`, `.git`, `dist`, `build`, `coverage`. O nome é o da pasta, em qualquer profundidade. Outros ocultos, como `.env`, entram.

Arquivo passado direto: `scan` lê só esse arquivo. Não entra em pasta e não aplica a lista de nomes acima. Um arquivo dentro de `node_modules` entra se foi o argumento.

Antes de decodificar, lê o arquivo inteiro como buffer. Se os primeiros 8192 bytes contêm um byte nulo, o arquivo inteiro fica de fora. No arquivo passado direto, isso também vale: stdout vazio e código 0. Senão, decodifica o buffer como UTF-8 e aplica a regra da tarefa.

## Fluxo

1. `cli` resolve o caminho: `process.argv[2]` ou `process.cwd()`.
2. `cli` decide `decorate` pelo TTY e por `NO_COLOR`.
3. `scan` devolve os arquivos com tarefa.
4. `format` monta o bloco.
5. `cli` escreve o bloco no stdout.

## Erros

Código 1 e uma linha no stderr, sem lista no stdout:

| Situação | stderr |
|---|---|
| Caminho não existe | `todo-scan: caminho não encontrado: <caminho>` |
| Caminho existe e não é arquivo nem diretório | `todo-scan: não é um arquivo nem um diretório: <caminho>` |
| Mais de um argumento (`process.argv.length > 3`) | `uso: todo-scan [caminho]` |
| O arquivo ou diretório passado direto não pode ser lido | `todo-scan: não foi possível ler <caminho>: <mensagem>` |

Arquivo ou pasta ilegível no meio de uma árvore: pula o item, escreve `todo-scan: não foi possível ler <caminho>: <mensagem>` no stderr e continua. Raiz válida termina com código 0, com ou sem tarefas. Achar tarefas não é falha.

## Testes

Sem dependência de runtime. `typescript` compila para `dist/`. Testes em `todo-scan/test/` com `node:test`. A decoração é testada chamando `format` com o flag ligado. O subprocesso do CLI cobre a saída desviada, que não é um TTY.

- Casamento da linha, sem disco: `TODO:` com texto, texto vazio, espaço removido, uma tarefa por linha, `todo:` e `FIXME:` de fora, `MYTODO:` dentro.
- Formatação sem decoração: `●` e `▸`, grupo, linha em branco entre arquivos, ordem dos caminhos, lista vazia vira string vazia, texto vazio, sem ANSI e sem `file://`.
- Formatação com decoração: linha do arquivo ciano apontando para a linha 1, `▸` amarelo, linha da tarefa apontando para a linha dela, URI `file://` com o absoluto e percent-encoding.
- Varredura em diretório temporário: `TODO:` aninhado, ignora `node_modules`, `.git`, `dist`, `build` e `coverage`, ignora arquivo com byte nulo nos primeiros 8192 bytes, não segue link simbólico.
- Arquivo passado direto: devolve só os `TODO:` dele, inclusive se a pasta pai se chama `node_modules`. Binário passado direto imprime nada e termina com código 0.
- CLI em subprocesso: caminho ausente, caminho que não é arquivo nem diretório, arquivo com tarefa no stdout e código 0, arquivo ilegível passado direto com código 1, argumento a mais, sem argumento usa o cwd do processo, arquivo ilegível no meio da árvore gera aviso no stderr, código 0 e as outras tarefas no stdout. Saída desviada traz `●` e `▸` sem cor e sem link.

## Fora deste programa

Não grava relatório. Não emite JSON. Não resume contagem. Não trata `FIXME`, `XXX` ou `HACK`. Não lê `.gitignore`. Não exige que `TODO:` esteja dentro de comentário. Não falha o processo porque encontrou tarefas. Não reconhece `FORCE_COLOR`.
