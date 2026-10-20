# Dogefuzz: esforço para separar o banco e permitir acesso paralelo

Data: 2026-10-04.

**Revisão posterior:** a exigência de evitar dupla escrita levou à proposta de
um banco central com isolamento por experimento e acervo derivado das próprias
transações. A estimativa e os esquemas desse escopo estão no
[plano detalhado sem dupla escrita](PLANO_ACERVO_COMPARTILHADO.md), que substitui
as faixas abaixo para essa alternativa.

## Escopo esclarecido: experimentos separados com seeds compartilhadas

O usuário esclareceu que as execuções realizam experimentos independentes e
compartilham inputs/seeds para tentar melhorar sua qualidade. Para esse escopo,
a recomendação e as estimativas desta seção substituem a alternativa de dois
fuzzers consumindo as mesmas tarefas discutida abaixo.

A proposta é preservar o banco operacional de cada experimento e acrescentar
um repositório compartilhado de seeds, com conexão e interface próprias.
Tarefas, transações, estados e eventos permanecem locais; ambos os processos
publicam e consultam seeds. Uma mesma seed pode ser consumida simultaneamente
pelos dois: não se trata de uma fila com consumidor exclusivo.

### Mudança necessária no código

Hoje, [power_schedule.go](dogefuzz/fuzz/power_schedule.go) obtém as seeds dos
inputs de transações concluídas, consultadas por `functionId`. Entretanto,
[function.go](dogefuzz/data/repo/function.go) atribui um UUID novo a cada função
criada. A mesma função Solidity em dois experimentos pode ter IDs diferentes;
portanto, compartilhar o banco ou copiar transações não estabelece uma
identidade comum para as seeds.

Introduzir uma interface `SeedRepository` com operações de publicação e
consulta, usada pelo power schedule, e um modelo próprio de seed. A identidade
de compatibilidade deve incluir uma identificação estável do contrato/artefato,
assinatura completa da função e versão da serialização. Endereços de deploy e
UUIDs locais não devem ser usados como identidade compartilhada. Quando houver
dependência de estado, incluir o cenário necessário ou limitar o MVP a casos
compatíveis e explicitamente definidos.

Cada seed contém inputs serializados, hash para deduplicação, origem do
experimento e contexto de compatibilidade. Observações de cobertura, distância
e falhas devem manter o contexto da execução que as produziu. O ganho incremental
de cobertura depende do que aquela execução já explorou; não é uma pontuação
universal da seed. Seeds externas devem ser reexecutadas e avaliadas localmente.
Inputs com endereços de contratos de outro ambiente podem exigir remapeamento;
apenas conseguir desserializar uma seed não comprova equivalência semântica.

Publicar candidatos após execução e avaliação, consultar periodicamente ou por
lote, combinar seeds externas com locais e manter uma parcela de exploração
aleatória. Usar restrição única por compatibilidade e hash dos inputs para
resolver publicações concorrentes da mesma seed. Indisponibilidade do acervo
compartilhado deve permitir continuar com seeds locais e tentar publicação
novamente de forma idempotente. Polling evita exigir um broker no MVP.

### Estimativa revisada

| Entrega | Dias de um desenvolvedor |
| --- | --- |
| MVP: repositório compartilhado, compatibilidade básica para o mesmo artefato/ABI, deduplicação, integração de publicação/consulta e teste com duas execuções | 6–10 |
| Versão ampliada: MVP mais recuperação de falhas, proveniência detalhada, seleção configurável, métricas e roteiro de comparação experimental | 10–16 |

As faixas são totais alternativos e incluem um banco compartilhado independente;
não é necessário migrar os bancos operacionais existentes. Pressupõem cada
execução já possuir seu próprio ambiente operacional/blockchain. Isolar o Geth,
remapear endereços entre cenários arbitrários, reproduzir sequências completas
de transações e executar uma campanha científica extensa são trabalhos
adicionais não quantificados aqui.

Para avaliar a hipótese de melhoria, comparar grupos sem e com compartilhamento,
mantendo orçamento total de execução comparável e fazendo várias repetições.
Registrar cobertura, tempo/transações até falhas e proporção de seeds importadas
que trouxeram ganho local. As execuções cooperantes permanecem operacionalmente
separadas, mas seus resultados passam a ter dependência experimental; isso deve
ser descrito no estudo. Compartilhar seeds é uma hipótese de benefício, não uma
garantia de qualidade ou de aumento de cobertura.

## Conclusão e premissas

