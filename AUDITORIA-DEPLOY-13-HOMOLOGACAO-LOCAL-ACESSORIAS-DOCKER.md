# AUDITORIA DEPLOY 13 - Homologação local Acessórias com Docker

Data: 10/06/2026  
Escopo: continuidade dos testes descritos em `Testes-da-aplicação-DEPLOY.md`, com foco na integração Acessórias, API local e homologação em MySQL via Docker.

## 1. Objetivo desta etapa

Validar localmente, com Docker e banco zerado, se as rotas da integração Acessórias conseguem:

- autenticar na API do Portal Sama;
- chamar a API externa da Acessórias com token configurado no backend;
- executar prévias e dry-runs sem persistência indevida;
- persistir empresas, catálogo técnico, entregas e reconciliação em MySQL;
- manter scheduler desligado durante testes manuais;
- confirmar migrations, RBAC, readiness e build após os ajustes.

Nenhum token, senha transitória, nome de cliente, CNPJ, email ou payload bruto da Acessórias foi registrado neste relatório.

## 2. Ambiente local usado

- Docker Desktop ativo.
- Container MySQL isolado: `portal-sama-audit-mysql`.
- Banco: `portal_sama_audit`.
- Porta local MySQL: `127.0.0.1:3308`.
- API local: `http://localhost:3100/api-v2`.
- Resultado sanitizado: `portal-sama-api/.ai-tests/acessorias-api-results-clean.json`.
- Logs da API: `portal-sama-api/.ai-tests/api-server/stdout.log` e `stderr.log`.

Flags importantes usadas na API durante a homologação:

- `ACESSORIAS_MAX_PAGES=1`.
- `ACESSORIAS_RATE_LIMIT_PER_MINUTE=30`.
- `ACESSORIAS_RETRY_ATTEMPTS=0`.
- `ACESSORIAS_INCREMENTAL_SYNC_ENABLED=false`.
- `ACESSORIAS_SCHEDULER_ENABLED=false`.
- `ACESSORIAS_COMPANY_CATALOG_SYNC_ENABLED=false`.
- `ACESSORIAS_RECONCILIATION_ENABLED=false`.

Observação: `ACESSORIAS_INCREMENTAL_SYNC_ENABLED=false` é necessário. Apenas `ACESSORIAS_SCHEDULER_ENABLED=false` não desligava o incremental, porque o código prioriza `ACESSORIAS_INCREMENTAL_SYNC_ENABLED`.

## 3. Correções feitas no código durante a homologação

### 3.1 Índice MySQL com nome acima de 64 caracteres

Ao aplicar migrations em MySQL limpo, a migration `20260609133000_add_acessorias_company_obligation_catalog` falhava por nome de índice longo demais.

Correção aplicada:

- `prisma/schema.prisma`: `@@unique([clientId, normalizedName], map: "uq_acess_catalog_client_norm")`.
- `prisma/migrations/20260609133000_add_acessorias_company_obligation_catalog/migration.sql`: nome do índice alterado para `uq_acess_catalog_client_norm`.

Resultado: `prisma migrate deploy` passou com 29 migrations aplicadas.

### 3.2 Filtro JSON incompatível com MySQL/Prisma

Durante os testes reais da Acessórias, as rotas de cadastro falharam com erro Prisma/MySQL:

```text
Argument `path`: Invalid value provided. Expected String, provided (String, String).
```

Causa:

- O serviço usava `metadata: { path: ['acessorias', 'externalId'], equals: ... }`.
- Em MySQL, o Prisma espera JSONPath string, por exemplo `$.acessorias.externalId`.

Correção aplicada:

- `src/modules/integrations/acessorias/acessorias-registrations.service.ts`.
- Troca para `metadata: { path: '$.acessorias.externalId', equals: ... }`.
- Teste unitário acrescentado em `acessorias-registrations.service.spec.ts` para travar esse comportamento.

Resultado: as rotas de preview, dry-run, sync de clientes e sync de catálogo passaram no banco MySQL local.

## 4. Validações executadas

### 4.1 Build e testes focados

Comandos executados no `portal-sama-api`:

- `npm run build`: passou.
- `npm test -- --runInBand src/modules/integrations/acessorias/acessorias-registrations.service.spec.ts src/modules/integrations/acessorias/acessorias-deliveries.service.spec.ts src/modules/rbac/default-rbac.spec.ts`: passou.

Resultado:

- 3 suites passaram.
- 19 testes passaram.

### 4.2 Banco, migrations, seed e bootstrap

O banco `portal_sama_audit` foi recriado do zero com `utf8mb4_unicode_ci`.

Resultado inicial antes das chamadas API:

| Tabela/contador | Total |
|---|---:|
| clients | 0 |
| users | 1 |
| acessorias_company_obligation_catalog | 0 |
| acessorias_deliveries | 0 |
| acessorias_delivery_sync_runs | 0 |
| acessorias_sync_runs | 0 |
| acessorias_fiscal_divergences | 0 |
| acessorias_responsible_aliases | 0 |

`prisma migrate status`:

- 29 migrations encontradas.
- Database schema up to date.

`validate-operational-readiness.js --soft`:

- Passou com warnings.
- Database: passou.
- Production schema: passou.
- Migrations: passou.
- RBAC: passou, com 91 permissões e 9 roles.
- Privileged users: passou.
- Storage: passou.
- Warnings restantes: ClamAV não validado em modo strict, backup/rollback ainda precisam de prova fora do app container, `SAMA_UPLOAD_SCAN_MODE` ainda não está strict.

