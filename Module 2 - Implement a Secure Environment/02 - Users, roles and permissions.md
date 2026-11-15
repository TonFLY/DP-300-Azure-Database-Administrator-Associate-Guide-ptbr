# Usuários, roles e permissões

> **Habilidades cobradas:** Configurar princípios de segurança, Criar usuários a partir de identidades Microsoft Entra, Configurar permissões de banco de dados e nível de objeto usando ferramentas gráficas, Aplicar o princípio do menor privilégio para todos os recursos protegidos, Gerenciar autenticação e autorização usando T-SQL
> **Tempo de leitura:** ~13 min

## Conceito central
Uma vez que alguém está logado, SQL Server decide o que eles podem tocar. Ele faz isso com **principais** (quem), **securáveis** (o que) e **permissões** (ações permitidas). Pense em um hotel. O hóspede é o principal. Os quartos são securáveis. As configurações do cartão de chave são permissões.

## Principais, Securáveis e Esquemas
- **Principal de segurança:** uma pessoa, grupo, aplicativo ou processo que pode pedir acesso e receber permissões.
- **Securável:** uma coisa protegida por permissões. Exemplos: um banco de dados, esquema, tabela, vista, procedimento armazenado.
- **Esquema:** uma pasta nomeada dentro de um banco de dados que agrupa objetos. Exemplo: `Sales.Customers` é a tabela `Customers` no esquema `Sales`.
  - Cada usuário tem um **esquema padrão**, geralmente `dbo`.
  - Melhor prática: sempre escreva o nome do esquema, e.g. `SELECT Name FROM Sales.Customers;`.

Os diretores vivem em dois níveis:

| Nível do servidor (inteiro) | Nível de banco de dados (um banco de dados) |
|-------------------------------|-------------------------------|
| Logins, funções do servidor | Usuários, funções de banco de dados, funções de aplicativos |

## Logins e usuários
- A **login** deixa você entrar no **servidor**. É armazenado no `master` banco de dados.
- A **usuário** deixa você entrar **uma banco de dados**. Um usuário geralmente é mapeado para um login.

**Exemplo:** A login é a chave para a porta da frente de um edifício. Um usuário é a chave para um apartamento dentro dele.

```sql
USE master;
CREATE LOGIN demo WITH PASSWORD = 'Pa55.w.rd';
USE WideWorldImporters;
CREATE USER demo FROM LOGIN demo;
```

### Usuários contidos
A **contained user** Vive apenas dentro do banco de dados. Tem **login** E- `master`Esta é a **Recomendado** caminho em Azure SQL Database. O banco de dados é mais fácil de mover, porque seus usuários viajam com ele.

## Criar usuários a partir de identidades Microsoft Entra
Use `FROM EXTERNAL PROVIDER` para apontar para um usuário, grupo ou aplicativo do Entra.

```sql
-- Contained database user for an Entra user (recommended in Azure SQL Database)
CREATE USER [dba@contoso.com] FROM EXTERNAL PROVIDER;

-- Server-level Entra login (in master)
CREATE LOGIN [nativeuser@contoso.com] FROM EXTERNAL PROVIDER;
```

- A entrada **logins** (`CREATE LOGIN ... FROM EXTERNAL PROVIDER`) está disponível em Azure SQL Database (virtual `master`), Managed Instance e SQL Server 2022+.
- Você deve estar conectado como uma conta **Entro** (por exemplo, o administrador do Entra) para criar usuários do Entra.

**Exemplo:** O aplicativo da web de uma loja usa um managed identity chamado `shop-web`. O DBA roda `CREATE USER [shop-web] FROM EXTERNAL PROVIDER;` e, em seguida, lhe dá direitos de leitura. Nenhuma senha é necessária.

## Funções: dar permissões a grupos, não pessoas
A **papel** é um grupo de diretores. Você concede permissões para o papel uma vez. Então você adiciona ou remove membros.

```sql
CREATE ROLE SalesReader;
ALTER ROLE SalesReader ADD MEMBER DP300User1;
ALTER ROLE SalesReader ADD MEMBER DP300User2;
GRANT SELECT, EXECUTE ON SCHEMA::Sales TO SalesReader;
```

Agora ambos os usuários podem ler tabelas e executar procedimentos no esquema `Sales`.

### Funções de banco de dados incorporadas

| Papel | O que permite |
|------|----------------|
| db_owner | Tudo no banco de dados |
| db_datareader | SELECT em todas as tabelas e vistas |
| db_datawriter | INSERT, UPDATE, DELETE em todas as tabelas e visualizações |
| db_ddladmin | Criar e alterar objetos |
| db_securityadmin | Gerenciar permissões (poder, cuidado) |
| db_accessadmin | Adicionar ou remover usuários do banco de dados |
| db_backupoperator | Fazer backup da banco de dados (SQL Server e MI; não concede nada em Azure SQL Database) |
| db_denydatareader / db_denydatawriter | Bloquear todas as leituras / todos os escritos |
| público | Cada usuário está nele; direitos mínimos |

