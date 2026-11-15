# Resolver uma migração

> **Habilidades cobradas:** Resolver uma migração
> **Tempo de leitura:** ~9 min

## Conceito central
A maioria das falhas de migração vem de quatro coisas: **rede** (não pode chegar à fonte), **permissões** (armazenamento ou direitos login), **Formato backup** (tipo errado de arquivo backup) e **características não suportadas** no alvo. Saiba onde procurar e conheça as mensagens de erro comuns.

Analogia: um pacote que nunca chegou. O endereço estava errado (rede)? Ninguém assinou por ele (permissões)? A caixa estava na forma errada (formato)? Ou o item foi banido no destino (característica)?

## Para onde olhar
| Lugar | O que mostra |
|-------|---------------|
| **DMS → Monitorar migrações** | Por banco de dados e por status da tabela e texto de erro |
| **Diário de atividades** (recurso → Diário de atividades → Resumo/ JSON) | Falhas de nível de recurso, mensagens de erro exatas |
| **Arc → Monitor e recorte** | Arquivos LRS detectados, enfilados, ignorados ou intransponíveis |
| Status LRS | `Get-AzSqlInstanceDatabaseLogReplay` / `az sql midb log-replay show` |
| Operação MI falhada | `Get-AzSqlInstanceOperation` / `az sql mi op show` |
| estado da ligação MI | `Get-AzSqlInstanceLink` / `az sql mi link show` / DMVs |
| log de erro SQL Server, perfilr XEvent | Erros do motor (link, certificados, semeadura) |

## Problemas de conectividade
| Sintoma | Causa | Correção |
|---------|-------|-----|
| Erro 26 – Erro ao localizar o servidor/instância | **Nomeado instância** ou porto dinâmico | Abrir **UDP 1434** (navegador SQL) **e** a instância TCP porta |
| DMS não pode chegar fonte, preso | **SHIR** não pode ver a fonte ou backups | Instalar SHIR em uma máquina com linha de visão para a fonte; dimensioná-lo bem |
| Não é possível ler backups em uma ação SMB | Direitos de partilha | Dê à conta de migração **e** a conta de serviço SQL controle total. Teste com `RESTORE HEADERONLY` |
| Tempo limite sobre ExpressRoute (clássico DMS) | Falta service endpoints | Habilitar os terminais de banco de dados de serviço, armazenamento e destino na sub-rede DMS |
| link MI não vai criar | Sem caminho privado, portos bloqueados | VPN / ExpressRoute, porta **5022** ambos os sentidos, portas MI HADR, regras NSG |

**Exemplo:** A DBA migrações `SQL01\SALES` com DMS. Falha com o erro 26. O firewall só permite TCP 1433. Abertura UDP 1434 e o porto da instância conserta-o.

## Regras Backup que quebram as migrações
Para DMS (para MI ou VM) e LRS:
- **Um backup por arquivo.** Backups **anexado** em um arquivo não são suportados.
- Use **`WITH CHECKSUM`**. Sem ele, você pode migrar um banco de dados corrompido e as restaurações são mais lentas.
- **Um arquivo de registro** por banco de dados. Bancos de dados com vários arquivos de log não são suportados.
- Somente arquivos **.bak,.diff,.log**. **Não BACPAC/DACPAC** com LRS.

## Lista de verificação LRS
| Problema | Causa / correção |
|---------|-------------|
| LRS não vai começar | **O SAS deve ser lido + lista somente.** Escrever faz com que falhe. Faça para todo o recipiente |
| SAS "inválido" | Janela de tempo errada (zonas horárias). Regenerar com uma janela mais larga |
| Conflito de nomes | Um banco de dados com o **mesmo nome** já existe no MI |
| Arquivos não encontrados | Cada banco de dados precisa de seu **pasta plana**. O caminho não pode conter a palavra **backup/backups** |
| Não é possível adicionar novo log backups | Você usou **autocompletar**. Use o modo **contínuo** |
| Job desapareceu | Atingir o limite **30 dias** |
| Reiniciado do zero | MI failover durante a primeira restauração completa ou durante o corte |
| "Incapaz de listar diretórios" (portal) | **O seu usuário** não tem **Armazenamento Blob Data Reader** |
| **Erro 2009** (managed identity não configurado corretamente) | O **MI managed identity** Falta **Armazenamento Blob Data Reader** |
| Nenhum resultado foi encontrado no diretório | Não full backup Ainda não |

Mais dois fatos MI:
- MI retém atualizações não críticas para até **36 horas** durante uma migração. Depois disso, uma migração interrompida **reinicia a partir de uma restauração completa**.
- **O corte Business Critical é lento** Porque ele sementes 3 réplicas. Corrig: migrar para **General Purpose** em seguida, atualizar on-line, ou usar **link MI**.

## Problemas de ligação MI
```sql
SELECT rs.synchronization_health_desc   -- HEALTHY / NOT_HEALTHY
FROM sys.availability_groups ag
JOIN sys.dm_hadr_availability_replica_states rs ON ag.group_id = rs.group_id
WHERE rs.is_local = 0 AND rs.role = 2 AND ag.is_distributed = 1 AND ag.name = '<link name>';
```
- Progresso da semeadura: `sys.dm_hadr_physical_seeding_stats`, `sys.dm_hadr_automatic_seeding`.
- **A primeira semeadura não é resiliente.** Uma queda de rede, reiniciar ou failover reinicia. Link criação auto-cancels depois **6 dias**.

