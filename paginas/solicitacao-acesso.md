# Solicitação de Acesso ao Sistema

## 1. Identificação da página

- **Arquivo HTML:** `SolicitacaoAcesso/solicitacao-acesso.html`
- **Módulo:** Solicitação de Acesso
- **Arquivos CSS relacionados:** `https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css`, `global.css`, `SolicitacaoAcesso/Styles.css`, `SolicitacaoAcesso/PageStyles.css`, `https://fonts.googleapis.com/css2?family=Sora:wght@400;600;700;800&family=Source+Sans+3:wght@400;500;600;700&display=swap`, `https://cdn.jsdelivr.net/npm/flatpickr/dist/flatpickr.min.css`
- **Arquivos JavaScript relacionados:** `auth.js`, `DEV/dev-data.js`, `global.js`, `https://cdn.jsdelivr.net/npm/flatpickr`, `https://cdn.jsdelivr.net/npm/flatpickr/dist/l10n/pt.js`, `SolicitacaoAcesso/acesso.js`
- **APIs/endpoints relacionados:** `/api/access_requests.php actions: colaborador_status, decision, gestor_approvals, list`, `/api/notifications.php actions: ack, alert_ack, clear_unread, list`, `/api/storage.php actions: audit_push, auth_logout, auth_session_status, get, set, user_presence_ping`, `SolicitacaoAcesso/enviar_acesso.php actions: submit`
- **Perfil de usuário provável:** Colaborador, gestor e master com fluxos distintos
- **Nível de criticidade:** Alto
- **Prioridade de refatoração:** Alta

---

## 2. Objetivo da página

Registrar solicitações de acesso temporário, permitir aprovação por gestor e consulta por master.

- **Problema resolvido:** centraliza o fluxo de solicitação de acesso para reduzir consulta manual, duplicação operacional e perda de rastreabilidade.
- **Quem utiliza:** Colaborador, gestor e master com fluxos distintos.
- **Etapa do fluxo:** Solicitação de Acesso dentro do Portal Sama.
- **Dados/documentos manipulados:** datas, horários, justificativa, perfil, colaborador, gestor, aprovação e histórico de solicitações.
- **Observação:** ponto a validar no código/backend para regras não explícitas no HTML.

---

## 3. Funcionamento detalhado da interface

Cabeçalho identificado com área de usuário, departamento e ação de logout ou retorno, carregado em conjunto com `global.js`/`auth.js` quando presentes.

Navegação lateral/superior com atalhos para Início, Departamento, Clientes e Área Dev, reaproveitando a identidade visual global do portal.

Conteúdo principal identificado: Solicitação de Acesso ao Sistema Fluxo automático por perfil do usuário logado. Usuário ▾ Departamento do usuário Logout Solicitação de acesso ao sistema Acesso; Solicitação de acesso ao sistema Acesso indisponível para este perfil. Histórico de solicitações Visualize as solicitações enviadas com filtro por mês e ano. Mê; Histórico de solicitações Visualize as solicitações enviadas com filtro por mês e ano. Mês Ano Colaborador Gestor Nome Data Departamento Cargo Nenhuma solicitaç; Nome Data Departamento Cargo Nenhuma solicitação de colaborador encontrada.; Colaboradores Data Departamento Nenhuma solicitação de gestor encontrada..

Títulos/seções visíveis: `Solicitação de Acesso ao Sistema`, `Solicitação de acesso ao sistema`, `Acesso indisponível para este perfil.`, `Histórico de solicitações`, `Formulário de colaborador`, `Status da solicitação`, `Quais os dias que você precisará de acesso ao sistema`, `Em quais horários você utilizará o sistema nos dias de semana selecionados`, `Justificativa (opcional)`, `Formulário de gestor`.

Tabelas identificadas: `master-colaborador-table` (`Nome`, `Data`, `Departamento`, `Cargo`); `master-gestor-table` (`Colaboradores`, `Data`, `Departamento`); `gestor-approval-table` (`Colaborador`, `Data`, `Dias`, `Status`, `Ações`).

Formulários identificados: `form-colaborador` método `post` com campos `perfil`, `solicitante_username`, `dias_acesso_colab`, `horario_inicio_colab`, `horario_fim_colab`, `justificativa_colab`; `form-gestor` método `post` com campos `perfil`, `solicitante_username`, `dias_acesso_gestor`, `horario_inicio_gestor`, `horario_fim_gestor`.

