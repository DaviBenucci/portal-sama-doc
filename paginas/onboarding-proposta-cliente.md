# Aprovação de Proposta pelo Cliente

## 1. Identificação da página

- **Arquivo HTML:** `Onboarding/proposta-cliente.html`
- **Módulo:** Onboarding / Proposta pública
- **Arquivos CSS relacionados:** `global.css`, `Onboarding/public-links.css`
- **Arquivos JavaScript relacionados:** `Onboarding/proposta-cliente.js`
- **APIs/endpoints relacionados:** `/api/onboarding.php actions: client_action, client_get`, `/api/propostas.php actions: client_action, client_get`
- **Perfil de usuário provável:** Cliente externo com token de proposta
- **Nível de criticidade:** Alto
- **Prioridade de refatoração:** Alta

---

## 2. Objetivo da página

Apresentar proposta ao cliente e registrar aprovação ou solicitação de ajuste.

- **Problema resolvido:** centraliza o fluxo de onboarding / proposta pública para reduzir consulta manual, duplicação operacional e perda de rastreabilidade.
- **Quem utiliza:** Cliente externo com token de proposta.
- **Etapa do fluxo:** Onboarding / Proposta pública dentro do Portal Sama.
- **Dados/documentos manipulados:** token público, proposta, mensagem de ajuste e decisão do cliente.
- **Observação:** Por ser uma página pública por token, todo dado retornado precisa ser mínimo, temporário e auditável.

Status em 2026-05-15 15:06: a rota React publica `/onboarding/publico/proposta/:token` foi criada em `portal-sama-web/src/pages/proposals/PublicProposalPage.tsx`, consumindo `GET/POST /api-v2/public/proposals/:token`. A tela consulta dados minimos por token, renderiza campos como texto, permite aprovar ou solicitar ajuste e nao grava o token em `localStorage`/`sessionStorage`. Falta validar com API v2/MySQL/backfill reais, tokens reais/legados, HTTPS/homologacao e Playwright.

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

- A página não usa `auth.js`; o acesso depende de token público na URL. O backend deve validar token, finalidade, expiração e escopo antes de retornar qualquer dado.

- Endpoints envolvidos: `/api/onboarding.php`, `/api/propostas.php`. As actions observadas são: `client_action`, `client_get`, `client_action`, `client_get`.

- Há manipulação de arquivos. Os dados saem como `FormData`, `multipart/form-data` ou dataURL, conforme a tela. O backend deve validar extensão, MIME real, tamanho, assinatura do arquivo, antivírus, armazenamento privado e permissão de download.

- O token público é dado de autenticação contextual. Ele não deve ser gravado em logs de acesso sem mascaramento, nem reutilizado para finalidades diferentes.

- Cookies de sessão são tratados pelo backend em `api/auth.php` com `HttpOnly`, `SameSite=Lax` e `Secure` condicional a HTTPS/proxy. Ponto a validar em produção: HTTPS obrigatório, HSTS efetivo e domínio/path restritivos.

Campos/dados de entrada identificados no HTML: `txt_ajuste` (textarea)

Quando alguma action for indireta ou montada dinamicamente no JavaScript, considerar: **ponto a validar no código/backend**.

---

## 5. APIs e integrações relacionadas

### Endpoint: `/api/onboarding.php`

- **Action observada:** `client_action`
  - **Método provável:** POST
  - **Finalidade:** Registrar decisão do cliente.
  - **Dados enviados:** Token, ação e mensagem.
  - **Dados retornados:** Status atualizado.
  - **Validações esperadas:** Token válido.
  - **Riscos de segurança:** Replay.
  - **Melhorias recomendadas:** Idempotência.
- **Action observada:** `client_get`
  - **Método provável:** GET
  - **Finalidade:** Carregar dados públicos do processo/proposta.
  - **Dados enviados:** Token.
  - **Dados retornados:** Dados permitidos.
  - **Validações esperadas:** Token válido.
  - **Riscos de segurança:** Vazamento de dados do processo.
  - **Melhorias recomendadas:** Escopo mínimo por tipo de token.

