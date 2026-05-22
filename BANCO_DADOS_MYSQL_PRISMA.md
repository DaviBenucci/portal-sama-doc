# Banco de dados — MySQL relacional com Prisma ORM

## 1. Decisão

A decisão para o Portal Sama é usar:

```txt
MySQL 8 como banco principal
Prisma ORM como camada de acesso ao banco e migrations
Redis apenas como apoio operacional, se necessário
Nenhum banco não relacional no início
```

---

## 2. Por que banco relacional?

O Portal Sama é uma aplicação de gestão interna com muitas entidades conectadas:

- usuários;
- roles;
- permissões;
- clientes;
- colaboradores;
- departamentos;
- documentos;
- certificados digitais;
- propostas;
- contratos;
- assinaturas;
- onboarding;
- solicitações de acesso;
- auditoria.

Esses dados exigem:

- chaves estrangeiras;
- transações;
- constraints;
- índices;
- integridade referencial;
- consistência;
- histórico;
- rastreabilidade;
- relatórios.

Por isso, MySQL é mais adequado que MongoDB como banco principal.

---

## 3. Por que manter MySQL no EasyPanel?

Como a aplicação já roda em VPS pelo EasyPanel com MySQL e phpMyAdmin, manter MySQL reduz o risco da migração. O foco principal deve ser migrar PHP para TypeScript/NestJS, reforçar segurança e organizar módulos.

Trocar a linguagem do backend e o banco ao mesmo tempo aumentaria muito o risco operacional.

---

## 4. Quando usar Redis?

Redis pode ser adicionado como apoio, não como fonte principal da verdade.

| Uso | Aplicação no Portal Sama |
|---|---|
| Rate limiting | Login, tokens públicos, downloads sensíveis. |
| Sessões temporárias | Caso use sessão server-side distribuída. |
| Refresh token store | Rotação e revogação de refresh tokens. |
| Filas BullMQ | ClamAV, e-mails, integrações e jobs. |
| Cache | Dashboards, listas e consultas repetidas. |
| Alertas | Certificados próximos de vencer. |

---

## 5. O que salvar no MySQL

### Identidade e autorização

```txt
users
roles
permissions
user_roles
role_permissions
refresh_tokens
login_attempts
```

### Clientes e colaboradores

```txt
clients
collaborators
departments
client_managers
client_departments
```

Status em 2026-05-13 14:43: `Client` foi expandido no Prisma para cobrir os campos operacionais de `DEV/dev-novo-cliente.html` e telas de clientes (`rank`, regime tributario, id empresa, atividade, telefone, endereco, grupo, metadata e soft delete). `User` foi expandido para a primeira fatia de colaboradores (`position`, `phone`, `extension`, `metadata`, `archivedAt`, `archivedById`), usada pelo `CollaboratorsModule`. As migrations `20260513113000_expand_clients_profile` e `20260513124500_add_collaborator_profile_to_users` foram aplicadas no MySQL local apos baseline controlado; `prisma:migrate:status` passou e `prisma migrate diff` nao encontrou diferencas. Ainda faltam tabelas/vinculos formais para carteira, departamentos, gestores e backfill de dados legados.

Status em 2026-05-19 12:16: a primeira fatia de transferencias foi modelada no Prisma com `TransferSession`, usando a tabela legada `sama_transfer_sessions` e migration `20260519120000_add_transfer_sessions` com criacao/indices condicionais para reduzir risco em bancos onde a tabela ja exista. A carteira ainda permanece em `Client.metadata.departamentos/depts/dept_*` e `User.metadata.clientes/colaboradorId`; uma modelagem formal `client_departments`/carteira segue pendente apos backfill e validacao real.

Status em 2026-05-20 11:45: historico operacional do gestor usa os modelos Prisma `CompanyHistoryEntry` e `CompanyLifeEntry`, mapeados para `sama_company_history_entries` e `sama_company_life_entries`. As novas mutacoes do `ManagersModule` gravam nessas tabelas preservando `topics_json`, datas ISO em string e `data_json` com metadados de ator/acao; validacao MySQL/homologacao e backfill real continuam pendentes.

Status em 2026-05-20 12:30: o overview do gestor passou a ler `sama_user_presence` pelo modelo Prisma `UserPresence`, com migration `20260520123000_add_user_presence_model`. A tabela continua sendo alimentada pelos pings/streams legados ate existir contrato v2 proprio de presenca; validacao em MySQL/homologacao segue pendente.

