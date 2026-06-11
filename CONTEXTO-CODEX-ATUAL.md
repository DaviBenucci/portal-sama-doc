# CONTEXTO CODEX ATUAL

Atualizado em: 2026-06-11
Sessao atual: Fase 1 - credencial administrativa rotacionada informada

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
| Fase 1 - Artefatos e segredos | Parcialmente implementada | Scripts `package:safe` criados na API/Web, ZIPs seguros gerados e validados, binarios de backup removidos do docs. Credencial administrativa compartilhada no chat foi informada como rotacionada em 11/06/2026. Rotacao externa dos demais segredos ainda pendente e critica. |
| Fase 2 - DB/Acessorias | Validada localmente em sessao anterior | Migration MySQL e JSONPath ja estavam consolidados no working tree limpo da API. |
| Fase 3 - clients.read | Implementada e validada com testes focados | `clients.read` nao concede mais leitura ampla sozinho; `ADMIN`/`DEV` seguem amplos; demais perfis dependem de escopo. |
| Fase 4 - home-summary | Implementada e validada com testes focados | Backend deriva perfil efetivo; `profile=admin` nao amplia escopo; frontend parou de enviar `profile`. |
| Fase 5 - E2E API Web Push/CSRF | Implementada e validada | `npm.cmd run test:e2e` passou com 136/136. |
| Fase 6 - E2E Web Home | Implementada e validada | E2E Web passou com 13/13 e 1 skip apos alinhar mocks/textos da Home atual. |
| Fase 7 - qs audit | Implementada e validada | `qs` transitivo atualizado para `6.15.2`; API audit retornou 0 vulnerabilidades. |
| Fase 8 - ClamAV strict | Validada localmente em container | Docker `portal-sama-api:clamav-runtime`; EICAR detectado por `/usr/bin/clamscan`; testes strict do scanner passaram. |
| Fase 9 - Backup/restore drill | Validada localmente em Docker | Backup, verify e restore drill passaram contra bancos isolados `portal_sama_backup_drill_source` e `portal_sama_backup_drill_restore`. |
| Fase 10 - Web Push real | Validada localmente com navegador real | Chromium headed/perfil persistente, endpoint real `jmt17.google.com`, push `1/1`, unsubscribe OK. Ainda falta repetir em HTTPS publico/confiavel com VAPID oficial e clique nativo assistido. |
| Fase 11 - Perfis reais | Validada localmente com matriz controlada; autenticacao admin real validada | API real local 102/102 checks apos correcao de documentos; browser smoke 4/4 perfis principais. Em 11/06/2026, dominio publico/admin real passou em smoke publico, auth HTTP e Playwright real. Ainda falta matriz com perfis reais. |

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
- Na Fase 10, foi usado Docker/MySQL local `portal-sama-audit-mysql` com banco isolado
  `portal_sama_webpush_phase10`; tambem foi usado Chromium via Playwright em modo headed e perfil
  persistente temporario. Nenhuma instalacao permanente foi feita no Windows host.
- Na Fase 11, foi usado Docker/MySQL local `portal-sama-audit-mysql` com banco isolado
  `portal_sama_profiles_phase11`; API local na porta `3120`, web local na porta `4176` e
  schedulers de Acessorias desligados explicitamente. Nenhuma instalacao permanente foi feita no
  Windows host.
- Na Fase 1, foram usados PowerShell e APIs .NET locais para criar/validar ZIPs seguros. Nenhuma
  dependencia npm nova e nenhuma instalacao permanente foi feita.
- Em 11/06/2026, foi usado o dominio publico `https://portal.samacontabil.com.br` para smoke
  publico, auth HTTP real e Playwright real. A credencial foi usada somente via variavel de
  ambiente temporaria e nao foi salva nos arquivos/evidencias.
- Em 11/06/2026, o usuario informou que rotacionou a credencial administrativa compartilhada no
  chat. A nova credencial nao foi fornecida, persistida nem verificada por login nesta rodada.

## O que foi feito nesta branch recente

