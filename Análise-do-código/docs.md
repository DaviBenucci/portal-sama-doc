## Diagnóstico executivo

O **Portal Sama** é uma plataforma interna de gestão operacional para uma empresa contábil. A aplicação já está bem além de um protótipo: existe um frontend React/Vite com rotas protegidas, navegação por perfil, dashboards operacionais, módulos de clientes, documentos, certificados, legalização, contratos, propostas, onboarding, solicitações de acesso, notificações e integração com Acessórias. No backend, há uma arquitetura NestJS modular com Prisma/MySQL, autenticação JWT, refresh token em cookie `httpOnly`, CSRF, RBAC, auditoria, uploads privados, criptografia de senha de certificado, notificações/Web Push e pipeline local persistido para dados da Acessórias.

Meu veredito técnico: **o projeto está próximo de um MVP operacional, mas ainda não está pronto para produção com dados empresariais sensíveis**. Os maiores bloqueadores são: **presença de `.env` real nos diretórios locais da API/legado e necessidade de rotação formal de segredos**, homologação real de banco/Acessórias/Web Push/fluxos públicos, ausência de integração ativa com ZapSign, necessidade de revisão final de escopo de clientes/RBAC e substituição de jobs longos via HTTP por fila/background jobs.

Não vou expor nenhum valor sensível encontrado. A presença de `.env` real nos diretórios locais e qualquer inclusão anterior em pacote/artefato da API devem ser tratadas como **incidente de vazamento de segredo**: rotacionar tokens, senhas, JWT secrets, chaves de integração, chaves de criptografia, VAPID, SMTP/IMAP, OpenAI, Acessórias, ZapSign e qualquer credencial de banco ou bootstrap admin. A OWASP recomenda gestão centralizada, rotação e auditoria de segredos para reduzir impacto de vazamentos e dificuldade de rastreamento. ([OWASP Cheat Sheet Series][1])

## Atualização de validação Codex - 2026-06-19

Após nova validação local e pela internet, o diagnóstico precisa ser ajustado em alguns pontos. O ambiente atual comprova mais coisas do que a análise original conseguiu comprovar:

* `portal-sama-api`: `prisma validate`, ESLint, build NestJS, testes unitários e e2e passaram.
* Backend unitário: **44 suítes / 291 testes passando**.
* Backend e2e: **1 suíte / 136 testes passando**.
* `portal-sama-web`: ESLint, testes contratuais, build Vite e Playwright local passaram.
* Testes contratuais web: **12/12 passando**.
* Playwright local: **22 testes passando / 1 teste real pulado**.
* `portal-sama` legado: lint PHP passou em **62 arquivos**.
* Smoke público real em `https://portal.samacontabil.com.br`: frontend, health da API, CORS e CSRF passaram.
* Smoke autenticado real com usuário autorizado: login, cookie `httpOnly`, refresh, `/auth/me` e logout passaram.
* Smoke mínimo de permissões real: anônimo recebeu `401` nos endpoints protegidos e o usuário autorizado acessou endpoints administrativos esperados.
* Playwright real de autenticação no domínio público passou, incluindo login, home, política de cookies, ausência de token sensível em `localStorage`/`sessionStorage` e logout.
* MySQL descartável local em Docker: `prisma migrate deploy` aplicou **30 migrações**, `prisma:seed` executou, `prisma:bootstrap-admin` criou usuário privilegiado de teste e `ops:readiness` passou com banco, migrations, RBAC, usuário privilegiado e storage.

O que foi atualizado em relação à análise original:

* `Prisma validate` deixou de ser inconclusivo e passou neste ambiente.
* Build Vite deixou de ser pendente e passou neste ambiente.
* Os ZIPs atuais visíveis em `portal-sama` não confirmaram presença de `.env`, `.git`, `node_modules` ou `dist`; ainda assim, há `.env` real em `portal-sama` e `portal-sama-api`, então o processo de empacotamento seguro e rotação de segredos continua obrigatório.
* `ops:secrets:check` passou sem expor valores, mas apontou avisos de hardening: chave de certificado e token Acessórias abaixo do tamanho recomendado e ausência de marcador formal de rotação.
* `ops:readiness` usando o host de banco do `.env` não conseguiu acessar o banco fora do ambiente Docker/EasyPanel; por outro lado, o health público retornou `database=up` e `storage=up`, e uma validação em MySQL descartável confirmou que as migrações e o seed/RBAC aplicam corretamente.
* A readiness no MySQL descartável alertou que a collation default do banco de teste ficou `utf8mb4_0900_ai_ci`, enquanto a expectativa operacional documentada é `utf8mb4_unicode_ci`.
* ClamAV não foi encontrado localmente e o modo de scanner ainda não estava em `strict` na validação local.

Ainda não ficam comprovados por esses testes: payload real da Acessórias, sincronização incremental/backfill com volume real, validade jurídica do fluxo sem ZapSign, backup/restore real, matriz completa de perfis não-admin e revisão de escopo por objeto/cliente.

---

# 1. Visão da aplicação como usuário da plataforma

Para o usuário final, o Portal Sama funciona como um **painel operacional contábil**. O objetivo é concentrar em um só lugar as rotinas que hoje parecem depender de planilhas, controles externos, integrações e comunicação manual.

O fluxo principal é este:

1. **Login**

   * O usuário entra com credenciais.
   * A sessão é protegida por access token em memória, refresh token em cookie seguro e CSRF.
   * Depois do login, o portal identifica papéis, permissões, departamento e perfil operacional.

