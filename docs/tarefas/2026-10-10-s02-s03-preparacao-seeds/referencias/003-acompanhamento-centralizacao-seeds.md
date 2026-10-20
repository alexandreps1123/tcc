# Acompanhamento — bancos locais e acervo compartilhado

Criado em: 2026-10-06. Última revisão: 2026-10-09.

Arquitetura vigente: persistência operacional local e banco adicional de seeds.
Reestruturação documental autorizada pelo usuário. Nenhum código, migração,
teste de aplicação ou benchmark foi executado nesta revisão.

Referências: [arquitetura](001-brainstorm-centralizacao-seeds.md),
[estimativa](002-custo-centralizacao-seeds.md) e [vocabulário](GLOSSARY.md).
A versão completa anterior, incluindo checkpoints e histórico, está no
[snapshot](docs/historico/2026-10-09-centralizacao-operacional/README.md).

## 1. Uso e situação

Marcar decisões quando aprovadas; marcar tarefas/checkpoints de implementação
somente com evidência verificável. Registrar responsável, datas, horas reais,
testes e próximo passo. Definição documental não equivale a código implementado.

| Campo | Situação |
| --- | --- |
| Situação | S01 concluída documentalmente; implementação das demais tarefas não iniciada |
| Próximo passo | Detalhar formato/compatibilidade em S02 e limites/timeout de consulta em D05; objeto aprovado e erro de consulta segue tratativa atual |
| Tarefas concluídas | 1 de 16 — S01, entrega documental |
| Pontos concluídos | 5 de 101 — S01, entrega documental |
| Estimativa | 98–176 horas; contingência de 20%: 118–212 horas |
| Responsável pelo acompanhamento | A definir |
| Pendência técnica identificada | Validar adapter e reutilização nas três greybox; ExecutedInstructions incluído no objeto após conferência dos orderers |

Os novos IDs S01–S16 substituem o backlog operacional centralizado. IDs antigos
não foram reutilizados nem marcados como concluídos.

## 2. Preparação

- [ ] Confirmar referências de código e regras locais dos repositórios antes de implementar.
- [ ] Recuperar configuração, comandos e relatórios do experimento anterior.
- [ ] Confirmar ambiente de execução, banco do acervo e acesso dos containers.
- [ ] Definir orçamento/cenários do teste curto e condição inicial do corpus.
- [ ] Confirmar responsável e evidências para cada tarefa.
- [x] Registrar a mudança arquitetural e preservar integralmente a versão anterior.

Referência informada pelo usuário: dogefuzz-benchmark/dataset/B2_parted.
A conferência estática anterior encontrou subdir1–subdir7, sete catálogos,
71 registros e arquivos existentes. Não foi repetida nem executado experimento.
Exemplo disponível: SimpleDAO em subdir7/simple_dao.sol. Rótulos conhecidos servem
de referência; verificar como interpretar resultados/detectores e recuperar o
protocolo histórico. O benchmark atual não propaga automaticamente todos os
rótulos do catálogo até o relatório.

## 3. Definições necessárias

Os códigos D01–D08 são mantidos para continuidade; os títulos e o conteúdo foram
adaptados ao novo escopo. Decisões antigas incompatíveis ficam no snapshot.

### D01 — Fronteira operacional e ciclo de vida

Estado: Definida documentalmente para a arquitetura vigente em 2026-10-09.
Relacionada: S01, S11, S12, S15.

- [x] Manter tarefas, contratos, funções e transações no banco local de cada Dogefuzz.
- [x] Manter UUIDs/taskId, Geth, auxiliares, jobs e callbacks no ambiente atual.
- [x] Acrescentar acervo comum de seeds, sem coordenar execução de contratos.
- [x] Permitir cooperação paralela, sequencial e em chamadas futuras.
- [x] Preservar acervo sem reset automático por repetição ou encerramento.
- [x] Manter relatórios e comandos por chamada, sem IDs experimentais novos obrigatórios.
- [x] Não incluir retomada ou nova garantia de persistência do histórico operacional local.
- [x] Exigir confirmação prévia caso uma nova tabela seja necessária.

Exemplo: A e B executam o mesmo código, com registros e estados locais distintos.
B consulta uma entrada publicada por A e produz uma transação local. Não lê nem
altera tasks/contracts/functions/transactions do SQLite de A.

A obrigatoriedade de instance_id nas quatro entidades e o redesign dos auxiliares
eram soluções para a centralização operacional; não são requisitos ativos.
Identificação de origem é opção futura e não será usada na entrega inicial.
Preservar registros sem retomada continua excluindo recuperação de tarefas;
somente o acervo recebe a garantia nova de sobreviver ao encerramento.
Não alterar por esta decisão a persistência/remoção do SQLite atual.

### D02 — Banco adicional, migração e acesso

Estado: Pendente. Relacionada: S04, S05, S09, S11, S14, S15.

- [ ] Confirmar PostgreSQL para o acervo, versão e configuração; proposta para estimativa, não migração do SQLite.
- [ ] Definir conexão própria, timeout, pool e comportamento de startup sem acervo.
- [ ] Definir aplicação única/reaplicável da migração e coordenação entre chamadas.
- [ ] Definir acesso de projetos Docker independentes e volume persistente do acervo.
- [x] Definir tratamento de operação: erro de consulta segue propagação atual; falha de publicação é aceita.
- [ ] Definir tratamento de falha na inicialização/conexão, separado do erro de consulta.
- [ ] Documentar fechamento das conexões e preservação do acervo.

### D03 — Esquema do acervo e consistência de publicação

