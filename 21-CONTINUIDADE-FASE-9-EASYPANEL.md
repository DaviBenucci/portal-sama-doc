# Continuidade Fase 9 - EasyPanel

Atualizado em: 2026-06-26  
Ambiente alvo: `https://portal.samacontabil.com.br`  
Status formal: Fase 9 `EM_EXECUCAO`  
Fase 10: nao autorizada

## 1. Resumo executivo

A Fase 8 esta `CONCLUIDA` com evidencia real de `ops:phase8` no EasyPanel. A Fase 9 esta quase fechada, mas ainda nao pode ser marcada como `CONCLUIDA` porque falta evidencia real final do smoke autenticado com acoes controladas.

O ponto mais sensivel e o upload de documento PDF valido. Em rodada anterior, o backend retornou `503 DOCUMENT_SCAN_UNAVAILABLE`, `reason=scanner_required`. Em 2026-06-26, o health publico ja retornou `uploadQuarantine=up`, provando que a quarentena esta gravavel no ambiente publicado. Isso ainda nao prova que o scanner de malware esta ativo.

Enquanto o scanner nao for validado por upload PDF valido e enquanto o smoke autenticado com acoes nao fechar sem `failed` e sem `blocked`, o Portal Sama segue como:

```txt
Nao pronto para usuarios reais.
```

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

## 3. O que ja esta comprovado

| Area | Evidencia | Status |
| --- | --- | --- |
| Fase 8 operacional | `ops:phase8` real em 2026-06-22 com `ok=true`, `failed=0`, `blocked=0` | OK |
| Rota Fase 9 | `/dev/fase-9-smoke` HTTP 200 em producao | OK |
| Health publico | `database=up`, `storage=up`, `uploadQuarantine=up` | OK |
| Smoke read-only autenticado anterior | Rodada de 2026-06-25 com `ok=true`, usuario DEV e checks read-only OK | OK historico |
| Contratos no smoke com acoes anterior | Criacao `INTERNAL` e `ZAPSIGN` sandbox passou | OK historico |
| Upload invalido | SVG invalido foi rejeitado | OK historico |
| Acessorias | Runs disparados pelo smoke fecharam `SUCCESS` em `sync-runs` | OK historico, precisa repetir com polling novo |

## 4. Bloqueios reais

| Bloqueio | Severidade | Motivo |
| --- | --- | --- |
| Scanner de upload ainda sem evidencia final | Bloqueante | Documento empresarial valido nao pode ser aceito se o scanner strict estiver indisponivel. |
| Smoke com acoes controladas ainda precisa ser repetido apos scanner | Bloqueante | A Fase 9 so fecha com upload valido, upload invalido, contratos e Acessorias comprovados na mesma trilha final. |
| E2E real da rota `/dev/fase-9-smoke` falhou no navegador | Alto | O teste esperava `Smoke backend`, mas o contexto mostrou a tela de login. Depois do scanner, reexecutar; se persistir, corrigir fluxo de redirecionamento pos-login ou o helper do teste. |

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
- se manualmente funciona, ajustar o helper `loginWithRealUser` em `portal-sama-web/tests/e2e/real-auth.spec.ts` para fazer login, esperar sessao autenticada e navegar explicitamente para `/dev/fase-9-smoke`;
- se manualmente volta para login, corrigir protecao de rota/sessao no frontend publicado.

## 6. Criterio para concluir a Fase 9

So alterar status da Fase 9 para `CONCLUIDA` quando todos estes itens estiverem comprovados:

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

## 7. Proxima etapa permitida

Depois da Fase 9 `CONCLUIDA`, a proxima etapa autorizada sera a Fase 10 do guia raiz:

```txt
Fase 10 - Base final do frontend e design system operacional
```

Antes disso, qualquer melhoria visual grande, nova tela final ou polimento amplo de UX deve aguardar.