2. **Home personalizada**

   * Um colaborador enxerga pendências e obrigações relacionadas ao seu departamento ou responsabilidade.
   * Um gestor enxerga visão de equipe, vencimentos, responsáveis pendentes, transferências, histórico e carteira.
   * Um DEV/ADMIN/TI enxerga áreas administrativas, auditoria, painéis de integração e validação de acessos.

3. **Menu lateral por contexto**
   A navegação é segmentada por grupos:

   * **Operação:** início, clientes, departamentos, documentos, certificados digitais, vencimentos, Integra-AI, legalização, propostas e contratos.
   * **Gestão:** painel do gestor, colaboradores, transferências, calendário/configurações e histórico.
   * **Entrada:** onboarding e entrada de cliente.
   * **T.I.:** solicitações e validação de acessos.
   * **Admin/DEV:** usuários, auditoria, integrações e ferramentas internas.

4. **Central de Vencimentos**
   Este parece ser o coração operacional do MVP. Ela consolida obrigações internas e obrigações vindas da Acessórias, com filtros por departamento, colaborador, competência, status, origem e período. A ideia é o usuário não precisar consultar diretamente a Acessórias para acompanhar tudo.

5. **Clientes e colaboradores**
   O sistema permite consultar clientes, painéis de cliente, responsáveis, colaboradores, carteiras e vínculos operacionais.

6. **Documentos e certificados**
   Há fluxo para upload, armazenamento privado, validação de tipo de arquivo, hash, scanner e controle de documentos. Certificados digitais recebem tratamento específico, com criptografia de senha.

7. **Legalização, propostas e contratos**
   O portal possui fluxo interno para propostas, contratos e assinatura pública via token. Porém, isso hoje é um fluxo interno por token público; **não encontrei integração ativa com ZapSign no código atual**.

8. **Onboarding e links públicos**
   Clientes externos podem acessar links públicos para enviar documentos, aprovar propostas ou assinar contratos. Esses fluxos não exigem login, então precisam de proteção forte contra abuso, expiração, rate limit e monitoramento.

9. **Notificações**
   Existe central de notificações, sino no header e infraestrutura de Web Push. A documentação define corretamente que push deve ser resumido e sem dados sensíveis.

Como experiência de usuário, a proposta é boa: o portal tenta transformar várias rotinas contábeis em um painel único, filtrado por perfil. O ponto que ainda precisa de polimento é a **simplicidade visual e textual**: há labels sem acento, páginas grandes, muitos módulos e nomes técnicos demais para usuários não desenvolvedores.

---

# 2. Visão como DEV Sênior: arquitetura geral

A arquitetura atual é um **monólito modular**, o que é uma decisão correta para o estágio do projeto. Não recomendo migrar para microserviços agora. A aplicação já tem muitos domínios de negócio; dividir prematuramente aumentaria custo operacional, segurança, deploy, observabilidade e consistência de dados.

## Frontend

Stack principal:

* React 19.
* TypeScript.
* Vite.
* React Router 7.
* TanStack React Query.
* Axios.
* Zustand.
* React Hook Form.
* Zod.
* Tailwind.
* Lucide.
* GSAP em alguns pontos visuais.

O frontend está estruturado com:

* rotas públicas;
* rotas autenticadas;
* layout protegido;
* sidebar filtrada por permissão/perfil;
* services por domínio;
* store de autenticação em memória;
* interceptors Axios para token, CSRF e refresh;
* páginas de operação, gestão, onboarding, TI, admin e notificações.

## Backend

Stack principal:

* NestJS 11.
* TypeScript.
* Prisma 6.
* MySQL.
* JWT.
* bcrypt.
* cookie-parser.
* Helmet.
* Throttler.
* Multer.
* Web Push.
* class-validator/class-transformer.
* Swagger fora de produção.

Módulos principais encontrados:

* `auth`;
* `users`;
* `roles`;
* `permissions`;
* `audit`;
* `clients`;
* `collaborators`;
* `documents`;
* `certificates`;
* `contracts`;
* `proposals`;
* `access-requests`;
* `legalization`;
* `onboarding`;
* `notifications`;
* `integrations/acessorias`;
* `calendar`;
* `managers`;
* `departments`;
* `health`;
* `transfers`;
* `accounting`.

O backend usa prefixo global:

```txt
/api-v2
```

E configura:

* `helmet`;
* CORS com origem explícita;
* cookies;
* validação global com `whitelist`, `forbidNonWhitelisted` e `transform`;
* filtro global de exceções;
* interceptor de request id;
* Swagger apenas fora de produção;
* Throttler global.

Do ponto de vista arquitetural, isso é positivo. A base não está improvisada.

---

# 3. Estado dos testes, build e qualidade técnica

Executei uma leitura semântica e também validações técnicas possíveis no ambiente. Em 2026-06-19, uma nova rodada no ambiente Codex com acesso local e internet ampliou a validação.

## Backend

Resultado encontrado:

* Build NestJS manual: **passou**.
* ESLint backend: **passou**.
* Testes backend: **passaram**.
* Total observado: **44 suítes / 291 testes passando**.
* E2E backend: **passou**, com **1 suíte / 136 testes passando**.
* Prisma validate: **passou**.
* Migrações Prisma em MySQL descartável: **30 migrações aplicadas com sucesso**.
* Seed/RBAC em MySQL descartável: **passou**.
* Readiness em MySQL descartável: **passou** após bootstrap admin de teste, com warnings de scanner não strict, collation default do banco de teste e backup/rollback.

Conclusão: o backend está com boa cobertura automatizada para o estágio atual e as migrações aplicam em um MySQL 8.4 descartável. Porém, testes unitários/e2e e banco descartável não provam que produção está pronta: ainda falta validar migrações no ambiente alvo, Acessórias com payload real, Web Push real, SMTP/IMAP, backup/restore e escopo por objeto/cliente.

