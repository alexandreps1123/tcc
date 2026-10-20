# Acompanhamento — banco centralizado

Criado em: 2026-10-06. Última revisão: 2026-10-08.

Referências: [brainstorm](001-brainstorm-banco-centralizada.md),
[custos revisados](002-custo-banco-centralizado.md) e [vocabulário](GLOSSARY.md).

Controle das **25 tarefas ativas** do escopo simplificado. IDs 19, 20 e 23 ficam
registrados fora do escopo, sem serem contados como concluídos. O documento de
custos e este controle usam os mesmos IDs, pontos e horas.

A tarefa 01 está em andamento para definição documental. D05 está parcialmente
definida. Nenhum código de centralização, compartilhamento ou reset foi
implementado; discutir uma definição não conclui sua implementação.

## 1. Uso e situação atual

Estados: **Pendente**, **Pronta**, **Em andamento**, **Bloqueada**,
**Em validação**, **Concluída** e **Fora do escopo**.

- A tabela é a fonte do status; os detalhes registram checkpoints e evidências.
- Uma tarefa fica Pronta quando suas predecessoras e definições de entrada
  estiverem atendidas. A tarefa responsável pode fechar sua própria definição.
- Registrar responsável, datas, horas reais, bloqueio e próximo passo ao atualizar.
- Marcar checkpoint com evidência: decisão registrada, diff/PR, teste, log,
  relatório ou comando e resultado reproduzível.
- Concluir somente após todos os checkpoints, inclusive a verificação relacionada.
- Atualizar o histórico quando mudar status, dependência ou escopo.
- Se ultrapassar 8 pontos, dividir por comportamento verificável e manter vínculo
  com o ID original. Pontos não são horas nem percentual de avanço.

| Campo | Situação |
| --- | --- |
| Em andamento | 01 — Isolamento mínimo e ciclo de vida, em definição documental |
| Próximo passo | D01 definida; fechar filtros/restrições e inventário em D03 e consulta em D05 |
| Responsável pelo acompanhamento | A definir |
| Bloqueios | Nenhum bloqueio de execução investigado; definições pendentes abaixo |
| Concluídas | 0 de 25 tarefas ativas |
| Pontos concluídos | 0 de 164 |
| Horas reais | Não registradas |
| Fora do escopo | 19, 20 e 23; não são tarefas concluídas |

### Decisões vigentes

- [x] Cooperação entre chamadas independentes, em paralelo e em sequência.
- [x] Acervo disponível para chamadas futuras, sem reset automático por repetição.
- [x] Preservar comandos, execução interna e relatórios próprios do benchmark.
- [x] Preservar UUIDs locais, seleção, mutadores e complemento atual para seeds insuficientes.
- [x] Sem coordenador único, relatório conjunto ou novos IDs experimentais obrigatórios.
- [x] Chave compartilhada: SHA-256 do fonte exato + contrato selecionado + assinatura ABI completa.
- [x] Fonte sem normalização; UUID/endereço/bytecode/configuração de compilação fora da identidade básica.
- [x] Isolamento operacional continua necessário ao centralizar tabelas comuns.
- [x] Adotar tabelas comuns com instance_id; schema por instância não será usado nesta etapa.
- [x] Gerar instance_id no próprio Dogefuzz uma única vez por inicialização; manter entre tarefas e gerar outro no reinício.
- [ ] Fechar configuração, isolamento, representação, compatibilidade adicional e consulta.
- [ ] Definir e orçar eventual script explícito de reset; nenhuma limpeza executada.

## 2. Preparação e referência

Itens de preparação não são pontos adicionais; registrar esforço nas tarefas
relacionadas. Reestimar se aparecer trabalho substancial não coberto.

- [ ] P01 — Confirmar revisões do Dogefuzz, benchmark e Geth do estudo anterior e da implementação.
- [ ] P02 — Vincular seleção de contratos, configuração, corpus inicial e relatórios históricos, preservando os originais.
- [ ] P03 — Confirmar ferramentas/dependências Go, Python, Docker e PostgreSQL para verificação.
- [ ] P04 — Executar testes existentes e um caso curto atual; registrar falhas preexistentes.
- [ ] P05 — Confirmar recursos para chamadas independentes e banco comum; usar duas inicialmente e cinco no exemplo de validação.
- [x] P06 — Revisar documentos e dependências para o escopo simplificado, mantendo estimativas preliminares.

Evidência de P06: revisão consolidada de 001/002/003 em 2026-10-07.
Evidências e pendências restantes: —

### B2_parted como referência

O usuário informou que [B2_parted](dogefuzz-benchmark/dataset/B2_parted) contém
contratos com vulnerabilidades conhecidas usados no experimento anterior.
Na inspeção de 2026-10-06, B2_parted/sol possui sete listas contracts.csv,
com 71 registros; todos os arquivos citados estão presentes. Isso não valida
compilação, execução ou correção dos rótulos.

- [x] Identificar o dataset de referência informado pelo usuário.
- [ ] Vincular revisão, seleção exata, configuração e relatórios do estudo anterior.
- [ ] Mapear rótulos do catálogo aos detectores, por exemplo gasless_send → gasless-send.
- [ ] Definir como usar esperado/observado na avaliação, sem exigir novo relatório conjunto.

O catálogo lê categorias, mas o modelo atual da requisição não as conserva.
Seu uso na análise deve ser definido em D07; não alterar esse fluxo sem necessidade.
P02 permanece pendente.

## 3. Definições necessárias

### D01 — Identidade operacional e ciclo de vida mínimo