Estado: Em andamento documental. Relacionada: S01, S05, S06, S09, S14.

- [x] Substituir esquema operacional unificado por acervo separado de seeds.
- [x] Não adicionar instance_id às quatro entidades locais para isolamento central.
- [x] Exigir confirmação antes de criar qualquer nova tabela.
- [ ] Confirmar esquema mínimo, nomes/tabelas e campos necessários à seleção, sem referência de origem.
- [x] Definir repetidas: registrar cada publicação separadamente, sem deduplicação, substituição de métricas ou idempotência exigida inicialmente.
- [x] Definir momento/frequência: a cada transação concluída e compatível, após análise e commit local confirmado, sem esperar lote ou fim da tarefa.
- [x] Escolher publicação síncrona inicialmente; manter assíncrona como possibilidade futura.
- [x] Definir ponto de integração: chamada explícita ao final do listener de análise, após gravações locais bem-sucedidas da transação e da tarefa.
- [x] Escolher publicação simples: aceitar falha de publicação e continuar localmente, sem outbox ou garantia de envio posterior.
- [x] Definir timeout de publicação configurável, inicialmente 500 ms para a tentativa inteira, com comentários das opções de ajuste.
- [x] Definir quantidade de tentativas configurável, inicialmente 1 no total, incluindo a primeira tentativa; sem repetição automática nesse valor.
- [x] Registrar origem como opção futura, sem incluir/utilizar esse campo ou linhagem na implementação inicial.
- [ ] Definir retenção dos metadados de seleção.
- [ ] Definir índices e limites de crescimento do acervo.
- [x] Definir commit remoto ambíguo: não reenviar ao Geth; nova tentativa pode inserir repetida, aceita.
- [ ] Detalhar critérios/intervalo se tentativas configuradas forem maiores que 1.

Proposta mínima: registro de seed com chave estável da função, inputs com separador `;`,
ID próprio por registro e metadados aprovados de seleção, sem unicidade por inputs. Uma tabela
é uma hipótese de desenho, ainda não aprovada; corpora, observations e outbox
não são automaticamente necessárias. A opção futura de origem não implica FK para banco remoto.

Decisão do usuário: origem da seed fica como opção futura e não será utilizada
na implementação inicial. Não incluir no candidato/esquema ativo os IDs de tarefa,
transação ou instância produtora para rastreabilidade, nem registrar linhagem dos
descendentes. Uma referência futura poderia auxiliar diagnóstico/auditoria, mas
exige nova decisão. Métricas atuais podem indicar uso do acervo como fonte, sem
identificar produtor ou afirmar que todo candidato do acervo veio de outra instância.
Metadados de pontuação continuam necessários conforme D05, sem que sua presença
identifique origem ou represente ganho obtido pelo consumidor. Campos temporais
de ordenação/retenção serão definidos separadamente.

**Decisão de 2026-10-09: publicação simples, sem outbox; falha aceita pelo usuário.**
Após salvar o resultado local, tentar publicar com tempo de espera limitado. Se
falhar, registrar o erro e continuar o fuzzing, preservando o resultado local, sem
garantia de envio posterior. Pode perder a oportunidade de compartilhar aquela
entrada. Quantidade de tentativas configurável, inicialmente 1. Repetição após commit remoto já realizado pode inserir outro registro, o que é aceito. Garantir entrega posterior exige estado persistente,
rotina de reenvio e nova estimativa; não está incluído por padrão.

Frequência confirmada em 2026-10-09: tentar publicar a cada transação concluída e
compatível, após análise e persistência local confirmada. Não esperar lote ou fim
da tarefa, nem publicar apenas porque a transação foi enviada ao Geth. Aumenta
o volume de operações no acervo; medir latência/custo. Publicações repetidas são aceitas.
Decisão de 2026-10-09: publicação síncrona inicialmente. O caminho de processamento
aguarda sucesso, erro ou timeout depois da persistência local e então continua.
Falha não desfaz o resultado local nem exige entrega posterior.

Ponto de integração confirmado: ao final do listener de análise, chamar
explicitamente o componente de seeds logo após salvar a transação concluída e
a atualização da tarefa com sucesso. O componente cuida da elegibilidade,
representação e acesso ao acervo. Não acoplar publicação ao Update genérico.
Se uma gravação local falhar, não tentar publicar naquele evento. As gravações
atuais são separadas: a transação pode permanecer concluída mesmo se a gravação
da tarefa falhar. Não haverá recuperação garantida dessa publicação nem nova
garantia de atomicidade local. Falha remota preserva os resultados locais.
Referência inspecionada: [listener](dogefuzz/listener/executation_analytics.go).

Decisão de 2026-10-09: timeout configurável, inicialmente 500 ms para a tentativa
inteira de publicação, incluindo espera por conexão e gravação/confirmação.
Não renovar o limite em cada etapa. Ao expirar, registrar falha e continuar localmente.
Na implementação, definir nome/unidade e validar valor positivo; documentar junto
ao parâmetro as opções ilustrativas: 100 ms reduz espera e tolerância a oscilações;
500 ms é o valor inicial escolhido; 1 segundo tolera respostas mais lentas com maior
custo de espera. Opções não são valores exclusivos nem medições de desempenho.
Timeout de consulta permanece separado e pendente. Timeout pode ocorrer depois
do commit remoto; não presumir que nenhuma seed foi gravada. Nova tentativa pode
inserir outro registro, aceito sem tratamento especial.

