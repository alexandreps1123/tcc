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
estratégia, modo de seeds e orçamento. Repetições que utilizam um acervo
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
A participação no grupo não dispensa compatibilidade nem obriga o modo local a compartilhar.
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
Identidade técnica da instância responsável por seus registros operacionais,
separando tarefas, deployments e resultados no banco comum. Não limita por si só
o compartilhamento de seeds compatíveis entre instâncias.
_Evitar_: confundir com campanha, repetição ou identidade compartilhada da função.

**Contrato auxiliar**:
Contrato de apoio implantado no Geth local para exercitar comportamentos como
reentrância, exceções e consumo de gás. Cada inicialização cria registros com
UUID próprio, associados ao instance_id e a um papel único nessa instância.
_Evitar_: compartilhar seu endereço entre cadeias ou confundir seu papel com um ID global de registro.
