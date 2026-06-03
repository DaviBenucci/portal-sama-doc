# [PARCIAL] Guia detalhado de implementação — migração para TypeScript, NestJS, React, MySQL e EasyPanel

## 1. Objetivo deste guia

Este documento descreve, em etapas práticas, como evoluir o Portal Sama do backend atual em PHP para uma nova arquitetura em **Node.js + TypeScript + NestJS**, mantendo **MySQL 8** como banco relacional principal no EasyPanel e adicionando gradualmente os frameworks recomendados no frontend e backend.

Este guia deve ser lido junto com:

- [`README_DOCUMENTACAO.md`](./README_DOCUMENTACAO.md)
- [`DECISOES_TECNICAS_POS_DOCUMENTACAO.md`](./DECISOES_TECNICAS_POS_DOCUMENTACAO.md)
- [`ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md`](./ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md)
- [`BANCO_DADOS_MYSQL_PRISMA.md`](./BANCO_DADOS_MYSQL_PRISMA.md)
- [`EASYPANEL_DEPLOY.md`](./EASYPANEL_DEPLOY.md)
- [`SEGURANCA.md`](./SEGURANCA.md)
- [`MAPEAMENTO_MIGRACAO_APIS.md`](./MAPEAMENTO_MIGRACAO_APIS.md)
- [`ROADMAP_REFATORACAO.md`](./ROADMAP_REFATORACAO.md)

A migração deve ser gradual. O sistema PHP atual deve continuar funcionando enquanto a nova API nasce em paralelo sob `/api-v2`.

---

## 2. Arquitetura final recomendada

```txt
Frontend atual temporário
HTML + CSS + JavaScript vanilla
        ↓
Consome /api PHP legado e /api-v2 NestJS novo
        ↓
Frontend alvo
React + TypeScript + Vite + Tailwind CSS + shadcn/ui
        ↓
API alvo
NestJS + TypeScript + Prisma
        ↓
MySQL 8 + Redis opcional + storage privado
```

### Decisão consolidada

| Camada | Decisão |
|---|---|
| Backend | Node.js + TypeScript + NestJS |
| Frontend | React + TypeScript + Vite |
| UI | Tailwind CSS + shadcn/ui |
| Formulários | React Hook Form + Zod |
| Consumo de API | Axios + TanStack Query |
| Estado global | Zustand |
| Banco principal | MySQL 8 |
| ORM | Prisma |
| Cache/filas | Redis, apenas quando necessário |
| Deploy | EasyPanel + Docker/Nixpacks + Nginx/reverse proxy |
| Arquivos sensíveis | Storage privado, não pasta pública |
| Segurança | RBAC, cookies seguros, CSRF, rate limiting, auditoria e validação server-side |

---

## 3. Etapa 0 — Preparação antes de alterar código

### 3.1 Criar backup completo

Antes de iniciar qualquer migração:

```bash
# Backup do código atual
zip -r portal-sama-backup-codigo.zip ./portal-sama

# Backup do banco atual pelo MySQL
mysqldump -u USUARIO -p BANCO_ATUAL > portal-sama-backup-banco.sql
```

No EasyPanel, também exporte backup do volume do MySQL e do volume de arquivos, se houver uploads salvos em disco.

### 3.2 Criar branch de migração

```bash
git checkout -b feat/migracao-nestjs-typescript
```

### 3.3 Corrigir itens sensíveis antes de versionar

A análise anterior identificou risco de arquivos que não devem ir para repositório/deploy, como `.env`, `.git`, dependências locais e ambientes virtuais.

Crie ou revise `.gitignore`:

```gitignore
.env
.env.*
!.env.example
node_modules/
dist/
build/
coverage/
.DS_Store
.vscode/
.idea/
.venv/
venv/
__pycache__/
*.log
storage/private/
uploads/private/
```

Referência de segurança: [`SEGURANCA.md`](./SEGURANCA.md).

### 3.4 Levantar inventário das APIs PHP atuais

Use o mapeamento já documentado em [`MAPEAMENTO_MIGRACAO_APIS.md`](./MAPEAMENTO_MIGRACAO_APIS.md). Os arquivos mais relevantes para migração são:

```txt
api/auth.php
api/storage.php
api/client_documents.php
api/certificados.php
api/legalizacao.php
api/onboarding.php
api/access_requests.php
api/manager_workspace.php
api/integra_ai.php
api/db.php
```

Esses arquivos devem ser tratados como **contratos legados**. A nova API NestJS deve reproduzir primeiro os fluxos essenciais, depois melhorar validações e segurança.

---

## 4. Etapa 1 — Criar estrutura de projeto moderna

### 4.1 Estrutura recomendada

```txt
portal-sama/
  apps/
    api/
    web/

  packages/
    shared/

  docs/
  docker/
  .env.example
  README.md
```

Se preferir começar mais simples:

```txt
portal-sama-api/
portal-sama-web/
```

Para o estágio atual, a estrutura separada `portal-sama-api` e `portal-sama-web` é mais simples de implantar no EasyPanel.

### 4.2 Criar backend NestJS

```bash
npm i -g @nestjs/cli
nest new portal-sama-api
cd portal-sama-api
```

Escolha `npm` ou `pnpm`. Se a equipe estiver começando, `npm` é suficiente.

### 4.3 Criar frontend React com Vite

```bash
npm create vite@latest portal-sama-web -- --template react-ts
cd portal-sama-web
npm install
```

---

## 5. Etapa 2 — Instalar dependências do backend

Dentro de `portal-sama-api`:

```bash
npm install @nestjs/config
npm install @nestjs/swagger swagger-ui-express
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install bcrypt cookie-parser helmet
npm install @nestjs/throttler
npm install prisma @prisma/client
npm install zod
npm install multer uuid
npm install nestjs-pino pino pino-pretty
npm install class-validator class-transformer
```

