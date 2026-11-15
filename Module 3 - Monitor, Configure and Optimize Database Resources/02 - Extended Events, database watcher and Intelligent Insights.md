# Extended Events, database watcher e Intelligent Insights

> **Habilidades cobradas:** Monitor usando Extended Events, Monitor usando database watcher, Monitor usando Intelligent Insights
> **Tempo de leitura:** ~11 min

## Conceito central
Estas são três ferramentas de monitoramento mais profundas.
- **Extended Events** é um gravador de luz que você liga para capturar eventos exatos, como deadlocks.
- **Database watcher** é um painel gerenciado que coleta números de muitos Azure SQL databases em tempo quase real.
- **Intelligent Insights** é integrado AI que percebe quando o desempenho piora e anota a causa provável.

Pense em uma loja. Extended Events é uma câmera de segurança que você aponta para uma porta. Database watcher é a sala de controle com telas para todas as lojas da cadeia. Intelligent Insights é um guarda inteligente que escreve um relatório: "As vendas desaceleraram hoje porque o toll foi quebrado".

## Extended Events (XE)
**Extended Events** é o sistema de rastreamento leve e moderno em SQL Server, Azure SQL Database e Managed Instance. Ele substitui o antigo **Perfilr SQL**.
- Baixo overhead (que retarda o servidor muito pouco), especialmente com bons filtros.
- Pode assistir quase tudo: consultas, fechaduras, deadlocks, E / O, erros.

**Utilizações comuns:**
- Apanhar **deadlocks** e bloquear.
- Encontrar **consultas de longa duração**.
- Watch **DDL** (mudanças para tabelas e outros objetos).
- Registro de estatísticas ausentes.
- Pressão de memória ou disco lento I/ O.

**Exemplo:** Um aplicativo escolar recebe um erro deadlock duas vezes por dia. O DBA inicia uma sessão XE que registra o relatório deadlock. No dia seguinte, o relatório mostra quais dois procedimentos lutam nas mesmas linhas.

### As quatro partes de uma sessão
| Parte | Significado | Exemplo |
|------|---------|---------|
| **Evento** | A coisa para pegar | `sql_statement_completed`, `lock_acquired` |
| **Ação** | Dados extras a serem adicionados a cada evento | Sessão ID, texto SQL |
| **Predicar (filtro)** | Quais eventos manter | Apenas um banco de dados, apenas consultas ao longo de 5 segundos |
| **Alvo** | Onde armazenar os resultados | Um arquivo ou memória |

### Metas
| Alvo | O que faz | ou async | Onde |
|--------|--------------|---------------|-------|
| **event_file** | Salva eventos em um arquivo para mais tarde | Assíncrono | Em Azure SQL DB e MI o arquivo vai para **Azure Blob Storage** |
| **ring_buffer** | Mantém eventos recentes na memória; perdido quando a sessão para | Assíncrono | Todos |
| **event_counter** | Apenas conta cada evento | síncron | Todos |
| **histograma** | Conta eventos agrupados por um valor de campo | síncron | Todos |
| **pair_matching** | Encontra eventos pareados que não correspondem (por exemplo, bloqueio tomado, mas não liberado) | Assíncrono | **Somente SQL Server** |
| **ETW** (Event Tracing for Windows) | Junta-se a eventos SQL com rastreamentos do Windows OS | síncron | **Somente SQL Server** |

- Use **event_file** para longas sessões que você deseja manter.
- Use **ring_buffer** para verificações ao vivo rápidas.
- Em **Azure SQL Database e MI**, o `event_file` O alvo precisa de a **credencial** para a conta de armazenamento (managed identity : SAS token).

### Server vs database scope
- **SQL Server e Managed Instance:** `CREATE EVENT SESSION ... ON SERVER`
- As sessões **Azure SQL Database:** são **banco de dados-scoped**: `CREATE EVENT SESSION ... ON DATABASE`

```sql
CREATE EVENT SESSION LongQueries ON DATABASE   -- Azure SQL Database
ADD EVENT sqlserver.sql_statement_completed
    (ACTION (sqlserver.sql_text, sqlserver.session_id)
     WHERE duration > 5000000)                 -- microseconds = 5 seconds
ADD TARGET package0.ring_buffer;
ALTER EVENT SESSION LongQueries ON DATABASE STATE = START;
```

