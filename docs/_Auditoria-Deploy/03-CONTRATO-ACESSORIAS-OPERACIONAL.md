# 03 — Contrato Acessórias operacional para o Portal Sama

**Status:** fonte ativa  
**Data:** 2026-06-08  
**Base:** documentação pública da API Acessórias em `https://api.acessorias.com/documentation`  
**Decisão central:** usar uma arquitetura híbrida: `companies` para cadastro/configuração e `deliveries` para operação/vencimentos, sempre persistindo os dados no banco local do Portal Sama.

---

## 1. Decisão arquitetural oficial

O Portal Sama não deve depender de chamadas em tempo real ao Acessórias para renderizar telas operacionais. O Acessórias é a **fonte externa de origem**, mas o Portal Sama precisa manter uma **base local persistida, auditável e resiliente**.

```txt
Acessórias = fonte externa de dados cadastrais e operacionais
Portal Sama = fonte operacional local para telas, filtros, notificações, auditoria e histórico
```

A integração deve ser separada em dois domínios:

| Domínio | Fonte Acessórias | Uso principal no Portal Sama | Persistência local obrigatória |
|---|---|---|---|
| Cadastro/configuração da empresa | `companies/ListAll?obligations&departments&registrationData` | Empresas, regime tributário, grupos, departamentos, responsáveis e catálogo de obrigações da empresa | `clients`, atribuições por departamento, aliases de responsáveis, catálogo local de obrigações da empresa e snapshot bruto controlado |
| Operação/vencimentos | `deliveries/ListAll` e `deliveries/{Identificador}` | Prazos, competência, status, baixa, atraso, responsável do prazo, responsável da entrega e notificações | `acessorias_deliveries`, sync runs, status normalizado, payload bruto sanitizado |
| Catálogo auxiliar | `departments/ListAll` | Catálogo geral de departamentos, quando necessário | catálogo local de departamentos, quando houver correspondência segura |

Regra definitiva:

```txt
companies/ListAll + obligations + departments + registrationData
= o que a empresa é, como ela está configurada e quais obrigações possui.

deliveries/ListAll ou deliveries/{empresa}
= o que está vencendo, venceu, foi entregue, mudou ou precisa notificar alguém.
```

---

## 2. Autenticação, rate limit e segurança de tráfego

A API do Acessórias usa:

```txt
Base URL: https://api.acessorias.com
Header: Authorization: Bearer <token>
Rate limit oficial: 100 requisições por minuto
Algoritmo: Sliding Window
```

Regras do Portal Sama:

- o token deve existir somente no backend;
- o frontend nunca deve conhecer token, header de autenticação nem URL com segredo;
- logs não podem registrar token, header `Authorization` ou payload sensível completo;
- o backend deve aplicar throttling interno abaixo da cota oficial;
- requisições automáticas devem ter fila, lock de execução e retentativa controlada.

Configuração recomendada para produção:

```env
ACESSORIAS_BASE_URL="https://api.acessorias.com"
ACESSORIAS_TOKEN="..."
ACESSORIAS_HOME_PATH="deliveries/ListAll"
ACESSORIAS_DELIVERIES_PATH="deliveries/ListAll"
ACESSORIAS_CLIENTS_PATH="companies/ListAll"
ACESSORIAS_COLLABORATORS_PATH=""
ACESSORIAS_AUTH_HEADER="Authorization"
ACESSORIAS_AUTH_SCHEME="Bearer"
ACESSORIAS_HOME_APPEND_CONTEXT_QUERY="false"
ACESSORIAS_TIMEOUT_SEC="45"
ACESSORIAS_RATE_LIMIT_PER_MINUTE="60"
ACESSORIAS_MAX_PAGES="1000"
ACESSORIAS_SCHEDULER_ENABLED="true"
ACESSORIAS_SCHEDULER_NOTIFICATIONS_ENABLED="true"
```

Observações:

