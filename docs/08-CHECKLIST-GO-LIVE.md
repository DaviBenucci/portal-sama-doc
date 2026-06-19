# Checklist Go-Live Portal Sama

Status base: atualização Codex de 2026-06-19.

## Já comprovado

- [x] API NestJS compila.
- [x] ESLint da API passa.
- [x] Testes unitários da API passam: 44 suítes / 291 testes.
- [x] E2E da API passa: 1 suíte / 136 testes.
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
- [ ] Validar `ops:readiness` no ambiente alvo.
- [ ] Colocar scanner de upload em modo `strict` e comprovar ClamAV/EICAR.
- [ ] Executar backup, verify e restore drill.
- [x] Corrigir links públicos gerados pelo backend para rotas React atuais.
- [ ] Decidir ZapSign: implementar agora ou retirar do escopo MVP.
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
