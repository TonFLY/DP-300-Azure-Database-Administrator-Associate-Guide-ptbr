# Diagnosticar problemas em jobs do SQL Server Agent

> **Habilidades cobradas:** Diagnosticar problemas em jobs do SQL Server Agent e configurar alertas e notificações de jobs
> **Tempo de leitura:** ~7 min

## Conceito central
Quando um job falha, siga uma lista curta de verificações. O SQL Server Agent está em execução? O que o histórico mostra? Qual conta executa o job? O e-mail foi enviado? É como verificar combustível, bateria e chave, nessa ordem, quando um carro não liga.

## 1. O serviço do Agent está em execução?
- **VM:** se o serviço do Agent estiver parado, os jobs não rodam. O SSMS exibe **“Agent XPs disabled”** ao lado de SQL Server Agent.
- Em **SQL Server Configuration Manager**, configure **Start mode = Automatic**. Se precisar alterar a conta do serviço, faça isso ali também.
- Verifique os **Error Logs** do SQL Server Agent se houver falha na inicialização.

```sql
SELECT servicename, status_desc, startup_type_desc, service_account
FROM sys.dm_server_services;
```

- **Managed Instance:** o Agent fica **sempre em execução**. Se um job não rodar, verifique o schedule, se o job está habilitado e quais permissões estão disponíveis.

## 2. Consultar o histórico do job
| Objeto em `msdb` | O que armazena |
|---|---|
| `dbo.sysjobs` | Definições de jobs (nome, status de habilitação e owner). |
| `dbo.sysjobsteps` | Definições de steps (tipo, comando, banco de dados e proxy). |
| `dbo.sysjobhistory` | Resultados. **`step_id = 0` representa o resultado geral do job.** Uma linha é gravada **depois** que o step termina. |
| `dbo.sysjobactivity` | Jobs **em execução agora** e a última execução. |

**`run_status`**: **0** falhou · **1** teve sucesso · **2** nova tentativa · **3** cancelado · **4** em andamento.

```sql
SELECT j.name, h.step_id, h.step_name, h.run_date, h.run_time, h.message
FROM msdb.dbo.sysjobhistory AS h
JOIN msdb.dbo.sysjobs AS j ON j.job_id = h.job_id
WHERE h.run_status = 0                 -- failures only
ORDER BY h.run_date DESC, h.run_time DESC;

EXEC msdb.dbo.sp_help_jobhistory @job_name = N'NightlyBackup', @run_status = 0, @mode = N'FULL';
```

### Quando o histórico mais antigo some
- Limites padrão de **1.000 linhas no total** e **100 linhas por job**. Um job frequente faz as linhas antigas saírem rapidamente.
- **VM:** altere os limites em **SQL Server Agent Properties → History**. Para limpar o histórico manualmente, use `sp_purge_jobhistory`.
- **Managed Instance:** os limites são fixos em **1.000 / 100** e não podem ser alterados.
- A coluna `message` guarda apenas **4.000 caracteres**. Mensagens mais longas são truncadas. Para capturá-las por inteiro, configure a saída do step.

## 3. Capturar a saída completa do step
Na página **Advanced** do step, ou com `sp_update_jobstep`, direcione a saída para um **arquivo** ou uma **tabela**:

| `@flags` | Efeito |
|---|---|
| `0` / `2` | Sobrescrever ou **acrescentar** ao arquivo de saída. |
| `8` / `16` | **Log to table** (tabela `msdb.dbo.sysjobstepslogs`), sobrescrevendo ou acrescentando registros. |

```sql
EXEC msdb.dbo.sp_update_jobstep @job_name = N'NightlyIndexMaint', @step_id = 1,
     @output_file_name = N'D:\SQLLogs\IndexMaint.txt', @flags = 2;  -- append
```
Em **Managed Instance**, use **Log to table**, pois não há acesso ao sistema de arquivos.

## 4. Qual conta executa o job?
| Problema | Sintoma | Solução |
|---|---|---|
| **O job owner saiu da empresa** (login excluído ou desabilitado) | “Unable to determine if the owner ... has server access” | Defina um **service login** ou `sa` como owner com `sp_update_job @owner_login_name = N'sa'`. |
| **O owner não é sysadmin** | O step T-SQL executa no contexto do owner e não tem as permissões necessárias. | Conceda as permissões necessárias ou defina um sysadmin como owner do job. |
| **O step CmdExec, PowerShell ou SSIS teve acesso negado** | O step não acessa um compartilhamento ou o sistema operacional. | Use um **proxy** com uma **credential** que tenha as permissões. |
| **O proxy não pode ser usado** | O proxy não aparece na lista ou o step falha. | Conceda acesso ao proxy usando `sp_grant_proxy_to_subsystem` e `sp_grant_login_to_proxy`. |

