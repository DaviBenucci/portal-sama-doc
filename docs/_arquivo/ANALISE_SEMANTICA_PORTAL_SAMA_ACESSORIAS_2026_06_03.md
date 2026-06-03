# Análise semântica — Portal Sama + Integração Acessórias

**Data:** 2026-06-03  
**Projeto analisado:** `Desktop.zip` extraído em `/mnt/data/portal_sama_desktop`  
**Escopo:** conciliação entre documentação, código atual, contrato público da API Acessórias, falhas internas e melhorias técnicas.

---

## 1. Resumo executivo

O projeto já possui uma base relevante da integração com o Acessórias: leitura de resumo para Home, sincronização manual de entregas, preview/importação DEV, persistência local de entregas, mapeamento de entregas para colunas, divergências e uma Central de Vencimentos em `/departamentos/vencimentos`.

Contudo, o código ainda não está totalmente alinhado à documentação funcional discutida. A principal divergência é que a documentação descreve o Portal Sama como uma camada local resiliente, auditável e completa sobre as obrigações do Acessórias, enquanto o código ainda depende de chamadas externas em alguns pontos críticos, filtra entregas de forma parcial e não resolve responsáveis das obrigações para colaboradores locais.

O erro interno ao buscar informações do Acessórias tem uma causa técnica bem provável: os serviços de entregas e cadastros não capturam adequadamente erros de rede, DNS, timeout e `AbortError` do `fetch`. Quando isso ocorre, a exceção não é convertida em `ServiceUnavailableException` e acaba chegando ao filtro global como erro genérico, resultando em `500 Erro interno`.

Além disso, existe um problema semântico no uso de `deliveries/ListAll`: o código reutiliza `DtLastDH` com base no último sync bem-sucedido. Pelo contrato do Acessórias, quando o identificador é `ListAll`, `DtLastDH` é obrigatório e aceita apenas o dia atual ou o dia anterior. Se o último sync tiver ocorrido há mais tempo, a API pode retornar erro ou dados incompletos.

---

## 2. Testes executados

### 2.1 Backend — testes unitários

Comando executado:

```bash
cd /mnt/data/portal_sama_desktop/portal-sama-api
node node_modules/jest/bin/jest.js --runInBand
```

Resultado:

```txt
Test Suites: 41 passed, 41 total
Tests:       232 passed, 232 total
Snapshots:   0 total
```

Conclusão: os testes unitários existentes do backend passam, inclusive os módulos relacionados ao Acessórias. Porém, os testes atuais não cobrem todos os cenários de produção identificados nesta análise, especialmente erro de rede/timeout real, `DtLastDH` antigo em `ListAll`, Central incluindo entregas `DELIVERED` e vinculação de responsáveis a colaboradores.

### 2.2 Backend — build NestJS

Comando executado:

```bash
cd /mnt/data/portal_sama_desktop/portal-sama-api
node node_modules/@nestjs/cli/bin/nest.js build
```

Resultado: build concluído sem erro.

Observação: `npm run build` diretamente pode falhar em ambiente de ZIP extraído por permissão de executável em `node_modules/.bin/nest`. Usar o binário via `node` contornou o problema.

### 2.3 Frontend — build React/Vite

Comando executado:

```bash
cd /mnt/data/portal_sama_desktop/portal-sama-web
npm run build
```

Resultado: falha de TypeScript.

Erro encontrado:

```txt
src/pages/accounting/IntegraAiPage.tsx(1277,77): error TS2339:
Property 'error' does not exist on type 'IntegraAiExportResponse'.
```

Arquivo afetado:

```txt
portal-sama-web/src/pages/accounting/IntegraAiPage.tsx
```

Trecho problemático:

```tsx
) : visibleExportResult && !visibleExportResult.ok ? (
  <div className="form-error mb-4">{visibleExportResult.error}</div>
) : null}
```

Motivo: o TypeScript não está estreitando corretamente o union discriminado dentro desse trecho JSX. A correção é criar uma variável/guarda específica para o caso de erro.

Correção sugerida:

```tsx
const visibleExportError = visibleExportResult?.ok === false ? visibleExportResult : null
```

E no JSX:

```tsx
{visibleExportError ? (
  <div className="form-error mb-4">{visibleExportError.error}</div>
) : null}
```

### 2.4 Prisma validate

O `prisma validate` não foi conclusivo porque o ambiente tentou baixar binários do Prisma e falhou por DNS/rede:

