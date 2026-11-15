# Jobs do SQL Server Agent

> **Habilidades cobradas:** Gerenciar agendamentos de jobs de manutenção periódica, configurar alertas e notificações de jobs e configurar alertas e notificações de tarefas de banco de dados
> **Tempo de leitura:** ~8 min

## Conceito central
Os bancos de dados precisam de manutenção periódica, com backups, verificações de corrupção e manutenção de índices e estatísticas. O **SQL Server Agent** é o agendador integrado que executa essas tarefas. Imagine um gerente noturno que percorre uma lista de tarefas a cada noite. Quando algo dá errado, ele liga para o responsável.

## Onde o SQL Server Agent está disponível
| Plataforma | Disponibilidade do SQL Server Agent |
|---|---|
| **SQL Server em uma Azure VM** | **Sim, com todos os recursos do Agent.** Você gerencia o serviço, a conta do serviço, os proxies e os alertas. |
| **Azure SQL Managed Instance** | **Sim, integrado e sempre em execução.** Não é possível pará-lo nem iniciá-lo manualmente. As configurações são **read-only**. Alguns recursos têm limitações, descritas na lição 4. |
| **Azure SQL Database** | **Não.** O serviço não tem o banco `msdb`, onde o Agent armazena os jobs. Use **Elastic Jobs**, Azure Automation ou Logic Apps (lição 5). |

## Jobs, steps e schedules
- Um **job** é uma tarefa com nome, como “NightlyBackup”.
- Um job tem um ou mais **steps** (etapas). Cada step pode executar T-SQL, PowerShell, um pacote SSIS (SQL Server Integration Services, usado para movimentar dados) ou CmdExec (um comando do sistema operacional).
- Um **schedule** define quando o job roda, por exemplo, uma vez, diariamente, toda semana ou na inicialização do Agent.
- Jobs e schedules têm uma relação **muitos para muitos**. Um schedule pode disparar vários jobs e um job pode ter vários schedules.
- Jobs, steps, schedules e o histórico ficam no banco de sistema **`msdb`**.

## Exemplo de agendamento
Um banco de dados de uma loja segue esta programação:

| Quando | O quê |
|---|---|
| Domingo, às 2 h | Full backup |
| De segunda a sábado, à 1 h | Differential backup |
| A cada hora | Transaction log backup (permite restaurar até um ponto específico no tempo) |
| Sábado, à meia-noite | Manutenção de índices |

## Planos de manutenção
- O **Maintenance Plan Wizard** do SSMS (SQL Server Management Studio) cria as tarefas de manutenção.
- O assistente salva cada plano como um **pacote SSIS** e cria um **job** do SQL Server Agent para executá-lo.
- Cada plano recebe um **schedule separado**.
- Você escolhe a conta sob a qual o job será executado, como a **SQL Agent service account** ou uma **proxy account**.

Tarefas disponíveis
| Tarefa | O que faz |
|---|---|
| Check Database Integrity | Executa `DBCC CHECKDB`. |
| Shrink Database | Reduz o tamanho dos arquivos. **Evite executar com frequência**, pois fragmenta os índices. |
| Reorganize Index / Rebuild Index | Trata a fragmentação (páginas de índice fora de ordem). |
| Update Statistics | Ajuda o otimizador a escolher planos de execução adequados. |
| History Cleanup | Remove o histórico antigo de backups e jobs em `msdb`. |
| Back Up Database | Executa um full backup, differential backup ou transaction log backup. |
| Maintenance Cleanup | Remove do disco os arquivos antigos de backups e relatórios. |
| Execute SQL Server Agent Job | Executa outro job existente. |

Boas práticas
- Execute tarefas pesadas, como rebuild de índices e CHECKDB, **em horários de menor atividade**.
- **Retenha os backups por mais tempo** que o intervalo entre as execuções do CHECKDB, pois a corrupção pode ser detectada com atraso.
- **Evite agendar Shrink Database** como rotina.

## Contas proxy
- Um **proxy** permite que um step CmdExec, PowerShell ou SSIS rode com uma **conta do Windows diferente**. O proxy se baseia em uma **credential**, que guarda o nome da conta e seu segredo. **Steps T-SQL nunca usam proxy.** Managed Instance não oferece suporte a proxies.
- **Exemplo:** um step precisa copiar backups para um compartilhamento de arquivos restrito. O step roda por meio de um proxy com acesso ao compartilhamento.

## Administração de múltiplos servidores
No SQL Server, um **master server (MSX)** pode distribuir jobs a vários **target servers (TSX)**. Esse recurso não está disponível em Managed Instance.

