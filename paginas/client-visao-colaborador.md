# [PARCIAL] Visão do Colaborador

## 1. Identificação da página

- **Arquivo HTML:** `Client/visao-colaborador.html`
- **Módulo:** Clientes / Colaboradores
- **Arquivos CSS relacionados:** `DEV/dev.css`, `Client/clientes.css`, `global.css`, `https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css`
- **Arquivos JavaScript relacionados:** `auth.js`, `DEV/dev-data.js`, `global.js`, `Client/visao-colaborador.js`
- **APIs/endpoints relacionados:** `/api/notifications.php actions: ack, alert_ack, clear_unread, list`, `/api/storage.php actions: audit_push, auth_logout, auth_session_status, get, set, user_presence_ping`
- **Perfil de usuário provável:** Master, gestor ou colaborador autorizado
- **Nível de criticidade:** Médio
- **Prioridade de refatoração:** Média

---

## 2. Objetivo da página

Mostrar informações resumidas de um colaborador e a carteira de clientes vinculada a ele.

- **Problema resolvido:** centraliza o fluxo de clientes / colaboradores para reduzir consulta manual, duplicação operacional e perda de rastreabilidade.
- **Quem utiliza:** Master, gestor ou colaborador autorizado.
- **Etapa do fluxo:** Clientes / Colaboradores dentro do Portal Sama.
- **Dados/documentos manipulados:** dados de colaborador, departamento, cargo e relação colaborador-cliente.
- **Observação:** ponto a validar no código/backend para regras não explícitas no HTML.

---

## 3. Funcionamento detalhado da interface

Cabeçalho identificado com área de usuário, departamento e ação de logout ou retorno, carregado em conjunto com `global.js`/`auth.js` quando presentes.

Navegação lateral/superior com atalhos para Início, Departamento, Clientes e Área Dev, reaproveitando a identidade visual global do portal.

Conteúdo principal identificado: Visão Geral Colaborador Nome do colaborador ▾ Departamento do usuário Logout ← Voltar para colaboradores Colaborador • ATIVO Geral Clientes Informações do colab; ← Voltar para colaboradores Colaborador • ATIVO Geral Clientes Informações do colaborador Nome Departamento principal Cargo Status Ramal E-mail corporativo Resu; Informações do colaborador Nome Departamento principal Cargo Status Ramal E-mail corporativo Resumo de clientes Total de clientes sob responsabilidade: Clique e; Clientes do colaborador Lista de clientes nos quais este colaborador é responsável. Cliente CNPJ Departamento(s) Status Nenhum cliente vinculado a este colabora.

Títulos/seções visíveis: `Visão Geral`, `Colaborador`, `Informações do colaborador`, `Resumo de clientes`.

Tabelas identificadas: `tabelaClientesColabView` (`Cliente`, `CNPJ`, `Departamento(s)`, `Status`).

Ações/botões identificados: `Início`, `Departamento ▾`, `Modelo padrão Departamento`, `Clientes`, `Área Dev`, `Nome do colaborador ▾`, `Departamento do usuário`, `Logout`, `← Voltar para colaboradores`, `Geral`.

Fluxo esperado do usuário:

1. Acessar a página pela navegação interna ou link público/legado, conforme o caso.
2. O JavaScript relacionado carrega sessão, dados iniciais, filtros e tabelas.
3. O usuário preenche campos, seleciona filtros ou aciona botões de criação/edição/envio/download.
4. As APIs PHP recebem a requisição, aplicam validações e retornam JSON ou arquivo.
5. A interface atualiza status, tabelas, mensagens de sucesso/erro ou redireciona conforme o resultado.

---

## 4. Fluxo de dados

- Ao abrir a página, `auth.js` consulta `/api/storage.php?action=auth_session_status`; se a sessão não estiver autenticada, redireciona para `/index.html`. O usuário público fica em `sessionStorage` por meio de shim compatível com a chave legada `usuarioLogado`.

- Endpoints envolvidos: `/api/notifications.php`, `/api/storage.php`. As actions observadas são: `ack`, `alert_ack`, `clear_unread`, `list`, `audit_push`, `auth_logout`, `auth_session_status`, `get`, `set`, `user_presence_ping`.

- Uso de armazenamento no navegador identificado nos scripts relacionados: `localStorage` em 40 ocorrência(s) e `sessionStorage` em 8 ocorrência(s). Tokens/segredos não devem ser armazenados nesses mecanismos; manter apenas estado não sensível.

- Cookies de sessão são tratados pelo backend em `api/auth.php` com `HttpOnly`, `SameSite=Lax` e `Secure` condicional a HTTPS/proxy. Ponto a validar em produção: HTTPS obrigatório, HSTS efetivo e domínio/path restritivos.

Campos/dados de entrada identificados no HTML: Nenhum campo HTML identificado diretamente.

Quando alguma action for indireta ou montada dinamicamente no JavaScript, considerar: **ponto a validar no código/backend**.

---

## 5. APIs e integrações relacionadas

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

- Exibe dados do colaborador, clientes vinculados e, em telas administrativas, permissões/status.
- Transferência de clientes exige seleção de destino e itens.

### Regras que devem existir obrigatoriamente no backend

- Validar role do operador.
- Derivar autor da sessão.
- Auditar alteração de status, senha, permissões e carteira.
- Impedir escalonamento de privilégios.

