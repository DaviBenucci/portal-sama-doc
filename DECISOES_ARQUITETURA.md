# [CONCLUIDO] Decisões de Arquitetura - Portal Sama

## ADR-0033 - Integra-AI usa Dominio Importador 01/02/03/99 para Lancamentos em Lote

- **Data:** 2026-05-27
- **Status:** Aceita
- **Contexto:** Evidencia nova do Dominio mostrou que o arquivo do Portal Sama esta sendo importado no conjunto correto `Lancamentos Contabeis em Lote`. Nesse conjunto, linhas `0451|...` sao lidas como registro fixo `04 - Rateios Gerenciais`, deslocando sequencial, contas e valor.
- **Decisao:** O leiaute oficial do Integra-AI para este fluxo e `dominio_importador_lancamentos_lote_01_02_03_99` (`Dominio Importador 01/02/03/99`). O frontend deve exibir apenas esse leiaute. O backend deve gerar registros fixos `01`, pares `02`/`03` e trailer `99`, rejeitando `dominio_separador_0000_0451`.
- **Alternativas consideradas:** Manter `0000/0451` como oficial; permitir escolha de leiaute; criar uma chave por importador sem homologacao; aguardar nova parametrizacao do Dominio antes de corrigir o Portal.
- **Consequencias positivas:** O TXT passa a ser compativel com `Lancamentos Contabeis em Lote`, sem pipes, com tamanhos fixos validados e sem deslocamento de campos no Dominio.
- **Consequencias negativas:** O leiaute com separador fica fora do fluxo principal atual; se outro conjunto do Dominio for homologado no futuro, deve entrar como fluxo separado e documentado.
- **Substitui:** ADR-0032 na parte que definia `dominio_separador_0000_0451` como leiaute principal.
- **Arquivos impactados:** `portal-sama-api/src/modules/accounting/integra-ai.engine.ts`, `portal-sama-api/src/modules/accounting/accounting.service.spec.ts`, `portal-sama-api/src/modules/accounting/integra-ai.engine.spec.ts`, `portal-sama-web/src/types/integra-ai.ts`, `portal-sama-web/tests/e2e/smoke.spec.ts`, `leiaute_dominio_integra_ai.md`.
- **Referencias:** `leiaute_dominio_integra_ai.md`, `paginas/contabil-integra-ai.md`.

## ADR-0032 - Leiaute Dominio oficial unico no Integra-AI

- **Data:** 2026-05-27
- **Status:** Substituida pela ADR-0033
- **Contexto:** A documentacao `leiaute_dominio_integra_ai.md` identificou que o Integra-AI antigo mantinha dois leiautes Dominio (`01/02/03/99` e `0000/0451`), criando ambiguidade no frontend e risco de erro de importacao no Dominio Contabilidade Fiscal.
- **Decisao:** O fluxo principal do Portal Sama deve usar somente `dominio_separador_0000_0451` (`Dominio Sistemas com Separador - 0000/0451`). O frontend nao deve oferecer seletor de leiaute; o backend deve gerar exclusivamente registros `0000` e `0451`, validar o TXT antes da exportacao e rejeitar `export_strategy` divergente.
- **Alternativas consideradas:** Manter os dois leiautes ativos; manter `01/02/03/99` como default antigo; permitir selecao por usuario; aguardar homologacao manual antes de remover a ambiguidade da UI.
- **Consequencias positivas:** Reduz erro operacional, alinha o Portal Sama a documentacao oficial de separador, simplifica preview/testes e melhora auditoria da exportacao.
- **Consequencias negativas:** Esta ADR foi substituida pela ADR-0033; a premissa de que `01/02/03/99` ficaria fora do caminho principal nao vale para o fluxo atual de `Lancamentos Contabeis em Lote`.
- **Arquivos impactados:** `portal-sama-api/src/modules/accounting/integra-ai.engine.ts`, `portal-sama-api/src/modules/accounting/accounting.service.ts`, `portal-sama-api/src/modules/accounting/dto/integra-ai-operation.dto.ts`, `portal-sama-web/src/pages/accounting/IntegraAiPage.tsx`, `portal-sama-web/src/types/integra-ai.ts`, `leiaute_dominio_integra_ai.md`.
- **Referencias:** `leiaute_dominio_integra_ai.md`, `paginas/contabil-integra-ai.md`, `SEGURANCA.md`.

## ADR-0031 - Restore drill protegido em alvo isolado

- **Data:** 2026-05-26
- **Status:** Aceita
- **Contexto:** A stack nova ja possui geracao e verificacao de backup operacional, mas a prontidao para producao ainda exige provar restore/rollback sem risco de sobrescrever o banco ou storage reais da aplicacao.
- **Decisao:** Criar `npm run ops:restore:drill` no `portal-sama-api`, operando em dry-run por padrao. A aplicacao real de restore exige banco/storage isolados, bloqueia alvo igual a `DATABASE_URL`/`STORAGE_PRIVATE_PATH` e exige a frase `RESTORE_DRILL_TARGET_IS_ISOLATED`.
- **Alternativas consideradas:** Manter restore apenas como procedimento manual; embutir restauracao no verificador de backup; permitir restore direto no `DATABASE_URL` com confirmacao simples; aguardar CI/CD externo antes de documentar rollback.
- **Consequencias positivas:** O plano de rollback passa a ser executavel e auditavel, reduz risco de erro operacional e mantem a restauracao real separada do ambiente produtivo.
- **Consequencias negativas:** O comando ainda depende de alvo MySQL/storage isolados e nao substitui snapshots/retencao externos do EasyPanel; a execucao real continua pendente ate haver artefatos reais e janela operacional.
- **Arquivos impactados:** `portal-sama-api/scripts/restore-drill-operational-backup.js`, `portal-sama-api/package.json`, `portal-sama-api/README.md`, `portal-sama-api/.env.example`, `PLANO_ROLLBACK_RESTORE_DRILL.md`, `EASYPANEL_DEPLOY.md`, `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`.
- **Referencias:** `EASYPANEL_DEPLOY.md`, `PLANO_ROLLBACK_RESTORE_DRILL.md`, `SEGURANCA.md`, `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`.

## ADR-0030 - Tres repositorios oficiais com documentacao centralizada

- **Data:** 2026-05-22
- **Status:** Aceita
- **Contexto:** A homologacao no EasyPanel sera feita a partir de repositorios Bitbucket separados para frontend e backend. A documentacao tambem precisa ficar versionada como fonte de verdade compartilhada, para que qualquer IA ou pessoa desenvolvedora leia o contexto completo antes de mexer em `portal-sama-api` ou `portal-sama-web`.
- **Decisao:** Criar a topologia oficial de tres repositorios: `portal-sama-docs` para toda a documentacao, `portal-sama-api` para a API NestJS e `portal-sama-web` para o frontend React/Vite. O repositorio `portal-sama-docs` deve ser lido primeiro em qualquer ciclo de trabalho e atualizado sempre que mudancas tecnicas alterarem status, decisao, teste, pendencia, deploy ou prontidao para producao.
- **Alternativas consideradas:** Manter docs apenas no repositorio monolitico atual; copiar docs completas nos repos de API e Web; manter apenas uma copia minima das instrucoes em cada repo; criar docs somente depois da homologacao.
- **Consequencias positivas:** A documentacao fica independente dos builds de aplicacao, a IA tem um ponto unico de contexto, e EasyPanel pode consumir API/Web sem carregar todo o legado.
- **Consequencias negativas:** E necessario manter disciplina de sincronizacao entre commits tecnicos e commits de documentacao; API/Web podem precisar de README minimo apontando para `portal-sama-docs`.
- **Arquivos impactados:** `docs/AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`, `docs/INSTRUÇÕEDS_AI.MD`, `docs/README_DOCUMENTACAO.md`, `docs/ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md`, `docs/EASYPANEL_DEPLOY.md`.
- **Referencias:** `docs/AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`, `docs/EASYPANEL_DEPLOY.md`, `docs/README_DOCUMENTACAO.md`.

## ADR-0029 - Painel obrigatorio de prontidao para deploy em producao

