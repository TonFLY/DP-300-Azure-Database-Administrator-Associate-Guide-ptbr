# Query Store

> **Habilidades cobradas:** Configurar Query Store, Monitorar usando Query Store
> **Tempo de leitura:** ~8 min

## Conceito central
Sem Query Store, SQL Server mantém apenas o plano **corrente** de cada consulta na memória, e ele é perdido no reinício. **Query Store** é um gravador dentro do banco de dados. Ele salva o texto da consulta, todos os planos usados, números de tempo de execução e esperas, e os mantém **após reinícios**. Você pode ver quando uma consulta ficou mais lenta e **força** o plano antigo e mais rápido.

Pense na caixa preta de um voo. Depois de um problema, você repete o que aconteceu e vê exatamente quando as coisas deram errado.

## O que o Query Store salva
| Loja | Sumário |
|-------|----------|
| **Plan loja** | Texto de consulta e planos de execução, incluindo planos forçados |
| **Runtime stats loja** | CPU, duração, leituras, contagem de linhas por plano por intervalo de tempo |
| **loja Wait stats** | Esperas por consulta (I/O, fechaduras, memória...) |

- Os dados são coletados quando uma consulta é executada pela primeira vez, recompiles e em cada execução.
- Está escrito no disco **assíncrona** (no fundo), então o custo é pequeno.

## Ligando-o
- **Por padrão** em bancos de dados Azure SQL Database, Azure SQL Managed Instance e **Novo** em SQL Server 2022+.
- In Azure SQL Database único e bancos de dados agrupados ele **não pode ser desligado**.
- Em SQL Server 2016–2019 Vira-o sobre ti mesmo:

```sql
ALTER DATABASE SalesDb SET QUERY_STORE = ON (OPERATION_MODE = READ_WRITE);
```

## Modos de operação
| Modo | Significado |
|------|---------|
| **READ_WRITE** | Normal: coleta novos dados |
| **READ_ONLY** | Para de coletar; você ainda pode ler dados antigos |
| **OFF** | Deficientes |

**Por que o Query Store muda para o READ_ONLY por si só?**
- Alcançou o **MAX_STORAGE_SIZE_MB** (mais comum).
- Fix: aumentar o tamanho máximo, ativar a limpeza baseada em tamanho, diminuir o limite de consulta obsoleto ou capturar menos consultas. Em seguida, defina-o de volta para READ_WRITE.
- Verifique o modo atual e desejado em `sys.database_query_store_options` (`actual_state_desc`, `desired_state_desc`, `readonly_reason`).

## Configurações de chave (padrão SQL Server 2019+ / Azure SQL DB)
| Configuração | Padrão | Significado |
|---------|---------|---------|
| `OPERATION_MODE` | READ_WRITE | Recolher ou não |
| `QUERY_CAPTURE_MODE` | **AUTO** (era ALL em 2016/2017) | AUTO pula consultas pequenas e raras; ALL pega tudo; NONE não captura novas consultas; CUSTOM permite que você defina regras |
| `MAX_STORAGE_SIZE_MB` | **1000** (era 100) | Tamanho máximo; quando → READ_ONLY completo |
| `SIZE_BASED_CLEANUP_MODE` | AUTO | Apaga dados mais antigos à medida que se aproxima do limite |
| `STALE_QUERY_THRESHOLD_DAYS` | **30** | Quanto tempo para manter os dados |
| `DATA_FLUSH_INTERVAL_SECONDS` | **900** (15 min) | Com que frequência os dados de memória são gravados no disco |
| `INTERVAL_LENGTH_MINUTES` | **60** | Tamanho de cada intervalo de tempo das estatísticas |

```sql
ALTER DATABASE SalesDb SET QUERY_STORE (
    MAX_STORAGE_SIZE_MB = 2048,
    QUERY_CAPTURE_MODE = AUTO,
    SIZE_BASED_CLEANUP_MODE = AUTO,
    CLEANUP_POLICY = (STALE_QUERY_THRESHOLD_DAYS = 30));
```

**Armadilha do exame:** o **intervalo de dados flush** controla a frequência com que os dados são gravados no disco. O **intervalo de recolha de estatísticas** (`INTERVAL_LENGTH_MINUTES`) apenas controla como os números de tempo de execução são agrupados. Perguntas sobre o ponto de pressão de gravação para o **intervalo de dados flush**. Perguntas sobre o Query Store tornando-se read-only porque é ponto completo para **tamanho máximo e limpeza**.

**Exemplo:** Um banco de dados de loja ocupado Query Store continua READ_ONLY A cada duas semanas. `readonly_reason` mostra que está cheio. O DBA raises MAX_STORAGE_SIZE_MB até 2048 e sets SIZE_BASED_CLEANUP_MODE = AUTO. Permanece em READ_WRITE.

**Query Store para secundários legíveis** (preview): always on em Azure SQL Database e MI (política sempre atualizada); opt-in em SQL Server 2025 com `ALTER DATABASE ... FOR SECONDARY SET QUERY_STORE = ON`.