Separar a persistência é viável. A estimativa inicial é de **5–9 dias de trabalho
para um banco servidor independente com acesso por duas aplicações**, e de
**12–21 dias para duas instâncias do fuzzer compartilharem trabalho com
coordenação e recuperação de falhas**. Estas faixas representam esforço de um
desenvolvedor familiarizado com Go, GORM e o projeto, considerando 8 horas por
dia. São estimativas de engenharia, não resultados de medições.

A análise considera o banco da aplicação `dogefuzz`, com contratos, funções,
tarefas e transações. Não considera separar o armazenamento interno do Geth.
Foi feita leitura estática do código; não houve execução de testes, benchmark,
inspeção de uma base real ou mudança na implementação.

O significado de “execução em paralelo” altera bastante o escopo: um processo
de relatórios consultando resultados exige menos trabalho que dois fuzzers
consumindo e atualizando as mesmas tarefas. As alternativas abaixo distinguem
esses casos. Os totais são alternativas completas, não parcelas a somar.

## Estado atual e pontos de acoplamento

| Evidência no código | Consequência para a separação |
| --- | --- |
| [data/connection.go](dogefuzz/data/connection.go) abre SQLite em `<DatabaseName>.db`, solicita WAL e executa `AutoMigrate` | Persistência embutida no processo; não existe servidor de banco independente |
| [interfaces/data.go](dogefuzz/pkg/interfaces/data.go) expõe `GetDB() *gorm.DB`; [interfaces/repo.go](dogefuzz/pkg/interfaces/repo.go) recebe `*gorm.DB` em todos os repositórios | Há uma camada de dados, mas o ORM atravessa a fronteira dos serviços |
| [cmd/dogefuzz/env.go](dogefuzz/cmd/dogefuzz/env.go) abre e migra no primeiro `DbConnection()` | Cada instância inicializa o esquema; é necessário definir um responsável pelas migrações |
| [service/transaction.go](dogefuzz/service/transaction.go) usa mutex local em `BulkCreate` e `BulkUpdate`, com transações SQL | O mutex protege somente aquela instância; erros de `Begin` e `Commit` não são verificados explicitamente |
| [job/transactions_checker.go](dogefuzz/job/transactions_checker.go) consulta tarefas e publica eventos, sem reservar o trabalho | Duas instâncias podem selecionar a mesma tarefa e gerar lotes simultaneamente |
| [pkg/bus/memory.go](dogefuzz/pkg/bus/memory.go) implementa eventos em memória | Uma escrita feita por outro processo não entrega automaticamente eventos ao fuzzer |
| [environment/contract_pool.go](dogefuzz/environment/contract_pool.go) usa IDs fixos e faz deploy dos auxiliares durante setup | Inicializações simultâneas precisam coordenar criação, deploy e atualização |
| [data/repo/transaction.go](dogefuzz/data/repo/transaction.go) usa `strftime` em `FindTimeTakenToWeakness` | Trocar o driver não basta: existe SQL específico do SQLite |

## Alternativas e esforço

| Alternativa | Entrega | Esforço estimado |
| --- | --- | --- |
| SQLite compartilhado localmente | Um fuzzer grava e outro processo consulta o mesmo arquivo, com configuração e ciclo de vida explícitos | 2–4 dias / 16–32 h |
| Pacote Go reutilizável de persistência | Outro executável importa repositórios e modelos sem depender do ambiente completo do fuzzer | 3–6 dias / 24–48 h |
| PostgreSQL em serviço/container próprio | Duas aplicações acessam o banco pela rede; inclui adaptação SQL, migração de dados e testes de integração | 5–9 dias / 40–72 h |
| Serviço de persistência com API | Clientes acessam operações HTTP/gRPC; somente o serviço conhece GORM e o banco | 10–16 dias / 80–128 h |
| PostgreSQL e dois fuzzers coordenados | Banco externo mais reserva de tarefas, recuperação, coordenação de eventos e validação com dois processos | 12–21 dias / 96–168 h |

Um pacote Go separa código, mas não executa sozinho nem cria comunicação entre
processos. Um PostgreSQL separado já oferece um endpoint de banco; uma API
própria só é necessária se houver intenção de centralizar regras de persistência
ou atender clientes que não devem conhecer o esquema.

