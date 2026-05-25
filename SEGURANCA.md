# Segurança do Portal Sama — versão atualizada para TypeScript/NestJS

## 1. Visão geral

O Portal Sama manipula documentos empresariais, contratos, propostas, certificados digitais, solicitações de acesso e dados internos de clientes/colaboradores. A migração para NestJS deve começar por segurança, não apenas por troca de linguagem.

Arquivos atuais mais relevantes para esta análise:

- `api/auth.php`
- `api/security_lib.php`
- `api/storage.php`
- `api/client_documents.php`
- `api/certificados.php`
- `api/legalizacao.php`
- `api/onboarding.php`
- `api/access_requests.php`
- `api/integra_ai.php`
- `auth.js`
- `Login.js`
- `DptClient/certificados-digitais.html`
- `Onboarding/documentos-cliente.html`
- `Legalizacao/assinatura.html`
- `Auditoria/autoria.html`

---

## 2. Matriz de risco atualizada

| Risco | Impacto | Probabilidade | Severidade | Recomendação |
|---|---:|---:|---:|---|
| Acesso direto a páginas HTML protegidas | Alto | Alta | Crítico | APIs devem validar sessão, permissão e escopo no backend. |
| IDOR em documentos de clientes | Crítico | Média | Crítico | Validar vínculo/propriedade antes de retornar metadados ou arquivo. |
| Upload de arquivo malicioso | Alto | Média | Alto | Validar extensão, MIME, assinatura, tamanho, usar ClamAV e storage privado. |
| XSS por conteúdo renderizado no DOM | Alto | Média | Alto | Evitar `innerHTML`, sanitizar HTML e aplicar CSP. |
| SQL Injection | Crítico | Baixa/Média | Crítico | Usar Prisma/queries parametrizadas e validação server-side. |
| Token público fraco/sem expiração | Crítico | Média | Crítico | Hash de token, expiração, escopo, revogação e rate limiting. |
| Certificado digital exposto | Crítico | Média | Crítico | Criptografia, storage privado, auditoria e permissão granular. |
| phpMyAdmin exposto | Alto | Média | Alto | Restringir acesso, senha forte e não expor MySQL publicamente. |
| `.env` versionado/distribuído | Crítico | Alta se ocorrer | Crítico | Remover, rotacionar segredos e usar variáveis do EasyPanel. |
| Falta de auditoria append-only | Alto | Média | Alto | Criar `AuditModule` centralizado. |

---

## 3. Autenticação

### Estado atual

O projeto usa `auth.js`, `Login.js`, `api/auth.php` e actions em `api/storage.php` para autenticação/sessão. Há uso de `sessionStorage` no frontend e indícios de CSRF/session no backend PHP.

### Arquitetura alvo

Criar:

```txt
apps/api/src/modules/auth/
  auth.module.ts
  auth.controller.ts
  auth.service.ts
  strategies/
    jwt-access.strategy.ts
    jwt-refresh.strategy.ts
  dto/
    login.dto.ts
```

Endpoints:

```txt
POST /api-v2/auth/login
POST /api-v2/auth/logout
POST /api-v2/auth/refresh
GET  /api-v2/auth/me
POST /api-v2/auth/forgot-password
POST /api-v2/auth/reset-password
```

Estratégia recomendada:

```txt
Access token curto + refresh token em cookie HttpOnly/Secure/SameSite
```

Status em 2026-05-08: a base foi iniciada em `portal-sama-api/src/modules/auth/` com `POST /api-v2/auth/login`, `/refresh`, `/logout`, `/forgot-password` e `GET /api-v2/auth/me`. O refresh token é opaco, salvo apenas como HMAC-SHA-256 em `RefreshToken.tokenHash` e rotacionado/revogado. Pontos a validar antes de produção: CSRF nas mutações com cookie, integração do frontend sem `localStorage`, seed/migração de usuários e MySQL real.

Atualização em 2026-05-08 14:14: `GET /api-v2/auth/csrf` foi adicionado e `POST /api-v2/auth/login`, `/refresh` e `/logout` exigem token CSRF double-submit assinado por HMAC. Ainda é ponto a validar no navegador/frontend real e com MySQL de homologação.

Atualizacao em 2026-05-14 14:54: `portal-sama-web` foi criado com login React inicial em `/login`, consumindo `/api-v2/auth/csrf`, `/login`, `/refresh`, `/logout`, `/me` e `/forgot-password`. Nesta fundacao React, o access token fica somente em memoria no Zustand; o refresh token permanece no cookie HttpOnly/Secure/SameSite da API v2. Ainda falta validar em navegador HTTPS com MySQL/homologacao e usuario real.

