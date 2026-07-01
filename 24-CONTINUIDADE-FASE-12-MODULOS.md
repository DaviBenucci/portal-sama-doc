# Continuidade Fase 12 - Telas de modulos apos painel do cliente

Atualizado em: 2026-07-01
Ambiente alvo: `https://portal.samacontabil.com.br`
Status formal: Fase 12 `EM_EXECUCAO`
Etapa atual: 12.2 - Refinar tela de Documentos

## 1. Entrada da fase

A Fase 12 foi iniciada depois da Fase 11 ficar formalmente `CONCLUIDA`.

Evidencia de entrada:

```txt
23-CONTINUIDADE-FASE-11-PAINEL-CLIENTE.md
Fase 11 CONCLUIDA
lint OK
22 web contract tests passed
build OK
test:e2e OK, 36 passed, 2 skipped
git diff --check OK
```

Escopo da Fase 12 conforme o guia raiz:

```txt
Fase 12 - Telas de modulos apos painel do cliente
```

## 2. Etapa 12.1 - Refinar tela de Clientes

Status: `CONCLUIDA`

Arquivos alterados:

```txt
portal-sama-web/src/pages/clients/ClientsPage.tsx
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- filtros da tela `/clientes` passaram a ser refletidos na URL: `search`, `status`, `cnpj`, `cidade`, `grupo`, `archived`, `scope` e `skip`;
- a lista usa estado local sincronizado com a URL para preservar preenchimentos rapidos sem perder parametros;
- o link de painel do cliente ficou textual, com icone e rotulo visivel `Painel`, mantendo `aria-label` especifico por cliente;
- o retorno do painel preserva os filtros da lista via `state.from.search`;
- filtro `Arquivados` recebeu `id` e `label` explicitos;
- foi adicionado botao `Limpar` para zerar filtros operacionais sem remover o escopo selecionado;
- status da listagem passou a ter fallback de label quando o backend nao enviar `status_label`;
- contrato web novo cobre a Etapa 12.1;
- E2E novo valida busca, filtro de status, cidade, grupo, arquivados, escopo, chamada de API, abertura do painel em um clique, retorno preservando filtros e limpeza.

## 3. Validacoes executadas apos a Etapa 12.1

Comandos executados em `portal-sama-web`:

```powershell
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "clients page keeps filters" --reporter=line
npm.cmd run lint
npm.cmd test -- --runInBand
npm.cmd run build
npm.cmd run test:e2e
git diff --check
```

Resultado:

```txt
Playwright focado OK, 1 passed
lint OK
23 web contract tests passed
build OK
test:e2e OK, 37 passed, 2 skipped
git diff --check OK no web e no docs
```

## 4. Sub-etapa urgente 12.1.1 - Integra-AI Extrato/Faturamento

Status: `CONCLUIDA` em 2026-07-01.

Motivo da urgencia:

- a rota `/contabil/integra-ai` hoje representa o fluxo de Extrato bancario para TXT Dominio;
- o escritorio tambem precisa iniciar o fluxo de Faturamento/PGDAS dentro do Portal;
- ja existe backend Python funcional em `C:\Users\Sama Contabilidade\Downloads\Faturamento`, com documentacao propria em `README.md` e `DOCUMENTACAO_FLUXO_INTEGRADO.md`;
- o escopo imediato e criar o frontend e a experiencia de entrada, sem reescrever a automacao Python.

Decisao de produto/UX:

1. Ao clicar para acessar Integra-AI, o usuario deve ver dois botoes principais: `Extrato` e `Faturamento`.
2. `Extrato` deve abrir o Integra-AI atual, preservando o comportamento existente.
3. `Faturamento` deve abrir novo frontend para selecionar empresa, ano, modo de processamento e codigo Dominio opcional.

Entradas obrigatorias do frontend de Faturamento:

- `codigo_empresa`: codigo/ID usado para localizar a empresa no Acessorias;
- `ano`: ano de competencia;
- modo de processamento: `Um mes` ou `Todos os meses`;
- `mes`: obrigatorio somente no modo `Um mes`;
- `codigo_dominio`: opcional, usado quando o codigo da empresa no Dominio for diferente do codigo do Acessorias.

Regras obrigatorias:

- se `codigo_dominio` estiver vazio, usar `codigo_empresa` no CSV Dominio;
- se `codigo_dominio` estiver preenchido, usar esse valor no CSV e manter `codigo_empresa` para a busca no Acessorias;
- a tela deve deixar explicito se o usuario vai gerar apenas um mes ou todos os meses;
- a tela nao deve expor `API_TOKEN`, `.env`, caminho local do backend, conteudo bruto de PDF ou logs sensiveis;
- o navegador deve chamar uma API autenticada/adaptador seguro, nao executar script Python local diretamente.

Validacao esperada:

- Playwright cobrindo a escolha inicial `Extrato`/`Faturamento`;
- teste garantindo que `Extrato` continua abrindo o fluxo atual;
- teste de Faturamento para campos obrigatorios, alternancia `Um mes`/`Todos os meses`, `codigo_dominio` opcional e estados de processamento mockados;
- `lint`, contratos web, `build`, `test:e2e` e `git diff --check`.

Execucao em 2026-07-01:

- `portal-sama-web/src/pages/accounting/IntegraAiPage.tsx` passou a abrir `/contabil/integra-ai` com escolha inicial `Extrato` e `Faturamento`;
- `Extrato` preserva o fluxo atual e so carrega workspace/jobs apos o clique no botao `Extrato`;
- `Faturamento` recebeu formulario com `codigo_empresa` obrigatorio para Acessorias, `ano`, modo `Todos os meses`/`Um mes`, `mes` condicional e `codigo_dominio` opcional;
- o frontend envia `codigo_empresa`, `ano`, `modo`, `mes` quando aplicavel e `codigo_dominio` somente quando preenchido para `POST /api-v2/accounting/faturamento/jobs`;
- `portal-sama-web/src/types/integra-ai.ts` e `portal-sama-web/src/services/integra-ai.service.ts` receberam o contrato TypeScript/HTTP de Faturamento;
- `portal-sama-web/tests/e2e/smoke.spec.ts` cobre a escolha inicial, preserva o fluxo de Extrato e valida Faturamento com codigo Acessorias `366`, codigo Dominio `179`, modo `um_mes` e mes `12`;
- `portal-sama-web/scripts/web-contract-tests.mjs` recebeu contrato da Etapa 12.1.1.
- `portal-sama-api/src/modules/accounting/accounting-faturamento.controller.ts` adicionou endpoints autenticados `POST /api-v2/accounting/faturamento/jobs`, `GET /api-v2/accounting/faturamento/jobs/:id` e `GET /api-v2/accounting/faturamento/jobs/:id/download`;
- `portal-sama-api/src/modules/accounting/accounting-faturamento.service.ts` encapsula a chamada ao backend Python `processar_faturamento.py` em processo servidor, usando `API_TOKEN` somente no ambiente do backend;
- o adaptador aceita `codigo_empresa` obrigatorio do Acessorias, `codigo_dominio` opcional, `ano`, modo `um_mes`/`todos_os_meses` e `mes` condicional;
- os artefatos CSV, planilha e log sao expostos apenas por download autenticado e com checagem de caminho dentro da raiz do Faturamento;
- `portal-sama-api/src/config/env.schema.ts` recebeu `SAMA_FATURAMENTO_ROOT`, `SAMA_FATURAMENTO_SCRIPT_PATH`, `SAMA_FATURAMENTO_PYTHON_BIN` e `SAMA_FATURAMENTO_TIMEOUT_SEC`;
- `portal-sama-api/src/modules/accounting/accounting.module.ts` registrou controller e service do Faturamento.

Validacoes executadas em `portal-sama-web`:

```powershell
npx.cmd tsc --noEmit --pretty false
npm.cmd run lint
npm.cmd test -- --runInBand
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "Integra-AI" --reporter=line
npm.cmd run build
npm.cmd run test:e2e
git diff --check
```

Resultado:

```txt
TypeScript OK
lint OK
24 web contract tests passed
Playwright Integra-AI OK, 4 passed
build OK
test:e2e OK, 38 passed, 2 skipped
git diff --check OK no web
```

Validacoes executadas em `portal-sama-api`:

```powershell
npm.cmd run build
npm.cmd run lint
npm.cmd test -- --runInBand src/modules/accounting/accounting.service.spec.ts src/modules/accounting/integra-ai.engine.spec.ts
git diff --check
$env:DATABASE_URL='mysql://portal_user:portal_password@localhost:3306/portal_sama'; npm.cmd run prisma:generate
$env:DATABASE_URL='mysql://portal_user:portal_password@localhost:3306/portal_sama'; npm.cmd run prisma:migrate:deploy
$env:DATABASE_URL='mysql://portal_user:portal_password@localhost:3306/portal_sama'; npm.cmd run prisma:seed
```

Resultado:

```txt
build OK
lint OK
Accounting tests OK, 18 passed
git diff --check OK na API
Prisma generate OK
Prisma migrate deploy OK em MySQL local Docker
Prisma seed OK
Backend local em start:dev OK, health 200 em http://127.0.0.1:3000/api-v2/health
```

Homologacao real local em 2026-07-01:

```txt
POST /api-v2/accounting/faturamento/jobs OK, HTTP 201
Payload homologado: codigo_empresa=366, codigo_dominio=179, ano=2026, modo=um_mes, mes=1
Job local atual: fat-366-2026-f1268d0f
Resultado: success, Processamento concluido com 12 linha(s) no CSV.
Artefatos retornados: CSV, planilha e log
GET /api-v2/accounting/faturamento/jobs/:id OK, HTTP 200
GET /api-v2/accounting/faturamento/jobs/:id/download?kind=csv OK, HTTP 200, 756 bytes
```

Ajuste adicional de validacao local:

- `portal-sama-api/src/modules/auth/auth.controller.ts` passou a omitir `COOKIE_DOMAIN` em `development` quando `CORS_ORIGIN`/`FRONTEND_URL` apontam para `localhost` ou `127.0.0.1`, permitindo login local sem conflito com dominio de producao no `.env`;
- API reiniciada com `NODE_ENV=development`, `CORS_ORIGIN=http://127.0.0.1:5174`, `FRONTEND_URL=http://127.0.0.1:5174`, `COOKIE_SECURE=false` e `DATABASE_URL` local;
- `GET /api-v2/auth/csrf` passou a retornar `Access-Control-Allow-Origin: http://127.0.0.1:5174` e cookie sem `Domain`/`Secure`;
- login local com cookie automatico validado com sucesso;
- `POST /api-v2/accounting/faturamento/jobs` e download CSV foram revalidados depois do restart da API;
- `npm.cmd run prisma:bootstrap-admin` criou o usuario local `admin (DEV)` no MySQL Docker, usando a senha configurada no ambiente local e sem imprimir segredo.

