# Autenticação

> **Habilidades cobradas:** Configurar autenticação Microsoft Entra ID para Azure SQL Database, Azure SQL Managed Instance e SQL Server, Configurar autenticação para SQL em Azure VMs e Azure SQL Managed Instance
> **Tempo de leitura:** ~10 min

## Conceito central
**Autenticação** responde "Quem é você?". **Autorização** responde "O que você tem permissão para fazer?". Esta lição é sobre a primeira pergunta. Pense em um prédio de escritórios: o guarda na porta verifica seu cartão ID (autenticação). Seu cartão-chave, em seguida, abre apenas alguns quartos (autorização).

## As duas principais maneiras de fazer login
SQL Server e Azure SQL suportam dois tipos principais de login.

- **SQL authentication.** Um nome de usuário e senha armazenados **dentro** SQL Server. Fácil, mas menos seguro. Cada senha deve ser gerenciada à mão.
- **Autenticação Microsoft Entra ID** (Entra ID era chamado Azure Active Directory, ou Azure AD, antes). Os usuários fazem login com sua conta de nuvem da empresa. Esta é a mesma conta que eles usam para o Azure portal e Microsoft 365.
- **Windows authentication.** Usado por on-premises SQL Server e SQL Server on Azure VMs. Os usuários fazem login com sua conta do Windows (Active Directory). Um VM pode se juntar à empresa Active Directory para logon único.

Por que o Entra ID é melhor:
- Um lugar para gerenciar pessoas. Se você **Desactivar** um usuário no Entra ID, eles perdem o acesso ao banco de dados de uma só vez.
- Suporta **MFA** (autenticação multifator: uma segunda verificação como um toque de telefone, SMS ou código único).
- Ele suporta **single sign-on** (logue uma vez, use muitos serviços) e **Acesso Condicional** (regras como "apenas de laptops da empresa").

**Exemplo:** Um trabalhador de loja deixa a empresa. Com SQL logins, o DBA deve encontrar e soltar o login em todos os servidores. Com o Entra ID, o HR desabilita uma conta e todas as paradas de acesso ao banco de dados.

## Qual opção funciona onde

| Plataforma | SQL auth | Entrada ID auth | Clássico Windows auth |
|----------|----------|---------------|--------------|
| SQL Server em Azure VM | Sim | Sim (SQL Server 2022+) | Sim (VM aderiu ao Active Directory) |
| Azure SQL Managed Instance | Sim | Sim | Nenhum domínio aderir |
| Azure SQL Database | Sim | Sim | Não |

Observação: Azure SQL Database e Managed Instance são serviços em nuvem. Eles usam **Entra ID** no lugar do on-premises Windows Active Directory. Em um VM, você controla tudo (é seu próprio SQL Server), então todas as três opções estão abertas para você.

## Configurar a autenticação Entra ID no Azure SQL Database e Managed Instance
O passo principal é definir um **Microsoft Entra admin** para o servidor.

- Configure-o no **Azure portal** (servidor lógico → **Configurações → Microsoft Entra ID** → set admin → Save), ou com **PowerShell** ou **Azure CLI**.
- Esta configuração é **servidor-lar**, não por banco de dados.
- O administrador do Entra obtém energia **sysadmin-like** em todas Os bancos de dados no servidor.
- **Práticas recomendadas:** fazer um Entra **grupo** o administrador, não uma pessoa. Se uma pessoa sai, o grupo ainda funciona.

**Exemplo:** Uma escola cria um grupo Entra chamado "SQL -Admins" com três DBAs. O grupo é definido como o administrador Entra. Quando um DBA sai, eles são apenas removidos do grupo.

### Autenticação somente Microsoft Entra
Você pode desligar o SQL authentication completamente. Isso é chamado **Autenticação somente Microsoft Entra**.

- Funciona em Azure SQL Database e Managed Instance.
- Ele também bloqueia o **SQL server admin** login (aquele feito com uma senha quando o servidor foi criado).
- Você deve definir o **Admin primeiro**.
- Ligue-o no portal, ou com:
  - `az sql server ad-only-auth enable` / `Enable-AzSqlServerActiveDirectoryOnlyAuthentication`
  - Managed Instance: `az sql mi ad-only-auth enable`
- Você também pode forçá-lo em muitos servidores com **Azure Policy**.
- Quem pode ligá-lo ou desligá-lo: **SQL Security Manager**, **Proprietário** ou **Contribuidor**. **SQL Server Contribuidor não pode** (isso mantém os deveres separados).

### Conectando-se com MFA de SSMS
Em SQL Server Management Studio (SSMS), escolha a opção de autenticação **Microsoft Entra MFA**. Seu nome antigo era "Universal com MFA".

## Entre ID em SQL Server (VM ou on-premises)
SQL Server 2022 e posterior também pode usar o Entra ID. Em suma:
- Registre uma aplicação Microsoft Entra e dê-lhe permissões.
- Criar e atribuir um certificado.
- Configure-o no Azure portal e, em seguida, crie o logins e os usuários.
- Conecte-se com o SSMS usando uma opção de entrada.