### Funções de aplicação
An **papel de aplicação** é ligado por um **app**, com uma senha. O aplicativo recebe as permissões do papel apenas para essa sessão.

### Funções do servidor em SQL Server e Managed Instance

| Papel | O que permite |
|------|----------------|
| sysadmin | Tudo no servidor |
| serveradmin | Configurações do servidor, desligado |
| segurançaadmin | Gerencie logins e suas permissões |
| processadmin | Matar sessões em execução |
| setupadmin | Gerenciar servidores vinculados |
| graneladmin | Correr BULK INSERT |
| diskadmin | Gerenciar dispositivos backup |
| dbcreator | Criar, restaurar e soltar bancos de dados |
| público | Todos os login; direitos mínimos |

### Funções do servidor em Azure SQL Database
Azure SQL Database tem sysadmin **Não** que você pode usar. Tem **sete funções de servidor `##MS_...##` fixas** (também em SQL Server 2022+). Você **não pode** cria funções de servidor personalizadas.

| Papel | O que permite |
|------|----------------|
| ##MS_DatabaseConnector## | Conectar-se a bancos de dados |
| ##MS_DatabaseManager## | Criar e excluir bancos de dados |
| ##MS_LoginManager## | Criar e excluir logins |
| ##MS_SecurityDefinitionReader## | Leia definições de segurança |
| ##MS_DefinitionReader## | Ler definições de objetos |
| ##MS_ServerStateReader## | Ver estado do servidor (e.g. DMVs) |
| ##MS_ServerStateManager## | Gerenciar estado do servidor |

- Adicione membros com T-SQL no virtual `master`: `ALTER SERVER ROLE ##MS_ServerStateReader## ADD MEMBER [login];`.
- Não através do portal ou CLI.
- Somente o **servidor admin** ou **admin entra** pode mudar de associação.
- Os membros devem ser **logins** (SQL ou Entra). Entre **Grupos não são apoiados** aqui.
- As antigas funções de banco de dados `master` **loginger** (criar logins) e **dbmanager** (criar bancos de dados) ainda existem. A Microsoft agora recomenda **##MS_LoginManager##** e **##MS_DatabaseManager##** em vez disso.

## Permissões
As quatro permissões diárias em tabelas e vistas:

| Permissão | Permite-lhe |
|------------|----------|
| SELECT | Ler linhas |
| INSERT | Adicionar linhas |
| UPDATE | Alterar linhas |
| DELETE | Remover linhas |

Mais permissões:
- **CONTROL:** plenos direitos sobre o objeto, como um proprietário.
- **ALTER:** altera a definição do objeto.
- **EXECUTE:** executar um procedimento ou função.
- **Referências:** necessário para criar uma chave estrangeira que aponta para a tabela.
- **TAKE OWNERSHIP:** torna-se o proprietário.
- **VIEW DEFINITION:** ver o código do objeto.
- **VIEW CHANGE TRACKING:** ler dados change tracking (lição 7).

### GRANT, DENY, REVOKE
- **GRANT** dá uma permissão.
- **DENY** bloqueia uma permissão. **DENY sempre ganha** sobre GRANT.
- O **REVOKE** remove um GRANT ou DENY anterior (de volta ao neutro).

**Exemplo:** Ravi está em um papel que tem SELECT em `Sales.Orders`. O DBA também roda `DENY SELECT ON Sales.Orders TO Ravi;`. Ravi não pode ler a tabela. O DENY bate o GRANT do papel.

```sql
GRANT SELECT ON SCHEMA::Sales TO SalesReader;
DENY UPDATE ON Sales.Customers TO Intern1;
REVOKE UPDATE ON Sales.Customers FROM Intern1;  -- clears the deny
```

## Permissões com ferramentas gráficas
Em **SSMS**, clique com o botão direito do mouse em uma tabela, vista ou procedimento → **Propriedades → Permissões**. Escolha um usuário ou função, em seguida, marque **Subvenção** ou **Negar** ao lado de cada permissão. Você pode fazer o mesmo no banco de dados (**Propriedades → Permissões**) ou em um usuário (**Propriedades → Securables**). O resultado é o mesmo que os comandos T-SQL.

**Exemplo:** Em um banco de dados CRM, o DBA abre as permissões `CustomerView` no SSMS. Ela marca Grant para SELECT e Deny para UPDATE para o papel " SalesTeam ". A equipe de vendas pode ler, mas não mudar os clientes.