---

## 7. Análise de segurança

### 7.1 Autenticação

`auth.js` está relacionado à página. A validação inicial ocorre no frontend chamando `/api/storage.php?action=auth_session_status`; endpoints PHP precisam manter a validação real de sessão.

### 7.2 Autorização

A autorização deve considerar perfil, departamento, propriedade do recurso e vínculo com cliente/processo. O frontend pode esconder botões, mas a decisão precisa estar no PHP.

### 7.3 Proteção contra XSS

Foram identificadas 25 ocorrência(s) de `innerHTML` nos scripts vinculados. Evitar inserir dados de API com `innerHTML`; preferir `textContent` ou sanitização contextual.

### 7.4 Proteção contra CSRF

Para actions de mutação, o padrão do projeto usa `X-CSRF-Token` via `SamaAuth.buildHeaders` e `sama_require_csrf()` em APIs críticas. Validar cobertura em todas as actions desta página.

### 7.5 Proteção contra SQL Injection

Os endpoints PHP analisados usam `PDO`/`prepare` em diversas consultas. Ainda assim, qualquer consulta dinâmica, filtro, `ORDER BY`, nome de tabela/coluna ou action genérica deve usar allowlist e prepared statements. Ponto a validar em cada endpoint relacionado.

### 7.6 Segurança em uploads

Não há upload direto identificado nesta página.

### 7.7 Dados sensíveis

Dados sensíveis envolvidos: dados de colaborador, departamento, cargo e relação colaborador-cliente. Não expor esses dados no HTML inicial, em logs do navegador, em mensagens de erro, em query strings permanentes ou em `localStorage`.

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

- Arquivos JavaScript extensos/acoplados à tela: `DEV/dev-data.js` (2582 linhas), `global.js` (2433 linhas).
- Uso de `innerHTML` em scripts relacionados (25 ocorrência(s)), exigindo revisão de XSS.
- Uso de `localStorage` em scripts relacionados (40 ocorrência(s)); não armazenar tokens ou dados sensíveis.
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

- TanStack Query para cache/estado de servidor e um componente `DataTable` reutilizável com paginação e filtros server-side.

### Backend

- API REST por recurso (`clients`, `client-documents`, `certificates`) com policies por cliente/departamento.

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
- `DataTable`
- `StatusBadge`
- `FilterBar`

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

A página `Client/visao-colaborador.html` atua no módulo **Clientes / Colaboradores** e manipula **dados de colaborador, departamento, cargo e relação colaborador-cliente**. Os principais riscos são: autorização por perfil/departamento e exposição de dados. A principal recomendação é reforçar validações server-side, autorização por recurso, auditoria e componentização gradual.

- **Nível de criticidade:** Médio
- **Prioridade de refatoração:** Média
- **Observações finais:** manter o HTML como camada de apresentação; regras de permissão, validação, persistência e auditoria devem residir no backend. Quando houver incerteza, tratar como **ponto a validar no código/backend**.


---

## 15. Atualização de stack alvo — TypeScript/NestJS

Esta seção complementa a análise original da página com a decisão técnica posterior de migrar o backend PHP para **Node.js + TypeScript + NestJS** e evoluir o frontend para **React + TypeScript + Vite**.

### 15.1 Mapeamento da tela para a arquitetura alvo

- **Arquivo HTML atual:** `Client/visao-colaborador.html`
- **Rota React sugerida:** `/clientes/colaboradores/:id`
- **Componente React sugerido:** `CollaboratorViewPage.tsx`
- **Backend alvo:** `/api-v2`, implementado em NestJS
- **Banco principal:** MySQL 8 com Prisma ORM

### 15.2 Módulos NestJS relacionados

- `CollaboratorsModule`
- `ClientsModule`

### 15.3 Arquivos atuais que devem ser usados como referência de migração

- `Client/visao-colaborador.html`
- `global.js`

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

---

## 16. Atualizacao de backend - 2026-05-13 14:43

- `CollaboratorsModule` foi criado em `portal-sama-api/src/modules/collaborators/`.
- A tela passa a ter contrato alvo inicial em `GET /api-v2/collaborators/:id`, com resposta contendo aliases legados (`usuario`, `nome`, `departamento`, `cargo`, `telefone`, `ramal`).
- A API exige JWT e `collaborators.read`; acesso fora do proprio departamento e bloqueado para perfis nao privilegiados.
- Ainda faltam vinculos reais com clientes/carteira, backfill legado e migracao React.

## 17. Atualizacao de frontend React - 2026-05-18 14:39

- A rota `/clientes/colaboradores/:id` passou a renderizar `CollaboratorViewPage.tsx`.
- A tela consome `GET /api-v2/collaborators/:id` por `collaborators.service.ts`, usando TanStack Query e `PermissionGate` com `collaborators.read`.
- A interface exibe identidade funcional, contato, departamento/cargo, status, datas, perfis, permissoes efetivas e metadados em JSON textual sem `innerHTML`.
- A listagem `/colaboradores/visao-geral` passou a linkar o nome do colaborador para esta visao individual.
- Permanecem pendentes os vinculos reais de cliente/carteira, backfill de `sama_colaboradores`, validacao com API v2/MySQL/usuarios reais e Playwright.
