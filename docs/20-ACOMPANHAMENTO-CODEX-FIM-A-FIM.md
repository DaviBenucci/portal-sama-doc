# Acompanhamento Codex - Implementacao fim a fim

Data do ciclo: 2026-06-19

Guia controlador: `20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md`

## Regra de execucao

Nenhuma fase seguinte deve avancar enquanto a fase anterior nao estiver `CONCLUIDA`, com evidencia tecnica registrada. Se uma validacao falhar, a falha vira sub-etapa corretiva antes da continuidade.

Status aceitos neste acompanhamento:

- `PENDENTE`
- `EM_EXECUCAO`
- `BLOQUEADA`
- `CONCLUIDA`

## Linha de base dos repositorios

| Repositorio | Caminho local | Branch | Commit | Status inicial |
| --- | --- | --- | --- | --- |
| `portal-sama-api` | `C:\Users\Sama Contabilidade\Desktop\portal-sama-api` | `main` | `239f534` | Sem alteracoes pendentes |
| `portal-sama-web` | `C:\Users\Sama Contabilidade\Desktop\portal-sama-web` | `main` | `646765e` | Sem alteracoes pendentes |
| `portal-sama-docs` | `C:\Users\Sama Contabilidade\Desktop\portal-sama-docs` | `main` | `47ca98c` | Sem alteracoes pendentes antes deste arquivo |
| `portal-sama` legado | `C:\Users\Sama Contabilidade\Desktop\portal-sama` | Consulta historica | N/A | Somente leitura para semantica |

## Escopo congelado

Este ciclo continua somente nos repositorios `portal-sama-api`, `portal-sama-web` e `portal-sama-docs`.

O legado `portal-sama` pode ser lido para entendimento semantico, mas nao deve receber implementacao nova.

Alteracoes ja existentes antes deste checkpoint formal e consideradas parte da linha de base:

- Links publicos novos para assinatura, proposta e documentos.
- Redirecionamentos das rotas publicas legadas para as rotas React novas.
- Rate limit local por token publico nos fluxos de assinatura, proposta e documentos.
- Escopo de `MANAGER` para gestao de links publicos de documentos.
- Smokes web de tokens publicos expirados/revogados e redirects legados.
- `.dockerignore` reforcado em API e Web.

## Decisoes de produto congeladas dos docs 15-19

- Backend e banco avancam antes de telas finais de frontend.
- Frontend so deve ir alem de smoke quando o contrato de API estiver validado.
- Painel do cliente deve reaproveitar a semantica do legado por abas: Geral, Responsaveis, Acessos, Documentos, Certificados e Vida da empresa.
- ZapSign legado e requisito funcional, nao sugestao.
- Assinatura interna e assinatura ZapSign devem conviver como providers distintos.
- Para contratos ZapSign, a assinatura publica interna deve ser bloqueada.
- Integracao Acessorias deve buscar obrigacoes/listas a partir do ano atual e colaboradores/responsaveis a partir do mes atual; dados antigos retornados pela API externa devem ser filtrados antes de persistir.
- Segredos reais nao devem ser lidos, impressos, versionados nem empacotados.

## Matriz de evidencia

Cada etapa concluida deve registrar:

- Fase e etapa.
- Status anterior e final.
- Arquivos alterados.
- Comandos executados.
- Resultado dos comandos.
- Testes executados.
- Pendencias.
- Observacoes de seguranca.

## Fase 0 - Preparacao, escopo e regra rigida de execucao

Status final da fase: `CONCLUIDA`

| Etapa | Status final | Evidencia |
| --- | --- | --- |
| 0.1 Criar checkpoint de execucao | `CONCLUIDA` | Este arquivo foi criado em `docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md`. |
| 0.2 Registrar regra de avanco | `CONCLUIDA` | Regra de execucao registrada no topo deste arquivo. |
| 0.3 Inventariar repositorios | `CONCLUIDA` | Branches, commits e paths registrados na linha de base. |
| 0.4 Bloquear edicao fora do escopo | `CONCLUIDA` | Escopo limitado a API, Web e Docs; legado somente leitura. |
| 0.5 Ler documentos obrigatorios | `CONCLUIDA` | Docs 15, 16, 17, 18, 19 e guia 20 considerados para este ciclo. |
| 0.6 Validar artefatos sensiveis no pacote de trabalho | `CONCLUIDA` | Varredura por nomes de arquivos sensiveis executada sem abrir conteudo. `.env` real existe localmente em API e legado, mas esta ignorado no Git; API/Web tambem excluem `.env*`, dumps, backups, chaves e pacotes no `.dockerignore`. Nenhum segredo foi impresso. |
| 0.7 Definir matriz de evidencia | `CONCLUIDA` | Matriz registrada neste arquivo. |
| 0.8 Congelar decisoes de produto | `CONCLUIDA` | Decisoes dos docs 15-19 registradas acima. |

### Comandos da Fase 0

```powershell
git status --short
git rev-parse --abbrev-ref HEAD
git rev-parse --short HEAD
rg --files -g ".env*" -g "*.pem" -g "*.key" -g "*.p12" -g "*.pfx" -g "*.dump" -g "*.bak" -g "*.backup" -g "*.sqlite" -g "*.db" -g "*.zip" -g "*.tar" -g "*.tar.gz"
rg -n "\.env|dump|backup|pem|key|p12|pfx|zip|tar" .gitignore .dockerignore
rg -n "Fase 0|0\.6|segredo|secret|ZAPSIGN|ZapSign|Contrato|Acessórias" "20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md"
```

