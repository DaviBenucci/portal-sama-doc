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

- Fases 0 a 7: registradas como concluidas no acompanhamento.
- Fase 8: `BLOQUEADA` formalmente apos conciliacao.
- Fase 9: nao esta formalmente autorizada enquanto a Fase 8 seguir bloqueada.

Motivo do bloqueio: as validacoes locais da Fase 8 foram executadas, mas o criterio da raiz exige readiness, schema, secrets, backup e restore drill reais no ambiente alvo ou bloqueio externo explicitamente aceito.

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

### Docs - Fase 8 e conciliacao

- Criado `docs/21-CONTRATO-API-FRONTEND.md`.
- Atualizados `docs/08-CHECKLIST-GO-LIVE.md`, `docs/09-MATRIZ-PERMISSOES-RBAC.md`, `docs/10-MATRIZ-ROTAS-PUBLICAS.md` e `docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md`.
- `00-LEIA-ME-NOVA-ARQUITETURA-CODEX.md` recebeu a secao `Precedencia documental`.
- `docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md` foi corrigido: Fase 8 reclassificada para `BLOQUEADA`, e Fase 9 marcada como nao autorizada formalmente.

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

- `npm.cmd run lint` - OK na rodada da Fase 7.
- `npm.cmd run build` - OK na rodada da Fase 7.
- `npm.cmd test -- --runInBand` - OK, 12 testes de contrato.
- `git diff --check` - OK.
- Servidor local Vite foi iniciado anteriormente em `http://127.0.0.1:5173`.

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

## Bloqueios formais da Fase 8

1. Rodar `ops:readiness` no ambiente alvo com:
   - `NODE_ENV=production`;
   - HTTPS/CORS produtivo;
   - usuario DEV/ADMIN ativo e controlado;
   - ClamAV instalado;
   - `SAMA_UPLOAD_SCAN_MODE=strict`;
   - storage real privado.
2. Rodar `ops:schema:check` no ambiente alvo e decidir sobre a collation default do database.
3. Rodar `ops:secrets:check` com secrets reais do ambiente alvo sem imprimir valores.
4. Criar backup real com dump de banco e storage alvo.
5. Rodar `ops:backup:verify` nos artefatos reais.
6. Rodar `ops:restore:drill` em banco/storage isolados reais.
7. Registrar evidencias sem segredos.

Comando consolidado preferencial para tentar desbloquear:

```powershell
npm.cmd run ops:phase8 -- --json --soft --backup-output-dir <backup-real> --target-database-url <mysql-isolado> --target-storage-path <storage-isolado> --apply-database --apply-storage --confirm RESTORE_DRILL_TARGET_IS_ISOLATED
```

O comando deve rodar no container/ambiente alvo com variaveis reais ja carregadas pelo provedor/secret manager. Nao usar `.env` impresso ou copiado para evidencia.

Enquanto esses itens estiverem pendentes, nao iniciar Fase 9 como fase formal.

## Status dos worktrees

### `portal-sama-api`

Alteracoes locais nao commitadas da Fase 7/Fase 8:

- `package.json`
- `scripts/run-operational-phase8.js`
- `scripts/validate-secret-rotation.js`
- `src/common/security/rate-limit-metadata.spec.ts`
- `src/modules/contracts/contracts.controller.ts`
- `src/modules/contracts/contracts.module.ts`
- `src/modules/contracts/contracts.service.spec.ts`
- `src/modules/contracts/contracts.service.ts`
- `src/modules/contracts/contracts.types.ts`
- `src/modules/contracts/dto/create-contract.dto.ts`
- `src/modules/contracts/dto/list-contracts.dto.ts`
- `src/modules/contracts/dto/send-signature.dto.ts`
- `src/modules/contracts/providers/`

### `portal-sama-web`

Alteracoes locais nao commitadas da Fase 7:

- `src/pages/contracts/ContractPage.tsx`
- `src/pages/contracts/PublicSignaturePage.tsx`
- `src/schemas/contract.schema.ts`
- `src/services/contracts.service.ts`
- `src/types/contracts.ts`

### `portal-sama-docs`

Alteracoes locais nao commitadas:

- `00-LEIA-ME-NOVA-ARQUITETURA-CODEX.md`
- `CONTEXTO-CODEX-ATUAL.md`
- `docs/08-CHECKLIST-GO-LIVE.md`
- `docs/09-MATRIZ-PERMISSOES-RBAC.md`
- `docs/10-MATRIZ-ROTAS-PUBLICAS.md`
- `docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md`
- `docs/21-CONTRATO-API-FRONTEND.md`

## Proximo chat deve fazer

1. Ler primeiro os documentos da raiz listados em `Precedencia obrigatoria`.
2. Ler `docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md` apenas como evidencia subordinada.
3. Confirmar que a Fase 8 esta `BLOQUEADA`.
4. Nao iniciar Fase 9 formalmente enquanto os bloqueios da Fase 8 nao forem resolvidos ou declarados como bloqueio externo pelo usuario.
5. Se o usuario fornecer acesso/configuracao do ambiente alvo, executar os checks operacionais reais na ordem:
   - preferencialmente `npm.cmd run ops:phase8 -- --json --soft --backup-output-dir <backup-real> --target-database-url <mysql-isolado> --target-storage-path <storage-isolado> --apply-database --apply-storage --confirm RESTORE_DRILL_TARGET_IS_ISOLATED`;
   - ou, se precisar depurar, executar readiness, schema, secrets, backup real, verify e restore drill isolado separadamente.
6. Se o usuario nao fornecer ambiente alvo, manter o bloqueio registrado e limitar alteracoes a documentacao, scripts de apoio ou correcoes que nao finjam liberar a Fase 8.
7. Ao alterar codigo, reexecutar lint/build/test correspondentes e `git diff --check`.

## Cuidados

- Nao ler nem imprimir `.env` real ou secrets.
- Nao reverter alteracoes locais.
- Nao usar `prisma db push` em ambiente compartilhado/producao.
- Nao mascarar falha operacional com `--skip-*` para liberar fase.
- Nao considerar backup valido sem restore drill.
- Nao iniciar frontend completo antes dos gates formais.
