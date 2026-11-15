# HA/DR para SQL Server on Azure VMs e híbrido

> **Habilidades cobradas:** Avalie soluções HA/DR específicas do Azure, avalie HA/DR para implantações híbridas, Configure Always On Failover Cluster Instances em Azure VMs, Configure Always On availability groups em SQL Managed Instance e Azure VMs, Configure log shipping
> **Tempo de leitura:** ~17 min

## Conceito central
Em um VM, o Azure mantém o hardware em funcionamento, mas o SQL Server é seu job. Você usa duas camadas. **Recursos do Azure** (conjuntos de disponibilidade, zonas, recuperação do local) protege o VMs. **Características de SQL Server** (FCI, AG, log shipping) protege o SQL Server e os dados. Você geralmente precisa de ambos.

Pense em uma empresa de ônibus escolar. Azure dá-lhe ônibus sobressalentes estacionados em diferentes garagens. Mas você ainda precisa de um motorista treinado e um plano para mover as crianças para o ônibus sobressalente.

## Recursos do Azure que protegem o VM
| Recurso | Protege contra | Scope |
|---|---|---|
| **Conjunto de disponibilidade** | Falha de hardware do host e reinicializações de manutenção do Azure | Um datacenter |
| **Availability zone** | Um datacenter inteiro falhando | Através de datacenters em uma região |
| **Azure Site Recovery (ASR)** | Toda uma região falha | Outra região |

### Conjuntos de disponibilidade
- Espalhe VMs através de **domínios de falha** (poder separado, rede e racks) e **domínios de atualização** (grupos que são reiniciados em diferentes momentos durante a manutenção).
- Dois SQL Server VMs em um conjunto não se sentarão no mesmo servidor físico.
- Use um conjunto de disponibilidade separado para cada camada (web, SQL, controladores de domínio).
- O **E** protege contra problemas SQL Server ou Windows dentro do VM.

### Availability zones
- As zonas 1, 2 e 3 são **Data centers fisicamente separados** em uma região, cada uma com sua própria energia, resfriamento e rede.
- Maior disponibilidade do que os sets.
- A latência de ida e volta entre as zonas é baixa (a Microsoft tem alvos abaixo de cerca de 2 ms), então réplicas **síncrono** AG ainda funcionam em todas as zonas.
- Você **não pode combinar** availability zones e disponibilidade para o mesmo VM.

**Exemplo:** Um hospital coloca SQL VM 1 na zona 1 e SQL VM 2 na zona 2, acompanhado por um AG. Um incêndio no prédio da zona 1 não interrompe o banco de dados.

### Azure Site Recovery (ASR)
- **Replica todo VMs** (discos) para outra região do Azure, ou de on-premises para Azure.
- **Não SQL-consciente.** Não entende transações. RTO e RPO estão no nível VM.
- Gives **crash-consistente** pontos de recuperação a cada 5 minutos. **App-consistente** pontos de uso **VSS** (Volume Shadow Copy Service, um recurso do Windows que congela brevemente o disco escreve para uma cópia limpa snapshot), o que pode afetar SQL Server Se não estiver sintonizado.
- Pode rolar um VM de volta a um ponto antes de uma infecção **ransomware ransomware**.
- **Consistência Multi- VM** recupera vários VMs ao mesmo ponto, mas adiciona sobrecarga. Use-o somente quando precisar.
- Depois de failover, o novo VMs é **não protegido até que você re-ativar a replicação**.
- Bom quando você quer DR simples para muitos VMs, ou SQL nível DR é muito complexo. Para missão crítica SQL, prefira AGs.

## Cluster Windows Server Failover (WSFC) no Azure
Tanto o FCIs quanto o AGs no Windows se sentam em um **WSFC** (um grupo de servidores do Windows que assistem uns aos outros e movem o trabalho quando um falha). O Linux usa o **Pacemaker** em vez disso.

| Termo | Significado |
|---|---|
| **Quorum** | Uma votação que decide se o cluster pode continuar funcionando |
| **Testemunha** | Um voto extra que quebra os laços |
| **Testemunha de nuvem** | Uma testemunha armazenada em Azure Storage. **Recomendado** no Azure e híbrido |
| **AD DS e DNS** | Necessário para um cluster normal. A **grupo de trabalho cluster** pula AD DS, mas ainda precisa de DNS |

Pontos específicos do Azure:
- Use um **testemunha nuvem** (Windows Server 2016+) quando puder. Use um **testemunha disco** quando o FCI usar o **Discos partilhados Azure**. Uma testemunha de compartilhamento de arquivos é o fallback.
- Os endereços IP são definidos **em Azure**, não dentro do VM. O VMs os vê como endereços DHCP.
- Normalmente **uma placa de rede** por VM.
- **Crie o cluster com PowerShell**, não o GUI, para lidar com a rede Azure.

