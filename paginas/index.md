# Login do Portal

## 1. Identificação da página

- **Arquivo HTML:** `index.html`
- **Módulo:** Autenticação
- **Arquivos CSS relacionados:** `global.css`, `Login.CSS`
- **Arquivos JavaScript relacionados:** `auth.js`, `Login.js`
- **APIs/endpoints relacionados:** `/api/storage.php actions: auth_forgot_password, auth_login, auth_logout, auth_session_status`
- **Perfil de usuário provável:** Usuário não autenticado e usuário autenticado em redirecionamento
- **Nível de criticidade:** Crítico
- **Prioridade de refatoração:** Urgente

---

## 2. Objetivo da página

Autenticar usuários internos e iniciar sessão server-side antes de permitir acesso às áreas internas.

- **Problema resolvido:** centraliza o fluxo de autenticação para reduzir consulta manual, duplicação operacional e perda de rastreabilidade.
- **Quem utiliza:** Usuário não autenticado e usuário autenticado em redirecionamento.
- **Etapa do fluxo:** Autenticação dentro do Portal Sama.
- **Dados/documentos manipulados:** credenciais de login, status da sessão, token CSRF retornado pelo backend.
- **Observação:** ponto a validar no código/backend para regras não explícitas no HTML.

---

## 3. Funcionamento detalhado da interface

Conteúdo principal identificado: Portal Interno Acesse sua conta Use suas credenciais para continuar na plataforma. Usuário Senha Entrar no portal Esqueceu a senha.

Títulos/seções visíveis: `Acesse sua conta`.

Formulários identificados: `loginForm` método `GET` com campos `loginUser`, `loginPass`.

Campos relevantes: `loginUser` (text), `loginPass` (password).

Ações/botões identificados: `Entrar no portal`, `Esqueceu a senha`.

Fluxo esperado do usuário:

1. Acessar a página pela navegação interna ou link público/legado, conforme o caso.
2. O JavaScript relacionado carrega sessão, dados iniciais, filtros e tabelas.
3. O usuário preenche campos, seleciona filtros ou aciona botões de criação/edição/envio/download.
4. As APIs PHP recebem a requisição, aplicam validações e retornam JSON ou arquivo.
5. A interface atualiza status, tabelas, mensagens de sucesso/erro ou redireciona conforme o resultado.

---

## 4. Fluxo de dados

- Ao abrir a página, `auth.js` consulta `/api/storage.php?action=auth_session_status`; se a sessão não estiver autenticada, redireciona para `/index.html`. O usuário público fica em `sessionStorage` por meio de shim compatível com a chave legada `usuarioLogado`.

- Endpoints envolvidos: `/api/storage.php`. As actions observadas são: `auth_forgot_password`, `auth_login`, `auth_logout`, `auth_session_status`.

- Dados preenchidos pelo usuário partem dos formulários/campos HTML e são tratados por JavaScript associado à página. Para campos críticos, a validação do frontend deve ser apenas auxiliar; a regra definitiva precisa existir no PHP.

- Uso de armazenamento no navegador identificado nos scripts relacionados: `localStorage` em 5 ocorrência(s) e `sessionStorage` em 8 ocorrência(s). Tokens/segredos não devem ser armazenados nesses mecanismos; manter apenas estado não sensível.

- Cookies de sessão são tratados pelo backend em `api/auth.php` com `HttpOnly`, `SameSite=Lax` e `Secure` condicional a HTTPS/proxy. Ponto a validar em produção: HTTPS obrigatório, HSTS efetivo e domínio/path restritivos.

Campos/dados de entrada identificados no HTML: `loginUser` (text), `loginPass` (password)

Quando alguma action for indireta ou montada dinamicamente no JavaScript, considerar: **ponto a validar no código/backend**.

---

## 5. APIs e integrações relacionadas

### Endpoint: `/api/storage.php`

- **Action observada:** `auth_forgot_password`
  - **Método provável:** GET
  - **Finalidade:** Informar fluxo operacional de redefinição de senha via usuário master.
  - **Dados enviados:** Nenhum dado sensível.
  - **Dados retornados:** Mensagem de suporte.
  - **Validações esperadas:** Não revelar existência de usuários.
  - **Riscos de segurança:** Fluxo manual pode gerar gargalo ou bypass social.
  - **Melhorias recomendadas:** Criar fluxo de reset com token temporário, expiração e auditoria.
