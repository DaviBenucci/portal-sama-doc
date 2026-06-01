# [PARCIAL] Documentação Técnica Atualizada do Portal Sama

## Visão geral

Esta documentação foi atualizada após a primeira entrega para refletir as decisões técnicas mais recentes do Portal Sama:

- O backend será migrado gradualmente de **PHP** para **Node.js + TypeScript + NestJS**.
- O banco principal continuará sendo **MySQL 8**, por compatibilidade com a VPS/EasyPanel e por ser adequado ao modelo relacional do sistema.
- O acesso ao banco será feito com **Prisma ORM**, substituindo gradualmente a criação manual de schema em `api/db.php`.
- **Redis** será considerado apenas como apoio para cache, rate limiting, filas, sessões temporárias e jobs; não como banco principal.
- O frontend será migrado gradualmente para **React + TypeScript + Vite + Tailwind CSS + shadcn/ui**.
- A segurança será centralizada no backend com autenticação robusta, RBAC, validação de escopo, auditoria, storage privado e upload seguro.

> Observação: esta versão atualiza a decisão técnica para **NestJS + TypeScript** no backend, **React + TypeScript + Vite** no frontend e **MySQL + Prisma** como base de dados e ORM.

---

## Novos documentos desta atualização

Observacao operacional: a topologia oficial para Bitbucket/EasyPanel passa a ter tres repositorios: `portal-sama-docs`/`portal-sama-doc` (documentacao), `portal-sama-api` e `portal-sama-web`. Em 2026-05-25, o remoto observado no Bitbucket para documentacao aparece como `portal-sama-doc`, enquanto o workspace local permanece `portal-sama-docs`. O repositorio de documentacao deve ser lido antes de qualquer alteracao na API ou no frontend.

| Documento | Finalidade |
|---|---|
| [`DECISOES_TECNICAS_POS_DOCUMENTACAO.md`](DECISOES_TECNICAS_POS_DOCUMENTACAO.md) | Consolida as decisões tomadas após a documentação inicial. |
| [`ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md`](ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md) | Define a arquitetura alvo com React, NestJS, Prisma, MySQL, Redis opcional e EasyPanel. |
| [`GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`](GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md) | Guia detalhado, com etapas, comandos, exemplos de código e referências aos arquivos atuais. |
| [`BANCO_DADOS_MYSQL_PRISMA.md`](BANCO_DADOS_MYSQL_PRISMA.md) | Detalha a escolha por banco relacional, MySQL 8, Prisma e modelagem inicial. |
| [`EASYPANEL_DEPLOY.md`](EASYPANEL_DEPLOY.md) | Explica como implantar a nova stack no EasyPanel/VPS. |
| [`ORDEM_IMPLEMENTACAO_DOCUMENTACOES.md`](ORDEM_IMPLEMENTACAO_DOCUMENTACOES.md) | Define a sequencia de implementacao para concluir documentos pendentes e parciais sem perder escopo. |
| [`AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`](AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD) | Painel obrigatorio do que falta para homologacao/producao, separacao Bitbucket/EasyPanel, percentual de prontidao e pendencias reais. |
| [`MAPEAMENTO_MIGRACAO_APIS.md`](MAPEAMENTO_MIGRACAO_APIS.md) | Mapeia endpoints PHP atuais para módulos e rotas NestJS futuras. |
| [`REFERENCIAS_TECNICAS.md`](REFERENCIAS_TECNICAS.md) | Lista referências oficiais das tecnologias recomendadas. |
| [`ux-ui-docs/README.md`](ux-ui-docs/README.md) | Referencia obrigatoria para melhoria de UI/UX, navegacao, wireframes e checklist de aceite visual. |

---

## Ordem recomendada de leitura atualizada

1. [`DECISOES_TECNICAS_POS_DOCUMENTACAO.md`](DECISOES_TECNICAS_POS_DOCUMENTACAO.md)
2. [`ARQUITETURA_ATUAL.md`](ARQUITETURA_ATUAL.md)
3. [`ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md`](ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md)
4. [`SEGURANCA.md`](SEGURANCA.md)
5. [`ux-ui-docs/README.md`](ux-ui-docs/README.md)
6. [`BANCO_DADOS_MYSQL_PRISMA.md`](BANCO_DADOS_MYSQL_PRISMA.md)
7. [`GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`](GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md)
8. [`ORDEM_IMPLEMENTACAO_DOCUMENTACOES.md`](ORDEM_IMPLEMENTACAO_DOCUMENTACOES.md)
9. [`EASYPANEL_DEPLOY.md`](EASYPANEL_DEPLOY.md)
10. [`AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`](AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD)
11. [`MAPEAMENTO_MIGRACAO_APIS.md`](MAPEAMENTO_MIGRACAO_APIS.md)
12. [`ROADMAP_REFATORACAO.md`](ROADMAP_REFATORACAO.md)
13. Documentos individuais em [`paginas/`](paginas/)

---

## QA de UI/UX antes de liberacao

