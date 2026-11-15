# Automatizar implantações

> **Habilidades cobradas:** Automatizar a implantação usando modelos ARM e Bicep, Automatizar a implantação utilizando Azure PowerShell, Automatizar implantação usando Azure CLI
> **Tempo de leitura:** ~8 min

## Conceito central
Em um data center, um novo servidor significa racks e cabos. No Azure, tudo é criado através de um API (programas de entrada usam para falar com o Azure). Então você pode escrever seus servidores e bancos de dados como código. Este é **Infraestrutura como Código (IaC)**. É como um cartão de receita: qualquer um que o siga recebe o mesmo bolo todas as vezes.

Por que o IAC ajuda:
- O mesmo arquivo constrói **Dev, Test e Prod** da mesma maneira.
- Menos erros humanos do que clicar no portal.
- Você mantém os arquivos em **controle de fonte** (GitHub, Repos do Azure). Você pode ver todas as alterações e voltar para uma versão mais antiga.
- A **gasoduto CI / CD** (Azure Pipelines : GitHub Actions) pode implantar o arquivo automaticamente em cada mudança.

**Exemplo:** Um aplicativo escolar precisa do mesmo banco de dados em 3 ambientes. Um arquivo Bicep mais 3 pequenos arquivos de parâmetros constrói todos os três.

## Dois modelos de implantação

| | Declarativo | Imperativo |
|---|---|---|
| Você escreve | O **resultado final** que você quer | **Cada passo**, em ordem |
| Quem decide a ordem | Azure (leu as dependências) | Você |
| Ferramentas | **Modelos ARM**, **Bicep** | **Azure PowerShell**, **Azure CLI** |
| Melhor para | Implantações grandes e repetíveis | jobs pequeno, mudanças rápidas, verificações |

**Exemplo:** Declarativo está dizendo um táxi "levar-me para o aeroporto". Imperativo está dando cada volta: "esquerda, então direita, então em linha reta".

