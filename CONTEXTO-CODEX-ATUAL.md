# CONTEXTO CODEX ATUAL

Atualizado em: 2026-06-11
Sessao atual: Homologacao EasyPanel Acessorias - terceira leva com E2E/Playwright local

## Objetivo ativo

Continuar a homologacao EasyPanel pos-integracao da Acessorias, seguindo:

- `README-HOMOLOGACAO-EASYPANEL.md`
- `PROMPT-CODEX-HOMOLOGACAO-EASYPANEL-ACESSORIAS.md`
- `AUDITORIA-DEPLOY-17-HOMOLOGACAO-EASYPANEL-ACESSORIAS-POS-INTEGRACAO.md`
- `AUDITORIA-DEPLOY-18-PLANO-CORRECAO-HOMOLOGACAO-EASYPANEL.md`
- `AUDITORIA-DEPLOY-19-TEMPLATE-EXECUCAO-HOMOLOGACAO-EASYPANEL.md`
- registro em `AUDITORIA-DEPLOY-19-EXECUCAO-HOMOLOGACAO-EASYPANEL.md`

Nao recomecar a auditoria do zero. Prosseguir das pendencias listadas abaixo.

## Branches atuais

- `portal-sama-docs`: `fix/deploy-readiness-go-live`
- `portal-sama-api`: `main`
- `portal-sama-web`: `main`

## Contexto lido nesta etapa

- `README-HOMOLOGACAO-EASYPANEL.md`
- `PROMPT-CODEX-HOMOLOGACAO-EASYPANEL-ACESSORIAS.md`
- `AUDITORIA-DEPLOY-17-HOMOLOGACAO-EASYPANEL-ACESSORIAS-POS-INTEGRACAO.md`
- `AUDITORIA-DEPLOY-18-PLANO-CORRECAO-HOMOLOGACAO-EASYPANEL.md`
- `AUDITORIA-DEPLOY-19-TEMPLATE-EXECUCAO-HOMOLOGACAO-EASYPANEL.md`
- `00-LEIA-ME-PARA-IA-MVP.md`
- `01-ESTADO-ATUAL-CODIGO-DOCUMENTACAO.md`
- `02-PLANO-FECHAMENTO-MVP.md`
- `03-CONTRATO-ACESSORIAS-OPERACIONAL.md`
- `04-DIVERGENCIAS-DOCS-CODIGO.md`
- `05-PROMPT-CODEX-FECHAMENTO-MVP.md`
- `06-NOTIFICACOES-WEB-PUSH-MVP.md`
- `07-PROMPT-CODEX-PIPELINE-ACESSORIAS-PERSISTENCIA-PUSH-DEV.md`
- `AUDITORIA-DEPLOY-13-HOMOLOGACAO-LOCAL-ACESSORIAS-DOCKER.md`
- `AUDITORIA-DEPLOY-16-EXECUCAO-CORRECOES-GO-LIVE.md`
- `evidencias/entrada/Homolacao-portal-sama-easypanel.txt`
- screenshots em `evidencias/screenshots/`

## Fase 0

Concluida.

- Worktrees conferidos em docs/API/web.
- Validacao inicial da API: lint OK, build OK, testes OK.
- Validacao inicial do Web: lint OK, build OK, contratos OK.
- Screenshots/evidencias analisados para reproduzir os sintomas documentados.

## Implementado nesta sessao

### API

- Adicionado status Prisma `PENDING_DEV` em `AccessRequestStatus`.
- Criada migration:
  `prisma/migrations/20260611130000_add_access_request_pending_dev/migration.sql`.
- Fluxo de solicitacoes de acesso atualizado:
  - colaborador cria `PENDING_MANAGER`;
  - gestor aprova colaborador e encaminha para `PENDING_DEV`;
  - gestor solicitando para si cria pedido `PENDING_DEV`;
  - DEV/ADMIN/TI fazem aprovacao/rejeicao final;
  - listagem de pendencias separa `PENDING_MANAGER` para gestor e `PENDING_DEV` para decisores privilegiados.