Atualizacao em 2026-05-25 17:37: `portal-sama-web` agora possui `npm.cmd run smoke:auth` para validar em homologacao a sequencia `csrf -> login -> me -> refresh -> me -> logout`. O script mantem cookies apenas em memoria, nao imprime senha/tokens e falha em HTTPS se o refresh cookie nao tiver `HttpOnly`, `SameSite` e `Secure`. Ainda falta executar contra EasyPanel com usuario real e registrar evidencia sem segredos.

Atualizacao em 2026-05-25 17:54: `portal-sama-web` agora possui `npm.cmd run test:e2e:real` para validar autenticacao real pelo navegador, usando Playwright Chromium contra o dominio publico. A suite fica bloqueada por `PORTAL_REAL_E2E=1`, recebe credenciais somente por variaveis de ambiente, desliga trace/video/screenshot e valida login, Home, ausencia de chaves sensiveis no storage, refresh cookie fora de `document.cookie`, politica `HttpOnly`/`SameSite`/`Secure` e logout. Ainda falta executar contra EasyPanel com usuario real.

Atualizacao em 2026-05-18 17:35: o motion system privado do `portal-sama-web` foi iniciado com `IntroGate`, splash padrao e boas-vindas somente apos autenticacao. A marcacao `welcomeAnimationSeen` usa `PATCH /api-v2/me/preferences/intro`, protegido por JWT e CSRF, com auditoria e persistencia em `User.metadata.portalIntro`. A intro nao aparece em rotas publicas, nao expoe dados sensiveis e nao grava tokens no navegador. Validacao com HTTPS/MySQL/usuario real e Playwright segue pendente.

Atualizacao em 2026-05-21 14:05: a intro visual oficial foi migrada para GSAP v2 em `portal-sama-web/src/features/intro`, mantendo `IntroGate`, `welcomeAnimationSeen`, `markWelcomeIntroSeen`, CSRF e auditoria. A nova camada consome apenas assets locais em `/brand/sama/intro/assets`, valida paths relativos e bloqueia remoto, `data:`, `javascript:`, `../`, `http:`, `https:` e `//`. Falha de manifesto, mapa, GSAP ou asset obrigatorio cai para fallback estatico e nao deve bloquear login, sessao ou navegacao. Rotas publicas seguem sem intro autenticada.

Atualizacao em 2026-05-21 14:24: o runtime GSAP deixou de ser importado estaticamente pelo hook da intro e passou a ser carregado dinamicamente dentro do efeito escopado. Se o import do GSAP falhar por cache/prebundle do Vite ou indisponibilidade do runtime, a intro marca falha de timeline, renderiza fallback estatico e preserva login, sessao e navegacao. Tambem foi adicionada pagina de erro de rota controlada para evitar a tela crua `Unexpected Application Error` em falhas futuras.

Atualizacao em 2026-05-22 14:05: a API v2 ganhou `npm run prisma:bootstrap-admin` para criar o primeiro usuario administrativo em banco limpo sem depender do legado. O comando usa apenas variaveis de ambiente `SAMA_BOOTSTRAP_ADMIN_*`, limita role inicial a `ADMIN` ou `DEV`, reaproveita o seed RBAC, registra auditoria, nao imprime senha e preserva senha existente salvo `SAMA_BOOTSTRAP_ADMIN_RESET_PASSWORD=true`. Validar no EasyPanel com `DATABASE_URL` real e usuario MySQL nao-root.

Evitar:

- refresh token em `localStorage`;
- refresh token em `sessionStorage`;
- autorização apenas no frontend;
- retorno de dados sensíveis sem sessão backend.

---

## 4. Autorização

Usar:

```txt
RBAC + permissões granulares + validação de escopo
```

Perfis iniciais:

```txt
ADMIN
MANAGER
CLIENT
DEPARTMENT
TI
ACCOUNTING
LEGALIZATION
AUDITOR
DEV
```

Permissões iniciais:

```txt
clients.read
clients.create
clients.update
clients.delete

documents.read
documents.upload
documents.download
documents.review
documents.delete
documents.requirements

certificates.read
certificates.download
certificates.manage

proposals.read
proposals.create
proposals.approve
proposals.reject

contracts.read
contracts.generate
contracts.sign

access_requests.read
access_requests.approve
access_requests.reject

audit.read

users.read
users.create
users.update
users.status
users.roles

roles.read
roles.create
roles.update
roles.permissions

permissions.read
permissions.create
permissions.update
```

