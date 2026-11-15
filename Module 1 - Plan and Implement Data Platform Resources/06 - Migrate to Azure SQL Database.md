# Migrar para Azure SQL Database

> **Habilidades cobradas:** Implementar uma estratégia de migração offline, implementar uma estratégia de migração online, implementar uma migração para o Azure, implementar uma migração entre serviços Azure SQL
> **Tempo de leitura:** ~8 min

## Conceito central
Azure SQL Database não pode restaurar um arquivo `.bak` normal. Então você move **schema e dados**, não backups. Offline: use **DMS** ou um **BACPAC**. Online: use **transactional replication**. Apenas algumas tabelas: use **bcp** ou **Azure Data Factory**.

Analogia: você não pode levar a geladeira inteira para um quarto de hotel. Você descompacta a comida e carrega-a em item por item.

## Azure Database Migration Service (DMS)
DMS é o principal motor de migração gerenciado. Para SQL Database é **Apenas offline**: o tempo de inatividade começa quando a cópia começa. Nos bastidores, ele executa pipelines **Azure Data Factory**.

Você pode dirigi-lo a partir do **Azure portal**, **Azure CLI** (`az datamigration`) ou **PowerShell** (`Az.DataMigration`). Os scripts são bons para dutos, muitos bancos de dados e execuções repetidas.

**O que você precisa:**
1. A **DMS** recursos.
2. O **banco de dados alvo** já foi criado.
3. A **Tempo de execução da integração auto-hospedada (SHIR)**: um pequeno agente instalado em uma máquina que pode chegar à fonte. Ele também faz o trabalho de cópia, para dimensioná-lo bem.
4. **Permissões:** Fonte login = **db_datareader + VIEW ANY DEFINITION**. Alvo login = **db_owner**.
5. O **esquema** no alvo. Uso DMS migração de esquema (tick **Migrar o esquema em falta**), ou implantar a DACPAC com SqlPackage Primeiro.

**Passos com Azure CLI:**
```bash
az datamigration sql-service create -g rg --sql-migration-service-name dms1 --location westeurope
az datamigration sql-server-schema --action MigrateSchema \
   --src-sql-connection-str "<source>" --tgt-sql-connection-str "<target>"
az datamigration sql-db create -g rg --sqldb-instance-name tgtsrv --target-db-name HR \
   --source-database-name HR --source-sql-connection ... --target-sql-connection ... \
   --scope "<target server id>" --migration-service "<dms id>"
az datamigration sql-db show -g rg --sqldb-instance-name tgtsrv --target-db-name HR --expand MigrationStatusDetails
```
- Adicionar `--table-list` (PowerShell: `-TableList`).
- PowerShell equivalente: `New-AzDataMigrationToSqlDb`.
- No portal, observe o progresso em **DMS recurso → Monitorar migrações**.

**Fluxo de estado:** Preparação para cópia (desativa autostats, gatilhos, índices no alvo) → Copia → Cópia terminada → Rebuilding indexes → **Sucedido**.

**Dicas de velocidade:**
- **Escala acima** o alvo antes, escalar para baixo depois.
- **Muitas pequenas tabelas** são lentos, porque ADF inicia uma atividade por tabela.
- **Colunas grandes BLOB** pode acabar.
- Até **10 migrações simultâneas de banco de dados por nó SHIR**. Adicione nós para mais.

**Exemplo:** O banco de dados HR de uma empresa de bicicletas é silencioso aos fins de semana. O DBA inicia uma migração offline DMS na sexta-feira à noite, verifica as migrações do Monitor no sábado e muda o aplicativo no domingo.

Observação: o velho **Azure SQL migration extension** (Azure Data Studio, aposentado 28 Feb 2026) e **DMA** (aposentado julho 2025) fez o mesmo job com um assistente. DMA tinha três modos: esquema e dados, esquema apenas, dados apenas. O motor hoje é DMS.

## BACPAC (exportação e importação)
A **BACPAC** é um arquivo compactado com **schema e dados**.
1. **Exportação** da fonte com SSMS ou **SqlPackage**.
2. Carregar para **Azure Blob Storage**.
3. **Importação** com o **portal** (servidor lógico → **Importar banco de dados**) ou **SqlPackage**.

```bash
SqlPackage /a:Import /sf:HR.bacpac \
  /tcs:"Data Source=tgtsrv.database.windows.net;Initial Catalog=HR;User Id=admin;Password=<pwd>" \
  /p:DatabaseEdition=Premium /p:DatabaseServiceObjective=P6
```

