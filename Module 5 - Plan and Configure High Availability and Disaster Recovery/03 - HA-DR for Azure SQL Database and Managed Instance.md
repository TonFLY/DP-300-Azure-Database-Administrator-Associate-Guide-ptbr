# HA/DR para Azure SQL Database e Managed Instance

> **Habilidades cobradas:** Avalie soluções HA/DR específicas do Azure, avalie HA/DR para implantações híbridas, Configure active geo-replication, Configure failover groups, Configure Always On availability groups em SQL Managed Instance e Azure VMs
> **Tempo de leitura:** ~12 min

## Conceito central
Em PaaS, a Microsoft já lida com HA dentro de uma região. Você não constrói clusters. Seu job é escolher proteção extra: **redundância de zona** para falhas no datacenter e **active geo-replication** ou **failover groups** para falhas regionais. Para híbrido, o **Managed Instance link** conecta SQL Server a SQL MI.

Pense em alugar um apartamento. O senhorio conserta a caldeira (HA embutido). Você decide se também alugar uma casa de férias em outra cidade em caso de uma inundação (geo-replication ou failover groups).

## Alta disponibilidade incorporada
- Azure SQL Database SLA: **99.99%**. Zona-redundante Business Critical /Premium: **99.995%**.
- Se o hardware falhar, o Azure inicia um novo nó e reacopla o armazenamento ou muda para outra réplica. Isto é automático.
- Durante failover, **conexões abertas drop** e **Transações em andamento roll back**. Os aplicativos devem usar **retry lógica** (tente novamente depois de uma curta espera).
- Business Critical / Premium manter várias réplicas dentro da região. Um pode ser usado para consultas read-only (**read scale-out**).

**Accelerated Database Recovery (ADR)** faz a recuperação do rollback e do crash muito rapidamente, mesmo para transações longas. É **ligado por padrão em PaaS e não pode ser desligado**. É igualmente em SQL Server 2019+.

O Azure também executa verificações de saúde ocultas: verificações automáticas de integridade no estilo **DBCC CHECKDB**, detecção de perda de escrita e leitura obsoleta e **reparação automática de página** de páginas danificadas.

Diferenças PaaS de um VM: você **Não posso.** define um banco de dados OFFLINE ou EMERGENCY, e as configurações failover são limitadas ao que o Azure oferece.

## redundância de zona
- Espalha as réplicas do banco de dados em **availability zones** em uma região. Sobrevive a uma falha no datacenter. Transparente para aplicativos.
- **SQL Database:** General Purpose, Business Critical, Hyperscale e Premium. **Não básico ou padrão.**
- **SQL Managed Instance:** General Purpose e Business Critical.

**Exemplo:** Um aplicativo escolar no Business Critical ativa a redundância de zona. Se um datacenter na região perder energia, o aplicativo continuará funcionando após uma pequena reconexão.

## Active geo-replication (somente Azure SQL Database)
- Cria **bancos de dados secundárias legíveis**, geralmente em **outras regiões**, atualizado **assíncrona** (algumas perda de dados é possível).
- Usa a tecnologia AG nos bastidores.
- Até **4 geo-secundários legíveis** por primário, na mesma ou em diferentes regiões. Acorrente-os para mais.
- Funciona para bancos de dados únicas e elastic pools, incluindo **Hyperscale**. **Não suportado no SQL Managed Instance.**
- O secundário deve usar o **mesmo nível de serviço**. A mesma redundância **tamanho de computação** e backup é fortemente recomendada. Uma causa secundária menor **lag** com gravações pesadas.
- Failover é **manual** (portal, PowerShell, CLI, T-SQL). Depois de failover o **mudanças de nome do servidor**, então os aplicativos devem **cordas de conexão de atualização**.
- **Subscrição cruzada:** mesmo inquilino Microsoft Entra → portal funciona. O inquilino diferente → **SQL authentication + T-SQL** (ou REST API). Não é suportado com autenticação somente Microsoft Entra quando um usuário do Entra o cria.

**Configure-o:** Portal → o banco de dados → **Gestão de dados → Replicas** → **Criar réplica** → escolher o servidor em outra região. Ou:

```bash
az sql db replica create -g rg-prod -s sql-prod-weu -n SalesDb --partner-server sql-prod-neu --partner-resource-group rg-dr
```

```sql
-- In master of the primary server
ALTER DATABASE [SalesDb] ADD SECONDARY ON SERVER [sql-prod-neu] WITH (ALLOW_CONNECTIONS = ALL);
```