```powershell
Install-WindowsFeature Failover-Clustering -IncludeManagementTools   # on each node
Test-Cluster                                                           # validate first
New-Cluster -Name MyWSFC -Node Node1,Node2 -StaticAddress 10.0.0.5 -NoStorage
# Workgroup cluster (no AD DS): add -AdministrativeAccessPoint DNS
```

- A partir de **Windows Server 2019**, clusters do Azure usam um **DNN** (distributed network name, um nome de cluster que aponta para todos os nodos IPs, então nenhum load balancer é necessário) por padrão em vez de um **VNN** (virtual network name, um floating IP).

## Always On failover cluster instance (FCI)
- Protege o **instância SQL Server inteira**: todas Os bancos de dados, logins, Agente jobs, servidores ligados.
- **Uma cópia dos dados** em **armazenamento compartilhado**. Os nós se revezam possuindo-o.
- Failover **reinicia SQL Server** em outro nó. Os usuários são desconectados brevemente. Mais lento do que um AG failover.
- Os clientes se conectam a um **nome virtual**, então eles não se importam com qual nó está ativo.
- Precisa de **AD DS e DNS**.
- **Opções de armazenamento compartilhado no Azure:** **Discos partilhados Azure**, **compartilhamentos de arquivos Premium**, **Espaços de Armazenamento Direto (S2D)** ou **Azure Elastic SAN** (SQL Server 2022+).

Como os clientes encontram o nó ativo:
- **Sub-redes múltiplas** (cada nó em sua própria sub-rede): **Recomendado**. Sem load balancer e sem necessidade de DNN.
- **Sub-rede única**: use um **DNN** (SQL Server 2019 CU2 +, Windows Server 2016 +, recomendado) ou um **VNN + padrão Azure Load Balancer**.
- Observação: O Basic Load Balancer foi aposentado em 30 de setembro de 2025. Use Standard.

Teste um FCI: Failover Cluster Manager → Papéis → **Mover** → Melhor Nó Possível ou Selecione Nó. Ou `Move-ClusterGroup`. Para discos compartilhados, verifique se cada nó pode pegar os discos (Armazenamento → Discos → Mover Armazenamento Disponível).

**Exemplo:** Uma fábrica executa um aplicativo antigo que precisa de SQL Agent jobs e DTC (Coordenador de Transação Distribuída). Ele usa um FCI de dois nós em discos compartilhados do Azure. Uma falha de nó move toda a instância sem alteração de aplicativo.

## Always On availability groups (AG)
- Protege **uma ou mais bancos de dados**, não a instância inteira.
- **Sem armazenamento compartilhado.** Cada réplica tem sua própria cópia completa, portanto, o custo de armazenamento é maior.
- **failover mais rápido** do que um FCI.
- **Secundários legíveis** para relatórios, e backups pode ser executado em secundários.
- **Commit síncrono** (esperas primárias para o secundário, sem perda de dados, use nas proximidades) ou **commit assíncrono** (sem espera, possível perda de dados, use longe).
- **Edições:** Standard = **básico AG**, 1 database and 2 replicas. Enterprise = many databases, up to **9 réplicas**.
- Objetos de nível de instância (logins, jobs) são copiados **E**. **Contendo AGs** (SQL Server 2022 +) corrige isso: cada um tem seu próprio `master` e `msdb`, então logins, permissões e o Agente jobs falham com o AG.

### Configurar um AG no Azure VMs
1. Construa o WSFC.
2. Habilite o recurso AG (SQL Server Configuration Manager ou PowerShell), então **reiniciar** SQL Server.
3. Crie o AG com SSMS, T-SQL ou PowerShell.
4. Crie o **listener** (aplicativos de um nome usados para alcançar a primária atual).

```powershell
Enable-SqlAlwaysOn -ServerInstance "SQLVM1"
```

Cada réplica tem duas configurações principais:
| Configuração | Escolhas | Significado |
|---|---|---|
| **Modo de disponibilidade** | `SYNCHRONOUS_COMMIT` / `ASYNCHRONOUS_COMMIT` | Espere o secundário endurecer o tronco, ou não |
| **modo Failover** | `AUTOMATIC` / `MANUAL` | Automatic failover precisa de **commit síncrono** em ambos os lados |

