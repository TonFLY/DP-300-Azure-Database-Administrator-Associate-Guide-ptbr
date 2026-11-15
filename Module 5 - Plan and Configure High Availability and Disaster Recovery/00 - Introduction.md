# Módulo 5 — Planejar e configurar um ambiente de alta disponibilidade e recuperação de desastres (HA/DR)

**Peso do exame:** 20–25% · **Tempo de leitura:** ~71 min · **Arquivos:** 6

## O que você vai aprender
Você vai aprender a manter o banco de dados disponível durante falhas e a recuperá-lo após um desastre. O módulo ajuda a escolher a ferramenta para cada opção Azure SQL: SQL Server em uma VM, Azure SQL Database e Azure SQL Managed Instance. Também cobre backup e restore por T-SQL e pelos backups automáticos do Azure, além de como testar, monitorar e diagnosticar a configuração de HA/DR.

Pense em um hospital ocupado. Se um gerador falhar, um segundo começa de uma vez e as luzes ficam acesas. Isso é **alta disponibilidade**. Se todo o edifício inundar, a equipe se mudar para um segundo hospital em outra cidade. Isso é **recuperação de desastre**. Toda noite o hospital também copia todos os registros de pacientes para um arquivo seguro. Isso é **backup**. O hospital executa exercícios de incêndio para provar que tudo isso realmente funciona. Isso é **Ensaios**.

## Lições neste módulo
| # | Lição | O que ela cobre | Tempo |
|---|--------|----------------|------|
| 1 | [Noções básicas de HA/DR - RPO, RTO e escolher uma estratégia](<01 - HA-DR basics - RPO, RTO and choosing a strategy.md>) | RPO, RTO, HA vs DR, IaaS vs PaaS, escolhendo uma ferramenta | ~9 min |
| 2 | [HA/DR para SQL Server on Azure VMs e híbrido](<02 - HA-DR for SQL Server on Azure VMs and hybrid.md>) | Clusters, FCI, availability groups, log shipping, Recuperação de local, híbrido | ~17 min |
| 3 | [HA/DR para Azure SQL Database e Managed Instance](<03 - HA-DR for Azure SQL Database and Managed Instance.md>) | Built-in HA, zonas, geo-replication, failover groups, MI link | ~12 min |
| 4 | [Backup e restaurar](<04 - Backup and restore.md>) | T-SQL backup /restauração, backup a URL, automatizado backups, PITR, LTR, geo-restore | ~15 min |
| 5 | [Monitor, teste e solução de problemas HA/DR](<05 - Monitor, test and troubleshoot HA-DR.md>) | brocas DR, comandos failover, DMVs, erros comuns | ~13 min |

## Palavras que você vai encontrar
| Palavra | Significado simples |
|------|----------------|
| HA (alta disponibilidade) | Fique online quando uma parte falhar, geralmente na mesma região |
| DR (recuperação do desastre) | Voltar a ficar online após um grande fracasso, geralmente em outra região |
| RTO (objetivo de tempo de recuperação) | A maior parte do tempo de inatividade que o negócio pode aceitar |
| RPO (objetivo de ponto de recuperação) | A maioria dos dados (medidos no tempo) que a empresa pode dar ao luxo de perder |
| Failover | Mudar o trabalho da cópia quebrada para uma cópia saudável |
| Replica | Cópia de uma banco de dados atualizada |
| síncrono/assíncrono | Aguarde a cópia confirmar / não espere |
| WSFC | Windows Server Failover Cluster, o recurso do Windows que gerencia failover |
| FCI | Failover cluster instance, protege uma instância SQL Server inteira |
| AG | Always On availability group, protege um conjunto de bancos de dados |
| Listener | Um aplicativo de nome fixo se conecta, seja qual for a cópia primária |
| Failover group | Funcionalidade do Azure que falha num grupo de bancos de dados com um listener |
| PITR | Point-in-time restore, restaurar a um minuto exato |
| LTR | Long-term retention, mantenha o backups por até 10 anos |

## exame de habilidades cobertas
- [ ] Recomendar estratégia HA/DR com base nos requisitos RPO / RTO → 1
- [ ] Avaliar HA/DR para implantações híbridas → 2, 3
- [ ] Avalie soluções HA/DR específicas do Azure → 1, 2, 3
- [ ] Planear um procedimento de teste para uma solução HA/DR → 5
- [ ] Recomende um banco de dados backup e restaure a estratégia → 4
- [ ] Executar um banco de dados backup usando ferramentas nativas → 4
- [ ] Executar uma restauração de banco de dados usando ferramentas nativas → 4
- [ ] Executar uma restauração de banco de dados para um ponto no tempo → 4
- [ ] Configurar a retenção backup de longo prazo → 4
- [ ] Backup e restaurar um banco de dados usando T-SQL → 4
- [ ] Backup para e restaurar a partir do armazenamento em nuvem → 4
- [ ] Configurar active geo-replication → 3
- [ ] Configurar Always On availability groups em SQL Managed Instance e Azure VMs → 2, 3
- [ ] Configurar failover groups → 3
- [ ] Configurar Always On Failover Cluster Instances em Azure VMs → 2
- [ ] Configurar log shipping → 2
- [ ] Monitorar uma solução HA/DR → 5
- [ ] Solução HA/DR Solução → 5
