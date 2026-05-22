# Assinatura de Contrato

## 1. Identificação da página

- **Arquivo HTML:** `Legalizacao/assinatura.html`
- **Módulo:** Legalização / Assinatura pública
- **Arquivos CSS relacionados:** `Legalizacao/assinatura.css`, `https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css`
- **Arquivos JavaScript relacionados:** `https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js`, `https://cdnjs.cloudflare.com/ajax/libs/dompurify/3.1.7/purify.min.js`, `Legalizacao/assinatura.js`
- **APIs/endpoints relacionados:** `/api/legalizacao.php actions: clientSubmit, flags, getByToken, renderPdf`
- **Perfil de usuário provável:** Cliente externo ou signatário com token de assinatura
- **Nível de criticidade:** Crítico
- **Prioridade de refatoração:** Urgente

---

## 2. Objetivo da página

Permitir que o cliente visualize, baixe e assine contrato por link exclusivo.

- **Problema resolvido:** centraliza o fluxo de legalização / assinatura pública para reduzir consulta manual, duplicação operacional e perda de rastreabilidade.
- **Quem utiliza:** Cliente externo ou signatário com token de assinatura.
- **Etapa do fluxo:** Legalização / Assinatura pública dentro do Portal Sama.
- **Dados/documentos manipulados:** token público, contrato, dados de assinatura, nome, CPF/CNPJ e imagem/traço de assinatura.
- **Observação:** Por ser uma página pública por token, todo dado retornado precisa ser mínimo, temporário e auditável.

Status em 2026-05-15 16:13: a rota React publica `/assinatura/:token` foi criada em `portal-sama-web/src/pages/contracts/PublicSignaturePage.tsx`, consumindo `GET/POST /api-v2/public/signatures/:token`. A tela consulta dados minimos por token, renderiza o contrato como texto derivado do HTML sem `innerHTML`, permite registrar nome/documento/assinatura e nao grava o token em `localStorage`/`sessionStorage`. Falta validar com API v2/MySQL/backfill reais, links reais/legados, PDF assinado real, HTTPS/homologacao e Playwright.

---

## 3. Funcionamento detalhado da interface

Cabeçalho identificado com área de usuário, departamento e ação de logout ou retorno, carregado em conjunto com `global.js`/`auth.js` quando presentes.

