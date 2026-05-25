# Inventário de Endpoints - Portal Sama

Status: inventário em andamento. Inventário baseado nos arquivos PHP reais, nas chamadas `fetch` em HTML/JS, em `docs/MAPEAMENTO_MIGRACAO_APIS.md` e nos endpoints já criados em `portal-sama-api`. Métodos exatos por action PHP ainda devem ser validados em cada controller legado.

## Endpoint novo: `/api-v2/health`

- **Arquivo:** `portal-sama-api/src/modules/health/health.controller.ts`
- **Método provável:** GET.
- **Finalidade:** Healthcheck básico da API NestJS para validação local/EasyPanel.
- **Página/tela relacionada:** Deploy/observabilidade; sem tela de negócio.
- **Novo endpoint proposto:** Implementado como `/api-v2/health`.
- **Módulo NestJS alvo:** `HealthModule`.
- **Status de migração:** Implementado.
- **Riscos de segurança:** Não deve expor segredos, DSN, paths sensíveis ou stack trace. Estado de banco é apenas `configured/not_checked` neste estágio.
- **Testes necessários:** E2E 200, sem exposição de variáveis sensíveis, validação de health em deploy real.
- **Observações:** Teste e2e passou em 2026-05-08; checagem real de MySQL/storage segue ponto a validar.

## Endpoint novo: `/api-v2/auth/login`

- **Arquivo:** `portal-sama-api/src/modules/auth/auth.controller.ts`
- **Método provável:** POST.
- **Finalidade:** Autenticar usuário por `username`/`password`, emitir access JWT e refresh token em cookie HttpOnly.
- **Página/tela relacionada:** futura tela React de login; legado `Login.js` ainda usa `api/storage.php?action=auth_login`.
- **Novo endpoint proposto:** Implementado como `/api-v2/auth/login`.
- **Módulo NestJS alvo:** `AuthModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** brute force, enumeração de usuário, exposição de token e CSRF em fluxos com cookie. Throttling por rota e CSRF double-submit foram adicionados.
- **Testes necessários:** login válido/inválido com banco de teste, rate limit, cookie HttpOnly/Secure/SameSite, auditoria sem senha/token, integração frontend sem `localStorage`.
- **Observações:** Unitários do `AuthService` e e2e de rejeição sem CSRF passaram em 2026-05-08.

## Endpoint novo: `/api-v2/auth/csrf`

- **Arquivo:** `portal-sama-api/src/modules/auth/auth.controller.ts`, `portal-sama-api/src/modules/auth/csrf.service.ts`
- **Método provável:** GET.
- **Finalidade:** Emitir token CSRF assinado e cookie CSRF para mutações de autenticação.
- **Página/tela relacionada:** futura tela React de login; legado ainda não usa este endpoint.
- **Novo endpoint proposto:** Implementado como `/api-v2/auth/csrf`.
- **Módulo NestJS alvo:** `AuthModule`.
- **Status de migração:** Implementado parcialmente; falta integração frontend/navegador real.
- **Riscos de segurança:** token deve ser reenviado por cabeçalho customizado e não deve ser tratado como segredo persistente. XSS ainda compromete CSRF, portanto CSP/sanitização continuam necessárias.
- **Testes necessários:** emissão de cookie/token, rejeição de token ausente/inválido, validação de Origin/Referer, fluxo browser completo.
- **Observações:** Unitários do `CsrfService` e e2e de emissão passaram em 2026-05-08.

## Endpoint novo: `/api-v2/auth/refresh`

- **Arquivo:** `portal-sama-api/src/modules/auth/auth.controller.ts`
- **Método provável:** POST.
- **Finalidade:** Rotacionar refresh token em cookie e emitir novo access JWT.
- **Página/tela relacionada:** futura sessão React.
- **Novo endpoint proposto:** Implementado como `/api-v2/auth/refresh`.
- **Módulo NestJS alvo:** `AuthModule`.
- **Status de migração:** Implementado parcialmente; dependente de MySQL real e integração frontend.
- **Riscos de segurança:** replay de refresh token, CSRF, sessão roubada. O refresh é opaco, armazenado como HMAC-SHA-256, revogado na rotação e agora exige CSRF.
- **Testes necessários:** rotação, token expirado/revogado, cookie ausente, CSRF válido/inválido, auditoria.
- **Observações:** Unitário de rotação passou em 2026-05-08.

## Endpoint novo: `/api-v2/auth/logout`

- **Arquivo:** `portal-sama-api/src/modules/auth/auth.controller.ts`
- **Método provável:** POST.
- **Finalidade:** Revogar refresh token atual e limpar cookie.
- **Página/tela relacionada:** futura sessão React.
- **Novo endpoint proposto:** Implementado como `/api-v2/auth/logout`.
- **Módulo NestJS alvo:** `AuthModule`.
- **Status de migração:** Implementado parcialmente; dependente de MySQL real e integração frontend.
- **Riscos de segurança:** inconsistência de revogação se banco indisponível; CSRF é obrigatório no controller.
- **Testes necessários:** logout com/sem cookie, cookie limpo, token revogado, CSRF válido/inválido.
- **Observações:** Sem integração frontend nesta etapa.

## Endpoint novo: `/api-v2/auth/me`

- **Arquivo:** `portal-sama-api/src/modules/auth/auth.controller.ts`
- **Método provável:** GET.
- **Finalidade:** Retornar usuário autenticado a partir do bearer access JWT.
- **Página/tela relacionada:** futura sessão React; legado `auth.js` ainda usa `api/storage.php?action=auth_session_status`.
- **Novo endpoint proposto:** Implementado como `/api-v2/auth/me`.
- **Módulo NestJS alvo:** `AuthModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** confiar em claims desatualizadas até expiração do access token; escopo real por recurso ainda depende dos módulos de usuários/permissões.
- **Testes necessários:** sem token 401, token inválido 401, token válido 200, expiração.
- **Observações:** E2E sem token passou em 2026-05-08.

## Endpoint novo: `/api-v2/auth/forgot-password`

- **Arquivo:** `portal-sama-api/src/modules/auth/auth.controller.ts`
- **Método provável:** POST.
- **Finalidade:** Resposta segura e genérica para recuperação de senha, alinhada ao legado que orienta contato com usuário MASTER.
- **Página/tela relacionada:** futura tela React de login.
- **Novo endpoint proposto:** Implementado como `/api-v2/auth/forgot-password`.
- **Módulo NestJS alvo:** `AuthModule`.
- **Status de migração:** Implementado parcialmente; fluxo real de recuperação ainda não definido.
- **Riscos de segurança:** enumeração de usuário se futuramente aceitar identificador; deve manter resposta genérica e rate limit.
- **Testes necessários:** resposta genérica, rate limit e ausência de enumeração.
- **Observações:** Sem envio de e-mail nesta etapa.

## Endpoint novo: `/api-v2/audit/logs`

- **Arquivo:** `portal-sama-api/src/modules/audit/audit.controller.ts`, `portal-sama-api/src/modules/audit/audit.service.ts`
- **Método provável:** GET.
- **Finalidade:** Listar logs de auditoria com filtros por `action`, `module`, `entityType`, `entityId`, `userId`, período, `skip` e `take`.
- **Página/tela relacionada:** futura rota React `/auditoria`; legado `Auditoria/autoria.js` ainda usa `api/storage.php?action=audit_list`.
- **Novo endpoint proposto:** Implementado como `/api-v2/audit/logs`.
- **Módulo NestJS alvo:** `AuditModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** exposição de logs sensíveis, enumeração de eventos e acesso por usuário sem permissão. Endpoint exige JWT e permissão `audit.read`.
- **Testes necessários:** listagem com MySQL real, filtros, paginação, usuário com `audit.read` recebe 200, usuário sem permissão recebe 403, sem token recebe 401, metadados sensíveis mascarados.
- **Observações:** E2E sem token e com token sem `audit.read` passou em 2026-05-08; unitário do `AuditService` passou para listagem e mascaramento.

## Endpoint novo: `/api-v2/audit/logs/:id`

- **Arquivo:** `portal-sama-api/src/modules/audit/audit.controller.ts`, `portal-sama-api/src/modules/audit/audit.service.ts`
- **Método provável:** GET.
- **Finalidade:** Consultar um evento de auditoria específico.
- **Página/tela relacionada:** futura rota React `/auditoria`.
- **Novo endpoint proposto:** Implementado como `/api-v2/audit/logs/:id`.
- **Módulo NestJS alvo:** `AuditModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** exposição de detalhe de evento para usuário sem permissão; endpoint exige JWT e `audit.read`.
- **Testes necessários:** 200 com permissão e registro existente, 404 para ID inexistente, 401 sem token e 403 sem `audit.read`.
- **Observações:** Unitário cobre 404; e2e de permissão foi feito em `/audit/logs`.

## Endpoint novo: `/api-v2/users`

- **Arquivo:** `portal-sama-api/src/modules/users/users.controller.ts`, `portal-sama-api/src/modules/users/users.service.ts`
- **Método provável:** GET e POST.
- **Finalidade:** Listar usuários administrativos com filtros por busca, username, status, departamento principal, role, `skip` e `take`; criar usuário administrativo com senha hasheada e roles validadas.
- **Página/tela relacionada:** futura rota React `/dev`; legado `DEV/dev-main.js` ainda usa `api/storage.php?action=admin_users_list`.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/users` e `POST /api-v2/users`.
- **Módulo NestJS alvo:** `UsersModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** exposição de contas, roles e permissões; criação indevida de usuário. GET exige JWT e `users.read`; POST exige JWT, `users.create`, CSRF, hash `bcrypt`, roles existentes e auditoria sem senha/hash.
- **Testes necessários:** listagem/criação com MySQL real, filtros, paginação, usuário com permissão recebe 200/201, usuário sem permissão recebe 403, sem token recebe 401, POST sem CSRF recebe 403.
- **Observações:** E2E sem token e com token sem `users.read` passou em 2026-05-08; unitário cobre formato da lista e ausência de `passwordHash`. Em 2026-05-12 15:25, unitário cobre criação com hash/roles/auditoria e e2e cobre POST sem CSRF.

## Endpoint novo: `/api-v2/users/:id`

- **Arquivo:** `portal-sama-api/src/modules/users/users.controller.ts`, `portal-sama-api/src/modules/users/users.service.ts`
- **Método provável:** GET, PATCH; `PATCH /api-v2/users/:id/status`; `PUT /api-v2/users/:id/roles`.
- **Finalidade:** Consultar detalhe administrativo de um usuário sem expor senha hash; atualizar dados cadastrais/senha; alterar status; substituir roles.
- **Página/tela relacionada:** futura rota React `/dev/colaboradores/:id`.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/users/:id`, `PATCH /api-v2/users/:id`, `PATCH /api-v2/users/:id/status` e `PUT /api-v2/users/:id/roles`.
- **Módulo NestJS alvo:** `UsersModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** enumeração de usuários e privilégios, alteração indevida de status/roles e auto-bloqueio. GET exige JWT e `users.read`; PATCH exige `users.update` e CSRF; status exige `users.status` e CSRF; roles exige `users.roles` e CSRF. O service bloqueia auto-inativação/bloqueio e remoção da própria role administrativa.
- **Testes necessários:** 200 com permissão, 404 para ID inexistente, 401 sem token, 403 sem permissão/CSRF, update real com MySQL, auditoria persistida.
- **Observações:** Unitário cobre 404; e2e de permissão foi feito em `/users`. Em 2026-05-12 15:25, unitários cobrem atualização/status/roles e e2e cobre mutações sem CSRF.

## Endpoint novo: `/api-v2/roles`

