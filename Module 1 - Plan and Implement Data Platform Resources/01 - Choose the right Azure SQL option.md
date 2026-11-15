# Escolha a opção Azure SQL certa

> **Habilidades cobradas:** Recomendar uma oferta de banco de dados com base em requisitos específicos, Identificar casos de uso para os serviços SQL habilitados para Azure Arc, Identificar casos de uso de Azure SQL Database em Microsoft Fabric
> **Tempo de leitura:** ~10 min

## Conceito central
O Azure tem três casas principais para dados SQL Server. **SQL Server em Azure VM** dá-lhe controle total. **Azure SQL Managed Instance** (MI) é quase completo SQL Server, mas a Microsoft o executa. **Azure SQL Database** é um único banco de dados em nuvem com menos trabalho de administração. Dois extras ficam ao lado deles: **Arco Azure** (gerenciar SQL Server fora do Azure) e **SQL database em Microsoft Fabric** (um banco de dados construído para análise).

Analogia: sua própria casa (VM), um apartamento com serviço (MI), um quarto de hotel (SQL Database). Quanto mais serviço você recebe, menos você pode mudar.

## Quem gerencia o que
| Camada | SQL Database (PaaS) | Managed Instance (PaaS) | SQL Server em VM (IaaS) |
|-------|--------------------|------------------------|------------------------|
| Hardware | Microsoft | Microsoft | Microsoft |
| OS e patching | Microsoft | Microsoft | **Você** |
| Configurações SQL Server | Parcial | extensa | **Completo** |
| Rede e armazenamento | Não | Parcial (sub-rede própria) | **Completo** |
| Backups e HA | Construído em | Construído em | Você os criou |

**PaaS** (Plataforma como Serviço) significa que a Microsoft executa o servidor para você. **IaaS** (Infraestrutura como Serviço) significa que o Azure lhe dá uma máquina virtual e você corre tudo dentro dela.

## SQL Server em Azure VM (IaaS)
Funciona como SQL Server no seu próprio servidor. Escolha-o quando precisar:
- **Versões antigas SQL Server** que um fornecedor ainda precisa.
- **SSIS, SSAS, SSRS** (Integração, Análise, Reporting Services) na mesma máquina.
- **acesso OS** para instalar agentes, scripts ou outros aplicativos.
- PaaS não tem: **FILESTREAM, FileTable, PolyBase, transações entre instâncias**.
- Um **Elevação e Mudança** simples (mover-se como-é) sem alterações.

As imagens do Marketplace hoje cobrem SQL Server **2016 a 2025**. As imagens 2012/2014 desapareceram; instale-as à mão.

**Exemplo:** Uma fábrica executa um aplicativo SQL Server 2016 antigo mais relatórios SSRS em uma caixa. O fornecedor não suportará nada mais novo. Escolha um SQL Server VM.

## Azure SQL Managed Instance (PaaS)
Um SQL Server **instância** completo executado pela Microsoft. É perto de 100% compatível com on-premises SQL Server. Tem coisas que o SQL Database não tem:
- **SQL Agent** jobs, **servidores ligados**, **CLR**, **Corretor de Serviço**.
- **Consultas entre bancos de dados** dentro da instância.
- Bancos de dados do sistema (msdb e outros).
- **Restaurar nativo.bak backups** de Azure Blob Storage.
- Muitas bancos de dados por instância (até 100; 500 em Next-gen General Purpose).
- Vive em seu próprio **rede virtual** (privado por padrão).

**Exemplo:** A banco de dados de catálogos de um retalhista de vestuário utiliza SQL Agent jobs e junções de bancos de dados cruzadas. Eles não querem alterações de código nem patches. Escolha MI.

## Azure SQL Database (PaaS)
Um banco de dados (ou um conjunto deles) com o mínimo trabalho de administração. Melhor para **novos aplicativos em nuvem** e **carga desconhecida ou spiky**. Características que você só chega aqui:
- **Hyperscale** até 128 TB por banco de dados.
- **Serverless** auto-escala e auto-pause.
- **Elastic jobs** e **elastic query** entre bancos de dados.
- **Automatic tuning** que pode criar e soltar índices.

**Exemplo:** Uma startup constrói um novo aplicativo de reserva. Não sabe o quão ocupado será. Escolha SQL Database, comece pequeno, escala mais tarde.

## Compare em um relance
| Necessidade | Melhor escolha |
|------|-------------|
| Elevador e mudança, sem alterações, acesso OS | SQL Server em VM |
| Versão antiga SQL, SSIS / SSRS / SSAS na mesma caixa | SQL Server em VM |
| FILESTREAM, FileTable, PolyBase | SQL Server em VM |
| SQL Agent, servidores vinculados, CLR, consultas de cross-db, poucas alterações de código | Managed Instance |
| Restaurar arquivos.bak nativos do on-premises | Managed Instance (ou VM) |
| Novo aplicativo nativo da nuvem, menos admin | SQL Database |
| Muitas pequenas bancos de dados com carga irregular | SQL Database elastic pool |
| Banco de dados sobre 4 TB, poucas alterações no aplicativo | SQL Database Hyperscale |
| Big data warehouse, varreduras enormes (MPP) | Azure Synapse Analytics |

Esforço de migração para um aplicativo antigo: SQL Database = high, MI = low, VM = very low.

