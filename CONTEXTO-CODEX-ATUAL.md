# CONTEXTO CODEX ATUAL

Atualizado em: 2026-06-22
Sessao atual: continuidade da implementacao fim a fim do Portal Sama apos conciliacao de precedencia documental.

## Precedencia obrigatoria

Os documentos Markdown da raiz do workspace tem precedencia sobre registros auxiliares em `docs/`, `evidencias/` e auditorias antigas.

Ordem pratica:

1. `00-LEIA-ME-NOVA-ARQUITETURA-CODEX.md`
2. `12-RUNBOOK-ACESSORIAS.md`
3. `15-ANALISE-SEMANTICA-LEGADO-VS-NOVA-ARQUITETURA.md`
4. `16-BACKEND-API-BANCO-DETALHADO.md`
5. `17-FRONTEND-UX-REAPROVEITAMENTO-LEGADO.md`
6. `18-ZAPSIGN-MIGRACAO-LEGADO-PARA-NESTJS.md`
7. `19-SEGURANCA-GOVERNANCA-DOCUMENTOS.md`
8. `CONTEXTO-CODEX-ATUAL.md`
9. `Testes-da-aplicação-DEPLOY.md`
10. `20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md`, como roteiro operacional de fases, status e evidencias.

Os arquivos em `docs/` sao acompanhamento/evidencia e nao podem liberar uma fase se conflitarem com a raiz.

## Estado formal do roteiro

- Fases 0 a 8: registradas como concluidas no acompanhamento.
- Fase 8: `CONCLUIDA` formalmente em 2026-06-22 apos execucao real do `ops:phase8` no ambiente alvo.
- Fase 9: `EM_EXECUCAO`; tela web de smoke backend, e2e real da rota e runner autenticado `smoke:phase9` implementados/validados em 2026-06-22.
- Fase 10: ainda nao autorizada enquanto a Fase 9 nao tiver smoke real/evidencia final.

Evidencia principal da conclusao: `npm run ops:phase8 -- --json --soft --backup-output-dir /tmp/portal-sama-phase8-backups --target-storage-path /tmp/portal-sama-restore-storage --apply-database --apply-storage --confirm RESTORE_DRILL_TARGET_IS_ISOLATED` executado no container da API do EasyPanel retornou `ok=true`, `failed=0`, `blocked=0`, `warnings=4`.

## Implementado recentemente

### API - Fase 7 ZapSign

- Provider ZapSign criado em `portal-sama-api/src/modules/contracts/providers/zapsign/`.
- `ZapsignClient`, mapper, service, controller de webhook e specs unitarios adicionados.
- `ContractsModule` passou a suportar `INTERNAL` e `ZAPSIGN`.
- `POST /contracts/:id/send-signature` envia internamente ou via ZapSign.
- `POST /contracts/:id/zapsign/sync` sincroniza status externo.
- `POST /webhooks/zapsign` publico exige segredo por header, aplica throttle e confirma status via consulta ativa.
- Assinatura publica interna e bloqueada para contrato `ZAPSIGN`.
- Auditoria ZapSign registra eventos started/succeeded/failed/webhook/signed sem token bruto.

### Web - Fase 7 ZapSign

- Tela de contratos exibe provider, ambiente, status/link ZapSign, envio externo e sincronizacao.
- Pagina publica de assinatura trata contrato ZapSign como assinatura externa.
- Services, tipos e schemas de contrato foram atualizados para provider/envelope ZapSign.

### Web - Fase 9 smoke backend

- Criada rota autenticada `/dev/fase-9-smoke`.
- Criados `src/services/health.service.ts` e `src/types/health.ts` para consultar `GET /health` sem mascarar HTTP 503.
- Criada `src/pages/dev/Phase9SmokePage.tsx` com checks de health, `/auth/me`, clientes, contratos, Acessorias e documentos.
- A tela lista dados via backend e deixa acoes mutaveis sob clique explicito: criar contrato `INTERNAL`/`ZAPSIGN` sandbox, executar sync Acessorias controlado sem aplicar workspace, e enviar documento para cliente.
- Navegacao admin recebeu item `Smoke F9`.
- Teste de contrato web passou a validar a rota e os servicos da Fase 9.
- `test:e2e:real` passou a cobrir tambem a rota `/dev/fase-9-smoke` apos login real.
- Criado `scripts/portal-phase9-smoke.mjs` e script `npm.cmd run smoke:phase9` para validar, contra o dominio publico, rota `/dev/fase-9-smoke`, CSRF/login, `/auth/me`, `GET /health`, clientes, contratos, Acessorias e documentos.
- `smoke:phase9` nunca imprime senha, tokens, CSRF, cookies ou corpo bruto de respostas; a saida fica limitada a status, contagens e ids nao sensiveis.
- O runner suporta modo read-only por padrao e acoes controladas somente com `--apply-actions` ou `PORTAL_PHASE9_APPLY_ACTIONS=1`: criar contrato `INTERNAL`, contrato `ZAPSIGN` sandbox, sync Acessorias sem aplicar workspace e upload valido/invalido.
- `homologation:real` passou a incluir `smoke:phase9`, com `--skip-phase9` para rodadas publicas sem credenciais.