- **Arquivo:** `portal-sama-api/src/modules/roles/roles.controller.ts`, `portal-sama-api/src/modules/roles/roles.service.ts`
- **Método provável:** GET e POST.
- **Finalidade:** Listar roles com permissões associadas e contagem de usuários; criar role administrativa com permissões validadas.
- **Página/tela relacionada:** futura rota React `/dev` e telas de permissões administrativas.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/roles` e `POST /api-v2/roles`.
- **Módulo NestJS alvo:** `RolesModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** exposição/alteração da matriz de privilégios; GET exige JWT e `roles.read`; POST exige JWT, `roles.create`, CSRF, validação de permissões existentes e auditoria.
- **Testes necessários:** listagem/criação com MySQL real, filtros, paginação, 401 sem token, 403 sem permissão, POST sem CSRF e auditoria persistida.
- **Observações:** E2E sem token e com token sem `roles.read` passou em 2026-05-08. Em 2026-05-12 16:13, unitário cobre criação com permissões validadas/auditoria e e2e cobre POST sem CSRF.

## Endpoint novo: `/api-v2/roles/:id`

- **Arquivo:** `portal-sama-api/src/modules/roles/roles.controller.ts`, `portal-sama-api/src/modules/roles/roles.service.ts`
- **Método provável:** GET, PATCH e PUT em `/api-v2/roles/:id/permissions`.
- **Finalidade:** Consultar detalhe de uma role com permissões associadas; atualizar metadados; substituir permissões da role.
- **Página/tela relacionada:** futura rota React administrativa de perfis/permissões.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/roles/:id`, `PATCH /api-v2/roles/:id` e `PUT /api-v2/roles/:id/permissions`.
- **Módulo NestJS alvo:** `RolesModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** exposição e alteração indevida de privilégios; GET exige JWT e `roles.read`; PATCH exige `roles.update` e CSRF; PUT permissions exige `roles.permissions`, CSRF e auditoria. O service bloqueia remover `roles.permissions` da própria role do ator.
- **Testes necessários:** 200 com permissão, 404 para ID inexistente, 401 sem token, 403 sem permissão/CSRF, update real com MySQL, troca de permissões e auditoria persistida.
- **Observações:** Unitário cobre 404; e2e de permissão foi feito em `/roles`. Em 2026-05-12 16:13, unitários cobrem atualização/troca de permissões e e2e cobre mutações sem CSRF.

## Endpoint novo: `/api-v2/permissions`

- **Arquivo:** `portal-sama-api/src/modules/permissions/permissions.controller.ts`, `portal-sama-api/src/modules/permissions/permissions.service.ts`
- **Método provável:** GET, POST e PATCH em `/api-v2/permissions/:id`.
- **Finalidade:** Listar permissões disponíveis com filtros e paginação; criar permissões customizadas; atualizar descrição/chave de permissões customizadas.
- **Página/tela relacionada:** futura rota React administrativa de perfis/permissões.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/permissions`, `POST /api-v2/permissions` e `PATCH /api-v2/permissions/:id`.
- **Módulo NestJS alvo:** `PermissionsModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** enumeração/alteração do catálogo de privilégios; GET exige JWT e `permissions.read`; POST exige `permissions.create` e CSRF; PATCH exige `permissions.update` e CSRF. Chaves padrão do catálogo não podem ser renomeadas.
- **Testes necessários:** listagem/criação/atualização com MySQL real, filtros, usuário com permissão recebe 200/201, usuário sem permissão recebe 403, sem token recebe 401, mutações sem CSRF recebem 403.
- **Observações:** E2E sem token e com token sem `permissions.read` passou em 2026-05-08; seed inicial do catálogo foi criado em `portal-sama-api/prisma/seed.ts`. Em 2026-05-12 16:13, unitários cobrem criação/atualização/proteção de chave padrão e e2e cobre mutações sem CSRF. Em 2026-05-15 14:13, `/dev` passou a consumir `/api-v2/users`, `/api-v2/users/:id`, `/api-v2/roles`, `/api-v2/roles/:id`, `/api-v2/roles/:id/permissions` e `/api-v2/permissions` no React; validacao integrada com API v2/MySQL/seed/backfill reais e Playwright segue pendente.

## Endpoint novo: `/api-v2/clients`

- **Arquivo:** `portal-sama-api/src/modules/clients/clients.controller.ts`, `portal-sama-api/src/modules/clients/clients.service.ts`
- **Método provável:** GET, POST, GET/PATCH/DELETE em `/api-v2/clients/:id` e GET em `/api-v2/clients/:id/dashboard`.
- **Finalidade:** Listar, consultar, criar, atualizar, arquivar logicamente e resumir dados operacionais de clientes.
- **Página/tela relacionada:** `Client/clientes.html`, `Client/painel.html`, `Client/visao-geral-clientes.html`, `DptClient/clientdpto.html`, `DEV/dev-novo-cliente.html`, rotas React `/clientes`, `/clientes/:id/painel`, `/clientes/visao-geral`, `/dev/clientes/novo` e `/departamentos/clientes`.
- **Novo endpoint proposto:** Implementado como `/api-v2/clients`, `/api-v2/clients/:id` e `/api-v2/clients/:id/dashboard`.
- **Módulo NestJS alvo:** `ClientsModule`.
- **Status de migração:** Implementado parcialmente; conectado a primeiras telas React de clientes, painel, visao geral, cadastro DEV e consulta departamental, mas ainda sem backfill de clientes/colaboradores/vinculos legados.
- **Riscos de segurança:** enumeração de clientes, IDOR por cliente, alteração indevida de cadastro e arquivamento indevido. GET exige JWT e `clients.read`; mutações exigem `clients.create/update/delete`, CSRF e auditoria.
- **Testes necessários:** 200 com dados reais, filtros, unicidade de CNPJ, cliente sem vínculo, 401 sem token, 403 sem permissão/CSRF, auditoria persistida e validação de backfill.
- **Observações:** Em 2026-05-13 10:50, unitários do `ClientsService`, e2e de 401/403/CSRF e migration local passaram. Respostas mantêm aliases legados como `razao_social`, `nome_fantasia`, `regime_tributario`, `id_empresa`, `faz_parte_grupo` e `grupo_nome`. Em 2026-05-18 15:58, `DepartmentClientsPage.tsx` passou a consumir este endpoint para `/departamentos/clientes`; atribuicao/transferencia departamental segue pendente por exigir contrato backend proprio e auditoria.

## Endpoint novo: `/api-v2/collaborators`

- **Arquivo:** `portal-sama-api/src/modules/collaborators/collaborators.controller.ts`, `portal-sama-api/src/modules/collaborators/collaborators.service.ts`
- **MÃ©todo provÃ¡vel:** GET, POST, GET/PATCH/DELETE em `/api-v2/collaborators/:id`.
- **Finalidade:** Listar, consultar, criar, atualizar e arquivar logicamente colaboradores internos baseados em `User`.
- **PÃ¡gina/tela relacionada:** `Client/visao-geral-colaboradores.html`, `Client/visao-colaborador.html`, `DEV/dev-colaborador.html`, `DEV/dev-novo-colaborador.html`, `Manager/manager-colaborador.html` e rotas React `/colaboradores/visao-geral`, `/clientes/colaboradores/:id`, `/dev/colaboradores`, `/dev/colaboradores/novo`, `/manager/colaboradores`.
- **Novo endpoint proposto:** Implementado como `/api-v2/collaborators` e `/api-v2/collaborators/:id`.
- **MÃ³dulo NestJS alvo:** `CollaboratorsModule`.
- **Status de migraÃ§Ã£o:** Implementado parcialmente; conectado a primeiras telas React de colaboradores, detalhe, gestao DEV, cadastro DEV e carteira do gestor, mas ainda sem backfill de `sama_colaboradores`, carteira ou vÃ­nculos cliente-gestor.
- **Riscos de seguranÃ§a:** enumeraÃ§Ã£o de colaboradores, exposiÃ§Ã£o de dados funcionais, alteraÃ§Ã£o indevida de roles/status e arquivamento indevido. GET exige JWT e `collaborators.read`; mutaÃ§Ãµes exigem `collaborators.create/update/delete`, CSRF e auditoria. O service bloqueia role `CLIENT` neste mÃ³dulo e aplica escopo por departamento para perfis nÃ£o privilegiados.
- **Testes necessÃ¡rios:** 200 com dados reais, filtros por departamento/status/role, 401 sem token, 403 sem permissÃ£o/CSRF, auditoria persistida, unicidade de username/email, backfill e validaÃ§Ã£o de vÃ­nculos reais.
- **ObservaÃ§Ãµes:** Em 2026-05-13 14:43, unitÃ¡rios do `CollaboratorsService`, e2e de 401/403/CSRF, migration local e seed RBAC passaram. Respostas mantÃªm aliases legados como `usuario`, `nome`, `departamento`, `cargo`, `telefone`, `ramal` e `perfis`. Em 2026-05-19 17:56, `/dev/colaboradores` passou a consumir listagem, edicao e arquivamento por esse contrato.

## Endpoint novo: `/api-v2/documents`

- **Arquivo:** `portal-sama-api/src/modules/documents/documents.controller.ts`, `portal-sama-api/src/modules/documents/documents.service.ts`
- **Método provável:** GET.
- **Finalidade:** Listar documentos com filtros por cliente, tipo, status, departamento e busca textual.
- **Página/tela relacionada:** futura rota React do painel do cliente e telas internas de documentos.
- **Novo endpoint proposto:** Implementado como `/api-v2/documents`.
- **Módulo NestJS alvo:** `DocumentsModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** enumeração de documentos, IDOR por `clientId`, exposição de metadados sensíveis. Endpoint exige JWT e `documents.read`; `storageKey` não é retornado.
- **Testes necessários:** 200 com permissão e dados reais, 401 sem token, 403 sem `documents.read`, filtros com MySQL real e validação de escopo por cliente/departamento.
- **Observações:** E2E 401/403 passou em 2026-05-11.

## Endpoint novo: `/api-v2/clients/:clientId/documents`

- **Arquivo:** `portal-sama-api/src/modules/documents/documents.controller.ts`, `portal-sama-api/src/modules/documents/documents.service.ts`
- **Método provável:** GET.
- **Finalidade:** Listar checklist de requisitos documentais de um cliente específico, combinando template, requisitos customizados e upload mais recente por tipo.
- **Página/tela relacionada:** `Client/painel.html`, futuro painel React do cliente.
- **Novo endpoint proposto:** Implementado como `/api-v2/clients/:clientId/documents`.
- **Módulo NestJS alvo:** `DocumentsModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** IDOR por troca de `clientId`; endpoint exige JWT e `documents.read`, mas vínculo real cliente/gestor ainda precisa ser validado em banco real.
- **Testes necessários:** cliente sem vínculo, gestor sem vínculo, departamento sem permissão, 401/403 e listagem com dados reais.
- **Observações:** E2E 401/403 passou em 2026-05-11; em 2026-05-11 08:59 passou a retornar checklist compatível com o legado.

## Endpoint novo: `/api-v2/documents/templates`

