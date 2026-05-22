# Roadmap de Refatoração do Portal Sama — TypeScript, NestJS, React, MySQL e EasyPanel

## 1. Visão geral

Este roadmap substitui a recomendação anterior baseada em Laravel como caminho principal de evolução. A direção atual definida para o Portal Sama é:

```txt
Frontend alvo: React + TypeScript + Vite + Tailwind CSS + shadcn/ui
Backend alvo: Node.js + TypeScript + NestJS
Banco principal: MySQL 8
ORM: Prisma
Infraestrutura: EasyPanel + Docker/Nixpacks + Nginx/reverse proxy
Banco auxiliar: Redis apenas quando necessário
Storage: volume privado ou S3 compatível no futuro
```

A estratégia recomendada é uma migração gradual, mantendo `/api` em PHP enquanto `/api-v2` em NestJS substitui os módulos mais críticos.

Atualizacao 2026-05-20 12:30: o eixo de Manager dashboards avancou com `GET /api-v2/managers/overview`, leitura/escrita inicial de historico operacional em `ManagersModule`, presenca da equipe em `/manager` e formularios React em `/manager/historico`; ainda faltam homologacao MySQL, migrations/seed RBAC em ambiente real, Playwright e desligamento controlado do legado.

---

## 2. Etapa 1 — Correções imediatas de segurança no sistema atual

Prioridade: **Urgente**

Antes de iniciar a migração estrutural, corrigir riscos que podem afetar produção imediatamente.

### Ações

- Remover `.env` do repositório/pacote e rotacionar segredos.
- Garantir que `.git`, ambientes virtuais, dependências locais e arquivos temporários não sejam enviados para produção.
- Desabilitar exposição de erros detalhados em produção, especialmente em arquivos PHP públicos.
- Revisar páginas que podem ser abertas diretamente pelo navegador.
- Garantir que endpoints PHP validem sessão antes de retornar dados.
- Validar permissões no backend, não apenas no JavaScript.
- Restringir phpMyAdmin com senha forte, IP allowlist ou autenticação adicional.
- Não expor porta pública do MySQL sem necessidade controlada.
- Criar backup do banco e dos arquivos enviados.

### Arquivos prioritários

```txt
.env
api/db.php
api/auth.php
api/storage.php
api/client_documents.php
api/certificados.php
api/legalizacao.php
api/onboarding.php
api/access_requests.php
SolicitacaoAcesso/enviar_acesso.php
```

Referência: [`SEGURANCA.md`](./SEGURANCA.md).

---

## 3. Etapa 2 — Fundação do backend NestJS

Prioridade: **Alta**

Criar a API `/api-v2` em paralelo ao backend PHP atual.

### Ações

- Criar projeto `portal-sama-api`.
- Configurar NestJS com TypeScript.
- Configurar prefixo global `/api-v2`.
- Configurar `@nestjs/config` com validação de `.env`.
- Configurar Swagger/OpenAPI para desenvolvimento.
- Configurar Helmet, CORS restrito e rate limiting.
- Configurar logs estruturados.
- Configurar tratamento global de erros.

### Entregáveis

```txt
portal-sama-api/src/main.ts
portal-sama-api/src/app.module.ts
portal-sama-api/src/config/env.schema.ts
portal-sama-api/src/common/guards/
portal-sama-api/src/common/filters/
portal-sama-api/src/common/interceptors/
```

Referência: [`GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`](./GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md).

---

## 4. Etapa 3 — MySQL + Prisma

Prioridade: **Alta**

Manter MySQL como banco relacional principal e adicionar Prisma para migrations e acesso tipado.

### Ações

- Criar serviço MySQL no EasyPanel ou reaproveitar o serviço existente.
- Criar usuário de banco específico para a API.
- Configurar `DATABASE_URL`.
- Criar `schema.prisma` com provider `mysql`.
- Criar modelos iniciais: `User`, `Role`, `Permission`, `Client`, `Document`, `AuditLog`.
- Criar migrations.
- Usar `prisma migrate deploy` em produção.

