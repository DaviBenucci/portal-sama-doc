# 07 — Prompt para Codex — Pipeline robusta Acessórias, rate limit, persistência local e Push DEV

**Status:** fonte ativa para implementação técnica  
**Data:** 2026-06-09  
**Base externa:** documentação oficial em `https://api.acessorias.com/documentation`  
**Objetivo:** orientar o Codex a implementar a nova arquitetura de sincronização do Acessórias no Portal Sama com fila global, persistência progressiva, categorização imediata, jobs por frequência, uso obrigatório do banco local e notificações Web Push técnicas somente para usuários `DEV`.

---

## 1. Papel do Codex nesta tarefa

Você está trabalhando no projeto **Portal Sama**, em especial na integração com a API do **Acessórias**.

Implemente uma pipeline resiliente de sincronização. Não implemente atalhos que chamem o Acessórias diretamente a partir das telas operacionais. A regra central é:

```txt
Acessórias = fonte externa de origem.
Portal Sama = base operacional local, persistida, auditável e resiliente.
```

O Portal Sama deve continuar exibindo os últimos dados conhecidos mesmo quando a API externa estiver indisponível, em rate limit ou retornando erro temporário.

---

## 2. Documentos que devem ser lidos antes de implementar

Leia estes documentos ativos antes de alterar código:

```txt
docs/00-LEIA-ME-PARA-IA-MVP.md
docs/01-ESTADO-ATUAL-CODIGO-DOCUMENTACAO.md
docs/02-PLANO-FECHAMENTO-MVP.md
docs/03-CONTRATO-ACESSORIAS-OPERACIONAL.md
docs/04-DIVERGENCIAS-DOCS-CODIGO.md
docs/05-PROMPT-CODEX-FECHAMENTO-MVP.md
docs/06-NOTIFICACOES-WEB-PUSH-MVP.md
docs/07-PROMPT-CODEX-PIPELINE-ACESSORIAS-PERSISTENCIA-PUSH-DEV.md
```

Não use `docs/_arquivo/` como requisito ativo. Arquivos arquivados servem apenas como histórico.

---

## 3. Motivação técnica

O comportamento atual pode buscar poucos clientes, poucas obrigações e poucas entregas quando a aplicação atinge o rate limit do Acessórias. A causa provável é a combinação de:

```txt
1. Limite oficial de 100 requisições/minuto com Sliding Window.
2. Delay interno muito próximo do limite real quando configurado como 90/min.
3. Fallback de retry curto demais em HTTP 429.
4. Rate limit implementado por serviço, não como fila global por token/API.
5. Jobs manuais, scheduler, preview, Home e importações podendo concorrer.
6. Fluxos que podem buscar muitos dados antes de persistir tudo.
```

A correção não é apenas aumentar o `sleep`. A correção é mudar a integração para uma pipeline com:

```txt
fila global -> busca paginada -> normalização -> categorização -> persistência imediata -> checkpoint -> reconciliação -> notificação técnica DEV quando necessário
```

---

## 4. Contrato relevante da API Acessórias

A API do Acessórias usa:

```txt
Base URL: https://api.acessorias.com
Autenticação: Authorization: Bearer <token>
Rate limit oficial: 100 requisições por minuto
Algoritmo de limite: Sliding Window
```

### 4.1 `companies`

Endpoint:

```txt
GET /companies/{Identificador}
```

Para listar todas as empresas:

```txt
GET /companies/ListAll?obligations&departments&registrationData&stateRegistrations&Pagina=N
```

Parâmetros úteis:

```txt
obligations       -> obrigações configuradas/snapshot da empresa
departments       -> departamentos e responsáveis da empresa
registrationData  -> regime tributário e grupo da empresa
stateRegistrations-> inscrições estaduais
contacts          -> contatos da empresa, não usar por padrão em carga automática
Pagina            -> paginação; companies retorna 20 registros por página
```

Uso correto no Portal Sama:

```txt
companies/ListAll = carga cadastral mestre.
```

Persistir localmente:

```txt
empresas/clientes
regime tributário
grupo empresarial
inscrições estaduais, se usadas
departamentos da empresa
responsáveis por departamento
catálogo/snapshot local de obrigações da empresa
snapshot bruto sanitizado/controlado
```

### 4.2 `deliveries`

Endpoint:

```txt
GET /deliveries/{Identificador}
```

Incremental recente para todas as empresas:

```txt
GET /deliveries/ListAll?DtInitial=YYYY-MM-DD&DtFinal=YYYY-MM-DD&DtLastDH=YYYY-MM-DD HH:mm:ss&situation=pending,delivered,read&config&Pagina=N
```

Backfill por empresa:

```txt
GET /deliveries/{CNPJ_OU_CPF}?DtInitial=YYYY-MM-DD&DtFinal=YYYY-MM-DD&situation=pending,delivered,read&config&Pagina=N
```

Parâmetros úteis:

