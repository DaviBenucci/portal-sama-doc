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
| Fase 6 - E2E Web Home | Implementada e validada | E2E Web passou com 13/13 e 1 skip apos alinhar mocks/textos da Home atual. |
| Fase 7 - qs audit | Implementada e validada | `qs` transitivo atualizado para `6.15.2`; API audit retornou 0 vulnerabilidades. |
| Fase 8 - ClamAV strict | Validada localmente em container | Docker `portal-sama-api:clamav-runtime`; EICAR detectado por `/usr/bin/clamscan`; testes strict do scanner passaram. |
| Fase 9 - Backup/restore drill | Validada localmente em Docker | Backup, verify e restore drill passaram contra bancos isolados `portal_sama_backup_drill_source` e `portal_sama_backup_drill_restore`. |
| Fases 10-11 - Operacao/producao | Pendente | Web Push real e perfis reais ainda sem prova final. |

## Ferramentas locais

- Docker autorizado pelo usuario e usado na Fase 8 com a imagem local
  `portal-sama-api:clamav-runtime`; nenhum ClamAV foi instalado no Windows host.
- Foi usado `npx.cmd prettier@3.8.4` de forma transitoria na sessao anterior; sem alteracao
  intencional de `package.json` ou `package-lock.json`.
- Foi usado `npm.cmd audit fix` no `portal-sama-api`; isso atualizou `qs` de `6.15.1` para
  `6.15.2` no `package-lock.json` e no `node_modules` local.
- Dentro do container da Fase 8, foi executado `npm run ops:clamav:update` para atualizar
  assinaturas ClamAV antes do readiness strict.
- Na Fase 9, foi usado Docker com `portal-sama-audit-mysql` e a imagem `mysql:8.4`; dentro do
  container efemero foram instalados `nodejs` e `which` via `microdnf` para rodar os scripts com
  clientes MySQL 8.4. Nada foi instalado no Windows host.

## O que foi feito nesta sessao

- Implementada Fase 6:
  - `tests/e2e/smoke.spec.ts` agora mocka `/api-v2/notifications/stream?take=20` como SSE.
  - `tests/e2e/smoke.spec.ts` agora mocka `/api-v2/me/security` para paginas autenticadas.
  - O E2E da Home admin foi alinhado aos textos atuais `Diagnostico Acessorias`,
    `Vencimentos` e `Solicitacoes`.
  - O E2E da Home colaborador foi alinhado ao bloco atual `Baixas sincronizadas`.
  - Nao houve uso de Docker nem instalacao local nova nesta fase.
- Implementada Fase 7:
  - `npm.cmd audit fix` atualizou a resolucao transitiva de `qs` para `6.15.2`.
  - Apenas `package-lock.json` mudou na API; `package.json` permaneceu igual.
  - `npm.cmd audit --audit-level=moderate` passou com 0 vulnerabilidades.
- Validada Fase 8 localmente:
  - readiness strict em Docker passou com `ok: true`, `failed: 0` e `clamav-eicar: passed`.
  - evidencia salva em `evidencias/auditoria-deploy-15/clamav-strict-readiness.json`.
  - adicionados testes do `DocumentUploadScannerService` para upload limpo em `strict` e bloqueio
    de deteccao tipo EICAR.
  - A warning restante do readiness e `backup-rollback`, esperada e pertencente a Fase 9.
- Validada Fase 9 localmente:
  - Criados bancos isolados `portal_sama_backup_drill_source` e
    `portal_sama_backup_drill_restore` no MySQL Docker local.
  - `create-operational-backup`, `verify-operational-backup` e
    `restore-drill-operational-backup` passaram.
  - Restore drill foi aplicado de verdade (`dryRun: false`) no banco/storage isolados.
  - Checagem pos-restore confirmou `rows_count: 1` e `sentinel_label=fase-9-drill-source`.
  - Evidencias salvas em `evidencias/auditoria-deploy-15/backup-drill/`.

## Arquivos alterados

### portal-sama-api

- `package-lock.json`
- `src/modules/documents/document-upload-scanner.service.spec.ts`
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
- `tests/e2e/smoke.spec.ts`

### portal-sama-docs

- `AUDITORIA-DEPLOY-16-EXECUCAO-CORRECOES-GO-LIVE.md`
- `CONTEXTO-CODEX-ATUAL.md`
- `evidencias/auditoria-deploy-15/validation-api-results.json`
- `evidencias/auditoria-deploy-15/validation-web-results.json`
- `evidencias/auditoria-deploy-15/acessorias-api-results-clean.json`
- `evidencias/auditoria-deploy-15/clamav-strict-readiness.json`
- `evidencias/auditoria-deploy-15/backup-drill/backup-create.json`
- `evidencias/auditoria-deploy-15/backup-drill/backup-verify.json`
- `evidencias/auditoria-deploy-15/backup-drill/restore-drill.json`
- `evidencias/auditoria-deploy-15/backup-drill/restore-target-check.json`
- `evidencias/auditoria-deploy-15/backup-drill/backup-artifacts.sha256`
- `evidencias/auditoria-deploy-15/backup-drill/backups/portal-sama-20260610T184955Z-e76aef/`

