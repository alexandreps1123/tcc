# Issue tracker: Markdown local

Tarefas e especificações deste workspace vivem em `docs/tarefas/`. Os remotos Git dos componentes servem ao versionamento do código; o tracker configurado para as skills é local.

## Estrutura

- Uma pasta por trabalho: `docs/tarefas/<slug>/`, incluindo data no slug quando útil.
- `README.md` contém o escopo e os critérios de aceite.
- `RELATORIO.md` registra entrega, verificações, limitações e referências do código.
- Quando o trabalho precisar de tickets separados, use `issues/<NN>-<slug>.md`, numerados a partir de `01`, um arquivo por ticket.
- Cada ticket novo identifica seu componente, objetivo, critérios de aceite, dependências e `Triage:` com um dos rótulos de `triage-labels.md`.
- Discussões do ticket são acrescentadas em `## Comentários`.

A estrutura existente de uma tarefa continua válida. Referências congeladas são preservadas; a especificação fornecida pelo usuário determina quais documentos regem cada execução.

## Operações das skills

**Buscar ticket ou especificação:** leia o caminho indicado pelo usuário. Para um número de ticket, resolva-o dentro da pasta do trabalho; se houver ambiguidade entre pastas, peça o caminho.

**Publicar no tracker:** crie ou atualize o arquivo local correspondente em `docs/tarefas/`. Essa operação não cria issues, comentários ou PRs externos.

**Implementar:** identifique o repositório do componente, confira seu estado e registre a referência inicial e final do código no relatório da tarefa.

**Revisar:** use o `README.md` da tarefa e suas referências como especificação, incluindo mudanças ainda não commitadas e arquivos novos quando fizerem parte da entrega.

## Wayfinder

Quando `/wayfinder` for utilizado:

- Mapa: `docs/tarefas/<slug>/map.md`, com as seções Notes, Decisions-so-far e Fog.
- Tickets: `docs/tarefas/<slug>/issues/NN-<slug>.md`, com a pergunta no corpo e `Type:` igual a research, prototype, grilling ou task.
- `Status:` registra open, claimed ou resolved. É separado da classificação `Triage:`.
- `Blocked by: NN, NN` identifica dependências na mesma pasta; o ticket fica desbloqueado quando todas estiverem resolved.
- A fronteira contém tickets open, desbloqueados e não claimed; priorize o menor número.
- Antes de trabalhar, grave `Status: claimed`.
- Ao resolver, acrescente a resposta em `## Answer`, grave `Status: resolved` e registre síntese e link em Decisions-so-far no mapa.
