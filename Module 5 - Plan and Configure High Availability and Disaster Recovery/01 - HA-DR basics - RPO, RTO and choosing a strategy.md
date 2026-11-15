# Noções básicas de HA/DR - RPO, RTO e escolher uma estratégia

> **Habilidades cobradas:** Recomendar a estratégia HA/DR com base nos requisitos RPO / RTO, Avaliar soluções HA/DR específicas do Azure
> **Tempo de leitura:** ~9 min

## Conceito central
Antes de escolher qualquer ferramenta HA/DR, a empresa deve responder a duas perguntas. Quanto tempo podemos ficar para baixo? Quantos dados podemos perder? As respostas são o **RTO** e o **RPO**. A ferramenta que você escolhe deve atender a ambos, a um custo que a empresa aceita.

Pense em uma pizzaria. Se o forno quebrar, quanto tempo a loja pode ficar fechada (RTO)? Se o livro de pedidos queimar, quantas ordens ele pode esquecer (RPO)?

## RTO - objetivo de tempo de recuperação
- **RTO** = the **tempo mais longo** O sistema pode estar desligado após uma falha.
- A pergunta responde: "Quanto tempo a empresa pode se dar ao luxo de ficar para baixo?"
- Pode ser definido para um sistema inteiro ou para uma parte (uma instância, um banco de dados).
- O **A parte mais lenta define o verdadeiro RTO**. Se o banco de dados estiver de volta em 5 minutos, mas o servidor do aplicativo precisar de 20 minutos, o total RTO São 20 minutos.

**Exemplo:** Um site de ingressos online tem um RTO de 30 minutos. Um servidor falha às 9:00. O site deve estar funcionando novamente às 9:30.

## RPO - objetivo do ponto de recuperação
- **RPO** = the **A maioria dos dados, medidos no tempo**Você pode perder.
- Pergunta responde: "Quanta perda de dados é aceitável?"
- O seu **backup ou frequência de cópia** limita o seu RPO. Se o registro backups correr a cada 30 minutos, o seu RPO não pode ser melhor do que 30 minutos.

**Exemplo:** Um aplicativo bancário tem um RPO de 15 minutos. Uma falha atinge as 10:00. Os dados devem ser reembolsáveis às 9:45 ou mais tarde. Os pagamentos feitos entre 9:45 e 10:00 podem ser perdidos.

## Como definir RTO e RPO
- **Os requisitos do negócio orientam essa escolha.** Se uma hora de downtime custar US$ 10.000, pode valer a pena investir em uma solução mais rápida e mais cara.
- O tempo de inatividade **Equilíbrio custo e risco.** Zero e a perda zero de dados são muito caros e muitas vezes não práticos.
- **Use metas realistas.** Se o restore leva 3 horas e o RTO é de 2 horas, o plano não atende à meta.
- **Anote-os, teste-os e revise-os** regularmente.

## HA vs DR
| | Alta disponibilidade (HA) | Recuperação de desastres (DR) |
|---|---|---|
| Protege contra | Um servidor, disco ou datacenter falhando | Todo um site ou região falhando |
| Onde mora a cópia | mesma região (mesma data center ou outra zona) | Outra região (ou on-premises para Azure) |
| Típico RTO / RPO | Segundos a minutos | Minutos a horas |
| Exemplo | Failover para outro nó na mesma região | Mudar para uma cópia em outra região |

**Exemplo:** Um aplicativo de loja tem duas cópias nas zonas da Europa Ocidental (HA) e mais uma cópia no Norte da Europa (DR). Um corte de energia de zona usa a cópia HA. Uma interrupção regional completa usa a cópia DR.

Backups não são HA. Eles são o **last line of defence** contra erros, bugs, ransomware e desastres. Você precisa deles mesmo quando você tem HA e DR.

## IaaS vs PaaS - quem faz o trabalho
| | IaaS: SQL Server em Azure VM | PaaS: Azure SQL Database / Managed Instance |
|---|---|---|
| Quem gerencia SQL Server e OS | Você | Microsoft |
| HA | Você o constrói (AG, FCI) mais conjuntos de disponibilidade / zonas | Construído em, automático |
| DR | Você o constrói (AG, distribuído AG, log shipping, Recuperação de Site) | Você liga (geo-replication, failover groups) |
| Backups | Você os criou | Automático |
| Controle | Completo | Limitado, mas muito menos trabalho |

**Exemplo:** Uma empresa com um aplicativo antigo que precisa de SQL Server Agent jobs e configurações de nível de instância pode escolher um VM e construir um AG. Um novo aplicativo SaaS escolhe Azure SQL Database e simplesmente adiciona um failover group.