Dependências de desenvolvimento:

```bash
npm install -D @types/bcrypt
npm install -D @types/cookie-parser
npm install -D @types/multer
npm install -D @types/passport-jwt
npm install -D supertest @types/supertest
```

### Por que essas dependências entram no projeto

| Pacote | Uso no Portal Sama |
|---|---|
| `@nestjs/config` | Carregar e validar variáveis de ambiente |
| `@nestjs/swagger` | Documentar a API `/api-v2` |
| `@nestjs/jwt` | Emitir access tokens curtos |
| `cookie-parser` | Ler refresh token em cookie seguro |
| `helmet` | Headers básicos de segurança |
| `@nestjs/throttler` | Rate limit para login, uploads e endpoints críticos |
| `prisma` / `@prisma/client` | ORM tipado para MySQL |
| `zod` | Validação de ambiente e contratos de dados |
| `multer` | Recebimento de arquivos |
| `nestjs-pino` | Logs estruturados |
| `class-validator` | Validação de DTOs em controllers |

---

## 6. Etapa 3 — Configurar `main.ts` com segurança mínima

Arquivo alvo: `portal-sama-api/src/main.ts`

```ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import helmet from 'helmet';
import cookieParser from 'cookie-parser';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, { bufferLogs: true });

  app.setGlobalPrefix('api-v2');

  app.use(helmet());
  app.use(cookieParser());

  app.enableCors({
    origin: process.env.FRONTEND_URL,
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  });

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );

  if (process.env.NODE_ENV !== 'production') {
    const config = new DocumentBuilder()
      .setTitle('Portal Sama API')
      .setDescription('API v2 do Portal Sama')
      .setVersion('2.0.0')
      .addBearerAuth()
      .build();

    const document = SwaggerModule.createDocument(app, config);
    SwaggerModule.setup('docs-api', app, document);
  }

  await app.listen(process.env.PORT || 3000);
}

bootstrap();
```

### Observações importantes

- `app.setGlobalPrefix('api-v2')` permite coexistência com `/api` em PHP.
- `ValidationPipe` impede campos inesperados no payload.
- Swagger deve ser protegido ou desligado em produção.
- `enableCors` deve aceitar somente o domínio real do frontend.

Referências relacionadas:

- [`ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md`](./ARQUITETURA_ALVO_TYPESCRIPT_NESTJS.md)
- [`SEGURANCA.md`](./SEGURANCA.md)

---

## 7. Etapa 4 — Configurar variáveis de ambiente

Crie `portal-sama-api/.env.example`:

```env
NODE_ENV=development
PORT=3000
FRONTEND_URL=https://portal.exemplo.com.br

DATABASE_URL="mysql://portal_user:senha_forte@mysql:3306/portal_sama"

JWT_ACCESS_SECRET="trocar_em_producao"
JWT_REFRESH_SECRET="trocar_em_producao"
ACCESS_TOKEN_TTL="15m"
REFRESH_TOKEN_TTL_DAYS=7

COOKIE_DOMAIN=".exemplo.com.br"
COOKIE_SECURE=true
COOKIE_SAMESITE="lax"

PRIVATE_STORAGE_PATH="/app/storage/private"
MAX_UPLOAD_MB=20
```

Crie `portal-sama-api/src/config/env.schema.ts`:

```ts
import { z } from 'zod';

export const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
  PORT: z.coerce.number().default(3000),
  FRONTEND_URL: z.string().url(),
  DATABASE_URL: z.string().min(1),
  JWT_ACCESS_SECRET: z.string().min(32),
  JWT_REFRESH_SECRET: z.string().min(32),
  ACCESS_TOKEN_TTL: z.string().default('15m'),
  REFRESH_TOKEN_TTL_DAYS: z.coerce.number().default(7),
  PRIVATE_STORAGE_PATH: z.string().min(1),
  MAX_UPLOAD_MB: z.coerce.number().default(20),
});
```

No `app.module.ts`:

```ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { envSchema } from './config/env.schema';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validate: (config) => envSchema.parse(config),
    }),
  ],
})
export class AppModule {}
```

### Regra de segurança

O arquivo `.env` real nunca deve ir para o repositório. Mantenha apenas `.env.example`.

Referência: [`SEGURANCA.md`](./SEGURANCA.md).

---
## 8. Etapa 5 — Configurar MySQL com Prisma

### 8.1 Inicializar Prisma

Dentro de `portal-sama-api`:

```bash
npx prisma init
```

Arquivo alvo: `portal-sama-api/prisma/schema.prisma`

```prisma
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}
```

### 8.2 Criar modelos iniciais

Comece pelos modelos de autenticação, clientes, documentos e auditoria.

