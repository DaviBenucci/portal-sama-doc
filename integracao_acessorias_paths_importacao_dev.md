# [PARCIAL] Integracao Acessorias - paths reais, importacao DEV e EasyPanel

Ultima atualizacao: 2026-06-01
Responsavel/IA: Codex
Status: base local corrigida; botoes DEV, paginacao/rate limit, aplicacao em planilha, revisao de divergencias, vencimentos no workspace e Central de Vencimentos implementados localmente; validacao real no EasyPanel pendente

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
ACESSORIAS_TIMEOUT_SEC=30
ACESSORIAS_RATE_LIMIT_PER_MINUTE=95
ACESSORIAS_MAX_PAGES=1000
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
- `src/modules/integrations/acessorias/acessorias-deliveries.controller.ts`
- `src/modules/integrations/acessorias/acessorias-deliveries.service.ts`
- `src/modules/integrations/acessorias/acessorias-registrations.service.ts`
- `.env.example`

Comportamento atualizado:

- Home e sincronizacao de entregas usam `deliveries/ListAll` como padrao.
- O backend monta `DtInitial`, `DtFinal`, `DtLastDH`, `situation`, `config` e `Pagina`.
- O backend pagina `ListAll` ate retorno vazio, com limite configuravel por `ACESSORIAS_MAX_PAGES`.
- O backend aguarda entre paginas conforme `ACESSORIAS_RATE_LIMIT_PER_MINUTE`, com default 95/minuto para manter margem abaixo do limite oficial de 100/minuto.
- Respostas `429` entram em retry controlado, respeitando `Retry-After` quando informado.
- Sincronizacao de entregas registra `incremental_since` e usa o ultimo run `SUCCESS` como marco incremental para proximas sincronizacoes.
- O retorno real `empresa -> Entregas[]` e achatado em entregas individuais.
- Entregas usam `Config.EntID` como `externalId` preferencial.
- Campos reais como `Razao`, `Identificador`, `Nome`, `EntCompetencia`, `EntDtPrazo`, `EntDtEntrega`, `Status`, `Config.DptoNome`, `Config.RespPrazo` e `Config.RespEntrega` sao normalizados.
- `0000-00-00` e tratado como data vazia.
- `companies/ListAll` recebe parametros de enriquecimento e paginacao.
- Clientes leem campos reais como `ID`, `Identificador`, `Razao`, `Fantasia`, `Status`, `Telefone`, `UF`, `Regime`, `GrupoDeEmpresas`, `Departamentos` e contatos.
- Status `Ativa`, `Inativa` e `Bloqueada` sao preservados como `ACTIVE`, `INACTIVE` e `BLOCKED`.
- Se `ACESSORIAS_COLLABORATORS_PATH` estiver vazio, a importacao de responsaveis usa `Departamentos[].RespNome` e `Departamentos[].RespEmail` vindos das empresas.
- Responsaveis extraidos de departamentos entram como `INACTIVE` por seguranca e com aviso de que nao representam listagem completa de colaboradores ativos.
- `GET /api-v2/integrations/acessorias/deliveries/preview` consulta o Acessorias, normaliza a amostra e retorna resumo sem persistir dados.

## Frontend aplicado em `portal-sama-web`

Arquivos principais:

- `src/pages/dev/DevAdminPage.tsx`
- `src/services/acessorias.service.ts`
- `src/types/acessorias.ts`
- `.env.example`

Regras declaradas:

- O frontend nao deve receber token, base URL privada ou qualquer `VITE_ACESSORIAS_*`.
- O Web consome apenas endpoints autenticados da propria API do Portal Sama.
- As chamadas Acessorias no Web usam timeout estendido de 10 minutos para suportar importacoes manuais longas.
- A rota `/departamentos/vencimentos` consolida vencimentos de calendario e Acessorias do workspace; ela esta implementada localmente e ainda depende de validacao real.

## Onde ficam os botoes de adicionar/importar clientes

Na rota React `/dev`, arquivo `portal-sama-web/src/pages/dev/DevAdminPage.tsx`:

- O botao manual `Novo cliente` fica no cabecalho da area DEV e abre `/dev/clientes/novo`; ele depende da permissao `clients.create`.
- O painel `Integracao Acessorias` / `Importacao DEV` fica logo abaixo dos cards de resumo da area DEV.
- Nesse painel existem os botoes `Testar conexao`, `Previa entregas`, `Sincronizar entregas`, `Previa clientes`, `Importar clientes`, `Previa responsaveis`, `Importar responsaveis` e `Importar tudo`.
- `Previa clientes` usa `GET /api-v2/integrations/acessorias/registrations/preview?entity=clients`.
- `Importar clientes` usa `POST /api-v2/integrations/acessorias/registrations/sync` com `entity=clients`.
- `Colaboradores ativos` fica desabilitado ate o Acessorias confirmar endpoint oficial de usuarios/colaboradores; por enquanto a importacao segura e de responsaveis extraidos dos departamentos das empresas.

## Relacao com Fase 2

Fase 2 passa a ter como criterio tecnico local:

1. EasyPanel com paths acima configurados no servico da API.
2. `home-summary` usando `deliveries/ListAll` com query params obrigatorios.
3. Preview/sync de empresas usando `companies/ListAll` paginado pela area DEV.
4. Responsaveis importados somente a partir de departamentos das empresas enquanto nao houver endpoint oficial.
5. Preview de entregas sem persistencia pela area DEV.
6. Validacao real no EasyPanel com dados reais antes de marcar `[CONCLUIDO]`.

## Relacao com Fase 3

Fase 3 depende das entregas persistidas localmente ja normalizadas:

1. `external_id` deve vir preferencialmente de `Config.EntID`.
2. `DELIVERED` deve vir de status/data de entrega real.
3. Planilhas departamentais so podem receber baixa automatica quando houver cliente, competencia, mapeamento confirmado e confianca suficiente.
4. Divergencias continuam sendo o caminho correto para casos sem seguranca.

## Validacao local executada

```txt
# portal-sama-api
npm.cmd test -- integrations/acessorias/acessorias-home.service.spec.ts integrations/acessorias/acessorias-deliveries.service.spec.ts integrations/acessorias/acessorias-registrations.service.spec.ts --runInBand
npm.cmd test -- acessorias-deliveries.service.spec.ts acessorias-registrations.service.spec.ts acessorias-home.service.spec.ts departments.service.spec.ts --runInBand
npm.cmd run build
npm.cmd run lint

# portal-sama-web
npx.cmd tsc --noEmit --pretty false
npm.cmd run build
npm.cmd run lint

# repos alterados
git diff --check
```

Resultado consolidado local atualizado: 4 suites focadas de Acessorias/Departamentos passaram com 20 testes; TypeScript do Web passou; build/lint da API passaram; build/lint do Web passaram; `git diff --check` deve ser mantido como ultimo gate antes de commit.

## Proximos passos no EasyPanel

1. Conferir as variaveis do servico `portal-sama-api` sem imprimir token.
2. Redeploy da API.
3. Executar `GET /api-v2/integrations/acessorias/home-summary` com usuario real.
4. Executar preview de empresas e responsaveis.
5. Executar sincronizacao de entregas em janela controlada.
6. Conferir `acessorias_deliveries`, auditoria e mapeamentos antes de aplicar em planilhas.
7. Registrar evidencia sanitizada em `RELATORIO_TESTES.md`.

# Integração Acessórias — Correção dos endpoints, normalização dos dados e importação pela área DEV

## 1. Objetivo

Esta documentação descreve o problema atual da integração entre o **Portal Sama** e a **API do Sistema Acessórias**, explica por que a Home passou a consultar a API mas ainda não consegue exibir corretamente as entregas, e define o plano técnico para corrigir os endpoints, normalizar o retorno da API e adicionar na área **DEV** botões de importação de clientes e responsáveis/colaboradores.

O foco desta documentação é garantir que a integração seja implementada de forma segura, previsível e compatível com o contrato real da API do Acessórias.

---

## 2. Situação atual observada

Na tela Home do Portal Sama, o diagnóstico passou a indicar:

```txt
Configuração no backend: Configurado
Consulta da Home: Disponível
```

Isso mostra que o backend agora consegue executar a chamada HTTP para o Acessórias sem cair no erro anterior de `HTTP 404 - caminho /`.

Porém, a tela de atrasos e riscos ainda exibe informações incompletas, como:

```txt
Obrigacao nao informada
Cliente nao informado
Vencimento: -
Sem status
```

Esse comportamento indica que a chamada HTTP provavelmente está chegando na API, mas o backend do Portal Sama ainda não está interpretando corretamente o formato real do JSON retornado pelo Acessórias.

---

## 3. Diagnóstico do problema

O problema atual não é somente configuração de `.env`. Existem três pontos principais:

1. Alguns paths configurados ainda não representam exatamente o contrato real da API.
2. O backend precisa montar parâmetros obrigatórios dinamicamente, especialmente para entregas.
3. O backend precisa normalizar os campos reais retornados pelo Acessórias, pois eles não seguem o mesmo formato esperado atualmente pelo Portal Sama.

---

## 4. Configuração atual informada

Configuração aplicada atualmente:

```env
ACESSORIAS_HOME_PATH=requests
ACESSORIAS_DELIVERIES_PATH=deliveries
ACESSORIAS_CLIENTS_PATH=companies
ACESSORIAS_COLLABORATORS_PATH=departments
ACESSORIAS_AUTH_HEADER=Authorization
ACESSORIAS_AUTH_SCHEME=Bearer
ACESSORIAS_TIMEOUT_SEC=15
```

Essa configuração está parcialmente correta, mas precisa de ajustes.

---

## 5. Avaliação dos paths atuais

### 5.1 `ACESSORIAS_HOME_PATH=requests`

Esse path não deve ser usado para a Home operacional do Portal Sama.

O endpoint `requests` na API do Acessórias representa solicitações, não entregas/obrigações. Como a Home do Portal Sama precisa exibir atrasos, pendências, clientes e obrigações, o endpoint correto para o resumo operacional deve ser o de entregas.

Recomendação:

```env
ACESSORIAS_HOME_PATH=deliveries/ListAll
```

A Home deve consultar entregas de todas as empresas usando `ListAll`, com filtros de data e status montados pelo backend.

---

### 5.2 `ACESSORIAS_DELIVERIES_PATH=deliveries`

Esse path está incompleto.

O endpoint de entregas exige um identificador no caminho. Para consultar todas as empresas, o identificador deve ser `ListAll`.

Path recomendado:

```env
ACESSORIAS_DELIVERIES_PATH=deliveries/ListAll
```

Além disso, a chamada precisa incluir query params obrigatórios, como:

```txt
DtInitial=YYYY-MM-DD
DtFinal=YYYY-MM-DD
DtLastDH=YYYY-MM-DD HH:MM:SS
```

Quando for usado `ListAll`, o parâmetro `DtLastDH` é obrigatório e, segundo a documentação do Acessórias, aceita somente o dia atual e/ou o dia anterior.

Também é recomendável enviar:

```txt
situation=pending,delivered,read
config
Pagina=1
```

O parâmetro `config` é importante porque retorna dados de departamento, responsáveis, ID da entrega e tipo de prazo.

---

### 5.3 `ACESSORIAS_CLIENTS_PATH=companies`

Esse path também está incompleto para importação em massa.

Para consultar todas as empresas, o endpoint deve usar `ListAll`.

Path recomendado:

```env
ACESSORIAS_CLIENTS_PATH=companies/ListAll
```

Query params recomendados:

```txt
obligations
departments
stateRegistrations
registrationData
contacts
Pagina=1
```

Exemplo de chamada esperada:

```txt
GET /companies/ListAll/?obligations&departments&stateRegistrations&registrationData&contacts&Pagina=1
```

A paginação é obrigatória para importar todos os clientes. A documentação informa que o retorno de empresas é paginado e que devem ser buscadas as páginas seguintes até vir uma lista vazia.

---

### 5.4 `ACESSORIAS_COLLABORATORS_PATH=departments`

Esse path não deve ser usado como importação de colaboradores.

O endpoint `departments/ListAll` retorna departamentos, não colaboradores internos. Se o backend usar esse retorno como colaboradores, existe risco de cadastrar registros incorretos, como usuários chamados `Fiscal`, `Pessoal`, `Contábil`, etc.

Path recomendado para departamentos, se for criado no futuro:

```env
ACESSORIAS_DEPARTMENTS_PATH=departments/ListAll
```

Para colaboradores, a documentação aberta do Acessórias não apresenta um endpoint específico como:

```txt
/collaborators/ListAll
/users/ListAll
```

Portanto, a importação de colaboradores não deve usar `departments` diretamente.

A alternativa tecnicamente segura é extrair responsáveis a partir dos dados de empresas, quando a consulta de empresas for feita com o parâmetro `departments`.

A resposta de empresas pode trazer departamentos com campos como:

```json
{
  "ID": "ID do departamento",
  "Nome": "Nome do departamento",
  "RespNome": "Nome do responsável",
  "RespEmail": "E-mail do responsável"
}
```

