# AUDITORIA DEPLOY 19 - Execucao da homologacao EasyPanel

Data: 2026-06-11
Branch API: `main`
Branch Web: `main`
Branch Docs: `fix/deploy-readiness-go-live`
Ambiente local: Windows / PowerShell
Ambiente EasyPanel: smoke publico nao autenticado executado; smoke autenticado pendente
Responsavel pela execucao: Codex

## 1. Objetivo

Registrar a execucao das correcoes e testes definidos em:

- `AUDITORIA-DEPLOY-17-HOMOLOGACAO-EASYPANEL-ACESSORIAS-POS-INTEGRACAO.md`
- `AUDITORIA-DEPLOY-18-PLANO-CORRECAO-HOMOLOGACAO-EASYPANEL.md`

## 2. Resumo executivo

| Frente | Status | Observacao |
|---|---|---|
| Home DEV/Gestor | Implementado e validado local | Escopo agora prioriza `responsibleUserId` e alerta pendencias/desatualizacao local. |
| Botao Integrar Acessorias | Implementado local | `Integrar` chama o endpoint operacional unico `POST /integrations/acessorias/operational-sync`. |
| Conciliacao de responsaveis | Implementado local | Responsaveis conciliados podem alimentar `client_department_assignments` sem sobrescrever conflitos. |
| Clientes / Ver meus clientes | Implementado local | Navegacao consolidada em `Clientes`; `scope=all`/`scope=mine` na tela principal. |
| Colaboradores publicos internos | Implementado local | API permite usuario interno sem `collaborators.read`; Web usa payload publico seguro por padrao. |
| Notificacoes | Implementado local | Header e pagina possuem `Ler todas` e `Confirmar alertas`, com invalidacao de cache/SSE. |
| Preferencias/configuracoes | Implementado local | `GET/PATCH /me/preferences` persiste preferencias visuais em `metadata.portalPreferences`. |
| Solicitacoes de acesso | Implementado local | Fluxo passou para dois estagios: gestor encaminha para DEV, DEV decide final. |
| Migration `PENDING_DEV` | OK em banco controlado | Docker MySQL local descartavel; 30 migrations aplicadas e schema em dia. Banco alvo ainda pendente. |
| Playwright | OK local | Suite local passou com 21 testes OK e 1 skip; spec cobre solicitacoes, clientes, colaboradores, notificacoes, preferencias e DEV. |
| Smoke EasyPanel | Parcial publico OK | Frontend, health, CORS e CSRF passaram no dominio real. Fluxos autenticados ainda pendentes. |

## 3. Fase 0 - Preparacao e reproducao

Status: concluida

### Comandos executados

```powershell
git status --short
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
```

### Resultado

| Comando | Resultado | Observacao |
|---|---|---|
| `git status --short` em `portal-sama-docs` | OK | Havia somente novos documentos/evidencias da homologacao 17-19. |
| `git status --short` em `portal-sama-api` | OK | Worktree limpo antes das alteracoes desta rodada. |
| `git status --short` em `portal-sama-web` | OK | Worktree limpo antes das alteracoes desta rodada. |
| `npm.cmd run lint` API | OK | Validacao inicial. |
| `npm.cmd run build` API | OK | Validacao inicial. |
| `npm.cmd test -- --runInBand` API | OK | 44 suites, 276 testes antes das novas alteracoes. |
| `npm.cmd run lint` Web | OK | Validacao inicial. |
| `npm.cmd run build` Web | OK | Validacao inicial. |
| `npm.cmd test -- --runInBand` Web | OK | 9 testes de contrato. |

### Bugs reproduzidos