Atualização em 2026-05-12 16:13: `UsersModule`, `RolesModule` e `PermissionsModule` foram iniciados em `portal-sama-api/src/modules/`. As rotas `GET /api-v2/users`, `GET /api-v2/users/:id`, `GET /api-v2/roles`, `GET /api-v2/roles/:id` e `GET /api-v2/permissions` exigem JWT e permissões `users.read`, `roles.read` ou `permissions.read`. Também foi criado `prisma/seed.ts` com catálogo inicial de permissões/roles, incluindo `audit.read`, sem criar usuário ou senha. `UsersModule` já possui criação, atualização, status e troca de roles; `RolesModule` possui criação, atualização e troca de permissões da role; `PermissionsModule` possui criação e atualização do catálogo. Mutações exigem permissões granulares, CSRF e auditoria. Pontos a validar antes de produção: execução do seed atualizado em MySQL real, migração de usuários legados, validação DEV/admin e validação de escopo por recurso.

Atualizacao em 2026-05-15 14:13: `portal-sama-web/src/pages/dev/DevAdminPage.tsx` passou a consumir `UsersModule`, `RolesModule` e `PermissionsModule` em `/dev`. A tela usa `users.read/create/update/status/roles`, `roles.read/create/update/permissions` e `permissions.read/create/update` apenas para UX; criacao, edicao, status, roles e permissoes seguem protegidos no backend por JWT, CSRF, auditoria e bloqueios de autopreservacao administrativa. Dados administrativos sao renderizados sem `innerHTML`. Validacao com API v2/MySQL/seed/backfill reais, usuarios reais, migracao legada e Playwright segue pendente.

Atualizacao em 2026-05-19 17:56: `portal-sama-web/src/pages/dev/DevCollaboratorsPage.tsx` passou a consumir `CollaboratorsModule` em `/dev/colaboradores`. A tela usa `collaborators.read/update/delete` e `roles.read` apenas para UX, envia edicao/arquivamento com CSRF via service centralizado, nao usa `innerHTML` e nao grava senha, perfis ou dados administrativos em storage do navegador. A validacao definitiva de roles, bloqueio de `CLIENT`, escopo, auditoria e hash de senha continuam no backend; validacao com API v2/MySQL/usuarios reais e Playwright segue pendente.

Atualizacao em 2026-05-15 15:06: `portal-sama-web/src/pages/proposals/ProposalPage.tsx` e `PublicProposalPage.tsx` passaram a consumir `ProposalsModule` em `/legalizacao/propostas`, `/legalizacao/propostas/:id` e `/onboarding/publico/proposta/:token`. A tela interna usa `proposals.read/create/approve/reject` apenas para UX, e as mutacoes continuam protegidas no backend por JWT, CSRF, escopo e auditoria. A tela publica opera por token opaco validado no backend, nao grava token em storage do navegador e renderiza campos comerciais como texto, sem `innerHTML`. Validacao com API v2/MySQL/backfill reais, tokens legados, HTTPS e Playwright segue pendente.

Atualizacao em 2026-05-15 16:13: `portal-sama-web/src/pages/contracts/ContractPage.tsx` e `PublicSignaturePage.tsx` passaram a consumir `ContractsModule` em `/legalizacao/contratos`, `/legalizacao/contratos/:id` e `/assinatura/:token`. A tela interna usa `contracts.read/generate/sign` apenas para UX; criacao, edicao, geracao, renderizacao PDF, importacao PDF e envio para assinatura continuam protegidos no backend por JWT, CSRF, escopo, validacao server-side, storage privado e auditoria. A tela publica opera por token opaco validado no backend, nao grava token em storage do navegador e exibe o contrato como texto derivado do HTML, sem `innerHTML`. Validacao com API v2/MySQL/storage/Legal Doc Service reais, tokens legados, HTTPS e Playwright segue pendente.

Atualizacao em 2026-05-13 10:50: `ClientsModule` foi iniciado em `portal-sama-api/src/modules/clients/`. As rotas de leitura exigem JWT e `clients.read`; criacao, atualizacao e arquivamento logico exigem `clients.create`, `clients.update` ou `clients.delete`, CSRF e auditoria. O escopo fino por vinculo cliente-usuario/gestor ainda depende de backfill/modelagem real.

Atualizacao em 2026-05-15 08:59: o frontend React passou a consumir `ClientsModule` em `/clientes` e `/clientes/:clientId/painel`. A tela usa permissoes apenas para esconder acoes de UX, nao grava dados de cliente em `localStorage` e envia mutacoes pelo client API v2 com CSRF. A validacao de escopo real, backfill e testes em homologacao continuam obrigatorios antes de producao.

