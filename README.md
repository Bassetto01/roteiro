# roteiro
---
A — Autenticação / Sessão
| ID   | Prioridade | Tipo      | Caso / Cenário                    | Pré-condição       | Passos                                                            | Dados de entrada                             | Resultado esperado                                                  | Resultado Obtido | Evidência |
| ---- | ---------: | --------- | --------------------------------- | ------------------ | ----------------------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------- | ---------------- | --------- |
| A-01 |    Crítico | Funcional | Login com credenciais válidas     | Usuário cadastrado | 1) Abrir login. 2) Inserir email/senha válidos. 3) Clicar Entrar. | email: `admin@mail.com` / senha: `Senha@123` | Redireciona para `index.php`; sessão criada; header mostra usuário. |                  |           |
| A-02 |    Crítico | Funcional | Login com senha inválida          | Usuário existente  | 1) Tentar logar com senha errada.                                 | email: `admin@mail.com` / senha: `errada`    | Mensagem de erro; não cria sessão; permanece no login.              |                  |           |
| A-03 |    Crítico | Segurança | SQL Injection na tela de login    | -                  | 1) Inserir payload `\' OR \'1\'=\'1` no campo email. 2) Enviar.   | email: `\' OR \'1\'=\'1`                     | Acesso NÃO concedido; aplicação trata entrada; exibe erro genérico. |                  |           |
| A-04 |       Alto | Funcional | Campos vazios no login            | -                  | 1) Enviar form sem preencher campos.                              | email: `/ senha:`                            | Validação (client/server); impedir envio; mostrar mensagem.         |                  |           |
| A-05 |      Médio | Funcional | Acesso a URL protegida sem sessão | Não logado         | 1) Acessar `customers/index.php` via URL.                         | -                                            | Redireciona para login; não mostrar conteúdo protegido.             |                  |           |
| A-06 |    Crítico | Funcional | Logout encerra sessão             | Logado             | 1) Clicar logout.                                                 | -                                            | Sessão destruída; volta ao login; páginas protegidas exigem login.  |                  |           |

---

B — Customers (Clientes)
B1 — Listagem e Busca

| ID   | Prioridade | Tipo        | Cenário                      | Pré-condição         | Passos                                      | Dados    | Resultado esperado                                               | Resultado Obtido | Evidência |
| ---- | ---------: | ----------- | ---------------------------- | -------------------- | ------------------------------------------- | -------- | ---------------------------------------------------------------- | ---------------- | --------- |
| B-01 |       Alto | Funcional   | Visualizar lista de clientes | >0 clientes no DB    | 1) Logar. 2) Acessar `customers/index.php`. | -        | Lista com colunas (nome, email, tel, ações); controles visíveis. |                  |           |
| B-02 |      Médio | Funcional   | Busca por nome existente     | Cliente `João Silva` | 1) Buscar `João` e submeter.                | `João`   | Retorna registros contendo `João`.                               |                  |           |
| B-03 |      Médio | Funcional   | Busca por termo inexistente  | -                    | 1) Buscar `xyz123`.                         | `xyz123` | Lista vazia + mensagem “Nenhum registro encontrado”.             |                  |           |
| B-04 |      Médio | Usabilidade | Ordenação e paginação        | > limite por página  | 1) Ordenar por coluna; 2) Navegar páginas.  | -        | Ordenação correta; paginação sem erros.                          |                  |           |

---

B2 — Adicionar (formulário)

| ID   | Prioridade | Tipo      | Cenário                            | Pré-condição              | Passos                                                   | Dados de entrada                                                                                          | Resultado esperado                                                                | Resultado Obtido | Evidência |
| ---- | ---------: | --------- | ---------------------------------- | ------------------------- | -------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- | ---------------- | --------- |
| B-10 |    Crítico | Funcional | Cadastrar cliente válido           | Logado                    | 1) Acessar `customers/add.php`. 2) Preencher. 3) Enviar. | nome: `Maria Silva`<br>email: `maria.silva@mail.com`<br>telefone: `11999998888`<br>endereço: `Rua A, 100` | Registro inserido no DB; mensagem de sucesso; retornar à lista com novo registro. |                  |           |
| B-11 |       Alto | Funcional | Enviar com campo obrigatório vazio | -                         | 1) Deixar campo obrigatório em branco. 2) Enviar.        | nome: `/ email:`                                                                                          | Validação impediu inserção; exibir mensagem.                                      |                  |           |
| B-12 |       Alto | Funcional | Email em formato inválido          | -                         | 1) Inserir `maria@@mail`. 2) Enviar.                     | email: `maria@@mail`                                                                                      | Rejeitar com mensagem de formato inválido.                                        |                  |           |
| B-13 |      Médio | Segurança | Upload com tipo inválido/tamanho   | Campo upload (se existir) | 1) Enviar `.exe` ou arquivo > limite.                    | arquivo: `arquivo.exe`                                                                                    | Rejeitar upload; exibir erro; não salvar arquivo.                                 |                  |           |
| B-14 |       Alto | Funcional | Duplicidade de email               | Existe `dup@mail.com`     | 1) Tentar cadastrar com `dup@mail.com`.                  | email: `dup@mail.com`                                                                                     | Se UNIQUE: recusar e mostrar erro; documentar comportamento.                      |                  |           |

