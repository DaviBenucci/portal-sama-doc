# [PARCIAL] Arquitetura alvo com TypeScript, NestJS, MySQL e EasyPanel

## 1. Objetivo

Definir a arquitetura futura do Portal Sama após a decisão de migrar o backend PHP para **Node.js + TypeScript + NestJS**, mantendo **MySQL 8** como banco relacional principal.

Esta arquitetura considera os principais pontos do código atual:

- `api/storage.php`: endpoint action-based muito grande, com várias responsabilidades.
- `api/db.php`: criação imperativa de tabelas MySQL.
- `api/client_documents.php`: upload/download de documentos.
- `api/certificados.php`: certificados digitais.
- `api/legalizacao.php`: legalização, contratos, tokens e assinatura.
- `api/onboarding.php`: onboarding, tokens públicos e documentos.
- `api/access_requests.php`: solicitações de acesso.
- `auth.js` e `Login.js`: autenticação no frontend legado.
- `global.js`, `DEV/dev-data.js` e `Client/painel.js`: JavaScript extenso e acoplado à interface.

---

## 2. Visão macro

```txt
Usuário
  ↓
EasyPanel / Reverse Proxy / HTTPS
  ↓
Frontend React + Vite
  ↓
API NestJS em /api-v2
  ↓
┌──────────────────────┬──────────────────────┬──────────────────────┐
│ MySQL 8              │ Redis opcional        │ Storage privado      │
│ Dados persistentes   │ Cache/filas/sessões   │ Arquivos sensíveis   │
└──────────────────────┴──────────────────────┴──────────────────────┘
```

---

## 3. Estrutura recomendada de repositório

Atualizacao 2026-05-22: para deploy operacional via Bitbucket/EasyPanel, a topologia oficial passa a ser tres repositorios separados:

```txt
portal-sama-docs  -> documentacao completa e fonte de verdade
portal-sama-api   -> backend NestJS/API v2
portal-sama-web   -> frontend React/Vite
```

Regra operacional: antes de alterar `portal-sama-api` ou `portal-sama-web`, ler a documentacao vigente em `portal-sama-docs`.

A estrutura monorepo abaixo permanece como referencia historica/organizacional para entender a arquitetura alvo, mas nao e a topologia operacional escolhida para o EasyPanel.

```txt
portal-sama/
  apps/
    api/
      src/
      prisma/
      test/
      package.json
      Dockerfile

    web/
      src/
      public/
      package.json
      vite.config.ts
      Dockerfile

  packages/
    shared/
      src/

  docs/
  docker/
  .github/
  .env.example
  README.md
```

### Por que essa estrutura?

- Mantém frontend e backend no mesmo repositório.
- Facilita migração gradual.
- Permite compartilhar tipos, constantes de permissões e contratos.
- Facilita CI/CD.
- Evita espalhar regras de negócio entre HTML, JS e PHP.

---

## 4. Estrutura do backend NestJS

```txt
apps/api/src/
  main.ts
  app.module.ts

  common/
    decorators/
      current-user.decorator.ts
      permissions.decorator.ts
      public.decorator.ts
    guards/
      jwt-auth.guard.ts
      permissions.guard.ts
      public-token.guard.ts
    filters/
      http-exception.filter.ts
    interceptors/
      audit.interceptor.ts
      request-id.interceptor.ts
    pipes/
      zod-validation.pipe.ts

  config/
    env.schema.ts
    cors.config.ts
    storage.config.ts
    security.config.ts

  database/
    prisma.service.ts
    database.module.ts

  modules/
    auth/
    users/
    roles/
    permissions/
    clients/
    collaborators/
    departments/
    documents/
    onboarding/
    legalization/
    proposals/
    contracts/
    signatures/
    certificates/
    access-requests/
    managers/
    accounting/
    integra-ai/
    audit/
    notifications/
    health/
```

---

## 5. Mapeamento do código atual para módulos alvo

| Arquivo atual | Função atual | Módulo NestJS alvo |
|---|---|---|
| `api/storage.php` | Login, usuários, dados, auditoria, actions diversas. | `AuthModule`, `UsersModule`, `ClientsModule`, `AuditModule`. |
| `api/auth.php` | Sessão, senha e usuário autenticado. | `AuthModule`, `UsersModule`. |
| `api/db.php` | Criação de tabelas e conexão MySQL. | `Prisma`, `DatabaseModule`. |
| `api/client_documents.php` | Documentos do cliente. | `DocumentsModule`. |
| `api/certificados.php` | Certificados digitais. | `CertificatesModule`, `DocumentsModule`. |
| `api/legalizacao.php` | Legalização, contrato, tokens públicos. | `LegalizationModule`, `ContractsModule`, `SignaturesModule`, `PublicLinksModule`. |
| `api/propostas.php` | Propostas. | `ProposalsModule`. |
| `api/onboarding.php` | Onboarding, documentos e links públicos. | `OnboardingModule`, `DocumentsModule`, `PublicLinksModule`. |
| `api/access_requests.php` | Solicitação/aprovação de acesso. | `AccessRequestsModule`, `TiModule`. |
| `api/manager_workspace.php` | Gestor, calendário e transferências. | `ManagersModule`, `CalendarModule`, `TransfersModule`. |
| `api/integra_ai.php` e `api/integra_ai_*` | Fluxo contábil/IA. | `AccountingModule`, `IntegraAiModule`, `JobsModule`. |

