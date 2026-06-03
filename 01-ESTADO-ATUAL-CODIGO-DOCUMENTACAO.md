# 01 — Estado atual do código e da documentação — Portal Sama

**Status:** fonte ativa  
**Data:** 2026-06-03  
**Base analisada:** `portal-sama-docs.zip`, `portal-sama-api.zip`, `portal-sama-web.zip`

---

## 1. Visão executiva

O Portal Sama já possui uma base ampla e funcional para centralizar a operação da empresa. O projeto não está mais em fase inicial. Há módulos relevantes de autenticação, clientes, colaboradores, documentos, contratos, propostas, certificados digitais, onboarding, solicitações de acesso, Integra-AI, departamentos, vencimentos, notificações, auditoria e integração com Acessórias.

O ponto crítico atual é que a documentação cresceu mais rápido do que o fechamento técnico. Existem muitos documentos com status semelhante, alguns planejando futuro, outros descrevendo implementações parciais e outros registrando decisões já superadas. Isso aumenta a chance de uma IA ou desenvolvedor tratar backlog futuro como requisito imediato.

A recomendação é congelar o escopo do MVP e consolidar o que já existe. A partir da decisão de produto de 2026-06-03, **Notificações internas e Web Push mínimo entram no MVP**, não como expansão futura ampla, mas como infraestrutura operacional essencial para evitar perda de avisos críticos.

---

## 2. Módulos existentes no backend

Foram identificados módulos NestJS para:

```txt
access-requests
audit
auth
calendar
certificates
client-assignments
clients
collaborators
contracts
departments
documents
health
integrations
legalization
managers
notifications
onboarding
permissions
proposals
rbac
roles
transfers
users
accounting / Integra-AI
```

Isso confirma que o Portal Sama já cobre os principais eixos operacionais:

- controle de clientes;
- carteira por departamento/colaborador;
- obrigações e vencimentos;
- documentos empresariais;
- certificados digitais;
- contratos e propostas;
- assinatura/fluxo contratual;
- entrada de clientes/onboarding;
- solicitações de acesso ao T.I;
- extratos/Integra-AI;
- auditoria;
- notificações internas e Web Push mínimo.

---

## 3. Banco de dados e entidades relevantes

O Prisma já possui modelos para áreas centrais, incluindo:

```txt
User
Department
Role
Permission
UserRole
RolePermission
RefreshToken
Client
ClientDepartmentAssignment
AcessoriasDelivery
AcessoriasDeliverySyncRun
AcessoriasDeliveryColumnMapping
AcessoriasFiscalApplyRun
AcessoriasFiscalDivergence
OnboardingProcess
LegalizationProcess
Notification
BrowserPushSubscription
PublicToken
Document
DigitalCertificate
Proposal
Contract
AccessRequest
CalendarRule
CalendarRuleCompany
IntegraAiJob
IntegraAiProfile
AuditLog
```

A estrutura demonstra que o produto já ultrapassou a fase de protótipo. O trabalho agora deve ser de estabilização, não de expansão indiscriminada.

---

## 4. Estado atual da integração Acessórias

### 4.1 O que já existe

O código já possui:

- cliente/serviço backend para consultar Acessórias;
- Home com resumo do Acessórias;
- preview/importação de empresas e responsáveis;
- sincronização de entregas;
- persistência local em `acessorias_deliveries`;
- registro de sync runs em `acessorias_delivery_sync_runs`;
- mapeamento de entregas para colunas operacionais;
- aplicação conservadora de baixas em workspace;
- divergências para casos inseguros;
- scheduler opcional por variável de ambiente;
- tentativas de rate limit e paginação;
- rota React `/departamentos/vencimentos`.

### 4.2 O que ainda está incompleto

A integração ainda não está fechada para operação plena porque:

- `deliveries/ListAll` está sendo usado como se pudesse servir para carga completa em alguns fluxos;
- não existe backfill robusto por empresa usando `companies/ListAll` + `deliveries/{Identificador}`;
- erros de rede/timeout em alguns serviços podem subir como erro inesperado;
- a Central de Vencimentos exclui `DELIVERED` e `CANCELED` em parte do backend;
- a Central ainda não exibe de forma completa competência, entregue em, responsável e status externo/interno;
- responsáveis do Acessórias ainda não são vinculados de forma segura a usuários locais por `responsibleUserId`;
- não há tabela de aliases/responsáveis pendentes para revisão;
- o papel `MASTER` está semanticamente inconsistente entre conversa, documentação e código;
- a Home exibe `...` enquanto carrega, gerando percepção de erro.

---

## 5. Estado atual da Central de Vencimentos

Existe a rota:

```txt
/departamentos/vencimentos
```

Ela já consolida parte dos vencimentos internos e itens do Acessórias, mas o backend filtra entregas externas apenas com status:

```txt
PENDING
DUE_SOON
OVERDUE
UNKNOWN
```

Isso exclui:

```txt
DELIVERED
CANCELED
```