---

B3 — Editar

| ID   | Prioridade | Tipo      | Cenário                     | Pré-condição      | Passos                                                     | Dados                               | Resultado esperado                               | Resultado Obtido | Evidência |
| ---- | ---------: | --------- | --------------------------- | ----------------- | ---------------------------------------------------------- | ----------------------------------- | ------------------------------------------------ | ---------------- | --------- |
| B-20 |    Crítico | Funcional | Editar dados válidos        | Cliente existente | 1) Acessar `customers/edit.php?id=X`. 2) Alterar e salvar. | telefone: `11977776666`             | Alteração persistida no DB; mensagem de sucesso. |                  |           |
| B-21 |       Alto | Funcional | Editar com email inválido   | -                 | 1) Inserir `email@invalido@@`. 2) Salvar.                  | email: `email@invalido@@`           | Rejeitar e exibir mensagem.                      |                  |           |
| B-22 |       Alto | Segurança | Injeção em campos de edição | -                 | 1) Inserir payload em nome. 2) Salvar.                     | nome: `'; DROP TABLE customers; --` | Entrada tratada; nenhum comando executado.       |                  |           |

---

B4 — Visualizar

| ID   | Prioridade | Tipo        | Cenário                     | Pré-condição                         | Passos               | Dados | Resultado esperado                                 | Resultado Obtido | Evidência |
| ---- | ---------: | ----------- | --------------------------- | ------------------------------------ | -------------------- | ----- | -------------------------------------------------- | ---------------- | --------- |
| B-30 |      Médio | Funcional   | Visualizar cliente completo | Cliente com todos campos preenchidos | 1) Abrir view/modal. | -     | Todos os campos exibidos; foto exibida se existir. |                  |           |
| B-31 |      Baixo | Usabilidade | Campos nulos/omissos        | Cliente com telefone vazio           | 1) Visualizar.       | -     | Exibir `Não informado`; layout preservado.         |                  |           |

---

B5 — Excluir

| ID   | Prioridade | Tipo       | Cenário                               | Pré-condição             | Passos                                   | Dados | Resultado esperado                                                               | Resultado Obtido | Evidência |
| ---- | ---------: | ---------- | ------------------------------------- | ------------------------ | ---------------------------------------- | ----- | -------------------------------------------------------------------------------- | ---------------- | --------- |
| B-40 |    Crítico | Funcional  | Excluir com confirmação               | Cliente sem dependências | 1) Clicar excluir. 2) Confirmar.         | -     | Registro removido do DB; mensagem de sucesso; não aparece na lista.              |                  |           |
| B-41 |       Alto | Funcional  | Cancelar exclusão                     | -                        | 1) Clicar excluir. 2) Cancelar no modal. | -     | Registro permanece sem alteração.                                                |                  |           |
| B-42 |    Crítico | Integração | Excluir cliente com dependências (FK) | Cliente referenciado     | 1) Tentar excluir.                       | -     | Sistema impede exclusão ou trata em cascata conforme regra; mensagem apropriada. |                  |           |

---

C — Livros

