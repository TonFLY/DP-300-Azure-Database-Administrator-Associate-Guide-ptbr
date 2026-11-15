# Automatic tuning e intelligent query processing

> **Habilidades cobradas:** Configurar banco de dados automatic tuning, Identificar casos de uso para recursos intelligent query processing (IQP)
> **Tempo de leitura:** ~9 min

## Conceito central
Dois recursos permitem que o banco de dados corrija alguns problemas de desempenho por si só.
- **Automatic tuning** observa consultas e **Acts**: força um bom plano antigo de volta, ou (em Azure SQL Database) cria e baixa índices.
- **Intelligent Query Processing (IQP)** é um conjunto de maneiras mais inteligentes para executar consultas. A maior parte liga apenas usando um novo **Nível de compatibilidade** suficiente.

Pense em um carro. Automatic tuning é o mecânico que percebe que o motor piorou após uma mudança de peça e coloca a parte antiga de volta. IQP é um motor mais novo que se ajusta enquanto você dirige.

## Automatic tuning
Automatic tuning usa dados do **Query Store** (o gravador de histórico de consultas, lição 7). Query Store deve ser **ON** e no modo **READ_WRITE**.

### 1. Automatic plan correction – FORCE_LAST_GOOD_PLAN
- A A **regressão do plano** acontece quando um novo plano de execução é mais lento do que um mais antigo. Causas: alterações de esquema, mudanças de índice, novas estatísticas, alterações de dados.
- Com **FORCE_LAST_GOOD_PLAN**, SQL Server percebe a regressão e **força o último bom plano conhecido**.
- Em seguida, verifica o resultado. Se o plano forçado for melhor, ele permanece. Caso contrário, é **não forçoso** e um novo plano é compilado.

```sql
ALTER DATABASE CURRENT SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON);
SELECT * FROM sys.dm_db_tuning_recommendations;   -- see recommendations
```

### 2. Automatic index management – CREATE_INDEX e DROP_INDEX
- **Somente Azure SQL Database.**
- **CREATE_INDEX** adiciona índices que ajudariam sua carga de trabalho. Ele espera se o banco de dados estiver ocupado.
- **DROP_INDEX** deixa cair os índices **Não utilizado** (90 dias) e **duplicata**. Nunca deixa cair índices de chave únicos ou primários. No Premium / Business Critical, cai apenas duplicatas.
- Se uma mudança piorar as coisas, o Azure **reverte-lo** automaticamente.
- O histórico de afinação é mantido para **21 dias**.

### Onde funciona
| Plataforma | FORCE_LAST_GOOD_PLAN | CREATE / DROP INDEX |
|----------|----------------------|---------------------|
| SQL Server 2017+ (VM ou on-premises) | Sim | Não |
| Azure SQL Database | Sim | **Sim** |
| Azure SQL Managed Instance | Sim | Não |

### Defaults e como configurar
- **Padrão Azure SQL Database:** FORCE_LAST_GOOD_PLAN = **ON**, CREATE_INDEX = **OFF**, DROP_INDEX = **OFF**.
- Melhores práticas: definir opções no **servidor lógico** e deixar bancos de dados **herdar**.
- Você pode configurá-los no portal (banco de dados ou servidor → **Automatic tuning**) ou com T-SQL:

```sql
ALTER DATABASE CURRENT SET AUTOMATIC_TUNING = INHERIT;
ALTER DATABASE CURRENT SET AUTOMATIC_TUNING (CREATE_INDEX = ON, DROP_INDEX = ON);
SELECT name, desired_state_desc, actual_state_desc, reason_desc
FROM sys.database_automatic_tuning_options;
```

**Exemplo:** Após uma implantação de terça-feira, a consulta "encontrar produto" de uma loja vai de 1 segundo a 12 segundos por causa de um novo plano. Com FORCE_LAST_GOOD_PLAN ligado, o Azure SQL Database detecta a regressão, força o plano antigo e a consulta volta para 1 segundo antes que os clientes se queixam.

**Exemplo 2:** Uma pequena equipe SaaS não tem DBA em tempo integral. Eles ativam **CREATE_INDEX** e **DROP_INDEX** no nível do servidor. O Azure adiciona um índice ausente no `Orders(CustomerID)` e remove um índice duplicado que ninguém usou.

## Intelligent Query Processing (IQP)
IQP é uma família de características que fixam padrões lentos comuns **sem alterar seu código**. Você principalmente os liga levantando o **Nível de compatibilidade**.

```sql
ALTER DATABASE SalesDb SET COMPATIBILITY_LEVEL = 160;
```
O módulo Aprender diz para usar **160** em SQL Server 2022 / Azure SQL para o conjunto completo. Novo padrão Azure SQL databases para **170**.

