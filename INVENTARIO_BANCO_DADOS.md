# [PARCIAL] Inventário de Banco de Dados - Portal Sama

Status: inventário em andamento baseado em `api/db.php`, no schema alvo `portal-sama-api/prisma/schema.prisma` e na migration inicial `portal-sama-api/prisma/migrations/20260512172700_init_current_schema`. O banco MySQL legado real ainda não foi comparado nesta etapa.

## Resumo

- **Origem atual:** schema imperativo em `api/db.php`.
- **Banco alvo documentado:** MySQL 8 + Prisma ORM.
- **Prisma:** Em andamento em `portal-sama-api/prisma/schema.prisma` com migration inicial validada e migrations incrementais para certificados, propostas, contratos, solicitacoes de acesso, clientes e colaboradores.
- **Validação do banco real:** ponto a validar.

## Schema Prisma inicial: `portal-sama-api/prisma/schema.prisma`

- **Status:** Criado, validado por `prisma validate` e coberto pela migration inicial `20260512172700_init_current_schema`, com incrementos `20260512195000_add_digital_certificates`, `20260513085700_add_proposals`, `20260513092300_add_contracts`, `20260513101500_add_access_requests`, `20260513113000_expand_clients_profile` e `20260513124500_add_collaborator_profile_to_users`
- **Origem:** Prisma/MySQL alvo
- **Finalidade:** Fundação inicial para identidade, RBAC, documentos, clientes, refresh tokens, auditoria, certificados digitais, propostas, contratos e solicitacoes de acesso.
- **Modelos principais:** `User`, `Role`, `Permission`, `UserRole`, `RolePermission`, `RefreshToken`, `Client`, `PublicToken`, `Document`, `DigitalCertificate`, `Proposal`, `Contract`, `AccessRequest`, `DocumentRequirement`, `DocumentStatusHistory`, `AuditLog`.
- **Relacionamentos:** usuários-roles, roles-permissões, usuário-refresh tokens, cliente-documentos, cliente-requisitos documentais, usuário-auditoria, usuário-documentos enviados.
- **Índices:** status/departamento de usuário, status/razão social de cliente, índices de documentos, refresh tokens e auditoria. `RefreshToken.tokenHash` é único para permitir lookup seguro por hash de token opaco; `Document.storageKey` é único para evitar colisão de arquivo privado.
- **Dados sensíveis:** `passwordHash`, `tokenHash`, metadados de auditoria e storage de documentos.
- **Regras de retenção:** ponto a validar.
- **Migrations relacionadas:** `20260512172700_init_current_schema` criada e validada em MySQL descartável com `prisma migrate deploy`; migrations incrementais existem para certificados, propostas, contratos, solicitacoes de acesso, clientes e colaboradores. Em 2026-05-13 14:43, o MySQL local `portal_sama` estava baselined, `20260513124500_add_collaborator_profile_to_users` foi aplicada por `prisma:migrate:deploy`, `prisma:migrate:status` passou e `prisma migrate diff` nao encontrou diferencas. Homologacao/producao ainda exigem procedimento com backup.

## Modelo Prisma: `User`

- **Status:** Expandido no schema alvo e usado por `UsersModule`, `AuthModule` e `CollaboratorsModule`; migration de campos funcionais aplicada no MySQL local em 2026-05-13 14:43.
- **Origem:** Prisma/MySQL alvo, substituindo gradualmente `sama_usuarios_login` e parte de `sama_colaboradores`.
- **Finalidade:** armazenar identidade de login, status, departamento principal, dados funcionais de colaborador e relacoes de RBAC.
- **Campos principais:** `id`, `username`, `name`, `email`, `passwordHash`, `status`, `mainDepartment`, `position`, `phone`, `extension`, `metadata`, `archivedAt`, `archivedById`, `createdAt`, `updatedAt`.
- **Relacionamentos:** usuario possui roles, refresh tokens, logs de auditoria, documentos enviados, requisitos documentais criados e historicos de status de documentos.
- **Indices:** `username` unico, `email` unico, `status`, `mainDepartment`, `position`, `archivedAt`.
- **Dados sensiveis:** `passwordHash`, dados pessoais/funcionais e metadados operacionais; respostas de `UsersModule`/`CollaboratorsModule` nao retornam `passwordHash`.
- **Regras de retencao:** `DELETE /api-v2/collaborators/:id` arquiva logicamente com `status=INACTIVE`, `archivedAt` e `archivedById`; exclusao fisica segue ponto a validar.
- **Migrations relacionadas:** `20260512172700_init_current_schema` e `20260513124500_add_collaborator_profile_to_users`; status/diff Prisma passaram no MySQL local.

