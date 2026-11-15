# DP-300 — Resumo de consulta rápida

> Leia na noite anterior à prova. ~30 min.

Siglas usadas neste resumo. **VM** = SQL Server em uma Azure virtual machine (IaaS, que você administra). **MI** = Azure SQL Managed Instance (PaaS). **SQL DB** = Azure SQL Database (PaaS). **PaaS** = a Microsoft administra o servidor. **IaaS** = você administra o SQL Server em uma máquina virtual.

## 1. Qual opção Azure SQL?

| | SQL Server em VM | SQL Managed Instance | Azure SQL Database | SQL database in Microsoft Fabric | SQL Server enabled by Azure Arc |
|---|---|---|---|---|---|
| Tipo | IaaS | PaaS | PaaS | SaaS, dentro do Microsoft Fabric | Seu próprio servidor (on-prem, outra nuvem, borda) gerenciado a partir do Azure |
| acesso OS | Sim | Não | Não | Não | Sim (é o seu servidor) |
| Mudanças de código para mover | Nenhum | Poucos ou nenhum | Muitos | n/a | Nenhum (não se move) |
| SQL Agent | Sim | Sim (always on) | Não (use elastic jobs) | Não | Sim |
| consultas de banco de dados, servidores vinculados, CLR | Sim | Sim | Limitado | Não | Sim |
| Esforço de administração | Alta | Mais baixo | Mais Baixa | Mais Baixa | O mesmo que hoje |
| Melhor para | Lift-and-shift as-is, versões antigas SQL, necessidades de nível OS, SSRS / SSAS, FILESTREAM, PolyBase | Movendo um aplicativo existente com poucas mudanças | Novos aplicativos em nuvem, SaaS, carga desconhecida ou pontiaguda | Análise dos seus dados SQL sem ETL | Manter servidores onde eles estão, mas gerenciá-los a partir do Azure |

**Se a pergunta diz X → escolha Y**

- Precisa de acesso OS, uma versão personalizada SQL ou nível de patch → **VM**.
- Necessidades SSRS, SSAS, FILESTREAM, PolyBase ou transações cruzadas → **VM**.
- Necessidades SQL Agent, servidores vinculados, consultas entre bancos de dados, CLR, com **trabalho OS** → **MI**.
- "Mudanças mínimas de código" + "menor esforço de administração" → **MI**.
- Novo aplicativo, escala automática, pague apenas quando usado → **SQL DB serverless** (General Purpose ou Hyperscale).
- Banco de dados maior que 4 TB, até 128 TB → **SQL DB Hyperscale**. Hyperscale é agora o nível padrão recomendado da Microsoft para novo trabalho OLTP.
- Muitas pequenas bancos de dados com carga desigual (por exemplo, uma por cliente) → **SQL DB elastic pool**.
- Os dados devem ser copiados para **OneLake** para relatórios e aprendizado de máquina sem ETL (sem cópia jobs) → **SQL database in Microsoft Fabric**. Ele cria um ponto final de análise SQL e um modelo semântico padrão.
- Os servidores On-premises precisam de inventário, verificações de melhores práticas, alertas Defender ou governança Purview, **sem migração** → **Arco Azure**.
- O dispositivo de IoT ou borda precisa de um pequeno banco de dados local → **Azure SQL Borda**.

**Picaretas rápidas de licenciamento**

| Necessidade | Escolha |
|---|---|
| Teste curto, sem licença | Pay-as-you-go (licença incluída) |
| Já possui licenças SQL com Software Assurance | Azure Hybrid Benefit |
| VM correndo anos 1–3 | Capacidade Reservada |
| Licença livre para uma cópia passiva DR | HA/DR tipo de licença (VM) ou uma réplica **standby** (SQL DB geo-secundário, MI failover group) |
| Hyperscale | Taxa de licença SQL. Azure Hybrid Benefit é **E** disponível para novas bancos de dados Hyperscale |

**Tamanho e números de nível para saber**

| Facto | Valor |
|---|---|
| SQL DB General Purpose / Business Critical tamanho máximo | 4 TB |
| SQL DB Hyperscale tamanho máximo | 128 TB |
| Hyperscale volta para General Purpose | Somente dentro de **45 dias** |
| réplicas Hyperscale | Até 4 réplicas HA, até 30 réplicas nomeadas, além de réplicas geográficas |
| Atraso Serverless auto-pause (General Purpose) | 15 min a 7 dias, ou -1 para desligar |
| Coisas que param serverless auto-pause | Geo-replication, failover groups, long-term retention, elastic job database, Data Sync |
| Porto de endpoint público MI | 3342 |
| MI stop/start | Somente General Purpose. Você ainda paga pelo armazenamento |

**Escolhas rápidas de armazenamento VM**: Arquivos de dados no Premium SSD, Premium SSD v2 ou Ultra Disk. Leia o cache **E-mail** para discos de dados, **fora** para discos de log. tempdb no local **D:** unidade (é limpo no reinício). Padrão HDD é para backups Apenas. Tamanhos de produção: memória otimizada, por exemplo **Ebdsv5**. Constrained-core VMs manter o RAM e velocidade do disco, mas corte SQL Custo da licença.

