# Continuidade Fase 9 - EasyPanel

Atualizado em: 2026-06-29
Ambiente alvo: `https://portal.samacontabil.com.br`  
Status formal: Fase 9 `CONCLUIDA`
Fase 10: autorizada como proxima etapa

## 1. Resumo executivo

A Fase 8 esta `CONCLUIDA` com evidencia real de `ops:phase8` no EasyPanel. A Fase 9 tambem esta `CONCLUIDA`: a homologacao real final de 2026-06-29 retornou `ok=true`, com `smoke:public`, `smoke:auth`, `smoke:phase9` e `test:e2e:real` aprovados.

O ponto mais sensivel era o upload de documento PDF valido. Em rodada anterior, o backend retornou `503 DOCUMENT_SCAN_UNAVAILABLE`, `reason=scanner_required`. Em 2026-06-29, apos `freshclam`, o smoke autenticado com acoes aceitou upload PDF valido com HTTP 201, provando o scanner ClamAV no fluxo real.

A trilha final da Fase 9 fechou sem `failed` e sem `blocked`:

```txt
Pronto para avancar para a Fase 10.
```

Historico 2026-06-29 12:20: o smoke autenticado com acoes passou (`ok=true`, `failed=0`, `blocked=0`) depois da atualizacao das bases do ClamAV. O upload PDF valido foi aceito com HTTP 201. A rodada seguinte de `homologation:real` passou em `smoke:public`, `smoke:auth` e `smoke:phase9`, mas ainda falhou no `test:e2e:real`; esse bloqueio foi superado na rodada final de 13:40.

Atualizacao 2026-06-29 13:40: a homologacao real final passou (`ok=true`) com `smoke:public`, `smoke:auth`, `smoke:phase9` e `test:e2e:real` aprovados. A Fase 9 fica formalmente `CONCLUIDA`.

## 2. Evidencias recentes

### 2.1 Smoke publico Fase 9

Comando executado em `portal-sama-web`:

```powershell
npm.cmd run smoke:phase9 -- --json --soft --evidence-dir .ai-tests/phase9-smoke
```

Resultado:

```txt
ok=false
failed=0
blocked=1
phase9-route-shell=passed
credentials=blocked
```

Evidencia:

```txt
portal-sama-web/.ai-tests/phase9-smoke/phase9-smoke-20260626T180901611Z.json
```

Interpretacao: a rota `/dev/fase-9-smoke` serviu o shell da SPA com HTTP 200. A validacao autenticada nao rodou porque este terminal nao tinha `PORTAL_AUTH_USERNAME` e `PORTAL_AUTH_PASSWORD` configurados.

### 2.2 Health publico

Comando:

```powershell
Invoke-RestMethod -Uri "https://portal.samacontabil.com.br/api-v2/health" -Method Get
```

Resultado sanitizado:

```json
{
  "ok": true,
  "service": "portal-sama-api",
  "database": "up",
  "storage": "up",
  "uploadQuarantine": "up",
  "timestamp": "2026-06-26T18:09:08.151Z"
}
```

Interpretacao: banco, storage e quarentena estao OK no health publico. O scanner ainda precisa ser provado pelo fluxo real de upload.

### 2.3 Smoke autenticado com acoes controladas

Log informado pelo usuario em 2026-06-26:

```powershell
npm.cmd run smoke:phase9 -- --json --soft --evidence-dir .ai-tests/phase9-smoke
```

Variaveis relevantes no log:

```txt
applyActions=true
username=mascarado
```

Resultado:

```txt
ok=false
failed=0
blocked=1
```

Checks aprovados:

- rota `/dev/fase-9-smoke` serviu o shell da SPA com HTTP 200;
- CSRF/login passaram;
- `/auth/me` retornou usuario DEV com 99 permissoes;
- `/health` retornou `database=up`, `storage=up`, `uploadQuarantine=up`;
- listagem de clientes, contratos, Acessorias e documentos passou;
- contrato `INTERNAL` foi criado;
- contrato `ZAPSIGN` sandbox foi criado;
- `acessorias-controlled-sync` passou por polling de `sync-runs`;
- upload SVG invalido foi rejeitado;
- logout passou.

