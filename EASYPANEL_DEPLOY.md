# Deploy da nova stack no EasyPanel

## 1. Objetivo

Este documento descreve como organizar o deploy da nova stack do Portal Sama no EasyPanel, considerando:

- frontend React + Vite;
- backend NestJS;
- MySQL 8;
- Redis opcional;
- phpMyAdmin protegido;
- storage privado para documentos, certificados e contratos.

Para acompanhar o percentual de prontidao, bloqueios de producao, separacao Bitbucket e checklist vivo de homologacao, atualizar tambem [`AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`](AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD).

Topologia oficial de repositorios para Bitbucket/EasyPanel:

```txt
portal-sama-docs  -> documentacao obrigatoria e fonte de verdade
portal-sama-api   -> backend NestJS/API v2
portal-sama-web   -> frontend React/Vite
```

O EasyPanel deve consumir os repositorios `portal-sama-api` e `portal-sama-web` para build/deploy. O repositorio `portal-sama-docs` nao vira servico de aplicacao; ele e a fonte obrigatoria de contexto antes de qualquer alteracao tecnica.

Atualizacao 2026-05-25 11:09 -03:00: `portal-sama-api/README.md` e `portal-sama-web/README.md` agora trazem instrucao minima apontando para `portal-sama-docs`, para preservar a regra de leitura obrigatoria mesmo quando alguem abrir primeiro um repo tecnico separado.

Atualizacao 2026-05-25 11:20 -03:00: o repo separado `portal-sama-web` agora expoe `npm.cmd run smoke:public` e versiona `scripts/portal-public-smoke.mjs`; o comando abaixo deve ser executado a partir desse workspace apos publicar Web/API no EasyPanel.

Atualizacao 2026-05-25 11:32 -03:00: `GET /api-v2/health` na API agora faz health operacional: quando `PRISMA_CONNECT_ON_BOOT=true`, executa `SELECT 1` via Prisma; tambem cria/verifica `STORAGE_PRIVATE_PATH` com leitura/escrita e retorna HTTP 503 se banco ou storage falharem.

---

## 2. Serviços recomendados

```txt
portal-sama-web
portal-sama-api
portal-sama-mysql
portal-sama-redis
portal-sama-phpmyadmin
portal-sama-storage
```

| Serviço | Responsabilidade |
|---|---|
| `portal-sama-web` | Servir o build do React/Vite. |
| `portal-sama-api` | Executar API NestJS em `/api-v2`. |
| `portal-sama-mysql` | Banco relacional principal. |
| `portal-sama-redis` | Cache, filas, rate limit e sessões temporárias, se necessário. |
| `portal-sama-phpmyadmin` | Administração controlada do MySQL. |
| Volume privado | Armazenar documentos fora da pasta pública. |

---

## 3. Variáveis de ambiente da API

Configurar no EasyPanel, não em arquivo `.env` versionado.

```env
NODE_ENV=production
PORT=3000
API_PREFIX=api-v2

DATABASE_URL=mysql://portal_user:SENHA_FORTE@portal-sama-mysql:3306/portal_sama
PRISMA_MIGRATE_ON_START=false

JWT_ACCESS_SECRET=gerar_chave_forte_com_mais_de_32_caracteres
JWT_REFRESH_SECRET=gerar_outra_chave_forte_com_mais_de_32_caracteres
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
CSRF_SECRET=gerar_chave_csrf_forte_com_mais_de_32_caracteres
CSRF_COOKIE_NAME=sama_csrf_token
CSRF_HEADER_NAME=x-csrf-token

COOKIE_NAME=sama_refresh_token
COOKIE_DOMAIN=.seudominio.com.br
COOKIE_SECURE=true
COOKIE_SAME_SITE=lax

CORS_ORIGIN=https://portal.seudominio.com.br

STORAGE_PRIVATE_PATH=/var/private/portal-sama
MAX_UPLOAD_SIZE_MB=25
SAMA_UPLOAD_SCAN_MODE=strict
SAMA_UPLOAD_SCAN_TIMEOUT_SEC=45
SAMA_UPLOAD_SCAN_BIN=/usr/bin/clamscan
SAMA_UPLOAD_SCAN_ARGS=
SAMA_UPLOAD_QUARANTINE_DIR=/var/private/portal-sama/uploads/_quarantine

REDIS_URL=redis://portal-sama-redis:6379

LOG_LEVEL=info
SENTRY_DSN=
```