| ID   | Prioridade | Tipo      | Cenário                    | Pré-condição           | Passos                                              | Dados                                                              | Resultado esperado                                 | Resultado Obtido | Evidência |
| ---- | ---------: | --------- | -------------------------- | ---------------------- | --------------------------------------------------- | ------------------------------------------------------------------ | -------------------------------------------------- | ---------------- | --------- |
| L-01 |       Alto | Funcional | Listagem de livros         | >0 livros no DB        | 1) Acessar `livros/index.php`.                      | -                                                                  | Lista apresenta título, autor, ano, ações.         |                  |           |
| L-10 |    Crítico | Funcional | Cadastrar livro válido     | Logado                 | 1) Acessar `livros/add.php`. 2) Preencher e enviar. | título: `Dom Casmurro`<br>autor: `Machado de Assis`<br>ano: `1899` | Inserção no DB; mensagem de sucesso; listar livro. |                  |           |
| L-11 |       Alto | Funcional | Campos obrigatórios vazios | -                      | 1) Enviar sem título.                               | título: \`\`                                                       | Rejeitar com mensagem de validação.                |                  |           |
| L-12 |      Médio | Funcional | Editar livro               | Livro existente        | 1) Editar e salvar.                                 | ano: `1900`                                                        | Alteração persistida.                              |                  |           |
| L-13 |    Crítico | Funcional | Excluir livro              | Livro sem dependências | 1) Excluir e confirmar.                             | -                                                                  | Registro removido; não aparece na listagem.        |                  |           |

---

D — Users (Administração)

| ID   | Prioridade | Tipo      | Cenário                                  | Pré-condição      | Passos                                             | Dados                                                           | Resultado esperado                                                   | Resultado Obtido | Evidência |
| ---- | ---------: | --------- | ---------------------------------------- | ----------------- | -------------------------------------------------- | --------------------------------------------------------------- | -------------------------------------------------------------------- | ---------------- | --------- |
| U-01 |    Crítico | Funcional | Listar usuários                          | Usuários no DB    | 1) Acessar `users/index.php`.                      | -                                                               | Lista com nome, email, papel, ações.                                 |                  |           |
| U-02 |    Crítico | Segurança | Criar usuário com senha segura e hash    | Logado como admin | 1) Acessar `users/add.php`. 2) Preencher e salvar. | nome: `Teste`<br>email: `teste@mail.com`<br>senha: `Senha!2025` | Usuário criado; senha armazenada em hash verificável no DB.          |                  |           |
| U-03 |       Alto | Funcional | Rejeitar senha fraca                     | -                 | 1) Inserir senha `123`. 2) Salvar.                 | senha: `123`                                                    | Rejeitar; mensagem sobre política de senha (se implementada).        |                  |           |
| U-04 |       Alto | Funcional | Não permitir exclusão do próprio usuário | Logado            | 1) Tentar excluir o próprio usuário.               | -                                                               | Sistema recusa ou exige confirmação extra; documentar comportamento. |                  |           |

---

E — Template / Navegação / Links

| ID   | Prioridade | Tipo        | Cenário                       | Pré-condição | Passos                                            | Dados | Resultado esperado                                    | Resultado Obtido | Evidência |
| ---- | ---------: | ----------- | ----------------------------- | ------------ | ------------------------------------------------- | ----- | ----------------------------------------------------- | ---------------- | --------- |
| T-01 |      Médio | Funcional   | Verificar menu/header e links | Logado       | 1) Clicar em todos os links do menu/header.       | -     | Todas as páginas carregam (HTTP 200); logout visível. |                  |           |
| T-02 |      Baixo | Usabilidade | Responsividade                | -            | 1) Reduzir largura do browser / testar em mobile. | -     | Menu colapsa; botões legíveis; layout preservado.     |                  |           |

---

F — Banco de Dados e Integração

| ID    | Prioridade | Tipo       | Cenário                               | Pré-condição             | Passos                                                | Dados                  | Resultado esperado                                           | Resultado Obtido | Evidência |
| ----- | ---------: | ---------- | ------------------------------------- | ------------------------ | ----------------------------------------------------- | ---------------------- | ------------------------------------------------------------ | ---------------- | --------- |
| DB-01 |    Crítico | Integração | Conexão correta ao DB                 | `config.php` configurado | 1) Acessar app e listar registros.                    | -                      | Consultas retornam dados; sem warnings.                      |                  |           |
| DB-02 |    Crítico | Segurança  | Falha de conexão tratada              | Credenciais erradas      | 1) Mudar credenciais em `config.php`. 2) Acessar app. | -                      | Mensagem amigável; não vazar detalhes sensíveis; log gerado. |                  |           |
| DB-03 |    Crítico | Segurança  | Teste de SQL Injection em buscas/CRUD | -                        | 1) Inserir payloads em campos de busca/CRUD.          | payload: `' OR '1'='1` | Entradas tratadas; prepared statements; sem vazamento.       |                  |           |

---

G — Usabilidade / Acessibilidade

| ID    | Prioridade | Tipo           | Cenário                  | Pré-condição | Passos                                     | Dados | Resultado esperado                                                       | Resultado Obtido | Evidência |
| ----- | ---------: | -------------- | ------------------------ | ------------ | ------------------------------------------ | ----- | ------------------------------------------------------------------------ | ---------------- | --------- |
| UI-01 |      Médio | Usabilidade    | Mensagens de erro claras | -            | 1) Forçar validações nos formulários.      | -     | Mensagens legíveis; foco no campo com erro; instruções de correção.      |                  |           |
| UI-02 |      Baixo | Acessibilidade | Labels e contraste       | -            | 1) Verificar labels/placeholder/contraste. | -     | Formulários com labels associados; contraste adequado; botões com texto. |                  |           |

---

H — Performance (mínimo)

| ID   | Prioridade | Tipo        | Cenário                           | Pré-condição                 | Passos                            | Dados | Resultado esperado                                                         | Resultado Obtido | Evidência |
| ---- | ---------: | ----------- | --------------------------------- | ---------------------------- | --------------------------------- | ----- | -------------------------------------------------------------------------- | ---------------- | --------- |
| P-01 |      Médio | Performance | Tempo de carregamento da listagem | 200–1000 registros na tabela | 1) Acessar `customers/index.php`. | -     | Tempo de carga aceitável (< 3s ambiente local razoável); paginação eficaz. |                  |           |
