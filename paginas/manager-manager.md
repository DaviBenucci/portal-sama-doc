# [PARCIAL] Vencimentos do Gestor

> Atualizacao 2026-05-20 11:20: primeira fatia de historico operacional criada em `/manager/historico`, consumindo `GET /api-v2/managers/history` e `GET /api-v2/managers/history/:companyId/timeline` do `ManagersModule`. A tela e somente leitura nesta etapa; escrita/edicao, validacao com MySQL real, usuarios reais por departamento e Playwright seguem pendentes.
> Atualizacao 2026-05-20 11:45: `/manager/historico` passou a criar/editar historico por `POST/PATCH /api-v2/managers/history` e salvar vida da empresa por `POST /api-v2/managers/company-life`, com `manager_history.write`, CSRF, escopo por departamento e auditoria. Validacao MySQL/usuarios reais/Playwright segue pendente.
> Atualizacao 2026-05-20 12:30: `/manager` passou a consumir `GET /api-v2/managers/overview` para presenca da equipe e KPIs online/offline; `sama_user_presence` foi modelada como `UserPresence`. Validacao MySQL/usuarios reais/Playwright segue pendente antes de desligar o `overview` legado.
> Atualizacao 2026-05-20 16:02: `/manager` passou a remover vencimentos por `DELETE /api-v2/calendar/entries/:id` quando a sessao possui `calendar.manage`; a API v2 exige CSRF, escopo por departamento e auditoria `calendar.entry.delete`. Validacao MySQL/usuarios reais/Playwright segue pendente antes de desligar `calendar_delete_entry`.
> Atualizacao 2026-06-01: o workspace departamental passou a incluir vencimentos oficiais de entregas Acessorias sincronizadas.
> Atualizacao 2026-06-01 Central: criada rota `/departamentos/vencimentos` para consolidar vencimentos de calendario e Acessorias; ainda falta validar no EasyPanel com dados reais antes de considerar homologado.

## 1. Identificação da página

- **Arquivo HTML:** `Manager/manager.html`
- **Módulo:** Gestor
- **Arquivos CSS relacionados:** `global.css`, `Manager/manager.css`, `https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css`
- **Arquivos JavaScript relacionados:** `auth.js`, `global.js`, `Manager/manager.js`
- **APIs/endpoints relacionados:** `/api/manager_workspace.php actions: calendar_delete_entry, calendar_view_day, calendar_view_month, history_list, history_timeline, history_append, history_update, company_life_save, overview`, `/api-v2/calendar/month`, `/api-v2/calendar/entries/:id`, `/api-v2/managers/overview`, `/api-v2/managers/history`, `/api-v2/managers/company-life`, `/api/notifications.php actions: ack, alert_ack, clear_unread, list`, `/api/storage.php actions: audit_push, auth_logout, auth_session_status, get, set, user_presence_ping`
- **Perfil de usuário provável:** Gestor ou master
- **Nível de criticidade:** Alto
- **Prioridade de refatoração:** Alta

---

## 2. Objetivo da página

Acompanhar vencimentos do departamento, status de colaboradores, histórico de empresas e eventos de calendário.

- **Problema resolvido:** centraliza o fluxo de gestor para reduzir consulta manual, duplicação operacional e perda de rastreabilidade.
- **Quem utiliza:** Gestor ou master.
- **Etapa do fluxo:** Gestor dentro do Portal Sama.
- **Dados/documentos manipulados:** vencimentos, responsáveis, histórico de empresas, presença e status dos colaboradores.
- **Observação:** ponto a validar no código/backend para regras não explícitas no HTML.

---

## 3. Funcionamento detalhado da interface

Cabeçalho identificado com área de usuário, departamento e ação de logout ou retorno, carregado em conjunto com `global.js`/`auth.js` quando presentes.

Navegação lateral/superior com atalhos para Início, Departamento, Clientes e Área Dev, reaproveitando a identidade visual global do portal.

