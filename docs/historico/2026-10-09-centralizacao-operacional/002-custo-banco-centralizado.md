# Custo e tarefas — banco centralizado

Criado em: 2026-10-06. Revisado em: 2026-10-08.

Status: estimativa preliminar revisada para o escopo simplificado. As horas
resultam de análise estática, sem medição de produtividade ou compromisso de prazo.

Referências: [brainstorm consolidado](001-brainstorm-banco-centralizada.md) e
[acompanhamento](003-acompanhamento-banco-centralizado.md).

## 1. Escopo e premissas

Compartilhar seeds entre chamadas independentes do benchmark, em paralelo,
sequencialmente e em chamadas futuras. Preservar comandos, execução sequencial
interna, taskId, relatórios individuais, seleção, mutadores e complemento atual
para candidatos insuficientes.

A chave compartilhada é
`(SHA-256(contractSource exato), contract_name, function_signature)`.
Hash e assinatura são calculados no cadastro, armazenados e indexados. Os UUIDs
locais continuam identificando registros operacionais.

A estimativa considera centralização de todas as tabelas operacionais em
PostgreSQL, com **tabelas comuns e instance_id**, escolha confirmada em 2026-10-08.
O próprio Dogefuzz gera um UUID por inicialização, constante entre tarefas e
novo a cada reinício. Propagação interna, filtros, restrições e índices precisam
ser implementados. Registros e estados anteriores serão preservados sem retomada,
reenvio ou reclassificação automática; somente transações concluídas poderão ser
candidatas a seeds compartilhadas. Auxiliares terão UUID próprio, instance_id e
papel, com unicidade em (instance_id, papel), novos registros e deployments por
inicialização e histórico preservado. A estimativa inclui sua implementação.
Escolher outra arquitetura de persistência exige reestimativa.

### Impacto da escolha no backlog

| Tarefas | Impacto |
| --- | --- |
| 01 e 04 | Gerar UUID no bootstrap e propagar internamente; preservar registros/estados sem retomada ou reenvio, sem novos IDs experimentais |
| 03 | Acrescentar instance_id obrigatório nas quatro entidades principais atuais; definir integridade de relações e índices; confirmar previamente qualquer nova tabela |
| 05–08 | Revisar todas as operações normais, inclusive SQL direto, jobs e callbacks |
| 09 | Criar auxiliares com UUID próprio e unicidade por instância/papel; adaptar setup e consumidores; novos deployments por inicialização, histórico preservado |
| 07 e 10 | Distinguir hashes por contexto e preservar dados no encerramento |
| 13–15 | Permitir leitura compartilhada pela chave de função sem autorizar operações externas |
| 17 e 18 | Configurar o contexto de cada instância e o acesso ao banco comum |
| 24–28 | Demonstrar isolamento e reuso do corpus em chamadas independentes |

Obrigatoriedade de instance_id confirmada em D03: tasks, contracts, functions e
transactions, inclusive auxiliares em contracts. Qualquer nova tabela exige
confirmação do usuário sobre inclusão, finalidade e necessidade de instance_id
antes da implementação; não há extensão automática da regra. Criação usa o
contexto interno, sem valor nulo/vazio; a propriedade permanece imutável. A escolha
detalha o escopo já estimado, sem alterar pontos ou horas. Repetir o campo exige
garantir consistência nas relações.

Integridade aprovada em D03: validação na aplicação e restrições no banco. Manter
UUIDs principais e incluir instance_id nas relações protegidas. Contrato/tarefa,
função/contrato e transação/tarefa/função devem respeitar a mesma instância; a função
da transação deve pertencer ao contrato da tarefa. Auxiliares podem existir sem
tarefa. Impactos nas tarefas 03, 05–07 e 24: migração, validações e testes negativos,
inclusive entre tarefas distintas da mesma instância. O desenho exato das chaves
e restrições permanece pendente; não basta validar só instance_id.
Estimativas mantidas por detalhar o escopo já previsto; reavaliar se o desenho
concreto revelar complexidade adicional.

A revisão inicial das operações é mais abrangente que configurar um schema
privado para cada instância. Em contrapartida, há um conjunto de tabelas e uma
consulta central de seeds. Schemas separados não foram escolhidos porque
distribuiriam o histórico persistente e acrescentariam descoberta/combinação de
schemas e migrações por schema ao acervo entre chamadas futuras.

Os pontos e horas desta revisão já consideram isolamento em tabelas centralizadas;
não foram aumentados ou reduzidos somente pela confirmação da escolha. Refazer
a estimativa se o inventário de consultas ou as restrições trouxerem trabalho
além das premissas. Não há benchmark de desempenho das alternativas.

