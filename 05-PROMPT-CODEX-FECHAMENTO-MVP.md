# 05 — Prompt para Codex/IA — Fechamento do MVP Portal Sama

Use este prompt para orientar uma IA de implementação sem ampliar escopo.

---

## Prompt

Você está trabalhando no projeto Portal Sama. O projeto está em fase de fechamento de MVP e consolidação operacional.

Antes de implementar, leia apenas os documentos ativos:

```txt
docs/00-LEIA-ME-PARA-IA-MVP.md
docs/01-ESTADO-ATUAL-CODIGO-DOCUMENTACAO.md
docs/02-PLANO-FECHAMENTO-MVP.md
docs/03-CONTRATO-ACESSORIAS-OPERACIONAL.md
docs/04-DIVERGENCIAS-DOCS-CODIGO.md
docs/06-NOTIFICACOES-WEB-PUSH-MVP.md
```

Não leia `docs/_arquivo` como requisito ativo. Arquivos arquivados são histórico, não escopo atual.

---

## Objetivo técnico

Consolidar o MVP do Portal Sama para operação e gestão, com foco em:

1. corrigir build frontend;
2. estabilizar erros externos do Acessórias;
3. corrigir estratégia de sincronização Acessórias;
4. consolidar Central de Vencimentos e Obrigações;
5. vincular responsáveis do Acessórias a colaboradores locais de forma segura;
6. permitir visão por colaborador para gestão;
7. implementar notificações internas e Web Push mínimo para eventos críticos;
8. manter segurança, RBAC, CSRF e auditoria.

Implementar Web Push somente no escopo mínimo definido em `06-NOTIFICACOES-WEB-PUSH-MVP.md`. Não implementar WhatsApp, Slack, Teams, SMS, recorrência inteligente avançada, novas integrações ou redesign amplo.

---

## Status das tarefas em 2026-06-05

Este status registra o que já foi concluído no código do MVP. Ele não substitui a homologação operacional em ambiente real.

| Tarefa | Status | Observação |
| --- | --- | --- |
| Tarefa 1 — Corrigir build frontend | Concluída | `IntegraAiPage` usa narrowing seguro e o build web passa. |
| Tarefa 2 — Tratar erros externos do Acessórias | Concluída | Serviços Acessórias retornam erros controlados/sanitizados para rede, timeout, auth, rate limit, 5xx e JSON inválido. |
| Tarefa 3 — Separar incremental e backfill Acessórias | Concluída | Incremental recente e backfill por empresa foram separados; endpoint manual protegido existe. |
| Tarefa 4 — Central única de Vencimentos e Obrigações | Concluída | Backend e frontend exibem filtros, entregues, cancelados, competência, responsável, origem e status. |
| Tarefa 5 — Loading correto na Home | Concluída | KPIs usam estado de loading/skeleton, sem `...` como carregamento. |
| Tarefa 6 — Resolver responsáveis Acessórias | Concluída | Campos, tabela de aliases, resolver, endpoints, RBAC e auditoria foram implementados. |
| Tarefa 7 — Painel do colaborador | Concluída | Painel do colaborador e visão do gestor exibem obrigações e competências por responsável/colaborador. |
| Tarefa 8 — Resolver decisão MASTER | Concluída | Opção B aplicada: `DEV`, `ADMIN` e `MANAGER`; sem uso semântico de `MASTER`. |
| Tarefa 9 — Notificações internas e Web Push mínimo | Concluída no código MVP | Central, sino, endpoints mínimos, dispositivos próprios, preferências, tentativas de entrega e payload seguro implementados. |
| Tarefa 10 — Testes obrigatórios | Concluída no escopo MVP | API build/test e web build/lint/test passam; o frontend agora possui `npm test` com testes contratuais para Notificações/Web Push e proteção de VAPID private key no bundle. Homologação real ainda pendente. |

### Evidências de validação

