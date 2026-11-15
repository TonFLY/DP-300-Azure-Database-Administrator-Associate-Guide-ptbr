Precisamos projetar a tabela **FactSales** em um **dedicated SQL pool do Azure Synapse Analytics** com os seguintes requisitos:

* Os dados serão retidos para **cinco anos**.
* A cada ano, dados com mais de cinco anos devem ser excluídos.
* Os dados devem ser **distribuídos uniformemente entre distribuições**.
* Deve **minimizar o tempo de eliminação** → usar **particionamento** por `OrderDateKey`.

---

### Quebra de opções:

* **Distribuição**:

  * `ROUND_ROBIN` → se espalha linhas uniformemente através de distribuições (✅ equilibra a carga, evita desvios).
  * `HASH` → requer uma boa coluna de distribuição (não especificada conforme necessário aqui).
  * `REPLICATE` → duplica dados em todas as distribuições (ruim para uma grande tabela de fatos).
    → Melhor opção: **ROUND \_ ROBIN**.

* **Particionamento**:

  * Como excluímos **por ano**, a chave de partição deve ser `OrderDateKey`.
  * Isso nos permite deixar de lado partições antigas em vez de excluir linha a linha.

---

### Resposta final:

```sql
WITH CLUSTERED COLUMNSTORE INDEX
DISTRIBUTION = ROUND_ROBIN
PARTITION (OrderDateKey RANGE RIGHT FOR VALUES 
(20170101, 20180101, 20190101, 20200101, 20210101))
```

👉 Selecções correctas:

* **ROUND \_ ROBIN**
* **OrderDateKey**

Vamos quebrá-lo:

* A questão: **tempDB contenção** identificado por **Intelligent Insights** em um **Azure SQL Database**.
* Objetivo: reduzir **contenção sobre tempDB**.

---

### Análise das opções:

**A. Implementar memory-optimized tables** ✅

* Memory-optimized tables reduzir ou eliminar o uso de tempDB (uma vez que não há necessidade de tempDB para versionamento de linha ou armazenamento intermediário).
* Isso ajuda diretamente a reduzir a contenção do TempDB.
* Suportado em Azure SQL Database.

**B. Run `DBCC FLUSHPROCINDB`** ❌

* Limpa planos de execução em cache para um banco de dados.
* Não aborda a contenção do TempDB de forma alguma.

**C. Substituir chaves de índice sequenciais por chaves não sequenciais** ❌

* Isso ajuda a reduzir a contenção de trava no **índices** (problema hotspot com chaves de identidade), não no **tempDB contenção**.

**D. Run `DBCC DBREINDEX`** ❌

* Comando depreciado para reconstruir índices (substituído por `ALTER INDEX`).
* Não tem nada a ver com o TempDB.

---

👉 resposta correta:
**A. Implementar memory-optimized tables**


Vamos analisar o cenário:

* **Requisito:** Contagem **novos eventos** em **intervalos de cinco minutos**.
* **Detalhes importantes:** Deve **apenas relatar eventos que chegam durante o intervalo** (sem chegadas tardias, sem retrações).
* Tabela **Sink:** Delta Lake.

---

### Modos de saída em Streaming Estruturado:

* **Completar** → reescreve toda a tabela de resultados todos os gatilhos. Usado para agregações quando você quer a tabela inteira. Não é eficiente aqui.
* **Anexar** → apenas novas linhas (resultados finalizados) são gravadas na pia. ✅
* **Atualização** → apenas linhas que mudaram desde o último gatilho são escritas (boas para agregações com dados tardios).

---

Uma vez que o requisito diz:

* “**apenas relatar eventos que chegam durante o intervalo**” → significa contagens finalizadas por janela, não atualizações.
* Isso corresponde ao **Modo de anexo**, porque os resultados são anexados quando a janela fecha.

---

👉 resposta correta:
**B. append**

Vamos verificar cuidadosamente os requisitos:

* Os bancos de dados estão atualmente em **SQL Server 2019 (SQL1)**.
* Deve ser capaz de **Comunicar-se com SQL2 (on-premises)** usando **conexões de servidor vinculadas**.
* Deve **minimizar o esforço administrativo**.

---

### Alternativas:

**A. Azure SQL Database** ❌

* Serviço PaaS, mas faz **não suporta servidores vinculados**.

**B. Azure SQL Database elastic pool** ❌

* As mesmas limitações que o único Azure SQL DB. Nenhum servidor vinculado.

**C. SQL Server on Azure Virtual Machines** ✅ (suporta servidores vinculados, uma vez que é completo SQL Server).

* Mas o → requer **esforço administrativo elevado** (patching, backups, HA, etc.).

**D. Azure SQL Managed Instance** ✅

* Serviço PaaS com **perto de 100% SQL Server compatibilidade**.
* **Suporta servidores vinculados** (a on-premises SQL, via private endpoint / VPN / ExpressRoute).
* **Minimiza o esforço administrativo** comparado com a execução do SQL no VMs.

---

👉 resposta correta:
**D. Azure SQL Managed Instance**


Estamos projetando um **schema estrela** para análise.
O extrato inclui **dimensões (atributos descritivos)** e **Factos (valores mensuráveis)**.

---

### Colunas dadas:

* **EventCategory** → e.g., "Vídeos" → **dimensão**
* **EventAction** → e.g., "Jogar" → **dimensão**
* **EventLabel** → e.g., "Contoso Promocional" → **dimensão**
* **ChannelGrouping** → e.g., "Social" → **dimensão**
* **TotalEvents** → numeral → **facto**
* **SessionsWithEvent** → numeral → **facto**
* **UsersWithEvent** → numeral → **facto**