Estado: Definida documentalmente. Responsável: Codex (registro documental). Fechada em: 2026-10-08.
Relacionada às tarefas 01 e 04. Não exigir hierarquia experimental nova.

- [x] Definir os termos e distinguir chamada do benchmark de execução de uma tarefa.
- [x] Preservar taskId e UUIDs locais gerados pelo Dogefuzz; resposta e webhook continuam correlacionados por taskId.
- [x] Definir que o acervo não depende da identidade da chamada ou repetição.
- [x] Escolher identidade/contexto técnico mínimo: instance_id em tabelas comuns, em conjunto com D03.
- [x] Definir geração no próprio Dogefuzz e propagação interna pelo contexto da aplicação, sem novos IDs experimentais no pedido do benchmark.
- [x] Definir validade: uma inicialização do processo; mesmo ID entre tarefas e novo UUID no reinício.
- [x] Definir tratamento de interrupções: preservar registros e estados do contexto anterior, sem retomada, reenvio ou reclassificação automática.
- [x] Definir separação de auxiliares: UUID próprio, unicidade por instância/papel e novos deployments por inicialização; contexto imutável gerado pelo próprio processo, sem adoção de contexto externo.

Exemplo ilustrativo: cinco comandos B1–B5 sobre B2_parted, com greybox e uma hora
por contrato. Cada comando tem seu ambiente e relatório; os rótulos B1–B5 não
exigem novo campo na API. Para SimpleDAO, o Dogefuzz de B1 cria T1, o de B2 cria
T2 e assim por diante. Todos podem consultar seeds pela mesma chave da função.

As chamadas podem começar juntas ou em sequência. Outra chamada B6 pode usar
as entradas já concluídas. Centenas de transações de T1 continuam sendo testes
daquela tarefa. O encerramento de B1 não encerra o acervo.

| Conceito | Uso nesta etapa |
| --- | --- |
| Campanha | Organização do estudo, sem novo ID obrigatório |
| Repetição | Reexecução de um caso; não implica corpus novo |
| Worker | Ambiente executor associado à chamada |
| Tarefa | Trabalho do contrato identificado pelo taskId atual |
| Execução de tarefa | Realização da tarefa, sem executionId adicional obrigatório |
| Execução do benchmark | Chamada que percorre contratos e gera relatório próprio |
| Grupo de corpus | Conceito de acesso ao acervo; ID separado não é requisito |

Não incluir protocolo novo de reenvio idempotente ou retomada automática só para
compartilhar seeds. O fluxo atual é a referência; a centralização deve preservar
sua separação operacional. Se uma limitação impedir a entrega, registrar necessidade
e estimativa separadamente.

#### Geração por inicialização e verificação do comportamento atual

Decisão de 2026-10-08: o Dogefuzz gera um UUID uma única vez no bootstrap, antes
de preparar auxiliares, jobs, listeners e API. Mantém o ID no contexto da aplicação
e o propaga internamente a serviços/repositórios. Não gerar UUID ao criar cada
serviço, acessar uma conexão ou iniciar nova tarefa. Não exigir ID fornecido
manualmente nem transportar esse contexto como campo livre do pedido público.

```text
Processo inicia: instance_id A → tarefas T1, T2 e T3 pertencem a A
Processo reinicia: instance_id B → novas tarefas pertencem a B
Banco mantém A e B; seeds concluídas de A continuam consultáveis em shared
```

A geração por inicialização é nova implementação. A inspeção estática encontrou:

- main cria NewEnv(cfg); NewEnv apenas guarda a configuração, sem gerar UUID.
- A conexão abre DatabaseName.db; o nome padrão é dogefuzz, não um ID aleatório de inicialização.
- UUIDs atuais são criados por tarefa, contrato, função e transação nos repositórios.
- O benchmark.sh cria projetos Docker separados, isolando arquivos locais entre chamadas.
- Destroy chama Clean, que remove o SQLite; porém main/waitForInterrupt não chama Destroy no encerramento normal.
- Docker restart pode conservar o SQLite do mesmo container. Não presumir banco novo em todo reinício.

Portanto, a analogia é o isolamento do ambiente, não uma geração existente de
identidade por inicialização. O novo ID separará contextos dentro do banco comum.
Decisão: preservar registros e estados do contexto anterior, sem retomar tarefas,
reenviar transações ou reclassificar automaticamente seus estados. Transações
concluídas continuam candidatas a seeds compartilhadas; incompletas ficam excluídas.
Auxiliares: cada registro recebe UUID próprio, instance_id e papel, com restrição
única em (instance_id, papel). O setup cria e implanta os cinco auxiliares no Geth
local a cada inicialização; consumidores consultam somente a instância atual e o
papel desejado. Constantes atuais representam papéis, não IDs globais. No reinício,
preservar registros antigos como histórico e criar novos, sem reutilizar endereços.

Exemplo: I1 tem o auxiliar UUID-A de papel reentrancy_agent no Geth A; I2 tem
UUID-B do mesmo papel no Geth B. Cada processo resolve seu próprio auxiliar.
Após reiniciar I1, a nova instância I3 cria UUID-C e faz novo deploy; UUID-A permanece
no histórico. O contexto é gerado no bootstrap e não pode ser fornecido ou alterado
por pedidos externos. Filtros e restrições de D03 devem garantir essa separação.

Impactos: adaptar setup, modelo/restrição e consumidores; repetir compilação/deploy
e acumular histórico por inicialização. Benchmark mantém o fluxo atual. Reutilizar
deployments anteriores exigiria validar cadeia e estado; IDs fixos com chave composta
alterariam chaves/relações; armazenamento só em memória reorganizaria consumidores
e reduziria histórico persistido. Foram descartados para manter o escopo mínimo.

