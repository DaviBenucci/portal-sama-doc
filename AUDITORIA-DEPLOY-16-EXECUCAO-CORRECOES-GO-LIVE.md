# AUDITORIA DEPLOY 16 - Execucao das correcoes Go-Live

Data: 10/06/2026

Escopo: registro incremental da execucao das fases planejadas em
`AUDITORIA-DEPLOY-15-PREPARACAO-CORRECOES-GO-LIVE.md`.

## Ferramentas locais usadas nesta rodada

- Docker: usado na Fase 8 com a imagem local `portal-sama-api:clamav-runtime` para validar
  ClamAV strict/EICAR em container efemero, sem instalar ClamAV no Windows host.
- Instalacoes/execucoes auxiliares: foi usado `npx.cmd prettier@3.8.4` de forma transitoria
  para formatacao. O primeiro uso no frontend baixou o pacote via `npx`; nao houve alteracao
  intencional de `package.json` ou `package-lock.json`.
- Fase 7: foi usado `npm.cmd audit fix` no `portal-sama-api`, atualizando a dependencia
  transitiva local `qs` de `6.15.1` para `6.15.2` e o `package-lock.json`.
- Fase 8: foi executado `npm run ops:clamav:update` dentro do container para atualizar
  assinaturas antes do readiness strict.
- Fase 9: foi usado Docker com o MySQL local `portal-sama-audit-mysql` e a imagem `mysql:8.4`.
  Dentro do container efemero foi instalado `nodejs` e `which` via `microdnf` para rodar os
  scripts Node com os clientes MySQL 8.4 nativos. Nada foi instalado no Windows host.
- Fase 10: foi usado o MySQL Docker local `portal-sama-audit-mysql`, com banco isolado
  `portal_sama_webpush_phase10`, para validar Web Push sem tocar banco real. Tambem foi usado
  Chromium via Playwright em modo headed/perfil persistente temporario, porque headless e
  contexto incognito bloqueiam a Push API/Notification. Nenhuma instalacao permanente foi feita
  no Windows host.
- Fase 11: foi usado o mesmo MySQL Docker local `portal-sama-audit-mysql`, com banco isolado
  `portal_sama_profiles_phase11`, para validar a matriz de perfis sem tocar banco real. A API
  local rodou na porta `3120`, o web local na porta `4176`, e os schedulers de Acessorias ficaram
  explicitamente desligados durante a bateria. Nenhuma instalacao permanente foi feita no Windows
  host.

## Fase 0 - Preparacao operacional

Status: concluida nesta rodada.

Evidencias preservadas nesta rodada:

- `evidencias/auditoria-deploy-15/validation-api-results.json`
- `evidencias/auditoria-deploy-15/validation-web-results.json`
- `evidencias/auditoria-deploy-15/acessorias-api-results-clean.json`

Branch de trabalho criada nos tres repositorios:

- `portal-sama-docs`: `fix/deploy-readiness-go-live`
- `portal-sama-api`: `fix/deploy-readiness-go-live`
- `portal-sama-web`: `fix/deploy-readiness-go-live`

## Decisao operacional para Fase 3

Enquanto nao houver decisao formal de produto dando leitura ampla a outros perfis, a politica
de `clients.read` deve seguir o principio de menor privilegio:

| Perfil | Decisao aplicada nesta correcao |
|---|---|
| `ADMIN` / `DEV` | leitura ampla permitida |
| `MANAGER` | limitado a carteira/departamento/vinculo gerenciado |
| `AUDITOR` | sem leitura ampla automatica de clientes |
| `CLIENT` | somente o proprio cliente/vinculo |
| `DEPARTMENT` | somente clientes vinculados ao proprio departamento ou responsabilidade |
| `ACCOUNTING` | somente clientes vinculados ao escopo contabil/departamental |
| `LEGALIZATION` | somente clientes vinculados ao escopo de legalizacao/departamental |
| `TI` | sem leitura ampla automatica de clientes |

Observacao: se o negocio decidir que `MANAGER`, `AUDITOR` ou `TI` precisa de leitura global,
a decisao deve virar permissao/regra explicita e testada, nao um efeito colateral de
`clients.read`.

