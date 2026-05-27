# Ainda Falta Para Deploy Em Producao - Portal Sama

Ultima atualizacao: 2026-05-27 10:03 -03:00
Responsavel/IA: Codex
Status do documento: obrigatorio e vivo

## Objetivo

Este documento e o painel oficial do que ainda falta para colocar a nova aplicacao do Portal Sama em producao, sem depender da aplicacao antiga.

Ele deve responder sempre:

- qual e o estado real da nova stack;
- quanto falta, em percentual aproximado, para homologacao e producao;
- quais bloqueios ainda existem;
- o que precisa ser feito no EasyPanel;
- como separar `portal-sama-api`, `portal-sama-web` e `portal-sama-docs` em repositorios Bitbucket;
- quais validacoes precisam passar antes do corte para producao.

## Regra obrigatoria de atualizacao

Este arquivo deve ser lido e atualizado em todo ciclo que envolva:

- deploy, EasyPanel, Docker, Nginx, HTTPS, dominio ou variaveis de ambiente;
- Bitbucket, separacao de repositorios ou mudanca de workspace;
- MySQL, Prisma, migrations, seed, bootstrap, backfill ou banco `banco-sama`;
- usuarios reais, permissoes reais, roles ou RBAC;
- storage privado, upload/download, ClamAV ou documentos sensiveis;
- QA visual, Playwright, smoke publico ou homologacao real;
- qualquer resposta sobre "quanto falta", "pronto para producao" ou "dependencia do legado".

Antes de atualizar este documento, a IA/desenvolvedor deve:

1. Ler `INSTRUÇÕEDS_AI.MD`.
2. Ler `STATUS_IMPLEMENTACAO.md`.
3. Ler `PENDENCIAS_TECNICAS.md`.
4. Ler `EASYPANEL_DEPLOY.md`.
5. Ler `RELATORIO_TESTES.md`.
6. Ler `CHANGELOG_TECNICO.md`.
7. Ler os arquivos de codigo afetados em `portal-sama-api` e/ou `portal-sama-web`.
8. Verificar o estado real do Git e dos arquivos existentes.
9. Atualizar este documento com data/hora, percentual e pendencias reais.

Se uma porcentagem for informada, ela deve ser marcada como estimativa operacional, nao como metrica matematica exata.

## Percentual atual

Estimativa operacional em 2026-05-27 10:03 -03:00:

| Marco | Percentual | Leitura |
|---|---:|---|
| Pronto para iniciar/continuar homologacao da nova stack | 99% | Deploy publico, smoke real, banco, migrations, RBAC, usuarios privilegiados, storage e ClamAV strict ja foram validados no container real via readiness; tambem passaram smoke/auth real, Playwright real e matriz minima DEV/bootstrap contra HTTPS. O restore drill agora possui comando e plano operacional, mas ainda falta executar com artefatos reais. |
| Pronto para producao sem depender do legado | 85% | A stack nova esta operacional no EasyPanel, o readiness real passou com warning apenas de backup/rollback, o fluxo publico de documentos de onboarding esta alinhado aos contratos documentados e ha evidencia real de login/refresh/logout, Playwright real e RBAC minimo DEV/bootstrap. O plano de rollback/restore drill foi instrumentalizado; permanecem pendentes backfills/conferencias reais, matriz por multiplos perfis, restore drill real, QA final, validacao real de uploads e desligamento seguro do legado. |
| Falta para producao | 15% | Principalmente execucao/conferencia real de backfills, evidencias funcionais por todos os perfis, backup externo verificado com restore testado, upload/download real de documentos, QA final e plano de corte/desligamento do legado. |

## Estado atual resumido

| Area | Estado | Observacao |
|---|---|---|
| Backend `portal-sama-api` | Deployado e operacional | NestJS/API v2 esta rodando no EasyPanel; logs mostram `Nest application successfully started`; smoke publico confirmou `/api-v2/health` com `database=up` e `storage=up`; aliases publicos de documentos de onboarding foram implementados. Falta validar fluxos de negocio e permissoes com dados reais. |
| Frontend `portal-sama-web` | Deployado e operacional | React/Vite esta publicado em `https://portal.samacontabil.com.br/`; smoke publico validou frontend/proxy/CORS/CSRF; `smoke:auth`, matriz minima DEV/bootstrap e `test:e2e:real` passaram em HTTPS com usuario real de bootstrap. Falta repetir por perfis operacionais e validar fluxos reais de negocio. |
| Banco MySQL/Prisma | Operacional em homologacao | EasyPanel possui servico `portal-sama-database`; log mostra MySQL Community Server 9.7.0 pronto em `:3306`; readiness real confirmou database `SELECT 1` e 19 migrations Prisma aplicadas. Falta rodar relatorio de backfill e conferir dados por modulo. |
| Primeiro admin/usuarios API v2 | Parcial operacional | Usuario(s) inicial(is) ja foram criados pelo operador e a aplicacao esta conectada ao banco real; falta registrar matriz de usuarios/perfis reais e validar permissoes por papel. |
| EasyPanel | Deployado | Projeto `portal-sama` contem `portal-sama-api`, `portal-sama-database` e `portal-sama-web` ativos. Readiness real passou com warning apenas de backup/rollback; falta restore drill e hardening operacional final. |
| Workspaces locais | Separado e preparado | Existem pastas irmas `portal-sama-docs`, `portal-sama-api` e `portal-sama-web` no Desktop; API/Web agora possuem README minimo com regra de leitura do repo de docs. |
| Bitbucket/Git | Remotos existentes | Repositorios Bitbucket existem para API, Web e documentacao (`portal-sama-api`, `portal-sama-web`, `portal-sama-doc` conforme evidencia visual). Falta apenas garantir push das atualizacoes documentais locais desta rodada. |
| Builds locais separados | Validado | Passaram lint/build da API, `prisma:validate` e lint/build do Web nos repos separados. |
| Storage privado | Parcial validado | Health publico retornou `storage=up` e readiness real confirmou leitura/escrita em `/var/private/portal-sama`. O Web ja aponta para o upload publico documentado de onboarding, mas ainda falta validar persistencia do volume, permissoes, backup externo e fluxos reais de upload/download. |
| ClamAV | Validado em homologacao real | Readiness real detectou EICAR com `/usr/bin/clamscan` em modo strict. Manter `npm run ops:clamav:update` como rotina operacional quando assinaturas precisarem ser atualizadas. |
| Usuarios/permissoes reais | Parcial validado | Usuario bootstrap/DEV autenticou em HTTPS e passou matriz minima 401/200 para `/auth/me`, `/users`, `/roles` e `/permissions`. Falta matriz oficial com CLIENT, MANAGER, departamentos e perfis operacionais, incluindo 403 esperados. |
| Backfills | Parcial instrumentalizado | `portal-sama-api` agora possui `npm run ops:backfill:report`, read-only, para listar tabelas atuais/legadas, contagens, usuarios sem role, roles sem permissoes e vinculos quebrados. Falta rodar no MySQL real e executar/conferir os backfills. |
| Backups/rollback | Parcial instrumentalizado | `portal-sama-api` possui `npm run ops:backup:create`, `npm run ops:backup:verify` e agora `npm run ops:restore:drill`, com dry-run, bloqueio de alvo igual ao banco/storage da aplicacao e confirmacao explicita para aplicar restore. Falta rodar no EasyPanel, copiar artefatos para fora do container e provar restore drill real. |
| Playwright real | Parcial validado | `npm.cmd run test:e2e:real` passou com usuario real de bootstrap/DEV contra `https://portal.samacontabil.com.br`, validando login, Home, storage do navegador, cookie HttpOnly/SameSite/Secure e logout. Falta repetir por perfis criticos e ampliar fluxos. |
| QA visual | Parcial | Aplicacao publica esta acessivel; ainda necessario aprovar desktop/mobile no EasyPanel/navegador real com usuarios e perfis reais. |
| Dependencia do legado | Parcial | Integra-AI ja possui mutacoes API v2 e agora importacao OFX opt-in atras de feature flag, mas ainda exige validacao funcional real/backfills e demais validacoes reais antes de desligar PHP. |

