# [PARCIAL] Área DEV - Administração

## 1. Identificação da página

- **Arquivo HTML:** `DEV/dev.html`
- **Módulo:** DEV / Administração
- **Arquivos CSS relacionados:** `global.css`, `DEV/dev.css`, `https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&display=swap`, `https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css`
- **Arquivos JavaScript relacionados:** `auth.js`, `DEV/dev-data.js`, `DEV/dev-main.js`, `global.js`
- **APIs/endpoints relacionados:** `/api/notifications.php actions: ack, alert_ack, clear_unread, list`, `/api/storage.php actions: admin_user_set_status, admin_user_stats, admin_user_stats_rollup, admin_users_list, audit_push, auth_logout, auth_session_status, get, set, user_presence_ping`
- **Perfil de usuário provável:** Master ou administrador técnico
- **Nível de criticidade:** Crítico
- **Prioridade de refatoração:** Urgente

---

## 2. Objetivo da página

Administrar usuários da plataforma, presença, status, estatísticas e criação de clientes/colaboradores.

- **Problema resolvido:** centraliza o fluxo de dev / administração para reduzir consulta manual, duplicação operacional e perda de rastreabilidade.
- **Quem utiliza:** Master ou administrador técnico.
- **Etapa do fluxo:** DEV / Administração dentro do Portal Sama.
- **Dados/documentos manipulados:** usuários, roles, departamentos, presença, eventos de auditoria, status e estatísticas de atividade.
- **Observação:** Por operar usuários, permissões e dados administrativos, esta tela deve ser tratada como superfície crítica de administração.

Atualizacao React 2026-06-01: a rota `/dev` em `portal-sama-web/src/pages/dev/DevAdminPage.tsx` agora tambem concentra a importacao operacional do Acessorias. O botao manual `Novo cliente` fica no cabecalho da area DEV; os botoes `Previa clientes`, `Importar clientes`, `Previa responsaveis`, `Importar responsaveis`, `Previa entregas`, `Sincronizar entregas`, `Gerar notificacoes` e `Importar tudo` ficam no painel `Integracao Acessorias`. A acao `Gerar notificacoes` foi adicionada em 2026-06-02 para disparo manual deduplicado de vencimento proximo, atraso, baixa e divergencia.

---

## 3. Funcionamento detalhado da interface

Cabeçalho identificado com área de usuário, departamento e ação de logout ou retorno, carregado em conjunto com `global.js`/`auth.js` quando presentes.

Navegação lateral/superior com atalhos para Início, Departamento, Clientes e Área Dev, reaproveitando a identidade visual global do portal.

Conteúdo principal identificado: Painel Administrativo Usuários da Plataforma Centro de controle para monitorar usuarios, presenca online e produtividade por periodo. Usuário ▾; Usuários 0 Base total de login Online agora 0 Atualização em tempo real Inativos/Bloqueados 0 Usuários sem acesso ativo Ativos (7 dias) 0 Com eventos na semana; Todos os departamentos Todos os papéis Master Gestor Colaborador Todos os status Ativo Inativo Bloqueado Excluído Somente online Atualizar Criar Novo colaborado; Usuários cadastrados 0 resultados Usuário Papel Departamento Status Presença Colaborador Eventos 7d Ações Nenhum usuário encontrado com os filtros atuais. Selec; Ritmo por dia -.

Títulos/seções visíveis: `Usuários da Plataforma`, `Usuários cadastrados`, `Selecione um usuário`, `-`.

Tabelas identificadas: `tabelaUsuarios` (`Usuário`, `Papel`, `Departamento`, `Status`, `Presença`, `Colaborador`, `Eventos 7d`, `Ações`).

Campos relevantes: `filtroBusca` (search), `filtroDepartamento` (select), `filtroRole` (select), `filtroStatus` (select), `filtroOnline` (checkbox), `inspectorMonth` (month).

Ações/botões identificados: `Início`, `Departamento ▾`, `Modelo padrão Departamento`, `Clientes`, `Área Dev`, `Usuário ▾`, `Departamento`, `Logout`, `Atualizar`, `Criar`, `Novo colaborador`, `Novo cliente`, `Ultimos 7 dias`, `Mensal`, `... mais 1 item(ns)`.

