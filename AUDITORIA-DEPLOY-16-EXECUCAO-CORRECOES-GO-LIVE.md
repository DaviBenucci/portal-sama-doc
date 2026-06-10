# AUDITORIA DEPLOY 16 - Execucao das correcoes Go-Live

Data: 10/06/2026

Escopo: registro incremental da execucao das fases planejadas em
`AUDITORIA-DEPLOY-15-PREPARACAO-CORRECOES-GO-LIVE.md`.

## Ferramentas locais usadas nesta rodada

- Docker: nao foi necessario nesta etapa.
- Instalacoes/execucoes auxiliares: foi usado `npx.cmd prettier@3.8.4` de forma transitoria
  para formatacao. O primeiro uso no frontend baixou o pacote via `npx`; nao houve alteracao
  intencional de `package.json` ou `package-lock.json`.

## Fase 0 - Preparacao operacional

Status: concluida nesta rodada.

Evidencias preservadas nesta rodada:

- `evidencias/auditoria-deploy-15/validation-api-results.json`
- `evidencias/auditoria-deploy-15/validation-web-results.json`
- `evidencias/auditoria-deploy-15/acessorias-api-results-clean.json`

Branch de trabalho criada nos tres repositorios:

- `portal-sama-docs`: `fix/deploy-readiness-go-live`
- `portal-sama-api`: `fix/deploy-readiness-go-live`
- `portal-sama-web`: `fix/deploy-readiness-go-live`

## Decisao operacional para Fase 3

Enquanto nao houver decisao formal de produto dando leitura ampla a outros perfis, a politica
de `clients.read` deve seguir o principio de menor privilegio:

| Perfil | Decisao aplicada nesta correcao |
|---|---|
| `ADMIN` / `DEV` | leitura ampla permitida |
| `MANAGER` | limitado a carteira/departamento/vinculo gerenciado |
| `AUDITOR` | sem leitura ampla automatica de clientes |
| `CLIENT` | somente o proprio cliente/vinculo |
| `DEPARTMENT` | somente clientes vinculados ao proprio departamento ou responsabilidade |
| `ACCOUNTING` | somente clientes vinculados ao escopo contabil/departamental |
| `LEGALIZATION` | somente clientes vinculados ao escopo de legalizacao/departamental |
| `TI` | sem leitura ampla automatica de clientes |

Observacao: se o negocio decidir que `MANAGER`, `AUDITOR` ou `TI` precisa de leitura global,
a decisao deve virar permissao/regra explicita e testada, nao um efeito colateral de
`clients.read`.

## Fase 3 - Corrigir clients.read e escopo de clientes

Status: implementada com validacao focada concluida. Gate E2E geral da API ainda bloqueado
pela falha conhecida da Fase 5 em Web Push/CSRF.

Arquivos alterados:

- `portal-sama-api/src/modules/clients/clients.service.ts`
- `portal-sama-api/src/modules/clients/clients.service.spec.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`

Mudancas aplicadas:

- Removido o atalho que permitia leitura ampla apenas por `clients.read`.
- `ADMIN` e `DEV` continuam com leitura ampla.
- `MANAGER`, `DEPARTMENT`, `ACCOUNTING`, `LEGALIZATION`, `TI` e perfis customizados com
  `clients.read` passam a depender de:
  - responsabilidade direta (`responsibleUserId`);
  - gestao direta (`managerUserId`);
  - departamento ativo permitido ao usuario.
- `CLIENT` passa a receber `clients.read` no RBAC padrao, mas o servico limita a leitura ao
  proprio cliente por `id`, `username` ou CNPJ derivado do `username`.
- `list()`, `getById()` e `getDashboard()` passam pela mesma politica de leitura.

Testes adicionados/ajustados:

- lista de clientes de usuario nao privilegiado recebe filtro por `ClientDepartmentAssignment`
  ativo;
- usuario `CLIENT` acessa somente o proprio cliente por CNPJ;
- usuario com `clients.read` fora do escopo recebe `ForbiddenException`;
- `MANAGER` sem vinculo nao tem leitura ampla;
- responsavel direto consegue ler cliente atribuido;
- RBAC padrao continua consistente apos incluir `clients.read` em `CLIENT`.

Validacoes executadas:

| Comando | Resultado |
|---|---|
| `npm.cmd test -- --runInBand src/modules/clients/clients.service.spec.ts src/modules/rbac/default-rbac.spec.ts` | OK: 2 suites, 13 testes |
| `npm.cmd run lint` | OK |
| `npm.cmd run build` | OK |
| `git diff --check` | OK |
| `npm.cmd run test:e2e` | Falhou: 134 passaram, 2 falharam em Web Push/CSRF por `403` esperado `200` |

Leitura:

- A correcao de escopo de clientes esta coberta por testes focados e nao apresentou falha em
  lint/build.
- O gate E2E API ainda permanece vermelho pelo bloqueio `QA-API-01`, que sera tratado na Fase 5.

