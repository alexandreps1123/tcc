# Acompanhamento — bancos locais e acervo compartilhado

Criado em: 2026-10-06. Última revisão: 2026-10-20.

Arquitetura vigente: persistência operacional local e banco adicional de seeds.
Revisão documental autorizada pelo usuário. A preparação S02/S03 já possui entrega
em branch separada, ainda sem commit e com revisão pendente. Esta revisão não
altera código nem repete testes; evidências do agente são identificadas no relatório.

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
| Situação | S01 concluída documentalmente; S02 parcialmente implementada e S03 implementada no escopo delegado, ambas com revisão pendente; S04 pronta para implementação; S05–S16 pendentes |
| Próximo passo | Preparar execução de S04, revisar entrega S02/S03 e definir esquema/migração de S05 |
| Tarefas concluídas | 1 de 16 — S01, entrega documental |
| Pontos concluídos | 5 de 101 — S01, entrega documental |
| Estimativa | 98–176 horas; contingência de 20%: 118–212 horas |
| Responsável pelo acompanhamento | A definir |
| Evidência de implementação | Relatório S02/S03 e manifesto de dez arquivos conferidos; testes relatados pelo agente. PostgreSQL e cooperação entre containers ainda não implementados |

Os novos IDs S01–S16 substituem o backlog operacional centralizado. IDs antigos
não foram reutilizados nem marcados como concluídos.

## 2. Preparação

### Status das tarefas

**Pronta para implementação**: definições necessárias ao escopo autorizado estão
fechadas, critérios de aceite registrados e predecessoras necessárias atendidas.
O trabalho pode começar; implementação e validação técnica ainda não estão
concluídas. Uma validação que integra a própria entrega não impede este status.

**Pendente**: ainda há definição ou dependência necessária ao início do escopo.
**Implementada — revisão pendente**: código e evidências entregues; revisão e aceite
ainda pendentes. Indicar quando a implementação cobre somente parte da tarefa.
**Concluída**: entrega realizada e critérios de aceite demonstrados; quando a
entrega for exclusivamente documental, usar **Concluída documentalmente**.

Se a liberação for parcial, registrar o escopo autorizado e as dependências
restantes. Pronta não equivale a Em andamento; registrar responsável e início
quando a execução efetivamente começar. Novas dependências que impeçam o escopo
exigem reavaliar o status. Nenhum ponto concluído é somado pela prontidão.

S02 e S03 foram implementadas no
[escopo delegado](docs/tarefas/2026-10-10-s02-s03-preparacao-seeds/README.md),
com S02 parcial e revisão pendente. O
[relatório](docs/tarefas/2026-10-10-s02-s03-preparacao-seeds/RELATORIO.md)
registra os testes locais. O manifesto de dez arquivos foi conferido nesta revisão;
HEAD e main continuam em `1aa40f3e238f3ad330201cff8401074651d1c99f`, com alterações
não commitadas na branch `task/s02-s03-preparacao-seeds`. Não houve operação Git
que alterasse branches ou commits nesta revisão. Banco e cooperação entre containers
permanecem nas tarefas posteriores; main continua intacta até os testes necessários.

### Checkpoints de preparação

Ambiente local preparado em 2026-10-20: [guia de execução/debug e chamada de teste](docs/tarefas/2026-10-20-ambiente-local/README.md)
e [evidências](docs/tarefas/2026-10-20-ambiente-local/RELATORIO.md).
Testes S02/S03 passaram novamente; SimpleDAO/greybox/60s concluiu com relatório,
cobertura e hits. PostgreSQL 17 está disponível para desenvolvimento, ainda sem
integração ao código. Esta atividade não conclui S04, S11 ou os testes de reuso.

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

Estado: Parcialmente definida. Relacionada: S04, S05, S09, S11, S14, S15.

- [x] Confirmar PostgreSQL para o acervo compartilhado de seeds, sem migração do SQLite operacional (2026-10-10).
- [x] Definir PostgreSQL 17 como versão principal do acervo (2026-10-10).
- [ ] Definir configuração do PostgreSQL e fixar versão de correção/imagem exata na infraestrutura.
- [x] Configurar conexão própria por uma DSN em `seedCorpus.dsn`, sobrescrevível por `DOGEFUZZ_SEEDCORPUS_DSN`, seguindo o padrão atual (2026-10-10).
- [ ] Validar carregamento do novo campo em config.json/ambiente e preservar credenciais fora dos logs.
- [x] Definir timeout de inicialização configurável em `seedCorpus.startupTimeout`, inicialmente `5s`, sobrescrevível por `DOGEFUZZ_SEEDCORPUS_STARTUPTIMEOUT` (2026-10-10).
- [x] Definir pool configurável por instância em `seedCorpus.maxOpenConns` e `seedCorpus.maxIdleConns`, inicialmente 2/2, sobrescrevíveis por `DOGEFUZZ_SEEDCORPUS_MAXOPENCONNS` e `DOGEFUZZ_SEEDCORPUS_MAXIDLECONNS` (2026-10-10).
- [x] Definir timeout de consulta configurável em `seedCorpus.queryTimeout`, inicialmente `500ms`, sobrescrevível por `DOGEFUZZ_SEEDCORPUS_QUERYTIMEOUT` (2026-10-10).
- [ ] Definir aplicação única/reaplicável da migração e coordenação entre chamadas.
- [ ] Definir acesso de projetos Docker independentes e volume persistente do acervo.
- [x] Definir tratamento de operação: erro de consulta segue propagação atual; falha de publicação é aceita.
- [x] Na inicialização shared, fazer uma única tentativa de verificar a conexão ao acervo, limitada por timeout; em falha, registrar o erro e encerrar a inicialização com erro, sem retentativa ou fallback (2026-10-10).
- [x] Definir fechamento: responsável único, bloqueio de novas operações e cancelamento imediato das iniciadas, preservando acervo e SQLite local (2026-10-20).

