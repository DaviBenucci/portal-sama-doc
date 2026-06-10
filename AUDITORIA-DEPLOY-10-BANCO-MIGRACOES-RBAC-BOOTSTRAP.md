# Auditoria Deploy - Banco, Migrations, RBAC e Bootstrap Admin

Data da análise: 10/06/2026

Escopo complementar: continuar a auditoria sobre execução local, Prisma, migrations, seed de RBAC e `npm run prisma:bootstrap-admin`, considerando o roteiro `C:\Users\Sama Contabilidade\Desktop\Rodar-local-Portal-sama.md` e os scripts reais da API.

## Veredito

Status: **parcialmente validado em código/testes, mas No-Go em produção até prova em banco real**.

Os scripts existem e têm bons controles: schema Prisma válido, catálogo RBAC testado, seed idempotente por `upsert`, bootstrap admin por variáveis de ambiente, senha com hash bcrypt e auditoria. Porém a etapa não está pronta para produção porque migrations, seed, readiness, usuário privilegiado e login real ainda não foram provados no MySQL alvo.

## O que foi verificado agora

| Item | Resultado | Evidência |
| --- | --- | --- |
| Script `prisma:seed` | Existe e chama wrapper runtime | `portal-sama-api/package.json:27`, `scripts/run-prisma-runtime-script.js` |
| Script `prisma:bootstrap-admin` | Existe e chama wrapper runtime | `portal-sama-api/package.json:28`, `prisma/bootstrap-admin.ts` |
| Seed RBAC | Idempotente para permissões/roles padrão | `prisma/seed.ts` usa `permission.upsert` e `role.upsert` |
| Bootstrap admin | Usa env, limita role a `ADMIN`/`DEV`, registra auditoria e preserva senha por padrão | `prisma/bootstrap-admin.ts` |
| Baseline de banco existente | Protegido por confirmação explícita | `scripts/prisma-baseline-existing-database.js` |
| Readiness | Valida env, DB, schema, migrations, RBAC, usuários privilegiados, storage, ClamAV e backup/rollback | `scripts/validate-operational-readiness.js` |
| Migrations versionadas | 29 diretórios de migration encontrados | `portal-sama-api/prisma/migrations` |

## Testes não destrutivos executados

| Comando | Resultado |
| --- | --- |
| `npm.cmd run prisma:validate` | Passou; schema Prisma válido |
| `npm.cmd test -- --runInBand src/modules/rbac/default-rbac.spec.ts src/config/env.schema.spec.ts` | Passou: 2 suites, 8 testes |

Não executei `prisma:migrate:deploy`, `prisma:seed`, `prisma:bootstrap-admin` nem `prisma:migrate:baseline-existing`, porque estes comandos alteram estado do banco e a solicitação atual foi de análise.

## Achados desta etapa

### DB-BOOT-01 - Roteiro local contém credencial demo em texto puro

Severidade: **Alta**.

O arquivo `C:\Users\Sama Contabilidade\Desktop\Rodar-local-Portal-sama.md` contém usuário e senha demo em texto puro. Não reproduzo os valores aqui por segurança.

Impacto: se esse roteiro for compartilhado, anexado, sincronizado em nuvem ou reaproveitado em homologação/produção, a credencial pode virar acesso real conhecido.

Correção recomendada:

1. Remover usuário/senha do Markdown.
2. Substituir por placeholders sem segredo real.
3. Usar variáveis de ambiente apenas no shell/secret manager.
4. Se a credencial demo já foi usada em qualquer banco real, redefinir senha ou remover usuário.

### DB-BOOT-02 - Seed RBAC sobrescreve permissões das roles padrão

Severidade: **Média**.

`seedRbac()` apaga e recria os vínculos de permissões para cada role padrão. Isso é adequado para manter o catálogo oficial, mas pode desfazer ajustes manuais feitos diretamente no banco em `ADMIN`, `DEV`, `MANAGER`, `CLIENT`, `DEPARTMENT`, `TI`, `ACCOUNTING`, `LEGALIZATION` e `AUDITOR`.

Correção recomendada: tratar as roles padrão como catálogo controlado por código. Customizações de cliente/ambiente devem usar roles próprias ou mudança versionada no catálogo.

### DB-BOOT-03 - Baseline exige disciplina operacional

Severidade: **Alta**.

`prisma:migrate:baseline-existing` é protegido por `SAMA_PRISMA_BASELINE_EXISTING_DATABASE=true`, o que reduz risco de execução acidental. Ainda assim, marcar baseline em banco errado ou sem backup pode impedir diagnóstico futuro de schema drift.

Correção recomendada: executar baseline somente após backup/snapshot, conferência do banco alvo e registro de evidência. Nunca rodar `prisma db push` em produção.

### DB-BOOT-04 - Roteiro local ainda não prova banco real

Severidade: **Alta**.

O código está pronto para validar banco real, mas o ambiente local anterior não respondeu ao MySQL. Portanto ainda não há evidência de:

- todas as migrations aplicadas;
- seed RBAC persistido;
- usuário `ADMIN`/`DEV` ativo;
- login real pós-bootstrap;
- readiness sem falhas.

## Ordem segura para homologação

1. Fazer backup/snapshot do banco alvo.
2. Confirmar `DATABASE_URL` aponta para homologação correta.
3. Rodar `npm.cmd run prisma:migrate:status`.
4. Se for banco existente sem `_prisma_migrations`, executar baseline controlado somente após backup.
5. Rodar `npm.cmd run prisma:migrate:deploy`.
6. Rodar `npm.cmd run prisma:seed`.
7. Rodar `npm.cmd run prisma:bootstrap-admin` com `SAMA_BOOTSTRAP_ADMIN_*` via shell/secret manager.
8. Rodar `npm.cmd run ops:readiness`.
9. Validar login real, CSRF, refresh, permissões e usuário privilegiado.
10. Registrar evidências sem expor segredos.

## Decisão da continuidade

Esta frente avançou em análise e testes seguros. O bloqueio de produção permanece: **banco/migrations/RBAC/bootstrap só viram Go após execução e evidência em ambiente real**.

