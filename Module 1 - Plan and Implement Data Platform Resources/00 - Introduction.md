# Módulo 1 — Planejar e implementar recursos da plataforma de dados

**Peso do exame:** 15–20% · **Tempo de leitura:** ~88 min · **Arquivos:** 10

## O que você vai aprender
Você vai aprender as três maneiras principais de executar SQL Server no Azure, como escolher uma delas e como dimensionar e escalar cada opção. Também vai aprender a dividir tabelas grandes com partitioning, reduzir o espaço com compression e distribuir dados entre vários bancos com sharding. Por fim, você verá como migrar bancos existentes para o Azure e diagnosticar problemas na migração.

Pense em mudar uma família para uma nova cidade. Primeiro você escolhe uma casa: sua própria casa (a VM), um apartamento com serviço (Managed Instance) ou um quarto de hotel (SQL Database). Então você escolhe o tamanho certo e organiza os quartos. Em seguida, você planeja a mudança: empacote tudo em um fim de semana (offline) ou mova o quarto por quarto enquanto ainda mora lá (on-line). Por último, você lida com caixas que se perderam.

## Lições neste módulo
| # | Lição | O que ela cobre | Tempo |
|---|--------|----------------|------|
| 1 | [Escolha a opção Azure SQL certa](<01 - Choose the right Azure SQL option.md>) | VM vs Managed Instance vs SQL Database, Arco, Tecido | ~10 min |
| 2 | [Implantar e escalar Azure SQL Database e Managed Instance](<02 - Deploy and scale Azure SQL Database and Managed Instance.md>) | DTU / vCore, níveis, serverless, piscinas, Hyperscale, opções MI | ~10 min |
| 3 | [SQL Server on Azure VMs, híbrido e patching](<03 - SQL Server on Azure VMs, hybrid and patching.md>) | Tamanho VM, discos, IaaS Agent, híbrido, Gestor de Atualização | ~11 min |
| 4 | [Particionamento, compressão e sharding](<04 - Partitioning, compression and sharding.md>) | Partition function /scheme, ROW / PAGE, Teclas shard | ~13 min |
| 5 | [Planear uma migração](<05 - Plan a migration.md>) | Estágios, online vs offline, escolha de ferramentas | ~8 min |
| 6 | [Migrar para Azure SQL Database](<06 - Migrate to Azure SQL Database.md>) | DMS, BACPAC, replicação, bcp, ADF | ~8 min |
| 7 | [Migrar para Azure SQL Managed Instance](<07 - Migrate to Azure SQL Managed Instance.md>) | LRS, link MI, migração Arc, copiar e mover | ~10 min |
| 8 | [Migrar para SQL Server on Azure VMs](<08 - Migrate to SQL Server on Azure VMs.md>) | Backup a URL, distribuído AG, outros métodos | ~6 min |
| 9 | [Resolver uma migração](<09 - Troubleshoot a migration.md>) | Erros e correções comuns | ~9 min |

## Palavras que você vai encontrar
| Palavra | Significado simples |
|------|----------------|
| IaaS | Azure dá-lhe um VM; você gerencia o OS e SQL Server. |
| PaaS | A Microsoft executa o servidor, OS e patches; você gerencia dados. |
| vCore | Um virtual CPU. Você compra computação por vCores. |
| DTU | Unidade de Transação de Banco de Dados. Um pacote fixo de CPU, memória e velocidade do disco. |
| Elastic pool | Muitos bancos de dados compartilham um pote de computação. |
| Hyperscale | Nível SQL Database que cresce para 128 TB. |
| Arco Azure | Permite que o Azure gerencie servidores que vivem fora do Azure. |
| Partição | Uma fatia de uma tabela, por exemplo, um mês de pedidos. |
| Shard | Um dos muitos bancos de dados que cada um detém parte das linhas. |
| Migração online | Fonte continua funcionando; tempo de inatividade apenas no interruptor final. |
| Migração offline | A fonte está parada para toda a cópia. |
| Cutover | Os aplicativos de momento mudam do banco de dados antigo para o novo. |
| DMS | Azure Database Migration Service, o principal motor de migração. |
| LRS | Log Replay Service. Restaura o seu backups em Managed Instance. |
| link MI | Cópia ao vivo de SQL Server para Managed Instance usando um distributed availability group. |

## exame de habilidades cobertas
- [ ] Recomendar uma oferta de banco de dados com base em requisitos específicos → 1
- [ ] Escolha um método de implantação automatizada → 2 (curto), detalhes completos no Módulo 4
- [ ] Identificar casos de uso para os serviços SQL habilitados para Azure Arc → 1, 3
- [ ] Identificar casos de uso para Azure SQL Database em Microsoft Fabric → 1
- [ ] Plano para particionamento de tabela → 4
- [ ] Recomendar um banco de dados sharding solução → 4
- [ ] Implantar ofertas de banco de dados em plataformas selecionadas → 2, 3
- [ ] Implantar soluções híbridas SQL Server → 3
- [ ] Aplicar patches e atualizações para implantações híbridas e IaaS → 3
- [ ] Configurar Azure SQL Database para escala e desempenho → 2
- [ ] Configurar Azure SQL Managed Instance para escala e desempenho → 2
- [ ] Configurar SQL Server on Azure Virtual Machines para escala e desempenho → 3
- [ ] Configurar o particionamento de tabela → 4
- [ ] Configurar data compression → 4
- [ ] Avaliar os requisitos para uma migração → 5
- [ ] Avaliar estratégias de migração offline ou online → 5
- [ ] Implementar uma estratégia de migração online → 6, 7, 8
- [ ] Implementar uma estratégia de migração offline → 6, 8
- [ ] Implementar uma migração para o Azure → 6, 7, 8
- [ ] Implementar uma migração entre os serviços Azure SQL → 6, 7
- [ ] Implementar Azure SQL Managed Instance database copy e mover → 7
- [ ] Solucionar problemas de migração → 9
