# Checklist Go-Live Portal Sama

Status base: atualizacao Codex de 2026-06-22.

## Já comprovado

- [x] API NestJS compila.
- [x] ESLint da API passa.
- [x] Testes unitarios da API passam: 56 suites / 347 testes.
- [x] E2E da API passa: 1 suite / 148 testes.
- [x] `prisma validate` passa.
- [x] Migrações aplicam em MySQL descartável: 30 migrações.
- [x] Seed/RBAC aplica em MySQL descartável.
- [x] Bootstrap admin de teste funciona em MySQL descartável.
- [x] Frontend compila com Vite.
- [x] ESLint do frontend passa.
- [x] Testes contratuais web passam: 12/12.
- [x] Playwright local passa: 22 testes / 1 real pulado.
- [x] Smoke público real passa: frontend, API health, CORS e CSRF.
- [x] Smoke autenticado real passa: login, refresh, `/auth/me`, cookie `httpOnly` e logout.

## Bloqueadores antes de produção ampla

- [ ] Rotacionar todos os segredos após homologação.
- [ ] Registrar data e responsável da rotação em controle externo.
- [ ] Remover `.env` de qualquer pacote, backup compartilhado e artefato.
- [ ] Gerar pacote seguro sem `.env`, `.git`, `node_modules`, `dist`, logs e `.ai-tests`.
- [ ] Validar `prisma migrate deploy` no banco alvo.
- [ ] Validar seed/RBAC no banco alvo.
- [ ] Desativar, trocar ou restringir bootstrap admin.
- [x] Validar `ops:readiness` no ambiente alvo; rodada real de 2026-06-22 retornou `ok=true`, com banco, migrations, RBAC, usuario privilegiado, storage e ClamAV/EICAR aprovados.
- [x] Executar `ops:phase8` no ambiente alvo com backup real, verify e restore drill apply em banco/storage isolados; rodada real retornou `ok=true`, `failed=0`, `blocked=0`.
- [x] Colocar scanner de upload em modo `strict` e comprovar ClamAV/EICAR.
- [x] Executar backup, verify e restore drill com dump real; backup id `portal-sama-20260622T165857Z-23e713`, restore em `banco-sama_restore_drill` e `/tmp/portal-sama-restore-storage`.
- [x] Corrigir links públicos gerados pelo backend para rotas React atuais.
- [x] ZapSign implementado no MVP com provider externo, webhook, sync e bloqueio de assinatura interna para contratos `ZAPSIGN`.
- [ ] Homologar Acessórias com token/payload real e volume representativo.
- [ ] Revisar escopo por objeto/cliente nos endpoints com IDs.
- [ ] Rodar matriz completa de permissões por perfil.
- [ ] Validar bot protection nos fluxos públicos.

## Critério mínimo de liberação controlada

- [ ] Go-live restrito a usuários piloto.
- [ ] Monitoramento ativo da API, banco, uploads, login e Acessórias.
- [ ] Plano de rollback documentado e testado.
- [ ] Usuários críticos com credenciais rotacionadas.
- [ ] Comunicação interna com canais de suporte e janela de observação.

## Implementações iniciadas em 2026-06-19

- [x] Backend passou a gerar `/assinatura/:token` para assinatura pública de contratos.
- [x] Backend passou a gerar `/onboarding/publico/proposta/:token` para propostas públicas.
- [x] Frontend ganhou redirects compatíveis para `/Legalizacao/assinatura.html?token=...` e `/Onboarding/proposta-cliente.html?token=...`.
- [x] `.dockerignore` da API e do Web endurecidos contra `.env`, `.git`, pacotes, dumps, backups, logs e artefatos de teste.

## Implementacoes iniciadas em 2026-06-22

- [x] ZapSign conectado a contratos por `POST /api-v2/contracts/:id/send-signature`, `POST /api-v2/contracts/:id/zapsign/sync` e `POST /api-v2/webhooks/zapsign`.
- [x] Contrato de API para frontend congelado em `docs/21-CONTRATO-API-FRONTEND.md`.
- [x] Orquestrador `npm.cmd run ops:phase8` criado para consolidar readiness/schema/secrets/backup/verify/restore sem liberar Fase 8 com skips.
- [x] `ops:secrets:check` passou a validar ZapSign com `--require-zapsign`.
- [x] `ops:secrets:check` real validou ZapSign com token/webhook configurados, sem expor valores.
- [ ] Homologar fluxo funcional ZapSign fim a fim com envio/sync/webhook real de contrato controlado.

## Avisos da Fase 8 real

- [ ] Copiar evidencia JSON e artefatos do backup real para armazenamento operacional seguro fora de `/tmp`.
- [ ] Registrar data/responsavel de rotacao de secrets em controle externo.
- [ ] Planejar hardening futuro de `CERTIFICATE_ENCRYPTION_KEY` e `ACESSORIAS_TOKEN` conforme warnings do secret check.