Outras premissas:

- Banco inicialmente novo, sem importação de SQLite histórico.
- Geth próprio por chamada; banco e acervo sobrevivem ao encerramento.
- Três variantes greybox, sem novo filtro obrigatório de candidatos por ganho.
- Código-fonte exato, sem normalização; bytecode/configuração de compilação não
  integram a identidade básica nesta etapa.
- Compartilhamento de argumentos; remapeamento de endereços e replay ficam fora.
- Novos IDs de campanha/repetição/execução não são requisitos por si só.
- Cinco chamadas são o exemplo atual; não há limite fixo de quatro workers.
- Usar B2_parted como referência informada pelo usuário e preparar testes curtos;
  campanha científica completa e análise estatística ficam fora.
- Correções da seleção permanecem como dívida técnica separada.

## 2. Estimativa revisada

**25 tarefas ativas, 164 pontos, média de 6,6 pontos; nenhuma acima de 8.**

Esforço: **158–284 horas**, incluindo implementação e verificação.
Equivale a aproximadamente **20–36 dias de 8 horas**, arredondados para cima,
para uma pessoa familiarizada com Go/GORM, Python e Docker. Disponibilidade parcial
aumenta o calendário.

Pontos são relativos à complexidade e risco, sem conversão fixa para horas.
Com 20% de contingência, reservar aproximadamente **190–341 horas**. A reserva
não autoriza ampliar o escopo nem corresponde a tarefas adicionais.

| Frente | Tarefas | Pontos | Horas |
| --- | ---: | ---: | ---: |
| A. Banco e isolamento operacional | 10 | 71 | 68–122 |
| B. Identificação e compartilhamento de seeds | 6 | 39 | 38–68 |
| C. Integração com chamadas independentes | 4 | 20 | 18–34 |
| D. Validação e documentação | 5 | 34 | 34–60 |
| **Total** | **25** | **164** | **158–284** |

As faixas continuam condicionadas aos detalhes do isolamento por instance_id e
aos filtros de compatibilidade que serão fechados. Não presumem que centralizar
a conexão basta para compartilhar seeds.

## 3. Backlog ativo

Os IDs originais foram preservados para rastreabilidade. Cada tarefa inclui
testes relacionados; a frente D complementa com integração. A ordem numérica
não é rígida: ver predecessoras no documento 003.

### A. Banco e isolamento operacional

| ID | Tarefa e entrega verificável | Pontos | Horas |
| --- | --- | ---: | ---: |
| 01 | **Isolamento mínimo e ciclo de vida:** Confirmar os IDs atuais e documentar o escopo operacional mínimo, sem exigir IDs de campanha ou repetição. Conferir o contrato com os fluxos atuais; registrar as decisões ainda necessárias. | 5 | 4–8 |
| 02 | **Conexão PostgreSQL:** Configurar driver, acesso, pool e readiness para uma instância. Verificar erro identificável na indisponibilidade e fechamento sem apagar dados. | 5 | 6–10 |
| 03 | **Esquema e migração:** Adicionar campos de identidade compartilhada, contexto mínimo e restrições. Verificar índices, integridade e reaplicação da migração. | 8 | 8–14 |
| 04 | **Contexto operacional mínimo:** Propagar somente o contexto necessário ao isolamento nos serviços e mapeadores. Rejeitar associações operacionais entre contextos distintos. | 8 | 6–12 |
| 05 | **Isolamento de tarefas:** Aplicar isolamento a leituras, atualizações e consultas de tarefas ativas. Testar IDs externos sem leitura ou alteração indevida. | 8 | 8–14 |
| 06 | **Isolamento de contratos e funções:** Isolar cadastro, busca, listagem e atualização de registros locais. Verificar deployments equivalentes em instâncias diferentes sem cruzamento operacional. | 8 | 8–14 |
| 07 | **Isolamento de transações:** Isolar CRUD e consultas operacionais por tarefa/função. Verificar que somente a consulta compartilhada de seeds atravessa a fronteira. | 8 | 8–14 |
| 08 | **Isolamento de jobs e callbacks:** Revisar geração, timeout, análise e finalização pelo inventário de operações. Demonstrar que um processo não executa ou finaliza tarefas de outro. | 8 | 8–14 |
| 09 | **Auxiliares por ambiente:** Criar UUID por auxiliar, garantir unicidade por instância/papel e adaptar setup/consumidores. Implantar novos auxiliares por inicialização, preservando histórico. Testar isolamento e reinício. | 8 | 8–14 |
| 10 | **SQL, erros e encerramento:** Adaptar SQL específico do SQLite, incluindo cálculo temporal. Preservar o banco no encerramento e impedir reenvio indevido ao Geth. | 5 | 4–8 |

