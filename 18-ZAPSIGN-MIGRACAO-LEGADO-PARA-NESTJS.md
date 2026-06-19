# ZapSign — migração do legado PHP para NestJS

Atualizado em: 2026-06-19

## 1. Objetivo

Migrar a integração ZapSign que existia no legado PHP para a nova arquitetura `portal-sama-api` em NestJS, preservando o comportamento funcional já validado e adicionando controles modernos de segurança, auditoria, testes e idempotência.

A nova API já possui módulo de contratos, renderização/importação de PDF, assinatura pública interna por token e armazenamento privado. Porém, a integração ZapSign real não está implementada como provider de assinatura. O legado deve ser usado como referência para essa migração.

## 2. O que o legado já fazia corretamente

Arquivos de referência:

```txt
portal-sama/api/zapsign.php
portal-sama/api/legalizacao.php
portal-sama/Legalizacao/contrato.html
portal-sama/Legalizacao/assinatura.html
```

Comportamentos existentes no legado:

- sanitização do token da ZapSign;
- suporte a ambiente produção e sandbox;
- chamada HTTP com `Authorization: Bearer <token>`;
- criação de documento por `base64_pdf` em `/docs/`;
- envio de signatários com nome, e-mail, telefone, CPF, modo de autenticação e travas de edição;
- suporte a `send_automatic_email` desligado;
- consulta de documento por `doc_token`;
- extração do primeiro signatário;
- armazenamento de `doc_token`, `status`, `original_file`, `signed_file`, `signer_token`, `sign_url`, `signer_status`, `signed_at` e `signers`;
- sincronização de status antes de listar/detalhar contrato;
- marcação do contrato como concluído quando ZapSign retorna assinado;
- bloqueio de assinatura interna quando o contrato usa ZapSign.

Esses comportamentos devem ser portados para NestJS com tipagem e testes.

## 3. Decisão arquitetural

A ZapSign deve ser tratada como **provider de assinatura externa**, não como tela pública interna.

A API deve suportar dois providers:

```txt
INTERNAL  -> assinatura pública dentro do Portal Sama
ZAPSIGN   -> assinatura externa via ZapSign
```

O frontend não deve saber como autenticar na ZapSign. O frontend apenas solicita envio/sincronização ao backend e recebe metadados seguros.

## 4. Alterações no banco de dados

### 4.1. Enums sugeridos

```prisma
enum ContractSignatureProvider {
  INTERNAL
  ZAPSIGN
}

enum ContractSignatureEnv {
  BR
  SANDBOX
}

enum ContractExternalSignatureStatus {
  CREATED
  WAITING_SIGNATURE
  SIGNED
  CANCELLED
  ERROR
  UNKNOWN
}
```

### 4.2. Modelo recomendado: envelope de assinatura

Evitar jogar tudo em `Contract.metadata`. Dados críticos de assinatura precisam de índice, auditoria e evolução.

```prisma
model ContractSignatureEnvelope {
  id                  String   @id @default(uuid())
  contractId          String   @unique @map("contract_id")
  provider            ContractSignatureProvider
  env                 ContractSignatureEnv @default(BR)
  externalDocToken    String?  @unique @map("external_doc_token")
  externalStatus      String?  @map("external_status") @db.VarChar(80)
  normalizedStatus    ContractExternalSignatureStatus @default(UNKNOWN) @map("normalized_status")
  signUrl             String?  @map("sign_url") @db.Text
  signerToken         String?  @map("signer_token") @db.VarChar(191)
  signerStatus        String?  @map("signer_status") @db.VarChar(80)
  originalFileUrl     String?  @map("original_file_url") @db.Text
  signedFileUrl       String?  @map("signed_file_url") @db.Text
  lastSyncedAt        DateTime? @map("last_synced_at")
  signedAt            DateTime? @map("signed_at")
  lastErrorCode       String?  @map("last_error_code") @db.VarChar(80)
  lastErrorMessage    String?  @map("last_error_message") @db.Text
  rawPayload          Json?    @map("raw_payload")
  createdById         String?  @map("created_by_id")
  updatedById         String?  @map("updated_by_id")
  createdAt           DateTime @default(now()) @map("created_at")
  updatedAt           DateTime @updatedAt @map("updated_at")

  @@index([provider, normalizedStatus])
  @@index([externalDocToken])
  @@index([signedAt])
  @@map("contract_signature_envelopes")
}
```

