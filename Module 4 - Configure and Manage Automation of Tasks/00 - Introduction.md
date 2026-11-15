# Módulo 4 — Configurar e gerenciar a automação de tarefas

**Peso do exame:** 15–20% · **Tempo de leitura:** ~50 min · **Arquivos:** 7

## O que você vai aprender
Você vai criar recursos Azure SQL com código, diagnosticar falhas de deployment e agendar jobs com SQL Server Agent. Também vai conhecer as alternativas de automação para Azure SQL Database e configurar alertas para falhas em tarefas.

Imagine uma rede de pizzarias. Cada loja nova segue uma mesma planta, ou **blueprint** (automação de deployment). À noite, o gerente segue um checklist para limpar a loja e contar o dinheiro (SQL Server Agent). Nas lojas sem gerente noturno, a sede executa o checklist (Elastic Jobs, Azure Automation e Logic Apps). Se uma tarefa falha, o responsável recebe um aviso (alertas).

## Lições neste módulo
| # | Lição | O que ela cobre | Tempo |
|---|--------|----------------|------|
| 1 | [Automatizar implantações](<01 - Automate deployments.md>) | Declarativo vs imperativo, ARM, Bicep, PowerShell, Azure CLI | ~8 min |
| 2 | [Monitorar e diagnosticar problemas em implantações](<02 - Monitor and troubleshoot deployments.md>) | What-if, histórico de implantação, registro de atividade, erros comuns, reversão | ~8 min |
| 3 | [Jobs do SQL Server Agent](<03 - SQL Server Agent jobs.md>) | Jobs, horários, planos de manutenção, operadores, alertas, Database Mail | ~8 min |
| 4 | [Diagnosticar problemas em jobs do SQL Server Agent](<04 - Troubleshoot SQL Server Agent jobs.md>) | História Job, proprietários, proxies, falta de e-mails, AG jobs, limites MI | ~7 min |
| 5 | [Automatizar tarefas no Azure](<05 - Automate tasks in Azure.md>) | Elastic jobs, Azure Automation runbooks, Logic Apps, que ferramenta quando | ~9 min |
| 6 | [Alertas, monitoramento e diagnóstico de tarefas automatizadas](<06 - Alerts, monitoring and troubleshooting automated tasks.md>) | Azure Monitor alerts, grupos de ação, história job, Log Analytics | ~5 min |

## Palavras que você vai encontrar
| Palavra | Significado simples |
|------|----------------|
| Infraestrutura como Código (IaC) | Descrever servidores e bancos de dados em um arquivo de código, para que você possa construí-los novamente e novamente da mesma maneira |
| Modelo ARM | Arquivo JSON que descreve os recursos do Azure |
| Bicep | Uma linguagem mais simples para a mesma coisa; ela se transforma em ARM JSON quando você implanta |
| SQL Server Agent | O programador job integrado do SQL Server |
| Operador | Uma pessoa ou grupo de email que recebe e-mails job |
| Database Mail | Recurso SQL Server que envia e-mail |
| Elastic job | A T-SQL job que roda em muitos Azure SQL databases de uma só vez |
| Runbook | script PowerShell ou Python executado por Azure Automation |
| App Lógica | Um fluxo de trabalho de baixo código, desenhado em um designer, que reage a eventos |
| Grupo de acção | A lista de "quem dizer e o que fazer" quando um alerta do Azure dispara |

## exame de habilidades cobertas
- [ ] Gerenciar horários para manutenção regular jobs → 3
- [ ] Configurar alertas e notificações job → 3, 4
- [ ] Solução de problemas SQL Server Agent jobs → 4
- [ ] Automatize a implantação usando modelos ARM e Bicep → 1
- [ ] Automatizar a implantação usando Azure PowerShell → 1
- [ ] Automatizar a implantação usando Azure CLI → 1
- [ ] Monitorar e solucionar problemas de implantação → 2
- [ ] Criar e configurar elastic jobs → 5
- [ ] Criar e configurar tarefas de banco de dados usando automação → 5
- [ ] Configurar alertas e notificações sobre tarefas de banco de dados → 3, 6
- [ ] Resolver problemas automatizados de banco de dados tarefas → 6
