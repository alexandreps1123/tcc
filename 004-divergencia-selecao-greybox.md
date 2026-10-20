# Divergência na seleção greybox — referência para investigação futura

Registrado em: 2026-10-10.
Status: análise estática registrada; investigação e alteração de comportamento adiadas.
Escopo: seleção de seeds do modo `greybox` atual do Dogefuzz.

## Questão registrada

O Dogefuzz ordena candidatos por `DeltaCoverage` crescente e mantém somente os
primeiros `seedsSize` antes da escolha aleatória. Assim, privilegia menores ganhos
históricos de cobertura na composição da lista elegível para mutação.

Esse comportamento difere da formação de corpus apresentada no Fuzzing Book e
não possui, nos trechos de código examinados, justificativa para preferir menores
ganhos. Registrar como divergência a investigar, sem concluir que todo greybox
deva ordenar por ganho decrescente ou que inverter o comparador seja suficiente.
A intenção original e o impacto experimental ainda não foram determinados.

## Referência teórica e limite da comparação

O capítulo descreve o uso de feedback de cobertura para incorporar entradas ao
corpus. No exemplo `GreyboxFuzzer.run()`, uma entrada é incorporada quando seu
conjunto de posições cobertas ainda não foi observado. O critério é novidade do
conjunto, não necessariamente aumento da união acumulada. Uma combinação nova
de posições já conhecidas também pode ser incorporada.
[Greybox Mutation-based Fuzzing](https://www.fuzzingbook.org/html/GreyboxFuzzer.html#Greybox-Mutation-based-Fuzzing).

O `PowerSchedule` básico dá energia igual às seeds e faz escolha uniforme na
população. A discussão geral apresenta energia como mecanismo para distribuir
esforço entre candidatos promissores; estratégias avançadas podem usar outros
critérios. O livro não estabelece uma regra universal de ordenar seeds por
`DeltaCoverage` decrescente.
[Seeds and Power Schedules](https://www.fuzzingbook.org/html/GreyboxFuzzer.html#Seeds-and-Power-Schedules).

Portanto, a escolha uniforme em si não é a divergência principal. A questão está
em quais candidatos chegam à escolha e no uso de um ganho histórico crescente
para limitar essa população.

## Evidências no Dogefuzz

| Etapa | Implementação observada | Referência |
| --- | --- | --- |
| Consulta | Recupera transações concluídas da função local, ordenadas por timestamp crescente | [Repositório](dogefuzz/data/repo/transaction.go) |
| Limite de recuperação | RequestSeeds passa `2 * seedsSize`, mas o repositório não aplica esse limite | [PowerSchedule](dogefuzz/fuzz/power_schedule.go) e [repositório](dogefuzz/data/repo/transaction.go) |
| Métrica | DeltaCoverage é a diferença entre cobertura acumulada após incluir a transação e cobertura acumulada anterior da tarefa | [Cálculo](dogefuzz/pkg/coverage/compute.go) |
| Ordenação greybox | Comparador `transactions[i].DeltaCoverage < transactions[j].DeltaCoverage` | [Coverage orderer](dogefuzz/fuzz/coverage_orderer.go) |
| Lista final | Mantém os primeiros seedsSize candidatos e completa com seeds configuradas quando necessário, após desserialização bem-sucedida | [PowerSchedule](dogefuzz/fuzz/power_schedule.go) |
| Escolha | RandomChoice na lista final, sem ponderação pelo ganho | [Greybox fuzzer](dogefuzz/fuzz/greybox.go) |

O valor de `seedsSize` em [config.json](dogefuzz/config.json) é 5 na configuração
examinada. A ordenação é estável: empates preservam a ordem recebida da consulta.
O ganho registrado não é percentual nem garantia de progresso ao reutilizar a seed.

## Exemplo do efeito

Considere sete registros, na ordem de timestamp, com os seguintes ganhos:

| Registro | DeltaCoverage | Elegível após corte de cinco |
| --- | ---: | --- |
| A | 0 | Sim |
| B | 0 | Sim |
| C | 1 | Sim |
| D | 2 | Sim |
| E | 3 | Sim |
| F | 8 | Não |
| G | 15 | Não |

A escolha aleatória ocorre entre A–E; F e G não participam daquela chamada.
Com cinco registros de ganho zero, todos os registros de ganho positivo podem
ficar fora da lista. Em empates de ganho, os mais antigos tendem a permanecer
primeiro devido à ordem temporal e à ordenação estável.

Esses efeitos decorrem do código; sua influência sobre cobertura, vulnerabilidades
e throughput é hipótese a medir. Não foram executados experimentos para comprová-la.

## Relação com o compartilhamento de seeds

O acervo compartilhado pode acumular candidatos de ganho zero e registros
repetidos, pois essas publicações foram aceitas no desenho vigente. Aplicar a
mesma seleção a esse conjunto pode aumentar sua concentração em poucas entradas.
Isso é uma hipótese, não um resultado observado.

Limitar a recuperação no banco também altera o universo ordenado: uma seed
excluída pela consulta não chega à seleção. Esse efeito é distinto de mudar o
comparador ou a política de admissão ao corpus.

A investigação da política de admissão e do comparador crescente continua adiada.
Decisões posteriores aprovadas no escopo do acervo: greybox recupera até seedsSize
por DeltaCoverage ASC, escrita DESC e ID DESC; modos dirigidos recuperam os 100
mais recentes por escrita DESC e ID DESC antes dos orderers atuais. O desempate
temporal do greybox muda em relação ao histórico local, e o limite dirigido
restringe o universo avaliado; essas mudanças devem ser registradas no comparativo.
Não se trata de correção da divergência teórica documentada aqui.

A nova versão tem somente comportamento compartilhado, inclusive em execução
local. Comparar a branch de implementação com main preservada, sem parâmetro de
modo local/shared. Mudanças adicionais de seleção exigem decisão própria e
condições comparáveis nas duas referências de código.

## Pontos para retomar

- [ ] Recuperar a justificativa original da ordenação crescente e do corte por seedsSize, incluindo histórico do código e estudo anterior.
- [ ] Separar políticas de admissão, recuperação, ordenação e escolha probabilística.
- [ ] Verificar como os ganhos históricos dependem do acumulado da tarefa produtora e como isso afeta comparações entre execuções.
- [ ] Medir concentração da seleção, diversidade de cobertura, frequência de ganho zero e participação de registros repetidos.
- [ ] Avaliar políticas alternativas como hipóteses independentes; não pressupor que ordenar por maior ganho resolva todos os efeitos.
- [ ] Definir protocolo com mesma configuração, dataset, orçamento e condição inicial do corpus; registrar versão do algoritmo em cada caso.
- [ ] Se uma alteração for aprovada, criar tarefa separada de 5–8 pontos; dividir se exceder 8.

Este registro não autoriza alterações adicionais de seleção nem modifica estimativas
ou status de implementação. As decisões posteriores acima estão nos documentos 001–003. Relaciona-se a D08 e às futuras análises de S07/S08/S16 no
[acompanhamento](003-acompanhamento-centralizacao-seeds.md).
