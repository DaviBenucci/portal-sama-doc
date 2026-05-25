# Pendências Técnicas - Portal Sama

## Atualizacao 2026-05-25 11:46 -03:00

- Atualizado o estado real de infraestrutura: repositorios Bitbucket ja existem, EasyPanel ja possui `portal-sama-api`, `portal-sama-database` e `portal-sama-web` online, e o dominio `https://portal.samacontabil.com.br/` esta ativo.
- `npm.cmd run smoke:public` passou sem `--soft`: frontend 200, `/api-v2/health` 200 com `database=up` e `storage=up`, CORS aceitou `https://portal.samacontabil.com.br` e `/api-v2/auth/csrf` emitiu token/cookie.
- `curl.exe` confirmou headers publicos de seguranca/CORS em `/api-v2/auth/csrf`, incluindo HSTS, CSP, `Access-Control-Allow-Credentials: true` e cookie CSRF `Secure; SameSite=Lax`.
- O banco real do EasyPanel esta operacional como `portal-sama-database`; log informado mostra MySQL Community Server 9.7.0 pronto em `:3306`. Usuarios iniciais da aplicacao ja foram criados pelo operador.
- Deixam de ser pendencias principais: criar repositorios Bitbucket, criar servicos EasyPanel, publicar dominio HTTPS, confirmar proxy `/api-v2`, validar health publico e rodar smoke publico sem `--soft`.
- Permanecem pendentes: confirmar migrations/seed/bootstrap ou procedimento equivalente, validar matriz de permissoes por perfil, backfills, storage persistente/backup, ClamAV strict, Playwright real, QA visual final, backups/rollback e desligamento seguro do legado.

## Atualizacao 2026-05-25 11:32 -03:00

- Reduzida a pendencia de health operacional da API: `GET /api-v2/health` agora checa banco e storage em vez de informar apenas configuracao.
- `HealthService` executa `SELECT 1` via Prisma quando `PRISMA_CONNECT_ON_BOOT=true`; com `false`, mantem `database=not_checked` para testes/diagnostico sem banco.
- O storage configurado em `STORAGE_PRIVATE_PATH` agora e criado/verificado com permissao de leitura/escrita; falhas retornam `storage=down` e HTTP 503.
- Passaram `npm.cmd test -- health.service.spec.ts --runInBand`, `npm.cmd run test:e2e -- --runInBand` com 136 testes, `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run prisma:validate` e `git diff --check` em `portal-sama-api`.
- Permanece pendente validar esse health contra o MySQL/volume reais do EasyPanel e rodar o smoke publico sem `--soft`.

## Atualizacao 2026-05-25 11:20 -03:00

- Resolvida a lacuna entre `EASYPANEL_DEPLOY.md` e o repo separado do Web: `portal-sama-web` agora possui `npm.cmd run smoke:public`.
- Criado `portal-sama-web/scripts/portal-public-smoke.mjs` para validar frontend publico, `/api-v2/health`, preflight CORS de `/api-v2/auth/me` e `/api-v2/auth/csrf` com token/cookie.
- `portal-sama-web/README.md` agora documenta uso, variaveis `PORTAL_PUBLIC_URL`, `PORTAL_API_BASE_URL`, `PORTAL_CORS_ORIGIN`, `PORTAL_SMOKE_TIMEOUT_MS` e modo `--soft`.
- Passaram `node --check`, smoke local com servidor HTTP fake, smoke `--soft` contra porta fechada, `npm.cmd run lint`, `npm.cmd run build` e `git diff --check` no repo `portal-sama-web`.
- Permanece pendente publicar API/Web no EasyPanel e rodar `npm.cmd run smoke:public` sem `--soft` contra o dominio real.

## Atualizacao 2026-05-25 11:09 -03:00

- Resolvido o ponto operacional de README minimo em `portal-sama-api` e `portal-sama-web` apontando para `portal-sama-docs`.
- `portal-sama-api/README.md` foi criado com documentos obrigatorios, comandos principais, notas de Prisma/MySQL e cuidados de versionamento.
- `portal-sama-web/README.md` foi atualizado com documentos obrigatorios, paginas relacionadas, comando E2E e nota do proxy `/api-v2` para EasyPanel.
- Passaram `git diff --check` nos repos de API, Web e Docs.
- Permanece pendente criar remotes Bitbucket, fazer push, configurar EasyPanel e validar banco/HTTPS/usuarios reais.

## Atualizacao 2026-05-25 10:51 -03:00

- A pendencia de E2E local do Web separado foi reduzida: `portal-sama-web/tests/e2e/smoke.spec.ts` agora cobre login mobile e intro `welcome` do primeiro login.
- O mock de autenticacao passou a simular `csrf -> login` com `welcomeAnimationSeen=false`, permitindo validar o `IntroGate` sem API real.
- O smoke mobile do login valida ausencia de overflow, logo dentro da area visual, texto manuscrito sem corte e painel no viewport.
- O smoke da welcome valida logo centralizada, texto dentro do viewport e mascaras de fade nas linhas laterais.
- Passaram `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e` com 9 testes Chromium e `git diff --check` no repo `portal-sama-web`.
- Permanece pendente Playwright contra homologacao real, QA visual no EasyPanel, usuarios/permissoes reais e validacao com API/MySQL/HTTPS reais.

## Atualizacao 2026-05-25 10:33 -03:00

- Corrigido desalinhamento visual da intro/welcome no frontend causado por dependencia da propriedade CSS individual `translate`.
- A intro agora ancora logo/texto via atributos `data-intro-x-percent` e `data-intro-y-percent`, aplicados pela timeline GSAP.
- As linhas laterais da welcome receberam mascaras CSS para esmaecerem antes do centro.
- A lateral do login deixou de depender de `translate`; offsets foram movidos para keyframes com `translateY(...)`.
- Passaram `npm.cmd run lint`, `npm.cmd run build`, `git diff --check`, smoke Playwright local em `/dev/intro-preview` e `/login`, e checagem de ausencia de `translate:` em `src`/`public`.
- Permanece pendente redeployar o frontend no EasyPanel e validar visualmente em navegador real com cache limpo.

## Atualizacao 2026-05-25 09:48 -03:00

- Diagnosticado o novo erro de deploy Prisma `P1013`: `invalid port number in database URL`.
- Corrigida a validacao de ambiente da API para rejeitar `DATABASE_URL` malformada antes da inicializacao do Prisma Client.
- Adicionado teste unitario para URL MySQL valida no formato EasyPanel, porta duplicada e senha com caracteres especiais sem URL encode.
- Documentado no EasyPanel que esse erro nao exige recriar o banco; exige corrigir a string `mysql://usuario:SENHA_URL_ENCODED@host:3306/banco`.
- Passaram `npm.cmd test -- env.schema.spec.ts --runInBand`, `npm.cmd run lint`, `npm.cmd run build` e `npm.cmd run prisma:validate`.
- Permanece pendente corrigir `DATABASE_URL` real no EasyPanel, redeployar API e executar seed/bootstrap/validacoes reais.

## Atualizacao 2026-05-25 09:25 -03:00

- Corrigido `npm run prisma:seed` no runtime Docker: o script agora usa `dist/prisma/seed.js` quando o build compilado existe.
- Corrigido tambem `npm run prisma:bootstrap-admin` para usar `dist/prisma/bootstrap-admin.js` no container.
- Passaram `node --check scripts/run-prisma-runtime-script.js`, `npm.cmd run build`, `npm.cmd run lint` e `npm.cmd run prisma:validate`.
- `npm.cmd run prisma:seed` com banco falso em `127.0.0.1:9` falhou apenas por conexao recusada, confirmando que o erro de import TypeScript foi removido.
- Permanece pendente redeployar a API no EasyPanel e rodar seed/bootstrap contra o MySQL real.

## Atualizacao 2026-05-22 16:38 -03:00

- Corrigido o fluxo que derrubava o container da API com Prisma `P3005`: migrations nao rodam mais no start por padrao.
- Corrigido o entrypoint de producao para `dist/src/main.js`, caminho real do build atual.
- Adicionada migration vazia `20260501000000_baseline_existing_database` para baseline de banco EasyPanel existente sem `_prisma_migrations`.
- Criado `npm run prisma:migrate:baseline-existing`, protegido por `SAMA_PRISMA_BASELINE_EXISTING_DATABASE=true`, para marcar baseline e aplicar migrations reais apos backup.
- Passaram `npm.cmd run prisma:generate`, `npm.cmd run prisma:validate`, `npm.cmd run build`, `npm.cmd run lint`, `node --check` do script de baseline, Docker build e `git diff --check`; o script de baseline sem confirmacao e o Docker run sem `DATABASE_URL` falharam como esperado.
- Permanece pendente executar backup/snapshot do `banco-sama`, rodar baseline controlado uma unica vez no EasyPanel, seed, bootstrap admin e validacoes health/csrf/login.

## Atualizacao 2026-05-22 16:08 -03:00

- Corrigido `portal-sama-api/Dockerfile` para o build nao depender da `DATABASE_URL` real durante `prisma generate`.
- O runtime da API continua exigindo `DATABASE_URL` real antes de migrations opcionais e start da API.
- Documentado o diagnostico do erro Prisma `P1012` no EasyPanel: configurar a variavel no servico `portal-sama-api`, apontando para `portal-sama_database:3306/banco-sama`.
- Passaram `npm.cmd run prisma:generate`, `npm.cmd run build`, `npm.cmd run prisma:validate` e `docker build --pull=false -t portal-sama-api:prisma-env-fix .` em `portal-sama-api`; `docker run --rm portal-sama-api:prisma-env-fix` sem env falhou como esperado com mensagem explicita; `git diff --check` passou nos docs e apontou apenas avisos LF/CRLF no repo da API.
- Permanece pendente fazer rebuild/redeploy real no EasyPanel, rodar migrations/seed/bootstrap admin e validar health/csrf/login.

## Atualizacao 2026-05-22 15:22 -03:00

- Passaram lint/build nos repos separados `portal-sama-api` e `portal-sama-web`.
- Passou `npm.cmd run prisma:validate` em `portal-sama-api`; o comando carregou `.env` automaticamente, sem exibir valores sensiveis.
- Estimativa operacional atualizada: 76% para iniciar homologacao e 64% para producao sem legado.
- Permanece pendente criar os tres repositorios no Bitbucket, adicionar remotes, fazer push e configurar o EasyPanel.
- Permanece pendente E2E/Playwright real, migrations/seed/bootstrap em MySQL real, storage/ClamAV, backfills, usuarios/permissoes reais e QA visual.

## Atualizacao 2026-05-22 15:16 -03:00

- Workspaces locais permanecem separados e preparados em `main`.
- Criado `.gitattributes` em `portal-sama-docs`, `portal-sama-api` e `portal-sama-web` para evitar churn de LF/CRLF antes de publicar no Bitbucket.
- Estimativa operacional atualizada: 74% para iniciar homologacao e 63% para producao sem legado.
- Permanece pendente criar os tres repositorios no Bitbucket, adicionar remotes, fazer push e validar E2E/Playwright a partir dos repos separados.

## Atualizacao 2026-05-22 15:10 -03:00

- Workspaces locais separados e Git inicializado em `portal-sama-docs`, `portal-sama-api` e `portal-sama-web`.
- Criados/ajustados `.gitignore` para evitar versionar `.env`, `node_modules`, `dist`, relatorios Playwright e artefatos locais.
- Permanece pendente criar os tres repositorios no Bitbucket e adicionar remotes.
- Permanece pendente enviar os commits iniciais para Bitbucket e configurar o EasyPanel para consumir apenas `portal-sama-api` e `portal-sama-web`.

