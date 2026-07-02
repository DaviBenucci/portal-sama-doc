# Continuidade Fase 12 - Telas de modulos apos painel do cliente

Atualizado em: 2026-07-02
Ambiente alvo: `https://portal.samacontabil.com.br`
Status formal: Fase 12 `EM_EXECUCAO`
Etapa atual: 12.5 - Refinar tela de contratos ZapSign

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
- ja existia backend Python funcional em `C:\Users\Sama Contabilidade\Downloads\Faturamento`; apos correcao pos-producao, o pacote oficial fica migrado para `portal-sama-api/services/faturamento`, com documentacao propria em `README.md` e `DOCUMENTACAO_FLUXO_INTEGRADO.md`;
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
- `portal-sama-api/src/modules/accounting/accounting-faturamento.service.ts` encapsula a chamada ao backend Python `processar_faturamento.py` em processo servidor, usando `ACESSORIAS_TOKEN` do `.env` principal da API e repassando `API_TOKEN` apenas por compatibilidade interna do script;
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

Correcao pos-producao da 12.1.1 em 2026-07-01:

- causa identificada: a API tinha fallback local para `C:\Users\Sama Contabilidade\Downloads\Faturamento`, caminho inexistente no deploy;
- migrado o pacote Python oficial para `portal-sama-api/services/faturamento`, incluindo `script/processar_faturamento.py`, `script/acessorias_client.py`, `script/livro_caixa.py`, `modelo/modelo.xlsx`, `modelo/modelo csv.csv`, `requirements.txt`, `README.md` e `DOCUMENTACAO_FLUXO_INTEGRADO.md`;
- removida a necessidade de `.env` separado no pacote de Faturamento; o fluxo usa `ACESSORIAS_TOKEN` do `.env` principal do `portal-sama-api`;
- o adaptador da API injeta `ACESSORIAS_TOKEN` e tambem `API_TOKEN` no processo Python apenas por compatibilidade com o script original;
- `SAMA_FATURAMENTO_ROOT` e `SAMA_FATURAMENTO_SCRIPT_PATH` viraram apenas overrides opcionais; sem eles, o padrao e `services/faturamento`;
- adaptado o runtime do Portal: `services/faturamento` guarda apenas script/modelo/documentacao, enquanto PDFs, CSVs, planilhas e logs sao gerados em `STORAGE_PRIVATE_PATH/faturamento/jobs/<jobId>` ou em `SAMA_FATURAMENTO_WORK_DIR` quando configurado;
- cada job passou a receber subpastas privadas `entrada` e `saida`, evitando sobrescrita quando empresa/ano se repetem;
- apos o processamento bem-sucedido, os PDFs temporarios baixados do Acessorias sao removidos da pasta `entrada`, mantendo disponiveis apenas CSV Dominio e Livro Caixa XLSX para download autenticado;
- o frontend de Faturamento passou a exibir botoes `Baixar CSV Dominio` e `Baixar Livro Caixa`, consumindo os `download_url` autenticados retornados pela API;
- quando nenhum PDF e baixado, o erro agora inclui os ultimos retornos do Acessorias por competencia, em vez de mostrar somente a pasta vazia;
- analisados os scripts legados abertos no IDE (`ler_pdf.py`, `livro_caixa_final.py`, `livro_caixa_completo.py`): eles mantem caminhos fixos, CNPJ/periodo fixos ou erros de sintaxe/variaveis, por isso o Portal usa somente o fluxo oficial parametrizado;
- `Dockerfile` da API instala tambem `services/faturamento/requirements.txt`, garantindo `requests`, `pdfplumber` e `openpyxl` no container;
- `.dockerignore` e `.gitignore` passaram a ignorar `.venv`, `entrada`, `saida`, `saida_corrigida` e `__pycache__` do pacote.

Validacoes da correcao pos-producao:

```powershell
python services\faturamento\script\processar_faturamento.py --help
python services\faturamento\script\acessorias_client.py --help
python -m py_compile services\faturamento\script\processar_faturamento.py services\faturamento\script\acessorias_client.py services\faturamento\script\livro_caixa.py
python services\faturamento\script\processar_faturamento.py --ano 2025 --codigo-empresa 366 --codigo-dominio 179 --mes 12 --sem-download --modelo services\faturamento\modelo\modelo.xlsx --entrada .ai-tests\faturamento-portal-adapter\entrada --saida .ai-tests\faturamento-portal-adapter\saida --apagar-pdfs-baixados
npm.cmd run build # API
npm.cmd run lint # API
npm.cmd test -- accounting --runInBand # API
npx.cmd tsc --noEmit --pretty false # Web
npm.cmd run lint # Web
npm.cmd test -- --runInBand # Web
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "starts faturamento" --reporter=line # Web
npm.cmd run build # Web
git diff --check # API, Web e Docs
```

Resultado:

```txt
Help dos scripts OK, exibindo ACESSORIAS_TOKEN/API_TOKEN
py_compile OK
Processamento local sem API OK, gerando CSV, planilha e log em .ai-tests
API build OK
API lint OK
Accounting tests OK, 18 passed
TypeScript web OK
Web lint OK
Web contract tests OK, 24 passed
Playwright Faturamento OK, 1 passed
Web build OK
git diff --check OK na API, Web e Docs
```

Pendencias operacionais para publicar/homologar fora da maquina local:

- configurar `ACESSORIAS_TOKEN` somente no backend API;
- configurar `SAMA_FATURAMENTO_ROOT`, `SAMA_FATURAMENTO_SCRIPT_PATH`, `SAMA_FATURAMENTO_WORK_DIR` ou `SAMA_FATURAMENTO_PYTHON_BIN` apenas se for necessario sobrescrever o pacote interno/runtime padrao;
- ajustar `SAMA_FATURAMENTO_TIMEOUT_SEC` se o processamento real precisar de mais tempo;
- publicar/rebuildar Web e API juntos, pois a 12.1.1 altera contrato entre ambos.

## 5. Etapa 12.2 - Refinar tela de Documentos

Status: `CONCLUIDA` em 2026-07-02.

Arquivos alterados:

```txt
portal-sama-web/src/pages/documents/DocumentsPage.tsx
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- filtros da tela `/documentos` passaram a ser refletidos na URL: `clientId`, `type`, `department`, `status` e `skip`;
- a tela preserva `clientId` recebido pela URL e preenche o cliente inicial do envio documental;
- o bloco `Documentos privados` exibe filtros ativos e desabilita `Limpar` quando nao ha filtro aplicado;
- a tabela ganhou atalho textual `Painel`, levando direto ao painel do cliente na aba `Documentos` por `/clientes/{id}/painel?tab=documentos#client-documents`;
- os cards de pendencias obrigatorias tambem ganharam atalho para o painel documental do respectivo cliente;
- a navegacao para o painel preserva `state.from` com `/documentos` e a query atual;
- o contrato web da Etapa 12.2 foi adicionado ao runner de contratos;
- o E2E cobre filtros persistidos em URL, parametros enviados para a API, paginacao por `skip`, limpeza de filtros e abertura do painel documental.

Validacoes executadas em `portal-sama-web`:

```powershell
npx.cmd tsc --noEmit --pretty false
npm.cmd test -- --runInBand
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "documents page" --reporter=line
npm.cmd run lint
npm.cmd run build
npm.cmd run test:e2e
git diff --check
```

Resultado:

```txt
TypeScript OK
25 web contract tests passed
Playwright Documentos OK, 4 passed
lint OK
build OK
test:e2e OK, 39 passed, 2 skipped
git diff --check OK no web
```

## 6. Etapa 12.3 - Refinar tela de Certificados

Status: `CONCLUIDA` em 2026-07-02.

Arquivos alterados:

```txt
portal-sama-web/src/pages/certificates/CertificatesPage.tsx
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- filtros da tela `/certificados-digitais` passaram a ser refletidos na URL: `search`, `clientId`, `department`, `deleted` e `skip`;
- a lista usa estado local com `filtersRef` para preservar alteracoes rapidas nos filtros sem perder parametros da URL;
- o cadastro global preenche o `ID do cliente` quando a URL recebe `clientId`;
- a tela ganhou visao global de vencimentos e riscos, com cards de `Vencendo`, `Vencidos` e `Sem validade`;
- os alertas `Atencao em certificados` e `Proximos vencimentos` foram alinhados com a aba de Certificados do painel do cliente;
- os formularios globais de cadastro e edicao passaram a expor `Departamento` e `Validade`, usando os campos ja suportados pelo schema e pelo service;
- a tabela global passou a exibir status de validade, texto operacional de prazo, senha `Protegida`/`Pendente` e arquivo;
- cada certificado com cliente vinculado ganhou atalho textual `Painel`, levando direto a `/clientes/{id}/painel?tab=certificados#client-certificates`;
- a navegacao para o painel preserva `state.from` com `/certificados-digitais` e a query atual;
- o contrato web da Etapa 12.3 foi adicionado ao runner de contratos;
- o E2E cobre filtros persistidos em URL, parametros enviados para a API, riscos de vencimento, paginacao por `skip`, limpeza de filtros e abertura do painel de certificados do cliente.

Validacoes executadas em `portal-sama-web`:

```powershell
npx.cmd tsc --noEmit --pretty false
npm.cmd run lint
npm.cmd test -- --runInBand
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "certificates page" --reporter=line
npm.cmd run build
npm.cmd run test:e2e
git diff --check
```

Resultado:

```txt
TypeScript OK
lint OK
26 web contract tests passed
Playwright Certificados OK, 1 passed
build OK
test:e2e OK, 40 passed, 2 skipped
git diff --check OK no web
```

## 7. Etapa 12.4 - Refinar Legalizacao

Status: `CONCLUIDA` em 2026-07-02.

Arquivos alterados:

```txt
portal-sama-web/src/pages/legalization/LegalizationPage.tsx
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- filtros da tela `/legalizacao` passaram a ser refletidos na URL: `search`, `status`, `stage`, `cnpj`, `type`, `protocol`, `department`, `archived` e `skip`;
- a lista usa estado local com `filtersRef` para preservar alteracoes rapidas nos filtros sem perder parametros da URL;
- a tela ganhou chips de filtros ativos, botao `Limpar`, checkbox `Arquivados` com `id` explicito e filtro de tipo de processo;
- o departamento vindo pela URL e preservado como opcao selecionada antes do catalogo de departamentos terminar de carregar;
- o detalhe do processo ganhou o painel `Status do fluxo`, exibindo Processo, Proposta e Contrato;
- quando ha proposta/contrato vinculados, a tela consulta `getProposal` e `getContract` para mostrar os status reais;
- links para propostas e contratos preservam `state.from` com `/legalizacao` e a query atual;
- o helper E2E de sessao passou a mockar `/api-v2/auth/me`, evitando logout falso em navegacoes com query string;
- o E2E cobre filtros persistidos em URL, parametros enviados para a API, paginacao por `skip`, limpeza, detalhe, painel de fluxo, atualizacao de status e navegacao para contrato;
- o contrato web da Etapa 12.4 foi adicionado ao runner de contratos.

Validacoes executadas em `portal-sama-web`:

```powershell
npx.cmd tsc --noEmit --pretty false
npm.cmd run lint
npm.cmd test -- --runInBand
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "legalization page" --reporter=line
npm.cmd run build
npm.cmd run test:e2e
git diff --check
```

Resultado:

```txt
TypeScript OK
lint OK
27 web contract tests passed
Playwright Legalizacao OK, 1 passed
build OK
test:e2e OK, 41 passed, 2 skipped
git diff --check OK no web
```

## 8. Pendencias da Fase 12

Ainda pendentes conforme `20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md`:

1. Etapa 12.5 - Refinar tela de contratos ZapSign.
2. Etapa 12.6 - Refinar Onboarding.
3. Etapa 12.7 - Refinar T.I./Acessos.
4. Etapa 12.8 - Refinar Gestao.
5. Etapa 12.9 - Refinar Administracao.

## 9. Acao em producao

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

Para a Etapa 12.2, quando a build for publicada no EasyPanel:

1. Conferir que o deploy do `portal-sama-web` usa o commit/build que contem esta etapa.
2. Rebuildar/redeployar apenas o servico web; nao ha migration, env nova ou alteracao obrigatoria na API para a Etapa 12.2.
3. Entrar com usuario que tenha `documents.read`, `documents.upload`, `documents.review`, `documents.download`, `documents.public_tokens` e `documents.requirements`, conforme o fluxo a validar.
4. Abrir `/documentos`, filtrar por cliente, tipo, departamento e status, confirmando que a URL reflete os filtros.
5. Usar `Proxima` e confirmar `skip=50` na URL quando houver mais de uma pagina.
6. Usar `Limpar` e confirmar que `clientId`, `type`, `department`, `status` e `skip` saem da URL.
7. Abrir um documento pelo botao `Painel` e confirmar `/clientes/{id}/painel?tab=documentos#client-documents`.
8. Validar que links publicos, revisao, download e pendencias obrigatorias seguem operacionais.