Exemplo T-SQL curto (executar no primário):
```sql
CREATE AVAILABILITY GROUP [AG1]
FOR DATABASE [SalesDb]
REPLICA ON
 'SQLVM1' WITH (ENDPOINT_URL = 'TCP://SQLVM1.contoso.com:5022',
   AVAILABILITY_MODE = SYNCHRONOUS_COMMIT, FAILOVER_MODE = AUTOMATIC),
 'SQLVM2' WITH (ENDPOINT_URL = 'TCP://SQLVM2.contoso.com:5022',
   AVAILABILITY_MODE = SYNCHRONOUS_COMMIT, FAILOVER_MODE = AUTOMATIC,
   SECONDARY_ROLE (ALLOW_CONNECTIONS = READ_ONLY));
-- Then on SQLVM2: ALTER AVAILABILITY GROUP [AG1] JOIN;
```
- O banco de dados deve usar o modelo de recuperação **FULL** e ter um **full backup** antes que ele possa participar.
- **Porto 5022** é o **banco de dados espelhando endpoint** padrão que as réplicas usam para falar. Abra-o no NSGs e no firewall do Windows.
- **Automatic seeding** (`SEEDING_MODE = AUTOMATIC`) copia o banco de dados para o secundário para você. Caso contrário, você faz backup e restaurá-lo `WITH NORECOVERY` você mesmo.

Opções Listener no Azure:
- **Sub-redes múltiplas** (replicas em diferentes sub-redes): **Recomendado**. O listener tem um IP por sub-rede. No load balancer.
- **DNN listener** (SQL Server 2016 SP3 / 2017 CU25 / 2019 CU8+, Windows Server 2016+).
- **VNN listener em uma sub-rede**: precisa de um **Padrão interno load balancer** com **Floating IP** (Retorno do Servidor Direto) ligado, e um **porto da ponta de sonda** (como **59999**) definido no recurso listener IP.

```powershell
Get-ClusterResource "AG1_IP" | Set-ClusterParameter ProbePort 59999
Test-NetConnection AG1-Listener -Port 1433   # check the listener answers
```

**Exemplo:** Uma loja online coloca duas réplicas de sincronização em duas zonas (HA, automatic failover) e uma réplica assíncrona em outra região (DR). Os relatórios são executados no secundário legível.

## availability groups distribuído
- An **"AG de AGs"**: liga dois AGs separados, cada um em seu **próprio cluster** com seu próprio quórum e testemunha.
- Adicionado em SQL Server 2016 (Enterprise).
- Bom para **multi-região DR** e **on-premises para Azure**. Nenhum único cluster se estendia por regiões.
- Cada lado gerencia a si mesmo, e **failback** é mais simples.
- No Azure, cada região precisa de seu próprio load balancer para a porta de endpoint **5022** (quando estiver usando o VNN).

## AG estendido através de regiões ou híbrido
- Um AG com réplicas em duas regiões ou on-premises mais Azure. Uma característica dá HA e DR.
- Funciona em Standard ou Enterprise.
- Precisa de uma boa rede, **AD DS e DNS em todos os sites**, e cuidado **colocação de testemunhas** para evitar **cérebro dividido** (dois nós ambos pensando que eles são primários).

## Log shipping
- Protege **bancos de dados**. Leva **transaction log backups** no primário, **cópias**-los para um secundário, e **restaura**-los lá em uma programação.
- Cria um **em standby quente** (uma cópia que está minutos atrás e pronta para trazer online).
- **Somente Manual failover.** Alguma perda de dados é esperada, a menos que o failover seja planejado.
- **Simples, barato, confiável.** Funciona mesmo em redes lentas ou não confiáveis.
- Pode proteger um **FCI** em um site DR, porque um FCI não copia dados por si só.
- Precisa do modelo de recuperação **FULL** ou **BULK_LOGGED**.
- **assistente SSMS** (Propriedades de Banco de dados → Transaction Log Shipping) ou **T-SQL**. Três SQL Agent jobs fazem o trabalho: **backup** (no primário), **cópia** e **restaurar** (no secundário). Um **servidor monitor** opcional registra histórico e levanta alertas.
- Pegue qualquer log extra backups com **COPY_ONLY**, ou você quebra a cadeia.

Passos para configurar:
1. Defina o banco de dados para recuperação **FULL** e pegue um full backup.
2. Crie um **pasta compartilhada** para o log backups que o secundário possa ler.
3. Inicie o assistente (ou T-SQL) no primário. Defina o cronograma backup e a pasta backup.
4. Adicione o servidor secundário. Escolha como inicializá-lo (novo full backup, backup existente ou já restaurado).
5. Escolha o modo de restauração no secundário:
   - **Nenhum modo de recuperação**: o banco de dados não pode ser lido.
   - **modo de espera**: read-only entre restaurações. Os usuários são desconectados durante cada restauração.
6. Opcionalmente, adicione um **servidor monitor**.

Os procedimentos armazenados T-SQL começam com `sp_add_log_shipping_` (por exemplo, `sp_add_log_shipping_primary_database` no primário e `sp_add_log_shipping_secondary_database` no secundário).

