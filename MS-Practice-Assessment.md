# DP-300 — Questões da avaliação prática do Microsoft Learn

**79 questões** da avaliação prática oficial da Microsoft (109 no total, 50 por tentativa).
As respostas e explicações são da Microsoft. ⚠️ indica que você errou a questão em uma tentativa anterior.

## Planejar e implementar recursos da plataforma de dados (17)

### Q1

**Questão:**
Você tem uma instância on-premises Microsoft SQL Server 2019 que usa FileTable.

Você precisa migrar a instância para o Azure. A solução deve suportar o código do aplicativo existente.

O que você deve usar?

**Alternativas:**
- **A.** Azure SQL Database HyperScale
- **B.** Azure SQL Database serverless
- **C.** Azure SQL Managed Instance
- **D.** SQL Server on Azure Virtual Machines

**✅ Resposta:** D

<details><summary>Explicação</summary>

SQL Server on Azure Virtual Machines suporta FileTable. Ele pode ser usado para migrar a instância para o Azure e suportar o código do aplicativo existente.

Azure SQL Database serverless e Azure SQL Database Hyperscale são camadas de Azure SQL Database que não suportam FileTable. Azure SQL Managed Instance não suporta FileTable.

[Explorar Azure SQL Database Managed Instance - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/prepare-to-maintain-sql-databases-azure/5-explore-azure-sql-database-managed-instance?source=learn)  
[Compare as características do motor de banco de dados de SQL Database e SQL Managed Instance - Azure SQL Database & SQL Managed Instance | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/database/features-comparison?view=azuresql)

</details>

---

### Q2

**Questão:**
Você tem um Azure SQL Database que só é usado para processar dados por três horas por dia. Atualmente, o banco de dados tem o nível de serviço padrão S3 (100 DTU) configurado.

Você precisa recomendar uma solução que aumente o desempenho sem aumentar o custo.

Qual nível de serviço de banco de dados você recomenda?

**Alternativas:**
- **A.** Premium DTU
- **B.** Business Critical vCore
- **C.** Hyperscale vCore
- **D.** General Purpose serverless vCore

**✅ Resposta:** D

<details><summary>Explicação</summary>

Ao escolher o nível de serviço vCore serverless, o banco de dados estará ativo apenas durante a programação diária, e a quantidade de vCores pode ser configurada dinamicamente para lidar com a carga de trabalho.

Os níveis de serviço Premium, Hyperscale e Business Critical serão uma solução mais cara.

[O que é o Hyperscale nível de serviço? - Azure SQL Database - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/database/service-tier-hyperscale?view=azuresql)

[Entender SQL database hyperscale - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/deploy-paas-solutions-with-azure-sql/5-understand-sql-database-hyperscale)

</details>

---

### Q3

**Questão:**
Você planeja implantar um Azure SQL Database que incluirá uma tabela chamada Table1. Table1 incluirá os seguintes campos:

**Alternativas:**
- **A.** StockItemTransactionID (Int)
- **B.** StockItemID (Int)
- **C.** TransactionTypeID (Int)
- **D.** CustomerID (Int)
- **E.** InvoiceID (Int)
- **F.** SupplierID (Int)
- **G.** PurchaseOrderID (Int)
- **H.** TransactionOccurredWhen (Data)
- Quantidade **I.** (Int)
- **J.** LastEditedBy (Int)
- **K.** LastEditedWhen (Data)

Você precisa recomendar um método de particionamento para a tabela. A solução deve atender aos seguintes requisitos:
- **L.** Os dados devem ser mantidos por 2 anos.
- **M.** Todos os dados com mais de dois anos devem ser removidos uma vez por mês.
- **N.** O desempenho das operações de remoção de dados deve ser maximizado

Quais ações você deve executar? Cada resposta correta apresenta parte da solução.
- **O.** Criar a partition function para cada mês.
- **P.** Crie um partition function para cada ano.
- **Q.** Crie um partition scheme para grupos de arquivos para cada ano.
- **R.** Crie um partition scheme para o grupo de arquivos PRIMARY.
- **S.** Crie a tabela usando o partition scheme na coluna StockItemTransactionID.
- **T.** Crie a tabela usando o partition scheme na coluna TransactionOccurredWhen.

**✅ Resposta:** O, R, T

<details><summary>Explicação</summary>

Os dados devem ser removidos a cada mês para que seja mais eficiente criar um partition function para cada mês. Você só pode usar o grupo de arquivos PRIMARY no Azure SQL Database, e os dados devem ser particionados na data.

Além disso, como os dados devem ser removidos todos os meses, será menos eficiente criar uma partição para cada ano. Você não pode criar grupos de arquivos no Azure SQL Database.

[Criar tabelas e índices particionados - SQL Server, Azure SQL Database, Azure SQL Managed Instance - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/partitions/create-partitioned-tables-and-indexes?view=azuresqldb-current)

[Explorar desempenho e segurança - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/deploy-iaas-solutions-with-azure-sql/4-explore-performance-and-security?ns-enrollment-type=learningpath&amp;ns-enrollment-id=learn.wwl.plan-implement-data-platform-resources)

</details>

---

### Q4

**Questão:**
Você tem vários bancos de dados em um Azure SQL Managed Instance que usa o nível de serviço General Purpose vCore.

Você descobre alta latência em alguns dos arquivos de banco de dados.

O que fazer para reduzir a latência?

**Alternativas:**
- **A.** Adicione mais arquivos ao banco de dados.
- **B.** Adicione mais vCores à instância.
- **C.** Mude o nível de serviço.
- **D.** Aumente o tamanho do arquivo.

**✅ Resposta:** D

<details><summary>Explicação</summary>

Você deve aumentar o tamanho do arquivo para melhorar o desempenho. Se você tiver um arquivo que é até 128 GB, então você pode ter até 500 IOPS. Se você alterar o tamanho do arquivo para 1 TB, então você pode ser até 5,000 IOPS.

Você não deve adicionar mais vCores à instância no nível General Purpose, pois isso não alterará o IOPS disponível. Adicionar mais arquivos ao banco de dados também não ajuda, pois o número de IOPS disponível é calculado por tamanho do arquivo, não pelo número de arquivos. A alteração do nível de serviço mudará o tipo de CPU, o tamanho máximo do arquivo e a quantidade de memória reservada por CPU.

[Limites de recursos - Azure SQL Managed Instance - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/managed-instance/resource-limits?view=azuresql)

[Implantar soluções PaaS com Azure SQL - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/deploy-paas-solutions-with-azure-sql/)

</details>

---

### Q5

**Questão:**
Sua organização está enfrentando problemas de desempenho com uma tabela grande em um Azure SQL Database devido a operações frequentes de leitura e escrita.  
  
Você precisa otimizar o desempenho da tabela, melhorando a eficiência da consulta e reduzindo o uso de armazenamento.  
  
O que você deve fazer?

**Alternativas:**
- **A.** Aplique a compressão no nível da página à tabela.
- **B.** Crie um clustered index na tabela.
- **C.** Ativar compressão de nível de linha para a tabela.
- **D.** Partição da tabela com base na data.

**✅ Resposta:** C

<details><summary>Explicação</summary>

A compressão no nível da linha otimiza o armazenamento reduzindo o tamanho dos dados armazenados e melhora o desempenho da consulta minimizando as operações de E/S, tornando-a a solução mais adequada para tabelas com operações frequentes de leitura e gravação. Criar um clustered index aprimora o desempenho da consulta, mas não aborda a otimização do armazenamento. A compactação no nível da página reduz efetivamente o armazenamento, mas introduz uma sobrecarga maior do CPU, tornando-o menos adequado para atualizações frequentes. A partição baseada na data melhora o desempenho da consulta para cenários específicos, mas não reduz inerentemente os requisitos de armazenamento.  
  
[Tune and maintain indexes - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/evaluate-performance-improvements/3-tune-and-maintain-indexes)  
[Explore database maintenance checks - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/configure-databases-for-optimal-performance/2-explore-database-maintenance-checks)

</details>

---

### Q6

**Questão:**
Você planeja implantar um SQL Server em uma instância Azure Virtual Machines em execução em um sistema operacional Windows Server.

Você precisa configurar o armazenamento para a máquina virtual. A solução deve garantir o mais alto desempenho para os arquivos de dados.

Quais ações você deve executar? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** Defina o cache do host para nenhum.
- **B.** Definir cache host para read-only.
- **C.** Defina o cache do host para ler/escrever.
- **D.** Desligue vários discos usando espaços de armazenamento.
- **E.** Use discos de aceleração de gravação.

**✅ Resposta:** B, D

<details><summary>Explicação</summary>

Você deve remover vários discos, uma vez que aumentará o IOPS, bem como definir o cache do host para read-only, pois os arquivos de dados funcionam melhor usando esse tipo de cache do host.

A configuração do cache do host para nenhum significa que as leituras não serão armazenadas em cache, a configuração do cache do host de ler / escrever degradará o desempenho e levará à perda de dados. Os discos do acelerador de gravação devem ser usados para discos de log de transação.

[Armazenamento: Melhores práticas e diretrizes de desempenho - SQL Server on Azure VMs - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage?source=recommendations&amp;view=azuresql)

[Optimize database storage - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-sql-server-resources-optimal-performance/4-optimize-database-storage)

</details>

---

### Q7

**Questão:**
Você está implantando uma instância SQL Server on Azure Virtual Machines que será usada como um ambiente de desenvolvimento. Você desenvolverá dois pequenos bancos de dados, um dos quais terá tabelas na memória.

Você precisa escolher o tipo de máquina virtual mais adequado para a implantação.

O que você deve escolher?

**Alternativas:**
- **A.** computação otimizada
- **B.** general purpose
- **C.** GPU Otimizado
- **D.** memória otimizada

**✅ Resposta:** B

<details><summary>Explicação</summary>

Os tamanhos de máquinas virtuais General purpose fornecem uma relação CPU -para-memória equilibrada. Isso é ideal para testes e desenvolvimento, bancos de dados pequenos a médios e servidores web de baixo a médio tráfego.

Tamanhos de máquinas virtuais otimizadas para computação têm uma alta relação CPU -para memória. Isso é bom para servidores web de tráfego médio, eletrodomésticos de rede, processos em lote e servidores de aplicativos.

Os tamanhos de máquina virtual otimizados para memória oferecem uma alta relação memória-para-CPU que é usada para servidores de banco de dados relacionais, caches médios a grandes e análises na memória.

Os tamanhos de máquinas virtuais otimizadas GPU são máquinas virtuais especializadas disponíveis com GPUs único, múltiplo ou fracionário. Esses tamanhos são projetados para cargas de trabalho intensivas em computação, gráficas e visualização.

