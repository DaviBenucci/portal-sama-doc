# AUDITORIA DEPLOY 15 - Preparacao das correcoes para Go-Live

Data: 10/06/2026  
Escopo: roteiro detalhado para corrigir os problemas confirmados em `Testes-da-aplicacao-DEPLOY.md` e nos relatorios `AUDITORIA-DEPLOY-00` a `AUDITORIA-DEPLOY-14`.

## 1. Objetivo deste documento

Este documento prepara a execucao das correcoes necessarias para levar o Portal Sama de **No-Go** para **homologacao controlada** e, depois, para **deploy correto em producao**.

Ele nao substitui os relatorios anteriores. Ele consolida:

- o que precisa mudar;
- onde mudar;
- em qual ordem mudar;
- como validar cada mudanca;
- quais evidencias devem ser registradas;
- quais criterios liberam ou bloqueiam o deploy.

## 2. Base documental referenciada

| Documento | Uso neste plano |
|---|---|
| `Testes-da-aplicacao-DEPLOY.md` | Roteiro original de auditoria, QA, seguranca, E2E, perfis, API, frontend, backend e producao. |
| `AUDITORIA-DEPLOY-00-RESUMO-EXECUTIVO.md` | Veredito geral No-Go e lista dos principais bloqueios. |
| `AUDITORIA-DEPLOY-01-QA-FUNCIONAL.md` | Matriz funcional e pendencias de fluxo real por perfil. |
| `AUDITORIA-DEPLOY-02-SEGURANCA.md` | Achados criticos e altos de seguranca, principalmente `SEC-01`, `SEC-02`, `SEC-09` e `SEC-10`. |
| `AUDITORIA-DEPLOY-03-FRONTEND-UI-UX.md` | Falhas E2E da Home, evidencias Playwright e pendencias de UX/mobile. |
| `AUDITORIA-DEPLOY-04-BACKEND-API-INTEGRACOES.md` | Diagnostico de API, integracoes, Web Push, Acessorias e readiness. |
| `AUDITORIA-DEPLOY-05-PLANO-CORRECAO-CHECKLIST-PRODUCAO.md` | Plano inicial de correcao e checklist Go/No-Go. |
| `AUDITORIA-DEPLOY-06-MATRIZ-RASTREABILIDADE-E-CONTINUACAO.md` | Rastreabilidade entre o roteiro original e evidencias executadas. |
| `AUDITORIA-DEPLOY-07-STATUS-ETAPAS-CUMPRIDAS.md` | Status por etapa da auditoria. |
| `AUDITORIA-DEPLOY-08-MATRIZ-ROTAS-ENDPOINTS-PERMISSOES.md` | Mapa de rotas, endpoints e permissoes, com foco em autorizacao. |
| `AUDITORIA-DEPLOY-09-WEB-PUSH-NOTIFICACOES.md` | Estado do MVP Web Push e pendencias de homologacao real. |
| `AUDITORIA-DEPLOY-10-BANCO-MIGRACOES-RBAC-BOOTSTRAP.md` | Banco, migrations, RBAC, seed e bootstrap admin. |
| `AUDITORIA-DEPLOY-11-CONTINUACAO-TESTES-CODIGO-DOCUMENTACAO.md` | Confirmacao dos bloqueios antes do Docker. |
| `AUDITORIA-DEPLOY-12-CONTINUACAO-DOCKER-MIGRATIONS-E2E.md` | Diagnostico da migration quebrada em MySQL limpo e causas dos E2E vermelhos. |
| `AUDITORIA-DEPLOY-13-HOMOLOGACAO-LOCAL-ACESSORIAS-DOCKER.md` | Homologacao local Acessorias + MySQL Docker validada. |
| `AUDITORIA-DEPLOY-14-VALIDACAO-GERAL-COM-DOCKER.md` | Revalidacao geral atual: banco/Acessorias melhoraram, mas release segue No-Go. |

## 3. Estado de partida

### 3.1 O que ja esta positivo

| Frente | Estado atual |
|---|---|
| API build/lint/unitarios | OK. `npm.cmd run build`, `npm.cmd run lint` e `npm.cmd test -- --runInBand` passaram. |
| Web build/lint/contratos | OK. `npm.cmd run build`, `npm.cmd run lint` e `npm.cmd test -- --runInBand` passaram. |
| MySQL Docker limpo | OK apos correcao local da migration de catalogo Acessorias. |
| Migrations | OK localmente: 29 migrations aplicadas em `portal_sama_validation`. |
| Seed/RBAC/bootstrap | OK localmente: 91 permissoes, 9 roles, usuario DEV bootstrap. |
| Readiness local com DB | OK com warnings: ClamAV strict e backup/rollback ainda pendentes. |
| Acessorias local | OK: 21 chamadas, 0 falhas, dados persistidos no MySQL Docker. |