## 2. IaaS e PaaS: responsabilidades

**Segurança**

| Área | VM (IaaS) | SQL DB / MI (PaaS) |
|---|---|---|
| Tipos Login | SQL auth, Windows auth (Active Directory) | SQL auth, **Microsoft Entra ID** (com MFA). Auth do Windows para usuários do Entra somente no **MI** |
| TDE (criptografa os arquivos no disco) | Você a liga | **Por padrão** |
| Rede | NSGs, Firewall do Windows, VPN / ExpressRoute | Regras de firewall de servidor e banco de dados (SQL DB), service endpoints, Private Link. MI usa NSGs |
| Patching | Você | Microsoft |
| Detecção de ameaças | Defender para Servidores SQL em Máquinas | Defender para Azure SQL Databases |

**Automação e implantação**

| Área | VM (IaaS) | SQL DB / MI (PaaS) |
|---|---|---|
| Instalar SQL | Você (ou uma imagem do Marketplace) | Azure faz isso |
| Remendos OS e SQL | Você. Use **Azure Update Manager** | Automático |
| Agendador Job | SQL Agent | MI: SQL Agent. SQL DB: **elastic jobs** |
| Deploy como código | ARM, Bicep, Terraform, PowerShell, CLI | Mais DACPAC em Azure DevOps / GitHub Actions |

**Desempenho**

| Área | VM (IaaS) | SQL DB / MI (PaaS) |
|---|---|---|
| Query Store | Ligue-se (SQL Server 2016–2019). Por padrão, para o novo DBs em SQL Server 2022+ | Por padrão. Não pode ser desligado em SQL DB |
| MAXDOP | `sp_configure` (Instância) | SQL DB: `ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP` |
| Escala acima | Redimensionar o VM (VM reinicia) | Alterar vCores ou nível em segundos a minutos |
| Contadores OS (PerfMon) | Sim | Não |
| Auto tuning | Não | Sim (correção do plano, ajuste de índice em SQL DB) |
| Index and statistics maintenance | Você | Você ainda (backups e DBCC verificações são feitas pela plataforma) |

**Backups**

| Área | VM (IaaS) | SQL DB | MI |
|---|---|---|---|
| Quem faz backup | Você (ou Azure Backup / IaaS Agent) | Azure, automático | Azure, automático |
| Seu próprio backups | Sim: completo, diferencial, log | Não | Sim, mas apenas `COPY_ONLY` a URL |
| Ponto de restauração no tempo | Sim, se modelo FULL recovery e log backups | 1–35 dias (padrão 7) | 1–35 dias (padrão 7) |
| Long-term retention | Azure Backup ou seu próprio armazenamento | Até 10 anos | Até 10 anos |

**HA/DR**

| Área | VM (IaaS) | SQL DB / MI (PaaS) |
|---|---|---|
| HA em uma região | Always On AG, FCI (você o constrói) | A redundância de zona é uma opção |
| DR para outra região | AG réplica async, distribuído AG, log shipping, ASR, backups em armazenamento GRS | SQL DB: active geo-replication ou failover groups. MI: failover groups (ou link MI) |
| Connection string após DR failover | Você corrige DNS ou a string | Mesmo nome com failover group listener |

## 3. escolha da ferramenta de migração

**Passo 1: avaliar.** Use **Azure Migrate** ou a avaliação **SQL Server enabled by Azure Arc**.

| Fonte → alvo | online / offline | Ferramenta | Factos-chave |
|---|---|---|---|
| SQL Server → VM | Offline | **Azure Migrate** (mover todo o servidor) | Lift-and-shift |
| SQL Server → VM | Offline | **Backup a URL + restaurar** | `BACKUP DATABASE ... TO URL` |
| SQL Server → VM | Online | **Log shipping** ou **Always On AG** | Adicione a réplica do Azure com **T-SQL** (o Assistente de Replica Adicionar Azure foi para o clássico VMs, aposentado) |
| SQL Server → VM | Offline | **Detach/attach**, **Serviço de Importação/Exportação Azure** (discos de navio) | Movimentos de rede muito grandes ou lentos |
| SQL Server → MI | Online | **Managed Instance link** | SQL Server 2016+. Cutover em segundos. Alvo legível. Sem limite de tempo. failback bidirecional com SQL Server 2022/2025 |
| SQL Server → MI | Online | **Log Replay Service (LRS)** | SQL Server 2012+, todas as edições. Deve terminar em **30 dias**. Alvo não legível. Modos: Autocomplete ou Contínuo |
| SQL Server → MI | Online ou offline | **Azure DMS** (portal, CLI `az datamigration`, PowerShell) | Movimentos grandes e complexos |
| SQL Server → MI | Offline | **Nativo backup /restaurar** `RESTORE ... FROM URL` | Simples simples |
| SQL Server (Arc) → MI | Online | **Migração de arco no portal** | Usa link MI ou LRS dentro |
| SQL Server → SQL DB | Online | **Transactional replication** | O caminho **apenas online** em SQL DB. As tabelas precisam de uma chave primária. SQL DB é apenas um **assinante push** |
| SQL Server → SQL DB | Offline | **Azure DMS** | grandes bancos de dados |
| SQL Server → SQL DB / MI | Offline | **BACPAC + SqlPackage** | Mais de ~150 GB, use SqlPackage, não o portal. MI não pode importar um BACPAC do portal |
| Algumas tabelas apenas | Offline | **bcp**, **Azure Data Factory** | Cópias parciais ou agendadas |
| SQL DB → MI | Offline | **Azure DMS** | Entre serviços Azure SQL |

