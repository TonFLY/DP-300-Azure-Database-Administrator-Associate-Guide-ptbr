# Estatísticas de bloqueio, bloqueio e espera

> **Habilidades cobradas:** Identificar e resolver o bloqueio de sessão, identificar problemas de desempenho usando visões de gerenciamento dinâmicas (DMVs)
> **Tempo de leitura:** ~11 min

## Conceito central
Quando muitas pessoas usam os mesmos dados, SQL Server usa **fechaduras** para que eles não prejudiquem as mudanças uns dos outros. Se uma sessão mantém um bloqueio por um longo tempo, outros devem esperar. Este é **bloqueio**. **Estatísticas de espera** registro *Explicação* consultas esperar (travamentos, disco, memória, CPU). Juntos eles dizem-lhe onde o tempo é perdido.

Pense em um único vestiário de loja. Uma pessoa entra e tranca a porta (fechamento). A fila do lado de fora espera (bloqueio). Se alguém adormecer dentro (uma transação aberta), toda a fila está presa. Uma nota na parede dizendo "as pessoas esperaram 2 horas para o vestiário hoje" é uma estatística de espera.

## Fechamento
- Bloqueios protegem as regras **ACID** (Atomicidade, Consistência, Isolamento, Durabilidade: as transações são tudo ou nada, corretas, separadas e permanentes).
- SQL Server leva bloqueios **automaticamente** quando você lê ou altera dados.
- Tamanhos de bloqueio: **linha** (melhor compartilhamento), **página**, **tabela** (menos compartilhamento).
- **Lock escalation:** se uma declaração leva cerca de bloqueios de linha **5,000**, SQL Server pode trocá-los por um bloqueio de tabela para salvar a memória. Isso pode bloquear muitos usuários.

## Bloqueio e deadlocks
- **Bloqueio:** Sessão A mantém um bloqueio; Sessão B quer os mesmos dados e deve esperar. Bloqueio curto é normal.
- O bloqueio torna-se um problema quando:
  1. Um cadeado é mantido **muito longo** (transação longa).
  2. Um bloqueio é **Nunca liberado**, por exemplo, uma transação sem `COMMIT` ou `ROLLBACK`.
- **Deadlock:** A espera por B, e B espera por A. Nenhum dos dois pode se mover. SQL Server detecta isso e mata uma sessão (o **Vítima deadlock**, erro 1205). O outro continua.

```sql
BEGIN TRANSACTION;
UPDATE dbo.Stock SET Qty = Qty - 1 WHERE ProductID = 5;
COMMIT TRANSACTION;   -- forget this line and row 5 stays locked
```

## Encontrando o bloqueio
```sql
SELECT session_id, blocking_session_id, wait_type, wait_time, status
FROM sys.dm_exec_requests
WHERE blocking_session_id <> 0;
```
- `sys.dm_exec_requests` – quem está bloqueado e por quem (`blocking_session_id`).
- `sys.dm_tran_locks` – quais fechaduras são mantidas ou solicitadas.
- `sys.dm_os_waiting_tasks` – tarefas esperando agora e em quê.
- `sys.dm_tran_active_transactions` / `sys.dm_tran_session_transactions` – transações abertas e há quanto tempo elas estão abertas.
- **Extended Events** (os eventos `blocked_process_report` e deadlock) ou **database watcher** para monitoramento contínuo.

**Encontre o bloqueador de cabeça:** seguir `blocking_session_id` até chegar a uma sessão que não está bloqueada em si.

## Resolver bloqueio
- **Curto prazo:** encontrar o bloqueador de cabeça. Se for seguro, peça ao usuário para se comprometer, ou terminá-lo com `KILL <session_id>` (seu trabalho é revertido).
- **Longo prazo:**
  - Mantenha as transações **curto**. Nunca espere a entrada do usuário dentro de uma transação.
  - Sempre **COMMIT ou ROLLBACK**; lidar com erros para que as transações não permaneçam abertas.
  - Adicione **índices** para que as atualizações toquem menos linhas (uma varredura bloqueia muito mais linhas do que uma busca).
  - Atualização em **lotes menores** para evitar lock escalation.
  - Use o isolamento **remando** (RCSI ou snapshot) para que os leitores não bloqueiem escritores.
  - Tabelas de acesso no **mesma ordem** em todo o código para evitar deadlocks.

