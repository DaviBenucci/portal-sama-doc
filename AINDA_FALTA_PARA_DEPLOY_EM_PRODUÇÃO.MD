# Ainda Falta Para Deploy Em Producao - Portal Sama

Ultima atualizacao: 2026-05-22 15:16 -03:00
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

Estimativa operacional em 2026-05-22 15:16 -03:00:

| Marco | Percentual | Leitura |
|---|---:|---|
| Pronto para iniciar homologacao da nova stack | 74% | Base tecnica existe e os workspaces locais ja foram preparados; falta publicar os repositorios no Bitbucket/EasyPanel e validar com banco/HTTPS reais. |
| Pronto para producao sem depender do legado | 63% | A separacao local reduz risco operacional, mas ainda faltam backfills, storage/ClamAV real, usuarios/permissoes reais, QA e fluxos criticos reais. |
| Falta para producao | 37% | Principalmente infraestrutura real, migracao de dados, validacao operacional e desligamento seguro do legado. |

## Estado atual resumido

| Area | Estado | Observacao |
|---|---|---|
| Backend `portal-sama-api` | Parcial avancado | NestJS/API v2 existe, com Auth, RBAC, auditoria e varios modulos migrados. Falta validar tudo com MySQL real. |
| Frontend `portal-sama-web` | Parcial avancado | React/Vite existe, login e muitas rotas ja foram migradas. Parte dos testes ainda usa mocks. |
| Banco MySQL/Prisma | Parcial | Schema/migrations existem, mas precisam rodar no MySQL real do EasyPanel (`banco-sama`). |
| Primeiro admin da API v2 | Implementado localmente | Existe `npm run prisma:bootstrap-admin`; falta executar no container real. |
| EasyPanel | Documentado, nao homologado | Existe guia e Dockerfiles; falta criar servicos reais a partir do Bitbucket. |
| Workspaces locais | Separado e preparado | Existem pastas irmas `portal-sama-docs`, `portal-sama-api` e `portal-sama-web` no Desktop. |
| Bitbucket/Git | Parcial | Git local inicializado em `main`; falta criar remotes Bitbucket e publicar os repositorios. |
| Storage privado | Parcial | Codigo e variaveis existem; falta volume real persistente e validacao de permissoes. |
| ClamAV | Pendente em homologacao | Deve ser validado em modo `strict` antes de producao. |
| Usuarios/permissoes reais | Pendente | Seed/roles existem; falta migrar/criar usuarios reais e testar perfis. |
| Backfills | Pendente critico | Dados legados precisam ser migrados/conferidos por modulo. |
| Playwright real | Pendente | Ha cobertura com mocks; falta suite contra API/MySQL/HTTPS reais. |
| QA visual | Pendente | Necessario aprovar desktop/mobile em navegador real. |
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

Para o EasyPanel buscar origem pelo Bitbucket e para a documentacao continuar sendo a fonte de verdade, criar tres repositorios oficiais:

```txt
portal-sama-api
portal-sama-web
portal-sama-docs
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

Ponto a validar operacional: se os repositorios `portal-sama-api` e `portal-sama-web` terao uma copia minima de `docs/INSTRUÇÕEDS_AI.MD` apontando para `portal-sama-docs`, para evitar que uma IA comece pelo repositorio errado.

## Configuracao necessaria no EasyPanel

### Servicos minimos

```txt
portal-sama-web
portal-sama-api
portal-sama_database
```

O banco ja informado no EasyPanel e:

```txt
Host interno provavel: portal-sama_database
Porta: 3306
Banco: banco-sama
Versao vista no phpMyAdmin: MySQL Community Server 9.6.0
```

### Variaveis essenciais da API

Configurar no EasyPanel, nunca em arquivo versionado:

```env
NODE_ENV=production
PORT=3000
API_PREFIX=api-v2
FRONTEND_URL=https://portal.samacontabil.com.br
CORS_ORIGIN=https://portal.samacontabil.com.br
DATABASE_URL=mysql://portal_user:SENHA_FORTE@portal-sama_database:3306/banco-sama
PRISMA_CONNECT_ON_BOOT=true

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

### Usuario MySQL da aplicacao

Nao usar `root` na API.

Criar usuario proprio:

```sql
CREATE USER 'portal_user'@'%' IDENTIFIED BY 'SENHA_FORTE';
GRANT ALL PRIVILEGES ON `banco-sama`.* TO 'portal_user'@'%';
FLUSH PRIVILEGES;
```

### Comandos iniciais no container da API

Depois de publicar a API e configurar o `DATABASE_URL` real:

```bash
npx prisma migrate deploy
npm run prisma:seed
SAMA_BOOTSTRAP_ADMIN_USERNAME=admin \
SAMA_BOOTSTRAP_ADMIN_PASSWORD='senha_forte_temporaria' \
SAMA_BOOTSTRAP_ADMIN_NAME='Administrador Portal Sama' \
SAMA_BOOTSTRAP_ADMIN_ROLE=DEV \
npm run prisma:bootstrap-admin
```

Depois trocar/gerenciar a senha por fluxo seguro e criar usuarios reais.

## Ordem segura para homologacao