## Fase 3 - Corrigir clients.read e escopo de clientes

Status: implementada com validacao focada concluida. Gate E2E geral da API ainda bloqueado
pela falha conhecida da Fase 5 em Web Push/CSRF.

Arquivos alterados:

- `portal-sama-api/src/modules/clients/clients.service.ts`
- `portal-sama-api/src/modules/clients/clients.service.spec.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`

Mudancas aplicadas:

- Removido o atalho que permitia leitura ampla apenas por `clients.read`.
- `ADMIN` e `DEV` continuam com leitura ampla.
- `MANAGER`, `DEPARTMENT`, `ACCOUNTING`, `LEGALIZATION`, `TI` e perfis customizados com
  `clients.read` passam a depender de:
  - responsabilidade direta (`responsibleUserId`);
  - gestao direta (`managerUserId`);
  - departamento ativo permitido ao usuario.
- `CLIENT` passa a receber `clients.read` no RBAC padrao, mas o servico limita a leitura ao
  proprio cliente por `id`, `username` ou CNPJ derivado do `username`.
- `list()`, `getById()` e `getDashboard()` passam pela mesma politica de leitura.

Testes adicionados/ajustados:

- lista de clientes de usuario nao privilegiado recebe filtro por `ClientDepartmentAssignment`
  ativo;
- usuario `CLIENT` acessa somente o proprio cliente por CNPJ;
- usuario com `clients.read` fora do escopo recebe `ForbiddenException`;
- `MANAGER` sem vinculo nao tem leitura ampla;
- responsavel direto consegue ler cliente atribuido;
- RBAC padrao continua consistente apos incluir `clients.read` em `CLIENT`.

Validacoes executadas:

| Comando | Resultado |
|---|---|
| `npm.cmd test -- --runInBand src/modules/clients/clients.service.spec.ts src/modules/rbac/default-rbac.spec.ts` | OK: 2 suites, 13 testes |
| `npm.cmd run lint` | OK |
| `npm.cmd run build` | OK |
| `git diff --check` | OK |
| `npm.cmd run test:e2e` | Falhou: 134 passaram, 2 falharam em Web Push/CSRF por `403` esperado `200` |

Leitura:

- A correcao de escopo de clientes esta coberta por testes focados e nao apresentou falha em
  lint/build.
- O gate E2E API ainda permanece vermelho pelo bloqueio `QA-API-01`, que sera tratado na Fase 5.

## Fase 4 - Corrigir escalada de perfil em Acessorias Home

Status: implementada com validacao focada concluida.

Arquivos alterados:

- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.controller.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.service.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.service.spec.ts`
- `portal-sama-web/src/services/acessorias.service.ts`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `portal-sama-web/src/pages/dev/DevAdminPage.tsx`

Mudancas aplicadas:

- O backend passa a derivar o perfil efetivo da Home pelo usuario autenticado.
- `profile` recebido por query continua aceito apenas por compatibilidade, mas nao amplia
  autorizacao.
- Perfil efetivo maximo:
  - `ADMIN` / `DEV`: `admin`, com visao global.
  - `MANAGER`/gestao equivalente: `manager`, com escopo por departamento permitido ou
    responsabilidade direta.
  - demais usuarios: `collaborator`, com escopo por responsabilidade direta e, quando houver
    departamento, itens sem responsavel apenas dentro do departamento.
- `profile=admin` para usuarios nao privilegiados sofre downgrade automatico para o perfil
  permitido.
- Usuarios sem departamento ou responsabilidade direta nao caem mais em visao global por efeito
  colateral.
- Frontend deixou de enviar `profile` para `/integrations/acessorias/home-summary`.
- A tela DEV tambem deixou de chamar `getAcessoriasHomeSummary('admin')`; o backend decide a visao
  com base no token.

Testes adicionados/ajustados:

- usuario departamental chamando `profile=admin` recebe `profile=collaborator` e nao recebe outro
  departamento;
- gestor sem departamento chamando `profile=admin` recebe `profile=manager`, sem visao global;
- `ADMIN` continua recebendo `profile=admin` e dados globais.

Validacoes executadas:

| Comando | Resultado |
|---|---|
| `npm.cmd test -- --runInBand src/modules/integrations/acessorias/acessorias-home.service.spec.ts` | OK: 1 suite, 6 testes |
| `npm.cmd test -- --runInBand src/modules/integrations/acessorias/acessorias-registrations.service.spec.ts src/modules/integrations/acessorias/acessorias-deliveries.service.spec.ts` | OK: 2 suites, 16 testes |
| `npm.cmd test -- --runInBand src/modules/clients/clients.service.spec.ts src/modules/rbac/default-rbac.spec.ts src/modules/integrations/acessorias/acessorias-home.service.spec.ts src/modules/integrations/acessorias/acessorias-registrations.service.spec.ts src/modules/integrations/acessorias/acessorias-deliveries.service.spec.ts` | OK: 5 suites, 35 testes |
| `npm.cmd run lint` em `portal-sama-api` | OK |
| `npm.cmd run build` em `portal-sama-api` | OK |
| `npm.cmd run lint` em `portal-sama-web` | OK |
| `npm.cmd run build` em `portal-sama-web` | OK |
| `npm.cmd test -- --runInBand` em `portal-sama-web` | OK: 9 contratos |
| `git diff --check` em `portal-sama-api` e `portal-sama-web` | OK |

Leitura:

- O achado `AUTHZ-03` esta corrigido no codigo local: alterar `profile` na URL nao concede visao
  maior.
- Ainda falta executar uma bateria Acessorias real/local completa apos as fases seguintes, pois a
  release ainda depende de E2E API/Web, artefatos limpos, audit `qs` e provas operacionais.

## Fase 5 - Corrigir E2E API Web Push/CSRF

Status: implementada e validada. Gate E2E API esta verde no codigo local.

Arquivo alterado:

- `portal-sama-api/test/app.e2e-spec.ts`

Mudanca aplicada:

- O setup E2E agora define explicitamente variaveis de cookie/CSRF antes de importar o
  `AppModule`:
  - `CSRF_SECRET=test_csrf_secret_with_at_least_32_chars`
  - `COOKIE_DOMAIN=''`
  - `COOKIE_SECURE=false`
  - `COOKIE_SAME_SITE=lax`
- Isso impede que a suite E2E herde `COOKIE_DOMAIN`, `COOKIE_SECURE`, `COOKIE_SAME_SITE` ou
  segredo CSRF do `.env` local/real.
- O comportamento de producao nao foi alterado; a mudanca fica restrita ao ambiente de teste.

Validacoes executadas:

| Comando | Resultado |
|---|---|
| `npm.cmd run test:e2e` | OK: 1 suite, 136 testes |
| `npm.cmd run lint` | OK |
| `npm.cmd run build` | OK |
| `npm.cmd test -- --runInBand` | OK: 44 suites, 272 testes |
| `git diff --check` em `portal-sama-api` | OK |

Leitura:

- Os dois testes que falhavam em Web Push/CSRF agora passam.
- Os testes negativos de CSRF continuam passando com `403` quando nao ha token/cookie valido.
- O bloqueio `QA-API-01` esta corrigido no codigo local.

## Fase 6 - Corrigir E2E Web Home

Status: implementada e validada. Gate E2E Web esta verde no codigo local.

Arquivo alterado:

- `portal-sama-web/tests/e2e/smoke.spec.ts`

Mudancas aplicadas:

- O mock global de notificacoes do Playwright agora cobre tambem
  `/api-v2/notifications/stream?take=20` como `text/event-stream`.
- O mock de sessao autenticada agora cobre `/api-v2/me/security`, evitando chamadas reais/proxy
  durante os testes de paginas autenticadas.
- O E2E da Home admin foi alinhado aos textos atuais da UI:
  - `Diagnostico Acessorias`;
  - atalhos visiveis no perfil mockado: `Vencimentos` e `Solicitacoes`.
- O E2E da Home colaborador foi alinhado ao bloco atual `Baixas sincronizadas`.
- Nao houve alteracao de comportamento de producao; a correcao ficou restrita aos testes E2E.

Validacoes executadas:

| Comando | Resultado |
|---|---|
| `npm.cmd run test:e2e` em `portal-sama-web` | OK: 13 passaram, 1 skip |
| `npm.cmd run lint` em `portal-sama-web` | OK |
| `npm.cmd run build` em `portal-sama-web` | OK |
| `npm.cmd test -- --runInBand` em `portal-sama-web` | OK: 9 contratos |
| `git diff --check` em `portal-sama-web` | OK |

Leitura:

- As duas falhas conhecidas da Home Web foram corrigidas sem reabrir a auditoria desde o inicio.
- O ruido de proxy para `notifications/stream` e `me/security` deixou de aparecer na execucao E2E.
- Docker nao foi usado nesta fase; nenhuma instalacao local nova foi necessaria.

## Fase 7 - Corrigir vulnerabilidade `qs`

Status: implementada e validada. `npm audit` da API esta sem vulnerabilidades moderadas ou
superiores no codigo local.

Arquivo alterado:

- `portal-sama-api/package-lock.json`

Mudanca aplicada:

- Executado `npm.cmd audit fix` no `portal-sama-api`.
- A resolucao transitiva de `qs` foi atualizada de `6.15.1` para `6.15.2`.
- `package.json` nao foi alterado.
- O `package-lock.json` tambem recebeu o metadado `hasInstallScript: true` na raiz, gerado pelo
  npm por existir `postinstall`.

Validacoes executadas:

| Comando | Resultado |
|---|---|
| `npm.cmd audit --audit-level=moderate` em `portal-sama-api` | OK: 0 vulnerabilidades |
| `npm.cmd run lint` em `portal-sama-api` | OK |
| `npm.cmd run build` em `portal-sama-api` | OK |
| `npm.cmd test -- --runInBand` em `portal-sama-api` | OK: 44 suites, 272 testes |
| `npm.cmd run test:e2e` em `portal-sama-api` | OK: 1 suite, 136 testes |
| `git diff --check` em `portal-sama-api` | OK |

Leitura:

- O achado `DEP-01`/`SEC-07` de `qs` foi corrigido no lockfile local.
- A correcao nao exigiu mudanca em codigo-fonte nem atualizacao direta de dependencias.
- Docker nao foi usado nesta fase.

## Fase 8 - Validar ClamAV strict

Status: validada localmente em container efemero. Ambiente de homologacao/producao ainda deve
replicar a configuracao `SAMA_UPLOAD_SCAN_MODE=strict` com ClamAV instalado no runtime correto.

Arquivos alterados/adicionados:

- `portal-sama-api/src/modules/documents/document-upload-scanner.service.spec.ts`
- `portal-sama-docs/evidencias/auditoria-deploy-15/clamav-strict-readiness.json`

Mudancas aplicadas:

- Adicionados testes focados do `DocumentUploadScannerService` em modo `strict`:
  - upload limpo e aceito quando o scanner externo retorna sucesso;
  - upload com assinatura tipo EICAR rejeitado quando o scanner externo retorna deteccao;
  - arquivo rejeitado permanece em quarentena para analise.
- Gerada evidencia operacional com Docker usando a imagem local `portal-sama-api:clamav-runtime`.
- Dentro do container, `freshclam` foi executado via `npm run ops:clamav:update` antes do
  readiness.
- O readiness foi executado com `SAMA_UPLOAD_SCAN_MODE=strict`, `--skip-env` e `--skip-database`
  para isolar storage + ClamAV/EICAR sem depender de segredos ou banco real.

Evidencia preservada:

- `evidencias/auditoria-deploy-15/clamav-strict-readiness.json`
  - `ok: true`
  - `failed: 0`
  - `clamav-eicar: passed`
  - scanner: `/usr/bin/clamscan`
  - `backup-rollback: warning`, esperado porque backup/restore e tratado na Fase 9.

Validacoes executadas:

| Comando | Resultado |
|---|---|
| `docker run --rm portal-sama-api:clamav-runtime sh -lc "node --version && clamscan --version"` | OK: Node 22 e ClamAV 1.4.4 |
| `docker run --rm portal-sama-api:clamav-runtime sh -lc "npm run ops:clamav:update"` | OK: `daily`, `main` e `bytecode` atualizados; aviso esperado de `clamd` nao rodando |
| readiness strict em container com `SAMA_UPLOAD_SCAN_MODE=strict --skip-env --skip-database` | OK: `clamav-eicar` passou; evidencia JSON salva |
| `npm.cmd test -- --runInBand src/modules/documents/document-upload-scanner.service.spec.ts` | OK: 1 suite, 5 testes |
| `npm.cmd run lint` em `portal-sama-api` | OK |
| `npm.cmd run build` em `portal-sama-api` | OK |
| `npm.cmd test -- --runInBand` em `portal-sama-api` | OK: 44 suites, 274 testes |
| `npm.cmd run test:e2e` em `portal-sama-api` | OK: 1 suite, 136 testes |
| `npm.cmd audit --audit-level=moderate` em `portal-sama-api` | OK: 0 vulnerabilidades |
| `git diff --check` em `portal-sama-api` | OK |

Leitura:

- A ausencia de ClamAV no host Windows foi contornada por validacao em container, sem instalacao
  permanente na maquina.
- O modo strict esta coberto por teste automatizado e por evidencia operacional EICAR.
- A warning restante do readiness e sobre backup/rollback, que permanece como escopo da Fase 9.

## Fase 9 - Provar backup, verify e restore drill

Status: validada localmente contra alvo isolado em Docker. Nenhum banco de producao ou `.env`
real foi usado.

Ambiente isolado:

- Container MySQL local: `portal-sama-audit-mysql`.
- Banco origem do drill: `portal_sama_backup_drill_source`.
- Banco alvo do restore: `portal_sama_backup_drill_restore`.
- Tabela sentinela: `backup_probe`, com `sentinel_label=fase-9-drill-source`.
- Storage origem: fixture temporaria criada dentro do container.
- Storage alvo: `/tmp/portal-sama-storage-restore`, dentro do container efemero.

Evidencias preservadas:

- `evidencias/auditoria-deploy-15/backup-drill/backup-create.json`
- `evidencias/auditoria-deploy-15/backup-drill/backup-verify.json`
- `evidencias/auditoria-deploy-15/backup-drill/restore-drill.json`
- `evidencias/auditoria-deploy-15/backup-drill/restore-target-check.json`
- `evidencias/auditoria-deploy-15/backup-drill/backup-artifacts.sha256`
- `evidencias/auditoria-deploy-15/backup-drill/backups/portal-sama-20260610T184955Z-e76aef/`

Resultados:

- Backup criado com `ok: true`, `failed: 0`, `warnings: 0`.
- Artefatos criados:
  - `database.sql.gz`;
  - `metadata.json`;
  - `storage-manifest.json`;
  - `storage.tar.gz`.
- Verify executado com `ok: true`, `failed: 0`, `warnings: 1`.
  - A warning e o lembrete padrao de que restore drill real ainda precisa ser feito; ela foi
    sanada na etapa seguinte desta mesma fase.
- Restore drill executado com `dryRun: false`, `ok: true`, `failed: 0`, `warnings: 2`.
  - `database-restore`: passou.
  - `storage-restore`: passou.
  - Warnings esperadas:
    - verify tinha aviso antes do restore real;
    - banco alvo usa o mesmo host Docker da origem, mas banco distinto e isolado.
- Checagem pos-restore confirmou `rows_count: 1` e
  `sentinel_label: fase-9-drill-source` no banco `portal_sama_backup_drill_restore`.
- Checksums SHA-256 dos quatro artefatos principais foram salvos em
  `backup-artifacts.sha256`.

Observacoes operacionais:

- A imagem `portal-sama-api:backup-runtime` possui cliente MariaDB, que nao conseguiu autenticar
  contra MySQL 8.4 com `caching_sha2_password`. Para evitar alterar o servidor, a validacao final
  usou a imagem oficial `mysql:8.4`, com `nodejs` e `which` instalados apenas no container
  efemero.
- Busca nas evidencias nao encontrou senha local gravada.
- Para homologacao/producao, repetir contra destino externo/seguro de backup e alvo de restore
  isolado equivalente ao ambiente real.

## Fase 10 - Homologar Web Push real

Status: validada localmente com navegador real e provedor Web Push real. Ainda falta repetir em
ambiente HTTPS publico/confiavel com VAPID oficial de homologacao/producao e clique nativo
assistido na notificacao do sistema operacional.

Arquivos alterados:

- `portal-sama-web/src/services/notifications.service.ts`
- `portal-sama-web/scripts/web-contract-tests.mjs`

Problema encontrado e corrigido:

- O Chromium persistente conseguiu conceder permissao de notificacao, mas a inscricao falhava com
  `Subscription failed - no active Service Worker`.
- A funcao de registro do service worker agora retorna `navigator.serviceWorker.ready` quando o
  registro ainda nao esta ativo. Assim o `pushManager.subscribe()` so roda com service worker
  ativo.
- O contrato web passou a exigir `navigator.serviceWorker.ready` no servico de notificacoes.

Ambiente isolado usado:

- MySQL Docker local: `portal-sama-audit-mysql`.
- Banco da Fase 10: `portal_sama_webpush_phase10`.
- API local: `http://127.0.0.1:3110/api-v2`.
- Web local: `http://localhost:4174`, com proxy Vite para a API.
- Usuario local bootstrap: `phase10.dev`.
- VAPID temporario gerado em runtime para a validacao; chave privada nao foi persistida na
  documentacao.
