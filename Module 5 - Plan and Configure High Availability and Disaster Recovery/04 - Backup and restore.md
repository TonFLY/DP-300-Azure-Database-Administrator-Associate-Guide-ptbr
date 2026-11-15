# Backup e restaurar

> **Habilidades cobradas:** Recomendar um banco de dados backup e restaurar estratégia, Executar um banco de dados backup usando ferramentas nativas, Executar uma restauração de banco de dados usando ferramentas nativas, Organizar uma restauração de banco de banco de dados para um ponto no tempo, Configurar a retenção backup de longo prazo, Backup e restaurar um banco de dados usando T-SQL, Backup e restaurar do armazenamento em nuvem
> **Tempo de leitura:** ~15 min

## Conceito central
Backups são os erros de cópia **last line of defence**. HA e DR também: se alguém exclui uma tabela, cada réplica a exclui. Apenas um backup permite que você volte no tempo. Em um VM, **tu** configurar backups. Em SQL Database e SQL MI, **Azure** leva-los automaticamente.

Pense em um diário que você fotocopia. Uma cópia completa todos os domingos, as páginas alteradas todos os dias e uma foto de cada nova linha a cada poucos minutos. Com todos os três você pode reconstruir o diário como era a qualquer momento.

## Tipos Backup
| Tipo | O que ele contém | Agenda de exemplos |
|---|---|---|
| **Completo** | Toda a banco de dados. A base para cada restauração | Todas as noites ou todas as semanas |
| **Diferencial** | Alterações **Desde o último período completo** backup | A cada poucas horas |
| **Registro de transacção** | Todas as alterações **Desde o último registro** backup. Permite restauração **point-in-time** | Cada minuto 5–15 |

- Um log backup também **truncates** o log (marca o espaço antigo como reutilizável; ele faz **E** encolher o arquivo). Sem log backups, o arquivo de log continua crescendo.
- **COPY_ONLY** = an extra backup isso faz **E** Mudar o normal backup chain. Use-o para cópias one-off.

## Modelos de recuperação
| Modelo | Log backups? | Point-in-time restore? | Uso para |
|---|---|---|---|
| **FULL** | Sim | Sim | Produção, baixo RPO |
| **BULK_LOGGED** | Sim | Sim, com limites (não dentro de uma operação em massa) | Grandes cargas a granel |
| **SIMPLE** | **Não** | **Não** | Dev/test |

**Exemplo:** Um banco de dados de folha de pagamento deve perder no máximo 15 minutos de dados. Ele usa o modelo FULL com log backups a cada 15 minutos.

## Nativo T-SQL backup e restaurar (SQL Server em VM)
Ordem de restauração para um point-in-time restore:
1. Restaure o **Cheio** backup `WITH NORECOVERY`.
2. Restaure o mais recente **diferencial** (opcional) `WITH NORECOVERY`.
3. Restaure cada **log** backup na ordem `WITH NORECOVERY`. Na última use `STOPAT` por um tempo exato.
4. Finish `WITH RECOVERY` (traz o banco de dados online).

Se você usar `RECOVERY` muito cedo, você **Não posso.** aplicar mais backups.

```sql
BACKUP DATABASE Sales TO DISK = 'F:\Backup\Sales_full.bak' WITH COMPRESSION, CHECKSUM;
BACKUP LOG Sales TO DISK = 'F:\Backup\Sales_log1.trn';

RESTORE DATABASE Sales FROM DISK = 'F:\Backup\Sales_full.bak' WITH NORECOVERY, REPLACE;
RESTORE LOG Sales FROM DISK = 'F:\Backup\Sales_log1.trn'
  WITH STOPAT = '2026-10-29 10:34:00', RECOVERY;
```

### Tail-log backup
Antes de restaurar um banco de dados danificado, faça backup do **A cauda do tronco** (as alterações feitas desde o último log backup). Sem ele, esses últimos minutos são perdidos.
```sql
BACKUP LOG Sales TO DISK = 'F:\Backup\Sales_tail.trn' WITH NORECOVERY;   -- add CONTINUE_AFTER_ERROR if damaged
```
O `NORECOVERY` aqui deixa o banco de dados no estado de restauração, pronto para a sequência de restauração.

Opções úteis: **COPY_ONLY**, **COMPRESSION**, **Criptografia** (precisa de um certificado), **CHECKSUM**, **MOVE** (colocar arquivos em novos caminhos), **STATS** (mostrar progresso).

