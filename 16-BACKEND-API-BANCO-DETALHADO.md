
# Backend, API e banco de dados — documentação detalhada da nova arquitetura

Atualizado em: 2026-06-19 12:00 -03:00

## 1. Objetivo

Documentar a arquitetura do `portal-sama-api`, seus módulos, banco de dados, contratos de API, regras de segurança, integrações e pontos que devem ser corrigidos antes da conclusão do projeto.

O backend deve ser finalizado antes do frontend completo. O frontend inicial só deve existir como ferramenta simples de validação do banco e da API.

## 2. Stack consolidada

| Camada | Tecnologia | Uso |
|---|---|---|
| Runtime | Node.js >= 22 | Execução da API |
| Framework | NestJS | Modularização, controllers, services, guards, pipes |
| Linguagem | TypeScript | Tipagem e segurança de contrato |
| ORM | Prisma | Schema, migrations e client tipado |
| Banco | MySQL 8 | Persistência principal |
| Validação HTTP | class-validator/class-transformer | DTOs em controllers |
| Validação de ambiente | Zod | `.env` tipado e fail-fast |
| Auth | JWT curto + refresh token | Sessão autenticada |
| Cookies | cookie-parser | Refresh e CSRF cookie |
| Segurança HTTP | Helmet, CORS, Throttler | Hardening inicial |
| Upload | Multer + validators próprios | Recebimento e validação de arquivos |
| Scanner | Regras estáticas + ClamAV opcional/strict | Segurança de documentos |
| Auditoria | `AuditModule` | Registro de eventos sensíveis |
| Deploy | Docker/EasyPanel | Imagem separada da API |

## 3. Bootstrap da API

A API usa prefixo global configurável, com padrão:

```txt
/api-v2
```

O bootstrap aplica:

- `helmet()`;
- `cookieParser()`;
- CORS com `credentials=true` e origem configurada por `CORS_ORIGIN`/`FRONTEND_URL`;
- `ValidationPipe` com `whitelist`, `forbidNonWhitelisted` e `transform`;
- filtro global de exceções HTTP;
- interceptor de request id;
- Swagger apenas fora de produção.

## 4. Módulos atuais do backend

| Módulo | Responsabilidade | Status técnico | Ajustes necessários |
|---|---|---|---|
| `AuthModule` | Login, refresh, logout, CSRF, `/me`, preferências, avatar | Boa fundação | Garantir cookies `Secure`, `HttpOnly`, `SameSite` em produção e política de rotação |
| `UsersModule` | Gestão de usuários internos | Parcial/boa | Consolidar backfill de usuários legados e trilhas de auditoria |
| `RolesModule`/`PermissionsModule` | RBAC | Boa fundação | Travar matriz e validar seed em produção |
| `ClientsModule` | Clientes, CRUD, dashboard | Boa fundação | Expandir dashboard para abas do cliente |
| `ClientAssignmentsModule` | Responsáveis/gestores por cliente/departamento | Boa fundação | Integrar totalmente à aba Responsáveis |
| `CollaboratorsModule` | Colaboradores internos | Boa fundação | Ajustar Acessórias para colaboradores desde mês atual |
| `DocumentsModule` | Checklist, upload, tokens públicos, revisão | Boa fundação | Garantir scanner strict em produção e retention |
| `CertificatesModule` | Certificados digitais e senha cifrada | Boa fundação | Auditar leitura de senha e exigir permissão específica |
| `ContractsModule` | Contratos internos, PDF, assinatura pública interna | Parcial | Implementar provider ZapSign real |
| `ProposalsModule` | Propostas e aceite público | Parcial/boa | Validar fluxo real com onboarding/legalização |
| `LegalizationModule` | Processos e templates de legalização | Parcial/boa | Integrar a contratos/propostas como jornada única |
| `OnboardingModule` | Esteira de entrada de cliente | Parcial/boa | Completar estados e automações documentais |
| `AccessRequestsModule` | Solicitações de acesso e aprovações | Boa fundação | Homologar perfis reais |
| `TransfersModule` | Transferência de carteira/clientes | Boa fundação | Amarrar no painel do cliente e gestor |
| `ManagersModule` | Overview, histórico, vida da empresa | Parcial | Trazer `Vida da empresa` para painel do cliente |
| `DepartmentsModule` | Workspace de departamentos e vencimentos | Boa fundação | Validar aplicação Acessórias com janela atual |
| `AccountingModule` | Integra-AI | Parcial/boa | Validar PDFs reais e storage |
| `NotificationsModule` | Notificações, SSE, Web Push | Boa fundação | Validar VAPID e navegadores reais |
| `AuditModule` | Logs e exportação | Boa fundação | Adicionar filtros por cliente/entity e retenção |
| `HealthModule` | Healthcheck | Boa fundação | Exigir em readiness de deploy |
| `Acessorias*` | Integração, entregas, catálogo, reconciliação, responsáveis | Avançado | Corrigir janela temporal e contratos externos |