Decisão de 2026-10-09: quantidade de tentativas configurável, inicialmente 1.
Conta o total, incluindo a primeira tentativa. Com 1, erro/timeout encerra a
publicação daquela entrada, registra falha e permite continuidade local, sem retry
ou recuperação persistente. Na implementação, documentar e validar inteiro positivo.
Para valores maiores que 1, repetir apenas a publicação da mesma entrada, nunca
o envio ao Geth. O timeout de 500 ms é por tentativa; mais tentativas aumentam
a espera acumulada. Critérios de erro e intervalo entre tentativas precisam ser
detalhados antes de usar valores maiores; nenhuma entrega posterior é garantida.

Assíncrona é possibilidade futura se a latência medida justificar a mudança;
não implementar fila/publicador agora. Antes de adotá-la, definir capacidade da
fila, tratamento de fila cheia, concorrência e encerramento; reestimar tarefas.
Essa possibilidade não aprova outbox ou garantia de entrega persistente.

As decisões antigas de validação/FKs entre instâncias permanecem históricas:
não são necessárias para compartilhar apenas seeds com bancos locais separados.
Integridade do acervo e validação dos candidatos continuam necessárias.

### D04 — Objeto de candidato e formato dos inputs

Estado: Parcialmente definida. Objeto aprovado; formato/compatibilidade pendentes.
Relacionada: S02, S06–S08, S13.

Objeto aprovado para publicação/consulta, a adaptar aos DTOs e à seleção
existentes. A compatibilidade efetiva será validada em S02/S08:

```go
type SeedCandidate struct {
    // SHA-256 da fonte UTF-8 exata recebida, antes de compilar/instrumentar.
    // Compõe a identidade compartilhada com contrato e assinatura da função.
    SourceHash string

    // Nome da declaração selecionada; distingue contratos na mesma fonte.
    ContractName string

    // Assinatura ABI completa com tipos canônicos, como "withdraw(uint256)".
    // Distingue funções sobrecarregadas; não é o UUID local da função.
    FunctionSignature string

    // Argumentos serializados na ordem dos parâmetros da ABI.
    // O formato deve preservar os tipos suportados e permitir desserialização.
    Inputs []string

    // Instruções executadas na avaliação produtora, no formato do TransactionDTO.
    // A estratégia distance_coverage_based usa o tamanho desta lista para cobertura.
    // Preservar a lista; não substituir informação ausente por uma lista vazia.
    ExecutedInstructions []string

    // Ganho de cobertura observado em relação ao acumulado da tarefa produtora.
    // No cálculo atual, conta novos pontos de cobertura; não é porcentagem
    // nem garantia de ganho na execução consumidora.
    DeltaCoverage uint64

    // Métrica produzida pelo cálculo atual de variação da distância aos alvos,
    // considerando o acumulado da tarefa produtora. Usada na seleção dirigida;
    // preservar a semântica atual, inclusive seus casos especiais.
    DeltaMinDistance uint64

    // Contagem de ocorrências de instruções classificadas como críticas
    // na execução avaliada, usada pelas estratégias dirigidas.
    // Não representa uma quantidade de vulnerabilidades confirmadas.
    CriticalInstructionsHits uint64
}
```

Não incluir origem, IDs operacionais ou endereço do deployment. O registro no
acervo pode ter ID próprio; ele não integra o objeto de candidato aprovado.
O adapter deve converter dados para a estrutura esperada sem alterar seleção
ou mutadores. Tipos e compatibilidade serão verificados em S02; hash e assinatura em S03.

Decisão de S02 (2026-10-10): manter a representação atual com separador `;`
entre argumentos, usando Join/Split sem adicionar escape ou converter para JSON.
Inputs permanece []string no objeto; o adapter faz a conversão na persistência.
O gerador e os mutadores de string atuais usam somente letras, e as seeds padrão
são "foo" e "bar". Entretanto, Deserialize aceita strings contendo `;`; esses
valores tornam a separação ambígua e não têm round-trip garantido. Esta limitação
fica registrada, sem introduzir tratamento novo nesta decisão.

Decisão de S02 (2026-10-10): remover o campo de versão do SeedCandidate.
Não exigir versionamento, filtro por versão ou conversão entre versões nesta entrega.
A compatibilidade permanece baseada na identidade da função e nos handlers atuais.

Decisão de S02 (2026-10-10): reutilizar os valores dos argumentos como estão,
inclusive endereços, sem excluir candidatos por dependência do ambiente e sem
remapear contas ou contratos auxiliares. Após a recuperação, aplicar normalmente
as avaliações e mutações existentes; não exigir reprodução do resultado original.
Um endereço pode não ter código ou saldo na execução consumidora, e o estado do
contrato pode ser diferente. Esses efeitos de contexto são aceitos nesta entrega.
A verificação técnica de serialização/desserialização dos tipos já suportados
continua necessária; esta decisão não adiciona suporte a tipos nem uma política
nova para dados inválidos ou falhas de desserialização.

Decisão de S02 (2026-10-10): manter o comportamento atual para erros detectados
na desserialização. RequestSeeds() retorna o erro, GenerateInput() o propaga e o
listener registra a falha e encerra o processamento daquele evento. Não descartar
individualmente candidatos, corrigir valores, excluir/marcar registros no acervo
ou adicionar fallback para essa falha. Um candidato selecionado inválido pode
impedir o aproveitamento dos demais naquela chamada. O complemento com seeds
configuradas continua reservado à consulta/desserialização bem-sucedida com
candidatos insuficientes; não garantir detecção de toda ambiguidade do separador.

