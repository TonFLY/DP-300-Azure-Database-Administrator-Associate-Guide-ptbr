# Planos de execução e DMVs

> **Habilidades cobradas:** Planos de execução da revisão, identificar problemas de desempenho usando visões de gerenciamento dinâmico (DMVs)
> **Tempo de leitura:** ~9 min

## Conceito central
An **plano de execução** A receita passo a passo SQL Server para executar uma consulta. Lendo ele mostra *Explicação* Uma consulta é lenta. **DMVs** (Visões de gestão dinâmica) são visões internas do sistema que mostram o que está acontecendo dentro SQL Server agora mesmo: consultas em execução, bloqueios, esperas, uso de índice.

Pense em um satnav. O plano é a rota que escolheu. Se ele envia você através de cada pequena rua (a table scan) em vez da auto-estrada (um index seek), a viagem é lenta. DMVs são o mapa de tráfego ao vivo.

## Como SQL Server escolhe um plano
SQL Server usa um **otimizador baseado em custo**. Ele estima o "custo" de possíveis planos e escolhe o mais barato que encontra. Não tenta todos os planos; usa atalhos.

Passos para cada consulta:
1. **Parse** – verifique a sintaxe.
2. **Bind (álgebrizer)** – verifique se existem tabelas e colunas; verifique os tipos de dados.
3. **Verificação de cache do plano** – reutilizar um plano se já estiver no **plano cache** (memória que armazena planos).
4. **Otimizar** – se nenhum plano for armazenado em cache, construa um usando **Estatísticas**, **índices** e o custo de junções, tipos e filtros.
5. **Executar.**

O otimizador depende de **Estatísticas**. Se eles são velhos, os palpites da linha estão errados e o plano é ruim.

**Exemplo:** O otimizador acha que o `WHERE City = 'Leeds'` retorna 100 linhas, então ele usa um index seek mais pesquisas. Realmente ele retorna 2 milhões de linhas. Uma varredura teria sido muito mais rápida. Atualizando estatísticas corrige o palpite.

### Bolsas de memória
Cada plano reserva memória para tipos e hashes (a **concessão memória**).
- Memória **muito grande** → desperdiçada; outras consultas esperam com `RESOURCE_SEMAPHORE`.
- Dados do Grant **muito pequeno** → **derrames em tempdb** (disco), que é lento. O plano mostra um aviso de derramamento.

## Plano estimado vs. plano real
| | Plano estimado | Plano real |
|--|----------------|-------------|
| Executa a consulta? | **Não** | **Sim** |
| Mostra | Linhas e custos estimados | Linhas reais, memória real, avisos, além de estimativas |
| Utilizar quando | Escrever ou verificar uma consulta com segurança | Solução de problemas de uma consulta lenta |
| SSMS | **Ctrl + L** | **Ctrl + M**, então corra |
| T-SQL | `SET SHOWPLAN_ALL ON` / `SET SHOWPLAN_XML ON` | `SET STATISTICS PROFILE ON` / `SET STATISTICS XML ON` |

- O **forma** do plano é quase sempre o mesmo. Apenas o plano real tem números de tempo de execução.
- **Live Query Estatísticas** mostra o plano em tempo real enquanto uma longa consulta é executada.

**Sinal de aviso grande:** **linhas estimadas muito diferentes das linhas reais**. Isso geralmente significa estatísticas antigas ou parameter sniffing.

## Ler um plano
- Leia **da direita para a esquerda, de cima para baixo**. Os dados fluem da direita em direção ao `SELECT` à esquerda.
- Cada ícone é um **operador** (Scan, Seek, Join, Sort...).
- **Custo %** mostra a parcela de cada passo do total.
- **Setas mais grossas** = more rows.
- Passe o mouse para obter uma dica de ferramenta; clique com o botão direito do mouse em → **Propriedades** para obter detalhes completos.

### Operadores para saber
| Operador | Significado | Bom ou mau? |
|----------|---------|--------------|
| **Index Seek** | Salta direto para as linhas necessárias | Geralmente bom |
| **Index Scan / Clustered Index Digitalização / Table Scan** | Lê todo o índice ou tabela | Mau em grandes tabelas quando você precisa de poucas linhas |
| **Key Lookup** (ou RID Lookup em uma pilha) | Volta para a tabela para colunas ausentes do índice | Costly quando repetido muitas vezes; corrigir com um **índice de cobertura** |
| **Loops aninhados / Hash Match / Junte-se** | Tipos de junção: pequenas entradas / grandes não ordenadas (precisa de memória) / grandes ordenadas | Depende da contagem de linhas |
| **Ordenar** | Linhas de pedidos; precisa de memória | Cuidado com os derramamentos |
| **Triângulo de aviso amarelo** | Spill, conversão implícita, estatísticas em falta, índice em falta | Investigar |

**Exemplo:** A consulta "encomendas por cliente" de uma loja leva 8 segundos. O plano mostra um Clustered Index Scan on Orders com 90% do custo e uma dica verde "índice ausente" em `CustomerID`. Depois de adicionar esse índice, o plano mostra um Index Seek e a consulta leva 50 ms.

## Perfil de perfil leve e o último plano real
**Perfil de consulta leve** custa cerca de **2%** e é **por defeito** em SQL Server 2019 + e Azure SQL. Maneiras de ativá-lo em versões mais antigas: rastreamento da bandeira 7412, a dica `USE HINT('QUERY_PLAN_PROFILE')` para uma consulta ou a configuração com escopo `LIGHTWEIGHT_QUERY_PROFILING`.