```prisma
enum UserStatus {
  ACTIVE
  INACTIVE
  BLOCKED
}

enum DocumentStatus {
  PENDING
  APPROVED
  REJECTED
  ARCHIVED
}

model User {
  id           String     @id @default(uuid())
  name         String
  email        String     @unique
  passwordHash String
  status       UserStatus @default(ACTIVE)

  roles        UserRole[]
  sessions     RefreshToken[]
  auditLogs    AuditLog[]

  createdAt    DateTime   @default(now())
  updatedAt    DateTime   @updatedAt
}

model Role {
  id          String           @id @default(uuid())
  name        String           @unique
  description String?

  users       UserRole[]
  permissions RolePermission[]

  createdAt   DateTime         @default(now())
  updatedAt   DateTime         @updatedAt
}

model Permission {
  id          String           @id @default(uuid())
  key         String           @unique
  description String?

  roles       RolePermission[]

  createdAt   DateTime         @default(now())
  updatedAt   DateTime         @updatedAt
}

model UserRole {
  userId String
  roleId String

  user   User @relation(fields: [userId], references: [id])
  role   Role @relation(fields: [roleId], references: [id])

  @@id([userId, roleId])
}

model RolePermission {
  roleId       String
  permissionId String

  role       Role       @relation(fields: [roleId], references: [id])
  permission Permission @relation(fields: [permissionId], references: [id])

  @@id([roleId, permissionId])
}

model RefreshToken {
  id        String   @id @default(uuid())
  userId    String
  tokenHash String
  revokedAt DateTime?
  expiresAt DateTime
  createdAt DateTime @default(now())

  user      User @relation(fields: [userId], references: [id])

  @@index([userId])
  @@index([expiresAt])
}

model Client {
  id          String   @id @default(uuid())
  razaoSocial String
  cnpj        String   @unique
  email       String?
  status      String   @default("ACTIVE")

  documents   Document[]

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}

model Document {
  id           String         @id @default(uuid())
  clientId     String
  type         String
  originalName String
  storageKey   String
  mimeType     String
  size         Int
  sha256Hash   String?
  status       DocumentStatus @default(PENDING)
  createdById  String?

  client       Client @relation(fields: [clientId], references: [id])

  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  @@index([clientId])
  @@index([type])
  @@index([status])
}

model AuditLog {
  id         String   @id @default(uuid())
  userId     String?
  action     String
  module     String
  entityType String?
  entityId   String?
  ipAddress  String?
  userAgent  String?
  metadata   Json?
  createdAt  DateTime @default(now())

  user       User? @relation(fields: [userId], references: [id])

  @@index([userId])
  @@index([module])
  @@index([entityType, entityId])
  @@index([createdAt])
}
```

### 8.3 Aplicar migrations

Desenvolvimento:

```bash
npx prisma migrate dev --name init
```

Produção no EasyPanel:

```bash
npx prisma migrate deploy
```

### 8.4 Criar PrismaService

Arquivo: `portal-sama-api/src/database/prisma.service.ts`

```ts
import { Injectable, OnModuleDestroy, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

Arquivo: `portal-sama-api/src/database/database.module.ts`

```ts
import { Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class DatabaseModule {}
```

Referência detalhada: [`BANCO_DADOS_MYSQL_PRISMA.md`](./BANCO_DADOS_MYSQL_PRISMA.md).

---

## 9. Etapa 6 — Implementar autenticação segura

### 9.1 Arquivos legados relacionados

Antes de migrar, consulte:

```txt
index.html
Login.js
auth.js
api/auth.php
api/storage.php
```

A documentação da página está em [`paginas/index.md`](./paginas/index.md).

### 9.2 Estrutura do módulo

```txt
src/modules/auth/
  auth.module.ts
  auth.controller.ts
  auth.service.ts
  dto/login.dto.ts
  strategies/jwt.strategy.ts
```

### 9.3 DTO de login

Arquivo: `src/modules/auth/dto/login.dto.ts`

```ts
import { IsEmail, IsString, MinLength } from 'class-validator';

export class LoginDto {
  @IsEmail()
  email!: string;

  @IsString()
  @MinLength(8)
  password!: string;
}
```

### 9.4 Controller de autenticação

Arquivo: `src/modules/auth/auth.controller.ts`

```ts
import { Body, Controller, Get, Post, Req, Res, UseGuards } from '@nestjs/common';
import type { Request, Response } from 'express';
import { AuthService } from './auth.service';
import { LoginDto } from './dto/login.dto';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';

@Controller('auth')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post('login')
  async login(@Body() dto: LoginDto, @Req() req: Request, @Res({ passthrough: true }) res: Response) {
    const result = await this.authService.login(dto, {
      ipAddress: req.ip,
      userAgent: req.headers['user-agent'],
    });

    res.cookie('refresh_token', result.refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'lax',
      path: '/api-v2/auth',
      maxAge: 7 * 24 * 60 * 60 * 1000,
    });

    return {
      accessToken: result.accessToken,
      user: result.user,
      permissions: result.permissions,
    };
  }

  @UseGuards(JwtAuthGuard)
  @Get('me')
  me(@Req() req: Request) {
    return req.user;
  }

  @Post('logout')
  async logout(@Req() req: Request, @Res({ passthrough: true }) res: Response) {
    await this.authService.logout(req.cookies?.refresh_token);
    res.clearCookie('refresh_token', { path: '/api-v2/auth' });
    return { success: true };
  }
}
```

### 9.5 Regras obrigatórias

- Não armazenar refresh token em `localStorage`.
- Armazenar refresh token em cookie `HttpOnly`, `Secure` e `SameSite`.
- Access token deve ter vida curta.
- Registrar auditoria de login, logout e falha de login.
- Aplicar rate limit em `/auth/login`.
- Bloquear temporariamente usuário com muitas tentativas inválidas.

### 9.6 Integração temporária com frontend antigo

Enquanto o frontend ainda estiver em HTML/JS, `Login.js` pode passar a chamar:

```js
fetch('/api-v2/auth/login', {
  method: 'POST',
  credentials: 'include',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email, password }),
});
```

---

## 10. Etapa 7 — Implementar RBAC e permissões

### 10.1 Perfis iniciais

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

### 10.2 Permissões iniciais

```txt
clients.read
clients.create
clients.update
clients.delete

documents.read
documents.upload
documents.download
documents.delete

proposals.read
proposals.create
proposals.approve
proposals.reject

contracts.read
contracts.generate
contracts.sign

certificates.read
certificates.download
certificates.manage

access_requests.read
access_requests.approve
access_requests.reject

audit.read
```

### 10.3 Decorator de permissões

Arquivo: `src/common/decorators/permissions.decorator.ts`

```ts
import { SetMetadata } from '@nestjs/common';

