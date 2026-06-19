# Auditoria Deploy - Matriz de Rotas, Endpoints e Permissões

Data da análise: 10/06/2026  
Escopo complementar: rotas front-end, guardas de navegação, endpoints API, permissões e perfis padrão.

## Objetivo

Dar continuidade às etapas de análise do `Testes-da-aplicação-DEPLOY.md` cruzando:

- rotas públicas e autenticadas do front-end;
- endpoints públicos, autenticados e protegidos por permissão da API;
- permissões padrão por perfil;
- pontos em que a UI esconde ações, mas a API precisa garantir segurança real.

## Modelo de proteção encontrado

| Camada | Como protege | Evidência | Observação |
| --- | --- | --- | --- |
| Front-end - sessão | `AppLayout` chama `bootstrapSession()` e redireciona status `anonymous` para `/login`. | `portal-sama-web/src/components/layout/AppLayout.tsx:15`, `:30` | Protege o shell autenticado. |
| Front-end - menu | `navigation.tsx` filtra itens por permissão, role, departamento e cargo. | `portal-sama-web/src/components/layout/navigation.tsx:323` | Esconde atalhos, mas não é barreira de segurança. |
| Front-end - componentes | `PermissionGate` renderiza ou oculta trechos por permissão. | `portal-sama-web/src/components/common/PermissionGate.tsx:24` | É controle de UX, não substitui autorização no backend. |
| API - autenticação | Controllers internos usam `JwtAuthGuard`. | Ex.: `portal-sama-api/src/modules/clients/clients.controller.ts:28` | Exige Bearer token em rotas internas. |
| API - permissão | `PermissionsGuard` exige todas as permissões declaradas em `@Permissions`. | `portal-sama-api/src/common/guards/permissions.guard.ts:16` | Se não houver `@Permissions`, usuário autenticado passa. |
| API - público | Rotas com `@Public()` ou health não exigem sessão. | Ex.: auth, documentos públicos, propostas públicas, assinatura pública. | Precisam de token público forte, expiração e rate limit. |

## Rotas front-end

### Rotas públicas

| Rota | Tela | Status |
| --- | --- | --- |
| `/login` | Login | Pública |
| `/assinatura/:token` | Assinatura pública de contrato | Pública por token |
| `/onboarding/publico/documentos/:token` | Upload/checklist público de documentos | Pública por token |
| `/onboarding/publico/proposta/:token` | Proposta pública | Pública por token |
| `/dev/intro-preview` | Preview dev | Só em `import.meta.env.DEV` |

### Rotas autenticadas principais

Todas as rotas abaixo ficam sob `AppLayout`, então exigem sessão, mas a autorização fina depende da API e dos componentes internos.

| Rota | Módulo | Permissão/visibilidade principal no menu |
| --- | --- | --- |
| `/home` | Home | Sessão autenticada |
| `/clientes` | Clientes | `clients.read`, visível para ADMIN/MANAGER/cargo gestor |
| `/clientes/visao-geral` | Visão geral de clientes | `clients.read`, `documents.requirements` em partes da tela |
| `/clientes/:clientId/painel` | Painel do cliente | `clients.read`; ações por `client_assignments.*` e documentos |
| `/documentos` | Documentos | `documents.read`, `documents.upload`, `documents.download`, `documents.review` |
| `/departamentos/clientes` | Meus clientes | `clients.read` |
| `/departamentos/modelo` | Modelo de departamento | `departments.workspace.read/write` |
| `/departamentos/vencimentos` | Vencimentos | `departments.workspace.read`, `calendar.manage` em ações |
| `/contabil/integra-ai` | Integra-AI | `accounting.integra_ai.*` |
| `/certificados-digitais` | Certificados | `certificates.read/manage/download` |
| `/solicitacao-acesso` | Solicitação de acesso | Autenticado; ações por perfil |
| `/ti/acessos` | Aprovações de TI | `access_requests.read/approve/reject` |
| `/legalizacao` | Legalização | `legalization.*` |
| `/legalizacao/propostas` | Propostas | `proposals.*` |
| `/legalizacao/contratos` | Contratos | `contracts.*` |
| `/onboarding/entrada-cliente` | Entrada de cliente | `onboarding.create`, `clients.read` |
| `/onboarding/processos` | Processos de onboarding | `onboarding.*` |
| `/auditoria` | Auditoria | `audit.read` |
| `/manager/*` | Gestão | `transfers.*`, `calendar.*`, `manager_history.*` |
| `/dev` | Administração | `users.*`, `roles.*`, `permissions.*`, integrações técnicas |
| `/notificacoes` | Notificações | `notifications.read/create` |
| `/configuracoes` | Configurações | Sessão, com blocos por permissão |

