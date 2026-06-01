# [PARCIAL] Integracao Acessorias - paths reais, importacao DEV e EasyPanel

Ultima atualizacao: 2026-06-01
Responsavel/IA: Codex
Status: base local corrigida; validacao real no EasyPanel pendente

## Objetivo

Registrar a correcao da integracao com a API do Sistema Acessorias para a Fase 2 e a Fase 3 de `ORDEM_IMPLEMENTACAO_DOCUMENTACOES.md`.

Este documento valida o arquivo recebido em `Downloads/integracao_acessorias_paths_importacao_dev.md`, confronta o conteudo com a documentacao oficial em `https://api.acessorias.com/documentation` e define como a configuracao deve ficar nos workspaces `portal-sama-docs`, `portal-sama-api` e `portal-sama-web`.

## Contrato oficial confirmado

- Autenticacao: `Authorization: Bearer <token>` no backend.
- Rate limit oficial: 100 requisicoes por minuto.
- Empresas: `GET /companies/{Identificador}` aceita `ListAll` para listar empresas paginadas.
- Empresas paginadas: usar `companies/ListAll` com `Pagina`; buscar paginas seguintes ate retorno vazio.
- Parametros uteis de empresas: `obligations`, `departments`, `stateRegistrations`, `registrationData`, `contacts`.
- Entregas: `GET /deliveries/{Identificador}` aceita `ListAll` para entregas de todas as empresas.
- Entregas `ListAll`: exige `DtInitial`, `DtFinal` e `DtLastDH`; quando `Identificador=ListAll`, `DtLastDH` aceita somente dia atual e/ou anterior.
- Parametros uteis de entregas: `situation=pending,delivered,read`, `department_id`, `Pagina`, `attachments`, `attachmentsId`, `config`.
- `config` em entregas retorna `EntID`, tipo, ID da obrigacao/tarefa, departamento e responsaveis.
- `departments/ListAll` lista departamentos. Nao e endpoint de colaboradores ativos.
- `requests/ListAll` lista solicitacoes. Nao deve alimentar a Home operacional de entregas/vencimentos.

## EasyPanel - variaveis da API

Configurar somente no servico `portal-sama-api`:

```env
ACESSORIAS_BASE_URL=https://api.acessorias.com
ACESSORIAS_TOKEN=<secret do Acessorias>
ACESSORIAS_HOME_PATH=deliveries/ListAll
ACESSORIAS_DELIVERIES_PATH=deliveries/ListAll
ACESSORIAS_CLIENTS_PATH=companies/ListAll
ACESSORIAS_COLLABORATORS_PATH=
ACESSORIAS_AUTH_HEADER=Authorization
ACESSORIAS_AUTH_SCHEME=Bearer
ACESSORIAS_TIMEOUT_SEC=15
ACESSORIAS_HOME_APPEND_CONTEXT_QUERY=false
```

Observacoes:

- `ACESSORIAS_COLLABORATORS_PATH` deve ficar vazio ate o Acessorias confirmar endpoint oficial de usuarios/colaboradores.
- Enquanto nao houver endpoint oficial, a API do Portal Sama extrai responsaveis a partir de `companies/ListAll?departments`.
- O token nunca deve existir no `portal-sama-web` como `VITE_*`.

## Estado verificado da `.env` local da API

Arquivo verificado: `portal-sama-api/.env`.

Resultado para Acessorias:

- `ACESSORIAS_BASE_URL`: configurado para a API oficial.
- `ACESSORIAS_TOKEN`: configurado, sem reproduzir o valor aqui.
- `ACESSORIAS_HOME_PATH`: corrigido para `deliveries/ListAll`.
- `ACESSORIAS_DELIVERIES_PATH`: correto como `deliveries/ListAll`.
- `ACESSORIAS_CLIENTS_PATH`: correto como `companies/ListAll`.
- `ACESSORIAS_COLLABORATORS_PATH`: vazio, correto ate confirmacao oficial.
- `ACESSORIAS_AUTH_HEADER`: `Authorization`.
- `ACESSORIAS_AUTH_SCHEME`: `Bearer`.
- `ACESSORIAS_TIMEOUT_SEC`: `15`.
- `ACESSORIAS_HOME_APPEND_CONTEXT_QUERY`: `false`.

Ajustes tambem aplicados para EasyPanel:

- `COOKIE_SECURE` efetivo corrigido para `true`.
- paths efetivos de storage corrigidos para `/var/private/portal-sama`, `/var/private/portal-sama/certificates` e `/var/private/portal-sama/contracts`.

