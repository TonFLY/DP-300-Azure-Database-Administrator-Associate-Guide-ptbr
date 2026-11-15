# Controles de conformidade

> **Habilidades cobradas:** Aplicar uma estratégia de classificação de dados, Configurar auditorias de servidor e banco de dados, Implementar dynamic data masking, Implementar livro contábil em Azure SQL, Implementar row-level security
> **Tempo de leitura:** ~17 min

## Conceito central
Leis como GDPR, HIPAA e PCI-DSS dizem que você deve saber onde os dados sensíveis estão, controlar quem vê e provar o que aconteceu com isso. Azure SQL dá-lhe uma caixa de ferramentas:
- Dados sensíveis **Localizar e etiquetar** Classificação de dados →, Microsoft Purview.
- **Esconder** ele → dynamic data masking, row-level security.
- **Gravar** que fez o que a auditoria →.
- **Prove** ninguém adulterou com ele → livro.
- **Cuidado com os ataques** → Microsoft Defender for SQL.

Pense em uma farmácia. Os rótulos marcam os medicamentos perigosos (classificação). A equipe vê apenas o estoque de sua própria loja (row-level security). O recibo até esconde a maior parte do número do cartão (mascaramento). Registros CCTV que entraram (auditoria). Um diário de bordo selado e numerado mostra se uma página foi arrancada. Um guarda de segurança vigia os ladrões (Defender).

## Classificação dos dados
**Classificação dos dados** significa marcar colunas com rótulos que dizem o quão sensíveis eles são.
- **Etiqueta da sensibilidade:** quão secreto é, e.g. Público, Geral, Confidencial, Altamente Confidencial.
- **Tipo de informação:** que tipo de dados é, e.g. E-mail, cartão de crédito, nacional ID.

**Exemplo:** Um aplicativo de saúde rotula `PatientName` como Confidencial e `MedicalHistory` como Altamente Confidencial. A equipe então sabe quais colunas precisam de criptografia ou mascaramento.

### Como classificar
- **Recomendações automáticas:** no Azure portal (banco de dados → **Segurança → Dados Descoberta e Classificação**) ou SSMS. O motor olha para **nomes de colunas** como "email", "dob", "ssn". Colunas mal nomeadas (como `col1`) serão perdidas.
- Portal **Manual:**, SSMS ou T-SQL.

```sql
ADD SENSITIVITY CLASSIFICATION TO Application.People.EmailAddress
WITH (LABEL = 'Confidential', INFORMATION_TYPE = 'Email');
```

### Onde os rótulos são armazenados
- SQL Server 2019+, Azure SQL Database e Managed Instance: vista do sistema **`sys.sensitivity_classifications`**.
- Older SQL Server: **propriedades estendidas** na coluna.

### Etiquetas e políticas personalizadas
- Faça seus próprios rótulos e tipos de informações no **Microsoft Defender for Cloud** (a política de proteção de informações). Você precisa de direitos no **grupo de gestão raiz**.
- Ou mude o locatário para a política **Proteção de Informação Microsoft Purview** para usar os rótulos de sensibilidade do Microsoft 365. Isso precisa do **Administrador de Segurança** em todo o locatário, e as recomendações automáticas são desativadas nesse modo.
- O **controle de acesso** para Azure SQL Database baseado em etiquetas é **aposentado**. Os rótulos são agora apenas para descoberta, relatórios e auditoria.

### Por que é importante
- Mostra onde os dados sensíveis vivem.
- Ajuda a planejar **Criptografia** e **mascaramento**.
- **Auditoria** pode gravar quando as colunas rotuladas são lidas (veja abaixo).

## Microsoft Purview
**Microsoft Purview** é um serviço **governança de dados** para o **organização inteira**, não apenas um banco de dados. Ele cobre on-premises, multi-nuvem (AWS, GCP) e fontes SaaS.
- Fontes de registro **Mapa e catálogo de dados:**, **scan**-los, e deixar as pessoas **search** para dados ("onde estão os e-mails do cliente?").
- O **Classificação:** encontra dados confidenciais automaticamente.
- **Linhagem:** mostra como os dados fluem da fonte para o destino (e.g. através de procedimentos armazenados e dutos).
- Relatórios **Insights:**, como quanto da propriedade é digitalizada ou contém dados confidenciais.

