# Design de índices e query tuning

> **Habilidades cobradas:** Identificar e implementar mudanças de índice para consultas, Recomendar modificações de construção de consulta com base no uso de recursos
> **Tempo de leitura:** ~12 min

## Conceito central
A maioria das consultas lentas são corrigidas por um **melhor índice** ou um **consulta melhor escrita**. Bom design de tabela (tipos de dados corretos, o nível certo de normalização) também ajuda. Dicas de consulta são o último recurso. Sempre verifique se uma mudança ajuda o **sistema inteiro**, não apenas uma consulta.

Pense em uma lista telefônica. É classificado por sobrenome (clustered index), então encontrar "Smith" é rápido. Encontrar todos em "Leeds" significa ler todas as páginas, a menos que você adicione uma segunda lista ordenada por cidade (nonclustered index).

## Tipos de índice
### Clustered index
- Ordena e armazena o **linhas de tabela si mesmos** pela chave. Apenas **um** por tabela.
- Uma tabela sem clustered index é um **heap**.
- As chaves boas são **estreito, único e sempre crescente**, por exemplo um `INT IDENTITY` ID.

### Nonclustered index
- A **estrutura separada** com as colunas-chave e um ponteiro para a linha. Muitos por tabela.
- Se a consulta precisar de colunas que não estejam no índice, o SQL Server faz um **Key Lookup** de volta à tabela.
- A **índice de cobertura** cada coluna que a consulta necessita, usando `INCLUDE`:

```sql
CREATE NONCLUSTERED INDEX IX_Products_ProductID
ON dbo.Products (ProductID) INCLUDE (ProductName, Price);
```

### Filtered index
Um índice em **Algumas linhas apenas**, usando `WHERE`. Menor e mais barato de manter.
```sql
CREATE NONCLUSTERED INDEX IX_Employees_Current
ON HR.Employee (BusinessEntityID) WHERE CurrentFlag = 1;
```

### Columnstore índice
- Armazena dados **por coluna** e comprime-os. Lê apenas as colunas necessárias.
- Melhor para **Análise e Data Warehouses** (grande `SUM`, `AVG`, `GROUP BY`). Usa **batch mode** (cerca de 900 linhas de cada vez).
- Cargas pequenas (sob linhas 102,400) vão primeiro para um **loja delta** baseado em linha; o **tuple mover** comprime-o mais tarde. Force-o com `ALTER INDEX ... REORGANIZE WITH (COMPRESS_ALL_ROW_GROUPS = ON)`.
- **Encomendado columnstore** (`... ORDER (col)`) classifica os dados antes de comprimir para melhor pular o segmento. SQL Server 2022+, Azure SQL DB e MI.

```sql
CREATE CLUSTERED COLUMNSTORE INDEX CCI_FactSales ON dbo.FactSales;
```

## Designing and tuning indexes
- Colunas de índice utilizadas em **WHERE**, **JOIN** e **ORDER BY**especialmente as chaves estrangeiras.
- **Ordem da coluna:** colocar colunas **igualdade** (`=`) primeiro, em seguida, **gama** colunas (`>`, `<`, `BETWEEN`).

```sql
-- WHERE CustomerID = 10 AND OrderDate > '2025-01-01'
CREATE INDEX IX_Orders_Cust_Date ON dbo.Orders (CustomerID, OrderDate);
```
- Mantenha as teclas **pequeno**. Integers comparar mais rápido do que cordas longas.
- **Cada índice diminui a velocidade de escrita.** INSERT, UPDATE e DELETE devem mudar cada índice. Use o menor número possível de índices; alargue um existente com INCLUDE em vez de adicionar outro.
- **Contenção de inserção de última página** (`PAGELATCH_EX` em uma chave cada vez maior como IDENTITY sob inserções pesadas): use `OPTIMIZE_FOR_SEQUENTIAL_KEY = ON` (SQL Server 2019 +, Azure SQL; padrão OFF).

### Encontrando mudanças no índice
| Necessidade | Para onde olhar |
|------|---------------|
| Missing indexes | dica verde no plano, `sys.dm_db_missing_index_details`, automatic tuning, Intelligent Insights |
| Como os índices são usados | `sys.dm_db_index_usage_stats` (procura, digitaliza, pesquisa, atualizações) |
| Atividade de índice de baixo nível | `sys.dm_db_index_operational_stats` |
| Consultas caras para indexar | Query Store, Extended Events |