- **Steps T-SQL nunca usam proxies.** Somente um **sysadmin** pode criar proxies.
- Usuários que não são sysadmin precisam de uma role do Agent em `msdb`: **SQLAgentUserRole**, **SQLAgentReaderRole** ou **SQLAgentOperatorRole**. SQLAgentUserRole dá visibilidade **apenas aos próprios jobs**.

**Exemplo:** o job de ETL de uma escola começa a falhar na segunda-feira. Seu owner era um DBA que saiu na sexta-feira e teve o login removido. Troque o owner por um service login para que o job volte a funcionar.

## 5. Os e-mails não chegam
1. Verifique a configuração do Database Mail na [lição 3](<03 - SQL Server Agent jobs.md>). Em VM, confira se o perfil está habilitado no Agent e se o Agent foi reiniciado. Em MI, verifique o perfil `AzureManagedInstance_dbmail_profile` e a regra de SMTP de saída no NSG.
2. Verifique se o **operator** está habilitado e selecionado na página **Notifications** do job.
3. Procure erros de envio em `msdb.dbo.sysmail_event_log` e `sysmail_faileditems`.

## 6. Jobs e availability groups
- Um **availability group (AG)** mantém cópias dos bancos de dados em vários servidores. O `msdb` fica fora do AG, portanto **os jobs não acompanham o failover**.
- Crie cada job em **todas as réplicas**. No primeiro step, confira se o servidor é o primário:

```sql
IF ISNULL(sys.fn_hadr_is_primary_replica(N'SalesDb'), 0) <> 1
BEGIN
    PRINT 'Not primary - skipping.';
    RETURN;
END
```
- A função retorna **1** no primário, **0** em um secundário e **NULL** quando o banco não está em um AG.
- A exceção são os **contained AGs no SQL Server 2022+**, que têm um `msdb` próprio. Neles, os jobs acompanham o primário.
- Os **Managed Instance failover groups** também não copiam `msdb`. Recrie os jobs no secundário.

## 7. Limites do Agent em Managed Instance
| Com suporte em MI | Sem suporte em MI |
|---|---|
| Steps T-SQL e SSIS e steps Snapshot, Log Reader e Distribution de replication. | **Proxies**, **Agent alerts**, Pager, NetSend e EventLog. |
| Schedules, operators e notificações por **e-mail**. | Ligar ou desligar o Agent e editar suas propriedades (`sp_set_agent_properties`). |
| Roles SQLAgent em `msdb`. | CmdExec, schedules **Idle-CPU**, steps Merge e Queue Reader, SSAS, MSX/TSX e PowerShell Core. |

- Usuários do Microsoft Entra precisam estar **mapeados diretamente para um login** para gerenciar jobs. O mapeamento apenas por grupo é insuficiente.

## Para lembrar na prova
- `run_status = 0` indica falha. `step_id = 0` indica o resultado geral do job.
- Para verificar se um job está rodando agora, consulte `sysjobactivity`.
- Quando o histórico antigo some, verifique os limites de **1.000 linhas no total / 100 por job**. Em MI, os limites são fixos.
- Para capturar uma mensagem truncada, grave a saída em **arquivo** ou use **Log to table**.
- Se o job falhar após a saída de um funcionário, troque o **job owner**.
- Se um step CmdExec não acessar um compartilhamento, use **proxy e credential** (disponível em VM).
- Se o job rodar ou falhar no secundário de um AG, adicione a verificação com **`sys.fn_hadr_is_primary_replica`** e crie o job em todas as réplicas.
- Em MI, **proxies e alertas do Agent não têm suporte**. O perfil de e-mail é **`AzureManagedInstance_dbmail_profile`**.

## Teste rápido
1. O histórico de um job só mostra os últimos dias. O que aconteceu com as entradas anteriores?
   <details><summary>Resposta</summary>As entradas antigas foram removidas pelos limites do histórico (1.000 linhas no total e 100 por job). Em uma VM, é possível aumentar os limites. Em MI, são fixos.</details>
2. Um job de backup roda nas duas réplicas do AG e falha no secundário. O que você adiciona?
   <details><summary>Resposta</summary>Um step inicial que verifica sys.fn_hadr_is_primary_replica e encerra a execução se o servidor não for o primário.</details>
3. O que acontece se você tentar criar um proxy para um job step em Managed Instance?
   <details><summary>Resposta</summary>Managed Instance não oferece suporte a proxies do SQL Server Agent.</details>