Para a Etapa 12.3, quando a build for publicada no EasyPanel:

1. Conferir que o deploy do `portal-sama-web` usa o commit/build que contem esta etapa.
2. Rebuildar/redeployar apenas o servico web; nao ha migration, env nova ou alteracao obrigatoria na API para a Etapa 12.3.
3. Entrar com usuario que tenha `certificates.read`, `certificates.manage` e `certificates.download`, conforme o fluxo a validar.
4. Abrir `/certificados-digitais`, filtrar por busca, cliente, departamento e `Removidos`, confirmando que a URL reflete os filtros.
5. Usar `Proxima` e confirmar `skip=50` na URL quando houver mais de uma pagina.
6. Confirmar que `Atencao em certificados` e `Proximos vencimentos` aparecem coerentes com certificados vencendo, vencidos ou sem validade.
7. Usar `Limpar` e confirmar que `search`, `clientId`, `department`, `deleted` e `skip` saem da URL.
8. Abrir um certificado pelo botao `Painel` e confirmar `/clientes/{id}/painel?tab=certificados#client-certificates`.
9. Validar que cadastro, edicao de validade/departamento, download, rotacao de senha e remocao seguem operacionais.

Para a Etapa 12.4, quando a build for publicada no EasyPanel:

1. Conferir que o deploy do `portal-sama-web` usa o commit/build que contem esta etapa.
2. Rebuildar/redeployar apenas o servico web; nao ha migration, env nova ou alteracao obrigatoria na API para a Etapa 12.4.
3. Entrar com usuario que tenha `legalization.read`, alem de `legalization.create`, `legalization.update`, `legalization.status`, `legalization.templates`, `proposals.*` e `contracts.*` conforme o fluxo a validar.
4. Abrir `/legalizacao`, filtrar por busca, status, etapa, tipo, CNPJ, protocolo, departamento e `Arquivados`, confirmando que a URL reflete os filtros.
5. Usar `Proxima` e confirmar `skip=50` na URL quando houver mais de uma pagina.
6. Abrir o detalhe de um processo e confirmar o painel `Status do fluxo` com Processo, Proposta e Contrato.
7. Confirmar que proposta e contrato vinculados exibem os status reais e abrem os respectivos detalhes.
8. Atualizar o status do processo e confirmar a atualizacao do status/timeline.
9. Usar `Limpar` e confirmar que filtros e `skip` saem da URL.

## 10. Proximo passo recomendado

Retomar a Etapa 12.5 - refinar tela de contratos ZapSign.