**Exemplo:** Um site de viagens no Leste US adiciona um geo-secundário no Oeste US para relatórios e DR. Durante uma tempestade, o DBA falha manualmente e atualiza o aplicativo para o novo nome do servidor.

## Failover groups
**Failover groups** (anteriormente "auto-failover groups") trabalha em **SQL Database e SQL Managed Instance**.
- Replicar **um grupo de bancos de dados** para um servidor ou instância em **Outra região** e falhe-os sobre **Juntos**.
- Dê dois **listeners** (nomes DNS que sempre apontam para o lugar certo):
  - Read-write: `<fog-name>.database.windows.net`
  - Read-only: `<fog-name>.secondary.database.windows.net` (use `ApplicationIntent=ReadOnly`)
  - MI usa `<fog-name>.<zone_id>.database.windows.net`.
- Depois de failover, **mudança connection string** é necessário.
- A replicação é **assíncrono**.

Escopo:
- **SQL Database:** alguns ou todos os bancos de dados em um servidor. Muitos grupos por servidor.
- **SQL MI:** **todas Os bancos de dados de usuários**, **um grupo por instância**. **Bancos de dados do sistema não são replicados** (logins e Agente jobs no mestre / msdb deve ser criado no secundário você mesmo).

### Política Failover
| Política | Valor em CLI / PowerShell | O que acontece |
|---|---|---|
| **Cliente gerenciado (recomendado)** | `Manual` | Você decide quando falhar (planejado ou forçado). Use-o para testar e executar seu plano DR |
| **Microsoft gerenciada** | `Automatic` | A Microsoft força failover de **Todos** tais grupos na região, apenas para um **ampla interrupção regional**, depois de um **período de graça** (min e padrão **1 hora**). Nunca para um grupo. Não confie nele |

- `GracePeriodWithDataLossHours` = how long Azure waits before a forced failover (possível perda de dados). Defina-o mais alto se você não puder pagar a perda de dados.
- `az sql failover-group create` agora padrão para **Manual**.
- **Read-only listener failover** é **desactivado por padrão**: se o secundário estiver offline, o tráfego read-only é **E** enviado para o primário (para proteger o desempenho primário). Você pode ligá-lo.
- **Failback é sempre manual.**

### Configurar um failover group
1. Criar um servidor secundário (SQL DB) ou instância (MI) em outra região.
2. Crie o failover group (portal: servidor → **Failover groups** → **Adicionar grupo**, ou CLI / PowerShell) e adicione bancos de dados.
3. Azure **sementes** (faz a primeira cópia completa) dos bancos de dados secundários.
4. Aponte aplicativos para o nome **listener**.

```bash
az sql failover-group create --name fog-sales -g rg-prod --server sql-prod-weu \
  --partner-server sql-prod-neu --add-db SalesDb --failover-policy Manual
```

**Requisitos MI:** ambas as instâncias no **mesma zona DNS** (set `DNSZonePartner` ao criar o secundário), conectado por **global VNet peering** (recomendado), VPN ou ExpressRoute.

**Exemplo:** Um aplicativo HR SaaS tem bancos de dados de folha de pagamento e despesas. Um failover group se move ambos juntos para o norte da Europa. O aplicativo continua usando `fog-hr.database.windows.net`.

## Réplica em standby sem licença
Se um secundário for usado **apenas para DR** (sem leitura), marque-o como uma réplica **standby**. Você paga **Licença SQL Server** por seu vCores. A computação e o armazenamento ainda são cobrados. Funciona para geo-secundários SQL DB e SQL MI failover groups.

### Cordas de conexão com um failover group
- Aplicações Read-write: `Server=tcp:fog-sales.database.windows.net;...`
- Relatório apps: `Server=tcp:fog-sales.secondary.database.windows.net;ApplicationIntent=ReadOnly;...`
- Não use os nomes do servidor (`sql-prod-weu...`) em aplicativos. Depois de um failover eles apontam para o papel errado.
- Logins: use **usuários de banco de dados contido** (usuários armazenados no banco de dados, então eles se movem com ele). Para o servidor logins em SQL DB, crie o mesmo login com o **mesmo SID** no servidor secundário.