- Cookies ajustados somente no processo local: `COOKIE_DOMAIN=localhost`, `COOKIE_SECURE=false`,
  `COOKIE_SAME_SITE=lax`.

Evidencias preservadas:

- `evidencias/auditoria-deploy-15/webpush-phase10/browser-persistent-final-validation.json`
- `evidencias/auditoria-deploy-15/webpush-phase10/db-summary.json`
- `evidencias/auditoria-deploy-15/webpush-phase10/notifications-webpush-final.png`

Resultado da validacao final:

- Login do usuario `phase10.dev`: OK.
- Pagina `/notificacoes`: OK.
- Browser com Push API/Service Worker/Notification suportados e `Notification.permission=granted`.
- Subscribe via UI: OK, API `POST /notifications/web-push/subscribe` retornou `200`.
- Assinatura real criada no navegador:
  - service worker registrado em `http://localhost:4174/`;
  - endpoint com origem `https://jmt17.google.com`;
  - chaves publicas `p256dh` e `auth` presentes.
- Preferencia Web Push alterada pela UI: API `PATCH /notifications/preferences` retornou `200`.
- Teste Web Push via UI: `Teste Web Push: 1/1 enviados.`
- O navegador registrou uma notificacao visivel via service worker: `Atualizacao do Portal Sama`.
- Unsubscribe via UI: OK, API `POST /notifications/web-push/unsubscribe` retornou `200` e
  `updated: 1`.