Requisito confirmado em S02 (2026-10-10): SeedCandidate deve ser totalmente
compatível com o sistema atual de seeds e permitir reutilização entre execuções
de contratos iguais, pela fonte exata, declaração e assinatura ABI já definidas.
Para atender às três estratégias greybox, o objeto inclui ExecutedInstructions
com o mesmo tipo do TransactionDTO atual. A análise estática dos orderers confirma:
coverage_based usa DeltaCoverage; distance_based usa CriticalInstructionsHits e
DeltaMinDistance; distance_coverage_based usa também ExecutedInstructions.
O adapter preserva esses campos e Inputs sem substituir dados ausentes por zeros.
CFG, DistanceMap e TargetInstructionsFreq continuam vindo do contrato consumidor,
como no fluxo atual; IDs locais não identificam o contrato compartilhado.
Preservar handlers, seleção e mutação atuais. Compatibilidade aqui significa
preservar a representação e os dados utilizados pelos algoritmos, não garantir
estado blockchain ou resultado de execução iguais. Verificação funcional do
adapter e dos tipos em S02/S08 continua pendente; análise estática não conclui
essas tarefas. Critérios de consulta e ordenação temporal permanecem em D05/D03.

- [x] Exigir compatibilidade completa com o sistema atual de seeds, incluindo ExecutedInstructions para seleção dirigida, sem referência de origem.
- [x] Manter representação dos inputs com separador `;`, conforme estrutura atual, sem escape novo.
- [x] Remover o campo de versão do objeto, sem exigir versionamento nesta entrega.
- [ ] Verificar tipos realmente suportados e round-trip sem perda.
- [ ] Verificar listas e coleções e documentar os limites do separador `;`.
- [x] Reutilizar valores como estão, inclusive endereços, sem exclusão por contexto ou remapeamento.
- [ ] Verificar casos não suportados pelos handlers atuais.
- [x] Manter propagação atual de falhas de desserialização, sem descarte individual, correção, exclusão de registros ou fallback novo.

Hash de argumentos não é requisito de deduplicação.
O separador local foi mantido por decisão explícita; verificar tipos e registrar
limitações de representação. Suporte a tipos adicionais ao atual exige tarefa própria.

### D05 — Identidade, elegibilidade e seleção

Estado: Parcialmente definida. Relacionada: S02, S03, S06–S08, S13.

- [x] Identidade por SHA-256 do contractSource UTF-8 exato + contract_name + assinatura ABI completa.
- [x] Não normalizar fonte nem usar endereço/UUID como identidade compartilhada.
- [x] Calcular identidade uma vez no cadastro; reutilizar na consulta/publicação.
- [x] Compartilhar entre chamadas paralelas, sequenciais e futuras sem grupo/repetição obrigatório.
- [x] Preservar seleção/mutação e complemento atual de candidatos insuficientes.
- [x] Não impor filtro obrigatório de ganho positivo; somente concluídas podem ser candidatas.
- [x] Definir ponto e fonte: no modo shared, consultar apenas o acervo em RequestSeeds(), antes da seleção; modo local mantém caminho atual.
- [ ] Definir filtros adicionais de cenário, construtor, formato e instrumentação.
- [ ] Definir fallback/receive e funções sem argumentos.
- [x] Incluir dados usados pelos três orderers no SeedCandidate, incluindo ExecutedInstructions.
- [ ] Validar adapter e compatibilidade dirigida com os dados do contrato consumidor.
- [x] Retirar combinação local/shared do escopo ativo; consulta de candidatos somente ao acervo no modo shared.
- [ ] Definir limites e ordenação; consulta direta em RequestSeeds() já escolhida.
- [x] Retirar preferência por métricas locais: não há consulta local de candidatos no modo shared.
- [x] Aceitar registros de entradas repetidas no acervo e na seleção, cada um com suas métricas; histórico local permanece atual.
- [ ] Definir tratamento de tipos/funções não suportados conforme D04; não introduzir deduplicação.
- [ ] Definir armazenamento/cache da identidade sem migração operacional ampla.

Exemplo: withdraw(uint256) de SimpleDAO é reconhecida entre deployments pela
mesma fonte, declaração e assinatura. Isso não afirma estado blockchain igual.
Score/DeltaCoverage de A não é ganho de B; B muta e avalia localmente.

Orientação vigente revisada: no modo shared, RequestSeeds() consulta candidatos
somente no acervo compartilhado, antes da seleção. Não consultar histórico local
como segunda fonte ou fallback. Leituras locais de contrato/função/ABI continuam
necessárias e não representam consulta de candidatos. Modo local mantém caminho
atual para o comparativo. Não exigir cache periódico na entrega inicial.

União local/shared e preferência por métricas locais foram substituídas; não são
requisitos ativos. Publicação que falhar mantém resultado local, mas aquela entrada
não participa da seleção shared enquanto não estiver no acervo. Se faltarem
candidatos após consulta bem-sucedida, usar complemento atual de seeds configuradas.
Em erro, propagar como atualmente, sem fallback novo ou consulta de candidatos locais.
RequestSeeds() retorna o erro, GenerateInput() o propaga e o listener de fuzzing
registra a falha e encerra aquele evento. Isso não implica encerrar todo o processo.
Timeout de consulta ainda será definido; publicação mantém sua política distinta.

Decisão vigente: registrar publicações repetidas sem tratamento especial. Cada
publicação aceita cria um registro com seus inputs/métricas; não substituir primeira
por última ou melhor, não combinar campos nem deduplicar na seleção. Registros
iguais podem ocupar várias posições, como ocorre atualmente no histórico local.
Não exigir idempotência; uma repetição após timeout com commit remoto pode criar
outra linha. Medir crescimento e efeito no conjunto de candidatos. Formato, limites
e ordenação continuam pendentes. Definição documental; nenhum código implementado.