---

### Mapeamento do esquema estelar:

1. **EventCategory** → vai para **DimEvent**
   (É descritivo sobre o tipo de evento.)

2. **ChannelGrouping** → vai para **DimChannel**
   (Ele descreve o canal, então pertence à dimensão do canal.)

3. **TotalEvents** → vai para **FactEvents**
   (É uma medida / métrica, pertence de fato tabela.)

---

### Resposta final:

* **EventCategory → DimEvent**
* **ChannelGrouping → DimChannel**
* **TotalEvents → FactEvents**


Vamos analisar este passo a passo:

* **Cenário:**

  * Azure Synapse Analytics (DW1) com **cargas automatizadas de dados**.
  * Ao mesmo tempo, **usuários executar consultas ad-hoc**.
  * Requisito: **garantir que as cargas automatizadas tenham memória suficiente para terminar rapidamente** apesar da concorrência.

---

### Alternativas:

**A. Atribuir uma classe de recurso menor às consultas automatizadas de carga de dados** ❌

* Smaller resource class = less memory → oposto ao que é necessário.

**B. Criar estatísticas amostradas para cada coluna** ❌

* Estatísticas melhoram os planos de consulta, mas não garantem a disponibilidade de memória para cargas.

**C. Atribuir uma classe de recurso maior às consultas automatizadas de carga de dados** ✅

* Classes de recursos em Synapse determinam quanta memória é alocada para uma consulta.
* Larger resource class = more memory → melhor para **desempenho da carga de dados**.

**D. Distribua as grandes tabelas de fatos antes de executar as cargas de dados automatizadas** ❌

* A distribuição de hash ajuda a consultar o desempenho e reduz o movimento dos dados, mas o **E** resolve o problema de disponibilidade de memória para o processo de carregamento.

---

👉 resposta correta:
**C. Atribua uma classe de recursos maior às consultas automatizadas de carga de dados.**


Precisamos criptografar o **Azure Data Factory v2 (DF1)** usando um **chave gerenciada pelo cliente (CMK)** do **Azure Key Vault (vault1)**.

---

### Factos importantes:

* Para usar **encriptação CMK** com ADF, o **Key Vault** deve ter:

  * **Soft-delete habilitado**
  * **Proteção contra purgas ativada**

Este é um **exigência dura** do Azure porque o ADF deve ser capaz de recuperar a chave se for apagado acidentalmente.

---

### Alternativas:

**A. Desativar a proteção de purga em vault1** ❌

* Errado, a proteção de purga deve ser **activado**, não desabilitada.

**B. Remover o serviço vinculado do df1** ❌

* Não é necessário antes de ativar a criptografia CMK.

**C. Criar um tempo de execução de integração auto-hospedado** ❌

* Não está relacionado com encriptação com CMK.

**D. Desativar delete suave em vault1** ❌

* Errado, **O soft delete deve ser ativado**, não desativado.

---

👉 O primeiro passo é, na verdade, o **oposta** de A e D:
Você deve **permitir a proteção de soft-delete e purga** no Cofre Chave.

Uma vez que nenhuma das respostas listadas diz **permitir**, e as opções de truque são sobre desativá-los, a interpretação correta é:

✅ **Nenhuma dessas ações incapacitantes está correta. O passo necessário é para ENABLE soft-delete e expurgar a proteção em vault1.**

---


Queremos um **alerta sobre o uso de CPU** para **Azure SQL databases** no mesmo servidor, incluindo **bancos de dados futuros**.

---

### Alternativas:

**A. Grupos de Recursos** ❌

* Alertas no escopo do grupo de recursos não rastreiam automaticamente métricas de nível de banco de dados, como o uso do CPU.

**B. Servidores SQL** ✅

* Se você definir o alerta no **Recurso SQL Server**, ele se aplica a todas Os bancos de dados sob esse servidor, incluindo **Novos criados mais tarde**.

**C. Bancos de Dados SQL** ❌

* Funciona apenas por banco de dados individual. Exigiria configurar alertas um a um e não se aplicaria automaticamente a novos bancos de dados.

**D. Máquinas Virtuais SQL** ❌

* Não é relevante aqui – estamos usando **Azure SQL Database PaaS**, não SQL Server em VMs.

---

👉 resposta correta:
**B. Servidores SQL**

Precisamos de **Notificações de voz** quando **eventos de manutenção afetam qualquer uma das 10 regiões** onde o nosso **Instâncias gerenciadas Azure SQL** correr.

---

### Alternativas:

**A. A partir do Azure portal, crie um alerta de saúde de serviço** ✅

* Alertas de saúde do serviço notificá-lo sobre **problemas em serviços ou regiões do Azure** (incluindo manutenção planejada).
* Pode entregar alertas via **e-mail, SMS ou chamada de voz**.
* **uma vez por assinatura/região** configurado, de modo que isso minimiza o esforço administrativo em todas as instâncias gerenciadas.

**B. Criar um alerta de excelência operacional Azure Advisor** ❌

* O consultor dá recomendações de melhores práticas (custo, desempenho, segurança), e não eventos de manutenção/saúde de serviço.

**C. Configurar um SQL Server Agent job em SSMS** ❌

* Local para um único managed instance, não se integra com notificações de eventos de manutenção em todo o Azure.

**D. Configurar um alerta de log de atividade** ❌

* O log de atividades captura eventos **dentro de seus recursos de assinatura** (e.g., modificações de recursos), não **eventos de saúde do serviço em nível regional**.