Status em 2026-05-20 16:02: a exclusao de vencimentos do `CalendarModule` opera sobre os modelos ja mapeados `CalendarRule`, `CalendarRuleCompany` e `CalendarDueNotified`, removendo o vinculo empresa-regra, limpando notificacoes por regra/empresa/departamento e excluindo a regra quando ela fica sem vinculos. Nao houve nova migration nesta fatia; validacao com MySQL/homologacao segue pendente.

Status em 2026-05-21 10:33: a primeira fatia read-only do Integra-AI foi criada sem nova migration Prisma. O `AccountingModule` consulta as tabelas legadas `sama_integra_ai_jobs`, `sama_integra_ai_job_rows`, `sama_integra_ai_profiles` e `sama_integra_ai_profile_settings` por SQL parametrizado via Prisma, somente para leitura e com sanitizacao de paths/payloads antes da resposta. A modelagem Prisma formal das tabelas `sama_integra_ai_*`, importacao/parser/exportacao e historico completo seguem pendentes para uma etapa com validacao MySQL/homologacao.

### Documentos

```txt
documents
document_requirements
document_versions
document_status_history
document_access_logs
```

Status em 2026-05-12 17:27: os modelos Prisma `Document`, `DocumentRequirement`, `DocumentStatusHistory` e `PublicToken` estão em `portal-sama-api/prisma/schema.prisma`. `Document` cobre metadados, hash, status, departamento, origem, `storageKey` único e vínculo com cliente/usuário; a revisão de status grava `DocumentStatusHistory` e mantém `Document.metadata.review` por compatibilidade. O upload seguro registra `Document.metadata.scanner`, e o upload público por token usa `PublicToken.tokenHash`, expiração, revogação, escopo e rastreio de último uso. A migration formal inicial `20260512172700_init_current_schema` foi criada e validada com `prisma migrate deploy` em banco descartável; versionamento/acessos detalhados, backfills e baseline de bancos já sincronizados por `db push` ainda precisam de validação real.

### Onboarding

```txt
onboarding_processes
onboarding_steps
onboarding_documents
onboarding_status_history
public_links
```

### Legalização

```txt
legalization_processes
legalization_templates
proposals
proposal_items
contracts
contract_signatures
public_links
```

Status em 2026-05-13 08:57: a primeira fatia de propostas foi modelada no Prisma com `Proposal` e `ProposalStatus`, usando a tabela `proposals` e `PublicToken` para links públicos opacos. A migration `20260513085700_add_proposals` foi criada e validada por `prisma validate`, mas ainda precisa ser aplicada com `prisma migrate deploy` em MySQL de homologação porque o banco local não estava disponível.

Status em 2026-05-13 09:34: a primeira fatia de contratos/assinatura foi modelada no Prisma com `Contract` e `ContractStatus`, usando a tabela `contracts` e `PublicToken` para links públicos de assinatura. Atualizacao em 2026-05-13 10:50: o MySQL local foi baselined, as migrations constam aplicadas e `prisma migrate diff` nao encontrou diferencas; homologacao/producao ainda exigem procedimento com backup.

Status em 2026-05-13 15:59: a primeira fatia de processos de legalizacao foi modelada no Prisma com `LegalizationProcess` e `LegalizationStatus`, usando a tabela `legalization_processes`. As migrations `20260513160000_add_legalization_processes` e `20260513160500_add_missing_relation_indexes` foram aplicadas no MySQL local; `prisma:migrate:status` ficou atualizado e `prisma migrate diff --from-schema-datasource --to-schema-datamodel` nao encontrou diferencas.

Status em 2026-05-13 16:16: templates de cabecalho/rodape foram modelados no Prisma com `LegalizationTemplate` e `LegalizationTemplateType`, usando a tabela `legalization_templates`. A migration `20260513164000_add_legalization_templates` foi aplicada no MySQL local; `prisma:migrate:status` ficou atualizado e o diff Prisma nao encontrou diferencas.

### Certificados digitais

```txt
digital_certificates
certificate_access_logs
certificate_expiration_alerts
```

Status em 2026-05-12 19:50: `DigitalCertificate` foi adicionado ao Prisma com tabela `digital_certificates`, senha criptografada em `passwordCipher`, `storageKey` único para arquivo em storage privado e exclusão lógica por `deletedAt`. A migration `20260512195000_add_digital_certificates` foi criada e validada anteriormente; backfill de `sama_certificados_clientes` segue pendente.