### D06 — Benchmark e falhas

Estado: Pendente. Relacionada: S09, S11–S14.

- [ ] Definir modo local/shared e comportamento de publicação em cada modo.
- [ ] Propagar somente configuração necessária ao acesso ao acervo.
- [ ] Definir infraestrutura independente e persistente; não usar down do benchmark para removê-la.
- [x] Distinguir consulta com erro (propagar) de sucesso insuficiente (complementar); publicação falha aceita.
- [ ] Definir timeout da consulta, startup e encerramento com publicação em curso.
- [ ] Verificar receptor/relatório próprios e destinos de arquivos concorrentes.
- [ ] Documentar efeito real de reinício e perda possível do histórico local.

No modo local, recomendação: não consultar nem publicar no acervo, para preservar
o baseline. Isso precisa de confirmação, inclusive efeito no experimento.

### D07 — Métricas e comparativo

Estado: Pendente. Relacionada: S10, S12, S16.

- [ ] Definir custo/latência, consultas, publicações, erros, complemento configurado e uso do acervo.
- [x] Excluir identificação do produtor e linhagem de descendentes da entrega inicial; origem fica como opção futura.
- [ ] Preservar cobertura/vulnerabilidades de cada tarefa no relatório próprio.
- [ ] Recuperar configuração anterior e selecionar cenários de B2_parted.
- [ ] Fixar orçamento/recursos e registrar corpus inicial vazio ou acumulado.
- [ ] Separar aceite curto de campanha científica e independência experimental.

### D08 — Seleção atual e dívida técnica

Estado: Pendente. Relacionada: S07, S08, S15.

- [ ] Inventariar limite ignorado, ordenação por cobertura e score dirigido não finito.
- [ ] Verificar se algum problema impede o novo fluxo.
- [ ] Se necessário, abrir correção separada de até 8 pontos e aplicar aos dois modos.
- [ ] Registrar limitação do coletor que descarta webhook de outra tarefa.
- [ ] Preservar política atual enquanto alterações não forem aprovadas.

## 4. Tarefas e dependências

A ordem abaixo é válida para execução. Predecessoras permitem antecipar caminhos
independentes; nenhuma mudança de código foi implementada. S01 está concluída
como entrega documental. Demais tarefas estão pendentes.

| ID | Tarefa | Pontos | Horas | Predecessoras | Definições | Status |
| --- | --- | ---: | --- | --- | --- | --- |
| S01 | Contrato mínimo e fluxo | 5 | 4–8 | — | D01, D03 | Concluída documentalmente |
| S02 | Formato e compatibilidade dos inputs | 8 | 8–14 | S01 | D04, D05 | Pendente |
| S03 | Identidade estável da função | 5 | 4–8 | S01 | D05 | Pendente |
| S04 | Conexão exclusiva do acervo | 5 | 6–10 | S01 | D02 | Pendente |
| S05 | Esquema e migração do acervo | 8 | 8–14 | S02, S03, S04 | D02, D03 | Pendente |
| S06 | Publicação de entradas concluídas | 8 | 8–14 | S05 | D03, D05 | Pendente |
| S07 | Consulta de candidatos do acervo | 8 | 8–14 | S05 | D05, D08 | Pendente |
| S08 | Integração das três greybox | 8 | 8–14 | S06, S07 | D04, D05, D08 | Pendente |
| S09 | Falhas e tratativa atual | 5 | 4–8 | S06, S07 | D02, D03, D06 | Pendente |
| S10 | Métricas mínimas de compartilhamento | 5 | 4–8 | S08, S09 | D07 | Pendente |
| S11 | Infraestrutura persistente do acervo | 5 | 6–10 | S04, S05 | D02, D06 | Pendente |
| S12 | Configuração e relatórios do benchmark | 5 | 4–8 | S10, S11 | D06, D07 | Pendente |
| S13 | Validação de reuso entre execuções | 8 | 8–14 | S08, S09, S11 | D04, D05, D06 | Pendente |
| S14 | Validação de falhas e concorrência | 8 | 8–14 | S09, S10, S11 | D02, D03, D06 | Pendente |
| S15 | Operação e limitações | 5 | 4–8 | S12, S13, S14 | D01–D08 | Pendente |
| S16 | Comparativo curto de aceitação | 5 | 6–10 | S12, S13, S14, S15 | D07 | Pendente |

As definições precisam estar suficientemente fechadas para a entrega dependente.
S01 concluiu contrato e política de publicação como entrega documental; não
dependeu de implementar o banco. D03/D04/D05 continuam parciais para tarefas seguintes. Não iniciar S05 sem confirmar novas tabelas e desenho de D03.

## 5. Checkpoints por tarefa

### S01 — Contrato mínimo e fluxo

Status: Concluída documentalmente em 2026-10-09; nenhum código implementado.
Responsável: Codex (documentação) | Início: 2026-10-09 | Término: 2026-10-09 | Horas reais: não medidas.