## Frontend

Resultado encontrado:

* TypeScript `tsc -b`: **passou**.
* ESLint frontend: **passou**.
* Testes contratuais web: **passaram**.
* Testes contratuais web observados na nova rodada: **12/12 passando**.
* Build Vite: **passou**, com `2032 modules transformed`.
* Playwright local: **passou**, com **22 testes passando / 1 teste real pulado**.
* Playwright real de autenticação no domínio público: **passou**.

O problema anterior de dependência nativa opcional do bundler/Rolldown não apareceu na nova rodada. Mesmo assim, o empacotamento correto continua sendo instalar dependências a partir de lockfile confiável em CI/CD, sem versionar ou zipar `node_modules`:

```bash
rm -rf node_modules package-lock.json
npm install
npm run build
```

ou, preferencialmente, usando lockfile confiável e:

```bash
npm ci
npm run build
```

---

# 4. O que já foi feito

## Produto e escopo

Já existe uma visão clara de MVP nas documentações ativas. A documentação define que o foco não é mais descoberta, e sim **fechamento/consolidação**. Isso é importante: o projeto não precisa de mais módulos novos agora; precisa estabilizar, homologar, proteger e polir.

Já estão previstos ou implementados:

* autenticação;
* RBAC;
* auditoria;
* CSRF;
* sessão com refresh token;
* clientes;
* colaboradores;
* documentos;
* certificados;
* contratos;
* propostas;
* legalização;
* onboarding;
* solicitações de acesso;
* Central de Vencimentos;
* integração Acessórias local persistida;
* notificações internas;
* Web Push;
* dashboards por perfil;
* painel DEV/admin;
* documentação de segurança/deploy/MVP;
* testes automatizados relevantes.

## Segurança básica

Pontos positivos já implementados:

* access token em memória no frontend;
* refresh token em cookie `httpOnly`;
* rotação de refresh token;
* CSRF para mutações autenticadas;
* validação global de DTOs;
* `helmet`;
* CORS com credenciais e origem configurável;
* throttling;
* auditoria de eventos;
* armazenamento privado de arquivos;
* hash SHA-256 de documentos;
* validação de extensão, MIME e assinatura de arquivo;
* scanner de upload com modo ClamAV;
* criptografia AES-GCM para senha de certificado;
* tokens públicos armazenados por hash, não em texto puro;
* expiração/revogação de tokens públicos;
* Swagger desabilitado em produção.

A estratégia de CSRF está alinhada com boas práticas: aplicações que usam cookies para autenticação devem proteger requisições state-changing com token validado no backend; a OWASP também recomenda checagem de origem/referer como defesa complementar. ([OWASP Cheat Sheet Series][2])

## Integração Acessórias

A integração Acessórias foi desenhada de forma correta conceitualmente: a API externa não é usada como fonte direta para a tela a cada acesso. O portal persiste localmente os dados e mostra ao usuário uma visão local, auditável e resiliente.

O código possui:

* catálogo local de obrigações por empresa;
* entregas locais;
* sincronização incremental;
* backfill por empresa;
* controle de execução;
* lock de job em banco;
* rate limiter;
* tratamento de `429`;
* retries;
* timeout;
* checkpoint;
* status de sincronização;
* warnings de stale data;
* reconciliação de responsáveis;
* aliases pendentes/ambíguos;
* home baseada em dados locais;
* Central de Vencimentos consolidada.

Isso é muito melhor do que chamar a Acessórias diretamente em cada página.

## Banco de dados

O schema Prisma está amplo e cobre os principais domínios. Há índices importantes em entregas Acessórias, responsáveis, status, competência, vencimento, cliente, departamento e sync run. Para a quantidade de módulos já existentes, a modelagem está razoavelmente madura.

Modelos importantes encontrados:

* `User`;
* `Role`;
* `Permission`;
* `UserRole`;
* `RolePermission`;
* `RefreshToken`;
* `Client`;
* `Collaborator`;
* `Document`;
* `DigitalCertificate`;
* `Proposal`;
* `Contract`;
* `AccessRequest`;
* `OnboardingProcess`;
* `Notification`;
* `BrowserPushSubscription`;
* `PublicToken`;
* `AuditLog`;
* `AcessoriasDelivery`;
* `AcessoriasCompanyObligationCatalog`;
* `AcessoriasResponsibleAlias`;
* `AcessoriasSyncRun`;
* `AcessoriasExternalJobLock`.

---

# 5. O que falta para completar o projeto

## Bloqueadores críticos antes de produção

### 1. Remover e rotacionar segredos

Há `.env` real nos diretórios locais do legado e da API, com múltiplas credenciais e chaves sensíveis. Os ZIPs atuais visíveis em `portal-sama` não confirmaram `.env`, `.git`, `node_modules` ou `dist`, mas qualquer histórico de pacote, backup compartilhado ou artefato gerado antes da revisão deve ser tratado como risco. Isso continua sendo o bloqueador mais crítico.

Ação necessária:

* remover `.env` de qualquer pacote, commit, backup compartilhado ou artefato;
* manter apenas `.env.example`;
* rotacionar todos os segredos;
* invalidar credenciais de bootstrap admin;
* trocar JWT secrets;
* trocar CSRF secret;
* trocar senha do banco;
* trocar token Acessórias;
* trocar token ZapSign, mesmo que ainda não esteja implementado;
* trocar SMTP/IMAP;
* trocar VAPID private key;
* trocar chaves de criptografia;
* trocar API keys;
* criar pipeline de empacotamento seguro que exclua `.env`, `.git`, `node_modules`, `dist`, logs, `.ai-tests` e artefatos de teste.