### Endpoint: `/api/propostas.php`

- **Action observada:** `client_action`
  - **Método provável:** POST
  - **Finalidade:** Registrar aprovação ou pedido de ajuste.
  - **Dados enviados:** Token, `action`, mensagem.
  - **Dados retornados:** Status atualizado.
  - **Validações esperadas:** Token válido e validação de ação.
  - **Riscos de segurança:** Replay ou alteração forjada.
  - **Melhorias recomendadas:** Idempotência e auditoria.
- **Action observada:** `client_get`
  - **Método provável:** GET
  - **Finalidade:** Carregar proposta pública por token.
  - **Dados enviados:** Token.
  - **Dados retornados:** Dados permitidos ao cliente.
  - **Validações esperadas:** Token válido.
  - **Riscos de segurança:** Acesso indevido por token vazado.
  - **Melhorias recomendadas:** Expiração e rate limiting.

---

## 6. Regras de negócio identificadas

### Regras identificadas no frontend

- Cliente aprova a proposta ou solicita ajuste com mensagem.
- Conteúdo da proposta é renderizado dinamicamente.

### Regras que devem existir obrigatoriamente no backend

- Token válido, ação permitida (`approve`/`adjust`) e idempotência.
- Mensagem de ajuste sanitizada.
- Atualização de status e notificação interna.
- Uploads devem ser validados por extensão, MIME, tamanho, assinatura, antivírus e armazenados fora da pasta pública.
- Links públicos devem possuir token forte, expiração, finalidade única, rate limiting e revogação.

---

## 7. Análise de segurança

### 7.1 Autenticação

A página opera por token público, não por sessão interna. A autenticação contextual deve ser feita exclusivamente no backend pelo token.

### 7.2 Autorização

A autorização é pelo escopo do token: o token deve permitir apenas aquele contrato, proposta ou checklist de documentos, nunca acesso amplo ao portal.

### 7.3 Proteção contra XSS

Foram identificadas 4 ocorrência(s) de `innerHTML` nos scripts vinculados. A presença de HTML editável/templates exige sanitização server-side e DOMPurify no cliente. Evitar inserir dados de API com `innerHTML`; preferir `textContent` ou sanitização contextual.

### 7.4 Proteção contra CSRF

Como a página pública não usa sessão interna, CSRF tradicional é menos central; o risco principal é replay/abuso do token. Ainda assim, actions POST devem validar token, origem quando aplicável, rate limit e idempotência.

### 7.5 Proteção contra SQL Injection

Os endpoints PHP analisados usam `PDO`/`prepare` em diversas consultas. Ainda assim, qualquer consulta dinâmica, filtro, `ORDER BY`, nome de tabela/coluna ou action genérica deve usar allowlist e prepared statements. Ponto a validar em cada endpoint relacionado.

### 7.6 Segurança em uploads

Há upload/manipulação de arquivo. Exigir allowlist de extensão, MIME real, assinatura mágica, tamanho máximo, nome gerado, quarentena, ClamAV, storage privado e download autenticado/autorizado.

### 7.7 Dados sensíveis

Dados sensíveis envolvidos: token público, proposta, mensagem de ajuste e decisão do cliente. Não expor esses dados no HTML inicial, em logs do navegador, em mensagens de erro, em query strings permanentes ou em `localStorage`.

### 7.8 Logs e auditoria

Ações críticas desta página devem gerar trilha de auditoria com usuário derivado da sessão, entidade, antes/depois quando aplicável, IP, user-agent e timestamp. Evitar registrar valores sensíveis em claro.

---

## 8. Checklist de segurança da página

- [ ] Página protegida contra acesso não autenticado.
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

