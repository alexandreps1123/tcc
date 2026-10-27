# Entrega — ambiente local, debug e chamada de avaliação

Data: 2026-10-20. Componente: Dogefuzz, com dependências locais Geth/Vandal e
PostgreSQL. Escopo: [README.md](README.md).

## Referência do código e preservação

Referência inicial e final do Dogefuzz: branch `task/s02-s03-preparacao-seeds`,
HEAD `1aa40f3e238f3ad330201cff8401074651d1c99f`. A referência `main` permanece
nesse mesmo commit. Não foram realizados commits, merges ou pushes.

As alterações S02/S03 já estavam presentes e não foram modificadas. Os dez
arquivos foram conferidos contra os SHA-256 do relatório daquela entrega.
Seus testes foram executados novamente nesta atividade, sem substituir revisão
independente ou encerrar os respectivos checkpoints globais. Referências
congeladas e arquivos originais de configuração/infraestrutura foram preservados.

## Arquivos entregues

- `dogefuzz/infra/docker-compose.local.yml`: builds dos fontes locais, portas
  separadas, healthchecks, PostgreSQL 17 e volume persistente do acervo.
- `dogefuzz/scripts/local.py`: preparação de runtime isolado, execução, build de
  debug, instalação de Delve 1.27.2, testes e chamada SimpleDAO com evidências.
- `dogefuzz/Makefile`: comandos `local.prepare/up/down/logs/run/debug/build/test/smoke/tools`.
- `dogefuzz/.gitignore`: runtime/cache e bytecode Python fora do versionamento.
- `dogefuzz/README.md`: acesso ao guia local.
- `.vscode/launch.json`, `tasks.json`, `settings.json`, `extensions.json` na raiz
  do workspace: perfis de debug, tarefas e caminho do Delve local.
- Esta pasta: guia e relatório; documento 003: referência ao ambiente de avaliação.

O helper gera `dogefuzz/.local/runtime/config.json`, um link para os assets,
SQLite próprio e diretório de armazenamento/compiladores. O arquivo base
`dogefuzz/config.json` não foi alterado. A preparação inclui `seedCorpus`, ainda
ignorado pelo código atual. Nenhuma tabela, migração ou integração PostgreSQL
foi implementada. Encerramento com cancelamento imediato continua na S04.

## Verificações realizadas

Ambiente: Go 1.26.0 linux/amd64, Python 3.12, Docker Compose v5.1.0 e Delve
1.27.2. O container iniciado com `postgres:17` reportou PostgreSQL 17.8; o tag
local não fixa patch/digest da infraestrutura definitiva.

| Verificação | Resultado |
| --- | --- |
| `make local.prepare` | Configuração e assets locais gerados. |
| `docker compose -p dogefuzz-local -f infra/docker-compose.local.yml config --quiet` | Configuração válida. |
| `make local.test` | Passaram fuzz, service, controller, seedidentity, mapper, solidity e config, com `-count=1`. |
| `make local.build` | Binário compilado com `-gcflags=all=-N -l`. |
| Instalação do Delve | Versão 1.27.2 instalada em `.local/bin/dlv`; comando fixado nessa versão. |
| Delve no binário local | Breakpoint em `main.main` atingido; stack exibida; processo encerrado antes do startup. |
| Perfis VS Code | JSON válido e propriedades conferidas com o schema da extensão Go 0.56.1 instalada. A interface F5 não foi operada pelo agente. |
| Build Docker de Geth/Vandal | Concluído a partir dos fontes locais, sem alterar os componentes. |
| `make local.up` com `--wait` | Três serviços healthy: Geth, Vandal e PostgreSQL. |
| Helper com servidor HTTP controlado | Preservou bytes da fonte, aceitou relatório válido e rejeitou relatório sem cobertura. |
| Chamada real SimpleDAO/greybox/60s | Concluída com relatório validado; resultado abaixo. |
| `git diff --check` | Sem erros de whitespace no workspace e no componente. |

Os comandos que acessam Docker, rede/loopback ou ptrace precisaram de execução
fora do sandbox. Não foram falhas do produto. O build Go emitiu um aviso ao tentar
gravar metadados no cache global somente leitura, mas retornou sucesso; o cache
de compilação local foi usado. Downloads de compiladores Solidity ocorreram no
startup atual, sem exigir instalação de solc no PATH.

## Resultado da chamada real

Artefatos locais em
`dogefuzz/.local/evaluations/20261020T033328537598Z/`: `request.json`,
`response.json`, `metadata.json` e `report.json`. Log do servidor em
`dogefuzz/.local/dogefuzz.log`.

| Campo | Resultado |
| --- | --- |
| Contrato | SimpleDAO, dataset B2_parted/sol/subdir7/simple_dao.sol |
| Estratégia / duração solicitada | greybox / 60 segundos |
| taskId | b710efde-5be6-4951-8331-92069d6f2ccf |
| taskStatus | done |
| totalInstructions | 25 |
| coverage | 16, valor bruto do relatório atual |
| Soma de instructionHitsHeatMap | 4.084 |
| detectedWeaknesses | reentrancy |

O script retornou exit code 0 após conferir taskId, status, CFG, cobertura e hits.
A detecção de reentrância foi observada, mas não é exigida em toda execução curta
e aleatória. Este resultado não mede ganho do acervo e não substitui o protocolo
experimental de comparação. Não houve execução real dos dois modos dirigidos
nesta atividade; os comandos estão preparados e os orderers passaram nos testes.

## Estado final e próximos passos

O servidor Dogefuzz e a sessão de debug de validação foram encerrados para liberar
a porta 3456. Os três containers locais permanecem disponíveis; `make local.down`
os encerra sem remover o volume do acervo. Para avaliar novamente, executar
`make local.run` ou o perfil F5 e, em outro terminal, `make local.smoke`.

S04 permanece pronta para implementação, sem código PostgreSQL entregue por esta
atividade. A infraestrutura local facilita seus testes; o desenho definitivo de
S11, esquema aprovado de S05 e testes de reuso/concorrência permanecem pendentes.