Exemplo: a instância I1 deixa uma tarefa T1 em andamento, com uma transação concluída
e outra ainda em execução. Após o reinício, I2 trabalha apenas em seu próprio
contexto. A transação concluída de I1 pode fornecer uma seed compatível; a incompleta
não entra no corpus. T1 mantém seu estado registrado, que não comprova atividade
do processo. Não haverá detecção de instâncias inativas nesta etapa; um ID diferente
também pode pertencer a uma execução paralela ainda ativa.

Evidência: [main](dogefuzz/cmd/dogefuzz/main.go),
[NewEnv/Destroy](dogefuzz/cmd/dogefuzz/env.go),
[conexão SQLite](dogefuzz/data/connection.go),
[configuração padrão](dogefuzz/config.json) e
[script do benchmark](dogefuzz-benchmark/benchmark.sh), inspecionados em 2026-10-08.
Nenhum processo foi iniciado para esta verificação. D01 está definida documentalmente;
geração/propagação e política de auxiliares ainda não foram implementadas.

### D02 — Banco, migração e preservação

Estado: Pendente. Responsável: —. Fechada em: —. Tarefas 02, 03, 10 e 17.

- [ ] Fixar versão PostgreSQL, acesso, pool, readiness e timeouts, sem registrar senhas aqui.
- [ ] Definir executor único da migração e armazenamento persistente.
- [ ] Definir indisponibilidade, fechamento sem exclusão e preservação/backup.
- [ ] Separar banco de testes dos resultados experimentais.
- [ ] Definir alcance do eventual reset: banco inteiro ou acervo/elegibilidade, com condições de uso e orçamento.

Apagar todo o banco remove histórico e proveniência. O acervo proposto é leitura
das transações; não existe necessariamente uma tabela de seeds para truncar.
Não foi autorizado nem executado um reset concreto.

Decisão e evidência: —

### D03 — Esquema e isolamento mínimo

Estado: Em andamento (definição documental). Responsável: Codex (registro).
Fechada em: —. Tarefas 01 e 03–10; nenhum código implementado.

- [x] Escolher tabelas comuns com instance_id, sem schema operacional por instância.
- [x] Definir a fronteira: operações normais restritas à instância, leitura shared de seeds permitida entre instâncias compatíveis.
- [x] Definir abrangência do contexto: instance_id obrigatório nas quatro entidades principais atuais (tasks, contracts, functions e transactions), inclusive auxiliares em contracts; novas tabelas exigem confirmação prévia do usuário.
- [x] Escolher garantia de integridade: validação na aplicação e restrições no banco, incluindo mesma instância e função pertencente ao contrato da tarefa.
- [ ] Definir os demais campos, relações, restrições e índices para o mecanismo escolhido.
- [ ] Inventariar todas as consultas, jobs e callbacks que operam registros locais.
- [ ] Definir auxiliares e busca por hash/endereço em cadeias distintas.
- [ ] Definir exceção de leitura shared sem permitir escrita em tarefas externas.
- [ ] Definir erro SQL sem reenvio indevido ao Geth.

#### Decisão, impactos e alternativa não escolhida — 2026-10-08

O usuário confirmou tabelas comuns com identificador técnico de instância.
Exemplo ilustrativo:

| Registro | Contexto | Operação permitida |
| --- | --- | --- |
| Tarefa T1 | A | Processo de A executa/atualiza T1 |
| Tarefa T2 | B | Processo de B executa/atualiza T2 |
| Seed concluída de T1 | A | Processo de B pode ler a entrada compatível e criar sua própria transação |

instance_id define propriedade operacional; não entra na chave compartilhada
source_hash + contract_name + function_signature para limitar o modo shared.
Os UUIDs atuais continuam identificando cada registro. O contexto vem da
configuração/aplicação conforme D01, não de alterações livres dos registros.
A geração por inicialização e propagação interna foram definidas em D01;
integração nos modelos e restrições ainda precisa ser implementada.

Decisão de 2026-10-08, com escopo esclarecido pelo usuário: instance_id obrigatório
somente nas quatro entidades principais atuais: tasks, contracts, functions e
transactions; auxiliares estão incluídos em contracts. Se uma nova tabela for
necessária, confirmar com o usuário sua inclusão, finalidade e necessidade de
instance_id antes de implementá-la; não aplicar automaticamente a mesma regra.
Preencher pelo contexto interno ao criar o registro, rejeitar
valor nulo/vazio e impedir mudança posterior de propriedade.

Exemplo: A lê uma seed concluída de B. O registro original mantém instance_id=B;
a nova transação criada por A recebe instance_id=A. O campo não entra na chave
compartilhada da função e não restringe essa leitura autorizada.

Impactos: adaptar modelos, migração e todos os caminhos de criação; repetir o
contexto exige consistência entre relações. A vantagem é filtrar cada tabela
diretamente, incluindo auxiliares sem tarefa. Contexto apenas em tasks não foi
escolhido por exigir consultas pelas relações e tratamento separado dos auxiliares.
Obrigatoriedade não substitui filtros de consulta nem garante que tarefa e função
de uma transação sejam compatíveis; desenho concreto de relações/restrições/índices segue pendente.
Estimativa mantida; nenhum código implementado.

#### Integridade das relações — decisão de 2026-10-08

Escolhida validação na aplicação e restrições no banco, mantendo UUIDs atuais como
identificadores principais e incluindo instance_id nas chaves estrangeiras das
relações operacionais. As regras aprovadas são:

- Contrato analisado referencia tarefa da mesma instância.
- Função referencia contrato da mesma instância.
- Transação referencia tarefa e função da mesma instância.
- A função da transação pertence ao contrato daquela tarefa, inclusive quando há várias tarefas na mesma instância.
- Auxiliar pode existir sem tarefa, mantendo instância e papel.

Exemplo inválido: transação de A aponta para tarefa de A e função de B. Outro
exemplo inválido: transação de T1 aponta para função do contrato de T2, embora
ambas as tarefas pertençam a A. Aplicação e banco devem rejeitar essas relações.
Exemplo válido: A lê a entrada concluída de B e cria uma transação vinculada à
tarefa e função locais de A, preservando o registro original de B.

Validar somente na aplicação foi descartado por deixar a integridade dependente
de todos os caminhos executarem a verificação. Impactos: adaptar migração e
validações (tarefas 03, 05–07) e testes negativos (24); restrições adicionam trabalho
nas escritas. Permanecem pendentes o desenho exato das chaves e da garantia
função/contrato/tarefa, demais campos e índices. Não afirmar que chaves por instância
sozinhas garantem compatibilidade entre tarefas; não criar nova tabela sem confirmação.
Decisão documental; nenhum código implementado e estimativas mantidas.

Impactos a verificar nas tarefas:

- [ ] Aplicar contexto e filtros a CRUD, listagens e SQL direto, sem confiar que um filtro genérico cobre todos os caminhos.
- [ ] Preservar relações operacionais da mesma instância por validação/restrições adequadas.
- [ ] Resolver auxiliares por instância/papel e buscas blockchain por instância/hash.
- [ ] Isolar jobs, callbacks, análise e finalização; centralizar leitura shared somente no caminho de seeds.
- [ ] Definir índices operacionais e índice da chave de seeds; validar plano de consulta sob carga representativa.
- [ ] Testar acesso com IDs externos e comprovar que esquecer/contornar o contexto não permite processamento cruzado nos caminhos implementados.

A vantagem escolhida é manter uma consulta sobre um acervo persistente central
e um conjunto comum de tabelas/migrações. O custo aceito é revisar muitos pontos
do código atual e testar ausência de filtros, sobretudo nos SQLs diretos.

Schemas separados poderiam preservar mais consultas locais e permitir auxiliares
com IDs iguais em namespaces diferentes. Foram descartados para esta etapa
porque o reuso de histórico entre chamadas futuras exigiria descobrir/reunir
schemas preservados e manter migrações em cada um. Views ou SQL gerado poderiam
resolver a leitura, mas acrescentariam manutenção ao foco principal de compartilhar seeds.
Schemas também exigiriam configurar corretamente conexões/permissões; sua existência
isolada não impediria acesso cruzado.

Não há medição de superioridade de desempenho entre as alternativas. Ambas
usariam o mesmo servidor PostgreSQL e seus recursos. As 158–284 horas permanecem
preliminares: já incluem isolamento das tabelas comuns, mas serão reavaliadas
após inventário das operações e detalhamento de integridade.

Decisão e evidência: escolha confirmada pelo usuário em 2026-10-08, impactos
registrados nesta seção e nos documentos 001/002. D03 continua parcialmente aberta.

### D04 — Representação de inputs

Estado: Pendente. Responsável: —. Fechada em: —. Tarefas 11, 13 e 14.

- [ ] Definir formato suficiente para tipos suportados, com versão se necessária.
- [ ] Verificar números, bytes, strings e coleções, incluindo delimitadores.
- [ ] Preservar interface externa e definir rejeição de entradas incompatíveis.
- [ ] Decidir representação canônica somente se necessária a identificação/deduplicação do input.

Decisão e evidência: —

### D05 — Identidade e consulta de seeds

Estado: Em andamento, em definição documental. Responsável: Codex (registro).
Fechada em: —. Tarefas 12–15; implementação ainda não iniciada.

- [x] Adotar código-fonte como base da identidade estável.
- [x] Distinguir declarações no mesmo fonte pelo nome do contrato selecionado.
- [x] Usar assinatura ABI completa da função com tipos canônicos.
- [x] Calcular uma vez, persistir e consultar por igualdade com índice composto.
- [x] Permitir reuso em paralelo, em sequência e em chamadas futuras.
- [x] Manter seleção/mutação/complemento atuais, sem filtro obrigatório de ganho.
- [ ] Fechar filtros adicionais de serialização, cenário e configuração dirigida.
- [ ] Definir fallback/receive e restrição de endereços locais sem remapeamento.
- [ ] Definir janela de candidatos, atualização/cache e tratamento de repetidos sem mudar silenciosamente a seleção.
- [ ] Definir finalização/elegibilidade e gravação da transação de origem.

A chave é:

```text
(source_hash, contract_name, function_signature)
```

Padrão técnico registrado: SHA-256 sobre bytes UTF-8 exatos de contractSource,
antes de compilação/instrumentação, sem remover comentários, espaços ou normalizar
quebras de linha. Armazenar em formato fixo; detalhes de armazenamento serão
fechados com D03. Bytecode e configuração de compilação não entram na chave básica.

| Exemplo | Resultado |
| --- | --- |
| Mesmo fonte, SimpleDAO e withdraw(uint256), UUIDs/endereços distintos | Mesma chave |
| Mesmo fonte/contrato, withdraw(address) | Outra função |
| Mesmo fonte, declarações Token e Vault com withdraw(uint256) | Outro contrato |
| Fonte alterado, inclusive comentário ou espaço | Outro source_hash |

A chave é compatibilidade de identidade, não prova de estado Geth equivalente.
Serialização, cenário e métricas dirigidas podem precisar de filtros adicionais.
Não recalcular fonte/ABI em cada seleção nem impor segundo hash/tabela só para
representar a combinação.

