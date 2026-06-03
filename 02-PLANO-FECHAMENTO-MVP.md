# 02 — Plano de fechamento do MVP — Portal Sama

**Status:** fonte ativa  
**Data:** 2026-06-03  
**Objetivo:** fechar um MVP operacional, estável e seguro para centralizar o fluxo de trabalho da empresa.

---

## 1. Definição do MVP

O MVP do Portal Sama deve permitir que a empresa use a plataforma como centro de operação para:

- controlar obrigações e vencimentos;
- acompanhar entregas do Acessórias;
- consultar pendências por cliente, colaborador e competência;
- gerenciar documentos empresariais;
- controlar contratos, propostas e assinatura/aceite;
- controlar certificados digitais;
- acompanhar entrada de clientes/onboarding;
- controlar solicitações de acessos do departamento de T.I;
- centralizar extratos e Integra-AI;
- receber notificações internas e Web Push para eventos operacionais críticos;
- dar visão para gestor e operação;
- manter trilha de auditoria e segurança.

O MVP não precisa conter todos os recursos futuros. Ele precisa funcionar bem no fluxo operacional principal.

---

## 2. Princípio de fechamento

Cada área deve terminar com:

```txt
1. Tela operacional clara
2. Endpoint estável
3. Permissão/RBAC definido
4. Auditoria quando houver ação crítica
5. Loading/erro vazio bem tratado
6. Notificação quando houver evento operacional relevante
7. Teste mínimo
8. Documentação curta de uso/decisão
```

---

## 3. Fase M0 — Congelamento documental e foco da IA

### Objetivo

Evitar que a IA se confunda com excesso de documentação.

### Tarefas

- Criar pasta `docs/_arquivo/`.
- Mover análises antigas, planos futuros e propostas duplicadas para `_arquivo`.
- Manter ativos apenas os documentos do pacote de fechamento MVP.
- Adicionar aviso no topo de documentos arquivados:

```md
> Documento arquivado. Não usar como requisito ativo do MVP.
> Mantido apenas como histórico de decisão.
```

### Critério de aceite

- IA/Codex consegue iniciar tarefa lendo poucos arquivos ativos, incluindo o contrato de Notificações/Web Push quando a tarefa envolver eventos operacionais.
- Não há documento antigo competindo com o plano atual.

---

## 4. Fase M1 — Estabilização técnica imediata

### Objetivo

Garantir que o projeto compile e erros externos não virem `500` genérico.

### Tarefas prioritárias

1. Corrigir build do frontend em `IntegraAiPage.tsx`.
2. Corrigir tratamento de erro de rede/timeout nos serviços Acessórias.
3. Garantir que falha externa retorne erro controlado:

```txt
503 Acessórias indisponível
504 timeout externo
429 rate limit externo
```

4. Garantir diagnóstico sanitizado sem token ou stack trace.
5. Revisar variáveis de produção/EasyPanel.
6. Remover/rotacionar segredos encontrados em `.env` enviados em ZIP.

### Critério de aceite

- `npm run build` do frontend passa.
- Build do backend passa.
- Erro de rede do Acessórias não retorna erro interno genérico.
- Nenhum token aparece no frontend/logs.

---

## 5. Fase M2 — Aquisição correta de dados do Acessórias

### Objetivo

Separar carga completa de sincronização incremental.

### Decisão técnica

```txt
deliveries/ListAll = incremental recente
companies/ListAll + deliveries/{Identificador} = backfill completo por empresa
```

### Tarefas

1. Criar método `syncIncrementalFromListAll()`.
2. Criar método `backfillDeliveriesByCompany()`.
3. Impedir uso de `DtLastDH` antigo em `deliveries/ListAll`.
4. Quando não houver sync recente, rodar backfill por empresa ou exigir ação manual autorizada.
5. Usar `companies/ListAll` paginado até lista vazia.
6. Para cada empresa, chamar `deliveries/{CNPJ/CPF}` no período desejado.
7. Salvar/atualizar entregas por `externalId` estável.
8. Registrar sync run com totais e erro resumido.

### Critério de aceite

- Carga inicial não depende de `deliveries/ListAll` com `DtLastDH` antigo.
- Incremental usa somente hoje/ontem quando `Identificador=ListAll`.
- Backfill pode ser executado manualmente por perfil autorizado.
- API externa fora do ar não quebra operação local.

