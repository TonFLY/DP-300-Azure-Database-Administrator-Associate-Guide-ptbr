# SQL Server on Azure VMs, híbrido e patching

> **Habilidades cobradas:** Implantar ofertas de banco de dados em plataformas selecionadas, Configurar SQL Server on Azure Virtual Machines para escala e desempenho, Implantar soluções híbridas SQL Server, Identificar casos de uso para serviços SQL habilitados para Azure Arc, Aplicar patches e atualizações para implantações híbridas e IaaS
> **Tempo de leitura:** ~11 min

## Conceito central
A SQL Server VM é o seu próprio servidor no Azure. Microsoft possui o hardware. Você possui o OS, SQL Server, discos, rede e **patching**Híbrido significa alguns SQL Servidores ficam on-premises e trabalhar com o Azure para backup, DR (recuperação de desastres) ou gestão.

Analogia: alugar um apartamento vazio. O senhorio conserta o edifício. Você compra os móveis, limpa e conserta tudo dentro.

## Implantando um SQL Server VM
- Use uma imagem **Azure Marketplace**, por exemplo SQL Server 2022 em Windows Server 2022. SQL já está instalado.
- Durante a instalação você escolhe SQL authentication, porta, layout de armazenamento e **Conectividade SQL**:
  - **Público:** acessível a partir da internet.
  - **Privado:** apenas dentro da rede virtual.
  - **Local:** apenas a partir do mesmo VM.
- A configuração do Portal não é fácil de repetir. Use modelos ARM / Bicep para muitos VMs.

**Opções de licenciamento:**
| Opção | Significado |
|--------|---------|
| Pay-as-you-go | Licença SQL incluída no preço VM, faturado por segundo |
| Azure Hybrid Benefit / BYOL | Reutilize suas próprias licenças SQL Server com **Software Assurance** |
| Licença HA/DR | Réplicas **passiva** grátis para HA/DR. O tipo de licença de mudança está online, sem reinicialização |
| Instâncias reservadas | Compromete 1–3 anos para economizar no VM |

## SQL Server IaaS Agent extension
O **SQL IaaS Agent extension** liga o VM a um recurso de "máquina virtual SQL" no portal. Ele lhe dá:
- Automated backup e (legado) Automated Patching.
- Integração **Azure Key Vault** e integração Microsoft **Defender for Cloud**.
- Visualização de uso do disco, alterações de licença e edição, **Avaliação das melhores práticas**.

Factos actuais:
- **Não há mais modos de gestão** (leve / cheio foram removidos em Mar 2023). O registro básico é gratuito.
- O agente instala apenas quando você liga um recurso que precisa dele, com direitos de menos privilégio.
- Auto-instalado SQL Server 2016 + VMs são **registrado automaticamente**.

## Tamanho VM para SQL
| família VM | Utilização |
|-----------|-----|
| General purpose | Dev/test, pequenas bancos de dados |
| Computação otimizada | CPU alto, servidores de aplicativos |
| **Memory optimized** | **A maioria das cargas de trabalho SQL.** Ebdsv5 oferece o melhor custo-benefício. Use a série M quando precisar de muita memória |
| Armazenamento otimizado | Discos locais rápidos para dados scale-out |
| GPU / HPC | ML, renderização, computação especial |

Se um VM é muito pequeno, **redimensionar** para um tamanho maior. Isso precisa de um reinício.

## Discos e armazenamento
| Disco | Utilização | Max IOPS | Max MB / s |
|------|-----|----------|----------|
| Disco Ultra | IO mais exigente | 400,000 | 10,000 |
| Premium SSD v2 | Melhor preço-desempenho para a maioria SQL | 80,000 | 2,000 |
| Premium SSD | Produção geral SQL | 20,000 | 900 |
| Padrão SSD | Cargas de trabalho leves | 6,000 | 750 |
| Padrão HDD | Backups, não crítico | 2,000 | 500 |

Regras:
- **C:** = OS disco. **D:** = temporary disk; data is **perdido na reinicialização**. Bom para **temdb** Apenas.
- Coloque arquivos **dados** e **log** em **discos de dados separados**.
- **Leia o cache** em discos de dados. **Sem cache** em discos de log.
- Desligue vários discos (Espaços de Armazenamento) para mais IOPS.
- Use Premium SSD, Premium SSD v2 ou Ultra para dados e log. O armazenamento padrão é bom para backups.