A documentacao em [`ux-ui-docs/`](ux-ui-docs/) deve ser seguida na fase de melhoria de UI/UX apos a integracao com o Acessorias. O checklist [`ux-ui-docs/09-checklist-aceite.md`](ux-ui-docs/09-checklist-aceite.md) e o ultimo gate de QA antes de liberar o sistema para uso dos usuarios.

---

## Stack alvo oficial

### Frontend

```txt
React
TypeScript
Vite
React Router
Tailwind CSS
shadcn/ui
React Hook Form
Zod
TanStack Query
Axios
Zustand
Playwright
```

### Backend

```txt
Node.js
TypeScript
NestJS
Prisma ORM
MySQL 8
Swagger/OpenAPI
Jest
Supertest
BullMQ + Redis quando houver filas/jobs
Pino ou Winston para logs estruturados
```

### Segurança

```txt
Cookies HttpOnly, Secure e SameSite
Access token curto + refresh token seguro em cookie HttpOnly
RBAC com permissões granulares
Validação de escopo para evitar IDOR
Guards do NestJS
Rate limiting
Helmet
CORS restrito
Validação server-side
Upload seguro
Storage privado
ClamAV/scanner configuravel
Auditoria obrigatória
Criptografia de dados sensíveis
Gestão segura de segredos no EasyPanel
```

### Infraestrutura

```txt
EasyPanel
Docker
MySQL 8
Redis opcional
phpMyAdmin protegido
Nginx/reverse proxy gerenciado pelo EasyPanel
Backups automatizados
Volumes privados para arquivos sensíveis
```

---

## Mapa geral das páginas documentadas

| Página | Documento | Módulo | Prioridade |
|---|---|---|---|
| `index.html` | [`paginas/index.md`](paginas/index.md) | Autenticação | Urgente |
| `HOME/index.html` | [`paginas/home-index.md`](paginas/home-index.md) | Home | Baixa |
| `HOME/Inicio.html` | [`paginas/home-inicio.md`](paginas/home-inicio.md) | Home / Dashboard | Média |
| `Client/clientes.html` | [`paginas/client-clientes.md`](paginas/client-clientes.md) | Clientes | Baixa |
| `Client/painel.html` | [`paginas/client-painel.md`](paginas/client-painel.md) | Clientes / Documentos | Urgente |
| `Client/visao-colaborador.html` | [`paginas/client-visao-colaborador.md`](paginas/client-visao-colaborador.md) | Colaboradores | Média |
| `Client/visao-geral-clientes.html` | [`paginas/client-visao-geral-clientes.md`](paginas/client-visao-geral-clientes.md) | Clientes | Alta |
| `Client/visao-geral-colaboradores.html` | [`paginas/client-visao-geral-colaboradores.md`](paginas/client-visao-geral-colaboradores.md) | Colaboradores | Média |
| `DptClient/clientdpto.html` | [`paginas/dptclient-clientdpto.md`](paginas/dptclient-clientdpto.md) | Departamento / Clientes | Alta |
| `DptClient/certificados-digitais.html` | [`paginas/dptclient-certificados-digitais.md`](paginas/dptclient-certificados-digitais.md) | Certificados digitais | Urgente |
| `Manager/manager.html` | [`paginas/manager-manager.md`](paginas/manager-manager.md) | Gestor | Alta |
| `Manager/manager-colaborador.html` | [`paginas/manager-colaborador.md`](paginas/manager-colaborador.md) | Gestor / Colaboradores | Alta |
| `Manager/manager-calendar-config.html` | [`paginas/manager-calendar-config.md`](paginas/manager-calendar-config.md) | Calendário | Alta |
| `Manager/manager-transfers.html` | [`paginas/manager-transfers.md`](paginas/manager-transfers.md) | Transferências | Alta |
| `Legalizacao/legalizacao.html` | [`paginas/legalizacao-legalizacao.md`](paginas/legalizacao-legalizacao.md) | Legalização | Urgente |
| `Legalizacao/proposta.html` | [`paginas/legalizacao-proposta.md`](paginas/legalizacao-proposta.md) | Propostas | Alta |
| `Legalizacao/contrato.html` | [`paginas/legalizacao-contrato.md`](paginas/legalizacao-contrato.md) | Contratos | Urgente |
| `Legalizacao/assinatura.html` | [`paginas/legalizacao-assinatura.md`](paginas/legalizacao-assinatura.md) | Assinatura pública | Urgente |
| `Onboarding/processo.html` | [`paginas/onboarding-processo.md`](paginas/onboarding-processo.md) | Onboarding | Alta |
| `Onboarding/entrada-cliente.html` | [`paginas/onboarding-entrada-cliente.md`](paginas/onboarding-entrada-cliente.md) | Entrada de cliente | Alta |
| `Onboarding/documentos-cliente.html` | [`paginas/onboarding-documentos-cliente.md`](paginas/onboarding-documentos-cliente.md) | Documentos públicos | Urgente |
| `Onboarding/proposta-cliente.html` | [`paginas/onboarding-proposta-cliente.md`](paginas/onboarding-proposta-cliente.md) | Proposta pública | Alta |
| `Contabil/integra-ai.html` | [`paginas/contabil-integra-ai.md`](paginas/contabil-integra-ai.md) | Integra-AI | Urgente |
| `Auditoria/autoria.html` | [`paginas/auditoria-autoria.md`](paginas/auditoria-autoria.md) | Auditoria | Urgente |
| `Depto/modelo.html` | [`paginas/depto-modelo.md`](paginas/depto-modelo.md) | Departamento | Alta |
| `DEV/dev.html` | [`paginas/dev-dev.md`](paginas/dev-dev.md) | DEV/Administração | Urgente |
| `DEV/dev-colaborador.html` | [`paginas/dev-colaborador.md`](paginas/dev-colaborador.md) | DEV/Colaboradores | Urgente |
| `DEV/dev-novo-cliente.html` | [`paginas/dev-novo-cliente.md`](paginas/dev-novo-cliente.md) | DEV/Clientes | Alta |
| `DEV/dev-novo-colaborador.html` | [`paginas/dev-novo-colaborador.md`](paginas/dev-novo-colaborador.md) | DEV/Colaboradores | Urgente |
| `SolicitacaoAcesso/solicitacao-acesso.html` | [`paginas/solicitacao-acesso.md`](paginas/solicitacao-acesso.md) | Solicitação de Acesso | Alta |
| `TI/acesso-ti.html` | [`paginas/ti-acesso-ti.md`](paginas/ti-acesso-ti.md) | TI | Urgente |