### Resultado da Fase 0

- `portal-sama-api`, `portal-sama-web` e `portal-sama-docs` estavam sem alteracoes pendentes no inicio da continuidade.
- `portal-sama-api` contem `.env.example` e `.env` local; `.env` e `.env.*` estao ignorados no Git e excluidos do contexto Docker.
- `portal-sama-web` contem `.env.example`; `.env` e `.env.*` estao ignorados no Git e excluidos do contexto Docker.
- `portal-sama-docs` nao retornou artefatos sensiveis na varredura por nomes.
- `portal-sama` legado contem `.env.example` e `.env` local; uso permitido somente para consulta semantica, sem empacotar arquivos.
- Nenhum valor de segredo foi aberto ou registrado.

## Proxima fase autorizada

Com a Fase 0 concluida, a continuidade deve seguir pela Fase 1 do guia, priorizando preparacao tecnica do backend/API antes de evoluir telas finais do frontend.

## Fase 1 - Ambiente base, banco limpo e configuracao inicial

Status final da fase: `CONCLUIDA`

Imprevisto corrigido: no primeiro teste, o Docker daemon `desktop-linux` nao
estava ativo. O Docker Desktop foi iniciado em segundo plano, o daemon respondeu,
o MySQL local subiu e a API foi validada contra o banco.

| Etapa | Status final | Evidencia |
| --- | --- | --- |
| 1.1 Validar versoes de runtime | `CONCLUIDA` | Node `v24.15.0`, npm `11.12.1`, Docker client/server `29.5.3`, Docker Compose `v5.1.4`, OS `Microsoft Windows NT 10.0.26200.0`, MySQL container `8.4.9`. `mysql --version` nao existe localmente; a validacao foi feita dentro do container. |
| 1.2 Criar/revisar Docker Compose inicial | `CONCLUIDA` | Criado `portal-sama-api/compose.yaml` com MySQL 8.4, volume persistente, rede local, usuario `portal_user` e charset/collation `utf8mb4`/`utf8mb4_0900_ai_ci`. `docker compose --env-file .env.example config` passou sem avisos. |
| 1.3 Criar `.env.example` da API | `CONCLUIDA` | `portal-sama-api/.env.example` recebeu variaveis ZapSign sem valores reais; `env.schema.ts` valida placeholders opcionais. |
| 1.4 Criar `.env.local.example` do web | `CONCLUIDA` | Criado `portal-sama-web/.env.local.example`; Web agora prefere `VITE_API_BASE_URL` e mantem fallback temporario para `VITE_API_URL`. `.gitignore` e `.dockerignore` reabrem somente o example. |
| 1.5 Subir banco limpo | `CONCLUIDA` | `docker compose --env-file .env.example up -d mysql` criou rede, volume e container; `docker compose --env-file .env.example ps` mostrou `portal-sama-mysql` `healthy`. |
| 1.6 Configurar usuario de banco com menor privilegio | `CONCLUIDA` | `SHOW GRANTS FOR CURRENT_USER()` retornou `portal_user` com privilegios no banco `portal_sama`, sem uso de root pela aplicacao. |
| 1.7 Validar charset/collation | `CONCLUIDA` | Consulta no MySQL retornou `@@character_set_server=utf8mb4` e `@@collation_server=utf8mb4_0900_ai_ci`. |
| 1.8 Criar rotina de reset local seguro | `CONCLUIDA` | README da API documenta `docker compose --env-file .env.example down` e `down -v`, com aviso para confirmar `localhost` e nao usar homologacao/producao. |
| 1.9 Validar health basico da API contra banco | `CONCLUIDA` | API subiu temporariamente em `localhost:3001`; `GET /api-v2/health` retornou `ok=true`, `database=up`, `storage=up`. Processo Nest temporario encerrado apos a validacao. |
| 1.10 Marcar fase apenas se banco estiver operacional | `CONCLUIDA` | MySQL local permanece `healthy`; Fase 1 concluida. |

### Arquivos alterados na Fase 1

- `portal-sama-api/compose.yaml`
- `portal-sama-api/.env.example`
- `portal-sama-api/src/config/env.schema.ts`
- `portal-sama-api/src/config/env.schema.spec.ts`
- `portal-sama-api/README.md`
- `portal-sama-web/.env.local.example`
- `portal-sama-web/.env.example`
- `portal-sama-web/.gitignore`
- `portal-sama-web/.dockerignore`
- `portal-sama-web/src/services/api.ts`
- `portal-sama-web/README.md`

### Comandos da Fase 1

```powershell
node --version
npm.cmd --version
docker --version
docker compose version
mysql --version
[System.Environment]::OSVersion.VersionString
docker compose config
Start-Process -FilePath "$Env:ProgramFiles\Docker\Docker\Docker Desktop.exe" -WindowStyle Hidden
docker info
docker compose --env-file .env.example config
docker compose --env-file .env.example up -d mysql
docker inspect --format '{{.State.Health.Status}}' portal-sama-mysql
docker compose --env-file .env.example exec -T mysql mysql -uportal_user -pportal_password portal_sama -e "SELECT @@version AS version, @@character_set_server AS charset_server, @@collation_server AS collation_server; SHOW GRANTS FOR CURRENT_USER();"
docker compose --env-file .env.example ps
npm.cmd run prisma:validate
npm.cmd test -- env.schema.spec.ts
npm.cmd run lint
npm.cmd run build
npm.cmd test
```

### Resultado da Fase 1