---

👉 resposta correta:
**A. A partir do Azure portal, crie um alerta de saúde de serviço**


Vamos analisar este passo a passo:

### Requisitos:

1. **Failovers que não exigem aplicativos de clientes para alterar suas cadeias de conexão**
   → Isso exclui **geo-replication**, porque geo-replication requer **mudanças manuais connection string** durante failover.

2. **Replicar o banco de dados para uma região secundária do Azure**
   → Isso significa que o suporte entre regiões é necessário (não apenas Availability Zones, que estão dentro da mesma região).

3. **Suporte failover para a região secundária**
   → deve apoiar **automático ou manual failover em regiões**.

---

### Alternativas:

* **A. Failover groups** ✅

  * Fornece **automatic failover** entre **servidores primários e secundários** entre regiões.
  * Os aplicativos podem se conectar usando um **ponto final listener**, portanto, não são necessárias alterações no connection string.
  * Cumpre todos os requisitos.

* **B. Transactional replication** ❌

  * Principalmente para replicação, não alta disponibilidade ou failover transparente.
  * Requer mudanças de conexão e não fornece auto-failover.

* **C. Availability Zones** ❌

  * Aumenta a disponibilidade dentro do **mesma região**, não entre regiões.
  * Não atende ao requisito de replicação para região secundária.

* **D. Geo-replication** ❌

  * Fornece legível secundário, mas **failover requer reconfiguração manual** de cordas de conexão.
  * Não atende ao requisito.

---

✅ **Resposta correta: A. failover groups**


Grande pergunta 👍 Vamos dividi-lo.

Você quer ver **os valores dos parâmetros da última vez que uma consulta foi executada** em **SQL Server correndo em uma Azure VM**.

---

### Características 🔎 Key SQL Server envolvidas:

1. **Perfil de Consulta Leve**

   * Introduzido para reduzir a sobrecarga de perfil de consulta.
   * Permite capturar estatísticas de execução em tempo de execução (e.g., contagem de linhas, desempenho do operador).
   * Necessário para obter **estatísticas de execução** em DMVs.

2. **Last\_Query\_Plano\_Estatísticas** (Configuração da Banco de dados Escopo)

   * Permite que você acesse o **última actual execution plan** para uma consulta (não apenas o plano estimado).
   * Inclui **valores dos parâmetros de tempo de execução** da última execução.

3. **PARAMETER \_ SNIFFING**

   * Controla se o SQL Server armazena em cache planos de consulta com base no primeiro conjunto de parâmetros.
   * Não relacionado à recuperação de valores de parâmetros da última consulta.

---

### ✅ Respostas corretas:

* **B. Habilite a definição de perfil leve \_ Query \_ em DB1**
  (Necessário coletar informações de perfil de consulta com sobrecarga mínima.)

* **C. Ativar última consulta \_ Plano \_ Estatísticas \_ em DB1**
  (Permite recuperar o último actual execution plan, incluindo valores de parâmetros.)

---

### ❌ Opções incorretas:

* **A & D (Ativar no mestre)** → Escopo errado. Estes são **configurações de escopo de banco de dados**, não definido como mestre.
* **(Ativar PARAMETER \_ SNIFFING)** → Controles plano de cache, não recuperação de valor do parâmetro.

---

✅ Resposta Final:
**B. Habilite a definição de perfil leve \_ Query \_ em DB1**
**C. Ativar última consulta \_ Plano \_ Estatísticas \_ em DB1**

---


Muito bem, vamos analisar cuidadosamente este ✅

---

### Requisitos:

1. **Suporte iniciado pelo usuário backups**

   * Necessário para flexibilidade (restaurar quando quiser, não apenas retenção automatizada).
   * ✅ Disponível em **SQL Server em Azure VM** (IaaS) → você controla SQL Server Agent jobs, backups a Blob Storage, etc.
   * ❌ Azure SQL Database / Managed Instance → backups são automatizados pelo Azure (você pode restaurar, mas não *Iniciar* backups diretamente).

2. **Suporte a múltiplas instâncias replicadas automaticamente em todas as regiões do Azure**

   * Significa geo-redundância / alta disponibilidade.

3. **Minimizar o esforço administrativo**

   * Regras para fora de gestão pesada como Always On AG configuração em SQL VMs.

---

### Opção de Implantação:

👉 **SQL Server on Azure Virtual Machines**

* Porque somente em IaaS SQL VM você pode executar **Iniciado pelo usuário backups**.
* PaaS (Azure SQL Database / Managed Instance) = automatic backups Apenas, não iniciado pelo usuário.

---

### Opção de resiliência:

👉 **Auto-failover group**

* Fornece **replicação automática entre regiões** + automatic failover com **mudança connection string**.
* Active geo-replication = database-level only (manual failover).
* Um zone-redundant deployment cobre zonas da mesma região, sem cruzar regiões.

---

### ✅ resposta correta:

* **Opção de implantação**: **SQL Server on Azure Virtual Machines**
* **Opção de resiliência**: **Auto-failover group**


Vamos analisar esta questão em detalhes 👇

---

### **NEW QUESTION 49 (Tópico de Exame 5)**

Você tem um Azure SQL Database.
Está a rever um **consulta de execução lenta**. A imagem mostra um plano de consulta com os operadores.

---

### **Passo 1 – Identifique o que a exposição mostra**

* Na captura de tela do plano de consulta, o cabeçalho indica "Query cost related to the batch: 100%".
* As caixas mostram operadores de execução de consulta com porcentagens.
* Este é um **plano de execução visual** que o SQL Server produz quando você executa consultas.
* Uma vez que mostra **Estimativa vs detalhes reais de execução** (com percentagens), este é um **actual execution plan**.