export const PERMISSIONS_KEY = 'permissions';
export const Permissions = (...permissions: string[]) => SetMetadata(PERMISSIONS_KEY, permissions);
```

### 10.4 Guard de permissões

Arquivo: `src/common/guards/permissions.guard.ts`

```ts
import { CanActivate, ExecutionContext, ForbiddenException, Injectable } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { PERMISSIONS_KEY } from '../decorators/permissions.decorator';

@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const required = this.reflector.getAllAndOverride<string[]>(PERMISSIONS_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!required?.length) return true;

    const request = context.switchToHttp().getRequest();
    const userPermissions: string[] = request.user?.permissions ?? [];

    const allowed = required.every((permission) => userPermissions.includes(permission));

    if (!allowed) {
      throw new ForbiddenException('Permissão insuficiente.');
    }

    return true;
  }
}
```

### 10.5 Uso em controller

```ts
@UseGuards(JwtAuthGuard, PermissionsGuard)
@Permissions('documents.download')
@Get(':id/download')
download(@Param('id') id: string, @CurrentUser() user: AuthenticatedUser) {
  return this.documentsService.download(id, user);
}
```

### 10.6 Regra mais importante

Permissão geral não basta. A API também precisa validar escopo:

```txt
Usuário cliente: só acessa documentos do próprio cliente.
Gerente: só acessa clientes vinculados.
Departamento: só acessa dados permitidos ao departamento.
Auditor: acessa logs, mas não necessariamente arquivos sensíveis.
```

Referência: [`SEGURANCA.md`](./SEGURANCA.md).

---
## 11. Etapa 8 — Implementar auditoria centralizada

### 11.1 Arquivos e páginas relacionados

```txt
Auditoria/autoria.html
api/storage.php
global.js
```

Documentação da tela: [`paginas/auditoria-autoria.md`](./paginas/auditoria-autoria.md).

### 11.2 Eventos que devem gerar auditoria

```txt
LOGIN_SUCCESS
LOGIN_FAILURE
LOGOUT
CLIENT_CREATED
CLIENT_UPDATED
DOCUMENT_UPLOADED
DOCUMENT_DOWNLOADED
DOCUMENT_DELETED
CERTIFICATE_VIEWED
CERTIFICATE_DOWNLOADED
PROPOSAL_CREATED
PROPOSAL_APPROVED
PROPOSAL_REJECTED
CONTRACT_GENERATED
CONTRACT_SIGNED
ACCESS_REQUEST_CREATED
ACCESS_REQUEST_APPROVED
ACCESS_REQUEST_REJECTED
USER_PERMISSION_CHANGED
MANAGER_TRANSFER_CREATED
```

### 11.3 Serviço de auditoria

Arquivo: `src/modules/audit/audit.service.ts`

```ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../../database/prisma.service';

export type AuditInput = {
  userId?: string;
  action: string;
  module: string;
  entityType?: string;
  entityId?: string;
  ipAddress?: string;
  userAgent?: string;
  metadata?: Record<string, unknown>;
};

@Injectable()
export class AuditService {
  constructor(private readonly prisma: PrismaService) {}

  async register(input: AuditInput) {
    await this.prisma.auditLog.create({
      data: {
        userId: input.userId,
        action: input.action,
        module: input.module,
        entityType: input.entityType,
        entityId: input.entityId,
        ipAddress: input.ipAddress,
        userAgent: input.userAgent,
        metadata: input.metadata ?? {},
      },
    });
  }
}
```

### 11.4 Cuidados

- Logs não devem armazenar senha, token, segredo, certificado bruto ou conteúdo completo de documentos.
- Logs devem armazenar identificadores, ação, módulo, IP, usuário, data e metadados controlados.
- Logs de auditoria não devem ser editáveis pela interface.
- Exportação de logs deve exigir permissão específica.

---

## 12. Etapa 9 — Implementar módulo de documentos e storage privado

### 12.1 Arquivos legados relacionados

```txt
api/client_documents.php
api/storage.php
Client/painel.js
Onboarding/documentos-cliente.html
Legalizacao/contrato.html
DptClient/certificados-digitais.html
```

Documentações relacionadas:

- [`paginas/client-painel.md`](./paginas/client-painel.md)
- [`paginas/onboarding-documentos-cliente.md`](./paginas/onboarding-documentos-cliente.md)
- [`paginas/legalizacao-contrato.md`](./paginas/legalizacao-contrato.md)
- [`paginas/dptclient-certificados-digitais.md`](./paginas/dptclient-certificados-digitais.md)

### 12.2 Estrutura do módulo

```txt
src/modules/documents/
  documents.module.ts
  documents.controller.ts
  documents.service.ts
  file-validation.service.ts
  private-storage.service.ts
  dto/upload-document.dto.ts
```

### 12.3 Controller de upload

```ts
import { Controller, Param, Post, UploadedFile, UseGuards, UseInterceptors } from '@nestjs/common';
import { FileInterceptor } from '@nestjs/platform-express';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { PermissionsGuard } from '../../common/guards/permissions.guard';
import { Permissions } from '../../common/decorators/permissions.decorator';
import { CurrentUser } from '../../common/decorators/current-user.decorator';
import { DocumentsService } from './documents.service';

@Controller('documents')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class DocumentsController {
  constructor(private readonly documentsService: DocumentsService) {}

  @Permissions('documents.upload')
  @Post('clients/:clientId/upload')
  @UseInterceptors(FileInterceptor('file'))
  upload(
    @Param('clientId') clientId: string,
    @UploadedFile() file: Express.Multer.File,
    @CurrentUser() user: AuthenticatedUser,
  ) {
    return this.documentsService.uploadForClient({ clientId, file, user });
  }
}
```

### 12.4 Validação de arquivo

```ts
import { BadRequestException, Injectable } from '@nestjs/common';

