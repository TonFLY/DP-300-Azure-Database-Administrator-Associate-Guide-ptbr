# Módulo 2 — Implementar um ambiente seguro

**Peso do exame:** 20–25% · **Tempo de leitura:** ~85 min · **Arquivos:** 8

## O que você vai aprender
Você vai aprender como pessoas e aplicativos comprovam sua identidade e como conceder somente as permissões necessárias. Também vai criptografar dados em disco, em colunas específicas e em trânsito, e definir quais redes podem acessar o banco. Por fim, você vai estudar os controles de conformidade, como rótulos, auditoria, masking, filtros de linha, ledger, alertas de ameaças e change tracking.

Imagine um banco. A porta da frente verifica o seu ID (autenticação). Seu cartão-chave abre apenas alguns quartos (permissões). O cofre está bloqueado (TDE), e algumas caixas seguras podem ser abertas apenas por seus proprietários, não por funcionários (Always Encrypted). O dinheiro move-se em vans blindadas (TLS) ao longo de estradas privadas (Private Link). As câmeras gravam tudo (auditando), os caixas veem apenas parte de um número de cartão (mascaramento), cada filial vê apenas seus próprios clientes (row-level security), e um diário de bordo selado mostra se uma página foi arrancada (ledger).

## Lições neste módulo
| # | Lição | O que ela cobre | Tempo |
|---|--------|----------------|------|
| 1 | [Autenticação](<01 - Authentication.md>) | SQL vs Entra ID login, Admin de entrada, Auth somente para entrada, managed identities, Azure RBAC | ~10 min |
| 2 | [Usuários, roles e permissões](<02 - Users, roles and permissions.md>) | Logins, usuários, papéis, GRANT / DENY / REVOKE, cadeias de propriedade, menos privilégio | ~13 min |
| 3 | [Solucionar problemas de login e permissão](<03 - Troubleshoot login and permission problems.md>) | Falhas transitórias, lógica de repetição, fixação "login falhou" | ~6 min |
| 4 | [Criptografia](<04 - Encryption.md>) | TDE, Chaves chave Vault, encriptação de células, Always Encrypted, VBS e SGX enclaves, TLS | ~20 min |
| 5 | [Segurança de rede](<05 - Network security.md>) | Regras de firewall, service endpoints, Private Link, política de conexão, injeção SQL | ~8 min |
| 6 | [Controles de conformidade](<06 - Compliance controls.md>) | Classificação, Purview, auditoria, mascaramento, row-level security, livro-razão, Defender for SQL | ~17 min |
| 7 | [Change tracking e change data capture](<07 - Change tracking and change data capture.md>) | CT vs CDC, configuração, consulta, limites da plataforma | ~10 min |

## Palavras que você vai encontrar
| Palavra | Significado simples |
|------|----------------|
| Autenticação | Provando quem você é |
| Autorização | O que você tem permissão para fazer |
| Microsoft Entra ID | Serviço de identidade em nuvem da Microsoft (nome antigo: Azure AD) |
| Managed identity | Um Azure-managed identity para um recurso, sem senha para armazenar |
| Login / Usuário | Acesso ao servidor / acesso a um banco de dados |
| Menor privilégio | Dê apenas os direitos necessários para o job |
| TDE | Criptografa toda a banco de dados e backups no disco |
| Always Encrypted | Criptografa colunas escolhidas no aplicativo, para que o servidor nunca veja valores simples |
| Secure enclave | Uma área bloqueada de memória do servidor onde os dados criptografados podem ser trabalhados com segurança |
| TLS | Criptografia de dados que se deslocam pela rede |
| Private Link | Dá ao banco de dados um IP privado dentro da sua rede virtual |
| Service endpoint | Permite que uma sub-rede escolhida alcance o banco de dados sobre a espinha dorsal do Azure |
| Dynamic data masking | Oculta parte de um valor nos resultados da consulta |
| Row-level security | Cada usuário vê apenas as linhas que pode ver |
| Ledger | Torna a manipulação de dados detectável |

## exame de habilidades cobertas
- [ ] Configurar a autenticação Microsoft Entra ID para Azure SQL Database, Azure SQL Managed Instance e SQL Server → 1
- [ ] Configurar autenticação para SQL em Azure VMs e Azure SQL Managed Instance → 1
- [ ] Configurar os princípios de segurança → 2
- [ ] Criar usuários a partir de identidades Microsoft Entra → 2
- [ ] Configurar permissões de banco de dados e nível de objeto usando ferramentas gráficas → 2
- [ ] Aplicar o princípio do privilégio mínimo para todos os bens protegidos → 2
- [ ] Resolver problemas de autenticação e autorização → 3
- [ ] Gerenciar autenticação e autorização usando T-SQL → 2, 3
- [ ] Implementar transparent data encryption (TDE) → 4
- [ ] Implemente a criptografia de nível de objeto → 4
- [ ] Configurar regras de firewall de nível servidor e banco de dados → 5
- [ ] Implementar Always Encrypted → 4
- [ ] Implementar Always Encrypted with secure enclaves → 4
- [ ] Configurar links privados e service endpoints → 5
- [ ] Aplicar uma estratégia de classificação de dados → 6
- [ ] Configurar auditorias de servidor e banco de dados → 6
- [ ] Implementar o rastreamento de dados de mudança → 7
- [ ] Implementar dynamic data masking → 6
- [ ] Implemente o livro-razão em Azure SQL → 6
- [ ] Implementar row-level security → 6