**Exemplo:** At 10:35 someone runs `DELETE FROM Orders` com não WHERE. O DBA restaura o full backup e logs para um novo banco de dados com `STOPAT 10:34`, em seguida, copia as linhas perdidas para trás.

Ferramentas nativas: você também pode fazer backup e restaurar em **SSMS** (banco de dados com o botão direito do mouse → Tasks → Back Up / Restore). Nota: Azure Data Studio foi aposentado em 28 de fevereiro de 2026, mas perguntas antigas ainda podem mencioná-lo.

## Backup a URL (Azure Blob Storage)
SQL Server pode fazer backup **direto para Azure Blob Storage** e restaurar a partir dele. Esta é a resposta para " backup e restaurar a partir do armazenamento em nuvem". Funciona para SQL Server em VMs, on-premises SQL Server e SQL MI.

Formato do caminho: `https://<account>.blob.core.windows.net/<container>/<file>.bak`

Duas maneiras de entrar no armazenamento:
| Credencial | Tipo de bolha | Notas |
|---|---|---|
| **Assinatura de Acesso Compartilhado (SAS)** | **Bloco Blob** | **Recomendado**: mais barato, mais seguro, pode risca |
| Nome da conta de armazenamento + chave | Página blob | Método mais antigo. Um backup max **1 TB** |

```sql
CREATE CREDENTIAL [https://myacc.blob.core.windows.net/sqlbackups]
  WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = '<SAS token without ?>';

BACKUP DATABASE Contoso TO URL = 'https://myacc.blob.core.windows.net/sqlbackups/contoso.bak';
BACKUP LOG Contoso TO URL = 'https://myacc.blob.core.windows.net/sqlbackups/contoso_log.trn';
RESTORE DATABASE Contoso FROM URL = 'https://myacc.blob.core.windows.net/sqlbackups/contoso.bak'
  WITH NORECOVERY;
```

- Com SAS, a credencial **o nome é o recipiente URL** e a identidade é `SHARED ACCESS SIGNATURE`.
- Um bloco de bloco detém cerca de **195 GB**. Para maior backups, **listra** em até **64 URLs** (cerca de 12.8 TB).
- **SQL Server 2022+** também pode fazer backup até **Armazenamento S3-compatible** (`TO URL = 's3://...'`, identidade de credencial `S3 Access Key`).
- Em SSMS, escolha **URL** como o backup ou restaurar mídia.

**Exemplo:** Uma empresa de logística faz backup de sua banco de dados on-premises para Blob Storage com um token SAS todas as noites. Durante uma broca DR, restaura o backup em uma Azure VM.

## Opções Backup para SQL Server on Azure VMs
Azure faz **E** voltar para cima SQL bancos de dados em a VM A não ser que você o crie.

| Opção | Restaurar uma banco de dados? | Melhor para |
|---|---|---|
| **Azure VM backup** (snapshot de todo o VM, VSS app-consistente) | No (todo VM /disks) | Recuperação completa VM, ransomware |
| **Azure Backup para SQL Server em Azure VMs** (trabalho de carga-consciente) | Sim, com point-in-time | Central, baseado em cofre backups e longa retenção |
| **Automated Backup** (SQL IaaS Agent extension) | Sim | backups automático simples para sua conta de armazenamento |
| **backups nativo para disco ou Azure Files** | Sim | SQL backups você controla |
| **Nativo backup a URL** | Sim | Armazenamento em nuvem backups você controla |

Detalhes:
- **Azure Backup para SQL Server em Azure VMs** instala a extensão `AzureBackupWindowsWorkload` e fluxos nativos completos, diferenciais e log backups para um **Serviços de Recuperação abóbada**. Log backups tão frequentemente quanto cada **15 minutos**. Snapshot - baseado backup de instâncias SQL está em **pré-visualização**.
- **Automated Backup** utiliza o **SQL IaaS Agent extension** (VM registrado como a **Máquinas virtuais SQL** recursos) e configura **Managed Backup para Microsoft Azure**. retenção **dias 1–90** (padrão 90). Log backups Cada **minutos 5–60**. Os bancos de dados do sistema e a encriptação são opcionais. Necessidades **SQL Server 2016+** (Standard, Enterprise, Desenvolvedor), **FULL** um modelo de recuperação padrão ou único chamado exemplo, e um **Windows** VM. Ele não restaura para você: restaurar com T-SQL : SSMS.
- Se VM snapshot backups causar problemas, defina o valor do registro `USEVSSCOPYBACKUP = TRUE` sob `HKLM\SOFTWARE\MICROSOFT\BCDRAGENT` para que o VSS backup seja copy-only.
- Não escreva backups para o **disco temporário** (D:). É limpo.
- **Não misture** log métodos backup (por exemplo Automated Backup **e** seu próprio log backup job). Ele quebra o **cadeia log** e restaura falha. Escolha a ferramenta **um** para log backups.