👉 **Resposta:** **actual execution plan**

---

### **Passo 2 – Identifique o operador no plano**

Olhando para a exposição:

* O operador inferior é rotulado **Digitalização Clustered Index**.
* Mas a pergunta pede o **operador no plano de execução que indica que a consulta se beneficiaria do ajuste**.
* O operador que causa problemas de desempenho aqui é o **Key Lookup (Clustered)** (uma vez que as principais pesquisas acessam repetidamente linhas de um clustered index – caro para desempenho).

👉 **Resposta:** **Key Lookup**

---

### ✅ Resposta Final:

1. A exposição mostra → **actual execution plan**
2. O operador no plano de execução que indica que a consulta se beneficiaria do ajuste de desempenho → **Key Lookup**

---

Dica ⚡ Dica: Nos exames SQL, sempre que você vê **Key Lookup** ou **Index Scan** com alto custo %, geralmente significa perder índices → de ajuste de desempenho necessário.

Boa pergunta 👇

Precisamos de detecção de anomalias para **streaming de dados do hub IoT** com **esforço mínimo de desenvolvimento**. Vamos avaliar:

---

### **Requisitos**

* **Enviar saída para Azure Synapse** → deve apoiar a integração da pia Synapse.
* Detecção de anomalias **Identificar picos e mergulhos em séries temporais** → em dados de streaming em tempo real.
* **Minimizar o esforço de desenvolvimento/configuração** → preferem serverless, características embutidas sobre codificação pesada.

---

### **Análise de Opção**

**A. Azure SQL Database**

* Não é adequado para detecção de anomalias em dados de streaming.
* É mais para armazenamento, não análise em tempo real.
  ❌

**B. Azure Databricks**

* Poderoso para aprendizado de máquina e análise avançada.
* Mas requer **modelos personalizados ML**, codificação e mais configuração.
* Não minimiza o esforço.
  ❌

**C. Azure Stream Analytics**

* Tem **Funções de detecção de anomalias incorporadas** (e.g., `AnomalyDetection_SpikeAndDip`).
* Nativamente integra-se com **Hub de IoT do Azure (entrada)** e **Azure Synapse Analytics (pia de saída)**.
* **Serverless, configuração mínima**, exatamente se encaixa requisitos.
  ✅

---

### ✅ resposta correta:

**C. Azure Stream Analytics**

---

Great one 👇 Trata-se de garantir a sua **O Azure Stream Analytics job tem unidades de streaming suficientes (SUs)** provisioned.

---

### **Métrica chave: SU % Utilização**

* Isso mostra quanto da sua capacidade de unidade de streaming alocada está sendo usada.
* Se estiver perto de 100%, talvez seja necessário aumentar a escala.

Mas monitorar a utilização do SU por si só não é suficiente. Você precisa verificar se o job está acompanhando o fluxo de dados de entrada.

---

### **Análise de Opções**

**A. Eventos de entrada tardia**

* Estes ocorrem quando os eventos chegam mais tarde do que o *Tolerância de chegada tardia* configurado.
* Eles são sobre qualidade de dados, não capacidade SU.
  ❌

**B. Fora de ordem Eventos**

* Estes são eventos que chegam fora de sequência.
* Também sobre a ordenação de eventos, não sobre se o SUs é suficiente.
  ❌

**C. Eventos de Entrada Backlogged**

* Muito importante.
* Mostra quantos eventos de entrada o job recebeu, mas ainda não foi processado.
* Se o backlog crescer → SUs são insuficientes.
  ✅

**D. Marca d'Água Atraso**

* Indica até que ponto o sistema está atrasado no processamento de eventos, medidos no tempo.
* Se o atraso da marca d'água estiver aumentando, → job está abaixo de provisioned.
  ✅

**E. Função Eventos**

* Refere-se a eventos enviados para Azure Functions (processamento personalizado).
* Não é relevante para o dimensionamento SU.
  ❌

---

### ✅ Respostas corretas:

**C. Eventos de Entrada Backlogged**
**D. Marca d'Água Atraso**

---


### **Pergunta Recap**

* DB: **Azure SQL database (db1)**
* Ferramenta de monitoramento: **Query Performance Insight (QPI)** → depende de **Query Store**.
* Requisito: **Os dados de monitoramento de desempenho devem estar disponíveis o mais rápido possível**.
* Tarefa: Escolha **configuração de configuração** e **Valor**.

---

### **Entendendo configurações Query Store**

1. **DATA \_ FLUSH \_ INTERVAL \_ SECONDS**

   * Controla a frequência com que os dados coletados são gravados (“flushed”) da memória para o disco.
   * **Default = 900s (15 min)**.
   * Reduzir este valor (e.g., 60s ou 1s) torna os dados disponíveis mais rapidamente em QPI.

2. **INTERVAL \_ LENGTH \_ MINUTOS**

   * Define a granularidade temporal da agregação de dados (padrão = 60 min).
   * Intervalos menores (e.g., 1 min) melhoram a granularidade, mas não afetam diretamente o quão cedo os dados estão disponíveis.

3. **MAX \_ PLANS \_ POR \_ QUERY**

   * Controla quantos planos de execução são armazenados → irrelevantes para pontualidade.

4. **QUERY \_ CAPTURA \_ MODO**

   * Controles quais consultas são capturadas (ALL, AUTO, CUSTOM).
   * Impactos **O que é capturado**, não **quando disponível**.