- Parcialmente implementada Fase 1:
  - adicionado `npm.cmd run package:safe` na API e no Web;
  - criados `scripts/create-safe-package.ps1` em `portal-sama-api` e `portal-sama-web`;
  - os scripts criam ZIP em staging temporario e validam entradas antes de finalizar;
  - bloqueados `.env*`, `.git`, `.ai-tests`, `node_modules`, `dist`, `coverage`, logs, dumps,
    backups, arquivos compactados, chaves/certificados e SQL fora de migrations Prisma;
  - preservadas migrations Prisma `prisma/migrations/*/migration.sql` por serem fonte necessaria
    de deploy/migracao, nao dump;
  - READMEs da API/Web atualizados com comando seguro e evidencia JSON;
  - gerados fora dos repositorios:
    - `C:\Users\Sama Contabilidade\Desktop\portal-sama-api.safe.zip`;
    - `C:\Users\Sama Contabilidade\Desktop\portal-sama-web.safe.zip`;
  - hashes registrados em `evidencias/auditoria-deploy-15/phase1-artifacts/`;
  - filtro independente com `tar -tf` passou nos dois ZIPs;
  - `C:\Users\Sama Contabilidade\Desktop\portal-sama-api.zip` antigo nao existe localmente;
  - removidos do docs os binarios `database.sql.gz`, `storage.tar.gz`, `metadata.json` e
    `storage-manifest.json` da pasta `backup-drill/backups/`;
  - a credencial administrativa real informada no chat em 11/06/2026 foi registrada apenas como
    item sensivel, sem salvar o valor em arquivos/evidencias;
  - em 11/06/2026, usuario informou que essa credencial administrativa foi rotacionada;
  - evidencia declarativa salva em
    `evidencias/auditoria-deploy-15/phase1-artifacts/admin-credential-rotation-reported.json`;
  - `package-validation-summary.json` atualizado para retirar a credencial administrativa da lista
    de rotacao pendente e manter os demais segredos como pendentes.
  - adicionado `npm run ops:secrets:check` na API para validar higiene/rotacao de segredos a
    partir de `process.env`, sem carregar `.env` e sem imprimir valores;
  - o preflight checa JWT, CSRF, chave de certificados, `DATABASE_URL`, Acessorias, Web Push/VAPID,
    reutilizacao entre segredos e marcador opcional `SAMA_SECRET_ROTATION_CONFIRMED_AT`;
  - validado com variaveis sinteticas em memoria, salvando evidencia sanitizada em
    `evidencias/auditoria-deploy-15/phase1-artifacts/secret-rotation-check-synthetic.json`;
  - `portal-sama-api.safe.zip` foi regenerado apos incluir o preflight de segredos.
- Validada homologacao real parcial em `https://portal.samacontabil.com.br`:
  - `smoke-public`: frontend publico `200`, API health `ok=true`, `database=up`, `storage=up`,
    CORS e CSRF OK;
  - `smoke-auth` com usuario administrativo real: CSRF, login, `auth/me`, refresh e logout OK;
  - refresh cookie validado com `HttpOnly`, `Secure` e `SameSite=Lax`;
  - Playwright real passou apos ajustar contrato textual obsoleto do teste;
  - `tests/e2e/real-auth.spec.ts` deixou de exigir `Sessao ativa` e passou a usar o botao `Sair`
    como sinal autenticado estavel junto do banner/Home;
  - `trace`, `screenshot` e `video` estavam desligados; `test-results` foi removido ao final;
  - evidencias salvas em `evidencias/auditoria-deploy-15/real-homologation-20260611/`;
  - varredura das evidencias reais nao encontrou senha, access token, refresh token nem CSRF.
  - apos a rotacao administrativa informada pelo usuario, `smoke-public` foi repetido e passou
    novamente sem usar credencial.
- Validada Fase 11 localmente com matriz controlada de perfis:
  - criado banco isolado `portal_sama_profiles_phase11` no MySQL Docker local;
  - aplicadas 29 migrations e seed no banco isolado;
  - criados dados sinteticos para `DEV`, `MANAGER`, `DEPARTMENT`, `CLIENT`, `ACCOUNTING`,
    `LEGALIZATION`, `TI` e `AUDITOR`;
  - API local em `3120` e web local em `4176`, com proxy Vite e schedulers de Acessorias
    desligados;
  - a primeira matriz confirmou bug critico: `MANAGER` nao tinha leitura ampla em clientes, mas
    ainda via documentos fora do escopo;
  - corrigido `portal-sama-api/src/modules/documents/documents.service.ts`, removendo `MANAGER`
    de `PRIVILEGED_DOCUMENT_ROLES`;
  - adicionados testes em `portal-sama-api/src/modules/documents/documents.service.spec.ts` para
    listagem e detalhe de documentos fora do escopo de gestor;
  - matriz API apos correcao passou com `102/102` checks;
  - browser smoke passou com `4/4` perfis principais (`DEV`, `MANAGER`, colaborador e `CLIENT`);
  - checagem de UI nao encontrou nomes de clientes fora do escopo para gestor, colaborador e
    cliente;
  - evidencias salvas em `evidencias/auditoria-deploy-15/profiles-phase11/`;
  - PDFs sinteticos de storage local e logs transitorios foram removidos, mantendo JSONs e
    screenshots.