- Uso de `innerHTML` em scripts relacionados (4 ocorrência(s)), exigindo revisão de XSS.
- Upload/arquivo exige maior segregação entre UI, validação, armazenamento, antivírus e autorização de download.
- Fluxo por link público depende fortemente de token; precisa expiração, escopo, rate limiting e mascaramento de logs.

---

## 10. Melhorias recomendadas

### 10.1 Melhorias rápidas

- Centralizar chamadas `fetch` em um client com tratamento de erro, CSRF e timeout.
- Padronizar mensagens de erro sem expor detalhes internos.
- Garantir que botões sensíveis sejam ocultados e bloqueados por permissão no backend.
- Revisar uploads com allowlist, MIME real, limite de tamanho e quarentena antes do storage final.
- Adicionar expiração e rate limiting aos tokens públicos e mascará-los em logs.

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

### Backend

- NestJS com tokens públicos opacos, expiração de token, rate limiting e guards/policies específicas por finalidade do link.
- NestJS com storage privado, DTOs/Pipes de validação, validação MIME/assinatura de arquivo, ClamAV e endpoints protegidos ou links temporários para download.

### Segurança

- Sessão server-side com cookies `HttpOnly`, `Secure` e `SameSite`, evitando armazenar tokens sensíveis em `localStorage`.
- RBAC por perfil e departamento, validado no backend em todas as actions.
- Tokens públicos assinados, com expiração, rotação, limitação de tentativas e escopo mínimo por recurso.
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
- `PublicTokenLayout`
- `TokenErrorState`
- `ClientActionPanel`

Serviços sugeridos:
- `authService`
- `auditService`
- `notificationService`
- `onboardingService`
- `publicLinkService`
- `documentService`

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
- Token expirado, inválido, revogado ou de outro recurso deve retornar erro genérico.

### Testes E2E

- Usuário autorizado acessa a página e executa o fluxo principal.
- Usuário sem permissão não visualiza nem executa ações críticas.
- Cliente acessa link público válido, conclui ação e não consegue repetir quando a regra impedir replay.

### Testes de segurança

- Tentar XSS em campos livres e dados retornados pela API.
- Tentar IDOR alterando IDs, client_id, doc_id ou token.
- Verificar CSRF em actions POST/PUT/PATCH/DELETE.
- Enviar arquivo com extensão dupla, MIME falso, payload EICAR e nome malicioso.
- Token público não deve aparecer em logs de aplicação sem mascaramento.

---

## 14. Conclusão da página

A página `Onboarding/proposta-cliente.html` atua no módulo **Onboarding / Proposta pública** e manipula **token público, proposta, mensagem de ajuste e decisão do cliente**. Os principais riscos são: upload/download de arquivos, token público. A principal recomendação é reforçar validações server-side, autorização por recurso, auditoria e componentização gradual.

- **Nível de criticidade:** Alto
- **Prioridade de refatoração:** Alta
- **Observações finais:** manter o HTML como camada de apresentação; regras de permissão, validação, persistência e auditoria devem residir no backend. Quando houver incerteza, tratar como **ponto a validar no código/backend**.


---

## 15. Atualização de stack alvo — TypeScript/NestJS

Esta seção complementa a análise original da página com a decisão técnica posterior de migrar o backend PHP para **Node.js + TypeScript + NestJS** e evoluir o frontend para **React + TypeScript + Vite**.

### 15.1 Mapeamento da tela para a arquitetura alvo

- **Arquivo HTML atual:** `Onboarding/proposta-cliente.html`
- **Rota React sugerida:** `/onboarding/publico/proposta/:token`
- **Componente React sugerido:** `PublicProposalPage.tsx`
- **Backend alvo:** `/api-v2`, implementado em NestJS
- **Banco principal:** MySQL 8 com Prisma ORM

### 15.2 Módulos NestJS relacionados

- `OnboardingModule`
- `ProposalsModule`
- `PublicLinksModule`

### 15.3 Arquivos atuais que devem ser usados como referência de migração

- `Onboarding/proposta-cliente.html`
- `api/onboarding.php`

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
