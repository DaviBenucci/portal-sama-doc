# Acompanhamento Codex - Implementacao fim a fim

Data do ciclo: 2026-06-19

Guia controlador: `20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md`

## Regra de execucao

Nenhuma fase seguinte deve avancar enquanto a fase anterior nao estiver `CONCLUIDA`, com evidencia tecnica registrada. Se uma validacao falhar, a falha vira sub-etapa corretiva antes da continuidade.

Status aceitos neste acompanhamento:

- `PENDENTE`
- `EM_EXECUCAO`
- `BLOQUEADA`
- `CONCLUIDA`

## Linha de base dos repositorios

| Repositorio | Caminho local | Branch | Commit | Status inicial |
| --- | --- | --- | --- | --- |
| `portal-sama-api` | `C:\Users\Sama Contabilidade\Desktop\portal-sama-api` | `main` | `239f534` | Sem alteracoes pendentes |
| `portal-sama-web` | `C:\Users\Sama Contabilidade\Desktop\portal-sama-web` | `main` | `646765e` | Sem alteracoes pendentes |
| `portal-sama-docs` | `C:\Users\Sama Contabilidade\Desktop\portal-sama-docs` | `main` | `47ca98c` | Sem alteracoes pendentes antes deste arquivo |
| `portal-sama` legado | `C:\Users\Sama Contabilidade\Desktop\portal-sama` | Consulta historica | N/A | Somente leitura para semantica |

## Escopo congelado

Este ciclo continua somente nos repositorios `portal-sama-api`, `portal-sama-web` e `portal-sama-docs`.

O legado `portal-sama` pode ser lido para entendimento semantico, mas nao deve receber implementacao nova.

Alteracoes ja existentes antes deste checkpoint formal e consideradas parte da linha de base:

- Links publicos novos para assinatura, proposta e documentos.
- Redirecionamentos das rotas publicas legadas para as rotas React novas.
- Rate limit local por token publico nos fluxos de assinatura, proposta e documentos.
- Escopo de `MANAGER` para gestao de links publicos de documentos.
- Smokes web de tokens publicos expirados/revogados e redirects legados.
- `.dockerignore` reforcado em API e Web.

## Decisoes de produto congeladas dos docs 15-19

- Backend e banco avancam antes de telas finais de frontend.
- Frontend so deve ir alem de smoke quando o contrato de API estiver validado.
- Painel do cliente deve reaproveitar a semantica do legado por abas: Geral, Responsaveis, Acessos, Documentos, Certificados e Vida da empresa.
- ZapSign legado e requisito funcional, nao sugestao.
- Assinatura interna e assinatura ZapSign devem conviver como providers distintos.
- Para contratos ZapSign, a assinatura publica interna deve ser bloqueada.
- Integracao Acessorias deve buscar obrigacoes/listas a partir do ano atual e colaboradores/responsaveis a partir do mes atual; dados antigos retornados pela API externa devem ser filtrados antes de persistir.
- Segredos reais nao devem ser lidos, impressos, versionados nem empacotados.

## Matriz de evidencia

Cada etapa concluida deve registrar:

- Fase e etapa.
- Status anterior e final.
- Arquivos alterados.
- Comandos executados.
- Resultado dos comandos.
- Testes executados.
- Pendencias.
- Observacoes de seguranca.

## Fase 0 - Preparacao, escopo e regra rigida de execucao

Status final da fase: `CONCLUIDA`

| Etapa | Status final | Evidencia |
| --- | --- | --- |
| 0.1 Criar checkpoint de execucao | `CONCLUIDA` | Este arquivo foi criado em `docs/20-ACOMPANHAMENTO-CODEX-FIM-A-FIM.md`. |
| 0.2 Registrar regra de avanco | `CONCLUIDA` | Regra de execucao registrada no topo deste arquivo. |
| 0.3 Inventariar repositorios | `CONCLUIDA` | Branches, commits e paths registrados na linha de base. |
| 0.4 Bloquear edicao fora do escopo | `CONCLUIDA` | Escopo limitado a API, Web e Docs; legado somente leitura. |
| 0.5 Ler documentos obrigatorios | `CONCLUIDA` | Docs 15, 16, 17, 18, 19 e guia 20 considerados para este ciclo. |
| 0.6 Validar artefatos sensiveis no pacote de trabalho | `CONCLUIDA` | Varredura por nomes de arquivos sensiveis executada sem abrir conteudo. `.env` real existe localmente em API e legado, mas esta ignorado no Git; API/Web tambem excluem `.env*`, dumps, backups, chaves e pacotes no `.dockerignore`. Nenhum segredo foi impresso. |
| 0.7 Definir matriz de evidencia | `CONCLUIDA` | Matriz registrada neste arquivo. |
| 0.8 Congelar decisoes de produto | `CONCLUIDA` | Decisoes dos docs 15-19 registradas acima. |