## Atualizacao operacional de 2026-05-27 10:03 -03:00

Foi analisada a documentacao externa `Downloads/docs/integra-ai/analise_semantica_integra_ai_portal_sama.md`. A recomendacao condiz com o estado do projeto: o parser Python ja suporta OFX, mas API/Web publicavam apenas PDF. A implementacao foi feita como rollout controlado:

- `portal-sama-api` ganhou `parseFile({ ext })` no parser, validacao/storage genericos para PDF/OFX e capabilities `pdf_import`/`ofx_import`.
- OFX fica desabilitado por padrao por `SAMA_INTEGRA_AI_OFX_IMPORT_ENABLED=false`; quando habilitado, a API valida extensao, MIME, assinatura OFX, tamanho, scanner/quarentena e audita `source_type=ofx`.
- `portal-sama-web` ajustou `/contabil/integra-ai` para aceitar dinamicamente PDF ou OFX conforme capability, mantendo compatibilidade com ambientes antigos que so retornam `import_mutation`.
- Nao houve migration de banco; o campo legado `source_type` passa a registrar `pdf` ou `ofx`.
- Validacao local: `npm.cmd test -- accounting.service.spec.ts --runInBand`, `npm.cmd run lint`, `npm.cmd run build` e `npm.cmd run prisma:validate` passaram na API; `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e -- -g "Integra-AI"` e `npm.cmd run test:e2e` passaram no Web.

Pendencia real remanescente: habilitar a flag somente em homologacao controlada, importar OFX real com usuario contabil, parser Python do container, MySQL/storage reais, ClamAV strict e evidencia sanitizada. Percentuais operacionais permanecem inalterados, porque a mudanca reduz risco tecnico mas nao substitui homologacao real.

## Atualizacao operacional de 2026-05-27 08:31 -03:00

Nesta rodada foi reduzida uma pendencia local/parcial do fluxo Integra-AI no frontend:

- `portal-sama-web/src/pages/accounting/IntegraAiPage.tsx` recebeu labels acessiveis nos campos customizados de contas.
- `portal-sama-web/tests/e2e/smoke.spec.ts` ganhou smoke Playwright para `/contabil/integra-ai`, com API mockada, cobrindo job aberto, pagina 2 da tabela de regras, busca por nome de conta, selecao de sugestao e autosave por blur/codigo numerico.
- `portal-sama-web/playwright.config.ts` passou a executar os smokes locais do mesmo arquivo em ordem para evitar concorrencia Vite/GSAP observada no Windows; `npm.cmd run test:e2e` voltou a passar no comando padrao.
- Validacao local: `npm.cmd run test:e2e -- -g "Integra-AI"`, `npm.cmd run test:e2e`, `npm.cmd run lint` e `npm.cmd run build` passaram no Web.

Pendencia real remanescente: validar o fluxo com job real do Integra-AI, usuario contabil, dados reais e MySQL real em homologacao apos deploy. Percentuais operacionais permanecem inalterados, porque esta rodada adiciona cobertura local e nao substitui a homologacao funcional real.

## Atualizacao operacional de 2026-05-26 16:50 -03:00

Nesta rodada foi criado o plano executavel de rollback/restore drill:

- `portal-sama-api` agora expoe `npm run ops:restore:drill`.
- O comando roda em dry-run por padrao, chama `ops:backup:verify`, valida o alvo de banco e storage, bloqueia alvo igual a `DATABASE_URL`/`STORAGE_PRIVATE_PATH` e exige `--confirm RESTORE_DRILL_TARGET_IS_ISOLATED` para aplicar restore.
- `PLANO_ROLLBACK_RESTORE_DRILL.md` foi criado com procedimento, evidencias esperadas e criterio para fechar a pendencia de backup/rollback.
- Validacao local: `node --check`, help do comando, backup descartavel sem banco com archive de storage, dry-run do restore, restore de storage em `.ai-tests`, lint/build/Prisma validate da API e `git diff --check` passaram.

