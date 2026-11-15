# Automatizar tarefas no Azure

> **Habilidades cobradas:** Criar e configurar Elastic Jobs e criar e configurar tarefas de banco de dados por meio de automação
> **Tempo de leitura:** ~9 min

## Conceito central
O Azure SQL Database não oferece **SQL Server Agent**. No Azure, **Elastic Jobs** executa T-SQL em vários bancos, **Azure Automation** agenda scripts em PowerShell ou Python, e **Logic Apps** executa workflows low-code em resposta a eventos. Imagine uma sede que envia a mesma instrução a todas as lojas (Elastic Jobs), tem uma pessoa de TI para executar scripts (Azure Automation) e usa um formulário para iniciar uma sequência de tarefas quando chega um pedido (Logic Apps).

## Elastic Jobs
Um **Elastic Job** executa **T-SQL** de forma agendada ou sob demanda em **vários bancos de dados** de uma vez. Ele é o equivalente de um job do SQL Server Agent para Azure SQL Database.
- **Em General Availability desde abril de 2024.** Funciona em **todos os tiers** do Azure SQL Database.
- Os alvos podem estar em servidores, pools, regiões e assinaturas diferentes. **Somente T-SQL.**

**Exemplo:** um aplicativo SaaS tem um banco por cliente, num total de 300 bancos. É preciso adicionar uma coluna a todos eles. Um Elastic Job faz isso em uma única execução.

### Componentes de um Elastic Job
| Componente | O que faz |
|---|---|
| **Elastic Job agent** | Recurso do Azure que executa os jobs. |
| **Job database** | Banco do Azure SQL Database que guarda as definições e o histórico dos jobs. Use um banco **novo e vazio**. Recomenda-se **S1 ou superior**. **Hyperscale** e **serverless com auto-pause** não têm suporte para essa finalidade. |
| **Job agent tier** | Define quantos alvos podem executar em paralelo: **JA100** (padrão, 100), JA200, JA400 e JA800. |
| **Target group** | Define onde o job roda: um banco individual, **todos os bancos de um servidor** ou um **elastic pool**. |
| **Job e job steps** | Definem o T-SQL a executar. |

### Como configurar
1. **Crie o job agent** pelo portal, PowerShell ou REST API e vincule-o ao job database. Elastic Jobs não oferece comandos de Azure CLI no grupo `az sql`.
2. **Configure a autenticação** nos alvos. Escolha **um** método por agent:
   - **Recomendado: Microsoft Entra ID com uma user-assigned managed identity (UMI)** associada ao job agent. Uma **system-assigned managed identity** não tem suporte.
   - Ou configure **database-scoped credentials** no job database. Se o alvo for um servidor ou pool, também é necessária uma credential para `master`, para que o agent consiga listar os bancos.
   - Conceda à identidade somente as permissões necessárias, seguindo o princípio de **least privilege**.
3. **Crie os target groups**, depois o **job e seus steps**. Execute o T-SQL abaixo no job database.
4. **Execute o job** uma vez com `jobs.sp_start_job` ou `Start-AzSqlElasticJob`, ou configure um schedule.

Para alvos com acesso público desabilitado, use **Elastic Job private endpoints** e **aprove** o endpoint em cada servidor-alvo.

```sql
-- In the job database
EXEC jobs.sp_add_target_group N'AllCustomers';
EXEC jobs.sp_add_target_group_member N'AllCustomers',
     @target_type = N'SqlServer', @server_name = N'cust-sql.database.windows.net';
EXEC jobs.sp_add_job @job_name = N'AddLoyaltyColumn';
EXEC jobs.sp_add_jobstep @job_name = N'AddLoyaltyColumn', @target_group_name = N'AllCustomers',
     @command = N'IF COL_LENGTH(''dbo.Customer'',''Loyalty'') IS NULL
                  ALTER TABLE dbo.Customer ADD Loyalty int NULL;';
EXEC jobs.sp_start_job N'AddLoyaltyColumn';
```

### Escreva steps idempotentes
Elastic Jobs **tenta executar novamente** os steps que falham. Escreva steps **idempotentes**, que possam ser executados duas vezes com segurança. A condição `IF ... IS NULL` acima impede que uma segunda execução faça alterações indevidas.

## Azure Automation
**Azure Automation** é um serviço de nuvem para executar scripts. Ele funciona com máquinas no **Azure e on-premises**.

**Exemplo:** todos os dias às 6 h, um runbook inicia uma VM e executa um script de backup do SQL. Às 7 h, ele desliga a VM para economizar.

### Componentes de uma Automation account
| Componente | O que faz |
|---|---|
| **Automation account** | Agrupa os componentes a seguir. |
| **Runbook** | Script em **PowerShell** ou **Python**, ou um workflow gráfico. |
| **Modules** | Módulos PowerShell que o runbook usa, como **Az.Accounts** e **Az.Sql**. |
| **Credentials** | Nomes de usuário e senhas armazenados, lidos pelo runbook com `Get-AutomationPSCredential`. |
| **Managed identity** | Permite que o runbook autentique no Azure com `Connect-AzAccount -Identity`. As **Run As accounts foram descontinuadas em 30 de setembro de 2023**. |
| **Hybrid Runbook Worker** | Executa runbooks em suas próprias máquinas. Atualmente, usa **extension-based Hybrid Runbook Worker**. A versão agent-based foi descontinuada em 31 de agosto de 2024. |