- `docker compose config` sem `--env-file` tentou ler o `.env` local da API e gerou avisos de interpolacao. Correcao aplicada: README passou a orientar sempre `docker compose --env-file .env.example ...` para nao tocar no `.env` real.
- `docker compose --env-file .env.example config`: passou.
- Docker daemon indisponivel no primeiro teste; Docker Desktop iniciado e `docker info` passou.
- `docker compose --env-file .env.example up -d mysql`: passou.
- MySQL health: `healthy`.
- MySQL runtime: versao `8.4.9`, charset `utf8mb4`, collation `utf8mb4_0900_ai_ci`.
- Usuario da aplicacao: `portal_user` com privilegios no banco local `portal_sama`.
- Health API local: `ok=true`, `database=up`, `storage=up`.
- API `npm.cmd run prisma:validate`: passou.
- API `npm.cmd test -- env.schema.spec.ts`: passou, 6 testes.
- API `npm.cmd run lint`: passou.
- API `npm.cmd run build`: passou.
- Web `npm.cmd run lint`: passou.
- Web `npm.cmd run build`: passou.
- Web `npm.cmd test`: passou, 12 testes de contrato.

### Estado operacional local apos a Fase 1

- Container `portal-sama-mysql` esta em execucao e `healthy`.
- Processo Nest temporario usado para health em `localhost:3001` foi encerrado.
- Nenhum segredo real foi aberto ou registrado; comandos usaram placeholders locais.

## Fase 2 - Prisma, schema e modelagem de dominio

Status final da fase: `CONCLUIDA`

### Mapa de modelos revisado

| Dominio | Modelos atuais | Decisao |
| --- | --- | --- |
| Usuarios e permissoes | `User`, `Role`, `Permission`, `UserRole`, `RolePermission`, `RefreshToken` | Mantidos; RBAC recebeu novas permissoes seed. |
| Clientes | `Client` | Mantido como raiz do painel; recebeu relacao com credenciais de acesso. |
| Departamentos e responsaveis | `Department`, `ClientDepartmentAssignment` | Mantidos como escopo operacional por cliente/departamento. |
| Documentos | `Document`, `DocumentRequirement`, `DocumentStatusHistory`, `PublicToken` | Mantidos; indices atuais cobrem cliente, status, departamento, token e datas. |
| Certificados | `DigitalCertificate` | Mantido com senha cifrada em `passwordCipher`. |
| Contratos | `Contract` | Recebeu `signatureProvider` e `signatureEnv` para consulta rapida e coexistencia `INTERNAL`/`ZAPSIGN`. |
| ZapSign | `ContractSignatureEnvelope`, `ContractSigner` | Criados conforme doc 18, com token externo, status normalizado, URLs externas e signatarios normalizados. |
| Acessos do cliente | `ClientAccessCredential`, `ClientAccessCredentialAudit` | Criados com `secretCipher`/`secretRef`; sem campo de segredo em claro; auditoria de revelacao sensivel prevista. |
| Acessorias | `AcessoriasCompanyObligationCatalog`, `AcessoriasDelivery`, aliases, sync runs, locks, mappings, fiscal runs/divergences | Mantidos; regra temporal segue docs 12/20. |
| Vida da empresa | `CompanyHistoryEntry`, `CompanyLifeEntry` | Mantidos nesta fase para preservar semantica legada; endpoint alvo continua no modulo `Managers`. |
| Auditoria | `AuditLog`, mais auditoria especifica de credenciais | Mantido; credenciais ganharam trilha dedicada para eventos sensiveis. |

| Etapa | Status final | Evidencia |
| --- | --- | --- |
| 2.1 Rodar validacao Prisma atual | `CONCLUIDA` | `npm.cmd run prisma:validate` passou. |
| 2.2 Revisar modelos existentes | `CONCLUIDA` | Mapa de modelos registrado acima. |
| 2.3 Modelar provider de assinatura | `CONCLUIDA` | Criados enums `ContractSignatureProvider`, `ContractSignatureEnv`, `ContractExternalSignatureStatus`; `Contract` recebeu provider/env; migration `20260619143000_add_contract_signature_provider`. |
| 2.4 Modelar signatarios de contrato | `CONCLUIDA` | Criado `ContractSigner` com campos normalizados, indices por contrato, envelope, email, CPF e token externo. |
| 2.5 Modelar acessos do cliente | `CONCLUIDA` | Criados `ClientAccessCredential` e `ClientAccessCredentialAudit`; segredo fica em `secretCipher` ou `secretRef`, com auditoria para `REVEALED`. |
| 2.6 Revisar Vida da empresa | `CONCLUIDA` | Decisao registrada: manter `CompanyHistoryEntry`/`CompanyLifeEntry` por compatibilidade semantica nesta etapa. |
| 2.7 Ajustar indices criticos | `CONCLUIDA` | Adicionados indices para provider/env de contrato, provider/status de envelope, signedAt, signers, client access por cliente/status, departamento/status, sistema e arquivamento. |
| 2.8 Criar migration incremental | `CONCLUIDA` | Criadas migrations `20260619143000_add_contract_signature_provider` e `20260619150000_add_client_access_credentials`, pequenas e nao destrutivas. |
| 2.9 Testar migration em banco limpo | `CONCLUIDA` | `npm.cmd run prisma:migrate:deploy` aplicou 32 migrations no MySQL local limpo; status ficou up to date. |
| 2.10 Testar migration em banco com dados fake | `CONCLUIDA` | Base `portal_sama_phase2_fake` recebeu tabela antiga minima de contratos + contrato fake; migration ZapSign preservou o registro com `INTERNAL/BR`; base fake removida. Credencial fake validou FK, ciphertext e auditoria no banco local. |
| 2.11 Atualizar seed RBAC | `CONCLUIDA` | `DEFAULT_PERMISSIONS` recebeu ZapSign, acessos, vida da empresa e revelacao sensivel; `npm.cmd run prisma:seed` criou 8 novas permissoes no banco local; teste RBAC passou. |
| 2.12 Gerar Prisma Client | `CONCLUIDA` | `npm.cmd run prisma:generate` passou com Prisma Client `v6.19.3`. |

