# Implantar e escalar Azure SQL Database e Managed Instance

> **Habilidades cobradas:** Implantar ofertas de banco de dados em plataformas selecionadas, Configurar Azure SQL Database para escala e desempenho, Configurar Azure SQL Managed Instance para escala e desempenho
> **Tempo de leitura:** ~10 min

## Conceito central
Quando você cria um banco de dados PaaS, você faz três escolhas. Como você **Pagar** (DTU ou vCore). Qual **nível** (General Purpose, Business Critical, Hyperscale). Como você **partilha** computa (um banco de dados ou uma piscina). Escalar mais tarde significa mudar essas escolhas.

Analogia: alugar um carro. Um DTU é um pacote fixo ("carro pequeno"). Um vCore permite que você escolha o tamanho do motor e da bota. Um elastic pool é uma piscina de carros que várias pessoas compartilham.

## Modelos de implantação para SQL Database
- **Banco de dados única:** um banco de dados com seu próprio computador. Faturado e dimensionado por banco de dados.
- **Elastic pool:** muitos bancos de dados compartilham um pote de computação e armazenamento.
- Ambos se sentam em um **servidor lógico**. É um nome e ponto login (`myserver.database.windows.net`), não uma máquina real. As regras de firewall vivem nele.

## Modelos de compra
**modelo DTU** (Database Transaction Unit, uma mistura fixa de CPU, memória e IO):
- Níveis: **Básico, Standard, Premium**.
- Simples, mas você não pode ajustar CPU e armazenamento separadamente.
- **Não disponível no Managed Instance.**

**modelo vCore** (você escolhe vCores, a memória segue, o armazenamento é separado):
- Níveis: **General Purpose, Business Critical, Hyperscale**.
- Funciona com **Azure Hybrid Benefit** (reutilize suas licenças SQL Server para economizar dinheiro).
- Recomendado para a maioria dos novos trabalhos.

| Nível vCore | Bom para | Armazenamento | Tamanho máximo |
|------------|----------|---------|----------|
| General Purpose | Cargas de trabalho normais, melhor preço | Armazenamento premium remoto | 4 TB |
| Business Critical | Baixa latência, alta IO | Local SSD, réplica legível embutida, In-Memory OLTP | 4 TB |
| Hyperscale | Bancos de dados grandes ou em rápido crescimento; Microsoft **padrão recomendado** para o novo OLTP | Camada de armazenamento separada com cache SSD local | **128 TB** |

**Exemplo:** Um pequeno site de caridade precisa de um banco de dados barato e simples. O padrão S2 (DTU) é bom. O banco de dados de negociação de um banco precisa de baixa latência. Escolha Business Critical (vCore).

## Computação Serverless
As escalas **Serverless** calculam por si mesmas e contas por segundo.
- Disponível em **General Purpose** e **Hyperscale**.
- **Auto-pause** quando ocioso: você paga apenas pelo armazenamento enquanto pausado. Auto-pause é GA em General Purpose, preview em Hyperscale.
- Atraso Auto-pause: **15 min a 7 dias** em General Purpose (mínimo de 60 min em Hyperscale). Defina **-1** para desligar o auto-pause.
- Estes **bloco auto-pause** (auto-escala ainda funciona): geo-replication ou failover groups, LTR, um alias DNS, um banco de dados Data Sync, um elastic jobs job banco de dados.
- A outra opção é a computação **provisioned**: tamanho fixo, faturado por hora.

**Exemplo:** O banco de dados de teste de uma escola é usado apenas em horas de aula. Serverless com um auto-pause de 1 hora economiza dinheiro à noite e fins de semana.