## 5. Modelos principais do Prisma

O schema já cobre os domínios relevantes:

```txt
User, UserPresence, Department, Role, Permission, UserRole, RolePermission,
RefreshToken, Client, ClientDepartmentAssignment,
AcessoriasCompanyObligationCatalog, AcessoriasDelivery, AcessoriasResponsibleAlias,
AcessoriasDeliverySyncRun, AcessoriasSyncRun, AcessoriasExternalJobLock,
AcessoriasDeliveryColumnMapping, AcessoriasFiscalApplyRun, AcessoriasFiscalDivergence,
OnboardingProcess, LegalizationProcess, LegalizationTemplate,
Notification, NotificationDeliveryAttempt, NotificationPreference, BrowserPushSubscription,
PublicToken, Document, DigitalCertificate, Proposal, Contract, AccessRequest,
DocumentRequirement, DocumentStatusHistory, TransferSession,
CalendarRule, CalendarRuleCompany, CalendarDueNotified,
CompanyHistoryEntry, CompanyLifeEntry,
IntegraAiJob, IntegraAiJobRow, IntegraAiProfile, IntegraAiProfileSetting,
IntegraAiRule, IntegraAiPlanAccount, IntegraAiAuditLog, AuditLog
```

## 6. Decisões de banco

### 6.1 Regras obrigatórias

1. Toda alteração de schema deve gerar migration versionada.
2. Não usar `prisma db push` em produção ou homologação compartilhada.
3. `prisma migrate deploy` deve ser o caminho padrão para ambientes.
4. O banco deve ser validado antes do frontend avançado.
5. Toda migration deve ter rollback lógico documentado, mesmo que o Prisma não gere down migration automaticamente.
6. Dados sensíveis devem ser cifrados ou hasheados conforme tipo:
   - senha de usuário: bcrypt;
   - refresh token: hash/HMAC;
   - token público: hash/HMAC;
   - senha de certificado: AES-256-GCM;
   - senha de portal do cliente: novo cofre cifrado, nunca texto puro;
   - documentos: arquivo em storage privado, metadados no banco, hash SHA-256.

### 6.2 Índices recomendados adicionais

Verificar e adicionar se ausentes:

| Modelo | Índices recomendados |
|---|---|
| `Client` | `cnpj`, `status`, `nomeFantasia`, `razaoSocial`, `idEmpresa` |
| `ClientDepartmentAssignment` | `clientId + departmentId + status`, `responsibleUserId`, `managerUserId` |
| `Document` | `clientId + status`, `sha256Hash`, `publicTokenId` |
| `DigitalCertificate` | `clientId`, `expiresAt`, `sha256Hash` |
| `Contract` | `clientId`, `status`, `publicTokenId`, futuro `providerDocumentToken` |
| `AcessoriasDelivery` | `clientExternalId`, `clientId`, `dueAt`, `competence`, `status`, `obligationName` |
| `AcessoriasCompanyObligationCatalog` | `clientId`, `normalizedObligationName`, `status` |
| `AuditLog` | `userId`, `module`, `entityType`, `entityId`, `createdAt` |
| `ClientAccessSecret` futuro | `clientId`, `portalName`, `status`, `lastRevealedAt` |

## 7. Contratos de API por domínio

### 7.1 Auth

```txt
GET    /api-v2/auth/csrf
POST   /api-v2/auth/login
POST   /api-v2/auth/refresh
POST   /api-v2/auth/logout
GET    /api-v2/auth/me
POST   /api-v2/auth/forgot-password
GET    /api-v2/me/security
GET    /api-v2/me/preferences
PATCH  /api-v2/me/preferences
PATCH  /api-v2/me/preferences/intro
PATCH  /api-v2/me/avatar
GET    /api-v2/me/avatar
```

Regras:

- `accessToken` deve ser curto.
- refresh token deve ficar em cookie seguro e ser rotacionado a cada refresh.
- mutações devem exigir CSRF header + cookie.
- respostas de login inválido devem ser genéricas.
- auditoria deve registrar sucesso/falha sem senha.

### 7.2 Clientes e responsáveis