- Validada Fase 10 localmente com navegador real:
  - criado banco isolado `portal_sama_webpush_phase10` no MySQL Docker local;
  - aplicadas migrations/seed/bootstrap admin locais para `phase10.dev`;
  - API local em `3110` e web em `4174`, com proxy Vite e cookies `localhost`;
  - geradas chaves VAPID temporarias somente em runtime; chave privada nao foi persistida;
  - Chromium headless e contexto incognito foram descartados porque bloqueiam Notification/Push API;
  - Chromium headed com perfil persistente temporario conseguiu subscribe real;
  - `POST /notifications/web-push/subscribe` retornou `200`;
  - endpoint real do Chrome Push Service registrado com origem `https://jmt17.google.com`;
  - preferencia Web Push alterada pela UI com `PATCH /notifications/preferences` `200`;
  - `POST /notifications/push/test` retornou `push: { attempted: 1, sent: 1, failed: 0 }`;
  - service worker mostrou notificacao `Atualizacao do Portal Sama`;
  - `POST /notifications/web-push/unsubscribe` retornou `updated: 1`;
  - estado final no navegador: `subscribed=false`;
  - resumo MySQL: `delivery_attempts SENT=1`, `subscriptions.active=0`, `fail_count_sum=0`.
- Corrigido bug de Web Push no frontend:
  - `registerPushServiceWorker()` agora aguarda `navigator.serviceWorker.ready` quando o registro
    ainda nao esta ativo;
  - contrato web passou a exigir `navigator.serviceWorker.ready`.
- Evidencias da Fase 10 salvas em:
  - `evidencias/auditoria-deploy-15/webpush-phase10/browser-persistent-final-validation.json`;
  - `evidencias/auditoria-deploy-15/webpush-phase10/db-summary.json`;
  - `evidencias/auditoria-deploy-15/webpush-phase10/notifications-webpush-final.png`.

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
- `package.json`
- `README.md`
- `scripts/create-safe-package.ps1`
- `scripts/validate-secret-rotation.js`
- `src/modules/documents/document-upload-scanner.service.spec.ts`
- `src/modules/documents/documents.service.ts`
- `src/modules/documents/documents.service.spec.ts`
- `src/modules/clients/clients.service.ts`
- `src/modules/clients/clients.service.spec.ts`
- `src/modules/rbac/default-rbac.ts`
- `src/modules/integrations/acessorias/acessorias-home.controller.ts`
- `src/modules/integrations/acessorias/acessorias-home.service.ts`
- `src/modules/integrations/acessorias/acessorias-home.service.spec.ts`
- `test/app.e2e-spec.ts`

### portal-sama-web

- `src/services/notifications.service.ts`
- `package.json`
- `README.md`
- `scripts/create-safe-package.ps1`
- `tests/e2e/real-auth.spec.ts`
- `scripts/web-contract-tests.mjs`
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
- `evidencias/auditoria-deploy-15/phase1-artifacts/api-safe-package.json`
- `evidencias/auditoria-deploy-15/phase1-artifacts/web-safe-package.json`
- `evidencias/auditoria-deploy-15/phase1-artifacts/package-validation-summary.json`
- `evidencias/auditoria-deploy-15/phase1-artifacts/admin-credential-rotation-reported.json`
- `evidencias/auditoria-deploy-15/phase1-artifacts/secret-rotation-check-synthetic.json`
- `evidencias/auditoria-deploy-15/real-homologation-20260611/smoke-public.json`
- `evidencias/auditoria-deploy-15/real-homologation-20260611/smoke-auth-admin.json`
- `evidencias/auditoria-deploy-15/real-homologation-20260611/playwright-real-auth-summary.json`
- `evidencias/auditoria-deploy-15/real-homologation-20260611/smoke-public-after-admin-rotation.json`
- `evidencias/auditoria-deploy-15/webpush-phase10/`
- `evidencias/auditoria-deploy-15/profiles-phase11/`

Removidos do `portal-sama-docs` nesta Fase 1:

- `evidencias/auditoria-deploy-15/backup-drill/backups/portal-sama-20260610T184955Z-e76aef/database.sql.gz`
- `evidencias/auditoria-deploy-15/backup-drill/backups/portal-sama-20260610T184955Z-e76aef/metadata.json`
- `evidencias/auditoria-deploy-15/backup-drill/backups/portal-sama-20260610T184955Z-e76aef/storage-manifest.json`
- `evidencias/auditoria-deploy-15/backup-drill/backups/portal-sama-20260610T184955Z-e76aef/storage.tar.gz`