### Docs - Fase 8 e conciliacao

- Criado `docs/21-CONTRATO-API-FRONTEND.md`.
- Atualizados `docs/08-CHECKLIST-GO-LIVE.md`, `docs/09-MATRIZ-PERMISSOES-RBAC.md`, `docs/10-MATRIZ-ROTAS-PUBLICAS.md` e `docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md`.
- `00-LEIA-ME-NOVA-ARQUITETURA-CODEX.md` recebeu a secao `Precedencia documental`.
- `docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md` registrou historicamente o bloqueio formal da Fase 8; nesta atualizacao, a evidencia real de 2026-06-22 concluiu a fase e autorizou a Fase 9.

### API - Orquestracao operacional Fase 8

- Criado `portal-sama-api/scripts/run-operational-phase8.js`.
- Adicionado `npm.cmd run ops:phase8`.
- Corrigida a invocacao dos sub-scripts no Windows: o orquestrador agora usa `npm_execpath` quando disponivel e `cmd.exe /c npm.cmd` como fallback, evitando `spawnSync npm.cmd EINVAL`.
- O parse das saidas JSON dos subchecks foi endurecido: o orquestrador agora parseia a saida bruta antes de sanitizar/truncar e sanitiza recursivamente o objeto parseado, evitando falso positivo se a saida real ultrapassar o limite textual.
- O comando orquestra readiness, schema, secrets, backup, verify e restore drill.
- Ele exige `ops:secrets:check` com `--require-integrations`, `--require-web-push` e `--require-zapsign`.
- Ele exige backup com dump de banco e storage archive, alem de restore drill em modo apply para banco e storage isolados.
- Skips, alvo ausente ou dry-run deixam a Fase 8 `BLOQUEADA`; isso e intencional para nao liberar Fase 9 por acidente.
- `ops:secrets:check` passou a aceitar `--require-zapsign` e validar `ZAPSIGN_API_TOKEN`, `ZAPSIGN_API_TOKEN_SANDBOX` quando aplicavel e `ZAPSIGN_WEBHOOK_SECRET`, sem imprimir valores.

## Validacoes executadas recentemente

### API

- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 56 suites e 347 testes.
- `npm.cmd run test:e2e` - OK, 1 suite e 148 testes.
- `npm.cmd run prisma:validate` - OK na rodada da Fase 7.
- Specs focadas contratos/ZapSign - OK, 4 suites e 20 testes.
- Spec de rate limit - OK, 1 suite e 6 testes.
- `git diff --check` - OK.

### Web

- `npm.cmd run lint` - OK apos Fase 9 smoke.
- `npm.cmd run build` - OK apos Fase 9 smoke.
- `npm.cmd test -- --runInBand` - OK, 13 testes de contrato.
- `npm.cmd run test:e2e:real` sem credenciais - OK como diagnostico, 2 testes pulados por env ausente.
- `node --check scripts/portal-phase9-smoke.mjs` - OK.
- `node --check scripts/portal-real-homologation.mjs` - OK.
- `npm.cmd run smoke:phase9 -- --json --soft --skip-zapsign --skip-acessorias-sync` - `phase9-route-shell` passou contra `https://portal.samacontabil.com.br/dev/fase-9-smoke`; falhou apenas em `credentials` por ausencia local de `PORTAL_AUTH_USERNAME` e `PORTAL_AUTH_PASSWORD`.
- `git diff --check` - OK.
- Servidor local Vite foi iniciado anteriormente em `http://127.0.0.1:5173`.

### Deploy real Fase 9

