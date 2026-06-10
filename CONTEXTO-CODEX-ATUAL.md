# CONTEXTO CODEX ATUAL

Atualizado em: 2026-06-10
Sessao atual: continuidade das fases do AUDITORIA-DEPLOY-15

## Objetivo ativo

Corrigir os bloqueios de deploy conforme:

- `AUDITORIA-DEPLOY-15-PREPARACAO-CORRECOES-GO-LIVE.md`
- registro incremental em `AUDITORIA-DEPLOY-16-EXECUCAO-CORRECOES-GO-LIVE.md`

## Branch atual

Branch nos tres repositorios:

- `portal-sama-docs`: `fix/deploy-readiness-go-live`
- `portal-sama-api`: `fix/deploy-readiness-go-live`
- `portal-sama-web`: `fix/deploy-readiness-go-live`

## Estado das fases

| Fase | Status | Observacao |
|---|---|---|
| Fase 0 - Preparacao operacional | Concluida | Worktrees conferidos, branch criada e evidencias copiadas para `evidencias/auditoria-deploy-15/`. |
| Fase 1 - Artefatos e segredos | Pendente critica | Ainda nao recriado ZIP limpo; segredos expostos ainda precisam rotacao externa. |
| Fase 2 - DB/Acessorias | Validada localmente em sessao anterior | Migration MySQL e JSONPath ja estavam consolidados no working tree limpo da API. |
| Fase 3 - clients.read | Implementada e validada com testes focados | `clients.read` nao concede mais leitura ampla sozinho; `ADMIN`/`DEV` seguem amplos; demais perfis dependem de escopo. |
| Fase 4 - home-summary | Implementada e validada com testes focados | Backend deriva perfil efetivo; `profile=admin` nao amplia escopo; frontend parou de enviar `profile`. |
| Fase 5 - E2E API Web Push/CSRF | Implementada e validada | `npm.cmd run test:e2e` passou com 136/136. |
| Fase 6 - E2E Web Home | Proximo passo | E2E Web ainda estava conhecido com 11 pass, 1 skip, 2 fail nos textos da Home. |
| Fase 7 - qs audit | Pendente | Ainda nao tratada nesta sessao. |
| Fases 8-11 - Operacao/producao | Pendente | ClamAV strict, backup/restore, Web Push real e perfis reais ainda sem prova final. |

## Ferramentas locais

- Docker autorizado pelo usuario para testes, mas nao foi necessario nesta sessao.
- Foi usado `npx.cmd prettier@3.8.4` de forma transitoria na sessao anterior; sem alteracao
  intencional de `package.json` ou `package-lock.json`.

## O que foi feito nesta sessao

- Implementada Fase 5:
  - `test/app.e2e-spec.ts` agora define `CSRF_SECRET`, `COOKIE_DOMAIN`, `COOKIE_SECURE` e
    `COOKIE_SAME_SITE` antes de importar o `AppModule`.
  - A suite E2E nao herda mais configuracao real/local de cookie do `.env`.
  - Os dois testes Web Push/CSRF que falhavam por `403` agora passam.
  - Testes negativos de CSRF continuam retornando `403` quando falta token/cookie valido.

## Arquivos alterados

### portal-sama-api

- `src/modules/clients/clients.service.ts`
- `src/modules/clients/clients.service.spec.ts`
- `src/modules/rbac/default-rbac.ts`
- `src/modules/integrations/acessorias/acessorias-home.controller.ts`
- `src/modules/integrations/acessorias/acessorias-home.service.ts`
- `src/modules/integrations/acessorias/acessorias-home.service.spec.ts`
- `test/app.e2e-spec.ts`

### portal-sama-web

- `src/services/acessorias.service.ts`
- `src/pages/home/HomePage.tsx`
- `src/pages/dev/DevAdminPage.tsx`

### portal-sama-docs

- `AUDITORIA-DEPLOY-16-EXECUCAO-CORRECOES-GO-LIVE.md`
- `CONTEXTO-CODEX-ATUAL.md`
- `evidencias/auditoria-deploy-15/validation-api-results.json`
- `evidencias/auditoria-deploy-15/validation-web-results.json`
- `evidencias/auditoria-deploy-15/acessorias-api-results-clean.json`

## Comandos executados e resultado

### Fase 5 API

- `npm.cmd run test:e2e` - OK, 1 suite e 136 testes.
- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 44 suites e 272 testes.
- `git diff --check` - OK.

### Resultados anteriores ainda validos nesta branch

- Fase 3 testes focados API - OK.
- Fase 4 testes focados API/Web, lint/build API/Web e contratos web - OK.

## Status atual do working tree

Antes da ultima conferencia final:

- `portal-sama-api`: arquivos das fases 3, 4 e 5 alterados.
- `portal-sama-web`: arquivos da Fase 4 alterados.
- `portal-sama-docs`: novo/alterado `AUDITORIA-DEPLOY-16...`, `CONTEXTO-CODEX-ATUAL.md` e `evidencias/`.

## Proximo passo exato

Implementar Fase 6:

- corrigir E2E Web da Home;
- decidir pelo contrato atual da UI como fonte: usar textos atuais `Diagnostico Acessorias` e
  `Baixas sincronizadas`, a menos que produto exija restaurar textos antigos;
- ajustar `portal-sama-web/tests/e2e/smoke.spec.ts` ou mocks relacionados;
- neutralizar/mocar ruido de `/api-v2/notifications/stream?take=20` e `/api-v2/me/security`;
- rodar `npm.cmd run test:e2e`, `npm.cmd run build`, `npm.cmd run lint` e contratos web;
- depois atualizar `AUDITORIA-DEPLOY-16...` e este arquivo.

## Cuidado

- Nao reverter mudancas locais.
- Nao expor `.env`.
- Fase 1 continua critica: ZIP da API contaminado e rotacao de segredos ainda pendentes.
- API E2E agora esta verde localmente.
- E2E Web Home ainda precisa ser tratado na Fase 6.