**Ferramentas aposentadas** (o exame ainda pode nomeá-los):

| Ferramenta | Aposentado | Use em vez disso |
|---|---|---|
| Data Migration Assistant (DMA) | Julho 2025 | Avaliação Azure Migrate ou Arc, DMS |
| Azure Data Studio e sua extensão de migração | 28 Fev 2026 | DMS (portal/ CLI / PowerShell), SSMS |
| Assistente de Experimentação de Banco de Dados (DEA) | 15 Dez 2024 | n/a |

**Dicas de migração**: Escolha on-line quando o tempo de inatividade deve estar próximo de zero (hospital, loja 24/7). Escale o alvo durante o movimento e depois diminua. Após a migração, verifique se há regressões no plano (novo cardinality estimator), índices ausentes e parameter sniffing.

## 4. picaretas rápidas de segurança

| Necessidade | Escolha |
|---|---|
| Uma identidade, MFA, logon único | Autenticação **Microsoft Entra ID** |
| Desligue o SQL logins completamente | **Autenticação somente Microsoft Entra** (criar um administrador de entrada primeiro) |
| App se conecta sem senha | **Managed identity** |
| Usuário que se move com o banco de dados | **Contained user** (`CREATE USER ... FROM EXTERNAL PROVIDER`) |
| Papéis de nível de servidor em SQL DB | Funções `##MS_...##` fixas (por exemplo `##MS_DatabaseManager##`, `##MS_LoginManager##`), definidas com T-SQL em `master` |
| Criar bancos de dados, mas não ler dados | **Contribuidor SQL DB** (Azure RBAC) |
| Gerenciar configurações de segurança, mas não ler dados | **SQL Security Manager** |
| Bloquear o acesso, mesmo que concedido em outro lugar | `DENY` (batidas `GRANT`). `REVOKE` apenas remove uma concessão ou negar |
| Criptografar banco de dados inteiro e backups em repouso | **TDE** (AES-256). Key in Key Vault = **chave gerenciada pelo cliente (BYOK)** |
| Ocultar dados da coluna até mesmo de DBAs | **Always Encrypted** (as chaves ficam com o aplicativo cliente) |
| Always Encrypted, precisa de associações / igualdade | encriptação **determinística** |
| Always Encrypted, mais seguro | **Aleatorizado** (consultas ricas precisam de **secure enclaves**) |
| Criptografar dados na rede | **TLS** (`Encrypt=True;TrustServerCertificate=False`). O mínimo TLS é agora **1.2** |
| Mostrar `a****@g****.com` call-centre staff | **Dynamic Data Masking**. Conceda `UNMASK` para pessoas que precisam de valores reais |
| Cada inquilino vê apenas suas próprias linhas | **Row-Level Security** (política de segurança + função predicado) |
| Colunas de etiquetas como Confidencial, GDPR, etc. | **Descoberta e classificação de dados** (`ADD SENSITIVITY CLASSIFICATION`) |
| Mapa de toda a Org de dados sensíveis e linhagem | **Microsoft Purview** |
| História à prova de tampões para auditores | Tabelas **Ledger** (digestos em Blob imutável ou Azure Confidential Ledger) |
| Log quem fez o quê | **Auditoria** para Armazenamento (barato), Log Analytics (query), Event Hubs (tempo real) |
| Alertas sobre injeção SQL, força bruta, ímpar logins + varredura de vulnerabilidade | **Microsoft Defender for SQL** |
| Mantenha a história das linhas ao longo do tempo | **Tabelas temporais** |
| Saiba quais linhas mudaram (sem valores antigos) | **Change tracking** |
| Mantenha o histórico completo de mudanças para ETL | **Change data capture (CDC)** |
| Chaves e segredos da loja | **Azure Key Vault** (papel **Key Vault Cripto Cripto Serviço de Criptografia Usuário**) |
| Injeção Stop SQL | Consultas parametrizadas ou procedimentos armazenados (filtros de palavras-chave não são suficientes) |

**Picaretas rápidas de rede**

