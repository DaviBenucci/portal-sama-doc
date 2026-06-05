# CHANGELOG — Atualização Web Push no MVP

**Data:** 2026-06-03

## 2026-06-05 — Consolidação implementada no código

O escopo mínimo de Notificações/Web Push foi implementado no código do MVP.

Concluído:

- endpoints mínimos de notificações e Web Push;
- Central de Notificações;
- sino/contador;
- service worker;
- inscrição e revogação de dispositivo;
- listagem de dispositivos próprios;
- preferências Web Push por usuário/tipo;
- tentativas de entrega por canal/provider/status;
- payload externo sanitizado;
- testes backend para preferências, tentativas, segurança do payload e escopo de dispositivo.

Validações executadas:

- API build;
- API tests com `--runInBand`;
- Web build;
- Web lint.

Ainda depende de ambiente:

- migrations aplicadas;
- VAPID real configurado;
- RBAC/seed sincronizado;
- homologação real dos eventos obrigatórios.

---

## Alteração principal

Web Push deixou de ser classificado como funcionalidade futura e passou a ser requisito mínimo do MVP operacional do Portal Sama.

## Arquivos atualizados

- `00-LEIA-ME-PARA-IA-MVP.md`
- `01-ESTADO-ATUAL-CODIGO-DOCUMENTACAO.md`
- `02-PLANO-FECHAMENTO-MVP.md`
- `03-CONTRATO-ACESSORIAS-OPERACIONAL.md`
- `04-DIVERGENCIAS-DOCS-CODIGO.md`
- `05-PROMPT-CODEX-FECHAMENTO-MVP.md`
- `06-NOTIFICACOES-WEB-PUSH-MVP.md`

## Limite de escopo

Entrou no MVP:

- notificações internas;
- Central de Notificações;
- sino/contador;
- Web Push por navegador/dispositivo;
- preferências básicas;
- tentativas de entrega;
- eventos críticos operacionais;
- payload seguro.

Continua fora do MVP:

- WhatsApp;
- Slack;
- Teams;
- SMS;
- resumo diário inteligente;
- agrupamentos avançados;
- regras complexas por horário;
- dados sensíveis no push.
