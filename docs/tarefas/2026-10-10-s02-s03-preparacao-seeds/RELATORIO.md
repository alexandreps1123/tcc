# Relatório — preparação de seeds (S02/S03)

Entrega realizada em 2026-10-10, no escopo fechado desta pasta. Implementados
`SeedCandidate`, adapter bidirecional e identidade estável calculada no cadastro,
com verificações funcionais independentes do acervo. S02 permanece parcial no
planejamento completo: persistência compartilhada, consulta/publicação remotas e
cooperação entre containers não foram implementadas nem demonstradas. A parte
autorizada de S03 foi implementada e validada para métodos ABI ordinários.

## Referências e estado do código

- Referências utilizadas: snapshots 001, 002, 003 e GLOSSARY desta pasta. Os quatro
  SHA-256 foram conferidos e correspondem ao manifesto `REFERENCIAS.md`.
- Componente: `/home/parkus/tcc/dogefuzz`, inicialmente sem alterações em
  `git status --short`. Não foram encontrados `AGENTS.md` no componente ou nos
  diretórios ancestrais consultados. Nenhum documento principal foi modificado.
- Commit inicial: `1aa40f3e238f3ad330201cff8401074651d1c99f`.
- Branch criada: `task/s02-s03-preparacao-seeds`.
- Entrega final: alterações não commitadas nessa branch, sobre o mesmo HEAD;
  identificadas pelos hashes dos arquivos abaixo. Nenhum push foi realizado.
- Ambiente de verificação: Go `go1.26.0 linux/amd64`; dependências existentes do
  `go.mod`, incluindo go-ethereum `v1.11.2`. Nenhuma dependência foi acrescentada.

## Implementação e pontos de integração

`pkg/dto/seed_candidate.go` contém exatamente os oito campos aprovados, com os
respectivos tipos e comentários sobre identidade, argumentos e métricas da
avaliação produtora. Não contém versão, endereço ou referência de origem.

`pkg/seeds/adapter.go` oferece `FromTransaction(identity, tx)` e
`ToTransaction(candidate)`. Copia argumentos, instruções executadas e as três
métricas sem calcular novos ganhos, remapear endereços ou deduplicar entradas.
As listas são copiadas para evitar que o consumidor altere os dados originais;
`nil` e listas vazias são preservados pelo adapter. A conversão para persistência
foi exercitada usando `TransactionMapper.MapDTOToEntity/MapEntityToDTO`, que
continua utilizando Join/Split com `;`. Não foi criado formato persistido novo.
O DTO adaptado fornece os dados de seleção; não é um registro operacional pronto
para gravação. IDs/status/timestamps de transações futuras continuam locais.

`pkg/seedidentity/identity.go` calcula SHA-256 dos bytes da string exata recebida.
`tasksController.Start` calcula o hash antes de chamar `CompileSource` e registra
as assinaturas a partir da ABI já interpretada. A declaração selecionada vem de
`request.ContractName`. As assinaturas são `abi.Method.Sig`, incluindo tipos
canônicos e tuplas; o lookup usa a chave ABI `method.Name`, que distingue as
sobrecargas na biblioteca atual. Não utiliza UUID, endereço, bytecode ou opções
do compilador na chave compartilhada.

`NewContractDTO` e `ContractDTO` recebem `SeedIdentity` com `json:"-"`. O serviço
mantém um `sync.Map` privado, por ID local do contrato, com a identidade imutável
criada no cadastro. `Create` guarda a identidade somente após gravação bem-sucedida;
`Get`, `Find` e `FindByTaskId` reutilizam a mesma identidade. Atualizações locais
não a recalculam. Entities, mappers operacionais e esquema não receberam campos.

### Fluxo inspecionado e dependências do adapter

| Ponto atual | Dados/uso e correspondência |
| --- | --- |
| `controller/tasks.go: Start/storeAvailableMethods/storeMethod` | Recebe fonte exata, seleciona declaração, compila, interpreta ABI e cadastra métodos; integração da identidade neste cadastro. |
| `listener/fuzzer.go: onEvent` | Escolhe função, chama `GenerateInput`, serializa handlers em ordem ABI e cria transações locais. Em erro de geração registra a falha e retorna do evento. |
| `pkg/mapper/transaction.go` | Inputs e instruções via Join/Split `;`; métricas via FormatUint/ParseUint. O adapter entrega esses mesmos campos ao mapper existente. |
| `fuzz/power_schedule.go: RequestSeeds` | Carrega função/contrato/ABI, recupera transações concluídas, ordena, seleciona Inputs, desserializa e completa somente após sucesso insuficiente. |
| `coverage_orderer.go` / greybox | Usa `DeltaCoverage`, preservado integralmente. Ordenação crescente e estável mantida. |
| `distance_orderer.go` / directed_greybox | Usa `CriticalInstructionsHits` e `DeltaMinDistance`; `TargetInstructionsFreq` e `DistanceMap` vêm do contrato consumidor. |
| `distance_coverage_orderer.go` / other_directed_greybox | Usa `len(ExecutedInstructions)`, `DeltaMinDistance` e `CriticalInstructionsHits`; CFG, DistanceMap e TargetInstructionsFreq vêm do consumidor. |
| Três `GenerateInput` greybox | Recuperam candidatos, escolhem seed e executam os mutadores atuais; erros de RequestSeeds são propagados. |
| `listener/execution_analytics.go` | Produz métricas e salva transação/tarefa; publicação futura deve chamar o adapter depois das gravações bem-sucedidas. Nenhuma publicação foi adicionada. |

