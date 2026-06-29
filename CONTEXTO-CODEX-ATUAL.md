# CONTEXTO CODEX ATUAL

Atualizado em: 2026-06-29
Sessao atual: Fase 10 concluida; Fase 11 autorizada como proximo passo.

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
8. `21-CONTINUIDADE-FASE-9-EASYPANEL.md`, como evidencia formal da conclusao da Fase 9
9. `22-CONTINUIDADE-FASE-10-FRONTEND.md`, como evidencia formal da conclusao da Fase 10
10. `CONTEXTO-CODEX-ATUAL.md`
11. `Testes-da-aplicação-DEPLOY.md`
12. `20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md`, como roteiro operacional de fases, status e evidencias.

Os arquivos em `docs/` sao acompanhamento/evidencia e nao podem liberar uma fase se conflitarem com a raiz.

## Estado formal do roteiro

- Fases 0 a 8: registradas como concluidas no acompanhamento.
- Fase 8: `CONCLUIDA` formalmente em 2026-06-22 apos execucao real do `ops:phase8` no ambiente alvo.
- Fase 9: `CONCLUIDA` em 2026-06-29 apos `homologation:real --skip-permissions` retornar `ok=true`, com `smoke:public`, `smoke:auth`, `smoke:phase9` e `test:e2e:real` aprovados.
- Fase 10: `CONCLUIDA` em 2026-06-29; etapas 10.1 a 10.9 concluidas e validadas.
- Fase 11: autorizada como proximo passo; ainda nao iniciada.

Evidencia principal da conclusao da Fase 8: `npm run ops:phase8 -- --json --soft --backup-output-dir /tmp/portal-sama-phase8-backups --target-storage-path /tmp/portal-sama-restore-storage --apply-database --apply-storage --confirm RESTORE_DRILL_TARGET_IS_ISOLATED` executado no container da API do EasyPanel retornou `ok=true`, `failed=0`, `blocked=0`, `warnings=4`.

Evidencia principal da conclusao da Fase 9: `portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260629T134038169Z.json`, com `ok=true`, `smoke:public=passed`, `smoke:auth=passed`, `smoke:phase9=passed` e `test:e2e:real=passed`.

Evidencia principal da conclusao da Fase 10: `22-CONTINUIDADE-FASE-10-FRONTEND.md`, com manifesto de rotas, navegacao por jornada, `QueryTabs`, `StateBlock`, primitivas `AppPanel`, `InfoCard`, `DataTable`, `Timeline`, `AppDrawer`, `AppModal` e `Form`, politica frontend de permissoes, normalizacao do API client, checklist de seguranca visual, lint OK, build OK, 21 testes de contrato web OK, `test:e2e` OK com 27 passed/2 skipped e `git diff --check` OK.

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
- `smoke:phase9` aceita `--evidence-dir` ou `PORTAL_PHASE9_EVIDENCE_DIR` para gravar JSON sanitizado proprio da Fase 9, sem sobrescrever evidencias antigas.
- `smoke:phase9` passou a incluir detalhes publicos/sanitizados de resposta quando uma checagem falha, sem imprimir corpo bruto, tokens ou cookies.
- Em 2026-06-25, o runner foi corrigido para extrair tambem erros envelopados pela API em `error.code`, `error.message` e `error.details`; a rodada autenticada passou a expor a causa sanitizada dos bloqueios de Acessorias e upload valido.
- Em 2026-06-25, o runner passou a diferenciar `failed` de `blocked` para credenciais ausentes, lock operacional Acessorias (`ACESSORIAS_HEAVY_JOB_LOCKED`) e indisponibilidade operacional do upload (`DOCUMENT_SCAN_UNAVAILABLE`/`DOCUMENT_QUARANTINE_UNAVAILABLE`), sem marcar a fase como aprovada.
- A tela e o runner Fase 9 passaram a exibir `uploadQuarantine` retornado por `/health`, para diagnosticar falhas de upload antes de executar uma mutacao de documento.
- Em 2026-06-25, a decisao tecnica para o timeout longo do `acessorias-controlled-sync` foi implementar polling por `GET /integrations/acessorias/sync-runs`: a tela mostra runs recentes e o runner passa a consultar os runs apos timeout local/HTTP 504, aceitando a acao somente quando catalogo e entregas fecharem `SUCCESS`.
- `homologation:real` passou a incluir `smoke:phase9`, com `--skip-phase9` para rodadas publicas sem credenciais.
- `homologation:real` tambem foi ajustado para evitar colisao de nome de evidencia quando duas rodadas iniciam no mesmo segundo; o arquivo agora usa milissegundos e tenta sufixo seguro sem sobrescrever evidencia antiga.

