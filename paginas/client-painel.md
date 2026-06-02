# [PARCIAL] Painel do Cliente

## 1. Identificação da página

- **Arquivo HTML:** `Client/painel.html`
- **Módulo:** Clientes
- **Arquivos CSS relacionados:** `global.css`, `DEV/dev.css`, `https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css`
- **Arquivos JavaScript relacionados:** `auth.js`, `DEV/dev-data.js`, `Client/painel.js`, `global.js`
- **APIs/endpoints relacionados:** `/api/client_documents.php actions: add_custom, download, list, upload`, `/api/notifications.php actions: ack, alert_ack, clear_unread, list`, `/api/storage.php actions: audit_push, auth_logout, auth_session_status, get, set, user_presence_ping`
- **Perfil de usuário provável:** Master, gestor, colaborador responsável ou usuário com permissão no departamento do cliente
- **Nível de criticidade:** Crítico
- **Prioridade de refatoração:** Urgente

**Status backend 2026-05-13 10:50:** `ClientsModule` criou `GET /api-v2/clients/:id`, `PATCH /api-v2/clients/:id` e `GET /api-v2/clients/:id/dashboard`; `DocumentsModule` continua cobrindo `/api-v2/clients/:clientId/documents`. Falta integrar o painel e validar vinculos reais cliente-usuario/gestor.

**Atualizacao React 2026-06-02:** `portal-sama-web/src/pages/clients/ClientDashboardPage.tsx` agora exibe `Equipe e responsaveis`, consumindo `GET /api-v2/clients/:clientId/assignments` por `listClientAssignments()`. A secao respeita `client_assignments.read` e lista departamento, responsavel, gestor, tipo, status e periodo. A acao `Nova responsabilidade` cria `POST /api-v2/clients/:clientId/assignments` por `createClientAssignment()`, usando departamento, responsavel operacional, tipo, inicio e gestor opcional, protegida por `client_assignments.create`. A acao `Editar` chama `PATCH /api-v2/client-assignments/:id` por `updateClientAssignment()`, protegida por `client_assignments.update`, permitindo ajustar departamento, responsavel, gestor, tipo, status e periodo. A acao `Encerrar` chama `POST /api-v2/client-assignments/:id/end` por `endClientAssignment()`, protegida por `client_assignments.end`, com data e motivo. A acao `Transferir` chama `POST /api-v2/client-assignments/transfer` por `transferClientAssignments()`, protegida por `client_assignments.transfer`, com novo responsavel, gestor opcional, data efetiva e motivo. Ainda falta validar no EasyPanel com dados reais, backfill e carteiras/filtros normalizados.

---

## 2. Objetivo da página

Exibir e editar dados de uma empresa cliente, responsáveis por departamento, acessos, documentos, certificados e histórico de vida da empresa.

- **Problema resolvido:** centraliza o fluxo de clientes para reduzir consulta manual, duplicação operacional e perda de rastreabilidade.
- **Quem utiliza:** Master, gestor, colaborador responsável ou usuário com permissão no departamento do cliente.
- **Etapa do fluxo:** Clientes dentro do Portal Sama.
- **Dados/documentos manipulados:** dados cadastrais de empresa, CNPJ, contatos, responsáveis, acessos, documentos empresariais, certificados digitais e histórico.
- **Observação:** ponto a validar no código/backend para regras não explícitas no HTML.

---

## 3. Funcionamento detalhado da interface

Cabeçalho identificado com área de usuário, departamento e ação de logout ou retorno, carregado em conjunto com `global.js`/`auth.js` quando presentes.

Navegação lateral/superior com atalhos para Início, Departamento, Clientes e Área Dev, reaproveitando a identidade visual global do portal.

Conteúdo principal identificado: Portal Sama Contabilidade Painel de Visualização (Cliente) Nome do colaborador ▾ Departamento do usuário Logout ← Voltar para todos os clientes Cliente • • B AT; ← Voltar para todos os clientes Cliente • • B ATIVO Ativo Inativo Bloqueado Geral Responsáveis Acessos Documentos 0 Certificados Vida da empresa Informações da; Informações da empresa Nome fantasia Razão social CNPJ Rank ID da empresa Regime tributário Atividade Telefone E-mail Endereço Grupo de empresas Status Status h; Responsáveis pelos departamentos Visão consolidada dos responsáveis em cada área para este cliente. Departamento Responsável Status Ação Nen; Acessos do cliente Credenciais por cliente (Portais). Você pode adicionar/atualizar acessos aqui. Cada alteração fica registrada. Portal Mun.