### Comandos da Fase 0

```powershell
git status --short
git rev-parse --abbrev-ref HEAD
git rev-parse --short HEAD
rg --files -g ".env*" -g "*.pem" -g "*.key" -g "*.p12" -g "*.pfx" -g "*.dump" -g "*.bak" -g "*.backup" -g "*.sqlite" -g "*.db" -g "*.zip" -g "*.tar" -g "*.tar.gz"
rg -n "\.env|dump|backup|pem|key|p12|pfx|zip|tar" .gitignore .dockerignore
rg -n "Fase 0|0\.6|segredo|secret|ZAPSIGN|ZapSign|Contrato|Acessórias" "20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md"
```

### Resultado da Fase 0

- `portal-sama-api`, `portal-sama-web` e `portal-sama-docs` estavam sem alteracoes pendentes no inicio da continuidade.
- `portal-sama-api` contem `.env.example` e `.env` local; `.env` e `.env.*` estao ignorados no Git e excluidos do contexto Docker.
- `portal-sama-web` contem `.env.example`; `.env` e `.env.*` estao ignorados no Git e excluidos do contexto Docker.
- `portal-sama-docs` nao retornou artefatos sensiveis na varredura por nomes.
- `portal-sama` legado contem `.env.example` e `.env` local; uso permitido somente para consulta semantica, sem empacotar arquivos.
- Nenhum valor de segredo foi aberto ou registrado.

## Proxima fase autorizada

Com a Fase 0 concluida, a continuidade deve seguir pela Fase 1 do guia, priorizando preparacao tecnica do backend/API antes de evoluir telas finais do frontend.

## Fase 1 - Ambiente base, banco limpo e configuracao inicial

Status final da fase: `CONCLUIDA`

Imprevisto corrigido: no primeiro teste, o Docker daemon `desktop-linux` nao
estava ativo. O Docker Desktop foi iniciado em segundo plano, o daemon respondeu,
o MySQL local subiu e a API foi validada contra o banco.

| Etapa | Status final | Evidencia |
| --- | --- | --- |
| 1.1 Validar versoes de runtime | `CONCLUIDA` | Node `v24.15.0`, npm `11.12.1`, Docker client/server `29.5.3`, Docker Compose `v5.1.4`, OS `Microsoft Windows NT 10.0.26200.0`, MySQL container `8.4.9`. `mysql --version` nao existe localmente; a validacao foi feita dentro do container. |
| 1.2 Criar/revisar Docker Compose inicial | `CONCLUIDA` | Criado `portal-sama-api/compose.yaml` com MySQL 8.4, volume persistente, rede local, usuario `portal_user` e charset/collation `utf8mb4`/`utf8mb4_0900_ai_ci`. `docker compose --env-file .env.example config` passou sem avisos. |
| 1.3 Criar `.env.example` da API | `CONCLUIDA` | `portal-sama-api/.env.example` recebeu variaveis ZapSign sem valores reais; `env.schema.ts` valida placeholders opcionais. |
| 1.4 Criar `.env.local.example` do web | `CONCLUIDA` | Criado `portal-sama-web/.env.local.example`; Web agora prefere `VITE_API_BASE_URL` e mantem fallback temporario para `VITE_API_URL`. `.gitignore` e `.dockerignore` reabrem somente o example. |
| 1.5 Subir banco limpo | `CONCLUIDA` | `docker compose --env-file .env.example up -d mysql` criou rede, volume e container; `docker compose --env-file .env.example ps` mostrou `portal-sama-mysql` `healthy`. |
| 1.6 Configurar usuario de banco com menor privilegio | `CONCLUIDA` | `SHOW GRANTS FOR CURRENT_USER()` retornou `portal_user` com privilegios no banco `portal_sama`, sem uso de root pela aplicacao. |
| 1.7 Validar charset/collation | `CONCLUIDA` | Consulta no MySQL retornou `@@character_set_server=utf8mb4` e `@@collation_server=utf8mb4_0900_ai_ci`. |
| 1.8 Criar rotina de reset local seguro | `CONCLUIDA` | README da API documenta `docker compose --env-file .env.example down` e `down -v`, com aviso para confirmar `localhost` e nao usar homologacao/producao. |
| 1.9 Validar health basico da API contra banco | `CONCLUIDA` | API subiu temporariamente em `localhost:3001`; `GET /api-v2/health` retornou `ok=true`, `database=up`, `storage=up`. Processo Nest temporario encerrado apos a validacao. |
| 1.10 Marcar fase apenas se banco estiver operacional | `CONCLUIDA` | MySQL local permanece `healthy`; Fase 1 concluida. |

### Arquivos alterados na Fase 1