Evidência: código-fonte e combinação confirmados pelo usuário em 2026-10-07.
SHA-256 e ausência de normalização foram registrados como padrão técnico.
D05 permanece parcialmente aberta.

### D06 — Integração mínima com o benchmark

Estado: Pendente. Responsável: —. Fechada em: —. Tarefas 17, 18, 21 e 22.

- [x] Preservar comandos independentes e laços sequenciais internos.
- [x] Preservar ambiente, endpoint/receptor e relatório próprios por chamada.
- [ ] Definir acesso das chamadas ao banco comum e configuração operacional mínima.
- [ ] Conferir roteamento por taskId e associação dos containers existentes.
- [ ] Definir efeitos mínimos de timeout/reinício, sem nova retomada automática.
- [ ] Validar caminhos de saída para não sobrescrever relatórios concorrentes.

Sem coordenador único de cinco chamadas, lançamento sincronizado ou classificação
obrigatória de grupo conjunto. O benchmark não seleciona nem muta seeds.

Decisão e evidência: fluxo preservado acordado; configuração e validação pendentes.

### D07 — Métricas e avaliação individual

Estado: Pendente. Responsável: —. Fechada em: —. Tarefas 16, 22 e 28.

- [ ] Definir métricas mínimas de origem externa, ganho local de descendentes e custo de seleção.
- [ ] Registrar fonte/configuração/versões e corpus inicialmente vazio ou acumulado.
- [ ] Mapear rótulos de B2_parted aos detectores e definir avaliação esperado/observado.
- [ ] Fixar contratos e orçamento dos testes curtos local/shared.
- [ ] Definir metadados mínimos sem exigir novo relatório conjunto.
- [ ] Registrar influência de chamadas anteriores e concorrentes na interpretação dos resultados.

Não pressupor repetições independentes nem mesmo corpus inicial quando há reuso.
Agregação/união de cobertura pode ser trabalho futuro, fora do caminho obrigatório.

Decisão e evidência: —

### D08 — Dívida técnica e preservação da seleção

Estado: Pendente. Responsável: —. Fechada em: —. Tarefas 13, 15 e 27.

- [ ] Registrar limite ignorado, ordenação crescente de DeltaCoverage e risco de score dirigido não finito.
- [ ] Registrar descarte de webhook de tarefa diferente da aguardada no coletor atual.
- [ ] Documentar comportamento preservado e mudanças mínimas introduzidas pelo compartilhamento.
- [ ] Se algum problema bloquear o escopo, abrir correção separada de até 8 pontos e aplicar mudança de política igualmente aos modos comparados.

Decisão e evidência: —

## 4. Quadro de tarefas e dependências

Estimativa revisada: **25 tarefas ativas, 164 pontos, 158–284 horas**.
Predecessoras são tarefas concluídas. Definições precisam estar fechadas para o
comportamento implementado; podem ser tratadas durante a própria tarefa responsável.
Os testes relacionados são realizados junto à implementação, não apenas no fim.

| ID | Tarefa | Pontos | Horas previstas | Predecessoras | Definições | Status |
| --- | --- | ---: | --- | --- | --- | --- |
| 01 | Isolamento mínimo e ciclo de vida | 5 | 4–8 | — | D01, D03 | Em andamento |
| 02 | Conexão PostgreSQL | 5 | 6–10 | 01 | D02 | Pendente |
| 03 | Esquema e migração | 8 | 8–14 | 01, 02 | D03 | Pendente |
| 04 | Contexto operacional mínimo | 8 | 6–12 | 03 | D01, D03 | Pendente |
| 05 | Isolamento de tarefas | 8 | 8–14 | 04 | D03 | Pendente |
| 06 | Isolamento de contratos e funções | 8 | 8–14 | 04 | D03 | Pendente |
| 07 | Isolamento de transações | 8 | 8–14 | 05, 06 | D03 | Pendente |
| 08 | Isolamento de jobs e callbacks | 8 | 8–14 | 05, 06, 07, 09 | D03, D06 | Pendente |
| 09 | Auxiliares por ambiente | 8 | 8–14 | 06 | D01, D03 | Pendente |
| 10 | SQL, erros e encerramento | 5 | 4–8 | 02, 07 | D02, D03 | Pendente |
| 11 | Representação de inputs | 5 | 6–10 | 01 | D04 | Pendente |
| 12 | Chave compartilhada da função | 5 | 4–8 | 06 | D05 | Pendente |
| 13 | Consulta local/shared | 8 | 8–14 | 11, 12, 14 | D04, D05, D08 | Pendente |
| 14 | Conclusão, elegibilidade e origem | 8 | 8–14 | 07, 10, 11, 12 | D04, D05 | Pendente |
| 15 | Integração das três greybox | 8 | 8–14 | 13, 14, 24 | D05, D08 | Pendente |
| 16 | Métricas mínimas de uso de seeds | 5 | 4–8 | 15 | D07 | Pendente |
| 17 | Banco comum e chamadas independentes | 5 | 6–10 | 02, 04, 09, 10 | D02, D06 | Pendente |
| 18 | Configuração mínima do benchmark | 5 | 4–8 | 04, 17 | D01, D06 | Pendente |
| 21 | Reinício e falhas no fluxo atual | 5 | 4–8 | 18 | D02, D06 | Pendente |
| 22 | Relatórios próprios e metadados mínimos | 5 | 4–8 | 16, 18 | D06, D07 | Pendente |
| 24 | Validação de isolamento multiprocesso | 8 | 8–14 | 05, 06, 07, 08, 09, 10 | D03, D06 | Pendente |
| 25 | Validação das estratégias e do reuso | 8 | 8–14 | 15, 16, 24 | D04, D05, D08 | Pendente |
| 26 | Validação de falhas e carga | 8 | 8–14 | 17, 21, 22, 25 | D02, D06, D07 | Pendente |
| 27 | Operação e dívida técnica | 5 | 4–8 | 22, 25, 26 | D01–D08 | Pendente |
| 28 | Comparativo curto de aceitação | 5 | 6–10 | 22, 25, 26, 27 | D07 | Pendente |