Bloqueio restante:

```txt
upload-valid-document=blocked
statusCode=503
code=DOCUMENT_SCAN_UNAVAILABLE
reason=scanner_required
```

Evidencia local:

```txt
portal-sama-web/.ai-tests/phase9-smoke/phase9-smoke-20260626T183520231Z.json
```

Conclusao: Acessorias nao e mais bloqueio nesta rodada. O unico bloqueio do `smoke:phase9` com acoes e o scanner de upload indisponivel em modo estrito.

### 2.4 Homologacao real final

Log informado pelo usuario em 2026-06-26:

```powershell
npm.cmd run homologation:real -- --json --soft --skip-permissions --evidence-dir .ai-tests/homologation-real-phase9
```

Resultado:

```txt
ok=false
smoke:public=passed
smoke:auth=passed
smoke:phase9=failed
smoke:permissions=skipped
test:e2e:real=failed
```

Detalhes importantes:

- `smoke:public` passou.
- `smoke:auth` passou, incluindo login, `/auth/me`, refresh, politica de cookie e logout.
- `smoke:phase9` falhou apenas porque o subcomando considera bloqueio de upload valido como saida 1.
- `test:e2e:real` falhou ao esperar o heading `Smoke backend`; o contexto de erro mostrou a tela de login em vez da tela `/dev/fase-9-smoke`.

Evidencia local:

```txt
portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260626T184910450Z.json
```

Conclusao: a autenticacao real por API/browser passou em parte relevante, mas a homologacao final ainda nao fecha por dois motivos: scanner indisponivel e falha do E2E real ao carregar a tela autenticada da Fase 9.

### 2.5 Continuidade apos ajuste de `clamscan` no EasyPanel

Log informado pelo usuario em 2026-06-29 apos ajuste no EasyPanel:

```powershell
npm.cmd run smoke:phase9 -- --json --soft --evidence-dir .ai-tests/phase9-smoke
```

Resultado:

```txt
ok=false
failed=0
blocked=1
applyActions=true
```

Checks aprovados:

- rota `/dev/fase-9-smoke` respondeu HTTP 200;
- CSRF/login, `/auth/me` e logout passaram;
- `/health` retornou `database=up`, `storage=up`, `uploadQuarantine=up`;
- listagem de clientes, contratos, Acessorias e documentos passou;
- contrato `INTERNAL` foi criado;
- contrato `ZAPSIGN` sandbox foi criado;
- `acessorias-controlled-sync` passou por polling de `sync-runs`;
- upload SVG invalido foi rejeitado.

Bloqueio persistente:

```txt
upload-valid-document=blocked
statusCode=503
code=DOCUMENT_SCAN_UNAVAILABLE
reason=scanner_required
```

Evidencia local:

```txt
portal-sama-web/.ai-tests/phase9-smoke/phase9-smoke-20260629T112420826Z.json
```

Conclusao: o `clamscan` ainda nao esta acessivel para o processo da API. O health valida quarentena, banco e storage, mas nao comprova execucao do scanner. O backend espera `SAMA_UPLOAD_SCAN_BIN` ou um binario `clamscan`/`clamdscan` no `PATH` do container da API.

Observacao de codigo: o `Dockerfile` atual da API ja instala `clamav` na imagem runner e define `SAMA_UPLOAD_SCAN_BIN=/usr/bin/clamscan`. Se o ambiente publicado ainda retorna `DOCUMENT_SCAN_UNAVAILABLE`, verificar especialmente:

- se o EasyPanel fez rebuild/redeploy com o Dockerfile atual;
- se alguma variavel de ambiente sobrescreveu `SAMA_UPLOAD_SCAN_BIN` para vazio ou caminho inexistente;
- se as assinaturas do ClamAV existem no container, pois `clamscan` sem database tambem fica indisponivel para o fluxo strict;
- se `freshclam` foi executado ou se `SAMA_CLAMAV_UPDATE_ON_START=true` foi usado conscientemente no redeploy.

