# Contrato API para Frontend

Status: congelado em 2026-06-22 para validacao frontend local.

Base URL da API: `/api-v2`.

## Regras gerais

- Rotas autenticadas exigem `Authorization: Bearer <access_token>`.
- Mutacoes autenticadas exigem CSRF emitido por `GET /api-v2/auth/csrf`.
- Respostas de contrato preservam campos camelCase e aliases snake_case/legados enquanto o frontend final e migrado.
- Tokens publicos brutos so podem aparecer na resposta imediata de envio interno; nao devem ir para `localStorage`, `sessionStorage`, logs ou auditoria.
- Contratos com `signatureProvider = ZAPSIGN` usam assinatura externa; o POST publico interno de assinatura deve ser bloqueado.

## Enums usados pelo frontend

| Campo | Valores |
| --- | --- |
| `status` | `DRAFT`, `GENERATED`, `WAITING_SIGNATURE`, `SIGNED`, `CANCELLED`, `ARCHIVED` |
| `signatureProvider` | `INTERNAL`, `ZAPSIGN` |
| `signatureEnv` | `BR`, `SANDBOX` |
| `zapsign.normalizedStatus` | `CREATED`, `WAITING_SIGNATURE`, `SIGNED`, `CANCELLED`, `ERROR`, `UNKNOWN` |

## Endpoints de contratos

| Metodo | Endpoint | Permissao | CSRF | Contrato |
| --- | --- | --- | --- | --- |
| GET | `/contracts` | `contracts.read` | Nao | Query: `clientId`, `status`, `provider`/`signatureProvider`, `search`, `includeArchived`, `skip`, `take`/`limit`. Retorna `{ ok, data, items, meta, generated_at }`. |
| POST | `/contracts` | `contracts.generate` | Sim | Body: `clientId`, `title`, `recipientName`, `recipientDocument`, `signatureProvider`, `signatureEnv`, `htmlContent`, `headerHtml`, `footerHtml`, `metadata`. Retorna detalhe. |
| GET | `/contracts/:id` | `contracts.read` | Nao | Retorna `{ ok, data, contract, generated_at }`. |
| PATCH | `/contracts/:id` | `contracts.generate` | Sim | Mesmo corpo parcial de criacao. Retorna detalhe. |
| POST | `/contracts/:id/generate` | `contracts.generate` | Sim | Body: `htmlContent`, `headerHtml`, `footerHtml`. Salva snapshot HTML validado e retorna detalhe. |
| POST | `/contracts/:id/render-pdf` | `contracts.generate` | Sim | Body opcional: `filename`. Renderiza PDF via Legal Doc Service quando habilitado e retorna detalhe com `pdfDownloadUrl`. |
| POST | `/contracts/:id/pdf` | `contracts.generate` | Sim | Multipart `file`. Importa PDF validado para storage privado. |
| GET | `/contracts/:id/pdf` | `contracts.read` | Nao | Download protegido do PDF, com `Cache-Control: no-store`. |
| POST | `/contracts/:id/send-signature` | `contracts.sign` | Sim | Envia assinatura interna ou ZapSign. Para `provider=zapsign`, tambem exige `contracts.zapsign.send`. |
| POST | `/contracts/:id/zapsign/sync` | `contracts.zapsign.sync` | Sim | Sincroniza status com ZapSign e retorna detalhe. Throttle: 10/min. |

## Resposta de contrato

Campos principais em `ContractItem`:

- Identificacao: `id`, `contract_id`, `clientId`, `client_id`, `title`, `titulo`.
- Pessoa/empresa: `recipientName`, `recipient_name`, `recipientDocument`, `recipient_document`.
- Status: `status`, `contract_status`, `signatureProvider`, `signature_provider`, `signatureEnv`, `signature_env`.
- Documento: `htmlContent`, `html_content`, `headerHtml`, `header_html`, `footerHtml`, `footer_html`, `hasPdf`, `has_pdf`, `pdfDownloadUrl`, `pdf_download_url`.
- Assinatura interna: `publicTokenId`, `public_token_id`, `signatureName`, `signature_name`, `signatureDocument`, `signature_document`, `signatureHash`, `signature_hash`.
- Assinatura externa: `signatureEnvelope`, `signature_envelope`, `zapsign`.