Nesse caso, o Portal Sama pode importar apenas os responsáveis encontrados nos departamentos das empresas, mas isso deve ser tratado como:

```txt
Responsáveis importados a partir dos departamentos das empresas
```

E não como:

```txt
Todos os colaboradores ativos do Acessórias
```

---

## 6. `.env` recomendado

### 6.1 Configuração recomendada para a próxima implementação

```env
ACESSORIAS_BASE_URL=https://api.acessorias.com

ACESSORIAS_HOME_PATH=deliveries/ListAll
ACESSORIAS_DELIVERIES_PATH=deliveries/ListAll
ACESSORIAS_CLIENTS_PATH=companies/ListAll
ACESSORIAS_COLLABORATORS_PATH=

ACESSORIAS_AUTH_HEADER=Authorization
ACESSORIAS_AUTH_SCHEME=Bearer
ACESSORIAS_TIMEOUT_SEC=15
ACESSORIAS_HOME_APPEND_CONTEXT_QUERY=false
```

### 6.2 Observação importante sobre `ACESSORIAS_COLLABORATORS_PATH`

Manter vazio até existir confirmação de endpoint real de colaboradores no Acessórias.

Não usar:

```env
ACESSORIAS_COLLABORATORS_PATH=departments
```

Motivo: `departments` retorna departamentos, não colaboradores.

---

## 7. Como o backend deve montar as URLs

O `.env` deve armazenar apenas os paths base. O backend deve montar query params dinamicamente.

### 7.1 Entregas/Home

Exemplo conceitual:

```txt
GET https://api.acessorias.com/deliveries/ListAll/?DtInitial=2026-06-01&DtFinal=2026-06-30&DtLastDH=2026-06-01%2000:00:00&situation=pending,delivered,read&config&Pagina=1
```

Regras:

- `DtInitial` deve ser calculado pelo backend.
- `DtFinal` deve ser calculado pelo backend.
- `DtLastDH` deve respeitar a restrição do Acessórias quando usar `ListAll`.
- `Pagina` deve iniciar em `1`.
- O backend deve continuar buscando `Pagina=2`, `Pagina=3`, etc., até receber lista vazia.
- O parâmetro `config` deve ser enviado para enriquecer o retorno.

### 7.2 Clientes

Exemplo conceitual:

```txt
GET https://api.acessorias.com/companies/ListAll/?obligations&departments&stateRegistrations&registrationData&contacts&Pagina=1
```

Regras:

- `Pagina` deve iniciar em `1`.
- O backend deve continuar buscando as próximas páginas até receber lista vazia.
- O importador deve preservar clientes ativos, inativos e bloqueados.
- O importador não deve excluir clientes locais que não vierem no retorno, salvo se houver regra explícita futura para isso.

---

## 8. Formato real de entregas retornado pelo Acessórias

A API de entregas retorna uma lista de empresas. Dentro de cada empresa existe um array `Entregas`.

Formato esperado:

```json
[
  {
    "ID": "1",
    "Identificador": "432.612.527-66",
    "Razao": "Razão Social da Empresa LTDA",
    "Fantasia": "Fantasia da Empresa",
    "Entregas": [
      {
        "Nome": "Guia da Previdência Social",
        "EntCompetencia": "2024-02-06",
        "EntDtPrazo": "2024-02-06",
        "EntDtAtraso": "2024-02-06",
        "EntDtEntrega": "0000-00-00",
        "EntMulta": "S",
        "Status": "Atrasada!",
        "EntLastDH": "2024-11-15 13:10:03",
        "Config": {
          "EntID": "16370",
          "Tipo": "O",
          "ID": "240",
          "DptoID": "3",
          "DptoNome": "Contábil",
          "RespPrazo": "Pedro Andrade",
          "RespEntrega": ""
        }
      }
    ]
  }
]
```

O Portal Sama precisa transformar esse formato em registros internos individualizados de entregas.

---

## 9. Normalização necessária para entregas

Cada item dentro de `Entregas` deve virar uma entrega local.

Mapeamento recomendado:

| Campo interno no Portal Sama | Campo vindo do Acessórias |
|---|---|
| `externalId` | `Entregas[].Config.EntID` |
| `clientExternalId` | empresa pai `ID` |
| `clientDocument` | empresa pai `Identificador` |
| `clientName` | empresa pai `Razao` |
| `clientFantasyName` | empresa pai `Fantasia` |
| `obligationName` | `Entregas[].Nome` |
| `competence` | `Entregas[].EntCompetencia` |
| `dueAt` | `Entregas[].EntDtPrazo` |
| `delayedAt` | `Entregas[].EntDtAtraso` |
| `deliveredAt` | `Entregas[].EntDtEntrega`, exceto `0000-00-00` |
| `hasFine` | `Entregas[].EntMulta === 'S'` |
| `rawStatus` | `Entregas[].Status` |
| `departmentExternalId` | `Entregas[].Config.DptoID` |
| `departmentName` | `Entregas[].Config.DptoNome` |
| `responsibleDeadlineName` | `Entregas[].Config.RespPrazo` |
| `responsibleDeliveryName` | `Entregas[].Config.RespEntrega` |
| `sourceUpdatedAt` | `Entregas[].EntLastDH` |
| `rawPayload` | JSON original da empresa + entrega |

---

## 10. Normalização de status das entregas

O backend deve normalizar os status textuais do Acessórias para status internos.

Sugestão:

| Status do Acessórias | Status interno sugerido |
|---|---|
| `Pendente` | `PENDING` |
| `Entregue` | `DELIVERED` |
| `Atrasada!` | `OVERDUE` |
| `Guia já acessada/lida` ou status `read` | `READ` |
| vazio/desconhecido | `UNKNOWN` |

Regras adicionais:

- Se `EntDtEntrega` for diferente de `0000-00-00`, considerar a entrega como entregue, salvo regra contrária definida posteriormente.
- Se `EntDtPrazo` for menor que a data atual e não houver entrega, considerar como atrasada.
- Preservar sempre o `rawStatus` original.

---

## 11. Formato real de clientes retornado pelo Acessórias

A API de empresas retorna campos como:

```json
{
  "ID": "1",
  "Identificador": "432.612.527-66",
  "Razao": "Razão Social da Empresa LTDA",
  "Fantasia": "Fantasia da Empresa",
  "Status": "Ativa",
  "Telefone": "1127690730",
  "UF": "SP",
  "ClienteDesde": "0000-00-00",
  "ClienteAte": "2022-08-04",
  "Regime": "Lucro Real",
  "GrupoDeEmpresas": "Nordeste",
  "Departamentos": [
    {
      "ID": "ID do departamento",
      "Nome": "Nome do departamento",
      "RespNome": "Nome do responsável",
      "RespEmail": "E-mail do responsável"
    }
  ]
}
```

---

## 12. Normalização necessária para clientes

Mapeamento recomendado:

| Campo interno no Portal Sama | Campo vindo do Acessórias |
|---|---|
| `externalId` | `ID` |
| `document` | `Identificador` |
| `corporateName` | `Razao` |
| `tradeName` | `Fantasia` |
| `status` | `Status` |
| `phone` | `Telefone` |
| `state` | `UF` |
| `clientSince` | `ClienteDesde`, exceto `0000-00-00` |
| `clientUntil` | `ClienteAte`, exceto `0000-00-00` |
| `taxRegime` | `Regime` |
| `businessGroup` | `GrupoDeEmpresas` |
| `rawPayload` | JSON original |

---

## 13. Normalização de status dos clientes

O Portal Sama deve importar clientes ativos, inativos e bloqueados.

Sugestão de mapeamento:

| Status do Acessórias | Status interno sugerido |
|---|---|
| `Ativa` / `Ativo` | `ACTIVE` |
| `Inativa` / `Inativo` | `INACTIVE` |
| `Bloqueada` / `Bloqueado` | `BLOCKED` |
| vazio/desconhecido | `UNKNOWN` |

Importante: cliente bloqueado não deve ser convertido para inativo automaticamente. O status `BLOCKED` precisa ser preservado para auditoria e regras futuras.

---

## 14. Importação de colaboradores/responsáveis

A documentação aberta do Acessórias não apresenta um endpoint dedicado para colaboradores internos ativos.

Portanto, existem duas opções:

### 14.1 Opção segura imediata

Extrair responsáveis a partir dos departamentos retornados em `companies/ListAll/?departments`.

Campos usados:

```txt
Departamentos[].RespNome
Departamentos[].RespEmail
Departamentos[].Nome
Departamentos[].ID
```

Esses registros devem ser tratados como responsáveis vinculados a departamentos de clientes.

Nome recomendado para a funcionalidade:

```txt
Importar responsáveis encontrados nas empresas
```

Evitar chamar isso de:

```txt
Importar todos os colaboradores ativos
```