### Arquivos alterados na Fase 2

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260619143000_add_contract_signature_provider/migration.sql`
- `portal-sama-api/prisma/migrations/20260619150000_add_client_access_credentials/migration.sql`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.spec.ts`

### Comandos da Fase 2

```powershell
npm.cmd run prisma:format
npm.cmd run prisma:validate
npm.cmd run prisma:generate
$env:DATABASE_URL='mysql://portal_user:portal_password@localhost:3306/portal_sama'; npm.cmd run prisma:migrate:deploy
$env:DATABASE_URL='mysql://portal_user:portal_password@localhost:3306/portal_sama'; npm.cmd run prisma:migrate:status
$env:DATABASE_URL='mysql://portal_user:portal_password@localhost:3306/portal_sama'; npm.cmd run prisma:seed
npm.cmd test -- default-rbac.spec.ts
npm.cmd test -- env.schema.spec.ts default-rbac.spec.ts contracts.service.spec.ts
npm.cmd run lint
npm.cmd run build
```

### Resultado da Fase 2

- Prisma validate: passou.
- Prisma generate: passou.
- Migration em banco limpo: 32 migrations aplicadas com sucesso.
- Migration status: banco local up to date.
- Seed RBAC: passou; 8 permissoes novas confirmadas.
- Teste fake de contrato preexistente: dados preservados com defaults `INTERNAL` e `BR`.
- Teste fake de credencial: `has_secret_cipher=1`, `audit_events=1`; registro fake removido.
- API lint: passou.
- API build: passou.
- API testes focados: passaram, 3 suites e 19 testes.

### Observacoes de seguranca da Fase 2

- Nenhum segredo real foi lido ou registrado.
- Campos de acesso do cliente nao armazenam segredo em claro; a modelagem aceita somente `secretCipher` ou `secretRef`.
- Permissoes de revelacao sensivel foram criadas, mas nao foram concedidas a perfis operacionais comuns como `LEGALIZATION`.

## Proxima fase autorizada

Com a Fase 2 concluida, a continuidade seguiu para a Fase 3 do guia, mantendo a regra de backend antes de frontend final.

## Fase 3 - Fundacao de seguranca do backend

Status final da fase: `CONCLUIDA`

Imprevisto corrigido: a primeira execucao de `npm.cmd run lint` falhou por detalhes nas specs novas (`unbound-method` ao ler metadata de throttle e variaveis descartadas em destructuring). As specs foram ajustadas por descriptor e por `Record<string, unknown>` com `delete`; lint passou em seguida.

Sub-etapa corretiva da Etapa 3.6: as rotas dedicadas de ZapSign sync/webhook ainda nao existem no backend atual e pertencem a Fase 7 do guia. Para nao criar endpoint fora de ordem, a Fase 3 registrou a pendencia como requisito bloqueante da Fase 7: qualquer controller ZapSign criado depois deve nascer com `@Throttle`, CSRF quando autenticado e auditoria. Nesta fase foram limitadas as rotas sensiveis ja existentes: login, refresh, uploads autenticados, assinatura publica interna e sync Acessorias.

| Etapa | Status final | Evidencia |
| --- | --- | --- |
| 3.1 Validar bootstrap da API | `CONCLUIDA` | `src/app.bootstrap.ts` revisado: prefixo `api-v2`, Helmet, cookie parser, CORS com credenciais, `ValidationPipe`, `HttpExceptionFilter`, `RequestIdInterceptor` e Swagger apenas fora de producao. |
| 3.2 Revisar schema de ambiente | `CONCLUIDA` | `env.schema.ts` agora exige `CSRF_SECRET` e `CERTIFICATE_ENCRYPTION_KEY` em producao, segredos JWT distintos, cookies seguros em producao e `COOKIE_SECURE=true` quando `COOKIE_SAME_SITE=none`; specs adicionadas. |
| 3.3 Revisar JWT e refresh token | `CONCLUIDA` | `AuthService` ja mantinha access token curto, refresh rotativo e hash HMAC no banco; `auth.service.spec.ts` e e2e permanecem verdes. |
| 3.4 Revisar CSRF | `CONCLUIDA` | Mutacoes autenticadas auditadas por varredura de controllers e e2e; `csrf.service.spec.ts` e `test:e2e` validam rejeicao sem token. |
| 3.5 Revisar guards de permissao | `CONCLUIDA` | Controllers de dominio usam `JwtAuthGuard` + `PermissionsGuard`; e2e validou 401 sem bearer e 403 sem permissao. Excecao justificada: `integrations/acessorias/home-summary` e autenticada, read-only e sem `@Permissions` explicito para servir a Home por perfil. |
| 3.6 Adicionar rate limit em rotas sensiveis | `CONCLUIDA` | Adicionados throttles explicitos para `auth/refresh`, uploads de documentos/certificados/PDF de contrato e sync/backfill Acessorias; `rate-limit-metadata.spec.ts` cobre login, refresh, upload, assinatura publica e sync Acessorias. |
| 3.7 Padronizar erros | `CONCLUIDA` | `HttpExceptionFilter` passou a sanitizar mensagens/detalhes antes da resposta; `http-exception.filter.spec.ts` cobre CPF, bearer token e payload longo. |
| 3.8 Adicionar sanitizacao de logs | `CONCLUIDA` | Criado `common/security/sensitive-data.ts`; `AuditService` reutiliza a sanitizacao para metadados. Specs cobrem keys sensiveis, token bearer, JWT, CPF e base64 longo. |
| 3.9 Revisar auditoria | `CONCLUIDA` | Auditoria de login/refresh/logout, exports de auditoria, downloads e mutacoes sensiveis revisada; falha de persistencia continua nao quebrando fluxo critico. |
| 3.10 Executar testes de seguranca base | `CONCLUIDA` | Specs focadas, lint, build e e2e passaram; resultados listados abaixo. |