- **Data:** 2026-05-22
- **Status:** Aceita
- **Contexto:** O projeto passou a precisar de um documento unico para responder quanto falta para homologacao/producao, quais bloqueios ainda existem e como separar `portal-sama-api` e `portal-sama-web` para deploy via Bitbucket/EasyPanel. Antes disso, parte desse diagnostico ficava distribuida entre status, pendencias, changelog, relatorio de testes e historico de chat.
- **Decisao:** Promover `docs/AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD` a painel obrigatorio e vivo de prontidao para producao. Sempre que houver impacto em deploy, Bitbucket, EasyPanel, banco real, backfills, usuarios/permissoes reais, storage/ClamAV, Playwright real, QA visual ou percentual de conclusao, esse documento deve ser lido e atualizado junto das docs de continuidade.
- **Alternativas consideradas:** Manter apenas `STATUS_IMPLEMENTACAO.md`; manter apenas `PENDENCIAS_TECNICAS.md`; deixar o acompanhamento de percentual no chat; criar um documento separado apenas quando a homologacao iniciar.
- **Consequencias positivas:** O estado de producao fica centralizado, rastreavel e mais facil de continuar entre chats; a separacao Bitbucket/EasyPanel passa a ter checklist proprio; a porcentagem de andamento fica ancorada em evidencias de documentacao e codigo.
- **Consequencias negativas:** Cada ciclo de deploy/progresso exige atualizar mais um documento, o que aumenta a disciplina documental necessaria.
- **Arquivos impactados:** `docs/AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`, `docs/INSTRUÇÕEDS_AI.MD`, `docs/README_DOCUMENTACAO.md`, `docs/EASYPANEL_DEPLOY.md`.
- **Referencias:** `docs/EASYPANEL_DEPLOY.md`, `docs/STATUS_IMPLEMENTACAO.md`, `docs/PENDENCIAS_TECNICAS.md`.

## ADR-0028 - Bootstrap administrativo explicito para homologacao sem legado

- **Data:** 2026-05-22
- **Status:** Aceita
- **Contexto:** A API v2 ja possuia autenticacao, RBAC e seed de roles/permissoes, mas o seed intencionalmente nao criava usuario/senha. Em um banco limpo do EasyPanel, isso deixava a homologacao dependente de backfill/legado ou de insercao manual de senha.
- **Decisao:** Manter `prisma:seed` restrito a RBAC e criar `prisma:bootstrap-admin` como comando operacional separado. O bootstrap le variaveis `SAMA_BOOTSTRAP_ADMIN_*`, exige senha minima, permite apenas roles `ADMIN` ou `DEV`, cria/atualiza o usuario, vincula role, registra auditoria e nao imprime segredo.
- **Alternativas consideradas:** Criar admin fixo no seed; inserir SQL manual no phpMyAdmin; depender da aplicacao antiga para cadastrar usuario inicial; liberar endpoint publico temporario de setup.
- **Consequencias positivas:** Credenciais nao entram no Git, o seed continua deterministico, a homologacao da nova stack ganha primeiro login sem legado e o comando pode ser repetido com seguranca.
- **Consequencias negativas:** Ainda exige acesso operacional ao shell/exec do container no EasyPanel e validacao real de MySQL/migrations; nao substitui backfill de usuarios/permissoes reais.
- **Arquivos impactados:** `portal-sama-api/prisma/seed.ts`, `portal-sama-api/prisma/bootstrap-admin.ts`, `portal-sama-api/package.json`, `portal-sama-api/.env.example`, `docs/EASYPANEL_DEPLOY.md`.
- **Referencias:** `docs/EASYPANEL_DEPLOY.md`, `docs/SEGURANCA.md`, `docs/BANCO_DADOS_MYSQL_PRISMA.md`.

## ADR-0027 - GSAP fora do caminho critico da rota autenticada

- **Data:** 2026-05-21
- **Status:** Aceita
- **Contexto:** A nova intro GSAP v2 estava funcionando em build/lint, mas em desenvolvimento o login podia cair na tela do React Router com `error loading dynamically imported module: /node_modules/.vite/deps/gsap.js?...`. Como o hook da intro importava `gsap`/`@gsap/react` no topo, uma falha de cache/prebundle do Vite derrubava o chunk do `AppLayout` antes do fallback visual da intro conseguir agir.
- **Decisao:** Manter GSAP como engine da intro, mas carregar o runtime `gsap` dinamicamente dentro de `usePortalSamaGsapTimeline`, com `try/catch`, `gsap.context` escopado e cleanup no unmount. Falha de import/inicializacao passa a acionar fallback estatico. O router tambem ganhou `RouteErrorPage` para falhas de lazy routes.
- **Alternativas consideradas:** Manter import top-level e depender apenas de limpar cache Vite; criar uma segunda intro sem GSAP; voltar para Framer Motion; manter somente `@gsap/react` no caminho critico.
- **Consequencias positivas:** O `AppLayout` nao quebra se o dep otimizado do GSAP falhar; falha visual da intro nao vira falha de autenticacao/navegacao; o cache Vite stale deixa de bloquear a rota autenticada.
- **Consequencias negativas:** A timeline usa `gsap.context` via `useEffect` em vez de `useGSAP` no caminho critico. A decisao e intencional para manter a falha do runtime capturavel pelo React e pela propria intro.
- **Arquivos impactados:** `portal-sama-web/src/features/intro/hooks/usePortalSamaGsapTimeline.ts`, `portal-sama-web/src/app/router.tsx`, `portal-sama-web/src/pages/errors/RouteErrorPage.tsx`.
- **Referencias:** `docs/design/intro-gsap-V2/documentacao_refatoracao_intro_portal_sama_gsap.md`, `docs/SEGURANCA.md`, `.ai-tests/runs/2026-05-21-1424-gsap-import-guard/summary.md`.

## ADR-0026 - Intro oficial GSAP v2 por manifesto e assets locais

- **Data:** 2026-05-21
- **Status:** Aceita
- **Contexto:** A intro anterior do `portal-sama-web` estava baseada em Framer Motion, SVGs/componentes visuais antigos e `layer-manifest.json`. A documentacao em `docs/design/intro-gsap-V2` definiu a nova direcao: GSAP + `@gsap/react`, `manifest.json`, `animation-map.json`, assets WebP/PNG/SVG simples locais e fallback estatico seguro.
- **Decisao:** Manter a feature oficial em `portal-sama-web/src/features/intro`, substituir a experiencia visual por `PortalSamaIntro`/renderer GSAP, consumir `/brand/sama/intro/manifest.json` e `/brand/sama/intro/animation-map.json`, publicar os assets finais em `/brand/sama/intro/assets`, remover a camada Framer visual e versionar a intro como `sama-gsap-layers-v2`.
- **Alternativas consideradas:** Continuar evoluindo a intro Framer antiga; manter duas engines durante a transicao; criar uma segunda pasta paralela de intro; usar assets remotos/CDN.
- **Consequencias positivas:** Reduz duplicidade visual, centraliza ordem/timeline em arquivos declarativos, melhora seguranca de paths, respeita reduced motion e preserva as integracoes funcionais ja auditaveis.
- **Consequencias negativas:** QA visual real ainda depende de navegador/homologacao; Playwright visual especifico ainda precisa ser criado; timings podem exigir refinamento fino apos avaliacao visual.
- **Arquivos impactados:** `portal-sama-web/src/features/intro/**`, `portal-sama-web/public/brand/sama/intro/**`, `portal-sama-web/src/components/layout/AppLayout.tsx`, `portal-sama-web/package.json`.
- **Referencias:** `docs/design/intro-gsap-V2/README.md`, `docs/design/intro-gsap-V2/documentacao_refatoracao_intro_portal_sama_gsap.md`, `docs/design/intro-gsap-V2/portal_sama_intro_codex_handoff.md`, `docs/SEGURANCA.md`.

## ADR-0025 - Padrao visual React primeiro, sem reestilizar o legado