## Atualizacao 2026-05-22 14:57 -03:00

- A separacao Bitbucket foi formalizada como tres repositorios: `portal-sama-docs`, `portal-sama-api` e `portal-sama-web`.
- Permanece pendente criar fisicamente os tres repositorios no Bitbucket e subir os conteudos corretos.
- Permanece pendente definir se `portal-sama-api` e `portal-sama-web` terao README/instrucao minima apontando para `portal-sama-docs`.
- A regra operacional ficou registrada: toda IA deve ler `portal-sama-docs` antes de alterar API ou Web.

## Atualizacao 2026-05-22 14:18 -03:00

- Criado/atualizado o painel obrigatorio `docs/AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD` para acompanhar o que falta para homologacao e producao.
- O documento registra a necessidade de separar `portal-sama-api` e `portal-sama-web` em repositorios Bitbucket, configurar ambos no EasyPanel e validar `banco-sama` com migrations, seed e bootstrap.
- A pendencia de acompanhamento de percentual agora tem fonte oficial: estimativa operacional atual de 72% para iniciar homologacao e 62% para producao sem legado.
- Permanecem pendentes as acoes reais: criar repos Bitbucket, configurar servicos EasyPanel, rodar migrations/seed/bootstrap no banco real, validar storage/ClamAV, executar backfills, validar usuarios/permissoes reais, Playwright real e QA visual.

## Atualizacao 2026-05-22 14:05 -03:00

- A pendencia de primeiro acesso em banco limpo foi reduzida: existe agora `npm run prisma:bootstrap-admin` em `portal-sama-api`.
- O seed RBAC foi refatorado para poder ser reaproveitado pelo bootstrap sem criar senha no seed padrao.
- O procedimento de EasyPanel foi atualizado para dois repositorios e banco existente `banco-sama`, usando `DATABASE_URL=mysql://portal_user:SENHA_FORTE@portal-sama_database:3306/banco-sama`.
- Passaram lint, build, Prisma validate e `git diff --check` localmente; o bootstrap ainda nao foi executado contra banco real.
- Permanecem pendentes criar/validar usuario MySQL nao-root no EasyPanel, rodar `prisma migrate deploy` contra MySQL 9.6.0, executar seed/bootstrap no container real, validar login/CSRF/cookies HTTPS, storage privado, ClamAV, backfills, usuarios/permissoes reais e Playwright com backend real.
- A independencia total do legado ainda nao pode ser declarada porque Integra-AI continua read-only na API v2 e parte das mutacoes/backfills ainda precisa de homologacao.

## Atualizacao 2026-05-22 10:02 -03:00

- A pendencia de Playwright para a central interna `/documentos` foi reduzida: `portal-sama-web/tests/e2e/smoke.spec.ts` agora cobre a rota autenticada com APIs mockadas.
- O novo smoke valida listagem de documento privado, cliente vinculado, extensoes permitidas, links publicos, resumo de pendencias obrigatorias e abertura do formulario de revisao.
- Passaram lint, build, E2E com 7 testes Chromium e `git diff --check`.
- Permanecem pendentes validacao com API v2/MySQL/storage/ClamAV reais, backfill, usuarios/permissoes reais, homologacao HTTPS, QA visual manual e Playwright de upload/download reais.
- O aviso de `HydrateFallback` do React Router continua registrado como ruido conhecido da execucao Playwright/Vite.

## Atualizacao 2026-05-22 09:39 -03:00

- A pendencia de cobertura mobile automatizada para Home/sidebar foi reduzida: `portal-sama-web/tests/e2e/smoke.spec.ts` agora cobre `/home` em viewport `390x844`.
- O novo smoke confirma sidebar expandida no mobile, labels visiveis, grid de navegacao em uma coluna e ausencia de overflow horizontal.
- Passaram lint, build e E2E com 6 testes Chromium.
- Permanece pendente QA visual manual em navegador real, homologacao HTTPS e validacao com usuario/permissoes reais.
- O aviso de `HydrateFallback` do React Router continua registrado como ruido conhecido da execucao Playwright/Vite.

## Atualizacao 2026-05-22 09:28 -03:00

- A pendencia de Playwright especifico para Home/sidebar foi reduzida: a suite E2E do `portal-sama-web` agora cobre `/home`, atalhos por permissao e sidebar compacta/expandida.
- Foram adicionados testes para fechamento da sidebar apos clique com mouse e para abertura por foco de teclado.
- Passaram lint, build, E2E com 5 testes e `git diff --check`.
- Permanece pendente QA visual manual em navegador real e ampliacao futura para mobile/homologacao com usuario real.

## Atualizacao 2026-05-22 09:07 -03:00

- Corrigido o bug da sidebar permanecer aberta apos clicar em um item do menu.
- A abertura por foco agora depende de `:focus-visible`, entao clique com mouse nao prende mais o menu aberto.
- A navegacao por teclado continua abrindo o menu corretamente.
- Passaram lint, build, smoke de clique/mouseleave, smoke de teclado, E2E e `git diff --check`.
- Permanece pendente QA visual manual em navegador real.

## Atualizacao 2026-05-22 08:54 -03:00

- A pendencia da sidebar recolhida foi corrigida: a logo Sama nao desaparece mais quando o menu esta fechado.
- O menu autenticado agora fica fechado por padrao no desktop e abre automaticamente no hover/foco.
- O botao `Recolher menu` foi removido por nao ser mais necessario no novo fluxo.
- Passaram lint, build, smoke Playwright `sidebar-hover-smoke`, E2E e `git diff --check`.
- Permanece pendente QA visual manual para aprovar a transicao e o comportamento em telas reais.

## Atualizacao 2026-05-22 08:42 -03:00

- A pendencia visual da pagina de login foi reduzida: a lateral esquerda agora usa a logo Sama animada em camadas, alinhada a linguagem da intro.
- O texto `Portal Interno` foi removido e substituido por `Bem vindo ao Portal Sama` com efeito manuscrito de escrita.
- O layout foi validado em desktop e mobile por smoke Playwright, sem corte do texto e sem overflow.
- Passaram `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e` e screenshot visual de `/login`.
- Permanece pendente QA visual manual em navegador real para aprovar escala da logo, intensidade dos glows e timing percebido da escrita.

## Atualizacao 2026-05-21 16:48 -03:00

- O texto manuscrito da welcome foi ajustado para nao cortar o final de `Sama`; smoke de navegador confirmou `clientWidth` igual a `scrollWidth`.
- A frase exibida agora e `Seja bem vindo ao Portal Sama`, sem hifen.
- As linhas laterais agora usam mascara/gradientes para esmaecer e sumir quanto mais proximas ficam do centro.
- Os efeitos ao redor da logo foram alinhados ao centro visual real da marca, corrigindo a orbita que parecia deslocada.
- Passaram build, E2E, lint reexecutado isoladamente, validacao JSON/SVG, smoke focado e `git diff --check`.

## Atualizacao 2026-05-21 16:32 -03:00

- A cena de boas-vindas agora exige `Enter` para continuar; nao sai mais por timer automatico.
- Apos 7s na tela, aparece uma notificacao discreta orientando `Pressione Enter para continuar`.
- Os SVGs laterais foram refeitos com linhas mais delicadas, paralelas e sutis; os WebPs de onda ficaram com opacidade reduzida.
- Passaram lint, build, Playwright E2E, validacao JSON, checagem estatica dos SVGs novos e smoke `welcome-enter-required-smoke`.
- Permanece pendente QA visual manual para comparar a delicadeza das linhas com a referencia final em desktop/mobile.

## Atualizacao 2026-05-21 16:12 -03:00

- A pendencia de timing da intro foi ajustada: entrada normal agora dura `2.7s`, boas-vindas `4.5s`, e o fade final foi ampliado.
- A cena welcome ganhou texto manuscrito com revelacao progressiva e duas camadas SVG laterais com efeitos de glow/desenho.
- As ondas/linhas laterais foram limitadas a aproximadamente 30-35% da tela para evitar que se juntem no centro.
- Passaram validacao JSON, checagem estatica dos SVGs novos, lint, build, Playwright E2E, smoke login-only e `git diff --check`.
- Permanecem pendentes QA visual manual desktop/mobile e homologacao com usuario real para aprovar timing, fonte manuscrita e densidade dos efeitos SVG.

## Atualizacao 2026-05-21 15:46 -03:00

- A pendencia visual da intro foi reduzida: a animacao pos-login foi retimada para cerca de 1.7s na entrada normal, ganhou fade de saida e deixou de conflitar `transform` de CSS com animacoes GSAP.
- A intro autenticada agora roda apenas quando a sessao nasce de `login`; refresh/snapshot/reload entram direto na aplicacao com bootstrap simples.
- A sidebar passou a usar a logo Sama sem fundo em container centralizado, e os assets decorativos do dashboard foram simplificados para linhas finas em vez de blocos/traços grossos.
- Passaram `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e`, smoke Playwright inline de intro login-only e `git diff --check`; o Playwright manteve apenas o aviso conhecido de `HydrateFallback`.
- Permanecem pendentes QA visual manual desktop/mobile, homologacao HTTPS e, se o visual for aprovado, criar smoke visual especifico para intro login-only, sidebar expandida/recolhida e KPIs.

## Atualizacao 2026-05-21 14:24 -03:00

- Corrigida a pendencia operacional do erro apos login em desenvolvimento: a intro nao importa mais GSAP no topo do chunk do layout autenticado.
- Se o Vite/GSAP falhar no runtime, a timeline agora aciona fallback estatico e a aplicacao segue navegavel.
- O cache `portal-sama-web/node_modules/.vite` foi limpo e o dev server foi reiniciado com `--force` em `http://127.0.0.1:5173/`.
- Passaram lint, build, Playwright E2E, smoke browser de `/home`, checagem HTTP de manifesto/mapa/GSAP e `git diff --check`.
- Permanecem pendentes QA visual real desktop/mobile, homologacao HTTPS e Playwright visual especifico para as cenas/modos da intro.

## Atualizacao 2026-05-21 14:05 -03:00

- A pendencia da intro visual foi reduzida: `portal-sama-web` agora usa GSAP/`@gsap/react` como engine oficial em `src/features/intro`, com `manifest.json`, `animation-map.json` e assets finais locais em `/brand/sama/intro/assets`.
- Foram removidos os componentes visuais antigos de Framer Motion e os assets SVG antigos da intro ativa; `framer-motion` tambem foi removido das dependencias do frontend.
- A versao da intro passou para `sama-gsap-layers-v2`, incluindo `INTRO_ANIMATION_VERSION`, manifesto e mapa de animacoes.
- Permanecem pendentes QA visual em navegador real desktop/mobile, homologacao HTTPS, Playwright visual para loading/welcome/static/animated/reduced motion/falha obrigatoria e eventuais refinamentos de timing/posicionamento apos avaliacao visual.
- A sidebar teve contraste reforcado para textos brancos; validar com usuario real/permissoes e responsividade em homologacao.

## Atualizacao 2026-05-21 10:57 -03:00

- A pendencia de padronizacao visual do `portal-sama-web` foi reduzida: existem agora sidebar premium recolhivel, topbar ajustada, tokens/classes compartilhadas e os componentes `PageShell`, `AppPanel`, `IconBadge`, `KpiCard` e `ShortcutCard`.
- A Home `/home` ja usa o novo padrao para cabecalho, KPIs e atalhos, mantendo rotas e permissoes reais.
- O aviso de build sobre `/brand/sama-logo-symbol.svg` foi eliminado ao apontar a identidade visual para o PNG de referencia existente em `public/brand/sama/source/sama-symbol-reference.png`.
- Permanecem pendentes propagar o padrao para Clientes, Documentos, Certificados, Solicitacoes, TI, Legalizacao, Onboarding, Gestor e demais telas; criar Playwright/QA visual para `/home`, sidebar expandida/recolhida e mobile; validar em homologacao; e decidir/restaurar os assets de intro ja deletados no workspace antes desta rodada.
- A intro/assets cinematicos nao foram reconstruidos nesta rodada; foram adicionados apenas assets decorativos novos de dashboard.

