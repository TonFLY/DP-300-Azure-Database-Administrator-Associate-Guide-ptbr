# Monitor, teste e solução de problemas HA/DR

> **Habilidades cobradas:** Planear um procedimento de teste para uma solução HA/DR, Monitorar uma solução HA/DR, Solucionar problemas uma solução HA/DR
> **Tempo de leitura:** ~13 min

## Conceito central
An HA/DR o design é apenas uma teoria até que você falhe, meça e falhe de volta. Você **ensaio** com brocas, **monitor** para que você veja o atraso antes que doe, e **Solução de problemas** quando as réplicas ficam para trás ou os failovers falham.

Pense em um exercício de fogo em uma escola. Você toca a campainha, tempo leva para tirar todos para fora, verifique se ninguém está faltando e conserte a porta bloqueada que você encontrou.

## Planejar um procedimento de teste (erro DR)
| etapa | O que fazer |
|---|---|
| 1. Scope | Lista de todas as partes: banco de dados, nível de aplicativo, DNS, armazenamento, logins, jobs, firewall / NSG regras |
| 2. Linha de base | Recorde de atraso atual e o alvo RPO / RTO |
| 3. Escolha o tipo de teste | **Planejado** failover (seguro em produção) ou **forçado** (apenas ambiente de teste) |
| 4. Executar | Execute o failover. Anote os tempos de início e fim |
| 5. Validar | O aplicativo se conecta através do **listener**, trabalho logins, jobs executar, lê e escreve com sucesso |
| 6. Gravar | RTO, real RPO, problemas, correções runbook |
| 7. Falha de volta | Use um **planejado** failover para retornar |

- **Planned failover** = sync first, **sem perda de dados**. Use-o para brocas na produção, regiões em movimento e **failback**.
- **Forced failover** = switch now, **possível perda de dados**. Apenas para uma interrupção real quando o primário é inalcançável.
- **broca Geo-restore**: use um **cópia** de produção. Simule a interrupção (por exemplo, renomeie o banco de dados de origem), geo-restore para outro servidor e renomeie o aplicativo.
- **broca ASR**: execute um **teste failover** de um plano de recuperação em uma rede de testes isolada. A produção continua funcionando.
- Executa brocas **regularmente**. Muitos padrões da indústria exigem-los.

**Exemplo:** Uma empresa de seguros administra a DR A cada trimestre. Faz um planned failover da sua failover group, verifica o aplicativo, notas RTO = 4 minutes, then fails back.

## Teste HA local em SQL Database e SQL MI
A **Iniciado pelo usuário failover** o banco de dados move-se para outra réplica para que você possa testar **retry lógica**.

| Alvo | PowerShell | Azure CLI |
|---|---|---|
| Banco de dados único | `Invoke-AzSqlDatabaseFailover` | Não `az sql db failover`. Uso `az rest` |
| Elastic pool | `Invoke-AzSqlElasticPoolFailover` | `az rest` |
| Managed Instance | `Invoke-AzSqlInstanceFailover` (`-ReadableSecondary` para BC) | `az sql mi failover` (`--replica-type ReadableSecondary`) |

```powershell
Invoke-AzSqlDatabaseFailover -ResourceGroupName rg-prod -ServerName sql-prod-weu -DatabaseName SalesDb
Invoke-AzSqlInstanceFailover -ResourceGroupName rg-prod -Name mi-prod-weu
```

- Limite: **um failover a cada 15 minutos** por banco de dados, elastic pool ou managed instance.
- Portal: **Configuração → Manutenção → Reiniciar** (preview) usa o mesmo API. Reiniciando um **pool reinicia todas as suas bancos de dados**.
- Não disponível para réplicas **Secundário legível Hyperscale**.
- MI failover é **bloqueado** até que o primeiro full backup de um novo banco de dados termine e enquanto um **restaurar** estiver em execução.
- Permissão MI: **Contribuidor SQL Managed Instance** (ou `Microsoft.Sql/managedInstances/failover/action`).

## Teste geo DR: geo-replication e failover groups
Execute-os contra o **secundário**.