**Exemplo:** Cada almoço um aplicativo de armazém congela. `sys.dm_exec_requests` mostra 30 sessões bloqueadas pela sessão 61. Sessão 61 é um relatório que executa `UPDATE` em estoque sem um índice `WHERE`, digitalizando e travando toda a tabela. Adicionando um índice na coluna de filtro e executando a atualização em lotes de linhas 1,000 termina o congelamento.

## Níveis de isolamento
O **isolation level** define quanto uma transação fica protegida da interferência das outras. Aumentar a proteção aumenta o blocking.

| Nível de isolamento | Leituras sujas? | Mantém bloqueios de leitura até o fim? | Usa versões de linha? |
|-----------------|--------------|-----------------------------|--------------------|
| Leia sem compromisso | Sim | Não | Não |
| **Ler Compromissado** (padrão SQL Server) | Não | Não | Não (a menos que o RCSI esteja ligado) |
| Leitura repetida | Não | Sim, para linhas lidas | Não |
| Serializable | Não | Sim, para toda a gama | Não |
| **Comprometimento de leitura snapshot (RCSI)** | Não | Não | Sim |
| **Snapshot** | Não | Nenhuma transação inteira vê uma versão; atualizar conflitos possíveis | Sim |

Um **dirty read** ocorre quando uma transação lê dados que outra transação alterou antes de fazer commit.

```sql
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
ALTER DATABASE SalesDb SET READ_COMMITTED_SNAPSHOT ON;
```

### Versão Row
- Com RCSI : snapshotOs leitores leem um **versão mais antiga comprometida** da fila em vez de esperar por uma fechadura. **Os leitores não bloqueiam escritores, e os escritores não bloqueiam leitores.**
- As versões de linha antigas são mantidas em **temdb**. Com **Accelerated Database Recovery (ADR)**, always on em Azure SQL Database e MI, elas vão para o **loja de versão persistente (PVS)** dentro do banco de dados do usuário.
- **RCSI é ON por padrão em Azure SQL Database** (não em SQL Server ou MI).

### Bloqueamento otimizado
- Always on em **Azure SQL Database** e em MI com a política de atualização Sempre atualizada ou SQL Server 2025. Opt-in em SQL Server 2025 (precisa de ADR).
- **Bloqueio TID:** detém um bloqueio na transação ID em vez de muitos bloqueios de linha e página.
- **Bloqueio após a qualificação (LAQ):** precisa de RCSI; verifica o filtro na versão mais recente comprometida antes de tomar um bloqueio.
- Resultado: menos bloqueio, menos memória de bloqueio, quase nenhum lock escalation.

**Exemplo:** Um aplicativo escolar no SQL Server tem relatórios de professores bloqueados por atualizações de marca durante todo o dia. O DBA liga o **READ_COMMITTED_SNAPSHOT**. Os relatórios agora leem as últimas marcas comprometidas sem esperar, e as atualizações não são mais bloqueadas pelos relatórios.

## Estatísticas de espera
Toda vez que uma tarefa deve esperar, o SQL Server registra o **wait type** e o tempo.
- **Esperas de recursos** – para um bloqueio, memória, disco, etc.
- **fila esperas** – trabalhadores ociosos à espera de trabalho (geralmente inofensivo).
- **Esperas externas** – algo fora do SQL Server, como um servidor vinculado.

Onde vê-los:
- `sys.dm_os_wait_stats` – total de ocorrências desde o início.
- `sys.dm_db_wait_stats` – um Azure SQL database.
- `sys.dm_exec_session_wait_stats` – por sessão.
- **Query Store** wait stats – por plano de consulta.

```sql
SELECT TOP 10 wait_type, wait_time_ms, waiting_tasks_count
FROM sys.dm_os_wait_stats
ORDER BY wait_time_ms DESC;
-- reset the counters (SQL Server):
DBCC SQLPERF('sys.dm_os_wait_stats', CLEAR);
```