---

### **Resposta correta**

Para fazer **dados de monitoramento de desempenho disponíveis ASAP**, você deve:

* Configurar **DATA \_ FLUSH \_ INTERVAL \_ SECONDS**
* Set para **1 (segundo)**

---

✅ **Resposta final:**
**Configuração Configuração:** `DATA_FLUSH_INTERVAL_SECONDS`
**Valor:** `1`

---

### Estrutura de comando:

```sql
RESTORE DATABASE MyDB1
FROM DISK = 'D:\DB1.bak'
WITH { option }
GO
```

---

### Alternativas:

1. **NORECOVERY**

   * Deixa o banco de dados **não-operacional**.
   * Usado quando você planeja aplicar **transaction log backups adicional**.
   * Mantém o banco de dados em um estado de "restauração".
   * Exemplo:

     ```sql
     RESTORE DATABASE MyDB1 
     FROM DISK = 'D:\DB1.bak' 
     WITH NORECOVERY;
     ```

2. **RECOVERY**

   * Faz o banco de dados **operacional** imediatamente após a restauração.
   * Você não pode restaurar mais backups depois disso.
   * Exemplo:

     ```sql
     RESTORE DATABASE MyDB1 
     FROM DISK = 'D:\DB1.bak' 
     WITH RECOVERY;
     ```

3. **STANDBY**

   * Faz o banco de dados **read-only** após a restauração.
   * Useful for log shipping (o banco de dados secundário é legível enquanto espera pela próxima restauração do log).
   * Exemplo:

     ```sql
     RESTORE DATABASE MyDB1 
     FROM DISK = 'D:\DB1.bak' 
     WITH STANDBY = 'D:\UNDO\DB1.undo';
     ```

---

✅ **Em suma:**

* Use **NORECOVERY** se mais restaurações estiverem chegando (como log backups).
* Use **RECOVERY** se esta for a restauração final e você quiser o DB online.
* Use **STANDBY** se você quiser uma cópia read-only (comum em log shipping).


### Ponto-chave na pergunta:

* Os clientes precisam ser capazes de **criar objetos de banco de dados**.
* A solução deve atender **objetivos de negócios** → geralmente significa **isolamento** entre os clientes, evitando conflitos.

---

### Funções:

* **`ddl_admin`**: Permite que um usuário execute comandos **Linguagem de Definição de Dados (DDL)**, como `CREATE`, `ALTER`, `DROP`.
* **`db_writer`**: Permite somente `INSERT`, `UPDATE`, dados `DELETE` → **não pode criar objetos de esquema**.

---

### Alternativas:

**A. `ddl_admin` no esquema existente**

* Problema: Todos os clientes compartilham o mesmo esquema → conflitos, sem isolamento. ❌

**B. Criar esquema adicional por cliente e conceder `ddl_admin`**

* ✅ Melhor opção.
* Cada cliente recebe seu próprio esquema, e o `ddl_admin` nesse esquema permite que eles criem suas próprias tabelas, procedimentos, etc.
* Previne conflitos, alinha-se com o isolamento multilocatário.

**C. Criar esquema adicional e conceder `db_writer`**

* ❌ `db_writer` só permite DML, não mudanças de esquema.

**D. `db_writer` no esquema existente**

* ❌ O mesmo que acima, além de esquema compartilhado (ruim para isolamento).

---

✅ **Resposta correta: B. Para cada cliente, crie um esquema adicional e conceda ao cliente ddl \_ admin o novo esquema.**

---

### Cenário:

* Você precisa implementar **segurança** para **ResearchDB1**.
* Requisito → deve atender **segurança e conformidade**.
* Claramente, isso é sobre **Always Encrypted** com integração **Azure Key Vault**.

---

### Passos (na sequência correta):

1. **Criar um Azure AD managed identity**

   * Necessário para que ResearchApp1 possa autenticar com segurança sem armazenar credenciais.

2. **Criar uma instância Azure Key Vault e configurar uma política de acesso**

   * Key Vault armazena chaves de criptografia.
   * A política de acesso garante que o managed identity tenha permissão para usar as chaves.

3. **Execute o assistente Always Encrypted**

   * Este assistente configura a criptografia de coluna e aponta para o Key Vault para gerenciamento de chaves.

---

### Resposta:

✅ Sequência correta é:

1. **Criar um Azure AD managed identity**
2. **Criar uma instância Azure Key Vault e configurar uma política de acesso**
3. **Execute o assistente Always Encrypted**

---

Dica ⚡ Pro:
A opção **Crie uma instância Azure Key Vault e gere um segredo** é uma distração. Para Always Encrypted, você armazena **chaves**, não segredos.

---


Boa pergunta 👌 Vamos analisá-la.

---

### Cenário:

* Você tem **e Azure SQL Database servidor** (`sqlsrv1`) com 10 bancos de dados.
* O desempenho é mais lento do que o esperado.
* Você quer especificamente verificar **Se o problema estiver relacionado ao `tempdb`**.

---

### Alternativas:

**A. Executar consultas baseadas em Query Store**

* Query Store ajuda com **desempenho da consulta** ao longo do tempo (planos de execução, regressões).
* Não sobre `tempdb`. ❌

**B. Informação de revisão fornecida por SQL Server Profiler-based traces**

* O SQL Profiler não é compatível com Azure SQL Database. ❌

**C. Informação da revisão fornecida por Query Performance Insight**

* Dá insights sobre **consultas de consumo superior** (DTU, CPU, IO, memória).
* Não ajuda especificamente a diagnosticar o uso do `tempdb`. ❌