- **Data:** 2026-05-21
- **Status:** Aceita parcialmente
- **Contexto:** A documentacao de design definiu um padrao premium para o `portal-sama-web`, enquanto o legado HTML/PHP continua ativo e nao deve ser reestilizado nesta fase. Varias telas React ja usam classes compartilhadas (`panel`, `stat-grid`, `quick-link-card`, `status-badge`) e o menu lateral depende de permissoes.
- **Decisao:** Evoluir primeiro a base visual do `portal-sama-web`: `AppLayout` com sidebar recolhivel, `Sidebar` premium preservando `NavLink`/`PermissionGate`, `Header` padronizado, componentes reutilizaveis (`PageShell`, `AppPanel`, `IconBadge`, `KpiCard`, `ShortcutCard`) e assets decorativos novos em `public/brand/sama/dashboard`. A Home foi a primeira tela aplicada; as demais herdam melhorias globais e devem migrar gradualmente para os componentes base.
- **Alternativas consideradas:** Reestilizar o legado junto; refatorar todas as telas React de uma vez; criar uma biblioteca visual externa antes da estabilizacao; substituir menus/cards por imagens estaticas.
- **Consequencias positivas:** Reduz duplicacao visual, preserva navegacao/permissoes reais, cria base reutilizavel para proximas telas e melhora a coerencia com a intro sem alterar regras de negocio.
- **Consequencias negativas:** O acabamento completo das demais telas ainda depende de propagacao gradual; QA visual especifico de sidebar/home/mobile segue pendente; assets da intro ja deletados no workspace continuam exigindo decisao separada.
- **Arquivos impactados:** `portal-sama-web/src/components/layout/**`, `portal-sama-web/src/components/ui/**`, `portal-sama-web/src/pages/home/HomePage.tsx`, `portal-sama-web/src/index.css`, `portal-sama-web/public/brand/sama/dashboard/**`.
- **Referencias:** `docs/design/codex_estilizacao_padrao_portal_sama.md`, `docs/paginas/home-inicio.md`, `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`.

## ADR-0024 - Integra-AI em fatia read-only antes de migrar upload/parser

- **Data:** 2026-05-21
- **Status:** Aceita parcialmente
- **Contexto:** `Contabil/integra-ai.html` e `api/integra_ai.php` manipulam extratos, regras contabeis, preview e TXT Dominio. A migracao completa envolve upload de arquivo, parser, gravacao de regras e download financeiro sensivel, portanto antecipar mutacoes sem isolar storage/parser aumentaria o risco.
- **Decisao:** Criar primeiro `AccountingModule` com `GET /api-v2/accounting/integra-ai/workspaces` e `GET /api-v2/accounting/integra-ai/jobs/:id`, protegidos por JWT, RBAC `accounting.integra_ai.read` e escopo contabil. A leitura usa SQL parametrizado sobre tabelas legadas `sama_integra_ai_*`, retorna somente campos sanitizados, remove `source_path`, `txt_path`, payload bruto e conteudo TXT, e audita abertura de detalhe de job. A rota React `/contabil/integra-ai` consome essa leitura e mantem link para o legado nas operacoes ainda nao migradas.
- **Alternativas consideradas:** Migrar todo o fluxo de uma vez; chamar o PHP legado diretamente pelo React; modelar todas as tabelas no Prisma antes da primeira tela; liberar download TXT pela API v2 desde a primeira fatia.
- **Consequencias positivas:** Reduz dependencia de leitura do PHP, entrega visibilidade operacional no React, diminui risco de vazamento por paths privados e prepara testes/contratos antes das mutacoes.
- **Consequencias negativas:** Na decisao inicial, upload/parser/regras/geracao/download TXT ficaram no legado; apos a evolucao de 2026-05-27, essas mutacoes existem parcialmente na API v2, mas ainda dependem de homologacao real, backfills/conferencia e validacao MySQL/storage/parser antes do corte. A modelagem Prisma formal das tabelas Integra-AI ainda precisa ser decidida.
- **Atualizacao 2026-05-27 10:03:** a decisao evoluiu para mutacoes API v2 com upload/parser/export/download. OFX foi publicado como capability opt-in por `SAMA_INTEGRA_AI_OFX_IMPORT_ENABLED`, sem migration de banco, mantendo PDF como padrao e exigindo homologacao real antes de desligar o PHP.
- **Arquivos impactados:** `portal-sama-api/src/modules/accounting/**`, `portal-sama-api/src/modules/rbac/default-rbac.ts`, `portal-sama-web/src/pages/accounting/IntegraAiPage.tsx`, `portal-sama-web/src/services/integra-ai.service.ts`, `portal-sama-web/src/types/integra-ai.ts`.
- **Referencias:** `docs/paginas/contabil-integra-ai.md`, `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`, `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`, `docs/INVENTARIO_SEGURANCA.md`, `api/integra_ai.php`, `api/db.php`.

## ADR-0023 - SSE e Web Push no React sem bearer em query string

- **Data:** 2026-05-20
- **Status:** Aceita parcialmente
- **Contexto:** O `NotificationsModule` ja possuia SSE autenticado, Web Push e teste operacional, mas a tela React `/notificacoes` ainda consumia apenas listagem/acoes por HTTP comum. `EventSource` nativo nao permite enviar `Authorization` header, e colocar access token na query string aumentaria exposicao em logs/historico.
- **Decisao:** Implementar o consumo de `GET /api-v2/notifications/stream` por `fetch` com `Accept: text/event-stream`, `Authorization: Bearer ...`, `credentials: include` e parser SSE no service React. A assinatura Web Push passa a ser ativada/desativada na tela React por gesto do usuario, com CSRF via `issueCsrfToken()` e Service Worker servido em `portal-sama-web/public/sama-push-sw.js`.
- **Alternativas consideradas:** Usar `EventSource` com token na URL; manter SSE apenas no legado; criar cookie de access token legivel pelo backend; adiar Web Push React ate Playwright.
- **Consequencias positivas:** Mantem access token apenas em memoria, evita bearer em query string, aproxima o React da capacidade do legado e permite validar Push API diretamente na central nova.
- **Consequencias negativas:** Ainda depende de HTTPS/homologacao, VAPID real, permissao do navegador, teste com portal fechado, Playwright e decisao de broker/event bus para multi-instancia.
- **Arquivos impactados:** `portal-sama-web/src/pages/notifications/NotificationsPage.tsx`, `portal-sama-web/src/services/notifications.service.ts`, `portal-sama-web/src/types/notifications.ts`, `portal-sama-web/public/sama-push-sw.js`.
- **Referencias:** `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`, `docs/INVENTARIO_ENDPOINTS.md`, `docs/SEGURANCA.md`, `api/notifications_stream.php`, `global.js`.

## ADR-0022 - Exclusao auditavel de vencimentos no CalendarModule

- **Data:** 2026-05-20
- **Status:** Aceita parcialmente
- **Contexto:** A leitura/configuracao do calendario do gestor ja estava em `CalendarModule`, mas a action `calendar_delete_entry` ainda mantinha uma mutacao critica em `api/manager_workspace.php`. O dashboard React exibia proximos vencimentos sem permitir remocao pela API v2.
- **Decisao:** Criar `DELETE /api-v2/calendar/entries/:id`, protegido por JWT, `calendar.manage`, papel ADMIN/DEV/MANAGER, CSRF e escopo por departamento. A exclusao remove o vinculo `sama_calendar_rule_companies`, limpa `sama_calendar_due_notified`, remove a regra quando fica sem empresas vinculadas e registra auditoria `calendar.entry.delete`. O frontend `/manager` passa a exibir a acao somente para quem possui `calendar.manage`.
- **Alternativas consideradas:** Manter exclusao apenas na tela de configuracao por replace completo; chamar a action PHP a partir do React; implementar soft delete antes de validar o legado; permitir exclusao apenas para ADMIN/DEV.
- **Consequencias positivas:** Reduz dependencia do endpoint action-based, evita replace amplo para excluir um unico vencimento, preserva limpeza de notificacoes e cria trilha auditavel centralizada.
- **Consequencias negativas:** Ainda depende de MySQL/homologacao, seed RBAC real, Playwright e validacao de dados reais de calendario/carteira antes de desligar `calendar_delete_entry` no PHP.
- **Arquivos impactados:** `portal-sama-api/src/modules/calendar/**`, `portal-sama-web/src/pages/manager/ManagerDashboardPage.tsx`, `portal-sama-web/src/services/calendar.service.ts`, `portal-sama-web/src/types/calendar.ts`.
- **Referencias:** `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`, `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`, `docs/INVENTARIO_ENDPOINTS.md`, `docs/paginas/manager-manager.md`, `api/manager_workspace.php`.

## ADR-0021 - Overview do gestor na API v2