Conteúdo principal identificado: Vencimento Painel de vencimentos por departamento Usuário ▾ Departamento Logout Departamento Mês de referência Status dos colaboradores Ver histórico das empres; Departamento Mês de referência Status dos colaboradores Ver histórico das empresas Transferências Configurar vencimento; Colaboradores no departamento 0 Online 0 Offline 0; Dia -- Sem dados; Online.

Títulos/seções visíveis: `Vencimento`, `Dia --`, `Status dos colaboradores`, `Histórico das empresas`, `Excluir evento`.

Campos relevantes: `managerDeptSelect` (select), `managerMonth` (month), `historyDeptSelect` (select), `historySearch` (search).

Ações/botões identificados: `Início`, `Departamento ▾`, `Modelo padrão Departamento`, `Clientes`, `Área Dev`, `Usuário ▾`, `Departamento`, `Logout`, `Status dos colaboradores`, `Ver histórico das empresas`, `Transferências`, `Configurar vencimento`, `Fechar`, `Cancelar edição`, `... mais 4 item(ns)`.

Foram identificados 15 elementos com comportamento de modal/dialog; recomenda-se centralizar em componente reutilizável com foco acessível e fechamento por teclado.

Fluxo esperado do usuário:

1. Acessar a página pela navegação interna ou link público/legado, conforme o caso.
2. O JavaScript relacionado carrega sessão, dados iniciais, filtros e tabelas.
3. O usuário preenche campos, seleciona filtros ou aciona botões de criação/edição/envio/download.
4. As APIs PHP recebem a requisição, aplicam validações e retornam JSON ou arquivo.
5. A interface atualiza status, tabelas, mensagens de sucesso/erro ou redireciona conforme o resultado.

---

## 4. Fluxo de dados

- Ao abrir a página, `auth.js` consulta `/api/storage.php?action=auth_session_status`; se a sessão não estiver autenticada, redireciona para `/index.html`. O usuário público fica em `sessionStorage` por meio de shim compatível com a chave legada `usuarioLogado`.

- Endpoints envolvidos: `/api/manager_workspace.php`, `/api-v2/calendar/month`, `/api-v2/calendar/entries/:id`, `/api-v2/managers/overview`, `/api-v2/managers/history`, `/api-v2/managers/company-life`, `/api/notifications.php`, `/api/storage.php`. As actions observadas são: `ack`, `alert_ack`, `clear_unread`, `list`, `audit_push`, `auth_logout`, `auth_session_status`, `get`, `set`, `user_presence_ping`, `calendar_delete_entry`, `calendar_view_day`, `calendar_view_month`, `history_list`, `history_timeline`, `history_append`, `history_update`, `company_life_save`, `overview`.

- Uso de armazenamento no navegador identificado nos scripts relacionados: `localStorage` em 24 ocorrência(s) e `sessionStorage` em 8 ocorrência(s). Tokens/segredos não devem ser armazenados nesses mecanismos; manter apenas estado não sensível.

- Cookies de sessão são tratados pelo backend em `api/auth.php` com `HttpOnly`, `SameSite=Lax` e `Secure` condicional a HTTPS/proxy. Ponto a validar em produção: HTTPS obrigatório, HSTS efetivo e domínio/path restritivos.

Campos/dados de entrada identificados no HTML: `managerDeptSelect` (select), `managerMonth` (month), `historyDeptSelect` (select), `historySearch` (search)

Quando alguma action for indireta ou montada dinamicamente no JavaScript, considerar: **ponto a validar no código/backend**.

---

## 5. APIs e integrações relacionadas

### Endpoint: `/api/manager_workspace.php`

