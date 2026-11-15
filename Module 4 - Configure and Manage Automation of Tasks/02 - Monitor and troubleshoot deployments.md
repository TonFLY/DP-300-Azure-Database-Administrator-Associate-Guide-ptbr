# Monitorar e diagnosticar problemas em implantações

> **Habilidades cobradas:** Monitorar e diagnosticar problemas em implantações
> **Tempo de leitura:** ~8 min

## Conceito central
Um modelo pode falhar antes de começar, ou a meio caminho. Você precisa saber **Para onde olhar** para cada tipo de falha. Pense em postar um pacote. Os correios podem recusá-lo no balcão (endereço ruim). Ou pode se perder no caminho. Você verifica lugares diferentes para cada problema.

## Três tipos de erros de implantação
| Tipo de erro | Quando acontece | Onde você vê |
|---|---|---|
| **Erro de validação (syntax)** | Antes de qualquer coisa ser enviada. Bad JSON ou Bicep. | Código VS (extensão Bicep / ARM Tools), `bicep build` |
| **Erro de pré-voo** | O comando corre, mas **Nada é implantado**. Exemplo: um parâmetro ruim ou o SKU (tamanho) não é oferecido nessa região. | **Apenas log de atividade**. Não no histórico de implantação. |
| **Erro de implantação** | Passou cheques, mas **Falha ao criar** um recurso. | **Histórico de Implantação** e o registro de atividades |

A Bicep arquivo com erros de sintaxe nunca se torna JSON. Por isso, nunca chega ao registro de atividades.

**Exemplo:** O script de um banco pede SKU "S9" em uma região que não vende. O comando retorna um erro, mas nenhuma implantação aparece sob as implantações do grupo de recursos. O DBA encontra a falha no **log de atividade**.

## Teste antes de implantar: valide e what-if
- **Validar** verifica se o modelo é válido. Não diz o que vai mudar.
- **What-if** mostra o que **Iria** mudar. Ele faz **sem alterações**. Alterar tipos: Criar, Excluir, Modificar, NoChange, Ignorar, NoEffect, Implantar.
- **Excluir** só aparece em **modo completo** (veja abaixo).

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName rg-sql -TemplateFile main.bicep   # validate
New-AzResourceGroupDeployment  -ResourceGroupName rg-sql -TemplateFile main.bicep -WhatIf
New-AzResourceGroupDeployment  -ResourceGroupName rg-sql -TemplateFile main.bicep -Confirm  # preview, then ask
```
```bash
az deployment group validate --resource-group rg-sql --template-file main.bicep
az deployment group what-if  --resource-group rg-sql --template-file main.bicep
az deployment group create   --resource-group rg-sql --template-file main.bicep --confirm-with-what-if  # or -c
```

## Histórico de Implantação
- Cada implantação para um escopo (grupo de recursos, assinatura, grupo de gerenciamento, inquilino) é salva com um **correlação ID**. Dê este ID ao Suporte da Microsoft.
- **Portal:** grupo de recursos → **Configurações > Configurações** → escolher uma implantação → **Detalhes do erro** / **Detalhes da operação**.
- Cada implantação tem **operações** (um por recurso). Encontre a falha e leia sua mensagem de status.

```powershell
Get-AzResourceGroupDeployment -ResourceGroupName rg-sql
Get-AzResourceGroupDeploymentOperation -ResourceGroupName rg-sql -DeploymentName sqlDeploy01 |
  Where-Object { $_.ProvisioningState -eq "Failed" } | Select-Object -ExpandProperty StatusMessage
```
```bash
az deployment group list --resource-group rg-sql -o table
az deployment operation group list --resource-group rg-sql --name sqlDeploy01
```

### O limite 800
- Cada escopo mantém no máximo **800 implantações** na história.
- Quando a história ultrapassa **700**, o Azure elimina o mais antigo até chegar ao **600**. What-if e valida também inicia esta limpeza.
- Se a história já estiver em 800, a próxima implantação falhará com o **`DeploymentQuotaExceeded`**. Aguarde e tente novamente ou exclua entradas antigas (`Remove-AzResourceGroupDeployment` / `az deployment group delete`).
- Auto-limpeza **não funciona** se houver um **bloqueio CanNotDelete** no grupo de recursos ou assinatura. Também falha se o usuário não tiver a permissão `Microsoft.Resources/deployments/delete`.

## Diário de atividades
- O registro de atividades registra ações **controle-plano** (criar, atualizar, excluir recursos) na assinatura: quem fez o quê e o que falhou.
- Mostra **falhas pré-voo** que nunca alcança o histórico de implantação.
- Portal: grupo de recursos → **Diário de atividades** → filter Status = Failed.
- Você pode enviá-lo para Log Analytics (tabela `AzureActivity`) e criar **Alertas de registro de atividade**.

## debugar o registro
```powershell
New-AzResourceGroupDeployment -Name sqlDebug01 -ResourceGroupName rg-sql `
  -TemplateFile main.bicep -DeploymentDebugLogLevel All   # All | RequestContent | ResponseContent
```
- **Somente o PowerShell pode ligá-lo.** O Azure CLI **não pode** ligá-lo, mas ele **pode ler** o pedido salvo e resposta.
- Ele pode salvar **Segredos**, como senhas. **Eliminar essa implantação da história** depois.

