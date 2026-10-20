# Escopo fechado — preparação do compartilhamento de seeds

Criado em: 2026-10-10. Atividades: S02 e S03, com entrega parcial independente do banco.
Situação inicial: escopo preparado; implementação e verificações funcionais não iniciadas.
Atualização de acompanhamento (2026-10-10): entrega registrada em [RELATORIO.md](RELATORIO.md),
com revisão pendente. Este escopo e suas referências fixas continuam sendo a base
da entrega; decisões posteriores de infraestrutura/consulta estão nos documentos
principais e não ampliam retroativamente a atividade delegada.

## Objetivo e referências

Implementar e validar o objeto de candidato, a adaptação ao sistema atual de seeds
e a identidade estável de contrato/função. A entrega prepara a integração futura;
não inclui compartilhamento efetivo entre containers.

Leia primeiro este escopo. Para o objeto e as decisões vigentes, consulte
[001](referencias/001-brainstorm-centralizacao-seeds.md); para custos e dependências,
[002](referencias/002-custo-centralizacao-seeds.md); para checkpoints e pendências,
[003](referencias/003-acompanhamento-centralizacao-seeds.md). Consulte o
[glossário](referencias/GLOSSARY.md) quando precisar esclarecer termos.
O [manifesto](REFERENCIAS.md) identifica a captura fixa destas referências.

Este documento delimita a atividade delegada. Os snapshots contêm o planejamento
completo, mas não autorizam executar tarefas posteriores. Decisões históricas
substituídas não constituem requisitos. Alterações posteriores nos documentos da
raiz só entram na tarefa mediante atualização explícita de escopo pelo usuário.

## Trabalho autorizado e sequência

1. Inspecionar as regras locais aplicáveis e o fluxo atual de cadastro, seleção,
   serialização e mutação. Registrar no relatório os pontos de integração e os
   campos efetivamente consumidos pelos três orderers. Critério: cada dependência
   do adapter tem correspondência identificada no código.
2. Implementar `SeedCandidate` conforme a definição capturada: `SourceHash`,
   `ContractName`, `FunctionSignature`, `Inputs`, `ExecutedInstructions`,
   `DeltaCoverage`, `DeltaMinDistance` e `CriticalInstructionsHits`, com os tipos
   e comentários documentados. Critério: campos implementados sem perda de dados
   de seleção e sem campo de versão ou referência de origem.
3. Implementar a adaptação para a estrutura usada pelo fluxo atual. Preservar
   argumentos e métricas; usar os dados do contrato consumidor onde os algoritmos
   atuais já os utilizam. Critério: candidatos equivalentes produzem os mesmos
   cálculos e ordenação com contexto e ordem inicial iguais nas três greybox.
4. Implementar a identidade compartilhada: SHA-256 dos bytes UTF-8 da fonte exata
   recebida antes da compilação/instrumentação, nome da declaração selecionada e
   assinatura ABI completa com tipos canônicos. Calcular no cadastro e reutilizar.
   Critério: a identidade independe de UUID local e endereço de deployment e distingue
   fontes, declarações e sobrecargas. Escolher representação em memória compatível
   com a estrutura existente; se a integração exigir mudança de esquema, registrar
   a dependência e entregar o cálculo isolado, sem criar tabela ou migração.
5. Verificar tipos suportados, adapter e identidade. Usar testes pertinentes às
   alterações e casos de aceite abaixo. Critério: evidências registradas com
   comandos, resultados e limitações; falhas preexistentes separadas das novas.

## Regras de compatibilidade

- `Inputs` continua como `[]string` no objeto. A representação persistida utiliza
  `Join/Split` com `;`, sem escape novo. O banco compartilhado será implementado
  em outra tarefa; validar aqui a conversão usando o mecanismo atual.
- Reutilizar valores como estão, inclusive endereços, sem exclusão por dependência
  do ambiente ou remapeamento. Resultado e estado blockchain podem diferir.