### 2. Homologar banco e migrações reais

Os testes passam, o domínio público respondeu health com `database=up` e `storage=up`, e um MySQL descartável local aplicou as 30 migrações com seed/RBAC/bootstrap admin de teste. Ainda assim, a readiness local não conseguiu alcançar o host de banco do `.env` fora do ambiente Docker/EasyPanel, então ainda falta comprovar de forma operacional no ambiente alvo:

* `prisma migrate deploy` no banco real;
* seed/RBAC em ambiente real;
* usuário bootstrap desativado ou trocado;
* readiness da API;
* rollback;
* backup e restore drill;
* volume de documentos;
* permissões de storage privado.

### 3. Corrigir empacotamento do frontend

O build Vite foi comprovado na rodada de 2026-06-19, mas o risco de empacotamento continua válido: o correto é nunca depender de `node_modules` zipado.

Ação:

* excluir `node_modules` do ZIP;
* garantir `package-lock.json`/lockfile confiável;
* rodar instalação limpa no CI/CD;
* buildar em ambiente compatível com produção.

### 4. Implementar ou remover ZapSign

Encontrei variáveis de ZapSign no `.env`, e documentação arquivada cita ZapSign em sistema antigo, mas **não encontrei integração ativa com ZapSign no código atual**.

Hoje o fluxo de contratos/propostas usa token público interno. Portanto:

* se assinatura oficial depende da ZapSign, falta implementar um módulo real;
* se ZapSign saiu do escopo, remover variáveis, docs antigas e expectativas de produto;
* se será usado depois, documentar como “futuro”, não como parte do MVP atual.

### 5. Corrigir links públicos divergentes

Existe uma divergência importante entre backend e frontend.

O frontend define rotas públicas modernas como:

```txt
/assinatura/:token
/onboarding/publico/proposta/:token
/onboarding/publico/documentos/:token
```

Mas alguns serviços backend ainda geram links com padrão legado, por exemplo:

```txt
/Legalizacao/assinatura.html?token=...
/Onboarding/proposta-cliente.html?token=...
```

Isso pode quebrar o fluxo real enviado ao cliente. Antes da homologação, padronizar todos os links públicos para as rotas React atuais ou criar redirects explícitos no proxy/frontend.

### 6. Homologar Acessórias com API real

A integração está bem desenhada, mas precisa validação real com:

* token real;
* paginação real;
* payload real;
* `429`;
* `401/403`;
* timeouts;
* mudanças de campos;
* volumes grandes;
* backfill por empresa;
* incremental por `DtLastDH`;
* falhas parciais;
* reprocessamento;
* divergência de responsáveis.

### 7. Proteger endpoints públicos contra bots

Os endpoints públicos por token são necessários, mas são pontos de ataque naturais:

* upload público de documentos;
* assinatura pública;
* aprovação/ajuste de proposta;
* links de onboarding.

Eles devem ter:

* rate limit específico;
* Turnstile/reCAPTCHA em ações sensíveis;
* expiração curta;
* limite por IP/token;
* limite de tamanho e quantidade;
* auditoria;
* bloqueio por comportamento;
* WAF/CDN;
* alertas de abuso.

A OWASP API Top 10 2023 destaca riscos como autorização quebrada por objeto, autorização quebrada por função e consumo irrestrito de recursos; isso é especialmente relevante para APIs com IDs, tokens públicos, uploads e operações custosas. ([OWASP][3])

---

# 6. Análise crítica do frontend

## Pontos fortes

O frontend tem boa base técnica:

* React com TypeScript;
* separação por services;
* autenticação centralizada;
* interceptors para refresh token;
* TanStack Query para cache/estado assíncrono;
* Zustand para sessão em memória;
* rotas lazy-loaded;
* layout autenticado;
* navegação por permissão;
* páginas públicas separadas;
* testes contratuais;
* lint sem erro;
* TypeScript sem erro.

A decisão de manter o access token em memória é positiva. Evita persistir token em `localStorage`, reduzindo impacto de roubo persistente via XSS.

## Navegação e experiência do usuário

A navegação está funcional, mas ainda precisa de polimento para usuário comum.

Problemas percebidos:

1. **Muitos módulos no menu**
   O sistema tem várias áreas: clientes, documentos, certificados, legalização, propostas, contratos, onboarding, TI, gestão, vencimentos, Integra-AI, DEV. Isso pode intimidar usuários novos.

2. **Labels técnicos**
   Alguns nomes parecem mais voltados a desenvolvedor ou equipe interna experiente do que usuário operacional.

   Exemplos de melhoria:

   * `Modelo dpto` → `Modelo operacional por departamento`;
   * `Config. vencimentos` → `Configurar vencimentos`;
   * `Acessos TI` → `Validação de acessos`;
   * `Integra-AI` → manter, mas adicionar subtítulo explicativo;
   * `Legalização` → separar claramente processos, propostas e contratos.

3. **Ausência de acentos em alguns textos**
   Aparecem termos como `Operacao`, `Inicio`, `Legalizacao`, `Transferencias`, `Historico`, `Usuarios`. Isso reduz percepção de acabamento profissional.

4. **Páginas muito grandes**
   Algumas páginas têm tamanho excessivo:

   * `DevAdminPage`: mais de 2600 linhas;
   * `IntegraAiPage`: mais de 1600 linhas;
   * `ClientDashboard`: mais de 1400 linhas;
   * `Legalization`: mais de 1200 linhas;
   * `Notifications`: mais de 1100 linhas;
   * `Documents`: mais de 1000 linhas.

   Isso dificulta manutenção, testes, revisão de segurança, acessibilidade e evolução.