- [x] Confirmar as fronteiras: quatro entidades locais, acervo separado e nenhuma retomada nova.
- [x] Mapear seleção e conclusão local; definir publicação mínima, falhas e campos necessários.
- [x] Aprovar SeedCandidate proposto, a adaptar à estrutura atual; validação dos modos dirigidos em S02/S08.
- [x] Manter tratativa atual de erro de consulta, distinguindo consulta vazia bem-sucedida.
- [x] Definir garantia de publicação: aceitar falha, preservar resultado local e continuar, sem outbox ou entrega posterior garantida.
- [x] Definir frequência: a cada transação concluída e compatível, após análise e gravação local confirmada.
- [x] Definir execução: publicação síncrona inicialmente; assíncrona como possibilidade futura, fora da entrega inicial.
- [x] Definir timeout de publicação: parâmetro configurável com valor inicial de 500 ms e comentários das opções de ajuste.
- [x] Definir tentativas: parâmetro configurável com valor inicial 1, contando a primeira tentativa.
- [x] Definir ponto de publicação: chamada ao final do listener de análise, após salvar transação concluída e atualização da tarefa com sucesso.
- [x] Definir tratamento de origem: opção futura, campo não utilizado na entrega inicial.
- [x] Definir ponto de consulta: RequestSeeds() busca candidatos somente no acervo no modo shared.
- [x] Registrar substituição de união local/shared e preferência por métricas locais; consulta somente ao acervo no modo shared.
- [x] Definir repetidas: registrar sem tratamento, preservando cada publicação e suas métricas.
- [x] Registrar decisões e pendências antes de iniciar esquema/integração.

Evidência / próximo passo: contrato documental consolidado em 001–003 e código atual conferido em RequestSeeds(), GenerateInput() e listeners de análise/fuzzing. SeedCandidate aprovado; erro de consulta propagado sem fallback novo. Próximo: S02 (formato e validação de compatibilidade); D03/D05 continuam parciais. Não declarar compatibilidade dirigida completa antes da validação.

### S02 — Formato e compatibilidade dos inputs

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [x] Confirmar manutenção do separador `;` na representação persistida dos inputs.
- [x] Remover o campo de versão e a exigência de versionamento.
- [x] Reutilizar argumentos como estão, sem exclusão ou remapeamento de endereços.
- [ ] Verificar round-trip de argumentos, delimitadores e coleções e casos não suportados pelos handlers atuais.
- [x] Manter tratamento atual de erro de desserialização, sem descarte de candidatos ou fallback novo.
- [ ] Registrar compatibilidade de formato sem alterar silenciosamente o baseline.

Evidência / bloqueio / próximo passo: decisão de 2026-10-10 mantém Join/Split
com `;`. Análise estática de pkg/solidity/string.go e config.json confirma que
geração/mutação e seeds padrão não introduzem `;`, mas Deserialize aceita o
caractere. Não garantir round-trip para esse caso. Campo de versão removido por decisão do usuário. Valores reutilizados como estão, inclusive endereços. Tratamento atual de erro de desserialização mantido. ExecutedInstructions incluído para compatibilidade dirigida após conferir os orderers. Próximo: validar o adapter, os tipos e a reutilização entre contratos iguais.
Nenhum código implementado; S02 permanece pendente.

### S03 — Identidade estável da função

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Implementar hash da fonte exata e assinatura ABI canônica, sem usar UUID/endereço como chave compartilhada.
- [ ] Verificar fonte igual/diferente, várias declarações e sobrecargas.
- [ ] Reutilizar identidade calculada, evitando hash a cada consulta.

Evidência / bloqueio / próximo passo: —

### S04 — Conexão exclusiva do acervo

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Configurar conexão própria do acervo, independente de SQLite e desabilitada no modo local.
- [ ] Definir timeout/pool e comportamento de inicialização em falha.
- [ ] Verificar fechamento sem apagar acervo ou comprometer conexão local.

Evidência / bloqueio / próximo passo: —

### S05 — Esquema e migração do acervo

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Obter confirmação prévia do esquema e de cada nova tabela necessária.
- [ ] Implementar ID único por registro, índices e migração do acervo, sem unicidade por inputs e sem migrar entidades locais.
- [ ] Verificar migração reaplicável e publicações concorrentes iguais criando registros independentes.

Evidência / bloqueio / próximo passo: —

### S06 — Publicação de entradas concluídas

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Localizar o ponto de conclusão e commit local confirmado; excluir incompletas/incompatíveis.
- [ ] Inserir cada publicação como registro independente, com metadados de seleção aprovados, sem origem e sem deduplicação.
- [ ] Exercitar falha antes/depois do commit remoto; não repetir envio ao Geth nem afirmar entrega garantida.

Evidência / bloqueio / próximo passo: —

### S07 — Consulta de candidatos do acervo

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Consultar candidatos compatíveis com limite e timeout definidos.
- [ ] Consultar apenas o acervo no modo shared, adaptar à seleção e complementar com seeds configuradas quando necessário; sem deduplicar repetidas.
- [ ] Verificar candidatos de outra execução sem leitura de seu banco operacional.

Evidência / bloqueio / próximo passo: —

### S08 — Integração das três greybox

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Adaptar candidatos às três estratégias sem substituir mutadores atuais.
- [ ] Validar compatibilidade dos metadados dirigidos e separar métricas do candidato compartilhado de ganho local.
- [ ] Demonstrar mutação e resultado no Geth consumidor.

Evidência / bloqueio / próximo passo: —

### S09 — Falhas e tratativa atual

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Aplicar tratativa atual de erro de consulta/timeout; tratar candidato inválido sem fallback novo não aprovado.
- [ ] Verificar complemento configurado após consulta bem-sucedida insuficiente, propagação de erro na falha e tempo de espera limitado.
- [ ] Distinguir erro remoto de erro local; documentar oportunidades de publicação perdidas.

Evidência / bloqueio / próximo passo: —

