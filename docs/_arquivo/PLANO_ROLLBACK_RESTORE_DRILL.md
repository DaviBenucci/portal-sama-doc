# [PARCIAL] Plano de Rollback e Restore Drill - Portal Sama

Ultima atualizacao: 2026-05-26 16:50 -03:00
Responsavel/IA: Codex
Status: plano operacional criado; restore real ainda pendente

Atualizacao 2026-06-01 09:30 -03:00: `portal-sama-api` ganhou `npm run ops:phase1`, um orquestrador da Fase 1 que executa migrations, seed, readiness, backup, verificacao e restore drill em sequencia, gerando evidencia JSON. O restore real em alvo isolado continua dependendo de `--apply-database`, `--apply-storage` e `--confirm RESTORE_DRILL_TARGET_IS_ISOLATED`.

## Objetivo

Definir o procedimento minimo para provar que os backups do `portal-sama-api`
podem ser restaurados em alvo isolado antes de backfills, corte para producao ou
desligamento do legado.

Este plano nao autoriza restaurar sobre o banco ou storage da aplicacao. O drill
deve usar banco e caminho de storage separados.

## Artefatos obrigatorios

Gerar no container real `portal-sama-api`:

```bash
npm run ops:backup:create -- --output-dir /tmp/portal-sama-backups --include-storage-archive
```

Verificar antes de qualquer restore:

```bash
npm run ops:backup:verify -- --backup-dir /tmp/portal-sama-backups/<backup-id>
```

Arquivos esperados:

- `metadata.json`
- `database.sql.gz`
- `storage-manifest.json`
- `storage.tar.gz`, quando o archive de storage for solicitado

## Alvos isolados

Criar um banco de drill diferente do banco da aplicacao. Exemplo:

```txt
banco_sama_restore_drill
```

Criar tambem um caminho de storage separado. Exemplo:

```txt
/tmp/portal-sama-restore-storage
```

Regras:

- O alvo de banco nao pode ser o mesmo `DATABASE_URL` da API.
- O alvo de storage nao pode ser o mesmo `STORAGE_PRIVATE_PATH` da API.
- O drill real so deve ser executado apos copiar os artefatos para fora do container ou garantir retencao externa equivalente.
- Nao registrar senhas, tokens, cookies ou segredos na documentacao.

## Preflight em modo seco

Antes de restaurar, rodar:

```bash
npm run ops:restore:drill -- \
  --backup-dir /tmp/portal-sama-backups/<backup-id> \
  --target-database-url mysql://portal_restore:SENHA_URL_ENCODED@portal-sama-database:3306/banco_sama_restore_drill \
  --target-storage-path /tmp/portal-sama-restore-storage \
  --json
```

Esse comando:

- roda a verificacao de integridade do backup;
- valida que o banco alvo nao e o `DATABASE_URL` da aplicacao;
- valida que o storage alvo nao e o `STORAGE_PRIVATE_PATH`;
- planeja a restauracao sem executar importacao nem extracao por padrao.

## Execucao real do restore drill

Somente apos o preflight passar, executar:

```bash
npm run ops:restore:drill -- \
  --backup-dir /tmp/portal-sama-backups/<backup-id> \
  --target-database-url mysql://portal_restore:SENHA_URL_ENCODED@portal-sama-database:3306/banco_sama_restore_drill \
  --target-storage-path /tmp/portal-sama-restore-storage \
  --apply-database \
  --apply-storage \
  --confirm RESTORE_DRILL_TARGET_IS_ISOLATED \
  --json
```

Se o drill for somente de storage, usar `--skip-database --apply-storage`.
Se o drill for somente de banco, usar `--apply-database` sem `--apply-storage`.

## Evidencia a registrar

Registrar em `RELATORIO_TESTES.md` e neste documento:

- data/hora;
- responsavel;
- backup id;
- local externo onde os artefatos foram guardados;
- hashes SHA-256 dos artefatos, conforme `metadata.json`;
- resultado de `ops:backup:verify`;
- resultado de `ops:restore:drill`;
- observacoes de falha, tempo de restauracao e acao corretiva.

## Status atual

Em 2026-05-26 16:50 -03:00:

- `portal-sama-api` possui `npm run ops:restore:drill`;
- o comando foi validado localmente com backup descartavel sem banco e restore de storage em `.ai-tests`;
- o restore real de `database.sql.gz` no EasyPanel ainda nao foi executado;
- o backup real com `DATABASE_URL`/`STORAGE_PRIVATE_PATH` reais ainda precisa ser gerado, verificado, copiado para fora e restaurado em alvo isolado.

## Criterio para fechar a pendencia

So marcar backup/rollback como concluido para producao quando:

- backup real do EasyPanel for gerado;
- artefatos forem verificados por `ops:backup:verify`;
- artefatos forem copiados para local externo ao container;
- `ops:restore:drill` for executado em banco/storage isolados;
- a aplicacao ou consultas minimas confirmarem que dados e arquivos restaurados estao coerentes;
- a evidencia sanitizada estiver registrada na documentacao.