const ALLOWED_MIME_TYPES = [
  'application/pdf',
  'image/png',
  'image/jpeg',
  'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
];

const ALLOWED_EXTENSIONS = ['pdf', 'png', 'jpg', 'jpeg', 'docx'];

@Injectable()
export class FileValidationService {
  validate(file: Express.Multer.File) {
    if (!file) {
      throw new BadRequestException('Arquivo obrigatório.');
    }

    const extension = file.originalname.split('.').pop()?.toLowerCase();

    if (!extension || !ALLOWED_EXTENSIONS.includes(extension)) {
      throw new BadRequestException('Extensão de arquivo não permitida.');
    }

    if (!ALLOWED_MIME_TYPES.includes(file.mimetype)) {
      throw new BadRequestException('Tipo MIME não permitido.');
    }

    const maxBytes = Number(process.env.MAX_UPLOAD_MB ?? 20) * 1024 * 1024;

    if (file.size > maxBytes) {
      throw new BadRequestException('Arquivo excede o tamanho máximo permitido.');
    }
  }
}
```

### 12.5 Storage privado

```ts
import { Injectable } from '@nestjs/common';
import { randomUUID } from 'crypto';
import { mkdir, writeFile } from 'fs/promises';
import { join } from 'path';

@Injectable()
export class PrivateStorageService {
  async save(file: Express.Multer.File, module: string) {
    const basePath = process.env.PRIVATE_STORAGE_PATH!;
    const safeName = `${randomUUID()}`;
    const dir = join(basePath, module);
    const path = join(dir, safeName);

    await mkdir(dir, { recursive: true });
    await writeFile(path, file.buffer, { flag: 'wx' });

    return {
      storageKey: `${module}/${safeName}`,
      absolutePath: path,
    };
  }
}
```

### 12.6 Download protegido

Nunca exponha links diretos como:

```txt
/uploads/contrato.pdf
/public/documentos/cliente.pdf
```

Use endpoint protegido:

```txt
GET /api-v2/documents/:id/download
```

Fluxo obrigatório:

```txt
1. Validar autenticação.
2. Validar permissão `documents.download`.
3. Validar se o usuário pode acessar aquele documento específico.
4. Registrar auditoria.
5. Entregar arquivo por stream.
```

### 12.7 Antivírus

Para produção, adicionar ClamAV ou serviço equivalente:

```txt
Upload recebido
  ↓
Validação de extensão/MIME/tamanho
  ↓
Scan antivírus
  ↓
Storage privado
  ↓
Metadados no MySQL
```

---

## 13. Etapa 10 — Migrar certificados digitais

### 13.1 Arquivos relacionados

```txt
DptClient/certificados-digitais.html
api/certificados.php
```

Documentação: [`paginas/dptclient-certificados-digitais.md`](./paginas/dptclient-certificados-digitais.md).

### 13.2 Módulo alvo

```txt
src/modules/certificates/
  certificates.module.ts
  certificates.controller.ts
  certificates.service.ts
  dto/create-certificate.dto.ts
```

### 13.3 Endpoints alvo

```txt
GET    /api-v2/certificates
GET    /api-v2/certificates/:id
POST   /api-v2/certificates
PATCH  /api-v2/certificates/:id
DELETE /api-v2/certificates/:id
GET    /api-v2/certificates/:id/download
```

### 13.4 Regras obrigatórias

- Nunca retornar senha de certificado pela API.
- Criptografar metadados sensíveis quando necessário.
- Auditar todo acesso, download, edição e exclusão.
- Bloquear download para perfis sem permissão explícita.
- Criar alerta de vencimento.
- Validar propriedade por cliente/departamento.

Exemplo de proteção:

```ts
@UseGuards(JwtAuthGuard, PermissionsGuard)
@Permissions('certificates.download')
@Get(':id/download')
downloadCertificate(@Param('id') id: string, @CurrentUser() user: AuthenticatedUser) {
  return this.certificatesService.download(id, user);
}
```

---

## 14. Etapa 11 — Migrar legalização, propostas, contratos e assinaturas

### 14.1 Arquivos relacionados

```txt
Legalizacao/legalizacao.html
Legalizacao/proposta.html
Legalizacao/contrato.html
Legalizacao/assinatura.html
api/legalizacao.php
api/legal_doc_service.php
api/public_token_lib.php
```

Documentações:

- [`paginas/legalizacao-legalizacao.md`](./paginas/legalizacao-legalizacao.md)
- [`paginas/legalizacao-proposta.md`](./paginas/legalizacao-proposta.md)
- [`paginas/legalizacao-contrato.md`](./paginas/legalizacao-contrato.md)
- [`paginas/legalizacao-assinatura.md`](./paginas/legalizacao-assinatura.md)

### 14.2 Módulos alvo

```txt
LegalizationModule
ProposalsModule
ContractsModule
SignaturesModule
DocumentsModule
AuditModule
```

### 14.3 Controle de status por enum

```ts
export enum ProposalStatus {
  DRAFT = 'DRAFT',
  PENDING_APPROVAL = 'PENDING_APPROVAL',
  APPROVED = 'APPROVED',
  REJECTED = 'REJECTED',
  CONVERTED_TO_CONTRACT = 'CONVERTED_TO_CONTRACT',
}

