# Testes de Homologação MVP

Status base: testes técnicos e smokes reais aprovados em 2026-06-19. Esta matriz cobre o que ainda precisa ser homologado como produto.

## Técnica automatizada

- [x] API lint.
- [x] API build.
- [x] API unit tests.
- [x] API e2e.
- [x] Prisma validate.
- [x] Migrations em MySQL descartável.
- [x] Seed/RBAC em MySQL descartável.
- [x] Web lint.
- [x] Web build.
- [x] Web contract tests.
- [x] Playwright local.
- [x] Smoke público real.
- [x] Smoke autenticado real.
- [x] Playwright real auth.

## Homologação funcional

- [ ] Login/logout/refresh por perfil.
- [ ] Home DEV.
- [ ] Home ADMIN.
- [ ] Home MANAGER.
- [ ] Home colaborador.
- [ ] Clientes e dashboard.
- [ ] Central de Vencimentos.
- [ ] Documentos privados.
- [ ] Upload público por token.
- [ ] Certificados digitais.
- [ ] Solicitação de acesso.
- [ ] Onboarding.
- [ ] Proposta pública.
- [ ] Contrato público.
- [ ] Notificações internas.
- [ ] Web Push.
- [ ] Auditoria.
- [ ] Integra-AI.
- [ ] Acessórias incremental.
- [ ] Acessórias backfill.

## Homologação de segurança

- [ ] Matriz completa de permissões.
- [ ] Escopo por cliente/objeto.
- [ ] CSRF em mutações autenticadas.
- [ ] Rate limit em login e rotas públicas.
- [ ] Bot protection nos fluxos públicos.
- [ ] Upload scanner strict.
- [ ] CSP e headers de produção.
- [ ] Logs sem dados sensíveis.
- [ ] Segredos rotacionados.

## Evidência mínima por teste

- Data.
- Ambiente.
- Usuário/perfil, sem senha.
- Fluxo testado.
- Resultado.
- Evidência técnica sem dados sensíveis.
- Responsável.
