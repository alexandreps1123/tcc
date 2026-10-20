# Custo e tarefas — bancos locais e acervo compartilhado

Criado em: 2026-10-06. Revisado em: 2026-10-09.

Status: reestimativa preliminar para a arquitetura escolhida em 2026-10-09.
Análise estática, sem implementação, medição de produtividade ou compromisso de prazo.

Referências: [arquitetura vigente](001-brainstorm-centralizacao-seeds.md) e
[checkpoints](003-acompanhamento-centralizacao-seeds.md).

## 1. Escopo e premissas da estimativa

Manter SQLite operacional, Geth, tarefas, UUIDs, auxiliares, callbacks e relatórios
locais. Acrescentar conexão e acervo compartilhado de seeds entre comandos
independentes em paralelo, sequência e chamadas futuras. O acervo permanece após
encerramento; não tornar histórico local durável automaticamente.

Preservar seleção/mutação/complemento atuais e integrar as três greybox.
Identidade básica: SHA-256 da fonte exata + nome do contrato + assinatura ABI completa.
Publicar a cada transação concluída e compatível, após análise e commit local
confirmado, sem esperar lote ou fim da tarefa. Frequência aprovada em 2026-10-09.
Publicação síncrona confirmada em 2026-10-09: aguardar sucesso, erro ou timeout
antes de continuar nesse caminho. Ponto de integração confirmado: chamada explícita
ao final do listener de análise, após salvar transação e tarefa com sucesso. Falha
local impede a publicação naquele evento; falha remota não desfaz gravações locais.
As gravações locais permanecem separadas, sem nova transação atômica entre elas.
Assíncrona é possibilidade futura, fora desta faixa, sujeita a decisão e reestimativa
após medir latência; não acrescentar fila ou publicador na entrega inicial.
Não exigir ganho positivo. Métricas externas não substituem avaliação local.

Consulta vigente revisada: no modo shared, RequestSeeds() consulta candidatos
apenas no acervo, antes da seleção. Não buscar histórico local como fonte ou
fallback. Leituras locais de contrato/função/ABI continuam necessárias. Modo local
mantém seleção atual para o comparativo. União local/shared e preferência por
métricas locais deixam de ser requisitos ativos. Acervo insuficiente após consulta
bem-sucedida usa seeds configuradas; erro de consulta é propagado conforme tratativa
atual, sem fallback novo. Uma entrada cuja publicação falhou permanece local, mas fica fora
da seleção shared. Publicações repetidas serão registradas separadamente, sem
deduplicação, substituição ou combinação de métricas; cada registro mantém sua avaliação.

S07 passa a tratar consulta exclusiva ao acervo e adaptação à seleção existente.
Estimativa mantida provisoriamente: 16 tarefas, 101 pontos, 98–176 horas; revisar
após decisões do acervo, sem presumir redução proporcional por retirar combinação.

Publicação simples confirmada pelo usuário em 2026-10-09: falha de publicação
é aceita, com resultado local preservado e continuidade do fuzzing, sem outbox
ou garantia de envio posterior. Timeout de publicação configurável confirmado,
com valor inicial de 500 ms para a tentativa inteira (conexão e gravação/confirmação).
Documentar junto ao parâmetro opções ilustrativas de 100 ms, 500 ms e 1 segundo,
com trade-offs de espera e tolerância a latência. Quantidade de tentativas também
configurável, inicialmente 1 no total (primeira tentativa incluída), sem retry
nesse valor. Valores maiores aumentam a espera e exigem critérios/intervalo de
repetição detalhados; nunca reenviar transação ao Geth. Timeout de consulta não
foi definido por essas escolhas.
A estimativa permanece inalterada, pois já considerava essa política.

Esta faixa considera as seguintes premissas; demais escolhas ainda aguardam confirmação:

- PostgreSQL apenas para o acervo, com acesso direto por componente interno.
- Estrutura mínima de seeds, registros independentes por publicação e metadados de seleção, sem referência de origem na entrega inicial.
- Publicação simples após commit local, sem garantia de entrega após falha ou encerramento.
- Sem outbox, recuperação persistente de publicação, replay ou histórico global de observações.
- Consultas limitadas, timeout e complemento de seeds configuradas no modo shared, sem consulta de candidatos locais.
- Valores reutilizados como estão, inclusive endereços; sem exclusão por dependência do ambiente ou remapeamento. Verificar os tipos já suportados.
- Novas tabelas precisam de confirmação prévia; a mudança arquitetural não aprova seu desenho.

Se for exigida publicação durável, esquema ampliado, suporte a novos tipos ou
compatibilidade complexa das variantes dirigidas, reestimar e dividir tarefas.
Cada tarefa permanece em 5 ou 8 pontos; uma entrega maior deve ser dividida.

## 2. Estimativa revisada

**16 tarefas, 101 pontos, média de 6,3 pontos; nenhuma acima de 8.**

