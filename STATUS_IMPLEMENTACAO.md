# Status de Implementação - Portal Sama


## Atualizacao complementar 2026-05-28 10:06 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Home por perfil passou a usar dados read-only do Acessorias via API v2, conforme variaveis existentes no `.env` da API.
- **Backend/API v2:** criado `AcessoriasHomeModule` com `GET /api-v2/integrations/acessorias/home-summary`, protegido por JWT. A rota usa `ACESSORIAS_BASE_URL` e `ACESSORIAS_TOKEN` somente no backend, com path/header/scheme/timeout configuraveis.
- **Normalizacao:** o servico aceita payloads comuns do Acessorias (`data`, `items`, `entregas`, `deliveries`, `obrigacoes`, `results`), normaliza cliente, obrigacao, responsavel, departamento, competencia, vencimento, baixa e status.
- **Frontend React:** `/home` agora monta cards e blocos por perfil com pendencias, vencimentos, atrasos, entregas baixadas, carteira por responsavel e diagnostico da integracao, mantendo atalhos por permissao.
- **Testes/validacao:** passaram teste unitario focado da API, lint/build da API, `prisma:validate` com `DATABASE_URL` dummy, lint/build do Web e Playwright local focado em Home.
- **Pendente:** publicar API/Web, validar contrato real da API do Acessorias no EasyPanel, repetir por usuarios reais de colaborador/gestor/admin e decidir o aceite UX/UI da Home.


## Atualizacao complementar 2026-05-27 16:44 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Segunda fatia backend de responsabilidades normalizadas de clientes implementada: transferencia auditada de responsabilidades ativas entre responsaveis.
- **Backend/API v2:** criado `TransferClientAssignmentDto` e endpoint `POST /api-v2/client-assignments/transfer`, protegido por CSRF e permissao `client_assignments.transfer`.
- **Regras de negocio:** a transferencia valida escopo, responsabilidade ativa, departamento controlado, responsavel interno ativo, compatibilidade do responsavel com o departamento e bloqueio de duplicidade `PRIMARY` ativa; a responsabilidade anterior vira `TRANSFERRED` e uma nova responsabilidade `ACTIVE` e criada com historico.
- **RBAC/auditoria:** a permissao ja existente `client_assignments.transfer` passou a proteger uma rota real; cada transferencia registra `client_assignments.transfer` em auditoria centralizada.
- **Testes/validacao:** passaram testes focados de client assignments/RBAC/catalogo, `prisma:validate`, lint e build da API.
- **Pendente:** aplicar migrations/seeds no MySQL real, validar o endpoint com usuario real/CSRF real no EasyPanel, criar UI de transferencia/carteira e executar backfill gradual de `clients.metadata`.


## Atualizacao complementar 2026-05-27 16:31 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Primeira fundacao backend para responsabilidade normalizada de clientes implementada, reduzindo a dependencia exclusiva de `clients.metadata` documentada em `ux-ui-docs/11-responsabilidade-clientes-usuarios.md`.
- **Backend/API v2:** criado `ClientAssignmentsModule`, modelo Prisma `ClientDepartmentAssignment`, migration `20260527162000_add_client_department_assignments` e endpoints `GET/POST /api-v2/clients/:clientId/assignments`, `PATCH /api-v2/client-assignments/:id` e `POST /api-v2/client-assignments/:id/end`.
- **Regras de negocio:** criacao/edicao validam departamento no catalogo controlado, responsavel/gestor ativos, usuario interno, escopo por departamento e bloqueio de dois vinculos `PRIMARY` ativos para o mesmo cliente/departamento.
- **RBAC/auditoria:** adicionadas permissoes `client_assignments.read/create/update/transfer/end/audit`; criacao, atualizacao e encerramento registram auditoria centralizada.
- **Testes/validacao:** passaram `prisma:format`, `prisma:generate`, testes focados de client assignments/RBAC/catalogo, `prisma:validate`, lint e build da API.
- **Pendente:** aplicar migrations/seeds no MySQL real, validar endpoints com usuarios reais, criar UI de responsabilidades no cliente/colaborador/gestor e migrar gradualmente leituras de `clients.metadata` para a tabela nova.


## Atualizacao complementar 2026-05-27 16:11 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Segunda fatia de governanca de departamentos controlados implementada: colaboradores deixam de aceitar departamento livre nas rotas administrativas e telas React principais.
- **Backend/API v2:** `CollaboratorsService` agora valida `mainDepartment`/`departamento` pelo `DepartmentCatalogService` em cadastro, edicao e filtro, rejeitando departamento inexistente/inativo com `DEPARTMENT_NOT_FOUND`.
- **Frontend React:** `/dev/colaboradores`, `/dev/colaboradores/novo` e `/colaboradores/visao-geral` carregam `GET /api-v2/departments` e usam `select` para filtro, criacao e edicao de colaboradores.
- **Testes/validacao:** passaram testes focados de colaboradores/usuarios/catalogo/RBAC, `prisma:validate`, lint/build da API, build/lint do Web. A primeira execucao focada falhou por ordem de validacao em `CollaboratorsService.create`; foi corrigida para validar departamento antes de resolver roles e reexecutada com sucesso.
- **Pendente:** aplicar migration/seed no MySQL real do EasyPanel, publicar API/Web atualizados e validar usuarios/colaboradores no ambiente real; clientes/responsabilidades e demais fluxos departamentais ainda precisam migrar para entidade propria e catalogo controlado.


## Atualizacao complementar 2026-05-27 15:35 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Primeira pendencia de governanca UX/UI implementada: departamento deixou de ser campo livre no cadastro/edicao de usuarios da area DEV/Admin.
- **Backend/API v2:** criado catalogo controlado de departamentos em `Department`/`departments`, migration `20260527153000_add_controlled_departments`, seed operacional e endpoint `GET /api-v2/departments` protegido por `departments.read`.
- **Usuarios/RBAC:** `UsersService` agora valida `mainDepartment` contra o catalogo controlado e rejeita departamento inexistente/inativo com `DEPARTMENT_NOT_FOUND`; RBAC ganhou `departments.read` e `departments.manage`.
- **Frontend React:** `/dev` carrega o catalogo de departamentos e usa `select` no filtro, criacao e edicao de usuarios; o envio continua compativel com `mainDepartment` canonico.
- **Testes/validacao:** passaram `prisma:generate`, testes focados de usuarios/catalogo/RBAC, `prisma:validate`, lint/build da API, build/lint do Web.
- **Pendente:** aplicar migration/seed no MySQL real via `prisma migrate deploy`/`prisma:seed`, validar com usuario real ADMIN/DEV no EasyPanel e depois expandir a governanca para colaboradores, clientes, responsabilidades e demais telas com departamento.

## Atualizacao complementar 2026-05-27 14:06 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Corrigida a decisao do leiaute Dominio do Integra-AI: o fluxo atual do Dominio usa o conjunto `Lancamentos Contabeis em Lote`, que espera registros fixos `01/02/03/99`, nao `0000/0451`.
- **Causa do erro anterior:** o Dominio interpretava `0451|...` como registro fixo `04 - Rateios Gerenciais`, deslocando sequencial, contas e valor.
- **Backend/API v2:** o leiaute oficial agora e `dominio_importador_lancamentos_lote_01_02_03_99`; o gerador monta registro `01`, pares `02`/`03` e trailer `99`, validando tamanhos 54/150/664/100 e ausencia de pipe.
- **Frontend React:** a constante exibida na tela passou para `Dominio Importador 01/02/03/99`; `Dominio Sistemas com Separador - 0000/0451` nao aparece no fluxo principal.
- **Testes:** os testes de engine agora validam TXT sem pipe, primeira linha `01`, pares `02`/`03`, ultima linha com 100 caracteres `9` e tamanhos fixos.
- **Validacao:** Passaram testes focados da API, suite completa da API (`176` testes), lint/build/Prisma validate da API, lint/build do Web, Playwright focado do Integra-AI e Playwright local completo.
- **Pendente:** importar TXT real no Dominio usando `Lancamentos Contabeis em Lote`, salvar evidencia sanitizada e congelar golden file aprovado.

## Atualizacao complementar 2026-05-27 12:16 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Padronizado o Integra-AI para um unico leiaute Dominio oficial no fluxo principal: `dominio_separador_0000_0451` (`Dominio Sistemas com Separador - 0000/0451`).
- **Backend/API v2:** `integra-ai.engine.ts` centraliza a constante, normaliza configuracoes antigas, gera somente registros `0000`/`0451` e valida estrutura antes da exportacao. `UpdateIntegraAiSettingsDto` e `AccountingService` rejeitam `export_strategy` divergente.
- **Seguranca/auditoria:** exportacao/download continuam protegidos por permissoes especificas, passam a verificar escopo de empresa/departamento quando cadastrado, registram leiaute/status/hash em auditoria e retornam download privado com headers `no-store`.
- **Frontend React:** `/contabil/integra-ai` removeu o seletor de leiaute conflitante e exibe apenas o leiaute oficial na etapa de plano.
- **Testes:** adicionados testes unitarios para geracao `0000/0451`, rejeicao de legado/incompleto, campos obrigatorios e escopo; Playwright valida ausencia do seletor antigo e download autenticado.
- **Validacao:** Passaram testes focados da API, suite completa da API (`176` testes), lint da API/Web, build da API/Web, Prisma validate, Playwright focado do Integra-AI e Playwright local completo do Web.
- **Pendente:** Homologar manualmente um TXT real no Dominio Contabilidade Fiscal usando o importador de separador `0000/0451` e congelar golden file aprovado.


## Atualizacao complementar 2026-05-27 11:05 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Corrigido o download do TXT Dominio no Integra-AI React, que podia retornar `UNAUTHORIZED` porque o link direto para `/api-v2/accounting/integra-ai/jobs/:id/download` nao carregava o access token mantido em memoria.
- **Frontend React:** `portal-sama-web/src/services/integra-ai.service.ts` passou a baixar o TXT com `api.get(..., { responseType: 'blob' })`, reaproveitando o interceptor de `Authorization: Bearer`. `IntegraAiPage.tsx` trocou os anchors de download por botoes que disparam o blob autenticado, extraem filename do `Content-Disposition` quando disponivel e exibem erro controlado se o download falhar.
- **Layout Integra-AI:** A pagina recebeu classe `integra-ai-page` e ajuste CSS para alinhar o conteudo ao lado da sidebar compacta/expandida, evitando a percepcao de sobreposicao/deslocamento da marca/titulo da tela em viewport larga.
- **Testes Web:** O smoke Playwright do Integra-AI agora valida que a pagina inicia apos a sidebar com folga controlada e que o download gerado usa o cliente autenticado com `Bearer access-e2e`.
- **Estabilidade local:** `eslint.config.js` passou a ignorar `test-results` e `playwright-report`, evitando falha intermitente quando lint e Playwright disputam a pasta temporaria.
- **Validacao:** Passaram `npm.cmd run build`, `npm.cmd run test:e2e -- -g "Integra-AI"`, `npm.cmd run lint` e `git diff --check` no Web.
- **Pendente:** Publicar o novo build do Web e validar em homologacao real com usuario contabil, job real e TXT real; esta rodada usou API mockada no Playwright local.

## Atualizacao complementar 2026-05-27 10:40 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Corrigido reconhecimento de PDF do Banco Inter no parser do Integra-AI, incluindo tratamento correto de `Saldo do dia`.
- **Backend/parser:** `parse_statement.py` ganhou layout `inter_pdf`, com leitura de datas longas em portugues, transacoes agrupadas pela data corrente, descricoes quebradas em multiplas linhas e valores com `R$`/`-R$`.
- **Saldo diario:** Linhas `Saldo do dia` sao classificadas como `row_kind=balance_snapshot` e `ignore_reason=daily_balance`; servem para reconciliacao/diagnostico, mas nao sao persistidas como linhas contabeis pela API, nao aparecem para o usuario final e nao entram no TXT.
- **Validacao:** O PDF real do Banco Inter passou no parser com 31 transacoes e 15 saldos tecnicos. Passaram `python -m py_compile services/integra_ai_parser/parse_statement.py`, `npm.cmd test -- accounting.service.spec.ts --runInBand`, `npm.cmd run lint` e `npm.cmd run build` na API.
- **Pendente:** Validar outros PDFs reais de bancos diferentes e ampliar a matriz de fixtures/layouts para cobrir variacoes de PDF antes de desligar o legado.

## Atualizacao complementar 2026-05-27 10:03 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A analise semantica externa do Integra-AI foi conferida contra docs/codigo e implementada parcialmente: importacao OFX foi publicada como capability opt-in, sem alterar schema de banco e mantendo PDF como padrao.
- **Backend/API v2:** `IntegraAiParserService` passou a expor `parseFile({ ext })` para `pdf`/`ofx`; `AccountingService` agora valida e armazena extratos PDF/OFX, usa `source_type` dinamico, audita extensao/hash e expoe `pdf_import`/`ofx_import` no workspace. OFX permanece atras de `SAMA_INTEGRA_AI_OFX_IMPORT_ENABLED=false`.
- **Frontend React:** `/contabil/integra-ai` passou a ajustar texto, label e `accept` do upload conforme capabilities, aceitando `.ofx` somente quando `ofx_import=true`; o service manteve compatibilidade com a rota atual `/accounting/integra-ai/import`.
- **Configuracao/deploy:** `portal-sama-api/.env.example` e schema de env receberam `SAMA_INTEGRA_AI_OFX_IMPORT_ENABLED`; a documentacao de EasyPanel registra a flag e as variaveis do parser.
- **Validacao:** Passaram `npm.cmd test -- accounting.service.spec.ts --runInBand`, `npm.cmd run lint`, `npm.cmd run build` e `npm.cmd run prisma:validate` na API; no Web passaram `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e -- -g "Integra-AI"` e `npm.cmd run test:e2e`.
- **Pendente:** Habilitar OFX apenas em homologacao controlada e validar com arquivo real, usuario contabil, parser Python no container, MySQL/storage reais e ClamAV strict antes de considerar producao.

## Atualizacao complementar 2026-05-27 08:31 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A pendencia parcial de validacao local do Integra-AI foi reduzida com cobertura Playwright para busca textual de contas e autosave em regras paginadas.
- **Frontend React:** `portal-sama-web/src/pages/accounting/IntegraAiPage.tsx` agora expoe labels acessiveis nos campos customizados de contas do plano e de conta editavel das regras, preservando a UX atual e melhorando testabilidade/acessibilidade.
- **Testes Web:** `portal-sama-web/tests/e2e/smoke.spec.ts` ganhou mock operacional de Integra-AI e smoke em `/contabil/integra-ai` que abre job, navega para pagina 2 das regras, busca conta por nome (`merc` -> Mercado Livre), seleciona sugestao e confirma autosave; tambem valida salvamento por blur com codigo numerico em outra regra da pagina 2.
- **Estabilidade Playwright:** `portal-sama-web/playwright.config.ts` deixou de executar testes do mesmo arquivo em paralelo, porque a suite local com Vite/GSAP/mocks estava falhando por concorrencia de dev server/dynamic import no Windows. O comando padrao `npm.cmd run test:e2e` voltou a passar.
- **Validacao:** Passaram `npm.cmd run test:e2e -- -g "Integra-AI"`, `npm.cmd run test:e2e`, `npm.cmd run lint` e `npm.cmd run build` no Web. O Playwright real continuou skipped por ser opt-in.
- **Pendente:** Validar em homologacao real apos deploy, com job real e usuario contabil, confirmando busca por nome, autosave da conta ao selecionar sugestao/perder foco e persistencia contra MySQL real.

## Atualizacao complementar 2026-05-26 16:50 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Criado o plano operacional de rollback/restore drill e disponibilizado `npm run ops:restore:drill` no `portal-sama-api`.
- **Backend/operacao:** O novo script `scripts/restore-drill-operational-backup.js` valida artefatos gerados por `ops:backup:create`, bloqueia alvo igual a `DATABASE_URL`/`STORAGE_PRIVATE_PATH`, opera em dry-run por padrao e exige `--confirm RESTORE_DRILL_TARGET_IS_ISOLATED` para aplicar restore em banco/storage isolados.
- **Documentacao:** Criado `PLANO_ROLLBACK_RESTORE_DRILL.md` e atualizados README da API, deploy, seguranca, status, pendencias, changelog e prontidao de producao.
- **Validacao:** Passaram `node --check` do novo script e do readiness, help do comando, backup descartavel sem banco com archive de storage, dry-run do restore, restore de storage em alvo `.ai-tests`, `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run prisma:validate` e `git diff --check` na API.
- **Pendente:** Rodar backup real no EasyPanel, verificar artefatos, copiar para fora do container e executar `ops:restore:drill` com banco/storage isolados reais.

## Atualizacao complementar 2026-05-26 16:08 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Corrigido o filtro de contas do plano no Integra-AI para busca textual por nome/classificacao, incluindo Conta banco da etapa 3 e contas editaveis das regras contabeis, e ajustada a marca do menu lateral recolhido para nao sobrepor os icones.
- **Backend/API v2:** `AccountingService.searchPlanAccounts` deixou de tratar busca textual sem digitos como match de todos os codigos; agora consulta por nome como `merc` retorna somente contas realmente compativeis.
- **Frontend React:** `/contabil/integra-ai` passou a salvar regras contabeis automaticamente; categoria, historico e sem uso seguem com autosave debounced, enquanto a conta editavel salva somente quando o campo perde foco ou quando a sugestao e selecionada. O sidebar desktop recolhido agora usa marca compacta no topo.
- **Validacao:** Passaram `npm.cmd test -- accounting.service.spec.ts --runInBand`, `npm.cmd run lint` e `npm.cmd run build` na API; `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e -- -g "sidebar"` e `git diff --check` no Web/API.
- **Pendente:** Validar em navegador/homologacao real apos deploy, com job real e usuario contabil, confirmando busca por nome e salvamento da conta ao sair do foco na tabela paginada.

## Atualizacao complementar 2026-05-26 09:38 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Com o `.env` real da API alocado localmente e ignorado pelo Git, foi possivel executar a bateria real autenticada usando as variaveis de bootstrap admin apenas no processo do shell.
- **Evidencia real:** `smoke:auth` passou contra `https://portal.samacontabil.com.br/api-v2` com usuario bootstrap/DEV: CSRF, login, `/auth/me`, refresh, novo `/auth/me` e logout.
- **Permissoes reais:** `smoke:permissions` passou com anonimos 401 em `/auth/me`, `/users` e `/documents`, e bootstrap/DEV 200 em `/auth/me`, `/users`, `/roles` e `/permissions`.
- **Playwright real:** `npm.cmd run test:e2e:real` passou com usuario bootstrap/DEV, validando login pela UI, Home, politica de storage/cookie e logout.
- **Runner consolidado:** `npm.cmd run homologation:real -- --soft --evidence-dir .ai-tests/homologation-real` passou os quatro passos e gerou evidencia JSON sanitizada local.
- **Backend/operacao:** O `DATABASE_URL` do `.env` aponta para `portal-sama_portal-sama-database:3306`, hostname interno do EasyPanel; por isso `ops:backfill:report` e checks de banco do readiness nao conectam a partir do Windows local e devem rodar no container.
- **Hardening:** `ops:readiness` agora falha se `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET`, `CSRF_SECRET` ou `CERTIFICATE_ENCRYPTION_KEY` estiverem configurados com menos de 32 caracteres.
- **Pendente:** Rodar backfill/backup/restore no container real da API, ampliar matriz por CLIENT/MANAGER/departamentos/perfis operacionais, validar upload/download real e concluir QA visual/corte.

## Atualizacao complementar 2026-05-26 09:23 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Foi iniciada a execucao dos itens reais de homologacao com o ambiente disponivel e criado um runner consolidado para a bateria Web.
- **Frontend/operacao:** `portal-sama-web/package.json` agora expoe `npm.cmd run homologation:real`, chamando `scripts/portal-real-homologation.mjs`.
- **Cobertura do runner:** Executa `smoke:public`, `smoke:auth`, `smoke:permissions` e `test:e2e:real`; antes dos checks autenticados, valida somente a presenca das variaveis obrigatorias e pode gravar evidencia JSON sanitizada em `.ai-tests/homologation-real`.
- **Execucao real parcial:** `npm.cmd run smoke:public` passou contra o dominio publico. `smoke:permissions --soft --json` confirmou anonimos 401 em `/auth/me`, `/users` e `/documents`. `smoke:auth`, matriz autenticada e Playwright real ficaram bloqueados/skipped por ausencia de credenciais e matriz no shell local.
- **Backend/operacao:** `ops:readiness --soft --json` foi corrigido para retornar resumo JSON controlado quando `DATABASE_URL` nao existe, sem abortar a coleta por erro Prisma.
- **Pendente:** Carregar credenciais reais/matriz no shell, rodar `npm.cmd run homologation:real -- --evidence-dir .ai-tests/homologation-real`, executar backfill/backup/restore no container EasyPanel e anexar evidencias sanitizadas.

## Atualizacao complementar 2026-05-26 09:10 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** O repo `portal-sama-api` ganhou um verificador operacional de artefatos de backup para reduzir risco antes do restore drill real.
- **Backend/operacao:** `portal-sama-api/package.json` agora expoe `npm run ops:backup:verify`, chamando `scripts/verify-operational-backup.js`.
- **Cobertura do verificador:** O script valida `metadata.json`, status dos passos, hash/tamanho de artefatos, integridade gzip de `database.sql.gz`, consistencia de `storage-manifest.json` e listagem de `storage.tar.gz` quando existir.
- **Readiness:** O aviso `backup-rollback` agora orienta a sequencia `ops:backup:create -> ops:backup:verify -> copia externa -> restore drill`.
- **Validacao:** Passaram `node --check scripts/verify-operational-backup.js`, help do comando, backup local descartavel com storage archive, `ops:backup:verify --json` sobre o backup gerado, `node --check scripts/validate-operational-readiness.js`, readiness local com skips, `npm.cmd run build`, `npm.cmd run prisma:validate` com `DATABASE_URL` dummy e `git diff --check` em `portal-sama-api`.
- **Pendente:** Rodar `ops:backup:create` e `ops:backup:verify` no EasyPanel com banco/storage reais, copiar artefatos para fora do container e executar restore drill em alvo isolado. O verificador nao substitui restauracao real.

## Atualizacao complementar 2026-05-26 08:49 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** O repo `portal-sama-web` ganhou um smoke operacional de permissoes para montar e executar matriz 401/403/200 por perfil contra a API v2.
- **Frontend/operacao:** `portal-sama-web/package.json` agora expoe `npm.cmd run smoke:permissions`, chamando `scripts/portal-permissions-smoke.mjs`.
- **Cobertura do smoke:** O script valida anonimos contra `/auth/me`, `/users` e `/documents` esperando 401 e aceita uma matriz JSON de perfis com credenciais via `usernameEnv`/`passwordEnv`, checks por rota/metodo e status esperado 200/403/401.
- **Seguranca:** O script mantem cookies apenas em memoria, mascara usuarios e nao imprime senha, access token, refresh token ou CSRF token. O formato recomendado evita senhas no JSON usando variaveis de ambiente.
- **Validacao:** Passaram `node --check scripts/portal-permissions-smoke.mjs`, `npm.cmd run smoke:permissions -- --help`, smoke `--soft --json` contra porta fechada, smoke positivo contra servidor fake local com cenarios ADMIN 200, CLIENT 200/403 e anonimo 401, `npm.cmd run lint`, `npm.cmd run build` e `git diff --check` em `portal-sama-web`.
- **Pendente:** Rodar `npm.cmd run smoke:permissions` no EasyPanel com usuarios reais por perfil, sem registrar credenciais/tokens/cookies, e anexar a evidencia sanitizada da matriz.

## Atualizacao complementar 2026-05-25 17:54 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** O repo `portal-sama-web` ganhou uma suite Playwright opt-in para homologacao real de autenticacao via navegador.
- **Frontend/testes:** `portal-sama-web/package.json` agora expoe `npm.cmd run test:e2e:real`, usando `playwright.real.config.ts` sem web server local.
- **Cobertura real planejada:** A spec `tests/e2e/real-auth.spec.ts` acessa `/login` no dominio publico, faz login pela UI React, valida chegada na Home, checa ausencia de chaves sensiveis em `localStorage`/`sessionStorage`, confirma que o refresh cookie nao aparece em `document.cookie`, valida `HttpOnly`, `SameSite` e `Secure` quando HTTPS e encerra a sessao pelo botao `Sair`.
- **Seguranca operacional:** A suite so roda com `PORTAL_REAL_E2E=1` e credenciais por variaveis de ambiente. Trace, screenshot e video ficam desligados na config real para reduzir risco de vazar credenciais de homologacao.
- **Validacao:** Passaram `npm.cmd run test:e2e:real` sem variaveis com 1 teste skipped, `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e` com 9 testes passados e 1 skipped, e `git diff --check` em `portal-sama-web`.
- **Pendente:** Rodar `npm.cmd run test:e2e:real` contra `https://portal.samacontabil.com.br` com usuario real de homologacao, sem registrar credenciais/cookies/tokens, e repetir por perfis criticos.

