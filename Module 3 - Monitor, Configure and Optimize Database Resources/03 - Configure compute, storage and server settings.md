# Configurar configurações de computação, armazenamento e servidor

> **Habilidades cobradas:** Configurar as configurações do servidor para desempenho, Configurar Resource Governor para desempenho, Configurar recursos de computação e armazenamento para escalabilidade
> **Tempo de leitura:** ~12 min

## Conceito central
Um banco de dados precisa de **compu** suficiente (CPU e memória) e rápido o suficiente **armazenamento** (discos). Muito pouco e é lento. Demasiado e você desperdiça dinheiro. Em uma Azure VM você escolhe discos, tamanho VM e configurações do servidor. Em Azure SQL Database e Managed Instance, o Azure faz muito disso, e você escolhe principalmente uma camada e tamanho.

Pense em uma cozinha de restaurante. Os chefs são o CPUs, a bancada é memória e a geladeira é armazenamento. Um chef rápido com uma pequena bancada e uma geladeira lenta ainda serve comida lentamente.

## Armazenamento para SQL Server on Azure VMs
Use **discos gerenciados** para dados e arquivos de log, **Blob storage** para backups ("backup a URL"), e **Azure Files** para armazenamento compartilhado em failover cluster instances.

### Tipos de discos gerenciados
| Disco | Melhor para | Notas |
|------|----------|-------|
| **Disco Ultra** | Cargas de trabalho mais críticas | IOPS muito alto (operações de entrada/saída por segundo), com latência inferior a 1 ms |
| **Premium SSD v2** | A maioria dos dados e logs de produção modernos | Latência de sub-ms; tamanho, IOPS e taxa de transferência definida separadamente; geralmente mais barato do que Ultra |
| **Premium SSD** | Produção | Boa velocidade e preço |
| **Padrão SSD** | Dev/test | Velocidade média |
| **Padrão HDD** | Dev/test, arquivo | mais lento |

Para uso da produção **Premium SSD v2, Premium SSD ou Ultra Disk**. Premium SSD v2 Recomenda-se com o **Ebdsv5** VM série.

### Práticas recomendadas de disco
- Coloque **dados e arquivos de log em discos separados**.
- **Discos de dados:** ligar o **read-only host cache** (leituras mais rápidas). Isso funciona em **Somente Premium SSD**. Premium SSD v2 e Ultra Disk não suportam cache host, e nem discos de 4 TiB ou mais. Pare SQL Server antes de alterar as configurações de cache.
- **Discos de registro:** conjunto de cache para **Nenhum**.
- Coloque **Tempdb no local D: unidade** (temporário local SSD). É limpo no reboot, mas tempdb é reconstruído em todos os começos de qualquer maneira.
- Ative o **Inicialização instantânea do arquivo** (arquivos de dados crescem mais rápido porque o SQL Server pula zeros).
- Formato de discos de dados com um **Tamanho da unidade de alocação 64- KB**.
- Planeje cerca de **20% extra IOPS e taxa de transferência** para picos.
- Ligado **Série M** VMs, use **Gravar Acelerador** para a **log** disco (nunca para dados).

**Disk Striping:** combinar vários discos em um volume com **Espaços de armazenamento** (Windows) para adicionar o seu IOPS e tamanho. Faça **E** ligar redundância na piscina; Azure já mantém cópias.

**Exemplo:** 10 Premium SSDs de 1 TB com 5.000 IOPS cada, combinados com striping, formam um volume de 10 TB com **50.000 IOPS**.

## Escolhendo e redimensionando o VM (computa)
SQL Server é licenciado **por núcleo**, so more cores = higher licence cost. Memory and IOPS Muitas vezes importa mais do que CPU.

**Famílias VM:**
- Os tamanhos **Memória-otimizada** são recomendados para produção: uma relação memória-para-vCore de **8:1** e pelo menos **4 vCores**.
- **Ebdsv5** é o ponto de partida recomendado (melhor preço-desempenho).
- **Série Mbdsv3/M** para cargas de trabalho de missão crítica ou maiores.
- **Séries D e B** (general purpose) são para desenvolvimento / teste.

**Núcleos restritos:** um tamanho VM que mantém toda a memória, armazenamento e E/S de um grande VM, mas expõe apenas **½ to ¼** do seu vCPUs (por exemplo **M64-32ms**, **M64-16ms**). O custo de computação permanece o mesmo, mas você paga licenças SQL Server por menos núcleos.