---

## 4. Dockerfile da API NestJS

Arquivo sugerido: `apps/api/Dockerfile`

Estado atual do repositório em 2026-05-08: a fundação foi criada em `portal-sama-api/`, portanto o Dockerfile inicial está em `portal-sama-api/Dockerfile`. Se o projeto for reorganizado para `apps/api`, este documento deve ser ajustado junto com o deploy.

```dockerfile
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:22-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
# Prisma generate valida env("DATABASE_URL"), mas nao conecta no banco.
ARG PRISMA_GENERATE_DATABASE_URL=mysql://portal_user:portal_password@localhost:3306/portal_sama
RUN DATABASE_URL="$PRISMA_GENERATE_DATABASE_URL" npx prisma generate
RUN npm run build

FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
ENV PRISMA_MIGRATE_ON_START=false
COPY package*.json ./
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/scripts ./scripts
COPY --from=builder /app/node_modules/.prisma ./node_modules/.prisma
EXPOSE 3000
CMD ["sh", "-c", "if [ -z \"${DATABASE_URL:-}\" ]; then echo \"DATABASE_URL is required. Configure it in the EasyPanel environment for portal-sama-api.\" >&2; exit 1; fi; if [ \"${PRISMA_MIGRATE_ON_START:-false}\" = \"true\" ]; then npx prisma migrate deploy; else echo \"Skipping Prisma migrations on startup. Run migrations from an EasyPanel one-off command after backup/baseline.\"; fi; exec node dist/src/main.js"]
```

> Atualizacao 2026-05-22 16:08 -03:00: o build da API usa `PRISMA_GENERATE_DATABASE_URL` apenas como URL dummy para `prisma generate`, porque o Prisma valida `env("DATABASE_URL")` mesmo sem conectar no banco durante a geracao do client. A `DATABASE_URL` real continua obrigatoria no ambiente runtime do servico `portal-sama-api` no EasyPanel.
>
> Atualizacao 2026-05-22 16:38 -03:00: migrations nao rodam mais no start por padrao. Em banco existente sem `_prisma_migrations`, `prisma migrate deploy` retorna `P3005`; portanto o container deve subir primeiro e a migration deve ser rodada por comando unico apos backup/baseline.

> Observação: no início, rodar `prisma migrate deploy` no start pode simplificar. Em uma operação mais madura, execute migrations em uma etapa separada de CI/CD.

---

## 5. Dockerfile do frontend React

Arquivo atual: `portal-sama-web/Dockerfile`

```dockerfile
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine AS runner
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Arquivo atual: `portal-sama-web/nginx.conf`

```nginx
map $http_x_forwarded_proto $portal_forwarded_proto {
  default $http_x_forwarded_proto;
  "" $scheme;
}

map $http_x_forwarded_port $portal_forwarded_port {
  default $http_x_forwarded_port;
  "" $server_port;
}

map $http_upgrade $portal_connection_upgrade {
  default upgrade;
  "" close;
}