### 3.2 O que ainda bloqueia deploy

| Bloqueio | Severidade | Referencia |
|---|---|---|
| ZIP da API contem `.env`, `.ai-tests`, backups locais e `node_modules`. | Critica | `SEC-01`, `AUDITORIA-DEPLOY-14` |
| Segredos potencialmente expostos precisam rotacao. | Critica | `SEC-01`, `AUDITORIA-DEPLOY-02` |
| `clients.read` permite leitura ampla sem escopo suficiente. | Critica | `SEC-02`, `AUTHZ-01` |
| `GET /integrations/acessorias/home-summary` confia em `profile` enviado pelo cliente. | Critica | `SEC-09`, `AUTHZ-03` |
| E2E API falha em Web Push/CSRF. | Alta | `QA-API-01` |
| E2E Web falha em dois contratos da Home. | Alta | `QA-WEB-01` |
| API tem vulnerabilidade moderada em `qs`. | Media | `DEP-01` |
| ClamAV strict nao esta validado. | Alta | `OPS-CLAMAV` |
| Backup, verify e restore drill nao estao provados fora do app container. | Alta | `OPS-BACKUP` |
| Web Push real com HTTPS/VAPID/navegador nao esta homologado. | Alta | `WEBPUSH-REAL` |
| Perfis DEV, Gestor, Colaborador e Cliente nao foram homologados ponta a ponta. | Alta | `PERFIS-REAL` |

## 4. Regras para executar as correcoes

1. Criar branch de correcao unica ou branches pequenas por fase.
2. Nao commitar `.env`, `.env.local`, logs, dumps, backups, `.ai-tests`, `node_modules`, certificados ou artefatos ZIP.
3. Preservar os relatorios de auditoria como evidencias historicas.
4. A cada fase, rodar testes focados antes de prosseguir.
5. Ao fim de cada fase, registrar saida sanitizada em novo relatorio ou no changelog de correcao.
6. Nao liberar ambiente real antes dos bloqueios criticos de autorizacao e segredo.
7. Para segredos, tratar qualquer valor empacotado em ZIP como potencialmente exposto.

## 5. Ordem recomendada das mudancas

### Fase 0 - Preparacao operacional

Objetivo: preparar o repositorio e evitar perda de evidencia.

O que fazer:

1. Conferir worktrees:
   - `portal-sama-docs`
   - `portal-sama-api`
   - `portal-sama-web`
2. Salvar evidencias atuais:
   - `portal-sama-api/.ai-tests/validation-api-results.json`
   - `portal-sama-web/.ai-tests/validation-web-results.json`
   - `portal-sama-api/.ai-tests/acessorias-api-results-clean.json`
3. Criar uma branch de correcao, por exemplo:
   - `fix/deploy-readiness-go-live`
4. Definir quem aprova decisoes de regra de negocio:
   - escopo de clientes por role;
   - se `MANAGER` ve todos os clientes ou apenas carteira/departamento;
   - se `AUDITOR` pode ter leitura ampla;
   - textos oficiais da Home.

Criterio de aceite:

- Branch criada.
- Evidencias preservadas.
- Decisoes de negocio registradas antes de alterar autorizacao.

### Fase 1 - Higienizar artefatos e tratar segredos

Referencias:

- `AUDITORIA-DEPLOY-02-SEGURANCA.md`, achado `SEC-01`.
- `AUDITORIA-DEPLOY-05-PLANO-CORRECAO-CHECKLIST-PRODUCAO.md`, secao "Seguranca de segredos".
- `AUDITORIA-DEPLOY-14-VALIDACAO-GERAL-COM-DOCKER.md`, secao "Validacao dos artefatos ZIP".

Problema:

- `C:\Users\Sama Contabilidade\Desktop\portal-sama-api.zip` contem `portal-sama-api/.env`.
- O mesmo ZIP contem `.ai-tests`, backups locais e `node_modules`.

O que mudar:

1. Apagar ou mover para quarentena o ZIP atual da API.
2. Criar novo processo de empacotamento que exclua:
   - `.env`
   - `.env.*`
   - `.ai-tests`
   - `node_modules`
   - `coverage`
   - `dist`, se o pacote for de fonte
   - `logs`
   - `*.log`
   - backups, dumps, `*.sql`, `*.tar`, `*.gz`
   - certificados e chaves: `*.pem`, `*.key`, `*.p12`, `*.pfx`
3. Adicionar uma checagem automatizada de pacote antes de compartilhar ZIP.
4. Rotacionar segredos potencialmente expostos:
   - `JWT_ACCESS_SECRET`
   - `JWT_REFRESH_SECRET`
   - `CSRF_SECRET`
   - `DATABASE_URL`
   - `ACESSORIAS_TOKEN`
   - chaves Web Push/VAPID
   - `CERTIFICATE_ENCRYPTION_KEY`
   - tokens de servicos externos
5. Atualizar documentacao de execucao local para usar placeholders e variaveis de ambiente, sem usuario/senha real.

Arquivos provaveis:

- Novo script de empacotamento em `portal-sama-api/scripts/` ou documentacao operacional.
- `README.md` da API, se hoje orientar pacote manual.
- `C:\Users\Sama Contabilidade\Desktop\Rodar-local-Portal-sama.md`, para remover credencial demo se ainda existir.

Comandos de validacao:

```powershell
tar -tf C:\Users\Sama Contabilidade\Desktop\portal-sama-api.zip | Select-String -Pattern '(^|/)\.env($|\.)|\.pem$|\.key$|\.pfx$|\.p12$|node_modules|\.ai-tests|backup|dump|\.sql$'
git ls-files | Select-String -Pattern '(^|/)\.env($|\.)|\.pem$|\.key$|\.pfx$|\.p12$'
```

Criterio de aceite:

- Nenhum ZIP contem `.env`, `.ai-tests`, `node_modules`, backups ou segredos.
- Segredos antigos foram rotacionados.
- Ha evidencia registrada da verificacao do pacote.

### Fase 2 - Consolidar correcoes ja feitas em Acessorias/DB

Referencias:

- `AUDITORIA-DEPLOY-12-CONTINUACAO-DOCKER-MIGRATIONS-E2E.md`, achado `DB-MIG-01`.
- `AUDITORIA-DEPLOY-13-HOMOLOGACAO-LOCAL-ACESSORIAS-DOCKER.md`, secoes 3, 4, 5 e 6.
- `AUDITORIA-DEPLOY-14-VALIDACAO-GERAL-COM-DOCKER.md`, secoes 4 e 6.

Problema anterior:

- Migration de catalogo Acessorias quebrava em MySQL limpo por nome de indice acima de 64 caracteres.
- Filtro Prisma JSON usava `path` em array, incompatavel com MySQL.

Mudancas locais ja feitas e que devem ser consolidadas:

- `portal-sama-api/prisma/schema.prisma`
  - usar `@@unique([clientId, normalizedName], map: "uq_acess_catalog_client_norm")`.
- `portal-sama-api/prisma/migrations/20260609133000_add_acessorias_company_obligation_catalog/migration.sql`
  - usar `UNIQUE INDEX uq_acess_catalog_client_norm`.
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-registrations.service.ts`
  - usar `path: '$.acessorias.externalId'`.
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-registrations.service.spec.ts`
  - teste cobrindo JSONPath string.

Comandos de validacao:

```powershell
npm.cmd run prisma:validate
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand src/modules/integrations/acessorias/acessorias-registrations.service.spec.ts src/modules/integrations/acessorias/acessorias-deliveries.service.spec.ts src/modules/rbac/default-rbac.spec.ts
```

Validacao com DB limpo:

```powershell
npm.cmd run prisma:migrate:deploy
npm.cmd run prisma:migrate:status
npm.cmd run prisma:seed
npm.cmd run prisma:bootstrap-admin
node scripts/validate-operational-readiness.js --soft
```

Criterio de aceite:

- 29 migrations aplicadas em banco limpo.
- Readiness passa com warnings esperados de ClamAV/backup local.
- Acessorias continua com sync local OK.

### Fase 3 - Corrigir `clients.read` e escopo de clientes

Referencias:

- `AUDITORIA-DEPLOY-02-SEGURANCA.md`, `SEC-02`.
- `AUDITORIA-DEPLOY-06-MATRIZ-RASTREABILIDADE-E-CONTINUACAO.md`, casos de teste Clientes/RBAC.
- `AUDITORIA-DEPLOY-08-MATRIZ-ROTAS-ENDPOINTS-PERMISSOES.md`, rotas de clientes.
- `AUDITORIA-DEPLOY-14-VALIDACAO-GERAL-COM-DOCKER.md`, `SEC-02 / AUTHZ-01`.

Arquivos afetados:

- `portal-sama-api/src/modules/clients/clients.service.ts`
  - `applyReadScope()` em torno de `:383`.
  - `assertCanReadClient()` em torno de `:418`.
  - atalho atual por `clients.read` em torno de `:432`.
- `portal-sama-api/src/modules/clients/clients.controller.ts`
  - rotas protegidas por `@Permissions('clients.read')`.
- `portal-sama-api/src/modules/clients/clients.service.spec.ts`
  - incluir testes negativos por escopo.
- Possiveis servicos auxiliares:
  - `client-assignments`
  - `departments`
  - modelos Prisma `ClientDepartmentAssignment`, `Client`, `User`.

Decisao de produto necessaria antes de codar:

| Perfil | Regra sugerida para confirmar |
|---|---|
| `ADMIN` / `DEV` | leitura ampla permitida. |
| `MANAGER` | confirmar se leitura ampla e intencional ou limitada a carteira/departamentos geridos. |
| `AUDITOR` | confirmar se leitura ampla e intencional para auditoria. |
| `CLIENT` | somente o proprio cliente/vinculo. |
| `DEPARTMENT` | somente clientes atribuidos ao departamento ou responsabilidade permitida. |
| `ACCOUNTING` | somente clientes com atribuicao contabil ou departamento permitido. |
| `LEGALIZATION` | somente clientes com atribuicao de legalizacao ou departamento permitido. |
| `TI` | confirmar se precisa leitura ampla ou somente suporte sob solicitacao. |

O que mudar:

1. Substituir a regra "tem `clients.read`, entao pode ler qualquer cliente" por regra de escopo real.
2. Garantir que `list()`, `getById()` e `getDashboard()` usem a mesma politica.
3. Centralizar a politica, por exemplo:
   - `buildClientReadScope(actor)`
   - `canReadClient(actor, client)`
4. Usar relacoes de escopo:
   - role do usuario;
   - `client_department_assignments`;
   - departamento principal do usuario;
   - gestor/responsavel, se a regra oficial exigir;
   - vinculo direto do cliente, para perfil `CLIENT`.
5. Retornar `ForbiddenException` quando o usuario tentar acessar cliente fora do escopo.
6. Garantir que contadores/dashboard tambem sejam filtrados.

Testes obrigatorios:

| Caso | Resultado esperado |
|---|---|
| `CLIENT` acessa proprio cliente | 200/OK |
| `CLIENT` altera id para outro cliente | 403 |
| `DEPARTMENT` lista clientes sem atribuicao | nao aparecem |
| `DEPARTMENT` acessa cliente fora do departamento | 403 |
| `ACCOUNTING` acessa cliente sem atribuicao contabil | 403 |
| `LEGALIZATION` acessa cliente fora do escopo | 403 |
| `MANAGER` sem regra ampla tenta cliente fora da carteira | 403, se essa for a regra aprovada |
| `ADMIN`/`DEV` acessa cliente | OK |

Comandos de validacao:

```powershell
npm.cmd test -- --runInBand src/modules/clients/clients.service.spec.ts
npm.cmd run lint
npm.cmd run build
npm.cmd run test:e2e
```

Criterio de aceite:

- Testes negativos de escopo passam.
- Nenhum perfil interno comum lista ou acessa cliente fora da regra aprovada.
- A matriz de rotas em `AUDITORIA-DEPLOY-08` pode ser marcada como corrigida para clientes.

### Fase 4 - Corrigir escalada de perfil em Acessorias Home

Referencias:

- `AUDITORIA-DEPLOY-02-SEGURANCA.md`, `SEC-09`.
- `AUDITORIA-DEPLOY-08-MATRIZ-ROTAS-ENDPOINTS-PERMISSOES.md`, `AUTHZ-03`.
- `AUDITORIA-DEPLOY-13-HOMOLOGACAO-LOCAL-ACESSORIAS-DOCKER.md`, pendencias.
- `AUDITORIA-DEPLOY-14-VALIDACAO-GERAL-COM-DOCKER.md`, `AUTHZ-03`.

Arquivos afetados:

- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.controller.ts`
  - `GET /integrations/acessorias/home-summary`.
  - hoje recebe `@Query('profile')`.
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.service.ts`
  - `getHomeSummary(actor, profile)`.
  - regra `profile === 'admin'`.
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.service.spec.ts`
  - adicionar testes negativos.
- `portal-sama-web/src/services/acessorias.service.ts`
  - hoje envia `profile` como parametro.
- `portal-sama-web/src/pages/home/HomePage.tsx`
  - hoje deriva perfil no client para UX.

O que mudar:

1. O backend deve derivar o perfil efetivo com base no usuario autenticado.
2. A query `profile` nao deve conceder visao maior que a autorizacao real.
3. Opcoes aceitaveis:
   - remover `profile` da API e retornar resumo conforme role;
   - manter `profile` apenas como preferencia de visualizacao, nunca como autorizacao;
   - se `profile=admin` for enviado por usuario sem permissao, fazer downgrade para o perfil real ou retornar 403.
4. Exigir permissao explicita para visao global, por exemplo:
   - role `ADMIN`/`DEV`;
   - ou permissao nova como `integrations.acessorias.home.admin`.
5. Filtrar sempre por departamento/responsavel quando o usuario nao tiver visao global.
6. Ajustar frontend para nao depender de `profile` como controle de seguranca.

Testes obrigatorios:

| Caso | Resultado esperado |
|---|---|
| `CLIENT` chama `?profile=admin` | nao recebe dados globais ou recebe 403 |
| `DEPARTMENT` chama `?profile=admin` | nao recebe outro departamento |
| `ACCOUNTING` chama `?profile=admin` | nao recebe dados globais |
| `LEGALIZATION` chama `?profile=admin` | nao recebe dados globais |
| `MANAGER` sem permissao global chama `?profile=admin` | nao recebe dados globais |
| `ADMIN`/`DEV` chama home-summary | pode receber visao global |

Comandos de validacao:

```powershell
npm.cmd test -- --runInBand src/modules/integrations/acessorias/acessorias-home.service.spec.ts
npm.cmd test -- --runInBand src/modules/integrations/acessorias/acessorias-registrations.service.spec.ts src/modules/integrations/acessorias/acessorias-deliveries.service.spec.ts
npm.cmd run lint
npm.cmd run build
```

Criterio de aceite:

- Alterar `profile` na URL nao amplia escopo.
- Home Acessorias ainda funciona para DEV.
- A bateria Acessorias local continua verde.

### Fase 5 - Corrigir E2E API Web Push/CSRF

Referencias:

- `AUDITORIA-DEPLOY-09-WEB-PUSH-NOTIFICACOES.md`, `WEBPUSH-01`.
- `AUDITORIA-DEPLOY-12-CONTINUACAO-DOCKER-MIGRATIONS-E2E.md`, `QA-API-01`.
- `AUDITORIA-DEPLOY-14-VALIDACAO-GERAL-COM-DOCKER.md`, `QA-API-01`.

Arquivos afetados:

- `portal-sama-api/test/app.e2e-spec.ts`
  - testes em torno de `notifications/push/subscribe` e `notifications/push/test`.
- `portal-sama-api/src/app.module.ts`
  - carrega `.env.local` e `.env`.
- `portal-sama-api/src/modules/auth/auth.controller.ts`
  - cookie usa `COOKIE_DOMAIN`, `COOKIE_SECURE`, `COOKIE_SAME_SITE`.

Problema:

- A suite E2E carrega configuracoes do `.env` local.
- `COOKIE_DOMAIN`/cookie config interfere no cookie CSRF do Supertest.
- Resultado atual: 134 testes passam, 2 falham com `403` onde esperavam `200`.

O que mudar:

1. Isolar ambiente E2E do `.env` real.
2. No setup E2E, definir explicitamente:

```ts
process.env.COOKIE_DOMAIN = '';
process.env.COOKIE_SECURE = 'false';
process.env.COOKIE_SAME_SITE = 'lax';
```

3. Alternativa mais robusta:
   - usar `.env.test`;
   - ou configurar `ConfigModule` de teste com `ignoreEnvFile: true`.
4. Manter testes negativos de CSRF:
   - sem header deve falhar;
   - sem cookie deve falhar;
   - token divergente deve falhar.
5. Melhorar mensagem de falha quando o E2E receber `403`.

Comandos de validacao:

```powershell
npm.cmd run test:e2e
```

Criterio de aceite:

- E2E API 100% verde.
- Os dois testes Web Push/CSRF passam.
- Testes de CSRF invalido continuam falhando com 403.

### Fase 6 - Corrigir E2E Web da Home

Referencias:

- `AUDITORIA-DEPLOY-03-FRONTEND-UI-UX.md`.
- `AUDITORIA-DEPLOY-12-CONTINUACAO-DOCKER-MIGRATIONS-E2E.md`, `QA-WEB-01`.
- `AUDITORIA-DEPLOY-14-VALIDACAO-GERAL-COM-DOCKER.md`, falhas E2E Web.

Arquivos afetados:

- `portal-sama-web/tests/e2e/smoke.spec.ts`
  - espera `Integracao`.
  - espera `Entregue pelo Acessorias`.
- `portal-sama-web/src/pages/home/HomePage.tsx`
  - renderiza `Diagnostico Acessorias`.
  - renderiza `Baixas sincronizadas`.
- `portal-sama-web/src/services/notifications.service.ts`
  - chama `/notifications/stream`.
- `portal-sama-web/src/services/auth.service.ts`
  - chama `/me/security`.

Decisao de produto necessaria:

| Opcao | Mudanca |
|---|---|
| UI atual esta correta | Atualizar Playwright para os textos atuais: `Diagnostico Acessorias` e `Baixas sincronizadas`. |
| Contrato antigo esta correto | Ajustar Home para voltar a exibir `Integracao` e `Entregue pelo Acessorias`. |

O que mudar:

1. Atualizar teste ou UI conforme decisao de produto.
2. Mockar ou neutralizar:
   - `/api-v2/notifications/stream?take=20`;
   - `/api-v2/me/security`.
3. Reduzir ruido de `ECONNREFUSED` no Playwright.
4. Verificar screenshots desktop/mobile apos ajuste.

Comandos de validacao:

```powershell
npm.cmd run test:e2e
npm.cmd run build
npm.cmd run lint
```

Criterio de aceite:

- Playwright Web 100% verde.
- Sem falhas por texto desalinhado.
- Sem erro de proxy recorrente em rotas nao mockadas.

### Fase 7 - Corrigir vulnerabilidade `qs`

Referencias:

- `AUDITORIA-DEPLOY-02-SEGURANCA.md`, `SEC-07`.
- `AUDITORIA-DEPLOY-14-VALIDACAO-GERAL-COM-DOCKER.md`, `DEP-01`.

Problema:

- `npm.cmd audit --audit-level=moderate` na API retorna 1 vulnerabilidade moderada em `qs`.

O que mudar:

1. Rodar atualizacao controlada:

```powershell
npm.cmd audit fix
```

2. Se alterar lock/dependencias de forma ampla, revisar diff de `package-lock.json`.
3. Repetir testes completos.

Comandos de validacao:

```powershell
npm.cmd audit --audit-level=moderate
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
npm.cmd run test:e2e
```

Criterio de aceite:

- `npm audit` da API retorna 0 vulnerabilidades moderadas ou superiores.

### Fase 8 - Validar ClamAV strict

Referencias:

- `AUDITORIA-DEPLOY-02-SEGURANCA.md`, `SEC-04`.
- `AUDITORIA-DEPLOY-05-PLANO-CORRECAO-CHECKLIST-PRODUCAO.md`, infraestrutura e operacao.
- `AUDITORIA-DEPLOY-14-VALIDACAO-GERAL-COM-DOCKER.md`, readiness warnings.

Problema:

- Localmente, ClamAV nao foi encontrado.
- `SAMA_UPLOAD_SCAN_MODE` ainda nao esta validado como `strict`.

O que mudar:

1. No ambiente de homologacao/producao, instalar ClamAV no container/host correto.
2. Atualizar assinaturas:

```powershell
npm.cmd run ops:clamav:update
```

3. Configurar:

```text
SAMA_UPLOAD_SCAN_MODE=strict
```

4. Rodar teste EICAR controlado.
5. Confirmar que upload malicioso e bloqueado.
6. Confirmar que upload valido continua funcionando.

Criterio de aceite:

- Readiness nao avisa ausencia de scanner.
- EICAR e bloqueado.
- Upload valido segue OK.

### Fase 9 - Provar backup, verify e restore drill

Referencias:

- `AUDITORIA-DEPLOY-02-SEGURANCA.md`, `SEC-08`.
- `AUDITORIA-DEPLOY-05-PLANO-CORRECAO-CHECKLIST-PRODUCAO.md`, infraestrutura e operacao.
- `AUDITORIA-DEPLOY-14-VALIDACAO-GERAL-COM-DOCKER.md`, `OPS-BACKUP`.

O que mudar/preparar:

1. Definir destino seguro para backups fora do app container.
2. Rodar:

```powershell
npm.cmd run ops:backup:create
npm.cmd run ops:backup:verify
npm.cmd run ops:restore:drill
```

3. Executar restore drill contra banco isolado, nunca contra producao.
4. Registrar:
   - timestamp;
   - tamanho dos artefatos;
   - hash/checksum, se disponivel;
   - resultado do restore;
   - responsavel pela validacao.

Criterio de aceite:

- Backup criado.
- Backup verificado.
- Restore drill concluido em alvo isolado.
- Evidencia sanitizada anexada na documentacao.

### Fase 10 - Homologar Web Push real

Referencias:

- `AUDITORIA-DEPLOY-09-WEB-PUSH-NOTIFICACOES.md`.
- `CHANGELOG-WEB-PUSH-MVP.md`.
- `AUDITORIA-DEPLOY-14-VALIDACAO-GERAL-COM-DOCKER.md`.

O que validar:

1. VAPID real configurado:
   - public key;
   - private key;
   - subject/contact.
2. HTTPS real.
3. Service worker instalado no navegador.
4. Usuario autenticado faz subscribe.
5. Usuario faz unsubscribe.
6. Preferencias de notificacao funcionam por usuario/tipo.
7. Evento de negocio dispara notificacao interna e Web Push.
8. Payload nao expoe documento sensivel, token, CNPJ completo ou segredo.
9. Click na notificacao abre area correta sem quebrar sessao.

Comandos de validacao local antes da homologacao real:

```powershell
npm.cmd test -- --runInBand src/modules/notifications/notifications.service.spec.ts src/modules/notifications/notifications-push.service.spec.ts
npm.cmd run test:e2e
```

Criterio de aceite:

- API E2E verde.
- Browser real recebe push.
- Payload sanitizado.
- Preferencias respeitadas.

### Fase 11 - Homologar perfis reais

Referencias:

- `Testes-da-aplicacao-DEPLOY.md`, etapas 4 e 5.
- `AUDITORIA-DEPLOY-01-QA-FUNCIONAL.md`.
- `AUDITORIA-DEPLOY-07-STATUS-ETAPAS-CUMPRIDAS.md`.

Perfis minimos:

- DEV
- Gestor
- Colaborador
- Cliente
- se aplicavel: Departamento, Contabil, Legalizacao, TI, Auditor

Fluxos obrigatorios:

| Fluxo | Perfis | Resultado esperado |
|---|---|---|
| Login/logout | todos | sessao criada e encerrada corretamente |
| Rotas protegidas | todos | sem token retorna 401 |
| Rotas sem permissao | todos | retorna 403 |
| Clientes | todos | escopo correto por role |
| Documentos | DEV/Gestor/Colaborador/Cliente | sem vazamento entre clientes |
| Upload | perfis autorizados | valida tipo, tamanho, scanner e auditoria |
| Download | perfis autorizados | apenas documentos permitidos |
| Acessorias Home | internos | sem escalada por query |
| Notificacoes | internos | preferencias e leitura funcionam |
| Auditoria | DEV/Auditor | logs criticos aparecem |

Criterio de aceite:

- Nenhum perfil ve dados fora do escopo.
- Logs de auditoria registram acoes criticas.
- Fluxos felizes e negativos estao documentados com evidencias.

## 6. Plano de consolidacao tecnica

### 6.1 Commits sugeridos

1. `docs: consolidate deploy correction plan`
   - adiciona este documento.
2. `fix(api): make acessorias catalog migration mysql-safe`
   - consolida migration/schema ja ajustados.
3. `fix(api): use mysql json path for acessorias metadata lookup`
   - consolida JSONPath string e teste.
4. `chore(release): exclude secrets and local artifacts from packages`
   - cria ou ajusta processo de empacotamento.
5. `fix(api): enforce scoped client reads`
   - corrige `clients.read`.
6. `fix(api): derive acessorias home profile server-side`
   - corrige `home-summary`.
7. `test(api): isolate e2e csrf cookie config`
   - fecha E2E API.
8. `test(web): align home e2e contract`
   - fecha E2E Web.
9. `chore(api): resolve qs audit finding`
   - fecha `npm audit`.
10. `docs(ops): add production evidence checklist`
   - registra ClamAV, backup/restore, Web Push real e perfis.

### 6.2 Ordem de merge

1. Correcoes ja validadas localmente de DB/Acessorias.
2. Higiene de artefatos e segredos.
3. Autorizacao de clientes.
4. Autorizacao Acessorias Home.
5. E2E API.
6. E2E Web.
7. Dependencias.
8. Documentacao operacional final.

Motivo: seguranca e integridade de dados precisam vir antes de ajuste de teste visual.

## 7. Bateria final antes de homologacao controlada

### 7.1 API

```powershell
cd C:\Users\Sama Contabilidade\Desktop\portal-sama-api
npm.cmd run prisma:validate
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
npm.cmd audit --audit-level=moderate
npm.cmd run test:e2e
```

Com MySQL Docker limpo:

```powershell
npm.cmd run prisma:migrate:deploy
npm.cmd run prisma:migrate:status
npm.cmd run prisma:seed
npm.cmd run prisma:bootstrap-admin
node scripts/validate-operational-readiness.js --soft
```

### 7.2 Web

```powershell
cd C:\Users\Sama Contabilidade\Desktop\portal-sama-web
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
npm.cmd audit --audit-level=moderate
npm.cmd run test:e2e
```

### 7.3 Acessorias

Reexecutar a bateria sanitizada descrita em `AUDITORIA-DEPLOY-13-HOMOLOGACAO-LOCAL-ACESSORIAS-DOCKER.md`.

Resultado esperado:

- 0 falhas nas chamadas;
- sync runs `SUCCESS`;
- rate limiter sem `failed_total`;
- scheduler manualmente desligado durante a bateria;
- nenhum dado sensivel impresso em relatorio.

## 8. Checklist Go/No-Go atualizado

| Item | Status esperado para Go | Evidencia exigida |
|---|---|---|
| ZIP/API sem `.env` e sem artefatos locais | OK | `tar -tf` sem achados sensiveis |
| Segredos rotacionados | OK | registro interno de rotacao |
| `clients.read` com escopo | OK | testes negativos e QA por perfil |
| Acessorias Home sem escalada por query | OK | testes negativos `profile=admin` |
| API lint/build/unit | OK | comandos verdes |
| API E2E | OK | 100% verde |
| API audit | OK | 0 moderadas ou superiores |
| Web lint/build/contratos | OK | comandos verdes |
| Web E2E | OK | 100% verde |
| DB migrations em banco limpo | OK | 29 migrations aplicadas |
| Seed/RBAC/bootstrap | OK | readiness e login bootstrap |
| ClamAV strict | OK | EICAR bloqueado |
| Backup/restore | OK | restore drill concluido |
| Web Push real | OK | navegador real recebe push |
| Perfis reais | OK | roteiro DEV/Gestor/Colaborador/Cliente aprovado |

## 9. Criterio para considerar pronto para deploy

A aplicacao so deve ser considerada pronta para deploy quando todos os itens abaixo forem verdadeiros:

1. Nenhum artefato contem segredo ou lixo local.
2. Nenhum segredo potencialmente exposto continua valido.
3. Autorizacao de clientes esta corrigida e testada.
4. Acessorias Home nao permite escalada por query.
5. API e Web estao com E2E 100% verde.
6. API e Web estao sem vulnerabilidades moderadas ou superiores no audit.
7. Migrations/seed/bootstrap/readiness passam no ambiente alvo.
8. ClamAV strict esta validado.
9. Backup/restore drill esta provado.
10. Web Push real esta homologado.
11. Perfis reais foram testados de ponta a ponta.
12. Todas as evidencias foram registradas sem expor dados sensiveis.

## 10. Conclusao

O proximo trabalho deve ser executado como uma sequencia controlada, nao como correcoes soltas.

Prioridade absoluta:

1. Segredos e artefatos.
2. Autorizacao de clientes.
3. Autorizacao de Acessorias Home.
4. E2E API/Web.
5. Dependencias.
6. Provas operacionais de producao.

Depois dessas fases, uma nova rodada de `AUDITORIA-DEPLOY` deve ser criada para registrar:

- quais itens foram corrigidos;
- quais comandos passaram;
- quais evidencias foram geradas;
- qual e o novo veredito de Go/No-Go.

Enquanto os itens criticos de segredo e autorizacao estiverem pendentes, o Portal Sama deve permanecer **No-Go para usuarios reais**.