export enum ContractStatus {
  DRAFT = 'DRAFT',
  GENERATED = 'GENERATED',
  SENT_TO_SIGNATURE = 'SENT_TO_SIGNATURE',
  SIGNED = 'SIGNED',
  CANCELLED = 'CANCELLED',
}
```

### 14.4 Regra de negócio essencial

```txt
Proposta aprovada pode gerar contrato.
Contrato gerado pode ser enviado para assinatura.
Contrato assinado não deve aceitar edição livre.
Assinatura deve gerar trilha de auditoria.
Links públicos de assinatura devem usar token opaco com expiração.
```

### 14.5 Token público seguro para assinatura

Nunca use ID incremental no link público. Use token opaco e salve apenas hash no banco.

```ts
import { randomBytes, createHash } from 'crypto';

export function createPublicToken() {
  const token = randomBytes(32).toString('hex');
  const tokenHash = createHash('sha256').update(token).digest('hex');

  return { token, tokenHash };
}
```

Endpoint público:

```txt
GET  /api-v2/public/signatures/:token
POST /api-v2/public/signatures/:token/sign
```

Mesmo sendo público, deve validar:

- token existente;
- token não expirado;
- token não revogado;
- contrato correto;
- limite de tentativas;
- auditoria de acesso.

---

## 15. Etapa 12 — Migrar onboarding

### 15.1 Arquivos relacionados

```txt
Onboarding/processo.html
Onboarding/entrada-cliente.html
Onboarding/documentos-cliente.html
Onboarding/proposta-cliente.html
api/onboarding.php
```

Documentações:

- [`paginas/onboarding-processo.md`](./paginas/onboarding-processo.md)
- [`paginas/onboarding-entrada-cliente.md`](./paginas/onboarding-entrada-cliente.md)
- [`paginas/onboarding-documentos-cliente.md`](./paginas/onboarding-documentos-cliente.md)
- [`paginas/onboarding-proposta-cliente.md`](./paginas/onboarding-proposta-cliente.md)

### 15.2 Endpoints alvo

```txt
GET    /api-v2/onboarding/processes
GET    /api-v2/onboarding/processes/:id
POST   /api-v2/onboarding/processes
PATCH  /api-v2/onboarding/processes/:id
PATCH  /api-v2/onboarding/processes/:id/status
POST   /api-v2/onboarding/processes/:id/documents
GET    /api-v2/onboarding/processes/:id/timeline
```

### 15.3 Regras

- Processo deve ter etapas controladas.
- Documentos obrigatórios devem ser configuráveis por tipo de cliente/processo.
- Upload público por token precisa expirar.
- Cliente só acessa seu próprio processo.
- Toda mudança de status deve gerar histórico.
- Mudança de status deve gerar auditoria.

---

## 16. Etapa 13 — Migrar solicitações de acesso e TI

### 16.1 Arquivos relacionados

```txt
SolicitacaoAcesso/solicitacao-acesso.html
TI/acesso-ti.html
api/access_requests.php
SolicitacaoAcesso/enviar_acesso.php
```

Documentações:

- [`paginas/solicitacao-acesso.md`](./paginas/solicitacao-acesso.md)
- [`paginas/ti-acesso-ti.md`](./paginas/ti-acesso-ti.md)

### 16.2 Endpoints alvo

```txt
POST   /api-v2/access-requests
GET    /api-v2/access-requests
GET    /api-v2/access-requests/:id
POST   /api-v2/access-requests/:id/approve
POST   /api-v2/access-requests/:id/reject
PATCH  /api-v2/access-requests/:id/status
```

### 16.3 Regras

- Solicitação deve gerar protocolo.
- TI aprova, rejeita ou solicita ajuste.
- Solicitante acompanha status.
- Anexos seguem a política de upload seguro.
- Toda aprovação/rejeição gera auditoria.
- E-mails/notificações devem ser disparados por fila quando houver Redis/BullMQ.

---
## 17. Etapa 14 — Criar frontend React com TypeScript

### 17.1 Instalar dependências

Dentro de `portal-sama-web`:

```bash
npm install react-router-dom
npm install axios
npm install @tanstack/react-query
npm install zustand
npm install react-hook-form zod @hookform/resolvers
npm install lucide-react
npm install class-variance-authority clsx tailwind-merge
```

Configurar Tailwind CSS e shadcn/ui conforme o guia oficial do shadcn para Vite.

### 17.2 Estrutura recomendada

```txt
src/
  app/
    router.tsx
    providers.tsx

  components/
    layout/
      AppLayout.tsx
      Sidebar.tsx
      Header.tsx
    ui/
    common/
      DataTable.tsx
      StatusBadge.tsx
      PermissionGate.tsx
      UploadField.tsx
      ConfirmDialog.tsx

  pages/
    auth/LoginPage.tsx
    home/HomePage.tsx
    clients/ClientsPage.tsx
    clients/ClientDashboardPage.tsx
    onboarding/OnboardingProcessesPage.tsx
    legalization/LegalizationPage.tsx
    legalization/ProposalPage.tsx
    legalization/ContractPage.tsx
    certificates/CertificatesPage.tsx
    access-requests/AccessRequestPage.tsx
    ti/TiAccessPage.tsx
    audit/AuditPage.tsx

  services/
    api.ts
    auth.service.ts
    clients.service.ts
    documents.service.ts
    certificates.service.ts
    proposals.service.ts
    contracts.service.ts
    audit.service.ts

  schemas/
  stores/
  hooks/
  types/
```

### 17.3 Axios centralizado

Arquivo: `src/services/api.ts`

```ts
import axios from 'axios';

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  withCredentials: true,
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      window.location.href = '/login';
    }

    return Promise.reject(error);
  },
);
```

### 17.4 Provider do TanStack Query

Arquivo: `src/app/providers.tsx`

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactNode, useState } from 'react';

export function AppProviders({ children }: { children: ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());

  return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>;
}
```

### 17.5 Rotas principais

Arquivo: `src/app/router.tsx`

