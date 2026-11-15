# Migrar para Azure SQL Managed Instance

> **Habilidades cobradas:** Implementar uma estratégia de migração online, implementar uma migração para o Azure, implementar uma migração entre serviços Azure SQL, implementar Azure SQL Managed Instance database copy e mover
> **Tempo de leitura:** ~10 min

## Conceito central
Managed Instance entende **nativo SQL Server backups**. Então a maioria das migrações MI são "restaurar backups" (LRS) ou "cópia ao vivo" (link MI). Azure Arc envolve ambos em um fluxo portal. Dentro do Azure, você pode **copiar ou mover** um banco de dados de um MI para outro.

Analogia: mover-se para um apartamento com serviço que aceita seu próprio mobiliário. Você pode enviá-lo em caixas (backups), ou manter um link de vídeo ao vivo até o dia em movimento (Link MI).

## Suas opções
| Método | Online? | Notas |
|--------|---------|-------|
| **Log Replay Service (LRS)** | Sim (modo contínuo) | Restaura o seu backups de Blob Storage |
| **Managed Instance link** | Sim | Replicação quase em tempo real usando um AG distribuído |
| **Migração SQL Server no Azure Arc** | Sim | Fluxo de portal que usa link MI ou LRS |
| **Nativo backup /restaurar** | Não | `RESTORE FROM URL` de um full backup |
| **Transactional replication** | Ambos | MI pode ser editor, distribuidor ou assinante |
| **BACPAC** | Não | Somente SqlPackage ou SSMS (sem importação de portal para MI) |

**Exemplo:** O banco de dados de catálogo de um varejista esportivo alimenta o site, o aplicativo de vendas e os relatórios. Ele usa SQL Agent e junções de bancos de dados cruzados. Eles escolhem MI para que nenhum código de aplicativo mude.

## Conectar on-premises a MI
- MI vive em um **sub-rede privada**. On-premises chega a ele por **site-site VPN** ou **ExpressRoute**. Point-to-site VPN é para uma máquina de administração.
- **endpoint público** opcional na porta **3342**. A DBA liga-o; a equipe de rede abre a porta.

## Nativo backup e restaurar
```sql
CREATE CREDENTIAL [https://acct.blob.core.windows.net/backups]
  WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = '<sas token>';
RESTORE DATABASE Sales FROM URL = 'https://acct.blob.core.windows.net/backups/Sales.bak';
```
Limites de restauração MI:
- Apenas a partir de **Azure Blob Storage** (sem disco local).
- Restaura bancos de dados únicas, não uma instância inteira. Não é possível substituir uma banco de dados existente.
- Não **FILESTREAM/FileTable** backups.
- Objetos **In-Memory OLTP** ativos → **Business Critical** apenas, não General Purpose.
- **TDE** backups precisa do certificado ou chave em MI primeiro.
- Para aplicar **diferencial** backups você precisa de LRS.

## Log Replay Service (LRS)
LRS é log shipping em MI. Você toma backups; LRS restaura-os.
1. Pegue o total, diferencial e log backups. Carregue-os para **Blob Storage**.
2. Start LRS. Ele restaura o que encontra.
3. Mantenha o registro de upload backups.
4. **Cutover**: complete a migração e troque de aplicativo.

| Modo | Como | Utilização |
|------|-----|-----|
| **Autocompletar** | Carregue o **cadeia inteira primeiro**, dê o último nome do arquivo. LRS termina por si só. | Tempo de inatividade planejado |
| **Contínuo** | LRS continua a observar novos arquivos. Você completa manualmente. | Tempo de inatividade mínimo |

Factos-chave:
- Fonte: **SQL Server 2012+, todas as edições**, qualquer host.
- O banco de dados permanece no estado **Restaurando** (não legível) até o cutover.
- Deve terminar dentro do **30 dias**, ou o job é cancelado.
- Acesso ao armazenamento: **token SAS ou managed identity**Não os dois. SAS = **Ler + Listar apenas**.
- Comandos: `Start-/Get-/Complete-/Stop-AzSqlInstanceDatabaseLogReplay` ou `az sql midb log-replay start|show|complete|stop`.
- Backups: **um backup por arquivo**, `WITH CHECKSUM`, `COMPRESSION`, dividir grande backups em vários arquivos.
- Cutover para **Business Critical** é lento (deve semear 3 réplicas). Prefere link MI para BC.

Quando LRS se encaixa: você precisa de controle total, não pode instalar DMS ou a fonte não pode abrir portas para o Azure. LRS só precisa de Blob Storage.

**Exemplo:** Um servidor da Edição Padrão de 2014 não pode usar o link MI. LRS no modo contínuo mantém o MI atualizado por uma semana. O cutover no sábado leva minutos.

## Managed Instance link
O link MI usa um **distributed availability group** para replicar um banco de dados de SQL Server para MI em tempo quase real.

| versão SQL Server | Direção |
|-------------------|-----------|
| 2016 SP3, 2017 CU31, 2019 CU20 (2016/2017 também precisa do pacote Azure Connect) | **Só de ida.** Failover para MI quebra o link; nenhum failback. |
| **2022, 2025** | **Dois sentidos.** Online failover **e failback**. MI deve usar a política de atualização correspondente (não sempre atualizada). |

Factos-chave:
- **One database = one link** (um modo multi-banco de dados está em pré-visualização).
- Edições: **Enterprise, Standard, Desenvolvedor**.
- Precisa de conectividade **privado** (VPN, ExpressRoute ou VNet peering). Ele usa o ponto final MI VNet-local, porta **5022**.
- MI DB é **legível** durante a migração (bom para relatórios).
- **Não há limite de tempo.** Cutover leva segundos.
- An MI com um link **Não pode estar em um failover group**.
- Não FILESTREAM : FileTable.
- Configure com o **assistente SSMS**, T-SQL ou PowerShell.
- Também usado para **DR** (MI como standby) e **ler offload**.