## Atualizacao 2026-05-21 10:33 -03:00

- A pendencia da pagina `Contabil/integra-ai.html` foi reduzida: existe agora `AccountingModule` inicial e a rota React `/contabil/integra-ai`.
- O contrato novo cobre leitura do workspace e detalhe de jobs com JWT, RBAC `accounting.integra_ai.read`, escopo contabil, SQL parametrizado, sanitizacao de caminhos privados e auditoria em detalhe de job.
- Permanecem pendentes migrar `parse_bank_statement`, `create_job`, `save_step2`, `save_step3`, `save_step4_rule`, `generate_txt` e `download_txt` com upload seguro, CSRF, auditoria e isolamento de parser/exportacao.
- Tambem permanecem pendentes validacao com MySQL/homologacao, seed RBAC real, Playwright para `/contabil/integra-ai` e desligamento de `api/integra_ai.php` somente apos homologacao.
- A intro/assets nao foram continuados nesta rodada.

## Atualizacao 2026-05-21 09:34 -03:00

- A pendencia da pagina `Depto/modelo.html` foi reduzida: existe agora `DepartmentsModule` inicial e a rota React `/departamentos/modelo`.
- O contrato novo cobre leitura do workspace Fiscal e mutacoes de status por celula com JWT, RBAC `departments.workspace.read/write`, CSRF, escopo por departamento e auditoria.
- Permanecem pendentes validar com MySQL/homologacao, rodar seed RBAC real, conferir backfill de responsaveis fiscais em `Client.metadata`, criar Playwright para a grade e desligar `api/fiscal_workspace.php` somente apos homologacao.
- A intro/assets nao foram continuados nesta rodada.

## Atualizacao 2026-05-21 09:13 -03:00

- A pendencia de topologia reproduzivel foi reduzida com `docker-compose.easypanel.example.yml`, usando `portal-sama-web`, `portal-sama-api` e `portal-sama-mysql`.
- `compose.easypanel.env.example` permite validar o compose sem ler o `.env` local do projeto, evitando mistura de segredos reais com comandos de diagnostico.
- `portal-sama-web/.dockerignore` foi adicionado para reduzir risco de build Docker carregar artefatos locais.
- Permanecem pendentes validar `nginx -t` com Docker daemon ativo ou no container do EasyPanel, subir os servicos reais, rodar migrations/seed, publicar HTTPS e executar `npm.cmd run smoke:public` sem `--soft`.
- A intro/assets nao foram continuados nesta rodada.

## Atualizacao 2026-05-21 08:56 -03:00

- A pendencia de proxy `/api-v2` no dominio unico foi reduzida: `portal-sama-web/nginx.conf` agora encaminha `/api-v2/` para `http://portal-sama-api:3000/api-v2/`.
- O Nginx do frontend tambem foi ajustado para uploads de ate 30 MB e SSE sem buffering em `/api-v2/notifications/stream`.
- Permanecem pendentes redeploy do web, resolucao interna do servico `portal-sama-api`, publicacao da API v2, smoke publico sem `--soft`, login/refresh/cookies/CSRF, upload/download e auditoria em HTTPS real.
- A intro/assets nao foram continuados nesta rodada.

## Atualizacao 2026-05-21 08:41 -03:00

- A pendencia de validacao publica do subdominio foi reduzida: existe agora `scripts/portal-public-smoke.mjs` e `npm run smoke:public` para checar raiz, `/api-v2/health`, CORS e CSRF em HTTPS real.
- O smoke publico em `--soft` confirmou que `https://portal.samacontabil.com.br` responde HTTP 200, mas a API v2 ainda nao esta publicada/proxyada em `/api-v2` (`/health` e `/auth/csrf` retornaram 404).
- Permanecem pendentes publicar/proxyar `portal-sama-api`, corrigir CORS/cookies no EasyPanel, reexecutar `npm.cmd run smoke:public` sem `--soft`, validar login/refresh/upload/download/auditoria e homologar com API v2/MySQL reais.
- A intro/assets nao foram continuados nesta rodada.

## Atualização 2026-05-20 17:36 -03:00

- A pendência de base Playwright do frontend foi reduzida: `portal-sama-web` agora possui `@playwright/test`, scripts `test:e2e` e suite smoke inicial.
- A cobertura inicial valida `/login`, `/onboarding/publico/documentos/:token` com checklist mockado e `/auditoria` com autenticação/listagem/exportação CSV mockadas.
- Permanecem pendentes Playwright ampliado para fluxos críticos autenticados, permissões, mobile, formulários principais, integrações com API v2/MySQL reais e homologação HTTPS.
- A intro/assets não foram continuados nesta rodada.

## Atualização 2026-05-20 17:24 -03:00

- A pendência de exportação operacional da auditoria foi reduzida: `GET /api-v2/audit/logs/export.csv` exporta CSV com filtros, limite de 1000 linhas e registro auditável da própria exportação.
- A rota React `/auditoria` ganhou botão `Exportar CSV`, usando os filtros atuais.
- Permanecem pendentes validação com MySQL real e volume operacional, Playwright, política formal de retenção e modelagem separada de backup/restauração.
- A intro/assets não foram continuados nesta rodada.

## Atualização 2026-05-20 17:11 -03:00

- A pendência de checklist público específico para documentos de onboarding foi tratada com `GET /api-v2/documents/public-checklist?token=...`.
- A rota React `/onboarding/publico/documentos/:token` agora consome o checklist real, exibe status mínimo dos itens e usa as extensões permitidas retornadas pela API.
- Permanecem pendentes homologação com MySQL/storage/ClamAV reais, compatibilidade com tokens legados, Playwright desktop/mobile e validação HTTPS.
- A intro/assets não foram continuados nesta rodada.

## Atualização 2026-05-20 16:51 -03:00

- A pendência de code splitting do frontend foi tratada em `portal-sama-web/src/app/router.tsx` com lazy routes do React Router.
- O alerta de chunk acima de 500 kB do Vite não apareceu na execução final de `npm.cmd run build`; o chunk inicial ficou em 320.90 kB e o `AppLayout` em 138.47 kB.
- Permanecem pendentes validação de navegação real desktop/mobile, Playwright em homologação e o aviso conhecido de `/brand/sama-logo-symbol.svg` resolvido em runtime.
- A intro/assets não foram continuados nesta rodada, conforme orientação de aguardar os ajustes de imagens.

## Atualização 2026-05-20 16:39 -03:00

- A intro modular em SVG saiu de pendente para integrada no `portal-sama-web`: o app agora serve `layer-manifest.json`, `animation-map.json` e 86 assets em `public/brand/sama/intro`.
- `StandardIntro`, `WelcomeIntro` e `/dev/intro-preview` passaram a renderizar cenas por manifesto com `motion.img`, `zIndex`, `animationGroup`, fallback estático e respeito a `prefers-reduced-motion`.
- Permanecem pendentes validação visual com navegador real desktop/mobile, homologação HTTPS, comparação final com referências aprovadas, Playwright visual e decisão de versionamento para os arquivos removidos de `docs/portal-sama-svg-intro-pack`.

## Atualizacao 2026-05-20 16:21 -03:00

- A rota React `/notificacoes` ganhou stream SSE autenticado via `fetch` para `GET /api-v2/notifications/stream`, com status de tempo real e fallback por refetch.
- A tela passou a ativar/desativar assinatura Web Push no navegador usando `/api-v2/notifications/push/subscribe` e `/unsubscribe`, mantendo CSRF via service centralizado.
- `portal-sama-web/public/sama-push-sw.js` foi adicionado para servir o Service Worker no app Vite.
- Permanecem pendentes validacao em HTTPS/homologacao com API v2/MySQL/VAPID reais, teste com portal fechado, browsers reais, unsubscribe, Playwright e broker/event bus se multi-instancia for exigida.

## Atualizacao 2026-05-20 16:02 -03:00

- `CalendarModule` ganhou `DELETE /api-v2/calendar/entries/:id` para substituir inicialmente a action `calendar_delete_entry` de `api/manager_workspace.php`.
- A exclusao exige `calendar.manage`, papel ADMIN/DEV/MANAGER, CSRF, validacao de regra/empresa/departamento e auditoria; tambem limpa notificacoes em `sama_calendar_due_notified` e remove regra orfa quando aplicavel.
- `/manager` passou a permitir remover vencimentos na lista de proximos eventos apenas quando a sessao possui `calendar.manage`.
- Permanecem pendentes validacao com MySQL/homologacao, seed real de `calendar.manage`, dados reais de calendario/carteira, Playwright e desligamento controlado das actions PHP `calendar_*`.

## Atualizacao 2026-05-20 12:30 -03:00

- `ManagersModule` ganhou `GET /api-v2/managers/overview` para substituir inicialmente a action `overview` de `api/manager_workspace.php`.
- `sama_user_presence` passou a ser modelada no Prisma como `UserPresence`, com migration compativel `20260520123000_add_user_presence_model`.
- `/manager` passou a exibir presenca da equipe e contadores online/offline usando `GET /api-v2/managers/overview`, mantendo carteiras/transferencias pelo `TransfersModule` e vencimentos pelo `CalendarModule`.
- Permanecem pendentes validacao com MySQL/homologacao, aplicacao de migrations/seed em ambiente real, usuarios reais por perfil/departamento, Playwright e desligamento do `overview` legado depois da homologacao.

## Atualizacao 2026-05-20 11:45 -03:00

- `ManagersModule` ganhou as mutacoes `POST /api-v2/managers/history`, `PATCH /api-v2/managers/history/:id` e `POST /api-v2/managers/company-life`.
- As actions legadas `history_append`, `history_update` e `company_life_save` agora possuem substituto inicial na API v2 com JWT, `manager_history.write`, escopo por departamento, CSRF e auditoria.
- `/manager/historico` deixou de ser apenas leitura e passou a permitir novo historico, edicao de registros de historico e salvamento de vida da empresa quando a permissao/escopo permite.
- Permanecem pendentes validacao com MySQL/homologacao, seed real de `manager_history.write`, usuarios reais por perfil/departamento, Playwright e migracao do `overview` do manager workspace.

## Atualizacao 2026-05-20 11:20 -03:00

- O `ManagersModule` ganhou a primeira fatia de historico operacional com `GET /api-v2/managers/history` e `GET /api-v2/managers/history/:companyId/timeline`.
- Foram mapeadas no Prisma as tabelas legadas `sama_company_history_entries` e `sama_company_life_entries`, com RBAC `manager_history.read` para ADMIN, DEV e MANAGER.
- A rota React `/manager/historico` passou a listar empresas por departamento, buscar historico e exibir timeline consolidada de historico do gestor e vida da empresa, sem `innerHTML`, sem storage local e sem mutacoes nesta fatia.
- Permanecem pendentes escrita/edicao do historico na API v2, validacao com MySQL/homologacao, usuarios reais por perfil/departamento e Playwright.

## Atualizacao 2026-05-20 10:30 -03:00