**Exemplo:** Uma empresa com bancos de dados 10,000 usa o Purview para encontrar todas as colunas que possuem números de cartão de crédito, em servidores Azure SQL, Power BI e on-premises.

### Digitalização Azure SQL Database
1. **Registro** o banco de dados em Purview.
2. Criar um **scan** (quais objetos, quais regras de classificação).
3. Teste a conexão e execute-a.

Autenticação para a varredura, melhor primeiro:
- **System-assigned managed identity** da conta Purview (simples, recomendado).
- User-assigned managed identity.
- **Service principal**.
- **SQL authentication** (menos recomendado).

Rede: permitir Purview através do firewall (Permitir serviços Azure), ou usar um **tempo de execução da integração auto-hospedada** ou um **rede virtual gerenciada**. Com um **Tempo de execução auto-hospedado, managed identities não funciona**: use um auth service principal ou SQL.

A linhagem de processo armazenado está em **pré-visualização**. A identidade Purview precisa de **db_owner**, o banco de dados precisa de uma chave mestra e "Permitir serviços Azure" deve estar ligado. Não funciona através de um tempo de execução VNet auto-hospedado ou gerenciado ou um private endpoint.

### Classificação vs Purview

| | Descoberta e classificação de dados | Microsoft Purview |
|--|------------------------------|-------------------|
| Scope | Um banco de dados | Organização inteira, muitos tipos de fonte |
| Recursos extras | Etiquetas, auditoria de acesso sensível | Catálogo, pesquisa, linhagem, insights |

## Auditoria
**Auditoria** registra eventos de banco de dados (logins, consultas, alterações) em um log. Ajuda na conformidade e na detecção de atividades incomuns.

### Para onde vão os logs

| Destino | Bom para |
|-------------|----------|
| **Conta Azure Storage** | Arquivo barato de longo prazo |
| **Log Analytics workspace** | Consultas e dashboards (Azure Monitor) |
| **Event Hubs** | Streaming para outros sistemas (e.g. a SIEM) |

### Nível de servidor vs nível de banco de dados

| Tipo | Scope | Quando |
|------|-------|------|
| **nível servidor** | Todos os bancos de dados no servidor, incluindo novos | padrão **Recomendado** |
| **nível de banco de dados** | Um banco de dados | Somente se essa banco de dados precisar de um destino diferente ou eventos diferentes |

- Se a auditoria do servidor estiver ativada, ela **sempre se aplica**. Uma auditoria de banco de dados executa **lado a lado** com ela. Ela substitui o **E**.
- Evite ligar os dois, a menos que você tenha um motivo (armazenamento diferente, regras diferentes).

**Exemplo:** Um banco ativa a auditoria de nível de servidor para o Log Analytics para todos os 50 bancos de dados. Seu banco de dados de fraude também deve manter registros em uma conta de armazenamento separada para reguladores, para que um banco de dados também receba uma auditoria de nível de banco de dados.

### Eventos padrão
Os grupos de ação padrão são:
- `BATCH_COMPLETED_GROUP` (cada consulta e procedimento executado)
- `SUCCESSFUL_DATABASE_AUTHENTICATION_GROUP`
- `FAILED_DATABASE_AUTHENTICATION_GROUP`

Você pode adicionar mais grupos de ação.

### Bom saber
- **Acesso a dados sensíveis:** se as colunas forem classificadas, o campo de registro de auditoria **`data_sensitivity_information`** mostra quais colunas rotuladas uma consulta tocada.
- Ligado no portal: **SQL server → Segurança → Auditoria**.
- Read-only replicas são auditados automaticamente.
- Sob carga muito pesada, alguns eventos podem ser descartados.
- O padrão de retenção de armazenamento é **0 = keep forever**.
- Escreva para armazenamento com um **managed identity** (recomendado) ou chaves de armazenamento. Os alvos Log Analytics e Event Hubs usam uma configuração de diagnóstico `SQLSecurityAuditEvents`.
- Desde julho de 2025, todos os registros de auditoria no nível do servidor (incluindo read-only replicas) vão para uma pasta **`master`**.
- Falha **Entro** logins são **E** no log de auditoria SQL. Use o **Entrar logs**.

