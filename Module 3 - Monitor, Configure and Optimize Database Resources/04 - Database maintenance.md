# Manutenção de bancos de dados

> **Habilidades cobradas:** Implementar tarefas de manutenção de índice, Implementar tarefas de manutenção estatística, Implementar verificações de integridade do banco de dados, Implementar configuração com escopo de banco de dados
> **Tempo de leitura:** ~10 min

## Conceito central
Os dados mudam todos os dias. Os índices ficam confusos, as estatísticas envelhecem e muito raramente uma página fica danificada. A manutenção regular mantém o banco de dados rápido e saudável. A configuração com escopo de banco de dados permite ajustar as configurações de um banco de dados sem tocar nos outros no mesmo servidor.

Imagine uma biblioteca. Livros voltam para a prateleira errada (índices fragmentados). O catálogo diz que há 10 cópias, quando na verdade há 50 (estatísticas antigas). Às vezes, uma página rasga (corrupção). O bibliotecário organiza as prateleiras, atualiza o catálogo e verifica se há danos.

## Manutenção de índice: reorganizar ou reconstruir
**Fragmentação** significa que as páginas de índice não estão mais em ordem ou estão meio vazias. SQL Server lê mais páginas do que o necessário.

Encontre fragmentação com:
- `sys.dm_db_index_physical_stats` – índices normais (rowstore).
- `sys.dm_db_column_store_row_group_physical_stats` – índices columnstore.

| Fragmentação (regra de ouro) | Ação | Notas |
|-------------------------------|--------|-------|
| 5% – 30% | **REORGANIZE** | Sempre **online**. Luz. O **E** atualiza as estatísticas. |
| 30% | **REBUILD** | Cria o índice novamente. Pode ser **online** ou offline. **Estatísticas de atualizações** (scan completo). Precisa de mais tempo e espaço. |

```sql
ALTER INDEX IX_Orders_Date ON dbo.Orders REORGANIZE;
ALTER INDEX IX_Orders_Date ON dbo.Orders REBUILD WITH (ONLINE = ON, RESUMABLE = ON);
```

- Os números 5% / 30% vêm do módulo Learn. Os documentos atuais da Microsoft avisam **Não confiar apenas em números fixos**. Meça o benefício. Muitas vezes, o **estatísticas de atualização** dá a mesma velocidade para muito menos trabalho.
- Em **Azure SQL Database e MI**, prefira **REORGANIZE** : **on-line + retomar** As operações de retomada sobrevivem a failover.

### Operações de índice resumíveis
- Você pode **PAUSE** e **RESUME** um grande índice de construção ou reconstrução.
- **reconstrução**: SQL Server 2017+. **criar**: SQL Server 2019+. Ambos em Azure SQL Database e MI.
- Funciona apenas com **ONLINE = ON**.
- Comandos: `ALTER INDEX ... PAUSE`, `... RESUME`, `... ABORT`. Tempo de execução limite com `MAX_DURATION = <minutes>`.
- Verificar o progresso em `sys.index_resumable_operations`.
- Configurações com escopo de banco de dados **ELEVATE_ONLINE** e **ELEVATE_RESUMABLE** podem tornar on-line / retomar o padrão.

**Exemplo:** A reconstrução de um índice 500 GB leva 6 horas, mas a janela de manutenção é de apenas 2 horas por noite. O DBA começa com `RESUMABLE = ON, MAX_DURATION = 120`. Ele faz uma pausa após 2 horas e retoma na noite seguinte.

Observação: **compactação automática índice** (`SET AUTOMATIC_INDEX_COMPACTION = ON`, desligado por padrão) em Azure SQL Database e MI mantém recentemente mudadas páginas cheias. Ele faz **E** reduzir a fragmentação ou atualizar estatísticas.

## Estatísticas de manutenção
**Estatísticas** descreve como os valores são distribuídos em uma coluna (por exemplo, quantas ordens por cidade). O **query optimizer** (a parte do SQL Server que escolhe um plano) usa-os para adivinhar a contagem de linhas (**cardinalidade**). Estatísticas antigas → adivinha mal planos lentos →.