## LRS vs MI link
| | link MI | LRS |
|-|---------|-----|
| Tecnologia | AG distribuído | Restauração Backup de Blob |
| Fonte | SQL Server 2016+ | SQL Server 2012+ |
| Edições | Enterprise, Standard, Desenvolvedor | Todos |
| Alvo legível durante a migração | Sim | Não |
| Prazo limite | Nenhum | 30 dias |
| Melhor para | Business Critical, segundos de tempo de inatividade | Versões antigas, Web/Express, endpoint público |
| Migração inversa | Possível (2022/2025) | Não |

## Migração SQL Server no Azure Arc
Para um **Arc-enabled** SQL Server, o portal tem **Migração → Migração de banco de dados** com quatro telhas:
1. Bloqueadores **Avalie a instância de origem:** e um MI de tamanho direito. Funciona **Todos os fins de semana** automaticamente, ou sob demanda.
2. **Selecione o alvo:** escolher ou criar um MI.
3. **Migrar dados:** escolher **link MI** (em tempo real) ou **LRS** (log shipping).
4. **Monitor e recorte:** estado do relógio e **atraso de replicação**, em seguida, cortar.

- Ele usa **link somente MI ou LRS** (não DMS, não BACPAC).
- link MI aqui: até bancos de dados **10** de uma só vez (extensão recente do Azure para SQL Server). LRS: ilimitado.
- Para LRS você deve primeiro carregar pelo menos um **full backup**.
- **Copiloto** pode explicar a avaliação e executar etapas.
- **Failover cluster instances não pode ser migrado** por aqui.
- Não há migração reversa no fluxo do Arco.

**Exemplo:** A empresa de bicicletas Arc habilitado seus servidores no ano passado. O DBA abre o portal, lê a avaliação de fim de semana, cria o MI recomendado e inicia um link MI. Cutover é um clique.

## MI database copy e mover
Copie ou mova um **banco de dados do usuário** de um MI para outro, **online**, usando a tecnologia Always On.

| Operação | Fonte após a conclusão | Uso típico |
|-----------|------------------------|-------------|
| **Copiar** | Estadias online | Teste de atualização da produção |
| **Mover** | **Dropped** | Reequilibrar a carga, consolidar instâncias |

etapas: **Iniciar** → semeadura → **Pronto para conclusão** (mudanças continuam fluindo) → **Completar** (você clica). Você deve completar dentro de **24 horas**, ou auto-cancela e deixa cair o alvo.

```powershell
Copy-AzSqlInstanceDatabase -DatabaseName SalesDb -InstanceName mi-prod -ResourceGroupName rg-prod `
  -TargetInstanceName mi-test -TargetResourceGroupName rg-test
Complete-AzSqlInstanceDatabaseCopy -DatabaseName SalesDb -InstanceName mi-prod -ResourceGroupName rg-prod `
  -TargetInstanceName mi-test -TargetResourceGroupName rg-test
# Move: Move-AzSqlInstanceDatabase. CLI: az sql midb copy|move start|complete|cancel
```

Regras:
- **Apenas na mesma região.** Assinatura cruzada OK dentro do **mesmo inquilino Entra**.
- Somente bancos de dados de usuários. A instância de origem e de destino deve diferir. Sem renome.
- Operações de até **8** ao mesmo tempo por instância de origem.
- **Não** para bancos de dados em um **failover group** ou com **link MI**.
- Diferentes VNets precisam de peering e portas **5022** e **11000–11999** abertas.
- **PITR backups não se mexa.** O alvo inicia um novo backup chain.
- TDE com cliente-gerenciado chave → alvo precisa do **Chave chave Vault mesmo**.
- O alvo **Política de atualização** deve ser igual ou superior.
- Mover é projetado para **perda de dados zero**.

**Outras maneiras de realocar um banco de dados MI:**
| Necessidade | Utilização |
|------|-----|
| Outra região | Geo-restore, failover group, ou COPY_ONLY backup a URL + restaurar |
| Como foi ontem | Point-in-time restore |
| instância inteira para DR | Failover group |

## Para lembrar na prova
- "SQL Server 2012, edição Web, a MI " → **LRS**.
- "Continue adicionando log backups" → LRS **contínuo**, não autocompletar. LRS limit = **30 dias**.
- "Alvo legível, Business Critical, segundos de inatividade" → **link MI**.
- Para fazer failback para on-premises, use MI link com SQL Server **2022/2025**.
- "Servidor habilitado para Arc, migrar do portal" → **Migração de arco** (usa o link MI ou LRS).
- "Refresh teste MI de prod, prod permanece online" → **database copy**.
- "Mover para MI em outra região" → **E** copiar / mover. Use geo-restore ou backup / restaurar.
- "Pronto para conclusão, nada acontece" → você deve clicar em **Completar** dentro **24 h**.

## Teste rápido
1. Sua fonte é SQL Server 2019 Enterprise. Você precisa de uma cópia MI legível durante a migração. LRS ou MI link?
   <details><summary>Resposta</summary>MI link. LRS mantém o banco de dados em estado de restauração, por isso não pode ser lido.</details>
2. Uma operação de cópia é "Pronto para conclusão" por dois dias, depois desaparece. Por quê?
   <details><summary>Resposta</summary>Não foi concluído dentro de 24 horas, por isso foi cancelado automaticamente.</details>
3. Quais são os dois motores que a migração do Azure Arc usa?
   <details><summary>Resposta</summary>Managed Instance link e Log Replay Service.</details>