Compressão **Configurações de SQL em um VM:** backup, **Inicialização instantânea do arquivo**, bancos de dados do sistema movidos para fora C:, autoshrink e autoclose **fora**, autocrescimento sensato, conjunto **memória do servidor max**, **Bloquear páginas na memória**, **otimizar para cargas de trabalho ad hoc**, **Query Store** em, DBCC CHECKDB regular e manutenção de índice / estatística.

A criptografia do lado do servidor **Criptografia em repouso:** está ativada por padrão. Encriptação de disco do Azure (BitLocker dentro do VM) **Aposenta 15 Sep 2028**. Use **criptografia no host** para o novo VMs.

**Exemplo:** Uma folha de pagamento VM tem gravações lentas no final do mês. O arquivo de log compartilha um disco em cache com dados. Mova o registro para seu próprio disco Premium SSD v2 sem cache.

## HA em VMs (versão curta)
- **Availability zones** (~99.99%) espalham VMs sobre centros de dados. **Conjuntos de disponibilidade** (~99.95%) espalha-os sobre racks.
- **Always On availability groups** (nível de banco de dados, até 9 réplicas) e **failover cluster instances** (nível de instância, armazenamento compartilhado).
- Detalhes completos estão no módulo de alta disponibilidade e recuperação de desastres.

## Configurações híbridas SQL Server
Hybrid significa que on-premises e Azure trabalham juntos. Configurações comuns:
1. **DR no Azure:** on-premises SQL executa o trabalho diário. Um Azure VM é o standby (por exemplo, uma réplica assíncrona AG). Você obtém um segundo site sem construir um segundo data center.
2. **Backups para Azure:** `BACKUP TO URL` envia backups para Blob Storage. Você pode restaurá-los em uma Azure VM em uma broca ou um desastre real.
3. **Arquivos de dados em Azure Storage:** SQL roda on-premises, mas os arquivos de banco de dados ficam em Azure Blob Storage.
4. **Link MI:** replicar um banco de dados on-premises para Managed Instance para DR ou ler descarga (lição 7).
5. O **Arco Azure:** gerencia, avalia e corrige Servidores on-premises SQL do Azure.

Redes para híbridos:
- **Site-to-site VPN:** mais barato, passa pela internet, maior latência.
- Linha privada **ExpressRoute:**, baixa latência, mais cara.
- **Ponto-a-local VPN:** apenas uma máquina, bom para administração ou teste.
- Active Directory e DNS devem trabalhar em ambos os lugares.

**Exemplo:** Um banco mantém seu SQL Server principal em seu próprio data center para conformidade. Ele adiciona uma réplica AG assíncrona em uma Azure VM sobre ExpressRoute para DR, e Arc habilita o servidor on-premises para uma visualização no portal.

## Quem corrige o que
| Plataforma | Quem remendos | Os seus controlos |
|----------|-------------|---------------|
| SQL Server em Azure VM | **Você** | Azure Update Manager, legado Automated Patching, manual |
| SQL Server Arc-enabled | **Você** | Atualizações automáticas, Azure Update Manager |
| Managed Instance | Microsoft | Janela de manutenção, política de atualização |
| SQL Database | Microsoft | Janela de manutenção |

## Automated Patching (legado, aposentadoria)
- Parte do SQL IaaS Agent extension.
- Define uma janela: dia, hora de início, duração **minutos 30–180**.
- Instala apenas atualizações do Windows e SQL marcadas como **Importante ou Crítico**. **Sem atualizações cumulativas (CUs).**
- **Não AG-consciente.** Dê a cada réplica um dia diferente, ou ambos podem reiniciar juntos.
- **Aposenta-se 17 Sep 2027.** Não o use para o novo VMs.

## Azure Update Manager (a substituição)
- Um serviço para **Servidores Azure VMs e Arc**, **Windows e Linux**.
- Não precisa de Log Analytics ou Azure Automation.
- **Instala SQL Server CUs** (Automated Patching não pode).
- **Avaliação periódica** a cada 24 horas mostra atualizações ausentes.
- **Atualizações únicas** sob demanda, ou **Atualizações programadas** com um **configuração de manutenção**. Suporta escopo dinâmico e Azure Policy.
- **Hotpapping** no Windows Server Azure Edition VMs suportado (sem reinicialização para muitas correções).