## cenários PaaS HA/DR
| Cenário | Escolha |
|---|---|
| Sobreviver a uma única falha no nó | O que fazer: built-in HA |
| Sobreviver a uma falha do datacenter na região | **redundância de zona** |
| Relatórios de descarregamento na mesma região (Business Critical) | Réplica **Read scale-out** (`ApplicationIntent=ReadOnly`) |
| Cópias legíveis em várias regiões para um banco de dados | **Active geo-replication** |
| Vários bancos de dados devem falhar juntos, mesmo connection string | **Failover group** |
| Região cruzada DR para SQL MI | **Failover group** |
| DR entre on-premises SQL Server 2022 e SQL MI com failback | **Managed Instance link** |
| Nenhuma réplica, região perdida | **Geo-restore** de backups (lição 4) |

**Exemplo:** Um aplicativo de entrega de alimentos é executado em Azure SQL Database Business Critical com redundância de zona (HA). A failover group para a região emparelhado dá DR. Relatórios lidos a partir do read-only listener. O secundário é marcado como uma réplica standby apenas se nenhum relatório usá-lo.

## Geo-replication vs failover groups
| | Active geo-replication | Failover group |
|---|---|---|
| SQL Database | Sim | Sim |
| SQL Managed Instance | **Não** | **Sim** (apenas opção geo- DR para MI) |
| Unidade | Um banco de dados | Grupo de bancos de dados |
| Número de secundários | até 4 | Um (múltiplo é a pré-visualização para SQL DB) |
| A mesma região permitida | Sim | Não, apenas entre regiões |
| Connection string após failover | **Alterações** | **Fica o mesmo** (listener) |
| Automatic failover | Não | Somente com a política gerenciada pela Microsoft |
| Secundário legível | Sim | Sim |

## Always On AGs e SQL Managed Instance: o link MI
Você não pode construir seu próprio AG dentro SQL MI. O HA incorporado já usa a tecnologia AG. Para conectar SQL Server e SQL MI, use o **Managed Instance link**.
- Ele usa um **distributed availability group** sob o capô. Cada link cobre **uma banco de dados**.
- **SQL Server 2016 SP3, 2017 CU31, 2019 CU20**: **one-way** a MI. Bom para migração, leia réplicas e DR. Falha ao MI quebra o link. Não failback.
- **SQL Server 2022 e 2025**: **bidirecional**. Manual **failover e failback**, online. O MI **Política de atualização** deve corresponder (SQL Server 2022 ou 2025, não "Sempre atualizado").
- Edições: Enterprise, Standard, Developer.
- O tráfego usa o MI **ponto final VNet-local**, então você precisa de VPN, ExpressRoute ou VNet.
- DR -apenas MI réplica pode ser **sem licença** (direitos híbridos failover).
- An MI com um link **Não pode estar em um failover group** (e vice-versa).

**Exemplo:** Um banco executa SQL Server 2022 on-premises. Ele liga seu banco de dados principal ao SQL MI no Azure como uma cópia DR. Durante uma interrupção do datacenter, ele falha em MI, depois falha de volta.

## Para lembrar na prova
- PaaS HA é **automático**. Apps precisam de **retry lógica**.
- Redundância de zona: não em Basic/Standard.
- **Geo-replication = SQL Database apenas**, até 4 segundos, manual failover, **mudanças de nome do servidor**.
- **Failover group = SQL DB e MI**, listener, **mudança connection string**, bancos de dados falham juntos.
- MI failover group: **todo usuário DBs**, **um grupo por instância**, sistema DBs **E** replicado, mesmo **zona DNS**.
- Recomenda-se a política de gestão do cliente (`Manual`). A Microsoft gerenciou esperas pelo menos **1 hora** e cobre toda a região.
- Read-only listener failover é **desactivado por padrão**.
- AG com SQL MI → **Managed Instance link** (necessidades bidirecionais SQL Server 2022/2025).
- DR -apenas réplica secundária → **standby**, sem custo de licença.

## Teste rápido
1. Você precisa de cross-region DR para SQL Managed Instance. Qual característica?
   <details><summary>Resposta</summary>failover group. Active geo-replication não está disponível para SQL MI.</details>
2. Um aplicativo deve manter o mesmo connection string após um failover regional de três Azure SQL databases. Qual recurso?
   <details><summary>Resposta</summary>A failover groupporque os aplicativos usam seu read-write listener e os três bancos de dados falham juntos.</details>
3. Você precisa de quatro cópias legíveis de um Azure SQL database em quatro regiões para relatórios locais. Qual característica?
   <details><summary>Resposta</summary>Active geo-replication. Suporta até 4 secundários legíveis. Um failover group tem um secundário.</details>
