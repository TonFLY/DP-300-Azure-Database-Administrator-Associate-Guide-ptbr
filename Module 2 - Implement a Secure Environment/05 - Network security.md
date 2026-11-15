# Segurança de rede

> **Habilidades cobradas:** Configurar regras de firewall de servidor e banco de dados, Configurar links privados e service endpoints
> **Tempo de leitura:** ~8 min

## Conceito central
Antes que qualquer um possa fazer login, seu tráfego de rede deve chegar ao banco de dados. A segurança de rede decide que o **quais redes e endereços IP** pode até bater na porta. Pense em um imóvel fechado. O firewall é o guarda de portão com uma lista de carros permitidos. O private link é um túnel privado da sua garagem direto para a propriedade.

## Regras de firewall no Azure SQL Database
O firewall Azure SQL Database permite ou bloqueia conexões por **Endereço IP** ou por **regra de rede virtual**. Cada conexão passa por ela primeiro. Por padrão, **Está tudo bloqueado**.

### Dois níveis

| Nível | Scope | Como definir |
|-------|-------|---------------|
| **nível servidor** | Todos os bancos de dados no servidor lógico | Portal, PowerShell, CLI, REST ou T-SQL `sp_set_firewall_rule` em **mestre** |
| **nível de banco de dados** | Apenas um banco de dados | **Somente T-SQL**: `sp_set_database_firewall_rule` em **Esse banco de dados** |

```sql
-- In master: allow the office IP to reach every database
EXEC sp_set_firewall_rule N'Office', '52.120.14.22', '52.120.14.22';
-- In SalesDB: allow a partner to reach only this database
EXEC sp_set_database_firewall_rule N'Partner', '203.0.113.5', '203.0.113.5';
```

### Ordem dos controlos
1. O Azure verifica as regras **nível de banco de dados** para esse banco de dados.
2. Se não houver correspondência, verifica as regras do **servidor-nível**.
3. Se ainda não houver correspondência, a conexão é **bloqueado**.

**Exemplo:** Uma escola tem 10 bancos de dados. Todos os funcionários usam o escritório da escola IP, então o DBA adiciona um **servidor-nível** Um quadro de exame externo precisa apenas da `ResultsDB`. O DBA adiciona a **nível de banco de dados** regra para sua IP E- `ResultsDB` Apenas.

### Limites e dicas
- Até regras IP de nível de servidor **256** e **256** de nível de banco de dados.
- Estas regras de firewall IP são para **Azure SQL Database**. **Managed Instance** usa **NSGs** (network security groups: permitir / negar regras em uma sub-rede).
- Em **SSMS**, se o seu IP está bloqueado, ele oferece para adicionar uma regra de firewall para você. (Azure Data Studio foi aposentado em 28 de fevereiro de 2026.)

### "Permitir que os serviços e recursos do Azure acessem este servidor"
- Isso cria uma regra especial `0.0.0.0` chamada **AllowAllWindowsAzureIps**.
- Ele permite que os serviços do Azure **Todos** entrem, **incluindo assinaturas de outros clientes**. Não são apenas seus recursos do Azure.
- Prefira **Regras VNet** ou **private endpoints** em vez disso.

## Rede virtual service endpoints
A **service endpoint** permite o tráfego a partir de um escolhido **subnet** alcance Azure SQL sobre a rede de backbone da Microsoft. Você então adiciona um **regra de rede virtual** E a SQL para permitir que essa sub-rede.

- O acesso é controlado **subnet**, não por IP.
- O banco de dados **Ainda usa seu endpoint público** (endereço público). O tráfego é apenas melhor encaminhado e marcado com a sub-rede.
- Aplica-se dentro de um **região Azure**.
- Fácil de configurar.

**Exemplo:** Um aplicativo web é executado em VMs na sub-rede 10.0.0.0/24. O DBA liga o SQL service endpoint para essa sub-rede e adiciona uma regra VNet. Somente essa sub-rede pode se conectar; outro tráfego do Azure está bloqueado.

## Private Link (private endpoint)
**Private Link** dá ao seu banco de dados um **Endereço IP privado dentro do seu VNet**. Isso é chamado de **private endpoint**.
- O tráfego permanece na rede privada da Microsoft. Nunca passa pela internet pública.
- Funciona **entre regiões**, e de on-premises sobre **VPN** ou **ExpressRoute**.
- É a escolha **mais seguro**, e melhor contra **exfiltração de dados** (dados sendo copiados para lugares que não deve ir).
- Set **Public network access = Disable** para que apenas conexões private endpoint sejam permitidas. Depois disso, você não pode adicionar ou editar regras de firewall (erro **42101**).

**Exemplo:** O DBA cria um private endpoint para que o banco de dados obtenha o 10.1.2.5 dentro do hospital VNet e, em seguida, desabilite o acesso à rede pública.