## Elastic pools
An **elastic pool** compartilha a computação entre bancos de dados. Um banco de dados ocupado empresta a capacidade de reposição dos silenciosos.
- Melhor para **muitos bancos de dados com picos irregulares e não sobrepostos**. Caso clássico: um aplicativo SaaS com um banco de dados por cliente.
- Piscinas DTU (eDTUs) ou piscinas vCore (General Purpose, Business Critical, **Hyperscale**). Você paga pela piscina, não por banco de dados.
- Set **min e max por banco de dados** para que um banco de dados barulhento não pode passar fome os outros.
- Redimensionar a piscina de **Configurar**. É principalmente on-line, mas as conexões podem cair brevemente.
- **Hyperscale elastic pools** (GA): até 100 TB por pool e até 4 réplicas de alta disponibilidade. Adicionando um banco de dados não- Hyperscale converte-o em Hyperscale.

**Exemplo:** Uma empresa de software de ginásio tem 200 pequenas bancos de dados de ginásio. Algumas academias estão ocupadas em 6 AM, outras em 7 PM. Uma piscina é muito mais barata do que 200 bancos de dados separados.

## Hyperscale
- A computação e o armazenamento são **Separados**. O armazenamento cresce por conta própria de 10 GB para **128 TB**.
- **Backups e restaura leva minutos**, qualquer que seja o tamanho. Eles usam armazenamento snapshots, não cheio / diff / log backups.
- **Escala rápida** de computação.
- **Read scale-out:** até **4 HA réplicas**, até **30 réplicas nomeadas** (cada um com seu próprio tamanho), além de geo-replicas.
- Enviar leituras para uma réplica com `ApplicationIntent=ReadOnly` no connection string.
- **Voltar para trás:** você pode retornar ao General Purpose somente **dentro de 45 dias** de mover-se para Hyperscale. Para alcançar Business Critical, passe por General Purpose.
- Preços (desde Dec 2023): sem taxa de licença SQL no preço. **Azure Hybrid Benefit não está disponível para novos bancos de dados Hyperscale.**

**Exemplo:** O banco de dados de pedidos de uma loja online chegará a 30 TB em dois anos. A equipe não quer alterar o código do aplicativo. Mova-o para Hyperscale.

## Scaling SQL Database: regras rápidas
- Nível de mudança **Escala para cima/para baixo:** ou vCores / DTUs. Principalmente on-line, reconexão curta no final.
- **Read scale-out:** descarrega relatórios para uma réplica legível (Business Critical, Premium, Hyperscale).
- **Escala para fora:** sharding em muitos bancos de dados (lição 4).
- Durante uma migração, aumente a velocidade e depois diminua depois.

## Implantar SQL Database: métodos
| Método | Use-o quando |
|--------|-------------|
| Azure portal | Configuração manual, one-off |
| PowerShell (`New-AzSqlServer`, `New-AzSqlDatabase`, `New-AzSqlServerFirewallRule`) | Administradores do Windows, scripts |
| Azure CLI (`az sql server create`, `az sql db create`) | Bash, Shell em nuvem, Linux/Mac |
| Modelo ARM ou Bicep | Implementações grandes e repetíveis (**declarativo**: você descreve o estado final) |

PowerShell e CLI são **imperativo** (você lista os passos). Módulo 4 abrange modelos e scripts em profundidade.

```bash
az sql server create -g rg-shop -n shopsrv -l westeurope -u sqladmin -p '<pwd>'
az sql db create -g rg-shop -s shopsrv -n orders --edition GeneralPurpose --compute-model Serverless --family Gen5 --capacity 2
```

## Managed Instance: níveis e opções
| Opção | Factos-chave |
|--------|-----------|
| **General Purpose** | Armazenamento remoto. Cargas de trabalho padrão. |
| **Próxima geração General Purpose** | Upgrade de GP no armazenamento Elastic SAN. Até **500 bancos de dados**, **32 TB**. Extra IOPS e conjunto de memória com um controle deslizante. Mesmo preço base como GP. |
| **Business Critical** | Local SSD, secundário legível incorporado, **In-Memory OLTP**, failover mais rápido. |
| **pools de instâncias** | Muitas pequenas instâncias compartilham um VM. **Somente GP**. Permite instâncias **2-vCore**. Até 40 instâncias e 500 bancos de dados por pool. |
| **Stop/start** | Somente GP. Nenhuma taxa de computação ou licença enquanto parada; o armazenamento ainda é cobrado. Pode ser agendado. |
| **Oferta gratuita** | Uma instância GP gratuita por assinatura por 12 meses (4 vCores, 720 vCore horas por mês, 64 GB). |