```txt
getaddrinfo EAI_AGAIN binaries.prisma.sh
```

Isso não comprova erro no schema. Indica apenas que a validação dependeu de download externo não disponível no ambiente.

---

## 3. Estado atual por módulo

## 3.1 Backend Acessórias — entregas

Arquivo principal:

```txt
portal-sama-api/src/modules/integrations/acessorias/acessorias-deliveries.service.ts
```

O serviço já possui:

- leitura de configuração via `.env`;
- `ACESSORIAS_BASE_URL`;
- `ACESSORIAS_TOKEN` apenas no backend;
- `ACESSORIAS_DELIVERIES_PATH`;
- paginação para caminhos `ListAll`;
- tentativa de retry em `429`;
- normalização de entregas;
- persistência em `acessorias_deliveries`;
- registro de sync run.

Ponto crítico identificado:

```ts
private async fetchJsonAttempt(...) {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), config.timeoutMs);

  try {
    const response = await fetch(url, { ... });
    ...
  } finally {
    clearTimeout(timeout);
  }
}
```

Esse método trata respostas HTTP não OK, JSON inválido e `429`, mas não captura exceções de rede/timeout. Se o `fetch` falhar antes de retornar `Response`, a exceção sobe crua.

Consequência: o filtro global transforma isso em `500 Erro interno`, em vez de retornar uma falha externa sanitizada como `503 Acessórias indisponível`.

## 3.2 Backend Acessórias — cadastros/importação

Arquivo principal:

```txt
portal-sama-api/src/modules/integrations/acessorias/acessorias-registrations.service.ts
```

O mesmo padrão ocorre em `fetchJsonAttempt`: há `try/finally`, mas não há `catch` para `AbortError`, DNS, conexão recusada ou falha de rede.

O serviço já extrai responsáveis de `Departamentos` vindos de `companies/ListAll`, usando campos como:

```txt
RespNome
RespEmail
Nome
ID
Identificador
```

Isso está parcialmente alinhado à documentação. Porém, ainda não resolve responsáveis das obrigações (`RespPrazo`, `RespEntrega`) para colaboradores locais e não possui tabela de alias/revisão.

## 3.3 Backend Home Acessórias

Arquivo principal:

```txt
portal-sama-api/src/modules/integrations/acessorias/acessorias-home.service.ts
```

A Home está mais resiliente do que preview/sync, porque trata falhas externas e retorna diagnóstico sanitizado. Porém, ela ainda consulta o Acessórias diretamente para montar resumo operacional, o que é pesado para Home, aumenta latência e pode sofrer com rate limit.

Melhor direção: Home deve preferir dados locais de `acessorias_deliveries`, usando a chamada externa somente em sincronizações controladas.

## 3.4 Backend Departments/Workspace

Arquivo principal:

```txt
portal-sama-api/src/modules/departments/departments.service.ts
```

Pontos atuais:

```ts
const PRIVILEGED_ROLES = new Set(['ADMIN', 'DEV']);
const MANAGER_ROLES = new Set(['ADMIN', 'DEV', 'MANAGER']);
const EDITOR_ROLES = new Set(['ADMIN', 'DEV', 'MANAGER', 'DEPARTMENT']);
```

O perfil `MASTER` não aparece como privilegiado. Isso diverge da necessidade operacional discutida de o MASTER conseguir ver painel de colaborador, competências e obrigações.

Outro ponto crítico:

```ts
this.prisma.acessoriasDelivery.findMany({
  where: {
    dueAt: { gte: monthStart, lt: nextMonthStart },
    status: { in: ['PENDING', 'DUE_SOON', 'OVERDUE', 'UNKNOWN'] },
    ...
  }
})
```

Esse filtro exclui `DELIVERED` e `CANCELED`. Por isso a Central de Vencimentos não mostra todas as competências/obrigações, apenas abertas ou problemáticas.

## 3.5 Frontend Home

Arquivo principal:

```txt
portal-sama-web/src/pages/home/HomePage.tsx
```

Problema visual:

```ts
function queryValue(isLoading: boolean, isError: boolean, value: string | number) {
  if (isLoading) return '...'
  if (isError) return '!'
  return value
}
```

Isso explica os cards mostrando apenas `...` durante carregamento. Funciona tecnicamente, mas é ruim para UX, porque parece ausência de dado ou erro.

## 3.6 Frontend Central de Vencimentos