To **falhar sobre**: tomar a **tail-log backup** do primário se ainda for alcançável (`BACKUP LOG ... WITH NORECOVERY`), copiar e restaurar todos os registros restantes no secundário, em seguida, restaurar o último `WITH RECOVERY`. Reaponte os aplicativos, porque log shipping tem **listener**.

**Exemplo:** Um site de notícias envia logs a cada 15 minutos para um VM em outra região. Se a região principal falhar, o DBA restaura os últimos logs e traz o standby online.

## FCI vs AG vs log shipping
| | FCI | AG | Log shipping |
|---|---|---|---|
| Protectores | instância inteira | Bancos de dados | Bancos de dados |
| Armazenamento compartilhado | Necessário | Não | Não |
| Cópias de dados | Um | Um por réplica | Um por secundário |
| Automatic failover | Sim | Sim (replicações sincronizadas) | **Não** |
| Cópia legível | Não | Sim | Somente no modo de espera, entre restaurações |
| Complexidade | Alta | Média | Baixa |
| Custo | Médio-alto | Alta | Baixa |

## Qual solução VM quando
| Cenário | Solução |
|---|---|
| HA em uma região | AG ou FCI |
| DR entre regiões do Azure | AG entre regiões ou AG distribuído |
| DR de on-premises para Azure | AG ou log shipping distribuído (ou MI link para SQL MI) |
| Simples VM - nível DR, muitos VMs | Azure Site Recovery |
| Proteger objetos em nível de instância | FCI (ou contém AG) |
| Mais barato DR, trabalho manual OK | Log shipping |

## Híbrido HA/DR
**híbrido** significa on-premises SQL Server mais Azure.
- Usos para Azure: um **nuvem DR site**, uma área de estadiamento **Migração**, **ler réplicas** para relatórios e **armazenamento barato backup** em Blob Storage.
- Principalmente **IaaS**: as características usuais do SQL (AG, AG distribuído, log shipping, FCI com log shipping) funcionam em todos os sites.
- Precisa de **AD DS, DNS e WSFC** para projetos baseados em AG.
- **A rede é a chave.** Use **ExpressRoute** (link privado dedicado, preferido) ou **site-site VPN** (mais barato, mais lento). **Nunca** expor SQL VMs para a Internet pública.
- PaaS não pode se juntar a um on-premises AG ou cluster. Exceções:
  - **Managed Instance link** (lição 3): DR entre SQL Server e SQL MI.
  - **Transactional replication**: on-premises SQL Server (editor) pode empurrar para SQL MI ou SQL Database (assinante). SQL MI também pode ser um editor e empurrar de volta para SQL Server. SQL Database só pode ser um assinante.
- Mesmo sem híbrido DR, você pode **backup de bancos de dados on-premises para Azure Blob Storage** (ver lição 4).

**Exemplo:** Um fabricante mantém sua réplica AG primária em seu próprio datacenter e uma secundária assíncrona em uma Azure VM, ligada por ExpressRoute, com uma testemunha de nuvem.

## Para lembrar na prova
- Availability set = one datacenter. Zone = across datacenters. Site Recovery = across regions.
- Conjuntos e zonas protegem o **VM**, não o SQL Server. Você ainda precisa do FCI / AG / log shipping.
- ASR é **E SQL-aware**. Escolha-o para "simples DR para muitos VMs".
- WSFC no Azure: **testemunha nuvem**, **PowerShell** para criar, IPs conjunto **em Azure**.
- FCI = whole instance + **armazenamento compartilhado** (discos compartilhados, compartilhamentos de arquivos Premium, S2D, Elastic SAN).
- Multi-subnet = no load balancer. Single subnet = **DNN** (preferido) ou VNN + **Padrão** LB com **porto da ponta de sonda** e **Floating IP**.
- AG Standard edition = basic AG, 1 DB, 2 replicas. Enterprise = up to 9 replicas.
- Log shipping = **manual failover**mais barato, funciona em redes fracas.
- Híbrido: **ExpressRoute** preferido, nunca internet pública.

## Teste rápido
1. O seu VMs deve sobreviver a uma interrupção total do datacenter dentro de uma região. O que você usa?
   <details><summary>Resposta</summary>Availability zones. Os conjuntos de disponibilidade protegem somente dentro de um datacenter.</details>
2. O AG tem ambas as réplicas em uma sub-rede com um VNN listener. Os aplicativos não podem chegar ao listener após o failover. O que você verifica?
   <details><summary>Resposta</summary>Sonda de saúde load balancer: a porta de sonda no recurso listener IP e Floating IP. Melhor: use um design DNN ou multi-subnet.</details>
3. Você precisa de DR barato para outra região e pode aceitar manual failover e alguns minutos de perda de dados. Qual recurso?
   <details><summary>Resposta</summary>Log shipping. É simples e de baixo custo, mas não tem automatic failover.</details>