- O dashboard React `/manager` passou a consumir `GET /api-v2/calendar/month` para exibir a visao mensal de vencimentos por departamento.
- A tela agora possui filtro de mes de referencia, calendario resumido, proximos vencimentos filtrados por busca geral e atalho para configuracao em `/manager/calendario/config`.
- Permanecem pendentes escrita/edicao do historico do gestor, validacao com MySQL/homologacao, usuarios reais por perfil/departamento e Playwright.

## Atualizacao 2026-05-20 10:00 -03:00

- A rota React `/manager/calendario/config` ganhou primeira experiencia funcional de configuracao de vencimentos, usando o novo `CalendarModule`.
- O backend agora possui `GET/POST /api-v2/calendar/config`, `GET /api-v2/calendar/month` e `GET /api-v2/calendar/day`, modelos Prisma para `sama_calendar_rules`, `sama_calendar_rule_companies` e `sama_calendar_due_notified`, permissoes `calendar.read/manage`, CSRF em salvamento e auditoria.
- Permanecem pendentes validacao com MySQL/homologacao, backfill real de departamentos/carteira, notificacoes de vencimento na API v2, escrita/edicao do historico do gestor e Playwright.

## Atualizacao 2026-05-20 -03:00

- A rota React `/manager` ganhou primeira experiencia funcional de painel do gestor, usando `GET /api-v2/transfers/dashboard`.
- A tela consolida colaboradores, empresas em carteira, transferencias ativas, sessoes recentes, filtro por departamento permitido e busca geral, com atalhos para `/manager/colaboradores` e `/manager/transferencias`.
- Permanecem pendentes validacao com MySQL/homologacao, usuarios reais por perfil/departamento, Playwright e mutacoes seguras do historico do gestor.

## Atualizacao 2026-05-19 17:56 -03:00

- A rota React `/dev/colaboradores` ganhou primeira experiencia funcional de gestao administrativa, usando `GET/PATCH/DELETE /api-v2/collaborators` e `GET /api-v2/roles` quando permitido.
- A pendencia de frontend de `DEV/dev-colaborador.html` saiu de backend parcial para parcial com filtros, estatisticas, edicao de dados funcionais, status, senha opcional, perfis e arquivamento logico por permissao.
- Permanecem pendentes validacao com API v2/MySQL/usuarios reais, backfill de `sama_colaboradores`, vinculos colaborador-cliente/gestor/carteira, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-19 15:53 -03:00

- A rota React `/manager/colaboradores` ganhou primeira experiencia funcional de consulta de carteira por colaborador, usando `GET /api-v2/transfers/dashboard`.
- A pendencia de frontend de `Manager/manager-colaborador.html` saiu de backend parcial para parcial com consulta, filtros, estatisticas, tabela de empresas e sessoes relacionadas; transferir/adicionar carteira continua direcionado para `/manager/transferencias`.
- Permanecem pendentes validacao com MySQL/homologacao, usuarios reais por perfil/departamento, Playwright, backfill/modelagem formal de carteira e mutacoes seguras no `ManagersModule`.

## Atualizacao 2026-05-19 15:33 -03:00

- A pendencia de frontend de transferencias saiu de backend parcial para parcial com primeira experiencia React funcional em `/manager/transferencias`.
- A tela consome `GET /api-v2/transfers/dashboard`, `POST /api-v2/transfers` e `POST /api-v2/transfers/:id/return`, com formulario validado por Zod, CSRF via service centralizado e UI condicionada a `transfers.read/create/return`.
- Permanecem pendentes validacao integrada com API v2/MySQL/homologacao, seed `transfers.*` em ambiente real, backfill/modelagem formal de carteira, Playwright desktop/mobile e teste com usuarios reais por perfil/departamento.

## Atualizacao 2026-05-19 12:16 -03:00

- A pendencia backend de transferencias saiu de mapeada para parcial: `TransfersModule` foi criado com `GET /api-v2/transfers`, `GET /api-v2/transfers/dashboard`, `POST /api-v2/transfers`, `POST /api-v2/transfers/return` e `POST /api-v2/transfers/:id/return`.
- O fluxo possui RBAC `transfers.*`, CSRF em mutacoes, auditoria de criacao/retorno, escopo por departamento, `TransferSession` no Prisma e migration compativel com `sama_transfer_sessions`.
- Permanecem pendentes validacao com MySQL/homologacao, seed atualizado em ambiente real, backfill/modelagem formal de carteira, teste HTTP com JWT/CSRF reais, tela React `/manager/transferencias` e Playwright.

## Atualizacao 2026-05-18 17:35 -03:00

- A base visual/motion do portal saiu de pendente para parcial: `portal-sama-web` agora possui feature `src/features/intro`, `framer-motion`, assets locais de marca, splash padrao, splash de boas-vindas, reduced motion e refinamento inicial do layout autenticado.
- A preferencia `welcomeAnimationSeen` saiu de pendente para parcial com `PATCH /api-v2/me/preferences/intro`, persistindo dados em `User.metadata.portalIntro` com JWT, CSRF e auditoria.
- Permanecem pendentes Playwright/validacao visual real, homologacao com MySQL/usuarios reais, SVG/imagens finais em camadas e expansao das animacoes especificas por pagina.

## Atualizacao 2026-05-18 15:58 -03:00

- A rota React `/departamentos/clientes` ganhou `DepartmentClientsPage.tsx` consumindo `GET /api-v2/clients`.
- A tela cobre consulta departamental inicial com filtros, estatisticas, tabela operacional e responsavel por departamento quando o metadata/backfill trouxer esse vinculo.
- A pendencia de frontend de clientes departamentais saiu de mapeada para parcial com primeira experiencia React funcional. Permanecem pendentes contrato backend para atribuicao/transferencia, backfill de carteira/departamento/responsavel, validacao com usuarios reais, escopo real por departamento, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-18 15:35 -03:00

- A rota React `/ti/acessos` ganhou `TiAccessPage.tsx` consumindo os contratos existentes de `/api-v2/access-requests`.
- A tela cobre operacao de TI para solicitacoes de acesso com fila pendente, historico filtrado, estatisticas, paginacao e aprovar/rejeitar por permissao.
- A pendencia de frontend de TI saiu de placeholder para parcial com primeira experiencia React funcional. A base editavel de acessos/credenciais internas da TI continua pendente por seguranca ate existir criptografia/vault, auditoria de leitura e politica de retencao; tambem faltam validacao com API v2/MySQL/backfill/usuarios reais, escopo por TI/gestor/departamento, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-18 15:14 -03:00

- A rota React `/dev/colaboradores/novo` ganhou `DevNewCollaboratorPage.tsx` consumindo `POST /api-v2/collaborators`.
- A tela cobre cadastro administrativo com usuario, dados funcionais, senha provisoria, status, perfis e feedback com atalho para a visao do colaborador criado.
- A pendencia de frontend do cadastro DEV de colaborador saiu de backend parcial para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/usuarios reais, backfill de colaboradores/usuarios legados, vinculos colaborador-cliente/gestor, carteira/transferencias, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-18 15:07 -03:00

- A rota React `/dev/clientes/novo` ganhou `DevNewClientPage.tsx` consumindo `POST /api-v2/clients`.
- A tela cobre cadastro administrativo com dados cadastrais, contato, endereco, regime, grupo e feedback com atalho para o painel do cliente criado.
- A pendencia de frontend do cadastro DEV de cliente saiu de backend parcial para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/usuarios reais, backfill de clientes legados, vinculos cliente-usuario/gestor/carteira, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-18 14:52 -03:00

- A rota React `/onboarding/entrada-cliente` ganhou `OnboardingClientEntryPage.tsx` consumindo `/api-v2/onboarding/processes` para criacao e esteira recente.
- A tela permite iniciar processo por cliente cadastrado quando houver `clients.read`, ou por dados manuais, com filtros por busca, servico, regime e status.
- A pendencia da entrada de cliente saiu de backend parcial para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/usuarios reais, maquina de estados completa, links publicos especificos, backfill, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-18 14:39 -03:00

- A rota React `/clientes/colaboradores/:id` ganhou `CollaboratorViewPage.tsx` consumindo `GET /api-v2/collaborators/:id`.
- Foram adicionados detalhe funcional, contato, status, perfis, permissoes efetivas, datas, metadados seguros e link a partir da listagem `/colaboradores/visao-geral`.
- A pendencia da visao individual de colaborador saiu de backend parcial para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/usuarios reais, backfill de `sama_colaboradores`, vinculos colaborador-cliente/gestor, carteira/transferencias, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-18 14:27 -03:00

- A rota React `/clientes/visao-geral` ganhou `ClientsOverviewPage.tsx` consumindo `/api-v2/clients` e `/api-v2/documents/required-pending-summary`.
- Foram adicionados filtros consolidados, estatisticas, coluna de documentos obrigatorios, cards de pendencias e atalhos para `/clientes/:id/painel` e `/documentos?clientId=...`.
- A pendencia de frontend da visao geral de clientes saiu de backend parcial para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/usuarios reais, backfill, vinculos cliente-usuario/gestor/carteira, escopo por departamento, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-18 12:27 -03:00

- A rota React interna `/documentos` ganhou `DocumentsPage.tsx` consumindo `/api-v2/documents`, `/download`, `/:id/status`, `/:id`, `/public-tokens` e `/required-pending-summary`.
- Foram ampliados tipos, schemas Zod e service de documentos para listagem, upload interno, download, revisao, arquivamento, resumo de pendencias e gestao de links publicos por token.
- A pendencia de frontend de documentos saiu de painel parcial para central interna parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/storage/ClamAV reais, backfill, usuarios/permissoes reais, compatibilidade operacional com links antigos, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-18 09:17 -03:00

- A rota React `/colaboradores/visao-geral` ganhou `CollaboratorsOverviewPage.tsx` consumindo `/api-v2/collaborators`.
- Foram adicionados tipo, schema Zod e service para listagem, detalhe local, filtros, cadastro, edicao, perfis, status e arquivamento com CSRF nas mutacoes.
- A pendencia de frontend de colaboradores saiu de backend parcial para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/seed reais, backfill de `sama_colaboradores`, vinculos colaborador-cliente/gestor, carteira/transferencias, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-18 08:46 -03:00

- A rota React publica `/onboarding/publico/documentos/:token` ganhou `PublicDocumentsPage.tsx` consumindo `POST /api-v2/documents/public-upload?token=...`.
- Foram adicionados tipo, schema Zod e service para upload publico multipart com token da rota, validacao visual de extensao/tamanho e historico local dos arquivos enviados na sessao.
- A pendencia de frontend de documentos publicos de onboarding saiu de backend parcial para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/storage/ClamAV reais, tokens legados, HTTPS/homologacao e Playwright.

## Atualizacao 2026-05-15 16:13 -03:00

- As rotas React `/legalizacao/contratos`, `/legalizacao/contratos/:id` e `/assinatura/:token` ganharam telas reais consumindo `/api-v2/contracts` e `/api-v2/public/signatures/:token`.
- Foram adicionados tipos, schema Zod e service `contracts.service.ts` para filtros, criacao, edicao, geracao de snapshot HTML, renderizacao PDF, importacao/download de PDF privado, envio por link publico e assinatura publica por token.
- A pendencia de frontend de contratos/assinatura saiu de nao iniciada para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/backfill reais, Legal Doc Service real, PDF assinado real, compatibilidade com tokens legados, usuarios/permissoes reais, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-15 15:06 -03:00