Na mesma data, o comando:

```powershell
npm.cmd run homologation:real -- --json --soft --skip-permissions --evidence-dir .ai-tests/homologation-real-phase9
```

retornou:

```txt
smoke:public=passed
smoke:auth=passed
smoke:phase9=failed por DOCUMENT_SCAN_UNAVAILABLE
test:e2e:real=failed esperando Smoke backend enquanto o contexto mostrou tela de login
```

Evidencia local:

```txt
portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260629T113002807Z.json
```

### 2.6 Reexecucao do smoke apos novo ajuste

Log informado pelo usuario em 2026-06-29:

```powershell
node scripts/portal-phase9-smoke.mjs --json --soft --evidence-dir .ai-tests/phase9-smoke
```

Resultado:

```txt
ok=false
failed=0
blocked=1
applyActions=true
```

Checks aprovados:

- rota `/dev/fase-9-smoke` respondeu HTTP 200;
- CSRF/login, `/auth/me` e logout passaram;
- `/health` retornou `database=up`, `storage=up`, `uploadQuarantine=up`;
- listagem de clientes, contratos, Acessorias e documentos passou;
- contrato `INTERNAL` foi criado;
- contrato `ZAPSIGN` sandbox foi criado;
- `acessorias-controlled-sync` passou por polling de `sync-runs`;
- upload SVG invalido foi rejeitado.

Bloqueio persistente:

```txt
upload-valid-document=blocked
statusCode=503
code=DOCUMENT_SCAN_UNAVAILABLE
reason=scanner_required
```

Evidencia local:

```txt
portal-sama-web/.ai-tests/phase9-smoke/phase9-smoke-20260629T120202608Z.json
```

Conclusao: a reexecucao confirmou que o scanner ainda nao esta disponivel para o processo da API. Nao ha novo bloqueio funcional alem do scanner; Acessorias e os demais checks seguem aprovados.

### 2.7 Smoke autenticado aprovado apos `freshclam`

Log informado pelo usuario em 2026-06-29 apos `freshclam` confirmar bases atualizadas:

```txt
daily.cvd database is up-to-date
main.cvd database is up-to-date
bytecode.cvd database is up-to-date
```

Comando:

```powershell
npm.cmd run smoke:phase9 -- --json --soft --evidence-dir .ai-tests/phase9-smoke
```

Resultado:

```txt
ok=true
failed=0
blocked=0
applyActions=true
```

Checks aprovados:

- rota `/dev/fase-9-smoke` respondeu HTTP 200;
- CSRF/login, `/auth/me` e logout passaram;
- `/health` retornou `database=up`, `storage=up`, `uploadQuarantine=up`;
- listagem de clientes, contratos, Acessorias e documentos passou;
- contrato `INTERNAL` foi criado;
- contrato `ZAPSIGN` sandbox foi criado;
- `acessorias-controlled-sync` passou por polling de `sync-runs`;
- upload PDF valido foi aceito;
- upload SVG invalido foi rejeitado.

Evidencia do upload valido:

```txt
upload-valid-document=passed
statusCode=201
documentId=366b48a6-8ce5-4e04-8e82-92309ab661c8
status=PENDING
mime=application/pdf
durationMs=18848
```

Evidencia local:

```txt
portal-sama-web/.ai-tests/phase9-smoke/phase9-smoke-20260629T122012310Z.json
```

Conclusao: o bloqueio `DOCUMENT_SCAN_UNAVAILABLE` foi resolvido para o smoke autenticado com acoes. O scanner ClamAV esta comprovado no fluxo real de upload PDF valido.

### 2.8 Homologacao real apos scanner OK

Comando executado pelo usuario:

```powershell
npm.cmd run homologation:real -- --json --soft --skip-permissions --evidence-dir .ai-tests/homologation-real-phase9
```

Resultado informado:

```txt
ok=false
smoke:public=passed
smoke:auth=passed
smoke:phase9=passed
smoke:permissions=skipped
test:e2e:real=failed
```

Evidencia local:

```txt
portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260629T122850371Z.json
```

Falhas do Playwright real:

- no teste de login/home/cookies/logout, o clique em `Sair` nao levou para `/login`; a pagina permaneceu em `/home`;
- no teste da rota `/dev/fase-9-smoke`, o navegador esperava o heading `Smoke backend`, mas o contexto mostrou a tela de login.

Conclusao: a parte de backend/API da Fase 9 ja passou na homologacao real; o bloqueio restante ficou no fluxo de navegador real do Playwright.

Ajuste aplicado no web:

```txt
portal-sama-web/tests/e2e/real-auth.spec.ts
```

O helper `loginWithRealUser` passou a esperar a rota autenticada alvo e navegar explicitamente para ela em fallback. O teste de logout passou a aguardar a resposta `POST /auth/logout` antes de exigir a URL `/login`.

Validacoes locais apos o ajuste:

```txt
npm.cmd run lint
npx.cmd playwright test tests/e2e/real-auth.spec.ts --reporter=line
npm.cmd test -- --runInBand
```

Resultado: lint OK, spec real-auth OK com os testes reais pulados localmente por ausencia de env real no ambiente do Codex, e testes de contrato web OK.

### 2.9 Homologacao real apos primeiro ajuste E2E

Comando executado pelo usuario:

```powershell
npm.cmd run homologation:real -- --json --soft --skip-permissions --evidence-dir .ai-tests/homologation-real-phase9
```

Resultado informado:

```txt
ok=false
smoke:public=passed
smoke:auth=passed
smoke:phase9=passed
smoke:permissions=skipped
test:e2e:real=failed
```

Evidencia local:

```txt
portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260629T124834080Z.json
```

Avanco comprovado:

- o teste real de login/home/cookies/logout passou;
- a Fase 9 backend continuou aprovada dentro do `homologation:real`;
- Acessorias fechou `SUCCESS` por polling;
- upload PDF valido continuou aceito com HTTP 201;
- upload SVG invalido continuou rejeitado.

Falha remanescente:

- apenas o teste real da rota `/dev/fase-9-smoke` falhou aguardando o heading `Smoke backend`;
- o contexto mostrou a tela de login renderizada enquanto o teste ainda esperava a rota operacional.

Conclusao: o primeiro ajuste resolveu o logout. O problema restante era o helper considerar a URL suficiente para decidir que nao precisava logar, mesmo quando a SPA ainda renderizava a tela de login.

Segundo ajuste aplicado no web:

```txt
portal-sama-web/tests/e2e/real-auth.spec.ts
```

O helper `loginWithRealUser` passou a detectar a tela de login pelo heading `Entrar no portal`, aguardando ate 10s apos abrir a rota alvo. Se o formulario aparecer, o teste faz login; se a URL ja estiver correta mas o formulario ainda estiver visivel, ele navega novamente para a rota alvo e exige que o login desapareca.

Validacoes locais apos o segundo ajuste:

```txt
npm.cmd run lint
npx.cmd playwright test tests/e2e/real-auth.spec.ts --reporter=line
npm.cmd test -- --runInBand
git diff --check
```

Resultado: lint OK, spec real-auth OK com os testes reais pulados localmente por ausencia de env real no ambiente do Codex, testes de contrato web OK e `git diff --check` OK no web.

### 2.10 Homologacao real apos segundo ajuste E2E

Log informado pelo usuario em 2026-06-29:

```powershell
npm.cmd run homologation:real -- --json --soft --skip-permissions --evidence-dir .ai-tests/homologation-real-phase9
```

Resultado informado:

```txt
ok=false
smoke:public=passed
smoke:auth=passed
smoke:phase9=failed
smoke:permissions=skipped
test:e2e:real=failed
```

Evidencia local:

```txt
portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260629T132822279Z.json
```