Limites:
- Somente **Offline**.
- **Não logins, Agente jobs ou servidores ligados** (apenas objetos de nível de banco de dados).
- Bancos de dados **mais de 150 GB** muitas vezes falham no portal → usar **SqlPackage**.
- A importação do Portal cria apenas um **novo banco de dados único** (não em uma piscina). O armazenamento não pode ficar atrás de um firewall.
- Para **Managed Instance**, importe com **SqlPackage** ou SSMS, não o portal.
- Assista ao progresso em **Histórico de importação/exportação**.

**Exemplo:** Um banco de dados de membros do clube de 20 GB move-se ao longo de um fim de semana. Exporte um BACPAC, carregue-o, importe-o em P6 e depois diminua a escala.

## Transactional replication (online)
A única maneira **online** em SQL Database. A fonte continua trabalhando enquanto as mudanças fluem através.

| Papel | Quem |
|------|-----|
| **Editora** | Fonte SQL Server (2016+) ou Managed Instance |
| **Distribuidor** | Move as mudanças. Muitas vezes no editor. |
| **Assinante** | Azure SQL Database (**somente assinatura push**) |
| **Artigo** | Uma tabela |
| **Publicação** | Um conjunto de artigos |

Como funciona: um **Agente Snapshot** copia os dados iniciais. O **Agente Leitor de Log** lê as alterações do log. O **Agente de Distribuição** os empurra para o Azure.

Configurar com `sp_adddistributor`, `sp_addpublication`, `sp_addarticle`, `sp_addsubscription`, `sp_addpushsubscription_agent`.

Regras:
- Toda tabela precisa de um **chave primária**.
- SQL Database só pode ser um **assinante push** de replicação **transacional ou snapshot**.
- Conectar-se com **SQL authentication** (Entra auth também é possível em alguns casos).
- Monitor em **SSMS** no editor, **E** no Azure portal.

**Cutover:** stop app escreve → verificar todas as alterações chegou → apontar o aplicativo para Azure → remover replicação.

**Exemplo:** Um sistema de inventário de armazém roda 24/7. Replicação mantém o Azure sincronizado por uma semana. Cutover leva cinco minutos em uma manhã de domingo.

## Mover apenas alguns dados
| Necessidade | Ferramenta |
|------|------|
| Cópia rápida de uma única vez de uma tabela | **bcp** (cópia em volume; tabela alvo deve existir) |
| Cargas filtradas, programadas ou transformadas | **Azure Data Factory** (precisa de um SHIR para on-premises) |

```bash
bcp Shop.dbo.Product out product.dat -S onpremsrv -T -n
bcp Shop.dbo.Product in product.dat -S tgtsrv.database.windows.net -U admin -P <pwd> -n
```

**Exemplo:** Uma empresa mantém dados de vendas sensíveis on-premises. Ele move apenas as tabelas de clientes e produtos para o Azure com Data Factory.

## Entre serviços Azure SQL
- **SQL Database → SQL Database ou MI:** BACPAC exportação / importação (SqlPackage para MI).
- **MI → SQL Database:** MI pode ser um **editor** para transactional replication, então você pode se mover online. Ou use BACPAC offline.
- **Tabelas selecionadas, em qualquer direção:** Azure Data Factory ou bcp.
- Banco de dados **MI → MI:** **copiar/mover** (lição 7).

## Limites DMS conhecidos para SQL Database
- **Colunas computadas** não são migrados.
- Colunas com padrões podem obter o **padrão em vez de NULL**.
- **Tabelas vazias são ignoradas.**
- **caracteres de bytes duplos** em nomes de tabela não são suportados. Renomear, migrar, renomear de volta.
- **Desativar CDC na fonte** antes da migração do esquema.

## Para lembrar na prova
- DMS E- SQL Database = **Apenas offline**. Online = **transactional replication**.
- Script muitas migrações → `az datamigration sql-db create` ou `New-AzDataMigrationToSqlDb`.
- Apenas algumas tabelas em DMS → `--table-list` / `-TableList`.
- "Algumas tabelas não migraram" → eram **vazio**.
- BACPAC sobre 150 GB → **SqlPackage**, não o portal.
- BACPAC nunca se move logins ou jobs.
- Replicação a SQL Database = **push** As tabelas precisam de uma **chave primária**, monitor em **SSMS**.

## Teste rápido
1. Você deve migrar 40 bancos de dados para SQL Database a partir de um pipeline CI / CD. O que você usa?
   <details><summary>Resposta</summary>Azure DMS conduzido por Azure CLI (`az datamigration`) ou PowerShell (`New-AzDataMigrationToSqlDb`).</details>
2. A importação do portal BACPAC de um banco de dados 300 GB continua falhando. O que você faz?
   <details><summary>Resposta</summary>Importação com SqlPackage. O portal muitas vezes falha mais de 150 GB.</details>
3. A configuração de replicação falha em uma tabela. Qual é a causa provável?
   <details><summary>Resposta</summary>A tabela não tem chave primária. Transactional replication precisa de uma.</details>