```sql
SELECT OBJECT_NAME(i.object_id) AS tbl, i.name, u.user_seeks, u.user_scans, u.user_lookups, u.user_updates
FROM sys.indexes AS i
JOIN sys.dm_db_index_usage_stats AS u
  ON i.object_id = u.object_id AND i.index_id = u.index_id;
```
- **Poucas buscas / varreduras, mas muitas atualizações** → o índice pode custar mais do que ajuda.
- O **E** deixa cair um índice apenas porque raramente é usado. Pode ser necessário para relatórios de fim de mês ou de fim de ano. Uma opção: criá-lo pouco antes do relatório ser executado.
- Teste novos índices fora da produção primeiro e compare os planos **antes e depois**.

**Exemplo:** A consulta "encontrar aluno por e-mail" de uma escola examina 2 milhões de linhas. `sys.dm_db_missing_index_details` sugere um índice no `Email`, incluindo `FullName`. Depois de criá-lo, o plano mostra um Index Seek e lê a queda das páginas 40,000 para 4.

## Design de tabela para desempenho
### Normalização
**Normalização** significa armazenar cada fato em apenas um lugar. Isso evita dados repetidos e erros de atualização.
- **1NF:** sem grupos de repetição (não `Color1, Color2, Color3`; uma linha por cor). Cada linha tem uma chave primária.
- **2NF:** com uma chave composta, cada coluna não-chave depende da chave **inteiro**.
- **3NF:** nenhuma coluna depende de outra coluna **não-chave** (mover o código do país para uma tabela Países).

| Carga de trabalho | Design |
|----------|--------|
| **OLTP** (muitos pequenos lê e escreve, e.g. um checkout de loja) | **Normalizar**, colunas de chave de índice, transações curtas |
| **OLAP / data warehouse** (grandes relatórios) | **desnormalizar** para reduzir junções; **schema estrela** (uma tabela de fatos + tabelas de dimensões) ou **snowflake esquema** (dimensões divididas em mais tabelas: menos dados repetidos, mais junções); columnstore; particionamento |

**Particionamento** divide uma grande tabela em partes, por exemplo Vendas por ano, então uma consulta por um ano lê apenas essa parte.

### Tipos de dados
- Use o **Tipo correto menor menor**. Smaller rows = more rows per page = less I/O and memory.
- Números em tipos numéricos (`INT`, `DECIMAL(10,2)` por dinheiro), datas em `DATE` / `DATETIME2`, não em `VARCHAR`.
- `UNIQUEIDENTIFIER` é de 16 bytes; use apenas quando necessário. Os tipos nativos **JSON** e **VECTOR** agora existem em Azure SQL DB, MI (novas políticas de atualização) e SQL Server 2025.

### Conversão implícita
Quando dois tipos diferentes são comparados, SQL Server converte um silenciosamente. O **O tipo de precedência mais baixa é convertido** para o superior. INT bate VARCHAR.

```sql
-- PhoneNumber is VARCHAR. The COLUMN is converted to INT on every row -> no seek
SELECT * FROM dbo.Customers WHERE PhoneNumber = 12345;
-- Better: compare with the same type
SELECT * FROM dbo.Customers WHERE PhoneNumber = '12345';
```
O plano mostra um **Aviso** para conversões implícitas. Corrigir combinando o tipo de parâmetro com o tipo de coluna.

## Reescrever consultas com base no uso de recursos
A **SARGable** Consulta (Pesquisar ARGument-able) lets SQL Server usar um index seekEnvolver a coluna em uma função ou cálculo impede isso.

| Bad (scan) | (Procurar possível) |
|------------|----------------------|
| `WHERE LEFT(City, 1) = 'M'` | `WHERE City LIKE 'M%'` |
| `WHERE YEAR(OrderDate) = 2025` | `WHERE OrderDate >= '2025-01-01' AND OrderDate < '2026-01-01'` |
| `WHERE CAST(OrderID AS VARCHAR(10)) = '100'` | `WHERE OrderID = 100` |

Outras reescritas:
| Padrão | Problema | Melhor |
|---------|---------|--------|
| Cursors / `WHILE` loops | Trabalho Row-by-row, alto CPU | **Set-based** `UPDATE` / `INSERT ... SELECT` |
| Funções multi-valorizadas de tabela | Estimativas de linha ruim | **Inline** TVFs (ou confiar na execução intercalada) |
| Funções escalares em todas as linhas | Lento, sem paralelismo | Inline a lógica (ou compat 150 + scalar UDF inlining) |
| Enorme único UPDATE / DELETE | Lock escalation, log grande | lotes menores |

**Acerte a correção para o recurso:**
- **CPU alta** → afinar top CPU consultas, remover código linha-a-linha, adicionar índices, verificar paralelismo.
- **I/O alto/PAGEIOLATCH** → índices para parar de digitalizar, cobrindo índices, menos colunas.
- **Subsídios de memória / derrames** → corrigir estimativas (estatísticas), evitar grandes tipos.
- **Limite da taxa de registro** → lotes menores, menos índice escreve.