## Endpoints API por família

| Família | Exemplos | Proteção declarada | Status de análise |
| --- | --- | --- | --- |
| Health | `GET /api-v2/health` | Público | OK para smoke. |
| Auth | `/auth/csrf`, `/auth/login`, `/auth/refresh`, `/auth/logout`, `/auth/forgot-password` | Públicos com CSRF onde aplicável | Parcialmente OK; E2E CSRF falhou por `.env` local. |
| Me | `/me/security`, `/me/avatar` | JWT | OK; validar fluxo real. |
| Usuários/Roles/Permissões | `/users`, `/roles`, `/permissions` | `users.*`, `roles.*`, `permissions.*` | Uso administrativo; precisa homologar DEV. |
| Clientes | `/clients`, `/clients/:id`, `/clients/:id/dashboard` | `clients.read/create/update/delete` | **Reprovado por escopo em `clients.read`.** |
| Responsabilidades | `/clients/:clientId/assignments`, `/client-assignments/*` | `client_assignments.*` | Tem checks departamentais; depende de clientes corrigido. |
| Colaboradores | `/collaborators` | `collaborators.*` | Parcial; validar escopo real. |
| Documentos | `/documents`, `/clients/:clientId/documents`, `/documents/upload` | `documents.*` | Parcial; ClamAV strict pendente. |
| Documentos públicos | `/public/onboarding/documents/:token`, `/documents/public-upload` | Público por token | Validar expiração/revogação/rate limit. |
| Certificados | `/certificates` | `certificates.*` | Parcial; exigir chave dedicada. |
| Propostas | `/proposals`, `/public/proposals/:token` | Interno por `proposals.*`; público por token | Validar fluxo real. |
| Contratos | `/contracts`, `/public/signatures/:token` | Interno por `contracts.*`; público por token | Validar assinatura real. |
| Onboarding | `/onboarding/processes` | `onboarding.*` | Parcial; validar real. |
| Legalização | `/legalization/processes`, `/legalization/templates` | `legalization.*` | Parcial; validar real. |
| Solicitações de acesso | `/access-requests` | Criação/autoconsulta autenticada; aprovação por `access_requests.*` | Intencional para self-service; validar regras de serviço. |
| Notificações/Web Push | `/notifications/*` | `notifications.read/create` | Parcial; E2E Web Push precisa ficar verde. |
| Integra-AI | `/accounting/integra-ai/*` | `accounting.integra_ai.*` | Parcial; testar arquivos reais. |
| Acessórias - técnico | `/integrations/acessorias/*` | `integrations.acessorias.*` | Parcial; token real pendente. |
| Acessórias - Home | `/integrations/acessorias/home-summary?profile=` | JWT, sem `@Permissions` | **Novo achado: profile escalation por query.** |

## Perfil padrão x permissões relevantes

| Perfil | Permissões resumidas | Observação de risco |
| --- | --- | --- |
| ADMIN | Todas | Amplo por desenho. |
| DEV | Todas | Amplo por desenho; deve ser altamente restrito. |
| MANAGER | Clientes, responsabilidades, documentos, propostas, contratos, onboarding, gestão, notificações | Documentação diz escopo próprio; precisa enforcement no serviço de clientes. |
| CLIENT | Documentos próprios, propostas, contratos, legalização e notificações | Precisa garantir escopo próprio em todas as consultas. |
| DEPARTMENT | `clients.read`, documentos, certificados, propostas, contratos, legalização, workspace | Risco atual: `clients.read` permite cliente fora do escopo. |
| ACCOUNTING | `clients.read`, documentos, Integra-AI, propostas, contratos, legalização | Risco atual: leitura ampla de clientes. |
| LEGALIZATION | `clients.read`, documentos, propostas, contratos, legalização | Risco atual: leitura ampla de clientes. |
| TI | Usuários/collaborators/departamentos/onboarding/access requests/notificações | Validar se o escopo de TI deve ver usuários administrativos. |
| AUDITOR | Auditoria e responsabilidades | Sem acesso automático aos recursos, bom desenho inicial. |