- **Arquivo:** `portal-sama-api/src/modules/documents/documents.controller.ts`, `portal-sama-api/src/modules/documents/document-template.ts`
- **Método provável:** GET.
- **Finalidade:** Listar template de documentos obrigatórios/opcionais e extensões permitidas.
- **Página/tela relacionada:** `Client/painel.html`, `Onboarding/processo.html`.
- **Novo endpoint proposto:** Implementado como `/api-v2/documents/templates`.
- **Módulo NestJS alvo:** `DocumentsModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** enumeração de requisitos documentais; endpoint exige JWT e `documents.read`, com filtro por escopo de departamento para usuários departamentais.
- **Testes necessários:** 200 com permissão, filtro por departamento, 401 sem token, 403 sem `documents.read`.
- **Observações:** Unitário de template e e2e 401/403 passaram em 2026-05-11 08:59.

## Endpoint novo: `/api-v2/documents/required-pending-summary`

- **Arquivo:** `portal-sama-api/src/modules/documents/documents.controller.ts`, `portal-sama-api/src/modules/documents/documents.service.ts`
- **Método provável:** GET.
- **Finalidade:** Listar resumo de documentos obrigatórios pendentes por cliente.
- **Página/tela relacionada:** `Client/visao-geral-clientes.html`, `Client/painel.html`.
- **Novo endpoint proposto:** Implementado como `/api-v2/documents/required-pending-summary`.
- **Módulo NestJS alvo:** `DocumentsModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** enumeração de pendências de clientes; exige JWT, `documents.requirements` e papel gerencial.
- **Testes necessários:** 200 com dados reais, cliente sem vínculo, 401 sem token, 403 sem `documents.requirements`.
- **Observações:** Unitário de cálculo e e2e 401/403 passaram em 2026-05-11 08:59.

## Endpoint novo: `/api-v2/documents/custom-requirements`

- **Arquivo:** `portal-sama-api/src/modules/documents/documents.controller.ts`, `portal-sama-api/src/modules/documents/documents.service.ts`
- **Método provável:** POST JSON.
- **Finalidade:** Criar requisito documental customizado para um cliente.
- **Página/tela relacionada:** `Client/painel.html`.
- **Novo endpoint proposto:** Implementado como `/api-v2/documents/custom-requirements`.
- **Módulo NestJS alvo:** `DocumentsModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** criação indevida de obrigação documental; exige JWT, `documents.requirements`, papel gerencial, CSRF e auditoria.
- **Testes necessários:** 200/201 com MySQL real, 403 sem CSRF, 403 sem permissão, 404 cliente inexistente, auditoria persistida.
- **Observações:** Unitário de criação/auditoria e e2e sem CSRF passaram em 2026-05-11 08:59.

## Endpoint novo: `/api-v2/documents/public-tokens`

- **Arquivo:** `portal-sama-api/src/modules/documents/documents.controller.ts`, `portal-sama-api/src/modules/documents/documents.service.ts`
- **Método provável:** GET, POST e DELETE em `/api-v2/documents/public-tokens/:id`.
- **Finalidade:** Listar, emitir e revogar tokens públicos para upload de documentos por cliente.
- **Página/tela relacionada:** futura tela interna de onboarding/documentos e gestão de links públicos.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/documents/public-tokens`, `POST /api-v2/documents/public-tokens` e `DELETE /api-v2/documents/public-tokens/:id`.
- **Módulo NestJS alvo:** `DocumentsModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real.
- **Riscos de segurança:** vazamento de token, emissão indevida de link público, revogação sem auditoria. Endpoints exigem JWT e `documents.public_tokens`; mutações exigem CSRF e auditoria. Listagem não retorna `tokenHash` nem segredo bruto; criação retorna o token bruto somente uma vez.
- **Testes necessários:** 200/201 com MySQL real, 403 sem CSRF, 403 sem `documents.public_tokens`, 404 cliente/token inexistente, auditoria persistida e token bruto ausente de logs.
- **Observações:** Unitários de emissão/listagem/revogação e e2e 401/403/CSRF passaram em 2026-05-12 13:52.

## Endpoint novo: `/api-v2/documents/public-upload` e aliases de onboarding publico

- **Arquivo:** `portal-sama-api/src/modules/documents/documents.controller.ts`, `portal-sama-api/src/modules/documents/documents.service.ts`, `portal-sama-web/src/pages/documents/PublicDocumentsPage.tsx`
- **Método provável:** POST multipart, campo `file`, token em query string `token`.
- **Finalidade:** Receber documento enviado por cliente externo via link publico de onboarding/documentos.
- **Página/tela relacionada:** `Onboarding/documentos-cliente.html`, `portal-sama-web/src/pages/documents/PublicDocumentsPage.tsx`.
- **Novo endpoint proposto:** Implementado como `POST /api-v2/documents/public-upload?token=...`.
- **Módulo NestJS alvo:** `DocumentsModule`, `PublicLinksModule`.
- **Status de migração:** Implementado parcialmente; conectado ao frontend React publico em 2026-05-18 e ainda nao validado contra MySQL/storage/ClamAV reais.
- **Riscos de segurança:** abuso de token publico, upload malicioso, replay de link, vazamento de token em logs e persistencia indevida de arquivo. O backend valida hash do token, expiracao, revogacao, escopo/modulo, cliente, extensao, MIME, assinatura, tamanho, scanner/quarentena, storage privado e auditoria.
- **Testes necessários:** token invalido/expirado/revogado, escopo incorreto, arquivo valido, extensao/MIME/assinatura invalidos, EICAR/ClamAV em modo strict, storage real, auditoria sem token bruto e Playwright da tela publica.
- **Observações:** A tela React nao grava token em storage do navegador e nao renderiza HTML bruto; o checklist publico minimo e os aliases de onboarding estao implementados, mantendo os endpoints genericos por compatibilidade.

Atualizacao 2026-05-25 17:19: o checklist publico minimo e os aliases `GET /api-v2/public/onboarding/documents/:token` e `POST /api-v2/public/onboarding/documents/:token/upload` estao implementados. O frontend React publico passou a consumir esses aliases; os endpoints genericos por query string seguem compativeis.

## Endpoint novo: `/api-v2/documents/:id`

- **Arquivo:** `portal-sama-api/src/modules/documents/documents.controller.ts`, `portal-sama-api/src/modules/documents/documents.service.ts`
- **Método provável:** GET.
- **Finalidade:** Consultar metadados de um documento sem expor caminho físico/storage key.
- **Página/tela relacionada:** futura visualização de detalhe de documento.
- **Novo endpoint proposto:** Implementado como `/api-v2/documents/:id`.
- **Módulo NestJS alvo:** `DocumentsModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** IDOR por `id`; exige JWT, `documents.read` e validação de escopo.
- **Testes necessários:** 200 com escopo, 404 para inexistente/sem escopo, 403 sem permissão.
- **Observações:** Unitários cobrem busca/escopo em service; teste real depende de MySQL.

## Endpoint novo: `/api-v2/documents/:id/status`

- **Arquivo:** `portal-sama-api/src/modules/documents/documents.controller.ts`, `portal-sama-api/src/modules/documents/documents.service.ts`.
- **Método provável:** PATCH JSON.
- **Finalidade:** Revisar status de documento para `PENDING`, `APPROVED` ou `REJECTED`.
- **Página/tela relacionada:** `Client/painel.html`, onboarding interno e futura tela React de documentos.
- **Novo endpoint proposto:** Implementado como `/api-v2/documents/:id/status`.
- **Módulo NestJS alvo:** `DocumentsModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** aprovação/rejeição indevida, IDOR por `id`, alteração de documento arquivado e CSRF. Endpoint exige JWT, `documents.review`, CSRF, escopo por papel/departamento e auditoria.
- **Testes necessários:** 200 com MySQL real, 403 sem CSRF, 403 sem `documents.review`, 404 inexistente/arquivado, IDOR cliente/departamento e persistência de auditoria.
- **Observações:** Unitário cobre metadata/auditoria e escopo; e2e cobre 401, 403 sem permissão e 403 sem CSRF. Fluxo real depende de MySQL/seed/storage de homologação.

## Endpoint novo: `/api-v2/documents/:id/download`

- **Arquivo:** `portal-sama-api/src/modules/documents/documents.controller.ts`, `portal-sama-api/src/modules/documents/documents.service.ts`, `portal-sama-api/src/modules/documents/document-storage.service.ts`
- **Método provável:** GET.
- **Finalidade:** Baixar documento por endpoint autenticado/autorizado, fazendo stream de storage privado.
- **Página/tela relacionada:** `Client/painel.html`, onboarding interno e futuras telas React.
- **Novo endpoint proposto:** Implementado como `/api-v2/documents/:id/download`.
- **Módulo NestJS alvo:** `DocumentsModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** download indevido, path traversal, exposição de caminho físico. Endpoint exige JWT e `documents.download`; storage resolve caminho dentro da raiz privada e registra auditoria.
- **Testes necessários:** 200 com arquivo real, 404 arquivo ausente, 403 sem `documents.download`, IDOR cliente/departamento, headers de download.
- **Observações:** E2E sem permissão retorna 403 em 2026-05-11; stream real depende de storage de homologação.

## Endpoint novo: `/api-v2/documents/clients/:clientId/upload`

- **Arquivo:** `portal-sama-api/src/modules/documents/documents.controller.ts`, `portal-sama-api/src/modules/documents/document-file-validator.service.ts`, `portal-sama-api/src/modules/documents/document-storage.service.ts`
- **Método provável:** POST multipart, campo `file`.
- **Finalidade:** Enviar documento para um cliente específico.
- **Página/tela relacionada:** `Client/painel.html`, futuro painel React de documentos.
- **Novo endpoint proposto:** Implementado como `/api-v2/documents/clients/:clientId/upload`.
- **Módulo NestJS alvo:** `DocumentsModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** upload malicioso, CSRF, path traversal, arquivo executável, metadado sensível. Exige JWT, `documents.upload`, CSRF, validação de extensão/MIME/assinatura/tamanho, quarentena, scanner configuravel e storage privado.
- **Testes necessários:** arquivo válido real, extensão inválida, MIME inválido, assinatura inválida, tamanho excedido, 403 sem CSRF, escopo por cliente/departamento.
- **Observações:** E2E sem CSRF retorna 403 em 2026-05-11; em 2026-05-11 13:55 a API v2 passou a usar quarentena privada e scanner configuravel antes do storage final. Validacao operacional de ClamAV/EICAR e modo `strict` ainda pendente.

## Endpoint novo: `/api-v2/documents/upload`

- **Arquivo:** `portal-sama-api/src/modules/documents/documents.controller.ts`
- **Método provável:** POST multipart, campo `file` e `clientId` no body.
- **Finalidade:** Alias de upload para compatibilizar migração gradual de clientes que enviam `clientId` no corpo.
- **Página/tela relacionada:** `Client/painel.html`.
- **Novo endpoint proposto:** Implementado como `/api-v2/documents/upload`.
- **Módulo NestJS alvo:** `DocumentsModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** mesmos riscos de `/api-v2/documents/clients/:clientId/upload`; exige JWT, `documents.upload` e CSRF.
- **Testes necessários:** validação de `clientId`, upload válido real, rejeições de segurança e ClamAV/EICAR em modo `strict` no host.
- **Observações:** Mantido como rota de transição; preferir rota com `clientId` no path em novos fluxos.

## Endpoint novo: `/api-v2/documents/:id` (DELETE)

- **Arquivo:** `portal-sama-api/src/modules/documents/documents.controller.ts`, `portal-sama-api/src/modules/documents/documents.service.ts`
- **Método provável:** DELETE.
- **Finalidade:** Arquivar logicamente documento, alterando status para `ARCHIVED`.
- **Página/tela relacionada:** futura gestão de documentos.
- **Novo endpoint proposto:** Implementado como `DELETE /api-v2/documents/:id`.
- **Módulo NestJS alvo:** `DocumentsModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** remoção indevida de documento; exige JWT, `documents.delete`, CSRF, escopo e auditoria.
- **Testes necessários:** 204 com permissão, 403 sem CSRF/permissão, 404 sem escopo, validação com MySQL real.
- **Observações:** Não remove fisicamente o arquivo nesta versão.

## Endpoint novo: `/api-v2/proposals`

- **Arquivo:** `portal-sama-api/src/modules/proposals/proposals.controller.ts`, `portal-sama-api/src/modules/proposals/proposals.service.ts`
- **Método provável:** GET e POST JSON.
- **Finalidade:** Listar e criar propostas comerciais migradas do fluxo legado.
- **Página/tela relacionada:** `Legalizacao/proposta.html`, `Onboarding/processo.html`, `portal-sama-web/src/pages/proposals/ProposalPage.tsx`.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/proposals` e `POST /api-v2/proposals`.
- **Módulo NestJS alvo:** `ProposalsModule`, `AuditModule`.
- **Status de migração:** Implementado parcialmente; conectado ao frontend React interno em 2026-05-15 e ainda não validado contra MySQL real/homologacao.
- **Riscos de segurança:** enumeração de propostas, IDOR por `clientId`, criação indevida e ausência de auditoria. GET exige JWT e `proposals.read`; POST exige JWT, `proposals.create`, CSRF e auditoria.
- **Testes necessários:** 200/201 com MySQL real, 401 sem token, 403 sem permissão, 403 sem CSRF, filtros por status/cliente e validação de escopo por cliente/gestor.
- **Observações:** Unitários do service e e2e 401/403/CSRF passaram em 2026-05-13 08:57.