- **Data:** 2026-05-20
- **Status:** Aceita
- **Contexto:** Depois das fatias de transferencias, calendario e historico/vida da empresa, a action `overview` ainda mantinha o resumo operacional do gestor em `api/manager_workspace.php`, incluindo KPIs de equipe e presenca por `sama_user_presence`.
- **Decisao:** Criar `GET /api-v2/managers/overview` dentro do `ManagersModule`, protegido por JWT, `manager_history.read`, papel ADMIN/DEV/MANAGER e escopo por departamento. Modelar `sama_user_presence` como `UserPresence` no Prisma, com migration idempotente, e conectar `/manager` a esse contrato para exibir presenca da equipe.
- **Consequencias positivas:** Reduz a ultima dependencia direta de leitura do manager workspace, reaproveita o escopo do `ManagersModule`, preserva o formato operacional do legado e evita duplicar a logica de presenca no frontend.
- **Consequencias negativas:** A presenca ainda depende do ping legado (`api/storage.php?action=user_presence_ping`/streams) ate existir contrato v2 proprio; a action PHP so deve ser desligada apos homologacao com dados reais.
- **Referencias:** `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`, `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`, `docs/INVENTARIO_ENDPOINTS.md`, `docs/paginas/manager-manager.md`, `api/manager_workspace.php`.

## ADR-0020 - Historico operacional do gestor com escrita auditavel

- **Data:** 2026-05-20
- **Status:** Aceita parcialmente
- **Contexto:** `Manager/manager.html` e `api/manager_workspace.php` ainda concentravam leitura e escrita de historico operacional (`history_list`, `history_timeline`, `history_append`, `history_update`) e vida da empresa (`company_life_save`) em um endpoint action-based. A primeira fatia do `ManagersModule` ja cobria leitura, mas criacao/edicao ainda dependiam do PHP legado.
- **Decisao:** Estender `ManagersModule` com `POST /api-v2/managers/history`, `PATCH /api-v2/managers/history/:id` e `POST /api-v2/managers/company-life`, usando os modelos Prisma legados `CompanyHistoryEntry` e `CompanyLifeEntry`. Criar a permissao `manager_history.write`, exigir CSRF em mutacoes, validar papel ADMIN/DEV/MANAGER e escopo por departamento, e registrar auditoria via `AuditService`.
- **Alternativas consideradas:** Manter escrita no PHP enquanto a leitura fica no NestJS; criar um modulo separado apenas para vida da empresa; aguardar modelagem relacional nova antes de gravar; permitir edicao apenas para ADMIN/DEV.
- **Consequencias positivas:** Reduz dependencia de `api/manager_workspace.php`, preserva tabelas legadas, cria trilha auditavel e libera `/manager/historico` para fluxo operacional real sem `innerHTML` ou storage local.
- **Consequencias negativas:** Ainda depende de MySQL/homologacao, seed RBAC atualizado, dados reais de departamento/carteira e Playwright; o desligamento do legado depende tambem da homologacao do novo overview.
- **Arquivos impactados:** `portal-sama-api/src/modules/managers/**`, `portal-sama-api/src/modules/rbac/default-rbac.ts`, `portal-sama-web/src/pages/manager/ManagerHistoryPage.tsx`, `portal-sama-web/src/services/manager-history.service.ts`, `portal-sama-web/src/types/manager-history.ts`.
- **Referencias:** `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`, `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`, `docs/INVENTARIO_ENDPOINTS.md`, `docs/paginas/manager-manager.md`, `api/manager_workspace.php`.

## ADR-0019 - Transferencias de carteira em modulo NestJS dedicado

- **Data:** 2026-05-19
- **Status:** Aceita parcialmente
- **Contexto:** `Manager/manager-transfers.html` depende das actions `transfer_dashboard`, `transfer_submit` e `transfer_return` em `api/manager_workspace.php`, manipulando clientes, colaboradores, periodo, justificativa e sessoes de transferencia. A estrategia definida foi criar primeiro o backend transacional com permissao granular e auditoria, e so entao conectar a tela React.
- **Decisao:** Criar `TransfersModule` com rotas `/api-v2/transfers`, permissoes `transfers.read`, `transfers.create` e `transfers.return`, CSRF em mutacoes, auditoria e escopo por ADMIN/DEV/MANAGER + departamento. A tabela legada `sama_transfer_sessions` foi mapeada como `TransferSession`; a carteira continua temporariamente em metadata de `Client` e `User` para preservar compatibilidade ate o backfill/modelagem formal.
- **Alternativas consideradas:** Migrar primeiro a tela React chamando PHP; criar tabela formal de carteira antes de entregar o contrato; embutir transferencias no `ClientsModule`; adiar toda a frente de transferencias.
- **Consequencias positivas:** Reduz dependencia do endpoint action-based, cria contrato auditavel para a futura tela, preserva o legado durante a transicao e permite validar regras criticas em testes unitarios.
- **Consequencias negativas:** Ainda depende de MySQL real, seed RBAC em ambiente, backfill de metadata, validacao de carteira real e futura modelagem relacional se os dados legados exigirem.
- **Arquivos impactados:** `portal-sama-api/src/modules/transfers/**`, `portal-sama-api/prisma/schema.prisma`, `portal-sama-api/prisma/migrations/20260519120000_add_transfer_sessions/migration.sql`, `portal-sama-api/src/modules/rbac/default-rbac.ts`, `portal-sama-api/src/app.module.ts`.
- **Referencias:** `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`, `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`, `docs/INVENTARIO_ENDPOINTS.md`, `docs/paginas/manager-transfers.md`, `api/transfer_session_lib.php`, `api/manager_workspace.php`.
- **Atualizacao 2026-05-19 15:33:** Com o contrato backend transacional ja criado, a primeira tela React `/manager/transferencias` foi implementada em `portal-sama-web/src/pages/manager/ManagerTransfersPage.tsx`, consumindo `TransfersModule` por `transfers.service.ts`. A decisao de manter validacao, RBAC, CSRF, escopo e auditoria no backend permanece; faltam homologacao MySQL, usuarios reais, backfill/modelagem de carteira e Playwright.
- **Atualizacao 2026-05-19 15:53:** A rota `/manager/colaboradores` foi criada como consulta de carteira por colaborador usando o mesmo `GET /api-v2/transfers/dashboard`, sem mutacoes no frontend. A decisao evita criar um contrato de `ManagersModule` incompleto enquanto historico/calendario ainda nao foram migrados.

## ADR-0018 - Fundacao React separada em `portal-sama-web`

- **Data:** 2026-05-14
- **Status:** Aceita parcialmente
- **Contexto:** A documentacao define o frontend alvo como React + TypeScript + Vite e a pendencia `Criar frontend React/Vite` ainda estava nao iniciada, enquanto as principais fatias backend em `/api-v2` ja tinham base de autenticacao, RBAC, auditoria e modulos de negocio.
- **Decisao:** Criar `portal-sama-web/` como projeto separado de Vite React, mantendo o legado HTML/PHP ativo. A primeira fatia inclui login em `/login`, layout autenticado, roteamento das telas mapeadas, Axios centralizado, TanStack Query, Zustand, React Hook Form/Zod e access token apenas em memoria. O refresh token continua em cookie HttpOnly da API v2.
- **Alternativas consideradas:** Migrar telas diretamente no legado; criar monorepo `apps/web` antes de existir uma base funcional; aguardar homologacao completa de todos os modulos backend antes de iniciar o frontend; armazenar access token em `sessionStorage` como ponte temporaria.
- **Consequencias positivas:** Desbloqueia a migracao visual com um app versionavel, preserva o legado, respeita a estrategia de auth alvo e cria base para Playwright e integracoes por modulo.
- **Consequencias negativas:** As telas de negocio ainda sao placeholders; o fluxo real depende de API v2, MySQL/homologacao, CORS/cookies em HTTPS e migracao gradual das telas.
- **Arquivos impactados:** `portal-sama-web/**`, `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`, `docs/EASYPANEL_DEPLOY.md`, `docs/SEGURANCA.md`.
- **Referencias:** `docs/ROADMAP_REFATORACAO.md`, `docs/GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`, `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`, `docs/paginas/index.md`, `docs/paginas/home-inicio.md`.

## ADR-0017 - Colaboradores como perfil operacional de User na primeira fatia

