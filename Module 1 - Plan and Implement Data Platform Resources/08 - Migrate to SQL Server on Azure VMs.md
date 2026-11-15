# Migrar para SQL Server on Azure VMs

> **Habilidades cobradas:** Implemente uma estratégia de migração offline, implemente uma estratégia online de migração, implemente uma migração para o Azure
> **Tempo de leitura:** ~6 min

## Conceito central
SQL Server VM é normal SQL Server. Assim, cada truque on-premises funciona: backup e restaurar, log shipping, availability groups. Escolha pelo tempo de inatividade: **horas OK** → backup / restauração; **minutos** → DMS ou log shipping; **Quase quase nenhum** → distribuído AG.

Analogia: movendo-se para uma casa idêntica em outra cidade. Tudo se encaixa. Você só escolhe o quão rápido você se move.

## Tempo de inatividade e método
| Tempo de inatividade | Método |
|----------|--------|
| Quase nenhum | **AG distribuído** / Always On AG |
| Minutos | **DMS online**, log shipping, transactional replication, Azure Migrate |
| Horas ou dias | Backup /restaurar, desacoplar/anexar, BACPAC, bcp |

## Nativo backup e restaurar
O método mais simples.
1. Faça backup do banco de dados, idealmente direto para **Azure Blob Storage** com `BACKUP TO URL`.
2. Restaure-o no VM do mesmo URL.

**Blob vs blob de página:**
| Blob | Credencial | Utilização |
|------|-----------|-----|
| **Bloco Blob** | **token SAS** | Recomendado (SQL 2016+). Maior, mais rápido, mais seguro. |
| Página blob | Chave da conta de armazenamento | Método mais antigo |

```sql
CREATE CREDENTIAL [https://myacc.blob.core.windows.net/sqlbackups]
  WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = '<sas token>';
BACKUP DATABASE contoso TO URL = 'https://myacc.blob.core.windows.net/sqlbackups/contoso.bak'
  WITH COMPRESSION, CHECKSUM;
RESTORE DATABASE contoso FROM URL = 'https://myacc.blob.core.windows.net/sqlbackups/contoso.bak'
  WITH NORECOVERY;   -- keep restoring diffs and logs, then WITH RECOVERY
```
- `NORECOVERY` deixa o banco de dados **Restaurando**, para que você possa adicionar mais log backups. Isso encurta o tempo de inatividade.
- Backups **mais de 200 GB**: **listra** sobre vários arquivos / URLs e comprima.
- Se você fizer backup de um arquivo local, faça o upload com **AzCopy** (linha de comando) ou **Azure Storage Explorer** (GUI).
- SQL em um VM não "sabe" que está no Azure. Você mesmo configura a credencial.

**Exemplo:** Um banco de dados de contas 300 GB se move em um fim de semana longo. Sexta-feira: full backup listou mais de 4 URLs. Sábado: restaurar com NORECOVERY, então o log final backup, então RECOVERY.

## Distributed availability group
A **distribuído AG** junta-se dois availability groups, por exemplo on-premises e Azure. Os fluxos de dados vivem até você falhar.
- O tempo de inatividade é apenas o **failover**.
- De um **instância autônoma**: nenhum cluster Windows ou listener necessário.
- De um **existente AG**: precisa de um cluster Windows failover e listener.
- Não é necessário armazenamento compartilhado.
- Precisa de **VPN ou ExpressRoute** entre os locais.
- Configurar com **T-SQL**. O velho "Add Azure Replica Wizard" só trabalhou com clássico VMs, que se aposentou em setembro de 2023.

**Exemplo:** O CRM de um varejista global não pode ser baixado. Um AG distribuído o copia para Azure VMs por duas semanas. Cutover é um planned failover de alguns segundos.

## Outros métodos
| Método | Tempo de inatividade | Quando |
|--------|----------|------|
| **DMS** (portal, CLI, PowerShell) | Online ou offline | Restaura o seu backups a partir de uma partilha ou Blob. Precisa de um SHIR para partilhas de rede |
| **Azure Migrate** | Baixa | Move o **servidor inteiro** (OS + SQL) como está |
| **Log shipping** | Baixa | Registro do navio backups, log final no cutover. Mais simples que AGs |
| **Transactional replication** | Baixa | Always On disponível |
| **Detach/attach de URL** | Alta | Mover arquivos `.mdf` / `.ldf` para Blob, anexar em VM |
| **Converter em VHD** | Média | Faça o upload de todo o disco VM. Versões antigas, sistema + banco de dados do usuário |
| **BACPAC** | Média | Bancos de dados pequenos e simples |
| **bcp** | Alta | Apenas dados de nível de tabela |
| **Azure Data Factory** | Baixo/médio | Muitas fontes, transforma |
| **Serviço de Importação/Exportação Azure** | Offline | Navio **discos físicos** quando a rede é muito lenta |

**DMS a um VM, coisas para saber:**
- DMS faz **não tomar backups**. Você fornece-los (comprimido é mais rápido).
- Backups em um **compartilhamento de rede** precisa de um **SHIR**. Você não pode reutilizar um Azure Data Factory SHIR.
- Até **10 migrações simultâneas** por SHIR.
- Em Blob Storage, manter cada banco de dados em **sua própria pasta**.
- Você **não pode substituir** um banco de dados existente no alvo. Solte-o ou use um novo nome.

Observação: DMA (aposentado em julho de 2025) também poderia mover **logins e funções de servidor** para um VM. Hoje, script logins você mesmo.

## Para lembrar na prova
- Tempo de inatividade próximo de zero para um SQL VM " → **distribuído AG**.
- "Simples, tempo de inatividade OK" → **backup a URL + restaurar**.
- Backup a URL: **bloco bloco blob + SAS** é recomendado.
- Mais de 200 GB → **listrado** backups.
- `WITH NORECOVERY` mantém o banco de dados pronto para mais restaurações.
- "Mova todo o servidor como está" → **Azure Migrate**.
- "Rede muito lenta para 50 TB" → **Azure Import/Export** (discos de navio).
- DMS compartilhamento de rede → **SHIR** necessário.

## Teste rápido
1. Que tipo de credencial dá uma bolha de bloco backup?
   <details><summary>Resposta</summary>Um token SAS (assinatura de acesso compartilhado). Uma chave de conta de armazenamento dá uma bolha de página.</details>
2. Você migra um SQL Server autônomo com um AG distribuído. Você precisa de um cluster do Windows na fonte?
   <details><summary>Resposta</summary>Não. Apenas uma fonte AG existente precisa de um cluster e listener.</details>
3. O banco de dados restaurado deve aceitar mais log backups. Que opção você usa?
   <details><summary>Resposta</summary>WITH NORECOVERY. Use WITH RECOVERY apenas na última restauração.</details>