- **Action observada:** `auth_login`
  - **Método provável:** POST
  - **Finalidade:** Autenticar usuário e criar sessão server-side.
  - **Dados enviados:** JSON com `username` e `password`.
  - **Dados retornados:** Usuário público, token CSRF e metadados de segurança.
  - **Validações esperadas:** Rate limiting por IP, bloqueio por tentativas, senha com hash e usuário ativo.
  - **Riscos de segurança:** Brute force, enumeração de usuário e vazamento de mensagens se não padronizadas.
  - **Melhorias recomendadas:** Adicionar MFA para perfis críticos, Argon2id, alertas de login anômalo e logs estruturados.
- **Action observada:** `auth_logout`
  - **Método provável:** POST
  - **Finalidade:** Encerrar sessão autenticada.
  - **Dados enviados:** Cookie de sessão e token CSRF quando houver usuário autenticado.
  - **Dados retornados:** Confirmação de logout.
  - **Validações esperadas:** CSRF para sessão autenticada.
  - **Riscos de segurança:** Logout CSRF ou sessão persistente indevida.
  - **Melhorias recomendadas:** Invalidar sessão no servidor e limpar cookies com atributos seguros.
- **Action observada:** `auth_session_status`
  - **Método provável:** GET
  - **Finalidade:** Validar sessão atual e retornar usuário público e token CSRF.
  - **Dados enviados:** Cookie de sessão `SAMASESSID`.
  - **Dados retornados:** JSON com `authenticated`, `user` e `csrf_token`.
  - **Validações esperadas:** Sessão válida, TTL absoluto/ocioso e usuário ativo.
  - **Riscos de segurança:** Se usado apenas no frontend, HTML ainda pode ser carregado sem sessão; endpoints devem bloquear dados.
  - **Melhorias recomendadas:** Manter sessão server-side, CSRF obrigatório para mutações e registrar login/logout em auditoria.

---

## 6. Regras de negócio identificadas

### Regras identificadas no frontend

- Campos de usuário e senha são obrigatórios para tentativa de login.
- Usuário autenticado é redirecionado para `/HOME/Inicio.html`.
- Erros de credencial são exibidos sem revelar senha.

### Regras que devem existir obrigatoriamente no backend

- Rate limiting por IP e por usuário.
- Bloqueio temporário/administrativo após tentativas inválidas.
- Hash de senha seguro e migração de senha legada.
- Geração de sessão server-side e token CSRF.

---

## 7. Análise de segurança

### 7.1 Autenticação

A página é de login; não deve exigir sessão prévia. Ela usa `/api/storage.php?action=auth_login` para criar sessão e `auth.js` para redirecionar usuário já autenticado.

### 7.2 Autorização

A autorização deve considerar perfil, departamento, propriedade do recurso e vínculo com cliente/processo. O frontend pode esconder botões, mas a decisão precisa estar no PHP.

### 7.3 Proteção contra XSS

Não foi identificada ocorrência direta de `innerHTML` nos scripts específicos vinculados, mas `global.js` pode renderizar conteúdos dinâmicos em páginas autenticadas. Evitar inserir dados de API com `innerHTML`; preferir `textContent` ou sanitização contextual.

### 7.4 Proteção contra CSRF

Para actions de mutação, o padrão do projeto usa `X-CSRF-Token` via `SamaAuth.buildHeaders` e `sama_require_csrf()` em APIs críticas. Validar cobertura em todas as actions desta página.

### 7.5 Proteção contra SQL Injection

Os endpoints PHP analisados usam `PDO`/`prepare` em diversas consultas. Ainda assim, qualquer consulta dinâmica, filtro, `ORDER BY`, nome de tabela/coluna ou action genérica deve usar allowlist e prepared statements. Ponto a validar em cada endpoint relacionado.

### 7.6 Segurança em uploads

Não há upload direto identificado nesta página.

### 7.7 Dados sensíveis