## Atualizacao complementar 2026-05-25 17:37 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** O repo `portal-sama-web` ganhou um smoke operacional de autenticacao real para homologacao HTTPS.
- **Frontend/operacao:** `portal-sama-web/package.json` agora expoe `npm.cmd run smoke:auth`, chamando `scripts/portal-auth-smoke.mjs`.
- **Cobertura do smoke:** O script executa `GET /api-v2/auth/csrf`, `POST /api-v2/auth/login`, `GET /api-v2/auth/me`, `POST /api-v2/auth/refresh`, novo `GET /api-v2/auth/me` com token renovado e `POST /api-v2/auth/logout`.
- **Seguranca:** O script mantem cookies apenas em memoria, mascara o usuario no resumo e nao imprime senha, access token, refresh token ou CSRF token. Tambem valida que o refresh cookie de login/refresh tenha `HttpOnly`, `SameSite` e `Secure` quando a API estiver em HTTPS.
- **Validacao:** Passaram `node --check scripts/portal-auth-smoke.mjs`, `npm.cmd run smoke:auth -- --help`, smoke positivo contra servidor fake local, smoke `--soft` sem credenciais, `npm.cmd run lint` e `npm.cmd run build` em `portal-sama-web`.
- **Pendente:** Rodar `npm.cmd run smoke:auth` no EasyPanel com usuario real de homologacao, sem registrar credenciais, e anexar o resultado ao plano de validacao por perfil.

## Atualizacao complementar 2026-05-25 17:19 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** O fluxo publico de documentos de onboarding passou a expor tambem os aliases documentados em `/api-v2/public/onboarding/documents/:token`.
- **Backend/documentos:** `DocumentsController` agora mantem compatibilidade com `GET /api-v2/documents/public-checklist?token=...` e `POST /api-v2/documents/public-upload?token=...`, mas tambem expoe `GET /api-v2/public/onboarding/documents/:token` e `POST /api-v2/public/onboarding/documents/:token/upload`, usando o mesmo `DocumentsService`, `PublicToken`, throttle, validacao de token, scanner/storage e auditoria existentes.
- **Frontend publico:** `portal-sama-web/src/services/documents.service.ts` passou a consumir os aliases especificos de onboarding; a rota React continua sendo `/onboarding/publico/documentos/:token`, sem gravar token em storage do navegador.
- **Validacao:** Passaram `npm.cmd test -- documents.controller.spec.ts --runInBand`, `npm.cmd run lint`, `npm.cmd run build` e `prisma:validate` na API; no Web passaram `npm.cmd run lint`, `npm.cmd run build`, Playwright focado `npm.cmd run test:e2e -- -g "public documents page"` e `git diff --check`.
- **Pendente:** Validar esses aliases no EasyPanel com token real/legado, MySQL/storage/ClamAV reais e executar Playwright de upload publico real em homologacao.

## Atualizacao complementar 2026-05-25 16:54 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Readiness real passou com ClamAV strict/EICAR e a pendencia de backup/rollback ganhou ferramenta operacional executavel.
- **Evidencia real:** O operador repetiu o readiness no console do `portal-sama-api`; passaram ambiente, banco, 19 migrations Prisma, RBAC, usuarios privilegiados, storage em `/var/private/portal-sama` e `clamav-eicar` com `/usr/bin/clamscan`.
- **Backend/operacao:** `portal-sama-api` agora expoe `npm run ops:backup:create`, gerando dump MySQL compactado, manifesto de storage, metadados com SHA-256 e arquivo de storage opcional.
- **Docker/runtime:** O runtime da API instala `mariadb-client` alem de `clamav`, deixando `mariadb-dump` disponivel dentro da imagem.
- **Validacao:** Passaram `node --check`, help do comando de backup, backup local sem banco com manifesto/archive, `prisma:validate`, lint, build, Jest completo com 165 testes, Docker build e checagem de `/usr/bin/mariadb-dump` dentro da imagem.
- **Pendente:** Rodar `ops:backfill:report -- --json` e `ops:backup:create` no EasyPanel, copiar artefatos para fora do container, provar restore drill, validar matriz real de perfis, Playwright real e QA final.

## Atualizacao complementar 2026-05-25 16:31 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Corrigido o gap real encontrado no `ops:readiness` executado no console do `portal-sama-api`: ClamAV nao estava instalado na imagem anterior.
- **Evidencia real:** O readiness do operador passou em environment, database, migrations, RBAC, privileged-users e storage; falhou somente em `clamav-eicar` por ausencia de scanner em modo `strict`.
- **Backend/Docker:** `portal-sama-api/Dockerfile` agora instala `clamav`, cria diretorios padrao do ClamAV e define `SAMA_UPLOAD_SCAN_BIN=/usr/bin/clamscan`.
- **Operacao:** Adicionado `npm run ops:clamav:update` para executar `freshclam` no container antes do readiness final. `SAMA_CLAMAV_UPDATE_ON_START=false` fica documentado como default, com opcao de habilitar update no boot conscientemente.
- **Validacao:** Passaram `node --check`, readiness local com skips, lint, build, Docker build `portal-sama-api:clamav-runtime`, checagem de `clamscan/freshclam/clamdscan` dentro da imagem e readiness Docker com falha esperada apenas por assinaturas ainda ausentes.
- **Pendente:** Redeployar a API, rodar `npm run ops:clamav:update` no console do container e repetir `npm run ops:readiness` sem skips.

## Atualizacao complementar 2026-05-25 16:18 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Adicionadas validacoes operacionais de API para readiness, RBAC/perfis, storage/ClamAV e relatorio read-only de backfill, alem de corrigir uma fragilidade temporal na suite de transferencias.
- **Backend/operacao:** `portal-sama-api/package.json` agora expoe `npm run ops:readiness` e `npm run ops:backfill:report`.
- **Readiness:** O comando valida ambiente de producao, MySQL, migrations Prisma aplicadas, seed RBAC contra catalogo padrao, usuarios ADMIN/DEV ativos, storage privado e ClamAV/EICAR. A execucao local foi feita com skips para banco/ClamAV; a execucao real sem skips continua pendente no EasyPanel.
- **Backfill:** O relatorio read-only lista tabelas atuais, candidatos legados, contagens por modulo e gaps de relacao sem alterar dados, servindo como evidencia antes de qualquer backfill real.
- **Transferencias:** `transfers.service.spec.ts` passou a congelar o relogio em `2026-05-19T12:00:00.000Z`, removendo flakiness causada pela data atual.
- **Validacao:** Passaram `node --check` dos scripts, smokes locais dos comandos operacionais, `npm.cmd run test -- --runInBand` com 165 testes, `npm.cmd run lint`, `npm.cmd run build` e `npm.cmd run prisma:validate` com `DATABASE_URL` dummy.
- **Pendente:** Rodar `ops:readiness` sem skips e `ops:backfill:report -- --json` no container real da API, anexar evidencias, executar backfills, backup/restore drill, validar matriz real de perfis e concluir Playwright/QA.

## Atualizacao complementar 2026-05-25 11:46 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Documentacao atualizada para refletir o estado real informado e validado: Bitbucket, EasyPanel, dominio publico e banco ja estao operacionais.
- **Bitbucket:** Repositorios existentes conforme evidencia visual: `portal-sama-api`, `portal-sama-web` e `portal-sama-doc` (alias remoto observado para a documentacao local `portal-sama-docs`).
- **EasyPanel:** Projeto `portal-sama` contem `portal-sama-api`, `portal-sama-database` e `portal-sama-web` online. Logs da API mostram NestJS iniciado; logs do Web mostram assets e chamadas `/api-v2/*`; logs do banco mostram MySQL Community Server 9.7.0 pronto na porta 3306.
- **Dominio publico:** `https://portal.samacontabil.com.br/` esta ativo. `npm.cmd run smoke:public` passou sem `--soft`: frontend HTTP 200, health com `database=up` e `storage=up`, preflight CORS 204 e CSRF 200 com token/cookie.
- **Seguranca HTTP:** `curl.exe` em `/api-v2/auth/csrf` confirmou `Access-Control-Allow-Origin: https://portal.samacontabil.com.br`, `Access-Control-Allow-Credentials: true`, HSTS, CSP e cookie CSRF com `Secure; SameSite=Lax`.
- **Pendente:** Registrar/confirmar migrations, seed/bootstrap ou procedimento equivalente de usuarios iniciais, matriz de permissoes por perfil, backups/rollback, storage persistente, ClamAV strict, Playwright real e QA visual final.

## Atualizacao complementar 2026-05-25 11:32 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** `GET /api-v2/health` deixou de ser apenas declarativo e passou a validar banco/storage quando aplicavel.
- **Backend/health:** `HealthService` agora retorna `database=up/down/not_checked` e `storage=up/down/not_configured`; em caso de falha, o controller responde com HTTP 503 mantendo o corpo de status.
- **Banco:** Quando `PRISMA_CONNECT_ON_BOOT=true`, o health executa `SELECT 1` via Prisma. Quando `PRISMA_CONNECT_ON_BOOT=false`, preserva `database=not_checked` para testes e diagnostico controlado.
- **Storage:** O health cria/verifica o caminho de `STORAGE_PRIVATE_PATH` e confirma permissao de leitura/escrita.
- **Validacao:** Passaram `npm.cmd test -- health.service.spec.ts --runInBand`, `npm.cmd run test:e2e -- --runInBand` com 136 testes, `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run prisma:validate` e `git diff --check` em `portal-sama-api`.
- **Pendente:** Validar o health real no EasyPanel com `DATABASE_URL` do `banco-sama`, volume persistente e HTTPS.

## Atualizacao complementar 2026-05-25 11:20 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** O smoke publico documentado para deploy foi implementado no repo separado `portal-sama-web`.
- **Frontend/deploy:** `portal-sama-web/package.json` agora expoe `npm.cmd run smoke:public`, chamando `scripts/portal-public-smoke.mjs`.
- **Smoke publico:** O script valida a raiz publica do frontend, `GET /api-v2/health`, preflight CORS de `/api-v2/auth/me` e `GET /api-v2/auth/csrf` com token e cookie.
- **Documentacao:** `portal-sama-web/README.md` e `EASYPANEL_DEPLOY.md` agora indicam que o smoke deve ser executado a partir do repo separado do Web.
- **Validacao:** Passaram `node --check scripts/portal-public-smoke.mjs`, smoke local com servidor HTTP fake, smoke `--soft` contra porta fechada, `npm.cmd run lint`, `npm.cmd run build` e `git diff --check` em `portal-sama-web`.
- **Pendente:** Rodar `npm.cmd run smoke:public` sem `--soft` contra o dominio real apos publicar API/Web no EasyPanel.

## Atualizacao complementar 2026-05-25 11:09 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Adicionada instrucao minima de continuidade nos repos separados de API e Web.
- **Backend/repo:** `portal-sama-api/README.md` foi criado com a regra de ler `portal-sama-docs` antes de alterar codigo, deploy, banco, seguranca ou testes, alem dos comandos principais de API/Prisma.
- **Frontend/repo:** `portal-sama-web/README.md` foi reforcado com a mesma regra operacional, comandos incluindo Playwright e nota de deploy do proxy `/api-v2`.
- **Validacao:** Passaram `git diff --check` em `portal-sama-api`, `portal-sama-web` e `portal-sama-docs`. Lint/build nao foram executados porque a rodada alterou apenas READMEs e documentacao.
- **Pendente:** Criar/publicar remotes Bitbucket e validar EasyPanel/API/MySQL/HTTPS reais.

## Atualizacao complementar 2026-05-25 10:51 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A cobertura E2E do `portal-sama-web` foi ampliada para proteger o login mobile e a intro `welcome` do primeiro login.
- **Frontend/testes:** `portal-sama-web/tests/e2e/smoke.spec.ts` agora possui mock de login com usuario `welcomeAnimationSeen=false`, simulando o fluxo real `csrf -> login -> /home -> IntroGate`.
- **Login mobile:** Adicionado smoke que valida a lateral visual do login em `390x844`, sem overflow horizontal, com logo dentro da area de boas-vindas, texto sem corte e painel dentro do viewport.
- **Welcome primeiro login:** Adicionado smoke que confirma a cena `welcome` apos login, logo centralizada, texto dentro do viewport e mascaras de fade nas linhas laterais.
- **Validacao:** Passaram `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e` com 9 testes Chromium e `git diff --check` em `portal-sama-web`. O aviso conhecido `No HydrateFallback element provided...` continuou aparecendo no Vite durante o E2E, sem falha funcional.
- **Pendente:** Playwright contra homologacao real com API/MySQL/HTTPS reais e QA visual final no EasyPanel continuam pendentes.

## Atualizacao complementar 2026-05-25 10:33 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Corrigido desalinhamento visual da intro/welcome observado no deploy e reforcado o fade das linhas laterais.
- **Frontend/intro:** `portal-sama-web` nao depende mais da propriedade CSS individual `translate` para centralizar logo/texto da intro. A ancoragem animada agora e preservada pelo GSAP via `data-intro-x-percent`/`data-intro-y-percent`, evitando deslocamento para baixo/direita em navegadores/caches que nao aplicam `translate`.
- **Login:** A animacao da lateral do login tambem deixou de depender de `translate`; os offsets foram incorporados nos keyframes com `translateY(...)`.
- **Welcome:** As linhas laterais da cena `welcome` receberam mascaras CSS adicionais para esmaecerem antes do centro, e o texto ganhou ajuste mobile para nao cortar dentro do preview/viewport estreito.
- **Validacao:** Passaram `npm.cmd run lint`, `npm.cmd run build`, `git diff --check`, smoke Playwright manual em `/dev/intro-preview` e `/login` com viewports `1920x900`, `390x844` e checagem extra `320x720`. `rg "\btranslate\s*:" src public -S` nao encontrou usos restantes.

## Atualizacao complementar 2026-05-25 09:48 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Corrigido o diagnostico do deploy com Prisma `P1013` (`invalid port number in database URL`).
- **Backend/config:** `portal-sama-api/src/config/env.schema.ts` agora valida `DATABASE_URL` antes do Prisma inicializar e retorna mensagem mais direta para URL MySQL malformada, porta duplicada ou senha com caracteres especiais sem URL encode.
- **EasyPanel:** Documentado que o valor deve seguir `mysql://usuario:SENHA_URL_ENCODED@host:3306/banco`, usando o host interno mostrado pelo EasyPanel e sem repetir a porta. O erro nao indica problema nas migrations nem necessidade de recriar o banco.
- **Validacao:** `npm.cmd test -- env.schema.spec.ts --runInBand`, `npm.cmd run lint`, `npm.cmd run build` e `npm.cmd run prisma:validate` passaram em `portal-sama-api`.
- **Pendente:** Ajustar a variavel `DATABASE_URL` real no servico `portal-sama-api`, redeployar e entao rodar `npm run prisma:seed`, `npm run prisma:bootstrap-admin`, health, csrf e login real.

## Atualizacao complementar 2026-05-25 09:25 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Corrigida a execucao de `npm run prisma:seed` dentro da imagem Docker da API.
- **Backend/deploy:** `portal-sama-api/package.json` agora chama `scripts/run-prisma-runtime-script.js` para `prisma:seed` e `prisma:bootstrap-admin`. O wrapper executa `dist/prisma/*.js` quando o build compilado existe no container e usa `prisma/*.ts` apenas em ambiente local sem `dist`.
- **Causa raiz:** A imagem de runtime copia `dist/`, `prisma/` e `scripts/`, mas nao copia `src/`; por isso `ts-node prisma/seed.ts` nao encontrava `../src/modules/rbac/default-rbac`.
- **Validacao:** `node --check scripts/run-prisma-runtime-script.js`, `npm.cmd run build`, `npm.cmd run lint`, `npm.cmd run prisma:validate` passaram. `npm.cmd run prisma:seed` com `DATABASE_URL` apontando para `127.0.0.1:9` falhou apenas por conexao recusada ao banco, confirmando que o script compilado foi usado.
- **Observacao:** Docker Desktop local nao estava ativo em 2026-05-25 09:25, entao o rebuild Docker local nao foi repetido nesta rodada. A validacao com `dist/prisma/seed.js` cobriu o erro reportado no EasyPanel.

## Atualizacao complementar 2026-05-22 16:38 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Corrigido o novo erro de deploy Prisma `P3005` em banco EasyPanel existente sem historico `_prisma_migrations`.
- **Backend/deploy:** `portal-sama-api/Dockerfile` nao executa mais `prisma migrate deploy` automaticamente no start; `PRISMA_MIGRATE_ON_START=false` e o container deve ficar online para abrir console/one-off command. O entrypoint tambem foi alinhado ao build real em `dist/src/main.js`.
- **Banco/Prisma:** Adicionada migration vazia `20260501000000_baseline_existing_database` e script `npm run prisma:migrate:baseline-existing`, protegido por `SAMA_PRISMA_BASELINE_EXISTING_DATABASE=true`, para marcar baseline e aplicar migrations reais em banco existente apos backup.
- **Documentacao:** Atualizados os procedimentos de EasyPanel, banco e painel de deploy para diferenciar banco limpo (`prisma migrate deploy`) de banco existente com `P3005` (baseline controlado).
- **Validacao:** `npm.cmd run prisma:generate`, `npm.cmd run prisma:validate`, `npm.cmd run build`, `npm.cmd run lint`, `node --check scripts/prisma-baseline-existing-database.js`, `npm.cmd run prisma:migrate:baseline-existing` sem confirmacao (falha esperada), `docker build --pull=false -t portal-sama-api:prisma-p3005-fix .`, `docker run --rm portal-sama-api:prisma-p3005-fix` sem env (falha esperada), smoke Docker com env minima e `PRISMA_CONNECT_ON_BOOT=false` e `git diff --check` passaram/produziram os resultados esperados.
- **Pendente:** Rebuild/redeploy real no EasyPanel, backup do `banco-sama`, execucao unica do baseline controlado no console da API, seed, bootstrap admin, health, csrf e login real.

## Atualizacao complementar 2026-05-22 16:08 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Corrigido o caminho de build/deploy da API para o erro Prisma `P1012` por ausencia de `DATABASE_URL`.
- **Backend/deploy:** `portal-sama-api/Dockerfile` agora passa `PRISMA_GENERATE_DATABASE_URL` dummy somente ao `prisma generate` no build e valida explicitamente `DATABASE_URL` no runtime antes de `prisma migrate deploy`.
- **EasyPanel:** Documentado que a `DATABASE_URL` real deve ficar nas variaveis do servico `portal-sama-api`, apontando para `portal-sama_database:3306/banco-sama` com usuario nao-root.
- **Validacao:** `npm.cmd run prisma:generate`, `npm.cmd run build`, `npm.cmd run prisma:validate` e `docker build --pull=false -t portal-sama-api:prisma-env-fix .` passaram em `portal-sama-api`; `docker run --rm portal-sama-api:prisma-env-fix` sem env falhou como esperado com mensagem explicita de `DATABASE_URL`; `git diff --check` passou no repo de docs e apontou apenas avisos LF/CRLF no repo da API por causa do Windows.
- **Pendente:** Rebuild/redeploy real no EasyPanel com `DATABASE_URL` configurada, depois executar/validar migrations, seed, bootstrap admin, health, csrf e login real.

## Atualizacao complementar 2026-05-22 15:22 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Validados lint/build nos workspaces tecnicos ja separados.
- **Backend:** Passaram `npm.cmd run lint`, `npm.cmd run build` e `npm.cmd run prisma:validate` em `portal-sama-api`.
- **Frontend:** Passaram `npm.cmd run lint` e `npm.cmd run build` em `portal-sama-web`.
- **Observacao:** `prisma validate` carregou `.env` automaticamente, mas nenhum valor de ambiente foi lido ou exibido.
- **Andamento:** Estimativa operacional ajustada para 76% de prontidao para iniciar homologacao e 64% para producao sem legado.
- **Pendente:** Remotes/push Bitbucket, EasyPanel real, migrations/seed/bootstrap em MySQL real, E2E/Playwright contra fluxos reais e QA visual.

## Atualizacao complementar 2026-05-22 15:16 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Preparados os tres workspaces separados para commits iniciais em `main`.
- **Git local:** `portal-sama-docs`, `portal-sama-api` e `portal-sama-web` estao com Git local ativo e branch `main`.
- **Normalizacao:** Criado `.gitattributes` nos tres repositorios para manter textos em LF e tratar imagens/arquivos binarios como binarios.
- **Protecao de versionamento:** Confirmado que `.env`, `node_modules`, `dist`, `playwright-report` e `test-results` nao entram no stage.
- **Validacao:** `git diff --cached --check` passou nos tres workspaces apos limpeza de linhas em branco extras no fim de arquivos herdados.
- **Andamento:** Estimativa operacional ajustada para 74% de prontidao para iniciar homologacao e 63% para producao sem legado.
- **Pendente:** Criar repositorios Bitbucket, adicionar remotes e fazer push dos commits iniciais.