### API - Fase 9 diagnostico Acessorias

- Em 2026-06-25, `AcessoriasRegistrationsService.syncCompaniesCatalog` foi corrigido para finalizar o `AcessoriasSyncRun` como `FAILED` quando o lock pesado do Acessorias nao puder ser adquirido.
- O comportamento HTTP continua preservando o conflito `409` para o chamador; a correcao evita deixar sync runs orfaos em `RUNNING` quando ha job concorrente.
- Teste unitario cobre o caso `ACESSORIAS_HEAVY_JOB_LOCKED`, garantindo `syncRuns.finish` com `lock_acquired=false` e sem notificar inicio de job que nao comecou.
- `GET /health` passou a validar tambem a area de quarentena de upload (`SAMA_UPLOAD_QUARANTINE_DIR` ou `STORAGE_PRIVATE_PATH/uploads/_quarantine`) e retorna `uploadQuarantine`; health fica `ok=false` se ela nao estiver gravavel.
- `AcessoriasOperationalSyncService` passou a marcar catalogo e entregas com o trigger `manual_operational_sync`, permitindo que o smoke Fase 9 diferencie os runs da acao operacional de outros jobs manuais.

### Docs/operacao - Continuidade 2026-06-26

- Criado `21-CONTINUIDADE-FASE-9-EASYPANEL.md` na raiz como documento principal da continuidade da Fase 9.
- `npm.cmd run smoke:phase9 -- --json --soft --evidence-dir .ai-tests/phase9-smoke` executado em `portal-sama-web` contra `https://portal.samacontabil.com.br` confirmou HTTP 200 na rota `/dev/fase-9-smoke`; resultado `ok=false`, `failed=0`, `blocked=1` somente por ausencia local de `PORTAL_AUTH_USERNAME` e `PORTAL_AUTH_PASSWORD`.
- Evidencia sanitizada gerada em `portal-sama-web/.ai-tests/phase9-smoke/phase9-smoke-20260626T180901611Z.json`.
- `GET https://portal.samacontabil.com.br/api-v2/health` retornou `ok=true`, `database=up`, `storage=up`, `uploadQuarantine=up`, `timestamp=2026-06-26T18:09:08.151Z`.
- Conclusao: o deploy publico ja expoe `uploadQuarantine`, mas isso ainda nao prova scanner de malware operacional; a prova continua sendo upload PDF valido no smoke autenticado com `PORTAL_PHASE9_APPLY_ACTIONS=1`.
- Logs informados pelo usuario em 2026-06-26:
  - `smoke:phase9` autenticado com `applyActions=true` retornou `ok=false`, `failed=0`, `blocked=1`; passaram CSRF/login, `/auth/me`, health, clientes, contratos, Acessorias, criacao de contrato `INTERNAL`, criacao de contrato `ZAPSIGN` sandbox, polling de `acessorias-controlled-sync`, rejeicao de SVG invalido e logout.
  - O unico bloqueio do `smoke:phase9` foi `upload-valid-document` com HTTP 503 `DOCUMENT_SCAN_UNAVAILABLE`, `reason=scanner_required`.
  - Evidencia local: `portal-sama-web/.ai-tests/phase9-smoke/phase9-smoke-20260626T183520231Z.json`.
  - `homologation:real --skip-permissions` retornou `smoke:public=passed`, `smoke:auth=passed`, `smoke:phase9=failed` por causa do bloqueio de upload e `test:e2e:real=failed` porque o navegador esperava o heading `Smoke backend`, mas o contexto mostrou a tela de login.
  - Evidencia local: `portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260626T184910450Z.json`.
