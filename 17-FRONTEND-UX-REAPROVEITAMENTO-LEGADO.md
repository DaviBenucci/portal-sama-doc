# Frontend — UX, navegação e reaproveitamento semântico do legado

Atualizado em: 2026-06-19

## 1. Objetivo

Este documento orienta a refatoração do `portal-sama-web` para preservar a base técnica moderna em React/TypeScript e, ao mesmo tempo, recuperar decisões de UX do legado `portal-sama.zip` que estavam mais claras para o usuário.

A nova arquitetura já possui uma boa fundação: React, Vite, React Router, TanStack Query, Axios com credenciais, Zustand, Zod, Tailwind, lazy routes e rotas separadas por módulo. O problema identificado não é falta de tecnologia; é a fragmentação da experiência. Em alguns pontos o frontend novo distribuiu informações que, no legado, ficavam agrupadas no contexto correto.

A decisão principal é: **o painel do cliente deve voltar a ser a tela central do cliente**, organizada por abas, com dados operacionais, documentos, certificados, acessos e vida da empresa no mesmo contexto.

## 2. O que deve ser reaproveitado do legado

O arquivo legado `Client/painel.html` deve ser tratado como referência funcional e semântica. O objetivo não é copiar HTML/CSS antigo, e sim reaproveitar a lógica de agrupamento e a jornada do usuário.

### 2.1. Painel do cliente por abas

No legado, o painel do cliente tinha as abas:

| Aba legado | Decisão para a nova arquitetura | Motivo |
|---|---|---|
| Geral | manter como `ClientGeneralTab` | Visão consolidada da empresa, status, rank, regime, contato e endereço. |
| Responsáveis | manter como `ClientAssignmentsTab` | A responsabilidade por departamento é crítica para operação. |
| Acessos | recriar como `ClientAccessesTab` | O usuário precisa enxergar credenciais e acessos relacionados ao cliente em uma tela. |
| Documentos | manter/evoluir como `ClientDocumentsTab` | Documentos pendentes e enviados precisam estar no contexto do cliente. |
| Certificados | recriar como `ClientCertificatesTab` | Certificados digitais são dados sensíveis, mas precisam de visibilidade operacional controlada. |
| Vida da empresa | recriar como `ClientCompanyLifeTab` | Histórico por departamento evita perda de contexto entre colaboradores. |

### 2.2. Cabeçalho do cliente

O cabeçalho do legado trazia:

- botão de voltar;
- nome do cliente;
- razão social;
- CNPJ;
- rank;
- status editável conforme permissão.

Na nova arquitetura, isso deve virar um componente estável:

```txt
src/pages/clients/components/ClientDashboardHeader.tsx
```

Responsabilidades:

- exibir identidade do cliente;
- exibir badge de status;
- exibir rank;
- exibir ações primárias contextualizadas;
- bloquear alteração de status quando o usuário não possuir permissão;
- manter retorno para `/clientes` preservando filtros quando possível.

### 2.3. Navegação lateral mais clara

O legado tinha menos caminhos e agrupava ações por perfil. A navegação nova é permission-based, o que é correto, mas alguns nomes e agrupamentos ficaram técnicos demais.

A navegação deve ser reorganizada com os seguintes grupos semânticos:

| Grupo | Itens | Observação |
|---|---|---|
| Operação | Início, Clientes, Colaboradores, Vencimentos | Rotas mais usadas no dia a dia. |
| Cliente | Documentos, Certificados, Acessos | Entradas que normalmente nascem do contexto do cliente. |
| Legalização | Processos, Propostas, Contratos | Fluxo comercial/legal completo. |
| Onboarding | Entrada de cliente, Processos | Separar entrada de empresa da operação recorrente. |
| T.I. | Solicitações, Acessos T.I. | Acesso com permissões próprias. |
| Gestão | Painel gestor, Transferências, Calendário, Histórico | Visível para gestores/admins. |
| Administração | Usuários, RBAC, Auditoria, Configurações | Visível apenas para papéis administrativos. |
| DEV | Ferramentas DEV | Somente desenvolvimento ou role explicitamente autorizada. |