## Atualizacao complementar 2026-05-22 15:10 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Confirmada a separacao local dos workspaces `portal-sama-docs`, `portal-sama-api` e `portal-sama-web`.
- **Workspaces:** As tres pastas existem como irmas em `C:\Users\Sama Contabilidade\Desktop\`.
- **Git local:** Git foi inicializado nos tres workspaces; ainda faltam repositorios/remotes Bitbucket.
- **Protecao de versionamento:** Criados `.gitignore` em `portal-sama-api` e `portal-sama-docs`; `.gitignore` do `portal-sama-web` foi reforcado para ignorar `.env`, artefatos e testes locais.
- **Documentacao:** `portal-sama-docs` agora possui `README.md` inicial indicando que a documentacao deve ser lida antes de alterar API/Web.
- **Validacao:** Checados os status Git com ignored files; `.env`, `node_modules`, `dist`, `playwright-report` e `test-results` ficaram ignorados onde aplicavel.

## Atualizacao complementar 2026-05-22 14:57 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Formalizada a topologia oficial de tres repositorios: `portal-sama-docs`, `portal-sama-api` e `portal-sama-web`.
- **Documentacao:** `portal-sama-docs` passa a ser a fonte de verdade obrigatoria antes de qualquer alteracao nos workspaces de API ou Web.
- **Deploy/Bitbucket:** `docs/AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`, `docs/EASYPANEL_DEPLOY.md`, `docs/README_DOCUMENTACAO.md` e `docs/ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md` agora descrevem a separacao em tres repositorios.
- **Arquitetura:** Registrada ADR-0030 para documentacao centralizada em repositorio proprio.
- **Validacao:** `git diff --check` passou, com avisos LF/CRLF do Git no Windows. Testes automatizados nao foram executados porque a rodada alterou apenas documentacao.

## Atualizacao complementar 2026-05-22 14:18 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** O documento `docs/AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD` foi promovido a painel obrigatorio de prontidao para homologacao/producao.
- **Deploy/Bitbucket:** O documento agora descreve a separacao dos workspaces `portal-sama-api` e `portal-sama-web` em repositorios Bitbucket, a configuracao minima no EasyPanel, o uso do banco real `banco-sama` e a ordem segura de homologacao.
- **Andamento:** Registrada estimativa operacional atual de 72% para iniciar homologacao da nova stack e 62% para producao sem legado, com bloqueios restantes explicitados.
- **Documentacao obrigatoria:** `docs/INSTRUÇÕEDS_AI.MD`, `docs/README_DOCUMENTACAO.md` e `docs/EASYPANEL_DEPLOY.md` agora apontam para esse documento como leitura/atualizacao obrigatoria em ciclos de deploy, progresso e producao.
- **Validacao:** `git diff --check` passou, com avisos LF/CRLF do Git no Windows. Testes automatizados nao foram executados porque a rodada alterou apenas documentacao.

## Atualizacao complementar 2026-05-22 14:05 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A API ganhou um bootstrap administrativo explicito para homologacao em banco limpo do EasyPanel, sem depender da aplicacao antiga para o primeiro login.
- **Backend:** `prisma:seed` continua populando apenas permissoes/roles; o novo `npm run prisma:bootstrap-admin` cria ou atualiza o primeiro usuario `ADMIN`/`DEV` a partir de variaveis `SAMA_BOOTSTRAP_ADMIN_*`, preservando senha existente salvo reset explicito.
- **Deploy/EasyPanel:** Documentado o caso real com dois repositorios (`portal-sama-api` e `portal-sama-web`) e banco existente `banco-sama` no host interno `portal-sama_database:3306`.
- **Validacao:** Passaram `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run prisma:validate` em `portal-sama-api` e `git diff --check`.
- **Observacao:** O comando desbloqueia login inicial na nova stack, mas a independencia total do legado ainda depende de migrations/backfills em MySQL real, storage/ClamAV, usuarios/permissoes reais, HTTPS e QA/Playwright com fluxos reais.

## Atualizacao complementar 2026-05-22 10:02 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A cobertura Playwright do `portal-sama-web` foi ampliada para a central interna `/documentos`.
- **Documentos:** A suite E2E agora valida a rota autenticada `/documentos` com API mockada, incluindo listagem de documento privado, cliente relacionado, extensoes permitidas, links publicos, resumo de pendencias obrigatorias e abertura do fluxo de revisao.
- **Validacao:** Passaram `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e` com 7 testes Chromium e `git diff --check`.
- **Observacao:** O Playwright continua exibindo o aviso conhecido do React Router sobre `HydrateFallback`; nao houve falha funcional associada na execucao final.

## Atualizacao complementar 2026-05-22 09:39 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A cobertura Playwright da Home/sidebar foi ampliada para viewport mobile.
- **Home mobile:** A suite E2E agora valida `/home` autenticado em `390x844`, mantendo titulo, atalhos principais e conteudo principal visiveis.
- **Sidebar mobile:** O teste confirma que o menu nao fica no estado compacto de `96px`, exibe labels de navegacao, usa uma coluna em telas pequenas e nao gera overflow horizontal.
- **Validacao:** Passaram `npm.cmd run lint`, `npm.cmd run build` e `npm.cmd run test:e2e` com 6 testes Chromium.
- **Observacao:** O Playwright continua exibindo o aviso conhecido do React Router sobre `HydrateFallback`; nao houve falha funcional associada nesta rodada.

## Atualizacao complementar 2026-05-22 09:28 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A cobertura Playwright do `portal-sama-web` foi ampliada para a Home e para o comportamento compacto/expandido da sidebar.
- **Home:** A suite E2E agora valida `/home` autenticado, KPIs de sessao, badge `Sessao ativa` e atalhos renderizados conforme permissoes mockadas.
- **Sidebar:** A suite E2E agora valida largura fechada `96px`, expansao por hover para `304px`, fechamento apos clique de mouse em `Inicio` e expansao por foco de teclado.
- **Validacao:** Passaram `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e` com 5 testes e `git diff --check`. Houve falha intermediaria corrigida nos seletores novos e uma corrida conhecida ao rodar lint em paralelo com Playwright; o lint isolado passou.

## Atualizacao complementar 2026-05-22 09:07 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Corrigido o bug em que a sidebar ficava aberta apos clicar em um link do menu.
- **Menu:** A regra de expansao por foco agora usa `:focus-visible` em vez de `:focus-within`, evitando que um clique de mouse mantenha a sidebar aberta.
- **Interacao:** Depois de clicar em um item do menu e tirar o mouse, a sidebar volta para `96px` automaticamente.
- **Acessibilidade:** A expansao por teclado foi preservada; ao navegar com `Tab`, o menu abre enquanto o foco visivel esta em um item.
- **Validacao:** Passaram lint, build, smoke `sidebar-click-mouseleave-smoke`, smoke `sidebar-keyboard-focus-smoke` e E2E.

## Atualizacao complementar 2026-05-22 08:54 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A sidebar autenticada agora fica fechada por padrao e abre automaticamente ao passar o mouse ou focar links pelo teclado.
- **Menu:** Removido o botao `Recolher menu`; o estado de colapso manual deixou de existir no React.
- **Logo:** A logo Sama permanece visivel no menu fechado, centralizada dentro do container compacto.
- **Responsividade:** No desktop a largura fechada e `96px` e no hover expande para `304px`; em telas ate `920px`, o menu segue expandido para facilitar toque/leitura.
- **Validacao:** Passaram `npm.cmd run lint`, `npm.cmd run build`, smoke Playwright `sidebar-hover-smoke` e `npm.cmd run test:e2e`.

## Atualizacao complementar 2026-05-22 08:42 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A pagina de login recebeu uma lateral visual alinhada a intro, com logo Sama animada e texto manuscrito.
- **Login:** Removido o titulo `Portal Interno`; no lugar entrou uma vitrine com logo em camadas, halo, glows e orbitas usando os assets locais da intro.
- **Texto/manuscrito:** Adicionado `Bem vindo ao Portal Sama` com efeito de escrita progressiva e destaque em azul/laranja para `Portal` e `Sama`.
- **Responsividade/acessibilidade:** O bloco foi ajustado para desktop/mobile sem cortar o texto, e `prefers-reduced-motion` exibe o conteudo completo sem animacoes.
- **Validacao:** Passaram `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e` e smoke Playwright `login-showcase-smoke` em desktop/mobile. Screenshot de `/login` foi conferido visualmente.

## Atualizacao complementar 2026-05-21 16:48 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Corrigido o corte final do texto manuscrito da welcome; agora a frase renderizada e validada e `Seja bem vindo ao Portal Sama`.
- **Texto:** A camada de texto da welcome ficou mais larga e ganhou respiro lateral para evitar clipping do `Sama`. O hifen de `bem-vindo` foi removido conforme pedido.
- **Linhas/SVG:** Os SVGs laterais receberam mascara de fade para que os tracos fiquem mais fortes nas bordas e esmaecam/sumam antes de chegar ao centro.
- **Logo/e efeitos:** Halo, glows e orbitas foram reposicionados para alinhar ao centro visual real da logo, medido no PNG como cerca de `+7.7%` no eixo Y em relacao ao centro do canvas.
- **Validacao:** Passaram validacao JSON, checagem estatica de SVGs, smoke Playwright `welcome-text-enter-smoke`, `npm.cmd run build`, `npm.cmd run test:e2e`, `npm.cmd run lint` reexecutado isoladamente e `git diff --check`.

## Atualizacao complementar 2026-05-21 16:32 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A intro de boas-vindas agora permanece na tela ate o usuario pressionar `Enter`.
- **Interacao:** A cena `welcome` nao possui mais timeout automatico nem botao de pular; o listener de teclado global finaliza a intro somente com `Enter`. Se o usuario ficar mais de 7s na tela, aparece o aviso `Pressione Enter para continuar`.
- **Linhas/SVG:** As camadas SVG laterais foram refeitas com multiplas curvas finas, opacidades menores, glow suave e particulas pequenas, aproximando o estilo da referencia enviada. As ondas WebP laterais foram reduzidas para atuar apenas como brilho de base.
- **Validacao:** Passaram validacao JSON da intro, checagem de tokens proibidos nos SVGs, `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e` e smoke Playwright inline `welcome-enter-required-smoke`.

## Atualizacao complementar 2026-05-21 16:12 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A intro foi prolongada conforme ajuste visual: cena normal em `2.7s`, boas-vindas em `4.5s` e encerramento com fade mais longo.
- **Texto/manuscrito:** A mensagem de boas-vindas agora usa tipografia manuscrita com revelacao progressiva, simulando escrita manual, mantendo destaque de cor em `Portal` e `Sama`.
- **Linhas/SVG:** Adicionadas camadas SVG laterais `side-lines-left.svg` e `side-lines-right.svg`, com glow e animacao de desenho. As ondas da cena welcome tambem foram contidas nas laterais para nao se encontrarem no centro, ficando limitadas a cerca de 30-35% da tela.
- **Validacao:** Passaram validacao JSON da intro, checagem de tokens proibidos nos SVGs novos, `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e`, smoke Playwright inline de intro login-only e `git diff --check`.

## Atualizacao complementar 2026-05-21 15:46 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Refinada a experiencia visual pos-login do `portal-sama-web`: intro mais fluida, sem flicker de logo, com duracao em torno de 1.7s para entrada normal e sem reaparecer em reload/refresh de sessao.
- **Intro/autenticacao:** `IntroGate` agora depende da origem da sessao `login`; refresh/snapshot carregam direto a aplicacao. A conclusao da intro limpa a origem da sessao e usa handler estavel para evitar reinicios de timeline. `PortalSamaIntro` ganhou fade de saida e timings reduzidos.
- **GSAP/CSS:** A centralizacao do logo/texto passou a usar `translate` separado do `transform`, evitando conflito com `scale`, `rotation`, `x` e `y` do GSAP. O mapa de animacao foi retimado com easing mais moderno, menor blur no logo e loops mais sutis.
- **Sidebar/Home:** A sidebar passou a usar o PNG sem fundo `sama-logo-contabilidade-sem-fundo.png` dentro de container centralizado e destacado. Os SVGs decorativos do dashboard/sidebar/KPIs foram trocados por linhas finas e discretas, removendo blocos grossos e riscos soltos.
- **Validacao:** Passaram `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e`, smoke Playwright inline de intro login-only e `git diff --check` em `portal-sama-web`/repo. O E2E passou 3 testes Chromium; ficou apenas o aviso conhecido de `HydrateFallback`.
- **Pendencias:** QA visual manual em navegador real desktop/mobile e homologacao HTTPS com usuario real/permissoes reais.

## Atualizacao complementar 2026-05-21 14:24 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Corrigido o erro apos login em desenvolvimento: `error loading dynamically imported module: /node_modules/.vite/deps/gsap.js?...`.
- **Causa tratada:** O import estatico de `gsap`/`@gsap/react` dentro do hook da intro entrava no chunk do layout autenticado. Quando o Vite ficava com cache otimizado antigo ou servidor dev stale, a falha de carregar `gsap.js` derrubava a rota antes do fallback da intro existir.
- **Frontend React:** `usePortalSamaGsapTimeline` agora carrega o runtime GSAP via import dinamico dentro do efeito, usa `gsap.context` escopado e captura falhas para acionar fallback estatico. `RouteErrorPage` foi adicionada ao router para substituir a tela crua de erro do React Router em falhas de rota futuras.
- **Cache/dev server:** Servidores Vite antigos deste workspace foram encerrados, `portal-sama-web/node_modules/.vite` foi removido com caminho absoluto verificado, e o Vite foi reiniciado em `http://127.0.0.1:5173/` com `--force`.
- **Documentacao/design:** Foram lidos os documentos centrais e todos os arquivos textuais de `docs/design` e `docs/design/intro-gsap-V2`; tambem foi inventariada a pasta `docs/design` com assets, previews e referencias.
- **Validacao:** Passaram `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e`, smoke Playwright de `/home` em `127.0.0.1:5173`, HTTP 200 para `manifest.json`, `animation-map.json` e `/node_modules/.vite/deps/gsap.js`, alem de `git diff --check`.
- **Evidencias:** `.ai-tests/runs/2026-05-21-1424-gsap-import-guard/`.

## Atualizacao complementar 2026-05-21 14:05 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A intro oficial do `portal-sama-web` foi migrada para GSAP v2 em `src/features/intro`, consumindo `manifest.json`, `animation-map.json` e assets locais finais em `/brand/sama/intro/assets`.
- **Frontend React:** `PortalSamaIntro`, `PortalSamaSceneRenderer`, `PortalSamaLayer`, `PortalSamaStaticFallback`, `IntroGate` e `/dev/intro-preview` foram refatorados para a engine GSAP/`@gsap/react`, com cenas `loading` e `welcome` e modos `auto`, `animated` e `static`.
- **Assets/legado:** Assets finais foram publicados em `portal-sama-web/public/brand/sama/intro/assets/{shared,logo,loading,welcome}`; `layer-manifest.json`, assets SVG antigos e componentes visuais Framer/Motion foram removidos da experiencia ativa.
- **Seguranca/fallback:** Implementada validacao local de paths de assets, bloqueando remoto, `data:`, `javascript:`, `../`, `http:`, `https:` e `//`; reduced motion, falha de manifesto/mapa, falha de GSAP ou asset obrigatorio caem para fallback estatico sem bloquear login, sessao ou navegacao.
- **Integracao preservada:** `IntroGate`, `welcomeAnimationSeen`, `markWelcomeIntroSeen`, CSRF, auditoria via `PATCH /api-v2/me/preferences/intro` e integracao com `AppLayout` foram preservados. Rotas publicas continuam sem intro autenticada.
- **Layout:** Reforcado contraste da sidebar para manter labels e botao de recolher em branco no fundo escuro.
- **Validacao:** Passaram `npm.cmd run lint` e `npm.cmd run build` em `portal-sama-web`; SVGs orbitais finais foram checados sem ocorrencias de `<script>`, `foreignObject`, handlers `on*`, href externo, `data:` ou `javascript:`.
- **Pendencias:** Validar visualmente em navegador/homologacao desktop/mobile, testar a rota `/dev/intro-preview` manualmente e ampliar Playwright visual quando o fluxo de design estiver aprovado.

## Atualizacao complementar 2026-05-21 10:57 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Implementadas as fases iniciais do padrao visual documentado em `docs/design/codex_estilizacao_padrao_portal_sama.md` no `portal-sama-web`, sem reestilizar o legado e sem alterar regras de negocio.
- **Frontend React:** `AppLayout` ganhou suporte a sidebar recolhida; `Sidebar` foi convertida para o padrao premium escuro com links reais, `PermissionGate` preservado e botao de recolher; `Header` recebeu topbar glass leve; foram criados `PageShell`, `AppPanel`, `IconBadge`, `KpiCard` e `ShortcutCard`.
- **Home/Dashboard:** `/home` passou a usar os novos componentes para cabecalho, KPIs e atalhos por permissao. As demais telas continuam usando as classes compartilhadas (`panel`, `stat-grid`, `quick-link-card`, `status-badge`) com a base visual atualizada.
- **Assets/estilos:** Adicionados assets decorativos leves em `portal-sama-web/public/brand/sama/dashboard/` e tokens/classes em `src/index.css`; a referencia de marca passou a usar o PNG existente `brand/sama/source/sama-symbol-reference.png`, removendo o aviso de asset `/brand/sama-logo-symbol.svg` no build.
- **Seguranca/acessibilidade:** Links e botoes seguem como HTML real, navegacao por teclado ganhou foco visivel, elementos decorativos nao capturam clique, reduced motion foi preservado e nao houve uso de `innerHTML`, storage local novo, backend ou permissoes novas.
- **Validacao:** Passaram `npm.cmd run lint`, `npm.cmd run build` e `npm.cmd run test:e2e` em `portal-sama-web`. O Playwright passou 3 testes; ficaram apenas avisos conhecidos de `HydrateFallback` e camadas de intro ausentes, relacionados a pendencias anteriores de assets da intro.
- **Pendencias:** propagar `PageShell`/componentes para as telas internas restantes, criar Playwright/QA visual especifico para `/home` e sidebar recolhida, validar desktop/mobile em homologacao e decidir/restaurar os assets de intro que ja apareciam como deletados no workspace antes desta rodada.

## Atualizacao complementar 2026-05-21 10:33 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Iniciada a migracao do `Contabil/integra-ai.html` para uma fatia read-only em `/contabil/integra-ai`, sem continuar a intro nem alterar imagens/assets.
- **Backend/API v2:** Criado `AccountingModule` com `GET /api-v2/accounting/integra-ai/workspaces` e `GET /api-v2/accounting/integra-ai/jobs/:id`. As rotas usam JWT, RBAC `accounting.integra_ai.read`, escopo contabil, SQL parametrizado para leitura das tabelas legadas `sama_integra_ai_*` e auditoria ao abrir detalhe de job.
- **Frontend React:** Criada `IntegraAiPage.tsx` com filtros por busca/empresa/status, tabela de jobs recentes, detalhe de job, resumo de linhas, preview seguro e link controlado para o legado enquanto importacao/parser/exportacao nao migram.
- **Seguranca:** A API nao retorna `source_path`, `txt_path`, payload de parser ou conteudo TXT; a UI renderiza dados pelo React, sem `innerHTML` e sem storage local. Mutacoes de upload, conciliacao, regras e exportacao seguem bloqueadas na API v2 ate haver isolamento de parser/upload e CSRF/auditoria completos.
- **Validacao:** Passaram `npm.cmd test -- accounting.service.spec.ts --runInBand`, `npm.cmd test -- rbac/default-rbac.spec.ts --runInBand`, `npm.cmd run lint` e `npm.cmd run build` na API; `npm.cmd run lint` e `npm.cmd run build` no frontend. O build Vite manteve apenas o aviso conhecido de `/brand/sama-logo-symbol.svg` resolvido em runtime.
- **Pendencias:** aplicar seed RBAC com `accounting.integra_ai.read`, validar com MySQL/homologacao e dados reais do legado, criar Playwright para `/contabil/integra-ai`, migrar com seguranca `parse_bank_statement/create_job/save_step*/generate_txt/download_txt` e desligar `api/integra_ai.php` somente depois da homologacao.

## Atualizacao complementar 2026-05-21 09:34 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Iniciada a migracao da pagina `Depto/modelo.html` para `/departamentos/modelo`, sem continuar a intro nem alterar imagens/assets.
- **Backend/API v2:** Criado `DepartmentsModule` com `GET /api-v2/departments/fiscal/workspace`, `POST /api-v2/departments/fiscal/workspace/cycle-cell` e `PATCH /api-v2/departments/fiscal/workspace/cell-status`. As rotas usam JWT, RBAC `departments.workspace.read/write`, CSRF nas mutacoes, escopo por departamento Fiscal e auditoria nas alteracoes de celula.
- **Frontend React:** Criada `DepartmentModelPage.tsx` com filtros por mes/colaborador, busca por empresa, cards de resumo, carrossel de vencimentos e grade mensal com colunas/status do legado. A rota foi adicionada ao router, menu lateral e atalhos da home.
- **Seguranca:** A leitura respeita escopo operacional de departamento; mutacoes validam permissao, papel operacional, mes atual, coluna permitida, empresa Fiscal e bloqueio por vencimento/mes fechado. A tela renderiza dados pelo React, sem `innerHTML` e sem persistencia em storage local.
- **Validacao:** Passaram `npm.cmd test -- departments.service.spec.ts --runInBand`, `npm.cmd run lint` e `npm.cmd run build` na API; `npm.cmd run lint` e `npm.cmd run build` no frontend. O build Vite manteve apenas o aviso conhecido de `/brand/sama-logo-symbol.svg` resolvido em runtime.
- **Pendencias:** aplicar seed RBAC com `departments.workspace.*`, validar com MySQL/homologacao e dados reais de `Client.metadata`, ampliar Playwright para `/departamentos/modelo`, conferir backfill de responsaveis fiscais e desligar `api/fiscal_workspace.php` somente depois da homologacao.

## Atualizacao complementar 2026-05-21 09:13 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Criada referencia versionada de compose/EasyPanel para a topologia nova, sem continuar a intro nem alterar imagens/assets.
- **Deploy/orquestracao:** Adicionados `docker-compose.easypanel.example.yml` e `compose.easypanel.env.example` com os servicos `portal-sama-mysql`, `portal-sama-api` e `portal-sama-web`, usando os nomes esperados pelo proxy Nginx.
- **Frontend Docker:** Adicionado `portal-sama-web/.dockerignore` para reduzir contexto de build e evitar envio de `node_modules`, `dist`, `.env`, relatorios Playwright e artefatos locais.
- **Nginx:** `portal-sama-web/nginx.conf` passou a resolver `portal-sama-api` via DNS interno Docker `127.0.0.11`, preservando o proxy `/api-v2` mesmo em ciclos de redeploy.
- **Validacao:** `docker compose --env-file compose.easypanel.env.example -f docker-compose.easypanel.example.yml config --services` retornou `portal-sama-mysql`, `portal-sama-api` e `portal-sama-web`; `config --volumes` retornou `portal_sama_mysql` e `portal_sama_private_storage`. `docker run nginx:alpine nginx -t` nao executou porque o Docker daemon nao esta ativo nesta maquina.

## Atualizacao complementar 2026-05-21 08:56 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Ajustado o Nginx do `portal-sama-web` para proxyar a API v2 no mesmo dominio publico, sem continuar a intro nem alterar imagens/assets.
- **Deploy/web:** `portal-sama-web/nginx.conf` agora redireciona `/api-v2` para `/api-v2/`, encaminha `/api-v2/` para `http://portal-sama-api:3000/api-v2/`, preserva headers `X-Forwarded-*`, define `client_max_body_size 30m` e desativa buffering em `/api-v2/notifications/stream` para SSE.
- **Documentacao:** `docs/EASYPANEL_DEPLOY.md` e `docs/INCIDENTE_SUBDOMINIO_PORTAL_SAMACONTABIL.md` foram atualizados com a estrategia de proxy interno e o proximo smoke esperado apos redeploy.
- **Validacao:** `npm.cmd run lint` e `npm.cmd run build` passaram em `portal-sama-web`. `nginx -t` nao foi executado porque o binario Nginx nao esta instalado localmente no Windows.
- **Pendencias:** redeployar `portal-sama-web`, garantir que `portal-sama-api` resolva na rede interna do EasyPanel, publicar API v2 e reexecutar `npm.cmd run smoke:public` sem `--soft`.

## Atualizacao complementar 2026-05-21 08:41 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Criado smoke operacional para validar o dominio publico e a publicacao da API v2, sem continuar a intro nem alterar imagens/assets.
- **Infra/deploy:** Adicionado `scripts/portal-public-smoke.mjs` e o script raiz `npm run smoke:public`, com suporte a `PORTAL_PUBLIC_URL`, `PORTAL_API_BASE_URL`, `PORTAL_CORS_ORIGIN`, `PORTAL_SMOKE_TIMEOUT_MS`, `--api-url`, `--origin`, `--soft` e `--json`.
- **Cobertura do smoke:** A rotina valida a raiz publica, `GET /api-v2/health`, preflight CORS de `/api-v2/auth/me` e emissao de cookie CSRF em `/api-v2/auth/csrf`.
- **Evidencia publica:** Em 2026-05-21 08:41 -03:00, `GET https://portal.samacontabil.com.br` retornou HTTP 200; `/api-v2/health` e `/api-v2/auth/csrf` retornaram HTTP 404; o preflight nao trouxe headers CORS.
- **Pendencias:** publicar/proxyar `portal-sama-api` em `/api-v2` ou validar dominio separado `api.portal.samacontabil.com.br`, reexecutar `npm.cmd run smoke:public` sem `--soft`, e depois validar login, refresh, upload/download e auditoria em HTTPS real.

## Atualização complementar 2026-05-20 17:36 -03:00

- **Responsável/IA:** Codex
- **Resumo da alteração:** Criada a base inicial de Playwright no `portal-sama-web`, sem continuar a intro nem alterar imagens/assets.
- **Frontend/E2E:** Adicionado `@playwright/test`, scripts `test:e2e` e `test:e2e:headed`, `playwright.config.ts` com dev server Vite próprio em `127.0.0.1:4173`, projeto Chromium, trace/screenshot/video em falha e downloads aceitos.
- **Cobertura inicial:** Criada suite `tests/e2e/smoke.spec.ts` cobrindo renderização do login, checklist público de documentos por token com API mockada e auditoria autenticada com listagem/exportação CSV mockadas.
- **Validação:** `npx.cmd playwright install chromium`, `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e` e `git diff --check` passaram. A primeira execução de E2E falhou por seletor ambíguo no campo de senha e passou após ajuste para seletor específico.
- **Pendências:** ampliar Playwright para fluxos críticos autenticados, permissões, mobile, formulários principais e homologação com API v2/MySQL reais.

## Atualização complementar 2026-05-20 17:24 -03:00

- **Responsável/IA:** Codex
- **Resumo da alteração:** Adicionada exportação CSV protegida para auditoria, sem continuar a intro nem alterar imagens/assets.
- **Backend/API v2:** `AuditModule` agora expõe `GET /api-v2/audit/logs/export.csv`, protegido por JWT/RBAC `audit.read`, reutilizando os filtros da listagem, com limite fixo de 1000 linhas, `Cache-Control: no-store`, filename controlado e registro auditável da própria exportação.
- **Segurança:** O CSV escapa aspas/quebras e prefixa células que começam com `=`, `+`, `-` ou `@`, reduzindo risco de CSV injection ao abrir em planilhas. O metadata continua mascarado pelo fluxo de gravação de auditoria.
- **Frontend React:** `/auditoria` ganhou ação `Exportar CSV`, usando os filtros atuais da tela e download por Blob sem persistir o arquivo ou filtros em storage local.
- **Validação:** `npm.cmd test -- audit.service.spec.ts --runInBand`, `npm.cmd run lint` e `npm.cmd run build` passaram na API; `npm.cmd run lint` e `npm.cmd run build` passaram no frontend. O build Vite manteve apenas o aviso conhecido de `/brand/sama-logo-symbol.svg` resolvido em runtime.
- **Pendências:** validar com MySQL real e volume operacional, ampliar Playwright para `/auditoria`, definir política formal de retenção e tratar backup/restauração em contrato separado.

## Atualização complementar 2026-05-20 17:11 -03:00