Títulos/seções visíveis: `Portal Sama Contabilidade`, `Cliente`, `Informações da empresa`, `Resumo por departamentos`, `Edição rápida (apenas administradores)`, `Resumo`, `Editar vida da empresa`, `Historico`.

Tabelas identificadas: `tabelaClienteDepartamentos` (`Departamento`, `Responsável`, `Status`, `Ação`); `tabelaClienteAcessos` (`Portal`, `Login`, `Senha`, `Ações`); `tabelaClienteCerts` (`Nome`, `Arquivo`, `Senha`, `Ações`).

Formulários identificados: `formEditarCliente` método `GET` com campos `editNomeFantasia`, `editRazaoSocial`, `editCnpj`, `editIdEmpresa`, `editRegime`, `editRank`, `editTelefone`, `editAtividade`, `editEmail`, `editCep`, `editEndereco`, `editComplemento`.

Campos relevantes: `editNomeFantasia` (text), `editRazaoSocial` (text), `editCnpj` (text), `editIdEmpresa` (text), `editRegime` (select), `editRank` (select), `editTelefone` (text), `editAtividade` (text), `editEmail` (email), `editCep` (text), `editEndereco` (text), `editComplemento` (text), `editBairro` (text), `editCidade` (text), `editEstado` (select), `editTemGrupo` (checkbox), `editGrupoNome` (text), `acessosMunicipioInput` (text) e mais 4 campo(s).

Ações/botões identificados: `Início`, `Departamento ▾`, `Modelo padrão Departamento`, `Clientes`, `Área Dev`, `Nome do colaborador ▾`, `Departamento do usuário`, `Logout`, `← Voltar para todos os clientes`, `ATIVO`, `Ativo`, `Inativo`, `Bloqueado`, `Geral`, `... mais 15 item(ns)`.

Fluxo esperado do usuário:

1. Acessar a página pela navegação interna ou link público/legado, conforme o caso.
2. O JavaScript relacionado carrega sessão, dados iniciais, filtros e tabelas.
3. O usuário preenche campos, seleciona filtros ou aciona botões de criação/edição/envio/download.
4. As APIs PHP recebem a requisição, aplicam validações e retornam JSON ou arquivo.
5. A interface atualiza status, tabelas, mensagens de sucesso/erro ou redireciona conforme o resultado.

---

## 4. Fluxo de dados

- Ao abrir a página, `auth.js` consulta `/api/storage.php?action=auth_session_status`; se a sessão não estiver autenticada, redireciona para `/index.html`. O usuário público fica em `sessionStorage` por meio de shim compatível com a chave legada `usuarioLogado`.

- Endpoints envolvidos: `/api/client_documents.php`, `/api/notifications.php`, `/api/storage.php`. As actions observadas são: `add_custom`, `download`, `list`, `upload`, `ack`, `alert_ack`, `clear_unread`, `list`, `audit_push`, `auth_logout`, `auth_session_status`, `get`, `set`, `user_presence_ping`.

- Status de migração em 2026-05-11 13:55: `DocumentsModule` inicial foi criado na API v2 para template, checklist/listagem por cliente, resumo de pendências, requisito customizado, detalhe, upload com quarentena/scanner configuravel, download, revisão de status e arquivamento lógico de documentos. A página ainda não foi conectada a `/api-v2/documents`; manter o PHP legado até validar MySQL, storage persistente, escopo, seed `documents.review` e ClamAV strict no host.

- Dados preenchidos pelo usuário partem dos formulários/campos HTML e são tratados por JavaScript associado à página. Para campos críticos, a validação do frontend deve ser apenas auxiliar; a regra definitiva precisa existir no PHP.

- Há manipulação de arquivos. Os dados saem como `FormData`, `multipart/form-data` ou dataURL, conforme a tela. O backend deve validar extensão, MIME real, tamanho, assinatura do arquivo, antivírus, armazenamento privado e permissão de download.

- Uso de armazenamento no navegador identificado nos scripts relacionados: `localStorage` em 40 ocorrência(s) e `sessionStorage` em 8 ocorrência(s). Tokens/segredos não devem ser armazenados nesses mecanismos; manter apenas estado não sensível.

- Cookies de sessão são tratados pelo backend em `api/auth.php` com `HttpOnly`, `SameSite=Lax` e `Secure` condicional a HTTPS/proxy. Ponto a validar em produção: HTTPS obrigatório, HSTS efetivo e domínio/path restritivos.