porque o retorno não garante que todos os colaboradores ativos do Acessórias estão presentes.

### 14.2 Opção ideal futura

Solicitar ao suporte/fornecedor do Acessórias confirmação sobre endpoint oficial para colaboradores/usuários internos.

Exemplos que precisariam ser confirmados:

```txt
/users/ListAll
/collaborators/ListAll
/employees/ListAll
```

Enquanto esse endpoint não for confirmado, não usar `departments/ListAll` como fonte de colaboradores.

---

## 15. Área DEV — funcionalidade recomendada

Adicionar uma seção na área DEV chamada:

```txt
Integração Acessórias
```

Essa seção deve permitir testes, pré-visualização e importação controlada dos dados.

---

## 16. Botões recomendados na área DEV

### 16.1 Botões principais

```txt
Testar conexão com Acessórias
Pré-visualizar entregas
Sincronizar entregas
Pré-visualizar clientes
Importar clientes
Pré-visualizar responsáveis
Importar responsáveis
Importar tudo
```

### 16.2 Botões que devem ficar desativados inicialmente

```txt
Importar colaboradores ativos
```

Motivo: falta endpoint oficial confirmado para colaboradores ativos.

Texto recomendado no tooltip:

```txt
A documentação atual do Acessórias não apresenta endpoint oficial para colaboradores ativos. Use a importação de responsáveis por departamento ou configure um endpoint oficial quando fornecido pelo Acessórias.
```

---

## 17. Comportamento esperado dos botões

### 17.1 Testar conexão com Acessórias

Executa uma requisição simples autenticada para validar:

- `ACESSORIAS_BASE_URL`;
- token;
- header de autenticação;
- tempo de resposta;
- status HTTP;
- rate limit, quando retornado pela API.

Não deve persistir dados.

---

### 17.2 Pré-visualizar entregas

Executa a consulta de entregas, normaliza os dados em memória e retorna resumo:

```json
{
  "totalCompaniesFetched": 10,
  "totalDeliveriesFetched": 153,
  "pending": 80,
  "delivered": 50,
  "overdue": 23,
  "unknown": 0,
  "sample": []
}
```

Não deve persistir dados.

---

### 17.3 Sincronizar entregas

Executa a consulta real e persiste as entregas no banco local.

Regras:

- usar upsert por `externalId`;
- preservar `rawPayload`;
- atualizar status e datas;
- não duplicar entregas;
- registrar log da sincronização;
- respeitar rate limit.

---

### 17.4 Pré-visualizar clientes

Consulta `companies/ListAll`, percorre a paginação, normaliza os dados em memória e retorna resumo:

```json
{
  "totalFetched": 240,
  "active": 180,
  "inactive": 45,
  "blocked": 15,
  "unknown": 0,
  "sample": []
}
```

Não deve persistir dados.

---

### 17.5 Importar clientes

Importa clientes ativos, inativos e bloqueados.

Regras:

- usar upsert por `Identificador` ou `ID` externo;
- preservar status real;
- preservar CNPJ/CPF/CAEPF;
- não apagar clientes locais automaticamente;
- registrar data da última sincronização;
- registrar payload bruto para auditoria.

---

### 17.6 Pré-visualizar responsáveis

Consulta empresas com `departments`, extrai responsáveis únicos por e-mail e retorna resumo:

```json
{
  "totalCompaniesScanned": 240,
  "totalDepartmentsScanned": 520,
  "uniqueResponsibles": 35,
  "withoutEmail": 4,
  "sample": []
}
```

Não deve persistir dados.

---

### 17.7 Importar responsáveis

Importa responsáveis encontrados nos departamentos das empresas.

Regras:

- usar e-mail como chave primária preferencial;
- se não houver e-mail, não criar usuário automaticamente;
- marcar origem como `ACESSORIAS_DEPARTMENT_RESPONSIBLE`;
- não conceder permissões administrativas automaticamente;
- não ativar login automaticamente sem validação interna;
- vincular responsável ao departamento/cliente quando possível.

---

### 17.8 Importar tudo

Executa em sequência:

1. Importar clientes.
2. Importar responsáveis encontrados nos departamentos.
3. Sincronizar entregas.

Deve exibir confirmação antes de executar.

Deve mostrar resultado final:

```json
{
  "clients": {
    "created": 10,
    "updated": 230,
    "failed": 0
  },
  "responsibles": {
    "created": 2,
    "updated": 33,
    "skippedWithoutEmail": 4
  },
  "deliveries": {
    "created": 80,
    "updated": 73,
    "failed": 0
  }
}
```