- **Responsável/IA:** Codex
- **Resumo da alteração:** Implementado checklist público mínimo de documentos por token para a rota pública de onboarding, sem continuar a intro nem alterar imagens/assets.
- **Backend/API v2:** `DocumentsModule` agora expõe `GET /api-v2/documents/public-checklist?token=...`, público e rate-limited, validando token forte, expiração, revogação, módulo/escopo de documentos e vínculo de cliente antes de retornar qualquer dado.
- **Resposta pública:** O contrato retorna apenas `client_id`, `client_name`, rótulo do token, expiração, itens esperados, status resumido, extensões permitidas e timestamp. Não expõe `doc_id`, URL de download, storage key, usuário interno, histórico ou metadados sensíveis.
- **Frontend React:** `/onboarding/publico/documentos/:token` passou a carregar o checklist real, exibir status dos itens solicitados, usar as extensões permitidas vindas da API e preencher documento/departamento ao selecionar um item.
- **Validação:** `npm.cmd test -- documents.service.spec.ts --runInBand`, `npm.cmd run lint` e `npm.cmd run build` passaram na API; `npm.cmd run lint` e `npm.cmd run build` passaram no frontend. Uma tentativa intermediária de build web falhou por referência ao array antigo de extensões e foi corrigida antes da validação final.
- **Pendências:** validar em homologação com MySQL/storage/ClamAV reais, tokens reais/legados, Playwright desktop/mobile e HTTPS.

## Atualização complementar 2026-05-20 16:51 -03:00

- **Responsável/IA:** Codex
- **Resumo da alteração:** Implementado code splitting por rota no `portal-sama-web`, sem continuar a intro nem alterar imagens/assets nesta rodada.
- **Frontend React:** `router.tsx` deixou de importar layout e páginas de forma ansiosa e passou a usar `lazy` do React Router para `AppLayout`, páginas privadas, páginas públicas, erro 404 e rota dev de preview.
- **Impacto de build:** o chunk inicial do Vite caiu para 320.90 kB (gzip 101.49 kB), com `AppLayout` separado em 138.47 kB (gzip 45.33 kB), eliminando o alerta de chunk acima de 500 kB na execução final.
- **Validação:** `npm.cmd run lint`, `npm.cmd run build` e `git diff --check` passaram. Uma tentativa intermediária de build falhou por uso de `fallbackElement` não suportado no `RouterProvider`; a prop foi removida antes da validação final.
- **Pendências:** validar navegação real desktop/mobile e Playwright em homologação. O aviso conhecido de `/brand/sama-logo-symbol.svg` resolvido em runtime permanece.

## Atualização complementar 2026-05-20 16:39 -03:00

- **Responsável/IA:** Codex
- **Resumo da alteração:** Integrada a intro visual modular do Portal Sama ao `portal-sama-web` usando o pacote SVG polido em camadas, conforme a documentação de assets.
- **Assets públicos:** Copiados `layer-manifest.json`, `animation-map.json` e `assets/` para `portal-sama-web/public/brand/sama/intro`, totalizando 86 arquivos servidos pela aplicação Vite.
- **Frontend React:** Criado renderer reutilizável por manifesto (`PortalSamaIntro`, `PortalSamaSceneRenderer`, `PortalSamaLayer`, `PortalSamaStaticFallback`) com ordenação por `zIndex`, animações por `animationGroup`, paths centralizados, fallback estático por cena e tolerância a erro de camada opcional.
- **Intro existente:** `StandardIntro`, `WelcomeIntro` e `/dev/intro-preview` passaram a usar o renderer em vez de montar fundo, texto, órbitas e logo por componentes React/CSS. A lógica de `IntroGate`, boas-vindas uma vez, skip e `prefers-reduced-motion` foi preservada.
- **Reduced motion/fallback:** Em `mode="auto"` com reduced motion, a intro usa composite estático; em erro de manifesto/camada obrigatória também cai para fallback sem quebrar o portal.
- **Validação:** `npm.cmd run lint` e `npm.cmd run build` passaram em `portal-sama-web`. O build Vite manteve avisos conhecidos de chunk acima de 500 kB e asset `/brand/sama-logo-symbol.svg` resolvido em runtime; `git diff --check` passou com avisos LF/CRLF do Git no Windows.
- **Pendências:** Validar visualmente em navegador desktop/mobile e em homologação HTTPS, revisar timing/fidelidade com as referências aprovadas e manter os arquivos deletados em `docs/portal-sama-svg-intro-pack` como mudança externa já existente até decisão de versionamento.

## Atualizacao complementar 2026-05-20 16:21 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Ativada a primeira integracao React de tempo real e Web Push na central `/notificacoes`, reduzindo a dependencia do `global.js` legado para esses fluxos.
- **Frontend React:** `notifications.service.ts` agora abre `GET /api-v2/notifications/stream` por `fetch` com bearer em memoria, parseia SSE e invalida as queries de notificacoes em eventos de criacao, atualizacao e limpeza. A tela `/notificacoes` exibe status de tempo real, consulta suporte/permissao do navegador, permite ativar/desativar a assinatura Push API e segue mantendo o teste operacional de Web Push.
- **Service Worker:** Adicionado `portal-sama-web/public/sama-push-sw.js` para que o app Vite sirva o worker usado por Push API no escopo raiz, abrindo `/notificacoes` ao clicar na notificacao.
- **Seguranca:** O stream React nao coloca access token em query string; usa `Authorization` via `fetch`, `credentials: include` e access token apenas em memoria. Subscribe/unsubscribe Web Push exigem `issueCsrfToken()` e continuam validados no backend por JWT, RBAC e CSRF. A tela nao usa `innerHTML` nem persiste tokens/segredos em storage local.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build` e `npm.cmd audit --audit-level=moderate` passaram em `portal-sama-web`. O build Vite manteve avisos conhecidos de chunk acima de 500 kB e asset `/brand/sama-logo-symbol.svg` resolvido em runtime; `git diff --check` passou com avisos LF/CRLF do Git no Windows.
- **Pendencias:** Validar manualmente em HTTPS/homologacao com API v2/MySQL/VAPID reais, permissao real do navegador, portal fechado, unsubscribe, comportamento mobile/desktop, Playwright e estrategia broker/event bus para multi-instancia se necessario.

## Atualizacao complementar 2026-05-20 16:02 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Migrada a exclusao direta de vencimentos do workspace do gestor para a API v2 e conectada ao dashboard React.
- **Backend/API v2:** `CalendarModule` agora expoe `DELETE /api-v2/calendar/entries/:id`, protegido por JWT, `calendar.manage`, papel ADMIN/DEV/MANAGER, CSRF, validacao de regra/empresa/departamento e auditoria `calendar.entry.delete`. A exclusao remove o vinculo `sama_calendar_rule_companies`, limpa notificacoes em `sama_calendar_due_notified` e remove a regra orfa quando nao ha mais empresas vinculadas.
- **Frontend React:** `/manager` ganhou acao de remover vencimento na lista de proximos eventos, exibida somente para sessoes com `calendar.manage`, usando `issueCsrfToken()` e invalidando o calendario mensal apos sucesso.
- **Testes:** Criado `calendar.service.spec.ts` cobrindo exclusao com limpeza/auditoria e bloqueio fora do escopo de departamento. Passaram `npm.cmd test -- calendar.service.spec.ts`, `npm.cmd run lint`, `npm.cmd run build` e `npm.cmd run prisma:validate` na API; `npm.cmd run lint`, `npm.cmd run build` no frontend e `git diff --check`. O build Vite manteve avisos conhecidos de chunk grande e asset de marca em runtime; o diff check teve apenas avisos LF/CRLF do Git no Windows.
- **Pendencias:** Validar com MySQL/homologacao, seed RBAC real de `calendar.manage`, usuarios reais por perfil/departamento, Playwright para remover vencimento/sem permissao e desligamento controlado das actions PHP `calendar_*` somente depois da homologacao.

## Atualizacao complementar 2026-05-20 12:30 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Migrada a action `overview` do workspace do gestor para contrato proprio na API v2 e conectada ao dashboard React.
- **Backend/API v2:** `ManagersModule` agora expoe `GET /api-v2/managers/overview`, protegido por JWT, `manager_history.read`, papel ADMIN/DEV/MANAGER e escopo por departamento. A resposta preserva o formato operacional do legado com `kpis`, `items`, `online`, `offline`, `allowed_depts`, `target_depts`, `dept_filter` e timeout de presenca.
- **Banco/Prisma:** Adicionado o modelo `UserPresence`, mapeado para `sama_user_presence`, com migration idempotente `20260520123000_add_user_presence_model`.
- **Frontend React:** `/manager` passou a chamar `getManagerOverview()` e exibir presenca da equipe, contadores online/offline e filtro integrado ao dashboard existente de carteiras, transferencias e vencimentos.
- **Seguranca:** Leitura exige permissao e escopo no backend; usuarios administrativos/clientes/auditores sao removidos da lista operacional; a presenca e somente leitura e nao usa `innerHTML` nem storage local.
- **Validacao:** `npm.cmd run prisma:format`, `npm.cmd run prisma:generate`, `npm.cmd run prisma:validate`, `npm.cmd run lint` e `npm.cmd run build` passaram na API; `npm.cmd run lint` e `npm.cmd run build` passaram no frontend; `git diff --check` passou com avisos LF/CRLF do Git no Windows. Build Vite manteve avisos conhecidos de chunk grande e asset de marca em runtime.
- **Pendencias:** Validar em MySQL/homologacao com usuarios reais, aplicar migrations/seed RBAC no ambiente, criar Playwright para `/manager` e `/manager/historico`, e desligar a action PHP `overview` somente apos homologacao.

## Atualizacao complementar 2026-05-20 11:45 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Continuada a migracao do historico operacional do gestor com mutacoes auditaveis na API v2 e formularios React na rota `/manager/historico`.
- **Backend/API v2:** `ManagersModule` agora expoe `POST /api-v2/managers/history`, `PATCH /api-v2/managers/history/:id` e `POST /api-v2/managers/company-life`, alem das leituras ja existentes. As mutacoes exigem JWT, `manager_history.write`, papel ADMIN/DEV/MANAGER, escopo por departamento, CSRF e auditoria.
- **RBAC:** Adicionada a permissao `manager_history.write` ao catalogo padrao e ao papel MANAGER; ADMIN/DEV continuam recebendo o catalogo completo.
- **Frontend React:** `/manager/historico` ganhou formulario para novo historico, edicao de registros de `company_history` e salvamento de snapshot de vida da empresa, usando CSRF pelo service centralizado e exibindo as acoes apenas quando `can_edit`/permissao permitirem.
- **Seguranca:** Topicos sao normalizados e limitados no backend, IDs sao validados, o escopo empresa/departamento e checado antes de gravar, e dados de historico continuam sem `innerHTML` ou storage local no frontend.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run prisma:validate` e `npx.cmd jest src/modules/rbac/default-rbac.spec.ts --runInBand` passaram na API; `npm.cmd run lint` e `npm.cmd run build` passaram no frontend. `git diff --check` passou, com avisos LF/CRLF do Git no Windows.
- **Pendencias:** Validar com MySQL/homologacao e usuarios reais, executar seed RBAC atualizado incluindo `manager_history.write`, criar Playwright para leitura/escrita/sem permissao e manter `api/manager_workspace.php?action=overview` no legado ate existir contrato proprio.

## Atualizacao complementar 2026-05-20 11:20 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Implementada a primeira fatia de leitura do historico operacional do gestor, sem continuar as implementacoes da introducao da plataforma.
- **Backend/API v2:** Criado `ManagersModule` com `GET /api-v2/managers/history` e `GET /api-v2/managers/history/:companyId/timeline`, protegidos por JWT, `manager_history.read`, papel ADMIN/DEV/MANAGER e escopo por departamento.
- **Banco/Prisma:** Adicionados modelos `CompanyHistoryEntry` e `CompanyLifeEntry`, mapeados para `sama_company_history_entries` e `sama_company_life_entries`, com migration compativel `20260520112000_add_manager_history_entries`.
- **Frontend React:** Criados `types/manager-history.ts`, `services/manager-history.service.ts` e `pages/manager/ManagerHistoryPage.tsx`. A rota `/manager/historico` lista empresas por departamento, busca historico e exibe timeline consolidada de historico + vida da empresa.
- **Navegacao:** `router.tsx`, `navigation.tsx`, `HomePage.tsx` e o dashboard do gestor ganharam acesso ao historico condicionado a `manager_history.read`.
- **Seguranca:** A tela e somente leitura nesta fatia, nao usa `innerHTML`, nao grava historico em storage local e delega RBAC/escopo ao backend. Escrita/edicao ficou pendente para rota dedicada com CSRF e auditoria.
- **Validacao:** `npx.cmd prisma format`, `npx.cmd prisma generate`, `npm.cmd run prisma:validate`, `npm.cmd run lint`, `npm.cmd run build` e `npx.cmd jest src/modules/rbac/default-rbac.spec.ts --runInBand` passaram na API; `npm.cmd run lint` e `npm.cmd run build` passaram no frontend.
- **Pendencias:** Validar com MySQL/homologacao e usuarios reais, executar seed RBAC atualizado, criar Playwright e implementar mutacoes auditaveis de historico quando o contrato for fechado.

## Atualizacao complementar 2026-05-19 17:56 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Implementada a primeira fatia React da rota `/dev/colaboradores`, substituindo a tela legada `DEV/dev-colaborador.html` por uma gestao administrativa inicial conectada ao `CollaboratorsModule`, sem continuar as implementacoes da introducao da plataforma.
- **Frontend React:** Criado `pages/dev/DevCollaboratorsPage.tsx`. A tela cobre filtros por busca/status/departamento/perfil, exibicao de colaboradores, estatisticas, resumo administrativo, edicao de dados funcionais, status, senha opcional, perfis e arquivamento logico por permissao.
- **Navegacao:** `router.tsx` passou a renderizar `/dev/colaboradores`; `DevAdminPage.tsx` ganhou atalho `Gerenciar colaboradores` condicionado a `collaborators.read`.
- **Seguranca:** A tela usa `collaborators.read/update/delete` e `roles.read` apenas para experiencia; leitura e mutacoes continuam protegidas no backend por JWT, RBAC, CSRF, validacao server-side, bloqueio de role `CLIENT` e auditoria. Dados sao renderizados pelo React, sem `innerHTML`, e senha/credenciais nao sao persistidas em storage do navegador.
- **Validacao:** `portal-sama-web` passou em `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate`, `git diff --check` nos arquivos da fatia e HTTP 200 em `http://127.0.0.1:5174/dev/colaboradores`. O build Vite manteve os avisos conhecidos de chunk acima de 500 kB e asset `/brand/sama-logo-symbol.svg` resolvido em runtime.
- **Artefatos:** `.ai-tests/runs/2026-05-19-1756-dev-collaborators-react/commands.md` e `summary.md`.
- **Pendencias:** Validar com API v2/MySQL/usuarios reais, revisar escopo administrativo em homologacao, executar backfill de `sama_colaboradores`, formalizar vinculos colaborador-cliente/gestor/carteira e criar Playwright.

## Atualizacao complementar 2026-05-19 15:53 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Implementada a primeira fatia React da rota `/manager/colaboradores`, usando o contrato seguro do `TransfersModule` para consulta de carteira por colaborador, sem continuar as implementacoes da introducao da plataforma.
- **Frontend React:** Criado `pages/manager/ManagerCollaboratorsPage.tsx`. A tela cobre departamento ativo, busca de colaboradores, estatisticas de carteira, empresas vinculadas ao colaborador selecionado, status de transferencia ativa, destino temporario quando houver e sessoes relacionadas.
- **Navegacao:** `router.tsx` passou a renderizar `/manager/colaboradores`; `navigation.tsx` e `HomePage.tsx` ganharam atalho `Carteira gestor` condicionado a `transfers.read`.
- **Seguranca:** A tela e somente consulta nesta fatia, usa `transfers.read` apenas para experiencia e consome `GET /api-v2/transfers/dashboard`; autorizacao, RBAC, escopo por departamento e dados de carteira continuam validados no backend. Dados sao renderizados pelo React, sem `innerHTML`, e nao sao persistidos em storage do navegador.
- **Validacao:** `portal-sama-web` passou em `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate`, `git diff --check` nos arquivos da fatia e HTTP 200 em `http://127.0.0.1:5174/manager/colaboradores`. O build Vite manteve os avisos conhecidos de chunk acima de 500 kB e asset `/brand/sama-logo-symbol.svg` resolvido em runtime.
- **Artefatos:** `.ai-tests/runs/2026-05-19-1553-manager-collaborators-react/commands.md` e `summary.md`.
- **Pendencias:** Validar com API v2/MySQL/usuarios reais, conferir escopo por departamento/carteira com seed `transfers.*`, criar Playwright e decidir modelagem formal de carteira antes de substituir totalmente o legado do gestor.

## Atualizacao complementar 2026-05-19 15:33 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Implementada a primeira fatia React da rota `/manager/transferencias`, consumindo o `TransfersModule` criado anteriormente, sem continuar as implementacoes da introducao da plataforma.
- **Frontend React:** Criados `types/transfers.ts`, `schemas/transfer.schema.ts`, `services/transfers.service.ts` e `pages/manager/ManagerTransfersPage.tsx`. A tela cobre dashboard por departamento, selecao de colaborador de origem, motivos legados, periodo/justificativa, busca de empresas, destino por empresa, criacao de sessao e retorno manual de transferencias por tempo indeterminado.
- **Navegacao:** `router.tsx` passou a renderizar `/manager/transferencias`; `navigation.tsx` e `HomePage.tsx` ganharam atalho condicionado a `transfers.read`.
- **Seguranca:** A tela usa `transfers.read/create/return` apenas para experiencia; leitura e mutacoes seguem protegidas no backend por JWT, RBAC, CSRF em criacao/retorno, escopo por departamento e auditoria. Dados sao renderizados pelo React, sem `innerHTML`, e nao sao persistidos em storage do navegador.
- **Validacao:** `portal-sama-web` passou em `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` e `git diff --check` nos arquivos da fatia. O dev server foi iniciado em `http://127.0.0.1:5174` porque a porta 5173 ja estava em uso. O build Vite manteve os avisos conhecidos de chunk acima de 500 kB e asset `/brand/sama-logo-symbol.svg` resolvido em runtime.
- **Artefatos:** `.ai-tests/runs/2026-05-19-1533-manager-transfers-react/commands.md` e `summary.md`.
- **Pendencias:** Validar com API v2/MySQL/seed/backfill reais, exercitar JWT/CSRF com usuarios MANAGER/ADMIN/DEV e sem permissao, criar Playwright e decidir modelagem formal de carteira antes de desligar o legado `transfer_*`.

## Atualizacao complementar 2026-05-19 12:16 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Criada a primeira fatia backend do `TransfersModule` para substituir as actions `transfer_dashboard`, `transfer_submit` e `transfer_return` de `api/manager_workspace.php`, sem continuar a implementacao da introducao da plataforma.
- **Backend/API v2:** Adicionados `GET /api-v2/transfers`, `GET /api-v2/transfers/dashboard`, `POST /api-v2/transfers`, `POST /api-v2/transfers/return` e `POST /api-v2/transfers/:id/return`. Mutacoes exigem CSRF, JWT e permissoes `transfers.create`/`transfers.return`; leitura exige `transfers.read`.
- **Banco/Prisma:** Adicionado o modelo `TransferSession` mapeado para a tabela legada `sama_transfer_sessions`, com migration compativel `20260519120000_add_transfer_sessions` usando `CREATE TABLE IF NOT EXISTS` e indices condicionais.
- **Regras implementadas:** Escopo por ADMIN/DEV/MANAGER e departamento, motivos legados (`ferias`, `saida_colab`, `falta_imprevisto`, `tempo_indeterminado`), validacao de periodo, justificativa obrigatoria para falta/imprevisto, bloqueio de cliente ja transferido, reconciliacao de sessoes ativas/expiradas e retorno manual de transferencia indeterminada.
- **Compatibilidade:** O modulo le e atualiza vinculos de carteira em `Client.metadata.departamentos/depts/dept_*` e `User.metadata.clientes/colaboradorId`, preservando aliases de resposta do legado para reduzir atrito na futura tela React.
- **Validacao:** `prisma:format`, `prisma:generate`, `prisma:validate`, `build`, `lint`, teste focado `transfers.service.spec.ts` e a suite completa `npm.cmd test -- --runInBand` passaram no backend.
- **Artefatos:** `.ai-tests/runs/2026-05-19-1216-transfers-module/commands.md` e `summary.md`.
- **Pendencias:** Validar migration em MySQL real/homologacao, executar seed de RBAC atualizado, fazer backfill/normalizacao definitiva de carteira e validar a tela React `/manager/transferencias` com dados reais e Playwright.

## Atualizacao complementar 2026-05-18 17:35 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Implementada a primeira fase do sistema visual/motion descrito em `docs/design/ANIMACOES_ENTRADA_E_MOTION_PORTAL_SAMA.md`.
- **Frontend React:** `portal-sama-web` recebeu `framer-motion`, assets locais em `public/brand/`, feature `src/features/intro` com `IntroGate`, `StandardIntro`, `WelcomeIntro`, `MotionLogo`, fundo, particulas, orbitas, loading cue, reduced motion e transicao suave para o conteudo privado. O layout autenticado passou a exibir a intro apenas em rotas privadas; login e fluxos publicos permanecem sem intro.
- **Design system aplicado:** Sidebar, header, home, botoes, cards de estatistica e atalhos foram refinados com navy institucional, acentos azul/laranja, sombras discretas, microinteracoes e logo local, mantendo bordas de 8px e sem adicionar dependencias pesadas de particulas.
- **Backend/API v2:** Criado `PATCH /api-v2/me/preferences/intro`, protegido por JWT e CSRF, para persistir `welcomeAnimationSeen`, `welcomeAnimationSeenAt` e `introAnimationVersion` em `User.metadata.portalIntro`, com auditoria `auth.intro_preferences.update`.
- **Seguranca:** A intro nao aparece antes da autenticacao, nao expoe dados sensiveis, nao grava token em storage e falha de marcacao da preferencia nao bloqueia a abertura do portal. Usuarios sem flag historica podem ver a tela de boas-vindas uma vez apos a implantacao.
- **Validacao:** `portal-sama-web` passou em lint, build e audit; `portal-sama-api` passou em lint, build, audit e teste focado `auth.service.spec.ts`; `git diff --check` passou. O build Vite manteve aviso de chunk acima de 500 kB, agora registrado como pendencia de otimizacao/code splitting.
- **Artefatos:** `.ai-tests/runs/2026-05-18-1735-design-motion-intro/commands.md` e `summary.md`.

## Atualizacao complementar 2026-05-18 15:58 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/departamentos/clientes` foi criada para substituir a primeira fatia da tela legada `DptClient/clientdpto.html`.
- **Frontend React:** Criado `pages/departments/DepartmentClientsPage.tsx`, consumindo `GET /api-v2/clients` via `clients.service.ts`. A tela cobre departamento ativo da sessao, filtros por busca/status/grupo, estatisticas, clientes com/sem responsavel quando o metadata legado/backfill trouxer vinculo, tabela operacional e atalhos para painel/documentos.
- **Navegacao:** `router.tsx` passou a renderizar `/departamentos/clientes`; `navigation.tsx` e `HomePage.tsx` ganharam atalho condicionado a `clients.read`.
- **Seguranca:** A tela usa `clients.read` apenas para experiencia; leitura continua protegida no backend por JWT, RBAC e escopo do `ClientsModule`. Dados sao renderizados pelo React, sem `innerHTML`, e nao sao persistidos em storage do navegador. Atribuicao/transferencia de responsavel nao foi migrada porque ainda falta contrato backend transacional, auditoria e modelagem real de carteira/departamento.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram antes da atualizacao documental. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff teve apenas aviso LF/CRLF do Git no Windows. Permanecem pendentes validacao com API v2/MySQL/usuarios reais, backfill de vinculos cliente-departamento-responsavel, Playwright e endpoint seguro para atribuicao/transferencia.
- **Estado atualizado:** Clientes agora possuem telas React de cadastro/listagem, painel, visao consolidada, cadastro DEV e visao departamental inicial.

## Atualizacao complementar 2026-05-18 15:35 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/ti/acessos` deixou de usar placeholder e passou a ter uma primeira tela operacional segura para TI.
- **Frontend React:** Criado `pages/ti/TiAccessPage.tsx`, consumindo `GET /api-v2/access-requests`, `GET /api-v2/access-requests/manager/approvals`, `POST /api-v2/access-requests/:id/approve` e `POST /api-v2/access-requests/:id/reject` via `access-requests.service.ts`. A tela cobre filtros, estatisticas, fila operacional, historico, paginacao e decisoes por permissao.
- **Navegacao:** `router.tsx` passou a renderizar `TiAccessPage` em `/ti/acessos`, mantendo o menu condicionado a `access_requests.read`.
- **Seguranca:** A tela usa `access_requests.read/approve/reject` apenas para experiencia; leitura e decisoes continuam protegidas no backend por JWT, CSRF nas decisoes, RBAC, escopo por solicitante/gestor/departamento/TI e auditoria. A base editavel de credenciais da TI nao foi migrada nesta fatia e segue bloqueada ate existir modelagem com criptografia/vault, auditoria de leitura e politica de retencao.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas aviso LF/CRLF do Git no Windows. Permanecem pendentes validacao com API v2/MySQL/backfill/usuarios reais, escopo real por TI/gestor/departamento, Playwright, HTTPS/homologacao e cofre seguro de credenciais.
- **Estado atualizado:** Solicitacoes de acesso agora possuem tela de solicitante/gestor em `/solicitacao-acesso` e tela operacional de TI em `/ti/acessos`, ambas sem migrar a base de segredos.