| ID | Bug | Evidencia | Severidade |
|---|---|---|---|
| HEP-01 | Home DEV/Gestor com numeros/escopo a validar | `evidencias/screenshots/02-home-dev-diagnostico-acessorias.png` | Alta |
| HEP-02 | Painel DEV Acessorias expunha muitas acoes de integracao | `evidencias/screenshots/01-operacao-dev-acessorias-botoes.png` | Critica |
| HEP-03 | Conciliacao podia confirmar responsavel de departamento divergente | Plano AUDITORIA-DEPLOY-18 e codigo do resolver | Media/Alta |
| HEP-04 | Clientes e `Ver meus clientes` ainda precisam consolidacao de escopo | AUDITORIA-DEPLOY-18 | Alta |
| HEP-05 | Notificacoes/preferencias ainda precisam correcao de massa/retencao/persistencia | AUDITORIA-DEPLOY-18 | Media/Alta |
| HEP-06 | Solicitacoes de acesso precisavam calendario multi-dia, gestor solicitar para si e aprovacao DEV | `evidencias/screenshots/03-solicitacao-acesso-calendario-nativo.png`, `evidencias/screenshots/04-solicitacao-acesso-formulario-gestor.png` | Critica |

## 4. Fase 1 - Botao Integrar

Status: implementado local

### Arquivos alterados

```txt
portal-sama-web/src/pages/dev/DevAdminPage.tsx
```

### Mudancas aplicadas

- O painel DEV Acessorias manteve apenas um botao operacional de integracao externa/API: `Integrar`.
- O botao `Integrar` usa o fluxo ja existente de `sync-all`, que integra catalogo e entregas.
- Acoes separadas de preview/import/sync/backfill deixaram de ficar expostas na interface DEV.
- Permaneceram visiveis botoes de diagnostico/local/interno, como status, filas, sync runs, entregas locais, responsaveis pendentes e reconciliacoes.

### Testes

| Teste | Resultado | Evidencia |
|---|---|---|
| `npm.cmd run lint` Web | OK | Sem erros. |
| `npm.cmd run build` Web | OK | Build Vite/TypeScript concluido. |
| `npm.cmd test -- --runInBand` Web | OK | 9 testes de contrato. |

### Pendencias

- Playwright local confirmou que o painel DEV expoe `Integrar` e nao expoe acoes dispersas de previa/importacao/backfill.
- Executar smoke EasyPanel apos deploy.

## 5. Fase 2 - Conciliacao de responsaveis

Status: parcialmente implementado local

### Arquivos alterados

```txt
portal-sama-api/src/modules/integrations/acessorias/acessorias-responsible-resolver.service.ts
portal-sama-api/src/modules/integrations/acessorias/acessorias-responsible-resolver.service.spec.ts
```

### Regras implementadas

| Regra | Status |
|---|---|
| Nome + departamento + usuario ativo unico | Parcial; a rodada focou nos matches exatos por email/username e alias confirmados. |
| Bloqueio de departamento diferente | Implementado para match exato e alias confirmado quando o departamento da Acessorias existe. |
| Ambiguidade vira pendencia | Mantido pelas regras existentes; nao expandido nesta rodada. |
| Alias dentro do mesmo departamento | Implementado para impedir alias confirmado fora do departamento fonte. |
| Usuario inativo nao concilia | Mantido pelas regras existentes; nao alterado nesta rodada. |

### Testes

| Teste | Resultado | Evidencia |
|---|---|---|
| `npm.cmd test -- --runInBand src/modules/access-requests/access-requests.service.spec.ts src/modules/integrations/acessorias/acessorias-responsible-resolver.service.spec.ts` | OK | 2 suites, 10 testes. |
| `npm.cmd test -- --runInBand` API | OK | 44 suites, 278 testes. |

## 6. Fase 3 - Home DEV/Gestor

Status: validado local sem alteracao de codigo nesta leva

### Evidencias de escopo

| Perfil | Resultado esperado | Resultado obtido |
|---|---|---|
| DEV | Global | Coberto por `acessorias-home.service.spec.ts`; full API test OK. |
| Gestor | Departamento/responsabilidade | Coberto por `acessorias-home.service.spec.ts`; full API test OK. |
| Colaborador | Responsabilidade direta | Coberto por `acessorias-home.service.spec.ts`; full API test OK. |

## 7. Fase 4 - Clientes e colaboradores

Status: parcialmente implementado local

Arquivos alterados nesta leva:

```txt
portal-sama-api/src/modules/clients/dto/list-clients.dto.ts
portal-sama-api/src/modules/clients/clients.service.ts
portal-sama-api/src/modules/clients/clients.service.spec.ts
portal-sama-api/src/modules/collaborators/dto/list-collaborators.dto.ts
portal-sama-api/src/modules/collaborators/collaborators.controller.ts
portal-sama-api/src/modules/collaborators/collaborators.service.ts
portal-sama-api/src/modules/collaborators/collaborators.service.spec.ts
portal-sama-api/src/modules/collaborators/collaborators.types.ts
portal-sama-web/src/pages/clients/ClientsOverviewPage.tsx
portal-sama-web/src/services/clients.service.ts
portal-sama-web/src/services/collaborators.service.ts
portal-sama-web/src/types/clients.ts
portal-sama-web/src/types/collaborators.ts
```

### Clientes

| Cenario | Resultado |
|---|---|
| DEV ve todos permitidos | Mantido pelo escopo privilegiado existente. |
| Gestor `Ver meus clientes` | Implementado via `scope=mine` e toggle Web. |
| Colaborador `Ver meus clientes` | Implementado via `scope=mine` para responsabilidade direta/proprio cliente. |
| URL fora de escopo bloqueada | Mantido por `assertCanReadClient`; suite existente segue verde. |

### Colaboradores

| Cenario | Resultado |
|---|---|
| Usuario interno ve dados publicos | Implementado em `GET /collaborators/public` e `visibility=company_public`. |
| Dados sensiveis ausentes | Teste cobre ausencia de roles, permissions, metadata, phone/telefone. |
| Cliente externo bloqueado | Mantido por permissao `collaborators.read`; sem teste E2E dedicado nesta leva. |

### Testes

| Teste | Resultado |
|---|---|
| `npm.cmd test -- --runInBand src/modules/clients/clients.service.spec.ts src/modules/collaborators/collaborators.service.spec.ts src/modules/notifications/notifications.service.spec.ts` | OK; 3 suites, 36 testes. |
| `npm.cmd test -- --runInBand` API | OK; 44 suites, 283 testes. |
| `npm.cmd run build` Web | OK. |

## 8. Fase 5 - Notificacoes e preferencias

Status: parcialmente implementado local

| Cenario | Resultado |
|---|---|
| Ler todas | Mantido por `PATCH /notifications/read-all`; testes existentes verdes. |
| Confirmar todos os alertas | Implementado em `PATCH /notifications/alert-all` e botao Web `Confirmar alertas`. |
| Retencao maxima 100 | Implementado: listagem limitada a 100 e poda por chave operacional apos criacao. |
| Preferencias persistidas apos reload | Mantido por `GET/PATCH /notifications/preferences`; testes existentes verdes. |

Arquivos alterados nesta leva:

```txt
portal-sama-api/src/modules/notifications/notifications.controller.ts
portal-sama-api/src/modules/notifications/notifications.service.ts
portal-sama-api/src/modules/notifications/notifications.service.spec.ts
portal-sama-api/src/modules/notifications/notifications.types.ts
portal-sama-web/src/pages/notifications/NotificationsPage.tsx
portal-sama-web/src/components/notifications/NotificationToasts.tsx
portal-sama-web/src/services/notifications.service.ts
portal-sama-web/src/types/notifications.ts
```

## 9. Fase 6 - Solicitacoes de acesso

Status: implementado local

| Fluxo | Resultado esperado | Resultado obtido |
|---|---|---|
| Colaborador cria | `PENDING_MANAGER` | Mantido. |
| Gestor aprova colaborador | `PENDING_DEV` | Implementado e coberto por teste. |
| Gestor rejeita colaborador | `DECLINED` | Mantido com escopo de decisao restrito ao gestor responsavel. |
| Gestor solicita para si | `PENDING_DEV` | Implementado: API inicia pedido de gestor como pendente DEV e web permite selecionar o proprio gestor. |
| DEV aprova | `APPROVED` | Implementado e coberto por teste. |
| DEV rejeita | `DECLINED` | Implementado pela mesma regra de decisao privilegiada; sem teste dedicado novo. |
| Calendario multi-dia | OK | Implementado no frontend sem calendario nativo e coberto por Playwright local. |