Campos/dados de entrada identificados no HTML: `editNomeFantasia` (text), `editRazaoSocial` (text), `editCnpj` (text), `editIdEmpresa` (text), `editRegime` (select), `editRank` (select), `editTelefone` (text), `editAtividade` (text), `editEmail` (email), `editCep` (text), `editEndereco` (text), `editComplemento` (text), `editBairro` (text), `editCidade` (text), `editEstado` (select), `editTemGrupo` (checkbox), `editGrupoNome` (text), `acessosMunicipioInput` (text), `acessosEmailInput` (email), `docsAddNome` (text), `docsAddDept` (select), `vidaDeptSelect` (select)

Quando alguma action for indireta ou montada dinamicamente no JavaScript, considerar: **ponto a validar no código/backend**.

---

## 5. APIs e integrações relacionadas

### Endpoint: `/api/client_documents.php`

- **Action observada:** `add_custom`
  - **Método provável:** POST
  - **Finalidade:** Criar documento customizado para cliente.
  - **Dados enviados:** JSON com `client_id`, `label`, `department`.
  - **Dados retornados:** Chave do novo documento.
  - **Validações esperadas:** CSRF, papel master/gestor e departamento permitido.
  - **Riscos de segurança:** Criação indevida de obrigação documental.
  - **Melhorias recomendadas:** Auditar e limitar nomes/escopo.
- **Action observada:** `download`
  - **Método provável:** GET
  - **Finalidade:** Baixar documento do cliente.
  - **Dados enviados:** `id` do documento.
  - **Dados retornados:** Arquivo binário.
  - **Validações esperadas:** Autenticação e autorização por departamento.
  - **Riscos de segurança:** Download indevido de documentos empresariais.
  - **Melhorias recomendadas:** Usar assinatura temporária e registrar download.
- **Action observada:** `list`
  - **Método provável:** GET
  - **Finalidade:** Listar documentos do cliente e status de upload/confirmação.
  - **Dados enviados:** `client_id`.
  - **Dados retornados:** Itens, metadados, status e extensões permitidas.
  - **Validações esperadas:** Autorização por departamento.
  - **Riscos de segurança:** IDOR em `client_id` ou `doc_id`.
  - **Melhorias recomendadas:** Validar permissão por cliente/departamento no backend.
- **Action observada:** `upload`
  - **Método provável:** POST
  - **Finalidade:** Enviar documentos do cliente.
  - **Dados enviados:** Multipart com `client_id` e arquivos `file__*`.
  - **Dados retornados:** Itens atualizados e erros parciais.
  - **Validações esperadas:** CSRF, departamento permitido, extensão/MIME/tamanho e scanner/quarentena.
  - **Riscos de segurança:** Upload malicioso, path traversal, armazenamento público.
  - **Melhorias recomendadas:** Manter arquivos fora do webroot, varrer com ClamAV e usar links temporários.

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

- Edição rápida de cliente deve ficar restrita a administradores.
- Documentos são agrupados por departamento e podem ter upload/download.
- Certificados vinculados ao cliente aparecem em aba própria.
- Vida da empresa possui histórico de autoria.

### Regras que devem existir obrigatoriamente no backend

- Validar permissão por cliente/departamento.
- Bloquear atualização de dados cadastrais por usuário sem papel adequado.
- Auditar upload/download e alteração de vida da empresa.
- Garantir IDOR protection em `client_id` e `doc_id`.
- Uploads devem ser validados por extensão, MIME, tamanho, assinatura, antivírus e armazenados fora da pasta pública.

---

## 7. Análise de segurança

### 7.1 Autenticação

`auth.js` está relacionado à página. A validação inicial ocorre no frontend chamando `/api/storage.php?action=auth_session_status`; endpoints PHP precisam manter a validação real de sessão.

### 7.2 Autorização

A autorização deve considerar perfil, departamento, propriedade do recurso e vínculo com cliente/processo. O frontend pode esconder botões, mas a decisão precisa estar no PHP.

### 7.3 Proteção contra XSS

Foram identificadas 26 ocorrência(s) de `innerHTML` nos scripts vinculados. Evitar inserir dados de API com `innerHTML`; preferir `textContent` ou sanitização contextual.

### 7.4 Proteção contra CSRF

Para actions de mutação, o padrão do projeto usa `X-CSRF-Token` via `SamaAuth.buildHeaders` e `sama_require_csrf()` em APIs críticas. Validar cobertura em todas as actions desta página.

### 7.5 Proteção contra SQL Injection

