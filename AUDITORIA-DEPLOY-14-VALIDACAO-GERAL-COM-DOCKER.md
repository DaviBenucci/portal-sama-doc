# AUDITORIA DEPLOY 14 - Validacao geral com Docker

Data: 10/06/2026  
Escopo: revalidacao das analises descritas em `Testes-da-aplicacao-DEPLOY.md` e nos relatorios `AUDITORIA-DEPLOY-00` a `AUDITORIA-DEPLOY-13`, com Docker/MySQL local ativo.

## 1. Veredito atualizado

Status atual: **nao pronto para producao e nao pronto para usuarios reais**.

O quadro melhorou em banco e Acessorias:

- migrations em MySQL limpo agora passam;
- seed, bootstrap admin e readiness com banco Docker passam;
- homologacao local da Acessorias segue validada com 21 chamadas API, 0 falhas e persistencia real no MySQL local.

Mas ainda permanecem bloqueios de release:

- ZIP da API ainda contem `.env`;
- ZIP da API tambem empacota `.ai-tests` e `node_modules`;
- `clients.read` continua amplo demais;
- `home-summary` da Acessorias continua aceitando `profile` pela query;
- E2E API continua vermelho em 2 testes Web Push/CSRF;
- E2E Web continua vermelho em 2 testes da Home;
- API ainda tem 1 vulnerabilidade moderada em `qs`;
- ClamAV strict, backup/restore e homologacao real por perfil ainda nao estao provados.

## 2. Ambiente validado

- Node: `v24.15.0`.
- npm: `11.12.1`.
- Docker: container `portal-sama-audit-mysql` ativo.
- MySQL: imagem `mysql:8.4`, porta local `127.0.0.1:3308`.
- Banco de validacao criado nesta rodada: `portal_sama_validation`.
- Banco de homologacao Acessorias anterior mantido: `portal_sama_audit`.

Arquivos de evidencia gerados:

- `portal-sama-api/.ai-tests/validation-api-results.json`
- `portal-sama-web/.ai-tests/validation-web-results.json`
- `portal-sama-api/.ai-tests/acessorias-api-results-clean.json`

## 3. Validacao dos artefatos ZIP

| Artefato | Resultado | Leitura |
|---|---|---|
| `portal-sama-api.zip` | Reprovado | Contem `portal-sama-api/.env`, `.env.example`, `.ai-tests`, backups locais e `node_modules`. |
| `portal-sama-web.zip` | Parcial | Contem apenas `.env.example` entre os envs, mas tambem inclui `node_modules`. |
| `portal-sama-docs.zip` | OK no filtro sensivel | Nenhum `.env` encontrado pelo filtro executado. |

Conclusao: o achado `SEC-01` permanece critico e ficou mais forte. O ZIP da API nao deve ser compartilhado nem usado como artefato de deploy.

## 4. Validacao da API

| Area | Comando | Resultado |
|---|---|---|
| Ambiente | `node -v; npm.cmd -v` | OK |
| Prisma schema | `npm.cmd run prisma:validate` | OK |
| Lint | `npm.cmd run lint` | OK |
| Build | `npm.cmd run build` | OK |
| Unitarios | `npm.cmd test -- --runInBand` | OK: 44 suites, 264 testes |
| Audit npm | `npm.cmd audit --audit-level=moderate` | Falhou: 1 vulnerabilidade moderada em `qs` |
| DB limpo Docker | drop/create `portal_sama_validation` | OK |
| Migrations | `npm.cmd run prisma:migrate:deploy` | OK: 29 migrations aplicadas |
| Migration status | `npm.cmd run prisma:migrate:status` | OK: schema atualizado |
| Seed | `npm.cmd run prisma:seed` | OK |
| Bootstrap admin | `npm.cmd run prisma:bootstrap-admin` | OK: usuario DEV local criado |
| Readiness | `node scripts/validate-operational-readiness.js --soft` | OK com warnings |
| E2E API | `npm.cmd run test:e2e` | Falhou: 2 falhas, 134 passaram |

Readiness com Docker:

- database: passou;
- production-schema: passou;
- migrations: passou;
- RBAC: passou, 91 permissoes e 9 roles;
- privileged-users: passou;
- storage: passou;
- warnings: `SAMA_UPLOAD_SCAN_MODE` nao strict, ClamAV ausente, backup/rollback sem prova externa.

## 5. Validacao do frontend

| Area | Comando | Resultado |
|---|---|---|
| Ambiente | `node -v; npm.cmd -v` | OK |
| Lint | `npm.cmd run lint` | OK |
| Build | `npm.cmd run build` | OK |
| Contratos web | `npm.cmd test -- --runInBand` | OK: 9 testes |
| Audit npm | `npm.cmd audit --audit-level=moderate` | OK: 0 vulnerabilidades |
| E2E Web | `npm.cmd run test:e2e` | Falhou: 2 falhas, 11 passaram, 1 skip |

Falhas E2E Web confirmadas:

- `home page renders authenticated shortcuts from session permissions`: nao encontrou `Integracao`.
- `home page renders collaborator daily priorities from available data`: nao encontrou `Entregue pelo Acessorias`.

Avisos de rede confirmados durante Playwright:

- proxy para `/api-v2/notifications/stream?take=20` com `ECONNREFUSED`;
- proxy para `/api-v2/me/security` com `ECONNREFUSED`;
- warnings de `No HydrateFallback`.

Leitura: as falhas continuam alinhadas ao diagnostico anterior. A Home renderiza, mas o contrato E2E parece desatualizado em relacao aos textos atuais, ou a UI precisa voltar ao contrato antigo.

## 6. Validacao Acessorias

Foi reutilizada a evidencia sanitizada da rodada anterior em `portal-sama-api/.ai-tests/acessorias-api-results-clean.json`.

Resumo:

- 21 chamadas API;
- 0 falhas;
- login local com CSRF/JWT OK;
- scheduler desligado;
- rate limiter sem falhas;
- preview de cadastros OK;
- sync de cadastros OK;
- sync de catalogo OK;
- preview de entregas OK;
- sync de entregas OK;
- reconciliacao OK;
- home-summary local depois do sync OK.

Contagens finais no banco `portal_sama_audit`:

| Item | Total |
|---|---:|
| clients | 20 |
| catalogo Acessorias | 892 |
| entregas Acessorias | 50 |
| delivery sync runs | 1 |
| sync runs tecnicos | 3 |
| divergencias fiscais | 100 |
| aliases responsaveis | 34 |

Runs tecnicos finais:

- `ACESSORIAS_COMPANY_CATALOG_SYNC`: `SUCCESS`, 20 fetched, 20 updated.
- `ACESSORIAS_DELIVERIES_INCREMENTAL_SYNC`: `SUCCESS`, 50 fetched, 50 created.
- `ACESSORIAS_RECONCILIATION_JOB`: `SUCCESS`, 976 fetched, 100 created.

Conclusao: a analise de Acessorias local com Docker esta validada.

## 7. Achados revalidados

| ID | Achado | Status nesta rodada | Severidade |
|---|---|---|---|
| SEC-01 | ZIP da API contem `.env` | Confirmado | Critico |
| SEC-02 / AUTHZ-01 | `clients.read` sem escopo suficiente | Confirmado por codigo | Critico |
| AUTHZ-03 | Acessorias Home aceita `profile=admin` pela query | Confirmado por codigo | Critico |
| DB-MIG-01 | Migration Acessorias falhava por indice longo | Corrigido localmente e validado em DB limpo | Resolvido localmente |
| QA-API-01 | E2E API falha em Web Push/CSRF | Confirmado | Alta |
| QA-WEB-01 | E2E Web falha na Home | Confirmado | Alta |
| DEP-01 | Vulnerabilidade moderada em `qs` | Confirmado | Media |
| OPS-CLAMAV | ClamAV strict nao validado | Confirmado | Alta |
| OPS-BACKUP | Backup/restore drill sem prova externa | Confirmado | Alta |
| WEBPUSH-REAL | VAPID/navegador/HTTPS real nao homologados | Confirmado | Alta |
| PERFIS-REAL | DEV/Gestor/Colaborador/Cliente reais nao homologados de ponta a ponta | Confirmado | Alta |