Pendencia real remanescente: gerar backup real no EasyPanel, verificar artefatos reais, copiar para fora do container, criar banco/storage isolados e executar `ops:restore:drill` com evidencia sanitizada.

## Atualizacao operacional de 2026-05-26 09:38 -03:00

Depois que o `.env` real da API foi alocado localmente e confirmado como ignorado pelo Git:

- `DATABASE_URL` foi carregado sem imprimir segredo; ele aponta para `portal-sama_portal-sama-database:3306`, hostname interno do EasyPanel, portanto nao conecta a partir do Windows local.
- `npm.cmd run ops:backfill:report -- --soft --json` confirmou bloqueio local de conectividade com esse hostname interno.
- `npm.cmd run ops:readiness -- --skip-clamav --storage-path .ai-tests/readiness-storage --soft --json` confirmou o mesmo bloqueio de banco local e agora tambem falha se segredos locais (`JWT_*`, `CSRF_SECRET`, `CERTIFICATE_ENCRYPTION_KEY`) tiverem menos de 32 caracteres.
- Usando `SAMA_BOOTSTRAP_ADMIN_USERNAME`/`SAMA_BOOTSTRAP_ADMIN_PASSWORD` apenas no processo do shell, `npm.cmd run smoke:auth -- --soft --json` passou contra HTTPS: CSRF, login, `/auth/me`, refresh, novo `/auth/me` e logout.
- Uma matriz minima `BOOTSTRAP_ADMIN` passou em `npm.cmd run smoke:permissions -- --soft --json`: anonimos 401 em `/auth/me`, `/users`, `/documents`; bootstrap/DEV 200 em `/auth/me`, `/users`, `/roles`, `/permissions`.
- `npm.cmd run test:e2e:real` passou com usuario real bootstrap/DEV.
- `npm.cmd run homologation:real -- --soft --evidence-dir .ai-tests/homologation-real` passou a bateria consolidada real e gerou evidencia JSON sanitizada local.

Pendencia real remanescente: rodar backfill/backup/restore dentro do console do container `portal-sama-api` no EasyPanel, gerar matriz oficial por multiplos perfis reais, validar upload/download real com token/storage e executar QA final antes do desligamento seguro do legado.

## Atualizacao operacional de 2026-05-26 09:23 -03:00

Nesta rodada foi iniciada a execucao dos itens reais pendentes com o ambiente local disponivel:

- `npm.cmd run smoke:public` passou novamente contra `https://portal.samacontabil.com.br`, com frontend 200, health `database=up/storage=up`, CORS 204 e CSRF 200.
- `npm.cmd run smoke:auth -- --soft --json` foi executado e bloqueou por ausencia de `PORTAL_AUTH_USERNAME`/`PORTAL_AUTH_PASSWORD` no shell local.
- `npm.cmd run smoke:permissions -- --soft --json` validou os anonimos reais com 401 em `/auth/me`, `/users` e `/documents`, mas bloqueou a matriz autenticada por ausencia de `PORTAL_PERMISSION_MATRIX_FILE`/`PORTAL_PERMISSION_MATRIX_JSON`.
- `npm.cmd run test:e2e:real` foi executado e ficou `skipped`, como esperado, porque `PORTAL_REAL_E2E=1` e credenciais Playwright reais nao estavam definidos.
- `npm.cmd run ops:backfill:report -- --soft --json` foi executado e bloqueou por ausencia de `DATABASE_URL` no ambiente local.
- `npm.cmd run ops:readiness -- --soft --json` agora retorna JSON controlado com falhas de ambiente/banco/storage ausentes, em vez de abortar a coleta por erro Prisma.
- `portal-sama-web` agora expoe `npm.cmd run homologation:real`, que roda `smoke:public`, `smoke:auth`, `smoke:permissions` e `test:e2e:real` em sequencia, com preflight sanitizado de variaveis e evidencia JSON opcional em `.ai-tests/homologation-real`.

Pendencia real remanescente: carregar credenciais reais/matriz no shell, rodar `npm.cmd run homologation:real -- --evidence-dir .ai-tests/homologation-real`, executar backfill/backup/restore no container EasyPanel com `DATABASE_URL`/storage reais e anexar evidencias sanitizadas.

## Atualizacao operacional de 2026-05-26 09:10 -03:00

Nesta rodada foi disponibilizado o verificador operacional de artefatos de backup:

- `portal-sama-api` agora expoe `npm run ops:backup:verify`.
- O script verifica `metadata.json`, status dos passos, SHA-256/tamanho de `database.sql.gz`, integridade gzip, hash/consistencia de `storage-manifest.json` e, quando existir, SHA-256/listagem de `storage.tar.gz`.
- O readiness passou a orientar `ops:backup:create -> ops:backup:verify -> copiar artefatos para fora -> restore drill`.
- Validacoes locais: `node --check`, help do comando, backup local descartavel com `--skip-database`/manifesto/archive, verificacao JSON do backup gerado, readiness local com skips, build da API, `prisma:validate` com `DATABASE_URL` dummy e `git diff --check` passaram em `portal-sama-api`.

Pendencia real remanescente: rodar `npm run ops:backup:create` e `npm run ops:backup:verify` no EasyPanel com banco/storage reais, copiar artefatos para fora do container e executar restore drill em alvo isolado. O verificador nao substitui a restauracao real.

## Atualizacao operacional de 2026-05-26 08:49 -03:00

Nesta rodada foi disponibilizado o smoke operacional de matriz de permissoes por perfil:

- `portal-sama-web` agora expoe `npm.cmd run smoke:permissions`.
- O script valida anonimos esperando 401 em `/auth/me`, `/users` e `/documents`.
- A matriz autenticada e parametrizada por JSON/arquivo e aceita `usernameEnv`/`passwordEnv` para evitar credenciais em arquivos versionados.
- Cada perfil pode declarar checks por metodo, rota e status esperado, por exemplo ADMIN 200 em `/users`, CLIENT 403 em `/users` e CLIENT 200 em `/documents`.
- O script mantem cookies apenas em memoria, mascara usuarios e nao imprime senha, access token, refresh token ou CSRF token.
- Validacoes locais: `node --check`, help do comando, smoke `--soft --json` contra porta fechada, smoke positivo contra servidor fake local, lint, build e `git diff --check` passaram em `portal-sama-web`.

Pendencia real remanescente: rodar `npm.cmd run smoke:permissions` contra `https://portal.samacontabil.com.br/api-v2` com usuarios reais por perfil, sem registrar credenciais/tokens/cookies, e anexar evidencia sanitizada da matriz 401/403/200.

## Atualizacao operacional de 2026-05-25 17:54 -03:00

Nesta rodada foi disponibilizada a base Playwright para homologacao real de autenticacao via navegador:

- `portal-sama-web` agora expoe `npm.cmd run test:e2e:real`.
- A config `playwright.real.config.ts` aponta para `https://portal.samacontabil.com.br`, nao sobe servidor local e desliga trace, screenshot e video.
- A spec `tests/e2e/real-auth.spec.ts` fica bloqueada por `PORTAL_REAL_E2E=1` e credenciais em variaveis de ambiente.
- O teste cobre login pela tela React, Home autenticada, ausencia de chaves sensiveis no storage do navegador, refresh cookie fora de `document.cookie`, politica `HttpOnly`/`SameSite`/`Secure` e logout.
- Validacoes locais: `npm.cmd run test:e2e:real` sem variaveis passou com 1 skipped; `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e` com 9 passados e 1 skipped, e `git diff --check` passaram em `portal-sama-web`.

Pendencia real remanescente: rodar `PORTAL_REAL_E2E=1 npm.cmd run test:e2e:real` contra `https://portal.samacontabil.com.br` com usuario real de homologacao, sem registrar credenciais/tokens/cookies, e repetir por perfis criticos.

## Atualizacao operacional de 2026-05-25 17:37 -03:00

Nesta rodada foi disponibilizada a ferramenta operacional para validar autenticacao real no EasyPanel:

- `portal-sama-web` agora expoe `npm.cmd run smoke:auth`.
- O script executa `csrf -> login -> me -> refresh -> me -> logout`.
- O smoke mantem cookies apenas em memoria, mascara o usuario e nao imprime senha, access token, refresh token ou CSRF token.
- A validacao local passou com servidor fake e o script checa `HttpOnly`, `SameSite` e `Secure` no refresh cookie quando a API esta em HTTPS.

Pendencia real remanescente: rodar `npm.cmd run smoke:auth` contra `https://portal.samacontabil.com.br/api-v2` com usuario real de homologacao, sem registrar credenciais ou tokens, e repetir por perfis criticos.

## Atualizacao operacional de 2026-05-25 17:27 -03:00

Nesta rodada foi concluida uma pendencia de aderencia entre documentacao, API e Web no fluxo publico de documentos de onboarding:

- `portal-sama-api` agora expoe os aliases documentados `GET /api-v2/public/onboarding/documents/:token` e `POST /api-v2/public/onboarding/documents/:token/upload`, mantendo os endpoints genericos antigos como compatibilidade.
- `portal-sama-web` agora consome esses aliases na pagina publica de documentos do cliente.
- Foi adicionado teste unitario focado na controller de documentos e o smoke Playwright local do Web passou mockando o alias novo.
- A documentacao viva foi atualizada em `STATUS_IMPLEMENTACAO.md`, `PENDENCIAS_TECNICAS.md`, `RELATORIO_TESTES.md`, `CHANGELOG_TECNICO.md`, matrizes de migracao, seguranca, inventario e pagina de onboarding.

Validacoes locais registradas nesta rodada: teste unitario da API, lint da API, build da API, `prisma:validate`, lint do Web, build do Web, smoke Playwright local do fluxo publico de documentos e `git diff --check` nos tres repositorios.

Pendencia real remanescente: validar no EasyPanel o fluxo publico completo com token real, MySQL real, storage privado real, ClamAV em modo strict e upload de arquivo real pelo navegador.

## Separacao em repositorios Bitbucket

### Estado local verificado em 2026-05-22 15:16 -03:00

As pastas ja foram separadas no workspace local:

```txt
C:\Users\Sama Contabilidade\Desktop\portal-sama-docs
C:\Users\Sama Contabilidade\Desktop\portal-sama-api
C:\Users\Sama Contabilidade\Desktop\portal-sama-web
```

Estado encontrado:

| Workspace | Estado local | Git | Observacao |
|---|---|---|---|
| `portal-sama-docs` | Separado | Git local inicializado em `main` | Contem os documentos na raiz do repo, com `paginas/` e `design/`; `.gitignore` e `.gitattributes` preparados. |
| `portal-sama-api` | Separado | Git local inicializado em `main` | Contem `.env`, `node_modules/` e `dist/` ignorados; `.gitignore` e `.gitattributes` preparados. |
| `portal-sama-web` | Separado | Git local inicializado em `main` | Contem `node_modules/`, `dist/`, `playwright-report/` e `test-results/` ignorados; `.gitignore` e `.gitattributes` preparados. |

Ponto importante: a documentacao esta na raiz de `portal-sama-docs`, nao dentro de uma subpasta `docs/`. Ao trabalhar nesse repositorio separado, leia os arquivos diretamente na raiz, por exemplo `INSTRUÇÕEDS_AI.MD` e `STATUS_IMPLEMENTACAO.md`.

Estado real informado em 2026-05-25 11:46 -03:00: os repositorios Bitbucket ja existem e o EasyPanel ja esta consumindo os repos tecnicos para deploy. A evidencia visual mostra:

