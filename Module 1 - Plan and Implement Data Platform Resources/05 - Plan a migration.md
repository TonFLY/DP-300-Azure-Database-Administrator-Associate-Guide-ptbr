# Planear uma migração

> **Habilidades cobradas:** Avaliar os requisitos para uma migração, Avaliar estratégias de migração offline ou online
> **Tempo de leitura:** ~8 min

## Conceito central
Uma boa migração é principalmente planejamento. Você encontra o que tem, verifica o que vai quebrar, escolhe um alvo e decide quanto tempo de inatividade a empresa pode aceitar. Essa última escolha decide **online** ou **offline**, e isso decide a ferramenta.

Analogia: casa em movimento. Primeiro liste seus móveis. Verifique o que se encaixa no novo lugar. Em seguida, escolha: mova tudo em um fim de semana (offline), ou mova sala por sala enquanto ainda mora lá (online).

## As cinco etapas
| Fase | O que você faz |
|-------|-------------|
| **1. Descobrir** | Encontre cada SQL Server, versão, banco de dados, tamanho e os aplicativos que os usam. Obtenha suporte de gerenciamento. |
| **2. Avaliar** | Encontre **bloqueadores** (características não suportadas), tamanho e crescimento. Decida o que se fundir ou se aposentar. |
| **3. Plano** | Escolha o alvo, o tipo de migração, online ou offline, a ferramenta e a ordem. |
| **4. Transforme e otimize** | Corrigir código incompatível, atualização, tamanho certo. Exemplo: mova pacotes SSIS para Azure Data Factory. |
| **5. Migre, valide, corrija** | Execute o movimento, teste dados e aplicativos, corrija problemas, teste DR e monitoramento. |

Dicas: comece com bancos de dados **Não-crítico**. Sempre faça um **migração teste** primeiro. Tenha um **plano rollback**.

**Exemplo:** Um varejista comprou três empresas. Tem 40 servidores SQL em cinco versões. Ninguém conhece todos os aplicativos. Descubra e avalie primeiro, antes de qualquer ferramenta ser escolhida.

## Tipos de migração (os "R")
- Elevador **Rehost:** e mudar para um VM. Sem alterações.
- Pequenas alterações **Refactor:** para se adequar PaaS (por exemplo, MI).
- O **Rearchitect:** altera o aplicativo para usar recursos de nuvem.
- **Reconstruir / substituir / aposentar:** grande mudança, ou remover o aplicativo.

## O que avaliar
- **Características utilizadas:** SQL Agent, CLR, servidores ligados, FILESTREAM, consultas de banco de dados cruzadas, Service Broker. Estes decidem SQL Database vs MI vs VM.
- **Tamanho e crescimento:** sobre 4 TB → Hyperscale ou MI / VM.
- **Tempo de inatividade permitido:** minutos, horas ou nenhum.
- **Versão e edição:** algumas ferramentas precisam de SQL Server 2016+ ou Enterprise / Standard.
- **Rede:** VPN ou ExpressRoute? A fonte pode alcançar o Azure? Largura de banda para grandes backups?
- **Objetos em nível de servidor:** logins, Agente jobs, servidores vinculados. Ferramentas de nível de banco de dados não os movem.
- Dados **Segurança e conformidade:** que devem permanecer on-premises.
- **HA/DR e necessidades de desempenho** após a mudança.

## Ferramentas de avaliação
- **Azure Migrate:** descobrir e avaliar servidores e bancos de dados, com dimensionamento alvo.
- **Avaliação de migração SQL Server no Azure Arc:** para servidores habilitados para Arc. Funciona automaticamente todos os fins de semana. Mostra bloqueadores e um MI de tamanho certo.
- **Componente de migração SSMS:** para upgrades de versão.
- Observação: **Data Migration Assistant (DMA)** se aposentou em julho de 2025. Ele fez verificações de compatibilidade e pequenos movimentos de esquema / dados. Perguntas antigas do exame ainda podem nomeá-lo.
- Observação: **Azure Data Studio** e seu **Azure SQL migration extension** se aposentou em **28 Fev 2026**. Ele dirigiu DMS de um assistente. Seu jobs agora é executado a partir do **Portal DMS, PowerShell ou CLI**, **Migração de arco** ou SSMS.
- Observação: **Assistente de Experimentação de Banco de Dados** aposentado 15 Dec 2024. **DMS (clássico)** SQL cenários aposentados 15 Mar 2026. Use o atual DMS.