## Modelos Prisma: `Role`, `Permission`, `UserRole`, `RolePermission`

- **Status:** Criados no schema alvo e usados pelos módulos administrativos; banco real ponto a validar
- **Origem:** Prisma/MySQL alvo
- **Finalidade:** armazenar catálogo de roles/permissões e vínculos de usuários, permitindo RBAC granular na API v2.
- **Campos principais:** `Role.id`, `Role.name`, `Role.description`, `Permission.id`, `Permission.key`, `Permission.description`, chaves compostas de `UserRole` e `RolePermission`.
- **Relacionamentos:** usuários-roles e roles-permissões em relações N:N por tabelas de junção.
- **Índices:** `Role.name` único, `Permission.key` único, chaves compostas nas tabelas de junção.
- **Dados sensíveis:** matriz de privilégios; não contém senha, mas deve ser protegida por `roles.read`/`permissions.read`.
- **Seed relacionado:** `portal-sama-api/prisma/seed.ts` cria permissões/roles iniciais, incluindo `audit.read`, permissões administrativas, `documents.requirements` e `documents.review`, sem criar usuário ou senha.
- **Regras de retenção:** não aplicável inicialmente; alterações futuras de permissão devem ser auditadas.
- **Migrations relacionadas:** migration inicial criada; validar baseline e dados reais em MySQL/homologação.

## Modelo Prisma: `RefreshToken`

- **Status:** Criado no schema alvo; banco real ponto a validar
- **Origem:** Prisma/MySQL alvo
- **Finalidade:** armazenar refresh tokens opacos emitidos pelo `AuthModule` sem persistir o token bruto.
- **Campos principais:** `id`, `userId`, `tokenHash`, `revokedAt`, `expiresAt`, `createdAt`.
- **Relacionamentos:** pertence a `User`.
- **Índices:** `tokenHash` único, `userId`, `expiresAt`.
- **Dados sensíveis:** HMAC-SHA-256 do refresh token; não deve armazenar token bruto.
- **Regras de retenção:** ponto a validar; tokens expirados/revogados devem ser limpos por rotina futura.
- **Migrations relacionadas:** migration real pendente de MySQL/homologação.

## Modelo Prisma: `Client`

- **Status:** Expandido no schema alvo e usado pelo `ClientsModule`; migration aplicada no MySQL local em 2026-05-13 10:50
- **Origem:** Prisma/MySQL alvo, substituindo gradualmente dados de clientes mantidos via storage legado/action-based.
- **Finalidade:** armazenar cadastro operacional de clientes para listagem, painel, documentos, certificados, propostas e contratos.
- **Campos principais:** `id`, `razaoSocial`, `nomeFantasia`, `cnpj`, `email`, `status`, `rank`, `regimeTributario`, `idEmpresa`, `atividade`, `telefone`, `endereco`, `complemento`, `cep`, `bairro`, `cidade`, `estado`, `fazParteGrupo`, `grupoNome`, `metadata`, `deletedAt`, `deletedById`, `createdAt`, `updatedAt`.
- **Relacionamentos:** cliente possui documentos e requisitos documentais; certificados/propostas/contratos ainda usam `clientId` textual nesta etapa.
- **Índices:** `status`, `razaoSocial`, `cidade`, `grupoNome`, `deletedAt`; `cnpj` é único.
- **Dados sensíveis:** dados cadastrais/contato do cliente e metadados operacionais; respostas não devem expor campos internos desnecessários.
- **Regras de retenção:** `DELETE /api-v2/clients/:id` arquiva logicamente com `status=ARCHIVED`, `deletedAt` e `deletedById`.
- **Migrations relacionadas:** `20260512172700_init_current_schema` e `20260513113000_expand_clients_profile`; status/diff Prisma passaram no MySQL local.