### Arquivos alterados na Fase 3

- `portal-sama-api/src/common/security/sensitive-data.ts`
- `portal-sama-api/src/common/security/sensitive-data.spec.ts`
- `portal-sama-api/src/common/security/rate-limit-metadata.spec.ts`
- `portal-sama-api/src/common/filters/http-exception.filter.ts`
- `portal-sama-api/src/common/filters/http-exception.filter.spec.ts`
- `portal-sama-api/src/config/env.schema.ts`
- `portal-sama-api/src/config/env.schema.spec.ts`
- `portal-sama-api/src/modules/audit/audit.service.ts`
- `portal-sama-api/src/modules/audit/audit.service.spec.ts`
- `portal-sama-api/src/modules/auth/auth.controller.ts`
- `portal-sama-api/src/modules/certificates/certificates.controller.ts`
- `portal-sama-api/src/modules/contracts/contracts.controller.ts`
- `portal-sama-api/src/modules/documents/documents.controller.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-deliveries.controller.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-pipeline.controller.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-registrations.controller.ts`
- `portal-sama-docs/docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md`

### Comandos da Fase 3

```powershell
npm.cmd test -- env.schema.spec.ts sensitive-data.spec.ts http-exception.filter.spec.ts audit.service.spec.ts rate-limit-metadata.spec.ts csrf.service.spec.ts auth.service.spec.ts
npm.cmd run lint
npm.cmd run build
npm.cmd run test:e2e
npm.cmd test
rg -n "@Controller|@UseGuards\(JwtAuthGuard, PermissionsGuard\)|@Public\(|@Throttle|csrfService\.assertValidRequest|@Permissions\(" src\modules -g "*.controller.ts"
git diff --check
git status --short
git diff --stat
```

### Resultado da Fase 3

- Specs focadas: passaram, 7 suites e 32 testes.
- `npm.cmd run lint`: falhou inicialmente por specs novas; corrigido e passou.
- `npm.cmd run build`: passou.
- `npm.cmd run test:e2e`: passou, 1 suite e 136 testes.
- `npm.cmd test`: passou, 48 suites e 311 testes; uma rodada inicial avisou sobre worker aberto, mas a suite completa posterior executada na Fase 4 passou sem repetir o aviso.
- `git diff --check`: passou em API e Docs.
- Varredura de controllers: confirmou guards/permissoes/CSRF nas rotas de dominio e throttles nas rotas sensiveis existentes.
- Nenhum segredo real foi aberto, impresso ou versionado.

### Observacoes de seguranca da Fase 3

- A sanitizacao comum cobre chaves sensiveis, bearer tokens, JWTs, CPF completo e payloads longos com formato base64/base64url.
- Producoes sem `CSRF_SECRET` dedicado ou sem `CERTIFICATE_ENCRYPTION_KEY` dedicada passam a falhar cedo no bootstrap de configuracao.
- A excecao `integrations/acessorias/home-summary` permanece apenas autenticada e read-only; caso vire mutavel ou exponha dados administrativos, deve receber permissao explicita antes de avancar.
- O aviso inicial de worker aberto do Jest nao se repetiu na rodada completa posterior.

## Proxima fase autorizada

Com a Fase 3 concluida, a continuidade deve seguir pela Fase 4 do guia: Clientes, colaboradores e responsabilidades.

## Fase 4 - Clientes, colaboradores e responsabilidades

Status final da fase: `CONCLUIDA`

Imprevistos corrigidos:

- Etapa 4.6: transferencia e encerramento de responsabilidade aceitavam `reason/motivo` vazio. O servico passou a exigir motivo e recebeu testes de rejeicao.
- Etapa 4.8: Vida da empresa existia apenas como `POST managers/company-life` com permissao generica de historico. Foram adicionados `GET/PATCH managers/company-life/:companyId` e a permissao propria `company_life.read/write`.
- Etapa 4.9: o dominio `ClientAccessCredential` ja existia no Prisma/RBAC, mas nao tinha API. Foram criados controller, service, DTOs, tipos e testes com listagem mascarada e auditoria.