## Modelos ARM
- An **Modelo ARM** (Azure Resource Manager é um modelo de **JSON** arquivo que lista recursos e suas configurações.
- Ele suporta **parâmetros**, então um arquivo funciona com valores diferentes (região, tamanho, nomes).
- ARM funciona o **ordem**. Por exemplo, ele cria o SQL server antes do banco de dados. Em JSON você declara esta ordem com **`dependsOn`**.
- Você pode criar um recurso no portal e **exportação** como um modelo.
- Você implanta modelos com o portal, PowerShell, Azure CLI ou Azure Pipelines.

## Bicep
- **Bicep** é a linguagem **Recomendado** da Microsoft para modelos.
- É muito mais curto e fácil de ler do que JSON. Uma conta de armazenamento é cerca de 6 linhas de Bicep, mas cerca de 20 linhas de JSON.
- A `.bicep` arquivo é **transformado em ARM JSON** quando você implantar. Qualquer coisa ARM pode fazer, Bicep pode fazer.
- Para transformar um modelo JSON exportado em Bicep, use **`bicep decompile`** (ou `az bicep decompile`).
- Em Bicep, um recurso infantil usa **`parent:`**. Bicep então resolve a dependência para você.

**Exemplo (Bicep): um SQL server e uma banco de dados**
```bicep
@secure()
param adminPassword string
resource srv 'Microsoft.Sql/servers@2021-11-01' = {
  name: 'shop-sql-${uniqueString(resourceGroup().id)}'
  location: resourceGroup().location
  properties: {
    administratorLogin: 'sqladmin'
    administratorLoginPassword: adminPassword
  }
}
resource db 'Microsoft.Sql/servers/databases@2021-11-01' = {
  parent: srv
  name: 'shopdb'
  location: resourceGroup().location
  sku: { name: 'S0' }
}
```
`@secure()` esconde a senha dos logs. `uniqueString()` faz um nome de servidor que ninguém mais usou.

**O mesmo banco de dados em ARM JSON (apenas a parte que difere):**
```json
{
  "type": "Microsoft.Sql/servers/databases",
  "apiVersion": "2021-11-01",
  "name": "[concat(parameters('serverName'), '/shopdb')]",
  "location": "[resourceGroup().location]",
  "sku": { "name": "S0" },
  "dependsOn": [ "[resourceId('Microsoft.Sql/servers', parameters('serverName'))]" ]
}
```
Em JSON você deve escrever `dependsOn` Você mesmo. Em Bicep, `parent: srv` Faz isso.

## Azure PowerShell
- PowerShell é uma linguagem de shell e script. É executado em **Windows, Linux e macOS**.
- O **módulo Az** gerencia o Azure. O **Az.Sql** é a parte para Azure SQL Database e Managed Instance.
- Os comandos usam nomes **Verbo-Noun**: `New-AzSqlServer`, `Get-AzSqlDatabase`.
- Execute-o em seu PC, no **Azure Cloud Shell** (no navegador, nada para instalar), ou no Docker.

**Exemplo (PowerShell): criar um servidor e uma banco de dados**
```powershell
Connect-AzAccount
New-AzResourceGroup -Name shopRG -Location eastus
New-AzSqlServer -ResourceGroupName shopRG -ServerName shop-sql-01 `
  -Location eastus -SqlAdministratorCredentials (Get-Credential)
New-AzSqlDatabase -ResourceGroupName shopRG -ServerName shop-sql-01 `
  -DatabaseName shopdb -RequestedServiceObjectiveName S0
```
- `New-AzSqlInstance` cria um Managed Instance. Nota: **Hardware Gen4 é aposentado**; use Gen5 ou mais recente.

**Implantar um modelo com PowerShell:**
```powershell
New-AzResourceGroupDeployment -ResourceGroupName shopRG `
  -TemplateFile ./main.bicep -TemplateParameterFile ./prod.parameters.json
```
- Azure PowerShell precisa do **Bicep CLI instalado à mão**.

## Azure CLI
- O **Azure CLI** (interface de linha de comando) também é executado em todos os lugares, incluindo o Cloud Shell.
- Funciona em **Cmd, Bash e PowerShell**. Azure PowerShell funciona apenas em PowerShell.
- Forma do comando: `az <group> <command> --parameter value`.
- Alguns comandos para Azure Database for PostgreSQL e MySQL existem **Somente no CLI**.

**Exemplo (Azure CLI): o mesmo servidor e banco de dados**
```bash
az login
az group create --name shopRG --location eastus
az sql server create --name shop-sql-01 --resource-group shopRG \
  --location eastus --admin-user sqladmin --admin-password "$PASS"
az sql server firewall-rule create --resource-group shopRG --server shop-sql-01 \
  -n AllowAzure --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
az sql db create --resource-group shopRG --server shop-sql-01 --name shopdb --service-objective S0
```
A regra de firewall `0.0.0.0` para `0.0.0.0` significa "permitir serviços do Azure".

**Implantar um modelo com o CLI:**
```bash
az deployment group create --resource-group shopRG --template-file main.bicep
az deployment group create --resource-group shopRG --template-uri "https://<storage>/main.json" --query-string $sasToken
```
- CLI **instala Bicep automaticamente**.
- O arquivo **local** `.bicep` é implantado diretamente. Você também pode passar `--parameters main.bicepparam`.
- **Nenhuma das ferramentas pode implantar um arquivo `.bicep` remoto.** Para um arquivo armazenado ou no GitHub, construa-o para o JSON primeiro e implante o JSON. Os modelos JSON remotos funcionam, com uma **token SAS** (uma chave de acesso limitada por tempo) para armazenamento privado.
- escopos CLI: `az deployment group` (grupo de recursos), `sub` (assinatura), `mg` (grupo de gerenciamento), `tenant`.

## Compare em um relance
| Tarefa | Azure CLI | PowerShell |
|---|---|---|
| Iniciar sessão | `az login` | `Connect-AzAccount` |
| Mudar assinatura | `az account set --subscription` | `Set-AzContext -Subscription` |
| Criar SQL server | `az sql server create` | `New-AzSqlServer` |
| Criar banco de dados | `az sql db create` | `New-AzSqlDatabase` |
| deploy template | `az deployment group create` | `New-AzResourceGroupDeployment` |
| Instala Bicep para você | Sim | Não |

## IaaS vs PaaS implantações
Em um **Azure VM**, você também deve rotear a instalação e as configurações do SQL Server e corrigi-lo. No **SQL Database / Managed Instance**, o Azure instala e patches SQL; você só escolhe as configurações de banco de dados.

## Para lembrar na prova
- "Descreva o estado final, repetível, no controle da fonte" → **ARM ou Bicep** (declarativo).
- "Alterar ou verificar rapidamente uma configuração" → **PowerShell ou CLI** (imperativo).
- **Bicep se transforma em ARM JSON** no momento da implantação. Bicep é a linguagem recomendada.
- JSON usa **`dependsOn`**. Bicep usa **`parent`** ou uma referência simbólica.
- **Arquivos `.bicep` remotos não podem ser implantados** por CLI ou PowerShell. Construir para JSON primeiro.
- PowerShell = `New-AzResourceGroupDeployment`. CLI = `az deployment group create`.

## Teste rápido
1. Você deve implantar um arquivo Bicep armazenado em uma conta de armazenamento usando o Azure CLI. O que você faz?
   <details><summary>Resposta</summary>Construa-o para ARM JSON primeiro, em seguida, implante o JSON com --template-uri. O CLI não pode implantar arquivos remotos. bicep.</details>
2. Sua equipe quer a mesma configuração SQL em Dev, Test e Prod, mantida no Git. Declarativo ou imperativo?
   <details><summary>Resposta</summary>Declarativo (Bicep ou ARM). Você descreve o estado final uma vez e reutiliza-o.</details>
3. Em um modelo ARM JSON, o banco de dados é criado antes de seu servidor e falha. O que está faltando?
   <details><summary>Resposta</summary>A dependsOn no servidor. Em Bicep Você usaria os pais.</details>