Fluxo esperado do usuário:

1. Acessar a página pela navegação interna ou link público/legado, conforme o caso.
2. O JavaScript relacionado carrega sessão, dados iniciais, filtros e tabelas.
3. O usuário preenche campos, seleciona filtros ou aciona botões de criação/edição/envio/download.
4. As APIs PHP recebem a requisição, aplicam validações e retornam JSON ou arquivo.
5. A interface atualiza status, tabelas, mensagens de sucesso/erro ou redireciona conforme o resultado.

---

## 4. Fluxo de dados

- Ao abrir a página, `auth.js` consulta `/api/storage.php?action=auth_session_status`; se a sessão não estiver autenticada, redireciona para `/index.html`. O usuário público fica em `sessionStorage` por meio de shim compatível com a chave legada `usuarioLogado`.

- Endpoints envolvidos: `/api/notifications.php`, `/api/storage.php`. As actions observadas são: `ack`, `alert_ack`, `clear_unread`, `list`, `admin_user_set_status`, `admin_user_stats`, `admin_user_stats_rollup`, `admin_users_list`, `audit_push`, `auth_logout`, `auth_session_status`, `get`, `set`, `user_presence_ping`.

- Uso de armazenamento no navegador identificado nos scripts relacionados: `localStorage` em 40 ocorrência(s) e `sessionStorage` em 8 ocorrência(s). Tokens/segredos não devem ser armazenados nesses mecanismos; manter apenas estado não sensível.

- Cookies de sessão são tratados pelo backend em `api/auth.php` com `HttpOnly`, `SameSite=Lax` e `Secure` condicional a HTTPS/proxy. Ponto a validar em produção: HTTPS obrigatório, HSTS efetivo e domínio/path restritivos.

Campos/dados de entrada identificados no HTML: `filtroBusca` (search), `filtroDepartamento` (select), `filtroRole` (select), `filtroStatus` (select), `filtroOnline` (checkbox), `inspectorMonth` (month)

Quando alguma action for indireta ou montada dinamicamente no JavaScript, considerar: **ponto a validar no código/backend**.

---

## 5. APIs e integrações relacionadas

### Endpoint: `/api/notifications.php`

- **Action observada:** `ack`
  - **Método provável:** POST
  - **Finalidade:** Marcar notificação como lida.
  - **Dados enviados:** ID da notificação.
  - **Dados retornados:** Confirmação.
  - **Validações esperadas:** Usuário dono ou autorizado.
  - **Riscos de segurança:** Alterar notificação de outro usuário.
  - **Melhorias recomendadas:** Validar propriedade no backend.
- **Action observada:** `alert_ack`
  - **Método provável:** POST
  - **Finalidade:** Marcar alerta como visualizado.
  - **Dados enviados:** ID da notificação.
  - **Dados retornados:** Confirmação.
  - **Validações esperadas:** Usuário dono ou autorizado.
  - **Riscos de segurança:** Perda de alerta crítico.
  - **Melhorias recomendadas:** Auditar alertas críticos.
- **Action observada:** `clear_unread`
  - **Método provável:** POST
  - **Finalidade:** Limpar não lidas.
  - **Dados enviados:** Escopo do usuário.
  - **Dados retornados:** Confirmação.
  - **Validações esperadas:** Usuário autenticado.
  - **Riscos de segurança:** Ocultar notificações importantes sem registro.
  - **Melhorias recomendadas:** Manter histórico e confirmação para alertas críticos.
- **Action observada:** `list`
  - **Método provável:** GET
  - **Finalidade:** Listar notificações do usuário/departamento.
  - **Dados enviados:** Filtros por usuário, departamento ou período.
  - **Dados retornados:** Lista de notificações.
  - **Validações esperadas:** Usuário autenticado e escopo por audiência.
  - **Riscos de segurança:** Exposição de notificações de outro usuário/departamento.
  - **Melhorias recomendadas:** Implementar paginação, escopo por servidor e retenção.

### Endpoint: `/api/storage.php`