## EXECUTE AS
`EXECUTE AS` executa o código como um usuário diferente por um tempo. Use `REVERT` para voltar.

```sql
EXECUTE AS USER = 'Tenant1';
SELECT * FROM Sales;   -- runs with Tenant1's rights
REVERT;
```

- Para testar o que um usuário pode ver.
- `EXECUTE AS USER` funciona em SQL Server, Azure SQL Database e Managed Instance.
- `EXECUTE AS LOGIN` funciona apenas em **SQL Server** e **Managed Instance**.

## Cadeia de propriedade
Se um procedimento armazenado e as tabelas que lê têm o **O mesmo proprietário**, um usuário só precisa de **EXECUTE** no procedimento. Eles não precisam de direitos sobre as tabelas.

**Exemplo:** Um usuário tem EXECUTE em `dbo.GetProducts`. O procedimento lê `Production.Product`. Ambos são de propriedade do `dbo`. O usuário executa o procedimento e funciona. Se o usuário executa `SELECT * FROM Production.Product` diretamente, ele falha.

**O SQL dinâmico quebra a cadeia.** Dynamic SQL é texto SQL construído e executado no tempo de execução, muitas vezes com `sp_executesql`. Então SQL Server verifica as próprias permissões **de chamador** nas tabelas.

```sql
DECLARE @sql NVARCHAR(MAX) = N'SELECT * FROM Production.Product';
EXEC sp_executesql @sql;   -- caller needs SELECT on the table
```

## Credenciais (T-SQL)
- **credencial servidor** (`CREATE CREDENTIAL`): um segredo armazenado usado para alcançar recursos externos. Pode ser usado por muitos bancos de dados.
- **Credencial de escopo de banco de dados** (`CREATE DATABASE SCOPED CREDENTIAL`): armazenado dentro de um banco de dados. Ele se move com o banco de dados. Não é mapeado para um login ou usuário.

**Exemplo:** Um banco de dados ETL lê arquivos do Azure Storage. Uma credencial com escopo de banco de dados possui a chave de armazenamento, para que o banco de dados possa se mover para outro servidor sem refazer a configuração do servidor.

## Menor privilégio
**Menor privilégio** significa: dar apenas as permissões necessárias para o job. Nada mais.

- Prefira **permissões pequenas e exatas** a papéis grandes como `db_owner`.
- Se um aplicativo executar apenas procedimentos, conceda apenas **EXECUTE**, não SELECT em tabelas.
- Use **papéis** e **grupos**, não doações únicas para pessoas.
- Mantenha o Azure RBAC (gerenciamento de recursos) além das permissões do banco de dados (acesso a dados).
- **Revisão** regularmente permissões. Defender para SQLA avaliação da vulnerabilidade pode sinalizar muitos direitos (lição 6).

**Exemplo:** Uma empresa de logística dá seu aplicativo de relatórios apenas SELECT nas tabelas de remessa. Mais tarde, o aplicativo é hackeado. O invasor pode ler os embarques, mas não pode excluir nada.

## Para lembrar na prova
- Login = server access (in `master`). User = database access.
- Azure SQL Database: prefira **usuários contidos** e `CREATE USER ... FROM EXTERNAL PROVIDER`.
- **DENY bate GRANT**. REVOKE apenas remove uma concessão ou negação.
- Grant para **papéis**, em seguida, adicionar membros.
- Azure SQL Database tem **7 funções de servidor `##MS_...##` fixas**, gerenciado somente com T-SQL em `master`. Sem funções de servidor personalizadas. Nenhum grupo de entrada como membros.
- `db_backupoperator` não dá nada em Azure SQL Database.
- A cadeia de propriedade funciona apenas para o **estática** SQL. O **Dynamic SQL** precisa de permissões diretas.
- `EXECUTE AS LOGIN` faz trabalho **E** em Azure SQL Database.
- O aplicativo só executa procedimentos → conceder apenas **EXECUTE**.

## Teste rápido
1. Um usuário tem SELECT através de um papel e DENY SELECT dado diretamente. Eles podem ler a tabela?
   <details><summary>Resposta</summary>Não. DENY sempre substitui GRANT.</details>
2. Você precisa de um usuário para ver DMVs no nível do servidor em Azure SQL Database. O que você faz?
   <details><summary>Resposta</summary>Adicione seu login ao # MS_ServerStateReader # com ALTER SERVER ROLE no banco de dados mestre virtual.</details>
3. Um procedimento usa sp_executesql para ler uma tabela. Os usuários com EXECUTE recebem um erro de permissão. Por quê?
   <details><summary>Resposta</summary>O SQL dinâmico quebra a cadeia de propriedade, então o chamador precisa do SELECT na tabela.</details>
