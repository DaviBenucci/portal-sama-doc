# CONTEXTO CODEX ATUAL

Atualizado em: 2026-06-12 11:40 -03:00
Sessao atual: Homologacao EasyPanel Acessorias - fechamento local das pendencias do TXT

## Objetivo ativo

Continuar a homologacao EasyPanel pos-integracao da Acessorias, seguindo:

- `README-HOMOLOGACAO-EASYPANEL.md`
- `PROMPT-CODEX-HOMOLOGACAO-EASYPANEL-ACESSORIAS.md`
- `AUDITORIA-DEPLOY-17-HOMOLOGACAO-EASYPANEL-ACESSORIAS-POS-INTEGRACAO.md`
- `AUDITORIA-DEPLOY-18-PLANO-CORRECAO-HOMOLOGACAO-EASYPANEL.md`
- `AUDITORIA-DEPLOY-19-TEMPLATE-EXECUCAO-HOMOLOGACAO-EASYPANEL.md`
- registro em `AUDITORIA-DEPLOY-19-EXECUCAO-HOMOLOGACAO-EASYPANEL.md`

Nao recomecar a auditoria do zero. Prosseguir das pendencias listadas abaixo.

## Branches atuais

- `portal-sama-docs`: `main`
- `portal-sama-api`: `main`
- `portal-sama-web`: `main`

## Contexto lido nesta etapa

- `README-HOMOLOGACAO-EASYPANEL.md`
- `PROMPT-CODEX-HOMOLOGACAO-EASYPANEL-ACESSORIAS.md`
- `AUDITORIA-DEPLOY-17-HOMOLOGACAO-EASYPANEL-ACESSORIAS-POS-INTEGRACAO.md`
- `AUDITORIA-DEPLOY-18-PLANO-CORRECAO-HOMOLOGACAO-EASYPANEL.md`
- `AUDITORIA-DEPLOY-19-TEMPLATE-EXECUCAO-HOMOLOGACAO-EASYPANEL.md`
- `00-LEIA-ME-PARA-IA-MVP.md`
- `01-ESTADO-ATUAL-CODIGO-DOCUMENTACAO.md`
- `02-PLANO-FECHAMENTO-MVP.md`
- `03-CONTRATO-ACESSORIAS-OPERACIONAL.md`
- `04-DIVERGENCIAS-DOCS-CODIGO.md`
- `05-PROMPT-CODEX-FECHAMENTO-MVP.md`
- `06-NOTIFICACOES-WEB-PUSH-MVP.md`
- `07-PROMPT-CODEX-PIPELINE-ACESSORIAS-PERSISTENCIA-PUSH-DEV.md`
- `AUDITORIA-DEPLOY-13-HOMOLOGACAO-LOCAL-ACESSORIAS-DOCKER.md`
- `AUDITORIA-DEPLOY-16-EXECUCAO-CORRECOES-GO-LIVE.md`
- `evidencias/entrada/Homolacao-portal-sama-easypanel.txt`
- screenshots em `evidencias/screenshots/`

## Fase 0

Concluida.

- Worktrees conferidos em docs/API/web.
- Validacao inicial da API: lint OK, build OK, testes OK.
- Validacao inicial do Web: lint OK, build OK, contratos OK.
- Screenshots/evidencias analisados para reproduzir os sintomas documentados.

## Implementado nesta sessao

### API

- Adicionado status Prisma `PENDING_DEV` em `AccessRequestStatus`.
- Criada migration:
  `prisma/migrations/20260611130000_add_access_request_pending_dev/migration.sql`.
- Fluxo de solicitacoes de acesso atualizado:
  - colaborador cria `PENDING_MANAGER`;
  - gestor aprova colaborador e encaminha para `PENDING_DEV`;
  - gestor solicitando para si cria pedido `PENDING_DEV`;
  - DEV/ADMIN/TI fazem aprovacao/rejeicao final;
  - listagem de pendencias separa `PENDING_MANAGER` para gestor e `PENDING_DEV` para decisores privilegiados.
- `getMyLatest` passou a contemplar tambem solicitacoes feitas por gestor.
- Notificacoes do fluxo foram ajustadas para diferenciar aprovacao do gestor e aprovacao final.
- Conciliacao de responsaveis Acessorias ficou mais restrita:
  - match exato por email/username so auto-confirma se o departamento local combina com o departamento fonte;
  - alias confirmado tambem passa a respeitar departamento;
  - divergencia vira `PENDING_REVIEW`.