- `ACESSORIAS_RATE_LIMIT_PER_MINUTE=60` é intencionalmente conservador. A cota oficial é 100/minuto, mas 60/minuto reduz risco de bloqueios por WAF/firewall e concorrência com outros sistemas.
- `ACESSORIAS_HOME_APPEND_CONTEXT_QUERY=false` evita envio desnecessário de dados internos do usuário para serviço externo.
- `ACESSORIAS_COLLABORATORS_PATH` deve permanecer vazio por padrão, pois os responsáveis devem ser extraídos de `companies.ListAll[].Departamentos` e conciliados localmente.

---

## 3. `companies` como fonte cadastral mestre

Endpoint:

```txt
GET /companies/{Identificador}
```

Uso para listar todas as empresas:

```txt
GET /companies/ListAll?obligations&departments&registrationData&stateRegistrations&Pagina=1
```

Uso detalhado/manual, somente quando a aplicação realmente precisar de contatos:

```txt
GET /companies/{CNPJ_OU_CPF}?obligations&departments&registrationData&stateRegistrations&contacts&Pagina=1
```

A documentação do Acessórias define `Identificador` como CNPJ/CPF da empresa ou `ListAll` para listagem paginada. A paginação de `companies` retorna 20 registros por página e deve continuar até vir lista vazia.

### 3.1 Query params relevantes

| Query param | Uso no Portal Sama | Regra |
|---|---|---|
| `obligations` | Trazer o catálogo/snapshot de obrigações da empresa | Deve alimentar catálogo local de obrigações, não substituir `deliveries` |
| `departments` | Trazer departamentos da empresa e responsáveis | Deve alimentar vínculos/aliases de responsáveis por departamento |
| `registrationData` | Trazer regime tributário e grupo empresarial | Deve alimentar campos cadastrais locais da empresa |
| `stateRegistrations` | Trazer inscrições estaduais | Pode ir para `Client.metadata.acessorias.inscricoesEstaduais` ou tabela própria futura |
| `contacts` | Trazer contatos da empresa | Não usar na carga automática padrão sem necessidade operacional explícita |
| `Pagina` | Paginação | Obrigatório no loop de sincronização |

### 3.2 Campos esperados de empresa

Campos relevantes do retorno:

```txt
ID
Identificador
Razao
Fantasia
Status
Telefone
UF
ClienteDesde
ClienteAte
DataDoCadastro
Regime
GrupoDeEmpresas
InscricoesEstaduais[]
ContatosNaEmpresa[]
Departamentos[]
Obrigacoes[]
```

### 3.3 Departamentos e responsáveis

Quando `departments` é enviado, o retorno pode trazer:

```txt
Departamentos[].ID
Departamentos[].Nome
Departamentos[].RespNome
Departamentos[].RespEmail
```

Uso correto:

1. normalizar o departamento;
2. normalizar `RespEmail` e `RespNome`;
3. procurar usuário local por e-mail exato;
4. se não houver match seguro, criar/atualizar alias pendente;
5. nunca criar usuário ativo automaticamente;
6. registrar origem e snapshot do vínculo.

### 3.4 Obrigações de `companies`

Quando `obligations` é enviado, o retorno pode trazer:

```txt
Obrigacoes[].Nome
Obrigacoes[].Status
Obrigacoes[].Entregues
Obrigacoes[].Atrasadas
Obrigacoes[].Proximos30D
Obrigacoes[].Futuras30+
```

Semântica no Portal Sama:

```txt
Obrigacoes de companies = catálogo/snapshot de obrigações configuradas para a empresa.
```

Esses dados devem responder perguntas como:

- esta empresa possui esta obrigação cadastrada no Acessórias?
- a obrigação está ativa ou inativa nessa empresa?
- quantas entregas aparecem como entregues, atrasadas, próximas ou futuras no snapshot cadastral?
- existe divergência entre catálogo da empresa e entregas operacionais recentes?

