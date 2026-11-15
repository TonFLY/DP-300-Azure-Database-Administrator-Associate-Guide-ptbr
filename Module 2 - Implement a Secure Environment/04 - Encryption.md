# Criptografia

> **Habilidades cobradas:** Implementar transparent data encryption (TDE), Implementar criptografia de nível de objeto, Implementar Always Encrypted, Implementar Always Encrypted with secure enclaves
> **Tempo de leitura:** ~20 min

## Conceito central
A criptografia embaralha os dados para que apenas alguém com o **chave** certo possa lê-lo. Existem três lugares para proteger os dados:
- **Em repouso** (armazenado em disco) → **TDE**.
- **Em uso, para colunas escolhidas** (mesmo o DBA não deve vê-lo) → **Always Encrypted**.
- **Em trânsito** (movendo-se sobre a rede) → **TLS**.

Pense em um banco. TDE é o cofre trancado. Always Encrypted é um envelope selado que só o cliente pode abrir, até mesmo a equipe do banco não pode. TLS é a van blindada que movimenta dinheiro entre filiais.

## Transparent Data Encryption (TDE)
**TDE** criptografa todo o banco de dados **em disco**. Ele criptografa arquivos de dados, arquivos de log e **backups**.
- Os dados são criptografados quando escritos em disco (página por página) e descriptografados quando lidos na memória.
- É **transparente**: aplicativos precisam de **sem alterações de código**.
- Qualquer pessoa com SELECT direitos ainda vê dados normais. TDE faz **E** ocultar dados de usuários ou DBAs.
- Ele protege contra alguém **Roubando os arquivos ou backups** e abri-los em outro servidor.
- Algoritmos: AES_128, AES_192, AES_256. Azure SQL usa **AES-256**. TRIPLE_DES_3KEY (3DES) é obsoleto desde SQL Server 2016.

**Exemplo:** Um ladrão copia um arquivo backup de um banco de dados hospitalar. Sem a chave TDE, o arquivo é ilegível em qualquer outro servidor.

### TDE em Azure SQL
- **Por padrão** para novas bancos de dados: Azure SQL Database desde **Maio 2017**, Managed Instance desde **fevereiro 2019**. Os bancos de dados mais antigas devem ser ligadas à mão.
- Chave padrão: **serviço-gerenciado** (a Microsoft cria e gira).
- Ou **chave gerenciada pelo cliente (CMK)**também chamado **BYOK** (traze sua própria chave). A chave vive em **Azure Key Vault** (veja abaixo).
- Portal: banco de dados → **Transparent data encryption**.
- Criptografar um grande banco de dados leva tempo. Ele é executado em segundo plano com baixa prioridade.

### TDE em SQL Server (VM ou on-premises)
Você configurar-se com T-SQL. A cadeia de chaves é:
**Chave mestra de serviço** (feito na configuração SQL) → protege o **chave mestre banco de dados** → protege um **certificado** → protege o **chave de criptografia de banco de dados (DEK)** → criptografa os dados.

```sql
USE master;
CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<pwd>';
CREATE CERTIFICATE MyServerCert WITH SUBJECT = 'TDE cert';
USE TDE_Demo;
CREATE DATABASE ENCRYPTION KEY WITH ALGORITHM = AES_256
    ENCRYPTION BY SERVER CERTIFICATE MyServerCert;
ALTER DATABASE TDE_Demo SET ENCRYPTION ON;
```

- **Faça backup do certificado e de sua chave privada.** Sem eles, você não pode restaurar a criptografia backups Em qualquer lugar.
- **Always On availability groups:** copie o certificado para **Cada réplica**. Caso contrário, a banco de dados não pode ser lida após um failover.

### Criptografia de disco em Azure VMs
Para SQL Server em um VM você também pode criptografar todo o disco, como uma camada extra sob TDE.
- **Criptografia de disco do Azure (ADE)** se aposenta no **15 Sep 2028**. ADE habilitado VMs deve sair dele antes disso.
- A Microsoft recomenda o **criptografia no host** para o novo VMs.