Nenhum orderer, handler, mutador, listener, query ou caminho de complemento foi
alterado. `FromTransaction` não decide elegibilidade: o chamador de publicação
futuro deverá confirmar conclusão e sucesso das gravações locais.

## Verificações e evidências

Comandos executados a partir de `dogefuzz`. Os comandos finais abaixo retornaram
exit code 0. `GOCACHE` aponta para `/tmp`, diretório gravável deste ambiente.

```bash
GOCACHE=/tmp/dogefuzz-s02-go-cache go test ./fuzz ./service ./controller ./pkg/seedidentity ./pkg/mapper ./pkg/solidity
GOCACHE=/tmp/dogefuzz-s02-go-cache go test -v ./fuzz ./controller ./service ./pkg/seedidentity -run 'Test(Candidate|Supported|RequestSeeds|Identity|ThreeGreybox|StartRegisters|ContractServiceReuses)'
GOCACHE=/tmp/dogefuzz-s02-go-cache go test ./... -run '^$'
GOCACHE=/tmp/dogefuzz-s02-go-cache go test ./api ./config ./data/... ./entities ./environment ./fuzz ./job ./listener ./pkg/bus ./pkg/common ./pkg/distance ./pkg/dto ./pkg/mapper ./pkg/oracle ./service ./topic
git diff --check
```

- `TestCandidateOrderersPreserveCalculationsAndOrder`: mesmos dados, contexto e
  ordem inicial produzem a mesma ordenação nos três orderers. Compara também
  pontuação dirigida, componentes de cobertura/distância/hits e preservação das
  listas. Na estratégia distance_based compara inclusive NaN/Inf sem corrigir
  o algoritmo. Verifica que mutar uma cópia não altera a entrada original.
- `TestSupportedInputsThroughCandidatePersistenceAndHandlers`: 74 casos de
  tipos/valores passando por handlers reais → candidato → adapter → mapper atual
  → desserialização do power schedule. Inclui int/uint de 8 a 256 bits em passos
  de 8, bool, string UTF-8, endereço externo ao contexto disponível, bytes,
  bytes3, array fixo de endereços, slices de uint256/string/bool, slice vazio e
  coleção aninhada uint8[][]. Compara valores, ordem, métricas (incluindo
  MaxUint64) e ExecutedInstructions. Tuplas são verificadas como identidade ABI,
  sem adicionar suporte de handler.
- `TestRequestSeedsErrorsAndSuccessfulComplement`: nas três estratégias, consulta
  vazia ou insuficiente bem-sucedida completa com seeds configuradas; candidato
  inválido, argumentos ausentes e erro de consulta retornam erro sem complemento.
- `TestThreeGreyboxPropagateDeserializationFailure`: as três `GenerateInput`
  propagam ErrInvalidBool e, com candidato válido, executam mutação normalmente.
  O retorno do listener após erro foi conferido no código existente; o pacote
  listener não possui testes e não foi demonstrado com um evento blockchain real.
- `TestIdentityExactSourceAndCanonicalOverloads` e `TestIdentityIgnoresLocalDeployment`:
  fonte/declaração/assinatura iguais dão identidade igual em registros distintos;
  fonte alterada, CRLF/LF, declaração diferente e sobrecargas distinguem chaves.
  Assinaturas incluem `f(uint256)`, `f(address)` e `g((uint256,address)[])`.
  UUID/endereço diferentes não influem; ausência de identidade/método retorna erro.
- `TestStartRegistersExactPreCompilationIdentity`: request com UTF-8/CRLF chega
  intacta ao compilador simulado e ao cadastro; nome selecionado prevalece sobre
  nome devolvido pelo compilador, mantendo assinatura e publicação do evento atual.
- `TestContractServiceReusesRegistrationIdentityAcrossReadsAndUpdates`: mesma
  identidade reaparece nas três leituras após atualização; outro serviço/processo
  não recebe identidade implicitamente reconstruída.
- Compilação/verificação padrão de todos os pacotes passou (`-run '^$'` não executa
  seus testes). Suite operacional passou, incluindo integração SQLite dos quatro
  repositórios, jobs, tópicos, mappers, serviços e oráculos. Resíduos SQLite criados
  por esses testes foram removidos após a conclusão.

A tentativa inicial sem GOCACHE explícito executou os checks então existentes de
mapper/solidity/service com sucesso, mas terminou com erro de permissão ao gravar
`/home/parkus/.cache/go-build/trim.txt`; uma tentativa subsequente também encontrou
arquivos de cache somente leitura. São restrições do ambiente, resolvidas usando
`/tmp`; não são regressões do produto. Nenhuma falha funcional apareceu nos checks
finais. Não foram executados testes externos de solc/Geth/Vandal/reporter nem a
suite integral `go test ./...` com execução de todos os testes: dependem de serviços,
rede ou ferramentas externos. Não há validação de deployment real nesta entrega.