Campos relevantes: `master-month-filter` (select), `master-year-filter` (select), `perfil` (hidden), `solicitante_username_colab` (hidden), `dias_acesso_colab` (text), `horario_inicio_colab` (text), `horario_fim_colab` (text), `justificativa_colab` (textarea), `perfil` (hidden), `solicitante_username_gestor` (hidden), `dias_acesso_gestor` (text), `horario_inicio_gestor` (text), `horario_fim_gestor` (text).

Ações/botões identificados: `Início`, `Departamento ▾`, `Modelo padrão Departamento`, `Clientes`, `Área Dev`, `Usuário ▾`, `Departamento do usuário`, `Logout`, `Colaborador`, `Gestor`, `Enviar`, `Solicitação`, `Aprovação`, `Fechar`.

Foram identificados 2 elementos com comportamento de modal/dialog; recomenda-se centralizar em componente reutilizável com foco acessível e fechamento por teclado.

Fluxo esperado do usuário:

1. Acessar a página pela navegação interna ou link público/legado, conforme o caso.
2. O JavaScript relacionado carrega sessão, dados iniciais, filtros e tabelas.
3. O usuário preenche campos, seleciona filtros ou aciona botões de criação/edição/envio/download.
4. As APIs PHP recebem a requisição, aplicam validações e retornam JSON ou arquivo.
5. A interface atualiza status, tabelas, mensagens de sucesso/erro ou redireciona conforme o resultado.

---

## 4. Fluxo de dados

- Ao abrir a página, `auth.js` consulta `/api/storage.php?action=auth_session_status`; se a sessão não estiver autenticada, redireciona para `/index.html`. O usuário público fica em `sessionStorage` por meio de shim compatível com a chave legada `usuarioLogado`.

- Endpoints envolvidos: `/api/access_requests.php`, `/api/notifications.php`, `/api/storage.php`, `SolicitacaoAcesso/enviar_acesso.php`. As actions observadas são: `colaborador_status`, `decision`, `gestor_approvals`, `list`, `ack`, `alert_ack`, `clear_unread`, `list`, `audit_push`, `auth_logout`, `auth_session_status`, `get`, `set`, `user_presence_ping`, `submit`.

- Dados preenchidos pelo usuário partem dos formulários/campos HTML e são tratados por JavaScript associado à página. Para campos críticos, a validação do frontend deve ser apenas auxiliar; a regra definitiva precisa existir no PHP.

- Há manipulação de arquivos. Os dados saem como `FormData`, `multipart/form-data` ou dataURL, conforme a tela. O backend deve validar extensão, MIME real, tamanho, assinatura do arquivo, antivírus, armazenamento privado e permissão de download.

- Uso de armazenamento no navegador identificado nos scripts relacionados: `localStorage` em 40 ocorrência(s) e `sessionStorage` em 8 ocorrência(s). Tokens/segredos não devem ser armazenados nesses mecanismos; manter apenas estado não sensível.

- Cookies de sessão são tratados pelo backend em `api/auth.php` com `HttpOnly`, `SameSite=Lax` e `Secure` condicional a HTTPS/proxy. Ponto a validar em produção: HTTPS obrigatório, HSTS efetivo e domínio/path restritivos.

Campos/dados de entrada identificados no HTML: `master-month-filter` (select), `master-year-filter` (select), `perfil` (hidden), `solicitante_username_colab` (hidden), `dias_acesso_colab` (text), `horario_inicio_colab` (text), `horario_fim_colab` (text), `justificativa_colab` (textarea), `perfil` (hidden), `solicitante_username_gestor` (hidden), `dias_acesso_gestor` (text), `horario_inicio_gestor` (text), `horario_fim_gestor` (text)

Quando alguma action for indireta ou montada dinamicamente no JavaScript, considerar: **ponto a validar no código/backend**.

---

## 5. APIs e integrações relacionadas

### Status backend NestJS em 2026-05-13