### Entregáveis

```txt
portal-sama-api/prisma/schema.prisma
portal-sama-api/prisma/migrations/
portal-sama-api/src/database/prisma.service.ts
portal-sama-api/src/database/database.module.ts
```

Referência: [`BANCO_DADOS_MYSQL_PRISMA.md`](./BANCO_DADOS_MYSQL_PRISMA.md).

---

## 5. Etapa 4 — Autenticação, sessão e usuários

Prioridade: **Crítica**

A autenticação deve ser migrada antes dos módulos de negócio.

### Ações

- Migrar lógica de login atual de `Login.js`, `auth.js`, `api/auth.php` e actions de autenticação em `api/storage.php`.
- Criar `AuthModule` em NestJS.
- Criar `UsersModule`.
- Implementar login com access token curto.
- Armazenar refresh token em cookie `HttpOnly`, `Secure` e `SameSite`.
- Evitar refresh token em `localStorage`.
- Implementar logout e revogação de refresh token.
- Implementar endpoint `/api-v2/auth/me`.
- Auditar login, logout e falhas.
- Aplicar rate limit em login.

### Endpoints alvo

```txt
POST /api-v2/auth/login
POST /api-v2/auth/logout
POST /api-v2/auth/refresh
GET  /api-v2/auth/me
```

### Páginas impactadas

```txt
index.html
HOME/index.html
HOME/Inicio.html
```

Referências:

- [`paginas/index.md`](./paginas/index.md)
- [`GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`](./GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md)

---

## 6. Etapa 5 — RBAC, permissões e escopo

Prioridade: **Crítica**

### Ações

- Criar tabelas `roles`, `permissions`, `user_roles`, `role_permissions`.
- Criar `PermissionsGuard`.
- Criar decorator `@Permissions()`.
- Criar validação de escopo por recurso.
- Definir permissões por módulo.
- Revisar todos os endpoints para impedir IDOR.

### Perfis iniciais

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

### Regra essencial

```txt
Permissão geral não autoriza acesso automático ao recurso.
O backend precisa validar se o usuário tem vínculo com o cliente, documento, proposta, contrato ou solicitação acessada.
```

Status em 2026-05-12 16:13: a base de RBAC foi ampliada com `UsersModule`, `RolesModule`, `PermissionsModule`, rotas `GET` protegidas por `users.read`, `roles.read` e `permissions.read`, seed inicial de roles/permissões em `portal-sama-api/prisma/seed.ts` e mutações principais de usuários, roles e permissões com CSRF/auditoria. Ainda faltam migrations/seed atualizado em MySQL, migração de usuários legados, integração DEV/admin e validação de escopo por recurso com dados reais.

Atualização em 2026-05-12 17:27: a migration Prisma formal inicial `20260512172700_init_current_schema` foi criada e validada com `prisma migrate deploy` em banco MySQL descartável. O seed RBAC atualizado foi validado no mesmo banco, com 9 roles, 37 permissões e 124 vínculos. Bancos já sincronizados anteriormente por `prisma db push` precisam de baseline controlado antes de `migrate deploy`.

Referência: [`SEGURANCA.md`](./SEGURANCA.md).

---

## 7. Etapa 6 — Auditoria centralizada

Prioridade: **Crítica**

### Ações

- Criar `AuditModule`.
- Criar serviço `AuditService`.
- Registrar ações críticas.
- Criar endpoints de consulta de logs.
- Criar filtros por usuário, módulo, entidade, ação e período.
- Impedir edição/exclusão de logs via interface.

### Endpoints alvo

```txt
GET /api-v2/audit/logs
GET /api-v2/audit/logs/:id
GET /api-v2/audit/entities/:entityType/:entityId
```

### Página impactada

```txt
Auditoria/autoria.html
```

Referência: [`paginas/auditoria-autoria.md`](./paginas/auditoria-autoria.md).

---

## 8. Etapa 7 — Documentos e storage privado