- `getMyLatest` passou a contemplar tambem solicitacoes feitas por gestor.
- Notificacoes do fluxo foram ajustadas para diferenciar aprovacao do gestor e aprovacao final.
- Conciliacao de responsaveis Acessorias ficou mais restrita:
  - match exato por email/username so auto-confirma se o departamento local combina com o departamento fonte;
  - alias confirmado tambem passa a respeitar departamento;
  - divergencia vira `PENDING_REVIEW`.
- Clientes:
  - `ListClientsDto` ganhou `scope`, `mine`, `myClientsOnly` e `my_clients_only`;
  - `scope=mine` restringe a cliente proprio ou atribuicoes ativas onde o usuario e responsavel/gestor;
  - `scope=all` continua significando "todos permitidos pelo backend", sem acesso global para perfil nao privilegiado.
- Colaboradores:
  - `GET /collaborators/public` e `visibility=company_public` implementados;
  - payload publico interno nao retorna roles, permissions, metadata, phone/telefone;
  - continua exigindo `collaborators.read`, mantendo cliente externo bloqueado por RBAC.
- Notificacoes:
  - `MAX_TAKE` reduzido para 100;
  - criacao/teste de notificacao agora poda excedentes por chave operacional;
  - `PATCH /notifications/alert-all` implementado para confirmar alertas em massa;
  - evento SSE `notifications.clear_alerts` adicionado.

### Web

- Tela DEV Acessorias simplificada:
  - somente um botao externo/API de integracao ficou exposto: `Integrar`;
  - demais botoes visiveis agora sao diagnosticos, locais ou reconciliacoes internas.
- Tipo `AccessRequestStatus` passou a incluir `PENDING_DEV`.
- Tela de solicitacao de acesso:
  - calendario multi-dia proprio substituiu o date picker nativo;
  - gestor pode selecionar o proprio usuario ativo para solicitar acesso para si;
  - painel de aprovacoes usa `PENDING_MANAGER` para gestor e `PENDING_DEV` para DEV/ADMIN/TI;
  - estatisticas consideram `PENDING_MANAGER` + `PENDING_DEV`.
- Clientes:
  - `ClientsOverviewPage` ganhou toggle `Todos permitidos` / `Ver meus clientes`;
  - services/types enviam `scope=mine` ao backend.
- Colaboradores:
  - services/types conhecem `listPublicCollaborators()` e `visibility=company_public`.
- Notificacoes:
  - pagina ganhou botao `Confirmar alertas`;
  - service chama `PATCH /notifications/alert-all`;
  - stream invalida cache tambem para `notifications.clear_alerts`.
- Toasts globais:
  - `NotificationToasts` tambem invalida cache ao receber `notifications.clear_alerts`.
- Playwright:
  - criada `tests/e2e/access-requests.spec.ts`;
  - cobre gestor solicitando para si com status `PENDING_DEV`;
  - cobre DEV aprovando solicitacao `PENDING_DEV`;
  - cobre painel DEV expondo `Integrar` sem botoes dispersos de previa/importacao/backfill;
  - cobre contratos de `Ver meus clientes` e `Confirmar alertas`.

### Docs

- Criado `AUDITORIA-DEPLOY-19-EXECUCAO-HOMOLOGACAO-EASYPANEL.md`.
- Este `CONTEXTO-CODEX-ATUAL.md` foi atualizado para a nova frente 17-19.

## Validacoes executadas

### API

- `npx.cmd prisma generate` - OK. O Prisma carregou `.env`, sem imprimir valores.
- `npm.cmd run prisma:validate` - OK. O Prisma carregou `.env`, sem imprimir valores.
- `npm.cmd test -- --runInBand src/modules/access-requests/access-requests.service.spec.ts` - OK.
- `npm.cmd test -- --runInBand src/modules/access-requests/access-requests.service.spec.ts src/modules/integrations/acessorias/acessorias-responsible-resolver.service.spec.ts` - OK, 2 suites e 10 testes.
- `npm.cmd test -- --runInBand src/modules/clients/clients.service.spec.ts src/modules/collaborators/collaborators.service.spec.ts src/modules/notifications/notifications.service.spec.ts` - OK, 3 suites e 36 testes.
- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 44 suites e 283 testes.
- `npm.cmd run test:e2e` - OK, 1 suite e 136 testes.
- `git diff --check` - OK.