server {
  listen 80;
  server_name _;

  root /usr/share/nginx/html;
  index index.html;
  client_max_body_size 30m;
  resolver 127.0.0.11 valid=30s ipv6=off;
  set $portal_api_upstream portal-sama-api:3000;

  location = /api-v2 {
    return 308 /api-v2/;
  }

  location = /api-v2/notifications/stream {
    proxy_pass http://$portal_api_upstream$request_uri;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Port $portal_forwarded_port;
    proxy_set_header X-Forwarded-Proto $portal_forwarded_proto;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $portal_connection_upgrade;
    proxy_buffering off;
    proxy_cache off;
    proxy_read_timeout 1h;
    add_header X-Accel-Buffering no always;
  }

  location /api-v2/ {
    proxy_pass http://$portal_api_upstream$request_uri;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Port $portal_forwarded_port;
    proxy_set_header X-Forwarded-Proto $portal_forwarded_proto;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $portal_connection_upgrade;
    proxy_connect_timeout 10s;
    proxy_send_timeout 120s;
    proxy_read_timeout 120s;
  }

  location / {
    try_files $uri $uri/ /index.html;
  }

  location /assets/ {
    expires 30d;
    add_header Cache-Control "public, immutable";
  }
}
```

---

Atualizacao 2026-05-21 08:56 -03:00: o `nginx.conf` do frontend passou a proxyar `/api-v2/` para `portal-sama-api:3000`, usando o DNS interno do Docker (`127.0.0.11`) para resolver o servico. Isso permite usar apenas `portal.samacontabil.com.br` no EasyPanel quando o dominio publico estiver apontado para o servico `portal-sama-web`, desde que o servico `portal-sama-api` exista na mesma rede interna. O limite `client_max_body_size 30m` acompanha `MAX_UPLOAD_SIZE_MB=25`, e o stream `/api-v2/notifications/stream` fica com buffering desativado para SSE.

---

## 6. Docker Compose de referencia para homologacao

Arquivo versionado: `docker-compose.easypanel.example.yml`.

O compose usa os nomes reais esperados pelo proxy do frontend:

```txt
portal-sama-web
portal-sama-api
portal-sama-mysql
```

Arquivo de variaveis de exemplo: `compose.easypanel.env.example`.

Para validar a topologia sem ler o `.env` local do projeto:

```bash
docker compose --env-file compose.easypanel.env.example -f docker-compose.easypanel.example.yml config --services
docker compose --env-file compose.easypanel.env.example -f docker-compose.easypanel.example.yml config --volumes
```

Saida esperada:

```txt
portal-sama-mysql
portal-sama-api
portal-sama-web

portal_sama_mysql
portal_sama_private_storage
```

> A ordem dos volumes pode variar conforme a versao do Docker Compose.

Observacoes:

- Nao usar os segredos do arquivo de exemplo em producao.
- No EasyPanel, configurar variaveis sensiveis diretamente no painel/secret manager.
- `portal-sama-web` depende de resolver `portal-sama-api` na rede interna para o proxy `/api-v2/`.
- `SAMA_UPLOAD_SCAN_MODE` esta como `best_effort` no exemplo para nao bloquear homologacao sem ClamAV instalado; mudar para `strict` somente quando o scanner estiver disponivel no ambiente.

### 6.1 Homologacao real com dois repositorios e banco existente

Atualizacao 2026-05-22 14:57 -03:00: para homologar sem depender da aplicacao antiga, manter tres repositorios no Bitbucket e dois builds de aplicacao no EasyPanel:

```txt
Repositorio/docs      -> docs completas do projeto, sem build no EasyPanel
Repositorio/API       -> raiz portal-sama-api, Dockerfile portal-sama-api/Dockerfile
Repositorio/frontend  -> raiz portal-sama-web, Dockerfile portal-sama-web/Dockerfile
Banco existente       -> database do EasyPanel com banco banco-sama
```

Nomes internos recomendados dos servicos:

```txt
portal-sama-api
portal-sama-web
portal-sama_database
```

O `nginx.conf` do frontend proxya `/api-v2/` para `portal-sama-api:3000`. Se o EasyPanel gerar outro nome interno para a API, ajustar esse upstream ou configurar path proxy/dominio separado no painel.

Pelo ambiente informado em 2026-05-22, o phpMyAdmin mostra:

```txt
Servidor: portal-sama_database via TCP/IP
Porta: 3306
Banco: banco-sama
Versao: MySQL Community Server 9.6.0
```

O nome interno do host pode variar conforme o projeto/servico criado no EasyPanel. Em 2026-05-25, o log da API tambem mostrou o host `portal-sama_portal-sama_database`. Use exatamente o host exibido no phpMyAdmin/log, sempre com uma unica porta `:3306`.

Logo, a API deve apontar para o banco real, com usuario proprio da aplicacao e sem usar `root`:

```env
DATABASE_URL=mysql://portal_user:SENHA_FORTE@HOST_INTERNO_MYSQL:3306/banco-sama
PRISMA_MIGRATE_ON_START=false
```

Se o log do deploy mostrar `Error code: P1012` e `Environment variable not found: DATABASE_URL`, a correcao operacional e:

1. configurar `DATABASE_URL` nas variaveis de ambiente do servico `portal-sama-api`, nao no servico web nem apenas no banco;
2. confirmar que o host interno e o banco batem com o phpMyAdmin/log (`HOST_INTERNO_MYSQL:3306/banco-sama`, conforme o caso real);
3. fazer novo build/deploy da API usando o Dockerfile atualizado, que nao exige a URL real durante `prisma generate`;
4. depois que o container subir, rodar/validar `npx prisma migrate deploy`, `npm run prisma:seed` e `npm run prisma:bootstrap-admin`.

Se o log mostrar `Error: P3005` e `The database schema is not empty`, a API esta conectando no banco, mas o banco existente ainda nao tem historico Prisma. Para homologacao no EasyPanel:

1. manter `PRISMA_MIGRATE_ON_START=false` no servico `portal-sama-api`;
2. fazer rebuild/redeploy para o container permanecer online;
3. fazer backup/snapshot do banco `banco-sama`;
4. abrir console/one-off command do servico API;
5. confirmar que o host/banco estao corretos no `DATABASE_URL`;
6. rodar o baseline controlado apenas uma vez:

```bash
SAMA_PRISMA_BASELINE_EXISTING_DATABASE=true npm run prisma:migrate:baseline-existing
```

Esse comando marca a migration vazia `20260501000000_baseline_existing_database` como aplicada e depois executa `prisma migrate deploy` para as migrations reais. Se o banco ja tiver tabelas Prisma parcialmente criadas por tentativa anterior, parar e revisar antes de forcar qualquer resolve adicional.

Se o log mostrar `P1013` com `invalid port number in database URL`, o valor de `DATABASE_URL` esta malformado. Conferir no EasyPanel:

```env
DATABASE_URL=mysql://portal_user:SENHA_URL_ENCODED@portal-sama_portal-sama_database:3306/banco-sama
```

Regras:

1. nao colocar duas portas, como `:3306:3306`;
2. nao colocar espacos, aspas ou `DATABASE_URL=` dentro do valor do campo se o painel ja separa chave/valor;
3. codificar caracteres especiais da senha (`@`, `#`, `/`, `?`, `:`, `&`, `=`, `+`, `%`) com URL encode.