| Necessidade | Escolha |
|---|---|
| Toda a empresa atinge DBs em um servidor | **Regra de firewall no nível do servidor** (`sp_set_firewall_rule` em `master`) |
| Um parceiro chega a um banco de dados | **Regra de firewall de nível de banco de dados** (`sp_set_database_firewall_rule`) |
| Apenas uma sub-rede VNet pode se conectar, endpoint público ainda existe | **Service endpoint** |
| Nenhum acesso público, privado IP dentro do VNet | **Private Link** (private endpoint) |
| Túnel criptografado pela internet, baixo custo | **Gateway VPN** |
| Fibra privada, alta velocidade, conformidade | **ExpressRoute** |
| Menor latência dentro do Azure | Política de conexão **redirecionamento** (portas 11000–11999). **Proxy** usa apenas 1433 |

Ordem de verificação de firewall: regra do banco de dados primeiro, depois regra do servidor e depois bloquear. Limites: 256 regras de nível de servidor e 256 regras IP de nível de banco de dados. "Permitir serviços do Azure" faz uma regra `0.0.0.0` que permite entrar nos serviços do cliente do Azure **Cada**.

**TDE ordem de construção (VM)**: Chave mestre de serviço → `CREATE MASTER KEY` → `CREATE CERTIFICATE` → `CREATE DATABASE ENCRYPTION KEY` → `ALTER DATABASE ... SET ENCRYPTION ON`. Para um AG, copie o certificado para cada réplica.

## 5. Desempenho: Query Store, planos, esperas, DMVs

**Query Store** (um gravador de consultas, planos e estatísticas; sobrevive reinicializações, ao contrário de DMVs).

| Configuração | Padrão | Lembre-se |
|---|---|---|
| Modo de operação | READ_WRITE | Vai **READ_ONLY** quando o tamanho máximo está cheio |
| Intervalo de dados flush | 15 min (900 s) | Com que frequência a memória é escrita no disco. O intervalo ligado ao **pressão de escrita** |
| Intervalo de coleta de estatísticas | 60 min | Apenas o tamanho dos baldes de tempo. Não vinculado ao read-only |
| Tamanho máximo | 1,000 MB | Aumentar se Query Store continuar read-only |
| Limiar de consulta | 30 dias | Dados antigos são limpos |
| Modo de captura | AUTO | Ignora pequenas consultas pontuais |

- Query Store read-only → aumentar **Tamanho máximo**, manter a limpeza baseada em tamanho ou diminuir o limite obsoleto.
- "Qual intervalo afeta a pressão de escrita?" → **Intervalo de dados flush**, não o intervalo de estatísticas.
- O plano piorou depois de uma mudança → Query Store **Consultas Regressadas**, então **força** o plano antigo (`sp_query_store_force_plan`).
- Corrigir a consulta de um aplicativo do fornecedor sem alterar seu código → **Query Store hints** (`sp_query_store_set_hints`).
- Automatic tuning precisa de Query Store **ON** e **READ_WRITE**:

```sql
ALTER DATABASE [Db1] SET QUERY_STORE = ON;
ALTER DATABASE [Db1] SET QUERY_STORE (OPERATION_MODE = READ_WRITE);
ALTER DATABASE [Db1] SET AUTOMATIC_TUNING = AUTO;
```

**padrão Automatic tuning**: FORCE_LAST_GOOD_PLAN **E-mail**. CREATE_INDEX e DROP_INDEX **fora**. MI suporta **apenas** FORCE_LAST_GOOD_PLAN. Sugestões estão em `sys.dm_db_tuning_recommendations`.

**Plano estimado vs. plano real**

| | Plano estimado | Plano real |
|---|---|---|
| Quando | Antes da consulta ser executada | Depois de correr |
| Mostra | Linhas e custos estimados | Estimado **e** linhas reais, derramamentos, avisos |
| Risco | Nenhum | A consulta realmente corre |
| Chave SSMS | Ctrl+L | Ctrl+M |

Linhas estimadas longe das linhas reais → **estatísticas stale** ou um plano ruim. Execute `UPDATE STATISTICS`.

**Problemas no plano →**

| Problema | Correção |
|---|---|
| Tabela ou clustered index **scan** para um pequeno resultado | Adicionar um nonclustered index na coluna de filtro |
| **Key lookup** | Covering index with `INCLUDE` colunas |
| Filtro não SARGable (`WHERE CAST(col ...)`) | Reescrever para que a coluna não seja envolvida em uma função |
| Parameter sniffing (rápido por um valor, lento por outro) | `OPTION (RECOMPILE)`, `OPTIMIZE FOR UNKNOWN` ou **Otimização PSP** (compat 160) |
| Bloqueio de leitores e escritores | **RCSI** ou snapshot isolation (RCSI está ativado por padrão em SQL DB) |
| Contenção de inserção de última página (`PAGELATCH_EX` na chave IDENTITY) | `OPTIMIZE_FOR_SEQUENTIAL_KEY = ON` |
| Reconstrução de índice longo que deve sobreviver failover | Reconstrução do índice **Resumível** (`RESUME`, `ABORT`, `MAX_DURATION`) |
| scans de análise | **Columnstore** índice |