Atualizacao em 2026-05-18 15:58: `portal-sama-web/src/pages/departments/DepartmentClientsPage.tsx` passou a consumir `ClientsModule` em `/departamentos/clientes`. A tela usa `clients.read` apenas para UX, nao usa `innerHTML`, nao grava dados de clientes em storage do navegador e nao implementa atribuicao/transferencia de responsavel enquanto nao houver contrato backend transacional, auditoria e modelagem/backfill de carteira/departamento. Validacao com API v2/MySQL/usuarios reais e escopo por departamento segue pendente.

Atualizacao em 2026-05-19 12:16: `TransfersModule` foi iniciado em `portal-sama-api/src/modules/transfers/`. As rotas de leitura exigem JWT e `transfers.read`; criacao e retorno manual exigem `transfers.create`/`transfers.return`, CSRF, papel ADMIN/DEV/MANAGER e escopo por departamento. A criacao e o retorno manual sao auditados em `AuditService`; erros retornam codigos controlados sem expor stack trace. Ainda faltam validacao HTTP com usuarios reais, MySQL/homologacao, seed RBAC atualizado e teste de IDOR/escopo com carteira real.

Atualizacao em 2026-05-19 15:33: `portal-sama-web/src/pages/manager/ManagerTransfersPage.tsx` passou a consumir o `TransfersModule` em `/manager/transferencias`. A tela usa `transfers.read/create/return` apenas para UX, envia criacao/retorno com CSRF via service centralizado, nao usa `innerHTML` e nao grava dados de carteira em storage do navegador. A autorizacao definitiva, escopo por departamento, auditoria e validacoes transacionais continuam no backend; validacao com API v2/MySQL/usuarios reais e Playwright segue pendente.

Atualizacao em 2026-05-19 15:53: `portal-sama-web/src/pages/manager/ManagerCollaboratorsPage.tsx` passou a consumir `GET /api-v2/transfers/dashboard` em `/manager/colaboradores` para consulta de carteira por colaborador. A tela usa `transfers.read` apenas para UX, nao faz mutacoes, nao usa `innerHTML` e nao grava dados de carteira em storage do navegador. A validacao de escopo por departamento e carteira continua no backend; validacao com MySQL/usuarios reais e Playwright segue pendente.

Atualizacao em 2026-05-20 11:20: `portal-sama-api/src/modules/managers/` passou a expor leitura de historico operacional em `GET /api-v2/managers/history` e `GET /api-v2/managers/history/:companyId/timeline`. As rotas exigem JWT, `manager_history.read`, papel ADMIN/DEV/MANAGER e escopo por departamento; a tela `/manager/historico` nao usa `innerHTML`, nao faz mutacoes e nao grava historico em storage local. Escrita/edicao do historico deve ser implementada separadamente com CSRF e auditoria.

Atualizacao em 2026-05-20 11:45: `ManagersModule` passou a expor `POST /api-v2/managers/history`, `PATCH /api-v2/managers/history/:id` e `POST /api-v2/managers/company-life`. As mutacoes exigem JWT, `manager_history.write`, papel ADMIN/DEV/MANAGER, escopo por departamento, CSRF double-submit e auditoria `AuditService`; a tela `/manager/historico` chama essas rotas via service centralizado com CSRF, sem `innerHTML` e sem storage local.

Atualizacao em 2026-05-20 12:30: `ManagersModule` passou a expor `GET /api-v2/managers/overview`. A leitura exige JWT, `manager_history.read`, papel ADMIN/DEV/MANAGER e escopo por departamento; a API filtra usuarios administrativos/clientes/auditores do painel operacional e le `sama_user_presence` apenas para presenca. `/manager` consome o contrato via service centralizado, sem `innerHTML` e sem storage local.

Atualizacao em 2026-05-20 16:02: `CalendarModule` passou a expor `DELETE /api-v2/calendar/entries/:id`. A mutacao exige JWT, `calendar.manage`, papel ADMIN/DEV/MANAGER, escopo por departamento, CSRF double-submit e auditoria; valida regra/empresa informadas, remove o vinculo empresa-vencimento, limpa notificacoes de vencimento e remove a regra quando fica orfa. `/manager` mostra a acao apenas por permissao e chama o service centralizado com `issueCsrfToken()`.

Atualizacao em 2026-05-20 16:21: `/notificacoes` no React passou a usar SSE por `fetch` autenticado contra `GET /api-v2/notifications/stream`, mantendo o bearer access token em memoria e fora de query string. A assinatura/cancelamento Web Push no navegador usa gesto do usuario, Service Worker servido por `portal-sama-web/public/sama-push-sw.js`, CSRF double-submit nas mutacoes e backend com JWT/RBAC/escopo. Nao ha persistencia de tokens/segredos em `localStorage`.