### 4.3. Modelo recomendado: signatários

```prisma
model ContractSigner {
  id                 String   @id @default(uuid())
  contractId         String   @map("contract_id")
  envelopeId         String?  @map("envelope_id")
  name               String   @db.VarChar(191)
  email              String?  @db.VarChar(191)
  cpf                String?  @db.VarChar(20)
  phoneCountry       String?  @map("phone_country") @db.VarChar(8)
  phoneNumber        String?  @map("phone_number") @db.VarChar(32)
  authMode           String?  @map("auth_mode") @db.VarChar(80)
  requireCpf         Boolean  @default(false) @map("require_cpf")
  externalSignerToken String? @map("external_signer_token") @db.VarChar(191)
  externalStatus     String? @map("external_status") @db.VarChar(80)
  signUrl            String? @map("sign_url") @db.Text
  signedAt           DateTime? @map("signed_at")
  createdAt          DateTime @default(now()) @map("created_at")
  updatedAt          DateTime @updatedAt @map("updated_at")

  @@index([contractId])
  @@index([envelopeId])
  @@index([email])
  @@index([cpf])
  @@map("contract_signers")
}
```

### 4.4. Campo de provider no contrato

Adicionar ao `Contract`:

```prisma
signatureProvider ContractSignatureProvider @default(INTERNAL) @map("signature_provider")
signatureEnv      ContractSignatureEnv      @default(BR) @map("signature_env")
```

Ou, se a equipe preferir menor alteração inicial, manter provider no envelope e refletir no DTO. A recomendação é persistir no `Contract` para consulta rápida e filtros.

## 5. Variáveis de ambiente

Adicionar ao `.env.example` sem valores reais:

```txt
ZAPSIGN_API_TOKEN=
ZAPSIGN_API_TOKEN_SANDBOX=
ZAPSIGN_DEFAULT_ENV=br
ZAPSIGN_WEBHOOK_SECRET=
ZAPSIGN_TIMEOUT_SEC=30
ZAPSIGN_SEND_AUTOMATIC_EMAIL=false
```

Regras:

- token real nunca deve ser commitado;
- token nunca deve aparecer em logs;
- `.env` real nunca deve entrar em pacote compartilhável;
- qualquer token que tenha sido empacotado deve ser rotacionado;
- produção deve usar secret manager ou variável do provedor de deploy.

## 6. Estrutura de código recomendada

```txt
src/modules/contracts/providers/zapsign/
  zapsign.module.ts
  zapsign.client.ts
  zapsign.service.ts
  zapsign.types.ts
  zapsign.mapper.ts
  zapsign-webhook.controller.ts
  zapsign-signature-verifier.service.ts
```

Responsabilidades:

| Arquivo | Responsabilidade |
|---|---|
| `zapsign.client.ts` | HTTP client isolado, headers, timeout, parse de erro. |
| `zapsign.service.ts` | Casos de uso: criar doc, consultar doc, sincronizar, normalizar status. |
| `zapsign.mapper.ts` | Mapear DTO interno para payload ZapSign e resposta ZapSign para domínio. |
| `zapsign.types.ts` | Tipos da API externa e tipos internos. |
| `zapsign-webhook.controller.ts` | Receber webhooks, validar assinatura/secret e sincronizar. |
| `zapsign-signature-verifier.service.ts` | Validar origem do webhook quando suportado. |