- Continuidade de 2026-06-29 apos ajuste informado de `clamscan` no EasyPanel:
  - `/api-v2/health` publico retornou `ok=true`, `database=up`, `storage=up`, `uploadQuarantine=up`, `timestamp=2026-06-29T11:14:28.271Z`.
  - `smoke:phase9` autenticado com `applyActions=true` continuou `ok=false`, `failed=0`, `blocked=1`; Acessorias passou por polling, contratos passaram, upload SVG invalido foi rejeitado, mas `upload-valid-document` continuou HTTP 503 `DOCUMENT_SCAN_UNAVAILABLE`, `reason=scanner_required`.
  - Evidencia local: `portal-sama-web/.ai-tests/phase9-smoke/phase9-smoke-20260629T112420826Z.json`.
  - `homologation:real --skip-permissions` continuou com `smoke:public=passed`, `smoke:auth=passed`, `smoke:phase9=failed` por scanner e `test:e2e:real=failed` esperando `Smoke backend` enquanto o contexto mostrou tela de login.
  - Evidencia local: `portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260629T113002807Z.json`.
  - Conclusao operacional: o `clamscan` ainda nao esta acessivel para o processo/container da API ou esta sem base de assinaturas. O Dockerfile atual ja instala `clamav` e define `SAMA_UPLOAD_SCAN_BIN=/usr/bin/clamscan`, entao confirmar se o EasyPanel fez rebuild/redeploy com essa imagem, se nenhuma env sobrescreveu o binario e se `freshclam`/assinaturas existem em `/var/lib/clamav`. Confirmar dentro do container da API `which clamscan`, `clamscan --version` e `ls -la /var/lib/clamav`, depois reiniciar/redeployar.
  - Nova reexecucao do `smoke:phase9` em 2026-06-29T12:02:02.608Z manteve `ok=false`, `failed=0`, `blocked=1`; todos os checks passaram exceto `upload-valid-document`, que continuou HTTP 503 `DOCUMENT_SCAN_UNAVAILABLE`, `reason=scanner_required`. Evidencia local: `portal-sama-web/.ai-tests/phase9-smoke/phase9-smoke-20260629T120202608Z.json`.
  - Apos `freshclam` confirmar bases `daily.cvd`, `main.cvd` e `bytecode.cvd` atualizadas, nova execucao de `smoke:phase9` em 2026-06-29T12:20:12.310Z retornou `ok=true`, `failed=0`, `blocked=0`.
  - Nessa rodada, `upload-valid-document` passou com HTTP 201, `documentId=366b48a6-8ce5-4e04-8e82-92309ab661c8`, `status=PENDING`, `mime=application/pdf`; upload SVG invalido continuou rejeitado; Acessorias fechou `SUCCESS` por polling.
  - Evidencia local: `portal-sama-web/.ai-tests/phase9-smoke/phase9-smoke-20260629T122012310Z.json`.
  - Em seguida, `homologation:real --skip-permissions` gerou `ok=false` porque somente `test:e2e:real` falhou; `smoke:public`, `smoke:auth` e `smoke:phase9` passaram, e `smoke:permissions` foi pulado conforme parametro.
  - Evidencia local: `portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260629T122850371Z.json`.
  - Falhas E2E reais: logout apos clique em `Sair` permaneceu em `/home`, e a rota `/dev/fase-9-smoke` voltou para tela de login em vez de exibir `Smoke backend`.
  - Ajustado `portal-sama-web/tests/e2e/real-auth.spec.ts`: `loginWithRealUser` agora espera a rota autenticada alvo e navega explicitamente como fallback; o teste de logout espera a resposta `POST /auth/logout` antes de exigir `/login`.
  - Validacoes locais apos o ajuste: `npm.cmd run lint` OK, `npx.cmd playwright test tests/e2e/real-auth.spec.ts --reporter=line` OK com testes reais pulados por env ausente, e `npm.cmd test -- --runInBand` OK.
  - Nova execucao de `homologation:real --skip-permissions` em 2026-06-29T12:48:34.080Z manteve `ok=false`, mas com avanco: `smoke:public`, `smoke:auth`, `smoke:phase9` passaram e o teste real de login/home/cookies/logout tambem passou.
  - Evidencia local: `portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260629T124834080Z.json`.
  - Falha remanescente: apenas o teste real da rota `/dev/fase-9-smoke` falhou esperando `Smoke backend`; o contexto mostrou a tela de login renderizada enquanto a expectativa ja estava na rota operacional.
  - Segundo ajuste em `portal-sama-web/tests/e2e/real-auth.spec.ts`: `loginWithRealUser` agora detecta a tela de login pelo heading `Entrar no portal`, aguarda ate 10s apos abrir a rota alvo, faz login se o formulario aparecer e exige que o login desapareca depois de navegar para a rota esperada.
  - Validacoes locais apos o segundo ajuste: `npm.cmd run lint` OK, `npx.cmd playwright test tests/e2e/real-auth.spec.ts --reporter=line` OK com testes reais pulados por env ausente, `npm.cmd test -- --runInBand` OK e `git diff --check` OK no web.
  - Nova execucao de `homologation:real --skip-permissions` em 2026-06-29T13:28:22.279Z retornou `ok=false`: `smoke:public` e `smoke:auth` passaram; `smoke:phase9` falhou apenas em `acessorias-controlled-sync` por HTTP 409 `ACESSORIAS_HEAVY_JOB_LOCKED`; upload PDF valido continuou HTTP 201 e upload SVG invalido continuou rejeitado.
  - Evidencia local: `portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260629T132822279Z.json`.
  - Nessa mesma rodada, `test:e2e:real` falhou porque o helper ainda podia sair cedo demais: `locator.isVisible()` nao aguardou a tela de login renderizar antes de decidir se precisava logar.
  - Terceiro ajuste em `portal-sama-web/tests/e2e/real-auth.spec.ts`: `loginWithRealUser` agora usa `waitFor({ state: 'visible' })`, aguarda resposta `POST /auth/login` OK e navega para a rota esperada somente depois do login HTTP OK.
  - Ajustado `portal-sama-web/scripts/portal-phase9-smoke.mjs`: o Acessorias operational sync agora faz retry de `409 ACESSORIAS_HEAVY_JOB_LOCKED` dentro de `PORTAL_PHASE9_ACESSORIAS_POLL_TIMEOUT_MS`, registrando apenas status/duracao/codigo publico.
  - Validacoes locais apos o terceiro ajuste: `node --check scripts/portal-phase9-smoke.mjs` OK, `node --check scripts/portal-real-homologation.mjs` OK, `npm.cmd run lint` OK, `npm.cmd run build` OK, `npm.cmd test -- --runInBand` OK, `npx.cmd playwright test tests/e2e/real-auth.spec.ts --reporter=line` OK com testes reais pulados por env ausente, `git diff --check` OK no web.
  - O processo do Codex nao enxergava `PORTAL_REAL_E2E`, `PORTAL_AUTH_USERNAME`, `PORTAL_AUTH_PASSWORD`, `PORTAL_E2E_USERNAME`, `PORTAL_E2E_PASSWORD` nem `PORTAL_PHASE9_APPLY_ACTIONS` nos escopos `Process`, `User` ou `Machine`; por isso a homologacao real final foi executada pelo usuario no terminal onde as credenciais estavam carregadas.
  - Homologacao real final executada pelo usuario em 2026-06-29T13:40:38.169Z retornou `ok=true`, concluindo a Fase 9.
  - Evidencia local: `portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260629T134038169Z.json`.
  - Resultado final: `smoke:public=passed`, `smoke:auth=passed`, `smoke:phase9=passed`, `smoke:permissions=skipped`, `test:e2e:real=passed`; os 2 testes reais do Playwright passaram em 9.3s.
  - Acessorias fechou `SUCCESS` por polling, upload PDF valido passou com HTTP 201, upload SVG invalido foi rejeitado com HTTP 400, contratos `INTERNAL` e `ZAPSIGN` sandbox passaram.
  - Historico superado: naquele momento o proximo passo era iniciar Fase 10; atualmente a Fase 10 ja esta `CONCLUIDA` e a Fase 11 esta autorizada.

