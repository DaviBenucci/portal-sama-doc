# Auditoria Deploy - Segurança

## Etapa 6 - Auditoria de segurança

## Achados críticos e altos

| ID | Severidade | Achado | Evidência | Correção |
| --- | --- | --- | --- | --- |
| SEC-01 | Crítica | ZIP local da API contém `.env`. | `portal-sama-api.zip` inclui `portal-sama-api/.env`. Git rastreia apenas `.env.example`, mas o pacote expõe segredo se compartilhado. | Apagar/recriar ZIP sem `.env`; rotacionar JWT, DB, Acessórias, VAPID, chaves de certificado e demais segredos que estavam no arquivo. |
| SEC-02 | Crítica | Quebra de escopo em leitura de clientes. | `ClientsService.applyReadScope()` retorna `where` para qualquer usuário não privilegiado que não seja `CLIENT`; `assertCanReadClient()` libera qualquer usuário com `clients.read`. | Aplicar escopo por departamento/responsável/atribuições e adicionar testes negativos por perfil. |
| SEC-03 | Alta | Banco, migrações, RBAC e usuários privilegiados não foram provados. | `prisma:migrate:status` falhou porque MySQL não respondeu; readiness pulou schema/migrations/rbac. | Validar em homologação/EasyPanel com DB real antes do corte. |
| SEC-04 | Alta | Scanner externo de upload não validado. | Readiness avisou que ClamAV não pôde ser lançado; default `SAMA_UPLOAD_SCAN_MODE=best_effort`. | Instalar ClamAV, atualizar assinaturas, rodar EICAR e mudar para `strict` em produção. |
| SEC-05 | Alta | Gate E2E API falha por CSRF/cookie dependente de `.env`. | Testes Web Push esperavam 200 e receberam 403 `CSRF_TOKEN_INVALID`; reprodução mostrou `COOKIE_DOMAIN` local quebrando cookie no agente de teste. | Isolar `.env.test` ou limpar `COOKIE_DOMAIN` no setup E2E. |
| SEC-06 | Média | Criptografia de senha de certificado cai para segredo JWT se `CERTIFICATE_ENCRYPTION_KEY` faltar. | `certificate-crypto.service.ts` usa fallback para `JWT_REFRESH_SECRET`/`JWT_ACCESS_SECRET`. | Exigir chave dedicada em produção e falhar readiness se ausente. |
| SEC-07 | Média | Vulnerabilidade moderada em dependência `qs`. | `npm audit` da API reportou DoS moderado com fix disponível. | Rodar `npm audit fix` ou atualizar lock/dependência compatível e repetir testes. |
| SEC-08 | Média | Backup/restore não provado no ambiente atual. | Readiness informa que backup e rollback precisam ser validados fora do container/app. | Executar `ops:backup:create`, `ops:backup:verify` e `ops:restore:drill` em alvo isolado. |

## Detalhe do achado SEC-02

Pontos de código:

- `portal-sama-api/src/modules/clients/clients.service.ts:33` lista clientes usando `applyReadScope`.
- `portal-sama-api/src/modules/clients/clients.service.ts:70` monta dashboard do cliente após `assertCanReadClient`.
- `portal-sama-api/src/modules/clients/clients.service.ts:383` deixa usuário não `CLIENT` sem filtro adicional.
- `portal-sama-api/src/modules/clients/clients.service.ts:432` libera qualquer usuário com `clients.read`.
- `portal-sama-api/src/modules/rbac/default-rbac.ts:244`, `287` e `313` dão `clients.read` para `DEPARTMENT`, `ACCOUNTING` e `LEGALIZATION`.

Impacto: colaborador departamental pode listar/ver clientes de outros departamentos e consultar contadores operacionais do dashboard. Mesmo que documentos tenham filtros departamentais próprios, o cadastro de cliente e metadados operacionais já são dados sensíveis.

Correção recomendada:

1. Criar função de escopo comum para cliente baseada em:
   - ADMIN/DEV/MANAGER conforme regra de negócio.
   - `CLIENT`: próprio `id`, `username` ou CNPJ.
   - Colaborador: `ClientDepartmentAssignment` ativa por departamento permitido e/ou responsável.
2. Usar a mesma função em `list`, `getById` e `getDashboard`.
3. Adicionar testes:
   - `DEPARTMENT` não lista cliente de outro departamento.
   - `ACCOUNTING` não lê cliente sem atribuição contábil.
   - `LEGALIZATION` não lê cliente fora do escopo.
   - `CLIENT` continua restrito ao próprio cadastro.

## Controles positivos encontrados

| Controle | Evidência |
| --- | --- |
| Headers de segurança | API usa `helmet()`. |
| CORS explícito | CORS usa `CORS_ORIGIN`/`FRONTEND_URL` e `credentials: true`. |
| Validação de payload | `ValidationPipe` com `whitelist`, `forbidNonWhitelisted` e `transform`. |
| Rate limit global | `ThrottlerGuard` global com limite 120/minuto. |
| Sessão | Refresh token em cookie HttpOnly, token de refresh rotacionado e hash HMAC. |
| CSRF | Cookie + header, assinatura, validação de origem/referer quando enviados. |
| Frontend | Access token fica na store em memória, sem persistência em `localStorage`. |
| Upload | Extensão/MIME/magic bytes, quarentena, scan estático e integração ClamAV opcional. |
| Storage | Caminhos privados, nomes aleatórios e bloqueio de traversal. |
| Auditoria | Serviços críticos registram eventos de login, tokens, alterações e fluxos operacionais. |

## Segredos e artefatos

Resultado:

- API Git: apenas `.env.example` rastreado.
- Web Git: apenas `.env.example` rastreado.
- API local: `.env` existe e não está rastreado.
- API ZIP: `.env` foi incluído.

Decisão: **tratar como incidente de vazamento de artefato** caso o ZIP tenha sido compartilhado, enviado para nuvem, anexado ou copiado para terceiros.

## Conclusão de segurança

A base de segurança é boa, mas os dois pontos críticos (`.env` no ZIP e escopo de clientes) impedem produção. Depois das correções, repetir audit, E2E, readiness e homologação manual.