Atualização em 2026-05-11 08:40: `DocumentsModule` foi iniciado em `portal-sama-api/src/modules/documents/`. As rotas de documentos exigem JWT e permissões `documents.read`, `documents.upload`, `documents.download` ou `documents.delete`; upload e arquivamento exigem CSRF. O escopo real cliente/gestor/departamento ainda precisa ser validado contra dados persistidos antes de produção.

Atualizacao em 2026-05-15 09:16: o painel React do cliente passou a consumir `DocumentsModule` para checklist, upload, download, revisao e requisitos customizados. A tela so consulta documentos com `documents.read`, nao persiste metadados sensiveis no navegador e usa CSRF via client centralizado nas mutacoes. A validacao definitiva continua no backend; antes de producao ainda e obrigatorio testar IDOR/escopo com clientes reais, storage privado, ClamAV/EICAR e HTTPS/homologacao.

Atualizacao em 2026-05-18: `portal-sama-web/src/pages/documents/PublicDocumentsPage.tsx` passou a consumir o upload publico do `DocumentsModule` em `/onboarding/publico/documentos/:token`. A tela publica nao grava token em storage do navegador, nao usa `innerHTML` e faz apenas validacao visual de extensao/tamanho; a autorizacao contextual, expiracao/revogacao/escopo do token, validacao de arquivo, quarentena, scanner, storage privado e auditoria seguem no backend. Atualizacao em 2026-05-25 17:19: a tela passou a consumir os aliases especificos `GET /api-v2/public/onboarding/documents/:token` e `POST /api-v2/public/onboarding/documents/:token/upload`; ainda faltam tokens reais/legados, HTTPS/homologacao, upload real e Playwright de upload.

Atualização em 2026-05-11 08:59: requisitos documentais customizados foram migrados para `DocumentRequirement` e `POST /api-v2/documents/custom-requirements`. A rota exige JWT, permissão `documents.requirements`, papel gerencial, CSRF e auditoria. A validação de vínculo real gestor-cliente segue ponto a validar em MySQL/homologação.

Atualizacao em 2026-05-11 13:33: `PATCH /api-v2/documents/:id/status` foi criado para revisao de documentos com status `PENDING`, `APPROVED` ou `REJECTED`. A rota exige JWT, permissao `documents.review`, CSRF, escopo por papel/departamento e auditoria; `ARCHIVED` segue reservado ao arquivamento logico.

Atualizacao em 2026-05-13 10:15: `AccessRequestsModule` foi criado em `portal-sama-api/src/modules/access-requests/`. As rotas de leitura/decisao usam permissoes `access_requests.read`, `access_requests.approve` e `access_requests.reject`; criacao e decisao exigem CSRF; o escopo e derivado do solicitante autenticado, gestor responsavel, departamento e papeis privilegiados (`ADMIN`, `DEV`, `TI`). Falta validar com MySQL real, backfill legado e frontend.

Atualizacao em 2026-05-15 09:44: `portal-sama-web/src/pages/access-requests/AccessRequestPage.tsx` passou a consumir o `AccessRequestsModule` em `/solicitacao-acesso`. A tela libera envio e acompanhamento individual para sessao autenticada e usa permissoes `access_requests.read/approve/reject` apenas para historico e decisoes na UX. A criacao/decisao segue protegida no backend por JWT, CSRF, escopo e auditoria. Validacao com API v2, MySQL/backfill, usuarios reais, escopo por gestor/departamento/TI e Playwright segue pendente.

Atualizacao em 2026-05-18 15:35: `portal-sama-web/src/pages/ti/TiAccessPage.tsx` passou a consumir o `AccessRequestsModule` em `/ti/acessos`. A tela operacional usa `access_requests.read/approve/reject` apenas para UX, nao usa `innerHTML`, nao grava dados em storage do navegador e delega leitura/decisao ao backend com JWT, CSRF nas decisoes, RBAC, escopo e auditoria. A base editavel de credenciais da TI nao foi migrada por conter possiveis segredos; segue ponto a validar ate existir cofre com criptografia/vault, auditoria de leitura e politica de retencao.

Atualizacao em 2026-05-15 13:51: `portal-sama-web/src/pages/onboarding/OnboardingProcessesPage.tsx` passou a consumir o `OnboardingModule` em `/onboarding/processos`. A tela usa `onboarding.read/create/update/status/documents/delete` apenas para UX; criacao, edicao, status, vinculo de documentos e arquivamento seguem protegidos no backend por JWT, CSRF, escopo por responsavel/criador/departamento e auditoria. Dados da timeline e metadados sao renderizados sem `innerHTML`. Validacao com API v2/MySQL/backfill reais, usuarios/permissoes reais, fluxos publicos especificos e Playwright segue pendente.

