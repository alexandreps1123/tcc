# Dogefuzz — vocabulário experimental

Termos usados para organizar o benchmark e o compartilhamento de seeds entre
execuções. Os exemplos estão no documento de acompanhamento da implementação.

## Linguagem

**Campanha**:
Conjunto planejado de avaliações que compartilham um objetivo experimental,
contratos-alvo e protocolo de comparação.
_Evitar_: usar campanha para uma única tarefa ou repetição.

**Repetição**:
Uma realização de um caso experimental definido por contrato, cenário,
estratégia, versão do componente, condição inicial do acervo e orçamento. Repetições que utilizam um acervo
compartilhado acumulado podem influenciar-se e não pressupõem corpus inicial vazio.
_Evitar_: chamar cada worker de repetição.

**Worker**:
Unidade identificada que realiza o fuzzing com seu próprio ambiente blockchain
e produz resultados locais. O mesmo worker pode participar de diferentes
repetições, sem que elas se tornem a mesma execução.
_Evitar_: usar worker como sinônimo de tarefa, execução ou repetição.

**Tarefa de fuzzing**:
Trabalho solicitado ao Dogefuzz para avaliar um contrato com argumentos de
construção, duração, estratégia e detectores definidos.
_Evitar_: confundir com tarefa de implementação do backlog.

**Execução**:
Realização concreta de uma tarefa de fuzzing por um worker em uma repetição,
com contexto e resultados próprios. Executar uma transação é um teste dentro
da execução, não uma nova execução experimental.
_Evitar_: usar execução para designar uma campanha inteira ou uma transação.

**Grupo de corpus**:
Domínio de compartilhamento de seeds compatíveis que pode abranger diferentes
repetições, chamadas do benchmark e campanhas, em paralelo ou em sequência.
A participação no grupo não dispensa compatibilidade nem garante estado blockchain igual.
_Evitar_: vincular sua duração obrigatoriamente a uma repetição ou tratá-lo como fila de consumo exclusivo.

**Execução do benchmark**:
Uma chamada do benchmark que avalia o conjunto de contratos solicitado e gera
seu próprio relatório, contendo as execuções das tarefas de fuzzing desse conjunto.
_Evitar_: confundir com uma única execução de tarefa de fuzzing.

**Identidade compartilhada da função**:
Combinação da identidade estável do contrato com a assinatura ABI completa da
função, incluindo os tipos dos argumentos, que reconhece a mesma função entre execuções.
_Evitar_: confundir com o identificador local da função ou usar somente o número de parâmetros.

**Identidade estável do contrato**:
Identidade baseada no código-fonte exato e na declaração de contrato selecionada,
que reconhece o mesmo contrato entre deployments distintos.
_Evitar_: usar somente nome do arquivo, endereço de deployment ou identificador local do registro.

**Contexto de instância**:
Ambiente de uma execução, com tarefas, deployments, estado e resultados próprios.
O compartilhamento de entradas não transfere esse contexto para outra execução.
_Evitar_: confundir com campanha, repetição ou identidade compartilhada da função.

**Contrato auxiliar**:
Contrato de apoio implantado no Geth local para exercitar comportamentos como
reentrância, exceções e consumo de gás.
_Evitar_: presumir que seu endereço representa o mesmo contrato em outra cadeia ou confundir seu papel com um ID global de registro.

**Acervo compartilhado de seeds**:
Conjunto persistente de entradas reutilizáveis por execuções compatíveis, que
pode receber contribuições e ser consultado por várias execuções simultaneamente.
_Evitar_: confundir com histórico operacional completo ou fila de consumo exclusivo.

**Publicação de seed**:
Disponibilização de uma entrada e seus metadados no acervo para reuso por outras
execuções. Publicar uma entrada não transfere seus resultados ao consumidor.
_Evitar_: confundir com envio de transação ao Geth ou retomada da execução original.

**Origem da seed**:
Referência à execução e ao teste que produziram uma entrada compartilhada.
_Evitar_: tratar o ganho observado na origem como resultado obtido por outra execução.

**Referência do comparativo**:
Versão identificada do componente usada para comparar comportamento e resultados
com a implementação avaliada, sob condições experimentais registradas.
_Evitar_: confundir uma referência de código com uma execução ou com garantia de resultados equivalentes.