```txt
portal-sama-api
portal-sama-web
portal-sama-doc
```

O workspace local de documentacao continua chamado `portal-sama-docs`. Se o remoto permanecer como `portal-sama-doc`, tratar esse nome como alias operacional do repositorio de documentacao.

Topologia-alvo de repositorios:

```txt
portal-sama-api
portal-sama-web
portal-sama-docs ou portal-sama-doc
```

Regra central: qualquer IA, pessoa desenvolvedora ou manutencao operacional deve ler primeiro o repositorio `portal-sama-docs` antes de alterar `portal-sama-api` ou `portal-sama-web`.

### Repositorio `portal-sama-api`

Conteudo esperado: apenas os arquivos de `portal-sama-api/` como raiz do repositorio.

Incluir:

```txt
Dockerfile
.dockerignore
package.json
package-lock.json
nest-cli.json
tsconfig.json
tsconfig.build.json
eslint.config.mjs
prisma/
src/
test/
scripts/
.env.example
```

Nao incluir:

```txt
.env
node_modules/
dist/
coverage/
.ai-tests/
logs/
arquivos privados
segredos reais
```

No EasyPanel, se o repo for realmente separado, o Dockerfile da API fica na raiz:

```txt
Dockerfile
```

Se o repo atual monolitico for usado temporariamente, configurar Root Directory como:

```txt
portal-sama-api
```

### Repositorio `portal-sama-web`

Conteudo esperado: apenas os arquivos de `portal-sama-web/` como raiz do repositorio.

Incluir:

```txt
Dockerfile
.dockerignore
nginx.conf
package.json
package-lock.json
vite.config.ts
index.html
public/
src/
tests/
playwright.config.ts
.env.example, se existir
```

Nao incluir:

```txt
.env
.env.local
node_modules/
dist/
coverage/
test-results/
playwright-report/
.ai-tests/
segredos reais
```

No EasyPanel, se o repo for realmente separado, o Dockerfile do Web fica na raiz:

```txt
Dockerfile
```

Se o repo atual monolitico for usado temporariamente, configurar Root Directory como:

```txt
portal-sama-web
```

### Repositorio `portal-sama-docs`

Conteudo esperado: toda a documentacao do projeto, incluindo os documentos de acompanhamento, arquitetura, seguranca, deploy, matrizes, inventarios, paginas mapeadas e design.

Incluir:

```txt
*.md
paginas/
design/
README.md especifico do repositorio de documentacao
.gitignore
```

O repositorio `portal-sama-docs` deve conter, no minimo:

```txt
INSTRUÇÕEDS_AI.MD
AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD
STATUS_IMPLEMENTACAO.md
CHANGELOG_TECNICO.md
DECISOES_ARQUITETURA.md
PENDENCIAS_TECNICAS.md
RELATORIO_TESTES.md
EASYPANEL_DEPLOY.md
README_DOCUMENTACAO.md
SEGURANCA.md
INVENTARIO_SEGURANCA.md
INVENTARIO_BANCO_DADOS.md
INVENTARIO_ENDPOINTS.md
MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md
MATRIZ_MIGRACAO_HTML_PARA_REACT.md
paginas/
design/
```

Nao incluir:

```txt
.env
node_modules/
.ai-tests/
arquivos privados
logs com dados reais
documentos/certificados reais de clientes
segredos reais
```

### Regra de leitura entre repositorios

Depois da separacao:

1. Toda tarefa inicia em `portal-sama-docs`.
2. A IA deve ler `INSTRUÇÕEDS_AI.MD` e os documentos relacionados.
3. Somente depois disso deve abrir `portal-sama-api` ou `portal-sama-web`.
4. Ao alterar API ou Web, atualizar a documentacao correspondente em `portal-sama-docs`.
5. O commit de documentacao deve acompanhar ou anteceder o commit tecnico.
6. Quando a mudanca afetar deploy/producao, atualizar este documento e `docs/EASYPANEL_DEPLOY.md`.

Resolvido operacionalmente em 2026-05-25 11:09 -03:00: `portal-sama-api/README.md` foi criado e `portal-sama-web/README.md` foi reforcado com a regra minima de leitura do workspace irmao `portal-sama-docs`, incluindo os documentos obrigatorios antes de alterar API/Web.

## Configuracao necessaria no EasyPanel

### Servicos minimos

```txt
portal-sama-web
portal-sama-api
portal-sama-database
```

O banco ja informado no EasyPanel e:

```txt
Host interno observado: portal-sama-database
Porta: 3306
Banco: banco-sama
Versao vista no log EasyPanel: MySQL Community Server 9.7.0
```

### Variaveis essenciais da API

Configurar no EasyPanel, nunca em arquivo versionado:

```env
NODE_ENV=production
PORT=3000
API_PREFIX=api-v2
FRONTEND_URL=https://portal.samacontabil.com.br
CORS_ORIGIN=https://portal.samacontabil.com.br
DATABASE_URL=mysql://portal_user:SENHA_FORTE@HOST_INTERNO_MYSQL:3306/banco-sama
PRISMA_CONNECT_ON_BOOT=true
PRISMA_MIGRATE_ON_START=false

JWT_ACCESS_SECRET=gerar_chave_forte
JWT_REFRESH_SECRET=gerar_outra_chave_forte
CSRF_SECRET=gerar_chave_csrf_forte

COOKIE_NAME=sama_refresh_token
COOKIE_DOMAIN=.samacontabil.com.br
COOKIE_SECURE=true
COOKIE_SAME_SITE=lax

STORAGE_PRIVATE_PATH=/var/private/portal-sama
CERTIFICATE_STORAGE_PATH=/var/private/portal-sama/certificates
CONTRACT_PDF_STORAGE_PATH=/var/private/portal-sama/contracts

SAMA_UPLOAD_SCAN_MODE=strict
SAMA_UPLOAD_SCAN_TIMEOUT_SEC=45
SAMA_UPLOAD_SCAN_BIN=/usr/bin/clamscan
SAMA_UPLOAD_QUARANTINE_DIR=/var/private/portal-sama/uploads/_quarantine
SAMA_CLAMAV_UPDATE_ON_START=false
SAMA_BACKUP_DIR=/var/private/portal-sama/_ops-backups
SAMA_MYSQL_DUMP_BIN=
SAMA_MYSQL_CLIENT_BIN=
SAMA_RESTORE_BACKUP_DIR=
SAMA_RESTORE_TARGET_DATABASE_URL=mysql://portal_restore:SENHA_FORTE@portal-sama-database:3306/banco_sama_restore_drill
SAMA_RESTORE_TARGET_STORAGE_PATH=/tmp/portal-sama-restore-storage
SAMA_RESTORE_DRILL_CONFIRM=

WEB_PUSH_PUBLIC_KEY=
WEB_PUSH_PRIVATE_KEY=
WEB_PUSH_SUBJECT=mailto:suporte@samacontabil.com.br
```