## Erros comuns para Azure SQL
| Erro | Causa | Correção |
|---|---|---|
| **InvalidTemplate** | Erro de sintaxe ou má expressão | Corrigir o arquivo, executar a validação ou `bicep build` |
| **DeploymentFailed** | Um wrapper geral | Leia o **Erro interno** nos detalhes da operação |
| **ResourceNotFound / ParentResourceNotFound** | Banco de dados criado antes de seu servidor | Adicionar **`dependsOn`** (JSON) ou **`parent`** (Bicep) |
| **SkuNotAvailable / RegionDoesNotAllowProvisioning** | Nível ou região não oferecida | Escolha outra SKU ou região. Verifique com `az sql db list-editions -l <region> --available` |
| **NameAlreadyExists** | Os nomes dos servidores são **nomes globais DNS** (`<name>.database.windows.net`): 1–63 chars, letras minúsculas, números, hífens, sem hífen no início ou no fim | Usar `uniqueString(resourceGroup().id)` |
| **AuthorizationFailed / RequestDisallowedByPolicy** | Papel RBAC ausente (acesso baseado em papel) ou um Azure Policy o bloqueia (e.g. uma tag necessária) | O papel ou seguir a política |
| **MissingSubscriptionRegistration** | Provedor `Microsoft.Sql` não registrado | `Register-AzResourceProvider -ProviderNamespace Microsoft.Sql` |

## Modos de implantação e reversão
| Modo | O que faz |
|---|---|
| **Incremental** (padrão) | Deixa os recursos que são **E** no modelo sozinho. Mas os recursos que **E-** no modelo obter **Todos** suas propriedades re-aplicado. Uma propriedade que você deixou de fora volta ao seu padrão. |
| **Completar** | Recursos **Exclui** no grupo de recursos que não estão no modelo. A Microsoft está lentamente se aposentando e recomenda **pilhas de implantação** para exclusões. Não é suportado no escopo da assinatura, em modelos aninhados ou no portal. |

**Rollback no erro** reimplanta a última boa implantação se esta falhar:
```powershell
New-AzResourceGroupDeployment -Name sqlDeploy02 -ResourceGroupName rg-sql -TemplateFile main.json -RollbackToLastDeployment
```
```bash
az deployment group create -g rg-sql -n sqlDeploy02 --template-file main.json --rollback-on-error
```
- Adicione um nome de implantação (`-RollBackDeploymentName sqlDeploy01`, ou um nome após `--rollback-on-error`) para voltar a um nome específico.
- O Rollback executa a implantação antiga no **modo completo**. Ele pode recursos **deletar** adicionados desde então.
- Apenas o escopo do grupo de recursos. Os nomes de implantação devem ser exclusivos. It **não reverta dados**.

## Para lembrar na prova
- **Erros de pré-voo não estão no histórico de implantação.** Olhe no **log de atividade**.
- "Mostrar o que vai mudar sem implantar" → **what-if**, não validar.
- **Registro de depuração: PowerShell somente** (`-DeploymentDebugLogLevel`). Apague a história depois.
- **800** por escopo. Limpeza automática em **700 → 600**. Bloqueado por **Fechaduras CanNotDelete**.
- `DeploymentQuotaExceeded` = history full.
- Banco de dados antes do servidor → **`dependsOn` / `parent`**.
- "O nome do servidor já existe" Os nomes → são globais → `uniqueString()`.
- **Incremental** redefine propriedades omitidas. **Completar** exclui extras. Rollback usa modo completo.
- Dê suporte ao **correlação ID**.

## Teste rápido
1. Um comando de implantação falha, mas nada é mostrado nas implantações do grupo de recursos. Onde você olha?
   <details><summary>Resposta</summary>O registro de atividades. Falhas de pré-voo nunca atingem o histórico de implantação.</details>
2. Você quer detalhes brutos de solicitação e resposta para uma implantação SQL falhada. Qual ferramenta ativa isso?
   <details><summary>Resposta</summary>Azure PowerShell com -DeploymentDebugLogLevel. O CLI não pode ligá-lo.</details>
3. As implantações agora falham com o DeploymentQuotaExceeded, mesmo que a limpeza automática deva ser executada. Qual é a causa provável?
   <details><summary>Resposta</summary>O bloqueio A CanNotDelete no grupo de recursos ou assinatura impede a exclusão automática da história antiga.</details>