## Online vs offline
| | Offline | Online |
|-|---------|--------|
| Fonte durante a cópia | Parado para escrever | Continua a trabalhar |
| Tempo de inatividade | cópia **inteiro** | Somente o **cutover** |
| Complexidade | Baixa | Mais alto |
| Bom para | Pequenas bancos de dados, janelas de fim de semana | sistemas 24/7 |

**Cutover** é o momento em que você interrompe as gravações na origem, espera as últimas alterações chegarem e aponta os aplicativos para o novo banco.

**Exemplo:** An HR banco de dados é silencioso nos fins de semana → off-line, sexta-feira à noite até segunda-feira. O sistema de pacientes de um hospital funciona 24/7 → online.

## ferramenta escooker
| Alvo | Ferramentas offline | Ferramentas online |
|--------|---------------|--------------|
| **SQL Database** | **DMS** (apenas offline), BACPAC / SqlPackage, bcp, Azure Data Factory | **Transactional replication** |
| **Managed Instance** | Nativo backup a URL + restaurar, modo autocomplete LRS, BACPAC via SqlPackage | **link MI**, **LRS contínuo**, **Migração de arco** (usa link ou LRS), DMS, transactional replication |
| **SQL Server em VM** | Backup /restore, destac/attach, BACPAC, bcp, upload VHD, Azure Import/Export | **AG distribuído**, Always On AG, log shipping, transactional replication, DMS online, Azure Migrate |

Armadilhas-chave:
- **DMS para SQL Database está offline apenas.** Online para SQL Database = transactional replication.
- **O link LRS e MI são apenas MI.**
- BACPAC está sempre offline e não se move logins ou jobs.
- Dados parciais (apenas algumas tabelas) → **bcp** ou **Azure Data Factory**.
- Migração de servidor → **Azure Migrate** (OS + SQL).

## Dicas de velocidade para qualquer migração
- Fix IO gargalos na fonte.
- **Escalar o alvo** durante a cópia (por exemplo, Business Critical ou Hyperscale), **escala para baixo** depois.
- Certifique-se de que a largura de banda da rede seja suficiente.
- Mantenha os arquivos (BACPAC, backups) no **mesma região** como o alvo.
- Use **comprimido** backups e **CHECKSUM**.
- Desligue as estatísticas de auto-criação e atualização automática durante a carga e, em seguida, ligue-as novamente.
- Os aplicativos devem usar o **retry lógica** com esperas crescentes (5 s, 10 s, 20 s, até 60 s) e uma nova conexão após uma falha.

## Para lembrar na prova
- Etapas: **Descubra → Avaliar → Plano → Transformar/Otimizar → Migrar/Validar/Remediar**.
- "Zero ou tempo de inatividade mínimo" → ferramenta online. "Semana janela OK " → ferramenta offline.
- "Online para Azure SQL Database" → **transactional replication** (não DMS).
- "Online para MI " → **link MI** ou **LRS contínuo**.
- "Online para VM " → **distribuído AG** ou DMS online.
- DMA (Julho 2025) e Azure Data Studio (28 Fev 2026) são **aposentado**. Saiba o que eles fizeram, escolha as ferramentas atuais.
- Ferramentas de nível de banco de dados nunca movem **logins, Agente jobs, servidores vinculados**. Script-los.

## Teste rápido
1. O sistema de estoque A 24/7 deve se mover para o Azure SQL Database com apenas alguns minutos de inatividade. Qual método?
   <details><summary>Resposta</summary>Transactional replication. DMS para SQL Database está offline apenas.</details>
2. Em que fase você acha que um banco de dados usa FILESTREAM?
   <details><summary>Resposta</summary>É um bloqueador para MI e SQL Database, então o alvo se torna um VM.</details>
3. Uma pergunta antiga diz "use DMA para avaliar". O que você usa hoje?
   <details><summary>Resposta</summary>Azure Migrate ou a avaliação de migração do Azure Arc. DMA se aposentou em julho de 2025.</details>