- `AccessRequestsModule` foi criado com `POST /api-v2/access-requests`, `GET /api-v2/access-requests`, `GET /api-v2/access-requests/my/latest`, `GET /api-v2/access-requests/manager/approvals`, `GET /api-v2/access-requests/:id`, `POST /api-v2/access-requests/:id/approve` e `POST /api-v2/access-requests/:id/reject`.
- A primeira fatia cobre criacao por colaborador/gestor, fila do gestor, aprovacao/rejeicao, aliases legados de resposta, CSRF, permissoes granulares, escopo por departamento/gestor e auditoria.
- A tela HTML/JS ainda nao consome `/api-v2`; faltam baseline/migrations, backfill de `sama_access_requests`, notificacoes e migracao React.

### Status frontend React em 2026-05-15

- `portal-sama-web/src/pages/access-requests/AccessRequestPage.tsx` foi criado para a rota `/solicitacao-acesso`.
- A tela consome `GET /api-v2/access-requests/my/latest`, `POST /api-v2/access-requests`, `GET /api-v2/access-requests`, `GET /api-v2/access-requests/manager/approvals`, `POST /api-v2/access-requests/:id/approve` e `POST /api-v2/access-requests/:id/reject`.
- Foram adicionados `types/access-requests.ts`, `schemas/access-request.schema.ts` e `services/access-requests.service.ts`, com CSRF obtido pelo client centralizado antes de mutacoes.
- A navegacao React libera `/solicitacao-acesso` para sessoes autenticadas porque criacao e acompanhamento individual nao exigem `access_requests.read`; historico e decisoes continuam condicionados a permissoes na UX e validados no backend.
- Ainda faltam validacao com API v2, MySQL/backfill, usuarios reais, escopo por gestor/departamento/TI, notificacoes em navegador real e Playwright.

### Endpoint: `/api/access_requests.php`

- **Action observada:** `colaborador_status`
  - **Método provável:** GET
  - **Finalidade:** Consultar status da solicitação do colaborador.
  - **Dados enviados:** Sessão do colaborador.
  - **Dados retornados:** Solicitação/status.
  - **Validações esperadas:** Usuário dono.
  - **Riscos de segurança:** Colaborador consultar dados de outro.
  - **Melhorias recomendadas:** Derivar usuário da sessão.
- **Action observada:** `decision`
  - **Método provável:** POST
  - **Finalidade:** Aprovar ou declinar solicitação.
  - **Dados enviados:** ID da solicitação e decisão.
  - **Dados retornados:** Status final e notificação.
  - **Validações esperadas:** Gestor responsável e CSRF.
  - **Riscos de segurança:** Aprovação indevida.
  - **Melhorias recomendadas:** Auditoria, notificação e dupla validação de vínculo.
- **Action observada:** `gestor_approvals`
  - **Método provável:** GET
  - **Finalidade:** Listar aprovações pendentes do gestor.
  - **Dados enviados:** Sessão do gestor.
  - **Dados retornados:** Solicitações pendentes.
  - **Validações esperadas:** Gestor responsável.
  - **Riscos de segurança:** Gestor visualizar solicitação de outro gestor.
  - **Melhorias recomendadas:** Filtrar por `gestor_aprovador_username` no backend.
- **Action observada:** `list`
  - **Método provável:** GET
  - **Finalidade:** Listar solicitações para master.
  - **Dados enviados:** Mês e ano.
  - **Dados retornados:** Solicitações de colaborador/gestor.
  - **Validações esperadas:** Somente master.
  - **Riscos de segurança:** Exposição de agenda/horários.
  - **Melhorias recomendadas:** Mascarar dados não necessários e paginar.

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

### Endpoint: `SolicitacaoAcesso/enviar_acesso.php`

- **Action observada:** `submit`
  - **Método provável:** POST
  - **Finalidade:** Receber formulário de solicitação de acesso e enviar notificações/e-mails.
  - **Dados enviados:** Campos de perfil, datas, horários e justificativa.
  - **Dados retornados:** JSON `status` e mensagem.
  - **Validações esperadas:** Usuário válido, perfil ativo e regras de horário.
  - **Riscos de segurança:** Código usa `display_errors=1`; pode expor detalhes internos em produção. Também precisa CSRF/session hardening.
  - **Melhorias recomendadas:** Desabilitar display_errors, validar CSRF, derivar solicitante da sessão e registrar auditoria.

---

## 6. Regras de negócio identificadas

### Regras identificadas no frontend

- Fluxo muda conforme perfil: colaborador solicita, gestor aprova, master consulta histórico.
- Dias úteis exigem faixa de horário entre 17:00 e 23:59.
- Filtros por mês/ano organizam histórico.