- **Action observada:** `calendar_delete_entry`
  - **Método provável:** POST/DELETE
  - **Finalidade:** Excluir evento de vencimento.
  - **Status API v2:** Substituto inicial em `DELETE /api-v2/calendar/entries/:id`, conectado ao dashboard React `/manager`.
  - **Dados enviados:** ID/data.
  - **Dados retornados:** Confirmação.
  - **Validações esperadas:** JWT, `calendar.manage`, papel ADMIN/DEV/MANAGER, CSRF, regra/empresa/departamento permitido e auditoria.
  - **Riscos de segurança:** Perda de vencimento.
  - **Melhorias recomendadas:** Validar em MySQL/homologacao e avaliar soft delete/motivo antes do desligamento total do legado.
- **Action observada:** `calendar_view_day`
  - **Método provável:** GET
  - **Finalidade:** Consultar eventos do dia.
  - **Dados enviados:** Data/departamento.
  - **Dados retornados:** Eventos.
  - **Validações esperadas:** Acesso ao departamento.
  - **Riscos de segurança:** Exposição de tarefas.
  - **Melhorias recomendadas:** Policy por departamento.
- **Action observada:** `calendar_view_month`
  - **Método provável:** GET
  - **Finalidade:** Consultar calendário mensal.
  - **Dados enviados:** Departamento/mês.
  - **Dados retornados:** Eventos por dia.
  - **Validações esperadas:** Acesso ao departamento.
  - **Riscos de segurança:** Exposição de vencimentos.
  - **Melhorias recomendadas:** Policy por departamento.
- **Action observada:** `history_list`
  - **Método provável:** GET
  - **Finalidade:** Listar histórico de empresas.
  - **Dados enviados:** Filtros por departamento/empresa.
  - **Dados retornados:** Histórico.
  - **Validações esperadas:** Gestor/master.
  - **Riscos de segurança:** Exposição de histórico operacional.
  - **Melhorias recomendadas:** Paginação e filtros server-side.
- **Action observada:** `history_timeline`
  - **Método provável:** GET
  - **Finalidade:** Carregar linha do tempo da empresa.
  - **Dados enviados:** Empresa/cliente.
  - **Dados retornados:** Eventos de histórico.
  - **Validações esperadas:** Acesso ao cliente/departamento.
  - **Riscos de segurança:** IDOR por cliente.
  - **Melhorias recomendadas:** Policy por cliente.
- **Action observada:** `history_append`
  - **Método provável:** POST
  - **Finalidade:** Registrar novo historico operacional da empresa.
  - **Status API v2:** Substituto inicial em `POST /api-v2/managers/history`.
  - **Validações esperadas:** JWT, `manager_history.write`, CSRF, empresa/departamento permitido e auditoria.
- **Action observada:** `history_update`
  - **Método provável:** POST/PATCH
  - **Finalidade:** Editar topicos de um registro de historico.
  - **Status API v2:** Substituto inicial em `PATCH /api-v2/managers/history/:id`.
  - **Validações esperadas:** JWT, `manager_history.write`, CSRF, escopo do registro e auditoria.
- **Action observada:** `company_life_save`
  - **Método provável:** POST
  - **Finalidade:** Salvar snapshot de vida da empresa.
  - **Status API v2:** Substituto inicial em `POST /api-v2/managers/company-life`.
  - **Validações esperadas:** JWT, `manager_history.write`, CSRF, empresa/departamento permitido e auditoria.
- **Action observada:** `overview`
  - **Método provável:** GET
  - **Finalidade:** Carregar visão de vencimentos/gestor.
  - **Status API v2:** Substituto inicial em `GET /api-v2/managers/overview`, conectado ao painel React `/manager`.
  - **Dados enviados:** Departamento e mês.
  - **Dados retornados:** KPIs, colaboradores, departamentos permitidos e presenca online/offline.
  - **Validações esperadas:** JWT, `manager_history.read`, papel ADMIN/DEV/MANAGER e departamento permitido.
  - **Riscos de segurança:** Exposição de presenca/departamento indevido.
  - **Melhorias recomendadas:** Policy por departamento.

### Endpoint: `/api/notifications.php`

