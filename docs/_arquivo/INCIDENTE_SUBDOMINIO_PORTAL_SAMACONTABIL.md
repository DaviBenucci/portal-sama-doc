# [CONCLUIDO] Incidente - subdominio `portal.samacontabil.com.br`

Data do registro: 2026-05-13 15:18 -03:00

Status atualizado em 2026-05-25 11:46 -03:00: resolvido para a nova stack. O dominio `https://portal.samacontabil.com.br/` responde HTTP 200, `portal-sama-web` proxya `/api-v2`, `portal-sama-api` responde health com `database=up` e `storage=up`, e `npm.cmd run smoke:public` passou sem `--soft`.

## Contexto

O subdominio `portal.samacontabil.com.br` ja foi apontado no EasyPanel para a aplicacao antiga, mas a aplicacao publica retorna erro e ainda nao deve receber a nova versao ate ficar estavel.

## Evidencias verificadas

- `portal.samacontabil.com.br` resolve para o EasyPanel via CNAME do projeto e IP `191.252.218.31`.
- A chamada HTTPS ao subdominio retornou HTTP 500.
- Na configuracao visual do EasyPanel, o dominio gerado aponta para `http://portal-sama_portal-sama:80/`, enquanto o dominio customizado aparece apontando para `https://portal-sama_portal-sama:80/`.

## Atualizacao 2026-05-21 08:41 -03:00

Foi adicionado o smoke operacional `scripts/portal-public-smoke.mjs`, tambem exposto por `npm run smoke:public`, para validar o dominio publico, o proxy de `/api-v2`, CORS e cookie CSRF em HTTPS real.

Comando executado:

```bash
npm.cmd run smoke:public -- --soft
```

Resultado observado em 2026-05-21 08:41 -03:00:

- `GET https://portal.samacontabil.com.br` retornou HTTP 200.
- `GET https://portal.samacontabil.com.br/api-v2/health` retornou HTTP 404.
- `OPTIONS https://portal.samacontabil.com.br/api-v2/auth/me` retornou HTTP 200, mas sem `Access-Control-Allow-Origin` e sem `Access-Control-Allow-Credentials`.
- `GET https://portal.samacontabil.com.br/api-v2/auth/csrf` retornou HTTP 404.

Interpretacao: o dominio raiz ja responde, mas a API v2 ainda nao esta publicada/proxyada em `/api-v2` nesse dominio. A pendencia principal passa a ser configurar o servico `portal-sama-api` e o roteamento publico para `/api-v2` ou, se o EasyPanel nao suportar proxy por path, validar `api.portal.samacontabil.com.br` com `--api-url`.

## Atualizacao 2026-05-21 08:56 -03:00

O `portal-sama-web/nginx.conf` foi ajustado para proxyar `/api-v2/` diretamente para `http://portal-sama-api:3000/api-v2/` quando o dominio publico estiver apontado para o container web. O ajuste tambem:

- redireciona `/api-v2` para `/api-v2/`;
- preserva `Host`, IP e headers `X-Forwarded-*` recebidos do proxy externo;
- usa o DNS interno do Docker (`127.0.0.11`) para resolver `portal-sama-api`;
- define `client_max_body_size 30m`, compativel com uploads de ate 25 MB configurados na API;
- desativa buffering no endpoint `/api-v2/notifications/stream` para manter SSE.

Atualizacao complementar 2026-05-21 09:11 -03:00: foi criado `docker-compose.easypanel.example.yml` com os servicos `portal-sama-web`, `portal-sama-api` e `portal-sama-mysql`, alem de `compose.easypanel.env.example` para validar a topologia sem ler o `.env` local. O comando `docker compose --env-file compose.easypanel.env.example -f docker-compose.easypanel.example.yml config --services` validou os nomes esperados.

Proximo teste esperado apos redeploy do `portal-sama-web` e publicacao do `portal-sama-api` na mesma rede interna:

```bash
npm.cmd run smoke:public
```

## Hipotese principal

O dominio customizado provavelmente esta usando protocolo interno incorreto. Se o proxy do EasyPanel termina TLS no dominio publico, o container interno que escuta na porta 80 deve ser acessado por HTTP:

```txt
https://portal.samacontabil.com.br -> http://portal-sama_portal-sama:80/
```

O formato `https://portal-sama_portal-sama:80/` tende a causar falha porque tenta negociar HTTPS com um servico que normalmente serve HTTP na porta 80.

## Recomendacao para a nova stack

Publicar a nova versao com separacao clara:

```txt
portal.samacontabil.com.br        -> http://portal-sama-web:80
portal.samacontabil.com.br/api-v2 -> http://portal-sama-api:3000
```

Alternativa caso o EasyPanel nao permita proxy por path:

```txt
portal.samacontabil.com.br     -> http://portal-sama-web:80
api.portal.samacontabil.com.br -> http://portal-sama-api:3000
```

Variaveis esperadas na API:

```env
FRONTEND_URL=https://portal.samacontabil.com.br
CORS_ORIGIN=https://portal.samacontabil.com.br
COOKIE_DOMAIN=.samacontabil.com.br
COOKIE_SECURE=true
COOKIE_SAME_SITE=lax
```

## Pendencias

- Manter monitoramento do dominio e certificado TLS.
- Validar login, refresh, upload e download via HTTPS real por perfil.
- Validar storage persistente, ClamAV strict, backups e rollback.
- Registrar qualquer novo incidente separado se o dominio voltar a responder 5xx.