## Modelo Prisma: `Document`

- **Status:** Criado no schema alvo e usado pelo `DocumentsModule`; banco real ponto a validar
- **Origem:** Prisma/MySQL alvo
- **Finalidade:** armazenar metadados de documentos enviados por clientes/colaboradores sem expor o caminho físico do arquivo.
- **Campos principais:** `id`, `clientId`, `type`, `department`, `originalName`, `storageKey`, `mimeType`, `size`, `sha256Hash`, `status`, `source`, `metadata`, `uploadedById`, `createdAt`, `updatedAt`. Em 2026-05-11 13:33, `metadata.review` passou a guardar status revisado, motivo opcional, usuario e horario ate existir tabela propria de historico. Em 2026-05-11 13:55, `metadata.scanner` passou a guardar status, modo, engine e horario da varredura de upload.
- **Relacionamentos:** pertence a `Client`; pode pertencer ao `User` que realizou upload.
- **Índices:** `clientId`, `type`, `department`, `status`, `source`, `createdAt`; `storageKey` é único.
- **Dados sensíveis:** `storageKey`, nome original do arquivo, metadados e hash do documento. Respostas da API v2 não devem retornar `storageKey`.
- **Regras de retenção:** ponto a validar; `DELETE /api-v2/documents/:id` arquiva logicamente (`ARCHIVED`) e não remove fisicamente nesta versão.
- **Migrations relacionadas:** migration real pendente de MySQL/homologação.

## Modelo Prisma: `DocumentRequirement`

- **Status:** Criado no schema alvo e usado pelo `DocumentsModule`; banco real ponto a validar
- **Origem:** Prisma/MySQL alvo
- **Finalidade:** armazenar requisitos documentais customizados por cliente, substituindo o uso de KV store do legado para `add_custom`.
- **Campos principais:** `id`, `clientId`, `key`, `label`, `department`, `required`, `active`, `createdById`, `createdAt`, `updatedAt`.
- **Relacionamentos:** pertence a `Client`; pode pertencer ao `User` que criou o requisito.
- **Índices:** chave única composta `clientId/key`, `clientId`, `department`, `active`.
- **Dados sensíveis:** metadados operacionais do cliente e departamento; não contém arquivo físico.
- **Regras de retenção:** ponto a validar; desativação futura deve preferir `active=false` com auditoria.
- **Migrations relacionadas:** migration real pendente de MySQL/homologação.

## Modelo Prisma: `AuditLog`

- **Status:** Criado no schema alvo e usado pelo `AuditModule`; banco real ponto a validar
- **Origem:** Prisma/MySQL alvo
- **Finalidade:** registrar eventos críticos de autenticação, auditoria e futuros módulos sensíveis.
- **Campos principais:** `id`, `userId`, `action`, `module`, `entityType`, `entityId`, `ipAddress`, `userAgent`, `metadata`, `createdAt`.
- **Relacionamentos:** pertence opcionalmente a `User`.
- **Índices:** `userId`, `module`, `entityType/entityId`, `createdAt`.
- **Dados sensíveis:** metadados podem conter contexto operacional; `AuditService` mascara chaves sensíveis como senha/token/secret/cookie nos novos eventos NestJS.
- **Regras de retenção:** ponto a validar.
- **Migrations relacionadas:** migration real pendente de MySQL/homologação.

## Modelo Prisma: `DigitalCertificate`

- **Status:** Criado no schema alvo e usado pelo `CertificatesModule`; banco/storage reais ponto a validar
- **Origem:** Prisma/MySQL alvo, substituindo gradualmente `sama_certificados_clientes`.
- **Finalidade:** armazenar metadados de certificados digitais sem expor senha nem caminho físico do arquivo.
- **Campos principais:** `id`, `clientId`, `clientName`, `department`, `name`, `passwordCipher`, `originalName`, `storageKey`, `mimeType`, `size`, `sha256Hash`, `metadata`, `createdById`, `updatedById`, `deletedAt`, `deletedById`, `createdAt`, `updatedAt`.
- **Relacionamentos:** vínculo por `clientId` ainda será consolidado com dados reais de clientes; neste incremento o modelo preserva compatibilidade durante o backfill.
- **Índices:** `clientId`, `department`, `deletedAt`, `createdAt`; `storageKey` é único.
- **Dados sensíveis:** `passwordCipher`, nome do arquivo, hash e metadados de certificado. Respostas da API v2 não devem retornar senha em claro nem `storageKey`.
- **Regras de retenção:** ponto a validar; exclusão atual é lógica por `deletedAt`.
- **Migrations relacionadas:** `20260512195000_add_digital_certificates`.