Substituir `HOST_INTERNO_MYSQL` pelo host real exibido no EasyPanel/phpMyAdmin/log. Exemplo atualmente observado: `portal-sama-database`. Exemplos historicos ja observados em logs antigos: `portal-sama_database` e `portal-sama_portal-sama_database`.

Observacao 2026-05-22 16:08 -03:00: o erro Prisma `P1012` com `Environment variable not found: DATABASE_URL` indica que a variavel nao chegou ao build/runtime da API. O Dockerfile de `portal-sama-api` agora usa uma URL dummy somente para `prisma generate` no build; a `DATABASE_URL` real continua obrigatoria nas variaveis do servico `portal-sama-api` no EasyPanel antes de `prisma migrate deploy` e start da API.

Observacao 2026-05-22 16:38 -03:00: o erro Prisma `P3005` com `The database schema is not empty` indica banco existente sem historico `_prisma_migrations`. O start da API nao roda mais migrations por padrao (`PRISMA_MIGRATE_ON_START=false`), para o container ficar online. A preparacao do banco existente deve ser feita uma unica vez com backup e `SAMA_PRISMA_BASELINE_EXISTING_DATABASE=true npm run prisma:migrate:baseline-existing`.

### Usuario MySQL da aplicacao

Nao usar `root` na API.

Criar usuario proprio:

```sql
CREATE USER 'portal_user'@'%' IDENTIFIED BY 'SENHA_FORTE';
GRANT ALL PRIVILEGES ON `banco-sama`.* TO 'portal_user'@'%';
FLUSH PRIVILEGES;
```

### Comandos iniciais no container da API

Depois de publicar a API e configurar o `DATABASE_URL` real em banco limpo:

```bash
npx prisma migrate deploy
npm run prisma:seed
SAMA_BOOTSTRAP_ADMIN_USERNAME=admin \
SAMA_BOOTSTRAP_ADMIN_PASSWORD='senha_forte_temporaria' \
SAMA_BOOTSTRAP_ADMIN_NAME='Administrador Portal Sama' \
SAMA_BOOTSTRAP_ADMIN_ROLE=DEV \
npm run prisma:bootstrap-admin
```

Se o banco existente retornar `P3005`, usar o baseline controlado antes do seed/bootstrap:

```bash
SAMA_PRISMA_BASELINE_EXISTING_DATABASE=true npm run prisma:migrate:baseline-existing
```

Observacao 2026-05-25 09:25 -03:00: se o container atual ainda falhar em `npm run prisma:seed` com `Cannot find module '../src/modules/rbac/default-rbac'`, ele esta usando uma imagem anterior ao wrapper de runtime. Workaround imediato: `node dist/prisma/seed.js`. Correcao definitiva: redeployar `portal-sama-api` com o script atualizado.

Observacao 2026-05-25 09:48 -03:00, atualizada em 2026-05-25 11:46 -03:00: se o deploy falhar com Prisma `P1013` e `invalid port number in database URL`, o problema e o valor de `DATABASE_URL`, nao as migrations. Usar o formato `mysql://portal_user:SENHA_URL_ENCODED@portal-sama-database:3306/banco-sama`, sem porta duplicada e com caracteres especiais da senha codificados por `encodeURIComponent`.

Atualizacao operacional 2026-05-25 11:46 -03:00: o banco do EasyPanel ja esta conectado a API, usuarios iniciais da aplicacao ja foram criados pelo operador e o health publico retornou `database=up`. Nao registrar nomes, e-mails ou senhas reais nesta documentacao.

Depois trocar/gerenciar a senha por fluxo seguro e manter usuarios reais auditados.

## Ordem segura para homologacao