Conteúdo principal identificado: — Baixar PDF Assinar e enviar Preencha seus dados e assine no campo abaixo. Ao enviar, o status do contrato será atualizado. Nome completo CPF/CNPJ Assinatura (; — Baixar PDF; Assinar e enviar Preencha seus dados e assine no campo abaixo. Ao enviar, o status do contrato será atualizado. Nome completo CPF/CNPJ Assinatura (opcional) Lim.

Títulos/seções visíveis: `Assinatura de Contrato`, `Assinar e enviar`.

Campos relevantes: `sigNome` (text), `sigDoc` (text).

Ações/botões identificados: `Baixar PDF`, `Limpar`, `Enviar contrato assinado`.

Fluxo esperado do usuário:

1. Acessar a página pela navegação interna ou link público/legado, conforme o caso.
2. O JavaScript relacionado carrega sessão, dados iniciais, filtros e tabelas.
3. O usuário preenche campos, seleciona filtros ou aciona botões de criação/edição/envio/download.
4. As APIs PHP recebem a requisição, aplicam validações e retornam JSON ou arquivo.
5. A interface atualiza status, tabelas, mensagens de sucesso/erro ou redireciona conforme o resultado.

---

## 4. Fluxo de dados

- A página não usa `auth.js`; o acesso depende de token público na URL. O backend deve validar token, finalidade, expiração e escopo antes de retornar qualquer dado.

- Endpoints envolvidos: `/api/legalizacao.php`. As actions observadas são: `clientSubmit`, `flags`, `getByToken`, `renderPdf`.

- O token público é dado de autenticação contextual. Ele não deve ser gravado em logs de acesso sem mascaramento, nem reutilizado para finalidades diferentes.

- Cookies de sessão são tratados pelo backend em `api/auth.php` com `HttpOnly`, `SameSite=Lax` e `Secure` condicional a HTTPS/proxy. Ponto a validar em produção: HTTPS obrigatório, HSTS efetivo e domínio/path restritivos.

Campos/dados de entrada identificados no HTML: `sigNome` (text), `sigDoc` (text)

Quando alguma action for indireta ou montada dinamicamente no JavaScript, considerar: **ponto a validar no código/backend**.

---

## 5. APIs e integrações relacionadas

### Endpoint: `/api/legalizacao.php`

- **Action observada:** `clientSubmit`
  - **Método provável:** POST
  - **Finalidade:** Registrar assinatura do cliente.
  - **Dados enviados:** Token, nome, documento e assinatura.
  - **Dados retornados:** Status atualizado.
  - **Validações esperadas:** Validação do token e integridade da assinatura.
  - **Riscos de segurança:** Submissão forjada ou replay.
  - **Melhorias recomendadas:** Assinar payload, carimbo de tempo e evidência de IP/user-agent.
- **Action observada:** `flags`
  - **Método provável:** GET
  - **Finalidade:** Obter feature flags do serviço de documentos/renderização.
  - **Dados enviados:** Nenhum ou sessão dependendo da action.
  - **Dados retornados:** Flags e health do serviço.
  - **Validações esperadas:** Não expor segredos/URLs internas sensíveis.
  - **Riscos de segurança:** Fingerprinting de infraestrutura.
  - **Melhorias recomendadas:** Retornar apenas flags necessárias ao frontend.
- **Action observada:** `getByToken`
  - **Método provável:** GET
  - **Finalidade:** Carregar contrato público por token.
  - **Dados enviados:** Token público.
  - **Dados retornados:** Dados permitidos para assinatura.
  - **Validações esperadas:** Token válido e não expirado.
  - **Riscos de segurança:** Vazamento por token compartilhado ou sem expiração.
  - **Melhorias recomendadas:** Rotacionar token, expirar e limitar tentativas.
- **Action observada:** `renderPdf`
  - **Método provável:** POST
  - **Finalidade:** Renderizar PDF server-side a partir de HTML.
  - **Dados enviados:** HTML de cabeçalho/corpo/rodapé.
  - **Dados retornados:** PDF/resultado.
  - **Validações esperadas:** Validação de HTML e serviço habilitado.
  - **Riscos de segurança:** Injeção em renderizador e exfiltração por recursos externos.
  - **Melhorias recomendadas:** Bloquear URLs externas e usar sandbox.

---

## 6. Regras de negócio identificadas

### Regras identificadas no frontend

- Cliente visualiza contrato por token, preenche nome/documento e assina no campo indicado.
- Botões permitem baixar PDF, limpar assinatura e enviar contrato assinado.

### Regras que devem existir obrigatoriamente no backend

- Token válido, único, expirável e escopado ao contrato.
- Impedir assinatura múltipla não autorizada.
- Registrar IP, user-agent, data/hora e hash do documento assinado.
- Proteger contra replay.
- Links públicos devem possuir token forte, expiração, finalidade única, rate limiting e revogação.

---

## 7. Análise de segurança

### 7.1 Autenticação

A página opera por token público, não por sessão interna. A autenticação contextual deve ser feita exclusivamente no backend pelo token.

### 7.2 Autorização

A autorização é pelo escopo do token: o token deve permitir apenas aquele contrato, proposta ou checklist de documentos, nunca acesso amplo ao portal.

### 7.3 Proteção contra XSS

Foram identificadas 5 ocorrência(s) de `innerHTML` nos scripts vinculados. A presença de HTML editável/templates exige sanitização server-side e DOMPurify no cliente. Evitar inserir dados de API com `innerHTML`; preferir `textContent` ou sanitização contextual.

### 7.4 Proteção contra CSRF

Como a página pública não usa sessão interna, CSRF tradicional é menos central; o risco principal é replay/abuso do token. Ainda assim, actions POST devem validar token, origem quando aplicável, rate limit e idempotência.

### 7.5 Proteção contra SQL Injection

Os endpoints PHP analisados usam `PDO`/`prepare` em diversas consultas. Ainda assim, qualquer consulta dinâmica, filtro, `ORDER BY`, nome de tabela/coluna ou action genérica deve usar allowlist e prepared statements. Ponto a validar em cada endpoint relacionado.

### 7.6 Segurança em uploads

Não há upload direto identificado nesta página.

### 7.7 Dados sensíveis

Dados sensíveis envolvidos: token público, contrato, dados de assinatura, nome, CPF/CNPJ e imagem/traço de assinatura. Não expor esses dados no HTML inicial, em logs do navegador, em mensagens de erro, em query strings permanentes ou em `localStorage`.

### 7.8 Logs e auditoria

Ações críticas desta página devem gerar trilha de auditoria com usuário derivado da sessão, entidade, antes/depois quando aplicável, IP, user-agent e timestamp. Evitar registrar valores sensíveis em claro.

---

## 8. Checklist de segurança da página

- [ ] Página protegida contra acesso não autenticado.
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
- [x] Uploads validados por extensão, MIME type e conteúdo.
- [x] Arquivos armazenados fora da pasta pública.
- [x] Downloads protegidos por autenticação e autorização.
- [ ] Ações críticas registradas em auditoria.
- [ ] Erros tratados sem expor detalhes internos.
- [ ] Logs não expõem dados sensíveis.

---

## 9. Problemas técnicos encontrados

- Uso de `innerHTML` em scripts relacionados (5 ocorrência(s)), exigindo revisão de XSS.
- Fluxo por link público depende fortemente de token; precisa expiração, escopo, rate limiting e mascaramento de logs.

---

## 10. Melhorias recomendadas

### 10.1 Melhorias rápidas

- Centralizar chamadas `fetch` em um client com tratamento de erro, CSRF e timeout.
- Padronizar mensagens de erro sem expor detalhes internos.
- Garantir que botões sensíveis sejam ocultados e bloqueados por permissão no backend.
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
- React + TypeScript com CKEditor encapsulado e DOMPurify obrigatório na renderização de HTML controlado.

### Backend

- NestJS com tokens públicos opacos, expiração de token, rate limiting e guards/policies específicas por finalidade do link.

### Segurança

- Sessão server-side com cookies `HttpOnly`, `Secure` e `SameSite`, evitando armazenar tokens sensíveis em `localStorage`.
- RBAC por perfil e departamento, validado no backend em todas as actions.
- Tokens públicos assinados, com expiração, rotação, limitação de tentativas e escopo mínimo por recurso.

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
- `PublicTokenLayout`
- `TokenErrorState`
- `ClientActionPanel`

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

### Testes de integração

- Endpoint deve rejeitar requisição sem autenticação quando a página for interna.
- Endpoint deve rejeitar usuário sem perfil/departamento permitido.
- Erro da API deve ser tratado na interface sem expor stack trace.
- Token expirado, inválido, revogado ou de outro recurso deve retornar erro genérico.

### Testes E2E

- Usuário autorizado acessa a página e executa o fluxo principal.
- Usuário sem permissão não visualiza nem executa ações críticas.
- Cliente acessa link público válido, conclui ação e não consegue repetir quando a regra impedir replay.

### Testes de segurança

- Tentar XSS em campos livres e dados retornados pela API.
- Tentar IDOR alterando IDs, client_id, doc_id ou token.
- Verificar CSRF em actions POST/PUT/PATCH/DELETE.
- Token público não deve aparecer em logs de aplicação sem mascaramento.

---

## 14. Conclusão da página

A página `Legalizacao/assinatura.html` atua no módulo **Legalização / Assinatura pública** e manipula **token público, contrato, dados de assinatura, nome, CPF/CNPJ e imagem/traço de assinatura**. Os principais riscos são: token público. A principal recomendação é reforçar validações server-side, autorização por recurso, auditoria e componentização gradual.

- **Nível de criticidade:** Crítico
- **Prioridade de refatoração:** Urgente
- **Observações finais:** manter o HTML como camada de apresentação; regras de permissão, validação, persistência e auditoria devem residir no backend. Quando houver incerteza, tratar como **ponto a validar no código/backend**.


---

## 15. Atualização de stack alvo — TypeScript/NestJS

Esta seção complementa a análise original da página com a decisão técnica posterior de migrar o backend PHP para **Node.js + TypeScript + NestJS** e evoluir o frontend para **React + TypeScript + Vite**.

### 15.1 Mapeamento da tela para a arquitetura alvo

- **Arquivo HTML atual:** `Legalizacao/assinatura.html`
- **Rota React sugerida:** `/assinatura/:token`
- **Componente React sugerido:** `PublicSignaturePage.tsx`
- **Backend alvo:** `/api-v2`, implementado em NestJS
- **Banco principal:** MySQL 8 com Prisma ORM

### 15.2 Módulos NestJS relacionados

- `SignaturesModule`
- `PublicLinksModule`
- `AuditModule`

### 15.3 Arquivos atuais que devem ser usados como referência de migração

- `Legalizacao/assinatura.html`
- `api/legalizacao.php`
- `api/public_token_lib.php`

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