- Clientes:
  - `ListClientsDto` ganhou `scope`, `mine`, `myClientsOnly` e `my_clients_only`;
  - `scope=mine` restringe a cliente proprio ou atribuicoes ativas onde o usuario e responsavel/gestor;
  - `scope=all` continua significando "todos permitidos pelo backend", sem acesso global para perfil nao privilegiado.
- Colaboradores:
  - `GET /collaborators/public` e `visibility=company_public` implementados;
  - payload publico interno nao retorna roles, permissions, metadata, phone/telefone;
  - apos a continuidade de 2026-06-12, usuario interno autenticado sem `collaborators.read` pode usar a visao publica; cliente externo segue bloqueado.
- Notificacoes:
  - `MAX_TAKE` reduzido para 100;
  - criacao/teste de notificacao agora poda excedentes por chave operacional;
  - `PATCH /notifications/alert-all` implementado para confirmar alertas em massa;
  - evento SSE `notifications.clear_alerts` adicionado.

### Web

- Tela DEV Acessorias simplificada:
  - somente um botao externo/API de integracao ficou exposto: `Integrar`;
  - demais botoes visiveis agora sao diagnosticos, locais ou reconciliacoes internas.
- Tipo `AccessRequestStatus` passou a incluir `PENDING_DEV`.
- Tela de solicitacao de acesso:
  - calendario multi-dia proprio substituiu o date picker nativo;
  - gestor pode selecionar o proprio usuario ativo para solicitar acesso para si;
  - painel de aprovacoes usa `PENDING_MANAGER` para gestor e `PENDING_DEV` para DEV/ADMIN/TI;
  - estatisticas consideram `PENDING_MANAGER` + `PENDING_DEV`.
- Clientes:
  - a pagina principal `/clientes` ganhou toggle `Todos permitidos` / `Ver meus clientes`;
  - `/clientes/visao-geral` redireciona para `/clientes`;
  - `/departamentos/clientes` redireciona para `/clientes?scope=mine`;
  - services/types enviam `scope=mine` ao backend.
- Colaboradores:
  - services/types conhecem `listPublicCollaborators()` e `visibility=company_public`.
- Notificacoes:
  - pagina ganhou botao `Confirmar alertas`;
  - service chama `PATCH /notifications/alert-all`;
  - stream invalida cache tambem para `notifications.clear_alerts`.
- Toasts globais:
  - `NotificationToasts` tambem invalida cache ao receber `notifications.clear_alerts`.
- Playwright:
  - criada `tests/e2e/access-requests.spec.ts`;
  - cobre gestor solicitando para si com status `PENDING_DEV`;
  - cobre DEV aprovando solicitacao `PENDING_DEV`;
  - cobre painel DEV expondo `Integrar` sem botoes dispersos de previa/importacao/backfill;
  - cobre contratos de `Ver meus clientes` e `Confirmar alertas`.

### Docs

- Criado `AUDITORIA-DEPLOY-19-EXECUCAO-HOMOLOGACAO-EASYPANEL.md`.
- Este `CONTEXTO-CODEX-ATUAL.md` foi atualizado para a nova frente 17-19.

## Continuidade executada nesta etapa

- `git status --short --branch` foi conferido em docs/API/web.
- Os tres repositorios estavam limpos antes da atualizacao documental desta etapa.
- Docker estava instalado, mas o daemon estava parado.
- Docker Desktop foi iniciado localmente.
- Foi criado um MySQL 8.4 descartavel em `portal-sama-homolog-mysql`, com banco sintetico `portal_sama_homolog_migration`.
- `prisma migrate deploy` foi executado contra esse banco local controlado, usando `DATABASE_URL` sintetica apontada explicitamente para `127.0.0.1:33307`.
- As 30 migrations foram aplicadas com sucesso, incluindo `20260611130000_add_access_request_pending_dev`.
- `prisma migrate status` retornou schema atualizado.
- A coluna `access_requests.status` foi conferida no MySQL local e contem `PENDING_MANAGER`, `PENDING_DEV`, `APPROVED`, `DECLINED`, `ARCHIVED`, com default `PENDING_MANAGER`.
- O container descartavel `portal-sama-homolog-mysql` foi removido ao final.
- O container antigo `portal-sama-audit-mysql`, ja existente no ambiente, nao foi alterado.
- Smoke publico real foi executado contra `https://portal.samacontabil.com.br`, sem credenciais e sem alteracao de dados.
- Resultado do smoke publico: frontend HTTP 200, `/api-v2/health` HTTP 200 com `database=up` e `storage=up`, CORS preflight HTTP 204 com credenciais, CSRF HTTP 200 com token/cookie presentes sem registrar os valores.