---

## 18. Endpoints internos recomendados no Portal Sama

Criar ou ajustar endpoints internos no backend:

```txt
GET  /api-v2/integrations/acessorias/health
GET  /api-v2/integrations/acessorias/home-summary
GET  /api-v2/integrations/acessorias/deliveries/preview
POST /api-v2/integrations/acessorias/deliveries/sync
GET  /api-v2/integrations/acessorias/clients/preview
POST /api-v2/integrations/acessorias/clients/sync
GET  /api-v2/integrations/acessorias/responsibles/preview
POST /api-v2/integrations/acessorias/responsibles/sync
POST /api-v2/integrations/acessorias/sync-all
```

Todos devem exigir autenticação e autorização de perfil DEV ou administrador técnico.

---

## 19. Regras de segurança obrigatórias

### 19.1 Token somente no backend

O token do Acessórias nunca deve ser exposto no frontend.

Não usar variáveis `VITE_*` para token.

Correto:

```env
ACESSORIAS_TOKEN=valor_secreto_no_backend
```

Incorreto:

```env
VITE_ACESSORIAS_TOKEN=valor_secreto
```

---

### 19.2 Não commitar `.env`

O `.env` real não deve ir para o repositório, ZIPs compartilhados ou documentação pública.

Manter apenas:

```txt
.env.example
```

---

### 19.3 Rate limit

A API do Acessórias possui limite de requisições. O backend deve implementar controle para evitar excesso de chamadas.

Recomendações:

- limitar concorrência;
- adicionar pequeno intervalo entre páginas;
- implementar retry com backoff para `429`;
- registrar falhas sem repetir indefinidamente;
- evitar que múltiplos usuários DEV disparem sincronização simultânea.

---

### 19.4 Auditoria

Toda importação deve gerar log técnico com:

```txt
usuário que executou
horário de início
horário de término
entidade importada
quantidade criada
quantidade atualizada
quantidade ignorada
quantidade com erro
status final
mensagem de erro, se houver
```

---

### 19.5 Permissões

A área DEV deve ser restrita.

Somente perfis autorizados devem acessar botões de importação.

Regras mínimas:

- Colaborador comum: sem acesso.
- Gestor: sem acesso ou somente visualização, conforme decisão futura.
- DEV/Admin técnico: acesso completo.

---

### 19.6 Proteção contra importação acidental

Botões destrutivos ou de grande impacto devem exigir confirmação.

Exemplo:

```txt
Você está prestes a importar clientes, responsáveis e entregas do Acessórias. Essa ação pode criar ou atualizar registros no Portal Sama. Deseja continuar?
```

Não permitir exclusão automática de registros locais nesta fase.

---

## 20. Melhorias recomendadas no backend

### 20.1 Criar client HTTP dedicado

Criar um serviço dedicado, por exemplo:

```txt
AcessoriasHttpClient
```

Responsabilidades:

- montar URL segura;
- aplicar token;
- aplicar timeout;
- tratar erro HTTP;
- tratar rate limit;
- fazer paginação;
- registrar logs técnicos sem expor token.

---

### 20.2 Criar normalizadores isolados

Criar normalizadores separados:

```txt
AcessoriasDeliveriesNormalizer
AcessoriasCompaniesNormalizer
AcessoriasResponsiblesNormalizer
```

Evitar deixar a transformação espalhada pelos services.

---

### 20.3 Validar payload com schema

Usar validação com `zod` ou `class-validator` para proteger o backend contra mudanças inesperadas na API externa.

Exemplo recomendado:

```txt
zod
```

Motivo:

- validação rápida de payload externo;
- melhor inferência TypeScript;
- facilidade para gerar mensagens de erro técnicas;
- proteção contra campos ausentes ou formatos inesperados.

---

### 20.4 Persistir payload bruto

Manter `rawPayload` em tabela de integração ou campo JSON para auditoria.

Isso ajuda a diagnosticar divergências futuras entre:

- API Acessórias;
- backend Portal Sama;
- tela Home;
- dados persistidos localmente.

---

## 21. Melhorias recomendadas no frontend

### 21.1 Painel DEV de integração

Criar cards para:

```txt
Conexão
Entregas
Clientes
Responsáveis
Logs de sincronização
```

Cada card deve mostrar:

```txt
Última execução
Último status
Quantidade criada
Quantidade atualizada
Quantidade com erro
Botões de preview/importação
```

---

### 21.2 Feedback visual

Durante importação:

- mostrar loading;
- bloquear novo clique no mesmo botão;
- mostrar toast de sucesso/erro;
- mostrar resumo final;
- permitir copiar resultado técnico para debug.

---

### 21.3 Evitar exposição de dados sensíveis

A tela DEV não deve mostrar token, header completo de autenticação ou payloads completos com dados sensíveis por padrão.

Se for necessário exibir payload bruto, colocar atrás de ação explícita:

```txt
Ver payload técnico
```

E mascarar documentos/e-mails quando possível.

---

## 22. Critérios de aceite

A implementação será considerada correta quando:

1. A Home não usar mais `requests` para resumo operacional de entregas.
2. `deliveries/ListAll` for usado com datas, status, `config` e paginação.
3. Clientes forem importados de `companies/ListAll` com paginação completa.
4. Clientes ativos, inativos e bloqueados forem preservados corretamente.
5. `departments` não for usado como fonte direta de colaboradores.
6. Responsáveis forem extraídos somente a partir dos departamentos das empresas, com nomenclatura correta.
7. A tela DEV permitir preview antes da importação.
8. A importação exigir permissão DEV/Admin técnico.
9. Nenhum token do Acessórias for exposto no frontend.
10. A Home deixar de exibir `Obrigacao nao informada`, `Cliente nao informado` e `Vencimento: -` quando os dados existirem no retorno do Acessórias.

---

## 23. Prompt recomendado para o Codex

```txt
Leia a documentação integracao_acessorias_paths_importacao_dev.md e aplique as correções no Portal Sama.

Objetivos principais:

1. Corrigir a integração com a API do Sistema Acessórias para usar os endpoints reais documentados.
2. Não usar requests como fonte da Home operacional; usar deliveries/ListAll.
3. Não usar departments como fonte direta de colaboradores.
4. Ajustar o backend para montar query params dinamicamente para deliveries/ListAll, incluindo DtInitial, DtFinal, DtLastDH, situation, config e Pagina.
5. Implementar paginação automática para companies/ListAll e deliveries/ListAll até receber lista vazia.
6. Criar normalizador para o formato real de entregas do Acessórias, onde o retorno é uma lista de empresas e cada empresa possui um array Entregas.
7. Criar normalizador para clientes usando campos reais como ID, Identificador, Razao, Fantasia, Status, Telefone, UF, ClienteDesde, ClienteAte, Regime e GrupoDeEmpresas.
8. Preservar status de clientes ativos, inativos e bloqueados.
9. Implementar importação de responsáveis a partir de companies/ListAll com departments, usando RespNome e RespEmail, sem chamar isso de importação completa de colaboradores ativos.
10. Adicionar na área DEV um painel de Integração Acessórias com botões de testar conexão, pré-visualizar entregas, sincronizar entregas, pré-visualizar clientes, importar clientes, pré-visualizar responsáveis, importar responsáveis e importar tudo.
11. Garantir que todos os endpoints internos de importação exijam autenticação e permissão DEV/Admin técnico.
12. Garantir que o token do Acessórias nunca seja exposto no frontend.
13. Registrar logs de sincronização com usuário executor, horário, entidade, criados, atualizados, ignorados, erros e status final.
14. Adicionar tratamento de rate limit, timeout, retry controlado e mensagens de erro seguras.
15. Atualizar .env.example e documentação de deploy sem incluir token real.

Ao final, informe os arquivos alterados, os endpoints internos criados ou modificados, as variáveis de ambiente necessárias e como testar a integração em ambiente de homologação.
```

---

## 24. Resumo final

A configuração atual melhorou porque a API deixou de responder `404` no caminho `/`, mas ainda não está correta para a finalidade do Portal Sama.

A Home deve usar `deliveries/ListAll`, não `requests`.

A importação de clientes deve usar `companies/ListAll`, não apenas `companies`.

A importação de colaboradores não deve usar `departments`, porque esse endpoint retorna departamentos. Até existir endpoint oficial de colaboradores, a solução segura é importar responsáveis encontrados nos departamentos das empresas.

O ponto mais importante agora é ajustar o backend para entender o JSON real do Acessórias. Sem essa normalização, a tela continuará exibindo dados genéricos como `Obrigacao nao informada`, `Cliente nao informado` e `Vencimento: -`, mesmo que a consulta HTTP esteja funcionando.