## Chaves Azure Key Vault e TDE gerenciadas pelo cliente
**Azure Key Vault** é uma loja de nuvem segura para **Segredos** (senhas), **chaves** e **certificados**.
- As chaves são protegidas por software (nível **Padrão**) ou por **HSMs** (módulos de segurança de hardware: dispositivos especiais à prova de adulteração) no nível **Premium** ou **Gerenciado HSM**.
- Tem seu **próprio controle de acesso**. Mesmo um administrador de assinatura não pode ler chaves a menos que tenha acesso.
- Ele registra quem usou qual chave, para auditorias.

### Chave gerenciada pelo cliente (CMK) para TDE em Azure SQL
- A chave TDE (chamada de **protetor TDE**) fica no seu cofre. Você controla isso.
- Se você **delete ou desative** a chave, o banco de dados se torna **inacessível**. Esse é o seu "kill switch".
- O servidor alcança o Key Vault com um **managed identity**. Um **user-assigned managed identity** pode ser configurado antes que o servidor exista.
- **Rotação automática da chave:** o servidor verifica Key Vault para uma nova versão chave e muda para ele **dentro de 24 horas**. Somente o DEK é re-criptografado, online. Portal: **Auto-rotar chave** caixa de seleção; PowerShell `-AutoRotationEnabled`; CLI `--auto-rotation-enabled`.
- **Nível de banco de dados CMK** (somente Azure SQL Database, não MI): cada banco de dados pode ter sua própria chave. Ele precisa de um **user-atributed** managed identity (sistema atribuído não é suportado no nível do banco de dados). Ele suporta chaves em outro inquilino e por rotação automática da banco de dados.

**Exemplo:** Um banco deve provar que ele, e não a Microsoft, controla suas chaves de criptografia. Ele armazena a chave TDE em seu próprio Cofre Chave. Se o contrato terminar, ele desabilita a chave e os dados estão bloqueados.

### Vault chave com SQL Server em um VM
SQL Server em um VM (ou on-premises) pode manter suas chaves de criptografia TDE, backup e criptografia de nível celular no Key Vault. Isso usa **EKM** (Gerenciamento de Chave Extensível: um plug-in que permite o uso do SQL Server fora das lojas de chaves). Passos em resumo:
1. Crie um Cofre Chave e uma chave. Registre um aplicativo no Entra ID.
2. Dê ao aplicativo o papel **Key Vault Cripto Cripto Serviço de Criptografia Usuário** (ou antigos direitos de política de acesso: obter, lista, wrapKey, unwrapKey).
3. Instale o **Conector SQL Server para Azure Key Vault**, habilite o EKM, crie o provedor criptográfico.
4. Crie um **credencial** e um **chave assimétrica** do provedor.

```sql
CREATE CREDENTIAL sysadmin_ekm_cred
WITH IDENTITY = 'mykeyvault',                       -- Key Vault name
SECRET = '<ClientIdWithoutHyphens><ClientSecret>'
FOR CRYPTOGRAPHIC PROVIDER AzureKeyVault_EKM;

CREATE ASYMMETRIC KEY MyKey FROM PROVIDER [AzureKeyVault_EKM]
WITH PROVIDER_KEY_NAME = 'MyAzureKey', CREATION_DISPOSITION = OPEN_EXISTING;
```

- **SQL Server 2022 CU17 + em uma Azure VM** (ou SQL Server 2025 habilitado pelo Azure Arc) pode usar um **managed identity** em vez do registro do aplicativo. On-premises SQL Server ainda precisa de um service principal.

## Criptografia em nível de objeto (nível celular)
Você pode criptografar valores ou colunas únicas dentro do SQL Server usando **chaves simétricas** e certificados. O aplicativo (ou T-SQL) criptografa e descriptografa com funções no servidor.

**Exemplo:** Um banco de dados HR criptografa a coluna salarial com uma chave simétrica. Somente a equipe da folha de pagamento que pode abrir a chave pode descriptografá-la.