**Exemplo:** Um banco de dados de relatórios precisa de 512 GB de memória, mas apenas luz CPU. Em vez de licenciar 64 núcleos, o DBA escolhe um tamanho de núcleo restrito com 16 núcleos ativos e a mesma memória. A conta da licença cai muito.

**Redimensionando um VM:**
- Você pode redimensionar a qualquer momento à medida que a carga de trabalho cresce ou encolhe.
- Azure **reinicializações** o VM, então espere um **interrupção curta**.
- Sometimes you must **deallocate** (completamente parado) o VM primeiro, por exemplo, quando se move para um VM Família.

## Armazenamento em Azure SQL Database e Managed Instance
- Escolha um **nível de serviço** (um nível de serviço com sua própria velocidade e preço).
- Para melhor IOPS e menor latência, use **Business Critical** ou **Hyperscale**.
- **Azure SQL Database** tem apenas **um arquivo de dados** (Hyperscale geralmente vários). Você não pode adicionar arquivos.
- **Managed Instance** permite adicionar arquivos e alterar o tamanho dos arquivos, mas não escolher discos físicos.

### temdb
**temdb** é um banco de dados de sistema compartilhado para tabelas temporárias, tipos, resultados temporários e versões de linha.
| Plataforma | temdb arquivos |
|----------|--------------|
| **SQL Server** | A configuração cria um arquivo por processador lógico, **até 8**. Mantenha os tamanhos iguais. Se a contenção permanecer, adicione arquivos em **múltiplos de 4** (até o número de núcleos). |
| **Azure SQL Database** | O número de arquivos cresce com vCores (e.g. 2 vCores = 4 files), **max 16**. Você não pode mudá-lo. |
| **Managed Instance** | **12 arquivos de dados** por padrão. Você **pode** altera o número (máximo 128), crescimento e tamanho máximo. Configurações sobrevivem reinicializações e failovers. |

Memory-optimized tempdb metadata (SQL Server 2019+) é **E** disponível em Azure SQL Database ou Managed Instance.

## Escalando computação e armazenamento em PaaS
- **Escala para cima/para baixo:** mudar o vCores ou nível de serviço. Use isso quando CPU, memória, I/O ou taxa de log permanece no limite.
- **Hyperscale** separa computação do armazenamento:
  - O armazenamento cresce automaticamente até **128 TB** por banco de dados.
  - A computação pode ser dimensionada sozinha.
  - Add **ler réplicas**: até **4 HA réplicas**, até **30 réplicas nomeadas**, além de geo-replicas. Envie read-only trabalhar lá com `ApplicationIntent=ReadOnly`.
  - Mover-se para Hyperscale é one-way, exceto que você pode voltar para **General Purpose dentro de 45 dias**.

**Exemplo:** O banco de dados de um site de notícias cresce 5 TB por ano. Em **Hyperscale**, o armazenamento cresce por conta própria e as réplicas nomeadas atendem à equipe de reportagem.

## Configurações do servidor para desempenho
Estas são principalmente configurações **nível de instância** em SQL Server (VM) e Managed Instance, definido com `sp_configure`.

| Configuração | O que faz | Dica |
|---------|--------------|-----|
| **grau máximo de paralelismo (MAXDOP)** | Quantos núcleos CPU uma consulta pode usar de uma vez | Too high = more memory use and `CXPACKET` espera em sistemas ocupados |
| **limiar de custo para paralelismo** | Quão "cara" uma consulta deve ser antes de SQL Server executá-la em paralelo (padrão 5) | Levantar para que pequenas consultas fiquem single-threaded |
| **memória do servidor max** | Limite superior de memória SQL Server leva | Configure-o para que o OS mantenha memória suficiente |
| **otimizar para cargas de trabalho ad hoc** | Armazena apenas um pequeno esboço para um plano de consulta na primeira vez que uma consulta é executada | Salva memória quando muitas consultas são executadas apenas uma vez |

```sql
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'max degree of parallelism', 4; RECONFIGURE;
EXEC sp_configure 'cost threshold for parallelism', 50; RECONFIGURE;
```

**Azure SQL Database** você não pode usar `sp_configure`. Defina MAXDOP por banco de dados em vez disso:
```sql
ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP = 4;
```
Novo padrão Azure SQL databases para **MAXDOP 8**. Você também pode usar a dica de consulta `OPTION (MAXDOP n)` para uma consulta.

**Exemplo:** Um sistema de pedidos em um VM de 16 núcleos mostra muitas esperas `CXPACKET`. Pequenas consultas estão indo paralelamente e lutando por CPU. O DBA define MAXDOP para 8 e aumenta o limite de custo para o paralelismo, então apenas grandes relatórios usam muitos núcleos.