### Web - Fase 10 frontend/design system

- Criado `portal-sama-web/src/app/route-manifest.ts` para documentar rotas publicas, privadas, redirects, DEV e fallback.
- Reorganizado `portal-sama-web/src/components/layout/navigation.tsx` em grupos de jornada: `Operacao`, `Cliente`, `Legalizacao`, `Onboarding`, `T.I`, `Gestao`, `Administracao` e `DEV`.
- Criado `portal-sama-web/src/components/ui/QueryTabs.tsx` para abas acessiveis controladas por query string.
- Criado `portal-sama-web/src/components/ui/StateBlock.tsx` para estados padrao `loading`, `empty`, `error`, `permission` e `rate-limit`.
- Atualizado `portal-sama-web/src/components/ui/AppPanel.tsx` para aceitar descricao e rodape.
- Criados `portal-sama-web/src/components/ui/InfoCard.tsx`, `DataTable.tsx`, `Timeline.tsx`, `AppDrawer.tsx` e `AppModal.tsx` para padronizar cards, tabelas, historicos e overlays.
- Criado `portal-sama-web/src/components/ui/Form.tsx` com `FormField`, `FormTextField`, `FormTextareaField`, `FormSelectField`, `FormCheckboxField`, `FormRadioGroup` e `FormActions`.
- Atualizado `portal-sama-web/src/pages/auth/LoginPage.tsx` para usar as primitivas de formulario mantendo `react-hook-form`, `zodResolver` e schema Zod.
- Criado `portal-sama-web/src/components/common/permission-policy.ts` e atualizado `PermissionGate`/navegacao para usar a mesma checagem frontend de permissoes.
- `PermissionGate` ganhou `blockedFallback`; a politica documenta que o frontend controla somente visibilidade/bloqueio visual, sem substituir autorizacao backend.
- Atualizado `portal-sama-web/src/services/api.ts` com `ApiErrorInfo`, `getApiErrorInfo`, `isApiStatus`, mensagens padrao para 401/403/409/422/429, leitura de `Retry-After` e limpeza de sessao quando 401 persiste apos refresh.
- Criado `portal-sama-web/src/app/frontend-security.ts` com checklist de seguranca visual, mascaramento de textos sensiveis, log DEV sanitizado e helper de confirmacao sensivel.
- Atualizados `portal-sama-web/src/features/intro/components/IntroGate.tsx` e `portal-sama-web/src/features/intro/intro-assets.ts` para nao logar erro bruto/payload bruto.
- Atualizado `portal-sama-web/src/index.css` com estilos globais para tabs, estados, paineis, cards, tabelas, timeline, drawer, modal e forms mantendo raio de 8px.
- Atualizado `portal-sama-web/scripts/web-contract-tests.mjs`; a suite agora tem 21 contratos e cobre os novos itens da Fase 10.
- Validacoes executadas: `npm.cmd run lint`, `npm.cmd test -- --runInBand`, `npm.cmd run build`, `npm.cmd run test:e2e` e `git diff --check` passaram no web.
- O E2E mobile da sidebar foi atualizado para abrir o novo grupo `Cliente` antes de validar o link `Clientes`, refletindo a navegacao por jornada da Fase 10.

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
- `npm.cmd test -- --runInBand` - OK, 56 suites e 349 testes na rodada de 2026-06-25.
- `npm.cmd run test:e2e` - OK, 1 suite e 148 testes na rodada de 2026-06-25.
- `npm.cmd run prisma:validate` - OK na rodada da Fase 7.
- Specs focadas contratos/ZapSign - OK, 4 suites e 20 testes.
- Spec de rate limit - OK, 1 suite e 6 testes.
- `git diff --check` - OK na rodada de 2026-06-25.