**98–176 horas**, incluindo implementação e validação relacionadas às entregas.
Aproximadamente **13–22 dias de 8 horas** para pessoa familiarizada com Go/GORM,
Python e Docker. Pontos e horas são estimados separadamente. Com 20% de
contingência: **118–212 horas**, sem autorização para ampliar escopo.

| Frente | Tarefas | Pontos | Horas |
| --- | ---: | ---: | ---: |
| A. Contrato, formato e identidade (S01–S03) | 3 | 18 | 16–30 |
| B. Banco e integração de seeds (S04–S10) | 7 | 47 | 46–82 |
| C. Infraestrutura e benchmark (S11–S12) | 2 | 10 | 10–18 |
| D. Validação e operação (S13–S16) | 4 | 26 | 26–46 |
| **Total** | **16** | **101** | **98–176** |

A arquitetura anterior estimava 25 tarefas, 164 pontos e 158–284 horas.
A revisão reduz 9 tarefas, 63 pontos e 60–108 horas nas extremidades das faixas.
Isso é uma comparação de estimativas com premissas diferentes, não uma medição
de economia ou performance. As faixas não devem ser somadas.

## 3. Backlog vigente

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

Novos IDs S01–S16 evitam confundir tarefas reformuladas com as antigas.
A ordem da tabela é uma sequência válida; predecessoras no documento 003 indicam
onde há caminhos independentes. S01 é entrega documental concluída; S02–S16 estão pendentes. Verificação de código
se aplica às tarefas de implementação; S13/S14 acrescentam integração entre ambientes.

| ID | Tarefa e entrega verificável | Pontos | Horas |
| --- | --- | ---: | --- |
| S01 | **Contrato mínimo e fluxo:** Documentar fronteira local/acervo, pontos de integração e política de publicação; mapear a passagem para resultado concluído. | 5 | 4–8 |
| S02 | **Formato e compatibilidade dos inputs:** Manter separador `;` via adapter, sem campo de versão, e preservar a propagação atual de erros de desserialização; verificar round-trip, coleções e endereços, documentando a ambiguidade quando um argumento contém `;`. | 8 | 8–14 |
| S03 | **Identidade estável da função:** Calcular hash da fonte exata e assinatura ABI completa uma vez no cadastro; testar múltiplas declarações e sobrecargas sem substituir UUIDs locais. | 5 | 4–8 |
| S04 | **Conexão exclusiva do acervo:** Adicionar configuração e conexão independente; modo local não depende do acervo; definir pool, timeout e fechamento. | 5 | 6–10 |
| S05 | **Esquema e migração do acervo:** Após confirmação das novas tabelas, criar estrutura mínima de seeds, metadados de seleção, índices e registros independentes mesmo com inputs iguais, sem referência de origem; verificar migração reaplicável. | 8 | 8–14 |
| S06 | **Publicação de entradas concluídas:** Publicar após conclusão e commit local confirmado; não publicar incompletas; tolerar duplicatas e falhas conforme política escolhida, sem reenviar ao Geth. | 8 | 8–14 |
| S07 | **Consulta de candidatos do acervo:** Consultar exclusivamente o acervo por chave dentro de RequestSeeds(), com limite explícito; adaptar à seleção, preservar complemento configurado e validar custo. | 8 | 8–14 |
| S08 | **Integração das três greybox:** Adaptar candidatos à seleção existente de greybox, directed_greybox e other_directed_greybox; mutar e avaliar no Geth local sem adotar estado remoto. | 8 | 8–14 |
| S09 | **Falhas e tratativa atual:** Aplicar timeouts e propagar erro de consulta como no fluxo atual; complemento configurado apenas após consulta bem-sucedida com candidatos insuficientes; falha de publicação preserva resultado local. | 5 | 4–8 |
| S10 | **Métricas mínimas de compartilhamento:** Medir consultas, publicação, complemento configurado, erros e uso de candidatos do acervo e custo, sem identificar produtor ou registrar linhagem; preservar resultados locais. | 5 | 4–8 |
| S11 | **Infraestrutura persistente do acervo:** Permitir acesso por projetos Docker independentes, readiness e volume próprio; encerrar benchmark sem remover o acervo. | 5 | 6–10 |
| S12 | **Configuração e relatórios do benchmark:** Expor modo local/shared e acesso ao acervo, preservar comando, Geth e receptor por chamada; registrar modo e arquivos de relatório distintos. | 5 | 4–8 |
| S13 | **Validação de reuso entre execuções:** Demonstrar três greybox, duas execuções paralelas, reuso sequencial/futuro, mutação local, ausência de candidato e propagação atual de erros de desserialização. | 8 | 8–14 |
| S14 | **Validação de falhas e concorrência:** Exercitar indisponibilidade, duplicatas concorrentes, commit remoto ambíguo e encerramento; comprovar propagação de erro na consulta e preservação local em falha de publicação. | 8 | 8–14 |
| S15 | **Operação e limitações:** Documentar migração, configuração, preservação do corpus, falhas e dívida técnica; reset permanece pendente, sem limpeza automática. | 5 | 4–8 |
| S16 | **Comparativo curto de aceitação:** Executar casos selecionados de B2_parted em local/shared com orçamento e corpus inicial registrados; relatar funcionalidade e custos sem inferir superioridade científica. | 5 | 6–10 |