## backups automático em SQL Database e SQL MI
| Backup | Com que frequência |
|---|---|
| Completo | cada **semana** |
| Diferencial | Cada **12 ou 24 horas** (SQL DB padrão: 12 h vCore, 24 h DTU) |
| Registro | Sobre cada **10 minutos** |

- **Hyperscale** usa **armazenamento snapshots** em vez disso (quase snapshot backups, restaurações rápidas).
- **Short-term retention** (para PITR): default **7 dias**, set **dias 1–35** (**Nível básico: 1–7 dias**).
- SQL Database: **Não** usuário `BACKUP` comando. Use automático backups, ou exportar um **BACPAC** (um arquivo com esquema e dados) para uma cópia.
- **SQL MI** permite o manual `BACKUP ... TO URL` **somente `WITH COPY_ONLY`**, porque o Azure possui o backup chain. Um banco de dados usando um **chave TDE gerenciada por serviço** não pode ser backup copy-only. Use um **chave TDE gerenciada pelo cliente** em vez disso.

```sql
-- SQL Managed Instance only
BACKUP DATABASE Contoso TO URL = 'https://myacc.blob.core.windows.net/mi/contoso.bak' WITH COPY_ONLY;
```

Mover backups entre produtos:
- SQL Server nativo backup → SQL MI: **Sim** (`RESTORE ... FROM URL`).
- SQL MI copy-only backup → **SQL Server 2022 ou 2025**: sim, quando o MI **Política de atualização** corresponder.
- SQL Database backup → SQL Server: **Não**. SQL MI ↔ SQL Database: **Não**.

## Redundidade de armazenamento Backup
| Opção | Cópias | Sobrevive | Notas |
|---|---|---|---|
| **LRS** (localmente redundante) | 3 em **um datacenter** | Falha do disco/rack | Não para DR |
| **ZRS** (zona redundante) | 3 em **zonas**, uma região | Falha do Datacenter (zona) | Não é uma interrupção regional |
| **GRS** (geo redundante) | 3 locais + 3 no **região pareada** | Interrupção regional | Secundário não legível até failover |
| **RA-GRS** (read-access GRS) | mesmo que GRS | Interrupção regional | Secundário legível a qualquer momento |
| **GZRS** | ZRS + região pareada | Zona e parada regional | |

- SQL DB / MI backups padrão para armazenamento **geo-redundante**. Você pode escolher LRS, ZRS, GRS ou GZRS.
- **Geo-restore funciona apenas com GRS ou GZRS.**
- Para **Hyperscale**, a redundância backup só pode ser definida quando você cria o banco de dados.

**Exemplo:** Um pequeno banco de dados de teste usa LRS para economizar dinheiro. O banco de dados de produção mantém GRS para que possa ser restaurado geograficamente se a região for perdida.

## Point-in-time restore (PITR) em PaaS
- Restaurar para **qualquer segundo dentro do período de retenção**.
- Restaura sempre como um **nova banco de dados** (sem restauração no local). Em seguida, renomear ou renomear o aplicativo.
- Ferramentas: portal (banco de dados → **Restaurar**), Azure CLI, PowerShell, REST API.
- SQL MI: restaurar para a mesma ou outra instância. PowerShell dá o suporte mais completo.

```bash
az sql db restore -g rg-prod -s sql-prod -n SalesDb --dest-name SalesDb_0934 \
  --time "2026-10-29T09:34:00Z"
```

```powershell
Restore-AzSqlDatabase -FromPointInTimeBackup -PointInTime "2026-10-29 09:34" `
  -ResourceGroupName rg-prod -ServerName sql-prod -TargetDatabaseName SalesDb_0934 -ResourceId $db.ResourceId