**Intelligent Query Processing (IQP)** liga, aumentando **Nível de compatibilidade** (140/150 / 160/170). Novos bancos de dados SQL DB padrão para **170**. Características: adaptive joins, memory grant feedback, scalar UDF inlining (150), table variable deferred compilation, contagem aproximada distinta, otimização PSP, feedback DOP e feedback CE (160).

**Wait types**

| Esperar | Meios |
|---|---|
| `PAGEIOLATCH_SH` | leituras lentas do disco |
| `PAGELATCH_UP` em tempdb | Tempdb atribuição contenção (sem disco I/O) → mais arquivos Tempdb |
| `RESOURCE_SEMAPHORE` | Consultas à espera de memória |
| `LCK_M_X` / `LCK_M_*` | Bloqueio (bloqueio) |
| `CXPACKET` | Paralelismo para sintonizar (MAXDOP, limite de custo). `CXCONSUMER` é normal |
| `LOG_RATE_GOVERNOR` | SQL DB está limitando a velocidade de gravação de log → transações menores ou maior nível |
| `INSTANCE_LOG_RATE_GOVERNOR` | Limite de log da instância MI |
| `THREADPOOL` | Não basta de linhas de trabalho |
| `HADR_SYNC_COMMIT` | Business Critical esperando pela sincronização da réplica |
| `HADR_THROTTLE_LOG_RATE_MISMATCHED_SLO` | Geo-secundário é menor do que o primário |

Erro **10928** E- SQL DB = worker limit reached → escala acima.

**Top DMVs** (permissão: `VIEW DATABASE STATE` em SQL DB, `VIEW SERVER STATE` em MI / VM)

| DMV | O que mostra | Use-o quando |
|---|---|---|
| `sys.dm_db_resource_stats` | CPU, I/O de dados, E/O de log a cada **15 s**, cerca de **1 hora** mantido (SQL DB) | "O meu banco de dados está no limite agora?" |
| `sys.resource_stats` (em `master`) | Médias **5 minutos** para cada DB no servidor, cerca de 14 dias | Planejamento da capacidade ao longo dos dias |
| `sys.elastic_pool_resource_stats` | Piscina CPU, I/O, memória | Tamanho direito um elastic pool |
| `sys.dm_user_db_resource_governance` | Limites do seu nível de serviço | Verifique o que o SKU permite |
| `sys.dm_exec_requests` | Consultas em execução agora, suas esperas e bloqueadores | Localizadores e corredores longos |
| `sys.dm_exec_sessions` | Quem está conectado, programa, login tempo | Aplicativos barulhentos Spot ou estranho logins |
| `sys.dm_exec_connections` | Cliente IP, protocolo, encriptado ou não | Conexões de verificação são criptografadas |
| `sys.dm_exec_input_buffer` | Last statement a session sent | O que essa sessão fez? |
| `sys.dm_exec_sql_text` | Texto de consulta completo a partir de um identificador | Transformar uma alça em SQL legível |
| `sys.dm_exec_query_plan` | Plano em cache XML | Ver exames, pesquisas, derramamentos |
| `sys.dm_exec_query_stats` | CPU, lê, escreve por plano em cache | Principais consultas CPU ou I/O desde o início do cache |
| `sys.dm_exec_cached_plans` | Usar e reutilizar o cache do plano | Muitos planos ad-hoc de uso único |
| `sys.dm_exec_query_plan_stats` | Last actual plan (opt-in: `LAST_QUERY_PLAN_STATS = ON`) | Plano real sem re-execução |
| `sys.dm_db_wait_stats` | Espera por este banco de dados (SQL DB) | Top esperas em SQL DB |
| `sys.dm_os_wait_stats` | Espera pela instância (MI, VM) | Aguarda no MI ou VM |
| `sys.dm_tran_locks` | Fechaduras realizadas e desejadas | cadeias de bloqueio |
| `sys.dm_io_virtual_file_stats` | Latência de leitura/gravação por arquivo | Armazenamento lento, lento AG redo |
| `sys.dm_db_index_usage_stats` | Buscas, exames, pesquisas, atualizações por índice | Encontre índices não utilizados para soltar |
| `sys.dm_db_index_physical_stats` | Fragmentação | Reorganizar vs reconstruir |
| `sys.dm_db_missing_index_details` / `_group_stats` | Índices sugeridos e impacto estimado | Qual índice criar primeiro |
| `sys.dm_db_session_space_usage` | páginas tempdb por sessão | Encontre o porco tempdb |
| `sys.dm_db_log_space_usage` | Registro usado vs total | Grande transação preenchendo o log |
| `sys.dm_db_tuning_recommendations` | Ações e Estado Automatic tuning | O ajuste automático corrigiu a regressão? |
| `sys.dm_database_encryption_keys` | Estado TDE por banco de dados | O TDE está ligado depois de uma restauração? |
| `sys.dm_operation_status` (em `master`) | Operações longas: restauração, cópia, escala | Recuperação ou cópia do progresso |
| `sys.dm_geo_replication_link_status` | Estado Geo-replication e atraso | Verifique a geo-rep antes de failover |
| `sys.dm_database_backups` | História Backup (SQL DB) | Confirme backups para restaurar perguntas |
| `sys.dm_hadr_database_replica_states` | Estado de sincronização AG, redo, lag | Replica antes de um planned failover |