## 10. Fase 7 - Testes locais

### API

| Comando | Resultado |
|---|---|
| `npm.cmd run prisma:validate` | OK; schema valido. O Prisma carregou `.env`, sem imprimir valores. |
| `npx.cmd prisma generate` | OK; client gerado apos novo enum. O Prisma carregou `.env`, sem imprimir valores. |
| `npx.cmd prisma migrate deploy --schema prisma/schema.prisma` com `DATABASE_URL` sintetica para MySQL Docker local | OK; 30 migrations aplicadas, incluindo `20260611130000_add_access_request_pending_dev`. |
| `npx.cmd prisma migrate status --schema prisma/schema.prisma` com `DATABASE_URL` sintetica para MySQL Docker local | OK; database schema up to date. |
| `npm.cmd run lint` | OK |
| `npm.cmd run build` | OK |
| `npm.cmd test -- --runInBand` | OK; 44 suites, 283 testes. |
| `npm.cmd run test:e2e` | OK; 1 suite, 136 testes. |
| `node scripts/validate-operational-readiness.js --soft` | Nao executado nesta rodada. |

### Web

| Comando | Resultado |
|---|---|
| `npm.cmd run lint` | OK |
| `npm.cmd run build` | OK |
| `npm.cmd test -- --runInBand` | OK; 9 testes de contrato. |
| `npm.cmd run test:e2e -- --reporter=line` | OK; 17 testes passaram e 1 skip. |
| `npx.cmd playwright test tests/e2e/smoke.spec.ts --reporter=line` | Coberto pela suite completa; smoke local OK. |
| `npx.cmd playwright test tests/e2e/access-requests.spec.ts --reporter=line` | OK; 4 testes passaram. |

Checagens adicionais:

```txt
portal-sama-api: git diff --check OK
portal-sama-web: git diff --check OK
portal-sama-api: coluna `access_requests.status` validada em MySQL Docker local com enum
`PENDING_MANAGER`, `PENDING_DEV`, `APPROVED`, `DECLINED`, `ARCHIVED`
```

### Validacao de migration em Docker local

Status: concluida em 2026-06-11 14:24 -03:00

Uso de Docker nesta etapa:

- Docker Desktop foi iniciado localmente porque o daemon estava parado.
- Foi criado o container descartavel `portal-sama-homolog-mysql`, com MySQL 8.4 e banco sintetico `portal_sama_homolog_migration`.
- O `DATABASE_URL` usado nesta validacao apontou explicitamente para esse banco local; a `.env` real nao foi usada como destino da migration.
- `prisma migrate deploy` aplicou as 30 migrations do projeto com sucesso.
- `prisma migrate status` retornou schema atualizado.
- A coluna `access_requests.status` ficou com o enum esperado, incluindo `PENDING_DEV` e default `PENDING_MANAGER`.
- O container `portal-sama-homolog-mysql` foi removido ao final da validacao.

## 11. Fase 8 - Smoke EasyPanel

Status: parcial publico OK; autenticado pendente

| Cenario | Resultado | Evidencia sanitizada |
|---|---|---|
| Frontend publico | OK; HTTP 200. | `npm.cmd run smoke:public` no dominio real. |
| Health | OK; HTTP 200, `database=up`, `storage=up`. | `GET https://portal.samacontabil.com.br/api-v2/health`. |
| CORS | OK; preflight HTTP 204 com credenciais e origem esperada. | Origem `https://portal.samacontabil.com.br`. |
| CSRF | OK; HTTP 200 com token JSON e cookie CSRF. | Sem registrar token/cookie. |
| Login DEV | Nao executado nesta rodada. | - |
| Home DEV | Nao executado nesta rodada. | - |
| Dev Integrar/status local | Nao executado nesta rodada. | - |
| Login Gestor | Nao executado nesta rodada. | - |
| Home Gestor | Nao executado nesta rodada. | - |
| Solicitacao teste | Nao executado nesta rodada. | - |
| Notificacao teste | Nao executado nesta rodada. | - |