## Endpoint novo: `/api-v2/proposals/:id`

- **Arquivo:** `portal-sama-api/src/modules/proposals/proposals.controller.ts`, `portal-sama-api/src/modules/proposals/proposals.service.ts`
- **Método provável:** GET e PATCH JSON.
- **Finalidade:** Consultar e atualizar proposta sem expor token público bruto.
- **Página/tela relacionada:** `Legalizacao/proposta.html`, `portal-sama-web/src/pages/proposals/ProposalPage.tsx`.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/proposals/:id` e `PATCH /api-v2/proposals/:id`.
- **Módulo NestJS alvo:** `ProposalsModule`.
- **Status de migração:** Implementado parcialmente; conectado ao frontend React interno em 2026-05-15 e ainda não validado contra MySQL real/homologacao.
- **Riscos de segurança:** IDOR por `id`, alteração sem autorização, exposição de dados comerciais. GET exige `proposals.read`; PATCH exige `proposals.create`, CSRF, escopo e auditoria.
- **Testes necessários:** 200 com escopo, 404 inexistente/sem escopo, 403 sem permissão, 403 sem CSRF no PATCH e persistência em MySQL real.
- **Observações:** Respostas preservam aliases legados como `company_name`, `tipo_proposta`, `proposal_status` e `proposal_fields`.

## Endpoint novo: `/api-v2/proposals/:id/send`

- **Arquivo:** `portal-sama-api/src/modules/proposals/proposals.controller.ts`, `portal-sama-api/src/modules/proposals/proposals.service.ts`
- **Método provável:** POST JSON.
- **Finalidade:** Enviar proposta ao cliente gerando link público por token opaco.
- **Página/tela relacionada:** `Legalizacao/proposta.html`, `portal-sama-web/src/pages/proposals/ProposalPage.tsx`.
- **Novo endpoint proposto:** Implementado como `POST /api-v2/proposals/:id/send`.
- **Módulo NestJS alvo:** `ProposalsModule`, `PublicLinksModule`, `AuditModule`.
- **Status de migração:** Implementado parcialmente; conectado ao frontend React interno em 2026-05-15 e ainda não validado contra MySQL real/homologacao.
- **Riscos de segurança:** vazamento/replay de token, envio sem autorização, token bruto em logs. Endpoint exige JWT, `proposals.create`, CSRF, escopo e auditoria; token bruto é retornado somente na resposta de envio.
- **Testes necessários:** envio com MySQL real, token expirado/revogado, revogação de tokens anteriores, auditoria sem token bruto e 403 sem CSRF.
- **Observações:** Usa `PublicToken` com hash SHA-256, módulo `proposals`, escopo por proposta e token `sama_pub_*`.

## Endpoint novo: `/api-v2/proposals/:id/approve|reject`

- **Arquivo:** `portal-sama-api/src/modules/proposals/proposals.controller.ts`, `portal-sama-api/src/modules/proposals/proposals.service.ts`
- **Método provável:** POST JSON.
- **Finalidade:** Aprovar ou rejeitar proposta internamente.
- **Página/tela relacionada:** `Legalizacao/proposta.html`, `Onboarding/processo.html`.
- **Novo endpoint proposto:** Implementado como `POST /api-v2/proposals/:id/approve` e `POST /api-v2/proposals/:id/reject`.
- **Módulo NestJS alvo:** `ProposalsModule`, `AuditModule`.
- **Status de migração:** Implementado parcialmente; conectado ao frontend React interno em 2026-05-15 e ainda não validado contra MySQL real/homologacao.
- **Riscos de segurança:** decisão indevida, CSRF e falta de trilha auditável. Rotas exigem JWT, permissões `proposals.approve`/`proposals.reject`, CSRF, escopo e auditoria.
- **Testes necessários:** aprovação/rejeição com MySQL real, 403 sem permissão, 403 sem CSRF, 404 sem escopo e persistência de auditoria.
- **Observações:** E2E cobre rejeição sem CSRF para aprovação; unitários cobrem transição de status no service.

## Endpoint novo: `/api-v2/public/proposals/:token`

- **Arquivo:** `portal-sama-api/src/modules/proposals/proposals.controller.ts`, `portal-sama-api/src/modules/proposals/proposals.service.ts`
- **Método provável:** GET e POST JSON.
- **Finalidade:** Permitir visualização e resposta pública do cliente para proposta enviada.
- **Página/tela relacionada:** `Onboarding/proposta-cliente.html`, `portal-sama-web/src/pages/proposals/PublicProposalPage.tsx`.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/public/proposals/:token` e `POST /api-v2/public/proposals/:token`.
- **Módulo NestJS alvo:** `ProposalsModule`, `PublicLinksModule`, `AuditModule`.
- **Status de migração:** Implementado parcialmente; conectado ao frontend React publico em 2026-05-15 e ainda não validado contra MySQL real/homologacao.
- **Riscos de segurança:** enumeração de token, replay de resposta pública, ajuste sem mensagem e token vencido/revogado. Rotas usam hash do token, expiração, revogação, escopo por módulo/entidade e auditoria.
- **Testes necessários:** token inválido, expirado, revogado, proposta fora de status `WAITING_CLIENT`, aprovar, pedir ajuste e verificar revogação após uso em MySQL real.
- **Observações:** POST público aceita `approve` ou `adjust`; ajuste exige mensagem e revoga o token após a resposta.

## Endpoint novo: `/api-v2/contracts`

Atualizacao 2026-05-15 16:13: rota conectada ao frontend React em `/legalizacao/contratos` para listagem, filtros e criacao; ainda falta validacao com MySQL/backfill reais e Playwright.

- **Arquivo:** `portal-sama-api/src/modules/contracts/contracts.controller.ts`, `portal-sama-api/src/modules/contracts/contracts.service.ts`
- **Método provável:** GET e POST JSON.
- **Finalidade:** Listar e criar contratos migrados do fluxo de legalização.
- **Página/tela relacionada:** `Legalizacao/contrato.html`, futura tela React de contratos.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/contracts` e `POST /api-v2/contracts`.
- **Módulo NestJS alvo:** `ContractsModule`, `AuditModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real/baselined.
- **Riscos de segurança:** enumeração de contratos, IDOR por `clientId`, HTML rico com XSS e criação sem auditoria. GET exige JWT e `contracts.read`; POST exige JWT, `contracts.generate`, CSRF, escopo e auditoria.
- **Testes necessários:** 200/201 com MySQL real, 401 sem token, 403 sem permissão, 403 sem CSRF, filtros por status/cliente, validação de escopo e HTML malicioso.
- **Observações:** Unitários do service e e2e 401/403/CSRF passaram em 2026-05-13 09:34.

## Endpoint novo: `/api-v2/contracts/:id`

Atualizacao 2026-05-15 16:13: rota conectada ao frontend React em `/legalizacao/contratos/:id` para detalhe e edicao; ainda falta validacao com MySQL/backfill reais e Playwright.

- **Arquivo:** `portal-sama-api/src/modules/contracts/contracts.controller.ts`, `portal-sama-api/src/modules/contracts/contracts.service.ts`
- **Método provável:** GET e PATCH JSON.
- **Finalidade:** Consultar e atualizar contrato sem expor token público bruto.
- **Página/tela relacionada:** `Legalizacao/contrato.html`.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/contracts/:id` e `PATCH /api-v2/contracts/:id`.
- **Módulo NestJS alvo:** `ContractsModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real/baselined.
- **Riscos de segurança:** IDOR por `id`, alteração sem autorização, XSS por HTML editável e exposição de dados contratuais. GET exige `contracts.read`; PATCH exige `contracts.generate`, CSRF, escopo e auditoria.
- **Testes necessários:** 200 com escopo, 404 inexistente/sem escopo, 403 sem permissão, 403 sem CSRF e persistência em MySQL real.
- **Observações:** Respostas preservam aliases legados como `titulo`, `contract_status`, `html_content`, `recipient_name` e `recipient_document`.

## Endpoint novo: `/api-v2/contracts/:id/generate`

Atualizacao 2026-05-15 16:13: rota conectada ao frontend React para gerar snapshot HTML a partir dos campos editaveis do contrato; ainda falta validacao integrada com dados reais.

- **Arquivo:** `portal-sama-api/src/modules/contracts/contracts.controller.ts`, `portal-sama-api/src/modules/contracts/contracts.service.ts`
- **Método provável:** POST JSON.
- **Finalidade:** Gerar snapshot HTML validado do contrato antes de assinatura.
- **Página/tela relacionada:** `Legalizacao/contrato.html`.
- **Novo endpoint proposto:** Implementado como `POST /api-v2/contracts/:id/generate`.
- **Módulo NestJS alvo:** `ContractsModule`.
- **Status de migração:** Implementado parcialmente; esta rota salva snapshot HTML, e a renderizacao PDF fica em `POST /api-v2/contracts/:id/render-pdf`.
- **Riscos de segurança:** XSS, injeção em renderizador e falta de trilha. Endpoint exige JWT, `contracts.generate`, CSRF, valida HTML por denylist inicial e audita hash do documento.
- **Testes necessários:** HTML válido, HTML malicioso, 403 sem CSRF, persistência de hash e futura validação do renderizador em sandbox.
- **Observações:** Esta etapa nao renderiza PDF; apenas prepara snapshot HTML auditavel. A importacao/download seguro de PDF foi separada em `/api-v2/contracts/:id/pdf`.

## Endpoint novo: `/api-v2/contracts/:id/render-pdf`

Atualizacao 2026-05-15 16:13: rota conectada ao frontend React para acionar renderizacao server-side; ainda depende de Legal Doc Service real em homologacao.

- **Arquivo:** `portal-sama-api/src/modules/contracts/contracts.controller.ts`, `portal-sama-api/src/modules/contracts/contracts.service.ts`, `portal-sama-api/src/modules/contracts/contract-pdf-render.service.ts`
- **Método provável:** POST JSON.
- **Finalidade:** Renderizar o HTML salvo do contrato em PDF usando Legal Doc Service server-side e gravar o resultado em storage privado.
- **Página/tela relacionada:** `Legalizacao/contrato.html`, futura tela React de contratos.
- **Novo endpoint proposto:** Implementado como `POST /api-v2/contracts/:id/render-pdf`.
- **Módulo NestJS alvo:** `ContractsModule`, `AuditModule`, Legal Doc Service externo.
- **Status de migração:** Implementado no codigo; depende de `LEGAL_DOC_SERVICE_ENABLED=true`, `LEGAL_RENDER_V2_ENABLED=true`, URL e credencial do serviço em ambiente.
- **Riscos de segurança:** renderizador externo indisponivel, HTML inseguro, PDF malicioso retornado, vazamento de storage key e abuso de recurso pesado. Endpoint exige JWT, `contracts.generate`, CSRF e escopo; valida HTML salvo, valida o PDF retornado, salva em storage privado e audita sem storage key.
- **Testes necessários:** render com serviço real, fidelidade visual, timeouts, fontes/margens, PDF inválido retornado, 403 sem CSRF, 503 quando desativado e persistência em MySQL/storage reais.
- **Observações:** A rota retorna o contrato atualizado com `pdfDownloadUrl`/`pdf_download_url`; o download continua em `GET /api-v2/contracts/:id/pdf`.