1. [x] Criar o repositorio de documentacao no Bitbucket (`portal-sama-doc` observado; alias local `portal-sama-docs`).
2. [x] Subir documentacao inicial para o repositorio de documentacao.
3. [x] Criar o repositorio `portal-sama-api` no Bitbucket.
4. [x] Subir `portal-sama-api` para o Bitbucket.
5. [x] Criar o repositorio `portal-sama-web` no Bitbucket.
6. [x] Subir `portal-sama-web` para o Bitbucket.
7. [x] Conferir que os tres repositorios apontam para a mesma decisao/documentacao vigente.
8. [x] Criar servico API no EasyPanel apontando para o repo da API.
9. [x] Criar servico Web no EasyPanel apontando para o repo do Web.
10. [x] Confirmar que `portal-sama-web` resolve/proxy `/api-v2` para `portal-sama-api`.
11. [x] Configurar `DATABASE_URL` para o banco real do EasyPanel.
12. Criar usuario MySQL nao-root.
13. [x] Disponibilizar comando operacional de backup no container da API (`npm run ops:backup:create`).
14. [x] Disponibilizar verificador operacional dos artefatos de backup (`npm run ops:backup:verify`).
14.1. [x] Disponibilizar restore drill operacional protegido (`npm run ops:restore:drill`) e plano documentado em `PLANO_ROLLBACK_RESTORE_DRILL.md`.
15. Rodar `npx prisma migrate deploy` em banco limpo ou `SAMA_PRISMA_BASELINE_EXISTING_DATABASE=true npm run prisma:migrate:baseline-existing` em banco existente com `P3005`.
16. Rodar `npm run prisma:seed`.
17. Rodar `npm run prisma:bootstrap-admin`.
18. [x] Rodar `npm run ops:readiness` no container da API sem `--skip` (passou com warning apenas de backup/rollback).
19. Rodar `npm run ops:backfill:report` no container da API e anexar JSON/evidencia ao plano de backfill.
20. [x] Validar `GET /api-v2/health` publicamente (`database=up`, `storage=up`).
21. [x] Validar `GET /api-v2/auth/csrf` publicamente com CORS/cookie CSRF.
22. [x] Validar acesso inicial ao frontend em HTTPS e criacao de usuarios iniciais.
23. Configurar volume privado e permissoes de storage.
24. [x] Ativar e validar ClamAV em modo `strict` com EICAR.
25. Executar backfills por modulo.
26. Criar/migrar usuarios reais e roles.
27. [x] Rodar smoke publico sem `--soft`.
28. [x] Implementar e testar localmente aliases publicos de documentos de onboarding na API/Web.
29. [x] Disponibilizar smoke operacional de autenticacao real (`npm.cmd run smoke:auth`).
30. [x] Rodar `npm.cmd run smoke:auth` com usuario real bootstrap/DEV.
31. [x] Disponibilizar suite Playwright opt-in contra homologacao real (`npm.cmd run test:e2e:real`).
32. [x] Disponibilizar smoke operacional de permissoes por perfil (`npm.cmd run smoke:permissions`).
33. [x] Disponibilizar bateria consolidada de homologacao real (`npm.cmd run homologation:real`).
34. [x] Rodar Playwright contra homologacao real com usuario real bootstrap/DEV.
35. [x] Rodar matriz minima `smoke:permissions` com usuario real bootstrap/DEV.
36. Rodar `npm.cmd run smoke:permissions` com usuarios reais por todos os perfis oficiais.
37. Fazer QA visual desktop/mobile.
38. Rodar `npm run ops:backup:create`, `npm run ops:backup:verify` e `npm run ops:restore:drill` no EasyPanel, copiar artefatos para fora do container e validar rollback/restore drill real.
39. Aprovar corte para producao.

## Checklist do que falta para producao

### Infraestrutura

- [x] Separar workspace local `portal-sama-api`.
- [x] Separar workspace local `portal-sama-web`.
- [x] Separar workspace local `portal-sama-docs`.
- [x] Criar `.gitignore` minimo em `portal-sama-api`.
- [x] Reforcar `.gitignore` em `portal-sama-web`.
- [x] Criar `.gitignore` e `README.md` inicial em `portal-sama-docs`.
- [x] Criar `.gitattributes` nos tres workspaces para normalizar quebras de linha.
- [x] Inicializar Git em `portal-sama-docs`.
- [x] Inicializar Git em `portal-sama-api`.
- [x] Inicializar Git em `portal-sama-web`.
- [x] Criar repo Bitbucket `portal-sama-api`.
- [x] Criar repo Bitbucket `portal-sama-web`.
- [x] Criar repo Bitbucket de documentacao (`portal-sama-doc` observado; alias local `portal-sama-docs`).
- [x] Subir documentacao inicial para o repo de documentacao.
- [x] Definir regra operacional para toda IA ler `portal-sama-docs` antes de API/Web.
- [x] Configurar builds do EasyPanel a partir dos repos.
- [x] Configurar dominio HTTPS.
- [x] Confirmar proxy `/api-v2`.
- [x] Confirmar CORS e cookie CSRF seguro em HTTPS.
- [x] Configurar logs basicos no EasyPanel.
- [x] Disponibilizar backup operacional da API (`npm run ops:backup:create`).
- [x] Disponibilizar verificador de artefatos de backup (`npm run ops:backup:verify`).
- [x] Disponibilizar restore drill operacional protegido (`npm run ops:restore:drill`) e plano de rollback.
- [ ] Rodar backup real no EasyPanel, verificar artefatos, copiar para fora do container e validar retencao.
- [x] Definir plano de rollback.

### Banco e dados

- [ ] Criar usuario MySQL nao-root.
- [x] Configurar e validar `DATABASE_URL` real no servico `portal-sama-api` do EasyPanel.
- [ ] Rodar/registrar `prisma migrate deploy` em banco limpo ou baseline controlado para `P3005` no MySQL 9.7.0 observado no EasyPanel.
- [ ] Rodar seed RBAC.
- [ ] Rodar/registrar bootstrap admin ou procedimento equivalente usado para usuarios iniciais.
- [ ] Comparar schema Prisma com schema legado.
- [x] Disponibilizar relatorio read-only de backfill/consistencia (`npm run ops:backfill:report`).
- [ ] Rodar relatorio read-only de backfill no MySQL real e anexar evidencia.
- [ ] Planejar backfills por modulo com base no relatorio real.
- [ ] Executar backfills com logs e conferencia.
- [ ] Validar contagens de registros.
- [ ] Validar vinculos cliente/usuario/gestor/departamento/carteira.

### Seguranca

- [ ] Validar login, refresh e logout em HTTPS.
- [x] Validar CSRF publico em HTTPS.
- [ ] Validar cookies `HttpOnly`, `Secure`, `SameSite`.
- [x] Disponibilizar smoke para validar login/refresh/logout e politica de refresh cookie em HTTPS.
- [x] Disponibilizar smoke para validar matriz 401/403/200 por perfil.
- [ ] Validar 401/403/200 por perfil.
- [ ] Validar IDOR em documentos, certificados, propostas e contratos.
- [ ] Validar storage privado sem acesso publico.
- [x] Disponibilizar validacao operacional de storage/ClamAV/RBAC (`npm run ops:readiness`).
- [x] Validar `npm run ops:readiness` no `portal-sama-api` sem `--skip`, com ClamAV/EICAR strict passando.
- [ ] Proteger phpMyAdmin.
- [ ] Confirmar Swagger desabilitado/protegido em producao.

