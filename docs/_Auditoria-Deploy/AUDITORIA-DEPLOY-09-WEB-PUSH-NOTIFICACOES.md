# Auditoria Deploy - Web Push e Notificacoes

Data da analise: 10/06/2026

Escopo complementar: validar o que esta implementado no MVP de notificacoes internas e Web Push, cruzando `CHANGELOG-WEB-PUSH-MVP.md`, `06-NOTIFICACOES-WEB-PUSH-MVP.md`, API, web, testes e pendencias de producao.

## Veredito

Status: **implementado tecnicamente com evidencia local, mas ainda No-Go para producao**.

O modulo atende boa parte do desenho do MVP: endpoints existem, mutacoes exigem CSRF, dispositivos sao escopados ao usuario, preferencias sao por usuario/tipo, payload de push e sanitizado e a web tem service worker dedicado. Mesmo assim, o aceite final ainda depende de E2E API verde, VAPID real, navegador real autenticado, banco/migrations reais e homologacao dos eventos de negocio.

## Evidencias positivas encontradas

| Requisito | Status | Evidencia |
| --- | --- | --- |
| Listagem de notificacoes internas | Implementado | `portal-sama-api/src/modules/notifications/notifications.controller.ts:41`, `notifications.service.ts:120` |
| SSE para atualizacao da Central | Implementado | `notifications.controller.ts:48`, `notifications.service.ts:511`, `portal-sama-web/src/services/notifications.service.ts:156` |
| Criacao de notificacao com auditoria | Implementado | `notifications.controller.ts:58`, `notifications.service.ts:160` |
| Mutacoes protegidas por CSRF | Implementado no codigo | `notifications.controller.ts:67`, `:91`, `:123`, `:141`, `:159`, `:176`, `:198`, `:210` |
| Public key Web Push sem private key | Implementado | `notifications.controller.ts:99`, `notifications-push.service.ts:55`, teste contratual web |
| Subscribe/unsubscribe por navegador | Implementado | `notifications-push.service.ts:67`, `:120`, `portal-sama-web/src/services/notifications.service.ts:304`, `:337` |
| Listar/revogar dispositivos proprios | Implementado | `notifications-push.service.ts:144`, `:160`, `:303` |
| Preferencias por usuario/tipo/canal | Implementado | `notifications.service.ts:288`, `notifications-push.service.spec.ts:555` |
| Tentativas de entrega registradas | Implementado | `notifications-push.service.spec.ts:345`, `:568`, `:658` |
| Payload de Web Push sanitizado | Implementado | `notifications-push.service.ts:405`, `notifications-push.service.spec.ts:373`, `:409` |
| Service worker de push e click seguro | Implementado | `portal-sama-web/public/sama-push-sw.js:1`, `:31`, `:59` |
| Modelos de banco existem no Prisma | Implementado no schema | `prisma/schema.prisma:622`, `:650`, `:667`, `:682` |

## Testes executados nesta continuacao

| Area | Comando | Resultado |
| --- | --- | --- |
| API notificacoes/Web Push | `npm.cmd test -- --runInBand src/modules/notifications/notifications.service.spec.ts src/modules/notifications/notifications-push.service.spec.ts` | Passou: 2 suites, 28 testes |
| Web contratos Web Push/notificacoes | `npm.cmd test -- --runInBand` | Passou: 9 testes contratuais |

Observacao: a suite E2E completa da API continua registrada como falha na auditoria anterior, com 2 testes Web Push/CSRF retornando 403 por herdar `COOKIE_DOMAIN=portal.samacontabil.com.br` no ambiente local. A reproducao ja indicou que, com `COOKIE_DOMAIN=''` e `COOKIE_SECURE=false`, o endpoint de teste retorna 200 com `reason: push_disabled`.

## Pontos que ainda impedem aceite

### WEBPUSH-01 - E2E API ainda nao fecha no ambiente atual

Severidade: **Alta**.

Os testes E2E de `POST /api-v2/notifications/push/subscribe` e `POST /api-v2/notifications/push/test` falham por CSRF quando a suite herda `COOKIE_DOMAIN` do `.env` local. Mesmo que o diagnostico aponte problema de setup, o gate automatizado segue vermelho.

Acao: isolar `.env.test` ou forcar `COOKIE_DOMAIN=''` e `COOKIE_SECURE=false` no setup E2E local, preservando dominio e cookie seguro em producao.

### WEBPUSH-02 - VAPID real e navegador real ainda nao foram homologados

Severidade: **Alta**.

O codigo aceita aliases `WEB_PUSH_PUBLIC_KEY`, `WEB_PUSH_PRIVATE_KEY`, `WEB_PUSH_VAPID_PUBLIC_KEY` e `WEB_PUSH_VAPID_PRIVATE_KEY`, mas a auditoria local nao validou chaves reais, permissao do navegador, service worker instalado em HTTPS real, envio pelo provedor e clique na notificacao em sessao autenticada.

Acao: configurar VAPID real em homologacao, testar Chrome/Edge autenticado e registrar evidencia de subscribe, push recebido, click e unsubscribe.

### WEBPUSH-03 - Migrations/tabelas reais nao foram comprovadas

Severidade: **Alta**.

O schema Prisma contem `Notification`, `NotificationDeliveryAttempt`, `NotificationPreference` e `BrowserPushSubscription`, mas o banco MySQL local nao respondeu. Logo, nao ha prova de que as tabelas existem no alvo real.

Acao: rodar `prisma:migrate:status`, `migrate:deploy`, seed/RBAC e readiness em homologacao.

### WEBPUSH-04 - Eventos de negocio precisam de prova ponta a ponta

Severidade: **Media**.

Ha testes unitarios para access requests, documentos, certificados, contratos, propostas, legalizacao, onboarding e Acessorias gerando notificacoes. Ainda falta provar com usuarios reais que os eventos minimos chegam ao destinatario certo, respeitam preferencias e nao vazam dados no payload de push.

Acao: criar roteiro por perfil DEV, Gestor, Colaborador e Cliente, disparar eventos reais e conferir Central, contador, Web Push e auditoria.

## Checklist obrigatorio para fechar Web Push

| Item | Status atual | Aceite |
| --- | --- | --- |
| API unit focada | Go | Ja passou nesta continuacao |
| Web contrato | Go | Ja passou nesta continuacao |
| API E2E completo | No-Go | 100% verde, incluindo Web Push/CSRF |
| VAPID real | No-Go | Public/private key reais configuradas no backend |
| Browser real | No-Go | Chrome/Edge recebem notificacao em HTTPS |
| Subscribe/unsubscribe | Parcial | Validado em codigo/testes; falta navegador real |
| Service worker click | Parcial | Validado em contrato/codigo; falta navegador real |
| Tabelas reais | No-Go | Migrations aplicadas e readiness verde |
| Eventos de negocio | Parcial | Unitarios existem; falta homologacao ponta a ponta |

## Decisao da continuidade

Web Push deixou de ser apenas requisito futuro: ele esta implementado no MVP em nivel de codigo e testes locais. A decisao de deploy, contudo, permanece **No-Go para usuarios reais** ate que a prova operacional seja concluida.

