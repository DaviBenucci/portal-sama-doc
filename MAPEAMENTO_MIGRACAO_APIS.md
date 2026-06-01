# [PARCIAL] Mapeamento de migração dos endpoints PHP para NestJS

## 1. Objetivo

Mapear os principais endpoints e arquivos PHP atuais para módulos, controllers e rotas da nova API NestJS em `/api-v2`.

Convenção:

```txt
/api     -> PHP atual, mantido temporariamente
/api-v2  -> NestJS novo
```

---

## 2. Padrão de resposta da nova API

Resposta de sucesso:

```json
{
  "data": {},
  "meta": {},
  "requestId": "uuid"
}
```

Erro padronizado:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Dados inválidos.",
    "details": []
  },
  "requestId": "uuid"
}
```

---

## 3. Autenticação

### Arquivos atuais

- `index.html`
- `Login.js`
- `auth.js`
- `api/auth.php`
- `api/storage.php?action=auth_login`
- `api/storage.php?action=auth_session_status`
- `api/storage.php?action=auth_logout`

### Módulo alvo

```txt
AuthModule
UsersModule
```

### Rotas alvo

| Atual | Nova rota | Método | Finalidade |
|---|---|---|---|
| `api/storage.php?action=auth_login` | `/api-v2/auth/login` | POST | Autenticar usuário. |
| `api/storage.php?action=auth_session_status` | `/api-v2/auth/me` | GET | Retornar usuário atual e permissões. |
| `api/storage.php?action=auth_logout` | `/api-v2/auth/logout` | POST | Encerrar sessão/revogar refresh token. |
| `api/storage.php?action=auth_forgot_password` | `/api-v2/auth/forgot-password` | POST | Recuperação de senha. |
| Preferencia visual de intro | `/api-v2/me/preferences/intro` | PATCH | Marcar animacao de boas-vindas como vista. |

---

## 4. Usuários, roles e permissões

### Arquivos atuais

- `DEV/dev.html`
- `DEV/dev-colaborador.html`
- `DEV/dev-novo-colaborador.html`
- `DEV/dev-data.js`
- `api/storage.php`
- `api/auth.php`

### Módulos alvo

```txt
UsersModule
RolesModule
PermissionsModule
AuditModule
```

### Rotas alvo

```txt
GET    /api-v2/users
GET    /api-v2/users/:id
POST   /api-v2/users
PATCH  /api-v2/users/:id
PATCH  /api-v2/users/:id/status
PUT    /api-v2/users/:id/roles

GET    /api-v2/roles
POST   /api-v2/roles
PATCH  /api-v2/roles/:id

GET    /api-v2/permissions
POST   /api-v2/permissions
PATCH  /api-v2/permissions/:id
PATCH  /api-v2/roles/:id/permissions
```

Status em 2026-05-12 16:13: `UsersModule` implementa listagem, detalhe, criação, atualização, status e troca de roles com permissões granulares, CSRF e auditoria. `RolesModule` implementa listagem, detalhe, criação, atualização e troca de permissões da role. `PermissionsModule` implementa listagem, criação e atualização do catálogo, com proteção para não renomear permissões padrão. Ainda faltam validação MySQL/homologação, migrations/seed atualizado, migração de usuários legados e integração frontend.

Status em 2026-05-15 14:13: `portal-sama-web` passou a ter `DevAdminPage.tsx` em `/dev`, consumindo `GET/POST/PATCH /api-v2/users`, `PATCH /api-v2/users/:id/status`, `PUT /api-v2/users/:id/roles`, `GET/POST/PATCH /api-v2/roles`, `PUT /api-v2/roles/:id/permissions` e `GET/POST/PATCH /api-v2/permissions` com TanStack Query, Zod e CSRF via client centralizado. A validacao integrada com API v2, MySQL/seed/backfill reais, usuarios reais, migracao legada e Playwright segue pendente.

---

## 5. Clientes e colaboradores

### Arquivos atuais

- `Client/clientes.html`
- `Client/painel.html`
- `Client/visao-geral-clientes.html`
- `Client/visao-geral-colaboradores.html`
- `Client/visao-colaborador.html`
- `DEV/dev-novo-cliente.html`
- `DEV/dev-novo-colaborador.html`
- `api/storage.php`

### Módulos alvo

```txt
ClientsModule
CollaboratorsModule
DepartmentsModule
ManagersModule
TransfersModule
CalendarModule
AuditModule
```

### Rotas alvo

```txt
GET    /api-v2/clients
GET    /api-v2/clients/:id
POST   /api-v2/clients
PATCH  /api-v2/clients/:id
DELETE /api-v2/clients/:id
GET    /api-v2/clients/:id/dashboard
GET    /api-v2/clients/:id/collaborators
GET    /api-v2/clients/:id/documents