Pendencias operacionais para publicar/homologar fora da maquina local:

- configurar `SAMA_FATURAMENTO_ROOT`, `SAMA_FATURAMENTO_SCRIPT_PATH`/`SAMA_FATURAMENTO_PYTHON_BIN` quando necessario e timeout no servico API;
- garantir token do Acessorias somente no backend;
- publicar/rebuildar Web e API juntos, pois a 12.1.1 altera contrato entre ambos.

## 5. Pendencias da Fase 12

Ainda pendentes conforme `20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md`:

1. Etapa 12.2 - Refinar tela de Documentos.
2. Etapa 12.3 - Refinar tela de Certificados.
3. Etapa 12.4 - Refinar Legalizacao.
4. Etapa 12.5 - Refinar tela de contratos ZapSign.
5. Etapa 12.6 - Refinar Onboarding.
6. Etapa 12.7 - Refinar T.I./Acessos.

## 6. Acao em producao

Nenhuma acao manual em producao e obrigatoria para concluir a Etapa 12.1 no repositorio local.

Quando a build desta etapa for publicada no EasyPanel, executar:

1. Conferir que o deploy do `portal-sama-web` usa o commit/build que contem este documento e os arquivos listados acima.
2. Rebuildar/redeployar apenas o servico web; nao ha migration, env nova ou alteracao obrigatoria na API para a Etapa 12.1.
3. Entrar com usuario que tenha `clients.read` e abrir `/clientes`.
4. Preencher busca, status, cidade, grupo e `Arquivados`, confirmando que a URL reflete os filtros.
5. Abrir um cliente pelo botao `Painel` em uma unica acao e confirmar `/clientes/{id}/painel`.
6. Usar o botao voltar `Clientes` no header do painel e confirmar que os filtros da lista continuam preenchidos.
7. Usar `Limpar` e confirmar que filtros sao removidos, preservando o escopo `Todos permitidos`/`Ver meus clientes` quando aplicavel.

Para a sub-etapa 12.1.1, a acao em producao exige publicar Web e API juntos e configurar o token do Acessorias apenas no backend. A publicacao do frontend nao deve depender de token do Acessorias no navegador.

## 7. Proximo passo recomendado

Retomar a Etapa 12.2 - refinar tela de Documentos.