Outros fatos MI:
- Cada banco de dados pode usar todo o armazenamento da instância: até **16 TB** (GP e Business Critical na série premium nas principais regiões), **32 TB** na próxima geração GP.
- **99.99% SLA**. Business Critical usa réplicas Always On no interior.
- Vive em um **subnet dedicada** no seu VNet. opcional **Ponto de extremidade público na porta 3342**.
- Escala vCores e armazenamento do portal. Pode demorar um pouco, e termina com um curto failover.

**Política de atualização** (Configurações → Manutenção e atualizações) decide a linha de versão do motor:
| Política | Significado |
|--------|---------|
| **SQL Server 2022** | Pode restaurar bancos de dados de volta para SQL Server 2022; MI link failback com 2022 |
| **SQL Server 2025** | Default para novas instâncias no portal; restaurar e vincular com 2025 |
| **Sempre atualizado** | Características mais recentes; **sem restaurar de volta** a caixa SQL Server |

Você só pode subir: 2022 → 2025 → Sempre atualizado. Nunca para baixo. Verifique com `SELECT SERVERPROPERTY('ProductUpdateType');` → `CU` ou `Continuous`. PowerShell e CLI ainda padrão para SQL Server 2022, então defina o formato do banco de dados em scripts.

**Exemplo:** Uma equipe pode precisar mover um banco de dados de volta para on-premises SQL Server 2022 um dia. Mantenha o MI na política **SQL Server 2022**.

## Backups (versão curta)
- Automático completo (semanalmente), diferencial (a cada 12 horas), log (a cada 5–10 min).
- Point-in-time restore **dias 1–35** (padrão 7). Long-term retention até **10 anos**.
- A restauração sempre cria um banco de dados **Novo**.
- MI também permite manual **COPY_ONLY** backups para Blob Storage.
- Detalhes completos estão no módulo de alta disponibilidade e recuperação de desastres.

## Para lembrar na prova
- **DTU não está disponível no Managed Instance.**
- Serverless = General Purpose e Hyperscale. Auto-pause é bloqueado por geo-replication, failover groups, LTR e características semelhantes.
- Hyperscale: **128 TB**, restaurações de minutos de duração, até **30 réplicas nomeadas**, volta para GP apenas dentro de **45 dias**.
- Use um elastic pool para vários bancos com picos em horários diferentes. Defina limites mínimos e máximos por banco.
- MI **pools de instância** = GP Permitir apenas 2-vCore instâncias.
- MI **stop/start** = GP Apenas.
- MI **Sempre atualizado** policy = no way back to SQL Server 2022/2025 Restaura.
- ARM/Bicep = declarative and repeatable. Portal = not repeatable.

## Teste rápido
1. Um banco de dados de desenvolvimento é ocioso todas as noites e fins de semana. Como você corta custos com menos esforço?
   <details><summary>Resposta</summary>Use o nível serverless com auto-pause. Você paga apenas pelo armazenamento enquanto ele estiver pausado.</details>
2. 150 bancos de dados de inquilinos têm picos curtos em momentos diferentes. O que você implanta?
   <details><summary>Resposta</summary>An elastic pool. Os bancos de dados compartilham computação porque seus picos não se sobrepõem.</details>
3. Mudou uma banco de dados para Hyperscale há 60 dias. Pode voltar para General Purpose?
   <details><summary>Resposta</summary>A migração reversa só funciona dentro de 45 dias.</details>
