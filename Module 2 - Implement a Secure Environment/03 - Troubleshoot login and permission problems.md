# Solucionar problemas de login e permissão

> **Habilidades cobradas:** Resolver problemas de autenticação e autorização
> **Tempo de leitura:** ~6 min

## Conceito central
Quando alguém não consegue se conectar ou não pode ler dados, verifique as coisas em ordem. É um soluço curto do Azure? A rede está bloqueada? O login existe? O usuário tem direitos? É como um carro que não vai começar: verifique o combustível, a bateria e a chave.

## Razões comuns para uma conexão falhar
1. **Eventos de reconfiguração.** Azure move ou atualiza seu banco de dados. Ele reinicia por um momento. Normalmente **menos de 60 segundos**.
2. **Firewall.** Nenhuma regra de firewall permite o IP do cliente (lição 5).
3. **Problemas Login.** Nome de usuário ou senha errados, o login E **Deficientes**Ou então **não existe**.
4. **Limites de recursos.** O banco de dados atingiu seus limites de memória, armazenamento ou conexão CPU.
5. **Tempo limite.** Redes lentas ou consultas de longa duração.

## Falhas transitórias
A **Falha transitória** é um erro curto, temporário. Ele se corrige. Isso acontece durante a manutenção planejada, escala, failoverou (raramente) problemas não planejados. O Azure se reconecta dentro de **60 segundos**.

Mensagens de erro transitórias típicas:
- "Não é possível abrir o banco de dados solicitado pelo login. O login falhou."
- "Não consigo processar o pedido. Não há recursos suficientes para processar o pedido."
- "Demasiadas operações em andamento para assinatura."

**Exemplo:** Um aplicativo de pedido de pizza mostra "login falhou" por 20 segundos em 3 AM, então funciona novamente. O Azure estava corrigindo o servidor. Esta é uma falha transitória, não uma senha quebrada.

### Tentar novamente a lógica
Os aplicativos devem **tentar novamente** silenciosamente, não mostrar o erro aos usuários. A Microsoft recomenda:
- Espere **5 segundos** antes da primeira tentativa.
- Use **backoff exponencial**: espere mais tempo cada vez, e.g. 5s → 10s → 20s → 40s, até **60s**.
- Defina um número **Máximo** de tentativas, para que nunca se repita para sempre.
- Se uma consulta falhar, **Feche a conexão e abra uma nova** para a nova tentativa. Não reutilize a conexão quebrada.

### O que verificar

| Problema | Ação |
|---------|--------|
| Falhas Login para muitos usuários | Check **Azure Service Health** para interrupções |
| Erros de limite de recursos | Relógio CPU, memória, armazenamento; **escala acima** se necessário |
| Falhas duradouras **ao longo de 60 segundos** | Contato **Suporte do Azure** |

## Fix " Login falhou para o usuário"
Trabalhe através dessas etapas.

1. **O login está desativado?**
   ```sql
   SELECT name, is_disabled FROM sys.sql_logins WHERE name = 'Ravi';
   ALTER LOGIN Ravi ENABLE;
   ```
2. **O login existe?** Se não, crie-o.
   ```sql
   CREATE LOGIN Ravi WITH PASSWORD = 'StrongP@ss1';
   ```
3. **Existe um usuário na banco de dados alvo?** Um login por si só não é suficiente.
   ```sql
   CREATE USER Ravi FOR LOGIN Ravi;
   ```
4. **O usuário tem permissões?** Adicionar a um papel ou conceder direitos.
   ```sql
   ALTER ROLE db_datareader ADD MEMBER Ravi;
   -- or
   GRANT SELECT ON SCHEMA::Sales TO Ravi;
   ```

**Exemplo:** Um novo analista pode fazer login no servidor, mas recebe um erro ao abrir o `SalesDB`. O login existe, mas ninguém criou um usuário no `SalesDB`. O passo 3 corrige isso.

Observação: Falha no **Microsoft Entra** logins do **E** no log de auditoria SQL. Verifique o **Entrar logs** para esses.

## Erros de autorização (logado, mas "permissão negada")
- Verifique se há **DENY** em algum lugar (direto ou através de um papel). DENY bate GRANT.
- Se um procedimento usa **dinâmico SQL**, o chamador precisa de direitos sobre as tabelas (cadeia de propriedade está quebrada).
- Teste como esse usuário: `EXECUTE AS USER = 'Ravi'; SELECT ...; REVERT;`.

## Verifique o connection string
Atualize o **connection string** do aplicativo se você:
- criou uma nova banco de dados,
- ou mudou o servidor, ou
- Senhas giradas.

Copie um novo do Azure portal: **SQL database → Cordas de conexão**. Em seguida, adicione o nome de usuário, senha e nome do servidor correto. Para conexões seguras, mantenha o `Encrypt=True;TrustServerCertificate=False;`.

## Para lembrar na prova
- A maioria dos erros curtos de "login falhou" ou "não há recursos suficientes" são **transitório**; eles desaparecem em **60 segundos**.
- Tente novamente: espere **5s** primeiro, depois **backoff exponencial**, com uma contagem de retry **max**.
- Tente novamente um **nova conexão**, não o falhado.
- Mais de 60 segundos de falhas → **Suporte do Azure**. Muitos usuários falham → **Azure Service Health**.
- Passos "Login falhou": login habilitado? → login existe? → **usuário** na banco de dados? → **permissões**?
- `sys.sql_logins` mostra `is_disabled`; corrigir com `ALTER LOGIN ... ENABLE`.
- Falha na auditoria logins → **Entrar logs**, não na auditoria SQL.

## Teste rápido
1. Um aplicativo recebe "Não há recursos suficientes para processar o pedido" por 30 segundos durante a escala. O que o aplicativo deve fazer?
   <details><summary>Resposta</summary>Tente novamente com um recuo exponencial em uma nova conexão. É uma falha transitória.</details>
2. O A login existe e está habilitado, mas o usuário recebe um erro ao abrir o banco de dados. Qual é o passo mais provável em falta?
   <details><summary>Resposta</summary>Não há nenhum usuário de banco de dados mapeado para o login. Crie um com CREATE USER... FOR LOGIN.</details>
3. Falhas de conexão duraram 10 minutos e Azure Service Health não mostra nenhuma interrupção. E depois?
   <details><summary>Resposta</summary>As falhas superiores a 60 segundos não são falhas transitórias normais.</details>