- Preservar handlers, algoritmos de seleção, mutações e tratamento atual de erros.
  Falha detectada na desserialização é propagada; o listener registra e encerra
  aquele evento. Complemento configurado continua após recuperação bem-sucedida
  com candidatos insuficientes, sem fallback novo para erro.
- Preservar `ExecutedInstructions` e métricas reais. Dados ausentes não devem ser
  convertidos artificialmente em zeros para aparentar compatibilidade.
- A análise estática já identificou os campos usados pelos três orderers, mas
  isso não substitui a validação funcional do adapter.

## Limites e decisões reservadas

Trabalhar no componente Dogefuzz e nas verificações necessárias à entrega acima.
Banco, conexão/pool, tabelas, migrações, publicação/consulta remotas, infraestrutura,
retenção, novos filtros e configuração do benchmark pertencem às tarefas seguintes.
Toda nova tabela depende de confirmação prévia do usuário, conforme decisão vigente.

Preservar algoritmos atuais, inclusive limitações já conhecidas. Uma correção que
altere pontuação ou ordenação requer atividade separada. Não introduzir deduplicação,
versionamento ou rastreabilidade de produtor nesta entrega.

Preservar os documentos principais 001–003 e os arquivos de `referencias/`: o
usuário continuará discutindo o planejamento em paralelo. Registrar resultados
somente em `RELATORIO.md` nesta pasta. Não atualizar status globais como concluídos.

## Checkpoints de aceite

- [ ] Inspeção do fluxo atual e mapeamento dos campos registrados.
- [ ] Objeto implementado conforme a referência capturada.
- [ ] Adapter preserva argumentos e todos os campos utilizados na seleção.
- [ ] Mesma fonte, declaração e assinatura geram identidade igual em execuções distintas.
- [ ] Fontes diferentes, declarações diferentes e sobrecargas geram identidades diferentes.
- [ ] UUIDs locais e endereços de deployment não interferem na identidade.
- [ ] Argumentos dos tipos atualmente suportados preservam valor e ordem na conversão.
- [ ] Limitação de argumentos contendo `;` registrada, sem exigir solução fora do escopo.
- [ ] Três orderers preservam resultados com candidatos equivalentes, mesmo contexto e ordem inicial.
- [ ] Propagação de erros e complemento de seeds mantidos nos caminhos alterados.
- [ ] Fluxo operacional existente verificado com os checks pertinentes.
- [ ] Entrega distingue o que foi implementado, validado e bloqueado.

## Execução e entrega

Use branch/worktree própria se o componente estiver versionado. Verifique o estado
do repositório antes de editar e preserve alterações existentes. Registre em
`RELATORIO.md` a referência do código inicial e final, os arquivos alterados, as
verificações realizadas e os checkpoints atendidos, com evidências.

Atualização explícita do usuário em 2026-10-10: `main` deve permanecer intacta até
a execução e avaliação dos testes necessários. Implemente e teste em branch
separada; não faça commits em `main` nem merge para ela nesta atividade. A entrega
é revisável na branch de trabalho, sem autorização de integração automática.
Esta regra complementa a execução; os arquivos de referências congeladas permanecem
inalterados. O conjunto de testes de liberação global será definido separadamente.

Se uma decisão reservada bloquear uma parte, descreva o requisito e a pergunta
necessária no relatório; continue as partes independentes. Mudanças fora deste
escopo precisam de definição explícita, sem inferir aprovação de silêncio.

A entrega termina com código e relatório revisáveis. S02 pode permanecer parcial
se a validação depender da persistência compartilhada; testes reais de cooperação
entre containers pertencem às tarefas posteriores. Não afirmar compartilhamento
funcional nem conclusão integral de S02/S03 sem evidência dos respectivos critérios.

Para iniciar o agente, envie: “Execute o escopo de
`docs/tarefas/2026-10-10-s02-s03-preparacao-seeds/README.md`, usando as referências
congeladas dessa pasta. Implemente as partes autorizadas e registre a entrega em
`RELATORIO.md`; preserve os documentos principais de planejamento.”