## Resource Governor
**Resource Governor** limita o quanto **CPU, memória e I/O** cada grupo de sessões pode usar. Ele impede que uma carga de trabalho passe fome em outras pessoas.
- Disponível em **SQL Server** e **Azure SQL Managed Instance**.
- **Não disponível em Azure SQL Database** (a plataforma gerencia recursos).
- **SQL Server 2025:** agora também na edição **Padrão** (antes: Enterprise/Developer apenas) e pode limitar **espaço tempdb** por grupo de carga de trabalho.

### Três partes
1. **pool de recursos** – uma fatia de recursos do servidor.
   - Piscina **Interna**: para o próprio SQL Server; não pode ser limitada.
   - Piscina **Padrão**: onde as sessões vão se não forem classificadas.
   - Piscinas **Definido pelo usuário**: você as cria.
   - Configurações: MIN / MAX CPU %, **CAP_CPU_PERCENT** (uma tampa dura, usada mesmo quando o servidor está ocioso), MIN / MAX memória %, MIN / MAX IOPS por volume, afinidade NUMA.
2. **Grupo de carga de trabalho** – um grupo de sessões. Cada grupo pertence ao **uma piscina**; uma piscina pode ter muitos grupos.
3. **Função do classificador** – uma função de usuário que executa **em login** e retorna o nome do grupo de carga de trabalho para a nova sessão. Mantenha-o rápido, porque ele funciona em todas as conexões.

```sql
CREATE FUNCTION dbo.RGClassifier() RETURNS sysname WITH SCHEMABINDING
AS BEGIN
  IF SUSER_NAME() = 'ReportUser' RETURN 'ReportGroup';
  RETURN 'default';
END;
GO
ALTER RESOURCE GOVERNOR WITH (CLASSIFIER_FUNCTION = dbo.RGClassifier);
ALTER RESOURCE GOVERNOR RECONFIGURE;
```

- As alterações só se aplicam ao **novas sessões**.
- Ligado **Managed Instance**, a configuração vive em `master` e **réplicas a réplicas secundárias**. Em SQL Server availability groups você deve configurar **Cada réplica**.

**Exemplo:** Um SQL Server executa pedidos on-line de uma loja e seus relatórios. Relatórios log in as `ReportUser`. O DBA coloca-los em uma piscina tampada em 30% CPU. O checkout permanece rápido mesmo quando um grande relatório é executado na hora do almoço.

## Para lembrar na prova
- Discos de produção: **Premium SSD v2, Premium SSD ou Ultra Disk**. Standard = dev/test.
- **Read-only cache em discos de dados, Nenhum em discos de log.** Anfitrião cache apenas no Premium SSD (não v2, não Ultra, não 4 TiB +).
- **Tempdb no local D: unidade** em um VM.
- **Núcleos restritos** = same memory and I/O, fewer cores, lower SQL Custo da licença.
- VM resize = **reiniciar** (breve): uma mudança de família pode precisar **deallocate**.
- arquivos tempdb: SQL Server até 8, em seguida, múltiplos de 4; SQL DB fixo (max 16); MI 12 por padrão e mutável.
- **limiar de custo para paralelismo** controla *Quando* uma consulta vai paralelo; **MAXDOP** controla núcleos *Quantos*.
- Azure SQL Database: não `sp_configure`; use **ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP**.
- **Resource Governor: Somente SQL Server e MI**, classificador é executado em login, piscina interna não pode ser limitada.
- **CAP_CPU_PERCENT** é um limite difícil; MAX_CPU_PERCENT só se aplica quando há contenção.

## Teste rápido
1. Uma carga de trabalho de memória pesada, CPU - leve em uma Azure VM tem altos custos de licença SQL Server. O que você escolhe?
   <details><summary>Resposta</summary>Um tamanho VM de núcleo restrito. Ele mantém a memória e E / S, mas expõe menos núcleos à licença.</details>
2. Você deve parar um relatório login de usar mais do que 20% CPU no Azure SQL Managed Instance, mesmo quando o servidor está ocioso. O que você usa?
   <details><summary>Resposta</summary>Resource Governor com um pool de recursos usando CAP_CPU_PERCENT = 20 and a classifier function that sends the login para o grupo de carga de trabalho dessa piscina.</details>
3. Qual configuração de cache você usa para um SSD Premium que contém o log de transação?
   <details><summary>Resposta</summary>As gravações de log devem ir direto para o disco; O cache read-only é para discos de dados.</details>