Você não precisa se lembrar de cada clique. Saiba que o **SQL Server 2022+** suporta Entra ID logins.

## Identidades para aplicações: service principals e managed identities
As pessoas não são as únicas que fazem login. Aplicativos e serviços do Azure também fazem login.

- **Service principal.** Uma identidade para um aplicativo, registrado no Entra ID. Prova-se com um **segredo** (como uma senha) ou um **certificado**. Você deve armazenar e girar esse segredo.
- **Managed identity.** Uma identidade que o Azure cria e gerencia para um recurso do Azure (a VMum aplicativo web, a SQL server). **Não senha** O Azure lida com o segredo para você.
  - **Sistema-atribuído:** vinculado a um recurso. É excluído quando o recurso é excluído.
  - **Atribuído pelo usuário:** um recurso separado do Azure. Você pode dá-lo a muitos recursos, e ele vive por conta própria.

**Exemplo:** Um aplicativo web lê ordens de Azure SQL Database. Com o auth SQL, a senha fica em um arquivo de configuração. Com um managed identity, o aplicativo não tem nenhuma senha. O DBA apenas cria um usuário de banco de dados para a identidade do aplicativo.

Onde o managed identities aparece neste módulo:
- Um aplicativo se conecta ao Azure SQL sem senha.
- A SQL server lê sua **Chave TDE** A partir Azure Key Vault (lição 4).
- A auditoria escreve para uma conta de armazenamento (lição 6).
- Microsoft Purview digitaliza um banco de dados (lição 6).

## Azure RBAC: gerenciamento do recurso, não dos dados
**Azure RBAC** (role-based access control) controla o que alguém pode fazer com o **Recursos do Azure**. Por exemplo: criar, excluir ou dimensionar um banco de dados. Ele faz **E** deixá-los ler os dados dentro.

| Papel | Pode fazer | Não pode fazer |
|------|--------|-----------|
| Contribuidor SQL DB | Criar e gerenciar bancos de dados | Ler dados |
| Contribuidor SQL Server | Criar, excluir e configurar servidores e bancos de dados | Ler dados |
| SQL Security Manager | Gerenciar configurações de segurança (auditoria, detecção de ameaças, autenticação apenas no intra) | Ler dados |

**Exemplo:** O engenheiro DevOps tem SQL DB Contributor. Ela pode implantar um novo banco de dados para a equipe de folha de pagamento. Ela ainda não pode executar `SELECT` na tabela salarial. Isso precisa de uma permissão **banco de dados**.

Isso é chamado **separação de deveres**: as pessoas que administram a infraestrutura não têm permissão automática para ver os dados.

## Compare em um relance

| Método | Senha armazenada? | Melhor para |
|--------|------------------|----------|
| SQL authentication | Sim, em SQL Server | Aplicativos antigos, testes rápidos |
| Intra ID usuário ou grupo | Não (Entra lida com isso) | Pessoas; suporta MFA |
| Service principal | Sim, um segredo ou certificado | Aplicativos fora do Azure, automação |
| Managed identity | Não | Recursos do Azure falando com Azure SQL ou Key Vault |
| Windows authentication | Não | SQL Server em um domínio-juntado VM |

## Para lembrar na prova
- Authentication = who you are. Authorization = what you can do.
- Prática recomendada: **Entre ID + MFA** e um Entra **grupo** como administrador do servidor.
- O administrador do Entra é definido **por servidor**, não por banco de dados.
- **Autenticação apenas intra** desliga todos os SQL logins, incluindo o administrador SQL server. Defina o administrador Entra primeiro.
- **SQL Server Contributor** não pode habilitar ou desabilitar a autenticação exclusiva do Microsoft Entra. **SQL Security Manager** pode.
- As funções do Azure RBAC gerenciam recursos; eles **Nunca** dão acesso a dados.
- "App no Azure, sem senhas" → **managed identity**.
- SSMS opção para MFA = **Microsoft Entra MFA** (nome antigo: Universal com MFA).

## Teste rápido
1. Uma empresa quer bloquear todo o logins baseado em senha para o seu Azure SQL Database, incluindo o administrador do servidor. O que ele deve habilitar?
   <details><summary>Resposta</summary>Autenticação somente Microsoft Entra. Desativa o SQL authentication completamente, mas um administrador do Entra deve ser definido primeiro.</details>
2. Um usuário tem a função de Colaborador SQL DB. Eles podem ler as linhas do cliente?
   <details><summary>Resposta</summary>Não. As funções do Azure RBAC gerenciam apenas o recurso. Ler dados precisa de permissões de banco de dados.</details>
3. Um aplicativo web no Azure deve se conectar ao Azure SQL Database sem senha armazenada. O que você deve usar?
   <details><summary>Resposta</summary>A managed identity. O Azure gerencia suas credenciais, portanto nada é armazenado em código ou configuração.</details>