GET    /api-v2/collaborators
GET    /api-v2/collaborators/:id
POST   /api-v2/collaborators
PATCH  /api-v2/collaborators/:id
DELETE /api-v2/collaborators/:id

GET    /api-v2/transfers
GET    /api-v2/transfers/dashboard
POST   /api-v2/transfers
POST   /api-v2/transfers/return
POST   /api-v2/transfers/:id/return

GET    /api-v2/calendar/config
POST   /api-v2/calendar/config
GET    /api-v2/calendar/month
GET    /api-v2/calendar/day
DELETE /api-v2/calendar/entries/:id

GET    /api-v2/departments/fiscal/workspace
POST   /api-v2/departments/fiscal/workspace/cycle-cell
PATCH  /api-v2/departments/fiscal/workspace/cell-status
POST   /api-v2/integrations/acessorias/deliveries/apply-to-fiscal

GET    /api-v2/managers/history
GET    /api-v2/managers/history/:companyId/timeline
```

Status em 2026-05-13 14:43: `ClientsModule` inicial foi criado em `portal-sama-api/src/modules/clients/` com listagem, detalhe, criacao, atualizacao, arquivamento logico e dashboard por cliente. `CollaboratorsModule` inicial foi criado em `portal-sama-api/src/modules/collaborators/` com listagem, detalhe, criacao, atualizacao e arquivamento logico de colaboradores internos baseados em `User`, permissoes `collaborators.*`, CSRF, auditoria e escopo por departamento. O modelo `Client` foi expandido com campos operacionais usados pelas telas legadas; o modelo `User` ganhou campos funcionais de colaborador (`position`, `phone`, `extension`, `metadata`, `archivedAt`, `archivedById`). As migrations `20260513113000_expand_clients_profile` e `20260513124500_add_collaborator_profile_to_users` foram aplicadas no MySQL local apos resolver o baseline Prisma. Atualizacao em 2026-05-15 08:59: `/clientes` e `/clientes/:clientId/painel` passaram a ter telas React iniciais consumindo `/api-v2/clients`. Atualizacao em 2026-05-18 15:58: `/departamentos/clientes` passou a ter tela React inicial consumindo `/api-v2/clients` para consulta departamental; atribuicao/transferencia segue pendente ate existir contrato backend seguro. Ainda faltam backfill de clientes/colaboradores, vinculos cliente-usuario/gestor/departamento, carteira/transferencias, documentos completos do painel e validacao em homologacao/producao.

Status em 2026-05-21 09:34: `DepartmentsModule` foi iniciado em `portal-sama-api/src/modules/departments/` com `GET /api-v2/departments/fiscal/workspace`, `POST /api-v2/departments/fiscal/workspace/cycle-cell` e `PATCH /api-v2/departments/fiscal/workspace/cell-status`, substituindo inicialmente `load_board`, `cycle_cell` e `set_cell_status` de `api/fiscal_workspace.php`. A rota React `/departamentos/modelo` consome o contrato novo com grade mensal, resumo, filtros e carrossel de vencimentos. Atualizacao em 2026-06-01 11:30: `POST /api-v2/integrations/acessorias/deliveries/apply-to-fiscal` aplica baixas `DELIVERED` com mapeamento confirmado na planilha Fiscal, gravando status `ACESSORIAS` e divergencias abertas para casos inseguros. Ainda faltam validacao MySQL/homologacao, contrato real do Acessorias, backfill de responsaveis fiscais, revisao manual de divergencias e Playwright.

Status em 2026-05-19 12:16: `TransfersModule` foi iniciado em `portal-sama-api/src/modules/transfers/` com dashboard, criacao de sessao e retorno manual. As rotas usam JWT, permissoes `transfers.read/create/return`, CSRF em mutacoes, auditoria e escopo por ADMIN/DEV/MANAGER + departamento. O modelo `TransferSession` mapeia `sama_transfer_sessions`; os vinculos de carteira seguem em metadata de `Client`/`User` ate haver backfill/modelagem formal. Ainda faltam MySQL real, seed atualizado, tela React `/manager/transferencias` e Playwright.

Status em 2026-05-19 15:33: `portal-sama-web` passou a ter `ManagerTransfersPage.tsx` em `/manager/transferencias`, consumindo `GET /api-v2/transfers/dashboard`, `POST /api-v2/transfers` e `POST /api-v2/transfers/:id/return` via `transfers.service.ts`. A tela usa `transfers.read/create/return` apenas para UX, envia mutacoes com CSRF pelo client centralizado, nao usa `innerHTML` e nao persiste dados de carteira em storage do navegador. Ainda faltam validacao integrada com MySQL/homologacao, usuarios reais por perfil/departamento, seed `transfers.*`, backfill/modelagem de carteira e Playwright.

Status em 2026-05-19 15:53: `portal-sama-web` passou a ter `ManagerCollaboratorsPage.tsx` em `/manager/colaboradores`, consumindo `GET /api-v2/transfers/dashboard` para consulta de carteira por colaborador, empresas transferidas e sessoes relacionadas. A tela usa `transfers.read` apenas para UX, nao usa `innerHTML` e nao grava carteira em storage do navegador. Transferencias continuam operadas em `/manager/transferencias`.

Status em 2026-05-20: `portal-sama-web` passou a ter `ManagerDashboardPage.tsx` em `/manager`, tambem consumindo `GET /api-v2/transfers/dashboard` para uma primeira visao consolidada do gestor. A tela adiciona KPIs de colaboradores, empresas, transferencias ativas e sessoes abertas, filtro por departamento permitido, busca geral e atalhos operacionais para carteiras, vencimentos, historico e transferencias.

Status em 2026-05-20: `CalendarModule` foi iniciado em `portal-sama-api/src/modules/calendar/` com `GET/POST /api-v2/calendar/config`, `GET /api-v2/calendar/month`, `GET /api-v2/calendar/day` e `DELETE /api-v2/calendar/entries/:id`. O modulo mapeia as tabelas legadas `sama_calendar_rules`, `sama_calendar_rule_companies` e `sama_calendar_due_notified`, usa permissoes `calendar.read/manage`, CSRF nas mutacoes, auditoria e escopo por departamento. O frontend `/manager/calendario/config` passou a consumir o contrato novo para configurar vencimentos por empresa, recorrencia, topicos e coluna vinculada; `/manager` passou a remover vencimentos por `calendar.manage`. Ainda faltam validacao MySQL/homologacao, backfill real de departamentos/carteira, notificacoes de vencimento na API v2 e Playwright.

Status em 2026-05-20: `/manager` passou a consumir `GET /api-v2/calendar/month` e `DELETE /api-v2/calendar/entries/:id` via `calendar.service.ts`, adicionando mes de referencia, calendario resumido, proximos vencimentos filtrados por busca geral e remocao segura por permissao `calendar.manage`. Ainda faltam validacao com dados reais, Playwright e desligamento das actions PHP de calendario apos homologacao.

Status em 2026-05-20 11:20: `ManagersModule` foi iniciado em `portal-sama-api/src/modules/managers/` com `GET /api-v2/managers/history` e `GET /api-v2/managers/history/:companyId/timeline`. O modulo mapeia `sama_company_history_entries` e `sama_company_life_entries`, usa JWT, RBAC `manager_history.read`, escopo por ADMIN/DEV/MANAGER + departamento e retorna leitura consolidada de historico/vida da empresa. O frontend `/manager/historico` passou a consumir esses contratos com lista por departamento, busca, estatisticas e timeline; ainda faltam MySQL/homologacao, usuarios reais, Playwright e escrita/edicao auditavel.

Status em 2026-05-20 11:45: `ManagersModule` adicionou `POST /api-v2/managers/history`, `PATCH /api-v2/managers/history/:id` e `POST /api-v2/managers/company-life` para substituir inicialmente `history_append`, `history_update` e `company_life_save`. As mutacoes exigem JWT, `manager_history.write`, CSRF, papel ADMIN/DEV/MANAGER, escopo por departamento e auditoria; `/manager/historico` passou a expor formularios condicionados a `can_edit`. Ainda faltam validacao MySQL/homologacao, seed real da nova permissao, usuarios reais e Playwright.

Status em 2026-05-20 12:30: `ManagersModule` adicionou `GET /api-v2/managers/overview` para substituir inicialmente a action `overview`, retornando `kpis`, `items`, `online`, `offline`, departamentos permitidos e timeout de presenca. O modulo passou a mapear `sama_user_presence` como `UserPresence`; `/manager` consome o novo endpoint para o painel de presenca da equipe. Ainda faltam homologacao MySQL, dados reais, Playwright e desligamento controlado do PHP legado.

Status em 2026-05-19 17:56: `portal-sama-web` passou a ter `DevCollaboratorsPage.tsx` em `/dev/colaboradores`, consumindo `GET/PATCH/DELETE /api-v2/collaborators` para listagem, edicao administrativa e arquivamento logico, alem de `GET /api-v2/roles` para opcoes de perfis quando permitido. A tela usa `collaborators.read/update/delete` apenas para UX; mutacoes continuam com CSRF, RBAC, validacao server-side e auditoria no backend.

---

## 6. Documentos

Status em 2026-05-12 13:52: `DocumentsModule` inicial foi criado em `portal-sama-api/src/modules/documents/` com template, checklist/listagem por cliente, detalhe, upload autenticado, upload público por `PublicToken`, emissão/listagem/revogação administrativa de tokens públicos, download, resumo de pendências, requisitos customizados, revisao de status com `DocumentStatusHistory`, quarentena/scanner configuravel no upload e arquivamento lógico. Atualizacao em 2026-05-15 09:16: o painel React `/clientes/:clientId/painel` passou a consumir checklist/upload/download/revisao/requisitos de documentos em `ClientDocumentsPanel.tsx`. Atualizacao em 2026-05-18: `/onboarding/publico/documentos/:token` passou a consumir `POST /api-v2/documents/public-upload?token=...` para upload publico por token. Ainda faltam validacao operacional de ClamAV strict, migration/backfill do historico em banco real, validação de escopo com dados persistidos, checklist publico especifico e homologacao do frontend.

### Arquivos atuais

- `Client/painel.html`
- `Client/painel.js`
- `api/client_documents.php`
- `Onboarding/documentos-cliente.html`
- `api/onboarding.php`

### Módulo alvo

```txt
DocumentsModule
```

### Rotas alvo

```txt
GET    /api-v2/documents
GET    /api-v2/documents/templates
GET    /api-v2/documents/required-pending-summary
GET    /api-v2/documents/:id
POST   /api-v2/documents/custom-requirements
POST   /api-v2/documents/upload
GET    /api-v2/documents/:id/download
PATCH  /api-v2/documents/:id/status
DELETE /api-v2/documents/:id
GET    /api-v2/clients/:clientId/documents
```

### Segurança obrigatória

- autenticação;
- permissão granular;
- validação de escopo;
- storage privado;
- ClamAV/scanner configuravel, com modo `strict` validado no host antes de producao;
- auditoria.

---

## 7. Certificados digitais

### Arquivos atuais

- `DptClient/certificados-digitais.html`
- `DptClient/certificados-digitais.js`
- `api/certificados.php`
- `api/certificate_secret_lib.php`

### Módulo alvo

```txt
CertificatesModule
DocumentsModule
AuditModule
```

### Rotas alvo

```txt
GET    /api-v2/certificates
GET    /api-v2/certificates/:id
POST   /api-v2/certificates
PATCH  /api-v2/certificates/:id
DELETE /api-v2/certificates/:id
GET    /api-v2/certificates/:id/download
POST   /api-v2/certificates/:id/rotate-password
```

---

## 8. Legalização, propostas, contratos e assinatura

### Arquivos atuais

- `Legalizacao/legalizacao.html`
- `Legalizacao/proposta.html`
- `Legalizacao/contrato.html`
- `Legalizacao/assinatura.html`
- `api/legalizacao.php`
- `api/propostas.php`
- `api/legal_doc_service.php`
- `api/zapsign.php`

### Módulos alvo

```txt
LegalizationModule
ProposalsModule
ContractsModule
SignaturesModule
PublicLinksModule
DocumentsModule
AuditModule
```

### Rotas alvo

```txt
GET    /api-v2/legalization/processes
GET    /api-v2/legalization/processes/:id
POST   /api-v2/legalization/processes
PATCH  /api-v2/legalization/processes/:id/status
GET    /api-v2/legalization/templates
POST   /api-v2/legalization/templates
PATCH  /api-v2/legalization/templates/:id
DELETE /api-v2/legalization/templates/:id