5. **Operações longas na interface**
   O frontend possui timeouts longos para ações da Acessórias, chegando a 10 minutos. Isso é ruim para UX e infraestrutura. O usuário não deve ficar preso em uma requisição HTTP aberta.

   Melhor abordagem:

   * usuário clica em “Sincronizar”;
   * backend retorna `202 Accepted` com `syncRunId`;
   * frontend mostra progresso;
   * polling ou WebSocket/SSE acompanha status;
   * notificação informa conclusão/falha.

## Recomendações frontend

### Refatoração estrutural

Separar páginas grandes em:

```txt
pages/
features/
components/
hooks/
services/
schemas/
types/
```

Exemplo para `IntegraAiPage`:

```txt
features/integra-ai/
  components/
    IntegraAiHeader.tsx
    IntegraAiUploadPanel.tsx
    IntegraAiJobTable.tsx
    IntegraAiRulesPanel.tsx
    IntegraAiResultDrawer.tsx
  hooks/
    useIntegraAiJobs.ts
    useIntegraAiUpload.ts
  services/
    integraAi.service.ts
  schemas/
    integraAi.schema.ts
  pages/
    IntegraAiPage.tsx
```

### Design system

Como o projeto já usa Tailwind, eu recomendo adotar um padrão baseado em:

* Radix UI;
* shadcn/ui;
* TanStack Table;
* React Hook Form + Zod;
* componentes internos padronizados.

Isso ajudaria em:

* modais;
* drawers;
* tabelas;
* badges;
* filtros;
* empty states;
* loading states;
* formulários;
* mensagens de erro;
* acessibilidade.

### Acessibilidade

Adicionar:

* testes com `axe-core`;
* validação Playwright para fluxos principais;
* foco visível;
* navegação por teclado;
* labels reais em inputs;
* aria-label nos botões de ícone;
* contraste adequado.

### Observabilidade frontend

Adicionar:

* Sentry ou equivalente;
* breadcrumbs de navegação;
* rastreamento de erro por rota;
* request id propagado da API;
* logging sanitizado.

---

# 7. Análise crítica do backend

## Pontos fortes

O backend é a parte mais madura do projeto.

Pontos positivos:

* NestJS modular;
* Prisma centralizado;
* DTO validation global;
* `forbidNonWhitelisted`;
* RBAC;
* guards por controller;
* auditoria;
* CSRF;
* refresh token rotativo;
* armazenamento privado;
* upload validation;
* scanner;
* criptografia de senha de certificado;
* public tokens com hash;
* integração Acessórias persistida;
* locks de sync no banco;
* testes passando;
* lint passando;
* build passando.

## Autenticação e sessão

A autenticação é bem superior à média de projetos em MVP.

Fluxo observado:

* login com senha bcrypt;
* access token JWT curto;
* refresh token opaco;
* refresh token armazenado como HMAC no banco;
* cookie `httpOnly`;
* rotação no refresh;
* logout revogando token;
* auditoria de sucesso/falha;
* CSRF exigido para mutações autenticadas.

Melhorias recomendadas:

1. **MFA/passkeys para contas críticas**
   DEV, ADMIN, TI e gestores deveriam ter MFA obrigatório.

2. **Rate limit distribuído**
   O throttling atual tende a ser in-memory. Em múltiplas réplicas, cada instância teria seu próprio limite. Usar Redis/Valkey para rate limit distribuído.

3. **Bloqueio progressivo de login**
   Além de throttle, aplicar:

   * lock temporário por usuário;
   * detecção de credential stuffing;
   * alertas de login suspeito;
   * CAPTCHA/Turnstile após tentativas falhas.

4. **Sessão com versionamento**
   Como o JWT carrega roles/permissões, alterações de RBAC podem demorar até o token expirar. Uma solução é incluir `sessionVersion` ou `permissionVersion` no usuário e invalidar tokens antigos em alterações críticas.

## RBAC e autorização

O backend usa `JwtAuthGuard` e `PermissionsGuard` em vários controllers. Isso é bom. Porém, eu recomendo endurecer o padrão:

* tornar autenticação global por padrão;
* usar decorator `@Public()` somente onde for realmente público;
* manter permissões explícitas em endpoints sensíveis;
* criar testes automatizados de matriz de permissões;
* validar escopo por objeto, não apenas por permissão global.

Esse último ponto é essencial: não basta o usuário ter `clients.read`; ele precisa poder ler **aquele cliente específico**. O risco de BOLA — acesso indevido por manipulação de ID — é um dos principais riscos em APIs modernas segundo a OWASP API Security Top 10. ([OWASP][3])

A documentação já marca preocupação com “client scope”. Eu trataria isso como item de homologação obrigatória: revisar `ClientsService`, painéis de cliente, documentos, certificados e dashboard para garantir que colaborador departamental não consiga listar ou acessar clientes fora do seu escopo operacional.

## Banco de dados e concorrência

A modelagem está boa para MVP, mas alguns pontos merecem atenção.

### Pontos positivos

* índices em tabelas de entregas Acessórias;
* tokens públicos por hash;
* refresh tokens revogáveis;
* auditoria estruturada;
* documentos com hash;
* contratos/propostas com status;
* aliases de responsáveis;
* sync runs persistidos;
* lock de job externo no banco.

### Pontos de atenção

1. **Pool de conexões**
   Não vi controle explícito de pool no código. Em Prisma/MySQL, isso normalmente fica no `DATABASE_URL` ou na infraestrutura. Para muitos usuários simultâneos, definir limites de conexão, timeout e eventualmente usar proxy/pooler.