| Etapa | Status final | Evidencia |
| --- | --- | --- |
| 4.1 Validar CRUD/listagem de clientes | `CONCLUIDA` | `ClientsService` ja cobria filtros, paginacao, status, soft delete e escopo; `clients.service.spec.ts` permaneceu verde. |
| 4.2 Validar dashboard de cliente | `CONCLUIDA` | `GET /clients/:id/dashboard` consolidado com documentos, certificados, propostas e contratos; e2e protege rota por bearer/permissao. |
| 4.3 Completar dados da aba Geral | `CONCLUIDA` | DTO/response de cliente ja retorna atividade, grupo, status, timestamps e aliases legados; specs de cliente verdes. |
| 4.4 Validar colaboradores ativos | `CONCLUIDA` | `ClientAssignmentsService` usa somente usuarios ativos e internos; adicionado teste que rejeita responsavel inativo antes de gravar responsabilidade. |
| 4.5 Validar responsabilidades por departamento | `CONCLUIDA` | Create, update, transfer, end e listagem por escopo permanecem cobertos por `client-assignments.service.spec.ts`. |
| 4.6 Exigir motivo em transferencia/encerramento | `CONCLUIDA` | `reason/motivo` agora e obrigatorio em `end` e `transfer`; specs rejeitam payload sem motivo. |
| 4.7 Auditar alteracao de responsabilidade | `CONCLUIDA` | Auditoria de create/update/end/transfer mantida; metadados passam pela sanitizacao global da Fase 3. |
| 4.8 Criar endpoints da Vida da empresa | `CONCLUIDA` | Adicionados `GET managers/company-life/:companyId` e `PATCH managers/company-life/:companyId`; `POST managers/company-life` passou a usar `company_life.write`; e2e cobre bearer/permissao/CSRF. |
| 4.9 Criar endpoints de acessos do cliente | `CONCLUIDA` | Criados `GET/POST /clients/:clientId/access-credentials`, alias `GET /clients/:clientId/accesses`, `PATCH/DELETE /client-access-credentials/:id`; respostas nunca incluem `secretCipher` nem `secretRef`. |
| 4.10 Testar escopo mine/all | `CONCLUIDA` | Specs de clientes validam `scope=mine`, leitura por responsavel direto e 403 fora de escopo; acessos do cliente reaproveitam `ClientsService.getById` para escopo. |

### Arquivos alterados na Fase 4