- **Data:** 2026-05-13
- **Status:** Aceita parcialmente
- **Contexto:** As telas `Client/visao-geral-colaboradores.html`, `Client/visao-colaborador.html`, `DEV/dev-colaborador.html`, `DEV/dev-novo-colaborador.html` e `Manager/manager-colaborador.html` precisam de dados funcionais de colaboradores, mas o inventario ainda marca `sama_colaboradores`, vinculos cliente-usuario/gestor e carteira como ponto a validar.
- **Decisao:** Criar `CollaboratorsModule` usando `User` como base da primeira fatia, expandindo `User` com campos funcionais (`position`, `phone`, `extension`, `metadata`, `archivedAt`, `archivedById`) e expondo rotas REST protegidas por JWT/permissoes `collaborators.*`. Mutacoes exigem CSRF e auditoria; leitura aplica escopo por departamento para perfis nao privilegiados. A role `CLIENT` e bloqueada neste modulo.
- **Alternativas consideradas:** Criar uma tabela `collaborators` separada antes do backfill; reutilizar apenas `UsersModule`; adiar colaboradores ate modelar carteira/transferencias; permitir role `CLIENT` em colaborador para reaproveitar cadastro.
- **Consequencias positivas:** Entrega contrato seguro para telas de colaboradores sem duplicar identidade/login, preserva aliases legados e prepara backfill gradual de `sama_colaboradores`.
- **Consequencias negativas:** Carteira, transferencias e vinculos cliente-gestor seguem pendentes; `User` fica com campos operacionais que devem ser revisitados se o backfill indicar necessidade de tabela separada.
- **Arquivos impactados:** `portal-sama-api/src/modules/collaborators/**`, `portal-sama-api/prisma/schema.prisma`, `portal-sama-api/prisma/migrations/20260513124500_add_collaborator_profile_to_users/migration.sql`, `portal-sama-api/src/modules/rbac/default-rbac.ts`, `portal-sama-api/src/app.module.ts`, `portal-sama-api/test/app.e2e-spec.ts`.
- **Referencias:** `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`, `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`, `docs/INVENTARIO_ENDPOINTS.md`, `docs/paginas/dev-novo-colaborador.md`, `docs/paginas/client-visao-geral-colaboradores.md`, `docs/paginas/client-visao-colaborador.md`, `docs/paginas/manager-colaborador.md`.

## ADR-0016 - Clientes com perfil operacional expandido e baseline Prisma local

- **Data:** 2026-05-13
- **Status:** Aceita parcialmente
- **Contexto:** As telas `Client/clientes.html`, `Client/painel.html`, `Client/visao-geral-clientes.html` e `DEV/dev-novo-cliente.html` dependem de dados de cliente mantidos em armazenamento legado/action-based. A migracao tambem estava bloqueada porque o MySQL local tinha tabelas fora do historico Prisma e 5 migrations pendentes.
- **Decisao:** Resolver o baseline local de forma controlada, aplicar a migration incremental `20260513113000_expand_clients_profile` e criar `ClientsModule` com rotas REST protegidas por JWT/permissoes, CSRF em mutacoes e auditoria. O modelo `Client` foi expandido com campos operacionais do legado e soft delete (`deletedAt`/`deletedById`), mantendo aliases de resposta para reduzir atrito na futura integracao.
- **Alternativas consideradas:** Continuar usando `api/storage.php?action=get/set/delete`; iniciar frontend antes do backend de clientes; criar tabelas separadas para cada tela DEV/Cliente; adiar baseline do banco local.
- **Consequencias positivas:** Desbloqueia migrations locais, reduz dependencia do endpoint generico, cria contrato tipado para clientes e prepara dashboard de cliente com totais de documentos/certificados/propostas/contratos.
- **Consequencias negativas:** Ainda faltam backfill de clientes/colaboradores, modelagem de vinculos cliente-usuario/gestor, validacao em homologacao/producao e integracao das telas.
- **Arquivos impactados:** `portal-sama-api/src/modules/clients/**`, `portal-sama-api/prisma/schema.prisma`, `portal-sama-api/prisma/migrations/20260513113000_expand_clients_profile/migration.sql`, `portal-sama-api/src/app.module.ts`, `portal-sama-api/test/app.e2e-spec.ts`.
- **Referencias:** `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`, `docs/INVENTARIO_ENDPOINTS.md`, `docs/paginas/dev-novo-cliente.md`, `docs/paginas/client-clientes.md`, `docs/paginas/client-painel.md`.

## ADR-0015 — Solicitacoes de acesso com escopo por gestor/departamento

- **Data:** 2026-05-13
- **Status:** Aceita parcialmente
- **Contexto:** `SolicitacaoAcesso/solicitacao-acesso.html`, `SolicitacaoAcesso/enviar_acesso.php` e `api/access_requests.php` sustentam um fluxo sensivel de pedido de acesso temporario, aprovacao de gestor e visao operacional de TI/master. O legado usa actions como `list`, `gestor_approvals`, `colaborador_status` e `decision`, com risco de IDOR se o escopo for derivado do cliente.
- **Decisao:** Criar `AccessRequestsModule` com rotas internas protegidas por JWT, permissoes granulares (`access_requests.read`, `access_requests.approve`, `access_requests.reject`), CSRF em criacao/decisao, auditoria e modelo Prisma `AccessRequest`. Solicitacoes de colaborador ficam em `PENDING_MANAGER` e sao vinculadas ao gestor ativo do mesmo departamento; solicitacoes de gestor validam colaboradores ativos do mesmo departamento e seguem como `APPROVED` para tratamento operacional. Respostas mantem aliases legados para facilitar a futura adaptacao da tela.
- **Alternativas consideradas:** Manter apenas o legado endurecido ate o React; criar modulo generico de TI antes das solicitacoes; permitir aprovacao por qualquer gestor com permissao; persistir decisoes apenas em metadata.
- **Consequencias positivas:** Reduz dependencia de endpoints action-based, centraliza regras de horario/data, melhora escopo por usuario/departamento e cria trilha auditavel para aprovacao/rejeicao.
- **Consequencias negativas:** Ainda depende de baseline/migrations em MySQL real, backfill de `sama_access_requests`, integracao com notificacoes e frontend React/legado, e historico estruturado separado para decisoes.
- **Arquivos impactados:** `portal-sama-api/src/modules/access-requests/**`, `portal-sama-api/prisma/schema.prisma`, `portal-sama-api/prisma/migrations/20260513101500_add_access_requests/migration.sql`, `portal-sama-api/src/app.module.ts`, `portal-sama-api/test/app.e2e-spec.ts`.
- **Referencias:** `docs/SEGURANCA.md`, `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`, `docs/paginas/solicitacao-acesso.md`, `docs/paginas/ti-acesso-ti.md`.

## ADR-0014 — Contratos com assinatura pública por token opaco

- **Data:** 2026-05-13
- **Status:** Aceita parcialmente
- **Contexto:** `Legalizacao/contrato.html` e `Legalizacao/assinatura.html` operam um fluxo crítico de contrato editável, HTML rico, link público e assinatura. O backend legado concentra essas actions em `api/legalizacao.php`, incluindo `get`, `update`, `send`, `getByToken` e `clientSubmit`.
- **Decisão:** Criar `ContractsModule` com rotas internas protegidas por JWT/permissões (`contracts.read`, `contracts.generate`, `contracts.sign`), CSRF em mutações internas, auditoria e modelo Prisma `Contract`. O envio para assinatura emite token opaco `sama_pub_*`, persiste apenas SHA-256 em `PublicToken`, revoga tokens ativos anteriores do mesmo contrato e expõe `GET /api-v2/public/signatures/:token` e `POST /api-v2/public/signatures/:token/sign`. A assinatura pública grava hash do documento/assinatura, IP e user-agent, revogando o token após uso.
- **Alternativas consideradas:** Migrar apenas a assinatura pública antes do contrato interno; manter contratos em KV/legado até o React; criar tabela separada para tokens de assinatura; apenas mascarar token legado sem trocar o modelo.
- **Consequências positivas:** Reduz dependência do endpoint action-based, evita token bruto persistido, cria trilha de auditoria e prepara as telas `Legalizacao/contrato.html` e `Legalizacao/assinatura.html` para contrato estável.
- **Consequências negativas:** Ainda depende de baseline/migrations em MySQL real, backfill de contratos legados, import/render PDF seguro, storage privado do PDF assinado e integração frontend.
- **Arquivos impactados:** `portal-sama-api/src/modules/contracts/**`, `portal-sama-api/prisma/schema.prisma`, `portal-sama-api/prisma/migrations/20260513092300_add_contracts/migration.sql`, `portal-sama-api/src/app.module.ts`, `portal-sama-api/test/app.e2e-spec.ts`.
- **Referências:** `docs/SEGURANCA.md`, `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`, `docs/paginas/legalizacao-contrato.md`, `docs/paginas/legalizacao-assinatura.md`.