Regras obrigatórias:

- cliente só acessa seus próprios recursos;
- gerente só acessa clientes vinculados;
- departamento só acessa processos/documentos autorizados;
- auditor acessa logs, mas não automaticamente todos os documentos;
- DEV/Admin é perfil altamente restrito;
- TI só acessa solicitações e recursos do próprio escopo.

---

## 5. Proteção contra IDOR

IDOR é um risco crítico. Mesmo autenticado, o usuário não pode acessar dados alterando um `id` na URL.

Exemplo de risco atual:

```txt
GET /api/client_documents.php?action=download&id=123
```

Nova rota:

```txt
GET /api-v2/documents/123/download
```

A API deve validar:

1. usuário autenticado;
2. permissão `documents.download`;
3. vínculo entre usuário e documento;
4. vínculo entre usuário e cliente dono do documento;
5. status do documento;
6. auditoria do download.

---

## 6. XSS

Pontos de atenção:

- contratos/propostas podem renderizar HTML rico;
- CKEditor/html2pdf podem lidar com conteúdo dinâmico;
- páginas legadas podem usar `innerHTML`;
- dados da API podem ir direto para DOM.

Recomendações:

- evitar `dangerouslySetInnerHTML` no React;
- sanitizar HTML com DOMPurify quando conteúdo rico for inevitável;
- sanitizar também no backend quando o conteúdo for persistido;
- aplicar CSP gradualmente;
- usar escaping padrão do React;
- validar campos textuais no backend.

---

## 7. CSRF

Se autenticação usa cookies, mutações críticas precisam de CSRF.

Rotas críticas:

- login/logout/refresh;
- upload;
- criação/edição/exclusão de cliente;
- aprovação/rejeição de proposta;
- geração/assinatura de contrato;
- solicitação/aprovação de acesso;
- alteração de permissões;
- download de certificado.

Estratégias:

- SameSite Lax/Strict quando aplicável;
- cabeçalho customizado obrigatório;
- validação de Origin/Referer;
- CSRF token double-submit para ações críticas.

Status em 2026-05-08 14:14: o `AuthModule` implementa token CSRF double-submit assinado, cookie `sama_csrf_token` e cabeçalho `x-csrf-token`. A estratégia deve ser reutilizada nas próximas mutações sensíveis, como upload, aprovações, permissões e alterações cadastrais.

Status em 2026-05-11 13:33: o upload, o arquivamento lógico, requisitos customizados e a revisao de status de documentos no `DocumentsModule` já exigem o mesmo fluxo de CSRF. Ainda falta validar em navegador/frontend real.

Status em 2026-05-13 10:15: `POST /api-v2/access-requests`, `POST /api-v2/access-requests/:id/approve` e `POST /api-v2/access-requests/:id/reject` tambem exigem CSRF. E2E cobre rejeicao sem CSRF; falta validar o fluxo em navegador/frontend real.

Status em 2026-05-20 11:45: `POST /api-v2/managers/history`, `PATCH /api-v2/managers/history/:id` e `POST /api-v2/managers/company-life` reutilizam o mesmo `CsrfService` e exigem cookie + cabecalho antes de gravar historico/vida da empresa.

Status em 2026-05-20 16:02: `DELETE /api-v2/calendar/entries/:id` tambem reutiliza o `CsrfService`; a exclusao de vencimento so ocorre com cookie + cabecalho validos, `calendar.manage` e escopo de departamento confirmado.

---

## 8. Upload seguro

Arquivos atuais envolvidos:

- `api/client_documents.php`
- `api/certificados.php`
- `api/onboarding.php`
- `api/legalizacao.php`

Requisitos obrigatórios:

- allowlist de extensão;
- validação de MIME type;
- validação de assinatura mágica;
- limite de tamanho;
- renomeação segura;
- hash SHA-256;
- armazenamento fora da pasta pública;
- bloqueio de execução;
- ClamAV;
- metadados no MySQL;
- auditoria;
- download somente por endpoint autenticado/autorizado.

Fluxo seguro:

```txt
Usuário envia arquivo
  ↓
API valida autenticação
  ↓
API valida autorização e escopo
  ↓
API valida extensão/MIME/assinatura/tamanho
  ↓
API grava em quarentena
  ↓
ClamAV verifica arquivo
  ↓
API move para storage privado
  ↓
API salva metadados no MySQL
  ↓
API registra auditoria
```