## Continuidade executada em 2026-06-12

- Implementado `POST /integrations/acessorias/operational-sync` como orquestrador operacional unico do botao `Integrar`.
- Implementado `POST /integrations/acessorias/responsibles/apply-client-assignments` para criar atribuicoes ativas em `client_department_assignments` a partir de entregas conciliadas.
- Implementado `POST /integrations/acessorias/deliveries/apply-to-workspace/bulk` para aplicar vencimentos locais por mes/departamento, sem chamar API externa.
- Painel DEV:
  - `Integrar` chama `operational-sync`;
  - `Aplicar vencimentos` chama somente aplicacao local;
  - acoes diagnosticas/localizadas permanecem separadas.
- Home Acessorias:
  - escopo por usuario prioriza `responsibleUserId`;
  - nome/username ficam como fallback;
  - avisos indicam responsaveis pendentes ou dados locais desatualizados.
- Clientes:
  - menu mostra uma unica entrada `Clientes`;
  - `/clientes` e a tela principal;
  - `/clientes/visao-geral` redireciona para `/clientes`;
  - `/departamentos/clientes` redireciona para `/clientes?scope=mine`;
  - tela usa `scope=all` e `scope=mine`.
- Colaboradores:
  - `GET /collaborators/public` permite usuario interno autenticado sem `collaborators.read`;
  - role `CLIENT` continua bloqueada;
  - tela usa payload publico quando o usuario nao tem permissao administrativa;
  - colunas sensiveis ficam ocultas no modo publico.
- Notificacoes:
  - sininho do header ganhou `Ler todas` e `Confirmar alertas`;
  - pagina renomeou `Limpar nao lidas` para `Ler todas`;
  - cache de header/pagina e eventos SSE cobrem clear unread/alerts.
- Configuracoes:
  - `GET/PATCH /me/preferences` persistem `density`, `startPage`, `reducedMotion` em `user.metadata.portalPreferences`;
  - tela salva preferencias visuais com feedback;
  - tela consome preferencias reais de notificacao por tipo.
- Solicitacoes de acesso:
  - testes agora cobrem gestor rejeitando colaborador;
  - DEV rejeitando solicitacao `PENDING_DEV`;
  - calendario desmarcando dia selecionado;
  - notificacao final de aceite/declinio.

## Validacoes executadas em 2026-06-12

### API

- `npm.cmd run prisma:validate` - OK. O Prisma carregou `.env`, sem imprimir valores.
- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand src/modules/integrations/acessorias/acessorias-responsible-resolver.service.spec.ts src/modules/collaborators/collaborators.service.spec.ts src/modules/auth/auth.service.spec.ts src/modules/access-requests/access-requests.service.spec.ts src/modules/notifications/notifications.service.spec.ts` - OK, 5 suites e 46 testes.
- `npm.cmd test -- --runInBand` - OK, 44 suites e 290 testes.
- `npm.cmd run test:e2e` - OK, 1 suite e 136 testes.
- `git diff --check` - OK.

### Web

- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 12 testes de contrato.
- `npx.cmd playwright test tests/e2e/access-requests.spec.ts --reporter=line` - OK, 8 testes.
- `npm.cmd run test:e2e -- --reporter=line` - OK, 21 testes passaram e 1 skip.
- `git diff --check` - OK.

### Ambiente 2026-06-12

- Nenhuma dependencia nova foi instalada.
- Docker nao foi usado nesta rodada.
- Nao foi executado smoke autenticado real.
- Nao foi executado `prisma migrate deploy` no banco alvo.

## Validacoes executadas

### API

- `npx.cmd prisma generate` - OK. O Prisma carregou `.env`, sem imprimir valores.
- `npm.cmd run prisma:validate` - OK. O Prisma carregou `.env`, sem imprimir valores.
- `npx.cmd prisma migrate deploy --schema prisma/schema.prisma` com MySQL Docker local - OK, 30 migrations aplicadas.
- `npx.cmd prisma migrate status --schema prisma/schema.prisma` com MySQL Docker local - OK, schema atualizado.
- `npm.cmd test -- --runInBand src/modules/access-requests/access-requests.service.spec.ts` - OK.
- `npm.cmd test -- --runInBand src/modules/access-requests/access-requests.service.spec.ts src/modules/integrations/acessorias/acessorias-responsible-resolver.service.spec.ts` - OK, 2 suites e 10 testes.
- `npm.cmd test -- --runInBand src/modules/clients/clients.service.spec.ts src/modules/collaborators/collaborators.service.spec.ts src/modules/notifications/notifications.service.spec.ts` - OK, 3 suites e 36 testes.
- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 44 suites e 283 testes.
- `npm.cmd run test:e2e` - OK, 1 suite e 136 testes.
- `git diff --check` - OK.

### Web

- `npm.cmd run lint` - OK.
- `npm.cmd run build` - OK.
- `npm.cmd test -- --runInBand` - OK, 9 testes de contrato.
- `npx.cmd playwright test tests/e2e/access-requests.spec.ts --reporter=line` - OK, 4 testes.
- `npm.cmd run test:e2e -- --reporter=line` - OK, 17 testes passaram e 1 skip.
- `npm.cmd run smoke:public -- --url https://portal.samacontabil.com.br --api-url https://portal.samacontabil.com.br/api-v2 --origin https://portal.samacontabil.com.br --timeout 10000` - OK.
- `git diff --check` - OK.