GET    /api-v2/proposals
GET    /api-v2/proposals/:id
POST   /api-v2/proposals
PATCH  /api-v2/proposals/:id
POST   /api-v2/proposals/:id/send
POST   /api-v2/proposals/:id/approve
POST   /api-v2/proposals/:id/reject
GET    /api-v2/public/proposals/:token
POST   /api-v2/public/proposals/:token

GET    /api-v2/contracts
GET    /api-v2/contracts/:id
POST   /api-v2/contracts
PATCH  /api-v2/contracts/:id
POST   /api-v2/contracts/:id/generate
POST   /api-v2/contracts/:id/render-pdf
GET    /api-v2/contracts/:id/pdf
POST   /api-v2/contracts/:id/pdf
POST   /api-v2/contracts/:id/send-signature

GET    /api-v2/public/signatures/:token
POST   /api-v2/public/signatures/:token/sign
```

Status em 2026-05-13 08:57: `ProposalsModule` foi iniciado em `portal-sama-api/src/modules/proposals/` com listagem, detalhe, criação, atualização, envio por token público opaco, aprovação/rejeição interna e fluxo público por token. As mutações internas exigem JWT, permissões granulares, CSRF e auditoria. O token público usa `PublicToken.tokenHash` SHA-256, expiração, revogação e escopo `proposals/proposal/proposal`. Ainda faltam validação com MySQL/homologação, backfill de `sama_propostas_v1`, compatibilidade com tokens legados e integração frontend.

Status em 2026-05-15 15:06: `portal-sama-web` passou a consumir `ProposalsModule` em `/legalizacao/propostas`, `/legalizacao/propostas/:id` e `/onboarding/publico/proposta/:token`, cobrindo listagem, detalhe, criacao, edicao, envio por link publico, aprovacao/rejeicao interna e resposta publica do cliente. Ainda faltam validacao com API v2/MySQL/backfill reais, usuarios/permissoes reais, compatibilidade com tokens legados, homologacao HTTPS e Playwright.

Status em 2026-05-13 09:34: `ContractsModule` foi iniciado em `portal-sama-api/src/modules/contracts/` com listagem, detalhe, criação, atualização, geração de snapshot HTML, envio para assinatura por token público opaco e assinatura pública por token. As mutações internas exigem JWT, permissões granulares, CSRF e auditoria. O token público usa `PublicToken.tokenHash` SHA-256, expiração, revogação e escopo `signatures/contract/signature`. Ainda faltam baseline/migrations em MySQL real, backfill de contratos legados, importação/renderização PDF segura e integração frontend.

Status em 2026-05-14 13:42: `ContractsModule` recebeu importacao/download seguro de PDF em `POST /api-v2/contracts/:id/pdf` e `GET /api-v2/contracts/:id/pdf`. O upload valida PDF por extensao, MIME, assinatura, tamanho e marcadores ativos conhecidos, grava em storage privado configuravel e audita hash/tamanho sem expor `storageKey`. Ainda faltam renderizacao HTML para PDF em sandbox, MySQL/storage reais, backfill legado e integracao frontend.

Status em 2026-05-14 13:54: `ContractsModule` recebeu renderizacao server-side em `POST /api-v2/contracts/:id/render-pdf`, usando Legal Doc Service externo configuravel e desligado por padrao. O PDF retornado e validado, persistido em storage privado e auditado sem expor chave interna. Ainda faltam configurar/homologar o servico real, validar fidelidade visual, backfill legado e integracao frontend.

Status em 2026-05-15 16:13: `portal-sama-web` passou a ter `ContractPage.tsx` em `/legalizacao/contratos` e `/legalizacao/contratos/:id`, consumindo `GET/POST/PATCH /api-v2/contracts`, `POST /api-v2/contracts/:id/generate`, `POST /api-v2/contracts/:id/render-pdf`, `POST/GET /api-v2/contracts/:id/pdf` e `POST /api-v2/contracts/:id/send-signature` com TanStack Query, Zod, upload multipart e CSRF via client centralizado. `PublicSignaturePage.tsx` passou a consumir `GET/POST /api-v2/public/signatures/:token`. Ainda faltam validar com MySQL/storage/Legal Doc Service reais, tokens legados, PDF assinado real e Playwright.

Status em 2026-05-13 15:59: `LegalizationModule` foi iniciado em `portal-sama-api/src/modules/legalization/` com processos internos em `GET/POST /api-v2/legalization/processes`, `GET/PATCH/DELETE /api-v2/legalization/processes/:id`, `PATCH /api-v2/legalization/processes/:id/status` e `GET /api-v2/legalization/processes/:id/timeline`. As mutações exigem JWT, permissões `legalization.*`, CSRF, escopo por responsavel/criador/departamento e auditoria.

Status em 2026-05-13 16:16: templates de cabecalho/rodape foram adicionados em `GET/POST /api-v2/legalization/templates`, `PATCH /api-v2/legalization/templates/:id` e `DELETE /api-v2/legalization/templates/:id`, com `legalization.read` para leitura, `legalization.templates` para mutacoes, CSRF, auditoria, soft delete e rejeicao de HTML inseguro. Ainda faltam backfill legado, importação/renderização PDF segura, integração com propostas/contratos em fluxo completo e frontend.

---

## 9. Onboarding

### Arquivos atuais

- `Onboarding/processo.html`
- `Onboarding/entrada-cliente.html`
- `Onboarding/documentos-cliente.html`
- `Onboarding/proposta-cliente.html`
- `api/onboarding.php`

### Módulos alvo

```txt
OnboardingModule
DocumentsModule
ProposalsModule
PublicLinksModule
AuditModule
```

### Rotas alvo

```txt
GET    /api-v2/onboarding/processes
GET    /api-v2/onboarding/processes/:id
POST   /api-v2/onboarding/processes
PATCH  /api-v2/onboarding/processes/:id
PATCH  /api-v2/onboarding/processes/:id/status
GET    /api-v2/onboarding/processes/:id/timeline
POST   /api-v2/onboarding/processes/:id/documents