2. **Jobs longos**
   Sincronização Acessórias, backfill e processamentos pesados não devem depender de request HTTP longa. Isso pode prender worker, proxy, navegador e gateway.

3. **Campos de data como string**
   Alguns modelos usam datas como `String`, provavelmente por compatibilidade com legado. Para consultas, ordenação e filtros consistentes, `DateTime` é preferível.

4. **Status como string livre**
   Em alguns pontos, status externos são strings. Isso é compreensível para payload externo, mas a camada operacional deve normalizar para enums internos sempre que possível.

5. **Índices futuros**
   Antes de produção com volume real, rodar explain/análise em:

   * Central de Vencimentos;
   * Home por colaborador;
   * Dashboard gestor;
   * busca de documentos;
   * listagem de clientes;
   * entregas por competência/vencimento/status.

## Integração Acessórias

A integração com a Acessórias está conceitualmente correta, mas precisa amadurecer operacionalmente.

### O que está bom

* Não depende da API externa a cada carregamento de tela.
* Persiste dados localmente.
* Possui sync incremental.
* Possui backfill.
* Possui lock em banco.
* Possui retries.
* Possui rate limiter.
* Trata `429`.
* Detecta dados desatualizados.
* Separa cadastro/catálogo de obrigações e entregas.
* Reconciliador de responsáveis evita criar usuário automaticamente a partir de nome externo.

### Riscos

1. **Rate limiter in-memory**
   Se a API rodar com duas ou mais réplicas, cada réplica pode disparar sua própria fila e furar limite da Acessórias.

2. **Jobs ainda acoplados ao HTTP**
   O frontend tem timeouts longos e o backend executa fluxos pesados no contexto da requisição. Isso deve migrar para fila.

3. **Falhas retornando `ok: true` com warning**
   Em alguns fluxos, falha parcial ou erro de sync pode retornar `ok: true` com aviso e status interno. Isso facilita UX otimista, mas é perigoso: integrações e telas podem interpretar como sucesso.

   Melhor padrão:

   ```json
   {
     "ok": false,
     "status": "FAILED_PARTIAL",
     "syncRunId": "...",
     "warning": "...",
     "recoverable": true
   }
   ```

4. **Payload real ainda precisa validação**
   A integração parece baseada em contratos esperados, mas produção exige homologação com payloads reais, páginas reais, campos ausentes, campos nulos e mudanças de nomenclatura.

## ZapSign

Aqui existe uma lacuna clara.

O projeto possui variáveis de ambiente relacionadas à ZapSign, mas não encontrei implementação ativa consumindo a API da ZapSign no backend ou frontend atual. As menções mais claras aparecem em documentação arquivada de migração/legado.

Conclusão:

* **A ZapSign não faz parte funcional do código atual.**
* Se contratos precisam assinatura juridicamente integrada via ZapSign, isso ainda falta.
* O fluxo atual é assinatura pública interna por token, com coleta de nome/documento/evidência.
* Antes de vender isso como “assinatura ZapSign”, precisa implementar:

  * client HTTP ZapSign;
  * criação de documento;
  * upload de PDF;
  * signers;
  * webhook de status;
  * persistência de external id;
  * reconciliação de status;
  * retries;
  * auditoria;
  * validação de callback;
  * ambiente sandbox/prod;
  * rotação de token.

## Uploads, documentos e certificados

A base é boa.

Documentos têm:

* validação de extensão;
* validação MIME;
* magic bytes;
* hash;
* storage privado;
* bloqueio de path traversal;
* scanner;
* cache-control `no-store`.

Certificados têm:

* tipos restritos `p12/pfx`;
* limite menor;
* armazenamento privado;
* senha criptografada com AES-GCM.

Isso está alinhado a recomendações de defesa em profundidade para upload: allowlist de extensão, validação de tipo, verificação de assinatura, armazenamento seguro e proteção contra arquivos maliciosos. ([OWASP Cheat Sheet Series][4])

Melhorias necessárias:

* usar scanner em modo `strict` em produção;
* validar ClamAV no readiness;
* limitar/evitar `.zip` e `.rar` se não forem indispensáveis;
* proteger contra zip bomb;
* limitar quantidade por token público;
* aplicar quota por cliente;
* analisar CDR para PDFs/Office se o risco justificar;
* nunca servir arquivo diretamente de pasta pública;
* garantir storage fora do webroot.

## Contratos e propostas públicas

O uso de `PublicToken` com hash é uma boa decisão. O token bruto não fica armazenado diretamente no banco.

Pontos bons:

* token com expiração;
* token revogável;
* escopo por módulo/entidade;
* hash no banco;
* assinatura/proposta vinculada a IP/user-agent;
* tokens anteriores revogados em alguns fluxos.

Pontos a corrigir:

* padronizar links públicos com as rotas React atuais;
* adicionar bot protection;
* definir limite de tentativas por token;
* definir limite de upload por token;
* melhorar status de token expirado/revogado para UX;
* revisar validade jurídica se não houver ZapSign;
* adicionar trilha de auditoria mais explícita para aceite/assinatura.

## Segurança contra bots e hackers

A aplicação já tem uma base defensiva, mas para dados empresariais ainda falta endurecimento.

Prioridades:

1. **Rotação de segredos imediatamente.**
2. **MFA obrigatório para perfis críticos.**
3. **Rate limit distribuído com Redis/Valkey.**
4. **WAF/CDN na frente da API.**
5. **Turnstile/reCAPTCHA nos fluxos públicos e login suspeito.**
6. **Revisão de BOLA/BFLA em todos os endpoints com IDs.**
7. **Testes de autorização por perfil.**
8. **CSP forte no frontend.**
9. **Logo e assets externos hospedados localmente.**
10. **Scanner de upload obrigatório em produção.**
11. **Observabilidade com alertas.**
12. **Backup/restore testado.**
13. **Logs sem dados sensíveis.**
14. **Segregação de ambientes.**
15. **Remoção de Swagger em produção, já prevista.**

---

# 8. Análise das documentações

As documentações do `portal-sama-docs` estão úteis, mas precisam ser tratadas com hierarquia.

## O que está bom

A documentação ativa define bem:

* objetivo de fechar MVP;
* escopo dentro/fora;
* importância de segurança;
* cuidado com dados empresariais;
* regra de não expor tokens;
* Acessórias como fonte externa e Portal como camada local persistida;
* Central de Vencimentos como tela operacional única;
* notificações sanitizadas;
* reconciliação de responsáveis;
* necessidade de deploy/homologação;
* itens pendentes de segurança.

A documentação também acerta ao dizer que o projeto já passou da fase de “descoberta” e precisa de fechamento.

## Problemas na documentação

1. **Docs arquivadas podem confundir**
   A pasta `_arquivo` contém histórico útil, mas pode levar alguém a acreditar que integrações antigas, como ZapSign PHP, ainda existem.

2. **Algumas informações estavam desatualizadas**
   A nova rodada comprovou TypeScript, build Vite, testes contratuais e Playwright local/real. O cuidado pendente deixou de ser "build não comprovado" e passou a ser "build reproduzível em CI/CD com instalação limpa e pacote seguro".

3. **A documentação reconhece o incidente do `.env`**
   Isso é positivo, mas o projeto ainda precisa tratar rotação, pacote seguro e registro de incidente como bloqueadores reais, não apenas observações.

4. **Matriz final de aceite criada**
   A rodada de correção criou documentação objetiva de go-live:

   ```md
   docs/08-CHECKLIST-GO-LIVE.md
   docs/09-MATRIZ-PERMISSOES-RBAC.md
   docs/10-MATRIZ-ROTAS-PUBLICAS.md
   docs/11-RUNBOOK-INCIDENTE-SEGREDOS.md
   docs/12-RUNBOOK-ACESSORIAS.md
   docs/13-RUNBOOK-BACKUP-RESTORE.md
   docs/14-TESTES-HOMOLOGACAO-MVP.md
   ```

   Esses documentos devem virar a fonte operacional de aceite. O `docs.md` continua como diagnóstico executivo e não deve carregar detalhes sensíveis nem evidências brutas.

---

# 9. Principais erros ou riscos encontrados

## Críticos

| Área           |                                                 Risco | Impacto                                                 |
| -------------- | ----------------------------------------------------: | ------------------------------------------------------- |
| Segredos       | `.env` real nos diretórios locais e histórico de empacotamento a revisar | Vazamento de credenciais, invasão, acesso a integrações |
| ZapSign        |            Variáveis existem, implementação ativa não | Expectativa de assinatura externa não atendida          |
| Links públicos | Backend gera links legados diferentes das rotas React | Cliente pode receber link quebrado                      |
| Acessórias     |                             Homologação real pendente | Dados incorretos ou sync falhando em produção           |
| Jobs longos    |                        Sync via HTTP com timeout alto | Travamento, falhas em proxy, UX ruim                    |
| Rate limit     |                    In-memory em cenário multi-réplica | Bots e abuso podem contornar limite                     |
| RBAC/escopo    |           Necessário provar escopo por cliente/objeto | Vazamento entre clientes/departamentos                  |

## Altos

| Área            |                             Risco | Impacto                              |
| --------------- | --------------------------------: | ------------------------------------ |
| Upload público  |           Abuse/bot/file flooding | Custo, DoS, malware                  |
| Scanner         |     Se não estiver strict em prod | Arquivo malicioso aceito             |
| Certificado     | Fallback de chave de criptografia | Acoplamento indevido de segredo      |
| Frontend        |             Páginas muito grandes | Manutenção difícil, bugs regressivos |
| Observabilidade |   Alertas e tracing insuficientes | Incidentes sem diagnóstico rápido    |
| Build           | Empacotamento com `node_modules` ou sem instalação limpa | Deploy não reprodutível              |

## Médios

| Área                |                         Risco | Impacto               |
| ------------------- | ----------------------------: | --------------------- |
| Labels sem acento   | Percepção de acabamento baixa | UX menos profissional |
| Menu grande         |        Usuário pode se perder | Adoção menor          |
| Status string livre |       Inconsistência de dados | Relatórios ruins      |
| Datas string        |      Ordenação/filtro frágeis | Bugs em consultas     |

---

# 10. Recomendações de arquitetura e frameworks

## Manter

Eu manteria:

* React + Vite;
* NestJS;
* Prisma;
* MySQL;
* monólito modular;
* TanStack Query;
* Zustand para sessão em memória;
* Zod/DTO validation;
* RBAC atual, mas endurecido.

## Adicionar no frontend

1. **Radix UI + shadcn/ui**
   Para padronizar componentes, acessibilidade, dialogs, selects, tooltips, dropdowns e forms.

2. **TanStack Table**
   Para tabelas grandes de clientes, documentos, vencimentos, entregas e auditoria.

3. **Storybook ou Ladle**
   Para documentar componentes e estados de tela.

4. **Playwright forte**
   Fluxos mínimos:

   * login;
   * Home por perfil;
   * Central de Vencimentos;
   * upload documento;
   * proposta pública;
   * assinatura pública;
   * solicitação de acesso;
   * sync Acessórias mockado.