- **Action observada:** `admin_user_set_status`
  - **Método provável:** POST/PATCH
  - **Finalidade:** Alterar status de usuário.
  - **Dados enviados:** Username e novo status.
  - **Dados retornados:** Confirmação.
  - **Validações esperadas:** Somente master e CSRF.
  - **Riscos de segurança:** Bloqueio/desbloqueio indevido de contas.
  - **Melhorias recomendadas:** Workflow com motivo, log e revisão.
- **Action observada:** `admin_user_stats`
  - **Método provável:** GET/POST
  - **Finalidade:** Consultar estatísticas de atividade do usuário.
  - **Dados enviados:** Username e período.
  - **Dados retornados:** Resumo semanal/mensal.
  - **Validações esperadas:** Somente master.
  - **Riscos de segurança:** Exposição comportamental.
  - **Melhorias recomendadas:** Controle de finalidade, retenção e minimização.
- **Action observada:** `admin_user_stats_rollup`
  - **Método provável:** POST
  - **Finalidade:** Consolidar estatísticas mensais.
  - **Dados enviados:** Mês/usuário.
  - **Dados retornados:** Resultado de rollup.
  - **Validações esperadas:** Somente master e CSRF.
  - **Riscos de segurança:** Manipulação de métricas ou perda de evidência.
  - **Melhorias recomendadas:** Job server-side agendado e assinado.
- **Action observada:** `admin_users_list`
  - **Método provável:** GET
  - **Finalidade:** Listar usuários administrativos.
  - **Dados enviados:** Filtros da tela.
  - **Dados retornados:** Usuários, perfis e status.
  - **Validações esperadas:** Somente master.
  - **Riscos de segurança:** Exposição de contas e privilégios.
  - **Melhorias recomendadas:** Paginação, mascaramento e RBAC rigoroso.
- **Action observada:** `audit_push`
  - **Método provável:** POST
  - **Finalidade:** Registrar evento de auditoria vindo do frontend/camada de serviços.
  - **Dados enviados:** JSON com usuário, mensagem, ação, entidade e metadados.
  - **Dados retornados:** Confirmação do registro.
  - **Validações esperadas:** Usuário autenticado, CSRF e normalização de metadados.
  - **Riscos de segurança:** Auditoria manipulável se aceitar usuário informado pelo cliente.
  - **Melhorias recomendadas:** Derivar usuário do servidor e tornar trilha append-only.
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
- **Action observada:** `get`
  - **Método provável:** GET
  - **Finalidade:** Carregar dados persistidos no KV/MySQL por chave.
  - **Dados enviados:** Query `key`.
  - **Dados retornados:** Valor persistido, metadados de atualização e backend.
  - **Validações esperadas:** Usuário autenticado e, para `sama_usuarios_login_v1`, papel master.
  - **Riscos de segurança:** Exposição horizontal de dados se a chave não for protegida por escopo.
  - **Melhorias recomendadas:** Substituir KV genérico por controllers por domínio e autorização por recurso.
- **Action observada:** `set`
  - **Método provável:** POST/PUT/PATCH
  - **Finalidade:** Salvar dados persistidos no KV/MySQL por chave.
  - **Dados enviados:** JSON com `value` e opcional `audit`.
  - **Dados retornados:** Confirmação, backend e auditoria opcional.
  - **Validações esperadas:** Usuário autenticado, CSRF e restrições por chave.
  - **Riscos de segurança:** Mass assignment, sobrescrita indevida e perda de histórico.
  - **Melhorias recomendadas:** Usar DTO/Form Requests, transações, validação por esquema e versionamento.
- **Action observada:** `user_presence_ping`
  - **Método provável:** GET/POST
  - **Finalidade:** Atualizar presença online do usuário.
  - **Dados enviados:** Sessão atual e dados de navegação.
  - **Dados retornados:** Status de presença.
  - **Validações esperadas:** Usuário autenticado.
  - **Riscos de segurança:** Rastreamento excessivo ou exposição de presença.
  - **Melhorias recomendadas:** Minimizar dados e documentar finalidade.

---

## 6. Regras de negócio identificadas

### Regras identificadas no frontend

- Filtros e formulários administram usuários, clientes, colaboradores, status e permissões.
- Ações administrativas incluem criação, alteração de status, senha e vínculo de clientes.

### Regras que devem existir obrigatoriamente no backend