### As principais características e quando eles ajudam
| Recurso | Problema que resolve | Nível de Compat |
|---------|-------------------|--------------|
| **Adaptive joins** | Tipo de junção errada porque a estimativa da linha estava errada. Escolha **hash join** ou **loops aninhados** no tempo de execução. Batch mode apenas. | 140 |
| **Execução intercalada** | As funções de valor de tabela de multi-declaração (MSTVFs) foram adivinhadas em uma contagem de linhas fixa. Agora SQL Server executa a função primeiro e usa a contagem real. | 140 |
| **Memory grant feedback** | Query tem muita memória (resíduo) ou muito pouco (pulveriza para tempdb). Próximas corridas obter uma quantidade corrigida. | 140 batch mode, 150 row mode, 160 percentil + salvo em Query Store |
| **Table variable deferred compilation** | As variáveis da tabela sempre foram adivinhadas como 1 linha. Agora a contagem real é usada. | 150 |
| **Batch mode on rowstore** | Grandes agregações, tipos e junções em tabelas normais foram processadas linha por linha. Agora em lotes. | 150 |
| **Scalar UDF inlining** | As funções definidas pelo usuário escalar funcionaram uma vez por linha e bloquearam o paralelismo. Agora elas são dobradas na consulta principal. | 150 |
| **Contagem aproximada distinta** | `COUNT(DISTINCT)` é lento em tabelas enormes. `APPROX_COUNT_DISTINCT()` é rápido, cerca de erro 2% com confiança 97%. | Qualquer (função) |
| **Otimização Parameter Sensitive Plan (PSP)** | Um plano em cache ruim para dados distorcidos. Mantém o **vários planos** para uma consulta parametrizada. | 160 |
| **feedback DOP** | Consultas paralelas usando mais núcleos do que úteis. Reduz DOP para consultas repetidas. Precisa de Query Store. | 160 |
| **feedback CE** | Suposições de estimativa erradas para consultas repetidas. Needs Query Store. | 160 |
| **Optimized plan forcing** | Alto custo de compilação dos planos forçados Query Store | Qualquer (SQL Server 2022+, Azure SQL) |
| **Percentil aproximado** | Percentil lento exato: `APPROX_PERCENTILE_CONT/DISC` | 110+ (SQL Server 2022+, Azure SQL) |
| **Otimização opcional do plano de parâmetros (OPPO)** | Planos ruins para padrões `@p IS NULL OR col = @p` | 170 |
| **Feedback CE para expressões** | Aprende a estimar escolhas para repetir expressões | 170 (Azure SQL DB) / 160 (SQL Server 2025) |
| **OPTIMIZED_SP_EXECUTESQL** | Compilar tempestades de `sp_executesql` | Configuração com escopo de banco de dados (SQL Server 2025, Azure SQL DB) |

- **feedback DOP** é **por padrão em SQL Server 2022**: `ALTER DATABASE SCOPED CONFIGURATION SET DOP_FEEDBACK = ON;`
- Ligado **Managed Instance**, DOP feedback, OPPO, CE Feedback para expressões e optimized plan forcing necessidade a **SQL Server 2025** : **Sempre atualizado** Política de atualização.

### Exemplos de casos de uso
- **Memory grant feedback:** Uma consulta noturna continua derramando para tempdb porque pediu muito pouca memória. Após a primeira corrida, a concessão é levantada e os derramamentos param.
- **Scalar UDF inlining:** Uma loja chama `dbo.GetTax(price)` em todas as linhas de uma tabela de 5 milhões de fileiras. Depois de passar para compat 150+, a função é alinhada e a consulta pode ser executada em paralelo.
- **Otimização PSP:** `GetOrders(@CustomerID)` é rápido para pequenos clientes e lento para um grande cliente. PSP mantém planos separados para valores de parâmetros pequenos e grandes.

## Para lembrar na prova
- **FORCE_LAST_GOOD_PLAN** = automatic plan correction; trabalha em SQL Server 2017+, SQL DB e MI.
- **CREATE_INDEX / DROP_INDEX** = **Somente Azure SQL Database**.
- O padrão Azure SQL DB: correção do plano **ON**, índice de criação / queda **OFF**.
- Configure no **servidor** e deixe Os bancos de dados **INHERIT**.
- Automatic tuning precisa de **Query Store ON, READ_WRITE**.
- DMVs: `sys.dm_db_tuning_recommendations` (sugestões), `sys.database_automatic_tuning_options` (configurações).
- A maioria dos recursos do IQP é ativada pelo **Nível de compatibilidade**: 140, 150, 160, 170.
- Tabela variável adivinhada como 1 linha → **table variable deferred compilation**. MSTVF má estimativa → **Execução intercalada**. Tempdb derrama → **memory grant feedback**. Função escalar lenta → **scalar UDF inlining**.
- Valores dos parâmetros de Skewed → **Otimização PSP** (160).

## Teste rápido
1. Você quer que o Azure SQL Managed Instance crie índices ausentes automaticamente. Isso é possível?
   <details><summary>Resposta</summary>No. Automatic index management (CREATE_INDEX/DROP_INDEX) é Azure SQL Database somente. MI apoia FORCE_LAST_GOOD_PLAN.</details>
2. Uma consulta que usa uma variável de tabela com linhas 100,000 obtém um plano ruim porque SQL Server adivinha 1 linha. Qual recurso IQP ajuda e como você o habilita?
   <details><summary>Resposta</summary>Table variable deferred compilation. Defina o nível de compatibilidade para 150 ou superior.</details>
3. Uma consulta continua derramando para tempdb porque sua concessão de memória é muito pequena. Qual recurso IQP corrige isso ao longo do tempo?
   <details><summary>Resposta</summary>Memory grant feedback. Ele ajusta a concessão de memória em execuções posteriores.</details>
