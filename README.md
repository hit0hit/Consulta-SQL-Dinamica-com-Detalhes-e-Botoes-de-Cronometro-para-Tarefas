# html.sql
Essa consulta SQL recupera informações relacionadas a tarefas (`OKS_TAREFA`) e realiza várias operações, incluindo a geração de botões de cronômetro dinâmicos. Vamos analisar a estrutura da consulta:

1. **Seleção de Colunas:**
   ```sql
   SELECT 
       T.ID_TAREFA,
       to_char(T.ID_TAREFA) as ID_TAREFA_STR,
       T.DESCRICAO,
       decode(F.NOME, NULL, (
           -- Criação de descrição para equipe
           CASE 
               WHEN G.ID_EQUIPE = 1 THEN 'FISCAL'
               WHEN G.ID_EQUIPE = 2 THEN 'PESSOAL'
               WHEN G.ID_EQUIPE = 3 THEN 'CONTABIL'
               WHEN G.ID_EQUIPE = 4 THEN 'PARALEGAL'
               WHEN G.ID_EQUIPE = 5 THEN 'MONETIZADO'
               WHEN G.ID_EQUIPE = 6 THEN 'TODOS'
           END
       ), F.NOME) AS RESPONSAVEL,
       E.NOME_EMPRESA AS CLIENTE,
       E.DESCRICAO as CLIENTE_STR,
       decode(F.NOME, NULL, D.DESCRICAO, F.NOME) as RESPONSAVEL_STR,
       T.DATA_PREVISTA,
       CASE 
           WHEN T.ID_STATUS = 4 THEN 2
           WHEN T.ID_STATUS = 41 THEN 1
           ELSE NULL END as TAREFA_TYPE,
   ```
   - A seleção de colunas inclui informações sobre a tarefa, como ID, descrição, responsável, cliente, data prevista, tipo de tarefa, entre outras.

2. **Listas de Seleção Dinâmicas:**
   ```sql
   APEX_ITEM.SELECT_LIST_FROM_QUERY(
       1, T.ID_STATUS, 'SELECT DESCRICAO, ID_TAREFA_EMPRESA_STATUS FROM TAREFA_EMPRESA_STATUS', p_show_null => 'NO') as STATUS,
   APEX_ITEM.SELECT_LIST_FROM_QUERY(
       2,  T.ID_PRIORIDADE, 'SELECT DESCRICAO, ID_PRIORIDADE FROM PRIORIDADE', p_show_null => 'NO') as URGENCIA,
   ```
   - Cria listas de seleção dinâmicas para os campos de status e urgência.

3. **Aliases e Conversões:**
   ```sql
   P.DESCRICAO as URGENCIA_HR,
   SS.DESCRICAO as STATUS_HR,
   ```
   - Define aliases para descrições humanas de urgência e status.

4. **Botão de Cronômetro Dinâmico:**
   ```sql
   CASE WHEN T.ATIVO = 1 AND T.ID_STATUS != 4 THEN'<span data-descricao="'|| T.ID_TAREFA || '#' || T.DESCRICAO || '" data-id="'|| T.ID_TAREFA ||'" class="tk-cron-btn t-Icon fa fa-alarm-clock" aria-hidden="true"></span>' END as "CRON_BTN"
   ```
   - Gera um botão de cronômetro dinâmico com base nas condições especificadas.

5. **Cláusulas FROM e JOIN:**
   ```sql
   FROM
       OKS_TAREFA T
   INNER JOIN OKS_GRUPO_TAREFA G ON G.ID_GRUPO_TAREFA = T.OKS_GRUPO_TAREFA_ID AND (G.ID_EQUIPE = :GLOBAL_USER_DEPARTAMENTO OR :GLOBAL_USER_DEPARTAMENTO = 6)
   LEFT JOIN FUNCIONARIO F ON F.ID_FUNCIONARIO = T.ID_RESPONSAVEL
   INNER JOIN EMPRESA E ON E.ID_EMPRESA = G.ID_EMPRESA
   INNER JOIN PRIORIDADE P ON P.ID_PRIORIDADE = T.ID_PRIORIDADE
   INNER JOIN TAREFA_EMPRESA_STATUS SS ON SS.ID_TAREFA_EMPRESA_STATUS = T.ID_STATUS
   INNER JOIN DEPARTAMENTO D ON D.ID_DEPARTAMENTO = T.ID_CATEGORIA
   ```
   - Realiza junções entre várias tabelas para obter informações completas sobre a tarefa.

6. **Condições WHERE:**
   ```sql
   WHERE
       G.ATIVO = 1
       AND T.ATIVO = 1
       AND (T.ID_CATEGORIA = :P1_ID_DEPARTAMENTO_FILTRO OR :P1_ID_DEPARTAMENTO_FILTRO IS NULL)
       AND (
           :GLOBAL_USER_PERFIL = 'PERM_ADMIN'
           OR
           (T.ID_USUARIO = :GLOBAL_AUTH_FUNC_ID OR T.ID_RESPONSAVEL = :GLOBAL_AUTH_FUNC_ID)
       )
   ```
   - Aplica condições para filtrar os resultados com base na ativação, categoria de tarefa e perfil do usuário.

7. **Ordenação:**
   ```sql
   ORDER BY E.ID_EMPRESA DESC, T.ID_TAREFA ASC
   ```
   - Ordena os resultados por ID da empresa de forma descendente e ID da tarefa de forma ascendente.

Essa consulta recupera dados detalh

ados sobre as tarefas e usa funções dinâmicas para criar listas de seleção e botões de cronômetro.