09 precede 08; 14 precede 13; 24 libera compartilhamento integrado em 15.
A fixture de dois processos para 24 não depende da infraestrutura de múltiplas
chamadas da tarefa 17. Configuração das chamadas pode avançar depois do contrato
operacional, enquanto o compartilhamento é desenvolvido.

| ID preservado | Trabalho | Status |
| --- | --- | --- |
| 19 | Coordenador paralelo único | Fora do escopo |
| 20 | Coletor central concorrente | Fora do escopo; limitação do coletor atual registrada em D08 |
| 23 | Relatório e agregação conjuntos | Fora do escopo |

Não somar pontos/horas dessas três tarefas nem marcá-las como concluídas.
Eventual reset não tem tarefa pontuada ainda; definir/orçar antes de incluí-lo.

## 5. Checkpoints por tarefa ativa

Registrar responsável, início, término, horas reais, evidência, bloqueio e próximo
passo. “—” significa ausência de registro, não zero horas.

### 01 — Isolamento mínimo e ciclo de vida

Responsável: Codex (definição documental) | Início: 2026-10-06 | Término: — | Horas reais: —

- [ ] Confirmar os IDs atuais e documentar o escopo operacional mínimo, sem exigir IDs de campanha ou repetição.
- [x] Definir separação entre instâncias, auxiliares e comportamento do banco no encerramento/reinício (D01; implementação pendente).
- [ ] Conferir o contrato com os fluxos atuais; registrar as decisões ainda necessárias.

Evidência / bloqueio / próximo passo: tabelas comuns e UUID por inicialização
confirmados; comportamento atual verificado estaticamente em D01. Preservação de
registros/estados sem retomada e auxiliares definidos; D01 fechada documentalmente.
Ainda fechar inventário e restrições de D03; tarefa 01 permanece em andamento.

### 02 — Conexão PostgreSQL

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Configurar driver, acesso, pool e readiness para uma instância.
- [ ] Demonstrar abertura, uso e fechamento da conexão.
- [ ] Verificar erro identificável na indisponibilidade e fechamento sem apagar dados.

Evidência / bloqueio / próximo passo: —

### 03 — Esquema e migração

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Adicionar campos de identidade compartilhada, contexto mínimo e restrições.
- [ ] Preparar banco vazio por procedimento único antes das instâncias.
- [ ] Verificar índices, integridade e reaplicação da migração.

Evidência / bloqueio / próximo passo: —

### 04 — Contexto operacional mínimo

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Propagar somente o contexto necessário ao isolamento nos serviços e mapeadores.
- [ ] Preservar os IDs locais e o formato de criação de tarefas sempre que possível.
- [ ] Rejeitar associações operacionais entre contextos distintos.

Evidência / bloqueio / próximo passo: —

### 05 — Isolamento de tarefas

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Aplicar isolamento a leituras, atualizações e consultas de tarefas ativas.
- [ ] Cobrir SQL direto, expiração e critérios de finalização.
- [ ] Testar IDs externos sem leitura ou alteração indevida.

Evidência / bloqueio / próximo passo: —

### 06 — Isolamento de contratos e funções

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Isolar cadastro, busca, listagem e atualização de registros locais.
- [ ] Preservar associação entre contrato, função e ambiente.
- [ ] Verificar deployments equivalentes em instâncias diferentes sem cruzamento operacional.

Evidência / bloqueio / próximo passo: —

### 07 — Isolamento de transações

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Isolar CRUD e consultas operacionais por tarefa/função.
- [ ] Distinguir hashes iguais em cadeias diferentes pelo contexto.
- [ ] Verificar que somente a consulta compartilhada de seeds atravessa a fronteira.

Evidência / bloqueio / próximo passo: —

### 08 — Isolamento de jobs e callbacks

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Revisar geração, timeout, análise e finalização pelo inventário de operações.
- [ ] Associar callbacks à instância e tarefa corretas.
- [ ] Demonstrar que um processo não executa ou finaliza tarefas de outro.

Evidência / bloqueio / próximo passo: —

### 09 — Auxiliares por ambiente

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Resolver auxiliares pelo contexto operacional e papel, evitando registro global compartilhado.
- [ ] Criar UUID próprio por auxiliar e restrição única em (instance_id, papel).
- [ ] Criar e implantar novos auxiliares por inicialização, preservando registros anteriores sem reutilizar endereços.
- [ ] Atualizar consumidores de IDs/endereços auxiliares.
- [ ] Testar instâncias distintas e encerramento/reinício conforme contrato mínimo.

Evidência / bloqueio / próximo passo: —

### 10 — SQL, erros e encerramento

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Adaptar SQL específico do SQLite, incluindo cálculo temporal.
- [ ] Verificar begin/commit/rollback e propagação de falhas.
- [ ] Preservar o banco no encerramento e impedir reenvio indevido ao Geth.

Evidência / bloqueio / próximo passo: —

