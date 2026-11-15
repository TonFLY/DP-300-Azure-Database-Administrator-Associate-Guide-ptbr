# Particionamento, compressão e sharding

> **Habilidades cobradas:** Plano para particionamento de tabela, Configurar particionamento de tabela, Configurar data compression, Recomendar um banco de dados sharding solução
> **Tempo de leitura:** ~13 min

## Conceito central
**Particionamento** corta uma grande tabela em fatias dentro do banco de dados **um**. **Compressão** torna os dados menores no disco. **Sharding** espalha linhas sobre bancos de dados **muitos**.

Imagine uma biblioteca. Com partitioning, há uma prateleira para cada ano dentro da mesma biblioteca. Com compression, usa-se uma fonte menor para caberem mais livros. Com sharding, abrem-se filiais em outras cidades, cada uma com parte dos livros.

## Particionamento: os blocos de construção
| Objeto | O que faz |
|--------|--------------|
| **Partition function** | Define o **valores-limite**. Decide qual partição um valor vai para. |
| **Partition scheme** | Mapas de cada partição para um **filegroup** (um grupo de arquivos de dados). |
| **Coluna de particionamento** | A coluna **um** que decide a partição (por exemplo, `OrderDate`). |
| **Aligned index** | Um índice no mesmo partition scheme que a tabela. Necessário para comutação rápida. |

- **RANGE LEFT** (padrão): o valor limite pertence à partição em seu **esquerda**.
- **RANGE RIGHT**: o valor limite pertence à partição em seu **DIREITO**. Use-o para **datas**, então 1 de março pousa na partição de março.
- **N boundary values = N + 1 partitions.**
- Você não pode particionar em **timestamp**, tipos LOB (`varchar(max)`, `xml`), tipos CLR ou tipos de alias.
- Max **Partições 15,000** por tabela. Não crie milhares sem uma razão.
- Linhas com um valor de particionamento **NULL** vão para a partição mais à esquerda.

## Configurar o particionamento
```sql
CREATE PARTITION FUNCTION pfOrderDate (date)
AS RANGE RIGHT FOR VALUES ('2026-01-01', '2026-02-01', '2026-03-01');

CREATE PARTITION SCHEME psOrderDate
AS PARTITION pfOrderDate ALL TO ([PRIMARY]);   -- Azure SQL Database: PRIMARY only

CREATE TABLE dbo.Orders (OrderID bigint NOT NULL, OrderDate date NOT NULL,
  CONSTRAINT PK_Orders PRIMARY KEY CLUSTERED (OrderDate, OrderID)
) ON psOrderDate (OrderDate);
```
- O índice A **único** deve **Incluir a coluna de particionamento** em sua chave, ou não está alinhado.
- Para um índice **não-único**, o SQL Server adiciona a coluna para você.
- `ALTER TABLE dbo.Orders SET (LOCK_ESCALATION = AUTO);` deixa fechaduras escalar para uma partição, não a tabela inteira.
- `SELECT $PARTITION.pfOrderDate('2026-02-15');` diz-lhe a que partição um valor vai.

## Onde funciona o particionamento
| Plataforma | Filegroups |
|----------|-----------|
| SQL Server em VM | Muitos grupos de arquivos. Os dados antigos podem ficar em discos mais baratos. Filegroup backup /store. |
| Managed Instance | Muitos grupos de arquivos permitidos |
| SQL Database | **Somente PRIMARY** |
| SQL database in Microsoft Fabric | Apenas PRIMARY. A compressão do interruptor e da per-partição é **bloqueado** (espelho) |

Use muitos grupos de arquivos apenas para armazenamento em camadas ou grupo de arquivos backups. Caso contrário, um grupo de arquivos é recomendado.

## Partition switching e a janela deslizante
`ALTER TABLE ... SWITCH` move uma partição inteira dentro ou fora como uma mudança **metadados-somente**. É quase snapshot. Nenhuma linha é copiada.