## Modelo Prisma: `Proposal`

- **Status:** Criado no schema alvo e usado pelo `ProposalsModule`; migration consta aplicada no MySQL local apos baseline de 2026-05-13 10:50, mas homologacao/producao seguem pendentes
- **Origem:** Prisma/MySQL alvo, substituindo gradualmente dados de `api/propostas.php` e fluxos de proposta no onboarding.
- **Finalidade:** armazenar propostas comerciais e seu estado, preservando compatibilidade com campos legados de resposta.
- **Campos principais:** `id`, `clientId`, `companyName`, `proposalType`, `status`, `fields`, `publicTokenId`, `feedback`, `sentAt`, `clientRespondedAt`, `createdById`, `updatedById`, `archivedAt`, `archivedById`, `metadata`, `createdAt`, `updatedAt`.
- **Relacionamentos:** proposta usa `clientId` textual opcional nesta primeira etapa; o token público é referenciado por `publicTokenId` e validado em `PublicToken`.
- **Índices:** `clientId`, `status`, `companyName`, `createdAt`.
- **Dados sensíveis:** dados comerciais da proposta, feedback do cliente e vínculo com token público. Token bruto não é armazenado no modelo.
- **Regras de retenção:** ponto a validar; arquivamento lógico previsto por `archivedAt`.
- **Migrations relacionadas:** `20260513085700_add_proposals`; baseline local resolvido em 2026-05-13 10:50 com status/diff Prisma limpos. Homologacao/producao ainda exigem procedimento com backup.

## Modelo Prisma: `Contract`

- **Status:** Criado no schema alvo e usado pelo `ContractsModule`; migration consta aplicada no MySQL local apos baseline de 2026-05-13 10:50, mas homologacao/producao seguem pendentes
- **Origem:** Prisma/MySQL alvo, substituindo gradualmente dados de `api/legalizacao.php` e fluxos de assinatura pública.
- **Finalidade:** armazenar contratos, snapshot HTML, estado de assinatura e evidência mínima auditável sem persistir token público bruto.
- **Campos principais:** `id`, `clientId`, `title`, `recipientName`, `recipientDocument`, `status`, `htmlContent`, `headerHtml`, `footerHtml`, `pdfStorageKey`, `publicTokenId`, `generatedAt`, `sentAt`, `signedAt`, `signatureName`, `signatureDocument`, `signatureHash`, `createdById`, `updatedById`, `archivedAt`, `archivedById`, `metadata`, `createdAt`, `updatedAt`.
- **Relacionamentos:** contrato usa `clientId` textual opcional nesta primeira etapa; token público de assinatura é referenciado por `publicTokenId` e validado em `PublicToken`.
- **Índices:** `clientId`, `status`, `title`, `archivedAt`, `createdAt`.
- **Dados sensíveis:** HTML contratual, dados do signatário, hash de assinatura e metadados de evidência. Token bruto não é armazenado no modelo nem na auditoria.
- **Regras de retenção:** ponto a validar; arquivamento lógico previsto por `archivedAt`. PDF assinado/storage privado ainda pendente.
- **Migrations relacionadas:** `20260513092300_add_contracts`; baseline local resolvido em 2026-05-13 10:50 com status/diff Prisma limpos. Homologacao/producao ainda exigem procedimento com backup.

## Modelo Prisma: `AccessRequest`