Arquivo principal:

```txt
portal-sama-web/src/pages/departments/DepartmentDueCenterPage.tsx
```

Estado atual:

- rota existe;
- consolida dados vindos do workspace;
- possui filtros por origem, status simples e busca textual;
- loading básico já existe;
- não envia `status`, `source`, `include_completed` nem `colaborador_id` como filtros operacionais completos;
- não mostra competência, entregue em, responsável, status Acessórias detalhado;
- não permite filtro específico por entregues/cancelados.

Tipo atual:

```ts
export type DepartmentWorkspaceDueItem = {
  source: 'calendar' | 'acessorias'
  delivery_id?: string | null
  external_id?: string | null
  delivery_status?: string | null
  raw_status?: string | null
  company_id: string
  company_name: string
  company_cnpj: string
  column_key: string
  column_label: string
  group_label: string
  title: string
  due_date: string
  days_until: number
  topics: Array<...>
}
```

Campos ausentes:

```txt
competence
delivered_at
is_delivered
is_canceled
status_label
responsible_user_id
responsible_name
responsible_match_status
```

## 3.7 Frontend painel do colaborador

Arquivo principal:

```txt
portal-sama-web/src/pages/collaborators/CollaboratorViewPage.tsx
```

A página mostra dados cadastrais, permissões e uma seção vazia de carteira de clientes. Ela ainda não possui aba/seção “Obrigações e Competências”. Portanto, a necessidade do MASTER ver todas as competências e obrigações do colaborador ainda não foi implementada.

---

## 4. Diferenças entre documentação e código

| Tema | Documentação/discussão | Código atual | Diferença/erro |
|---|---|---|---|
| Fonte de vencimentos | Acessórias como fonte oficial; Portal Sama como camada local resiliente | Parte da Home ainda consulta Acessórias diretamente | Home deveria priorizar dados locais sincronizados |
| Central de Vencimentos | Deve mostrar vencimentos oficiais, internos, previstos, pendentes, atrasados, entregues e divergências | Central filtra somente `PENDING`, `DUE_SOON`, `OVERDUE`, `UNKNOWN` no backend | Entregues/canceladas não aparecem |
| Entregas concluídas | Deve informar se a competência foi entregue | `DELIVERED` é excluído da busca do workspace | Usuário não consegue validar competências concluídas |
| Responsáveis | Usar responsáveis de empresas e obrigações para conciliar colaboradores | Usa `Departamentos[].RespNome/RespEmail`, mas não resolve `RespPrazo/RespEntrega` para usuário local | Falta resolver responsável operacional das obrigações |
| Colaboradores sem endpoint oficial | Criar vínculo seguro/pendente, não usuário ativo automático | Cria usuário inativo quando há e-mail, mas sem tabela de alias/revisão | Falta fluxo seguro de revisão de alias |
| MASTER | MASTER deve conseguir visão ampla operacional | `MASTER` não existe em `DEFAULT_ROLES` nem em `PRIVILEGED_ROLES` | Inconsistência de RBAC |
| Erros externos Acessórias | Falhas externas não devem quebrar operação local | `fetch` em deliveries/registrations pode estourar erro cru | Pode gerar `500 Erro interno` |
| `deliveries/ListAll` | `DtLastDH` em `ListAll` limitado a hoje/ontem | `incrementalSince` antigo pode virar `DtLastDH` | Pode gerar erro externo ou dados incompletos |
| Loading Home | Deve indicar carregamento claro | Mostra `...` | UX ruim |
| Build do projeto | Projeto deveria compilar | Web falha em `IntegraAiPage.tsx` | Bloqueia build/deploy frontend |

---

## 5. Por que aparece erro interno ao buscar dados do Acessórias

A causa mais provável é a combinação de três fatores.

### 5.1 Erros de rede/timeout não são tratados nos serviços certos

Em `acessorias-deliveries.service.ts` e `acessorias-registrations.service.ts`, o código converte respostas HTTP ruins em `ServiceUnavailableException`, mas não converte falhas de `fetch`.

Exemplos de falhas não tratadas:

```txt
AbortError por timeout
ENOTFOUND / EAI_AGAIN DNS
ECONNREFUSED
ECONNRESET
TLS/socket error
falha temporária de rede
```

Quando essas falhas sobem cruas, o Nest trata como exceção inesperada e o frontend recebe erro interno.

### 5.2 `DtLastDH` pode estar inválido para `deliveries/ListAll`