Regras para SWITCH:
- A tabela de destino ou partição deve **Existir e estar vazio**.
- Mesmo **filegroup**, mesmas colunas e índices, índices **alinhado**.
- **A compressão deve corresponder.**

**Janela deslizante** (mantenha os últimos 12 meses):
1. `SPLIT RANGE` para adicionar uma partição vazia para o próximo mês (set `NEXT USED` primeiro).
2. `SWITCH` a partição mais antiga para uma tabela de arquivo.
3. `MERGE RANGE` para remover o velho limite vazio.

```sql
ALTER PARTITION SCHEME psOrderDate NEXT USED [PRIMARY];
ALTER PARTITION FUNCTION pfOrderDate() SPLIT RANGE ('2026-04-01');
ALTER TABLE dbo.Orders SWITCH PARTITION 1 TO dbo.Orders_Archive;
ALTER PARTITION FUNCTION pfOrderDate() MERGE RANGE ('2026-01-01');
TRUNCATE TABLE dbo.Orders WITH (PARTITIONS (1));   -- or just empty a partition
```
Divida e mergulhe sempre partições **vazio**. Caso contrário, as linhas se movem e são fortemente registradas.

**Exemplo:** Uma loja on-line mantém 12 meses de pedidos. No 1o de cada mês, ele desliga o mês mais antigo em segundos, em vez de um `DELETE` lento de 10 milhões de linhas.

## O particionamento torna as consultas mais rápidas?
- **Eliminação da partição:** se o `WHERE` filtra na coluna de particionamento, SQL ignora as outras partições.
- Se a consulta faz o filtro **E** nessa coluna, toca cada partição. Pode obter **mais lento**.
- A Microsoft diz que o particionamento **raramente acelera OLTP**. Seu valor principal é **capacidade de gestão**: carga rápida e arquivo, reconstrução por partição e compressão, bloqueios de nível de partição.

## Tipos Data compression
| Tipo | Funciona em | Utilização |
|------|----------|-----|
| **ROW** | Tabelas e índices Rowstore | Armazena tipos de comprimento fixo em menos espaço. Baixo CPU. Bom OLTP padrão. |
| **PAGE** | Rowstore | Compressão ROW + **prefixo + dicionário**. Maior economia, mais CPU. Tabelas grandes e pesadas. |
| **COLUMNSTORE** | Columnstore índices | Always on. Análise. |
| **COLUMNSTORE_ARCHIVE** | Columnstore índices | Menor tamanho, leitura mais lenta. Dados **Frio, histórico**. |
| **compressão XML** | colunas `xml` | SQL Server 2022+, SQL Database, MI. |

Regras-chave:
- Você pode definir compressão **por partição** (PAGE para meses antigos, ROW para este mês).
- **Nonclustered indexes não herda** a compressão da tabela. Definir cada um.
- ROW / PAGE do **E** comprime os dados LOB fora da linha. Use a compressão `COMPRESS()` ou XML.
- Um heap ajustado para PAGE apenas comprime novas páginas em carga a granel, `INSERT ... WITH (TABLOCK)` ou reconstrução.
- Você pode reconstruir com `ONLINE = ON`. Faça uma partição de cada vez para limitar o espaço extra.
- Columnstore em níveis DTU precisa de **Padrão S3 ou superior**.
- Depois de SPLIT, ambas as partições mantêm a configuração antiga. Após MERGE, o resultado leva a configuração da partição em que se funde.

```sql
EXEC sys.sp_estimate_data_compression_savings
     @schema_name = 'dbo', @object_name = 'Orders',
     @index_id = NULL, @partition_number = NULL, @data_compression = 'PAGE';

ALTER TABLE dbo.Orders REBUILD PARTITION = ALL
WITH (DATA_COMPRESSION = PAGE ON PARTITIONS (1 TO 3),
      DATA_COMPRESSION = ROW  ON PARTITIONS (4), ONLINE = ON);
```
Sempre **primeiro estimativa** com `sp_estimate_data_compression_savings`. Se a estimativa crescer, não comprima.