### 11 — Representação de inputs

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Definir representação suficiente para os tipos suportados sem ambiguidade.
- [ ] Testar round-trip e strings com delimitadores.
- [ ] Preservar interface externa e registrar a versão do formato, se necessária.

Evidência / bloqueio / próximo passo: —

### 12 — Chave compartilhada da função

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Calcular SHA-256 do código-fonte exato e persistir nome da declaração selecionada.
- [ ] Extrair assinatura ABI completa e associar a chave aos UUIDs locais.
- [ ] Testar iguais/diferentes fontes, múltiplas declarações e funções sobrecarregadas; indexar a consulta.

Evidência / bloqueio / próximo passo: —

### 13 — Consulta local/shared

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Consultar candidatos concluídos pela chave source_hash + contract_name + function_signature.
- [ ] Preservar modo local, seleção atual e regras adicionais de compatibilidade acordadas.
- [ ] Demonstrar consulta de seeds de outra chamada e de chamadas anteriores, excluindo incompletas/incompatíveis.

Evidência / bloqueio / próximo passo: —

### 14 — Conclusão, elegibilidade e origem

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Disponibilizar o input somente com resultado utilizável e commit confirmado, sem filtro obrigatório de ganho.
- [ ] Registrar a transação de origem selecionada antes da mutação.
- [ ] Verificar rastreabilidade e ausência de cópia para segundo acervo persistente.

Evidência / bloqueio / próximo passo: —

### 15 — Integração das três greybox

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Confirmar Marco A e integrar greybox, directed_greybox e other_directed_greybox.
- [ ] Preservar escolha, mutadores e complemento de seeds insuficientes.
- [ ] Demonstrar mutação de seed externa e resultado próprio no Geth consumidor.

Evidência / bloqueio / próximo passo: —

### 16 — Métricas mínimas de uso de seeds

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Definir dados mínimos de origem, ganho local de descendentes e custo da seleção.
- [ ] Registrar uso externo e condição inicial do acervo sem criar relatório conjunto obrigatório.
- [ ] Verificar consistência dos dados e preservação das métricas atuais.

Evidência / bloqueio / próximo passo: —

### 17 — Banco comum e chamadas independentes

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Permitir que projetos Docker independentes alcancem o banco persistente.
- [ ] Preservar um par Dogefuzz/Geth e rede/callback próprios por chamada.
- [ ] Validar readiness e encerramento de uma chamada sem remover o banco comum.

Evidência / bloqueio / próximo passo: —

### 18 — Configuração mínima do benchmark

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Expor somente configuração necessária ao banco, isolamento e modo local/shared.
- [ ] Manter comando, laços sequenciais e taskId atual; não exigir coordenador de múltiplos endpoints.
- [ ] Validar chamadas independentes sem novos IDs experimentais obrigatórios.

Evidência / bloqueio / próximo passo: —

### 21 — Reinício e falhas no fluxo atual

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Verificar reinício do par pertencente à chamada e efeitos no banco persistente.
- [ ] Manter tratamento atual de timeout/erro sem implementar retomada experimental automática.
- [ ] Demonstrar que falha de uma chamada não encerra nem altera tarefas de outra.

Evidência / bloqueio / próximo passo: —

### 22 — Relatórios próprios e metadados mínimos

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Preservar estrutura atual de relatório por chamada.
- [ ] Associar versões, configuração e condição inicial do corpus em metadados mínimos de avaliação.
- [ ] Verificar roteamento de cada relatório e caminhos distintos para salvar resultados concorrentes.

Evidência / bloqueio / próximo passo: —

### 24 — Validação de isolamento multiprocesso

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Executar dois processos/Geths no banco comum com compartilhamento desativado.
- [ ] Exercitar IDs externos, hashes iguais, auxiliares, jobs e callbacks.
- [ ] Registrar evidência do Marco A; usar fixture de dois processos sem depender de 17.

Evidência / bloqueio / próximo passo: —

### 25 — Validação das estratégias e do reuso

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Exercitar três greybox nos modos local/shared.
- [ ] Verificar chave, compatibilidade, proveniência e complemento de seeds.
- [ ] Demonstrar reuso de chamadas anteriores sem reset por repetição.

Evidência / bloqueio / próximo passo: —

### 26 — Validação de falhas e carga

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Exercitar indisponibilidade do banco e reinício de uma instância.
- [ ] Medir consultas e contenção com várias chamadas independentes, registrando recursos.
- [ ] Verificar persistência, erros identificáveis e ausência de processamento cruzado ou reenvios indevidos.

Evidência / bloqueio / próximo passo: —

### 27 — Operação e dívida técnica

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Documentar banco, migração, modos, reuso do acervo e limitações.
- [ ] Registrar dívidas de seleção/scores e do coletor atual sem corrigi-las silenciosamente.
- [ ] Conferir roteiro e registrar reset como pendente, sem implementar limpeza não definida.

Evidência / bloqueio / próximo passo: —

### 28 — Comparativo curto de aceitação

Responsável: — | Início: — | Término: — | Horas reais: —

- [ ] Executar chamadas independentes em paralelo e em sequência sobre contratos selecionados de B2_parted.
- [ ] Demonstrar seeds externas, persistência após encerramento e relatórios próprios em local/shared.
- [ ] Registrar orçamento e corpus inicial; não inferir ganho científico a partir do teste curto.

Evidência / bloqueio / próximo passo: —

## 6. Checkpoints dos marcos