## Fase 4 - Corrigir escalada de perfil em Acessorias Home

Status: implementada com validacao focada concluida.

Arquivos alterados:

- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.controller.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.service.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.service.spec.ts`
- `portal-sama-web/src/services/acessorias.service.ts`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `portal-sama-web/src/pages/dev/DevAdminPage.tsx`

Mudancas aplicadas:

- O backend passa a derivar o perfil efetivo da Home pelo usuario autenticado.
- `profile` recebido por query continua aceito apenas por compatibilidade, mas nao amplia
  autorizacao.
- Perfil efetivo maximo:
  - `ADMIN` / `DEV`: `admin`, com visao global.
  - `MANAGER`/gestao equivalente: `manager`, com escopo por departamento permitido ou
    responsabilidade direta.
  - demais usuarios: `collaborator`, com escopo por responsabilidade direta e, quando houver
    departamento, itens sem responsavel apenas dentro do departamento.
- `profile=admin` para usuarios nao privilegiados sofre downgrade automatico para o perfil
  permitido.
- Usuarios sem departamento ou responsabilidade direta nao caem mais em visao global por efeito
  colateral.
- Frontend deixou de enviar `profile` para `/integrations/acessorias/home-summary`.
- A tela DEV tambem deixou de chamar `getAcessoriasHomeSummary('admin')`; o backend decide a visao
  com base no token.

Testes adicionados/ajustados:

- usuario departamental chamando `profile=admin` recebe `profile=collaborator` e nao recebe outro
  departamento;
- gestor sem departamento chamando `profile=admin` recebe `profile=manager`, sem visao global;
- `ADMIN` continua recebendo `profile=admin` e dados globais.

Validacoes executadas:

| Comando | Resultado |
|---|---|
| `npm.cmd test -- --runInBand src/modules/integrations/acessorias/acessorias-home.service.spec.ts` | OK: 1 suite, 6 testes |
| `npm.cmd test -- --runInBand src/modules/integrations/acessorias/acessorias-registrations.service.spec.ts src/modules/integrations/acessorias/acessorias-deliveries.service.spec.ts` | OK: 2 suites, 16 testes |
| `npm.cmd test -- --runInBand src/modules/clients/clients.service.spec.ts src/modules/rbac/default-rbac.spec.ts src/modules/integrations/acessorias/acessorias-home.service.spec.ts src/modules/integrations/acessorias/acessorias-registrations.service.spec.ts src/modules/integrations/acessorias/acessorias-deliveries.service.spec.ts` | OK: 5 suites, 35 testes |
| `npm.cmd run lint` em `portal-sama-api` | OK |
| `npm.cmd run build` em `portal-sama-api` | OK |
| `npm.cmd run lint` em `portal-sama-web` | OK |
| `npm.cmd run build` em `portal-sama-web` | OK |
| `npm.cmd test -- --runInBand` em `portal-sama-web` | OK: 9 contratos |
| `git diff --check` em `portal-sama-api` e `portal-sama-web` | OK |

Leitura:

- O achado `AUTHZ-03` esta corrigido no codigo local: alterar `profile` na URL nao concede visao
  maior.
- Ainda falta executar uma bateria Acessorias real/local completa apos as fases seguintes, pois a
  release ainda depende de E2E API/Web, artefatos limpos, audit `qs` e provas operacionais.

## Fase 5 - Corrigir E2E API Web Push/CSRF

Status: implementada e validada. Gate E2E API esta verde no codigo local.

Arquivo alterado:

- `portal-sama-api/test/app.e2e-spec.ts`

Mudanca aplicada:

- O setup E2E agora define explicitamente variaveis de cookie/CSRF antes de importar o
  `AppModule`:
  - `CSRF_SECRET=test_csrf_secret_with_at_least_32_chars`
  - `COOKIE_DOMAIN=''`
  - `COOKIE_SECURE=false`
  - `COOKIE_SAME_SITE=lax`
- Isso impede que a suite E2E herde `COOKIE_DOMAIN`, `COOKIE_SECURE`, `COOKIE_SAME_SITE` ou
  segredo CSRF do `.env` local/real.
- O comportamento de producao nao foi alterado; a mudanca fica restrita ao ambiente de teste.

Validacoes executadas:

| Comando | Resultado |
|---|---|
| `npm.cmd run test:e2e` | OK: 1 suite, 136 testes |
| `npm.cmd run lint` | OK |
| `npm.cmd run build` | OK |
| `npm.cmd test -- --runInBand` | OK: 44 suites, 272 testes |
| `git diff --check` em `portal-sama-api` | OK |

Leitura:

- Os dois testes que falhavam em Web Push/CSRF agora passam.
- Os testes negativos de CSRF continuam passando com `403` quando nao ha token/cookie valido.
- O bloqueio `QA-API-01` esta corrigido no codigo local.
