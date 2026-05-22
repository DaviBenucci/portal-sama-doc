# Recomendações técnicas atualizadas para evolução do Portal Sama

## 1. Decisão técnica atualizada

A recomendação de backend foi atualizada para seguir a preferência por JavaScript/TypeScript.

```txt
Frontend: React + TypeScript + Vite + Tailwind CSS + shadcn/ui
Backend: Node.js + TypeScript + NestJS
Banco: MySQL 8
ORM: Prisma ORM
Infraestrutura: EasyPanel + Docker
Cache/filas: Redis opcional
```

---

## 2. Frontend

### React + TypeScript

Aplicar na substituição gradual das páginas HTML atuais por telas componentizadas.

Prioridade inicial:

1. `index.html` -> `/login`.
2. `Client/painel.html` -> `/clientes/:id/painel`.
3. `DptClient/certificados-digitais.html` -> `/certificados-digitais`.
4. `Legalizacao/contrato.html` -> `/legalizacao/contratos/:id`.
5. `Onboarding/documentos-cliente.html` -> `/onboarding/publico/documentos/:token`.
6. `Auditoria/autoria.html` -> `/auditoria`.
7. `TI/acesso-ti.html` -> `/ti/acessos`.

Benefícios:

- componentização;
- redução de código duplicado;
- tipagem de dados;
- melhor manutenção;
- base para testes E2E;
- menor risco em formulários críticos.

---

### Vite

Usar Vite como ferramenta de build do frontend.

Motivos:

- configuração simples;
- build rápido;
- boa integração com React e TypeScript;
- adequado para SPA administrativa;
- fácil deploy com Nginx no EasyPanel.

---

### Tailwind CSS + shadcn/ui

Usar Tailwind para padronizar estilos e shadcn/ui para componentes reutilizáveis.

Aplicar em:

- tabelas;
- cards;
- formulários;
- modais;
- uploads;
- badges de status;
- filtros;
- sidebar;
- header;
- telas de auditoria;
- telas de certificados, propostas e contratos.

Componentes sugeridos:

```txt
Button
Input
Select
Dialog
DropdownMenu
Tabs
Table
Card
Badge
Textarea
Form
Toast
Alert
Calendar
Command
```

---

### React Hook Form + Zod

Aplicar em formulários:

- login;
- cadastro de cliente;
- cadastro de colaborador;
- solicitação de acesso;
- upload de documentos;
- cadastro de certificado digital;
- proposta;
- contrato;
- onboarding.

Exemplo:

```ts
import { z } from 'zod';

export const createClientSchema = z.object({
  razaoSocial: z.string().min(3, 'Informe a razão social'),
  cnpj: z.string().min(14, 'Informe um CNPJ válido'),
  email: z.string().email('E-mail inválido').optional(),
  responsavelId: z.string().uuid('Responsável inválido'),
});

export type CreateClientInput = z.infer<typeof createClientSchema>;
```

A validação frontend melhora a experiência, mas a segurança real deve ser validada novamente no backend.

---

### TanStack Query

Usar para dados vindos da API:

- clientes;
- colaboradores;
- documentos;
- certificados;
- onboarding;
- legalização;
- propostas;
- contratos;
- solicitações de acesso;
- auditoria.

Exemplo:

```ts
const { data, isLoading, error } = useQuery({
  queryKey: ['clients'],
  queryFn: clientService.list,
});
```

Benefícios:

- cache;
- loading padronizado;
- tratamento de erro;
- refetch;
- invalidação após mutações;
- redução de chamadas duplicadas.

---

### Axios

Criar client HTTP centralizado.

Arquivo sugerido: `apps/web/src/services/api.ts`

```ts
import axios from 'axios';

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  withCredentials: true,
});

api.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      window.location.href = '/login';
    }

    return Promise.reject(error);
  }
);
```

---

### Zustand

Usar para estado global leve:

- usuário autenticado;
- permissões;
- layout;
- filtros;
- preferências de UI.

Não armazenar refresh token ou segredos no Zustand.

---

## 3. Backend

### NestJS

Substituir gradualmente endpoints PHP por módulos NestJS.

Estrutura recomendada:

```txt
apps/api/src/modules/
  auth/
  users/
  roles/
  permissions/
  clients/
  documents/
  certificates/
  onboarding/
  legalization/
  proposals/
  contracts/
  signatures/
  access-requests/
  managers/
  accounting/
  audit/
```

Aplicação direta:

| Arquivo PHP atual | Destino NestJS |
|---|---|
| `api/auth.php` | `AuthModule`, `UsersModule` |
| `api/storage.php` | `AuthModule`, `UsersModule`, `ClientsModule`, `AuditModule` |
| `api/client_documents.php` | `DocumentsModule` |
| `api/certificados.php` | `CertificatesModule` |
| `api/legalizacao.php` | `LegalizationModule`, `ContractsModule`, `SignaturesModule` |
| `api/propostas.php` | `ProposalsModule` |
| `api/onboarding.php` | `OnboardingModule`, `DocumentsModule` |
| `api/access_requests.php` | `AccessRequestsModule` |
| `api/manager_workspace.php` | `ManagersModule`, `TransfersModule`, `CalendarModule` |
| `api/integra_ai.php` | `AccountingModule`, `IntegraAiModule` |
| `api/db.php` | `prisma/schema.prisma` e migrations |

---

### Prisma ORM

Usar para substituir SQL manual e criação imperativa de schema.

Benefícios:

- migrations versionadas;
- modelos tipados;
- menor risco de SQL injection por SQL manual;
- melhor rastreabilidade;
- compatibilidade com MySQL.

---

### Swagger/OpenAPI

Documentar a API em `/api-v2/docs`, protegendo ou desabilitando em produção.

Prioridade de documentação:

- `AuthModule`;
- `DocumentsModule`;
- `CertificatesModule`;
- `ContractsModule`;
- `AccessRequestsModule`;
- `AuditModule`.

---

### BullMQ + Redis

Usar quando houver processamento assíncrono real:

- varredura de upload com ClamAV;
- envio de e-mail;
- notificações;
- alertas de vencimento de certificado;
- integração contábil;
- geração de PDF/contrato.

---

## 4. Segurança

### Autenticação

Recomendação:

```txt
Access token curto + refresh token em cookie HttpOnly/Secure/SameSite
```

Evitar refresh token em `localStorage` ou `sessionStorage`.

---

### Autorização

Usar RBAC com permissões granulares e validação de escopo.

Exemplos de permissões:

```txt
documents.download
certificates.manage
contracts.sign
proposals.approve
access_requests.approve
audit.read
```

Regra crítica:

```txt
Ter a permissão documents.download não basta.
O backend também precisa validar se o usuário pode baixar aquele documento específico.
```

---

### Upload seguro

Obrigatório para:

- `api/client_documents.php` -> `DocumentsModule`;
- `api/certificados.php` -> `CertificatesModule`;
- `api/onboarding.php` -> `OnboardingModule`;
- `api/legalizacao.php` -> `ContractsModule`.

Requisitos:

- limite de tamanho;
- allowlist de extensão;
- validação de MIME;
- validação de assinatura mágica;
- renomeação segura;
- hash SHA-256;
- storage privado;
- ClamAV;
- auditoria;
- download via endpoint protegido.

---

### Auditoria

Registrar:

- login;
- logout;
- falha de login;
- upload;
- download;
- alteração de cliente;
- criação de proposta;
- aprovação/rejeição de proposta;
- geração de contrato;
- assinatura;
- acesso a certificado;
- solicitação/aprovação/rejeição de acesso;
- alteração de permissões.

---

## 5. Infraestrutura

### EasyPanel

Criar serviços separados:

```txt
portal-sama-web
portal-sama-api
portal-sama-mysql
portal-sama-redis
portal-sama-phpmyadmin
```

### Storage privado

Usar volume privado para documentos sensíveis.

Futuro possível:

- MinIO;
- Wasabi;
- Backblaze B2;
- AWS S3;
- DigitalOcean Spaces.

---

## 6. Testes

### Backend

- Jest para services e guards.
- Supertest para endpoints.
- Testes de permissão.
- Testes de upload.
- Testes de auditoria.
- Testes de autenticação.

### Frontend

- Playwright para E2E.
- Testes de login.
- Testes de formulário.
- Testes de upload.
- Testes de rotas protegidas.
- Testes de permissões visuais.

---

## 7. Stack final recomendada

```txt
Frontend:
React + TypeScript + Vite + React Router + Tailwind CSS + shadcn/ui + React Hook Form + Zod + TanStack Query + Axios + Zustand + Playwright

Backend:
Node.js + TypeScript + NestJS + Prisma ORM + MySQL 8 + Swagger/OpenAPI + Jest + Supertest

Segurança:
Cookies HttpOnly/Secure/SameSite + RBAC + Guards + validação server-side + rate limiting + Helmet + CORS restrito + storage privado + ClamAV + auditoria

Infraestrutura:
EasyPanel + Docker + MySQL + Redis opcional + phpMyAdmin protegido + backups automatizados
```