## Operators, notificações e alertas
| Objeto | O que é |
|---|---|
| **Operator** | Pessoa ou grupo que recebe mensagens, normalmente por e-mail. |
| **Notification** | Mensagem enviada quando um **job** é bem-sucedido, falha ou termina. |
| **Alert** | Regra disparada por um **erro** ou uma **condição** para avisar um operator ou executar um job. |

### Operators
- Um operator tem um nome e um **endereço de e-mail**.
- **Boa prática:** use um **e-mail de grupo** para que as mensagens cheguem mesmo quando alguém estiver ausente.
- **Exemplo:** o operator `DBATeam` recebe mensagens em `dba-support@shop.com`.

### Notificações de jobs
- Na página **Notifications** do job, escolha o operator e **quando** enviar a notificação. As opções são **On failure** (a mais comum), **On success** e **On completion**.
- **Exemplo:** envie um e-mail para `DBATeam` quando o backup noturno **falhar**.

### Alertas
O SQL Server Agent pode disparar um alerta nestas situações:
| Tipo | Exemplo |
|---|---|
| **Número do erro SQL** | Erros **823, 824 e 825** (erros de E/S em disco que podem indicar corrupção). |
| **Nível de severidade** | Qualquer erro de severidade **16 ou superior**. |
| **Condição de desempenho** | Page Life Expectancy abaixo de 300 (pressão de memória). |
| **Evento WMI** | Failover de um availability group. |

Quando o alerta dispara, o Agent pode:
1. Notificar um operator, normalmente por e-mail.
2. **Executar um job** em resposta ao alerta. Por exemplo, ao receber o erro 823, executar um job com `DBCC CHECKDB`.

**Managed Instance:** os **alertas do SQL Server Agent**, Pager e NetSend não têm suporte. Os **resultados dos jobs** podem gerar notificações por e-mail. Para os demais alertas em MI, use **Azure Monitor alerts**.

## Database Mail
O Agent envia e-mails usando o **Database Mail**. A configuração depende do ambiente.

**SQL Server (VM):**
1. Ative o Database Mail e crie um **mail profile** com o servidor SMTP e a conta.
2. Em **SQL Server Agent Properties → Alert System**, selecione **Enable mail profile** e escolha o perfil.
3. **Reinicie o SQL Server Agent.**

**Managed Instance:**
- As propriedades do Agent são **read-only**, portanto a interface não permite selecionar um perfil.
- O perfil do Database Mail **deve se chamar `AzureManagedInstance_dbmail_profile`**.
- O **NSG** (network security group) da sub-rede deve permitir tráfego **de saída** para a porta SMTP (25, 587 ou 465).

```sql
EXEC msdb.dbo.sp_add_operator @name = N'DBATeam', @email_address = N'dba-support@shop.com';
EXEC msdb.dbo.sp_update_job  @job_name = N'NightlyBackup',
     @notify_level_email = 2, @notify_email_operator_name = N'DBATeam';  -- 2 = on failure
```
`@notify_level_email`: **1** = on success, **2** = on failure, **3** = on completion.

## Para lembrar na prova
- O Agent está disponível em **VM** e **Managed Instance**. No **Azure SQL Database**, use Elastic Jobs.
- Em MI, o Agent está **sempre ativo** e suas configurações são **read-only**.
- Jobs, schedules e histórico ficam em **`msdb`**. Planos de manutenção são **pacotes SSIS**.
- A relação entre jobs e schedules é **muitos para muitos**.
- **Evite incluir Shrink Database** em um schedule de manutenção.
- Operator indica quem recebe. Notification informa o resultado do job. Alert reage a um erro, severidade, condição de desempenho ou evento WMI.
- MI não oferece suporte a **alertas do Agent nem proxies**. O perfil de e-mail deve ser **`AzureManagedInstance_dbmail_profile`**.
- No SQL Server, habilite o perfil de e-mail nas propriedades do Agent e depois **reinicie o Agent**.
- Um alerta pode **executar um job** para tratar o problema.

## Teste rápido
1. No Managed Instance, os e-mails dos jobs não chegam e você não pode editar as propriedades do Agent. O que deve verificar primeiro?
   <details><summary>Resposta</summary>Verifique se o perfil do Database Mail se chama AzureManagedInstance_dbmail_profile e se o NSG permite tráfego SMTP de saída.</details>
2. Você quer receber um e-mail quando o erro 824 ocorrer no SQL Server em uma VM. O que deve criar?
   <details><summary>Resposta</summary>Crie um alerta do SQL Server Agent para o erro 824 e indique qual operator deve receber a notificação.</details>
3. Um job step precisa copiar arquivos para um compartilhamento que a conta do serviço do Agent não consegue acessar. O que você usa?
   <details><summary>Resposta</summary>Use uma proxy account baseada em uma credential com acesso ao compartilhamento. Esse recurso está disponível em VM e não tem suporte em MI.</details>