```tsx
import { createBrowserRouter } from 'react-router-dom';
import { AppLayout } from '@/components/layout/AppLayout';
import { LoginPage } from '@/pages/auth/LoginPage';
import { HomePage } from '@/pages/home/HomePage';
import { ClientsPage } from '@/pages/clients/ClientsPage';
import { ClientDashboardPage } from '@/pages/clients/ClientDashboardPage';
import { CertificatesPage } from '@/pages/certificates/CertificatesPage';
import { AuditPage } from '@/pages/audit/AuditPage';

export const router = createBrowserRouter([
  { path: '/login', element: <LoginPage /> },
  {
    path: '/',
    element: <AppLayout />,
    children: [
      { path: 'home', element: <HomePage /> },
      { path: 'clientes', element: <ClientsPage /> },
      { path: 'clientes/:clientId/painel', element: <ClientDashboardPage /> },
      { path: 'certificados-digitais', element: <CertificatesPage /> },
      { path: 'auditoria', element: <AuditPage /> },
    ],
  },
]);
```

### 17.6 Formulário com React Hook Form + Zod

Exemplo para cadastro de cliente.

Arquivo: `src/schemas/create-client.schema.ts`

```ts
import { z } from 'zod';

export const createClientSchema = z.object({
  razaoSocial: z.string().min(3, 'Informe a razão social.'),
  cnpj: z.string().min(14, 'Informe um CNPJ válido.'),
  email: z.string().email('E-mail inválido.').optional().or(z.literal('')),
});

export type CreateClientInput = z.infer<typeof createClientSchema>;
```

Uso na tela:

```tsx
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { createClientSchema, CreateClientInput } from '@/schemas/create-client.schema';

export function CreateClientForm() {
  const form = useForm<CreateClientInput>({
    resolver: zodResolver(createClientSchema),
    defaultValues: {
      razaoSocial: '',
      cnpj: '',
      email: '',
    },
  });

  async function onSubmit(data: CreateClientInput) {
    // Chamar clientService.create(data)
  }

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      {/* Inputs padronizados com shadcn/ui */}
    </form>
  );
}
```

### 17.7 PermissionGate

Arquivo: `src/components/common/PermissionGate.tsx`

```tsx
import { ReactNode } from 'react';
import { useAuthStore } from '@/stores/auth.store';

export function PermissionGate({
  permission,
  children,
  fallback = null,
}: {
  permission: string;
  children: ReactNode;
  fallback?: ReactNode;
}) {
  const permissions = useAuthStore((state) => state.permissions);

  if (!permissions.includes(permission)) {
    return <>{fallback}</>;
  }

  return <>{children}</>;
}
```

### Atenção

`PermissionGate` melhora experiência visual, mas **não substitui autorização no backend**. O backend sempre deve validar permissão e escopo.

---

## 18. Etapa 15 — Mapa de migração das páginas para React

| HTML atual | Rota React alvo | Componente alvo | Módulos NestJS |
|---|---|---|---|
| `index.html` | `/login` | `LoginPage.tsx` | `AuthModule` |
| `HOME/Inicio.html` | `/home` | `HomePage.tsx` | `AuthModule`, `UsersModule` |
| `Client/clientes.html` | `/clientes` | `ClientsPage.tsx` | `ClientsModule` |
| `Client/painel.html` | `/clientes/:clientId/painel` | `ClientDashboardPage.tsx` | `ClientsModule`, `DocumentsModule` |
| `DptClient/certificados-digitais.html` | `/certificados-digitais` | `CertificatesPage.tsx` | `CertificatesModule`, `AuditModule` |
| `Legalizacao/legalizacao.html` | `/legalizacao` | `LegalizationPage.tsx` | `LegalizationModule` |
| `Legalizacao/proposta.html` | `/legalizacao/propostas/:id` | `ProposalPage.tsx` | `ProposalsModule` |
| `Legalizacao/contrato.html` | `/legalizacao/contratos/:id` | `ContractPage.tsx` | `ContractsModule` |
| `Legalizacao/assinatura.html` | `/assinatura/:token` | `PublicSignaturePage.tsx` | `SignaturesModule` |
| `Onboarding/processo.html` | `/onboarding/processos` | `OnboardingProcessesPage.tsx` | `OnboardingModule` |
| `Onboarding/documentos-cliente.html` | `/onboarding/publico/documentos/:token` | `PublicDocumentsPage.tsx` | `OnboardingModule`, `DocumentsModule` |
| `SolicitacaoAcesso/solicitacao-acesso.html` | `/solicitacao-acesso` | `AccessRequestPage.tsx` | `AccessRequestsModule` |
| `TI/acesso-ti.html` | `/ti/acessos` | `TiAccessPage.tsx` | `TiModule`, `AccessRequestsModule` |
| `Auditoria/autoria.html` | `/auditoria` | `AuditPage.tsx` | `AuditModule` |

Cada documentação individual em `docs/paginas/` recebeu uma seção adicional chamada **Atualização de stack alvo — TypeScript/NestJS** com a rota React, módulos NestJS e arquivos atuais relacionados.

---

## 19. Etapa 16 — Configurar EasyPanel

### 19.1 Serviços recomendados

```txt
portal-sama-web      React + Vite
portal-sama-api      NestJS
portal-sama-mysql    MySQL 8
portal-sama-redis    Redis opcional
portal-sama-storage  volume privado
phpmyadmin           apenas administração controlada
```

### 19.2 Variáveis do backend no EasyPanel

```env
NODE_ENV=production
PORT=3000
FRONTEND_URL=https://portal.exemplo.com.br
DATABASE_URL=mysql://portal_user:senha@portal-sama-mysql:3306/portal_sama
PRIVATE_STORAGE_PATH=/app/storage/private
JWT_ACCESS_SECRET=valor_longo_seguro
JWT_REFRESH_SECRET=valor_longo_seguro
```