**D. Executar consultas baseadas na visão de gestão dinâmica (DMV)**

* DMVs em Azure SQL Database pode mostrar o uso de recursos, esperas e especificamente **tempdb contenção** (e.g., `sys.dm_db_resource_stats`, `sys.dm_exec_requests`, `sys.dm_db_session_space_usage`).
* Esta é a maneira correta de verificar os gargalos `tempdb`. ✅

---

### ✅ resposta correta:

**D. Executar consultas baseadas em visualização de gerenciamento dinâmico**

---

Grande pergunta 👍 Vamos dividi-lo claramente:

---

### **Por que “Layout simples (strip)” é a escolha correta aqui**

* Você tem **dois discos P30**, cada um fornece **5,000 IOPS**.
* Requisito: **10,000 IOPS**.
* O **layout simples** (strip) combina a taxa de transferência de ambos os discos e IOPS, dando-lhe **Desempenho agregado** (5,000 + 5,000 = 10,000 IOPS).
* Isso atende ao requisito.

---

### **Outros layouts e quando são usados**

1. **Layout do espelho**

   * As gravações são duplicadas entre discos.
   * Você não se combina IOPS; em vez disso, você obtém **redundância**.
   * Exemplo: Se você tivesse dois discos com 5,000 IOPS cada, o IOPS efetivo permanece 5,000.
   * Caso de uso: **Alta disponibilidade e redundância** (à custa do desempenho/espaço).

2. **layout paridade**

   * Dados + bits de paridade são listrados em todos os discos.
   * Fornece tolerância a falhas com melhor capacidade do que o espelho, mas **Escreve mais lento**.
   * Caso de uso: **Arquivamento ou cargas de trabalho que precisam de equilíbrio entre capacidade e resiliência**.
   * (Não mostrado em suas opções, mas é bom saber.)

3. **Layout simples (striping)**

   * Os dados são listrados através dos discos, **sem redundância**.
   * Todos os discos contribuem para o desempenho (IOPS + taxa de transferência).
   * Caso de uso: **Cargas de trabalho de alto desempenho onde a redundância não é crítica** (e.g., tempdb, cargas de trabalho analíticas não críticas, ou quando a redundância é fornecida em outro lugar).

4. **Piscina de armazenamento**

   * Agrupamento lógico de múltiplos discos físicos.
   * Fundação passo antes de criar discos virtuais com layouts.
   * Caso de uso: Sempre o **primeiro passo** ao usar espaços de armazenamento no Windows / Azure VM.

---

✅ No seu cenário:

* Desempenho (10,000 IOPS) é o **requisito primário**.
* A redundância é **não mencionado**.
* O → **Piscina de armazenamento → Layout simples → Volume** está correto.

---


Correct ✅

A resposta é: **B. Diminuição pela metade do valor do intervalo de fluxo de dados**

### Explicação:

* O Query Store coleta dados de desempenho da consulta na memória primeiro, depois periodicamente os libera no disco (o banco de dados).
* Se o **Dados Flush Intervalo** for muito longo, o Query Store pode acumular muitos dados na memória. Se o SQL Server ficar sem espaço para o Query Store, ele pode fazer a transição para o **modo read-only** para evitar mais crescimento.
* Por **Diminuição do intervalo de fluxo de dados**, os dados são gravados no disco com mais frequência, reduzindo a pressão da memória e minimizando o risco de Query Store se tornar read-only.

Outras opções:

* **A. Intervalo Duplo Dados Flush** → Aumenta o risco (mantém mais dados na memória).
* **C & D. Estatísticas Coleção Intervalo** → Controla com que frequência as estatísticas de tempo de execução são agregadas, não relacionadas às transições Query Store read-only.

👉 Então a melhor solução é **B**.

Perfeito ✅ vamos quebrar este passo a passo usando a política na captura de tela.

### **Política de Retenção em Screenshot**

* **Semana LTR Backups** → Retido para **6 semanas**.
* **Mensal LTR Backups** → Retido para **12 meses** (o **Semana 2 backup** de cada mês).
* **Anual LTR Backups** → Retido para **10 anos** (o **Semana 2 backup de Janeiro** a cada ano).

---

### **Exemplo: Janeiro 2020***

1. **Jan 4, 2020** → Semana 1 de Jan

   * Armazenado como **semanal backup**.
   * Retido apenas **6 semanas**.
   * ❌ Não mensalmente, porque a política diz Semana 2.

2. **Jan 11, 2020** → Semana 2 de janeiro

   * Armazenado como **semanal backup** → retido **6 semanas**.
   * ✅ Promovido a **mensal backup** → retido **12 meses**.
   * ✅ Também promovido a **anual backup** (já que é a Semana 2 backup de janeiro) → reteve **10 anos**.

---

### **fevereiro 2020**

* **Feb 1, 2020** → Semana 1 → semanalmente apenas → 6 semanas.
* **Feb 8, 2020** → Semana 2 → semanalmente + promovido a mensal → retido **12 meses**.

(sem anual em fevereiro, apenas janeiro fornece anualmente).

---

### **Resumo**

* **Semana backups**: Toda semana, mantido **6 semanas**.
* **Mensal backups**: O **Semana 2 backup** de cada mês, mantido **12 meses**.
* **Anual backups**: O **Semana 2 backup de Janeiro**, mantido **10 anos**.

---

👉 Assim, a razão **Jan 4, 2020** é **E** por mês backup é simplesmente porque o **A regra mensal é configurada para a semana 2**, não a Semana 1.