## Endpoint novo: `/api-v2/contracts/:id/pdf`

Atualizacao 2026-05-15 16:13: rota conectada ao frontend React para importacao multipart e download protegido de PDF; ainda falta validacao com storage/MySQL reais.

- **Arquivo:** `portal-sama-api/src/modules/contracts/contracts.controller.ts`, `portal-sama-api/src/modules/contracts/contracts.service.ts`, `portal-sama-api/src/modules/contracts/contract-pdf-file-validator.service.ts`, `portal-sama-api/src/modules/contracts/contract-pdf-storage.service.ts`
- **Método provável:** POST multipart e GET download.
- **Finalidade:** Importar PDF final de contrato para storage privado e baixar o PDF protegido sem expor chave interna de storage.
- **Página/tela relacionada:** `Legalizacao/contrato.html`, futura tela React de contratos.
- **Novo endpoint proposto:** Implementado como `POST /api-v2/contracts/:id/pdf` e `GET /api-v2/contracts/:id/pdf`.
- **Módulo NestJS alvo:** `ContractsModule`, `AuditModule`.
- **Status de migração:** Implementado parcialmente; importacao/download seguro existe e a renderizacao HTML para PDF fica em `POST /api-v2/contracts/:id/render-pdf`.
- **Riscos de segurança:** upload malicioso, path traversal, vazamento de storage key, IDOR no download e cache indevido. POST exige JWT, `contracts.generate`, CSRF, escopo, validacao de PDF por assinatura/MIME/extensao/tamanho/marcadores ativos e auditoria sem storage key. GET exige JWT, `contracts.read`, escopo e retorna `Cache-Control: no-store`.
- **Testes necessários:** upload/download com MySQL/storage reais, PDF ativo/malicioso, substituicao de PDF anterior, 401 sem token, 403 sem permissao, 403 sem CSRF no POST e 404 sem PDF.
- **Observações:** `CONTRACT_PDF_STORAGE_PATH` permite separar o storage privado de contratos; `MAX_CONTRACT_PDF_UPLOAD_SIZE_MB` controla o limite. A resposta de contrato inclui `pdfDownloadUrl`/`pdf_download_url` quando existe PDF.

## Endpoint novo: `/api-v2/contracts/:id/send-signature`

Atualizacao 2026-05-15 16:13: rota conectada ao frontend React para gerar link local `/assinatura/:token`; o token bruto continua exibido apenas na resposta de envio e nao e armazenado no navegador.

- **Arquivo:** `portal-sama-api/src/modules/contracts/contracts.controller.ts`, `portal-sama-api/src/modules/contracts/contracts.service.ts`
- **Método provável:** POST JSON.
- **Finalidade:** Enviar contrato para assinatura gerando link público por token opaco.
- **Página/tela relacionada:** `Legalizacao/contrato.html`, `Legalizacao/assinatura.html`.
- **Novo endpoint proposto:** Implementado como `POST /api-v2/contracts/:id/send-signature`.
- **Módulo NestJS alvo:** `ContractsModule`, `PublicLinksModule`, `AuditModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real/baselined.
- **Riscos de segurança:** vazamento/replay de token, envio sem autorização, token bruto em logs. Endpoint exige JWT, `contracts.sign`, CSRF, escopo e auditoria; token bruto é retornado somente uma vez.
- **Testes necessários:** envio com MySQL real, token expirado/revogado, revogação de tokens anteriores, auditoria sem token bruto e 403 sem CSRF.
- **Observações:** Usa `PublicToken` com hash SHA-256, módulo `signatures`, entidade `contract` e escopo `signature`.

## Endpoint novo: `/api-v2/public/signatures/:token`

Atualizacao 2026-05-15 16:13: rota conectada ao frontend publico React em `/assinatura/:token`, com exibicao textual do contrato e submissao de assinatura sem `localStorage`/`sessionStorage`.

- **Arquivo:** `portal-sama-api/src/modules/contracts/contracts.controller.ts`, `portal-sama-api/src/modules/contracts/contracts.service.ts`
- **Método provável:** GET e POST JSON em `/sign`.
- **Finalidade:** Permitir visualização e assinatura pública de contrato por token.
- **Página/tela relacionada:** `Legalizacao/assinatura.html`, futura `PublicSignaturePage.tsx`.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/public/signatures/:token` e `POST /api-v2/public/signatures/:token/sign`.
- **Módulo NestJS alvo:** `ContractsModule`, `SignaturesModule`, `PublicLinksModule`, `AuditModule`.
- **Status de migração:** Implementado parcialmente; não conectado ao frontend e não validado contra MySQL real/baselined.
- **Riscos de segurança:** enumeração de token, replay de assinatura, assinatura múltipla e token vencido/revogado. Rotas usam hash do token, expiração, revogação, escopo por módulo/entidade, throttle e auditoria.
- **Testes necessários:** token inválido, expirado, revogado, contrato fora de status `WAITING_SIGNATURE`, assinatura com nome/documento, revogação após uso e persistência de evidência em MySQL real.
- **Observações:** POST público grava hash do documento/assinatura, IP, user-agent e revoga o token; não grava token bruto em auditoria.

## Endpoint novo: `/api-v2/legalization/processes`

- **Arquivo:** `portal-sama-api/src/modules/legalization/legalization.controller.ts`, `portal-sama-api/src/modules/legalization/legalization.service.ts`
- **Metodo provavel:** GET e POST JSON.
- **Finalidade:** Listar e criar processos internos de legalizacao, ligando cliente, proposta, contrato, etapa, status, responsavel e protocolo.
- **Pagina/tela relacionada:** `Legalizacao/legalizacao.html`, futura tela React de legalizacao.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/legalization/processes` e `POST /api-v2/legalization/processes`.
- **Modulo NestJS alvo:** `LegalizationModule`, `AuditModule`.
- **Status de migracao:** Implementado parcialmente; nao conectado ao frontend e sem backfill legado.
- **Riscos de seguranca:** enumeracao de processos, IDOR por cliente/protocolo, alteracao sem escopo e perda de trilha. GET exige JWT e `legalization.read`; POST exige JWT, `legalization.create`, CSRF, escopo e auditoria.
- **Testes necessarios:** 200/201 com MySQL real, filtros por status/etapa/cliente/protocolo, 401 sem token, 403 sem permissao, 403 sem CSRF e validacao de escopo.
- **Observacoes:** Respostas preservam aliases legados como `company_name`, `cnpj`, `tipo_processo`, `protocolo`, `etapa` e `status_legacy`.

## Endpoint novo: `/api-v2/legalization/processes/:id`

- **Arquivo:** `portal-sama-api/src/modules/legalization/legalization.controller.ts`, `portal-sama-api/src/modules/legalization/legalization.service.ts`
- **Metodo provavel:** GET, PATCH e DELETE.
- **Finalidade:** Consultar, atualizar e arquivar logicamente processos internos de legalizacao.
- **Pagina/tela relacionada:** `Legalizacao/legalizacao.html`.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/legalization/processes/:id`, `PATCH /api-v2/legalization/processes/:id`, `DELETE /api-v2/legalization/processes/:id` e `GET /api-v2/legalization/processes/:id/timeline`.
- **Modulo NestJS alvo:** `LegalizationModule`, `AuditModule`.
- **Status de migracao:** Implementado parcialmente; restore/hard delete/purge e backfill legado seguem pendentes.
- **Riscos de seguranca:** IDOR por `id`, mudanca indevida de responsavel/status e arquivamento sem auditoria. Rotas internas exigem permissoes `legalization.*`, CSRF nas mutacoes e escopo por responsavel/criador/departamento.
- **Testes necessarios:** detalhe com escopo, 404 inexistente/arquivado, patch sem CSRF, delete sem CSRF, timeline com historico de status e persistencia em MySQL real.
- **Observacoes:** `PATCH /:id/status` registra evento de timeline em `metadata.status_history`.

## Endpoint novo: `/api-v2/legalization/templates`