### Aplicacao

- [x] Validar acesso inicial ao frontend publico em HTTPS.
- [ ] Validar login/Home com usuario real por perfil.
- [ ] Validar usuarios, roles e permissoes reais por matriz.
- [ ] Validar clientes e colaboradores reais.
- [ ] Validar documentos com upload/download reais.
- [ ] Validar certificados digitais com storage real.
- [ ] Validar propostas e contratos.
- [ ] Validar assinatura publica.
- [x] Conectar pagina publica de documentos de onboarding aos aliases documentados da API v2.
- [ ] Validar solicitacoes de acesso.
- [ ] Validar notificacoes/Web Push se forem liberadas.
- [ ] Validar gestor/calendario/transferencias.
- [ ] Migrar ou bloquear corretamente mutacoes ainda legadas do Integra-AI.

### Testes e QA

- [x] Rodar lint/build da API no repo separado.
- [x] Rodar lint/build do Web no repo separado.
- [x] Rodar E2E do Web no repo separado.
- [x] Disponibilizar smoke publico automatizado no repo separado do Web.
- [x] Implementar health operacional da API para banco/storage.
- [x] Rodar smoke publico sem `--soft`.
- [x] Rodar smoke local da pagina publica de documentos usando alias `/api-v2/public/onboarding/documents/:token`.
- [x] Disponibilizar smoke operacional de autenticacao real (`npm.cmd run smoke:auth`).
- [x] Disponibilizar smoke operacional de permissoes por perfil (`npm.cmd run smoke:permissions`).
- [x] Disponibilizar bateria consolidada de homologacao real (`npm.cmd run homologation:real`).
- [x] Rodar smoke/auth, matriz minima DEV/bootstrap e Playwright real contra HTTPS com usuario real.
- [x] Disponibilizar validadores operacionais da API para RBAC, storage, ClamAV, usuarios privilegiados e backfill read-only.
- [x] Disponibilizar gerador de backup operacional da API (`npm run ops:backup:create`).
- [x] Disponibilizar verificador de artefatos de backup da API (`npm run ops:backup:verify`).
- [x] Disponibilizar restore drill operacional da API (`npm run ops:restore:drill`).
- [x] Criar Playwright contra homologacao real (`npm.cmd run test:e2e:real`).
- [ ] Rodar Playwright contra homologacao real com usuario real.
- [ ] Testar fluxos por perfil real.
- [ ] Fazer QA visual desktop.
- [ ] Fazer QA visual mobile.
- [ ] Registrar evidencias em `.ai-tests/` e resumos em docs.

## Bloqueios atuais principais

1. Atualizacoes tecnicas e documentais desta rodada precisam ser enviadas aos remotos.
2. Confirmar historico de migrations/seed/bootstrap no banco real e registrar o procedimento usado.
3. Rodar `npm run ops:backfill:report -- --json` no container `portal-sama-api` e usar o resultado para planejar/executar backfills reais por modulo.
4. Rodar `npm run ops:backup:create`, `npm run ops:backup:verify` e `npm run ops:restore:drill` no container `portal-sama-api`, copiar os artefatos para fora do container e documentar restore drill real.
5. Storage privado retornou `storage=up` e readiness real confirmou escrita em `/var/private/portal-sama`; aliases publicos de onboarding ja estao implementados, mas persistencia, permissoes, backup externo e fluxos reais de upload/download ainda precisam ser homologados.
6. O smoke operacional de autenticacao passou com usuario bootstrap/DEV; ainda precisa ser repetido por perfis criticos quando as credenciais forem disponibilizadas.
7. Smoke de permissoes passou na matriz minima bootstrap/DEV; usuarios/permissoes reais ainda precisam ser validados por matriz oficial de roles/perfis no EasyPanel.
8. Suite Playwright real passou com usuario bootstrap/DEV, mas precisa ser ampliada por perfis/fluxos e o QA visual final ainda nao foi concluido.
9. Backup operacional foi instrumentalizado com geracao, verificacao e restore drill protegido, mas ainda precisa ser executado no EasyPanel, armazenado fora do container e testado em alvo isolado real; retencao de logs segue pendente.
10. Dependencias funcionais remanescentes do legado ainda precisam ser desligadas com seguranca.

## Criterio para dizer "pronto para producao"

So marcar como pronto quando todos estes pontos forem verdadeiros:

- API e Web publicados no EasyPanel a partir do Bitbucket.
- Documentacao publicada e atualizada em `portal-sama-docs`.
- Banco real com migrations aplicadas e backup validado.
- Seed RBAC e bootstrap admin executados.
- Usuarios reais criados/migrados.
- Backfills conferidos.
- Storage privado validado.
- ClamAV strict validado.
- HTTPS, CORS, CSRF e cookies seguros funcionando.
- Fluxos criticos testados com dados reais.
- Playwright real passando.
- QA visual aprovado.
- Plano de rollback documentado.
- Dependencias do legado removidas ou formalmente bloqueadas.

## Proximo passo recomendado

Com Bitbucket, EasyPanel, dominio HTTPS, banco real e smoke publico ja funcionando, o proximo passo operacional e transformar a homologacao tecnica em homologacao funcional:

```txt
registrar migrations/seed/bootstrap -> carregar credenciais no shell -> rodar homologation:real -> rodar ops:backfill:report --json -> rodar ops:backup:create -> rodar ops:backup:verify -> rodar ops:restore:drill em alvo isolado -> copiar artefatos/evidencias -> testar fluxos criticos reais -> validar uploads/downloads/storage -> QA visual final -> desligamento seguro do legado
```

Tambem e necessario enviar este commit/documentacao atualizada ao remoto de documentacao para manter Bitbucket e workspace local sincronizados.