## Comandos executados e resultado

### Fase 5 API

- `npm.cmd run test:e2e` - OK, 1 suite e 136 testes.
- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 44 suites e 272 testes.
- `git diff --check` - OK.

### Fase 6 Web

- `npm.cmd run test:e2e` - OK, 13 passaram e 1 skip.
- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 9 contratos.
- `git diff --check` - OK.

### Fase 7 API

- `npm.cmd audit --audit-level=moderate` - OK, 0 vulnerabilidades.
- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 44 suites e 272 testes.
- `npm.cmd run test:e2e` - OK, 1 suite e 136 testes.
- `git diff --check` - OK.

### Fase 8 API/Operacao

- `docker run --rm portal-sama-api:clamav-runtime sh -lc "node --version && clamscan --version"` -
  OK, Node 22 e ClamAV 1.4.4.
- `docker run --rm portal-sama-api:clamav-runtime sh -lc "npm run ops:clamav:update"` - OK,
  assinaturas `daily`, `main` e `bytecode` atualizadas; aviso de `clamd` ausente esperado.
- readiness strict em Docker com `SAMA_UPLOAD_SCAN_MODE=strict --skip-env --skip-database` - OK,
  `clamav-eicar` passou e evidencia JSON foi salva.
- `npm.cmd test -- --runInBand src/modules/documents/document-upload-scanner.service.spec.ts` -
  OK, 1 suite e 5 testes.
- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 44 suites e 274 testes.
- `npm.cmd run test:e2e` - OK, 1 suite e 136 testes.
- `npm.cmd audit --audit-level=moderate` - OK, 0 vulnerabilidades.
- `git diff --check` - OK.

### Fase 9 Operacao

- Preparacao SQL no MySQL Docker local - OK, origem e alvo isolados criados.
- `create-operational-backup` em container `mysql:8.4` com Node efemero - OK, `failed: 0`,
  `warnings: 0`.
- `verify-operational-backup` - OK, `failed: 0`, `warnings: 1` (lembrete padrao antes do
  restore real).
- `restore-drill-operational-backup --apply-database --apply-storage --confirm RESTORE_DRILL_TARGET_IS_ISOLATED`
  - OK, `dryRun: false`, `failed: 0`, `warnings: 2`.
- Checagem pos-restore no banco alvo - OK, `rows_count: 1`.
- `Get-FileHash` dos artefatos principais - OK, salvo em `backup-artifacts.sha256`.
- `rg` nas evidencias de backup procurando senha local - OK, sem ocorrencias.

### Resultados anteriores ainda validos nesta branch

- Fase 3 testes focados API - OK.
- Fase 4 testes focados API/Web, lint/build API/Web e contratos web - OK.

## Status atual do working tree

Conferencia final desta sessao:

- `portal-sama-api`: `M package-lock.json` e
  `M src/modules/documents/document-upload-scanner.service.spec.ts`.
- `portal-sama-web`: `M tests/e2e/smoke.spec.ts`.
- `portal-sama-docs`: `M AUDITORIA-DEPLOY-16-EXECUCAO-CORRECOES-GO-LIVE.md` e
  `M CONTEXTO-CODEX-ATUAL.md`, alem de
  `?? evidencias/auditoria-deploy-15/clamav-strict-readiness.json` e
  `?? evidencias/auditoria-deploy-15/backup-drill/`.

## Proximo passo exato

Implementar Fase 10:

- homologar Web Push real conforme `AUDITORIA-DEPLOY-15` e documentos Web Push;
- confirmar VAPID real, HTTPS, service worker, subscribe/unsubscribe, preferencias por usuario
  e disparo por evento de negocio;
- usar apenas ambiente/credenciais de homologacao ou mocks controlados quando nao houver acesso
  real;
- preservar evidencias sanitizadas da navegacao/teste;
- atualizar `AUDITORIA-DEPLOY-16...` e este arquivo.

## Cuidado

- Nao reverter mudancas locais.
- Nao expor `.env`.
- Fase 1 continua critica: ZIP da API contaminado e rotacao de segredos ainda pendentes.
- API E2E agora esta verde localmente.
- E2E Web Home agora esta verde localmente.
- API audit agora esta verde localmente.
- ClamAV strict foi validado localmente em container, mas homologacao/producao ainda deve replicar
  `SAMA_UPLOAD_SCAN_MODE=strict` no runtime correto.
- Backup/verify/restore drill foi validado localmente em Docker, mas producao ainda deve usar
  destino de backup externo/seguro e alvo de restore isolado equivalente ao ambiente real.