### Web

- `npm.cmd run lint` - OK apos correcao do runner em 2026-06-25.
- `npm.cmd run build` - OK apos correcao do runner em 2026-06-25.
- `npm.cmd test -- --runInBand` - OK, 13 testes de contrato.
- `npm.cmd run test:e2e:real` sem credenciais - OK como diagnostico, 2 testes pulados por env ausente.
- `node --check scripts/portal-phase9-smoke.mjs` - OK.
- `node --check scripts/portal-real-homologation.mjs` - OK.
- `npm.cmd run smoke:phase9 -- --json --soft --skip-zapsign --skip-acessorias-sync` - `phase9-route-shell` passou contra `https://portal.samacontabil.com.br/dev/fase-9-smoke`; falhou apenas em `credentials` por ausencia local de `PORTAL_AUTH_USERNAME` e `PORTAL_AUTH_PASSWORD`.
- Apos a correcao de evidencia, `npm.cmd test`, `npm.cmd run lint`, `npm.cmd run build`, `node --check scripts/portal-real-homologation.mjs` e `git diff --check` passaram no web.
- Apos a adicao de evidencia direta/detalhes sanitizados ao `smoke:phase9`, `node --check scripts/portal-phase9-smoke.mjs`, `npm.cmd test`, `npm.cmd run lint`, `npm.cmd run build` e `git diff --check` passaram.
- Em 2026-06-25, `npm.cmd run smoke:phase9 -- --json --soft --evidence-dir .ai-tests/phase9-smoke` confirmou novamente o shell HTML de `/dev/fase-9-smoke` em producao; a rodada diagnostica mais recente ficou `failed=0`, `blocked=1` somente por ausencia local de `PORTAL_AUTH_USERNAME` e `PORTAL_AUTH_PASSWORD`, criando `.ai-tests/phase9-smoke/phase9-smoke-20260625T191512609Z.json`.
- Em 2026-06-25, as credenciais corretas foram carregadas de `portal-sama-api/.env` pelas chaves `SAMA_BOOTSTRAP_ADMIN_USERNAME`/`SAMA_BOOTSTRAP_ADMIN_PASSWORD`, sem imprimir valores. As chaves `PORTAL_AUTH_*` retornaram HTTP 401 e `PORTAL_E2E_*` continha referencia literal, entao nao foram usadas.
- `npm.cmd run smoke:phase9 -- --json --soft --evidence-dir .ai-tests/phase9-smoke` autenticado read-only retornou `ok=true`, `failed=0`, `blocked=0`, usuario DEV com 99 permissoes, health/banco/storage/clientes/contratos/Acessorias/documentos OK e evidencia `.ai-tests/phase9-smoke/phase9-smoke-20260625T193014387Z.json`.
- `PORTAL_PHASE9_APPLY_ACTIONS=1 npm.cmd run smoke:phase9 -- --json --soft --evidence-dir .ai-tests/phase9-smoke` criou contrato `INTERNAL`, contrato `ZAPSIGN` sandbox e confirmou upload SVG invalido rejeitado. A evidencia `.ai-tests/phase9-smoke/phase9-smoke-20260625T193028929Z.json` ficou `failed=1`, `blocked=1`: Acessorias excedeu timeout local de 120s e upload PDF valido retornou `DOCUMENT_SCAN_UNAVAILABLE` em modo estrito.
- Consultas autenticadas a `GET /integrations/acessorias/sync-runs` confirmaram que os jobs disparados pelo smoke concluiram com `SUCCESS`: catalogo `fetched=494`, `updated=494`, `failed=0`, seguido de incremental de entregas `SUCCESS`. A repeticao com `--timeout 360000 --skip-zapsign` gerou `.ai-tests/phase9-smoke/phase9-smoke-20260625T193547773Z.json`; o endpoint sincrono ainda retornou `504` apos cerca de 240s, mas o respectivo sync run tambem fechou `SUCCESS`.
- Continuidade em 2026-06-25: implementado polling do Acessorias no runner Fase 9 com `PORTAL_PHASE9_ACESSORIAS_POLL_TIMEOUT_MS`/`--acessorias-poll-timeout`, mantendo saida sanitizada; checks focados `node --check scripts/portal-phase9-smoke.mjs`, `npm.cmd test -- --runInBand` no web e spec nova `acessorias-operational-sync.service.spec.ts` passaram antes da bateria final.
- Historico superado da Fase 9: o upload PDF valido chegou a retornar `503 DOCUMENT_SCAN_UNAVAILABLE`, `reason=scanner_required`; em 2026-06-29, apos `freshclam`, o upload PDF valido passou com HTTP 201 e a homologacao real final confirmou scanner operacional.
- `git diff --check` - OK na rodada de 2026-06-25.
- Servidor local Vite foi iniciado anteriormente em `http://127.0.0.1:5173`.