```txt
DtInitial     -> data inicial do prazo, obrigatória
DtFinal       -> data final do prazo, obrigatória
DtLastDH      -> alterações após data/hora; obrigatório em ListAll e limitado ao dia atual ou anterior
situation     -> pending, delivered, read
department_id -> filtro opcional por ID de departamento
config        -> obrigatório para trazer departamento, ID da entrega, obrigação/tarefa e responsáveis
Pagina        -> paginação; deliveries retorna 50 registros por página
```

Uso correto no Portal Sama:

```txt
deliveries/ListAll       = sincronização incremental recente.
deliveries/{empresa}     = backfill operacional/histórico por empresa.
companies[].Obrigacoes   = catálogo/snapshot de obrigações, não substitui deliveries.
```

---

## 5. Decisão arquitetural obrigatória

Implemente quatro tipos de job:

```txt
ACESSORIAS_DELIVERIES_INCREMENTAL_SYNC
ACESSORIAS_COMPANY_CATALOG_SYNC
ACESSORIAS_DELIVERIES_BACKFILL_SYNC
ACESSORIAS_RECONCILIATION_JOB
```

A relação entre eles é:

```txt
1. Incremental operacional frequente
   deliveries/ListAll?DtLastDH=hoje/ou/ontem&config&Pagina=N
   Frequência inicial: a cada 30 minutos.
   Frequência futura, se estável: a cada 20 minutos.

2. Carga cadastral mestre
   companies/ListAll?obligations&departments&registrationData&stateRegistrations&Pagina=N
   Frequência: 1 vez ao dia no fim do expediente ou madrugada.
   Pode ser 2 vezes ao dia se o volume for estável.

3. Backfill operacional pesado
   companies/ListAll -> deliveries/{IdentificadorEmpresa}?config&Pagina=N
   Frequência: manual, primeira implantação, madrugada ou rotina controlada.
   Não rodar a cada 20/30 minutos.

4. Reconciliação pesada
   catálogo local de obrigações x entregas operacionais x departamentos x responsáveis x usuários locais
   Frequência: após a carga cadastral pesada, preferencialmente no final do dia.
```

A Central de Vencimentos, Home, painéis de colaborador/gestor e notificações devem consultar somente o banco local.

---

## 6. Requisito 1 — Fila global de requisições ao Acessórias

Crie um serviço centralizado:

```txt
portal-sama-api/src/modules/integrations/acessorias/acessorias-rate-limiter.service.ts
```

Todos os serviços que chamam o Acessórias devem usar esta fila global:

```txt
AcessoriasDeliveriesService
AcessoriasRegistrationsService
AcessoriasHomeService, se ainda existir chamada externa
AcessoriasSchedulerService
Preview/importação/backfill/manual DEV
Qualquer serviço futuro da integração Acessórias
```

Não manter rate limit isolado por classe.

### 6.1 Comportamento esperado

A fila deve garantir:

```txt
concurrency = 1 para chamadas externas ao Acessórias
limite interno padrão = 45 ou 60 requisições/minuto
respeito a Retry-After quando existir
fallback seguro de 65 segundos em HTTP 429 sem Retry-After
retentativas controladas
logs sanitizados
métricas mínimas de fila
```

Configuração recomendada:

```env
ACESSORIAS_RATE_LIMIT_PER_MINUTE=45
ACESSORIAS_RETRY_ATTEMPTS=5
ACESSORIAS_429_FALLBACK_DELAY_SEC=65
ACESSORIAS_TIMEOUT_SEC=60
ACESSORIAS_QUEUE_CONCURRENCY=1
```

Se precisar começar menos conservador, usar `ACESSORIAS_RATE_LIMIT_PER_MINUTE=60`. Não usar `90` em produção até homologar volume real.

### 6.2 Pseudocódigo

```ts
@Injectable()
export class AcessoriasRateLimiterService {
  private readonly queue = new PQueue({
    concurrency: 1,
    interval: 60_000,
    intervalCap: this.rateLimitPerMinute(),
  });

  schedule<T>(task: () => Promise<T>, metadata?: AcessoriasQueueMetadata): Promise<T> {
    return this.queue.add(async () => {
      const startedAt = Date.now();
      try {
        return await task();
      } finally {
        await this.recordQueueMetric(metadata, Date.now() - startedAt);
      }
    });
  }
}
```

Pode usar `p-queue`, `bottleneck` ou implementação própria. Se adicionar dependência, atualizar `package.json`, `package-lock.json` e testes.

### 6.3 Retry de 429

Substituir fallback atual de aproximadamente `1500ms` por regra segura:

```ts
private retryDelayMs(response: Response): number {
  const retryAfter = response.headers.get('retry-after');

  if (retryAfter) {
    const seconds = Number(retryAfter);
    if (Number.isFinite(seconds) && seconds > 0) {
      return Math.min(seconds * 1000, 5 * 60_000);
    }

    const retryDate = new Date(retryAfter);
    const delay = retryDate.getTime() - Date.now();
    if (Number.isFinite(delay) && delay > 0) {
      return Math.min(delay, 5 * 60_000);
    }
  }

  return this.configured429FallbackMs(); // default: 65_000
}
```

