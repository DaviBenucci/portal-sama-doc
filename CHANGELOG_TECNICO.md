# Changelog Técnico - Portal Sama

## 2026-05-27 11:05

### Arquivos alterados

- `portal-sama-web/src/services/integra-ai.service.ts`
- `portal-sama-web/src/pages/accounting/IntegraAiPage.tsx`
- `portal-sama-web/src/index.css`
- `portal-sama-web/tests/e2e/smoke.spec.ts`
- `portal-sama-web/eslint.config.js`
- Documentacao de status, pendencias, testes e pagina Integra-AI.

### O que mudou

- O download TXT do Integra-AI deixou de usar link direto e passou a usar `api.get(..., { responseType: 'blob' })`, preservando o `Authorization` injetado pelo interceptor Axios.
- A tela substituiu anchors de download por botoes com estado de carregamento, tratamento de erro e download por blob local.
- A pagina `/contabil/integra-ai` recebeu alinhamento proprio para ficar ao lado da sidebar em viewport larga.
- O smoke Playwright do Integra-AI passou a validar download autenticado e alinhamento da pagina com a sidebar.
- ESLint passou a ignorar artefatos temporarios de Playwright.

### Motivo da alteracao

O usuario relatou `UNAUTHORIZED` ao baixar o TXT gerado. Como o access token da stack React fica em memoria, navegacao direta para a URL protegida nao envia o header `Authorization`, mesmo com a sessao ativa. Tambem foi relatado desalinhamento/sobreposicao visual da marca/titulo do Integra-AI com a sidebar.

### Impacto esperado

- O botao "Baixar" do TXT usa a mesma sessao autenticada das demais chamadas API v2.
- O erro de token invalido por download direto deve desaparecer apos deploy do Web.
- A pagina Integra-AI fica visualmente ancorada ao lado da sidebar.

### Testes executados

- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e -- -g "Integra-AI"` em `portal-sama-web`: passou, 2 testes.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `git diff --check` em `portal-sama-web`: passou.

### Riscos ou pendencias

- Falta validar em homologacao real com usuario contabil, job real e TXT real apos deploy.
- A validacao local usou API mockada; ela prova o header no request do browser, mas nao substitui auditoria real de download no backend.

## 2026-05-27 10:40

### Arquivos alterados

- `portal-sama-api/services/integra_ai_parser/parse_statement.py`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUCAO.MD`
- `paginas/contabil-integra-ai.md`

### O que mudou

- Criado layout `inter_pdf` para Banco Inter.
- O parser agora reconhece datas longas em portugues, mantem a data corrente para transacoes seguintes, junta descricoes em multiplas linhas e entende valores `R$`/`-R$`.
- `Saldo do dia` e classificado como `balance_snapshot`/`daily_balance`, ficando fora das linhas contabeis exibidas/exportadas.

### Motivo da alteracao

O PDF real do Banco Inter retornava banco e periodo, mas nenhuma linha de transacao, porque o layout nao segue o padrao generico com data em cada lancamento.

### Impacto esperado

- PDFs do Banco Inter com esse layout passam a gerar transacoes validas.
- Saldos diarios podem apoiar reconciliacao futura sem poluir a tela final, regras ou TXT Dominio.

### Testes executados

- Parser direto no PDF real: passou, com 31 transacoes e 15 saldos tecnicos.
- `python -m py_compile services/integra_ai_parser/parse_statement.py`: passou.
- `npm.cmd test -- accounting.service.spec.ts --runInBand`, `npm.cmd run lint` e `npm.cmd run build` em `portal-sama-api`: passaram.

### Riscos ou pendencias

- Ainda falta matriz automatizada de fixtures PDF por banco.
- Outros layouts do Banco Inter ou de bancos diferentes podem exigir adaptadores especificos.

## 2026-05-27 10:03

### Arquivos alterados

- `portal-sama-api/.env.example`
- `portal-sama-api/src/config/env.schema.ts`
- `portal-sama-api/src/modules/accounting/accounting.controller.ts`
- `portal-sama-api/src/modules/accounting/accounting.service.ts`
- `portal-sama-api/src/modules/accounting/accounting.service.spec.ts`
- `portal-sama-api/src/modules/accounting/accounting.types.ts`
- `portal-sama-api/src/modules/accounting/integra-ai-parser.service.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-web/src/pages/accounting/IntegraAiPage.tsx`
- `portal-sama-web/src/services/integra-ai.service.ts`
- `portal-sama-web/src/types/integra-ai.ts`
- `portal-sama-web/tests/e2e/smoke.spec.ts`
- Documentacao de status, pendencias, testes, deploy, seguranca e matrizes.

### O que mudou

- A importacao OFX do Integra-AI foi liberada como recurso opt-in por `SAMA_INTEGRA_AI_OFX_IMPORT_ENABLED`.
- A API passou a validar e armazenar PDF/OFX, chamar o parser com `--ext pdf|ofx`, persistir `source_type` dinamico e expor capabilities `pdf_import`/`ofx_import`.
- O frontend passou a ajustar textos e `accept` do upload conforme capability, mantendo PDF como fluxo padrao.
- A documentacao externa analisada foi incorporada sem migration de banco.

### Motivo da alteracao

O parser Python ja tinha suporte a OFX, mas API/Web publicavam apenas PDF. A mudanca reduz uma lacuna documentada sem abrir o recurso diretamente em producao.

### Impacto esperado

- Ambientes atuais continuam aceitando PDF.
- Homologacao pode habilitar OFX por flag e testar bancos reais sem criar parser por banco.
- Producao permanece protegida enquanto a flag ficar `false`.

### Testes executados

- `npm.cmd test -- accounting.service.spec.ts --runInBand` em `portal-sama-api`: passou, 7 testes.
- `npm.cmd run lint`, `npm.cmd run build` e `npm.cmd run prisma:validate` em `portal-sama-api`: passaram.
- `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e -- -g "Integra-AI"` e `npm.cmd run test:e2e` em `portal-sama-web`: passaram.

### Riscos ou pendencias

- Falta validar com OFX real, parser Python no container, usuario contabil, MySQL/storage reais e ClamAV strict.
- Manter `SAMA_INTEGRA_AI_OFX_IMPORT_ENABLED=false` em producao ate haver evidencia real.

## 2026-05-27 08:31

### Arquivos alterados

- `portal-sama-web/src/pages/accounting/IntegraAiPage.tsx`
- `portal-sama-web/tests/e2e/smoke.spec.ts`
- `portal-sama-web/playwright.config.ts`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `paginas/contabil-integra-ai.md`
- `INVENTARIO_SEGURANCA.md`

### O que mudou

- Adicionados `aria-label` nos campos customizados de conta do Integra-AI para contas fixas do plano e conta editavel das regras.
- Criado smoke Playwright local para `/contabil/integra-ai`, com API v2 mockada, cobrindo pagina 2 da tabela de regras, busca por nome de conta, selecao de sugestao e autosave por blur/codigo numerico.
- Ajustado `playwright.config.ts` para nao executar testes do mesmo arquivo em paralelo, pois a suite local com Vite/GSAP/dynamic import apresentou falhas de concorrencia no Windows; a execucao padrao voltou a passar.

### Motivo da alteracao

Dar continuidade a pendencia parcial registrada em 2026-05-26 para Integra-AI, criando evidencia local repetivel antes da validacao real com job contabil e usuario de homologacao.

### Impacto esperado

- Reduz risco de regressao no autosave das regras contabeis em tabelas paginadas.
- Melhora acessibilidade/testabilidade dos campos customizados de contas.
- Deixa `npm.cmd run test:e2e` mais estavel no ambiente local.

### Testes executados

- `npm.cmd run test:e2e -- -g "Integra-AI"` em `portal-sama-web`: passou, 1 teste.
- `npm.cmd run test:e2e` em `portal-sama-web`: passou, 10 testes e 1 skipped opt-in real.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.

### Riscos ou pendencias

- A validacao local usa API mockada; ainda falta homologar com job real do Integra-AI, usuario contabil, MySQL real e deploy publicado.
- O Playwright real de autenticacao continua opt-in/skipped quando variaveis reais nao estao definidas.

## 2026-05-26 16:50

### Arquivos alterados

- `portal-sama-api/package.json`
- `portal-sama-api/.env.example`
- `portal-sama-api/README.md`
- `portal-sama-api/scripts/restore-drill-operational-backup.js`
- `portal-sama-api/scripts/validate-operational-readiness.js`
- `PLANO_ROLLBACK_RESTORE_DRILL.md`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `EASYPANEL_DEPLOY.md`
- `SEGURANCA.md`
- `INVENTARIO_SEGURANCA.md`
- `DECISOES_ARQUITETURA.md`

### O que mudou

- Criado `npm run ops:restore:drill` no `portal-sama-api`.
- O comando executa preflight/dry-run por padrao, valida integridade dos artefatos via `ops:backup:verify`, bloqueia alvo igual a `DATABASE_URL`/`STORAGE_PRIVATE_PATH` e exige frase explicita para aplicar restore.
- Criado `PLANO_ROLLBACK_RESTORE_DRILL.md` com procedimento, comandos, evidencias esperadas e criterio para fechar a pendencia.
- O aviso `backup-rollback` do readiness passou a apontar a sequencia com `ops:restore:drill`.

### Motivo da alteracao

Reduzir a pendencia de rollback documentada em producao, transformando o restore drill em um procedimento executavel e com protecoes contra restauracao acidental no banco/storage da aplicacao.

### Impacto esperado

- Operadores passam a ter um caminho repetivel para provar restore em alvo isolado.
- O plano de rollback fica documentado e versionado.
- O restore real continua pendente ate ser executado no EasyPanel com backup, banco e storage reais.

### Testes executados

- `node --check scripts/restore-drill-operational-backup.js`: passou.
- `npm.cmd run ops:restore:drill -- --help`: passou.
- `npm.cmd run ops:backup:create -- --skip-database --storage-path .ai-tests/restore-drill-storage --output-dir .ai-tests/restore-drill-backups --include-storage-archive --json`: passou.
- `npm.cmd run ops:restore:drill -- --backup-dir <backup-local> --skip-database --target-storage-path .ai-tests/restore-drill-target --json`: passou em dry-run.
- `npm.cmd run ops:restore:drill -- --backup-dir <backup-local> --skip-database --target-storage-path .ai-tests/restore-drill-target --apply-storage --confirm RESTORE_DRILL_TARGET_IS_ISOLATED --json`: passou e restaurou `sample.txt` em alvo isolado local.
- `node --check scripts/validate-operational-readiness.js`: passou.
- `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run prisma:validate` e `git diff --check` em `portal-sama-api`: passaram.

### Riscos ou pendencias

- Ainda falta gerar backup real no EasyPanel, verificar artefatos reais, copiar para fora do container e executar `ops:restore:drill` contra banco/storage isolados reais.
- O teste local validou storage com backup sem banco; importacao real de `database.sql.gz` depende de alvo MySQL isolado.

## 2026-05-26 16:08

### Arquivos alterados

- `portal-sama-api/src/modules/accounting/accounting.service.ts`
- `portal-sama-api/src/modules/accounting/accounting.service.spec.ts`
- `portal-sama-web/src/index.css`
- `portal-sama-web/src/pages/accounting/IntegraAiPage.tsx`
- `portal-sama-web/tests/e2e/smoke.spec.ts`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`

### O que mudou

- Corrigida a busca de contas do plano por texto no Integra-AI: termos sem digitos nao sao mais interpretados como match de todos os codigos.
- A tela React `/contabil/integra-ai` passou a autosalvar regras contabeis por linha; categoria, historico e sem uso usam debounce, e a conta editavel salva somente quando o campo perde foco ou quando a sugestao e selecionada.
- Removida a coluna/botao manual `Salvar` da tabela de regras contabeis.
- O sidebar desktop recolhido passou a usar uma marca compacta no topo, com teste para garantir que ela nao sobreponha o primeiro item do menu.

### Motivo da alteracao

Permitir filtrar contas por nome na etapa 3 e nas regras contabeis, alem de reduzir o risco operacional de esquecer de salvar uma regra preenchida.

### Impacto esperado

- Digitar nomes como `merc` passa a filtrar por nome/classificacao em vez de retornar as contas bancarias padrao.
- O operador nao precisa clicar em `Salvar` a cada regra; ao editar contas, a persistencia espera a saida do foco para permitir selecionar a sugestao correta.
- O menu lateral recolhido deixa de ter a imagem da marca invadindo a area dos icones.

### Testes executados

- `npm.cmd test -- accounting.service.spec.ts --runInBand` em `portal-sama-api`: passou, 4 testes.
- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e -- -g "sidebar"` em `portal-sama-web`: passou, 2 testes.
- `git diff --check` em `portal-sama-api` e `portal-sama-web`: passou.

### Riscos ou pendencias

- Ainda falta validar em homologacao real, com job contabil real, que o filtro por nome usa os dados reais do plano e que a conta editavel salva ao sair do foco em tabela paginada.

## 2026-05-26 09:38

### Arquivos alterados

- `portal-sama-api/scripts/validate-operational-readiness.js`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `SEGURANCA.md`
- `INVENTARIO_SEGURANCA.md`

### O que mudou

- `ops:readiness` passou a validar tamanho minimo de 32 caracteres para `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET`, `CSRF_SECRET` e `CERTIFICATE_ENCRYPTION_KEY` quando configurados.
- Documentacao atualizada com a execucao real autenticada usando o usuario bootstrap/DEV e com o diagnostico de que `DATABASE_URL` aponta para hostname interno do EasyPanel.

### Motivo da alteracao

Fechar o maximo possivel da homologacao real com as credenciais disponiveis localmente e impedir que secrets curtos/placeholders passem como ambiente pronto.

### Impacto esperado

- Autenticacao, refresh cookie, Playwright real e matriz minima DEV/bootstrap deixam de ser apenas ferramentas disponiveis e passam a ter evidencia real.
- Backfill/backup/restore ficam corretamente direcionados para execucao no container EasyPanel.
- Readiness fica mais fiel ao schema de ambiente usado pela aplicacao.

### Testes executados

- `npm.cmd run smoke:auth -- --soft --json`: passou com usuario bootstrap/DEV.
- `npm.cmd run smoke:permissions -- --soft --json`: passou com anonimos 401 e bootstrap/DEV 200 em rotas administrativas basicas.
- `npm.cmd run test:e2e:real`: passou com 1 teste real.
- `npm.cmd run homologation:real -- --soft --evidence-dir .ai-tests/homologation-real`: passou.
- `npm.cmd run ops:backfill:report -- --soft --json`: bloqueou por hostname interno do EasyPanel, como esperado em execucao local.
- `npm.cmd run ops:readiness -- --skip-clamav --storage-path .ai-tests/readiness-storage --soft --json`: retornou falhas controladas para banco interno e secrets curtos.
- `node --check scripts/validate-operational-readiness.js`, `npm.cmd run lint` e `npm.cmd run build` na API: passaram.

### Riscos ou pendencias

- Ainda falta matriz oficial por multiplos perfis reais.
- Ainda falta rodar backfill/backup/restore no container EasyPanel.
- Ainda faltam upload/download real, QA final e plano de corte/desligamento do legado.

## 2026-05-26 09:23

### Arquivos alterados

- `portal-sama-web/package.json`
- `portal-sama-web/scripts/portal-real-homologation.mjs`
- `portal-sama-web/README.md`
- `portal-sama-api/scripts/validate-operational-readiness.js`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `EASYPANEL_DEPLOY.md`

### O que mudou

- Criado `npm.cmd run homologation:real` no `portal-sama-web`.
- O runner executa `smoke:public`, `smoke:auth`, `smoke:permissions` e `test:e2e:real` em sequencia.
- Antes dos checks autenticados, o runner verifica somente a presenca das variaveis necessarias e registra blockers sem imprimir valores sensiveis.
- O runner pode gerar resumo JSON sanitizado em `.ai-tests/homologation-real`.
- `ops:readiness --soft --json` na API passou a retornar falhas controladas quando `DATABASE_URL`/ambiente real nao existem, em vez de abortar por erro Prisma.

### Motivo da alteracao

Dar continuidade aos itens reais pendentes de homologacao e deixar a proxima execucao com credenciais reais em um unico comando auditavel, sem depender de requests manuais.

### Impacto esperado

- Operadores conseguem rodar a bateria real Web/HTTP/navegador com uma unica chamada.
- Ambientes sem credenciais passam a mostrar exatamente quais variaveis faltam, sem vazar segredos.
- Readiness em modo diagnostico fica mais util para evidencia de ambiente incompleto.

### Testes executados

- `npm.cmd run smoke:public`: passou contra o dominio publico.
- `npm.cmd run smoke:auth -- --soft --json`: bloqueou por falta de credenciais, como esperado.
- `npm.cmd run smoke:permissions -- --soft --json`: anonimos 401 passaram; matriz autenticada bloqueou por falta de matriz, como esperado.
- `npm.cmd run test:e2e:real`: passou com 1 skipped por falta de opt-in/credenciais, como esperado.
- `npm.cmd run ops:backfill:report -- --soft --json`: bloqueou por falta de `DATABASE_URL`, como esperado.
- `npm.cmd run ops:readiness -- --soft --json`: passou a retornar JSON controlado em modo soft.
- `node --check scripts/portal-real-homologation.mjs`: passou.
- `npm.cmd run homologation:real -- --help`: passou.
- `npm.cmd run homologation:real -- --soft --json`: passou com smoke publico e blockers autenticados.

### Riscos ou pendencias

- Ainda falta executar com credenciais reais, matriz real, `DATABASE_URL` real e acesso ao container EasyPanel.
- Ainda faltam upload/download real, restore drill e QA final por perfil.

## 2026-05-26 09:10

### Arquivos alterados

- `portal-sama-api/package.json`
- `portal-sama-api/scripts/verify-operational-backup.js`
- `portal-sama-api/scripts/validate-operational-readiness.js`
- `portal-sama-api/README.md`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `EASYPANEL_DEPLOY.md`

### O que mudou

- Criado `npm run ops:backup:verify` no `portal-sama-api`.
- O script verifica `metadata.json`, status dos passos do backup, SHA-256/tamanho dos artefatos, integridade gzip de `database.sql.gz`, consistencia interna do `storage-manifest.json` e listagem de `storage.tar.gz` quando existir.
- O aviso `backup-rollback` do readiness passou a orientar `ops:backup:create -> ops:backup:verify -> copia externa -> restore drill`.
- O README da API e a documentacao viva foram atualizados para diferenciar verificacao de artefatos de restore drill real.

### Motivo da alteracao

Reduzir a pendencia de backup/rollback com uma verificacao repetivel dos artefatos gerados antes de copiar evidencias ou iniciar restauracao em alvo isolado.

### Impacto esperado

- Operadores conseguem detectar backup incompleto/corrompido antes de sair do container.
- A trilha operacional de backup fica mais auditavel: gerar, verificar, copiar para fora e restaurar em ambiente separado.
- O restore drill continua pendente e obrigatorio; o novo comando nao restaura banco nem storage.

### Testes executados

- `node --check scripts/verify-operational-backup.js`: passou.
- `npm.cmd run ops:backup:verify -- --help`: passou.
- `npm.cmd run ops:backup:create -- --skip-database --storage-path .ai-tests/backup-verify-storage --output-dir .ai-tests/backup-verify-output --include-storage-archive --json`: passou.
- `npm.cmd run ops:backup:verify -- --backup-dir .ai-tests/backup-verify-output/<backup-id> --json`: passou com avisos esperados por banco pulado e restore real pendente.
- `node --check scripts/validate-operational-readiness.js`: passou.
- `npm.cmd run ops:readiness -- --skip-env --skip-database --skip-clamav --storage-path .ai-tests/readiness-storage --soft --json`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run prisma:validate` com `DATABASE_URL` dummy: passou.
- `git diff --check` em `portal-sama-api`: passou.

### Riscos ou pendencias

- Ainda falta executar backup e verificacao no EasyPanel com `DATABASE_URL` e `STORAGE_PRIVATE_PATH` reais.
- Ainda falta copiar os artefatos para fora do container e executar restore drill em alvo isolado.

## 2026-05-26 08:49

### Arquivos alterados

- `portal-sama-web/package.json`
- `portal-sama-web/scripts/portal-permissions-smoke.mjs`
- `portal-sama-web/README.md`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `EASYPANEL_DEPLOY.md`
- `SEGURANCA.md`
- `INVENTARIO_SEGURANCA.md`

### O que mudou

- Criado `npm.cmd run smoke:permissions` no `portal-sama-web`.
- O script `scripts/portal-permissions-smoke.mjs` valida checks anonimos esperando 401 e uma matriz parametrizada de perfis autenticados com status esperados 200/403/401.
- A matriz pode ser passada por `PORTAL_PERMISSION_MATRIX_FILE`, `PORTAL_PERMISSION_MATRIX_JSON`, `--matrix-file` ou `--matrix-json`.
- O formato recomendado usa `usernameEnv` e `passwordEnv` para evitar credenciais em arquivos versionados.
- A saida mascara usuarios e nao imprime senha, access token, refresh token ou CSRF token.

### Motivo da alteracao

Reduzir a pendencia de matriz 401/403/200 por perfil com uma ferramenta operacional repetivel, segura para evidencias e alinhada ao fluxo real de autenticacao da API v2.

### Impacto esperado

- Operadores podem validar permissoes reais por perfil no EasyPanel sem montar requests manuais.
- A documentacao diferencia ferramenta disponivel de execucao real ainda pendente.
- Nao houve mudanca de runtime do frontend nem da API; a alteracao adiciona script operacional e documentacao.

### Testes executados

- `node --check scripts/portal-permissions-smoke.mjs`: passou.
- `npm.cmd run smoke:permissions -- --help`: passou.
- `npm.cmd run smoke:permissions -- --api-url http://127.0.0.1:9/api-v2 --url http://127.0.0.1:9 --origin http://127.0.0.1:9 --timeout 250 --soft --json`: passou com falhas esperadas e exit code 0.
- Smoke positivo contra servidor fake local: passou.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `git diff --check` em `portal-sama-web`: passou.

### Riscos ou pendencias

- Ainda falta executar o smoke contra `https://portal.samacontabil.com.br/api-v2` com usuarios reais por perfil.
- Ainda falta fechar a matriz oficial de rotas/status esperados e cruzar com escopo real/IDOR.

## 2026-05-25 17:54

### Arquivos alterados

- `portal-sama-web/package.json`
- `portal-sama-web/playwright.real.config.ts`
- `portal-sama-web/tests/e2e/real-auth.spec.ts`
- `portal-sama-web/README.md`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `EASYPANEL_DEPLOY.md`
- `SEGURANCA.md`
- `MATRIZ_MIGRACAO_HTML_PARA_REACT.md`

### O que mudou

- Criado `npm.cmd run test:e2e:real` no `portal-sama-web`.
- Adicionada config `playwright.real.config.ts` sem servidor local, com target padrao `https://portal.samacontabil.com.br`.
- Adicionada spec `tests/e2e/real-auth.spec.ts`, bloqueada por `PORTAL_REAL_E2E=1` e credenciais em ambiente.
- A spec cobre login pela UI, Home autenticada, armazenamento local sem chaves sensiveis, refresh cookie fora de `document.cookie`, politica `HttpOnly`/`SameSite`/`Secure` e logout.
- A config real desliga trace, screenshot e video para reduzir risco de vazar credenciais de homologacao.

### Motivo da alteracao

Reduzir a pendencia "Criar Playwright contra homologacao real" com uma suite operacional segura, repetivel e alinhada ao fluxo real do navegador, complementando o smoke HTTP de autenticacao.

### Impacto esperado

- Operadores podem validar login/Home/logout contra o deploy publico usando usuario real de homologacao.
- A suite nao altera runtime do Web/API e fica opt-in para nao executar login real acidentalmente.
- A documentacao passa a separar ferramenta Playwright disponivel de execucao real ainda pendente.

### Testes executados

- `npm.cmd run test:e2e:real`: passou com 1 skipped sem variaveis reais.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e` em `portal-sama-web`: passou com 9 testes e 1 skipped.
- `git diff --check` em `portal-sama-web`: passou.

### Riscos ou pendencias

- Ainda falta executar a suite contra `https://portal.samacontabil.com.br` com usuario real e sem registrar segredos.
- Ainda falta repetir por perfis criticos, cruzar com a matriz 401/403/200 e anexar evidencia sanitizada.

## 2026-05-25 17:37

### Arquivos alterados

- `portal-sama-web/package.json`
- `portal-sama-web/scripts/portal-auth-smoke.mjs`
- `portal-sama-web/README.md`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `EASYPANEL_DEPLOY.md`
- `SEGURANCA.md`
- `INVENTARIO_SEGURANCA.md`

### O que mudou

- Criado `npm.cmd run smoke:auth` no `portal-sama-web`.
- O script `scripts/portal-auth-smoke.mjs` valida `GET /auth/csrf`, `POST /auth/login`, `GET /auth/me`, `POST /auth/refresh`, novo `GET /auth/me` e `POST /auth/logout`.
- O cookie jar do script e mantido apenas em memoria; usuario e mascarado no resumo; senha, access token, refresh token e CSRF token nao sao impressos.
- O smoke valida que o refresh cookie tenha `HttpOnly`, `SameSite` e `Secure` quando a API estiver em HTTPS.

### Motivo da alteracao

Reduzir a pendencia de homologacao de login/refresh/logout/cookies em HTTPS com uma ferramenta repetivel, parametrizada por ambiente e segura para registrar evidencias sem segredos.

### Impacto esperado

- Operadores podem validar a autenticacao real no EasyPanel usando usuario de homologacao.
- A documentacao passa a diferenciar ferramenta disponivel de validacao real ainda pendente.
- Nao houve mudanca de runtime do frontend nem da API; a alteracao adiciona script operacional e documentacao.

### Testes executados

- `node --check scripts/portal-auth-smoke.mjs`: passou.
- `npm.cmd run smoke:auth -- --help`: passou.
- Smoke positivo contra servidor fake local: passou.
- `npm.cmd run smoke:auth -- --api-url http://127.0.0.1:9/api-v2 --url http://127.0.0.1:9 --origin http://127.0.0.1:9 --timeout 250 --soft --json`: passou com falha esperada de credenciais ausentes e exit code 0.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.

### Riscos ou pendencias

- Ainda falta executar o smoke contra `https://portal.samacontabil.com.br/api-v2` com usuario real e sem registrar credenciais.
- Ainda falta repetir por perfis criticos e cruzar com a matriz 401/403/200.

## 2026-05-25 17:19

### Arquivos alterados

- `portal-sama-api/src/modules/documents/documents.controller.ts`
- `portal-sama-api/src/modules/documents/documents.controller.spec.ts`
- `portal-sama-web/src/services/documents.service.ts`
- `portal-sama-web/tests/e2e/smoke.spec.ts`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `SEGURANCA.md`
- `INVENTARIO_ENDPOINTS.md`
- `paginas/onboarding-documentos-cliente.md`

### O que mudou

- `DocumentsController` passou a expor `GET /api-v2/public/onboarding/documents/:token` e `POST /api-v2/public/onboarding/documents/:token/upload`.
- Os endpoints publicos genericos de documentos foram preservados para compatibilidade.
- `portal-sama-web/src/services/documents.service.ts` passou a consumir os aliases especificos de onboarding.
- O smoke Playwright da tela publica foi atualizado para mockar o novo alias.

### Motivo da alteracao

Alinhar o codigo real ao contrato documentado para documentos publicos de onboarding, reduzindo a divergencia entre a rota React `/onboarding/publico/documentos/:token` e o endpoint alvo em `/api-v2/public/onboarding/documents/:token`.

### Impacto esperado

- O frontend publico passa a conversar com a API pelo contrato semantico de onboarding.
- Backends/clients que ainda usem `documents/public-checklist` e `documents/public-upload` continuam funcionando.
- Nao muda schema, migration, storage ou regra de validacao de token; a seguranca permanece centralizada no `DocumentsService`.

### Testes executados

- `npm.cmd test -- documents.controller.spec.ts --runInBand`: passou com 2 testes.
- `npm.cmd run lint` em `portal-sama-api`: passou apos ajuste do mock no spec.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run prisma:validate` com `DATABASE_URL` dummy: passou.
- `npm.cmd run lint` em `portal-sama-web`: passou apos reexecucao isolada.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e -- -g "public documents page"`: passou com 1 teste Chromium.
- `git diff --check` em API e Web: passou.

### Riscos ou pendencias

- Ainda falta validar os aliases publicos contra o EasyPanel com token real/legado.
- Ainda falta Playwright de upload publico real, MySQL/storage/ClamAV reais e evidencia HTTPS.

## 2026-05-25 16:54

### Arquivos alterados

- `portal-sama-api/Dockerfile`
- `portal-sama-api/.env.example`
- `portal-sama-api/package.json`
- `portal-sama-api/README.md`
- `portal-sama-api/scripts/create-operational-backup.js`
- `portal-sama-api/scripts/validate-operational-readiness.js`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `EASYPANEL_DEPLOY.md`
- `INVENTARIO_SEGURANCA.md`

### O que mudou

- Criado `npm run ops:backup:create` na API para gerar artefatos operacionais de backup.
- O backup cria `database.sql.gz` via `mariadb-dump/mysqldump`, `storage-manifest.json`, `metadata.json` com SHA-256 e `storage.tar.gz` opcional.
- O Dockerfile da API passou a instalar `mariadb-client`, mantendo `clamav`.
- O readiness passou a apontar o comando de backup no aviso `backup-rollback`.
- Documentacoes vivas foram atualizadas para registrar que ClamAV/EICAR strict ja passou no container real e que o unico warning atual e restore drill.

### Motivo da alteracao

Depois que o operador confirmou o readiness real aprovado com warning apenas de backup/rollback, a proxima pendencia tecnica implementavel era criar um comando repetivel para gerar artefatos antes de backfills, rollback ou corte do legado.

### Impacto esperado

- O container da API passa a ter um caminho operacional para extrair backup de banco e manifesto de storage.
- O restore drill continua separado e precisa ser executado/documentado fora do container.
- A pendencia de backup sai de "sem ferramenta" para "ferramenta criada, falta prova real de restore".

### Testes executados

- `node --check scripts/create-operational-backup.js`: passou.
- `npm.cmd run ops:backup:create -- --help`: passou.
- `node --check scripts/validate-operational-readiness.js`: passou.
- `npm.cmd run ops:backup:create -- --skip-database --storage-path .ai-tests/backup-storage --output-dir .ai-tests/ops-backups --include-storage-archive`: passou.
- `npm.cmd run prisma:validate` com `DATABASE_URL` dummy: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd run build`: passou.
- `npm.cmd run test -- --runInBand`: passou com 30 suites/165 testes.
- `git diff --check`: passou com avisos LF/CRLF esperados no Windows.
- `docker build --pull=false -t portal-sama-api:backup-runtime .`: passou.
- `docker run --rm --entrypoint sh portal-sama-api:backup-runtime -lc "which mariadb-dump || which mysqldump"`: passou e retornou `/usr/bin/mariadb-dump`.
- `docker run --rm --entrypoint sh portal-sama-api:backup-runtime -lc "npm run ops:backup:create -- --help"`: passou.

### Riscos ou pendencias

- Ainda falta executar `npm run ops:backup:create` no EasyPanel com o `DATABASE_URL` real.
- Ainda falta copiar os artefatos para fora do container, armazenar fora do volume da aplicacao e provar restore drill.
- Backfills, validacao funcional por perfil, Playwright real, QA final e desligamento seguro do legado continuam pendentes.

## 2026-05-25 16:31

### Arquivos alterados

- `portal-sama-api/Dockerfile`
- `portal-sama-api/.env.example`
- `portal-sama-api/package.json`
- `portal-sama-api/README.md`
- `portal-sama-api/scripts/validate-operational-readiness.js`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `EASYPANEL_DEPLOY.md`
- `INVENTARIO_SEGURANCA.md`

### O que mudou

- A imagem runtime da API passou a instalar `clamav` via `apk add --no-cache clamav`.
- O Dockerfile define `SAMA_UPLOAD_SCAN_BIN=/usr/bin/clamscan` e `SAMA_CLAMAV_UPDATE_ON_START=false` como defaults.
- Criado `npm run ops:clamav:update`, que executa `freshclam` dentro do container para baixar/atualizar assinaturas.
- `ops:readiness` ganhou hint operacional quando o scanner nao existe ou quando ClamAV abre mas nao detecta EICAR.
- README e docs de deploy passaram a orientar: redeploy -> `npm run ops:clamav:update` -> `npm run ops:readiness`.

### Motivo da alteracao

O operador executou corretamente `npm run ops:readiness` no console real do `portal-sama-api`. A validacao provou banco, migrations, RBAC, usuarios e storage, mas falhou em `clamav-eicar` porque a imagem anterior nao continha `clamscan/clamdscan`.

### Impacto esperado

- A proxima imagem da API tera o scanner ClamAV disponivel no container.
- A falha esperada apos instalar o scanner, se existir, passa a ser apenas assinatura ausente/desatualizada, corrigivel com `npm run ops:clamav:update`.
- O readiness strict deve conseguir validar EICAR apos update de assinaturas.

### Testes executados

- `node --check scripts/validate-operational-readiness.js`: passou.
- `npm.cmd run ops:readiness -- --skip-env --skip-database --skip-clamav --storage-path .ai-tests/readiness-storage --soft`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd run build`: passou.
- `docker build --pull=false -t portal-sama-api:clamav-runtime .`: passou.
- `docker run --rm portal-sama-api:clamav-runtime sh -lc "which clamscan && which freshclam && which clamdscan && clamscan --version"`: passou e mostrou `ClamAV 1.4.4`.
- `docker run --rm portal-sama-api:clamav-runtime npm run ops:readiness -- --skip-env --skip-database --storage-path /tmp/portal-sama-readiness --soft`: passou com warning esperado de assinaturas ausentes antes do `freshclam`.