[Azure VM tamanhos - General purpose - Azure Virtual Machines - Training | Microsoft Learn](https://learn.microsoft.com/azure/virtual-machines/sizes-general)

[Prepare-se para manter SQL databases on Azure - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/prepare-to-maintain-sql-databases-azure/)

</details>

---

### Q8

**Questão:**
Você está implantando uma instância SQL Server on Azure Virtual Machines para um ambiente de produção. Uma das bancos de dados neste caso tem tabelas na memória.

Você precisa escolher o tipo de máquina virtual mais adequado para a implantação.

O que você deve escolher?

**Alternativas:**
- **A.** computação otimizada
- **B.** general purpose
- **C.** GPU Otimizado
- **D.** memória otimizada

**✅ Resposta:** D

<details><summary>Explicação</summary>

Os tamanhos de máquina virtual otimizados para memória oferecem uma alta relação memória-para-CPU que é boa para servidores de banco de dados relacionais, caches médios a grandes e análises na memória.

As máquinas virtuais General purpose fornecem uma relação CPU-para-memória equilibrada. Estes são melhores para testes e desenvolvimento, bancos de dados pequenos a médios e servidores web de tráfego baixo a médio.

As máquinas virtuais otimizadas para computação têm uma alta relação CPU -para memória. Estas são usadas para servidores web de tráfego médio, eletrodomésticos de rede, processos em lote e servidores de aplicativos.

Os tamanhos de máquinas virtuais otimizadas GPU são máquinas virtuais especializadas disponíveis com GPUs único, múltiplo ou fracionário. Esses tamanhos são projetados para cargas de trabalho intensivas em computação, gráficas e visualização.

[Azure VM tamanhos - Memória - Azure Virtual Machines - Training | Microsoft Learn](https://learn.microsoft.com/azure/virtual-machines/sizes-memory)

[Prepare-se para manter SQL databases on Azure - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/prepare-to-maintain-sql-databases-azure/)

</details>

---

### Q9

**Questão:**
Você está planejando uma migração de on-premises Microsoft SQL Server para Azure usando o Azure Database Migration Service.

Você precisa manter bancos de dados on-line durante a migração.

Quais dois serviços você pode usar? Cada resposta correta apresenta uma solução completa.

**Alternativas:**
- **A.** Azure Cosmos DB
- **B.** Azure SQL Database
- **C.** Azure SQL Managed Instance
- **D.** SQL Server on Azure Virtual Machines

**✅ Resposta:** C, D

<details><summary>Explicação</summary>

SQL Server on Azure Virtual Machines e Azure SQL Managed Instance apoiam a migração online.

O Azure SQL Database não suporta migração online e o Azure CosmosDB não é um banco de dados relacional.

[Cenários de migração de banco de dados suportados - Azure Database Migration Service - Training | Microsoft Learn](https://learn.microsoft.com/azure/dms/resource-scenario-status)

[Migrar SQL cargas de trabalho para Azure SQL Databases - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/migrate-sql-workloads-azure-sql-databases/)

</details>

---

### Q10

**Questão:**
Você tem um banco de dados on-premises que usa Transparent Data Encryption (TDE).

Você planeja migrar o banco de dados para um Azure SQL Managed Instance chamado Instance1. 

Você exporta o certificado TDE para um arquivo. 

Você precisa garantir que você pode restaurar o banco de dados para Instance1.

Onde você deve fazer o upload do certificado?

**Alternativas:**
- **A.** um cofre de serviços de recuperação
- **B.** Azure key vault
- **C.** Instance1
- **D.** Azure Blob Storage

**✅ Resposta:** C

<details><summary>Explicação</summary>

O certificado deve ser carregado para o managed instance, pois o certificado não deve ser armazenado em um cofre de serviços de recuperação ou um cofre chave para o managed instance ter acesso.

[Migrar TDE certificado - managed instance - Azure SQL Managed Instance - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/managed-instance/tde-certificate-migrate?view=azuresql&amp;tabs=azure-powershell)

[Migrar SQL workloads to Azure Managed Instances - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/migrate-sql-workloads-azure-managed-instances/)

</details>

---

### Q11

**Questão:**
Você tem 20 instâncias on-premises Microsoft SQL Server.

Você precisa migrar as instâncias para o Azure. A solução deve suportar a auditoria no nível do servidor.

Quais dois serviços você pode usar? Cada resposta correta apresenta uma solução completa.

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Azure SQL Managed Instance; SQL Server on Azure Virtual Machines

<details><summary>Explicação</summary>

SQL Server on Azure Virtual Machines e Azure SQL Managed Instance suportam SQL Server Audit e podem ser usados.

Azure SQL Database Serverless serverless é uma camada de computação de Azure SQL Database que não suporta SQL Server Audit. Azure SQL Database Elastic PoolElastic O pool é uma oferta de banco de dados compartilhada de Azure SQL Database que não suporta SQL Server Audit. Azure SQL Database HyperScale Hyperscale é um nível de serviço de Azure SQL Database que não suporta SQL Server Audit.

Explorar Azure SQL Database Managed Instance - Training | Microsoft Learn

Diferenças de T-SQL entre SQL Server e Azure SQL Managed Instance - Azure SQL Managed Instance | Microsoft Learn, e SQL Server Audit (Database Engine) - SQL Server | Microsoft Learn

</details>

---

### Q12

**Questão:**
Você está construindo um aplicativo multi-tenant que usará um Azure SQL Database.

Você precisa dividir o banco de dados. A solução deve atender aos seguintes requisitos:

Melhorar a escalabilidade.
Reduza a contenção.
Otimize o desempenho.
Isolar os dados de cada inquilino.
O que você deve usar?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** uma lista shard mapa apenas

<details><summary>Explicação</summary>

Usando apenas uma lista shard mapa é a solução correta, porque os dados de cada inquilino serão armazenados em um banco de dados separado.

Um mapa de alcance shard permitirá que os dados de vários inquilinos sejam armazenados no mesmo banco de dados.

Explorar desempenho e segurança - Treinamento | Microsoft Learn

Estratégias de data partitioning - Azure Architecture Center - Treinamento | Microsoft Learn

</details>

---

### Q13

**Questão:**
Você tem uma instância on-premises do Microsoft SQL Server 2019 que usa um procedimento armazenado SQL (CLR) para acessar um compartilhamento de arquivos local no servidor.

Você precisa migrar a instância para o Azure.

Para qual serviço do Azure você deve migrar a instância?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** SQL Server on Azure Virtual Machines

<details><summary>Explicação</summary>

SQL Server em Azure Virtual Machine suporta objetos SQL CLR. Os objetos podem acessar arquivos compartilhamentos, discos e pastas.

Azure SQL Database serverless é uma camada de computação para Azure SQL Database que não suporta objetos SQL CLR.

Azure SQL Database Hyperscale é um nível de serviço do Azure SQL Database que não suporta objetos SQL CLR.

O Azure SQL Managed Instance não suporta objetos SQL CLR que acessam compartilhamentos de arquivos ou pastas do sistema operacional.

Compare as características do motor de banco de dados de SQL Database e SQL Managed Instance - Azure SQL Database & SQL Managed Instance - Training | Microsoft Learn

T-SQL Diferenças entre SQL Server & Azure SQL Managed Instance - Azure SQL Managed Instance - Training | Microsoft Learn

</details>

---

### Q14

**Questão:**
Você planeja migrar um banco de dados 40- TB OLTP para Azure SQL Database.

Qual nível de serviço você deve recomendar para o banco de dados?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Nível de serviço Hyperscale vCore

<details><summary>Explicação</summary>

Você deve recomendar o nível de serviço Hyperscale vCore, pois ele suporta tamanhos de banco de dados de até 100 TB.

O nível de serviço General Purpose e Business Critical vCore e o nível de serviço DTU Premium suportam apenas um tamanho de banco de dados de até 4 TB.

O que é o Hyperscale nível de serviço? - Azure SQL Database - Training | Microsoft Learn

Entender SQL database hyperscale - Training | Microsoft Learn

</details>

---

### Q15

**Questão:**
Você tem um grupo de recursos chamado RG1 que contém um nível de serviço General Purpose Azure SQL Managed Instance chamado Instance1.

Você precisa mudar o nível de serviço do Instance1 para Business Critical.

Qual comando você deve executar?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Set-AzSqlInstance -Name Instance1 -ResourceGroupName RG1 -Edition BusinessCritical

<details><summary>Explicação</summary>

Set-AzSqlInstance -Name Instance1 -ResourceGroupName RG1 -Edition BusinessCritical é o comando correto. Ele define a edição como Business Critical.

Set-AzSqlElasticPool -Edition BusinessCritical -Name mydb -ServerName Instance1 -ResourceGroupName RG1 definirá a edição de um Azure SQL Database elastic pool para Business Critical.

Set-AzSqlInstance -Name Instance1 -ResourceGroupName RG1 -Edition GeneralPurpose definirá a edição para General Purpose.

Set-AzSqlDatabase -ResourceGroupName RG1 -Name mydb -ServerName Instance1 -Edition BusinessCritical definirá a edição de um Azure SQL Database para Business Critical.

Set-AzSqlInstance (Az.Sql) - Training | Microsoft Learn

Prepare-se para manter SQL databases on Azure - Training | Microsoft Learn

</details>

---

### Q16

**Questão:**
Sua organização usa o Azure SQL Database para armazenar dados transacionais. O banco de dados contém tabelas grandes com inserções, atualizações e exclusões frequentes, e a análise de desempenho mostra varreduras de tabela frequentes que estão aumentando as operações de E/S e diminuindo as cargas de trabalho do OLTP.

Você precisa melhorar o desempenho da consulta e reduzir as operações de E/S, mantendo a integridade dos dados.

Cada resposta correta apresenta parte da solução. Que duas ações você deve realizar?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Criar nonclustered indexes.; Habilitar row compression.

<details><summary>Explicação</summary>

A criação do nonclustered indexes melhora o desempenho da consulta, permitindo pesquisas eficientes para linhas específicas, o que reduz a necessidade de varreduras completas de tabela e minimiza as operações de E/S. O Row compression diminui o tamanho dos dados armazenados, levando a menos páginas sendo lidas durante consultas e melhor desempenho de E/S. Columnstore compression não é adequado para esse cenário porque foi projetado para dados raramente acessados e pode aumentar o uso do CPU para tabelas com alta atividade de leitura e gravação.

Design indexes - Training | Microsoft Learn
Descreva intelligent query processing - Training | Microsoft Learn

</details>

---

### Q17

**Questão:**
Você tem uma instância on-premises Microsoft SQL Server 2019.

Você precisa migrar a instância para um Azure SQL Managed Instance. A solução deve minimizar o tempo de inatividade.

O que você deve fazer primeiro?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Crie um Azure SQL Managed Instance com hardware de série premium. Crie backups completo e log com WITH CHECKSUM. Armazene o backups em uma partilha de rede SMB.

<details><summary>Explicação</summary>

O hardware Premium deve ser usado para uma migração on-line. O backups também deve ser tomado com o CHECKSUM ativado.

O hardware padrão não pode ser usado para uma migração on-line e armazenar o backups no Blob storage cria um Azure snapshot dos arquivos de banco de dados.

Tutorial: Migrar SQL Server online para SQL Managed Instance - Azure Database Migration Service - Training | Microsoft Learn

</details>

---

## Implementar um ambiente seguro (18)

### Q18

**Questão:**
Você cria um procedimento armazenado que executa código SQL dinâmico para selecionar dados de uma tabela chamada Vendas. 

Todos os usuários têm permissões de execução para o procedimento armazenado. 

O que você deve fazer para garantir que os usuários possam executar o procedimento sem erros?

**Alternativas:**
- **A.** Adicione uma cláusula WITH RECOMPILE ao procedimento armazenado.
- **B.** Adicione uma cláusula EXECUTE AS [proprietário] ao procedimento armazenado.
- **C.** Adicione a permissão SELECT à tabela de vendas para os usuários.

**✅ Resposta:** C

<details><summary>Explicação</summary>

Você deve adicionar a permissão SELECT à tabela de vendas porque a execução do código SQL dinâmico requer que o chamador tenha todas as permissões necessárias para os objetos que são chamados no código.

Usar a cláusula EXECUTE AS [proprietário] ou a cláusula WITH RECOMPILE não garante que o chamador tenha todas as permissões necessárias para os objetos que são chamados no código.

[EXECUTE AS (Transact-SQL) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/t-sql/statements/execute-as-transact-sql?view=sql-server-ver16)

[Configurar autenticação e autorização do banco de dados - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-database-authentication-authorization/)

</details>

---

### Q19

**Questão:**
Você tem um aplicativo que armazena dados para vários inquilinos usando Azure SQL Database elastic pools. Cada inquilino tem um banco de dados. 

Você precisa configurar uma solução de autenticação para o aplicativo. A solução deve atender aos seguintes requisitos:

**Alternativas:**
- **A.** Certifique-se de que os usuários só podem acessar o banco de dados de seus respectivos inquilinos.
- **B.** Certifique-se de que os bancos de dados podem ser movidos para uma instância dedicada.
- **C.** Minimizar o esforço administrativo.

O que você deve incluir na solução?
- **D.** usuários de banco de dados contido
- **E.** funções de banco de dados
- **F.** funções do servidor

**✅ Resposta:** D

<details><summary>Explicação</summary>

Os usuários de banco de dados contidos permitem que os bancos de dados sejam movidos sem recriar o logins.

As funções de servidor gerenciam permissões apenas no nível do servidor e as funções de banco de dados gerenciam permissões dentro de um banco de dados.

[Contained user acesso a bancos de dados confinadas - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/security/contained-database-users-making-your-database-portable?view=sql-server-ver16)

[Configurar autenticação e autorização do banco de dados - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-database-authentication-authorization/)

</details>

---

### Q20

**Questão:**
Você tem um servidor Azure SQL que hospeda 10 Azure SQL Databases.

Você cria um aplicativo chamado App1 com um sistema atribuído managed identity.

Você precisa garantir que o App1 tenha acesso de leitura a cada banco de dados. usando o managed identity do App1.

Quais ações você deve executar? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** Atribua ao usuário do banco de dados a função de leitor db \_ para cada banco de dados.
- **B.** Atribua ao usuário do banco de dados a função de leitor db \_ para o banco de dados mestre.
- **C.** Criar um login para App1 de um provedor externo.
- **D.** Crie um login para App1 que use uma senha.
- **E.** Crie um usuário em cada banco de dados para o App1 login.
- **F.** Crie um usuário no banco de dados mestre para o App1 login.

**✅ Resposta:** A, C, E

<details><summary>Explicação</summary>

A managed identity exige que a login ser criado a partir de um provedor externo, cada banco de dados requer um usuário para o logine atribuindo ao usuário da banco de dados o db\_A função de leitor para cada banco de dados permitirá o acesso de leitura às bancos de dados do usuário.

A managed identity não pode usar Windows authentication. O aplicativo não requer um usuário no banco de dados mestre e atribui ao usuário do banco de dados o db\_O papel do leitor para o banco de dados mestre permitirá que o usuário leia o acesso ao banco de dados mestre, mas não leia o acesso aos bancos de dados do usuário.

[Describe Security Principals - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-database-authentication-authorization/4-describe-security-principals)

</details>

---

### Q21

**Questão:**
Você tem um Azure SQL Database chamado db1 que inclui dois papéis personalizados chamados leitores db1 \_ e escritores db2 \_. Você tem um grupo Microsoft Entra chamado dbadmin.

Você precisa garantir que os membros do grupo dbadmin possam gerenciar a associação de leitores db1 \_ e escritores db1 \_. A solução deve seguir o princípio do menor privilégio.

Quais ações você deve executar? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** Atribua ao usuário do banco de dados a função accessadmin db \_ para o banco de dados db1.
- **B.** Atribua ao usuário do banco de dados a função securityadmin db \_ para o banco de dados db1.
- **C.** Atribua ao usuário do banco de dados o papel de gerenciador de login para o banco de dados mestre.
- **D.** Atribua ao usuário do banco de dados a função sysadmin para o banco de dados mestre.
- **E.** Crie um login e um usuário de banco de dados no db1 para o grupo dbadmin.
- **F.** Crie um login e um usuário de banco de dados no banco de dados mestre para o grupo dbadmin.

**✅ Resposta:** B, E

<details><summary>Explicação</summary>

Você deve criar um login e um usuário de banco de dados no db1 para o grupo dbadmin, porque os membros do grupo dbadmin precisam de um usuário no banco de dados db1. Os membros da função securityadmin db \_ podem modificar a associação de funções para funções personalizadas.

As opções restantes estão incorretas. Os membros do grupo dbadmin não precisam de permissões de nível de servidor. Os membros da função sysadmin podem realizar qualquer atividade na instância. Os membros da função accessadmin do db \_ podem adicionar e remover o acesso ao banco de dados. Os membros da função loginmanager podem criar e excluir logins no banco de dados mestre virtual.

https://learn.microsoft.com/sql/relational-databases/security/authentication-access/database-level-roles?view=sql-server-ver16[Funções de nível de banco de dados - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles?view=sql-server-ver16)

[Configurar autenticação e autorização do banco de dados - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-database-authentication-authorization/)

</details>

---

### Q22

**Questão:**
Você tem um banco de dados do Azure SQL Database chamado db1 que contém uma tabela chamada Vendas.

Você precisa criptografar dados em uma coluna chamada CreditCard na tabela de vendas.

Quais ações você deve executar antes de criptografar os dados? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** Crie um certificado no banco de dados db1.
- **B.** Crie um certificado no banco de dados mestre.
- **C.** Crie uma chave simétrica criptografada por um certificado.
- **D.** Crie uma chave assimétrica criptografada por um certificado.

**✅ Resposta:** A, C

<details><summary>Explicação</summary>

O certificado criado em db1 será usado para criptografar uma chave simétrica, que, por sua vez, será usada para proteger os dados na coluna CreditCard na tabela de vendas. A coluna contendo dados criptografados deve ser coluna varbinária.

A chave simétrica, não uma chave assimétrica, é necessária para esta operação. Criar o certificado no banco de dados mestre não funcionaria, pois o certificado necessário para criptografia deve ser criado no banco de dados do usuário.  

[Criptografar uma coluna de dados - SQL Server & Azure Synapse Analytics & Azure SQL Database & SQL Managed Instance - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/security/encryption/encrypt-a-column-of-data?view=sql-server-ver16)

[Proteger dados em trânsito e em repouso - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/protect-data-transit-rest/)

</details>

---

### Q23

**Questão:**
Você tem um Azure SQL Database chamado db1.

Você precisa garantir que todos os dados no db1 estejam criptografados em repouso.

O que você deve usar?

**Alternativas:**
- **A.** Always Encrypted
- **B.** Azure SQL auditing
- **C.** Microsoft Defender for SQL
- **D.** Transparent Data Encryption (TDE)

**✅ Resposta:** D

<details><summary>Explicação</summary>

TDE é o recurso Azure SQL Database que criptografa dados em repouso. Todos os dados salvos em um disco são criptografados.

A auditoria não é usada para executar a criptografia de dados em repouso, ela é usada para coletar informações de diagnóstico.

Defender for SQL é usado para mitigar possíveis vulnerabilidades de banco de dados e detectar atividades anômalas.

O Always Encrypted protege os dados que estão em uso ou em trânsito.

[Explorar Transparent Data Encryption - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/protect-data-transit-rest/2-explore-transparent-data-encryption)

</details>

---

### Q24

**Questão:**
Em um Azure SQL database, você tem uma tabela que contém uma coluna chamada Column1. Column1 armazena dados de cartão de crédito CVV.

Você precisa configurar o Column1 para atender aos seguintes requisitos:

**Alternativas:**
- **A.** Os dados na coluna devem ser protegidos em repouso.
- **B.** Os dados na coluna devem ser protegidos quando em uso.
- **C.** Os administradores de banco de dados devem ser impedidos de visualizar o conteúdo do Column1.

Quais duas características você pode usar? Cada resposta correta apresenta uma solução completa.
- **D.** Criptografia determinística Always Encrypted
- **E.** Criptografia aleatória Always Encrypted
- **F.** Cript encriptação a nível de coluna utilizando chaves simétricas
- **G.** dynamic data masking

**✅ Resposta:** D, E

<details><summary>Explicação</summary>

Você deve usar Always Encrypted (criptografia aleatória ou criptografia determinística), porque isso garante que os administradores são incapazes de visualizar e descriptografar dados. 

A criptografia com chaves simétricas permitirá que os administradores acessem os dados. O Dynamic data masking não protege os dados em repouso ou em uso.

[Explicar a criptografia de objeto e secure enclaves - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/protect-data-transit-rest/4-explain-object-encryption-secure-enclaves)

</details>

---

### Q25

**Questão:**
Você tem um Azure SQL Database que contém uma tabela chamada Funcionário. A tabela tem colunas que são mascaradas usando dynamic data masking. 

Quando um usuário copia a tabela Empregado usando o comando SELECT INTO, os dados são mascarados em uma nova tabela. O usuário deve ver os dados desmascarados na nova tabela. 

Você precisa conceder as permissões apropriadas ao usuário. A solução deve seguir o princípio do menor privilégio.

O que você deve conceder ao usuário?

**Alternativas:**
- **A.** a permissão ALTER ANY MASK
- **B.** a permissão SELECT para a nova tabela
- **C.** a permissão UNMASK para a tabela de Empregado
- **D.** a permissão UNMASK para a nova tabela

**✅ Resposta:** C

<details><summary>Explicação</summary>

Você deve conceder a permissão UNMASK ao usuário para a tabela de funcionários porque os dados devem ser desmascarados antes de copiá-los para a nova tabela.

A concessão da permissão UNMASK ao usuário para a nova tabela não permite que o usuário veja os dados desmascarados na nova tabela, pois os dados não podem ser desmascarados depois de serem copiados para a nova tabela.

A permissão do ALTER ANY MASK é dedicada ao oficial de segurança, pois permite que o gerenciamento do dynamic data masking e a permissão SELECT não torne os dados mascarados legíveis.

[Dynamic Data Masking - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/security/dynamic-data-masking?view=sql-server-ver16)

[Implementar controles de conformidade para dados confidenciais - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/implement-compliance-controls-sensitive-data/)

</details>

---

### Q26

**Questão:**
Você tem um Azure SQL managed instance Azure SQL Managed Instance que hospeda um banco de dados.   
  
O banco de dados contém uma coluna chamada Column1 que armazena informações confidenciais.

Você precisa auditar o acesso ao Column1. A solução deve atender aos seguintes requisitos:

**Alternativas:**
- **A.** Manter os dados de auditoria por pelo menos dezoito anos.
- **B.** Minimizar o esforço administrativo.

Quais ações você deve executar? Cada resposta correta apresenta parte da solução.
- **C.** Aplique uma classificação de sensibilidade à coluna.
- **D.** Configure um destino de log de auditoria Log Analytics.
- **E.** Configure um destino de log de auditoria de armazenamento.
- **F.** Habilitar Always Encrypted para a coluna.
- **G.** Habilitar Azure SQL Auditing.
- **H.** Habilitar Transparent Data Encryption (TDE).

**✅ Resposta:** C, E, G

<details><summary>Explicação</summary>

Azure SQL Auditing precisa estar habilitado para monitorar o acesso. Uma conta de armazenamento permite retenção ilimitada. A coluna precisa de um rótulo de classificação para que o acesso seja auditado.

Log Analytics tem um limite máximo de retenção de sete anos, Always Encrypted é usado para proteger dados em uso ou em trânsito e TDE criptografa arquivos de dados.

[Explore server and database audit - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/implement-compliance-controls-sensitive-data/3-explore-server-and-database-audit)  
[Comece com a auditoria Azure SQL Managed Instance](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/auditing-configure?view=azuresql)

</details>

---

### Q27

**Questão:**
Você tem um Azure SQL Database chamado db1 que contém uma tabela. A tabela contém uma coluna que armazena endereços de email.

Você precisa garantir que os usuários que relatam só podem visualizar endereços de e-mail em um formato de aXXX@XXX.com. A solução deve minimizar o impacto sobre outros usuários.

O que você deve configurar?

**Alternativas:**
- **A.** Always Encrypted
- **B.** criptografia de coluna
- **C.** dynamic data masking
- **D.** row-level security (RLS)

**✅ Resposta:** C

<details><summary>Explicação</summary>

O Dynamic data masking permitirá que os usuários vejam apenas valores mascarados em vez de endereços de e-mail.

Always Encrypted protege os dados em uso ou em trânsito. RLS O acesso a linhas em uma tabela e a criptografia de coluna criptografará os dados na coluna, o que pode dificultar a capacidade dos usuários de visualizar os dados.

[Dynamic Data Masking - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/security/dynamic-data-masking?view=sql-server-ver16)

[Implementar controles de conformidade para dados confidenciais - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/implement-compliance-controls-sensitive-data/)

</details>

---

### Q28

**Questão:**
Você tem um banco de dados Microsoft SQL Server chamado SalesDB hospedado em um Azure virtual machine. SalesDB faz parte de um Always On availability group que usa um listener chamado SalesAGListener e contém uma tabela chamada dbo.Orders.

Um aplicativo Azure Functions usa a ligação de gatilho Azure SQL para processar inserções e atualizações do dbo.Orders, mas a função não dispara.

Você precisa configurar o SalesDB para que a função seja acionada quando ocorrerem alterações comprometidas no dbo.Orders. A solução deve usar um período de retenção de dois dias e limpeza automática.

Qual código T-SQL você deve executar?

**Alternativas:**
- **A.** ALTER DATABASE [SalesDB]  
SET CHANGE\_TRACKING = ON  
(CHANGE\_RETENTION = 2 DAYS, AUTO\_CLEANUP = ON);  
ALTER TABLE dbo.Orders  
ENABLE CHANGE \_ TRACKING;
- **B.** EXEC sys.sp \_ cdc \_ habilitar \_ db;  
EXEC sys.sp \_ cdc \_ ativar a tabela \_  
  @source\_schema = 'dbo',  
  @source\_name = 'Orders',  
  @role\_name = NULL,  
  @capture\_instance = 'dbo\_«Ordens»,  
  @supports\_net\_changes = 1;
- **C.** CREATE EVENT STREAMING GROUP MyOrderStream  
WITH (  
  TARGET\_TYPE = 'eventhub',  
  TARGET\_NAME = 'orders-eventhub',  
  TARGET\_CONNECTION = '&lt;event-hub-connection-string&gt;'  
);  
ALTER EVENT STREAMING GROUP MyOrderStream  
ADD TABLE dbo.Orders;
- **D.** CREATE TRIGGER dbo. MyOrderStream ON dbo.Orders  
AFTER INSERT  
AS  
BEGIN  
    SET NOCOUNT ON;  
    DECLARE @payload NVARCHAR(MAX) = (  
        SELECT \*  
        FROM inserido  
        FOR JSON AUTO  
    );  
    DECLARE @response NVARCHAR(MAX);  
    EXEC sys.sp \_ invocar \_ externo \_ resto \_ endpoint  
        @url = N'https://&lt;your-function&gt; ',  
        @method = 'POST',  
        @headers = N'{"Content-Type":"application/json"}',  
        @payload = @payload,  
        @response = @response OUTPUT;  
END;

**✅ Resposta:** A

<details><summary>Explicação</summary>

**Objectivo:**

2.3 Implementar controles de conformidade para dados confidenciais

**O que este item testa:**

Implementar o rastreamento de dados de mudança

**Leitura adicional:**  

- [Replicação, change tracking, & change data capture - Always On availability groups](https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/replicate-track-change-data-capture-always-on-availability?view=sql-server-ver17#change-tracking)
- [Alterações de dados da faixa (SQL Server)](https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/track-data-changes-sql-server?view=sql-server-ver17)

**Racionale:**

Azure SQL As ligações de gatilho dependem de SQL Server Change Tracking, assim permitindo change tracking ao nível do banco de dados com um período de retenção de dois dias e limpeza automática, e então permitindo change tracking E-mail dbo.Orders, permite que a função detecte mudanças comprometidas. Change data capture (CDC) usa tabelas de digitalização e captura de logs e não é o mecanismo usado por Azure SQL grupos de transmissão de eventos enviam mudanças de tabela para alvos de streaming externos, em vez de permitir a detecção de mudanças dentro SQL Servere tabelas temporais versionadas pelo sistema armazenam o histórico da linha, mas não fornecem change tracking metadados exigidos pela ligação de gatilho.

</details>

---

### Q29

**Questão:**
Você tem um Azure SQL Database chamado db1.

Você precisa fornecer acesso ao db1 para um usuário Microsoft Entra ID.

O que você deve fazer?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Em db1, corra o CREATE USER declaração com a FROM EXTERNAL PROVIDER opção.

<details><summary>Explicação</summary>

Você deve usar a declaração CREATE USER com a opção FROM EXTERNAL PROVIDER em db1, pois esse usuário deve ter acesso a esse banco de dados.

Usando a instrução CREATE USER com a opção FROM LOGIN ou WITH PASSWORD cria um login na banco de dados mestre. Por padrão, o login não tem acesso às bancos de dados.

CREATE USER (Transact-SQL) - SQL Server - Training | Microsoft Learn

Configurar autenticação e autorização do banco de dados - Treinamento | Microsoft Learn

</details>

---

### Q30

**Questão:**
Você tem um Azure SQL Database chamado db1.

Você cria um usuário chamado User1 no banco de dados mestre.

Você precisa garantir que o User1 possa criar bancos de dados. A solução deve seguir o princípio do menor privilégio.

Qual papel você deve atribuir ao User1?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** dbmanager

<details><summary>Explicação</summary>

Você deve atribuir dbmanager, como membros deste papel pode criar bancos de dados.

Os membros do papel de gerenciador de login podem criar e excluir usuários, mas não podem criar bancos de dados. Os membros do papel db_ddladmin podem executar quaisquer declarações DDL. Os membros do papel db_owner podem realizar todas as atividades no escopo do banco de dados.

Configurar autenticação e autorização do banco de dados - Treinamento | Microsoft Learn

</details>

---

### Q31 ⚠️

**Questão:**
Você tem um banco de dados chamado db1 em um Azure SQL managed instance chamado Instance1.

Você precisa criptografar db1 usando Transparent Data Encryption (TDE).

Quais ações você deve executar antes de iniciar a criptografia? Cada resposta correta apresenta parte da solução.

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Criar um certificado no banco de dados mestre; Criar uma chave de criptografia de banco de dados no db1; Criar uma chave mestre no banco de dados mestre.

<details><summary>Explicação</summary>

O procedimento de preparação do banco de dados para usar o TDE é primeiro criar uma chave mestra e um certificado no banco de dados mestra e, em seguida, criar uma chave de criptografia de banco de dados no db1.

Você não deve criar um certificado ou uma chave assimétrica no db1, já que o TDE só pode usar certificados criados no banco de dados mestre.

Transparent data encryption (TDE) - SQL Server - Training | Microsoft Learn

Proteger dados em trânsito e em repouso - Treinamento | Microsoft Learn

</details>

---

### Q32

**Questão:**
Você tem um Azure SQL Database chamado db1. O banco de dados é um banco de dados contido.

Você precisa atualizar uma regra de firewall existente para alterar o intervalo de endereços IP a partir do qual o banco de dados está acessível.

O que você deve correr?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** sp_set_database_firewall_rule no contexto db1

<details><summary>Explicação</summary>

Você deve executar o sp_set_database_firewall_rule a partir do banco de dados db1 porque o banco de dados é um banco de dados contido, portanto, a alteração da regra de firewall deve ser aplicada diretamente a este banco de dados. O procedimento atualiza a regra existente com base em seu nome.

As regras precisam ser alteradas para o banco de dados db1, não o mestre, portanto, a execução do sp_set_database_firewall_rule do banco de dados mestre está incorreta. Você não precisa alterar a regra do firewall do servidor executando sp_set_firewall_rule. Os usuários que tentam acessar o db1 serão redirecionados diretamente para o banco de dados. A execução do sp_delete_database_firewall_rule do db1 exclui a regra do firewall.

sp_set_database_firewall_rule - Azure SQL Database - Training | Microsoft Learn

Proteger dados em trânsito e em repouso - Treinamento | Microsoft Learn

</details>

---

### Q33 ⚠️

**Questão:**
Você tem um Azure SQL Database chamado db1 que contém uma tabela chamada Vendas.

Você precisa implementar o change tracking para Vendas.

O que você deve fazer primeiro?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Habilitar change tracking para db1.

<details><summary>Explicação</summary>

Você deve ativar o change tracking no nível do banco de dados primeiro, então você pode habilitar o change tracking para cada tabela.

Change tracking deve estar habilitado no nível da banco de dados antes que possa ser habilitado para a tabela de vendas. Criar uma chave mestra na banco de dados master e criar um certificado em db1 não são necessários para change tracking.

Ativar e Desativar Change Tracking - SQL Server - Training | Microsoft Learn

Implementar controles de conformidade para dados confidenciais - Treinamento | Microsoft Learn

</details>

---

### Q34

**Questão:**
Uma organização usa o Azure SQL Database para armazenar informações confidenciais do cliente.

Você precisa limitar a exposição de dados sensíveis em resultados de consulta para usuários não privilegiados, enquanto ainda permite que os usuários autorizados visualizem os dados completos. A solução deve ser transparente para o aplicativo e exigir mudanças mínimas nas consultas existentes.

Qual característica você deve usar?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Dynamic Data Masking

<details><summary>Explicação</summary>

Dynamic Data Masking é a solução mais adequada porque garante que os usuários não autorizados vejam dados mascarados enquanto os usuários autorizados podem visualizar dados desmascarados, atendendo diretamente ao requisito. Always Encrypted aumenta a segurança criptografando dados confidenciais, mas não fornece a capacidade de mascarar dados dinamicamente para usuários não autorizados. Row-level security A auditoria é projetada para rastrear e monitorar o acesso a dados, mas não impede que usuários não autorizados visualizem informações confidenciais.

Explore data classification - Training | Microsoft Learn
Explicar a criptografia de objeto e secure enclaves - Training | Microsoft Learn

</details>

---

### Q35

**Questão:**
Sua organização usa o Microsoft Azure SQL Database para armazenar dados confidenciais de clientes, incluindo informações financeiras e pessoais.

Você precisa classificar e rotular dados sensíveis no banco de dados para conformidade.

Cada resposta correta apresenta parte da solução. Que duas ações você deve realizar?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Classificar colunas e aplicar rótulos de sensibilidade.; Use Data Discovery & Classification.

<details><summary>Explicação</summary>

Para classificar e rotular dados sensíveis para conformidade, é necessário usar o recurso Descoberta e Classificação de Dados para identificar colunas sensíveis e aplicar rótulos de sensibilidade para garantir a classificação adequada. Criptografar colunas de banco de dados com Always Encrypted e habilitar Transparent Data Encryption são medidas de segurança que protegem os dados, mas não atendem aos requisitos específicos de classificação e rotulagem.

Implementar Microsoft Purview - Training | Microsoft Learn
Projetar Azure SQL Database para aplicativos cloud-native - Treinamento | Microsoft Learn

</details>

---

## Monitorar, configurar e otimizar recursos de banco de dados (16)

### Q36

**Questão:**
Você precisa configurar o monitoramento Extended Events para capturar um evento quando um novo banco de dados é adicionado a um Azure SQL Managed Instance.

Qual categoria de evento você deve usar?

**Alternativas:**
- **A.** admin
- **B.** Analítica
- **C.** debug
- **D.** operacional

**✅ Resposta:** D

<details><summary>Explicação</summary>

Eventos operacionais são usados para analisar e diagnosticar um problema ou ocorrência. Eles podem ser usados para acionar ferramentas ou tarefas com base no problema ou ocorrência. Um exemplo de evento operacional é quando um banco de dados é adicionado ou removido do servidor.

Um evento de administrador indica um problema dentro de um conjunto bem definido de ações que um administrador pode tomar, eventos analíticos descrevem as operações do programa e são normalmente usados em investigações de desempenho, e eventos de depuração são usados apenas por desenvolvedores para diagnosticar um problema para depuração.

[SQL Server Extended Events Pacotes - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/extended-events/sql-server-extended-events-packages?view=sql-server-ver16)

[Describe performance monitoring - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/describe-performance-monitoring/)

</details>

---

### Q37

**Questão:**
Você precisa configurar uma linha de base para o monitoramento de desempenho de um Azure SQL Database. 

Qual visão dinâmica de gerenciamento você deve consultar para obter os dados de desempenho da linha de base?

**Alternativas:**
- **A.** sys.dm \_ db \_ esperar \_ estatísticas
- **B.** sys.dm \_ exec \_ consulta \_ estatísticas
- **C.** sys.dm \_ exec \_ sessões \_ esperar \_ estatísticas
- **D.** sys.dm \_ os \_ à espera de tarefas \_

**✅ Resposta:** A

<details><summary>Explicação</summary>

sys.dm \_ db \_ espera \_ estatísticas retorna informações sobre todas as esperas encontradas no banco de dados. Isso permite que você veja os dados de desempenho da linha de base.

sys.dm\_os\_esperando\_Esta visão não armazena nenhuma espera histórica, por isso não deve ser usada para criar a linha de base de desempenho. sys.dm\_exec\_sessões\_Espera\_As estatísticas são usadas para encontrar as estatísticas de espera para uma sessão específica. sys.dm\_exec\_consulta\_Estatísticas retorna estatísticas de desempenho agregado para planos de consulta em cache na Microsoft SQL Server.

[sys.dm_db_wait_stats (Azure SQL Database) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-db-wait-stats-azure-sql-database?view=azuresqldb-current)

[Establish baseline metrics - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/describe-performance-monitoring/4-establish-baseline-metrics?ns-enrollment-type=learningpath&amp;ns-enrollment-id=learn.wwl.monitor-optimize-operational-resources-sql-server)

</details>

---

### Q38

**Questão:**
Você planeja usar Query Store em um Azure SQL Managed Instance. 

Você precisa garantir que você possa forçar manualmente os planos de execução para consultas regredidas. A solução deve seguir o princípio do menor privilégio. 

Para qual função de banco de dados você deve ser atribuído?

**Alternativas:**
- **A.** db \_ accessadmin
- **B.** db \_ ddldadmin
- **C.** proprietário db \_
- **D.** db \_ securityadmin

**✅ Resposta:** C

<details><summary>Explicação</summary>

Como membro da função de banco de dados fixo proprietário do db \_, você pode executar todas as atividades de configuração e manutenção no banco de dados.

Membros do db\_O papel do dddadmin pode executar qualquer DDL Os membros desse papel podem potencialmente elevar seus privilégios manipulando código que pode ser executado sob altos privilégios, e suas ações devem ser monitoradas. Um membro do db\_função de banco de dados fixo accessadmin pode adicionar ou remover o acesso ao banco de dados para Windows logins, grupos Windows e Microsoft SQL Server logins. Membros do db\_O papel do securityadmin pode modificar a associação de funções apenas para funções personalizadas e gerenciar permissões.

[Funções de nível de banco de dados - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/security/authentication-access/database-level-roles?view=sql-server-ver16)

[Configurar autenticação e autorização do banco de dados - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-database-authentication-authorization/)

</details>

---

### Q39

**Questão:**
Você tem um Azure SQL managed instance.

Você descobre um alto número de consultas de bloqueio durante o horário comercial de pico.

Você precisa investigar a causa raiz das consultas de bloqueio.

Quais duas visões dinâmicas de gerenciamento você deve consultar? Cada resposta correta apresenta uma solução completa.

**Alternativas:**
- **A.** sys.dm \_ exec \_ consulta \_ estatísticas
- **B.** sys.dm \_ exec \_ pedidos
- **C.** sys.dm \_ exec \_ sessões \_ esperar \_ estatísticas
- **D.** sys.dm \_ os \_ à espera de tarefas \_

**✅ Resposta:** B, D

<details><summary>Explicação</summary>

Você deve consultar o sys.dm \_ exec \_ requests view para obter a sessão de bloqueio ID informações. Esta visão retorna informações sobre cada pedido que está sendo executado no Microsoft SQL Server. Você também deve usar sys.dm \_ os \_ esperando tarefas \_ porque as informações estão relacionadas às conexões e sessões atuais. Há sessão de bloqueio ID informações que podem ser usadas para identificar a raiz da questão.

O sys.dm\_exec\_sessões\_Espera\_view estatísticas apresenta informações históricas sobre as esperas que ocorreram no servidor, e o sys.dm\_exec\_consulta\_stats view retorna estatísticas de desempenho agregado para planos de consulta em cache em SQL Server.

[sys.dm_exec_requests (Transact-SQL) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql?view=sql-server-ver16)

[sys.dm_os_waiting_tasks (Transact-SQL) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-os-waiting-tasks-transact-sql?view=sql-server-ver16)

[Explore query performance optimization - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/explore-query-performance-optimization/)

</details>

---

### Q40

**Questão:**
Você precisa encontrar todos os índices que têm alta fragmentação.

Quais duas visões dinâmicas de gerenciamento você deve consultar? Cada resposta apresenta uma parte da solução.

**Alternativas:**
- **A.** sys.dm \_ db \_ coluna \_ loja \_ linha \_ grupo \_ física \_ estatísticas
- **B.** sys.dm \_ db \_ índice \_ operacional \_ estatísticas
- **C.** sys.dm \_ db \_ índice \_ estatísticas físicas \_
- **D.** sys.dm \_ db \_ índice \_ uso \_ estatísticas

**✅ Resposta:** A, C

<details><summary>Explicação</summary>

Você deve consultar o sys.dm\_db\_índice\_físico\_view estatísticas, à medida que retorna informações de tamanho e fragmentação para os dados e índices da tabela especificada ou view na Microsoft SQL Server. Para um índice, uma linha é devolvida para cada nível da árvore B em cada partição. Para um heap, uma linha é retornada para o IN\_ROW\_Unidade de alocação DATA de cada partição. Você também deve consultar o sys.dm\_db\_coluna\_loja\_linha\_grupo\_físico\_view stats view porque fornece informações atuais de nível de grupo de linha sobre todas as columnstore Com base nas informações, você pode calcular a fragmentação do índice.

O sys.dm\_db\_índice\_utilização\_estatísticas ver retorna contagens de diferentes tipos de operações de índice e o tempo em que cada tipo de operação foi realizada pela última vez, e o sys.dm\_db\_índice\_operacional\_A visualização de estatísticas retorna a atividade atual de nível inferior de E/S, bloqueio, travamento e método de acesso para cada partição de uma tabela ou índice no banco de dados.

[sys.dm_db_index_physical_stats (Transact-SQL) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-db-index-physical-stats-transact-sql?view=sql-server-ver16)

[Configurar bancos de dados para otimizar o desempenho - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-databases-for-optimal-performance/)

</details>

---

### Q41 ⚠️

**Questão:**
Você tem um Azure SQL Managed Instance que contém bancos de dados.

Uma das bancos de dados experimenta bloqueio excessivo.

Você precisa identificar qual sessão está causando o bloqueio.

Qual visão de gestão dinâmica você deve consultar?

**Alternativas:**
- **A.** sys.dm \_ exec \_ consulta \_ estatísticas
- **B.** sys.dm \_ exec \_ pedidos
- **C.** sys.dm \_ exec \_ sessão \_ esperar \_ estatísticas
- **D.** sys.dm \_ exec \_ sessões

**✅ Resposta:** B

<details><summary>Explicação</summary>

sys.dm \_ exec \_ requests devolve informações sobre cada pedido que está sendo executado no Microsoft SQL Server. Uma das colunas expostas por esta visão é o bloqueio da sessão \_ \_ id.

sys.dm \_ exec \_ sessões retorna uma linha por sessão autenticada em SQL Server. Esta visão não retorna informações sobre problemas de bloqueio.

sys.dm \_ exec \_ consulta \_ estatísticas retorna estatísticas de desempenho agregado para planos em cache em SQL Server.

sys.dm \_ exec \_ sessão \_ espera retorna informações sobre todas as esperas encontradas por threads que são executados para cada sessão.

[sys.dm_exec_requests (Transact-SQL) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql?view=sql-server-ver16)

</details>

---

### Q42

**Questão:**
Você tem um Azure SQL database.

Você precisa solucionar problemas de bloqueio e de desempenho.

Qual característica você deve usar?

**Alternativas:**
- **A.** e Azure SQL Database livro-razão
- **B.** Extended Events
- **C.** Microsoft Defender for SQL
- **D.** Query Store

**✅ Resposta:** B

<details><summary>Explicação</summary>

O Extended Events pode capturar métricas de bloqueio e bloqueio que podem ser usadas para investigar e solucionar problemas de desempenho.

O Query Store não captura métricas de bloqueio e bloqueio, o Azure SQL Defender apresenta vulnerabilidades no banco de dados e um livro-razão Azure SQL Database captura incrementalmente o estado do banco de dados.

[Explorar extended events - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/describe-performance-monitoring/5-explore-extended-events)

</details>

---

### Q43 ⚠️

**Questão:**
Você tem um Azure SQL managed instance que hospeda duas bancos de dados chamadas db1 e db2.

Você precisa limitar o uso de sessões db1 a um máximo de 30% do CPU disponível.

Quais ações você deve executar? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** Configure o grau máximo de paralelismo.
- **B.** Crie um pool de recursos.
- **C.** Crie um grupo de carga de trabalho.
- **D.** Crie uma sessão Extended Events.
- **E.** Habilitar ajuste automático de banco de dados.
- **F.** Habilitar Resource Governor.

**✅ Resposta:** B, C, F

<details><summary>Explicação</summary>

Resource Governor precisa ser ativado, um pool de recursos precisa ser criado e um grupo de carga de trabalho precisa ser criado para limitar o db1 a um máximo de 30%.

A Sintonização Automática de Banco de Dados é usada para notificá-lo sempre que um problema de desempenho potencial for detectado e permitir que você aplique ações corretivas. Extended Events é um sistema de monitoramento de desempenho leve. O Max Degree of Parallelism limita o número de processadores a serem usados em uma execução de plano paralelo.

[Resource Governor - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/resource-governor/resource-governor?view=sql-server-ver15)

[Configurar SQL Server resources for optimal performance - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-sql-server-resources-optimal-performance/)

</details>

---

### Q44

**Questão:**
Você tem um Azure SQL Managed Instance que contém um banco de dados chamado DB1. O nível de compatibilidade do banco de dados para DB1 é 150.

As consultas que usam variáveis de tabela estão com mau desempenho. Você identifica que o problema é causado por estimativas incorretas de cardinalidade.

Você precisa configurar uma opção para garantir que o mecanismo de banco de dados defina a estimativa de cardinalidade quando a consulta é executada e a contagem de linhas real é conhecida.

Qual opção você deve configurar?

**Alternativas:**
- **A.** DEFERRED\_COMPILATION\_TV
- **B.** ELEVATE \_ ONLINE
- **C.** INTERLEAVED\_EXECUTION\_TVF
- **D.** TSQL \_ SCALAR \_ UDF \_ INLINING

**✅ Resposta:** A

<details><summary>Explicação</summary>

Você deve usar a opção DEFERRED \_ COMPILATION \_ TV porque permite ou desabilita a compilação de variáveis de tabela.

A opção INTERLEAVED \_ EXECUTION \_ TVF permite ou desativa a execução intercalada para funções com valor de tabela de vários estados.

A opção TSQL \_ SCALAR \_ UDF \_ INLINING permite ou desativa o T-SQL Scalar UDF inlining.

A opção ELEVATE \_ ONLINE faz com que o motor eleve automaticamente as operações suportadas online.

[ALTER DATABASE SCOPED CONFIGURATION - SQL Server (Transact-SQL) - Training | Microsoft Learn](https://learn.microsoft.com/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql?view=sql-server-ver16)

</details>

---

### Q45

**Questão:**
Você tem um Azure SQL Database que usa Query Store.

Você percebe um desempenho ruim para um relatório causado pelo plano de execução e pelos parâmetros do relatório.

Você precisa melhorar o desempenho do relatório. A solução deve minimizar o esforço administrativo.

O que você deve fazer?

**Alternativas:**
- **A.** Desativar parameter sniffing.
- **B.** Habilitar snapshot isolation.
- **C.** Execute o comando DBCC FREEPROCCACHE.
- **D.** Forçar o plano de execução mais eficiente.

**✅ Resposta:** D

<details><summary>Explicação</summary>

Forçar o plano que é o mais eficiente para a maioria das execuções melhorará o desempenho do relatório.

A desativação do parameter sniffing ignorará quaisquer parâmetros e não escolherá o plano mais eficiente. O DBCC FREEPROCCACHE removerá todos os planos do cache.

Snapshot isolation é usado para permitir a execução simultânea de transações.

[Monitorando o desempenho usando o Query Store - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store?view=sql-server-ver16#Regressed)

[Evaluate performance improvements - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/evaluate-performance-improvements/)

</details>

---

### Q46 ⚠️

**Questão:**
Você planeja configurar o monitoramento Extended Events para coletar informações sobre consultas.

Você precisa adicionar informações sobre o session_id a todos os eventos.

O que você deve configurar para a sessão estendida do evento?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Ação

<details><summary>Explicação</summary>

Uma ação é uma resposta programática ou uma série de respostas a um evento. As ações estão vinculadas a um evento e cada evento pode ter um conjunto único de ações. As ações são campos globais. A configuração de uma ação permite adicionar as informações necessárias a todos os eventos.

Os predicados são um conjunto de regras lógicas usadas para avaliar eventos quando eles são processados, e um pacote é um recipiente para objetos Microsoft SQL Server Extended Events.

SQL Server Extended Events Pacotes - SQL Server - Training | Microsoft Learn

Describe performance monitoring - Training | Microsoft Learn

</details>

---

### Q47 ⚠️

**Questão:**
Você tem uma instância SQL Server 2019.

Você precisa habilitar Extended Events para monitorar deadlocks.

Qual declaração T-SQL você deve executar?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** 
```sql
CREATE EVENT SESSION deadlocks
ON SERVER
ADD EVENT sqlserver.lock_deadlock(ACTION(sqlserver.sql_text)),
ADD EVENT sqlserver.lock_deadlock_chain(ACTION(sqlserver.sql_text))
ADD TARGET package0.event_file
(
SET filename=C:\traces\deadlocks.xel'
)
WITH (MAX_MEMORY=4MB, MAX_DISPATCH_LATENCY = 3 SECONDS);
GO
```

<details><summary>Explicação</summary>

O alvo event_file permitirá a solução de problemas históricos.

O alvo contador de eventos conta quantas vezes cada evento especificado ocorre e o alvo do histograma conta ocorrências de diferentes tipos de itens.

Alvos para Extended Events E- SQL Server - SQL Server - Training | Microsoft Learn

Explorar extended events - Training | Microsoft Learn

</details>

---

### Q48 ⚠️

**Questão:**
Você tem uma instância do Microsoft SQL Server instalada em um Azure virtual machine chamado SQLVM1.

Você precisa analisar os contadores de motores SQL Server, como cache de buffer e solicitações de lote por segundo.

O que você deve usar para coletar essas métricas?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Windows Performance Monitor (PerfMon) em SQLVM1

<details><summary>Explicação</summary>

Objectivo:

3.1 Monitorar a atividade e o desempenho dos recursos

O que este item testa:

Determinar fontes para métricas de desempenho

Leitura adicional:

Descrever ferramentas de monitoramento de desempenho
Resumo
desempenho do banco de dados do monitor
Monitor e ajuste de desempenho em Azure SQL Database e Azure SQL Managed Instance
Monitor Azure SQL Managed Instance
Racionale:

Windows Performance Monitor (PerfMon) em SQLVM1 pode coletar SQL Server contadores de motores, como a taxa de sucesso do cache de buffer e solicitações por segundo, porque ele acessa SQL Server objetos de desempenho expostos através do subsistema de contador de desempenho do Windows no host da máquina virtual. Azure Monitor metrics para a Microsoft.Compute/virtualMachines recursos expor apenas métricas de infraestrutura, tais como CPUuso de disco e rede, em vez de SQL Server Contadores de motores. Query Performance Insight aplica-se a Azure SQL Database em vez de SQL Server O registro de atividades do Azure registra operações de controle de avião no Azure, em vez de métricas de desempenho do motor de banco de dados.

</details>

---

### Q49

**Questão:**
Sua organização usa Azure SQL Database com Query Store habilitado para monitorar o desempenho da consulta.

Você precisa configurar o Query Store para capturar consultas com consumo significativo de recursos, minimizando o uso de armazenamento.

Quais ações você deve realizar? Cada resposta correta apresenta parte da solução. Escolha 2.

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Ativar o modo de limpeza baseado em tamanho.; Defina o modo de captura de consulta como automático.

<details><summary>Explicação</summary>

Desativar o Query Store evita o monitoramento de consultas, o que contradiz o objetivo de capturar consultas com consumo significativo de recursos. A ativação do modo de limpeza baseado em tamanho garante que os dados menos relevantes sejam removidos automaticamente quando os limites de armazenamento são abordados, ajudando a minimizar o uso de armazenamento, mantendo informações críticas. Definir o modo de captura de consultas para todas as consultas resulta em coleta excessiva de dados, o que aumenta o uso de armazenamento e não se alinha com o objetivo de minimizar o uso de armazenamento.

Desempenho do monitor usando o Query Store

</details>

---

### Q50 ⚠️

**Questão:**
Você tem um banco de dados em um Azure SQL Managed Instance.

Você precisa controlar a configuração MAXDOP no nível da carga de trabalho. A solução deve minimizar o esforço administrativo e de desenvolvimento.

O que você deve fazer?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Em Resource Governor, defina a configuração do grupo de carga de trabalho MAX_DOP.

<details><summary>Explicação</summary>

Configure a configuração do grupo de carga de trabalho MAX_DOP no Resource Governor para poder controlar o grau de paralelismo por carga de trabalho definida. Esta opção pode ser definida na declaração CREATE WORKLOAD GROUP.

A declaração ALTER DATABASE SCOPED CONFIGURATION gerencia as configurações por banco de dados e não por carga de trabalho. Usando a dica MAXDOP no nível de consulta requer alterações de código, e você não pode controlar a carga de trabalho. Usando o procedimento armazenado sp_configure altera as configurações no nível do servidor, e você não pode gerenciar a carga de trabalho.

ALTER DATABASE SCOPED CONFIGURATION - SQL Server (Transact-SQL) - Training | Microsoft Learn

CREATE WORKLOAD GROUP (Transact-SQL) - SQL Server - Training | Microsoft Learn

Configurar bancos de dados para otimizar o desempenho - Treinamento | Microsoft Learn

</details>

---

### Q51

**Questão:**
Você executa o comando DBCC CHECKDB para um banco de dados de produção. O comando mostra erros em vários nonclustered indexes.

Você precisa reparar os índices e garantir a integridade do banco de dados após a operação ter sido concluída.

Qual parâmetro você deve especificar no comando DBCC CHECKDB?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** REPAIR_REBUILD

<details><summary>Explicação</summary>

Adicionar a opção REPAIR_REBUILD ao comando DBCC CHECKDB executa reparos que não têm possibilidade de perda de dados. Esta opção pode incluir reparos rápidos, como reparar linhas ausentes no nonclustered indexes e reparos mais demorados, como reconstruir um índice.

A opção REPAIR_FAST mantém a sintaxe apenas para compatibilidade com versões anteriores. Nenhuma ação de reparo é executada. A opção REPAIR_ALLOW_DATA_LOSS pode introduzir alguma perda de dados no banco de dados.

DBCC CHECKDB (Transact-SQL) - SQL Server - Training | Microsoft Learn

Configurar bancos de dados para otimizar o desempenho - Treinamento | Microsoft Learn

</details>

---

## Configurar e gerenciar a automação de tarefas (13)

### Q52

**Questão:**
Você tem um Azure SQL managed instance.

Você precisa garantir que você receba uma notificação quando um SQL Server Agent job falhar.

Quais ações você deve executar? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** Crie uma conta Database Mail.
- **B.** Crie um perfil Database Mail.
- **C.** Crie um SQL Server Agent proxy.
- **D.** Crie uma credencial SQL Server.
- **E.** Habilitar Database Mail.

**✅ Resposta:** A, B, E

<details><summary>Explicação</summary>

Para garantir que você receba uma notificação quando um SQL Server Agent job falhar, o Database Mail precisa ser ativado, uma conta Database Mail precisa ser criada e um perfil Database Mail precisa ser criado.

Uma credencial SQL Server e um SQL Server Agent proxy não são necessários.

[Describe task status notifications - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/schedule-tasks-using-sql-server-agent/3-describe-task-status-notifications?ns-enrollment-type=learningpath&amp;ns-enrollment-id=learn.wwl.automate-tasks-sql-server)[Configurar SQL Server Agent para usar Database Mail - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/database-mail/configure-sql-server-agent-mail-to-use-database-mail?view=sql-server-ver16)

</details>

---

### Q53

**Questão:**
Você implanta uma instância SQL Server on Azure Virtual Machines. 

Você precisa garantir que o automated patching esteja ativado.

Quais dois cmdlets você deve executar? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** New-AzVMConfig
- **B.** New-AzVMSqlServerAutoPatchingConfig
- **C.** Set-AzVMSqlServerExtension
- **D.** Set-AzVMCustomScriptExtension

**✅ Resposta:** B, C

<details><summary>Explicação</summary>

O cmdlet New-AzVMSqlServerAutoPatchingConfig permite que você crie um objeto de configuração para automatic patching em uma máquina virtual, e o cmdlet Set-AzVMSqlServerExtension estabelece uma janela de manutenção para uma instância SQL Server on Azure Virtual Machines. Atualizações automatizadas só podem ser instaladas durante esta janela de manutenção.

Set-AzVMCustomScriptExtensionUpdate  Adiciona uma extensão de script personalizado a uma máquina virtual., e o New-AzVMConfig O cmdlet cria um objeto de máquina virtual local configurável para o Azure.

[Set-AzVMSqlServerExtension (Az.Compute) - Training | Microsoft Learn](https://learn.microsoft.com/powershell/module/az.compute/set-azvmsqlserverextension?view=azps-9.2.0)

[Automatizar a implantação de recursos de banco de dados - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-automatic-deployment-azure-sql-database/)

</details>

---

### Q54

**Questão:**
Você tem um arquivo bacpac de um banco de dados Microsoft SQL Server existente. Você copia o arquivo para uma conta Azure Storage. 

Você precisa aplicar o arquivo bacpac a um Azure SQL Database existente.

Qual comando você deve executar?

**Alternativas:**
- **A.** sql db create
- **B.** sql db import
- **C.** az sql db restaurar
- **D.** az sql db update

**✅ Resposta:** B

<details><summary>Explicação</summary>

Você pode usar o comando az sql db import para importar o banco de dados do arquivo bacpac. O banco de dados deve ser criado antes que o arquivo bacpac seja importado.

O comando az sql db restaure cria um novo banco de dados a partir do backup, e o comando az sQL db update atualiza o banco de dados existente.

[az sql db - Training | Microsoft Learn](https://learn.microsoft.com/cli/azure/sql/db?view=azure-cli-latest#az-sql-db-create)

[Automatizar a implantação de recursos de banco de dados - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-automatic-deployment-azure-sql-database/)

</details>

---

### Q55

**Questão:**
Você tem uma assinatura do Azure.

Você planeja implantar 100 Azure SQL Databases.

Você precisa automatizar a implantação. A solução deve ser parametrizável, repetível e ser capaz de ser versionada no controle da fonte.

Quais componentes você pode usar? Cada resposta correta apresenta uma solução completa.

**Alternativas:**
- **A.** Azure Cloud Shell
- **B.** Azure Policy
- **C.** scripts Azure PowerShell
- **D.** Modelos Azure Resource Manager (ARM)
- **E.** Modelos Bicep

**✅ Resposta:** C, D, E

<details><summary>Explicação</summary>

Modelos ARM, modelos Bicep e scripts PowerShell podem ser parametrizados e versionados.

O Cloud Shell é um shell interativo, autenticado e acessível ao navegador, e o Azure Policy ajuda a impor padrões organizacionais e avaliar a conformidade em escala.

[Automatize a implantação usando Azure Resource Manager templates e Bicep - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-automatic-deployment-azure-sql-database/3-automate-deployment-using-azure-resource-manager-bicep?ns-enrollment-type=learningpath&ns-enrollment-id=learn.wwl.automate-tasks-sql-server)

</details>

---

### Q56

**Questão:**
Você tem um Elastic Job agent que executará tarefas em um Azure SQL Database elastic pool. 

Você cria um elastic job. 

Você precisa garantir que o job funcione a cada hora.

Qual cmdlet você deve executar?

**Alternativas:**
- **A.** Set-AzSqlDatabase
- **B.** Set-AzSqlElasticJob
- **C.** Set-AzSqlElasticJobStep
- **D.** Set-AzSqlElasticPool

**✅ Resposta:** B

<details><summary>Explicação</summary>

Usando o cmdlet Set-AzSqlElasticJob atualiza as propriedades elastic job. Você pode definir o intervalo, bem como a data de início e fim do job.

O cmdlet Set-AzSqlElasticJobStep atualiza as etapas elastic job. Você não pode modificar o cronograma no nível do passo. O cmdlet Seth- Set-AzSqlElasticPool modifica as propriedades de um Azure SQL Database elastic pool, e o cmdlet Set - Set-AzSqlDatabase define as propriedades de um banco de dados ou move um banco de dados existente em um elastic pool.

[Set-AzSqlElasticJob (Az.Sql) - Training | Microsoft Learn](https://learn.microsoft.com/powershell/module/az.sql/set-azsqlelasticjob?view=azps-9.2.0)

[Manage Azure PaaS tasks using automation - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/manage-azure-paas-resources-using-automated-methods/)

</details>

---

### Q57

**Questão:**
Você tem uma assinatura do Azure que inclui 20 servidores lógicos Azure SQL.

Você precisa automatizar tarefas de gerenciamento usando um Azure Automation runbook.

O que você deve fazer primeiro?

**Alternativas:**
- **A.** Crie uma conta Azure Automation.
- **B.** Crie uma credencial Azure Automation.
- **C.** Crie um calendário Azure Automation runbook.
- **D.** Implantar um Azure Automation híbrido runbook corredor.

**✅ Resposta:** A

<details><summary>Explicação</summary>

Para automatizar tarefas de gerenciamento usando um Azure Automation runbook, primeiro você precisa criar uma conta Azure Automation.

Você não pode criar uma credencial ou um cronograma sem primeiro criar uma conta Azure Automation, e você não precisa de um corredor híbrido runbook.

[Construa uma automação runbook - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/manage-azure-paas-resources-using-automated-methods/4-build-automation-runbook)

</details>

---

### Q58

**Questão:**
Sua empresa usa o Azure SQL Database elastic pools para gerenciar recursos para bancos de dados de inquilinos.  
  
Você usa o elastic jobs e precisa garantir que um job agendado tenha como alvo bancos de dados específicos dentro da piscina.  
  
O que você deve usar?

**Alternativas:**
- **A.** Criar separado elastic pools.
- **B.** Defina um target group.
- **C.** Execute manualmente o job.
- **D.** Passe o job em toda a piscina.

**✅ Resposta:** B

<details><summary>Explicação</summary>

A definição de um target group permite um controle preciso sobre a execução do job, incluindo ou excluindo bancos de dados específicos, tornando-o a solução mais eficaz para esse cenário. A criação de elastic pools separado aumenta desnecessariamente a complexidade e o custo, o que não está alinhado com o objetivo de gerenciamento eficiente de recursos. A execução manual do job é ineficiente e inadequada para tarefas agendadas ou recorrentes. A execução do job em toda a piscina não atende à necessidade de atingir bancos de dados específicos, pois não possui a seletividade necessária.  
  
[Explorar Elastic jobs - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/manage-azure-paas-resources-using-automated-methods/2-explore-elastic-jobs)  
[Projetar Azure SQL Database para aplicativos cloud-native - Treinamento | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/prepare-to-maintain-sql-databases-azure/4-design-azure-sql-database-for-cloud-native-applications)

</details>

---

### Q59

**Questão:**
Sua empresa tem vários Azure SQL Databases em diferentes regiões.  
  
Você precisa automatizar a execução de declarações T-SQL em todos os bancos de dados.  
  
Quais ações você deve executar? Cada resposta correta apresenta parte da solução. Escolha 3.

**Alternativas:**
- **A.** Crie um elastic job agent.
- **B.** Defina os passos job.
- **C.** Execute as instruções T-SQL manualmente.
- **D.** Agende o elastic job.
- **E.** Use gráfico runbooks.

**✅ Resposta:** A, B, D

<details><summary>Explicação</summary>

Criando um elastic job agent é essencial para automatizar a execução de T-SQL declarações em vários bancos de dados, abordando a necessidade de escalabilidade e eficiência. Definindo job As etapas garantem que as tarefas a serem executadas sejam claramente especificadas, tornando o processo de automação bem estruturado. elastic job garante que as tarefas são executadas automaticamente em intervalos regulares, alinhando-se com o objetivo de automação. Executar T-SQL declarações contradiz manualmente o objetivo de automação, pois requer esforço humano e falta escalabilidade. Gráfico runbooks não são projetados para esse fim, tornando-os inadequados para o cenário.  
  
[Entender Azure Automation - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/manage-azure-paas-resources-using-automated-methods/3-understand-azure-automation)  
[Automatize a implantação usando PowerShell - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/configure-automatic-deployment-azure-sql-database/4-automate-deployment-using-powershell)

</details>

---

### Q60

**Questão:**
Sua organização usa Azure SQL Database para aplicações críticas. O desempenho do banco de dados é afetado por índices desatualizados e dados obsoletos.  
  
Você precisa automatizar tarefas de manutenção de banco de dados para melhorar o desempenho.  
  
Quais ações você deve executar? Cada resposta correta apresenta parte da solução. Escolha 3.

**Alternativas:**
- **A.** Crie um PowerShell runbook.
- **B.** Habilitar managed identities.
- **C.** Acione manualmente o runbook.
- **D.** Agende o runbook.
- **E.** Use Logic Apps.

**✅ Resposta:** A, B, D

<details><summary>Explicação</summary>

Criando a PowerShell runbook é essencial para automatizar tarefas de manutenção de banco de dados, abordando a necessidade de um melhor desempenho. runbook essas tarefas são executadas regularmente sem intervenção manual, alinhando-se com o objetivo de automação. managed identities fornece autenticação segura e simplificada para acessar os recursos do Azure, o que é fundamental para manter a segurança durante a automação. runbook contradiz o objetivo de automação, pois requer intervenção humana. Enquanto Logic Apps podem automatizar fluxos de trabalho, eles não são a escolha ideal para esta tarefa específica, tornando-os menos adequados para o cenário.  
  
[Entender Azure Automation - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/manage-azure-paas-resources-using-automated-methods/3-understand-azure-automation)  
[Automatize a implantação usando PowerShell - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/configure-automatic-deployment-azure-sql-database/4-automate-deployment-using-powershell)  
[Adicionar autenticação para gerenciar recursos do Azure](https://learn.microsoft.com/en-us/azure/automation/learn/automation-tutorial-runbook-textual#add-authentication-to-manage-azure-resources)

</details>

---

### Q61

**Questão:**
Você tem uma assinatura do Azure que contém um grupo de recursos chamado RG1.

Você precisa criar um Azure SQL Managed Instance no RG1 usando PowerShell.

Qual cmdlet você deve executar?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** New-AzSqlInstance

<details><summary>Explicação</summary>

Usando o cmdlet New-AzSqlInstance, você pode criar um novo SQL managed instance.

New-AzSQLServer cria um servidor Microsoft SQL. New-AzSqlDatabase cria um banco do Azure SQL Database. Set-AzSqlInstance define as propriedades de uma SQL managed instance e Set-AzSqlServer define as propriedades de um servidor Microsoft SQL.

New-AzSqlInstance (Az.Sql) - Training | Microsoft Learn

Automatizar a implantação de recursos de banco de dados - Treinamento | Microsoft Learn

</details>

---

### Q62

**Questão:**
Você tem uma assinatura do Azure que contém um grupo de recursos chamado RG1. O RG1 está hospedado na região West US Azure.

Você precisa criar um novo servidor Azure SQL chamado srv1.

Qual comando Azure CLI você deve executar?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** az sql server create -l westus -g rg1 -n srv1 -u adminuser -p adminpassword

<details><summary>Explicação</summary>

Usando o comando az sql server create -l westus -g rg1-n srv1 -u adminuser -p adminpassword criará um novo servidor Azure SQL.

O az sql db create -g rg1-s srv1-n mydb --service-objective S0 O comando cria um novo Azure SQL Database, o az sql mi create -g rg1-n srv1-l westus -i -u adminuser -p adminpassword comando irá criar um novo Azure SQL Managed Instance, e o az sql elástico-pool criar -g rg1-s srv1-n db1 O comando criará um novo Azure SQL Database elastic pool.

az sql server | Microsoft Learn

Automatizar a implantação de recursos de banco de dados - Treinamento | Microsoft Learn

</details>

---

### Q63

**Questão:**
Você planeja implantar vários Azure SQL Databases para uma nova aplicação. A automação é necessária para consistência e eficiência.

Você precisa determinar as ferramentas e métodos para automatizar a implantação desses bancos de dados.

Cada resposta correta apresenta parte da solução. Que três ações você deve tomar?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Adicione scripts de implantação a um pipeline Azure DevOps.; Defina configurações de banco de dados usando modelos ARM.; Execute scripts de implantação usando o módulo Az.Sql PowerShell.

<details><summary>Explicação</summary>

Automatização da implantação de Azure SQL Databases requer ferramentas e métodos que garantam consistência e eficiência. Adicionando scripts de implantação a um Azure DevOps O pipeline facilita processos automatizados e suporta a entrega contínua. Definindo configurações de banco de dados usando ARM Os modelos de modelos fornecem uma abordagem declarativa para a implantação de recursos do Azure, garantindo repetibilidade e alinhamento com os requisitos de automação. Az.Sql PowerShell utiliza cmdlets adaptados para gerenciar Azure SQL O uso de uma ferramenta de terceiros não integrada com o Azure introduz complexidade desnecessária e não consegue alavancar as capacidades nativas. Configuração manual de cada banco de dados no Azure portal é demorado e propenso a erro humano, contradizendo o requisito de automação. Implantando bancos de dados interativamente usando SSMS É principalmente adequado para gerenciar SQL Server não suporta as instâncias de automação necessárias para Azure SQL Database implantações.

Automatize a implantação usando PowerShell - Training | Microsoft Learn
Explorar um único SQL database - Training | Microsoft Learn

</details>

---

### Q64

**Questão:**
Você tem uma assinatura do Azure. A assinatura contém vários Azure SQL Databases que suportam aplicativos departamentais.

Você planeja usar o elastic jobs para executar periodicamente tarefas de manutenção baseadas no T-SQL em todos os bancos de dados.

O que você deve fazer primeiro?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Crie um Azure SQL Database.

<details><summary>Explicação</summary>

Você precisa criar um Azure SQL Database para manter a configuração do elastic job.

O Elastic Job agent pode ser configurado assim que o banco de dados e a configuração do elastic job estiverem preparados.

Você não pode criar alvos Elastic Job até ter um agente.

Você não precisa criar um servidor elastic pool para esta tarefa.

Job visão geral de automação com Elastic Jobs - Azure SQL Database - Training | Microsoft Learn

</details>

---

## Planejar e configurar um ambiente de alta disponibilidade e recuperação de desastres (HA/DR) (15)

### Q65

**Questão:**
Você precisa recomendar uma estratégia de alta disponibilidade e recuperação de desastres (HA/DR) para uma instância SQL Server on Azure Virtual Machines. A solução deve atender aos seguintes requisitos:

**Alternativas:**
- **A.** Todos os bancos de dados, incluindo bancos de dados do sistema, devem ser protegidos.
- **B.** SQL Server Agent jobs deve ser protegido.
- **C.** O tempo de inatividade deve ser minimizado.
- **D.** O processo failover deve ser feito automaticamente.

O que você deve incluir na recomendação?
- **E.** Always On availability group
- **F.** Always On failover cluster instance
- **G.** log shipping
- **H.** transactional replication

**✅ Resposta:** F

<details><summary>Explicação</summary>

Você deve recomendar um Always On failover cluster instance porque esse recurso protege toda a instância. Uma instância SQL Server on Azure Virtual Machines é toda a instalação do Microsoft SQL Server, como binários e todos os objetos dentro da instância, incluindo logins, SQL Server Agent jobs e bancos de dados. A proteção em nível de instância significa que toda a instância é contabilizada no recurso de disponibilidade.

An Always On availability group funciona no nível do banco de dados em vez do nível da instância. Transactional replication não pode replicar bancos de dados do sistema, e nem transactional replication nem log shipping permitir que você execute automatic failover.

[Always On failover cluster instances - SQL Server Always On - Training | Microsoft Learn](https://learn.microsoft.com/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server?view=sql-server-ver16)

[Describe high availability and disaster recovery strategies - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/describe-high-availability-disaster-recovery-strategies/)

</details>

---

### Q66

**Questão:**
Você tem 100 servidores on-premises Microsoft SQL.

Você planeja migrar os servidores para SQL Server on Azure Virtual Machines.

Você precisa recomendar uma solução de alta disponibilidade que manterá a disponibilidade se um datacenter falhar.

O que você deve incluir na recomendação?

**Alternativas:**
- **A.** conjuntos de disponibilidade
- **B.** availability zones
- **C.** Azure Backup
- **D.** Azure Site Recovery

**✅ Resposta:** B

<details><summary>Explicação</summary>

Você deve recomendar o availability zones, pois eles são responsáveis por falhas no nível do datacenter no Azure.

Os conjuntos de disponibilidade fornecem tempo de atividade contra a manutenção relacionada ao Azure e pontos únicos de falha em um único datacenter, o Azure Site Recovery replica cargas de trabalho executadas em máquinas físicas e virtuais de um local primário para um local secundário e o Azure Backup é usado para fornecer backups independente e isolado para se proteger contra a destruição não intencional dos dados em suas máquinas virtuais.

[Descrever recursos de alta disponibilidade e recuperação de desastres do Azure para Azure Virtual Machines - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/describe-high-availability-disaster-recovery-strategies/4-describe-azure-high-availability-disaster-recovery-features)

</details>

---

### Q67 ⚠️

**Questão:**
Você precisa recomendar uma solução de alta disponibilidade e recuperação de desastres (HADR) para um Azure SQL managed instance chamado SQLMI1. A solução deve atender aos seguintes requisitos:

**Alternativas:**
- **A.** Suporte automatic failover entre regiões do Azure.
- **B.** Mantenha a banco de dados sincronizada entre regiões.
- **C.** Suporte failback para a região primária.
- **D.** Seja escalável e rentável.

O que você deve incluir na recomendação?
- **E.** active geo-replication
- **F.** Always On availability groups
- **G.** failover groups
- **H.** transactional replication

**✅ Resposta:** G

<details><summary>Explicação</summary>

Failover groups são a melhor escolha, porque eles são projetados para Azure SQL Managed Instance, apoiando a replicação síncrona dentro de uma região e replicação assíncrona entre regiões pareadas, juntamente com automatic failover e failback. Always On availability groups são apenas para on-premises : Azure VMs e exigir Windows Server Failover Clustering, que não está disponível para PaaS. Active geo-replication aplica-se a Azure SQL Database, não gerenciadas instâncias, e falta automatic failover. Transactional replication destina-se a distribuição de dados ou relatórios e não fornece sincronização completa ou automatic failover.  
  
[Explore auto-failover groups para Azure SQL Database e Azure SQL Managed Instance](https://learn.microsoft.com/en-us/training/modules/explore-iaas-paas-platform-tools-for-high-availability-disaster-recovery/6-explore-auto-failover-groups)

</details>

---

### Q68

**Questão:**
Você tem um availability group em uma instância SQL Server on Azure Virtual Machines. 

Você precisa adicionar um banco de dados existente para o availability group. 

Você executa um full backup do banco de dados no nó primário do availability group. 

Qual comando de restauração você deve executar na réplica secundária para garantir que você possa se juntar ao banco de dados para o availability group?

**Alternativas:**
- **A.** RESTORE WITH NORECOVERY
- **B.** RESTORE WITH PARTIAL
- **C.** RESTORE WITH RECOVERY
- **D.** RESTORE WITH STANDBY

**✅ Resposta:** A

<details><summary>Explicação</summary>

Você precisa restaurar o banco de dados usando a opção NORECOVERY na réplica secundária antes de poder juntar o banco de dados ao availability group. 

Restaurar o banco de dados usando as opções RECOVERY e STANDBY não permite que ele se junte ao availability group. Restaurar o banco usando o PARTIAL é usado se você precisar executar uma restauração fragmentada.

[Preparar um banco de dados secundário para um availability group - SQL Server Always On - Training | Microsoft Learn](https://learn.microsoft.com/sql/database-engine/availability-groups/windows/manually-prepare-a-secondary-database-for-an-availability-group-sql-server?view=sql-server-ver16)

[Fazer backup e restore de bancos de dados - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/backup-restore-databases/)

</details>

---

### Q69

**Questão:**
Você tem uma instância SQL Server on Azure Virtual Machines. 

Durante uma tarefa de manutenção semanal, você descobre que um banco de dados chamado db1 tem erros de consistência. 

Você planeja restaurar o banco de dados do backups.

Qual comando você deve executar antes de executar o comando RESTORE?

**Alternativas:**
- **A.** ALTER DATABASE db1 SET EMERGENCY
- **B.** ALTER DATABASE db1 SET OFFLINE
- **C.** ALTER DATABASE db1 SET RESTRICTED \_ USUÁRIO
- **D.** ALTER DATABASE db1 SET SINGLE \_ USUÁRIO

**✅ Resposta:** D

<details><summary>Explicação</summary>

Você deve executar o comando ALTER DATABASE db1 SET SINGLE \_ USER, pois apenas um usuário de cada vez pode acessar o banco de dados. Se você especificar SINGLE \_ USER e outro usuário se conectar ao banco de dados, o comando ALTER DATABASE será bloqueado até que todos os usuários se desconectem do banco de dados especificado. Este é o comando que deve ser executado antes do comando RESTORE.

A execução do comando ALTER DATABASE db1 SET EMERGENCY não funcionaria porque o banco de dados está marcado como READ \_ SOMENTE, o registro está desativado e o acesso é limitado aos membros da função de servidor fixo sysadmin.

Ao executar ALTER DATABASE db1 SET OFFLINE, o banco de dados será desligado e você não pode executar a operação de restauração. Você não pode modificar um banco de dados enquanto ele estiver offline.

A execução do comando ALTER DATABASE db1 SET RESTRICTED \_ USER permite que apenas os membros do db \_ proprietário fixo banco de dados papel dbcreator, e fixo servidor funções para se conectar ao banco de dados.

[ALTER DATABASE SET Opções (Transact-SQL) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-set-options?view=sql-server-ver16)

[Fazer backup e restore de bancos de dados - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/backup-restore-databases/)

</details>

---

### Q70

**Questão:**
Você tem um Azure SQL Database chamado db1. 

Você precisa configurar uma estratégia backup para garantir que o db1 tenha o backups retido por seis anos.

O que você deve configurar?

**Alternativas:**
- **A.** frequência differential backup
- **B.** long-term retention
- **C.** point-in-time-restaurar

**✅ Resposta:** B

<details><summary>Explicação</summary>

A política long-term retention permite especificar quantas semanas, meses e anos um backup deve ser mantido. O tempo máximo é de 10 anos.

Uma política de restauração pontual só permite que você mantenha o backups por 35 dias, e uma política de frequência differential backup especifica com que frequência você deseja que o differential backup seja tomado.

[Azure SQL Database: Gerenciar a longo prazo backup retenção - Azure SQL Database - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/database/long-term-backup-retention-configure?view=azuresql&amp;tabs=portal)

[Fazer backup e restore de bancos de dados - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/backup-restore-databases/)

</details>

---

### Q71

**Questão:**
Você tem uma instância do Microsoft SQL Server 2019. 

Você está configurando uma solução de manutenção backup.

Você precisa garantir que os bancos de dados possam ser restaurados a qualquer ponto no tempo.

Quais dois tipos de backups você deve incluir? Cada resposta correta apresenta parte da solução.

**Alternativas:**
- **A.** backup copy-only
- **B.** arquivo
- **C.** filegroup
- **D.** banco de dados completo
- **E.** log de transação

**✅ Resposta:** D, E

<details><summary>Explicação</summary>

Para configurar uma solução de manutenção backup e garantir que Os bancos de dados possam ser restauradas a qualquer momento, é necessário um banco de dados completo backup e um transaction log backup.

A copy-only backup é a SQL Server backup that is independent of the sequence of conventional SQL Server backups. Um arquivo backup contém todos os dados em um ou mais arquivos, e um grupo de arquivos backup contém todos os dados em um ou mais grupos de arquivos.

[Voltar para cima e restaurar SQL Server correr em Azure virtual machines - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/backup-restore-databases/2-backup-restore-sql-server-running-azure-virtual-machines)

</details>

---

### Q72

**Questão:**
Você tem um Azure SQL Managed Instance que hospeda cinco bancos de dados.

Você precisa configurar a recuperação de desastres para os bancos de dados em caso de uma interrupção regional. A solução não deve criar configurações adicionais read-only replicas ou manual failover.

O que você pode usar?

**Alternativas:**
- **A.** active geo-replication
- **B.** Always On availability groups
- **C.** auto-failover groups
- **D.** retenção de longo prazo backup

**✅ Resposta:** C

<details><summary>Explicação</summary>

O recurso auto-failover groups permite gerenciar a replicação e failover de alguns ou todos os bancos de dados em um servidor lógico para outra região, o que permite configurar alta disponibilidade no caso de uma interrupção regional.

Active geo-replication cria uma réplica de banco de dados secundário em outra região, mas é assincronicamente mantido atualizado para Azure SQL Database. O recurso Always On availability groups é uma solução de alta disponibilidade e recuperação de desastres (HA/DR), mas não é para Azure SQL Database. A longo prazo backup banco de dados de lojas de retenção backups em Azure Blob storage.

[Explorar auto-failover groups E Azure SQL Database e Azure SQL Managed Instance - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/explore-iaas-paas-platform-tools-for-high-availability-disaster-recovery/6-explore-auto-failover-groups)

</details>

---

### Q73

**Questão:**
Você tem uma assinatura do Azure que contém quatro máquinas virtuais. Todas as máquinas virtuais têm o Microsoft SQL Server 2019 instalado.

A distributed availability group chamada DG1 tem uma réplica em cada máquina virtual.

Você precisa habilitar o monitoramento para DG1. A solução deve ajudar a diagnosticar problemas de desempenho.

O que você deve usar?

**Alternativas:**
- **A.** Extended Events
- **B.** Query Performance Insight
- **C.** SQL Database Advisor

**✅ Resposta:** A

<details><summary>Explicação</summary>

Extended Events é uma solução abrangente para monitorar qualquer produto da família Azure SQL.

Query Performance Insight fornece análise de consulta inteligente para bancos de dados individuais e agrupados, mas não pode fornecer informações sobre um distributed availability group.

SQL Database Advisor fornece recomendações de desempenho, mas não pode fornecer informações sobre um distributed availability group.

[Monitoramento e ajuste de desempenho - Azure SQL Database - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/database/monitor-tune-overview?view=azuresql)

</details>

---

### Q74

**Questão:**
Você tem uma assinatura do Azure que contém cinco máquinas virtuais. As máquinas virtuais hospedam um Microsoft SQL Server 2019 Always On availability group chamado AG1.

Você precisa habilitar o monitoramento para AG1. A solução deve garantir que você possa monitorar a saúde do AG1 e diagnosticar problemas de desempenho.

O que você deve usar?

**Alternativas:**
- **A.** Intelligent Insights
- **B.** Query Performance Insight
- **C.** SQL Database Advisor
- **D.** Extended Events

**✅ Resposta:** D

<details><summary>Explicação</summary>

Extended events permite monitorar a saúde do AG1 e diagnosticar quaisquer problemas de desempenho.

Query Performance Insight fornece análise de consulta inteligente para bancos de dados individuais e agrupados, SQL Database Advisor fornece recomendações de desempenho e Intelligent Insights usa inteligência embutida para monitorar continuamente o uso de banco de dados através da inteligência artificial (AI) e detectar eventos disruptivos que causam mau desempenho.

[Azure SQL Database e Azure SQL Managed Instance documentação de monitoramento e afinação - Azure SQL - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/database/monitoring-tuning-index?view=azuresql&amp;viewFallbackFrom=auresql)

[Explorar soluções IaaS e PaaS para alta disponibilidade e recuperação de desastres - Treinamento | Microsoft Learn](https://learn.microsoft.com/training/modules/explore-iaas-paas-platform-tools-for-high-availability-disaster-recovery/)

</details>

---

### Q75

**Questão:**
Você tem um Azure SQL Database que usa o nível de serviço General Purpose.

Você precisa garantir que haja pelo menos duas cópias legíveis do banco de dados e o banco de dados esteja disponível se um datacenter ficar indisponível.

O que você deve incluir na recomendação?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** active geo-replication

<details><summary>Explicação</summary>

O Active geo-replication é um recurso que permite criar uma banco de dados secundária continuamente sincronizada e legível para uma banco de dados primária. A banco de dados secundária legível pode estar na mesma região do Azure que a banco de dados primária, ou, mais comumente, numa região diferente.

O failover groups permite que você gerencie a replicação e o failover de alguns ou todos os bancos de dados em um servidor lógico para outra região, mas não permite que você crie várias réplicas geo-secundárias Azure SQL Database. O backups automatizado não permite que você tenha duas cópias legíveis do banco de dados disponíveis e o elastic pools é usado para gerenciar e dimensionar vários bancos de dados que têm demandas de uso variáveis e imprevisíveis. Os bancos de dados em um elastic pool estão em um único servidor e um único número compartilhado.

Active geo-replication - Azure SQL Database - Training | Microsoft Learn

Explorar soluções IaaS e PaaS para alta disponibilidade e recuperação de desastres - Treinamento | Microsoft Learn

</details>

---

### Q76

**Questão:**
Você tem um servidor on-premises Microsoft SQL Server 2019 que hospeda 10 bancos de dados atualizados com patches. É necessário replicar todos os bancos de dados para a nuvem.

Quais duas opções você poderia recomendar? Cada resposta correta apresenta uma solução completa.

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** instância SQL Server on Azure Virtual Machines; Um link Azure SQL Managed Instance

<details><summary>Explicação</summary>

O SQL Server on Azure Virtual Machines é necessário para a alta disponibilidade híbrida e recuperação de desastres (HA/DR).

O link Azure SQL Managed Instance permite a replicação de dados quase em tempo real entre SQL Server e Azure SQL Managed Instance.

Azure SQL Database e um Azure SQL Database elastic pool não são as opções corretas, pois não podem ser usadas com o Always On availability groups.

Alta disponibilidade, recuperação de desastres, continuidade dos negócios - SQL Server on Azure VMs - Training | Microsoft Learn

Planejar e implementar um ambiente de alta disponibilidade e recuperação de desastres - Treinamento | Microsoft Learn

</details>

---

### Q77

**Questão:**
Você tem um Azure SQL Managed Instance que contém um banco de dados chamado db1. Você tem um backup válido de db1.

Você precisa restaurar o db1 ao ponto no tempo do backupdb1 sobre o banco de dados existente.

Quais ações você deve executar? Cada resposta correta apresenta parte da solução.

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** Drop db1.; Restaurar db1 de um URL.

<details><summary>Explicação</summary>

Você deve soltar o banco de dados existente primeiro porque o banco de dados existente não pode ser sobrescrito no SQL managed instance. Você também deve restaurar o banco de dados db1 de um URL.

A configuração db1 para o modo de usuário único não é permitida no SQL managed instance, e enquanto o SQL managed instance permite que você execute uma restauração manual do banco de dados a partir de um URL, ele não permite que você restaure o banco de dados de uma fita ou disco.

Restaurar um banco de dados de a backup - Azure SQL Managed Instance - Training | Microsoft Learn

Fazer backup e restore de bancos de dados - Treinamento | Microsoft Learn

</details>

---

### Q78 ⚠️

**Questão:**
Você tem um servidor lógico Azure SQL chamado SQL1 que hospeda cinco bancos de dados.

Você precisa garantir que você pode restaurar Os bancos de dados se SQL1 é excluído.

O que você deve incluir na solução?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** retenção de longo prazo backup

<details><summary>Explicação</summary>

O uso do long-term retention permitirá que você restaure os bancos de dados se o servidor for excluído.

As opções restantes não permitem uma restauração das bancos de dados se um servidor for excluído. Accelerated database recovery é uma reformulação do processo de recuperação do motor de banco de dados Microsoft SQL Server, permitindo uma recuperação rápida e consistente da banco de dados. Azure SQL Database elastic pools são uma solução para gerir e dimensionar várias bancos de dados. O serviço Azure Backup fornece soluções simples, seguras e económicas para fazer backup dos seus dados e recuperá-los a partir da nuvem Microsoft Azure.

A longo prazo backup retenção - Azure SQL Database & Azure SQL Managed Instance - Training | Microsoft Learn

Fazer backup e restore de bancos de dados - Treinamento | Microsoft Learn

</details>

---

### Q79

**Questão:**
Você tem um Always On availability group que contém três réplicas do Microsoft SQL.

Você tem um processo de negócios que é executado em cada réplica. O processo é executado a cada hora e deve ser iniciado apenas na réplica primária.

Qual ponto de vista você deve consultar para obter as informações necessárias sobre o estado de cada réplica?

**Alternativas:** Não salvo (apenas a resposta foi salva)

**✅ Resposta:** sys.dm_hadr_availability_replica_states

<details><summary>Explicação</summary>

A vista sys.dm_hadr_availability_replica_states retorna uma linha para cada réplica local e uma linha para cada réplica remota no mesmo Always On availability group que uma réplica local. Cada linha contém informações sobre o estado de uma determinada réplica. O papel das colunas e o role_desc contêm informações sobre a descrição do papel da réplica.

sys.availability_groups retorna uma linha para cada availability group para o qual a instância local de SQL Server Apresenta uma réplica de disponibilidade. sys.dm_hadr_database_replica_cluster_states retorna uma linha que contém informações destinadas a fornecer insights sobre a saúde dos bancos de dados de disponibilidade no Always On availability groups em cada Always On availability group a Windows Server Failover Cluster (WSFC) Cluster. sys.availabilty_replicas retorna uma linha para cada réplica de disponibilidade que pertence a qualquer Always On availability group na WSFC failover Cluster.

Monitor availability groups utilizando Transact-SQL (T-SQL) - SQL Server Always On - Training | Microsoft Learn

sys.dm_hadr_availability_replica_states (Transact-SQL) - SQL Server - Training | Microsoft Learn

Explorar soluções IaaS e PaaS para alta disponibilidade e recuperação de desastres - Treinamento | Microsoft Learn

</details>

---