- Somente master/admin.
- Validação server-side de senha, role e departamentos.
- Auditoria obrigatória de toda alteração administrativa.
- Impedir que usuário altere o próprio papel para escalar privilégios.

---

## 7. Análise de segurança

### 7.1 Autenticação

`auth.js` está relacionado à página. A validação inicial ocorre no frontend chamando `/api/storage.php?action=auth_session_status`; endpoints PHP precisam manter a validação real de sessão.

### 7.2 Autorização

A tela é administrativa. O backend deve exigir master/admin em todas as actions, inclusive leitura, alteração e exportação.

### 7.3 Proteção contra XSS

Foram identificadas 25 ocorrência(s) de `innerHTML` nos scripts vinculados. Evitar inserir dados de API com `innerHTML`; preferir `textContent` ou sanitização contextual.

### 7.4 Proteção contra CSRF

Para actions de mutação, o padrão do projeto usa `X-CSRF-Token` via `SamaAuth.buildHeaders` e `sama_require_csrf()` em APIs críticas. Validar cobertura em todas as actions desta página.

### 7.5 Proteção contra SQL Injection

Os endpoints PHP analisados usam `PDO`/`prepare` em diversas consultas. Ainda assim, qualquer consulta dinâmica, filtro, `ORDER BY`, nome de tabela/coluna ou action genérica deve usar allowlist e prepared statements. Ponto a validar em cada endpoint relacionado.

### 7.6 Segurança em uploads

Não há upload direto identificado nesta página.

### 7.7 Dados sensíveis

Dados sensíveis envolvidos: usuários, roles, departamentos, presença, eventos de auditoria, status e estatísticas de atividade. Não expor esses dados no HTML inicial, em logs do navegador, em mensagens de erro, em query strings permanentes ou em `localStorage`.

### 7.8 Logs e auditoria

Ações críticas desta página devem gerar trilha de auditoria com usuário derivado da sessão, entidade, antes/depois quando aplicável, IP, user-agent e timestamp. Evitar registrar valores sensíveis em claro.

---

## 8. Checklist de segurança da página

- [x] Página protegida contra acesso não autenticado.
- [x] Permissões validadas no backend.
- [ ] Inputs validados no frontend.
- [ ] Inputs validados no backend.
- [ ] Dados sensíveis não expostos no HTML.
- [ ] Dados sensíveis não expostos no JavaScript.
- [ ] Tokens não armazenados de forma insegura.
- [x] Cookies configurados com `HttpOnly`, `Secure` e `SameSite` no backend de autenticação; validar HTTPS/proxy em produção.
- [ ] Requisições críticas protegidas contra CSRF.
- [ ] Dados renderizados no DOM protegidos contra XSS.
- [ ] Endpoints protegidos contra SQL Injection.
- [x] Uploads validados por extensão, MIME type e conteúdo.
- [x] Arquivos armazenados fora da pasta pública.
- [x] Downloads protegidos por autenticação e autorização.
- [ ] Ações críticas registradas em auditoria.
- [ ] Erros tratados sem expor detalhes internos.
- [ ] Logs não expõem dados sensíveis.

---

## 9. Problemas técnicos encontrados

- Arquivos JavaScript extensos/acoplados à tela: `DEV/dev-data.js` (2582 linhas), `DEV/dev-main.js` (1199 linhas), `global.js` (2433 linhas).
- Uso de `innerHTML` em scripts relacionados (25 ocorrência(s)), exigindo revisão de XSS.
- Uso de `localStorage` em scripts relacionados (40 ocorrência(s)); não armazenar tokens ou dados sensíveis.
- Persistência genérica por `/api/storage.php?action=get|set` e chaves de domínio dificulta contratos claros de API, testes e autorização por recurso.

---

## 10. Melhorias recomendadas

### 10.1 Melhorias rápidas

- Centralizar chamadas `fetch` em um client com tratamento de erro, CSRF e timeout.
- Padronizar mensagens de erro sem expor detalhes internos.
- Garantir que botões sensíveis sejam ocultados e bloqueados por permissão no backend.
- Aplicar bloqueio server-side para master/admin em todos os endpoints usados pela tela.

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
- TanStack Query para cache/estado de servidor e um componente `DataTable` reutilizável com paginação e filtros server-side.

### Backend