## Escolhendo uma estratégia
| Se você precisar... | Escolha |
|---|---|
| HA para uma instância SQL Server inteira (logins, jobs, tudo) no VMs | **Failover cluster instance (FCI)** |
| HA rápido para algumas bancos de dados, com cópias legíveis, em VMs | **Always On availability group (AG)** |
| DR simples e barato com manual failover | **Log shipping** |
| DR entre regiões ou clusters para VMs | **AG entre regiões** ou **distribuído AG** |
| DR simples para VMs inteiro, não SQL -ciente | **Azure Site Recovery** |
| Sobreviver a uma falha do datacenter para VMs | **Availability zones** |
| Sobreviver a hardware/manutenção de host em um datacenter | **Conjuntos de disponibilidade** |
| PaaS HA em uma região | **Built-in HA**, adicione **redundância de zona** |
| PaaS DR, cópias legíveis, Azure SQL Database apenas | **Active geo-replication** |
| PaaS DR, várias bancos de dados em conjunto, nenhuma mudança connection string | **Failover group** |
| Corrigir erro humano (linhas excluídas) | **Point-in-time restore** |
| Mantenha backups por anos (conformidade) | **Long-term retention (LTR)** |

**Exemplo:** Um site de comércio eletrônico precisa de RPO de 5 minutos e RTO de 10 minutos em Azure SQL Database. log automático backups dar PITR. A failover group dá DR regional rápido sem alterar o connection string do aplicativo.

## Da exigência ao design: três casos trabalhados
Leia as palavras obrigatórias em uma pergunta e mapeie-as para uma ferramenta.

**Caso 1 - "o menor custo, algumas horas de inatividade é bom".** Uma pequena instituição de caridade executa SQL Server em uma Azure VM. RTO é de 8 horas, RPO é de 1 hora. Registro por hora backups a Blob Storage (backup a URL) com armazenamento geo-redundante são suficientes. Um segundo servidor sempre em execução seria desperdiçado dinheiro.

**Caso 2 - "perda de dados quase zero dentro da região, automatic failover".** Um hospital corre SQL Server em VMs. RPO está perto de zero e RTO está abaixo de um minuto. Use um **AG com duas réplicas síncronas em dois availability zones** e automatic failover. Adicione uma réplica assíncrona em outra região para DR.

**Caso 3 - "esforço mínimo de administração, DR regional, aplicativos não devem mudar".** Uma empresa de SaaS usa Azure SQL Database. Use o HA integrado mais **redundância de zona** e um **failover group** para outra região. Os aplicativos usam o listener, portanto, nada muda depois do failover.

Palavras-chave a serem observadas:
| Pergunta diz | Pensar |
|---|---|
| "minimizar o esforço administrativo" | PaaS built-in características |
| "sem perda de dados" | Commit síncrono, planned failover |
| "custo mínimo" | Log shipping, backups, LRS para não-crítico |
| "objetos de nível de instância" | FCI ou contido AG |
| "sem alterar as cordas de conexão" | Failover group ou AG listener |

## Para lembrar na prova
- **RTO = time** para voltar. **RPO = data** você pode perder (a tempo).
- Limites de frequência Backup RPO. Limites de velocidade de restauração RTO.
- O componente mais lento decide o RTO real.
- HA = same region, short RTO. DR = other region, longer RTO.
- IaaS: você constrói HA/DR. PaaS: HA está embutido, você configura DR.
- "Proteger toda a instância" → FCI. "Proteger algumas bancos de dados, cópias legíveis" → AG.
- "Mais barato DR, manual failover é OK " → log shipping.
- "Nenhuma mudança connection string após failover " → failover group.

## Teste rápido
1. Log backups correr a cada hora. O negócio pede um RPO de 10 minutos. O plano é bom?
   <details><summary>Resposta</summary>Não. Até uma hora de dados pode ser perdido. Tome log backups pelo menos a cada 10 minutos, ou use uma réplica.</details>
2. Um banco de dados recupera em 5 minutos, mas a camada web leva 25 minutos para reiniciar. Qual é o verdadeiro RTO?
   <details><summary>Resposta</summary>25 minutos. A parte mais lenta define o RTO.</details>
3. Você precisa de HA para SQL Server em VMs em uma região e também deve manter o logins e o Agente jobs junto com os dados. Qual recurso SQL protege toda a instância?
   <details><summary>Resposta</summary>A failover cluster instance (FCI). An AG protege apenas os bancos de dados (a menos que seja um AG).</details>
