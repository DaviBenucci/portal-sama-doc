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
| Fase 5 - E2E API Web Push/CSRF | Proximo passo | E2E API segue conhecido com 134 pass, 2 fail por `403` em Web Push/CSRF. |
| Fase 6 - E2E Web Home | Pendente | Ainda nao tratada nesta sessao. |
| Fase 7 - qs audit | Pendente | Ainda nao tratada nesta sessao. |
| Fases 8-11 - Operacao/producao | Pendente | ClamAV strict, backup/restore, Web Push real e perfis reais ainda sem prova final. |

## Ferramentas locais

- Docker autorizado pelo usuario para testes, mas nao foi necessario nesta sessao.
- Foi usado `npx.cmd prettier@3.8.4` de forma transitoria. O primeiro uso no frontend baixou o
  pacote via `npx`; nao houve alteracao intencional de `package.json` ou `package-lock.json`.
- O ruido de formatacao gerado inicialmente no frontend foi revertido mecanicamente; o diff final
  do web ficou limitado as chamadas sem `profile`.

## O que foi feito nesta sessao

- Reconfirmado o contexto em:
  - `CONTEXTO-CODEX-ATUAL.md`
  - `AUDITORIA-DEPLOY-15-PREPARACAO-CORRECOES-GO-LIVE.md`
  - `AUDITORIA-DEPLOY-16-EXECUCAO-CORRECOES-GO-LIVE.md`
- Implementada Fase 4:
  - `AcessoriasHomeService` agora calcula perfil efetivo pelo usuario autenticado.
  - `profile` por query virou preferencia compatibilizada e nunca amplia autorizacao.
  - `ADMIN`/`DEV` continuam com visao global.
  - `MANAGER`/gestao equivalente fica limitado a departamento permitido ou responsabilidade direta.
  - demais usuarios ficam em escopo de colaborador, por responsabilidade direta e itens sem
    responsavel apenas dentro do departamento permitido.
  - usuarios sem departamento/responsabilidade nao recebem visao global por fallback.
  - frontend removeu `profile` da chamada `/integrations/acessorias/home-summary`.

## Arquivos alterados

### portal-sama-api

- `src/modules/clients/clients.service.ts`
- `src/modules/clients/clients.service.spec.ts`
- `src/modules/rbac/default-rbac.ts`
- `src/modules/integrations/acessorias/acessorias-home.controller.ts`
- `src/modules/integrations/acessorias/acessorias-home.service.ts`
- `src/modules/integrations/acessorias/acessorias-home.service.spec.ts`

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

### Fase 4 API

- `npm.cmd test -- --runInBand src/modules/integrations/acessorias/acessorias-home.service.spec.ts` - OK, 1 suite e 6 testes.
- `npm.cmd test -- --runInBand src/modules/integrations/acessorias/acessorias-registrations.service.spec.ts src/modules/integrations/acessorias/acessorias-deliveries.service.spec.ts` - OK, 2 suites e 16 testes.
- `npm.cmd test -- --runInBand src/modules/clients/clients.service.spec.ts src/modules/rbac/default-rbac.spec.ts src/modules/integrations/acessorias/acessorias-home.service.spec.ts src/modules/integrations/acessorias/acessorias-registrations.service.spec.ts src/modules/integrations/acessorias/acessorias-deliveries.service.spec.ts` - OK, 5 suites e 35 testes.
- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `git diff --check` - OK.

### Fase 4 Web

- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 9 contratos.
- `git diff --check` - OK.

## Status atual do working tree

Ultimo status observado antes da atualizacao final de docs:

- `portal-sama-api`: 6 arquivos alterados.
- `portal-sama-web`: 3 arquivos alterados.
- `portal-sama-docs`: novo/alterado `AUDITORIA-DEPLOY-16...`, `CONTEXTO-CODEX-ATUAL.md` e `evidencias/`.

## Proximo passo exato

Implementar Fase 5:

- corrigir E2E API Web Push/CSRF;
- isolar a suite E2E da configuracao real de cookie em `.env`;
- definir explicitamente `COOKIE_DOMAIN=''`, `COOKIE_SECURE='false'` e `COOKIE_SAME_SITE='lax'`
  no setup E2E ou equivalente;
- manter testes negativos de CSRF;
- rodar `npm.cmd run test:e2e`;
- depois atualizar `AUDITORIA-DEPLOY-16...` e este arquivo.

## Cuidado

- Nao reverter mudancas locais.
- Nao expor `.env`.
- Fase 1 continua critica: ZIP da API contaminado e rotacao de segredos ainda pendentes.
- E2E API ainda esta vermelho por `QA-API-01`; tratar na Fase 5.
- E2E Web Home ainda esta vermelho por `QA-WEB-01`; tratar na Fase 6.