Esses dados **não** substituem `deliveries`, pois não trazem todos os campos operacionais necessários para a Central de Vencimentos, como `EntCompetencia`, `EntDtPrazo`, `EntDtEntrega`, `EntLastDH`, `Config.EntID`, `Config.DptoNome`, `Config.RespPrazo` e `Config.RespEntrega`.

---

## 4. `deliveries` como fonte operacional da Central de Vencimentos

Endpoint:

```txt
GET /deliveries/{Identificador}
```

Uso incremental para todas as empresas:

```txt
GET /deliveries/ListAll?DtInitial=YYYY-MM-DD&DtFinal=YYYY-MM-DD&DtLastDH=YYYY-MM-DD HH:mm:ss&situation=pending,delivered,read&config&Pagina=1
```

Uso de backfill por empresa:

```txt
GET /deliveries/{CNPJ_OU_CPF}?DtInitial=YYYY-MM-DD&DtFinal=YYYY-MM-DD&situation=pending,delivered,read&config&Pagina=1
```

Regra crítica:

```txt
Quando Identificador = ListAll, DtLastDH é obrigatório e só aceita o dia atual ou o dia anterior.
```

Portanto:

```txt
deliveries/ListAll não é carga histórica completa.
deliveries/ListAll é sincronização incremental recente.
deliveries/{IdentificadorEmpresa} é o caminho de backfill/carga por empresa.
```

A paginação de `deliveries` retorna 50 registros por página e deve continuar até vir lista vazia.

### 4.1 Query params relevantes

| Query param | Uso no Portal Sama | Regra |
|---|---|---|
| `DtInitial` | início da janela por data de prazo | obrigatório |
| `DtFinal` | fim da janela por data de prazo | obrigatório |
| `DtLastDH` | alterações após data/hora específica | obrigatório somente em `ListAll`; em `ListAll` usar apenas hoje ou ontem |
| `situation` | filtrar pendentes, entregues, lidas | usar `pending,delivered,read` no MVP |
| `department_id` | filtrar departamentos específicos | opcional; útil em cargas específicas |
| `config` | trazer ID da entrega, tipo, obrigação/tarefa, departamento e responsáveis | obrigatório para a Central e conciliação |
| `attachments` | links de anexos | não usar por padrão no MVP; links expiram e podem expor documentos sensíveis |
| `attachmentsId` | IDs de anexos | usar somente em fluxo específico e auditado |
| `Pagina` | paginação | obrigatório no loop |

### 4.2 Campos operacionais importantes

Campos úteis do payload de `deliveries`:

```txt
Identificador
Razao
Fantasia
Entregas[].Nome
Entregas[].EntCompetencia
Entregas[].EntDtPrazo
Entregas[].EntDtAtraso
Entregas[].EntDtEntrega
Entregas[].EntMulta
Entregas[].Status
Entregas[].EntGuiaLida
Entregas[].EntLastDH
Entregas[].Config.EntID
Entregas[].Config.Tipo
Entregas[].Config.ID
Entregas[].Config.DptoID
Entregas[].Config.DptoNome
Entregas[].Config.RespPrazo
Entregas[].Config.RespEntrega
```

Uso no Portal Sama:

- `EntDtPrazo` alimenta vencimento;
- `EntCompetencia` alimenta competência;
- `EntDtEntrega` alimenta baixa/conclusão;
- `Status` alimenta status bruto e status normalizado;
- `EntLastDH` alimenta controle incremental e auditoria;
- `Config.EntID` deve compor o identificador externo estável;
- `Config.DptoNome` alimenta filtro por departamento;
- `Config.RespPrazo` e `Config.RespEntrega` alimentam conciliação de responsável.

---

## 5. Fluxos oficiais de sincronização

### 5.1 Carga cadastral completa — empresas, departamentos, responsáveis e catálogo de obrigações