---

## Arquivos atuais mais importantes para a migração

| Arquivo atual | Motivo | Destino sugerido |
|---|---|---|
| `auth.js` | Sessão, CSRF, logout e usuário no frontend. | `apps/web/src/services/auth.service.ts` e `apps/web/src/stores/auth.store.ts`. |
| `Login.js` | Login via `api/storage.php?action=auth_login`. | `AuthModule` e `POST /api-v2/auth/login`. |
| `api/auth.php` | Sessão e senha no backend PHP. | `AuthModule`, `UsersModule` e `RefreshTokenService`. |
| `api/storage.php` | Endpoint action-based grande e multifuncional. | Dividir em `AuthModule`, `UsersModule`, `ClientsModule`, `AuditModule`. |
| `api/db.php` | Criação imperativa de tabelas MySQL. | `prisma/schema.prisma` e migrations. |
| `api/client_documents.php` | Upload/download de documentos. | `DocumentsModule`. |
| `api/certificados.php` | Certificados digitais. | `CertificatesModule`. |
| `api/legalizacao.php` | Contratos, assinatura, tokens e legalização. | `LegalizationModule`, `ContractsModule`, `SignaturesModule`. |
| `api/propostas.php` | Propostas. | `ProposalsModule`. |
| `api/onboarding.php` | Onboarding, documentos e links públicos. | `OnboardingModule`, `DocumentsModule`, `PublicLinksModule`. |
| `api/access_requests.php` | Solicitações de acesso. | `AccessRequestsModule`. |
| `api/manager_workspace.php` | Área do gestor, overview/presença, histórico/vida da empresa, transferências e calendário. | `ManagersModule`, `TransfersModule`, `CalendarModule`. |
| `api/integra_ai.php` | Fluxo contábil/IA. | `AccountingModule`, `IntegraAiModule`. |
| `.env` | Segredos sensíveis. | Não versionar; usar variáveis no EasyPanel. |

---

## Leitura prática para implementação

Para começar a implementação, use diretamente:

1. [`GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`](GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md)
2. [`BANCO_DADOS_MYSQL_PRISMA.md`](BANCO_DADOS_MYSQL_PRISMA.md)
3. [`ORDEM_IMPLEMENTACAO_DOCUMENTACOES.md`](ORDEM_IMPLEMENTACAO_DOCUMENTACOES.md)
4. [`EASYPANEL_DEPLOY.md`](EASYPANEL_DEPLOY.md)
5. [`AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`](AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD)
6. [`SEGURANCA.md`](SEGURANCA.md)
7. [`MAPEAMENTO_MIGRACAO_APIS.md`](MAPEAMENTO_MIGRACAO_APIS.md)

## Topologia oficial de repositorios

```txt
portal-sama-docs  -> documentacao completa e fonte de verdade local
portal-sama-doc   -> alias remoto observado no Bitbucket em 2026-05-25
portal-sama-api   -> backend NestJS/API v2
portal-sama-web   -> frontend React/Vite
```

Ao trabalhar nos repositorios separados, a ordem correta e:

1. Abrir `portal-sama-docs`.
2. Ler `docs/INSTRUÇÕEDS_AI.MD` e os documentos relacionados.
3. Verificar status, pendencias, decisoes, testes e deploy.
4. Somente depois abrir `portal-sama-api` ou `portal-sama-web`.
5. Atualizar `portal-sama-docs` sempre que a mudanca tecnica alterar estado, teste, arquitetura, deploy ou pendencia.