## Outras opções de rede
- **Gateway VPN:** um túnel criptografado pela internet de on-premises para Azure. Mais barato. Bom para dev / teste ou cargas de trabalho menores.
- **ExpressRoute:** a **linha privada dedicada** do seu datacenter para o Azure. Ele não usa a internet. Rápido, confiável, mais caro. Usado para conformidade estrita.
- **perímetro de segurança da rede** (preview para Azure SQL Database): grupos SQL Database com outros recursos PaaS (Armazenamento, Cofre Chave) e controla o tráfego dentro e fora. Começa no modo **transição** (antigo modo "aprender"), então você muda para **Forçado**.

### Política de ligação
No servidor: **Conectividade de rede →**.
- **redirecionamento** (recomendado): o cliente se conecta diretamente ao nó do banco de dados. Precisa de portas de saída **11000–11999**.
- **Proxy:** todo o tráfego passa pelo gateway no porto **1433**.
- **Padrão:** Redirecionamento para clientes dentro do Azure, Proxy para clientes fora do Azure.

### SQL Server on Azure VMs
A VM É protegido como qualquer servidor: **NSGs** Na sub-rede, o **Firewall do Windows** dentro da VMe, VPN : ExpressRoute para ligações híbridas.

## Injeção SQL (básico)
**injeção SQL** é um ataque onde um usuário digita código SQL em uma caixa de entrada e o aplicativo executa.

**Exemplo:** Um aplicativo constrói `SELECT * FROM Orders WHERE OrderId = ` + entrada do usuário. Um atacante digita `25; DELETE FROM Orders;`. O banco de dados executa ambas as declarações e exclui todas as ordens.

Como evitá-lo:
- **Consultas parametrizadas** (melhor): `WHERE OrderId = @OrderId`. A entrada é tratada como dados, nunca como código.
- Bibliotecas **Procedimentos armazenados** e **ORM** (e.g. Estrutura de Entidade).
- **Validar a entrada** (é realmente um número?).
- **Menor privilégio** para a conta do aplicativo, então uma consulta ruim pode fazer pouco dano.
- **Firewall de Aplicação Web (WAF)** e **Microsoft Defender for SQL**, que alerta sobre tentativas de injeção SQL (lição 6).

Palavras de bloqueio como "DELETE" é **não o suficiente**. Os atacantes podem ocultar comandos em dados codificados ou binários.

## Compare em um relance

| Opção | Como o acesso é controlado | Endpoint público usado? | Escolha-o quando |
|--------|-------------------------|----------------------|--------------|
| Regra IP de nível de servidor | Faixa IP, todas Os bancos de dados | Sim | Escritório da empresa inteira IP |
| Regra IP de nível de banco de dados | Faixa IP, uma banco de dados | Sim | Parceiro precisa de um banco de dados |
| Regra Service endpoint + VNet | Sub-rede | **Sim** | Acesso simples apenas à sub-rede, ponto final público é OK |
| Private Link | IP privado em seu VNet | **Não** (pode desativar o acesso público) | Maior segurança, sem exposição pública, cross-region, on-premises via VPN / ExpressRoute |
| Gateway VPN | Túnel encriptado através da Internet | – | Ligação híbrida de baixo custo |
| ExpressRoute | circuito dedicado privado | – | Ligação híbrida de alta largura de banda, conformidade |

## Para lembrar na prova
- Ordem de verificação: **regra de banco de dados → regra do servidor → bloco**.
- Regras de nível de banco de dados: **Somente T-SQL**, no banco de dados do usuário. Nível do servidor: portal / CLI / PowerShell ou T-SQL em **mestre**.
- **256** regras por nível. Managed Instance usos **NSGs**Não estes IP Regras.
- "Allow Azure services" = `0.0.0.0` regra, abre-se a **Todos** Clientes Azure. Evite isso.
- **Service endpoint** é configurado na subnet, usa um endpoint público e fica restrito a uma região.
- **Private Link** = private IP, sem internet pública, melhor proteção; desativar o acesso público para forçá-lo.
- Redirecionar precisa de portas **11000–11999**; Proxy usa **1433**.
- Injeção SQL → **consultas parametrizadas**; Defender para SQL detecta tentativas.

## Teste rápido
1. Um auditor externo precisa ter acesso a apenas uma das suas 20 bancos de dados. Que regra você cria e onde?
   <details><summary>Resposta</summary>Uma regra de firewall de nível de banco de dados com sp_set_database_firewall_rule, é executada dentro desse banco de dados.</details>
2. A segurança diz que o banco de dados não deve ter nenhum ponto final público. O que você usa?
   <details><summary>Resposta</summary>A private endpoint (Private Link), e definir acesso à rede pública para desabilitar.</details>
3. Os clientes dentro do Azure não conseguem se conectar usando a política de Redirecionamento. Quais portas você deve verificar?
   <details><summary>Resposta</summary>Portas de saída 11000–11999 (mais 1433 para o gateway). Redirecionar conecta diretamente ao nó nessas portas.</details>