### Riscos ou pendencias

- Ainda falta redeployar o `portal-sama-api` no EasyPanel com a imagem nova.
- Ainda falta rodar `npm run ops:clamav:update` no container real; `freshclam` depende de rede/DNS e pode sofrer rate limit do mirror.
- Depois do update, repetir `npm run ops:readiness` sem skips.

## 2026-05-25 16:18

### Arquivos alterados

- `portal-sama-api/package.json`
- `portal-sama-api/README.md`
- `portal-sama-api/scripts/validate-operational-readiness.js`
- `portal-sama-api/scripts/backfill-readiness-report.js`
- `portal-sama-api/src/modules/transfers/transfers.service.spec.ts`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `EASYPANEL_DEPLOY.md`

### O que mudou

- Criado `npm run ops:readiness` na API para validar ambiente, banco, migrations, RBAC, usuarios ADMIN/DEV, storage privado e ClamAV/EICAR.
- Criado `npm run ops:backfill:report` na API para gerar relatorio read-only de tabelas atuais/legadas, contagens, gaps de usuarios/roles e vinculos de cliente.
- Documentado o uso dos comandos no README da API e no guia do EasyPanel.
- Corrigido `transfers.service.spec.ts` congelando o relogio em `2026-05-19T12:00:00.000Z`, removendo uma falha dependente da data atual.
- Atualizadas as documentacoes vivas para marcar as ferramentas como implementadas e a execucao real sem skips como pendente.

### Motivo da alteracao

Dar continuidade aos itens pendentes de backfills, validacao por perfil/RBAC, storage/ClamAV em fluxo real, backups/rollback e desligamento seguro do legado, criando evidencias operacionais repetiveis antes de qualquer corte em producao.

### Impacto esperado

- O container da API passa a ter comandos claros para homologacao profunda no EasyPanel.
- Backfills podem ser planejados com relatorio read-only antes de qualquer migracao destrutiva.
- Falhas de seed RBAC, roles sem permissoes, usuarios ativos sem role, ClamAV ausente ou storage mal configurado devem aparecer mais cedo.

### Testes executados

- `node --check scripts/validate-operational-readiness.js`: passou.
- `node --check scripts/backfill-readiness-report.js`: passou.
- `npm.cmd run ops:readiness -- --help`: passou.
- `npm.cmd run ops:readiness -- --skip-env --skip-database --skip-clamav --storage-path .ai-tests/readiness-storage --soft`: passou com warning esperado de backup/rollback.
- `npm.cmd run ops:backfill:report -- --help`: passou.
- `npm.cmd run ops:backfill:report -- --soft` com `DATABASE_URL` apontando para `127.0.0.1:9`: falhou por conexao esperada sem quebrar o processo.
- `npm.cmd run test -- transfers.service.spec.ts --runInBand`: passou.
- `npm.cmd run test -- --runInBand`: passou com 30 suites/165 testes.
- `npm.cmd run lint`: passou.
- `npm.cmd run build`: passou.
- `npm.cmd run prisma:validate` com `DATABASE_URL` dummy: passou.

### Riscos ou pendencias

- Ainda falta rodar `npm run ops:readiness` sem skips no container real do EasyPanel.
- Ainda falta rodar `npm run ops:backfill:report -- --json` no MySQL real e anexar a evidencia.
- Os comandos nao substituem backfill, restore drill, QA por perfil, Playwright real ou desligamento seguro do legado.

## 2026-05-25 11:46

### Arquivos alterados

- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`
- `EASYPANEL_DEPLOY.md`
- `README_DOCUMENTACAO.md`
- `INCIDENTE_SUBDOMINIO_PORTAL_SAMACONTABIL.md`
- `INVENTARIO_SEGURANCA.md`

### O que mudou

- Atualizado o estado real: Bitbucket ja possui repositorios, EasyPanel ja esta com API/database/Web online e `https://portal.samacontabil.com.br/` esta ativo.
- Registrados nomes reais observados: `portal-sama-api`, `portal-sama-database`, `portal-sama-web` e repo de documentacao remoto `portal-sama-doc`.
- Ajustado o painel de deploy para remover como bloqueio principal a criacao de repositorios, publicacao no EasyPanel, dominio HTTPS, proxy `/api-v2`, health publico e smoke publico.
- Atualizada a estimativa operacional para 92% de prontidao para iniciar/continuar homologacao e 73% para producao sem legado.
- Registrado que o banco real esta conectado, usuarios iniciais foram criados pelo operador e o health publico retornou `database=up`/`storage=up`.

### Motivo da alteracao

Sincronizar a documentacao com o estado real informado pelo operador e validado por smoke HTTP publico, evitando que os documentos continuem tratando Bitbucket/EasyPanel/banco/dominio como pendencias ja resolvidas.

### Impacto esperado

- Proximos ciclos deixam de focar criacao de infraestrutura e passam a focar homologacao funcional, permissoes reais, backfills, storage/ClamAV, backups/rollback e QA.
- O smoke publico passa a ser evidencia registrada de que dominio, proxy, CORS, CSRF, API, banco e storage estao respondendo.

### Testes executados

- `npm.cmd run smoke:public` em `portal-sama-web`: passou sem `--soft`.
- `curl.exe -sS https://portal.samacontabil.com.br/api-v2/health`: retornou `ok=true`, `database=up`, `storage=up`.
- `curl.exe -sS -D - -o NUL -H "Origin: https://portal.samacontabil.com.br" https://portal.samacontabil.com.br/api-v2/auth/csrf`: retornou 200, CORS esperado, HSTS/CSP e cookie CSRF `Secure; SameSite=Lax`.

### Riscos ou pendencias

- Confirmar e registrar migrations/seed/bootstrap ou procedimento equivalente usado para usuarios iniciais.
- Validar matriz de permissoes por perfil, backfills, storage persistente, ClamAV strict, backups/rollback, Playwright real e QA visual final.
- Enviar esta atualizacao documental ao remoto de documentacao.

## 2026-05-25 11:32

### Arquivos alterados

- `portal-sama-api/src/modules/health/health.controller.ts`
- `portal-sama-api/src/modules/health/health.module.ts`
- `portal-sama-api/src/modules/health/health.service.ts`
- `portal-sama-api/src/modules/health/health.service.spec.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- `portal-sama-api/README.md`
- `STATUS_IMPLEMENTACAO.md`
- `CHANGELOG_TECNICO.md`
- `RELATORIO_TESTES.md`
- `PENDENCIAS_TECNICAS.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `EASYPANEL_DEPLOY.md`
- `INVENTARIO_SEGURANCA.md`

### O que mudou

- `GET /api-v2/health` passou a validar banco e storage de forma operacional.
- O health retorna `database=up/down/not_checked` e `storage=up/down/not_configured`.
- Quando `PRISMA_CONNECT_ON_BOOT=true`, a API executa `SELECT 1` via Prisma; quando `false`, mantem `database=not_checked`.
- O storage configurado em `STORAGE_PRIVATE_PATH` e criado/verificado com permissao de leitura/escrita.
- O controller responde HTTP 503 quando algum item critico fica indisponivel.
- O E2E foi alinhado ao novo `storage=up` e a mensagem atual de token publico obrigatorio para documentos.

### Motivo da alteracao

Reduzir a pendencia documentada de health real antes da homologacao, permitindo que EasyPanel/smoke detectem indisponibilidade de MySQL ou storage privado em vez de apenas confirmar que variaveis existem.

### Impacto esperado

- O smoke publico passa a ter um sinal mais confiavel de API pronta.
- Falhas de banco/storage devem aparecer cedo em homologacao.
- Ambientes de teste continuam podendo evitar conexao real com banco usando `PRISMA_CONNECT_ON_BOOT=false`.

### Testes executados

- `npm.cmd test -- health.service.spec.ts --runInBand`: passou com 4 testes.
- `npm.cmd run test:e2e -- --runInBand`: passou com 136 testes.
- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run prisma:validate` em `portal-sama-api`: passou.
- `git diff --check` em `portal-sama-api`: passou.

### Riscos ou pendencias

- Health real ainda nao foi executado contra o MySQL/volume do EasyPanel.
- Se `STORAGE_PRIVATE_PATH` apontar para caminho errado mas gravavel dentro do container, o health confirma escrita local, nao a persistencia do volume; a montagem real ainda precisa ser conferida no EasyPanel.

## 2026-05-25 11:20

### Arquivos alterados

- `portal-sama-web/package.json`
- `portal-sama-web/scripts/portal-public-smoke.mjs`
- `portal-sama-web/README.md`
- `STATUS_IMPLEMENTACAO.md`
- `CHANGELOG_TECNICO.md`
- `RELATORIO_TESTES.md`
- `PENDENCIAS_TECNICAS.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `EASYPANEL_DEPLOY.md`

### O que mudou

- Criado smoke publico no repo separado `portal-sama-web`.
- Exposto `npm.cmd run smoke:public` no `package.json` do Web.
- O script valida frontend publico, `/api-v2/health`, preflight CORS de `/api-v2/auth/me` e CSRF em `/api-v2/auth/csrf`.
- O README do Web passou a documentar uso, variaveis de ambiente e modo `--soft`.
- A documentacao de deploy foi ajustada para deixar claro que o comando deve ser rodado no repo separado do Web.

### Motivo da alteracao

Fechar a divergencia entre a documentacao de EasyPanel, que ja previa `npm.cmd run smoke:public`, e o estado real do workspace separado `portal-sama-web`, que ainda nao expunha esse comando.

### Impacto esperado

- Homologacao passa a ter uma checagem repetivel para dominio, proxy `/api-v2`, CORS e cookie CSRF.
- O smoke pode ser usado em diagnostico com `--soft` e em aprovacao final sem `--soft`.
- Nao altera runtime do frontend nem da API; adiciona apenas ferramenta operacional.

### Testes executados

- `node --check scripts/portal-public-smoke.mjs`: passou.
- Smoke com servidor HTTP fake local: passou nos quatro checks.
- `npm.cmd run smoke:public -- --url http://127.0.0.1:9 --api-url http://127.0.0.1:9/api-v2 --origin http://127.0.0.1:9 --timeout 250 --soft`: passou com falhas esperadas e exit code 0.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `git diff --check` em `portal-sama-web`: passou.
- `git diff --check` em `portal-sama-docs`: passou.

### Riscos ou pendencias

- O smoke real contra `https://portal.samacontabil.com.br` nao foi executado sem `--soft`, pois API/Web ainda precisam ser publicados/validados no EasyPanel.
- Banco MySQL real, HTTPS, CORS/cookies em navegador e usuarios/permissoes reais continuam pendentes de homologacao.

## 2026-05-25 11:09

### Arquivos alterados

- `portal-sama-api/README.md`
- `portal-sama-web/README.md`
- `STATUS_IMPLEMENTACAO.md`
- `CHANGELOG_TECNICO.md`
- `RELATORIO_TESTES.md`
- `PENDENCIAS_TECNICAS.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `EASYPANEL_DEPLOY.md`

### O que mudou

- Criado README minimo no repo separado da API.
- Reforcado README do repo separado do Web.
- Ambos os READMEs apontam para `portal-sama-docs` como fonte de verdade antes de alterar API/Web.
- Registrados comandos principais, cuidados de deploy e itens que nao devem ser versionados.
- O painel de prontidao marcou como concluida a regra operacional de leitura do repo de docs antes de API/Web.
- O guia de EasyPanel registrou a existencia dos READMEs minimos nos repos tecnicos separados.

### Motivo da alteracao

Fechar o ponto operacional documentado apos a separacao em tres repositorios: evitar que uma IA ou pessoa desenvolvedora comece por `portal-sama-api` ou `portal-sama-web` sem ler o contexto vigente em `portal-sama-docs`.

### Impacto esperado

- Melhora a continuidade entre repositorios separados.
- Reduz risco de alteracoes tecnicas fora do contexto documental.
- Nao muda comportamento de runtime da API nem do frontend.

### Testes executados

- `git diff --check` em `portal-sama-api`: passou.
- `git diff --check` em `portal-sama-web`: passou.
- `git diff --check` em `portal-sama-docs`: passou.

### Riscos ou pendencias

- Lint/build nao foram executados porque nao houve alteracao de codigo.
- Ainda faltam remotes Bitbucket, push dos repos, configuracao EasyPanel e validacao real com MySQL/HTTPS.

## 2026-05-25 10:51

### Arquivos alterados

- `portal-sama-web/tests/e2e/smoke.spec.ts`
- `STATUS_IMPLEMENTACAO.md`
- `CHANGELOG_TECNICO.md`
- `RELATORIO_TESTES.md`
- `PENDENCIAS_TECNICAS.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`

### O que mudou

- O smoke E2E do frontend ganhou helper compartilhado para resposta de sessao autenticada.
- Foi adicionado mock de login de primeiro acesso com `welcomeAnimationSeen=false`.
- Foi adicionado teste mobile do login, cobrindo overflow, logo da lateral, texto manuscrito e painel.
- Foi adicionado teste da intro `welcome` apos login, cobrindo logo centralizada, texto dentro do viewport e `mask-image` nas linhas laterais.

### Motivo da alteracao

Depois da correcao visual aprovada no deploy, a pendencia local mais util era transformar o smoke manual da intro/login em cobertura versionada, reduzindo risco de regressao no proximo ajuste do frontend.

### Impacto esperado

- O item de E2E local do Web no workspace separado passa a ter evidencia automatizada recente.
- Regressao de alinhamento mobile do login e da welcome deve falhar na suite antes de chegar ao deploy.
- A validacao real de EasyPanel/HTTPS segue separada, pois estes testes usam mocks locais.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e` em `portal-sama-web`: passou com 9 testes Chromium.
- `git diff --check` em `portal-sama-web`: passou.

### Riscos ou pendencias

- O E2E continua mockado e nao substitui Playwright contra homologacao real.
- QA visual desktop/mobile no EasyPanel ainda precisa confirmar cache, assets e CSS servidos pelo deploy.
- O aviso conhecido do React Router sobre `HydrateFallback` segue aparecendo no web server do Playwright sem quebrar os testes.

## 2026-05-25 10:33

### Arquivos alterados

- `portal-sama-web/src/features/intro/components/PortalSamaLayer.tsx`
- `portal-sama-web/src/features/intro/hooks/usePortalSamaGsapTimeline.ts`
- `portal-sama-web/src/features/intro/styles/portal-sama-intro.css`
- `portal-sama-web/src/index.css`
- `STATUS_IMPLEMENTACAO.md`
- `CHANGELOG_TECNICO.md`
- `RELATORIO_TESTES.md`
- `PENDENCIAS_TECNICAS.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`

### O que mudou

- A centralizacao da logo e do texto da intro deixou de depender da propriedade CSS individual `translate`.
- `PortalSamaLayer` passou a declarar percentuais de ancoragem via `data-intro-x-percent` e `data-intro-y-percent`.
- A timeline GSAP aplica esses percentuais depois de limpar `transform`, preservando o centro durante `scale`, `rotation` e `y`.
- As linhas laterais da cena `welcome` receberam `mask-image`/`-webkit-mask-image` para reforcar o esmaecimento antes do centro.
- O login incorporou o deslocamento vertical das camadas de logo nos keyframes com `translateY(...)`.
- A welcome em telas estreitas pode quebrar em duas linhas para evitar corte.

### Motivo da alteracao

No deploy, a cena de introducao podia aparecer com logo e texto deslocados para baixo/direita quando o navegador/cache nao aplicava `translate`. O sintoma era a frase de boas-vindas saindo pela direita e as camadas da logo desalinhadas.

### Impacto esperado

- A intro fica centralizada de forma mais compativel no deploy.
- O primeiro login (`welcome`) preserva o texto e as linhas laterais sem invadir o centro.
- A lateral do login continua alinhada sem depender de propriedades CSS individuais de transform.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `git diff --check` em `portal-sama-web`: passou.
- `rg "\btranslate\s*:" src public -S` em `portal-sama-web`: sem ocorrencias.
- Smoke Playwright manual em `/dev/intro-preview` e `/login` com viewports `1920x900` e `390x844`: sem overflow horizontal e com logo/texto centralizados.
- Checagem extra Playwright `320x720`: texto da welcome sem `scrollWidth` maior que `clientWidth`.

### Riscos ou pendencias

- Ainda falta validar no EasyPanel apos novo deploy, em navegador real, com cache limpo ou versao nova do asset/CSS.
- QA visual final de homologacao continua pendente para aprovar timing e intensidade visual.

## 2026-05-25 09:48

### Arquivos alterados

- `portal-sama-api/src/config/env.schema.ts`
- `portal-sama-api/src/config/env.schema.spec.ts`
- `EASYPANEL_DEPLOY.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `STATUS_IMPLEMENTACAO.md`
- `CHANGELOG_TECNICO.md`
- `RELATORIO_TESTES.md`
- `PENDENCIAS_TECNICAS.md`

### O que mudou

- `DATABASE_URL` passou a ser validada pela configuracao da API antes do Prisma Client tentar conectar.
- URLs MySQL malformadas agora recebem erro direto sobre formato, porta e nome do banco.
- A documentacao do EasyPanel ganhou o diagnostico do Prisma `P1013` com `invalid port number in database URL`.

### Motivo da alteracao

O deploy chegou ao start da API, mas o Prisma falhou com `P1013` porque a string de conexao do banco estava malformada. Esse erro nao e de migration nem de seed; ele acontece antes da conexao real com o MySQL.

### Impacto esperado

- O log da API passa a apontar a configuracao incorreta de forma mais clara.
- A equipe consegue corrigir `DATABASE_URL` no EasyPanel sem recriar o banco.
- Senhas com caracteres especiais ficam documentadas como dependentes de URL encode.

### Testes executados

- `npm.cmd test -- env.schema.spec.ts --runInBand` em `portal-sama-api`: passou.
- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run prisma:validate` em `portal-sama-api`: passou.

### Riscos ou pendencias

- Ainda falta corrigir a variavel `DATABASE_URL` real no EasyPanel e redeployar.
- Depois do start da API, ainda faltam seed, bootstrap admin e validacoes funcionais do ambiente.

## 2026-05-25 09:25

### Arquivos alterados

- `portal-sama-api/package.json`
- `portal-sama-api/scripts/run-prisma-runtime-script.js`
- `STATUS_IMPLEMENTACAO.md`
- `CHANGELOG_TECNICO.md`
- `RELATORIO_TESTES.md`
- `PENDENCIAS_TECNICAS.md`
- `EASYPANEL_DEPLOY.md`

### O que mudou

- `npm run prisma:seed` e `npm run prisma:bootstrap-admin` passaram a usar um wrapper de runtime.
- No container Docker, o wrapper executa `dist/prisma/seed.js` ou `dist/prisma/bootstrap-admin.js`.
- Em ambiente local sem `dist`, o wrapper continua executando `prisma/seed.ts` ou `prisma/bootstrap-admin.ts` via `ts-node`.

### Motivo da alteracao

No EasyPanel, `npm run prisma:seed` rodava `ts-node prisma/seed.ts`, mas a imagem de runtime nao copia `src/`. Como `seed.ts` importa `../src/modules/rbac/default-rbac`, o comando falhava antes de chegar ao banco.

### Impacto esperado

- A imagem Docker consegue rodar `npm run prisma:seed` sem precisar copiar `src/`.
- O bootstrap administrativo tambem fica preparado para rodar a partir do JS compilado.
- Quem estiver em um container antigo pode usar temporariamente `node dist/prisma/seed.js` ate redeployar a imagem nova.

### Testes executados

- `node --check scripts/run-prisma-runtime-script.js`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run prisma:validate` em `portal-sama-api`: passou.
- `npm.cmd run prisma:seed` com `DATABASE_URL=mysql://portal_user:portal_password@127.0.0.1:9/portal_sama_test`: falhou por conexao recusada ao banco, confirmando que chegou em `dist/prisma/seed.js` e nao falhou mais por TypeScript/import.

### Riscos ou pendencias

- Docker Desktop local nao estava ativo, entao o rebuild Docker local nao foi repetido.
- Ainda falta redeployar a API no EasyPanel para que `npm run prisma:seed` use o wrapper novo.

## 2026-05-22 16:38

### Arquivos alterados

- `portal-sama-api/Dockerfile`
- `portal-sama-api/.env.example`
- `portal-sama-api/package.json`
- `portal-sama-api/scripts/prisma-baseline-existing-database.js`
- `portal-sama-api/prisma/migrations/20260501000000_baseline_existing_database/migration.sql`
- `EASYPANEL_DEPLOY.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `BANCO_DADOS_MYSQL_PRISMA.md`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`

### O que mudou

- O start Docker da API deixou de executar `prisma migrate deploy` por padrao.
- Adicionada variavel operacional `PRISMA_MIGRATE_ON_START=false` para evitar que `P3005` derrube o container antes de abrir console no EasyPanel.
- Corrigido o entrypoint de producao para `node dist/src/main.js`, caminho real gerado pelo build Nest atual.
- Adicionada migration vazia `20260501000000_baseline_existing_database` para representar o baseline de bancos legados existentes.
- Criado `npm run prisma:migrate:baseline-existing`, protegido por confirmacao via `SAMA_PRISMA_BASELINE_EXISTING_DATABASE=true`, para marcar a baseline e entao rodar `prisma migrate deploy`.
- Documentado o fluxo separado para banco limpo e banco existente com `P3005`.

### Motivo da alteracao

O EasyPanel mostrou `P3005` porque `banco-sama` nao esta vazio e ainda nao possui historico Prisma. Rodar migrations no start impedia abrir o console do container para executar baseline com backup.

### Impacto esperado

- O servico `portal-sama-api` passa a subir sem tentar migration automatica.
- A equipe consegue abrir console/one-off command e preparar o banco existente de forma controlada.
- Bancos limpos continuam podendo usar `npx prisma migrate deploy` diretamente.

### Testes executados

- `npm.cmd run prisma:generate` em `portal-sama-api`: passou.
- `npm.cmd run prisma:validate` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run lint` em `portal-sama-api`: passou.
- `node --check scripts/prisma-baseline-existing-database.js`: passou.
- `npm.cmd run prisma:migrate:baseline-existing` sem confirmacao: falhou como esperado.
- `docker build --pull=false -t portal-sama-api:prisma-p3005-fix .` em `portal-sama-api`: passou.
- `docker run --rm portal-sama-api:prisma-p3005-fix` sem `DATABASE_URL`: falhou como esperado com mensagem explicita.
- Smoke Docker com variaveis minimas e `PRISMA_CONNECT_ON_BOOT=false`: API iniciou e registrou `Nest application successfully started`.
- `git diff --check` em `portal-sama-api` e `portal-sama-docs`: passou com avisos LF/CRLF esperados no Windows.

### Riscos ou pendencias

- Ainda falta executar backup/snapshot real do `banco-sama`.
- O baseline controlado ainda precisa ser executado uma unica vez no EasyPanel.
- Se o banco ja tiver tabelas Prisma parcialmente criadas por tentativa anterior, parar e revisar antes de forcar novos `migrate resolve`.

## 2026-05-22 16:08

### Arquivos alterados

- `portal-sama-api/Dockerfile`
- `EASYPANEL_DEPLOY.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`

### O que mudou

- O build da API deixou de depender da `DATABASE_URL` real para executar `prisma generate`.
- Adicionado `PRISMA_GENERATE_DATABASE_URL` dummy no stage `builder` do Dockerfile.
- O start do container agora falha com mensagem direta se `DATABASE_URL` nao estiver configurada no runtime.
- As docs de EasyPanel passaram a registrar a correcao para o erro Prisma `P1012`.

### Motivo da alteracao

Desbloquear o deploy da API no EasyPanel quando o Prisma valida `env("DATABASE_URL")` antes de o console/runtime estar acessivel.

### Impacto esperado

- Build da imagem deixa de quebrar apenas por falta de URL real durante `prisma generate`.
- Runtime continua protegido: migrations e aplicacao so iniciam com `DATABASE_URL` real configurada no servico `portal-sama-api`.

### Testes executados

- `npm.cmd run prisma:generate` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run prisma:validate` em `portal-sama-api`: passou.
- `docker build --pull=false -t portal-sama-api:prisma-env-fix .` em `portal-sama-api`: passou sem `.env` no contexto.
- `docker run --rm portal-sama-api:prisma-env-fix` sem `DATABASE_URL`: falhou como esperado com mensagem explicita de configuracao.
- `git diff --check` em `portal-sama-docs`: passou.
- `git diff --check` em `portal-sama-api`: passou com avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Ainda falta configurar a `DATABASE_URL` real no EasyPanel e fazer novo deploy da API.
- Ainda falta validar `prisma migrate deploy`, seed, bootstrap admin, health, csrf e login real no MySQL do EasyPanel.

## 2026-05-22 15:22

### Arquivos alterados

- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `CHANGELOG_TECNICO.md`

### O que mudou

- Registrada validacao de lint/build nos repos separados.
- Atualizado o percentual operacional para 76% de prontidao para homologacao e 64% para producao sem legado.
- Marcados como concluidos os itens de lint/build local separados; E2E separado permanece pendente.

### Motivo da alteracao

Confirmar que a separacao fisica dos workspaces nao quebrou os builds locais antes de publicar no Bitbucket.

### Impacto esperado

- Menor risco ao conectar `portal-sama-api` e `portal-sama-web` no EasyPanel.
- Proximo bloqueio passa a ser remoto/deploy real, nao build local.

### Testes executados

- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run prisma:validate` em `portal-sama-api`: passou.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.

### Riscos ou pendencias

- Ainda falta criar remotes Bitbucket e fazer push.
- Ainda falta executar E2E/Playwright no repo separado e validar em EasyPanel com MySQL/HTTPS reais.

## 2026-05-22 15:16

### Arquivos alterados

- `portal-sama-docs/.gitattributes`
- `portal-sama-api/.gitattributes`
- `portal-sama-web/.gitattributes`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `RELATORIO_TESTES.md`
- `design/intro-gsap-V2/Prompt-codex.md`
- `design/intro-gsap-V2/README.md`
- `design/intro-gsap-V2/documentacao_refatoracao_intro_portal_sama_gsap.md`
- `design/intro-gsap-V2/portal_sama_intro_codex_handoff.md`
- `integracao_acessorias_entregas_vencimentos.md`
- `portal-sama-api/prisma/migrations/20260513092300_add_contracts/migration.sql`

### O que mudou

- Criada normalizacao de quebras de linha nos tres workspaces separados.
- Atualizado o painel de deploy/producao com o estado real: Git local em `main`, remotes Bitbucket ainda pendentes.
- Ajustada a estimativa operacional para 74% de prontidao para homologacao e 63% para producao sem legado.
- Removidas linhas em branco extras no fim de arquivos herdados que impediam o `git diff --cached --check` limpo.

### Motivo da alteracao

Evitar churn de LF/CRLF no Windows antes dos commits iniciais e deixar a documentacao refletindo a separacao local ja realizada.

### Impacto esperado

- Commits iniciais mais estaveis para Bitbucket/EasyPanel.
- Menor risco de versionar artefatos locais ou sofrer diferencas de quebra de linha em ambiente Linux.

### Testes executados

- `git branch --show-current` nos tres workspaces: `main`.
- `git status --short --ignored` nos tres workspaces: ignorados esperados confirmados.
- Busca textual por espacos finais em arquivos Markdown/configuracao no repo de docs: sem ocorrencias.
- `git diff --cached --check` nos tres workspaces: passou.

### Riscos ou pendencias

- Ainda falta criar os repositorios remotos no Bitbucket e executar o push.
- Ainda falta rodar lint/build nos repos separados antes da homologacao no EasyPanel.

## 2026-05-22 15:10

### Arquivos alterados

- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `INSTRUÇÕEDS_AI.MD`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`
- `CHANGELOG_TECNICO.md`
- `RELATORIO_TESTES.md`
- `portal-sama-docs/.gitignore`
- `portal-sama-docs/README.md`
- `portal-sama-api/.gitignore`
- `portal-sama-web/.gitignore`

### O que mudou

- Confirmada a separacao local dos tres workspaces.
- Git foi inicializado localmente em `portal-sama-docs`, `portal-sama-api` e `portal-sama-web`.
- Criado `.gitignore` minimo para API e Docs.
- Reforcado `.gitignore` do Web para ignorar `.env`, artefatos locais e Playwright.
- Criado `README.md` inicial no repositorio de documentacao.

### Motivo da alteracao

Preparar os tres workspaces para virarem repositorios Bitbucket sem risco de versionar segredos ou artefatos pesados.

### Impacto esperado

- Os repositorios locais ficam prontos para receber commits iniciais e remotes Bitbucket.
- `.env`, `node_modules`, `dist`, `playwright-report` e `test-results` ficam fora do versionamento.

### Testes executados

- `git status --short --ignored` em `portal-sama-docs`: confirmou arquivos documentais e ignorados esperados.
- `git status --short --ignored` em `portal-sama-api`: confirmou `.env`, `dist/` e `node_modules/` ignorados.
- `git status --short --ignored` em `portal-sama-web`: confirmou `dist/`, `node_modules/`, `playwright-report/` e `test-results/` ignorados.

### Riscos ou pendencias

- Ainda falta criar os repositorios remotos no Bitbucket e executar `git remote add origin`.
- Ainda falta validar os builds a partir dos repos separados depois do push.

## 2026-05-22 14:57

### Arquivos alterados

- `docs/AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `docs/INSTRUÇÕEDS_AI.MD`
- `docs/README_DOCUMENTACAO.md`
- `docs/ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md`
- `docs/EASYPANEL_DEPLOY.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/CHANGELOG_TECNICO.md`

### O que mudou

- Formalizada a topologia oficial de tres repositorios: `portal-sama-docs`, `portal-sama-api` e `portal-sama-web`.
- `portal-sama-docs` foi definido como fonte de verdade da documentacao e leitura obrigatoria antes de alterar API ou Web.
- O painel de deploy passou a listar estrutura esperada do repositorio de documentacao, regra de leitura entre repos e ordem segura de homologacao com tres repositorios.
- Registrada ADR-0030 para a decisao de documentacao centralizada.

### Motivo da alteracao

Preparar a separacao operacional para Bitbucket/EasyPanel sem perder continuidade documental entre os workspaces.

### Impacto esperado

- A IA e pessoas desenvolvedoras passam a ter um ponto unico de contexto antes de mexer em qualquer workspace tecnico.
- API e Web podem ser publicados no EasyPanel como repos menores, enquanto a documentacao fica versionada separadamente.

### Testes executados

- `git diff --check`: passou, com avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- A separacao fisica dos repositorios ainda nao foi executada.
- Ainda e ponto a validar se API/Web terao README minimo apontando para `portal-sama-docs`.

## 2026-05-22 14:18

### Arquivos alterados

- `docs/AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
- `docs/INSTRUÇÕEDS_AI.MD`
- `docs/README_DOCUMENTACAO.md`
- `docs/EASYPANEL_DEPLOY.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/CHANGELOG_TECNICO.md`