## 7. DTOs necessários

### 7.1. `SendSignatureDto`

Expandir o DTO atual para aceitar provider:

```ts
provider?: 'internal' | 'zapsign';
signatureProvider?: 'internal' | 'zapsign';
signatureEnv?: 'br' | 'sandbox';
signers?: Array<{
  name?: string;
  nome?: string;
  email?: string;
  cpf?: string;
  phone?: string;
  phoneCountry?: string;
  phoneNumber?: string;
  authMode?: 'email' | 'sms' | 'whatsapp' | 'certificadoDigital';
  requireCpf?: boolean;
}>;
sendAutomaticEmail?: boolean;
```

Regra de compatibilidade: aceitar `nome` e `name`, mas normalizar internamente para `name`.

### 7.2. `SyncZapsignContractDto`

```ts
force?: boolean;
```

Endpoint sugerido:

```txt
POST /api-v2/contracts/:id/zapsign/sync
```

## 8. Payload para criação de documento

Mapeamento baseado no legado:

```json
{
  "name": "Contrato - Cliente X",
  "base64_pdf": "...",
  "signers": [
    {
      "name": "Nome do signatario",
      "email": "email@cliente.com",
      "phone_country": "55",
      "phone_number": "11999999999",
      "auth_mode": "certificadoDigital",
      "require_cpf": true,
      "cpf": "00000000000",
      "lock_name": true,
      "lock_email": true,
      "lock_phone": true
    }
  ],
  "send_automatic_email": false
}
```

Regras:

- remover prefixo `data:application/pdf;base64,` se existir;
- validar que o PDF existe no storage ou gerar antes do envio;
- nunca aceitar HTML cru do cliente público para renderizar contrato;
- signatário deve ter nome e pelo menos e-mail ou telefone conforme modo de autenticação;
- CPF deve ser sanitizado;
- `certificadoDigital` deve exigir CPF.

## 9. Fluxo de envio ZapSign

```txt
1. Usuário autenticado cria ou edita contrato.
2. Usuário gera snapshot HTML do contrato.
3. Backend renderiza/importa PDF e salva em storage privado.
4. Usuário aciona "enviar para assinatura" com provider=zapsign.
5. Backend valida CSRF, permissão `contracts.sign`, contrato e signatários.
6. Backend verifica se já existe envelope ZapSign para o contrato.
7. Se não existir, backend lê PDF do storage e converte para base64.
8. Backend cria documento na ZapSign.
9. Backend persiste envelope, signatários e status.
10. Backend atualiza contrato para `WAITING_SIGNATURE`.
11. Backend registra auditoria.
12. Frontend exibe status e `sign_url` permitido.
```

Regra de idempotência: se já existir `externalDocToken`, não criar novo documento automaticamente. A ação deve sincronizar o existente, exceto se houver endpoint explícito de cancelamento/reenvio.

## 10. Fluxo de sincronização

```txt
1. Usuário autenticado solicita sincronização ou webhook chega.
2. Backend busca envelope por contrato/doc token.
3. Backend chama ZapSign `GET /docs/{doc_token}/`.
4. Backend normaliza status.
5. Backend atualiza envelope e signatários.
6. Se status normalizado for `SIGNED`, atualizar contrato para `SIGNED` e `signedAt`.
7. Backend registra auditoria.
8. Frontend invalida query de contrato/listagem.
```

Mapeamento de status sugerido:

| ZapSign | Interno |
|---|---|
| signed | SIGNED |
| completed | SIGNED |
| pending | WAITING_SIGNATURE |
| waiting_signature | WAITING_SIGNATURE |
| cancelled | CANCELLED |
| refused | CANCELLED |
| erro desconhecido | UNKNOWN |

## 11. Fluxo público de assinatura

Quando o contrato usa `INTERNAL`:

```txt
GET  /api-v2/public/signatures/:token
POST /api-v2/public/signatures/:token/sign
```