## Relatórios integrados (banco de dados SSMS → → Query Store)
| Relatório | Use-o para |
|--------|-----------|
| **Consultas Regressadas** | Encontre consultas que ficaram mais lentas; compare planos antigos e novos; **força** o plano antigo |
| **Consumo global de recursos** | Ver CPU total, duração, lê ao longo do tempo e perfurar a causa |
| **Top Resource Consumir Consultas** | Encontre as 25 consultas mais caras |
| **Queries com planos forçados** | Ver e **unforce** planos forçados |
| **Queries com alta variação** | Encontre consultas que às vezes são rápidas, às vezes lentas (muitas vezes parameter sniffing) |
| **Query Wait Estatísticas** | Veja as categorias de espera e as consultas que as causam |
| **Consultas Rastreadas** | Siga a história de uma consulta pelo seu ID |

## Forçar e abrir um plano
```sql
EXEC sp_query_store_force_plan   @query_id = 73, @plan_id = 79;
EXEC sp_query_store_unforce_plan @query_id = 73, @plan_id = 79;
```

**Exemplo:** Após uma atualização estatística, o procedimento de fatura mensal leva 12 segundos em vez de 1. No **Consultas Regressadas**, o DBA vê o plano 79 (antigo, 1 segundo) e o plano 85 (novo, 12 segundos). Eles forçam o plano 79. O procedimento é rápido novamente enquanto procuram a causa real.

Automatic tuning (lição 5) pode fazer a mesma coisa por você com **FORCE_LAST_GOOD_PLAN**.

## Query Store hints
Às vezes, você não pode alterar o texto da consulta, como em um aplicativo de terceiros. Com **Query Store hints**, você aplica um hint a uma consulta **sem mudar seu código**. O recurso está disponível em Azure SQL Database, MI e SQL Server 2022+.

```sql
-- 1. find the query_id
SELECT q.query_id, qt.query_sql_text
FROM sys.query_store_query_text AS qt
JOIN sys.query_store_query AS q ON qt.query_text_id = q.query_text_id
WHERE qt.query_sql_text LIKE N'%ORDER BY CustomerName DESC%';
-- 2. add the hint
EXEC sys.sp_query_store_set_hints @query_id = 42, @query_hints = N'OPTION(RECOMPILE, MAXDOP 1)';
```
- Veja dicas em `sys.query_store_query_hints`. Remova com `sys.sp_query_store_clear_hints @query_id = 42`.
- As dicas sobrevivem a reinícios e failovers.
- Eles **sobreposição** dicas escritas na consulta e **guias de plano**.

**Exemplo:** O aplicativo de faturamento de um fornecedor executa uma consulta codificada que sofre de parameter sniffing. A empresa pode não editar o aplicativo. O DBA adiciona uma dica Query Store `OPTION(RECOMPILE)` a esse query_id. O problema desaparece e nenhum código mudou.

## Vistas de catálogo para saber
| Ver | Mostra |
|------|-------|
| `sys.database_query_store_options` | Configurações atuais e estado |
| `sys.query_store_query` / `sys.query_store_query_text` | Consultas e seu texto |
| `sys.query_store_plan` | Planos, incluindo `is_forced_plan` |
| `sys.query_store_runtime_stats` | Números de tempo de execução por intervalo |
| `sys.query_store_wait_stats` | Esperas por plano |

## Para lembrar na prova
- Query Store é **por defeito** em Azure SQL DB, MI e novas bancos de dados SQL Server 2022+. Ligue-o com `SET QUERY_STORE = ON`.
- → completo vai **READ_ONLY**. Corrigir com maior **MAX_STORAGE_SIZE_MB** e **limpeza**.
- Questão de pressão de escrita → **DATA_FLUSH_INTERVAL_SECONDS** (padrão 900), não o intervalo de estatísticas.
- O modo de captura **AUTO** pula consultas triviais; **Todos** captura tudo.
- Encontre consultas mais lentas → **Consultas Regressadas** relatório. Corrigir → **forçar o plano** com `sp_query_store_force_plan`.
- Sometimes fast, sometimes slow → **Queries com alta variação**.
- Não é possível alterar o código do aplicativo → **Query Store hints** (`sp_query_store_set_hints`).
- Automatic plan correction depende de Query Store.

## Teste rápido
1. Query Store em seu banco de dados mostra `actual_state_desc = READ_ONLY` e a razão é tamanho. O que você faz?
   <details><summary>Resposta</summary>Aumente MAX_STORAGE_SIZE_MB e/ou permita a limpeza baseada em tamanho e um limite mais curto e, em seguida, defina OPERATION_MODE de volta para READ_WRITE.</details>
2. Um procedimento tornou-se lento após uma implantação. Você quer o plano antigo de volta imediatamente. O que você usa?
   <details><summary>Resposta</summary>As Consultas Regressadas relatam para encontrar o plano antigo, em seguida, forçá-lo (em SSMS ou com sp_query_store_force_plan).</details>
3. Um aplicativo de terceiros envia uma consulta que você não pode editar e precisa do MAXDOP 1. Como você aplica?
   <details><summary>Resposta</summary>A Query Store dica: sys.sp_query_store_set_hints com N'OPTION(MAXDOP 1) para isso query_id.</details>