Objetivo: manter no banco local a estrutura cadastral vinda do Acessórias.

```txt
Para Pagina = 1 até retorno vazio:
  GET /companies/ListAll?obligations&departments&registrationData&stateRegistrations&Pagina=Pagina

  Para cada empresa:
    upsert Client
    upsert snapshot cadastral em Client.metadata.acessorias
    upsert catálogo local de obrigações da empresa
    upsert/conciliar departamentos e responsáveis
    criar/atualizar alias pendente quando responsável não tiver usuário local seguro
    registrar métricas da página no sync run
```

Periodicidade recomendada:

```txt
Full load cadastral: madrugada, 1 vez ao dia
Delta operacional: durante o dia, via deliveries/ListAll
Reprocessamento manual: DEV/ADMIN, com auditoria
```

### 5.2 Carga operacional incremental — entregas recentes

Objetivo: capturar alterações recentes sem varrer todo o histórico.

```txt
GET /deliveries/ListAll
  ?DtInitial=<início da janela operacional>
  &DtFinal=<fim da janela operacional>
  &DtLastDH=<hoje ou ontem>
  &situation=pending,delivered,read
  &config
  &Pagina=N
```

Regras:

- usar apenas quando `DtLastDH` estiver em hoje ou ontem;
- paginar até lista vazia;
- fazer upsert em `acessorias_deliveries`;
- normalizar status;
- resolver responsável;
- gerar eventos de notificação somente após persistir a entrega;
- registrar totais de criados, atualizados, ignorados e erros.

### 5.3 Backfill operacional por empresa

Objetivo: primeira carga, reconstrução de base, correção de inconsistências ou histórico ampliado.

```txt
1. Buscar empresas:
   GET /companies/ListAll?departments&registrationData&Pagina=N

2. Para cada empresa:
   GET /deliveries/{IdentificadorEmpresa}
     ?DtInitial=<início>
     &DtFinal=<fim>
     &situation=pending,delivered,read
     &config
     &Pagina=N
```

Regras:

- rodar somente com permissão DEV/ADMIN;
- registrar auditoria de início, fim, usuário e parâmetros;
- usar fila serial ou concorrência muito baixa;
- permitir retomada por página/empresa;
- nunca bloquear a operação local se o Acessórias estiver fora do ar.

---

## 6. Persistência local obrigatória

A Central de Vencimentos, painéis, filtros, notificações e relatórios devem consultar o banco local do Portal Sama. A API externa deve ser chamada apenas por jobs de sincronização, previews técnicos e ações autorizadas.

### 6.1 Modelos existentes que devem continuar sendo usados

| Modelo atual | Uso |
|---|---|
| `Client` | empresa local, CNPJ/CPF, razão social, fantasia, regime, grupo, status e metadata Acessórias |
| `ClientDepartmentAssignment` | vínculo local empresa/departamento/responsável quando houver match seguro |
| `AcessoriasDelivery` | entregas/vencimentos vindos de `deliveries` |
| `AcessoriasResponsibleAlias` | responsáveis externos pendentes, confirmados ou ignorados |
| `AcessoriasDeliverySyncRun` | auditoria e métricas de sincronização operacional |
| `AcessoriasDeliveryColumnMapping` | mapeamento de obrigações Acessórias para colunas/visões internas |

### 6.2 Modelos/tabelas recomendados para evoluir a persistência cadastral

Adicionar uma tabela explícita para o catálogo de obrigações por empresa. Ela não substitui `AcessoriasDelivery`; ela guarda o snapshot vindo de `companies[].Obrigacoes`.