- `npm.cmd run homologation:real -- --json --soft --skip-auth --skip-permissions --skip-e2e --evidence-dir .ai-tests/homologation-real-phase9` contra `https://portal.samacontabil.com.br` retornou `ok=true`.
- Checks publicos aprovados: `public-web` HTTP 200, `api-health` HTTP 200 com `database=up` e `storage=up`, `cors-preflight` HTTP 204, `auth-csrf` HTTP 200 com token/cookie.
- Checagem Playwright anonima em `https://portal.samacontabil.com.br/dev/fase-9-smoke` redirecionou corretamente para `https://portal.samacontabil.com.br/login` e exibiu `Entrar no portal`.
- `npm.cmd run homologation:real -- --json --soft --skip-auth --skip-permissions --skip-e2e --skip-phase9 --evidence-dir .ai-tests/homologation-real-phase9` retornou `ok=true` para rodada publica sem credenciais.
- `npm.cmd run homologation:real -- --json --soft --skip-auth --skip-permissions --skip-e2e --evidence-dir .ai-tests/homologation-real-phase9` retornou `ok=false` somente porque `smoke:phase9` ficou `blocked` por ausencia local de `PORTAL_AUTH_USERNAME` e `PORTAL_AUTH_PASSWORD`; `smoke:public` passou.
- Runner completo sem skips manteve Fase 9 `EM_EXECUCAO`: autenticacao, matriz de permissoes e e2e real ainda dependem de variaveis locais reais (`PORTAL_AUTH_USERNAME`, `PORTAL_AUTH_PASSWORD`, `PORTAL_REAL_E2E`, `PORTAL_E2E_USERNAME`, `PORTAL_E2E_PASSWORD`, matriz de permissoes).
- Evidencias locais ignoradas pelo git incluem `.ai-tests/homologation-real-phase9/homologation-real-20260622T172552Z.json`, `.ai-tests/homologation-real-phase9/homologation-real-20260622T172645Z.json`, `.ai-tests/homologation-real-phase9/homologation-real-20260622T195241Z.json` e `.ai-tests/homologation-real-phase9/homologation-real-20260622T195316Z.json`.

### Operacional Fase 8

- `npm.cmd run ops:readiness -- --soft --json` sem env carregado apontou env incompleto e banco inacessivel pelo hostname de container.
- Segunda rodada local com env sintetico e MySQL em `127.0.0.1:3306` confirmou:
  - database OK;
  - migrations aplicadas: 32;
  - RBAC: 99 permissoes e 9 papeis;
  - storage sintetico OK.
- Readiness local continuou falhando por ausencia de usuario ativo DEV/ADMIN e ClamAV ausente.
- `npm.cmd run ops:schema:check -- --soft --json` contra MySQL local passou com `critical=0`, `warnings=1`; aviso de collation default do database.
- `npm.cmd run ops:secrets:check -- --soft --json` passou com secrets sinteticos fortes e URL remota sintetica.
- Backup/verify/restore drill sintetico passou com storage em `.ai-tests`, `--include-storage-archive` e `--skip-database`.
- `node --check scripts/run-operational-phase8.js` - OK.
- `node --check scripts/validate-secret-rotation.js` - OK.
- `npm.cmd run ops:phase8 -- --help` - OK.
- `npm.cmd run ops:phase8 -- --json --soft --no-evidence --skip-readiness --skip-schema --skip-secrets --skip-backup --skip-backup-artifacts --skip-verify --skip-restore` - OK como diagnostico bloqueado: `ok=false`, `blocked=8`.
- Apos a correcao Windows, `npm.cmd run ops:phase8 -- --json --soft --no-evidence --skip-backup --skip-backup-artifacts --skip-verify --skip-restore` executou os subchecks reais; resultado esperado neste terminal sem env alvo: `ok=false`, `failed=3`, `blocked=5`, com readiness/schema/secrets falhando por ambiente/banco/secrets ausentes.
- Apos hardening do parse/sanitizacao, tanto `npm.cmd run ops:phase8 -- --json --soft --no-evidence --skip-backup --skip-backup-artifacts --skip-verify --skip-restore` quanto `node scripts/run-operational-phase8.js --json --soft --no-evidence --skip-backup --skip-backup-artifacts --skip-verify --skip-restore` mantiveram resultado esperado `ok=false`, `failed=3`, `blocked=5`.
- `ops:secrets:check -- --require-integrations --require-web-push --require-zapsign` com secrets sinteticos fortes - OK, `failed=0`, `warnings=0`, sem valores sensiveis na saida.
- Rodada real no EasyPanel em 2026-06-22: `ops:phase8` retornou `ok=true`, `failed=0`, `blocked=0`, `warnings=4`.
- Readiness real: `ok=true`; ambiente de producao configurado, banco respondeu, schema/migrations passaram, RBAC com 99 permissoes e 9 papeis, usuario privilegiado ativo, storage privado OK e ClamAV detectou EICAR.
- Schema real: `ok=true`, `critical=0`, `warnings=0`, database `banco-sama` em `utf8mb4_unicode_ci`.
- Secrets reais: `ok=true`, `failed=0`, `warnings=3`; nenhum valor foi registrado.
- Backup real: `database.sql.gz`, `storage-manifest.json`, `storage.tar.gz` criados em `/tmp/portal-sama-phase8-backups/portal-sama-20260622T165857Z-23e713`.
- Verify real: `ok=true`, `failed=0`; gzip do banco validado; hashes registrados pela saida do comando.
- Restore drill real: `ok=true`, `failed=0`, `dryRun=false`; `database.sql.gz` importado no banco isolado `banco-sama_restore_drill` e `storage.tar.gz` extraido em `/tmp/portal-sama-restore-storage`.
- Evidencia JSON no ambiente alvo: `/app/.ai-tests/phase8-operational/phase8-2026-06-22T16-58-37-116Z.json`.