**Marco A:** duas instâncias operam no banco comum sem cruzamento de tarefas,
deployments, auxiliares, transações ou callbacks, com compartilhamento desativado.

### B. Identificação e compartilhamento de seeds

| ID | Tarefa e entrega verificável | Pontos | Horas |
| --- | --- | ---: | ---: |
| 11 | **Representação de inputs:** Definir representação suficiente para os tipos suportados sem ambiguidade. Preservar interface externa e registrar a versão do formato, se necessária. | 5 | 6–10 |
| 12 | **Chave compartilhada da função:** Calcular SHA-256 do código-fonte exato e persistir nome da declaração selecionada. Testar iguais/diferentes fontes, múltiplas declarações e funções sobrecarregadas; indexar a consulta. | 5 | 4–8 |
| 13 | **Consulta local/shared:** Consultar candidatos concluídos pela chave source_hash + contract_name + function_signature. Demonstrar consulta de seeds de outra chamada e de chamadas anteriores, excluindo incompletas/incompatíveis. | 8 | 8–14 |
| 14 | **Conclusão, elegibilidade e origem:** Disponibilizar o input somente com resultado utilizável e commit confirmado, sem filtro obrigatório de ganho. Verificar rastreabilidade e ausência de cópia para segundo acervo persistente. | 8 | 8–14 |
| 15 | **Integração das três greybox:** Confirmar Marco A e integrar greybox, directed_greybox e other_directed_greybox. Demonstrar mutação de seed externa e resultado próprio no Geth consumidor. | 8 | 8–14 |
| 16 | **Métricas mínimas de uso de seeds:** Definir dados mínimos de origem, ganho local de descendentes e custo da seleção. Verificar consistência dos dados e preservação das métricas atuais. | 5 | 4–8 |

**Marco B:** uma chamada encontra uma seed externa pela chave, aplica mutação,
avalia resultado próprio e usa o complemento atual quando necessário.

### C. Integração com chamadas independentes

| ID | Tarefa e entrega verificável | Pontos | Horas |
| --- | --- | ---: | ---: |
| 17 | **Banco comum e chamadas independentes:** Permitir que projetos Docker independentes alcancem o banco persistente. Validar readiness e encerramento de uma chamada sem remover o banco comum. | 5 | 6–10 |
| 18 | **Configuração mínima do benchmark:** Expor somente configuração necessária ao banco, isolamento e modo local/shared. Validar chamadas independentes sem novos IDs experimentais obrigatórios. | 5 | 4–8 |
| 21 | **Reinício e falhas no fluxo atual:** Verificar reinício do par pertencente à chamada e efeitos no banco persistente. Demonstrar que falha de uma chamada não encerra nem altera tarefas de outra. | 5 | 4–8 |
| 22 | **Relatórios próprios e metadados mínimos:** Preservar estrutura atual de relatório por chamada. Verificar roteamento de cada relatório e caminhos distintos para salvar resultados concorrentes. | 5 | 4–8 |

**Marco C:** chamadas independentes usam o mesmo banco e mantêm seus endpoints,
Geths e relatórios; encerramento/reinício de uma não apaga o acervo de outra.

### D. Validação e documentação

| ID | Tarefa e entrega verificável | Pontos | Horas |
| --- | --- | ---: | ---: |
| 24 | **Validação de isolamento multiprocesso:** Executar dois processos/Geths no banco comum com compartilhamento desativado. Registrar evidência do Marco A; usar fixture de dois processos sem depender de 17. | 8 | 8–14 |
| 25 | **Validação das estratégias e do reuso:** Exercitar três greybox nos modos local/shared. Demonstrar reuso de chamadas anteriores sem reset por repetição. | 8 | 8–14 |
| 26 | **Validação de falhas e carga:** Exercitar indisponibilidade do banco e reinício de uma instância. Verificar persistência, erros identificáveis e ausência de processamento cruzado ou reenvios indevidos. | 8 | 8–14 |
| 27 | **Operação e dívida técnica:** Documentar banco, migração, modos, reuso do acervo e limitações. Conferir roteiro e registrar reset como pendente, sem implementar limpeza não definida. | 5 | 4–8 |
| 28 | **Comparativo curto de aceitação:** Executar chamadas independentes em paralelo e em sequência sobre contratos selecionados de B2_parted. Registrar orçamento e corpus inicial; não inferir ganho científico a partir do teste curto. | 5 | 6–10 |