- Estado final no navegador: `subscribed=false`.
- Resumo do banco isolado:
  - `delivery_attempts`: `SENT=1`;
  - `subscriptions.active=0`;
  - `subscriptions.with_success=1`;
  - `fail_count_sum=0`.

Tentativas e limitacoes relevantes:

- `COOKIE_DOMAIN=portal.samacontabil.com.br` herdado localmente quebrou CSRF em `127.0.0.1`; a
  validacao final usou `localhost` e proxy Vite para manter origem/cookies coerentes.
- Chromium headless retornou `Notification.permission=denied`.
- Contexto nao persistente/incognito do Playwright bloqueou Push API.
- A validacao final precisou de Chromium headed com perfil persistente temporario, removido ao
  final.
- `localhost` HTTP e tratado pelo Chromium como secure context para Service Worker/Push API, mas
  isso nao substitui homologacao em HTTPS publico/confiavel.
- O clique real na notificacao nativa do sistema operacional ainda precisa de confirmacao
  assistida em ambiente real.

Validacoes executadas:

| Comando | Resultado |
|---|---|
| Validacao Playwright headed/persistente contra API local real | OK: subscribe, preferencia, push `1/1`, unsubscribe |
| Consulta MySQL `portal_sama_webpush_phase10` | OK: `SENT=1`, `active=0`, `fail_count_sum=0` |
| `npm.cmd test` em `portal-sama-web` | OK: 9 contratos |
| `npm.cmd run lint` em `portal-sama-web` | OK |
| `npm.cmd run build` em `portal-sama-web` | OK |
| `npm.cmd test -- --runInBand --runTestsByPath ...notifications.service.spec.ts ...notifications-push.service.spec.ts` em `portal-sama-api` | OK: 2 suites, 28 testes |
| `npm.cmd run build` em `portal-sama-api` | OK |
| `npm.cmd run lint` em `portal-sama-api` | OK |

