# Bases de base de desempenho e ferramentas de monitoramento

> **Habilidades cobradas:** Preparar uma linha de base de desempenho operacional, Determinar fontes para métricas de desempenho, Interpretar métricas de desempenho, Configurar e monitorar atividade e desempenho
> **Tempo de leitura:** ~11 min

## Conceito central
Você não pode dizer "o banco de dados é lento" a menos que você saiba o que "normal" parece. Então, primeiro você registra números normais (a **basal basal**). Então você assiste os mesmos números todos os dias com as ferramentas certas. Quando um número se move longe do normal, você recebe um alerta e começa a procurar.

Imagine uma consulta médica. Sua frequência cardíaca normal é de 70. Se chegar a 140 enquanto você está sentado, há um problema. O médico sabe disso porque registrou o valor de 70 antes.

## O que é uma linha de base?
- A **basal basal** é um registro de seus números normais de desempenho (CPU, memória, disco, consultas por segundo) durante um período normal.
- Mais tarde, você compara novos números com ele.
- Sem uma linha de base, você não pode dizer um **pico normal ocupado ocupado** de um **problema real**.

**Exemplo:** Um aplicativo escolar normalmente usa 30–40% CPU durante o dia. Uma segunda-feira mostra 95%. Porque você tem uma linha de base, você sabe que isso não é normal e você começa a investigar.

