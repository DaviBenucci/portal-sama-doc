# [PARCIAL] Editor de Proposta

## 1. Identificação da página

- **Arquivo HTML:** `Legalizacao/proposta.html`
- **Módulo:** Legalização / Propostas
- **Arquivos CSS relacionados:** `DEV/dev.css`, `global.css`, `Legalizacao/proposta.css`, `https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css`
- **Arquivos JavaScript relacionados:** `auth.js`, `DEV/dev-data.js`, `global.js`, `Legalizacao/proposta.js`
- **APIs/endpoints relacionados:** `/api/legalizacao.php actions: get, list, listTemplates, send`, `/api/notifications.php actions: ack, alert_ack, clear_unread, list`, `/api/propostas.php actions: get, list, save, send`, `/api/storage.php actions: audit_push, auth_logout, auth_session_status, get, set, user_presence_ping`
- **Perfil de usuário provável:** Usuário interno autorizado no fluxo de Legalização/Comercial
- **Nível de criticidade:** Alto
- **Prioridade de refatoração:** Alta

---

## 2. Objetivo da página

Criar, editar, salvar e gerar link público para propostas comerciais, abertura e serviços extras.

- **Problema resolvido:** centraliza o fluxo de legalização / propostas para reduzir consulta manual, duplicação operacional e perda de rastreabilidade.
- **Quem utiliza:** Usuário interno autorizado no fluxo de Legalização/Comercial.
- **Etapa do fluxo:** Legalização / Propostas dentro do Portal Sama.
- **Dados/documentos manipulados:** dados comerciais, honorários, forma de pagamento, destinatário, empresa, templates e link público.
- **Observação:** ponto a validar no código/backend para regras não explícitas no HTML.

Status em 2026-05-15 15:06: a rota React `/legalizacao/propostas`/`/legalizacao/propostas/:id` foi criada em `portal-sama-web/src/pages/proposals/ProposalPage.tsx`, consumindo `/api-v2/proposals` para filtros, listagem, criacao, edicao, detalhe, envio por link publico e aprovacao/rejeicao interna. A renderizacao dos campos comerciais usa texto/escape padrao do React, sem `innerHTML`. Falta validar com API v2/MySQL/backfill reais, usuarios/permissoes reais, compatibilidade com tokens legados, HTTPS/homologacao e Playwright.

---

## 3. Funcionamento detalhado da interface

A tela é minimalista e atua como rota de redirecionamento/compatibilidade. Não há menu, formulário, tabela ou lógica visual relevante no HTML; o navegador executa o redirecionamento definido no próprio documento.

Fluxo esperado do usuário:

1. Acessar a página pela navegação interna ou link público/legado, conforme o caso.
2. O JavaScript relacionado carrega sessão, dados iniciais, filtros e tabelas.
3. O usuário preenche campos, seleciona filtros ou aciona botões de criação/edição/envio/download.
4. As APIs PHP recebem a requisição, aplicam validações e retornam JSON ou arquivo.
5. A interface atualiza status, tabelas, mensagens de sucesso/erro ou redireciona conforme o resultado.

---

## 4. Fluxo de dados

- Ao abrir a página, `auth.js` consulta `/api/storage.php?action=auth_session_status`; se a sessão não estiver autenticada, redireciona para `/index.html`. O usuário público fica em `sessionStorage` por meio de shim compatível com a chave legada `usuarioLogado`.

- Endpoints envolvidos: `/api/legalizacao.php`, `/api/notifications.php`, `/api/propostas.php`, `/api/storage.php`. As actions observadas são: `get`, `list`, `listTemplates`, `send`, `ack`, `alert_ack`, `clear_unread`, `list`, `audit_push`, `auth_logout`, `auth_session_status`, `get`, `set`, `user_presence_ping`, `get`, `list`, `save`, `send`.

- Há manipulação de arquivos. Os dados saem como `FormData`, `multipart/form-data` ou dataURL, conforme a tela. O backend deve validar extensão, MIME real, tamanho, assinatura do arquivo, antivírus, armazenamento privado e permissão de download.

- Uso de armazenamento no navegador identificado nos scripts relacionados: `localStorage` em 40 ocorrência(s) e `sessionStorage` em 8 ocorrência(s). Tokens/segredos não devem ser armazenados nesses mecanismos; manter apenas estado não sensível.

- Cookies de sessão são tratados pelo backend em `api/auth.php` com `HttpOnly`, `SameSite=Lax` e `Secure` condicional a HTTPS/proxy. Ponto a validar em produção: HTTPS obrigatório, HSTS efetivo e domínio/path restritivos.