- As rotas React `/legalizacao/propostas`, `/legalizacao/propostas/:id` e `/onboarding/publico/proposta/:token` ganharam telas reais consumindo `/api-v2/proposals` e `/api-v2/public/proposals/:token`.
- Foram adicionados tipos, schema Zod e service `proposals.service.ts` para filtros, paginacao, criacao, edicao, envio por token publico, aprovacao/rejeicao interna e resposta publica do cliente.
- A pendencia de frontend de propostas saiu de nao iniciada para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/backfill reais, usuarios/permissoes reais, compatibilidade com tokens legados, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-15 14:13 -03:00

- A rota React `/dev` ganhou `DevAdminPage.tsx` consumindo `/api-v2/users`, `/api-v2/roles` e `/api-v2/permissions`.
- Foram adicionados tipos, schemas Zod e service `admin.service.ts` para filtros, paginacao, criacao/edicao de usuarios, troca de status, roles do usuario, criacao/edicao de roles, permissoes da role e criacao/edicao de permissoes.
- A pendencia de frontend DEV/admin saiu de backend parcial para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/seed/backfill reais, usuarios/permissoes reais, migracao de usuarios legados, presenca/estatisticas administrativas, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-15 13:51 -03:00

- A rota React `/onboarding/processos` ganhou `OnboardingProcessesPage.tsx` consumindo `/api-v2/onboarding/processes`, `/:id/status`, `/:id/timeline` e `/:id/documents`.
- Foram adicionados tipos, schemas Zod e service `onboarding.service.ts` para filtros, paginacao, criacao, edicao, arquivamento, mudanca de status/etapa, consulta de timeline e vinculo leve de documentos/requisitos/token publico.
- A pendencia de frontend de onboarding/processos saiu de backend parcial para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/backfill reais, usuarios/permissoes reais, fluxos publicos especificos, maquina de estados completa, Playwright e homologacao HTTPS.

## Atualizacao 2026-05-15 13:35 -03:00

- A rota React `/legalizacao` ganhou `LegalizationPage.tsx` consumindo `/api-v2/legalization/processes`, `/:id/status`, `/:id/timeline` e `/api-v2/legalization/templates`.
- Foram adicionados tipos, schemas Zod e service `legalization.service.ts` para filtros, paginacao, criacao, edicao, arquivamento, mudanca de status/etapa, consulta de timeline e gestao de templates.
- A pendencia de frontend de legalizacao/processos saiu de backend parcial para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/backfill reais, Legal Doc Service/PDF em homologacao, proposta/contrato/assinatura, Playwright e testes com usuarios/permissoes reais.

## Atualizacao 2026-05-15 10:24 -03:00

- A rota React `/notificacoes` ganhou `NotificationsPage.tsx` consumindo `/api-v2/notifications`, `/clear-unread`, `/:id/ack`, `/:id/alert-ack`, `/push/public-key`, `/push/test`, stream SSE e `POST /api-v2/notifications` por permissao.
- Foram adicionados tipos, schema Zod e service `notifications.service.ts` para filtros, paginacao, detalhe, criacao manual, ack/alert-ack, limpeza de nao lidas, teste Web Push, stream autenticado e assinatura/cancelamento Push API.
- A pendencia de frontend de notificacoes saiu de backend/legado parcial para parcial com experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/VAPID reais, navegadores HTTPS, portal fechado, Playwright e estrategia broker/event bus multi-instancia se necessario.

## Atualizacao 2026-05-15 10:05 -03:00

- A rota React `/auditoria` ganhou `AuditPage.tsx` consumindo `/api-v2/audit/logs` e `/api-v2/audit/logs/:id`.
- Foram adicionados tipos, schema Zod e service `audit.service.ts` para filtros por acao/modulo/entidade/usuario/periodo, paginacao e detalhe de evento.
- A pendencia de frontend de auditoria saiu de nao iniciada para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/usuarios reais, seed `audit.read`, Playwright, politica de retencao e integracao operacional em homologacao.
- Backup/restauracao continuam fora desta fatia porque ainda precisam de contrato NestJS proprio, confirmacao dupla, janela operacional e armazenamento privado/criptografado antes de sair do legado.

## Atualizacao 2026-05-15 09:44 -03:00

- A rota React `/solicitacao-acesso` ganhou `AccessRequestPage.tsx` consumindo `/api-v2/access-requests`, `/my/latest`, `/manager/approvals`, `/approve` e `/reject`.
- Foram adicionados tipos, schema Zod e service `access-requests.service.ts` para envio de solicitacao por colaborador/gestor, acompanhamento da ultima solicitacao, historico filtrado e decisoes de gestor/TI por permissao.
- A pendencia de frontend de solicitacoes de acesso saiu de backend parcial para parcial com primeira experiencia React funcional; permanecem pendentes validacao com API v2/MySQL/backfill/usuarios reais, escopo por gestor/departamento/TI, Playwright e integracao com notificacoes em navegador real.
- A rota `/ti/acessos` continua pendente porque a base editavel de acessos internos da TI pode conter segredos e deve ser modelada separadamente com criptografia/vault, auditoria de leitura e politica de retencao antes de sair do legado.

## Atualizacao 2026-05-15 09:16 -03:00

- O painel React `/clientes/:clientId/painel` ganhou `ClientDocumentsPanel.tsx` consumindo `/api-v2/clients/:clientId/documents` e rotas de documentos.
- Foram adicionados tipos, schema Zod e service `documents.service.ts` para checklist, upload multipart, download protegido, revisao de status e requisito customizado.
- A pendencia de frontend de documentos saiu de nao iniciada para parcial dentro do painel do cliente; permanecem pendentes validacao com API v2/MySQL/storage/ClamAV reais, escopo por cliente/gestor, Playwright e demais abas do painel.
- A primeira execucao de lint teve um aviso de hook no checklist; o fallback foi estabilizado com `useMemo` e lint/build passaram sem avisos.

## Atualizacao 2026-05-15 08:59 -03:00

- As rotas React `/clientes` e `/clientes/:clientId/painel` ganharam telas reais consumindo `/api-v2/clients`.
- Foram adicionados tipos, schema Zod e service `clients.service.ts` para listagem, detalhe, dashboard, cadastro, atualizacao e arquivamento logico.
- A pendencia de frontend de clientes saiu de nao iniciada/backend parcial para parcial; permanecem pendentes validacao com MySQL real, backfill legado, vinculos cliente-usuario/gestor, documentos do painel e Playwright.
- A primeira tentativa de build falhou por tipagem do schema Zod/React Hook Form; o schema foi ajustado e o build passou.

## Atualizacao 2026-05-15 08:33 -03:00

- A rota React `/certificados-digitais` ganhou `CertificatesPage.tsx` consumindo `/api-v2/certificates`.
- Foram adicionados tipos, schemas Zod e service `certificates.service.ts` para listagem, cadastro multipart, download protegido, atualizacao, rotacao de senha e remocao.
- A pendencia de frontend de certificados saiu de nao iniciada para parcial; permanecem pendentes validacao com login/API v2/MySQL/storage reais, backfill legado e Playwright.

## Atualizacao 2026-05-14 14:54 -03:00

- Criada a fundacao `portal-sama-web` com React, TypeScript, Vite, Tailwind, React Router, Axios, TanStack Query, Zustand, React Hook Form/Zod e lucide-react.
- Login React inicial usa `/api-v2/auth/csrf`, `/login`, `/refresh`, `/logout`, `/me` e `/forgot-password`, mantendo access token apenas em memoria e refresh em cookie HttpOnly da API.
- Foram adicionados `Dockerfile`, `nginx.conf` e `.env.example` para deploy do frontend. A pendencia de frontend mudou para integracao das telas reais, validacao em homologacao/HTTPS e Playwright.

## Atualizacao 2026-05-14 13:54 -03:00

- Contratos ganharam renderizacao server-side de PDF em `POST /api-v2/contracts/:id/render-pdf`, usando serviço externo configuravel compatível com `/render/html-to-pdf`.
- O fluxo renderiza o HTML salvo, valida o PDF retornado, salva em storage privado e audita hash/tamanho/engine/paginas sem expor storage key.
- A pendencia de renderizacao saiu do codigo-base e passou para validacao operacional: configurar Legal Doc Service real, testar fidelidade visual em HTTPS/homologacao, integrar frontend e planejar backfill.

## Atualizacao 2026-05-14 13:42 -03:00

- Contratos ganharam importacao e download seguro de PDF em `/api-v2/contracts/:id/pdf`.
- O upload valida extensao, MIME, assinatura `%PDF`, tamanho maximo configuravel e marcadores ativos conhecidos; o arquivo fica em storage privado e auditoria registra apenas hash/tamanho.
- A pendencia de PDF em contratos ficou dividida nesta etapa; a atualizacao seguinte adicionou renderizacao server-side, restando validar o serviço real, backfill legado, frontend e MySQL/storage reais.

## Atualizacao 2026-05-14 10:39 -03:00

- Web Push ganhou um teste operacional: `POST /api-v2/notifications/push/test` cria uma notificacao de teste para o usuario atual e retorna resumo de despacho quando VAPID esta habilitado.
- O painel legado de notificacoes agora mostra `Testar` depois do opt-in, garantindo sessao API v2, CSRF e assinatura Push API antes de acionar o teste.
- A pendencia de homologacao ficou mais objetiva: configurar VAPID real em HTTPS, abrir o portal, ativar notificacoes, clicar `Testar`, fechar o portal e repetir recebimento. Multi-instancia ainda depende de decisao sobre broker/event bus.

## Atualizacao 2026-05-14 10:25 -03:00

- Documentos e certificados receberam emissores automaticos de notificacoes internas para eventos sensiveis.
- Em documentos: upload interno/publico, revisao de status, arquivamento, requisito customizado e criacao/revogacao de link publico.
- Em certificados: cadastro, atualizacao, rotacao de senha e remocao. Os payloads nao incluem token bruto, senha, segredo criptografado nem storage key.
- Com isso, os emissores iniciais de negocio estao cobertos no codigo; permanecem pendentes a validacao manual Web Push em HTTPS/VAPID real e eventual broker/event bus para multi-instancia.

## Atualizacao 2026-05-14 10:13 -03:00

- Onboarding e legalizacao passaram a emitir notificacoes internas para responsavel/departamento em criacao, mudanca de status e arquivamento; onboarding tambem notifica vinculacao de documentos.
- Os emissores usam o ator real com permissao tecnica para criar notificacoes, mantendo o filtro contra auto-notificacao quando o responsavel e o proprio ator.
- Ainda faltam os testes manuais de Web Push em HTTPS com VAPID real e a decisao operacional sobre broker/event bus para multi-instancia.

## Atualizacao 2026-05-14 10:04 -03:00

- Propostas e contratos passaram a emitir notificacoes internas quando o cliente responde por link publico: aprovacao/ajuste de proposta e assinatura de contrato.
- As notificacoes tentam localizar o responsavel por `updatedById/createdById`, usam fallback para `Legalizacao`, dedupe por token publico e nao bloqueiam o fluxo do cliente em caso de falha.
- Ainda faltam os testes manuais de Web Push em HTTPS com VAPID real e a decisao operacional sobre broker/event bus para multi-instancia.

## Atualizacao 2026-05-14 09:49 -03:00

- A ponte temporaria do frontend legado com a API v2 agora guarda token/expiracao/CSRF separados, tenta refresh em `/api-v2/auth/refresh`, faz retry de chamadas Web Push apos `401` e executa logout API v2 coordenado.
- Adicionado `npm run webpush:vapid:generate` para preparar chaves VAPID em ambiente sem versionar segredo real.
- Ainda falta validar esse ciclo em navegador real, configurar VAPID de homologacao/producao e substituir a ponte por uma sessao React definitiva.

## Atualizacao 2026-05-14 09:35 -03:00