- `AUTO_CREATE_STATISTICS` e `AUTO_UPDATE_STATISTICS` são **por defeito**. SQL Server cria e atualiza estatísticas por conta própria.
- Você ainda os atualiza à mão depois de grandes cargas de dados ou quando os planos dão errado.

```sql
UPDATE STATISTICS dbo.Orders;                     -- one table
UPDATE STATISTICS dbo.Orders WITH FULLSCAN;       -- read every row
EXEC sp_updatestats;                              -- whole database
CREATE STATISTICS st_City ON dbo.Customers (City);
```

Verifique quando as estatísticas foram atualizadas pela última vez:
```sql
SELECT s.name, sp.last_updated, sp.rows, sp.rows_sampled, sp.modification_counter
FROM sys.stats AS s
CROSS APPLY sys.dm_db_stats_properties(s.object_id, s.stats_id) AS sp;
```

**Crie estatísticas à mão quando:**
- Uma consulta filtra várias colunas que nenhum índice cobre.
- A consulta lê um pequeno subconjunto de dados, ou o plano avisa sobre estatísticas ausentes.
- O Database Engine Tuning Advisor sugere isso.

**Exemplo:** Todas as noites uma escola carrega 2 milhões de novos resultados do exame. Na manhã seguinte, um relatório é lento porque o otimizador ainda acha que a tabela é pequena. Adicionando `UPDATE STATISTICS` ao final da carga job corrige-o.

## Verificações de integridade da banco de dados (DBCC CHECKDB)
**DBCC CHECKDB** verifica toda uma banco de dados para **corrupção** (páginas danificadas ou links quebrados entre tabelas e índices).

```sql
DBCC CHECKDB ('SalesDb') WITH NO_INFOMSGS;
```

- **SQL Server em um VM:** este é **job**. Agende-o com uma tarefa **SQL Server Agent job** ou **plano de manutenção** ("Verificar Integridade do Banco de Dados").
- O Azure **Azure SQL Database e Managed Instance:** executa o **a integridade verifica automaticamente**, juntamente com a detecção de gravação perdida e reparo automático da página. Você ainda pode executar o DBCC CHECKDB sozinho.
- CHECKDB usa um monte de CPU e I/O. Executá-lo **off-horas** (por exemplo, nos fins de semana).
- Mantenha backups **maior do que a diferença entre os controlos**. Se a corrupção for encontrada, você precisa de um backup antes de começar.
- A backup só é comprovada boa após um **restauração do teste + DBCC CHECKDB**. `RESTORE VERIFYONLY` por si só não prova que os dados são consistentes.
- Do **E** bancos de dados como uma tarefa de rotina; encolher causa fragmentação.

**Exemplo:** Um hospital funciona SQL Server em um VM. Um agente semanal job executa CHECKDB no domingo em 2 a.m. e mantém 2 semanas de backups, então um backup limpo sempre existe se a corrupção aparecer.

## Automating manutenção
A manutenção do cronograma com **SQL Server Agent** em um VM ou Managed Instance. Azure SQL Database tem **Agente no**, então use **elastic jobs** ou **Azure Automation runbooks** (consulte o módulo 4). Em Azure SQL Database e MI, backups e verificações de integridade são feitas pela plataforma, mas **A manutenção de estatísticas e índice ainda é o seu job**.

## Configuração de banco de dados
Versões mais antigas tinham muitas configurações apenas no nível **instância** (servidor inteiro). Agora muitos podem ser definidos **por banco de dados**. Isso ajuda quando um servidor hospeda muitos aplicativos diferentes.

Duas opções de nível de banco de dados:
- `ALTER DATABASE` – modelo de recuperação, automatic tuning, auto estatísticas, Query Store, snapshot isolation (SI e RCSI), nível de compatibilidade.
- `ALTER DATABASE SCOPED CONFIGURATION` – opções de processamento de consultas.