### Comum wait types
| Wait type | Significado | Correção provável |
|-----------|---------|-----------|
| **LCK_M_X**, **LCK_M_S** (LCK_M_*) | Waiting for a lock (blocking) | Transações, índices, RCSI |
| **PAGEIOLATCH_SH** | Esperando para ler uma página de dados do **disco** | Armazenamento mais rápido, ou índices para evitar varreduras, mais memória |
| **PAGELATCH_UP / _EX** | Conteúdo em páginas na memória (muitas vezes tempdb páginas de alocação, ou inserções de última página). Sem disco I/O. | Arquivos Tempdb mais iguais; `OPTIMIZE_FOR_SEQUENTIAL_KEY` |
| **RESOURCE_SEMAPHORE** | Esperando um **concessão memória** | Afinar grandes consultas, corrigir estimativas ruins, adicionar memória |
| **SOS_SCHEDULER_YIELD** | Alta pressão **CPU** | Tune CPU - consultas pesadas ou escala acima |
| **CXPACKET** | threads paralelos esperando um pelo outro | MAXDOP mais baixo, limite de custo do aumento paralelismo |
| **WRITELOG** | Esperando por log escreve | Disco de log mais rápido |
| **ASYNC_NETWORK_IO** | Cliente demora a ler resultados | Corrigir o aplicativo ou rede |
| **THREADPOOL** | Sem tópicos de trabalhador livre | Menos pedidos simultâneos, escalar |

**Exemplo:** Os usuários dizem que o aplicativo é lento. A espera superior é `PAGEIOLATCH_SH`. O DBA verifica os planos das principais consultas e encontra grandes varreduras de tabela. Adicionar dois índices corta as leituras do disco e a espera cai.

## Correr ou esperar? Um caminho de solução de problemas simples
1. Check **CPU** (portal ou `sys.dm_db_resource_stats`).
2. **CPU (problema de execução):** encontrar consultas top CPU com Query Store, `sys.dm_exec_query_stats`, `sys.dm_exec_procedure_stats`, `sys.dm_exec_requests`. Sintonizá-los ou escalar.
3. Verifique **CPU (problema de espera):** wait stats e `sys.dm_os_waiting_tasks`. Em seguida, corrija fechaduras, E / O, memória ou escala.

### Esperas específicas do Azure
| Espera / erro | Significado |
|--------------|---------|
| `LOG_RATE_GOVERNOR` | Azure SQL database atingiu seu **limite de escrita log** |
| `POOL_LOG_RATE_GOVERNOR` | Elastic pool atingiu seu limite de log |
| `INSTANCE_LOG_RATE_GOVERNOR` | Limite de log Managed Instance |
| `HADR_SYNC_COMMIT` | Business Critical: esperando por uma réplica para confirmar um commit |
| `HADR_THROTTLE_LOG_RATE_*` | A taxa de registro foi estrangulada porque uma réplica ou geosecundária está atrasada |
| `RBIO_*` | servidor de página Hyperscale lê |
| Erro **10928** | Azure SQL Database atingiu seu **limite de trabalhadores** (a mensagem diz "limite de solicitação") |

Os limites de trabalho e sessão dependem do **nível de serviço e tamanho da computação**. No MI, muitos trabalhadores mostram como o `THREADPOOL` espera.

## Para lembrar na prova
- Localizador de bloqueio: `sys.dm_exec_requests` (`blocking_session_id`) + `sys.dm_tran_locks`.
- Transação aberta sem COMMIT = classic cause of long blocking.
- Deadlock = SQL Server mata um **Vítima** (erro 1205). Capture detalhes com Extended Events.
- **RCSI / snapshot** usar versões de linha: leitores e escritores não bloquear uns aos outros. RCSI é **por padrão em Azure SQL Database**.
- Lock escalation em cerca de **Fechaduras 5,000**; corrigir com lotes e índices menores.
- `PAGEIOLATCH` está ligado a I/O de disco. `PAGELATCH` está ligado à contenção em páginas na memória, muitas vezes no tempdb. Diferencie essas duas esperas.
- `RESOURCE_SEMAPHORE` = memory grant. `SOS_SCHEDULER_YIELD` = CPU. `CXPACKET` Paralelismo.
- `LOG_RATE_GOVERNOR` = Azure log write limit → aumentar ou reduzir as gravações. Erro **10928** = worker limit.

## Teste rápido
1. Leitores em sua banco de dados SQL Server são bloqueados por escritores durante todo o dia. Você quer que os leitores parem de esperar sem alterações de código. O que você habilita?
   <details><summary>Resposta</summary>READ_COMMITTED_SNAPSHOT (RCSI). Os leitores usam versões de linha em vez de esperar por bloqueios.</details>
2. A espera no topo é RESOURCE_SEMAPHORE. O que significa?
   <details><summary>Resposta</summary>As consultas estão esperando por doações de memória. Ajuste grandes consultas, corrija estimativas ruins (estatísticas) ou adicione memória.</details>
3. Qual coluna DMV mostra a sessão que está bloqueando um pedido?
   <details><summary>Resposta</summary>blocking_session_id em sys.dm_exec_requests.</details>