Decisão de 2026-10-20: cancelamento imediato no encerramento. Um único componente é responsável pelo pool do acervo. Ao iniciar o encerramento, impedir novas operações, solicitar o cancelamento das consultas/publicações em andamento e fechar o pool, sem aguardar seus timeouts como política de conclusão. Não adicionar prazo de drenagem, fila ou retentativa de encerramento. Preservar os dados do PostgreSQL e o ciclo de vida atual do SQLite local. Cancelamento é uma solicitação ao driver, não garantia de retorno instantâneo nem de rollback: uma publicação pode já ter sido gravada. Aplicar a política existente de falha aceita, sem reenviar ao Geth ou garantir publicação posterior.

O timeout de inicialização limita a tentativa inteira de conexão, autenticação e
verificação de acesso, sem incluir migrações. Em erro anterior ao prazo, encerrar
imediatamente; não esperar os 5 segundos. Na implementação, validar duração positiva
e confirmar o carregamento por configuração/ambiente. O valor não altera o timeout
de publicação nem define o de consulta.

O pool limita cada instância a duas conexões totais; as livres fazem parte desse
limite. Manter tempos de vida/ociosidade nos padrões da biblioteca. Validar
`maxOpenConns > 0` e `0 <= maxIdleConns <= maxOpenConns` na implementação.
O tamanho 2/2 é inicial, sem garantia de suficiência: medir WaitCount/WaitDuration,
latências, timeouts e throughput e comparar com 4/4 se houver espera relevante.
Os controles e estatísticas são descritos na
[documentação de Go](https://go.dev/doc/database/manage-connections).

### D03 — Esquema do acervo e consistência de publicação

Estado: Em andamento documental. Relacionada: S01, S05, S06, S09, S14.

- [x] Substituir esquema operacional unificado por acervo separado de seeds.
- [x] Não adicionar instance_id às quatro entidades locais para isolamento central.
- [x] Exigir confirmação antes de criar qualquer nova tabela.
- [x] Aprovar uma única tabela de seeds para o acervo, com identidade, inputs, métricas, ID e horário de escrita no mesmo registro (2026-10-20).
- [x] Aprovar `seed_candidates` como nome da tabela única do acervo (2026-10-20).
- [x] Aprovar NOT NULL em todos os campos; ID e horário gerados pelo banco, sem defaults para métricas ou identidade (2026-10-20).
- [x] Aprovar nomes/tipos dos dez campos conforme tabela de D03, incluindo métricas NUMERIC(20,0) para uint64 (2026-10-20).
- [ ] Aprovar validações de conteúdo, incluindo CHECK de faixa das métricas, sem referência de origem.
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

Decisão de 2026-10-20: usar uma única tabela de seeds no acervo. Cada publicação
armazena a identidade compartilhada da função, inputs com separador `;`, métricas
do SeedCandidate, ID sequencial próprio e horário de escrita. Preservar publicações
repetidas como registros independentes, sem unicidade por inputs. A identidade se
repete entre registros; não criar tabelas separadas de contratos, funções, corpora,
observations ou outbox nesta entrega. Isso reduz relacionamentos e operações de
cadastro coordenadas entre instâncias, ao custo de repetir os dados de identidade.
Nome aprovado em 2026-10-20: `seed_candidates`. Todos os campos serão NOT NULL;
ID e horário de escrita são gerados pelo banco, sem defaults para métricas ou
identidade. Zero e texto vazio não são NULL; preservar zeros legítimos e a
representação atual das listas, sem fabricar métricas ausentes. NOT NULL não
valida identidade vazia nem detecta campos Go deixados em zero; os produtores
devem publicar somente após análise e gravações locais bem-sucedidas. Em violação
da restrição, a publicação falha conforme política já aprovada, preservando o
resultado local. Validações de conteúdo, índices e migração continuam pendentes. Tabelas
adicionais, inclusive de controle de migração caso necessárias, exigem confirmação
própria. A opção futura de origem não implica FK para banco remoto.

Tipos e nomes dos campos aprovados em 2026-10-20 para `seed_candidates`.
Todos os campos são NOT NULL; somente ID e horário são gerados pelo banco.

| Campo | Tipo PostgreSQL | Conteúdo |
| --- | --- | --- |
| `id` | `BIGINT GENERATED ALWAYS AS IDENTITY` | Chave primária sequencial do registro |
| `source_hash` | `TEXT` | SHA-256 hexadecimal da fonte exata |
| `contract_name` | `TEXT` | Declaração do contrato selecionada |
| `function_signature` | `TEXT` | Assinatura ABI canônica completa |
| `inputs` | `TEXT` | Argumentos na representação atual com `;` |
| `executed_instructions` | `TEXT` | Instruções executadas na representação atual com `;` |
| `delta_coverage` | `NUMERIC(20,0)` | Métrica uint64 de cobertura do produtor |
| `delta_min_distance` | `NUMERIC(20,0)` | Métrica uint64 de distância do produtor |
| `critical_instructions_hits` | `NUMERIC(20,0)` | Métrica uint64 de ocorrências críticas do produtor |
| `written_at` | `TIMESTAMPTZ` | Horário atribuído pelo PostgreSQL na inserção |

Validação técnica: MaxUint32 (4.294.967.295) cabe em BIGINT e NUMERIC(20,0),
mas não limita todas as métricas. ComputeDeltaMinDistance retornou 8.589.934.590
com dois alvos inalcançáveis e MaxUint64 (18.446.744.073.709.551.615) com instruções
vazias em chamada controlada. SELECT no PostgreSQL confirmou a representação
exata desses valores em NUMERIC(20,0). Não reduzir a faixa nem converter por
float64 na persistência; testar o round-trip pelo driver na implementação.
A restrição CHECK de faixa 0..MaxUint64 e demais validações de conteúdo ainda
precisam ser aprovadas. A aprovação dos tipos não cria tabelas ou migrações.
Referência técnica: [tipos numéricos do PostgreSQL 17](https://www.postgresql.org/docs/17/datatype-numeric.html).

Decisão do usuário: origem da seed fica como opção futura e não será utilizada
na implementação inicial. Não incluir no candidato/esquema ativo os IDs de tarefa,
transação ou instância produtora para rastreabilidade, nem registrar linhagem dos
descendentes. Uma referência futura poderia auxiliar diagnóstico/auditoria, mas
exige nova decisão. Métricas atuais podem indicar uso do acervo como fonte, sem
identificar produtor ou afirmar que todo candidato do acervo veio de outra instância.
Metadados de pontuação estão definidos conforme D04/D05, sem identificar origem
ou representar ganho do consumidor. Horário de inserção e ID sequencial estão
definidos para ordenação; nomes/tipos SQL aprovados. Índices e retenção ainda precisam de aprovação.

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
Timeout de consulta permanece separado, configurável e inicialmente 500 ms. Timeout pode ocorrer depois
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

Estado: Definições de formato e erros fechadas; preparação implementada, revisão
e validação no acervo pendentes. Relacionada à aprovação do esquema em D03.
Relacionada: S02, S06–S08, S13.

Objeto aprovado para publicação/consulta, a adaptar aos DTOs e à seleção
existentes. Preparação entregue em S02/S03; integração com acervo ainda pendente em S08:

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
ou mutadores. Preparação de tipos/adapter e identidade entregue em S02/S03,
com revisão pendente; integração remota ainda não implementada.

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
Preservar handlers, cálculos dos orderers e mutações; aplicar os limites e
desempates de recuperação explicitamente aprovados nesta proposta. Compatibilidade aqui significa
preservar a representação e os dados utilizados pelos algoritmos, não garantir
estado blockchain ou resultado de execução iguais. A entrega delegada implementou
objeto, adapter e identidade e relata testes de 74 casos de argumentos e dos três
orderers; ver [relatório](docs/tarefas/2026-10-10-s02-s03-preparacao-seeds/RELATORIO.md).
O manifesto do relatório corresponde aos dez arquivos atuais da branch
`task/s02-s03-preparacao-seeds`, ainda sem commit. Revisão independente pendente.
Persistência PostgreSQL, consultas/publicações e cooperação entre containers não
foram implementadas. A identidade é reutilizada em memória durante o processo,
sem retomada; constructor/fallback/receive e auxiliares continuam pendentes em D05.
Além do `;`, o mapper atual perde uma string vazia isolada; o agente reproduziu
ambos os limites sem alterar o comportamento. Não afirmar round-trip universal.
Critérios de recuperação estão definidos; esquema e integração seguem pendentes.

- [x] Exigir compatibilidade completa com o sistema atual de seeds, incluindo ExecutedInstructions para seleção dirigida, sem referência de origem.
- [x] Manter representação dos inputs com separador `;`, conforme estrutura atual, sem escape novo.
- [x] Remover o campo de versão do objeto, sem exigir versionamento nesta entrega.
- [x] Registrar verificação dos tipos suportados no escopo delegado: 74 casos, respeitando limites do mapper; revisão pendente.
- [x] Registrar verificação de listas/coleções e limites de `;` e string vazia isolada no relatório do agente.
- [x] Reutilizar valores como estão, inclusive endereços, sem exclusão por contexto ou remapeamento.
- [x] Registrar ausência de suporte adicional a tuplas e métodos especiais; identidade ABI de tuplas foi verificada separadamente.
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
- [x] Preservar cálculos dos orderers, mutação e complemento atual de candidatos insuficientes, com limites/desempates de recuperação aprovados.
- [x] Não impor filtro obrigatório de ganho positivo; somente concluídas podem ser candidatas.
- [x] Definir ponto e fonte: no comportamento compartilhado, consultar apenas o acervo em RequestSeeds(), antes da seleção; PostgreSQL obrigatório também em execução local.
- [ ] Definir filtros adicionais de cenário, construtor, formato e instrumentação.
- [ ] Definir fallback/receive e funções sem argumentos.
- [x] Incluir dados usados pelos três orderers no SeedCandidate, incluindo ExecutedInstructions.
- [x] Registrar testes dos três orderers com candidatos adaptados e mesmo contexto, conforme relatório; revisão pendente.
- [ ] Validar integração dirigida com PostgreSQL e contexto do contrato consumidor em S08.
- [x] Retirar combinação local/shared do escopo ativo; consulta de candidatos somente ao acervo no comportamento compartilhado.
- [x] Definir recuperação greybox por DeltaCoverage crescente até seedsSize e limite de 100 para cada estratégia dirigida (2026-10-10).
- [x] Recuperar os 100 candidatos mais recentes nos modos dirigidos, com ordem temporal decrescente (2026-10-10).
- [x] Definir recência dos modos dirigidos pelo momento da escrita no acervo, atribuído pelo PostgreSQL na inserção (2026-10-10).
- [x] Desempatar horários iguais dos modos dirigidos por ID numérico sequencial do registro, em ordem decrescente (2026-10-10).
- [x] Unificar desempate do greybox com os modos dirigidos: horário de escrita DESC, ID sequencial DESC após DeltaCoverage ASC (2026-10-10).
- [x] Aprovar nomes/tipos dos campos de armazenamento conforme D03 (2026-10-20).
- [x] Definir timeout total de recuperação configurável de 500ms; aplicar antes da seleção, conforme D02.
- [x] Retirar preferência por métricas locais: não há consulta local de candidatos no comportamento compartilhado.
- [x] Aceitar registros de entradas repetidas no acervo e na seleção, cada um com suas métricas; histórico local permanece atual.
- [ ] Definir tratamento de tipos/funções não suportados conforme D04; não introduzir deduplicação.
- [x] Registrar cache em memória do serviço de contratos na entrega S03, sem migrar entidades locais; revisão pendente.
- [ ] Confirmar escopo dos métodos especiais/auxiliares e necessidade de identidade além da vida do processo, preservando ausência de retomada.

Exemplo: withdraw(uint256) de SimpleDAO é reconhecida entre deployments pela
mesma fonte, declaração e assinatura. Isso não afirma estado blockchain igual.
Score/DeltaCoverage de A não é ganho de B; B muta e avalia localmente.

Orientação vigente revisada: no comportamento compartilhado, RequestSeeds() consulta candidatos
somente no acervo compartilhado, antes da seleção. Não consultar histórico local
como segunda fonte ou fallback. Leituras locais de contrato/função/ABI continuam
necessárias e não representam consulta de candidatos. Não existe modo local
selecionável; comparar com referência histórica em protocolo ainda a definir.
Não exigir cache periódico na entrega inicial.

União local/shared e preferência por métricas locais foram substituídas; não são
requisitos ativos. Publicação que falhar mantém resultado local, mas aquela entrada
não participa da seleção shared enquanto não estiver no acervo. Se faltarem
candidatos após consulta bem-sucedida, usar complemento atual de seeds configuradas.
Em erro, propagar como atualmente, sem fallback novo ou consulta de candidatos locais.
RequestSeeds() retorna o erro, GenerateInput() o propaga e o listener de fuzzing
registra a falha e encerra aquele evento. Isso não implica encerrar todo o processo.
Timeout de consulta definido em `seedCorpus.queryTimeout`, inicialmente `500ms`,
sobrescrevível por `DOGEFUZZ_SEEDCORPUS_QUERYTIMEOUT`. Cobre espera pelo pool,
eventual conexão, consulta e leitura dos resultados. Ordenação dos orderers,
desserialização e mutação ficam fora do prazo; validar duração positiva na
implementação. Em timeout, propagar erro sem fallback/complemento novo, como já
definido. Publicação mantém sua política distinta.

Decisão vigente: registrar publicações repetidas sem tratamento especial. Cada
publicação aceita cria um registro com seus inputs/métricas; não substituir primeira
por última ou melhor, não combinar campos nem deduplicar na seleção. Registros
iguais podem ocupar várias posições, como ocorre atualmente no histórico local.
Não exigir idempotência; uma repetição após timeout com commit remoto pode criar
outra linha. Medir crescimento e efeito no conjunto de candidatos. Representação dos inputs, limites, desempates e nomes/tipos dos campos de armazenamento estão definidos; validações, índices e migração continuam pendentes. Consulta remota ainda não implementada; objeto/adapter já entregues no escopo delegado.


Decisão vigente de D05/S07 (2026-10-10): filtrar pela identidade compartilhada
(fonte exata, declaração e assinatura ABI) e personalizar a consulta por estratégia.

| Estratégia | Ordem da recuperação no acervo | Limite | Seleção no Dogefuzz |
| --- | --- | --- | --- |
| greybox | DeltaCoverage ASC, horário de escrita DESC, ID sequencial DESC | seedsSize, atualmente 5 | Escolha aleatória na lista retornada; ordenação atual pode ser reaplicada sem alterar sua ordem |
| directed_greybox | Horário de escrita DESC, ID sequencial DESC | 100 | Orderer dirigido atual; manter até seedsSize antes da escolha aleatória |
| other_directed_greybox | Horário de escrita DESC, ID sequencial DESC | 100 | Orderer dirigido atual; manter até seedsSize antes da escolha aleatória |

O horário é atribuído pelo PostgreSQL na inserção, não pelo produtor. ID numérico
sequencial e horário pertencem ao registro persistido, sem integrar SeedCandidate.
Os campos `written_at` (TIMESTAMPTZ) e `id` (BIGINT identity) estão aprovados no esquema. Publicações repetidas
recebem novos registros/horários. Horário e ID não garantem ordem de commit.

A consulta greybox mantém o critério primário crescente atual, mas substitui a
preferência por timestamps locais antigos, nos empates, pela escrita mais recente.
Os dirigidos avaliam apenas os 100 mais recentes; não há garantia de selecionar
os mesmos candidatos que ordenar todo o histórico. A compatibilidade do objeto,
handlers e cálculos permanece exigida dentro do conjunto recuperado. Ordenação
SQL está dentro do timeout de consulta; os orderers em Go ficam fora dele.

Consulta/desserialização bem-sucedida insuficiente usa complemento configurado;
erro propaga sem novo fallback. A nova implementação tem somente comportamento
compartilhado, inclusive em execução local. O código anterior em main é a
referência separada do comparativo, sem seletor local/shared na nova versão.

### D06 — Benchmark e falhas

Estado: Parcialmente definida. Relacionada: S04, S09, S11–S14.

- [x] Definir somente comportamento compartilhado: exigir PostgreSQL e DSN também na execução local, sem parâmetro de modo local/shared (2026-10-10).
- [x] Separar versões por branch: main intacta como referência; implementar e testar acervo em branch separada, sem merge/alteração de main antes dos testes necessários (2026-10-10).
- [ ] Definir protocolo do comparativo com referência histórica anterior ao acervo obrigatório.
- [ ] Detalhar testes necessários e evidências para avaliar integração futura em main; registrar commits e configurações das versões.
- [ ] Propagar somente configuração necessária ao acesso ao acervo.
- [ ] Definir infraestrutura independente e persistente; não usar down do benchmark para removê-la.
- [x] Distinguir consulta com erro (propagar) de sucesso insuficiente (complementar); publicação falha aceita.
- [x] Inicialização shared falha imediatamente em erro de conexão, conforme D02.
- [x] Definir timeout de inicialização configurável de 5s, conforme D02.
- [x] Definir timeout de consulta configurável de 500ms, conforme D02/D05.
- [x] Definir encerramento com publicação em curso: cancelar imediatamente, aceitando publicação perdida ou commit remoto já realizado, sem drenagem/retentativa (2026-10-20).
- [ ] Verificar receptor/relatório próprios e destinos de arquivos concorrentes.
- [ ] Documentar efeito real de reinício e perda possível do histórico local.

Decisão vigente: compartilhamento obrigatório de seeds, inclusive fora do benchmark.
Exigir DSN e PostgreSQL disponível na inicialização. Não implementar parâmetro
local/shared ou funcionamento sem acervo. Substitui a decisão anterior de modo
local sem conexão/consulta/publicação. SQLite, Geth e relatórios continuam próprios
por instância; não centralizar as quatro entidades operacionais. A comparação com
a versão anterior deve ter referência de código/protocolo explícita, ainda pendente.

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

Referência para retomada: [divergência na seleção greybox](004-divergencia-selecao-greybox.md).
Investigação adiada; não autoriza alterar o algoritmo no escopo do compartilhamento.

- [x] Registrar limite local ignorado, DeltaCoverage crescente e score dirigido não finito na análise e no relatório S02/S03; investigação/correção separadas.
- [ ] Verificar se algum problema impede o novo fluxo.
- [ ] Se necessário, abrir correção separada de até 8 pontos e definir aplicação comparável na referência histórica.
- [ ] Registrar limitação do coletor que descarta webhook de outra tarefa.
- [ ] Preservar os cálculos atuais e aplicar somente alterações de recuperação/limites/desempates já aprovadas; correções adicionais exigem decisão própria.

## 4. Tarefas e dependências

A ordem abaixo é válida para execução. Predecessoras permitem antecipar caminhos
independentes. S01 está concluída como entrega documental. S02/S03 têm entrega
no escopo delegado, com S02 parcial e revisão pendente; S04 está pronta para implementação e S05–S16 permanecem pendentes.

| ID | Tarefa | Pontos | Horas | Predecessoras | Definições | Status |
| --- | --- | ---: | --- | --- | --- | --- |
| S01 | Contrato mínimo e fluxo | 5 | 4–8 | — | D01, D03 | Concluída documentalmente |
| S02 | Formato e compatibilidade dos inputs | 8 | 8–14 | S01 | D04, D05 | Parcialmente implementada — revisão pendente |
| S03 | Identidade estável da função | 5 | 4–8 | S01 | D05 | Implementada no escopo delegado — revisão pendente |
| S04 | Conexão exclusiva do acervo | 5 | 6–10 | S01 | D02 | Pronta para implementação |
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
dependeu de implementar o banco. D03/D04/D05 continuam parciais para tarefas seguintes.
A tabela única e seus campos/tipos estão aprovados; não iniciar S05 sem fechar
validações, índices e migração, além de atender suas predecessoras.

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
- [x] Definir ponto de consulta: RequestSeeds() busca candidatos somente no acervo no comportamento compartilhado.
- [x] Registrar substituição de união local/shared e preferência por métricas locais; consulta somente ao acervo no comportamento compartilhado.
- [x] Definir repetidas: registrar sem tratamento, preservando cada publicação e suas métricas.
- [x] Registrar decisões e pendências antes de iniciar esquema/integração.

Evidência / próximo passo: contrato documental consolidado em 001–003 e código atual conferido em RequestSeeds(), GenerateInput() e listeners de análise/fuzzing. SeedCandidate aprovado; erro de consulta propagado sem fallback novo. Próximo: revisar entrega S02/S03 e fechar esquema/conexão; integração remota continua pendente em D03/D05/S08.

### S02 — Formato e compatibilidade dos inputs

Status: Parcialmente implementada — revisão pendente.
Responsável: agente da tarefa delegada | Entrega: 2026-10-10 | Horas reais: não medidas.

- [x] Confirmar separador `;`, sem campo de versão.
- [x] Reutilizar argumentos como estão, inclusive endereços.
- [x] Manter tratamento atual de erro de desserialização, sem descarte ou fallback novo.
- [x] Implementar SeedCandidate e adapter bidirecional, preservando listas e métricas.
- [x] Registrar testes de 74 casos de tipos/coleções e equivalência dos três orderers, conforme relatório.
- [x] Reproduzir e documentar limites: `;` em argumentos e string vazia isolada.
- [ ] Revisar independentemente código e evidências da entrega.
- [ ] Verificar representação no esquema PostgreSQL aprovado, sem alterar o baseline inadvertidamente.

Evidência: [relatório da entrega](docs/tarefas/2026-10-10-s02-s03-preparacao-seeds/RELATORIO.md),
com manifesto conferido em dez arquivos. Testes foram executados pelo agente;
não repetidos nesta revisão documental. Identidade e adapter não demonstram ainda
consulta/publicação ou compartilhamento entre containers. Métodos especiais e
novos tipos não foram acrescentados. Limites existentes não foram corrigidos.

### S03 — Identidade estável da função

Status: Implementada no escopo delegado — revisão pendente.
Responsável: agente da tarefa delegada | Entrega: 2026-10-10 | Horas reais: não medidas.

- [x] Implementar hash da fonte exata antes da compilação, declaração selecionada e assinatura ABI canônica.
- [x] Registrar testes de fontes, declarações, sobrecargas e independência de UUID/endereço.
- [x] Calcular no cadastro e reutilizar em memória no serviço de contratos, sem alterações de esquema.
- [ ] Revisar independentemente a integração e os testes.

Evidência: [relatório da entrega](docs/tarefas/2026-10-10-s02-s03-preparacao-seeds/RELATORIO.md).
Cache em memória durante o processo; sem reconstrução de registros antigos ou
retomada após reinício. Constructor/fallback/receive e auxiliares não integram o
lookup compartilhável entregue; políticas pendentes em D05. A existência de um
hash não comprova cooperação entre containers, reservada a S13/S14.

### S04 — Conexão exclusiva do acervo

Status: Pronta para implementação.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Configurar conexão própria do acervo, independente de SQLite e obrigatória também na execução local.
- [ ] Verificar que execução local exige DSN/PostgreSQL e valida acesso na inicialização, sem seletor local/shared; consulta/publicação reais pertencem a S06/S07.
- [ ] Implementar startupTimeout configurável de 5s para verificar conexão na inicialização com uma tentativa e encerramento em falha, conforme D02.
- [ ] Implementar pool configurável 2/2, validar os parâmetros e reutilizar o pool da instância.
- [ ] Implementar queryTimeout configurável de 500ms e validar o prazo total da operação.
- [ ] Implementar responsável único pelo pool; bloquear novas operações e solicitar cancelamento imediato das iniciadas antes de fechar, sem drenagem ou retentativa.
- [ ] Verificar cancelamento, recusa de novas operações e fechamento sem apagar acervo ou comprometer conexão local; liberar o pool também em falha posterior da inicialização.

Escopo liberado em 2026-10-20: configuração, conexão, pool, timeouts e encerramento.
S01 atende a predecessora; revisão S02/S03 pode ocorrer em paralelo. Esquema e
migração ficam em S05; infraestrutura definitiva em S11. Validar a conexão com
PostgreSQL 17 sem criar tabelas. Não incluir consultas/publicações reais nesta entrega.
A política será exercitada com operações controladas em S04 e com os fluxos reais
em S06/S07/S14. A prontidão não inicia a implementação nem soma pontos concluídos.

Evidência / bloqueio / próximo passo: decisão de encerramento registrada em D02/D06; preparar escopo fechado da execução e registrar responsável, branch e evidências antes das alterações.

### S05 — Esquema e migração do acervo

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [x] Aprovar estrutura com uma única tabela de seeds, sem tabelas próprias de contratos/funções (2026-10-20).
- [x] Aprovar nome da tabela: `seed_candidates` (2026-10-20).
- [x] Aprovar obrigatoriedade: NOT NULL em todos os campos, ID/horário gerados pelo banco e métricas/identidade sem defaults (2026-10-20).
- [x] Aprovar nomes/tipos dos dez campos de seed_candidates, conforme tabela de D03 (2026-10-20).
- [ ] Aprovar validações de conteúdo e índices; confirmar separadamente eventual tabela de controle de migração.
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

- [ ] Implementar consulta greybox por DeltaCoverage ASC, horário de escrita DESC e ID DESC até seedsSize, validando desempates.
- [ ] Implementar recuperação dirigida dos 100 candidatos mais recentes, seguida dos orderers atuais; validar campo temporal e desempates.
- [ ] Aplicar queryTimeout de 500ms e verificar limite antes da seleção.
- [ ] Consultar apenas o acervo no comportamento compartilhado, adaptar à seleção e complementar com seeds configuradas quando necessário; sem deduplicar repetidas.
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

- [ ] Expor somente configuração de acesso obrigatório ao acervo; preservar fluxo e IDs do benchmark.
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
- [ ] Exercitar encerramento com cancelamento imediato e resposta perdida após commit, respeitando falha aceita e sem garantia de rollback remoto.
- [ ] Registrar latência/carga, propagação de erro na consulta e preservação local em falha de publicação; repetidas aceitas, sem reenvio ao Geth.

Evidência / bloqueio / próximo passo: —

### S15 — Operação e limitações

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Documentar operação, migração, acesso obrigatório e preservação do acervo.
- [ ] Documentar perda possível de publicação/histórico local e dívida técnica.
- [ ] Conferir roteiro; manter reset fora da entrega enquanto não definido.

Evidência / bloqueio / próximo passo: —

### S16 — Comparativo curto de aceitação

Status: Pendente.
Responsável: — | Início: — | Término: — | Horas reais: —.

- [ ] Selecionar contratos/cenários de B2_parted e recuperar configuração anterior quando disponível.
- [ ] Executar versão com acervo obrigatório e referência histórica após definição do protocolo, com orçamento, recursos e condição inicial registrados.
- [ ] Comparar branch do acervo com main preservada, registrando commits/configurações; manter main intacta durante os testes.
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
| 2026-10-10 | Status das tarefas | Usuário solicitou status para tarefas prontas para implementação | Criado Pronta para implementação e aplicado a S02/S03 no escopo delegado. Validação técnica continua pendente; pontos concluídos e estimativas preservados |
| 2026-10-10 | D02 | Usuário confirmou PostgreSQL para o banco compartilhado de seeds | Tecnologia definida; versão, conexão, configuração e infraestrutura pendentes. S04 permanece pendente; estimativa mantida, referências congeladas preservadas |
| 2026-10-10 | D02 | Usuário definiu PostgreSQL 17 | Versão principal confirmada; conexão, configuração e startup pendentes. Versão de correção/imagem exata será fixada na infraestrutura; S04 permanece pendente, referências congeladas preservadas |
| 2026-10-10 | D02 / S04 | Usuário escolheu a opção mais simples para configurar a conexão: uma DSN | Usar seedCorpus.dsn e DOGEFUZZ_SEEDCORPUS_DSN no mecanismo atual; pool, timeout e startup pendentes. Estimativa mantida; nenhum código implementado, referências congeladas preservadas |
| 2026-10-10 | D02 / D06 / S04 | Usuário escolheu falhar imediatamente se o acervo estiver indisponível na inicialização | Verificar conexão uma vez, com timeout; registrar falha e encerrar com erro, sem retentativa ou fallback. Valor do timeout pendente; S04 permanece pendente, nenhum código implementado |
| 2026-10-10 | D02 / D06 / S04 | Usuário definiu timeout de inicialização configurável, inicialmente 5 segundos | Usar seedCorpus.startupTimeout e DOGEFUZZ_SEEDCORPUS_STARTUPTIMEOUT para conexão/autenticação/verificação, sem migrações. Pool e timeout de consulta pendentes; estimativa mantida, nenhum código implementado |
| 2026-10-10 | D02 / S04 | Usuário aprovou pool configurável 2/2 por instância | Definidos maxOpenConns/maxIdleConns e variáveis de ambiente; medir suficiência, comparar com 4/4 se houver espera relevante. Timeout de consulta e fechamento pendentes; estimativa mantida, referências congeladas preservadas |
| 2026-10-10 | D02 / D05 / D06 / S04 | Usuário definiu timeout de consulta configurável, inicialmente 500 ms | Usar seedCorpus.queryTimeout e DOGEFUZZ_SEEDCORPUS_QUERYTIMEOUT; cobrir espera pelo pool, eventual conexão e recuperação dos resultados. Timeout propaga erro; limites de candidatos e fechamento pendentes. Nenhum código implementado |
| 2026-10-10 | D05 / S07 / S08 | Usuário definiu consulta específica greybox e limite de 100 para os demais modos | Greybox até seedsSize por DeltaCoverage crescente; directed_greybox e other_directed_greybox até 100, com seleção atual no Dogefuzz. Critério dos 100 e desempates pendentes; restrição do conjunto documentada, nenhum código implementado |
| 2026-10-10 | D05 / S07 | Usuário escolheu os mais recentes para os modos dirigidos | Recuperar até 100 em ordem temporal decrescente, depois aplicar os orderers atuais. Campo temporal e desempates pendentes; consulta greybox preservada, nenhum código implementado |
| 2026-10-10 | D03 / D05 / S07 | Usuário definiu recência pelo momento da escrita na base compartilhada | PostgreSQL atribui horário de inserção ao registro; sem timestamp produtor ou mudança no SeedCandidate. Publicações repetidas são recentes novamente; campo temporal do greybox e desempates pendentes. Nenhum código implementado |
| 2026-10-10 | D03 / D05 / S07 | Usuário escolheu desempate por ID após proposta de ID sequencial | Modos dirigidos: horário de escrita DESC, ID sequencial DESC, até 100; ID apenas no registro persistido. Esquema e desempate greybox pendentes; nenhum código implementado |
| 2026-10-10 | D05 / S07 | Usuário definiu para o greybox o mesmo critério temporal/ID dos modos dirigidos | Greybox: DeltaCoverage ASC, escrita DESC, ID sequencial DESC, até seedsSize. Dirigidos: escrita DESC, ID DESC, até 100. Substitui preferência temporal crescente nos empates do greybox; esquema ainda pendente, nenhum código implementado |
| 2026-10-10 | D06 / S04 / S12 | Usuário definiu modo local sem consultar nem publicar no acervo | Sem conexão ao PostgreSQL no modo local; DSN e disponibilidade do acervo exigidas somente em shared. Parâmetro de modo e valor padrão pendentes; nenhum código implementado |
| 2026-10-10 | D06 / S04 / S12 / S16 | Usuário definiu somente shared, exigindo PostgreSQL também na execução local | Substitui modo local sem acesso ao acervo; sem parâmetro de modo. DSN e verificação na inicialização obrigatórias; SQLite operacional local preservado. Comparativo com referência histórica requer protocolo próprio; nenhuma alteração de código nesta revisão |
| 2026-10-10 | D06 / S16 / Delegação | Usuário definiu separação por branch e main intacta até os testes necessários | Desenvolver/testar em branch própria; main é referência anterior, sem alteração ou merge nessa etapa. Detalhar testes/protocolo e registrar commits; referências congeladas preservadas, nenhuma operação Git realizada nesta revisão |
| 2026-10-10 | Revisão consolidada | Consolidado estado vigente após decisões e entrega delegada S02/S03 | PostgreSQL obrigatório, configuração/consultas/desempates e branches consolidados; S02 parcial e S03 implementada no escopo delegado com revisão pendente, sem aumentar pontos concluídos. Manifestos conferidos; testes não repetidos nesta revisão |
| 2026-10-20 | D02 / D06 / S04 / S14 | Usuário escolheu cancelamento imediato no encerramento | Responsável único pelo pool, recusa de novas operações e cancelamento das iniciadas antes de fechar, sem apagar dados ou garantir rollback. Substitui aguardar os timeouts. S04 pronta para implementação, 5 pontos/6–10 horas mantidos; implementação não iniciada e pontos concluídos inalterados |
| 2026-10-20 | D03 / S05 | Usuário aprovou uma única tabela de seeds | Identidade, inputs, métricas, ID sequencial e horário de escrita no mesmo registro; sem tabelas próprias de contratos/funções. Menos relacionamentos e cadastros coordenados, com repetição da identidade. Nome, campos/tipos, índices e migração pendentes; S05 permanece pendente, 8 pontos/8–14 horas mantidos; nenhuma tabela criada |
| 2026-10-20 | D03 / S05 | Usuário aprovou o nome `seed_candidates` | Nome alinhado ao objeto SeedCandidate; campos/tipos, obrigatoriedade, índices e migração pendentes. S05 permanece pendente; nenhuma tabela criada |
| 2026-10-20 | D03 / S05 | Usuário aprovou NOT NULL em todos os campos | ID/horário gerados pelo banco; métricas e identidade sem defaults. Zero e texto vazio não são NULL; validar conteúdo e não fabricar métricas. Violação segue falha aceita de publicação; nomes/tipos SQL, índices e migração pendentes. S05 continua pendente; nenhuma tabela criada |
| 2026-10-20 | D03 / D05 / S05 | Usuário aprovou os tipos SQL propostos | Dez campos registrados, com TEXT, BIGINT identity, TIMESTAMPTZ e NUMERIC(20,0) para métricas uint64. Validação confirmou valores acima de MaxUint32; CHECK de faixa, demais validações, índices e migração pendentes. S05 continua pendente; estimativa mantida e nenhuma tabela criada |

## 8. Controle de escopo

Sem migração das quatro entidades, instance_id obrigatório local, redesign dos
auxiliares, revisão global de jobs/callbacks, retomada, RLS ou histórico operacional
central. Integração pontual pode alterar seleção e ponto de publicação, mas deve
preservar comportamento operacional atual.

Uma única tabela de seeds está aprovada, com desenho SQL ainda pendente.
Outbox, entrega garantida, tabelas adicionais, observações múltiplas e rastreabilidade
detalhada precisam de decisão própria. Toda tabela adicional exige confirmação prévia.
Reset continua pendente; nenhuma limpeza feita. Importação, replay, remapeamento,
coordenador, coletor global, relatório conjunto e campanha científica fora do escopo.

S01 concluída documentalmente; reavaliar custos ao detalhar D03/D05 e após Marco A. Se uma tarefa superar 8 pontos,
dividir por entrega verificável e registrar atualização das dependências.