Retentativas recomendadas:

```txt
Tentativa 1 -> 65s
Tentativa 2 -> 90s
Tentativa 3 -> 120s
Tentativa 4 -> 180s
Tentativa 5 -> falha controlada, checkpoint preservado e push DEV se for job pesado
```

---

## 7. Requisito 2 — Lock global para jobs pesados

Crie ou consolide um serviço:

```txt
AcessoriasSyncLockService
```

Objetivo: impedir que jobs pesados concorram entre si.

Jobs pesados:

```txt
ACESSORIAS_COMPANY_CATALOG_SYNC
ACESSORIAS_DELIVERIES_BACKFILL_SYNC
ACESSORIAS_RECONCILIATION_JOB
Importação manual completa
Preview paginado amplo, se consumir muitas páginas
```

Regra:

```txt
Não permitir backfill + carga cadastral + scheduler pesado ao mesmo tempo.
Não permitir dois backfills simultâneos.
Não bloquear incremental leve se o job pesado permitir, salvo quando o lock estiver em modo exclusivo.
```

Modelo sugerido:

```prisma
model AcessoriasExternalJobLock {
  id          String   @id @default(uuid())
  lockKey     String   @unique @map("lock_key") @db.VarChar(120)
  jobType     String   @map("job_type") @db.VarChar(80)
  ownerId     String?  @map("owner_id") @db.VarChar(191)
  acquiredAt  DateTime @default(now()) @map("acquired_at")
  expiresAt   DateTime @map("expires_at")
  metadata    Json?

  @@index([expiresAt])
  @@map("acessorias_external_job_locks")
}
```

O lock deve expirar automaticamente para evitar travamento permanente se o processo cair.

---

## 8. Requisito 3 — Persistência progressiva por página/empresa

Não implemente fluxo que busca tudo em memória para salvar apenas no final.

Errado:

```txt
buscar 400 empresas
buscar milhares de entregas
guardar tudo em memória
salvar no final
```

Correto:

```txt
buscar página 1
normalizar página 1
categorizar página 1
persistir página 1
atualizar checkpoint
continuar página 2
```

Para backfill por empresa:

```txt
empresa 1 / página 1 -> persistir -> checkpoint
empresa 1 / página 2 -> persistir -> checkpoint
empresa 2 / página 1 -> persistir -> checkpoint
...
```

Se ocorrer erro, manter status como:

```txt
FAILED_PARTIAL
```

ou equivalente, preservando:

```txt
última empresa processada
última página processada
último DtLastDH usado
quantidade criada/atualizada/pulada
erro sanitizado
próxima tentativa sugerida
```

---

## 9. Requisito 4 — Modelo de sync run/checkpoint

O modelo atual `AcessoriasDeliverySyncRun` pode ser mantido por compatibilidade, mas a nova arquitetura precisa registrar jobs além de deliveries. Preferir criar um modelo genérico:

```prisma
model AcessoriasSyncRun {
  id                String    @id @default(uuid())
  jobType           String    @map("job_type") @db.VarChar(80)
  status            String    @default("RUNNING") @db.VarChar(40)
  trigger           String    @default("manual") @db.VarChar(40)
  scope             String?   @db.VarChar(80)
  startedAt         DateTime  @default(now()) @map("started_at")
  finishedAt        DateTime? @map("finished_at")
  nextRetryAt       DateTime? @map("next_retry_at")
  fetched           Int       @default(0)
  created           Int       @default(0)
  updated           Int       @default(0)
  skipped           Int       @default(0)
  failed            Int       @default(0)
  currentPage       Int?      @map("current_page")
  currentIdentifier String?   @map("current_identifier") @db.VarChar(191)
  checkpoint         Json?
  errorCode          String?   @map("error_code") @db.VarChar(80)
  errorMessage       String?   @map("error_message") @db.Text
  metadata           Json?

  @@index([jobType, status])
  @@index([startedAt])
  @@index([nextRetryAt])
  @@map("acessorias_sync_runs")
}
```

Status permitidos:

```txt
RUNNING
SUCCESS
FAILED
FAILED_PARTIAL
SKIPPED
PAUSED_RATE_LIMIT
CANCELED
```

Se preferir não criar modelo genérico nesta etapa, estender `AcessoriasDeliverySyncRun` e criar um segundo run para cadastro. Porém a solução final deve convergir para `AcessoriasSyncRun`.

---

## 10. Requisito 5 — Categorização imediata dos dados

Cada registro recebido deve ser categorizado antes ou durante a persistência. Não deixar a categorização para uma etapa manual posterior.

### 10.1 Empresas

Origem:

```txt
companies/ListAll
```

Persistir/categorizar:

```txt
client_id local
Identificador normalizado
Razao
Fantasia
Status
UF
Telefone, se necessário
Regime
GrupoDeEmpresas
ClienteDesde
ClienteAte
DataDoCadastro
metadata.acessorias.lastSyncedAt
metadata.acessorias.rawSnapshot sanitizado
```

### 10.2 Departamentos e responsáveis

Origem:

```txt
companies[].Departamentos
```

Persistir/categorizar:

```txt
client_id
department_external_id
department_name
normalized_department_name
responsible_name
responsible_email
responsible_user_id, se match seguro
match_status
match_score
last_synced_at
raw_snapshot sanitizado
```

Regras de segurança:

```txt
Não criar usuário ativo automaticamente.
Não conceder permissões automaticamente.
Criar alias pendente quando não houver match seguro.
Exigir revisão humana para vínculo duvidoso.
```

### 10.3 Catálogo de obrigações da empresa

Origem:

```txt
companies[].Obrigacoes
```

Criar tabela local. Nome sugerido:

```txt
AcessoriasCompanyObligationCatalog
```

Modelo sugerido:

```prisma
model AcessoriasCompanyObligationCatalog {
  id                 String   @id @default(uuid())
  clientId           String?  @map("client_id")
  clientDocument     String   @map("client_document") @db.VarChar(20)
  clientName         String?  @map("client_name") @db.VarChar(255)
  obligationName     String   @map("obligation_name") @db.VarChar(255)
  normalizedName     String   @map("normalized_name") @db.VarChar(255)
  status             String?  @db.VarChar(80)
  deliveredCount     Int      @default(0) @map("delivered_count")
  overdueCount       Int      @default(0) @map("overdue_count")
  next30DaysCount    Int      @default(0) @map("next_30_days_count")
  future30PlusCount  Int      @default(0) @map("future_30_plus_count")
  sourceRunId        String?  @map("source_run_id")
  rawPayload         Json?    @map("raw_payload")
  firstSyncedAt      DateTime @default(now()) @map("first_synced_at")
  lastSyncedAt       DateTime @updatedAt @map("last_synced_at")

  @@unique([clientDocument, normalizedName], map: "uq_acessorias_company_obligation_catalog")
  @@index([clientId])
  @@index([status])
  @@index([normalizedName])
  @@map("acessorias_company_obligation_catalog")
}
```

Semântica:

```txt
Esta tabela representa o que a empresa tem configurado no cadastro do Acessórias.
Ela não substitui AcessoriasDelivery.
Ela permite auditoria de coerência entre obrigação configurada e entrega operacional existente.
```

### 10.4 Entregas/vencimentos operacionais

Origem:

```txt
deliveries/ListAll
deliveries/{Identificador}
```

Persistir/categorizar em `AcessoriasDelivery`:

```txt
externalId / Config.EntID
client_id
client_document
client_name
obligation_name
competence
status normalizado
raw_status
due_at
delivered_at
department / Config.DptoNome
responsible_name
responsible_username/responsible_email quando disponível
responsible_user_id se match seguro
responsible_match_status
last_external_change_at / EntLastDH, se adicionar campo
raw_payload sanitizado
source_run_id, se adicionar campo
```

Se o campo ainda não existir, avaliar migration para:

```prisma
lastExternalChangedAt DateTime? @map("last_external_changed_at")
sourceRunId           String?   @map("source_run_id")
externalDepartmentId  String?   @map("external_department_id") @db.VarChar(80)
externalObligationId  String?   @map("external_obligation_id") @db.VarChar(80)
```

---

## 11. Requisito 6 — Frequência dos jobs

Configurar frequências por `.env`:

```env
ACESSORIAS_RATE_LIMIT_PER_MINUTE=45
ACESSORIAS_RETRY_ATTEMPTS=5
ACESSORIAS_429_FALLBACK_DELAY_SEC=65
ACESSORIAS_TIMEOUT_SEC=60
ACESSORIAS_HOME_APPEND_CONTEXT_QUERY=false
ACESSORIAS_INCLUDE_CONTACTS=false
ACESSORIAS_INCLUDE_ATTACHMENTS=false

ACESSORIAS_INCREMENTAL_SYNC_ENABLED=true
ACESSORIAS_INCREMENTAL_SYNC_INTERVAL_MINUTES=30
ACESSORIAS_INCREMENTAL_SYNC_LOOKBACK_MINUTES=90

ACESSORIAS_COMPANY_CATALOG_SYNC_ENABLED=true
ACESSORIAS_COMPANY_CATALOG_SYNC_CRON="0 22 * * *"
ACESSORIAS_COMPANY_CATALOG_SYNC_SECONDARY_CRON="0 2 * * *"

ACESSORIAS_BACKFILL_SYNC_ENABLED=false
ACESSORIAS_BACKFILL_SYNC_CRON="0 2 * * *"
ACESSORIAS_BACKFILL_SYNC_MANUAL_ONLY=true

ACESSORIAS_RECONCILIATION_ENABLED=true
ACESSORIAS_RECONCILIATION_CRON="30 23 * * *"
```