```prisma
model AcessoriasCompanyObligationCatalog {
  id                 String   @id @default(uuid())
  clientId           String   @map("client_id")
  clientDocument     String   @map("client_document") @db.VarChar(20)
  obligationName     String   @map("obligation_name") @db.VarChar(255)
  normalizedName     String   @map("normalized_name") @db.VarChar(255)
  rawStatus          String?  @map("raw_status") @db.VarChar(120)
  isActive           Boolean  @default(false) @map("is_active")
  deliveredCount     Int      @default(0) @map("delivered_count")
  overdueCount       Int      @default(0) @map("overdue_count")
  next30DaysCount    Int      @default(0) @map("next_30_days_count")
  future30PlusCount  Int      @default(0) @map("future_30_plus_count")
  rawPayload         Json?    @map("raw_payload")
  snapshotAt         DateTime @default(now()) @map("snapshot_at")
  createdAt          DateTime @default(now()) @map("created_at")
  updatedAt          DateTime @updatedAt @map("updated_at")

  @@unique([clientId, normalizedName])
  @@index([clientDocument])
  @@index([normalizedName])
  @@index([isActive])
  @@map("acessorias_company_obligation_catalog")
}
```

Recomendado também registrar snapshots cadastrais por execução, pelo menos em `Client.metadata.acessorias`. Se o volume crescer ou for necessário histórico/auditoria avançada, criar tabela própria:

```txt
acessorias_company_sync_runs
acessorias_company_snapshots
acessorias_company_department_responsibility_snapshots
```

### 6.3 Mapeamento `companies` -> banco local

| Campo Acessórias | Destino recomendado | Observação |
|---|---|---|
| `ID` | `Client.idEmpresa` ou `Client.metadata.acessorias.id` | ID externo do Acessórias |
| `Identificador` | `Client.cnpj` | normalizar apenas dígitos para comparação, preservar formatado em metadata se necessário |
| `Razao` | `Client.razaoSocial` | upsert por CNPJ/CPF |
| `Fantasia` | `Client.nomeFantasia` | opcional |
| `Status` | `Client.status` + metadata bruto | mapear para status interno |
| `Telefone` | `Client.telefone` | avaliar minimização |
| `UF` | `Client.estado` | normalizar UF |
| `Regime` | `Client.regimeTributario` | vindo de `registrationData`/retorno de empresa |
| `GrupoDeEmpresas` | `Client.grupoNome` e `Client.fazParteGrupo` | `fazParteGrupo=true` quando houver grupo válido |
| `InscricoesEstaduais` | `Client.metadata.acessorias.inscricoesEstaduais` | tabela própria futura se virar tela operacional |
| `Departamentos` | `ClientDepartmentAssignment` e/ou snapshot de responsabilidade | criar vínculo ativo somente com usuário confirmado |
| `Obrigacoes` | `AcessoriasCompanyObligationCatalog` | catálogo/snapshot; não é entrega operacional |
| `ContatosNaEmpresa` | metadata/tabela específica somente quando necessário | não puxar na rotina padrão se não houver caso de uso |

### 6.4 Mapeamento `deliveries` -> banco local

| Campo Acessórias | Destino recomendado | Observação |
|---|---|---|
| `Identificador` | `AcessoriasDelivery.clientDocument` + vínculo com `Client` | normalizar CNPJ/CPF |
| `Razao`/`Fantasia` | `AcessoriasDelivery.clientName` e metadata | manter cópia para histórico mesmo se cliente mudar nome |
| `Entregas[].Nome` | `AcessoriasDelivery.obligationName` | normalizar para filtros/mapeamentos |
| `Entregas[].EntCompetencia` | `AcessoriasDelivery.competence` | string `YYYY-MM` ou data normalizada conforme regra interna |
| `Entregas[].EntDtPrazo` | `AcessoriasDelivery.dueAt` | base para vencimento |
| `Entregas[].EntDtEntrega` | `AcessoriasDelivery.deliveredAt` | `0000-00-00` deve virar `null` |
| `Entregas[].Status` | `rawStatus` + `status` normalizado | manter valor bruto e status interno |
| `Entregas[].EntLastDH` | `rawPayload` e/ou metadata de sincronização | útil para delta/auditoria |
| `Config.EntID` + `Config.ID` + competência + documento | `externalId` estável | evitar duplicidade |
| `Config.DptoNome` | `department` | filtro por departamento |
| `Config.RespPrazo`/`Config.RespEntrega` | `responsibleName`, `responsibleUsername`, `responsibleUserId` | passar pelo resolver |
| payload original | `rawPayload` | sanitizar logs; manter para auditoria técnica |