### Regras que devem existir obrigatoriamente no backend

- Derivar solicitante da sessão.
- Validar dias/horários no servidor.
- Gestor só decide solicitação vinculada a ele.
- Enviar notificações/e-mails e registrar auditoria.
- Uploads devem ser validados por extensão, MIME, tamanho, assinatura, antivírus e armazenados fora da pasta pública.

---

## 7. Análise de segurança

### 7.1 Autenticação

`auth.js` está relacionado à página. A validação inicial ocorre no frontend chamando `/api/storage.php?action=auth_session_status`; endpoints PHP precisam manter a validação real de sessão.

### 7.2 Autorização

A autorização deve considerar perfil, departamento, propriedade do recurso e vínculo com cliente/processo. O frontend pode esconder botões, mas a decisão precisa estar no PHP.

### 7.3 Proteção contra XSS

Foram identificadas 40 ocorrência(s) de `innerHTML` nos scripts vinculados. Evitar inserir dados de API com `innerHTML`; preferir `textContent` ou sanitização contextual.

### 7.4 Proteção contra CSRF

Para actions de mutação, o padrão do projeto usa `X-CSRF-Token` via `SamaAuth.buildHeaders` e `sama_require_csrf()` em APIs críticas. Validar cobertura em todas as actions desta página.

### 7.5 Proteção contra SQL Injection

Os endpoints PHP analisados usam `PDO`/`prepare` em diversas consultas. Ainda assim, qualquer consulta dinâmica, filtro, `ORDER BY`, nome de tabela/coluna ou action genérica deve usar allowlist e prepared statements. Ponto a validar em cada endpoint relacionado.

### 7.6 Segurança em uploads

Há upload/manipulação de arquivo. Exigir allowlist de extensão, MIME real, assinatura mágica, tamanho máximo, nome gerado, quarentena, ClamAV, storage privado e download autenticado/autorizado.

### 7.7 Dados sensíveis

Dados sensíveis envolvidos: datas, horários, justificativa, perfil, colaborador, gestor, aprovação e histórico de solicitações. Não expor esses dados no HTML inicial, em logs do navegador, em mensagens de erro, em query strings permanentes ou em `localStorage`.

### 7.8 Logs e auditoria

Ações críticas desta página devem gerar trilha de auditoria com usuário derivado da sessão, entidade, antes/depois quando aplicável, IP, user-agent e timestamp. Evitar registrar valores sensíveis em claro.

---

## 8. Checklist de segurança da página

- [x] Página protegida contra acesso não autenticado.
- [x] Permissões validadas no backend.
- [x] Inputs validados no frontend.
- [ ] Inputs validados no backend.
- [ ] Dados sensíveis não expostos no HTML.
- [ ] Dados sensíveis não expostos no JavaScript.
- [ ] Tokens não armazenados de forma insegura.
- [x] Cookies configurados com `HttpOnly`, `Secure` e `SameSite` no backend de autenticação; validar HTTPS/proxy em produção.
- [x] Requisições críticas protegidas contra CSRF na API v2; fluxo real em navegador/homologacao segue pendente.
- [ ] Dados renderizados no DOM protegidos contra XSS.
- [ ] Endpoints protegidos contra SQL Injection.
- [ ] Uploads validados por extensão, MIME type e conteúdo.
- [ ] Arquivos armazenados fora da pasta pública.
- [ ] Downloads protegidos por autenticação e autorização.
- [x] Ações críticas registradas em auditoria na API v2; validacao com MySQL real segue pendente.
- [ ] Erros tratados sem expor detalhes internos.
- [ ] Logs não expõem dados sensíveis.

---

## 9. Problemas técnicos encontrados

- Arquivos JavaScript extensos/acoplados à tela: `DEV/dev-data.js` (2582 linhas), `global.js` (2433 linhas), `SolicitacaoAcesso/acesso.js` (834 linhas).
- Uso de `innerHTML` em scripts relacionados (40 ocorrência(s)), exigindo revisão de XSS.
- Uso de `localStorage` em scripts relacionados (40 ocorrência(s)); não armazenar tokens ou dados sensíveis.
- Persistência genérica por `/api/storage.php?action=get|set` e chaves de domínio dificulta contratos claros de API, testes e autorização por recurso.
- Upload/arquivo exige maior segregação entre UI, validação, armazenamento, antivírus e autorização de download.
- `SolicitacaoAcesso/enviar_acesso.php` contém `display_errors=1`, inadequado para produção por risco de exposição de detalhes internos.