Em SSMS: **Gestão → Extended Events → Sessões → Nova Sessão** (modelos como **Bloqueios e bloqueios** e **Execução de Consulta**).

**Práticas recomendadas:** sempre adicionar filtros, pegar apenas os eventos que você precisa, e parar as sessões que você não usa mais.

## Database watcher
**Database watcher** é o serviço de monitoramento gerenciado do Azure para **Azure SQL Database** e **Azure SQL Managed Instance**. Está em **pré-visualização**.
- Ele coleta dados de monitoramento (de DMVs) **em segundos** (quase em tempo real).
- Ele mostra **dashboard** pronto no Azure portal.
- Ele substitui o **SQL Insights**, que foi aposentado em 31 de dezembro de 2024.

Contadores de desempenho **O que ele pode assistir:** (CPU, I/O, esperas), desempenho de consulta, deadlocks e bloqueio, uso elastic pool, mudanças de configuração, saúde Managed Instance.

**Alvos apoiados:** todos os níveis de serviço Azure SQL Database e MI e modelos de compra (DTU, vCore, serverless, Hyperscale, elastic pools), além de réplicas legíveis, geo e nomeadas. **Não SQL Server on Azure VMs.**

**Onde os dados são armazenados (armazenamento de dados):**
| Armazenamento de dados | Notas |
|-----------|-------|
| **Azure Data Explorer (ADX) cluster** | Bom para grandes quantidades de dados de séries temporais; consulta com KQL |
| **Cluster ADX grátis** | Para pequenas cargas de trabalho ou testes |
| **Inteligência em tempo real em Microsoft Fabric** | O mesmo motor que ADX, como um serviço SaaS (anteriormente Real-Time Analytics) |

**Configuração em resumo:** criar um observador (assinatura, grupo de recursos, nome, região) → escolher o armazenamento de dados → adicionar SQL alvos → criar → imprensa **Iniciar**. Um novo observador faz **E** começar por si só.

**Limites e custos (preview):**
- Até **Alvo 100 SQL por observador** e **20 observadores por assinatura**.
- O observador e os dashboards são **livre**. Você paga pelo cluster ADX ou capacidade de tecido (e alertas e Key Vault se você usar SQL authentication).
- **Alertas** (Alertas de busca de log Azure Monitor) são **E** disponíveis se o armazenamento de dados for Fabric Real-Time Intelligence ou um cluster ADX gratuito.

**Exemplo:** Uma empresa de software tem 60 bancos de dados de clientes em vários elastic pools. Ele cria um database watcher com um cluster ADX pago. Agora, um painel mostra CPU, espera e bloqueia todos os 60 bancos de dados e um alerta dispara quando um deadlock acontece.

## Intelligent Insights
O **Intelligent Insights** usa o AI integrado para assistir ao seu banco de dados e encontrar eventos que prejudicam o desempenho.
- Funciona para **Azure SQL Database** (único e agrupado) e **Azure SQL Managed Instance**.
- Status: ainda **pré-visualização**, e não disponível na Europa Ocidental, Norte da Europa, Oeste US 1 e Leste US 1.
- Quando encontra um problema, escreve uma entrada de log na categoria **`SQLInsights`** com:
  - **análise da causa raiz** (uma razão de linguagem simples),
  - **consultas afetadas** (como hashes de consulta) ou códigos de erro,
  - **Recomendações** sempre que possível.

**Como funciona:**
- Compara o **last hour** de carga de trabalho com um **linha de base de 7 dias**.
- Também verifica limites fixos: muitas esperas, erros críticos, problemas de parametrização.
- A análise geralmente leva cerca de **1 hora**. Para um problema acontecendo agora, use **Query Store** ou DMVs.
- Cada edição tem um status: **→ Active Verificando → Completo**.

Duração da consulta **O que mede:**, solicitações de tempo limite, tempos de espera excessivos, solicitações de erro.

