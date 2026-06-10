# Auditoria Deploy - Backend, API e Integrações

## Etapas 9, 10 e 11 - API, backend e prontidão técnica

## Stack e execução

| Item | Resultado |
| --- | --- |
| Framework | NestJS 11 |
| ORM | Prisma 6 |
| Banco alvo | MySQL |
| Build | Passou |
| Lint | Passou |
| Prisma validate | Passou |
| Testes unitários | 44 suites, 264 testes passaram |
| Testes focados notificações/Web Push | 2 suites, 28 testes passaram |
| Testes focados RBAC/env | 2 suites, 8 testes passaram |
| E2E API | 134 passaram, 2 falharam |
| Health local | 200 |
| Migrações | Não validadas: banco indisponível |
| Readiness | Falhou em DB; avisos de ClamAV e backup/restore |

## Resultado dos testes API

Passou:

- `npm.cmd run prisma:validate`
- `npm.cmd run lint`
- `npm.cmd run build`
- `npm.cmd test -- --runInBand`

Falhou:

- `npm.cmd run test:e2e`

Falhas:

| Teste | Local | Esperado | Recebido | Diagnóstico |
| --- | --- | --- | --- | --- |
| `POST /api-v2/notifications/push/subscribe` | `test/app.e2e-spec.ts:1848` | 200 | 403 | `CSRF_TOKEN_INVALID` por cookie CSRF não reenviada no ambiente local. |
| `POST /api-v2/notifications/push/test` | `test/app.e2e-spec.ts:1889` | 200 | 403 | Mesma causa. |

Reprodução adicional sem editar código:

- Com `COOKIE_DOMAIN` herdado do `.env`, o cookie foi emitido com `Domain=portal.samacontabil.com.br` e o agente local não o reenviou.
- Com `COOKIE_DOMAIN=''` e `COOKIE_SECURE=false`, `POST /api-v2/notifications/push/test` retornou 200 e `reason: push_disabled`.

Correção recomendada: tornar o E2E imune ao `.env` local. Opções:

- Definir `COOKIE_DOMAIN=''` e `COOKIE_SECURE=false` no `beforeAll` de `test/app.e2e-spec.ts`.
- Usar `.env.test` isolado e `envFilePath` condicional.
- Em testes CSRF, enviar o header `Cookie` explicitamente quando necessário.

## Arquitetura API

Controles globais:

- Prefixo global `api-v2`.
- `helmet`.
- `cookie-parser`.
- CORS com origem configurável e credenciais.
- `ValidationPipe` com whitelist e bloqueio de campos extras.
- Filtro global de erro.
- Interceptor de request id.
- Swagger apenas fora de produção.
- Throttle global.

Autenticação:

- Login e refresh exigem CSRF.
- Refresh token fica em cookie HttpOnly.
- CSRF usa cookie legível + header e assinatura.
- Refresh token é rotacionado e armazenado como hash HMAC.
- Access token é Bearer.

## Módulos críticos

| Módulo | Status | Observação |
| --- | --- | --- |
| Auth/session | Parcialmente aprovado | Boa base; E2E precisa isolar cookie domain. |
| RBAC/permissões | Parcial | Permissões globais funcionam, mas escopo de clientes precisa correção. |
| Clientes | Reprovado | `clients.read` vaza lista/detalhe/dashboard para perfis departamentais. |
| Documentos | Parcialmente aprovado | Escopo departamental mais robusto; scanner externo pendente. |
| Certificados | Parcial | Criptografia AES-GCM existe, mas fallback de segredo deve ser proibido em produção. |
| Contratos/propostas/onboarding | Parcial | Cobertura existe; fluxo real precisa homologação. |
| Notificações/Web Push | Parcial | Implementado; VAPID real e E2E precisam fechar. |
| Integra-AI | Parcial | Testes cobrem motor; homologar arquivos reais/exportação. |
| Acessórias | Parcial | Rate limit, retry, checkpoint e backfill existem; falta validação com API real. |

## Acessórias

Pontos positivos encontrados:

- Serviço de rate limit expõe status com concorrência, fila, tentativas e pausa.
- `Retry-After` é respeitado e há fallback de 65s.
- Backoff escala para 90s, 120s e 180s.
- Incremental antigo gera `backfill_required` e notifica DEV.
- Persistência local separa catálogo operacional de entregas.

Pendências:

- Validar `ListAll` com token real.
- Validar paginação, 429, 401/403, timeout e payload parcial.
- Confirmar que nenhum token Acessórias é exposto no frontend.
- Rodar backfill controlado antes de incremental em ambiente novo.

## Banco, migrações e readiness

Resultados:

- `prisma validate` passou.
- Catálogo RBAC/env passou em testes focados.
- Foram encontrados 29 diretórios de migrations versionadas.
- `prisma migrate status` não conseguiu consultar o banco configurado.
- Readiness com `.env` carregado:
  - ambiente: warning por scanner não strict;
  - banco: failed;
  - schema/migrations/RBAC/usuários privilegiados: skipped;
  - storage: passed;
  - ClamAV: warning;
  - backup/rollback: warning.

Conclusão: não há evidência suficiente para afirmar que o banco de produção/homologação está pronto.

## Conclusão backend

Backend compila e tem boa cobertura, mas **não fecha produção** por causa de escopo de clientes, E2E vermelho, banco/readiness não provados, ClamAV não validado e artefato com segredo.
