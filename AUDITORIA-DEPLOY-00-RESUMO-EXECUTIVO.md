# Auditoria Deploy - Resumo Executivo

Data da execução: 10/06/2026  
Escopo: `portal-sama-docs`, `portal-sama-api` e `portal-sama-web` locais.  
Base usada: `Testes-da-aplicação-DEPLOY.md` e documentos ativos `00` a `07`.

## Veredito

**Não aprovado para produção / usuários reais neste momento.**

O Portal Sama está tecnicamente próximo de uma homologação controlada: API e web compilam, lint passam, testes unitários/contratuais passam e a aplicação sobe localmente. Porém há bloqueios de segurança, QA e operação que impedem liberar para usuários reais com confiança.

## Principais bloqueios

| Prioridade | Achado | Impacto | Ação obrigatória |
| --- | --- | --- | --- |
| Crítica | `C:\Users\Sama Contabilidade\Desktop\portal-sama-api.zip` contém `portal-sama-api/.env`. | Segredos podem ter sido empacotados e compartilhados fora do Git. | Remover o ZIP, recriar pacote sem `.env` e rotacionar segredos potencialmente expostos. |
| Crítica | Escopo de leitura de clientes permite que qualquer perfil com `clients.read` liste/veja qualquer cliente. | Vazamento de cadastro, CNPJ, contato/localização e contadores operacionais entre departamentos. | Corrigir `ClientsService` para respeitar responsável/departamento/atribuições e adicionar testes negativos. |
| Alta | E2E da API falha em Web Push/CSRF no ambiente atual. | Gate automatizado de release não fecha. | Isolar `.env` nos testes ou zerar `COOKIE_DOMAIN` no setup E2E; manter cookie seguro em produção. |
| Alta | E2E web falha em duas validações da Home/Acessórias. | Gate visual/funcional do front não fecha. | Atualizar expectativas/mocks ou corrigir UI se os textos esperados forem contrato de produto. |
| Alta | Banco, migrações, RBAC seed e usuários privilegiados não foram provados porque MySQL não respondeu localmente. | Não há evidência de cutover seguro em ambiente alvo. | Rodar readiness/migrate/status/seed em EasyPanel ou homologação com DB real. |
| Alta | ClamAV não disponível e `SAMA_UPLOAD_SCAN_MODE` está em `best_effort`. | Uploads podem passar sem antivírus externo se a infraestrutura não estiver pronta. | Validar ClamAV e usar modo `strict` antes de produção. |

## Evidências executadas

| Área | Comando/validação | Resultado |
| --- | --- | --- |
| Ambiente | `node -v` | `v24.15.0` |
| API | `npm.cmd run prisma:validate` | Passou |
| API | `npm.cmd run lint` | Passou |
| API | `npm.cmd run build` | Passou |
| API | `npm.cmd test -- --runInBand` | 44 suites, 264 testes passaram |
| API | `npm.cmd run test:e2e` | Falhou: 2 testes Web Push/CSRF; 134 passaram |
| API | `npm.cmd audit --audit-level=moderate` | 1 vulnerabilidade moderada em `qs` |
| API | `npm.cmd run prisma:migrate:status` | Falhou: DB MySQL indisponível |
| API | `node -r dotenv/config scripts/validate-operational-readiness.js --soft` | Falhou em DB; avisos de ClamAV e backup/restore |
| Web | `npm.cmd run lint` | Passou |
| Web | `npm.cmd run build` | Passou |
| Web | `npm.cmd test -- --runInBand` | 9 testes contratuais passaram |
| Web | `npm.cmd run test:e2e` | Falhou: 2 testes da Home; 11 passaram, 1 skip |
| Web | `npm.cmd audit --audit-level=moderate` | 0 vulnerabilidades |
| Local smoke | API `GET /api-v2/health` | 200, `database: not_checked`, `storage: up` |
| Local smoke | Web `GET /login` via Vite | 200 |
| Artefatos | ZIPs no Desktop | API ZIP contém `.env`; web ZIP contém apenas `.env.example` |

## O que está forte

- Stack coerente: NestJS + Prisma + MySQL na API; React + Vite + TypeScript no web.
- Autenticação tem bons fundamentos: access token em memória, refresh token em cookie HttpOnly, rotação/revogação de refresh, CSRF double-submit, auditoria.
- API usa `helmet`, CORS com credenciais, `ValidationPipe` com whitelist/forbid, throttle global e filtros/interceptores.
- Uploads têm validação de extensão/MIME/magic bytes, quarentena, regras estáticas e caminho privado.
- Documentos, certificados, contratos, onboarding, notificações e Acessórias têm boa cobertura unitária/contratual.
- Pipeline Acessórias tem rate limit, retry/backoff, lock, checkpoint, divergências e aviso de backfill.

## Limitações desta auditoria

- Não houve login real com usuários DEV/Gestor/Colaborador/Cliente por falta de credenciais e banco local funcional.
- Não houve homologação contra EasyPanel, domínio real, HTTPS real, MySQL real, Acessórias real, Web Push com VAPID real ou navegador real autenticado.
- As evidências E2E web usam mocks; elas são úteis para contrato de interface, mas não substituem homologação operacional.

## Arquivos gerados

- `AUDITORIA-DEPLOY-01-QA-FUNCIONAL.md`
- `AUDITORIA-DEPLOY-02-SEGURANCA.md`
- `AUDITORIA-DEPLOY-03-FRONTEND-UI-UX.md`
- `AUDITORIA-DEPLOY-04-BACKEND-API-INTEGRACOES.md`
- `AUDITORIA-DEPLOY-05-PLANO-CORRECAO-CHECKLIST-PRODUCAO.md`