Resultado do `smoke:phase9` nessa rodada:

- contratos `INTERNAL` e `ZAPSIGN` sandbox passaram;
- upload PDF valido continuou aceito com HTTP 201;
- upload SVG invalido continuou rejeitado;
- Acessorias retornou HTTP 409 `ACESSORIAS_HEAVY_JOB_LOCKED`.

Falhas E2E reais:

- no teste de login/home/cookies/logout, o navegador permaneceu em `/login`;
- no teste da rota `/dev/fase-9-smoke`, o contexto ainda mostrou a tela de login.

Conclusao: o scanner continua resolvido. O bloqueio operacional novo foi lock temporario do Acessorias, e o bloqueio E2E vinha do helper usar `locator.isVisible()` como se aguardasse renderizacao; em Playwright esse metodo nao espera a visibilidade como uma assertion.

Terceiro ajuste aplicado no web:

```txt
portal-sama-web/tests/e2e/real-auth.spec.ts
```

O helper `loginWithRealUser` passou a usar `loginHeading.waitFor({ state: 'visible', timeout: 10000 })`, aguardar a resposta `POST /auth/login` e navegar para a rota esperada somente depois de login HTTP OK.

Ajuste adicional no runner:

```txt
portal-sama-web/scripts/portal-phase9-smoke.mjs
```

O `smoke:phase9` passou a repetir o POST de `integrations/acessorias/operational-sync` quando receber `409 ACESSORIAS_HEAVY_JOB_LOCKED`, respeitando `PORTAL_PHASE9_ACESSORIAS_POLL_TIMEOUT_MS`. A saida registra apenas status, duracao e codigo publico das tentativas.

Validacoes locais apos os ajustes:

```txt
node --check scripts/portal-phase9-smoke.mjs
node --check scripts/portal-real-homologation.mjs
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
npx.cmd playwright test tests/e2e/real-auth.spec.ts --reporter=line
git diff --check
```

Resultado: todos OK no web; a spec real-auth local ficou com os testes reais pulados porque o processo do Codex nao possui `PORTAL_REAL_E2E`, `PORTAL_AUTH_USERNAME`, `PORTAL_AUTH_PASSWORD`, `PORTAL_E2E_USERNAME` ou `PORTAL_E2E_PASSWORD` no escopo `Process`, `User` ou `Machine`.

### 2.11 Homologacao real final aprovada

Comando executado pelo usuario:

```powershell
npm.cmd run homologation:real -- --json --soft --skip-permissions --evidence-dir .ai-tests/homologation-real-phase9
```

Resultado informado:

```txt
ok=true
smoke:public=passed
smoke:auth=passed
smoke:phase9=passed
smoke:permissions=skipped
test:e2e:real=passed
```

Periodo:

```txt
startedAt=2026-06-29T13:40:38.169Z
completedAt=2026-06-29T13:44:50.729Z
```

Evidencia local:

```txt
portal-sama-web/.ai-tests/homologation-real-phase9/homologation-real-20260629T134038169Z.json
```

Checks finais aprovados:

- `smoke:public`: frontend publico, health da API, CORS e CSRF passaram;
- `smoke:auth`: login, `/auth/me`, refresh e logout passaram;
- `smoke:phase9`: contratos `INTERNAL` e `ZAPSIGN` sandbox passaram;
- Acessorias fechou `SUCCESS` por polling de `sync-runs`;
- upload PDF valido foi aceito com HTTP 201;
- upload SVG invalido foi rejeitado com HTTP 400;
- `test:e2e:real`: os 2 testes reais do Playwright passaram.

Conclusao formal: todos os criterios documentados para a Fase 9 foram comprovados com evidencia sanitizada. A Fase 9 passa para `CONCLUIDA` e a Fase 10 fica autorizada como proxima etapa.

## 3. O que ja esta comprovado