Campos/dados de entrada identificados no HTML: `f_tipo` (text), `f_header_tpl` (select), `f_destinatario` (text), `f_empresa` (text), `f_cidade_data` (text), `f_honorario_mensal` (text), `f_dia_pagamento` (text), `f_parcelas` (text), `f_honorario_reduzido` (text), `f_qtd_lanc_contabeis` (number), `f_qtd_lanc_fiscais` (number), `f_qtd_funcionarios` (number), `f_honorario_abertura` (text), `f_valor_abertura` (text), `f_desconto_abertura` (text), `f_total_abertura` (text), `f_forma_pagamento` (text), `rt_heading` (select), `rt_font` (select), `rt_size` (select), `rt_spacing` (select), `rt_color` (color), `rt_highlight` (color), `f_assinatura_nome` (text), `p_link` (text), `p_feedback` (textarea)

Quando alguma action for indireta ou montada dinamicamente no JavaScript, considerar: **ponto a validar no código/backend**.

---

## 5. APIs e integrações relacionadas

### Endpoint: `/api/legalizacao.php`

- **Action observada:** `get`
  - **Método provável:** GET
  - **Finalidade:** Carregar contrato por ID.
  - **Dados enviados:** ID do contrato.
  - **Dados retornados:** Dados completos do contrato.
  - **Validações esperadas:** Autorização por perfil.
  - **Riscos de segurança:** IDOR por ID previsível.
  - **Melhorias recomendadas:** IDs opacos e política por recurso.
- **Action observada:** `list`
  - **Método provável:** GET
  - **Finalidade:** Listar contratos.
  - **Dados enviados:** Filtros opcionais.
  - **Dados retornados:** Contratos e status.
  - **Validações esperadas:** Usuário autenticado com acesso Legalização/Financeiro/master.
  - **Riscos de segurança:** Exposição de contratos de clientes.
  - **Melhorias recomendadas:** Paginação e escopo por departamento/perfil.
- **Action observada:** `listTemplates`
  - **Método provável:** GET
  - **Finalidade:** Listar templates de cabeçalho/rodapé.
  - **Dados enviados:** Tipo opcional.
  - **Dados retornados:** Templates.
  - **Validações esperadas:** Usuário autorizado ou dado público mínimo.
  - **Riscos de segurança:** XSS por HTML de template.
  - **Melhorias recomendadas:** Sanitizar HTML e versionar templates.
- **Action observada:** `send`
  - **Método provável:** POST
  - **Finalidade:** Gerar/enviar link de assinatura.
  - **Dados enviados:** ID, PDF/HTML e dados de envio.
  - **Dados retornados:** Link/token/status.
  - **Validações esperadas:** CSRF, autorização e token aleatório.
  - **Riscos de segurança:** Link público reutilizável sem expiração.
  - **Melhorias recomendadas:** Token assinado, expiração e revogação.

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

### Endpoint: `/api/propostas.php`

- **Action observada:** `get`
  - **Método provável:** GET
  - **Finalidade:** Carregar proposta por ID.
  - **Dados enviados:** ID.
  - **Dados retornados:** Dados da proposta.
  - **Validações esperadas:** Autorização.
  - **Riscos de segurança:** IDOR.
  - **Melhorias recomendadas:** Política por recurso.
- **Action observada:** `list`
  - **Método provável:** GET
  - **Finalidade:** Listar propostas.
  - **Dados enviados:** Filtros.
  - **Dados retornados:** Propostas e status.
  - **Validações esperadas:** Usuário autenticado autorizado.
  - **Riscos de segurança:** Exposição de propostas comerciais.
  - **Melhorias recomendadas:** Escopo e paginação.
- **Action observada:** `save`
  - **Método provável:** POST
  - **Finalidade:** Salvar proposta interna.
  - **Dados enviados:** Campos comerciais e HTML.
  - **Dados retornados:** Proposta atualizada.
  - **Validações esperadas:** CSRF e validação.
  - **Riscos de segurança:** XSS ou alteração não auditada.
  - **Melhorias recomendadas:** Sanitização e versionamento.
- **Action observada:** `send`
  - **Método provável:** POST
  - **Finalidade:** Gerar link público da proposta.
  - **Dados enviados:** ID e payload da proposta.
  - **Dados retornados:** Token/link.
  - **Validações esperadas:** CSRF, autorização e token seguro.
  - **Riscos de segurança:** Link público sem expiração.
  - **Melhorias recomendadas:** Token assinado, expiração e revogação.

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

- Campos de proposta comercial, abertura e extras alimentam preview/documento.
- Salvar proposta e gerar link público são ações críticas.
- Valores monetários e quantidades devem ser normalizados.

### Regras que devem existir obrigatoriamente no backend

