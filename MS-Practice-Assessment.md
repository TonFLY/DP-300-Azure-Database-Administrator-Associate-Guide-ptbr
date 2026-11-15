# DP-300 — Microsoft Learn Practice Assessment Questions

**79 questions** from Microsoft's official practice test (109 in total, 50 per attempt).
Answers and explanations are Microsoft's. ⚠️ = you got it wrong in an earlier attempt.

## Plan and implement data platform resources (17)

### Q1

**Question:**
You have an on-premises Microsoft SQL Server 2019 instance that uses FileTable.

You need to migrate the instance to Azure. The solution must support the existing application code.

What should you use?

**Options:**
- **A.** Azure SQL Database HyperScale
- **B.** Azure SQL Database serverless
- **C.** Azure SQL Managed Instance
- **D.** SQL Server on Azure Virtual Machines

**✅ Answer:** D

<details><summary>Why</summary>

SQL Server on Azure Virtual Machines supports FileTable. It can be used to migrate the instance to Azure and support the existing application code.

Azure SQL Database serverless and Azure SQL Database Hyperscale are tiers of Azure SQL Database that do not support FileTable. Azure SQL Managed Instance does not support FileTable.

[Explore Azure SQL Database Managed Instance - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/prepare-to-maintain-sql-databases-azure/5-explore-azure-sql-database-managed-instance?source=learn)  
[Compare the database engine features of SQL Database and SQL Managed Instance - Azure SQL Database & SQL Managed Instance | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/database/features-comparison?view=azuresql)

</details>

---

### Q2

**Question:**
You have an Azure SQL Database that is only used to process data for three hours a day. Currently, the database has the Standard S3 (100 DTU) service tier configured.

You need to recommend a solution that will increase the performance without increasing the cost.

Which database service tier do you recommend?

**Options:**
- **A.** Premium DTU
- **B.** Business Critical vCore
- **C.** Hyperscale vCore
- **D.** General Purpose serverless vCore

**✅ Answer:** D

<details><summary>Why</summary>

By choosing the vCore serverless service tier, the database will be active only during the daily schedule, and the amount of vCores can be configured dynamically to handle the workload.

The Premium, Hyperscale, and Business Critical service tiers will be a more expensive solution.

[What is the Hyperscale service tier? - Azure SQL Database - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/database/service-tier-hyperscale?view=azuresql)

[Understand SQL database hyperscale - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/deploy-paas-solutions-with-azure-sql/5-understand-sql-database-hyperscale)

</details>

---

### Q3

**Question:**
You plan to deploy an Azure SQL Database that will include a table named Table1. Table1 will include the following fields:

**Options:**
- **A.** StockItemTransactionID (Int)
- **B.** StockItemID (Int)
- **C.** TransactionTypeID (Int)
- **D.** CustomerID (Int)
- **E.** InvoiceID (Int)
- **F.** SupplierID (Int)
- **G.** PurchaseOrderID (Int)
- **H.** TransactionOccurredWhen (Datetime)
- **I.** Quantity (Int)
- **J.** LastEditedBy (Int)
- **K.** LastEditedWhen (Datetime)

You need to recommend a partitioning method for the table. The solution must meet the following requirements:
- **L.** The data must be kept for 2 years.
- **M.** All data older than two years must be removed once a month.
- **N.** Performance of data removal operations must be maximized

Which three actions should you perform? Each correct answer presents part of the solution.
- **O.** Create a partition function for each month.
- **P.** Create a partition function for each year.
- **Q.** Create a partition scheme for filegroups for each year.
- **R.** Create a partition scheme for the PRIMARY filegroup.
- **S.** Create the table by using the partition scheme on the StockItemTransactionID column.
- **T.** Create the table by using the partition scheme on the TransactionOccurredWhen column.

**✅ Answer:** O, R, T

<details><summary>Why</summary>

The data must be removed each month so it will be more efficient to create a partition function for each month. You can only use the PRIMARY filegroup in Azure SQL Database, and the data should be partitioned on datetime.

Also, because the data must be removed each month, it will be less efficient to create a partition for each year. You cannot create filegroups in Azure SQL Database.

[Create partitioned tables and indexes - SQL Server, Azure SQL Database, Azure SQL Managed Instance - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/partitions/create-partitioned-tables-and-indexes?view=azuresqldb-current)

[Explore performance and security - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/deploy-iaas-solutions-with-azure-sql/4-explore-performance-and-security?ns-enrollment-type=learningpath&amp;ns-enrollment-id=learn.wwl.plan-implement-data-platform-resources)

</details>

---

### Q4

**Question:**
You have several databases in an Azure SQL Managed Instance that uses the General Purpose vCore service tier.

You discover high latency in some of the database files.

What should you do to reduce latency?

**Options:**
- **A.** Add more files to the database.
- **B.** Add more vCores to the instance.
- **C.** Change the service tier.
- **D.** Increase the file size.

**✅ Answer:** D

<details><summary>Why</summary>

You should increase the file size to improve performance. If you have a file that is up to 128 GB, then you can have up to 500 IOPS. If you change the file size to 1 TB, then you can have up to 5,000 IOPS.

You should not add more vCores to the instance in the General Purpose tier, as this will not change the available IOPS. Adding more files to the database also does not help, as the number of available IOPS is calculated per file size, not by the number of files. Changing the service tier will change the type of CPU, the maximum file size, and the amount of memory reserved per CPU.

[Resource limits - Azure SQL Managed Instance - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/managed-instance/resource-limits?view=azuresql)

[Deploy PaaS solutions with Azure SQL - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/deploy-paas-solutions-with-azure-sql/)

</details>

---

### Q5

**Question:**
Your organization is facing performance issues with a large table in an Azure SQL Database due to frequent read and write operations.  
  
You need to optimize the table's performance by improving query efficiency and reducing storage usage.  
  
What should you do?

**Options:**
- **A.** Apply page-level compression to the table.
- **B.** Create a clustered index on the table.
- **C.** Enable row-level compression for the table.
- **D.** Partition the table based on date.

**✅ Answer:** C

<details><summary>Why</summary>

Row-level compression optimizes storage by reducing the size of data stored and improves query performance by minimizing I/O operations, making it the most suitable solution for tables with frequent read and write operations. Creating a clustered index enhances query performance but does not address storage optimization. Page-level compression effectively reduces storage but introduces higher CPU overhead, making it less suitable for frequent updates. Partitioning based on date improves query performance for specific scenarios but does not inherently reduce storage requirements.  
  
[Tune and maintain indexes - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/evaluate-performance-improvements/3-tune-and-maintain-indexes)  
[Explore database maintenance checks - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/configure-databases-for-optimal-performance/2-explore-database-maintenance-checks)

</details>

---

### Q6

**Question:**
You plan to deploy a SQL Server on an Azure Virtual Machines instance running on a Windows Server operating system.

You need to configure storage for the virtual machine. The solution must ensure the highest performance for the data files.

Which two actions should you perform? Each correct answer presents part of the solution.

**Options:**
- **A.** Set host caching to none.
- **B.** Set host caching to read-only.
- **C.** Set host caching to read/write.
- **D.** Stripe multiple disks by using Storage Spaces.
- **E.** Use Write Accelerator disks.

**✅ Answer:** B, D

<details><summary>Why</summary>

You should stripe multiple disks, as it will increase IOPS, as well as set host caching to read-only, as data files perform best by using this type of host caching.

Setting host caching to none means that reads will not be cached, setting host caching to read/write will degrade performance and lead to data loss. Write Accelerator Disks should be used for transaction log disks.

[Storage: Performance best practices & guidelines - SQL Server on Azure VMs - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage?source=recommendations&amp;view=azuresql)

[Optimize database storage - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-sql-server-resources-optimal-performance/4-optimize-database-storage)

</details>

---

### Q7

**Question:**
You are deploying a SQL Server on Azure Virtual Machines instance that will be used as a development environment. You will develop two small databases, one of which will have in-memory tables.

You need to choose the most suitable virtual machine type for the deployment.

What should you choose?

**Options:**
- **A.** compute optimized
- **B.** general purpose
- **C.** GPU Optimized
- **D.** memory optimized

**✅ Answer:** B

<details><summary>Why</summary>

General purpose virtual machine sizes provide a balanced CPU-to-memory ratio. This is ideal for testing and development, small to medium databases, and low to medium traffic web servers.

Compute optimized virtual machine sizes have a high CPU-to-memory ratio. This is good for medium traffic web servers, network appliances, batch processes, and application servers.

Memory optimized virtual machine sizes offer a high memory-to-CPU ratio that is used for relational database servers, medium to large caches, and in-memory analytics.

GPU optimized virtual machine sizes are specialized virtual machines available with single, multiple, or fractional GPUs. These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads.