## Envio de assinatura

Body aceito por `POST /contracts/:id/send-signature`:

```json
{
  "provider": "internal",
  "signatureEnv": "br",
  "expiresInDays": 7,
  "sendAutomaticEmail": false,
  "signers": [
    {
      "name": "Cliente",
      "email": "cliente@example.com",
      "cpf": "00000000000",
      "phone": "+5511999999999",
      "authMode": "email",
      "requireCpf": true
    }
  ]
}
```

Resposta para `INTERNAL`:

- `token` e `raw_token`: token bruto somente nesta resposta.
- `link`: URL React `/assinatura/:token`.
- `publicApiUrl`: URL API publica correspondente.
- `signatureProvider`: `INTERNAL`.

Resposta para `ZAPSIGN`:

- `token`, `raw_token` e `publicApiUrl`: `null`.
- `link`, `signUrl` e `sign_url`: URL externa de assinatura.
- `signatureProvider`: `ZAPSIGN`.
- `contract.zapsign`: envelope externo com `signers`, `normalizedStatus`, URLs externas e token externo mascarado.

## Rotas publicas

| Metodo | Endpoint | Autenticacao | Contrato |
| --- | --- | --- | --- |
| GET | `/public/signatures/:token` | Publica por token | Retorna dados minimos do contrato. Para `ZAPSIGN`, retorna `signUrl` para a UI encaminhar ao provedor externo. |
| POST | `/public/signatures/:token/sign` | Publica por token | Apenas `INTERNAL`. Body: `name`, `document`, `signatureData`. Para `ZAPSIGN`, retorna erro `CONTRACT_ZAPSIGN_INTERNAL_SIGNATURE_FORBIDDEN`. |
| POST | `/webhooks/zapsign` | Publica com segredo | Webhook ZapSign. Header aceito: `x-zapsign-webhook-secret`, `x-zapsign-signature` ou `x-webhook-secret`. Body deve conter `token`, `doc_token`, `docToken`, `document_token`, `document.token` ou `doc.token`. Throttle: 20/min. |

## Variaveis de ambiente relevantes

| Variavel | Uso |
| --- | --- |
| `ZAPSIGN_API_TOKEN` | Token ZapSign ambiente BR. |
| `ZAPSIGN_API_TOKEN_SANDBOX` | Token ZapSign sandbox. |
| `ZAPSIGN_DEFAULT_ENV` | `br` ou `sandbox`; default `br`. |
| `ZAPSIGN_WEBHOOK_SECRET` | Segredo obrigatorio para aceitar webhook. |
| `ZAPSIGN_TIMEOUT_SEC` | Timeout HTTP da integracao. |
| `ZAPSIGN_SEND_AUTOMATIC_EMAIL` | Default para envio automatico de e-mail pela ZapSign. |
| `CONTRACT_PDF_STORAGE_PATH` | Storage privado de PDFs de contratos. |
| `LEGAL_DOC_SERVICE_*` | Renderizacao server-side de HTML para PDF. |

## Observacoes de integracao

- O frontend deve preferir `signatureProvider`/`signatureEnv` e manter fallback para aliases `signature_provider`/`signature_env`.
- O filtro de listagem pode enviar `provider=zapsign` ou `signatureProvider=ZAPSIGN`.
- Antes de enviar ZapSign, o contrato precisa ter PDF salvo em storage privado.
- `SANDBOX` e restrito no backend a usuarios `DEV` ou `ADMIN`.
- A homologacao real precisa aplicar a migration `20260619143000_add_contract_signature_provider` e configurar os segredos ZapSign no ambiente alvo.