Dados sensíveis envolvidos: credenciais de login, status da sessão, token CSRF retornado pelo backend. Não expor esses dados no HTML inicial, em logs do navegador, em mensagens de erro, em query strings permanentes ou em `localStorage`.

### 7.8 Logs e auditoria

Ações críticas desta página devem gerar trilha de auditoria com usuário derivado da sessão, entidade, antes/depois quando aplicável, IP, user-agent e timestamp. Evitar registrar valores sensíveis em claro.

---

## 8. Checklist de segurança da página

- [ ] Página protegida contra acesso não autenticado.
- [ ] Permissões validadas no backend.
- [ ] Inputs validados no frontend.
- [ ] Inputs validados no backend.
- [ ] Dados sensíveis não expostos no HTML.
- [ ] Dados sensíveis não expostos no JavaScript.
- [ ] Tokens não armazenados de forma insegura.
- [x] Cookies configurados com `HttpOnly`, `Secure` e `SameSite` no backend de autenticação; validar HTTPS/proxy em produção.
- [ ] Requisições críticas protegidas contra CSRF.
- [x] Dados renderizados no DOM protegidos contra XSS.
- [ ] Endpoints protegidos contra SQL Injection.
- [x] Uploads validados por extensão, MIME type e conteúdo.
- [x] Arquivos armazenados fora da pasta pública.
- [x] Downloads protegidos por autenticação e autorização.
- [ ] Ações críticas registradas em auditoria.
- [ ] Erros tratados sem expor detalhes internos.
- [ ] Logs não expõem dados sensíveis.

---

## 9. Problemas técnicos encontrados

- Uso de `localStorage` em scripts relacionados (5 ocorrência(s)); não armazenar tokens ou dados sensíveis.
- Persistência genérica por `/api/storage.php?action=get|set` e chaves de domínio dificulta contratos claros de API, testes e autorização por recurso.

---

## 10. Melhorias recomendadas

### 10.1 Melhorias rápidas

- Centralizar chamadas `fetch` em um client com tratamento de erro, CSRF e timeout.
- Padronizar mensagens de erro sem expor detalhes internos.
- Garantir que botões sensíveis sejam ocultados e bloqueados por permissão no backend.

### 10.2 Melhorias de médio prazo

- Separar componentes de layout, tabelas, filtros, formulários e modais.
- Criar serviços de API por domínio em vez de usar actions genéricas espalhadas.
- Implementar RBAC por perfil/departamento com testes automatizados.
- Criar logs de auditoria estruturados para todas as ações críticas.

### 10.3 Melhorias estruturais

- Migrar gradualmente para React + TypeScript + Vite.
- Reorganizar backend em NestJS com Modules, Controllers, DTOs, Services, Guards, Pipes, Prisma Migrations e Permissions/Policies.
- Documentar contratos com OpenAPI/Swagger.
- Adicionar CI/CD com testes, lint e análise estática.
- Containerizar ambiente com Docker Compose e separar dev/homolog/prod.

---

## 11. Frameworks e tecnologias recomendadas para esta página

### Frontend

- React Hook Form + Zod para formulários tipados, validação consistente e mensagens de erro padronizadas.

### Backend

- NestJS com guards/middlewares de autenticação, CSRF quando houver cookies, autorização e filtro global de erros JSON.

### Segurança

- Sessão server-side com cookies `HttpOnly`, `Secure` e `SameSite`, evitando armazenar tokens sensíveis em `localStorage`.
- RBAC por perfil e departamento, validado no backend em todas as actions.

### Infraestrutura

- Docker Compose com Apache/Nginx, PHP 8+, MySQL e volumes privados para `data/private`, `logs` e backups.
- GitHub Actions com ESLint/Prettier, TypeScript strict, Jest/Supertest e Playwright em páginas críticas.

---

## 12. Sugestão de refatoração da página

Componentes sugeridos:
- `PageLayout`
- `Sidebar`
- `Header`
- `ActionButton`
- `FormSection`
- `ValidatedInput`
- `ConfirmModal`

Serviços sugeridos:
- `authService`
- `auditService`
- `notificationService`