Para ver o **último plano real** de uma consulta em cache sem executá-lo novamente:
```sql
ALTER DATABASE SCOPED CONFIGURATION SET LAST_QUERY_PLAN_STATS = ON;
SELECT st.text, qps.query_plan
FROM sys.dm_exec_cached_plans AS cp
CROSS APPLY sys.dm_exec_sql_text(cp.plan_handle) AS st
CROSS APPLY sys.dm_exec_query_plan_stats(cp.plan_handle) AS qps;
```
O `sys.dm_exec_query_plan_stats` funciona em SQL Server 2019+, Azure SQL Database e MI. É **opt-in**: ligue o `LAST_QUERY_PLAN_STATS` por banco de dados ou rastreie a bandeira 2451 para todo o servidor.

## Vistas de gestão dinâmicas (DMVs)
- DMVs e DMFs (funções) todos começam com **`sys.dm_`**.
- Eles mostram dados internos ao vivo: consultas em execução, sessões, esperas, bloqueios, planos em cache, uso de índice, uso de recursos.
- A maioria dos dados é reiniciada quando o servidor reiniciar. Para um histórico de longo prazo, use **Query Store**.

### Permissões
| Scope | Permissão |
|-------|------------|
| DMVs com escopo de servidor | `VIEW SERVER STATE` |
| DMVs de banco de dados | `VIEW DATABASE STATE` |

SQL Server 2022+ adiciona permissões mais finas, como `VIEW SERVER PERFORMANCE STATE` e `VIEW DATABASE PERFORMANCE STATE`. Em Azure SQL Database, a função de servidor **`##MS_ServerStateReader##`** dá acesso ao DMV.

### DMVs para saber
| Área | DMV | Mostra |
|------|-----|-------|
| Trabalho em execução | `sys.dm_exec_requests` | Consultas em execução, wait type, sessão de bloqueio |
| | `sys.dm_exec_sessions` | Sessões Conectadas |
| Custo de consulta | `sys.dm_exec_query_stats` | CPU, leituras, duração das consultas em cache |
| | `sys.dm_exec_procedure_stats` | O mesmo, por procedimento armazenado |
| Planos e texto | `sys.dm_exec_cached_plans`, `sys.dm_exec_sql_text`, `sys.dm_exec_query_plan` | Planos em cache e seu texto de consulta |
| Esperas | `sys.dm_os_wait_stats` | Espera desde o início (instance) |
| | `sys.dm_db_wait_stats` | Espera por um Azure SQL database |
| | `sys.dm_exec_session_wait_stats` | Esperas por sessão |
| | `sys.dm_os_waiting_tasks` | Tarefas à espera agora |
| Fechadura | `sys.dm_tran_locks` | correntes fechaduras |
| | `sys.dm_tran_active_transactions` | Transações abertas |
| Indexes | `sys.dm_db_index_usage_stats` | Buscas, exames, pesquisas, atualizações por índice |
| | `sys.dm_db_index_physical_stats` | Fragmentação |
| | `sys.dm_db_missing_index_details` | SQL Server deseja ter |
| Recursos (Azure) | `sys.dm_db_resource_stats` | CPU, I/O, log, memória para um Azure SQL database |
| | `sys.server_resource_stats` | O mesmo para Managed Instance |

- `sys.dm_db_resource_stats` existe **Somente em Azure SQL**.
- Em Azure SQL Database, use banco de dados-scoped `sys.dm_db_wait_stats` estatísticas de espera.

### Encontre as melhores consultas CPU
```sql
SELECT TOP 5 qs.total_worker_time / qs.execution_count AS avg_cpu,
       qs.execution_count, st.text
FROM sys.dm_exec_query_stats AS qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) AS st
ORDER BY qs.total_worker_time DESC;
```

**Exemplo:** A 11 a.m. um aplicativo do banco trava. O DBA roda `sys.dm_exec_requests` e vê 40 sessões esperando com `LCK_M_X`, todas bloqueadas pela sessão 77. `sys.dm_exec_sessions` mostra que a sessão 77 pertence a um usuário que deixou uma transação aberta. (A lição 8 mostra como corrigir o bloqueio.)

## Para lembrar na prova
- **Plano estimado** = does not run the query (Ctrl+L). **Plano real** = runs it and shows real rows (Ctrl+M).
- Leia planos **direita para esquerda**. Scans em grandes tabelas, Key Lookups, derramamentos e avisos são bandeiras vermelhas.
- Grande diferença entre as linhas estimadas e reais → **Estatísticas** ou **parameter sniffing**.
- **Key Lookup** → adicionar um **índice de cobertura** (colunas INCLUDE).
- Last actual plan → `sys.dm_exec_query_plan_stats` com **LAST_QUERY_PLAN_STATS = ON**.
- DMVs começar com `sys.dm_`; precisa de **VIEW SERVER STATE** ou **VIEW DATABASE STATE**.
- Consultas em execução ao vivo → `sys.dm_exec_requests`. Locks → `sys.dm_tran_locks`. Missing indexes → `sys.dm_db_missing_index_details`.
- Azure SQL DB uso de recursos → `sys.dm_db_resource_stats`; MI → `sys.server_resource_stats`.

## Teste rápido
1. Você deve ver o plano de uma consulta pesada sem executá-lo na produção. O que você usa?
   <details><summary>Resposta</summary>estimated execution plan (Ctrl+L ou SET SHOWPLAN_XML ON). Não executa a consulta.</details>
2. Um plano mostra um Index Seek no `ProductID` seguido por milhares de Key Lookups para obter o `Price`. Que correção?
   <details><summary>Resposta</summary>Crie uma cobertura nonclustered index no ProductID com INCLUDE (Preço) para que nenhum lookup seja necessário.</details>
3. Qual a permissão que um usuário precisa para consultar o `sys.dm_db_wait_stats` em um Azure SQL database?
   <details><summary>Resposta</summary>VIEW DATABASE STATE, porque é um DMV com escopo de banco de dados.</details>
