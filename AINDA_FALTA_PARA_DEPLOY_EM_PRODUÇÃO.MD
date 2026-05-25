# Ainda Falta Para Deploy Em Producao - Portal Sama

Ultima atualizacao: 2026-05-25 11:46 -03:00
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

Estimativa operacional em 2026-05-25 11:46 -03:00:

| Marco | Percentual | Leitura |
|---|---:|---|
| Pronto para iniciar/continuar homologacao da nova stack | 92% | Repositorios Bitbucket existem, API/Web/database estao publicados no EasyPanel, `https://portal.samacontabil.com.br/` esta ativo, o banco real esta conectado, usuarios iniciais foram criados e o smoke publico sem `--soft` passou com health `database=up` e `storage=up`; faltam QA por perfil, Playwright real, backfills e validacoes operacionais profundas. |
| Pronto para producao sem depender do legado | 73% | Deploy publico, banco real, proxy `/api-v2`, CORS e CSRF ja funcionam; ainda faltam backfills, validacao de permissoes por perfil, storage/ClamAV em fluxo real, QA final, backups/rollback e desligamento seguro do legado. |
| Falta para producao | 27% | Principalmente migracao/conferencia de dados, seguranca operacional, QA final, Playwright real, storage/ClamAV, backups e desligamento seguro do legado. |

## Estado atual resumido

| Area | Estado | Observacao |
|---|---|---|
| Backend `portal-sama-api` | Deployado e operacional | NestJS/API v2 esta rodando no EasyPanel; logs mostram `Nest application successfully started`; smoke publico confirmou `/api-v2/health` com `database=up` e `storage=up`. Falta validar fluxos de negocio e permissoes com dados reais. |
| Frontend `portal-sama-web` | Deployado e operacional | React/Vite esta publicado em `https://portal.samacontabil.com.br/`; logs do Web mostram assets e chamadas `/api-v2/*` respondendo 200; smoke publico validou frontend, proxy, CORS e CSRF. |
| Banco MySQL/Prisma | Operacional em homologacao | EasyPanel possui servico `portal-sama-database`; log mostra MySQL Community Server 9.7.0 pronto em `:3306`; health real da API retornou `database=up`. Falta confirmar historico de migrations/backfills e conferir dados por modulo. |
| Primeiro admin/usuarios API v2 | Parcial operacional | Usuario(s) inicial(is) ja foram criados pelo operador e a aplicacao esta conectada ao banco real; falta registrar matriz de usuarios/perfis reais e validar permissoes por papel. |
| EasyPanel | Deployado | Projeto `portal-sama` contem `portal-sama-api`, `portal-sama-database` e `portal-sama-web` ativos. Falta completar validacoes finais, backups, rollback e hardening operacional. |
| Workspaces locais | Separado e preparado | Existem pastas irmas `portal-sama-docs`, `portal-sama-api` e `portal-sama-web` no Desktop; API/Web agora possuem README minimo com regra de leitura do repo de docs. |
| Bitbucket/Git | Remotos existentes | Repositorios Bitbucket existem para API, Web e documentacao (`portal-sama-api`, `portal-sama-web`, `portal-sama-doc` conforme evidencia visual). Falta apenas garantir push das atualizacoes documentais locais desta rodada. |
| Builds locais separados | Validado | Passaram lint/build da API, `prisma:validate` e lint/build do Web nos repos separados. |
| Storage privado | Parcial validado | Health publico retornou `storage=up`, indicando caminho gravavel no container. Falta validar persistencia do volume, permissoes, backup e fluxos reais de upload/download. |
| ClamAV | Pendente em homologacao | Deve ser validado em modo `strict` antes de producao. |
| Usuarios/permissoes reais | Parcial | Usuarios iniciais foram criados no banco real; falta testar a matriz completa de roles/permissoes e fluxos por perfil. |
| Backfills | Pendente critico | Dados legados precisam ser migrados/conferidos por modulo. |
| Playwright real | Parcial | E2E local do Web separado passou com 9 testes Chromium usando mocks; smoke HTTP real passou contra HTTPS/API/MySQL. Falta suite Playwright navegando contra homologacao real. |
| QA visual | Parcial | Aplicacao publica esta acessivel; ainda necessario aprovar desktop/mobile no EasyPanel/navegador real com usuarios e perfis reais. |
| Dependencia do legado | Parcial | Ainda ha pontos como Integra-AI mutacoes/backfills e validacoes reais antes de desligar PHP. |

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
13. Fazer backup/snapshot do banco antes de migrations.
14. Rodar `npx prisma migrate deploy` em banco limpo ou `SAMA_PRISMA_BASELINE_EXISTING_DATABASE=true npm run prisma:migrate:baseline-existing` em banco existente com `P3005`.
15. Rodar `npm run prisma:seed`.
16. Rodar `npm run prisma:bootstrap-admin`.
17. [x] Validar `GET /api-v2/health` publicamente (`database=up`, `storage=up`).
18. [x] Validar `GET /api-v2/auth/csrf` publicamente com CORS/cookie CSRF.
19. [x] Validar acesso inicial ao frontend em HTTPS e criacao de usuarios iniciais.
20. Configurar volume privado e permissoes de storage.
21. Ativar e validar ClamAV em modo `strict`.
22. Executar backfills por modulo.
23. Criar/migrar usuarios reais e roles.
24. [x] Rodar smoke publico sem `--soft`.
25. Rodar Playwright contra homologacao real.
26. Fazer QA visual desktop/mobile.
27. Validar rollback.
28. Aprovar corte para producao.

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
- [ ] Configurar backup do banco e do storage.
- [ ] Definir plano de rollback.