Estratégia de refatoração:
- Separar layout, manipulação de DOM, regra de negócio e consumo de API.
- Criar camada de API com autenticação, CSRF, timeout, tratamento de erro e logs padronizados.
- Migrar primeiro os fluxos críticos para componentes tipados e cobertos por testes.
- Remover regras de negócio do frontend que possam ser burladas e revalidá-las no backend.
- Criar testes de regressão para permissões, XSS, CSRF e fluxos principais.

---

## 13. Testes recomendados

### Testes unitários

- Validar funções de formatação/normalização usadas pela página.
- Validar renderização de status e mensagens de erro sem HTML inseguro.
- Validar campos obrigatórios de login.
- Validar limpeza de senha em memória após erro.

### Testes de integração

- Endpoint deve rejeitar requisição sem autenticação quando a página for interna.
- Endpoint deve rejeitar usuário sem perfil/departamento permitido.
- Erro da API deve ser tratado na interface sem expor stack trace.
- Login com credenciais inválidas deve incrementar tentativas e respeitar rate limit.
- Logout deve exigir CSRF quando sessão estiver ativa.

### Testes E2E

- Usuário autorizado acessa a página e executa o fluxo principal.
- Usuário sem permissão não visualiza nem executa ações críticas.
- Usuário autenticado é redirecionado para o painel inicial.
- Usuário bloqueado não consegue acessar áreas internas.

### Testes de segurança

- Tentar XSS em campos livres e dados retornados pela API.
- Tentar IDOR alterando IDs, client_id, doc_id ou token.
- Verificar CSRF em actions POST/PUT/PATCH/DELETE.

---

## 14. Conclusão da página

A página `index.html` atua no módulo **Autenticação** e manipula **credenciais de login, status da sessão, token CSRF retornado pelo backend**. Os principais riscos são: autorização por perfil/departamento e exposição de dados. A principal recomendação é reforçar validações server-side, autorização por recurso, auditoria e componentização gradual.

- **Nível de criticidade:** Crítico
- **Prioridade de refatoração:** Urgente
- **Observações finais:** manter o HTML como camada de apresentação; regras de permissão, validação, persistência e auditoria devem residir no backend. Quando houver incerteza, tratar como **ponto a validar no código/backend**.


---

## 15. Atualização de stack alvo — TypeScript/NestJS

Esta seção complementa a análise original da página com a decisão técnica posterior de migrar o backend PHP para **Node.js + TypeScript + NestJS** e evoluir o frontend para **React + TypeScript + Vite**.

### 15.1 Mapeamento da tela para a arquitetura alvo

- **Arquivo HTML atual:** `index.html`
- **Rota React sugerida:** `/login`
- **Componente React sugerido:** `LoginPage.tsx`
- **Backend alvo:** `/api-v2`, implementado em NestJS
- **Banco principal:** MySQL 8 com Prisma ORM

### 15.2 Módulos NestJS relacionados

- `AuthModule`
- `UsersModule`

### 15.3 Arquivos atuais que devem ser usados como referência de migração

- `Login.js`
- `auth.js`
- `api/auth.php`
- `api/storage.php?action=auth_login`

### 15.4 Diretriz de migração para esta tela

1. Mapear todas as chamadas atuais a `/api` ou arquivos PHP relacionados.
2. Criar contratos equivalentes em `/api-v2` com DTOs tipados.
3. Validar autenticação, permissão e escopo no backend NestJS.
4. Substituir gradualmente `fetch`/JavaScript vanilla por services TypeScript no frontend.
5. Migrar a interface para React apenas depois de o endpoint crítico estar seguro.
6. Registrar auditoria para ações críticas desta página, principalmente criação, alteração, upload, download, aprovação, assinatura ou acesso a dados sensíveis.

### 15.5 Referências complementares

- [`../GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`](../GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md)
- [`../ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md`](../ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md)
- [`../BANCO_DADOS_MYSQL_PRISMA.md`](../BANCO_DADOS_MYSQL_PRISMA.md)
- [`../SEGURANCA.md`](../SEGURANCA.md)
- [`../MAPEAMENTO_MIGRACAO_APIS.md`](../MAPEAMENTO_MIGRACAO_APIS.md)