Status em 2026-05-11 08:40: `DocumentsModule` implementa a primeira versão NestJS do fluxo para documentos de cliente, com validação de extensão, MIME, assinatura mágica, tamanho configurável e hash SHA-256, gravação em storage privado e metadados no Prisma.

Status em 2026-05-11 08:59: `GET /api-v2/documents/templates`, `GET /api-v2/clients/:clientId/documents` e `GET /api-v2/documents/required-pending-summary` foram adicionados para checklist e pendências documentais. Esses endpoints não entregam arquivo físico; ainda precisam ser validados com dados reais para IDOR/escopo.

Status em 2026-05-11 13:33: `PATCH /api-v2/documents/:id/status` foi adicionado para aprovar, rejeitar ou reabrir documentos pendentes. O endpoint nao entrega arquivo fisico, mas altera estado sensivel e por isso exige `documents.review`, CSRF, escopo e auditoria.

Status em 2026-05-11 13:55: o upload NestJS passou a gravar o arquivo primeiro em quarentena privada, executar regras estaticas de seguranca e tentar scanner externo `clamscan`/`clamdscan` configuravel por `SAMA_UPLOAD_SCAN_*` antes do storage final. Se a varredura rejeitar, o arquivo nao e persistido no storage final nem no Prisma. Ainda falta validar ClamAV real, EICAR e `SAMA_UPLOAD_SCAN_MODE=strict` em homologacao/producao.

---

## 9. Certificados digitais

`DptClient/certificados-digitais.html` e `api/certificados.php` tratam dados críticos.

Regras obrigatórias:

- nunca retornar senha de certificado em resposta JSON;
- criptografar campos sensíveis;
- auditar todo acesso/download/alteração;
- restringir por permissão granular;
- controlar vencimento;
- gerar alerta de vencimento;
- separar arquivo em storage privado e metadados no MySQL.

Status em 2026-05-12 19:50: `CertificatesModule` implementa a primeira versão NestJS de certificados digitais com CRUD, rotação de senha, validação `.p12/.pfx`, storage privado, criptografia AES-256-GCM, download protegido por `certificates.download`, CSRF nas mutações e auditoria. Ainda faltam validação com MySQL/storage reais, backfill do legado e integração frontend.

Atualizacao em 2026-05-15 08:33: `portal-sama-web/src/pages/certificates/CertificatesPage.tsx` foi criado para consumir `/api-v2/certificates` com listagem, cadastro multipart, download, edicao, rotacao de senha e remocao. A tela nao revela senha de certificado e usa permissoes apenas para UX; a protecao real permanece no backend. Ainda faltam validar o fluxo integrado em HTTPS/homologacao com MySQL/storage reais e criar Playwright.

---

## 10. Tokens públicos

Arquivos atuais com tokens públicos:

- `api/onboarding.php`
- `api/legalizacao.php`
- `api/public_token_lib.php`
- `Onboarding/documentos-cliente.html`
- `Onboarding/proposta-cliente.html`
- `Legalizacao/assinatura.html`

Status em 2026-05-12 13:52: `DocumentsModule` implementa `PublicToken` para upload público de documentos, persiste apenas SHA-256 do token, valida expiração/escopo/revogação, registra auditoria, atualiza último uso e expõe gestão administrativa protegida por `documents.public_tokens` em `/api-v2/documents/public-tokens`. Atualizacao em 2026-05-18: a tela React publica `/onboarding/publico/documentos/:token` passou a consumir esse upload sem persistir o token no navegador. Atualizacao em 2026-05-25 17:19: o fluxo tambem expoe/consome `GET /api-v2/public/onboarding/documents/:token` e `POST /api-v2/public/onboarding/documents/:token/upload`, mantendo os endpoints genericos por compatibilidade. Ainda faltam migration/seed/backfill, validação MySQL real, tokens legados e rate limiting distribuído.

Status em 2026-05-13 08:57: `ProposalsModule` reutiliza `PublicToken` para propostas públicas em `GET/POST /api-v2/public/proposals/:token`; o envio interno gera token opaco `sama_pub_*`, grava apenas SHA-256, revoga tokens ativos anteriores, valida expiração/revogação/módulo/escopo e revoga o token após a resposta pública. Atualizacao em 2026-05-15 15:06: o frontend React interno/publico de propostas foi criado sem expor token em storage nem renderizar HTML bruto. Ainda faltam aplicar/validar em MySQL de homologação, backfill de tokens/propostas legadas, compatibilidade com tokens antigos e Playwright.