5. **axe-core**
   Para acessibilidade.

6. **Sentry frontend**
   Para capturar erros reais por rota.

## Adicionar no backend

1. **BullMQ + Redis/Valkey**
   Para:

   * sync Acessórias;
   * backfill;
   * notificações;
   * scanner assíncrono;
   * exports;
   * OCR;
   * Integra-AI jobs.

2. **Redis-backed rate limit**
   Necessário para produção com mais de uma instância.

3. **OpenTelemetry**
   Para traces entre frontend, API, banco e integrações externas.

4. **Prometheus/Grafana**
   Para métricas:

   * latência;
   * erro por endpoint;
   * fila;
   * sync Acessórias;
   * uploads;
   * uso de banco;
   * rate limit;
   * jobs falhos.

5. **Sentry backend**
   Para exceptions com request id, usuário e módulo, sem dados sensíveis.

6. **S3/MinIO privado**
   Se o volume de documentos crescer, migrar storage local para objeto privado com criptografia server-side e política de acesso restrita.

7. **Secret manager**
   Pode ser Vault, Doppler, 1Password Secrets Automation, AWS/GCP/Azure Secrets Manager ou variáveis protegidas do ambiente EasyPanel, desde que haja rotação, controle de acesso e auditoria.

---

# 11. Roadmap recomendado para finalizar o MVP

## Fase 1 — Segurança emergencial

1. Rotacionar todos os segredos.
2. Remover `.env` dos pacotes.
3. Criar script de empacotamento seguro.
4. Confirmar `.gitignore` e `.dockerignore`.
5. Remover `.git`, `node_modules`, `dist`, logs e artefatos de teste dos ZIPs.
6. Desativar/trocar bootstrap admin.
7. Garantir Swagger off em produção.
8. Garantir CORS com origem exata de produção.

## Fase 2 — Build e deploy reproduzível

1. Build frontend em ambiente limpo.
2. Build backend em ambiente limpo.
3. Rodar migrations.
4. Rodar seeds/RBAC.
5. Rodar testes em CI.
6. Validar health/readiness.
7. Validar permissões de storage.
8. Testar backup/restore.

## Fase 3 — Homologação de fluxos reais

1. Login/logout/refresh.
2. Home colaborador.
3. Home gestor.
4. Home DEV/ADMIN.
5. Clientes e dashboard.
6. Central de Vencimentos.
7. Documentos.
8. Certificados.
9. Solicitação de acesso.
10. Onboarding.
11. Proposta pública.
12. Contrato público.
13. Notificações/Web Push.
14. Acessórias incremental.
15. Acessórias backfill.
16. Auditoria.

## Fase 4 — Correções funcionais

1. Corrigir links públicos.
2. Implementar ZapSign ou remover do escopo.
3. Refatorar jobs longos para fila.
4. Ajustar retorno de erro da Acessórias.
5. Revisar client scope.
6. Revisar labels/UX.
7. Quebrar páginas grandes.
8. Adicionar bot protection.

## Fase 5 — Produção controlada

1. Piloto com poucos usuários.
2. Monitoramento ativo.
3. Auditoria de logs.
4. Ajuste de índices.
5. Revisão de permissões.
6. Hardening final.
7. Treinamento de usuários.

---

# 12. Veredito final

O Portal Sama tem uma base técnica forte e coerente com uma aplicação empresarial: backend modular, autenticação bem pensada, RBAC, CSRF, auditoria, integração Acessórias persistida, upload protegido, certificados criptografados e frontend com rotas por perfil. A arquitetura escolhida — monólito modular com React/Nest/Prisma/MySQL — é adequada para o momento.

Como produto, a ideia está clara: ser o painel operacional central da empresa contábil, substituindo consultas dispersas, controles manuais e dependência direta da Acessórias. Como usuário, o portal já oferece os blocos principais: Home, clientes, vencimentos, documentos, certificados, legalização, contratos, propostas, onboarding, acessos, notificações e gestão.

O projeto ainda precisa de uma rodada séria de fechamento antes de produção. O maior problema não é falta de código; é **hardening, homologação de negócio e confiabilidade operacional**. A presença de `.env` real nos diretórios locais e o histórico de empacotamento precisam ser tratados com rotação formal de segredos e pacote seguro. A integração ZapSign precisa ser implementada ou removida do escopo. A Acessórias precisa ser homologada com dados reais. Os jobs longos precisam ir para fila. E os fluxos públicos precisam continuar protegidos contra abuso, apesar dos smokes públicos e autenticados terem passado.

Minha classificação atual:

```txt
Código backend:        bom / avançado para MVP
Código frontend:       buildado e testado, mas precisa modularização e polimento UX
Documentação:          boa, atualizada com nova rodada de testes, mas precisa checklist final
Segurança base:        boa, com smoke público/autenticado aprovado; hardening ainda pendente
Acessórias:            arquitetura boa, homologação com payload real ainda pendente
ZapSign:               não implementado ativamente
Prontidão para MVP:    mais próxima, após links públicos, segredos, scanner e escopo/RBAC
Prontidão produção:    ainda não
```

[1]: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html?utm_source=chatgpt.com "Secrets Management - OWASP Cheat Sheet Series"
[2]: https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html?utm_source=chatgpt.com "Cross-Site Request Forgery Prevention Cheat Sheet - OWASP"
[3]: https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/?utm_source=chatgpt.com "API1:2023 Broken Object Level Authorization - OWASP API Security Top 10"
[4]: https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html?utm_source=chatgpt.com "File Upload - OWASP Cheat Sheet Series"