### Banco e dados

- [ ] Criar usuario MySQL nao-root.
- [x] Configurar e validar `DATABASE_URL` real no servico `portal-sama-api` do EasyPanel.
- [ ] Rodar/registrar `prisma migrate deploy` em banco limpo ou baseline controlado para `P3005` no MySQL 9.7.0 observado no EasyPanel.
- [ ] Rodar seed RBAC.
- [ ] Rodar/registrar bootstrap admin ou procedimento equivalente usado para usuarios iniciais.
- [ ] Comparar schema Prisma com schema legado.
- [ ] Planejar backfills por modulo.
- [ ] Executar backfills com logs e conferencia.
- [ ] Validar contagens de registros.
- [ ] Validar vinculos cliente/usuario/gestor/departamento/carteira.

### Seguranca

- [ ] Validar login, refresh e logout em HTTPS.
- [x] Validar CSRF publico em HTTPS.
- [ ] Validar cookies `HttpOnly`, `Secure`, `SameSite`.
- [ ] Validar 401/403/200 por perfil.
- [ ] Validar IDOR em documentos, certificados, propostas e contratos.
- [ ] Validar storage privado sem acesso publico.
- [ ] Validar ClamAV strict com EICAR.
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
- [ ] Criar Playwright contra homologacao real.
- [ ] Testar fluxos por perfil real.
- [ ] Fazer QA visual desktop.
- [ ] Fazer QA visual mobile.
- [ ] Registrar evidencias em `.ai-tests/` e resumos em docs.

## Bloqueios atuais principais

1. Atualizacoes documentais locais desta rodada ainda precisam ser enviadas ao remoto de documentacao.
2. Confirmar historico de migrations/seed/bootstrap no banco real e registrar o procedimento usado.
3. Backfills reais ainda nao foram executados/conferidos por modulo.
4. Storage privado retornou `storage=up`, mas persistencia, permissoes, backup e fluxos reais de upload/download ainda precisam ser homologados.
5. ClamAV strict ainda nao foi homologado com EICAR.
6. Usuarios/permissoes reais ainda precisam ser validados por matriz de roles/perfis.
7. Playwright real e QA visual final ainda nao foram concluidos.
8. Backups, retencao de logs e plano de rollback ainda precisam ser formalizados.
9. Dependencias funcionais remanescentes do legado ainda precisam ser desligadas com seguranca.

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
registrar migrations/seed/bootstrap -> validar matriz de usuarios/perfis -> testar fluxos criticos reais -> validar storage/ClamAV -> configurar backups/rollback -> Playwright real -> QA visual final
```

Tambem e necessario enviar este commit/documentacao atualizada ao remoto de documentacao para manter Bitbucket e workspace local sincronizados.
