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