- `portal-sama-api/compose.yaml`
- `portal-sama-api/.env.example`
- `portal-sama-api/src/config/env.schema.ts`
- `portal-sama-api/src/config/env.schema.spec.ts`
- `portal-sama-api/README.md`
- `portal-sama-web/.env.local.example`
- `portal-sama-web/.env.example`
- `portal-sama-web/.gitignore`
- `portal-sama-web/.dockerignore`
- `portal-sama-web/src/services/api.ts`
- `portal-sama-web/README.md`

### Comandos da Fase 1

```powershell
node --version
npm.cmd --version
docker --version
docker compose version
mysql --version
[System.Environment]::OSVersion.VersionString
docker compose config
Start-Process -FilePath "$Env:ProgramFiles\Docker\Docker\Docker Desktop.exe" -WindowStyle Hidden
docker info
docker compose --env-file .env.example config
docker compose --env-file .env.example up -d mysql
docker inspect --format '{{.State.Health.Status}}' portal-sama-mysql
docker compose --env-file .env.example exec -T mysql mysql -uportal_user -pportal_password portal_sama -e "SELECT @@version AS version, @@character_set_server AS charset_server, @@collation_server AS collation_server; SHOW GRANTS FOR CURRENT_USER();"
docker compose --env-file .env.example ps
npm.cmd run prisma:validate
npm.cmd test -- env.schema.spec.ts
npm.cmd run lint
npm.cmd run build
npm.cmd test
```

### Resultado da Fase 1

- `docker compose config` sem `--env-file` tentou ler o `.env` local da API e gerou avisos de interpolacao. Correcao aplicada: README passou a orientar sempre `docker compose --env-file .env.example ...` para nao tocar no `.env` real.
- `docker compose --env-file .env.example config`: passou.
- Docker daemon indisponivel no primeiro teste; Docker Desktop iniciado e `docker info` passou.
- `docker compose --env-file .env.example up -d mysql`: passou.
- MySQL health: `healthy`.
- MySQL runtime: versao `8.4.9`, charset `utf8mb4`, collation `utf8mb4_0900_ai_ci`.
- Usuario da aplicacao: `portal_user` com privilegios no banco local `portal_sama`.
- Health API local: `ok=true`, `database=up`, `storage=up`.
- API `npm.cmd run prisma:validate`: passou.
- API `npm.cmd test -- env.schema.spec.ts`: passou, 6 testes.
- API `npm.cmd run lint`: passou.
- API `npm.cmd run build`: passou.
- Web `npm.cmd run lint`: passou.
- Web `npm.cmd run build`: passou.
- Web `npm.cmd test`: passou, 12 testes de contrato.

### Estado operacional local apos a Fase 1

- Container `portal-sama-mysql` esta em execucao e `healthy`.
- Processo Nest temporario usado para health em `localhost:3001` foi encerrado.
- Nenhum segredo real foi aberto ou registrado; comandos usaram placeholders locais.

## Fase 2 - Prisma, schema e modelagem de dominio

Status final da fase: `CONCLUIDA`

### Mapa de modelos revisado

| Dominio | Modelos atuais | Decisao |
| --- | --- | --- |
| Usuarios e permissoes | `User`, `Role`, `Permission`, `UserRole`, `RolePermission`, `RefreshToken` | Mantidos; RBAC recebeu novas permissoes seed. |
| Clientes | `Client` | Mantido como raiz do painel; recebeu relacao com credenciais de acesso. |
| Departamentos e responsaveis | `Department`, `ClientDepartmentAssignment` | Mantidos como escopo operacional por cliente/departamento. |
| Documentos | `Document`, `DocumentRequirement`, `DocumentStatusHistory`, `PublicToken` | Mantidos; indices atuais cobrem cliente, status, departamento, token e datas. |
| Certificados | `DigitalCertificate` | Mantido com senha cifrada em `passwordCipher`. |
| Contratos | `Contract` | Recebeu `signatureProvider` e `signatureEnv` para consulta rapida e coexistencia `INTERNAL`/`ZAPSIGN`. |
| ZapSign | `ContractSignatureEnvelope`, `ContractSigner` | Criados conforme doc 18, com token externo, status normalizado, URLs externas e signatarios normalizados. |
| Acessos do cliente | `ClientAccessCredential`, `ClientAccessCredentialAudit` | Criados com `secretCipher`/`secretRef`; sem campo de segredo em claro; auditoria de revelacao sensivel prevista. |
| Acessorias | `AcessoriasCompanyObligationCatalog`, `AcessoriasDelivery`, aliases, sync runs, locks, mappings, fiscal runs/divergences | Mantidos; regra temporal segue docs 12/20. |
| Vida da empresa | `CompanyHistoryEntry`, `CompanyLifeEntry` | Mantidos nesta fase para preservar semantica legada; endpoint alvo continua no modulo `Managers`. |
| Auditoria | `AuditLog`, mais auditoria especifica de credenciais | Mantido; credenciais ganharam trilha dedicada para eventos sensiveis. |