O padrão de produção deve começar conservador. `45/min` reduz risco de rate limit e concorrência com outros sistemas usando o mesmo token. Subir para `60/min` somente após homologação. Não usar `90/min` como padrão. Backfill automático deve começar desabilitado; habilitar apenas depois que checkpoint, lock e push DEV estiverem homologados.

### 11.1 Incremental operacional frequente

Rodar a cada 30 minutos inicialmente:

```txt
ACESSORIAS_DELIVERIES_INCREMENTAL_SYNC
```

Usar:

```txt
deliveries/ListAll?DtLastDH=...&config&Pagina=N
```

Regra:

```txt
DtLastDH deve ser hoje ou ontem.
Se o último sync bem-sucedido for mais antigo que ontem, não usar ListAll para recuperar histórico.
Abrir backfill por empresa ou marcar necessidade de backfill controlado.
```

Após homologação, pode reduzir para 20 minutos:

```env
ACESSORIAS_INCREMENTAL_SYNC_INTERVAL_MINUTES=20
```

### 11.2 Sincronização por departamento

O Acessórias aceita `department_id` em `deliveries`. Use esse filtro apenas quando houver catálogo confiável de IDs de departamento e quando a redução de volume justificar.

Opções aceitas:

```txt
Opção A — recomendada no início:
1. Buscar deliveries/ListAll com config=true.
2. Categorizar localmente por Config.DptoNome e Config.DptoID.
3. Atualizar telas por departamento usando o banco local.

Opção B — otimização futura:
1. Manter catálogo local de departamentos com ID externo.
2. Rodar sync incremental segmentado por department_id.
3. Escalonar departamentos em rodízio para não multiplicar chamadas.
```

Não fazer chamadas por departamento se isso aumentar o número total de requisições sem necessidade.

### 11.3 Carga cadastral pesada

Rodar 1 vez ao dia, preferencialmente fim do dia ou madrugada:

```txt
ACESSORIAS_COMPANY_CATALOG_SYNC
```

Pode rodar 2 vezes ao dia após homologação:

```txt
02:00 e 22:00
```

Essa carga atualiza empresas, departamentos, responsáveis, regime, grupo e catálogo de obrigações.

### 11.4 Backfill pesado

Rodar:

```txt
primeira implantação
manual pela área DEV
madrugada
quando houver inconsistência detectada
rotina semanal ou diária controlada, se necessário
```

Nunca rodar backfill completo a cada 20 ou 30 minutos.

### 11.5 Reconciliação final do dia

Após carga cadastral e/ou backfill, rodar:

```txt
ACESSORIAS_RECONCILIATION_JOB
```

Objetivo:

```txt
catálogo de obrigações da empresa x entregas operacionais
responsáveis externos x usuários locais
entregas sem catálogo correspondente
obrigações ativas sem entrega futura recente
empresas sem responsável por departamento obrigatório
dados desatualizados acima do SLA
```

---

## 12. Requisito 7 — Tela de vencimentos e Home sempre pelo banco local

A Central de Vencimentos, Home, painel do colaborador e painel do gestor não devem chamar o Acessórias diretamente.

Fluxo correto:

```txt
Job de sincronização
  -> API Acessórias
  -> fila global
  -> persistência local
  -> categorização
  -> reconciliação
  -> telas consultam banco local
```

Se a API externa estiver indisponível:

```txt
1. Manter a tela funcionando com últimos dados persistidos.
2. Exibir `ultima_sincronizacao`.
3. Exibir alerta de dados possivelmente desatualizados quando ultrapassar SLA.
4. Não mostrar erro bruto do Acessórias para usuário final.
```

---

## 13. Requisito 8 — Notificação Web Push somente para DEV em jobs pesados

Adicionar notificação técnica para usuários com role `DEV`, usando a infraestrutura existente de notificações/Web Push.

Arquivos existentes úteis:

```txt
portal-sama-api/src/modules/notifications/notifications.service.ts
portal-sama-api/src/modules/notifications/notifications-push.service.ts
portal-sama-api/src/modules/notifications/notifications.controller.ts
portal-sama-web/src/services/notifications.service.ts
portal-sama-web/public/sama-push-sw.js
```

Crie um serviço:

```txt
portal-sama-api/src/modules/integrations/acessorias/acessorias-dev-notification.service.ts
```

### 13.1 Eventos DEV obrigatórios

```txt
ACESSORIAS_HEAVY_SYNC_STARTED
ACESSORIAS_HEAVY_SYNC_COMPLETED
ACESSORIAS_HEAVY_SYNC_FAILED
ACESSORIAS_RATE_LIMIT_REACHED
ACESSORIAS_BACKFILL_STARTED
ACESSORIAS_BACKFILL_COMPLETED
ACESSORIAS_BACKFILL_FAILED
ACESSORIAS_RECONCILIATION_STARTED
ACESSORIAS_RECONCILIATION_COMPLETED
ACESSORIAS_RECONCILIATION_FAILED
ACESSORIAS_PARTIAL_SYNC_SAVED
ACESSORIAS_SYNC_RESUMED
```