**Exemplo:** O histórico de transações de 5 anos de um banco raramente é lido. Suas partições columnstore para anos antigos usam o COLUMNSTORE_ARCHIVE para economizar espaço.

## Sharding: o que é
**Sharding** é **escala out**: linhas do mesmo design de tabela estão espalhadas por muitas bancos de dados separadas (**shards**). O **Chave shard** (muitas vezes `TenantID` ou `CustomerID`) decide onde cada linha vive.
- **Escala acima** = a bigger single database.
- **Escale fora** = more databases.

## Tente opções mais simples primeiro
| Necessidade | Melhor ajuste |
|------|----------|
| Banco de dados outgrowing 4 TB | **Hyperscale** (128 TB), sem alterações no aplicativo |
| Mais capacidade de leitura | Read scale-out ou Hyperscale com o nome de réplicas |
| Muitas pequenas bancos de dados com carga irregular | Elastic pool |
| throughput ou tamanho além de qualquer banco de dados único | **Sharding** |
| Os inquilinos precisam de isolamento físico | Banco de dados por inquilino (uma forma de sharding) |
| Dados devem permanecer em diferentes países | Shard por região |

Sharding adiciona trabalho: roteamento, consultas shard cruzadas, reequilíbrio, alterações de esquema em bancos de dados N. Recomende-o apenas quando necessário. Se você fraturou apenas por causa do tamanho, pense em se mudar para o **Hyperscale**.

## Escolhendo uma chave shard
- In **Cada** tabela de sharded, idealmente o **primeira coluna da chave primária**.
- **Alta cardinalidade** (muitos valores distintos) e **mesmo espalhar**, portanto nenhum shard é "quente".
- Corresponde ao **Consulta mais comum**, então a maioria das consultas atinge um shard.
- **Estável.** Mudar significa mover a linha.
- Tipos suportados na biblioteca de clientes do Elastic Database: int, bigint, GUID, varbinary, datetime, timespan, datetimeoffset.

## Estratégias Sharding
| Estratégia | Como | Bom | Ruim |
|----------|-----|------|-----|
| **Gama** | Intervalos de valor (IDs 1–1M → Shard a) | Simples; consultas de alcance permanecem em um shard | **pontos quentes** se novos dados sempre chegam na faixa mais nova |
| **Hash** | Hash da chave (`UserID % 4`) | Mesmo carga, sem pontos quentes | **As consultas de alcance são difíceis**, os dados são dispersos |
| **Lookup (diretório)** | Uma tabela de mapeamento diz onde cada chave vive | Muito flexível; mova um inquilino facilmente | Passo lookup extra; o mapa deve permanecer consistente |

A biblioteca de cliente do Elastic Database tem apenas mapas **lista** e **gama** shard. Para hash, o aplicativo calcula um balde e mapeia baldes com uma lista ou mapa de alcance.

**Exemplo:** Um aplicativo de foto armazena usuários por `UserID % 8` (hash). Carga é uniforme. Mas "todos os usuários que se juntaram entre ID 1000 e 2000" devem perguntar a todos os 8 shards.

## Padrões multilocatários (SaaS)
| Padrão | Isolamento | Custo por inquilino | Escala |
|---------|-----------|-----------------|-------|
| Aplicativo autônomo + único DB por inquilino | Mais Alta | Mais Alta | 1–100s |
| **Banco por tenant** | Alta | Baixo com **elastic pools** | Centenas de milhares |
| **Sharded multi-tenant DBs** | Baixa | **Mais Baixa** | Até milhões |
| híbrido | Misto | Misto | Muito alta |