## Dynamic data masking (DDM)
**DDM** esconde parte do valor **em resultados de consulta** de uma coluna. Os dados reais na tabela são **Não alterado** e **não encriptado**.
- Usuários sem a permissão **UNMASK** ver valores mascarados.
- Usuários com UNMASK, ou com **CONTROL** (db_owner, sysadmin), sempre veem dados reais.
- Não são necessárias alterações no app.

**Exemplo:** Um agente do call center executa `SELECT Phone FROM Customers` e vê `XXX-XXX-1234`. O gerente, que tem UNMASK, vê o número completo.

### Masking funções

| Função | Resultado | Exemplo |
|----------|--------|---------|
| `default()` | Máscara completa: `XXXX`, `0`, `1900-01-01` | Endereço |
| `email()` | `aXXX@XXXX.com` | E-mail |
| `partial(prefix,"pad",suffix)` | Mostra começo e fim, mascara o meio | `partial(0,"XXXX-XXXX-",4)` para telefone |
| `random(low,high)` | Um número aleatório em um intervalo | `random(1,12)` |
| `datetime("Y")` | Máscaras parte de uma data (SQL Server 2022+) | Data de nascimento |

O portal também oferece uma máscara **cartão de crédito** (`XXXX-XXXX-XXXX-1234`).

```sql
ALTER TABLE dbo.Customer ALTER COLUMN Email
ADD MASKED WITH (FUNCTION = 'email()');
GRANT UNMASK TO Manager1;
GRANT UNMASK ON SCHEMA::Data TO ServiceManager;            -- granular (SQL Server 2022+, Azure SQL)
GRANT UNMASK ON Data.Membership(FirstName) TO Attendant;   -- one column
```

- Portal: banco de dados → **Segurança → Dynamic Data Masking**.
- Adicionar uma máscara precisa de **ALTER ANY MASK** mais ALTER na tabela.
- **A cópia mantém a máscara:** `SELECT * INTO Copy FROM Customers` executado por um usuário sem UNMASK armazena valores **mascarada**.
- **Fraqueza:** usuários inteligentes podem adivinhar valores com consultas (**inferência**, e.g. `WHERE Salary > 50000`). Combine DDM com auditoria, criptografia e RLS.

## Row-level security (RLS)
**RLS** controla **quais linhas** um usuário pode ver ou mudar. Funciona como uma cláusula automática `WHERE` adicionada a cada consulta pelo próprio banco de dados.

**Exemplo:** Um aplicativo de vendas SaaS armazena todos os inquilinos em uma tabela `Sales`. Quando o Tenant1 o consulta, eles vêem apenas linhas onde o `TenantName = 'Tenant1'`. Eles não podem ver as ordens do Tenant2, no entanto, escrevem a consulta.

### Como funciona
1. Crie um **função em linha de tabela-valor** (o **predicado**) com a regra.
2. Crie um **política de segurança** que atribua a função à tabela.

```sql
CREATE FUNCTION sec.fn_TenantFilter(@TenantName NVARCHAR(10))
RETURNS TABLE WITH SCHEMABINDING
AS RETURN SELECT 1 AS ok
WHERE @TenantName = USER_NAME() OR USER_NAME() = 'TenantAdmin';
GO
CREATE SECURITY POLICY sec.SalesPolicy
ADD FILTER PREDICATE sec.fn_TenantFilter(TenantName) ON dbo.Sales
WITH (STATE = ON);
```

Teste com `EXECUTE AS USER = 'Tenant1'; SELECT * FROM dbo.Sales; REVERT;`.

### Dois tipos de predicado