## ADR-0013 — Propostas com token público opaco e respostas legadas compatíveis

- **Data:** 2026-05-13
- **Status:** Aceita parcialmente
- **Contexto:** Após documentos, certificados e RBAC, a próxima etapa prioritária da migração backend é substituir gradualmente `api/propostas.php`, que usa KV legado e links públicos de proposta, por contratos tipados em `/api-v2`.
- **Decisão:** Criar `ProposalsModule` com rotas internas protegidas por JWT/permissões (`proposals.read`, `proposals.create`, `proposals.approve`, `proposals.reject`), CSRF em mutações internas, auditoria e modelo Prisma `Proposal`. O envio de proposta emite token opaco `sama_pub_*`, persiste apenas SHA-256 em `PublicToken`, revoga tokens ativos anteriores da mesma proposta e expõe `GET/POST /api-v2/public/proposals/:token` para visualização/aprovação/ajuste público. As respostas mantêm aliases legados (`company_name`, `tipo_proposta`, `proposal_status`, `proposal_fields`) para reduzir atrito na futura integração.
- **Alternativas consideradas:** Manter propostas apenas no KV legado até o React; migrar contratos/assinaturas antes de proposta; criar tabela separada de tokens por módulo; retornar apenas nomenclatura nova em inglês.
- **Consequências positivas:** Reduz dependência do endpoint action-based, evita token bruto persistido, cria trilha de auditoria e prepara a tela `Legalizacao/proposta.html`/futura React para contrato estável.
- **Consequências negativas:** Ainda depende de `migrate deploy` em MySQL de homologação, backfill de `sama_propostas_v1`, compatibilidade com tokens legados e integração frontend; não implementa contratos/assinaturas.
- **Arquivos impactados:** `portal-sama-api/src/modules/proposals/**`, `portal-sama-api/prisma/schema.prisma`, `portal-sama-api/prisma/migrations/20260513085700_add_proposals/migration.sql`, `portal-sama-api/src/app.module.ts`, `portal-sama-api/test/app.e2e-spec.ts`.
- **Referências:** `docs/SEGURANCA.md`, `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`, `docs/paginas/legalizacao-proposta.md`, `docs/paginas/onboarding-proposta-cliente.md`.

## ADR-0012 — Mutações administrativas de roles e permissões com proteção de autogestão

- **Data:** 2026-05-12
- **Status:** Aceita parcialmente
- **Contexto:** Após as mutações de usuários, a futura tela DEV/admin ainda dependia de contratos seguros para criar/editar roles, trocar permissões de uma role e manter o catálogo de permissões sem voltar ao endpoint genérico `api/storage.php`.
- **Decisão:** Implementar no `RolesModule` as rotas `POST /api-v2/roles`, `PATCH /api-v2/roles/:id` e `PUT /api-v2/roles/:id/permissions`, e no `PermissionsModule` as rotas `POST /api-v2/permissions` e `PATCH /api-v2/permissions/:id`. Todas as mutações exigem JWT, permissão granular, CSRF e auditoria. Roles validam permissões existentes; a troca de permissões bloqueia remover `roles.permissions` da própria role do ator; permissões padrão do catálogo não podem ter a chave renomeada.
- **Alternativas consideradas:** Deixar roles/permissões somente leitura até o frontend React; permitir edição livre do catálogo; implementar exclusão de permissões agora.
- **Consequências positivas:** Fecha a principal superfície administrativa de RBAC na API v2, reduz risco de perda acidental de acesso administrativo e mantém trilha auditável para futuras telas DEV/admin.
- **Consequências negativas:** Ainda depende de validação com MySQL/homologação, migrations/seed atualizado, migração de usuários legados e integração frontend; exclusão de permissões foi adiada por risco operacional.
- **Arquivos impactados:** `portal-sama-api/src/modules/roles/**`, `portal-sama-api/src/modules/permissions/**`, `portal-sama-api/src/modules/rbac/default-rbac.ts`, `portal-sama-api/test/app.e2e-spec.ts`.
- **Referências:** `docs/INVENTARIO_ENDPOINTS.md`, `docs/PENDENCIAS_TECNICAS.md`, `docs/SEGURANCA.md`, `docs/ROADMAP_REFATORACAO.md`.

## ADR-0011 — Mutações administrativas de usuários com CSRF e auditoria

- **Data:** 2026-05-12
- **Status:** Aceita parcialmente
- **Contexto:** O legado concentra administração de usuários em actions genéricas de `api/storage.php`. A API v2 já possuía leitura de usuários, roles e permissões, mas ainda faltavam mutações seguras para desbloquear a futura tela DEV/admin.
- **Decisão:** Implementar no `UsersModule` as rotas `POST /api-v2/users`, `PATCH /api-v2/users/:id`, `PATCH /api-v2/users/:id/status` e `PUT /api-v2/users/:id/roles`, protegidas por JWT, permissões granulares, CSRF e auditoria. Senhas são hasheadas com `bcrypt`, roles são validadas antes de persistir e a resposta nunca seleciona `passwordHash`.
- **Alternativas consideradas:** Aguardar o frontend React; manter criação/edição apenas no PHP; implementar mutações de roles/permissions no mesmo bloco.
- **Consequências positivas:** Reduz dependência do endpoint action-based, cria trilha de auditoria para alterações administrativas e preserva segurança de senha/roles desde o service.
- **Consequências negativas:** Ainda depende de validação com MySQL real, migração de usuários legados, migrations/seed atualizado e integração do frontend DEV/admin.
- **Arquivos impactados:** `portal-sama-api/src/modules/users/**`, `portal-sama-api/test/app.e2e-spec.ts`.
- **Referências:** `docs/INVENTARIO_ENDPOINTS.md`, `docs/PENDENCIAS_TECNICAS.md`, `docs/ROADMAP_REFATORACAO.md`.

## ADR-0010 — Tokens públicos de documentos com segredo opaco e gestão administrativa

- **Data:** 2026-05-12
- **Status:** Aceita parcialmente
- **Contexto:** O upload público de onboarding precisa aceitar documentos de cliente externo sem sessão interna, mas sem expor IDs diretos nem persistir token bruto. Após criar `PublicToken` e `POST /api-v2/documents/public-upload`, faltava governança interna para emitir, listar e revogar os links.
- **Decisão:** Usar tokens opacos `sama_pub_*`, persistir apenas SHA-256 em `PublicToken.tokenHash`, retornar o token bruto somente no `POST /api-v2/documents/public-tokens`, listar metadados sem hash/segredo em `GET /api-v2/documents/public-tokens` e revogar por `DELETE /api-v2/documents/public-tokens/:id`. As rotas administrativas exigem `documents.public_tokens`; mutações exigem CSRF e auditoria.
- **Alternativas consideradas:** Reusar token como ID público; expor `tokenHash` para o frontend interno; criar tabela separada por módulo; adiar emissão administrativa até o frontend React.
- **Consequências positivas:** O fluxo público fica operável sem armazenar segredo bruto, mantém revogação e expiração centralizadas, e dá rastreabilidade administrativa antes da integração do frontend.
- **Consequências negativas:** Migration/seed/backfill, limpeza de tokens expirados, rate limit distribuído e validação com MySQL real ainda dependem da etapa de homologação.
- **Arquivos impactados:** `portal-sama-api/src/modules/documents/**`, `portal-sama-api/src/modules/rbac/default-rbac.ts`, `portal-sama-api/test/app.e2e-spec.ts`, `portal-sama-api/prisma/schema.prisma`.
- **Referências:** `docs/SEGURANCA.md`, `docs/INVENTARIO_ENDPOINTS.md`, `docs/paginas/onboarding-documentos-cliente.md`, `docs/BANCO_DADOS_MYSQL_PRISMA.md`.

## ADR-0009 — Quarentena e scanner configuravel para upload de documentos