## Comandos executados e resultado

### Homologacao real parcial

- `node scripts/portal-public-smoke.mjs --json` em `portal-sama-web` - OK, frontend publico,
  health, CORS e CSRF.
- `node scripts/portal-auth-smoke.mjs --json` em `portal-sama-web` - OK, CSRF, login,
  `auth/me`, refresh e logout com usuario administrativo real; usuario mascarado na evidencia.
- `npm.cmd run test:e2e:real` em `portal-sama-web` - primeira execucao falhou por texto
  `Sessao ativa` ausente; contrato ajustado; segunda execucao OK, 1 teste passou.
- `npm.cmd run lint` em `portal-sama-web` - OK.
- `npm.cmd test` em `portal-sama-web` - OK, 9 contratos.
- Varredura de evidencias reais por senha/token/CSRF - OK, sem ocorrencias sensiveis.
- `test-results` do Playwright real removido ao final - OK.
- Apos usuario informar rotacao da credencial administrativa, `node scripts/portal-public-smoke.mjs --json`
  foi repetido - OK, frontend publico, health, CORS e CSRF.

### Fase 1 Artefatos/Segredos

- `npm.cmd run package:safe -- -EvidencePath ..\portal-sama-docs\evidencias\auditoria-deploy-15\phase1-artifacts\api-safe-package.json`
  em `portal-sama-api` - OK, ZIP `portal-sama-api.safe.zip`, 350 arquivos incluidos, 32198
  excluidos.
- `npm.cmd run package:safe -- -EvidencePath ..\portal-sama-docs\evidencias\auditoria-deploy-15\phase1-artifacts\web-safe-package.json`
  em `portal-sama-web` - OK, ZIP `portal-sama-web.safe.zip`, 184 arquivos incluidos, 13008
  excluidos.
- Filtro independente com `tar -tf` nos dois ZIPs - OK, sem `.env`, `.git`, `.ai-tests`,
  `node_modules`, logs, dumps, backups, chaves ou certificados; migrations Prisma permitidas.
- Inventario `portal-sama*.zip` no Desktop - OK, apenas os dois ZIPs `.safe.zip`; ZIP antigo
  `portal-sama-api.zip` ausente.
- Remocao segura de `evidencias/auditoria-deploy-15/backup-drill/backups/` - OK.
- Evidencia consolidada salva em
  `evidencias/auditoria-deploy-15/phase1-artifacts/package-validation-summary.json`.
- Evidencia declarativa de rotacao administrativa informada salva em
  `evidencias/auditoria-deploy-15/phase1-artifacts/admin-credential-rotation-reported.json`.
- `node scripts/validate-secret-rotation.js --json --require-integrations --require-web-push`
  com variaveis sinteticas em memoria - OK, 0 falhas e 0 warnings; evidencia sem valores
  sensiveis salva em `secret-rotation-check-synthetic.json`.
- `node scripts/validate-secret-rotation.js --help` - OK.
- `npm.cmd run lint` em `portal-sama-api` - OK.
- `npm.cmd run package:safe -- -EvidencePath ..\portal-sama-docs\evidencias\auditoria-deploy-15\phase1-artifacts\api-safe-package.json`
  reexecutado apos o novo preflight - OK, novo SHA-256 da API:
  `852022dcac4512a0fe30a52a3744353a4e67589c22f5dfdf159f571732575822`.
- Filtro independente com `tar -tf` no ZIP seguro da API regenerado - OK.

### Fase 11 Perfis

- Matriz API por perfil contra API local real - OK, 102 checks e 0 falhas apos correcao.
- Smoke Playwright por perfil contra web/API locais reais - OK, 4 perfis e 0 falhas.
- `npm.cmd test -- --runInBand src/modules/documents/documents.service.spec.ts` em
  `portal-sama-api` - OK, 1 suite e 33 testes.
- `npm.cmd test -- --runInBand --runTestsByPath ...documents.service.spec.ts ...clients.service.spec.ts ...rbac/default-rbac.spec.ts`
  em `portal-sama-api` - OK, 3 suites e 46 testes.
- `npm.cmd test -- --runInBand` em `portal-sama-api` - OK, 44 suites e 276 testes.
- `npm.cmd run test:e2e` em `portal-sama-api` - OK, 1 suite e 136 testes.
- `npm.cmd run lint` em `portal-sama-api` - OK.
- `npm.cmd run build` em `portal-sama-api` - OK.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-api` - OK, 0 vulnerabilidades.
- `npm.cmd test` em `portal-sama-web` - OK, 9 contratos.
- `npm.cmd run test:e2e` em `portal-sama-web` - OK, 13 passaram e 1 skip.
- `npm.cmd run lint` em `portal-sama-web` - OK.
- `npm.cmd run build` em `portal-sama-web` - OK.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web` - OK, 0 vulnerabilidades.
- Varredura de evidencias da Fase 11 por senha/segredos temporarios - OK, sem ocorrencias novas.