### 13.2 Quem recebe

Somente usuários ativos com role `DEV`.

Não usar `dept = SISTEMA` para esse requisito, porque o push atual pode expandir departamentos técnicos para `DEV`, `ADMIN` e `TI`. Para garantir DEV-only, criar uma notificação individual por `username` de cada usuário DEV.

Pseudocódigo:

```ts
async notifyDevUsers(input: AcessoriasDevNotificationInput) {
  if (!this.config.devPushEnabled()) return;

  const devUsers = await this.prisma.user.findMany({
    where: {
      status: 'ACTIVE',
      roles: {
        some: { role: { name: 'DEV' } },
      },
    },
    select: { id: true, username: true },
  });

  for (const devUser of devUsers) {
    await this.notificationsService.create({
      dto: {
        username: devUser.username,
        title: input.title,
        body: input.body,
        level: input.level,
        module: 'integrations.acessorias',
        eventType: input.eventType,
        entityType: input.entityType ?? 'acessorias_sync_run',
        entityId: input.entityId,
        dedupeKey: `${input.dedupeKey}:${devUser.username}`,
        meta: input.safeMeta,
      },
      user: this.systemActor(),
      request: {
        ipAddress: 'system',
        userAgent: 'acessorias-sync-pipeline',
      },
    });
  }
}
```

### 13.3 Payload seguro

Push não pode conter:

```txt
CNPJ completo
CPF
nomes completos de clientes se não for necessário
token
Authorization header
payload bruto do Acessórias
link de anexo
e-mail de responsável
stack trace
SQL
dados de documento empresarial
```

Push pode conter:

```txt
tipo do job
status
horário resumido
ID interno do sync run
orientação para abrir área DEV
resumo numérico não sensível
```

Exemplos corretos:

```txt
Título: Sincronização pesada iniciada
Corpo: Um job técnico do Acessórias foi iniciado. Acompanhe pela área DEV.

Título: Rate limit atingido
Corpo: A fila do Acessórias foi pausada e será retomada automaticamente.

Título: Backfill concluído
Corpo: O job pesado do Acessórias foi finalizado. Verifique o relatório técnico.
```

### 13.4 Deduplicação

Para eventos frequentes, deduplicar:

```txt
Rate limit: no máximo 1 push por hora
Falha repetida do mesmo job: no máximo 1 push por job/status
Partial sync: no máximo 1 push por run
```

Exemplos de `dedupeKey`:

```txt
acessorias:rate-limit:YYYY-MM-DD-HH
acessorias:heavy-sync:failed:{runId}
acessorias:backfill:completed:{runId}
acessorias:partial-sync:{runId}
```

### 13.5 Configuração

Adicionar ao `.env.example`:

```env
ACESSORIAS_DEV_PUSH_ENABLED=true
ACESSORIAS_DEV_PUSH_ON_STARTED=true
ACESSORIAS_DEV_PUSH_ON_COMPLETED=true
ACESSORIAS_DEV_PUSH_ON_FAILED=true
ACESSORIAS_DEV_PUSH_ON_RATE_LIMIT=true
ACESSORIAS_DEV_PUSH_DEDUPE_MINUTES=60
```

Garantir que Web Push continue usando:

```env
WEB_PUSH_ENABLED=true
WEB_PUSH_PUBLIC_KEY=
WEB_PUSH_PRIVATE_KEY=
WEB_PUSH_SUBJECT=mailto:suporte@samacontabil.com.br
```

ou aliases existentes:

```env
WEB_PUSH_VAPID_PUBLIC_KEY=
WEB_PUSH_VAPID_PRIVATE_KEY=
WEB_PUSH_CONTACT=mailto:suporte@samacontabil.com.br
```

### 13.6 Atualização do payload seguro do Web Push

Atualizar `NotificationsPushService.toSafePushMessage()` para tratar os novos eventos técnicos sem vazar dados:

```ts
if (eventType.startsWith('ACESSORIAS_HEAVY_SYNC_') ||
    eventType.startsWith('ACESSORIAS_BACKFILL_') ||
    eventType.startsWith('ACESSORIAS_RECONCILIATION_') ||
    eventType === 'ACESSORIAS_RATE_LIMIT_REACHED' ||
    eventType === 'ACESSORIAS_PARTIAL_SYNC_SAVED' ||
    eventType === 'ACESSORIAS_SYNC_RESUMED') {
  return {
    title: 'Atualizacao tecnica Acessorias',
    body: 'Uma rotina tecnica da integracao precisa de acompanhamento.',
  };
}
```

Adicionar os novos tipos ao conjunto de preferências padrão de Web Push, se necessário.

---