[Azure VM sizes - General purpose - Azure Virtual Machines - Training | Microsoft Learn](https://learn.microsoft.com/azure/virtual-machines/sizes-general)

[Prepare to maintain SQL databases on Azure - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/prepare-to-maintain-sql-databases-azure/)

</details>

---

### Q8

**Question:**
You are deploying a SQL Server on Azure Virtual Machines instance for a production environment. One of the databases in this instance has in-memory tables.

You need to choose the most suitable virtual machine type for the deployment.

What should you choose?

**Options:**
- **A.** compute optimized
- **B.** general purpose
- **C.** GPU Optimized
- **D.** memory optimized

**✅ Answer:** D

<details><summary>Why</summary>

Memory optimized virtual machine sizes offer a high memory-to-CPU ratio that is good for relational database servers, medium to large caches, and in-memory analytics.

General purpose virtual machines provide a balanced CPU-to-memory ratio. These are best for testing and development, small to medium databases, and low to medium traffic web servers.

Compute optimized virtual machines have a high CPU-to-memory ratio. These are used for medium traffic web servers, network appliances, batch processes, and application servers.

GPU optimized virtual machine sizes are specialized virtual machines available with single, multiple, or fractional GPUs. These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads.

[Azure VM sizes - Memory - Azure Virtual Machines - Training | Microsoft Learn](https://learn.microsoft.com/azure/virtual-machines/sizes-memory)

[Prepare to maintain SQL databases on Azure - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/prepare-to-maintain-sql-databases-azure/)

</details>

---

### Q9

**Question:**
You are planning a migration from on-premises Microsoft SQL Server to Azure using Azure Database Migration Service.

You need to keep databases online during the migration.

Which two services can you use? Each correct answer presents a complete solution.

**Options:**
- **A.** Azure Cosmos DB
- **B.** Azure SQL Database
- **C.** Azure SQL Managed Instance
- **D.** SQL Server on Azure Virtual Machines

**✅ Answer:** C, D

<details><summary>Why</summary>

SQL Server on Azure Virtual Machines and Azure SQL Managed Instance both support online migration.

Azure SQL Database does not support online migration and Azure CosmosDB is not a relational database.

[Supported database migration scenarios - Azure Database Migration Service - Training | Microsoft Learn](https://learn.microsoft.com/azure/dms/resource-scenario-status)

[Migrate SQL workloads to Azure SQL Databases - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/migrate-sql-workloads-azure-sql-databases/)

</details>

---

### Q10

**Question:**
You have an on-premises database that uses Transparent Data Encryption (TDE).

You plan to migrate the database to an Azure SQL Managed Instance named Instance1. 

You export the TDE certificate to a file. 

You need to ensure that you can restore the database to Instance1.

Where should you upload the certificate?

**Options:**
- **A.** a Recovery Services vault
- **B.** an Azure key vault
- **C.** Instance1
- **D.** Azure Blob Storage

**✅ Answer:** C

<details><summary>Why</summary>

The certificate should be uploaded to the managed instance, as the certificate should not be stored in either a Recovery Services vault or a key vault for the managed instance to have access.

[Migrate TDE certificate - managed instance - Azure SQL Managed Instance - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/managed-instance/tde-certificate-migrate?view=azuresql&amp;tabs=azure-powershell)

[Migrate SQL workloads to Azure Managed Instances - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/migrate-sql-workloads-azure-managed-instances/)

</details>

---

### Q11

**Question:**
You have 20 on-premises Microsoft SQL Server instances.

You need to migrate the instances to Azure. The solution must support auditing at the server level.

Which two services can you use? Each correct answer presents a complete solution.

**Options:** not saved (only the answer was saved)

**✅ Answer:** Azure SQL Managed Instance; SQL Server on Azure Virtual Machines

<details><summary>Why</summary>

SQL Server on Azure Virtual Machines and Azure SQL Managed Instance support SQL Server Audit and can be used.

Azure SQL Database Serverless serverless is a compute tier of Azure SQL Database that does not support SQL Server Audit. Azure SQL Database Elastic PoolElastic pool is a shared database offering of Azure SQL Database that doesn’t support SQL Server Audit. Azure SQL Database HyperScale Hyperscale is a service tier of Azure SQL Database that does not support SQL Server Audit.

Explore Azure SQL Database Managed Instance - Training | Microsoft Learn

T-SQL differences between SQL Server & Azure SQL Managed Instance - Azure SQL Managed Instance | Microsoft LearnSQL Server Audit (Database Engine) - SQL Server | Microsoft Learn

</details>

---

### Q12

**Question:**
You are building a multi-tenant app that will use an Azure SQL Database.

You need to partition the database. The solution must meet the following requirements:

Improve scalability.
Reduce contention.
Optimize performance.
Isolate each tenant’s data.
What should you use?

**Options:** not saved (only the answer was saved)

**✅ Answer:** a list shard map only

<details><summary>Why</summary>

Using only a list shard map is the correct solution because each tenant’s data will be stored in a separate database.

A range shard map will allow data from multiple tenants to be in stored in the same database.

Explore performance and security - Training | Microsoft Learn

Data partitioning strategies - Azure Architecture Center - Training | Microsoft Learn

</details>

---

### Q13

**Question:**
You have an on-premises instance of Microsoft SQL Server 2019 that uses a SQL common language runtime (CLR) stored procedure to access a local file share on the server.

You need to migrate the instance to Azure.

To which Azure service should you migrate the instance?

**Options:** not saved (only the answer was saved)

**✅ Answer:** SQL Server on Azure Virtual Machines

<details><summary>Why</summary>

SQL Server on Azure Virtual Machine supports SQL CLR objects. The objects can access files shares, disks, and folders.

Azure SQL Database serverless is a compute tier for Azure SQL Database that does not support SQL CLR objects.

Azure SQL Database Hyperscale is a service tier of Azure SQL Database that does not support SQL CLR objects.

Azure SQL Managed Instance does not support SQL CLR objects that access file shares or folders of the operating system.

Compare the database engine features of SQL Database and SQL Managed Instance - Azure SQL Database & SQL Managed Instance - Training | Microsoft Learn

T-SQL differences between SQL Server & Azure SQL Managed Instance - Azure SQL Managed Instance - Training | Microsoft Learn

</details>

---

### Q14

**Question:**
You plan to migrate a 40-TB OLTP database to Azure SQL Database.

Which service tier should you recommend for the database?

**Options:** not saved (only the answer was saved)

**✅ Answer:** Hyperscale vCore service tier

<details><summary>Why</summary>

You should recommend the Hyperscale vCore service tier, as it supports database sizes of up to 100 TB.

The General Purpose and Business Critical vCore service tier and the DTU Premium service tier only support a database size of up to 4 TB.

What is the Hyperscale service tier? - Azure SQL Database - Training | Microsoft Learn

Understand SQL database hyperscale - Training | Microsoft Learn

</details>

---

### Q15

**Question:**
You have a resource group named RG1 that contains a General Purpose service tier Azure SQL Managed Instance named Instance1.

You need to change the service tier of Instance1 to Business Critical.

Which command should you run?

**Options:** not saved (only the answer was saved)

**✅ Answer:** Set-AzSqlInstance -Name Instance1 -ResourceGroupName RG1 -Edition BusinessCritical

<details><summary>Why</summary>

Set-AzSqlInstance -Name Instance1 -ResourceGroupName RG1 -Edition BusinessCritical is correct. This will set the edition to Business Critical.

Set-AzSqlElasticPool -Edition BusinessCritical -Name mydb -ServerName Instance1 -ResourceGroupName RG1 will set the edition of an Azure SQL Database elastic pool to Business Critical.

Set-AzSqlInstance -Name Instance1 -ResourceGroupName RG1 -Edition GeneralPurpose will set the edition to General Purpose.

Set-AzSqlDatabase -ResourceGroupName RG1 -Name mydb -ServerName Instance1 -Edition BusinessCritical will set the edition of an Azure SQL Database to Business Critical.

Set-AzSqlInstance (Az.Sql) - Training | Microsoft Learn

Prepare to maintain SQL databases on Azure - Training | Microsoft Learn

</details>

---

### Q16

**Question:**
Your organization uses Azure SQL Database to store transactional data. The database contains large tables with frequent inserts, updates, and deletes, and performance analysis shows frequent table scans that are increasing I/O operations and slowing down OLTP workloads.

You need to improve query performance and reduce I/O operations while maintaining data integrity.

Each correct answer presents part of the solution. Which two actions should you perform?

**Options:** not saved (only the answer was saved)

**✅ Answer:** Create nonclustered indexes.; Enable row compression.

<details><summary>Why</summary>

Creating nonclustered indexes enhances query performance by enabling efficient lookups for specific rows, which reduces the need for full table scans and minimizes I/O operations. Row compression decreases the size of stored data, leading to fewer pages being read during queries and improved I/O performance. Columnstore compression is not suitable for this scenario because it is designed for rarely accessed data and can increase CPU usage for tables with high read and write activity. Rebuilding indexes does not specifically address the performance issues described, as it focuses on maintaining index structure rather than optimizing query execution.

Design indexes - Training | Microsoft Learn
Describe intelligent query processing - Training | Microsoft Learn

</details>

---

### Q17

**Question:**
You have an on-premises Microsoft SQL Server 2019 instance.

You need to migrate the instance to an Azure SQL Managed Instance. The solution must minimize downtime.

What should you do first?

**Options:** not saved (only the answer was saved)

**✅ Answer:** Create an Azure SQL Managed Instance with premium series hardware. Create full and log backups with WITH CHECKSUM. Store the backups in an SMB network share.

<details><summary>Why</summary>

Premium hardware must be used for an online migration. The backups must also be taken with CHECKSUM enabled.

Standard hardware cannot be used for an online migration and storing the backups in Blob storage creates an Azure snapshot of the database files.

Tutorial: Migrate SQL Server online to SQL Managed Instance - Azure Database Migration Service - Training | Microsoft Learn

</details>

---

## Implement a secure environment (18)

### Q18

**Question:**
You create a stored procedure that runs dynamic SQL code to select data from a table named Sales. 

All users have execute permissions to the stored procedure. 

What should you do to ensure that the users can run the procedure without errors?

**Options:**
- **A.** Add a WITH RECOMPILE clause to the stored procedure.
- **B.** Add an EXECUTE AS [owner] clause to the stored procedure.
- **C.** Add the SELECT permission to the Sales table for the users.

**✅ Answer:** C

<details><summary>Why</summary>

You should add the SELECT permission to the Sales table because executing dynamic SQL code requires that the caller has all the necessary permissions to the objects that are called in the code.

Using the EXECUTE AS [owner] clause or the WITH RECOMPILE clause does not ensure that the caller has all the necessary permissions to the objects that are called in the code.

[EXECUTE AS (Transact-SQL) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/t-sql/statements/execute-as-transact-sql?view=sql-server-ver16)

[Configure database authentication and authorization - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-database-authentication-authorization/)

</details>

---

### Q19

**Question:**
You have an app that stores data for multiple tenants using Azure SQL Database elastic pools. Each tenant has a database. 

You need to configure an authentication solution for the app. The solution must meet the following requirements:

**Options:**
- **A.** Ensure that users can only access the database of their respective tenant.
- **B.** Ensure databases can be moved to a dedicated instance.
- **C.** Minimize administrative effort.

What should you include in the solution?
- **D.** contained database users
- **E.** database roles
- **F.** server roles

**✅ Answer:** D

<details><summary>Why</summary>

Contained database users enable databases to be moved without recreating logins.

Server roles manage permissions at the server level only, and database roles manage permissions within a database.

[Contained user access to contained databases - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/security/contained-database-users-making-your-database-portable?view=sql-server-ver16)

[Configure database authentication and authorization - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-database-authentication-authorization/)

</details>

---

### Q20

**Question:**
You have an Azure SQL server that hosts 10 Azure SQL Databases.

You create an app named App1 with a system assigned managed identity.

You need to ensure that App1 has read access to each database. by using the managed identity of App1.

Which three actions should you perform? Each correct answer presents part of the solution.

**Options:**
- **A.** Assign the database user the db\_reader role for each database.
- **B.** Assign the database user the db\_reader role for the master database.
- **C.** Create a login for App1 from an external provider.
- **D.** Create a login for App1 that uses a password.
- **E.** Create a user in each database for the App1 login.
- **F.** Create a user in the master database for the App1 login.

**✅ Answer:** A, C, E

<details><summary>Why</summary>

A managed identity requires that a login be created from an external provider, each database requires a user for the login, and assigning the database user the db\_reader role for each database will allow read access to the user databases.

A managed identity cannot use Windows authentication. The app does not require a user in the master database and assigning the database user the db\_reader role for the master database will allow the user read access to the master database, but not read access to the user databases.

[Describe Security Principals - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-database-authentication-authorization/4-describe-security-principals)

</details>

---

### Q21

**Question:**
You have an Azure SQL Database named db1 that includes two custom roles named db1\_readers and db2\_writers. You have a Microsoft Entra group named dbadmin.

You need to ensure that the members of the dbadmin group can manage the membership of db1\_readers and db1\_writers. The solution must follow the principle of least privilege.

Which two actions should you perform? Each correct answer presents part of the solution.

**Options:**
- **A.** Assign the database user the db\_accessadmin role for the db1 database.
- **B.** Assign the database user the db\_securityadmin role for the db1 database.
- **C.** Assign the database user the loginmanager role for the master database.
- **D.** Assign the database user the sysadmin role for the master database.
- **E.** Create a login and a database user in db1 for the dbadmin group.
- **F.** Create a login and a database user in the master database for the dbadmin group.

**✅ Answer:** B, E

<details><summary>Why</summary>

You should create a login and a database user in db1 for the dbadmin group, because members of the dbadmin group need a user in the db1 database. Members of the db\_securityadmin role can modify role membership for custom roles.

The remaining options are incorrect. Members of the dbadmin group do not need server level permissions. Members of the sysadmin role can perform any activity on the instance. Members of the db\_accessadmin role can add and remove access to the database. Members of the loginmanager role can create and delete logins in the virtual master database.

https://learn.microsoft.com/sql/relational-databases/security/authentication-access/database-level-roles?view=sql-server-ver16[Database-Level Roles - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles?view=sql-server-ver16)

[Configure database authentication and authorization - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-database-authentication-authorization/)

</details>

---

### Q22

**Question:**
You have an Azure SQL database named db1 that contains a table named Sales.

You need to encrypt data in a column named CreditCard in the Sales table.

Which two actions should you perform before you can encrypt the data? Each correct answer presents part of the solution.

**Options:**
- **A.** Create a certificate in the db1 database.
- **B.** Create a certificate in the master database.
- **C.** Create a symmetric key encrypted by a certificate.
- **D.** Create an asymmetric key encrypted by a certificate.

**✅ Answer:** A, C

<details><summary>Why</summary>

The certificate created in db1 will be used to encrypt a symmetric key, which, in turn, will be used to protect data in the CreditCard column in the Sales table. The column containing encrypted data must be varbinary column.

A symmetric key, not an asymmetric key, is required for this operation. Creating the certificate in the master database would not work, as the certificate that is needed for encryption must be created in the user database.  

[Encrypt a Column of Data - SQL Server & Azure Synapse Analytics & Azure SQL Database & SQL Managed Instance - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/security/encryption/encrypt-a-column-of-data?view=sql-server-ver16)

[Protect data in-transit and at rest - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/protect-data-transit-rest/)

</details>

---

### Q23

**Question:**
You have an Azure SQL Database named db1.

You need to ensure that all the data in db1 is encrypted at rest.

What should you use?

**Options:**
- **A.** Always Encrypted
- **B.** Azure SQL auditing
- **C.** Microsoft Defender for SQL
- **D.** Transparent Data Encryption (TDE)

**✅ Answer:** D

<details><summary>Why</summary>

TDE is the Azure SQL Database feature that encrypts data at rest. All the data saved to a disk is encrypted.

Auditing is not used for performing data encryption at rest, it is used for gathering diagnostic information.

Defender for SQL is used to mitigate potential database vulnerabilities and detect anomalous activities.

Always Encrypted protects data that is in use or in transit.

[Explore Transparent Data Encryption - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/protect-data-transit-rest/2-explore-transparent-data-encryption)

</details>

---

### Q24

**Question:**
In an Azure SQL database, you have a table that contains a column named Column1. Column1 stores credit card CVV data.

You need to configure Column1 to meet the following requirements:

**Options:**
- **A.** The data in the column must be protected at rest.
- **B.** The data in the column must be protected when in use.
- **C.** Database administrators must be prevented from viewing the contents of Column1.

Which two features can you use? Each correct answer presents a complete solution.
- **D.** Always Encrypted Deterministic encryption
- **E.** Always Encrypted Randomized encryption
- **F.** Column-level encryption using symmetric keys
- **G.** dynamic data masking

**✅ Answer:** D, E

<details><summary>Why</summary>

You should use Always Encrypted (Randomized encryption or Deterministic encryption), because this ensures that the administrators are unable to view and decrypt data. 

Encryption with symmetric keys will allow the administrators to access the data. Dynamic data masking does not protect data at rest or in use.

[Explain object encryption and secure enclaves - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/protect-data-transit-rest/4-explain-object-encryption-secure-enclaves)

</details>

---

### Q25

**Question:**
You have an Azure SQL Database that contains a table named Employee. The table has columns that are masked by using dynamic data masking. 

When a user copies the Employee table by using the SELECT INTO command, the data is masked in a new table. The user must see the unmasked data in the new table. 

You need to grant the appropriate permissions to the user. The solution must follow the principle of least privilege.

What should you grant to the user?

**Options:**
- **A.** the ALTER ANY MASK permission
- **B.** the SELECT permission for the new table
- **C.** the UNMASK permission for the Employee table
- **D.** the UNMASK permission for the new table

**✅ Answer:** C

<details><summary>Why</summary>

You should grant the UNMASK permission to the user for the Employee table because data must be unmasked before copying it to the new table.

Granting the UNMASK permission to the user for the new table does not allow the user to see the unmasked data in the new table, as the data cannot be unmasked after it is copied to the new table.

The ALTER ANY MASK permission is dedicated to the security officer as it allows that management of dynamic data masking, and the SELECT permission does not make masked data readable.

[Dynamic Data Masking - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/security/dynamic-data-masking?view=sql-server-ver16)

[Implement compliance controls for sensitive data - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/implement-compliance-controls-sensitive-data/)

</details>

---

### Q26

**Question:**
You have an Azure SQL managed instance Azure SQL Managed Instance that hosts a database.   
  
The database contains a column named Column1 that stores confidential information.

You need to audit access to Column1. The solution must meet the following requirements:

**Options:**
- **A.** Retain audit data for at least eighteen years.
- **B.** Minimize administrative effort.

Which three actions should you perform? Each correct answer presents part of the solution.
- **C.** Apply a sensitivity classification to the column.
- **D.** Configure a Log Analytics audit log destination.
- **E.** Configure a Storage audit log destination.
- **F.** Enable Always Encrypted for the column.
- **G.** Enable Azure SQL Auditing.
- **H.** Enable Transparent Data Encryption (TDE).

**✅ Answer:** C, E, G

<details><summary>Why</summary>

Azure SQL Auditing needs to be enabled to monitor access. A storage account allows unlimited retention. The column needs a classification label so that access is audited.

Log Analytics has a maximum retention limit of seven years, Always Encrypted is used to protect data in use or in transit, and TDE encrypts data files.

[Explore server and database audit - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/implement-compliance-controls-sensitive-data/3-explore-server-and-database-audit)  
[Get started with Azure SQL Managed Instance auditing](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/auditing-configure?view=azuresql)

</details>

---

### Q27

**Question:**
You have an Azure SQL Database named db1 that contains a table. The table contains a column that stores email addresses.

You need to ensure that reporting users can only view email addresses in a format of aXXX@XXX.com. The solution must minimize the impact on other users.

What should you configure?

**Options:**
- **A.** Always Encrypted
- **B.** column encryption
- **C.** dynamic data masking
- **D.** row-level security (RLS)

**✅ Answer:** C

<details><summary>Why</summary>

Dynamic data masking will enable users to only see masked values instead of email addresses.

Always Encrypted protects data in use or in transit. RLS controls access to rows in a table, and column encryption will encrypt the data in the column, which can hinder users’ ability to view the data.

[Dynamic Data Masking - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/security/dynamic-data-masking?view=sql-server-ver16)

[Implement compliance controls for sensitive data - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/implement-compliance-controls-sensitive-data/)

</details>

---

### Q28

**Question:**
You have a Microsoft SQL Server database named SalesDB hosted on an Azure virtual machine. SalesDB is part of an Always On availability group that uses a listener named SalesAGListener and contains a table named dbo.Orders.

An Azure Functions app uses the Azure SQL trigger binding to process inserts and updates from dbo.Orders, but the function fails to fire.

You need to configure SalesDB so that the function is triggered when committed changes occur in dbo.Orders. The solution must use a two-day retention period and automatic cleanup.

What T-SQL code should you run?

**Options:**
- **A.** ALTER DATABASE [SalesDB]  
SET CHANGE\_TRACKING = ON  
(CHANGE\_RETENTION = 2 DAYS, AUTO\_CLEANUP = ON);  
ALTER TABLE dbo.Orders  
ENABLE CHANGE\_TRACKING;
- **B.** EXEC sys.sp\_cdc\_enable\_db;  
EXEC sys.sp\_cdc\_enable\_table  
  @source\_schema = 'dbo',  
  @source\_name = 'Orders',  
  @role\_name = NULL,  
  @capture\_instance = 'dbo\_Orders',  
  @supports\_net\_changes = 1;
- **C.** CREATE EVENT STREAMING GROUP MyOrderStream  
WITH (  
  TARGET\_TYPE = 'eventhub',  
  TARGET\_NAME = 'orders-eventhub',  
  TARGET\_CONNECTION = '&lt;event-hub-connection-string&gt;'  
);  
ALTER EVENT STREAMING GROUP MyOrderStream  
ADD TABLE dbo.Orders;
- **D.** CREATE TRIGGER dbo.  MyOrderStream ON dbo.Orders  
AFTER INSERT  
AS  
BEGIN  
    SET NOCOUNT ON;  
    DECLARE @payload NVARCHAR(MAX) = (  
        SELECT \*  
        FROM inserted  
        FOR JSON AUTO  
    );  
    DECLARE @response NVARCHAR(MAX);  
    EXEC sys.sp\_invoke\_external\_rest\_endpoint  
        @url = N'https://&lt;your-function&gt; ',  
        @method = 'POST',  
        @headers = N'{"Content-Type":"application/json"}',  
        @payload = @payload,  
        @response = @response OUTPUT;  
END;

**✅ Answer:** A

<details><summary>Why</summary>

**Objective:**

2.3 Implement compliance controls for sensitive data

**What This Item Tests:**

Implement change data tracking

**Additional Reading:**  

- [Replication, change tracking, & change data capture - Always On availability groups](https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/replicate-track-change-data-capture-always-on-availability?view=sql-server-ver17#change-tracking)
- [Track data changes (SQL Server)](https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/track-data-changes-sql-server?view=sql-server-ver17)

**Rationale:**

Azure SQL trigger bindings rely on SQL Server Change Tracking, so enabling change tracking at the database level with a two-day retention period and automatic cleanup, and then enabling change tracking on dbo.Orders, enables the function to detect committed changes. Change data capture (CDC) uses log scanning and capture tables and is not the mechanism used by Azure SQL trigger bindings, event streaming groups send table changes to external streaming targets rather than enabling change detection within SQL Server, and system-versioned temporal tables store row history but do not provide the change tracking metadata required by the trigger binding.

</details>

---

### Q29

**Question:**
You have an Azure SQL Database named db1.

You need to provide access to db1 for a Microsoft Entra ID user .

What should you do?

**Options:** not saved (only the answer was saved)

**✅ Answer:** In db1, run the CREATE USER statement with the FROM EXTERNAL PROVIDER option.

<details><summary>Why</summary>

You should use the CREATE USER statement with FROM EXTERNAL PROVIDER option in db1, as this user should have access to this database.

Using the CREATE USER statement with the FROM LOGIN or WITH PASSWORD option creates a login in the master database. By default, the login does not have any access to the databases.

CREATE USER (Transact-SQL) - SQL Server - Training | Microsoft Learn

Configure database authentication and authorization - Training | Microsoft Learn

</details>

---

### Q30

**Question:**
You have an Azure SQL Database named db1.

You create a user named User1 in the master database.

You need to ensure that User1 can create databases. The solution must follow the principle of least privilege.

Which role should you assign to User1?

**Options:** not saved (only the answer was saved)

**✅ Answer:** dbmanager

<details><summary>Why</summary>

You should assign dbmanager, as members of this role can create databases.

Members of the loginmanager role can create and delete users, but they cannot create databases. Members of the db_ddladmin role can run any DDL statements. Members of the db_owner role can perform all activities at the database scope.

Configure database authentication and authorization - Training | Microsoft Learn

</details>

---

### Q31 ⚠️

**Question:**
You have a database named db1 in an Azure SQL managed instance named Instance1.

You need to encrypt db1 by using Transparent Data Encryption (TDE).

Which three actions should you perform before you can start the encryption? Each correct answer present part of the solution.

**Options:** not saved (only the answer was saved)

**✅ Answer:** Create a certificate in the master database.; Create a database encryption key in db1.; Create a master key in the master database.

<details><summary>Why</summary>

The procedure of preparing the database to use TDE is first to create a master key and a certificate in the master database, and then to create a database encryption key in the db1.

You should not create a certificate or an asymmetric key in db1, as TDE can only use certificates created in the master database.

Transparent data encryption (TDE) - SQL Server - Training | Microsoft Learn

Protect data in-transit and at rest - Training | Microsoft Learn

</details>

---

### Q32

**Question:**
You have an Azure SQL Database named db1. The database is a contained database.

You need to update an existing firewall rule to change the range of IP addresses from which the database is accessible.

What should you run?

**Options:** not saved (only the answer was saved)

**✅ Answer:** sp_set_database_firewall_rule in the db1 context

<details><summary>Why</summary>

You should run sp_set_database_firewall_rule from the db1 database because the database is a contained database, so the firewall rule change must be applied directly to this database. The procedure updates the existing rule based on its name.

The rules need to be changed for the db1 database, not the master, so running sp_set_database_firewall_rule from the master database is incorrect. You do not need to change the server firewall rule by running sp_set_firewall_rule. Users trying to access db1 will be redirected directly to the database. Running sp_delete_database_firewall_rule from db1 deletes the firewall rule.

sp_set_database_firewall_rule - Azure SQL Database - Training | Microsoft Learn

Protect data in-transit and at rest - Training | Microsoft Learn

</details>

---

### Q33 ⚠️

**Question:**
You have an Azure SQL Database named db1 that contains a table named Sales.

You need to implement change tracking for Sales.

What should you do first?

**Options:** not saved (only the answer was saved)

**✅ Answer:** Enable change tracking for db1.

<details><summary>Why</summary>

You should enable change tracking at the database level first, then you can enable change tracking for each table.

Change tracking must be enabled at the database level before it can be enabled for the Sales table. Creating a master key in the master database and creating a certificate in db1 are not needed for change tracking.

Enable and Disable Change Tracking - SQL Server - Training | Microsoft Learn

Implement compliance controls for sensitive data - Training | Microsoft Learn

</details>

---

### Q34

**Question:**
An organization uses Azure SQL Database to store sensitive customer information.

You need to limit exposure of sensitive data in query results for non-privileged users, while still allowing authorized users to view the full data. The solution must be transparent to the application and require minimal changes to existing queries.

Which feature should you use?

**Options:** not saved (only the answer was saved)

**✅ Answer:** Dynamic Data Masking

<details><summary>Why</summary>

Dynamic Data Masking is the most suitable solution because it ensures that unauthorized users see masked data while authorized users can view unmasked data, directly addressing the requirement. Always Encrypted enhances security by encrypting sensitive data but does not provide the capability to dynamically mask data for unauthorized users. Row-level security focuses on controlling access to specific rows based on user roles but does not include functionality for masking sensitive data. Auditing is designed to track and monitor access to data but does not prevent unauthorized users from viewing sensitive information.

Explore data classification - Training | Microsoft Learn
Explain object encryption and secure enclaves - Training | Microsoft Learn

</details>

---

### Q35

**Question:**
Your organization uses Microsoft Azure SQL Database to store sensitive customer data, including financial and personal information.

You need to classify and label sensitive data in the database for compliance.

Each correct answer presents part of the solution. Which two actions should you perform?

**Options:** not saved (only the answer was saved)

**✅ Answer:** Classify columns and apply sensitivity labels.; Use Data Discovery & Classification.

<details><summary>Why</summary>

To classify and label sensitive data for compliance, it is necessary to use the Data Discovery & Classification feature to identify sensitive columns and apply sensitivity labels to ensure proper classification. Encrypting database columns with Always Encrypted and enabling Transparent Data Encryption are security measures that protect data but do not address the specific requirement of classification and labeling.

Implement Microsoft Purview - Training | Microsoft Learn
Design Azure SQL Database for cloud-native applications - Training | Microsoft Learn

</details>

---

## Monitor, configure, and optimize database resources (16)

### Q36

**Question:**
You need to configure Extended Events monitoring to capture an event when a new database is added to an Azure SQL Managed Instance.

Which event category should you use?

**Options:**
- **A.** admin
- **B.** analytic
- **C.** debug
- **D.** operational

**✅ Answer:** D

<details><summary>Why</summary>

Operational events are used for analyzing and diagnosing a problem or occurrence. They can be used to trigger tools or tasks based on the problem or occurrence. An example of an operational event is when a database is added or removed from the server.

An admin event indicates a problem within a well-defined set of actions an administrator can take, analytic events describe program operations and are typically used in performance investigations, and debug events are used solely by developers to diagnose a problem for debugging.

[SQL Server Extended Events Packages - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/extended-events/sql-server-extended-events-packages?view=sql-server-ver16)

[Describe performance monitoring - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/describe-performance-monitoring/)

</details>

---

### Q37

**Question:**
You need to set up a baseline for performance monitoring of an Azure SQL Database. 

Which dynamic management view should you query to get the baseline performance data?

**Options:**
- **A.** sys.dm\_db\_wait\_stats
- **B.** sys.dm\_exec\_query\_stats
- **C.** sys.dm\_exec\_sessions\_wait\_stats
- **D.** sys.dm\_os\_waiting\_tasks

**✅ Answer:** A

<details><summary>Why</summary>

sys.dm\_db\_wait\_stats returns information about all the waits encountered in the database. This allows you to see the baseline performance data.

sys.dm\_os\_waiting\_tasks is related to current connections and sessions. This view does not store any historical waits, so it should not be used to create the performance baseline. sys.dm\_exec\_sessions\_wait\_stats is used to find the wait statistics for a particular session. sys.dm\_exec\_query\_stats returns aggregate performance statistics for cached query plans in Microsoft SQL Server.

[sys.dm_db_wait_stats (Azure SQL Database) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-db-wait-stats-azure-sql-database?view=azuresqldb-current)

[Establish baseline metrics - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/describe-performance-monitoring/4-establish-baseline-metrics?ns-enrollment-type=learningpath&amp;ns-enrollment-id=learn.wwl.monitor-optimize-operational-resources-sql-server)

</details>

---

### Q38

**Question:**
You plan to use Query Store in an Azure SQL Managed Instance. 

You need to ensure that you can manually force execution plans for regressed queries. The solution must follow the principle of least privilege. 

To which database role should you be assigned?

**Options:**
- **A.** db\_accessadmin
- **B.** db\_ddladmin
- **C.** db\_owner
- **D.** db\_securityadmin

**✅ Answer:** C

<details><summary>Why</summary>

As a member of the db\_owner fixed database role, you can perform all configuration and maintenance activities on the database., while following the principle of least privilege.

Members of the db\_ddladmin role can run any DDL commands for a database. Members of this role can potentially elevate their privileges by manipulating code that may be executed under high privileges, and their actions should be monitored. A member of the db\_accessadmin fixed database role can add or remove access to the database for Windows logins, Windows groups, and Microsoft SQL Server logins. Members of the db\_securityadmin role can modify role membership for custom roles only and manage permissions.

[Database-Level Roles - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/security/authentication-access/database-level-roles?view=sql-server-ver16)

[Configure database authentication and authorization - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-database-authentication-authorization/)

</details>

---

### Q39

**Question:**
You have an Azure SQL managed instance.

You discover a high number of blocking queries during peak business hours.

You need to investigate the root cause of the blocking queries.

Which two dynamic management views should you query? Each correct answer presents a complete solution.

**Options:**
- **A.** sys.dm\_exec\_query\_stats
- **B.** sys.dm\_exec\_requests
- **C.** sys.dm\_exec\_sessions\_wait\_stats
- **D.** sys.dm\_os\_waiting\_tasks

**✅ Answer:** B, D

<details><summary>Why</summary>

You should query the sys.dm\_exec\_requests view to get the blocking session ID information. This view returns information about each request that is executing in Microsoft SQL Server. You should also use sys.dm\_os\_waiting\_tasks because the information is related to current connections and sessions. There is blocking session ID information that can be used to identify the root of the issue.

The sys.dm\_exec\_sessions\_wait\_stats view presents historical information about the waits that occurred on the server, and the sys.dm\_exec\_query\_stats view returns aggregate performance statistics for cached query plans in SQL Server.

[sys.dm_exec_requests (Transact-SQL) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql?view=sql-server-ver16)

[sys.dm_os_waiting_tasks (Transact-SQL) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-os-waiting-tasks-transact-sql?view=sql-server-ver16)

[Explore query performance optimization - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/explore-query-performance-optimization/)

</details>

---

### Q40

**Question:**
You need to find all indexes that have high fragmentation.

Which two dynamic management views should you query? Each answer presents a part of the solution.

**Options:**
- **A.** sys.dm\_db\_column\_store\_row\_group\_physical\_stats
- **B.** sys.dm\_db\_index\_operational\_stats
- **C.** sys.dm\_db\_index\_physical\_stats
- **D.** sys.dm\_db\_index\_usage\_stats

**✅ Answer:** A, C

<details><summary>Why</summary>

You should query the sys.dm\_db\_index\_physical\_stats view, as it returns size and fragmentation information for the data and indexes of the specified table or view in Microsoft SQL Server. For an index, one row is returned for each level of the B-tree in each partition. For a heap, one row is returned for the IN\_ROW\_DATA allocation unit of each partition. You should also query the sys.dm\_db\_column\_store\_row\_group\_physical\_stats view because it provides current rowgroup-level information about all the columnstore indexes in the current database. Based on the information, you can calculate the index fragmentation.

The sys.dm\_db\_index\_usage\_stats view returns counts of different types of index operations and the time each type of operation was last performed, and the sys.dm\_db\_index\_operational\_stats view returns current lower-level I/O, locking, latching, and access method activity for each partition of a table or index in the database.

[sys.dm_db_index_physical_stats (Transact-SQL) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-db-index-physical-stats-transact-sql?view=sql-server-ver16)

[Configure databases for optimal performance - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-databases-for-optimal-performance/)

</details>

---

### Q41 ⚠️

**Question:**
You have an Azure SQL Managed Instance that contains databases.

One of the databases experiences excessive blocking.

You need to identify which session is causing the blocking.

Which dynamic management view should you query?

**Options:**
- **A.** sys.dm\_exec\_query\_stats
- **B.** sys.dm\_exec\_requests
- **C.** sys.dm\_exec\_session\_wait\_stats
- **D.** sys.dm\_exec\_sessions

**✅ Answer:** B

<details><summary>Why</summary>

sys.dm\_exec\_requests returns information about each request that is executing in Microsoft SQL Server. One of the columns exposed by this view is blocking\_session\_id.

sys.dm\_exec\_sessions returns one row per authenticated session in SQL Server. This view does not return information about blocking issues.

sys.dm\_exec\_query\_stats returns aggregate performance statistics for cached plans in SQL Server.

sys.dm\_exec\_session\_waits returns information about all the waits encountered by threads that are executed for each session.

[sys.dm_exec_requests (Transact-SQL) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql?view=sql-server-ver16)

</details>

---

### Q42

**Question:**
You have an Azure SQL database.

You need to troubleshoot blocking and deadlocking performance issues.

What feature should you use?

**Options:**
- **A.** an Azure SQL Database ledger
- **B.** Extended Events
- **C.** Microsoft Defender for SQL
- **D.** Query Store

**✅ Answer:** B

<details><summary>Why</summary>

Extended Events can capture blocking and deadlocking metrics that can be used to investigate and troubleshoot the performance issues.

Query Store does not capture blocking and deadlocking metrics, Azure SQL Defender surfaces database vulnerabilities, and an Azure SQL Database ledger incrementally captures the state of the database.

[Explore extended events - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/describe-performance-monitoring/5-explore-extended-events)

</details>

---

### Q43 ⚠️

**Question:**
You have an Azure SQL managed instance that hosts two databases named db1 and db2.

You need to limit usage of db1 sessions to a maximum of 30 percent of the available CPU.

Which three actions should you perform? Each correct answer presents part of the solution.

**Options:**
- **A.** Configure the Max Degree of Parallelism.
- **B.** Create a resource pool.
- **C.** Create a workload group.
- **D.** Create an Extended Events session.
- **E.** Enable Automatic Database Tuning.
- **F.** Enable Resource Governor.

**✅ Answer:** B, C, F

<details><summary>Why</summary>

Resource Governor needs to be enabled, a resource pool needs to be created, and a workload group needs to be created to limit db1 to a maximum of 30 percent.

Automatic Database Tuning is used to notify you whenever a potential performance issue is detected and lets you apply corrective actions. Extended Events is a lightweight performance monitoring system. Max Degree of Parallelism limits the number of processors to use in a parallel plan execution.

[Resource Governor - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/resource-governor/resource-governor?view=sql-server-ver15)

[Configure SQL Server resources for optimal performance - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-sql-server-resources-optimal-performance/)

</details>

---

### Q44

**Question:**
You have an Azure SQL Managed Instance that contains a database named DB1. The database compatibility level for DB1 is 150.

Queries that use table variables are performing poorly. You identify that the issue is caused by incorrect cardinality estimations.

You need to configure an option to ensure that the database engine defines the cardinality estimate when the query is executed, and the actual row count is known.

Which option should you configure?

**Options:**
- **A.** DEFERRED\_COMPILATION\_TV
- **B.** ELEVATE\_ONLINE
- **C.** INTERLEAVED\_EXECUTION\_TVF
- **D.** TSQL\_SCALAR\_UDF\_INLINING

**✅ Answer:** A

<details><summary>Why</summary>

You should use the DEFERRED\_COMPILATION\_TV option because it enables or disables table variable compilation.

The INTERLEAVED\_EXECUTION\_TVF option enables or disables interleaved execution for multi-statement table-valued functions.

The TSQL\_SCALAR\_UDF\_INLINING option enables or disables the T-SQL Scalar UDF inlining.

The ELEVATE\_ONLINE option causes the engine to automatically elevate supported operations online.

[ALTER DATABASE SCOPED CONFIGURATION - SQL Server (Transact-SQL) - Training | Microsoft Learn](https://learn.microsoft.com/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql?view=sql-server-ver16)

</details>

---

### Q45

**Question:**
You have an Azure SQL Database that uses Query Store.

You notice poor performance for a report caused by the execution plan and the report parameters.

You need to improve the performance of the report. The solution must minimize administrative effort.

What should you do?

**Options:**
- **A.** Disable parameter sniffing.
- **B.** Enable snapshot isolation.
- **C.** Execute the DBCC FREEPROCCACHE command.
- **D.** Force the most efficient execution plan.

**✅ Answer:** D

<details><summary>Why</summary>

Forcing the plan that is the most efficient for most the executions will improve report performance.

Disabling parameter sniffing will ignore any parameters and not choose the most efficient plan. DBCC FREEPROCCACHE will remove all plans from the cache.

Snapshot isolation is used to enable concurrent execution of transactions.

[Monitor performance by using the Query Store - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store?view=sql-server-ver16#Regressed)

[Evaluate performance improvements - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/evaluate-performance-improvements/)

</details>

---

### Q46 ⚠️

**Question:**
You plan to configure Extended Events monitoring to gather information about queries.

You need to add information about the session_id to all events.

What should you configure for the extended event session?

**Options:** not saved (only the answer was saved)

**✅ Answer:** action

<details><summary>Why</summary>

An action is a programmatic response or series of responses to an event. Actions are bound to an event, and each event may have a unique set of actions. Actions are global fields. Configuring an action allows you to add the required information to all events.

Targets process events, predicates are a set of logical rules used to evaluate events when they are processed, and a package is a container for Microsoft SQL Server Extended Events objects.

SQL Server Extended Events Packages - SQL Server - Training | Microsoft Learn

Describe performance monitoring - Training | Microsoft Learn

</details>

---

### Q47 ⚠️

**Question:**
You have a SQL Server 2019 instance.

You need to enable Extended Events to monitor deadlocks.

Which T-SQL Statement should you run?

**Options:** not saved (only the answer was saved)

**✅ Answer:** 
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

<details><summary>Why</summary>

The event_file target will enable historical troubleshooting.

The event counter target merely counts how many times each specified event occurs and the histogram target counts occurrences of different types of items.

Targets for Extended Events in SQL Server - SQL Server - Training | Microsoft Learn

Explore extended events - Training | Microsoft Learn

</details>

---

### Q48 ⚠️

**Question:**
You have a Microsoft SQL Server instance installed on an Azure virtual machine named SQLVM1.

You need to analyze SQL Server engine counters, such as buffer cache and batch requests per second.

What should you use to collect these metrics?

**Options:** not saved (only the answer was saved)

**✅ Answer:** Windows Performance Monitor (PerfMon) on SQLVM1

<details><summary>Why</summary>

Objective:

3.1 Monitor resource activity and performance

What This Item Tests:

Determine sources for performance metrics

Additional Reading:

Describe performance monitoring tools
Summary
Monitor database performance
Monitor and performance tuning in Azure SQL Database and Azure SQL Managed Instance
Monitor Azure SQL Managed Instance
Rationale:

Windows Performance Monitor (PerfMon) on SQLVM1 can collect SQL Server engine counters, such as the buffer cache hit ratio and batch requests per second, because it accesses SQL Server performance objects exposed through the Windows performance counter subsystem on the virtual machine host. Azure Monitor metrics for the Microsoft.Compute/virtualMachines resource expose only infrastructure metrics, such as CPU, disk, and network usage, rather than SQL Server engine counters. Query Performance Insight applies to Azure SQL Database rather than SQL Server on a virtual machine. The Azure Activity log records control-plane operations in Azure rather than database engine performance metrics.

</details>

---

### Q49

**Question:**
Your organization uses Azure SQL Database with Query Store enabled to monitor query performance.

You need to configure Query Store to capture queries with significant resource consumption while minimizing storage usage.

Which two actions should you perform? Each correct answer presents part of the solution. Choose 2.

**Options:** not saved (only the answer was saved)

**✅ Answer:** Enable size-based cleanup mode.; Set query capture mode to automatic.

<details><summary>Why</summary>

Turning off Query Store prevents query monitoring, which contradicts the goal of capturing queries with significant resource consumption. Enabling size-based cleanup mode ensures that less relevant data is automatically removed when storage limits are approached, helping to minimize storage usage while retaining critical information. Setting query capture mode to all queries results in excessive data collection, which increases storage usage and does not align with the goal of minimizing storage usage. Setting query capture mode to automatic focuses on capturing queries with significant resource consumption, aligning with the goal of monitoring performance while reducing unnecessary data collection.

Monitor performance by using the Query Store

</details>

---

### Q50 ⚠️

**Question:**
You have a database in an Azure SQL Managed Instance.

You need to control the MAXDOP configuration at the workload level. The solution must minimize administrative and development effort.

What should you do?

**Options:** not saved (only the answer was saved)

**✅ Answer:** In Resource Governor, set the MAX_DOP workload group configuration.

<details><summary>Why</summary>

Configure the MAX_DOP workload group configuration in Resource Governor to be able to control the degree of parallelism per defined workload. This option can be set in the CREATE WORKLOAD GROUP statement.

The ALTER DATABASE SCOPED CONFIGURATION statement manages the settings per database and not per workload. Using the MAXDOP hint on the query level requires code changes, and you cannot control the workload. Using the sp_configure stored procedure changes the settings at the server level, and you cannot control the workload.

ALTER DATABASE SCOPED CONFIGURATION - SQL Server (Transact-SQL) - Training | Microsoft Learn

CREATE WORKLOAD GROUP (Transact-SQL) - SQL Server - Training | Microsoft Learn

Configure databases for optimal performance - Training | Microsoft Learn

</details>

---

### Q51

**Question:**
You run the DBCC CHECKDB command for a production database. The command shows errors in several nonclustered indexes.

You need to repair the indexes and ensure the integrity of the database after the operation has complete.

Which parameter should you specify in the DBCC CHECKDB command?

**Options:** not saved (only the answer was saved)

**✅ Answer:** REPAIR_REBUILD

<details><summary>Why</summary>

Adding the REPAIR_REBUILD option to the DBCC CHECKDB command performs repairs that have no possibility of data loss. This option may include quick repairs, such as repairing missing rows in nonclustered indexes, and more time-consuming repairs, such as rebuilding an index.

The REPAIR_FAST option maintains syntax for backward compatibility only. No repair actions are performed. The REPAIR_ALLOW_DATA_LOSS option can introduce some data loss in the database.

DBCC CHECKDB (Transact-SQL) - SQL Server - Training | Microsoft Learn

Configure databases for optimal performance - Training | Microsoft Learn

</details>

---

## Configure and manage automation of tasks (13)

### Q52

**Question:**
You have an Azure SQL managed instance.

You need to ensure that you receive a notification when a SQL Server Agent job fails.

Which three actions should you perform? Each correct answer presents part of the solution.

**Options:**
- **A.** Create a Database Mail account.
- **B.** Create a Database Mail profile.
- **C.** Create a SQL Server Agent proxy.
- **D.** Create a SQL Server credential.
- **E.** Enable Database Mail.

**✅ Answer:** A, B, E

<details><summary>Why</summary>

To ensure that you receive a notification when a SQL Server Agent job fails, Database Mail needs to be enabled, a Database Mail account needs to be created, and a Database Mail profile needs to be created.

A SQL Server credential and a SQL Server Agent proxy are not required.

[Describe task status notifications - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/schedule-tasks-using-sql-server-agent/3-describe-task-status-notifications?ns-enrollment-type=learningpath&amp;ns-enrollment-id=learn.wwl.automate-tasks-sql-server)[Configure SQL Server Agent mail to use Database Mail - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/relational-databases/database-mail/configure-sql-server-agent-mail-to-use-database-mail?view=sql-server-ver16)

</details>

---

### Q53

**Question:**
You deploy a SQL Server on Azure Virtual Machines instance. 

You need to ensure that automated patching is enabled.

Which two cmdlets should you run? Each correct answer presents part of the solution.

**Options:**
- **A.** New-AzVMConfig
- **B.** New-AzVMSqlServerAutoPatchingConfig
- **C.** Set-AzVMSqlServerExtension
- **D.** Set-AzVMCustomScriptExtension

**✅ Answer:** B, C

<details><summary>Why</summary>

The New-AzVMSqlServerAutoPatchingConfig cmdlet allows you to create a configuration object for automatic patching on a virtual machine, and the Set-AzVMSqlServerExtension cmdlet establishes a maintenance window for a SQL Server on Azure Virtual Machines instance. Automated updates can only be installed during this maintenance window.

Set-AzVMCustomScriptExtensionUpdate  Adds a custom script extension to a virtual machine., and the New-AzVMConfig cmdlet creates a configurable local virtual machine object for Azure.

[Set-AzVMSqlServerExtension (Az.Compute) - Training | Microsoft Learn](https://learn.microsoft.com/powershell/module/az.compute/set-azvmsqlserverextension?view=azps-9.2.0)

[Automate deployment of database resources - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-automatic-deployment-azure-sql-database/)

</details>

---

### Q54

**Question:**
You have a bacpac file from an existing Microsoft SQL Server database. You copy the file to an Azure Storage account. 

You need to apply the bacpac file to an existing Azure SQL Database.

Which command should you run?

**Options:**
- **A.** az sql db create
- **B.** az sql db import
- **C.** az sql db restore
- **D.** az sql db update

**✅ Answer:** B

<details><summary>Why</summary>

You can use the az sql db import command to import the database from the bacpac file. The database must be created before the bacpac file is imported.

The az sql db restore command creates a new database from backup, and the az sql db update command updates the existing database.

[az sql db - Training | Microsoft Learn](https://learn.microsoft.com/cli/azure/sql/db?view=azure-cli-latest#az-sql-db-create)

[Automate deployment of database resources - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-automatic-deployment-azure-sql-database/)

</details>

---

### Q55

**Question:**
You have an Azure subscription.

You plan to deploy 100 Azure SQL Databases.

You need to automate the deployment. The solution must be parameterizable, repeatable, and be able to be versioned in source control.

Which three components can you use? Each correct answer presents a complete solution.

**Options:**
- **A.** Azure Cloud Shell
- **B.** Azure Policy
- **C.** Azure PowerShell scripts
- **D.** Azure Resource Manager (ARM) templates
- **E.** Bicep templates

**✅ Answer:** C, D, E

<details><summary>Why</summary>

ARM templates, Bicep templates, and PowerShell scripts can all be parameterized and versioned.

Cloud Shell is an interactive, authenticated, browser-accessible shell, and Azure Policy helps to enforce organizational standards and to assess compliance at-scale.

[Automate deployment by using Azure Resource Manager templates and Bicep - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/configure-automatic-deployment-azure-sql-database/3-automate-deployment-using-azure-resource-manager-bicep?ns-enrollment-type=learningpath&ns-enrollment-id=learn.wwl.automate-tasks-sql-server)

</details>

---

### Q56

**Question:**
You have an Elastic Job agent that will run tasks on an Azure SQL Database elastic pool. 

You create an elastic job. 

You need to ensure that the job runs every hour.

Which cmdlet should you run?

**Options:**
- **A.** Set-AzSqlDatabase
- **B.** Set-AzSqlElasticJob
- **C.** Set-AzSqlElasticJobStep
- **D.** Set-AzSqlElasticPool

**✅ Answer:** B

<details><summary>Why</summary>

Using the Set-AzSqlElasticJob cmdlet updates the elastic job properties. You can set the interval, as well as the start and end date of the job.

The Set-AzSqlElasticJobStep cmdlet updates the elastic job steps. You cannot modify the schedule at the step level. The Set-AzSqlElasticPool cmdlet modifies the properties of an Azure SQL Database elastic pool, and the Set-AzSqlDatabase cmdlet sets the properties of a database or moves an existing database into an elastic pool.

[Set-AzSqlElasticJob (Az.Sql) - Training | Microsoft Learn](https://learn.microsoft.com/powershell/module/az.sql/set-azsqlelasticjob?view=azps-9.2.0)

[Manage Azure PaaS tasks using automation - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/manage-azure-paas-resources-using-automated-methods/)

</details>

---

### Q57

**Question:**
You have an Azure subscription that includes 20 Azure SQL logical servers.

You need to automate management tasks using an Azure Automation runbook.

What should you do first?

**Options:**
- **A.** Create an Azure Automation account.
- **B.** Create an Azure Automation credential.
- **C.** Create an Azure Automation runbook schedule.
- **D.** Deploy an Azure Automation hybrid runbook runner.

**✅ Answer:** A

<details><summary>Why</summary>

To automate management tasks by using an Azure Automation runbook, you first need to create an Azure Automation account.

You cannot create a credential or a schedule without first creating an Azure Automation account, and you do not need a hybrid runbook runner.

[Build an automation runbook - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/manage-azure-paas-resources-using-automated-methods/4-build-automation-runbook)

</details>

---

### Q58

**Question:**
Your company uses Azure SQL Database elastic pools to manage resources for tenant databases.  
  
You use elastic jobs and need to ensure a scheduled job targets specific databases within the pool.  
  
What should you use?

**Options:**
- **A.** Create separate elastic pools.
- **B.** Define a target group.
- **C.** Manually execute the job.
- **D.** Run the job on the entire pool.

**✅ Answer:** B

<details><summary>Why</summary>

Defining a target group allows precise control over job execution by including or excluding specific databases, making it the most effective solution for this scenario. Creating separate elastic pools unnecessarily increases complexity and cost, which is not aligned with the goal of efficient resource management. Manually executing the job is inefficient and unsuitable for scheduled or recurring tasks. Running the job on the entire pool does not fulfill the requirement to target specific databases, as it lacks the necessary selectivity.  
  
[Explore Elastic jobs - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/manage-azure-paas-resources-using-automated-methods/2-explore-elastic-jobs)  
[Design Azure SQL Database for cloud-native applications - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/prepare-to-maintain-sql-databases-azure/4-design-azure-sql-database-for-cloud-native-applications)

</details>

---

### Q59

**Question:**
Your company has multiple Azure SQL Databases across different regions.  
  
You need to automate the execution of T-SQL statements across all databases.  
  
Which three actions should you perform? Each correct answer presents part of the solution. Choose 3.

**Options:**
- **A.** Create an elastic job agent.
- **B.** Define job steps.
- **C.** Execute T-SQL statements manually.
- **D.** Schedule the elastic job.
- **E.** Use graphical runbooks.

**✅ Answer:** A, B, D

<details><summary>Why</summary>

Creating an elastic job agent is essential for automating the execution of T-SQL statements across multiple databases, addressing the need for scalability and efficiency. Defining job steps ensures that the tasks to be performed are clearly specified, making the automation process well-structured. Scheduling the elastic job guarantees that tasks are executed automatically at regular intervals, aligning with the automation goal. Executing T-SQL statements manually contradicts the automation objective, as it requires human effort and lacks scalability. Graphical runbooks are not designed for this purpose, making them unsuitable for the scenario.  
  
[Understand Azure Automation - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/manage-azure-paas-resources-using-automated-methods/3-understand-azure-automation)  
[Automate deployment by using PowerShell - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/configure-automatic-deployment-azure-sql-database/4-automate-deployment-using-powershell)

</details>

---

### Q60

**Question:**
Your organization uses Azure SQL Database for critical applications. The database performance is affected by outdated indexes and stale data.  
  
You need to automate database maintenance tasks to improve performance.  
  
Which three actions should you perform? Each correct answer presents part of the solution. Choose 3.

**Options:**
- **A.** Create a PowerShell runbook.
- **B.** Enable managed identities.
- **C.** Manually trigger the runbook.
- **D.** Schedule the runbook.
- **E.** Use Logic Apps.

**✅ Answer:** A, B, D

<details><summary>Why</summary>

Creating a PowerShell runbook is essential for automating database maintenance tasks, addressing the need for improved performance. Scheduling the runbook ensures these tasks are executed regularly without manual intervention, aligning with the automation goal. Enabling managed identities provides secure and simplified authentication for accessing Azure resources, which is critical for maintaining security during automation. Manually triggering the runbook contradicts the automation objective, as it requires human intervention. While Logic Apps can automate workflows, they are not the optimal choice for this specific task, making them less suitable for the scenario.  
  
[Understand Azure Automation - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/manage-azure-paas-resources-using-automated-methods/3-understand-azure-automation)  
[Automate deployment by using PowerShell - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/configure-automatic-deployment-azure-sql-database/4-automate-deployment-using-powershell)  
[Add authentication to manage Azure resources](https://learn.microsoft.com/en-us/azure/automation/learn/automation-tutorial-runbook-textual#add-authentication-to-manage-azure-resources)

</details>

---

### Q61

**Question:**
You have an Azure subscription that contains a resource group named RG1.

You need to create an Azure SQL Managed Instance in RG1 by using PowerShell.

Which cmdlet should you run?

**Options:** not saved (only the answer was saved)

**✅ Answer:** New-AzSqlInstance

<details><summary>Why</summary>

By using the New-AzSqlInstance cmdlet, you can create a new SQL managed instance.

New-AzSQLServer creates a new Microsoft SQL server, New-AzSqlDatabase creates a new Azure SQL Database, Set-AzSqlInstance sets the properties for a SQL managed instance, and Set-AzSqlServer sets the properties for a Microsoft SQL server.

New-AzSqlInstance (Az.Sql) - Training | Microsoft Learn

Automate deployment of database resources - Training | Microsoft Learn

</details>

---

### Q62

**Question:**
You have an Azure subscription that contains a resource group named RG1. RG1 is hosted in the West US Azure region.

You need to create a new Azure SQL server named srv1.

Which Azure CLI command should you run?

**Options:** not saved (only the answer was saved)

**✅ Answer:** az sql server create -l westus -g rg1 -n srv1 -u adminuser -p adminpassword

<details><summary>Why</summary>

Using the az sql server create -l westus -g rg1-n srv1 -u adminuser -p adminpassword command will create a new Azure SQL server.

The az sql db create -g rg1-s srv1-n mydb --service-objective S0 command creates a new Azure SQL Database, the az sql mi create -g rg1-n srv1-l westus -i -u adminuser -p adminpassword command will create a new Azure SQL Managed Instance, and the az sql elastic-pool create -g rg1-s srv1-n db1 command will create a new Azure SQL Database elastic pool.

az sql server | Microsoft Learn

Automate deployment of database resources - Training | Microsoft Learn

</details>

---

### Q63

**Question:**
You plan to deploy multiple Azure SQL Databases for a new application. Automation is required for consistency and efficiency.

You need to determine the tools and methods to automate the deployment of these databases.

Each correct answer presents part of the solution. Which three actions should you take?

**Options:** not saved (only the answer was saved)

**✅ Answer:** Add deployment scripts to an Azure DevOps pipeline.; Define database configurations using ARM templates.; Run deployment scripts using the Az.Sql PowerShell module.

<details><summary>Why</summary>

Automating the deployment of Azure SQL Databases requires tools and methods that ensure consistency and efficiency. Adding deployment scripts to an Azure DevOps pipeline facilitates automated processes and supports continuous delivery. Defining database configurations using ARM templates provides a declarative approach to deploying Azure resources, ensuring repeatability and alignment with automation requirements. Running deployment scripts using the Az.Sql PowerShell module utilizes cmdlets tailored for managing Azure SQL resources, enabling efficient scripting and automation. Using a third-party tool not integrated with Azure introduces unnecessary complexity and fails to leverage native capabilities. Manually configuring each database in the Azure portal is time-consuming and prone to human error, contradicting the automation requirement. Deploying databases interactively using SSMS is primarily suited for managing SQL Server instances and does not support the automation needed for efficient Azure SQL Database deployments.

Automate deployment by using PowerShell - Training | Microsoft Learn
Explore single SQL database - Training | Microsoft Learn

</details>

---

### Q64

**Question:**
You have an Azure subscription. The subscription contains several Azure SQL Databases that support departmental applications.

You plan to use elastic jobs to periodically execute T-SQL-based maintenance tasks against all the databases.

What should you do first?

**Options:** not saved (only the answer was saved)

**✅ Answer:** Create an Azure SQL Database.

<details><summary>Why</summary>

You need to create an Azure SQL Database to keep the elastic job configuration.

The Elastic Job agent can be configured once the database and the elastic job configuration is prepared.

You cannot create Elastic Job targets until you have an agent.

You do not need to create an elastic pool server for this task.

Job automation overview with Elastic Jobs - Azure SQL Database - Training | Microsoft Learn

</details>

---

## Plan and configure a high availability and disaster recovery (HA/DR) environment (15)

### Q65

**Question:**
You need to recommend a high-availability and disaster recovery (HA/DR) strategy for a SQL Server on Azure Virtual Machines instance. The solution must meet the following requirements:

**Options:**
- **A.** All databases, including system databases, must be protected.
- **B.** SQL Server Agent jobs must be protected.
- **C.** Downtime must be minimized.
- **D.** The failover process must be done automatically.

What should you include in the recommendation?
- **E.** an Always On availability group
- **F.** an Always On failover cluster instance
- **G.** log shipping
- **H.** transactional replication

**✅ Answer:** F

<details><summary>Why</summary>

You should recommend an Always On failover cluster instance because this feature protects the entire instance. A SQL Server on Azure Virtual Machines instance is the entire installation of Microsoft SQL Server, such as binaries and all the objects inside the instance, including logins, SQL Server Agent jobs, and databases. Instance-level protection means that the entire instance is accounted for in the availability feature.

An Always On availability group works at the database level rather than the instance level. Transactional replication cannot replicate system databases, and neither transactional replication nor log shipping allow you to perform automatic failover.

[Always On failover cluster instances - SQL Server Always On - Training | Microsoft Learn](https://learn.microsoft.com/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server?view=sql-server-ver16)

[Describe high availability and disaster recovery strategies - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/describe-high-availability-disaster-recovery-strategies/)

</details>

---

### Q66

**Question:**
You have 100 on-premises Microsoft SQL servers.

You plan to migrate the servers to SQL Server on Azure Virtual Machines.

You need to recommend a high-availability solution that will maintain availability if a datacenter fails.

What should you include in the recommendation?

**Options:**
- **A.** availability sets
- **B.** availability zones
- **C.** Azure Backup
- **D.** Azure Site Recovery

**✅ Answer:** B

<details><summary>Why</summary>

You should recommend availability zones, as they account for datacenter-level failure in Azure.

Availability sets provide uptime against Azure-related maintenance and single points of failure in a single datacenter, Azure Site Recovery replicates workloads running on physical and virtual machines from a primary site to a secondary location, and Azure Backup is used to provide independent and isolated backups to guard against the unintended destruction of the data on your virtual machines.

[Describe Azure high availability and disaster recovery features for Azure Virtual Machines - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/describe-high-availability-disaster-recovery-strategies/4-describe-azure-high-availability-disaster-recovery-features)

</details>

---

### Q67 ⚠️

**Question:**
You need to recommend a high availability and disaster recovery (HADR) solution for an Azure SQL managed instance named SQLMI1. The solution must meet the following requirements:

**Options:**
- **A.** Support automatic failover between Azure regions.
- **B.** Keep the database synchronized across regions.
- **C.** Support failback to the primary region.
- **D.** Be scalable and cost-effective.

What should you include in the recommendation?
- **E.** active geo-replication
- **F.** Always On availability groups
- **G.** failover groups
- **H.** transactional replication

**✅ Answer:** G

<details><summary>Why</summary>

Failover groups are the best choice because they are designed for Azure SQL Managed Instance, supporting synchronous replication within a region and asynchronous replication across paired regions, along with automatic failover and failback. Always On availability groups are only for on-premises or Azure VMs and require Windows Server Failover Clustering, which is not available for PaaS. Active geo-replication applies to Azure SQL Database, not managed instances, and lacks automatic failover. Transactional replication is intended for data distribution or reporting and does not provide full synchronization or automatic failover.  
  
[Explore auto-failover groups for Azure SQL Database and Azure SQL Managed Instance](https://learn.microsoft.com/en-us/training/modules/explore-iaas-paas-platform-tools-for-high-availability-disaster-recovery/6-explore-auto-failover-groups)

</details>

---

### Q68

**Question:**
You have an availability group on a SQL Server on Azure Virtual Machines instance. 

You need to add an existing database to the availability group. 

You perform a full backup of the database on the primary node of the availability group. 

Which restore command should you run on the secondary replica to ensure that you can join the database to the availability group?

**Options:**
- **A.** RESTORE WITH NORECOVERY
- **B.** RESTORE WITH PARTIAL
- **C.** RESTORE WITH RECOVERY
- **D.** RESTORE WITH STANDBY

**✅ Answer:** A

<details><summary>Why</summary>

You need to restore the database by using the NORECOVERY option on the secondary replica before you can join the database to the availability group. 

Restoring the database by using the RECOVERY and STANDBY options does not allow it to join the availability group. Restoring the database by using PARTIAL is used if you need to perform a piecemeal restore.

[Prepare a secondary database for an availability group - SQL Server Always On - Training | Microsoft Learn](https://learn.microsoft.com/sql/database-engine/availability-groups/windows/manually-prepare-a-secondary-database-for-an-availability-group-sql-server?view=sql-server-ver16)

[Back up and restore databases - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/backup-restore-databases/)

</details>

---

### Q69

**Question:**
You have a SQL Server on Azure Virtual Machines instance. 

During a weekly maintenance task, you discover that a database named db1 has consistency errors. 

You plan to restore the database from backups.

Which command should you run before you run the RESTORE command?

**Options:**
- **A.** ALTER DATABASE db1 SET EMERGENCY
- **B.** ALTER DATABASE db1 SET OFFLINE
- **C.** ALTER DATABASE db1 SET RESTRICTED\_USER
- **D.** ALTER DATABASE db1 SET SINGLE\_USER

**✅ Answer:** D

<details><summary>Why</summary>

You should run the ALTER DATABASE db1 SET SINGLE\_USER command, as only one user at a time can access the database. If you specify SINGLE\_USER, and another user connects to the database, the ALTER DATABASE command is blocked until all users disconnect from the specified database. This is the command that should be run before the RESTORE command is run.

Running the ALTER DATABASE db1 SET EMERGENCY command would not work because the database is marked as READ\_ONLY, logging is disabled, and access is limited to members of the sysadmin fixed server role.

By running ALTER DATABASE db1 SET OFFLINE, the database will shut down, and you cannot perform the restore operation. You cannot modify a database while it is offline.

Running the ALTER DATABASE db1 SET RESTRICTED\_USER command allows for only members of the db\_owner fixed database role dbcreator, and fixed server roles to connect to the database.

[ALTER DATABASE SET Options (Transact-SQL) - SQL Server - Training | Microsoft Learn](https://learn.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-set-options?view=sql-server-ver16)

[Back up and restore databases - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/backup-restore-databases/)

</details>

---

### Q70

**Question:**
You have an Azure SQL Database named db1. 

You need to configure a backup strategy to ensure that the db1 has backups retained for six years.

What should you configure?

**Options:**
- **A.** differential backup frequency
- **B.** long-term retention
- **C.** point-in-time-restore

**✅ Answer:** B

<details><summary>Why</summary>

A long-term retention policy allows you to specify how many weeks, months, and years a backup should be kept. The maximum time is 10 years.

A point-in-time-restore policy only allow you to keep backups for 35 days, and a differential backup frequency policy specifies how often you want differential backup to be taken.

[Azure SQL Database: Manage long-term backup retention - Azure SQL Database - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/database/long-term-backup-retention-configure?view=azuresql&amp;tabs=portal)

[Back up and restore databases - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/backup-restore-databases/)

</details>

---

### Q71

**Question:**
You have a Microsoft SQL Server 2019 instance. 

You are configuring a backup maintenance solution.

You need to ensure that the databases can be restored to any point in time.

Which two types of backups should you include? Each correct answer presents part of the solution.

**Options:**
- **A.** backup copy-only
- **B.** file
- **C.** filegroup
- **D.** full database
- **E.** transaction log

**✅ Answer:** D, E

<details><summary>Why</summary>

To configure a backup maintenance solution and ensure that the databases can be restored to any point in time, a full database backup and a transaction log backup are required.

A copy-only backup is a SQL Server backup that is independent of the sequence of conventional SQL Server backups. A file backup contains all the data in one or more files, and a filegroup backup contains all the data in one or more filegroups.

[Back up and restore SQL Server running on Azure virtual machines - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/backup-restore-databases/2-backup-restore-sql-server-running-azure-virtual-machines)

</details>

---

### Q72

**Question:**
You have an Azure SQL Managed Instance that hosts five databases.

You need to configure disaster recovery for the databases in the event of a regional outage. The solution must not create additional read-only replicas or manual failover configurations.

What can you use?

**Options:**
- **A.** active geo-replication
- **B.** Always On availability groups
- **C.** auto-failover groups
- **D.** long-term backup retention

**✅ Answer:** C

<details><summary>Why</summary>

The auto-failover groups feature allows you to manage the replication and failover of some or all databases on a logical server to another region, which allows you to configure high availability in the event of a regional outage.

Active geo-replication creates a secondary database replica in another region, but it is asynchronously kept up-to-date for Azure SQL Database. The Always On availability groups feature is a high-availability and disaster recovery (HA/DR) solution, but it is not for Azure SQL Database. Long-term backup retention stores database backups in Azure Blob storage.

[Explore auto-failover groups for Azure SQL Database and Azure SQL Managed Instance - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/explore-iaas-paas-platform-tools-for-high-availability-disaster-recovery/6-explore-auto-failover-groups)

</details>

---

### Q73

**Question:**
You have an Azure subscription that contains four virtual machines. All the virtual machines have Microsoft SQL Server 2019 installed.

A distributed availability group named DG1 has a replica on each virtual machine.

You need to enable monitoring for DG1. The solution must help diagnose performance issues.

What should you use?

**Options:**
- **A.** Extended Events
- **B.** Query Performance Insight
- **C.** SQL Database Advisor

**✅ Answer:** A

<details><summary>Why</summary>

Extended Events is a comprehensive solution for monitoring any product in the Azure SQL family.

Query Performance Insight provides intelligent query analysis for single and pooled databases but cannot provide information about a distributed availability group.

SQL Database Advisor provides performance recommendations but cannot provide information about a distributed availability group.

[Monitoring and performance tuning - Azure SQL Database - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/database/monitor-tune-overview?view=azuresql)

</details>

---

### Q74

**Question:**
You have an Azure subscription that contains five virtual machines. The virtual machines host a Microsoft SQL Server 2019 Always On availability group named AG1.

You need to enable monitoring for AG1. The solution must ensure that you can monitor the health of AG1 and diagnose performance issues.

What should you use?

**Options:**
- **A.** Intelligent Insights
- **B.** Query Performance Insight
- **C.** SQL Database Advisor
- **D.** Extended Events

**✅ Answer:** D

<details><summary>Why</summary>

Extended events allows you to monitor the health of AG1 and diagnose any performance issues.

Query Performance Insight provides intelligent query analysis for single and pooled databases, SQL Database Advisor provides performance recommendations, and Intelligent Insights uses built-in intelligence to continuously monitor database usage through artificial intelligence (AI) and detect disruptive events that cause poor performance.

[Azure SQL Database and Azure SQL Managed Instance monitoring and tuning documentation - Azure SQL - Training | Microsoft Learn](https://learn.microsoft.com/azure/azure-sql/database/monitoring-tuning-index?view=azuresql&amp;viewFallbackFrom=auresql)

[Explore IaaS and PaaS solutions for high availability and disaster recovery - Training | Microsoft Learn](https://learn.microsoft.com/training/modules/explore-iaas-paas-platform-tools-for-high-availability-disaster-recovery/)

</details>

---

### Q75

**Question:**
You have an Azure SQL Database that uses the General Purpose service tier.

You need to ensure that there are at least two readable copies of the database, and the database is available if a datacenter becomes unavailable.

What should you include in the recommendation?

**Options:** not saved (only the answer was saved)

**✅ Answer:** active geo-replication

<details><summary>Why</summary>

Active geo-replication is a feature that lets you create a continuously synchronized, readable secondary database for a primary database. The readable secondary database can be in the same Azure region as the primary database, or, more commonly, in a different region.

The failover groups allow you to manage the replication and failover of some or all databases on a logical server to another region, but do not allow you to create multiple Azure SQL Database geo-secondary replicas. Automated backups do not let you have two readable copies of the database available, and elastic pools are used for managing and scaling multiple databases that have varying and unpredictable usage demands. The databases in an elastic pool are on a single server and share a set number of resources.

Active geo-replication - Azure SQL Database - Training | Microsoft Learn

Explore IaaS and PaaS solutions for high availability and disaster recovery - Training | Microsoft Learn

</details>

---

### Q76

**Question:**
You have an on-premises Microsoft SQL Server 2019 server that hosts 10 databases that is up to date with patching. You are required to replicate all of the databases to the cloud.

Which two options could you recommend? Each correct answer presents a complete solution.

**Options:** not saved (only the answer was saved)

**✅ Answer:** a SQL Server on Azure Virtual Machines instance; An Azure SQL Managed Instance link

<details><summary>Why</summary>

A SQL Server on Azure Virtual Machines is required for hybrid high-availability and disaster recovery (HA/DR).

Azure SQL Managed Instance link enables near real-time data replication between SQL Server and Azure SQL Managed Instance.

Azure SQL Database, and an Azure SQL Database elastic pool are not the correct options, as they cannot be used with Always On availability groups.

High availability, disaster recovery, business continuity - SQL Server on Azure VMs - Training | Microsoft Learn

Plan and implement an high availability and disaster recovery environment - Training | Microsoft Learn

</details>

---

### Q77

**Question:**
You have an Azure SQL Managed Instance that contains a database named db1. You have a valid backup of db1.

You need to restore db1 to the point in time of the backupdb1 over the existing database.

Which two actions should you perform? Each correct answer presents part of the solution.

**Options:** not saved (only the answer was saved)

**✅ Answer:** Drop db1.; Restore db1 from a URL.

<details><summary>Why</summary>

You should drop the existing database first because the existing database cannot be overwritten on the SQL managed instance. You should also restore the database db1 from a URL.

Setting db1 to single-user mode is not allowed on the SQL managed instance, and while the SQL managed instance allows you to perform a manual database restore from a URL, it does not allow you to restore the database from a tape or disk.

Restore a database from a backup - Azure SQL Managed Instance - Training | Microsoft Learn

Back up and restore databases - Training | Microsoft Learn

</details>

---

### Q78 ⚠️

**Question:**
You have an Azure SQL logical server named SQL1 that hosts five databases.

You need to ensure that you can restore the databases if SQL1 is deleted.

What should you include in the solution?

**Options:** not saved (only the answer was saved)

**✅ Answer:** long-term backup retention

<details><summary>Why</summary>

Using long-term retention will allow you to restore the databases if the server is deleted.

The remaining options do not allow for a restore of the databases if a server is deleted. Accelerated database recovery is a redesigning of the Microsoft SQL Server database engine recovery process, enabling fast and consistent database recovery. Azure SQL Database elastic pools are a solution for managing and scaling multiple databases. The Azure Backup service provides simple, secure, and cost-effective solutions to back up your data and recover it from the Microsoft Azure cloud.

Long-term backup retention - Azure SQL Database & Azure SQL Managed Instance - Training | Microsoft Learn

Back up and restore databases - Training | Microsoft Learn

</details>

---

### Q79

**Question:**
You have an Always On availability group that contains three Microsoft SQL replicas.

You have a business process that runs on each replica. The process runs every hour and must be started only on the primary replica.

Which view should you query to obtain the necessary information about the state of each replica?

**Options:** not saved (only the answer was saved)

**✅ Answer:** sys.dm_hadr_availability_replica_states

<details><summary>Why</summary>

The sys.dm_hadr_availability_replica_states view returns a row for each local replica and a row for each remote replica in the same Always On availability group as a local replica. Each row contains information about the state of a given replica. The columns role and role_desc contain information about the description of the replica role.

sys.availability_groups returns a row for each availability group for which the local instance of SQL Server hosts an availability replica. sys.dm_hadr_database_replica_cluster_states returns a row that contains information intended to provide you with insights into the health of the availability databases in the Always On availability groups in each Always On availability group on a Windows Server Failover Cluster (WSFC) cluster. sys.availabilty_replicas returns a row for each availability replica that belongs to any Always On availability group in the WSFC failover cluster.

Monitor availability groups using Transact-SQL (T-SQL) - SQL Server Always On - Training | Microsoft Learn

sys.dm_hadr_availability_replica_states (Transact-SQL) - SQL Server - Training | Microsoft Learn

Explore IaaS and PaaS solutions for high availability and disaster recovery - Training | Microsoft Learn

</details>

---