**Fluxo de solução de problemas rápido**: App lento → Verificar `sys.dm_db_resource_stats`. Alto CPU → Query Store consultas superiores. Normal CPU → wait stats. Top consultas no portal (SQL DB apenas) → **Query Performance Insight**. Rastreamento profundo (deadlocks) → **Extended Events** em SQL DB utilizar `ON DATABASE` Sessões de monitoramento). Painel de controle para muitos SQL DB/MI → **Database watcher** (dados vão para o Azure Data Explorer ou Fabric Real-Time Intelligence; não para VMs).

**temdb**: VM = one file per core up to 8, same size. SQL DB = set by Azure, cannot change. MI = 12 files by default, can change.

## 6. picaretas rápidas da automatização

| Ferramenta | Onde corre | Melhor para | Exemplo |
|---|---|---|---|
| **SQL Server Agent** | VM, MI (não SQL DB) | Programação T-SQL jobs em uma instância | Nightly index rebuild |
| **Elastic jobs** | SQL DB (todos os níveis) | Mesmo T-SQL em muitos bancos de dados, entre servidores, piscinas, assinaturas | Adicionar uma coluna a 30 locatário DBs |
| **Azure Automation** runbooks | ou on-prem via Hybrid Runbook Worker | scripts PowerShell / Python em um cronograma | Iniciar VM, executar um procedimento, parar VM |
| **Logic Apps** | Azure (serverless, low-code) | Fluxos orientados a eventos que ligam muitos serviços | Nova ordem linha → enviar e-mail |
| **Azure Functions** | Azure (código serverless) | Pequeno código acionado por eventos ou alertas | Fix job chamado por um alerta |
| **Alerta Azure Monitor + grupo de ação** | Azure | Reagir a métricas ou logs | E-mail CPU sobre 90% → + runbook |
| **Maintenance Plan Wizard** (SSMS) | VM, MI | Backups, índice e estatísticas jobs sem scripting | Armazenado como pacotes SSIS, jobs em `msdb` |
| **Multi-servidor (mestre/alvo) jobs** | VM | Empurrar um job para muitos servidores | 50 servidores, um job |

**Fatos Elastic jobs**: Parts = job agente, **banco de dados job** (lojas jobs), target group, job Passos (T-SQL). Job deve ser novo e vazio; não Hyperscale, não auto-pause serverless. Best auth = **user-assigned managed identity** ou database-scoped credentials). Níveis do agente JA100 (default) a JA800. História mantida **45 dias**. Os scripts devem ser **idempotent** (seguro para correr duas vezes). Alerta sobre falhas com o **Falha Elastic jobs** métrica.

**Fatos do agente**: No MI, o Agente está sempre em execução e suas configurações são read-only. MI Database Mail perfil deve ser nomeado **`AzureManagedInstance_dbmail_profile`**. SQL Agent **Alertas** não são suportados no MI (apenas job notificações de e-mail) → usar **Azure Monitor alerts**. Use um **Conta proxy** quando um passo job precisa de outros direitos.

**Deploy como código**

| Necessidade | Escolha |
|---|---|
| Implantação repetitiva, estado desejado (declarativo) | **Bicep** (idioma recomendado pela Microsoft) ou ARM JSON |
| Mudança rápida, passo a passo (imperativa) | PowerShell ou Azure CLI |
| Transforme ARM JSON em Bicep | `az bicep decompile` |
| Corra em cada push de código | Pipelines Azure DevOps ou GitHub Actions |
| Schema muda como um pacote | DACPAC (SqlPackage) |
| Nenhuma instalação local | Azure Cloud Shell |

**Patching**: PaaS é automático. VM → **Azure Update Manager** (SQL IaaS Agent Automated Patching se aposenta em 17 de setembro de 2027).

## 7. HA/DR escolhas rápidas

**RPO** = how much data you may lose (time). **RTO** = how long you may be down. If backups correr a cada hora, mas RPO é de 5 minutos, você falha. Use log frequente backups ou réplicas.

**Option chooper**