## Novos achados desta continuação

### AUTHZ-03 - Acessórias Home permite escalada de perfil via query

Severidade: **Crítico**.

Evidência:

- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.controller.ts:20` expõe `GET home-summary`.
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.controller.ts:22` recebe `profile` da query e repassa ao serviço.
- `portal-sama-api/src/common/guards/permissions.guard.ts:21` permite a rota quando não há `@Permissions`.
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.service.ts:91` não aplica filtro departamental quando `profile === 'admin'`.
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.service.ts:202` retorna todos os itens quando `profile === 'admin'`.
- `portal-sama-web/src/services/acessorias.service.ts:25` envia `profile` como parâmetro de query.

Impacto:

Um usuário autenticado pode chamar manualmente `/api-v2/integrations/acessorias/home-summary?profile=admin` e receber visão administrativa de entregas Acessórias. Os itens incluem cliente, documento do cliente, obrigação, departamento, responsável e vencimentos. Isso pode vazar dados operacionais entre departamentos/usuários.

Correção recomendada:

1. Não aceitar `profile` confiável vindo do cliente.
2. Derivar o perfil no backend a partir de roles/permissões do usuário autenticado.
3. Se `profile=admin` for mantido, exigir role `ADMIN`/`DEV` ou permissão técnica explícita.
4. Adicionar testes negativos:
   - `CLIENT` + `profile=admin` deve retornar 403 ou dados próprios.
   - `DEPARTMENT` + `profile=admin` não pode ver outros departamentos.
   - `ACCOUNTING`/`LEGALIZATION` não podem elevar para visão admin.

### UX-SEC-01 - Rotas autenticadas dependem de API para autorização fina

Severidade: **Médio**, com impacto maior quando combinado com falhas de API.

Evidência:

- `portal-sama-web/src/app/router.tsx` registra rotas autenticadas sem guard de permissão por rota.
- `AppLayout` valida apenas sessão.
- `navigation.tsx` e `PermissionGate` escondem menu/ações, mas não impedem navegação direta.

Impacto:

Usuário sem item visível no menu ainda pode digitar a URL. Isso é aceitável se a API bloquear corretamente, mas aumenta a importância de testes diretos por rota e endpoint. No caso de `clients.read`, a API não bloqueia o escopo, então a navegação direta reforça o vazamento.

Correção recomendada:

1. Manter API como fonte de verdade.
2. Adicionar guard de rota por permissão para melhorar UX e reduzir chamadas indevidas.
3. Testar navegação direta para todas as rotas críticas por perfil.

## Testes adicionais obrigatórios

| Teste | Tipo | Resultado esperado |
| --- | --- | --- |
| `DEPARTMENT` chama `/integrations/acessorias/home-summary?profile=admin` | API E2E | 403 ou resposta limitada ao escopo real. |
| `CLIENT` chama `/integrations/acessorias/home-summary?profile=admin` | API E2E | 403 ou resposta vazia/própria. |
| `MANAGER` chama `profile=admin` sem role admin | API E2E | Não deve receber visão global. |
| Usuário sem `clients.read` acessa `/clientes` direto no front | Playwright | Tela sem dados e API 403. |
| `DEPARTMENT` acessa `/clientes` direto | Playwright + API | Não deve listar clientes fora do escopo. |
| `ACCOUNTING` acessa `/clientes/:id/painel` de cliente fora do escopo | API E2E | 403. |
| Rotas públicas com token inválido/expirado | API + Playwright | 401/403/404 controlado, sem vazamento. |

## Decisão da continuidade

O mapeamento de rotas/endpoints reforça a decisão anterior: **não pronto para usuários reais**.

Além dos bloqueios já registrados, agora há um segundo ponto crítico de autorização ligado ao Acessórias Home. A ordem de correção recomendada passa a ser:

1. Remover `.env` do ZIP e rotacionar segredos.
2. Corrigir escopo de clientes.
3. Corrigir `home-summary` para derivar perfil no backend.
4. Fechar E2E API/web.
5. Validar ambiente real.