- **Action observada:** `ack`
  - **Método provável:** POST
  - **Finalidade:** Marcar notificação como lida.
  - **Dados enviados:** ID da notificação.
  - **Dados retornados:** Confirmação.
  - **Validações esperadas:** Usuário dono ou autorizado.
  - **Riscos de segurança:** Alterar notificação de outro usuário.
  - **Melhorias recomendadas:** Validar propriedade no backend.
- **Action observada:** `alert_ack`
  - **Método provável:** POST
  - **Finalidade:** Marcar alerta como visualizado.
  - **Dados enviados:** ID da notificação.
  - **Dados retornados:** Confirmação.
  - **Validações esperadas:** Usuário dono ou autorizado.
  - **Riscos de segurança:** Perda de alerta crítico.
  - **Melhorias recomendadas:** Auditar alertas críticos.
- **Action observada:** `clear_unread`
  - **Método provável:** POST
  - **Finalidade:** Limpar não lidas.
  - **Dados enviados:** Escopo do usuário.
  - **Dados retornados:** Confirmação.
  - **Validações esperadas:** Usuário autenticado.
  - **Riscos de segurança:** Ocultar notificações importantes sem registro.
  - **Melhorias recomendadas:** Manter histórico e confirmação para alertas críticos.
- **Action observada:** `list`
  - **Método provável:** GET
  - **Finalidade:** Listar notificações do usuário/departamento.
  - **Dados enviados:** Filtros por usuário, departamento ou período.
  - **Dados retornados:** Lista de notificações.
  - **Validações esperadas:** Usuário autenticado e escopo por audiência.
  - **Riscos de segurança:** Exposição de notificações de outro usuário/departamento.
  - **Melhorias recomendadas:** Implementar paginação, escopo por servidor e retenção.

### Endpoint: `/api/storage.php`

- **Action observada:** `audit_push`
  - **Método provável:** POST
  - **Finalidade:** Registrar evento de auditoria vindo do frontend/camada de serviços.
  - **Dados enviados:** JSON com usuário, mensagem, ação, entidade e metadados.
  - **Dados retornados:** Confirmação do registro.
  - **Validações esperadas:** Usuário autenticado, CSRF e normalização de metadados.
  - **Riscos de segurança:** Auditoria manipulável se aceitar usuário informado pelo cliente.
  - **Melhorias recomendadas:** Derivar usuário do servidor e tornar trilha append-only.
- **Action observada:** `auth_logout`
  - **Método provável:** POST
  - **Finalidade:** Encerrar sessão autenticada.
  - **Dados enviados:** Cookie de sessão e token CSRF quando houver usuário autenticado.
  - **Dados retornados:** Confirmação de logout.
  - **Validações esperadas:** CSRF para sessão autenticada.
  - **Riscos de segurança:** Logout CSRF ou sessão persistente indevida.
  - **Melhorias recomendadas:** Invalidar sessão no servidor e limpar cookies com atributos seguros.
- **Action observada:** `auth_session_status`
  - **Método provável:** GET
  - **Finalidade:** Validar sessão atual e retornar usuário público e token CSRF.
  - **Dados enviados:** Cookie de sessão `SAMASESSID`.
  - **Dados retornados:** JSON com `authenticated`, `user` e `csrf_token`.
  - **Validações esperadas:** Sessão válida, TTL absoluto/ocioso e usuário ativo.
  - **Riscos de segurança:** Se usado apenas no frontend, HTML ainda pode ser carregado sem sessão; endpoints devem bloquear dados.
  - **Melhorias recomendadas:** Manter sessão server-side, CSRF obrigatório para mutações e registrar login/logout em auditoria.