- **Data:** 2026-05-11
- **Status:** Aceita parcialmente
- **Contexto:** `docs/SEGURANCA.md`, `docs/INVENTARIO_SEGURANCA.md` e o legado `api/client_documents_lib.php` exigem que uploads passem por quarentena e scanner antes do storage definitivo. O `DocumentsModule` ja validava extensao, MIME, assinatura, tamanho e hash, mas ainda salvava diretamente no storage privado.
- **Decisao:** Criar `DocumentUploadScannerService` no `DocumentsModule`. O service grava o buffer em quarentena privada, executa regras estaticas para texto/PDF/Office zip-like, tenta `clamscan`/`clamdscan` configuravel e so permite `DocumentStorageService.storeClientDocument` depois da aprovacao. O modo `best_effort` permite fallback para regras estaticas quando o scanner externo esta indisponivel; o modo `strict` bloqueia indisponibilidade do scanner.
- **Alternativas consideradas:** Bloquear todo upload ate ClamAV estar instalado localmente; manter apenas validacao de assinatura; fazer scanner assíncrono por fila; migrar exatamente o helper PHP antes de criar service NestJS.
- **Consequencias positivas:** O fluxo NestJS fica alinhado ao desenho de upload seguro, nao persiste arquivo final quando a varredura rejeita, permite homologacao sem ClamAV em `best_effort` e prepara producao para `strict`.
- **Consequencias negativas:** A validacao real de ClamAV/EICAR ainda depende do host; regras estaticas nao substituem antivirus; filas/retencao formal de quarentena e historico estruturado ainda podem ser evoluidos.
- **Arquivos impactados:** `portal-sama-api/src/modules/documents/**`, `portal-sama-api/src/config/env.schema.ts`, `portal-sama-api/.env.example`, `.env.example`.
- **Referencias:** `docs/SEGURANCA.md`, `docs/INVENTARIO_SEGURANCA.md`, `docs/INVENTARIO_ENDPOINTS.md`, `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`, `api/client_documents_lib.php`, `scripts/check_upload_scanner.php`.

## ADR-0008 — Revisao de status de documentos na API v2

- **Data:** 2026-05-11
- **Status:** Aceita parcialmente
- **Contexto:** `docs/MAPEAMENTO_MIGRACAO_APIS.md` ja mapeava `PATCH /api-v2/documents/:id/status` como pendencia do `DocumentsModule`. O legado confirma documentos em `api/onboarding.php?action=docs_confirm`, exigindo CSRF, escopo de gestao/departamento e trilha de evento. A API v2 ja possuia `Document.status`, mas ainda nao havia rota para aprovacao/rejeicao.
- **Decisao:** Criar `PATCH /api-v2/documents/:id/status` com JWT, permissao `documents.review`, CSRF, validacao de escopo por papel/departamento e auditoria. A rota aceita `PENDING`, `APPROVED` e `REJECTED`; `ARCHIVED` continua reservado ao `DELETE /api-v2/documents/:id`. A revisao imediata fica em `Document.metadata.review` ate existir modelagem propria de historico.
- **Alternativas consideradas:** Criar imediatamente `document_status_history`; reutilizar `documents.requirements`; permitir apenas gestores; adiar ate o `OnboardingModule`.
- **Consequencias positivas:** Fecha um contrato backend mapeado, separa revisao de requisitos documentais, evita ampliar schema sem MySQL real e preserva auditoria das mudancas de status.
- **Consequencias negativas:** Historico estruturado de status ainda depende de migration real; a tela legada/React ainda nao consome a rota; vinculo real cliente-gestor-departamento ainda precisa ser validado com dados persistidos.
- **Arquivos impactados:** `portal-sama-api/src/modules/documents/**`, `portal-sama-api/src/modules/rbac/default-rbac.ts`, `portal-sama-api/test/app.e2e-spec.ts`.
- **Referencias:** `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`, `docs/INVENTARIO_ENDPOINTS.md`, `docs/SEGURANCA.md`, `docs/paginas/client-painel.md`.

## ADR-0007 — Requisitos documentais customizados no Prisma

- **Data:** 2026-05-11
- **Status:** Aceita parcialmente
- **Contexto:** O legado `api/client_documents.php?action=add_custom` salva requisitos documentais customizados por cliente em KV store. A migração para NestJS precisa manter a compatibilidade funcional sem depender de KV genérico.
- **Decisão:** Criar o modelo Prisma `DocumentRequirement` para requisitos customizados por cliente, adicionar a permissão `documents.requirements` e expor `POST /api-v2/documents/custom-requirements`, `GET /api-v2/documents/templates` e `GET /api-v2/documents/required-pending-summary`. A criação de requisito exige JWT, `documents.requirements`, CSRF, papel gerencial e auditoria.
- **Alternativas consideradas:** Continuar usando KV store no NestJS; guardar requisitos customizados em `Document.metadata`; adiar `add_custom` até o frontend React.
- **Consequências positivas:** O checklist de documentos fica tipado, auditável e preparado para migrations; reduz dependência do endpoint PHP action-based.
- **Consequências negativas:** Depende de migration/seed real no MySQL; ainda falta validar vínculo real cliente/gestor e criar fluxo de aprovação/rejeição de documentos.
- **Arquivos impactados:** `portal-sama-api/prisma/schema.prisma`, `portal-sama-api/src/modules/documents/**`, `portal-sama-api/src/modules/rbac/default-rbac.ts`, `portal-sama-api/test/app.e2e-spec.ts`.
- **Referências:** `docs/MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`, `docs/INVENTARIO_ENDPOINTS.md`, `docs/SEGURANCA.md`, `docs/paginas/client-painel.md`.

## ADR-0006 — DocumentsModule inicial e storage privado

- **Data:** 2026-05-11
- **Status:** Aceita parcialmente
- **Contexto:** A próxima etapa após RBAC administrativo era migrar documentos sensíveis de `api/client_documents.php`/`api/client_documents_lib.php` para a API v2. O legado já tinha validações importantes de upload, mas ainda estava acoplado a actions PHP e ao painel HTML/JS.
- **Decisão:** Criar `DocumentsModule` em `portal-sama-api/src/modules/documents/` com rotas iniciais de listagem, detalhe, upload, download e arquivamento lógico. As rotas usam JWT, permissões granulares (`documents.read`, `documents.upload`, `documents.download`, `documents.delete`), CSRF em mutações, storage privado configurado por `STORAGE_PRIVATE_PATH`, validação de extensão/MIME/assinatura/tamanho/hash SHA-256 e auditoria via `AuditService`.
- **Alternativas consideradas:** Migrar o painel frontend antes do backend; manter somente o endpoint PHP até ClamAV estar pronto; implementar todos os fluxos de templates/pendências/onboarding público de uma vez.
- **Consequências positivas:** Cria contrato tipado para documentos, reduz risco de expor caminho físico, reaproveita a base de RBAC/CSRF/auditoria e permite testar upload/download antes da migração do painel.
- **Consequências negativas:** O módulo ainda não substitui todo o legado; ClamAV/quarentena real no host, migrations, rate limit distribuído e validação de vínculo cliente/gestor em dados reais continuam pendentes.
- **Arquivos impactados:** `portal-sama-api/src/modules/documents/**`, `portal-sama-api/prisma/schema.prisma`, `portal-sama-api/src/app.module.ts`, `portal-sama-api/test/app.e2e-spec.ts`, `portal-sama-api/package.json`.
- **Referências:** `docs/SEGURANCA.md`, `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/ROADMAP_REFATORACAO.md`, `docs/paginas/client-painel.md`, `docs/paginas/onboarding-documentos-cliente.md`.

## ADR-0005 — RBAC administrativo inicial e seed explícito

- **Data:** 2026-05-08
- **Status:** Aceita parcialmente
- **Contexto:** A documentação define RBAC como etapa crítica antes dos módulos sensíveis e o legado concentra administração de usuários em `api/storage.php?action=admin_users_*` e chaves genéricas como `sama_usuarios_login_v1`. O `PermissionsGuard` e os modelos Prisma já existiam, mas ainda não havia contratos administrativos nem seed de permissões.
- **Decisão:** Criar `UsersModule`, `RolesModule` e `PermissionsModule` com rotas iniciais somente leitura, protegidas por JWT e permissões `users.read`, `roles.read` e `permissions.read`. Criar catálogo central `DEFAULT_PERMISSIONS`/`DEFAULT_ROLES` e script explícito `npm.cmd run prisma:seed` para popular roles/permissões, incluindo `audit.read`, sem criar usuários ou senhas.
- **Alternativas consideradas:** Implementar CRUD completo de usuários imediatamente; continuar usando somente `api/storage.php`; criar seed com usuário master e senha provisória.
- **Consequências positivas:** Estabelece contratos tipados para a futura tela DEV/admin, reduz dependência do endpoint genérico legado, evita hardcode de senha em seed e desbloqueia permissões persistidas para auditoria e próximos módulos.
- **Consequências negativas:** RBAC ainda não está completo; mutações administrativas, CSRF, auditoria de alteração, migração de usuários legados e validação de escopo por recurso continuam pendentes. O seed depende de MySQL real/homologação.
- **Arquivos impactados:** `portal-sama-api/src/modules/users/**`, `portal-sama-api/src/modules/roles/**`, `portal-sama-api/src/modules/permissions/**`, `portal-sama-api/src/modules/rbac/**`, `portal-sama-api/prisma/seed.ts`, `portal-sama-api/test/app.e2e-spec.ts`.
- **Referências:** `docs/SEGURANCA.md`, `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/paginas/dev-dev.md`, `docs/paginas/dev-colaborador.md`, `docs/ROADMAP_REFATORACAO.md`.