| Necessidade | Escolha | Perda de dados / failover |
|---|---|---|
| HA dentro de uma região, PaaS | Built-in HA (nada para fazer) | Automático |
| Sobreviver a uma falha de zona, PaaS | **redundância de zona** | Automático |
| Cópias legíveis em outras regiões, SQL DB, você falha | **Active geo-replication** (até **4** secundários) | Async, manual, novo connection string |
| Grupo de DBs falham juntos, mesmo connection string | **Failover group** (SQL DB ou MI) | Async, nomes listener permanecem os mesmos |
| DR para **MI** | **Failover group** (MI não tem geo-replication) | Um grupo por instância, todo o usuário DBs |
| HA para algumas bancos de dados sobre VMs + secundários legíveis | **Always On AG** | Sync = no data loss in region |
| Proteja toda a instância (logins, jobs) em VMs | **FCI** (necessidade de armazenamento compartilhado) | uma cópia automática dos dados |
| AGs em duas regiões, cada uma com seu próprio cluster | **AG distribuído** | cross-região |
| Barato, simples DR, manual | **Log shipping** | Alguma perda de dados, manual |
| Copie VMs inteiro (app + SQL) para outra região | **Azure Site Recovery** | Não SQL -ciente |
| On-prem SQL Server com DR no Azure | AG a uma Azure VM, ou **link MI** (duas vias com SQL Server 2022/2025) | Mais de VPN ou ExpressRoute |

**Geo-replication vs failover groups**

| | Active geo-replication | Failover groups |
|---|---|---|
| Funciona em | Somente SQL DB | SQL DB e MI |
| Scope | Um banco de dados | Muitos DBs (SQL DB); todos os usuários DBs (MI) |
| Secundários | Até 4 legível | Um (mais é visualização para SQL DB) |
| Conexão após failover | Alterar o connection string | Mesmos nomes **read-write** e **read-only listener** |
| Automatic failover | Não | Somente com a política **Microsoft gerenciada** (período de graça min **1 hora**). **Cliente gerido** é recomendado |
| Verificar a saúde | `sys.dm_geo_replication_link_status` | O mesmo, mais o portal |

Regra: secundário deve usar o **mesmo nível de serviço**. O mesmo tamanho de computação é fortemente recomendado.

**Números SLA**: SQL DB 99.99%. Zona-redundante Business Critical / Premium 99.995%. VMs em availability zones 99.99%. VMs em conjuntos de disponibilidade 99.95%. Redundância de zona não está nos níveis Básico ou Padrão.

**Clusters em VM.** Use **Cloud Witness** para quorum, ou disk witness quando a FCI usa Azure shared disks. Coloque as réplicas em **várias sub-redes** para dispensar o load balancer. Em uma sub-rede, use um listener **DNN**, ou VNN com um Load Balancer **Standard**. Basic foi descontinuado em 30 de setembro de 2025. Para shared storage de FCI, use Azure shared disks, Premium file shares, Storage Spaces Direct ou Elastic SAN (2022+). Um AG aceita até 9 réplicas. Os **contained AGs do SQL Server 2022** levam logins e jobs do Agent com o AG.

**Redundância de armazenamento**

| Tipo | Cópias | Sobrevive | Leu a cópia? |
|---|---|---|---|
| **LRS** | 3 em um datacenter | Falha de disco ou rack | n/a. Mais barato. Não para DR |
| **ZRS** | 3 zonas em uma região | Falha na zona, não região | n/a |
| **GRS** | 6: 3 local + 3 na região pareada | Falha na região | **Não**, somente após failover |
| **RA-GRS** | mesmo que GRS | Falha na região | **Sim**, a qualquer hora |
| **GZRS** | Zonas na região primária + pareada | Falha na zona e região | Não (RA-GZRS = yes) |

O armazenamento PaaS backup é **geo-redundante por padrão**. Você pode escolher LRS, ZRS, GRS ou GZRS. **Geo-restore** funciona apenas com GRS ou GZRS. Para Hyperscale, escolha redundância backup apenas na criação.

**Números PaaS backup**

| Item | Valor |
|---|---|
| Full backup | Semanal |
| Diferencial | A cada 12 h (vCore) ou 24 h (DTU) |
| Log backup | Sobre cada 5–10 min |
| Janela Point-in-time restore | **dias 1–35**, padrão **7** (máx. básico 7) |
| Long-term retention (LTR) | Semanal, mensal, anual, até **10 anos** |
| Hyperscale | Armazenamento **snapshots**, não completo/diff/log |
| Resultado da restauração | Sempre um banco de dados **Novo** (sem restauração no lugar) |
| Servidor lógico excluído | O PITR backups desapareceu. **LTR backups ficar** |
| Apagado managed instance | Entre em contato com suporte dentro de 5 dias |
| SQL DB backup → SQL Server | Não é possível |
| MI copy-only backup → SQL Server 2022/2025 | Possível quando a política de atualização MI corresponder |
| SQL Server backup → MI | `RESTORE ... FROM URL` |
| MI `COPY_ONLY` com chave TDE gerenciada por serviços | Não permitido. Use uma chave gerenciada pelo cliente |

**Opções VM backup**