| Erro | Significado | Correção |
|-------|---------|-----|
| 41962 | Link não iniciado em 5 minutos | Verificar rede, tentar novamente |
| 41973 / 41974 | Certificado de endpoint não importado (SQL Server → MI / MI → SQL Server) | Re-importar o certificado |
| 41976 / 41986 | AG não respondeu / conexão falhou | Nomes de verificação, configuração, rede |
| **1408 / 1412** | **Log truncado antes da semeadura terminada** (um log backup run) | **Drop e recriar** o link. Pausa log backups ou utilizar **bandeira de rastreamento 12381** |

Outro link gotchas:
- Planned failover **vezes fora** → o secundário está atrasado. Espere por isso.
- Ambos os lados PRIMARY após forced failover (**cérebro dividido**) → backup do primário antigo, configurá-lo para SECONDARY, falhar corretamente.
- **Certificado expirado** → gire-o.
- Não solte o link antes MI 's **primeiro full backup** após failover.
- Os firewalls devem permitir **ICMP** e o protocolo SQL Server UCS.
- Um link feito à mão em blocos SSMS criando um a partir do portal Arc.

## Problemas com Azure SQL Database (DMS)
| Você vê | Explicação |
|---------|-----|
| Não pode clicar em Next | Não há tabelas no alvo. Tick **Migrar o esquema em falta** |
| Faltam colunas computadas | Não migrado |
| NULLs tornou-se valores padrão | Colunas com padrão obter o padrão |
| Algumas tabelas não migradas | **Tabelas vazias são ignoradas** |
| Tempos da tabela fora | colunas **Grande BLOB** |
| Nome da tabela rejeitado | Personagens **Double-byte**. Renomear, migrar, renomear de volta |
| Nome da banco de dados rejeitado | Palavras reservadas ou ponto e vírgulas |
| CDC não vai ativar no alvo | Desativar o CDC na fonte antes da migração do esquema |
| Devagar com milhares de tabelas | ADF sobrecarga por tabela |

Permissões: fonte **db_datareader + VIEW ANY DEFINITION**; alvo **db_owner**. A migração do esquema também precisa de db_owner na fonte e SHIR 5.37 +.

Erros antigos DMS (clássicos) que você ainda pode ver em perguntas: **4 bancos de dados** máximo por atividade; uma atividade presa no **enfil** significa que a instância está cheia; você não pode parar o DMS enquanto as atividades são executadas.

## Após a migração
| Problema | Correção |
|---------|-----|
| **Usuários órfãos** (usuário sem correspondência login) | `ALTER USER [app] WITH LOGIN = [app];` |
| Logins, Agente jobs, servidores vinculados ausentes | As ferramentas de nível de banco de dados não as movem. |
| FILESTREAM / FileTable falha em MI | Não suportado. Use um **VM** |
| In-Memory OLTP falha em MI | Necessidades **Business Critical** |
| Corretor de serviços inutilizável em MI | Foi desligado na fonte. Habilitá-lo e re-migrar |
| Não é possível ativar o ADR no MI | ADR estava fora da fonte (SQL 2019+). Habilitar e re-migrar |
| Consultas mais lentas | Atualize estatísticas, verifique **Nível de compatibilidade**, compare planos com **Query Store** |

**Exemplo:** Depois de restaurar para um VM, o aplicativo não pode fazer login. O usuário do banco de dados `app` existe, mas seu SID não corresponde ao novo servidor login. `ALTER USER [app] WITH LOGIN = [app]` corrige isso.

## Para lembrar na prova
- **Erro 2009** → dá o leitor de dados **MI managed identity** Storage Blob.
- LRS não vai começar, SAS tem escrever → SAS deve ser **Ler + Listar apenas**.
- **Erro 1412** no MI link → log truncado durante a semeadura → gota e recriar, pausar log backups ou **TF 12381**.
- A instância nomeada não acessível → **UDP 1434** + porta de instância.
- DMS para SQL Database é **Apenas offline**. As tabelas vazias são ignoradas. As colunas computadas não são migradas.
- Slow Business Critical cutover → migrar para GP, em seguida, atualizar, ou usar o link MI.
- Usuários órfãos → `ALTER USER ... WITH LOGIN`.

## Teste rápido
1. O LRS falha com o erro 2009. O que você corrige?
   <details><summary>Resposta</summary>Conceda ao managed identity do managed instance a função Storage Blob Data Reader na conta de armazenamento.</details>
2. A criação do link MI falha com o erro 1412. O que aconteceu?
   <details><summary>Resposta</summary>Um log backup truncado o log antes da semeadura terminar. Recrie o link e pause log backups (ou use TF 12381).</details>
3. Depois de DMS a SQL Database, faltam três tabelas. Eles não tinham filas. Isso é um fracasso?
   <details><summary>Resposta</summary>No. DMS pula tabelas vazias pelo projeto.</details>