### 19.3 Comando de build da API

```bash
npm ci
npx prisma generate
npm run build
npx prisma migrate deploy
npm run start:prod
```

### 19.4 Dockerfile básico da API

```dockerfile
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:22-alpine AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npx prisma generate
RUN npm run build

FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY --from=build /app/prisma ./prisma
COPY package*.json ./
EXPOSE 3000
CMD ["sh", "-c", "npx prisma migrate deploy && node dist/main.js"]
```

Referência: [`EASYPANEL_DEPLOY.md`](./EASYPANEL_DEPLOY.md).

---

## 20. Etapa 17 — Testes obrigatórios

### 20.1 Testes de backend

Exemplo com Supertest:

```ts
import request from 'supertest';

describe('Auth', () => {
  it('deve rejeitar login com senha inválida', async () => {
    await request(app.getHttpServer())
      .post('/api-v2/auth/login')
      .send({ email: 'usuario@empresa.com', password: 'errada' })
      .expect(401);
  });
});
```

### 20.2 Testes de permissão

```txt
Usuário não autenticado não acessa `/api-v2/clients`.
Usuário CLIENT não acessa documento de outro cliente.
Usuário sem `certificates.download` não baixa certificado.
Usuário sem `audit.read` não acessa logs.
```

### 20.3 Testes de upload

```txt
Arquivo `.php` deve ser recusado.
Arquivo com MIME inválido deve ser recusado.
Arquivo acima do limite deve ser recusado.
Upload válido deve salvar metadados no MySQL.
Upload válido deve salvar arquivo no storage privado.
Upload válido deve gerar log de auditoria.
```

### 20.4 Testes E2E com Playwright

Fluxos prioritários:

```txt
Login e logout
Criação de cliente
Upload de documento
Download protegido
Criação de proposta
Geração de contrato
Assinatura por token público
Solicitação de acesso
Aprovação pela TI
Consulta de auditoria
```

---

## 21. Etapa 18 — CI/CD e qualidade

### 21.1 GitHub Actions básico

Arquivo: `.github/workflows/ci.yml`

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  api:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: npm ci
        working-directory: portal-sama-api
      - run: npx prisma generate
        working-directory: portal-sama-api
      - run: npm run lint
        working-directory: portal-sama-api
      - run: npm test
        working-directory: portal-sama-api
      - run: npm run build
        working-directory: portal-sama-api

  web:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: npm ci
        working-directory: portal-sama-web
      - run: npm run lint
        working-directory: portal-sama-web
      - run: npm run build
        working-directory: portal-sama-web
```

### 21.2 Qualidade mínima

```txt
ESLint
Prettier
TypeScript strict
Testes unitários
Testes de API
Testes E2E para fluxos críticos
Análise de dependências vulneráveis
Build obrigatório antes de deploy
```

---

## 22. Etapa 19 — Checklist antes de produção

```md
- [ ] `.env` real fora do repositório.
- [ ] MySQL sem porta pública exposta, exceto necessidade controlada.
- [ ] phpMyAdmin protegido por senha forte, IP allowlist ou autenticação extra.
- [ ] `DATABASE_URL` usando usuário com permissões mínimas necessárias.
- [ ] `npx prisma migrate deploy` validado em homologação.
- [ ] CORS limitado ao domínio do frontend.
- [ ] Cookies com `HttpOnly`, `Secure` e `SameSite`.
- [ ] Tokens não armazenados em `localStorage`.
- [ ] RBAC funcionando no backend.
- [ ] Validação de escopo implementada contra IDOR.
- [ ] Uploads validados por extensão, MIME, tamanho e conteúdo.
- [ ] Storage privado configurado fora da pasta pública.
- [ ] Downloads passando pelo backend.
- [ ] Auditoria para login, upload, download, contratos, propostas e certificados.
- [ ] Swagger desligado ou protegido em produção.
- [ ] Rate limiting em login, upload e endpoints públicos.
- [ ] Logs sem senhas, tokens ou dados sensíveis brutos.
- [ ] Backup automatizado do MySQL e storage.
- [ ] Testes de autenticação, autorização e upload executados no CI.
```

---

## 23. Ordem recomendada de implementação

```txt
1. Preparar repositório, backups e `.gitignore`.
2. Criar `portal-sama-api` em NestJS.
3. Configurar MySQL + Prisma.
4. Criar autenticação e sessão/token seguro.
5. Criar usuários, roles e permissões.
6. Criar auditoria.
7. Migrar documentos e storage privado.
8. Migrar certificados digitais.
9. Migrar legalização, propostas, contratos e assinaturas.
10. Migrar onboarding.
11. Migrar solicitações de acesso e TI.
12. Criar `portal-sama-web` em React + TypeScript.
13. Migrar telas críticas para React.
14. Adicionar testes, CI/CD e observabilidade.
15. Desativar endpoints PHP legados conforme forem substituídos.
```

---

## 24. Conclusão

A migração recomendada não é uma troca simples de linguagem. É uma evolução arquitetural: sair de endpoints PHP dispersos para uma API TypeScript organizada por módulos, com autenticação robusta, autorização por permissão e escopo, auditoria, storage privado e contratos de API documentados.

O Portal Sama deve manter MySQL como banco relacional principal, usar Prisma para migrations e acesso tipado, adicionar Redis apenas quando houver necessidade real de cache/fila/sessão temporária, e migrar o frontend para React + TypeScript de forma gradual.

A primeira entrega prática deve ser uma API `/api-v2` com autenticação, permissões, auditoria e documentos seguros. Depois, os módulos sensíveis devem ser migrados por prioridade de risco.