| Opção (configuração com escopo) | O que faz |
|-------------------------------|--------------|
| `MAXDOP` | Max núcleos por consulta neste banco de dados, seja qual for a configuração do servidor |
| `LEGACY_CARDINALITY_ESTIMATION` | Usa o modelo antigo de estimativa de linha; ajuda alguns aplicativos antigos |
| `PARAMETER_SNIFFING` | Ligar ou desligar o parameter sniffing |
| `LAST_QUERY_PLAN_STATS` | Mantém o último plano real para cada consulta |
| `OPTIMIZE_FOR_AD_HOC_WORKLOADS` | Armazena apenas um pequeno esboço de plano para consultas de uso único |
| `LIGHTWEIGHT_QUERY_PROFILING` | Coleta de dados do plano de tempo de execução |
| `ELEVATE_ONLINE` / `ELEVATE_RESUMABLE` | Tornar as operações de índice on-line / retomar por padrão |
| `DOP_FEEDBACK` | Comentários sobre Turn DOP (lição 5) |

```sql
ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP = 4;
ALTER DATABASE SCOPED CONFIGURATION SET LEGACY_CARDINALITY_ESTIMATION = ON;
ALTER DATABASE SCOPED CONFIGURATION CLEAR PROCEDURE_CACHE;
```

### Nível de compatibilidade
- Cada banco de dados tem um **Nível de compatibilidade**, por exemplo 120, 140, 150, 160, 170. Ele decide como o otimizador se comporta.
- Níveis mais novos ativam as características do **Intelligent Query Processing** (lição 5).
- Default para bancos de dados **Novo**: **170** em Azure SQL Database e SQL Server 2025. Managed Instance: **160** com a política de atualização SQL Server 2022, **170** com a política SQL Server 2025 / Always-up-to-date.
- Restaurado ou migrado bancos de dados **Mantenha seu velho nível**. Teste, em seguida, elevá-lo.

```sql
ALTER DATABASE SalesDb SET COMPATIBILITY_LEVEL = 160;
```

**Exemplo:** Uma empresa de hospedagem executa 20 bancos de dados de clientes em um servidor. Um aplicativo antigo fica mais lento após uma atualização. O DBA liga o `LEGACY_CARDINALITY_ESTIMATION` **Apenas para esse banco de dados**. Os outros 19 mantêm o novo comportamento.

## Para lembrar na prova
- **REORGANIZE**: sempre online, leve, faz **E** atualizar estatísticas. **REBUILD**: atualizações estatísticas, on-line ou off-line.
- Regra de ouro: 5–30% reorganizar, sobre 30% reconstruir (mas medir; atualizações estatísticas são muitas vezes o suficiente).
- **Resumível** precisa de **ONLINE = ON**; use PAUSE / RESUME / ABORT e MAX_DURATION.
- Fragmentação DMV: `sys.dm_db_index_physical_stats`. Estatísticas DMF: `sys.dm_db_stats_properties`.
- **DBCC CHECKDB** = corruption check. On a VM você agendá-lo; Azure SQL DB e MI Faça isso automaticamente.
- Azure SQL Database tem **SQL Agent** → elastic jobs ou Automation runbooks.
- `ALTER DATABASE SCOPED CONFIGURATION` = per-database MAXDOP, legado CE, parameter sniffing, último plano de consulta estatísticas.
- Aumente o **Nível de compatibilidade** para obter recursos IQP mais recentes; os bancos de dados migrados mantêm seu nível antigo.

## Teste rápido
1. Um índice é 12% fragmentado e deve permanecer disponível para os usuários. Qual ação?
   <details><summary>Resposta</summary>ALTER INDEX... REORGANIZE. É sempre online e se adequa à fragmentação leve a média.</details>
2. Você precisa verificar se há corrupção no SQL Server em uma Azure VM toda semana. O que você faz?
   <details><summary>Resposta</summary>DBCC CHECKDB com um SQL Server Agent job ou um plano de manutenção. Em um VM a plataforma não faz isso por você.</details>
3. Um banco de dados em um servidor compartilhado precisa do antigo cardinality estimator, mas os outros não devem mudar. Como?
   <details><summary>Resposta</summary>ALTER DATABASE SCOPED CONFIGURATION SET LEGACY_CARDINALITY_ESTIMATION = ON apenas nessa banco de dados.</details>