---

## 7. Regra da Central de Vencimentos

A Central de Vencimentos deve ser alimentada localmente por:

```txt
AcessoriasDelivery
+ Client
+ AcessoriasCompanyObligationCatalog
+ mapeamentos internos de calendário/colunas
+ responsáveis locais/aliases
```

A tela não deve chamar diretamente `api.acessorias.com`.

Fluxo correto:

```txt
Job de sync Acessórias
        -> persiste dados no banco local
        -> normaliza status e responsáveis
        -> gera eventos internos de notificação
        -> Central consulta banco local
        -> usuário vê dados mesmo se Acessórias estiver indisponível
```

A Central deve exibir:

- empresa;
- CNPJ/CPF;
- obrigação;
- competência;
- departamento;
- responsável;
- origem;
- vencimento;
- entregue em;
- status bruto Acessórias;
- status normalizado Portal Sama;
- data da última sincronização;
- indicador de dado desatualizado quando a última sincronização passar do SLA.

---

## 8. Normalização de status

Mapeamento sugerido:

```txt
Entregue / delivered         -> DELIVERED
Atrasada / atrasado/overdue  -> OVERDUE
Pendente / pending           -> PENDING
Lida / read                  -> READ ou DUE_SOON conforme regra visual
Cancelada/dispensada         -> CANCELED
Inativa nessa empresa        -> INACTIVE_IN_COMPANY, no catálogo cadastral
Ativa                        -> ACTIVE_IN_COMPANY, no catálogo cadastral
Indefinido                   -> UNKNOWN
```

Regras:

- status de `companies[].Obrigacoes` representa situação cadastral/snapshot da obrigação na empresa;
- status de `deliveries[].Entregas` representa situação operacional de uma entrega/vencimento;
- `DELIVERED` e `CANCELED` devem aparecer no histórico da Central, mas não devem bloquear célula de vencimento aberto;
- `OVERDUE`, `PENDING`, `READ/DUE_SOON` podem gerar alerta conforme regras de notificação.

---

## 9. Responsáveis e colaboradores

Fontes possíveis:

```txt
companies/ListAll + departments:
- Departamentos[].RespNome
- Departamentos[].RespEmail

deliveries + config:
- Config.RespPrazo
- Config.RespEntrega
```

Ordem de resolução de colaborador local:

```txt
1. e-mail exato, quando existir
2. username/login exato
3. alias confirmado
4. nome normalizado + departamento, somente se houver candidato único
5. pendente de revisão quando não houver confiança suficiente
```

Regra de segurança:

```txt
Responsável externo não vira usuário ativo automaticamente.
```

Fluxo permitido:

```txt
responsável externo
  -> alias pendente
  -> revisão por DEV/ADMIN/gestor autorizado
  -> vínculo confirmado com usuário local existente
  -> atribuição operacional passa a usar responsibleUserId
```

---

## 10. Resiliência, idempotência e rate limit

A integração deve seguir estas regras:

1. **Throttling:** intervalo recomendado de 1 segundo entre requisições automáticas ou limite interno de 60 requisições/minuto.
2. **429:** respeitar `Retry-After` quando existir; se não existir, aplicar backoff mínimo de 30 segundos em jobs grandes.
3. **Retry:** retentar somente erros transitórios (`429`, `5xx`, timeout, rede), com limite de tentativas.
4. **Idempotência:** todo dado externo deve entrar por upsert com chave estável.
5. **Lock de job:** impedir duas cargas completas simultâneas com o mesmo token.
6. **Retomada:** backfill deve salvar progresso por empresa/página para continuar sem duplicar dados.
7. **Falha parcial:** erro em uma empresa não deve descartar toda a carga; registrar erro e seguir quando seguro.
8. **Dados obsoletos:** telas devem exibir última sincronização e aviso quando dados passarem do SLA definido.