- Web Push saiu de pendencia conceitual para base parcial implementada: API v2 possui rotas `/api-v2/notifications/push/*`, tabela `browser_push_subscriptions`, envio via VAPID e service worker legado `sama-push-sw.js`.
- Validacoes desta etapa passaram: sintaxe JS dos arquivos legados, lint/build/audit da API, 23 suites/131 testes unitarios, 1 suite/130 e2e, `prisma:migrate:deploy/status` e diff Prisma sem diferencas.
- Ainda falta para considerar concluido: configurar VAPID real, testar em HTTPS/homologacao com navegador suportado e portal fechado, garantir bearer/refresh da API v2 no frontend definitivo, ligar emissores dos demais modulos e definir broker/event bus para multi-instancia se necessario.

## Pendências críticas

| Pendência | Área | Severidade | Arquivos relacionados | Status | Próxima ação |
|---|---|---|---|---|---|
| Criar fundação NestJS `/api-v2` | Backend | Alta | `portal-sama-api/`, `docs/GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`, `docs/ROADMAP_REFATORACAO.md` | Parcial | Fundação criada e validada com `HealthModule`, `AuthModule`, `AuditModule`, `UsersModule`, `RolesModule`, `PermissionsModule`, `ClientsModule`, `CollaboratorsModule`, `OnboardingModule`, `LegalizationModule`, `NotificationsModule`, `DocumentsModule`, `CertificatesModule`, `ProposalsModule`, `ContractsModule` e `AccessRequestsModule`; próxima ação é backfill/integração frontend e validação em homologação. |
| Criar frontend React/Vite | Frontend | Alta | `portal-sama-web/`, `docs/MATRIZ_MIGRACAO_HTML_PARA_REACT.md`, `docs/paginas/` | Parcial | Fundacao, certificados, clientes/cadastro/painel/visao geral/departamental, colaboradores, DEV colaboradores, carteira do gestor, transferencias, documentos internos, documentos do painel, documentos publicos de onboarding, solicitacoes, auditoria, notificacoes, legalizacao, onboarding/processos, DEV/admin, propostas, contratos/assinatura e TI operacional iniciais criados; Playwright inicial cobre login/documentos publicos/auditoria; proxima acao e validar login em homologacao/HTTPS, ampliar Playwright, telas gerenciais/contabeis e homologar fluxos reais. |
| Criar `prisma/schema.prisma` e migrations | Banco | Alta | `portal-sama-api/prisma/schema.prisma`, `portal-sama-api/prisma/migrations/`, `api/db.php`, `docs/BANCO_DADOS_MYSQL_PRISMA.md` | Parcial | Schema inicial e migrations incrementais existem; em 2026-05-14 09:35 as migrations de `browser_push_subscriptions` foram aplicadas no MySQL local e `migrate status/diff` passaram. Backfills e validacao em homologacao/producao seguem pendentes. |
| Criar `.env.example` seguro | Configuração | Alta | `.env.example`, `.gitignore`, `docs/SEGURANCA.md`, `docs/EASYPANEL_DEPLOY.md` | Concluído | Revisar placeholders quando o deploy EasyPanel real for configurado. |
| Formalizar comandos de teste | Testes | Alta | `portal-sama-api/package.json`, `composer.json`, `scripts/` | Parcial | Scripts da API NestJS criados; comandos PHP legados ainda dependem de PHP no PATH. |

## Pendências de segurança

| Pendência | Risco | Arquivos relacionados | Recomendação | Status |
|---|---|---|---|---|
| `.env` real existe no workspace | Vazamento de credenciais | `.env`, `.gitignore`, `.dockerignore`, `.env.example` | Não expor valores; `.env.example` criado; rotacionar segredos se já foram compartilhados. | Parcial |
| `display_errors=1` em endpoints de solicitação de acesso | Exposição de detalhes internos em produção | `SolicitacaoAcesso/enviar_acesso.php`, `SolicitacaoAcesso/verificar_codigo.php` | Debug agora é opt-in por variável de ambiente; executar lint PHP quando possível. | Corrigido no legado; lint pendente |
| Endpoints action-based exigem revisão de escopo | IDOR e autorização insuficiente | `api/storage.php`, `api/client_documents.php`, `api/certificados.php`, `api/onboarding.php`, `api/legalizacao.php`, `api/access_requests.php`, `portal-sama-api/src/modules/documents/*`, `portal-sama-api/src/modules/onboarding/*`, `portal-sama-api/src/modules/legalization/*` | Validar usuário, permissão e vínculo com recurso em cada action/rota. `OnboardingModule` e `LegalizationModule` iniciaram escopo por responsavel/criador/departamento; vínculo real cliente-usuário/gestor ainda é pendente. | Parcial |
| Tokens públicos precisam continuar centralizados | Acesso indevido por link vazado | `api/public_token_lib.php`, `api/onboarding.php`, `api/legalizacao.php`, `api/propostas.php`, `portal-sama-api/src/modules/documents/*`, `portal-sama-api/src/modules/proposals/*`, `portal-sama-api/src/modules/contracts/*` | `DocumentsModule` implementou upload público por `PublicToken`; `ProposalsModule` e `ContractsModule` reutilizam token opaco com hash SHA-256, expiração, revogação, escopo e auditoria para proposta/assinatura pública. Ainda faltam Redis para rate limit distribuído, backfill de tokens legados e validação com MySQL/homologação. | Parcial |
| Certificados digitais precisam de testes de proteção | Vazamento de arquivo/senha | `api/certificados.php`, `api/certificate_secret_lib.php`, `portal-sama-api/src/modules/certificates/*`, `portal-sama-web/src/pages/certificates/*` | `CertificatesModule` adicionou criptografia AES-GCM, validação `.p12/.pfx`, download protegido por `certificates.download`, CSRF nas mutações, auditoria e testes unitários/e2e de proteção. A tela React inicial foi integrada sem revelar senha. Falta validar storage/MySQL reais, backfill e fluxo em homologacao. | Parcial |
| CSRF da autenticação NestJS precisa ser validado em fluxo real | Requisições mutáveis com cookie HttpOnly | `portal-sama-api/src/modules/auth/*` | CSRF double-submit assinado foi implementado em Auth; validar em frontend/navegador e MySQL de homologação antes de produção. | Parcial |
| RBAC administrativo precisa de validação real | Escalada de privilégio ou leitura administrativa indevida | `portal-sama-api/src/modules/users/*`, `portal-sama-api/src/modules/roles/*`, `portal-sama-api/src/modules/permissions/*`, `portal-sama-api/prisma/seed.ts` | Rotas `GET`, seed inicial e mutações principais de usuários/roles/permissões foram implementados com CSRF, auditoria, hash de senha, validações e proteções administrativas. Validar com MySQL real, executar seed atualizado e testar escopo/dados persistidos. | Parcial |
| Upload de documentos NestJS precisa de scanner real | Upload malicioso ou arquivo persistido sem inspeção operacional | `portal-sama-api/src/modules/documents/*`, `api/client_documents_lib.php`, `scripts/check_upload_scanner.php` | Quarentena privada e scanner configuravel foram criados na API v2; falta validar ClamAV instalado no host, fixture EICAR e modo `strict` no ambiente real. | Parcial |

## Pendências de backend