| Recurso | Planejado (sem perda de dados) | Forçado (possível perda de dados) |
|---|---|---|
| Geo-replication, PowerShell | `Set-AzSqlDatabaseSecondary ... -Failover` | adicionar `-AllowDataLoss` |
| Geo-replication, CLI | `az sql db replica set-primary` | adicionar `--allow-data-loss` |
| Geo-replication, T-SQL (mestre do servidor secundário) | `ALTER DATABASE [SalesDb] FAILOVER;` | `ALTER DATABASE [SalesDb] FORCE_FAILOVER_ALLOW_DATA_LOSS;` |
| Failover group (SQL DB), PowerShell | `Switch-AzSqlDatabaseFailoverGroup` | adicionar `-AllowDataLoss` |
| Failover group (SQL DB), CLI | `az sql failover-group set-primary` | adicionar `--allow-data-loss` |
| Failover group (SQL MI) | `Switch-AzSqlDatabaseInstanceFailoverGroup` / `az sql instance-failover-group set-primary` | opção de add-data-loss |

```powershell
# -ServerName must be the SECONDARY server
Switch-AzSqlDatabaseFailoverGroup -ResourceGroupName rg-dr -ServerName sql-prod-neu -FailoverGroupName fog-sales
```

- `-TryPlannedBeforeForcedFailover` tenta primeiro planejado, depois forçado.
- Se o banco de dados for **em um failover group**, você não pode usar o comando geo-replication failover. Fail over the **grupo**.
- Verifique as funções com `Get-AzSqlDatabaseFailoverGroup` (`ReplicationRole`) ou `az sql failover-group show`.
- **Diário de atividades** mostra "Failover Azure SQL failover group". Se a Microsoft começou, **Evento iniciado por** mostra `-`.
- **brocas MI:** usar um **planejado** failover. Grupos MI não pode falhar enquanto **semeadura**. Um novo full backup corre após cada failover, então falhar de volta muito cedo pode ser adiado.

## Teste HA/DR em SQL Server VMs
```sql
-- Planned: run on the TARGET secondary (sync commit + SYNCHRONIZED)
ALTER AVAILABILITY GROUP [AG1] FAILOVER;
-- Forced (DR only): run on a secondary
ALTER AVAILABILITY GROUP [AG1] FORCE_FAILOVER_ALLOW_DATA_LOSS;
-- After a forced failover, secondary DBs are SUSPENDED: resume on each secondary
ALTER DATABASE [SalesDb] SET HADR RESUME;
```

- PowerShell: `Switch-SqlAvailabilityGroup` (adicione `-AllowDataLoss` à força). FCI: `Move-ClusterGroup` ou Failover Cluster Manager → Papéis → **Mover**.
- Prontidão de verificação: `is_failover_ready = 1` em `sys.dm_hadr_database_replica_cluster_states`, ou **Failover prontidão** no **painel Always On**.
- Antes de forçar, escolha o secundário com o **`end_of_log_lsn` mais alto** (menos perda de dados). **Bloquear clientes** do primário antigo para evitar **cérebro dividido**.
- Para falhar em uma réplica **async** sem perda de dados: alterne-a para **SYNCHRONOUS_COMMIT**, espere por **SYNCHRONIZED** e execute `FAILOVER`.

### Teste seu backups
O Abackup só é comprovado por uma restauração.
```sql
RESTORE VERIFYONLY FROM URL = 'https://stbackups.blob.core.windows.net/sql/SalesDb_full.bak' WITH CHECKSUM;
RESTORE DATABASE SalesDb_Test FROM URL = '...' WITH MOVE ..., STATS = 10;  -- on a test server
DBCC CHECKDB ('SalesDb_Test') WITH NO_INFOMSGS;                            -- the real proof
```
`RESTORE VERIFYONLY` verifica se o arquivo é legível. Faz **E** O banco de dados é consistente.

## Monitor: o que assistir
**Nível Azure**
- **Página de status do Azure**: interrupções globais (alimentação RSS).
- **Azure Service Health**: problemas, manutenção e interrupções para sua assinatura. Alertas por e-mail ou SMS.
- **Recursos Saúde**: saúde de um banco de dados ou VM e **Por que um failover aconteceu** (manutenção planejada ou falha).
- **Diário de atividades**: restaura, criação de banco de dados, operações failover.
- Status de CLI / PowerShell: `az sql db list`, `az sql mi list`, `Get-AzSqlDatabase`.