Leitura:

- O bug local que impedia `subscribe()` antes de o service worker estar ativo foi corrigido.
- A cadeia browser real -> endpoint Web Push real -> API -> persistencia -> dispatch -> service
  worker -> unsubscribe foi validada em ambiente controlado.
- Para fechar o aceite estrito de go-live da Fase 10, repetir a mesma matriz em HTTPS real com
  VAPID oficial e validar o click da notificacao nativa.

## Fase 11 - Homologar perfis reais

Status: validada localmente em matriz controlada com dados sinteticos. Ainda falta homologacao
assistida em ambiente real com usuarios/dados reais, mas a bateria local ja cobriu fluxos felizes
e negativos dos perfis minimos.

Arquivos alterados:

- `portal-sama-api/src/modules/documents/documents.service.ts`
- `portal-sama-api/src/modules/documents/documents.service.spec.ts`

Problema encontrado e corrigido:

- A primeira execucao da matriz local confirmou que `clients.read` ja estava escopado para
  `MANAGER`, mas documentos ainda tratavam `MANAGER` como perfil privilegiado amplo.
- Resultado observado antes da correcao:
  - listagem de documentos de `MANAGER` via todos os documentos da base sintetica, incluindo
    clientes fora do departamento/carteira;
  - detalhe de documento fora do escopo retornava `200`, quando o esperado era `403`.