A desvantagem: o **servidor** detém as chaves e vê os valores simples ao descriptografar. Um poderoso DBA ainda pode lê-los. Para manter os dados ocultos até mesmo do DBA, use **Always Encrypted**.

## Always Encrypted
O **Always Encrypted** criptografa as colunas escolhidas **do lado do cliente** (dentro do driver de banco de dados do aplicativo).
- SQL Server **Nunca vê os dados simples** ou as chaves. Ele só armazena bytes criptografados.
- DBAs, administradores de nuvem e funcionários da Microsoft **não pode** ler os valores.
- Bom para números de cartão de crédito, números nacionais ID, dados de saúde.

### duas chaves

| Chave | O que faz | Onde ele mora |
|-----|-------------|----------------|
| **chave mestra da coluna (CMK)** | Protege as chaves de criptografia da coluna | Fora SQL: **Azure Key Vault**, Windows Certificate Store, ou um HSM |
| **Chave de encriptação da coluna (CEK)** | Criptografa os dados da coluna | Armazenado no banco de dados, mas **encriptado** pelo CMK |

O motorista do cliente lê o CEK criptografado da banco de dados, descriptografa-o com o CMK e depois descriptografa os dados. Somente o aplicativo pode chegar ao CMK.

Observação: E- Always Encrypted acr CMK (coluna mestre chave) é diferente de TDE's CMK (chave gerenciada pelo cliente). Leia a pergunta com cuidado.

### Dois tipos de criptografia

| Tipo | Como funciona | Pode consultar? |
|------|--------------|-----------|
| **determinística** | O mesmo valor → sempre o mesmo valor criptografado | Igualdade: `=`, junta-se, GROUP BY, índices. Mais fácil de adivinhar. |
| **Aleatorizado** | Valor mesmo valor → valor criptografado diferente cada vez | Mais seguro, mas sem pesquisa ou adesão (a menos que você use enclaves) |

**Exemplo:** Uma clínica criptografa `NationalID` com **determinista** para que a equipe possa procurar um paciente por ID exato. Ele criptografa `Diagnosis` com **randomizado** porque ninguém pesquisa nele.

### Como configurar isso
- Use o **assistente Always Encrypted** em SSMS (escolher colunas, escolher tipo, escolher loja chave), ou PowerShell, ou T-SQL para os metadados chave:

```sql
CREATE COLUMN MASTER KEY MyCMK
WITH (KEY_STORE_PROVIDER_NAME = 'AZURE_KEY_VAULT',
      KEY_PATH = 'https://<vault>.vault.azure.net/keys/<key>');
CREATE COLUMN ENCRYPTION KEY MyCEK
WITH VALUES (COLUMN_MASTER_KEY = MyCMK, ALGORITHM = 'RSA_OAEP',
             ENCRYPTED_VALUE = 0x01...);
```

- O aplicativo deve usar um driver compatível com Always Encrypted (e.g. ADO.NET / Microsoft.Data.SqlClient, ODBC, JDBC) com **`Column Encryption Setting=Enabled`** no connection string.
- Nunca guarde o CMK na máquina SQL Server. **Fazer backup do CMK**: se ele for perdido, os dados serão perdidos.

### Onde ajuda

| Configuração | O que protege contra |
|-------|--------------------------|
| Aplicativo On-premises + banco de dados on-premises | On-premises DBAs |
| app On-premises + Azure SQL | Administradores de nuvem (chaves ficar on-premises) |
| App e banco de dados no Azure | Ainda criptografado, mas os administradores do Azure podem gerenciar as chaves |

## Always Encrypted with secure enclaves
Always Encrypted simples tem um limite. O servidor não pode trabalhar nos dados simples, por isso só pode fazer colunas **Igualdade em Determinismo**. Pesquisas de alcance, LIKE, classificação e até mesmo criptografia de primeira vez devem acontecer no aplicativo cliente. Para uma grande tabela, isso significa puxar todos os dados para fora e empurrá-lo para trás.