- Validar valores e tipo de proposta no servidor.
- Versionar proposta enviada.
- Token de cliente com expiração e escopo.
- Auditar aprovação/ajuste.
- Uploads devem ser validados por extensão, MIME, tamanho, assinatura, antivírus e armazenados fora da pasta pública.

---

## 7. Análise de segurança

### 7.1 Autenticação

`auth.js` está relacionado à página. A validação inicial ocorre no frontend chamando `/api/storage.php?action=auth_session_status`; endpoints PHP precisam manter a validação real de sessão.

### 7.2 Autorização

A autorização deve considerar perfil, departamento, propriedade do recurso e vínculo com cliente/processo. O frontend pode esconder botões, mas a decisão precisa estar no PHP.

### 7.3 Proteção contra XSS

Foram identificadas 35 ocorrência(s) de `innerHTML` nos scripts vinculados. A presença de HTML editável/templates exige sanitização server-side e DOMPurify no cliente. Evitar inserir dados de API com `innerHTML`; preferir `textContent` ou sanitização contextual.

### 7.4 Proteção contra CSRF

Para actions de mutação, o padrão do projeto usa `X-CSRF-Token` via `SamaAuth.buildHeaders` e `sama_require_csrf()` em APIs críticas. Validar cobertura em todas as actions desta página.

### 7.5 Proteção contra SQL Injection

Os endpoints PHP analisados usam `PDO`/`prepare` em diversas consultas. Ainda assim, qualquer consulta dinâmica, filtro, `ORDER BY`, nome de tabela/coluna ou action genérica deve usar allowlist e prepared statements. Ponto a validar em cada endpoint relacionado.

### 7.6 Segurança em uploads

Há upload/manipulação de arquivo. Exigir allowlist de extensão, MIME real, assinatura mágica, tamanho máximo, nome gerado, quarentena, ClamAV, storage privado e download autenticado/autorizado.

### 7.7 Dados sensíveis

Dados sensíveis envolvidos: dados comerciais, honorários, forma de pagamento, destinatário, empresa, templates e link público. Não expor esses dados no HTML inicial, em logs do navegador, em mensagens de erro, em query strings permanentes ou em `localStorage`.

### 7.8 Logs e auditoria

Ações críticas desta página devem gerar trilha de auditoria com usuário derivado da sessão, entidade, antes/depois quando aplicável, IP, user-agent e timestamp. Evitar registrar valores sensíveis em claro.

---

## 8. Checklist de segurança da página

- [x] Página protegida contra acesso não autenticado.
- [x] Permissões validadas no backend.
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

- Arquivos JavaScript extensos/acoplados à tela: `DEV/dev-data.js` (2582 linhas), `global.js` (2433 linhas), `Legalizacao/proposta.js` (1288 linhas).
- Uso de `innerHTML` em scripts relacionados (35 ocorrência(s)), exigindo revisão de XSS.
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
- Componente `SecureUploadField` com validação visual de extensão/tamanho e progresso, mantendo validação definitiva no backend.
- React + TypeScript com CKEditor encapsulado e DOMPurify obrigatório na renderização de HTML controlado.

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
- `legalizacaoService`
- `proposalService`
- `contractService`
- `signatureService`

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

A página `Legalizacao/proposta.html` atua no módulo **Legalização / Propostas** e manipula **dados comerciais, honorários, forma de pagamento, destinatário, empresa, templates e link público**. Os principais riscos são: upload/download de arquivos. A principal recomendação é reforçar validações server-side, autorização por recurso, auditoria e componentização gradual.

- **Nível de criticidade:** Alto
- **Prioridade de refatoração:** Alta
- **Observações finais:** manter o HTML como camada de apresentação; regras de permissão, validação, persistência e auditoria devem residir no backend. Quando houver incerteza, tratar como **ponto a validar no código/backend**.


---

## 15. Atualização de stack alvo — TypeScript/NestJS

Esta seção complementa a análise original da página com a decisão técnica posterior de migrar o backend PHP para **Node.js + TypeScript + NestJS** e evoluir o frontend para **React + TypeScript + Vite**.

### 15.1 Mapeamento da tela para a arquitetura alvo

- **Arquivo HTML atual:** `Legalizacao/proposta.html`
- **Rota React sugerida:** `/legalizacao/propostas/:id`
- **Componente React sugerido:** `ProposalPage.tsx`
- **Backend alvo:** `/api-v2`, implementado em NestJS
- **Banco principal:** MySQL 8 com Prisma ORM

### 15.2 Módulos NestJS relacionados

- `ProposalsModule`
- `LegalizationModule`
- `AuditModule`

### 15.3 Arquivos atuais que devem ser usados como referência de migração

- `Legalizacao/proposta.html`
- `api/legalizacao.php`

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