**Marco D:** testes paralelos e sequenciais demonstram reuso persistente e
relatórios próprios. O aceite funcional não demonstra ganho científico.

## 4. Tarefas retiradas do caminho obrigatório

| ID original | Tarefa anterior | Tratamento atual |
| --- | --- | --- |
| 19 | Coordenador paralelo único | Fora do escopo; o usuário executa chamadas independentes |
| 20 | Coletor central concorrente de webhooks | Fora do escopo; preservar receptor por chamada e registrar limitação atual como dívida técnica |
| 23 | Agregação e relatório conjunto | Fora do escopo; cada chamada continua produzindo seu relatório |

Essas tarefas não estão concluídas nem somam pontos/horas da faixa revisada.
Só retornam ao backlog por nova decisão. Verificar roteamento de relatórios
continua incluído nas tarefas 18/22; isso não exige um coletor global.

A estimativa original era **28 tarefas, 194 pontos, 208–356 horas**, para uma
arquitetura mais ampla. Esta revisão substitui aquela referência, sem somar as
faixas. A redução também considera simplificação de identidade, infraestrutura,
reinício e metadados; não é apenas a subtração das três tarefas.

O [plano anterior](PLANO_ACERVO_COMPARTILHADO.md), de 88–144 horas, tinha outras
premissas, incluindo foco inicial em greybox de cobertura. Não representa o
escopo completo deste backlog e não foi alterado nesta revisão.

## 5. Dependências, aceite e questões abertas

Sequência: **isolamento mínimo → banco/esquema → operações isoladas → consulta
de seeds → integração → validação entre chamadas**.

A configuração do benchmark pode avançar em paralelo após o contrato operacional.
Tarefa 24 verifica o isolamento e libera integração compartilhada na tarefa 15.
Não exigir um coordenador único ou relatório conjunto para concluir a entrega.

Interfaces mínimas previstas:

- Identidade estável do contrato e assinatura persistidas nos modelos locais.
- Consulta de seeds por chave compartilhada, com modo local/shared.
- Contexto interno necessário ao isolamento, sem novos IDs experimentais obrigatórios.
- Origem da seed preservada até a nova transação.
- Configuração de acesso ao banco comum e metadados mínimos de avaliação.

O aceite exige operação isolada, reuso entre chamadas paralelas/sequenciais,
persistência em chamada futura, mutação e avaliação local, fallback atual para
seeds insuficientes e relatórios separados.

Preservar estados pode deixar tarefas antigas marcadas como em andamento. Detecção
de instâncias inativas e reclassificação automática ficam fora desta etapa.

Novos auxiliares por inicialização repetem compilação/deploy e aumentam o histórico
persistido; não alteram o fluxo do benchmark. A escolha detalha o escopo já estimado
da tarefa 09, mantida em 8 pontos e 8–14 horas; totais permanecem inalterados.

Ainda fechar: filtros/restrições gerais,
configuração/migração, formato dos inputs,
filtros adicionais de cenário e variantes dirigidas, fallback/receive, janela
de candidatos e detalhes das métricas. D01 está definida documentalmente; D03/D05
permanecem parciais. Definição documental não equivale a implementação.

No comparativo, manter orçamento e recursos documentados e registrar se o
acervo começou vazio ou acumulado. Não impor corpus novo a cada repetição.
Chamadas que herdam ou trocam seeds não têm independência experimental presumida.

## 6. Dívida técnica e itens fora da faixa

Registrar sem corrigir silenciosamente:

- Limite ignorado na consulta atual de transações.
- Ordenação crescente por DeltaCoverage, cuja intenção precisa ser revisada.
- Risco de divisão por zero/score não finito na seleção dirigida.
- Descarte de webhook de tarefa diferente da aguardada no coletor atual.

Se algum problema bloquear o escopo preservado, abrir tarefa separada de até
8 pontos e aplicar correções de política igualmente aos dois modos comparados.
Identificar e documentar essas dívidas está incluído; corrigi-las não está.

Eventual script explícito de reset permanece **pendente de definição e orçamento**.
Não incluir limpeza automática. Apagar todo o banco elimina histórico e origem
das seeds; limpar somente o acervo depende da forma escolhida de elegibilidade.
Não foi implementado script nem realizada limpeza.

Importação histórica, replay, remapeamento, alta disponibilidade, coordenador
único, relatório conjunto e campanha científica completa ficam fora da faixa.

Reavaliar custos após a tarefa 01 e o Marco A. Se uma tarefa superar 8 pontos,
dividi-la por comportamento verificável, preservando seu vínculo histórico.
