# Alertas, monitoramento e diagnóstico de tarefas automatizadas

> **Habilidades cobradas:** Configurar alertas e notificações sobre tarefas de banco de dados, Solucionar problemas de tarefas automatizadas de banco de dados
> **Tempo de leitura:** ~5 min

## Conceito central
Uma tarefa automatizada que falha silenciosamente é pior do que nenhuma tarefa. Você precisa de **ver** cada execução, **ser dito** quando um falha, e **Descubra por que**. É como um alarme de fumaça: não pára o fogo, mas acorda-lo a tempo.

## Azure Monitor alerts e grupos de ação
No Azure, os alertas vêm do **Azure Monitor**. Uma regra de alerta tem três partes:
1. **Scope** – o recurso para assistir (uma conta de automação, um elastic job agent, um banco de dados).
2. **Condição** – o sinal e o limiar. Exemplo: "Falha elastic jobs maior que 0".
3. **Grupo de acção** – o que fazer quando dispara.

An **grupo de acção** pode:
- Enviar **e-mail ou SMS**.
- Ligue para um **webhook** (um endereço web que outro sistema escuta).
- Inicie um **Automação runbook** (por exemplo, para corrigir o problema em si).

Você pode filtrar um alerta por **dimensão**, como um nome runbook.

**Exemplo:** " Uma loja de noite "PurgeOldOrders" runbook falha. Um alerta na conta de automação job métrica, filtrada para que runbook e status falhou, e-mails o DBA equipe através de um grupo de ação.

## Monitor Azure Automation runbooks
| Onde | O que você vê |
|---|---|
| Conta de automação → **Jobs** | Cada execução e seu status (concluído, falhado, em execução) |
| A job's **Erros** / **Todos os logs** fluxos | O texto de erro e a saída para essa execução |
| **Diário de atividades** | **Quem** começou um runbook e quando |
| **Log Analytics** | Registros Job que você pode consultar com KQL, através de muitas corridas |

- Para enviar logs job para Log Analytics, ligue primeiro o **Configurações diagnósticas** para a conta de automação.

```kusto
AzureDiagnostics
| where Category == "JobLogs"
| where ResultType == "Failed"
```
KQL = Kusto Query Language.

## Monitor elastic jobs
| Onde | O que você vê |
|---|---|
| Portal → elastic job agent → **Visão geral** | Job corre e status |
| PowerShell | `Get-AzSqlElasticJobExecution`, mais `Get-AzSqlElasticJobStepExecution` e `Get-AzSqlElasticJobTargetExecution` para **Cada passo** e **cada banco de dados alvo** |
| T-SQL na banco de dados job | Vista `jobs.job_executions` |

```sql
-- Failed runs, newest first
SELECT job_name, target_database_name, lifecycle, last_message, start_time
FROM jobs.job_executions
WHERE lifecycle = 'Failed'
ORDER BY start_time DESC;

-- Jobs running now
SELECT * FROM jobs.job_executions WHERE is_active = 1;
```

- Elastic jobs **tentar novamente** em erros curtos e temporários (como um blip de rede).
- **Alertas:** o agente job trabalha com **Azure Monitor**. Use a métrica **Falha Elastic jobs** com um grupo de ação.
- A história em `jobs.job_executions` é **deletado após 45 dias**. Você pode limpá-lo mais cedo com `jobs.sp_purge_jobhistory`.
- Dê às pessoas que só precisam assistir jobs o papel **`jobs_reader`** no banco de dados job.

**Exemplo:** An elastic job adiciona uma coluna a 300 bancos de dados de clientes. Mostra "Falha". `Get-AzSqlElasticJobTargetExecution` mostra apenas 2 bancos de dados falharam, porque a identidade job não tem usuário neles. Adicione o usuário e execute novamente. O passo é idempotente, então as 298 boas bancos de dados não são prejudicadas.

## Jobs do SQL Server Agent
Use notificações e alertas do agente (lição 3) e histórico job (lição 4). No **Managed Instance** não há alertas do agente, então use **Azure Monitor alerts**.

## Lista de verificação de solução de problemas
| Sintoma | Verificar |
|---|---|
| Runbook falha ao iniciar sessão | Managed identity está ligado e tem o direito **Papel RBAC** |
| Runbook diz que um cmdlet não foi encontrado | O módulo é **Importado** (Az.Accounts antes de Az.Sql; SqlServer para `Invoke-Sqlcmd`) |
| Elastic job falha em algumas bancos de dados | Resultados por alvo. A identidade job tem um usuário em cada alvo? O alvo é alcançável (firewall, private endpoint aprovado)? |
| App Logic nunca dispara em novas linhas | Intervalo de polling; em Standard, **change tracking** sobre a tabela |

## Para lembrar na prova
- Azure alerts = **Regra de alerta Azure Monitor + grupo de ação**.
- O grupo de ação pode enviar e-mail, SMS, chamar um **webhook** ou iniciar um **runbook**.
- Status Runbook → Conta de automação **Jobs**. Quem o executou → **log de atividade**.
- Os logs Runbook no Log Analytics precisam de **Configurações diagnósticas** primeiro.
- Alerta Elastic job métrica → **Falha Elastic jobs**.
- História Elastic job: `jobs.job_executions`, mantido **45 dias**. Os observadores recebem **`jobs_reader`**.
- Falhas por alvo → `Get-AzSqlElasticJobTargetExecution`.
- MI não tem alertas de agente → **Azure Monitor**.

## Teste rápido
1. Você quer um e-mail quando algum elastic job falhar. O que você configura?
   <details><summary>Resposta</summary>An Azure Monitor Alerta sobre o falhado Elastic jobs métrica da job com um grupo de ação que envia e-mail.</details>
2. Você precisa saber quem começou um runbook ontem. Onde você olha?
   <details><summary>Resposta</summary>O log de atividades da conta Automation.</details>
3. Você quer consultar logs runbook job com KQL, mas a tabela AzureDiagnostics está vazia. O que está faltando?
   <details><summary>Resposta</summary>Configurações de diagnóstico na conta Automação, enviando logs job para o Log Analytics workspace.</details>