| Predicar | O que faz | Afectes |
|-----------|--------------|---------|
| **Filtro** | Esconde silenciosamente as linhas que o usuário pode não ver | SELECT, UPDATE, DELETE |
| **Bloco** | Escreve que quebra a regra | INSERT, UPDATE, DELETE |

### Dicas
- RLS faz qualquer coisa **não encriptar**
- Assista ao **ataques de canal lateral** (e.g. dividir por zero erros que revelam uma linha oculta existe).
- Coloque funções e políticas em um **esquema separado**. Mantenha as funções de predicados simples e rápidas.

## Ledger
**Ledger** faz dados **invio-evidente**. Prova aos auditores ou parceiros que as linhas não foram secretamente alteradas. É como blockchain, mas dentro de um banco de dados, sem rede de computadores.
- Disponível em **Azure SQL Database, Managed Instance e SQL Server 2022+**.
- Cada transação é hashed com **SHA-256**. Os hashes são **acorrentado** juntos. Se alguém muda dados fora do normal SQL (e.g. edita os arquivos), as quebras de cadeia e **Falha na verificação**.
- Ledger **detecta** adulteração. Ele faz **não evitar** ele.

### Dois tipos ledger table

| Tipo | Permite | Uso para |
|------|--------|---------|
| **Tabela Updatable ledger** | Inserir, atualizar, apagar. Versões antigas mantidas em um **tabela história** | Sistemas de registro, e.g. saldos bancários |
| **Tabela Append-only ledger** | **Inserir apenas**. Atualizações e exclusões são bloqueadas | Logs, diários de contabilidade, eventos de segurança |

```sql
CREATE TABLE dbo.Balances (ID INT PRIMARY KEY, Amount MONEY)
WITH (SYSTEM_VERSIONING = ON, LEDGER = ON);           -- updatable
CREATE TABLE dbo.AuditEvents (ID INT, Msg NVARCHAR(200))
WITH (LEDGER = ON (APPEND_ONLY = ON));                -- append-only
```

### Banco de dados e resumos Ledger
- A **banco de dados** makes **Cada** tabela a ledger table. Você escolhe este **somente quando se cria** o banco de dados (portal → **Segurança** tab → Ledger → **Ativar para todas as tabelas futuras**). Você **não pode** Ligue ou desligue mais tarde.
- Você ainda pode fazer ledger tables único em um banco de dados normal com T-SQL.
- O A **digest** é o hash mais recente da cadeia. Armazenar digeri fora do banco de dados: **Azure Blob Storage com uma política de imutabilidade** ou **Azure Confidencial Ledger** (automático), ou salvá-los manualmente.
- Verifique com `sys.sp_verify_database_ledger_from_digest_storage` (ou `sys.sp_verify_database_ledger` com um resumo salvo).

**Exemplo:** Uma empresa fintech mantém aprovações de empréstimos em uma tabela updatable ledger e envia digests para Blob Storage imutável. Um auditor executa a verificação e vê que a cadeia está intacta, então nenhuma aprovação foi editada silenciosamente.

## Microsoft Defender for SQL
**Defender para SQL** é um serviço de segurança. Ele cobre Azure SQL Database, Managed Instance, SQL Server on Azure VMs, SQL Server habilitado por **Arco Azure** (on-premises ou outras nuvens), e Synapse piscinas SQL dedicadas. Tem duas partes.

### 1. Vulnerability Assessment (VA)
Scans para **Configurações fracas**: muitas permissões, autenticação fraca, dados sensíveis desprotegidos, configuração ruim. Ele dá recomendações fixas.
- **Configuração expressa** (recomendado): resultados armazenados pela Microsoft, **sem conta de armazenamento** necessário.
- **Configuração clássica**: os resultados vão para **sua conta de armazenamento** (clássico VA / ATP APIs se aposentar 16 Aug 2027).

**Exemplo:** VA sinaliza que o usuário do aplicativo de relatório está no `db_owner`, mas só precisa ler. Você move para `db_datareader`.