Para codificar somente a senha:

```bash
node -e "console.log(encodeURIComponent(process.argv[1]))" "SUA_SENHA_AQUI"
```

Depois substituir apenas a parte da senha na URL. Exemplo: se a senha codificada saiu `abc%23123`, usar `portal_user:abc%23123@...`.

Se o usuario ainda nao existir, criar pelo phpMyAdmin/SQL do EasyPanel:

```sql
CREATE USER 'portal_user'@'%' IDENTIFIED BY 'SENHA_FORTE';
GRANT ALL PRIVILEGES ON `banco-sama`.* TO 'portal_user'@'%';
FLUSH PRIVILEGES;
```

Observacoes importantes:

- O nome `banco-sama` contem hifen; em SQL manual usar crase, como em `` `banco-sama` ``. Na `DATABASE_URL`, o path `/banco-sama` e suficiente.
- A documentacao original mirava MySQL 8/8.4; MySQL 9.6.0 precisa ser validado com `npx prisma migrate deploy`, SQL modes e collation antes de declarar homologacao aprovada.
- Nao criar outro MySQL se o banco do EasyPanel ja sera usado. O compose versionado continua sendo referencia local/reproduzivel.

Primeira preparacao da API em um banco limpo:

```bash
npx prisma migrate deploy
npm run prisma:seed
SAMA_BOOTSTRAP_ADMIN_USERNAME=admin \
SAMA_BOOTSTRAP_ADMIN_PASSWORD='trocar_por_senha_forte' \
SAMA_BOOTSTRAP_ADMIN_NAME='Administrador Portal Sama' \
SAMA_BOOTSTRAP_ADMIN_ROLE=DEV \
npm run prisma:bootstrap-admin
```

> Atualizacao 2026-05-25 09:25 -03:00: em imagens antigas, `npm run prisma:seed` pode falhar com `Cannot find module '../src/modules/rbac/default-rbac'`, porque o runtime Docker nao copia `src/`. Ate redeployar a API com a imagem corrigida, rode temporariamente `node dist/prisma/seed.js`. Apos o redeploy, `npm run prisma:seed` volta a ser o comando recomendado.

Primeira preparacao da API em um banco existente sem `_prisma_migrations`:

```bash
SAMA_PRISMA_BASELINE_EXISTING_DATABASE=true npm run prisma:migrate:baseline-existing
npm run prisma:seed
SAMA_BOOTSTRAP_ADMIN_USERNAME=admin \
SAMA_BOOTSTRAP_ADMIN_PASSWORD='trocar_por_senha_forte' \
SAMA_BOOTSTRAP_ADMIN_NAME='Administrador Portal Sama' \
SAMA_BOOTSTRAP_ADMIN_ROLE=DEV \
npm run prisma:bootstrap-admin
```

O comando `prisma:bootstrap-admin` cria ou atualiza o primeiro usuario administrativo da API v2, vincula a role `DEV` ou `ADMIN`, registra auditoria e nao imprime a senha. Se o usuario ja existir, a senha e preservada por padrao; redefinir somente com `SAMA_BOOTSTRAP_ADMIN_RESET_PASSWORD=true`.

Checks minimos apos publicar:

```txt
GET https://portal.samacontabil.com.br/api-v2/health
GET https://portal.samacontabil.com.br/api-v2/auth/csrf
Login com o usuario criado pelo bootstrap
npm.cmd run smoke:public
```

Esta preparacao remove a dependencia da aplicacao antiga para o primeiro login da nova stack. Ainda nao remove 100% do legado funcional enquanto backfills, storage/ClamAV real, usuarios/permissoes reais, Integra-AI mutacoes e QA HTTPS/Playwright real nao forem homologados.

---

## 7. Domínios recomendados

Opção 1:

```txt
portal.seudominio.com.br      -> frontend React
api.portal.seudominio.com.br  -> API NestJS
```

Opção 2, geralmente mais simples para cookies e CORS:

```txt
portal.seudominio.com.br        -> frontend React
portal.seudominio.com.br/api-v2 -> API NestJS via proxy
```

### 7.1 Caso real: `portal.samacontabil.com.br`

Diagnostico registrado em 2026-05-13:

- DNS respondeu `portal.samacontabil.com.br` como CNAME para o host do EasyPanel e A `191.252.218.31`.
- A requisicao HTTPS ao subdominio retornou HTTP 500, portanto o problema nao parece ser ausencia de apontamento DNS.
- A tela do EasyPanel mostra o dominio customizado encaminhando para `https://portal-sama_portal-sama:80/`. Para um servico interno ouvindo HTTP na porta 80, o alvo deve ser `http://portal-sama_portal-sama:80/`.
- Nao usar HTTPS no destino interno se quem termina TLS e o proxy do EasyPanel/Traefik. HTTPS fica no dominio publico; entre proxy e container, normalmente e HTTP.

Configuracao recomendada para a nova versao:

```txt
portal.samacontabil.com.br        -> http://portal-sama-web:80
portal.samacontabil.com.br/api-v2 -> http://portal-sama-api:3000
```

Se o EasyPanel nao permitir path proxy no mesmo dominio, usar:

```txt
portal.samacontabil.com.br      -> http://portal-sama-web:80
api.portal.samacontabil.com.br  -> http://portal-sama-api:3000
```

Variaveis da API para este dominio:

```env
FRONTEND_URL=https://portal.samacontabil.com.br
CORS_ORIGIN=https://portal.samacontabil.com.br
COOKIE_DOMAIN=.samacontabil.com.br
COOKIE_SECURE=true
COOKIE_SAME_SITE=lax
```

Checklist especifico do subdominio:

- [ ] Corrigir o alvo interno do dominio customizado para `http://...:80` quando o container for HTTP.
- [ ] Validar se o container da aplicacao antiga responde internamente em HTTP e na porta esperada.
- [ ] Validar logs do container/proxy no momento do HTTP 500.
- [ ] Conferir se o servico web da nova stack escuta `0.0.0.0` e a porta exposta no EasyPanel.
- [ ] Validar `https://portal.samacontabil.com.br/api-v2/health` quando a API nova estiver publicada.

Detalhes do incidente estao em [`INCIDENTE_SUBDOMINIO_PORTAL_SAMACONTABIL.md`](INCIDENTE_SUBDOMINIO_PORTAL_SAMACONTABIL.md).

---

## 8. Segurança no EasyPanel

### MySQL

- Não expor porta pública sem necessidade.
- Usar usuário específico da aplicação.
- Não usar root na aplicação.
- Usar senha forte.
- Fazer backup automático.

### phpMyAdmin

- Proteger com senha forte.
- Restringir acesso por IP, se possível.
- Evitar exposição pública aberta.
- Não usar como fonte principal de alteração de schema em produção.