**Alguns dos 15 padrões que ele pode detectar** (outros incluem aumento da carga de trabalho, pressão de memória, contenda tempdb e cliente lento):
| Padrão | correção típica |
|---------|-------------|
| Alcançar os limites de recursos | Consultas ou aumento de volume |
| Fechamento | Transações mais curtas, lotes menores |
| Aumento MAXDOP | Set MAXDOP mais baixo |
| Missing index | Crie o índice ou ligue automatic tuning |
| Elastic pool DTU escassez | Sintonize ou aumente o tamanho da piscina (**Somente SQL Database**, nunca no MI) |
| Regressão do plano | Forçar o bom plano / **FORCE_LAST_GOOD_PLAN** |
| Mudança de configuração com escopo de banco de dados | Desfazer a mudança de configuração recente |
| Nível de preços downgrade | Recue ou reduza a carga |

### Ligar: configurações de diagnóstico
Insights são **não armazenado por padrão**. Você deve adicionar um **definição diagnóstica** em **cada banco de dados** e marcar a categoria **`SQLInsights`**.

| Destino | Uso para |
|-------------|---------|
| **Log Analytics workspace** | Consultas KQL, alertas de log, workbooks (tabela `AzureDiagnostics`) |
| **Event Hubs** | Transmissão para uma ferramenta SIEM ou de terceiros |
| **Azure Storage** | Arquivo barato de longo prazo |

- **Managed Instance:** configurá-lo em cada **banco de dados de instância**. Você não pode configurá-lo em MI **banco de dados do sistema**.
- **Elastic pool:** o colocou em **Cada banco de dados agrupado**.

**Exemplo:** A banco de dados de pagamentos de um banco fica mais lenta uma tarde. Uma hora depois, Intelligent Insights registra: "Plane a regressão na consulta hash X". O banco já envia `SQLInsights` para Log Analytics, e um alerta de log envia e-mails para o DBA. Automatic tuning já havia forçado o último bom plano.

Observação: Não misture o `SQLInsights` **categoria log** (ainda existe) com o **Azure Monitor SQL Insights** (um produto separado, aposentado em 31 de dezembro de 2024).

## Compare em um relance
| Necessidade | Ferramenta |
|------|------|
| Apanhar eventos exatos, como deadlocks ou uma declaração lenta | Extended Events |
| Dashboards quase em tempo real em muitos Azure SQL databases | Database watcher |
| Causa raiz AI de uma regressão, transmitida para Log Analytics ou Event Hubs | Intelligent Insights |

## Para lembrar na prova
- Extended Events substitui SQL Profiler e tem baixa sobrecarga.
- As sessões Azure SQL Database XE usam **ON DATABASE**; SQL Server e MI usam **ON SERVER**.
- Azure SQL DB / MI o alvo **event_file** escreve para **Blob Storage**; **ETW** e **pair_matching** são apenas SQL Server.
- **ring_buffer** = memory, lost when stopped. **event_file** = kept.
- Database watcher: SQL DB e MI apenas (não VMs), dados em **ADX** ou **Tecido RTI**, deve pressionar **Iniciar**.
- Não há alertas com um armazenamento de dados **cluster ADX livre** ou **Tecido RTI**.
- Intelligent Insights: **linha de base de 7 dias**, sobre **1 hora** to analyse, output = **`SQLInsights`** log.
- deve permitir a **definição diagnóstica** por banco de dados. Arquivo → Armazenamento, SIEM → Event Hubs, KQL e alertas → Log Analytics.
- **Elastic pool DTU escassez** nunca aparece no Managed Instance.

## Teste rápido
1. Você deve capturar detalhes deadlock em Azure SQL Database e mantê-los após a sessão parar. Qual alvo?
   <details><summary>Resposta</summary>event_file (escrito para Azure Blob Storage). ring_buffer está na memória e é perdido quando a sessão pára.</details>
2. Você precisa de análise de causa raiz de regressões de desempenho enviadas para um SIEM de terceiros. O que você configura?
   <details><summary>Resposta</summary>Uma configuração de diagnóstico em cada banco de dados com a categoria SQLInsights enviada para o Event Hubs.</details>
3. Pode database watcher monitor SQL Server rodando em uma Azure VM?
   <details><summary>Resposta</summary>Não. Suporta apenas Azure SQL Database e Azure SQL Managed Instance.</details>