## ADR-0004 — AuditModule centralizado na API v2

- **Data:** 2026-05-08
- **Status:** Aceita parcialmente
- **Contexto:** A documentação exige auditoria para ações críticas e a tela `Auditoria/autoria.html` usa actions legadas em `api/storage.php?action=audit_*`. O `AuthModule` já registrava eventos diretamente via Prisma, mas isso duplicaria lógica em cada módulo futuro.
- **Decisão:** Criar `AuditModule` em `portal-sama-api/src/modules/audit/`, com `AuditService` centralizado para gravação, mascaramento de metadados sensíveis e endpoints de consulta `GET /api-v2/audit/logs` e `GET /api-v2/audit/logs/:id`, protegidos por JWT e permissão `audit.read`.
- **Alternativas consideradas:** Manter gravação direta de auditoria dentro de cada service; migrar toda a tela de auditoria antes do backend; adiar auditoria para depois de documentos/certificados.
- **Consequências positivas:** Centraliza trilha de auditoria, reduz repetição, cria contrato protegido para a futura tela React e prepara documentos/certificados para registrar eventos críticos.
- **Consequências negativas:** A consulta real ainda depende de MySQL/homologação e seed de permissões; por enquanto o endpoint não substitui completamente `audit_push`, lixeira, backup/restauração ou retenção do legado.
- **Arquivos impactados:** `portal-sama-api/src/modules/audit/**`, `portal-sama-api/src/modules/auth/auth.service.ts`, `portal-sama-api/src/app.module.ts`, `portal-sama-api/test/app.e2e-spec.ts`.
- **Referências:** `docs/SEGURANCA.md`, `docs/MAPEAMENTO_MIGRACAO_APIS.md`, `docs/paginas/auditoria-autoria.md`, `docs/ROADMAP_REFATORACAO.md`.

## ADR-0003 — CSRF double-submit assinado no AuthModule

- **Data:** 2026-05-08
- **Status:** Aceita parcialmente
- **Contexto:** A autenticação alvo usa refresh token em cookie HttpOnly/Secure/SameSite. `docs/SEGURANCA.md` exige CSRF para mutações críticas quando autenticação usa cookies, especialmente login/logout/refresh.
- **Decisão:** Implementar `GET /api-v2/auth/csrf` e exigir token CSRF double-submit assinado por HMAC em `POST /api-v2/auth/login`, `/refresh` e `/logout`, validando cookie, cabeçalho `x-csrf-token` e `Origin`/`Referer` quando presentes.
- **Alternativas consideradas:** Confiar apenas em SameSite; exigir bearer token em todas as mutações; adiar CSRF para o frontend React.
- **Consequências positivas:** Reduz risco de CSRF antes da integração com frontend, mantém refresh token fora do JavaScript e cria contrato explícito para o client.
- **Consequências negativas:** O frontend precisará buscar e reenviar token CSRF; o fluxo completo ainda depende de teste em navegador e MySQL real.
- **Arquivos impactados:** `portal-sama-api/src/modules/auth/csrf.service.ts`, `portal-sama-api/src/modules/auth/auth.controller.ts`, `portal-sama-api/src/config/env.schema.ts`, `.env.example`, `portal-sama-api/.env.example`.
- **Referências:** `docs/SEGURANCA.md`, `docs/INVENTARIO_SEGURANCA.md`, `docs/paginas/index.md`.

## ADR-0002 — Autenticação inicial em `/api-v2/auth`

- **Data:** 2026-05-08
- **Status:** Aceita parcialmente
- **Contexto:** O legado autentica via sessão PHP em `api/storage.php?action=auth_login`, `auth_session_status` e `auth_logout`. A documentação alvo pede access token curto, refresh token em cookie HttpOnly/Secure/SameSite, rotação/revogação, auditoria e evitar tokens persistidos em `localStorage`.
- **Decisão:** Criar `AuthModule` em `portal-sama-api` com access JWT bearer, refresh token opaco em cookie HttpOnly, HMAC-SHA-256 do refresh token em `RefreshToken.tokenHash`, rotação em `/api-v2/auth/refresh`, revogação em `/api-v2/auth/logout` e auditoria de eventos de autenticação.
- **Alternativas consideradas:** Reusar sessão PHP no NestJS; guardar refresh token em resposta JSON; aguardar React antes de implementar autenticação.
- **Consequências positivas:** A API v2 ganha base testável de autenticação sem alterar o frontend legado, reduz exposição de refresh token e prepara RBAC por claims.
- **Consequências negativas:** A estratégia CSRF final ainda precisa ser implementada antes de produção; claims do access token podem ficar defasadas até expiração; integração real depende de MySQL/migrations e frontend React.
- **Arquivos impactados:** `portal-sama-api/src/modules/auth/**`, `portal-sama-api/prisma/schema.prisma`, `portal-sama-api/src/app.module.ts`, `portal-sama-api/src/common/decorators/current-user.decorator.ts`.
- **Referências:** `docs/SEGURANCA.md`, `docs/GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`, `docs/MAPEAMENTO_MIGRACAO_APIS.md`.

## ADR-0001 — Fundação NestJS em `portal-sama-api`

- **Data:** 2026-05-08
- **Status:** Aceita
- **Contexto:** A documentação permite iniciar a migração pela estrutura simples `portal-sama-api`/`portal-sama-web` ou pela estrutura monorepo `apps/api`/`apps/web`. O estado atual ainda é PHP/HTML legado, sem workspace Node formal no raiz.
- **Decisão:** Criar a fundação da API v2 em `portal-sama-api/`, em paralelo ao PHP legado, expondo rotas sob `/api-v2`.
- **Alternativas consideradas:** Criar `apps/api` imediatamente; aguardar scaffold completo de frontend; manter somente PHP até autenticação.
- **Consequências positivas:** Facilita deploy separado no EasyPanel, reduz risco de conflito com o legado e permite validar NestJS/Prisma antes de migrar módulos de negócio.
- **Consequências negativas:** O repositório terá temporariamente uma estrutura mista até a criação do frontend React e eventual padronização de workspace.
- **Arquivos impactados:** `portal-sama-api/**`, `.gitignore`, `docs/STATUS_IMPLEMENTACAO.md`, `docs/EASYPANEL_DEPLOY.md`.
- **Referências:** `docs/GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`, `docs/ROADMAP_REFATORACAO.md`, `docs/EASYPANEL_DEPLOY.md`.

## 2026-05-08 - Debug PHP legado

Até a migração para NestJS, endpoints PHP públicos não devem habilitar `display_errors` por padrão. Quando debug local for necessário, a ativação deve ser explícita por variável de ambiente (`SAMA_DEBUG`, `APP_DEBUG` ou `SAMA_DISPLAY_ERRORS`) e nunca por alteração permanente de código.

Motivo: reduzir risco de exposição de stack trace, caminhos internos e dados sensíveis em produção enquanto a migração gradual ainda não foi concluída.

## 2026-05-08 - Template de ambiente

`.env.example` deve ser versionado com placeholders, sem segredos reais. `.env` e `.env.*` continuam ignorados no Git.

Motivo: permitir setup reproduzível e deploy documentado sem expor credenciais.

As decisões técnicas já documentadas antes desta auditoria estão em:

- `docs/DECISOES_TECNICAS_POS_DOCUMENTACAO.md`
- `docs/ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md`
- `docs/BANCO_DADOS_MYSQL_PRISMA.md`
- `docs/EASYPANEL_DEPLOY.md`