## Atualizacao complementar 2026-05-18 15:14 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/dev/colaboradores/novo` foi criada para substituir a tela legada de cadastro administrativo de colaborador.
- **Frontend React:** Criado `pages/dev/DevNewCollaboratorPage.tsx`, consumindo `POST /api-v2/collaborators` via `collaborators.service.ts` e, quando permitido, `GET /api-v2/roles` para opcoes de perfis. A tela cobre usuario, nome, e-mail, senha provisoria com confirmacao local, departamento, cargo, telefone, ramal, status, perfis e feedback com atalho para a visao do colaborador.
- **Navegacao:** `router.tsx` passou a renderizar `/dev/colaboradores/novo`; `DevAdminPage.tsx` ganhou atalho condicionado a `collaborators.create`.
- **Seguranca:** A tela usa `collaborators.create` e `roles.read` apenas para experiencia; criacao continua protegida no backend por JWT, CSRF, RBAC, bloqueio de role `CLIENT`, hash de senha, validacao server-side e auditoria sem senha/hash. Dados sao renderizados pelo React, sem `innerHTML`, e senha provisoria nao e persistida em storage do navegador.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas avisos LF/CRLF do Git no Windows. Permanecem pendentes validacao com API v2/MySQL/usuarios reais, backfill de colaboradores/usuarios legados, vinculos cliente-usuario/gestor/carteira e Playwright.
- **Estado atualizado:** DEV/admin agora possui gestao de usuarios/roles/permissoes, cadastro dedicado de cliente e cadastro dedicado de colaborador em React.

## Atualizacao complementar 2026-05-18 15:07 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/dev/clientes/novo` foi criada para substituir a tela legada de cadastro administrativo de cliente.
- **Frontend React:** Criado `pages/dev/DevNewClientPage.tsx`, consumindo `POST /api-v2/clients` via `clients.service.ts`. A tela cobre dados da empresa, classificacao, contato, endereco, grupo empresarial, feedback do cliente criado e atalho para o painel do cliente.
- **Navegacao:** `router.tsx` passou a renderizar `/dev/clientes/novo`; `DevAdminPage.tsx` ganhou atalho condicionado a `clients.create`.
- **Seguranca:** A tela usa `clients.create` para experiencia; criacao continua protegida no backend por JWT, CSRF, RBAC, validacao server-side, normalizacao de CNPJ e auditoria. Dados sao renderizados pelo React, sem `innerHTML`, e nao sao persistidos em storage do navegador.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas avisos LF/CRLF do Git no Windows. Permanecem pendentes validacao com API v2/MySQL/usuarios reais, backfill de clientes legados, vinculos cliente-usuario/gestor/carteira e Playwright.
- **Estado atualizado:** DEV/admin agora possui gestao de usuarios/roles/permissoes e cadastro dedicado de cliente em React.

## Atualizacao complementar 2026-05-18 14:52 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/onboarding/entrada-cliente` foi criada para substituir a tela legada de entrada de cliente.
- **Frontend React:** Criado `pages/onboarding/OnboardingClientEntryPage.tsx`, consumindo `POST /api-v2/onboarding/processes` para iniciar processos e `GET /api-v2/onboarding/processes` para a esteira recente. Quando a sessao possui `clients.read`, a tela tambem consome `GET /api-v2/clients` para preencher empresa/CNPJ a partir de cliente ja cadastrado.
- **Navegacao:** `router.tsx` passou a renderizar `/onboarding/entrada-cliente`; o menu, a Home e `/onboarding/processos` ganharam atalho para a nova entrada.
- **Seguranca:** A tela usa `onboarding.create`, `onboarding.read` e `clients.read` apenas para experiencia; criacao e leitura continuam protegidas no backend por JWT, CSRF nas mutacoes, RBAC, escopo por responsavel/departamento, auditoria e validacao server-side. Nenhum token publico ou dado sensivel e persistido em storage do navegador.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas avisos LF/CRLF do Git no Windows. Permanecem pendentes validacao com API v2/MySQL/usuarios reais, maquina de estados completa, links publicos especificos, backfill, Playwright e homologacao HTTPS.
- **Estado atualizado:** Onboarding agora possui gestao de processos e entrada dedicada de cliente em React, alem dos fluxos publicos ja migrados.

## Atualizacao complementar 2026-05-18 14:39 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/clientes/colaboradores/:id` foi criada para substituir a visao legada individual de colaborador.
- **Frontend React:** Criado `pages/collaborators/CollaboratorViewPage.tsx`, consumindo `GET /api-v2/collaborators/:id`. A tela cobre identidade funcional, contato, status, perfis, permissoes efetivas, datas, metadados renderizados como JSON seguro e estado vazio para carteira de clientes quando o backend nao retorna vinculos.
- **Navegacao:** `router.tsx` passou a renderizar `/clientes/colaboradores/:id`; a listagem `/colaboradores/visao-geral` agora linka o nome do colaborador para o detalhe.
- **Seguranca:** A tela usa `collaborators.read` para experiencia; a decisao real continua protegida no backend por JWT, RBAC, escopo por departamento, auditoria e validacao server-side. Metadados sao exibidos como texto/JSON, sem `innerHTML`, e nenhum dado funcional e gravado em storage do navegador.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; a primeira tentativa de audit falhou por erro temporario do endpoint do registry e passou no retry. O diff final teve apenas avisos LF/CRLF do Git no Windows. Permanecem pendentes validacao com API v2/MySQL/usuarios reais, backfill, vinculos colaborador-cliente/gestor, carteira/transferencias e Playwright.
- **Estado atualizado:** Colaboradores agora possuem visao geral React e detalhe individual React. A carteira real de clientes do colaborador segue dependente de modelagem/backfill de vinculos.

## Atualizacao complementar 2026-05-18 14:27 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/clientes/visao-geral` foi criada para substituir a visao consolidada legada de clientes.
- **Frontend React:** Criado `pages/clients/ClientsOverviewPage.tsx`, consumindo `ClientsModule` e o resumo de pendencias obrigatorias do `DocumentsModule`. A tela cobre filtros por busca, status, CNPJ, cidade, grupo e arquivados, estatisticas da pagina, documentos obrigatorios por cliente quando houver `documents.requirements`, cards de pendencias e atalhos para painel do cliente e central de documentos.
- **Navegacao:** `router.tsx` passou a renderizar `/clientes/visao-geral`; `ClientsPage.tsx` ganhou atalho para a visao geral e `DocumentsPage.tsx` passou a aceitar `clientId` vindo da query string para abrir a central ja filtrada.
- **Seguranca:** A tela usa `clients.read` e `documents.requirements` apenas para experiencia; leitura e resumo documental continuam protegidos no backend por JWT, RBAC, escopo e validacao server-side. A tela nao usa `innerHTML` nem grava dados de cliente em storage do navegador.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas avisos LF/CRLF do Git no Windows. Ainda faltam validacao com API v2/MySQL/usuarios reais, backfill, escopo real por carteira/departamento e Playwright.
- **Estado atualizado:** Clientes agora possuem cadastro React, painel individual, documentos no painel e visao consolidada React. Ainda faltam validacao integrada e vinculos/carteira reais.

## Atualizacao complementar 2026-05-18 12:27 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React interna `/documentos` deixou de ser placeholder e passou a consumir o `DocumentsModule`.
- **Frontend React:** Criado `pages/documents/DocumentsPage.tsx` e ampliados `types/documents.ts`, `document.schema.ts` e `documents.service.ts`. A tela cobre filtros por cliente/tipo/departamento/status, estatisticas da pagina, upload interno por cliente, download protegido, revisao de status, arquivamento, links publicos por token, revogacao de links e resumo de pendencias obrigatorias por permissao.
- **Navegacao:** `router.tsx` passou a renderizar `DocumentsPage` em `/documentos`, mantendo o menu condicionado a `documents.read`.
- **Seguranca:** A tela usa `documents.read/upload/download/review/delete/requirements/public_tokens` apenas para experiencia; leitura e mutacoes continuam protegidas no backend por JWT, CSRF, RBAC, escopo por cliente/departamento, token publico opaco, storage privado, scanner/quarentena, auditoria e validacao server-side. Tokens brutos gerados sao mostrados apenas no retorno imediato da criacao e nao sao persistidos em storage do navegador.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas avisos LF/CRLF do Git no Windows. Ainda faltam validacao com API v2/MySQL/storage/ClamAV reais, backfill, usuarios/permissoes reais e Playwright.
- **Estado atualizado:** Documentos internos agora possuem primeira central React funcional. O painel do cliente e a rota publica de upload continuam usando o mesmo service/contrato.

## Atualizacao complementar 2026-05-18 09:17 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/colaboradores/visao-geral` deixou de ser placeholder e passou a consumir o `CollaboratorsModule`.
- **Frontend React:** Criados `types/collaborators.ts`, schema Zod `collaborator.schema.ts`, service `collaborators.service.ts` e `pages/collaborators/CollaboratorsOverviewPage.tsx`. A tela cobre filtros por busca, status, departamento, perfil e arquivados, estatisticas da pagina, listagem, detalhe seguro, cadastro, edicao, troca de status/perfis e arquivamento por permissao.
- **Navegacao:** `router.tsx` passou a renderizar `CollaboratorsOverviewPage` em `/colaboradores/visao-geral`, mantendo o menu condicionado a `collaborators.read`.
- **Seguranca:** A tela usa `collaborators.read/create/update/delete` apenas para experiencia; leitura e mutacoes continuam protegidas no backend por JWT, CSRF, RBAC, bloqueio de role `CLIENT`, escopo por departamento para perfis nao privilegiados, auditoria e validacao server-side. Dados e metadados sao renderizados sem `innerHTML`, e senha nova nao fica persistida em storage do navegador.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas avisos LF/CRLF do Git no Windows. Ainda faltam validacao com API v2/MySQL/usuarios reais, backfill, vinculos/carteira/transferencias e Playwright.
- **Estado atualizado:** Colaboradores agora possuem primeira experiencia React funcional para visao geral e manutencao cadastral. A parte de carteira/transferencias de gestor segue mapeada para modulos futuros.

## Atualizacao complementar 2026-05-18 08:46 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React publica `/onboarding/publico/documentos/:token` deixou de ser placeholder e passou a enviar documentos pelo `DocumentsModule`.
- **Frontend React:** Adicionados `PublicDocumentsPage.tsx`, schema Zod `publicDocumentUploadSchema`, tipo `PublicDocumentUploadInput` e service `uploadPublicDocument()` em `documents.service.ts`. A tela publica cobre tipo de documento, departamento opcional, selecao de arquivo, validacao visual de extensao/tamanho, envio multipart e lista local dos documentos recebidos na sessao.
- **Navegacao:** `router.tsx` passou a renderizar `PublicDocumentsPage` em `/onboarding/publico/documentos/:token`.
- **Seguranca:** A tela opera por token da URL, nao persiste token em `localStorage`/`sessionStorage`, nao usa `innerHTML` e trata validacao de frontend apenas como apoio. A validacao definitiva continua no backend em `POST /api-v2/documents/public-upload?token=...`, com `PublicToken`, hash SHA-256, expiracao, revogacao, escopo, throttle, scanner/quarentena, storage privado, auditoria e notificacao interna.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas avisos LF/CRLF do Git no Windows.
- **Estado atualizado:** Documentos publicos de onboarding agora possuem primeira experiencia React funcional para upload por token e checklist publico minimo na API v2. Ainda faltam validacao com API v2/MySQL/storage/ClamAV reais, tokens legados, HTTPS/homologacao e Playwright.

## Atualizacao complementar 2026-05-15 16:13 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** As rotas React de contratos e assinatura publica deixaram de ser placeholders e passaram a consumir o `ContractsModule`.
- **Frontend React:** Criados `types/contracts.ts`, `schemas/contract.schema.ts`, `services/contracts.service.ts`, `pages/contracts/ContractPage.tsx` e `pages/contracts/PublicSignaturePage.tsx`. A tela interna cobre filtros, estatisticas, listagem, criacao, edicao, geracao de snapshot HTML, renderizacao PDF via backend, importacao/download de PDF privado, envio por link publico e copia do link React `/assinatura/:token`. A tela publica consulta `GET /api-v2/public/signatures/:token`, exibe dados minimos do contrato e permite registrar assinatura em `POST /api-v2/public/signatures/:token/sign`.
- **Navegacao:** `router.tsx` passou a renderizar `ContractPage` em `/legalizacao/contratos` e `/legalizacao/contratos/:id`, e `PublicSignaturePage` em `/assinatura/:token`. `navigation.tsx`, `HomePage.tsx` e `LegalizationPage.tsx` ganharam atalhos condicionados a `contracts.read`.
- **Seguranca:** A tela interna usa `contracts.read/generate/sign` apenas para experiencia; mutacoes continuam protegidas no backend por JWT, CSRF, escopo, validacao server-side, storage privado, token publico opaco, expiracao, revogacao, auditoria e notificacoes. A tela publica nao persiste token em storage do navegador e converte HTML de contrato para texto antes de exibir, evitando `innerHTML`.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas avisos LF/CRLF do Git no Windows.
- **Estado atualizado:** Contratos/assinatura agora possuem primeira experiencia React funcional. Ainda faltam validar com API v2/MySQL/backfill reais, Legal Doc Service real, PDF assinado real, tokens legados, usuarios/permissoes reais, HTTPS/homologacao e Playwright.

## Atualizacao complementar 2026-05-15 15:06 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** As rotas React de propostas deixaram de ser placeholders e passaram a consumir o `ProposalsModule`.
- **Frontend React:** Criados `types/proposals.ts`, `schemas/proposal.schema.ts`, `services/proposals.service.ts`, `pages/proposals/ProposalPage.tsx` e `pages/proposals/PublicProposalPage.tsx`. A tela interna cobre filtros, estatisticas, listagem, criacao, edicao, detalhe, envio por link publico e decisoes internas de aprovar/rejeitar por permissao. A tela publica `/onboarding/publico/proposta/:token` consulta o token, exibe dados minimos da proposta sem `innerHTML` e permite aprovar ou solicitar ajuste.
- **Navegacao:** `router.tsx` passou a renderizar `ProposalPage` em `/legalizacao/propostas` e `/legalizacao/propostas/:id`, e `PublicProposalPage` em `/onboarding/publico/proposta/:token`. `navigation.tsx`, `HomePage.tsx` e `LegalizationPage.tsx` ganharam atalhos condicionados a `proposals.read`.
- **Seguranca:** A tela interna usa `proposals.read/create/approve/reject` apenas para experiencia; mutacoes continuam protegidas no backend por JWT, CSRF, escopo, token publico opaco, expiracao, revogacao, auditoria e notificacoes. A tela publica nao persiste token em storage do navegador e renderiza campos como texto.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas avisos LF/CRLF do Git no Windows.
- **Estado atualizado:** Propostas agora possuem primeira experiencia React funcional, incluindo fluxo publico de resposta do cliente. Ainda faltam validar com API v2/MySQL/backfill reais, compatibilidade com tokens legados, usuarios/permissoes reais, Playwright e homologacao HTTPS.

## Atualizacao complementar 2026-05-15 14:13 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/dev` deixou de ser placeholder e passou a consumir `UsersModule`, `RolesModule` e `PermissionsModule`.
- **Frontend React:** Criados `types/admin.ts`, `schemas/admin.schema.ts`, `services/admin.service.ts` e `pages/dev/DevAdminPage.tsx`. A tela cobre abas de usuarios, roles e permissoes, filtros, estatisticas, listagens, criacao, edicao, troca de status de usuario, troca de roles do usuario, troca de permissoes da role e criacao/edicao de permissoes por permissao.
- **Navegacao:** `router.tsx` passou a renderizar `DevAdminPage` em `/dev`, mantendo o menu condicionado a `users.read`.
- **Seguranca:** A tela usa `users.*`, `roles.*` e `permissions.*` apenas para experiencia; leitura e mutacoes continuam protegidas no backend por JWT, permissoes granulares, CSRF, auditoria e protecoes contra perda de acesso administrativo. Dados sao renderizados pelo escape padrao do React, sem `innerHTML`.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. A primeira tentativa de build falhou por tipagem union no helper de formulario de usuario; o tipo comum foi ajustado e a validacao foi reexecutada. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas aviso LF/CRLF do Git no Windows.
- **Estado atualizado:** DEV/admin agora possui primeira experiencia React funcional para administracao de usuarios, roles e permissoes. Ainda faltam validar com API v2/MySQL/seed/backfill reais, usuarios/permissoes reais, migracao de usuarios legados, Playwright e homologacao HTTPS.

## Atualizacao complementar 2026-05-15 13:51 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/onboarding/processos` deixou de ser placeholder e passou a consumir o `OnboardingModule`.
- **Frontend React:** Criados `types/onboarding.ts`, `schemas/onboarding.schema.ts`, `services/onboarding.service.ts` e `pages/onboarding/OnboardingProcessesPage.tsx`. A tela cobre filtros, estatisticas, listagem, selecao de processo, criacao, edicao, arquivamento, mudanca de status/etapa, consulta de timeline e vinculo leve de documentos/requisitos/token publico por permissao.
- **Navegacao:** `router.tsx` passou a renderizar `OnboardingProcessesPage` em `/onboarding/processos`, mantendo menu e atalhos ja condicionados a `onboarding.read`.
- **Seguranca:** A tela usa `onboarding.read/create/update/status/documents/delete` apenas para experiencia; leitura e mutacoes continuam protegidas no backend por JWT, CSRF, escopo por responsavel/criador/departamento, auditoria e validacao server-side. Dados da timeline e metadados sao exibidos sem `innerHTML`.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas aviso LF/CRLF do Git no Windows.
- **Estado atualizado:** Onboarding agora possui primeira experiencia React funcional para processos internos. Ainda faltam validar com API v2/MySQL/backfill reais, usuarios/permissoes reais, fluxos publicos especificos de documentos/proposta, maquina de estados completa, Playwright e homologacao HTTPS.

## Atualizacao complementar 2026-05-15 13:35 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/legalizacao` deixou de ser placeholder e passou a consumir o `LegalizationModule`.
- **Frontend React:** Criados `types/legalization.ts`, `schemas/legalization.schema.ts`, `services/legalization.service.ts` e `pages/legalization/LegalizationPage.tsx`. A tela cobre filtros, paginacao, criacao, edicao, arquivamento, atualizacao de status/etapa, consulta de timeline e gestao de templates de cabecalho/rodape por permissao.
- **Navegacao:** `router.tsx` passou a renderizar `LegalizationPage` em `/legalizacao`.
- **Seguranca:** A tela usa `legalization.read/create/update/status/delete/templates` apenas para experiencia; leitura e mutacoes continuam protegidas no backend por JWT, CSRF, escopo por responsavel/criador/departamento, auditoria e validacao de HTML inseguro nos templates.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas avisos LF/CRLF do Git no Windows.
- **Estado atualizado:** Legalizacao agora possui primeira experiencia React funcional para processos internos e templates. Ainda faltam validar com API v2/MySQL/backfill reais, Legal Doc Service/PDF em homologacao, proposta/contrato integrados, Playwright e fluxo operacional com usuarios reais.

## Atualizacao complementar 2026-05-15 10:24 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/notificacoes` deixou de ser placeholder e passou a consumir o `NotificationsModule`.
- **Frontend React:** Criados `types/notifications.ts`, `schemas/notification.schema.ts`, `services/notifications.service.ts` e `pages/notifications/NotificationsPage.tsx`. A tela cobre filtros por busca, nivel, modulo e nao lidas, paginacao, detalhe de notificacao com metadados, ack, alert-ack, limpeza de nao lidas, criacao manual por permissao e teste operacional de Web Push.
- **Navegacao:** `router.tsx` passou a renderizar `NotificationsPage`; `HomePage.tsx` recebeu atalho de notificacoes condicionado a `notifications.read`.
- **Seguranca:** A tela usa `notifications.read`/`notifications.create` apenas para experiencia; leitura, criacao, ack, clear e teste Web Push seguem protegidos no backend por JWT, CSRF, escopo por usuario/departamento, auditoria e filtro de autoevento.
- **Validacao:** `npm.cmd run lint` e `npm.cmd run build` passaram em `portal-sama-web`. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha.
- **Estado atualizado:** Notificacoes agora possuem primeira experiencia React funcional. Ainda faltam validar em navegador HTTPS/homologacao com API v2/MySQL/VAPID reais, ativacao de Push API no React, SSE/tempo real no frontend, Playwright e estrategia broker/event bus multi-instancia se necessaria.

## Atualizacao complementar 2026-05-15 10:05 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/auditoria` deixou de ser placeholder e passou a consumir o `AuditModule`.
- **Frontend React:** Criados `types/audit.ts`, `schemas/audit.schema.ts`, `services/audit.service.ts` e `pages/audit/AuditPage.tsx`. A tela cobre filtros por acao, modulo, entidade, usuario e periodo, paginacao server-side, estatisticas da pagina e consulta de detalhe de evento com metadados renderizados como texto/JSON.
- **Navegacao:** `router.tsx` passou a renderizar `AuditPage`; `HomePage.tsx` recebeu atalho de auditoria condicionado a `audit.read`.
- **Seguranca:** A tela e somente leitura, consulta apenas com `audit.read` e preserva a protecao real no backend por JWT/RBAC. Metadados sao exibidos sem `innerHTML`, mantendo escape padrao do React.
- **Validacao:** `npm.cmd run lint` e `npm.cmd run build` passaram em `portal-sama-web`. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha.
- **Estado atualizado:** Auditoria agora possui primeira experiencia React funcional para consulta de logs, exportacao CSV e smoke Playwright inicial. Ainda faltam validar com API v2/MySQL/usuarios reais, ampliar Playwright, definir retencao e modelar separadamente backup/restauracao antes de migrar as mutacoes legadas.

## Atualizacao complementar 2026-05-15 09:44 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/solicitacao-acesso` deixou de ser placeholder e passou a consumir o `AccessRequestsModule`.
- **Frontend React:** Criados `types/access-requests.ts`, `schemas/access-request.schema.ts`, `services/access-requests.service.ts` e `pages/access-requests/AccessRequestPage.tsx`. A tela cobre envio de solicitacao por colaborador, envio de solicitacao por gestor para equipe, acompanhamento da ultima solicitacao, filtros de historico e fila de aprovacoes com aprovar/rejeitar por permissao.
- **Navegacao:** `router.tsx` passou a renderizar `AccessRequestPage`; `navigation.tsx` libera o menu de solicitacoes para sessoes autenticadas porque `POST /api-v2/access-requests` e `GET /api-v2/access-requests/my/latest` nao exigem `access_requests.read`; `HomePage.tsx` recebeu o mesmo ajuste e corrigiu o atalho de documentos para `/documentos`.
- **Seguranca:** A tela usa permissoes apenas para UX; criacao e decisao continuam protegidas no backend por JWT, CSRF, escopo por solicitante/gestor/departamento, `access_requests.*`, auditoria e notificacoes.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram. O build Vite emitiu aviso de chunk acima de 500 kB, sem falha; o diff final teve apenas avisos LF/CRLF do Git no Windows.
- **Estado atualizado:** Solicitacoes de acesso agora possuem primeira experiencia React funcional. Ainda faltam validar com API v2/MySQL/backfill/usuarios reais, testar escopo por gestor/departamento/TI, criar Playwright e modelar separadamente a base editavel de acessos internos da TI com criptografia/vault antes de migrar `/ti/acessos`.

