# Execução local, debug e avaliação curta

Componente: Dogefuzz. Criado em 2026-10-20.
Triage: needs-triage
Status: entregue — avaliação pendente.

Objetivo: preparar execução reproduzível da branch de desenvolvimento, debug e
uma chamada curta para avaliar implementações. Não implementar S04 ou criar
tabelas do acervo nesta atividade. Preservar `main` e alterações S02/S03 existentes.

## Preparação e execução

Requisitos: Linux, Python 3, Go compatível com o componente, compilador C para
SQLite/CGO, Docker com Compose e acesso à internet para os builds e compiladores
Solidity. Para debug no VS Code, abrir a raiz `tcc` e instalar a extensão Go e
as ferramentas que ela solicitar, incluindo Delve.

No terminal:

```bash
cd /home/parkus/tcc/dogefuzz
make local.up
make local.test
make local.run
```

`local.up` aguarda os healthchecks (até 180s após o build) e usa os fontes locais
de Geth modificado e Vandal. Não substituir pelo
Geth oficial: o fluxo depende dos callbacks de execução da versão modificada.
Os builds antigos desses componentes podem exigir diagnóstico próprio.
O servidor Dogefuzz roda no host, acessível aos callbacks do Geth pelo endereço
`host.docker.internal:3456`. No Linux, o Compose configura `host-gateway`.

| Serviço | Endereço no host |
| --- | --- |
| Dogefuzz | http://127.0.0.1:3456 |
| Geth modificado | http://127.0.0.1:18545 |
| Vandal | http://127.0.0.1:15005 |
| PostgreSQL 17 | 127.0.0.1:15432 |

PostgreSQL: usuário `dogefuzz`, senha de desenvolvimento `local_dev_only`, banco
`dogefuzz_seeds`. DSN local:
`postgres://dogefuzz:local_dev_only@127.0.0.1:15432/dogefuzz_seeds?sslmode=disable`.
Portas das dependências são publicadas apenas em loopback; a API atual do Dogefuzz
escuta em todas as interfaces para receber callbacks do container.

O helper gera `dogefuzz/.local/runtime/config.json` a partir do arquivo original,
com endpoints locais, SQLite próprio, reporter de arquivo e `seedCorpus` preparado
com os valores aprovados. O código atual ainda ignora `seedCorpus`: a presença do
banco/configuração não demonstra a implementação de S04. `postgres:17` acompanha
a versão principal aprovada; fixar patch/digest da infraestrutura definitiva é S11.
`DOGEFUZZ_SEEDCORPUS_DSN` pode substituir a DSN gerada durante a preparação.

Os fontes, `config.json` original e snapshots das tarefas permanecem preservados.
Artefatos, cache e compiladores ficam em `.local/`, ignorada pelo Git. Cada
preparação regenera a configuração local; ajustes manuais nela serão substituídos.

## Debug

1. Executar `make local.tools` uma vez para instalar Delve 1.27.2 em `.local/bin`, depois
   `make local.up` e aguardar a disponibilidade das dependências.
2. No VS Code aberto em `tcc`, selecionar **Dogefuzz: debug local** e pressionar F5.
3. Colocar breakpoints no cadastro (`controller/tasks.go`), identidade
   (`pkg/seedidentity/identity.go`) e seleção (`fuzz/power_schedule.go`).
4. Em outro terminal, executar `make local.smoke`.

O perfil **Dogefuzz: debug testes de seeds** permite avaliar adapter e seleção
sem blockchain ou containers. Para identidade, executar/debugar também os testes
de `pkg/seedidentity`, `controller` e `service` pela extensão Go.

Alternativa no terminal, após `make local.tools`: `make local.debug`, seguido
de `break`, `continue` e demais comandos do Delve. Não executar os dois servidores
simultaneamente na mesma porta. Ao pausar threads no debugger, os relógios de
timeout/duração continuam correndo; aumentar os prazos da chamada se necessário.

## Chamada de avaliação

Com o servidor ativo em outro terminal:

```bash
cd /home/parkus/tcc/dogefuzz
make local.smoke
```

A chamada envia `POST /tasks` com o SimpleDAO do dataset
`dogefuzz-benchmark/dataset/B2_parted/sol/subdir7/simple_dao.sol`, fonte exata,
`arguments: []`, `duration: 60s`, `fuzzingType: greybox` e detector `reentrancy`.
`arguments` são argumentos do construtor; a seleção de métodos segue o fuzzer.

Para os demais modos ou uma sessão de debug mais longa:

```bash
python3 scripts/local.py smoke --strategy directed_greybox
python3 scripts/local.py smoke --strategy other_directed_greybox
python3 scripts/local.py smoke --duration 300 --request-timeout 600 --wait-timeout 600
```

O helper valida `/ping`, registra request, resposta/taskId, branch, commit e
estado do working tree, aguarda o relatório do mesmo taskId e salva cópia em
`.local/evaluations/<data-UTC>/`. Exige status `done`, CFG, cobertura e hits
positivos; falha HTTP, relatório de erro ou timeout retornam exit code diferente
de zero. Não repete o POST automaticamente. Vulnerabilidade específica não é
critério obrigatório em uma execução aleatória curta. Executar uma chamada por
vez: o reporter atual sobrescreve o mesmo arquivo antes da cópia pelo helper.

O primeiro startup compila e implanta auxiliares, podendo baixar solc 0.4.26 e
0.8.19 automaticamente. O SimpleDAO usa um pragma amplo da série 0.4; o compilador
atual escolhe a versão compatível disponível. Não é necessário `solc` no PATH.

`make local.test` executa os testes de preparação de seeds, controller, serviço,
identidade, mapper, handlers Solidity e configuração, sem serviços externos.
Esses checks não são uma revisão independente nem demonstram cooperação remota.
O smoke confirma o fluxo local; ainda não comprova uso do PostgreSQL, eficiência
do compartilhamento ou detecção garantida de vulnerabilidades. S04 e S06–S14
acrescentarão os testes específicos de conexão, publicação, consulta e concorrência.

## Encerramento e diagnóstico

Interromper o Dogefuzz com Ctrl+C ou encerrar a sessão de debug. Para parar
containers: `make local.down`. O comando não usa `-v`: preserva o volume do acervo.
O Geth reinicializa sua cadeia no startup conforme seu entrypoint atual; o SQLite
local não representa retomada garantida dessa cadeia. Nenhum reset automático do
PostgreSQL foi adicionado. A política nova de cancelamento imediato pertence a S04
e ainda não está implementada pelo preparo do ambiente.

Para investigar falhas, usar `make local.logs`, logs do Dogefuzz e os artefatos de
avaliação. Verificar `/ping` somente depois do startup dos auxiliares. Resposta
200 de `/tasks` significa aceitação, não conclusão. Mais detalhes e verificações
efetivamente realizadas estão em [RELATORIO.md](RELATORIO.md).

## Critérios de aceite

- Configuração de runtime isolada e fontes locais para as dependências.
- Perfis de execução/debug e testes acessíveis no VS Code e terminal.
- Chamada real preparada com contrato do dataset e evidências por taskId.
- Limitações e diferença entre preparo de ambiente e implementação registradas.