Para mudar um SQL VM para o Update Manager:
1. Desativar Automated Patching.
2. Ative o **Habilitar atualizações da Microsoft** (necessário para obter atualizações do SQL Server, não apenas o Windows).
3. Defina a orquestração do patch para **Agendas Gerenciadas pelo Cliente** e inclua a classificação **Atualizações** para obter CUs.

Ou execute o script oficial: `Convert-SQLVMPatchingSchedule` (módulo MigrateSQLVMPatchingSchedule-Module). Ele faz tudo isso por você.

Nunca use ferramentas **Ambos** em um VM. O Gerenciador de Atualização é **também não AG -aware**, então coloque réplicas em diferentes configurações de manutenção.

## Patching Arc-enabled SQL Server
- **Atualizações automáticas Arc:** programação semanal, Importante / Crítica apenas, sem CUs, **Apenas janelas**, aplica-se a todas as instâncias no host.
- Ele precisa do tipo de licença **Pay-as-you-go ou Software Assurance**. Está acinzentado para o **Licença somente**.
- Ou use **Azure Update Manager** para servidores Arc (mesmo que Azure VMs).

## Remendo de rolamento para um availability group
Objetivo: apenas uma interrupção curta (um manual failover).
1. Defina todas as réplicas síncronas para **manual failover**.
2. Patch **assíncronas**, em seguida, secundários de sincronização remota e, em seguida, sedários de sincronização locais.
3. Verifique se o alvo é **SYNCHRONIZED**, depois **falhar manualmente** para um secundário de sincronização corrigido.
4. Patch o **velho primário**.
5. Ligue o automatic failover de volta, se quiser, volte a falhar.

```sql
ALTER AVAILABILITY GROUP [AG1] MODIFY REPLICA ON 'SQLVM02' WITH (FAILOVER_MODE = MANUAL);
-- after patching the secondary, run on it:
ALTER AVAILABILITY GROUP [AG1] FAILOVER;
```

- **Nunca corrija o primário primeiro.** Uma primária mais recente não pode enviar log para secundários mais antigos.
- **A atualização do Cluster-Aware não é suportada** para AGs.
- **FCI:** remendo nós passivos, fail over, em seguida, patch o nó ativo antigo.

## Janela de manutenção PaaS
Microsoft patches SQL Database e MI. Você escolhe **Quando**:
| Opção | Tempo |
|--------|------|
| Padrão do sistema | Grandes atualizações evitadas 8 AM –5 PM hora local |
| Semana | 10 PM–6 AM, Mon-Thu |
| Fim-de-semana | 10 PM–6 AM, Fri-Sol |

- Grátis. Não para piscinas de instância SQL Database Basic, S0, S1 ou MI.
- **Notificações antecipadas** 24 h antes (SQL Database precisa de uma janela não padrão para isso).
- Ligado MI, mudando a janela **altera o endereço IP**. Verifique as regras de firewall.
- Geo-replicas não em regiões pareadas devem usar janelas diferentes.

## Para lembrar na prova
- "Instalar SQL Server **CUs** em um cronograma" → **Azure Update Manager**.
- Automated Patching = Important/Critical only, no CUs, aposentados **17 Sep 2027**.
- "Ambos os nós AG reiniciaram juntos" → sobrepondo horários. Nenhuma ferramenta de patch é AG -consciente.
- "Menos esforço para sair do Automated Patching" → `Convert-SQLVMPatchingSchedule`.
- As atualizações automáticas da Arc acinzentadas do tipo de licença → são **Licença somente**.
- Temporário D: unidade é apagada na reinicialização. Logs em discos não cacheados, dados em discos lidos.
- "Pare patches durante o horário comercial em SQL Database ou MI " → **janela de manutenção**.

## Teste rápido
1. Você deve instalar atualizações cumulativas SQL Server no 50 SQL VMs todos os meses. Qual ferramenta?
   <details><summary>Resposta</summary>Azure Update Manager. Automated Patching não instala CUs.</details>
2. As atualizações automáticas do Arc estão acinzentadas para um servidor. Por quê?
   <details><summary>Resposta</summary>Seu tipo de licença é apenas Licença. Mude-o para pay-as-you-go ou Software Assurance.</details>
3. Em uma atualização de rolamento AG, qual réplica você corrige por último?
   <details><summary>Resposta</summary>O primário, depois de um manual failover para um secundário síncrono corrigido.</details>