## Atualizacao complementar 2026-05-15 09:16 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** O painel React `/clientes/:clientId/painel` ganhou a primeira integracao com documentos do cliente via `DocumentsModule`.
- **Frontend React:** Criados `types/documents.ts`, `schemas/document.schema.ts`, `services/documents.service.ts` e `ClientDocumentsPanel.tsx`. A tela cobre checklist por cliente, estatisticas, envio multipart, download protegido, revisao de status e requisito customizado por permissao.
- **Seguranca:** A secao de documentos so consulta quando existe `documents.read`; upload, download, revisao e requisitos respeitam `documents.upload`, `documents.download`, `documents.review` e `documents.requirements` na UX, mantendo JWT, CSRF, storage privado, scanner/quarentena, escopo e auditoria no backend.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` e `git diff --check` passaram. A primeira execucao de lint teve um aviso de hook no checklist; o fallback foi estabilizado com `useMemo` e a validacao foi reexecutada sem avisos.
- **Estado atualizado:** O painel do cliente ja possui base funcional para documentos. Ainda faltam validar upload/download/revisao com API v2/MySQL/storage/ClamAV reais, escopo por cliente/gestor, Playwright e as demais abas do painel.

## Atualizacao complementar 2026-05-15 08:59 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** As rotas React `/clientes` e `/clientes/:clientId/painel` deixaram de ser placeholders e passaram a consumir o `ClientsModule`.
- **Frontend React:** Criados tipos, schema Zod, service Axios/TanStack Query e telas `ClientsPage.tsx`/`ClientDashboardPage.tsx` com filtros, paginacao, cadastro, edicao, arquivamento logico, painel de resumo e detalhes cadastrais.
- **Seguranca:** A UI usa permissoes `clients.read/create/update/delete` apenas para experiencia; as mutacoes continuam protegidas no backend por JWT, CSRF, permissoes granulares, escopo e auditoria. Dados de cliente nao sao persistidos em `localStorage`.
- **Validacao:** `npm.cmd run lint`, `npm.cmd audit --audit-level=moderate` e `git diff --check` passaram. A primeira tentativa de `npm.cmd run build` falhou por incompatibilidade de tipo no schema Zod/React Hook Form; o schema foi ajustado e o build passou em seguida.
- **Estado atualizado:** Clientes agora possuem primeira experiencia React funcional. Ainda faltam validar com API v2/MySQL reais, backfill legado, vinculos cliente-usuario/gestor, documentos do painel e Playwright.

## Atualizacao complementar 2026-05-15 08:33 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** A rota React `/certificados-digitais` deixou de ser placeholder e passou a ter `CertificatesPage.tsx` integrada ao contrato `/api-v2/certificates`.
- **Frontend React:** Criados tipos, schemas Zod, service Axios/TanStack Query e tela com filtros, estatisticas, listagem, cadastro multipart `.p12/.pfx`, download protegido, edicao de metadados/arquivo, rotacao de senha e remocao logica por permissao.
- **Seguranca:** A tela usa `PermissionGate` apenas para UX; as acoes sensiveis continuam protegidas no backend por JWT, permissoes `certificates.*`, CSRF, storage privado, criptografia de senha e auditoria. A senha nao e exibida na listagem nem persistida no frontend.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram.
- **Estado atualizado:** Certificados digitais agora possuem primeira tela React funcional. Ainda faltam validar login/fluxo completo com API v2, MySQL/storage reais, dados de homologacao, backfill legado e Playwright.

## Atualizacao complementar 2026-05-14 14:54 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Criada a fundacao `portal-sama-web` em React + TypeScript + Vite, conforme a etapa frontend registrada na matriz HTML -> React.
- **Frontend React:** O app novo inclui roteamento com `react-router-dom`, providers do TanStack Query, cliente Axios centralizado, store Zustand de autenticacao, login em `/login`, layout interno autenticado, menu com `PermissionGate` e rotas reservadas para as telas prioritarias mapeadas.
- **Autenticacao/seguranca:** A tela React chama `/api-v2/auth/csrf`, `/login`, `/refresh`, `/logout`, `/me` e `/forgot-password`. O access token fica somente em memoria no Zustand; o refresh continua em cookie HttpOnly da API v2. O Vite possui proxy local de `/api-v2` para `http://localhost:3000`.
- **Deploy:** Adicionados `portal-sama-web/Dockerfile`, `nginx.conf` e `.env.example` com `VITE_API_URL`/`VITE_API_PROXY_TARGET`.
- **Validacao:** `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` em `portal-sama-web` e `git diff --check` passaram.
- **Estado atualizado:** Frontend React saiu de nao iniciado para primeira fundacao funcional e possui base Playwright inicial. Ainda faltam ligar telas reais aos modulos da API v2, testar login com MySQL/homologacao, validar fluxo em navegador HTTPS e ampliar Playwright.

## Atualizacao complementar 2026-05-14 13:54 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Adicionada renderizacao server-side de PDF de contratos na API v2 em `POST /api-v2/contracts/:id/render-pdf`, reaproveitando o desenho legado do `LEGAL_DOC_SERVICE_URL` em `/render/html-to-pdf`.
- **Seguranca/operacao:** O renderizador fica desligado por padrao e exige `LEGAL_DOC_SERVICE_ENABLED=true`, `LEGAL_RENDER_V2_ENABLED=true`, URL e credencial. O PDF retornado em base64 e validado como PDF, passa pelo mesmo bloqueio de conteudo ativo conhecido, e e gravado em storage privado sem expor `storageKey`.
- **Contratos:** A rota renderiza o HTML salvo do contrato, com header/footer e layout controlado, atualiza metadados `generation`/`pdf`, registra auditoria `contracts.render_pdf` com hash/tamanho/engine/paginas e preserva o download protegido em `/api-v2/contracts/:id/pdf`.
- **Validacao:** `npm.cmd test -- contracts.service.spec.ts contract-pdf-file-validator.service.spec.ts contract-pdf-render.service.spec.ts --runInBand`, `npm.cmd exec tsc -- --noEmit`, `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e -- --runInBand` e `npm.cmd test -- --runInBand` passaram.
- **Estado atualizado:** Renderizacao HTML para PDF esta integrada no codigo, mas ainda precisa de um Legal Doc Service real em HTTPS/homologacao, teste de fidelidade visual, backfill legado e frontend consumindo a rota.

## Atualizacao complementar 2026-05-14 13:42 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Adicionada primeira fatia segura de PDF em contratos: importacao multipart em `POST /api-v2/contracts/:id/pdf` e download protegido em `GET /api-v2/contracts/:id/pdf`.
- **Seguranca/storage:** O upload aceita apenas PDF por extensao, MIME permitido, assinatura `%PDF`, limite configuravel e bloqueio inicial de marcadores ativos como JavaScript/OpenAction/Launch/EmbeddedFile. O arquivo fica em storage privado, a resposta expoe somente `pdfDownloadUrl`/`pdf_download_url` e a auditoria registra hash/tamanho sem `storageKey`.
- **Contratos:** A importacao atualiza `pdfStorageKey`, metadados do PDF, `generatedAt` quando necessario e status `DRAFT` para `GENERATED`; contratos cancelados/arquivados nao aceitam novo PDF. Download exige `contracts.read`, escopo do contrato e retorna `application/pdf` com `Cache-Control: no-store`.
- **Validacao:** `npm.cmd test -- contracts.service.spec.ts contract-pdf-file-validator.service.spec.ts --runInBand`, `npm.cmd exec tsc -- --noEmit`, `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e -- --runInBand` e `npm.cmd test -- --runInBand` passaram.
- **Estado atualizado:** A pendencia de importacao/download seguro de PDF em contratos ficou parcialmente enderecada. Ainda faltam escolher/validar renderizador HTML para PDF em sandbox, backfill legado, integracao frontend e validacao com MySQL/storage reais em homologacao.

## Atualizacao complementar 2026-05-14 10:39 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Adicionado teste operacional de Web Push para homologacao. A API v2 ganhou `POST /api-v2/notifications/push/test`, protegido por JWT `notifications.read` e CSRF, para criar uma notificacao de teste ao usuario autenticado e retornar resumo de despacho Web Push (`enabled`, `attempted`, `sent`, `failed`).
- **Frontend legado:** O painel de notificacoes em `global.js` ganhou o botao `Testar` apos o opt-in de notificacoes do sistema; ele garante sessao API v2, CSRF, assinatura Push API e chama o endpoint de teste. `global.css` recebeu o estilo do novo botao.
- **Seguranca/operacao:** O teste nao cria notificacao quando VAPID esta desabilitado, registra auditoria `notifications.push_test` quando dispara, usa ator tecnico `portal-sama` para nao ser filtrado como autoevento e nao expõe endpoint/keys de assinatura.
- **Validacao:** `node --check global.js`, `npm.cmd test -- notifications.service.spec.ts notifications-push.service.spec.ts --runInBand`, `npm.cmd exec tsc -- --noEmit`, `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e -- --runInBand` e `npm.cmd test -- --runInBand` passaram.
- **Estado atualizado:** A homologacao Web Push agora tem um acionador interno para testar o navegador assinado. Ainda faltam configurar VAPID real em HTTPS/homologacao, validar navegadores/portal fechado e decidir broker/event bus para multi-instancia se necessario.

## Atualizacao complementar 2026-05-14 10:25 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Concluida a cobertura inicial de emissores automaticos nos modulos de negocio pendentes: `DocumentsModule` e `CertificatesModule` agora emitem notificacoes internas em eventos sensiveis.
- **Documentos:** notificacoes em upload interno, upload publico por token, revisao de status, arquivamento, requisito customizado criado, token publico criado e token publico revogado. Os payloads evitam token bruto, storage key e conteudo de arquivo.
- **Certificados:** notificacoes em cadastro, atualizacao, rotacao de senha e remocao de certificado digital. Os payloads evitam senha bruta, senha criptografada e storage key.
- **Validacao:** `npm.cmd test -- documents.service.spec.ts certificates.service.spec.ts --runInBand`, `npm.cmd exec tsc -- --noEmit`, `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e -- --runInBand`, `npm.cmd test -- --runInBand` e `git diff --check` passaram.
- **Estado atualizado:** Emissores automaticos iniciais existem para access requests, propostas, contratos, onboarding, legalizacao, documentos e certificados. A principal pendencia desta frente passa a ser validacao manual Web Push em HTTPS/homologacao com VAPID real, alem de estrategia de broker/event bus multi-instancia se exigida.

## Atualizacao complementar 2026-05-14 10:13 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Ampliados os emissores automaticos de notificacoes para `OnboardingModule` e `LegalizationModule`. Criacao, mudanca de status, arquivamento e, no onboarding, vinculacao de documentos passam a emitir notificacao interna para o responsavel do processo, com fallback para o departamento do processo.
- **Escopo tecnico:** `OnboardingModule` e `LegalizationModule` importam `NotificationsModule`; os services usam o ator real com permissao temporaria `notifications.create` e papel tecnico `DEV`, preservando o bloqueio de auto-notificacao quando o responsavel e o proprio ator.
- **Validacao:** `npm.cmd test -- onboarding.service.spec.ts legalization.service.spec.ts --runInBand`, `npm.cmd exec tsc -- --noEmit`, `npm.cmd run lint`, `npm.cmd run build` e `npm.cmd run test:e2e -- --runInBand` passaram.
- **Estado atualizado:** A pendencia de emissores dos modulos de negocio foi reduzida novamente: access requests, propostas, contratos, onboarding e legalizacao agora disparam notificacoes internas. Ainda faltam emissores em documentos/certificados e validacao Web Push manual em HTTPS com VAPID real.

## Atualizacao complementar 2026-05-14 10:04 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Ligados emissores automaticos de notificacoes nos retornos publicos de propostas e contratos. Quando o cliente aprova/solicita ajuste de proposta por token publico ou assina contrato por token publico, a API agora cria notificacao interna para o responsavel encontrado pelo `updatedById/createdById`, com fallback para o departamento `Legalizacao`.
- **Escopo tecnico:** `ProposalsModule` e `ContractsModule` passaram a importar `NotificationsModule`; os services emitem notificacoes com ator tecnico isolado, dedupe por token publico e falha nao bloqueante para preservar o fluxo principal do cliente.
- **Validacao:** `npm.cmd test -- proposals.service.spec.ts contracts.service.spec.ts --runInBand`, `npm.cmd exec tsc -- --noEmit`, `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e -- --runInBand` e `git diff --check` passaram.
- **Estado atualizado:** A pendencia de emissores dos demais modulos foi parcialmente reduzida: propostas e contratos publicos ja avisam responsaveis internos; ainda faltam emissores em onboarding/legalizacao/documentos/certificados, teste manual de Web Push em HTTPS e configuracao VAPID real.

## Atualizacao complementar 2026-05-14 09:49 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Fortalecido o ciclo de sessao API v2 no frontend legado para Web Push: `auth.js` agora guarda token/expiracao/CSRF da API v2 separados do CSRF PHP, tenta refresh em `/api-v2/auth/refresh`, faz logout coordenado em `/api-v2/auth/logout` e expoe helpers de headers para chamadas API v2.
- **Frontend legado:** `Login.js` passa a guardar `expiresIn` e CSRF retornado pelo login API v2; `global.js` tenta renovar a sessao API v2 antes de assinar Web Push e refaz chamadas que voltarem `401` apos refresh.
- **Operacao:** Adicionado script `npm run webpush:vapid:generate` em `portal-sama-api` para gerar placeholders VAPID sem colocar segredo real no repositorio.
- **Validacao:** `node --check auth.js`, `node --check Login.js`, `node --check global.js`, `node --check portal-sama-api/scripts/generate-vapid-keys.js`, `npm.cmd run lint` e `npm.cmd run build` passaram.
- **Estado atualizado:** A pendencia de refresh/bearer no legado ficou parcialmente enderecada para o uso de Web Push. Ainda faltam teste manual em navegador, refresh React definitivo, VAPID real em ambiente e validação de recebimento com portal fechado.

## Atualizacao complementar 2026-05-14 09:35 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Implementada a base de Web Push para notificacoes com portal fechado. A API v2 ganhou rotas `GET /api-v2/notifications/push/public-key`, `POST /api-v2/notifications/push/subscribe` e `POST /api-v2/notifications/push/unsubscribe`, persistencia de assinaturas por navegador e envio Web Push ao criar notificacoes.
- **Frontend legado:** Adicionado `sama-push-sw.js`; `global.js` agora registra Service Worker, busca chave publica VAPID, assina o navegador via Push API e tenta cadastrar a assinatura na API v2 quando o usuario ativa notificacoes do sistema. `Login.js` tenta obter access token da API v2 em paralelo ao login legado, sem trocar o fluxo PHP principal.
- **Banco/seguranca:** Criado modelo `BrowserPushSubscription` e migrations `20260514092000_add_browser_push_subscriptions` + `20260514093500_align_browser_push_updated_at`; `.env.example` recebeu `WEB_PUSH_PUBLIC_KEY`, `WEB_PUSH_PRIVATE_KEY` e `WEB_PUSH_SUBJECT`. O cookie CSRF da API v2 passou a usar path `/api-v2`, permitindo mutacoes fora de `/auth`.
- **Validacao:** `node --check` em `global.js`, `auth.js`, `Login.js` e `sama-push-sw.js` passou; `prisma format/generate/validate`, testes focados de notificacoes, lint, e2e com 130 testes, suite completa com 23 suites/131 testes, build, audit, `prisma:migrate:deploy/status` e diff Prisma passaram.
- **Estado atualizado:** Web Push esta tecnicamente preparado para notificacoes com portal fechado quando VAPID estiver configurado e o usuario tiver permissao/assinatura ativa. Ainda faltam teste manual em HTTPS/homologacao, configurar VAPID em ambiente real, validar browsers suportados e ligar emissores dos demais modulos.

## Atualizacao complementar 2026-05-14 09:09 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** O frontend legado em `global.js` ganhou controle de permissao para notificacoes nativas do navegador no painel de notificacoes. O usuario pode ativar/desativar alertas do sistema e, quando o portal esta aberto mas fora de foco/aba oculta, novas notificacoes tambem aparecem no computador via `Notification`.
- **Escopo:** Esta etapa cobre notificacao nativa enquanto alguma aba do portal esta aberta. Ainda nao e Web Push com Service Worker para portal totalmente fechado.
- **Validacao:** `node --check global.js` passou. A suite backend anterior permanece verde e nao foi alterada nesta etapa.
- **Estado atualizado:** O legado agora possui permissao opt-in por usuario no navegador, persistida em `localStorage`, com fallback seguro quando a API de notificacao nao esta disponivel ou esta bloqueada.

## Atualizacao complementar 2026-05-14 08:40 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** O stream `GET /api-v2/notifications/stream` passou a aceitar `last_id`/`lastId` e buscar periodicamente no MySQL notificacoes com id maior que o cursor, alem dos eventos em memoria. Isso aproxima a API v2 do comportamento do `api/notifications_stream.php` legado e melhora reconexao/restart.
- **Seguranca/escopo:** O polling do stream reutiliza o mesmo escopo por usuario/departamento, filtros da listagem e filtro de autoevento. Eventos ja entregues pelo snapshot/live sao deduplicados para evitar repeticao.
- **Validacao:** `notifications.service.spec.ts` passou com 10 casos, `tsc --noEmit`, lint, e2e com 125 testes, suite completa com 22 suites/126 testes, build e audit passaram.
- **Estado atualizado:** Stream agora cobre snapshot, live events da instancia e catch-up pelo banco para novas notificacoes criadas. Ainda faltam validar em navegador/frontend, definir estrategia para eventos de update em multi-instancia e integrar as telas.

## Atualizacao complementar 2026-05-14 08:30 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Estendido `NotificationsModule` com `GET /api-v2/notifications/stream` via SSE, snapshot inicial, ping periodico e eventos de criacao/atualizacao/limpeza. `AccessRequestsModule` passou a emitir notificacoes automaticas para gestor, solicitante e TI nos fluxos de criacao, aprovacao e rejeicao.
- **Seguranca:** O stream exige JWT e `notifications.read`, reaproveita o escopo por usuario/departamento, respeita filtros da listagem e nao entrega eventos do proprio ator. Emissores internos usam `NotificationsService.create`, dedupe, auditoria e falha isolada para nao bloquear a solicitacao principal.
- **Validacao:** `tsc --noEmit`, testes focados de `access-requests` + `notifications` com 14 casos, `lint`, e2e com 125 testes, suite completa com 22 suites/125 testes, build, audit e `git diff --check` passaram.
- **Estado atualizado:** SSE inicial e primeiro emissor automatico de negocio foram implementados. Ainda faltam broker/event bus distribuido para multi-instancia, emissores nos demais modulos, frontend React/legado e validacao em homologacao/producao.

## Atualizacao complementar 2026-05-13 16:34 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Criado `NotificationsModule` com `GET/POST /api-v2/notifications`, `POST /api-v2/notifications/:id/ack`, `POST /api-v2/notifications/:id/alert-ack` e `POST /api-v2/notifications/clear-unread`.
- **Banco:** Adicionado modelo Prisma `Notification` mapeado para a tabela legada `notifications`, com migration compativel `20260513170000_add_notifications_model`, aplicada no MySQL local por `prisma:migrate:deploy`.
- **Seguranca:** Criadas permissoes `notifications.read/create`; leitura e ack aplicam escopo por usuario/departamento, criacao respeita departamento permitido, mutacoes exigem CSRF e eventos sao auditados.
- **Validacao:** `prisma format/generate/validate`, `tsc --noEmit`, `notifications.service.spec.ts` com 7 casos, RBAC focado, lint, e2e com 123 testes, `prisma:migrate:deploy/status`, diff Prisma, seed, 22 suites/123 unitarios, build, audit e `git diff --check` passaram.
- **Estado atualizado:** Notificacoes internas agora possuem primeira API v2. Ainda faltam SSE/stream em NestJS, integracao automatica dos modulos de negocio emitindo notificacoes, frontend React/legado e validacao em homologacao/producao.

## Atualizacao complementar 2026-05-13 16:16 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Estendido `LegalizationModule` com templates de cabecalho/rodape em `GET/POST /api-v2/legalization/templates`, `PATCH /api-v2/legalization/templates/:id` e `DELETE /api-v2/legalization/templates/:id`.
- **Banco:** Adicionados enum `LegalizationTemplateType`, modelo `LegalizationTemplate`, tabela `legalization_templates` e migration `20260513164000_add_legalization_templates`, aplicada no MySQL local por `prisma:migrate:deploy`.
- **Seguranca:** Criada permissao `legalization.templates`; leitura usa `legalization.read`, mutacoes exigem CSRF, permissao granular, auditoria e rejeitam HTML com scripts, handlers inline, iframe/object/embed/link/meta e URLs `javascript:`.
- **Validacao:** `prisma format/generate/validate`, `tsc --noEmit`, teste focado de legalizacao com 10 casos, RBAC focado, e2e com 117 testes, lint, `prisma:migrate:deploy/status`, diff Prisma, seed, 21 suites/116 unitarios, build, audit e `git diff --check` passaram.
- **Estado atualizado:** Legalizacao cobre processos internos e templates no backend e ja possui primeira tela React em `/legalizacao`. Ainda faltam backfill legado, validacao operacional de PDF/Legal Doc Service, integracao fim a fim com propostas/contratos, Playwright e validacao em homologacao/producao.

## Atualizacao complementar 2026-05-13 15:59 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Criado `LegalizationModule` inicial com rotas `GET/POST /api-v2/legalization/processes`, `GET/PATCH/DELETE /api-v2/legalization/processes/:id`, `PATCH /api-v2/legalization/processes/:id/status` e `GET /api-v2/legalization/processes/:id/timeline`.
- **Banco:** Adicionados enum `LegalizationStatus`, modelo `LegalizationProcess`, migration `20260513160000_add_legalization_processes` e migration complementar `20260513160500_add_missing_relation_indexes`; ambas aplicadas no MySQL local por `prisma:migrate:deploy`.
- **Seguranca:** Criadas permissoes `legalization.read/create/update/status/delete`; mutacoes exigem CSRF, permissoes granulares, escopo por responsavel/criador/departamento e auditoria.
- **Validacao:** `prisma format/generate/validate`, `tsc --noEmit`, teste focado de legalizacao, RBAC focado, lint, `prisma:migrate:deploy/status`, seed, diff Prisma, 21 suites/112 unitarios, 1 suite/112 e2e, build, audit e `git diff --check` passaram.
- **Estado atualizado:** Backend NestJS agora inclui `LegalizationModule` em primeira fatia. Ainda faltam backfill legado, templates de cabecalho/rodape, importacao/renderizacao PDF segura, integracao com propostas/contratos em fluxo completo, notificacoes e frontend React.

## Atualizacao complementar 2026-05-13 15:18 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Criado `OnboardingModule` inicial com rotas internas de processos, detalhe, criacao, atualizacao, mudanca de etapa/status, timeline, vinculo leve de documentos e arquivamento logico.
- **Banco:** Adicionados enum `OnboardingStatus`, modelo `OnboardingProcess` e migration `20260513152000_add_onboarding_processes`, aplicada no MySQL local por `prisma:migrate:deploy`.
- **Seguranca:** Criadas permissoes `onboarding.read/create/update/status/documents/delete`; mutacoes exigem CSRF, permissoes granulares, escopo por responsavel/criador/departamento e auditoria.
- **Deploy/subdominio:** Criado `docs/INCIDENTE_SUBDOMINIO_PORTAL_SAMACONTABIL.md`; DNS do subdominio resolve para EasyPanel, mas HTTPS retornou 500 e o alvo interno customizado parece usar `https://...:80` em vez de `http://...:80`.
- **Validacao:** `format`, `prisma generate/validate`, `tsc --noEmit`, lint, teste focado de onboarding, RBAC focado, `prisma:migrate:deploy/status/diff`, seed, 20 suites/106 unitarios, 1 suite/102 e2e, build, audit e `git diff --check` passaram.
- **Estado atualizado:** Backend NestJS agora inclui `OnboardingModule` em primeira fatia. Ainda faltam fluxo publico especifico de onboarding, maquina de estados completa, notificacoes, backfill legado e frontend React.

## Atualizacao complementar 2026-05-13 14:43 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Criado `CollaboratorsModule` inicial com rotas `GET/POST /api-v2/collaborators`, `GET/PATCH/DELETE /api-v2/collaborators/:id`, permissoes `collaborators.*`, CSRF em mutacoes, auditoria e escopo de leitura por departamento.
- **Banco:** O modelo `User` ganhou campos funcionais de colaborador (`position`, `phone`, `extension`, `metadata`, `archivedAt`, `archivedById`) e a migration `20260513124500_add_collaborator_profile_to_users` foi aplicada no MySQL local por `prisma:migrate:deploy`.
- **Validacao:** `prisma format/generate/validate`, `tsc --noEmit`, teste focado de colaboradores, lint, RBAC focado, `prisma:migrate:deploy`, `prisma:migrate:status`, `prisma migrate diff`, seed, 19 suites/99 unitarios, 1 suite/91 e2e, build, `npm audit` e `git diff --check` passaram.
- **Observacao operacional:** A primeira tentativa de migration falhou porque Docker/MySQL local estavam parados; Docker Desktop e o container `portal-sama-mysql` foram iniciados, e os comandos Prisma foram reexecutados com sucesso.
- **Estado atualizado:** Backend NestJS agora inclui `CollaboratorsModule` em primeira fatia. Ainda faltam backfill de `sama_colaboradores`, vinculos colaborador-cliente/gestor, transferencias/carteira e integracao frontend.