SQLite com WAL permite leitores e escritor em paralelo, mas mantém apenas um
escritor por vez e exige processos no mesmo host, sem compartilhar o arquivo
via filesystem de rede. Por isso, a alternativa local pode atender relatórios,
mas não deve ser tratada como uma solução geral de banco remoto.
Fonte: [documentação oficial do SQLite sobre WAL](https://www.sqlite.org/wal.html).

GORM oferece drivers para SQLite e PostgreSQL, o que permite reaproveitar parte
dos modelos e repositórios. Isso não garante compatibilidade do SQL manual, dos
tipos ou dos dados históricos.
Fonte: [documentação oficial de conexão do GORM](https://gorm.io/docs/connecting_to_the_database.html).

## Plano estimado para um banco independente

Recomendação para acesso por processos em containers ou máquinas diferentes:
usar PostgreSQL como serviço independente, mantendo a camada de persistência
no projeto inicialmente. Esta é uma escolha arquitetural para o cenário, não
uma conclusão de benchmark.

| Etapa | Trabalho | Dias |
| --- | --- | --- |
| Conexão e infraestrutura | Driver, DSN, pool, volume persistente, readiness e encerramento explícito | 1–2 |
| Compatibilidade | Revisar os quatro modelos/repositórios e adaptar SQL, especialmente `strftime` | 1–2 |
| Esquema e dados | Migrações com execução controlada; transferência de uma base SQLite representativa e conferência | 1–2 |
| Integração e documentação | Validar duas conexões/processos, falhas de conexão, reabertura e preservação de dados | 2–3 |
| **Total** | **Banco servidor acessível por duas aplicações** | **5–9** |

Essa entrega permite conexões concorrentes. Não garante distribuição correta
do trabalho entre fuzzers nem comunicação de eventos entre aplicações. Um
processo externo que criar tarefas diretamente no banco pode deixar de acionar
o fluxo esperado pela API e pelos tópicos em memória.

## Acréscimo necessário para dois fuzzers ativos

Sobre a entrega anterior, estimam-se **7–12 dias adicionais**, totalizando
**12–21 dias**:

| Etapa | Trabalho | Dias adicionais |
| --- | --- | --- |
| Reserva de trabalho e recuperação | Definir a unidade de trabalho; reservar atomicamente tarefa ou lote; registrar executor e prazo; recuperar após queda | 2–3 |
| Consistência das atualizações | Atualizações condicionais por estado/versão, verificação de commit, prevenção de efeitos duplicados e política de repetição | 1–2 |
| Startup, eventos e blockchain | Definir responsável pelo setup, roteamento dos resultados e política para carteiras/nonces e nós Geth | 2–4 |
| Validação multiprocesso | Duas instâncias reais, queda durante execução, retomada e medição inicial de contenção | 2–3 |

Exemplo do risco atual: A e B consultam a mesma tarefa em execução; ambos veem
o limite de transações disponível e publicam seu evento local; cada listener
gera um lote. O banco pode aceitar as duas gravações corretamente, enquanto o
limite de trabalho e a coordenação do fuzzer ficam incorretos.

A reserva deve ser curta e atômica. Não se deve manter uma transação SQL aberta
durante compilação ou chamadas ao Geth. Recuperação precisa distinguir trabalho
ainda não enviado de transação já enviada à blockchain, para evitar reenviar
um efeito externo ao repetir a execução. Compartilhar o mesmo Geth, carteiras
ou contratos também requer uma política explícita; o banco não resolve essa
coordenação por si só.

Se forem experimentos independentes, considerar um identificador de execução
nos dados ou bancos/esquemas separados. No código atual, tarefas e transações
não possuem `run_id`, e os auxiliares usam identificadores globais fixos.
A estimativa multiprocesso pressupõe compartilhamento coordenado de trabalho,
não isolamento completo de experimentos e ambientes blockchain.

## Critérios de aceite e limites

- A execução B consegue consultar os dados gravados por A e continuar após A encerrar.
- Reiniciar o serviço de banco preserva dados e produz erros identificáveis durante indisponibilidade.
- Migrações têm um responsável definido e são concluídas antes de liberar consumidores.
- Dados migrados preservam IDs, associações e resultados das consultas relevantes.
- Para dois fuzzers: a mesma unidade de trabalho não é atribuída simultaneamente a ambos; uma queda permite retomada sem reenvio indevido de transações.
- Fechar uma conexão não exclui o banco. A rotina atual `Clean()` deve permanecer restrita à limpeza explícita de dados de teste e ser revista.

As faixas incluem validação da solução escolhida, mas não incluem alta
disponibilidade, implantação em nuvem, operação de produção, isolamento completo
de múltiplas blockchains ou garantia de desempenho sob carga ainda desconhecida.
Uma base histórica grande ou divergente e a escolha de um broker externo podem
exigir revisão da estimativa. A taxa de gravação e o comportamento com dois
processos devem ser medidos antes de prometer ganho de desempenho.