- **Action observada:** `get`
  - **Método provável:** GET
  - **Finalidade:** Carregar dados persistidos no KV/MySQL por chave.
  - **Dados enviados:** Query `key`.
  - **Dados retornados:** Valor persistido, metadados de atualização e backend.
  - **Validações esperadas:** Usuário autenticado e, para `sama_usuarios_login_v1`, papel master.
  - **Riscos de segurança:** Exposição horizontal de dados se a chave não for protegida por escopo.
  - **Melhorias recomendadas:** Substituir KV genérico por controllers por domínio e autorização por recurso.
- **Action observada:** `set`
  - **Método provável:** POST/PUT/PATCH
  - **Finalidade:** Salvar dados persistidos no KV/MySQL por chave.
  - **Dados enviados:** JSON com `value` e opcional `audit`.
  - **Dados retornados:** Confirmação, backend e auditoria opcional.
  - **Validações esperadas:** Usuário autenticado, CSRF e restrições por chave.
  - **Riscos de segurança:** Mass assignment, sobrescrita indevida e perda de histórico.
  - **Melhorias recomendadas:** Usar DTO/Form Requests, transações, validação por esquema e versionamento.
- **Action observada:** `user_presence_ping`
  - **Método provável:** GET/POST
  - **Finalidade:** Atualizar presença online do usuário.
  - **Dados enviados:** Sessão atual e dados de navegação.
  - **Dados retornados:** Status de presença.
  - **Validações esperadas:** Usuário autenticado.
  - **Riscos de segurança:** Rastreamento excessivo ou exposição de presença.
  - **Melhorias recomendadas:** Minimizar dados e documentar finalidade.

---

## 6. Regras de negócio identificadas

### Regras identificadas no frontend

- Departamento e mês controlam visão de vencimentos.
- Configurações de calendário definem recorrência, dia, coluna vinculada e empresas.
- Transferências exigem origem, destino/período e justificativa quando aplicável.

### Regras que devem existir obrigatoriamente no backend

- Validar gestor do departamento.
- Máquina de estados para transferências.
- Auditar inclusão/exclusão de vencimentos e transferência de empresas.
- Impedir alteração global por usuário sem perfil master.

---

## 7. Análise de segurança

### 7.1 Autenticação

`auth.js` está relacionado à página. A validação inicial ocorre no frontend chamando `/api/storage.php?action=auth_session_status`; endpoints PHP precisam manter a validação real de sessão.

### 7.2 Autorização

A autorização deve considerar perfil, departamento, propriedade do recurso e vínculo com cliente/processo. O frontend pode esconder botões, mas a decisão precisa estar no PHP.

### 7.3 Proteção contra XSS

Foram identificadas 47 ocorrência(s) de `innerHTML` nos scripts vinculados. Evitar inserir dados de API com `innerHTML`; preferir `textContent` ou sanitização contextual.

### 7.4 Proteção contra CSRF

Para actions de mutação, o padrão do projeto usa `X-CSRF-Token` via `SamaAuth.buildHeaders` e `sama_require_csrf()` em APIs críticas. Validar cobertura em todas as actions desta página.

### 7.5 Proteção contra SQL Injection

Os endpoints PHP analisados usam `PDO`/`prepare` em diversas consultas. Ainda assim, qualquer consulta dinâmica, filtro, `ORDER BY`, nome de tabela/coluna ou action genérica deve usar allowlist e prepared statements. Ponto a validar em cada endpoint relacionado.

### 7.6 Segurança em uploads

Não há upload direto identificado nesta página.

### 7.7 Dados sensíveis

Dados sensíveis envolvidos: vencimentos, responsáveis, histórico de empresas, presença e status dos colaboradores. Não expor esses dados no HTML inicial, em logs do navegador, em mensagens de erro, em query strings permanentes ou em `localStorage`.

### 7.8 Logs e auditoria

Ações críticas desta página devem gerar trilha de auditoria com usuário derivado da sessão, entidade, antes/depois quando aplicável, IP, user-agent e timestamp. Evitar registrar valores sensíveis em claro.

---

## 8. Checklist de segurança da página