### Web

- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 9 testes de contrato.
- `npx.cmd playwright test tests/e2e/access-requests.spec.ts --reporter=line` - OK, 4 testes.
- `npm.cmd run test:e2e -- --reporter=line` - OK, 17 testes passaram e 1 skip.
- `git diff --check` - OK.

## Nao executado nesta sessao

- Smoke EasyPanel real.
- `prisma migrate deploy` contra banco alvo.
- Docker.
- Instalacao de dependencias novas.

## Status atual dos arquivos

### portal-sama-api

- `M prisma/schema.prisma`
- `M src/modules/access-requests/access-requests.service.spec.ts`
- `M src/modules/access-requests/access-requests.service.ts`
- `M src/modules/clients/dto/list-clients.dto.ts`
- `M src/modules/clients/clients.service.ts`
- `M src/modules/clients/clients.service.spec.ts`
- `M src/modules/collaborators/dto/list-collaborators.dto.ts`
- `M src/modules/collaborators/collaborators.controller.ts`
- `M src/modules/collaborators/collaborators.service.ts`
- `M src/modules/collaborators/collaborators.service.spec.ts`
- `M src/modules/collaborators/collaborators.types.ts`
- `M src/modules/integrations/acessorias/acessorias-responsible-resolver.service.spec.ts`
- `M src/modules/integrations/acessorias/acessorias-responsible-resolver.service.ts`
- `M src/modules/notifications/notifications.controller.ts`
- `M src/modules/notifications/notifications.service.ts`
- `M src/modules/notifications/notifications.service.spec.ts`
- `M src/modules/notifications/notifications.types.ts`
- `?? prisma/migrations/20260611130000_add_access_request_pending_dev/`

### portal-sama-web

- `M src/pages/access-requests/AccessRequestPage.tsx`
- `M src/components/notifications/NotificationToasts.tsx`
- `M src/pages/clients/ClientsOverviewPage.tsx`
- `M src/pages/dev/DevAdminPage.tsx`
- `M src/pages/notifications/NotificationsPage.tsx`
- `M src/services/clients.service.ts`
- `M src/services/collaborators.service.ts`
- `M src/services/notifications.service.ts`
- `M src/types/access-requests.ts`
- `M src/types/clients.ts`
- `M src/types/collaborators.ts`
- `M src/types/notifications.ts`
- `?? tests/e2e/access-requests.spec.ts`

### portal-sama-docs

- Novos documentos/evidencias de homologacao 17-19 ainda nao versionados.
- `M CONTEXTO-CODEX-ATUAL.md`
- `?? AUDITORIA-DEPLOY-19-EXECUCAO-HOMOLOGACAO-EASYPANEL.md`

## Pendencias principais

1. Aplicar a migration `20260611130000_add_access_request_pending_dev` em banco controlado antes de testar o fluxo novo.
2. Continuar validacao real/controlada das fases pendentes:
   - Home DEV/Gestor;
   - Home DEV/Gestor em Playwright/smoke com dados reais;
   - Clientes / Ver meus clientes em Playwright/smoke;
   - colaboradores publicos internos em Playwright/smoke;
   - notificacoes e preferencias em Playwright/smoke;
   - revisao final da conciliacao nominal por nome + departamento, se exigida pelo plano.
3. Depois de build/deploy controlado, executar smoke EasyPanel real.

## Veredito atual

`Nao pronto para usuarios reais`.

As correcoes estao implementadas e verdes em validacoes locais, incluindo API E2E e Playwright Web.
Ainda faltam migration aplicada em ambiente alvo e smoke EasyPanel com dados reais.

## Cuidados

- Nao reverter mudancas locais.
- Nao expor nem ler valores de `.env`.
- A credencial administrativa informada em chat anterior foi rotacionada pelo usuario; nao salvar credenciais em docs/evidencias.
- Se usar Docker ou instalar algo em proximas etapas, registrar neste contexto e na auditoria 19.