- API: `npm.cmd run build`.
- API: `npm.cmd test -- --runInBand` — 42 suites, 251 testes.
- Web: `npm.cmd run build`.
- Web: `npm.cmd run lint`.
- Web: `npm.cmd test -- --runInBand` — 8 testes contratuais.

### Pendências para aceite final

Não há pendência técnica aberta para as tarefas 1 a 10 no escopo de código/testes do MVP. Permanecem pendências reais de ambiente e homologação:

- Aplicar migrations novas nos ambientes.
- Rodar seed/sincronização de RBAC e renovar sessões.
- Configurar VAPID real no backend.
- Homologar fluxos reais no EasyPanel: Acessórias, Central, gestor/colaborador, notificações, Web Push, contratos, documentos, certificados, onboarding, T.I e Integra-AI.
- Confirmar produtores de todos os eventos obrigatórios em cenário real.

---

## Tarefa 1 — Corrigir build frontend

Arquivo provável:

```txt
portal-sama-web/src/pages/accounting/IntegraAiPage.tsx
```

Erro:

```txt
Property 'error' does not exist on type 'IntegraAiExportResponse'
```

Corrigir narrowing do union type usando condição explícita:

```ts
visibleExportResult?.ok === false
```

ou type guard equivalente.

Executar:

```bash
npm run build
```

---

## Tarefa 2 — Tratar erros externos do Acessórias

Arquivos principais:

```txt
portal-sama-api/src/modules/integrations/acessorias/acessorias-deliveries.service.ts
portal-sama-api/src/modules/integrations/acessorias/acessorias-registrations.service.ts
portal-sama-api/src/modules/integrations/acessorias/acessorias-home.service.ts
```

Implementar captura de:

```txt
AbortError
TypeError/fetch failed
EAI_AGAIN
ECONNRESET
ECONNREFUSED
JSON inválido
429
401/403
5xx
```

Retornar exceções controladas e sanitizadas:

```txt
ServiceUnavailableException
GatewayTimeoutException, se disponível
UnauthorizedException/ForbiddenException quando token inválido
```

Nunca expor token, headers ou stack trace no frontend.

---

## Tarefa 3 — Separar incremental e backfill Acessórias

Arquivo principal:

```txt
portal-sama-api/src/modules/integrations/acessorias/acessorias-deliveries.service.ts
```

Implementar:

```ts
syncIncrementalFromListAll()
backfillDeliveriesByCompany()
```

Regra:

```txt
ListAll só pode usar DtLastDH hoje ou ontem.
Se último sync bem-sucedido for mais antigo, não usar como DtLastDH.
Executar backfill por empresa.
```

Backfill:

```txt
companies/ListAll?departments&Pagina=N
  -> deliveries/{Identificador}?DtInitial=...&DtFinal=...&config=true&Pagina=N
```

Adicionar endpoint protegido, se ainda não existir:

```txt
POST /api-v2/integrations/acessorias/deliveries/backfill
```

Permissão sugerida:

```txt
integrations.acessorias.deliveries.sync_manual
```

---

## Tarefa 4 — Central única de Vencimentos e Obrigações

Backend:

```txt
portal-sama-api/src/modules/departments/dto/list-fiscal-workspace.dto.ts
portal-sama-api/src/modules/departments/departments.service.ts
```

Adicionar no DTO:

```txt
status
source
include_completed
colaborador_id
```

Alterar query de Acessórias para permitir:

```txt
DELIVERED
CANCELED
```

quando:

```txt
status=all
status=delivered
status=canceled
include_completed=true
```

Adicionar no retorno do item:

```txt
competence
delivered_at
responsible_name
responsible_username
responsible_user_id
is_delivered
is_canceled
status_label
```

Garantir que `DELIVERED` e `CANCELED` não bloqueiem célula por vencimento.

Frontend:

```txt
portal-sama-web/src/types/departments.ts
portal-sama-web/src/services/departments.service.ts
portal-sama-web/src/pages/departments/DepartmentDueCenterPage.tsx
```

Adicionar filtros:

```txt
Todos
Pendentes
Vencidos
Entregues
Cancelados
Hoje
Próximos 7 dias
Futuros
Origem
Colaborador
Competência
```

Adicionar colunas:

```txt
Empresa
CNPJ
Obrigação
Competência
Responsável
Origem
Vencimento
Entregue em
Situação
Status Acessórias
Status Portal Sama
```

---

## Tarefa 5 — Loading correto na Home

Arquivos:

```txt
portal-sama-web/src/components/ui/KpiCard.tsx
portal-sama-web/src/pages/home/HomePage.tsx
```

Remover comportamento visual de `...` como loading.

Adicionar `loading?: boolean` ao `KpiCard`.

Exibir skeleton ou texto:

```txt
Carregando informações do Acessórias...
```

---

## Tarefa 6 — Resolver responsáveis Acessórias

Backend:

Adicionar campos em `AcessoriasDelivery`:

```txt
responsibleUserId
responsibleMatchStatus
responsibleMatchScore
responsibleMatchReason
```

Criar tabela:

```txt
AcessoriasResponsibleAlias
```

Criar serviço:

```txt
AcessoriasResponsibleResolverService
```

Resolver por:

```txt
e-mail exato
username exato
alias confirmado
nome normalizado + departamento único
```

Se não houver match seguro:

```txt
PENDING_REVIEW
AMBIGUOUS
CREATED_INACTIVE, se criar usuário inativo
```

Não criar usuário ativo automaticamente.

Endpoints:

```txt
GET /api-v2/integrations/acessorias/responsibles
GET /api-v2/integrations/acessorias/responsibles/pending
PATCH /api-v2/integrations/acessorias/responsibles/:id/link-user
PATCH /api-v2/integrations/acessorias/responsibles/:id/ignore
```

Permissões:

```txt
integrations.acessorias.responsibles.read
integrations.acessorias.responsibles.manage
```

---

## Tarefa 7 — Painel do colaborador

Arquivos prováveis:

```txt
portal-sama-web/src/pages/collaborators/CollaboratorViewPage.tsx
portal-sama-web/src/pages/manager/ManagerCollaboratorsPage.tsx
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
total
pendentes
vencidas
entregues
canceladas
empresa
obrigação
competência
vencimento
entregue em
origem
status
```

---

## Tarefa 8 — Resolver decisão MASTER

Arquivos:

```txt
portal-sama-api/src/modules/departments/departments.service.ts
portal-sama-api/src/modules/rbac/default-rbac.ts
```

Escolher uma decisão e aplicar em todo o código:

### Opção A

Formalizar `MASTER` como papel real:

```txt
PRIVILEGED_ROLES = ADMIN, DEV, MASTER
MANAGER_ROLES = ADMIN, DEV, MASTER, MANAGER
```

### Opção B

Remover uso semântico de MASTER e padronizar:

```txt
DEV = técnico total
ADMIN = administração
MANAGER = gestão
```

Não deixar os dois significados coexistirem.

Optaremos pela opção B e Removeremos o uso semântico do MASTER.
---

## Tarefa 9 — Notificações internas e Web Push mínimo

Arquivos prováveis no backend:

```txt
portal-sama-api/src/modules/notifications/notifications.module.ts
portal-sama-api/src/modules/notifications/notifications.service.ts
portal-sama-api/src/modules/notifications/notification-dispatcher.service.ts
portal-sama-api/src/modules/notifications/notification-preferences.service.ts
portal-sama-api/src/modules/notifications/web-push/web-push.module.ts
portal-sama-api/src/modules/notifications/web-push/web-push.service.ts
portal-sama-api/src/modules/notifications/web-push/web-push-subscriptions.service.ts
portal-sama-api/prisma/schema.prisma
```

Arquivos prováveis no frontend:

```txt
portal-sama-web/src/services/notifications.service.ts
portal-sama-web/src/services/web-push.service.ts
portal-sama-web/src/hooks/useWebPush.ts
portal-sama-web/src/components/notifications/NotificationBell.tsx
portal-sama-web/src/components/notifications/NotificationCenter.tsx
portal-sama-web/src/components/notifications/EnablePushNotificationsButton.tsx
portal-sama-web/public/service-worker.js
```