## 5. Chamadas API Acessórias testadas

Todas as chamadas abaixo foram autenticadas com usuário bootstrap local e CSRF válido quando necessário.

| Chamada | Status | Resultado |
|---|---:|---|
| `GET /health` | 200 | API respondeu. |
| `GET /auth/csrf` | 200 | CSRF emitido. |
| `POST /auth/login` | 200 | Login local OK. |
| `GET /auth/me` | 200 | Usuário `DEV`, 91 permissões. |
| `GET /integrations/acessorias/scheduler/status` antes | 200 | Scheduler desligado. |
| `GET /integrations/acessorias/rate-limiter/status` antes | 200 | Fila vazia, sem falhas. |
| `GET /integrations/acessorias/sync-runs?take=10` antes | 200 | Total 0. |
| `GET /integrations/acessorias/deliveries?take=10` antes | 200 | Total 0. |
| `GET /integrations/acessorias/home-summary?profile=admin` antes | 200 | Cache local vazio. |
| `GET /integrations/acessorias/registrations/preview?entity=clients` | 200 | Prévia retornou 20 itens. |
| `POST /integrations/acessorias/registrations/sync` com `dryRun=true` | 201 | Dry-run retornou 20 itens. |
| `POST /integrations/acessorias/registrations/sync` com persistência | 201 | Persistiu 20 clientes e catálogo relacionado. |
| `POST /integrations/acessorias/companies/catalog/sync` | 201 | Sync técnico `SUCCESS`, 20 fetched, 20 updated. |
| `GET /integrations/acessorias/deliveries/preview` | 200 | 50 entregas normalizadas, 19 clientes na prévia. |
| `POST /integrations/acessorias/deliveries/sync` | 201 | Sync `SUCCESS`, 50 fetched, 50 created. |
| `POST /integrations/acessorias/reconciliation/run` | 201 | Reconciliation `SUCCESS`, 892 catálogo checked, 50 entregas checked. |
| `GET /integrations/acessorias/sync-runs?take=10` depois | 200 | 3 runs técnicos, todos `SUCCESS`. |
| `GET /integrations/acessorias/home-summary?profile=admin` depois | 200 | 50 entregas no resumo local, `data_stale=false`. |

Status final do rate limiter:

- `completed_total=6`.
- `failed_total=0`.
- `rate_limited_total=0`.
- fila `queued=0`, `active=0`.

## 6. Evolução dos dados no banco local

| Etapa | clients | catálogo | entregas | delivery runs | sync runs | divergências | aliases |
|---|---:|---:|---:|---:|---:|---:|---:|
| Antes das chamadas | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| Após sync de cadastros | 20 | 892 | 0 | 0 | 0 | 0 | 25 |
| Após sync de catálogo | 20 | 892 | 0 | 0 | 1 | 0 | 25 |
| Após sync de entregas | 20 | 892 | 50 | 1 | 2 | 0 | 34 |
| Após reconciliação | 20 | 892 | 50 | 1 | 3 | 100 | 34 |

Leitura: localmente, a integração conseguiu consumir a API externa da Acessórias, normalizar dados e persistir no MySQL. Isso aumenta bastante a chance de funcionar no banco real, desde que as mesmas migrations/correções sejam aplicadas e os segredos/envs estejam corretos no ambiente final.

## 7. Conclusão desta etapa

Homologação local Acessórias + MySQL Docker: concluída com sucesso.

O que foi provado localmente:

- migrations completas em MySQL limpo;
- seed e bootstrap funcionando;
- API sobe e autentica;
- RBAC atual possui permissões esperadas;
- rotas Acessórias principais funcionam com chamada externa real;
- persistência local de clientes, catálogo, entregas e reconciliação funciona;
- scheduler pode ficar desligado para teste manual;
- rate limit não foi atingido na bateria controlada.

## 8. Pendências antes de considerar deploy pronto

Ainda faltam pontos importantes já levantados nas auditorias anteriores:

1. Corrigir segurança do `home-summary`: hoje o perfil vem por query string e pode ampliar escopo indevidamente.
2. Revisar permissão ampla em `clients.read`, já registrada como risco crítico em auditorias anteriores.
3. Remover `.env` sensível do ZIP `portal-sama-api.zip` e tratar esse arquivo como incidente de segredo.
4. Atualizar/rodar E2E da API após correções de CSRF/cookie e contaminação por `.env`.
5. Atualizar E2E do frontend com os textos/fluxos atuais.
6. Validar ClamAV em produção com `SAMA_UPLOAD_SCAN_MODE=strict`.
7. Provar backup, restore drill e rollback fora do app container.
8. Replicar no ambiente real as correções de migration e JSONPath antes de aplicar migrations.

## 9. Quanto falta localmente

Estimativa após esta etapa:

- Análise local Acessórias + banco Docker: 100% concluída.
- Análise local geral de API/build/migrations/RBAC/readiness: aproximadamente 90% concluída.
- Prontidão total para deploy: aproximadamente 75% a 80%, porque ainda restam correções de segurança, E2E e provas operacionais de produção.

O próximo melhor passo é corrigir os dois pontos críticos de autorização/escopo (`home-summary` e `clients.read`) e depois repetir a bateria curta de testes focados + E2E.