- [ ] **Marco A:** 01–10 e 24 concluídas; duas instâncias isoladas no banco comum. Evidência: —
- [ ] **Marco B:** Marco A e 11–16 concluídos; chave, seleção externa, mutação local e complemento demonstrados. Evidência: —
- [ ] **Marco C:** 17, 18, 21 e 22 concluídas; chamadas independentes e relatórios próprios com banco persistente. Evidência: —
- [ ] **Marco D:** marcos anteriores e 25–28 concluídos; compartilhamento validado em paralelo, sequência e chamada futura. Evidência: —

Nenhum marco requer tarefas 19, 20 ou 23.

## 7. Histórico de execução e decisões

O histórico anterior é preservado para rastreabilidade; menções a quatro workers,
corpus separado e totais anteriores descrevem o estado daquela data, não requisitos
vigentes. A revisão atual substitui o corpo anterior dos documentos.

Adicionar entradas ao alterar estado, fechar decisão, encontrar bloqueio ou revisar
escopo. Manter a evidência detalhada na tarefa/definição correspondente.

| Data | Tarefa/definição | Mudança ou decisão | Evidência / próximo passo |
| --- | --- | --- | --- |
| 2026-10-06 | Acompanhamento | Documento criado; 28 tarefas pendentes, sem implementação iniciada registrada | Referência: documento 002; próximo passo: preparação e tarefa 01 |
| 2026-10-06 | 01 / D01 | Iniciada definição documental; vocabulário e exemplos registrados, primeiro checkpoint de D01 atendido | GLOSSARY.md e seção D01; demais decisões de ciclo de vida pendentes |
| 2026-10-06 | P02 / D07 | B2_parted identificado pelo usuário como dataset do estudo anterior; catálogo conferido e exemplo atualizado para SimpleDAO | Sete listas, 71 registros e arquivos presentes; configuração histórica e mapeamento dos rótulos pendentes |
| 2026-10-07 | Escopo / D01 / D05 | Acordado reuso de seeds em chamadas paralelas e sequenciais, com acervo disponível para chamadas futuras | Corpus não muda obrigatoriamente por repetição; revisão do backlog e escopo do reset pendentes |
| 2026-10-07 | Escopo / D01 / D03 / D05 | Solicitada implementação mínima preservando fluxo e IDs atuais; foco em identificar e consultar seeds por contrato/função | Chave compartilhada e isolamento mínimo ainda a definir; novos IDs experimentais não são requisitos confirmados |
| 2026-10-07 | D05 | Confirmada chave compartilhada: identidade estável do contrato + assinatura ABI completa | Registro em D05/GLOSSARY.md; conteúdo da identidade do contrato e demais filtros ainda pendentes |
| 2026-10-07 | D05 | Confirmado código-fonte como base da identidade; registrado padrão SHA-256 do texto exato e nome da declaração selecionada | Chave: source_hash + contract_name + function_signature; filtros adicionais ainda pendentes |
| 2026-10-07 | Revisão 001/002/003 | Consolidado escopo mínimo, acervo persistente e chave por fonte/contrato/assinatura; retiradas 19/20/23 do caminho obrigatório | Backlog revisado: 25 tarefas, 164 pontos, 158–284 horas; nenhum código implementado |
| 2026-10-08 | D01 / D03 | Escolhidas tabelas comuns com instance_id; impactos e motivo de não usar schemas separados documentados | Geração/ciclo de vida e restrições ainda pendentes; estimativa mantida, nenhum código implementado |
| 2026-10-08 | D01 | Escolhida geração de UUID pelo Dogefuzz a cada inicialização; verificado que hoje só há UUIDs por registro e isolamento por SQLite/ambiente | NewEnv não gera ID e main não chama Destroy; interrupções/auxiliares pendentes, nenhum código implementado |
| 2026-10-08 | D01 | Escolhida preservação de registros e estados sem retomada, reenvio ou reclassificação automática | Transações concluídas seguem candidatas; incompletas excluídas. Auxiliares pendentes; nenhum código implementado |
| 2026-10-08 | D01 / 09 | Confirmados auxiliares com UUID próprio, unicidade por instância/papel e novos deployments a cada inicialização; histórico preservado | D01 definida documentalmente; tarefa 01 em andamento por D03 e tarefa 09 pendente. Estimativas mantidas; nenhum código implementado |
| 2026-10-08 | D03 | Confirmado instance_id obrigatório em todas as tabelas de dados persistidos, incluindo auxiliares; preenchimento interno e propriedade imutável | Abrangência do campo definida; relações, demais restrições, índices e inventário pendentes. Estimativa mantida; nenhum código implementado |
| 2026-10-08 | D03 / escopo | Esclarecida abrangência: instance_id obrigatório nas quatro entidades principais atuais; qualquer nova tabela exige confirmação prévia do usuário sobre inclusão, finalidade e contexto | Substitui a extensão automática a futuras tabelas. Estimativa mantida; nenhum código implementado |
| 2026-10-08 | D03 / integridade | Aprovadas validação na aplicação e restrições no banco; relações da mesma instância e função pertencente ao contrato da tarefa, com exceção de auxiliares sem tarefa | Desenho concreto de chaves/restrições pendente; D03 em andamento. Estimativas mantidas; nenhum código implementado |

## 8. Controle de escopo

Inicializar banco novo sem importar histórico. Preservar comandos, taskId,
relatórios, seleção e mutadores. Geths independentes; centralizar persistência
sem permitir que uma instância processe tarefas de outra.

Não criar IDs experimentais, coordenador central, relatório conjunto, replay,
remapeamento ou alta disponibilidade sem nova decisão. Reset explícito permanece
pendente, sem limpeza executada. Campanha científica completa está fora desta entrega.

Custos revisados estão no documento 002. Contingência de 20% cobre incerteza de
integração; não amplia o escopo. Reavaliar após D01/D03 e o Marco A.