### Solicitações de acesso

```txt
access_requests
access_request_approvals
access_request_status_history
```

Status em 2026-05-13 10:15: a primeira fatia de solicitacoes de acesso foi modelada no Prisma com `AccessRequest`, `AccessRequestProfile` e `AccessRequestStatus`, usando a tabela `access_requests`. Atualizacao em 2026-05-13 10:50: o MySQL local foi baselined, a migration consta aplicada e `prisma migrate diff` nao encontrou diferencas; homologacao/producao ainda exigem procedimento com backup e backfill.

### Notificacoes

```txt
notifications
```

Status em 2026-05-13 16:34: a tabela legada `notifications` foi mapeada no Prisma com `Notification`, preservando id numerico, datas ISO em string e campos `meta`, `event_type`, `entity_type`, `dedupe_key` e `expires_at`. A migration compativel `20260513170000_add_notifications_model` foi aplicada no MySQL local; `prisma:migrate:status` ficou atualizado e o diff Prisma nao encontrou diferencas.

Status em 2026-05-14 09:35: Web Push foi modelado com `BrowserPushSubscription` na tabela `browser_push_subscriptions`, vinculada opcionalmente a `User` por `user_id`, endpoint hasheado em `endpoint_hash`, chaves `p256dh/auth`, departamento, user agent, revogacao e contadores de falha. As migrations `20260514092000_add_browser_push_subscriptions` e `20260514093500_align_browser_push_updated_at` foram aplicadas no MySQL local; `prisma:migrate:status` ficou atualizado e `prisma migrate diff` nao encontrou diferencas.

### Auditoria

```txt
audit_logs
security_events
```

---

## 6. O que não salvar diretamente no MySQL

Não salvar arquivos binários como BLOB no banco:

```txt
documents.file_blob
contracts.pdf_blob
certificates.file_blob
```

Salvar apenas metadados:

```txt
id
client_id
original_name
mime_type
size
sha256_hash
storage_key
status
created_by
created_at
```

O arquivo físico deve ficar em storage privado:

```txt
/var/private/portal-sama/documents
/var/private/portal-sama/certificates
/var/private/portal-sama/contracts
```

---

## 7. Configuração inicial do Prisma com MySQL

Arquivo: `apps/api/prisma/schema.prisma`

```prisma
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}
```

Arquivo: `apps/api/.env.example`

```env
DATABASE_URL="mysql://portal_user:senha_forte@portal-sama-mysql:3306/portal_sama"
```

---

## 8. Modelagem inicial sugerida

```prisma
enum UserStatus {
  ACTIVE
  INACTIVE
  BLOCKED
}

enum DocumentStatus {
  PENDING
  APPROVED
  REJECTED
  ARCHIVED
}

model User {
  id           String     @id @default(uuid())
  name         String
  email        String     @unique
  passwordHash String
  status       UserStatus @default(ACTIVE)

  roles        UserRole[]
  auditLogs    AuditLog[]

  createdAt    DateTime   @default(now())
  updatedAt    DateTime   @updatedAt

  @@index([status])
}

model Role {
  id          String           @id @default(uuid())
  name        String           @unique
  description String?

  users       UserRole[]
  permissions RolePermission[]

  createdAt   DateTime         @default(now())
  updatedAt   DateTime         @updatedAt
}

model Permission {
  id          String           @id @default(uuid())
  key         String           @unique
  description String?

  roles       RolePermission[]
}

model UserRole {
  userId String
  roleId String

  user   User @relation(fields: [userId], references: [id], onDelete: Cascade)
  role   Role @relation(fields: [roleId], references: [id], onDelete: Cascade)

  @@id([userId, roleId])
}

model RolePermission {
  roleId       String
  permissionId String

  role         Role       @relation(fields: [roleId], references: [id], onDelete: Cascade)
  permission   Permission @relation(fields: [permissionId], references: [id], onDelete: Cascade)

  @@id([roleId, permissionId])
}

model Client {
  id           String     @id @default(uuid())
  razaoSocial  String
  nomeFantasia String?
  cnpj         String     @unique
  email        String?
  status       String     @default("ACTIVE")

  documents    Document[]
  documentRequirements DocumentRequirement[]

  createdAt    DateTime   @default(now())
  updatedAt    DateTime   @updatedAt

  @@index([status])
  @@index([razaoSocial])
}

model Document {
  id           String         @id @default(uuid())
  clientId     String
  type         String
  department   String?
  originalName String
  storageKey   String         @unique
  mimeType     String
  size         Int
  sha256Hash   String?
  status       DocumentStatus @default(PENDING)
  source       String         @default("manual")
  metadata     Json?
  uploadedById String?

  client       Client         @relation(fields: [clientId], references: [id])
  uploadedBy   User?          @relation("UploadedDocuments", fields: [uploadedById], references: [id], onDelete: SetNull)

  createdAt    DateTime       @default(now())
  updatedAt    DateTime       @updatedAt

  @@index([clientId])
  @@index([type])
  @@index([department])
  @@index([status])
  @@index([source])
  @@index([createdAt])
}

model DocumentRequirement {
  id          String   @id @default(uuid())
  clientId    String
  key         String
  label       String
  department  String
  required    Boolean  @default(false)
  active      Boolean  @default(true)
  createdById String?

  client      Client   @relation(fields: [clientId], references: [id])
  createdBy   User?    @relation("CreatedDocumentRequirements", fields: [createdById], references: [id], onDelete: SetNull)

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@unique([clientId, key])
  @@index([clientId])
  @@index([department])
  @@index([active])
}

model AuditLog {
  id         String   @id @default(uuid())
  userId     String?
  action     String
  module     String
  entityType String?
  entityId   String?
  ipAddress  String?
  userAgent  String?
  metadata   Json?
  createdAt  DateTime @default(now())

  user       User?    @relation(fields: [userId], references: [id])

  @@index([userId])
  @@index([module])
  @@index([entityType, entityId])
  @@index([createdAt])
}
```