Na prática, a tela vira uma lista de pendências/vencimentos, não uma central completa de obrigações por competência.

Para o MVP, a Central precisa listar também obrigações entregues/concluídas e canceladas/dispensadas, com filtros.

---

## 6. Estado atual dos responsáveis Acessórias

O código já extrai nomes de responsáveis das entregas, especialmente de campos como:

```txt
Config.RespEntrega
RespEntrega
Config.RespPrazo
RespPrazo
```

Também há extração de responsáveis das empresas/departamentos em importação quando `ACESSORIAS_COLLABORATORS_PATH` está vazio.

Porém, o dado ainda fica principalmente como texto:

```txt
responsibleName
responsibleUsername
```

Ainda falta o vínculo operacional seguro:

```txt
responsibleName/responsibleUsername
        -> resolver colaborador local
        -> salvar responsibleUserId
        -> permitir filtro por colaborador
        -> exibir painel do colaborador com obrigações entregues, pendentes e vencidas
```

---

## 7. Estado atual do frontend

O frontend possui páginas/rotas para:

- Home;
- clientes;
- documentos;
- departamentos/clientes;
- departamentos/modelo;
- departamentos/vencimentos;
- Integra-AI;
- certificados;
- solicitação de acesso;
- T.I/acessos;
- legalização;
- contratos;
- propostas;
- onboarding;
- auditoria;
- painel gestor;
- colaboradores;
- transferências;
- calendário/configuração;
- notificações;
- configurações.

O produto já tem a base necessária para ser o centro operacional da empresa.

### Problema de build identificado

O build do frontend falha em:

```txt
src/pages/accounting/IntegraAiPage.tsx(1277,77)
Property 'error' does not exist on type 'IntegraAiExportResponse'.
```

A causa provável é falta de estreitamento correto do union type. A correção recomendada é usar condição explícita:

```ts
visibleExportResult?.ok === false
```

ou criar um type guard para o caso de erro.

---

## 8. Estado atual das notificações e Web Push

O schema Prisma já indica base parcial para notificações, com modelos como:

```txt
Notification
BrowserPushSubscription
```

Isso mostra que a plataforma já caminhou para uma fundação de notificações. Porém, para o MVP operacional, ainda é necessário consolidar semanticamente a camada de notificação como fluxo transversal da aplicação.

### O que precisa existir no MVP

- Central de Notificações acessível no frontend;
- sino/indicador de notificações no layout;
- criação de `NotificationEvent` interno antes de qualquer Web Push;
- inscrição de navegador/dispositivo vinculada ao usuário autenticado;
- envio de Web Push com payload seguro e sem dados sensíveis;
- registro de tentativa de entrega por canal;
- preferência básica por usuário/canal;
- marcação de notificação como lida;
- tratamento de falha de push sem quebrar o fluxo principal.

### Eventos mínimos que precisam notificar

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

### Lacuna atual

A existência de modelos relacionados a notificações não garante, por si só, que todo evento operacional crítico esteja emitindo notificação interna e Web Push. O MVP precisa padronizar isso via `NotificationDispatcherService` ou serviço equivalente.

---

## 9. Estado dos testes/build local

### Backend

Build NestJS executado com sucesso em base equivalente analisada.

Testes direcionados executados com sucesso:

```txt
5 test suites passed
27 tests passed
```

Testes direcionados cobriram módulos de Acessórias e departamentos.

### Frontend

Build falhou pelo erro de TypeScript em `IntegraAiPage.tsx`.

### Prisma

A validação Prisma não pôde ser concluída no ambiente de análise porque o engine tentou ser baixado de `binaries.prisma.sh` e o ambiente não tinha acesso externo. Isso não prova erro no schema; apenas impede validação naquele ambiente.

---

## 10. Estado da documentação

A documentação é extensa e contém valor, mas está fragmentada.

Problemas encontrados:

- muitos documentos ativos com status parcial;
- documentos antigos ainda parecendo mandatórios;
- escopo futuro misturado com escopo atual;
- documentos antigos tratavam Web Push como futuro; a decisão atual coloca Web Push mínimo no MVP, mantendo recorrência avançada como futuro;
- integração Acessórias documentada como “não implementar agora”, mas já implementada parcialmente;
- documentos de análise acumulados sem virarem plano de execução fechado.

Recomendação:

```txt
Mover documentos antigos para docs/_arquivo/
Manter apenas o pacote de fechamento MVP como fonte ativa
Criar ADRs curtos para decisões técnicas estáveis
```

---

## 11. Conclusão do estado atual

O Portal Sama não precisa de mais expansão conceitual para o MVP. Precisa de:

1. estabilização técnica;
2. consolidação das telas existentes;
3. fechamento da integração Acessórias;
4. correção do build frontend;
5. tratamento robusto de erros externos;
6. central única de vencimentos/obrigações;
7. vínculo seguro de responsáveis a colaboradores;
8. notificações internas e Web Push mínimo para eventos críticos;
9. documentação ativa menor e objetiva.