O método atual:

```ts
private defaultDeliveryDateWindow(incrementalSince?: Date | null) {
  const lastDh = incrementalSince ? new Date(incrementalSince) : new Date(now);
  ...
}
```

Se `incrementalSince` for antigo, o Portal Sama envia `DtLastDH` antigo com `deliveries/ListAll`. Isso conflita com o contrato do Acessórias, que restringe `DtLastDH` em `ListAll` ao dia atual ou anterior.

### 5.3 Carga externa grande pode estourar timeout/rate limit

O código pagina `ListAll` até vir lista vazia, respeitando `ACESSORIAS_MAX_PAGES`. Isso é correto para paginação, mas perigoso para requisições acionadas por tela, especialmente Home/Preview. Uma carga grande pode ultrapassar timeout do backend, proxy ou frontend.

---

## 6. Correções prioritárias

## P0 — Corrigir build do frontend

Arquivo:

```txt
portal-sama-web/src/pages/accounting/IntegraAiPage.tsx
```

Adicionar variável de estreitamento:

```tsx
const visibleExportError = visibleExportResult?.ok === false ? visibleExportResult : null
```

Trocar:

```tsx
visibleExportResult && !visibleExportResult.ok ? (
  <div className="form-error mb-4">{visibleExportResult.error}</div>
) : null
```

por:

```tsx
visibleExportError ? (
  <div className="form-error mb-4">{visibleExportError.error}</div>
) : null
```

## P0 — Tratar erro externo no Acessórias como 503 sanitizado

Aplicar em:

```txt
acessorias-deliveries.service.ts
acessorias-registrations.service.ts
```

Modelo:

```ts
try {
  const response = await fetch(url, {
    headers: { Accept: 'application/json', [config.authHeader]: token },
    signal: controller.signal,
  });

  // regras atuais...
} catch (error) {
  if (error instanceof ServiceUnavailableException) throw error;

  const isAbort = error instanceof Error && error.name === 'AbortError';

  throw new ServiceUnavailableException({
    code: isAbort
      ? 'ACESSORIAS_DELIVERIES_TIMEOUT'
      : 'ACESSORIAS_DELIVERIES_FETCH_FAILED',
    message: isAbort
      ? 'Tempo limite ao consultar entregas do Acessorias.'
      : 'Falha de comunicacao ao consultar entregas do Acessorias.',
  });
} finally {
  clearTimeout(timeout);
}
```

No serviço de cadastros, usar códigos equivalentes:

```txt
ACESSORIAS_REGISTRATIONS_TIMEOUT
ACESSORIAS_REGISTRATIONS_FETCH_FAILED
```

## P0 — Corrigir estratégia de `DtLastDH`

Regra mínima:

```ts
private clampListAllDtLastDh(value?: Date | null) {
  const now = new Date();
  const yesterdayStart = new Date(now.getFullYear(), now.getMonth(), now.getDate() - 1);

  if (!value || value < yesterdayStart) return yesterdayStart;
  return value;
}
```

Regra correta arquitetural:

```txt
Sincronização incremental recente:
  deliveries/ListAll + DtLastDH hoje/ontem

Carga completa/backfill:
  companies/ListAll
  → deliveries/{Identificador}
  → DtInitial/DtFinal por empresa
```

`deliveries/ListAll` não deve ser tratado como backfill completo.

## P1 — Central deve incluir entregues/canceladas

Backend:

```txt
portal-sama-api/src/modules/departments/dto/list-fiscal-workspace.dto.ts
portal-sama-api/src/modules/departments/departments.service.ts
```

Adicionar filtros:

```txt
status=all|open|delivered|overdue|canceled
source=all|calendar|acessorias
include_completed=true|false
colaborador_id=<id>
```

Trocar filtro fixo:

```ts
status: { in: ['PENDING', 'DUE_SOON', 'OVERDUE', 'UNKNOWN'] }
```

por filtro dinâmico:

```ts
const statusFilter =
  query.status === 'delivered'
    ? { in: ['DELIVERED'] }
    : query.status === 'canceled'
      ? { in: ['CANCELED'] }
      : query.status === 'all' || query.include_completed === 'true'
        ? undefined
        : { in: ['PENDING', 'DUE_SOON', 'OVERDUE', 'UNKNOWN'] };
```

Importante: entregas `DELIVERED` devem aparecer na Central, mas não devem bloquear célula por vencimento.