## 14. Requisito 9 — Implementação dos fluxos

### 14.1 `syncCompaniesCatalog()`

Arquivo provável:

```txt
portal-sama-api/src/modules/integrations/acessorias/acessorias-registrations.service.ts
```

Implementar ou consolidar:

```ts
async syncCompaniesCatalog(input: SyncCompaniesCatalogInput): Promise<SyncRunResponse>
```

Fluxo:

```txt
1. Criar sync run com jobType=ACESSORIAS_COMPANY_CATALOG_SYNC.
2. Adquirir lock de job pesado.
3. Notificar DEV: ACESSORIAS_HEAVY_SYNC_STARTED.
4. Para Pagina = checkpoint.page ou 1:
   4.1. Chamar companies/ListAll com obligations, departments, registrationData, stateRegistrations.
   4.2. Se lista vazia ou 204, finalizar.
   4.3. Para cada empresa:
        - upsert Client;
        - persistir regime/grupo/metadados;
        - persistir departamentos/responsáveis/aliases;
        - upsert AcessoriasCompanyObligationCatalog;
        - registrar contadores.
   4.4. Atualizar checkpoint da página.
5. Finalizar run SUCCESS.
6. Notificar DEV: ACESSORIAS_HEAVY_SYNC_COMPLETED.
7. Liberar lock.
```

Se houver erro:

```txt
1. Persistir progresso já obtido.
2. Marcar FAILED_PARTIAL se houve dados salvos.
3. Salvar erro sanitizado.
4. Notificar DEV se job pesado.
5. Liberar lock.
```

### 14.2 `syncIncrementalFromListAll()`

Arquivo provável:

```txt
portal-sama-api/src/modules/integrations/acessorias/acessorias-deliveries.service.ts
```

Consolidar com persistência por página.

Fluxo:

```txt
1. Calcular DtLastDH seguro.
2. Se DtLastDH for mais antigo que ontem, não usar ListAll para histórico.
3. Criar sync run jobType=ACESSORIAS_DELIVERIES_INCREMENTAL_SYNC.
4. Para Pagina = 1..N:
   4.1. Chamar deliveries/ListAll com DtInitial, DtFinal, DtLastDH, config, situation e Pagina.
   4.2. Se vazio, finalizar.
   4.3. Normalizar entregas.
   4.4. Categorizar por empresa, departamento, responsável, obrigação, competência e status.
   4.5. Upsert em AcessoriasDelivery imediatamente.
   4.6. Resolver responsável local ou alias pendente.
   4.7. Atualizar checkpoint.
5. Gerar notificações operacionais conforme regras existentes.
6. Finalizar run.
```

### 14.3 `backfillDeliveriesByCompany()`

Arquivo provável:

```txt
portal-sama-api/src/modules/integrations/acessorias/acessorias-deliveries.service.ts
```

Consolidar com checkpoint por empresa e página.

Fluxo:

```txt
1. Criar sync run jobType=ACESSORIAS_DELIVERIES_BACKFILL_SYNC.
2. Adquirir lock de job pesado.
3. Notificar DEV: ACESSORIAS_BACKFILL_STARTED.
4. Obter lista de empresas local ou companies/ListAll.
5. Para cada empresa a partir do checkpoint:
   5.1. Para cada página de deliveries/{Identificador}:
        - buscar;
        - normalizar;
        - categorizar;
        - persistir;
        - atualizar checkpoint.
6. Finalizar SUCCESS.
7. Notificar DEV: ACESSORIAS_BACKFILL_COMPLETED.
8. Liberar lock.
```

### 14.4 `reconcileAcessoriasData()`

Criar serviço novo se não existir:

```txt
portal-sama-api/src/modules/integrations/acessorias/acessorias-reconciliation.service.ts
```

Objetivo:

```txt
comparar catálogo local de obrigações da empresa com entregas operacionais
identificar obrigação ativa sem entrega recente/futura
identificar entrega sem obrigação ativa correspondente no catálogo
identificar empresas sem responsável por departamento
identificar responsáveis externos sem vínculo local confirmado
identificar dados desatualizados por SLA
registrar divergências ou alertas DEV/operacionais
```

Não usar a API externa nesta etapa, salvo necessidade explícita. A reconciliação deve ser preferencialmente local.

---

## 15. Requisito 10 — Endpoints DEV e RBAC

Adicionar ou consolidar endpoints protegidos:

```txt
POST /api-v2/integrations/acessorias/companies/catalog/sync
POST /api-v2/integrations/acessorias/deliveries/sync
POST /api-v2/integrations/acessorias/deliveries/backfill
POST /api-v2/integrations/acessorias/reconciliation/run
GET  /api-v2/integrations/acessorias/sync-runs
GET  /api-v2/integrations/acessorias/sync-runs/:id
GET  /api-v2/integrations/acessorias/rate-limiter/status
GET  /api-v2/integrations/acessorias/scheduler/status
```

Permissões sugeridas:

```txt
integrations.acessorias.read
integrations.acessorias.sync
integrations.acessorias.sync_manual
integrations.acessorias.backfill
integrations.acessorias.reconcile
integrations.acessorias.dev_notifications.read
```

Somente `DEV` e perfis explicitamente autorizados devem executar jobs pesados.

---

## 16. Requisito 11 — Tratamento de erro e segurança

Tratar pelo menos:

```txt
AbortError
TypeError/fetch failed
EAI_AGAIN
ECONNRESET
ECONNREFUSED
JSON inválido
401
403
429
5xx
resposta vazia
payload com estrutura inesperada
```

Não vazar:

```txt
token
Authorization header
payload bruto completo em logs
stack trace para frontend
CNPJ/CPF completo em push
anexos ou links temporários
VAPID private key
```

Armazenar erro sanitizado:

```txt
errorCode
safeErrorMessage
httpStatus, se houver
retryScheduledAt, se houver
runId
jobType
checkpoint resumido
```

---

## 17. Requisito 12 — Testes obrigatórios

Adicionar ou atualizar testes unitários/contratuais.

### 17.1 Rate limiter

Testar:

```txt
usa fila global
respeita concurrency 1
usa intervalCap configurado
parseia Retry-After em segundos
parseia Retry-After em data HTTP
usa fallback de 65s quando não há Retry-After
não loga token
```

### 17.2 Persistência progressiva

Testar:

```txt
salva página 1 antes de buscar página 2
atualiza checkpoint por página
marca FAILED_PARTIAL se falhar após salvar dados
retoma a partir de checkpoint
```

### 17.3 Companies catalog

Testar:

```txt
companies/ListAll usa obligations, departments, registrationData e stateRegistrations
persiste Client
persiste catálogo de obrigações
persiste/gera alias de responsável
não cria usuário ativo automaticamente
```

### 17.4 Deliveries incremental

Testar:

```txt
ListAll usa DtLastDH apenas hoje/ontem
quando último sync é antigo, não usa DtLastDH inválido para histórico
usa config=true
persiste entregas por página
categoriza departamento e responsável
```

### 17.5 Backfill

Testar:

```txt
usa deliveries/{Identificador}
persiste por empresa/página
mantém checkpoint de empresa atual
não roda concorrente com outro job pesado
```

### 17.6 Push DEV

Testar:

```txt
notifica somente usuários com role DEV
não usa dept=SISTEMA para DEV-only
aplica dedupeKey
payload Web Push é sanitizado
não inclui CNPJ/CPF/token/payload bruto
```

Comandos esperados:

```bash
cd portal-sama-api
npm run build
npm test -- --runInBand

cd portal-sama-web
npm run build
npm run lint
npm test -- --runInBand
```

---

## 18. Critérios de aceite

A implementação só está aceita quando:

```txt
1. Toda chamada ao Acessórias passa pela fila global.
2. HTTP 429 sem Retry-After espera pelo menos 65 segundos antes de nova tentativa.
3. Jobs pesados não executam concorrentemente.
4. Companies catalog persiste empresas, regime, grupos, departamentos, responsáveis e catálogo de obrigações.
5. Deliveries persiste entregas/vencimentos por página.
6. Nenhum fluxo crítico depende de manter todos os dados em memória até o final.
7. Central de Vencimentos e Home usam banco local.
8. Incremental roda com frequência configurável de 30 minutos.
9. Carga cadastral pesada roda 1 ou 2 vezes ao dia.
10. Backfill pesado roda manualmente, de madrugada ou sob lock controlado.
11. Reconciliação final do dia usa dados locais e gera divergências/alertas.
12. Push técnico é enviado somente para usuários DEV.
13. Push técnico não contém dados sensíveis.
14. Sync runs registram checkpoint, totais, status e erro sanitizado.
15. Testes e builds passam.
```

---

## 19. Fora do escopo desta instrução

Não implementar nesta tarefa:

```txt
WhatsApp, Slack, Teams ou SMS
IA automática decidindo responsável sem revisão
alteração bidirecional no Acessórias
upload/download automático de anexos do Acessórias
uso de contacts por padrão na carga automática
criação automática de usuário ativo a partir de RespEmail
redesign amplo de telas
```

---

## 20. Resumo executivo para implementação

Implemente a integração como uma pipeline robusta:

```txt
Acessórias
  -> AcessoriasRateLimiterService global
  -> sync job com lock
  -> busca paginada
  -> normalização imediata
  -> categorização imediata
  -> persistência local por página/empresa
  -> checkpoint
  -> reconciliação local
  -> notificações operacionais e push técnico DEV
  -> telas lendo somente banco local
```

A prioridade é confiabilidade, rastreabilidade, segurança e continuidade operacional. O objetivo não é buscar tudo mais rápido; é buscar sem perder dados, sem estourar rate limit, sem travar a aplicação e sem depender da disponibilidade em tempo real da API externa.