| Area | Evidencia | Status |
| --- | --- | --- |
| Fase 8 operacional | `ops:phase8` real em 2026-06-22 com `ok=true`, `failed=0`, `blocked=0` | OK |
| Rota Fase 9 | `/dev/fase-9-smoke` HTTP 200 em producao | OK |
| Health publico | `database=up`, `storage=up`, `uploadQuarantine=up` | OK |
| Smoke read-only autenticado anterior | Rodada de 2026-06-25 com `ok=true`, usuario DEV e checks read-only OK | OK historico |
| Contratos no smoke com acoes anterior | Criacao `INTERNAL` e `ZAPSIGN` sandbox passou | OK historico |
| Upload invalido | SVG invalido foi rejeitado | OK historico |
| Acessorias | Runs disparados pelo smoke fecharam `SUCCESS` em `sync-runs`; polling novo passou no smoke de 2026-06-29T12:20:12.310Z | OK |
| Homologacao real final | `homologation-real-20260629T134038169Z.json` com `ok=true` | OK final |

## 4. Bloqueios reais

| Bloqueio | Severidade | Motivo |
| --- | --- | --- |
| Nenhum bloqueio remanescente da Fase 9 | - | A homologacao real final retornou `ok=true`; falhas anteriores ficam registradas apenas como historico. |

## 5. Passo a passo para fazer no EasyPanel

Execute em janela controlada, com backup recente e alguem disponivel para rollback.

### Passo 1 - Confirmar imagem publicada

Confirmar que API e Web publicados usam os commits atuais ou posteriores:

```txt
API: 63eb6be fix: tag acessorias operational sync runs
Web: 49ddab2 test: poll acessorias sync runs in phase 9 smoke
```

Sinal minimo de API nova:

```txt
GET /api-v2/health retorna uploadQuarantine=up
```

Esse sinal ja foi observado em 2026-06-26.

### Passo 2 - Configurar scanner na API

No servico da API do EasyPanel, garantir uma destas opcoes:

```txt
clamscan
clamdscan
SAMA_UPLOAD_SCAN_BIN apontando para binario equivalente
```

Regras:

- nao desativar scanner para passar teste;
- nao trocar modo strict por modo permissivo para liberar fase;
- se usar `clamdscan`, confirmar daemon ativo;
- confirmar assinaturas atualizadas;
- confirmar que o usuario da aplicacao consegue executar o binario;
- manter quarentena fora de pasta publica.

Diagnostico obrigatorio dentro do container/servico da API:

```sh
which clamscan || command -v clamscan
clamscan --version
ls -la /var/lib/clamav
```

Se `which clamscan` retornar, por exemplo, `/usr/bin/clamscan`, configurar no EasyPanel:

```txt
SAMA_UPLOAD_SCAN_MODE=strict
SAMA_UPLOAD_SCAN_BIN=/usr/bin/clamscan
SAMA_UPLOAD_SCAN_ARGS=
SAMA_UPLOAD_SCAN_TIMEOUT_SEC=45
```

Se `/var/lib/clamav` estiver sem bases como `daily.cvd`, `main.cvd` ou equivalentes, executar no container ou habilitar no redeploy:

```sh
freshclam
```

ou:

```txt
SAMA_CLAMAV_UPDATE_ON_START=true
```

Depois reiniciar/redeployar o servico da API. Se o comando `clamscan --version` nao funcionar dentro do container da API, o pacote foi instalado fora do container correto ou a imagem precisa ser recriada com ClamAV.

### Passo 3 - Conferir variaveis de storage/quarentena

Se houver caminho dedicado:

```txt
SAMA_UPLOAD_QUARANTINE_DIR=/caminho/privado/uploads/_quarantine
```

Se nao houver caminho dedicado, o backend usa:

```txt
STORAGE_PRIVATE_PATH/uploads/_quarantine
```

Resultado esperado em `/health`:

```txt
uploadQuarantine=up
```

### Passo 4 - Redeployar API

Depois de configurar scanner/quarentena, redeployar o servico da API no EasyPanel.

### Passo 5 - Validar health publico

No terminal local:

```powershell
Invoke-RestMethod -Uri "https://portal.samacontabil.com.br/api-v2/health" -Method Get
```