## 8. O que mudou desde as auditorias anteriores

Melhorou:

- O bloqueio de migration em MySQL limpo foi corrigido no codigo local.
- `prisma:migrate:deploy`, `migrate:status`, `seed`, `bootstrap-admin` e readiness agora passam contra MySQL Docker limpo.
- Acessorias deixou de ser pendencia local: as rotas principais foram homologadas com chamada externa real e persistencia local.

Nao mudou:

- A aplicacao continua No-Go por seguranca/autorizacao e gates E2E.
- O ZIP da API continua contaminado.
- Os E2E oficiais continuam falhando.
- Dependencia `qs` continua com vulnerabilidade moderada.
- ClamAV, backup/restore, Web Push real e perfis reais seguem pendentes de ambiente/prova.

## 9. Matriz do checklist principal

| Exigencia de `Testes-da-aplicacao-DEPLOY.md` | Status validado | Evidencia |
|---|---|---|
| Subir/validar backend localmente | Parcial | Build, lint, unit, DB e readiness OK; E2E falha. |
| Subir/validar frontend localmente | Parcial | Build, lint, contratos OK; E2E falha. |
| Rodar build | OK | API e Web passaram. |
| Rodar lint | OK | API e Web passaram. |
| Rodar testes existentes | Parcial | Unit/contratos OK; E2E API/Web falham. |
| Criar/executar E2E | Parcial | Suites existentes rodadas; gates vermelhos. |
| Testar usuario real | Parcial | Bootstrap DEV local validado na Acessorias; falta Gestor/Colaborador/Cliente reais. |
| Testar autenticacao | Parcial | Auth/CSRF cobertos por E2E e Acessorias; E2E Web Push/CSRF falha. |
| Testar autorizacao | Reprovado | `clients.read` e `home-summary?profile=admin` seguem criticos. |
| Testar documentos | Parcial | Testes unit/E2E cobrem rotas, mas ClamAV strict e fluxo real pendem. |
| Testar integracoes | Parcial/OK local | Acessorias OK local; Web Push real e demais integracoes reais pendem. |
| Capturar logs/screenshots/traces | OK parcial | Playwright gerou screenshots/videos/traces das falhas. |
| Decidir prontidao | OK | Decisao segue No-Go para usuarios reais. |

## 10. Proximos passos obrigatorios

1. Remover/substituir `portal-sama-api.zip` e recriar artefatos sem `.env`, `.ai-tests`, `node_modules`, dumps, backups e logs.
2. Rotacionar segredos potencialmente expostos pelo ZIP.
3. Corrigir escopo real de `clients.read` em listagem, detalhe e dashboard.
4. Corrigir `home-summary` para derivar perfil no backend e exigir permissao/role para visao admin.
5. Adicionar testes negativos para os dois pontos de autorizacao.
6. Corrigir isolamento do E2E API para Web Push/CSRF.
7. Atualizar contrato E2E Web da Home ou ajustar UI para o contrato esperado.
8. Corrigir vulnerabilidade `qs` e repetir `npm audit`.
9. Validar ClamAV em modo strict com EICAR no ambiente alvo.
10. Executar backup, verify e restore drill em alvo isolado.
11. Homologar DEV, Gestor, Colaborador e Cliente com dados reais controlados.
12. Homologar Web Push com HTTPS, VAPID real e navegador real autenticado.

## 11. Conclusao

As analises de `Testes-da-aplicacao-DEPLOY.md` e `AUDITORIA-DEPLOY` foram revalidadas localmente com Docker.

A base tecnica esta mais forte do que nos primeiros relatorios: banco limpo, migrations, RBAC, bootstrap e Acessorias agora tem evidencia local positiva. Ainda assim, a decisao final permanece:

**Nao liberar para producao nem para usuarios reais.**

O sistema pode seguir para uma proxima rodada de correcao/homologacao controlada somente depois de resolver os bloqueios criticos de autorizacao e artefatos, e depois de deixar E2E API/Web verdes.