Pendencia de higiene da `.env`:

- O arquivo ainda possui chaves duplicadas historicas. Como o parser usa o ultimo valor, a configuracao efetiva esta correta para os pontos acima, mas a duplicidade deve ser limpa em uma rodada separada para reduzir risco operacional.

## Backend aplicado em `portal-sama-api`

Arquivos principais:

- `src/modules/integrations/acessorias/acessorias-home.service.ts`
- `src/modules/integrations/acessorias/acessorias-deliveries.service.ts`
- `src/modules/integrations/acessorias/acessorias-registrations.service.ts`
- `.env.example`

Comportamento atualizado:

- Home e sincronizacao de entregas usam `deliveries/ListAll` como padrao.
- O backend monta `DtInitial`, `DtFinal`, `DtLastDH`, `situation`, `config` e `Pagina`.
- O backend pagina `ListAll` ate retorno vazio, com limite interno conservador.
- O retorno real `empresa -> Entregas[]` e achatado em entregas individuais.
- Entregas usam `Config.EntID` como `externalId` preferencial.
- Campos reais como `Razao`, `Identificador`, `Nome`, `EntCompetencia`, `EntDtPrazo`, `EntDtEntrega`, `Status`, `Config.DptoNome`, `Config.RespPrazo` e `Config.RespEntrega` sao normalizados.
- `0000-00-00` e tratado como data vazia.
- `companies/ListAll` recebe parametros de enriquecimento e paginacao.
- Clientes leem campos reais como `ID`, `Identificador`, `Razao`, `Fantasia`, `Status`, `Telefone`, `UF`, `Regime`, `GrupoDeEmpresas`, `Departamentos` e contatos.
- Status `Ativa`, `Inativa` e `Bloqueada` sao preservados como `ACTIVE`, `INACTIVE` e `BLOCKED`.
- Se `ACESSORIAS_COLLABORATORS_PATH` estiver vazio, a importacao de responsaveis usa `Departamentos[].RespNome` e `Departamentos[].RespEmail` vindos das empresas.
- Responsaveis extraidos de departamentos entram como `INACTIVE` por seguranca e com aviso de que nao representam listagem completa de colaboradores ativos.

## Frontend aplicado em `portal-sama-web`

Arquivo principal:

- `.env.example`

Regra declarada:

- O frontend nao deve receber token, base URL privada ou qualquer `VITE_ACESSORIAS_*`.
- O Web consome apenas endpoints autenticados da propria API do Portal Sama.

## Relacao com Fase 2

Fase 2 passa a ter como criterio tecnico local:

1. EasyPanel com paths acima configurados no servico da API.
2. `home-summary` usando `deliveries/ListAll` com query params obrigatorios.
3. Preview/sync de empresas usando `companies/ListAll` paginado.
4. Responsaveis importados somente a partir de departamentos das empresas enquanto nao houver endpoint oficial.
5. Validacao real no EasyPanel com dados reais antes de marcar `[CONCLUIDO]`.

## Relacao com Fase 3

Fase 3 depende das entregas persistidas localmente ja normalizadas:

1. `external_id` deve vir preferencialmente de `Config.EntID`.
2. `DELIVERED` deve vir de status/data de entrega real.
3. Planilhas departamentais so podem receber baixa automatica quando houver cliente, competencia, mapeamento confirmado e confianca suficiente.
4. Divergencias continuam sendo o caminho correto para casos sem seguranca.

## Validacao local executada

```txt
npm.cmd test -- integrations/acessorias/acessorias-home.service.spec.ts integrations/acessorias/acessorias-deliveries.service.spec.ts integrations/acessorias/acessorias-registrations.service.spec.ts --runInBand
npm.cmd run build
npm.cmd run lint
git diff --check
```

Resultado: 3 suites passaram, 14 testes passaram; build/lint da API passaram; `git diff --check` passou em API/Web/Docs com aviso apenas de normalizacao CRLF/LF em Docs.

## Proximos passos no EasyPanel

1. Conferir as variaveis do servico `portal-sama-api` sem imprimir token.
2. Redeploy da API.
3. Executar `GET /api-v2/integrations/acessorias/home-summary` com usuario real.
4. Executar preview de empresas e responsaveis.
5. Executar sincronizacao de entregas em janela controlada.
6. Conferir `acessorias_deliveries`, auditoria e mapeamentos antes de aplicar em planilhas.
7. Registrar evidencia sanitizada em `RELATORIO_TESTES.md`.
