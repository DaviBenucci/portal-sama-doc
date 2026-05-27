# Integra-AI Contábil

## 1. Identificação da página

- **Arquivo HTML:** `Contabil/integra-ai.html`
- **Módulo:** Contábil / Integra-AI
- **Arquivos CSS relacionados:** `global.css`, `Contabil/integra-ai.css`, `https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css`
- **Arquivos JavaScript relacionados:** `auth.js`, `global.js`, `Contabil/integra-ai.js`
- **APIs/endpoints relacionados:** `/api/integra_ai.php actions: create_job, download_txt, flags, generate_txt, list_step4_rules, load_job, parse_bank_statement, save_step2, save_step3, save_step4_rule, search_companies, search_plan_accounts`, `/api/notifications.php actions: ack, alert_ack, clear_unread, list`, `/api/storage.php actions: audit_push, auth_logout, auth_session_status, get, set, user_presence_ping`
- **Perfil de usuário provável:** Equipe Contábil, master ou usuário autorizado
- **Nível de criticidade:** Crítico
- **Prioridade de refatoração:** Urgente

### Atualizacao de migracao 2026-05-21 10:33 -03:00

- **Status:** Implementado parcialmente em React/API v2 para leitura.
- **Frontend novo:** `portal-sama-web/src/pages/accounting/IntegraAiPage.tsx`, rota `/contabil/integra-ai`, com workspace, filtros, jobs recentes, detalhe de job, resumo e preview seguro.
- **Backend novo:** `portal-sama-api/src/modules/accounting/`, com `GET /api-v2/accounting/integra-ai/workspaces` e `GET /api-v2/accounting/integra-ai/jobs/:id`.
- **Seguranca aplicada:** JWT, RBAC `accounting.integra_ai.read`, escopo contabil, SQL parametrizado, auditoria no detalhe de job, sem retorno de `source_path`, `txt_path`, payload bruto do parser ou conteudo TXT.
- **Pendente no legado em 2026-05-21:** upload/parser de extrato, criacao de job, salvamento de etapas, regras contabeis, geracao e download TXT Dominio. Ver atualizacao de 2026-05-27: parte dessas mutacoes ja existe na API v2, mas ainda falta validacao real antes do corte.

### Atualizacao de migracao 2026-05-27 10:03 -03:00

- **Status:** Implementado parcialmente em React/API v2 para importacao PDF e OFX opt-in.
- **Backend novo:** `POST /api-v2/accounting/integra-ai/import` valida PDF/OFX, armazena em storage privado, chama parser Python por extensao (`pdf`/`ofx`) e grava `source_type` dinamico.
- **Feature flag:** OFX fica desligado por padrao em `SAMA_INTEGRA_AI_OFX_IMPORT_ENABLED=false`; a API so expoe `ofx_import=true` quando a permissao de importacao existe e a flag esta ativa.
- **Frontend novo:** `IntegraAiPage.tsx` mostra `PDF/OFX` e aceita `.ofx` somente quando a capability `ofx_import` vem da API; ambientes antigos continuam com PDF.
- **Sem migration:** a implementacao reutiliza as tabelas operacionais existentes e o campo `source_type`.
- **Pendente real:** validar OFX/PDF com arquivos reais, usuario contabil, parser Python no container, MySQL/storage reais, ClamAV strict e homologacao HTTPS.

### Atualizacao de migracao 2026-05-27 10:40 -03:00

- **Banco Inter PDF:** implementado layout `inter_pdf` no parser Python.
- **Formato reconhecido:** datas longas em portugues (`1 de Dezembro de 2025`), transacoes abaixo da data sem repeticao de data, descricoes quebradas em multiplas linhas e valores com `R$`/`-R$`.
- **Saldo do dia:** classificado como `row_kind=balance_snapshot` e `ignore_reason=daily_balance`.
- **Regra de exibicao:** saldo diario nao deve aparecer para o usuario final, nao deve virar regra contabil e nao deve entrar no TXT Dominio. Ele fica apenas no payload tecnico para diagnostico/reconciliacao.
- **Uso recomendado do saldo:** usar futuramente para conferir se o ultimo saldo acumulado do dia bate com o saldo final informado pelo banco; em caso de divergencia, gerar alerta tecnico, nao lancamento.
- **Validacao local:** PDF real do Banco Inter retornou 31 transacoes e 15 saldos tecnicos; unit/lint/build da API passaram.

---