Você tem isso 👍

Para **Acionando um pipeline Azure Data Factory quando um arquivo chega ao Azure Data Lake Storage Gen2**, o serviço correto é **Event Grid**.

* **Microsoft.EventHub** → Usado para transmitir grandes volumes de eventos (ingestão em tempo real).
* **Microsoft.Sql** → SQL fornecedor de recursos (não relevante para gatilhos de arquivo).
* **Microsoft.Automation** → Usado para automação runbooks, não gatilhos orientados a eventos.
* ✅ **Microsoft.EventGrid** → Correcto. Event Grid integra-se com **ADLS Gen2** e pode levantar um evento quando um novo arquivo é criado, o que pode desencadear a **gasoduto ADF**.

**Resposta final: B. Microsoft.EventGrid** ✔️

Great question — this comes up a lot in Synapse design ✅

Você está criando um **tabela de dimensão** em um **dedicated SQL pool** e precisa de um **chave de substituição** com **desempenho rápido da consulta**.

Vamos analisar:

* **A. coluna IDENTITY**

  * Comum em sistemas OLTP (SQL Server).
  * Mas na dedicated SQL pool à Synapse, os valores `IDENTITY` são gerados por distribuição, o que pode levar ao **valores duplicados entre distribuições**, a menos que seja cuidadosamente gerenciado. Não é o melhor para escalabilidade ou desempenho no MPP (Processamento Massivo Paralelo).

* **B. coluna GUID**

  * Globalmente único, mas **grandes (16 bytes)** e não sequencial.
  * Leva à fragmentação e **desempenho de consulta mais lento**. Má escolha para chaves de substituição em Synapse.

* **C. um objeto de sequência**

  * ✅ Recomendado em Synapse.
  * Centralizado e gera valores únicos entre distribuições.
  * Prover **Chaves de substituição escaláveis, rápidas e sequenciais** Evita as armadilhas de `IDENTITY`.

📌 **Resposta correta: C. um objeto de sequência**

Great question 👍 Vamos explicar por que o **outras opções** não estão corretos para este requisito específico e quando você *Iria* usá-los:

---

### **Opção A. Anexar dois espelhos 4- TB SSDs (RAID-1 / espelhamento)**

* **Por que não aqui?**

  * O espelhamento corta a capacidade utilizável pela metade (4 TB utilizável a partir de 8 TB cru).
  * IOPS faz **não escala** muito em comparação com o striping.
  * O objetivo aqui é **máximo IOPS**, não redundância.
* **Quando usar:**

  * Se o seu principal requisito é **alta disponibilidade** ou **redundância**, e.g., protegendo contra uma única falha no disco.
  * Não é ideal para o Azure Premium SSDs, uma vez que o Azure já fornece redundância no nível da infraestrutura.

---

### **Opção C. Anexar uma matriz RAID-5 com cinco 1- TB SSDs**

* **Por que não aqui?**

  * RAID-5 tem **escrever penalidades** (cálculo de paridade) → não é bom para cargas de trabalho SQL Server com alta atividade de gravação.
  * Configuração mais complexa e desempenho mais lento em comparação com o striping (RAID-0).
* **Quando usar:**

  * Apropriado quando você quer um **equilíbrio entre capacidade e redundância** e cargas de trabalho são **lido-pesado** (mas raramente usado para SQL Server no Azure devido a trade-offs de desempenho).

---

### **Opção D. Anexar um único 4- TB SSD**

* **Por que não aqui?**

  * O IOPS e a taxa de transferência de um único disco são limitados pelos limites do Azure para esse tamanho de disco.
  * Mesmo o maior Premium SSD não pode chegar ao IOPS que você recebe por **striping múltiplo menor SSDs**.
* **Quando usar:**

  * Se sua exigência é **simplicidade** (fácil de gerenciar, um disco).
  * Se a carga de trabalho IOPS precisar se encaixar dentro do limite de disco único.

---

### ✅ **Escolha correta: Opção B. Stripe conjunto com quatro 1- TB SSDs (RAID-0 / Espaços de armazenamento)**

* Premium SSDs em Azure **escala IOPS linearmente quando listrado**.
* O stripping fornece o **maior rendimento e IOPS**.
* Quando o requisito é **desempenho** sobre redundância.

---

⚖️ **Resumo:**

* Use **Striping (RAID-0)** → para máximo IOPS /throughput.
* Use **Espelho (RAID-1)** → para redundância / alta disponibilidade.
* Use **RAID-5** → para eficiência de capacidade + alguma tolerância a falhas (mas raramente no SQL Server Azure devido ao desempenho de escrita ruim).
* Use **Grande SSD** → para simplificar quando as necessidades do IOPS são moderadas.

---

Great question 👍 Vamos detalhar quando cada opção (distribuição + índice) pode ser usada:

---

### **Tipos de distribuição**

1. **Hash**
   ✅ Melhor para: **Tabelas Fact** (tabelas grandes, número alto de linhas).

   * Use quando você pode escolher um **coluna de alta cardinalidade** como chave de distribuição (como CustomerID, OrderID).
   * Otimiza associações e agregações.

2. **robin redondo**
   ✅ Melhor para: **Tabelas de estágio** ou quando não existe uma boa chave de distribuição.

   * Os dados são distribuídos uniformemente, mas não com base em uma coluna → pode levar a mais movimento de dados durante as junções.
   * Exemplo: Aterrar dados brutos antes das transformações.