Implementar ou consolidar:

```txt
NotificationEvent interno persistido
NotificationDeliveryAttempt por canal
NotificationPreference básica por usuário/canal
WebPushSubscription vinculada ao usuário autenticado
NotificationDispatcherService
NotificationBell no layout
Central de Notificações
botão Ativar notificações neste dispositivo
```

Endpoints mínimos:

```txt
GET    /api-v2/notifications
PATCH  /api-v2/notifications/:id/read
PATCH  /api-v2/notifications/read-all
GET    /api-v2/notifications/preferences
PATCH  /api-v2/notifications/preferences
POST   /api-v2/notifications/web-push/subscribe
POST   /api-v2/notifications/web-push/unsubscribe
GET    /api-v2/notifications/web-push/devices
DELETE /api-v2/notifications/web-push/devices/:id
```

Eventos obrigatórios:

```txt
ACESSORIAS_DELIVERY_COMPLETED
ACESSORIAS_DELIVERY_DUE_SOON
ACESSORIAS_DELIVERY_OVERDUE
ACESSORIAS_SYNC_FAILED
ACESSORIAS_SYNC_COMPLETED
ACCESS_REQUEST_CREATED
ACCESS_REQUEST_MANAGER_REVIEW
ACCESS_REQUEST_DEV_REVIEW
ACCESS_REQUEST_APPROVED
ACCESS_REQUEST_REJECTED
CONTRACT_SENT
CONTRACT_SIGNED
CONTRACT_PENDING
CERTIFICATE_EXPIRING
CERTIFICATE_EXPIRED
DOCUMENT_APPROVED
DOCUMENT_REJECTED
SYSTEM_ACTION_SUCCESS
SYSTEM_ACTION_FAILED
```

Regras obrigatórias:

```txt
Criar notificação interna antes de tentar Web Push.
Falha de push não quebra ação principal.
Payload de Web Push deve ser curto e sanitizado.
Detalhes completos só aparecem dentro do Portal Sama autenticado.
VAPID_PRIVATE_KEY nunca vai para o frontend.
Subscription pertence ao usuário autenticado.
Usuário não lista nem revoga dispositivo de outro usuário.
Não duplicar notificação para o mesmo evento/destinatário.
```

Exemplos de mensagens seguras:

```txt
Nova solicitação de acesso
Uma solicitação aguarda validação.

Obrigação vencida
Você possui uma obrigação pendente vencida.

Falha de sincronização
Uma integração precisa de atenção técnica.
```

---

## Tarefa 10 — Testes obrigatórios

Criar/ajustar testes para:

```txt
- frontend build passa;
- erro de rede Acessórias retorna erro controlado;
- ListAll não aceita DtLastDH antigo;
- backfill por empresa chama companies/ListAll e deliveries/{Identificador};
- Central inclui DELIVERED quando include_completed=true;
- DELIVERED não bloqueia célula por vencimento;
- resolver responsável por e-mail;
- resolver responsável por username;
- nome ambíguo vira PENDING_REVIEW/AMBIGUOUS;
- usuário ativo não é criado automaticamente;
- gestor consegue filtrar por colaborador;
- notificação interna é criada para evento crítico;
- Web Push não contém dados sensíveis;
- falha no envio de Web Push não quebra a ação principal;
- usuário não consegue listar/revogar subscription de outro usuário;
- VAPID private key não aparece no bundle/frontend.
```

Executar:

```bash
# API
npm run build
npm test -- --runInBand

# Web
npm run build
npm test -- --runInBand
```

Se algum teste não puder ser executado, informar motivo técnico real.

---

## Resultado esperado

Ao final, informar:

```txt
Arquivos alterados
Comportamento corrigido
Testes executados
Build executado
Pendências restantes
Riscos de segurança restantes
```
