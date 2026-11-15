# Change tracking e change data capture

> **Habilidades cobradas:** Implementar o rastreamento de dados de mudança
> **Tempo de leitura:** ~10 min

## Conceito central
Os aplicativos muitas vezes precisam saber **O que mudou** em uma tabela desde a última vez. SQL Server e Azure SQL têm dois recursos internos para isso:
- **Change Tracking (CT)**: luz. Respostas " **quais linhas** mudou?"
- **Change Data Capture (CDC)**: detalhado. Respostas " **O que exatamente** mudou, com valores antigos e novos?"

Pense em um registro escolar. Change tracking é uma nota pegajosa dizendo "Rows 4 e 9 mudou". CDC é um diário completo: "Row 4: grade passou de B para A às 10:02, então A para A + às 11:15."

Ambos substituem gatilhos caseiros e colunas timestamp. Você pode ativar ambos no mesmo banco de dados.

## Compare em um relance

| | **Change Tracking** | **Change Data Capture** |
|--|---------------------|-------------------------|
| Como | **síncron**: registrado na mesma transação que a mudança | **Assíncrono**: um processo de captura lê o **log de transação** mais tarde |
| O que é armazenado | **Chave primária** de linhas alteradas + operação (inserir/atualizar/excluir) + opcionalmente quais colunas foram alteradas | **Valores das colunas completas**: linhas inseridas e excluídas, e valores **antes e depois** de atualizações |
| História | **Apenas mudança líquida** (último estado) | Mudança **Cada** |
| Latest data | Volte para a tabela de origem | Na tabela de mudanças |
| Chave primária necessária? | **Sim** | **Não** (apenas para consultas de "mudanças líquidas") |
| Alterações visíveis | Imediatamente após o commit | Após um breve atraso |
| Retenção padrão | **2 dias** | **3 dias** (4,320 minutos) |
| Acima | Baixa | Mais alto: leitura de log, mais armazenamento, pode **Segure o log** |
| Uso típico | Aplicativos **Sincronização**: sincronização móvel/offline, atualização de cache, sincronização bidirecional com verificações de conflito | **ETL**: carregar cada mudança em um data warehouse |

**Exemplo (CT):** Um aplicativo de driver de entrega funciona offline. Quando ele se reconecta, ele pergunta "quais ordens mudaram desde a versão 1050?" e baixa apenas essas linhas.

**Exemplo (CDC):** Um ETL job noturno carrega cada inserção, atualização e exclusão da tabela `Orders` da loja no data warehouse, incluindo o preço antigo antes de cada alteração.

## Onde eles trabalham

| Plataforma | Change Tracking | CDC |
|----------|-----------------|-----|
| SQL Server (incl. Azure VM) | Sim | Sim, edição **Padrão ou Enterprise**. Usa **SQL Server Agent** jobs |
| Azure SQL Managed Instance | Sim | Sim, usa **SQL Agent** jobs |
| Azure SQL Database | Sim | Sim: **Nível vCore**; **DTU S3 e acima apenas** (não básico, S0, S1, S2). Usa um **Agendador CDC** embutido (sem SQL Agent) |

### CDC em Azure SQL Database
- O programador corre **captura a cada 20 segundos** e **Limpeza a cada hora**. Você **não pode mudar** estes tempos.
- Você ainda pode sintonizar `maxtrans` e `maxscans` com `sys.sp_cdc_change_job`. A tabela job `cdc.cdc_jobs` está no **esquema cdc** (não `msdb`).
- Para habilitá-lo, você precisa de **db_owner**. (Em SQL Server e MI, `sp_cdc_enable_db` precisa de **sysadmin**.)
- A banco de dados **serverless pausado** não executa CDC.
- CDC desliga o truncamento de log agressivo do ADR, então **uso do log do relógio**. Considere mais vCores ou **Hyperscale**.
- Depois de um failover, o CDC continua automaticamente no novo primário.
- Monitor com **`sys.dm_cdc_log_scan_sessions`** e **`sys.dm_cdc_errors`**.

## Ligue Change Tracking
Ligue-o para o **banco de dados primeiro**, então para cada **tabela**.

```sql
ALTER DATABASE SalesDB SET ALLOW_SNAPSHOT_ISOLATION ON;   -- recommended
ALTER DATABASE SalesDB
SET CHANGE_TRACKING = ON (CHANGE_RETENTION = 2 DAYS, AUTO_CLEANUP = ON);

ALTER TABLE dbo.Customers
ENABLE CHANGE_TRACKING WITH (TRACK_COLUMNS_UPDATED = ON);   -- table needs a PK
```

- `TRACK_COLUMNS_UPDATED` é **OFF** por padrão (custa armazenamento extra).
- Verifique com `sys.change_tracking_databases` e `sys.change_tracking_tables`.
- Para desligar: desative **tabelas primeiro**, depois o banco de dados.

### Leia as alterações

```sql
SELECT c.CustomerID, c.SYS_CHANGE_OPERATION, cu.Name
FROM CHANGETABLE(CHANGES dbo.Customers, @last_sync_version) AS c
LEFT JOIN dbo.Customers AS cu ON cu.CustomerID = c.CustomerID;
SELECT CHANGE_TRACKING_CURRENT_VERSION();   -- save for next time
```