### Deploy real Fase 9

- `npm.cmd run homologation:real -- --json --soft --skip-auth --skip-permissions --skip-e2e --evidence-dir .ai-tests/homologation-real-phase9` contra `https://portal.samacontabil.com.br` retornou `ok=true`.
- Checks publicos aprovados: `public-web` HTTP 200, `api-health` HTTP 200 com `database=up` e `storage=up`, `cors-preflight` HTTP 204, `auth-csrf` HTTP 200 com token/cookie.
- Checagem Playwright anonima em `https://portal.samacontabil.com.br/dev/fase-9-smoke` redirecionou corretamente para `https://portal.samacontabil.com.br/login` e exibiu `Entrar no portal`.
- `npm.cmd run homologation:real -- --json --soft --skip-auth --skip-permissions --skip-e2e --skip-phase9 --evidence-dir .ai-tests/homologation-real-phase9` retornou `ok=true` para rodada publica sem credenciais.
- `npm.cmd run homologation:real -- --json --soft --skip-auth --skip-permissions --skip-e2e --evidence-dir .ai-tests/homologation-real-phase9` retornou `ok=false` somente porque `smoke:phase9` ficou `blocked` por ausencia local de `PORTAL_AUTH_USERNAME` e `PORTAL_AUTH_PASSWORD`; `smoke:public` passou.
- Apos o deploy informado em 2026-06-22, `npm.cmd run smoke:phase9 -- --json --soft` confirmou `/dev/fase-9-smoke` em HTTP 200 servindo o shell HTML em producao; o unico bloqueio foi ausencia local de `PORTAL_AUTH_USERNAME` e `PORTAL_AUTH_PASSWORD`.
- `npm.cmd run smoke:phase9 -- --json --soft --evidence-dir .ai-tests/phase9-smoke` criou evidencia sanitizada em `.ai-tests/phase9-smoke/phase9-smoke-20260622T200528083Z.json`; rota publica passou e o unico bloqueio foi ausencia local de credenciais.
- Rodadas autenticadas com `PORTAL_PHASE9_APPLY_ACTIONS=1` em 2026-06-22 criaram evidencias `.ai-tests/phase9-smoke/phase9-smoke-20260622T205252516Z.json` e `.ai-tests/phase9-smoke/phase9-smoke-20260622T205313213Z.json`.
- Nessas rodadas autenticadas passaram: rota `/dev/fase-9-smoke`, CSRF/login, `/auth/me`, `GET /health`, listagem de clientes/contratos/documentos, resumo Acessorias, criacao de contrato `INTERNAL`, criacao de contrato `ZAPSIGN` sandbox, rejeicao de upload SVG invalido e logout.
- Historico superado: `acessorias-controlled-sync` retornou HTTP 409 e `upload-valid-document` retornou HTTP 503 em rodadas anteriores; Acessorias depois fechou `SUCCESS` por polling e o upload PDF valido passou com scanner ativo.
- Rodadas paralelas de `homologation:real` apos a correcao de evidencia passaram a criar arquivos distintos: publica `ok=true` em `homologation-real-20260622T195954391Z.json` e integrada `ok=false` apenas por `smoke:phase9` bloqueado em `homologation-real-20260622T195954392Z.json`.
- Historico superado: runner completo sem skips dependia de variaveis locais reais; a rodada final com credenciais reais carregadas passou em `homologation:real --skip-permissions`. A matriz de permissoes segue pulada por opcao nesta conclusao, sem bloquear a Fase 9 porque o criterio final aceito usou `--skip-permissions`.
- Evidencias locais ignoradas pelo git incluem `.ai-tests/homologation-real-phase9/homologation-real-20260622T172552Z.json`, `.ai-tests/homologation-real-phase9/homologation-real-20260622T172645Z.json`, `.ai-tests/homologation-real-phase9/homologation-real-20260622T195241Z.json`, `.ai-tests/homologation-real-phase9/homologation-real-20260622T195316Z.json`, `.ai-tests/homologation-real-phase9/homologation-real-20260622T195954391Z.json` e `.ai-tests/homologation-real-phase9/homologation-real-20260622T195954392Z.json`.

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
- `6961179 fix: avoid homologation evidence filename collisions`
- `f8ebcfa test: write phase 9 smoke evidence files`
- `605453f test: include sanitized phase 9 failure details`