### O que mudou

- O arquivo `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD` foi reestruturado como painel obrigatorio de prontidao para homologacao/producao.
- Registrado o percentual operacional atual: 72% para iniciar homologacao da nova stack e 62% para producao sem dependencia do legado.
- Documentada a separacao dos workspaces em repositorios Bitbucket: `portal-sama-api` e `portal-sama-web`.
- Documentados os requisitos de EasyPanel, banco `banco-sama`, usuario MySQL nao-root, migrations, seed, bootstrap admin, storage, ClamAV, backfills, Playwright real e QA visual.
- O prompt mestre e o indice de documentacao agora apontam esse arquivo como leitura/atualizacao obrigatoria em ciclos de deploy, progresso e producao.
- Registrada ADR-0029 para formalizar o painel obrigatorio de prontidao para producao.

### Motivo da alteracao

Transformar o documento criado pelo usuario em fonte oficial e atualizavel do que falta para terminar o projeto, evitando que porcentagem de andamento e bloqueios de producao fiquem apenas no historico do chat.

### Impacto esperado

- Proximos ciclos de trabalho passam a ter um checklist unico para deploy/producao.
- A separacao Bitbucket/EasyPanel fica documentada antes de executar mudancas operacionais.
- A equipe ganha um ponto de controle claro para saber se a nova stack ainda depende do legado.

### Testes executados

- `git diff --check`: passou, com avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Nenhum teste automatizado de API/Web foi executado porque a alteracao foi apenas documental.
- Ainda falta executar a separacao real dos repositorios Bitbucket e a homologacao no EasyPanel.

## 2026-05-22 14:05

### Arquivos alterados

- `portal-sama-api/prisma/seed.ts`
- `portal-sama-api/prisma/bootstrap-admin.ts`
- `portal-sama-api/package.json`
- `portal-sama-api/.env.example`
- `compose.easypanel.env.example`
- `docs/EASYPANEL_DEPLOY.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`

### O que mudou

- O seed RBAC foi refatorado para exportar `seedRbac`, mantendo `npm run prisma:seed` sem criar usuarios/senhas.
- Criado `npm run prisma:bootstrap-admin`, que usa variaveis `SAMA_BOOTSTRAP_ADMIN_*` para criar ou atualizar o primeiro usuario administrativo da API v2 em banco limpo.
- O bootstrap vincula role `DEV` ou `ADMIN`, mantem o usuario ativo, registra auditoria e preserva senha existente salvo `SAMA_BOOTSTRAP_ADMIN_RESET_PASSWORD=true`.
- Documentado o caminho de EasyPanel com dois repositorios (`portal-sama-api` e `portal-sama-web`) e o banco existente `banco-sama` no host interno `portal-sama_database:3306`.

### Motivo da alteracao

Permitir homologacao da nova stack sem depender da aplicacao antiga para o primeiro login, mantendo credenciais fora do Git e separando dados estruturais de RBAC do bootstrap operacional de ambiente.

### Impacto esperado

- Um banco novo do EasyPanel pode receber migrations, seed RBAC e primeiro admin da API v2 diretamente pelo container da API.
- O frontend React consegue autenticar contra a API v2 depois do bootstrap, sem exigir cadastro manual pelo legado.
- O comando e idempotente para usuario existente e evita reset acidental de senha.

### Testes executados

- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run prisma:validate` em `portal-sama-api`: passou com `DATABASE_URL` de teste no ambiente do comando.
- `git diff --check`: passou, com avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- `prisma migrate deploy`, seed e bootstrap ainda precisam ser executados no container real com `DATABASE_URL` do EasyPanel.
- MySQL 9.6.0 do ambiente informado precisa ser validado contra as migrations Prisma, ja que a referencia anterior era MySQL 8/8.4.
- A independencia total do legado ainda depende de backfills, usuarios/permissoes reais, storage/ClamAV, HTTPS e fluxos Playwright reais; Integra-AI ainda tem mutacoes pendentes.

## 2026-05-22 10:02

### Arquivos alterados

- `portal-sama-web/tests/e2e/smoke.spec.ts`
- `.ai-tests/runs/2026-05-22-portal-documents-playwright/*`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/paginas/client-painel.md`

### O que mudou

- Adicionado teste Playwright persistido para `/documentos`, usando sessao autenticada e APIs mockadas.
- O novo teste cobre listagem de documento privado, cliente vinculado, extensoes permitidas, links publicos, resumo de pendencias obrigatorias e abertura do formulario de revisao.
- As permissoes mockadas do usuario administrativo da suite foram ampliadas para cobrir acoes documentais usadas pela central.

### Motivo da alteracao

Reduzir a pendencia documentada de Playwright para a central interna de documentos, uma tela critica por envolver storage privado, upload/download, revisao e links publicos.

### Impacto esperado

- A suite E2E do frontend passa de 6 para 7 testes Chromium.
- Maior protecao contra regressao de renderizacao/UX basica na central `/documentos`.
- Nenhuma alteracao em backend, banco, CSS, contratos de API ou regras reais de permissao.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e` em `portal-sama-web`: passou, 7 testes.
- `git diff --check`: passou, com aviso LF/CRLF do Git no Windows.

### Riscos ou pendencias

- A validacao ainda usa APIs mockadas; permanecem pendentes API v2/MySQL/storage/ClamAV reais, backfill, usuarios/permissoes reais, homologacao HTTPS e QA manual.
- Houve falhas intermediarias de seletor/mock corrigidas antes da execucao final.
- O aviso conhecido `No HydrateFallback element provided to render during initial hydration` segue aparecendo no servidor Vite durante os E2E.

## 2026-05-22 09:39

### Arquivos alterados

- `portal-sama-web/tests/e2e/smoke.spec.ts`
- `.ai-tests/runs/2026-05-22-0939-home-sidebar-mobile-playwright/*`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/paginas/home-inicio.md`

### O que mudou

- Adicionado teste Playwright persistido para `/home` em viewport mobile `390x844`.
- O novo teste valida sidebar expandida no mobile, labels visiveis, navegacao em uma coluna e ausencia de overflow horizontal.
- A suite E2E do frontend passou de 5 para 6 testes Chromium.

### Motivo da alteracao

Reduzir a pendencia documentada de QA automatizado mobile para Home/sidebar, dando continuidade a estabilizacao visual do `portal-sama-web`.

### Impacto esperado

- Maior protecao contra regressao responsiva na Home autenticada.
- Nenhuma alteracao em CSS, componentes, backend, banco ou contratos de API.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e` em `portal-sama-web`: passou, 6 testes.

### Riscos ou pendencias

- QA visual manual em navegador real e homologacao HTTPS com usuario/permissoes reais continuam pendentes.
- O aviso conhecido `No HydrateFallback element provided to render during initial hydration` segue aparecendo no servidor Vite durante os E2E.

## 2026-05-22 09:28

### Arquivos alterados

- `portal-sama-web/tests/e2e/smoke.spec.ts`
- `.ai-tests/runs/2026-05-22-0928-home-sidebar-playwright/*`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/paginas/home-inicio.md`

### O que mudou

- Adicionados testes Playwright persistidos para `/home`, cobrindo sessao autenticada, KPIs basicos e atalhos por permissao.
- Adicionado teste Playwright persistido para a sidebar compacta, cobrindo hover, clique em link ativo, mouseleave e expansao por foco de teclado.
- O mock de usuario administrativo da suite passou a conter permissoes representativas para liberar os principais atalhos da Home.

### Motivo da alteracao

Transformar os smokes recentes de Home/sidebar em cobertura versionada, reduzindo a pendencia documentada de QA automatizado para `/home` e sidebar recolhida.

### Impacto esperado

- E2E do frontend passa de 3 para 5 testes Chromium.
- Regressao visual/comportamental basica da Home e sidebar fica coberta em CI/local.
- Nao houve alteracao em backend, banco, contratos de API ou regras de permissao reais.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e` em `portal-sama-web`: passou, 5 testes.
- `git diff --check`: passou, com aviso LF/CRLF do Git no Windows.

### Riscos ou pendencias

- QA visual manual em navegador real ainda e recomendado para aprovar a sensacao da transicao da sidebar e o layout da Home em tamanhos reais.
- Uma tentativa de lint em paralelo com Playwright falhou por corrida conhecida no diretorio `test-results`; o lint isolado passou.

## 2026-05-22 09:07

### Arquivos alterados

- `portal-sama-web/src/index.css`
- `docs/*`

### O que mudou

- Corrigido o caso em que a sidebar permanecia aberta depois de clicar em um item do menu.
- A expansao por foco deixou de usar `:focus-within` e passou a usar `:focus-visible`, preservando abertura por teclado sem prender o menu apos clique com mouse.
- Ao clicar em um item e mover o mouse para fora, a sidebar volta automaticamente para o estado fechado.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- Smoke Playwright inline `sidebar-click-mouseleave-smoke`: passou, validando clique em item do menu e fechamento ao tirar o mouse.
- Smoke Playwright inline `sidebar-keyboard-focus-smoke`: passou, confirmando que `Tab` ainda expande a sidebar.
- `npm.cmd run test:e2e` em `portal-sama-web`: 3 testes passaram.

### Riscos ou pendencias

- QA visual manual em navegador real ainda e recomendado para confirmar a sensacao de fechamento apos navegacao.

## 2026-05-22 08:54

### Arquivos alterados

- `portal-sama-web/src/components/layout/AppLayout.tsx`
- `portal-sama-web/src/components/layout/Sidebar.tsx`
- `portal-sama-web/src/index.css`
- `docs/*`

### O que mudou

- A sidebar autenticada passou a ficar fechada por padrao no desktop.
- Ao passar o mouse sobre o menu, ou focar links pelo teclado, a sidebar expande automaticamente.
- O botao `Recolher menu` foi removido, ja que o comportamento agora e automatico por hover/foco.
- A logo Sama permanece visivel mesmo com a sidebar fechada; o container compacto exibe a marca no centro.
- Em telas ate `920px`, o menu permanece expandido para preservar usabilidade mobile.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- Smoke Playwright inline `sidebar-hover-smoke`: passou, validando largura fechada `96px`, expansao para `304px`, logo visivel e ausencia do botao de recolher.
- `npm.cmd run test:e2e` em `portal-sama-web`: 3 testes passaram.

### Riscos ou pendencias

- QA visual manual ainda e recomendado para aprovar a sensacao de hover em telas grandes e monitores menores.

## 2026-05-22 08:42

### Arquivos alterados

- `portal-sama-web/src/pages/auth/LoginPage.tsx`
- `portal-sama-web/src/index.css`
- `docs/*`

### O que mudou

- A lateral esquerda da pagina de login deixou de exibir o texto `Portal Interno`.
- A lateral agora usa a logo Sama em camadas com os mesmos assets da intro: core, halo, glows e orbitas animadas.
- Adicionado texto manuscrito `Bem vindo ao Portal Sama`, com revelacao progressiva simulando escrita e destaque de cor em `Portal` e `Sama`.
- O fundo azul do login foi refinado com gradientes, linhas SVG sutis nas bordas e brilho central para aproximar a pagina da linguagem visual da intro.
- O layout foi ajustado para desktop e mobile, evitando corte do texto manuscrito e respeitando `prefers-reduced-motion`.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e` em `portal-sama-web`: 3 testes passaram.
- Smoke Playwright inline `login-showcase-smoke`: passou em desktop `1920x920` e mobile `390x844`, confirmando ausencia de `Portal Interno`, texto completo e sem overflow.
- Screenshot Playwright de `/login`: gerado e conferido visualmente em desktop.

### Riscos ou pendencias

- QA visual manual em navegador real ainda e recomendado para aprovar tamanho/posicionamento final da logo e velocidade percebida da escrita.

## 2026-05-21 16:48

### Arquivos alterados

- `portal-sama-web/src/features/intro/components/PortalSamaIntro.tsx`
- `portal-sama-web/src/features/intro/components/PortalSamaLayer.tsx`
- `portal-sama-web/src/features/intro/styles/portal-sama-intro.css`
- `portal-sama-web/public/brand/sama/intro/manifest.json`
- `portal-sama-web/public/brand/sama/intro/assets/welcome/side-lines-left.svg`
- `portal-sama-web/public/brand/sama/intro/assets/welcome/side-lines-right.svg`
- `docs/*`

### O que mudou

- Corrigido o corte do texto manuscrito no final da frase; `Sama` agora aparece completo.
- A frase da welcome passou para `Seja bem vindo ao Portal Sama`, sem hifen.
- A camada de texto ganhou mais largura e respiro lateral para evitar clipping durante a animacao de escrita.
- As linhas laterais receberam mascaras/gradientes para ficarem fortes nas bordas e esmaecerem antes do centro.
- Halo, glows e orbitas foram alinhados ao centro visual real da logo, medido no PNG como cerca de `+7.7%` no eixo Y em relacao ao centro do canvas.

### Testes executados

- Validacao JSON de `manifest.json` e `animation-map.json`: passou.
- Checagem estatica dos SVGs da intro para tokens perigosos: sem ocorrencias.
- Smoke Playwright inline `welcome-text-enter-smoke`: passou, confirmando `Seja bem vindo ao Portal Sama` e sem overflow (`scrollWidth` igual a `clientWidth`).
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e` em `portal-sama-web`: 3 testes passaram.
- `npm.cmd run lint` em `portal-sama-web`: passou quando reexecutado isoladamente.
- `git diff --check`: passou, com avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- QA visual desktop/mobile continua pendente para confirmar no olho o fade das linhas e o alinhamento das orbitas/glows em navegadores reais.

## 2026-05-21 16:32

### Arquivos alterados

- `portal-sama-web/src/features/intro/components/PortalSamaIntro.tsx`
- `portal-sama-web/src/features/intro/styles/portal-sama-intro.css`
- `portal-sama-web/public/brand/sama/intro/manifest.json`
- `portal-sama-web/public/brand/sama/intro/assets/welcome/side-lines-left.svg`
- `portal-sama-web/public/brand/sama/intro/assets/welcome/side-lines-right.svg`
- `docs/*`

### O que mudou

- A cena `welcome` deixou de sair automaticamente por timer.
- O usuario precisa pressionar `Enter` para continuar.
- Apos 7s parado na tela, aparece uma notificacao discreta orientando `Pressione Enter para continuar`.
- O botao de pular nao aparece mais na cena `welcome`.
- Os SVGs laterais foram refeitos com varias curvas finas, opacidade menor, glow leve e particulas pequenas.
- As ondas WebP laterais tiveram opacidade reduzida para nao parecerem faixas grossas.

### Testes executados

- Validacao JSON de `manifest.json` e `animation-map.json`: passou.
- Checagem estatica dos SVGs novos para tokens perigosos: sem ocorrencias.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e` em `portal-sama-web`: 3 testes passaram.
- Smoke Playwright inline `welcome-enter-required-smoke`: passou.
- `git diff --check`: passou, com avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- QA visual desktop/mobile continua pendente para confirmar se as linhas ficaram suficientemente proximas da referencia.

## 2026-05-21 16:12

### Arquivos alterados

- `portal-sama-web/src/features/intro/components/PortalSamaIntro.tsx`
- `portal-sama-web/src/features/intro/components/PortalSamaLayer.tsx`
- `portal-sama-web/src/features/intro/hooks/useIntroTiming.ts`
- `portal-sama-web/src/features/intro/styles/portal-sama-intro.css`
- `portal-sama-web/public/brand/sama/intro/animation-map.json`
- `portal-sama-web/public/brand/sama/intro/manifest.json`
- `portal-sama-web/public/brand/sama/intro/assets/welcome/side-lines-left.svg`
- `portal-sama-web/public/brand/sama/intro/assets/welcome/side-lines-right.svg`
- `docs/*`

### O que mudou

- A intro normal passou para `2.7s` e a cena de boas-vindas para `4.5s`.
- O fade de encerramento foi ampliado para a saida ficar menos seca.
- O texto de boas-vindas agora usa tipografia manuscrita e animacao de revelacao progressiva, como escrita manual.
- Foram adicionadas duas camadas SVG laterais com glow, particulas discretas e animacao de desenho.
- As ondas/linhas da cena welcome foram contidas nas laterais para nao se encontrarem no centro, limitando o alcance visual a cerca de 30-35% da tela.

### Testes executados

- Validacao JSON de `manifest.json` e `animation-map.json`: passou.
- Checagem estatica dos SVGs novos para tokens perigosos: sem ocorrencias.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e` em `portal-sama-web`: 3 testes passaram.
- Smoke Playwright inline de intro login-only: passou.
- `git diff --check`: passou, com avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- QA visual desktop/mobile em navegador real continua pendente para aprovar timing, fonte manuscrita e intensidade dos efeitos.

## 2026-05-21 15:46

### Arquivos alterados

- `portal-sama-web/src/features/intro/components/IntroGate.tsx`
- `portal-sama-web/src/features/intro/components/PortalSamaIntro.tsx`
- `portal-sama-web/src/features/intro/hooks/useIntroTiming.ts`
- `portal-sama-web/src/features/intro/hooks/usePortalSamaGsapTimeline.ts`
- `portal-sama-web/src/features/intro/styles/portal-sama-intro.css`
- `portal-sama-web/public/brand/sama/intro/animation-map.json`
- `portal-sama-web/src/components/layout/Sidebar.tsx`
- `portal-sama-web/src/index.css`
- `portal-sama-web/public/brand/sama/dashboard/*.svg`
- `docs/*`

### O que mudou

- A intro autenticada foi ajustada para rodar somente quando a sessao vem de `login`, nao em refresh/snapshot/reload.
- `PortalSamaIntro` ganhou saida com fade e timings mais curtos: cerca de 1.7s na entrada normal e 2.3s na cena de boas-vindas.
- O conflito visual que causava flick do logo foi reduzido movendo a centralizacao CSS para `translate` e deixando o GSAP controlar `transform`.
- O mapa GSAP foi retimado com easings mais suaves, loops discretos e menos blur no logo.
- A sidebar passou a usar a logo Sama sem fundo dentro de um container centralizado e destacado.
- Os SVGs decorativos da sidebar, topo escuro e KPIs foram simplificados para linhas finas, removendo marcas grossas e riscos soltos.

### Motivo da alteracao

Atender ao ajuste visual solicitado apos revisao por GIF/screenshot: a experiencia precisava parecer mais fluida, moderna e profissional, sem reaparecer ao recarregar a pagina.

### Impacto esperado

- Login mostra a intro uma vez; reload da rota autenticada nao mostra a intro novamente.
- Menos flicker visual no logo durante escala/rotacao.
- Sidebar e cards da Home ficam mais alinhados ao padrao escuro/fino da referencia visual.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e` em `portal-sama-web`: 3 testes passaram.
- Smoke Playwright inline de intro login-only: passou; a intro apareceu apos login e nao voltou apos reload autenticado.
- `git diff --check`: passou, com avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- QA visual desktop/mobile em navegador real continua pendente.
- Homologacao HTTPS com usuario real/permissoes reais continua pendente.
- Criar Playwright visual especifico para intro/sidebar/KPIs se o visual for aprovado.

## 2026-05-21 14:24

### Arquivos alterados

- `portal-sama-web/src/features/intro/hooks/usePortalSamaGsapTimeline.ts`
- `portal-sama-web/src/app/router.tsx`
- `portal-sama-web/src/pages/errors/RouteErrorPage.tsx`
- `.ai-tests/runs/2026-05-21-1424-gsap-import-guard/*`
- `docs/*`

### O que mudou

- O GSAP deixou de ser importado no topo do hook da intro e passou a ser carregado dinamicamente dentro do efeito da timeline.
- Falha ao carregar/inicializar GSAP agora chama fallback estatico da intro, em vez de derrubar o carregamento da rota autenticada.
- Foi adicionada uma pagina de erro de rota propria para evitar a tela crua `Unexpected Application Error` em falhas futuras de lazy routes.
- O cache otimizado do Vite em `portal-sama-web/node_modules/.vite` foi limpo apos parar servidores Vite antigos do workspace, e o dev server foi reiniciado com `--force` em `127.0.0.1:5173`.

### Motivo da alteracao

Corrigir o erro visto apos login em desenvolvimento: `error loading dynamically imported module: http://localhost:5173/node_modules/.vite/deps/gsap.js?...`, mantendo a regra de que falha visual da intro nunca bloqueia login, sessao ou navegacao.

### Impacto esperado

- O chunk do `AppLayout` nao depende mais de import estatico de GSAP.
- Problemas de cache/prebundle do Vite no GSAP caem no fallback visual da intro.
- A rota autenticada continua carregando mesmo quando a timeline animada nao estiver disponivel.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd run test:e2e` em `portal-sama-web`: 3 testes passaram.
- Smoke Playwright em `http://127.0.0.1:5173/home`: passou sem `Unexpected Application Error` e sem erro de import dinamico.
- `Invoke-WebRequest` para `manifest.json`, `animation-map.json` e `/node_modules/.vite/deps/gsap.js`: HTTP 200.
- `git diff --check`: passou, com avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- QA visual desktop/mobile e homologacao HTTPS continuam pendentes.
- A rota `/dev/intro-preview` ainda merece teste visual manual com falha forcada, static, animated e reduced motion.

## 2026-05-21 14:05

### Arquivos alterados

- `portal-sama-web/package.json`
- `portal-sama-web/package-lock.json`
- `portal-sama-web/public/brand/sama/intro/**`
- `portal-sama-web/src/features/intro/**`
- `portal-sama-web/src/components/layout/AppLayout.tsx`
- `portal-sama-web/src/index.css`
- `.ai-tests/runs/2026-05-21-1405-intro-gsap-v2/*`
- `docs/*`

### O que mudou

- A intro oficial foi migrada para GSAP e `@gsap/react`, com `manifest.json` e `animation-map.json` versionados como `sama-gsap-layers-v2`.
- A feature continua em `src/features/intro` e agora consome assets finais locais em `/brand/sama/intro/assets`.
- Foram mantidas as integracoes funcionais seguras: `IntroGate`, `welcomeAnimationSeen`, `markWelcomeIntroSeen`, CSRF, auditoria e `AppLayout`.
- Foram removidos componentes visuais Framer/Motion antigos e assets SVG antigos da intro.
- A sidebar recebeu reforco de cor para labels e botao de recolher em branco.

### Motivo da alteracao

Atender a `docs/design/intro-gsap-V2`, tornando a intro GSAP a unica experiencia visual oficial e aposentando a camada visual antiga baseada em Framer Motion/SVGs legados.

### Impacto esperado

- Menos duplicidade de engine visual na intro.
- Fallback estatico seguro em reduced motion, erro de configuracao ou asset obrigatorio ausente.
- Paths de assets passam por validacao antes de renderizar.
- A falha visual da intro nao deve bloquear autenticacao, sessao ou navegacao.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `Select-String` em `public/brand/sama/intro/assets/logo/*.svg` para tokens proibidos: sem ocorrencias.

### Riscos ou pendencias

- Validar visualmente em navegador real desktop/mobile e homologacao.
- Criar Playwright visual especifico para loading, welcome, static, animated, reduced motion e falha de asset obrigatorio.

## 2026-05-21 10:57

### Arquivos alterados

- `portal-sama-web/src/components/layout/AppLayout.tsx`
- `portal-sama-web/src/components/layout/Sidebar.tsx`
- `portal-sama-web/src/components/layout/Header.tsx`
- `portal-sama-web/src/components/layout/PageShell.tsx`
- `portal-sama-web/src/components/ui/*`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `portal-sama-web/src/index.css`
- `portal-sama-web/public/brand/sama/dashboard/*`
- `docs/*`

### O que mudou

- Implementado o padrao visual inicial do `portal-sama-web` conforme a documentacao de design.
- A sidebar agora e premium, escura, responsiva e recolhivel, preservando `NavLink` e `PermissionGate`.
- Criados componentes reutilizaveis `PageShell`, `AppPanel`, `IconBadge`, `KpiCard` e `ShortcutCard`.
- `/home` passou a usar os novos componentes para KPIs e atalhos por permissao.
- Adicionados assets decorativos de dashboard e removido o aviso de build do asset de marca antigo, apontando para o PNG de referencia existente.

### Motivo da alteracao

Atender a `docs/design/codex_estilizacao_padrao_portal_sama.md`, criando uma base visual reutilizavel antes de propagar o acabamento para todas as telas internas.

### Impacto esperado

- A aplicacao React passa a ter uma identidade visual mais consistente, corporativa e alinhada a intro.
- Novas paginas podem reutilizar os componentes base em vez de duplicar CSS.
- O layout preserva rotas, permissoes, botoes e links reais.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou, sem o aviso antigo de `/brand/sama-logo-symbol.svg`.
- `npm.cmd run test:e2e` em `portal-sama-web`: passou, 3 testes Chromium.

### Riscos ou pendencias

- Propagar `PageShell` e componentes base para as demais telas.
- Criar QA visual/Playwright para `/home`, sidebar recolhida e mobile.
- Validar contraste, navegacao por teclado e layout em homologacao.
- Decidir/restaurar os assets de intro que ja estavam deletados no workspace antes desta rodada.

## 2026-05-21 10:33

### Arquivos alterados

- `portal-sama-api/src/modules/accounting/*`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-web/src/pages/accounting/IntegraAiPage.tsx`
- `portal-sama-web/src/services/integra-ai.service.ts`
- `portal-sama-web/src/types/integra-ai.ts`
- `portal-sama-web/src/app/router.tsx`
- `portal-sama-web/src/components/layout/navigation.tsx`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `docs/*`

### O que mudou

- Criado `AccountingModule` para iniciar a substituicao de `api/integra_ai.php` com uma fatia read-only.
- Adicionados `GET /api-v2/accounting/integra-ai/workspaces` e `GET /api-v2/accounting/integra-ai/jobs/:id`.
- Adicionada permissao RBAC `accounting.integra_ai.read` ao catalogo padrao e ao papel `ACCOUNTING`.
- Criada a rota React `/contabil/integra-ai`, com filtros, jobs recentes, detalhe, resumo de linhas e preview sem expor caminhos privados.
- A intro/assets nao foram continuados nesta rodada.

### Motivo da alteracao

Avancar uma pagina critica documentada em `docs/paginas/contabil-integra-ai.md`, reduzindo dependencia de consulta do PHP legado sem antecipar upload/parser/exportacao antes da camada segura.

### Impacto esperado

- Usuarios com `accounting.integra_ai.read` passam a consultar workspace e jobs do Integra-AI no React.
- O legado continua responsavel por importacao, regras, geracao e download ate a proxima fatia segura.
- A matriz de migracao passa de "mapeado" para "implementado parcialmente" nessa area.

### Testes executados

- `npm.cmd test -- accounting.service.spec.ts --runInBand` em `portal-sama-api`: passou, 3 testes.
- `npm.cmd test -- rbac/default-rbac.spec.ts --runInBand` em `portal-sama-api`: passou, 2 testes.
- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; aviso conhecido de asset de marca em runtime permaneceu.

### Riscos ou pendencias

- Validar com MySQL/homologacao e volume real das tabelas `sama_integra_ai_*`.
- Rodar seed RBAC real para `accounting.integra_ai.read`.
- Migrar upload/parser/TXT com quarentena, limites, auditoria de download e CSRF nas mutacoes.
- Criar Playwright para leitura, sem permissao, detalhe de job e fallback de base legada indisponivel.

## 2026-05-21 09:34

### Arquivos alterados

- `portal-sama-api/src/modules/departments/*`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-web/src/pages/departments/DepartmentModelPage.tsx`
- `portal-sama-web/src/services/departments.service.ts`
- `portal-sama-web/src/types/departments.ts`
- `portal-sama-web/src/app/router.tsx`
- `portal-sama-web/src/components/layout/navigation.tsx`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `docs/*`

### O que mudou

- Criado `DepartmentsModule` para substituir inicialmente `api/fiscal_workspace.php?action=load_board/cycle_cell/set_cell_status`.
- Adicionados contratos `GET /api-v2/departments/fiscal/workspace`, `POST /cycle-cell` e `PATCH /cell-status`.
- Adicionadas permissoes RBAC `departments.workspace.read` e `departments.workspace.write`.
- Criada a rota React `/departamentos/modelo`, com grade mensal, status por celula, bloqueios, resumo e carrossel de vencimentos vinculados.
- A intro/assets nao foram continuados nesta rodada.

### Motivo da alteracao

Avancar uma pagina pendente de alta prioridade documentada em `docs/paginas/depto-modelo.md`, mantendo a regra operacional de planilha fiscal e reduzindo dependencia do PHP legado.

### Impacto esperado

- Usuarios com `departments.workspace.read` passam a consultar o modelo departamental no React.
- Usuarios com `departments.workspace.write` e escopo Fiscal podem alternar status de celulas no mes atual com CSRF e auditoria.
- A matriz de migracao passa de "mapeado" para "implementado parcialmente" nessa fatia.

### Testes executados

- `npm.cmd test -- departments.service.spec.ts --runInBand` em `portal-sama-api`: passou, 4 testes.
- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; aviso conhecido de asset de marca em runtime permaneceu.

### Riscos ou pendencias

- Validar com MySQL/homologacao e dados reais de `Client.metadata`/responsaveis fiscais.
- Rodar seed RBAC real para `departments.workspace.*`.
- Criar Playwright para leitura, sem permissao, celula bloqueada e mutacao com CSRF.
- Desligar `api/fiscal_workspace.php` somente depois da homologacao.

## 2026-05-21 09:13

### Arquivos alterados

- `docker-compose.easypanel.example.yml`
- `compose.easypanel.env.example`
- `portal-sama-web/.dockerignore`
- `portal-sama-web/nginx.conf`
- `docs/*`

### O que mudou

- Criado compose de referencia para homologacao/EasyPanel com `portal-sama-mysql`, `portal-sama-api` e `portal-sama-web`.
- Criado arquivo de variaveis de exemplo para validar o compose sem ler o `.env` local.
- Adicionado `.dockerignore` no frontend para reduzir contexto de build.
- O Nginx do frontend passou a resolver `portal-sama-api` via DNS interno Docker `127.0.0.11` e proxyar com `$request_uri`.
- A intro/assets nao foram continuados nesta rodada.

### Motivo da alteracao

Alinhar os artefatos de deploy aos nomes reais usados pelo proxy `/api-v2`, reduzindo risco de erro de DNS interno no EasyPanel e deixando uma topologia reproduzivel para homologacao.

### Impacto esperado

- O proxy `/api-v2` fica compatível com os nomes de servico documentados/versionados.
- A equipe ganha comandos objetivos para validar servicos e volumes antes do deploy.
- Builds Docker do web deixam de carregar arquivos locais desnecessarios.

### Testes executados

- `docker compose --env-file compose.easypanel.env.example -f docker-compose.easypanel.example.yml config --services`: passou.
- `docker compose --env-file compose.easypanel.env.example -f docker-compose.easypanel.example.yml config --volumes`: passou.
- `docker run ... nginx:alpine nginx -t`: nao executou por Docker daemon indisponivel.

### Riscos ou pendencias

- Validar `nginx -t` com Docker daemon ativo ou no container do EasyPanel.
- Substituir placeholders por segredos reais no EasyPanel, nunca versionados.
- Rodar migrations/seed e smoke publico sem `--soft` depois do redeploy.

## 2026-05-21 08:56

### Arquivos alterados

- `portal-sama-web/nginx.conf`
- `docs/*`

### O que mudou

- `portal-sama-web` passou a proxyar `/api-v2/` para `http://portal-sama-api:3000/api-v2/`.
- Adicionado redirecionamento de `/api-v2` para `/api-v2/`.
- Preservados headers de proxy `Host`, `X-Real-IP`, `X-Forwarded-For`, `X-Forwarded-Host`, `X-Forwarded-Port` e `X-Forwarded-Proto`.
- Definido `client_max_body_size 30m` para compatibilidade com uploads da API.
- Configurado `/api-v2/notifications/stream` sem buffering para SSE.
- A intro/assets nao foram continuados nesta rodada.

### Motivo da alteracao

O smoke publico mostrou `portal.samacontabil.com.br` respondendo HTTP 200 na raiz e HTTP 404 em `/api-v2/*`. Como o dominio pode estar apontado para o container web, o Nginx do frontend precisa encaminhar o prefixo da API para o servico NestJS interno.

### Impacto esperado

- Depois do redeploy, `https://portal.samacontabil.com.br/api-v2/health` deve chegar ao `portal-sama-api`.
- O dominio unico fica viavel sem depender de path proxy externo no EasyPanel.
- Uploads e SSE passam a ter configuracao compativel no proxy web.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; o build manteve apenas o aviso conhecido de asset de marca em runtime.
- `Get-Command nginx -ErrorAction SilentlyContinue`: Nginx local nao encontrado, portanto `nginx -t` ficou pendente.

### Riscos ou pendencias

- Validar a sintaxe Nginx no container/imagem do deploy.
- Garantir que `portal-sama-api` seja resolvido pelo DNS interno do EasyPanel na mesma rede do `portal-sama-web`.
- Reexecutar `npm.cmd run smoke:public` sem `--soft` apos redeploy.

## 2026-05-21 08:41

### Arquivos alterados

- `scripts/portal-public-smoke.mjs`
- `package.json`
- `docs/*`
- `.ai-tests/runs/2026-05-21-0841-public-deploy-smoke/*`

### O que mudou

- Criado smoke publico para validar dominio, `/api-v2/health`, CORS e cookie CSRF.
- Exposto `npm run smoke:public` na raiz do repositorio.
- Atualizados os documentos de incidente, deploy, status, pendencias e testes com a evidencia de 2026-05-21.
- A intro/assets nao foram continuados nesta rodada.

### Motivo da alteracao

Transformar a pendencia do subdominio/API publica em uma validacao repetivel antes da troca definitiva de trafego no EasyPanel.

### Impacto esperado

- O time passa a ter um comando unico para homologar o dominio publico.
- Falhas de proxy `/api-v2`, CORS e cookie CSRF aparecem de forma objetiva.
- O mesmo script pode validar `api.portal.samacontabil.com.br` usando `--api-url`.

### Testes executados

- `node scripts/portal-public-smoke.mjs --help`: passou.
- `npm.cmd run smoke:public -- --help`: passou.
- `npm.cmd run smoke:public -- --soft`: executou contra o dominio real; raiz HTTP 200, API v2 ainda HTTP 404 em `/health` e `/auth/csrf`.
- `git diff --check`: passou com avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Publicar/proxyar `portal-sama-api` em `/api-v2` ou configurar dominio separado de API.
- Reexecutar `npm.cmd run smoke:public` sem `--soft` depois do deploy.
- Validar fluxos reais de login, refresh, upload/download e auditoria em HTTPS.

## 2026-05-20 17:36

### Arquivos alterados

- `portal-sama-web/package.json`
- `portal-sama-web/package-lock.json`
- `portal-sama-web/.gitignore`
- `portal-sama-web/playwright.config.ts`
- `portal-sama-web/tests/e2e/smoke.spec.ts`
- `docs/*`
- `.ai-tests/runs/2026-05-20-1736-playwright-smoke/*`

### O que mudou

- Adicionado `@playwright/test` ao frontend.
- Criados scripts `test:e2e` e `test:e2e:headed`.
- Criado `playwright.config.ts` com dev server Vite, projeto Chromium, downloads aceitos e artefatos em falha.
- Criada suite smoke cobrindo `/login`, `/onboarding/publico/documentos/:token` com API mockada e `/auditoria` com auth/listagem/exportacao CSV mockadas.
- A intro/assets nao foram continuados nesta rodada.

### Motivo da alteracao

Reduzir a pendencia recorrente de Playwright/navegacao real e criar uma base de E2E que nao dependa de MySQL/API reais para rodar localmente.

### Impacto esperado

- O frontend passa a ter uma porta inicial para testes de navegador automatizados.
- Rotas publicas e uma rota autenticada critica passam a ter smoke test.
- Novos fluxos podem reaproveitar mocks e config existentes.

### Testes executados

- `npx.cmd playwright install chromium`: passou.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite manteve apenas o aviso conhecido de asset de marca em runtime.
- `npm.cmd run test:e2e` em `portal-sama-web`: passou, 3 testes em Chromium.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou; houve apenas avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Ampliar Playwright para fluxos criticos autenticados, permissoes, mobile e formularios principais.
- Rodar E2E integrado em homologacao com API v2/MySQL reais.
- Avaliar o aviso de `HydrateFallback` do React Router em rodada separada.

## 2026-05-20 17:24

### Arquivos alterados

- `portal-sama-api/src/modules/audit/audit.controller.ts`
- `portal-sama-api/src/modules/audit/audit.service.ts`
- `portal-sama-api/src/modules/audit/audit.service.spec.ts`
- `portal-sama-api/src/modules/audit/audit.types.ts`
- `portal-sama-web/src/pages/audit/AuditPage.tsx`
- `portal-sama-web/src/services/audit.service.ts`
- `docs/*`
- `.ai-tests/runs/2026-05-20-1724-audit-csv-export/*`

### O que mudou

- Criado `GET /api-v2/audit/logs/export.csv` com os mesmos filtros da listagem de auditoria.
- A exportacao usa limite fixo de 1000 linhas, `Cache-Control: no-store`, filename controlado e registra `audit.logs.export`.
- Celulas CSV sao escapadas e protegidas contra valores iniciados por `=`, `+`, `-` ou `@`.
- `/auditoria` ganhou botao `Exportar CSV` usando os filtros atuais.
- A intro/assets nao foram continuados nesta rodada.

### Motivo da alteracao

Reduzir a lacuna documentada de exportacao operacional da auditoria sem migrar backup/restauracao antes de existir contrato separado.

### Impacto esperado

- Administradores com `audit.read` conseguem extrair logs filtrados para analise operacional.
- Exportacoes passam a deixar trilha em auditoria.
- A consulta visual existente permanece inalterada.

### Testes executados

- `npm.cmd test -- audit.service.spec.ts --runInBand` em `portal-sama-api`: passou, 5 testes.
- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite manteve apenas o aviso conhecido de asset de marca em runtime.
- `git diff --check`: passou; houve apenas avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar com MySQL real e volume operacional.
- Criar Playwright para filtro/detalhe/exportacao em `/auditoria`.
- Definir politica formal de retencao e tratar backup/restauracao em modulo separado.

## 2026-05-20 17:11

### Arquivos alterados

- `portal-sama-api/src/modules/documents/documents.controller.ts`
- `portal-sama-api/src/modules/documents/documents.service.ts`
- `portal-sama-api/src/modules/documents/documents.service.spec.ts`
- `portal-sama-api/src/modules/documents/documents.types.ts`
- `portal-sama-web/src/pages/documents/PublicDocumentsPage.tsx`
- `portal-sama-web/src/services/documents.service.ts`
- `portal-sama-web/src/types/documents.ts`
- `docs/*`
- `.ai-tests/runs/2026-05-20-1711-public-documents-checklist/*`

### O que mudou

- Criado `GET /api-v2/documents/public-checklist?token=...` para substituir a leitura publica de checklist documental do legado.
- A resposta publica valida token, escopo, expiracao e revogacao antes de retornar dados minimos do cliente e dos itens solicitados.
- `/onboarding/publico/documentos/:token` passou a carregar checklist real, status resumido, extensoes permitidas da API e selecao de documento/departamento pelos itens do checklist.
- A intro/assets nao foram continuados nesta rodada.

### Motivo da alteracao

Fechar a lacuna documentada de checklist publico especifico para documentos de onboarding, reduzindo a dependencia de `api/onboarding.php?action=docs_client_get`.

### Impacto esperado

- Cliente externo passa a ver os documentos solicitados pelo contrato novo da API v2.
- A tela publica deixa de depender somente de sugestoes estaticas para tipo/departamento.
- O endpoint publico nao entrega dados internos de documento, download ou armazenamento.

### Testes executados

- `npm.cmd test -- documents.service.spec.ts --runInBand` em `portal-sama-api`: passou, 28 testes.
- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite manteve apenas o aviso conhecido de asset de marca em runtime.
- `git diff --check`: passou; houve apenas avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar com MySQL/storage/ClamAV reais, tokens reais/legados, HTTPS e Playwright desktop/mobile.
- Manter o PHP legado ate homologacao fim a fim da leitura do checklist e upload publico.

## 2026-05-20 16:51

### Arquivos alterados

- `portal-sama-web/src/app/router.tsx`
- `docs/*`
- `.ai-tests/runs/2026-05-20-1651-route-code-splitting/*`

### O que mudou

- O roteador React passou a carregar `AppLayout`, paginas privadas, paginas publicas, 404 e rota dev de preview com `lazy` do React Router.
- Os imports ansiosos de paginas foram substituidos por um helper tipado para exports nomeados.
- A intro/assets nao foram continuados nesta rodada.

### Motivo da alteracao

Tratar a pendencia documentada de code splitting e reduzir o alerta de chunk grande do Vite sem alterar fluxo funcional de telas.

### Impacto esperado

- Menor bundle inicial no frontend.
- Carregamento sob demanda das paginas conforme navegacao.
- Build final sem alerta de chunk acima de 500 kB.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; chunk inicial em 320.90 kB e `AppLayout` separado em 138.47 kB.
- `git diff --check`: passou; houve apenas avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar navegacao real desktop/mobile e Playwright em homologacao.
- O aviso conhecido de `/brand/sama-logo-symbol.svg` resolvido em runtime permanece.

## 2026-05-20 16:21

### Arquivos alterados

- `portal-sama-web/src/pages/notifications/NotificationsPage.tsx`
- `portal-sama-web/src/services/notifications.service.ts`
- `portal-sama-web/src/types/notifications.ts`
- `portal-sama-web/public/sama-push-sw.js`
- `docs/*`

### O que mudou

- `/notificacoes` passou a abrir o stream `GET /api-v2/notifications/stream` por `fetch` autenticado, com status visual de tempo real e fallback por refetch quando o stream cai.
- A tela agora consulta suporte/permissao de Web Push no navegador e permite ativar/desativar a assinatura Push API usando `/api-v2/notifications/push/subscribe` e `/unsubscribe`.
- O service worker de push foi disponibilizado tambem em `portal-sama-web/public/sama-push-sw.js` para o app Vite.

### Motivo da alteracao

Fechar a lacuna documentada de SSE/Push API no React, mantendo access token fora de storage e sem expor bearer em query string.

### Impacto esperado

- A central React passa a receber atualizacoes de notificacoes em tempo real quando a API v2 estiver acessivel.
- Usuarios podem assinar e cancelar Web Push diretamente no React quando VAPID/HTTPS estiverem configurados.
- O legado permanece ativo ate homologacao completa e validacao dos browsers.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite manteve avisos conhecidos de chunk grande e asset de marca resolvido em runtime.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou; houve apenas avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar em HTTPS/homologacao com API v2, MySQL, VAPID real e permissao real do navegador.
- Testar portal fechado, unsubscribe, mobile/desktop e fallback do stream.
- Criar Playwright para `/notificacoes` e decidir broker/event bus se houver multi-instancia.

## 2026-05-20 16:02

### Arquivos alterados

- `portal-sama-api/src/modules/calendar/calendar.controller.ts`
- `portal-sama-api/src/modules/calendar/calendar.service.ts`
- `portal-sama-api/src/modules/calendar/calendar.service.spec.ts`
- `portal-sama-api/src/modules/calendar/dto/delete-calendar-entry.dto.ts`
- `portal-sama-web/src/pages/manager/ManagerDashboardPage.tsx`
- `portal-sama-web/src/services/calendar.service.ts`
- `portal-sama-web/src/types/calendar.ts`
- `docs/*`

### O que mudou

- Criado `DELETE /api-v2/calendar/entries/:id` para remover um vencimento vinculado a empresa.
- A exclusao valida regra/empresa/departamento, exige `calendar.manage`, CSRF e papel operacional, limpa notificacoes do vencimento e remove a regra quando ela fica sem vinculos.
- `/manager` passou a exibir botao de remover em proximos vencimentos apenas para usuarios com `calendar.manage`.

### Motivo da alteracao

Reduzir a dependencia de `api/manager_workspace.php?action=calendar_delete_entry` e completar mais uma mutacao critica do calendario do gestor em NestJS/React.

### Impacto esperado

- Gestores autorizados removem vencimentos pela API v2 sem depender da action PHP direta.
- A remocao passa a ter trilha centralizada em `AuditService`.
- O PHP legado permanece ativo ate validacao com dados reais e desligamento controlado.

### Testes executados

- `npm.cmd test -- calendar.service.spec.ts` em `portal-sama-api`: passou, 2 testes.
- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run prisma:validate` em `portal-sama-api`: passou.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite manteve avisos conhecidos de chunk grande e asset de marca resolvido em runtime.
- `git diff --check`: passou; houve apenas avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar exclusao com MySQL/homologacao e dados reais de calendario/carteira.
- Testar usuarios MANAGER/ADMIN/DEV, usuario sem `calendar.manage` e escopo por departamento.
- Criar Playwright para remover vencimento, estado de carregamento, erro e usuario sem permissao.
- Desligar `calendar_delete_entry` no PHP somente depois da homologacao.

## 2026-05-20 12:30

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260520123000_add_user_presence_model/migration.sql`
- `portal-sama-api/src/modules/managers/*`
- `portal-sama-api/src/modules/managers/dto/*`
- `portal-sama-web/src/pages/manager/ManagerDashboardPage.tsx`
- `portal-sama-web/src/services/managers.service.ts`
- `portal-sama-web/src/types/manager-overview.ts`
- `docs/*`

### O que mudou

- Criado `GET /api-v2/managers/overview` para KPIs, lista operacional e presenca da equipe do gestor.
- Modelada a tabela legada `sama_user_presence` no Prisma como `UserPresence`, com migration idempotente.
- `/manager` passou a consumir o overview novo e mostrar presenca online/offline integrada aos filtros do dashboard.

### Motivo da alteracao

Reduzir a dependencia restante de `api/manager_workspace.php?action=overview` e fechar mais uma fatia do workspace do gestor em NestJS/React.

### Impacto esperado

- Gestores com `manager_history.read` passam a consultar o resumo do time pela API v2.
- A leitura de presenca fica sujeita a JWT, RBAC e escopo por departamento no backend.
- O PHP legado permanece ativo ate homologacao e desligamento controlado.

### Testes executados

- `npm.cmd run prisma:format` em `portal-sama-api`: passou.
- `npm.cmd run prisma:generate` em `portal-sama-api`: passou.
- `npm.cmd run prisma:validate` em `portal-sama-api`: passou.
- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite manteve avisos conhecidos de chunk grande e asset de marca resolvido em runtime.
- `git diff --check`: passou; houve apenas avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar migration e leitura de `sama_user_presence` em MySQL/homologacao.
- Testar usuarios MANAGER/ADMIN/DEV e escopo por departamento com dados reais.
- Criar Playwright para `/manager` e rotas sem permissao.
- Desligar a action PHP `overview` apenas depois da homologacao.

## 2026-05-20 11:45

### Arquivos alterados

- `portal-sama-api/src/modules/managers/*`
- `portal-sama-api/src/modules/managers/dto/*`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.spec.ts`
- `portal-sama-web/src/pages/manager/ManagerHistoryPage.tsx`
- `portal-sama-web/src/services/manager-history.service.ts`
- `portal-sama-web/src/types/manager-history.ts`
- `docs/*`

### O que mudou

- Adicionadas mutacoes auditaveis do historico do gestor: criar historico, editar historico e salvar snapshot de vida da empresa.
- Incluida permissao `manager_history.write` no RBAC padrao e no papel MANAGER.
- `/manager/historico` ganhou formularios para registrar/editar topicos e salvar vida da empresa quando a API retorna `can_edit`.

### Motivo da alteracao

Dar continuidade ao contrato de `Manager/manager.html` que ainda dependia de `history_append`, `history_update` e `company_life_save` no PHP legado.

### Impacto esperado

- Gestores autorizados deixam de depender do PHP legado para gravar historico operacional e vida da empresa.
- Escrita passa por CSRF, RBAC, escopo por departamento e auditoria centralizada.

### Testes executados

- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd run prisma:validate` em `portal-sama-api`: passou.
- `npx.cmd jest src/modules/rbac/default-rbac.spec.ts --runInBand` em `portal-sama-api`: passou, 2 testes.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite manteve avisos conhecidos de chunk grande e asset de marca resolvido em runtime.

### Riscos ou pendencias

- Validar em MySQL/homologacao com dados legados reais.
- Executar seed RBAC atualizado para persistir `manager_history.write`.
- Criar Playwright para leitura, criacao, edicao, vida da empresa, usuario sem permissao e responsividade.
- `overview` do manager workspace continua no PHP legado.

## 2026-05-20 11:20

### Arquivos alterados

- `portal-sama-api/src/modules/managers/*`
- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260520112000_add_manager_history_entries/migration.sql`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-web/src/pages/manager/ManagerHistoryPage.tsx`
- `portal-sama-web/src/services/manager-history.service.ts`
- `portal-sama-web/src/types/manager-history.ts`
- `portal-sama-web/src/app/router.tsx`
- `portal-sama-web/src/components/layout/navigation.tsx`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `portal-sama-web/src/pages/manager/ManagerDashboardPage.tsx`
- `docs/*`

### O que mudou

- Criado `ManagersModule` com leitura de historico operacional e timeline por empresa.
- Mapeadas as tabelas legadas `sama_company_history_entries` e `sama_company_life_entries` no Prisma.
- Criada a rota React `/manager/historico`, com filtros por departamento/busca e timeline consolidada.

### Motivo da alteracao

Dar continuidade ao dominio de gestor cobrindo o historico documentado em `Manager/manager.html`, sem mexer na intro.

### Impacto esperado

- Gestores com `manager_history.read` passam a consultar historico por departamento na API v2.
- A tela nova permanece somente leitura ate existir contrato de escrita com CSRF e auditoria.

### Testes executados

- `npx.cmd prisma format` em `portal-sama-api`: passou.
- `npx.cmd prisma generate` em `portal-sama-api`: passou.
- `npm.cmd run prisma:validate` em `portal-sama-api`: passou.
- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npx.cmd jest src/modules/rbac/default-rbac.spec.ts --runInBand` em `portal-sama-api`: passou, 2 testes.
- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite manteve avisos conhecidos de chunk grande e asset de marca resolvido em runtime.

### Riscos ou pendencias

- Validar MySQL/homologacao com dados legados reais e seed `manager_history.read`.
- Criar Playwright para `/manager/historico`.
- Implementar escrita/edicao auditavel depois de fechar contrato de permissao e CSRF.

## 2026-05-19 17:56

### Arquivos alterados

- `portal-sama-web/src/pages/dev/DevCollaboratorsPage.tsx`
- `portal-sama-web/src/app/router.tsx`
- `portal-sama-web/src/pages/dev/DevAdminPage.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/paginas/dev-colaborador.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-19-1756-dev-collaborators-react/*`

### O que mudou

- Criada a rota React `/dev/colaboradores` para gestao administrativa de colaboradores.
- A tela consome `GET/PATCH/DELETE /api-v2/collaborators` por `collaborators.service.ts` e usa `GET /api-v2/roles` como apoio quando permitido.
- `DevAdminPage.tsx` ganhou atalho `Gerenciar colaboradores` condicionado a `collaborators.read`.

### Motivo da alteracao

Fechar a tela DEV administrativa de colaboradores usando contratos seguros ja existentes, sem migrar carteira/transferencias alem do que ja esta coberto por `/manager/transferencias`.

### Impacto esperado

- DEV/Admin com permissao passa a ter uma tela dedicada para revisar e editar colaboradores internos.
- Status, senha opcional, perfis e arquivamento continuam validados no backend com CSRF/RBAC/auditoria.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite manteve avisos conhecidos de chunk grande e asset de marca resolvido em runtime.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check` nos arquivos da fatia: passou com avisos LF/CRLF do Git no Windows.
- `Invoke-WebRequest http://127.0.0.1:5174/dev/colaboradores`: HTTP 200.

### Riscos ou pendencias

- Validar com API v2/MySQL/usuarios reais, especialmente perfis administrativos.
- Criar Playwright para leitura, edicao, arquivamento, usuario sem permissao e responsividade.
- Formalizar backfill/vinculos de colaboradores antes de substituir totalmente as telas DEV legadas.

## 2026-05-19 15:53

### Arquivos alterados

- `portal-sama-web/src/pages/manager/ManagerCollaboratorsPage.tsx`
- `portal-sama-web/src/app/router.tsx`
- `portal-sama-web/src/components/layout/navigation.tsx`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/paginas/manager-colaborador.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-19-1553-manager-collaborators-react/*`

### O que mudou

- Criada a rota React `/manager/colaboradores` para consulta de carteira por colaborador.
- A tela usa `GET /api-v2/transfers/dashboard` para listar colaboradores, empresas vinculadas, transferencias ativas e sessoes relacionadas.
- Sidebar e Home ganharam atalho `Carteira gestor` condicionado a `transfers.read`.

### Motivo da alteracao

Dar continuidade ao dominio de gestor usando o contrato de transferencias ja auditavel, sem migrar ainda calendario/historico que dependem de contratos dedicados futuros.

### Impacto esperado

- Gestores com permissao conseguem consultar rapidamente carteira por colaborador em React.
- Operacoes de transferencia seguem concentradas em `/manager/transferencias`, com CSRF/RBAC/auditoria no backend.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite manteve avisos conhecidos de chunk grande e asset de marca resolvido em runtime.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check` nos arquivos da fatia: passou com avisos LF/CRLF do Git no Windows.
- `Invoke-WebRequest http://127.0.0.1:5174/manager/colaboradores`: HTTP 200.

### Riscos ou pendencias

- Validar escopo real por departamento/carteira com MySQL e usuarios reais.
- Criar Playwright para consulta, usuario sem permissao, responsividade e navegacao para transferencias.
- Evoluir `ManagersModule`/`CalendarModule` antes de implementar o dashboard `/manager` completo.

## 2026-05-19 15:33

### Arquivos alterados

- `portal-sama-web/src/types/transfers.ts`
- `portal-sama-web/src/schemas/transfer.schema.ts`
- `portal-sama-web/src/services/transfers.service.ts`
- `portal-sama-web/src/pages/manager/ManagerTransfersPage.tsx`
- `portal-sama-web/src/app/router.tsx`
- `portal-sama-web/src/components/layout/navigation.tsx`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/paginas/manager-transfers.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-19-1533-manager-transfers-react/*`

### O que mudou

- Criada a rota React `/manager/transferencias` integrada ao `TransfersModule`.
- Adicionados tipos, schema Zod e service para dashboard, criacao e retorno de transferencias.
- A tela permite escolher departamento, origem, motivo, periodo/justificativa, empresas e destino por empresa, alem de listar sessoes recentes e retornar transferencias indeterminadas.
- Sidebar e Home ganharam atalho condicionado a `transfers.read`.

### Motivo da alteracao

Dar continuidade a migracao operacional de transferencias, aproveitando o backend seguro ja criado e mantendo fora do escopo a introducao da plataforma nesta rodada.

### Impacto esperado

- Gestores com permissao passam a ter uma primeira experiencia React para operar transferencias sem depender da tela HTML legada.
- Mutacoes seguem centralizadas no backend com CSRF, RBAC, escopo por departamento e auditoria.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou apos trocar `watch()` por `useWatch` e remover estados derivados em effects.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite manteve avisos conhecidos de chunk grande e asset de marca resolvido em runtime.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check` nos arquivos da fatia: passou com avisos LF/CRLF do Git no Windows.
- Dev server Vite iniciado em `http://127.0.0.1:5174` porque a porta 5173 ja estava em uso; HTTP 200 confirmado.

### Riscos ou pendencias

- Validar fluxo completo em homologacao com MySQL, seed `transfers.*`, JWT/CSRF reais e carteiras legadas.
- Criar Playwright para leitura, criacao, retorno, usuario sem permissao e responsividade.
- Formalizar modelagem/backfill de carteira para reduzir dependencia de metadata legado.

## 2026-05-19 12:16

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260519120000_add_transfer_sessions/migration.sql`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-api/src/modules/transfers/**`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/BANCO_DADOS_MYSQL_PRISMA.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-19-1216-transfers-module/*`

### O que mudou

- Criado `TransfersModule` com dashboard, criacao de transferencia e retorno manual por `/api-v2/transfers`.
- Adicionadas permissoes `transfers.read`, `transfers.create` e `transfers.return` ao catalogo RBAC; `MANAGER`, `ADMIN` e `DEV` passam a receber o fluxo pelo seed.
- Adicionado modelo Prisma `TransferSession` sobre `sama_transfer_sessions` e migration compativel com ambientes onde a tabela legada ja exista.
- Implementadas regras legadas de motivos, periodo, origem/destino, bloqueio de cliente ja em transferencia, reconciliacao de sessoes e auditoria de criacao/retorno.

### Motivo da alteracao

Destravar o backend transacional e auditavel para `Manager/manager-transfers.html` antes de migrar a tela React, conforme a orientacao de nao continuar a introducao da plataforma nesta rodada.

### Impacto esperado

- Gestores com permissao podem consultar e criar sessoes de transferencia dentro do proprio departamento.
- ADMIN/DEV podem operar por departamento informado.
- A futura rota React `/manager/transferencias` podera consumir contrato v2 sem depender das actions PHP `transfer_*`.

### Testes executados

- `npm.cmd run prisma:format`: passou.
- `npm.cmd run prisma:generate`: passou.
- `npm.cmd run prisma:validate`: passou.
- `npm.cmd run build`: passou.
- `npm.cmd run lint`: passou apos remover parametro nao usado.
- `npm.cmd test -- transfers.service.spec.ts`: passou, 4 testes.
- `npm.cmd test -- --runInBand`: passou, 26 suites e 148 testes.

### Riscos ou pendencias

- Aplicar migration/seed em MySQL de homologacao com backup antes de ligar a tela.
- Validar dados reais de `Client.metadata`/`User.metadata` e decidir se `client_departments`/carteira deve virar tabela formal.
- Criar frontend React `/manager/transferencias` e Playwright de gestor/admin/sem permissao.

## 2026-05-18 17:35

### Arquivos alterados

- `portal-sama-web/package.json`
- `portal-sama-web/package-lock.json`
- `portal-sama-web/index.html`
- `portal-sama-web/public/brand/*`
- `portal-sama-web/src/features/intro/*`
- `portal-sama-web/src/components/layout/AppLayout.tsx`
- `portal-sama-web/src/components/layout/Header.tsx`
- `portal-sama-web/src/components/layout/Sidebar.tsx`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `portal-sama-web/src/index.css`
- `portal-sama-web/src/types/auth.ts`
- `portal-sama-api/src/modules/auth/*`
- `portal-sama-api/src/common/decorators/current-user.decorator.ts`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/design/ANIMACOES_ENTRADA_E_MOTION_PORTAL_SAMA.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-18-1735-design-motion-intro/*`

### O que mudou

- Instalada a dependencia `framer-motion`.
- Criada a feature `src/features/intro` com splash padrao, boas-vindas, logo SVG animada em fitas azul/laranja, particulas discretas, orbitas, loading cue, reduced motion e reveal do conteudo.
- Adicionados assets locais de marca em `public/brand` e preload da logo vetorial.
- Refinado o layout autenticado com sidebar navy, header mais limpo, home mais profissional, stats com icones e atalhos com microinteracoes.
- Criado `PATCH /api-v2/me/preferences/intro` para marcar a intro como vista em `User.metadata.portalIntro`, com JWT, CSRF e auditoria.

### Motivo da alteracao

Aplicar a nova direcao visual aprovada em `docs/design/ANIMACOES_ENTRADA_E_MOTION_PORTAL_SAMA.md` sem migrar regras de negocio nem enfraquecer contratos de seguranca.

### Impacto esperado

- Usuarios autenticados passam a ver uma entrada visual consistente no portal privado.
- Usuarios com `welcomeAnimationSeen === false` veem a mensagem `Seja bem-vindo ao Portal Sama` e a preferencia e marcada como vista de forma idempotente.
- Login, assinatura publica, proposta publica e documentos publicos nao recebem intro.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou. Vite emitiu aviso de chunk acima de 500 kB.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `npm.cmd run lint` em `portal-sama-api`: passou.
- `npm.cmd run build` em `portal-sama-api`: passou.
- `npm.cmd test -- auth.service.spec.ts` em `portal-sama-api`: passou, 3 testes.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-api`: passou, 0 vulnerabilidades.
- `git diff --check`: passou; houve apenas aviso LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar visualmente no navegador real e criar Playwright para intro padrao, welcome, reduced motion, pular introducao e rotas publicas.
- Avaliar code splitting/lazy loading de rotas, pois o build Vite segue alertando chunk acima de 500 kB.
- Evoluir para o SVG final oficial da marca se a equipe fornecer uma versao vetorial em camadas.

## 2026-05-18 15:58

### Arquivos alterados

- `portal-sama-web/src/pages/departments/DepartmentClientsPage.tsx`
- `portal-sama-web/src/app/router.tsx`
- `portal-sama-web/src/components/layout/navigation.tsx`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/paginas/dptclient-clientdpto.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-18-15-58/*`

### O que mudou

- Criada a tela React `DepartmentClientsPage.tsx` para `/departamentos/clientes`.
- A tela consome `GET /api-v2/clients` para consulta departamental inicial com filtros, estatisticas, responsavel por metadata/backfill e atalhos para painel/documentos.
- `router.tsx`, `navigation.tsx` e `HomePage.tsx` passaram a expor a rota para sessoes com `clients.read`.

### Motivo da alteracao

Dar continuidade a migracao de `DptClient/clientdpto.html` sem inventar endpoints novos nem migrar atribuicao/transferencia de carteira sem contrato backend seguro.

### Impacto esperado

- Usuarios com `clients.read` passam a ter uma primeira experiencia React para consulta de clientes por contexto departamental.
- A leitura segue protegida pelo `ClientsModule`; a tela nao usa `innerHTML` nem persiste dados em storage do navegador.
- Atribuicao/transferencia de responsavel permanece pendente ate existir modelagem transacional, escopo real e auditoria.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou. Vite emitiu aviso de chunk acima de 500 kB.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou; houve apenas aviso LF/CRLF do Git no Windows.
- Artefatos locais: `.ai-tests/runs/2026-05-18-15-58/commands.md` e `summary.md`.

### Riscos ou pendencias

- Validar com API v2, MySQL/backfill, usuarios reais, escopo por departamento/carteira e HTTPS/homologacao.
- Criar Playwright para filtros, responsavel, usuario sem permissao e links para painel/documentos.
- Criar contrato backend seguro para atribuicao/transferencia de responsavel antes de substituir essa acao do legado.

## 2026-05-18 15:35

### Arquivos alterados

- `portal-sama-web/src/pages/ti/TiAccessPage.tsx`
- `portal-sama-web/src/app/router.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/paginas/ti-acesso-ti.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-18-15-35/*`

### O que mudou

- Criada a tela React `TiAccessPage.tsx` para `/ti/acessos`.
- A rota deixou de usar `PlaceholderPage` e passou a consumir os contratos de `AccessRequestsModule` para fila operacional, historico, filtros, estatisticas e decisoes de aprovar/rejeitar.
- A tela explicita que a base editavel de credenciais/acessos internos da TI permanece bloqueada ate existir cofre seguro, criptografia e auditoria de leitura.

### Motivo da alteracao

Dar continuidade a migracao da tela critica `TI/acesso-ti.html` sem migrar segredos para o frontend nem reutilizar KV generico do legado.

### Impacto esperado

- TI/ADMIN/DEV passam a ter uma primeira experiencia React para operar solicitacoes de acesso usando API v2.
- Decisoes seguem protegidas no backend por JWT, permissao granular, CSRF, escopo e auditoria.
- A base de credenciais da TI permanece fora do escopo desta entrega por risco de exposicao de segredo.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou. Vite emitiu aviso de chunk acima de 500 kB.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou; houve apenas aviso LF/CRLF do Git no Windows.
- Artefatos locais: `.ai-tests/runs/2026-05-18-15-35/commands.md` e `summary.md`.

### Riscos ou pendencias

- Validar com API v2, MySQL/backfill, usuarios reais, escopo por TI/gestor/departamento e HTTPS/homologacao.
- Criar Playwright para fila operacional, filtros, paginacao, approve/reject e usuario sem permissao.
- Modelar separadamente o cofre de credenciais da TI com criptografia/vault, auditoria de leitura, mascaramento e politica de retencao.

## 2026-05-18 08:46

### Arquivos alterados

- `portal-sama-web/src/types/documents.ts`
- `portal-sama-web/src/schemas/document.schema.ts`
- `portal-sama-web/src/services/documents.service.ts`
- `portal-sama-web/src/pages/documents/PublicDocumentsPage.tsx`
- `portal-sama-web/src/app/router.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/paginas/onboarding-documentos-cliente.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`

### O que mudou

- Criada a tela React publica `PublicDocumentsPage.tsx` para `/onboarding/publico/documentos/:token`.
- Adicionados tipos, schema Zod e service Axios/TanStack Query para `POST /api-v2/documents/public-upload?token=...`.
- A tela cobre tipo de documento, departamento opcional, selecao de arquivo, validacao visual de extensao/tamanho e historico local dos documentos recebidos na sessao.
- `router.tsx` deixou de usar placeholder nessa rota publica.

### Motivo da alteracao

Dar continuidade ao fluxo publico de documentos de onboarding, aproveitando o `DocumentsModule` ja existente para upload por `PublicToken`.

### Impacto esperado

- Clientes externos com link valido conseguem enviar documentos pelo frontend React.
- O token nao e salvo em storage do navegador e a tela nao renderiza HTML bruto.
- A validacao definitiva permanece no backend, com token opaco, scanner/quarentena, storage privado, auditoria e notificacao interna.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou. Vite emitiu aviso de chunk acima de 500 kB.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou; houve apenas avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar com API v2, MySQL/storage/ClamAV reais, tokens publicos reais/legados e HTTPS/homologacao.
- Criar endpoint/checklist publico minimo para o cliente enxergar documentos solicitados antes de enviar.
- Criar Playwright para token invalido, arquivo invalido e upload publico bem-sucedido.

## 2026-05-15 16:13

### Arquivos alterados

- `portal-sama-web/src/types/contracts.ts`
- `portal-sama-web/src/schemas/contract.schema.ts`
- `portal-sama-web/src/services/contracts.service.ts`
- `portal-sama-web/src/pages/contracts/ContractPage.tsx`
- `portal-sama-web/src/pages/contracts/PublicSignaturePage.tsx`
- `portal-sama-web/src/app/router.tsx`
- `portal-sama-web/src/components/layout/navigation.tsx`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `portal-sama-web/src/pages/legalization/LegalizationPage.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/paginas/legalizacao-contrato.md`
- `docs/paginas/legalizacao-assinatura.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`

### O que mudou

- Criadas as telas React `ContractPage.tsx` e `PublicSignaturePage.tsx`.
- Adicionados tipos, schema Zod e service Axios/TanStack Query para `/api-v2/contracts` e `/api-v2/public/signatures/:token`.
- A tela interna cobre filtros, estatisticas, listagem, criacao, edicao, geracao de snapshot HTML, renderizacao PDF, importacao/download de PDF privado e envio para assinatura por token.
- A tela publica cobre consulta por token, exibicao textual do contrato e registro de assinatura.
- `router.tsx`, `navigation.tsx`, `HomePage.tsx` e `LegalizationPage.tsx` passaram a expor as rotas de contratos/assinatura.

### Motivo da alteracao

Dar continuidade a migracao frontend de `Legalizacao/contrato.html` e `Legalizacao/assinatura.html`, aproveitando o `ContractsModule` ja implementado no backend.

### Impacto esperado

- Contratos deixam de ser placeholder no React e passam a consumir contratos tipados da API v2.
- Mutacoes internas usam CSRF pelo client centralizado; permissoes no frontend sao apenas UX.
- O fluxo publico por token nao grava token em storage do navegador e nao renderiza HTML bruto com `innerHTML`.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou. Vite emitiu aviso de chunk acima de 500 kB.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou; houve apenas avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar com API v2, MySQL/backfill reais, usuarios com permissoes `contracts.*`, Legal Doc Service real e HTTPS/homologacao.
- Validar compatibilidade com tokens/contratos legados, PDF assinado real e storage privado real.
- Criar Playwright para listagem, criacao, edicao, geracao, PDF, envio por link e assinatura publica.

## 2026-05-15 15:06

### Arquivos alterados

- `portal-sama-web/src/types/proposals.ts`
- `portal-sama-web/src/schemas/proposal.schema.ts`
- `portal-sama-web/src/services/proposals.service.ts`
- `portal-sama-web/src/pages/proposals/ProposalPage.tsx`
- `portal-sama-web/src/pages/proposals/PublicProposalPage.tsx`
- `portal-sama-web/src/app/router.tsx`
- `portal-sama-web/src/components/layout/navigation.tsx`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `portal-sama-web/src/pages/legalization/LegalizationPage.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/paginas/legalizacao-proposta.md`
- `docs/paginas/onboarding-proposta-cliente.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`

### O que mudou

- Criadas as telas React `ProposalPage.tsx` e `PublicProposalPage.tsx`.
- Adicionados tipos, schema Zod e service Axios/TanStack Query para `/api-v2/proposals` e `/api-v2/public/proposals/:token`.
- A tela interna cobre filtros, estatisticas, listagem, criacao, edicao, detalhe, envio por link publico e aprovacao/rejeicao interna por permissao.
- A tela publica cobre consulta por token, exibicao textual dos campos comerciais, aprovacao e solicitacao de ajuste.
- `router.tsx`, `navigation.tsx`, `HomePage.tsx` e `LegalizationPage.tsx` passaram a expor as rotas de propostas.

### Motivo da alteracao

Dar continuidade a migracao frontend de `Legalizacao/proposta.html` e `Onboarding/proposta-cliente.html`, aproveitando o `ProposalsModule` ja implementado no backend.

### Impacto esperado

- Propostas deixam de ser placeholder no React e passam a consumir contratos tipados da API v2.
- Mutacoes internas usam CSRF pelo client centralizado; permissoes no frontend sao apenas UX.
- O fluxo publico por token nao grava token em storage do navegador e nao renderiza HTML bruto.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou. Vite emitiu aviso de chunk acima de 500 kB.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou; houve apenas avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar com API v2, MySQL/backfill reais, usuarios com permissoes `proposals.*` e HTTPS/homologacao.
- Validar compatibilidade com tokens/propostas legadas e fluxo publico real.
- Criar Playwright para listagem, criacao, edicao, envio por link e resposta publica.

## 2026-05-15 14:13

### Arquivos alterados

- `portal-sama-web/src/types/admin.ts`
- `portal-sama-web/src/schemas/admin.schema.ts`
- `portal-sama-web/src/services/admin.service.ts`
- `portal-sama-web/src/pages/dev/DevAdminPage.tsx`
- `portal-sama-web/src/app/router.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/paginas/dev-dev.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-15-14-13-dev-admin-react/*`

### O que mudou

- Criada a tela React `DevAdminPage.tsx` para `/dev`.
- Adicionados tipos, schema Zod e service Axios/TanStack Query para `/api-v2/users`, `/api-v2/roles` e `/api-v2/permissions`.
- A tela cobre filtros, estatisticas, listagens, criacao/edicao de usuario, status, roles do usuario, criacao/edicao de role, permissoes da role e criacao/edicao de permissao.
- `router.tsx` passou a usar a tela real no lugar do placeholder de DEV.

### Motivo da alteracao

Dar continuidade a migracao frontend de `DEV/dev.html`, aproveitando os contratos administrativos ja implementados em `UsersModule`, `RolesModule` e `PermissionsModule`.

### Impacto esperado

- DEV/admin deixa de ser placeholder no React e passa a consumir contratos tipados da API v2.
- Mutacoes usam CSRF pelo client centralizado; permissoes no frontend sao apenas UX.
- A migracao de usuarios legados, presenca/estatisticas e validacao real em homologacao seguem como frentes separadas.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: falhou inicialmente por tipagem union no helper de formulario; apos ajuste, passou. Vite emitiu aviso de chunk acima de 500 kB.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou; houve apenas aviso LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar com API v2, MySQL, seed RBAC, usuarios reais e HTTPS/homologacao.
- Migrar/backfill usuarios legados, presenca online e estatisticas administrativas do legado.
- Criar Playwright para listagem, criacao, edicao, status, roles e permissoes.

## 2026-05-15 13:51

### Arquivos alterados

- `portal-sama-web/src/types/onboarding.ts`
- `portal-sama-web/src/schemas/onboarding.schema.ts`
- `portal-sama-web/src/services/onboarding.service.ts`
- `portal-sama-web/src/pages/onboarding/OnboardingProcessesPage.tsx`
- `portal-sama-web/src/app/router.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/SEGURANCA.md`
- `docs/paginas/onboarding-processo.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-15-13-51-onboarding-react/*`

### O que mudou

- Criada a tela React `OnboardingProcessesPage.tsx` para `/onboarding/processos`.
- Adicionados tipos, schema Zod e service Axios/TanStack Query para `/api-v2/onboarding/processes`.
- A tela cobre filtros, estatisticas, listagem, criacao, edicao, arquivamento, mudanca de status/etapa, timeline e vinculo leve de documentos/requisitos/token publico por permissao.
- `router.tsx` passou a usar a tela real no lugar do placeholder de onboarding.

### Motivo da alteracao

Dar continuidade a migracao frontend de `Onboarding/processo.html`, aproveitando o `OnboardingModule` ja implementado no backend.

### Impacto esperado

- Onboarding deixa de ser placeholder no React e passa a consumir contratos tipados da API v2.
- Mutacoes usam CSRF pelo client centralizado; permissoes no frontend sao apenas UX.
- Fluxos publicos de documentos/proposta continuam como fatias separadas.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite emitiu aviso de chunk acima de 500 kB.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou; houve apenas aviso LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar com API v2, MySQL/backfill reais, usuarios com permissoes `onboarding.*` e HTTPS/homologacao.
- Validar maquina de estados completa, documentos publicos, proposta publica e fluxos integrados com dados reais.
- Criar Playwright para listagem, criacao, edicao, status, timeline, documentos e arquivamento.

## 2026-05-15 13:35

### Arquivos alterados

- `portal-sama-web/src/types/legalization.ts`
- `portal-sama-web/src/schemas/legalization.schema.ts`
- `portal-sama-web/src/services/legalization.service.ts`
- `portal-sama-web/src/pages/legalization/LegalizationPage.tsx`
- `portal-sama-web/src/app/router.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`

### O que mudou

- Criada a tela React `LegalizationPage.tsx` para `/legalizacao`.
- Adicionados tipos, schema Zod e service Axios/TanStack Query para `/api-v2/legalization/processes` e `/api-v2/legalization/templates`.
- A tela cobre filtros, paginacao, criacao, edicao, arquivamento, mudanca de status/etapa, consulta de timeline e gestao de templates por permissao.
- `router.tsx` passou a usar a tela real no lugar do placeholder de legalizacao.

### Motivo da alteracao

Dar continuidade a migracao frontend de `Legalizacao/legalizacao.html`, aproveitando o `LegalizationModule` ja implementado no backend.

### Impacto esperado

- Legalizacao deixa de ser placeholder no React e passa a consumir contratos tipados da API v2.
- Mutacoes usam CSRF pelo client centralizado; permissoes no frontend sao apenas UX.
- Propostas, contratos e assinatura publica continuam como proximas fatias separadas.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite emitiu aviso de chunk acima de 500 kB.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou; houve apenas avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar com API v2, MySQL/backfill reais, usuarios com permissoes `legalization.*` e HTTPS/homologacao.
- Validar Legal Doc Service/PDF e fluxo fim a fim com propostas, contratos e assinatura.
- Criar Playwright para listagem, criacao, edicao, status, timeline e templates.

## 2026-05-15 09:44

### Arquivos alterados

- `portal-sama-web/src/types/access-requests.ts`
- `portal-sama-web/src/schemas/access-request.schema.ts`
- `portal-sama-web/src/services/access-requests.service.ts`
- `portal-sama-web/src/pages/access-requests/AccessRequestPage.tsx`
- `portal-sama-web/src/app/router.tsx`
- `portal-sama-web/src/components/layout/navigation.tsx`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/paginas/solicitacao-acesso.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-15-09-44-access-requests-react/*`

### O que mudou

- Criada a tela React `AccessRequestPage.tsx` para `/solicitacao-acesso`.
- Adicionados tipos, schema Zod e service Axios/TanStack Query para `/api-v2/access-requests`.
- A tela cobre envio por colaborador, envio por gestor para equipe, acompanhamento da ultima solicitacao, filtros de historico, fila de aprovacoes e decisoes de aprovar/rejeitar por permissao.
- `router.tsx` passou a usar a tela real; `navigation.tsx` e `HomePage.tsx` passaram a liberar `Solicitacoes` para sessoes autenticadas porque criacao/acompanhamento individual nao dependem de `access_requests.read`.
- O atalho de documentos da Home foi ajustado para `/documentos`, evitando apontar para `/clientes/novo/painel`.

### Motivo da alteracao

Dar continuidade a migracao frontend prioritaria de `SolicitacaoAcesso/solicitacao-acesso.html`, aproveitando o `AccessRequestsModule` ja implementado no backend.

### Impacto esperado

- Solicitacoes de acesso deixam de ser placeholder no React e passam a consumir contratos tipados da API v2.
- Mutacoes usam CSRF pelo client centralizado; permissoes no frontend sao apenas UX.
- A base editavel de acessos internos da TI continua fora do escopo ate existir modelagem segura com criptografia/vault.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou; Vite emitiu aviso de chunk acima de 500 kB.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou; houve apenas avisos LF/CRLF do Git no Windows.

### Riscos ou pendencias

- Validar a tela com API v2 rodando, usuario real, MySQL/backfill reais e HTTPS/homologacao.
- Testar escopo por colaborador, gestor, TI, ADMIN/DEV e usuario sem permissao.
- Criar Playwright para envio, ultima solicitacao, fila de aprovacao, approve/reject e estados de erro.
- Modelar `/ti/acessos` com criptografia/vault antes de migrar segredos de TI.

## 2026-05-15 09:16

### Arquivos alterados

- `portal-sama-web/src/types/documents.ts`
- `portal-sama-web/src/schemas/document.schema.ts`
- `portal-sama-web/src/services/documents.service.ts`
- `portal-sama-web/src/pages/clients/ClientDocumentsPanel.tsx`
- `portal-sama-web/src/pages/clients/ClientDashboardPage.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/paginas/client-painel.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-15-09-16-client-documents-react/*`

### O que mudou

- Criada a secao `ClientDocumentsPanel.tsx` dentro do painel React do cliente.
- Adicionados tipos, schema Zod e service para o contrato `/api-v2/documents`.
- A secao cobre checklist por cliente, estatisticas, upload multipart, download protegido, revisao de status e requisito customizado.
- `ClientDashboardPage.tsx` passou a renderizar a nova secao abaixo do resumo operacional.

### Motivo da alteracao

Dar continuidade a migracao do `Client/painel.html`, fechando a primeira integracao frontend com o `DocumentsModule` depois da tela base de clientes.

### Impacto esperado

- O painel do cliente passa a usar o checklist tipado da API v2, reduzindo dependencia do fluxo legado de `api/client_documents.php`.
- Upload/revisao/requisitos usam CSRF pelo client centralizado e permissoes de UX alinhadas ao backend.
- A tela fica pronta para validacao real de storage, scanner, escopo e auditoria em homologacao.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou. A primeira execucao teve um aviso de hook; apos ajuste com `useMemo`, passou sem avisos.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou, com avisos LF/CRLF do Windows em docs e arquivos React.

### Riscos ou pendencias

- Validar com API v2, usuario real, MySQL/storage reais, ClamAV/EICAR e HTTPS/homologacao.
- Testar IDOR/escopo cliente-gestor-usuario com dados persistidos.
- Integrar certificados por cliente, acessos e vida da empresa no painel.
- Criar Playwright para checklist, upload, download e revisao.

## 2026-05-15 08:59

### Arquivos alterados

- `portal-sama-web/src/app/router.tsx`
- `portal-sama-web/src/components/layout/navigation.tsx`
- `portal-sama-web/src/index.css`
- `portal-sama-web/src/types/clients.ts`
- `portal-sama-web/src/schemas/client.schema.ts`
- `portal-sama-web/src/services/clients.service.ts`
- `portal-sama-web/src/pages/clients/ClientsPage.tsx`
- `portal-sama-web/src/pages/clients/ClientDashboardPage.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/paginas/client-clientes.md`
- `docs/paginas/client-painel.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-15-08-59-clients-react/*`

### O que mudou

- Criadas as telas React `ClientsPage.tsx` e `ClientDashboardPage.tsx`.
- Adicionados tipos, schema Zod e service Axios/TanStack Query para `/api-v2/clients`.
- `/clientes` cobre filtros, paginacao, cadastro, edicao e arquivamento logico por permissao.
- `/clientes/:clientId/painel` cobre primeira visao de dados cadastrais e resumo operacional do dashboard.
- O item de menu `Documentos` deixou de apontar para `/clientes/novo/painel`, evitando chamada a um cliente inexistente depois que o painel passou a ser real.
- `index.css` passou a padronizar `select`/`textarea` nos formularios.

### Motivo da alteracao

Dar continuidade a migracao frontend registrada para `Client/clientes.html`, `Client/painel.html` e `DEV/dev-novo-cliente.html`, aproveitando o `ClientsModule` ja implementado no backend.

### Impacto esperado

- Clientes deixam de ser placeholder no React e passam a consumir contrato tipado da API v2.
- Mutacoes de cliente usam CSRF pelo client centralizado; permissoes no frontend sao apenas UX.
- Painel do cliente ganha base para integrar documentos, certificados, acessos e vida da empresa em fatias seguintes.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: falhou inicialmente por tipo do schema Zod/React Hook Form; apos ajuste, passou.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou, com avisos LF/CRLF do Windows em docs e arquivos React.

### Riscos ou pendencias

- Validar as telas com API v2 rodando, usuario real, MySQL/storage reais e HTTPS/homologacao.
- Planejar backfill de clientes legados e vinculos cliente-usuario/gestor.
- Integrar documentos/upload/download, certificados por cliente, acessos e vida da empresa no painel.
- Criar Playwright para login, `/clientes`, permissoes de criacao/edicao e painel do cliente.

## 2026-05-15 08:33

### Arquivos alterados

- `portal-sama-web/src/app/router.tsx`
- `portal-sama-web/src/types/certificates.ts`
- `portal-sama-web/src/schemas/certificate.schema.ts`
- `portal-sama-web/src/services/certificates.service.ts`
- `portal-sama-web/src/pages/certificates/CertificatesPage.tsx`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/paginas/dptclient-certificados-digitais.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-15-08-33-certificates-react/*`

### O que mudou

- Criada a tela React `CertificatesPage.tsx` para `/certificados-digitais`.
- Adicionados tipos, schemas Zod e service Axios/TanStack Query para `/api-v2/certificates`.
- A tela cobre filtros, listagem, cadastro multipart `.p12/.pfx`, download protegido, edicao de metadados/arquivo, rotacao de senha e remocao logica por permissao.
- `router.tsx` passou a usar a tela real no lugar do placeholder.

### Motivo da alteração

Dar continuidade a migracao frontend prioritaria registrada para `DptClient/certificados-digitais.html`, aproveitando o `CertificatesModule` ja implementado no backend.

### Impacto esperado

- Certificados digitais passam a ter primeira experiencia React funcional.
- Senhas continuam fora das respostas/listagens do frontend; a rotacao e uma acao explicita.
- As permissoes no frontend sao apenas UX; o backend segue validando JWT, CSRF, `certificates.*`, escopo, storage privado, criptografia e auditoria.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou, com aviso LF/CRLF do Windows em `portal-sama-web/src/app/router.tsx`.

### Riscos ou pendências

- Validar a tela com API v2 rodando, usuario real, MySQL/storage reais e HTTPS/homologacao.
- Planejar backfill dos certificados legados.
- Criar Playwright para login, acesso a `/certificados-digitais`, permissao sem `certificates.manage`, cadastro e download.

## 2026-05-14 14:54

### Arquivos alterados

- `portal-sama-web/**`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/EASYPANEL_DEPLOY.md`
- `.ai-tests/runs/2026-05-14-14-54-frontend-react/*`

### O que mudou

- Criada a fundacao `portal-sama-web` com React, TypeScript, Vite, Tailwind, React Router, Axios, TanStack Query, Zustand, React Hook Form/Zod e lucide-react.
- Implementado login React inicial em `/login`, consumindo `/api-v2/auth/csrf`, `/login`, `/refresh`, `/logout`, `/me` e `/forgot-password`.
- Criado layout interno autenticado com menu por permissao, `PermissionGate`, `StatusBadge`, home inicial e rotas reservadas para as telas da matriz HTML -> React.
- Adicionados `Dockerfile`, `nginx.conf` e `.env.example` para o frontend.

### Motivo da alteracao

Dar continuidade a pendencia documentada de criacao do frontend React/Vite apos a consolidacao das principais fatias backend em `/api-v2`.

### Impacto esperado

- O projeto passa a ter um frontend React versionavel para iniciar a migracao visual sem desligar o legado HTML/PHP.
- O fluxo React evita persistir access token em `localStorage`/`sessionStorage`; o token fica em memoria e o refresh permanece no cookie HttpOnly da API v2.
- As telas de negocio ainda nao estao conectadas aos dados reais; a entrega prepara a base de rotas, layout e sessao.

### Testes executados

- `npm.cmd run lint` em `portal-sama-web`: passou.
- `npm.cmd run build` em `portal-sama-web`: passou.
- `npm.cmd audit --audit-level=moderate` em `portal-sama-web`: passou, 0 vulnerabilidades.
- `git diff --check`: passou.

### Riscos ou pendencias

- Validar login/refresh/logout em navegador HTTPS com API v2, MySQL e usuario real.
- Integrar telas reais por prioridade: documentos, certificados, solicitacoes/TI, legalizacao, onboarding, auditoria e DEV/admin.
- Criar Playwright para login, protecao de rota, logout e fluxo publico.

## 2026-05-14 13:54

### Arquivos alterados

- `portal-sama-api/src/modules/contracts/contract-pdf-render.service.ts`
- `portal-sama-api/src/modules/contracts/contract-pdf-render.service.spec.ts`
- `portal-sama-api/src/modules/contracts/dto/render-contract-pdf.dto.ts`
- `portal-sama-api/src/modules/contracts/contracts.controller.ts`
- `portal-sama-api/src/modules/contracts/contracts.module.ts`
- `portal-sama-api/src/modules/contracts/contracts.service.ts`
- `portal-sama-api/src/modules/contracts/contracts.service.spec.ts`
- `portal-sama-api/src/config/env.schema.ts`
- `portal-sama-api/.env.example`
- `portal-sama-api/test/app.e2e-spec.ts`
- documentacao de status, pendencias, testes, changelog e inventarios.

### O que mudou

- Criado `POST /api-v2/contracts/:id/render-pdf` para renderizar o HTML salvo do contrato via Legal Doc Service externo.
- Adicionado `ContractPdfRenderService`, com falha fechada quando `LEGAL_DOC_SERVICE_ENABLED`/`LEGAL_RENDER_V2_ENABLED` nao estao ativos ou quando faltam URL/credencial.
- O PDF base64 retornado pelo renderizador e validado como PDF, passa pelo validador de conteudo ativo, e e salvo no storage privado de contratos.
- `ContractsService.renderPdf` registra metadados de geracao/renderizacao, hash do documento, hash/tamanho do PDF, engine/paginas e auditoria `contracts.render_pdf` sem storage key.

### Testes executados

- `npm.cmd test -- contracts.service.spec.ts contract-pdf-file-validator.service.spec.ts contract-pdf-render.service.spec.ts --runInBand`: passou, 13 testes.
- `npm.cmd exec tsc -- --noEmit`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd run build`: passou.
- `npm.cmd run test:e2e -- --runInBand`: passou, 136 testes.
- `npm.cmd test -- --runInBand`: passou, 25 suites/144 testes.

### Riscos ou pendencias

- A rota depende de Legal Doc Service real configurado em ambiente; por padrao permanece desativada.
- Falta homologar fidelidade visual, fontes, margens, paginas, tempo de render e comportamento com HTMLs reais.
- Frontend, backfill legado e validacao com MySQL/storage reais seguem pendentes.

## 2026-05-14 13:42

### Arquivos alterados

- `portal-sama-api/src/modules/contracts/contract-pdf-file-validator.service.ts`
- `portal-sama-api/src/modules/contracts/contract-pdf-storage.service.ts`
- `portal-sama-api/src/modules/contracts/contracts.controller.ts`
- `portal-sama-api/src/modules/contracts/contracts.service.ts`
- `portal-sama-api/src/modules/contracts/contracts.module.ts`
- `portal-sama-api/src/modules/contracts/contracts.types.ts`
- `portal-sama-api/src/modules/contracts/contracts.service.spec.ts`
- `portal-sama-api/src/modules/contracts/contract-pdf-file-validator.service.spec.ts`
- `portal-sama-api/src/config/env.schema.ts`
- `portal-sama-api/.env.example`
- `portal-sama-api/test/app.e2e-spec.ts`
- documentacao de status, pendencias, testes, changelog e inventarios.

### O que mudou

- Criado `POST /api-v2/contracts/:id/pdf` para importar PDF de contrato via multipart com JWT `contracts.generate` e CSRF.
- Criado `GET /api-v2/contracts/:id/pdf` para download protegido com JWT `contracts.read`, escopo do contrato, `application/pdf` e `Cache-Control: no-store`.
- O validador aceita somente `.pdf`, MIME permitido, assinatura `%PDF`, tamanho maximo configuravel e bloqueia marcadores ativos conhecidos como JavaScript, OpenAction, Launch, EmbeddedFile, RichMedia e XFA.
- O storage de PDFs de contrato fica em area privada configuravel por `CONTRACT_PDF_STORAGE_PATH`, usando chave interna por cliente/contrato e path traversal guard.
- A auditoria registra hash/tamanho e se houve substituicao, sem expor `storageKey`; as respostas mostram somente `pdfDownloadUrl`/`pdf_download_url`.

### Testes executados

- `npm.cmd test -- contracts.service.spec.ts contract-pdf-file-validator.service.spec.ts --runInBand`: passou, 10 testes.
- `npm.cmd exec tsc -- --noEmit`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd run build`: passou.
- `npm.cmd run test:e2e -- --runInBand`: passou, 135 testes.
- `npm.cmd test -- --runInBand`: passou, 24 suites/141 testes.

### Riscos ou pendencias

- A validacao de PDF e defensiva, mas nao substitui scanner/normalizador dedicado em ambiente real.
- Renderizacao HTML para PDF ainda depende da escolha de engine e sandbox.
- Falta validar storage/MySQL reais, backfill legado e integracao frontend.

## 2026-05-14 10:39

### Arquivos alterados

- `portal-sama-api/src/modules/notifications/notifications.controller.ts`
- `portal-sama-api/src/modules/notifications/notifications.service.ts`
- `portal-sama-api/src/modules/notifications/notifications.service.spec.ts`
- `portal-sama-api/src/modules/notifications/notifications.types.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- `global.js`
- `global.css`
- documentacao de status, pendencias, testes e inventarios.

### O que mudou

- Criado `POST /api-v2/notifications/push/test` para testar Web Push do usuario autenticado com JWT `notifications.read` e CSRF.
- Quando VAPID esta habilitado, o endpoint cria uma notificacao direcionada ao usuario atual, registra auditoria `notifications.push_test`, publica evento no stream e retorna resumo do envio Web Push.
- Quando VAPID esta ausente, o endpoint responde `push_disabled` sem criar linha no banco.
- O painel legado de notificacoes ganhou botao `Testar` apos o opt-in, reaproveitando a ponte API v2 para sessao, CSRF e assinatura Push API.

### Testes executados

- `node --check global.js`: passou.
- `npm.cmd test -- notifications.service.spec.ts notifications-push.service.spec.ts --runInBand`: passou, 17 testes.
- `npm.cmd exec tsc -- --noEmit`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd run build`: passou.
- `npm.cmd run test:e2e -- --runInBand`: passou, 132 testes.
- `npm.cmd test -- --runInBand`: passou, 23 suites/137 testes.

### Riscos ou pendencias

- O recebimento real ainda precisa de VAPID real e HTTPS em homologacao/producao.
- Broker/event bus multi-instancia segue como decisao operacional se updates em tempo real entre instancias forem exigidos.

## 2026-05-14 10:25

### Arquivos alterados

- `portal-sama-api/src/modules/documents/documents.module.ts`
- `portal-sama-api/src/modules/documents/documents.service.ts`
- `portal-sama-api/src/modules/documents/documents.service.spec.ts`
- `portal-sama-api/src/modules/certificates/certificates.module.ts`
- `portal-sama-api/src/modules/certificates/certificates.service.ts`
- `portal-sama-api/src/modules/certificates/certificates.service.spec.ts`
- documentacao de status, pendencias e testes.

### O que mudou

- `DocumentsModule` e `CertificatesModule` passaram a importar `NotificationsModule`.
- `DocumentsService` emite notificacoes para requisito customizado, link publico criado/revogado, upload interno, upload publico, revisao de status e arquivamento.
- `CertificatesService` emite notificacoes para cadastro, atualizacao, rotacao de senha e remocao de certificado digital.
- Os payloads evitam dados sensiveis: token bruto, storage key, conteudo de arquivo, senha bruta e senha criptografada nao sao enviados nas notificacoes.
- As falhas de notificacao ficam isoladas para nao bloquear o fluxo principal de documentos ou certificados.

### Testes executados

- `npm.cmd test -- documents.service.spec.ts certificates.service.spec.ts --runInBand`: passou, 32 testes.
- `npm.cmd exec tsc -- --noEmit`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd run build`: passou.
- `npm.cmd run test:e2e -- --runInBand`: passou, 130 testes.
- `npm.cmd test -- --runInBand`: passou, 23 suites/135 testes.
- `git diff --check`: passou, apenas avisos LF/CRLF do Windows.

### Riscos ou pendencias

- Recebimento Web Push em navegador real ainda depende de homologacao HTTPS e VAPID configurado.
- Broker/event bus multi-instancia segue como decisao operacional se updates em tempo real entre instancias forem exigidos.
- Backfill, validacao com dados reais e integracao frontend continuam pendentes nas frentes de migracao.

## 2026-05-14 10:13

### Arquivos alterados

- `portal-sama-api/src/modules/onboarding/onboarding.module.ts`
- `portal-sama-api/src/modules/onboarding/onboarding.service.ts`
- `portal-sama-api/src/modules/onboarding/onboarding.service.spec.ts`
- `portal-sama-api/src/modules/legalization/legalization.module.ts`
- `portal-sama-api/src/modules/legalization/legalization.service.ts`
- `portal-sama-api/src/modules/legalization/legalization.service.spec.ts`
- documentacao de status, pendencias e testes.

### O que mudou

- `OnboardingModule` e `LegalizationModule` passaram a importar `NotificationsModule`.
- `OnboardingService` emite notificacoes em criacao, mudanca de status, vinculacao de documentos e arquivamento.
- `LegalizationService` emite notificacoes em criacao, mudanca de status e arquivamento.
- As notificacoes procuram o responsavel por `responsibleId/updatedById/createdById`, usam fallback para o departamento do processo e isolam falhas para nao bloquear o fluxo principal.

### Testes executados

- `npm.cmd test -- onboarding.service.spec.ts legalization.service.spec.ts --runInBand`: passou, 19 testes.
- `npm.cmd exec tsc -- --noEmit`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd run build`: passou.
- `npm.cmd run test:e2e -- --runInBand`: passou, 130 testes.

### Riscos ou pendencias

- Emissores especificos de documentos e certificados foram enderecados na entrada de 2026-05-14 10:25.
- O recebimento Web Push em navegador real ainda depende de homologacao HTTPS e VAPID configurado.

## 2026-05-14 10:04

### Arquivos alterados

- `portal-sama-api/src/modules/proposals/proposals.module.ts`
- `portal-sama-api/src/modules/proposals/proposals.service.ts`
- `portal-sama-api/src/modules/proposals/proposals.service.spec.ts`
- `portal-sama-api/src/modules/contracts/contracts.module.ts`
- `portal-sama-api/src/modules/contracts/contracts.service.ts`
- `portal-sama-api/src/modules/contracts/contracts.service.spec.ts`
- documentacao de status, pendencias e testes.

### O que mudou

- `ProposalsModule` e `ContractsModule` passaram a importar `NotificationsModule`.
- Quando o cliente aprova ou solicita ajuste em proposta publica, `ProposalsService` emite notificacao interna para o responsavel localizado por `updatedById/createdById`, com fallback para `Legalizacao`.
- Quando o cliente assina contrato publico, `ContractsService` emite notificacao interna equivalente para o responsavel/departamento.
- As notificacoes usam ator tecnico `portal-sama`, permissao `notifications.create`, dedupe por token publico e tratamento isolado de falhas.

### Testes executados

- `npm.cmd test -- proposals.service.spec.ts contracts.service.spec.ts --runInBand`: passou, 12 testes.
- `npm.cmd exec tsc -- --noEmit`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd run build`: passou.
- `npm.cmd run test:e2e -- --runInBand`: passou, 130 testes.
- `git diff --check`: passou, apenas avisos LF/CRLF do Windows.

### Riscos ou pendencias

- Ainda falta validar recebimento Web Push em navegador real com VAPID configurado.
- Demais emissores automaticos seguem pendentes para onboarding, legalizacao/processos, documentos e certificados.

## 2026-05-14 09:49

### Arquivos alterados

- `auth.js`
- `Login.js`
- `global.js`
- `portal-sama-api/package.json`
- `portal-sama-api/.env.example`
- `portal-sama-api/scripts/generate-vapid-keys.js`
- documentacao de status, pendencias e testes.

### O que mudou

- `auth.js` agora guarda access token, expiracao e CSRF da API v2 separados do CSRF PHP legado.
- Adicionados helpers `buildApiV2Headers`, `ensureApiV2CsrfToken`, `refreshApiV2Session` e `logoutApiV2`.
- `Login.js` salva `expiresIn` e CSRF retornado pelo login API v2.
- `global.js` tenta renovar a sessao API v2 antes de assinar Web Push e faz retry de chamadas API v2 que retornarem `401`.
- Adicionado `npm run webpush:vapid:generate` para gerar chaves VAPID operacionais sem versionar segredo real.

### Testes executados

- `node --check auth.js`: passou.
- `node --check Login.js`: passou.
- `node --check global.js`: passou.
- `node --check portal-sama-api/scripts/generate-vapid-keys.js`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd run build`: passou.

### Riscos ou pendencias

- Validar manualmente refresh/logout API v2 no navegador real junto do login legado.
- Configurar VAPID real em homologacao/producao e testar recebimento com portal fechado.
- O frontend React definitivo ainda deve substituir esta ponte temporaria do legado.

## 2026-05-14 09:35

### Arquivos alterados

- `portal-sama-api/src/modules/notifications/**`
- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260514092000_add_browser_push_subscriptions/migration.sql`
- `portal-sama-api/prisma/migrations/20260514093500_align_browser_push_updated_at/migration.sql`
- `portal-sama-api/src/modules/auth/auth.controller.ts`
- `portal-sama-api/src/modules/auth/auth.service.ts`
- `portal-sama-api/src/modules/auth/auth.types.ts`
- `portal-sama-api/src/modules/auth/guards/jwt-auth.guard.ts`
- `portal-sama-api/src/config/env.schema.ts`
- `portal-sama-api/.env.example`
- `portal-sama-api/package.json`
- `portal-sama-api/package-lock.json`
- `global.js`
- `auth.js`
- `Login.js`
- `sama-push-sw.js`
- documentacao de status, seguranca, endpoints, pendencias, matriz de migracao e relatorio de testes.

### O que mudou

- Criada persistencia `browser_push_subscriptions` para assinaturas Web Push por navegador, com hash do endpoint, usuario, departamento, user agent e controle de falhas/revogacao.
- Adicionado `NotificationsPushService` usando `web-push`, VAPID configuravel por env e revogacao automatica de subscriptions com erro 404/410 ou falhas repetidas.
- Novas rotas: `GET /api-v2/notifications/push/public-key`, `POST /api-v2/notifications/push/subscribe` e `POST /api-v2/notifications/push/unsubscribe`.
- `NotificationsService.create` passou a despachar Web Push em segundo plano apos criar/publicar a notificacao.
- `global.js` registra Service Worker e assina Push API quando o usuario ativa notificacoes; `sama-push-sw.js` exibe notificacoes recebidas mesmo com o portal fechado.
- `Login.js` tenta autenticar tambem em `/api-v2/auth/login` para obter bearer token quando a API v2 estiver disponivel, mantendo o login PHP legado como fluxo principal.
- Cookie CSRF da API v2 passou a ter path `/api-v2`, permitindo validacao em mutacoes fora de `/api-v2/auth`.

### Testes executados

- `node --check global.js`: passou.
- `node --check auth.js`: passou.
- `node --check Login.js`: passou.
- `node --check sama-push-sw.js`: passou.
- `npm.cmd run prisma:format`: passou.
- `npm.cmd run prisma:generate`: passou.
- `npm.cmd run prisma:validate`: passou.
- `npm.cmd test -- notifications-push.service.spec.ts notifications.service.spec.ts --runInBand`: passou, 15 testes.
- `npm.cmd run lint`: passou.
- `npm.cmd run test:e2e -- --runInBand`: passou, 130 testes.
- `npm.cmd test -- --runInBand`: passou, 23 suites/131 testes.
- `npm.cmd run build`: passou.
- `npm.cmd audit`: passou, 0 vulnerabilidades.
- `npm.cmd run prisma:migrate:deploy`: passou, aplicando as migrations de Web Push.
- `npm.cmd run prisma:migrate:status`: passou, banco atualizado.
- `npx.cmd prisma migrate diff --from-schema-datasource prisma/schema.prisma --to-schema-datamodel prisma/schema.prisma --exit-code`: passou, sem diferencas.

### Riscos ou pendencias

- Produzir e configurar chaves VAPID reais em homologacao/producao.
- Validar manualmente em HTTPS com navegadores suportados, permissao concedida, portal fechado e usuario autenticado via API v2.
- O login legado segue principal; a assinatura Web Push so cadastra no backend quando tambem houver access token da API v2.
- Ainda faltam emissores dos demais modulos e decisao sobre broker/event bus para multi-instancia.

## 2026-05-14 09:09

### Arquivos alterados

- `global.js`
- `global.css`
- documentacao de status, seguranca, pendencias, matriz frontend e relatorio de testes.

### O que mudou

- Adicionado botao `Ativar`/`Desativar` no painel legado de notificacoes para pedir permissao de notificacoes nativas do navegador.
- A preferencia fica isolada por usuario em `localStorage`.
- Quando a permissao esta concedida e o portal esta sem foco ou em aba oculta, novas notificacoes tambem disparam `new Notification(...)`.
- Quando a permissao esta bloqueada no navegador, o controle fica desabilitado como `Bloqueado`.

### Testes executados

- `node --check global.js`: passou.

### Riscos ou pendencias

- Esta entrega depende de alguma aba do portal aberta. Push com portal fechado ainda exige Service Worker, Push API, VAPID, cadastro de subscriptions e envio Web Push pelo backend.

## 2026-05-14 08:40

### Arquivos alterados

- `portal-sama-api/src/modules/notifications/dto/list-notifications.dto.ts`
- `portal-sama-api/src/modules/notifications/notifications.service.ts`
- `portal-sama-api/src/modules/notifications/notifications.service.spec.ts`
- documentacao de status e testes.

### O que mudou

- `GET /api-v2/notifications/stream` agora aceita `last_id`/`lastId`.
- O stream busca periodicamente novas notificacoes no MySQL com `id > cursor`, em ordem crescente e com limite por lote.
- O snapshot nao avanca o cursor quando o cliente envia `last_id`, evitando perda de notificacoes intermediarias em reconexao.
- Eventos vindos do snapshot/live sao deduplicados contra o polling do banco.

### Testes executados

- `npm.cmd test -- notifications.service.spec.ts`: passou, 10 testes.
- `npm.cmd exec tsc -- --noEmit`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd run test:e2e -- --runInBand`: passou, 125 testes.
- `npm.cmd test`: passou, 22 suites/126 testes.
- `npm.cmd run build`: passou.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.

### Riscos ou pendencias

- O polling cobre novas notificacoes criadas por outras instancias, mas eventos instantaneos de update/ack em multi-instancia ainda podem exigir broker/event bus se forem necessarios em tempo real.

## 2026-05-14 08:30

### Arquivos alterados

- `portal-sama-api/src/modules/notifications/**`
- `portal-sama-api/src/modules/access-requests/access-requests.module.ts`
- `portal-sama-api/src/modules/access-requests/access-requests.service.ts`
- `portal-sama-api/src/modules/access-requests/access-requests.service.spec.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- documentacao de status, pendencias, inventarios, matrizes e relatorio de testes.

### O que mudou

- Adicionado `GET /api-v2/notifications/stream` com SSE autenticado, snapshot inicial, ping periodico e eventos de criacao, atualizacao e limpeza de notificacoes.
- O stream reaproveita o escopo por usuario/departamento, filtros de listagem e filtro de autoevento do `NotificationsService`.
- `AccessRequestsModule` passou a emitir notificacoes automaticas ao gestor quando colaborador cria solicitacao, ao solicitante quando gestor aprova/rejeita e ao TI quando uma solicitacao aprovada precisa de execucao tecnica.
- A emissao automatica usa `NotificationsService.create`, dedupe, auditoria e isolamento de falha para nao bloquear o fluxo principal de solicitacao.

### Testes executados

- `npm.cmd test -- access-requests.service.spec.ts notifications.service.spec.ts`: passou, 14 testes.
- `npm.cmd exec tsc -- --noEmit`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd run test:e2e -- --runInBand`: passou, 125 testes.
- `npm.cmd test`: passou, 22 suites/125 testes.
- `npm.cmd run build`: passou.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou, apenas avisos LF/CRLF do Windows.

### Riscos ou pendencias

- O stream atual usa eventos em memoria por instancia; para producao horizontal, ainda falta broker/event bus distribuido.
- Faltam emissores automaticos em outros modulos de negocio, integracao frontend e validacao em homologacao/producao.

## 2026-05-13 16:34

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260513170000_add_notifications_model/migration.sql`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/notifications/**`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.spec.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- documentacao de status, inventarios, matrizes e relatorio de testes.

### O que mudou

- Criado `NotificationsModule` com listagem, criacao, ack, alert-ack e limpeza de nao lidas.
- Modelada a tabela legada `notifications` no Prisma com `Notification`, preservando id numerico e colunas `created_at`, `read_at`, `alerted_at`, `meta`, `event_type`, `dedupe_key` e `expires_at`.
- Adicionadas permissoes `notifications.read` e `notifications.create` ao catalogo RBAC.
- Listagem e mutacoes aplicam escopo por usuario/departamento; criacao respeita departamento permitido e evita notificacao para o proprio ator.
- Mutacoes exigem CSRF e gravam auditoria.

### Testes executados

- `npm.cmd run prisma:format`: passou.
- `npm.cmd run prisma:generate`: passou.
- `npm.cmd run prisma:validate`: passou.
- `npm.cmd exec tsc -- --noEmit`: passou.
- `npm.cmd test -- notifications.service.spec.ts`: passou, 7 testes.
- `npm.cmd test -- rbac/default-rbac.spec.ts`: passou, 2 testes.
- `npm.cmd run lint`: passou.
- `npm.cmd run test:e2e -- --runInBand`: passou, 123 testes.
- `npm.cmd run prisma:migrate:deploy`: passou e aplicou `20260513170000_add_notifications_model`.
- `npm.cmd run prisma:migrate:status`: passou, banco atualizado.
- `npm.cmd exec prisma -- migrate diff --from-schema-datasource prisma/schema.prisma --to-schema-datamodel prisma/schema.prisma --exit-code`: passou, sem diferencas.
- `npm.cmd run prisma:seed`: passou.
- `npm.cmd test`: passou, 22 suites/123 testes.
- `npm.cmd run build`: passou.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou, apenas avisos LF/CRLF do Windows.

### Riscos ou pendencias

- SSE/stream de notificacoes, emissores automaticos nos modulos de negocio, backfill/retencao em homologacao/producao e frontend ainda seguem pendentes.

## 2026-05-13 16:16

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260513164000_add_legalization_templates/migration.sql`
- `portal-sama-api/src/modules/legalization/**`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.spec.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- documentacao de status, inventarios, matrizes e relatorio de testes.

### O que mudou

- Adicionado CRUD de templates de legalizacao em `/api-v2/legalization/templates`.
- Criado modelo Prisma `LegalizationTemplate`, enum `LegalizationTemplateType` e tabela `legalization_templates`.
- Criada permissao `legalization.templates`, atribuida ao perfil `LEGALIZATION` e herdada por `ADMIN`/`DEV`.
- Leitura de templates exige `legalization.read`; criacao, atualizacao e exclusao logica exigem `legalization.templates`, CSRF e auditoria.
- HTML de templates e rejeitado quando contem tags/atributos perigosos como `script`, handlers inline, `iframe`, `object`, `embed`, `link`, `meta` ou URL `javascript:`.

### Testes executados

- `npm.cmd run prisma:format`: passou.
- `npm.cmd run prisma:generate`: passou.
- `npm.cmd run prisma:validate`: passou.
- `npm.cmd exec tsc -- --noEmit`: passou.
- `npm.cmd test -- legalization.service.spec.ts`: passou, 10 testes.
- `npm.cmd test -- rbac/default-rbac.spec.ts`: passou, 2 testes.
- `npm.cmd run test:e2e -- --runInBand`: passou, 117 testes.
- `npm.cmd run lint`: passou.
- `npm.cmd run prisma:migrate:deploy`: passou e aplicou `20260513164000_add_legalization_templates`.
- `npm.cmd run prisma:migrate:status`: passou, banco atualizado.
- `npm.cmd exec prisma -- migrate diff --from-schema-datasource prisma/schema.prisma --to-schema-datamodel prisma/schema.prisma --exit-code`: passou, sem diferencas.
- `npm.cmd run prisma:seed`: passou.
- `npm.cmd test`: passou, 21 suites/116 testes.
- `npm.cmd run build`: passou.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou, apenas avisos LF/CRLF do Windows.

### Riscos ou pendencias

- Backfill de templates legados, render/import PDF seguro, integracao com frontend e validacao em homologacao/producao seguem pendentes.

## 2026-05-13 15:59

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260513160000_add_legalization_processes/migration.sql`
- `portal-sama-api/prisma/migrations/20260513160500_add_missing_relation_indexes/migration.sql`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/legalization/**`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- documentacao de status, inventarios, matrizes e relatorio de testes.

### O que mudou

- Criado `LegalizationModule` com processos internos em `/api-v2/legalization/processes`.
- Criado modelo Prisma `LegalizationProcess`, enum `LegalizationStatus` e migration `20260513160000_add_legalization_processes`.
- Adicionada migration complementar `20260513160500_add_missing_relation_indexes` para alinhar indices relacionais esperados pelo Prisma apos o baseline local.
- Adicionadas permissoes `legalization.read/create/update/status/delete` ao catalogo RBAC; seed executado no MySQL local.
- Mutacoes exigem CSRF, permissao granular, escopo por responsavel/criador/departamento e auditoria.
- Respostas preservam aliases do legado (`company_name`, `cnpj`, `tipo_processo`, `protocolo`, `etapa`, `status_legacy`) para facilitar integracao futura.

### Testes executados

- `npm.cmd run prisma:format`: passou.
- `npm.cmd run prisma:generate`: passou.
- `npm.cmd run prisma:validate`: passou.
- `npm.cmd exec tsc -- --noEmit`: passou.
- `npm.cmd test -- legalization.service.spec.ts`: passou, 6 testes.
- `npm.cmd test -- rbac/default-rbac.spec.ts`: passou, 2 testes.
- `npm.cmd run test:e2e -- --runInBand`: passou, 112 testes.
- `npm.cmd run lint`: passou.
- `npm.cmd run prisma:migrate:deploy`: passou e aplicou as 2 migrations novas.
- `npm.cmd run prisma:migrate:status`: passou, banco atualizado.
- `npm.cmd exec prisma -- migrate diff --from-schema-datasource prisma/schema.prisma --to-schema-datamodel prisma/schema.prisma --exit-code`: passou, sem diferencas.
- `npm.cmd run prisma:seed`: passou.
- `npm.cmd test`: passou, 21 suites/112 testes.
- `npm.cmd run build`: passou.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou, apenas avisos LF/CRLF do Windows.

### Riscos ou pendencias

- Templates de cabecalho/rodape, importacao/renderizacao PDF segura, restore/hard_delete/purge, backfill legado, notificacoes e frontend React seguem pendentes.

## 2026-05-13 15:18

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260513152000_add_onboarding_processes/migration.sql`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/onboarding/**`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.spec.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- `docs/EASYPANEL_DEPLOY.md`
- `docs/INCIDENTE_SUBDOMINIO_PORTAL_SAMACONTABIL.md`
- documentacao de status, inventarios, matrizes e relatorio de testes.

### O que mudou

- Criado `OnboardingModule` com rotas internas para processos, detalhe, criacao, atualizacao, status/etapa, timeline, documentos e arquivamento logico.
- Criado modelo Prisma `OnboardingProcess`, enum `OnboardingStatus` e migration `20260513152000_add_onboarding_processes`.
- Adicionadas permissoes `onboarding.read/create/update/status/documents/delete` ao catalogo RBAC; seed executado no MySQL local.
- Mutacoes exigem CSRF, permissao granular, escopo por responsavel/criador/departamento e auditoria.
- Respostas preservam aliases do legado (`company_name`, `tipo_servico`, `regime_lucro`, `stage`, `created_by`) para facilitar integracao futura das telas.
- Registrado diagnostico do subdominio `portal.samacontabil.com.br`: DNS resolve para EasyPanel, HTTPS retornou 500 e o destino interno customizado parece usar `https://...:80`, que deve ser `http://...:80` quando o servico interno e HTTP.

### Testes executados

- `npm.cmd run format`: passou.
- `npm.cmd run prisma:generate`: passou.
- `npm.cmd run prisma:validate`: passou.
- `npx.cmd tsc --noEmit`: passou.
- `npm.cmd test -- onboarding.service.spec.ts`: passou, 7 testes.
- `npm.cmd test -- default-rbac.spec.ts`: passou, 2 testes.
- `npm.cmd run lint`: passou.
- `npm.cmd run prisma:migrate:deploy`: passou e aplicou `20260513152000_add_onboarding_processes`.
- `npm.cmd run prisma:migrate:status`: passou, banco atualizado.
- `npx.cmd prisma migrate diff --from-schema-datasource prisma\\schema.prisma --to-schema-datamodel prisma\\schema.prisma --exit-code`: passou, sem diferencas.
- `npm.cmd run prisma:seed`: passou.
- `npm.cmd test`: passou, 20 suites/106 testes.
- `npm.cmd run test:e2e`: passou, 1 suite/102 testes.
- `npm.cmd run build`: passou.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou, apenas avisos LF/CRLF do Windows.

### Riscos ou pendencias

- Fluxos publicos especificos `/api-v2/public/onboarding/*` ainda nao foram criados; documentos/propostas publicas seguem cobertos por `DocumentsModule` e `ProposalsModule`.
- Regras completas de maquina de estados, notificacoes, backfill de `onboarding_process` legado e frontend React seguem pendentes.
- O subdominio antigo ainda retorna HTTP 500; corrigir alvo interno no EasyPanel ou confirmar logs antes de trocar trafego da nova versao.

## 2026-05-13 14:43

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260513124500_add_collaborator_profile_to_users/migration.sql`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/collaborators/**`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.spec.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- `docs/CHANGELOG_TECNICO.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/RELATORIO_TESTES.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/BANCO_DADOS_MYSQL_PRISMA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `.ai-tests/runs/2026-05-13-14-43-collaborators/*`

### O que mudou

- Criado `CollaboratorsModule` com `GET/POST /api-v2/collaborators`, `GET/PATCH/DELETE /api-v2/collaborators/:id`, DTOs, service, testes unitarios e e2e de protecao.
- `User` foi expandido com `position`, `phone`, `extension`, `metadata`, `archivedAt` e `archivedById` para cobrir a primeira fatia funcional de colaboradores sem criar uma tabela paralela antes do backfill.
- Adicionadas permissoes `collaborators.read`, `collaborators.create`, `collaborators.update` e `collaborators.delete` ao catalogo RBAC; `ADMIN`/`DEV` herdam todas e perfis internos recebem leitura.
- Mutacoes de colaboradores exigem CSRF, permissao granular, validam roles, bloqueiam role `CLIENT` no modulo de colaboradores e registram auditoria sem senha/hash.

### Motivo da alteracao

Dar continuidade a etapa documentada de clientes e colaboradores apos `ClientsModule`, substituindo parte do uso generico de `api/storage.php?action=get/set/delete` por contratos REST seguros em `/api-v2`.

### Impacto esperado

- Backend: primeira fatia de colaboradores disponivel e registrada no `AppModule`.
- Banco: migration incremental aplicada no MySQL local sobre `users`.
- Seguranca: leitura exige `collaborators.read`; mutacoes exigem `collaborators.create/update/delete`, CSRF e auditoria.
- Frontend: telas HTML/React ainda nao foram conectadas; respostas ja preservam aliases legados (`usuario`, `nome`, `departamento`, `cargo`, `telefone`, `ramal`).

### Testes executados

- Comando: `npm.cmd run prisma:format`; Resultado: passou.
- Comando: `npm.cmd run prisma:generate`; Resultado: passou.
- Comando: `npm.cmd run prisma:validate`; Resultado: passou.
- Comando: `npx.cmd tsc --noEmit --project tsconfig.json`; Resultado: passou.
- Comando: `npm.cmd test -- collaborators.service.spec.ts`; Resultado: passou, 6 testes.
- Comando: `npm.cmd run lint`; Resultado: passou.
- Comando: `npm.cmd test -- default-rbac.spec.ts`; Resultado: passou, 2 testes.
- Comando: `npm.cmd run prisma:migrate:deploy`; Resultado: passou e aplicou `20260513124500_add_collaborator_profile_to_users`.
- Comando: `npm.cmd run prisma:migrate:status`; Resultado: passou, banco atualizado.
- Comando: `npx.cmd prisma migrate diff --from-schema-datasource prisma\\schema.prisma --to-schema-datamodel prisma\\schema.prisma --exit-code`; Resultado: passou, sem diferencas.
- Comando: `npm.cmd run prisma:seed`; Resultado: passou.
- Comando: `npm.cmd test`; Resultado: passou, 19 suites/99 testes.
- Comando: `npm.cmd run test:e2e`; Resultado: passou, 1 suite/91 testes.
- Comando: `npm.cmd run build`; Resultado: passou.
- Comando: `npm.cmd audit --audit-level=moderate`; Resultado: passou, 0 vulnerabilidades.
- Comando: `git diff --check`; Resultado: passou, apenas avisos LF/CRLF do Windows.

### Riscos ou pendencias

- Backfill de `sama_colaboradores` e usuarios legados ainda nao foi feito.
- Vinculos colaborador-cliente/gestor, carteira e transferencias seguem pendentes.
- Frontend legado/React ainda nao consome `/api-v2/collaborators`.
- A primeira tentativa de migration falhou por Docker/MySQL local parado; o ambiente foi religado e a validacao final passou.

## 2026-05-13 10:50

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260513113000_expand_clients_profile/migration.sql`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/clients/**`
- `portal-sama-api/test/app.e2e-spec.ts`
- `docs/CHANGELOG_TECNICO.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/RELATORIO_TESTES.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/BANCO_DADOS_MYSQL_PRISMA.md`
- `.ai-tests/runs/2026-05-13-10-50-clients-baseline/*`

### O que mudou

**Baseline Prisma/MySQL local resolvido:**
- O banco local `portal_sama` foi alinhado ao schema atual sem expor valores do `.env`.
- As 5 migrations que apareciam pendentes foram marcadas como aplicadas apos sincronizacao controlada do schema.
- A migration nova `20260513113000_expand_clients_profile` foi aplicada por `prisma:migrate:deploy`.
- `prisma:migrate:status` voltou a retornar `Database schema is up to date` e `prisma migrate diff` nao encontrou diferencas.

**Clientes - backend seguro inicial:**
- Criado `ClientsModule` com `GET /api-v2/clients`, `GET /api-v2/clients/:id`, `GET /api-v2/clients/:id/dashboard`, `POST /api-v2/clients`, `PATCH /api-v2/clients/:id` e `DELETE /api-v2/clients/:id`.
- `Client` ganhou campos usados pelas telas legadas DEV/Clientes: `rank`, `regimeTributario`, `idEmpresa`, `atividade`, telefone/endereco, grupo, `metadata`, `deletedAt` e `deletedById`.
- Mutacoes exigem permissoes `clients.create`, `clients.update` ou `clients.delete`, CSRF e auditoria centralizada.
- Respostas mantem aliases legados (`razao_social`, `nome_fantasia`, `regime_tributario`, `id_empresa`, `faz_parte_grupo`, `grupo_nome`) para facilitar a futura integracao das telas.

### Motivo da alteracao

Resolver o ponto pendente de migrations antes de continuar a migracao e abrir a primeira fatia do backend de clientes, substituindo parte do uso generico de `api/storage.php?action=get/set/delete` por contratos REST em `/api-v2`.

### Impacto esperado

- Backend: `ClientsModule` inicial disponivel e registrado no `AppModule`.
- Banco: modelo `Client` expandido e migration incremental aplicada no MySQL local.
- Seguranca: leitura protegida por `clients.read`, mutacoes com permissao granular, CSRF e auditoria.
- Frontend: telas HTML/React ainda nao foram conectadas; contratos de resposta ja preservam aliases do legado.

### Testes executados

- Comando: `npm.cmd run prisma:format`; Resultado: passou.
- Comando: `npm.cmd run prisma:generate`; Resultado: passou.
- Comando: `npm.cmd run prisma:validate`; Resultado: passou.
- Comando: `npm.cmd run prisma:migrate:deploy`; Resultado: passou e aplicou `20260513113000_expand_clients_profile`.
- Comando: `npm.cmd run prisma:migrate:status`; Resultado: passou, banco atualizado.
- Comando: `npx.cmd prisma migrate diff --from-schema-datasource prisma\\schema.prisma --to-schema-datamodel prisma\\schema.prisma --exit-code`; Resultado: passou, sem diferencas.
- Comando: `npm.cmd run prisma:seed`; Resultado: passou.
- Comando: `npx.cmd tsc --noEmit --project tsconfig.json`; Resultado: passou.
- Comando: `npm.cmd test -- clients.service.spec.ts`; Resultado: passou, 5 testes.
- Comando: `npm.cmd run lint`; Resultado: passou.
- Comando: `npm.cmd test`; Resultado: passou, 18 suites/93 testes.
- Comando: `npm.cmd run test:e2e`; Resultado: passou, 1 suite/84 testes.
- Comando: `npm.cmd run build`; Resultado: passou.
- Comando: `npm.cmd audit --audit-level=moderate`; Resultado: passou, 0 vulnerabilidades.
- Comando: `git diff --check`; Resultado: passou, apenas avisos LF/CRLF do Windows.

### Riscos ou pendencias

- Escopo real cliente-usuario/gestor ainda depende de modelagem/backfill de vinculos reais.
- Backfill de dados legados de clientes e colaboradores ainda nao foi feito.
- Frontend legado/React ainda nao consome `/api-v2/clients`.
- `git status` pode mostrar arquivos antigos como modificados por fim de linha apos o formatter no Windows; `git diff --raw` confirmou conteudo alterado apenas nos arquivos do escopo.

## 2026-05-13 10:15

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260513101500_add_access_requests/migration.sql`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/access-requests/**`
- `portal-sama-api/test/app.e2e-spec.ts`
- `docs/CHANGELOG_TECNICO.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/RELATORIO_TESTES.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/BANCO_DADOS_MYSQL_PRISMA.md`
- `docs/SEGURANCA.md`
- `docs/paginas/solicitacao-acesso.md`
- `docs/paginas/ti-acesso-ti.md`
- `.ai-tests/runs/2026-05-13-10-15-access-requests/*`

### O que mudou

**Solicitacoes de acesso - backend seguro inicial:**
- Criado `AccessRequestsModule` com `GET /api-v2/access-requests`, `GET /api-v2/access-requests/:id`, `GET /api-v2/access-requests/my/latest`, `GET /api-v2/access-requests/manager/approvals`, `POST /api-v2/access-requests`, `POST /api-v2/access-requests/:id/approve` e `POST /api-v2/access-requests/:id/reject`.
- Criado modelo Prisma `AccessRequest`, enums `AccessRequestProfile`/`AccessRequestStatus` e migration `20260513101500_add_access_requests`.
- Criacao de solicitacao de colaborador valida gestor ativo no mesmo departamento, grava status `PENDING_MANAGER` e exige decisao do gestor responsavel ou papel privilegiado (`ADMIN`, `DEV`, `TI`).
- Criacao de solicitacao de gestor valida colaboradores ativos do mesmo departamento e encaminha direto para status `APPROVED`, preservando aliases legados de resposta para reduzir atrito na futura integracao da tela.
- Decisao aprovar/rejeitar exige permissao granular (`access_requests.approve` ou `access_requests.reject`), CSRF e auditoria sem gravar justificativas ou dados sensiveis em excesso nos metadados.
- Regras de data/horario foram centralizadas no service: datas aceitas em `DD/MM/YYYY` ou `YYYY-MM-DD`, dias uteis exigem janela entre 17:00 e 23:59 e horario final maior que inicial.

### Motivo da alteracao

Dar continuidade a migracao da tela `SolicitacaoAcesso/solicitacao-acesso.html` e das aprovacoes de gestor/TI para a API v2, reduzindo a dependencia de `api/access_requests.php` e de `SolicitacaoAcesso/enviar_acesso.php`.

### Impacto esperado

- Backend: primeira base de `AccessRequestsModule` disponivel em NestJS.
- Banco: nova tabela `access_requests` e enums de perfil/status via migration Prisma.
- Seguranca: escopo por solicitante/gestor/departamento, CSRF em mutacoes, permissoes granulares e auditoria centralizada.
- Frontend: telas HTML legadas ainda nao foram conectadas; a migracao React continua pendente.

### Testes executados

- Comando: `npm.cmd run prisma:format`; Resultado: passou.
- Comando: `npm.cmd run prisma:generate`; Resultado: passou.
- Comando: `npm.cmd run prisma:validate`; Resultado: passou.
- Comando: `npx.cmd tsc --noEmit --project tsconfig.json`; Resultado: passou apos ajustes de tipagem.
- Comando: `npm.cmd test -- access-requests.service.spec.ts`; Resultado: passou, 5 testes.
- Comando: `npm.cmd run lint`; Resultado: passou apos ajustes de stringificacao segura.
- Comando: `npm.cmd test`; Resultado: passou, 17 suites/88 testes; Jest emitiu aviso de worker sem encerrar graciosamente.
- Comando: `npm.cmd run test:e2e`; Resultado: passou, 1 suite/75 testes.
- Comando: `npm.cmd run build`; Resultado: passou.
- Comando: `npm.cmd audit --audit-level=moderate`; Resultado: passou, 0 vulnerabilidades.
- Comando: `npm.cmd run prisma:migrate:status`; Resultado: falhou porque as 5 migrations estao pendentes no banco local `portal_sama`.
- Comando: `git diff --check`; Resultado: passou, apenas avisos LF/CRLF do Windows.

### Riscos ou pendencias

- `prisma:migrate:status` informou 5 migrations pendentes no MySQL local (`20260512172700_init_current_schema`, `20260512195000_add_digital_certificates`, `20260513085700_add_proposals`, `20260513092300_add_contracts`, `20260513101500_add_access_requests`); o banco possui tabelas fora do historico Prisma e precisa baseline/ajuste controlado antes de `migrate deploy`.
- A integracao com `SolicitacaoAcesso/acesso.js`, `TI/ti.js` e notificacoes ainda nao foi feita.
- Backfill/compatibilidade com `sama_access_requests` legado e historico estruturado de decisoes ainda precisam ser validados em homologacao.

## 2026-05-13 09:34

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260513092300_add_contracts/migration.sql`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/contracts/**`
- `portal-sama-api/test/app.e2e-spec.ts`
- `docs/CHANGELOG_TECNICO.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/RELATORIO_TESTES.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/BANCO_DADOS_MYSQL_PRISMA.md`
- `docs/SEGURANCA.md`
- `.ai-tests/runs/2026-05-13-09-34-contracts/*`

### O que mudou

**Contratos e assinatura - backend seguro inicial:**
- Criado `ContractsModule` com `GET /api-v2/contracts`, `GET /api-v2/contracts/:id`, `POST /api-v2/contracts`, `PATCH /api-v2/contracts/:id`, `POST /api-v2/contracts/:id/generate` e `POST /api-v2/contracts/:id/send-signature`.
- Criadas rotas públicas `GET /api-v2/public/signatures/:token` e `POST /api-v2/public/signatures/:token/sign`.
- Criado modelo Prisma `Contract`, enum `ContractStatus` e migration `20260513092300_add_contracts`.
- Envio para assinatura gera token público opaco `sama_pub_*`, persiste apenas SHA-256 em `PublicToken`, revoga tokens ativos anteriores do contrato e audita sem token bruto.
- Assinatura pública valida token, expiração, revogação, módulo/escopo e status; grava evidência com hash do documento/assinatura, IP e user-agent, revoga o token após uso e audita.
- HTML de contrato é rejeitado quando contém padrões de alto risco (`script`, handlers `on*`, `javascript:`, `iframe`, `object`, `embed`, `link`, `meta`), reduzindo risco inicial de XSS até existir sanitização/sandbox completa.

### Motivo da alteração

Dar continuidade à etapa de legalização definida após propostas, criando a base tipada e auditável para contratos e assinatura pública antes de integrar renderização PDF/importação de documentos.

### Impacto esperado

- Backend: primeira base de `ContractsModule` disponível em NestJS.
- Banco: nova tabela `contracts` e enum `ContractStatus` via migration Prisma.
- Segurança: token público escopado, trilha de auditoria, CSRF em mutações internas e evidência de assinatura sem gravar token bruto.
- Frontend: ainda não conectado; telas `Legalizacao/contrato.html` e `Legalizacao/assinatura.html` precisam adaptação futura.

### Testes executados

- Comando: `npm.cmd run prisma:format`; Resultado: passou.
- Comando: `npm.cmd run prisma:generate`; Resultado: passou.
- Comando: `npm.cmd run prisma:validate`; Resultado: passou.
- Comando: `npx.cmd tsc --noEmit --project tsconfig.json`; Resultado: passou.
- Comando: `npm.cmd test -- contracts.service.spec.ts`; Resultado: passou, 5 testes.
- Comando: `npm.cmd run lint`; Resultado: passou.
- Comando: `npm.cmd test`; Resultado: passou, 16 suites/83 testes.
- Comando: `npm.cmd run test:e2e`; Resultado: passou, 1 suite/66 testes.
- Comando: `npm.cmd run build`; Resultado: passou.
- Comando: `npm.cmd audit --audit-level=moderate`; Resultado: passou, 0 vulnerabilidades.
- Comando: `git diff --check`; Resultado: passou, apenas avisos LF/CRLF do Windows.

### Riscos ou pendências

- `prisma:migrate:status` encontrou Docker/MySQL local disponível, mas informou 4 migrations ainda não aplicadas no banco `portal_sama`.
- `prisma db pull --print` mostrou que o banco local tem tabelas iniciais criadas fora do histórico de migrations e ainda não possui todos os modelos atuais (`public_tokens`, `document_status_history`, `digital_certificates`, `proposals`, `contracts`). Aplicar migrations exige baseline/ajuste controlado para não colidir com tabelas existentes.
- Importação de PDF/Word, renderização PDF real, sandbox do renderizador e backfill de contratos legados seguem pendentes.

## 2026-05-13 09:20

### Arquivos alterados

- `portal-sama-api/tsconfig.json`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `.ai-tests/runs/2026-05-13-09-20-typescript-baseurl/*`

### O que mudou

**TypeScript 6 - `baseUrl` depreciado:**
- Removido `compilerOptions.baseUrl` do `tsconfig.json`.
- A busca por imports não encontrou aliases absolutos que dependessem de `baseUrl`; o projeto usa imports relativos.

### Motivo da alteração

Corrigir alerta do editor: `Option 'baseUrl' is deprecated and will stop functioning in TypeScript 7.0`.

### Testes executados

- Comando: `npx.cmd tsc --noEmit --project tsconfig.json`; Resultado: passou.
- Comando: `npm.cmd run lint`; Resultado: passou.
- Comando: `npm.cmd run build`; Resultado: passou.

## 2026-05-13 09:15

### Arquivos alterados

- `portal-sama-api/tsconfig.json`
- `.vscode/settings.json`
- `docs/CHANGELOG_TECNICO.md`
- `docs/RELATORIO_TESTES.md`
- `.ai-tests/runs/2026-05-13-09-15-typescript-editor/*`

### O que mudou

**Configuração TypeScript/Jest local:**
- `tsconfig.json` da API passou a declarar explicitamente `types: ["node", "jest"]`.
- `tsconfig.json` passou a incluir `src/**/*.ts`, `test/**/*.ts` e `prisma/**/*.ts`, evitando que `test/app.e2e-spec.ts` fique em projeto inferido no editor.
- `.vscode/settings.json` passou a apontar `typescript.tsdk` para `portal-sama-api/node_modules/typescript/lib`, fazendo o VS Code usar o TypeScript instalado na API.
- Dependências locais da API foram conferidas com `npm.cmd install`.

### Motivo da alteração

Corrigir falsos erros do editor em arquivos de teste TypeScript, como `Cannot find name 'describe'`, `beforeAll`, `afterAll`, `it` e `expect`, mesmo com `@types/jest` instalado.

### Testes executados

- Comando: `npm.cmd install`; Resultado: dependências já atualizadas, 0 vulnerabilidades.
- Comando: `npx.cmd tsc --noEmit --project tsconfig.json`; Resultado: passou.
- Comando: `npm.cmd run prisma:validate`; Resultado: passou.
- Comando: `npm.cmd test -- proposals.service.spec.ts`; Resultado: passou, 5 testes.
- Comando: `npm.cmd run lint`; Resultado: passou.
- Comando: `npm.cmd run build`; Resultado: passou.
- Comando: `node --version`; Resultado: `v24.15.0`.
- Comando: `npm.cmd --version`; Resultado: `11.12.1`.

### Observações

- No PowerShell, `npm --version` via `npm.ps1` foi bloqueado pela política de execução local. `npm.cmd` funciona normalmente e deve ser usado nos comandos do projeto.

## 2026-05-13 08:57

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260513085700_add_proposals/migration.sql`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/proposals/**`
- `portal-sama-api/test/app.e2e-spec.ts`
- `docs/BANCO_DADOS_MYSQL_PRISMA.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/SEGURANCA.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `.ai-tests/runs/2026-05-13-08-57-proposals/*`

### O que mudou

**Propostas - backend seguro inicial:**
- Criado `ProposalsModule` com `GET /api-v2/proposals`, `GET /api-v2/proposals/:id`, `POST /api-v2/proposals`, `PATCH /api-v2/proposals/:id`, `POST /api-v2/proposals/:id/send`, `POST /api-v2/proposals/:id/approve` e `POST /api-v2/proposals/:id/reject`.
- Criadas rotas públicas `GET /api-v2/public/proposals/:token` e `POST /api-v2/public/proposals/:token` para visualização e resposta do cliente por token opaco.
- Criado modelo Prisma `Proposal` e enum `ProposalStatus`; campos legados como `company_name`, `tipo_proposta`, `proposal_status` e `proposal_fields` são preservados nas respostas.
- Envio de proposta gera token `sama_pub_*`, persiste apenas SHA-256 em `PublicToken`, revoga tokens ativos anteriores da proposta, exige CSRF e registra auditoria sem token bruto.
- Aprovação/ajuste público valida token, expiração, revogação, módulo/escopo, exige mensagem para ajuste, revoga o token após uso e registra auditoria.

**Documentação de acompanhamento:**
- Inventários de endpoints, banco e segurança foram sincronizados com `CertificatesModule` e `ProposalsModule`.
- Matrizes PHP->NestJS e HTML->React passaram a marcar certificados e propostas como backend parcial.
- Pendências, status e relatório de testes registram a validação local e o bloqueio de MySQL/Docker para migration real de propostas.

### Motivo da alteração

Dar continuidade à migração definida após documentos e certificados, substituindo gradualmente `api/propostas.php` por contratos tipados em `/api-v2` com RBAC, CSRF, token público seguro e auditoria.

### Impacto esperado

- Backend: primeira base de `ProposalsModule` disponível em NestJS.
- Banco: nova tabela `proposals` e enum `ProposalStatus` via migration Prisma.
- Segurança: menor exposição de links públicos, trilha auditável e bloqueio de mutações sem CSRF.
- Frontend: ainda não conectado; tela legada/React precisa adaptação futura.

### Testes executados

- Comando: `npm.cmd run prisma:format`; Resultado: passou.
- Comando: `npm.cmd run prisma:generate`; Resultado: passou.
- Comando: `npm.cmd test -- proposals.service.spec.ts`; Resultado: passou, 5 testes.
- Comando: `npm.cmd run lint`; Resultado: passou.
- Comando: `npm.cmd run prisma:validate`; Resultado: passou.
- Comando: `npm.cmd test`; Resultado: passou, 15 suites/78 testes.
- Comando: `npm.cmd run test:e2e`; Resultado: passou, 1 suite/59 testes.
- Comando: `npm.cmd run build`; Resultado: passou.
- Comando: `npm.cmd audit --audit-level=moderate`; Resultado: passou, 0 vulnerabilidades.
- Comando: `git diff --check`; Resultado: passou, apenas avisos LF/CRLF do Windows.
- Rechecagem final em 2026-05-13 09:07: `git diff --check`, `npm.cmd run prisma:validate` e `npm.cmd test -- proposals.service.spec.ts` passaram após atualização documental.

### Riscos ou pendências

- `docker ps` falhou porque o Docker Desktop/MySQL local não está disponível.
- `npm.cmd run prisma:migrate:status` ficou bloqueado com `Schema engine error` contra MySQL em `localhost:3306`; `migrate deploy` da migration de propostas ainda precisa ser validado em MySQL de homologação.
- Backfill/compatibilidade dos dados legados de `api/propostas.php` e tokens antigos seguem pendentes.
- Integração frontend da tela de proposta e da página pública ainda não foi feita.

---

## 2026-05-12 19:50

### Arquivos alterados

- `portal-sama-api/src/modules/certificates/*` (novo `CertificatesModule`)
- `portal-sama-api/prisma/schema.prisma` e migration `20260512195000_add_digital_certificates`
- `.env.example` e `portal-sama-api/.env.example` (variáveis de certificado)
- `portal-sama-api/test/app.e2e-spec.ts` e testes unitários de certificados
- `docs/PENDENCIAS_TECNICAS.md`

### O que mudou

**Certificados digitais - backend seguro inicial:**
- Criadas rotas `/api-v2/certificates`, `/api-v2/certificates/:id`, `/api-v2/certificates/:id/download` e `/api-v2/certificates/:id/rotate-password`.
- Upload restrito a `.p12/.pfx`, com limite dedicado, validação de MIME/assinatura PKCS#12 e storage privado fora do MySQL.
- Senhas são gravadas apenas como `passwordCipher` com AES-256-GCM; respostas JSON retornam somente `hasPassword`, sem senha nem ciphertext.
- CRUD e download usam RBAC granular (`certificates.read`, `certificates.download`, `certificates.manage`), escopo por role/departamento/cliente, CSRF em mutações e auditoria de criação, atualização, remoção, download e rotação de senha.
- Criada tabela `digital_certificates` para metadados, hash do arquivo, soft delete e trilha de autoria.

### Validação

- `prisma validate`: passou.
- `prisma generate`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd test`: 14 suítes/73 testes.
- `npm.cmd run test:e2e`: 1 suíte/52 testes.
- `npm.cmd run build`: passou.
- `npm.cmd audit --audit-level=moderate`: 0 vulnerabilidades.
- `prisma migrate deploy`: aplicou as duas migrations em `portal_sama_migration_check_cert`.
- `prisma migrate diff --from-migrations --to-schema-datamodel`: sem diferenças.
- `prisma:seed`: passou no banco descartável; conferência SQL retornou 9 roles, 37 permissões, 124 vínculos e tabela `digital_certificates` criada.

### Riscos ou pendências

- Validar a migration incremental, storage real e backfill dos certificados legados em MySQL de homologação.
- Integrar a tela `/certificados-digitais` no frontend React após validar o contrato de API.

---

## 2026-05-12 17:27

### Arquivos alterados

- `portal-sama-api/prisma/migrations/migration_lock.toml` (provider MySQL para Prisma Migrate)
- `portal-sama-api/prisma/migrations/20260512172700_init_current_schema/migration.sql` (migration inicial do schema atual)
- `portal-sama-api/package.json` (script `prisma:migrate:deploy`)
- Documentação em `docs/` atualizada com validação de migration/seed

### O que mudou

**Banco - migration Prisma formal inicial:**
- Criada migration inicial para as tabelas `users`, `roles`, `permissions`, `user_roles`, `role_permissions`, `refresh_tokens`, `clients`, `public_tokens`, `documents`, `document_requirements`, `document_status_history` e `audit_logs`.
- A migration inclui índices, chaves únicas, enums, JSON metadata e foreign keys coerentes com `schema.prisma`.
- Adicionado script `npm.cmd run prisma:migrate:deploy` para homologação/produção.

**Validação:**
- `prisma validate`: passou.
- `prisma migrate deploy`: aplicado com sucesso no banco descartável `portal_sama_migration_check`.
- `prisma migrate diff --from-migrations --to-schema-datamodel`: sem diferenças entre migrations e schema.
- `prisma:seed`: passou no banco descartável.
- Conferência SQL pós-seed: 9 roles, 37 permissões e 124 vínculos em `role_permissions`.

### Riscos ou pendências

- Bancos já sincronizados por `prisma db push` precisam de baseline controlado antes de usar `migrate deploy`.
- Backfills de dados legados, vínculo cliente/usuário e histórico herdado seguem pendentes para homologação.

---

## 2026-05-12 16:13

### Arquivos alterados

- `portal-sama-api/src/modules/roles/dto/create-role.dto.ts` (DTO de criação de role)
- `portal-sama-api/src/modules/roles/dto/update-role.dto.ts` (DTO de atualização de role)
- `portal-sama-api/src/modules/roles/dto/update-role-permissions.dto.ts` (DTO de troca de permissões da role)
- `portal-sama-api/src/modules/roles/roles.controller.ts` (rotas mutáveis de roles)
- `portal-sama-api/src/modules/roles/roles.service.ts` (criação, atualização, troca de permissões e auditoria)
- `portal-sama-api/src/modules/roles/roles.module.ts` (integração com `AuthModule`/`AuditModule`)
- `portal-sama-api/src/modules/roles/roles.service.spec.ts` (testes unitários de mutações)
- `portal-sama-api/src/modules/permissions/dto/create-permission.dto.ts` (DTO de criação de permissão)
- `portal-sama-api/src/modules/permissions/dto/update-permission.dto.ts` (DTO de atualização de permissão)
- `portal-sama-api/src/modules/permissions/permissions.controller.ts` (rotas mutáveis do catálogo de permissões)
- `portal-sama-api/src/modules/permissions/permissions.service.ts` (criação, atualização, proteção de chaves padrão e auditoria)
- `portal-sama-api/src/modules/permissions/permissions.module.ts` (integração com `AuthModule`/`AuditModule`)
- `portal-sama-api/src/modules/permissions/permissions.service.spec.ts` (testes unitários de mutações)
- `portal-sama-api/src/modules/rbac/default-rbac.ts` (permissões `permissions.create` e `permissions.update`)
- `portal-sama-api/src/modules/rbac/default-rbac.spec.ts` (cobertura do catálogo RBAC)
- `portal-sama-api/test/app.e2e-spec.ts` (proteções CSRF das mutações de roles/permissões)
- Documentação em `docs/` atualizada com o novo estado

### O que mudou

**Roles e permissões - mutações administrativas:**
- Criados `POST /api-v2/roles`, `PATCH /api-v2/roles/:id` e `PUT /api-v2/roles/:id/permissions`.
- Criados `POST /api-v2/permissions` e `PATCH /api-v2/permissions/:id`.
- Cada mutação exige JWT, permissão granular (`roles.create`, `roles.update`, `roles.permissions`, `permissions.create`, `permissions.update`) e CSRF.
- Roles validam chaves de permissão contra o catálogo persistido antes de criar/substituir vínculos.
- O service bloqueia remover da própria role a permissão `roles.permissions`, reduzindo risco de perda acidental de administração.
- Permissões padrão do catálogo não podem ter a chave renomeada; apenas permissões customizadas podem mudar de chave.
- Auditoria registra criação/atualização de roles, troca de permissões de roles e criação/atualização de permissões.

**Validação:**
- `prisma validate`: passou com `DATABASE_URL` descartável.
- `prisma generate`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd test`: 63 testes em 11 suites.
- `npm.cmd run test:e2e`: 42 testes em 1 suite.
- `npm.cmd run build`: passou.
- `npm.cmd audit --audit-level=moderate`: 0 vulnerabilidades.

### Riscos ou pendências

- Validar fluxo com MySQL real/homologação, seed atualizado e usuários/roles/permissões persistidos.
- Integração com frontend DEV/admin ainda não foi feita.
- Migrations formais e migração de usuários legados continuam pendentes.

---

## 2026-05-12 15:25

### Arquivos alterados

- `portal-sama-api/src/modules/users/dto/create-user.dto.ts` (DTO de criação administrativa)
- `portal-sama-api/src/modules/users/dto/update-user.dto.ts` (DTO de atualização cadastral)
- `portal-sama-api/src/modules/users/dto/update-user-status.dto.ts` (DTO de status)
- `portal-sama-api/src/modules/users/dto/update-user-roles.dto.ts` (DTO de roles)
- `portal-sama-api/src/modules/users/users.controller.ts` (rotas administrativas mutáveis)
- `portal-sama-api/src/modules/users/users.service.ts` (hash de senha, roles, status e auditoria)
- `portal-sama-api/src/modules/users/users.module.ts` (integração com `AuthModule`/`AuditModule`)
- `portal-sama-api/src/modules/users/users.service.spec.ts` (testes unitários de mutações)
- `portal-sama-api/test/app.e2e-spec.ts` (proteções CSRF das mutações de usuários)
- Documentação em `docs/` atualizada com o novo estado

### O que mudou

**Usuários - mutações administrativas:**
- Criados `POST /api-v2/users`, `PATCH /api-v2/users/:id`, `PATCH /api-v2/users/:id/status` e `PUT /api-v2/users/:id/roles`.
- Cada rota exige JWT, permissão granular (`users.create`, `users.update`, `users.status`, `users.roles`) e CSRF nas mutações.
- Criação e atualização de senha usam `bcrypt` e nunca retornam `passwordHash`.
- Roles são validadas contra o catálogo persistido antes de criar/alterar vínculos.
- O service bloqueia auto-bloqueio/inativação e impede que um usuário remova a própria role administrativa (`ADMIN`/`DEV`).
- Auditoria registra criação, atualização, status e troca de roles sem gravar senha ou hash.

**Validação:**
- `prisma validate`: passou com `DATABASE_URL` descartável.
- `prisma generate`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd test`: 56 testes em 11 suites.
- `npm.cmd run test:e2e`: 37 testes em 1 suite.
- `npm.cmd run build`: passou.
- `npm.cmd audit --audit-level=moderate`: 0 vulnerabilidades.

### Riscos ou pendências

- Validar fluxo com MySQL real/homologação e usuários/roles persistidos.
- Mutações de `RolesModule` e `PermissionsModule` ainda seguem pendentes.
- Integração com frontend DEV/admin ainda não foi feita.

---

## 2026-05-12 13:52

### Arquivos alterados

- `portal-sama-api/src/modules/documents/dto/create-public-token.dto.ts` (DTO de emissão de token público)
- `portal-sama-api/src/modules/documents/dto/list-public-tokens.dto.ts` (DTO de listagem de tokens públicos)
- `portal-sama-api/src/modules/documents/documents.controller.ts` (rotas administrativas `GET/POST/DELETE /api-v2/documents/public-tokens`)
- `portal-sama-api/src/modules/documents/documents.service.ts` (emissão, listagem e revogação de tokens públicos com auditoria)
- `portal-sama-api/src/modules/documents/documents.types.ts` (tipos de resposta sem hash/segredo)
- `portal-sama-api/src/modules/rbac/default-rbac.ts` (permissão `documents.public_tokens`)
- `portal-sama-api/src/modules/documents/documents.service.spec.ts` (testes unitários de tokens públicos)
- `portal-sama-api/test/app.e2e-spec.ts` (proteções 401/403/CSRF dos endpoints administrativos)
- Documentação em `docs/` atualizada com o novo estado

### O que mudou

**Documentos - gestão administrativa de tokens públicos:**
- Criados `GET /api-v2/documents/public-tokens`, `POST /api-v2/documents/public-tokens` e `DELETE /api-v2/documents/public-tokens/:id`.
- As rotas exigem JWT e `documents.public_tokens`; `POST` e `DELETE` também exigem CSRF.
- A criação valida cliente, gera token opaco `sama_pub_*`, salva apenas SHA-256 em `PublicToken.tokenHash` e retorna o token bruto somente na resposta de criação.
- A listagem retorna metadados seguros (`clientId`, escopo, expiração, revogação, último uso e label) sem expor `tokenHash` nem segredo bruto.
- A revogação marca `revokedAt`, é idempotente para token já revogado e registra auditoria `documents.public_token.revoke`.
- A permissão `documents.public_tokens` foi adicionada ao catálogo RBAC, atribuída a `MANAGER` e herdada por `ADMIN`/`DEV`.

**Validação:**
- `prisma validate`: passou com `DATABASE_URL` descartável.
- `prisma generate`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd test`: 52 testes em 11 suites.
- `npm.cmd run test:e2e`: 33 testes em 1 suite.
- `npm.cmd run build`: passou.
- `npm.cmd audit --audit-level=moderate`: 0 vulnerabilidades.

### Riscos ou pendências

- Migration formal, seed/backfill e validação com MySQL de homologação seguem pendentes.
- Rate limiting distribuído com Redis segue pendente; a rota pública de upload continua usando o `ThrottlerGuard` atual da API com limite específico em memória.
- Ainda falta validar ClamAV real/EICAR em modo `strict` no host.

---

## 2026-05-12 10:48

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma` (modelo `PublicToken`)
- `portal-sama-api/src/modules/documents/documents.controller.ts` (rota pública `POST /api-v2/documents/public-upload`)
- `portal-sama-api/src/modules/documents/documents.service.ts` (validação de token público e upload público seguro)
- `portal-sama-api/src/modules/documents/documents.service.spec.ts` (testes de token público)
- `portal-sama-api/test/app.e2e-spec.ts` (rota pública exige token sem exigir bearer)
- Documentação em `docs/` atualizada com o novo estado

### O que mudou

**Documentos - upload público por token:**
- Criado modelo Prisma `PublicToken` com `tokenHash`, `module`, `entityType`, `entityId`, `scope`, `expiresAt`, `revokedAt`, `lastUsedAt`, `lastUsedIp` e `metadata`.
- Adicionado `POST /api-v2/documents/public-upload?token=...` com `@Public()` e throttle específico.
- O token é validado por hash SHA-256, expiração, revogação, módulo e escopo antes de validar ou persistir o arquivo.
- Upload público reaproveita validação de extensão/MIME/assinatura/tamanho, quarentena/scanner e storage privado.
- Documento público é salvo com `source: onboarding`, sem `uploadedById`, e com metadados de `publicUpload` sem gravar token bruto.
- O uso do token atualiza `lastUsedAt`/`lastUsedIp` e registra auditoria `documents.public_upload`.

**Validação:**
- `prisma validate`: passou com `DATABASE_URL` descartável.
- `prisma generate`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd test`: 49 testes em 11 suites.
- `npm.cmd run test:e2e`: 29 testes em 1 suite.
- `npm.cmd run build`: passou.
- `npm.cmd audit --audit-level=moderate`: 0 vulnerabilidades.

### Riscos ou pendências

- Ainda faltam endpoints administrativos para emissão/revogação/listagem de tokens públicos na API v2.
- Rate limiting distribuído com Redis segue pendente; a rota usa o `ThrottlerGuard` atual da API com limite específico em memória.
- Migration formal, seed/backfill e validação com MySQL de homologação seguem pendentes.

---

## 2026-05-12 10:26

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma` (relacao formal `Document` -> `DocumentStatusHistory`)
- `portal-sama-api/src/modules/documents/documents.service.ts` (update de status e historico em transacao)
- `portal-sama-api/src/modules/documents/documents.service.spec.ts` (mocks Prisma corrigidos e cobertura de historico)
- `portal-sama-api/test/app.e2e-spec.ts` (proteção de `GET /api-v2/documents/:id/status-history`)
- `docs/STATUS_IMPLEMENTACAO.md` e `docs/PENDENCIAS_TECNICAS.md` (status atualizado)

### O que mudou

**Documentos - historico estruturado de status:**
- Corrigida a implementacao iniciada pelo Copilot para compilar com `AuthenticatedUser.permissions`.
- `DocumentStatusHistory` agora tem relacao Prisma formal com `Document`, com `onDelete: Cascade`.
- `PATCH /api-v2/documents/:id/status` atualiza o documento e grava historico na mesma transacao.
- `GET /api-v2/documents/:id/status-history` segue protegido por JWT/permissao `documents.read`.

**Validacao:**
- `prisma validate`: passou com `DATABASE_URL` descartavel.
- `prisma generate`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd test`: 45 testes em 11 suites.
- `npm.cmd run test:e2e`: 28 testes em 1 suite.
- `npm.cmd run build`: passou.

### Riscos ou pendencias

- Migration Prisma formal e backfill de historico antigo em `Document.metadata.review` ainda dependem de banco/homologacao.
- Tokens publicos de onboarding e validacao ClamAV/EICAR em modo `strict` continuam pendentes.

---

## 2026-05-12 08:31

### Arquivos alterados

- `portal-sama-api/src/modules/documents/documents.service.ts` (adicionado método `validateUserDocumentAccess()`)
- `portal-sama-api/src/modules/documents/documents.service.spec.ts` (adicionados 6 testes)
- `docs/STATUS_IMPLEMENTACAO.md` (atualizado com progresso)
- `docs/CHANGELOG_TECNICO.md` (este arquivo)

### O que mudou

**Documentos - Validação centralizada:**
- ✅ Implementado `DocumentsService.validateUserDocumentAccess(userId, documentId)` que centraliza validação de acesso com dados reais em MySQL.
- ✅ Método retorna documento, usuário autenticado e flag `allowed: true` ou lança `ForbiddenException`/`NotFoundException`.
- ✅ Valida que cliente existe, usuário existe e aplica regras de escopo (ADMIN, CLIENT, DEPARTMENT, roles com departamentos mapeados).

**Testes adicionados:**
- ✅ Permite acesso a ADMIN (qualquer documento).
- ✅ Permite CLIENT acessar apenas seus próprios documentos (por `clientId`).
- ✅ Permite DEPARTMENT acessar documentos em seu departamento (com mapeamento de roles).
- ✅ Rejeita acesso quando fora de escopo (ForbiddenException).
- ✅ Retorna NotFoundException se cliente não encontrado.
- ✅ Retorna NotFoundException se usuário não encontrado.

**Validação:**
- ✅ TypeScript válido: `documents.service.ts` (1.153 linhas) e `documents.service.spec.ts` (821 linhas).

### Motivo da alteração

Desbloquear Tarefa 2.3: centralizar e melhorar lógica de validação de acesso a documentos com validação de dados reais em MySQL (cliente, usuário), preparando ground para:
1. Tarefa 2.2: Histórico estruturado de status com `DocumentStatusHistory`.
2. Tarefa 2.1: Tokens públicos de onboarding com validação segura.

### Impacto esperado

- ✅ Documentos agora possuem método de validação reutilizável por outros módulos (certificados, onboarding, etc.).
- ✅ Segurança IDOR melhorada: validação de cliente/usuário/role em uma função centralizada.
- ⚠️ Integração com telas (frontend React) ainda depende de testes e2e com navegador real.

### Riscos ou pendências

- `validateUserDocumentAccess()` foi adicionado mas ainda não integrado em endpoints HTTP (é um método público para reutilização futura).
- Testes unitários cobrem lógica, mas testes e2e com dados reais em MySQL ainda não rodaram (depende de infraestrutura de teste).
- Relação cliente-gestor-departamento pode precisar de ajustes após validação com dados reais de produção.

---

## 2026-05-11 14:45

### Arquivos alterados

- `portal-sama-api/.env` (criado com `DATABASE_URL` para MySQL em Docker)
- `docs/STATUS_IMPLEMENTACAO.md`
- `.ai-tests/runs/2026-05-11-14-45-mysql-validation/commands.md`
- `.ai-tests/runs/2026-05-11-14-45-mysql-validation/summary.md`

### O que mudou

**Infraestrutura:**
- ✅ Iniciado Docker Desktop no Windows (estava desligado).
- ✅ Criado container MySQL 8.0 em Docker na porta 3306 com `MYSQL_DATABASE=portal_sama`, `MYSQL_USER=portal_user`, `MYSQL_PASSWORD=portal_test`.
- ✅ Criado arquivo `.env` em `portal-sama-api/` com credenciais de teste (`DATABASE_URL=mysql://portal_user:portal_test@localhost:3306/portal_sama`).

**Banco de dados:**
- ✅ Schema Prisma sincronizado com MySQL via `prisma db push --skip-generate` (sem migrations formais).
- ✅ Seed RBAC executado com sucesso: 9 roles (`ADMIN`, `MANAGER`, `CLIENT`, `DEPARTMENT`, `TI`, `ACCOUNTING`, `LEGALIZATION`, `AUDITOR`, `DEV`) e 30+ permissões inseridos em banco.

**Testes:**
- ✅ Suite completa de testes reexecutada com MySQL real em 2026-05-11 14:45.
  - Lint: ✅ 0 erros
  - Unitários: ✅ 36 testes em 11 suites
  - E2e: ✅ 26 testes em 1 suite
  - Build: ✅ sem erros
  - Audit: ✅ 0 vulnerabilidades
- ✅ Confirmado: todos os 62 testes continuam passando com banco real.

### Motivo da alteração

Desbloquear Tarefa 1.1 descrita em `docs/STATUS_IMPLEMENTACAO.md`: validar MySQL real, executar seed RBAC e confirmar que todos os testes continuam passando com banco persistido.

### Impacto esperado

- ✅ Backend NestJS agora possui banco MySQL real e seed RBAC validado.
- ✅ Próximas tarefas (Tarefa 2.1, 2.2, 2.3) podem prosseguir com dados reais em MySQL.
- ✅ Integração futura de telas (auth, auditoria, admin, documentos) pode ser validada contra banco real.
- ⚠️ Docker daemon deve estar rodando para que container MySQL permaneça ativo.

### Testes executados

```bash
# Na pasta portal-sama-api/
npm.cmd run lint
npm.cmd test
npm.cmd run test:e2e
npm.cmd run build
npm.cmd audit --audit-level=moderate

# Prisma
npx prisma db push --skip-generate
npm.cmd run prisma:seed
```

**Resultado:** ✅ Todos os testes passaram com MySQL real.

### Riscos ou pendências

- Docker daemon pode parar se máquina for reiniciada; container MySQL precisará ser recriado.
- `.env` com credenciais de teste existe localmente; **não versionar**, está em `.gitignore`.
- Migrations Prisma formais não foram criadas (bloqueadas por permissões de banco); `db push` continua funcionando e sincronizando schema.
- ClamAV ainda não foi testado em modo `strict`; validação operacional é próxima tarefa.
- Vínculo cliente/gestor/departamento com dados reais não foi validado; Tarefa 2.3 cobrirá isso.

---

## 2026-05-11 13:55

### Arquivos alterados

- `portal-sama-api/src/modules/documents/document-upload-scanner.service.ts`
- `portal-sama-api/src/modules/documents/document-upload-scanner.service.spec.ts`
- `portal-sama-api/src/modules/documents/documents.service.ts`
- `portal-sama-api/src/modules/documents/documents.service.spec.ts`
- `portal-sama-api/src/modules/documents/documents.module.ts`
- `portal-sama-api/src/config/env.schema.ts`
- `portal-sama-api/.env.example`
- `.env.example`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/BANCO_DADOS_MYSQL_PRISMA.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/ROADMAP_REFATORACAO.md`
- `docs/EASYPANEL_DEPLOY.md`
- `.ai-tests/runs/2026-05-11-13-55-documents-scanner/commands.md`
- `.ai-tests/runs/2026-05-11-13-55-documents-scanner/summary.md`

### O que mudou

Criado `DocumentUploadScannerService` para o upload de documentos da API v2. O arquivo enviado agora e gravado primeiro em uma area de quarentena privada, passa por regras estaticas para conteudo texto/PDF/Office zip-like e tenta scanner externo `clamscan`/`clamdscan` antes de ser salvo no storage final.

O scanner e configuravel por `SAMA_UPLOAD_SCAN_MODE` (`off`, `best_effort`, `strict`), `SAMA_UPLOAD_SCAN_TIMEOUT_SEC`, `SAMA_UPLOAD_SCAN_BIN`, `SAMA_UPLOAD_SCAN_ARGS` e `SAMA_UPLOAD_QUARANTINE_DIR`. Em `best_effort`, a API permite o upload quando as regras estaticas passam e o scanner externo nao esta disponivel; em `strict`, indisponibilidade do scanner bloqueia o upload.

O `DocumentsService.uploadForClient` passou a executar a varredura antes de `DocumentStorageService.storeClientDocument`. Se a varredura rejeitar o arquivo, nada e gravado no storage final nem no Prisma. Quando o upload passa, `Document.metadata.scanner` guarda o resultado e a auditoria de `documents.upload` recebe status, engine e modo do scanner.

### Motivo da alteracao

Fechar a pendencia critica de quarentena/scanner no fluxo NestJS de documentos antes de qualquer integracao produtiva do painel de documentos.

### Impacto esperado

- Upload autenticado de documentos fica alinhado ao fluxo seguro documentado: validacao, quarentena, scanner, storage privado, metadata e auditoria.
- Ambientes sem ClamAV ainda conseguem evoluir em `best_effort`, mas producao deve validar scanner real e migrar para `strict`.
- O frontend legado continua usando PHP; nenhuma tela foi conectada ao NestJS nesta etapa.

### Testes executados

- `npm.cmd run format`: passou.
- `npm.cmd run lint`: falhou uma vez por `no-control-regex`, foi corrigido e passou na reexecucao.
- `npm.cmd test`: passou, 11 suites/36 testes.
- `npm.cmd run test:e2e`: passou, 1 suite/26 testes.
- `npm.cmd run build`: passou.
- `npm.cmd run prisma:generate`: passou com `DATABASE_URL` de teste.
- `npm.cmd run prisma:validate`: passou com `DATABASE_URL` de teste.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou sem erro de whitespace; apenas avisos LF/CRLF do Git no Windows.

### Testes bloqueados

- `npm.cmd run prisma:migrate:status`: bloqueado por `Schema engine error` contra MySQL em `localhost:3306`, compativel com ausencia de MySQL local/homologacao validado.
- Smoke operacional com EICAR/ClamAV real nao foi executado porque depende do scanner instalado/configurado no host.

### Riscos ou pendencias

- Validar `SAMA_UPLOAD_SCAN_MODE=strict` em homologacao com ClamAV real e fixture EICAR antes de producao.
- Validar upload/download reais com MySQL, storage persistente e dados de escopo cliente/gestor/departamento.
- Fluxos publicos por token continuam pendentes.

## 2026-05-11 13:33

### Arquivos alterados

- `portal-sama-api/src/modules/documents/documents.controller.ts`
- `portal-sama-api/src/modules/documents/documents.service.ts`
- `portal-sama-api/src/modules/documents/documents.service.spec.ts`
- `portal-sama-api/src/modules/documents/dto/update-document-status.dto.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.spec.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- `.ai-tests/runs/2026-05-11-13-33-documents-review/commands.md`
- `.ai-tests/runs/2026-05-11-13-33-documents-review/summary.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/BANCO_DADOS_MYSQL_PRISMA.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/paginas/client-painel.md`

### O que mudou

Criado `PATCH /api-v2/documents/:id/status` para atualizar o status de documentos para `PENDING`, `APPROVED` ou `REJECTED`.

A rota exige JWT, permissao `documents.review`, CSRF double-submit, validacao de escopo por papel/departamento e registra auditoria via `AuditService`. O historico imediato da revisao fica em `Document.metadata.review` com status, motivo opcional, usuario e horario, sem criar nova coluna antes de migration real.

O catalogo RBAC ganhou a permissao `documents.review`, atribuida a `MANAGER`, `DEPARTMENT`, `ACCOUNTING`, `LEGALIZATION` e herdada por `ADMIN`/`DEV`.

### Motivo da alteracao

Avancar a pendencia mapeada de aprovacao/rejeicao de documentos sem alterar o schema MySQL antes de validar migrations em homologacao.

### Impacto esperado

- Documentos enviados como `PENDING` podem ser aprovados, rejeitados ou reabertos para pendencia pela API v2.
- Checklists passam a exibir `confirmed_by` e `confirmed_at` quando o documento aprovado possui metadata de revisao.
- O frontend legado continua usando PHP; nenhuma tela foi conectada ao NestJS nesta etapa.
- A modelagem definitiva de historico (`document_status_history`) segue pendente para MySQL real.

### Testes executados

- `npm.cmd run format`: passou na reexecucao.
- `npm.cmd run lint`: passou na reexecucao.
- `npm.cmd test`: passou, 10 suites/32 testes.
- `npm.cmd run test:e2e`: passou, 1 suite/26 testes.
- `npm.cmd run build`: passou.
- `npm.cmd run prisma:generate`: passou com `DATABASE_URL` de teste.
- `npm.cmd run prisma:validate`: passou com `DATABASE_URL` de teste.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou sem erro de whitespace; apenas avisos LF/CRLF do Git no Windows.

### Testes bloqueados

- `npm.cmd run prisma:migrate:status`: bloqueado por `Schema engine error` contra MySQL em `localhost:3306`, compativel com ausencia de MySQL local/homologacao validado.

### Riscos ou pendencias

- Validar `documents.review` em seed real e usuarios/roles persistidos.
- Criar migration/modelo de historico de status quando o banco MySQL real estiver disponivel.
- Conectar a tela `Client/painel.html` ou o futuro React somente depois de validar escopo real cliente/gestor/departamento.
- Ainda faltam validacao operacional de ClamAV em modo `strict` e fluxos publicos por token.

## 2026-05-11 08:59

### Arquivos alterados

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/src/modules/documents/document-template.ts`
- `portal-sama-api/src/modules/documents/documents.controller.ts`
- `portal-sama-api/src/modules/documents/documents.service.ts`
- `portal-sama-api/src/modules/documents/documents.types.ts`
- `portal-sama-api/src/modules/documents/dto/create-document-requirement.dto.ts`
- `portal-sama-api/src/modules/documents/dto/required-pending-summary.dto.ts`
- `portal-sama-api/src/modules/documents/documents.service.spec.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.spec.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- `.ai-tests/runs/2026-05-11-08-59-documents-requirements/commands.md`
- `.ai-tests/runs/2026-05-11-08-59-documents-requirements/summary.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/BANCO_DADOS_MYSQL_PRISMA.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/ROADMAP_REFATORACAO.md`
- `docs/paginas/client-painel.md`

### O que mudou

O `DocumentsModule` foi ampliado para cobrir mais actions do legado `api/client_documents.php`: `template`, `required_pending_summary` e `add_custom`.

Foram criados `GET /api-v2/documents/templates`, `GET /api-v2/documents/required-pending-summary` e `POST /api-v2/documents/custom-requirements`. A rota `GET /api-v2/clients/:clientId/documents` agora retorna checklist de requisitos documentais, combinando template, requisitos customizados e upload mais recente por tipo.

O Prisma ganhou o modelo `DocumentRequirement`, com chave única por cliente/tipo customizado. O RBAC ganhou `documents.requirements`, atribuído a `MANAGER` e herdado por `ADMIN`/`DEV`.

### Motivo da alteração

Avançar a migração de documentos sem depender de frontend React ou MySQL real, fechando contratos backend mapeados antes da integração de tela.

### Impacto esperado

- O backend v2 passa a expor o template de documentos e o checklist por cliente.
- Gestores podem criar requisitos documentais customizados via endpoint protegido por JWT, `documents.requirements`, CSRF e auditoria.
- O frontend legado continua usando PHP; nenhuma tela foi conectada ao NestJS nesta etapa.
- Migrations reais e validação com dados persistidos seguem pendentes até haver MySQL/homologação.

### Testes executados

- `npm.cmd run format`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd test`: passou, 10 suítes/30 testes.
- `npm.cmd run test:e2e`: passou, 1 suíte/23 testes.
- `npm.cmd run build`: passou.
- `npm.cmd run prisma:generate`: passou com `DATABASE_URL` de teste.
- `npm.cmd run prisma:validate`: passou com `DATABASE_URL` de teste.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou sem erro de whitespace; apenas avisos LF/CRLF do Git no Windows.

### Testes bloqueados

- `npm.cmd run prisma:migrate:status`: bloqueado por `Schema engine error` contra MySQL em `localhost:3306`, compatível com ausência de MySQL local/homologação validado.

### Riscos ou pendências

- Criar migration real e executar seed RBAC em MySQL/homologação.
- Validar `DocumentRequirement` e o checklist com dados reais.
- Implementar aprovação/rejeição de documentos, ClamAV/quarentena real e fluxos públicos por token.

## 2026-05-11 08:40

### Arquivos alterados

- `portal-sama-api/package.json`
- `portal-sama-api/package-lock.json`
- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/documents/**`
- `portal-sama-api/test/app.e2e-spec.ts`
- `.ai-tests/runs/2026-05-11-08-40-documents/commands.md`
- `.ai-tests/runs/2026-05-11-08-40-documents/summary.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MAPEAMENTO_MIGRACAO_APIS.md`
- `docs/ROADMAP_REFATORACAO.md`

### O que mudou

Criado `DocumentsModule` inicial na API NestJS, retomando o ponto posterior a `UsersModule`, `RolesModule` e `PermissionsModule`.

Foram implementadas rotas protegidas por JWT e permissões para listar documentos, consultar detalhe, enviar arquivo, baixar arquivo e arquivar logicamente um documento: `GET /api-v2/documents`, `GET /api-v2/clients/:clientId/documents`, `GET /api-v2/documents/:id`, `GET /api-v2/documents/:id/download`, `POST /api-v2/documents/clients/:clientId/upload`, `POST /api-v2/documents/upload` e `DELETE /api-v2/documents/:id`.

O upload valida extensão, MIME, assinatura mágica, tamanho e SHA-256, salva em storage privado configurado por `STORAGE_PRIVATE_PATH`, não expõe `storageKey` na resposta e registra auditoria de upload/download/archive via `AuditService`. As mutações exigem CSRF.

O modelo Prisma `Document` foi ampliado com `department`, `storageKey` único, `source`, `metadata` e índices adicionais. As dependências `multer` e `@types/multer` foram adicionadas para multipart.

### Motivo da alteração

Dar continuidade à etapa crítica de documentos e storage privado antes da integração do painel do cliente ou dos fluxos públicos de onboarding.

### Impacto esperado

- A API v2 passa a ter contrato inicial de documentos com validações de segurança.
- O frontend legado continua usando PHP; nenhuma tela foi conectada ao NestJS nesta etapa.
- Templates de documentos obrigatórios, resumo de pendências, tokens públicos de onboarding, ClamAV/quarentena real e escopo cliente/gestor com dados reais permanecem pendentes.

### Testes executados

- `npm.cmd install multer`: passou.
- `npm.cmd install -D @types/multer`: passou.
- `npm.cmd run format`: passou.
- `npm.cmd run prisma:generate`: passou com `DATABASE_URL` de teste.
- `npm.cmd run lint`: passou.
- `npm.cmd test`: passou, 10 suítes/26 testes.
- `npm.cmd run test:e2e`: passou, 1 suíte/18 testes após correção de wiring do `DocumentsModule`.
- `npm.cmd run build`: passou.
- `npm.cmd run prisma:validate`: passou com `DATABASE_URL` de teste.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou; Git exibiu apenas avisos LF/CRLF no Windows.

### Testes bloqueados

- `npm.cmd run prisma:migrate:status`: bloqueado por `Schema engine error` contra MySQL em `localhost:3306`, compatível com ausência de MySQL local/homologação validado.
- Teste real de upload/download com banco e storage persistente não foi executado porque depende de MySQL/homologação e ambiente de arquivos definitivo.

### Riscos ou pendências

- Validar o módulo com MySQL real, seed RBAC e permissões persistidas.
- Atualizado em 2026-05-11 13:55: quarentena/scanner configuravel foi implementado no fluxo NestJS; falta validar ClamAV strict no host.
- Substituir a heurística temporária de escopo por vínculo real entre usuário, cliente, gestor e departamento.
- Implementar templates/pendências obrigatórias e fluxos públicos de onboarding antes de substituir `api/client_documents.php`.

## 2026-05-08 14:57

### Arquivos alterados

- `portal-sama-api/package.json`
- `portal-sama-api/prisma/seed.ts`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/rbac/*`
- `portal-sama-api/src/modules/permissions/*`
- `portal-sama-api/src/modules/roles/*`
- `portal-sama-api/src/modules/users/*`
- `portal-sama-api/test/app.e2e-spec.ts`
- `.ai-tests/runs/2026-05-08-14-57/commands.md`
- `.ai-tests/runs/2026-05-08-14-57/summary.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`

### O que mudou

Criados `UsersModule`, `RolesModule` e `PermissionsModule` iniciais na API NestJS, com rotas somente leitura `GET /api-v2/users`, `GET /api-v2/users/:id`, `GET /api-v2/roles`, `GET /api-v2/roles/:id` e `GET /api-v2/permissions`.

As rotas exigem bearer JWT e permissões granulares `users.read`, `roles.read` e `permissions.read`. As respostas de usuários não selecionam `passwordHash` e agregam roles/permissões a partir das relações Prisma.

Também foi criado o catálogo inicial de RBAC em `DEFAULT_PERMISSIONS`/`DEFAULT_ROLES` e o script `npm.cmd run prisma:seed`, que popula permissões e roles sem criar usuário, senha ou segredo.

### Motivo da alteração

Dar continuidade à etapa crítica de RBAC/permissões antes dos módulos sensíveis, substituindo gradualmente o endpoint legado `api/storage.php?action=admin_users_*` por contratos tipados e protegidos em `/api-v2`.

### Impacto esperado

- A API v2 passa a expor consultas administrativas protegidas para usuários, roles e permissões.
- A permissão `audit.read` passa a existir no seed inicial junto das permissões administrativas.
- O frontend legado continua usando PHP; nenhuma tela foi conectada ao NestJS nesta etapa.
- Mutações administrativas permanecem pendentes para etapa posterior com CSRF e auditoria de alteração.

### Testes executados

- `npm.cmd run format`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd test`: passou, 8 suítes/18 testes.
- `npm.cmd run test:e2e`: passou, 1 suíte/12 testes.
- `npm.cmd run build`: passou.
- `npm.cmd run prisma:validate`: passou quando executado com `DATABASE_URL` de teste.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou sem erro de whitespace; apenas avisos LF/CRLF do Git no Windows.

### Testes bloqueados

- Primeira tentativa de `npm.cmd run prisma:validate` falhou por `DATABASE_URL` ausente no ambiente do shell; reexecutado com `DATABASE_URL` de teste e passou.
- `npm.cmd run prisma:migrate:status`: bloqueado porque não há MySQL local/homologação validado em `localhost:3306`.
- `npm.cmd run prisma:seed`: não executado porque depende de MySQL real/homologação.

### Riscos ou pendências

- Validar listagens administrativas com MySQL real e seed executado.
- Implementar mutações administrativas com CSRF, auditoria e bloqueios contra escalonamento de privilégio.
- Migrar ou mapear usuários legados de `sama_usuarios_login` para o modelo Prisma.
- Criar validação de escopo real por recurso antes de liberar módulos de documentos, certificados e clientes.

## 2026-05-08 14:40

### Arquivos alterados

- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/modules/audit/*`
- `portal-sama-api/src/modules/auth/auth.module.ts`
- `portal-sama-api/src/modules/auth/auth.service.ts`
- `portal-sama-api/src/modules/auth/auth.service.spec.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- `.ai-tests/runs/2026-05-08-14-40/commands.md`
- `.ai-tests/runs/2026-05-08-14-40/summary.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/ROADMAP_REFATORACAO.md`

### O que mudou

Criado `AuditModule` inicial na API NestJS, com `AuditService`, `AuditController`, DTO de filtros, tipos de resposta e endpoints protegidos `GET /api-v2/audit/logs` e `GET /api-v2/audit/logs/:id`.

O `AuthService` passou a registrar eventos por `AuditService` centralizado. O novo serviço mascara metadados sensíveis por chave antes de persistir, evitando gravar valores associados a senha, token, secret, cookie, JWT ou credenciais.

Também foi registrado no roadmap que a Integração Acessórias permanece como funcionalidade futura pós-plano principal, sem implementação nesta etapa.

### Motivo da alteração

Atender à etapa de auditoria centralizada do roadmap antes da migração de documentos, certificados e demais módulos sensíveis, mantendo rastreabilidade desde a base da API v2.

### Impacto esperado

- A API v2 passa a ter módulo de auditoria reutilizável por outros módulos.
- Leitura de logs exige bearer JWT e permissão `audit.read`.
- O frontend legado continua usando `api/storage.php?action=audit_list`; nenhuma tela foi conectada ao NestJS nesta etapa.
- Fluxo real ainda depende de MySQL/homologação e seed de permissões.

### Testes executados

- `npm.cmd run format`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd test`: passou, 4 suítes/11 testes.
- `npm.cmd run test:e2e`: passou, 1 suíte/6 testes.
- `npm.cmd run build`: passou.
- `npm.cmd run prisma:validate`: passou.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou sem erro de whitespace; apenas avisos LF/CRLF do Git no Windows.

### Testes bloqueados

- `npm.cmd run prisma:migrate:status`: bloqueado porque não há MySQL local/homologação validado em `localhost:3306`.

### Riscos ou pendências

- Validar `/api-v2/audit/logs` com MySQL real e usuário com `audit.read`.
- Criar seed para roles/permissões, incluindo `audit.read`.
- Integrar futura tela `/auditoria` ao endpoint NestJS.
- Definir política de retenção/exportação e cobertura de auditoria para próximos módulos.

## 2026-05-08 14:14

### Arquivos alterados

- `.env.example`
- `portal-sama-api/.env.example`
- `portal-sama-api/src/config/env.schema.ts`
- `portal-sama-api/src/modules/auth/auth.controller.ts`
- `portal-sama-api/src/modules/auth/auth.module.ts`
- `portal-sama-api/src/modules/auth/auth.types.ts`
- `portal-sama-api/src/modules/auth/csrf.service.ts`
- `portal-sama-api/src/modules/auth/csrf.service.spec.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- `.ai-tests/runs/2026-05-08-14-14/summary.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/SEGURANCA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/EASYPANEL_DEPLOY.md`

### O que mudou

Adicionada proteção CSRF double-submit assinada ao `AuthModule`. A API agora expõe `GET /api-v2/auth/csrf`, grava cookie CSRF não sensível, retorna o token no JSON e exige o mesmo token no cabeçalho `x-csrf-token` para `POST /api-v2/auth/login`, `/refresh` e `/logout`.

O token CSRF é assinado por HMAC, usa `CSRF_SECRET` quando configurado e usa `JWT_REFRESH_SECRET` como fallback. Também há validação de `Origin`/`Referer` quando esses cabeçalhos existem.

### Motivo da alteração

Fechar a pendência de segurança documentada para mutações baseadas em cookie antes de conectar qualquer frontend React ou página legada ao `/api-v2/auth/*`.

### Impacto esperado

- Reduz risco de CSRF em login, refresh e logout da API v2.
- Cria contrato explícito para o frontend: chamar `/auth/csrf` antes de mutações de autenticação.
- Não altera o PHP legado nem conecta telas ao NestJS.
- Fluxo completo ainda precisa ser validado com MySQL real e frontend.

### Testes executados

- `npm.cmd run format`: passou.
- `npm.cmd run lint`: passou.
- `npm.cmd test`: passou, 3 suítes/7 testes.
- `npm.cmd run test:e2e`: passou, 1 suíte/4 testes.
- `npm.cmd run build`: passou.
- `npm.cmd run prisma:validate`: passou.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou sem erro de whitespace; apenas avisos LF/CRLF do Git no Windows.

### Testes bloqueados

- `npm.cmd run prisma:migrate:status`: bloqueado porque não há MySQL local/homologação validado em `localhost:3306`.

### Riscos ou pendências

- Testar o fluxo real `csrf -> login -> refresh -> logout` com banco MySQL de homologação.
- Implementar client frontend que mantenha access token em memória e envie `x-csrf-token`.
- Reaplicar a estratégia CSRF nas próximas mutações sensíveis de módulos de negócio.

## 2026-05-08 10:25

### Arquivos alterados

- `portal-sama-api/package.json`
- `portal-sama-api/package-lock.json`
- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/common/decorators/current-user.decorator.ts`
- `portal-sama-api/src/modules/auth/*`
- `portal-sama-api/test/app.e2e-spec.ts`
- `.ai-tests/runs/2026-05-08-10-25/summary.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`

### O que mudou

Criado `AuthModule` inicial na API NestJS, expondo `/api-v2/auth/login`, `/api-v2/auth/refresh`, `/api-v2/auth/logout`, `/api-v2/auth/me` e `/api-v2/auth/forgot-password`.

O login usa `username`/`password`, valida senha com `bcrypt`, emite access JWT curto, cria refresh token opaco em cookie HttpOnly/Secure/SameSite, grava apenas HMAC-SHA-256 do refresh token, rotaciona refresh em `/refresh` e revoga em `/logout`.

Também foram adicionados tipos de usuário autenticado, guard JWT, auditoria de eventos de autenticação e índice único em `RefreshToken.tokenHash`.

O script `npm run format` da API foi corrigido para usar Prettier nos arquivos TypeScript e `prisma format` no schema Prisma.

### Motivo da alteração

Dar continuidade à migração descrita no roadmap, priorizando autenticação antes de usuários/permissões e módulos sensíveis.

### Impacto esperado

- A API v2 passa a ter base de autenticação testável.
- O frontend legado continua usando PHP/sessão; nenhuma tela foi conectada ao NestJS nesta etapa.
- Persistência real de usuários/refresh tokens depende de MySQL/migrations de homologação.
- CSRF final para mutações baseadas em cookie segue ponto a validar antes de produção.

### Testes executados

- `npm.cmd run prisma:format`: passou.
- `npm.cmd run prisma:generate`: passou com `DATABASE_URL` de teste.
- `npm.cmd run format`: passou após corrigir o script de formatação.
- `npm.cmd run lint`: passou.
- `npm.cmd test`: passou, 2 suítes/4 testes.
- `npm.cmd run test:e2e`: passou, 1 suíte/2 testes.
- `npm.cmd run build`: passou.
- `npm.cmd run prisma:validate`: passou.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou sem erro de whitespace; apenas avisos LF/CRLF do Git no Windows.

### Testes bloqueados

- `npm.cmd run prisma:migrate:status`: bloqueado porque não há MySQL local/homologação validado em `localhost:3306`.
- Primeira tentativa de `npm.cmd run format`: falhou porque o Prettier não infere parser para `schema.prisma`; corrigido para chamar `prisma format`.

### Riscos ou pendências

- Implementar/validar CSRF para `/api-v2/auth/refresh` e `/api-v2/auth/logout` antes de uso em produção.
- Validar schema e seed/migração de usuários contra MySQL real.
- Conectar frontend React somente após definir armazenamento em memória do access token e fluxo de refresh.

## 2026-05-08 10:04

### Arquivos alterados

- `.gitignore`
- `portal-sama-api/package.json`
- `portal-sama-api/package-lock.json`
- `portal-sama-api/nest-cli.json`
- `portal-sama-api/tsconfig.json`
- `portal-sama-api/tsconfig.build.json`
- `portal-sama-api/eslint.config.mjs`
- `portal-sama-api/.prettierrc`
- `portal-sama-api/.env.example`
- `portal-sama-api/Dockerfile`
- `portal-sama-api/.dockerignore`
- `portal-sama-api/src/main.ts`
- `portal-sama-api/src/app.bootstrap.ts`
- `portal-sama-api/src/app.module.ts`
- `portal-sama-api/src/config/env.schema.ts`
- `portal-sama-api/src/common/decorators/current-user.decorator.ts`
- `portal-sama-api/src/common/decorators/permissions.decorator.ts`
- `portal-sama-api/src/common/decorators/public.decorator.ts`
- `portal-sama-api/src/common/guards/permissions.guard.ts`
- `portal-sama-api/src/common/filters/http-exception.filter.ts`
- `portal-sama-api/src/common/interceptors/request-id.interceptor.ts`
- `portal-sama-api/src/database/database.module.ts`
- `portal-sama-api/src/database/prisma.service.ts`
- `portal-sama-api/src/modules/health/*`
- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/.gitkeep`
- `portal-sama-api/test/*`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/EASYPANEL_DEPLOY.md`

### O que mudou

Criada a fundação `portal-sama-api` em NestJS/TypeScript para conviver com o PHP legado em `/api` e expor a nova API sob `/api-v2`.

A API recebeu configuração de ambiente validada com Zod, Helmet, CORS, throttling, `ValidationPipe`, filtro global de exceções, request id, Swagger fora de produção, Prisma/MySQL inicial e `GET /api-v2/health`.

### Motivo da alteração

Atender à Etapa 2 do `docs/ROADMAP_REFATORACAO.md` e ao `docs/GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`, criando a base técnica antes da migração de autenticação e módulos de negócio.

### Impacto esperado

- O backend alvo deixa de estar "não iniciado" e passa a ter fundação testável.
- Módulos de negócio PHP ainda não foram substituídos.
- Prisma passa a ter schema inicial validado, mas sem migrations reais aplicadas.
- EasyPanel passa a ter um Dockerfile inicial para a API NestJS.

### Testes executados

- `npm.cmd run prisma:format`: passou.
- `npm.cmd run prisma:generate`: passou com `DATABASE_URL` de teste.
- `npm.cmd run prisma:validate`: passou com `DATABASE_URL` de teste.
- `npm.cmd run lint`: passou.
- `npm.cmd test`: passou, 1 suíte/1 teste.
- `npm.cmd run test:e2e`: passou, 1 suíte/1 teste.
- `npm.cmd run build`: passou.
- `npm.cmd audit --audit-level=moderate`: passou, 0 vulnerabilidades.
- `git diff --check`: passou sem erro de whitespace; apenas avisos LF/CRLF do Git no Windows.

### Testes bloqueados

- `npm.cmd run prisma:migrate:status`: bloqueado porque não há MySQL local/homologação validado em `localhost:3306`.

### Riscos ou pendências

- Implementar `AuthModule` antes de qualquer migração de tela ou módulo sensível.
- Validar o schema Prisma contra o MySQL real antes de gerar migrations.
- Proteger/desabilitar Swagger em produção, já condicionado por `NODE_ENV`.
- Continuam pendentes os lints PHP por ausência de `php` no PATH local.

## 2026-05-08 09:31

### Arquivos alterados

- `.gitignore`
- `.env.example`
- `SolicitacaoAcesso/enviar_acesso.php`
- `SolicitacaoAcesso/verificar_codigo.php`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/INVENTARIO_SEGURANCA.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/DECISOES_ARQUITETURA.md`

### O que mudou

Criado `.env.example` com placeholders e sem segredos reais. O `.gitignore` passou a permitir o versionamento desse template enquanto mantém `.env` e `.env.*` ignorados.

`SolicitacaoAcesso/enviar_acesso.php` e `SolicitacaoAcesso/verificar_codigo.php` deixaram de habilitar `display_errors` de forma incondicional. O debug público agora só é ativado explicitamente por `SAMA_DEBUG`, `APP_DEBUG` ou `SAMA_DISPLAY_ERRORS`.

### Motivo da alteração

Atender às pendências de segurança registradas após a auditoria inicial, reduzindo risco de exposição de stack trace em produção e criando uma base segura de configuração para os próximos passos de migração.

### Impacto esperado

- Setup local/deploy passa a ter template de variáveis sem expor credenciais reais.
- Erros PHP deixam de ser exibidos por padrão nos endpoints públicos alterados.
- Não houve migração de regra de negócio nem mudança de contrato de API intencional.

### Testes executados

- `git diff --check` nos arquivos alterados: sem erro de whitespace.
- `git check-ignore -v .env.example`: confirmado que `.env.example` não está ignorado.
- `git check-ignore -v .ai-tests/README.md`: confirmado que `.ai-tests/` continua ignorado.
- `rg` nos dois endpoints: confirmado que `display_errors` ficou condicional por variável de ambiente.

### Testes bloqueados

- `php -v`, `php -l SolicitacaoAcesso/enviar_acesso.php` e `php -l SolicitacaoAcesso/verificar_codigo.php`: bloqueados porque `php` não está disponível no PATH local.

### Riscos ou pendências

- Executar lint PHP assim que o ambiente permitir.
- Validar CSRF, sessão e escopo dos fluxos de solicitação de acesso em etapa posterior.
- Revisar `.env.example` com os valores esperados do EasyPanel real, sem versionar segredos.

## 2026-05-08 09:10

### Arquivos alterados

- `.gitignore`
- `.ai-tests/README.md`
- `docs/STATUS_IMPLEMENTACAO.md`
- `docs/CHANGELOG_TECNICO.md`
- `docs/DECISOES_ARQUITETURA.md`
- `docs/PENDENCIAS_TECNICAS.md`
- `docs/RELATORIO_TESTES.md`
- `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `docs/INVENTARIO_ENDPOINTS.md`
- `docs/INVENTARIO_BANCO_DADOS.md`
- `docs/INVENTARIO_SEGURANCA.md`

### O que mudou

Criados os arquivos obrigatórios de acompanhamento, criada a estrutura local `.ai-tests/`, adicionada `.ai-tests/` ao `.gitignore` e removida a regra que ignorava `docs/` no Git.

Também foi registrada a auditoria inicial de continuidade: estado real do projeto, divergências principais, testes disponíveis, testes não executados, pendências, matrizes e inventários iniciais.

### Motivo da alteração

Atender ao Prompt Mestre em `docs/INSTRUÇÕEDS_AI.MD`, manter continuidade entre ciclos de trabalho e corrigir a divergência em que `docs/` estava ignorado apesar de ser a fonte de verdade do projeto.

### Impacto esperado

- Documentação de continuidade passa a existir e fica rastreável pelo Git.
- `.ai-tests/` permanece local e ignorada.
- Nenhuma funcionalidade de negócio foi implementada.
- Nenhum endpoint, tela, schema ou fluxo de produção foi alterado.

### Testes executados

- Comando: não executado teste automatizado.
- Resultado: Não executado.
- Observações: `php -v` falhou por PHP ausente no PATH; `npm -v` falhou por política de execução do PowerShell; não há `package.json` no estado atual.

### Riscos ou pendências

- Criar `.env.example` seguro.
- Revisar `.env` local sem expor valores.
- Criar NestJS/React/Prisma conforme roadmap.
- Formalizar comandos de teste.
- Validar endpoints PHP com `php -l` quando o ambiente permitir.