A **secure enclave** Isso é um **área de memória protegida e bloqueada dentro do processo SQL Server**. Ninguém de fora pode olhar para dentro: não o DBANão o OS admin, não um depurador.
- O motorista do cliente envia as teclas de coluna **no enclave** através de um canal seguro.
- SQL Server entrega o trabalho criptografado ao enclave. O enclave descriptografa e computa **Apenas dentro de si mesmo**.

**Exemplo:** O aplicativo An HR deve encontrar funcionários com salário entre 40,000 e 45,000. O salário é criptografado com criptografia aleatória. Com um enclave, o SQL Server pode executar o `BETWEEN` dentro do enclave. O DBA ainda nunca vê um salário.

### O que os enclaves adicionam

| Capacidade | Always Encrypted | Com secure enclaves |
|---------|------------------------|----------------------|
| `=` | Apenas determinista | Determinaistic **e randomizados** |
| `<`, `>`, `BETWEEN`, `IN`, `LIKE`, `DISTINCT` | Não | Sim (colunas aleatórias) |
| Join | Igualdade determinista junta-se | Sim (SQL Server 2019: loop aninhado se junta apenas) |
| `ORDER BY`, `GROUP BY` | Não | Sim em Azure SQL Database e SQL Server 2022+ (não 2019) |
| Indexes on randomized columns | Não | Somente **Nonclustered** |
| Criptografar, girar chaves, descriptografar | Ferramenta cliente deve mover os dados | **No lugar** com `ALTER TABLE ... ALTER COLUMN` |

- Consultas ricas (intervalo, LIKE, tipo) precisam de criptografia **randomizado**. Uma coluna determinística habilitada para enclave ainda recebe **igualdade apenas**.
- O **Cript encriptação no local** funciona para ambos os tipos, se todas as chaves estiverem habilitadas para enclave.
- As colunas de texto em consultas de enclave precisam de um **Colação `_BIN2`** (Azure SQL Database e SQL Server 2022 também aceitam colações **UTF-8**).

### Dois tipos de enclave: VBS e Intel SGX

| | **Enclaves VBS** | **Enclaves Intel SGX** |
|--|------------------|------------------------|
| O que é | **Software**-based, usa o hypervisor do Windows (segurança baseada em virtualização). Nenhum hardware especial. | Baseado em **Hardware**, incorporado no Intel CPUs |
| SQL Server 2019 / 2022 / 2025 em Windows (on-premises ou VM) | **Sim – o único tipo** | Não |
| Azure SQL Database | Sim, no nível **qualquer** e modelo (vCore, DTU, serverless, elastic pools) **DC-série** | Apenas em hardware **DC-série** (vCore) |
| Protege contra | Ataques dentro do VM / convidado OS (e.g. DBA, depósitos de memória) | Ataques de hóspedes OS **e** host OS |
| Status | Atual, **Recomendado** | **Fim do apoio 31 Oct 2027** |

- **Managed Instance: não secure enclaves.** Plain Always Encrypted funciona em MI.
- **SQL Server no Linux: não secure enclaves.**
- Após 31 de outubro de 2027, o Azure move bancos de dados ainda em DC -série para série padrão e **Ativa os enclaves VBS**.
- Fatos da série DC, se perguntado: usa núcleos **físico**, núcleos **40** max, **serverless**, poucas regiões.

### Atestado
**Atestado** significa que o cliente verifica se o enclave é genuíno e sem obstáculos **antes** enviando-lhe chaves.

| Produto | Enclave | Atestado |
|---------|---------|-------------|
| SQL Server 2019+ | VBS | **Serviço de Guardião Anfitrião (HGS)** ou **Nenhum** |
| Azure SQL Database | SGX (série DC) | **Microsoft Azure Attestation (MAA)** – necessário |
| Azure SQL Database | VBS | Apenas **Nenhum** (teste não suportado) |

- **HGS** é um papel Windows Server. O host SQL Server está registrado com ele.
- **MAA**: crie um provedor de atestado, defina sua política, dê acesso ao servidor, compartilhe o **atestação URL** com a equipe do aplicativo.
- **Nenhum** é mais rápido de configurar, mas o cliente não pode detectar um enclave adulterado.
- Mantenha os papéis separados: o administrador do atestado não deve ser o DBA.

