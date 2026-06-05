# 06 — Notificações internas e Web Push — MVP Portal Sama

**Status:** fonte ativa do MVP  
**Data:** 2026-06-03  
**Objetivo:** definir o escopo mínimo, seguro e obrigatório de notificações internas e Web Push para o fechamento do MVP.

---

## Status de implementação em 2026-06-05

O escopo mínimo de Notificações/Web Push está implementado no código do MVP.

### Concluído

- Central de Notificações acessível no frontend.
- Sino/contador no layout.
- Listagem de notificações internas.
- Marcação individual como lida.
- Marcação em massa como lida.
- SSE/polling para atualização da Central.
- Web Push por navegador/dispositivo.
- Service worker de push no frontend.
- Ativação/desativação de Web Push no navegador.
- Listagem e revogação dos próprios dispositivos.
- Preferências Web Push por usuário/tipo de evento.
- Registro de tentativa de entrega por canal/provider/status.
- Payload externo sanitizado e genérico.
- Falha de Web Push não quebra a ação principal.
- Endpoint de subscribe exige usuário autenticado e CSRF.
- Usuário não lista nem revoga dispositivo de outro usuário.
- VAPID private key permanece somente no backend.

### Reaproveitamento adotado

- `Notification` foi usado como evento interno persistido do MVP.
- `BrowserPushSubscription` foi usado como assinatura Web Push vinculada ao usuário.
- Foram adicionadas tabelas específicas para `notification_delivery_attempts` e `notification_preferences`.

### Pendente para aceite final

- Aplicar migrations nos ambientes.
- Configurar VAPID real no backend.
- Homologar envio Web Push em navegador real.
- Validar em fluxo real todos os produtores de eventos obrigatórios.
- Confirmar recebimento por colaborador, gestor e DEV/ADMIN nos cenários operacionais.

---

## 1. Decisão de produto

Notificações internas e Web Push fazem parte do MVP do Portal Sama.

Motivo: o Portal Sama centraliza obrigações, contratos, documentos, certificados, solicitações de acesso, onboarding, extratos, Integra-AI e gestão operacional. Se uma ação crítica ocorrer e o usuário não estiver olhando a tela, a operação pode perder prazo, validação ou correção necessária.

A implementação do MVP deve seguir esta regra:

```txt
Notificação interna persistida = obrigatória
Web Push = canal complementar em tempo real, quando autorizado pelo usuário
```

O Web Push não substitui a Central de Notificações. Ele apenas avisa o usuário e o direciona ao Portal Sama, onde autenticação e RBAC carregam os detalhes.

---

## 2. Escopo permitido no MVP

Implementar agora:

- criação de eventos internos de notificação;
- Central de Notificações no frontend;
- sino/contador de notificações no layout;
- marcação de notificação como lida;
- preferências básicas por usuário/canal;
- inscrição de navegador/dispositivo para Web Push;
- listagem e revogação dos próprios dispositivos;
- envio de Web Push para eventos críticos;
- registro de tentativas de entrega;
- payload externo sanitizado;
- auditoria em ações críticas de notificação.

---

## 3. Fora do escopo do MVP

Não implementar agora:

- WhatsApp;
- Slack;
- Teams;
- SMS;
- resumo diário inteligente;
- agrupamento avançado;
- regras complexas de silêncio por horário;
- IA decidindo sozinha prioridade operacional;
- push com dados sensíveis;
- notificações públicas sem sessão/RBAC.

---

## 4. Arquitetura recomendada

### Backend

```txt
src/modules/notifications/
  notifications.module.ts
  notifications.controller.ts
  notifications.service.ts
  notification-events.service.ts
  notification-dispatcher.service.ts
  notification-preferences.service.ts
  notification-delivery-attempts.service.ts
  dto/
  types/

src/modules/notifications/web-push/
  web-push.module.ts
  web-push.controller.ts
  web-push.service.ts
  web-push-subscriptions.service.ts
  web-push-dispatcher.service.ts
  dto/
  types/
```

Se o projeto já possuir `Notification` e `BrowserPushSubscription`, reaproveitar ou migrar sem duplicar responsabilidades.

### Frontend

```txt
src/services/notifications.service.ts
src/services/web-push.service.ts
src/hooks/useWebPush.ts
src/components/notifications/NotificationBell.tsx
src/components/notifications/NotificationCenter.tsx
src/components/notifications/EnablePushNotificationsButton.tsx
src/components/notifications/NotificationPreferencesPanel.tsx
public/service-worker.js
```

---

## 5. Banco de dados mínimo

### `notification_events`

```txt
id
recipient_user_id
actor_user_id
type
severity
title
message
safe_payload
entity_type
entity_id
read_at
created_at
```

### `notification_delivery_attempts`

```txt
id
notification_event_id
channel
provider
status
attempted_at
error_message
metadata
```

### `notification_preferences`