### Fase 10 Web Push

- Validacao Playwright Chromium headed/persistente contra API local real - OK, subscribe,
  preferencia, push `1/1` e unsubscribe.
- Consulta MySQL no banco `portal_sama_webpush_phase10` - OK, `SENT=1`, `active=0`,
  `fail_count_sum=0`.
- `npm.cmd test` em `portal-sama-web` - OK, 9 contratos.
- `npm.cmd run lint` em `portal-sama-web` - OK.
- `npm.cmd run build` em `portal-sama-web` - OK.
- `npm.cmd test -- --runInBand --runTestsByPath ...notifications.service.spec.ts ...notifications-push.service.spec.ts`
  em `portal-sama-api` - OK, 2 suites e 28 testes.
- `npm.cmd run build` em `portal-sama-api` - OK.
- `npm.cmd run lint` em `portal-sama-api` - OK.

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

- `portal-sama-api`: `M src/modules/documents/documents.service.ts` e
  `M src/modules/documents/documents.service.spec.ts`, alem das mudancas de Fase 1 em
  `README.md`, `package.json`, `scripts/create-safe-package.ps1` e
  `scripts/validate-secret-rotation.js`.
- `portal-sama-web`: `M scripts/web-contract-tests.mjs`,
  `M src/services/notifications.service.ts`, alem das mudancas de Fase 1 em `README.md`,
  `package.json` e `scripts/create-safe-package.ps1`, e ajuste em
  `tests/e2e/real-auth.spec.ts`.
- `portal-sama-docs`: `M AUDITORIA-DEPLOY-16-EXECUCAO-CORRECOES-GO-LIVE.md` e
  `M CONTEXTO-CODEX-ATUAL.md`, `D evidencias/auditoria-deploy-15/backup-drill/backups/...`,
  alem de `?? evidencias/auditoria-deploy-15/phase1-artifacts/`,
  `?? evidencias/auditoria-deploy-15/real-homologation-20260611/`,
  `?? evidencias/auditoria-deploy-15/webpush-phase10/` e
  `?? evidencias/auditoria-deploy-15/profiles-phase11/`.

## Proximo passo exato

Continuar sem recomecar a auditoria:

1. Prioridade seguinte: confirmar/rotacionar os demais segredos potencialmente expostos:
   `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET`, `CSRF_SECRET`, `DATABASE_URL`, `ACESSORIAS_TOKEN`,
   Web Push/VAPID, `CERTIFICATE_ENCRYPTION_KEY` e tokens externos.
   - Com os valores reais ja no ambiente, rodar no container/API:
     `npm run ops:secrets:check -- --require-integrations --require-web-push`.
2. Se a nova credencial administrativa puder ser fornecida de forma segura somente via variavel de
   ambiente, repetir `smoke:auth` e `test:e2e:real` para verificar a credencial rotacionada.
3. Para fechar Fase 11 real, obter credenciais/dados autorizados por perfil e rodar
   `smoke:permissions`/matriz real sem salvar senhas.
4. Para fechar Fase 10 externa, validar Web Push em HTTPS publico com VAPID oficial e clique
   nativo assistido.

## Cuidado

- Nao reverter mudancas locais.
- Nao expor `.env`.
- Fase 1 esta mitigada para artefatos locais seguros; credencial administrativa compartilhada no
  chat foi informada como rotacionada. Continua critica ate rotacao/confirmacao externa dos demais
  segredos.
- Fase 11 local encontrou e corrigiu autorizacao ampla indevida de `MANAGER` em documentos.
- API E2E agora esta verde localmente.
- E2E Web Home agora esta verde localmente.
- API audit agora esta verde localmente.
- ClamAV strict foi validado localmente em container, mas homologacao/producao ainda deve replicar
  `SAMA_UPLOAD_SCAN_MODE=strict` no runtime correto.
- Backup/verify/restore drill foi validado localmente em Docker, mas producao ainda deve usar
  destino de backup externo/seguro e alvo de restore isolado equivalente ao ambiente real.
- Web Push foi validado localmente com navegador real e provedor real, mas HTTPS publico,
  VAPID oficial e clique nativo ainda dependem de ambiente real assistido.
- Autenticacao admin real foi validada no dominio publico; perfis reais completos ainda dependem
  de credenciais/dados autorizados por perfil.