| Etapa | Status final | Evidencia |
| --- | --- | --- |
| 2.1 Rodar validacao Prisma atual | `CONCLUIDA` | `npm.cmd run prisma:validate` passou. |
| 2.2 Revisar modelos existentes | `CONCLUIDA` | Mapa de modelos registrado acima. |
| 2.3 Modelar provider de assinatura | `CONCLUIDA` | Criados enums `ContractSignatureProvider`, `ContractSignatureEnv`, `ContractExternalSignatureStatus`; `Contract` recebeu provider/env; migration `20260619143000_add_contract_signature_provider`. |
| 2.4 Modelar signatarios de contrato | `CONCLUIDA` | Criado `ContractSigner` com campos normalizados, indices por contrato, envelope, email, CPF e token externo. |
| 2.5 Modelar acessos do cliente | `CONCLUIDA` | Criados `ClientAccessCredential` e `ClientAccessCredentialAudit`; segredo fica em `secretCipher` ou `secretRef`, com auditoria para `REVEALED`. |
| 2.6 Revisar Vida da empresa | `CONCLUIDA` | Decisao registrada: manter `CompanyHistoryEntry`/`CompanyLifeEntry` por compatibilidade semantica nesta etapa. |
| 2.7 Ajustar indices criticos | `CONCLUIDA` | Adicionados indices para provider/env de contrato, provider/status de envelope, signedAt, signers, client access por cliente/status, departamento/status, sistema e arquivamento. |
| 2.8 Criar migration incremental | `CONCLUIDA` | Criadas migrations `20260619143000_add_contract_signature_provider` e `20260619150000_add_client_access_credentials`, pequenas e nao destrutivas. |
| 2.9 Testar migration em banco limpo | `CONCLUIDA` | `npm.cmd run prisma:migrate:deploy` aplicou 32 migrations no MySQL local limpo; status ficou up to date. |
| 2.10 Testar migration em banco com dados fake | `CONCLUIDA` | Base `portal_sama_phase2_fake` recebeu tabela antiga minima de contratos + contrato fake; migration ZapSign preservou o registro com `INTERNAL/BR`; base fake removida. Credencial fake validou FK, ciphertext e auditoria no banco local. |
| 2.11 Atualizar seed RBAC | `CONCLUIDA` | `DEFAULT_PERMISSIONS` recebeu ZapSign, acessos, vida da empresa e revelacao sensivel; `npm.cmd run prisma:seed` criou 8 novas permissoes no banco local; teste RBAC passou. |
| 2.12 Gerar Prisma Client | `CONCLUIDA` | `npm.cmd run prisma:generate` passou com Prisma Client `v6.19.3`. |

### Arquivos alterados na Fase 2

- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/prisma/migrations/20260619143000_add_contract_signature_provider/migration.sql`
- `portal-sama-api/prisma/migrations/20260619150000_add_client_access_credentials/migration.sql`
- `portal-sama-api/src/modules/rbac/default-rbac.ts`
- `portal-sama-api/src/modules/rbac/default-rbac.spec.ts`

### Comandos da Fase 2

```powershell
npm.cmd run prisma:format
npm.cmd run prisma:validate
npm.cmd run prisma:generate
$env:DATABASE_URL='mysql://portal_user:portal_password@localhost:3306/portal_sama'; npm.cmd run prisma:migrate:deploy
$env:DATABASE_URL='mysql://portal_user:portal_password@localhost:3306/portal_sama'; npm.cmd run prisma:migrate:status
$env:DATABASE_URL='mysql://portal_user:portal_password@localhost:3306/portal_sama'; npm.cmd run prisma:seed
npm.cmd test -- default-rbac.spec.ts
npm.cmd test -- env.schema.spec.ts default-rbac.spec.ts contracts.service.spec.ts
npm.cmd run lint
npm.cmd run build
```

### Resultado da Fase 2

- Prisma validate: passou.
- Prisma generate: passou.
- Migration em banco limpo: 32 migrations aplicadas com sucesso.
- Migration status: banco local up to date.
- Seed RBAC: passou; 8 permissoes novas confirmadas.
- Teste fake de contrato preexistente: dados preservados com defaults `INTERNAL` e `BR`.
- Teste fake de credencial: `has_secret_cipher=1`, `audit_events=1`; registro fake removido.
- API lint: passou.
- API build: passou.
- API testes focados: passaram, 3 suites e 19 testes.

### Observacoes de seguranca da Fase 2

- Nenhum segredo real foi lido ou registrado.
- Campos de acesso do cliente nao armazenam segredo em claro; a modelagem aceita somente `secretCipher` ou `secretRef`.
- Permissoes de revelacao sensivel foram criadas, mas nao foram concedidas a perfis operacionais comuns como `LEGALIZATION`.