1. Criar o repositorio `portal-sama-docs` no Bitbucket.
2. Subir todo o conteudo de `portal-sama-docs` para o repositorio homonimo.
3. Criar o repositorio `portal-sama-api` no Bitbucket.
4. Subir `portal-sama-api` para o Bitbucket.
5. Criar o repositorio `portal-sama-web` no Bitbucket.
6. Subir `portal-sama-web` para o Bitbucket.
7. Conferir que os tres repositorios apontam para a mesma decisao/documentacao vigente.
8. Criar servico API no EasyPanel apontando para o repo da API.
9. Criar servico Web no EasyPanel apontando para o repo do Web.
10. Confirmar que `portal-sama-web` resolve `portal-sama-api:3000` na rede interna.
11. Configurar `DATABASE_URL` para `banco-sama`.
12. Criar usuario MySQL nao-root.
13. Fazer backup/snapshot do banco antes de migrations.
14. Rodar `npx prisma migrate deploy`.
15. Rodar `npm run prisma:seed`.
16. Rodar `npm run prisma:bootstrap-admin`.
17. Validar `GET /api-v2/health`.
18. Validar `GET /api-v2/auth/csrf`.
19. Validar login real no frontend.
20. Configurar volume privado e permissoes de storage.
21. Ativar e validar ClamAV em modo `strict`.
22. Executar backfills por modulo.
23. Criar/migrar usuarios reais e roles.
24. Rodar smoke publico sem `--soft`.
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
- [ ] Criar repo Bitbucket `portal-sama-api`.
- [ ] Criar repo Bitbucket `portal-sama-web`.
- [ ] Criar repo Bitbucket `portal-sama-docs`.
- [ ] Subir toda a documentacao para `portal-sama-docs`.
- [ ] Definir regra operacional para toda IA ler `portal-sama-docs` antes de API/Web.
- [ ] Configurar builds do EasyPanel a partir dos repos.
- [ ] Configurar dominio HTTPS.
- [ ] Confirmar proxy `/api-v2`.
- [ ] Confirmar CORS e cookies seguros.
- [ ] Configurar logs e retencao.
- [ ] Configurar backup do banco e do storage.
- [ ] Definir plano de rollback.

### Banco e dados

- [ ] Criar usuario MySQL nao-root.
- [ ] Validar `DATABASE_URL` real.
- [ ] Rodar `prisma migrate deploy` no MySQL 9.6.0.
- [ ] Rodar seed RBAC.
- [ ] Rodar bootstrap admin.
- [ ] Comparar schema Prisma com schema legado.
- [ ] Planejar backfills por modulo.
- [ ] Executar backfills com logs e conferencia.
- [ ] Validar contagens de registros.
- [ ] Validar vinculos cliente/usuario/gestor/departamento/carteira.

### Seguranca

- [ ] Validar login, refresh e logout em HTTPS.
- [ ] Validar CSRF no navegador real.
- [ ] Validar cookies `HttpOnly`, `Secure`, `SameSite`.
- [ ] Validar 401/403/200 por perfil.
- [ ] Validar IDOR em documentos, certificados, propostas e contratos.
- [ ] Validar storage privado sem acesso publico.
- [ ] Validar ClamAV strict com EICAR.
- [ ] Proteger phpMyAdmin.
- [ ] Confirmar Swagger desabilitado/protegido em producao.

### Aplicacao

- [ ] Validar Home/login com usuario real.
- [ ] Validar usuarios, roles e permissoes reais.
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

- [ ] Rodar lint/build da API no repo separado.
- [ ] Rodar lint/build/E2E do Web no repo separado.
- [ ] Rodar smoke publico sem `--soft`.
- [ ] Criar Playwright contra homologacao real.
- [ ] Testar fluxos por perfil real.
- [ ] Fazer QA visual desktop.
- [ ] Fazer QA visual mobile.
- [ ] Registrar evidencias em `.ai-tests/` e resumos em docs.

## Bloqueios atuais principais

1. Workspaces locais foram separados e inicializados com Git, mas ainda nao possuem remote Bitbucket.
2. Repositorios Bitbucket ainda nao foram criados/publicados (`portal-sama-api`, `portal-sama-web`, `portal-sama-docs`).
3. EasyPanel ainda nao esta rodando API e Web novos a partir desses repos.
4. `prisma migrate deploy` ainda nao foi validado no banco real `banco-sama`.
5. Seed/bootstrap ainda nao foram executados no container real.
6. Backfills reais ainda nao foram executados.
7. Storage privado e ClamAV strict ainda nao foram homologados.
8. Usuarios/permissoes reais ainda nao foram validados.
9. Playwright real e QA visual final ainda nao foram concluidos.
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

O proximo passo operacional e criar os tres repositorios no Bitbucket e adicionar os remotes:

```txt
portal-sama-docs
portal-sama-api
portal-sama-web
```

Depois disso, criar os dois servicos de aplicacao no EasyPanel e executar a primeira homologacao com:

```txt
health -> csrf -> migrate -> seed -> bootstrap-admin -> login real -> smoke publico
```

Somente depois dessa base estar estavel faz sentido executar backfills e Playwright contra fluxos reais.