### S10 — Métricas mínimas de compartilhamento

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Definir métricas do candidato selecionado e uso do acervo, sem identificação do produtor ou linhagem.
- [ ] Registrar custo de consultas/publicação, erros, complemento configurado e uso do acervo sem misturar cobertura de tarefas.
- [ ] Conferir o que a API atual permite medir e documentar ausência de atribuição a produtor/descendentes.

Evidência / bloqueio / próximo passo: —

### S11 — Infraestrutura persistente do acervo

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Configurar banco/volume independente e acesso por projetos Docker distintos.
- [ ] Validar readiness e endereço acessível pelos containers.
- [ ] Verificar que down de um benchmark preserva acervo e não encerra outra chamada.

Evidência / bloqueio / próximo passo: —

### S12 — Configuração e relatórios do benchmark

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Expor somente modo/acesso do acervo; preservar fluxo e IDs do benchmark.
- [ ] Registrar configuração/corpus inicial e destino distinto dos relatórios.
- [ ] Verificar webhook e relatório por chamada, sem coletor global.

Evidência / bloqueio / próximo passo: —

### S13 — Validação de reuso entre execuções

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Executar duas instâncias com SQLite e Geth próprios, demonstrando reuso paralelo.
- [ ] Demonstrar reuso sequencial/futuro, três greybox e mutação local.
- [ ] Testar acervo vazio, não publicação de entradas incompletas, propagação atual de erro de desserialização e preservação dos resultados locais.

Evidência / bloqueio / próximo passo: —

### S14 — Validação de falhas e concorrência

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Exercitar indisponibilidade, timeout e publicações duplicadas concorrentes.
- [ ] Exercitar encerramento e resposta perdida após commit, respeitando garantia aprovada de publicação.
- [ ] Registrar latência/carga, propagação de erro na consulta e preservação local em falha de publicação; repetidas aceitas, sem reenvio ao Geth.

Evidência / bloqueio / próximo passo: —

### S15 — Operação e limitações

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Documentar operação, migração, modos e preservação do acervo.
- [ ] Documentar perda possível de publicação/histórico local e dívida técnica.
- [ ] Conferir roteiro; manter reset fora da entrega enquanto não definido.

Evidência / bloqueio / próximo passo: —

### S16 — Comparativo curto de aceitação

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Selecionar contratos/cenários de B2_parted e recuperar configuração anterior quando disponível.
- [ ] Executar local/shared com orçamento, recursos e condição inicial do acervo registrados.
- [ ] Consolidar relatórios e custos; distinguir aceite funcional de conclusão científica.

Evidência / bloqueio / próximo passo: —

## 6. Marcos e aceite

- [ ] Marco A: S01–S05 concluídas; esquema aprovado, conexão e identidade verificadas.
- [ ] Marco B: S06–S10 concluídas; publicação/consulta, mutação local, complemento configurado, tratativa de erros e métricas.
- [ ] Marco C: S11–S12 concluídas; comandos independentes alcançam acervo persistente.
- [ ] Marco D: S13–S16 concluídas; reuso/falhas, operação e comparativo curto registrados.

Critério central: A publica entrada concluída, B a consulta pela chave,
muta e avalia no Geth B, conservando seu próprio histórico/relatório.
Confirmar reuso futuro; consulta bem-sucedida vazia usa complemento, consulta com erro segue tratativa atual.
Não afirmar entrega de todas as seeds: publicação simples foi aprovada e admite falhas.

## 7. Histórico

As decisões de 2026-10-06 a 2026-10-08 e respectivos checkpoints foram preservadas
integralmente no [snapshot anterior](docs/historico/2026-10-09-centralizacao-operacional/003-acompanhamento-banco-centralizado.md).
Não foram apagadas nem transformadas em implementação concluída. As linhas
abaixo registram a sequência das decisões, incluindo escolhas substituídas;
as seções D01–D08 e o quadro de tarefas descrevem a situação vigente.