3. **Replicado**
   ✅ Melhor para: **Tabelas de pequena dimensão** (usually < 2 GB comprimida).

   * Uma cópia completa da tabela é armazenada em todos os nós de computação.
   * Elimina o movimento de dados em associações entre tabelas de fato e dimensão.
   * Exemplo: Dimensão da data, país lookup.

---

### **Tipos de índice**

1. **Clustered columnstore index (CCI)**
   ✅ Melhor para: **Tabelas de fatos em análise/ OLAP**.

   * Fornece alta compressão + varreduras muito rápidas para agregados.
   * Default for large fact tables in Rio de Janeiro.

2. **Clustered index**
   ✅ Melhor para: **Tabelas pequenas ou cargas de trabalho semelhantes a OLTP**.

   * Se você precisar de pesquisas de pontos rápidos (e.g., obter 1 registro por chave).
   * Sometimes used on small dimension tables instead of CCI (para evitar sobrecarga de compressão).

3. **Nonclustered index**
   ✅ Melhor para: **Melhorar padrões de consulta específicos**.

   * Exemplo: Quando você precisa otimizar a filtragem em algumas colunas.
   * Geralmente aplicado em cima de CCI para consultas direcionadas (mas use com moderação para evitar sobrecarga de carga).

---

✅ **Regra de resumo do polegar:**

* **Tabelas de fatos (grandes, >1M linhas):** Hash + CCI.
* **Small dimension tables (<2GB):** Replicado + Clustered ou CCI.
* **Tabelas de estagiamento:** Rodada robin + Clustered index.
* **Consultas especiais:** Adicione Nonclustered index conforme necessário.

---

Vamos quebrar este aqui com cuidado:

---

### Requisitos ✅:

* **App1 autentica com SQL authentication** → significa nome de usuário / senha dentro DB, não Azure AD ou Windows login.
* **App1 só deve ver DB2** → não deve ver DB1.
* **Esfor mínimo administrativo** → solução mais simples e direta.

---

### Opções 🔎:

**A. contained database user para App1 em DB2**

* A contained database user E **criado apenas dentro do banco de dados** (DB2).
* Ele não requer um servidor de nível login em `Server1`.
* Isso significa que App1 só vê **DB2** e não DB1.
* ✅ Isso atende **todos os requisitos**.

---

**B. login para App1 em Server1**

* Um servidor de nível login existiria em `Server1`.
* Por padrão, ele tem visibilidade para o servidor e pode ser mapeado para DB1 e DB2.
* ❌ Mais admin overhead, não restringe a visibilidade ao DB2 apenas sem etapas extras.

---

**C. a contained database user de um fornecedor externo para App1 E-mail DB2**

* "External provider" = Azure AD autenticação.
* ❌ Não se aplica aqui, uma vez que App1 está usando **SQL authentication**, não Azure AD.

---

**D. contained database user de um Windows login para App1 em DB2**

* Isto é Windows authentication.
* ❌ Não é válido, uma vez que App1 deve autenticar via **SQL authentication**.

---

### ✅ resposta correta:

**A. contained database user para App1 em DB2**

---


Perfeito, vamos analisar este um 👇

O plano de execução da consulta mostra um operador **Key Lookup (Clustered)**.

### Discriminação:

* **Key Lookup** acontece quando o SQL Server encontra uma linha em um **nonclustered index**, mas precisa de mais colunas não presentes no índice, então ele volta para o **clustered index (ou pilha)** para buscá-las.
* Isso causa E/S extra e retarda o desempenho.
* A correção é adicionar essas colunas ausentes no **nonclustered index** como **colunas INCLUDE**, então o SQL Server não precisa fazer o lookup.

### Resposta correta:

* Operador **O problema de desempenho decorre do:** 👉 **Key Lookup**
* **O problema de desempenho pode ser resolvido adicionando colunas include ao:** 👉 **nonclustered index**

✅ Resposta Final:

* **Key Lookup**
* **nonclustered index**

Vamos resolver esta questão passo a passo 👇

### Cenário:

* **DB1** = Azure SQL Database (Hyperscale nível, sem réplicas secundárias ainda).
* **App1** = Azure Web Apps (multiple instances, read-only acesso a DB1).
* Requisito: Crie um **read-only replica** de DB1 e configure o **App1** para usá-lo.

---

### Conceitos-chave:

1. **Nível Hyperscale** suporta **réplicas em escala de leitura** (read-only replicas).

   * Você pode criar **1–4 chamado réplicas** no mesmo servidor lógico.
   * Essas réplicas podem ser usadas para **cargas de trabalho read-only**, como relatórios ou consultas.

2. Para direcionar aplicativos para réplicas:

   * Use o parâmetro **ApplicationIntent=ReadOnly** no connection string.
   * Isso garante que o aplicativo encaminhe consultas para a réplica, não para a primária.

---

### Passos corretos:

* **Adicionar read-only replicas de DB1** → ✅ *Crie uma réplica no mesmo servidor lógico.*
* **Para configurar instâncias App1 para acessar o read-only replica** → ✅ *Adicione uma entrada ApplicationIntent ao connection string.*

---

✅ **Resposta final:**

* **Criar uma réplica no mesmo servidor lógico**
* **Adicionar uma entrada ApplicationIntent ao connection string**

---

**Recomendação (tipos de distribuição dedicados da tabela da piscina SQL da Synapse):**

* **Dados da loja de varejo:** **Replicado** (pequena dimensão / tabela de referência → replicar a todas as distribuições para evitar o movimento de dados em junções).
* **Dados promocionais:** **Hash** (tabela maior juntou-se aos dados de fato → hash-distribuir na chave de junção para melhorar junções paralelas).