| Pendência | Módulo | Arquivos relacionados | Status | Próxima ação |
|---|---|---|---|---|
| Migrar autenticação PHP para NestJS | AuthModule | `portal-sama-api/src/modules/auth/*`, `api/auth.php`, `api/storage.php`, `Login.js`, `auth.js` | Parcial | Base `/api-v2/auth/login`, `/refresh`, `/logout`, `/me`, `/csrf` e `/forgot-password` criada; falta banco real, seeds/migração de usuários e integração frontend. |
| Migrar usuários, roles e permissões | UsersModule/RolesModule/PermissionsModule | `portal-sama-api/src/modules/users/*`, `portal-sama-api/src/modules/roles/*`, `portal-sama-api/src/modules/permissions/*`, `api/storage.php`, `DEV/*`, `portal-sama-web/src/pages/dev/DevAdminPage.tsx` | Parcial | `UsersModule`, `RolesModule` e `PermissionsModule` possuem consultas e mutações principais com permissões granulares, CSRF e auditoria; `/dev` possui primeira tela React. Falta validar com MySQL real, migrar usuários legados, seed/backfill e homologar DEV/admin. |
| Migrar auditoria | AuditModule | `portal-sama-api/src/modules/audit/*`, `api/storage.php`, `api/event_hub.php`, `Auditoria/autoria.js`, `portal-sama-web/src/pages/audit/AuditPage.tsx` | Parcial | `AuditModule` inicial criado com consulta protegida por `audit.read` e exportacao CSV auditada; tela React consome listagem/detalhe/exportacao com filtros e paginacao; smoke Playwright inicial cobre a rota. Falta validar com MySQL real/usuarios reais, ampliar Playwright e definir retencao. |
| Migrar documentos | DocumentsModule | `portal-sama-api/src/modules/documents/*`, `api/client_documents.php`, `api/client_documents_lib.php`, `Client/painel.js`, `portal-sama-web/src/pages/clients/ClientDocumentsPanel.tsx` | Parcial | Rotas iniciais, template, checklist, requisitos customizados, resumo de pendências, storage privado, validação de upload, quarentena/scanner configuravel, upload público por token, emissão/listagem/revogação administrativa de tokens, auditoria, arquivamento, revisão de status com `DocumentStatusHistory`, validação centralizada de acesso e emissores de notificacoes criados; painel React inicial consome checklist/upload/download/revisao/requisitos; falta ClamAV strict no host, backfill/seed com dados reais, validação MySQL/storage/escopo e Playwright. |
| Migrar certificados | CertificatesModule | `portal-sama-api/src/modules/certificates/*`, `portal-sama-api/prisma/schema.prisma`, `api/certificados.php`, `DptClient/certificados-digitais.js`, `portal-sama-web/src/pages/certificates/CertificatesPage.tsx` | Parcial | Backend `/api-v2/certificates` criado com CRUD, rotação de senha, download, criptografia, storage privado, migration `digital_certificates`, auditoria e emissores de notificacoes; tela React inicial criada; falta validar MySQL/storage reais, backfill legado e fluxo integrado. |
| Migrar propostas | ProposalsModule | `portal-sama-api/src/modules/proposals/*`, `portal-sama-api/prisma/schema.prisma`, `api/propostas.php`, `Legalizacao/proposta.js`, `Onboarding/proposta-cliente.js`, `portal-sama-web/src/pages/proposals/*` | Parcial | Backend `/api-v2/proposals` e `/api-v2/public/proposals/:token` criado com CRUD inicial, envio por token publico opaco, aprovacao/rejeicao interna, resposta publica, CSRF e auditoria; telas React interna/publica criadas em `/legalizacao/propostas` e `/onboarding/publico/proposta/:token`; falta validar MySQL/homologacao, backfill de KV legado/tokens antigos, compatibilidade com tokens legados e Playwright. |
| Migrar contratos e assinatura | ContractsModule/SignaturesModule | `portal-sama-api/src/modules/contracts/*`, `portal-sama-api/prisma/schema.prisma`, `api/legalizacao.php`, `Legalizacao/contrato.js`, `Legalizacao/assinatura.js`, `portal-sama-web/src/pages/contracts/*` | Parcial | Backend `/api-v2/contracts` e `/api-v2/public/signatures/:token` criado com CRUD inicial, snapshot HTML, renderizacao server-side configuravel, importacao/download seguro de PDF, envio por token publico opaco, assinatura publica, CSRF e auditoria; telas React interna/publica criadas em `/legalizacao/contratos` e `/assinatura/:token`; falta baseline/migrations em MySQL real, Legal Doc Service real/homologacao, backfill legado, PDF assinado real, tokens legados e Playwright. |
| Migrar solicitações de acesso | AccessRequestsModule/TiModule | `portal-sama-api/src/modules/access-requests/*`, `api/access_requests.php`, `SolicitacaoAcesso/acesso.js`, `TI/ti.js`, `portal-sama-web/src/pages/access-requests/AccessRequestPage.tsx`, `portal-sama-web/src/pages/ti/TiAccessPage.tsx` | Parcial | Backend `/api-v2/access-requests` criado com criacao, listagem, ultima solicitacao, fila do gestor, aprovacao/rejeicao, escopo por departamento/gestor, CSRF e auditoria; emissores automaticos para gestor/solicitante/TI foram ligados ao `NotificationsModule`; `/solicitacao-acesso` e `/ti/acessos` possuem primeiras telas React. Falta backfill legado, validacao com usuarios reais, Playwright e modelagem separada da base de acessos TI. |
| Migrar clientes | ClientsModule | `portal-sama-api/src/modules/clients/*`, `portal-sama-api/prisma/schema.prisma`, `api/storage.php`, `Client/*`, `DptClient/clientdpto.html`, `DEV/dev-novo-cliente.html`, `portal-sama-web/src/pages/clients/*`, `portal-sama-web/src/pages/departments/DepartmentClientsPage.tsx` | Parcial | Backend `/api-v2/clients` criado com listagem, detalhe, criacao, atualizacao, arquivamento logico, dashboard, aliases legados, CSRF e auditoria; frontend inicial de listagem/cadastro/painel/visao geral/departamental criado; falta backfill de clientes/colaboradores, vinculos cliente-usuario/gestor/carteira/departamento, atribuicao/transferencia segura e validacao real. |
| Migrar colaboradores | CollaboratorsModule/TransfersModule | `portal-sama-api/src/modules/collaborators/*`, `portal-sama-api/src/modules/transfers/*`, `portal-sama-api/prisma/schema.prisma`, `api/storage.php`, `Client/visao-geral-colaboradores.html`, `Client/visao-colaborador.html`, `DEV/dev-colaborador.html`, `DEV/dev-novo-colaborador.html`, `Manager/manager-colaborador.html`, `portal-sama-web/src/pages/collaborators/CollaboratorsOverviewPage.tsx`, `portal-sama-web/src/pages/collaborators/CollaboratorViewPage.tsx`, `portal-sama-web/src/pages/dev/DevCollaboratorsPage.tsx`, `portal-sama-web/src/pages/dev/DevNewCollaboratorPage.tsx`, `portal-sama-web/src/pages/manager/ManagerCollaboratorsPage.tsx` | Parcial | Backend `/api-v2/collaborators` criado com listagem, detalhe, criacao, atualizacao, arquivamento logico, campos funcionais no `User`, aliases legados, CSRF, escopo por departamento e auditoria; frontend inicial `/colaboradores/visao-geral`, detalhe `/clientes/colaboradores/:id`, gestao DEV `/dev/colaboradores`, cadastro DEV `/dev/colaboradores/novo` e consulta de carteira `/manager/colaboradores` criados. Falta backfill de `sama_colaboradores`, vinculos reais colaborador-cliente/gestor, validacao MySQL/usuarios reais e Playwright. |
| Migrar onboarding | OnboardingModule | `portal-sama-api/src/modules/onboarding/*`, `portal-sama-api/prisma/schema.prisma`, `api/onboarding.php`, `Onboarding/*`, `portal-sama-web/src/pages/onboarding/OnboardingProcessesPage.tsx` | Parcial | Backend `/api-v2/onboarding/processes` criado com listagem, detalhe, criacao, atualizacao, status/etapa, timeline, documentos, arquivamento logico, aliases legados, CSRF, escopo por responsavel/criador/departamento, auditoria e emissores de notificacoes; `/onboarding/processos` possui primeira tela React. Falta maquina de estados completa, fluxos publicos especificos, backfill, validacao integrada e Playwright. |
| Migrar legalização/processos | LegalizationModule | `portal-sama-api/src/modules/legalization/*`, `portal-sama-api/prisma/schema.prisma`, `api/legalizacao.php`, `Legalizacao/*`, `portal-sama-web/src/pages/legalization/LegalizationPage.tsx` | Parcial | Backend `/api-v2/legalization/processes` e `/api-v2/legalization/templates` criado com processos internos, templates de cabecalho/rodape, aliases legados, CSRF, escopo, auditoria, rejeicao de HTML inseguro e emissores de notificacoes; `/legalizacao` possui primeira tela React para processos, status/timeline e templates. Ainda faltam backfill legado, validacao operacional do renderizador/PDF, proposta/contrato/assinatura e Playwright. |
| Migrar notificacoes | NotificationsModule | `portal-sama-api/src/modules/notifications/*`, `portal-sama-api/prisma/schema.prisma`, `api/notifications.php`, `api/notifications_stream.php`, `global.js`, `portal-sama-web/src/pages/notifications/NotificationsPage.tsx`, `portal-sama-web/src/services/notifications.service.ts`, `portal-sama-web/public/sama-push-sw.js` | Parcial | Backend `/api-v2/notifications` criado com listagem, criacao, ack, alert-ack, limpeza de nao lidas, SSE, Web Push e emissores iniciais; tela React consome listagem, criacao manual, ack/alert-ack, clear-unread, push/test, stream SSE autenticado por `fetch` e assinatura/cancelamento Push API. Falta validar navegador/homologacao, configurar VAPID real, testar portal fechado, broker/event bus se necessario, backfill/retencao e Playwright. |
| Notificacoes nativas/Web Push no navegador | Frontend legado/Notifications | `global.js`, `global.css`, `auth.js`, `Login.js`, `sama-push-sw.js`, `api/notifications_stream.php`, `portal-sama-web/public/sama-push-sw.js` | Parcial | Painel legado possui opt-in, notificacoes nativas quando o portal esta aberto sem foco, Service Worker/Push API para portal fechado e botao `Testar`; React agora tambem registra assinatura Push API, unsubscribe e stream SSE autenticado sem bearer em query string. Falta teste manual em HTTPS/homologacao, validacao de browsers, portal fechado e VAPID real. |

## Pendências de frontend

| Pendência | Tela/Rota | Arquivos relacionados | Status | Próxima ação |
|---|---|---|---|---|
| Migrar login | `/login` | `portal-sama-web/src/pages/auth/LoginPage.tsx`, `index.html`, `Login.js`, `docs/paginas/index.md` | Parcial | Tela React criada usando API v2; falta validar com MySQL/homologacao, usuario real, HTTPS e fluxo de erro/refresh em navegador. |
| Migrar clientes | `/clientes`, `/clientes/visao-geral`, `/departamentos/clientes` | `Client/clientes.html`, `Client/visao-geral-clientes.html`, `DptClient/clientdpto.html`, `DEV/dev-novo-cliente.html`, `portal-sama-api/src/modules/clients/*`, `portal-sama-web/src/pages/clients/ClientsPage.tsx`, `portal-sama-web/src/pages/clients/ClientsOverviewPage.tsx`, `portal-sama-web/src/pages/departments/DepartmentClientsPage.tsx` | Parcial | Telas React iniciais criadas com filtros, paginacao, cadastro, edicao, arquivamento, visao consolidada, pendencias documentais e consulta departamental por permissao; validar com API v2/MySQL reais, backfill, vinculos cliente-usuario/gestor/carteira/departamento, atribuicao/transferencia e Playwright. |
| Migrar colaboradores | `/colaboradores/visao-geral`, `/clientes/colaboradores/:id`, `/dev/colaboradores`, `/dev/colaboradores/novo` | `Client/visao-geral-colaboradores.html`, `Client/visao-colaborador.html`, `DEV/dev-colaborador.html`, `DEV/dev-novo-colaborador.html`, `portal-sama-api/src/modules/collaborators/*`, `portal-sama-web/src/pages/collaborators/CollaboratorsOverviewPage.tsx`, `portal-sama-web/src/pages/collaborators/CollaboratorViewPage.tsx`, `portal-sama-web/src/pages/dev/DevCollaboratorsPage.tsx`, `portal-sama-web/src/pages/dev/DevNewCollaboratorPage.tsx` | Parcial | Telas React iniciais criadas com listagem, filtros, cadastro, edicao, perfis, arquivamento, detalhe individual, gestao DEV e cadastro DEV dedicado; validar com API v2/MySQL reais, backfill, vinculos colaborador-cliente/gestor, carteira/transferencias e Playwright. |
| Migrar painel de documentos do cliente | `/clientes/:clientId/painel` | `Client/painel.html`, `Client/painel.js`, `portal-sama-api/src/modules/clients/*`, `portal-sama-api/src/modules/documents/*`, `portal-sama-web/src/pages/clients/ClientDashboardPage.tsx`, `portal-sama-web/src/pages/clients/ClientDocumentsPanel.tsx` | Parcial | Painel React inicial criado com resumo do `ClientsModule` e secao de documentos com checklist, upload, download, revisao e requisito customizado; ainda falta validar em ambiente real, integrar certificados por cliente, acessos, vida da empresa e Playwright. |
| Migrar central interna de documentos | `/documentos` | `portal-sama-api/src/modules/documents/*`, `Client/painel.html`, `Onboarding/documentos-cliente.html`, `portal-sama-web/src/pages/documents/DocumentsPage.tsx`, `portal-sama-web/src/services/documents.service.ts` | Parcial | Central React criada com listagem, filtros, upload interno, download, revisao, arquivamento, links publicos e resumo de pendencias; falta validar com API v2/MySQL/storage/ClamAV reais, backfill, usuarios/permissoes reais e Playwright. |
| Migrar certificados digitais | `/certificados-digitais` | `DptClient/certificados-digitais.html`, `DptClient/certificados-digitais.js`, `portal-sama-api/src/modules/certificates/*`, `portal-sama-web/src/pages/certificates/CertificatesPage.tsx` | Parcial | Tela React inicial criada com listagem, cadastro, download, edicao, rotacao de senha e remocao; validar com API v2/MySQL/storage reais, backfill legado e Playwright. |
| Migrar legalizacao/processos | `/legalizacao` | `Legalizacao/legalizacao.html`, `Legalizacao/legalizacao.js`, `portal-sama-api/src/modules/legalization/*`, `portal-sama-web/src/pages/legalization/LegalizationPage.tsx` | Parcial | Tela React inicial criada com filtros, listagem, cadastro, edicao, arquivamento, mudanca de status/etapa, timeline e templates; validar com API v2/MySQL/backfill reais, Legal Doc Service/PDF, usuarios/permissoes reais e Playwright. |
| Migrar onboarding/processos | `/onboarding/processos`, `/onboarding/entrada-cliente` | `Onboarding/processo.html`, `Onboarding/entrada-cliente.html`, `portal-sama-api/src/modules/onboarding/*`, `portal-sama-web/src/pages/onboarding/OnboardingProcessesPage.tsx`, `portal-sama-web/src/pages/onboarding/OnboardingClientEntryPage.tsx` | Parcial | Telas React iniciais criadas com gestao de processos, entrada dedicada de cliente, filtros, estatisticas, listagem, cadastro, edicao, arquivamento, mudanca de status/etapa, timeline e vinculo leve de documentos/requisitos/token publico; validar com API v2/MySQL/backfill reais, usuarios/permissoes reais, maquina de estados completa e Playwright. |
| Migrar documentos publicos de onboarding | `/onboarding/publico/documentos/:token` | `Onboarding/documentos-cliente.html`, `Onboarding/documentos-cliente.js`, `portal-sama-api/src/modules/documents/*`, `portal-sama-web/src/pages/documents/PublicDocumentsPage.tsx` | Parcial | Tela React publica criada com envio multipart por token via `/api-v2/documents/public-upload?token=...` e checklist publico minimo via `/api-v2/documents/public-checklist?token=...`; smoke Playwright inicial cobre a rota com API mockada. Falta validacao com API v2/MySQL/storage/ClamAV reais, tokens legados, HTTPS/homologacao e Playwright de upload. |
| Migrar propostas | `/legalizacao/propostas`, `/legalizacao/propostas/:id`, `/onboarding/publico/proposta/:token` | `Legalizacao/proposta.html`, `Legalizacao/proposta.js`, `Onboarding/proposta-cliente.html`, `Onboarding/proposta-cliente.js`, `portal-sama-api/src/modules/proposals/*`, `portal-sama-web/src/pages/proposals/*` | Parcial | Telas React interna e publica criadas com filtros/listagem, criacao, edicao, envio por link, aprovacao/rejeicao interna e resposta publica; validar com API v2/MySQL/backfill reais, usuarios/permissoes reais, tokens legados, HTTPS/homologacao e Playwright. |
| Migrar contratos/assinatura | `/legalizacao/contratos`, `/legalizacao/contratos/:id`, `/assinatura/:token` | `Legalizacao/contrato.html`, `Legalizacao/contrato.js`, `Legalizacao/assinatura.html`, `Legalizacao/assinatura.js`, `portal-sama-api/src/modules/contracts/*`, `portal-sama-web/src/pages/contracts/*` | Parcial | Telas React interna e publica criadas com filtros/listagem, criacao, edicao, geracao de snapshot HTML, renderizacao PDF, importacao/download de PDF privado, envio por link e assinatura publica por token; validar com API v2/MySQL/backfill reais, Legal Doc Service real, PDF assinado real, tokens legados, HTTPS/homologacao e Playwright. |
| Migrar auditoria | `/auditoria` | `Auditoria/autoria.html`, `Auditoria/autoria.js`, `portal-sama-api/src/modules/audit/*`, `portal-sama-web/src/pages/audit/AuditPage.tsx` | Parcial | Tela React inicial criada com filtros, paginacao e detalhe de log; validar com API v2/MySQL/usuarios reais, seed `audit.read`, Playwright e politica de retencao/exportacao. |
| Migrar solicitação de acesso/TI | `/solicitacao-acesso`, `/ti/acessos` | `SolicitacaoAcesso/`, `TI/`, `portal-sama-api/src/modules/access-requests/*`, `portal-sama-web/src/pages/access-requests/AccessRequestPage.tsx`, `portal-sama-web/src/pages/ti/TiAccessPage.tsx` | Parcial | `/solicitacao-acesso` possui primeira tela React com envio, ultima solicitacao, historico, fila de aprovacao e aprovar/rejeitar; `/ti/acessos` possui primeira tela operacional React para fila/historico/decisoes de acesso. Validar com API v2/MySQL/backfill/usuarios reais, escopo por TI/gestor/departamento e Playwright. A base de segredos da TI segue pendente ate modelagem com criptografia/vault. |