## 4. Marcos

- **A — Acervo preparado:** contrato mínimo, formato, identidade, conexão e esquema
  aprovados/implementados (S01–S05); banco local permanece operacional.
- **B — Integração funcional:** publicação, consulta, mutação local, complemento
  após sucesso insuficiente e tratativa de erros aprovada (S06–S09), com métricas (S10).
- **C — Chamadas independentes:** acesso ao acervo persistente e relatórios próprios
  configurados (S11–S12).
- **D — Aceite:** reuso paralelo/sequencial/futuro, concorrência e falhas validados,
  documentação e comparativo curto registrados (S13–S16).

## 5. Relação com o backlog anterior

| IDs antigos | Tratamento na arquitetura vigente |
| --- | --- |
| 01 | Reformulado em S01; D01 antigo preservado como histórico |
| 02–03 | Substituídos por conexão/esquema apenas do acervo (S04/S05) |
| 04–10 | Isolamento operacional central, auxiliares e SQL PostgreSQL das quatro entidades retirados do caminho obrigatório |
| 11–12 | Reaproveitados conceitualmente em S02/S03 |
| 13–14 | Reformulados em consulta e publicação entre dois bancos (S06/S07) |
| 15–16 | Reformulados em S08/S10 |
| 17–18, 21–22 | Reformulados em S09/S11/S12/S14 |
| 24–28 | Reformulados em S13–S16 |
| 19–20, 23 | Coordenador, coletor global e relatório conjunto continuam fora do escopo |

Tarefas retiradas não estão concluídas. O inventário completo, estimativas e
decisões antigos estão no [snapshot](docs/historico/2026-10-09-centralizacao-operacional/README.md).
O [plano de 88–144 horas](PLANO_ACERVO_COMPARTILHADO.md) também tinha outras premissas;
não representa a estimativa vigente.

## 6. Aceite, riscos e limites

Origem da seed é opção futura, excluída da implementação inicial por decisão do
usuário. Não criar campos para rastrear tarefa/transação/instância produtora ou
descendentes. Métricas podem identificar uso do acervo, mas não provar produtor
externo nem atribuir ganhos a ele. Metadados de seleção permanecem a definir.
Estimativas mantidas como preliminares; não presumir redução proporcional de horas
por retirar esse campo.

Aceite: duas instâncias mantêm bancos/Geths próprios, uma consulta entrada da
outra, aplica mutação e registra resultado local; reuso sequencial/futuro funciona;
entradas não concluídas não são publicadas; erros detectados na desserialização seguem propagação atual; consulta bem-sucedida vazia usa complemento
configurado; erro de consulta segue propagação atual; comandos e relatórios permanecem individuais.

Risco principal: publicação entre dois bancos. Sob a política aprovada de publicação
simples, um resultado local concluído pode não entrar no acervo após falha.
Registros repetidos são aceitos, inclusive se uma tentativa anterior gravou sem
confirmação recebida; não exigir idempotência nem deduplicação na entrega inicial.
Esse limite foi aceito em D03; timeout e número de tentativas estão definidos
documentalmente, assim como o ponto de integração. Detalhes de erro/intervalo com
mais de uma tentativa permanecem necessários antes de implementar S06. Repetidas
podem ocupar mais posições na seleção e aumentar armazenamento/custo de consulta;
medir esses efeitos. Estimativa mantida preliminar, sujeita a revisão após esquema.

Compatibilidade e ordenação das três greybox exigem validação: não importar
pontuações dependentes da tarefa como ganhos universais. A consulta ocorrerá dentro de RequestSeeds();
limites, timeout e custo do volume de publicações por transação precisam ser definidos
ou medidos antes do comparativo. Estimativas mantidas, sujeitas ao desenho da integração.

Dívidas atuais: limite ignorado na consulta local, ordenação crescente por
DeltaCoverage, divisão por zero/score não finito em seleção dirigida e descarte
de webhook de outra tarefa. Não corrigir silenciosamente apenas no modo shared.
Se alguma bloquear a entrega, criar tarefa separada de até 8 pontos.

Fora da faixa: centralização operacional, instance_id obrigatório nas quatro
entidades, redesign dos auxiliares, RLS, outbox durável, novos tipos Solidity,
replay, remapeamento de endereços, importação histórica, alta disponibilidade,
coordenador, coletor global, relatório conjunto e campanha científica completa.

Script de reset permanece pendente de definição/orçamento; nenhuma limpeza foi
executada. S01 concluída documentalmente; reavaliar a faixa ao detalhar D03/D05 e após o Marco A.