- **Status:** Criado no schema alvo e usado pelo `AccessRequestsModule`; migration consta aplicada no MySQL local apos baseline de 2026-05-13 10:50, mas homologacao/producao seguem pendentes
- **Origem:** Prisma/MySQL alvo, substituindo gradualmente `sama_access_requests` e fluxos de `api/access_requests.php`/`SolicitacaoAcesso/enviar_acesso.php`.
- **Finalidade:** armazenar solicitacoes de acesso temporario, perfil solicitante, datas/horarios, gestor aprovador, colaboradores vinculados e estado da decisao sem depender de action PHP.
- **Campos principais:** `id`, `profile`, `status`, `requesterUsername`, `requesterName`, `requesterEmail`, `department`, `position`, `accessDates`, `accessStartTime`, `accessEndTime`, `managerUsername`, `managerName`, `managerDecisionAt`, `managerDecisionBy`, `forwardedToMasterAt`, `collaboratorJustification`, `collaborators`, `createdById`, `decidedById`, `archivedAt`, `archivedById`, `metadata`, `createdAt`, `updatedAt`.
- **Relacionamentos:** solicitante, gestor e decisor sao vinculados inicialmente por username/id textual para preservar compatibilidade durante o backfill; `createdById`/`decidedById` apontam para usuarios quando disponiveis.
- **Índices:** `requesterUsername`, `managerUsername`, `department`, `profile/status`, `createdAt`.
- **Dados sensíveis:** justificativa, datas/horarios, dados de colaboradores e escopo de acesso. Auditoria deve evitar token/senha e excesso de justificativa.
- **Regras de retenção:** ponto a validar; arquivamento logico previsto por `archivedAt`.
- **Migrations relacionadas:** `20260513101500_add_access_requests`; baseline local resolvido em 2026-05-13 10:50 com status/diff Prisma limpos. Homologacao/producao ainda exigem procedimento com backup e backfill.

## Tabela: `kv_store`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado e `portal-sama-api/prisma/schema.prisma`
- **Finalidade:** armazenamento chave/valor usado por fluxos legados.
- **Campos principais:** ponto a validar no bloco `CREATE TABLE` de `api/db.php`.
- **Relacionamentos:** ponto a validar
- **Índices:** chave primária e índices ponto a validar
- **Dados sensíveis:** possível, conforme chave armazenada
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma específica

## Tabela: `audit_log`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** logs de auditoria.
- **Campos principais:** identificadores, ação, entidade, metadados e data, ponto a validar no schema real.
- **Relacionamentos:** ponto a validar
- **Índices:** `created_at`, `action`, `entity_type/entity_id` identificados em `api/db.php`
- **Dados sensíveis:** pode conter metadados sensíveis; não deve armazenar senhas/tokens
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma

## Tabela: `notifications`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** notificações e alertas internos.
- **Campos principais:** ponto a validar
- **Relacionamentos:** usuário/departamento por campos textuais, inferência baseada em `api/db.php`
- **Índices:** username, created_at, dept, expires_at, dedupe_key, event_type/module
- **Dados sensíveis:** possível conteúdo operacional
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma

## Tabela: `sama_access_requests`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** solicitações de acesso e aprovações.
- **Campos principais:** solicitante, perfil, departamento, status de aprovação, gestor aprovador, datas, ponto a validar no schema real.
- **Relacionamentos:** usuários/gestores por username, inferência baseada em `api/access_requests.php`
- **Índices:** created_at, perfil, solicitante_username, departamento, approval_status, gestor_aprovador_username
- **Dados sensíveis:** justificativas, horários e dados de acesso podem ser sensíveis
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** substituicao gradual pelo modelo Prisma `AccessRequest` e migration `20260513101500_add_access_requests`; backfill/compatibilidade ainda pendentes.

## Tabela: `onboarding_process`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** processos de onboarding, etapas, propostas, documentos e contratos.
- **Campos principais:** stage, status, company_name, data_json, proposal fields/tokens, contract fields, ponto a validar.
- **Relacionamentos:** cliente/contrato/proposta por campos legados, ponto a validar
- **Índices:** stage, updated_at, proposal_token
- **Dados sensíveis:** dados de cliente, documentos, proposta e contrato
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma

## Tabela: `sama_colaboradores`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** colaboradores.
- **Campos principais:** username, nome, departamento e dados complementares, ponto a validar.
- **Relacionamentos:** usuários e clientes, ponto a validar
- **Índices:** username, departamento
- **Dados sensíveis:** dados pessoais e vínculo operacional
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** primeira fatia mapeada para `users` por `20260513124500_add_collaborator_profile_to_users`; tabela/vinculos dedicados seguem ponto a validar apos backfill.