```txt
id
user_id
notification_type
channel
enabled
created_at
updated_at
```

### `web_push_subscriptions`

```txt
id
user_id
endpoint
p256dh_key
auth_key
user_agent
device_name
last_used_at
revoked_at
created_at
updated_at
```

Observação: `endpoint`, `p256dh_key` e `auth_key` são dados operacionais sensíveis. Não devem aparecer em logs de aplicação nem em respostas de listagem sem mascaramento.

---

## 6. Variáveis de ambiente

```env
WEB_PUSH_VAPID_PUBLIC_KEY="..."
WEB_PUSH_VAPID_PRIVATE_KEY="..."
WEB_PUSH_CONTACT="mailto:suporte@empresa.com"
WEB_PUSH_ENABLED="true"
```

Regras:

- `WEB_PUSH_VAPID_PUBLIC_KEY` pode ser exposta ao frontend;
- `WEB_PUSH_VAPID_PRIVATE_KEY` fica somente no backend;
- nenhuma chave VAPID deve ser commitada;
- se chave real aparecer em ZIP, commit, log ou prompt, rotacionar imediatamente.

---

## 7. Endpoints mínimos

```txt
GET    /api-v2/notifications
PATCH  /api-v2/notifications/:id/read
PATCH  /api-v2/notifications/read-all
GET    /api-v2/notifications/preferences
PATCH  /api-v2/notifications/preferences
POST   /api-v2/notifications/web-push/subscribe
POST   /api-v2/notifications/web-push/unsubscribe
GET    /api-v2/notifications/web-push/devices
DELETE /api-v2/notifications/web-push/devices/:id
```

Todos os endpoints devem exigir autenticação. Ações mutáveis devem respeitar CSRF, se o padrão do projeto exigir.

---

## 8. Eventos obrigatórios do MVP

### Acessórias e obrigações

```txt
ACESSORIAS_DELIVERY_COMPLETED
ACESSORIAS_DELIVERY_DUE_SOON
ACESSORIAS_DELIVERY_OVERDUE
ACESSORIAS_SYNC_FAILED
ACESSORIAS_SYNC_COMPLETED
ACESSORIAS_BACKFILL_COMPLETED
ACESSORIAS_BACKFILL_FAILED
ACESSORIAS_DIVERGENCE_CREATED
ACESSORIAS_RESPONSIBLE_PENDING_REVIEW
```

### Solicitações de acesso / T.I.

```txt
ACCESS_REQUEST_CREATED
ACCESS_REQUEST_MANAGER_REVIEW
ACCESS_REQUEST_DEV_REVIEW
ACCESS_REQUEST_APPROVED
ACCESS_REQUEST_REJECTED
ACCESS_REQUEST_COMPLETED
```

### Contratos e propostas

```txt
CONTRACT_SENT
CONTRACT_SIGNED
CONTRACT_PENDING
CONTRACT_REJECTED
PROPOSAL_SENT
PROPOSAL_APPROVED
PROPOSAL_REJECTED
```

### Certificados digitais

```txt
CERTIFICATE_EXPIRING
CERTIFICATE_EXPIRED
CERTIFICATE_RENEWED
```

### Documentos

```txt
DOCUMENT_APPROVED
DOCUMENT_REJECTED
DOCUMENT_UPLOADED
```

### Sistema

```txt
SYSTEM_ACTION_SUCCESS
SYSTEM_ACTION_FAILED
SECURITY_RELEVANT_ACTION
```

---

## 9. Destinatários mínimos

### Colaborador

Recebe:

- obrigação próxima do prazo;
- obrigação vencida;
- obrigação entregue/concluída vinculada a ele;
- documento aprovado/recusado quando aplicável;
- contrato/proposta que exija ação dele;
- resultado de solicitação de acesso criada por ele.

### Gestor

Recebe:

- solicitação de acesso aguardando validação;
- obrigações vencidas do seu departamento/time;
- divergências do Acessórias;
- pendências relevantes por colaborador;
- contrato/proposta/certificado que exija validação gerencial.

### DEV/ADMIN/T.I.

Recebe:

- solicitação de acesso aprovada e pronta para execução;
- erro crítico do Acessórias;
- token inválido/sem permissão;
- rate limit persistente;
- falha de backfill/sync;
- erro sistêmico relevante.

---

## 10. Segurança do payload Web Push

O payload Web Push deve ser curto, genérico e seguro.

### Permitido

```txt
Título: Nova solicitação de acesso
Mensagem: Uma solicitação aguarda validação.

Título: Obrigação vencida
Mensagem: Você possui uma obrigação pendente vencida.

Título: Falha de sincronização
Mensagem: Uma integração precisa de atenção técnica.
```

### Proibido

Não enviar no push:

- CNPJ completo;
- CPF;
- senha;
- token;
- certificado digital;
- contrato ou cláusula;
- documento ou anexo;
- dados bancários;
- stack trace;
- endpoint externo com segredo;
- payload bruto do Acessórias;
- detalhes sensíveis do cliente.