## Avisos nao bloqueantes apos Fase 8

1. Copiar a evidencia JSON e os artefatos de backup para armazenamento operacional seguro fora de `/tmp`.
2. Registrar externamente a data/responsavel da rotacao de secrets (`SAMA_SECRET_ROTATION_CONFIRMED_AT` nao estava configurado).
3. Planejar rotacao/hardening de `CERTIFICATE_ENCRYPTION_KEY` para 48+ caracteres e `ACESSORIAS_TOKEN` para 40+ caracteres.
4. O banco de restore esta no mesmo host do banco da aplicacao; foi aceito por estar isolado por nome (`banco-sama_restore_drill`) e por confirmacao explicita `RESTORE_DRILL_TARGET_IS_ISOLATED`.

## Status dos worktrees

### `portal-sama-api`

Ultimo commit registrado:

- `3523716 feat: add zapsign contracts and phase 8 ops gate`

### `portal-sama-web`

Ultimo commit registrado:

- `b17a167 test: cover phase 9 smoke route in real e2e`
- `d3bd23e test: add phase 9 authenticated smoke runner`

### `portal-sama-docs`

Ultimo commit registrado antes desta atualizacao de Fase 9:

- `190f815 docs: record phase 9 post deploy smoke`

## Proximo chat deve fazer

1. Ler primeiro os documentos da raiz listados em `Precedencia obrigatoria`.
2. Ler `docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md` apenas como evidencia subordinada.
3. Confirmar que a Fase 8 esta `CONCLUIDA`.
4. Confirmar que a Fase 9 esta `EM_EXECUCAO` e que a tela/runner web estao nos commits `73da23f`, `b17a167` e `d3bd23e`.
5. Para concluir formalmente a Fase 9, executar smoke real autenticado com usuario autorizado, preferencialmente `npm.cmd run smoke:phase9 -- --json --soft` primeiro em modo read-only e depois com `PORTAL_PHASE9_APPLY_ACTIONS=1` para as acoes controladas, sem imprimir valores de credenciais.
6. Depois, executar `npm.cmd run homologation:real -- --json --soft --skip-permissions --evidence-dir .ai-tests/homologation-real-phase9` com `PORTAL_REAL_E2E=1`, `PORTAL_AUTH_USERNAME`, `PORTAL_AUTH_PASSWORD`, `PORTAL_E2E_USERNAME` e `PORTAL_E2E_PASSWORD` configurados no ambiente sem imprimir valores.
7. Nao iniciar Fase 10 enquanto a Fase 9 nao tiver evidencia real final.
8. Ao alterar codigo, reexecutar lint/build/test correspondentes e `git diff --check`.

## Cuidados

- Nao ler nem imprimir `.env` real ou secrets.
- Nao reverter alteracoes locais.
- Nao usar `prisma db push` em ambiente compartilhado/producao.
- Nao mascarar falha operacional com `--skip-*` para liberar fase.
- Nao considerar backup valido sem restore drill.
- Fase 8 ja comprovou backup/restore drill real; preservar a evidencia fora de `/tmp`.