Os endpoints PHP analisados usam `PDO`/`prepare` em diversas consultas. Ainda assim, qualquer consulta dinâmica, filtro, `ORDER BY`, nome de tabela/coluna ou action genérica deve usar allowlist e prepared statements. Ponto a validar em cada endpoint relacionado.

### 7.6 Segurança em uploads

Há upload/manipulação de arquivo. Exigir allowlist de extensão, MIME real, assinatura mágica, tamanho máximo, nome gerado, quarentena, ClamAV, storage privado e download autenticado/autorizado.

### 7.7 Dados sensíveis

Dados sensíveis envolvidos: dados cadastrais de empresa, CNPJ, contatos, responsáveis, acessos, documentos empresariais, certificados digitais e histórico. Não expor esses dados no HTML inicial, em logs do navegador, em mensagens de erro, em query strings permanentes ou em `localStorage`.

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
- [ ] Uploads validados por extensão, MIME type e conteúdo.
- [ ] Arquivos armazenados fora da pasta pública.
- [ ] Downloads protegidos por autenticação e autorização.
- [ ] Ações críticas registradas em auditoria.
- [ ] Erros tratados sem expor detalhes internos.
- [ ] Logs não expõem dados sensíveis.

---

## 9. Problemas técnicos encontrados

- Arquivos JavaScript extensos/acoplados à tela: `DEV/dev-data.js` (2582 linhas), `Client/painel.js` (2503 linhas), `global.js` (2433 linhas).
- Uso de `innerHTML` em scripts relacionados (26 ocorrência(s)), exigindo revisão de XSS.
- Uso de `localStorage` em scripts relacionados (40 ocorrência(s)); não armazenar tokens ou dados sensíveis.
- Persistência genérica por `/api/storage.php?action=get|set` e chaves de domínio dificulta contratos claros de API, testes e autorização por recurso.
- Upload/arquivo exige maior segregação entre UI, validação, armazenamento, antivírus e autorização de download.

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
- API REST por recurso (`clients`, `client-documents`, `certificates`) com policies por cliente/departamento.

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
- `clientService`
- `documentService`
- `certificateService`

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

A página `Client/painel.html` atua no módulo **Clientes** e manipula **dados cadastrais de empresa, CNPJ, contatos, responsáveis, acessos, documentos empresariais, certificados digitais e histórico**. Os principais riscos são: upload/download de arquivos. A principal recomendação é reforçar validações server-side, autorização por recurso, auditoria e componentização gradual.

- **Nível de criticidade:** Crítico
- **Prioridade de refatoração:** Urgente
- **Observações finais:** manter o HTML como camada de apresentação; regras de permissão, validação, persistência e auditoria devem residir no backend. Quando houver incerteza, tratar como **ponto a validar no código/backend**.


---

## 15. Atualização de stack alvo — TypeScript/NestJS

Esta seção complementa a análise original da página com a decisão técnica posterior de migrar o backend PHP para **Node.js + TypeScript + NestJS** e evoluir o frontend para **React + TypeScript + Vite**.

### 15.1 Mapeamento da tela para a arquitetura alvo

- **Arquivo HTML atual:** `Client/painel.html`
- **Rota React sugerida:** `/clientes/:clientId/painel`
- **Componente React sugerido:** `ClientDashboardPage.tsx`
- **Backend alvo:** `/api-v2`, implementado em NestJS
- **Banco principal:** MySQL 8 com Prisma ORM

### 15.2 Módulos NestJS relacionados

- `ClientsModule`
- `DocumentsModule`
- `AuditModule`

### 15.3 Arquivos atuais que devem ser usados como referência de migração

- `Client/painel.js`
- `api/client_documents.php`
- `api/storage.php`

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

## 16. Atualizacao de implementacao - 2026-05-15 08:59 -03:00

- A rota React `/clientes/:clientId/painel` passou a renderizar `portal-sama-web/src/pages/clients/ClientDashboardPage.tsx`.
- A tela consome `/api-v2/clients/:id/dashboard` por `clients.service.ts`, exibindo dados cadastrais e resumo de documentos, certificados, propostas e contratos.
- Esta fatia substitui o placeholder por painel inicial de consulta, mas ainda nao migra as abas completas de documentos, upload/download, certificados por cliente, acessos e vida da empresa.
- Dados sensiveis continuam vindo da API autenticada; a tela nao grava dados de cliente em `localStorage`.
- Validacao local: `npm.cmd run lint` passou; `npm.cmd run build` falhou inicialmente por tipo do schema Zod/React Hook Form usado na tela de clientes, foi corrigido e passou.
- Pendencias: validar com usuario real, MySQL/storage reais, escopo por cliente/gestor, `DocumentsModule`, `CertificatesModule` filtrado por cliente e Playwright.

