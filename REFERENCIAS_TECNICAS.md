# Referências técnicas oficiais

Este arquivo reúne referências oficiais para as tecnologias recomendadas na nova arquitetura do Portal Sama.

## Backend

- NestJS Documentation — referência principal para criação de APIs Node.js com TypeScript, módulos, controllers, services, guards, pipes e OpenAPI.
  - https://docs.nestjs.com/
- NestJS OpenAPI/Swagger — documentação para geração de contrato da API.
  - https://docs.nestjs.com/openapi/introduction
- NestJS Security — referência para autenticação, guards e proteção de rotas.
  - https://docs.nestjs.com/security/authentication
- NestJS Rate Limiting — referência para uso de throttling em endpoints críticos.
  - https://docs.nestjs.com/security/rate-limiting

## Banco de dados e ORM

- Prisma ORM com MySQL — referência para conexão do Prisma com MySQL/MariaDB.
  - https://www.prisma.io/docs/orm/core-concepts/supported-databases/mysql
- Prisma Migrate — referência para criação, versionamento e aplicação de migrations.
  - https://www.prisma.io/docs/cli/migrate
- Prisma Migrate Deploy — comando recomendado para aplicar migrations em produção.
  - https://www.prisma.io/docs/cli/migrate/deploy

## Frontend

- Vite Guide — referência para criação e build de frontend moderno com React + TypeScript.
  - https://vite.dev/guide/
- Vite Env Variables — referência para variáveis `VITE_*` no frontend.
  - https://vite.dev/guide/env-and-mode
- shadcn/ui com Vite — referência para instalar componentes de UI no frontend React.
  - https://ui.shadcn.com/docs/installation/vite
- TanStack Query — referência para gerenciamento de cache, queries e mutações.
  - https://tanstack.com/query/latest/docs/framework/react/overview
- React Hook Form — referência para gerenciamento de formulários.
  - https://react-hook-form.com/get-started
- Zod — referência para validação e schemas TypeScript-first.
  - https://zod.dev/
- Zustand — referência para estado global leve.
  - https://zustand.docs.pmnd.rs/

## Segurança

- OWASP Top 10 — referência para riscos comuns em aplicações web.
  - https://owasp.org/www-project-top-ten/
- OWASP Injection — referência para riscos de injection e necessidade de consultas seguras.
  - https://owasp.org/Top10/A03_2021-Injection/
- OWASP Cheat Sheet Series — referência prática para autenticação, sessão, upload e CSRF.
  - https://cheatsheetseries.owasp.org/

## Infraestrutura e EasyPanel

- EasyPanel Services — referência geral dos serviços suportados pelo EasyPanel.
  - https://easypanel.io/docs/services
- EasyPanel MySQL Service — referência do serviço MySQL no EasyPanel.
  - https://easypanel.io/docs/services/mysql
- EasyPanel phpMyAdmin Template — referência para uso controlado do phpMyAdmin.
  - https://easypanel.io/docs/templates/phpmyadmin
- EasyPanel Redis Service — referência para Redis, caso seja usado em filas/cache.
  - https://easypanel.io/docs/services/redis

## Arquivos internos desta documentação

- [`README_DOCUMENTACAO.md`](./README_DOCUMENTACAO.md)
- [`DECISOES_TECNICAS_POS_DOCUMENTACAO.md`](./DECISOES_TECNICAS_POS_DOCUMENTACAO.md)
- [`ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md`](./ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md)
- [`BANCO_DADOS_MYSQL_PRISMA.md`](./BANCO_DADOS_MYSQL_PRISMA.md)
- [`EASYPANEL_DEPLOY.md`](./EASYPANEL_DEPLOY.md)
- [`SEGURANCA.md`](./SEGURANCA.md)
- [`MAPEAMENTO_MIGRACAO_APIS.md`](./MAPEAMENTO_MIGRACAO_APIS.md)
- [`GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md`](./GUIA_IMPLEMENTACAO_TYPESCRIPT_NESTJS.md)
- [`ROADMAP_REFATORACAO.md`](./ROADMAP_REFATORACAO.md)