### Criar um runbook
1. Crie uma **Automation account**.
2. Importe os módulos. Comece por **Az.Accounts**, depois **Az.Sql**. `Invoke-Sqlcmd` requer o módulo **SqlServer**. Ambientes de runtime do PowerShell 7.4+ já incluem uma versão padrão de Az.
3. Opcionalmente, adicione uma **credential**, como um login SQL chamado `SQLUser`.
4. Crie o runbook com o tipo **PowerShell**. O recomendado é o runtime **PowerShell 7.4** ou 7.6. As versões 7.1 e 7.2 perderam o suporte. A 5.1 ainda está disponível. Runbooks em Python usam a versão **3.10**.
5. Execute o teste no **Test pane**, publique, vincule um **schedule** e atribua as **RBAC roles** adequadas à managed identity.

```powershell
Connect-AzAccount -Identity                       # sign in with the managed identity
$cred = Get-AutomationPSCredential -Name "SQLUser"
Invoke-Sqlcmd -ServerInstance "shop-sql.database.windows.net" -Database "shopdb" `
  -Username $cred.UserName -Password $cred.GetNetworkCredential().Password `
  -Query "EXEC dbo.PurgeOldOrders;"
```

Detalhe sobre a execução
- Se um runbook for interrompido antes de terminar, ele **reinicia do começo**. Prepare o script para rodar novamente com segurança. Ele pode guardar o progresso em um banco de dados ou em storage.

**Tags** são rótulos no formato `key:value`, como `Environment:Production`. Elas ajudam os runbooks a selecionar recursos. Como tags são texto simples, nunca armazene segredos nelas.

## Logic Apps
**Azure Logic Apps** é um serviço serverless para **workflows**. Você monta o fluxo em um designer com pouco código ou nenhum. Um **trigger** inicia o workflow, que executa as **actions**.

**Exemplo:** ao inserir uma linha na tabela `Orders`, a Logic App lê a linha, envia um e-mail à equipe de vendas pelo Office 365 e publica no Microsoft Teams.

### O conector SQL Server
- O conector executa consultas e **stored procedures**, insere, atualiza e exclui linhas, e inicia workflows quando ocorrem eventos em tabelas.
- Funciona com **SQL Server** (on-premises ou VM), **Azure SQL Database** e **Azure SQL Managed Instance**.
- Métodos de autenticação: SQL auth, Entra integrated, service principal ou **managed identity** (recomendada).
- O trigger **“When an item is created”** consulta a tabela em intervalos fixos e lê **uma linha** por evento.
- Nos workflows **Standard**, os triggers SQL integrados, como **When a row is inserted**, exigem **change tracking** habilitado na tabela.
- Para Managed Instance sem data gateway, use o **public endpoint na porta 3342**.

### Tipos de plano
- **Consumption** (multi-tenant): cobrança por execução e managed connectors.
- **Standard** (single-tenant): ambiente isolado, conector SQL integrado, integração com VNet e private endpoints.
- O **Integration Service Environment (ISE)** foi descontinuado em **31 de agosto de 2024**. Migre para Standard.

## Qual ferramenta usar
| Necessidade | Ferramenta |
|---|---|
| Executar **T-SQL agendado em vários bancos Azure SQL** | **Elastic Jobs** |
| Agendar T-SQL em Azure SQL Database, onde não há Agent | **Elastic Jobs** |
| Agendar T-SQL, SSIS ou outros steps em **VM ou Managed Instance** | **SQL Server Agent** |
| Agendar **PowerShell ou Python**, gerenciar recursos do Azure e operar com máquinas on-premises | **Azure Automation** |
| Criar workflow low-code **orientado por eventos** entre serviços de e-mail, Teams, SharePoint e storage | **Logic Apps** |
| Executar pequenos trechos de código serverless em resposta a eventos | **Azure Functions** |

**Exemplo:** reduza o banco para S0 toda noite e volte para S3 às 8 h. Como a mudança é em um recurso do Azure, fora do T-SQL executado no banco, use um **runbook do Azure Automation**.

## Para lembrar na prova
- Para automatizar tarefas no **Azure SQL Database**, use Elastic Jobs, Azure Automation ou Logic Apps. O serviço não oferece SQL Server Agent.
- Elastic Jobs executa **somente T-SQL** em vários alvos, tem **GA desde abril de 2024** e funciona com todos os tiers.
- Use um job database novo e vazio, de tier **S1 ou superior**. Hyperscale e serverless com auto-pause não têm suporte para o job database.
- Para autenticar um Elastic Job, use **user-assigned managed identity** (recomendada) ou database-scoped credentials com uma credential para `master`. System-assigned managed identity não tem suporte.
- Escreva steps **idempotentes**, pois o job pode tentar executá-los outra vez.
- Runbooks autenticam com **managed identity**. Run As foi descontinuado. Importe **Az.Accounts antes de Az.Sql**.
- Triggers SQL de Logic Apps Standard exigem **change tracking**. O public endpoint de MI usa a **porta 3342**. **ISE foi descontinuado.**

## Teste rápido
1. Você precisa adicionar um índice a 50 bancos Azure SQL por semana. Qual ferramenta usa?
   <details><summary>Resposta</summary>Elastic Jobs executa T-SQL agendado em vários bancos do Azure SQL Database.</details>
2. Você cria um Elastic Job agent e habilita a system-assigned managed identity para acessar os alvos. Funciona?
   <details><summary>Resposta</summary>Não. Elastic Jobs exige uma user-assigned managed identity ou database-scoped credentials.</details>
3. Quando uma linha é inserida na tabela Orders, você precisa enviar um e-mail à equipe de vendas sem escrever código. Qual ferramenta usa?
   <details><summary>Resposta</summary>Logic Apps, com um trigger SQL e uma action de e-mail do Office 365.</details>