- [x] Página protegida contra acesso não autenticado.
- [ ] Permissões validadas no backend.
- [ ] Inputs validados no frontend.
- [ ] Inputs validados no backend.
- [ ] Dados sensíveis não expostos no HTML.
- [ ] Dados sensíveis não expostos no JavaScript.
- [ ] Tokens não armazenados de forma insegura.
- [x] Cookies configurados com `HttpOnly`, `Secure` e `SameSite` no backend de autenticação; validar HTTPS/proxy em produção.
- [ ] Requisições críticas protegidas contra CSRF.
- [ ] Dados renderizados no DOM protegidos contra XSS.
- [ ] Endpoints protegidos contra SQL Injection.
- [x] Uploads validados por extensão, MIME type e conteúdo.
- [x] Arquivos armazenados fora da pasta pública.
- [x] Downloads protegidos por autenticação e autorização.
- [ ] Ações críticas registradas em auditoria.
- [ ] Erros tratados sem expor detalhes internos.
- [ ] Logs não expõem dados sensíveis.

---

## 9. Problemas técnicos encontrados

- Arquivos JavaScript extensos/acoplados à tela: `global.js` (2433 linhas), `Manager/manager.js` (1324 linhas).
- Uso de `innerHTML` em scripts relacionados (47 ocorrência(s)), exigindo revisão de XSS.
- Uso de `localStorage` em scripts relacionados (24 ocorrência(s)); não armazenar tokens ou dados sensíveis.
- Persistência genérica por `/api/storage.php?action=get|set` e chaves de domínio dificulta contratos claros de API, testes e autorização por recurso.

---

## 10. Melhorias recomendadas

### 10.1 Melhorias rápidas

- Centralizar chamadas `fetch` em um client com tratamento de erro, CSRF e timeout.
- Padronizar mensagens de erro sem expor detalhes internos.
- Garantir que botões sensíveis sejam ocultados e bloqueados por permissão no backend.

### 10.2 Melhorias de médio prazo

- Separar componentes de layout, tabelas, filtros, formulários e modais.
- Criar serviços de API por domínio em vez de usar actions genéricas espalhadas.
- Implementar RBAC por perfil/departamento com testes automatizados.
- Criar logs de auditoria estruturados para todas as ações críticas.

### 10.3 Melhorias estruturais

- Migrar gradualmente para React + TypeScript + Vite.
- Reorganizar backend em NestJS com Modules, Controllers, DTOs, Services, Guards, Pipes, Prisma Migrations e Permissions/Policies.
- Documentar contratos com OpenAPI/Swagger.
- Adicionar CI/CD com testes, lint e análise estática.
- Containerizar ambiente com Docker Compose e separar dev/homolog/prod.

---

## 11. Frameworks e tecnologias recomendadas para esta página

### Frontend

- React Hook Form + Zod para formulários tipados, validação consistente e mensagens de erro padronizadas.

### Backend

- Services transacionais para calendário, histórico e transferência, com validação de estado no backend.

### Segurança

- Sessão server-side com cookies `HttpOnly`, `Secure` e `SameSite`, evitando armazenar tokens sensíveis em `localStorage`.
- RBAC por perfil e departamento, validado no backend em todas as actions.

### Infraestrutura

- Docker Compose com Apache/Nginx, PHP 8+, MySQL e volumes privados para `data/private`, `logs` e backups.
- GitHub Actions com ESLint/Prettier, TypeScript strict, Jest/Supertest e Playwright em páginas críticas.

---

## 12. Sugestão de refatoração da página

Componentes sugeridos:
- `PageLayout`
- `Sidebar`
- `Header`
- `ActionButton`
- `FormSection`
- `ValidatedInput`
- `ConfirmModal`

Serviços sugeridos:
- `authService`
- `auditService`
- `notificationService`
- `managerWorkspaceService`
- `calendarService`
- `transferService`