## Veja o SQL Server e o OS juntos
Ligado **SQL Server em uma Azure VM** (uma máquina virtual onde você instala e executa SQL Server um problema pode vir de dois lugares:
- **Dentro SQL Server**, por exemplo, uma consulta ruim.
- **fora SQL Server**, no sistema operacional (OS) ou hardware, por exemplo, um disco lento.

Então você coleta os dois tipos de números e olha para eles lado a lado.
- Ligado **Windows**, use **Performance Monitor (Perfmon)** (a ferramenta Windows integrada que registra contadores).
- Ligado **Linux**, ferramentas como **InfluxDB**, **coleccionado** e **Grafana** Faça o mesmo job.

**Exemplo:** A DMV (uma visão interna do sistema, explicada na lição 6) mostra leituras lentas de um arquivo de dados. Perfmon também mostra leituras lentas no disco. Então o disco é lento, não a consulta. Se Perfmon mostrou um disco rápido, você olharia para a consulta em vez disso.

## Contadores Key Perfmon para uma linha de base
| Contador | O que mostra | O que procurar |
|---------|---------------|------------------|
| `Processor(_Total)\% Processor Time` | Utilização total CPU | Alta CPU = heavy queries or a VM Isso é muito pequeno |
| `Paging File(_Total)\% Usage` | Uso do arquivo de página (disk usado como memória extra) | Pressão da memória |
| `PhysicalDisk(_Total)\Avg. Disk sec/Read` e `/Write` | Delay do disco (latência) por leitura ou escrita | Normalmente com menos de 20 ms; abaixo de 10 ms no armazenamento Premium |
| `System\Processor Queue Length` | Tarefas aguardando um CPU | Acima de 0 significa pressão CPU |
| `SQLServer:Buffer Manager\Page life expectancy` | Quanto tempo as páginas de dados ficam na memória SQL Server | A sudden drop = memory pressure or bad queries |
| `SQLServer:SQL Statistics\Batch Requests/sec` | Queries run por segundo | Quão ocupado é o servidor |
| `SQLServer:SQL Statistics\SQL Compilations/sec` e `Re-Compilations/sec` | Com que frequência o SQL Server constrói planos de consulta | Very high values = query or memory issues |

Também gravar **estatísticas de espera** (por que as consultas estão esperando) com `sys.dm_os_wait_stats`. Lição 8 cobre esperas em detalhes.

## De onde vêm os números de desempenho (fontes métricas)
Ferramentas diferentes vêem camadas diferentes. Este é um tema de exame comum.

| Fonte | O que vê | Notas |
|--------|--------------|-------|
| **Azure Monitor** (serviço de monitoramento integrado da Azure) | Números VM de nível de host: CPU, E/O do disco, rede | **E** mostra números internos SQL Server |
| **VM Insights / monitoramento aprimorado** | Números OS: memória livre, latência do disco, espaço em disco | Instala o **agente Azure Monitor**; os dados vão para um Log Analytics workspace |
| **Log Analytics workspace** | Armazena logs e métricas | Você consulta com **KQL** (Kusto Query Language) |
| **SQL IaaS Agent extension** | Gerenciamento específico SQL para um VM no portal | As imagens do Marketplace SQL registram-se automaticamente; o SQL Server 2016+ auto-instalado geralmente é registrado automaticamente, caso contrário, registre-se manualmente |
| **Perfmon** | Contadores OS e SQL Server dentro do VM | Pode enviar seus dados para Azure Monitor |
| **DMVs** | O que está acontecendo dentro do SQL Server agora | Funciona em todos os tipos Azure SQL |
| **Query Store / Query Performance Insight** | Histórico de consultas e principais consultas | Veja abaixo e lição 7 |

Observação: O SQL IaaS Agent extension também pode fazer o automated patching, mas isso está se movendo para o **Azure Update Manager**. O Automated Patching da extensão se aposenta em 17 de setembro de 2027.

## Azure Monitor metrics e alertas
- O Azure Monitor coleta números de recursos do Azure em horários regulares.
- Ele mantém dados métricos para **93 dias**. Para mantê-lo por mais tempo, envie-o para **Azure Storage** (ou um Log Analytics workspace).
- Você pode desenhar gráficos e criar **Regras de alerta**.

**Peças da regra de alerta:**
- **Scope** – quais recursos assistir. Para VMs, esta pode ser uma lista de VMs, todos os VMs em grupos de recursos ou todos os VMs em uma assinatura (sempre em **uma região do Azure**).
- **Condição** – uma métrica, um operador (`>`, `<`) e uma agregação (média, max, min, total, contagem).
- **Lim limiar** – dois tipos:

| Tipo de limiar | Como funciona | Bom para |
|----------------|--------------|----------|
| **Estática** | Você define o número, por exemplo CPU > 80% | Limites simples e fixos |
| **Dinâmico** | Azure aprende com a história e define o limite em si | Cargas de trabalho com padrões diários ou sazonais |

**Grupo de acção** – o que acontece quando o alerta dispara:
- **Notificar:** e-mail, SMS, notificação push, chamada de voz, e-mail um papel Azure Resource Manager.
- **Automate:** Azure Automation runbook, Função do Azure, App Lógica, ITSM, webhook, webhook seguro.
- Sem nenhum grupo de ação, o alerta é registrado apenas.

**Exemplo:** Uma loja online define um alerta CPU dinâmico. Na Black Friday CPU sobe, mas o Azure conhece esse padrão, portanto, nenhum alarme falso. Um estranho pico em 3 a.m. dispara o alerta e um runbook envia e-mails ao administrador de plantão.

## Métricas-chave para Azure SQL Database e Managed Instance
Em PaaS (Azure executa o servidor para você) você não pode usar Perfmon. Você usa as métricas do portal e DMVs.
- `sys.dm_db_resource_stats` – CPU, I/O de dados, E/O de log e memória para **Azure SQL database**. Uma linha cada **15 segundos**, aproximadamente **1 hora** mantido.
- `sys.server_resource_stats` – uso de recursos para um **Managed Instance**. Cerca de **14 dias** mantido.
- `sys.dm_user_db_resource_governance` – os limites do seu Azure SQL database (o que o seu nível permite).
- `sys.dm_instance_resource_governance` – o mesmo para Managed Instance.

**Como ler os números:**
- CPU perto de 100% por um longo tempo → encontrar o top CPU consultas (Query Store), ajustá-los, ou escalar.
- I/O de dados perto do limite → índices ausentes (excesso de varreduras) ou um nível que é muito pequeno.
- Log I/O perto do limite → muitas escritas; você pode ver `LOG_RATE_GOVERNOR` espera.

## Query Performance Insight (QPI)
**QPI** é uma página no Azure portal que mostra suas consultas mais caras. Ele lê seus dados do **Query Store**.
- Onde: Azure SQL database → **Intelligent Performance** → **Query Performance Insight**.
- **Somente Azure SQL Database** (único e agrupado). **Não** Managed Instance.
- Needs **Query Store** ligado (está ativado por padrão no Azure SQL Database).

**Três pontos de vista:**
| Ver | O que mostra |
|------|---------------|
| **Consultas de consumo de recursos** | Consultas de topo por CPU, I/O de dados ou log I/O |
| **Consultas de longa duração** | Top consultas por duração (padrão: últimas 24 horas) |
| **Personalizado** | Escolha métrica (documentos atuais: CPU, duração, contagem de execução) e intervalo de tempo (24 h, semana passada, mês passado) |

**Limites:**
- Mostra apenas as consultas **top 5–20**.
- Mostra **totais e médias**, não corridas únicas.
- O **E** mostra planos de execução. Clique em uma consulta, anote seu **Consulta ID** e abra essa consulta no Query Store para ver o plano.
- Para uma análise mais profunda, a Microsoft aponta para **database watcher** (lição 2).

**Exemplo:** Um aplicativo de entrega de alimentos é lento nas noites de sexta-feira. Em QPI, "Consultas de consumo de recursos" classificadas por CPU mostra um `SELECT` juntando Pedidos e Clientes usa 60% do CPU. O DBA leva sua Query ID para Query Store e encontra um table scan no plano.

## Para lembrar na prova
- Uma **baseline** registra os valores normais primeiro, para que você possa identificar anomalias depois.
- **Azure Monitor não mostra métricas internas SQL Server** em um VM. Use Perfmon, DMVs ou o SQL IaaS Agent extension.
- Azure Monitor mantém métricas **93 dias**; arquivo para armazenamento por mais tempo.
- **limiares dinâmicos** aprender com a história; bom para padrões sazonais.
- O escopo de alerta para o VMs está sempre dentro do **uma região**.
- **QPI = Azure SQL Database apenas**, precisa de Query Store, sem planos mostrados, top 5–20 consultas.
- A latência do disco deve ser **Menos de 20 ms** (menos de 10 ms no Premium).
- `sys.dm_db_resource_stats` = one Azure SQL database, 15 segundos linhas, cerca de 1 hora de história.

## Teste rápido
1. Seu SQL Server VM mostra alta CPU em Azure Monitor. Você precisa ver o contador SQL Server "Solicitação de correspondência / seg". Qual ferramenta você usa?
   <details><summary>Resposta</summary>Performance Monitor (Perfmon) dentro do VM, ou DMVs. Azure Monitor não mostra contadores internos SQL Server.</details>
2. Você precisa encontrar o top 5 CPU -pesados consultas de um Azure SQL Managed Instance no portal usando Query Performance Insight. Será que este trabalho?
   <details><summary>Resposta</summary>No. QPI trabalha somente para Azure SQL Database. Em Managed Instance use Query Store ou DMVs.</details>
3. Sua carga de trabalho está ocupada todas as manhãs de segunda-feira. Você quer alertas apenas para o incomum CPU, não o pico normal de segunda-feira. Que tipo de limite?
   <details><summary>Resposta</summary>Um limiar dinâmico. Aprende o padrão normal da história.</details>
