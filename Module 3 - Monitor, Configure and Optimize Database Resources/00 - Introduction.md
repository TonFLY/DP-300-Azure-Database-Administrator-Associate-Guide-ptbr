# Módulo 3 — Monitorar, configurar e otimizar recursos de banco de dados

**Peso do exame:** 20–25% · **Tempo de leitura:** ~96 min · **Arquivos:** 10

## O que você vai aprender
Você vai aprender a identificar e diagnosticar a lentidão de um banco de dados. O módulo cobre ferramentas de monitoramento para VMs, Azure SQL Database e Managed Instance, além de configurações de computação, armazenamento, servidor e manutenção de índices. Você também vai ler planos de execução, usar Query Store, resolver bloqueios e reescrever consultas lentas.

Pense em um restaurante movimentado. O gerente anota como é uma noite normal (a **baseline**) e observa a cozinha com câmeras (**ferramentas de monitoramento**). Ele define quantos cozinheiros e quanta refrigeração são necessários (**computação e armazenamento**) e organiza a limpeza ao fim da noite (**manutenção**). Uma ficha de receita mostra as etapas (**plano de execução**) e um registro destaca a receita que funcionou melhor (**Query Store**). Dois cozinheiros disputando o mesmo fogão criam uma fila (**bloqueio**). Uma despensa organizada agiliza o preparo (**índices**).

## Lições neste módulo
| # | Lição | O que ela cobre | Tempo |
|---|--------|----------------|------|
| 1 | [Bases de base de desempenho e ferramentas de monitoramento](<01 - Performance baselines and monitoring tools.md>) | Linhas de base, fontes métricas, alertas, QPI | ~11 min |
| 2 | [Extended Events, database watcher e Intelligent Insights](<02 - Extended Events, database watcher and Intelligent Insights.md>) | Sessões XE, database watcher, SQLInsights | ~11 min |
| 3 | [Configurar configurações de computação, armazenamento e servidor](<03 - Configure compute, storage and server settings.md>) | Discos, tamanho VM, tempdb, escala, MAXDOP, Resource Governor | ~12 min |
| 4 | [Manutenção de bancos de dados](<04 - Database maintenance.md>) | Indexes, statistics, DBCC CHECKDB, configuração de escopo | ~10 min |
| 5 | [Automatic tuning e intelligent query processing](<05 - Automatic tuning and intelligent query processing.md>) | Correção de plano, índices auto, IQP | ~9 min |
| 6 | [Planos de execução e DMVs](<06 - Execution plans and DMVs.md>) | Estimado vs planos reais, chave DMVs | ~9 min |
| 7 | [Query Store](<07 - Query Store.md>) | Configurações, relatórios, plan forcing, dicas | ~8 min |
| 8 | [Estatísticas de bloqueio, bloqueio e espera](<08 - Blocking, locking and wait statistics.md>) | Fechaduras, deadlocks, RCSI, wait types | ~11 min |
| 9 | [Design de índices e query tuning](<09 - Index design and query tuning.md>) | Tipos de índice, design, reescritas de consulta, dicas | ~12 min |

## Palavras que você vai encontrar
| Palavra | Significado simples |
|------|----------------|
| linha de base | Seus números normais de desempenho, registrados para comparar com os números mais tarde |
| DMV | Uma visão interna do sistema (`sys.dm_...`) que mostra o que o SQL Server está fazendo agora |
| Plano de execução | A receita passo a passo SQL Server usa para executar uma consulta |
| Query Store | Um gravador no banco de dados que mantém planos de consulta e tempos de execução ao longo do tempo |
| Regressão do plano | Um novo plano que é mais lento do que um mais velho |
| Estatísticas | Notas sobre como os valores são distribuídos em uma coluna, usados para adivinhar a contagem de linhas |
| MAXDOP | A maioria dos núcleos CPU que uma consulta pode usar |
| Resource Governor | Uma característica que limita CPU, memória e E/S por grupo de usuários |
| Bloqueio | Uma sessão à espera de outra para liberar um bloqueio |
| SARGable | Escrito para que SQL Server possa usar um index seek |

## exame de habilidades cobertas
- [ ] Preparar uma linha de base de desempenho operacional → 1
- [ ] Determinar fontes para métricas de desempenho → 1
- [ ] Interpretar métricas de desempenho → 1, 8
- [ ] Configurar e monitorar a atividade e o desempenho → 1
- [ ] Monitor usando database watcher → 2
- [ ] Monitor usando Extended Events → 2
- [ ] Configurar Query Store → 7
- [ ] Usando Query Store → 7
- [ ] Identificar e resolver bloqueio de sessão → 8
- [ ] Identificar problemas de desempenho usando visões dinâmicas de gerenciamento (DMVs) → 6, 8
- [ ] Identificar e implementar alterações de índice para consultas → 9
- [ ] Recomendar modificações de construção de consulta com base no uso de recursos → 9
- [ ] Planos de execução da revisão → 6
- [ ] Monitor usando Intelligent Insights → 2
- [ ] Implementar tarefas de manutenção do índice → 4
- [ ] Implementar estatísticas tarefas de manutenção → 4
- [ ] Implementar verificações de integridade do banco de dados → 4
- [ ] Configurar banco de dados automatic tuning → 5
- [ ] Configurar as configurações do servidor para o desempenho → 3
- [ ] Configurar Resource Governor para desempenho → 3
- [ ] Implementar configuração de banco de dados com escopo → 4
- [ ] Configurar recursos de computação e armazenamento para escala → 3
- [ ] Identificar casos de uso para intelligent query processing (IQP) características → 5