| Opção | Fatos |
|---|---|
| **Azure Backup para SQL em Azure VM** | Abóbada de serviços de recuperação, log backups cada **15 min**, point-in-time restore de um DB, long-term retention |
| **Automated Backup** (SQL IaaS Agent extension) | Retenção **dias 1–90**, registra cada minuto 5–60, precisa do modelo FULL recovery, SQL Server 2016+ |
| **Backup a URL** | `BACKUP DATABASE ... TO URL`. Use **SAS** + blocos de bolhas. Um bloco de bolha ~195 GB → até 64 URLs |
| **VM snapshot** | VM. Bom para ransomware. Não para restaurar um banco de dados |

Modelos de recuperação: **FULL** = point-in-time restore. **SIMPLE** = no log backups, sem point-in-time. **BULK_LOGGED** = faster bulk loads, partial point-in-time. Do **E** mistura backup ferramentas em um banco de dados: quebra a cadeia de log.

## 8. Principais armadilhas de exame

- Azure SQL Database tem **SQL Agent**. Use elastic jobs. MI e VM têm agente.
- MI tem **active geo-replication**. Sua região cruzada DR é um **failover group**.
- Um MI com um **link MI** não pode estar em um failover group.
- Geo-replication failover precisa de um **Mudança connection string**. Failover groups manter o nome **listener**.
- Failover group "automático" (Microsoft gerenciado) espera pelo menos **1 hora** e é apenas para grandes interrupções. O cliente gerenciado é recomendado.
- A restauração PaaS sempre cria um banco de dados **Novo**. Você não pode restaurar o antigo.
- PITR é **dias 1–35**. Qualquer coisa mais longa é **LTR**, até 10 anos.
- Excluindo o servidor lógico exclui PITR backups, mas **LTR backups sobreviver**.
- SQL DB backups **não pode** ser restaurado para SQL Server.
- MI permite apenas o usuário **COPY_ONLY** backups.
- Geo-restore precisa de armazenamento **GRS ou GZRS** backup. LRS e ZRS não podem geo-restore.
- GRS secundário é **não legível** até failover. RA-GRS é legível o tempo todo.
- TDE é **por defeito** em PaaS e deve ser ligado por você em um VM.
- TDE faz **E** esconder dados de DBAs. **Always Encrypted** faz.
- O Always Encrypted determinístico permite que a igualdade se junte. Aleatorizado é mais seguro, mas precisa de **enclaves** para consultas ricas.
- `DENY` bate `GRANT`. `REVOKE` é neutro.
- Usuários com `CONTROL` (db_owner) sempre ver dados **desmascarado** sob Dynamic Data Masking.
- As regras de firewall do banco de dados são verificadas nas regras do servidor **antes**. O MI usa o **NSGs**, não as regras de firewall IP.
- **Service endpoint** mantém o endpoint público. **Private Link** remove a exposição pública.
- Windows authentication para usuários do Entra funciona no **Somente MI**, não no SQL DB.
- `EXECUTE AS LOGIN` funciona apenas em SQL Server e MI. `EXECUTE AS USER` funciona em todos os lugares.
- O SQL dinâmico (`sp_executesql`) quebra o **a propriedade da cadeia**. O usuário precisa de direitos diretos.
- Automatic tuning precisa de Query Store **ON + READ_WRITE**.
- Query Store read-only = it is **Cheio**. "Write pressure interval" = **intervalo de dados flush**.
- Query Performance Insight é **Somente SQL DB**, não MI.
- Em SQL DB utilizar `sys.dm_db_wait_stats` e `sys.dm_db_resource_stats` (15 s, 1 h). `sys.resource_stats` é de 5 min e vive em `master`.
- MAXDOP em SQL DB é um **banco de dados scoped configuration**, não `sp_configure`. Novo padrão SQL DB é MAXDOP 8.
- As esperas `CXCONSUMER` são normais. Sintonize `CXPACKET`.
- `PAGELATCH` indica contenção em páginas na memória, muitas vezes no tempdb. `PAGEIOLATCH` indica I/O de disco lento.
- RCSI é **por defeito** em SQL DB, não em SQL Server ou MI.
- O plano estimado não executa a consulta. O plano real faz.
- BACPAC é **offline**. Sobre ~150 GB uso **SqlPackage**. MI não pode importar um BACPAC do portal.
- Transactional replication é a migração **apenas online** para SQL DB, e as tabelas precisam de um **chave primária**.
- LRS (Log Replay Service) deve terminar em **30 dias** e o alvo não é legível. MI link não tem limite de tempo e é legível.
- DMA (julho de 2025), Azure Data Studio (28 de fevereiro de 2026) e DEA (dezembro de 2024) são **aposentado**. Avalie com Azure Migrate ou Arc.
- Hyperscale pode voltar para General Purpose apenas dentro de **45 dias**.
- Azure Site Recovery copia VMs inteiro; é **E SQL-aware**.
- Discos de dados VM: ler cache **E-mail**. Discos de registro: cache **fora**. tempdb em **D:** (desligado no reinício).
- O Load Balancer básico é aposentado. Use **Padrão**, um DNN listener, ou várias sub-redes.
- Elastic job database não pode ser **Hyperscale** ou **auto-pause serverless**.