## 12. Arquivos alterados

### portal-sama-api

```txt
prisma/schema.prisma
prisma/migrations/20260611130000_add_access_request_pending_dev/migration.sql
src/modules/access-requests/access-requests.service.ts
src/modules/access-requests/access-requests.service.spec.ts
src/modules/integrations/acessorias/acessorias-responsible-resolver.service.ts
src/modules/integrations/acessorias/acessorias-responsible-resolver.service.spec.ts
src/modules/clients/dto/list-clients.dto.ts
src/modules/clients/clients.service.ts
src/modules/clients/clients.service.spec.ts
src/modules/collaborators/dto/list-collaborators.dto.ts
src/modules/collaborators/collaborators.controller.ts
src/modules/collaborators/collaborators.service.ts
src/modules/collaborators/collaborators.service.spec.ts
src/modules/collaborators/collaborators.types.ts
src/modules/notifications/notifications.controller.ts
src/modules/notifications/notifications.service.ts
src/modules/notifications/notifications.service.spec.ts
src/modules/notifications/notifications.types.ts
```

### portal-sama-web

```txt
src/pages/access-requests/AccessRequestPage.tsx
src/components/notifications/NotificationToasts.tsx
src/pages/dev/DevAdminPage.tsx
src/types/access-requests.ts
src/pages/clients/ClientsOverviewPage.tsx
src/pages/notifications/NotificationsPage.tsx
src/services/clients.service.ts
src/services/collaborators.service.ts
src/services/notifications.service.ts
src/types/clients.ts
src/types/collaborators.ts
src/types/notifications.ts
tests/e2e/access-requests.spec.ts
```

### portal-sama-docs

```txt
AUDITORIA-DEPLOY-19-EXECUCAO-HOMOLOGACAO-EASYPANEL.md
CONTEXTO-CODEX-ATUAL.md
```

## 13. Evidencias geradas