## 2. Objetivo da página

Processar extratos bancários e gerar arquivo TXT para Domínio com etapas de confirmação bancária, plano de contas e regras contábeis.

- **Problema resolvido:** centraliza o fluxo de contábil / integra-ai para reduzir consulta manual, duplicação operacional e perda de rastreabilidade.
- **Quem utiliza:** Equipe Contábil, master ou usuário autorizado.
- **Etapa do fluxo:** Contábil / Integra-AI dentro do Portal Sama.
- **Dados/documentos manipulados:** empresas, CNPJ, extratos, contas bancárias, plano de contas, regras contábeis, TXT Domínio e possíveis dados financeiros.
- **Observação:** ponto a validar no código/backend para regras não explícitas no HTML.

---

## 3. Funcionamento detalhado da interface

Cabeçalho identificado com área de usuário, departamento e ação de logout ou retorno, carregado em conjunto com `global.js`/`auth.js` quando presentes.

Navegação lateral/superior com atalhos para Início, Departamento, Clientes e Área Dev, reaproveitando a identidade visual global do portal.

Conteúdo principal identificado: Integra-AI Extrato bancario para TXT Dominio (fluxo simplificado em 4 etapas) Usuario ▾ Departamento do usuario Logout Processamento atual: Nenhum processamento; Processamento atual: Nenhum processamento carregado Retomar: Carregar 1. Upload 2. Confirmacao bancaria 3. Plano e regras contabeis 4. Gerar TXT Dominio Etapa 1; Etapa 1 - Empresa e Upload Empresa Selecione a empresa Digite pelo menos 2 caracteres para buscar empresas. Nenhuma empresa selecionada. Extrato Arquivo (PDF, X; Etapa 2 - Confirmacao bancaria Banco (key) Banco (nome) Agencia Conta Salvar etapa 2 Extrato computado Revise o mapeamento do extrato tabular e todas as linhas; Etapa 3 - Plano, modo e contas fixas Plano de contas Padrao Imunes/Isentas Case mode normal upper lower Conta banco (fixa) Conta clientes Conta fornecedor Desco.

Títulos/seções visíveis: `Integra-AI`, `Etapa 1 - Empresa e Upload`, `Etapa 2 - Confirmacao bancaria`, `Etapa 3 - Plano, modo e contas fixas`, `Regras contabeis`, `Etapa 4 - Gerar e baixar TXT Dominio`, `Empresa`, `Extrato`, `Extrato computado`, `Resumo`, `Acoes`, `Preview exato do TXT final`.

Campos relevantes: `jobIdInput` (number), `companySearch` (text), `statementFile` (file), `step2BankKey` (text), `step2BankName` (text), `step2Agency` (text), `step2Account` (text), `step3PlanKey` (select), `step3CaseMode` (select), `step3BankAccount` (text), `step3ClientAccount` (text), `step3SupplierAccount` (text), `step3DiscountPayAccount` (text), `step3DiscountReceiveAccount` (text), `step3InterestPayAccount` (text), `step3InterestReceiveAccount` (text), `step3DefaultHistoryCode` (text), `step3ThirdPartyAccount` (text) e mais 3 campo(s).

Ações/botões identificados: `Inicio`, `Departamento ▾`, `Modelo padrao Departamento`, `Clientes`, `Area Dev`, `Usuario ▾`, `Departamento do usuario`, `Logout`, `Carregar`, `1. Upload`, `2. Confirmacao bancaria`, `3. Plano e regras contabeis`, `4. Gerar TXT Dominio`, `Iniciar processamento`, `... mais 6 item(ns)`.

Fluxo esperado do usuário:

1. Acessar a página pela navegação interna ou link público/legado, conforme o caso.
2. O JavaScript relacionado carrega sessão, dados iniciais, filtros e tabelas.
3. O usuário preenche campos, seleciona filtros ou aciona botões de criação/edição/envio/download.
4. As APIs PHP recebem a requisição, aplicam validações e retornam JSON ou arquivo.
5. A interface atualiza status, tabelas, mensagens de sucesso/erro ou redireciona conforme o resultado.

---

## 4. Fluxo de dados

- Ao abrir a página, `auth.js` consulta `/api/storage.php?action=auth_session_status`; se a sessão não estiver autenticada, redireciona para `/index.html`. O usuário público fica em `sessionStorage` por meio de shim compatível com a chave legada `usuarioLogado`.

- Endpoints envolvidos: `/api/integra_ai.php`, `/api/notifications.php`, `/api/storage.php`. As actions observadas são: `ack`, `alert_ack`, `clear_unread`, `list`, `audit_push`, `auth_logout`, `auth_session_status`, `get`, `set`, `user_presence_ping`, `create_job`, `download_txt`, `flags`, `generate_txt`, `list_step4_rules`, `load_job`, `parse_bank_statement`, `save_step2`, `save_step3`, `save_step4_rule`, `search_companies`, `search_plan_accounts`.

- Há manipulação de arquivos. Os dados saem como `FormData`, `multipart/form-data` ou dataURL, conforme a tela. O backend deve validar extensão, MIME real, tamanho, assinatura do arquivo, antivírus, armazenamento privado e permissão de download.

- Uso de armazenamento no navegador identificado nos scripts relacionados: `localStorage` em 23 ocorrência(s) e `sessionStorage` em 8 ocorrência(s). Tokens/segredos não devem ser armazenados nesses mecanismos; manter apenas estado não sensível.

- Cookies de sessão são tratados pelo backend em `api/auth.php` com `HttpOnly`, `SameSite=Lax` e `Secure` condicional a HTTPS/proxy. Ponto a validar em produção: HTTPS obrigatório, HSTS efetivo e domínio/path restritivos.

Campos/dados de entrada identificados no HTML: `jobIdInput` (number), `companySearch` (text), `statementFile` (file), `step2BankKey` (text), `step2BankName` (text), `step2Agency` (text), `step2Account` (text), `step3PlanKey` (select), `step3CaseMode` (select), `step3BankAccount` (text), `step3ClientAccount` (text), `step3SupplierAccount` (text), `step3DiscountPayAccount` (text), `step3DiscountReceiveAccount` (text), `step3InterestPayAccount` (text), `step3InterestReceiveAccount` (text), `step3DefaultHistoryCode` (text), `step3ThirdPartyAccount` (text), `step3CostCenter` (text), `step3Branch` (text), `step3DefaultCnpj` (text)

Quando alguma action for indireta ou montada dinamicamente no JavaScript, considerar: **ponto a validar no código/backend**.

---

## 5. APIs e integrações relacionadas

### Endpoint: `/api/integra_ai.php`

- **Action observada:** `create_job`
  - **Método provável:** POST
  - **Finalidade:** Criar processamento Integra-AI.
  - **Dados enviados:** Empresa e arquivo.
  - **Dados retornados:** ID do processamento.
  - **Validações esperadas:** Autorização e upload válido.
  - **Riscos de segurança:** Processamento de empresa errada.
  - **Melhorias recomendadas:** Confirmação de empresa e trilha de auditoria.
- **Action observada:** `download_txt`
  - **Método provável:** GET
  - **Finalidade:** Baixar TXT gerado.
  - **Dados enviados:** ID do job/arquivo.
  - **Dados retornados:** Arquivo TXT.
  - **Validações esperadas:** Autorização por job.
  - **Riscos de segurança:** Exfiltração de dados financeiros.
  - **Melhorias recomendadas:** Link temporário e auditoria de download.
- **Action observada:** `flags`
  - **Método provável:** GET
  - **Finalidade:** Obter flags de rollout/recursos.
  - **Dados enviados:** Nenhum.
  - **Dados retornados:** Flags.
  - **Validações esperadas:** Não expor segredos.
  - **Riscos de segurança:** Fingerprinting de recursos.
  - **Melhorias recomendadas:** Retornar apenas flags necessárias.
- **Action observada:** `generate_txt`
  - **Método provável:** POST
  - **Finalidade:** Gerar TXT Domínio.
  - **Dados enviados:** Job e configurações.
  - **Dados retornados:** Preview/arquivo.
  - **Validações esperadas:** Job autorizado e etapas concluídas.
  - **Riscos de segurança:** Arquivo contábil incorreto.
  - **Melhorias recomendadas:** Validação e reconciliação antes do download.
- **Action observada:** `list_step4_rules`
  - **Método provável:** GET
  - **Finalidade:** Listar regras contábeis da etapa 4.
  - **Dados enviados:** Job/empresa.
  - **Dados retornados:** Regras.
  - **Validações esperadas:** Autorização.
  - **Riscos de segurança:** Regra incorreta exposta.
  - **Melhorias recomendadas:** Versionamento de regras.
- **Action observada:** `load_job`
  - **Método provável:** GET
  - **Finalidade:** Retomar processamento.
  - **Dados enviados:** ID do processamento.
  - **Dados retornados:** Estado do job.
  - **Validações esperadas:** Autorização por empresa/processo.
  - **Riscos de segurança:** IDOR de processamento.
  - **Melhorias recomendadas:** Policy por job.
- **Action observada:** `parse_bank_statement`
  - **Método provável:** POST
  - **Finalidade:** Processar extrato bancário enviado.
  - **Dados enviados:** Arquivo de extrato e metadados.
  - **Dados retornados:** Transações interpretadas.
  - **Validações esperadas:** Upload seguro, autenticação e limites.
  - **Riscos de segurança:** Vazamento financeiro, OCR inseguro e upload malicioso.
  - **Melhorias recomendadas:** Quarentena, ClamAV, storage privado e isolamento de parser.
- **Action observada:** `save_step2`
  - **Método provável:** POST
  - **Finalidade:** Salvar confirmação bancária.
  - **Dados enviados:** Banco/agência/conta.
  - **Dados retornados:** Etapa salva.
  - **Validações esperadas:** CSRF e validação.
  - **Riscos de segurança:** Mapeamento bancário incorreto.
  - **Melhorias recomendadas:** Validação de conta e histórico.
- **Action observada:** `save_step3`
  - **Método provável:** POST
  - **Finalidade:** Salvar plano, modo e contas fixas.
  - **Dados enviados:** Plano de contas, contas fixas e parâmetros.
  - **Dados retornados:** Etapa salva.
  - **Validações esperadas:** CSRF e validação contábil.
  - **Riscos de segurança:** Geração contábil incorreta.
  - **Melhorias recomendadas:** Regras versionadas e revisão.
- **Action observada:** `save_step4_rule`
  - **Método provável:** POST
  - **Finalidade:** Salvar regra contábil.
  - **Dados enviados:** Regra e parâmetros.
  - **Dados retornados:** Confirmação.
  - **Validações esperadas:** CSRF e autorização.
  - **Riscos de segurança:** Regra maliciosa/incorreta.
  - **Melhorias recomendadas:** Aprovação e validação semântica.
- **Action observada:** `search_companies`
  - **Método provável:** GET
  - **Finalidade:** Pesquisar empresas para o processamento.
  - **Dados enviados:** Termo de busca.
  - **Dados retornados:** Empresas encontradas.
  - **Validações esperadas:** Usuário autorizado.
  - **Riscos de segurança:** Exposição de cadastro empresarial.
  - **Melhorias recomendadas:** Escopo e limite de resultados.
- **Action observada:** `search_plan_accounts`
  - **Método provável:** GET
  - **Finalidade:** Pesquisar contas do plano.
  - **Dados enviados:** Plano e termo.
  - **Dados retornados:** Contas.
  - **Validações esperadas:** Autorização.
  - **Riscos de segurança:** Exposição de plano de contas.
  - **Melhorias recomendadas:** Limitar por empresa/plano.

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

- Fluxo em quatro etapas: empresa/upload, confirmação bancária, plano/regras, geração de TXT.
- Job pode ser carregado por ID para retomada.
- Conta bancária, plano de contas e regras determinam o TXT final.

### Regras que devem existir obrigatoriamente no backend

- Isolar processamento de arquivos.
- Validar empresa selecionada e autorização por job.
- Registrar versão das regras usadas no TXT.
- Auditar download/exportação do TXT.
- Uploads devem ser validados por extensão, MIME, tamanho, assinatura, antivírus e armazenados fora da pasta pública.

---

## 7. Análise de segurança

### 7.1 Autenticação

`auth.js` está relacionado à página. A validação inicial ocorre no frontend chamando `/api/storage.php?action=auth_session_status`; endpoints PHP precisam manter a validação real de sessão.

### 7.2 Autorização

A autorização deve considerar perfil, departamento, propriedade do recurso e vínculo com cliente/processo. O frontend pode esconder botões, mas a decisão precisa estar no PHP.

### 7.3 Proteção contra XSS

Foram identificadas 54 ocorrência(s) de `innerHTML` nos scripts vinculados. Evitar inserir dados de API com `innerHTML`; preferir `textContent` ou sanitização contextual.

### 7.4 Proteção contra CSRF

Para actions de mutação, o padrão do projeto usa `X-CSRF-Token` via `SamaAuth.buildHeaders` e `sama_require_csrf()` em APIs críticas. Validar cobertura em todas as actions desta página.

### 7.5 Proteção contra SQL Injection

Os endpoints PHP analisados usam `PDO`/`prepare` em diversas consultas. Ainda assim, qualquer consulta dinâmica, filtro, `ORDER BY`, nome de tabela/coluna ou action genérica deve usar allowlist e prepared statements. Ponto a validar em cada endpoint relacionado.

### 7.6 Segurança em uploads

Há upload/manipulação de arquivo. Exigir allowlist de extensão, MIME real, assinatura mágica, tamanho máximo, nome gerado, quarentena, ClamAV, storage privado e download autenticado/autorizado.

### 7.7 Dados sensíveis

Dados sensíveis envolvidos: empresas, CNPJ, extratos, contas bancárias, plano de contas, regras contábeis, TXT Domínio e possíveis dados financeiros. Não expor esses dados no HTML inicial, em logs do navegador, em mensagens de erro, em query strings permanentes ou em `localStorage`.

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

- Arquivos JavaScript extensos/acoplados à tela: `global.js` (2433 linhas), `Contabil/integra-ai.js` (4257 linhas).
- Uso de `innerHTML` em scripts relacionados (54 ocorrência(s)), exigindo revisão de XSS.
- Uso de `localStorage` em scripts relacionados (23 ocorrência(s)); não armazenar tokens ou dados sensíveis.
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
- Vite + React + TypeScript para fluxo multi-etapas, estado de job e tratamento robusto de uploads/erros assíncronos.

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
- `integraAiService`
- `uploadService`
- `exportService`

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
- Atualizacao 2026-05-27 08:31: `portal-sama-web/tests/e2e/smoke.spec.ts` cobre localmente `/contabil/integra-ai` com API mockada, navegando para pagina 2 das regras, pesquisando conta por nome e validando autosave de conta editavel por selecao/blur. Ainda falta repetir com job real e usuario contabil em homologacao.
- Atualizacao 2026-05-27 10:03: o mesmo smoke valida que a tela reconhece `ofx_import=true` e publica `.ofx` no `accept`; API unit cobre OFX habilitado/desabilitado na validacao de upload.
- Atualizacao 2026-05-27 10:40: parser direto no PDF real do Banco Inter passou com transacoes reconhecidas e saldos diarios filtrados como linhas tecnicas.

### Testes de segurança

- Tentar XSS em campos livres e dados retornados pela API.
- Tentar IDOR alterando IDs, client_id, doc_id ou token.
- Verificar CSRF em actions POST/PUT/PATCH/DELETE.
- Enviar arquivo com extensão dupla, MIME falso, payload EICAR e nome malicioso.

---

## 14. Conclusão da página

A página `Contabil/integra-ai.html` atua no módulo **Contábil / Integra-AI** e manipula **empresas, CNPJ, extratos, contas bancárias, plano de contas, regras contábeis, TXT Domínio e possíveis dados financeiros**. Os principais riscos são: upload/download de arquivos. A principal recomendação é reforçar validações server-side, autorização por recurso, auditoria e componentização gradual.

- **Nível de criticidade:** Crítico
- **Prioridade de refatoração:** Urgente
- **Observações finais:** manter o HTML como camada de apresentação; regras de permissão, validação, persistência e auditoria devem residir no backend. Quando houver incerteza, tratar como **ponto a validar no código/backend**.


---

## 15. Atualização de stack alvo — TypeScript/NestJS

Esta seção complementa a análise original da página com a decisão técnica posterior de migrar o backend PHP para **Node.js + TypeScript + NestJS** e evoluir o frontend para **React + TypeScript + Vite**.

### 15.1 Mapeamento da tela para a arquitetura alvo

- **Arquivo HTML atual:** `Contabil/integra-ai.html`
- **Rota React sugerida:** `/contabil/integra-ai`
- **Componente React sugerido:** `IntegraAiPage.tsx`
- **Backend alvo:** `/api-v2`, implementado em NestJS
- **Banco principal:** MySQL 8 com Prisma ORM

### 15.2 Módulos NestJS relacionados

- `AccountingModule`
- `IntegraAiModule`
- `JobsModule`

### 15.3 Arquivos atuais que devem ser usados como referência de migração

- `Contabil/integra-ai.html`
- `api/integra_ai.php`
- `api/integra_ai_*`

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