Prioridade: **Crítica**

Status em 2026-05-12 13:52: etapa iniciada com `DocumentsModule` em `portal-sama-api/src/modules/documents/`. Foram criados endpoints de template, checklist/listagem por cliente, detalhe, upload autenticado, upload público por `PublicToken`, emissão/listagem/revogação administrativa de tokens públicos, download, resumo de pendências, requisitos customizados, revisao de status, quarentena/scanner configuravel e arquivamento lógico, com JWT/permissões, CSRF em mutações sensíveis, validação de arquivo, storage privado e auditoria. Ainda faltam validacao ClamAV strict no host, migrations/MySQL real, storage persistente, rate limit distribuído e integração frontend.

### Ações

- Criar `DocumentsModule`.
- Criar validação de extensão, MIME type, tamanho e assinatura do arquivo.
- Validar ClamAV ou equivalente em produção com `SAMA_UPLOAD_SCAN_MODE=strict`.
- Armazenar arquivos fora da pasta pública.
- Salvar apenas metadados no MySQL.
- Criar download por endpoint protegido.
- Registrar auditoria de upload/download/delete.

### Arquivos impactados

```txt
api/client_documents.php
api/storage.php
Client/painel.js
Onboarding/documentos-cliente.html
Legalizacao/contrato.html
```

### Endpoints alvo

```txt
POST   /api-v2/documents/clients/:clientId/upload
POST   /api-v2/documents/public-upload
GET    /api-v2/documents/public-tokens
POST   /api-v2/documents/public-tokens
DELETE /api-v2/documents/public-tokens/:id
GET    /api-v2/documents
GET    /api-v2/documents/:id
GET    /api-v2/documents/:id/download
DELETE /api-v2/documents/:id
PATCH  /api-v2/documents/:id/status
```

Referência: [`GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`](./GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md).

---

## 9. Etapa 8 — Certificados digitais

Prioridade: **Crítica**

### Ações

- Criar `CertificatesModule`.
- Migrar lógica de `api/certificados.php`.
- Criptografar campos sensíveis quando necessário.
- Nunca retornar senha de certificado pela API.
- Auditar todo acesso e download.
- Criar alertas de vencimento.

### Página impactada

```txt
DptClient/certificados-digitais.html
```

Referência: [`paginas/dptclient-certificados-digitais.md`](./paginas/dptclient-certificados-digitais.md).

---

## 10. Etapa 9 — Legalização, propostas, contratos e assinaturas

Prioridade: **Alta**

### Ações

- Criar `LegalizationModule`.
- Criar `ProposalsModule`.
- Criar `ContractsModule`.
- Criar `SignaturesModule`.
- Migrar `api/legalizacao.php` e serviços relacionados.
- Controlar status por enums.
- Implementar token público opaco com expiração para assinatura.
- Auditar aprovação, geração e assinatura.

### Páginas impactadas

```txt
Legalizacao/legalizacao.html
Legalizacao/proposta.html
Legalizacao/contrato.html
Legalizacao/assinatura.html
```

Referências:

- [`paginas/legalizacao-legalizacao.md`](./paginas/legalizacao-legalizacao.md)
- [`paginas/legalizacao-proposta.md`](./paginas/legalizacao-proposta.md)
- [`paginas/legalizacao-contrato.md`](./paginas/legalizacao-contrato.md)
- [`paginas/legalizacao-assinatura.md`](./paginas/legalizacao-assinatura.md)

---

## 11. Etapa 10 — Onboarding

Prioridade: **Alta**

### Ações

- Criar `OnboardingModule`.
- Migrar `api/onboarding.php`.
- Modelar processos, etapas, documentos obrigatórios e histórico de status.
- Criar rotas públicas por token para envio de documentos/propostas.
- Auditar ações críticas.

### Páginas impactadas

```txt
Onboarding/processo.html
Onboarding/entrada-cliente.html
Onboarding/documentos-cliente.html
Onboarding/proposta-cliente.html
```