```txt
Nao foram gerados prints ou arquivos externos novos nesta rodada.
Foi usada validacao operacional em Docker local para a migration `PENDING_DEV`.
Foram usadas as evidencias fornecidas em evidencias/entrada/ e evidencias/screenshots/.
Os resultados de validacao local estao registrados neste documento.
Artefatos temporarios de Playwright em `test-results/` foram removidos apos validacao.
O container Docker descartavel `portal-sama-homolog-mysql` foi removido apos a checagem.
Smoke publico real executado sem credenciais contra `https://portal.samacontabil.com.br`.
```

## 14. Pendencias restantes

| Pendencia | Severidade | Bloqueia homologacao? | Proxima acao |
|---|---|---|---|
| Aplicar migration `20260611130000_add_access_request_pending_dev` no banco alvo | Alta | Sim | Migration validada em MySQL Docker local; rodar `prisma migrate deploy` no ambiente alvo antes do smoke real. |
| Smoke EasyPanel autenticado apos deploy | Critica | Sim | Health publico passou; validar login, home, Integrar, solicitacao e notificacoes no dominio real com credenciais atuais. |
| Home DEV/Gestor com dados reais | Alta | Sim | Local coberto por unit/Playwright; falta confirmar numeros no EasyPanel apos deploy. |
| Clientes / Ver meus clientes / colaboradores publicos internos em EasyPanel | Alta | Sim | Local Playwright cobriu `scope=mine` e listagem publica; falta smoke real. |
| Notificacoes e preferencias em EasyPanel | Media/Alta | Sim | Local Playwright cobriu header, pagina e preferencias; falta reload/smoke real. |
| Conciliacao por nome + departamento | Media/Alta | Parcial | Revisar se a regra nominal precisa ficar explicita alem dos matches exatos/alias. |

## 15. Veredito

Escolher uma opcao:

- `Pronto para homologacao controlada`
- `Nao pronto para usuarios reais`
- `Nao testavel por falha de ambiente`

Veredito selecionado: `Nao pronto para usuarios reais`

Justificativa:

```txt
As correcoes estao implementadas e verdes em lint/build/test/E2E locais.
A migration foi validada em MySQL Docker local controlado e o smoke publico real
passou no dominio `portal.samacontabil.com.br`. Ainda falta aplicar a migration
no banco alvo, executar smoke autenticado e validar em dados reais as fases Home,
Clientes/Colaboradores e Notificacoes/Preferencias.
```

## 16. Continuidade 2026-06-12 - Fechamento das pendencias do TXT

Status: implementado e validado local

### Mudancas aplicadas

- Criado orquestrador backend `POST /integrations/acessorias/operational-sync`.
- Criada aplicacao local de responsaveis conciliados para `client_department_assignments`, sem sobrescrever responsavel ativo divergente.
- Criado endpoint local `POST /integrations/acessorias/responsibles/apply-client-assignments`.
- Criado endpoint local `POST /integrations/acessorias/deliveries/apply-to-workspace/bulk`.
- Home Acessorias passou a priorizar `responsibleUserId` e a retornar aviso quando ha responsaveis pendentes/dados locais defasados.
- Painel DEV manteve `Integrar` como acao externa unica e adicionou `Aplicar vencimentos` como acao local.
- Navegacao de clientes foi consolidada em uma unica pagina `Clientes`; rotas antigas redirecionam para `/clientes` ou `/clientes?scope=mine`.
- Tela `Clientes` passou a usar `scope=all` e `scope=mine`.
- `GET /collaborators/public` passou a permitir usuario interno autenticado sem `collaborators.read`; cliente externo continua bloqueado.
- Tela `Colaboradores` usa payload publico para usuario interno sem permissao administrativa e oculta perfis, permissoes, metadados e telefone.
- Header de notificacoes ganhou `Ler todas` e `Confirmar alertas`.
- Pagina de notificacoes renomeou `Limpar nao lidas` para `Ler todas`.
- Criado `GET/PATCH /me/preferences`, persistindo `density`, `startPage` e `reducedMotion` em `user.metadata.portalPreferences`.
- Tela `Configuracoes` passou a salvar preferencias visuais reais e consumir preferencias de notificacao por tipo.
- Testes cobrem rejeicao do gestor, rejeicao DEV, calendario desmarcando dia, notificacao final, colaboradores publicos, preferencias e endpoints DEV.

### Validacoes 2026-06-12

| Comando | Resultado |
|---|---|
| API `npm.cmd run prisma:validate` | OK; schema valido. O Prisma carregou `.env`, sem imprimir valores. |
| API `npm.cmd run lint` | OK |
| API `npm.cmd run build` | OK |
| API `npm.cmd test -- --runInBand` | OK; 44 suites, 290 testes. |
| API `npm.cmd run test:e2e` | OK; 1 suite, 136 testes. |
| Web `npm.cmd run lint` | OK |
| Web `npm.cmd run build` | OK |
| Web `npm.cmd test -- --runInBand` | OK; 12 testes de contrato. |
| Web `npx.cmd playwright test tests/e2e/access-requests.spec.ts --reporter=line` | OK; 8 testes. |
| Web `npm.cmd run test:e2e -- --reporter=line` | OK; 21 testes passaram e 1 skip. |
| `git diff --check` em docs/API/Web | OK |

### Ambiente

- Nenhuma dependencia nova foi instalada nesta rodada.
- Docker nao foi usado nesta rodada de 2026-06-12.
- Nao houve smoke autenticado real nesta rodada.
- Nao houve `prisma migrate deploy` contra banco alvo nesta rodada.

### Veredito apos a continuidade

`Pronto para homologacao controlada`, condicionado a:

- aplicar migrations pendentes no banco alvo;
- realizar deploy controlado;
- executar smoke autenticado EasyPanel com credenciais atuais;
- validar contadores Home/Clientes/Colaboradores/Notificacoes com dados reais.