### API

- Usar HTTPS.
- Configurar `COOKIE_SECURE=true` em produção.
- Restringir CORS.
- Não publicar Swagger aberto em produção.
- Definir limite de upload.
- Usar volume privado para arquivos.
- Não versionar `.env`.

### Storage

- Nunca salvar documentos em pasta pública.
- Usar volume privado.
- Download sempre via backend.
- Auditar upload e download.

---

## 9. Healthcheck

Criar rota:

```txt
GET /api-v2/health
```

Estado atual: rota criada em `portal-sama-api/src/modules/health/health.controller.ts` e validada por unit/e2e. A resposta informa banco como `up`, `down` ou `not_checked` e storage como `up`, `down` ou `not_configured`. Em falha critica, a API responde HTTP 503 com o corpo de status. A validacao com MySQL/storage reais no EasyPanel segue pendente.

Resposta esperada:

```json
{
  "ok": true,
  "service": "portal-sama-api",
  "database": "up",
  "storage": "up",
  "timestamp": "2026-05-07T00:00:00.000Z"
}
```

Em testes/diagnostico com `PRISMA_CONNECT_ON_BOOT=false`, a resposta esperada pode trazer:

```json
{
  "ok": true,
  "service": "portal-sama-api",
  "database": "not_checked",
  "storage": "up",
  "timestamp": "2026-05-07T00:00:00.000Z"
}
```

### 9.1 Smoke publico automatizado

Para homologar o dominio publico depois de publicar `portal-sama-web` e `portal-sama-api`, execute na raiz do repo separado `portal-sama-web`:

```bash
npm.cmd run smoke:public
```

O script `portal-sama-web/scripts/portal-public-smoke.mjs` valida:

- `GET https://portal.samacontabil.com.br`;
- `GET https://portal.samacontabil.com.br/api-v2/health`;
- preflight CORS para `/api-v2/auth/me`;
- emissao de cookie CSRF em `/api-v2/auth/csrf`.

Variaveis suportadas:

```env
PORTAL_PUBLIC_URL=https://portal.samacontabil.com.br
PORTAL_API_BASE_URL=https://portal.samacontabil.com.br/api-v2
PORTAL_CORS_ORIGIN=https://portal.samacontabil.com.br
PORTAL_SMOKE_TIMEOUT_MS=10000
```

Se o EasyPanel usar dominio separado para API, rode:

```bash
npm.cmd run smoke:public -- --api-url https://api.portal.samacontabil.com.br
```

Durante diagnostico, `--soft` coleta evidencias sem falhar o processo; para homologacao final, rodar sem `--soft`.

---

## 10. Checklist de deploy

- [ ] Criar serviço MySQL 8 no EasyPanel ou usar o database existente do EasyPanel.
- [ ] Criar banco `portal_sama` ou apontar para o banco real `banco-sama`.
- [ ] Criar usuário não-root da aplicação.
- [ ] Validar `docker-compose.easypanel.example.yml` com `compose.easypanel.env.example` ou configurar serviços equivalentes manualmente no EasyPanel.
- [ ] Criar serviço NestJS.
- [ ] Configurar variáveis de ambiente.
- [ ] Criar volume privado para documentos.
- [ ] Rodar migrations Prisma.
- [ ] Rodar seed RBAC inicial com `npm.cmd run prisma:seed` ou comando equivalente do container, sem criar usuário/senha.
- [ ] Criar primeiro admin da API v2 com `npm run prisma:bootstrap-admin` usando variaveis `SAMA_BOOTSTRAP_ADMIN_*` no ambiente seguro do EasyPanel.
- [ ] Criar serviço React/Vite.
- [ ] Garantir que `portal-sama-web` consiga resolver `portal-sama-api` na rede interna do EasyPanel.
- [ ] Redeployar `portal-sama-web` apos alteracoes no `nginx.conf`.
- [ ] Configurar domínio e HTTPS.
- [ ] Restringir CORS.
- [ ] Proteger phpMyAdmin.
- [ ] Configurar backups.
- [ ] Validar `/api-v2/health`.
- [ ] Rodar `npm.cmd run smoke:public` sem `--soft`.
- [ ] Validar login.
- [ ] Validar permissões.
- [ ] Validar upload/download.
- [ ] Validar auditoria.