```txt
GET    /api-v2/clients
POST   /api-v2/clients
GET    /api-v2/clients/:id
GET    /api-v2/clients/:id/dashboard
PATCH  /api-v2/clients/:id
DELETE /api-v2/clients/:id
GET    /api-v2/clients/:clientId/assignments
POST   /api-v2/clients/:clientId/assignments
POST   /api-v2/client-assignments/transfer
PATCH  /api-v2/client-assignments/:id
POST   /api-v2/client-assignments/:id/end
```

Regras:

- `scope=mine` deve retornar somente clientes associados ao usuário/responsável/gestor quando aplicável.
- painel do cliente deve consumir uma combinação de dashboard + queries por aba.
- alterações em responsáveis devem auditar antes/depois.
- transferências devem preservar histórico, não sobrescrever vínculo sem rastro.

### 7.3 Documentos

```txt
GET    /api-v2/documents
GET    /api-v2/clients/:clientId/documents
POST   /api-v2/documents/clients/:clientId/upload
GET    /api-v2/documents/:id/download
PATCH  /api-v2/documents/:id/status
GET    /api-v2/documents/:id/status-history
POST   /api-v2/documents/custom-requirements
POST   /api-v2/documents/public-tokens
GET    /api-v2/public/onboarding/documents/:token
POST   /api-v2/public/onboarding/documents/:token/upload
```

Regras:

- upload sempre passa por allowlist, MIME, assinatura mágica, limite de tamanho, SHA-256 e scanner.
- storage deve ser privado, fora da pasta pública.
- download exige permissão e auditoria.
- token público deve ser opaco, hasheado no banco, com expiração e revogação.

### 7.4 Certificados digitais

```txt
GET    /api-v2/certificates
POST   /api-v2/certificates
GET    /api-v2/certificates/:id
PATCH  /api-v2/certificates/:id
GET    /api-v2/certificates/:id/download
POST   /api-v2/certificates/:id/rotate-password
DELETE /api-v2/certificates/:id
```

Regras:

- aceitar somente `.p12` e `.pfx`.
- validar assinatura PKCS#12.
- cifrar senha com AES-256-GCM.
- download e revelação/cópia de senha devem ser auditados.
- filtrar por cliente no painel do cliente.

### 7.5 Legalização, propostas e contratos

```txt
GET    /api-v2/legalization/processes
POST   /api-v2/legalization/processes
GET    /api-v2/legalization/processes/:id
PATCH  /api-v2/legalization/processes/:id
PATCH  /api-v2/legalization/processes/:id/status
GET    /api-v2/legalization/processes/:id/timeline
DELETE /api-v2/legalization/processes/:id
GET    /api-v2/legalization/templates
POST   /api-v2/legalization/templates
PATCH  /api-v2/legalization/templates/:id
DELETE /api-v2/legalization/templates/:id
GET    /api-v2/proposals
POST   /api-v2/proposals
POST   /api-v2/proposals/:id/send
POST   /api-v2/proposals/:id/approve
POST   /api-v2/proposals/:id/reject
GET    /api-v2/contracts
POST   /api-v2/contracts
POST   /api-v2/contracts/:id/generate
POST   /api-v2/contracts/:id/render-pdf
POST   /api-v2/contracts/:id/send-signature
GET    /api-v2/public/signatures/:token
POST   /api-v2/public/signatures/:token/sign
```

Ajuste obrigatório:

- `ContractsModule` deve suportar `signatureProvider = internal | zapsign`.
- `send-signature` deve chamar ZapSign quando provider for `zapsign`.
- contrato ZapSign não deve ser assinado pela rota pública interna; a rota pública deve redirecionar ou retornar o `sign_url` oficial.

### 7.6 Acessórias

```txt
POST /api-v2/integrations/acessorias/operational-sync
POST /api-v2/integrations/acessorias/companies/catalog/sync
POST /api-v2/integrations/acessorias/reconciliation/run
GET  /api-v2/integrations/acessorias/sync-runs
GET  /api-v2/integrations/acessorias/rate-limiter/status
GET  /api-v2/integrations/acessorias/deliveries
GET  /api-v2/integrations/acessorias/deliveries/preview
POST /api-v2/integrations/acessorias/deliveries/sync
POST /api-v2/integrations/acessorias/deliveries/backfill
POST /api-v2/integrations/acessorias/deliveries/apply-to-workspace/bulk
GET  /api-v2/integrations/acessorias/responsibles
GET  /api-v2/integrations/acessorias/responsibles/pending
POST /api-v2/integrations/acessorias/responsibles/apply-client-assignments
```

## 8. Regra obrigatória da API Acessórias

### 8.1 Obrigações/listas de entrega