## Tabela: `sama_usuarios_login`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** usuários de login.
- **Campos principais:** username, password hash, nome, departamento, role, ativo, data_json, ponto a validar.
- **Relacionamentos:** colaboradores/roles por campos legados, ponto a validar
- **Índices:** role, departamento
- **Dados sensíveis:** senha hash, status de usuário e permissões
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma

## Tabela: `sama_login_security`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** tentativas de login, bloqueios e segurança de sessão.
- **Campos principais:** username, failed_attempts, last_failed_at, blocked_at, ponto a validar.
- **Relacionamentos:** usuário por username
- **Índices:** blocked_at
- **Dados sensíveis:** eventos de segurança
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma

## Tabela: `sama_rate_limit_buckets`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** rate limiting por bucket.
- **Campos principais:** bucket_key, hits, window_started_at, blocked_until, data_json.
- **Relacionamentos:** nenhum formal identificado
- **Índices:** blocked_until
- **Dados sensíveis:** IPs/metadados de segurança podem aparecer em `data_json`
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma

## Tabela: `sama_public_tokens` / `public_tokens`

- **Status:** Existente no código PHP; modelo Prisma alvo `PublicToken` implementado e usado pelo `DocumentsModule`; banco real ponto a validar
- **Origem:** PHP legado e `portal-sama-api/prisma/schema.prisma`
- **Finalidade:** tokens públicos com hash, escopo, expiração, revogação, rastreio de último uso e emissão administrativa.
- **Campos principais:** id, token_hash/tokenHash, module, entity_type/entityType, entity_id/entityId, scope, expires_at/expiresAt, revoked_at/revokedAt, last_used_ip/lastUsedIp, created_by_id/createdById, metadata.
- **Relacionamentos:** entidade por `module/entity_type/entity_id`
- **Índices:** token_hash único, module/entity/scope, expires_at/revoked_at
- **Dados sensíveis:** hash de token e IP de uso
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** modelo Prisma alvo criado; migration formal, seed/backfill e compatibilidade com `sama_public_tokens` ainda pendentes

## Tabela: `sama_user_presence`

- **Status:** Existente no código PHP e modelada no Prisma em 2026-05-20
- **Origem:** PHP legado; leitura inicial pela API v2
- **Finalidade:** presença/atividade recente de usuários e overview do gestor.
- **Campos principais:** username, session_id, role, departamento, page, status, first_seen_at, last_seen_at, last_offline_at, ip, user_agent, updated_at
- **Relacionamentos:** usuário por username
- **Índices:** status, last_seen_at
- **Dados sensíveis:** presença e atividade
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** `20260520123000_add_user_presence_model`; modelo Prisma `UserPresence`

## Tabela: `sama_user_activity_raw`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** eventos brutos de atividade de usuário.
- **Campos principais:** ponto a validar
- **Relacionamentos:** usuário por username
- **Índices:** username/created_at, month_key/username, event_day
- **Dados sensíveis:** rastros de atividade
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma

## Tabela: `sama_user_activity_monthly`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** rollup mensal de atividade.
- **Campos principais:** ponto a validar
- **Relacionamentos:** usuário por username
- **Índices:** username/month_key
- **Dados sensíveis:** métricas de atividade
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma

## Tabela: `sama_clientes`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** clientes.
- **Campos principais:** CNPJ, razão social/nome fantasia, status e dados complementares, ponto a validar.
- **Relacionamentos:** documentos, certificados, gestores e módulos operacionais, ponto a validar
- **Índices:** status, cnpj
- **Dados sensíveis:** dados empresariais e contatos
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma

## Tabela: `sama_portais_config`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** configuração de portais/tipos.
- **Campos principais:** ponto a validar
- **Relacionamentos:** ponto a validar
- **Índices:** ponto a validar
- **Dados sensíveis:** configuração operacional
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma

## Tabela: `sama_certificados_clientes`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** certificados digitais de clientes.
- **Campos principais:** client_id, dept, dados do certificado, senha criptografada/metadados de arquivo, ponto a validar.
- **Relacionamentos:** cliente por `client_id`
- **Índices:** client_id, dept
- **Dados sensíveis:** certificado digital e segredo criptografado
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma

## Tabela: `sama_ti_acessos`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** acessos/itens de TI.
- **Campos principais:** ponto a validar
- **Relacionamentos:** ponto a validar
- **Índices:** sort_order
- **Dados sensíveis:** acessos internos podem conter informações altamente sensíveis
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma

## Tabela: `sama_planilhas_departamento`

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Finalidade:** planilhas/modelos por departamento.
- **Campos principais:** dept e dados estruturados, ponto a validar.
- **Relacionamentos:** departamento por campo textual
- **Índices:** dept
- **Dados sensíveis:** dados operacionais por departamento
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** não há migration Prisma

## Tabelas de gestor, calendário, transferências e Integra-AI

As tabelas abaixo foram identificadas em `api/db.php`; o detalhamento de campos deve ser feito ao modelar o Prisma:

| Tabela | Finalidade inicial | Índices identificados / observações |
|---|---|---|
| `sama_company_history_entries` | historico de empresas usado por `api/manager_workspace.php` e `ManagersModule`; leitura, criacao e edicao inicial pela API v2 | modelada como `CompanyHistoryEntry`; indices company_id/dept, created_at, author_username |
| `sama_company_life_entries` | vida da empresa usada por `api/manager_workspace.php` e `ManagersModule`; leitura e salvamento de snapshots pela API v2 | modelada como `CompanyLifeEntry`; indices company_id/dept, created_at, author_username |
| `sama_user_presence` | presenca de usuarios usada por `api/storage.php`, streams legados e overview do gestor | modelada como `UserPresence`; indices status e last_seen_at |
| `sama_calendar_rules` | regras de calendario usadas pelo `CalendarModule`; configuracao, leitura e exclusao de regra orfa pela API v2 | owner/is_active, recurrence, scope |
| `sama_calendar_rule_companies` | vinculo regra-calendario/empresa; `DELETE /api-v2/calendar/entries/:id` remove este vinculo | rule_id, company_id/dept |
| `sama_calendar_due_notified` | controle de vencimentos notificados; exclusao de vencimento limpa notificacoes por regra/empresa/departamento | notification_day |
| `sama_transfer_sessions` | sessoes de transferencia | modelada no Prisma como `TransferSession` em 2026-05-19; indices departamento/status, origem/status, tipo/status |
| `sama_integra_ai_jobs` | jobs Integra-AI | company/created_at, profile/updated_at, status/updated_at |
| `sama_integra_ai_job_rows` | linhas de jobs Integra-AI | job/canonical_key, job/status |
| `sama_integra_ai_profiles` | perfis Integra-AI | company/bank |
| `sama_integra_ai_profile_settings` | configurações de perfil Integra-AI | ponto a validar |
| `sama_integra_ai_rules` | regras Integra-AI | profile/priority |
| `sama_integra_ai_plan_accounts` | plano de contas Integra-AI | plan_key/sort_order/account_code |
| `sama_integra_ai_import_profiles` | perfis de importação | active/updated_at |
| `sama_integra_ai_financial_reports` | relatórios financeiros | company/updated_at |
| `sama_integra_ai_financial_rows` | linhas financeiras | job/hash_line, job/status/direction |
| `sama_integra_ai_match_suggestions` | sugestões de conciliação | job/status/source, job/financial_row_id |
| `sama_integra_ai_learned_rules` | regras aprendidas | company/active/priority |
| `sama_integra_ai_audit_log` | auditoria Integra-AI | job/created_at, company/created_at |

Para todas as tabelas acima:

- **Status:** Existente no código PHP; banco real ponto a validar
- **Origem:** PHP legado
- **Campos principais:** ponto a validar
- **Relacionamentos:** ponto a validar
- **Dados sensíveis:** possível, especialmente dados contábeis e operacionais
- **Regras de retenção:** ponto a validar
- **Migrations relacionadas:** `sama_transfer_sessions` possui migration Prisma `20260519120000_add_transfer_sessions`; as demais tabelas ainda nao possuem migration Prisma.