## Atualizacao complementar 2026-05-13 10:50 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Resolvido o ponto pendente de baseline do MySQL local: o schema foi sincronizado de forma controlada, as 5 migrations antigas foram marcadas como aplicadas, a nova migration `20260513113000_expand_clients_profile` foi aplicada por `prisma migrate deploy` e o banco voltou a ficar `Database schema is up to date`.
- **Implementacao:** Criado `ClientsModule` com `GET/POST /api-v2/clients`, `GET/PATCH/DELETE /api-v2/clients/:id` e `GET /api-v2/clients/:id/dashboard`; o modelo `Client` ganhou campos operacionais do legado (`rank`, regime, id empresa, atividade, telefone, endereco, grupo, metadata e soft delete).
- **Validacao:** `prisma format`, `prisma generate`, `prisma validate`, `prisma:migrate:deploy`, `prisma:migrate:status`, `prisma migrate diff`, seed, `tsc --noEmit`, lint, teste focado, 18 suites/93 testes unitarios, 1 suite/84 e2e, build, `npm audit` e `git diff --check` passaram.
- **Estado atualizado:** Backend NestJS agora inclui `ClientsModule` em primeira fatia e o MySQL local esta alinhado ao historico Prisma atual.

## Atualizacao complementar 2026-05-13 10:15 -03:00

- **Responsavel/IA:** Codex
- **Resumo da alteracao:** Criado `AccessRequestsModule` inicial com rotas de solicitacao, listagem, ultima solicitacao do colaborador, aprovacoes do gestor e decisao aprovar/rejeitar; adicionados modelo/migration Prisma `AccessRequest`, enums de perfil/status, testes unitarios/e2e e documentacao.
- **Validacao:** `prisma format`, `prisma generate`, `prisma validate`, `tsc --noEmit`, lint, 5 testes focados, 17 suites/88 testes unitarios, 1 suite/75 e2e, build, `npm audit` e `git diff --check` passaram.
- **Pendencia critica:** `prisma:migrate:status` ainda falha porque o MySQL local `portal_sama` tem 5 migrations pendentes e tabelas fora do historico Prisma; nao foi aplicado `migrate deploy`.
- **Estado atualizado:** Backend NestJS agora inclui `ProposalsModule`, `ContractsModule` e `AccessRequestsModule` em primeira fatia. Frontend React segue nao iniciado.

## Última atualização

- **Data/hora:** 2026-05-13 09:34 -03:00
- **Responsável/IA:** Codex
- **Resumo da alteração:** Criado `ContractsModule` inicial com CRUD/snapshot HTML, envio para assinatura por token público opaco, assinatura pública, modelo/migration Prisma `Contract` e testes unitários/e2e; migrations reais seguem pendentes porque o MySQL local está fora do histórico Prisma.

---

## Estado geral

| Área | Status | Observações |
|---|---|---|
| Backend NestJS | Em andamento | `portal-sama-api/` criado com NestJS, `/api-v2`, `HealthModule`, `AuthModule`, `AuditModule`, `UsersModule`, `RolesModule`, `PermissionsModule`, `ClientsModule`, `CollaboratorsModule`, `OnboardingModule`, `LegalizationModule`, `NotificationsModule`, `DocumentsModule`, `CertificatesModule`, `ProposalsModule`, `ContractsModule`, filtro de erros, request id, Swagger fora de produção, Helmet, CORS, throttling e `ValidationPipe`. Módulos de negócio ainda estão em migração gradual. |
| Frontend React | Parcial | `portal-sama-web/` criado com React, TypeScript, Vite, Tailwind, React Router, Axios, TanStack Query, Zustand, React Hook Form/Zod, login `/login`, layout autenticado, telas iniciais de certificados, clientes, clientes departamentais, colaboradores, documentos internos/no painel/publicos, solicitacoes, TI operacional, auditoria, notificacoes, legalizacao, onboarding/processos/entrada, DEV/admin, propostas interna/publica e contratos/assinatura. Playwright inicial cobre login, documentos publicos e auditoria com mocks; ainda falta integrar telas gerenciais/contabeis, validar em navegador/homologacao e ampliar E2E. |
| Banco Prisma/MySQL | Parcial | ✅ MySQL local foi baselined em 2026-05-13 10:50 e recebeu as migrations ate `20260514093500_align_browser_push_updated_at` em 2026-05-14 09:35; `prisma:migrate:status` retornou `Database schema is up to date` e `prisma migrate diff --from-schema-datasource --to-schema-datamodel` nao encontrou diferencas. Ainda faltam backfills/dados reais e validacao em homologacao/producao. |
| Autenticação | Parcial | Existe autenticação PHP por sessão no legado. A base NestJS existe em `/api-v2/auth/*` com JWT de acesso, refresh cookie HttpOnly e CSRF double-submit assinado para `login`, `refresh` e `logout`; o frontend React ja consome esse fluxo e `smoke:auth` foi disponibilizado para validar em HTTPS. Ainda falta rodar com MySQL/usuario real por perfil. |
| RBAC/Permissões | Parcial | ✅ Seed RBAC executado em MySQL com 9 roles padrão (`ADMIN`, `MANAGER`, `CLIENT`, `DEPARTMENT`, `TI`, `ACCOUNTING`, `LEGALIZATION`, `AUDITOR`, `DEV`) e 30+ permissões. A API NestJS possui decorator/guard de permissões, JWT com roles/permissões, endpoints `GET /api-v2/users`, `/roles` e `/permissions` protegidos por JWT. Permissões incluem `notifications.read/create`, `documents.requirements`, `documents.review`, `documents.public_tokens`, `audit.read`, `users.*`, `roles.*`, `permissions.*` e permissões de recurso granulares. Mutações administrativas de usuários, roles e permissões foram implementadas com CSRF/auditoria; validação real com MySQL, migração de usuários legados e escopo por recurso ainda estão pendentes. |
| Upload seguro | Parcial | `api/client_documents_lib.php` possui allowlist, MIME, assinatura, quarentena e scanner configurável. `DocumentsModule` agora valida extensão/MIME/assinatura/tamanho/hash SHA-256, grava o arquivo primeiro em quarentena privada, executa regras estaticas e scanner ClamAV/clamdscan configuravel antes do storage final, exige CSRF em upload autenticado, aceita upload público por token validado e possui emissão/listagem/revogação administrativa de tokens públicos com auditoria. Ainda falta validar ClamAV real/EICAR, modo `strict`, storage persistente e MySQL reais em homologacao. |
| Auditoria | Parcial | Existem `audit_log`, eventos e actions em `api/storage.php`; `AuditModule` NestJS foi iniciado com gravacao centralizada, mascaramento de metadados sensiveis, consulta protegida por `audit.read` e exportacao CSV auditada. A tela React `/auditoria` consome listagem/detalhe/exportacao com filtros e possui smoke Playwright inicial; ainda falta validar com MySQL real/usuarios reais, ampliar Playwright e definir retencao. |
| Testes | Parcial | ✅ Suite completa passou com MySQL real em 2026-05-11 14:45. Em 2026-05-14 10:39, notificacoes/Web Push foram revalidadas com testes focados, `tsc --noEmit`, lint, build, e2e com 132 testes e suite completa com 23 suites/137 unitarios. Lint PHP segue bloqueado por ausencia de `php` no PATH. |
| Ambiente local TypeScript | Parcial | ✅ Em 2026-05-13 09:15, `portal-sama-api/tsconfig.json` passou a incluir explicitamente tipos `node`/`jest` e arquivos `src`, `test` e `prisma`; `.vscode/settings.json` aponta para o TypeScript local da API. Em 2026-05-13 09:20, `baseUrl` foi removido para eliminar alerta de depreciação do TypeScript 6. `tsc --noEmit`, lint, build e teste focado passaram. No PowerShell, usar `npm.cmd`, pois `npm.ps1` é bloqueado pela política local. |
| Deploy EasyPanel | Parcial avancado | Nova stack ja esta publicada no EasyPanel com `portal-sama-api`, `portal-sama-database` e `portal-sama-web`; `https://portal.samacontabil.com.br/` responde HTTP 200 e o smoke publico validou frontend, `/api-v2/health`, CORS e CSRF. Ainda faltam backups/rollback, ClamAV/storage em fluxo real, matriz de permissoes, Playwright real e QA final. |

---

## O que foi feito

- 2026-05-08 09:10 -03:00: Lido o prompt mestre em `docs/INSTRUÇÕEDS_AI.MD`.
- 2026-05-08 09:10 -03:00: Lidas as documentações existentes em `docs/` relevantes para a auditoria inicial.
- 2026-05-08 09:10 -03:00: Criados arquivos obrigatórios de acompanhamento que estavam ausentes.
- 2026-05-08 09:10 -03:00: Criada a pasta local `.ai-tests/` com `README.md`.
- 2026-05-08 09:10 -03:00: Atualizado `.gitignore` para ignorar `.ai-tests/` e permitir versionamento de `docs/`.
- 2026-05-08 09:10 -03:00: Verificada a estrutura real do projeto, endpoints PHP, páginas HTML, scripts de apoio, banco legado e disponibilidade local de ferramentas.
- 2026-05-08 09:31 -03:00: Criado `.env.example` com placeholders, sem leitura nem exposição do `.env` real.
- 2026-05-08 09:31 -03:00: Atualizado `.gitignore` para permitir versionamento de `.env.example` mantendo `.env` e `.env.*` ignorados.
- 2026-05-08 09:31 -03:00: Removido `display_errors` incondicional de `SolicitacaoAcesso/enviar_acesso.php` e `SolicitacaoAcesso/verificar_codigo.php`; debug agora depende de `SAMA_DEBUG`, `APP_DEBUG` ou `SAMA_DISPLAY_ERRORS`.
- 2026-05-08 09:31 -03:00: Executadas validações disponíveis e registrados artefatos em `.ai-tests/runs/2026-05-08-09-31-04/`.
- 2026-05-08 10:04 -03:00: Criada a fundação `portal-sama-api` em NestJS/TypeScript.
- 2026-05-08 10:04 -03:00: Configurados `/api-v2`, Helmet, CORS, throttling, `ValidationPipe`, filtro global de exceções, request id e Swagger fora de produção.
- 2026-05-08 10:04 -03:00: Criados `ConfigModule` com validação Zod, `.env.example` da API, `DatabaseModule`, `PrismaService` e `prisma/schema.prisma` com provider MySQL.
- 2026-05-08 10:04 -03:00: Criado `HealthModule` com `GET /api-v2/health`.
- 2026-05-08 10:04 -03:00: Instaladas dependências Node da API e gerado `portal-sama-api/package-lock.json`.
- 2026-05-08 10:04 -03:00: Executados lint, testes unitários, e2e, build, Prisma generate/validate e npm audit da API.
- 2026-05-08 10:25 -03:00: Criado `AuthModule` em `portal-sama-api` com `/api-v2/auth/login`, `/refresh`, `/logout`, `/me` e `/forgot-password`.
- 2026-05-08 10:25 -03:00: Implementado access JWT, refresh token em cookie HttpOnly, HMAC-SHA-256 do refresh token em banco, rotação/revogação e auditoria de eventos de autenticação.
- 2026-05-08 10:25 -03:00: Adicionado índice único em `RefreshToken.tokenHash` no Prisma e testes unitários/e2e para autenticação.
- 2026-05-08 10:25 -03:00: Instaladas dependências `@nestjs/jwt`, `bcrypt` e `@types/bcrypt`.
- 2026-05-08 10:25 -03:00: Corrigido script `format` da API para combinar Prettier em TypeScript com `prisma format`.
- 2026-05-08 14:14 -03:00: Criado `CsrfService` com token HMAC double-submit para o `AuthModule`.
- 2026-05-08 14:14 -03:00: Criado `GET /api-v2/auth/csrf` e CSRF obrigatório em `POST /api-v2/auth/login`, `/refresh` e `/logout`.
- 2026-05-08 14:14 -03:00: Adicionadas variáveis `CSRF_SECRET`, `CSRF_COOKIE_NAME` e `CSRF_HEADER_NAME` nos templates de ambiente.
- 2026-05-08 14:14 -03:00: Adicionados testes unitários do `CsrfService` e e2e para emissão/rejeição de CSRF.
- 2026-05-08 14:40 -03:00: Criado `AuditModule` com `AuditService`, `AuditController`, DTO de filtros e tipos de resposta.
- 2026-05-08 14:40 -03:00: Criados `GET /api-v2/audit/logs` e `GET /api-v2/audit/logs/:id`, protegidos por JWT e permissão `audit.read`.
- 2026-05-08 14:40 -03:00: `AuthService` passou a registrar eventos por `AuditService`, com mascaramento de metadados sensíveis e testes unitários/e2e.
- 2026-05-08 14:57 -03:00: Criados `UsersModule`, `RolesModule` e `PermissionsModule` com endpoints `GET /api-v2/users`, `GET /api-v2/users/:id`, `GET /api-v2/roles`, `GET /api-v2/roles/:id` e `GET /api-v2/permissions`.
- 2026-05-08 14:57 -03:00: Criado catálogo inicial `DEFAULT_PERMISSIONS`/`DEFAULT_ROLES` e script `npm.cmd run prisma:seed` para popular permissões e roles sem criar usuário ou senha, incluindo permissões `documents.*` usadas pela etapa seguinte.
- 2026-05-08 14:57 -03:00: Adicionados testes unitários dos services administrativos e e2e 401/403 para as novas rotas.
- 2026-05-11 08:40 -03:00: Lidas novamente as documentações em `docs/`, identificado que o último ponto era o RBAC inicial de 2026-05-08 14:57 e retomada a etapa crítica de documentos.
- 2026-05-11 08:40 -03:00: Criado `DocumentsModule` com `GET /api-v2/documents`, `GET /api-v2/clients/:clientId/documents`, `GET /api-v2/documents/:id`, `GET /api-v2/documents/:id/download`, `POST /api-v2/documents/clients/:clientId/upload`, `POST /api-v2/documents/upload` e `DELETE /api-v2/documents/:id`.
- 2026-05-11 08:40 -03:00: Implementada validação de upload por extensão, MIME, assinatura mágica, tamanho configurável e SHA-256; arquivos são salvos em storage privado configurado por `STORAGE_PRIVATE_PATH`.
- 2026-05-11 08:40 -03:00: `Document` no Prisma ganhou `department`, `storageKey` único, `source`, `metadata` e índices complementares.
- 2026-05-11 08:40 -03:00: Adicionados testes unitários de documentos e e2e 401/403/CSRF para rotas sensíveis de documentos.
- 2026-05-11 08:59 -03:00: Criado template estático de documentos compatível com o legado e endpoint `GET /api-v2/documents/templates`.
- 2026-05-11 08:59 -03:00: `GET /api-v2/clients/:clientId/documents` passou a retornar checklist de requisitos documentais com uploads mais recentes.
- 2026-05-11 08:59 -03:00: Criados `GET /api-v2/documents/required-pending-summary` e `POST /api-v2/documents/custom-requirements`.
- 2026-05-11 08:59 -03:00: Adicionado modelo Prisma `DocumentRequirement` e permissão `documents.requirements` no catálogo RBAC, atribuída a `MANAGER` e herdada por `ADMIN`/`DEV`.
- 2026-05-11 08:59 -03:00: Testes atualizados para 30 unitários e 23 e2e, incluindo template, checklist, resumo de pendências, requisito customizado e CSRF.
- 2026-05-11 13:33 -03:00: Criado `PATCH /api-v2/documents/:id/status` para alterar status de documentos entre `PENDING`, `APPROVED` e `REJECTED`.
- 2026-05-11 13:33 -03:00: Adicionada permissao `documents.review` ao catalogo RBAC, atribuida a `MANAGER`, `DEPARTMENT`, `ACCOUNTING`, `LEGALIZATION` e herdada por `ADMIN`/`DEV`.
- 2026-05-11 13:33 -03:00: Mudancas de status de documentos passam a exigir CSRF, validar escopo por papel/departamento, registrar auditoria e gravar metadata `review` no documento.
- 2026-05-11 13:33 -03:00: Testes atualizados para 32 unitarios e 26 e2e, incluindo revisao de status, permissao `documents.review` e CSRF.
- 2026-05-11 13:55 -03:00: Criado `DocumentUploadScannerService` com staging em quarentena privada antes do storage final, regras estaticas para texto/PDF/Office zip-like e scanner externo ClamAV/clamdscan configuravel.
- 2026-05-11 13:55 -03:00: Adicionadas variaveis `SAMA_UPLOAD_SCAN_MODE`, `SAMA_UPLOAD_SCAN_TIMEOUT_SEC`, `SAMA_UPLOAD_SCAN_BIN`, `SAMA_UPLOAD_SCAN_ARGS` e `SAMA_UPLOAD_QUARANTINE_DIR` na API NestJS.
- 2026-05-11 13:55 -03:00: Upload de documentos passa a gravar metadata `scanner` em `Document.metadata` e auditoria com status/engine/mode do scanner; se a varredura rejeitar, nada e salvo no storage final nem no Prisma.
- 2026-05-11 13:55 -03:00: Testes atualizados para 36 unitarios e 26 e2e, incluindo staging em quarentena, modo `off`, rejeicao de PDF ativo e bloqueio de persistencia quando scanner rejeita.
- 2026-05-11 14:45 -03:00: Iniciado Docker Desktop (estava desligado no Windows).
- 2026-05-11 14:45 -03:00: Criado container MySQL 8.0 em Docker na porta 3306 com credenciais de teste.
- 2026-05-11 14:45 -03:00: Criado `.env` em `portal-sama-api/` com `DATABASE_URL=mysql://portal_user:portal_test@localhost:3306/portal_sama`.
- 2026-05-11 14:45 -03:00: Sincronizado schema Prisma com MySQL via `prisma db push --skip-generate`; schema agora persiste no banco.
- 2026-05-11 14:45 -03:00: Executado `npm.cmd run prisma:seed` com sucesso; 9 roles e 30+ permissões inseridos em MySQL.
- 2026-05-11 14:45 -03:00: Reexecutada suite completa de testes (lint, unitários, e2e, build, audit) com MySQL real.
- 2026-05-11 14:45 -03:00: Confirmado: 36 unitários em 11 suites, 26 e2e em 1 suite, 0 vulnerabilidades, build e lint sem erros.
- 2026-05-11 14:45 -03:00: Criados artefatos de documentação em `.ai-tests/runs/2026-05-11-14-45-mysql-validation/` com summary e commands.
- 2026-05-12 08:31 -03:00: Implementado método `DocumentsService.validateUserDocumentAccess(userId, documentId)` que centraliza lógica de validação de acesso com dados reais em MySQL.
- 2026-05-12 08:31 -03:00: Método valida que cliente existe, usuário existe, e aplica regras de escopo (ADMIN, CLIENT, DEPARTMENT) para retornar documento e dados de usuário autenticado.
- 2026-05-12 08:31 -03:00: Adicionados 6 novos testes unitários: permite acesso ADMIN, permite CLIENT acessar próprio, permite DEPARTMENT acessar seu departamento, rejeita fora de escopo, retorna erro se cliente não existe, retorna erro se usuário não existe.
- 2026-05-12 08:31 -03:00: Validação TypeScript passou para `documents.service.ts` e `documents.service.spec.ts`; 1.153 linhas com tipos corretos.
- 2026-05-12 10:26 -03:00: Corrigida a finalização da Tarefa 2.2 iniciada na worktree: `DocumentStatusHistory` ganhou relação formal com `Document`, `PATCH /api-v2/documents/:id/status` passou a atualizar documento e histórico em transação, e `validateUserDocumentAccess()` agora monta `AuthenticatedUser` completo com `permissions`.
- 2026-05-12 10:26 -03:00: Ajustados mocks Prisma dos testes de documentos (`user`, `documentStatusHistory` e `$transaction`) e removidos casts `any` dos novos cenários.
- 2026-05-12 10:26 -03:00: Adicionada cobertura e2e de proteção para `GET /api-v2/documents/:id/status-history`.
- 2026-05-12 10:26 -03:00: Validação local concluída: `prisma validate`, `prisma generate`, `npm.cmd run lint`, `npm.cmd test` (45 unitários/11 suites), `npm.cmd run test:e2e` (28 e2e/1 suite) e `npm.cmd run build` passaram.
- 2026-05-12 10:48 -03:00: Criado modelo Prisma `PublicToken` para tokens públicos com hash, expiração, revogação, escopo e rastreio de último uso.
- 2026-05-12 10:48 -03:00: Criado `POST /api-v2/documents/public-upload?token=...` como rota pública com throttle específico; o endpoint valida token por SHA-256, escopo, expiração e revogação antes de aceitar upload.
- 2026-05-12 10:48 -03:00: Upload público reaproveita validação de arquivo, quarentena/scanner, storage privado, auditoria e registra documento com `source: onboarding`, `uploadedById: null` e metadata `publicUpload`.
- 2026-05-12 10:48 -03:00: Adicionados testes unitários para upload público por token, token ausente, token expirado e escopo inválido; e2e confirma que a rota pública não exige bearer, mas exige token.
- 2026-05-12 10:48 -03:00: Validação local concluída: `prisma validate`, `prisma generate`, `npm.cmd run lint`, `npm.cmd test` (49 unitários/11 suites), `npm.cmd run test:e2e` (29 e2e/1 suite), `npm.cmd run build` e `npm.cmd audit --audit-level=moderate` passaram.
- 2026-05-12 13:52 -03:00: Criados `GET /api-v2/documents/public-tokens`, `POST /api-v2/documents/public-tokens` e `DELETE /api-v2/documents/public-tokens/:id` para gestão administrativa de tokens públicos de documentos.
- 2026-05-12 13:52 -03:00: Adicionada permissão `documents.public_tokens` ao catálogo RBAC, atribuída a `MANAGER` e herdada por `ADMIN`/`DEV`; mutações exigem CSRF e auditoria.
- 2026-05-12 13:52 -03:00: Criação de token gera segredo opaco `sama_pub_*`, persiste apenas SHA-256 em `PublicToken.tokenHash`, retorna o token bruto uma única vez e lista/revoga sem expor hash/segredo.
- 2026-05-12 13:52 -03:00: Adicionados DTOs, tipos de resposta, testes unitários de emissão/listagem/revogação e e2e 401/403/CSRF para as rotas administrativas de tokens públicos.
- 2026-05-12 13:52 -03:00: Validação local concluída: `prisma validate`, `prisma generate`, `npm.cmd run lint`, `npm.cmd test` (52 unitários/11 suites), `npm.cmd run test:e2e` (33 e2e/1 suite), `npm.cmd run build` e `npm.cmd audit --audit-level=moderate` passaram.
- 2026-05-12 15:25 -03:00: Criadas mutações administrativas de usuários: `POST /api-v2/users`, `PATCH /api-v2/users/:id`, `PATCH /api-v2/users/:id/status` e `PUT /api-v2/users/:id/roles`.
- 2026-05-12 15:25 -03:00: Mutações de usuários exigem permissões granulares (`users.create`, `users.update`, `users.status`, `users.roles`), CSRF, auditoria e não retornam `passwordHash`.
- 2026-05-12 15:25 -03:00: Criação/atualização de usuário aplica hash `bcrypt`, normaliza email/username, valida roles existentes e protege contra auto-bloqueio ou remoção da própria role administrativa.
- 2026-05-12 15:25 -03:00: Adicionados DTOs e testes unitários/e2e para criação, atualização, troca de status, troca de roles, auditoria e rejeição sem CSRF.
- 2026-05-12 15:25 -03:00: Validação local concluída: `prisma validate`, `prisma generate`, `npm.cmd run lint`, `npm.cmd test` (56 unitários/11 suites), `npm.cmd run test:e2e` (37 e2e/1 suite), `npm.cmd run build` e `npm.cmd audit --audit-level=moderate` passaram.
- 2026-05-12 16:13 -03:00: Criadas mutações administrativas de roles: `POST /api-v2/roles`, `PATCH /api-v2/roles/:id` e `PUT /api-v2/roles/:id/permissions`.
- 2026-05-12 16:13 -03:00: Criadas mutações administrativas do catálogo de permissões: `POST /api-v2/permissions` e `PATCH /api-v2/permissions/:id`, incluindo permissões `permissions.create` e `permissions.update`.
- 2026-05-12 16:13 -03:00: Mutações de roles/permissões exigem CSRF, permissões granulares, auditoria e validação de chaves persistidas; o service bloqueia remoção da própria capacidade de gerenciar roles e impede renomear chaves padrão do catálogo.
- 2026-05-12 16:13 -03:00: Validação local concluída: `prisma validate`, `prisma generate`, `npm.cmd run lint`, `npm.cmd test` (63 unitários/11 suites), `npm.cmd run test:e2e` (42 e2e/1 suite), `npm.cmd run build` e `npm.cmd audit --audit-level=moderate` passaram.
- 2026-05-12 17:27 -03:00: Criada migration Prisma formal inicial `portal-sama-api/prisma/migrations/20260512172700_init_current_schema/migration.sql` e `migration_lock.toml` para provider MySQL.
- 2026-05-12 17:27 -03:00: Adicionado script `npm.cmd run prisma:migrate:deploy` para uso em homologação/produção.
- 2026-05-12 17:27 -03:00: Migration aplicada com `prisma migrate deploy` em banco MySQL descartável `portal_sama_migration_check`; `migrate diff --from-migrations --to-schema-datamodel` não encontrou diferenças.
- 2026-05-12 17:27 -03:00: `prisma:seed` executado no banco descartável após a migration; resultado conferido com 9 roles, 37 permissões e 124 vínculos em `role_permissions`.
- 2026-05-12 19:50 -03:00: Criado `CertificatesModule` com CRUD, download protegido, rotação de senha, criptografia AES-256-GCM, validação `.p12/.pfx`, storage privado, auditoria e migration `20260512195000_add_digital_certificates`.
- 2026-05-12 19:50 -03:00: Validação local de certificados passou com `prisma validate`, `prisma generate`, lint, 73 unitários em 14 suites, 52 e2e, build, audit, `migrate deploy` em banco descartável e seed RBAC.
- 2026-05-13 08:57 -03:00: Criado `ProposalsModule` com `GET/POST/PATCH /api-v2/proposals`, envio por link público, aprovação/rejeição interna e resposta pública por token em `/api-v2/public/proposals/:token`.
- 2026-05-13 08:57 -03:00: Criado modelo Prisma `Proposal`, enum `ProposalStatus` e migration `20260513085700_add_proposals`.
- 2026-05-13 08:57 -03:00: Envio de proposta gera token opaco `sama_pub_*`, persiste apenas SHA-256 em `PublicToken`, revoga tokens ativos anteriores, exige CSRF nas mutações internas e audita sem token bruto.
- 2026-05-13 08:57 -03:00: Validação local de propostas passou com `prisma format`, `prisma generate`, `prisma validate`, lint, 78 unitários em 15 suites, 59 e2e, build, `npm audit` e `git diff --check`; `prisma:migrate:status` ficou bloqueado por Docker/MySQL local indisponível.
- 2026-05-13 09:15 -03:00: Corrigida configuração TypeScript/Jest local com `types: ["node", "jest"]`, inclusão explícita de `src/test/prisma` no `tsconfig.json` e `typescript.tsdk` do VS Code apontando para `portal-sama-api/node_modules/typescript/lib`.
- 2026-05-13 09:15 -03:00: `npm.cmd install` confirmou dependências atualizadas da API; `tsc --noEmit`, `prisma validate`, teste focado de propostas, lint e build passaram.
- 2026-05-13 09:20 -03:00: Removido `compilerOptions.baseUrl` do `tsconfig.json` porque o projeto usa imports relativos e o TypeScript 6 marca a opção como depreciada; `tsc --noEmit`, lint e build passaram.
- 2026-05-13 09:34 -03:00: Criado `ContractsModule` com `GET/POST/PATCH /api-v2/contracts`, `POST /api-v2/contracts/:id/generate`, `POST /api-v2/contracts/:id/send-signature` e fluxo público `GET/POST /api-v2/public/signatures/:token`.
- 2026-05-13 09:34 -03:00: Criado modelo Prisma `Contract`, enum `ContractStatus` e migration `20260513092300_add_contracts`.
- 2026-05-13 09:34 -03:00: Envio de contrato para assinatura gera token opaco `sama_pub_*`, persiste apenas SHA-256 em `PublicToken`, revoga tokens ativos anteriores, exige CSRF nas mutações internas e audita sem token bruto.
- 2026-05-13 09:34 -03:00: Assinatura pública grava evidência com hash do documento/assinatura, IP e user-agent; token é revogado após uso.
- 2026-05-13 09:34 -03:00: Validação local de contratos passou com `prisma format`, `prisma generate`, `prisma validate`, `tsc --noEmit`, lint, 83 unitários em 16 suites, 66 e2e, build, `npm audit` e `git diff --check`.
- 2026-05-13 10:15 -03:00: Criado `AccessRequestsModule` com `GET/POST /api-v2/access-requests`, `GET /api-v2/access-requests/my/latest`, `GET /api-v2/access-requests/manager/approvals`, `GET /api-v2/access-requests/:id`, `POST /api-v2/access-requests/:id/approve` e `POST /api-v2/access-requests/:id/reject`.
- 2026-05-13 10:15 -03:00: Criado modelo Prisma `AccessRequest`, enums `AccessRequestProfile`/`AccessRequestStatus` e migration `20260513101500_add_access_requests`.
- 2026-05-13 10:15 -03:00: Solicitacoes de colaborador validam gestor ativo do mesmo departamento; solicitacoes de gestor validam colaboradores ativos do mesmo departamento; decisoes exigem permissao granular, CSRF e auditoria.
- 2026-05-13 10:15 -03:00: Validacao local de solicitacoes de acesso passou com `prisma format`, `prisma generate`, `prisma validate`, `tsc --noEmit`, lint, 88 unitarios em 17 suites, 75 e2e, build, `npm audit` e `git diff --check`; `prisma:migrate:status` segue bloqueado por baseline.
- 2026-05-13 10:50 -03:00: Resolvido baseline local do MySQL/Prisma sem expor `.env`: `db push --skip-generate`, `migrate resolve --applied` para as 5 migrations pendentes, `prisma:migrate:status` limpo, diff sem diferencas e seed executado.
- 2026-05-13 10:50 -03:00: Criado `ClientsModule` com CRUD inicial, dashboard operacional por cliente, aliases legados de resposta, CSRF em mutacoes, permissoes `clients.*` e auditoria.
- 2026-05-13 10:50 -03:00: Expandido modelo Prisma `Client` e aplicada migration `20260513113000_expand_clients_profile` no MySQL local por `prisma:migrate:deploy`.
- 2026-05-13 10:50 -03:00: Validacao local de clientes passou com `tsc --noEmit`, lint, 93 unitarios em 18 suites, 84 e2e, build, audit, status/diff Prisma e `git diff --check`.
- 2026-05-13 14:43 -03:00: Criado `CollaboratorsModule` com listagem, detalhe, criacao, atualizacao e arquivamento logico de colaboradores internos baseados em `User`, com aliases legados (`usuario`, `nome`, `departamento`, `cargo`, `telefone`, `ramal`).
- 2026-05-13 14:43 -03:00: Adicionadas permissoes `collaborators.read/create/update/delete` ao catalogo RBAC e aplicado seed no MySQL local.
- 2026-05-13 14:43 -03:00: Expandido modelo Prisma `User` com campos funcionais de colaborador e aplicada migration `20260513124500_add_collaborator_profile_to_users`.
- 2026-05-13 14:43 -03:00: Validacao local de colaboradores passou com `tsc --noEmit`, lint, 99 unitarios em 19 suites, 91 e2e, build, audit, status/diff Prisma, seed e `git diff --check`.

