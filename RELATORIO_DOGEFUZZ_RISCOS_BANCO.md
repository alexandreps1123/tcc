# Dogefuzz: riscos e recomendacoes para o banco de dados

Data da analise: 2026-10-04.

## Base da avaliacao

Este documento complementa o
[relatorio de inicializacao](RELATORIO_DOGEFUZZ_INICIALIZACAO_BANCO.md).
Os achados foram obtidos por leitura do codigo. As prioridades representam
impacto potencial e ordem sugerida de tratamento; nao sao evidencias de falhas
observadas em producao. Nenhuma alteracao no componente foi implementada.

## Achados

### 1. Erros dos PRAGMAs nao interrompem a inicializacao

Prioridade: alta.

Em [connection.go](dogefuzz/data/connection.go), linhas 31 a 36, os cinco
comandos `db.Exec()` nao verificam `.Error`. Uma abertura bem-sucedida pode,
portanto, ser seguida de falha de configuracao sem que `NewConnection()` a
retorne. O log de abertura e a migracao nao comprovam os parametros efetivos.

Recomendacao: verificar cada erro e consultar os valores efetivos relevantes.
Definir quais divergencias devem impedir o startup. Validar especialmente a
ordem e o efeito de `page_size` em bancos novos e existentes, em conjunto com
WAL, antes de tratar `32768` como valor garantido.

### 2. Limpeza remove o banco sem fechar a conexao

Prioridade: alta quando a limpeza e utilizada.

`Clean()` chama apenas `os.Remove("<nome>.db")`
([connection.go](dogefuzz/data/connection.go), linha 45). Nao fecha a conexao,
nao trata explicitamente os arquivos auxiliares `.db-wal` e `.db-shm` e nao
coordena a limpeza com usuarios da conexao. `Destroy()` chama essa rotina e
ignora seu erro, inclusive no ambiente de testes.

Recomendacao: separar fechamento de conexao e exclusao dos dados. Na limpeza
de testes, encerrar o uso do banco, fechar a conexao e tratar os arquivos
associados em um diretorio temporario isolado. Propagar os erros. Evitar
adicionar `Destroy()` ao shutdown de producao sem rever seu comportamento,
pois a implementacao atual exclui o arquivo principal.

### 3. O shutdown nao fecha explicitamente o banco

Prioridade: media.

O fluxo em [main.go](dogefuzz/cmd/dogefuzz/main.go), linha 58, termina com
`os.Exit(0)` depois de encerrar outros componentes. Nao ha chamada ao
`Close()` da conexao SQL subjacente. O sistema operacional recupera recursos
do processo, mas o codigo nao estabelece um encerramento coordenado do banco.

Recomendacao: disponibilizar fechamento explicito e executa-lo depois que os
componentes pararem de usar a conexao, preservando os dados. Incluir tratamento
de erros e garantir que a saida do processo nao pule a finalizacao necessaria.

### 4. A localizacao do banco depende do diretorio de trabalho

Prioridade: media.

O caminho e montado diretamente de `DatabaseName` em
[connection.go](dogefuzz/data/connection.go), linha 22. Alterar o diretorio de
execucao pode levar a abertura de outro arquivo com o mesmo nome. `StorageFolder`
nao controla esse caminho. `LoadConfig()` tambem nao valida um nome vazio.

Recomendacao: definir e validar um caminho de banco explicito, registrar seu
caminho absoluto e documentar a persistencia no ambiente de execucao. Validar
permissoes e existencia do diretorio pai antes de abrir o arquivo.

### 5. A inicializacao depende da ordem de criacao do logger

Prioridade: media.

`DbConnection()` passa `e.logger` diretamente a `NewConnection()`
([env.go](dogefuzz/cmd/dogefuzz/env.go), linha 179), que imediatamente chama
`logger.Info()`. `NewEnv()` inicializa apenas `cfg`. Uma chamada direta a
`NewEnv(cfg).DbConnection()` antes de `Logger()` pode usar um logger nulo.

No startup atual, `NewContractPool()` solicita `Logger()` antes dos servicos,
o que atende essa dependencia. O risco surge em outros consumidores do ambiente
ou em mudancas na ordem de construcao.

Recomendacao: obter o logger por `e.Logger()` no proprio `DbConnection()` e
adicionar um teste para acesso ao banco imediatamente apos `NewEnv()`.

### 6. O esquema nao possui historico de migracoes versionado

Prioridade: media conforme a evolucao do projeto.

`Migrate()` usa apenas `AutoMigrate()`
([connection.go](dogefuzz/data/connection.go), linha 54). Nao foram encontrados
scripts de migracao ou controle explicito de versao do esquema. A leitura do
codigo nao permite comprovar como uma base historica atravessa cada alteracao
de modelo ou transformacao de dados.

Recomendacao: adotar migracoes versionadas quando a evolucao exigir transformacao
de dados, renomeacoes ou mudancas destrutivas. Testar a atualizacao a partir de
uma base representativa, alem da criacao de um banco vazio.

### 7. O preenchimento inicial envolve banco, compilador e blockchain

Prioridade: media.

`contractPool.Setup()` processa contratos em sequencia e `deployContract()`
realiza criacao, deploy e atualizacao
([contract_pool.go](dogefuzz/environment/contract_pool.go), linha 34).
Uma falha pode deixar registros parcialmente preparados ou contratos ja
implantados no Geth. Registros existentes nao impedem um novo deploy na
proxima execucao.

Recomendacao: definir a politica de reexecucao do setup, verificar o estado
existente e testar recuperacao apos falhas entre criacao, deploy e atualizacao.
Uma transacao SQLite isolada nao resolve a coordenacao com o efeito externo
do deploy; essa recuperacao precisa fazer parte do fluxo da aplicacao.

## Verificacoes sugeridas

| Verificacao | Evidencia esperada |
| --- | --- |
| Abrir um banco novo em diretorio temporario | Arquivo criado e entidades migradas |
| Reabrir uma base existente | Dados preservados e esquema compativel |
| Consultar PRAGMAs depois da abertura | Valores efetivos registrados e comparados com os solicitados |
| Usar diretorio sem permissao de escrita | Erro identificavel e inicializacao interrompida |
| Chamar `DbConnection()` antes de `Logger()` | Inicializacao sem dependencia implicita de ordem |
| Fechar e limpar banco de teste | Conexao encerrada e arquivos associados tratados |
| Interromper o setup entre deploy e atualizacao | Reexecucao com politica de recuperacao definida |
| Encerrar o processo normalmente | Usuarios da conexao parados e banco fechado sem exclusao |

As verificacoes acima sao propostas para trabalho futuro. Esta atividade
produziu documentacao e conferiu as referencias no codigo, sem executar o
aplicativo, criar bancos ou modificar a implementacao.