**DMVs de replicação** (dynamic management views, exibições de sistema que mostram o estado atual do servidor)
| Plataforma | DMV | Colunas-chave |
|---|---|---|
| AG em VM | `sys.dm_hadr_database_replica_states` | `synchronization_state_desc`, `synchronization_health_desc`, `is_suspended`, `log_send_queue_size`, `redo_queue_size`, `secondary_lag_seconds` |
| AG em VM | `sys.dm_hadr_availability_replica_states` | `role_desc`, `connected_state_desc` |
| Geo-replication / failover group | `sys.dm_geo_replication_link_status` (executar no **banco de dados do usuário** no primário; vazio em mestre) | `replication_state_desc`, `replication_lag_sec`, `last_replication` |
| SQL DB / réplicas MI | `sys.dm_database_replica_states` | `redo_queue_size`, `secondary_lag_seconds` |
| Log shipping | `sp_help_log_shipping_monitor`, `msdb.dbo.log_shipping_monitor_*` | último backup, copiar e restaurar arquivo e tempo |

- Dados **log enviar fila** ou `replication_lag_sec` → em risco (**RPO**).
- Big **redo fila** → mais lento failover (**RTO**).

**Monitoramento Backup**
- SQL Database: `sys.dm_database_backups` (vazio em Hyperscale). SQL MI: `msdb.dbo.backupset` ou Extended Events (XEvents).
- LTR backups: portal, `az sql db ltr-backup list`, `Get-AzSqlDatabaseLongTermRetentionBackup`.

**Exemplo:** Um aplicativo de varejo alerta quando o `replication_lag_sec` ultrapassa 5 segundos, porque seu RPO é de 5 segundos.

Aplicativos de design para disponibilidade também: mantenha o **aplicativo e banco de dados na mesma região** e adicione **retry lógica**.

## Solução de problemas AGs em Azure VMs
| Sintoma | Causa provável | Correção |
|---|---|---|
| Listener inacessível após failover (sub-rede única, VNN) | Sonda Load balancer não respondida | Set `ProbePort` no listener IP, ativar **Floating IP**, abra a porta de sonda (e.g. 59999) no firewall |
| Reconectar lentamente após failover | Cliente armazena em cache o antigo IP | `MultiSubnetFailover=True`. Else `RegisterAllProvidersIP=0`, `HostRecordTTL=300` |
| A conexão DNN listener falha | Driver não tem `MultiSubnetFailover` ou porto errado | Use um driver de suporte e o **porto DNN** no connection string |
| Erros **19407 / 19419** (a locação expirou), evento **1135** (nodo removido) | Monitoramento de cluster muito rigoroso para o Azure | Limites e tempos limite **Relaxar** (abaixo) |
| Erros **35201 / 35206** | Porta de endpoint, NSG ou firewall, ou tempo limite de sessão muito baixo | Verifique a porta **5022**, NSGs, ponto final. Raise `SESSION_TIMEOUT` |
| Banco de Dados **NOT SYNCHRONIZING / suspenso** | Forced failover, erro de disco/redo, ou o usuário suspendeu | Corrigir a causa, então `ALTER DATABASE ... SET HADR RESUME` |
| Cluster offline, **quórum perdido** | Votos perdidos ou testemunha | **Testemunha de nuvem**, número ímpar de votos. Emergência: `Start-ClusterNode -FixQuorum`, então force failover |
| "Não falhando sobre o grupo" | **Falhas máximas no período especificado** alcançado | Aumento de 2 para **6** |

Configurações relaxadas para o Azure:
```powershell
(Get-Cluster).SameSubnetThreshold = 40
(Get-Cluster).CrossSubnetThreshold = 40
Get-ClusterLog -Destination C:\Temp -TimeSpan 60 -UseLocalTime   # collect the cluster log
```
```sql
ALTER AVAILABILITY GROUP [AG1] SET (HEALTH_CHECK_TIMEOUT = 60000);  -- default 30000 ms
ALTER AVAILABILITY GROUP [AG1] SET (FAILURE_CONDITION_LEVEL = 2);   -- default 3
ALTER AVAILABILITY GROUP [AG1] MODIFY REPLICA ON 'SQLVM1' WITH (SESSION_TIMEOUT = 20); -- default 10 s
```
- **Tempo limite de locação**: default 20 s, aumentar para **40 s** (em Failover Cluster Manager).
- Onde procurar: log de erros SQL, sessão **AlwaysOn_health** Extended Events (criado pelo assistente, inicie você mesmo se você usou T-SQL), **painel Always On**, **log de cluster**, Azure **Diário de atividades** e **Recursos Saúde**.

**Exemplo:** A 2-node AG continua falhando durante a noite com erro 19407. O DBA eleva `SameSubnetThreshold` para 40 e o tempo limite de arrendamento para 40 s. Os failovers falsos param.