---

## O que está em andamento

- ✅ MySQL real validado em Docker e seed RBAC executado.
- ✅ Validação centralizada de acesso implementada em `DocumentsService.validateUserDocumentAccess()`.
- ✅ Histórico estruturado de status implementado no código (`DocumentStatusHistory`, endpoint de consulta e gravação transacional em mudança de status).
- ✅ Upload público por token implementado no código (`PublicToken`, `POST /api-v2/documents/public-upload`, hash/expiração/revogação/escopo/throttle/auditoria).
- ✅ Gestão administrativa de tokens públicos implementada no código (`GET/POST/DELETE /api-v2/documents/public-tokens`, `documents.public_tokens`, CSRF e auditoria).
- ✅ Mutações administrativas de usuários implementadas no código (`POST/PATCH/PUT /api-v2/users*`, hash de senha, roles, status, CSRF e auditoria).
- ✅ Mutações administrativas de roles e permissões implementadas no código (`POST/PATCH /api-v2/roles`, `PUT /api-v2/roles/:id/permissions`, `POST/PATCH /api-v2/permissions`, CSRF e auditoria).
- ✅ Migration Prisma formal inicial criada e validada em MySQL descartável com seed RBAC atualizado.
- ✅ Certificados digitais implementados no backend (`CertificatesModule`, criptografia de senha, storage privado, download protegido e migration incremental).
- ✅ Propostas implementadas no backend em primeira fatia (`ProposalsModule`, token público opaco, resposta pública, CSRF e auditoria).
- ✅ Contratos e assinatura implementados no backend em primeira fatia (`ContractsModule`, snapshot HTML, token público opaco, assinatura pública, CSRF e auditoria).
- ✅ Solicitacoes de acesso implementadas no backend em primeira fatia (`AccessRequestsModule`, escopo por gestor/departamento, aprovacao/rejeicao, CSRF e auditoria).
- ✅ Notificacoes implementadas no backend em primeira fatia (`NotificationsModule`, listagem/criacao/ack/limpeza, escopo por usuario/departamento, CSRF e auditoria).
- ✅ Baseline local do Prisma/MySQL resolvido em `portal_sama`; migration incremental de clientes aplicada e diff Prisma vazio.
- ✅ Clientes implementados no backend em primeira fatia (`ClientsModule`, CRUD inicial, dashboard, aliases legados, CSRF e auditoria).
- ✅ Colaboradores implementados no backend em primeira fatia (`CollaboratorsModule`, CRUD inicial, campos funcionais no `User`, aliases legados, escopo por departamento, CSRF e auditoria).
- Continuidade da migração gradual para NestJS/React agora parte da fundação `portal-sama-api` em `/api-v2` com banco real.
- Integração segura do frontend futuro com `/api-v2/auth/*` agora depende de armazenamento em memória do access token, fluxo `csrf -> login -> refresh -> logout` em cliente real e testes em navegador.
- Integracao inicial da tela `/auditoria` com `/api-v2/audit/logs` existe no React; agora depende de validacao com MySQL/usuarios reais, seed `audit.read`, Playwright e politica de retencao/exportacao.
- Integração futura das telas DEV/admin com `/api-v2/users`, `/api-v2/roles` e `/api-v2/permissions` agora depende de validação MySQL real, migração de usuários legados, escopo por recurso e frontend React.
- Integração futura do painel de documentos com `/api-v2/documents` agora depende de validação operacional de ClamAV/mode `strict`, backfill/seed de dados reais, validação MySQL/storage/permissões reais e frontend React.
- Integração futura da tela de propostas com `/api-v2/proposals` depende de validar migration/backfill em MySQL de homologação e adaptar o frontend.
- Integração futura das telas de contrato/assinatura com `/api-v2/contracts` e `/api-v2/public/signatures/:token` depende de baseline/migrations, validacao operacional da renderizacao/importacao PDF e adaptação do frontend.
- Integracao futura das telas de solicitacao de acesso/TI com `/api-v2/access-requests` e `/api-v2/notifications` depende de backfill de `sama_access_requests`, validacao dos emissores automaticos em homologacao e adaptacao do frontend.
- Validação sintática PHP dos endpoints alterados depende de `php` disponível no PATH local.
- Migrations Prisma formais iniciais já existem; bancos previamente sincronizados por `db push` devem ser baselined com cuidado antes de `migrate deploy`.

---

## O que ainda falta

- ✅ **Validação de acesso centralizada:** `DocumentsService.validateUserDocumentAccess()` implementada e testada.
- ✅ **Histórico estruturado no código:** `DocumentStatusHistory`, relação com `Document`, `GET /api-v2/documents/:id/status-history` e gravação transacional em `PATCH /api-v2/documents/:id/status` implementados e testados; backfill de `metadata.review` em banco legado continua pendente.
- ✅ **Upload público por token no código:** `POST /api-v2/documents/public-upload?token=...` implementado com hash SHA-256, expiração, revogação, escopo, throttle, scanner/quarentena/storage e auditoria; `GET/POST/DELETE /api-v2/documents/public-tokens` implementados para emissão/listagem/revogação administrativa. Ainda faltam Redis para rate limit distribuído, backfill/limpeza operacional e validação com MySQL real.
- **ClamAV validação:** Testar scanner em modo `strict` com EICAR no host real.
- ✅ **Mutações administrativas:** Usuários, roles e permissões já possuem mutações principais com permissões granulares, CSRF e auditoria; validar com MySQL/homologação e integrar às telas DEV/admin.
- ✅ **Migrations Prisma:** Migration formal inicial criada e validada com `prisma migrate deploy`; bancos existentes que receberam `db push` exigem baseline controlado antes de produção.
- ✅ **Certificados digitais no código:** `CertificatesModule` implementado e testado; tela React inicial criada em `/certificados-digitais`; faltam validação MySQL/storage reais, backfill legado, fluxo integrado em homologacao e Playwright.
- ✅ **Propostas no código:** `ProposalsModule` implementado e testado; faltam `migrate deploy` em MySQL de homologação, backfill de `sama_propostas_v1`/tokens legados e integração frontend.
- ✅ **Contratos/assinatura no código:** `ContractsModule` implementado e testado; faltam baseline/migrations em MySQL real, importação/render PDF segura, backfill legado e tela React/legada.
- ✅ **Solicitacoes de acesso no codigo:** `AccessRequestsModule` implementado e testado; emissor automatico de notificacoes para criacao/aprovacao/rejeicao foi adicionado; faltam backfill legado, validacao com usuarios reais e tela React/legada.
- ✅ **Notificacoes no codigo:** `NotificationsModule` implementado e testado; SSE/stream, Web Push, teste operacional `/push/test` e emissores iniciais dos modulos de negocio foram adicionados; tela React `/notificacoes` consome listagem, acoes, stream SSE autenticado e assinatura/cancelamento Push API; faltam validacao manual em HTTPS/homologacao com VAPID real, broker/event bus distribuido se necessario, backfill/retencao e Playwright.
- ✅ **Clientes no codigo:** `ClientsModule` implementado e testado; faltam backfill de clientes/colaboradores legados, vinculos cliente-usuario/gestor e integracao frontend.
- ✅ **Colaboradores no codigo:** `CollaboratorsModule` implementado e testado; faltam backfill de `sama_colaboradores`, vinculos colaborador-cliente/gestor, transferencias/carteira e integracao frontend.
- ✅ Fundação frontend React/Vite criada em `portal-sama-web/` com login API v2, refresh por cookie, access token em memoria e layout interno inicial.
- Completar a autenticação alvo com testes com MySQL real/homologação, seeds/migração de usuários e validação do fluxo React em navegador HTTPS.
- Completar auditoria com seeds/permissoes reais, testes com MySQL, validacao da tela React, Playwright e politica de retencao/exportacao.
- Definir comandos formais de teste em `package.json`, `composer.json` ou scripts documentados.
- Executar lint/test/build quando o ambiente local tiver ferramentas disponíveis.
- Revisar exposição de `.env` local e rotacionar segredos se esse arquivo tiver sido compartilhado fora do ambiente seguro.
- Executar `php -l` em `SolicitacaoAcesso/enviar_acesso.php` e `SolicitacaoAcesso/verificar_codigo.php` quando PHP estiver disponível.
- Integrar `Client/painel.js`/futuro React aos endpoints de documentos somente após validar storage, banco, escopo e auditoria em homologação.
- Integrar `Legalizacao/proposta.js`/futuro React aos endpoints de propostas somente após validar migration, token público e escopo com dados reais.

---

## Riscos atuais

- `.env` existe no repositório local. Valores não foram lidos nem expostos nesta auditoria.
- `.env.example` existe, mas deve ser revisado junto ao deploy real antes de produção.
- `docs/` estava ignorado no `.gitignore`, impedindo continuidade versionável; regra corrigida nesta auditoria.
- `docs/` segue listado no `.dockerignore`, o que pode ser aceitável para deploy, mas precisa estar alinhado com a estratégia de entrega de documentação.
- `package-lock.json` existe sem `package.json` correspondente, indicando resíduo ou configuração Node incompleta.
- `php` não está disponível no PATH local, bloqueando lint PHP nesta máquina.
- `npm` via `npm.ps1` está bloqueado pela política de execução do PowerShell; `npm.cmd` funciona e foi usado em `portal-sama-api`.
- `prisma migrate deploy` passou em banco descartável e, em 2026-05-13 10:50, o MySQL local `portal_sama` foi baselined; `prisma:migrate:status` e `prisma migrate diff` passaram. Homologação/produção ainda exigem backup e procedimento próprio antes de aplicar migrations.
- `AuthModule` novo ainda não foi validado contra dados reais de usuários nem conectado ao frontend.
- `AuditModule` e módulos administrativos novos foram validados por unit/e2e sem banco real; listagens reais e permissões persistidas dependem de MySQL/homologação e seed RBAC.
- `DocumentsModule` novo foi validado por unit/e2e sem banco real; upload/download/revisao reais dependem de MySQL/homologação, storage privado persistente, ClamAV real no host, seed/permissões reais e escopo por vínculo de cliente/gestor.
- `ProposalsModule` novo foi validado por unit/e2e sem banco real; fluxos reais dependem de MySQL/homologação, backfill de propostas legadas, compatibilidade de tokens públicos antigos e integração frontend.
- `ContractsModule` novo foi validado por unit/e2e sem banco real; fluxos reais dependem de baseline/migrations, backfill de contratos legados, Legal Doc Service real, validacao de renderizacao/importacao PDF e integração frontend.
- `LegalizationModule` novo foi validado por teste focado/e2e de protecao e migration local; processos internos, templates de cabecalho/rodape e emissores de notificacoes estao cobertos, mas fluxos reais dependem de backfill legado, validacao operacional de PDF e integracao frontend.
- `AccessRequestsModule` novo foi validado por unit/e2e sem banco real; fluxos reais dependem de baseline/migrations, backfill de `sama_access_requests`, escopo com usuarios reais e integracao frontend.
- `NotificationsModule` novo foi validado por teste focado/e2e de protecao e migration local; SSE/stream, Web Push, teste operacional `/push/test`, emissores iniciais dos modulos de negocio e integracao React de stream/Push existem, mas validacao manual com VAPID real, broker distribuido e Playwright seguem pendentes.
- `CollaboratorsModule` novo foi validado por unit/e2e e migration local, mas fluxos reais dependem de backfill de `sama_colaboradores`, vinculos cliente-usuario/gestor, carteiras/transferencias e integracao frontend.
- CSRF foi implementado nas rotas de autenticação com cookie, mas ainda precisa ser validado em navegador/frontend real e ampliado para futuras mutações de módulos sensíveis.
- CSRF já foi reutilizado no upload/arquivamento de documentos em NestJS, mas ainda precisa de validação em navegador/frontend real.
- Endpoints PHP action-based continuam concentrando responsabilidades e exigem validação action por action contra IDOR, CSRF e autorização por escopo. O risco de stack trace público nos dois endpoints alterados foi mitigado por debug opt-in.

---

## Próximos passos recomendados

- ✅ **Tarefa 2.3 concluída (2026-05-12):** Validação centralizada de acesso implementada com `validateUserDocumentAccess()`.
- ✅ **Tarefa 2.2 concluída no código (2026-05-12):** `DocumentStatusHistory` criado, endpoint de consulta protegido e mudança de status gravando histórico em transação; executar backfill de `metadata.review` em homologação quando houver dados legados.
- ✅ **Tarefa 2.1 concluída no código (2026-05-12):** upload público por token e gestão administrativa (`GET/POST/DELETE /api-v2/documents/public-tokens`) implementados no `DocumentsModule`; validar MySQL real, definir rate limit distribuído e política de limpeza de tokens.
- **Tarefa 1.1:** Validar ClamAV em modo `strict` com EICAR no host para desbloquear upload em produção.
- ✅ **Tarefa admin-mutations concluída no código (2026-05-12):** Users, Roles e Permissions possuem mutações principais com CSRF, auditoria e proteções de auto-preservação administrativa; validar com MySQL/homologação, migration/seed e frontend DEV/admin.
- ✅ **Tarefa migrations inicial concluída (2026-05-12):** Migration Prisma inicial criada, `prisma:migrate:deploy` adicionado e validação feita em MySQL descartável com seed RBAC atualizado.
- ✅ **Tarefa certificados concluída no código (2026-05-12):** `CertificatesModule` criado com storage privado, criptografia, download protegido e auditoria; validar storage/MySQL reais e backfill legado.
- ✅ **Tarefa propostas concluída no código (2026-05-13):** `ProposalsModule` criado com CRUD inicial, envio por token opaco, aprovação/rejeição interna e resposta pública; validar migration/backfill em MySQL de homologação.
- ✅ **Tarefa contratos/assinatura concluída no código (2026-05-13):** `ContractsModule` criado com CRUD inicial, snapshot HTML, envio para assinatura e assinatura pública por token opaco; validar baseline/migrations e implementar render/import seguro.
- ✅ **Tarefa solicitacoes de acesso concluida no codigo (2026-05-13):** `AccessRequestsModule` criado com criacao, listagem, ultima solicitacao do colaborador, fila do gestor e aprovacao/rejeicao; validar baseline/migrations, backfill legado e integracao com notificacoes/frontend.
- ✅ **Tarefa clientes concluida no codigo (2026-05-13):** `ClientsModule` criado com CRUD inicial, dashboard, aliases legados, CSRF e auditoria; validar backfill/vinculos reais e integrar telas de clientes.
- ✅ **Tarefa colaboradores concluida no codigo (2026-05-13):** `CollaboratorsModule` criado com CRUD inicial, campos funcionais no `User`, permissoes `collaborators.*`, CSRF, escopo por departamento e auditoria; validar backfill/vinculos/carteira e integrar telas de colaboradores.
- ✅ **Tarefa legalizacao/processos concluida no codigo (2026-05-13):** `LegalizationModule` criado com CRUD inicial de processos, etapa/status, timeline, arquivamento logico, permissoes `legalization.*`, CSRF, escopo por responsavel/criador/departamento e auditoria.
- ✅ **Tarefa legalizacao/templates concluida no codigo (2026-05-13):** `GET/POST/PATCH/DELETE /api-v2/legalization/templates` criado com permissao `legalization.templates`, auditoria, soft delete e rejeicao de HTML inseguro; validar backfill, Legal Doc Service/PDF e frontend.
- ✅ **Tarefa notificacoes concluida no codigo (2026-05-13/2026-05-14):** `NotificationsModule` criado com `GET/POST /api-v2/notifications`, ack, alert-ack, clear-unread, `GET /api-v2/notifications/stream`, Web Push, `/push/test` e emissores iniciais nos modulos de negocio; validar HTTPS/VAPID real, broker distribuido se necessario, backfill/retencao e frontend.
- Corrigir documentação e inventários mantidos nesta auditoria quando novas evidências forem coletadas.
- Revisar `.env.example` com o ambiente EasyPanel real, mantendo placeholders e sem segredos.
- Resolver ambiente local de validação: PHP no PATH e forma segura de executar npm.
- Rodar `php -l` nos endpoints críticos quando PHP estiver disponível.
- Definir estratégia de baseline para bancos que já foram sincronizados por `prisma db push` antes de usar `prisma migrate deploy`.
- Validar `AuthModule` com MySQL real/homologação e testar fluxo CSRF completo no frontend antes de uso em produção.
- Validar `AuditModule`, `UsersModule`, `RolesModule` e `PermissionsModule` com MySQL real/homologacao; `/auditoria` ja tem primeira tela React, e DEV/admin seguem pendentes.
- Validar `DocumentsModule` com MySQL real/homologação, storage privado persistente, arquivo real por tipo permitido, migration/backfill de histórico, scanner ClamAV/EICAR e permissão/escopo com dados reais.
- Validar `ProposalsModule` com MySQL real/homologação, `prisma migrate deploy`, tokens públicos reais, backfill de KV legado `sama_propostas_v1` e integração com `Legalizacao/proposta.js`.
- Validar `ContractsModule` com MySQL real/homologação, baseline seguro, tokens públicos reais, backfill de contratos legados e integração com `Legalizacao/contrato.js`/`Legalizacao/assinatura.js`.
- Iniciar proxima etapa backend prioritaria: validacao operacional do Legal Doc Service/PDF, validacao manual Web Push em HTTPS/VAPID real, manager workspace/transfers, backfills ou integracao frontend das fatias ja criadas, conforme destravamento do banco.
- Integrar as proximas telas prioritarias do React aos endpoints ja criados (`UsersModule`/admin, propostas, contratos/assinatura e fluxos publicos de onboarding); documentos, solicitacoes, auditoria, notificacoes, certificados, legalizacao e onboarding/processos ja possuem primeira tela React e agora precisam de validacao integrada.
- Validar autenticacao React em homologacao (`csrf -> login -> refresh -> logout` com access token em memoria e refresh cookie HttpOnly).
- Priorizar autenticação, usuários/permissões, auditoria e documentos, conforme `docs/ROADMAP_REFATORACAO.md`.
