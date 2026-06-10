# Auditoria Deploy - Continuação com Docker, Migrations e E2E

Data da continuação: 10/06/2026  
Escopo: execução local assistida com Docker, banco MySQL isolado, migrations, E2E API/Web e estimativa do que falta para concluir a auditoria local.  
Base: `Testes-da-aplicação-DEPLOY.md`, `AUDITORIA-DEPLOY-00` a `AUDITORIA-DEPLOY-11`.

## Resumo executivo desta rodada

Foi possível avançar a análise local usando Docker. O Docker Desktop já estava ativo e foi criado um MySQL isolado para auditoria:

- Container: `portal-sama-audit-mysql`
- Imagem: `mysql:8.4`
- Porta local: `127.0.0.1:3308`
- Banco: `portal_sama_audit`
- Uso: apenas auditoria local, sem banco real e sem alterar `.env`.

O teste revelou um bloqueio novo e objetivo: **as migrations não aplicam até o fim em banco MySQL limpo**. A migration `20260609133000_add_acessorias_company_obligation_catalog` falha por nome de índice/constraint acima do limite de 64 caracteres do MySQL.

Com isso, a análise local de banco/RBAC/bootstrap não pode ser finalizada sem corrigir a migration.

## Quanto falta para terminar localmente

### Situação atual

**A auditoria local está aproximadamente 80% concluída.**

O que já está bem coberto localmente:

- leitura e rastreabilidade da documentação;
- build/lint de API e Web;
- testes unitários da API;
- testes contratuais do Web;
- análise de segurança/autorização por código;
- análise de ZIPs/segredos;
- análise de Web Push;
- análise de banco/migrations por código;
- execução real de Docker/MySQL isolado;
- Playwright com screenshots/traces;
- identificação dos bloqueios principais.

O que falta para concluir a parte local:

| Bloco local restante | Status | Estimativa após correção prévia |
| --- | --- | ---: |
| Corrigir migration quebrada e reaplicar em banco limpo | Bloqueado por código | 30-60 min |
| Rodar `prisma:seed` e `prisma:bootstrap-admin` no banco isolado | Bloqueado pela migration | 10-20 min |
| Rodar readiness completo contra banco isolado | Bloqueado pela migration; ClamAV ainda será limitação local | 10-20 min |
| Ajustar isolamento do E2E API para não carregar `COOKIE_DOMAIN` do `.env` | Pendente | 15-30 min |
| Atualizar contrato E2E Web da Home ou ajustar UI se os textos antigos forem oficiais | Pendente | 20-45 min |
| Reexecutar bateria final local | Pendente | 45-90 min |
| Atualizar consolidação final dos relatórios | Pendente | 30-45 min |

Estimativa prática: **3 a 5 horas de trabalho local após corrigir os bloqueios de código/teste**.

Sem corrigir a migration, a auditoria local não chega a 100%, porque não há como provar migrations, seed, RBAC, bootstrap admin e login real em banco limpo.

## Achado novo: migration inválida para MySQL

### DB-MIG-01 - Nome de índice/constraint excede limite do MySQL

Severidade: **Bloqueante**.

Evidência:

- Com MySQL `8.4` em Docker e banco limpo, `npm.cmd run prisma:migrate:deploy` aplicou migrations anteriores e falhou em `20260609133000_add_acessorias_company_obligation_catalog`.
- Erro Prisma: `P3018`.
- Erro MySQL: `1059`.
- Mensagem: `Identifier name 'acessorias_company_obligation_catalog_client_id_normalized_name_key' is too long`.
- O nome tem 67 caracteres, acima do limite de 64 do MySQL.

Arquivo afetado:

- `portal-sama-api/prisma/migrations/20260609133000_add_acessorias_company_obligation_catalog/migration.sql`

Trecho afetado:

```sql
UNIQUE INDEX `acessorias_company_obligation_catalog_client_id_normalized_name_key`(`client_id`, `normalized_name`)
```

Relação com o schema:

- `portal-sama-api/prisma/schema.prisma:293` define `AcessoriasCompanyObligationCatalog`.
- `portal-sama-api/prisma/schema.prisma:312` usa `@@unique([clientId, normalizedName])` sem `map`, fazendo o Prisma gerar nome longo.
- O código usa o compound unique em `acessorias-registrations.service.ts:969` a `:974` via `clientId_normalizedName`.

Impacto:

- Banco novo não consegue aplicar todas as migrations.
- `20260609170000_add_acessorias_pipeline_runtime` não é aplicada porque a migration anterior fica em estado falho.
- `prisma:seed`, `prisma:bootstrap-admin`, readiness completo e homologação local com login real ficam bloqueados.
- Em ambiente EasyPanel com banco existente, o risco depende do estado atual do banco. Se essa migration ainda não foi aplicada, o deploy também tende a falhar.

Correção recomendada:

1. Definir nome curto no schema:

```prisma
@@unique([clientId, normalizedName], map: "uq_acess_catalog_client_norm")
```

2. Ajustar a migration SQL correspondente para usar o mesmo nome curto.
3. Recriar banco limpo de auditoria e rodar:

```powershell
npm.cmd run prisma:migrate:deploy
npm.cmd run prisma:migrate:status
npm.cmd run prisma:seed
npm.cmd run prisma:bootstrap-admin
node scripts/validate-operational-readiness.js --soft
```

4. Se algum ambiente real já registrou a migration como falha em `_prisma_migrations`, seguir procedimento de recuperação do Prisma antes de tentar reaplicar.

## Evidência do banco Docker

| Verificação | Resultado |
| --- | --- |
| `docker version` | Docker Desktop ativo, engine Linux disponível. |
| `docker ps -a` antes | Havia `portal-sama-mysql` antigo parado; não foi reutilizado. |
| Novo container | `portal-sama-audit-mysql` criado e ficou `Up`. |
| `prisma:migrate:status` antes do deploy | 29 migrations pendentes em banco limpo. |
| `prisma:migrate:deploy` | Falhou em `20260609133000_add_acessorias_company_obligation_catalog`. |
| `_prisma_migrations` no banco isolado | Migrations anteriores finalizadas; migration de catálogo ficou com `finished_at = NULL` e logs do erro 1059. |

## E2E API: causa refinada

### QA-API-01 - E2E API continua vermelho por isolamento incompleto de `.env`

Severidade: **Alta**.

Evidência:

- `npm.cmd run test:e2e` continua com 134 testes passando e 2 falhando.
- Falhas:
  - `POST /api-v2/notifications/push/subscribe` com CSRF válido esperava 200 e recebeu 403.
  - `POST /api-v2/notifications/push/test` com CSRF válido esperava 200 e recebeu 403.
- `test/app.e2e-spec.ts:14` a `:27` configura várias variáveis de teste, mas não limpa `COOKIE_DOMAIN`, `COOKIE_SECURE` nem `COOKIE_SAME_SITE`.
- `src/app.module.ts:33` a `:36` carrega `.env.local` e `.env`.
- `src/modules/auth/auth.controller.ts:211` a `:227` usa `COOKIE_DOMAIN` nas opções de cookie.

Reprodução defensiva:

- Uma chamada manual mínima, dentro do mesmo app Nest e com `process.env.COOKIE_DOMAIN = ''` antes de importar `AppModule`, retornou `200` em `/api-v2/notifications/push/test` com `reason: push_disabled`.
- No PowerShell, `$env:COOKIE_DOMAIN = ''` remove a variável do ambiente, permitindo que o `.env` volte a ser carregado pelo Nest. Isso explica por que tentar limpar via shell não fechou o E2E.

Impacto:

- O endpoint funciona em condição local isolada, mas a suíte oficial continua falhando.
- O gate automatizado segue vermelho até o próprio setup E2E neutralizar `COOKIE_DOMAIN`.

Correção recomendada:

1. No `beforeAll` de `test/app.e2e-spec.ts`, definir explicitamente:

```ts
process.env.COOKIE_DOMAIN = '';
process.env.COOKIE_SECURE = 'false';
process.env.COOKIE_SAME_SITE = 'lax';
```

2. Alternativa mais robusta: usar `ConfigModule.forRoot({ ignoreEnvFile: true })` em módulo de teste ou carregar `.env.test`.
3. Adicionar `expect` do corpo quando o teste receber `403`, para preservar o motivo em logs futuros.

## E2E Web: causa refinada

### QA-WEB-01 - Testes da Home estão desalinhados com a UI atual

Severidade: **Alta** para gate, **Média** para produto se a UI atual for aceita.

Evidência:

- `npm.cmd run test:e2e` no Web continua com 11 testes passando, 1 skip e 2 falhas.
- Contexto Playwright mostra que a Home renderiza e tem conteúdo, mas os textos esperados não existem mais.

Falha admin:

- Teste espera `Integracao`.
- UI atual mostra `Base local`, `Atrasos`, `Pendencias`, `Clientes` e o bloco `Diagnostico Acessorias`.

Falha colaborador:

- Teste espera heading `Entregue pelo Acessorias`.
- UI atual mostra heading `Baixas sincronizadas`, com item `EFD Contribuicoes`.

Arquivos envolvidos:

- `portal-sama-web/tests/e2e/smoke.spec.ts:1316` a `:1346`
- `portal-sama-web/src/pages/home/HomePage.tsx`

Impacto:

- O gate Playwright não fecha.
- A evidência aponta mais para contrato de teste desatualizado do que para tela quebrada, mas a decisão precisa ser de produto: manter textos atuais ou voltar ao contrato antigo.

Correção recomendada:

1. Se a UI atual estiver correta, atualizar os asserts para `Base local` e `Baixas sincronizadas`.
2. Se os textos antigos forem contrato de produto, ajustar a Home para renderizar `Integracao` e `Entregue pelo Acessorias`.
3. Mockar/neutralizar `/api-v2/notifications/stream` no Playwright, pois ainda aparecem avisos `ECONNREFUSED` de SSE quando a API não está ativa.

## O que agora dá para considerar fechado localmente

| Item | Status | Observação |
| --- | --- | --- |
| Docker disponível | OK | Docker Desktop já estava rodando. |
| MySQL isolado de auditoria | OK | Container novo criado e acessível. |
| Prova de migrations em banco limpo | Reprovada com evidência | Bloqueio real em migration. |
| Diagnóstico E2E API Web Push/CSRF | Refinado | Endpoint passa em reprodução manual; setup E2E oficial continua contaminado por `.env`. |
| Diagnóstico E2E Web Home | Refinado | Falha por expectativa textual desalinhada com UI atual. |

## O que ainda não pode ser fechado localmente

| Item | Motivo |
| --- | --- |
| Seed RBAC em banco limpo | Migration falha antes. |
| Bootstrap admin em banco limpo | Migration falha antes. |
| Readiness DB/migrations/RBAC/privileged-users | Migration falha antes. |
| Login real local com usuário bootstrap | Bootstrap não pode ser executado ainda. |
| Homologação DEV/Gestor/Colaborador/Cliente real | Exige banco íntegro, usuários e dados. |
| ClamAV strict | Scanner não está disponível localmente. |
| Web Push real | Exige VAPID real, HTTPS/navegador real autenticado e banco íntegro. |
| Acessórias real | Exige token e ambiente externo real. |

## Decisão atualizada

**Não pronto para produção.**  
**Não pronto para usuários reais.**  
**Auditoria local ainda não finalizável em 100% até corrigir a migration `DB-MIG-01`.**

Depois da correção da migration e dos ajustes de E2E, a auditoria local pode chegar perto de 95%. Os 5% restantes dependem naturalmente de ambiente real: HTTPS, EasyPanel, domínio, ClamAV, VAPID, Acessórias real, backup/restore e usuários reais por perfil.