- NestJS Guards, decorators de permissões e `PermissionService` para RBAC administrativo, com controllers separados por domínio e eliminação de chaves genéricas em KV.

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
- `DataTable`
- `StatusBadge`
- `FilterBar`
- `FormSection`
- `ValidatedInput`
- `ConfirmModal`

Serviços sugeridos:
- `authService`
- `auditService`
- `notificationService`
- `adminUserService`
- `clientAdminService`
- `permissionService`

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

### Testes de integração

- Endpoint deve rejeitar requisição sem autenticação quando a página for interna.
- Endpoint deve rejeitar usuário sem perfil/departamento permitido.
- Erro da API deve ser tratado na interface sem expor stack trace.

### Testes E2E

- Usuário autorizado acessa a página e executa o fluxo principal.
- Usuário sem permissão não visualiza nem executa ações críticas.

### Testes de segurança

- Tentar XSS em campos livres e dados retornados pela API.
- Tentar IDOR alterando IDs, client_id, doc_id ou token.
- Verificar CSRF em actions POST/PUT/PATCH/DELETE.

---

## 14. Conclusão da página

A página `DEV/dev.html` atua no módulo **DEV / Administração** e manipula **usuários, roles, departamentos, presença, eventos de auditoria, status e estatísticas de atividade**. Os principais riscos são: administração privilegiada. A principal recomendação é reforçar validações server-side, autorização por recurso, auditoria e componentização gradual.

- **Nível de criticidade:** Crítico
- **Prioridade de refatoração:** Urgente
- **Observações finais:** manter o HTML como camada de apresentação; regras de permissão, validação, persistência e auditoria devem residir no backend. Quando houver incerteza, tratar como **ponto a validar no código/backend**.


---

## 15. Atualização de stack alvo — TypeScript/NestJS

Esta seção complementa a análise original da página com a decisão técnica posterior de migrar o backend PHP para **Node.js + TypeScript + NestJS** e evoluir o frontend para **React + TypeScript + Vite**.

### 15.1 Mapeamento da tela para a arquitetura alvo

- **Arquivo HTML atual:** `DEV/dev.html`
- **Rota React sugerida:** `/dev`
- **Componente React sugerido:** `DevAdminPage.tsx`
- **Backend alvo:** `/api-v2`, implementado em NestJS
- **Banco principal:** MySQL 8 com Prisma ORM

### 15.2 Módulos NestJS relacionados

- `UsersModule`
- `RolesModule`
- `PermissionsModule`
- `AuditModule`

### 15.3 Arquivos atuais que devem ser usados como referência de migração

- `DEV/dev.html`
- `DEV/dev-main.js`
- `DEV/dev-data.js`
- `api/storage.php`

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

---

## 16. Atualizacao de implementacao - 2026-05-15 14:13

- **Rota React criada:** `/dev`.
- **Componente:** `portal-sama-web/src/pages/dev/DevAdminPage.tsx`.
- **Contratos consumidos:** `/api-v2/users`, `/api-v2/users/:id`, `/api-v2/users/:id/status`, `/api-v2/users/:id/roles`, `/api-v2/roles`, `/api-v2/roles/:id`, `/api-v2/roles/:id/permissions`, `/api-v2/permissions` e `/api-v2/permissions/:id`.
- **Servicos/tipos/schemas:** `portal-sama-web/src/services/admin.service.ts`, `portal-sama-web/src/types/admin.ts` e `portal-sama-web/src/schemas/admin.schema.ts`.
- **Cobertura da tela:** abas de usuarios, roles e permissoes; filtros; estatisticas; criacao/edicao; troca de status de usuario; troca de roles do usuario; troca de permissoes da role; criacao/edicao de permissoes.
- **Seguranca:** `PermissionGate` e checagens `users.*`, `roles.*`, `permissions.*` sao usadas apenas para UX. A autorizacao real continua no backend com JWT, CSRF, permissoes granulares, auditoria e bloqueios contra perda acidental de privilegios.
- **Validacao executada:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram.
- **Pendencias:** validar com API v2/MySQL/seed/backfill reais, usuarios/permissoes reais, migrar usuarios legados, decidir se presenca/estatisticas administrativas do legado continuam necessarias e criar Playwright.