Quando o contrato usa `ZAPSIGN`:

- `GET /public/signatures/:token` pode retornar um payload público mínimo com `provider=zapsign`, título e `sign_url` se permitido;
- `POST /public/signatures/:token/sign` deve responder erro controlado, pois a assinatura deve ocorrer na ZapSign;
- o frontend deve exibir orientação clara ou redirecionar conforme decisão do produto.

Nunca aceitar assinatura desenhada/interna para contrato ZapSign.

## 12. Webhook ZapSign

Endpoint sugerido:

```txt
POST /api-v2/webhooks/zapsign
```

Regras:

- rota pública, mas protegida por secret/header/assinatura quando a ZapSign suportar;
- aplicar rate limit;
- validar payload mínimo;
- nunca confiar apenas no payload do webhook para marcar como assinado;
- após webhook, buscar o documento na ZapSign por `doc_token` e só então atualizar estado;
- responder rápido e processar idempotentemente.

## 13. Auditoria obrigatória

Eventos mínimos:

```txt
contracts.zapsign.create.started
contracts.zapsign.create.succeeded
contracts.zapsign.create.failed
contracts.zapsign.sync.started
contracts.zapsign.sync.succeeded
contracts.zapsign.sync.failed
contracts.zapsign.webhook.received
contracts.zapsign.webhook.rejected
contracts.zapsign.signed
```

Auditar:

- usuário;
- contrato;
- cliente;
- provider;
- ambiente;
- doc token mascarado quando necessário;
- status anterior e novo;
- IP/user-agent quando aplicável;
- erro sanitizado sem token.

## 14. Segurança

| Risco | Controle obrigatório |
|---|---|
| Token ZapSign exposto | secret manager, `.env` fora do pacote, rotação. |
| Criação duplicada de documentos | envelope único por contrato e idempotência. |
| Assinar contrato errado | snapshot HTML/PDF imutável no envio. |
| Alterar contrato após envio | bloquear edição sensível após `WAITING_SIGNATURE`, exceto cancelamento/versionamento. |
| Webhook falso | validação de secret + consulta ativa à ZapSign. |
| Vazamento de PDF assinado | salvar/download via backend protegido, não URL aberta. |
| Logs com PII | mascarar CPF, e-mail parcial e tokens. |
| Ambiente sandbox em produção | restringir sandbox a admin/dev ou ambiente de homologação. |

## 15. Testes obrigatórios

### 15.1. Unitários

- sanitização de token;
- base URL por ambiente;
- montagem de payload;
- normalização de signatário;
- normalização de status;
- erro HTTP 401/403/500 da ZapSign;
- idempotência quando envelope já existe.

### 15.2. Integração com mock

Usar mock HTTP para:

- `POST /docs/` com sucesso;
- `POST /docs/` com erro;
- `GET /docs/{token}/` pendente;
- `GET /docs/{token}/` assinado;
- webhook válido;
- webhook inválido.

### 15.3. E2E backend

Cenários:

1. Criar contrato.
2. Renderizar PDF.
3. Enviar via ZapSign sandbox/mock.
4. Garantir contrato em `WAITING_SIGNATURE`.
5. Sincronizar como assinado.
6. Garantir contrato em `SIGNED`.
7. Tentar assinatura interna em contrato ZapSign e receber erro.
8. Garantir auditoria.

## 16. Critérios de aceite

A migração ZapSign estará pronta quando:

- existir provider `ZAPSIGN` no domínio;
- envio criar documento real em sandbox/mock;
- status sincronizar e concluir contrato;
- webhook for validado e idempotente;
- frontend exibir provider/status/link corretamente;
- assinatura interna estiver bloqueada para contrato ZapSign;
- tokens não aparecerem em logs, docs ou pacotes;
- testes unitários, integração e E2E estiverem verdes;
- documentação do `.env.example` estiver atualizada sem secrets reais.