## Parameter sniffing
SQL Server constrói um plano usando o **primeiro valor do parâmetro** que vê, em seguida, reutiliza esse plano. Se o primeiro valor foi incomum, o plano pode ser ruim para valores posteriores.

**Exemplo:** `EXEC GetOrders @CustomerID = 42` (cliente 42 tem 10 milhões de pedidos) constrói um plano de digitalização. Mais tarde `@CustomerID = 5` (10 pedidos) reutiliza o plano de digitalização e é lento.

| Correção | Trade-off |
|-----|-----------|
| `OPTION (RECOMPILE)` | Novo plano a cada execução; mais CPU |
| `OPTIMIZE FOR (@p = value)` | Plano construído para um valor típico |
| `OPTIMIZE FOR UNKNOWN` | Usa estatísticas médias, ignora o valor real |
| Recompile apenas para valores incomuns (filial IF) | Mais código |
| **Otimização PSP** (compat 160) | Mantém vários planos automaticamente |
| Forçar um bom plano em Query Store | Plano fixado até não forçoso |

## Query dicas
As dicas forçam a escolha do otimizador. Adicione-os com `OPTION (...)`.
| Sugestão | Efeito |
|------|--------|
| `MAXDOP n` | Núcleos limite para esta consulta |
| `RECOMPILE` | Construa um novo plano a cada corrida |
| `OPTIMIZE FOR (@p = x)` / `OPTIMIZE FOR UNKNOWN` | Escolha qual valor otimizar para |
| `FAST n` | Retornar as primeiras n linhas rapidamente |
| `LOOP JOIN` / `MERGE JOIN` / `HASH JOIN` | Forçar um tipo de junção |
| `USE PLAN N'<xml>'` | Forçar um determinado plano XML |

```sql
SELECT ProductID, SUM(LineTotal)
FROM Sales.SalesOrderDetail
WHERE UnitPrice < 5.00
GROUP BY ProductID
OPTION (MAXDOP 2, RECOMPILE);
```
- Use dicas **raramente**: para correções urgentes ou código que você não pode mudar. Eles impedem o otimizador de se adaptar a novos dados, novos índices e novas versões.
- Corrigir o **causa raiz** primeiro (índice, estatísticas, texto de consulta).
- Se você não puder alterar o código, use **Query Store hints** (lição 7).

## Verifique todo o sistema
Uma mudança que ajuda uma consulta pode prejudicar os outros. Um novo índice acelera um `SELECT`, mas retarda `INSERT` s nessa tabela. Compare planos, Query Store, wait stats, CPU e I/O **antes e depois**, e teste em um servidor **encenação** com uma carga de trabalho semelhante primeiro.

## Para lembrar na prova
- Um **agrupado** index per table; many **nonclustered**. No clustered = **heap**.
- **Key Lookup** → índice de cobertura com **INCLUDE**.
- Pequeno subconjunto de linhas → **índice filtrado**. Análise grande → **columnstore**.
- Ordem da coluna: **igualdade em primeiro lugar, então alcance**.
- Muitos índices lentos escreve. Verifique `sys.dm_db_index_usage_stats` antes de cair; raro ≠ inútil.
- Funções ou conversões em uma coluna na varredura WHERE → **não- SARGable** →. Reescrever.
- **Conversão implícita:** o lado de menor precedência é convertido; VARCHAR coluna vs INT valor converte a coluna.
- OLTP → normalizar. OLAP → desnormalizar, esquema de estrelas, columnstore.
- Parameter sniffing → RECOMPILE, OPTIMIZE FOR (UNKNOWN), PSP, ou forçar um plano.
- As dicas são um último recurso; Query Store hints quando o código não pode mudar.

## Teste rápido
1. Consulta: `WHERE Status = 'Open'`, e apenas 2% de 50 milhões de linhas estão abertas. Qual índice é menor e melhor?
   <details><summary>Resposta</summary>Use um filtered nonclustered index com WHERE Status = 'Open'. Ele só armazena as linhas com status Open.</details>
2. `WHERE YEAR(OrderDate) = 2025` verifica a tabela mesmo que OrderDate tenha um índice. Como você conserta?
   <details><summary>Resposta</summary>Torne a consulta SARGable: WHERE OrderDate >= '2025-01-01' AND OrderDate < '2026-01-01'. A função aplicada à coluna impedia o seek.</details>
3. Uma consulta é rápida para a maioria dos clientes, mas lenta para um cliente muito grande, porque um plano em cache é reutilizado. O que é isso e nomeie uma correção?
   <details><summary>Resposta</summary>Parameter sniffing. Correções: OPTION (RECOMPILE), OPTIMIZE FOR UNKNOWN, PSP otimização em compat 160, ou forçando um plano adequado.</details>