A busca de obrigações/listas de entregas deve começar no ano atual.

Exemplo de regra:

```ts
const now = new Date();
const dtInitial = `${now.getFullYear()}-01-01`;
const dtFinal = `${now.getFullYear()}-12-31`;
```

Quando o objetivo operacional precisar olhar vencimentos futuros curtos, o backend pode buscar até alguns meses à frente, desde que o início continue no ano atual e que a regra esteja documentada.

Não permitir como padrão:

```txt
mês atual - 11 meses
últimos anos
backfill amplo sem confirmação explícita
```

### 8.2 Colaboradores/responsáveis

A busca de colaboradores deve começar no mês atual.

Exemplo de regra:

```ts
const now = new Date();
const dtInitial = `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-01`;
```

Se o endpoint externo não aceitar período:

1. buscar normalmente;
2. identificar campos como `status`, `ativo`, `data_desligamento`, `dt_desligamento`, `updated_at`, `competencia` ou equivalentes;
3. descartar colaboradores desligados ou inativos;
4. descartar registros antigos sem confirmação atual;
5. persistir somente responsáveis ativos ou pendentes de revisão.

### 8.3 Backfill Acessórias

Backfill amplo só pode ocorrer com:

- permissão `integrations.acessorias.backfill`;
- CSRF válido;
- confirmação explícita no DTO;
- janela de data informada;
- modo `dryRun` executado antes;
- evidência registrada em sync run;
- plano de rollback.

## 9. ZapSign no backend novo

Criar serviços dedicados:

```txt
src/modules/contracts/providers/zapsign/zapsign.module.ts
src/modules/contracts/providers/zapsign/zapsign.service.ts
src/modules/contracts/providers/zapsign/zapsign.types.ts
src/modules/contracts/providers/zapsign/zapsign-webhook.controller.ts
```

Variáveis esperadas:

```txt
ZAPSIGN_API_TOKEN=
ZAPSIGN_API_TOKEN_SANDBOX=
ZAPSIGN_DEFAULT_ENV=br
ZAPSIGN_WEBHOOK_SECRET=
ZAPSIGN_TIMEOUT_SEC=30
ZAPSIGN_SEND_AUTOMATIC_EMAIL=false
```

Nunca imprimir token em logs. Nunca retornar token da ZapSign ao frontend. Retornar somente `sign_url`, status e metadados seguros.

## 10. Segurança mínima obrigatória do backend

| Tema | Obrigatório |
|---|---|
| Secrets | `.env` nunca versionado/empacotado; rotação se exposto |
| Auth | access token curto, refresh token rotativo, cookie seguro |
| CSRF | obrigatório para mutações autenticadas |
| RBAC | toda rota de domínio com `@Permissions` |
| DTO | `whitelist` e `forbidNonWhitelisted` ativos |
| Rate limit | login, refresh, token público, upload e assinatura |
| Upload | validação server-side e scanner |
| Storage | privado, sem servir direto por Nginx |
| Auditoria | mutações, downloads, revelação de segredo, assinatura, ZapSign |
| Logs | sem PII desnecessária e sem segredo |
| Backup | antes de migrations e backfills |
| Readiness | banco, storage, migrations, RBAC, ClamAV e segredos |

## 11. Testes obrigatórios do backend

Antes do frontend completo:

```bash
npm run prisma:validate
npm run prisma:generate
npm run lint
npm run build
npm test -- --runInBand
npm run test:e2e
npm run ops:readiness -- --soft --json
```

Testes específicos que devem existir:

- auth login/refresh/logout/CSRF;
- RBAC para cada módulo;
- clients dashboard e scope mine/all;
- client assignments create/update/transfer/end;
- documents upload/download/status/public token;
- certificates encrypt/decrypt/rotate/download;
- contracts internal signature;
- contracts ZapSign create/sync/webhook;
- Acessórias date window atual;
- Acessórias rate limit/retry/lock;
- audit log para mutações sensíveis;
- backup/restore drill em alvo isolado.

## 12. Critérios de aceite do backend

O backend estará pronto para frontend completo quando:

- migrations aplicarem em banco limpo e banco homologação;
- seed RBAC criar roles/permissões corretas;
- todos os endpoints críticos tiverem testes;
- Acessórias respeitar ano/mês atual;
- ZapSign funcionar em sandbox e produção controlada;
- documentos e certificados usarem storage privado;
- secrets estiverem fora dos pacotes;
- readiness operacional passar;
- Swagger/contratos estiverem atualizados;
- `npm run lint`, `npm run build`, `npm test` e `npm run test:e2e` estiverem verdes.