---

## 6. Fase M3 — Central única de Vencimentos e Obrigações

### Objetivo

Transformar `/departamentos/vencimentos` na página única para vencimentos internos e obrigações Acessórias.

### Backend

Atualizar `GET /api-v2/departments/workspace` para aceitar:

```txt
status=all|open|pending|overdue|delivered|canceled|today|next7|future
source=all|calendar|acessorias
include_completed=true|false
colaborador_id=<id>
month_key=YYYY-MM
dept_key=<departamento>
```

O retorno deve incluir nos itens:

```txt
competence
delivered_at
responsible_name
responsible_username
responsible_user_id
is_delivered
is_canceled
status_label
source
```

### Frontend

A tela deve exibir:

- Empresa;
- CNPJ;
- obrigação;
- competência;
- responsável;
- origem;
- vencimento;
- entregue em;
- situação;
- status Acessórias;
- status Portal Sama.

Filtros mínimos:

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
Departamento
```

### Regra crítica

Entregas `DELIVERED` e `CANCELED` devem aparecer na Central, mas **não devem bloquear célula por vencimento**.

### Critério de aceite

- A Central mostra pendentes, vencidas, entregues e canceladas.
- A Central substitui a visão operacional separada de vencimentos.
- Configuração de calendário fica como configuração, não operação.
- Gestor consegue filtrar por colaborador.

---

## 7. Fase M4 — Conciliação de responsáveis Acessórias

### Objetivo

Vincular obrigações a colaboradores reais do Portal Sama.

### Regras

Resolver colaborador local por ordem de confiança:

```txt
1. e-mail exato
2. username/login exato
3. alias confirmado
4. nome normalizado + departamento, somente se único candidato
```

Se não houver match seguro:

```txt
criar alias pendente para revisão
não criar usuário ativo automaticamente
não alterar carteira oficial do cliente
```

### Banco recomendado

Adicionar em `AcessoriasDelivery`:

```txt
responsibleUserId
responsibleMatchStatus
responsibleMatchScore
responsibleMatchReason
```

Criar tabela:

```txt
acessorias_responsible_aliases
```

### Endpoints sugeridos

```txt
GET /api-v2/integrations/acessorias/responsibles
GET /api-v2/integrations/acessorias/responsibles/pending
PATCH /api-v2/integrations/acessorias/responsibles/:id/link-user
PATCH /api-v2/integrations/acessorias/responsibles/:id/ignore
```

### Critério de aceite

- Obrigações aparecem no painel do colaborador por `responsibleUserId`.
- Aliases ambíguos ficam pendentes.
- Nenhum usuário ativo é criado automaticamente.
- Vínculo confirmado gera auditoria.

---

## 8. Fase M5 — Painéis de operação e gestão

### Objetivo

Garantir que operação e gestão tenham visão clara.

### Painel do colaborador

Adicionar seção:

```txt
Obrigações e Competências
```

Com:

- total;
- pendentes;
- vencidas;
- entregues;
- canceladas;
- empresa;
- obrigação;
- competência;
- vencimento;
- status;
- origem.

### Painel do gestor

Deve permitir:

- ver carteira por colaborador;
- ver obrigações por colaborador;
- ver atrasos por departamento;
- ver entregas concluídas;
- revisar responsáveis pendentes;
- acessar divergências Acessórias.

### Papel MASTER

Definir uma decisão única:

- opção A: `MASTER` vira papel real de gestão/técnico e entra no RBAC;
- opção B: usar `DEV`/`ADMIN` como papel técnico total e não usar `MASTER` no código.

Não manter o estado atual ambíguo.

 Optaremos por utilizar a Opção B.

---

## 9. Fase M6 — Notificações internas e Web Push mínimo

### Objetivo

Consolidar uma camada transversal de notificações para que o Portal Sama avise colaboradores, gestores, DEV/ADMIN e operação sobre eventos críticos, inclusive quando o usuário não estiver com a tela principal aberta.

### Decisão técnica

```txt
Notificação interna = fonte oficial e persistida
Web Push = canal complementar em tempo real do MVP
```

Toda notificação Web Push deve nascer de um evento interno persistido. Se o push falhar, a notificação interna permanece salva e o fluxo principal não deve quebrar.

### Escopo permitido no MVP

- criar/listar/marcar notificações internas como lidas;
- exibir sino/contador de notificações no layout;
- criar Central de Notificações;
- registrar navegador/dispositivo para Web Push;
- permitir ativar/desativar Web Push por dispositivo;
- enviar Web Push para eventos operacionais críticos;
- registrar tentativa de entrega por canal;
- manter preferências básicas por usuário/canal;
- sanitizar payload externo para não expor dados sensíveis.

### Fora do escopo desta fase

- WhatsApp, Slack, Teams, SMS ou e-mail avançado;
- resumo diário inteligente;
- agrupamento complexo;
- regras avançadas por horário;
- IA decidindo prioridade sem regra explícita;
- dados sensíveis no payload do push.

### Eventos obrigatórios

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

### Endpoints mínimos

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

### Banco mínimo

```txt
notification_events
notification_delivery_attempts
notification_preferences
web_push_subscriptions
```

Se o projeto já possuir `Notification` e `BrowserPushSubscription`, reaproveitar ou migrar com cuidado, evitando criar estruturas duplicadas sem necessidade.

### Segurança do Web Push

- `VAPID_PUBLIC_KEY` pode ir ao frontend;
- `VAPID_PRIVATE_KEY` deve ficar somente no backend;
- subscription deve pertencer ao usuário autenticado;
- usuário não pode listar/revogar dispositivo de outro usuário;
- payload do push deve ser curto e sem dados sensíveis;
- detalhes completos aparecem somente dentro do Portal Sama após autenticação e RBAC;
- falha de push não deve expor stack trace, token ou chave.

### Critério de aceite

- usuário consegue ativar Web Push em um navegador suportado;
- subscription é salva vinculada ao usuário autenticado;
- notificação interna é criada antes de qualquer push;
- eventos mínimos geram notificação para os destinatários corretos;
- push não contém CNPJ/CPF completo, credenciais, documentos, certificados, contratos ou stack trace;
- falha de envio gera tentativa registrada, mas não quebra a ação principal;
- usuário consegue marcar notificações como lidas;
- DEV/ADMIN recebe falhas críticas do Acessórias;
- gestor recebe solicitações que exigem validação;
- colaborador recebe aviso de obrigação próxima, vencida ou entregue.

---

## 10. Fase M7 — Segurança, auditoria e homologação

### Objetivo

Fechar o MVP com segurança operacional.

### Checklist

- RBAC em endpoints críticos;
- CSRF em ações mutáveis;
- auditoria em sincronizações, importações, mapeamentos e vínculos;
- auditoria em criação, leitura crítica, envio e falha de notificações;
- token Acessórias somente no backend;
- VAPID private key somente no backend;
- segredos fora de ZIP/commit;
- upload/documentos com controle de acesso;
- certificados digitais com criptografia e controle rígido;
- contratos/propostas com trilha de aceite;
- solicitações de acesso de T.I com responsável e histórico;
- tratamento global de erro sem stack trace em produção;
- logs sem dados sensíveis.

### Homologação

Executar no EasyPanel com dados reais:

1. preview Acessórias;
2. backfill por empresa;
3. incremental;
4. Central de Vencimentos;
5. painel colaborador;
6. mapeamento/divergências;
7. notificações internas;
8. ativação de Web Push;
9. envio de Web Push para evento de teste;
10. contratos/propostas;
11. certificados;
12. documentos;
13. onboarding;
14. solicitações T.I;
15. Integra-AI.

---

## 10. Entregáveis finais do MVP

O MVP só deve ser considerado fechado quando:

- backend builda;
- frontend builda;
- testes críticos passam;
- Acessórias não gera erro interno genérico;
- Central mostra todas as obrigações relevantes;
- gestor vê operação por colaborador;
- colaborador vê obrigações próprias;
- documentos/contratos/certificados/onboarding/acessos estão navegáveis;
- notificações internas e Web Push mínimo funcionam para eventos críticos;
- permissões e auditoria estão aplicadas;
- segredos estão protegidos;
- documentação ativa está reduzida e objetiva.