## P1 — Ampliar `DepartmentWorkspaceDueItem`

Adicionar campos:

```ts
competence?: string | null
responsible_user_id?: string | null
responsible_name?: string | null
delivered_at?: string | null
is_delivered?: boolean
is_canceled?: boolean
status_label?: string
```

Com isso, a Central pode exibir:

```txt
Competência
Responsável
Vencimento
Entregue em
Status Acessórias
Origem
```

## P1 — Resolver responsáveis do Acessórias para colaboradores locais

Criar um serviço:

```txt
AcessoriasResponsibleResolverService
```

Ordem de conciliação:

```txt
1. E-mail exato
2. Username/login
3. Alias confirmado
4. Nome normalizado + departamento, apenas se houver um único candidato
5. Caso ambíguo: pendência de revisão
```

Adicionar campos em `AcessoriasDelivery`:

```prisma
responsibleUserId      String? @map("responsible_user_id")
responsibleMatchStatus String? @map("responsible_match_status") @db.VarChar(40)
responsibleMatchScore  Int?    @map("responsible_match_score")
responsibleMatchReason String? @map("responsible_match_reason") @db.VarChar(255)
```

Criar tabela:

```prisma
model AcessoriasResponsibleAlias {
  id             String   @id @default(uuid())
  userId         String?  @map("user_id")
  externalName   String   @map("external_name") @db.VarChar(191)
  normalizedName String   @map("normalized_name") @db.VarChar(191)
  externalEmail  String?  @map("external_email") @db.VarChar(191)
  department     String?  @db.VarChar(160)
  source         String   @db.VarChar(80)
  status         String   @default("PENDING_REVIEW") @db.VarChar(40)
  confidence     Int      @default(0)
  firstSeenAt    DateTime @default(now()) @map("first_seen_at")
  lastSeenAt     DateTime @updatedAt @map("last_seen_at")
  metadata       Json?

  @@index([normalizedName])
  @@index([externalEmail])
  @@index([userId, status])
  @@map("acessorias_responsible_aliases")
}
```

Regra de segurança: não criar usuário ativo automaticamente a partir do Acessórias. Quando necessário, criar alias pendente ou usuário inativo sem acesso.

## P1 — Painel do colaborador com obrigações

Arquivo:

```txt
portal-sama-web/src/pages/collaborators/CollaboratorViewPage.tsx
```

Adicionar seção:

```txt
Obrigações e Competências
```

Consumir:

```txt
GET /api-v2/departments/workspace?colaborador_id=<id>&status=all&include_completed=true
```

Exibir:

```txt
Total
Pendentes
Vencidas
Entregues
Canceladas
Empresa
Obrigação
Competência
Vencimento
Entregue em
Status
Origem
```

## P1 — Definir oficialmente o papel do MASTER

Existem duas opções.

### Opção A — Usar DEV como papel técnico total

Se essa for a decisão oficial, então remover expectativa operacional de `MASTER` e usar `DEV` para visão total.

### Opção B — Formalizar MASTER

Adicionar `MASTER` em `DEFAULT_ROLES` e definir permissões. Para a necessidade discutida, o mínimo seria:

```txt
collaborators.read
departments.workspace.read
integrations.acessorias.deliveries.read
calendar.read
clients.read
client_assignments.read
```

E ajustar:

```ts
const PRIVILEGED_ROLES = new Set(['ADMIN', 'DEV', 'MASTER']);
const MANAGER_ROLES = new Set(['ADMIN', 'DEV', 'MASTER', 'MANAGER']);
```

Recomendação: se `MASTER` existir para supervisão ampla, deixá-lo como leitura ampla e não como editor total por padrão.

## P2 — Melhorar loading da Home

Substituir `queryValue(...)= '...'` por `loading` explícito no `KpiCard`.

Exemplo:

```tsx
type KpiCardProps = {
  label: string
  value: string | number
  caption?: string
  loading?: boolean
}
```

Renderizar skeleton:

```tsx
{loading ? (
  <div className="mt-3 h-8 w-20 animate-pulse rounded bg-slate-200" />
) : (
  <strong>{value}</strong>
)}
```

E caption:

```tsx
{loading ? 'Carregando informações do Acessórias...' : caption}
```

## P2 — Renomear navegação

A rota:

```txt
/manager/calendario/config
```

não deve parecer uma segunda página operacional de vencimentos. Recomenda-se renomear no menu para:

```txt
Config. vencimentos
```

A página operacional única deve ser:

```txt
/departamentos/vencimentos
```

---

## 7. Testes adicionais necessários

Adicionar testes backend:

```txt
1. fetchJsonAttempt retorna ServiceUnavailableException em AbortError.
2. fetchJsonAttempt retorna ServiceUnavailableException em erro de rede.
3. deliveries/ListAll nunca usa DtLastDH anterior a ontem.
4. Central inclui DELIVERED quando include_completed=true.
5. Central não bloqueia célula por vencimento quando entrega está DELIVERED.
6. Resolver responsável vincula por e-mail.
7. Resolver responsável vincula por username.
8. Resolver responsável cria alias PENDING_REVIEW quando só existe nome ambíguo.
9. MASTER, se formalizado, consegue ler workspace sem permissão de escrita.
```

Adicionar testes frontend:

```txt
1. Home mostra skeleton em vez de `...`.
2. Central possui filtro Entregues.
3. Central renderiza competência e entregue em.
4. Painel do colaborador renderiza Obrigações e Competências.
5. IntegraAiPage compila com union discriminado de exportResult.
```

---

## 8. Segurança

## 8.1 Token do Acessórias

Foi identificado arquivo `.env` dentro do ZIP analisado contendo `ACESSORIAS_TOKEN`. O token não foi reproduzido nesta documentação.

Recomendação imediata:

```txt
1. Rotacionar o token no Acessórias.
2. Remover `.env` de qualquer ZIP enviado para análise.
3. Garantir `.gitignore` com `.env`.
4. Manter somente `.env.example` com placeholders.
5. Verificar se o token não foi commitado no histórico.
```

## 8.2 Não expor dados sensíveis no frontend

O padrão atual de manter o token apenas no backend está correto. O frontend deve continuar chamando apenas endpoints internos do Portal Sama.

## 8.3 Responsáveis importados

Responsáveis vindos de `RespPrazo`, `RespEntrega`, `RespNome` e `RespEmail` não devem virar usuários ativos automaticamente. O fluxo seguro é:

```txt
responsável externo
→ alias pendente ou usuário inativo
→ revisão por MASTER/DEV/Admin
→ vínculo confirmado
```

---

## 9. Plano de execução recomendado

### Fase 1 — Estabilização técnica

1. Corrigir build do frontend em `IntegraAiPage.tsx`.
2. Corrigir tratamento de erro externo nos serviços Acessórias.
3. Corrigir `DtLastDH` para não usar datas antigas em `ListAll`.
4. Aumentar timeout de importações manuais para ambiente real.

### Fase 2 — Central única de vencimentos

1. Adicionar filtros backend de status/source/include_completed.
2. Incluir `DELIVERED` e `CANCELED` quando solicitado.
3. Adicionar campos de competência, entregue em e responsável.
4. Atualizar tabela da Central.
5. Renomear configuração de calendário para “Config. vencimentos”.

### Fase 3 — Responsáveis e painel do colaborador

1. Criar resolver de responsáveis.
2. Criar tabela de alias.
3. Vincular `responsibleUserId` nas entregas.
4. Criar tela de revisão de responsáveis Acessórias.
5. Adicionar seção “Obrigações e Competências” no painel do colaborador.

### Fase 4 — Resiliência e operação

1. Home baseada em dados locais.
2. Scheduler incremental real.
3. Backfill por empresa.
4. Notificações D-7/D-3/D-1/D0/D+1.
5. Auditoria completa.

---

## 10. Conclusão

O projeto está em um estado intermediário: a base da integração foi implementada, mas ainda não está semanticamente completa em relação à documentação e às necessidades operacionais discutidas.

Os maiores problemas são:

```txt
1. Erro interno por falta de tratamento de falhas reais do fetch.
2. Uso perigoso de DtLastDH antigo com deliveries/ListAll.
3. Central de Vencimentos filtrando fora entregas concluídas.
4. Ausência de vínculo real entre responsáveis das obrigações e colaboradores locais.
5. Perfil MASTER inconsistente.
6. Build frontend quebrado em IntegraAiPage.tsx.
7. UX de loading da Home ainda fraca.
```

A prioridade imediata deve ser corrigir build, transformar falhas externas do Acessórias em erro controlado, ajustar `DtLastDH` e ampliar a Central para mostrar todas as obrigações por competência, inclusive entregues e canceladas.