| Migrar DEV/admin | `/dev`, `/dev/clientes/novo`, `/dev/colaboradores`, `/dev/colaboradores/novo` | `DEV/dev.html`, `DEV/dev-main.js`, `DEV/dev-data.js`, `DEV/dev-colaborador.html`, `DEV/dev-novo-cliente.html`, `DEV/dev-novo-colaborador.html`, `portal-sama-api/src/modules/users/*`, `portal-sama-api/src/modules/roles/*`, `portal-sama-api/src/modules/permissions/*`, `portal-sama-api/src/modules/clients/*`, `portal-sama-api/src/modules/collaborators/*`, `portal-sama-web/src/pages/dev/DevAdminPage.tsx`, `portal-sama-web/src/pages/dev/DevCollaboratorsPage.tsx`, `portal-sama-web/src/pages/dev/DevNewClientPage.tsx`, `portal-sama-web/src/pages/dev/DevNewCollaboratorPage.tsx` | Parcial | Tela React inicial criada com usuarios, roles, permissoes, cadastro administrativo de cliente, gestao administrativa de colaboradores e cadastro administrativo de colaborador; validar com API v2/MySQL/seed/backfill reais, usuarios/permissoes reais, migracao de usuarios legados, presenca/estatisticas, vinculos cliente/colaborador-usuario/gestor/carteira e Playwright. |

## Pendências de banco de dados

| Pendência | Tabela/Migration | Status | Próxima ação |
|---|---|---|---|
| Converter schema imperativo para Prisma | `api/db.php`, `portal-sama-api/prisma/schema.prisma` | Em andamento | Migration inicial criada; comparar schema real do MySQL, ajustar modelos e criar migrations incrementais/backfills. |
| Validar banco local/produção | Todas as tabelas em `api/db.php` | ponto a validar | Comparar schema real do MySQL com código. |
| Documentar campos sensíveis | `sama_certificados_clientes`, `digital_certificates`, `sama_usuarios_login`, `sama_public_tokens` | Parcial | `digital_certificates.passwordCipher` guarda senha criptografada e arquivos ficam fora do MySQL em storage privado; falta plano de backfill/retenção dos certificados legados. |
| Validar migration de propostas | `20260513085700_add_proposals` / `proposals` | Parcial | Baseline local resolvido em 2026-05-13 10:50; migration consta aplicada no MySQL local e diff Prisma ficou vazio. Repetir processo com backup em homologacao/producao e planejar backfill. |
| Validar migration de contratos | `20260513092300_add_contracts` / `contracts` | Parcial | Baseline local resolvido em 2026-05-13 10:50; migration consta aplicada no MySQL local e diff Prisma ficou vazio. Repetir processo com backup em homologacao/producao e planejar backfill/render PDF. |
| Validar migration de solicitações de acesso | `20260513101500_add_access_requests` / `access_requests` | Parcial | Baseline local resolvido em 2026-05-13 10:50; migration consta aplicada no MySQL local e diff Prisma ficou vazio. Repetir processo com backup em homologacao/producao e planejar backfill de `sama_access_requests`. |
| Validar migration de clientes | `20260513113000_expand_clients_profile` / `clients` | Parcial | Aplicada no MySQL local por `prisma:migrate:deploy`; `prisma:migrate:status` passou e `prisma migrate diff` nao encontrou diferencas. Repetir processo com backup em homologacao/producao e planejar backfill. |
| Validar migration de colaboradores | `20260513124500_add_collaborator_profile_to_users` / `users` | Parcial | Aplicada no MySQL local por `prisma:migrate:deploy`; `prisma:migrate:status` passou e `prisma migrate diff` nao encontrou diferencas. Repetir processo com backup em homologacao/producao e planejar backfill de `sama_colaboradores`. |
| Validar migration de onboarding | `20260513152000_add_onboarding_processes` / `onboarding_processes` | Parcial | Aplicada no MySQL local por `prisma:migrate:deploy`; `prisma:migrate:status` passou e `prisma migrate diff` nao encontrou diferencas. Repetir processo com backup em homologacao/producao e planejar backfill de `onboarding_process`. |
| Validar migration de legalizacao | `20260513160000_add_legalization_processes` / `legalization_processes` | Parcial | Aplicada no MySQL local por `prisma:migrate:deploy`; `prisma:migrate:status` passou e `prisma migrate diff --from-schema-datasource --to-schema-datamodel` nao encontrou diferencas. Repetir processo com backup em homologacao/producao e planejar backfill legado. |
| Validar indices relacionais de baseline | `20260513160500_add_missing_relation_indexes` | Parcial | Migration complementar aplicada no MySQL local para alinhar indices esperados por Prisma/MySQL; repetir com backup em homologacao/producao. |
| Validar migration de templates de legalizacao | `20260513164000_add_legalization_templates` / `legalization_templates` | Parcial | Aplicada no MySQL local por `prisma:migrate:deploy`; `prisma:migrate:status` passou, diff Prisma nao encontrou diferencas e seed RBAC atualizou `legalization.templates`. Repetir processo com backup em homologacao/producao e planejar backfill legado. |
| Validar migration de notificacoes | `20260513170000_add_notifications_model` / `notifications` | Parcial | Aplicada no MySQL local por `prisma:migrate:deploy`; `prisma:migrate:status` passou, diff Prisma nao encontrou diferencas e seed RBAC atualizou `notifications.read/create`. Repetir processo com backup em homologacao/producao e validar tabela legada existente. |
| Validar migrations de Web Push | `20260514092000_add_browser_push_subscriptions`, `20260514093500_align_browser_push_updated_at` / `browser_push_subscriptions` | Parcial | Aplicadas no MySQL local por `prisma:migrate:deploy`; `prisma:migrate:status` passou e diff Prisma nao encontrou diferencas. Repetir processo com backup em homologacao/producao, configurar VAPID e validar retencao/revogacao de subscriptions. |

## Pendências de testes

| Pendência | Tipo de teste | Status | Próxima ação |
|---|---|---|---|
| PHP lint dos endpoints críticos | PHP lint | Bloqueado | Instalar/disponibilizar `php` no PATH e rodar `php -l`; tentativa registrada em `.ai-tests/runs/2026-05-08-09-31-04/`. |
| Smoke Integra-AI | Smoke PHP/Python | Não executado | Validar dependências e rodar `php scripts/integra_ai_smoke_current_flow.php`. |
| Scanner de upload | Smoke operacional | Parcial | Unitarios NestJS passaram para quarentena/scanner; rodar `php scripts/check_upload_scanner.php` e teste EICAR/ClamAV no host quando PHP/ClamAV estiverem disponíveis. |
| Build/lint Node | npm | Parcial | `portal-sama-api`, `portal-sama-web` possuem lint/build; usar `npm.cmd` no PowerShell. |
| Testes NestJS/React | Jest/Supertest/Playwright | Parcial | Jest/Supertest criado para `portal-sama-api`; AuthService, CsrfService, AuditService, catalogo RBAC, Users/Roles/Permissions, Clients, Collaborators, Onboarding, Legalization, Notifications, Documents, Certificates, Proposals, Contracts e AccessRequests cobertos, incluindo CSRF, criptografia de senha, download autorizado, ausencia de senha em JSON, token publico opaco, assinatura publica, renderizacao/importacao/download seguro de PDF de contratos, aprovacao/rejeicao de acesso, SSE de notificacoes, `last_id`/catch-up por banco, emissores de negocio e Web Push. Frontend React agora possui Playwright inicial com smoke de `/login`, documentos publicos e auditoria; ainda falta ampliar cobertura para fluxos reais, mobile, permissoes e homologacao. |
| Validação de propostas com MySQL real | Integração backend/banco | Parcial | Unitários/e2e locais passaram para `ProposalsModule` (15 suítes/78 unitários e 1 suíte/59 e2e); falta aplicar migration e testar fluxo real em MySQL/homologação. |
| Validação de contratos com MySQL real | Integração backend/banco | Parcial | Unitários/e2e locais passaram para `ContractsModule` (16 suítes/83 unitários e 1 suíte/66 e2e); falta baseline, aplicar migration e testar fluxo real em MySQL/homologação. |
| Validação de solicitações de acesso com MySQL real | Integração backend/banco | Parcial | Unitários/e2e locais passaram para `AccessRequestsModule` (17 suítes/88 unitários e 1 suíte/75 e2e); falta baseline, aplicar migration, backfill de `sama_access_requests` e testar fluxo real em MySQL/homologação. |
