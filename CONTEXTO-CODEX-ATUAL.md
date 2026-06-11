# CONTEXTO CODEX ATUAL

Atualizado em: 2026-06-11
Sessao atual: Homologacao EasyPanel Acessorias - execucao inicial das auditorias 17, 18 e 19

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

### Docs

- Criado `AUDITORIA-DEPLOY-19-EXECUCAO-HOMOLOGACAO-EASYPANEL.md`.
- Este `CONTEXTO-CODEX-ATUAL.md` foi atualizado para a nova frente 17-19.

## Validacoes executadas

### API

- `npx.cmd prisma generate` - OK. O Prisma carregou `.env`, sem imprimir valores.
- `npm.cmd run prisma:validate` - OK. O Prisma carregou `.env`, sem imprimir valores.
- `npm.cmd test -- --runInBand src/modules/access-requests/access-requests.service.spec.ts` - OK.
- `npm.cmd test -- --runInBand src/modules/access-requests/access-requests.service.spec.ts src/modules/integrations/acessorias/acessorias-responsible-resolver.service.spec.ts` - OK, 2 suites e 10 testes.
- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 44 suites e 278 testes.
- `git diff --check` - OK.

### Web

- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 9 testes de contrato.
- `git diff --check` - OK.

## Nao executado nesta sessao

- `npm.cmd run test:e2e` da API.
- Playwright/E2E Web.
- Smoke EasyPanel real.
- `prisma migrate deploy` contra banco alvo.
- Docker.
- Instalacao de dependencias novas.

## Status atual dos arquivos

### portal-sama-api

- `M prisma/schema.prisma`
- `M src/modules/access-requests/access-requests.service.spec.ts`
- `M src/modules/access-requests/access-requests.service.ts`
- `M src/modules/integrations/acessorias/acessorias-responsible-resolver.service.spec.ts`
- `M src/modules/integrations/acessorias/acessorias-responsible-resolver.service.ts`
- `?? prisma/migrations/20260611130000_add_access_request_pending_dev/`

### portal-sama-web

- `M src/pages/access-requests/AccessRequestPage.tsx`
- `M src/pages/dev/DevAdminPage.tsx`
- `M src/types/access-requests.ts`

### portal-sama-docs

- Novos documentos/evidencias de homologacao 17-19 ainda nao versionados.
- `M CONTEXTO-CODEX-ATUAL.md`
- `?? AUDITORIA-DEPLOY-19-EXECUCAO-HOMOLOGACAO-EASYPANEL.md`

## Pendencias principais

1. Aplicar a migration `20260611130000_add_access_request_pending_dev` em banco controlado antes de testar o fluxo novo.
2. Rodar API E2E.
3. Rodar Playwright Web, com foco em:
   - painel DEV Acessorias;
   - solicitacoes de acesso;
   - calendario multi-dia;
   - gestor solicitando para si;
   - aprovacao DEV.
4. Continuar fases pendentes do plano:
   - Home DEV/Gestor;
   - Clientes / Ver meus clientes;
   - colaboradores publicos internos;
   - notificacoes e preferencias;
   - revisao final da conciliacao nominal por nome + departamento, se exigida pelo plano.
5. Depois de build/deploy controlado, executar smoke EasyPanel real.

## Veredito atual

`Nao pronto para usuarios reais`.

A primeira leva de correcoes esta implementada e verde em validacoes locais, mas ainda faltam
migration aplicada em ambiente alvo, E2E, Playwright, smoke EasyPanel e fases funcionais pendentes.

## Cuidados

- Nao reverter mudancas locais.
- Nao expor nem ler valores de `.env`.
- A credencial administrativa informada em chat anterior foi rotacionada pelo usuario; nao salvar credenciais em docs/evidencias.
- Se usar Docker ou instalar algo em proximas etapas, registrar neste contexto e na auditoria 19.