### 2. Proteção Avançada contra Ameaças (ATP)
atividade relógios e levanta **Alertas em tempo real**:
- **Vulnerabilidade à injeção SQL** (uma consulta mal escrita que poderia ser explorada).
- **Injeção potencial SQL** (uma tentativa de ataque real).
- Acesso a partir de um **Localização incomum**, um **Centro de dados Azure desconhecido** ou um **principal desconhecido**.
- Acesso a partir de um **aplicação potencialmente prejudicial** (ferramentas de ataque conhecidas).
- Credenciais **força bruta** SQL (muitas senhas falhadas).

**Exemplo:** 100 logins falhados chegam de um país desconhecido em dois minutos. Defender levanta um alerta de força bruta.

### Ligando-o
- Por servidor ou instância: **Microsoft Defender for Cloud** seção → set Defender para SQL **Ligado**.
- Ou para todo o **assinatura**: Defender for Cloud → **Configurações do ambiente → Planos de defesa → Bancos de dados**. Planos: **Defender para Azure SQL Databases** (SQL DB, MI, Synapse) e **Defender para Servidores SQL em Máquinas** (VMs, Arc). Novos recursos são cobertos automaticamente.
- Ligue o **Auditoria** também, para que você possa investigar alertas.
- Quem pode configurá-lo: **SQL Security Manager** ou um administrador de servidor/banco de dados.

## Compare em um relance

| Necessidade | Recurso |
|------|---------|
| Localizar e rotular colunas sensíveis em um banco de dados | Descoberta e classificação de dados |
| Encontre dados sensíveis e linhagens em toda a empresa | Microsoft Purview |
| Registrar quem logou e o que eles correram | Auditoria |
| Mostrar apenas parte de um valor para alguns usuários | Dynamic data masking |
| Cada usuário vê apenas suas próprias linhas | Row-level security |
| Os dados não foram adulterados com | Ledger |
| Localizar configurações fracas / obter alertas de ataque | Defensor para SQL (VA / ATP) |
| Ocultar valores até mesmo de DBAs | Always Encrypted (lição 4) |

## Para lembrar na prova
- Os rótulos de classificação estão em **`sys.sensitivity_classifications`**; T-SQL `ADD SENSITIVITY CLASSIFICATION`. Auto-recomendações dependem de **nomes de colunas**.
- Auditoria a **Armazenamento** (arquiva), **Log Analytics** (query), **Event Hubs** (stream). Prefere **servidor-nível**; auditoria de banco de dados roda **Ao lado**, não em vez disso.
- `data_sensitivity_information` em registros de auditoria mostra o acesso a colunas classificadas.
- DDM muda **apenas saída**; os dados não são criptografados. **UNMASK** revela; db_owner sempre vê dados reais. As máscaras sobrevivem `SELECT INTO` para usuários não UNMASK.
- RLS = inline TVF + **política de segurança**. **Filtro** esconde linhas; **bloco** Paradas de escrita.
- Ledger **adulter- evidência**, SHA-256 corrente. Updatable vs **apenas append**. Ledger **banco de dados** apenas na criação. Digest para **Blob imutável** : **Azure Confidencial Ledger**.
- Defensor para SQL = **VA** (errosconfigurações) + **ATP** (SQL injeção, força bruta, acesso incomum).
- Varredura Purview com um **tempo de execução auto-hospedado** → no managed identity; use um auth service principal ou SQL.

## Teste rápido
1. A equipe do call center deve ver apenas os últimos quatro dígitos dos números de cartão; os gerentes devem ver o número completo. O que você usa?
   <details><summary>Resposta</summary>Dynamic data masking com uma máscara parcial() (ou cartão de crédito) e GRANT UNMASK para gerentes.</details>
2. Os reguladores precisam de provas de que um diário de contabilidade nunca foi editado ou excluído. Que tipo de tabela?
   <details><summary>Resposta</summary>Uma tabela append-only ledger. Permite apenas inserções e sua cadeia de hash mostra qualquer adulteração.</details>
3. Você quer as mesmas configurações de auditoria em cada banco de dados atual e futuro em um servidor. Onde você configura a auditoria?
   <details><summary>Resposta</summary>No nível do servidor. Aplica-se a todas Os bancos de dados no servidor.</details>
