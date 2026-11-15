# DP-300: Administering Microsoft Azure SQL Solutions — Banco de questões para prática

**230 questões**, com respostas verificadas com base na documentação do Microsoft Learn. Todas seguem o guia atual da DP-300.

**Como ler cada questão**
- **✅ Resposta** indica a alternativa correta. **Explicação** apresenta o motivo.
- **Dumps:** ✔ indica que os dumps do exame concordam com a resposta. ⚠️ indica divergência em alguns dumps. Nesse caso, siga a alternativa marcada com ✅ neste arquivo.
- **Confiança:** *Alta* indica confirmação na documentação da Microsoft. *Média* indica que a resposta tem respaldo, mas o enunciado admite outra leitura. *Baixa* indica divergência, com as duas interpretações na explicação.
- Azure AD = Microsoft Entra ID.

## Sumário
- [D1 — Planejar e implementar recursos da plataforma de dados (15–20%)](#d1--plan-and-implement-data-platform-resources-1520) — 65 questões
- [D2 — Implementar um ambiente seguro (20–25%)](#d2--implement-a-secure-environment-2025) — 65 questões
- [D3 — Monitorar, configurar e otimizar recursos de banco de dados (20–25%)](#d3--monitor-configure-and-optimize-database-resources-2025) — 59 questões
- [D4 — Configurar e gerenciar a automação de tarefas (15–20%)](#d4--configure-and-manage-automation-of-tasks-1520) — 28 questões
- [D5 — Planejar e configurar um ambiente de alta disponibilidade e recuperação de desastres (HA/DR) (20–25%)](#d5--plan-and-configure-a-high-availability-and-disaster-recovery-hadr-environment-2025) — 13 questões
- [Estudos de caso](#case-studies)
- [Questões excluídas (fora do conteúdo atual)](#excluded-questions-not-in-current-outline)

<a name="d1--plan-and-implement-data-platform-resources-1520"></a>

## D1 — Planejar e implementar recursos da plataforma de dados (15–20%)

*65 perguntas*

### Q1. Ativar page compression na tabela CHAR(400)

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1. Você tem uma tabela chamada Table1 que tem 20 colunas do tipo CHAR(400). Row compression para Table1 está ativado. Durante uma auditoria de banco de dados, você descobre que nenhum dos campos contém mais de 150 caracteres. Você precisa garantir que você possa aplicar page compression ao Table1. O que você deve fazer?

**Alternativas:**
- **A.** Configure as colunas como esparsas.
- **B.** Altere o tipo de coluna para NVARCHAR(MAX).
- **C.** Altere o tipo de coluna para VARCHAR(MAX).
- **D.** Altere o tipo de coluna para VARCHAR(200).

**✅ Resposta:** D

**Explicação:** 20 × CHAR(400) = 8,000 bytes de dados fixos; com sobrecarga de compressão o tamanho máximo da linha excede 8,060 bytes, para que a compressão não possa ser ativada ("tamanho máximo da linha mais a regra de sobrecarga de compressão"). VARCHAR(200) ainda detém os valores de 150 caracteres e corta o tamanho máximo da linha para ~4,000 Bytes. Sparse columns são incompatíveis com data compression; (N)VARCHAR(MAX) dados podem ser empurrados fora da linha, onde não é comprimido (e NVARCHAR duplica o armazenamento).

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/data-compression/data-compression)

---

### Q2. factSales 6bn linhas: maior redução de espaço

**Questão:**
Você tem um Azure SQL database que contém uma tabela chamada factSales. FactSales contém as colunas mostradas na tabela a seguir.

| Nome | Tipo de dados |
|---|---|
| SalesID | Int |
| Product | Int |
| Total Number | Numeric(8,4) |
| Tax Number | Numeric(8,4) |
| SalesRep | Varchar(30) |

FactSales tem 6 bilhões de linhas e é carregado todas as noites usando um processo em lote. Você deve fornecer a maior redução de espaço para o banco de dados e maximizar o desempenho. Que tipo de compressão fornece a maior redução de espaço do banco de dados?

**Alternativas:**
- **A.** page compression
- **B.** row compression
- **C.** columnstore compression
- **D.** columnstore archival compression

**✅ Resposta:** D

**Explicação:** A pergunta que tipo de pergunta dá o *Maior redução de espaço*. Columnstore archival compression (COLUMNSTORE_ARCHIVE) aplica compressão XPRESS em cima de columnstore compression, por isso dá a menor pegada. Page e linha (rowstore) compressão dar proporções muito mais baixas do que columnstore em uma tabela fato de 6 bilhões de linhas. Plain columnstore compression é mais rápido para consultar, mas comprime menos do que arquivo.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/data-compression/data-compression)

---

### Q3. columnstore particionado: relação máxima, compressão indexável

**Questão:**
Você tem uma assinatura do Azure que contém um Azure SQL database. O banco de dados contém uma tabela chamada table1 que usa columnstores particionados. Você precisa configurar o table1 para atender aos seguintes requisitos:
- Cada partição deve ser comprimida.
- A taxa de compressão deve ser maximizada.
- Você deve ser capaz de indexar os dados compactados.

O que você deve usar?

**Alternativas:**
- **A.** page compression
- **B.** columnstore compression
- **C.** compressão GZIP
- **D.** columnstore archival compression

**✅ Resposta:** D

**Explicação:** As tabelas columnstore já usam columnstore compression. Para *maximizar* a proporção que você adiciona compressão de arquivo (COLUMNSTORE_ARCHIVE), que pode ser definida por partição e ainda é um índice columnstore, então os dados permanecem indexados. O requisito "índice" está lá para excluir GZIP: os dados comprimidos com a função COMPRESS não podem ser indexados. Page compression se aplica apenas ao rowstore e B.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/data-compression/data-compression)

---

### Q4. Tabela analítica retida: maximizar a redução do armazenamento

**Questão:**
Você tem um Azure SQL database. O banco de dados contém uma tabela de dados analíticos que devem ser retidos. Você precisa implementar uma solução data compression que maximize uma redução no armazenamento. Que tipo de compressão você deve usar?

**Alternativas:**
- **A.** página
- **B.** Arquivo columnstore
- **C.** linha
- **D.** columnstore

**✅ Resposta:** B

**Explicação:** COLUMNSTORE_ARCHIVE dá a maior taxa de compressão e se adequa aos dados que devem ser mantidos, mas raramente é consultado. Custa CPU extra e tempo de acesso. Linha e page compression (rowstore) e padrão columnstore todos comprimem menos.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/data-compression/data-compression)

---

### Q5. Efeitos ROW compression em colunas table1

**Questão:**
Você tem uma assinatura do Azure que contém um banco de dados do Azure SQL Database chamado DB1. Você executa as seguintes declarações T-SQL contra DB1.
```sql
CREATE TABLE table1 (
  column1 TINYINT,
  column2 VARCHAR(255),
  column3 NCHAR(10),
  column4 BINARY(50),
  column5 DATETIME2
);

ALTER TABLE table1 REBUILD WITH (DATA_COMPRESSION = ROW);
```
Para cada uma das seguintes afirmações, selecione Sim se a afirmação for verdadeira. Caso contrário, selecione Não.

**Alternativas:**
- S1. Todos os campos em column3 ocuparão 10 bytes no disco.
- S2. A compressão do dicionário será usada para reduzir o tamanho do table1 no disco.
- S3. Os tipos de dados armazenados em column1, column2 e column4 suportam compressão.

**✅ Resposta:** S1: Não; S2: Não; S3: Não

**Explicação:** S1: sem compressão, NCHAR(10) ocupa 20 bytes (2 bytes por caractere). Row compression remove o preenchimento no fim da cadeia e aplica compressão Unicode. O tamanho varia entre linhas, em vez de ser fixo em 10 bytes. S2: a compressão de dicionário e de prefixo faz parte de PAGE compression, não de ROW. S3: na tabela de row compression, tinyint aparece como “No” (1 byte já é o mínimo) e varchar como “No effect”. Entre esses tipos, só binary é afetado, com a remoção dos zeros à direita. Por isso, a afirmação sobre o conjunto é falsa.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/data-compression/row-compression-implementation)

---

### Q6. SIM: page-compress PK_SalesOrderHeader clustered index

**Questão:**
SIMULATION. Em um banco de dados do Azure SQL Database chamado db1, você precisa habilitar page compression no PK_SalesOrderHeader_SalesOrderID clustered index da tabela SalesLT.SalesOrderHeader. Para concluir essa tarefa, entre na máquina virtual. Você pode precisar usar SQL Server Management Studio e o Azure portal.

**Alternativas:**

**✅ Resposta:**
Opção 1, T-SQL (SSMS conectado ao db1):
```sql
ALTER INDEX PK_SalesOrderHeader_SalesOrderID
ON SalesLT.SalesOrderHeader
REBUILD WITH (DATA_COMPRESSION = PAGE);
```
Opção 2, SSMS GUI:
1. Conectar SSMS para o servidor lógico e expandir db1 Tabelas > SalesLT.SalesOrderHeader > Indexes.
2. Clique com o botão direito **PK_SalesOrderHeader_SalesOrderID** > **Armazenamento** > **Gerenciar Compressão**.
3. Na Data Compression Feiticeiro, conjunto **Tipo de compressão** = **Página** (opcionalmente selecione **Calcular** para estimar a economia) > **Próxima**.
4. Escolha **Correr imediatamente** > **Próxima** > **Acabamento**.
5. Verifique: `SELECT data_compression_desc FROM sys.partitions WHERE object_id = OBJECT_ID('SalesLT.SalesOrderHeader') AND index_id = 1;` retorna PAGE.

**Explicação:** Page compression em um índice existente é aplicado pela reconstrução do índice com DATA_COMPRESSION = PAGE (ALTER INDEX … REBUILD). O SSMS Data Compression O Assistente gera a mesma declaração. Porque este é o clustered index, os dados da tabela são comprimidos. Nonclustered indexes Não herde o cenário.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/data-compression/data-compression)

---

### Q7. ADatum: SVR1 para Instance1 serviço de migração / ponto final

**Estudo de caso:** ADatum v1 (serviços financeiros)

**Questão:**
Você precisa recomendar qual serviço e objetivo de destino usar ao migrar Os bancos de dados de SVR1 para Instance1. A solução deve atender aos requisitos de disponibilidade. O que você deve recomendar?
(Fatos do caso: SVR1 / SVR2 executar SQL Server 2016 Enterprise com AG1 contendo DB1 e DB2. O datacenter está conectado ao Network1 por S2S VPN. Instance1 é implantado para Network1. Exigência: minimizar o tempo de inatividade durante a migração de DB1 e DB2.)

**Alternativas:**

**Caixa 1 – Serviço de Migração:**
- **A.** Log Replay Service (LRS)
- **B.** Managed Instance link
- **C.** SQL Data Sync

**Caixa 2 – Endpoint alvo:**
- **A.** A private endpoint
- **B.** Um endpoint público
- **C.** Endpoint A VNet-local

**✅ Resposta:** Caixa 1: Managed Instance link; Caixa 2: Um ponto final VNet-local

**Explicação:** O MI link (distribuído AG), dá replicação quase em tempo real e o menor tempo de inatividade de qualquer MI método de migração. Ele suporta SQL Server 2016 SP3 + Pacote Azure Connect e posterior. LRS também funciona, mas tem mais tempo de inatividade de corte, e SQL Data Sync não apoia MI. Microsoft Learn A ligação só pode ser estabelecida com o VNet-local endpoint; público e private endpoints não pode ser usado. O S2S VPN E- Network1 fornece conectividade privada.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/managed-instance-link-feature-overview)

---

### Q8. Litware: migração SERVER1 Nível DMS + recurso

**Estudo de caso:** Litware

**Questão:**
Você está planejando a migração das bancos de dados SERVER1. A solução deve atender aos requisitos de negócios. O que você deve incluir no plano de migração? Para responder, selecione as opções apropriadas na área de resposta.
(Fatos do caso: SERVER1 é on-premises SQL Server 2012 com duas bancos de dados 1- TB. Alvo: Azure SQL Database. Exigência de negócio: minimizar o tempo de inatividade durante a migração das bancos de dados SERVER1. O datacenter primário tem ExpressRoute com peering privado para HubVNet.)

**Alternativas:**

**Caixa 1 – Nível de preços Azure Database Migration Service:**
- **A.** Padrão 2- vCore
- **B.** Padrão 4- vCore
- **C.** Premium 4- vCore

**Caixa 2 – Recursos necessários do Azure:**
- **A.** Uma rede virtual que tem service endpoints
- **B.** gateway AVPN
- **C.** Um aplicativo Azure Logic

**✅ Resposta:** Caixa 1: **Premium 4- vCore**; Caixa 2: **Uma rede virtual que tem service endpoints**

**Explicação:** Caixa 1: Em DMS (clássico) os níveis padrão fazem migrações offline somente. Premium é necessário para um *online* (mínimo de tempo de inatividade) migração, que é o requisito comercial. Caixa 2: DMS precisa de um Azure VNet Com conectividade para o on-premises fonte através de ExpressRoute : VPN. Litware **já tem ExpressRoute** (Microsoft + peering privado), então um novo VPN O gateway não acrescenta nada, e a exigência de minimizar on-premises A infraestrutura da Microsoft argumenta contra uma. DMS Os pré-requisitos indicam que quando ExpressRoute é usado, o DMS Necessidades de sub-rede **service endpoints** (alvo SQL, Armazenamento, Serviço Bus), porque DMS Não tem conectividade com a Internet. Um aplicativo Logic é irrelevante.
Nota da era do exame: DMS (clássico) aposentado em 15 Mar 2026. Este item testa o comportamento clássico.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/dms/pre-reqs) [Doc 2](https://learn.microsoft.com/en-us/azure/dms/tutorial-sql-server-azure-sql-database-offline)

---

### Q9. SQL 2019 de 2 TB para MI com o menor downtime e esforço

**Questão:**
Você tem um datacenter on-premises que contém um banco de dados 2-TB Microsoft SQL Server 2019 chamado DB1. Você precisa recomendar uma solução para migrar DB1 para um Azure SQL managed instance. A solução deve minimizar o tempo de inatividade e o esforço administrativo. O que você deve incluir na recomendação?

**Alternativas:**
- **A.** Log Replay Service (LRS)
- **B.** log shipping
- **C.** transactional replication
- **D.** SQL Data Sync

**✅ Resposta:** A

**Explicação:** LRS é o gerenciado, log-shipping-based MI Serviço de migração. Restaura o serviço completo/diff/log backups A partir Blob Storage continuamente e, em seguida, corta, então o tempo de inatividade é baixo com pouca configuração. Você não pode configurar clássico log shipping E- MI Você mesmo (sem manual NORECOVERY restaurações). Transactional replication é uma opção "alternativa" com configuração relativamente complexa e limitações por tabela. SQL Data Sync não apoia MI.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/data-migration/sql-server/managed-instance/overview)

---

### Q10. SQL 2019 de 4 TB para MI com o menor downtime e perda de dados

**Questão:**
Você tem uma Microsoft SQL Server 2019 instance in an on-premises A instância contém um 4-TB banco de dados nomeado DB1. Você planeja migrar DB1 a uma Azure SQL Database managed instance. O que você deve usar para minimizar o tempo de inatividade e perda de dados durante a migração?

**Alternativas:**
- **A.** distribuído availability groups
- **B.** banco de dados espelhamento
- **C.** Always On Availability Group
- **D.** Azure Database Migration Service

**✅ Resposta:** D

**Explicação:** DMS Execu *online* Migrações para SQL MI: ele continuamente restaura log backups, e o tempo de inatividade e a perda de dados são limitados ao corte final. Você não pode configurar o espelhamento do banco de dados ou um AG com MI como uma réplica. Um distribuído AG E- MI Existe apenas dentro do Managed Instance link recurso, não como um recurso autônomo que você configura. Com DMS Na lista, D é a resposta pretendida. Nota: uma variante diferente desta pergunta não tem DMS (opções: distribuído AGs / espelhamento / log shipping / DMA). Nessa variante, "distribuído availability groups" (= MI tecnologia) seria a resposta.

**Segunda opinião:** Disputado: o link MI (construído em **distribuído availability groups**, opção A) dá o menor tempo de inatividade e pode falhar sem perda de dados. Algumas chaves e revisores independentes escolhem A, enquanto as chaves CertyIQ e SPOTO escolhem D (migração on-line DMS). Ambos atendem ao objetivo. D é a chave publicada aqui.

**Dumps:** ✔ concordam
**Confiança:** Baixa · [Doc 1](https://learn.microsoft.com/en-us/azure/dms/dms-overview) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/managed-instance-link-feature-overview)

---

### Q11. 60- TB sobre 10 Mbps: migração mais rápida

**Questão:**
Sua rede on-premises contém um servidor que hospeda um banco de dados 60-TB chamado DB1. A rede tem uma conexão de internet de 10 Mbps. Você precisa migrar DB1 para Azure. A solução deve minimizar quanto tempo leva para migrar o banco de dados. O que você deve usar?

**Alternativas:**
- **A.** Data Migration Assistant (DMA)
- **B.** Azure Migrate
- **C.** Azure Data Studio
- **D.** Caixa de dados Azure

**✅ Resposta:** D

**Explicação:** O envio de 60 TB por um link de 10 Mbps leva bem mais de um ano (~1.5 anos a taxa de linha completa), então cada ferramenta on-line (DMA, Azure Migrate, Azure Data Studio / DMS) é limitada pelo link. Azure Data Box é offline da Microsoft, aparelho enviado para transferências maiores que 40 TB sem conectividade de rede ou limitada. Você envia o backups / data no dispositivo.

**Variante vista em outros dumps:** mesma pergunta com opções `A. Azure Migrate / B. Azure Data Box / C. Azure Database Migration Service / D. Data Migration Assistant (DMA)` → resposta **B**.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/databox/data-box-overview)

---

### Q12. Backup para bloquear blob para MI: segurança máxima

**Questão:**
Você tem um on-premises Microsoft SQL Server instância de 2019 nomeada SQL1 que hospeda um banco de dados chamado db1. Você tem uma assinatura do Azure que contém um Azure SQL managed instance nomeado MI1 e um Azure Storage conta nomeada storage1. Você planeja migrar db1 E- MI1 usando o backup e restaurar o processo. Você precisa garantir que você pode fazer backup db1 E- storage1. A solução deve atender aos seguintes requisitos:
- Use o bloco blob storage.
- Maximize a segurança.

O que você deve fazer no storage1?

**Alternativas:**
- **A.** Gerar uma assinatura de acesso compartilhado (SAS).
- **B.** Crie uma política de acesso.
- **C.** Girar as chaves de armazenamento.
- **D.** Habilitar criptografia de infraestrutura.

**✅ Resposta:** A

**Explicação:** BACKUP TO URL escreve bloco blobs somente quando o SQL credencial detém a SAS token. Uma chave de conta de armazenamento na credencial produz blobs de página. Microsoft chama SAS a maneira mais segura de autorizar o acesso ao blob em comparação com a chave de armazenamento, porque é escopo e tempo limitado. Uma política de acesso por si só não dá uma credencial. A rotação da chave e a criptografia de infraestrutura não permitem backup E- URL.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/sql-server-backup-to-url)

---

### Q13. Cópia MI banco de dados DB2 para SQL Server 2022

**Questão:**
Você tem um servidor on-premises chamado Server1 que tem o Microsoft SQL Server 2022 instalado. Você tem uma assinatura do Azure que contém um Azure SQL managed instance chamado MI1. MI1 contém um banco de dados chamado DB2. Você precisa criar uma cópia do DB2 no Server1. Quais três ações você deve executar em sequência?

**Alternativas:**
Acções:
- **A.** Faça backup do DB2 em um arquivo e use a opção COPY_ONLY.
- **B.** Faça backup do DB2 para um URL e use a opção COPY_ONLY.
- **C.** Recupere DB2 do arquivo e use a opção RESTORE WITH RECOVERY.
- **D.** Crie uma credencial no MI1.
- **E.** Crie uma credencial no Server1.
- **F.** Recupere o DB2 do URL e use a opção WITH MOVE.

**✅ Resposta:** 1. Crie uma credencial no MI1 → 2. Faça backup do DB2 para um URL e use a opção COPY_ONLY → 3. Restaure o DB2 do URL e use a opção WITH MOVE

**Explicação:** O procedimento “Restore a database to SQL Server from SQL MI” do Microsoft Learn orienta a criar uma credential na MI (managed identity ou SAS) para o storage container, fazer um backup COPY_ONLY TO URL e executar RESTORE … FROM URL WITH MOVE no SQL Server 2022. WITH MOVE com caminhos explícitos para os arquivos é obrigatório. MI não faz backup para arquivo local, então as ações de “file” estão incorretas. Na prática, Server1 também precisa de acesso de leitura ao blob, com uma credential. A resposta em três etapas esperada pela questão segue a sequência documentada. Pré-requisito: MI1 deve usar a política de atualização do SQL Server 2022.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/restore-database-to-sql-server)

---

### Q14. DMS sobre ExpressRoute a SQLMI1: Configuração VNET1

**Questão:**
Você tem um on-premises Microsoft SQL Server servidor 2019 que hospeda um banco de dados chamado DB1. Você tem uma assinatura do Azure que contém um Azure SQL managed instance nomeado SQLMI1 e uma rede virtual chamada VNET1. SQLMI1 reside em VNET1. O on-premises A rede conecta-se a VNET1 usando um ExpressRoute conexão. Você planeja migrar DB1 E- SQLMI1 usando Azure Database Migration Service. Você precisa configurar VNET1 para apoiar a migração. O que você deve fazer?

**Alternativas:**
- **A.** Configurar service endpoints.
- **B.** Configurar peering de rede virtual.
- **C.** Implemente um Azure firewall.
- **D.** Configurar network security groups (NSGs).

**✅ Resposta:** A

**Explicação:** O DMS (clássico) pré-requisitos disse: se você usar ExpressRoute network peering para a Microsoft, adicione service endpoints (banco de dados alvo/Microsoft.Sql, Armazenamento, Serviço Bus) para a sub-rede onde DMS E provisioned, porque DMS Não tem conectividade direta com a Internet. VNet não é necessário peering porque SQLMI1 e DMS são sobre o mesmo VNET1 e on-prem A conectividade já existe através de ExpressRoute. NSGs apenas precisa de *não bloquear* os portos requeridos (443, 1433, etc.); criando NSGs não é a configuração necessária. An Azure Firewall Não é necessário.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-ca/azure/dms/resource-network-topologies)

---

### Q15. Bancos de dados Five SQL1 para MI: sem perda de dados

**Questão:**
Você tem um on-premises Microsoft SQL Server chamado SQL1 que hospeda cinco bancos de dados. Você precisa migrar os bancos de dados para um Azure SQL managed instance. A solução deve minimizar o tempo de inatividade e evitar a perda de dados. O que você deve usar?

**Alternativas:**
- **A.** Always On availability groups
- **B.** Backup e restauração
- **C.** log shipping
- **D.** Database Migration Assistant

**✅ Resposta:** B

**Explicação:** Das opções listadas, apenas nativas backup e restaurar (full.bak from Azure Blob Storage, RESTORE FROM URLé um método de migração suportado para SQL MI. A Microsoft a chama de "a opção de migração mais fácil", e com a fonte quiesced antes da final backup não há perda de dados. Você não pode adicionar MI como uma réplica em um regular Always On AG; AG tecnologia alcança MI apenas através da Managed Instance link Característica. Manual log shipping não é possível porque MI não permite usuário RESTORE … WITH NORECOVERY; o equivalente gerenciado é LRS. DMA apenas avalia e não pode migrar para MI. Se "Managed Instance link", "LRS" ou "DMS" foram oferecidos, eles dariam menor tempo de inatividade.

**Segunda opinião:** Disputado: com o Managed Instance link (tecnologia Always On AG, opção A), o tempo de inatividade é próximo de zero. Pass4Success marca C, enquanto MD, CertyIQ e SPOTO marca B. B é a chave mais amplamente publicada e o único método clássico totalmente suportado na lista.

**Dumps:** ✔ concordam
**Confiança:** Baixa · [Doc 1](https://learn.microsoft.com/en-us/data-migration/sql-server/managed-instance/overview)

---

### Q16. Hyperscale a Business Critical: primeiro passo

**Questão:**
Você tem dois Azure SQL databases nomeado SQLDB1 e SQLDB2 na Hyperscale nível de serviço. SQLDB1 foi definido para o Hyperscale nível de serviço quando o banco de dados foi criado. SQLDB2 foi alterado a partir do Business Critical nível de serviço para o Hyperscale nível de serviço durante os últimos 14 dias. Você precisa mudar SQLDB1 e SQLDB2 E a Business Critical nível de serviço. A solução deve minimizar o esforço administrativo. O que você deve fazer primeiro?

**Alternativas:**

**On SQLDB1:**
- **A.** Exporte os dados.
- **B.** Definir nível de serviço para Business Critical.
- **C.** Definir nível de serviço para General Purpose.

**On SQLDB2:**
- **A.** Exporte os dados.
- **B.** Definir nível de serviço para Business Critical.
- **C.** Definir nível de serviço para General Purpose.

**✅ Resposta:** SQLDB1: Exportar os dados; SQLDB2: Definir nível de serviço para General Purpose

**Explicação:** Bancos de dados originalmente criadas em Hyperscale A única saída é a exportação / importação (bacpac ou outro movimento de dados), então SQLDB1 devem ser exportados primeiro. SQLDB2 foi convertido dentro da janela de 45 dias, para que possa ser migrado para trás. General Purpose somente; você então muda GP → Business Critical como uma operação de escala normal. Uma mudança direta para Business Critical Não é permitido para qualquer banco de dados.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/reverse-migrate-from-hyperscale)

---

### Q17. cmdlet LRS com o auth de armazenamento mais seguro

**Questão:**
Você tem um Azure SQL managed instance nomeado Server1 e um Azure Blob Storage conta nomeada storage1 que contém a Microsoft SQL Server banco de dados backup arquivos. Você planeja usar Log Replay Service para migrar o backup arquivos de storage1 E- Server1. A solução deve usar o mais alto nível de segurança ao se conectar a storage1. Que PowerShell cmdlet você deve executar, e qual parâmetro você deve especificar para garantir a conexão?

**Alternativas:**

**Cmdlet:**
- **A.** Completo- AzSqlInstanceDatabaseLogReplay
- **B.** Get-AzSqlInstanceDatabaseLogReplay
- **C.** Start-AzSqlInstanceDatabaseLogReplay

**Parâmetro:**
- **A.** ManagedIdentity
- **B.** SharedAccessSignature
- **C.** StorageAccountKey

**✅ Resposta:** Cmdlet: Start-AzSqlInstanceDatabaseLogReplay; Parâmetro: ManagedIdentity

**Explicação:** Start-AzSqlInstanceDatabaseLogReplay inicia a restauração LRS. Seu parâmetro -StorageContainerIdentity leva ManagedIdentity ou SharedAccessSignature. ManagedIdentity (a identidade do MI concedida Storage Blob Data Reader no contêiner) não precisa de nenhum segredo ou token, por isso é o mais seguro. SharedAccessSignature precisa de um token SAS (um segredo). StorageAccountKey não é apenas uma opção de autenticação LRS. Obter

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/powershell/module/az.sql/start-azsqlinstancedatabaselogreplay) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/log-replay-service-migrate)

---

### Q18. Monitor MI -to- MI database copy desempenho

**Questão:**
Você tem uma assinatura do Azure que contém duas instâncias gerenciadas Azure SQL chamadas SQLMI1 e SQLMI2. SQLMI1 hospeda um banco de dados chamado DB1. Você inicia uma operação de cópia de DB1 para SQLMI2. Você precisa monitorar o desempenho da operação de cópia. Qual visão de gerenciamento dinâmico você deve usar?

**Alternativas:**
- **A.** sys.dm_continuous_copy_status
- **B.** sys.dm_database_replica_states
- **C.** sys.dm_hadr_physical_seeding_stats
- **D.** sys.dm_db_mirroring_connections

**✅ Resposta:** C

**Explicação:** MI database copy/move usa Always On Semeadura. Microsoft Learn “O processo de semeadura e sua velocidade podem ser monitorados via DMV sys.dm_hadr_physical_seeding_stats" (transfer_rate_bytes_per_second, estimate_time_complete_utc, …). sys.dm_continuous_copy_status é para Azure SQL Database geo-replication / database copy. sys.dm_database_replica_states mostra AG estado de sincronização, não taxa de transferência de semeadura. sys.dm_db_mirroring_connections é para conexões de espelhamento de banco de dados.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/database-copy-move-how-to)

---

### Q19. Migração offline MI via ADS: instalar em SQL1

**Questão:**
Você tem uma instância on-premises Microsoft SQL Server chamada SQL1 que hospeda vários bancos de dados. Você tem uma assinatura do Azure que contém um Azure SQL managed instance chamado SQLMI1. Você precisa executar uma migração off-line das bancos de dados SQL1 para SQLMI1 usando Azure Data Studio. O que você deve instalar no SQL1?

**Alternativas:**
- **A.** Data Migration Assistant (DMA)
- **B.** Ferramentas de Dados SQL Server (SSDT)
- **C.** um tempo de execução de integração auto-hospedado
- **D.** aparelho Azure Migrate

**✅ Resposta:** C

**Explicação:** O Azure SQL Migration extension (Azure Data Studio) utiliza Azure DMS. Quando backups estão em um on-premises SMB partilha, DMS precisa de um tempo de execução de integração auto-hospedado (SHIR), que pode chegar à fonte SQL Server e o compartilhamento e upload do backups E- Azure Storage. DMA É uma ferramenta de avaliação. SSDT é uma ferramenta de desenvolvimento. O Azure Migrate Nota: A Microsoft recomenda uma máquina separada perto da fonte para a SHIRmas destas opções é o único componente necessário. Azure Data Studio foi aposentado em 28 de fevereiro de 2026; o mesmo DMS + SHIR A exigência agora se aplica através do Azure portal/PowerShell/CLI.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/data-migration/sql-server/self-hosted-integration-runtime) [Doc 2](https://learn.microsoft.com/en-us/AZURE/dms/migration-using-azure-data-studio)

---

### Q20. Migração de tempo mínimo para Azure SQL Database

**Questão:**
Você precisa migrar um banco de dados on-premises Microsoft SQL Server para Azure SQL Database. A solução deve minimizar o tempo de inatividade. O que você deve fazer?

**Alternativas:**
- **A.** Configurar a Transação Log Shipping.
- **B.** Implemente o Always On availability groups.
- **C.** Configurar transactional replication.
- **D.** Importe um BACPAC.

**✅ Resposta:** C

**Explicação:** Azure SQL Database pode ser um assinante push do transactional replication unidirecional do SQL Server; A Microsoft documenta isso como o cenário de migração de dados (replicar, redirecionar aplicativos, soltar a publicação), portanto, o tempo de inatividade é limitado ao cutover. Log shipping e AGs não podem atingir o Azure SQL Database (eles restauram / semeiam um banco de dados de instância completo) e uma importação BACPAC é um método offline com tempo de inatividade para toda a exportação / importação.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/replication-to-sql-database)

---

### Q21. Migração online DMS: Tipo e opção backup

**Questão:**
Você tem um servidor on-premises Microsoft SQL Server 2016 chamado Server1 que contém um banco de dados chamado DB1. Você precisa realizar uma migração on-line de DB1 para um Azure SQL Managed Instance usando Azure Database Migration Service. Como você deve configurar o backup de DB1?

**Alternativas:**

**Tipo Backup:**
- **A.** Somente backups completo e log
- **B.** Somente Full backup
- **C.** Somente o Log backup

**Opção Backup:**
- **A.** WITH CHECKSUM
- **B.** WITH NOINIT
- **C.** WITH UNLOAD

**✅ Resposta:** Tipo Backup: Somente backups completo e log; Opção Backup: WITH CHECKSUM

**Explicação:** Uma migração on-line (restauração contínua) precisa de um full backup mais o log subsequente backups que DMS Restabelece até o corte. Backups devem ser tomadas WITH CHECKSUM (clássico DMS necessário; atual DMS valida checksums automaticamente). WITH NOINIT que não é suportado (cada um backup deve estar em seu próprio arquivo), e WITH UNLOAD É uma opção de fita.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/dms/faq) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/log-replay-service-migrate)

---

### Q22. Migração online DMS: minimizar arquivos backup

**Questão:**
Você tem um on-premises Microsoft SQL Server instância de 2016 que hospeda um banco de dados chamado db1. Você tem uma assinatura do Azure que contém um Azure SQL managed instance nomeado MI1. Você planeja realizar uma migração online de db1 E- MI1 usando Azure Database Migration Service. Você precisa criar o backups A solução deve minimizar o número de backup arquivos criados. Que tipo de backups você deve criar, e como você deve armazenar o backups?

**Alternativas:**

**Tipo Backup:**
- **A.** Apenas completo
- **B.** Completo e diferencial
- **C.** Log completo e de transação
- **D.** Somente log de transação

**Para armazenar o backups:**
- **A.** Anexe todo o backups a um único arquivo.
- **B.** Inclua cada backup em um arquivo separado.
- **C.** Escreva cada backup em um arquivo separado.

**✅ Resposta:** Tipo Backup: log completo e de transação; Para armazenar o backups: Escreva cada backup em um arquivo separado.

**Explicação:** A migração on-line requer um full backup seguido de log backups (diferenciais adicionam arquivos sem ser necessário). DMS não suporta anexar vários backups (e.g., completo e log) em um único mídia backup, então o mínimo é um arquivo por backup ("escrever cada backup para um arquivo separado"); "adicionar cada backup a um arquivo separado" é uma palavra distrativa.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/dms/faq)

---

### Q23. Migração online DMS: comando e localização backup

**Questão:**
Você tem uma instância on-premises Microsoft SQL Server 2019 que hospeda um banco de dados chamado DB1. Você planeja realizar uma migração on-line de DB1 para um Azure SQL managed instance usando Azure Database Migration Service. Você precisa criar um backup do DB1 que seja acessível ao Azure Database Migration Service. O que você deve executar para o backup e onde você deve armazenar o backup?

**Alternativas:**

**Executar:**
- **A.** A full backup e um log backup anexado ao mesmo arquivo usando o WITH CHECKSUM opção
- **B.** A full backup e um log backup para separar arquivos usando o WITH CHECKSUM opção
- **C.** A full backup e um log backup para separar arquivos usando o WITH FILE_SNAPSHOT opção

**Armazene o backup em:**
- **A.** Um cofre de serviços de recuperação
- **B.** Conta Azure Blob storage
- **C.** Compartilhamento de arquivos An SMB

**✅ Resposta:** Executar: A full backup e um log backup para separar arquivos usando a opção WITH CHECKSUM; Armazene o backup em: Um compartilhamento de arquivo SMB

**Explicação:** DMS restaura nativo backups que deve estar em arquivos separados (aplicando-se a uma mídia não é suportado) e tomadas WITH CHECKSUM; FILE_SNAPSHOT apenas para bancos de dados cujos arquivos já estão em Azure Blob storage. Clássico DMS (o serviço da era do exame) exigiu o backups em um SMB compartilhamento de rede que poderia alcançar. Nota: atual DMS (Azure SQL migration extension/portal) aceita um ou SMB ou um recipiente Azure Blob, então "Azure Blob storage conta" também funcionaria hoje, mas um cofre de serviços de recuperação nunca funciona. Manter SMB compartilhar como a resposta do exame.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/dms/faq)

---

### Q24. Tipo de replicação para Azure SQL Database

**Questão:**
Você tem uma instância on-premises Microsoft SQL Server 2019 que hospeda um banco de dados chamado DB1. Você tem uma assinatura do Azure que contém um banco de dados do Azure SQL Database chamado SQLDB1. Você precisa replicar DB1 para SQLDB1. Que tipo de replicação você deve usar?

**Alternativas:**
- **A.** transacional
- **B.** peer-to-peer
- **C.** snapshot
- **D.** merge

**✅ Resposta:** A

**Explicação:** Azure SQL Database suporta apenas transacional padrão (one-way) e snapshot replication, e apenas como um assinante push. A fusão e peer-to-peer não são suportados. Snapshot copia apenas dados point-in-time e não mantém o assinante em sincronia, portanto, replicação contínua de DB1 significa transacional.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/replication-to-sql-database)

---

### Q25. Replicação para SQLDB1: papel e tipo de assinante

**Questão:**
Você tem uma instância on-premises Microsoft SQL Server chamada SQLSVR1 que hospeda um banco de dados chamado DB1. Você tem uma assinatura do Azure que contém um banco de dados do Azure SQL Database chamado SQLDB1. Você precisa migrar os dados armazenados em DB1 para SQLDB1 usando a replicação SQL Server. A solução deve minimizar o impacto no desempenho do DB1. Como você deve configurar a replicação?

**Alternativas:**

**Configurar SQLDB1:**
- **A.** Apenas como um assinante push
- **B.** Como assinante e distribuidor
- **C.** Para usar apenas uma assinatura pull

**Tipo de replicação:**
- **A.** Snapshot
- **B.** Transacional com peer-to-peer replication
- **C.** Transacional com subscrições atualizadas

**✅ Resposta:** Configurar SQLDB1: Apenas como assinante push; Tipo de replicação: Snapshot

**Explicação:** Azure SQL Database só pode ser um assinante push. O banco de dados de distribuição e os agentes de replicação não podem ser colocados no Azure SQL Database, e as assinaturas pull não são suportadas. Dos tipos listados, apenas o snapshot é suportado: Azure SQL Database não suporta assinaturas peer-to-peer ou updatable (atualização imediata / filada). O transactional replication unidirecional seria ideal, mas não está entre as escolhas.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/replication-to-sql-database)

---

### Q26. AG em Windows SQL 2017 para Linux SQL 2019

**Questão:**
Você tem dois on-premises servidores que executam Windows Server 2019 e hospedar uma Microsoft SQL Server 2017 Always On availability group nomeado AG1. AG1 contém um único banco de dados chamado DB1. Você tem uma assinatura do Azure. A assinatura contém uma máquina virtual chamada VM1 que roda Linux. Você precisa migrar DB1 a a SQL Server 2019 instance on VM1. A solução deve minimizar o tempo de inatividade de DB1 durante a migração. O que você deve fazer?

**Alternativas:**

**Para se preparar para a migração:**
- **A.** Adicione uma réplica secundária ao AG1.
- **B.** Crie um SQL Server 2019 Always On availability group em VM1.
- **C.** Atualize os servidores on-premises SQL para SQL Server 2019.

**Para realizar a migração, use:**
- **A.** A distributed availability group
- **B.** Azure Migrate
- **C.** Log shipping

**✅ Resposta:** Preparar: Criar um SQL Server 2019 Always On availability group em VM1; Executar: A distributed availability group

**Explicação:** Um distributed AG reúne dois AGs separados e funciona entre plataformas, como um AG Windows WSFC e um AG Linux. Desde o SQL Server 2017, também pode combinar versões principais quando o AG primário usa a versão mais antiga. A Microsoft descreve esse cenário de migração ou upgrade com downtime limitado ao failover final. O seeding precisa ser manual para um destino de versão mais recente. Como é necessário um AG em cada lado, crie um AG em VM1. Adicionar uma réplica SQL 2019 em Linux ao Windows 2017 AG1 não é uma configuração compatível de longo prazo entre versões e clusters. Azure Migrate faz lift-and-shift de servidores inteiros e não move um banco para uma instância SQL 2019 em Linux que já existe. Log shipping funciona, mas requer um cutover mais longo.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/distributed-availability-groups?view=sql-server-ver17) [Doc 2](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-availability-group-cross-platform?view=sql-server-ver16)

---

### Q27. Recurso de bloqueio migração para Azure SQL Database

**Questão:**
Você tem um banco de dados Microsoft SQL Server 2019 chamado DB1 que usa os seguintes recursos de nível de banco de dados e de nível de instância:
- Clustered columnstore indexes
- Automatic tuning
- Change tracking
- PolyBase

Você planeja migrar o DB1 para um Azure SQL database. Que recurso deve ser removido ou substituído antes que o DB1 possa ser migrado?

**Alternativas:**
- **A.** Clustered columnstore indexes
- **B.** PolyBase
- **C.** Change tracking
- **D.** Automatic tuning

**✅ Resposta:** B

**Explicação:** Azure SQL Database suportes columnstore índices (S3 e acima, e todos vCore camadas), change tracking e automatic tuning. PolyBase é um nível de instância SQL Server recursos (fontes de dados externos como Hadoop, Oracle, Teradata e outros SQL Servidores) que Azure SQL Database Sua única substituição é limitada "virtualização de dados" (OPENROWSET e tabelas externas sobre CSV, Parquet ou Delta arquivos em Azure Storage), assim PolyBase O uso deve ser substituído.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/features-comparison)

---

### Q28. Ferramentas para avaliar SQL Server para alvos Azure

**Questão:**
Você tem oito servidores on-premises que têm o Microsoft SQL Server 2022 instalado. Cada servidor contém vários bancos de dados. Você planeja migrar os bancos de dados on-premises para o Azure. Você precisa analisar os servidores para identificar quais serviços Azure SQL podem ser usados para hospedar os bancos de dados. Quais duas ferramentas você pode usar? Cada resposta correta apresenta uma solução completa.

**Alternativas:** *(algumas opções reconstruídas)*
- **A.** Data Migration Assistant (DMA)
- **B.** SQL Server Migration Assistant (SSMA)
- **C.** Azure Database Migration Service
- **D.** Azure Migrate
- **E.** *(desconhecido)*

**✅ Resposta:** Com ferramentas atuais: Azure Migrate, mais SQL Server enabled by Azure Arc (avaliação de migração) no lugar de DMA.

**Explicação:** Azure Migrate Descobrir SQL Server em escala e produz recomendações de prontidão e tamanho certo para Azure SQL Database, SQL Managed Instance e SQL Server on Azure VMs. DMA compatibilidade e recomendado Azure SQL foi a segunda resposta clássica. No entanto, a Microsoft se aposentou DMA em 16 de julho de 2025 e aponta para Azure Arc-enabled SQL Server avaliações de migração e Azure Migrate Em vez disso. SSMA é para não-SQL Server fontes, e DMS realiza migrações em vez de avaliar as opções-alvo.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/sql/sql-server/azure-arc/migration-overview?view=sql-server-ver17) [Doc 2](https://techcommunity.microsoft.com/blog/microsoftdatamigration/announcing-retirement-of-microsoft-data-migration-assistant-dma-tool/4424400)

---

### Q29. 14- Banco de dados TB: SQL MI nível / suporte de hardware

**Questão:**
Você tem um datacenter on-premises que contém um banco de dados TB Microsoft SQL Server 14-X. Você planeja criar um Azure SQL managed instance e migrar o banco de dados on-premises para a nova instância. Quais três níveis de serviço suportam o SQL managed instance? Cada resposta correta apresenta uma solução completa.

**Alternativas:**
- **A.** Padrão General Purpose
- **B.** Memória Business Critical Otimizada Premium
- **C.** General Purpose Premium
- **D.** Business Critical Premium
- **E.** Padrão Business Critical

**✅ Resposta:** A, B, C

**Explicação:** Armazenamento máximo de instâncias de acordo com os limites atuais de recursos:
- General Purpose: 16 TB em todo o hardware (32 TB na próxima geração GP).
- Business Critical, série padrão: 4 TB, então não pode conter o banco de dados.
- Business Critical, memória otimizado premium-série: 16 TB em 64 ou mais vCores.
- Business Critical, série premium: 5.5 TB na maioria das regiões, mas 16 TB em 64 ou mais vCores nas principais regiões.

Os três pretendidos são série padrão GP, série premium GP e série premium otimizada para memória BC. Caveata: nas principais regiões, a série premium BC (D) agora também pode chegar a 16 TB, então D é tecnicamente possível hoje. A resposta garantida regionalmente permanece A, B, C.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits)

---

### Q30. 60- TB banco de dados: Azure SQL Database camada

**Questão:**
Você tem uma instância on-premises Microsoft SQL Server 2022 que hospeda um banco de dados de produção 60- TB chamado DB1. Você planeja migrar DB1 para Azure. Você precisa recomendar uma solução de hospedagem para DB1. Qual nível de serviço Azure SQL Database você deve usar para hospedar DB1?

**Alternativas:**
- **A.** Hyperscale
- **B.** Business Critical
- **C.** General Purpose

**✅ Resposta:** A

**Explicação:** Os níveis General Purpose e Business Critical de Azure SQL Database estão no topo em 4 TB de dados. O Hyperscale suporta bancos de dados de até 128 TB (o limite mais antigo era 100 TB), então é o único nível Azure SQL Database que pode hospedar 60 TB.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale?view=azuresql)

---

### Q31. FileTable / FILESTREAM: escolha a oferta Azure SQL

**Questão:**
Você tem um on-premises Microsoft SQL Server que usa os recursos FileTables e Filestream. Você planeja migrar para Azure SQL. Qual serviço você deve usar?

**Alternativas:**
- **A.** Azure SQL Database
- **B.** SQL Server em um Azure Virtual Machine
- **C.** Azure SQL Managed Instance
- **D.** Banco de Dados do Azure para MySQL

**✅ Resposta:** B

**Explicação:** FILESTREAM e FileTable não são suportados em Azure SQL Database ou Azure SQL Managed Instance. As listas de comparação de recursos "Não" para ambos. Apenas SQL Server em uma Azure VM (SQL Server completo com acesso ao OS /file-system) os suporta. O Banco de Dados do Azure para MySQL não é SQL Server.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/features-comparison)

---

### Q32. Latest SQL versão + SQL Agent, menos admin

**Questão:**
Você tem um servidor Microsoft SQL Server 2017. Você precisa migrar o servidor para o Azure. A solução deve atender aos seguintes requisitos:
- Certifique-se de que a versão mais recente do SQL Server é usada.
- Suporte ao serviço SQL Server Agent.
- Minimizar o esforço administrativo.

O que você deve usar?

**Alternativas:**
- **A.** Azure SQL Database elastic pool
- **B.** Azure SQL Database
- **C.** SQL Server on Azure Virtual Machines
- **D.** Azure SQL Managed Instance

**✅ Resposta:** D

**Explicação:** SQL Managed Instance é um motor PaaS (sempre verde / sempre atualizado) que a Microsoft corrige e atualiza, e inclui SQL Server Agent. Azure SQL Database (banco de dados único ou elastic pool) não tem SQL Server Agent e usa elastic jobs em vez disso. Um SQL Server VM apoia o agente, mas você deve instalar, corrigir e atualizá-lo sozinho, o que significa mais esforço administrativo.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/features-comparison)

---

### Q33. Linked server to on-premises SQL2, menos administrador

**Questão:**
Você tem duas instâncias on-premises Microsoft SQL Server 2019 chamadas SQL1 e SQL2. Você precisa migrar Os bancos de dados hospedadas no SQL1 para o Azure. A solução deve atender aos seguintes requisitos:
- O serviço que hospeda os bancos de dados migrados deve ser capaz de se comunicar com o SQL2 usando conexões de servidor vinculadas.
- O esforço administrativo deve ser minimizado.

O que você deve usar para hospedar Os bancos de dados?

**Alternativas:**
- **A.** um único Azure SQL database
- **B.** SQL Server on Azure Virtual Machines
- **C.** Azure SQL Managed Instance
- **D.** Azure SQL Database elastic pool

**✅ Resposta:** C, Azure SQL Managed Instance

**Explicação:** O SQL Managed Instance suporta servidores vinculados aos alvos SQL Server e SQL Database (sem transações distribuídas) e é PaaS, por isso precisa de menos administração. O Azure SQL Database (único ou agrupado) não suporta servidores vinculados. O SQL Server VM suporta servidores vinculados, mas requer gerenciamento OS e SQL.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/features-comparison)

---

### Q34. Merge replication: escolha a oferta Azure SQL

**Questão:**
Você tem um banco de dados on-premises Microsoft SQL Server 2019 chamado SQL1 que usa merge replication. Você precisa migrar SQL1 para Azure. Qual serviço você deve usar?

**Alternativas:**
- **A.** Azure SQL Borda
- **B.** Azure SQL Database
- **C.** SQL Server on Azure Virtual Machines
- **D.** Azure SQL Managed Instance

**✅ Resposta:** C

**Explicação:** Nenhum dos Azure SQL Database nem Azure SQL Managed Instance suportes merge replication. Managed Instance apoia transacional, snapshot replicação bidirecional, e SQL Database só pode ser transacional ou snapshot Assinante. Apenas completo SQL Server em uma Azure VM mantém merge replication. Azure SQL O Edge é um motor de IoT / edge (agora aposentado) e não é um alvo de migração.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/replication-to-sql-database)

---

### Q35. SQL MI: menor latência, maior memória por vCore

**Questão:**
Você tem uma assinatura do Azure. Você precisa implantar um Azure SQL managed instance que atenda aos seguintes requisitos:
- Otimizar latência.
- Maximize a relação memória-para-vCore.

Qual nível de serviço e geração de hardware você deve usar?

**Alternativas:**

**Nível de serviço:**
- **A.** Business Critical
- **B.** General Purpose
- **C.** Hyperscale

**Geração de hardware:**
- **A.** Premium-série
- **B.** Série-padrão (Gen 5)
- **C.** Premium-series - memória otimizada

**✅ Resposta:** Nível de serviço: Business Critical; Geração de hardware: Série Premium - memória otimizada

**Explicação:** Business Critical usa armazenamento local SSD com cerca de 1-2 ms de latência de E/S, em comparação com 5-10 ms para General Purpose e 3-5 ms para General Purpose de próxima geração. Hyperscale não é um nível SQL Managed Instance. Memória por vCore é 5.1 GB na série padrão, 7 GB premium na série e 13.6 GB na memória premium-opized

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits)

---

### Q36. Nível SQL MI para minimizar a latência do disco

**Questão:**
Sua rede on-premises contém um servidor Microsoft SQL Server 2016 que hospeda um banco de dados chamado db1. Você tem uma assinatura do Azure. Você planeja migrar db1 para um Azure SQL managed instance. Você precisa criar o SQL managed instance. A solução deve minimizar a latência do disco da instância. Qual nível de serviço você deve usar?

**Alternativas:**
- **A.** Business Critical
- **B.** Hyperscale
- **C.** General Purpose
- **D.** Premium

**✅ Resposta:** A

**Explicação:** SQL Managed Instance oferece apenas General Purpose (armazenamento remoto, 5-10 ms; 3-5 ms em Next-gen GP) e Business Critical (SSD local, 1-2 ms). Hyperscale e Premium (DTU) são níveis Azure SQL Database, não níveis Managed Instance.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits)

---

### Q37. Pico sazonal: 24 vCores, > 50k IOPS nível

**Questão:**
Você está planejando uma solução que usará Azure SQL Database. O uso da solução atingirá o pico de 1 de outubro a 1 de janeiro de cada ano. Durante o uso de pico, o banco de dados exigirá o seguinte:
- 24 núcleos
- 500 GB de armazenagem
- 124 GB da memória
- Mais do que 50,000 IOPS

Durante períodos de uso fora do pico, o nível de serviço do Azure SQL Database será definido como Padrão. Qual nível de serviço você deve usar durante o uso máximo?

**Alternativas:**
- **A.** Business Critical
- **B.** Premium
- **C.** Hyperscale

**✅ Resposta:** A

**Explicação:** Aos 24 vCores (série padrão, cerca de 124 GB memória), Business Critical entrega 4,000 IOPS por vCore, aproximadamente 96,000 IOPS. General Purpose fornece apenas 320 IOPS por vCore. Hyperscale poderia atender os números, mas o banco de dados deve voltar ao padrão a cada ano: você pode reverter-migrar fora de Hyperscale apenas para General Purpose, e apenas dentro de 45 dias da migração original. Isso exclui uma scale-down. Premium é um DTU não é dimensionado em núcleos e memória. Business Critical pode ser escalado para cima e para baixo livremente.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale?view=azuresql) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/reverse-migrate-from-hyperscale?view=azuresql)

---

### Q38. Dois projetos: acesso OS vs CDC /auditing/reporting

**Questão:**
Você está trabalhando em dois projetos chamados Project1 e Project2 que migrarão um aplicativo on-premises para o Azure.

Project1 deve atender aos seguintes requisitos:
- Tenha acesso ao sistema operacional do servidor de banco de dados que hospeda o aplicativo.
- Tenha 25 TB de armazenamento.

Project2 deve atender aos seguintes requisitos:
- Tenha o change data capture (CDC) ativado.
- Nível de instância de suporte SQL auditing.
- Suporte gerenciado backups.
- Tenha 10 TB de armazenamento.
- Relatórios de descarga.

Você precisa recomendar uma solução Azure SQL para os projetos. A solução deve minimizar os custos. O que você deve incluir na recomendação para cada projeto?

**Alternativas:**

**Project1:**
- **A.** Serviço Azure Kubernetes (AKS)
- **B.** Azure SQL Database
- **C.** Azure SQL Managed Instance no nível de serviço Business Critical
- **D.** Azure SQL Managed Instance no nível de serviço General Purpose
- **E.** SQL Server on Azure Virtual Machines
Project2: (as mesmas cinco escolhas)

**✅ Resposta:** Project1: SQL Server on Azure Virtual Machines; Project2: Azure SQL Managed Instance no nível de serviço Business Critical

**Explicação:**
- Somente **Project1:** SQL Server on Azure VMs dá acesso ao sistema operacional do servidor de banco de dados, e um VM pode anexar 25 TB.
- **Project2, por que Managed Instance:** Nível de instância SQL Server Audit e CDC apontam para Managed Instance (Azure SQL Database não tem auditoria de nível de instância).
- **Project2, por que Business Critical:** "Relatórios de descarregamento" requer um secundário legível. Somente Business Critical inclui uma réplica read scale-out embutida; General Purpose não tem nenhuma, a menos que você pague por uma segunda instância em um failover group.
- **Project2, armazenamento:** Business Critical agora suporta 10 TB. A série premium otimizada para memória atinge 12-16 TB em 48 ou mais vCores e a série premium atinge 16 TB em 64 ou mais vCores nas principais regiões. Portanto, o Business Critical é a opção única mais barata que atende a todos os requisitos.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Baixa · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/features-comparison)

---

### Q39. Secure enclaves: vCore DC -series + Azure Atestado

**Questão:**
Você tem um banco de dados on-premises Microsoft SQL Server chamado DB1. Você tem uma assinatura do Azure. Você precisa migrar o DB1 para um serviço Azure SQL que atenda aos seguintes requisitos:
- Protege a confidencialidade de dados sensíveis contra malware e administradores de banco de dados não autorizados altamente privilegiados
- Suporta correspondência de padrões para operações de banco de dados do lado do servidor
- Usa uma tecnologia de criptografia baseada em hardware

Qual serviço Azure SQL e serviço de atestado você deve incluir na solução?

**Alternativas:**

**Serviço Azure SQL:**
- **A.** Azure SQL Managed Instance
- **B.** Modelo de compra Azure SQL Database DTU
- **C.** Modelo de compra Azure SQL Database vCore

**Serviço de atestado:**
- **A.** Serviço de Guardião Anfitrião (HGS)
- **B.** Microsoft Azure Attestation
- **C.** Sem atestado

**✅ Resposta:** Serviço Azure SQL: O modelo de compra Azure SQL Database vCore; Serviço de atestado: Microsoft Azure Attestation

**Explicação:** Always Encrypted with secure enclaves rich computs (padrão compatível com LIKE) em colunas criptografadas. O enclave baseado em hardware é Intel SGX, que está disponível apenas com o DC-configuração de hardware da série no vCore modelo de compra de Azure SQL Databasee, SGX atestado em Azure SQL Database É obrigatório e usa Microsoft Azure Attestation. VBS enclaves (DTU ou não-DC vCore) são baseados em software e não suportam nenhum atestado; HGS é para SQL Server on-premises/VMs. Managed Instance não oferece SGX Nota: A Microsoft anunciou o fim do suporte para a Intel SGX enclaves em 31 Oct 2027 (DCbancos de dados serão movidos para a série padrão com VBS). A resposta do exame permanece vCore + Azure Atestado.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/always-encrypted-enclaves-plan) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/always-encrypted-enclaves-configure-attestation)

---

### Q40. Single database autoscaling compute: service tier

**Questão:**
Você tem uma assinatura do Azure. Você planeja provisionar um único Azure SQL database. Você precisa garantir que o banco de dados suporte o escalonamento automático de recursos de computação. Qual nível de serviço você deve escolher?

**Alternativas:**
- **A.** Premium
- **B.** General Purpose
- **C.** Business Critical
- **D.** Padrão

**✅ Resposta:** B

**Explicação:** O escalonamento automático de computação é o nível de computação serverless, que existe apenas no modelo vCore para os níveis de serviço General Purpose (e Hyperscale). Premium e Standard são níveis DTU (sem serverless); Business Critical suporta apenas computação provisioned.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql)

---

### Q41. Plataforma de escolha para três projetos

**Questão:**
Você tem os projetos mostrados na tabela a seguir.

| Nome | Descrição |
|---|---|
| Project1 | Uma solução baseada em nuvem que integra várias soluções de software como serviço (SaaS) |
| Project2 | A migração de um banco de dados existente do on-premises Microsoft SQL Server 2019 para o Azure |
| Project3 | Um aplicativo que suporta vários bancos de dados 20-TB |

Você precisa recomendar uma solução de banco de dados do Azure para cada projeto. A solução deve atender aos seguintes requisitos:
- O Project2 deve minimizar as ameaças de malware ao sistema operacional subjacente que hospeda o banco de dados migrado.
- O esforço administrativo deve ser minimizado sempre que possível.

O que você deve recomendar para cada projeto?

**Alternativas:**

**Project1 / Project2 / Project3 (a mesma lista cada):**
- **A.** Azure SQL Managed Instance
- **B.** Azure SQL Database
- **C.** Azure Cosmos DB
- **D.** SQL Server em Azure VM

**✅ Resposta:** Project1: Azure Cosmos DB; Project2: Azure SQL Managed Instance; Project3: Azure SQL Database (Hyperscale)

**Explicação:** Project1 uma solução de iteração rápida que integra muitas fontes SaaS se encaixa em uma loja NoSQL totalmente gerenciada e flexível em termos de esquema (Cosmos DB). Project2 – um alvo PaaS remove o cliente-gerenciado OS (e sua exposição a malware); Managed Instance dá quase-100% SQL Server Compatibilidade 2019 para um elevador-e-deslocamento. Project3 – Azure SQL Database Hyperscale suporta bancos de dados até 128 TB e é PaaS (menor esforço de administração). Managed Instance tops para fora em 16 TB (32 TB por *instância* Apenas em Next-gen GP), então "vários 20-TB não se encaixa; o MI A própria página diz para considerar Hyperscale para maior armazenamento. SQL Server E-mail Azure VM trabalha, mas acrescenta OS/esforço de envio, violando "minimizar o esforço administrativo". A chave de origem (SQL VM) reflete a velha suposição de que Azure SQL Database não pode segurar 20 TB.

**Segunda opinião:** Disputado: Project1 não tem nenhuma exigência difícil. Cosmos DB (a chave de origem) se encaixa em um esquema em rápida mudança em muitos feeds SaaS, enquanto uma revisão independente Azure SQL Database para a DP-300 contexto. Project2 (MI) e Project3 (Azure SQL Database Hyperscale, até 128 TB) são firmes.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Baixa · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits?view=azuresql)

---

### Q42. Bancos de dados Web-app compartilhando computação, minimizar custo

**Questão:**
Você tem cinco servidores on-premises que têm o Microsoft SQL Server 2022 Enterprise instalado. Cada servidor contém vários bancos de dados que armazenam dados para aplicativos da web e são apoiados usando uma solução backup de terceiros. Você planeja migrar os bancos de dados para o Azure. Você precisa recomendar uma solução para hospedar os bancos de dados. A solução deve atender aos seguintes requisitos:
- Os recursos de computação e armazenamento devem ser compartilhados nas bancos de dados.
- Os custos devem ser minimizados.

O que você deve incluir na recomendação?

**Alternativas:**
- **A.** Azure SQL Managed Instance
- **B.** Azure SQL Database
- **C.** Azure SQL Database elastic pool
- **D.** SQL Server on Azure Virtual Machines

**✅ Resposta:** C

**Explicação:** An elastic pool é a maneira de compartilhar um conjunto de computação (vCores/eDTUs) e armazenamento entre muitos bancos de dados com uso variável – típico para bancos de dados de aplicativos web / SaaS. Bancos de dados únicos (B) não compartilham recursos; Managed Instance a) Partilha recursos da instância, mas custa mais; SQL Server E-mail VMs (D) adiciona VM e custos de licença e esforço administrativo. O terceiro backup ferramenta é uma descrição do estado atual, não um requisito para mantê-lo.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-pool-overview?view=azuresql) [Doc 2](https://www.pass4success.com/microsoft/discussions/exam-dp-300-topic-4-question-132-discussion))

---

### Q43. Criar tabela particionada: sequência de objetos

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1. Você precisa criar uma tabela particionada em DB1. Quais três objetos você deve criar em sequência? Para responder, mova os objetos apropriados da lista de objetos para a área de resposta e organize-os na ordem correta.

**Alternativas:**

**Objetos:**
- **A.** partition scheme
- **B.** um índice alinhado
- **C.** um grupo de arquivos
- **D.** uma tabela
- **E.** partition function

**✅ Resposta:** 1. a partition function → 2. a partition scheme → 3. uma tabela

**Explicação:** O particionamento de uma tabela leva três passos: criar o partition function (valores limite), criar o partition scheme que mapeia partições para grupos de arquivos (em Azure SQL Database apenas `ALL TO ([PRIMARY])` é possível), em seguida, criar a tabela ON o partition scheme com a coluna de particionamento. Um grupo de arquivos não pode ser adicionado em Azure SQL Database, e um índice alinhado é opcional - não é o que faz "uma tabela dividida"; o objetivo.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/partitions/create-partitioned-tables-and-indexes)

---

### Q44. ADatum: análise de descarga para DB1 / DB2

**Estudo de caso:** ADatum v1 (serviços financeiros)

**Questão:**
(Estudo de caso: DB1 e DB2 em SVR1 – SQL Server 2016 Enterprise em AG1 – são migrados para um Azure SQL managed instance chamado Instance1. Exigência de disponibilidade: "Para DB1 e DB2, descarregue cargas de trabalho analíticas para uma réplica de banco de dados read-only na mesma região do Azure." Requisitos de negócios: minimizar custos e esforço administrativo.)
Você precisa recomendar um nível de serviço e um método para descarregar cargas de trabalho analíticas para os bancos de dados migrados do SVR1. A solução deve atender aos requisitos de disponibilidade e negócios. O que você deve recomendar?

**Alternativas:**

**Caixa 1 – Nível de serviço:**
- **A.** Business Critical
- **B.** General Purpose
- **C.** Premium

**Caixa 2 – Método:**
- **A.** A failover group read-only listener
- **B.** réplicas secundárias geo-replicadas
- **C.** Read scale-out

**✅ Resposta:** Caixa 1: Business Critical; Caixa 2: Read scale-out

**Explicação:** DB1/DB2 ir a Instance1, a SQL Managed Instance. Managed Instance tem apenas General Purpose (incl. Next-gen GP) e Business Critical tiers – “Premium” é um Azure SQL Database DTU não existe para MI. Apenas Business Critical inclui um secundário legível embutido (read scale-out, incluído no preço) na mesma região; General Purpose não tem réplica legível. A failover- grupo listener e geo-replicas colocar a cópia legível em outra região e custo extra.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits?view=azuresql) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/read-scale-out)

---

### Q45. Contoso: Nível de computação do protótipo PaaS

**Estudo de caso:** Contoso

**Questão:**
(Estudo de caso: A Contoso deve minimizar os custos de computação das soluções PaaS. A exposição de utilização de computação do protótipo PaaS (últimos 7 dias, Max) mostra o uso próximo ao 0% na maior parte do tempo com picos curtos e intermitentes cerca de uma vez por dia.)
Com base no protótipo PaaS, qual camada de computação Azure SQL Database você deve usar?

**Alternativas:**
- **A.** Business Critical 4-vCore
- **B.** Hyperscale
- **C.** General Purpose v- vCore
- **D.** Serverless

**✅ Resposta:** D

**Explicação:** Períodos ociosos longos com picos curtos e intermitentes são o padrão serverless do livro-texto: escalas automáticas de computação entre min / max vCores, é cobrado por segundo e (em General Purpose) pode auto-pause durante períodos ociosos para que apenas o armazenamento seja faturado - reunião "minimizar os custos de computação". Provisioned Business Critical, Hyperscale ou General Purpose conta para a computação provisioned completa 24x7.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql)

---

### Q46. Auto-escala + faturamento por segundo: serverless GP

**Questão:**
Você tem um servidor Microsoft SQL Server 2017 que hospeda cinco bancos de dados. Você planeja migrar os bancos de dados para o Azure. Você precisa recomendar uma solução que atenda aos seguintes requisitos:
- Escala automaticamente a computação com base na demanda de carga de trabalho
- Fornece faturamento por segundo

O que você deve incluir na recomendação?

**Alternativas:**

**Serviço Azure:**
- **A.** Um único Azure SQL database no nível de computação provisioned
- **B.** Um único Azure SQL database no nível de computação serverless
- **C.** Azure SQL Database elastic pool
- **D.** Azure SQL Managed Instance

**Nível de serviço:**
- **A.** Básico
- **B.** General Purpose
- **C.** Padrão

**✅ Resposta:** Serviço Azure: Um único Azure SQL database no nível de computação serverless; Nível de serviço: General Purpose

**Explicação:** Serverless escala automaticamente a computação com a demanda e as contas calculam por segundo. É uma opção de modelo vCore disponível em General Purpose (e Hyperscale); Básico e padrão são camadas DTU que não têm opção serverless. Provisioned computar, elastic pools e Managed Instance são cobrados por hora para recursos fixos provisioned.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql)

---

### Q47. Escalabilidade dinâmica CPU + pausa: serverless

**Questão:**
Você tem uma assinatura do Azure. Você precisa implantar um Azure SQL database. A solução deve atender aos seguintes requisitos:
- Expanda dinamicamente os recursos CPU.
- Certifique-se de que o banco de dados pode ser pausado para reduzir os custos.

O que você deve usar?

**Alternativas:**
- **A.** o nível de serviço Business Critical
- **B.** o nível de computação serverless
- **C.** elastic pool
- **D.** o nível de serviço General Purpose

**✅ Resposta:** B

**Explicação:** O serverless Auto-escalas de camada de computação vCores dentro do intervalo min/max e suporta auto-pause (General Purpose serverless), durante o qual apenas o armazenamento é cobrado. A General Purpose provisioned banco de dados, Business Criticalou um elastic pool Não pode ser pausado.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql)

---

### Q48. Hyperscale: adicionar read-only replica, ApplicationIntent

**Questão:**
Você tem uma assinatura do Azure que contém os recursos mostrados na tabela a seguir.

| Nome | Tipo | Configuração |
|---|---|---|
| DB1 | Azure SQL Database | Nível de serviço Hyperscale; Sem réplicas secundárias |
| App1 | Azure Web Apps | App1 tem acesso read-only a DB1. Há várias instâncias de App1. |

Você precisa criar um read-only replica de DB1 e configurar as instâncias App1 para usar a réplica. O que você deve fazer?

**Alternativas:**

**Caixa 1 – Para adicionar read-only replicas de DB1:**
- **A.** Criar uma réplica no mesmo servidor lógico
- **B.** Criar um novo servidor lógico e configurar geo-replication
- **C.** Criar um novo servidor lógico e configurar um auto-failover group

**2 – Para configurar instâncias App1 para acessar o read-only replica:**
- **A.** Adicionar uma entrada ApplicationIntent ao connection string
- **B.** Adicionar uma entrada MultiSubnetFailover ao App1 connection string
- **C.** Criar um ponto final dedicado e configurar o App1 connection string para apontar para o ponto final

**✅ Resposta:** Caixa 1: Criar uma réplica no mesmo servidor lógico; Caixa 2: Adicionar uma entrada ApplicationIntent ao connection string

**Explicação:** Hyperscale alta disponibilidade (HAAs réplicas secundárias compartilham os servidores de página do primário, usam o mesmo servidor lógico e nome do banco de dados, e são adicionadas apenas definindo a HA Contagem (0-4) – sem novo servidor ou cópia de dados. Read-only As conexões são encaminhadas para eles por `ApplicationIntent=ReadOnly` na connection string. Geo-replication/failover groups precisa de um segundo servidor lógico e uma cópia completa dos dados (overkill for read scale-out). MultiSubnetFailover é para multi-subnet AG listeners; não é necessário nenhum "endpoint dedicado".

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale-replicas?view=azuresql)

---

### Q49. Exposição Serverless: retomar o tempo e padrão de uso

**Questão:**
Você está provisionando um Azure SQL database no Azure portal como mostrado na exposição a seguir.

(Criar exposição SQL Database > Configurar): Configuração do hardware Gen5 (até 40 vCores, 120 GB memória); **Max vCores: 6**; **Min vCores: 0.75** (2.25 GB min / 18 GB memória max); Auto-pause atraso: **Caixa de seleção "Ativar auto-pause" UNCHECKED** (Dias 0, Horas 4, Minutos 0); Dados tamanho máximo 800 GB (240 GB espaço de registro alocado). Resumo do custo: General Purpose serverless (GP_S_Gen5), custo por GB 0.12 USD, custo de armazenamento estimado/mês 119.60 USD, custo de computação/vCore/segundo 0.000145 USD; "Serverless bancos de dados são cobrados em vCores baseado em uma combinação de CPU e utilização da memória”.

Use os menus suspensos para selecionar a opção de resposta que completa cada declaração com base nas informações apresentadas no gráfico.

**Alternativas:**

**Declaração 1: "Após quatro horas de inatividade, o banco de dados requer [escolha de resposta] para retomar as operações para novas atividades.":**
- **A.** sem tempo extra
- **B.** até 10 minutos
- **C.** até um minuto

**Declaração 2: "A configuração do banco de dados reduz o custo dos padrões de uso [escolha de resposta].":**
- **A.** intermitente e imprevisível
- **B.** regular e alta
- **C.** estável e baixo

**✅ Resposta:** Declaração 1: sem tempo extra; Declaração 2: intermitente e imprevisível

**Explicação:** O Auto-pause está desativado (caixa de seleção desmarcada), então o banco de dados nunca faz pausas e não há atraso no currículo - o valor de 4 horas é ignorado. (Se o auto-pause tivesse sido ativado, o currículo levaria até cerca de um minuto.) O Serverless é otimizado para uso intermitente e imprevisível com baixa utilização média; o uso regular / alto ou constante é melhor servido pela computação provisioned.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql)

---

### Q50. SIM: db1 serverless 1–4 vCores, 45 GB

**Questão:**
SIMULATION. Você precisa garantir que o db1 suporte um mínimo de um vCore, um máximo de quatro vCores e uma banco de dados de até 45 GB. Para concluir esta tarefa, entre na máquina virtual. Você pode precisar usar SQL Server Management Studio e o Azure portal.

**Alternativas:**

**✅ Resposta:** Configurar db1 como a **General Purpose serverless** Banco de Dados com Min vCores = 1, Max vCores = 4, Data max size = 45 GB.
Passos do Portal:
1. Azure portal → SQL bancos de dados → **db1** → **Configurações > Compute + armazenamento** (ou Visão geral → Ligação de nível de preços).
2. Nível de serviço: **General Purpose** (modelo vCore); Nível de computação: **Serverless**; Hardware: série padrão (Gen5).
3. Set **Max vCores = 4** e **Min vCores = 1**.
4. Set **Data max size = 45 GB**. (Leave auto-pause as is - não faz parte da tarefa.)
5. Selecione **Aplicar** e aguarde até que a operação de escala termine.

Alternativa PowerShell:
```powershell
Set-AzSqlDatabase -ResourceGroupName <rg> -ServerName <server> -DatabaseName db1 `
  -Edition GeneralPurpose -ComputeModel Serverless -ComputeGeneration Gen5 `
  -MinVcore 1 -MaxVcore 4 -MaxSizeBytes 48318382080   # 45 GB
```

**Explicação:** Um mínimo *e* Máximo vCore Existe apenas no intervalo serverless camada de computação (provisioned A computação tem um fixo vCore Contagem). T-SQL `ALTER DATABASE ... MODIFY (SERVICE_OBJECTIVE = 'GP_S_Gen5_4', MAXSIZE = 45 GB)` conjuntos max vCores e tamanho, mas aplica o min padrão vCoresO portal/PowerShell/CLI é necessário definir min vCores = 1. If db1 está em um elastic pooldeve ser movido para fora primeiro (serverless é apenas para bancos de dados únicas).

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-create-configure?view=azuresql) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql)

---

### Q51. SIM: db1 auto-pause após uma hora

**Questão:**
SIMULATION. Você precisa configurar o db1 para pausar automaticamente após uma hora de inatividade. Para concluir essa tarefa, entre na máquina virtual. Você pode precisar usar o SQL Server Management Studio e o Azure portal.

**Alternativas:**

**✅ Resposta:** Faça db1 um banco de dados único **General Purpose serverless** com **auto-pause enabled, delay = 1 hour (60 minutes)**.
Passos do Portal:
1. Azure portal → SQL bancos de dados → **db1** → **Configurações > Compute + armazenamento**.
2. (Se db1 estiver em um elastic pool, defina **Elastic pool = No / None** primeiro – serverless se aplica apenas a bancos de dados únicos.)
3. Nível de serviço: **General Purpose**; Nível de computação: **Serverless**.
4. Em **atraso Auto-pause**, selecione **Ativar auto-pause** e defina **1 hora** (Dias 0, Horas 1, Minutos 0).
5. Selecione **Aplicar**.

Alternativa PowerShell:
```powershell
Set-AzSqlDatabase -ResourceGroupName <rg> -ServerName <server> -DatabaseName db1 `
  -Edition GeneralPurpose -ComputeModel Serverless -ComputeGeneration Gen5 `
  -MaxVcore <n> -AutoPauseDelayInMinutes 60
```

**Explicação:** Somente os bancos de dados serverless podem auto-pause (General Purpose GA; Hyperscale pré-visualização). O atraso auto-pause é definido em minutos (GP: 15 min - 7 dias; -1 desabilita); 60 minutos atende ao requisito. Elastic pools não tem nível de computação serverless, então mudar o "nível de computação" de uma piscina (como a fonte descreve) não é um método válido.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-create-configure?view=azuresql)

---

### Q52. Tamanho elastic pool: três métricas

**Questão:**
Você tem 20 Azure SQL databases provisioned usando o modelo de compra vCore. Você planeja criar um Azure SQL Database elastic pool e adicionar as 20 bancos de dados. Quais três métricas você deve usar para dimensionar o elastic pool para atender às demandas de sua carga de trabalho? Cada resposta correta apresenta parte da solução. NOTE: Cada seleção correta vale um ponto.

**Alternativas:**
- **A.** tamanho total de todas Os bancos de dados
- **B.** Suporte geo-replication
- **C.** número de bancos de dados de pico simultâneo * utilização máxima CPU por banco de dados
- **D.** número máximo de sessões simultâneas para todas Os bancos de dados
- **E.** número total de bancos de dados * utilização média CPU por banco de dados

**✅ Resposta:** A, C, E

**Explicação:** O método de dimensionamento de piscina da Microsoft: vCores = MAX(número total de DBs × Média vCore utilização por DB, número de picos simultâneos DBs × pico vCore utilização por DB), and storage = sum of the data size of all databases in the pool. Geo-replication As sessões de suporte e simultâneas não estão dimensionando entradas.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-pool-overview?view=azuresql)

---

### Q53. SQL VM: 4- TB volume com máximo IOPS

**Questão:**
Você tem SQL Server em um Azure virtual machine. Você precisa adicionar um volume 4- TB que atenda aos seguintes requisitos:
- Maximiza IOPs
- Usa unidades de estado sólido premium (SSDs)

O que você deve fazer?

**Alternativas:**
- **A.** Anexe dois espelhos 4- TB SSDs.
- **B.** Anexe um conjunto de listras que contém quatro 1-TB SSDs.
- **C.** Anexe uma matriz RAID-5 que contém cinco 1-TB SSDs.
- **D.** Anexe um único 4- TB SSD.

**✅ Resposta:** B

**Explicação:** Premium SSD IOPS com o tamanho do disco por disco, e o striping (Storage Spaces) soma a IOPS de cada disco: quatro 1-TiB P30 Os discos dão cerca de 4 × 5,000 = 20,000 IOPS, versus 7,500 para um único 4-TiB P50. P30s também apoio read-only cache de host (discos de 4 TiB ou mais não). Espelhamento dá não IOPS ganho para escrita e metade da capacidade; RAID-5 tem uma pena de paridade de escrita e não é um layout Azure recomendado.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage?view=azuresql)

---

### Q54. Modelo ARM: SQL VM cache de disco

**Questão:**
Você precisa usar um modelo Azure Resource Manager (ARM) para implantar um Azure virtual machine que hospedará uma instância Microsoft SQL Server. A solução deve maximizar o desempenho de E/S do disco para o banco de dados e arquivos de log SQL Server. Como você deve concluir o modelo?

```json
"variables": {
  "dataDisks": {
    "caching": [Box 1], "dataDiskCount": 8, "logDisksCount": 1,
  ...
},
"resources": [
  ...
  { "type": "Microsoft.Compute/virtualMachines",
    ...
    "storageProfile": {
      "osDisk": { ... },
      "copy": [
        { "name": "dataDisks",
          "count": "[add(variables('dataDiskCount'), variables('logDisksCount'))]",
          "input": { "lun": "[copyIndex('dataDisks')]", "createOption": "empty",
            "caching": "[if(greaterOrEquals(copyIndex('dataDisks'), parameters('dataDiskCount')), [Box 2], variables('dataDisks').caching )]",
            "diskSizeGB": 1023,
  ...
```

**Alternativas:**

**Caixa 1:**
- **A.** Nenhum
- **B.** ReadOnly
- **C.** ReadWrite

**Caixa 2:**
- **A.** Nenhum
- **B.** ReadOnly
- **C.** ReadWrite

**✅ Resposta:** Caixa 1: ReadOnly; Caixa 2: Nenhum

**Explicação:** O copy loop cria 8 discos de dados (LUN 0–7) e 1 disco de log (LUN 8). `if()` retorna a Caixa 2 quando `copyIndex >= dataDiskCount`, ou seja, apenas para o disco de log. Para os discos de dados, usa `variables('dataDisks').caching` (Caixa 1). A orientação da Microsoft é usar ReadOnly para host caching de discos de dados e None para discos de transaction log. ReadWrite não se aplica a discos de dados ou de log do SQL. Assim, Caixa 1 (dados) = ReadOnly e Caixa 2 (log) = None.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage?view=azuresql)

---

### Q55. Dois discos P30 para 10,000 IOPS: sequência de objeto

**Questão:**
Você está construindo um Azure virtual machine. Você aloca dois discos de armazenamento premium 1-TiB, P30 para a máquina virtual. Cada disco fornece 5,000 IOPS. Você planeja migrar uma instância on-premises do Microsoft SQL Server para a máquina virtual. A instância tem um banco de dados que contém um arquivo de dados 1.2 -TiB. O banco de dados requer 10,000 IOPS. Você precisa configurar a partir da área de resposta virtual.

**Alternativas:**
- **A.** um disco virtual que usa o layout da faixa
- **B.** um disco virtual que usa o layout do espelho
- **C.** um volume
- **D.** um disco virtual que usa o layout simples
- **E.** uma piscina de armazenamento

**✅ Resposta:** 1. um pool de armazenamento → 2. um disco virtual que usa o layout simples → 3.

**Explicação:** A ordem (piscina de armazenamento → disco virtual → é certo. Os dados devem ser listrados em ambos os discos para obter 2 TiB de capacidade (> 1.2 TiB) e 2 × 5,000 = 10,000 IOPS; espelho dá apenas 1 TiB e não escrever-IOPS Em espaços de armazenamento do Windows, o layout de listras é nomeado **Simples simples** ("stripes dados através de discos físicos, maximiza a capacidade e aumenta o rendimento, sem resiliência"); os únicos layouts são Simples, Espelho e Paridade (`New-VirtualDisk -ResiliencySettingName Simple -NumberOfColumns 2`). Não há nenhum layout chamado "stripe" em Espaços de Armazenamento, então "layout simples" é a escolha tecnicamente exata. Caveata: a chave clássica do dump escolhe "layout de risca", lendo-a como listras em inglês simples; ambos descrevem o mesmo RAID-0-estilo de striping, para que a chave marcada do exame possa seguir o despejo.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Baixa · [Doc 1](https://learn.microsoft.com/en-us/windows-server/storage/storage-spaces/deploy-standalone-storage-spaces) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage?view=azuresql)

---

### Q56. Anexar disco ultra ao SQL VM rebentável

**Questão:**
Você tem um Azure virtual machine rebentável chamado VM1 que hospeda uma instância do Microsoft SQL Server. Você precisa anexar um disco ultra Azure ao VM1. A solução deve minimizar o tempo de inatividade no VM1. Em qual ordem você deve executar as ações? Para responder, mova todas as ações da lista de ações para a área de resposta e organize-as na ordem correta.

**Alternativas:**

**Acções:**
- **A.** Redimensionar VM1.
- **B.** Ativar compatibilidade de disco Ultra para Sim.
- **C.** Pare e deallocate VM1.
- **D.** Anexe o disco ultra.
- **E.** Start VM1.

**✅ Resposta:** 1. Parar e deallocate VM1 → 2. redimensionar VM1 → 3. Definir Habilitar compatibilidade de disco Ultra para Sim → 4. Iniciar VM1 → 5. Anexe o disco ultra

**Explicação:** Os tamanhos clássicos da série B rebentáveis não estão na lista de tamanhos compatíveis com ultra-disco, então VM1 Fazer ambas as mudanças em uma deallocation: redimensionar primeiro (a compatibilidade não pode ser habilitada em um tamanho não suportado), então ativar a compatibilidade do disco Ultra, que requer o VM a ser parado/desalocado. Microsoft CLI/PowerShell "Processo é desvendado → permitir compatibilidade → Início → ": uma vez que a compatibilidade está ativada, o disco ultra pode ser anexado à corrente VM, então anexar depois do início mantém o tempo de inatividade o mais curto possível. (O portal passa a se anexa enquanto ainda deallocou e, em seguida, inicia o VM. Isso também funciona e é a ordem do despejo, mas mantém VM1 offline por mais tempo.)

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/virtual-machines/disks-enable-ultra-ssd)

---

### Q57. Imagem personalizada SQL VM: patches automatizados e backups

**Questão:**
Você tem um Azure virtual machine baseado em uma imagem personalizada chamada VM1. VM1 hospeda uma instância do Microsoft SQL Server 2019 Standard. Você precisa automatizar a manutenção do VM1 para atender aos seguintes requisitos:
- Automatize a correção de SQL Server e Windows Server.
- Automatize a banco de dados completa backups e transaction log backups das bancos de dados em VM1.
- Minimizar o esforço administrativo.

O que você deve fazer primeiro?

**Alternativas:**
- **A.** Habilitar um system-assigned managed identity para VM1
- **B.** Registre a assinatura do Azure para o provedor de recursos Microsoft.Sql
- **C.** Instale uma extensão Azure virtual machine Desired State Configuration (DSC) em VM1
- **D.** Registre a assinatura do Azure para o provedor de recursos Microsoft.SqlVirtualMachine

**✅ Resposta:** D

**Explicação:** Automated Patching e Automated Backup são características do SQL Server IaaS Agent extensionA VM construído a partir de uma imagem personalizada (auto-instalado SQL Server) não é registrado automaticamente, então você deve registrá-lo com a extensão. O primeiro passo é registrar a assinatura com o Microsoft.SqlVirtualMachine fornecedor de recursos; o SQL VM é então registrado, criando SQL recurso de máquina virtual. Microsoft.Sql é o provedor para Azure SQL Database/MI, não SQL E-mail VMs. Nem a managed identity nem DSC fornece essas características.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://docs.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/sql-vm-resource-provider-register) [Doc 2](https://learn.microsoft.com/en-au/azure/azure-sql/virtual-machines/windows/sql-agent-extension-manually-register-single-vm?view=azuresql)

---

### Q58. Disco SQL VM: personalizado IOPS / throughput a um custo mínimo

**Questão:**
Você tem uma assinatura do Azure. Você precisa implantar uma instância de SQL Server on Azure Virtual Machines. A solução deve atender aos seguintes requisitos:
- A configuração de desempenho personalizado, como IOPS, capacidade e taxa de transferência, deve ser suportada.
- Os custos devem ser minimizados.

Que tipo de disco você deve incluir na solução?

**Alternativas:**
- **A.** Premium SSD v2
- **B.** Premium SSD
- **C.** Padrão SSD
- **D.** Ultra SSD

**✅ Resposta:** A

**Explicação:** Apenas Premium SSD v2 e Ultra Disk deixá-lo definir a capacidade, IOPS e throughput independentemente. Dos dois, Premium SSD v2 é a opção de baixo custo (base 3,000 IOPS / 125 MBps e, em seguida, pagar por extra), e a Microsoft recomenda que para SQL Server Ultra Disk é a escolha de preço premium para extrema IOPS ou latência sub-milissegundo. Premium SSD e padrão SSD ter níveis de desempenho fixos ligados ao tamanho do disco.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage?view=azuresql) [Doc 2](https://learn.microsoft.com/en-us/azure//virtual-machines/disks-types)

---

### Q59. Tipos de disco SQL VM: TempDB e logs

**Questão:**
Você tem uma assinatura do Azure. Você planeja implantar um novo Azure virtual machine que hospedará uma instância do Microsoft SQL Server. Você precisa configurar os discos na máquina virtual. A solução deve atender aos seguintes requisitos:
- Minimize a latência para registros de transações.
- Minimize o impacto na taxa de transferência IO da máquina virtual. *(Paginação MD: "Minimizar VM permitido IO.")*

Cada tipo de disco pode ser usado uma vez, mais de uma vez, ou não.

**Alternativas:**

**Tipos de disco:**
- **A.** Local
- **B.** Premium SSD
- **C.** Padrão HDD
- **D.** Padrão SSD
- **E.** Disco Ultra
Área de resposta: TempDB: _____; Registros de transação: ____

**✅ Resposta:** TempDB: **Local** (efêmero local SSD, D:\); Transaction logs: **Disco Ultra**

**Explicação:** O VM's uncached (disco remoto) IOPS"Aplica-se apenas a E/S contra unidades de dados remotamente ligadas ao VM, not the local I/O against the temp drive (D:\)", so putting tempdb on the local SSD O Tempdbb I/O não usa o VMO orçamento de E/S do disco remoto. A Microsoft também recomenda o Tempdb no disco temp local para a maioria das cargas de trabalho. Ultra Disk (ou Premium SSD v2) é a escolha para o log quando você precisa de latência sub-milissegundos. SSD para tempdb contaria contra o VMA tampa de E/S não em cache, que é o oposto da segunda exigência.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage)

---

### Q60. Download da edição SQL VM: desinstalar primeiro

**Questão:**
Você tem uma instância SQL Server on Azure Virtual Machines chamada SQLVM1 que foi implantada usando uma imagem do Azure Marketplace SQL Server 2019 Enterprise. Você precisa mudar a instância Microsoft SQL Server no SQLVM1 para a edição Standard. A solução deve garantir a conformidade com o licenciamento. O que você deve fazer primeiro?

**Alternativas:**
- **A.** A partir do Centro de Instalação SQL Server no SQLVM1, execute o assistente de atualização da edição.
- **B.** De SQLVM1, desinstale a instância SQL Server.
- **C.** A partir do Centro de Instalação SQL Server no SQLVM1, execute o assistente de reparo.
- **D.** A partir do Azure portal, reconfigure SQLVM1.

**✅ Resposta:** B

**Explicação:** Ir da Enterprise para a Standard é um downgrade. O procedimento da Microsoft é fazer backup de todas Os bancos de dados, desinstalar completamente SQL Server, reinstalá-lo a partir da mídia padrão e, em seguida, atualizar o SQL VM A propriedade de edição no portal para faturamento. Edição Upgrade (A) só vai para cima. Reparar (C) não muda a edição. Alterar a propriedade de edição do portal (D) é o último passo e só é válido uma vez que a edição instalada corresponda. Antes de desinstalar, você também faria backup das bancos de dados e remover SQL IaaS Agent extension, mas também não é oferecido como uma opção.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/change-sql-server-edition)

---

### Q61. Instalar SQL Server 2022 em RHEL VM

**Questão:**
Você tem um Azure virtual machine chamado VM1 que executa Red Hat Enterprise Linux (RHEL). Você precisa instalar e configurar o Microsoft SQL Server 2022 Standard em VM1. Quais três ações você deve executar em sequência?

**Alternativas:**
Acções:
- **A.** Run sudo yum localinstall mssql-server_2022.x86_64.rpm.
- **B.** Reinicie o mssql-server.
- **C.** Configuração do sudo /opt/mssql/bin/mssqls-conf.
- **D.** Baixe o arquivo mssql-server-2022.repo.
- **E.** Rundo yum install -y mssql-server.

**✅ Resposta:** 1 → Baixar o mssql-server-2022.repo arquivo; 2 → Correr `sudo yum install -y mssql-server`; 3 → Correr `sudo /opt/mssql/bin/mssql-conf setup`

**Explicação:** Esta é a ordem de início rápido do RHEL. Primeiro faça o download da configuração do repositório (`curl ... mssql-server-2022.repo` em /etc/ yum.repos.d), depois instale o pacote com yum, depois execute a configuração mssql-conf para definir a senha sa e escolha a edição (Padrão). `yum localinstall` de um.rpm é o método offline e não é necessário uma vez que o repo está configurado. Uma reinicialização não é necessária porque msssql-con inicia o serviço de configuração.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-red-hat)

---

### Q62. Linux SQL IaaS Agent extension: OS e benefício

**Questão:**
Você planeja implantar uma instância do SQL Server no Linux Azure Virtual Machines. A instância executará o Microsoft SQL Server 2022 e usará o SQL Server IaaS Agent extension para Linux. Qual sistema operacional Linux você deve implantar e qual benefício o SQL Server IaaS Agent extension fornecerá?

**Alternativas:**

**Sistema operacional:**
- **A.** Red Hat Enterprise Linux (RHEL)
- **B.** SUSE Linux Enterprise Server (SLES)
- **C.** Ubuntu Linux

**Benefícios:**
- **A.** automático backup
- **B.** Automatic patching
- **C.** Registro de um recurso de máquina virtual SQL no Azure

**✅ Resposta:** Sistema operacional: **Ubuntu Linux**; Benefício: **Registro de um recurso de máquina virtual SQL no Azure**

**Explicação:** Quando esta questão foi escrita, a limitação documentada da extensão Linux era "apenas SQL Server VMs no Ubuntu Linux são suportados; outras distribuições Linux não são." No Linux, a extensão é executada apenas no modo leve, que oferece o tipo de licença e gerenciamento de edição além do registro (cria o separado *Máquina virtual SQL* recursos). Automated backup e automated patching Nota: as páginas atuais do Learn (atualizadas em setembro de 2026) não listam mais uma limitação de distribuição, e a Red Hat anunciou RHEL O Ubuntu é a resposta que a chave do exame espera.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/linux/sql-server-iaas-agent-extension-linux) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/linux/sql-iaas-agent-extension-register-vm-linux)

---

### Q63. Gerencie 10 SQL VMs com uma conta

**Questão:**
Você tem uma assinatura do Azure vinculada a um locatário Microsoft Entra (CQ: " locatário Azure AD"). A assinatura contém 10 máquinas virtuais que executam Windows Server 2019 e hospedam instâncias Microsoft SQL Server 2019. Você precisa garantir que você possa gerenciar as instâncias SQL Server usando uma única conta de usuário. O que você deve fazer primeiro?

**Alternativas:**
- **A.** Habilite um user-assigned managed identity em cada máquina virtual.
- **B.** Implantar um domínio Microsoft Entra Domain Services (Azure AD DS) e juntar as máquinas virtuais para o domínio.
- **C.** Habilite um system-assigned managed identity em cada máquina virtual.
- **D.** Junte-se às máquinas virtuais para o inquilino Microsoft Entra.

**✅ Resposta:** B

**Explicação:** Microsoft Entra authentication E SQL Server necessidades SQL Server 2022 ou mais tarde. SQL Server 2019 suporta apenas Windows (AD) e SQL authentication. Para usar uma conta em todas as 10 instâncias, o VMs devem ser unidos a um AD domínio. Microsoft Entra Os Serviços de Domínios fornecem um domínio gerenciado cujas contas são sincronizadas do inquilino, para que uma conta de domínio possa ser concedida a um Windows. login em todos os casos. Managed identities c) identificar o VM não um administrador. Juntando-se ao VMs d) não dá SQL Server 2019 um principal de domínio do Windows que ele pode usar.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/sql/relational-databases/security/authentication-access/azure-ad-authentication-sql-server-overview)

---

### Q64. Série SQL VM: IO e rácios de memória

**Questão:**
Você tem uma assinatura do Azure. Você precisa implantar duas máquinas virtuais chamadas VM1 e VM2 que hospedarão instâncias Microsoft SQL Server. A solução deve atender aos seguintes requisitos:
- VM1 deve fornecer a maior taxa de transferência de E/S para vCore.
- VM2 deve fornecer a maior relação memória-para-vCore.

Qual série você deve selecionar para cada máquina virtual?

**Alternativas:**

**VM1:**
- **A.** DCadsv5-series
- **B.** Ebdsv5-series
- **C.** Mv2-series

**VM2:**
- **A.** DCadsv5-series
- **B.** Ebdsv5-series
- **C.** Mv2-series

**✅ Resposta:** VM1: **Ebdsv5-series**; VM2: **Mv2-series**

**Explicação:** A orientação de dimensionamento de VM da Microsoft indica Ebdsv5 para SQL Server com alto throughput de I/O por vCore, cerca de 89 MB/s por vCore, ao lado da série Mbdsv3, que não aparece nas alternativas. A família da série M (Mv2 neste caso, com até cerca de 12 TB) oferece as maiores proporções de memória por vCore, bem acima dos 8:1 da Ebdsv5. DCadsv5 é uma série de confidential computing de uso geral, com uma proporção de 4:1 e baixo throughput de armazenamento. Por isso, não atende a nenhum dos dois requisitos.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-vm-size)

---

### Q65. Acelerador de escrita requer série M VM

**Questão:**
Você tem uma assinatura do Azure. Você planeja implantar uma instância do SQL Server on Azure Virtual Machines que suporte o Acelerador de Escrita. Qual série de máquinas virtuais você deve usar?

**Alternativas:**
- **A.** E-series
- **B.** série G
- **C.** H-série
- **D.** Série M

**✅ Resposta:** D

**Explicação:** "Write Accelerator é um recurso de disco que está disponível apenas para o M-Series VMs". É usado na unidade de registro de transações (Premium Storage) para latência de gravação muito baixa. As séries E, G e H não suportam isso.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage)

---

<a name="d2--implement-a-secure-environment-2025"></a>

## D2 — Implementar um ambiente seguro (20–25%)

*65 perguntas*

### Q66. Quatro clientes DBs: isolar o acesso por banco de dados

**Questão:**
Você tem uma assinatura do Azure. Você cria um SQL server lógico que hospeda quatro bancos de dados. Cada banco de dados será usado por um cliente separado. Você precisa garantir que cada cliente possa acessar apenas seu próprio banco de dados. A solução deve minimizar o esforço administrativo. Quais duas ações você deve executar? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** Criar um network security group (NSG).
- **B.** Crie uma regra de firewall no nível do servidor.
- **C.** Crie um private endpoint.
- **D.** Crie uma regra de firewall em nível de banco de dados.
- **E.** Negar acesso público.

**✅ Resposta:** C, D (criar um private endpoint; criar uma regra de firewall de nível de banco de dados)

**Explicação:** As regras de firewall de firewall de nível de banco de dados do IP são armazenadas e aplicadas a um único banco de dados. A Microsoft diz para usá-las quando "usuários de um banco de dados também devem ser totalmente isolados de outro banco de dados completo", enquanto as regras de nível de servidor (B) abrem todos os bancos de dados no servidor. O private endpoint oferece aos clientes conectividade privada ao servidor, que é a segunda parte da chave publicada. Um NSG (A) não pode filtrar por banco de dados, e negar acesso público (E).

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure)

---

### Q67. cliente 40 DBs: usuários + firewall de banco de dados

**Questão:**
Você tem 40 Azure SQL databases, cada um para um cliente diferente. Todas Os bancos de dados residem no mesmo servidor Azure SQL Database. Você precisa garantir que cada cliente só pode se conectar e acessar seu respectivo banco de dados. Quais duas ações você deve executar? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** Implemente o row-level security (RLS).
- **B.** Crie usuários em cada banco de dados.
- **C.** Configure o firewall do banco de dados.
- **D.** Configure o firewall do servidor.
- **E.** Crie logins no banco de dados mestre.
- **F.** Implemente o Always Encrypted.

**✅ Resposta:** B, C

**Explicação:** Os usuários contidos criados no banco de dados de cada cliente (sem servidor login) autenticar-se diretamente a esse banco de dados e pode se conectar apenas a ele. IP As regras de firewall permitem que um cliente IP As regras de firewall do servidor (D) abrem todos os bancos de dados. Logins in master (E) são princípios de nível de servidor, que este requisito não precisa e que também permitiria que os clientes se conectassem ao master. RLS a) e Always Encrypted (F) controlar linhas ou colunas, não a qual banco de dados um cliente pode se conectar.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure)

---

### Q68. Litware ResearchDB1: Sequência de configuração Always Encrypted

**Estudo de caso:** Litware

**Questão:**
Você cria todas as tabelas e pontos de vista para ResearchDB1. Você precisa implementar a segurança para ResearchDB1. A solução deve atender aos requisitos de segurança e conformidade. Quais três ações você deve executar em sequência?
*(Requisitos relevantes de Litware: armazene chaves de criptografia em Azure Key Vault; criptografe os dados PII em repouso, em trânsito e em uso; use o menor privilégio. ResearchApp1 preenche e acessa ResearchDB1.)*

**Alternativas:**
Acções:
- **A.** Execute o assistente Always Encrypted.
- **B.** Crie uma instância Azure Key Vault e gere um segredo.
- **C.** Crie uma instância Azure Key Vault e configure uma política de acesso.
- **D.** Criar um Microsoft Entra (Azure AD) managed identity.
- **E.** Register ResearchApp1 to Microsoft Entra ID (Azure AD).

**✅ Resposta:** 1 → Registro ResearchApp1 E- Microsoft Entra ID; 2 → Criar um Azure Key Vault e configurar uma política de acesso; 3 → Executar o Always Encrypted assistente

**Explicação:** Criptografar em uso significa Always Encrypted, e as chaves devem viver em Key Vault. O aplicativo cliente descriptografa os dados em si, por isso precisa de uma identidade Entra (um registro de aplicativo / service principal) que a política de acesso do cofre pode conceder permissões-chave para (obter, unwrapKey, wrapKey, sinal, verificar). Este é o clássico AE + AKV fluxo tutorial: registrar o aplicativo, criar o cofre e conceder o acesso ao aplicativo, em seguida, execute o Always Encrypted O assistente para criar o CMK no cofre e o CEK e criptografar o PII "Gerar um segredo" é errado porque o CMK Deve ser um cofre chave *chave*, não um segredo. Um independente "managed identity" não é algo com o qual o aplicativo está registrado neste fluxo, e a opção nomeia nenhum recurso para anexá-lo. ResearchApp1 é o passo da identidade de chave-titular.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-tutorial-getting-started) [Doc 2](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine)

---

### Q69. Coluna usada para junções: CEK + deterministic

**Questão:**
Você tem um Azure SQL database nomeado DB1 que contém duas tabelas denominadas Table1 e Table2. Ambas as tabelas contêm uma coluna chamada Column1. Column1 é usado para junções por um aplicativo chamado App1. Você precisa proteger o conteúdo de Column1 em repouso, em trânsito e em uso. Como você deve proteger o conteúdo de Column1?

**Alternativas:**

**Chave de criptografia:**
- **A.** Chave de encriptação da coluna
- **B.** chave de criptografia de banco de dados
- **C.** Chave mestra de serviço

**Tipo de criptografia:**
- **A.** determinística
- **B.** Aleatorizado
- **C.** Transparent Data Encryption (TDE)

**✅ Resposta:** Chave de criptografia: **Chave de encriptação da coluna**; Tipo de criptografia: **determinística**

**Explicação:** A proteção de dados "em uso" requer Always Encrypted, onde uma chave de criptografia de coluna (ela própria protegida por uma chave mestra de coluna) criptografa os dados da coluna. A criptografia determinística permite associações de igualdade, pesquisas pontuais, agrupamento e indexação. A criptografia aleatória não suporta junções (sem enclaves). O TDE e a chave de criptografia de banco de dados protegem os dados apenas em repouso. A chave mestra de serviço é uma chave de nível de instância, não usada para proteção de coluna.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine)

---

### Q70. Comparações sobre criptografia aleatória: enclaves

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1. Você precisa criptografar DB1. A solução deve atender aos seguintes requisitos:
- Criptografar dados em movimento.
- Operadores de comparação de suporte.
- Forneça criptografia aleatória.

O que você deve incluir na solução?

**Alternativas:**
- **A.** Always Encrypted with secure enclaves
- **B.** Always Encrypted
- **C.** Encriptação de nível de coluna
- **D.** Transparent Data Encryption (TDE)

**✅ Resposta:** A

**Explicação:** "Para realizar correspondência de padrões usando operadores de comparação, classificação e indexação em colunas criptografadas, adote Always Encrypted with secure enclaves e aplique criptografia aleatória." Plain Always Encrypted (B) não suporta computação em colunas aleatórias e apenas igualdade em deterministas. Criptografia em nível de coluna com chaves simétricas (C) e TDE (D) descriptografar dentro do motor e não manter os dados criptografados em movimento e em uso da mesma maneira.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine)

---

### Q71. Ocultar salário de DBAs: AE ordem chave

**Questão:**
Você tem um Azure SQL database que contém uma tabela chamada Funcionários. Os funcionários contêm uma coluna chamada Salário. Você precisa criptografar a coluna Salário. A solução deve impedir os administradores de banco de dados de ler os dados na coluna Salário e deve fornecer a criptografia mais segura. Quais três ações você deve executar em sequência?

**Alternativas:**
Acções:
- **A.** Criptografar a coluna Salário usando o tipo de criptografia aleatória.
- **B.** Crie uma chave de criptografia de coluna.
- **C.** Habilitar Transparent Data Encryption (TDE).
- **D.** Criptografe a coluna Salário usando o tipo de criptografia determinística.
- **E.** Aplique uma máscara de dados dinâmica na coluna Salário.
- **F.** Crie uma chave mestra de coluna.

**✅ Resposta:** 1→ Criar uma chave mestra da coluna; 2 → Criar uma chave de criptografia da coluna; 3 → Criptografar a coluna Salário usando o tipo de criptografia aleatória

**Explicação:** Always Encrypted mantém as chaves fora do motor de banco de dados, então DBAs ver apenas o texto cifrado. O CEK é encriptado pelo CMK, então o CMK A criptografia aleatória é mais segura do que determinística, o que vaza padrões de igualdade. TDE apenas protege os dados em repouso, e dynamic data masking não impede usuários privilegiados, portanto nenhum dos dois pára DBAs.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine)

---

### Q72. SIM: Always Encrypted LastName com cert store

**Questão:**
SIMULATION. Você precisa criptografar a coluna LastName da tabela SalesLT.Customer em db1 usando Always Encrypted. Você deve usar o Windows Certificate Store. Você pode precisar usar SQL Server Management Studio e o Azure portal.

**Alternativas:** Nenhum (tarefa de laboratório manual)

**Resposta ✅ (passos, assistente SSMS Always Encrypted):**
1. Em SSMS, conecte-se ao servidor lógico de hospedagem **db1** (obter o nome do servidor do Azure portal se necessário).
2. Explorador de Objetos → Bancos de dados → **db1** → Tabelas → clique com o botão direito **SalesLT.Customer** → **Criptografar colunas...** (ou expanda Colunas e clique com o botão direito **LastName** → Encriptar coluna).
3. Página de introdução → **Próxima**.
4. Em Column Selection, selecione **LastName**. Em Encryption Type, escolha **Deterministic**. Esse tipo aceita buscas por igualdade. Randomized também é aceitável porque a tarefa não especifica o tipo. Mantenha Encryption Key = **CEK_Auto1 (New)** → **Next**.
5. Em Master Key Configuration, defina o key store provider como **Windows certificate store** e a origem da master key como **Current User**. Isso gera automaticamente o certificado da CMK. Selecione **Next**.
6. Configurações de criptografia no local (se mostrado) → **Próxima**. Executar configurações: **Continue para terminar agora** → **Próxima**.
7. Resumo → **Acabamento** → esperar todas as tarefas para ter sucesso → **Fechar**.
8. (Verificar) db1 → Segurança → Always Encrypted Chaves mostra CMK_Auto1 / CEK_Auto1, ou executar: SELECT nome, encryption_type_desc FROM sys.columns WHERE encryption_type IS NOT NULL;

**Explicação:** O assistente cria a column master key (um certificado no repositório CurrentUser\My), a column encryption key e seus metadados. Em seguida, criptografa a coluna no cliente e muda a collation da cadeia para o tipo _BIN2 exigido. T-SQL sozinho não faz isso porque o engine nunca tem acesso às chaves. Alternativa manual: Security → Always Encrypted Keys → New Column Master Key (Windows Certificate Store – Current User) → New Column Encryption Key. Depois, execute o assistente e escolha essa CEK.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-tutorial-getting-started)

---

### Q73. Tipos de dados Always Encrypted suporta

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1. DB1 tem uma tabela chamada Table1 que contém as seguintes colunas.

| Nome | Tipo |
|---|---|
| Column1 | Ntext |
| Column2 | Geometry |
| Column3 | Image |
| Column4 | Varchar |
| Column5 | Datetime2 |

Você planeja habilitar Always Encrypted para Table1. Quais duas colunas suportam criptografia? Cada resposta correta apresenta uma solução completa.

**Alternativas:** *(algumas opções reconstruídas)*
- **A.** Column1
- **B.** Column2
- **C.** Column3
- **D.** Column4
- **E.** Column5

**✅ Resposta:** Column4 (varchar) e Column5 (datetime2), i.e. D, E sob as letras reconstruídas

**Explicação:** Always Encrypted não é suportado para colunas do tipo xml, timestamp/rowversion, **imagem**, **ntext**, texto, sql_variantHierarquia, Geografia, **geometria**, vector, alias ou tipos definidos pelo usuário. Isso exclui Column1 (ntexto), Column2 (geometria) e Column3 (imagem). varchar pode ser criptografado (precisa de um agrupamento _BIN2, que o assistente define), e datetime2 podem ser criptografados.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine)

---

### Q74. Contoso: destino de auditoria para um único painel

**Estudo de caso:** Contoso

**Questão:**
Qual destino de registro de auditoria você deve usar para atender aos requisitos de monitoramento?
*(Requisito de monitoramento contínuo: "Use um único painel para revisar dados de segurança e auditoria para todos os bancos de dados PaaS.")*

**Alternativas:**
- **A.** Azure Storage
- **B.** Azure Event Hubs
- **C.** Azure Log Analytics

**✅ Resposta:** C

**Explicação:** Envio de registros de auditoria de todos os bancos de dados PaaS para um Log Analytics workspace permite que você consulta e visualizá-los juntos (Azure Monitor workbooks/dashboards, Defender for Cloud), que lhe dá um único lugar para rever dados de segurança e auditoria. Azure Storage arquivos.xel por servidor ou banco de dados que você tem que ler com SSMS : sys.fn_get_audit_file, por isso não dá painel. Event Hubs apenas transmite eventos para um consumidor externo (como a SIEM) e não armazená-los para revisão.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/auditing-overview)

---

### Q75. SQL VM: auditoria SELECTs no CustomerPII

**Questão:**
Você tem SQL Server em um Azure virtual machine que contém um banco de dados chamado DB1. DB1 contém uma tabela chamada CustomerPII. Você precisa gravar sempre que os usuários consultam a tabela CustomerPII. Quais duas opções você deve habilitar? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** SQL Server audit
- **B.** database audit specification
- **C.** server audit specification
- **D.** um servidor principal

**✅ Resposta:** A, B (um objeto SQL Server audit mais um database audit specification)

**Explicação:** Em SQL Server Audit, o *SQL Server Audit* objeto é criado no nível da instância e define o alvo (arquivo ou log de eventos). *Database Audit Specification* que pertence a essa auditoria e coleta ações de nível de banco de dados, tais como `SELECT` acção em `dbo.CustomerPII`A server audit specification (C) só mantém grupos de ação em nível de servidor e não pode ter como alvo uma única tabela. Um servidor principal (D) não é um componente de auditoria. Uma especificação não pode gravar nada sem o objeto de auditoria, então A é necessário.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/auditing/sql-server-audit-database-engine)

---

### Q76. Leituras da coluna confidencial, 365 dias

**Questão:**
Você tem um novo Azure SQL database. O banco de dados contém uma coluna que armazena informações confidenciais. Você precisa rastrear cada vez que os valores da coluna são retornados em uma consulta. As informações de rastreamento devem ser armazenadas por 365 dias a partir da data em que a consulta foi executada. Quais três ações você deve executar? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** Ative a auditoria e escreva logs de auditoria em uma conta Azure Storage.
- **B.** Adicione propriedades estendidas à coluna.
- **C.** Ative a auditoria e escreva registros de auditoria em um Event Hub.
- **D.** Aplique rótulos de sensibilidade chamados Altamente Confidenciais à coluna.
- **E.** Ative o Azure Defender para SQL.

**✅ Resposta:** A, D, E

**Explicação:** Um rótulo de sensibilidade na coluna (D) faz SQL auditing preencher o `data_sensitivity_information` um campo, que registra os rótulos dos dados uma consulta retornou. Auditando a uma conta de armazenamento (A) com retenção definida para 365 dias armazena os registros para o ano necessário. Um Event Hub (C) apenas transmite eventos e não retê-los por 365 dias, para que ele não atenda ao requisito de armazenamento. Na versão original deste item a terceira opção correta foi "Turn on Advanced Data Security", que foi renomeado Azure Defender para SQL / Microsoft Defender for SQLEssa opção é E nesta versão (Pass4Success chave A, C, D na lettering original, sobre 97% Hoje Data Discovery & Classification é construído em sem Defender, então E é uma escolha de chave de legado.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/data-discovery-and-classification-overview) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/auditing-overview)

---

### Q77. ADatum: Acesso de Kerberos App1 a Instance1

**Estudo de caso:** ADatum v1 (financeiro)

**Questão:**
Você precisa recomendar uma solução de autenticação para o acesso App1 ao DB1 e DB2 após a migração para Instance1. A solução deve atender aos requisitos de disponibilidade. Quais ações você deve executar em sequência? Cada ação pode ser usada uma vez, mais de uma vez, ou não.
*(Contexto do caso: App1 é executado em servidores Microsoft Entra híbrido juntou Windows Server 2022 e usa autenticação Kerberos. O domínio on-prem corp.adatum.com AD **já sincroniza** com o inquilino adatum.com Entra. Instance1 é um Azure SQL managed instance. Exigência de disponibilidade: "Após a migração, App1 deve manter o acesso a DB1 e DB2 ")*

**Alternativas:**
Área de resposta: Passo 1 (pré-preenchido): Habilitar Microsoft Entra authentication no Instance1. Passo 2: ____ Passo 3: ___
Acções:
- **A.** Habilitar um system-assigned managed identity.
- **B.** Habilitar um service principal atribuído ao sistema.
- **C.** Consentimento do administrador da concessão para um registro do app em Microsoft Entra.
- **D.** Implemente o Microsoft Entra Cloud Sync.

**✅ Resposta:** Passo 2: **Habilitar um service principal atribuído ao sistema**; Passo 3: **Consentimento do administrador Grant para um registro de aplicativo em Microsoft Entra**

**Explicação:** Windows Authentication (Kerberos) para os principais do Entra em SQL MI A autenticação intra requer autenticação de entrada na instância (Passo 1). Microsoft MI página de configuração, em seguida, diz: "você precisa habilitar um sistema-atribu service principal em cada SQL managed instanceVocê também precisa conceder consentimento administrativo a cada service principal". O consentimento é concedido via Entra ID → inscrições App → `<instance> principal` → API permissões → Consentimento do administrador da concessão. ADA sincronização -to-Entra é um pré-requisito único "se isso ainda não tiver sido feito", e o caso diz: corp.adatum.com A implementação do Cloud Sync não acrescenta nada. O system-assigned managed identity é a MIidentidade primária do Graph ou Key Vault acesso, não o Kerberos service principal.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/winauth-azuread-kerberos-managed-instance) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/winauth-azuread-setup)

---

### Q78. Litware SERVER1: sequência de autenticação intra

**Estudo de caso:** Litware

**Questão:**
Você precisa configurar a autenticação do usuário para os bancos de dados SERVER1. A solução deve atender aos requisitos de segurança e conformidade. Quais três ações você deve executar em sequência?
*(Litware: SERVER1 bancos de dados migram para Azure SQL Database. Requisito: "Autenticar usuários de banco de dados usando credenciais Active Directory" (hybrid Entra ID, litwareinc.com). Menor privilégio.)*

**Alternativas:**
Acções:
- **A.** Criar um usuário no banco de dados mestre
- **B.** Modificar a conta de administrador do servidor Azure SQL
- **C.** Criar usuários de banco de dados contidos
- **D.** Criar um administrador Microsoft Entra (Azure AD) para o servidor lógico
- **E.** Conecte-se aos bancos de dados usando uma conta Microsoft Entra (Azure AD)
- **F.** Ativar a opção de autenticação do banco de dados contido

**✅ Resposta:** 1 → Criar um administrador Microsoft Entra para o servidor lógico; 2 → Criar usuários de banco de dados contidos; 3 → Conecte-se aos bancos de dados usando uma conta Microsoft Entra

**Explicação:** This follows Microsoft's documented sequence: assign a Microsoft Entra administrador para o servidor → criar SQL principais (usuários, `CREATE USER … FROM EXTERNAL PROVIDER`) mapeado para identidades Entra → Os usuários contidos evitam master logins "Ativar a opção de autenticação do banco de dados contido" é um `sp_configure` configuração para boxed SQL Server e não é necessário ou disponível em Azure SQL Databaseonde Os bancos de dados de usuários já estão contidas.
Disputa: MD ordena que admin → conecte → crie usuários, argumentando que o administrador do Entra deve se conectar antes de criar usuários. Isso é verdade na prática, mas a etapa "Connect..." do item significa que os usuários se conectam com suas contas do Entra, o que corresponde à etapa final documentada.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-overview)

---

### Q79. App1 SQL acesso apenas ao DB2

**Questão:**
Você tem uma assinatura do Azure que contém um servidor chamado Server1. Server1 hospeda dois Azure SQL databases chamados DB1 e DB2.
Você planeja implantar um aplicativo do Windows chamado App1 que se autenticará no DB2 usando o SQL authentication.
Você precisa garantir que o App1 possa acessar o DB2. A solução deve atender aos seguintes requisitos:
- App1 deve ser capaz de ver apenas DB2.
- O esforço administrativo deve ser minimizado.

O que você deve criar?

**Alternativas:**
- **A.** contained database user para App1 em DB2
- **B.** login para App1 em Server1
- **C.** a contained database user de um fornecedor externo para App1 E-mail DB2
- **D.** contained database user de um Windows login para App1 em DB2

**✅ Resposta:** A

**Explicação:** A contained database user com uma senha (`CREATE USER App1 WITH PASSWORD = '...'` E- DB2) autentica no nível do banco de dados com SQL authentication, então App1 só pode ver DB2 e nada tem que ser criado no mestre. A login E-mail Server1 (B) necessita de a login mais um usuário mapeado, que é mais trabalho e é um principal de nível de servidor. FROM EXTERNAL PROVIDER c) é para Microsoft Entra identidades, não SQL authentication. Janelas logins (D) não são suportados em Azure SQL Database.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/logins-create-manage)

---

### Q80. Reutilizável SQL login para App1 (db_datareader)

**Questão:**
Você tem uma instância Azure SQL Database chamada DatabaseA em um servidor chamado Server1.
Você planeja adicionar um novo usuário chamado App1 ao DatabaseA e conceder permissões App1 db_datareader. App1 usará SQL Server Authentication.
Você precisa criar App1. A solução deve garantir que o App1 possa ter acesso a outras bancos de dados usando as mesmas credenciais.
Para responder, mova as ações apropriadas da lista de ações para a área de resposta e organize-as na ordem correta.

**Alternativas:**
Acções:
- **A.** No banco de dados mestre, execute `CREATE LOGIN [App1] FROM EXTERNAL PROVIDER;`
- **B.** Em DatabaseA, execute `CREATE USER [App1] WITH PASSWORD = 'P@ssW0rd!';`
- **C.** Em DatabaseA, execute `ALTER ROLE db_datareader ADD MEMBER [App1];`
- **D.** No banco de dados mestre, execute `CREATE LOGIN [App1] WITH PASSWORD = 'P@ssW0rd!';`
- **E.** Em DatabaseA, execute `CREATE USER [App1] FROM LOGIN [App1];`

(A senha lê `P@aaW0rd!` nas cópias MD / CQ e `P@ssW0rd!` na cópia NOTE. Esta é apenas uma diferença de transcrição e não altera a resposta.)

**✅ Resposta:** 1. No mestre: `CREATE LOGIN [App1] WITH PASSWORD = '...';` → 2. Na base de dadosA: `CREATE USER [App1] FROM LOGIN [App1];` → 3. Na base de dadosA.: `ALTER ROLE db_datareader ADD MEMBER [App1];`

**Explicação:** A SQL authentication login no mestre detém um conjunto de credenciais que podem ser mapeadas para um usuário em qualquer banco de dados no servidor, App1 pode reutilizá-los. O usuário é então criado a partir desse login em DatabaseA e adicionado a db_datareaderA contained user WITH PASSWORD Existe apenas em um banco de dados, então suas credenciais não podem ser reutilizadas. FROM EXTERNAL PROVIDER E Microsoft Entra authentication, não SQL authentication.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/logins-create-manage)

---

### Q81. Usuário de banco de dados Managed identity WITH OBJECT_ID

**Questão:**
Você tem uma assinatura do Azure que contém um banco de dados do Azure SQL Database chamado DB1 e um managed identity.
Você precisa criar um usuário no DB1 que será usado pelo managed identity para realizar tarefas automatizadas.
Como você deve completar a consulta? Para responder, selecione as opções apropriadas na área de resposta.
```sql
CREATE USER "AutomationAccount"
FROM [Box 1]
WITH [Box 2] = '1133a5a2-4c19-47d7-c2d5-42b289997918f'
EXEC sp_addrolemember 'db_owner', "AutomationAccount"
```

**Alternativas:**

**Caixa 1:**
- **A.** ASYMMETRIC KEY
- **B.** EXTERNAL PROVIDER
- **C.** FOR CERTIFICATE

**Caixa 2:**
- **A.** KEY
- **B.** LOGIN
- **C.** OBJECT_ID

**✅ Resposta:** Caixa 1: EXTERNAL PROVIDER; Caixa 2: OBJECT_ID

**Explicação:** Managed identities são Microsoft Entra service principals, então o usuário é criado com `FROM EXTERNAL PROVIDER`. A extensão `WITH OBJECT_ID = '<Entra object ID>'` liga o usuário a essa identidade exata, o que importa quando os nomes de exibição não são únicos e permite que você escolha seu próprio nome de usuário (" AutomationAccount "). Os usuários ASYMMETRIC KEY e CERTIFICATE são para assinatura do módulo, e LOGIN não é uma opção WITH válida aqui.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure)

---

### Q82. Usuário incluído no novo DB1

**Questão:**
Você tem um novo Azure SQL database chamado DB1 em um servidor Azure SQL chamado AzSQL1. O único usuário que foi criado é o administrador do servidor.
Você precisa criar um contained database user no DB1 que usará o Microsoft Entra ID (Azure AD) para autenticação.
Para responder, mova as ações apropriadas da lista de ações para a área de resposta e organize-as na ordem correta.

**Alternativas:**
Acções:
- **A.** Crie um login no banco de dados mestre.
- **B.** A partir do Azure portal, atribuir a função de Colaborador SQL DB ao usuário.
- **C.** Conecte-se ao DB1 usando a conta de administrador do servidor.
- **D.** Defina o Microsoft Entra (Active Directory) Admin para AzSQL1.
- **E.** Conecte-se ao DB1 usando a conta de administrador do Microsoft Entra (Active Directory).
- **F.** Crie um usuário usando a cláusula FROM EXTERNAL PROVIDER.

**✅ Resposta:** 1. Defina o administrador Microsoft Entra para AzSQL1 → 2. Conecte-se ao DB1 usando a conta de administrador Microsoft Entra → 3. Crie um usuário usando a cláusula FROM EXTERNAL PROVIDER.

**Explicação:** Microsoft Entra authentication é ativado pela configuração do administrador do Entra. `CREATE USER ... FROM EXTERNAL PROVIDER` deve ser executado por um diretor que é assinado com um Microsoft Entra identidade. Microsoft Learn diz: "Criar usuários contidos mapeados para Microsoft Entra você deve estar logado usando uma Microsoft Entra conta." A SQL authentication O administrador do servidor não pode criar usuários do Entra. A login em mestre não é necessário para um contained usere, SQL DB O Azure Control-Plane Contributor é uma função de plano de controle do Azure que não dá acesso a dados.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/logins-create-manage) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure)

---

### Q83. A entrada User1 não consegue se conectar ao servidor lógico

**Questão:**
Você cria cinco instâncias Azure SQL Database no mesmo servidor lógico. Em cada banco de dados, você cria um usuário para um usuário Microsoft Entra (Azure AD) chamado User1.
O User1 tenta se conectar ao servidor lógico usando o Azure Data Studio e recebe um erro do login.
Você precisa garantir que quando User1 se conecta ao servidor lógico usando Azure Data Studio, User1 pode ver todas Os bancos de dados.
O que você deve fazer?

**Alternativas:**
- **A.** Crie User1 no banco de dados mestre.
- **B.** Atribua User1 o papel db_datareader para o banco de dados mestre.
- **C.** Atribua User1 o papel db_datareader para os bancos de dados que o User1 cria.
- **D.** Conceder SELECT em sys.databases para público na banco de dados mestre.

**✅ Resposta:** A

**Explicação:** User1 existe apenas como usuários contidos nos cinco bancos de dados de usuários. Uma conexão com o servidor lógico que não nomeia um banco de dados vai para mestre, onde User1 não tem nenhum principal, então o login falha. Criando User1 no mestre (`CREATE USER [User1] FROM EXTERNAL PROVIDER`, ou um login mais um usuário) permite que a conexão seja bem-sucedida, e User1 pode então listar os bancos de dados. B e D não podem ser concedido a um mestre não existe a principal 'não

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/logins-create-manage)

---

### Q84. Usuário de banco de dados do aplicativo Web managed identity

**Questão:**
Você tem uma assinatura do Azure que está vinculada a um Microsoft Entra (Azure AD) inquilino nomeado contoso.com. A assinatura contém um Azure SQL database nomeado SQL1 e um aplicativo web do Azure chamado app1. App1 tem a managed identity recurso habilitado.
Você precisa criar um novo usuário de banco de dados para app1.
Como você deve completar a declaração Transact-SQL? Para responder, selecione as opções apropriadas na área de resposta.
```sql
CREATE USER [Box 1] FROM [Box 2]
```

**Alternativas:**

**Caixa 1:**
- **A.** [App1]
- **B.** [Contoso\app1]
- **C.** [App1@contoso.com]

**Caixa 2:**
- **A.** login
- **B.** Windows
- **C.** EXTERNAL PROVIDER

**✅ Resposta:** Caixa 1: [App1 ]; Caixa 2: EXTERNAL PROVIDER

**Explicação:** A system-assigned managed identity é a service principal cujo nome de exibição é o mesmo que o recurso (o aplicativo web) nome, então o usuário é criado como `CREATE USER [App1] FROM EXTERNAL PROVIDER;`. O UPN formulário (`App1@contoso.com`) é usado apenas para entrada *usuários*. O `DOMAIN\name` forma e FROM WINDOWS são para on-premises Windows logins, que Azure SQL Database não apoia. FROM LOGIN Precisaria de um existente login.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure)

---

### Q85. Service principal app1 acesso ao db1

**Questão:**
Você tem uma assinatura do Azure que contém um SQL server lógico. O servidor hospeda dois bancos de dados chamados db1 e db2 e um Microsoft Entra service principal chamado app1.
Você precisa garantir que o app1 possa acessar o db1. A solução deve usar o princípio do menor privilégio.
Como você deve completar a declaração Transact-SQL? Para responder, selecione as opções apropriadas na área de resposta.
```sql
CREATE [Box 1] [app1] [Box 2]
```

**Alternativas:**

**Caixa 1:**
- **A.** CREDENTIAL
- **B.** LOGIN
- **C.** USER

**Caixa 2:**
- **A.** FOR LOGIN app1
- **B.** FROM EXTERNAL PROVIDER
- **C.** FROM LOGIN app1
- **D.** WITHOUT LOGIN

**✅ Resposta:** Caixa 1: USER; Caixa 2: FROM EXTERNAL PROVIDER

**Explicação:** Correr `CREATE USER [app1] FROM EXTERNAL PROVIDER;` E- db1 cria a contained database user para a service principal, com escopo para db1 apenas, o que é menos privilégio. A LOGIN é um servidor de nível principal e também precisa de um usuário mapeado. FROM/FOR LOGIN necessidade a login Isso não existe. WITHOUT LOGIN cria um usuário que não pode autenticar. CREDENTIAL Não é um principal.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-service-principal)

---

### Q86. SIM: adicionar Entra user2 read-only a db1

**Questão:**
SIMULATION. Você precisa adicionar um Microsoft Entra usuário nomeado user2-12345678@examusers.com E- db1. User2-12345678 deve ser capaz de ler dados de todas as tabelas em db1 sem poder modificar os dados.
Para concluir esta tarefa, inicie sessão na máquina virtual. Poderá ter de utilizar o SQL Server Management Studio e o Azure portal.

**Alternativas:**

**Resposta ✅ (passos):**
1. (Se ainda não estiver definido) Azure portal > servidores SQL > o servidor que hospeda db1 > **Configurações > Microsoft Entra ID** > **Definir administrador** > selecione sua conta de laboratório > **Selecionar** > **Gravar**.
2. Em SSMS, conecte-se ao servidor com **Microsoft Entra MFA / Entra Integrado** como o administrador do Entra. Na guia **Propriedades da conexão**, defina o banco de dados para **db1**.
3. Abra uma nova janela de consulta no db1 e execute:
```sql
CREATE USER [user2-12345678@examusers.com] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [user2-12345678@examusers.com];
```
4. Verificação opcional: `SELECT name, type_desc FROM sys.database_principals WHERE name LIKE 'user2%';`

**Explicação:** Um usuário deve ser criado com FROM EXTERNAL PROVIDER enquanto você estiver conectado com uma identidade Entra. db_datareader Subvenções SELECT em todas as tabelas e vistas e sem permissões de escrita. A fonte SSMS rota de diálogo ("Windows user" tipo, então por-tabela SELECT na página Securables) também dá read-only "Windows user" não é um tipo principal válido em Azure SQL Databasee as bolsas por tabela não cobrem tabelas adicionadas mais tarde.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/logins-create-manage)

---

### Q87. SIM: deixe runbook criar usuários Entra

**Questão:**
SIMULATION. Você planeja criar uma automação runbook que criará usuários de banco de dados em db1 a partir de identidades Microsoft Entra.
Você precisa configurar o sql12345678 para apoiar a criação de novos usuários de banco de dados.
Para concluir esta tarefa, inicie sessão na máquina virtual. Poderá ter de utilizar o SQL Server Management Studio e o Azure portal.

**Alternativas:**

**Resposta ✅ (passos):** A runbook autentica como um service principal ou managed identity, que não pode usar pesquisas de gráficos delegados. Então o **identidade do servidor** do sql12345678 deve existir e ser permitido ler Microsoft Entra ID.
1. Azure portal: verifique se o diretório correto está selecionado (**Diretórios + subscrições**).
2. **Servidores SQL > sql12345678 > Segurança > Identidade** > conjunto **Sistema atribuído managed identity** a **Ligado** > **Gravar**. Alternativamente, use PowerShell: `Set-AzSqlServer -ResourceGroupName <rg> -ServerName sql12345678 -AssignIdentity`.
3. Dê permissão de identidade do servidor para ler o Entra ID: adicione-o ao papel Microsoft Entra **Diretório Leitores** (Entra ID > Funções e administradores > Leitores de diretório > Adicionar atribuições > selecionar a identidade sql12345678) ou atribua as permissões do aplicativo Graph User.Read.All, GroupMember.Read.All e Application.Read.All com PowerShell. Você precisa do papel de administrador de função privilegiada para isso.
4. Certifique-se de que um **Microsoft Entra admin** está definido em sql12345678 (**Configurações > Microsoft Entra ID > Definir administrador > Salvar**). A identidade do runbook pode ser que admin ou pode ser concedido ALTER ANY USER em db1.

**Explicação:** Microsoft Learn diz que para um service principal executar o `CREATE USER ... FROM EXTERNAL PROVIDER`, "o motor SQL tenta usar sua identidade de servidor... A identidade do servidor deve existir e ter as permissões de consulta do Microsoft Graph ou as operações falham" (erro 33134 "A identidade do servidor não está configurada" de outra forma). Os passos da fonte (o banner "Permissões de concessão" na página de administração do Entra) são escritos para SQL Managed Instance. Para um servidor lógico, você pode atribuir a identidade ao seu próprio.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-service-principal) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity)

---

### Q88. SIM: configure sua conta como administrador do Entra

**Questão:**
SIMULATION. Você precisa configurar sua conta de usuário como o administrador Microsoft Entra para o servidor chamado sql12345678.
Para concluir esta tarefa, inicie sessão na máquina virtual. Poderá ter de utilizar o SQL Server Management Studio e o Azure portal.

**Alternativas:**

**Resposta ✅ (passos):**
1. Azure portal: em **Diretórios + subscrições**, selecione o diretório que contém sql12345678.
2. Pesquisar **Servidores SQL** > abrir **sql12345678**.
3. **Configurações > Microsoft Entra ID** > **Definir administrador**.
4. Procure sua conta de usuário de laboratório, selecione-a > **Selecionar**.
5. Clique em **Gravar** e aguarde até que a operação termine (o administrador então aparece no campo de administração Microsoft Entra).
Alternativa PowerShell: `Set-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName <rg> -ServerName sql12345678 -DisplayName "<your UPN>"`. Alternativa CLI: `az sql server ad-admin create --resource-group <rg> --server-name sql12345678 --display-name <UPN> --object-id <id>`.

**Explicação:** Este é o procedimento documentado para um servidor lógico. O banner de "permissões de leitura de concessão" da fonte se aplica ao SQL Managed Instance, e sua alternativa de atribuição de papel "Azure AD B2C" não é relevante.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure)

---

### Q89. Deixe user1 criar SP logins em SQLMI1

**Questão:**
Você tem a Microsoft Entra (Azure AD) inquilino nomeado contoso.com que contém um usuário chamado user1@contoso.com e um Azure SQL managed instance nomeado SQLMI1.
Você precisa garantir que user1@contoso.com pode criar logins E- SQLMI1 Esse mapa para Microsoft Entra (Azure AD) service principals.
Para responder, mova as ações apropriadas da lista de ações para a área de resposta e organize-as na ordem correta.

**Alternativas:**
Acções:
- **A.** Execute o `CREATE LOGIN [user1@contoso.com] FROM EXTERNAL PROVIDER` na banco de dados mestre.
- **B.** Execute `ALTER SERVER ROLE securityadmin ADD MEMBER [user1@contoso.com]`.
- **C.** Crie um managed identity para SQLMI1.
- **D.** SQLMI1 ler o acesso ao Microsoft Entra ID (Azure AD).
- **E.** Execute `CREATE USER [user1@contoso.com] FROM LOGIN [user1@contoso.com]`.

**✅ Resposta:** 1. Grant SQLMI1 acesso de leitura para Microsoft Entra ID (leitores diretos) → 2. Run `CREATE LOGIN [user1@contoso.com] FROM EXTERNAL PROVIDER` no mestre → 3. Run `ALTER SERVER ROLE securityadmin ADD MEMBER [user1@contoso.com]`.

**Explicação:**
- SQL MI precisa de sua identidade de instância para ter leitores de diretório (ou as permissões de gráfico equivalentes) para que ele possa resolver os usuários do Entra e service principals. Concessão que vem em primeiro lugar.
- O user1 precisa então de um servidor Entra principal (login) no mestre.
- Criando logins precisa de ALTER ANY LOGIN, que vem com o papel **segurançaadmin**. Sem ele, user1 só pode se conectar.
- "Criar um managed identity" não é necessário: Microsoft Learn afirma que "Um SMI é automaticamente atribuído ao Azure SQL Managed Instance quando é criado."
- O `CREATE USER ... FROM LOGIN` só adiciona um usuário de banco de dados, o que não dá permissão no nível do servidor para criar o logins.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure)

---

### Q90. Synapse Pool1: MFA e autenticação de nível de banco de dados

**Questão:**
Você tem uma assinatura do Azure vinculada a um inquilino híbrido Microsoft Entra (Azure AD). A assinatura contém um pool Azure Synapse Analytics SQL chamado Pool1.
Você precisa recomendar uma solução de autenticação para Pool1. A solução deve suportar autenticação multifator (MFA) e autenticação em nível de banco de dados.
Qual solução ou soluções de autenticação você deve incluir na recomendação? Para responder, selecione as opções apropriadas na área de resposta.

**Alternativas:**

**MFA:**
- **A.** Autenticação Microsoft Entra (Azure AD)
- **B.** Microsoft SQL Server authentication
- **C.** Autenticação sem senha
- **D.** Windows authentication

**Autenticação em nível de banco de dados:**
- **A.** Funções de aplicação
- **B.** Usuários de banco de dados contidos
- **C.** Funções de banco de dados
- **D.** Microsoft SQL Server logins

**✅ Resposta:** MFA: Microsoft Entra authentication; Autenticação ao nível da banco de dados: Usuários de bancos de dados contê

**Explicação:** O Microsoft Entra authentication só suporta MFA (Universal / Interativo com MFA, imposto pelo Acesso Condicional). SQL authentication é apenas senha e o Windows authentication não está disponível no Azure SQL ou Synapse. Microsoft Entra authentication "usa usuários de banco de dados contidos para autenticar identidades no nível do banco de dados". Logins são de nível de servidor e funções de banco de dados ou aplicativo são para autorização, não autenticação.
*Observação:* o produto aqui é Synapse, mas o mesmo comportamento se aplica a Azure SQL Database e Managed Instance.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-overview) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure)

---

### Q91. ADatum DB3: audit all changes — first step

**Estudo de caso:** ADatum v1 (serviços financeiros)

**Questão:**
(Estudo de caso: ADatum Corporation, serviços financeiros. DB3 em SVR3 (SQL Server 2019) será migrado para Azure SQL Database. Exigência de segurança: "Garantir que todas as alterações para DB3, incluindo aquelas dentro de transações individuais, sejam auditadas e registradas." Requisitos de negócios: minimizar custos e esforço administrativo.)
Você precisa recomendar uma solução para atender aos requisitos de segurança e os requisitos de negócios para DB3. O que você deve recomendar como o primeiro passo da solução?

**Alternativas:**
- **A.** Execute o procedimento armazenado sp_addarticle.
- **B.** Execute a instrução ALTER TABLE e especifique a cláusula ENABLE CHANGE_TRACKING.
- **C.** Executar o ALTER DATABASE Especifique a declaração e SET CHANGE_TRACKING = ON cláusula.
- **D.** Execute o procedimento armazenado sys.sp_cdc_enable_db.

**✅ Resposta:** D

**Explicação:** Change data capture lê o diário de transações e registra cada inserção, atualização e exclusão, com valores antes/depois, incluindo várias alterações na mesma linha dentro de uma transação. Change tracking (B, C) registra apenas que uma linha mudou (mudança líquida, chave primária apenas), de modo que não pode registrar todas as alterações dentro de uma transação. CDC deve ser habilitado no nível do banco de dados com `sys.sp_cdc_enable_db` antes `sys.sp_cdc_enable_table`Então esse é o primeiro passo. sp_addarticle a) é para transactional replication e não é um recurso de auditoria de mudança.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/change-data-capture-overview)

---

### Q92. SIM: habilitar CDC para db1

**Questão:**
SIMULATION. Você precisa habilitar change data capture (CDC) para db1.
Para concluir esta tarefa, inicie sessão na máquina virtual. Poderá ter de utilizar o SQL Server Management Studio e o Azure portal.

**Alternativas:**

**Resposta ✅ (passos):**
1. Em SSMSconectar-se ao servidor lógico com uma conta que é **db_owner** E-mail db1 (o administrador do servidor funciona, ou o administrador do Entra se o banco de dados foi criado por um usuário do Entra).
2. Abra uma nova janela de consulta contra o **db1** (não mestre) e execute:
```sql
EXEC sys.sp_cdc_enable_db;
GO
SELECT name, is_cdc_enabled FROM sys.databases WHERE name = 'db1';  -- verify = 1
```
3. Opcional, somente se a tarefa solicitar captura em nível de tabela: `EXEC sys.sp_cdc_enable_table @source_schema = N'<schema>', @source_name = N'<table>', @role_name = NULL;`
Pré-requisito: db1 deve ser vCore (qualquer nível) ou DTU **S3 ou superior**. Se você receber um erro no Basic ou S0 – S2, aumente primeiro (Portal > db1 > Compute + armazenamento).

**Explicação:** CDC está habilitado por banco de dados com `sys.sp_cdc_enable_db`, que precisa de db_owner. Em Azure SQL Database o agendador CDC substitui SQL Agent jobs, então nada mais precisa ser configurado.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/change-data-capture-overview)

---

### Q93. Change tracking para exportação webapp1

**Questão:**
Você tem uma assinatura do Azure que contém um banco de dados do Azure SQL Database chamado SQLDb1. SQLDb1 contém uma tabela chamada Table1.
Você planeja implantar um aplicativo web do Azure chamado webapp1 que exportará as linhas em Table1 que mudaram.
Você precisa garantir que webapp1 possa identificar as alterações para Table1. A solução deve atender aos seguintes requisitos:
- Minimize os tempos de computação.
- Minimizar o armazenamento.

Para responder, mova as ações apropriadas da lista de ações para a área de resposta e organize-as na ordem correta.

**Alternativas:**
Acções:
- **A.** De webapp1, conecte-se ao SQLDb1, obtenha o conjunto de dados inicial e execute a função CHANGE_TRACKING_CURRENT_VERSION ().
- **B.** Conecte-se ao SQLDb1 e execute a seguinte declaração Transact-SQL: `EXEC sys.sp_cdc_enable_table`
- **C.** Conecte-se ao SQLDb1 e execute a seguinte declaração Transact-SQL: `EXEC sys.sp_cdc_enable_db`
- **D.** De webapp1, conecte-se ao SQLDb1, obtenha o conjunto de dados inicial e execute a função CHANGETABLE ().
- **E.** Conecte-se ao SQLDb1 e execute a seguinte declaração Transact-SQL: `ALTER DATABASE SQLDb1 SET CHANGE_TRACKING = ON`
- **F.** Conecte-se ao SQLDb1 e execute a seguinte declaração Transact-SQL: `ALTER TABLE dbo.Table1 ENABLE CHANGE_TRACKING`

**✅ Resposta:** 1. `ALTER DATABASE SQLDb1 SET CHANGE_TRACKING = ON` → 2. `ALTER TABLE dbo.Table1 ENABLE CHANGE_TRACKING` → 3. De webapp1, obter o conjunto de dados inicial e executar **CHANGE_TRACKING_CURRENT_VERSION()**.

**Explicação:**
- Change tracking, não CDC, atende "minimizar computação e armazenamento": armazena apenas as chaves primárias de linhas alteradas e não faz varredura de log assíncrona. Deve estar habilitado no banco de dados primeiro, depois na tabela.
- Microsoft Learn ("Obter dados iniciais"): "Antes que um aplicativo possa obter alterações pela primeira vez, o aplicativo deve enviar uma consulta para obter os dados iniciais e a versão de sincronização. O aplicativo deve obter os dados apropriados diretamente da tabela e, em seguida, usar CHANGE_TRACKING_CURRENT_VERSION () para obter a versão inicial." Isso corresponde à primeira ação palavra por palavra.
- CHANGETABLE (CHANGES...) é usado *Mais tarde*, em cada exportação, com essa versão salva. Não é usado para obter o conjunto de dados inicial.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/work-with-change-tracking-sql-server)

---

### Q94. SIM: classificar StandardCost / ListPrice como Financeiro / Confidencial

**Questão:**
SIMULATION. Você precisa classificar as colunas StandardCost e ListPrice da tabela SalesLT.Product em db1. As colunas devem ter as seguintes configurações:
- um tipo de informação financeira.
- Um rótulo de sensibilidade de Confidencial.

Para concluir esta tarefa, inicie sessão na máquina virtual. Poderá ter de utilizar o SQL Server Management Studio e o Azure portal.

**Alternativas:**

**Resposta ✅ (passos):** Use qualquer um dos seguintes métodos.
- **Azure portal:** db1 > **Segurança > Descoberta e classificação de dados** > **Classificação** tab > **+ Adicionar classificação** > Schema **VendasLT**, Tabela **Produto**, Coluna **StandardCost**, Tipo de informação **Financeiro**, Etiqueta de sensibilidade **Confidencial** > **Adicionar classificação**. Repita para **ListPrice**, depois clique em **Gravar**.
- **SSMS:** clique com o botão direito db1 > **Tarefas > Descoberta e classificação de dados > Classificar dados...** > **Adicionar classificação** (ou edite as recomendações para essas colunas) com os mesmos valores para ambas as colunas > **Gravar**.
- **T-SQL** (mais rápido, executado em db1):
```sql
ADD SENSITIVITY CLASSIFICATION TO SalesLT.Product.StandardCost, SalesLT.Product.ListPrice
WITH (LABEL = 'Confidential', INFORMATION_TYPE = 'Financial');
-- verify
SELECT * FROM sys.sensitivity_classifications;
```

**Explicação:** A Descoberta e Classificação de Dados armazena o rótulo e o tipo de informação como metadados de coluna. Uma declaração `ADD SENSITIVITY CLASSIFICATION` pode classificar várias colunas (documentadas). Com o portal ou SSMS, nada é salvo até que você clique em **Gravar**.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/t-sql/statements/add-sensitivity-classification-transact-sql)

---

### Q95. Clientes Synapse: ocultar dados de cartão de crédito

**Questão:**
Você está projetando um data warehouse corporativo em Azure Synapse Analytics que conterá uma tabela chamada Clientes. Os clientes conterão informações de cartão de crédito.
Você precisa recomendar uma solução para fornecer aos vendedores a capacidade de ver todas as entradas em Clientes. A solução deve impedir que todos os vendedores vejam ou inferindo as informações do cartão de crédito.
O que você deve incluir na recomendação?

**Alternativas:**
- **A.** row-level security
- **B.** mascaramento de dados
- **C.** Always Encrypted
- **D.** Segurança em nível de coluna

**✅ Resposta:** D

**Explicação:** Segurança de nível de coluna (`GRANT SELECT ON dbo.Customers (col1, col2, …)` sem a coluna do cartão, ou DENY nessa coluna) significa que os vendedores não podem ler ou referenciar a coluna do cartão de crédito, então eles não podem inferir isso com predicados. Dynamic data masking b) apenas mascarar a saída; e Microsoft Learn Os usuários com acesso ad hoc a consultas podem inferir valores mascarados por força bruta (por exemplo, `WHERE` sondagem de alcance). Row-level security (A) filtra linhas, não colunas, e entraria em conflito com "ver todas as entradas". Always Encrypted (C) é uma solução mais pesada destinada a esconder dados de DBAs e o servidor, e não é o ajuste mínimo para este requisito.
*Observação:* o produto é Synapse, mas o mesmo comportamento se aplica ao Azure SQL Database e Managed Instance (coluna-nível GRANT / DENY e DDM trabalho de inferência da mesma maneira).

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-gb/azure/synapse-analytics/sql-data-warehouse/column-level-security) [Doc 2](https://learn.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking)

---

### Q96. Corda personalizada DDM para Customer_Phone

**Questão:**
Você tem um Azure SQL database que contém uma tabela chamada Cliente. O cliente tem as colunas mostradas na tabela a seguir.

| Customer_ID | Customer_Name | Customer_Phone |
|---|---|---|
| 11001 | Contoso, Ltd. | 555-555-0173 |
| 11002 | Litware, Inc. | 555-505-3124 |
| 11003 | ADatum Corporation | 555-689-4312 |

Você planeja implementar uma máscara de dados dinâmica para a coluna Customer_Phone. A máscara deve atender aos seguintes requisitos:
- Os primeiros seis números do número de telefone de cada cliente devem ser mascarados.
- Os últimos quatro dígitos do número de telefone de cada cliente devem ser visíveis.
- Os hifens devem ser preservados e exibidos.

Como você deve configurar a máscara de dados dinâmica? Para responder, selecione as opções apropriadas na área de resposta.

**Alternativas:**

**Prefixo exposto:**
- **A.** 0
- **B.** 1
- **C.** 3
- **D.** 5

**Corda de enchimento:**
- **A.** x
- **B.** xxxxxxxxx
- **C.** xxx-xxx
- **D.** xxx-xxx-
- **E.** x[3]-x[3]

**Sufixo exposto:**
- **A.** 0
- **B.** 1
- **C.** 3
- **D.** 5

**✅ Resposta:** Prefixo Exposto: 0; Corda Padding: xxx-xxx; Sufixo Exposto: 5

**Explicação:** A máscara de corda personalizada é `partial(prefix,"padding",suffix)` e mostra caracteres prefixo + preenchimento + sufixo. Com o prefixo 0, `555-555` é substituído pelo preenchimento `xxx-xxx`. Sufixo 5 expõe os últimos cinco caracteres, `-0173`, que inclui o hífen. Resultado: `xxx-xxx-0173`, que atende todas as três regras. `xxx-xxx-` precisaria do sufixo 4, que não é oferecido, e sufixo 5 com esse preenchimento produziria um hífen.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/dynamic-data-masking-overview) [Doc 2](https://learn.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking)

---

### Q97. SIM: e-mail de máscara em SalesLT.Customer

**Questão:**
SIMULATION. Você precisa garantir que quando os usuários não administrativos consultam a tabela SalesLT.Customer em db1, os endereços de e-mail são obscurecidos. Por exemplo, um endereço de e-mail do alice@contoso.com deve aparecer como aXXX@XXXX.com.
Para concluir esta tarefa, inicie sessão na máquina virtual. Poderá ter de utilizar o SQL Server Management Studio e o Azure portal.

**Alternativas:**

**Resposta ✅ (passos):** Use qualquer um dos métodos.
- Bancos de dados **Azure portal:** SQL > **db1** > **Segurança > Dynamic Data Masking** > **+ Adicionar máscara** > Esquema **VendasLT**, Tabela **Cliente**, Coluna **EmailAddress** > Formato de campo de mascaramento **E-mail (aXXX@XXXX.com)** > **Adicionar** > **Gravar**.
- **T-SQL em db1:**
```sql
ALTER TABLE SalesLT.Customer
ALTER COLUMN EmailAddress ADD MASKED WITH (FUNCTION = 'email()');
```
Os administradores (admin do servidor, admin de entrada, db_owner) ainda vêem os valores reais. Os não-admins veem `aXXX@XXXX.com`.

**Explicação:** A função DDM email() mostra a primeira letra e substitui o resto pelo padrão constante `XXX@XXXX.com`, que corresponde exatamente à saída necessária. (A coluna AdventureWorksLT é chamada EmailAddress.)

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/dynamic-data-masking-overview)

---

### Q98. Máscara de coluna de e-mail do portal

**Questão:**
Você tem um pool SQL no Azure Synapse que contém uma tabela chamada dbo.Customers. A tabela contém uma coluna chamada Email.
Você precisa evitar que usuários não administrativos vejam os endereços de e-mail completos na coluna E-mail. Os usuários devem ver valores em um formato de aXXX@XXXX.com.
O que você deve fazer?

**Alternativas:**
- **A.** A partir do Azure portal, defina uma máscara na coluna Email.
- **B.** A partir do Azure portal, defina uma classificação de sensibilidade de Confidencial para a coluna Email.
- **C.** A partir do Microsoft SQL Server Management Studio, defina uma máscara de e-mail na coluna E-mail.
- **D.** Da Microsoft SQL Server Management Studio, conceda a permissão SELECT aos usuários para todas as colunas na tabela dbo.Customers, exceto E-mail.

**✅ Resposta:** A

**Explicação:** Dynamic data masking com o formato de e-mail dá exatamente `aXXX@XXXX.com`, e é configurado no dedicado SQL da piscina **Dynamic Data Masking** A lâmina na Azure portal. Classificação (B) apenas rotula a coluna e não oculta dados. SSMS não tem designer de mascaramento (C). Uma máscara pode ser adicionada a partir de SSMS apenas digitando T-SQL `ALTER COLUMN … ADD MASKED`que não é o que a opção descreve, então a chave do exame é A. nível de coluna GRANT (D) esconde a coluna inteiramente em vez de mostrar o formato mascarado.
*Observação:* o produto é Synapse, mas o mesmo comportamento se aplica ao Azure SQL Database (lâmina portal Dynamic Data Masking, máscara de e-mail).

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/dynamic-data-masking-overview) [Doc 2](https://learn.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking)

---

### Q99. Criptografia de nível celular da sequência Col1

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1. DB1 contém uma tabela que tem uma coluna chamada Col1. Você precisa criptografar os dados em Col1. Quais quatro ações você deve executar para DB1 em sequência? (Mova as ações apropriadas para a área de resposta e organize-as na ordem correta.)

**Alternativas:**
Acções:
- **A.** Crie uma chave mestra do banco de dados.
- **B.** Crie uma chave mestra de coluna.
- **C.** Abra a chave simétrica.
- **D.** Crie um certificado.
- **E.** Atualização Col1.
- **F.** Crie uma chave simétrica.

**✅ Resposta:** 1. Criar uma chave mestre de banco de dados → 2. Criar um certificado → 3. Criar uma chave simétrica → 4. Abrir a chave simétrica

**Explicação:** A criptografia simétrica de nível celular (nível de coluna) precisa de uma DMK (para proteger a chave privada do certificado), um certificado, uma chave simétrica criptografada por esse certificado, e depois `OPEN SYMMETRIC KEY ... DECRYPTION BY CERTIFICATE` antes `EncryptByKey()` Pode ser utilizado em um UPDATE. "Criar uma chave mestre coluna" pertence a Always Encrypted (uma característica diferente, chaves mantidas fora do banco de dados), por isso é um distrator. "Atualizar Col1"" é o passo que escreve dados criptografados após a chave estar aberta; com apenas quatro slots e nenhuma DMK como já existente, a sequência aceita começa com o DMK e termina com a abertura da chave.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/encrypt-a-column-of-data)

---

### Q100. Série Firewall: regras de nível de banco de dados em cada DB

**Questão:**
Você tem as redes on-premises mostradas na tabela a seguir.

| Nome | Localização | Sub-rede |
|---|---|---|
| Branch1 | Sydney Sydney | 131.107.10.0/24 |
| Branch2 | Melbourne | 131.107.11.0/24 |

Você tem uma assinatura do Azure que contém um servidor Azure SQL Database chamado SQL1. SQL1 contém duas bancos de dados chamadas DB1 e DB2. Você precisa configurar o acesso a DB1 e DB2. A solução deve atender aos seguintes requisitos:
- Certifique-se de que o DB1 só pode ser acessado por usuários no Branch1.
- Certifique-se de que o DB2 só pode ser acessado por usuários no Branch2.

Solução: Você se conecta ao DB1 e corre:
```sql
EXECUTE sp_set_database_firewall_rule 'Allow db1 users', '131.107.10.0', '131.107.10.255';
```
Você se conecta ao DB2 e corre:
```sql
EXECUTE sp_set_database_firewall_rule 'Allow db2 users', '131.107.11.0', '131.107.11.255';
```
Isso cumpre o objetivo?

**Alternativas:**
- **A.** Sim
- **B.** Não

**✅ Resposta:** A (Sim)

**Explicação:** As regras de firewall IP de nível de banco de dados são criadas com `sp_set_database_firewall_rule` enquanto conectadas ao banco de dados específico do usuário, e elas concedem acesso apenas a esse banco de dados. Executá-lo em DB1 para 131.107.10.0/24 e em DB2 para 131.107.11.0/24 isola cada banco de dados para sua filial sem uma regra de nível de servidor que abriria ambas Os bancos de dados.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure)

---

### Q101. Série de firewall: sp_set_firewall_rule executado em DB1

**Questão:**
(O mesmo cenário que Q100: Branch1 Sydney 131.107.10.0/24; Branch2 Melbourne 131.107.11.0/24; SQL1 hospeda DB1 e DB2; DB1 deve ser acessível apenas por usuários Branch1, DB2 apenas por usuários Branch2.)

Solução: Você se conecta ao DB1 e corre:
```sql
EXECUTE sp_set_firewall_rule 'Allow db1 users', '131.107.10.0', '131.107.10.255';
```
Você se conecta ao DB2 e corre:
```sql
EXECUTE sp_set_database_firewall_rule 'Allow db2 users', '131.107.11.0', '131.107.11.255';
```
Isso cumpre o objetivo?

**Alternativas:**
- **A.** Sim
- **B.** Não

**✅ Resposta:** B (Não)

**Explicação:** `sp_set_firewall_rule` cria regras de nível de servidor e só pode ser executado no banco de dados `master`. Execute em DB1 ele falha, e mesmo no mestre que abriria todos os bancos de dados em SQL1 (incluindo DB2) para Branch1. Apenas a metade DB2 da solução está correta.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure)

---

### Q102. Série Firewall: Regras "Negar" em DB1 / DB2

**Questão:**
(O mesmo cenário como Q100.)

Solução: Você se conecta ao DB1 e corre:
```sql
EXECUTE sp_set_firewall_rule 'Deny db1 users', '131.107.11.0', '131.107.11.255';
```
Você se conecta ao DB2 e corre:
```sql
EXECUTE sp_set_database_firewall_rule 'Deny db2 users', '131.107.10.0', '131.107.10.255';
```
Isso cumpre o objetivo?

**Alternativas:**
- **A.** Sim
- **B.** Não

**✅ Resposta:** B (Não)

**Explicação:** As regras de firewall Azure SQL IP são apenas permitidas; nomear uma regra "Deny..." não faz dela uma regra de negação. A regra DB2 realmente permitiria Branch1 em DB2, o inverso do requisito. Além disso, `sp_set_firewall_rule` é um procedimento no nível do servidor que funciona apenas em `master`, então o comando DB1 falha.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure)

---

### Q103. Série de firewall: regra do servidor mestre 131.107.10.0/24

**Questão:**
(O mesmo cenário como Q100.)

Solução: Você se conecta ao banco de dados mestre do SQL1 e executa:
```sql
EXECUTE sp_set_firewall_rule 'Allow db1 and db2 users', '131.107.10.0', '131.107.10.255';
```
Isso cumpre o objetivo?

**Alternativas:**
- **A.** Sim
- **B.** Não

**✅ Resposta:** B (Não)

**Explicação:** Uma regra de nível de servidor concede o acesso de intervalo a todas Os bancos de dados no SQL1. Branch1 pode chegar ao DB2 e o Branch2 não tem acesso. Em vez disso, use o `sp_set_database_firewall_rule` em cada banco de dados.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure)

---

### Q104. Série de firewall: regra do servidor mestre 131.107.11.0/24

**Questão:**
(O mesmo cenário como Q100.)

Solução: Você se conecta ao banco de dados mestre do SQL1 e executa:
```sql
EXECUTE sp_set_firewall_rule 'Allow db1 and db2 users', '131.107.11.0', '131.107.11.255';
```
Isso cumpre o objetivo?

**Alternativas:**
- **A.** Sim
- **B.** Não

**✅ Resposta:** B (Não)

**Explicação:** A regra de nível do servidor permite que o Branch2 alcance tanto o DB1 quanto o DB2 e não dá acesso ao Branch1, portanto, nenhum requisito é cumprido. As regras por banco de dados (`sp_set_database_firewall_rule`) são necessárias.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure)

---

### Q105. Regra Azure CLI que permite serviços Azure

**Questão:**
Você tem uma assinatura do Azure que contém um grupo de recursos chamado RG1. RG1 contém um servidor Azure SQL chamado Server1 na região West US Azure. Você precisa garantir que todos os serviços do Azure implantados no RG1 possam acessar o Server1. Como você deve concluir o comando Azure CLI?
```
az sql server [Box1] create \
  --resource-group rg1 \
  --server server1 \
  --name AllowAzureServices \
  [Box2]
```

**Alternativas:**

**Caixa 1:**
- **A.** firewall-rule
- **B.** ipv6-firewall-rule
- **C.** outbound-firewall-rule
- **D.** vnet-rule

**Caixa 2:**
- **A.** --outbound-rule-fqdn permitidoFQDN
- **B.** --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
- **C.** --start-ip-address 13.86.217.0 --end-ip-address 13.86.217.255
- **D.** --start-ipv6-address 9a41:a145:2a80:6c8d:4628:a1b3:5812:3283 --end-ipv6-address 9a41:a145:2a80:6c8d:4628:a1b3:5812:3283

**✅ Resposta:** Caixa 1: regra de firewall; Caixa 2: --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

**Explicação:** An IPv4 regra de firewall do servidor com endereço de início e fim 0.0.0.0 é a regra especial (mostrada no portal como "Permitir que os serviços e recursos do Azure acessem este servidor", AllowAllWindowsAzureIps) que permite que os serviços hospedados pelo Azure se conectem. vnet-rule needs a subnet/service endpointNota: os médicos alertam que esta regra admite todos os serviços do Azure, incluindo assinaturas de outros clientes, e não apenas RG1. Ainda é a única opção aqui que atende ao requisito.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure)

---

### Q106. Acesso DB1 limitado a 10 VMs em Subnet1

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1. Você tem 10 Azure virtual machines que se conectam a uma sub-rede de rede virtual chamada Subnet1. Você precisa implementar um firewall de nível de banco de dados que atenda aos seguintes requisitos:
- Garante que apenas as 10 máquinas virtuais podem acessar o DB1.
- Segue o princípio do menor privilégio.

Como você deve configurar a regra de firewall, e como você deve estabelecer conectividade de rede das máquinas virtuais para DB1?

**Alternativas:**

**Regra de firewall:**
- **A.** Adicione o endereço IPv4 do seu cliente.
- **B.** Permitir que os serviços e recursos do Azure acessem o DB1.
- **C.** Permitir o tráfego de uma rede virtual específica.

**Conectividade de rede:**
- **A.** Atribua endereços públicos estáticos IP às máquinas virtuais.
- **B.** Crie um private endpoint.
- **C.** Crie um service endpoint.

**✅ Resposta:** Regra de firewall: **Permitir o tráfego de uma rede virtual específica**; Conectividade de rede: **Criar um service endpoint**

**Explicação:** Subnet1 contém exatamente o 10 VMsuma regra de rede virtual para Subnet1 admitem apenas aqueles VMs (privilégio mínimo) sem rastreamento individual IPsA VNet A regra exige a **Microsoft.Sql service endpoint** na sub-rede: "para criar uma regra de rede virtual, deve haver primeiro uma rede virtual service endpoint". As outras opções falham. "Permitir serviços Azure" admite todos os recursos do inquilino Azure. Um cliente-IPv4 regra não corresponde VM Quando o tráfego utiliza a service endpointe público estático IPs mais IP as regras adicionam sobrecarga de gestão. A private endpoint Não usa regras de firewall.
Caveat: VNet regras são tecnicamente **servidor-nível**, enquanto o tronco diz "firewall de nível de banco de dados". O caule está solto, mas este é o único emparelhamento consistente entre as opções.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Baixa · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/vnet-service-endpoint-rule-overview) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure)

---

### Q107. Declarações de tabela Ledger + SYSTEM_VERSIONING

**Questão:**
Você tem um servidor lógico Azure SQL. Você executa o script a seguir.
```sql
CREATE DATABASE Sales
GO
CREATE TABLE [dbo].[Orders]
  (
    [OrderID] INT NOT NULL,
    [OrderDescription] NVARCHAR (MAX) NOT NULL,
    [Timestamp] Datetime2 NOT NULL
  )
  WITH (
    SYSTEM_VERSIONING = ON,
    LEDGER = ON
  );
GO
```
Para cada uma das seguintes afirmações, selecione Sim se a afirmação for verdadeira. Caso contrário, selecione Não.

**Alternativas:**
Declarações:
1. A tabela Pedidos permitirá que apenas as linhas sejam inseridas.
2. Para criar tabelas adicionais no banco de dados de vendas, o LEDGER = ON parâmetros devem ser utilizados.
3. Para garantir que um carimbo de data/hora seja adicionado a cada linha na tabela Pedidos, a coluna GENERATED ALWAYS deve ser adicionada ao script.

**✅ Resposta:** 1: Não; 2: Não; 3: Sim

**Explicação:** (1) Sem `APPEND_ONLY = ON`, `SYSTEM_VERSIONING = ON, LEDGER = ON` cria um *updatable* ledger table, então atualizações e exclusões são permitidas e versões antigas vão para a tabela de histórico. (2) O banco de dados não foi criado `WITH LEDGER = ON`, por isso não é um banco de dados de contabilidade. Outras tabelas podem ser tabelas comuns e LEDGER = ON só é necessário se você quiser que eles sejam ledger tables. (3) O livro-razão das colunas adiciona automaticamente são transação IDs and sequence numbers (bigint), not timestamps. The `[Timestamp]` A coluna é uma coluna comum que o aplicativo deve preencher. Para ter o carimbo do sistema cada linha, você adiciona um `datetime2 GENERATED ALWAYS AS ROW START/END` período (a tabela pode ser tanto ledger quanto temporal).

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/ledger/ledger-updatable-ledger-tables) [Doc 2](https://learn.microsoft.com/en-us/sql/relational-databases/security/ledger/ledger-limits)

---

### Q108. Desenvolvedores remotos ADatum acessam Instance1

**Estudo de caso:** ADatum v1 (serviços financeiros)

**Questão:**
(Processo: ADatum planeja implantar SQL managed instance Instance1 para Network1 (East US) e migrar DB1 / DB2 para ele. Depois, o desenvolvimento de banco de dados é entregue aos desenvolvedores remotos que usam Microsoft Entra se juntou aos dispositivos Windows 11. Requisitos de segurança: "Garantir que apenas os desenvolvedores designados que usam Microsoft Entra se juntaram aos dispositivos Windows 11 possam acessar remotamente DB1 e DB2".

Você precisa recomendar uma solução que permitirá que desenvolvedores remotos acessem DB1 e DB2. A solução deve apoiar as mudanças planejadas e atender aos requisitos de segurança. O que você deve incluir na recomendação?

**Alternativas:**
- **A.** um ponto de extremidade público através de uma regra de firewall em nível de banco de dados
- **B.** Ponto-a-Site (P2S) VPN
- **C.** um ponto de extremidade público através de uma regra de firewall no nível do servidor
- **D.** private endpoint

**✅ Resposta:** B

**Explicação:** Os desenvolvedores são clientes individuais remotos fora do Azure. A P2S VPN E- Network1 permite que cada dispositivo alcance Instance1's VNet-local endpoint. Com Microsoft Entra ID Autenticação (Azure VPN Cliente no Windows) e Acesso Condicional, pode ser limitado a usuários designados em dispositivos conectados/compatíveis. private endpoint apenas projetos de MI em outro Azure VNet, então laptops remotos ainda precisam de um VPN A e C não se aplicam a SQL Managed Instance, que não tem servidor ou banco de dados IP regras de firewall (seu endpoint público é controlado por NSG), e um IP Não é possível identificar o usuário ou dispositivo.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/vpn-gateway/point-to-site-entra-gateway) [Doc 2](https://learn.microsoft.com/en-us/azure/vpn-gateway/point-to-site-about)

---

### Q109. Power BI a MI: endpoint público + NSG

**Questão:**
Você tem uma assinatura do Azure que contém um Azure SQL managed instance chamado SQL1. Você usa o serviço Microsoft Power BI. Você precisa configurar a conectividade do serviço Power BI para SQL1. A solução deve garantir que apenas o serviço Power BI possa iniciar conexões com SQL1. Que tipo de endpoint você deve usar para SQL1, e o que você deve usar para controlar as conexões com SQL1?

**Alternativas:**

**Tipo de endpoint:**
- **A.** Privado Privado
- **B.** Público
- **C.** Serviço

**Controle de conexões usando:**
- **A.** Um firewall de banco de dados
- **B.** A network security group (NSG)
- **C.** Um firewall de nível servidor
- **D.** Regras de firewall de rede virtual

**✅ Resposta:** Tipo de endpoint: Público; Controle conexões usando: A network security group (NSG)

**Explicação:** O Power BI é um serviço de nuvem multilocatário fora do seu VNet, então (sem um gateway) atinge a managed instance através da MI endpoint público (port 3342). On MI, o acesso a esse endpoint é controlado por inbound NSG regras sobre o MI subnet. Use uma regra Permitir para TCP 3342 com tag de serviço de origem `PowerBI`, então apenas Power BI pode iniciar conexões. MI não tem servidor/nível de banco de dados IP firewall ou VNet As regras de firewall (estas são Azure SQL Database características), e MI não usa "serviço" terminais.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/public-endpoint-configure?view=azuresql) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/public-endpoint-overview?view=azuresql)

---

### Q110. Acesso privado IP e único- VNet: Private Link

**Questão:**
Você planeja implantar um aplicativo que inclua um Azure SQL database e um aplicativo da Web do Azure. O aplicativo tem os seguintes requisitos:
- O aplicativo da web deve ser hospedado em um Azure virtual network.
- O Azure SQL database deve receber um endereço IP privado.
- O Azure SQL database deve permitir conexões somente de uma rede virtual específica.

Você precisa recomendar uma solução que atenda aos requisitos. O que você deve incluir na recomendação?

**Alternativas:**
- **A.** Azure Private Link
- **B.** a network security group (NSG)
- **C.** um firewall de nível de banco de dados
- **D.** um firewall de nível servidor

**✅ Resposta:** A

**Explicação:** Azure Private Link cria a private endpoint, a NIC com um privado IP a partir de uma sub-rede do escolhido VNetCom o acesso à rede pública desabilitado, a banco de dados só é acessível através dessa VNet (e redes conectadas a ele). NSGs filtro de sub-rede de tráfego, mas não pode dar o banco de dados um privado IP. Servidor e banco de dados IP as regras de firewall agem sobre o ponto final público e não atribuir um IP.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/private-endpoint-overview?view=azuresql)

---

### Q111. SIM: conecte-se ao db1 através do IP privado em VNET1

**Questão:**
SIMULATION. Você precisa garantir que você pode se conectar ao db1 usando um endereço IP privado em uma rede virtual chamada VNET1. Você pode precisar usar SQL Server Management Studio e o Azure portal.

**Alternativas:**

**✅ Resposta:** Crie um private endpoint para o servidor lógico que hospeda o db1, em uma sub-rede de VNET1.
1. Azure portal → abrir o **SQL server** (servidor lógico) que hospeda db1 → **Security › Networking** → **Acesso privado** guia → **+ Criar um private endpoint**. (Equivalente: pesquisa **Private endpoints** → **+ Criar**.)
2. Assinatura **Noções básicas:** e grupo de recursos (igual ao do servidor), um nome como `db1-pe`, e a região de VNET1.
3. **Recurso:** Tipo de recurso **Microsoft.Sql/servidores**, Resource = the server hosting db1, Sub-recurso alvo **sqlServer**.
4. Rede virtual **Rede Virtual:** **VNET1**, escolha uma sub-rede (a alocação dinâmica IP é boa).
5. **DNS:** Integrar-se ao privado DNS zone = **Sim** (`privatelink.database.windows.net`) assim `<server>.database.windows.net` resolve para o privado IP dentro VNET1.
6. **Revisão + criar** → **Criar**. Aguarde até que o estado de conexão do ponto final seja **Aprovado** (aprovado automaticamente porque você possui o servidor).
7. Na página Networking do servidor, defina **Acesso à rede pública** para **Desativar** para que apenas as conexões privadas funcionem.
8. Verifique: de um VM em VNET1, `nslookup <server>.database.windows.net` retorna um IP privado; conecte-se com SSMS para `<server>.database.windows.net` / db1.

**Explicação:** private endpoint (Azure Private Link) atribui ao servidor lógico um IP privado de VNET1. Uma regra service endpoint ou VNet ainda usaria o servidor público IP.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/private-endpoint-overview?view=azuresql)

---

### Q112. VM1 pode alcançar apenas SqlSrv1: o que criar no VNet1

**Questão:**
Você tem um Azure virtual machine chamado VM1 em uma rede virtual chamada VNet1. O tráfego de saída do VM1 para a internet está bloqueado. Você tem um banco de dados do Azure SQL Database chamado SqlDb1 em um servidor lógico chamado SqlSrv1. Você precisa implementar conectividade entre VM1 e SqlDb1 para atender aos seguintes requisitos:
- Certifique-se de que o VM1 não pode se conectar a nenhum servidor Azure SQL além do SqlSrv1.
- Restringir a conectividade de rede ao SqlSrv1.

O que você deve criar no VNet1?

**Alternativas:**
- **A.** gateway VPN
- **B.** service endpoint
- **C.** private endpoint
- **D.** e ExpressRoute gateway

**✅ Resposta:** C

**Explicação:** A private endpoint mapas de um recurso específico (SqlSrv1) em VNet1. Com internet outbound bloqueado, VM1 só pode chegar ao servidor, que protege contra a exfiltração de dados para outros servidores. A Microsoft.Sql service endpoint abre uma rota para todos Azure SQL servidores na região, não apenas SqlSrv1. VPN e ExpressRoute Os gateways são para on-premisesConectividade entre redes.

**Variante vista em outros dumps:** mesma pergunta com opções `A. a VPN gateway / B. a service endpoint / C. a private link / D. an ExpressRoute gateway` → resposta **C**.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/private-endpoint-overview?view=azuresql)

---

### Q113. Bloquear endpoint público, minimizar a exfiltração: Private Link

**Questão:**
Você tem um Azure virtual machine chamado VM1 em uma rede virtual chamada VNet1. O tráfego de saída do VM1 para a internet está bloqueado. Você tem um banco de dados do Azure SQL Database chamado SqlDb1 em um servidor lógico chamado SqlSrv1. Você precisa implementar conectividade entre VM1 e SqlDb1 para atender aos seguintes requisitos:
- Certifique-se de que todo o tráfego para o ponto final público do SqlSrv1 esteja bloqueado.
- Minimizar a possibilidade de VM1 exfiltrating dados armazenados em SqlDb1.

O que você deve criar no VNet1?

**Alternativas:**
- **A.** gateway VPN
- **B.** service endpoint
- **C.** private link
- **D.** e ExpressRoute gateway

**✅ Resposta:** C

**Explicação:** Com Private Link, VM1 alcances SqlSrv1 Através de a private endpointpara que o acesso à rede pública do servidor possa ser definido como **Negar/Desativado**, bloqueando todo o tráfego de endpoint público. Porque os mapas de endpoint apenas SqlSrv1, VM1 não pode empurrar dados para outros SQL servidores (protecção de filtragem). A service endpoint ainda usa o ponto de extremidade público do servidor e permite que cada SQL server na região.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/private-endpoint-overview?view=azuresql)

---

### Q114. VM1 (West US 2) a MI1 (East US) baixa latência

**Questão:**
Você tem uma assinatura do Azure que contém os recursos mostrados na tabela a seguir.

| Nome | Tipo | região Azure |
|---|---|---|
| VM1 | Azure virtual machine | US 2 |
| MI1 | Azure SQL Managed Instance | Leste US |

Você precisa configurar uma conexão entre VM1 e MI1. A solução deve atender aos seguintes requisitos:
- A conexão deve ser criptografada.
- A latência da rede deve ser minimizada.

O que você deve implementar?

**Alternativas:**
- **A.** peering de rede virtual
- **B.** private endpoints
- **C.** service endpoints
- **D.** a site-a-site VPN

**✅ Resposta:** A

**Explicação:** O peering de rede virtual global conecta os dois VNets sobre o backbone da Microsoft, então os médicos dizem que "não há penalidade de latência perceptível". Os clientes podem usar MI's VNet-local com o tipo de conexão de redirecionamento de baixa latência, e o TDS A própria sessão é TLS- Criptografado. Private endpoints E- MI Use sempre o proxy tipo de conexão (pulso extra), então a latência é maior. Um site-a-site VPN adiciona gateway/IPsec e destina-se a on-premises links. Service endpoints não se aplica a SQL Managed Instance.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/connect-application-instance) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/private-endpoint-overview?view=azuresql)

---

### Q115. Clientes da Contoso criam seus próprios objetos

**Estudo de caso:** Contoso

**Questão:**
(O objetivo do caso de negócio: "Garantir que os usuários da solução PaaS possam criar seus próprios objetos de banco de dados, mas sejam impedidos de modificar qualquer um dos objetos de banco de dados existentes fornecidos pela Contoso." Além disso: use recursos internos, minimize o esforço de desenvolvimento.)

Você precisa recomendar uma solução para garantir que os clientes possam criar os objetos do banco de dados. A solução deve atender aos objetivos de negócios. O que você deve incluir na recomendação?

**Alternativas:**
- **A.** Para cada cliente, conceda ao cliente ddl_admin o esquema existente.
- **B.** Para cada cliente, crie um esquema adicional e conceda ao cliente ddl_admin o novo esquema.
- **C.** Para cada cliente, crie um esquema adicional e conceda ao cliente db_writer o novo esquema.
- **D.** Para cada cliente, conceda ao cliente db_writer o esquema existente.

**✅ Resposta:** B

**Explicação:** Os clientes precisam DDL direitos (CREATE/ALTER objetos), que db_datawriter não dá: só permite INSERT/UPDATE/DELETE. Então C e D estão fora. Concessão DDL Os direitos sobre o esquema existente (Contoso) permitiriam que alterassem ou abandonassem os objetos da Contoso (A). Um esquema separado por cliente, de propriedade ou com ALTER/CREATE direitos para esse cliente, isola seus objetos de Contoso. (Estritamente, db_ddladmin é um papel em toda a banco de dados, então na prática você concede CREATE TABLE etc. mais ALTER no novo esquema, mas B é a opção pretendida.)

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles)

---

### Q116. atribuições de papel Contoso DBAGroup1 / DBAGroup2

**Estudo de caso:** Contoso

**Questão:**
Para o servidor Azure SQL Database de cada cliente, você planeja essas atribuições (escopo: Este recurso):

| Papel | Nome | Tipo | Scope |
|---|---|---|---|
| Contribuidor | DBAGroup1 | Grupo | Este recurso |
| Contribuidor SQL DB | DBAGroup2 | Grupo | Este recurso |

Para cada uma das seguintes afirmações, selecione Sim se a afirmação for verdadeira. Caso contrário, selecione Não.

**Alternativas:**
Declarações:
1. O DBAGroup1 poderá entrar no Azure SQL database de cada cliente usando o Azure Data Studio.
2. O DBAGroup1 poderá atribuir o papel de Colaborador SQL DB a outros usuários.
3. O DBAGroup2 será capaz de criar um novo Azure SQL database no servidor Azure SQL Database de cada cliente.

**✅ Resposta:** 1: Não; 2: Não; 3: Sim

**Explicação:** (1) Azure RBAC As funções atuam apenas no plano de controle. Entrar no banco de dados precisa de um SQL login/contained user ou o administrador do Entra, que o Contribuinte não concede. (2) O Contribuidor exclui `Microsoft.Authorization/*/Write`, por isso não pode criar atribuições de função (que precisa do proprietário ou administrador de acesso do usuário). (3) SQL DB Contribuidor inclui `Microsoft.Sql/servers/databases/*` (“Criar e gerenciar SQL bancos de dados”) e é atribuído no escopo do servidor, então DBAGroup2 pode criar bancos de dados nesse servidor (mas não acessar seus dados).

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/databases)

---

### Q117. Entre User1 adicionar/remover colunas, menos privilégio

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1. Um usuário chamado User1 tem uma conta Microsoft Entra. Você precisa fornecer User1 com a capacidade de adicionar e remover colunas das tabelas em DB1. A solução deve usar o princípio do menor privilégio. Quais duas ações você deve executar? Cada resposta correta apresenta parte da solução.

**Alternativas:** *(algumas opções reconstruídas)*
- **A.** Atribua ao usuário do banco de dados a função db_owner.
- **B.** Atribua ao usuário do banco de dados a função db_ddladmin.
- **C.** Crie um contained database user.
- **D.** Crie um login e um usuário de banco de dados associado.

**✅ Resposta:** B e C

**Explicação:** Para uma identidade intra em Azure SQL Database, você cria a contained database user E- DB1 (`CREATE USER [user1@contoso.com] FROM EXTERNAL PROVIDER;`). Nenhum servidor login é necessário mestre, o que mantém o acesso DB1 Apenas (menos privilégios do que D). Adicionar ou soltar colunas é DDL (`ALTER TABLE`), que db_ddladmin permite (`ALTER ROLE db_ddladmin ADD MEMBER [user1@contoso.com];`). db_owner (CONTROL DATABASE) dá muito mais do que o necessário, o que viola o menor privilégio.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles)

---

### Q118. Representação do teste com EXECUTE AS USER / REVERT

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado db1 que contém um usuário Microsoft Entra (Azure AD) chamado user1. Você precisa testar a personificação do user1 em db1 executando uma declaração SELECT e retornando ao contexto de execução original. Como você deve completar a declaração Transact-SQL?

```sql
EXECUTE AS [Box 1] = 'user1@contoso.com'
GO
SELECT SUSER_SNAME()
[Box 2]
GO
```

**Alternativas:**

**Caixa 1:**
- **A.** CALLER
- **B.** LOGIN
- **C.** OWNER
- **D.** USER

**Caixa 2:**
- **A.** REVERT
- **B.** REVOKE
- **C.** ROLLBACK

**✅ Resposta:** Caixa 1: USER; Caixa 2: REVERT

**Explicação:** Em Azure SQL Database, EXECUTE AS LOGIN não está disponível (a personificação do escopo do servidor não é suportada), de modo que um usuário do banco de dados é representado com EXECUTE AS USER. REVERT Muda o contexto de volta para o chamador anterior. CALLER não tem efeito fora de um módulo; OWNER válida apenas em um módulo WITH EXECUTE AS cláusula; REVOKE remove permissões e ROLLBACK desfaz uma transação – nem redefine o contexto de execução.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/t-sql/statements/execute-as-transact-sql)

---

### Q119. Grupo de Subsídios lido em tabelas futuras (esquema)

**Questão:**
Você tem uma assinatura do Azure que contém um grupo chamado Group1 e um Azure SQL managed instance que hospeda um banco de dados chamado DB1. Você precisa garantir que o Group1 tenha acesso de leitura a novas tabelas criadas no DB1. A solução deve usar o princípio do menor privilégio. Como você deve completar a declaração Transact-SQL?

```sql
GRANT [Box 1] ON [Box 2] :: [dbo] TO [Contoso\group1]
```
(A fonte transcreve o alvo como `::[table]`; o securável após `::` é o nome do esquema.)

**Alternativas:**

**Caixa 1:**
- **A.** DELETE
- **B.** INSERT
- **C.** SELECT
- **D.** UPDATE

**Caixa 2:**
- **A.** DATABASE
- **B.** SCHEMA
- **C.** TABLE

**✅ Resposta:** Caixa 1: SELECT; Caixa 2: SCHEMA

**Explicação:** SELECT é a permissão de leitura. Uma permissão concedida em um esquema aplica-se a todos os objetos desse esquema, incluindo tabelas criadas mais tarde, para que atenda ao requisito de "novas tabelas". Uma concessão de nível de tabela não cobriria novas tabelas. Uma concessão de nível de banco de dados também as cobriria, mas é mais ampla do que o necessário, portanto, o SCHEMA é o escopo menos privilegiado.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/t-sql/statements/grant-schema-permissions-transact-sql)

---

### Q120. SIM: papel que lê apenas uma tabela

**Questão:**
SIMULATION. Você precisa criar uma nova função de banco de dados chamada role1 que possa ler o conteúdo da tabela SalesLT.SalesOrderDetail apenas em db1. Para concluir essa tarefa, entre na máquina virtual. Você pode precisar usar SQL Server Management Studio e o Azure portal.

**Alternativas:** Nenhum (tarefa de laboratório manual)

**✅ Resposta:** Crie um papel **banco de dados** em db1 e conceda-lhe SELECT nessa única tabela:
1. Em SSMSConecte-se ao servidor lógico e abra uma nova janela de consulta **db1** (não mestre).
2. Executar:
```sql
CREATE ROLE role1;
GRANT SELECT ON OBJECT::SalesLT.SalesOrderDetail TO role1;
```
3. (GUI alternativa) db1 Segurança > Segurança > Funções > Funções de banco de dados > Nova função de banco de dados. role1, vá para Securables > Busca > Objetos específicos > Tabelas > SalesLT.SalesOrderDetail, tick **Subvenção** E **Selecionar**então, OK.
4. Verificação opcional: `SELECT * FROM sys.database_permissions WHERE grantee_principal_id = DATABASE_PRINCIPAL_ID('role1');`

**Explicação:** Uma função de banco de dados definida pelo usuário com uma concessão SELECT de nível de objeto dá acesso de leitura a essa tabela e nada mais. Não adicione o papel ao db_datareader, porque isso concede lido em todas as tabelas. Um papel de servidor é o tipo errado de objeto: é mapeado por servidor e a tarefa pede um papel dentro do db1. O Azure SQL Database também oferece apenas funções de servidor fixo, não definidas pelo usuário.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles)

---

### Q121. Permitir que o usuário mestre crie bancos de dados

**Questão:**
Você tem uma assinatura do Azure que contém um SQL server lógico chamado Server1. O banco de dados mestre do Server1 contém um usuário chamado User1. Você precisa garantir que o User1 possa criar bancos de dados no Server1. Qual função de banco de dados você deve atribuir ao User1?

**Alternativas:**
- **A.** db_owner
- **B.** dbmanager
- **C.** dbo
- **D.** db_ddladmin

**✅ Resposta:** B

**Explicação:** dbmanager é um papel especial que existe apenas no banco de dados mestre virtual do Azure SQL Database. Seus membros podem criar e excluir bancos de dados, e o criador se torna o dbo de cada novo banco de dados. db_owner e db_ddladmin são funções por banco de dados e não concedem CREATE DATABASE no servidor lógico. dbo é um usuário, não um papel. Nota: o usuário deve ser login -based, porque o fazer

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles)

---

### Q122. Capacidades db_datawriter e papel de leitura

**Questão:**
Você tem um banco de dados Microsoft SQL Server chamado DB1 que contém uma tabela chamada Table1. A associação à função de banco de dados para um usuário chamado User1 é mostrada na exposição.

Exposição (SSMS "Database User - User1" > Associação): **Verificado: db_datawriter somente**. Desmarcado: db_accessadmin, db_backupoperator, db_datareader, db_ddladmin, db_denydatareader, db_denydatawriter, db_owner, db_securityadmin, além de papéis personalizados (Vendas Externas, Vendas do Extremo Oeste, Grandes Lagos Vendas, Vendas do Oriente Médio, Vendas da Nova Inglaterra, Vendas Planícies, Vendas Rocky Mountain, SalesSchemaDeny, SalesSchemaRead, Vendas do Sudeste, Vendas do Sudoeste, WebsiteExecute).

Use os menus suspensos para selecionar a opção de resposta que completa cada declaração.

**Alternativas:**

**Declaração 1: User1 pode ____.**
- **A.** Adicionar uma coluna ao Table1
- **B.** Apagar uma linha de Table1
- **C.** drop Table1

**Declaração 2: Para garantir que o User1 possa executar consultas para recuperar dados do DB1, você deve atribuir ao User1 a função de banco de dados ____.**
- **A.** db_datareader
- **B.** db_ddladmin
- **C.** db_denydatareader
- **D.** db_denydatawriter

**✅ Resposta:** Declaração 1: delete uma linha de Table1; Declaração 2: db_datareader

**Explicação:** db_datawriter Subvenções INSERT, UPDATE e DELETE em todas as tabelas de usuário, então User1 Adicionar uma coluna ou deixar cair a tabela é DDL, que precisa db_ddladmin, ALTER : CONTROL. db_datareader Subvenções SELECT em todas as tabelas de usuário e visualizações. db_denydatareader bloquearia as leituras, e db_ddladmin não concede SELECT.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles)

---

### Q123. Contoso: acesso a tabelas e linhas em camadas

**Estudo de caso:** Contoso

**Questão:**
(Estudo de caso Contoso: cada cliente obtém seu próprio Azure SQL database com uma cópia completa dos dados financeiros. Os dados que um cliente pode acessar dependem do nível do contrato de serviço e os clientes podem mudar de nível. Exigência comercial: "Fornecer aos clientes acesso diferente à tabela e à linha com base no contrato de serviço do cliente.")
Você está avaliando os objetivos de negócios. Qual recurso você deve usar para fornecer aos clientes o nível necessário de acesso com base em seu contrato de serviço?

**Alternativas:**
- **A.** dynamic data masking
- **B.** Acesso Condicional no Azure
- **C.** service principals
- **D.** row-level security (RLS)

**✅ Resposta:** D

**Explicação:** O requisito é limitar quais linhas (e tabelas) cada cliente pode ver em um banco de dados que contém o conjunto completo de dados. RLS filtra as linhas através de uma política de segurança e função de predicado. O acesso ao nível da tabela é tratado por permissões comuns de objetos ao lado dele. Dynamic data masking apenas ofusca os valores das colunas nos resultados da consulta e não restringe as linhas. As condições de login dos controles de acesso condicional e service principals são identidades, portanto, nenhum deles restringe os dados dentro do banco de dados.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/row-level-security)

---

### Q124. RLS em Pedidos: política mais TVF

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1 que contém uma tabela chamada Orders. A tabela Orders contém uma linha para cada ordem de vendas. Cada pedido de venda inclui o nome do usuário que colocou a ordem. Você precisa implementar row-level security (RLS). A solução deve garantir que os usuários possam visualizar apenas suas respectivas ordens de vendas. O que você deve incluir na solução?

**Alternativas:**

**Caixa 1 – Criar:**
- **A.** Uma visão materializada em DB1
- **B.** Uma política de segurança na tabela Pedidos
- **C.** Credenciais com escopo de banco de dados em DB1

**Caixa 2 – Controle o acesso às linhas usando:**
- **A.** Uma regra de mascaramento
- **B.** Uma função de tabela-valor
- **C.** predicado CONTAINS

**✅ Resposta:** Caixa 1: Uma política de segurança na tabela Pedidos; Caixa 2: Uma função com valor de tabela

**Explicação:** RLS é implementado com CREATE SECURITY POLICY, que adiciona um predicado de filtro à tabela Pedidos. O predicado é uma função de valor de tabela em linha que compara o nome de usuário da linha com USER_NAME() ou SESSION_CONTEXTUma regra de mascaramento é dynamic data masking, que esconde valores de coluna, mas nunca remove linhas. CONTAINS é um predicado de pesquisa de texto completo. Vistas materializadas e credenciais com escopo de banco de dados não têm nada a ver com RLS.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/row-level-security)

---

### Q125. Multi-empresa RLS em piscina dedicada SQL

**Questão:**
Você está projetando um modelo de segurança para uma piscina Azure Synapse Analytics dedicada SQL que apoiará várias empresas. Você precisa garantir que os usuários de cada empresa possam visualizar apenas os dados de sua respectiva empresa. Quais dois objetos você deve incluir na solução? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** uma chave de criptografia coluna
- **B.** chaves assimétricas
- **C.** uma função
- **D.** um papel personalizado role-based access control (RBAC)
- **E.** uma política de segurança

**✅ Resposta:** C, E

**Explicação:** Isto é row-level security. Você cria uma função de valor da tabela em linha como o predicado do filtro, por exemplo, combinando o CompanyID da linha com a empresa do usuário e vinculando-o à tabela com CREATE SECURITY POLICY. Chaves de encriptação da coluna (Always Encrypted) e chaves assimétricas manipulam criptografia, não filtragem de linha. Azure RBAC As funções controlam o acesso ao plano de gerenciamento e ao espaço de trabalho, não as linhas dentro de uma tabela. *Nota de escopo: a questão é definida em Synapse, mas o RLS funciona da mesma maneira no Azure SQL Database e Azure SQL Managed Instance, que é a habilidade DP-300 no escopo.*

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/row-level-security)

---

### Q126. Novo- New-AzSqlServer: identidade mais chave CMK

**Questão:**
Você tem uma assinatura do Azure. Você precisa implantar um SQL server lógico usando PowerShell. A solução deve garantir que o SQL server lógico possa criar usuários Microsoft Entra e fornecer Transparent Data Encryption (TDE) com uma chave gerenciada pelo cliente. Como você deve concluir o comando?

```powershell
New-AzSqlServer -ResourceGroupName "RG1" -ServerName "SQL1" -Location "EastUS" -ErrorAction Stop `
  -Tags @{Environment="Databases";"Department"="Data Tech"} [Box 1] [Box 2] `
  "https://db1.vault.azure.net/keys/dbkey/01234234512345678901234561823942"
```

**Alternativas:**

**Caixa 1:**
- **A.** -AssignIdentity
- **B.** -FederatedClientId
- **C.** -KeyId

**Caixa 2:**
- **A.** -AssignIdentity
- **B.** -FederatedClientId
- **C.** -KeyId

**✅ Resposta:** Caixa 1: -AssignIdentity; Caixa 2: -KeyId

**Explicação:** -AssignIdentity cria e atribui a Microsoft Entra (gerenciado) identidade para o servidor. O servidor precisa dessa identidade para chegar ao Key Vault para o TDE protetor, e a consulta Microsoft Entra ID ao criar usuários do Entra (com permissões de Leitores de Diretório / Gráfico). -KeyId leva a chave Key Vault URI que se segue, então a Caixa 2 deve ser -KeyId. -FederatedClientId é apenas para cross-tenant CMK ("não defina esse valor se você não pretende usar o Cross-Tenant CMK"), e nada na questão é cross-tenant. O cmdlet doc's prop CMK exemplo usa `-AssignIdentity ... -KeyId "<vault key URI>"`.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/powershell/module/az.sql/new-azsqlserver)

---

### Q127. TDE em SQL VM: crie DMK primeiro

**Questão:**
Você tem um Azure virtual machine que hospeda um banco de dados Microsoft SQL Server 2022 chamado DB1. Você precisa habilitar Transparent Data Encryption (TDE) para DB1. O que você deve criar primeiro?

**Alternativas:**
- **A.** uma credencial com escopo de banco de dados
- **B.** uma chave de criptografia (DEK)
- **C.** certificado an.509
- **D.** uma chave mestre de banco de dados (DMK)

**✅ Resposta:** D

**Explicação:** Ligado SQL Server (incluindo em uma Azure VM), TDE está configurado nesta ordem: (1) criar uma chave mestra no mestre, (2) criar um certificado protegido por essa chave mestra, (3) criar a chave mestra no mestre DEK no banco de dados do usuário, protegido pelo certificado, (4) ALTER DATABASE … SET ENCRYPTION ONO certificado e DEK Depende da DMKUma credencial com escopo de banco de dados não faz parte da TDE.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/transparent-data-encryption)

---

### Q128. TDE em SQL VM: sequência de objeto

**Questão:**
Você tem uma instância de SQL Server on Azure Virtual Machines chamada SQL1. SQL1 contém um banco de dados chamado DB1. Você precisa habilitar Transparent Data Encryption (TDE) para DB1. Quais três objetos você deve criar em sequência? Para responder, mova os objetos apropriados da lista de objetos para a área de resposta e organize-os na ordem correta.

**Alternativas:**
- **A.** uma chave de criptografia no banco de dados mestre
- **B.** uma chave mestra na banco de dados master
- **C.** um certificado em DB1
- **D.** um certificado no banco de dados mestre
- **E.** uma chave mestra em DB1
- **F.** uma chave de criptografia de banco de dados em DB1

**✅ Resposta:** 1 → uma chave mestra na banco de dados master; 2 → um certificado na banco de dados mestra; 3 → uma chave de encriptação da banco de dados em DB1

**Explicação:** De acordo com os documentos TDE, o DMK e o certificado devem ser armazenados no mestre, enquanto o DEK é criado e armazenado no banco de dados do usuário (`USE DB1; CREATE DATABASE ENCRYPTION KEY … ENCRYPTION BY SERVER CERTIFICATE …`), seguido pelo `ALTER DATABASE DB1 SET ENCRYPTION ON`. Um certificado ou chave mestra no DB1 não pode proteger o DEK, e o DEK não pode viver no mestre.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/transparent-data-encryption)

---

### Q129. TDE com sequência Key Vault EKM

**Questão:**
Você tem um Azure Key Vault chamado Vault1 e uma instância de SQL Server on Azure Virtual Machines chamada SQL1. SQL1 hospeda um banco chamado DB1. Configure Transparent Data Encryption (TDE) em DB1 para usar uma chave de Vault1. Quais quatro ações você deve executar, em ordem?

**Alternativas:**
- **A.** Ligado SQL1, criar um provedor criptográfico e uma Microsoft SQL Server credencial.
- **B.** Ligado SQL1, crie uma chave simétrica.
- **C.** Ligado SQL1, criar a login da chave assimétrica.
- **D.** Ligado SQL1, criar a login da chave simétrica.
- **E.** Crie um Microsoft Entra service principal e conceda as permissões service principal para o Vault1.
- **F.** Ligado SQL1, crie uma chave assimétrica.

**✅ Resposta:**
1. Crie um Microsoft Entra service principal e conceda as permissões service principal para o Vault1.
2. Ligado SQL1, criar um provedor criptográfico e uma Microsoft SQL Server credencial.
3. Ligado SQL1, crie uma chave assimétrica.
4. Ligado SQL1, criar a login da chave assimétrica.

**Explicação:** O documentado EKM fluxo é: (1) registro de aplicativo / service principal com chave envoltório e desembrulhar direitos sobre o cofre; (2) `CREATE CRYPTOGRAPHIC PROVIDER … FROM FILE = '…Microsoft.AzureKeyVaultService.EKM.dll'` e `CREATE CREDENTIAL … FOR CRYPTOGRAPHIC PROVIDER`; (3) `CREATE ASYMMETRIC KEY … FROM PROVIDER [AzureKeyVault_EKM] WITH PROVIDER_KEY_NAME = …`; (4) `CREATE LOGIN TDE_Login FROM ASYMMETRIC KEY …`, mapeado para a credencial. O DEK é então criado `ENCRYPTION BY SERVER ASYMMETRIC KEY`. A chave assimétrica é aberta *do provedor*Portanto, o provedor deve existir primeiro. EKM apenas suportes RSA (assimétrica) chaves, de modo que as opções simétricas-chave são distratores. (SQL Server 2022 CU17+ em uma Azure VM pode usar a managed identity em vez de a service principalMas a ordem é a mesma.)

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/setup-steps-for-extensible-key-management-using-the-azure-key-vault)

---

### Q130. Criptografar o armazém de dados Synapse em repouso

**Questão:**
Você tem um data warehouse em Azure Synapse Analytics. Você precisa garantir que os dados no data warehouse estejam criptografados em repouso. O que você deve habilitar?

**Alternativas:**
- **A.** Transparent Data Encryption (TDE)
- **B.** Segurança Avançada de Dados para este banco de dados
- **C.** Always Encrypted para todas as colunas
- **D.** Transferência segura necessária

**✅ Resposta:** A

**Explicação:** TDE criptografa dados e arquivos de log (e backups) em repouso para SQL Server, Azure SQL Database, Azure SQL Managed Instance e Azure Synapse dedicado SQL pools. Segurança avançada de dados (agora Microsoft Defender for SQL) abrange a avaliação de vulnerabilidades e detecção de ameaças, não criptografia. Always Encrypted A criptografia do lado do cliente no nível da coluna, não é a maneira de criptografar todo o armazém em repouso. "Transferência segura necessária" é uma configuração de conta de armazenamento para criptografia em trânsito. *Nota de escopo: a questão é definida em Synapse, mas o TDE funciona da mesma maneira no Azure SQL Database e Managed Instance, que é a habilidade DP-300 no escopo. No Azure SQL, o TDE está ativado por padrão para novas bancos de dados.*

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/transparent-data-encryption)

---

<a name="d3--monitor-configure-and-optimize-database-resources-2025"></a>

## D3 — Monitorar, configurar e otimizar recursos de banco de dados (20–25%)

*59 perguntas*

### Q131. Automatic tuning em Managed Instance

**Questão:**
Você tem um Azure SQL managed instance chamado MI1. Você precisa implementar automatic tuning para os bancos de dados do MI1. O que você deve fazer?

**Alternativas:**
- **A.** Use o REST API para chamar a operação de patch e modificar a propriedade AutomaticTuningServerMode.
- **B.** A partir do Azure portal, configure o automatic tuning.
- **C.** Use Transact-SQL para ativar a opção FORCE_LAST_GOOD_PLAN.

**✅ Resposta:** C

**Explicação:** SQL Managed Instance suporta apenas a opção FORCE LAST GOOD PLAN automatic tuning, e os documentos dizem que só pode ser configurado através de T-SQL (`ALTER DATABASE … SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON)`). A configuração do Portal e REST API (modo servidor automatic tuning) aplica-se apenas aos servidores lógicos Azure SQL Database.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/automatic-tuning-enable) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/automatic-tuning-overview)

---

### Q132. SIM: desativar FORCE_LAST_GOOD_PLAN em db1

**Questão:**
SIMULATION. Você planeja realizar testes de desempenho do db1. Você precisa evitar que o db1 reverta para o último bom plano de consulta conhecido. Para concluir essa tarefa, entre na máquina virtual. Você pode precisar usar o SQL Server Management Studio e o Azure portal.

**Alternativas:** Nenhum (tarefa de laboratório manual)

**✅ Resposta:** Desligue a opção FORCE_LAST_GOOD_PLAN (automatic plan correction) para db1. É ativado por padrão através dos padrões do Azure que o db1 herda.
- **T-SQL (SSMS, ligado ao db1):**
```sql
ALTER DATABASE [db1] SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = OFF);
-- verify
SELECT name, desired_state_desc, actual_state_desc
FROM sys.database_automatic_tuning_options;
```
- **Portal alternativo:** Azure portal > SQL bancos de dados > db1 > Intelligent Performance > **Automatic tuning** > conjunto **Plano de força** E- **OFF** (em vez de Herança) > **Aplicar**.

**Explicação:** Os padrões do Azure ativam o FORCE_LAST_GOOD_PLAN, que força o bom plano anterior sempre que detecta uma regressão de plano. Definir a opção explicitamente para OFF no nível do banco de dados substitui o servidor herdado ou o padrão do Azure, para que o teste veja o comportamento real do plano. Alterar a configuração do nível do servidor também afetaria outros bancos de dados.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/automatic-tuning-enable)

---

### Q133. SQL VM: Query Store então FORCE_LAST_GOOD_PLAN

**Questão:**
Você tem SQL Server em um Azure virtual machine que contém um banco de dados chamado Db1. Você precisa habilitar automatic tuning para Db1. Como você deve completar as declarações?

```sql
ALTER DATABASE [Db1]
  [Box 1]
GO
ALTER DATABASE [Db1]
  [Box 2]
```

**Alternativas:** (a mesma lista para cada caixa)
- **A.** SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = OFF)
- **B.** SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON)
- **C.** SET AUTOMATIC_TUNING = AUTO
- **D.** SET QUERY_STORE = OFF
- **E.** SET QUERY_STORE = ON (OPERATION_MODE = READ_ONLY)
- **F.** SET QUERY_STORE = ON (OPERATION_MODE = READ_WRITE)

**✅ Resposta:** Caixa 1: SET QUERY_STORE = ON (OPERATION_MODE = READ_WRITE); Caixa 2: SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON)

**Explicação:** Automatic tuning E- SQL Server (2017+), inclusive em uma Azure VM, meios automatic plan correction, que depende de Query Store. `FORCE_LAST_GOOD_PLAN = ON` "falha se o Query Store não está habilitado ou se o Query Store não está em Read-Write "modo", então Query Store deve ser habilitado em READ_WRITE primeiro. Para SQL Server, o ALTER DATABASE A sintaxe é apenas `AUTOMATIC_TUNING ( FORCE_LAST_GOOD_PLAN = { DEFAULT | ON | OFF } )`. O `AUTOMATIC_TUNING = AUTO | INHERIT | CUSTOM` A forma existe apenas em Azure SQL Database (servidor lógico), que não se aplica a um SQL Server VM. READ_ONLY Query Store e FORCE_LAST_GOOD_PLAN = OFF Não permita nada.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-set-options?view=sql-server-ver16) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/automatic-tuning-enable)

---

### Q134. Parar de ler bloqueio escreve: RCSI

**Questão:**
Você implanta um banco de dados em um Azure SQL Database managed instance. Você precisa impedir que consultas de leitura bloqueiem consultas que estão tentando escrever no banco de dados. Qual opção de banco de dados você deve definir?

**Alternativas:**
- **A.** PARAMETERIZATION a FORCED
- **B.** PARAMETERIZATION a SIMPLE
- **C.** Delayed Durability a Forçado
- **D.** READ_COMMITTED_SNAPSHOT a ON

**✅ Resposta:** D

**Explicação:** Com READ_COMMITTED_SNAPSHOT ON, leitores read-committed usam versões de linha em tempdb (ou a loja de versão persistente sob ADR) em vez de bloqueios compartilhados, para que os leitores não bloqueiem escritores e escritores não bloqueiem leitores (`ALTER DATABASE … SET READ_COMMITTED_SNAPSHOT ON`). PARAMETERIZATION FORCED : SIMPLE apenas muda o cache plano e reutilização. Delayed durability reduz a latência do log-flush no commit, mas não muda o bloqueio do leitor/escritor. (RCSI é por padrão em Azure SQL Database; em a managed instance, verificar `sys.databases.is_read_committed_snapshot_on` e defini-lo explicitamente.)

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/t-sql/statements/set-transaction-isolation-level-transact-sql)

---

### Q135. Mudar MAXDOP em Azure SQL Database

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado db1 em um servidor chamado server1. Você precisa modificar as configurações MAXDOP para db1. O que você deve fazer?

**Alternativas:**
- **A.** Conecte-se ao db1 e execute o comando sp_configure.
- **B.** Conecte-se ao banco de dados mestre do server1 e execute o comando sp_configure.
- **C.** Configure as propriedades estendidas do db1.
- **D.** Modifique a configuração de escopo do banco de dados do db1.

**✅ Resposta:** D

**Explicação:** Em Azure SQL Database, MAXDOP é definido por banco de dados com `ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP = n;` enquanto conectado a db1. Nível do servidor `sp_configure 'max degree of parallelism'` não está disponível em um Azure SQL Database servidor lógico, então nenhum sp_configure As propriedades estendidas são apenas anotações de metadados.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql)

---

### Q136. SIM: definir nível de compatibilidade 110

**Questão:**
SIMULATION. Você tem um aplicativo legado escrito para o Microsoft SQL Server 2012. O aplicativo será o único aplicativo que acessa o db1. Você precisa garantir que o db1 seja compatível com todos os recursos e sintaxe do SQL Server 2012. Para concluir essa tarefa, entre na máquina virtual. Você pode precisar usar o SQL Server Management Studio e o Azure portal.

**Alternativas:** Nenhum (tarefa de laboratório manual)

**✅ Resposta:** Defina o nível de compatibilidade do banco de dados db1 para **110** (SQL Server 2012).
- **T-SQL (SSMS, ligado ao servidor lógico, janela de consulta em db1 ou mestre):**
```sql
ALTER DATABASE [db1] SET COMPATIBILITY_LEVEL = 110;
-- verify
SELECT name, compatibility_level FROM sys.databases WHERE name = 'db1';
```
- **SSMS GUI alternativa:** Object Explorer > clique com o botão direito db1 Propriedades > **Alternativas** page > Compatibility level = **SQL Server 2012 (110)** > OK.

**Explicação:** O nível de compatibilidade controla o comportamento T-SQL e de processamento de consulta de um banco de dados. Azure SQL Database suporta níveis 100–170, então 110 é válido. Porque apenas este aplicativo legado usa db1, a redução do nível não tem efeitos colaterais em outros aplicativos. (Os recursos descontinuados não são restaurados pelo nível de compatibilidade, mas 110 corresponde ao comportamento SQL Server 2012.) Não use apenas o LEGACY_CARDINALITY_ESTIMATION: ele afeta apenas os recursos cardinality estimator ou sintaxe.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-compatibility-level)

---

### Q137. SIM: QUERY_OPTIMIZER_HOTFIXES em db1, db2

**Questão:**
SIMULATION. Você precisa garantir que quaisquer melhorias feitas para o Query Optimizer através de patches estão disponíveis para db1 e db2 em sql12345678. Para concluir esta tarefa, entre na máquina virtual. Você pode precisar usar SQL Server Management Studio e o Azure portal.

**Alternativas:** Nenhum (tarefa de laboratório manual)

**✅ Resposta:** Habilite a configuração de escopo do banco de dados QUERY_OPTIMIZER_HOTFIXES no banco de dados **Cada**:
1. Em SSMS, conecte-se a sql12345678.database.windows.net.
2. Abra uma janela de consulta **E-mail db1** e execute:
```sql
ALTER DATABASE SCOPED CONFIGURATION SET QUERY_OPTIMIZER_HOTFIXES = ON;
```
3. Abra uma janela de consulta **E-mail db2** (Azure SQL Database não suporta `USE` para mudar de banco de dados) e execute a mesma instrução.
4. Verifica opcional em cada banco de dados: `SELECT name, value FROM sys.database_scoped_configurations WHERE name = 'QUERY_OPTIMIZER_HOTFIXES';`
(GUI alternativas: banco de dados Propriedades > Opções > Configurações do Database Scoped > Query Optimizer Fixes = ON.)

**Explicação:** QUERY_OPTIMIZER_HOTFIXES = ON permite query optimizer hotfixes liberados após o mais alto nível de compatibilidade, independentemente do nível de compatibilidade do banco de dados. É equivalente a rastrear a bandeira 4199, que você não pode definir em Azure SQL Database. É uma configuração de banco de dados, por isso deve ser definido em ambos db1 e db2.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql)

---

### Q138. Planos de uso único: OPTIMIZE_FOR_AD_HOC_WORKLOADS

**Questão:**
Você tem um Azure SQL database. Você descobre que o cache do plano está cheio de planos compilados que foram usados apenas uma vez. Você executa o comando `SELECT * FROM sys.database_scoped_configurations` Transact-SQL e recebe os resultados mostrados na tabela a seguir.

| configuration_id | Nome | Valor | is_value_default |
|---|---|---|---|
| 1 | LEGACY_CARDINALITY_ESTIMATION | 0 | 1 |
| 2 | QUERY_OPTIMIZER_HOTFIXES | 0 | 1 |
| 3 | OPTIMIZE_FOR_AD_HOC_WORKLOADS | 0 | 1 |
| 4 | ACCELERATED_PLAN_FORCING | 1 | 1 |

Você precisa aliviar a pressão da memória. O que você deve configurar?

**Alternativas:**
- **A.** LEGACY_CARDINALITY_ESTIMATION
- **B.** QUERY_OPTIMIZER_HOTFIXES
- **C.** OPTIMIZE_FOR_AD_HOC_WORKLOADS
- **D.** ACCELERATED_PLAN_FORCING

**✅ Resposta:** C

**Explicação:** Com o `OPTIMIZE_FOR_AD_HOC_WORKLOADS = ON`, a primeira compilação de um lote armazena apenas um pequeno esboço do plano compilado. O plano completo é armazenado em cache somente se o lote for executado novamente, então os planos de uso único param de preencher o cache do plano. Comportamento do otimizador de mudança A e B, não tamanho do cache. D (plan forcing acelerado) é sobre planos forçados no Query Store.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql) [Doc 2](https://learn.microsoft.com/en-us/archive/blogs/sqlserverstorageengine/database-scoped-optimizing-for-ad-hoc-workloads)

---

### Q139. Plano estimado no editor de consulta portal

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1. Você precisa exibir o estimated execution plan de uma consulta usando o editor de consulta no Azure portal. O que você deve fazer primeiro?

**Alternativas:**
- **A.** Execute a declaração SET SHOWPLAN_ALL Transact-SQL.
- **B.** Para DB1, definir QUERY_CAPTURE_MODE de Query Store para Todos.
- **C.** Execute a declaração SET FORCEPLAN Transact-SQL.
- **D.** Habilitar Query Store para DB1.

**✅ Resposta:** A

**Explicação:** O Azure portal O editor de consulta não tem nenhum botão gráfico "exiba o plano estimado". Ele só executa T-SQL e mostra conjuntos de resultados. Correr `SET SHOWPLAN_ALL ON` as primeiras declarações posteriores retornam seu plano estimado como linhas sem execução. Microsoft Learn's DP-300 "Para ver um plano estimado baseado em texto, execute o comando SET SHOWPLAN_ALL ON antes de realizar a consulta”. Query Store (B, D) registra os planos de consultas que já foram executadas e não é necessário para um plano estimado. Também está ativado por padrão em Azure SQL Database. SET FORCEPLAN (C) apenas força a ordem de união.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/training/modules/explore-query-performance-optimization/2-understand-query-plans) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/query-editor)

---

### Q140. Identificar tipo de plano e operador para sintonizar

**Questão:**
*(A introdução do tronco é cortada na fonte e nenhuma versão completa foi encontrada. Reconstruído: você executa uma consulta contra o WideWorldImporters e vê a seguinte exibição.)*

Exposição (plano gráfico em SSMS):
- Cabeçalho: **"Progresso estimado da consulta: 100%"**. Consulta 1: Custo de consulta (em relação ao lote): 100%
- `select OrderID, PickedbyPersonID, OrderDate from WideWorldImporters.Sales.Orders where CustomerID = 120`
- Missing Index (Impact 99.6573): CREATE NONCLUSTERED INDEX … ON [Vendas] [Ordens]...
- SELECT (0.126s) ← Nested Loops (Inner Join) 0.126s, 392 de 446 (87%)
  - Index Seek (NonClustered) [Orders].[FK_Sales_Orders_CustomerID]: 0.126s, 392 de 446 (87%)
  - Key Lookup (Clustered) [Orders].[PK_Sales_Orders]: 0.126s, 392 de 446 (87%)

Use os menus suspensos para selecionar a opção de resposta que completa cada declaração com base nas informações apresentadas no gráfico. NOTE: Cada seleção correta vale um ponto.

**Alternativas:**

**Declaração 1: A exposição mostra ____.**
- **A.** actual execution plan
- **B.** estimated execution plan
- **C.** Live Query Estatísticas

**Declaração 2: O operador _____ no plano de execução indica que a consulta se beneficiaria do ajuste de desempenho.**
- **A.** Index Seek
- **B.** Key Lookup
- **C.** Loops aninhados

**✅ Resposta:** Declaração 1: **Live Query Estatísticas**; Declaração 2: **Key Lookup**

**Explicação:** O banner geral "Progresso estimado da consulta: N%" e o display "tempo decorrido + real das linhas estimadas (percent)" pertencem ao SSMS Live Query Statistics view, que "exibe o progresso geral da consulta e as estatísticas de execução em tempo de execução no nível do operador". Um plano estimado não tem números de tempo de execução. Um plano real concluído não tem banner de progresso de consulta. O Key Lookup Voltando ao cluster PK, juntamente com a dica de índice ausente, mostra o nonclustered index não cobre a consulta. Adicionando as colunas selecionadas como INCLUDE As colunas removeriam o lookup.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/performance/live-query-statistics) [Doc 2](https://learn.microsoft.com/en-us/training/modules/explore-query-performance-optimization/2-understand-query-plans)

---

### Q141. Correção Key Lookup: incluir colunas

**Questão:**
*(A introdução do tronco é cortada na fonte. Reconstruído: você tem uma consulta com o plano de execução mostrado na exposição.)*

Exposição:
`SELECT [SalesOrderID],[SalesOrderDetailID],[OrderQty],[ProductID],[UnitPrice] FROM [Sales].[SalesOrderDetail] WHERE [ModifiedDate]=@1`
Plano: SELECT (custo 0%) ← (Inner Join) (Custo 0%, 148 de 148) ← [Index Seek (NonClustered) [SalesOrderDetail].[NCI_SalesOrderDetail_M...] Custo 1%] + [Key Lookup (Clustered) [SalesOrderDetail].[PK_SalesOrderDetail_Sal...] Custo 99%]

Use os menus suspensos para selecionar a opção de resposta que completa cada declaração com base nas informações apresentadas no gráfico. NOTE: Cada seleção correta vale um ponto.

**Alternativas:**

**Declaração 1: O problema de desempenho decorre do operador ____.**
- **A.** Selecionar
- **B.** Index Seek
- **C.** Key Lookup
- **D.** Loops aninhados

**Declaração 2: O problema de desempenho pode ser resolvido adicionando incluir colunas para o ____.**
- **A.** heap
- **B.** clustered index
- **C.** nonclustered index

**✅ Resposta:** Declaração 1: **Key Lookup**; Declaração 2: **nonclustered index**

**Explicação:** O Key Lookup leva 99% do custo. Ele busca OrderQty, ProductID e UnitPrice do clustered index para cada linha que o buscador não agrupado no ModifiedDate encontra. Adicionando essas colunas como colunas INCLUDE ao nonclustered index faz com que cubra e remova o lookup. Você não pode adicionar colunas INCLUDE a um clustered index ou a uma pilha.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/training/modules/explore-query-performance-optimization/2-understand-query-plans)

---

### Q142. Last actual plan parameter values

**Questão:**
Você tem SQL Server em um Azure virtual machine que contém um banco de dados chamado DB1. Você tem um aplicativo que consulta DB1 para gerar um relatório de vendas. Você precisa ver os valores dos parâmetros da última vez que a consulta foi executada. Quais duas ações você deve executar? Cada resposta correta apresenta parte da solução. NOTE: Cada seleção correta vale um ponto.

**Alternativas:**
- **A.** Habilitar Last_Query_Plan_Stats no banco de dados mestre
- **B.** Ativar Lightweight_Query_Profiling em DB1
- **C.** Ativar Last_Query_Plan_Stats em DB1
- **D.** Habilitar Lightweight_Query_Profiling no banco de dados mestre
- **E.** Ativar PARAMETER_SNIFFING em DB1

**✅ Resposta:** B, C

**Explicação:** `sys.dm_exec_query_plan_stats` retorna o último plano real conhecido, incluindo valores de parâmetro de tempo de execução. É opt-in por banco de dados através `ALTER DATABASE SCOPED CONFIGURATION SET LAST_QUERY_PLAN_STATS = ON`, executado em DB1. Ele também "funciona sob a infra-estrutura de perfil de estatísticas de execução de consulta leve", a opção LIGHTWEIGHT_QUERY_PROFILING com escopo de banco de dados em DB1. Configurações com escopo de banco de banco de dados se aplicam ao banco de dados que você configurá-los, não para dominar (A, D). PARAMETER_SNIFFING controla a compilação do plano, não captura.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-plan-stats-transact-sql) [Doc 2](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql)

---

### Q143. ADatum: Alvo event_file para DB3

**Estudo de caso:** ADatum v1 (serviços financeiros)

**Questão:**
*(Estudo de caso: ADatum v1. DB3 é atualmente monitorado com Extended Events cujo alvo event_file é uma compartilhamento de arquivos em um disco local de SVR3. DB3 será migrado para Azure SQL Database. Requisito de gerenciamento: "Continuar usando Extended Events para monitorar DB3.")*
Você precisa identificar o alvo event_file para monitoramento DB3 após a migração para Azure SQL Database. A solução deve atender aos requisitos de gerenciamento. O que você deve usar como o alvo event_file?

**Alternativas:**
- **A.** a SQL Server filegroup
- **B.** Azure SQL database
- **C.** compartilhamento de Azure Files
- **D.** e Azure Blob Storage recipiente

**✅ Resposta:** D

**Explicação:** Em Azure SQL Database (e Managed Instance), o event_file o alvo apoia somente blobs em Azure Storage como o `filename` valor, dado como `https://<account>.blob.core.windows.net/<container>/<file>.xel` com uma correspondência database-scoped credential. Discos locais, partilhas de arquivos (Azure Files), grupos de arquivos e bancos de dados não são válidos event_file destinos.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server)

---

### Q144. Correlate XE sessão com ETW

**Questão:**
Você tem uma instância de SQL Server on Azure Virtual Machines nomeado SQL1. SQL1 contém um Extended Events sessão nomeada session1 que captura a Microsoft SQL Server Você precisa correlacionar os eventos da sessão com eventos capturados pelo Event Tracing for Windows (ETW). O que você deve fazer para session1?

**Alternativas:**
- **A.** Modifique as configurações de Filtros de eventos de sessão de conjunto.
- **B.** Adicione um alvo.
- **C.** Adicione uma ação.
- **D.** Modifique as configurações Especificar armazenamento de dados de sessão.

**✅ Resposta:** B

**Explicação:** Extended Events Interopera com ETW através do alvo Event Tracing para Windows, `package0.etw_classic_sync_target`. Adicionando esse alvo (`ALTER EVENT SESSION session1 ON SERVER ADD TARGET package0.etw_classic_sync_target`) sends the session's events to ETWonde você pode correlacioná-los com OS e aplicação ETW Este objetivo está disponível apenas em SQL ServerÉ por isso que a pergunta usa a VM. Filtros e ações mudam o que é capturado, não para onde vai. D refere-se à página de destino do assistente da Nova Sessão, mas a ação definida é adicionar um alvo.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server)

---

### Q145. Dados brutos XE para Armazenamento, impacto mínimo

**Questão:**
Você tem uma assinatura do Azure que contém uma instância de SQL Server on Azure Virtual Machines. A máquina virtual hospeda um banco de dados chamado DB1. Você precisa monitorar DB1 usando Extended Events. A solução deve atender aos seguintes requisitos:
- Capture dados de eventos brutos e armazene os dados em Azure Storage.
- Minimize o impacto de desempenho da captura de extended events.

Como você deve completar a declaração Transact-SQL? Para responder, selecione as opções apropriadas na área de resposta. NOTE: Cada seleção correta vale um ponto.
```sql
CREATE EVENT SESSION session1 ON DATABASE
  ADD EVENT sqlserver.sql_statement_starting
    (
    ACTION (sqlserver.sql_text)
    WHERE statement LIKE 'UPDATE gmTabEmployee%'
    )
  ADD TARGET
    package0.[Box1]
      (
      SET filename = 'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent/anyfilenamexel242b.xel'
      )
    WITH
      (MAX_MEMORY = 10 MB,
      EVENT_RETENTION_MODE = [Box2],
  MAX_DISPATCH_LATENCY = 3 SECONDS)
```

**Alternativas:**

**Box1:**
- **A.** event_file
- **B.** event_stream
- **C.** ring_buffer

**Box2:**
- **A.** ALLOW_MULTIPLE_EVENT_LOSS
- **B.** ALLOW_SINGLE_EVENT_LOSS
- **C.** NO_EVENT_LOSS

**✅ Resposta:** Box1: **event_file**; Box2: **ALLOW_MULTIPLE_EVENT_LOSS**

**Explicação:** event_file é o único alvo listado que escreve dados de eventos brutos para armazenamento persistente, incluindo um blob em Azure Storage [o `filename` URL). ring_buffer Mantém os dados apenas na memória. event_stream É para.NET consumidores e não pode ser adicionado através T-SQL. ALLOW_MULTIPLE_EVENT_LOSS permite que buffers inteiros sejam descartados sob pressão e tenha o menor impacto de desempenho. ALLOW_SINGLE_EVENT_LOSS É o meio-termo padrão. NO_EVENT_LOSS pode causar degradação de desempenho perceptível porque as sessões podem parar de esperar que os buffers fluam.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server) [Doc 2](https://learn.microsoft.com/sv-se/sql/t-sql/statements/alter-event-session-transact-sql?view=sql-server-2017)

---

### Q146. SIM: captura consultas e usuários para armazenamento

**Questão:**
SIMULATION. Você precisa capturar as seguintes informações para todas Os bancos de dados em sq12345678:
- Consultas às bancos de dados
- Usuários que executaram as consultas

As informações capturadas devem ser armazenadas em sa12345678. Para concluir esta tarefa, entre na máquina virtual. Você pode precisar usar SQL Server Management Studio e o Azure portal.

**Alternativas:** Nenhum (tarefa de laboratório manual)

**Resposta ✅ (recomendado, se sq12345678 é o servidor lógico Azure SQL, como no laboratório DP-300):** Habilitar **servidor-nível Azure SQL Auditing** para a conta de armazenamento sa12345678.
1. No Azure portal, abra o SQL server **sq12345678** → **Segurança** → **Auditoria**.
2. Set **Ativar Azure SQL Auditing** to On. Este é nível do servidor, por isso aplica-se a todos os bancos de dados existentes e novos no servidor.
3. Selecione o destino **Armazenamento** → escolha a assinatura → conta de armazenamento **sa12345678**. Defina a retenção se você quiser. Escolha chaves de acesso de armazenamento ou managed identity para autenticação, o que corresponder à configuração da conta de armazenamento.
4. Selecionar **Gravar**. Os grupos de ação de auditoria padrão (BATCH_COMPLETED_GROUP, SUCCESSFUL_DATABASE_AUTHENTICATION_GROUP, FAILED_DATABASE_AUTHENTICATION_GROUP) gravar o texto de cada declaração completa (`statement`e o diretor que a dirigiu (`server_principal_name` / `database_principal_name`).

**Alternativa (método da fonte; válido se sq12345678 é uma instância SQL Server no VM ou um managed instance):** Em SSMS → **Gestão** → **Extended Events** → **Sessões** → **Nova Sessão**. Adicione o `sql_statement_completed` / `sql_batch_completed` eventos com as ações `sql_text`, `username` / `server_principal_name` e `database_name`. On **Armazenamento de dados**, escolha **event_file** com a URL `https://sa12345678.blob.core.windows.net/<container>/<name>.xel`. Isso precisa de uma credencial com o nome do contêiner URL (SAS : managed identity). Em seguida, inicie a sessão. Em um Azure SQL servidor lógico, XE As sessões são de banco de dados (`ON DATABASE`) e aparecem sob cada banco de dados, não sob Gestão. Você precisaria de uma sessão por banco de dados, então a auditoria é a resposta prática "todas Os bancos de dados".

**Explicação:** A auditoria de nível de servidor é a maneira integrada de gravar, para todas Os bancos de dados em um servidor lógico, quais declarações foram executadas e quem as executou, escritas como arquivos.xel para uma conta de armazenamento. Extended Events com um alvo de bolha event_file também satisfaz o requisito, mas apenas no escopo do servidor em SQL Server ou MI.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/auditing-overview) [Doc 2](https://learn.microsoft.com/en-in/training/modules/configure-azure-sql-auditing/2-describe-azure-sql-auditing-capabilities) [Doc 3](https://learn.microsoft.com/en-us/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server)

---

### Q147. Informações sobre fragmentação do índice de consulta

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1. Você precisa consultar as informações de fragmentação de dados e índices para as tabelas em DB1. Qual comando você deve usar?

**Alternativas:**
- **A.** sys.dm_db_index_usage_stats
- **B.** DBCC CHECKALLOC
- **C.** DBCC SHOWCONTIG
- **D.** sys.dm_db_index_physical_stats

**✅ Resposta:** D

**Explicação:** `sys.dm_db_index_physical_stats` "retorna tamanho e informações de fragmentação para os dados e índices de uma tabela ou visualização especificada" (`avg_fragmentation_in_percent`, densidade da página), e é suportado em Azure SQL Database. DBCC SHOWCONTIG é obsoleto e substituído por este DMF. index_usage_stats mostra buscas, varreduras e atualizações, não fragmentação. CHECKALLOC verifica a consistência da alocação.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://msdn2.microsoft.com/en-us/library/ms188917.aspx)

---

### Q148. Intelligent Insights missing indexes: automatic tuning

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado db1 em um servidor chamado server1. O registro de diagnósticos Intelligent Insights identifica que várias tabelas estão faltando índices. Você precisa garantir que os índices sejam criados para as tabelas. O que você deve fazer?

**Alternativas:**
- **A.** Execute o comando DBCC SQLPERF.
- **B.** Execute o comando DBCC DBREINDEX.
- **C.** Modifique as configurações do automatic tuning para db1.
- **D.** Modifique as configurações do Query Store para db1.

**✅ Resposta:** C

**Explicação:** Para o padrão de "índice ausente", Intelligent Insights recomenda a habilitação automatic tuning, que "automaticamente sintoniza e cria índices". Ligue o CREATE INDEX automatic tuning opção para db1 (`ALTER DATABASE CURRENT SET AUTOMATIC_TUNING (CREATE_INDEX = ON)`). Desativado por padrão, já que os padrões do Azure permitem apenas FORCE_LAST_GOOD_PLAN. DBCC DBREINDEX Reconstrói os índices existentes e DBCC SQLPERF relatórios de espaço; nem cria novos índices. Query Store apenas configurações de captura de controle.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/intelligent-insights-troubleshoot-performance) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/automatic-tuning-overview)

---

### Q149. DMV para operações em um índice

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1 que contém um nonclustered index chamado index1. Os usuários finais relatam consultas lentas quando usam index1. Você precisa identificar as operações que estão sendo executadas no índice. Qual visão de gerenciamento dinâmico você deve usar?

**Alternativas:**
- **A.** sys.dm_exec_query_plan_stats
- **B.** sys.dm_db_index_physical_stats
- **C.** sys.dm_db_index_operational_stats
- **D.** sys.dm_db_index_usage_stats

**✅ Resposta:** C

**Explicação:** `sys.dm_db_index_operational_stats` Os contadores cobrem as operações realizadas no índice (inserção/atualização/delete/contagem de fantasmas, varreduras de alcance, pesquisas de singleton, divisões de página) além de esperas de bloqueio de linha e página, esperas de trava e esperas de trava de página I/O. É isso que você precisa para ver o que está acontecendo index1 Por que é lento. physical_stats (B) dá fragmentação e tamanho. usage_stats (D) dá apenas busca / varredura grosseira /lookup/update contagens e últimos tempos usados, sem detalhes de contenção. query_plan_stats (A) retorna o último plano real, não a atividade do índice.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-db-index-operational-stats-transact-sql)

---

### Q150. SIM: índice de reconstrução com fator de preenchimento 70

**Questão:**
SIMULATION. Você precisa reconstruir o índice para PK_Address_AddressID na tabela SalesLT.Address em db1. A solução deve atender aos seguintes requisitos:
- Certifique-se de que 30% da página do índice esteja reservada para futuras atualizações e inserções.
- Minimize as divisões de página.

Para concluir esta tarefa, inicie sessão na máquina virtual. Poderá ter de utilizar o SQL Server Management Studio e o Azure portal.

**Alternativas:** Nenhum (tarefa de laboratório manual)

**✅ Resposta:** Reconstrua o índice com **FILLFACTOR = 70** (70% completo, 30% livre).

T-SQL (SSMS ou o editor de consulta do portal conectado ao db1):
```sql
ALTER INDEX PK_Address_AddressID ON SalesLT.Address
REBUILD WITH (FILLFACTOR = 70);
```
SSMS GUI alternativa:
1. Conecte-se ao servidor e expanda **db1** → **Tabelas** → **SalesLT.Address** → **Indexes**.
2. Clique com o botão direito **PK_Address_AddressID** → **Propriedades** → **Alternativas** página. Defina **Fator de preenchimento** para **70**. Opcionalmente, defina **índice Pad** para True para aplicar o mesmo espaço livre a páginas de nível intermediário.
3. Selecione **OK**. O índice é reconstruído com o novo fator de preenchimento. Alternativamente, clique com o botão direito do mouse em → **Reconstruir** depois de definir a propriedade.

**Explicação:** Fill factor define a porcentagem preenchida em cada leaf page quando o índice é criado ou reconstruído. Com 30% de espaço reservado, 100 − 30 = 70% de preenchimento. O espaço livre acomoda linhas novas e atualizadas sem page splits. A configuração só passa a valer na criação ou no rebuild, então execute REBUILD.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/sql/relational-databases/indexes/specify-fill-factor-for-an-index)

---

### Q151. Mais rápido DBCC CHECKDB em 50- TB DB

**Questão:**
Você tem um banco de dados 50- TB Microsoft SQL Server chamado DB1. Você precisa reduzir o tempo necessário para realizar verificações de consistência do banco de dados do DB1. Qual comando Transact-SQL você deve executar? Para responder, selecione as opções apropriadas na área de resposta. NOTE: Cada seleção correta vale um ponto.
```sql
DBCC CHECKDB ([DB1], [Box 1]) WITH [Box 2]
```

**Alternativas:**

**Caixa 1:**
- **A.** NOINDEX
- **B.** REPAIR_FAST
- **C.** REPAIR_REBUILD

**Caixa 2:**
- **A.** ALL_ERRORMSGS
- **B.** NO_INFOMSGS
- **C.** PHYSICAL_ONLY

**✅ Resposta:** Caixa 1: **NOINDEX**; Caixa 2: **PHYSICAL_ONLY**

**Explicação:** NOINDEX pula verificações intensivas de nonclustered indexes em tabelas de usuário, e os médicos dizem que "diminui o tempo total de execução". PHYSICAL_ONLY verificação de limites para página e registro cabeçalhos e alocação consistência, por isso "pode causar um tempo de execução muito mais curto para DBCC CHECKDB em grandes bancos de dados" e é recomendado para uso frequente na produção. REPAIR_* opções de reparação em vez de verificações de excesso de velocidade; eles precisam do modo de usuário único e não podem ser combinados com PHYSICAL_ONLY. ALL_ERRORMSGS e NO_INFOMSGS apenas alterar a saída.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/t-sql/database-console-commands/dbcc-checkdb-transact-sql)

---

### Q152. Recuperar DB de erro CHECKSUM

**Questão:**
Você tem SQL Server em um Azure virtual machine que contém um banco de dados chamado DB1. O banco de dados relata um erro CHECKSUM. Você precisa recuperar o banco de dados. Como você deve completar as declarações? Para responder, selecione as opções apropriadas na área de resposta. NOTE: Cada seleção correta vale um ponto.
```sql
USE master;
ALTER DATABASE [DB1] SET [Box 1] WITH ROLLBACK IMMEDIATE;
GO
DBCC CHECKDB ('DB1', [Box 2]) WITH NO_INFOMSGS;
GO
ALTER DATABASE [DB1] SET [Box 3]
GO
```

**Alternativas:**

**Caixa 1:**
- **A.** OFFLINE
- **B.** ONLINE
- **C.** SINGLE_USER
- **D.** TRUSTWORTHY

**Caixa 2:**
- **A.** NOINDEX
- **B.** PHYSICAL_ONLY
- **C.** REPAIR_ALLOW_DATA_LOSS
- **D.** REPAIR_FAST

**Caixa 3:**
- **A.** MULTI_USER
- **B.** ONLINE
- **C.** OPEN
- **D.** TRUSTWORTHY

**✅ Resposta:** Caixa 1: **SINGLE_USER**; Caixa 2: **REPAIR_ALLOW_DATA_LOSS**; Caixa 3: **MULTI_USER**

**Explicação:** DBCC CHECKDB As opções de reparo exigem que o banco de dados esteja no modo de usuário único, e `WITH ROLLBACK IMMEDIATE` desconecta outras sessões. A CHECKSUM Erro de página é corrupção física que apenas REPAIR_ALLOW_DATA_LOSS pode corrigir, possivelmente por deallocação de páginas. REPAIR_FAST é mantido para compatibilidade com versões anteriores e não realiza reparos. NOINDEX e PHYSICAL_ONLY Depois disso, defina o banco de dados de volta para MULTI_USER. A recomendação principal da Microsoft ainda é restaurar de um último bem conhecido backupA reparação é o último recurso.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/t-sql/database-console-commands/dbcc-checkdb-transact-sql)

---

### Q153. Intelligent Insights tempDB correção de contenção

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado db1 em um servidor chamado server1. O log de diagnósticos Intelligent Insights identifica consultas que causam problemas de desempenho devido à contenção do TempDB. Você precisa resolver os problemas de desempenho. O que você deve fazer?

**Alternativas:**
- **A.** Implemente o memory-optimized tables.
- **B.** Execute o comando DBCC FLUSHPROCINDB.
- **C.** Substitua as chaves de índice sequenciais por chaves não sequenciais.
- **D.** Execute o comando DBCC DBREINDEX.

**✅ Resposta:** A

**Explicação:** O Intelligent Insights O guia de solução de problemas diz que, para aliviar a contenção do tempdb (muitas consultas simultâneas criando, usando e soltando pequenas tabelas temporárias), "Você pode parar de usar as tabelas temporárias. Você também pode usar memory-optimized tables." Replacing sequential keys with nonsequential keys (C) is the documented fix for *pagelatch* Contenção sobre índices de usuários, não tempdb. FLUSHPROCINDB Limpa planos em cache e DBREINDEX reconstrói índices; nem afeta a contenção do tempdb.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/intelligent-insights-troubleshoot-performance)

---

### Q154. Litware: coletar e transmitir métricas SalesSQLDb1

**Estudo de caso:** Litware

**Questão:**
*(Estudo de caso: Litware. SalesSQLDb1 é um Azure SQL database geo-replicado no servidor lógico SalesSrv01A, no elastic pool SalesSQLDb1Pool. Exigência técnica: "Todas as métricas SQL Server e Azure SQL Database relacionadas ao CPU e uso de armazenamento e limites devem ser analisados usando a funcionalidade integrada do Azure.")*
Você precisa implementar o monitoramento do SalesSQLDb1. A solução deve atender aos requisitos técnicos. Como você deve coletar e transmitir métricas? Para responder, selecione as opções apropriadas na área de resposta. NOTE: Cada seleção correta vale um ponto.

**Alternativas:**

**Colete métricas de:**
- **A.** Apenas a banco de dados
- **B.** O elastic pool e a banco de dados
- **C.** Somente o elastic pool
- **D.** O servidor, o elastic pool e o banco de dados

**Stream métricas para:**
- **A.** Azure Event Hubs
- **B.** Azure Log Analytics
- **C.** Azure Storage

**✅ Resposta:** Coletar métricas de: **O elastic pool e a banco de dados**; Stream metrics para: **Azure Log Analytics**

**Explicação:** A telemetria diagnóstica para bancos de dados agrupados deve ser configurada em cada recurso separadamente. Por os documentos: "Para configurar o streaming da telemetria diagnóstica para elastic pools e bancos de dados agrupados, você precisa configurar separadamente cada recurso... O elastic pool O contêiner tem sua própria telemetria separada da telemetria de cada banco de dados agrupado individual." As métricas básicas do pool incluem CPU/eDTU, limite de armazenamento e armazenamento. O servidor lógico não é uma fonte de telemetria diagnóstica para essas métricas, portanto, "servidor, piscina e banco de dados" está errado. Para *analisar* as métricas com a funcionalidade integrada do Azure, enviá-las para um Log Analytics workspace (Azure Monitor logs, alertas, workbooks). Event Hubs é para streaming a terceiros SIEMs ou plataformas de telemetria personalizadas. O armazenamento é para arquivamento barato.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure)

---

### Q155. Litware: DMVs para bloqueio SalesSQLDb1

**Estudo de caso:** Litware

**Questão:**
*(Estudo de caso: Litware. " SalesSQLDb1 experimenta problemas de desempenho que são provavelmente devido a estatísticas desatualizadas e consultas frequentes de bloqueio." SalesSQLDb1 é um Azure SQL database em um elastic pool.)*
Você precisa identificar a causa dos problemas de desempenho no SalesSQLDb1. Quais duas visões dinâmicas de gerenciamento você deve usar? Cada resposta correta apresenta parte da solução. NOTE: Cada seleção correta vale um ponto.

**Alternativas:**
- **A.** sys.dm_pdw_nodes_tran_locks
- **B.** sys.dm_exec_compute_node_errors
- **C.** sys.dm_exec_requests
- **D.** sys.dm_cdc_errors
- **E.** sys.dm_pdw_nodes_os_wait_stats
- **F.** sys.dm_tran_locks

**✅ Resposta:** C, F

**Explicação:** da Microsoft Azure SQL Database Usos do guia de bloqueio `sys.dm_exec_requests` (`blocking_session_id`, `wait_type`, `wait_resource`) para encontrar sessões bloqueadas e bloqueadoras de cabeça. Ele usa `sys.dm_tran_locks` (uni-se a `sys.dm_os_waiting_tasks`) para ver quais fechaduras são mantidas e aguardadas. O `sys.dm_pdw_*` views (A, E) existem apenas no Azure Synapse dedicado SQL piscinas (PDW/SQL DW), não em Azure SQL Database. B é a PolyBase compu-nodo DMV e D é para change data capture; nem se aplica.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/understand-resolve-blocking)

---

### Q156. Declarações Database watcher KQL elastic pool

**Questão:**
Você tem um servidor lógico para Azure SQL Database chamado Server1 que tem um elastic pool chamado Pool1. Você tem um database watcher chamado Watcher1. Você executa a seguinte consulta KQL.
```kusto
let logicalServer = @"Server1";
let elasticPoolName = @"Pool1";
let replicaType = "Primary";
let duration = 20h;
sqldb_elastic_pool_resource_utilization
| where sample_time_utc > ago(duration)
| where logical_server_name =~ logicalServer
| where replica_type =~ replicaType
| project sample_time_utc,
  avg_cpu_percent,
  avg_instance_cpu_percent,
  avg_data_io_percent,
  avg_log_write_percent,
  max_worker_percent
| sort by sample_time_utc desc
| render timechart;
```
*(Fonte erros de digitação "logicelServer", "elacticPoolName" e "logical_server_nane" são corrigidos aqui.)*
Para cada uma das seguintes afirmações, selecione Sim se a afirmação for verdadeira. Caso contrário, selecione No. NOTE: Cada seleção correta vale um ponto.

**Alternativas:**
- S1. A consulta retorna o tempo de execução das consultas individuais.
- S2. A consulta retorna as métricas de consumo de recursos do último dia.
- S3. A consulta visualiza o conjunto de resultados e inclui uma legenda para cada série.

**✅ Resposta:** S1: **Não**; S2: **Não**; S3: **Sim**

**Explicação:** S1: `sqldb_elastic_pool_resource_utilization` contém amostras de recursos em nível de piscina (CPU, dados IO, log write, workers), não por-consulte tempos de execução. Estes estão nos conjuntos de dados de estatísticas de tempo de execução da consulta. S2: `duration = 20h` com `ago(duration)` apenas retorna o **últimas 20 horas**, não o último dia (24 h). O exemplo da própria Microsoft descreve `duration = 1h` O valor ímpar de 20h é a armadilha; uma declaração "Sim" teria usado 1d ou 24h. S3: `render timechart` traça um gráfico de linha com `sample_time_utc` no eixo x e em cada coluna numérica como uma série, com uma legenda mostrada por padrão.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Baixa · [Doc 1](https://learn.microsoft.com/azure/azure-sql/database-watcher-analyze) [Doc 2](https://learn.microsoft.com/en-us/kusto/query/render-operator)

---

### Q157. Monitor cinco SQL VMs em uma única vista

**Questão:**
Você tem cinco instâncias de SQL Server on Azure Virtual Machines. Você precisa monitorar o desempenho do Microsoft SQL Server para todas as instâncias, consolidando métricas em uma única exibição gráfica. A solução deve minimizar o esforço administrativo. O que você deve usar?

**Alternativas:**
- **A.** Azure Monitor
- **B.** Log Analytics
- **C.** SQL Insights
- **D.** Azure SQL Analytics

**✅ Resposta:** A

**Explicação:** Azure Monitor é a plataforma integrada que coleta métricas e contadores de desempenho de todos os cinco VMs e mostra-los juntos em um gráfico, painel ou workbook (VM insights mais regras de coleta de dados). SQL Insights (preview), a opção escrita para este cenário, foi aposentado em 31 de dezembro de 2024. SQL Server on Azure VMs E VM Insights com DCRs E- Azure Monitor. Azure SQL Analytics (D) é uma solução legada que abrange apenas Azure SQL Database e Managed Instance, não SQL E-mail VMs. Log Analytics (B) é apenas a ferramenta de espaço de trabalho / consulta dentro Azure Monitor, não uma exibição consolidada pronta.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/answers/a/1896644) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/sql-insights-enable)

---

### Q158. Diagnose tempdb contetion on sqlsrv1 using DMVs

**Questão:**
Você tem um servidor Azure SQL Database chamado sqlsrv1 que hospeda 10 Azure SQL databases. Os bancos de dados funcionam mais devagar do que o esperado. Você precisa identificar se o problema de desempenho está relacionado ao uso do tempdb por Azure SQL databases no sqlsrv1. O que você deve fazer?

**Alternativas:**
- **A.** Executar consultas baseadas em Query Store
- **B.** Informação de revisão fornecida por SQL Server Profiler-based traces
- **C.** Informação da revisão fornecida por Query Performance Insight
- **D.** Executar consultas baseadas em visualização de gerenciamento dinâmico

**✅ Resposta:** D

**Explicação:** "Monitoramento do desempenho da Microsoft usando DMVs" artigo tem uma seção "Identificar problemas de desempenho Tempdb". Diz para procurar `PAGELATCH_*` espera (não `PAGEIOLATCH_*`) e para confirmar em `sys.dm_exec_requests` que `wait_resource` começa com `2:x:y`, onde banco de dados ID 2 é tempdb. Query Store e Query Performance Insight planos de consulta de faixa e estatísticas de tempo de execução por banco de dados, não tempdb latch contenção. SQL Server O Profiler não é suportado por Azure SQL Database.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/monitoring-with-dmvs)

---

### Q159. Linux SQL VM ferramentas de linha de base de desempenho

**Questão:**
Você tem uma assinatura do Azure que contém uma instância SQL Server no Linux Azure Virtual Machines chamada VM1. VM1 contém um banco de dados chamado DB1. Você precisa criar linhas de base de desempenho operacional para DB1 e VM1. O que você deve usar?

**Alternativas:**
- **A.** InfluxDB, Coletado e Grafana
- **B.** Azure Monitor Log Analytics
- **C.** Azure Event Hubs
- **D.** top, grep e sed

**✅ Resposta:** A

**Explicação:** O DP-300 A unidade de treinamento "Estabelecer métricas de linha de base" diz: "Se você estiver usando o Linux como sistema operacional, você precisa instalar InfluxDB, Coletado e Grafana para capturar dados semelhantes ao Windows Performance MonitorA unidade acrescenta que essas ferramentas coletam dados de SQL Server e dar-lhe uma interface gráfica para correlacionar SQL Server e OS que é o que uma linha de base para ambos DB1 e VM1 necessidades. Log Analytics pode coletar VM métrica, mas não captura SQL Server Contadores de um Linux VM por padrão, e não é a ferramenta os nomes oficiais de cursos. Event Hubs apenas fluxos de dados. top, grep e sed são comandos ad-hoc, não uma solução de base.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/training/modules/describe-performance-monitoring/4-establish-baseline-metrics)

---

### Q160. Consulta para detectar esperas RESOURCE_SEMAPHORE

**Questão:**
Você tem uma assinatura do Azure que contém um Azure SQL database. O banco de dados não responde às consultas em tempo hábil. Você precisa identificar se o problema se relaciona com esperas resource_semaphore. Como você deve completar a consulta Transact-SQL?
```sql
SELECT [Box1],
   SUM(wait_time) AS total_wait_time_ms
FROM sys.[Box2] AS dmv1
  JOIN sys.dm_exec_sessions AS dmv2
    ON dmv1.session_id = dmv2.session_id
WHERE is_user_process = 1
GROUP BY wait_type
ORDER BY SUM(wait_time) DESC;
```

**Alternativas:**

**Caixa 1:**
- **A.** is_user_process
- **B.** wait_time
- **C.** wait_type

**Caixa 2:**
- **A.** dm_exec_query_stats
- **B.** dm_exec_requests
- **C.** query_store_query

**✅ Resposta:** Caixa 1: wait_type; Caixa 2: dm_exec_requests

**Explicação:** Isso corresponde à consulta documentada em "Identificar problemas de desempenho de espera de concessão de memória": `SELECT wait_type, SUM(wait_time) ... FROM sys.dm_exec_requests AS req INNER JOIN sys.dm_exec_sessions AS sess ... WHERE is_user_process = 1 GROUP BY wait_type`. Os grupos de consulta por wait_type, então wait_type tem que ser a coluna que você selecionar. Somente sys.dm_exec_requests tem session_id, wait_type e wait_time. dm_exec_query_stats e query_store_query não têm session_id ou colunas de espera.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/monitoring-with-dmvs)

---

### Q161. Encontre banco de dados usando a maioria da memória e I/O em SQLMI1

**Questão:**
Você tem um Azure SQL managed instance chamado SQLMI1 que hospeda vários bancos de dados. Você precisa monitorar o desempenho do SQLMI1 e identificar qual banco de dados usa mais memória e mais disco I/O. Quais objetos você deve consultar? Para responder, selecione as opções apropriadas na área de resposta.

**Alternativas:**

**Memória:**
- **A.** sys.dm_exec_query_stats
- **B.** sys.dm_io_virtual_file_stats
- **C.** sys.dm_os_buffer_descriptors
- **D.** sys.dm_os_performance_counters

**E/O do disco:**
- **A.** sys.dm_exec_query_stats
- **B.** sys.dm_io_virtual_file_stats
- **C.** sys.dm_os_buffer_descriptors
- **D.** sys.dm_os_performance_counters

**✅ Resposta:** Memória: sys.dm_os_buffer_descriptors; Disco I/O: sys.dm_io_virtual_file_stats

**Explicação:** sys.dm_os_buffer_descriptors uma linha por página em cache e inclui database_id. O exemplo documentado "Retornar a contagem de páginas em cache para cada banco de dados" agrupa essas páginas por banco de dados, o que mostra qual banco de dados usa mais memória buffer-pool. sys.dm_io_virtual_file_stats retorna contagens de leitura/escrita, bytes e barracas por dados e arquivo de log com database_id, então mostra qual banco de dados gera mais disco I /O. sys.dm_os_performance_counters relatórios Memory Manager contadores para toda a instância e não pode quebrar memória para baixo por banco de dados. sys.dm_exec_query_stats é agregado por plano em cache, não por banco de dados.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-buffer-descriptors-transact-sql)

---

### Q162. Alertas SQLMI1 Intelligent Insights a um custo mínimo

**Questão:**
Você tem um Azure SQL managed instance chamado SQLMI1 que hospeda 10 bancos de dados. Você precisa implementar alertas usando Azure Monitor. A solução deve atender aos seguintes requisitos:
- Minimizar custos.
- Agregar a telemetria Intelligent Insights de cada banco de dados.

O que você deve fazer?

**Alternativas:**
- **A.** A partir das configurações de diagnóstico de cada banco de dados, selecione Enviar para Log Analytics.
- **B.** A partir das configurações de diagnóstico de cada banco de dados, selecione Stream para um hub de eventos.
- **C.** Nas configurações de diagnóstico do SQLMI1, selecione Enviar para o Log Analytics.
- **D.** A partir das configurações de diagnóstico do SQLMI1, selecione Stream para um hub de eventos.

**✅ Resposta:** A

**Explicação:** Ligado SQL Managed Instance, a configuração de diagnóstico de nível de instância tem apenas o ResourceUsageStats categoria. SQLInsights (Intelligent Insights), Query Store As estatísticas e os erros são categorias de **banco de dados de instância** A Microsoft afirma que a telemetria deve ser configurada separadamente para a instância e para cada banco de dados de instância ("Repetir essas etapas para cada banco de dados da instância que você deseja monitorar"). Isso exclui C e D, porque SQLMI1As próprias configurações não podem enviar Intelligent InsightsA Log Analytics workspace agrega todos os registros das bancos de dados em um só lugar e pode conduzir Azure Monitor Um hub de eventos é apenas um tubo de streaming: ele precisa de um consumidor extra, custa mais e não levanta alertas por conta própria.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure)

---

### Q163. Stream métricas SQLMI1 para o primeiro passo do Workspace1

**Questão:**
Você tem uma assinatura do Azure que contém um Azure SQL managed instance chamado SQLMI1 e um Log Analytics workspace chamado Workspace1. Você precisa coletar métricas de desempenho para SQLMI1 e transmitir as métricas para Workspace1. O que você deve fazer primeiro?

**Alternativas:**
- **A.** Crie uma conexão private endpoint no SQLMI1.
- **B.** Configure o Azure SQL Analytics para usar o Workspace1.
- **C.** Modifique as configurações de Compute + storage para SQLMI1.
- **D.** Modifique as configurações de diagnóstico para SQLMI1.

**✅ Resposta:** D

**Explicação:** Você transmite telemetria de instância gerenciada (ResourceUsageStats, além de métricas) adicionando uma configuração de diagnóstico no recurso SQLMI1 e escolhendo "Enviar para Log Analytics workspace" → Workspace1. Azure SQL Analytics só lê dados que já estão no espaço de trabalho e é uma solução aposentado / legado, por isso não pode vir em primeiro lugar. A private endpoint e Compute + configurações de armazenamento não têm nada a ver com a exportação de telemetria.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure)

---

### Q164. Coluna de espaço livre na consulta sys.database_files

**Questão:**
Você tem a seguinte consulta Transact-SQL.
```sql
SELECT
    [file_id] AS [File ID],
    [type] AS [File Type],
    substring([physical_name], 1,1) AS [Drive],
    [name] AS [Logical Name],
    [physical_name] AS [Physical Name],
    CAST([size] as DECIMAL(38,0))/128.0 AS [ColumnA],
    CAST(FILEPROPERTY([name], 'SpaceUsed') AS DECIMAL(38,0))/128.0 AS [ColumnB],
    (CAST([size] AS DECIMAL(38,0))/128.0) - (CAST(FILEPROPERTY([name], 'SpaceUsed') AS DECIMAL (38,0))/128.0) AS [ColumnC],
    [max_size] AS [ColumnD],
    [is_percent_growth] AS [Percent Growth Enabled],
    [growth] AS [Growth Rate],
    SYSDATETIME() AS [Current Date]
FROM sys.database_files;
```
Qual coluna retornada pela consulta representa o espaço livre em cada arquivo?

**Alternativas:**
- **A.** ColunaA
- **B.** ColunaB
- **C.** ColunaC
- **D.** ColunaD

**✅ Resposta:** C

**Explicação:** `size` é o tamanho do arquivo em 8-KB e dividindo por 128 converte-o em MB, então ColumnA é o tamanho alocado. `FILEPROPERTY(name,'SpaceUsed')` são as páginas usadas, então ColumnB é o MB. ColumnC = allocated − used = free space in MB. ColunaD (`max_size`) é o limite de crescimento.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/file-space-manage)

---

### Q165. Recupere o uso de recursos db1 para a semana passada

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado db1. Você precisa recuperar o uso de recursos do db1 da última semana. Como você deve completar a declaração?
```sql
SELECT *
FROM [Box1]
WHERE database_name = 'db1' AND
      start_time > [Box2] (day, -7, GETDATE())
ORDER BY start_time DESC;
```

**Alternativas:**

**Caixa 1:**
- **A.** sys.dm_db_resource_stats
- **B.** sys.dm_exec_requests
- **C.** sys.dm_user_db_resource_governance
- **D.** sys.resource_stats

**Caixa 2:**
- **A.** DATEADD
- **B.** DATEDIFF
- **C.** DATEPART
- **D.** TODATETIMEOFFSET

**✅ Resposta:** Caixa 1: sys.resource_stats; Caixa 2: DATEADD

**Explicação:** sys.resource_stats vive no banco de dados mestre do servidor lógico. Tem colunas database_name e start_time, é agregado a cada 5 minutos e mantém cerca de 14 dias, por isso cobre uma semana. sys.dm_db_resource_stats mantém apenas cerca de 1 hora, em 15 segundos granularidade, e não tem colunas database_name ou start_time. `DATEADD(day, -7, GETDATE())` retorna o ponto no tempo há 7 dias. DATEDIFF e DATEPART retornam inteiros, não uma data.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/monitoring-with-dmvs)

---

### Q166. Leia uma exposição Query Performance Insight top-5 CPU

**Questão:**
Você tem uma assinatura do Azure que contém um banco de dados do Azure SQL Database chamado DB1. Você tem a análise Query Performance Insight mostrada na exposição a seguir.

Exposição: "TOP 5 consultas por: CPU · Agregação de consulta: SUM · Período de tempo: 08/02/2024 06:00:00 – 08/02/2024 10:00:00 · Agregação de métricas: AVG". Gráfico superior: a **linha vermelha, rotulada na legenda "OVERALL DTU 1,34 %"**, sobe a um pico de cerca de **4%** (em a 0–4.5% Uma barra vermelho-escuro para consulta 3964 atinge cerca de **3.3–3.5%**. Gráfico inferior: CPU (azul) 1,34 %, DATA IO 0 %, LOG IO 0,38 %.

| QUERY ID | CPU[%] | DATA IO[%] | LOG IO[%] | DURATION [hh:mm:ss] | EXECUTIONS COUNT |
|---|---|---|---|---|---|
| 3964 | 0.86 | 0 | 0 | 00:04:00.140 | 8 |
| 21195 | 0.03 | 0 | 0.02 | 00:00:01.180 | 12723 |
| 21198 | 0.02 | 0 | 0.01 | 00:00:00.850 | 9297 |
| 21196 | 0.01 | 0 | 0 | 00:00:00.250 | 2511 |
| 21197 | 0.01 | 0 | 0 | 00:00:00.630 | 19251 |

Para cada uma das seguintes afirmações, selecione Sim se a afirmação for verdadeira. Caso contrário, selecione Não.

**Alternativas:**
- S1. O consumo máximo de DTU é de 4%.
- S2. O consumo máximo da consulta que tem ID 3964 é de 4%.
- S3. A consulta que tem ID 21195 tem uma duração de 00:00: 01.180 cada vez que é executado.

**✅ Resposta:** S1: Sim; S2: Não; S3: Não

**Explicação:** da Microsoft QPI A documentação diz: "A linha superior mostra geral DTU Percentagem para o banco de dados. As barras mostram CPU A linha vermelha, que a legenda rotula OVERALL DTU, picos em cerca de 4%, então S1 É verdade. Query 3964 é a barra vermelho-escuro, e ele tops para fora em aproximadamente 3.3–3.5%, não 4%, então S2 A duração da grade são agregados que dependem da função de agregação escolhida (Sum/Avg/Max) através 12,723 execução, não uma constante por execução, S3 É falso.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/query-performance-insight-use)

---

### Q167. Manter Query Store de ir read-only

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado sqldb1. Você precisa minimizar a possibilidade de transição Query Store para um estado read-only. O que você deve fazer?

**Alternativas:**
- **A.** O dobro do valor do intervalo de fluxo de dados
- **B.** Halve (diminua pela metade) o valor do intervalo de fluxo de dados
- **C.** O dobro do valor do intervalo de coleta de estatísticas
- **D.** Halve (diminução pela metade) o valor do intervalo de coleta de estatísticas

**✅ Resposta:** B

**Explicação:** Por Microsoft, "O tamanho do armazenamento é verificado somente quando Query Store Este intervalo é definido pela DATA_FLUSH_INTERVAL_SECONDS "Se a opção de Query Store tem violado o MAX_STORAGE_SIZE_MB entre o limite de tamanho de armazenamento, ele faz a transição para read-only Modo de descarga." Flushing mais frequentemente significa verificações de tamanho, e limpeza baseada em tamanho, correr com mais frequência. Dobrar o intervalo faz o oposto. A redução para metade do intervalo de coleta de estatísticas (INTERVAL_LENGTH_MINUTES) cria mais linhas de runtime-stats, que cresce Query Store Dobrar isso não afeta quando o tamanho é verificado.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/performance/manage-the-query-store)

---

### Q168. Tornar os dados Query Store disponíveis mais cedo para QPI

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado db1 em um servidor chamado server1. Você usa Query Performance Insight para monitorar db1. Você precisa modificar a configuração Query Store para garantir que os dados de monitoramento de desempenho estejam disponíveis o mais rápido possível. Qual configuração você deve modificar e qual valor você deve configurar?

**Alternativas:**

**Configuração Configuração:**
- **A.** DATA_FLUSH_INTERVAL_SECONDS
- **B.** INTERVAL_LENGTH_MINUTES
- **C.** MAX_PLANS_PER_QUERY
- **D.** QUERY_CAPTURE_MODE

**Valor:**
- **A.** 1
- **B.** 60
- **C.** CUSTOM
- **D.** ON

**✅ Resposta:** Configuração: **INTERVAL_LENGTH_MINUTES**; Valor: **1**

**Explicação:** Query Performance Insight mostra Query Store estatísticas de tempo de execução agregadas por intervalo de coleta. INTERVAL_LENGTH_MINUTES (valores permitidos 1, 5, 10, 15, 30, 60, 1440; padrão 60) define essa janela de agregação. A Microsoft diz para "considerar o uso de um valor menor se você precisar de granularidade mais fina ou menos tempo para detectar e mitigar problemas", e 1 minuto é o mínimo. DATA_FLUSH_INTERVAL_SECONDS apenas controla com que frequência na memória Query Store os dados são mantidos no disco. O Query Store As visualizações de catálogo já mesclam dados na memória, portanto, reduzi-los não torna os dados visíveis mais cedo. MAX_PLANS_PER_QUERY e QUERY_CAPTURE_MODE não correspondem ao objetivo "disponível mais cedo" ou à lista de valores.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Baixa · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/performance/manage-the-query-store) [Doc 2](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-set-options)

---

### Q169. Forçar o plano Query Store mais rápido

**Questão:**
*(Reconstruído: o caule acima da exposição não foi capturado e nenhuma versão completa foi encontrada online. Presume-se que "Você tem um Azure SQL database... Você revisa o resumo do Plano Query Store para uma consulta...")*
Exposição: Query Store "Plan summary for query 1178902". Y-axis = Total (duration), 0–250,000. Plano **1220917** (Vermelho) tem dois pontos em aproximadamente 265,000 e sobre 220,000. Plano **1221065** (verde) tem um ponto perto de 0.
O que você deve fazer para garantir que a consulta use o plano de execução que é executado no menor período de tempo?

**Alternativas:**
- **A.** Forçar o plano de execução da consulta para o plano 1221065.
- **B.** Execute o comando DBCC FREEPROCCACHE.
- **C.** Forçar o plano de execução da consulta para o plano 1220917.
- **D.** Desativar parameter sniffing.

**✅ Resposta:** A

**Explicação:** O plano mostra que o plano 1221065 tem de longe a menor duração total. Forçá-lo em Query Store (Plano Força, ou `sp_query_store_force_plan @query_id = 1178902, @plan_id = 1221065`) faz com que o otimizador sempre use esse plano. O plano 1220917 é o plano lento. DBCC FREEPROCCACHE só causa uma recompilação, o que pode escolher qualquer um dos planos novamente. A desativação do parameter sniffing afeta todas as consultas e não garante o plano rápido.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/performance/best-practice-with-the-query-store)

---

### Q170. SIM: capture todas as consultas em Query Store

**Questão:**
SIMULATION. Você precisa garantir que todas as consultas executadas contra db1 sejam capturadas no Query Store. Para concluir essa tarefa, entre na máquina virtual. Você pode precisar usar SQL Server Management Studio e o Azure portal.

**Alternativas:**

**✅ Resposta:** Query Store deve estar no modo de escrita de leitura **e** Query Store Modo de captura deve ser **Todos**.
- **T-SQL** (SSMS ou o editor do portal Query, conectado ao db1):
  ```sql
  ALTER DATABASE [db1]
  SET QUERY_STORE = ON (OPERATION_MODE = READ_WRITE, QUERY_CAPTURE_MODE = ALL);
  ```
- **SSMS GUI:** clique com o botão direito db1 → Propriedades → Query Store página → Operation Mode (Requested) = **Ler a escrever** → Query Store Capture Mode = **Todos** → OK.
- Verificação opcional: `SELECT actual_state_desc, query_capture_mode_desc FROM sys.database_query_store_options;` deve retornar READ_WRITE / ALL.

**Explicação:** Em Azure SQL Database, Query Store é por padrão, mas seu modo de captura é **AUTO**, que ignora consultas pouco frequentes e insignificantes. Apenas `QUERY_CAPTURE_MODE = ALL` A configuração Read write alonely, como a solução da fonte faz, deixa AUTO no lugar, para que não capturasse todas as consultas.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/performance/manage-the-query-store) [Doc 2](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-set-options)

---

### Q171. SIM: ajuste o tamanho máximo de Query Store para 4000 MB

**Questão:**
SIMULATION. Você precisa aumentar o tamanho máximo de armazenamento do query store no db1 para 4,000 MB. Você pode precisar usar SQL Server Management Studio e o Azure portal.

**Alternativas:**

**✅ Resposta:**
- **T-SQL** (SSMS ou Azure portal → db1 → Editor de consulta, conectado ao db1):
  ```sql
  ALTER DATABASE [db1] SET QUERY_STORE (MAX_STORAGE_SIZE_MB = 4000);
  ```
- **Ou SSMS GUI:** clique com o botão direito db1 → Propriedades → Query Store página → Tamanho máximo (MB) = **4000** → OK.
- Verifique: `SELECT max_storage_size_mb FROM sys.database_query_store_options;` retorna 4000.

**Explicação:** MAX_STORAGE_SIZE_MB define a cota de espaço Query Store. O máximo Azure SQL Database é 10,240 MB, então 4,000 é válido. O Azure portal não tem configuração GUI para isso, então você usa T-SQL ou SSMS.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/performance/manage-the-query-store)

---

### Q172. Query Store real read-only vs solicitado read-write

**Questão:**
Você tem um banco de dados em uma instância SQL Server on Azure Virtual Machines. O estado atual do Query Store para o banco de dados é mostrado na exposição a seguir.

| Propriedade | Valor |
|---|---|
| Modo de Operação (Atual) | Ler apenas |
| Modo de Operação (Solicitado) | Ler a escrever |
| Intervalo de fluxo de dados (minutos) | 15 |
| *(linhas restantes, como Max Size (MB), Query Store Capture Mode e Size Based Cleanup Mode, são cortadas na captura de tela de origem)* | |

Use os menus suspensos para concluir as declarações.

**Alternativas:**

**Declaração 1: Para alterar o Modo de Operação (Atual) para Ler escrever sem perder nenhum dado, você deve modificar a configuração [escolha de resposta]:**
- **A.** Tamanho máximo (MB)
- **B.** Modo de captura Query Store
- **C.** Modo de limpeza baseado tamanho
- **D.** Modo de Operação (Solicitado)

**Declaração 2: Query Store reterá consultas de [escolha de resposta] para avaliação:**
- **A.** Todos
- **B.** da
- **C.** um conjunto seletivo de

**✅ Resposta:** Declaração 1: **Tamanho máximo (MB)**; Declaração 2: **um conjunto seletivo de**

**Explicação:** O modo solicitado já é Read write e o modo efetivo aparece como Read only. Esse é o sinal de que Query Store atingiu MAX_STORAGE_SIZE_MB. A Microsoft documenta que Query Store muda para READ_ONLY quando atinge o limite de espaço. Aumentar **Max Size (MB)** permite voltar ao modo read-write sem apagar dados. Habilitar Size Based Cleanup Mode liberaria espaço ao **remover** dados, o que descumpre o requisito de preservá-los. Alterar Operation Mode (Requested) não resolve porque o modo solicitado já é Read write. Na afirmação 2, Capture Mode, que não aparece nessa imagem, está definido como Auto na versão padrão desse exemplo. Esse é o padrão desde o SQL Server 2019. Auto ignora consultas pouco frequentes e consultas com tempo de compilação e execução pouco significativo, então só parte das consultas é capturada.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/performance/manage-the-query-store) [Doc 2](https://learn.microsoft.com/en-us/sql/relational-databases/performance/best-practice-with-the-query-store)

---

### Q173. Resource Governor: App1 tudo CPU, metade garantido

**Questão:**
Você tem uma assinatura do Azure que contém um Azure SQL managed instance, um banco de dados chamado db1 e um aplicativo web do Azure chamado App1. App1 usa db1. Você precisa habilitar Resource Governor para App1. A solução deve atender aos seguintes requisitos:
- App1 deve ser capaz de consumir todos os recursos CPU disponíveis.
- O App1 deve ter pelo menos metade dos recursos CPU disponíveis sempre disponíveis.

Que três ações você deve executar em sequência? (Mais de uma ordem de opções de resposta está correta.)

**Alternativas:**
Ações (lista visível):
- **A.** Crie um plano.
- **B.** Crie uma função de classificador em db1.
- **C.** Crie um grupo de carga de trabalho.
- **D.** Crie uma função de classificador no banco de dados mestre.
- **E.** Crie um pool de recursos que tenha as seguintes configurações: MAX_CPU_PERCENT = 100, MIN_CPU_PERCENT = 50

**✅ Resposta:**
1. Criar um conjunto de recursos que MAX_CPU_PERCENT = 100, MIN_CPU_PERCENT = 50
2. Criar um grupo de carga de trabalho (USING que piscina)
3. Criar uma função classificador no banco de dados mestre

**Explicação:** MIN_CPU_PERCENT = 50 reserva metade da CPU para o pool quando há contenção. MAX_CPU_PERCENT = 100 é um limite flexível que permite ao pool usar toda a CPU disponível. Um workload group é o contêiner em que as sessões são classificadas e deve referenciar um pool existente. A Microsoft orienta a criar a classifier function no banco master com schema binding. Uma função em db1 é inválida, e “Create a plan” não é um objeto do Resource Governor. A classifier function pode ser criada antes ou depois do workload group, pois retorna o nome do grupo como uma cadeia de texto. Por isso, mais de uma ordem é aceita. Crie o pool antes do grupo.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor-resource-pool) [Doc 2](https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor-classifier-function)

---

### Q174. Limite App2 IOPS em managed instance SQL1

**Questão:**
Você tem um Azure SQL managed instance chamado SQL1 e dois aplicativos web Azure chamados App1 e App2. Você precisa limitar o número de IOPs que as consultas App2 geram em SQL1. Quais duas ações você deve executar no SQL1? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** Ativar correções query optimizer.
- **B.** Habilitar Resource Governor.
- **C.** Habilitar parameter sniffing.
- **D.** Crie um grupo de carga de trabalho.
- **E.** Configurar In-memory OLTP.
- **F.** Execute o Database Engine Tuning Advisor.
- **G.** Reduza o valor máximo do grau de paralelismo.

**✅ Resposta:** B, D

**Explicação:** Resource Governor é apoiado em SQL Managed Instance pode "reservar ou limitar a quantidade de CPU, memória e I/O físico" que as cargas de trabalho usam. O I/O físico é limitado através do pool de recursos MIN/MAX_IOPS_PER_VOLUME Você ativa as configurações. Resource Governor e criar um grupo de carga de trabalho (ligado a uma piscina com MAX_IOPS_PER_VOLUME) que App2As outras opções são ajustar botões que não colocam um limite no I /O.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor) [Doc 2](https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor-resource-pool)

---

### Q175. SQLMI1: limite App1 CPU e memória, quatro passos

**Questão:**
Você tem um Azure SQL managed instance chamado SQLMI1 que tem o Resource Governor habilitado e é usado por dois aplicativos chamados App1 e App2. Você precisa configurar o SQLMI1 para limitar os recursos de memória e CPU que podem ser alocados para App1. Quais quatro ações você deve executar em sequência?

**Alternativas:**
Acções:
- **A.** Crie um grupo de carga de trabalho.
- **B.** Crie uma função de classificador definida pelo usuário.
- **C.** Modificar Resource Governor.
- **D.** Crie um contained database user.
- **E.** Crie um pool de recursos.

**✅ Resposta:**
1. Crie um pool de recursos.
2. Crie um grupo de carga de trabalho.
3. Crie uma função de classificador definida pelo usuário.
4. Modificar Resource Governor.

**Explicação:** O pool de recursos detém os limites CPU / memória (MAX_CPU_PERCENT, MAX_MEMORY_PERCENT). O grupo de carga de trabalho é criado USING esse pool. O classificador UDF (em master, schema-bound) rotas sessões do App1 para o grupo. Finalmente, `ALTER RESOURCE GOVERNOR WITH (CLASSIFIER_FUNCTION = dbo.fn) ; ALTER RESOURCE GOVERNOR RECONFIGURE;` registra o classificador e aplica a configuração, que a Microsoft lista como etapas necessárias. A contained database user não tem nada a ver com a governança dos recursos.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor-classifier-function)

---

### Q176. Desempenho ADatum DB3 após migração

**Estudo de caso:** ADatum v1 (serviços financeiros)

**Questão:**
*(Estudo de caso: DB3 está em SVR3 (SQL Server 2019), é usado por App2 e App3, e tem sua alocação de recursos gerenciada por Resource Governor. Ele será migrado para Azure SQL Database e adicionado a um auto-failover group. Exigência de disponibilidade: "Para DB3, gerencie possíveis problemas de desempenho causados por mudanças na demanda de recursos por App2 e App3.")*
Você precisa recomendar uma solução para garantir que o desempenho do DB3 seja otimizado após a migração para o Azure SQL Database. A solução deve atender aos requisitos de disponibilidade. O que você deve incluir na recomendação?

**Alternativas:**
- **A.** escala vertical
- **B.** um pool de recursos personalizado
- **C.** Resource Governor
- **D.** escala horizontal

**✅ Resposta:** A

**Explicação:** Em Azure SQL Database, "configuração do usuário de conjuntos de recursos e grupos de carga de trabalho... não é suportado", então o on-premises Resource Governor A abordagem (B e C) não pode ser transportada. Alterar a demanda de recursos de App2/App3 é tratado por escalabilidade DB3s tamanho de computação ou nível de serviço para cima ou para baixo (escala vertical). Essa é uma operação on-line com apenas uma breve reconexão, por isso se encaixa no requisito de disponibilidade. Escala horizontal (sharding : read scale-out) significa redesenhar o aplicativo e não atende à demanda de mudança em um único banco de dados.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor)

---

### Q177. Dados IO em 100% durante uma carga Table1

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1 que contém uma tabela chamada Table1. Você executa uma consulta para carregar dados no Table1. As métricas de desempenho do Table1 durante a operação de carga são mostradas na exposição a seguir.

Exposição: Azure portal gráfico métrico, cerca de 8:45 para depois 9:05 (UTC-04:00). "Dados IO Percentagem (Avg)" (azul): 0%, um breve pico para cerca de 35%, então a **planalto 100%** para a maior parte da carga, então uma gota para um constante 25%. " Dados IO Percentagem (Max)" (laranja): 0% até que a linha azul caia, então 100% e fica lá. Ambas as lendas mostram 100%.

Use os menus suspensos para selecionar a opção de resposta que completa cada declaração com base nas informações apresentadas no gráfico.

**Alternativas:**

**Declaração 1: Para reduzir o tempo que leva para completar a consulta, você deve [responder à escolha]:**
- **A.** escala o recurso
- **B.** use um recurso elástico
- **C.** executar afinação de consulta

**Declaração 2: Para reduzir a carga de log IO da operação, a consulta deve ser atualizada para usar a tabela [escolha de resposta]:**
- **A.** uma temporária
- **B.** e In-Memory OLTP durável
- **C.** e In-Memory OLTP não durável

**✅ Resposta:** Declaração 1: **escala o recurso**; Declaração 2: tabela **e In-Memory OLTP não durável**

**Explicação:** Declaração 1: Dados IO % é medido em relação ao limite do objetivo de serviço e fica em 100% para a maior parte da carga, de modo que a carga é estrangulada pelo limite de I/O do nível. orientação da Microsoft: "considere aumentar o nível de desempenho como estas percentagens perto 100% "Usar um recurso elástico" (elastic pool) não levanta o limite de I/O de um único banco de dados. Declaração 2: uma tabela otimizada para memória com `DURABILITY = SCHEMA_ONLY` (não durável) não gera E/S de registro de transações, o que o torna ideal para cargas de preparação. memory-optimized tables ainda log, e #temp tabelas ainda log em tempdb.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Baixa · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/monitoring-with-dmvs) [Doc 2](https://learn.microsoft.com/en-us/sql/relational-databases/in-memory-oltp/defining-durability-for-memory-optimized-objects)

---

### Q178. Reduza a porcentagem de log IO a um custo mínimo

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1 no nível de serviço General Purpose. As métricas de desempenho para o DB1 são mostradas na exposição a seguir.

*Exposição:* "Log IO porcentagem (Max)" gráfico de linha, 12: 25–12: 52: 0% até cerca de 12:42, em seguida, um platô plano 100% até cerca de 12:50, em seguida, uma queda para cerca de 37% e de volta para 0%.

Você precisa reduzir a porcentagem do Log IO. A solução deve minimizar os custos. O que você deve fazer?

**Alternativas:**
- **A.** Alterar nível de serviço para Business Critical.
- **B.** Aumente o número de vCores.
- **C.** Execute uma operação de checkpoint.
- **D.** Mudar o modelo de recuperação para simples.

**✅ Resposta:** B

**Explicação:** Na General Purpose O limite de taxa de logs escala com vCores (série standard: 4.5 MiB/s por vCore, então 2 vCores = 9, 4 = 18, 8 = 36, up to a 50 MiB/s cap). Adding vCores portanto, aumenta o teto de taxa de log e reduz Log IO e custa menos de Business Critical, que custa sobre 2.5–3x tanto porque adiciona réplicas. Um ponto de verificação não reduz a geração de logs. Azure SQL Database sempre utiliza o FULL recovery modelo, então você não pode mudá-lo para Simples.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/resource-limits-vcore-single-databases)

---

### Q179. Pressão de memória SQL MI, custo mínimo

**Questão:**
Você tem um Azure SQL managed instance chamado SQLMI1 que tem as seguintes configurações:
- vCores: 4
- Nível de serviço: General Purpose
- Geração de hardware: série Standard (Gen5)

Você descobre que a pressão da memória sobre SQLMI1 é alta. Você precisa reduzir a pressão da memória sobre o SQLMI1. A solução deve minimizar os custos. O que você deve fazer?

**Alternativas:**
- **A.** Ative o Query Store.
- **B.** Mudança vCores para 8.
- **C.** Mude a geração de hardware para a série Premium.
- **D.** Alterar nível de serviço para Business Critical.

**✅ Resposta:** C

**Explicação:** Série Standard dá 5.1 GB por vCore (4 vCores = 20.4 GB). Série Premium dá 7 GB por vCore (4 vCores = 28 GB, sobre 37% mais memória) para apenas sobre 15% mais por vCore. Dobrando para 8 vCores duplica aproximadamente o custo de computação e licença. Business Critical não adiciona memória por vCore (a série padrão permanece em 5.1 GB/vCore) e custa muito mais. Query Store não reduz a pressão da memória.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits)

---

### Q180. Erros transitórios Serverless após inatividade

**Questão:**
Você tem uma assinatura do Azure que contém os recursos mostrados na tabela a seguir.

| Nome | Tipo |
|---|---|
| App1 | Azure web app |
| db1 | Azure SQL database in the serverless tier |

App1 experimenta erros de conexão transientes e tempo limite quando tenta acessar db1 após longos períodos de inatividade. Você precisa modificar db1 para resolver os problemas experimentados pelo App1 o mais rápido possível, sem considerar custos imediatos. O que você deve fazer?

**Alternativas:**
- **A.** Habilitar automatic tuning para db1.
- **B.** Aumentar o número de vCores atribuído ao db1.
- **C.** Diminua o atraso auto-pause para db1.
- **D.** Desativar o atraso auto-pause para db1.

**✅ Resposta:** D: Desabilite o atraso auto-pause para db1.

**Explicação:** A banco de dados A serverless pausa automaticamente após o atraso auto-pause. A primeira conexão após uma pausa aciona um auto-resume, e essa conexão falha com um erro transitório até que o banco de dados esteja on-line novamente. Desativar o auto-pause mantém o banco de dados on-line o tempo todo e custa mais, o que a pergunta permite. Um atraso mais curto torna as pausas mais frequentes. Mais vCores e automatic tuning não afetam a pausa/resumo.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview)

---

### Q181. Erro de limite de armazenamento Elastic pool 1132

**Questão:**
Você tem um Azure SQL Database elastic pool que contém 10 bancos de dados. Você recebe o seguinte alerta.

```
Msg 1132, Level 16, State 1, Line 1
The elastic pool has reached its storage limit. The storage used for the elastic pool cannot exceed (76800) MBs.
```

Você precisa resolver o alerta. A solução deve minimizar o esforço administrativo. Quais três ações você pode executar? Cada resposta correta apresenta uma solução completa.

**Alternativas:**
- **A.** Aumente o armazenamento máximo do elastic pool.
- **B.** Excluir dados de um banco de dados.
- **C.** Remova um banco de dados da piscina.
- **D.** Habilitar data compression.
- **E.** Encolher bancos de dados individuais.

**✅ Resposta:** A, C, E

**Explicação:** O limite da piscina aplica-se ao espaço de dados *afectado*. Cada um deles é uma correção completa por conta própria: aumentar o tamanho máximo da piscina, tirar um banco de dados (e seu espaço alocado) da piscina ou encolher arquivos de dados para recuperar o espaço alocado, mas não usado. A exclusão de dados (B) ou compactação (D) reduz o espaço *Utilizado*, mas nunca reduz o espaço alocado automaticamente, portanto, nenhum dos dois corrige o erro sem um psiquiatra depois.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/file-space-manage)

---

### Q182. Minimize o espaço de dados e arquivos de log

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado sqldb1. Você precisa minimizar a quantidade de espaço usado pelos dados e arquivos de log do sqldb1. O que você deve executar?

**Alternativas:**
- **A.** DBCC SHRINKDATABASE
- **B.** sp_clean_db_free_space
- **C.** sp_clean_db_file_free_space
- **D.** DBCC SHRINKFILE

**✅ Resposta:** A

**Explicação:** `DBCC SHRINKDATABASE` encolhe todos os arquivos de log **e** do banco de dados em um comando. `DBCC SHRINKFILE` tem como alvo um arquivo por comando. `sp_clean_db_free_space` e `sp_clean_db_file_free_space` apenas removem resíduos de registros fantasmas das páginas e não reduzem o tamanho do arquivo.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/file-space-manage)

---

### Q183. Tempdb PAGELATCH_UP: adicionar arquivos tempdb

**Questão:**
*Observação: Esta questão faz parte de uma série de perguntas que apresentam o mesmo cenário. Cada pergunta da série contém uma solução única que pode atender aos objetivos declarados.*

Você tem SQL Server 2019 em um Azure virtual machine. Você está solucionando problemas de desempenho para uma consulta em uma instância SQL Server. Para obter mais informações, consulte `sys.dm_exec_requests` e descubra que o wait type é `PAGELATCH_UP` e o wait_resource é `2:3:905856`. Você precisa melhorar o desempenho do sistema.

Solução: Você cria arquivos Tempdb adicionais.

Isso cumpre o objetivo?

**Alternativas:**
- **A.** Sim
- **B.** Não

**✅ Resposta:** A (Sim)

**Explicação:** Esperar recurso `2:3:905856` means database 2 (tempdb), file 3, page 905856. 905856 = 8088 × 112, portanto é um PFS página, e as esperas mostram tempdb alocação-página (PFS) contenção. A correção da Microsoft é adicionar arquivos de dados tempdb igualmente dimensionados: um por processador lógico até 8, depois mais em etapas de 4. As alocações são então espalhadas por mais PFS/GAM/SGAM páginas.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention)

---

### Q184. Tempdb PAGELATCH_UP: mestre autogrow 10%

**Questão:**
*(O mesmo cenário da série como Q183: SQL Server 2019 em uma Azure VM. `sys.dm_exec_requests` mostra wait type `PAGELATCH_UP` e wait_resource `2:3:905856`. Você precisa melhorar o desempenho do sistema.)*

Solução: Você altera o arquivo de dados para o banco de dados mestre para crescer automaticamente em 10%.

Isso cumpre o objetivo?

**Alternativas:**
- **A.** Sim
- **B.** Não

**✅ Resposta:** B (Não)

**Explicação:** A disputa está nas páginas de alocação do tempdb (database_id 2), não no mestre (database_id 1). A mudança do autocrescimento do mestre não faz nada pela contenção de trava no tempdb.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention)

---

### Q185. Tempdb PAGELATCH_UP: reduzir objetos temporários

**Questão:**
*(O mesmo cenário da série como Q183: SQL Server 2019 em uma Azure VM. `sys.dm_exec_requests` mostra wait type `PAGELATCH_UP` e wait_resource `2:3:905856`. Você precisa melhorar o desempenho do sistema.)*

Solução: Você reduz o uso de variáveis de tabela e tabelas temporárias.

Isso cumpre o objetivo?

**Alternativas:**
- **A.** Sim
- **B.** Não

**✅ Resposta:** A (Sim)

**Explicação:** A contenção da página de alocação do Tempdb vem da criação e queda de muitos objetos temporários (tabelas temporárias e variáveis de tabela) com muita frequência. As listas de orientação da Microsoft reduzem essa carga de trabalho ou alteram o código, como uma mitigação juntamente com a adição de arquivos de dados.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention)

---

### Q186. Tempdb PAGELATCH_UP: log de transação de encolhimento

**Questão:**
*(O mesmo cenário da série como Q183: SQL Server 2019 em uma Azure VM. `sys.dm_exec_requests` mostra wait type `PAGELATCH_UP` e wait_resource `2:3:905856`. Você precisa melhorar o desempenho do sistema.)*

Solução: Você encolhe o arquivo de log da transação.

Isso cumpre o objetivo?

**Alternativas:**
- **A.** Sim
- **B.** Não

**✅ Resposta:** B (Não)

**Explicação:** `PAGELATCH_UP` em uma página tempdb PFS é na memória trava de contenção em páginas de alocação em um arquivo de dados. Encolher um arquivo de log de transação não toca isso e pode até prejudicar o desempenho.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention)

---

### Q187. Arquivos de dados Tempdb após escala para 16 vCPUs

**Questão:**
Você tem SQL Server 2019 em um Azure virtual machine que executa Windows Server 2019. A máquina virtual tem 4 vCPUs e 28 GB de memória. Você escala a máquina virtual para 16 vCPUs e 64 GB de memória. Você precisa fornecer a menor latência para o tempdb. Qual é o número total de arquivos de dados que o tempdb deve conter?

**Alternativas:**
- **A.** 2
- **B.** 4
- **C.** 8
- **D.** 64

**✅ Resposta:** C (8)

**Explicação:** Com mais de 8 processadores lógicos, comece com 8 arquivos de dados tempdb de tamanho igual. Adicione arquivos em múltiplos de 4, até o número de processadores lógicos, apenas se a contenção continuar. 64 arquivos seriam demais e 2 ou 4 deixariam a contenção.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention)

---

### Q188. Arquivos de dados secundários Tempdb em 8 vCPUs

**Questão:**
Você tem SQL Server 2019 em um Azure virtual machine que executa Windows Server 2019. A máquina virtual tem 4 vCPUs e 28 GB de memória. Você escala a máquina virtual para 8 vCPUs e 64 GB de memória. Você precisa reduzir a contenção do tempdb sem afetar negativamente o desempenho do servidor. Qual é o número de arquivos de dados secundários que você deve configurar para o tempdb?

**Alternativas:**
- **A.** 2
- **B.** 4
- **C.** 8
- **D.** 64

**✅ Resposta:** C (8)

**Explicação:** Com 8 ou menos processadores lógicos, use um arquivo de dados tempdb por processador lógico, que é 8 aqui. Estritamente falando, 8 arquivos significa 1 primário mais 7 arquivos secundários. 7 não é uma opção, e o exame trata "arquivos de dados" e "arquivos de dados secundários" como a mesma coisa aqui, então 8 é a resposta pretendida. 2 ou 4 deixaria contenda, e 64 adiciona sobrecarga sem benefício.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention)

---

### Q189. Arquivos de dados totais Tempdb a 8 vCPUs

**Questão:**
Você tem SQL Server 2019 em um Azure virtual machine que roda Windows Server 2019. A máquina virtual tem 4 vCPUs e 28 GB de memória. Você escala a máquina virtual para 8 vCPUs e 64 GB de memória. Você precisa fornecer a menor latência para o tempdb. Qual é o número total de arquivos de dados que o tempdb deve conter?

**Alternativas:**
- **A.** 2
- **B.** 4
- **C.** 8
- **D.** 64

**✅ Resposta:** C (8)

**Explicação:** Com 8 ou menos processadores lógicos, use tantos arquivos de dados tempdb de tamanho igual como processadores lógicos, então 8 arquivos para 8 vCPUs.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention)

---

<a name="d4--configure-and-manage-automation-of-tasks-1520"></a>

## D4 — Configurar e gerenciar a automação de tarefas (15–20%)

*28 questões*

### Q190. Operador de e-mail quando o agente job falhar

**Questão:**
Você tem SQL Server em um Azure virtual machine chamado SQL1. SQL1 tem um agente job para fazer backup de todos os bancos de dados. Você adiciona um usuário chamado dbadmin1 como um operador SQL Server Agent. Você precisa garantir que o dbadmin1 receba um alerta por e-mail se um job falhar.

Para responder, mova as ações apropriadas da lista de ações para a área de resposta e organize-as na ordem correta.

**Alternativas:**
Acções:
- **A.** Criar um alerta job
- **B.** Criar uma notificação job
- **C.** Ativar Database Mail
- **D.** Ativar as configurações de email para o SQL Server Agent
- **E.** Criar um alvo job

**✅ Resposta:**
1. Ativar Database Mail
2. Ativar as configurações de email para o SQL Server Agent
3. Criar uma notificação job

**Explicação:** SQL Server Agent envia e-mail através Database Mail, então Database Mail (com uma conta e perfil) deve existir primeiro. Em seguida, habilite o perfil de correio em SQL Server Agent (Propriedades do agente > Alert System > Enable mail profile, em seguida, reinicie o agente). Por último, no jobPágina de Notificações de 's, set "Email dbadmin1 quando o job falha". A job *Alerta* resposta a erros ou eventos de desempenho, não a um jobresultado final. A job *alvo* é para multi-servidor (MSX/TSX) jobs.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/database-mail/configure-sql-server-agent-mail-to-use-database-mail)

---

### Q191. Agent job passo acesso ao compartilhamento de arquivos

**Questão:**
Você tem uma assinatura do Azure que contém uma instância de SQL Server on Azure Virtual Machines chamada SQLVM1 e uma máquina virtual chamada Server1 que executa Windows Server. SQLVM1 e Server1 são unidos a um domínio Active Directory Domain Services (AD DS). Server1 hospeda uma compartilhamento de arquivos chamada Share1.

Você precisa garantir que um passo SQL Server Agent job no SQLVM1 possa acessar os arquivos em Share1. A solução deve usar o princípio do menor privilégio.

Quais ações você deve executar em sequência?

**Alternativas:**
Acções:
- **A.** Atribua o proxy ao passo job.
- **B.** Crie um login.
- **C.** Crie um usuário de banco de dados.
- **D.** Crie uma credencial.
- **E.** Crie um proxy.

**✅ Resposta:**
1. Crie uma credencial.
2. Crie um proxy.
3. Atribua o proxy ao passo job.

**Explicação:** A proxy A credencial armazena uma conta de domínio que tem direitos sobre Share1. Uma vez o proxy existe, você o habilita para o subsistema certo (por exemplo, CmdExec : PowerShell) e selecione-o no job "Corra como". Só que job em seguida, o passo é executado sob a conta de acesso a ações, não toda a conta de serviço do agente, o que é menos privilégio. login ou o usuário do banco de dados não dá acesso ao sistema de arquivos.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/ssms/agent/create-a-sql-server-agent-proxy)

---

### Q192. Ativar notificações de e-mail do agente no SQL MI

**Questão:**
Você tem um Azure SQL managed instance. Você precisa ativar as notificações de e-mail SQL Agent Job. O que você deve fazer?

**Alternativas:**
- **A.** Use a opção Agente XPs.
- **B.** Ative o SQL Server Agent.
- **C.** Execute o comando sp_configure.
- **D.** Execute o comando sp_set_agent_properties.

**✅ Resposta:** C

**Explicação:** Ligado SQL Managed Instance você permitir Database Mail com `sp_configure` (`'show advanced options', 1`então, `'Database Mail XPs', 1`então, `RECONFIGURE`). Você então cria a Database Mail perfil nomeado `AzureManagedInstance_dbmail_profile` Atribuir operadores e notificações ao jobs. Microsoft Learn diz claramente que SQL Agent as configurações são read-only E-mail MI e que `sp_set_agent_properties` Não é suportado, então D não pode funcionar. SQL Agent sempre corre em MI e não pode ser ativado ou desativado (B). Agente XPs (A) não tem nada a ver com o correio.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/job-automation-managed-instance) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/transact-sql-tsql-differences-sql-server)

---

### Q193. MSX / TSX escolha de implantação de administração multiserver

**Questão:**
Você tem uma assinatura do Azure. Você planeja migrar 10 instâncias on-premises Microsoft SQL Server para o Azure. Você precisa garantir que o ambiente migrado possa ser gerenciado usando a administração multiservidor e suporta master / target (MSX / TSX) jobs. A solução deve minimizar o esforço administrativo. Quais opções de implantação SQL você deve selecionar como servidor mestre (MSX) e o servidor alvo (TSX)?

**Alternativas:**

**MSX:**
- **A.** SQL database
- **B.** instâncias gerenciadas SQL
- **C.** Máquinas virtuais SQL

**TSX:**
- **A.** SQL database
- **B.** instâncias gerenciadas SQL
- **C.** Máquinas virtuais SQL

**✅ Resposta:** MSX: **Máquinas virtuais SQL**. TSX: **Máquinas virtuais SQL**.

**Explicação:** Azure SQL Database não tem SQL Server Agent De todo. Microsoft Learn's MI T-SQL A página de diferenças diz: "O recurso Administração Multi Servidor para mestre / alvo (MSX/TSX) jobs não são suportados." Apenas cheio SQL Server (em Azure VMs) supports both MSX e TSX Tanto o mestre quanto os alvos devem, portanto, ser SQL Server on Azure VMs.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/transact-sql-tsql-differences-sql-server)

---

### Q194. OS direito para a conta SQL Agent

**Questão:**
Você tem uma instância de SQL Server on Azure Virtual Machines chamada VM1. Você planeja agendar um SQL Server Agent job que reconstruirá índices das bancos de dados hospedadas no VM1. Você precisa configurar a conta que será usada pelo agente. A solução deve usar o princípio do menor privilégio. Qual direito do usuário do sistema operacional você deve atribuir à conta?

**Alternativas:**
- **A.** Aumentar a prioridade de agendamento
- **B.** Faça logon como um serviço
- **C.** Desempenho do sistema de perfil
- **D.** Faça logon como um lote job

**✅ Resposta:** B

**Explicação:** A conta de serviço SQL Server Agent precisa do **Faça logon como um serviço** direito do Windows (SeServiceLogonRight), juntamente com alguns direitos de token e cota padrão. Uma reconstrução de índice é executada dentro do motor de banco de dados, para que não sejam necessários outros direitos OS. "Log on as a batch job" aplica-se a contextos de tarefa agendada e proxy, e os outros dois direitos não estão relacionados.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/previous-versions/sql/2014/ssms/agent/configure-sql-server-agent?view=sql-server-2014)

---

### Q195. gravidade sp_add_alert 20 declarações de notificação

**Questão:**
Você tem uma assinatura do Azure que contém uma instância SQL Server em uma máquina virtual chamada Server1. Você executa a seguinte declaração T-SQL no Server1.

```sql
EXEC msdb.dbo.sp_add_alert @name=N'Fatal Error',
  @message_id=0,
  @severity=20,
  @enabled=1,
  @delay_between_responses=0,
  @include_event_description_in=1;
GO

EXEC msdb.dbo.sp_add_operator
  @name = 'DBA-Team',
  @enabled = 1,
  @email_address = 'DBA-Team@contoso.com';
GO

EXEC msdb.dbo.sp_add_notification
  @alert_name = N'Fatal Error',
  @operator_name = 'DBA-Team',
  @notification_method = 1;
GO
```

Para cada uma das seguintes afirmações, selecione Sim se a declaração for verdadeira, caso contrário selecione Não.

**Alternativas:**
- S1. Um alerta será gerado quando ocorrer um erro fatal.
- S2. Um SQL Server Agent job será executado quando um alerta for gerado.
- S3. Um e-mail será enviado para DBA -Team@contoso.com quando um alerta é gerado.

**✅ Resposta:** S1 **Sim**. S2 **Não**. S3 **Sim**.

**Explicação:** A gravidade 20 é um erro fatal ("erro fatal no processo atual"). Erros de gravidade 19–25 são sempre escritos para o log de aplicativos do Windows, de modo que os incêndios de alerta habilitados (S1). `sp_add_alert` dá não `@job_id`/`@job_name`, então não job runs em resposta (S2). `sp_add_notification` com `@notification_method = 1` O e-mail para o DBAOperador -Team (S3). Isso assume o correio do agente (Database Mail) está configurado, que é a leitura pretendida do exame.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/ro-ro/ssms/agent/create-an-alert-using-severity-level)

---

### Q196. Função para configurar as contas do Agente proxy

**Questão:**
Você tem uma instância de SQL Server on Azure Virtual Machines. Você precisa garantir que um usuário chamado User1 possa configurar contas proxy para SQL Server Agent jobs. A solução deve usar o princípio do menor privilégio. Qual papel você deve atribuir ao User1?

**Alternativas:**
- **A.** sysadmin
- **B.** SQLAgentUserRole
- **C.** SQLAgentReaderRole
- **D.** SQLAgentOperatorRole

**✅ Resposta:** A

**Explicação:** Somente os membros do papel de servidor fixo **sysadmin** podem criar, modificar ou excluir contas SQL Server Agent proxy. As funções do agente msdb podem no máximo *utilizar* proxies concedidas a eles e o SQLAgentOperatorRole pode visualizar as propriedades proxy. Nenhum deles pode criar proxies, mesmo que pareçam menos privilegiados.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/ssms/agent/create-a-sql-server-agent-proxy) [Doc 2](https://learn.microsoft.com/en-us/ssms/agent/sql-server-agent-fixed-database-roles)

---

### Q197. Papel de agente para criar e ver jobs

**Questão:**
Você tem uma assinatura do Azure que contém uma instância de SQL Server on Azure Virtual Machines chamada SQLVM1 e um usuário chamado User1. SQLVM1 hospeda um banco de dados chamado DB1. Você precisa garantir que o User1 possa executar as seguintes tarefas no DB1:
- Criar jobs.
- Ver tudo jobs.
- Modifique, exclua e desative o jobs que o usuário criou.

A solução deve usar o princípio do menor privilégio. Qual função de banco de dados embutida você deve atribuir ao User1, e onde a função é definida?

**Alternativas:**

**Localização:**
- **A.** DB1
- **B.** mestre
- **C.** msdb

**Papel incorporado:**
- **A.** SQLAgentOperatorRole
- **B.** SQLAgentReaderRole
- **C.** SQLAgentUserRole

**✅ Resposta:** Localização: **msdb**. Papel embutido: **SQLAgentReaderRole**.

**Explicação:** O SQL Server Agent funções de banco de dados fixo existem apenas em **msdb**. SQLAgentUserRole permite que um usuário crie e gerencie apenas suas *própria* jobs e ver apenas aqueles, por isso não pode "ver todos jobs". SQLAgentReaderRole A visualização de todos os locais e multi-servidores jobs e sua história, e ainda limita modificar, excluir e habilitar/desativar a propriedade jobs. Isso é uma correspondência exata. SQLAgentOperatorRole também permite ao usuário executar, parar e ativar/desativar *Todos* jobs, o que é mais do que necessário.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/ssms/agent/sql-server-agent-fixed-database-roles)

---

### Q198. Papel de agente menos privilegiado para agendar backup job

**Questão:**
Você tem uma assinatura do Azure que contém uma instância de SQL Server em um Azure virtual machine nomeado SQLVM1 e um usuário chamado User1. SQLVM1 hospeda um banco de dados chamado DB1. User1 tem a db_backupoperator Você precisa garantir que User1 pode criar uma tarefa agendada para executar um full backup E-mail DB1. A solução deve usar o princípio do menor privilégio. Qual função de banco de dados embutida você deve atribuir a User1?

**Alternativas:**
- **A.** db_owner
- **B.** SQLAgentReaderRole
- **C.** SQLAgentUserRole
- **D.** SQLAgentOperatorRole

**✅ Resposta:** C

**Explicação:** Uma tarefa agendada em SQL Server = a SQL Server Agent job. SQLAgentUserRole (msdb) é o papel de agente menos privilegiado e pode criar / modificar / deletar e executar seu próprio local jobs e job Horários. SQLAgentReaderRole e SQLAgentOperatorRole incluir esses direitos e mais (por isso, não menos importante privilégio); db_owner E-mail DB1 não concede acesso ao agente e é muito mais amplo. User1 já tem db_backupoperator, então o backup próprio é permitido.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/ssms/agent/sql-server-agent-fixed-database-roles)

---

### Q199. Papel necessário para criar o Agente proxy

**Questão:**
Você tem uma assinatura do Azure que contém uma instância de SQL Server em um Azure virtual machine nomeado VM1 e a Microsoft Entra Domínio de serviços de domínio que contém dois usuários nomeados User1 e User2. Na instância padrão de SQL Server E-mail VM1, você cria uma credencial chamada Credential1 E User1. Você precisa garantir que User2 pode criar a SQL Server Agent proxy que usará Credential1. A solução deve usar o princípio do menor privilégio. Qual papel você deve atribuir User2?

**Alternativas:**
- **A.** SQLAgentUserRole
- **B.** SQLAgentReaderRole
- **C.** SQLAgentOperatorRole
- **D.** sysadmin

**✅ Resposta:** D

**Explicação:** Somente os membros do servidor fixo sysadmin podem criar, modificar ou excluir SQL Server Agent proxy As três funções do agente msdb só podem *utilizar* proxies foram concedidos (Criar/modificar/excluir Proxies = No even for SQLAgentOperatorRole), então sysadmin é o privilégio menos que funciona.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/ssms/agent/create-a-sql-server-agent-proxy) [Doc 2](https://learn.microsoft.com/en-us/ssms/agent/sql-server-agent-fixed-database-roles)

---

### Q200. Tipo de recurso de alerta para CPU em todas Os bancos de dados

**Questão:**
Você tem vários Azure SQL databases no mesmo servidor Azure SQL Database em um grupo de recursos chamado ResourceGroup1. Você deve ser alertado quando o uso do CPU exceder 80% para qualquer banco de dados. A solução deve se aplicar a quaisquer bancos de dados adicionais criados no servidor Azure SQL. Que tipo de recurso você deve usar para criar o alerta?

**Alternativas:**
- **A.** Grupos de Recursos
- **B.** Servidores SQL
- **C.** Bancos de Dados SQL
- **D.** Máquinas Virtuais SQL

**✅ Resposta:** C

**Explicação:** CPU é uma métrica do recurso de banco de dados (Microsoft.Sqlservidores/bancos de dados), e Azure Monitor suportes *multi-recursos* regras de alerta métrica para "SQL Server bancos de dados" (Microsoft.Sql//servers/databases). No portal você escolhe o tipo de recurso SQL a regra para o grupo servidor/recurso (não individual DBs), então uma regra abrange todos os bancos de dados atuais e futuros desse tipo nesse escopo. O servidor lógico (Microsoft.Sql/servers) o tipo de recurso não expõe a banco de dados CPU métrica por cento para uma regra multi-recursos; "Grupos de Recursos" é apenas o recipiente de escopo, não o tipo de recurso a que a métrica pertence; SQL VMs são irrelevantes.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-types) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/alerts-create?view=azuresql)

---

### Q201. Notificar sobre alterações de tamanho/nível do banco de dados

**Questão:**
Você tem 50 Azure SQL databases. Você precisa notificar o proprietário do banco de dados quando as configurações do banco de dados, como o tamanho do banco de dados e a camada de preços, forem modificadas no Azure. O que você deve fazer?

**Alternativas:**
- **A.** Crie uma configuração de diagnóstico para o log de atividade que tenha o log de segurança ativado.
- **B.** Para o banco de dados, crie uma configuração de diagnóstico que tenha a métrica InstanceAndAppAdvanced ativada.
- **C.** Crie uma regra de alerta que use um tipo de sinal métrico.
- **D.** Crie uma regra de alerta que usa um tipo de sinal de registro de atividade.

**✅ Resposta:** D

**Explicação:** A alteração de tamanho ou nível de preços é uma operação de escrita no plano de controle (Microsoft.Sql /servidores/bancos de dados/gravação) registrada no log de atividades do Azure; uma regra de alerta de registro de atividade dispara nesse evento e notifica através de um grupo de ação. Configurações de diagnóstico (A, B) apenas route logs/metrics e não envie nenhuma notificação; um alerta métrico (C) observa valores numéricos, não eventos de mudança de configuração.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-types)

---

### Q202. SIM: alerta de e-mail db1 CPU sobre 50%

**Questão:**
SIMULATION. Você precisa gerar um alerta de e-mail para o db1 se a utilização percentual média do CPU for maior que 50% por cinco minutos amostrados em intervalos de um minuto. O alerta deve ser enviado para o admin@contoso.com. Você pode precisar usar o SQL Server Management Studio e o Azure portal.

**Alternativas:**

**✅ Resposta:** Criar uma regra de alerta métrico Azure Monitor em db1 (Azure portal):
1. Azure portal → SQL bancos de dados → **db1** → **Monitoramento > Alertas** → **+ Criar > Regra de alerta** (scope = db1).
2. **Condição** tab → Ver todos os sinais → **% CPU** → Aplicar.
3. Threshold = **Estática**, Aggregation type = **Média**, Operator = **Maior que**, Threshold value = **50**.
4. Quando avaliar: **Check every = 1 minute**, **Lookback period = 5 minutes** (mais velho UI: Frequência de avaliação 1 minuto, granularidade de agregação / período 5 minutos).
5. **Acções** tab → **Criar grupo de ação** (ou ação rápida) → Notificações: tipo **E-mail/ Mensagem SMS/Push/Voice** → Email = **admin@contoso.com** → OK → criar/anexar o grupo de ação.
6. **Detalhes** guia → Digite um nome de regra alerta (e.g., db1-cpu-50), gravidade como desejado, Habilitar após a criação.
7. **Revisão + criar** → **Criar**.

**Explicação:** A porcentagem CPU é uma métrica de plataforma do banco de dados, portanto, um alerta métrico com agregação média ao longo de uma janela de 5 minutos avaliada a cada minuto corresponde "média > 50% por cinco minutos amostrados em intervalos de um minuto"; a entrega por e-mail é feita por um grupo de ação. SSMS não é necessário.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/alerts-create?view=azuresql)

---

### Q203. Reduzir alertas ruidosos: limiar dinâmico, baixa sensibilidade

**Questão:**
Você recebe inúmeros alertas do Azure Monitor para uma instância Azure SQL Database. Você precisa reduzir o número de alertas. Você só deve receber alertas se houver uma mudança significativa nos padrões de uso por um período prolongado. Quais duas ações você deve executar? Cada resposta correta apresenta parte da solução. NOTE: Cada seleção correta vale um ponto.

**Alternativas:**
- **A.** Sensibilidade do Limiar ao Alto
- **B.** Defina o limite lógico de alerta para dinâmico
- **C.** Definir o limite lógico de Alerta para Estático
- **D.** Sensibilidade de limiar definido para baixo
- **E.** Definir plano de força para On

**✅ Resposta:** B, D

**Explicação:** Dynamic thresholds aprendem o padrão histórico e sazonal da métrica e alertam sobre desvios relevantes. Com Threshold sensitivity em Low, os limites ficam mais amplos e os alertas disparam apenas em desvios maiores, o que reduz a quantidade de alertas. High usa limites mais estreitos e gera mais alertas. Static thresholds não acompanham padrões de uso. Force Plan é uma configuração de Automatic Tuning, sem relação com alertas.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-dynamic-thresholds) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/alerts-create?view=azuresql)

---

### Q204. Notificação de voz para eventos de manutenção MI

**Questão:**
Você gerencia 100 instâncias gerenciadas Azure SQL localizadas em 10 regiões do Azure. Você precisa receber notificações de mensagem de voz quando um evento de manutenção afeta qualquer uma das 10 regiões. A solução deve minimizar o esforço administrativo. O que você deve fazer?

**Alternativas:**
- **A.** A partir do Azure portal, crie um alerta de saúde de serviço.
- **B.** A partir do Azure portal, crie um alerta de excelência operacional Azure Advisor.
- **C.** A partir do Microsoft SQL Server Management Studio (SSMS), configure um SQL Server agent job.
- **D.** A partir do Azure portal, configure um alerta de log de atividade.

**✅ Resposta:** A

**Explicação:** Alertas de saúde do serviço notificam você sobre interrupções e manutenção planejada que afetam os serviços / regiões que você usa; uma regra pode cobrir todas as 10 regiões e SQL Managed Instance, e seu grupo de ação pode incluir uma notificação de voz. Os alertas do consultor são para recomendações; um agente job não pode ver a manutenção do Azure; um alerta de registro de atividade genérico sobre operações de recursos não é a opção construída especificamente (categoria Saúde do Serviço) - Os alertas de saúde do serviço são tecnicamente um tipo de alerta de registro de atividades, mas o Aef é o alerta de resposta pretendido.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-types) [Doc 2](https://learn.microsoft.com/en-us/azure/azure-sql/database/alerts-create?view=azuresql)

---

### Q205. Litware: SalesSQLDb1 manutenção de estatísticas runbook

**Estudo de caso:** Litware

**Questão:**
(Case study — Litware. SalesSQLDb1 é uma Azure SQL database E- elastic pool SalesSQLDb1Pool e tem problemas de desempenho provavelmente devido a estatísticas desatualizadas. Requisitos técnicos: tarefas de manutenção devem ser automatizadas; uso de on-premises infra-estrutura deve ser minimizada.)
Você precisa implementar a manutenção estatística para SalesSQLDb1. A solução deve atender aos requisitos técnicos. Quais quatro ações você deve executar em sequência? Para responder, mova as ações apropriadas da lista de ações para a área de resposta e organize-as na ordem correta.
*(A captura de tela da fonte está faltando o cabeçalho da pergunta; caule e ações estão completas.)*

**Alternativas:**
Acções:
- **A.** Crie e configure um cronograma.
- **B.** Crie um SQL Server Agent job.
- **C.** Publique o runbook.
- **D.** Crie uma conta Azure Automation.
- **E.** Importe o módulo SqlServer.
- **F.** Crie um runbook que execute um script PowerShell.
- **G.** Execute sp_add_jobserver.

**✅ Resposta:** 1. Criar uma conta Azure Automation → 2. Importar o módulo SqlServer → 3. Criar um runbook que executa um script PowerShell → 4. Publicar o runbook

**Explicação:** Azure SQL Database (único DB/elastic pool) não tem SQL Server AgentAssim, o agente-job e sp_add_jobserver opções estão fora, e executá-los a partir de um on-premises : VM SQL Server violaria "minimizar on-premises infra-estrutura”. Azure Automation é a opção de nuvem. Crie a conta e, em seguida, importe SqlServer (ele fornece o módulo Invoke-Sqlcmd e não está lá por padrão). Em seguida, escreva o PowerShell runbook que atualiza as estatísticas, depois publica-as, porque apenas uma publicação runbook pode ser iniciado ou vinculado a um cronograma. Na prática, você vincularia um cronograma como um quinto passo; DP-300 laboratório não conta → SqlServer módulo → credencial → runbook → publicar → Com apenas quatro slots, a publicação deve vir em primeiro lugar, então o cronograma é o passo deixado de fora.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://microsoftlearning.github.io/dp-300-database-administrator/Instructions/Labs/13-deploy-automation-runbook-rebuild-indexes.html)

---

### Q206. Recriar App1Dev em App1Test (ARM + SSDT)

**Questão:**
Você tem um resource group chamado App1Dev, que contém um servidor do Azure SQL Database chamado DevServer1. DevServer1 contém o banco DB1. O schema e as permissões de DB1 estão salvos em um database project do Microsoft SQL Server Data Tools (SSDT). Você precisa preencher um novo resource group, App1Test, com o banco DB1 e um servidor Azure SQL chamado TestServer1. Os recursos de App1Test precisam ter as mesmas configurações dos recursos de App1Dev. Quais quatro ações você deve executar, em ordem?

**Alternativas:**
Acções:
- **A.** Alterar o administrador Active Directory no TestServer1
- **B.** Alterar o nome do servidor e variáveis relacionadas nos modelos
- **C.** A partir do projeto de banco de dados, implantar o esquema de banco de dados e permissões
- **D.** Adicionar endereços IP ao firewall
- **E.** A partir do Azure portal, exportar os modelos Azure Resource Manager
- **F.** A partir do Azure portal, implantar os modelos.

**✅ Resposta:** 1. A partir do Azure portal, exportar os modelos Azure Resource Manager → 2. Alterar o nome do servidor e variáveis relacionadas nos modelos → 3. A partir do Azure portal, implantar os modelos → 4. A partir do projeto de banco de dados, implantar o esquema de banco de dados e permissões

**Explicação:** Exportando o App1Dev ARM Os modelos de templates capturam a configuração existente do servidor/banco de dados (incluindo regras de firewall e Entra admin), portanto, nada precisa ser alterado manualmente.TestServer1), e variáveis relacionadas, implantar para App1TestEm seguida, publique o SSDT projeto para recriar DB1schema e permissões. Mudando o AD O administrador faria a configuração diferente de App1Deve o firewall IPs Já estão no template.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/training/modules/configure-automatic-deployment-azure-sql-database/3-automate-deployment-using-azure-resource-manager-bicep)

---

### Q207. Modelo ARM: tipo de servidor e dependsOn

**Questão:**
Você tem uma assinatura do Azure. Você planeja implantar um Azure SQL database usando um modelo Azure Resource Manager. Como você deve completar o modelo?
```
{
  "resources": [
   {
    "type": [Box 1],
    "apiVersion": "2020-02-02-preview",
    "name": "[parameters('name1')]",
    "location": "[parameters('location')]",
   ...
    "resources": [
      {
       "type": "databases",
       "apiVersion": "2020-02-02-preview",
   ...
       [Box 2]
       "[resourceId('Microsoft.Sql/servers',concat(parameters('name1')))]"
   ...
  }
```

**Alternativas:**

**Caixa 1:**
- **A.** "Microsoft.Sql /servidores"
- **B.** "Microsoft.SqlVirtualMachines/sqlVirtualMachines"
- **C.** "Microsoft.Synapse /workspaces/sqldatabases"

**Caixa 2:**
- **A.** "dependsOn":[
- **B.** "propriedades":[
- **C.** "tags":[

**✅ Resposta:** Caixa 1: "Microsoft.Sql / servidores"; Caixa 2: " dependsOn ":[

**Explicação:** An Azure SQL database é uma criança (bancos de dados) de um servidor lógico, tipo de recurso Microsoft.SqlA banco de dados da criança deve ser implantada após o servidor, expressa por um dependsOn array contendo resourceId('Microsoft.Sql/servers', name). propriedades e tags são objetos, não listas de recursos IDs.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/sql-database/sql-database-single-database-get-started-template)

---

### Q208. Modelo ARM: MI para consultas de banco de dados cruzado

**Questão:**
Você tem uma assinatura do Azure. Você precisa implantar um recurso Azure SQL que suporte consultas cruzadas de banco de dados usando um modelo Azure Resource Manager (ARM). Como você deve completar o modelo ARM?
```
"resources": [
  ...
  "type": [Box 1],
  "name": "[parameters('targetName')]",
  "location": "[parameters('location')]",
  "sku": { "name": "[parameters('skuName')]" ... },
  "dependsOn": [ [Box 2] ],
  "properties": {
    "administratorLogin": "[parameters('administratorLogin')]",
    "administratorLoginPassword": "[parameters('administratorLoginPassword')]",
    "subnetId": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworkName'), parameters('subnetName'))]",
    "storageSizeInGB": "[parameters('storageSizeInGB')]",
    "vCores": "[parameters('vCores')]",
    "licenseType": "[parameters('licenseType')]"
  ...
]
```

**Alternativas:**

**Caixa 1:**
- **A.** Microsoft.Sql/servidores
- **B.** Microsoft.Sql /servers/banco de dados
- **C.** Microsoft.Sql/managedInstances

**Caixa 2:**
- **A.** "["Parâmetros(" targetName ')]"
- **B.** "["Parâmetros(" virtualNetworkName ')]"
- **C.** "[variables(" networkSecurityGroupName ')]"

**✅ Resposta:** Caixa 1: Microsoft.Sql / managedInstances; Caixa 2: "[parâmetros(' virtualNetworkName ')]"

**Explicação:** As consultas de banco de dados cruzado nativo (nome de três partes) são suportadas em SQL Managed Instance, não Azure SQL Database. O modelo é subnetId, vCores, storageSizeInGB e licenseType As propriedades de instância gerenciada também são propriedades de instância gerenciada. managed instance é implantado em a VNet subnet, por isso depende da rede virtual. O NSG é uma dependência da sub-rede, não da instância, e targetName É a própria instância.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-only-authentication-create-server?view=azuresql) [Doc 2](https://learn.microsoft.com/ko-kr/azure/azure-sql/managed-instance/create-template-quickstart?view=azuresql-mi)

---

### Q209. Declarações de modelo ARM: Capacidade padrão 10

**Questão:**
Você tem o seguinte modelo Azure Resource Manager.
```
"variable": { "serverName": "azsqldbserver0001" },
"resources": [
  {
    "name": "[variables('serverName')]",
    "type": "Microsoft.Sql/servers",
    "apiVersion": "2019-06-01-preview",
    "location": "[parameters('location')]",
    "properties": {
      "administratorLogin": "[parameters('administratorLogin')]",
      "administratorLoginPassword": "[parameters('administratorLoginPassword')]",
      "version": "12.0"
    },
    "resources": [
      {
        "name": "[concat(variables('serverName'),'/',parameters('databaseName'))]",
        "type": "Microsoft.Sql/servers/databases",
        "apiVersion": "2020-08-01-preview",
        "location": "[parameters('location')]",
        "kind": "v12.0",
        "sku": { "name": "Standard", "tier": "Standard", "capacity": 10 },
        "dependsOn": [ "[concat('Microsoft.Sql/servers/', variables('serverName'))]" ],
        "properties": { },
        "resources": [ ]
      }
    ]
  }
],
```
Para cada uma das seguintes afirmações, selecione Sim se a afirmação for verdadeira. Caso contrário, selecione Não.

**Alternativas:**
1. O modelo implanta um serverless Azure SQL database.
2. O modelo implanta um banco de dados para um Azure SQL Database managed instance.
3. O nível de preços da implantação do banco de dados é baseado em DTUs.

**✅ Resposta:** 1: Não; 2: Não; 3: Sim

**Explicação:** A SKU Standard/Standard com capacity 10 corresponde a S0 (10 DTUs) no modelo de compra DTU. Serverless só existe no modelo vCore (SKUs GP_S_Gen5 ou Hyperscale serverless). Os tipos de recurso são Microsoft.Sql/servers e servers/databases (logical server e single database). Microsoft.Sql/managedInstances é outro tipo de recurso.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/sql-database/sql-database-single-database-get-started-template)

---

### Q210. az sql db criar parâmetros necessários

**Questão:**
Você tem uma assinatura do Azure. Você precisa implantar um novo Azure SQL database usando a Interface de Linha de Comando do Azure (CLI). Quais três parâmetros são necessários?

**Alternativas:**
- **A.** --name, --edition e --capacity
- **B.** --name, --tier e --min-capacity
- **C.** --name, --resource-group e --server
- **D.** --name, --licence-type e --capacity

**✅ Resposta:** C

**Explicação:** Os parâmetros necessários do `az sql db create` são `--name/-n`, `--resource-group/-g` e `--server/-s`. Edição, capacidade, nível, capacidade mínima e tipo de licença são opcionais; aplicam-se padrões (ou eles são necessários apenas para configurações vCore / serverless).

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/cli/azure/sql/db?view=azure-cli-latest)

---

### Q211. Implantar banco de dados do BACPAC com CLI

**Questão:**
Você tem uma assinatura do Azure. Você precisa implantar um Azure SQL database usando um arquivo BACPAC. Qual comando você deve executar?

**Alternativas:**
- **A.** sql db create
- **B.** sql db op
- **C.** sql db import
- **D.** az sql db cópia

**✅ Resposta:** C

**Explicação:** `az sql db import` "importa um bacpac em um novo banco de dados, ou um banco de dados vazio existente" (de Azure Storage via --storage-uri). `create` faz um DB vazio, `op` gerencia operações em execução e `copy` copia um banco de dados existente, não um BACPAC.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/cli/azure/sql/db?view=azure-cli-latest)

---

### Q212. Servidor Bicep + Declarações Database1

**Questão:**
Você tem uma assinatura do Azure. Você implanta recursos usando o seguinte modelo Bicep.
```
resource sqlServer 'Microsoft.Sql/server@2022-05-01-preview' = {
     name: 'Server1'
     location: 'Sweden central'
     properties: {
          administrators: {
               login: sqladmins
               principalType: 'Group'
               administratorType: 'ActiveDirectory'
          }
     }
}

resource sqlDB 'Microsoft.Sql/servers/databases@2022-05-01-preview' = {
     parent: 'Server1'
     name: 'Database1'
     location: 'Sweden central'
     sku: {
          name: 'Standard'
          tier: 'Standard'
     }
}
```
Para cada uma das seguintes afirmações, selecione Sim se a afirmação for verdadeira. Caso contrário, selecione Não.

**Alternativas:**
1. Server1 vai apoiar autenticação mista.
2. Database1 escalará dinamicamente com base na carga.
3. Server1 será criado como um Azure SQL managed instance.

**✅ Resposta:** 1: Sim; 2: Não; 3: Não

**Explicação:** (1) Um administrador do grupo Entra é definido, mas `azureADOnlyAuthentication` não é especificado. Seu padrão é falso, então SQL authentication (misturado). (2) sku Standard/Standard é o DTU- modelo provisioned Nível padrão com computação fixa. Somente vCore serverless (3) O tipo de recurso é o servidor lógico (Microsoft.Sql/servidores), não Microsoft.Sql/managedInstances. Nota de intenção de exame: em uma implantação real, deixar a autenticação apenas intra-somente falsa também requer administratorLogin/password, que este modelo omite. Isso não altera as respostas pretendidas.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-only-authentication-create-server?view=azuresql)

---

### Q213. New-AzSqlDatabase: ComputeGeneration e ComputeModel

**Questão:**
Você tem uma assinatura do Azure. Você precisa implantar um Azure SQL database. A solução deve atender aos seguintes requisitos:
- Apoiar pelo menos 36 núcleos.
- Minimizar custos.

Como você deve completar o script PowerShell?
```
$SqlDatabaseParams = @{
  ResourceGroupName = "RG1"
  ServerName = "Server01"
  DatabaseName = "DB1"
  Edition = "GeneralPurpose"
  Vcore = 2
  ComputeGeneration = [Box1]
  ComputeModel = [Box2]
}
New-AzSqlDatabase @SqlDatabaseParams
```

**Alternativas:**

**Caixa 1 (ComputeGeneration):**
- **A.** "4"
- **B.** "5"
- **C.** "Gen4"
- **D.** "Gen5"

**Caixa 2 (ComputeModel):**
- **A.** Livre
- **B.** Provisioned
- **C.** Serverless
- **D.** Padrão

**✅ Resposta:** Caixa 1: "Gen5"; Caixa 2: Serverless

**Explicação:** ComputeGeneration leva nomes de família ("Gen4"/"Gen5"), não "4"/"5". Gen4 está aposentado e encapado em 24 vCores, então Gen5 (série padrão) é necessário para 36 + núcleos. ComputeModel aceita apenas Provisioned : Serverless. Serverless General Purpose Gen5 escalas até 80 vCores e contas por segundo para computação usada (com auto-pause), assim que minimiza o custo para a exigência. O cmdlet é próprio serverless exemplo é `-Edition GeneralPurpose -Vcore 2 -ComputeGeneration "Gen5" -ComputeModel Serverless`. Livre e Padrão não são ComputeModel valores.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/powershell/module/az.sql/new-azsqldatabase)

---

### Q214. Mudar SQL MI para Business Critical

**Questão:**
Você tem uma assinatura do Azure que contém um Azure SQL managed instance chamado SQLMI1. Você precisa configurar o SQLMI1 para usar o nível de serviço Business Critical. Qual cmdlet PowerShell você deve executar?

**Alternativas:**
- **A.** Update-SqlVM
- **B.** Set-AzSqlServerConfigurationOption
- **C.** Set-AzSqlDatabase
- **D.** Set-AzSqlInstance

**✅ Resposta:** D

**Explicação:** Set-AzSqlInstance modificando um existente managed instance, incluindo `-Edition BusinessCritical` (nível de serviço), vCores e armazenamento. Set-AzSqlDatabase muda um único banco de dados em um servidor lógico. Set-AzSqlServerConfigurationOption conjuntos MI opções de configuração do servidor (e.g., allowPolybaseExport), não o nível. Update-SqlVM é para SQL Server on Azure VMs.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/powershell/module/az.sql/set-azsqlinstance)

---

### Q215. ADatum: primeiro passo para DB3 elastic jobs

**Estudo de caso:** ADatum v1 (financeiro)

**Questão:**
(Case study — ADatum Corporação v1. DB3 é migrado de SQL Server 2019 SVR3 E- Azure SQL Database. Requisito de gestão: "Em Azure SQL Database, automatizar a gestão de DB3 usando elastic jobs que tem database-scoped credentials." Requisitos de negócio: minimizar custos e esforço administrativo.)
Você precisa recomendar um processo para automatizar a gestão de DB3. A solução deve atender aos requisitos de gerenciamento. Qual deve ser o primeiro passo do processo?

**Alternativas:**
- **A.** Configurar Microsoft Entra authentication para o servidor lógico que hospeda DB3.
- **B.** Configure um private endpoint para conectividade com o DB3.
- **C.** Criar database-scoped credentials em DB3.
- **D.** Crie um banco de dados que tenha database-scoped credentials.

**✅ Resposta:** D

**Explicação:** An elastic job agent só pode ser criado em um existente Azure SQL database, que se torna o *banco de dados job* (uma nova banco de dados vazia, S1 ou superior recomendado). Com database-scoped credential Os médicos da Microsoft dizem para "criar a database-scoped credentials na *banco de dados job*". O alvo (DB3) only needs a login/user (jobuser) cujo nome e senha correspondem à credencial. Então o processo começa criando a job banco de dados que detém as credenciais, então o agente, target group e jobsC coloca as credenciais no alvo, o que só é necessário quando o alvo é ele próprio o job A é o banco de dados UMIMétodo de entrada, não o necessário database-scoped credentialsB é uma rede opcional.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-overview?view=azuresql)

---

### Q216. Elastic jobs Auth com Acesso Condicional

**Questão:**
Você tem uma assinatura do Azure que contém um banco de dados do Azure SQL Database chamado DB1. Você precisa hospedar o elastic jobs usando DB1. DB1 também será configurado como um alvo job. A solução deve apoiar o uso de políticas de acesso condicional baseadas em localização. O que o elastic jobs deve usar para acessar o DB1?

**Alternativas:**
- **A.** system-assigned managed identity
- **B.** Credenciais de login Azure SQL
- **C.** database-scoped credentials
- **D.** user-assigned managed identity

**✅ Resposta:** D

**Explicação:** Elastic jobs suporte exatamente dois métodos de autenticação-alvo. Um é Microsoft Entra authentication com a user-assigned managed identity, que a Microsoft recomenda. O outro é database-scoped credentials (SQL authentication). A system-assigned managed identity Não é explicitamente suportado (A ✘), e B não é um elástico-job O acesso condicional, incluindo políticas baseadas em localização, aplica-se apenas a Microsoft Entra authentication. SQLcredenciais de autenticação (C) ignoram totalmente o Entra, para que não possam ser governadas por CA. Isso deixa o UMI. Caveata: "Acesso condicional para identidades de carga de trabalho" da Entra não cobre atualmente managed identities, então a CA Na verdade, a política não avaliaria o UMI Mesmo sob a leitura "deve continuar trabalhando onde CA As políticas estão em vigor", o UMI funciona e é a opção recomendada.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Baixa · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-overview?view=azuresql) [Doc 2](https://learn.microsoft.com/entra/identity/conditional-access/workload-identity)

---

### Q217. Credenciais mínimas com escopo de banco de dados para elastic jobs

**Questão:**
Você tem uma assinatura do Azure que contém os recursos mostrados na tabela a seguir.

| Nome | Tipo | Descrição |
|---|---|---|
| SQLSVR1 | Logical SQL server | Hospeda um Azure SQL database |
| SQLDB11 | Azure SQL database | É hospedado por SQLSVR1 |
| SQLSVR2 | Logical SQL server | hospeda dois Azure SQL databases |
| SQLDB21 | Azure SQL database | É hospedado por SQLSVR2 |
| SQLDB22 | Azure SQL database | É hospedado por SQLSVR2 |

Você planeja usar SQLDB11 como um elastic job database para executar jobs em SQLDB11 e SQLDB22. Qual é o número mínimo de credenciais com escopo de banco de dados necessárias para o elastic jobs?

**Alternativas:**
- **A.** 1
- **B.** 2
- **C.** 3
- **D.** 4

**✅ Resposta:** A

**Explicação:** Todos elásticos-job Credenciais são criadas em job banco de dados (SQLDB11). Os alvos são dois bancos de dados individuais, para que possam ir em um target group como um único banco de dados, que precisa apenas do job-credencial de execução (jobuser). Criar um jobuser correspondente login/user com a mesma senha em SQLDB11 e SQLDB22Uma segunda credencial de "mestre" (refresh) é necessária apenas quando uma target group contém a *servidor* : *elastic pool*, porque o agente deve se conectar ao mestre para enumerar bancos de dados. Segmentar apenas dois bancos de dados nomeados não precisa disso. O agente atinge seu próprio job banco de dados com auth certificado interno, de modo que não precisa de credencial também.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-overview?view=azuresql)

---

<a name="d5--plan-and-configure-a-high-availability-and-disaster-recovery-hadr-environment-2025"></a>

## D5 — Planejar e configurar um ambiente de alta disponibilidade e recuperação de desastres (HA/DR) (20–25%)

*13 perguntas*

### Q218. Declarações BACKUP LOG NORECOVERY COPY_ONLY

**Questão:**
Você tem uma instância SQL Server on Azure Virtual Machines chamada VM1 que hospeda um banco de dados chamado DB1. Você executa a seguinte consulta.
```sql
BACKUP LOG DB1 TO DISK = '\File1\SQLBackups\DB1.trn'
WITH NORECOVERY, COPY_ONLY, CONTINUE_AFTER_ERROR;
GO
```
Para cada uma das seguintes afirmações, selecione Sim se a afirmação for verdadeira. Caso contrário, selecione Não.

**Alternativas:**
1. O arquivo de log será truncado.
2. O DB1 será colocado em um estado offline.
3. Você está executando um tail-log backup.

**✅ Resposta:** 1. Não; 2. Não; 3. Sim

**Explicação:** Por o tail-log backup doc, a BACKUP LOG ... WITH NORECOVERY "leva o banco de dados para o estado de restauração" e "o log é truncado a menos que o NO_TRUNCATE opção ou COPY_ONLY opção também é especificada”, assim COPY_ONLY O banco de dados vai para RESTORING, que é um estado diferente de OFFLINE. NORECOVERY juntamente com CONTINUE_AFTER_ERROR (que o médico diz que é apenas para fazer backup da cauda de um banco de dados danificado) é um tail-log backup.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/tail-log-backups-sql-server)

---

### Q219. Retenção de política LTR semanal backups

**Questão:** (PARCIAL. O cabeçalho e a imagem da política LTR não foram registrados e a versão completa não foi encontrada online. Reconstrução: um banco Azure SQL tem uma política de long-term retention mostrada em uma imagem, com períodos de retenção de 6 semanas, 12 meses e 10 anos. O cenário provavelmente corresponde ao exemplo da Microsoft com W = 6 semanas, M = 12 meses, Y = 10 anos e WeekOfYear diferente das semanas 1 e 2.)
O primeiro backup semanal ocorreu em 4 de janeiro de 2020. As datas para o primeiro 10 backups semanal são: 4 de janeiro, 11 de janeiro, 18 de janeiro, 25 de janeiro, 1 de fevereiro, 8 de fevereiro, 15 de fevereiro, 22 de fevereiro, 29 de fevereiro e 7 de março de 2020.
Use os menus suspensos para selecionar a opção de resposta que completa cada declaração com base nas informações apresentadas no gráfico.

**Alternativas:**

**Declaração 1: O backup salvo para o long-term retention em 4 de janeiro de 2020, será mantido para:**
- **A.** 6 semanas
- **B.** 12 meses
- **C.** 10 anos

**Declaração 2: O backup salvo no long-term retention em 11 de janeiro de 2020, será mantido para:**
- **A.** 6 semanas
- **B.** 12 meses
- **C.** 10 anos

**Resposta ✅ (melhor reconstrução):** Declaração 1: 12 meses; Declaração 2: 6 semanas

**Explicação:** Regras de LTR. Com W definido, cada full backup semanal é mantido por W. Com M definido, o **primeiro full backup de cada mês** é mantido por M. Com Y definido, o backup da semana WeekOfYear é mantido por Y. Prevalece o período mais longo aplicável (exemplo da documentação: W = 6, M = 12, Y = 10…). O backup de 4 de janeiro é o primeiro do mês e fica retido por 12 meses. O de 11 de janeiro é um backup semanal comum e fica retido por 6 semanas. Se WeekOfYear na imagem for 1, o backup de 4 de janeiro fica retido por 10 anos. Se for 2, o de 11 de janeiro fica retido por 10 anos.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Baixa · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/long-term-retention-overview)

---

### Q220. Managed instance BACKUP TO URL COPY_ONLY

**Questão:**
Você tem um Azure SQL Database managed instance chamado sqldbmi1 que contém um banco de dados chamado Vendas. Você precisa iniciar um backup de Vendas. Como você deve completar a declaração Transact-SQL?
```
BACKUP DATABASE Sales
[Box 1]
WITH STATS = 5,
[Box 2]
```

**Alternativas:**

**Caixa 1:**
- **A.** TO DISK = '\BackupSystem\BackupDisk1\Sales.bak'
- **B.** TO DISK = 'X:\BAK\Sales.bak'
- **C.** TO 'Sales_Backup'
- **D.** TO URL = 'https://storage1.blob.core.windows.net/blob1/Sales.bak'

**Caixa 2:**
- **A.** COPY_ONLY
- **B.** Criptografia
- **C.** FILE_SNAPSHOT
- **D.** NO_TRUNCATE

**✅ Resposta:** Caixa 1: TO URL = 'https://storage1.blob.core.windows.net/blob1/Sales.bak'; Caixa 2: COPY_ONLY

**Explicação:** SQL Managed Instance não tem acesso ao local ou UNC discos ou backup dispositivos, então o usuário backups só pode ir para Azure Blob storage com TO URL. Automatizado backups O próprio backup chain, então um usuário iniciado backup deve ser COPY_ONLY. FILE_SNAPSHOT precisa de arquivos de banco de dados que são armazenados no Azure Blob, ENCRYPTION precisa de um certificado ou chave e não é necessário aqui, e NO_TRUNCATE Aplica-se apenas ao log backups.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/automated-backups-overview)

---

### Q221. PowerShell STR 21 dias e declarações LTR

**Questão:**
Você tem um Azure SQL database. Você executa o seguinte script PowerShell.
```powershell
$serverName = "SERVER1"
$resourceGroup = "RG1"
$dbName = "DB1"
Connect-AzAccount
$server = Get-AzSqlServer -ServerName $serverName -ResourceGroupName $resourceGroup
Set-AzSqlDatabaseBackupShortTermRetentionPolicy -ResourceGroupName $resourceGroup -ServerName $server `
    -DatabaseName $dbName -RetentionDays 21
Set-AzSqlDatabaseBackupLongTermRetentionPolicy -ServerName $serverName -DatabaseName $dbName `
    -ResourceGroupName $resourceGroup -WeeklyRetention P52W -YearlyRetention P5Y -WeekOfYear 52
```
Para cada declaração, selecione Sim se for verdade; caso contrário, selecione Não.

**Alternativas:**
1. DB1 pode ser restaurado a um ponto específico no tempo há 30 dias.
2. O DB1 pode ser restaurado a partir de um backup semanal realizado há seis meses.
3. O DB1 pode ser restaurado a partir de um backup anual realizado há seis anos.

**✅ Resposta:** 1. Não; 2. Sim; 3. Não

**Explicação:** A retenção de curto prazo (PITR) é de 21 dias, então um ponto há 30 dias está fora da janela. Semanal LTR backups são mantidos 52 semanas, então um backup semanal de 6 meses atrás ainda existe. Anual backups (semana 52) são mantidos 5 anos, então um de 6 anos atrás expirou.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/long-term-retention-overview)

---

### Q222. Sequência de restauração completa, diferencial, registros

**Questão:**
Você tem SQL Server em um Azure virtual machine que contém um banco de dados chamado DB1. DB1 é 30 TB e tem um 1-GB taxa diária de mudança. Você faz backup do banco de dados usando um Microsoft SQL Server Agent job que corre Transact-SQL Você executa um comando semanal full backup Domingo, diariamente differential backups at 01:00, and transaction log backups a cada cinco minutos. O banco de dados falha na quarta-feira às 10:00. Que três backups should you restore in sequence?

**Alternativas:**
- **A.** Segunda-feira, Terça-feira e Quarta-feira differential backups
- **B.** Quarta, Terça e Segunda-feira log backups
- **C.** full backup
- **D.** Segunda, Terça e Quarta-feira log backups
- **E.** Quarta-feira, Terça-feira e Segunda-feira differential backups
- **F.** quarta-feira log backups
- **G.** Quarta-feira differential backup

**✅ Resposta:** 1. full backup → 2. quarta-feira differential backup → 3. quarta-feira log backups

**Explicação:** Differential backups são cumulativos desde o último full backup, portanto, apenas o mais recente (quarta-feira 01:00) é necessário. Depois disso, restaure o log backups tomado após 01:00 quarta-feira, a fim, até o fracasso. Restaurar os diferenciais ou registros de segunda e terça-feira é desnecessário e a ordem inversa é inválida.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/cs-cz/sql/relational-databases/backup-restore/restore-a-differential-database-backup-sql-server?view=sql-server-2016)

---

### Q223. Teste DR failover para Hyper-V app

**Questão:**
Você tem uma aplicação multi-camada on-premises chamada App1 que inclui um nível web, um nível de aplicativo e um nível Microsoft SQL Server. Todos os níveis são executados em máquinas virtuais Hyper-V. Seu novo plano de recuperação de desastres exige que todas as aplicações críticas para negócios possam ser recuperadas para o Azure. Você precisa recomendar uma solução para falhar sobre o nível de banco de dados do App1 para o Azure. A solução deve fornecer a capacidade de testar o failover para o ambiente atual do Azure.

**Alternativas:**
- **A.** Azure Backup
- **B.** Proteção de Informações do Azure
- **C.** Windows Server Failover Cluster
- **D.** Azure Site Recovery

**✅ Resposta:** D

**Explicação:** Azure Site Recovery replica Hyper-V VMs para o Azure e suporta *teste failover* em uma rede isolada sem impacto na replicação da produção ou cargas de trabalho. Azure Backup é para backup e restaurar, não orquestrado failover. AIP é classificação de dados. Um WSFC por si só não dá uma broca DR não disruptiva ao Azure.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-test-failover-to-azure)

---

### Q224. Litware ManufacturingSQLDb1 testemunha e listener

**Estudo de caso:** Litware

**Questão:**
Você precisa recomendar uma configuração para ManufacturingSQLDb1 após a migração para o Azure. A solução deve atender aos requisitos de negócios. O que você deve incluir na recomendação?
(Litware: ManufacturingSQLDb1 está em um on-premises Always On AG em SERVER2 / SERVER3 e será migrado para Azure VMs. Exigência de negócios: "Conheça um SLA de disponibilidade 99.99% para todas as implantações do Azure"; minimizar os custos.)

**Alternativas:**

**Modelo do Quórum:**
- **A.** Testemunha de nuvem
- **B.** Testemunha de disco
- **C.** Arquivo compartilhar testemunha

**Recursos do Azure para o availability group listener:**
- **A.** Gateway de Aplicação Azure
- **B.** Azure Basic Load Balancer

**✅ Resposta:** Modelo do Quórum: testemunha da nuvem; AG listener: Azure Basic Load Balancer (o único válido) listener opção oferecida)

**Explicação:** Para a failover cluster em Azure VMs cujos nós podem chegar à internet, a Microsoft recomenda uma Testemunha de Nuvem (Azure Storage). Não precisa de nenhum extra VMAo contrário de uma testemunha de compartilhamento de arquivos, e discos compartilhados não são necessários. VNN AG listener E-mail Azure VMs precisa de uma Azure Load Balancer. O Gateway de Aplicação é um L7 HTTP proxy e não pode frente a SQL listener. Nota sobre os documentos atuais: o Básico Load Balancer SKU foi aposentado em 30 de setembro de 2025, e a Microsoft agora recomenda um padrão Load Balancer (ou a DNN listener ou multi-subnet AG). Se um "Standard Load Balancer" opção aparece, escolha-o. própria explicação da demo cita o padrão LB 99.99% SLA.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener)

---

### Q225. Zona-redundante automatic failover a um custo mínimo

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado DB1. Você precisa garantir que o DB1 suportará automatic failover sem perda de dados se um datacenter falhar. A solução deve minimizar os custos. Qual opção de implantação e nível de preços você deve configurar?

**Alternativas:**
- **A.** Azure SQL Database Premium
- **B.** Azure SQL Database serverless
- **C.** Azure SQL Database managed instance Business Critical
- **D.** Padrão Azure SQL Database

**✅ Resposta:** A

**Explicação:** Sobrevivendo a um Data Center (availability zone) falha com automatic failover e zero perda de dados precisa de redundância de zona. No DTU Apenas o modelo Premium suporta redundância de zona, e Padrão e Básico não. Redundância de zona em Premium e Business Critical não tem custo extra porque usa as réplicas existentes. MI Business Critical é mais caro. atual-docs ressalva: General Purpose (incluindo serverless) agora também suporta redundância de zona, o que torna B discutível hoje. A chave do exame, escrita quando serverless e General Purpose redundância de zona não estavam disponíveis, é A.

**Dumps:** ✔ concordam
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/database/high-availability-sla-local-zone-redundancy)

---

### Q226. Failover sem alterações connection string

**Questão:**
Você precisa recomendar uma estratégia de disponibilidade para um Azure SQL database. A estratégia deve atender aos seguintes requisitos:
- Failovers de suporte que não exigem que os aplicativos do cliente alterem suas cadeias de conexão.
- Replicar o banco de dados para uma região secundária do Azure.
- Suporte failover para a região secundária.
O que você deve incluir na recomendação?

**Alternativas:**
- **A.** failover groups
- **B.** transactional replication
- **C.** Availability Zones
- **D.** geo-replication

**✅ Resposta:** A

**Explicação:** Failover groups replicar para outra região e expor read-write e read-only listener endpoints cujo DNS segue o primário, então as cordas de conexão não mudam após failover. Active geo-replication sozinho precisa de aplicativos para apontar para o novo nome do servidor. Availability Zones ficar dentro de uma região. Transactional replication não tem automatic failover ou redirecionamento de endpoint.

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/azure/azure-sql/database/auto-failover-group-sql-db)

---

### Q227. Log completo devido à correção AVAILABILITY_REPLICA

**Questão:**
Você tem um banco de dados chamado db1. O log para db1 contém a seguinte entrada.
```
Date 10/5/2021 10:57:08 AM
Log SQL Server (Current - 10/5/2021 11:26:00 AM)
Sources spid1595
Message
The transaction log for database 'db1' is full due to 'AVAILABILITY_REPLICA'
```
Você precisa garantir que o db1 possa processar transações. Quais três ações você deve executar em sequência?

**Alternativas:**
- **A.** Faça backup do arquivo de log da transação.
- **B.** Remover db1 do availability group.
- **C.** Encolher o arquivo de registro da transação.
- **D.** Shrink db1.
- **E.** Adicione db1 de volta ao availability group.

**✅ Resposta:** 1. Remova db1 do availability group → 2. Encolher o arquivo de log da transação → 3. Adicione db1 de volta ao availability group

**Explicação:** O erro 9002 com AVAILABILITY_REPLICA significa que os registros de log não foram endurecidos ou refeitos em um secundário, então o primário não pode truncar o log. Um log backup não libera o espaço enquanto o secundário fica atrasado. A solução da Microsoft é remover o banco de dados do AG, que imediatamente permite truncamento, depois encolher o arquivo de log e adicionar novamente o banco de dados (re-seeding o secundário).

**Dumps:** ✔ concordam
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/en-us/troubleshoot/sql/availability-groups/error-9002-transaction-log-large)

---

### Q228. Resultados da política automática MI failover group

**Questão:**
Você está planejando a recuperação de desastres para o failover group de um Azure SQL Database managed instance. O SLA da sua empresa exige que o banco de dados no failover group fique disponível o mais rápido possível se ocorrer uma grande interrupção. Você define a política de leitura / gravação failover como automática. Quais são os dois resultados da configuração? Cada resposta correta apresenta uma solução completa.

**Alternativas:**
- **A.** No caso de um datacenter ou falha regional do Azure, Os bancos de dados falharão automaticamente.
- **B.** No caso de uma interrupção, Os bancos de dados na instância primária falharão imediatamente.
- **C.** No caso de uma interrupção, você pode falhar seletivamente sobre bancos de dados individuais.
- **D.** No caso de uma interrupção, você pode definir um período de graça diferente para falhar em cada banco de dados.
- **E.** Em caso de interrupção, o atraso mínimo para que Os bancos de dados falhem na instância primária será de uma hora.

**✅ Resposta:** A, E

**Explicação:** Com a política automática de leitura/escrita failover (chamada "Microsoft managed" no portal atual), o Azure inicia um geo-failover após uma interrupção, uma vez que o período de carência (GracePeriodWithDataLossHours) passa. O período mínimo de graça é de 1 hora, o que também é o padrão, então o failover não é imediato. Uma instância gerenciada failover group falha em todas Os bancos de dados de usuários juntos, com um período de carência para todo o grupo, portanto C e D são falsos.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/powershell/module/az.sql/new-azsqldatabaseinstancefailovergroup) [Doc 2](https://learn.microsoft.com/en-us/training/modules/explore-iaas-paas-platform-tools-for-high-availability-disaster-recovery/6-explore-auto-failover-groups)

---

### Q229. Prepare AG secundário com RESTORE NORECOVERY

**Questão:**
Você tem dois Azure virtual machines nomeado VM1 e VM2 essa corrida Windows Server 2019. VM1 e VM2 cada host um padrão Microsoft SQL Server Exemplo de 2019. VM1 contém um banco de dados chamado DB1 that is backed up to a file named D:\DB1.bak. Você planeja implantar um Always On availability group com VM1 Abrigando a réplica principal de DB1 e VM2 Uma réplica secundária de DB1. Você precisa preparar o banco de dados secundário em VM2 para a availability group. Como você deve completar o Transact-SQL declaração?
```
[Box 1] DATABASE MyDB1
  FROM DISK = 'D:\DB1.bak'
  WITH [Box 2]
GO
```

**Alternativas:**

**Caixa 1:**
- **A.** BACKUP
- **B.** CREATE
- **C.** RESTORE

**Caixa 2:**
- **A.** NORECOVERY
- **B.** RECOVERY
- **C.** STANDBY

**✅ Resposta:** Caixa 1: RESTORE; Caixa 2: NORECOVERY

**Explicação:** Um banco de dados secundário é preparado restaurando o full backup (e mais tarde log backups) na instância secundária WITH NORECOVERYO que o deixa no RESTORING para que ele possa ser unido ao AG. RECOVERY traz o banco de dados on-line e quebra a cadeia de log necessário para se juntar. STANDBY (read-only é para log shipping, não AG Joins.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Alta · [Doc 1](https://learn.microsoft.com/En-Us/Sql/database-engine/availability-groups/windows/manually-prepare-a-secondary-database-for-an-availability-group-sql-server?view=aps-pdw-2016)

---

### Q230. Usuário backups mais réplicas multi-região

**Questão:**
Você planeja migrar bancos de dados on-premises Microsoft SQL Server para o Azure. Você precisa identificar quais opções de implantação e resiliência atendem aos seguintes requisitos:
- Suporte iniciado pelo usuário backups.
- Suporte a várias instâncias replicadas automaticamente nas regiões do Azure.
- Minimizar o esforço administrativo para implementar e manter a continuidade dos negócios.
O que você deve identificar?

**Alternativas:**

**Opção de implantação:**
- **A.** Azure SQL Managed Instance
- **B.** SQL Server on Azure Virtual Machines
- **C.** Banco de dados único An Azure SQL Database

**Opção de resiliência:**
- **A.** Auto-failover group
- **B.** Active geo-replication
- **C.** implantação zona-redundante

**✅ Resposta:** Opção de implantação: Azure SQL Managed Instance; Opção de resiliência: Auto-failover group

**Explicação:** SQL Managed Instance suportes iniciados pelo usuário (COPY_ONLY E- URL) backups, que um único Azure SQL Database não, e é PaaS, então a continuidade dos negócios leva muito menos esforço do que em SQL Server E-mail VMs Onde você constrói e corre AGs e você mesmo). Para MI, replicação automática entre regiões é fornecida por auto-failover groups. Active geo-replication não está disponível para MI, e redundância zona permanece dentro de uma região.

**Dumps:** ⚠️ Alguns dumps apresentam uma resposta incorreta
**Confiança:** Média · [Doc 1](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/automated-backups-overview) [Doc 2](https://learn.microsoft.com/ar-sa/azure/azure-sql/managed-instance/failover-group-sql-mi)

---

<a name="case-studies"></a>

## Estudos de caso

Perguntas marcadas **Estudo de caso: X** referem-se ao texto abaixo.

---

### Litware, Inc. (energia renovável; SERVER1, SalesSQLDb1, ResearchDB1 / ResearchApp1, ManufacturingSQLDb1)

Questões: Q8; Q68; Q78; Q154; Q155; Q205; Q224

Observação: na fonte, a sentença de licenciamento fica sob o título "Problemas atuais". É mantido lá como no original.

Visão geral
Litware, Inc. é uma empresa de energia renovável que tem um escritório principal em Boston. O escritório principal hospeda um departamento de vendas e o centro de dados primário para a empresa.

Locais Físicos
A Litware tem um escritório de fabricação e um escritório de pesquisa em locais separados perto de Boston. Cada escritório tem seu próprio datacenter e conexão à Internet.

Ambiente existente

Ambiente de rede
Os data centers de fabricação e pesquisa se conectam ao datacenter primário usando um VPN.
O datacenter primário tem uma conexão ExpressRoute que usa tanto o peering da Microsoft quanto o peering privado. O peering privado se conecta a um Azure virtual network chamado HubVNet.

Ambiente de Identidade
O Litware tem uma implantação híbrida Microsoft Entra ID que usa um domínio chamado litwareinc.com. Todas as assinaturas do Azure estão associadas ao inquilino litwareinc.com Microsoft Entra.

Ambiente de banco de dados
O departamento de vendas tem a seguinte carga de trabalho de banco de dados:
- Um servidor on-premises chamado SERVER1 hospeda uma instância do Microsoft SQL Server 2012 e dois bancos de dados 1-TB.
- Um servidor lógico chamado SalesSrv01A contém um Azure SQL database geo-replicado chamado SalesSQLDb1. SalesSQLDb1 está em um elastic pool chamado SalesSQLDb1Pool. SalesSQLDb1 usa regras de firewall de banco de dados e usuários de banco de dados contidos.
- Um aplicativo chamado SalesSQLDb1App1 usa SalesSQLDb1.
O escritório de fabricação contém dois servidores on-premises SQL Server 2016 chamados SERVER2 e SERVER3. Os servidores são nós no mesmo Always On availability group. O availability group contém um banco de dados chamado ManufacturingSQLDb1.
Os administradores de banco de dados têm dois Azure virtual machines no HubVnet chamados VM1 e VM2 que executam o Windows Server 2019 e são usados para gerenciar todos os bancos de dados do Azure.

Problemas atuais
Litware é um cliente de Licenciamento por Volume da Microsoft que possui Mobilidade de Licença através do Software Assurance.
SalesSQLDb1 experimenta problemas de desempenho que são provavelmente devido a estatísticas desatualizadas e consultas frequentes de bloqueio.

Requisitos

Mudanças Planejadas
A Litware planeja implementar as seguintes mudanças:
- Implemente 30 novas bancos de dados no Azure, que serão utilizadas por aplicações de fabrico sensíveis ao tempo que têm padrões de utilização variáveis. Cada banco de dados será aproximadamente 20 GB.
- Criar um novo Azure SQL database chamado ResearchDB1 em um servidor lógico chamado ResearchSrv01. ResearchDB1 conterá dados de informações pessoalmente identificáveis (PII).
- Desenvolva um aplicativo chamado ResearchApp1 que será usado pelo departamento de pesquisa para preencher e acessar o ResearchDB1.
- Migre o ManufacturingSQLDb1 para a plataforma Azure virtual machine.
- Migre os bancos de dados SERVER1 para a plataforma Azure SQL Database.

Requisitos técnicos
A Litware identifica os seguintes requisitos técnicos:
- As tarefas de manutenção devem ser automatizadas.
- As 30 novas bancos de dados devem ser dimensionadas automaticamente.
- O uso de uma infraestrutura on-premises deve ser minimizado.
- Os benefícios de uso híbrido do Azure devem ser aproveitados para implantações Azure SQL Database.
- As métricas SQL Server e Azure SQL Database relacionadas ao CPU e ao uso e limites de armazenamento devem ser analisadas usando a funcionalidade integrada do Azure.

Requisitos de Segurança e Conformidade
A Litware identifica os seguintes requisitos de segurança e conformidade:
- Chaves de criptografia da loja em Azure Key Vault.
- Retenha backups dos dados PII por dois meses.
- Criptografe os dados PII em repouso, em trânsito e em uso.
- Use o princípio do menor privilégio sempre que possível.
- Autenticar usuários de banco de dados usando credenciais Active Directory.
- Proteja instâncias Azure SQL Database usando regras de firewall de nível de banco de dados.
- Certifique-se de que todos os bancos de dados hospedados no Azure estejam acessíveis a partir do VM1 e VM2 através da rede de backbone do Azure.

Requisitos de Negócio
A Litware identifica os seguintes requisitos de negócios:
- Conheça um SLA de disponibilidade 99.99% para todas as implantações do Azure.
- Minimize o tempo de inatividade durante a migração dos bancos de dados SERVER1.
- Utilize os Benefícios de Utilização Híbrida do Azure ao migrar cargas de trabalho para o Azure.
- Uma vez que todos os requisitos sejam atendidos, minimize os custos sempre que possível.

---

### ADatum Corporation v1 (serviços financeiros; SVR1-3, DB1 / DB2 / DB3, Instance1)

Questões: Q7; Q44; Q77; Q91; Q108; Q143; Q176; Q215

Visão geral
ADatum Corporation é uma empresa de serviços financeiros que tem um escritório principal em Nova York.

Ambiente existente. Acordo de licenciamento
O ADatum tem um contrato de Licenciamento por Volume da Microsoft que inclui o Software Assurance.

Ambiente existente. Infraestrutura de rede
O ADatum possui um datacenter on-premises e uma assinatura do Azure chamada Sub1.
O Sub1 contém uma rede virtual chamada Network1 na região leste do US Azure.
O datacenter está conectado ao Network1 usando um Site-to-Site (S2S) VPN.

Ambiente existente. Ambiente de identidade
A rede on-premises contém uma floresta de Serviços de Domínio Active Directory (AD DS).
A floresta contém um único domínio chamado corp.adatum.com.
O domínio corp.adatum.com sincroniza com um inquilino Microsoft Entra chamado adatum.com.

Ambiente existente. Ambiente de banco de dados
O datacenter contém os servidores mostrados na tabela a seguir.

| Nome | Sistema operacional | Descrição |
|---|---|---|
| SVR1 | Windows Server 2016 | Tem Microsoft SQL Server 2016 Enterprise instalado. Tem um Always On availability group chamado AG1 que contém dois bancos de dados chamados DB1 e DB2. |
| SVR2 | Windows Server 2016 | Tem Microsoft SQL Server 2016 Enterprise instalado. Tem um Always On availability group chamado AG1 que contém dois bancos de dados chamados DB1 e DB2. |
| SVR3 | Windows Server 2019 | Tem o Microsoft SQL Server 2019 Enterprise instalado que contém um banco de dados chamado DB3. |

DB1 e DB2 são usados para cargas de trabalho transacionais e analíticas por um aplicativo chamado App1.
App1 roda em servidores híbridos Microsoft Entra unidos que executam Windows Server 2022. App1 usa a autenticação Kerberos.
O DB3 armazena dados de conformidade usados por dois aplicativos chamados App2 e App3.
O desempenho do DB3 é monitorado usando sessões Extended Events, com o alvo event_file definido para uma compartilhamento de arquivos em um disco local do SVR3.
A alocação de recursos para DB3 é gerenciada usando Resource Governor.

Requisitos. Alterações Planejadas
ADatum planeja implementar as seguintes mudanças:
- Implante um Azure SQL managed instance chamado Instance1 para Network1.
- Migrar DB1 e DB2 para Instance1.
- Migrar DB3 para Azure SQL Database.
- Após a migração de DB1 e DB2, entregue o desenvolvimento do banco de dados para desenvolvedores remotos que usam Microsoft Entra juntou dispositivos Windows 11.
- Após a migração do DB3, configure o banco de dados para fazer parte de um auto-failover group.

Requisitos de disponibilidade
O ADatum identifica os seguintes requisitos de disponibilidade pós-migração:
- Para DB1 e DB2, descarregue cargas de trabalho analíticas para uma réplica de banco de dados read-only na mesma região do Azure.
- Certifique-se de que, se ocorrer um desastre regional, DB1 e DB2 podem ser recuperados do backups.
- Após a migração, o App1 deve manter o acesso ao DB1 e ao DB2.
- Para DB3, gerenciar possíveis problemas de desempenho causados por mudanças na demanda de recursos por App2 e App3.
- Certifique-se de que o DB3 ainda estará acessível após um planned failover.
- Certifique-se de que o DB3 pode ser restaurado se o servidor lógico for excluído.
- Minimizar o tempo de inatividade durante a migração de DB1 e DB2.

Requisitos de segurança. Requisitos de segurança
ADatum identifica os seguintes requisitos de segurança para após a migração:
- Certifique-se de que apenas desenvolvedores designados que usam dispositivos Windows 11 associados ao Microsoft Entra possam acessar o DB1 e o DB2 remotamente.
- Certifique-se de que todas as alterações ao DB3, incluindo aquelas dentro de transações individuais, sejam auditadas e registradas.

Requisitos de Gestão. Requisitos de Gestão
ADatum identifica os seguintes requisitos de gestão pós-migração:
- Continue usando o Extended Events para monitorar o DB3.
- Em Azure SQL Database, automatizar a gestão de DB3 usando elastic jobs que tem database-scoped credentials.

Requisitos de negócio. Requisitos de negócios
ADatum identifica os seguintes requisitos de negócios:
- Minimizar custos sempre que possível, sem afetar outros requisitos.
- Minimizar o esforço administrativo.

---

### Contoso, Ltd. (dados financeiros; um Azure SQL database por cliente)

Questões: Q45; Q74; Q115; Q116; Q123

Visão geral

Visão geral geral
Contoso, Ltd. é uma empresa de dados financeiros que tem 100 funcionários. A empresa fornece dados financeiros aos clientes.

Locais Físicos
A Contoso tem um datacenter em Los Angeles e uma assinatura do Azure. Todos os recursos do Azure estão na região do Azure US West 2. A Contoso tem uma conexão ExpressRoute de 10 Gb com o Azure.
A empresa tem clientes em todo o mundo.

Ambiente existente

Active Directory
A Contoso tem uma implantação híbrida Microsoft Entra ID que sincroniza com on-premises Active Directory.

Ambiente de banco de dados
Contoso tem SQL Server 2017 no Azure virtual machines mostrado na tabela a seguir.

| Nome | Papel |
|---|---|
| SQL1 | Armazém de dados primário |
| SQL2 | armazém de dados secundários |
| SQL3 | servidor de extração, transformação e carga (ETL) |

SQL1 e SQL2 estão em um Always On availability group e são ativamente consultados. SQL3 roda jobs, fornece dados históricos e lida com a entrega de dados aos clientes.
O datacenter on-premises contém um servidor PostgreSQL que possui um banco de dados 50-TB.

Modelo de Negócio Atual
A Contoso usa o Microsoft SQL Server Integration Services (SSIS) para criar arquivos planos para clientes. Os clientes recebem os arquivos usando o FTP.

Requisitos

Mudanças Planejadas
A Contoso planeja migrar para um modelo no qual eles entregam dados para bancos de dados de clientes que funcionam como ofertas de plataforma como serviço (PaaS).
Quando um cliente estabelece um contrato de serviço com a Contoso, um grupo de recursos separado que contém um Azure SQL database será provisioned para o cliente. O banco de dados terá uma cópia completa dos dados financeiros. Os dados aos quais cada cliente terá acesso dependerão do nível do contrato de serviço. Os clientes podem alterar as camadas alterando seu contrato de serviço.
O tamanho estimado de cada banco de dados PaaS é 1 TB.
A Contoso planeja implementar as seguintes mudanças:
- Mova o banco de dados PostgreSQL para Azure Database for PostgreSQL durante os próximos seis meses.
- Atualize SQL1, SQL2 e SQL3 para SQL Server 2019 durante os próximos meses.
- Comece a integrar os clientes à nova solução PaaS dentro de seis meses.

Objetivos de Negócios
A Contoso identifica os seguintes requisitos de negócio:
- Use recursos integrados do Azure sempre que possível.
- Minimizar o esforço de desenvolvimento sempre que possível.
- Minimize os custos de computação das soluções PaaS.
- Forneça a todos os clientes sua própria cópia do banco de dados usando a solução PaaS.
- Forneça aos clientes diferentes acessos à tabela e à linha com base no contrato de serviço do cliente.
- No caso de uma interrupção regional do Azure, certifique-se de que os clientes possam acessar a solução PaaS com tempo de inatividade mínimo. A solução deve fornecer automatic failover.
- Certifique-se de que os usuários da solução PaaS possam criar seus próprios objetos de banco de dados, mas sejam impedidos de modificar qualquer um dos objetos existentes fornecidos pela Contoso.

Requisitos técnicos
A Contoso identifica os seguintes requisitos técnicos:
- Os usuários da solução PaaS devem ser capazes de entrar usando suas próprias credenciais corporativas Microsoft Entra ou ter credenciais Microsoft Entra fornecidas a eles pela Contoso. A solução deve evitar usar o Microsoft Entra ID interno da Contoso para minimizar os usuários convidados.
- Todos os clientes devem ter seu próprio grupo de recursos, servidor Azure SQL e Azure SQL database. A implantação de recursos para cada cliente deve ser feita de forma consistente.
- Os usuários devem ser capazes de rever as consultas emitidas contra os bancos de dados PaaS e identificar quaisquer novos objetos criados.
- O tempo de inatividade durante a migração do banco de dados PostgreSQL deve ser minimizado.

Requisitos de monitoramento
A Contoso identifica os seguintes requisitos de monitoramento:
- Notificar os administradores quando um banco de dados PaaS tiver um uso CPU maior que a média.
- Use um único painel para revisar dados de segurança e auditoria para todos os bancos de dados PaaS.
- Use um único painel para monitorar o desempenho da consulta e os gargalos em todas Os bancos de dados PaaS.
- Monitore os bancos de dados PaaS para identificar consultas com desempenho ruim e resolver problemas de desempenho da consulta automaticamente sempre que possível.

Protótipo PaaS
Durante a prototipagem da solução PaaS no Azure, você registra a utilização de computação do Azure SQL database de um cliente, conforme mostrado na exposição a seguir.
Imagem de referência (q080-2.png): gráfico “Compute utilization” no Azure portal. Show data for last: 7 dias. Aggregation type: Max. Séries: CPU percentage (Max), Data IO percentage (Max) e Log IO percentage (Max). Eixo X: aproximadamente de 4 a 10 de fevereiro (UTC-06:00). A utilização fica perto de 0% na maior parte do tempo, com picos curtos e intermitentes cerca de uma vez por dia. Os picos chegam a 40–100%. Data IO vai até ~100%, Log IO também chega a ~100% em alguns picos, e CPU costuma ficar entre 20–50%, com um pico de ~100%. O padrão é de uso intermitente, com picos e longos períodos ociosos.

Atribuções de Papel
Para o servidor Azure SQL Database de cada cliente, você planeja atribuir os papéis mostrados na exposição a seguir.
Imagem de referência (q080-3.png): no Azure portal, Access control > Role assignments. Filtros Type: Groups, Role: 2 selecionadas e Scope: All scopes. A tela mostra 15 role assignments no total e 2 itens (2 grupos):
| Papel | Nome | Tipo | Papel | Scope |
|---|---|---|---|---|
| Contribuidor | DBAGroup1 | Grupo | Contribuidor | Este recurso |
| Contribuidor SQL DB | DBAGroup2 | Grupo | Contribuidor SQL DB | Este recurso |

---

---

<a name="excluded-questions-not-in-current-outline"></a>

## Questões excluídas (fora do conteúdo atual)

Estes apareceram nas fontes, mas habilidades de teste que não estão no esboço de habilidades DP-300 a partir de 27 out 2026 (principalmente conteúdo legado DP-200 / DP-203 ou recursos aposentados), então eles não estão incluídos acima.

| Fonte IDs | Tópico | Motivo excluído |
|---|---|---|
| MD-214 | Migração/sincronização: Data Sync DB1 - BULK INSERT opção + CREATE INDEX param | SQL Data Sync - não no esboço atual (aposentando-se) |
| MD-064 | Migração/sincronização: Grupo Data Sync com private link - aprovar private endpoint | SQL Data Sync - não no esboço atual (aposentando-se) |
| MD-212 | Migração/sincronização: Data Sync sem endpoint público - Private Link + snapshot isolation | SQL Data Sync - não no esboço atual (aposentando-se) |
| MD-054 | Migração/sincronização: SQL Data Sync entre SQL1 e SQL2 sequência de configuração | SQL Data Sync - não no esboço atual (aposentando-se) |
| MD-110 | Classificação: registrar DB1 em Purview para digitalização | Governança Microsoft Purview - não no esboço |
| MD-195 | Defender: monitor central SQL VMs + DBs para vulnerabilidades | Microsoft Defender for SQL - removido do esboço |
| MD-130 | Defender: SIM protege bancos de dados da injeção SQL | Microsoft Defender for SQL - removido do esboço |
| NOTE-235 | Governança: Iniciativa Azure Policy para 20 políticas sobre SQL DBs | Azure Policy - não em resumo |
| MD-140 | Governança: bloqueio de recursos SIM para evitar a exclusão do db1 | Bloqueios de recursos do Azure - não em esboço |
| MD-207 | Monitoramento: Centralizado para DBs / MIs / VMs - SQL Insights | A resposta é aposentado SQL Insights |
| NOTE-030 | Design de tabela: Tipos de dados dos funcionários para reduzir o armazenamento | Design de tabela tipo de dados - não em esboço |
| MD-092, CQ-072, NOTE-16 | ADLS Gen2: acesso limitado por tempo - SAS | Armazenamento de dados do lago |
| MD-097, CQ-077 | Data Factory: O primeiro passo da criptografia CMK | Data Factory |
| CQ-020 | Data Factory: Git config adf_publish | Data Factory |
| NOTE-057 | Data Factory: gatilho de evento de armazenamento - Microsoft.EventGrid | Data Factory |
| MD-168 | Databricks: log de diagnóstico para mudanças de computação - clusters | Databricks |
| CQ-018 | Databricks: comando mágico % <language> | Databricks |
| CQ-009 | Databricks: partitionBy StoreID/Year/... parquet | Databricks |
| MD-007, CQ-024 | Databricks: reduzir o tempo de início do cluster - piscina | Databricks |
| MD-164 | Databricks: envie métricas para a sequência Azure Monitor | Databricks |
| MD-163 | Databricks: Spark analytics para detecção de intrusão | Databricks |
| CQ-021, NOTE-10-img030 | Databricks: modo de saída de streaming estruturado append | Databricks |
| CQ-010 | Event Hubs Dedicado: contagem de partição de mudança | Event Hubs |
| MD-198, NOTE-002 | SQL Insights: recolher com / armazenar em | SQL Insights |
| MD-191 | Insights SQL: dois recursos para criar primeiro | SQL Insights |
| CQ-002 | SSISDB após a perda mestre - anexar, TRUSTWORTHY, chave mestre aberta / criptografar | SSIS catálogo admin |
| NOTE-145 | Esquema da estrela: dimensão da data YYYYMMDD chave | DW star-schema design |
| CQ-014, NOTE-21 | Esquema de estrela: colunas de análise do site para tabelas | DW star-schema design |
| MD-153 | Análise de Stream Analytics (ADatum v2): contador para dimensionamento - SU % | Análise de Stream |
| MD-152 | Análise de Stream Analytics (ADatum v2): janela - tumbling | Análise de Stream |
| CQ-017, NOTE-55 | Análise de fluxo: picos de detecção de anomalias/dips | Análise de Stream |
| MD-166, NOTE-063 | Análise de fluxo: eventos de entrada backlogged - aumentar SUs | Análise de Stream |
| MD-169 | Analytics: dispositivos de duplicação - atraso da marca d'água | Análise de Stream |
| MD-162 | Análise de Stream Analytics: TopOne por jogo caindo 5 min | Análise de Stream |
| NOTE-60, MD-167 | Análise de Stream Analytics: duas métricas além SU % | Análise de Stream |
| NOTE-258 | Synapse (caso de varejo desconhecido): tipos de tabela para loja / dados promocionais | Distribuição Synapse |
| MD-010, CQ-027 | Arquivo da série Synapse DateTime: tabela externa de piscina dedicada | Tabelas externas Synapse |
| MD-008, CQ-025 | Arquivo da série Synapse DateTime: Get Metadata atividade | gasodutos Synapse |
| MD-009, CQ-026 | Arquivo da série Synapse DateTime: serverless tabela externa | Synapse serverless |
| CQ-011, NOTE-img075 | Synapse serverless: tabela somente de endereço do Parquet | Synapse serverless CETAS |
| CQ-022, NOTE-155 | Synapse serverless: OPENROWSET tripdata_2020 * FIRSTROW | Synapse serverless |
| CQ-012 | Sinapse Spark: estrutura variável de carga JSON - PySpark | Synapse Spark |
| CQ-016 | Sinapse Spark: tabelas visíveis na piscina serverless - Parquet | Synapse Spark |
| CQ-008 | Sinapse: CREATE EXTERNAL DATA SOURCE ADLS - dfs + HADOOP | Synapse PolyBase |
| MD-165, NOTE-023 | Synapse: data loads memory - classe de recurso maior | carga de trabalho Synapse mgmt |
| MD-171, NOTE-005 | Sinapse: desvio de dados - dm_pdw_nodes_db_partition_stats | Piscina dedicada Synapse |
| CQ-013, NOTE-3-img043 | Synapse: distribuição de dimensão de data - REPLICATE | Distribuição Synapse |
| CQ-015, NOTE-009 | Synapse: Distribuição FactSales + coluna de partição | Distribuição Synapse |
| MD-172 | Sinapse: PDW_SHOWSPACEUSED espeto | Piscina dedicada Synapse |
| CQ-019, NOTE-006 | Sinapse: PolyBase sequência de carga parquet | Synapse PolyBase |
| MD-170 | Synapse: consultas comuns lentas - porcentagem de cache hit | Piscina dedicada Synapse |
| NOTE-66 | Synapse: chave de substituição para tabela de dimensões | Synapse DW design |
| NOTE-085 | Synapse: website-visitas fact table distribuição + índice | Distribuição Synapse |
| MD-062 | Migração: What DMA migra para Azure SQL DB | A habilidade do núcleo é trivial sobre o Data Migration Assistant (que objetos DMA migra). O DMA foi aposentado em 16 de julho de 2025 e essa não é uma habilidade atual de migração. A chave B da fonte MD-062 (funções, procedimentos armazenados, visões e papéis) está correta para o exame clássico. |