Regra: a navegação deve continuar usando permissões do backend. O frontend não deve confiar apenas em role ou label textual.

### 2.4. Integra-AI com escolha de fluxo

A entrada do Integra-AI deve deixar de assumir que todo usuário quer processar extrato bancário. Ao abrir `/contabil/integra-ai`, a primeira decisão do usuário deve ser clara e curta:

- `Extrato`: fluxo atual do Integra-AI para extrato bancário, regras contábeis e TXT Domínio.
- `Faturamento`: novo fluxo para PGDAS/Livro Caixa/CSV Domínio, usando o backend Python existente em `C:\Users\Sama Contabilidade\Downloads\Faturamento` por meio de API/adaptador seguro.

O seletor deve ser uma tela operacional, não uma landing page. Usar dois botões grandes o suficiente para toque, com ícone e descrição curta, mantendo permissão `accounting.integra_ai.read` ou permissão específica equivalente definida pelo backend.

Para `Faturamento`, o frontend deve oferecer:

- empresa/código usado no Acessórias;
- ano;
- controle de modo entre `Um mês` e `Todos os meses`;
- seletor de mês obrigatório apenas em `Um mês`;
- campo opcional `codigo_dominio` para quando o código do Domínio for diferente do código do Acessórias;
- estado de processamento, avisos por mês e downloads seguros dos artefatos liberados pela API.

O frontend não deve executar Python no navegador, nem receber `API_TOKEN` do Acessórias. Toda execução deve passar pelo backend do Portal ou por adaptador autenticado, com auditoria e sem expor paths locais, `.env`, PDF bruto ou logs sensíveis.

## 3. Diagnóstico do frontend novo

Arquivos relevantes encontrados na nova arquitetura:

```txt
src/app/router.tsx
src/components/layout/AppLayout.tsx
src/components/layout/Sidebar.tsx
src/components/layout/navigation.tsx
src/pages/clients/ClientDashboardPage.tsx
src/pages/clients/ClientDocumentsPanel.tsx
src/services/api.ts
src/services/clients.service.ts
src/services/certificates.service.ts
src/services/documents.service.ts
src/services/contracts.service.ts
src/stores/auth.store.ts
```

Pontos positivos:

- rotas lazy-loaded;
- React Router bem separado;
- TanStack Query para cache e invalidação;
- Axios com `withCredentials`, CSRF e refresh automático;
- Zustand para sessão;
- tipos por domínio;
- componentes de layout reutilizáveis;
- Playwright disponível para E2E.

Pontos que precisam ajuste:

- painel do cliente novo está mais próximo de um resumo do que de uma central operacional;
- documentos aparecem no cliente, mas certificados, acessos e vida da empresa não estão no mesmo contexto;
- a navegação está correta tecnicamente, porém menos intuitiva que o legado;
- telas de legalização e assinatura precisam suportar ZapSign de forma clara;
- algumas páginas parecem separadas por módulo técnico, e não por jornada do usuário;
- falta uma estratégia visual padronizada para abas, timeline, painéis sensíveis e estados vazios.

## 4. Arquitetura frontend alvo

### 4.1. Estrutura recomendada

Criar uma organização por domínio dentro de `src/pages/clients` ou migrar gradualmente para `src/features`.

Opção recomendada para baixo risco:

```txt
src/pages/clients/
  ClientDashboardPage.tsx
  ClientDocumentsPanel.tsx
  components/
    ClientDashboardHeader.tsx
    ClientDashboardTabs.tsx
    ClientGeneralTab.tsx
    ClientAssignmentsTab.tsx
    ClientAccessesTab.tsx
    ClientDocumentsTab.tsx
    ClientCertificatesTab.tsx
    ClientCompanyLifeTab.tsx
    ClientSensitiveValue.tsx
    ClientEmptyState.tsx
    ClientActivityTimeline.tsx
```

Quando o módulo estiver estável, pode ser promovido para:

```txt
src/features/clients/dashboard/
```

### 4.2. Rota alvo do painel do cliente

Rota principal:

```txt
/clientes/:clientId/painel
```

A aba ativa deve ser controlada por query string:

```txt
/clientes/:clientId/painel?tab=geral
/clientes/:clientId/painel?tab=responsaveis
/clientes/:clientId/painel?tab=acessos
/clientes/:clientId/painel?tab=documentos
/clientes/:clientId/painel?tab=certificados
/clientes/:clientId/painel?tab=vida
```

Motivos:

- permite compartilhar link direto para uma aba;
- mantém histórico do navegador;
- facilita testes E2E;
- evita estado oculto apenas em memória;
- preserva o padrão do legado de centralizar informações sem criar várias rotas soltas.

### 4.3. Carregamento de dados

O painel deve carregar primeiro o resumo principal:

```txt
GET /api-v2/clients/:id/dashboard
```

Depois, cada aba deve carregar seus dados sob demanda com TanStack Query:

| Aba | Query key | Endpoint esperado |
|---|---|---|
| Geral | `['client-dashboard', clientId]` | `GET /clients/:id/dashboard` |
| Responsáveis | `['client-assignments', clientId]` | `GET /clients/:id/assignments` |
| Acessos | `['client-accesses', clientId]` | `GET /clients/:id/accesses` |
| Documentos | `['client-documents', clientId]` | `GET /documents?clientId=...` ou endpoint dedicado |
| Certificados | `['client-certificates', clientId]` | `GET /certificates?clientId=...` |
| Vida da empresa | `['client-company-life', clientId]` | `GET /clients/:id/company-life` |

Regra: a aba pode exibir skeleton/loading próprio, mas o cabeçalho do cliente deve permanecer estável.

## 5. Especificação do painel do cliente

### 5.1. Layout principal

Estrutura visual:

```txt
[Voltar]  Cliente / Razão / CNPJ / Rank / Status
[Ações rápidas: editar, novo documento, novo certificado, novo contrato]

[Abas]
Geral | Responsáveis | Acessos | Documentos | Certificados | Vida da empresa

[Conteúdo da aba]
```

O conteúdo não deve empurrar o usuário para várias telas sem necessidade. Quando uma ação exigir modal, confirmar se há risco de segurança. Para dados sensíveis, preferir fluxo explícito com confirmação.

### 5.2. Aba Geral

Componentes:

```txt
ClientGeneralTab
ClientInfoCard
ClientContactCard
ClientAddressCard
ClientTaxProfileCard
ClientGroupCard
ClientQuickEditForm
```

Dados mínimos:

- nome fantasia;
- razão social;
- CNPJ;
- ID da empresa;
- regime tributário;
- atividade;
- telefone;
- e-mail;
- endereço completo;
- grupo de empresas;
- status;
- rank;
- data de cadastro;
- data de atualização.

Regras:

- edição rápida somente com permissão `clients.update`;
- CNPJ deve ser exibido formatado, mas enviado como dígitos quando aplicável;
- campos vazios devem exibir `-` ou estado vazio claro;
- alteração de status deve gerar auditoria backend.

### 5.3. Aba Responsáveis

Componentes:

```txt
ClientAssignmentsTab
AssignmentTable
AssignmentCreateForm
AssignmentTransferForm
AssignmentEndForm
AssignmentHistoryDrawer
```

Dados mínimos:

- departamento;
- responsável operacional;
- gestor;
- tipo de responsabilidade;
- status;
- início;
- fim;
- ações.

Regras:

- usar endpoints existentes de `client-assignments`;
- filtrar colaboradores ativos no frontend, mas a regra real deve estar no backend;
- transferência deve exigir motivo;
- encerramento deve exigir motivo;
- alteração deve invalidar `['client-assignments', clientId]` e `['client-dashboard', clientId]`;
- ações devem depender de permissões específicas: create, update, transfer, end.

### 5.4. Aba Acessos

Esta aba deve recuperar uma funcionalidade visível no legado. Ela não deve expor senhas em texto puro nem perpetuar práticas inseguras.

Componentes:

```txt
ClientAccessesTab
ClientAccessRequestCard
ClientAccessList
SensitiveAccessRevealDialog
AccessAuditTrail
```

Dados mínimos:

- tipo de acesso;
- sistema;
- usuário/login;
- status;
- responsável;
- criado em;
- atualizado em;
- último uso se existir;
- botão para solicitar alteração/novo acesso.

Regras de segurança:

- senha nunca deve ser armazenada em texto puro;
- se houver segredo herdado, migrar para storage criptografado com chave de aplicação/KMS;
- exibição de segredo deve exigir permissão própria, confirmação e auditoria;
- segredo revelado deve ser mascarado por padrão;
- copiar para área de transferência deve ser ação auditada;
- frontend nunca deve registrar segredo em console, erro, analytics ou localStorage.

Permissões sugeridas:

```txt
client_accesses.read
client_accesses.request
client_accesses.manage
client_accesses.reveal
```

### 5.5. Aba Documentos

O novo frontend já possui `ClientDocumentsPanel`. Ele deve ser refinado para operar como aba.

Funcionalidades:

- checklist de documentos obrigatórios;
- pendências com badge na aba;
- upload seguro;
- status por documento;
- histórico de status;
- download protegido;
- filtros por departamento/tipo/status;
- estado vazio claro.

Regras:

- upload sempre via backend;
- validação de extensão/MIME no frontend é apenas UX; a segurança real é backend;
- após upload, invalidar queries de documentos e dashboard;
- arquivos devem abrir/download via rota protegida; nunca por URL pública de storage.

### 5.6. Aba Certificados

O legado deixava certificados visíveis no painel do cliente. A nova arquitetura possui módulo de certificados e criptografia de senha, mas precisa aproximar isso do contexto do cliente.

Componentes:

```txt
ClientCertificatesTab
CertificateList
CertificateStatusBadge
CertificateUploadDialog
CertificatePasswordRevealDialog
CertificateExpirationTimeline
```

Dados mínimos:

- titular;
- tipo;
- validade;
- status;
- dias para expirar;
- responsável;
- arquivo disponível;
- data de upload;
- ações permitidas.

Regras de segurança:

- senha do certificado deve permanecer criptografada no backend;
- revelação de senha deve exigir permissão e registrar auditoria;
- download do certificado deve ser protegido e auditado;
- alerta visual para certificados próximos do vencimento;
- ação de substituir certificado deve preservar histórico.

Permissões sugeridas:

```txt
certificates.read
certificates.upload
certificates.download
certificates.password.reveal
certificates.update
certificates.archive
```

### 5.7. Aba Vida da empresa

Essa é uma das principais perdas da nova UX em relação ao legado.

Objetivo: registrar contexto histórico por cliente/departamento, evitando que informações importantes fiquem em conversas, memória individual ou documentos soltos.

Componentes:

```txt
ClientCompanyLifeTab
CompanyLifeDepartmentFilter
CompanyLifeTopicEditor
CompanyLifeTimeline
CompanyLifeHistoryDrawer
```

Dados mínimos:

- departamento;
- tópicos;
- descrição;
- autor;
- data;
- origem;
- última atualização.

Regras:

- criar entrada por departamento;
- permitir histórico/timeline;
- não sobrescrever registros antigos sem auditoria;
- permitir busca textual;
- permitir anexar referência a documento ou processo;
- separar anotação operacional de dados sensíveis.

Endpoint sugerido:

```txt
GET    /api-v2/clients/:id/company-life
POST   /api-v2/clients/:id/company-life
PATCH  /api-v2/clients/:id/company-life/:entryId
GET    /api-v2/clients/:id/company-life/history
```

## 6. Legalização e ZapSign no frontend

A interface de contratos deve suportar dois modos:

| Modo | Comportamento |
|---|---|
| Assinatura interna | usar rota pública `/assinatura/:token` e capturar assinatura no Portal. |
| ZapSign | gerar/enviar documento pelo backend e exibir link externo da ZapSign. |

Tela de contrato deve exibir:

- provedor de assinatura;
- ambiente: sandbox ou produção controlada;
- status da assinatura;
- link seguro de assinatura, quando permitido;
- botão de sincronizar status;
- aviso de que assinatura ZapSign não deve ser feita no formulário interno;
- histórico de eventos.

