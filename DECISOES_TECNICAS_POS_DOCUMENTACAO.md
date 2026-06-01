# [CONCLUIDO] Decisões técnicas pós-documentação

## 1. Contexto

Após a documentação inicial do Portal Sama, a estratégia técnica foi ajustada conforme a preferência por **JavaScript/TypeScript**, o uso atual de **EasyPanel**, **MySQL** e **phpMyAdmin**, e a necessidade de uma arquitetura mais segura para documentos empresariais.

---

## 2. Decisão oficial de backend

A stack backend oficial passa a ser:

```txt
Node.js + TypeScript + NestJS
```

O NestJS foi escolhido por oferecer estrutura adequada para um sistema modular, com:

- módulos;
- controllers;
- services/providers;
- guards;
- pipes;
- interceptors;
- filters;
- decorators;
- integração com OpenAPI/Swagger;
- boa testabilidade;
- boa aderência a TypeScript.

### Por que não Express puro?

Express funcionaria, mas deixaria muita arquitetura por conta da equipe. Para o Portal Sama, isso é arriscado porque o sistema possui módulos sensíveis como:

- autenticação;
- documentos;
- certificados digitais;
- contratos;
- propostas;
- onboarding;
- solicitação de acesso;
- auditoria;
- Integra-AI;
- áreas de gerente, departamento, TI e DEV.

A recomendação é migrar para **TypeScript estruturado com NestJS**, não para JavaScript solto.

---

## 3. Decisão oficial de banco de dados

O banco principal será:

```txt
MySQL 8
```

Motivos:

- já está disponível na VPS/EasyPanel;
- já é usado no projeto atual;
- é compatível com Prisma ORM;
- atende ao modelo relacional do Portal Sama;
- reduz complexidade de migração;
- permite concentrar esforço na migração de PHP para TypeScript.

---

## 4. Relacional ou não relacional?

A decisão é usar **banco relacional como fonte principal da verdade**.

O Portal Sama possui dados altamente relacionados:

```txt
usuários
roles
permissões
clientes
colaboradores
departamentos
gerentes
documentos
certificados digitais
onboarding
propostas
contratos
assinaturas
solicitações de acesso
auditoria
```

Esses dados exigem:

- chaves estrangeiras;
- transações;
- constraints;
- índices;
- integridade referencial;
- rastreabilidade;
- relatórios;
- histórico de status.

Portanto, **MongoDB não deve ser usado como banco principal neste momento**.

---

## 5. Redis

Redis pode ser usado como apoio, não como banco principal.

Usos recomendados:

- rate limiting;
- cache;
- refresh tokens/sessões temporárias;
- filas com BullMQ;
- jobs de e-mail;
- alertas de vencimento de certificado;
- processamento assíncrono de upload;
- notificações.

---

## 6. ORM

O ORM recomendado é:

```txt
Prisma ORM
```

Motivos:

- integração forte com TypeScript;
- migrations versionadas;
- cliente tipado;
- menos SQL manual;
- melhor rastreabilidade;
- boa compatibilidade com MySQL.

O arquivo atual `api/db.php` deve ser gradualmente substituído por `apps/api/prisma/schema.prisma` e migrations.

---

## 7. Frontend

A stack frontend recomendada é:

```txt
React + TypeScript + Vite + Tailwind CSS + shadcn/ui
```

Ferramentas complementares:

- React Router;
- React Hook Form;
- Zod;
- TanStack Query;
- Axios;
- Zustand;
- Playwright.

---

## 8. Arquitetura

A arquitetura recomendada é:

```txt
Monólito modular
```

Não recomendamos microserviços no início. Antes disso, o Portal Sama precisa consolidar:

- autenticação;
- autorização;
- auditoria;
- documentos;
- storage privado;
- banco versionado;
- testes;
- deploy confiável;
- organização modular.

---

## 9. Estratégia de migração

Manter convivência temporária:

```txt
/api     -> PHP atual
/api-v2  -> NestJS novo
```

Ordem recomendada:

1. Autenticação.
2. Usuários, roles e permissões.
3. Auditoria.
4. Documentos e storage privado.
5. Certificados digitais.
6. Propostas.
7. Contratos.
8. Assinaturas.
9. Solicitações de acesso e TI.
10. Clientes e colaboradores.
11. Onboarding.
12. Manager.
13. Integra-AI.
14. Frontend React por grupos de telas.

---

## 10. Autenticação

A estratégia recomendada é:

```txt
Access token curto + refresh token em cookie HttpOnly/Secure/SameSite
```

Alternativa aceitável:

```txt
Sessão server-side com cookie HttpOnly/Secure/SameSite
```

Não armazenar refresh token em `localStorage` ou `sessionStorage`.

---

## 11. Autorização

Usar:

```txt
RBAC + permissões granulares + validação de escopo do recurso
```

Perfis iniciais:

```txt
ADMIN
MANAGER
CLIENT
DEPARTMENT
TI
ACCOUNTING
LEGALIZATION
AUDITOR
DEV
```

Permissões iniciais:

```txt
clients.read
clients.create
clients.update
clients.delete

documents.read
documents.upload
documents.download
documents.delete

certificates.read
certificates.download
certificates.manage

proposals.read
proposals.create
proposals.approve
proposals.reject

contracts.read
contracts.generate
contracts.sign

access_requests.read
access_requests.approve
access_requests.reject

audit.read
```

Regra obrigatória: possuir uma permissão não basta. O backend deve validar se o usuário pode acessar **aquele recurso específico**.

---

## 12. Armazenamento de arquivos

Arquivos não devem ser salvos como BLOB no MySQL.

### Correto

```txt
MySQL:
metadados do arquivo

Storage privado:
arquivo físico
```

Download sempre via backend:

```txt
GET /api-v2/documents/:id/download
```

O backend deve validar autenticação, autorização, vínculo com cliente/departamento e registrar auditoria.

---

## 13. Infraestrutura alvo no EasyPanel

```txt
portal-sama-web        -> React/Vite
portal-sama-api        -> NestJS
portal-sama-mysql      -> MySQL 8
portal-sama-redis      -> Redis opcional
portal-sama-phpmyadmin -> phpMyAdmin protegido
volume privado         -> documentos, certificados e contratos
```

---

## 14. Decisão final consolidada

```txt
Frontend:
React + TypeScript + Vite + Tailwind CSS + shadcn/ui

Backend:
Node.js + TypeScript + NestJS

Banco principal:
MySQL 8

ORM:
Prisma

Banco auxiliar:
Redis, se necessário

Banco não relacional:
Não usar no início

Arquivos:
Storage privado, não BLOB no banco

Infraestrutura:
EasyPanel + Docker

Segurança:
RBAC, cookies seguros, refresh token seguro, validação backend, auditoria e upload seguro
```