- Se `@last_sync_version` é mais velho que `CHANGE_TRACKING_MIN_VALID_VERSION(...)`, os dados foram limpos. O cliente deve **Recarregar tudo**.
- `CHANGETABLE(VERSION ...)` dá a versão de uma linha. Ele é usado para **detectar conflitos** em sincronia bidirecional.
- Os leitores precisam de **VIEW CHANGE TRACKING** na tabela e SELECT nas colunas principais.
- `TRUNCATE TABLE` não é rastreado linha por linha. É **resets** rastreamento para a tabela, então os clientes devem re-sincronizar.
- A retenção deve ser **maior do que o intervalo mais longo entre as sincronizações**.

## Ligue Change Data Capture
Novamente: **banco de dados primeiro**, então cada **tabela**.

```sql
EXEC sys.sp_cdc_enable_db;   -- creates the cdc schema and cdc user

EXEC sys.sp_cdc_enable_table
    @source_schema = N'dbo', @source_name = N'Orders',
    @role_name = N'cdc_reader',            -- NULL = no gating role
    @supports_net_changes = 1;             -- needs a PK or unique index
```

- Isso faz um **instância de captura** `dbo_Orders` e uma tabela de mudança **`cdc.dbo_Orders_CT`** no mesmo banco de dados.
- Verifique com `is_cdc_enabled` em `sys.databases` e `is_tracked_by_cdc` em `sys.tables`.
- Uma tabela pode ter no máximo **2 instâncias de captura** (usado para lidar com mudanças de esquema).
- Desligue com `sys.sp_cdc_disable_table` e `sys.sp_cdc_disable_db`.

### Leia as alterações

```sql
DECLARE @from BINARY(10) = sys.fn_cdc_get_min_lsn('dbo_Orders'),
        @to   BINARY(10) = sys.fn_cdc_get_max_lsn();
SELECT * FROM cdc.fn_cdc_get_all_changes_dbo_Orders(@from, @to, N'all update old');
SELECT * FROM cdc.fn_cdc_get_net_changes_dbo_Orders(@from, @to, N'all');
```

- **Todas as alterações** Cada mudança retorna. **Mudanças líquidas** retorna apenas o estado final de cada linha (necessidades `@supports_net_changes = 1`).
- `__$operation`: **1** = delete, **2** = insert, **3** = update (before), **4** = update (after).
- Converter vezes em LSNs (números de sequência de log: posições no log) com `sys.fn_cdc_map_time_to_lsn`.

### Limpeza e o log
- SQL Server / MI: uma limpeza **Agente job** funciona diariamente em 2 AM e mantém os minutos 4,320 por padrão. Mude-o com `sys.sp_cdc_change_job @job_type = N'cleanup', @retention = 2880;`, depois pare e inicie o job.
- **O log não pode ser truncado** até CDC já leu as mudanças, mesmo em SIMPLE recovery. Uma captura parada job = a **crescente log**.
- Restaurar um banco de dados CDC para **outro servidor** desliga CDC, a menos que você use **`RESTORE ... WITH KEEP_CDC`**.

## Qual o recurso para qual pergunta?

| Recurso | Respostas | Valores antigos? | Quem o fez? | Tamper-proof? |
|---------|---------|-------------|-------------|---------------|
| Change Tracking | Que linhas mudaram desde a minha última sincronização? | Não | Não | Não |
| CDC | Cada inserção/atualização/exclusão, com valores? | Sim, antes e depois | Não | Não |
| Tabelas temporais | Como era essa linha no tempo X? | Sim, em uma tabela de história | Não | Não |
| Auditoria | Quem fez o quê e quando (incluindo SELECT e logins)? | Não | **Sim** | Não |
| Ledger | Posso provar que nada foi adulterado? | Sim (updatable ledger) | Sim | **Sim** |

**Exemplo:** Uma loja quer ver qual foi o preço de um produto em março passado → **tabela temporal**. Ele quer saber qual funcionário executou uma consulta sobre os salários → **Auditoria**.

## Para lembrar na prova
- "Sync com **overhead mínima**, que as linhas mudaram" → **Change Tracking**.
- "Levar **Cada mudança com valores de coluna** em um armazém" → **CDC**.
- CT precisa de um **chave primária**; CDC não (exceto para mudanças líquidas).
- Padrão: CT **2 dias**; CDC **3 dias**.
- Ativar no nível **banco de dados**, em seguida, o nível **tabela**. Para CT, desative **tabelas antes** a banco de dados.
- CDC em Azure SQL Database: **vCore em qualquer nível** ou **DTU S3+**. Não Básico/ S0 / S1 / S2.
- CDC em SQL Server / MI usa **SQL Server Agent**. " CDC parou de encher" → verifique o agente. Azure SQL Database usa um agendador embutido (20 s captura, limpeza horária, fixo).
- "Log continua crescendo em um banco de dados CDC" → a captura é interrompida ou para trás.
- CDC em SQL Server precisa de edição **Padrão ou Enterprise**.
- CT não mantém valores antigos ou o nome do usuário. Para a história → **Tabelas temporais**; para quem → **Auditoria**; para a prova → **livro-razão**.

## Teste rápido
1. Você tenta ativar o Change Tracking em uma tabela e ele falha. Qual é a causa provável?
   <details><summary>Resposta</summary>A tabela não tem chave primária. Change Tracking requer uma.</details>
2. Seu Azure SQL Database está no nível S1 e você precisa de CDC. O que você deve fazer?
   <details><summary>Resposta</summary>Escala para S3 ou superior, ou passar para um nível vCore. CDC não é suportado abaixo de S3 no modelo DTU.</details>
3. Em um Managed Instance, as tabelas de mudança CDC pararam de obter novas linhas. O que você verifica primeiro?
   <details><summary>Resposta</summary>Isso SQL Server Agent está em execução, porque a captura CDC é executada como um agente job em MI e SQL Server.</details>