Estratégia de refatoração:
- Separar layout, manipulação de DOM, regra de negócio e consumo de API.
- Criar camada de API com autenticação, CSRF, timeout, tratamento de erro e logs padronizados.
- Migrar primeiro os fluxos críticos para componentes tipados e cobertos por testes.
- Remover regras de negócio do frontend que possam ser burladas e revalidá-las no backend.
- Criar testes de regressão para permissões, XSS, CSRF e fluxos principais.

---

## 13. Testes recomendados

### Testes unitários

- Validar funções de formatação/normalização usadas pela página.
- Validar renderização de status e mensagens de erro sem HTML inseguro.

### Testes de integração

- Endpoint deve rejeitar requisição sem autenticação quando a página for interna.
- Endpoint deve rejeitar usuário sem perfil/departamento permitido.
- Erro da API deve ser tratado na interface sem expor stack trace.

### Testes E2E

- Usuário autorizado acessa a página e executa o fluxo principal.
- Usuário sem permissão não visualiza nem executa ações críticas.

### Testes de segurança

- Tentar XSS em campos livres e dados retornados pela API.
- Tentar IDOR alterando IDs, client_id, doc_id ou token.
- Verificar CSRF em actions POST/PUT/PATCH/DELETE.

---

## 14. Conclusão da página

A página `Manager/manager.html` atua no módulo **Gestor** e manipula **vencimentos, responsáveis, histórico de empresas, presença e status dos colaboradores**. Os principais riscos são: autorização por perfil/departamento e exposição de dados. A principal recomendação é reforçar validações server-side, autorização por recurso, auditoria e componentização gradual.

- **Nível de criticidade:** Alto
- **Prioridade de refatoração:** Alta
- **Observações finais:** manter o HTML como camada de apresentação; regras de permissão, validação, persistência e auditoria devem residir no backend. Quando houver incerteza, tratar como **ponto a validar no código/backend**.


---

## 15. Atualização de stack alvo — TypeScript/NestJS

Esta seção complementa a análise original da página com a decisão técnica posterior de migrar o backend PHP para **Node.js + TypeScript + NestJS** e evoluir o frontend para **React + TypeScript + Vite**.

### 15.1 Mapeamento da tela para a arquitetura alvo

- **Arquivo HTML atual:** `Manager/manager.html`
- **Rota React sugerida:** `/manager` e `/manager/historico`
- **Componente React sugerido:** `ManagerDashboardPage.tsx` e `ManagerHistoryPage.tsx`
- **Backend alvo:** `/api-v2`, implementado em NestJS
- **Banco principal:** MySQL 8 com Prisma ORM

### 15.2 Módulos NestJS relacionados

- `ManagersModule`
- `ClientsModule`
- `AuditModule`

### 15.3 Arquivos atuais que devem ser usados como referência de migração

- `Manager/manager.html`
- `api/manager_workspace.php`

### 15.4 Diretriz de migração para esta tela

1. Mapear todas as chamadas atuais a `/api` ou arquivos PHP relacionados.
2. Criar contratos equivalentes em `/api-v2` com DTOs tipados.
3. Validar autenticação, permissão e escopo no backend NestJS.
4. Substituir gradualmente `fetch`/JavaScript vanilla por services TypeScript no frontend.
5. Migrar a interface para React apenas depois de o endpoint crítico estar seguro.
6. Registrar auditoria para ações críticas desta página, principalmente criação, alteração, upload, download, aprovação, assinatura ou acesso a dados sensíveis.

### 15.5 Referências complementares

- [`../GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`](../GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md)
- [`../ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md`](../ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md)
- [`../BANCO_DADOS_MYSQL_PRISMA.md`](../BANCO_DADOS_MYSQL_PRISMA.md)
- [`../SEGURANCA.md`](../SEGURANCA.md)
- [`../MAPEAMENTO_MIGRACAO_APIS.md`](../MAPEAMENTO_MIGRACAO_APIS.md)