---

## 10. Melhorias recomendadas

### 10.1 Melhorias rápidas

- Centralizar chamadas `fetch` em um client com tratamento de erro, CSRF e timeout.
- Padronizar mensagens de erro sem expor detalhes internos.
- Garantir que botões sensíveis sejam ocultados e bloqueados por permissão no backend.
- Revisar uploads com allowlist, MIME real, limite de tamanho e quarentena antes do storage final.

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
- TanStack Query para cache/estado de servidor e um componente `DataTable` reutilizável com paginação e filtros server-side.
- Componente `SecureUploadField` com validação visual de extensão/tamanho e progresso, mantendo validação definitiva no backend.

### Backend

- NestJS com storage privado, DTOs/Pipes de validação, validação MIME/assinatura de arquivo, ClamAV e endpoints protegidos ou links temporários para download.

### Segurança

- Sessão server-side com cookies `HttpOnly`, `Secure` e `SameSite`, evitando armazenar tokens sensíveis em `localStorage`.
- RBAC por perfil e departamento, validado no backend em todas as actions.
- Upload seguro com allowlist de extensão, MIME real, assinatura mágica, quarentena, ClamAV e armazenamento fora da pasta pública.

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
- `DataTable`
- `StatusBadge`
- `FilterBar`
- `FormSection`
- `ValidatedInput`
- `ConfirmModal`
- `UploadField`
- `SecureDownloadButton`
- `FileStatusBadge`

Serviços sugeridos:
- `authService`
- `auditService`
- `notificationService`

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
- Validar mensagens para extensão/tamanho inválidos no frontend.

### Testes de integração

- Endpoint deve rejeitar requisição sem autenticação quando a página for interna.
- Endpoint deve rejeitar usuário sem perfil/departamento permitido.
- Erro da API deve ser tratado na interface sem expor stack trace.
- Upload inválido deve ser bloqueado pelo backend antes de persistir arquivo.

### Testes E2E

- Usuário autorizado acessa a página e executa o fluxo principal.
- Usuário sem permissão não visualiza nem executa ações críticas.

### Testes de segurança

- Tentar XSS em campos livres e dados retornados pela API.
- Tentar IDOR alterando IDs, client_id, doc_id ou token.
- Verificar CSRF em actions POST/PUT/PATCH/DELETE.
- Enviar arquivo com extensão dupla, MIME falso, payload EICAR e nome malicioso.

---

## 14. Conclusão da página

A página `SolicitacaoAcesso/solicitacao-acesso.html` atua no módulo **Solicitação de Acesso** e manipula **datas, horários, justificativa, perfil, colaborador, gestor, aprovação e histórico de solicitações**. Os principais riscos são: upload/download de arquivos. A principal recomendação é reforçar validações server-side, autorização por recurso, auditoria e componentização gradual.

- **Nível de criticidade:** Alto
- **Prioridade de refatoração:** Alta
- **Observações finais:** manter o HTML como camada de apresentação; regras de permissão, validação, persistência e auditoria devem residir no backend. Quando houver incerteza, tratar como **ponto a validar no código/backend**.


---

## 15. Atualização de stack alvo — TypeScript/NestJS

Esta seção complementa a análise original da página com a decisão técnica posterior de migrar o backend PHP para **Node.js + TypeScript + NestJS** e evoluir o frontend para **React + TypeScript + Vite**.

### 15.1 Mapeamento da tela para a arquitetura alvo

- **Arquivo HTML atual:** `SolicitacaoAcesso/solicitacao-acesso.html`
- **Rota React sugerida:** `/solicitacao-acesso`
- **Componente React sugerido:** `AccessRequestPage.tsx`
- **Backend alvo:** `/api-v2`, implementado em NestJS
- **Banco principal:** MySQL 8 com Prisma ORM

### 15.2 Módulos NestJS relacionados

- `AccessRequestsModule`
- `NotificationsModule`
- `AuditModule`

### 15.3 Arquivos atuais que devem ser usados como referência de migração

- `SolicitacaoAcesso/solicitacao-acesso.html`
- `api/access_requests.php`
- `SolicitacaoAcesso/enviar_acesso.php`

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