A página pública `/assinatura/:token` deve receber do backend uma resposta que indique o provider. Se o provider for ZapSign, ela deve exibir instrução e botão para abrir `sign_url`, ou redirecionar de forma controlada se esse comportamento for definido pelo produto.

Regra: a URL de assinatura da ZapSign não deve ser inventada pelo frontend. Ela vem do backend.

## 7. API client e contrato de erros

O `src/services/api.ts` já faz refresh automático, CSRF e retry em 401. Manter esta fundação.

Ajustes recomendados:

- padronizar `getErrorMessage` para exibir `message`, `error`, `code` e fallback seguro;
- não exibir stack trace em produção;
- tratar 403 como falta de permissão e não como erro genérico;
- tratar 409 como conflito de regra de negócio;
- tratar 422/400 como validação;
- tratar 429 como limite temporário.

Componentes de erro:

```txt
PermissionDeniedState
ValidationErrorBanner
RateLimitWarning
RetryableErrorState
EmptyState
```

## 8. Segurança frontend obrigatória

| Risco | Regra |
|---|---|
| XSS | não usar `dangerouslySetInnerHTML` sem sanitização e fonte confiável. |
| Token leakage | access token somente em memória/Zustand; nunca em localStorage. |
| CSRF | mutações sempre enviam header CSRF exigido pelo backend. |
| Segredos | não renderizar senha/certificado sem ação explícita e auditoria backend. |
| Logs | não imprimir payload sensível em console. |
| Upload | validar tamanho/extensão antes do envio, mas confiar na validação backend. |
| Rotas públicas | não expor dados além do necessário para assinatura/proposta/documentos públicos. |
| Permissões | esconder ação no frontend e validar no backend. |

## 9. Padrões visuais recomendados

### 9.1. Abas

- usar botão com `aria-selected`;
- suportar teclado;
- refletir aba na URL;
- badge em documentos pendentes, certificados vencendo e acessos pendentes;
- abas não autorizadas não devem aparecer ou devem aparecer bloqueadas com explicação, conforme decisão de produto.

### 9.2. Dados sensíveis

- usar ícone de cadeado;
- mascarar por padrão;
- exigir confirmação;
- expirar visualização após alguns segundos quando aplicável;
- registrar auditoria no backend antes de retornar segredo.

### 9.3. Estados vazios

Estados vazios devem orientar ação:

```txt
Nenhum certificado cadastrado para este cliente.
[Adicionar certificado]
```

Evitar telas vazias sem próxima ação.

## 10. Testes frontend obrigatórios

### 10.1. Testes de contrato

Rodar:

```bash
npm run test
```

Cobrir:

- parse dos schemas Zod;
- contrato de `ClientDashboard`;
- contrato de `ClientAssignments`;
- contrato de documentos/certificados;
- contrato de ZapSign no contrato.

### 10.2. E2E Playwright

Rodar:

```bash
npm run test:e2e
```

Cenários mínimos:

1. Login.
2. Abrir `/clientes`.
3. Abrir painel do cliente.
4. Alternar abas por URL e clique.
5. Visualizar dados gerais.
6. Criar/editar responsabilidade quando autorizado.
7. Upload de documento válido.
8. Upload de documento inválido deve falhar.
9. Certificado vencendo aparece com alerta.
10. Acesso sensível exige permissão.
11. Usuário sem permissão não vê ação sensível.
12. Contrato ZapSign exibe provider e link retornado pelo backend.
13. Página pública de assinatura interna funciona.
14. Página pública de assinatura ZapSign não permite assinatura interna.

## 11. Checklist de aceite

O frontend estará pronto quando:

- painel do cliente tiver as seis abas do legado;
- cada aba consumir endpoints reais da nova API;
- abas preservarem estado via query string;
- navegação estiver agrupada por jornada, não apenas por módulo técnico;
- ZapSign estiver visível no fluxo de contratos;
- ações sensíveis dependerem de permissão;
- não houver segredo em console/localStorage;
- `npm run lint`, `npm run build`, `npm run test` e `npm run test:e2e` passarem;
- testes cobrirem navegação e permissões principais;
- usuários conseguirem resolver a rotina principal do cliente sem abrir várias páginas desconexas.