```

**Exemplo:** At 09:35 a faulty release overwrites prices. The DBA restaura `SalesDb` às 09:34 as `SalesDb_0934` e copia os preços corretos de volta.

### Restaurar um banco de dados excluído
- Os bancos de dados excluídos são mostrados sob a lâmina **Bancos de dados excluídos** do servidor. Restaure o tempo de exclusão com um **novo nome**.
- Se o **O servidor lógico é excluído**, suas bancos de dados e PITR backups são **Perdido**. **LTR backups são mantidos** e pode ser restaurado para outro servidor.
- Para um **managed instance** excluído, entre em contato com o suporte dentro do **5 dias**.

### Geo-restore
- Recupera do **geo-replicado backups** para o **qualquer servidor em qualquer região**.
- Usado quando toda a região está em baixo e você não tem geo-replica.
- RTO mais longo e maior RPO do que geo-replication, porque backups são copiados com um atraso. Precisa de armazenamento GRS ou GZRS backup.

## Long-term retention (LTR)
- Mantém **full backups** por mais de 35 dias, até **10 anos**, para conformidade.
- A política tem quatro configurações: **semanal (W)**, **mensal (M)**, retenção **anual (Y)** e **WeekOfYear** (que a semana backup é mantida como anual).
- Conjunto no portal (servidor → **Backups** → **Políticas de retenção**), PowerShell ou CLI. Funciona para SQL DB e SQL MI.
- Em um MI failover group, defina a política LTR em instâncias **Ambos**.

```bash
az sql db ltr-policy set -g rg-prod -s sql-prod -n SalesDb \
  --weekly-retention P8W --monthly-retention P12M --yearly-retention P10Y --week-of-year 1
az sql db ltr-backup list --location westeurope --server sql-prod --database SalesDb
```

```powershell
Set-AzSqlDatabaseBackupLongTermRetentionPolicy -ResourceGroupName rg-prod -ServerName sql-prod `
  -DatabaseName SalesDb -WeeklyRetention P8W -MonthlyRetention P12M -YearlyRetention P10Y -WeekOfYear 1
Restore-AzSqlDatabase -FromLongTermRetentionBackup -ResourceId $ltr.ResourceId ...
```

**Exemplo:** Um escritório de advocacia deve manter os dados do caso por 7 anos. Ele define a retenção anual `P7Y` e mensal `P12M`.

## Recomendando uma estratégia
| Necessidade | Recomendar |
|---|---|
| Desfazer um erro de 20 minutos atrás (PaaS) | **PITR** |
| Mantenha backups 7 anos (PaaS) | **LTR** |
| Região perdida, sem georreplica (PaaS) | **Geo-restore** (GRS/GZRS backups) |
| SQL VM, ponto-em-tempo por banco de dados, cofre central | **Azure Backup para SQL Server em Azure VMs** |
| SQL VM, simples automático backups para armazenamento | **Automated Backup** (SQL IaaS Agent extension) |
| VM inteiro de volta após ransomware | **Azure VM backup** |
| Armazenamento em nuvem para on-premises ou VM backups | **Backup a URL** com SAS |
| Cópia de uma banco de dados MI para SQL Server 2022 | MI **COPY_ONLY** backup a URL |

Sempre **restaurações de teste**. Um backup só é comprovado quando restaura (lição 5).

## Para lembrar na prova
- Point-in-time restore precisa de **FULL** (ou BULK_LOGGED) e **log backups**. SIMPLE não pode.
- Restore order: full → diff → logs, todos `NORECOVERY`, último um `RECOVERY` (`STOPAT` por um tempo).
- Backup E- URL: **SAS + blocos de blocos** recommended. Credential name = container URL.
- SQL MI manual backups deve ser **COPY_ONLY**. SQL Database tem **Não** usuário backups.
- PaaS: semanal completo, diff 12/24 h, log ~10 min. PITR **dias 1–35**, padrão **7**. Basic max 7.
- O PITR e o banco de dados excluído restauram sempre criar um banco de dados **Novo**.
- **LTR até 10 anos**, W/M/Y + WeekOfYear.
- Armazenamento **Geo-restore precisa de GRS ou GZRS** backup.
- Não misture ferramentas log backup em um VM: ele quebra a cadeia de log.

## Teste rápido
1. Um banco de dados usa o modelo SIMPLE recovery. Você pode restaurá-lo para 10:15 hoje?
   <details><summary>Resposta</summary>No. SIMPLE não tem nenhum log backups, assim que você só pode restaurar ao tempo de um completo ou differential backup.</details>
2. Você deve levar um manual backup de um banco de dados SQL Managed Instance para Blob Storage. Qual opção é necessária?
   <details><summary>Resposta</summary>WITH COPY_ONLY, porque o Azure possui o backup chain normal em MI.</details>
3. O armazenamento Backup para um Azure SQL database está definido como LRS. A região vai para baixo. Você pode geo-restore?
   <details><summary>Resposta</summary>Não. O Geo-restore funciona apenas com o armazenamento GRS ou GZRS backup.</details>