## Nao executado nesta sessao

- Smoke EasyPanel autenticado/por perfil e fluxos reais.
- `prisma migrate deploy` contra banco alvo.
- Instalacao de dependencias novas.

## Ferramentas/ambiente usados nesta etapa

- Docker Desktop foi iniciado para validacao controlada.
- Container criado e removido: `portal-sama-homolog-mysql`.
- Banco sintetico usado: `portal_sama_homolog_migration`.
- Nenhuma dependencia nova foi instalada.

## Status atual dos arquivos

### portal-sama-api

- Ha alteracoes locais de implementacao/testes ainda nao commitadas:
- arquivos de auth/preferencias;
- colaboradores publicos;
- endpoints e services Acessorias;
- testes de solicitacoes, auth, colaboradores, Home Acessorias e responsaveis.

### portal-sama-web

- Ha alteracoes locais de implementacao/testes ainda nao commitadas:
- navegacao/rotas de clientes;
- header de notificacoes;
- pagina Clientes;
- pagina Colaboradores;
- painel DEV;
- Configuracoes;
- testes Playwright/contratos.

### portal-sama-docs

- Alteracoes documentais ainda nao commitadas:
- `M AUDITORIA-DEPLOY-19-EXECUCAO-HOMOLOGACAO-EASYPANEL.md`
- `M CONTEXTO-CODEX-ATUAL.md`

## Pendencias principais

1. Aplicar a migration `20260611130000_add_access_request_pending_dev` no banco alvo/EasyPanel. Ela ja foi validada em MySQL Docker local controlado.
2. Fazer deploy controlado com as alteracoes locais de API/Web.
3. Executar smoke EasyPanel autenticado com credenciais atuais:
   - login DEV;
   - Home DEV/Gestor com dados reais;
   - painel DEV `Integrar` e `Aplicar vencimentos`;
   - Clientes `scope=all`/`scope=mine`;
   - Colaboradores publico interno;
   - Notificacoes `Ler todas`/`Confirmar alertas`;
   - Configuracoes salvando preferencias e recarregando.
4. Revisar se a conciliacao nominal por nome + departamento precisa de regra adicional alem dos matches exatos/alias e da aplicacao local por `responsibleUserId`.
5. Para smoke autenticado, usar somente credenciais atuais de homologacao; a credencial administrativa antiga foi rotacionada e nao deve ser reutilizada.

## Veredito atual

`Pronto para homologacao controlada`.

As correcoes do TXT ficaram implementadas e verdes em lint/build/test/E2E locais.
Ainda falta aplicar migration/deploy no ambiente alvo e executar smoke EasyPanel autenticado com dados reais antes de liberar usuarios reais.

## Cuidados

- Nao reverter mudancas locais.
- Nao expor nem ler valores de `.env`.
- A credencial administrativa informada em chat anterior foi rotacionada pelo usuario; nao salvar credenciais em docs/evidencias.
- Docker ja foi usado nesta etapa para validacao controlada; se usar novamente ou instalar algo em proximas etapas, registrar neste contexto e na auditoria 19.
