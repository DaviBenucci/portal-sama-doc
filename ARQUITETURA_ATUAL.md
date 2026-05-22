# Arquitetura Atual do Portal Sama

## Descrição da arquitetura atual

O Portal Sama é uma aplicação web tradicional composta por páginas HTML estáticas, CSS modular por pasta, JavaScript vanilla e endpoints PHP na pasta `/api`. O frontend consome APIs por `fetch`, com autenticação central em `auth.js` e persistência genérica em `/api/storage.php` para vários domínios.

## Organização de pastas

```txt
/
  index.html
  auth.js
  global.js
  global.css
  Login.js
  Login.CSS
  api/
  HOME/
  Client/
  DptClient/
  Manager/
  Legalizacao/
  Onboarding/
  Contabil/
  Auditoria/
  Depto/
  DEV/
  SolicitacaoAcesso/
  TI/
  data/
  logs/
  secure/
  docker/
  scripts/
```

## Relação entre HTML, CSS, JavaScript e PHP

- Cada módulo possui HTML próprio e, em muitos casos, CSS/JS dedicado.
- `global.js` e `global.css` padronizam navegação, notificações, layout e estado global.
- `auth.js` executa gate de sessão no carregamento de páginas internas.
- `DEV/dev-data.js` centraliza parte relevante do modelo de dados no frontend e usa `/api/storage.php`.
- APIs PHP recebem `action` por query string e retornam JSON ou arquivo.

## Relação entre frontend e backend

O frontend utiliza `fetch` diretamente. Não há camada formal de client API tipado. O padrão predominante é:

```txt
HTML -> JS da tela/global -> fetch('/api/*.php?action=...') -> PHP -> MySQL/KV/storage privado -> JSON/arquivo
```

Esse padrão funciona para evolução rápida, mas dificulta tipagem, testes, documentação OpenAPI e políticas por recurso.

## Organização da pasta `/api`

A pasta `/api` concentra endpoints por domínio:

- `storage.php`: autenticação, KV/MySQL estruturado, auditoria, presença, admin users, backup/restore.
- `auth.php`: funções de sessão, senha, rate limit, CSRF e autorização básica.
- `security_lib.php`: rate limiting, IP, signing key e utilitários de segurança.
- `db.php`: conexão PDO, schema e migrations imperativas.
- `client_documents.php`: documentos de clientes, upload/download e pendências.
- `certificados.php`: certificados digitais, senhas criptografadas e arquivos `.p12/.pfx`.
- `legalizacao.php`: contratos, templates, assinatura pública e PDF.
- `propostas.php`: propostas e fluxo público do cliente.
- `onboarding.php`: esteira de entrada, links públicos, documentos e propostas.
- `manager_workspace.php`: gestor, calendário, histórico e transferências.
- `fiscal_workspace.php`: planilhas de departamento.
- `integra_ai.php`: processamento contábil e exportação TXT.
- `access_requests.php`: consulta/decisão de solicitações de acesso.
- `notifications.php` e `notifications_stream.php`: notificações.

## Fluxo geral de autenticação identificado

1. Usuário acessa `index.html`.
2. `Login.js` envia `POST /api/storage.php?action=auth_login`.
3. `api/storage.php` delega a autenticação para `api/auth.php`.
4. `api/auth.php` cria sessão `SAMASESSID`, usa cookie `HttpOnly`, `SameSite=Lax` e `Secure` quando HTTPS/proxy indica segurança.
5. `auth.js` consulta `GET /api/storage.php?action=auth_session_status` nas páginas internas.
6. Mutations devem enviar `X-CSRF-Token` obtido da sessão.

## Fluxo geral de dados

- Dados principais ficam no MySQL, com algumas estruturas ainda encapsuladas em KV/JSON.
- Uploads sensíveis usam `data/private/uploads/...` e Dockerfile cria diretórios privados.
- Logs e backups ficam em `logs/`, `audits/` e áreas privadas, com `.htaccess` negando acesso em Apache.
- Notificações são registradas em tabela própria e podem ser consumidas por polling/SSE.

## Pontos fortes da arquitetura atual