Resultado esperado:

```txt
ok=true
database=up
storage=up
uploadQuarantine=up
```

### Passo 6 - Configurar credenciais de teste localmente

No terminal local, sem imprimir valores:

```powershell
$env:PORTAL_AUTH_USERNAME = '<usuario-teste-dev-ou-admin>'
$env:PORTAL_AUTH_PASSWORD = '<senha-teste>'
```

Nao registrar usuario/senha em Markdown, print ou commit.

### Passo 7 - Rodar smoke autenticado read-only

```powershell
cd "C:\Users\Sama Contabilidade\Desktop\portal-sama-web"
npm.cmd run smoke:phase9 -- --json --soft --evidence-dir .ai-tests/phase9-smoke
```

Resultado esperado:

```txt
ok=true
failed=0
blocked=0
```

### Passo 8 - Rodar smoke com acoes controladas

```powershell
$env:PORTAL_PHASE9_APPLY_ACTIONS = '1'
npm.cmd run smoke:phase9 -- --json --soft --evidence-dir .ai-tests/phase9-smoke
```

Resultado esperado:

```txt
failed=0
blocked=0
upload-valid-document=passed
upload-invalid-document=passed
acessorias-controlled-sync=passed
```

Observacao: se a chamada sincrona do Acessorias exceder timeout/HTTP 504, o runner pode aprovar por polling de `GET /integrations/acessorias/sync-runs`, desde que catalogo e entregas terminem `SUCCESS`.

Na rodada informada em 2026-06-26, Acessorias ja passou por esse caminho. Se o proximo smoke repetir o mesmo comportamento, tratar como OK.

### Passo 9 - Rodar homologacao real final

```powershell
$env:PORTAL_REAL_E2E = '1'
$env:PORTAL_E2E_USERNAME = $env:PORTAL_AUTH_USERNAME
$env:PORTAL_E2E_PASSWORD = $env:PORTAL_AUTH_PASSWORD
npm.cmd run homologation:real -- --json --soft --skip-permissions --evidence-dir .ai-tests/homologation-real-phase9
```

Resultado esperado:

```txt
ok=true
failed=0
blocked=0
```

Se a matriz de permissoes estiver disponivel, remover `--skip-permissions` e configurar `PORTAL_PERMISSION_MATRIX_FILE` ou `PORTAL_PERMISSION_MATRIX_JSON`.

Se `test:e2e:real` falhar novamente esperando `Smoke backend` e mostrando a tela de login, investigar antes de concluir a fase:

- confirmar se o usuario real tem acesso a `/dev/fase-9-smoke`;
- abrir a rota manualmente apos login no navegador;
- verificar os arquivos de erro em `portal-sama-web/test-results`;
- se manualmente volta para login, corrigir protecao de rota/sessao no frontend publicado;
- se manualmente funciona, revisar novamente o helper `loginWithRealUser` e o fluxo de logout em `portal-sama-web/tests/e2e/real-auth.spec.ts`.

## 6. Criterio para concluir a Fase 9

Todos estes itens foram comprovados na trilha final:

- smoke publico passa;
- smoke autenticado read-only passa;
- smoke autenticado com acoes passa;
- upload PDF valido passa com scanner ativo;
- upload invalido continua rejeitado;
- contratos `INTERNAL` e `ZAPSIGN` sandbox passam;
- Acessorias fecha `SUCCESS`, direto ou por polling;
- `homologation:real` final passa;
- evidencias sanitizadas ficam registradas;
- nenhum segredo, cookie, token, senha ou payload bruto foi salvo.

Status final: Fase 9 `CONCLUIDA`.

## 7. Proxima etapa permitida

A proxima etapa autorizada e a Fase 10 do guia raiz:

```txt
Fase 10 - Base final do frontend e design system operacional
```

Melhorias visuais grandes, novas telas finais e polimento amplo de UX devem partir da Fase 10, mantendo as evidencias da Fase 9 preservadas.