- **Arquivo:** `portal-sama-api/src/modules/legalization/legalization-templates.controller.ts`, `portal-sama-api/src/modules/legalization/legalization.service.ts`
- **Metodo provavel:** GET, POST, PATCH e DELETE JSON.
- **Finalidade:** Listar, criar, atualizar e remover logicamente templates de cabecalho/rodape usados no fluxo de legalizacao.
- **Pagina/tela relacionada:** `Legalizacao/legalizacao.html`, `Legalizacao/contrato.html`, futura tela React de legalizacao/contratos.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/legalization/templates`, `POST /api-v2/legalization/templates`, `PATCH /api-v2/legalization/templates/:id` e `DELETE /api-v2/legalization/templates/:id`.
- **Modulo NestJS alvo:** `LegalizationModule`, `AuditModule`.
- **Status de migracao:** Implementado parcialmente; backfill de templates legados e frontend seguem pendentes.
- **Riscos de seguranca:** XSS por HTML persistido, alteracao indevida de template, perda de trilha e exclusao sem auditoria. GET exige JWT e `legalization.read`; mutacoes exigem JWT, `legalization.templates`, CSRF e auditoria.
- **Testes necessarios:** listagem com MySQL real, criacao/edicao com CSRF valido, rejeicao de HTML inseguro, exclusao logica, 401/403 e backfill legado.
- **Observacoes:** Respostas preservam aliases `tipo`/`nome`; HTML com `script`, handlers inline, iframe/object/embed/link/meta ou `javascript:` e rejeitado no service.

## Endpoint atual: `/api/storage.php`

- **Arquivo:** `api/storage.php`
- **Método provável:** GET/POST conforme action.
- **Finalidade:** Autenticação, sessão, KV store, auditoria, presença, administração de usuários, backups e restauração.
- **Página/tela relacionada:** `index.html`, `auth.js`, `global.js`, `Auditoria/autoria.html`, `DEV/*`, múltiplas páginas internas.
- **Novo endpoint proposto:** Dividir entre `/api-v2/auth/*`, `/api-v2/users/*`, `/api-v2/clients/*`, `/api-v2/audit/*`, `/api-v2/admin/backups/*`.
- **Módulo NestJS alvo:** `AuthModule`, `UsersModule`, `ClientsModule`, `AuditModule`, `AdminModule`.
- **Status de migração:** Parcialmente implementado para auth, auditoria, users, roles, permissions e primeira fatia de clients; users/roles/permissions já possuem mutações administrativas principais e `ClientsModule` cobre CRUD/dashboard inicial. KV store, backups/restauração, colaboradores e várias actions genéricas seguem mapeadas.
- **Riscos de segurança:** Endpoint genérico com muitas responsabilidades; risco de mass assignment, IDOR, manipulação de auditoria e autorização insuficiente por action.
- **Testes necessários:** Login/logout/me, CSRF em mutações, autorização por perfil, acesso negado sem sessão, auditoria sem usuário forjado.
- **Observações:** Actions identificadas incluem `auth_login`, `auth_session_status`, `auth_logout`, `get`, `set`, `delete`, `audit_push`, `audit_list`, `backup_now`, `restore_select`, `admin_users_list`.

## Endpoint atual: `/api/client_documents.php`

- **Arquivo:** `api/client_documents.php`, `api/client_documents_lib.php`
- **Método provável:** GET para list/template/download; POST multipart para upload e add_custom.
- **Finalidade:** Template, listagem, upload, download e pendências de documentos de clientes.
- **Página/tela relacionada:** `Client/painel.html`, `Client/visao-geral-clientes.html`, `Onboarding/processo.html`.
- **Novo endpoint proposto:** `/api-v2/documents`, `/api-v2/documents/templates`, `/api-v2/documents/required-pending-summary`, `/api-v2/documents/custom-requirements`, `/api-v2/documents/public-tokens`, `/api-v2/documents/public-upload`, `/api-v2/documents/upload`, `/api-v2/documents/:id/status`, `/api-v2/documents/:id/download`, `/api-v2/clients/:clientId/documents`.
- **Módulo NestJS alvo:** `DocumentsModule`.
- **Status de migração:** Parcialmente implementado no NestJS para template, checklist/listagem, detalhe, upload autenticado, upload público por `PublicToken`, emissão/listagem/revogação administrativa de tokens públicos, download, resumo de pendências, add_custom/requisitos customizados, revisão de status e arquivamento lógico.
- **Riscos de segurança:** IDOR por `client_id`/`id`, upload malicioso, download indevido, escopo por departamento.
- **Testes necessários:** Upload válido real, MIME inválido, assinatura inválida, download sem permissão, download de outro cliente, CSRF em upload, escopo por departamento/gestor, ClamAV/EICAR e quarentena em host real.
- **Observações:** O legado já possui allowlist, MIME, assinatura, limite de tamanho, quarentena e scanner configurável em `api/client_documents_lib.php`. Em 2026-05-11 13:55, `DocumentsModule` passou a ter quarentena privada e scanner configuravel no upload NestJS; falta validacao operacional de ClamAV strict.

## Endpoint atual: `/api/certificados.php`

- **Arquivo:** `api/certificados.php`, `api/certificate_secret_lib.php`
- **Método provável:** GET/POST conforme action.
- **Finalidade:** Listar, criar, atualizar, excluir e baixar certificados digitais.
- **Página/tela relacionada:** `DptClient/certificados-digitais.html`, `Client/painel.html`.
- **Novo endpoint proposto:** `/api-v2/certificates`, `/api-v2/certificates/:id/download`.
- **Módulo NestJS alvo:** `CertificatesModule`, `AuditModule`.
- **Status de migração:** Implementado parcialmente no NestJS para CRUD, download protegido, rotação de senha, storage privado, criptografia AES-256-GCM, CSRF em mutações e auditoria; não conectado ao frontend e não validado contra MySQL/storage reais.
- **Riscos de segurança:** Exposição de senha/arquivo de certificado, download indevido, falta de auditoria completa.
- **Testes necessários:** Senha nunca retorna em JSON, criptografia configurada, download exige permissão, CRUD exige CSRF/role, validação com storage/MySQL reais e backfill legado.
- **Observações:** O arquivo legado usa extensões `p12`/`pfx`, limite de 5 MB e criptografia de segredo com sodium ou OpenSSL. A API v2 possui migration `20260512195000_add_digital_certificates`.

## Endpoint atual: `/api/legalizacao.php`

- **Arquivo:** `api/legalizacao.php`, `api/legal_doc_service.php`, `api/public_token_lib.php`
- **Método provável:** GET/POST conforme action.
- **Finalidade:** Legalização, contratos, templates, importação/renderização de documentos, assinatura pública por token.
- **Página/tela relacionada:** `Legalizacao/legalizacao.html`, `Legalizacao/proposta.html`, `Legalizacao/contrato.html`, `Legalizacao/assinatura.html`, `Onboarding/processo.html`.
- **Novo endpoint proposto:** `/api-v2/legalization/processes`, `/api-v2/legalization/templates`, `/api-v2/contracts`, `/api-v2/public/signatures/:token`.
- **Módulo NestJS alvo:** `LegalizationModule`, `ContractsModule`, `SignaturesModule`, `PublicLinksModule`.
- **Status de migração:** Parcialmente implementado para contratos e assinatura pública em `ContractsModule` e processos/templates em `LegalizationModule`; renderizacao/importacao/download seguro de PDF de contrato ja existe no codigo, mas importacao Word, validacao operacional do renderizador, restore/hard_delete/purge e backfill legado seguem mapeados.
- **Riscos de segurança:** XSS por HTML de template, IDOR por contrato, token público vazado, renderização PDF com conteúdo não sanitizado.
- **Testes necessários:** Token expirado/revogado, usuário sem permissão, sanitização HTML, aprovação/assinatura auditada, baseline/migrations em MySQL real e renderizador em sandbox.
- **Observações:** Actions identificadas incluem `list`, `get`, `create`, `update`, `delete`, `send`, `getByToken`, `clientSubmit`, `listTemplates`, `createTemplate`, `updateTemplate` e `deleteTemplate`. Em 2026-05-13 16:16, processos internos e templates de legalizacao foram iniciados em `/api-v2/legalization/*`; em 2026-05-14, contratos receberam renderizacao/importacao/download seguro de PDF. Homologacao do renderizador e compatibilidade de tokens legados seguem pendentes.

## Endpoint atual: `/api/propostas.php`

- **Arquivo:** `api/propostas.php`, `api/public_token_lib.php`
- **Método provável:** GET/POST conforme action.
- **Finalidade:** Propostas internas e resposta pública do cliente.
- **Página/tela relacionada:** `Legalizacao/proposta.html`, `Onboarding/proposta-cliente.html`, `Onboarding/processo.html`.
- **Novo endpoint proposto:** `/api-v2/proposals`, `/api-v2/public/proposals/:token`.
- **Módulo NestJS alvo:** `ProposalsModule`, `PublicLinksModule`, `AuditModule`.
- **Status de migração:** Implementado parcialmente no NestJS para listagem, detalhe, criação, edição, envio com token opaco, decisão interna e resposta pública; conectado ao frontend React interno/publico em 2026-05-15 e ainda não validado contra MySQL real.
- **Riscos de segurança:** Proposta comercial exposta por token, replay de ação pública, alteração sem auditoria.
- **Testes necessários:** Token inválido, token expirado, token revogado, aprovar/rejeitar proposta, pedido de ajuste público, usuário sem permissão, CSRF em mutações internas e validação com MySQL real.
- **Observações:** Actions identificadas incluem `list`, `get`, `create`, `save`, `send`, `delete`, `client_get`, `client_action`. Em 2026-05-13 08:57, `delete` e backfill de tokens/propostas legadas seguem pendentes.

## Endpoint atual: `/api/onboarding.php`

- **Arquivo:** `api/onboarding.php`, `api/public_token_lib.php`
- **Método provável:** GET/POST conforme action.
- **Finalidade:** Processos de onboarding, etapas, proposta, documentos, links públicos e finalização.
- **Página/tela relacionada:** `Onboarding/processo.html`, `Onboarding/entrada-cliente.html`, `Onboarding/documentos-cliente.html`, `Onboarding/proposta-cliente.html`.
- **Novo endpoint proposto:** `/api-v2/onboarding/processes`, `/api-v2/public/onboarding/documents/:token`, `/api-v2/public/onboarding/proposals/:token`.
- **Módulo NestJS alvo:** `OnboardingModule`, `DocumentsModule`, `ProposalsModule`, `PublicLinksModule`.
- **Status de migração:** Parcialmente implementado em rotas internas por `OnboardingModule`; fluxos públicos específicos seguem pendentes.
- **Riscos de segurança:** Upload público, token sem escopo correto, avanço indevido de etapa, download indevido.
- **Testes necessários:** Token inválido/expirado, upload inválido, etapa bloqueada, escopo por departamento, máquina de estados completa e validação com dados reais.
- **Observações:** Actions identificadas incluem `create`, `delete`, `move`, `stage_set`, `proposal_send`, `docs_upload`, `docs_download`, `docs_finish`. Em 2026-05-13 15:18, CRUD interno, status/etapa, timeline, documentos em metadata, CSRF, RBAC e auditoria foram implementados em `/api-v2/onboarding/processes`.

Atualizacao 2026-05-25 17:19: o fluxo publico especifico de documentos de onboarding deixou de estar pendente no contrato de rota e passou a existir por `GET /api-v2/public/onboarding/documents/:token` e `POST /api-v2/public/onboarding/documents/:token/upload`; seguem pendentes proposta publica especifica de onboarding, finalizacao e validacao real.

## Endpoint novo: `/api-v2/onboarding/processes`

- **Arquivo:** `portal-sama-api/src/modules/onboarding/onboarding.controller.ts`, `portal-sama-api/src/modules/onboarding/onboarding.service.ts`.
- **Método provável:** GET e POST.
- **Finalidade:** Listar processos de onboarding com filtros/paginacao e criar processo de entrada de cliente.
- **Página/tela relacionada:** `Onboarding/entrada-cliente.html`, `Onboarding/processo.html`, futura rota React `/onboarding/processos`.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/onboarding/processes` e `POST /api-v2/onboarding/processes`.
- **Módulo NestJS alvo:** `OnboardingModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** Criacao indevida de processo, listagem fora do departamento, IDOR por cliente/processo. GET exige JWT e `onboarding.read`; POST exige JWT, `onboarding.create`, CSRF e auditoria.
- **Testes necessários:** 200 com dados reais, 401 sem token, 403 sem permissao/CSRF, escopo por departamento/responsavel, backfill de `onboarding_process`.
- **Observações:** Resposta preserva aliases `company_name`, `tipo_servico`, `regime_lucro`, `stage`, `created_by`.

## Endpoint novo: `/api-v2/onboarding/processes/:id`

- **Arquivo:** `portal-sama-api/src/modules/onboarding/onboarding.controller.ts`.
- **Método provável:** GET, PATCH e DELETE.
- **Finalidade:** Consultar, atualizar e arquivar processo de onboarding.
- **Página/tela relacionada:** `Onboarding/processo.html`, futura rota React `/onboarding/processos/:id`.
- **Novo endpoint proposto:** Implementado como `GET/PATCH/DELETE /api-v2/onboarding/processes/:id`.
- **Módulo NestJS alvo:** `OnboardingModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** IDOR, alteracao indevida de dados comerciais, exclusao destrutiva. PATCH/DELETE exigem CSRF e permissoes `onboarding.update`/`onboarding.delete`; delete e arquivamento logico.
- **Testes necessários:** 401/403, 404, usuario fora do escopo, arquivamento com dados reais.
- **Observações:** Escopo por responsavel, criador, departamento ou papel privilegiado.

## Endpoint novo: `/api-v2/onboarding/processes/:id/status`

- **Arquivo:** `portal-sama-api/src/modules/onboarding/onboarding.controller.ts`.
- **Método provável:** PATCH.
- **Finalidade:** Alterar etapa e status do onboarding.
- **Página/tela relacionada:** `Onboarding/processo.html`.
- **Novo endpoint proposto:** Implementado como `PATCH /api-v2/onboarding/processes/:id/status`.
- **Módulo NestJS alvo:** `OnboardingModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** Pular etapa obrigatoria, concluir processo indevidamente. Rota exige JWT, `onboarding.status`, CSRF, escopo e auditoria.
- **Testes necessários:** transicoes invalidas, pendencias obrigatorias, usuario fora do escopo e auditoria persistida.
- **Observações:** Primeira fatia grava historico em `metadata.status_history`; uma maquina de estados formal ainda e pendente.

## Endpoint novo: `/api-v2/onboarding/processes/:id/timeline`

- **Arquivo:** `portal-sama-api/src/modules/onboarding/onboarding.controller.ts`.
- **Método provável:** GET.
- **Finalidade:** Retornar timeline auditavel do processo a partir dos timestamps e de `metadata.status_history`.
- **Página/tela relacionada:** futura esteira React de onboarding.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/onboarding/processes/:id/timeline`.
- **Módulo NestJS alvo:** `OnboardingModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** Expor historico de outro processo; rota exige JWT, `onboarding.read` e escopo.
- **Testes necessários:** timeline com eventos reais, ordenacao, 401/403/404.
- **Observações:** E2E cobre protecao de rota.

## Endpoint novo: `/api-v2/onboarding/processes/:id/documents`

- **Arquivo:** `portal-sama-api/src/modules/onboarding/onboarding.controller.ts`.
- **Método provável:** POST.
- **Finalidade:** Vincular estado/requisitos/documentos ao processo de onboarding.
- **Página/tela relacionada:** `Onboarding/processo.html`, `Onboarding/documentos-cliente.html`.
- **Novo endpoint proposto:** Implementado como `POST /api-v2/onboarding/processes/:id/documents`.
- **Módulo NestJS alvo:** `OnboardingModule`, com integracao futura a `DocumentsModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** Vinculo indevido de documentos ou token; rota exige JWT, `onboarding.documents`, CSRF, escopo e auditoria.
- **Testes necessários:** vinculo com documentos reais, requisitos obrigatorios, docs_finish e upload publico especifico.
- **Observações:** A primeira fatia registra estado em `metadata.documents`; upload/download continuam no `DocumentsModule`.

## Endpoint novo: `/api-v2/access-requests`

- **Arquivo:** `portal-sama-api/src/modules/access-requests/access-requests.controller.ts`, `portal-sama-api/src/modules/access-requests/access-requests.service.ts`.
- **Método provável:** GET e POST.
- **Finalidade:** Listar solicitacoes de acesso com filtros/paginacao e criar solicitacao de acesso por colaborador ou gestor.
- **Página/tela relacionada:** `SolicitacaoAcesso/solicitacao-acesso.html`, rota React `/solicitacao-acesso`, `TI/acesso-ti.html` e rota React `/ti/acessos`.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/access-requests` e `POST /api-v2/access-requests`.
- **Módulo NestJS alvo:** `AccessRequestsModule`.
- **Status de migração:** Implementado parcialmente; conectado ao frontend React inicial em `/solicitacao-acesso` e à tela operacional `/ti/acessos`, mas ainda não validado contra MySQL real/backfill.
- **Riscos de segurança:** Exposicao de agenda/justificativa, colaborador consultando solicitacao de outro, criacao fora do escopo de departamento. GET exige JWT e `access_requests.read`; POST exige JWT, CSRF e valida solicitante/colaboradores a partir do usuario autenticado.
- **Testes necessários:** 200 com dados reais, 401 sem token, 403 sem permissao/CSRF, escopo por gestor/departamento, backfill de `sama_access_requests`.
- **Observações:** Unitarios e e2e passaram em 2026-05-13 10:15. Em 2026-05-15 09:44, `AccessRequestPage.tsx` passou a consumir listagem/criacao pela API v2. Em 2026-05-18 15:35, `TiAccessPage.tsx` passou a consumir a listagem para historico operacional de TI; validacao integrada e Playwright seguem pendentes.

## Endpoint novo: `/api-v2/access-requests/my/latest`

- **Arquivo:** `portal-sama-api/src/modules/access-requests/access-requests.controller.ts`.
- **Método provável:** GET.
- **Finalidade:** Retornar a ultima solicitacao do colaborador autenticado, compatibilizando o fluxo `colaborador_status` do legado.
- **Página/tela relacionada:** `SolicitacaoAcesso/solicitacao-acesso.html`.
- **Novo endpoint proposto:** Implementado como `/api-v2/access-requests/my/latest`.
- **Módulo NestJS alvo:** `AccessRequestsModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** Retornar status de outro colaborador; o service deriva `requesterUsername` do usuario autenticado.
- **Testes necessários:** usuario sem solicitacao, usuario com solicitacao, 401 sem token e validacao com dados reais.
- **Observações:** Não exige permissao administrativa, apenas JWT, pois retorna somente o proprio recurso.

Atualizacao em 2026-05-15 09:44: `AccessRequestPage.tsx` consome este endpoint para o painel "Minha ultima solicitacao"; validacao integrada com usuario real segue pendente.

## Endpoint novo: `/api-v2/access-requests/manager/approvals`

- **Arquivo:** `portal-sama-api/src/modules/access-requests/access-requests.controller.ts`.
- **Método provável:** GET.
- **Finalidade:** Listar solicitacoes pendentes para aprovacao do gestor autenticado.
- **Página/tela relacionada:** `SolicitacaoAcesso/solicitacao-acesso.html`, `TI/acesso-ti.html`, rota React `/ti/acessos` e fila de gestor/TI.
- **Novo endpoint proposto:** Implementado como `/api-v2/access-requests/manager/approvals`.
- **Módulo NestJS alvo:** `AccessRequestsModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** Gestor visualizar solicitacoes de outro gestor. Endpoint exige JWT e `access_requests.read`; o service filtra por `managerUsername`, exceto papeis privilegiados (`ADMIN`, `DEV`, `TI`).
- **Testes necessários:** 401 sem token, 403 sem permissao, gestor com/sem fila, privilegiado com visao operacional.
- **Observações:** Unit/e2e cobrem protecao de rota; validacao com usuarios reais segue pendente.

Atualizacao em 2026-05-15 09:44: `AccessRequestPage.tsx` consome este endpoint para a fila de aprovacoes quando a sessao possui escopo de gestor/TI; Playwright e validacao com dados reais seguem pendentes.

Atualizacao em 2026-05-18 15:35: `TiAccessPage.tsx` consome este endpoint para a fila operacional de TI quando a sessao possui role `TI`, `ADMIN` ou `DEV` e permissao `access_requests.read`; Playwright e validacao real seguem pendentes.

## Endpoint novo: `/api-v2/access-requests/:id`

- **Arquivo:** `portal-sama-api/src/modules/access-requests/access-requests.controller.ts`.
- **Método provável:** GET.
- **Finalidade:** Consultar uma solicitacao especifica com escopo por solicitante, gestor ou papel privilegiado.
- **Página/tela relacionada:** `SolicitacaoAcesso/solicitacao-acesso.html`, `TI/acesso-ti.html`.
- **Novo endpoint proposto:** Implementado como `/api-v2/access-requests/:id`.
- **Módulo NestJS alvo:** `AccessRequestsModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** IDOR por troca de `id`; endpoint exige JWT e `access_requests.read`, e o service restringe o acesso por usuario/departamento/papel.
- **Testes necessários:** 401/403, 404, usuario fora do escopo, gestor responsavel, TI/admin.
- **Observações:** A resposta preserva aliases legados como `perfil`, `approval_status`, `dias_acesso` e `horario_inicio`.

## Endpoint novo: `/api-v2/access-requests/:id/approve` e `/api-v2/access-requests/:id/reject`

- **Arquivo:** `portal-sama-api/src/modules/access-requests/access-requests.controller.ts`.
- **Método provável:** POST.
- **Finalidade:** Aprovar ou rejeitar solicitacao de colaborador pendente.
- **Página/tela relacionada:** `SolicitacaoAcesso/solicitacao-acesso.html`, `TI/acesso-ti.html`, rotas React `/solicitacao-acesso` e `/ti/acessos`.
- **Novo endpoint proposto:** Implementado como `/api-v2/access-requests/:id/approve` e `/api-v2/access-requests/:id/reject`.
- **Módulo NestJS alvo:** `AccessRequestsModule`.
- **Status de migração:** Implementado parcialmente.
- **Riscos de segurança:** Aprovacao indevida, replay de decisao, ausencia de CSRF. Rotas exigem JWT, permissao `access_requests.approve` ou `access_requests.reject`, CSRF e escopo por gestor responsavel ou papel privilegiado.
- **Testes necessários:** POST sem CSRF, decisao fora do escopo, status invalido, auditoria persistida em MySQL real.
- **Observações:** E2E cobre rejeicao sem CSRF; unitarios cobrem aprovacao, rejeicao e protecoes principais. Em 2026-05-15 09:44, `AccessRequestPage.tsx` passou a consumir estes endpoints para a fila de aprovacoes quando ha permissao de decisao. Em 2026-05-18 15:35, `TiAccessPage.tsx` passou a consumir estes endpoints para decisoes operacionais de TI; validacao integrada com usuario real segue pendente.

## Endpoint atual: `/api/access_requests.php`

- **Arquivo:** `api/access_requests.php`
- **Método provável:** GET/POST conforme action.
- **Finalidade:** Solicitações de acesso, status do colaborador, aprovações do gestor e decisão.
- **Página/tela relacionada:** `SolicitacaoAcesso/solicitacao-acesso.html`, `TI/acesso-ti.html`.
- **Novo endpoint proposto:** `/api-v2/access-requests`, `/api-v2/access-requests/:id/approve`, `/api-v2/access-requests/:id/reject`.
- **Módulo NestJS alvo:** `AccessRequestsModule`, `TiModule`.
- **Status de migração:** Parcialmente implementado em `/api-v2/access-requests`; legado ainda ativo.
- **Riscos de segurança:** Colaborador consultar solicitação de outro, gestor aprovar fora do escopo, CSRF em decisão.
- **Testes necessários:** 401 sem sessão, 403 fora do escopo, decisão exige CSRF, auditoria de aprovação/rejeição, backfill e compatibilidade com dados reais.
- **Observações:** Actions identificadas: `list`, `gestor_approvals`, `colaborador_status`, `decision`. Em 2026-05-13 10:15, a primeira fatia NestJS foi criada e testada por unit/e2e. Em 2026-05-18 15:35, `/ti/acessos` passou a ter primeira tela React operacional para fila, historico e decisoes usando API v2; a base editavel de credenciais de `TI/ti.js` segue no legado e nao foi migrada.

## Endpoint atual: `SolicitacaoAcesso/enviar_acesso.php`

- **Arquivo:** `SolicitacaoAcesso/enviar_acesso.php`
- **Método provável:** POST.
- **Finalidade:** Envio de solicitação de acesso.
- **Página/tela relacionada:** `SolicitacaoAcesso/solicitacao-acesso.html`.
- **Novo endpoint proposto:** `/api-v2/access-requests`.
- **Módulo NestJS alvo:** `AccessRequestsModule`.
- **Status de migração:** Parcialmente implementado em `/api-v2/access-requests`; legado endurecido ainda ativo.
- **Riscos de segurança:** Validação de sessão/CSRF ponto a validar, exposição de dados do solicitante e escopo do colaborador ainda exigem revisão. Exposição de stack trace por `display_errors=1` foi mitigada no legado.
- **Testes necessários:** Validação server-side, CSRF, usuário derivado da sessão, erros sem stack trace, migracao da tela para a API v2.
- **Observações:** Em 2026-05-08, `display_errors` e `display_startup_errors` passaram a ficar desabilitados por padrão e só ativam com `SAMA_DEBUG`, `APP_DEBUG` ou `SAMA_DISPLAY_ERRORS`; PHP lint ficou bloqueado por ausência de `php` no PATH local. Em 2026-05-13 10:15, `POST /api-v2/access-requests` passou a cobrir criacao de solicitacoes no backend NestJS.

## Endpoint novo: `/api-v2/transfers`

- **Arquivo:** `portal-sama-api/src/modules/transfers/transfers.controller.ts`, `portal-sama-api/src/modules/transfers/transfers.service.ts`.
- **Método provável:** GET e POST.
- **Finalidade:** Dashboard de transferencias, criacao de sessao e retorno manual de transferencia por tempo indeterminado.
- **Página/tela relacionada:** `Manager/manager-transfers.html`, `Manager/manager-colaborador.html`; rotas React `/manager/transferencias` e `/manager/colaboradores`.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/transfers`, `GET /api-v2/transfers/dashboard`, `POST /api-v2/transfers`, `POST /api-v2/transfers/return` e `POST /api-v2/transfers/:id/return`.
- **Módulo NestJS alvo:** `TransfersModule`.
- **Status de migração:** Parcialmente implementado; legado `api/manager_workspace.php?action=transfer_*` ainda ativo.
- **Riscos de segurança:** Transferencia indevida de carteira, gestor operando fora do departamento, CSRF em mutacoes e inconsistencias no retorno automatico/manual.
- **Testes necessários:** HTTP com JWT/CSRF reais, escopo por MANAGER/ADMIN/DEV, MySQL real, seed `transfers.*`, dados legados de carteira e Playwright da tela React.
- **Observações:** Em 2026-05-19 12:16, unitarios focados, suite completa, lint, build, `prisma:generate` e `prisma:validate` passaram. O modulo usa `TransferSession` em `sama_transfer_sessions` e atualiza carteira em `Client.metadata`/`User.metadata` ate existir modelagem formal. Em 2026-05-19 15:33, `/manager/transferencias` ganhou primeira tela React consumindo dashboard, criacao e retorno, validada com lint/build/audit frontend. Em 2026-05-19 15:53, `/manager/colaboradores` passou a consumir o dashboard para consulta de carteira por colaborador.

## Endpoint novo: `/api-v2/calendar`

- **Arquivo:** `portal-sama-api/src/modules/calendar/calendar.controller.ts`, `portal-sama-api/src/modules/calendar/calendar.service.ts`.
- **Metodo provavel:** GET, POST e DELETE.
- **Finalidade:** Configurar, listar e excluir vencimentos do calendario do gestor.
- **Pagina/tela relacionada:** `Manager/manager.html`, `Manager/manager-calendar-config.html`; rotas React `/manager` e `/manager/calendario/config`.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/calendar/config`, `POST /api-v2/calendar/config`, `GET /api-v2/calendar/month`, `GET /api-v2/calendar/day` e `DELETE /api-v2/calendar/entries/:id`.
- **Modulo NestJS alvo:** `CalendarModule`.
- **Status de migracao:** Parcialmente implementado; legado `api/manager_workspace.php?action=calendar_*` ainda ativo ate homologacao.
- **Riscos de seguranca:** Exposicao de vencimentos de outro departamento, alteracao/exclusao indevida de prazos, CSRF em mutacoes e perda de notificacoes vinculadas.
- **Testes necessarios:** HTTP com JWT/CSRF reais, escopo por MANAGER/ADMIN/DEV, MySQL real, seed `calendar.read/manage`, dados legados de calendario/carteira e Playwright das telas React.
- **Observacoes:** Em 2026-05-20 10:00, `CalendarModule` iniciou configuracao/leitura sobre `sama_calendar_rules`, `sama_calendar_rule_companies` e `sama_calendar_due_notified`; em 2026-05-20 10:30, `/manager` passou a consumir a leitura mensal; em 2026-05-20 16:02, `DELETE /api-v2/calendar/entries/:id` passou a substituir inicialmente `calendar_delete_entry`, limpando vinculos/notificacoes, removendo regra orfa e auditando `calendar.entry.delete`.

## Endpoint novo: `/api-v2/managers/overview` e `/api-v2/managers/history`

- **Arquivo:** `portal-sama-api/src/modules/managers/managers.controller.ts`, `portal-sama-api/src/modules/managers/managers.service.ts`.
- **Método provável:** GET/POST/PATCH.
- **Finalidade:** Listar overview do gestor, presenca da equipe, historico operacional por empresa/departamento, alem de criar/editar historico, salvar snapshot de vida da empresa e consultar timeline consolidada.
- **Página/tela relacionada:** `Manager/manager.html`; rota React `/manager/historico`.
- **Novo endpoint proposto:** Implementado como `GET /api-v2/managers/overview`, `GET /api-v2/managers/history`, `GET /api-v2/managers/history/:companyId/timeline`, `POST /api-v2/managers/history`, `PATCH /api-v2/managers/history/:id` e `POST /api-v2/managers/company-life`.
- **Módulo NestJS alvo:** `ManagersModule`.
- **Status de migração:** Parcialmente implementado para overview, leitura, escrita/edicao de `company_history` e salvamento de `company_life`; legado `api/manager_workspace.php?action=overview` ainda ativo ate homologacao.
- **Riscos de segurança:** Exposicao de presenca/historico de outro departamento, IDOR por `companyId`/`id`, edicao sem CSRF/auditoria em caminhos legados remanescentes.
- **Testes necessários:** HTTP com JWT/CSRF reais, escopo por departamento e perfil MANAGER/ADMIN/DEV, MySQL real com tabelas legadas, seed `manager_history.read/write` e Playwright da tela React.
- **Observações:** Em 2026-05-20 11:20, `CompanyHistoryEntry` e `CompanyLifeEntry` foram mapeados no Prisma com migration compativel `CREATE TABLE IF NOT EXISTS`; leitura exige JWT e RBAC `manager_history.read`, aplica escopo por departamento. Em 2026-05-20 11:45, mutacoes iniciais passaram a exigir `manager_history.write`, CSRF, escopo por departamento e auditoria centralizada. Em 2026-05-20 12:30, `GET /api-v2/managers/overview` passou a substituir inicialmente a action `overview`, lendo `UserPresence`/`sama_user_presence` para presenca.

## Endpoint atual: `/api/manager_workspace.php`

- **Arquivo:** `api/manager_workspace.php`
- **Método provável:** GET/POST conforme action.
- **Finalidade:** Workspace do gestor, histórico, vida da empresa, calendário e transferências.
- **Página/tela relacionada:** `Manager/manager.html`, `Manager/manager-colaborador.html`, `Manager/manager-calendar-config.html`, `Manager/manager-transfers.html`, `HOME/Inicio.html`.
- **Novo endpoint proposto:** `/api-v2/managers/overview`, `/api-v2/managers/history`, `/api-v2/managers/company-life`, `/api-v2/calendar`, `/api-v2/transfers`.
- **Módulo NestJS alvo:** `ManagersModule`, `CalendarModule`, `TransfersModule`.
- **Status de migração:** Parcialmente implementado para `transfer_*` via `/api-v2/transfers`, calendario via `/api-v2/calendar`, overview via `/api-v2/managers/overview` e historico/vida da empresa via `/api-v2/managers/history` + `/api-v2/managers/company-life`; PHP legado segue ativo ate homologacao.
- **Riscos de segurança:** Escopo por gestor, alterações de histórico/calendário sem permissão, transferência indevida.
- **Testes necessários:** Escopo por gestor/departamento, CSRF em mutações, auditoria.
- **Observações:** Actions identificadas incluem `overview`, `history_*`, `company_life_*`, `calendar_*`, `transfer_*`. Em 2026-05-19, `transfer_dashboard`, `transfer_submit` e `transfer_return` ganharam primeira implementacao NestJS parcial em `TransfersModule`; `/manager/transferencias` ganhou a tela de operacao e `/manager/colaboradores` ganhou a tela de consulta de carteira. Em 2026-05-20, `calendar_*` ganhou primeira fatia em `CalendarModule`, incluindo `calendar_delete_entry` por `DELETE /api-v2/calendar/entries/:id`; leitura e mutacoes iniciais de `history_*`/`company_life_save` ganharam primeira fatia em `ManagersModule`; `overview` ganhou substituto inicial em `GET /api-v2/managers/overview`.

## Endpoint atual: `/api/integra_ai.php`

- **Arquivo:** `api/integra_ai.php` e bibliotecas `api/integra_ai_*`
- **Método provável:** POST/GET conforme action.
- **Finalidade:** Fluxo contábil Integra-AI: importação, parsing, regras, conciliação, exportação TXT e jobs.
- **Página/tela relacionada:** `Contabil/integra-ai.html`.
- **Novo endpoint proposto:** `/api-v2/accounting/integra-ai/*`.
- **Módulo NestJS alvo:** `AccountingModule`, `IntegraAiModule`, `JobsModule`.
- **Status de migração:** Parcialmente implementado para leitura em `GET /api-v2/accounting/integra-ai/workspaces` e `GET /api-v2/accounting/integra-ai/jobs/:id`; mutacoes e download seguem no legado.
- **Riscos de segurança:** Upload/processamento de arquivos contábeis, dados sensíveis, jobs sem escopo, download de TXT.
- **Testes necessários:** Smoke do parser/exportador, permissoes por escopo contabil, upload invalido, auditoria de job, Playwright para leitura/sem permissao e validacao MySQL real.
- **Observações:** Há smoke script `scripts/integra_ai_smoke_current_flow.php`. Em 2026-05-21 10:33, a primeira fatia NestJS passou a consultar workspace/jobs de modo read-only com RBAC `accounting.integra_ai.read`, SQL parametrizado, sanitizacao de `source_path`/`txt_path` e auditoria ao abrir detalhe.

## Endpoint atual: `/api/notifications.php`

- **Arquivo:** `api/notifications.php`, `api/notifications_stream.php`
- **Método provável:** GET/POST conforme action; SSE em stream.
- **Finalidade:** Notificações, alertas, leitura, confirmação e limpeza.
- **Página/tela relacionada:** `global.js` e páginas internas.
- **Novo endpoint proposto:** `/api-v2/notifications`, `/api-v2/notifications/stream`, `/api-v2/notifications/push/public-key`, `/api-v2/notifications/push/subscribe`, `/api-v2/notifications/push/unsubscribe`, `/api-v2/notifications/push/test`.
- **Módulo NestJS alvo:** `NotificationsModule`.
- **Status de migração:** Parcialmente implementado para listagem, criacao, SSE, ack, alert-ack, limpeza de nao lidas, Web Push e teste operacional de push.
- **Riscos de segurança:** Usuário alterar/ler notificação de outro, ocultar alerta crítico. API v2 exige JWT/permissoes, escopo por usuario/departamento, CSRF nas mutacoes e auditoria.
- **Testes necessários:** Persistencia com MySQL real/homologacao, stream em navegador/frontend, Web Push em HTTPS com VAPID real e portal fechado usando o botao `Testar`, estrategia para updates instantaneos multi-instancia e integracao frontend.
- **Observações:** Actions identificadas: `list`, `create`, `ack`, `alert_ack`, `clear_unread`; `api/notifications_stream.php` foi mapeado para `GET /api-v2/notifications/stream`. Em 2026-05-14 08:40, `NotificationsModule` possui SSE com `last_id`/`lastId`, catch-up por MySQL e `AccessRequestsModule` emite as primeiras notificacoes automaticas. Em 2026-05-14 09:09, `global.js` ganhou opt-in para notificacoes nativas do navegador quando o portal esta aberto. Em 2026-05-14 09:35, a API v2 ganhou persistencia/envio Web Push e o legado ganhou `sama-push-sw.js`. Em 2026-05-14 10:39, foi adicionado `/api-v2/notifications/push/test` e botao `Testar` no painel legado. Em 2026-05-20 16:21, `/notificacoes` no React passou a consumir o stream por `fetch` com bearer em memoria e a operar Push API subscribe/unsubscribe; `portal-sama-web/public/sama-push-sw.js` serve o service worker no app Vite.

## Endpoint atual: `/api/fiscal_workspace.php`

- **Arquivo:** `api/fiscal_workspace.php`
- **Método provável:** GET/POST conforme action.
- **Finalidade:** Workspace fiscal/departamental.
- **Página/tela relacionada:** `Depto/modelo.html`.
- **Novo endpoint proposto:** `/api-v2/departments/fiscal/workspace`.
- **Módulo NestJS alvo:** `DepartmentsModule`.
- **Status de migração:** Mapeado.
- **Riscos de segurança:** Alteração de células/status fora do departamento permitido.
- **Testes necessários:** Escopo por departamento, CSRF em mudança de status, auditoria.
- **Observações:** Actions identificadas: `load_board`, `cycle_cell`, `set_cell_status`.
