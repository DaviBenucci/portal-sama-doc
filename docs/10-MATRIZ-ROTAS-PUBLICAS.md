# Matriz de Rotas Públicas

Status base: smoke público real aprovado em 2026-06-19 para frontend, health, CORS e CSRF.

## Rotas React atuais

| Fluxo | Rota frontend atual | Endpoint API relacionado | Estado |
| --- | --- | --- | --- |
| Assinatura publica interna | `/assinatura/:token` | `/api-v2/public/signatures/:token` e `/api-v2/public/signatures/:token/sign` | Provider `INTERNAL`; API existe; link backend corrigido em 2026-06-19 |
| Assinatura ZapSign | `/assinatura/:token` redireciona/mostra link externo | `/api-v2/public/signatures/:token` | Provider `ZAPSIGN`; POST interno de assinatura e bloqueado |
| Proposta pública | `/onboarding/publico/proposta/:token` | `/api-v2/public/proposals/:token` | API existe; link backend corrigido em 2026-06-19 |
| Documentos públicos | `/onboarding/publico/documentos/:token` | `/api-v2/public/onboarding/documents/:token` | Rota atual usada no frontend |

## Rotas publicas externas

| Origem | Endpoint API | Estado |
| --- | --- | --- |
| ZapSign webhook | `POST /api-v2/webhooks/zapsign` | Implementado em 2026-06-22; exige segredo em header e aplica throttle 20/min |

## Links legados encontrados

| Origem | Link gerado | Correção |
| --- | --- | --- |
| Contratos | `/Legalizacao/assinatura.html?token=...` | Backend gera `/assinatura/:token`; frontend redireciona link legado |
| Propostas | `/Onboarding/proposta-cliente.html?token=...` | Backend gera `/onboarding/publico/proposta/:token`; frontend redireciona link legado |

## Regras de segurança

- [ ] Token bruto nunca deve ser persistido em banco.
- [ ] Token deve ser armazenado por hash.
- [ ] Token deve ter expiração.
- [ ] Token deve ser revogável.
- [ ] Token usado deve ser revogado quando o fluxo permitir uso único.
- [x] Rate limit por IP e por token em camada MVP local.
- [ ] Limite de tamanho e quantidade para uploads públicos.
- [ ] Auditoria de IP, user-agent e evento.
- [ ] Mensagens de erro não devem revelar se entidade interna existe.
- [ ] Bot protection em ações sensíveis antes de produção ampla.
- [x] Webhook ZapSign rejeita chamada sem segredo configurado/valido.

## Testes de aceite

- [x] Token válido abre tela pública correta para redirects legados mockados em Playwright.
- [x] Token expirado mostra estado amigável e não expõe dados em smoke Playwright mockado.
- [x] Token revogado mostra estado amigável e não expõe dados em smoke Playwright mockado.
- [ ] Upload público rejeita extensão, MIME e assinatura inválidos.
- [ ] Assinatura pública registra evidência e revoga token quando aplicável.
- [x] Contrato `ZAPSIGN` bloqueia assinatura publica interna e preserva link externo do provedor.
- [ ] Proposta pública registra aceite/ajuste/rejeição com auditoria.

## Implementações iniciadas em 2026-06-19

- [x] `@Throttle` mantido nas rotas públicas para limite por IP.
- [x] Limiter por hash de token público aplicado em contratos, propostas e documentos.
- [ ] Substituir limiter local por Redis/Valkey antes de escalar API com múltiplas réplicas.