## 17. Atualizacao de implementacao - 2026-05-15 09:16 -03:00

- O painel React ganhou `portal-sama-web/src/pages/clients/ClientDocumentsPanel.tsx`.
- A secao consome `/api-v2/clients/:clientId/documents`, `/api-v2/documents/clients/:clientId/upload`, `/api-v2/documents/:id/download`, `/api-v2/documents/:id/status` e `/api-v2/documents/custom-requirements`.
- Foram adicionados `portal-sama-web/src/types/documents.ts`, `portal-sama-web/src/schemas/document.schema.ts` e `portal-sama-web/src/services/documents.service.ts`.
- A primeira fatia cobre checklist, estatisticas, upload multipart, download blob protegido, revisao de status e requisito customizado por permissao.
- A tela consulta documentos somente com `documents.read`; acoes usam `documents.upload`, `documents.download`, `documents.review` e `documents.requirements` na UX, mantendo a decisao final no backend.
- Validacao local: `npm.cmd run lint` e `npm.cmd run build` passaram; um aviso inicial de hook foi corrigido com `useMemo`.
- Pendencias: validar upload/download/revisao reais com API v2, MySQL, storage privado, ClamAV/EICAR, dados de escopo cliente/gestor e Playwright.

## 18. Atualizacao de implementacao - 2026-05-18 12:27 -03:00

- A rota React interna `/documentos` passou a renderizar `portal-sama-web/src/pages/documents/DocumentsPage.tsx`.
- A central consome `/api-v2/documents`, `/api-v2/documents/:id/download`, `/api-v2/documents/:id/status`, `/api-v2/documents/:id`, `/api-v2/documents/public-tokens` e `/api-v2/documents/required-pending-summary`.
- `types/documents.ts`, `document.schema.ts` e `documents.service.ts` foram ampliados para listagem global, upload interno, revisao, arquivamento, links publicos por token, revogacao e pendencias obrigatorias.
- A tela usa permissoes `documents.read/upload/download/review/delete/requirements/public_tokens` apenas na UX; JWT, CSRF, RBAC, escopo, storage privado, scanner e auditoria continuam no backend.
- Validacao local: `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd audit --audit-level=moderate` e `git diff --check` passaram; o build emitiu apenas o aviso conhecido de chunk acima de 500 kB e o diff final teve apenas avisos LF/CRLF do Git no Windows.
- Pendencias: validar com API v2/MySQL/storage/ClamAV reais, backfill, usuarios/permissoes reais, links legados, homologacao HTTPS e Playwright.

## 19. Atualizacao de testes - 2026-05-22 10:02 -03:00

- A central React `/documentos` ganhou smoke Playwright persistido em `portal-sama-web/tests/e2e/smoke.spec.ts`.
- O teste usa sessao autenticada e mocks de `/api-v2/documents`, `/api-v2/clients`, `/api-v2/documents/public-tokens` e `/api-v2/documents/required-pending-summary`.
- A cobertura valida documento privado listado, cliente vinculado, extensoes permitidas, link publico ativo, resumo de pendencias obrigatorias e abertura do formulario de revisao.
- Validacao local: `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e` com 7 testes Chromium e `git diff --check` passaram.
- Pendencias: validar upload/download/revisao reais com API v2, MySQL, storage privado, ClamAV/EICAR, dados de escopo cliente/gestor, usuarios/permissoes reais e homologacao HTTPS.

## 20. Atualizacao de backend - 2026-06-02 16:30 -03:00

- O backend do `DocumentsModule` passou a usar `client_department_assignments` ativas no escopo de documentos internos.
- Para usuarios departamentais, o documento precisa estar no departamento permitido e, quando o cliente ja tem responsabilidade normalizada ativa, essa responsabilidade tambem precisa apontar para o mesmo departamento.
- A regra foi aplicada em listagem, checklist por cliente, detalhe, historico, download, revisao, arquivamento e upload interno.
- Quando nao existe responsabilidade normalizada ativa para o cliente, `document.department` segue como fallback temporario ate backfill/conferencia real.
- Validacao local: `documents.service.spec.ts`, TypeScript sem emit, lint, build e suite completa da API passaram.
- Pendencias: validar no EasyPanel com MySQL/storage/ClamAV reais, usuarios reais por perfil/departamento, auditoria persistida e documentos sem departamento.