Status em 2026-05-13 09:34: `ContractsModule` reutiliza `PublicToken` para assinatura pública em `GET /api-v2/public/signatures/:token` e `POST /api-v2/public/signatures/:token/sign`; o envio interno gera token opaco `sama_pub_*`, grava apenas SHA-256, revoga tokens ativos anteriores, valida expiração/revogação/módulo/escopo e revoga o token após assinatura. A evidência inicial armazena hash do documento/assinatura, IP e user-agent, sem token bruto em auditoria. Atualizacao em 2026-05-15 16:13: o frontend React interno/publico de contratos foi criado sem expor token em storage nem renderizar HTML bruto com `innerHTML`. Ainda faltam baseline/migrations em MySQL real, PDF assinado em storage privado, backfill legado, tokens legados e Playwright.

Requisitos:

- token aleatório forte;
- armazenamento apenas de hash;
- expiração;
- escopo;
- revogação;
- rate limiting;
- auditoria;
- não permitir enumeração;
- limitar ações possíveis com token público.

---

## 11. Auditoria

Criar `AuditModule` como prioridade.

Eventos obrigatórios:

- login;
- logout;
- falha de login;
- refresh token;
- upload;
- download;
- acesso a certificado;
- alteração de cliente;
- alteração de colaborador;
- alteração de responsável;
- aprovação/rejeição de proposta;
- geração de contrato;
- assinatura;
- solicitação/aprovação/rejeição de acesso;
- alteração de permissões;
- backup/restauração;
- erro crítico.

Modelo recomendado:

```txt
AuditLog
  id
  userId
  action
  module
  entityType
  entityId
  ipAddress
  userAgent
  metadata
  createdAt
```

Status em 2026-05-08 14:40: `AuditModule` foi iniciado em `portal-sama-api/src/modules/audit/`. O `AuthModule` passou a usar `AuditService` centralizado, metadados sensíveis são mascarados por chaves como senha/token/secret/cookie e a consulta `GET /api-v2/audit/logs` exige JWT com permissão `audit.read`. Pontos a validar antes de produção: MySQL real, seeds de permissão, retenção, integração da tela `/auditoria` e filtros finais.

Status em 2026-05-13 14:43: `CollaboratorsModule` registra auditoria em `collaborators.create`, `collaborators.update` e `collaborators.archive`, sem persistir senha ou hash nos metadados. As mutacoes exigem permissoes `collaborators.*` e CSRF; leitura exige `collaborators.read` e aplica escopo por departamento para perfis nao privilegiados.

---

## 12. Segurança no NestJS

Aplicar em `main.ts`:

```ts
app.use(helmet());
app.enableCors({
  origin: process.env.CORS_ORIGIN,
  credentials: true,
});
```

Também configurar:

- `ValidationPipe` global;
- filtro global de exceções;
- rate limiting;
- Swagger protegido/desabilitado em produção;
- logs estruturados;
- request ID;
- guards globais;
- validação de DTOs;
- auditoria.

---

## 13. Segurança no EasyPanel

- MySQL sem porta pública desnecessária.
- phpMyAdmin protegido.
- HTTPS obrigatório.
- variáveis sensíveis no painel, não no repositório.
- backups automáticos.
- logs sem senhas/tokens/documentos.
- volumes privados para documentos.
- ambientes separados para desenvolvimento, homologação e produção.

---

## 14. Checklist geral de segurança

- [ ] `.env` removido do repositório/pacote.
- [ ] Segredos rotacionados.
- [ ] MySQL sem exposição pública indevida.
- [ ] phpMyAdmin protegido.
- [ ] Autenticação centralizada no NestJS.
- [ ] Tokens sensíveis fora do `localStorage`.
- [ ] Cookies com `HttpOnly`, `Secure` e `SameSite`.
- [x] Smoke operacional disponivel para validar login/refresh/logout e politica do refresh cookie em HTTPS.
- [x] Playwright opt-in disponivel para validar login/Home/logout real sem trace/video/screenshot.
- [ ] RBAC implementado.
- [ ] Escopo de recurso validado no backend.
- [ ] Proteção contra IDOR.
- [ ] Validação server-side em todas as mutações.
- [ ] Upload com extensão, MIME, assinatura e tamanho.
- [x] ClamAV/scanner configuravel no fluxo de upload NestJS.
- [ ] Validacao operacional de ClamAV/EICAR e `SAMA_UPLOAD_SCAN_MODE=strict` no host.
- [ ] Storage privado.
- [ ] Download apenas via API protegida.
- [ ] Auditoria em eventos críticos.
- [ ] Rate limiting em login, tokens públicos e downloads.
- [ ] CORS restrito.
- [ ] CSP configurada gradualmente.
- [ ] Erros sem detalhes internos.
- [ ] Logs sem dados sensíveis.
- [ ] Backups testados.