---

## 6. Estrutura do frontend React

```txt
apps/web/src/
  main.tsx
  App.tsx

  app/
    providers.tsx
    router.tsx

  layouts/
    AppLayout.tsx
    PublicLayout.tsx

  components/
    ui/
    layout/
    forms/
    data-table/
    upload/
    audit/
    permissions/

  pages/
    auth/
    home/
    clients/
    managers/
    onboarding/
    legalization/
    accounting/
    audit/
    access-requests/
    ti/
    dev/

  services/
    api.ts
    auth.service.ts
    client.service.ts
    document.service.ts
    certificate.service.ts
    proposal.service.ts
    contract.service.ts
    onboarding.service.ts
    access-request.service.ts
    audit.service.ts

  stores/
    auth.store.ts
    layout.store.ts
    permissions.store.ts

  schemas/
    auth.schema.ts
    client.schema.ts
    document.schema.ts
    proposal.schema.ts
    contract.schema.ts
    access-request.schema.ts
```

---

## 7. Mapeamento de telas atuais para rotas React

| Página atual | Rota React alvo | Página React sugerida | Backend relacionado |
|---|---|---|---|
| `index.html` | `/login` | `pages/auth/LoginPage.tsx` | `AuthModule` |
| `HOME/Inicio.html` | `/home` | `pages/home/HomePage.tsx` | `AuthModule`, `UsersModule` |
| `Client/clientes.html` | `/clientes` | `pages/clients/ClientsPage.tsx` | `ClientsModule` |
| `Client/painel.html` | `/clientes/:clientId/painel` | `pages/clients/ClientDashboardPage.tsx` | `ClientsModule`, `DocumentsModule` |
| `DptClient/certificados-digitais.html` | `/certificados-digitais` | `pages/certificates/CertificatesPage.tsx` | `CertificatesModule`, `AuditModule` |
| `Manager/manager.html` | `/manager` | `pages/managers/ManagerDashboardPage.tsx` | `ManagersModule` |
| `Manager/manager-transfers.html` | `/manager/transferencias` | `pages/managers/TransfersPage.tsx` | `TransfersModule` |
| `Legalizacao/legalizacao.html` | `/legalizacao` | `pages/legalization/LegalizationPage.tsx` | `LegalizationModule` |
| `Legalizacao/proposta.html` | `/legalizacao/propostas/:id` | `pages/legalization/ProposalPage.tsx` | `ProposalsModule` |
| `Legalizacao/contrato.html` | `/legalizacao/contratos/:id` | `pages/legalization/ContractPage.tsx` | `ContractsModule` |
| `Legalizacao/assinatura.html` | `/assinatura/:token` | `pages/legalization/PublicSignaturePage.tsx` | `SignaturesModule`, `PublicLinksModule` |
| `Onboarding/processo.html` | `/onboarding/processos` | `pages/onboarding/OnboardingProcessesPage.tsx` | `OnboardingModule` |
| `Onboarding/documentos-cliente.html` | `/onboarding/publico/documentos/:token` | `pages/onboarding/PublicDocumentsPage.tsx` | `OnboardingModule`, `DocumentsModule` |
| `Contabil/integra-ai.html` | `/contabil/integra-ai` | `pages/accounting/IntegraAiPage.tsx` | `AccountingModule`, `IntegraAiModule` |
| `Auditoria/autoria.html` | `/auditoria` | `pages/audit/AuditPage.tsx` | `AuditModule` |
| `SolicitacaoAcesso/solicitacao-acesso.html` | `/solicitacao-acesso` | `pages/access-requests/AccessRequestPage.tsx` | `AccessRequestsModule` |
| `TI/acesso-ti.html` | `/ti/acessos` | `pages/ti/TiAccessPage.tsx` | `TiModule`, `AccessRequestsModule` |

---

## 8. Estratégia de compatibilidade temporária

Durante a migração:

```txt
Frontend legado HTML/JS -> /api PHP
Frontend React novo     -> /api-v2 NestJS
```

Também é possível fazer páginas legadas consumirem `/api-v2` antes de serem migradas para React, reduzindo risco e permitindo migrar o backend por domínio.

---

## 9. Padrões obrigatórios para novos módulos

Todo novo módulo deve seguir:

```txt
controller -> service -> repository/prisma -> banco/storage
```

E deve incluir:

- DTOs/validação;
- guards de autenticação;
- guards de permissão;
- validação de escopo;
- auditoria;
- testes;
- documentação Swagger/OpenAPI;
- tratamento de erros padronizado.

---

## 10. Decisão final de arquitetura

A arquitetura final recomendada é um **monólito modular em NestJS**, com frontend React separado e banco MySQL relacional. Essa abordagem é mais simples de operar no EasyPanel, reduz risco e permite refatoração gradual sem cair na complexidade de microserviços.