| Data | Item | Decisão / ação | Resultado |
| --- | --- | --- | --- |
| 2026-10-09 | Arquitetura | Usuário escolheu persistência operacional análoga à atual e banco compartilhado somente de seeds | Centralização das quatro entidades substituída; docs reestruturados |
| 2026-10-09 | Backlog | Novo backlog S01–S16 para contrato, acervo, integração, benchmark e validação | 101 pontos, 98–176 horas preliminares; publicação simples e esquema mínimo ainda são hipóteses |
| 2026-10-09 | Controle | Histórico preservado; D01 reformulada; D03 passa a tratar esquema/publicação do acervo | Nenhum código implementado, nenhuma migração ou execução de benchmark |
| 2026-10-09 | S01 / D03 | Usuário aceitou falha de publicação; aprovada publicação simples com continuidade local, sem outbox ou entrega posterior garantida | Checkpoint da garantia concluído; momento de integração, timeout e tentativas pendentes. Estimativa mantida; nenhum código implementado |
| 2026-10-09 | S01 / D03 | Usuário escolheu publicação a cada transação; registrar somente concluídas/compatíveis após análise e commit local confirmado | Checkpoint de frequência concluído; execução síncrona/assíncrona e ponto exato de integração pendentes. Nenhum código implementado |
| 2026-10-09 | S01 / D03 | Usuário escolheu publicação síncrona inicialmente, mantendo assíncrona como possibilidade futura | Checkpoint de execução concluído; timeout, tentativas e integração pendentes. Estimativas mantidas; nenhum código implementado |
| 2026-10-09 | S01 / D03 | Usuário definiu timeout configurável de publicação, inicialmente 500 ms, com comentários sobre opções de ajuste | Checkpoint de timeout concluído; tentativas e integração pendentes. Estimativas mantidas; nenhum parâmetro implementado no código |
| 2026-10-09 | S01 / D03 | Usuário definiu quantidade de tentativas configurável, inicialmente 1 no total, incluindo a primeira | Checkpoint de quantidade concluído; integração e detalhes para valores maiores pendentes. Estimativas mantidas; nenhum código implementado |
| 2026-10-09 | S01 / D03 | Usuário escolheu publicação logo após gravações locais; chamada explícita ao final do listener após salvar transação e tarefa com sucesso | Checkpoint de integração concluído; dados mínimos do componente pendentes. Gravações locais separadas; estimativas mantidas, nenhum código implementado |
| 2026-10-09 | S01 / D03 / D07 | Usuário solicitou registrar origem como opção, mas não utilizar o campo na entrega inicial | Referência de produtor/linhagem excluída do esquema ativo e métricas; opção futura preservada. Estimativas mantidas; nenhum código implementado |
| 2026-10-09 | S01 / D05 | Usuário corrigiu a orientação para buscar local e compartilhado antes da seleção, dentro de RequestSeeds() | Substitui consulta só ao acervo; ponto/fontes definidos. Combinação, limites e timeout pendentes; nenhum código implementado |
| 2026-10-09 | S01 / D05 / S07 | Usuário escolheu união de candidatos locais/compartilhados com deduplicação antes da seleção | Sem cotas/prioridade por fonte; métricas de duplicatas e representação pendentes. Estimativas mantidas; nenhum código implementado |
| 2026-10-09 | D05 / S07 | Usuário escolheu preferir métricas locais quando uma entrada aparece no local e no acervo | Conservar avaliação local inteira, sem prioridade geral por fonte nem mistura de campos. Política de publicações repetidas no acervo pendente; nenhum código implementado |
| 2026-10-09 | Revisão S01 / D05 / S07 | Usuário decidiu consulta de candidatos exclusivamente ao acervo dentro de RequestSeeds() no modo shared | Substitui união local/shared e preferência por métricas locais. Modo local preservado; política de repetidas no acervo pendente. Nenhum código implementado |
| 2026-10-09 | D03 / D05 / S05–S07 | Usuário escolheu registrar entradas repetidas sem tratamento | Registros independentes, sem deduplicação, atualização de métricas ou idempotência exigida inicialmente. Estimativa mantida; nenhum código implementado |
| 2026-10-09 | S01 / D04 / D05 | Usuário confirmou o objeto SeedCandidate proposto, compatibilidade via adapter e tratativa normal de erro de consulta | S01 concluída documentalmente; formato/metadados dirigidos permanecem em S02/S08. Erro propagado e complemento apenas após consulta bem-sucedida insuficiente. Nenhum código implementado |

| 2026-10-09 | Revisão documental | Consolidado escopo vigente e corrigidas referências antigas a fallback de erro, unicidade dos inputs e combinação de fontes; adapter dirigido continua pendente | S01 concluída documentalmente, S02–S16 pendentes; estimativa mantida. Documentos e links validados, sem mudança de código |
| 2026-10-10 | S02 / D04 | Usuário decidiu manter o separador `;` após análise do gerador de strings | Manter Join/Split via adapter, sem escape novo; ambiguidade para argumentos contendo `;` registrada. Versionamento e compatibilidade seguem pendentes; estimativa mantida, nenhum código implementado |

| 2026-10-10 | S02 / D04 | Usuário solicitou remover o campo de versão do SeedCandidate | Objeto atualizado em 001–003, sem requisito de versionamento ou filtro por versão. Tipos e compatibilidade seguem pendentes; nenhum código implementado |

| 2026-10-10 | S02 / D04 | Usuário escolheu reutilizar valores como estão | Inclui endereços, sem exclusão por dependência do ambiente nem remapeamento. Diferenças de estado/código/saldo são aceitas; verificação técnica dos tipos e tratamento de dados inválidos seguem pendentes. Nenhum código implementado |

| 2026-10-10 | S02 / D04 | Usuário decidiu manter o comportamento atual em falhas de desserialização | Propagar erro até o listener e encerrar o evento; sem descarte individual, correção, exclusão ou fallback novo. Verificações técnicas continuam pendentes; nenhum código implementado |

| 2026-10-10 | S02 / D04 / D05 | Usuário exigiu SeedCandidate totalmente compatível com seeds atuais e reutilizável entre contratos iguais | ExecutedInstructions incluído com tipo do DTO atual após análise dos três orderers; preservar dados de seleção, handlers e mutação. Validação funcional pendente; nenhum código implementado |

## 8. Controle de escopo

Sem migração das quatro entidades, instance_id obrigatório local, redesign dos
auxiliares, revisão global de jobs/callbacks, retomada, RLS ou histórico operacional
central. Integração pontual pode alterar seleção e ponto de publicação, mas deve
preservar comportamento operacional atual.

Outbox, entrega garantida, novas tabelas, observações múltiplas e rastreabilidade
detalhada precisam de decisão própria. Toda nova tabela exige confirmação prévia.
Reset continua pendente; nenhuma limpeza feita. Importação, replay, remapeamento,
coordenador, coletor global, relatório conjunto e campanha científica fora do escopo.

S01 concluída documentalmente; reavaliar custos ao detalhar D03/D05 e após Marco A. Se uma tarefa superar 8 pontos,
dividir por entrega verificável e registrar atualização das dependências.