- `portal-sama-api/src/modules/client-assignments/client-assignments.service.ts`
- `portal-sama-api/src/modules/client-assignments/client-assignments.service.spec.ts`
- `portal-sama-api/src/modules/clients/client-access-credentials.controller.ts`
- `portal-sama-api/src/modules/clients/client-access-credentials.service.ts`
- `portal-sama-api/src/modules/clients/client-access-credentials.service.spec.ts`
- `portal-sama-api/src/modules/clients/client-access-credentials.types.ts`
- `portal-sama-api/src/modules/clients/clients.module.ts`
- `portal-sama-api/src/modules/clients/dto/create-client-access-credential.dto.ts`
- `portal-sama-api/src/modules/clients/dto/list-client-access-credentials.dto.ts`
- `portal-sama-api/src/modules/clients/dto/update-client-access-credential.dto.ts`
- `portal-sama-api/src/modules/managers/managers.controller.ts`
- `portal-sama-api/src/modules/managers/managers.service.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- `portal-sama-docs/docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md`

### Comandos da Fase 4

```powershell
npm.cmd test -- client-access-credentials.service.spec.ts client-assignments.service.spec.ts clients.service.spec.ts
npm.cmd run lint
npm.cmd run build
npm.cmd run test:e2e
npm.cmd test
git diff --check
git status --short
```

### Resultado da Fase 4

- Specs focadas: passaram, 3 suites e 26 testes.
- `npm.cmd run lint`: passou.
- `npm.cmd run build`: passou.
- `npm.cmd run test:e2e`: passou, 1 suite e 147 testes.
- `npm.cmd test`: passou, 49 suites e 320 testes.
- `git diff --check`: passou na API e Docs.
- Nenhum segredo real foi aberto, impresso ou versionado.

### Observacoes de seguranca da Fase 4

- Endpoints de acessos do cliente nao retornam `secretCipher` nem `secretRef`; retornam apenas `hasSecret`, `hasSecretCipher`, `hasSecretRef` e `secretHint`.
- Criacao, rotacao e arquivamento de acessos registram auditoria especifica em `ClientAccessCredentialAudit` e auditoria global em `AuditService`.
- Revelacao sensivel continua nao exposta nesta fase; `client_accesses.reveal_sensitive` permanece permissao reservada para fluxo auditado futuro.

## Proxima fase autorizada

Com a Fase 4 concluida, a continuidade deve seguir pela Fase 5 do guia: Documentos, certificados e storage privado.

## Fase 5 - Documentos, certificados e storage privado

Status final da fase: `CONCLUIDA`

Imprevistos corrigidos:

- Etapa 5.6/5.9: o schema atual de `DigitalCertificate` nao possui coluna propria de validade. Para nao abrir migracao fora de necessidade imediata, a validade foi implementada em `metadata.validUntil`, com status derivado nas respostas e alertas no dashboard do cliente.
- Etapa 5.10: uma rodada completa inicial de `npm.cmd test` passou, mas avisou worker aberto por streams de storage em specs novas. As specs passaram a destruir explicitamente os streams e a suite completa posterior passou sem o aviso.

| Etapa | Status final | Evidencia |
| --- | --- | --- |
| 5.1 Revisar configuracao de storage | `CONCLUIDA` | Storages de documentos e certificados gravam sob raiz privada, geram somente storage keys relativas, bloqueiam path traversal e aplicam modo restrito best-effort em diretorios/arquivos. Specs novas cobrem armazenamento e rejeicao fora da raiz. |
| 5.2 Validar upload de documentos | `CONCLUIDA` | `DocumentFileValidatorService` ja validava extensao, MIME, magic bytes, tamanho e hash; `documents.service.spec.ts` e `document-file-validator.service.spec.ts` permaneceram verdes. |
| 5.3 Validar scanner de malware | `CONCLUIDA` | Scanner existente mantem modos `off`, `best_effort` e `strict`, staging em quarentena, regras estaticas e fallback ClamAV; specs focadas permaneceram verdes. |
| 5.4 Validar download protegido | `CONCLUIDA` | Criado `buildPrivateDownloadHeaders`; downloads de documentos e certificados usam `private, no-store`, `Pragma`, `Expires`, `nosniff` e `Content-Disposition` com `filename*`. |
| 5.5 Completar checklist de documentos do cliente | `CONCLUIDA` | Resumo de pendencias agora aceita filtro `department`, separa `required_uploaded` de `required_approved`, retorna `pending_badge`, `pending_departments` e status por pendencia. |
| 5.6 Validar certificados digitais | `CONCLUIDA` | Certificados aceitam `validUntil/valid_until`, persistem validade em metadata, retornam `status`, `expiresInDays`, `expired` e `expiresSoon`; validacao de arquivo PKCS#12 segue por extensao, MIME, tamanho, hash e assinatura. |
| 5.7 Validar criptografia de senha de certificado | `CONCLUIDA` | AES-256-GCM permanece em `CertificateCryptoService`, chave fora do banco por `CERTIFICATE_ENCRYPTION_KEY`; listagem/detalhe continuam sem `passwordCipher`. |
| 5.8 Implementar revelacao auditada | `CONCLUIDA` | Criado `POST /certificates/:id/reveal-password`, exigindo `certificates.manage` + `sensitive.reveal`, CSRF, throttle e auditoria `certificates.password.reveal` sem registrar a senha revelada. |
| 5.9 Criar alertas de vencimento | `CONCLUIDA` | Dashboard do cliente passou a retornar contadores `certificates.expiring_soon`, `expired` e `validity_not_informed` a partir da validade em metadata. |
| 5.10 Executar testes do modulo | `CONCLUIDA` | Specs focadas, lint, build, e2e e suite completa passaram; resultados listados abaixo. |

### Arquivos alterados na Fase 5

- `portal-sama-api/src/common/security/download-headers.ts`
- `portal-sama-api/src/common/security/download-headers.spec.ts`
- `portal-sama-api/src/common/security/rate-limit-metadata.spec.ts`
- `portal-sama-api/src/modules/documents/document-storage.service.ts`
- `portal-sama-api/src/modules/documents/document-storage.service.spec.ts`
- `portal-sama-api/src/modules/documents/documents.controller.ts`
- `portal-sama-api/src/modules/documents/documents.service.ts`
- `portal-sama-api/src/modules/documents/documents.service.spec.ts`
- `portal-sama-api/src/modules/documents/documents.types.ts`
- `portal-sama-api/src/modules/documents/dto/required-pending-summary.dto.ts`
- `portal-sama-api/src/modules/certificates/certificate-storage.service.ts`
- `portal-sama-api/src/modules/certificates/certificate-storage.service.spec.ts`
- `portal-sama-api/src/modules/certificates/certificates.controller.ts`
- `portal-sama-api/src/modules/certificates/certificates.service.ts`
- `portal-sama-api/src/modules/certificates/certificates.service.spec.ts`
- `portal-sama-api/src/modules/certificates/certificates.types.ts`
- `portal-sama-api/src/modules/certificates/dto/create-certificate.dto.ts`
- `portal-sama-api/src/modules/certificates/dto/update-certificate.dto.ts`
- `portal-sama-api/src/modules/clients/clients.service.ts`
- `portal-sama-api/src/modules/clients/clients.service.spec.ts`
- `portal-sama-api/src/modules/clients/clients.types.ts`
- `portal-sama-api/test/app.e2e-spec.ts`
- `portal-sama-docs/docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md`

### Comandos da Fase 5

```powershell
npm.cmd test -- download-headers.spec.ts document-storage.service.spec.ts certificate-storage.service.spec.ts documents.service.spec.ts certificates.service.spec.ts clients.service.spec.ts rate-limit-metadata.spec.ts
npm.cmd test -- document-storage.service.spec.ts certificate-storage.service.spec.ts
npm.cmd run lint
npm.cmd run build
npm.cmd run test:e2e
npm.cmd test
git diff --check
git status --short
```

### Resultado da Fase 5

- Specs focadas: passaram, 7 suites e 67 testes.
- Specs de storage apos fechamento explicito de stream: passaram, 2 suites e 4 testes.
- `npm.cmd run lint`: passou.
- `npm.cmd run build`: passou.
- `npm.cmd run test:e2e`: passou, 1 suite e 148 testes.
- `npm.cmd test`: passou, 52 suites e 330 testes.
- `git diff --check`: passou na API e Docs.
- Nenhum segredo real foi aberto, impresso ou versionado.

### Observacoes de seguranca da Fase 5

- Download direto por URL publica continua inexistente: arquivos ficam atras de controllers autenticados, com permissao especifica ou token publico apenas para upload/checklist, nao para download.
- A senha revelada de certificado retorna somente no endpoint dedicado; o audit log recebe apenas metadados e `hasPassword`, nunca o valor sensivel.
- `sensitive.reveal` permanece permissao altamente restrita; perfis operacionais comuns como `LEGALIZATION` nao recebem essa permissao pelo RBAC padrao.
- O modo POSIX restrito de arquivo/diretorio e best-effort para Windows e filesystems que ignoram `chmod`, mas a protecao principal continua sendo storage fora de rotas publicas e path traversal bloqueado.

## Proxima fase autorizada

Com a Fase 5 concluida, a continuidade deve seguir pela Fase 6 do guia: Integracao Acessorias com janela temporal correta.

## Fase 6 - Integracao Acessorias com janela temporal correta

Status final da fase: `CONCLUIDA`

Imprevistos corrigidos:

- Etapa 6.2/6.4: o incremental antigo podia pular o sync quando `DtLastDH` ficava velho e exigir backfill. O fluxo operacional agora sempre usa a janela do ano atual, clampando `DtLastDH` para `01/01` do ano corrente quando necessario.
- Etapa 6.6: o lock ja existia no servico, mas faltava spec unitario direto para concorrencia, expiracao por TTL e release. Foi criado `acessorias-sync-lock.service.spec.ts`.
- Etapa 6.10: a suite completa paralela passou, mas emitiu aviso transitorio de worker. A rodada posterior com `--runInBand --detectOpenHandles` passou sem apontar handles abertos.

| Etapa | Status final | Evidencia |
| --- | --- | --- |
| 6.1 Ler runbook Acessorias | `CONCLUIDA` | `12-RUNBOOK-ACESSORIAS.md` revisado e atualizado com a implementacao local da janela temporal. |
| 6.2 Corrigir janela padrao de obrigacoes | `CONCLUIDA` | `AcessoriasDeliveriesService` envia `DtInitial` em `01/01` do ano atual, `DtFinal` em `31/12` e `DtLastDH` limitado ao ano operacional; spec valida os parametros. |
| 6.3 Corrigir janela de colaboradores/responsaveis | `CONCLUIDA` | `AcessoriasRegistrationsService` envia `DtInitial/DtFinal` do mes atual para colaboradores e filtra inativos/desligados antigos antes de persistir. |
| 6.4 Separar backfill historico de sync operacional | `CONCLUIDA` | Backfill de entregas exige `dt_initial` e `dt_final`; sem janela explicita retorna `ACESSORIAS_BACKFILL_DATE_WINDOW_REQUIRED`. |
| 6.5 Validar filtros pos-consulta | `CONCLUIDA` | Specs cobrem payload antigo de entregas e colaborador desligado antigo; ambos sao contados como `skipped` e nao persistidos no workspace operacional. |
| 6.6 Validar rate limit e lock | `CONCLUIDA` | Specs de rate limiter permanecem verdes e novo spec do lock cobre concorrencia, TTL e release. |
| 6.7 Auditar sync | `CONCLUIDA` | Sync runs/auditoria registram politica de janela, datas, contadores, max pages e rate limit; erros continuam sanitizados. |
| 6.8 Notificar falhas criticas | `CONCLUIDA` | Specs de notificacao DEV permanecem verdes e fluxo de falha incremental continua notificando falhas pesadas/rate limit sem vazar segredo. |
| 6.9 Atualizar docs do Acessorias | `CONCLUIDA` | Runbook documenta ano atual para entregas, mes atual para colaboradores, backfill manual e filtros pos-consulta. |
| 6.10 Executar testes Acessorias | `CONCLUIDA` | Specs focadas, lint, build, e2e, suite completa e detectOpenHandles passaram; resultados listados abaixo. |

### Arquivos alterados na Fase 6

- `portal-sama-api/src/modules/integrations/acessorias/acessorias-deliveries.service.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-deliveries.service.spec.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-registrations.service.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-registrations.service.spec.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-sync-lock.service.spec.ts`
- `portal-sama-docs/12-RUNBOOK-ACESSORIAS.md`
- `portal-sama-docs/docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md`

### Comandos da Fase 6

```powershell
npm.cmd test -- acessorias-deliveries.service.spec.ts acessorias-registrations.service.spec.ts acessorias-sync-lock.service.spec.ts acessorias-rate-limiter.service.spec.ts acessorias-scheduler.service.spec.ts acessorias-dev-notification.service.spec.ts
npm.cmd run lint
npm.cmd run build
npm.cmd run test:e2e
npm.cmd test
npm.cmd test -- --runInBand --detectOpenHandles
git diff --check
git status --short
```

### Resultado da Fase 6

- Specs focadas: passaram, 6 suites e 33 testes.
- `npm.cmd run lint`: passou.
- `npm.cmd run build`: passou.
- `npm.cmd run test:e2e`: passou, 1 suite e 148 testes.
- `npm.cmd test`: passou, 53 suites e 335 testes; a rodada paralela emitiu aviso de worker encerrado a forca.
- `npm.cmd test -- --runInBand --detectOpenHandles`: passou, 53 suites e 335 testes, sem handles abertos reportados.
- `git diff --check`: passou na API e Docs.
- Nenhum segredo real foi aberto, impresso ou versionado.

### Observacoes da Fase 6

- A janela operacional de entregas nao usa mais "ultimos 12 meses"; ela e anual, sempre do ano corrente.
- Historico antigo deve continuar entrando apenas por backfill manual com datas explicitas e revisao antes de alimentar qualquer fluxo operacional.
- Colaboradores ativos nao sao descartados apenas por admissao antiga; o filtro mira desligamento/inatividade antiga e datas de alteracao antigas.

## Proxima fase autorizada

Com a Fase 6 concluida, a continuidade deve seguir pela Fase 7 do guia: Contratos, Legalizacao e ZapSign.