GET    /api-v2/public/onboarding/documents/:token
POST   /api-v2/public/onboarding/documents/:token/upload
GET    /api-v2/public/onboarding/proposals/:token
POST   /api-v2/public/onboarding/proposals/:token/accept
```

Status em 2026-05-13 15:18: `OnboardingModule` foi criado em `portal-sama-api/src/modules/onboarding/` com rotas internas `GET/POST /api-v2/onboarding/processes`, `GET/PATCH/DELETE /api-v2/onboarding/processes/:id`, `PATCH /api-v2/onboarding/processes/:id/status`, `GET /api-v2/onboarding/processes/:id/timeline` e `POST /api-v2/onboarding/processes/:id/documents`. As mutacoes exigem CSRF; leitura e escrita usam permissoes `onboarding.*`; o service aplica escopo por responsavel, criador, departamento e papeis privilegiados. Fluxos publicos especificos `/api-v2/public/onboarding/*`, notificacoes, maquina de estados completa e backfill legado seguem pendentes.

Status em 2026-05-15 13:51: `portal-sama-web` passou a ter `OnboardingProcessesPage.tsx` em `/onboarding/processos`, consumindo listagem, criacao, atualizacao, arquivamento, status/timeline e vinculo leve de documentos em `/api-v2/onboarding/processes`. A validacao integrada com API v2, MySQL/backfill, usuarios/permissoes reais, fluxos publicos especificos e Playwright segue pendente.

Status em 2026-05-18: `portal-sama-web` passou a ter `PublicDocumentsPage.tsx` em `/onboarding/publico/documentos/:token`, consumindo o upload publico ja existente em `DocumentsModule`. O endpoint alvo especifico `GET /api-v2/public/onboarding/documents/:token` e o checklist publico minimo seguem pendentes.

---

## 10. Solicitação de acesso e TI

### Arquivos atuais

- `SolicitacaoAcesso/solicitacao-acesso.html`
- `SolicitacaoAcesso/acesso.js`
- `SolicitacaoAcesso/enviar_acesso.php`
- `SolicitacaoAcesso/enviar_codigo.php`
- `SolicitacaoAcesso/verificar_codigo.php`
- `api/access_requests.php`
- `TI/acesso-ti.html`
- `TI/ti.js`

### Módulos alvo

```txt
AccessRequestsModule
TiModule
NotificationsModule
AuditModule
```

### Rotas alvo

```txt
POST   /api-v2/access-requests
GET    /api-v2/access-requests
GET    /api-v2/access-requests/my/latest
GET    /api-v2/access-requests/manager/approvals
GET    /api-v2/access-requests/:id
POST   /api-v2/access-requests/:id/approve
POST   /api-v2/access-requests/:id/reject
PATCH  /api-v2/access-requests/:id/status
GET    /api-v2/ti/access-requests
GET    /api-v2/notifications
GET    /api-v2/notifications/stream
GET    /api-v2/notifications/push/public-key
POST   /api-v2/notifications
POST   /api-v2/notifications/push/subscribe
POST   /api-v2/notifications/push/unsubscribe
POST   /api-v2/notifications/push/test
POST   /api-v2/notifications/:id/ack
POST   /api-v2/notifications/:id/alert-ack
POST   /api-v2/notifications/clear-unread
```

Status em 2026-05-14 08:30: `AccessRequestsModule` agora emite notificacoes automaticas via `NotificationsModule` na criacao, aprovacao e rejeicao, direcionando gestor, solicitante e TI conforme o fluxo. Ainda faltam backfill de `sama_access_requests`, validacao com usuarios reais, frontend e homologacao/producao.

Status em 2026-05-15 09:44: `portal-sama-web` passou a ter `AccessRequestPage.tsx` em `/solicitacao-acesso`, consumindo `/api-v2/access-requests`, `/my/latest`, `/manager/approvals`, `/approve` e `/reject` com TanStack Query, Zod e CSRF via client centralizado. A validacao integrada com API v2, MySQL/backfill, usuarios reais, escopo por gestor/departamento/TI e Playwright segue pendente.

Status em 2026-05-18 15:35: `portal-sama-web` passou a ter `TiAccessPage.tsx` em `/ti/acessos`, consumindo `/api-v2/access-requests` e `/api-v2/access-requests/manager/approvals` para fila operacional, historico, filtros e decisoes por permissao. A base editavel de credenciais/acessos internos da TI nao foi migrada; segue pendente ate existir `TiModule`/cofre seguro com criptografia/vault, auditoria de leitura e politica de retencao.

Status em 2026-05-14 08:40: `NotificationsModule` foi estendido com `GET /api-v2/notifications/stream`, SSE autenticado com snapshot inicial, ping, eventos live de criacao/atualizacao/limpeza, `last_id`/`lastId` e catch-up periodico pelo MySQL. O service aplica escopo por usuario/departamento, filtros de query e autoevento tambem no stream. Ainda faltam validar navegador/frontend, definir broker/event bus se updates instantaneos multi-instancia forem exigidos, emissores nos demais modulos, backfill/retencao em homologacao/producao e frontend.

Status em 2026-05-14 09:35: `NotificationsModule` ganhou base Web Push com VAPID, tabela `browser_push_subscriptions`, rotas `/api-v2/notifications/push/*` e despacho em segundo plano ao criar notificacoes. O legado ganhou `sama-push-sw.js` e assinatura Push API em `global.js` quando houver permissao do navegador e bearer API v2. Ainda faltam VAPID real, teste HTTPS/homologacao com portal fechado, emissores dos demais modulos e integracao React futura.

Status em 2026-05-14 10:39: `NotificationsModule` ganhou `POST /api-v2/notifications/push/test`, protegido por JWT `notifications.read` e CSRF, para criar notificacao de teste ao usuario autenticado e retornar resumo de despacho Web Push. O painel legado ganhou botao `Testar` apos opt-in, reduzindo a pendencia manual a validacao em HTTPS/homologacao com VAPID real e navegador suportado.

Status em 2026-05-15 10:24: `portal-sama-web` passou a ter `NotificationsPage.tsx` em `/notificacoes`, consumindo `GET/POST /api-v2/notifications`, `POST /api-v2/notifications/clear-unread`, `POST /api-v2/notifications/:id/ack`, `POST /api-v2/notifications/:id/alert-ack`, `GET /api-v2/notifications/push/public-key` e `POST /api-v2/notifications/push/test` com TanStack Query, Zod e CSRF via client centralizado.

Status em 2026-05-20 16:21: `/notificacoes` passou a abrir `GET /api-v2/notifications/stream` por `fetch` autenticado com bearer em memoria, sem token em query string, e adicionou assinatura/cancelamento Push API pelo React usando `/push/subscribe` e `/push/unsubscribe`. O service worker tambem foi publicado em `portal-sama-web/public/sama-push-sw.js`. Ainda faltam validacao com MySQL/VAPID reais, navegadores HTTPS, portal fechado, unsubscribe real e Playwright.

---

## 11. Auditoria

### Arquivos atuais

- `Auditoria/autoria.html`
- `Auditoria/autoria.js`
- `api/storage.php?action=audit_*`

### Módulo alvo

```txt
AuditModule
```

### Rotas alvo

```txt
GET /api-v2/audit/logs
GET /api-v2/audit/logs/:id
GET /api-v2/audit/entities/:entityType/:entityId
GET /api-v2/audit/users/:userId
```

Status em 2026-05-15 10:05: `portal-sama-web` passou a ter `AuditPage.tsx` em `/auditoria`, consumindo `GET /api-v2/audit/logs` e `GET /api-v2/audit/logs/:id` com filtros por acao, modulo, entidade, usuario e periodo, paginacao e detalhe de metadados sem `innerHTML`. Ainda faltam validar com MySQL/homologacao, seed/permissao `audit.read`, Playwright, politica de retencao/exportacao e contrato proprio para backup/restauracao.

---

## 12. Contábil / Integra-AI

### Arquivos atuais

- `Contabil/integra-ai.html`
- `Contabil/integra-ai.js`
- `api/integra_ai.php`
- `api/integra_ai_*`

### Módulos alvo

```txt
AccountingModule
IntegraAiModule
JobsModule
AuditModule
```

### Rotas alvo

```txt
GET    /api-v2/accounting/integra-ai/workspaces
POST   /api-v2/accounting/integra-ai/import
POST   /api-v2/accounting/integra-ai/reconcile
POST   /api-v2/accounting/integra-ai/export
GET    /api-v2/accounting/integra-ai/jobs/:id
```

Status em 2026-05-21 10:33: `AccountingModule` foi iniciado em `portal-sama-api/src/modules/accounting/` com `GET /api-v2/accounting/integra-ai/workspaces` e `GET /api-v2/accounting/integra-ai/jobs/:id`, cobrindo somente leitura. As rotas usam JWT, RBAC `accounting.integra_ai.read`, escopo contabil, leitura parametrizada das tabelas legadas `sama_integra_ai_*`, sanitizacao de caminhos privados e auditoria ao abrir detalhe de job. O frontend `/contabil/integra-ai` passou a consumir esses contratos para workspace, jobs recentes e preview seguro. Importacao, parser, salvamento de etapas, regras, geracao e download TXT seguem no PHP legado ate a camada segura de mutacoes.

---

## 13. Priorização da migração

| Prioridade | Módulo | Motivo |
|---|---|---|
| 1 | `AuthModule` | Base de segurança. |
| 2 | `Users/Roles/Permissions` | Permissões antes dos módulos sensíveis. |
| 3 | `AuditModule` | Rastreabilidade desde o início. |
| 4 | `DocumentsModule` | Documentos empresariais sensíveis. |
| 5 | `CertificatesModule` | Certificados digitais têm criticidade máxima. |
| 6 | `Contracts/Proposals/Signatures` | Valor legal e comercial. |
| 7 | `AccessRequestsModule` | Controle de acesso. |
| 8 | `Clients/Collaborators` | Base operacional. |
| 9 | `OnboardingModule` | Tokens públicos e documentos. |
| 10 | `Managers/Accounting/Integra-AI` | Operação e contabilidade. |