**Azure Synapse Analytics** é apenas para grandes análises. É ruim para muitas pequenas leituras e escritos.

## Azure Arc-enabled SQL Server
**Arco Azure** coloca Servidores SQL que correm fora do Azure (seu data center, outras nuvens, a borda) no Azure portal. O servidor permanece onde está.

Casos de uso:
- **Um inventário** de todos os servidores SQL em todos os lugares.
- **Avaliações de melhores práticas** e **varreduras de segurança** com Microsoft Defender for Cloud.
- **Governação** com Azure Policy e Microsoft Purview.
- **Patching** através de atualizações automáticas Azure Update Manager ou Arc.
- **Licenciamento Pay-as-you-go** faturado através do Azure.
- **Atualizações de Segurança Estendidas (ESU)** para versões SQL após o fim do suporte.
- **Avaliação de migração e migração para MI** do portal (lição 7).
- Automated backups (preview) e Microsoft Entra authentication.

**Exemplo:** Uma empresa tem Servidores SQL em 20 fábricas remotas. Quer um lugar para ver versões, procurar configurações fracas e corrigi-las. Não quer movê-las. Arc-habilitá-las.

## SQL database em Microsoft Fabric
O exame diz "Azure SQL Database em Microsoft Fabric". Os documentos dizem "SQL database in Microsoft Fabric". A mesma coisa.

É um banco de dados **OLTP** (transações do dia-a-dia) que vive **dentro de um espaço de trabalho Tecido**. É SaaS, não um recurso Azure. Ele usa o mesmo motor que Azure SQL Database. É **geralmente disponível desde Nov 2025**.

Sua principal característica: cada tabela é **espelhado automaticamente** para **OneLake** em tempo quase real, como arquivos Delta (Parquet). Um read-only **Ponto de extremidade da análise SQL** é criado para você. Power BI e Spark podem ler os dados com **ETL** (sem cópia jobs). A espelhamento não pode ser desligada.

Limites principais em comparação com Azure SQL Database:
- **Somente Microsoft Entra authentication.** No SQL logins.
- Max **4 TB**, auto-escala até **32 vCores**.
- **Não** geo-replication, failover groups, geo-restore ou **LTR** (retenção de longo prazo backup).
- Backups: point-in-time restore 7 dias por padrão, até 35.
- **Não** Always Encrypted, livro-razão, In-Memory OLTP ou CDC.
- **Não** elastic pools (um tecido **capacidade** desempenha esse papel) e **Não** SQL Agent ou elastic jobs. Use **oleodutos Data Factory** para agendar o trabalho.
- **RLS, dynamic data masking e permissões de objeto não são copiadas** para o espelho OneLake.
- Implantar com o portal de tecido, REST API ou Tecido CLI. Não ARM, Bicep, Az PowerShell ou Azure CLI.

Use-o para:
- **Apps translíticos** (transações mais análises quase em tempo real sobre os mesmos dados).
- **ETL reverso** (empurre os dados da casa do lago de volta para um banco de dados de aplicativos).
- An **armazenamento de dados operacionais** ou **Aplicações AI / RAG** perto de dados de tecido.

Se você precisar apenas de um Azure SQL Database existente copiado para o OneLake, use o **tecido espelhamento**. No MI, o espelhamento precisa da política de atualização do **SQL Server 2025 ou sempre atualizado**.

**Exemplo:** Uma loja já usa Tecido e Power BI. Ele quer que novas ordens apareçam nos painéis de controle em segundos, sem ETL jobs. Escolha SQL database no tecido.

## Para lembrar na prova
- "Mudanças mínimas de código + SQL Agent / servidores vinculados / CLR " → **Managed Instance**.
- "Necessidade de acesso OS, FILESTREAM ou uma versão antiga" → **SQL Server em VM**.
- "Novo aplicativo de nuvem, menos admin, carga imprevisível" → **SQL Database** (serverless).
- Elastic jobs, elastic query e Hyperscale são **Somente SQL Database**.
- "Gerenciar servidores on-premises e multi-nuvem SQL do Azure" → **Arco Azure**.
- "Dados OLTP em Power BI quase em tempo real sem ETL" → **SQL database in Microsoft Fabric** (ou espelhamento de tecido).
- SQL database: **Somente intra**, sem geo- DR, sem LTR, sem Always Encrypted. Se a pergunta precisar de qualquer um desses, escolha Azure SQL Database.
- Não confunda o tecido SQL database (ler/escrever) com o seu **Ponto de extremidade da análise SQL** (read-only).

## Teste rápido
1. Um aplicativo usa SQL Agent jobs e servidores vinculados. A equipe quer que a Microsoft lide com patches. Qual opção?
   <details><summary>Resposta</summary>Azure SQL Managed Instance. Ele suporta Agente e servidores vinculados e é PaaS.</details>
2. Uma equipe deve usar SQL authentication e geo-restore. Eles podem usar SQL database in Microsoft Fabric?
   <details><summary>Resposta</summary>Não. Tecido SQL database é apenas intra e não tem geo-restore. Use Azure SQL Database.</details>
3. Você tem 300 servidores on-premises SQL e quer um inventário e uma visão de segurança no Azure sem movê-los. O que você usa?
   <details><summary>Resposta</summary>Azure Arc-enabled SQL Server. Traz servidores externos para o gerenciamento do Azure.</details>
