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
7. manter segurança, RBAC, CSRF e auditoria.

Não implementar Web Push, WhatsApp, recorrência inteligente avançada, novas integrações ou redesign amplo.

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

## Tarefa 9 — Testes obrigatórios

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
- gestor consegue filtrar por colaborador.
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