- Database per tenant = **lista** (uma chave de mapeamento → um banco de dados). Fácil restauração por inquilino.
- Multi-tenant shards = **gama** Precisa de um inquilino ID Em cada tabela e **row-level security**.
- Você **Não pode restaurar um inquilino** de um banco de dados multi-tenant. Mova-o com split-merge ou seu próprio ETL.

## Ferramentas de banco de dados elásticas (somente SQL Database)
| Ferramenta | Finalidade | Status |
|------|---------|--------|
| **Elastic banco de dados biblioteca cliente** | Constrói o mapa shard; **roteamento dependente de dados** (`OpenConnectionForKey`) | Apoiado |
| **Shard map manager** | Banco de dados global de mapas shard + um mapa local em cada shard | Parte da biblioteca |
| **Ferramenta Split-merge** | Move **shardlet** (todos os dados para uma chave) entre shards | (construído em serviços de nuvem aposentados) |
| **Elastic query** | Read-only T-SQL em bancos de dados através de tabelas externas | Pré-visualização. Horizontal (`SHARD_MAP_MANAGER`) modo **Termina 31 Mar 2027** |
| **Elastic jobs** | Executar T-SQL em muitos bancos de dados em um cronograma | GA |
| **transações elásticas** | Transações distribuídas em bancos de dados SQL | Apoiado |

- Os métodos do mapa Shard mudam **apenas metadados**. Eles nunca movem dados.
- Elastic query: read-only, **Somente SQL authentication**, um shard offline falha toda a consulta.
- Elastic jobs: alvo um **servidor** ou **piscina** e novo shards são pegos automaticamente. Use um **user-assigned managed identity** (sistema atribuído não é suportado). Job banco de dados S1 ou superior, **não Hyperscale**. Scripts devem ser seguros para re-executar.
- No **Managed Instance**, use consultas de banco de dados cruzado ou servidores vinculados em vez dessas ferramentas.

## Para lembrar na prova
- Datas → **RANGE RIGHT**. N boundaries = N + 1 partitions.
- "Remover um mês de dados instantaneamente" → **SWITCH fora** ou `TRUNCATE ... WITH (PARTITIONS)`, não DELETE.
- "Partições antigas no armazenamento mais barato" → filegroups em **VM ou MI**. Não é possível em SQL Database.
- SWITCH falha → verificação alvo vazio, mesmo grupo de arquivos, índices alinhados, **mesma compressão**.
- Frio columnstore → **COLUMNSTORE_ARCHIVE**. Estimativa primeiro → `sp_estimate_data_compression_savings`.
- "50 TB, mudanças mínimas de aplicativos" → **Hyperscale**, não sharding.
- "Milhares de inquilinos, isolamento, restauração por inquilino, baixo custo" → **banco de dados por inquilino em elastic pools**.
- "Milhões de pequenos inquilinos, o menor custo" → **sharded multi-tenant + RLS**.
- "Roteie uma conexão com o shard direito " → **Elastic banco de dados biblioteca cliente**.
- "A mesma mudança de esquema em todos os shards, incluindo os futuros" → **elastic job** visando o servidor ou a piscina.

## Teste rápido
1. Um interruptor de partição mensal falha. A tabela de arquivo está vazia e no mesmo grupo de arquivos. A partição de origem usa PAGE compression; a tabela de arquivo não usa nenhum. O que há de errado?
   <details><summary>Resposta</summary>A compressão deve corresponder entre a fonte e o alvo. Reconstruir a tabela de arquivo com PAGE.</details>
2. Qual estratégia shard evita pontos quentes, mas torna as consultas de alcance caras?
   <details><summary>Resposta</summary>Hash sharding. Ele se espalha linhas uniformemente, então um intervalo é espalhado por todo o shards.</details>
3. Você quer manter partições de pedidos antigos em discos mais baratos. Você pode fazer isso em Azure SQL Database?
   <details><summary>Resposta</summary>No. SQL Database tem somente o grupo de arquivos PRIMARY. Use um VM ou Managed Instance.</details>