- A correcao removeu `MANAGER` de `PRIVILEGED_DOCUMENT_ROLES`, preservando leitura ampla apenas
  para `ADMIN` e `DEV`.
- `MANAGER` continua permitido nos fluxos em que a regra de negocio ja era explicita:
  gerenciamento de requisitos/documentos publicos e tokens publicos de documentos.
- Foram adicionados testes unitarios cobrindo:
  - listagem de documentos de `MANAGER` limitada ao escopo departamental/carteira;
  - detalhe de documento fora do escopo de `MANAGER` retornando `ForbiddenException`.

Ambiente isolado usado:

- MySQL Docker local: `portal-sama-audit-mysql`.
- Banco da Fase 11: `portal_sama_profiles_phase11`.
- Migrations aplicadas no banco isolado: 29.
- Seed executado no banco isolado.
- API local: `http://127.0.0.1:3120/api-v2`.
- Web local: `http://127.0.0.1:4176`, com proxy Vite para a API.
- Schedulers de Acessorias desligados explicitamente durante a validacao local:
  - `ACESSORIAS_SCHEDULER_ENABLED=false`;
  - `ACESSORIAS_INCREMENTAL_SYNC_ENABLED=false`;
  - `ACESSORIAS_COMPANY_CATALOG_SYNC_ENABLED=false`;
  - `ACESSORIAS_RECONCILIATION_ENABLED=false`;
  - `ACESSORIAS_BACKFILL_SYNC_ENABLED=false`.
- Dados sinteticos criados para perfis `DEV`, `MANAGER`, `DEPARTMENT`, `CLIENT`, `ACCOUNTING`,
  `LEGALIZATION`, `TI` e `AUDITOR`.
- Arquivos PDF sinteticos usados durante a execucao local foram removidos ao final; as evidencias
  preservadas sao JSONs e screenshots.