Detalhes completos devem ser buscados dentro do Portal Sama após login e validação de permissão.

---

## 11. Fluxo de inscrição Web Push

```txt
Usuário autenticado acessa Portal Sama
        ↓
Frontend verifica suporte do navegador
        ↓
Usuário clica em Ativar notificações neste dispositivo
        ↓
Navegador pede permissão
        ↓
Frontend registra service worker
        ↓
Frontend cria PushSubscription
        ↓
Frontend envia subscription para backend autenticado
        ↓
Backend salva subscription vinculada ao usuário
        ↓
Portal Sama pode enviar Web Push para aquele dispositivo
```

Estados visuais do botão:

```txt
Não suportado pelo navegador
Permissão pendente
Ativar notificações
Ativado neste dispositivo
Permissão negada
Erro ao ativar
```

---

## 12. Fluxo de envio

```txt
Evento de domínio ocorre
        ↓
NotificationDispatcher identifica destinatários
        ↓
Cria NotificationEvent interno
        ↓
Verifica preferências do usuário
        ↓
Se Web Push ativo, busca subscriptions válidas
        ↓
Envia Web Push sanitizado
        ↓
Registra tentativa de entrega
        ↓
Se subscription inválida, marca como revogada/inativa
```

Falha de Web Push não deve desfazer a ação principal. Exemplo: se uma solicitação de acesso foi criada, ela permanece criada mesmo que o push falhe.

---

## 13. Exemplos operacionais

### Solicitação de acesso criada

Notificação interna para gestor:

```txt
João Silva enviou uma solicitação de acesso para o departamento Fiscal.
```

Web Push para gestor:

```txt
Nova solicitação de acesso
Uma solicitação aguarda validação do gestor.
```

### Solicitação aprovada pelo gestor e enviada ao T.I.

Notificação interna para DEV/ADMIN/T.I.:

```txt
João Silva solicitou acesso para o departamento Fiscal. Validação do gestor aprovada.
```

Web Push:

```txt
Solicitação de acesso aprovada
Uma solicitação aguarda execução pelo T.I.
```

### Obrigação vencida

Notificação interna:

```txt
A obrigação vinculada ao colaborador está vencida e pendente de baixa/conclusão.
```

Web Push:

```txt
Obrigação vencida
Você possui uma obrigação pendente vencida.
```

### Erro crítico no Acessórias

Notificação interna para DEV/ADMIN:

```txt
A sincronização do Acessórias falhou por erro externo ou credencial inválida. Verifique o diagnóstico técnico.
```

Web Push:

```txt
Falha de sincronização
Uma integração precisa de atenção técnica.
```

---

## 14. Permissões sugeridas

```txt
notifications.read
notifications.read_all
notifications.mark_read
notifications.manage_preferences
notifications.web_push.subscribe
notifications.web_push.unsubscribe
notifications.web_push.manage_own_devices
notifications.web_push.audit
notifications.dispatch.system
```

Apenas DEV/ADMIN deve ter permissão para auditoria ampla de tentativas e falhas técnicas. Usuário comum só gerencia os próprios dispositivos.

---

## 15. Testes obrigatórios

Backend:

- cria `NotificationEvent` para evento crítico;
- não duplica notificação para mesmo evento/destinatário;
- envia Web Push quando usuário tem subscription ativa;
- não envia quando preferência está desativada;
- registra `notification_delivery_attempt` em sucesso e falha;
- falha de provider não quebra fluxo principal;
- usuário não lista/revoga dispositivo de outro usuário;
- payload externo não contém dados sensíveis;
- VAPID private key não aparece em resposta.

Frontend:

- exibe sino/contador;
- lista notificações;
- marca notificação como lida;
- mostra botão de ativação com estados corretos;
- lida com permissão negada;
- registra service worker;
- não tenta ativar Web Push em navegador sem suporte.

Segurança:

- endpoint de subscribe exige autenticação;
- subscription fica vinculada ao usuário autenticado;
- push não contém CNPJ/CPF completo, certificado, contrato, documento, token ou stack trace;
- detalhes só aparecem após login/RBAC.

---

## 16. Critérios de aceite

A fase de Notificações/Web Push do MVP só está concluída quando:

- Central de Notificações está acessível;
- sino/contador aparece no layout;
- usuário consegue ativar/desativar Web Push no navegador;
- subscription é salva por usuário autenticado;
- eventos mínimos criam notificação interna;
- eventos mínimos tentam Web Push quando permitido;
- falhas de push são registradas e não quebram a operação;
- payload de Web Push é seguro;
- DEV/ADMIN recebe falhas críticas;
- gestor recebe validações pendentes;
- colaborador recebe obrigações próximas/vencidas/entregues;
- testes críticos passam;
- documentação ativa está atualizada.