## Checkpoints do escopo fechado

| Checkpoint | Resultado/evidência |
| --- | --- |
| Inspeção e mapeamento de campos | Atendido; tabela de integração acima. |
| Objeto conforme referência | Atendido; oito campos, tipos e comentários, sem versão/origem. |
| Adapter preserva argumentos e seleção | Atendido; testes de dados, métricas, isolamento das cópias e orderers. |
| Identidade igual entre execuções distintas | Atendido em teste de instâncias/registros independentes; sem prova entre containers. |
| Fontes, declarações e sobrecargas distintas | Atendido; testes de identidade e tupla canônica. |
| UUID e endereço independentes | Atendido; teste com dois DTOs locais distintos. |
| Tipos suportados preservam valores/ordem | Atendido para os 74 casos; limitações da representação abaixo. |
| Limitação de `;` registrada | Atendido e reproduzido em teste. |
| Equivalência dos três orderers | Atendido com mesmo contexto e ordem inicial, incluindo métricas não finitas atuais. |
| Erros e complemento mantidos | Atendido funcionalmente até GenerateInput; listener conferido estaticamente e não alterado. |
| Fluxo operacional existente | Atendido nos checks locais pertinentes e compilação geral; sem execução blockchain real. |
| Implementado/validado/pendente distinguido | Atendido neste relatório. |

## Limites e dependências seguintes

1. `;` dentro de argumentos não tem round-trip: `["a;b", "tail"]` torna-se
   `["a", "b", "tail"]`. Uma string vazia isolada também se perde: `[""]`
   resulta em Inputs nil no mapper. Ambos foram reproduzidos, sem escape novo ou
   mudança dos handlers. A desserialização atual pode aceitar argumentos extras;
   não foi criada detecção nova dessas ambiguidades.
2. `distance_based` pode produzir pontuações não finitas pela soma de distâncias
   zero; ordenação coverage_based é crescente; limite de consulta local permanece
   ignorado. Essas dívidas já documentadas não foram corrigidas.
3. A identidade vive no serviço durante o processo. Não há restauração para
   registros antigos/reinício, nem identidade para auxiliares `CreateWithId`.
   Constructor/fallback/receive não entram no lookup compartilhável desta entrega;
   sua política permanece em D05. O fluxo operacional atual continua igual.
4. Não há bloqueio por decisão reservada para a preparação entregue. Armazenamento
   durável de identidade, política de métodos especiais e desenho do acervo são
   dependências futuras. Se exigir alteração de esquema/tabela, será necessária
   decisão explícita sobre campos/localização antes de implementar.
5. Métricas uint64 são reutilizadas como recebidas; o adapter não fabrica avaliação
   para registros incompletos. Diferenças de estado blockchain, saldo, código em
   endereços e resultados entre produtor/consumidor continuam possíveis.

Nenhum compartilhamento funcional foi afirmado. S04–S16, publicação/consulta,
infraestrutura, filtros, retenção e configuração do benchmark permanecem fora
da entrega. Os status globais e as referências congeladas foram preservados.

## Manifesto do código final

Arquivos relativos a `dogefuzz`. Este manifesto identifica o código revisável
sem exigir commit. Hash SHA-256 do manifesto textual (linhas `hash  caminho`,
ordenadas pelo caminho, com LF final): `b8c3bbe1a1d5eca195d8ce30b455557e8e45cc06ddf325b89eff7c53675cda3c`.

```text
a38b40d7320cc2d2000060c293aaf2aef80eec13f6f39626bce736575e22a1c4  controller/seed_identity_test.go
3c25ac75ab747af37ecec1a3fc39a41acff4a02bf56fe0be02f40dc551ef1436  controller/tasks.go
93d94af839d34e3c021b1548303ee5680ddfbe64242b728f8db18011f8490145  fuzz/seed_candidate_test.go
9c93de61350531613e46887b6aa471a3bc263daba5f571d93bccd2e62afcb71a  pkg/dto/contract.go
d13c37c3f1d64ff0655836e7297fb4f305835bcb9d9592cfdf22c8ee5d1b096c  pkg/dto/seed_candidate.go
b4c4a456035930498b97c75810dca89dbe4a4882df1726c032a7744b38c52bad  pkg/seedidentity/identity.go
53f6f95eb2d2b5e8b95a5a3370f279f7c5eb597e4f760c90f74309953f3d7013  pkg/seedidentity/identity_test.go
05eba583a06e8bb29f76d0a0529eb7747d28ff1fe8b43ee344906111348727dd  pkg/seeds/adapter.go
7e9c7a587fa987bdc64eaedf272f1f85d0b435c6ab1817067a8f73ad92b2279e  service/contract.go
9972c6be233fbca4dc4dff54b3be31b4ddb31dbab36ba4805a580161be90e573  service/seed_identity_test.go
```