Evidencias preservadas:

- `evidencias/auditoria-deploy-15/profiles-phase11/api-profile-matrix.json`
- `evidencias/auditoria-deploy-15/profiles-phase11/browser-profile-smoke.json`
- `evidencias/auditoria-deploy-15/profiles-phase11/runtime-api-start.json`
- `evidencias/auditoria-deploy-15/profiles-phase11/runtime-web-start.json`
- `evidencias/auditoria-deploy-15/profiles-phase11/browser-*.png`

Resultado da matriz API:

- `102/102` checks passaram apos a correcao.
- Coberturas principais:
  - acesso anonimo a rota protegida retorna `401`;
  - login/logout validado para os perfis sinteticos;
  - role da sessao conferida para cada perfil;
  - clientes respeitam escopo por perfil;
  - documentos respeitam escopo por perfil;
  - detalhes fora do escopo retornam `403`;
  - download de documento proprio para `CLIENT` retorna `200`;
  - upload autorizado dentro do escopo retorna `201`;
  - upload fora do escopo retorna `403`;
  - Acessorias Home com `profile=admin` sofre downgrade para gestor/colaborador quando aplicavel;
  - notificacoes respondem para perfis esperados;
  - auditoria responde `200` para `DEV`/`AUDITOR` e `403` para perfis sem permissao.

Resultado do smoke em browser real:

- `4/4` perfis principais passaram: `DEV`, `MANAGER`, `DEPARTMENT`/colaborador e `CLIENT`.
- Fluxos conferidos:
  - login com redirecionamento para Home;
  - ausencia de tokens/sessoes sensiveis em `localStorage`/`sessionStorage`;
  - cookie de refresh marcado como `httpOnly`;
  - navegacao por rotas de Home, Clientes/Departamento, Documentos e Auditoria;
  - logout com retorno para Login.
- Checagem visual/textual nao encontrou nomes de clientes fora do escopo na UI para gestor,
  colaborador e cliente.

Validacoes executadas:

| Comando | Resultado |
|---|---|
| Matriz API local por perfil contra API real local | OK: 102 checks, 0 falhas |
| Smoke Playwright por perfil contra web/API reais locais | OK: 4 perfis, 0 falhas |
| `npm.cmd test -- --runInBand src/modules/documents/documents.service.spec.ts` | OK: 1 suite, 33 testes |
| `npm.cmd test -- --runInBand --runTestsByPath ...documents.service.spec.ts ...clients.service.spec.ts ...rbac/default-rbac.spec.ts` | OK: 3 suites, 46 testes |
| `npm.cmd test -- --runInBand` em `portal-sama-api` | OK: 44 suites, 276 testes |
| `npm.cmd run test:e2e` em `portal-sama-api` | OK: 1 suite, 136 testes |
| `npm.cmd run lint` em `portal-sama-api` | OK |
| `npm.cmd run build` em `portal-sama-api` | OK |
| `npm.cmd audit --audit-level=moderate` em `portal-sama-api` | OK: 0 vulnerabilidades |
| `npm.cmd test` em `portal-sama-web` | OK: 9 contratos |
| `npm.cmd run test:e2e` em `portal-sama-web` | OK: 13 passaram, 1 skip |
| `npm.cmd run lint` em `portal-sama-web` | OK |
| `npm.cmd run build` em `portal-sama-web` | OK |
| `npm.cmd audit --audit-level=moderate` em `portal-sama-web` | OK: 0 vulnerabilidades |

Limitacoes e leitura:

- A Fase 11 esta validada localmente com dados sinteticos e runtime controlado; isso reduz risco,
  mas nao substitui homologacao assistida com usuarios reais do ambiente alvo.
- O achado de autorizacao em documentos era critico para dados sensiveis e foi corrigido antes de
  concluir a fase.
- A pendencia externa da Fase 10 permanece: HTTPS publico/confiavel, VAPID oficial e clique nativo
  assistido.
- A Fase 1 segue critica para go-live: ZIP limpo e rotacao externa de segredos potencialmente
  expostos ainda precisam ser tratados.