Erros obrigatoriamente tratados:

```txt
401/403 -> credencial/token inválido
429 -> rate limit externo
5xx -> Acessórias instável
AbortError -> timeout
TypeError/fetch failed -> rede/DNS/conexão
JSON inválido -> resposta externa inválida
204 -> retorno válido sem conteúdo, quando aplicável
```

O frontend deve receber erro sanitizado, sem stack trace e sem token.

---

## 11. Segurança e LGPD

Regras obrigatórias:

- Token somente no backend.
- Nunca salvar token no React, localStorage, bundle frontend ou documentação.
- Rotacionar token caso apareça em `.env`, ZIP, commit, print, log ou prompt.
- Não logar payload completo com contatos, documentos, anexos ou credenciais.
- Não buscar `contacts` por padrão sem finalidade operacional clara.
- Não buscar `attachments` por padrão; anexos podem conter documentos empresariais sensíveis.
- Sanitizar payload de Web Push: título e mensagem resumidos, sem CNPJ completo, nomes sensíveis, anexos, links temporários ou detalhes documentais.
- Aplicar RBAC em preview, sync incremental, backfill, importação cadastral, revisão de aliases e divergências.
- Registrar auditoria em importação, sync, backfill, vínculo de responsável e correção manual.

---

## 12. Eventos Acessórias que devem gerar notificação

A integração Acessórias deve emitir `NotificationEvent` interno somente depois de persistir a alteração localmente.

Eventos mínimos:

```txt
ACESSORIAS_DELIVERY_COMPLETED
ACESSORIAS_DELIVERY_DUE_SOON
ACESSORIAS_DELIVERY_OVERDUE
ACESSORIAS_SYNC_FAILED
ACESSORIAS_SYNC_COMPLETED
ACESSORIAS_BACKFILL_COMPLETED
ACESSORIAS_BACKFILL_FAILED
ACESSORIAS_COMPANY_SYNC_COMPLETED
ACESSORIAS_COMPANY_SYNC_FAILED
ACESSORIAS_RESPONSIBLE_PENDING_REVIEW
ACESSORIAS_DIVERGENCE_CREATED
ACESSORIAS_DATA_STALE
```

Destinatários sugeridos:

```txt
Colaborador responsável -> obrigação próxima, vencida ou entregue
Gestor do departamento -> atrasos, divergências e pendências críticas
DEV/ADMIN/T.I. -> falha crítica de sync, token inválido, rate limit persistente, erro de backfill
```

Payload de Web Push deve ser sanitizado. Detalhes ficam dentro do Portal Sama, protegidos por autenticação, RBAC e auditoria.

---

## 13. Checklist de aceite da integração Acessórias

A integração estará adequada quando:

- `companies/ListAll` importar empresas, regime, grupo, departamentos, responsáveis e catálogo de obrigações para o banco local;
- `companies[].Obrigacoes` estiver persistido como catálogo/snapshot, separado de entregas operacionais;
- `deliveries/ListAll` estiver limitado a incremental recente com `DtLastDH` hoje/ontem;
- `deliveries/{Identificador}` estiver disponível para backfill por empresa;
- a Central de Vencimentos consultar somente o banco local;
- o Portal Sama continuar exibindo últimos dados sincronizados se o Acessórias estiver indisponível;
- todo sync registrar execução, totais, erros sanitizados e data/hora;
- responsáveis externos forem conciliados sem criação automática de usuários ativos;
- dados sensíveis não forem enviados em notificações externas;
- token não estiver presente em `.env` versionado, ZIP, logs ou frontend.