---

## 12. Etapa 11 — Solicitação de acesso e TI

Prioridade: **Alta**

### Ações

- Criar `AccessRequestsModule`.
- Criar `TiModule`.
- Migrar `api/access_requests.php`.
- Criar fluxo de solicitação, aprovação, rejeição e histórico.
- Adicionar notificações por fila no futuro.

### Páginas impactadas

```txt
SolicitacaoAcesso/solicitacao-acesso.html
TI/acesso-ti.html
```

---

## 13. Etapa 12 — Frontend React + TypeScript

Prioridade: **Média/Alta**

### Ações

- Criar `portal-sama-web` com Vite React TypeScript.
- Configurar Tailwind CSS e shadcn/ui.
- Criar layout base.
- Criar componentes reutilizáveis.
- Criar serviços de API com Axios.
- Adicionar TanStack Query.
- Migrar formulários com React Hook Form + Zod.
- Criar store de autenticação com Zustand.

### Componentes base

```txt
AppLayout
Sidebar
Header
DataTable
StatusBadge
UploadField
DownloadButton
ConfirmDialog
FormSection
PermissionGate
AuditTimeline
```

### Ordem de migração das telas

```txt
1. Login
2. Home
3. Documentos
4. Certificados digitais
5. Solicitações de acesso/TI
6. Legalização
7. Onboarding
8. Clientes/colaboradores
9. Manager dashboards
10. Auditoria
11. DEV/admin
```

---

## 14. Etapa 13 — Testes, CI/CD e observabilidade

Prioridade: **Alta**

### Ações

- Criar testes unitários para services.
- Criar testes de API com Jest/Supertest.
- Criar testes E2E com Playwright.
- Criar pipeline GitHub Actions.
- Configurar ESLint e Prettier.
- Configurar Sentry.
- Configurar logs estruturados.
- Adicionar OpenTelemetry quando a API estiver estável.

---

## 15. Matriz de prioridade

| Prioridade | Módulo | Motivo |
|---|---|---|
| Crítica | Auth | Base de segurança de todo o sistema |
| Crítica | RBAC/Permissions | Impede acesso indevido por perfil/recurso |
| Crítica | Documents | Manipula arquivos empresariais sensíveis |
| Crítica | Certificates | Manipula certificados digitais e dados altamente sensíveis |
| Crítica | Audit | Necessário para rastreabilidade e investigação |
| Alta | Legalização/Contratos | Fluxo jurídico e documental sensível |
| Alta | Onboarding | Fluxo externo com documentos de clientes |
| Alta | Solicitação de acesso/TI | Pode abrir permissões indevidas se mal protegido |
| Média | Clientes/Colaboradores | Dados sensíveis e relacionamentos operacionais |
| Média | Manager dashboards | Alto volume de dados e filtros |
| Média | Frontend React | Melhora manutenção e UX, mas backend seguro vem primeiro |

---

## 16. Conclusão

O caminho recomendado é evoluir o Portal Sama por risco e não por ordem visual de telas. Primeiro devem entrar autenticação, autorização, auditoria e documentos seguros. Depois, os módulos de certificados, legalização, onboarding e solicitações. O frontend React deve ser criado em paralelo, mas a segurança real precisa nascer no backend NestJS.

## 17. Funcionalidades futuras pós-plano principal

### Integração Acessórias — Entregas e Vencimentos

- **Status:** Planejada para fase futura.
- **Referência:** `docs/integracao_acessorias_entregas_vencimentos.md`.
- **Regra:** não implementar antes de concluir as bases principais de migração, autenticação, RBAC, auditoria, notificações, planilhas departamentais, integrações e frontend React.
- **Escopo futuro:** consultar entregas do Acessórias, gerar planilha Fiscal por competência, atualizar status `Acessórias`, criar Central de Vencimentos, notificar colaboradores e registrar auditoria das sincronizações.
- **Próxima ação atual:** manter como pendência documentada e continuar a migração principal.