### Chaves habilitadas para Enclave
- **Enclave habilitado CMK:** tem a propriedade **`ENCLAVE_COMPUTATIONS`** e uma assinatura. Em SSMS, marque **Permitir cálculos enclave** quando você criar o CMK.
- **Enclave habilitado CEK:** qualquer CEK protegido por um CMK habilitado para enclave. Somente estes podem ser usados no enclave.
- O CMKs habilitado para Enclave pode viver apenas em **Windows Certificate Store** ou **Azure Key Vault**.

### Vire o enclave
**Azure SQL Database (VBS)** – desligado por padrão. Ele define **`preferredEnclaveType = VBS`** no banco de dados ou elastic pool.
- Portal: ao criar, guia **Segurança** → **Always Encrypted** → **Ativar secure enclaves = ON**. Para um banco de dados existente: guia **Segurança → Encriptação de Dados → Always Encrypted**. Para uma piscina existente: **Configuração → Configuração → Always Encrypted**.
- PowerShell: `Set-AzSqlDatabase ... -PreferredEnclaveType VBS`. CLI: `az sql db update ... --preferred-enclave-type VBS`.
- Bancos de dados adicionados a um pool VBS **herdar** VBS. Adicionar um banco de dados VBS a um pool não VBS é **não apoiado**.

**Azure SQL Database (SGX):** escolher hardware **DC-série** e configurar **MAA**.

**SQL Server 2019+ (VBS):** precisa de Windows Server 2019+ (ou Windows 11 24H2+), virtualização CPU e VBS em execução. No Azure, use um **Geração 2 VM**.

```sql
EXEC sp_configure 'column encryption enclave type', 1;   -- 0 = none, 1 = VBS
RECONFIGURE;
-- Restart the SQL Server instance, then check:
SELECT * FROM sys.dm_column_encryption_enclave;   -- one row = enclave ready
```

### Usá-lo
- Connection string: `Column Encryption Setting=Enabled;Attestation Protocol=None;` (ou `HGS` / `AAS` mais `Enclave Attestation Url`). Se **Protocolo de Atestado está faltando, enclaves estão fora**.
- SSMS: Conecte → Opções → **Always Encrypted** guia → Habilitar Always Encrypted → **Ativar secure enclaves** → escolher o protocolo.
- As consultas devem usar **parâmetros** (e.g. `WHERE Salary BETWEEN @Min AND @Max`).
- Exemplo de criptografia no local:

```sql
ALTER TABLE dbo.Employees
ALTER COLUMN SSN CHAR(11) COLLATE Latin1_General_BIN2
ENCRYPTED WITH (COLUMN_ENCRYPTION_KEY = CEK1, ENCRYPTION_TYPE = RANDOMIZED,
  ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256') NOT NULL
WITH (ONLINE = ON);
```

### Limites para saber
- Não clustered index, chave primária, chave estrangeira ou restrição única em enclave habilitado **randomizado** colunas.
- Não para memory-optimized tables.
- Ligue o **Accelerated Database Recovery (ADR)** antes de indexar colunas de enclave aleatório. O ADR está ativado por padrão no Azure SQL, desativado por padrão no SQL Server.
- Cada **réplica** (availability group, geo-replica) deve suportar enclaves, ou as consultas de enclave falham após o failover.

## Conexões encriptadas (TLS)
**TLS** (Transport Layer Security) criptografa dados **enquanto se move** entre o aplicativo e o banco de dados. Sem ele, alguém na rede poderia ler senhas e consultar os resultados.

### Azure SQL Database e Managed Instance
- TLS é usado **por predefinição**.
- Set **Versão mínima TLS** (servidor → **Conectividade de rede →**, ou `az sql server update --set minimalTlsVersion="1.2"`).
- **TLS 1.0 e 1.1 se aposentaram em 31 de agosto de 2025.** O mínimo mais baixo que você pode definir é **1.2**. TLS 1.3 também é aceito.
- Depois de aplicar uma versão mínima, você não pode voltar ao padrão.
- No aplicativo: `Encrypt=True;TrustServerCertificate=False;`.