- Separação inicial por módulos de negócio.
- Autenticação server-side já existente.
- CSRF implementado em APIs críticas por `sama_require_csrf()`.
- Uso de PDO/prepared statements em diversas consultas.
- Headers de segurança configurados em `.htaccess`, incluindo CSP, `X-Frame-Options`, `nosniff` e HSTS.
- Dockerfile com ClamAV e limites de PHP.
- Storage privado para uploads sensíveis.
- Eventos/auditoria começam a ser publicados por `event_hub.php`.

## Limitações da arquitetura atual

- JavaScript vanilla acoplado a DOM e regras de negócio.
- Arquivos JS e PHP muito extensos.
- APIs action-based em vez de REST/documentadas.
- Ausência de TypeScript, contratos formais e testes automatizados.
- Dependência de CDNs externos em páginas críticas.
- `.env` presente no pacote analisado, com chaves de banco, SMTP, OCR, Zapsign, OpenAI, tokens e chaves de criptografia. Valores não foram expostos nesta documentação.
- Autorização por perfil/departamento ainda precisa ser verificada action por action.

## Riscos técnicos

- Risco de IDOR em endpoints que recebem `client_id`, `doc_id`, `id`, `username` ou `token`.
- Risco de XSS onde dados retornados de API são renderizados com `innerHTML`.
- Risco de CSRF em mutations não cobertas por `sama_require_csrf()`.
- Risco de vazamento se `.env`, `data`, `logs` ou `audits` forem servidos por Nginx sem regras equivalentes ao `.htaccess`.
- Risco de regras de negócio aplicadas apenas no frontend.

## Recomendações de arquitetura

- Migrar gradualmente o backend para NestJS + TypeScript, preservando endpoints antigos em PHP com uma camada de compatibilidade temporária em `/api-v2`.
- Criar controllers por domínio: `AuthController`, `ClientController`, `DocumentController`, `CertificateController`, `LegalizationController`, `OnboardingController`, `ManagerWorkspaceController`, `AuditController`.
- Implementar Services e Repositories para remover SQL e regra de negócio dos arquivos de rota.
- Criar Policies/Gates para RBAC por perfil/departamento/recurso.
- Criar Form Requests para validação server-side.
- Documentar APIs com OpenAPI/Swagger.
- Criar frontend novo em React + TypeScript + Vite, migrando primeiro páginas críticas.
- Criar Design System e componentes compartilhados.
- Adotar testes automatizados, CI/CD, análise estática e observabilidade.

---

## Atualização pós-documentação — arquitetura alvo definida

Após a documentação inicial, foi definida uma nova direção técnica para evolução do Portal Sama.

### Decisão atual

```txt
Backend alvo:
Node.js + TypeScript + NestJS

Frontend alvo:
React + TypeScript + Vite + Tailwind CSS + shadcn/ui

Banco principal:
MySQL 8

ORM:
Prisma

Deploy:
EasyPanel + Docker/Nixpacks + Nginx/reverse proxy
```

### Impacto na arquitetura atual

A arquitetura atual em HTML, CSS, JavaScript vanilla e PHP não deve ser removida de uma só vez. A recomendação é manter os endpoints legados em `/api` e criar uma nova API em `/api-v2`.

```txt
/api      → PHP legado mantido temporariamente
/api-v2   → NestJS novo
```

### Motivo da decisão

- A equipe deseja manter JavaScript/TypeScript como linguagem principal.
- NestJS oferece estrutura mais adequada que Express puro para módulos, autenticação, autorização, validação e documentação de API.
- MySQL deve ser mantido por compatibilidade com a infraestrutura atual em EasyPanel e phpMyAdmin.
- Prisma será usado para tipagem, migrations e padronização do acesso ao banco.
- React + TypeScript permitirá substituir gradualmente as páginas HTML por telas componentizadas.

### Documentos complementares

- [`DECISOES_TECNICAS_POS_DOCUMENTACAO.md`](./DECISOES_TECNICAS_POS_DOCUMENTACAO.md)
- [`ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md`](./ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md)
- [`BANCO_DADOS_MYSQL_PRISMA.md`](./BANCO_DADOS_MYSQL_PRISMA.md)
- [`EASYPANEL_DEPLOY.md`](./EASYPANEL_DEPLOY.md)
- [`GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`](./GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md)