Status em 2026-05-13 09:34: o schema real em `portal-sama-api/prisma/schema.prisma` também contém `DigitalCertificate`, `Proposal`, `ProposalStatus`, `Contract` e `ContractStatus`. Este bloco é uma referência resumida; a fonte de verdade para campos e índices é o arquivo Prisma versionado.

---

## 9. Comandos de migration

### Desenvolvimento

```bash
cd apps/api
npx prisma migrate dev --name init
npx prisma generate
```

### Produção

```bash
cd apps/api
npx prisma migrate deploy
npx prisma generate
```

Em produção, usar `migrate deploy`, não `migrate dev`.

---

## 10. Relação com o código atual

| Arquivo atual | Ação recomendada |
|---|---|
| `api/db.php` | Mapear tabelas e substituir por `schema.prisma`. |
| `api/migrate.php` | Substituir por migrations Prisma. |
| `api/migrate_sqlite_to_mysql.php` | Validar se ainda é necessário; transformar em script de migração controlado, se aplicável. |
| `api/storage.php` | Separar dados por domínio e modelar tabelas específicas. |
| `api/client_documents.php` | Criar tabela `documents` e tabelas de histórico/acesso. |
| `api/certificados.php` | Criar `digital_certificates` e logs de acesso. |
| `api/propostas.php` | Criar `proposals` e reutilizar `PublicToken` para resposta pública por token opaco. |
| `api/onboarding.php` | Criar `onboarding_processes`, `onboarding_steps`, `public_links`. |
| `api/legalizacao.php` | Criar `legalization_processes`, `legalization_templates`, `contracts`, `contract_signatures`, `public_links`. |
| `api/notifications.php` | Mapear `notifications` e evoluir stream/SSE em modulo proprio. |

---

## 11. phpMyAdmin

phpMyAdmin pode continuar sendo usado para inspeção e manutenção controlada, mas não deve ser a fonte principal de alteração de estrutura em produção.

Recomendações:

- não expor publicamente sem proteção;
- usar senha forte;
- restringir por IP, se possível;
- não expor porta do MySQL publicamente;
- criar migrations para alterações estruturais;
- evitar alterações manuais em produção.

---

## 12. Estratégia de migração do banco atual

1. Fazer backup do MySQL atual.
2. Levantar tabelas atuais criadas em `api/db.php`.
3. Criar schema Prisma equivalente.
4. Criar banco de homologação.
5. Rodar migrations em homologação.
6. Criar scripts de migração para renomeações ou normalizações.
7. Validar contagem de registros.
8. Validar login, clientes, documentos, certificados, contratos e onboarding.
9. Validar auditoria.
10. Aplicar em produção somente após validação.