### SQL Server em um VM
1. Instale um **Certificado TLS** válido no servidor. Ele deve estar na loja local da máquina (ou usuário atual), não expirado e legível pela conta de serviço SQL Server. Se não, SQL Server **Pode falhar ao começar**.
2. Abrir **SQL Server Configuration Manager** → Rede Configuração → Protocolos para sua instância → Propriedades.
3. Guia **Certificado**: escolha o certificado. Guia **Bandeiras**: **Force Encryption = Yes**.
4. SQL Server **2022+** também tem **Encriptação Estrita Força** (TDS 8.0: toda a sessão é embrulhada em TLS; necessário para **TLS 1.3**).
5. **Reiniciar** SQL Server.

Em **SSMS 20+**, defina **Criptografia** na guia Login para **Obrigatório** ou **Estrita**. Marque **Certificado de servidor de confiança** apenas se precisar.

## Compare em um relance

| | TDE | Always Encrypted | TLS |
|--|-----|------------------|-----|
| Protectores | Banco de dados inteiro + backups no disco | Colunas Escolhidas | Dados na rede |
| Servidor vê dados simples? | Sim | **Não** | n/a |
| Oculta dados de DBA? | Não | **Sim** | Não |
| Mudanças de aplicativo? | Nenhum | Condutor + configuração de conexão | Connection string |
| Default em Azure SQL? | Sim | Não | Sim |

## Para lembrar na prova
- "Stolen backup ou disco" → **TDE**. "Ocultar dados de DBAs / administradores de nuvem" → **Always Encrypted**.
- TDE está ativado por padrão em Azure SQL; certificado backup é vital em SQL Server; copie o certificado para todas as réplicas AG.
- A chave TDE gerenciada pelo cliente vive em **Key Vault**; a exclusão ou desativação torna o banco de dados inacessível; auto-rotação dentro de **24 horas**.
- Always Encrypted: CMK protege CEK; CEK criptografa dados. O aplicativo precisa de `Column Encryption Setting=Enabled`.
- Deterministic = equality and joins. Randomized = safer, no queries without enclaves.
- Faixa / LIKE / ORDER BY em dados criptografados, ou criptografar uma grande coluna sem mover dados → **secure enclaves**, **randomizado**, chaves habilitadas para enclave.
- **VBS** = software, any Azure SQL DB nível, atestado **Nenhum**, o único tipo para SQL Server. **SGX** = DC-apenas séries, **MAA** necessário, aposentar-se **31 Oct 2027**. **HGS** = SQL Server Apenas.
- **Sem enclaves** em Managed Instance ou Linux.
- Enclave SQL Server: `sp_configure 'column encryption enclave type', 1` + **reiniciar**.
- TLS mínimo é **1.2** em Azure SQL (1.0/1.1 aposentado 31 Aug 2025).

## Teste rápido
1. Os auditores exigem que o DBAs nunca deve ver números de cartão de crédito, mas o aplicativo ainda deve pesquisar pelo número exato do cartão. Qual recurso e tipo?
   <details><summary>Resposta</summary>Always Encrypted com criptografia determinística. Ele esconde dados do DBAs e ainda permite pesquisas de igualdade.</details>
2. Você precisa de pesquisas LIKE em uma coluna criptografada no Azure SQL Database ao menor custo. O que você configura?
   <details><summary>Resposta</summary>Use Always Encrypted com VBS secure enclaves (preferredEnclaveType = VBS), randomized encryption, chaves habilitadas para enclave e Attestation Protocol = None. VBS funciona em todos os tiers.</details>
3. Uma empresa desativa sua chave TDE no Azure Key Vault. O que acontece com o banco de dados?
   <details><summary>Resposta</summary>Torna-se inacessível, porque o servidor não pode mais desbloquear a chave de criptografia do banco de dados.</details>