### `portal-sama-docs`

Ultimo commit registrado antes desta atualizacao de Fase 9:

- `fed6452 docs: record phase 9 acessorias polling`

## Proximo chat deve fazer

1. Ler primeiro os documentos da raiz listados em `Precedencia obrigatoria`.
2. Ler `docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md` apenas como evidencia subordinada.
3. Confirmar que a Fase 8 esta `CONCLUIDA`.
4. Confirmar que a Fase 9 esta `CONCLUIDA` pela evidencia `portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260629T134038169Z.json`.
5. Confirmar que a Fase 10 esta `CONCLUIDA` conforme `22-CONTINUIDADE-FASE-10-FRONTEND.md`.
6. Iniciar Fase 11 - painel do cliente final reaproveitando o legado.
7. Comecar pela Etapa 11.1 - Criar `ClientDashboardHeader`.
8. Ao alterar codigo, reexecutar lint/build/test correspondentes e `git diff --check`.

## Cuidados

- Nao ler nem imprimir `.env` real ou secrets.
- Nao reverter alteracoes locais.
- Nao usar `prisma db push` em ambiente compartilhado/producao.
- Nao mascarar falha operacional com `--skip-*` para liberar fase.
- Nao considerar backup valido sem restore drill.
- Fase 8 ja comprovou backup/restore drill real; preservar a evidencia fora de `/tmp`.