## Solução de problemas geo-replication e failover groups
- **Lag continua crescendo** (`replication_lag_sec` para cima, `redo_queue_size` subindo): geralmente o **secundário é menor** do que o primário. Wait type `HADR_THROTTLE_LOG_RATE_MISMATCHED_SLO`. Corrigir: **mesmo nível e tamanho de computação**. Outras causas: grandes reconstruções de índice, cargas a granel, consultas longas no secundário legível.
- `replication_lag_sec` é **NULL** por um longo tempo → primário não pode chegar ao secundário (rede). Alertar sobre ele.
- **Preso em SEEDING**: você não pode falhar ou se conectar ainda. A semeadura é executada em cerca de **500 GB/hSQL DB)** e **360 GB /h (MI)**. A criação do grupo MI é cancelada após o **6 dias**.
- **Os papéis MI não mudam**: verifique as regras **VNet peering** e NSG /firewall. Ambas as instâncias devem compartilhar o **zona DNS**.
- **Logins falha após failover**: use **usuários contidos** e **regras de firewall de nível de banco de dados** (eles replicam). Para o servidor logins, crie-os no secundário com o **mesmo SID**. Os bancos de dados do sistema MI não são replicadas.
- **Reconectar**: DNS TTL é **30 s**. As conexões existentes caem. Os aplicativos precisam de **retry lógica**.
- **Proteger uma transação crítica**: ligue para `sp_wait_for_database_copy_sync` após o commit. Ele espera até que o log atinja o geo-secundário.

## Solução de problemas log shipping e backups
```sql
EXEC master.dbo.sp_help_log_shipping_monitor;
SELECT * FROM msdb.dbo.log_shipping_monitor_error_detail ORDER BY log_time DESC;
```
- Alerta **14420** = no **backup** dentro do limiar. Alerta **14421** = no **restaurar** dentro do limiar. O alerta job em execução no **servidor monitor** (ou primário/secundário se nenhum).
- **Cadeia de log quebrada** (erro de restauração **4305**, "muito recente para aplicar"): alguém tomou um log extra backup fora do log shipping job, ou mudou para SIMPLE. Corrigir: restaurar o log ausente, ou re-semente de um full backup. Use **COPY_ONLY** para ad-hoc backups.
- Geo-secundários: **A retenção PITR não é replicada** (o padrão é de 7 dias). Defina você mesmo.

## Para lembrar na prova
- Brocas: **planejado** failover (sem perda de dados) em produção e para failback. **Forçado** apenas para interrupções reais.
- Teste a lógica de repetição do aplicativo em SQL DB → `Invoke-AzSqlDatabaseFailover`. **1 por 15 minutos**.
- Falha sobre um failover group → `Switch-AzSqlDatabaseFailoverGroup` no servidor **secundário**.
- Região primária para baixo → `-AllowDataLoss` / `--allow-data-loss` / `FORCE_FAILOVER_ALLOW_DATA_LOSS`.
- Após AG forced failover → segundos **Suspensão** → `SET HADR RESUME`.
- Planned AG failover precisa de **commit + SYNCHRONIZED**, executado no **alvo**.
- métrica RPO: `replication_lag_sec`, `log_send_queue_size`. métrica RTO: `redo_queue_size`.
- `sys.dm_geo_replication_link_status` → é executado no **banco de dados do usuário**, não mestre.
- Erro 19407 em Azure VMs → relax **SameSubnetThreshold 40**, leasing **40 s**, verificação de saúde **60 s**.
- Alertas Log shipping: **14420 backup**, **restauração 14421**.

## Teste rápido
1. Você quer verificar se um aplicativo se reconecta corretamente quando o Azure SQL database falhar dentro da região. Qual comando?
   <details><summary>Resposta</summary>Use Invoke-AzSqlDatabaseFailover. O cmdlet dispara um failover local de HA. Os comandos de failover group testam o DR regional.</details>
2. Depois de um AG failover forçado, Os bancos de dados secundárias mostram NOT SYNCHRONIZING. O que você executa?
   <details><summary>Resposta</summary>ALTER DATABASE... SET HADR RESUME em cada secundário. Forced failover suspende o movimento de dados.</details>
3. Um geo-secundário continua ficando para trás e mostra wait type HADR_THROTTLE_LOG_RATE_MISMATCHED_SLO. Qual é a correção?
   <details><summary>Resposta</summary>Escalar o secundário para o mesmo nível de serviço e tamanho de computação como o primário.</details>
