# [PENDENTE] Ainda Falta Para Deploy Em Producao - Portal Sama

Ultima atualizacao: 2026-06-02
Responsavel/IA: Codex
Status do documento: obrigatorio e vivo

## Objetivo

Este documento e o painel oficial do que ainda falta para colocar a nova aplicacao do Portal Sama em producao, sem depender da aplicacao antiga.

Ele deve responder sempre:

- qual e o estado real da nova stack;
- quanto falta, em percentual aproximado, para homologacao e producao;
- quais bloqueios ainda existem;
- o que precisa ser feito no EasyPanel;
- como separar `portal-sama-api`, `portal-sama-web` e `portal-sama-docs` em repositorios Bitbucket;
- quais validacoes precisam passar antes do corte para producao.

## Regra obrigatoria de atualizacao

Este arquivo deve ser lido e atualizado em todo ciclo que envolva:

- deploy, EasyPanel, Docker, Nginx, HTTPS, dominio ou variaveis de ambiente;
- Bitbucket, separacao de repositorios ou mudanca de workspace;
- MySQL, Prisma, migrations, seed, bootstrap, backfill ou banco `banco-sama`;
- usuarios reais, permissoes reais, roles ou RBAC;
- storage privado, upload/download, ClamAV ou documentos sensiveis;
- QA visual, Playwright, smoke publico ou homologacao real;
- qualquer resposta sobre "quanto falta", "pronto para producao" ou "dependencia do legado".

Antes de atualizar este documento, a IA/desenvolvedor deve:

1. Ler `INSTRUÇÕEDS_AI.MD`.
2. Ler `ORDEM_IMPLEMENTACAO_DOCUMENTACOES.md`.
3. Ler `STATUS_IMPLEMENTACAO.md`.
4. Ler `PENDENCIAS_TECNICAS.md`.
5. Ler `EASYPANEL_DEPLOY.md`.
6. Ler `RELATORIO_TESTES.md`.
7. Ler `CHANGELOG_TECNICO.md`.
8. Ler `integracao_acessorias_paths_importacao_dev.md` quando a tarefa envolver Acessorias, `.env`, importacao pela area DEV, Fase 2 ou Fase 3.
9. Ler `integracao_acessorias_entregas_vencimentos.md` quando a tarefa envolver entregas, vencimentos, planilhas departamentais, notificacoes ou integracoes externas.
10. Ler `ux-ui-docs/README.md` e `ux-ui-docs/09-checklist-aceite.md` quando a tarefa envolver UX/UI, navegacao, configuracoes, solicitacoes, departamentos, permissoes ou responsabilidades de clientes.
11. Ler os arquivos de codigo afetados em `portal-sama-api` e/ou `portal-sama-web`.
12. Verificar o estado real do Git e dos arquivos existentes.
13. Atualizar este documento com data/hora, percentual e pendencias reais.

Se uma porcentagem for informada, ela deve ser marcada como estimativa operacional, nao como metrica matematica exata.

## Percentual atual

Estimativa operacional em 2026-05-28 10:06 -03:00:

| Marco | Percentual | Leitura |
|---|---:|---|
| Pronto para iniciar/continuar homologacao da nova stack | 99% | Deploy publico, smoke real, banco, migrations, RBAC, usuarios privilegiados, storage e ClamAV strict ja foram validados no container real via readiness; tambem passaram smoke/auth real, Playwright real e matriz minima DEV/bootstrap contra HTTPS. O restore drill agora possui comando e plano operacional, e a Home local ja consome resumo read-only do Acessorias via API v2. Ainda falta executar restore com artefatos reais e validar a Home com o contrato real do Acessorias no EasyPanel. |
| Pronto para producao sem depender do legado | 88% | A stack nova esta operacional no EasyPanel, o readiness real passou com warning apenas de backup/rollback, o fluxo publico de documentos de onboarding esta alinhado aos contratos documentados e ha evidencia real de login/refresh/logout, Playwright real e RBAC minimo DEV/bootstrap. O Integra-AI local tem leiaute Dominio unico e validavel; responsabilidades normalizadas incluem transferencia auditada; a Home por perfil deixou de depender de KPIs tecnicos genericos e passou a usar Acessorias por rota backend autenticada. Permanecem pendentes backfills/conferencias reais, matriz por multiplos perfis, restore drill real, QA final, validacao real de uploads/exportacoes, validacao do contrato real do Acessorias, UI/fluxos reais de carteiras e desligamento seguro do legado. |
| Falta para producao | 12% | Principalmente execucao/conferencia real de backfills, evidencias funcionais por todos os perfis, backup externo verificado com restore testado, upload/download/exportacao real, QA final, homologacao do TXT Dominio real, aceite UX/UI, completar UI/migracao de responsabilidades de clientes, validar a Home com dados reais do Acessorias e plano de corte/desligamento do legado. |

## Estado atual resumido

| Area | Estado | Observacao |
|---|---|---|
| Backend `portal-sama-api` | Deployado e operacional | NestJS/API v2 esta rodando no EasyPanel; logs mostram `Nest application successfully started`; smoke publico confirmou `/api-v2/health` com `database=up` e `storage=up`; aliases publicos de documentos de onboarding foram implementados. Falta validar fluxos de negocio e permissoes com dados reais. |
| Frontend `portal-sama-web` | Deployado e operacional | React/Vite esta publicado em `https://portal.samacontabil.com.br/`; smoke publico validou frontend/proxy/CORS/CSRF; `smoke:auth`, matriz minima DEV/bootstrap e `test:e2e:real` passaram em HTTPS com usuario real de bootstrap. Ha correcoes locais pendentes de deploy para download TXT autenticado do Integra-AI, alinhamento da tela com a sidebar e remocao do seletor de leiaute Dominio conflitante. Falta repetir por perfis operacionais e validar fluxos reais de negocio. |
| Banco MySQL/Prisma | Operacional em homologacao | EasyPanel possui servico `portal-sama-database`; log mostra MySQL Community Server 9.7.0 pronto em `:3306`; readiness real confirmou database `SELECT 1` e 19 migrations Prisma aplicadas. Falta rodar relatorio de backfill e conferir dados por modulo. |
| Primeiro admin/usuarios API v2 | Parcial operacional | Usuario(s) inicial(is) ja foram criados pelo operador e a aplicacao esta conectada ao banco real; falta registrar matriz de usuarios/perfis reais e validar permissoes por papel. |
| EasyPanel | Deployado | Projeto `portal-sama` contem `portal-sama-api`, `portal-sama-database` e `portal-sama-web` ativos. Readiness real passou com warning apenas de backup/rollback; falta restore drill e hardening operacional final. |
| Workspaces locais | Separado e preparado | Existem pastas irmas `portal-sama-docs`, `portal-sama-api` e `portal-sama-web` no Desktop; API/Web agora possuem README minimo com regra de leitura do repo de docs. |
| Bitbucket/Git | Remotos existentes | Repositorios Bitbucket existem para API, Web e documentacao (`portal-sama-api`, `portal-sama-web`, `portal-sama-doc` conforme evidencia visual). Falta apenas garantir push das atualizacoes documentais locais desta rodada. |
| Builds locais separados | Validado | Passaram lint/build da API, `prisma:validate` e lint/build do Web nos repos separados. |
| Storage privado | Parcial validado | Health publico retornou `storage=up` e readiness real confirmou leitura/escrita em `/var/private/portal-sama`. O Web ja aponta para o upload publico documentado de onboarding, mas ainda falta validar persistencia do volume, permissoes, backup externo e fluxos reais de upload/download. |
| ClamAV | Validado em homologacao real | Readiness real detectou EICAR com `/usr/bin/clamscan` em modo strict. Manter `npm run ops:clamav:update` como rotina operacional quando assinaturas precisarem ser atualizadas. |
| Usuarios/permissoes reais | Parcial validado | Usuario bootstrap/DEV autenticou em HTTPS e passou matriz minima 401/200 para `/auth/me`, `/users`, `/roles` e `/permissions`. Falta matriz oficial com CLIENT, MANAGER, departamentos e perfis operacionais, incluindo 403 esperados. |
| Backfills | Parcial instrumentalizado | `portal-sama-api` agora possui `npm run ops:backfill:report`, read-only, para listar tabelas atuais/legadas, contagens, usuarios sem role, roles sem permissoes e vinculos quebrados. Falta rodar no MySQL real e executar/conferir os backfills. |
| Backups/rollback | Parcial instrumentalizado | `portal-sama-api` possui `npm run ops:backup:create`, `npm run ops:backup:verify` e agora `npm run ops:restore:drill`, com dry-run, bloqueio de alvo igual ao banco/storage da aplicacao e confirmacao explicita para aplicar restore. Falta rodar no EasyPanel, copiar artefatos para fora do container e provar restore drill real. |
| Playwright real | Parcial validado | `npm.cmd run test:e2e:real` passou com usuario real de bootstrap/DEV contra `https://portal.samacontabil.com.br`, validando login, Home, storage do navegador, cookie HttpOnly/SameSite/Secure e logout. Falta repetir por perfis criticos e ampliar fluxos. |
| QA visual | Parcial | Aplicacao publica esta acessivel; o Integra-AI ganhou smoke local de alinhamento com a sidebar em viewport larga. Ainda necessario aprovar desktop/mobile no EasyPanel/navegador real com usuarios e perfis reais. |
| Dependencia do legado | Parcial | Integra-AI ja possui mutacoes API v2, importacao OFX opt-in atras de feature flag e leiaute Dominio oficial unico no fluxo local. Ainda exige validacao funcional real/backfills, homologacao manual do TXT no Dominio e demais validacoes reais antes de desligar PHP. |
| Acessorias, entregas e vencimentos | Parcial local; validacao real pendente | A Home local consulta o Acessorias via backend sem expor token no Web; a area `/dev` agora possui botoes para testar, pre-visualizar/importar clientes, responsaveis e entregas, gerar notificacoes manuais e consultar `Status scheduler`, tudo com paginacao ate lista vazia, rate limit e timeout longo no Web; planilhas departamentais aplicam baixas, revisam divergencias e exibem vencimentos oficiais Acessorias no carrossel/workspace; `/departamentos/vencimentos` consolida vencimentos de calendario e Acessorias em tela dedicada. Scheduler seguro existe como opt-in e desabilitado por padrao. Ainda falta executar esses fluxos no EasyPanel com dados reais, registrar evidencia sanitizada, conferir notificacoes/auditoria reais e habilitar scheduler em rollout controlado. |
| UX/UI e governanca operacional | Parcial implementado | `ux-ui-docs/` define navegacao, Home por perfil, wireframes, configuracoes, departamentos, solicitacoes, roles/permissoes e responsabilidade cliente x usuario. As duas primeiras fatias de departamentos controlados foram implementadas localmente para usuarios DEV/Admin e colaboradores; a fundacao backend de `client_department_assignments` cobre transferencia normalizada auditada; `/clientes/:id` agora exibe `Equipe e responsaveis`, cria atribuicao inicial, edita, encerra e transfere responsabilidade ativa pela tabela normalizada; o dashboard de carteira do gestor prioriza responsabilidades normalizadas em leitura; a Home por perfil consome resumo do Acessorias. Falta aplicar migrations/seeds no MySQL real, validar responsabilidades reais no EasyPanel, executar `ux-ui-docs/09-checklist-aceite.md` e fechar escrita normalizada de transferencias em lote, solicitacoes globais e permissoes guiadas. |

## Atualizacao operacional de 2026-06-02 dashboard de carteira normalizado

Foi implementada localmente a leitura normalizada da carteira do gestor no dashboard de transferencias.

- `GET /api-v2/transfers/dashboard` passou a carregar responsabilidades `ACTIVE` de `client_department_assignments`;
- `/manager/colaboradores`, que consome esse dashboard, passa a refletir a tabela normalizada quando ha responsabilidade ativa para cliente/departamento;
- o backend resolve `responsibleUserId` para o `colaboradorId` legado usado pela UI;
- o fallback por `clients.metadata` continua quando nao existe responsabilidade normalizada ativa;
- a transferencia operacional em lote ainda nao escreve em `client_department_assignments` nesta fatia.

Validacao local:

- `npm.cmd test -- transfers.service.spec.ts --runInBand` passou na API com 5 testes;
- `npm.cmd run build` passou na API;
- `npm.cmd run lint` passou na API.

Pendencia real remanescente: publicar a API atualizada, validar no EasyPanel com MySQL real/backfill, gestor real por departamento e carteira real; depois migrar a escrita de transferencias em lote para `client_department_assignments`.

## Atualizacao operacional de 2026-06-02 edicao e encerramento de responsabilidades no painel

Foi implementada localmente a edicao e o encerramento de responsabilidades normalizadas no painel do cliente.

- `/clientes/:id` ganhou acoes `Editar` e `Encerrar` por responsabilidade;
- `Editar` chama `PATCH /api-v2/client-assignments/:id` por `updateClientAssignment()`;
- a edicao permite ajustar departamento, responsavel operacional, gestor, tipo, status e periodo;
- `Encerrar` chama `POST /api-v2/client-assignments/:id/end` por `endClientAssignment()`;
- o encerramento coleta data e motivo;
- a UI exige `client_assignments.update` para edicao e `client_assignments.end` para encerramento;
- a edicao depende de `departments.read` e `collaborators.read` para carregar listas auxiliares;
- `types/clients.ts` passou a tipar `ClientAssignmentUpdateInput` e `ClientAssignmentEndInput`.

Validacao local:

- `npx.cmd tsc --noEmit --pretty false` passou no Web;
- `npm.cmd run build` passou no Web;
- `npm.cmd run lint` passou no Web;
- `npm.cmd run test:e2e` passou no Web com 12 testes e 1 teste real opt-in pulado.

Pendencia real remanescente: publicar o Web atualizado, validar no EasyPanel com responsabilidades reais e usuario autorizado, conferir auditoria persistida para `client_assignments.update` e `client_assignments.end`, validar bloqueios para perfis sem essas permissoes e seguir com backfill e carteiras/filtros normalizados.

## Atualizacao operacional de 2026-06-02 painel do cliente e responsabilidades

Foi implementada localmente a primeira conexao visual da Fase 4.

- `/clientes/:id` ganhou a secao `Equipe e responsaveis`;
- a secao consome `GET /api-v2/clients/:clientId/assignments`;
- a consulta respeita `client_assignments.read` e nao chama o endpoint sem permissao;
- o painel exibe departamento, responsavel, gestor, tipo de vinculo, status e datas;
- a acao `Nova responsabilidade` cria `POST /api-v2/clients/:clientId/assignments` com responsavel operacional, departamento, tipo, inicio e gestor opcional;
- a criacao exige `client_assignments.create` e listas auxiliares com `departments.read`/`collaborators.read`;
- `clients.service.ts` ganhou `listClientAssignments()` e `createClientAssignment()`;
- `types/clients.ts` passou a tipar leitura e criacao de responsabilidades normalizadas.

Validacao local:

- `npx.cmd tsc --noEmit --pretty false` passou no Web;
- `npm.cmd run build` passou no Web;
- `npm.cmd run lint` passou no Web.

Pendencia real remanescente: validar no EasyPanel com responsabilidades reais, conferir perfis com/sem `client_assignments.read` e `client_assignments.create`, aplicar migrations/backfill no MySQL real e migrar carteiras/filtros normalizados.

## Atualizacao operacional de 2026-06-02 transferencia de responsabilidades no painel

Foi implementada localmente a transferencia auditada de responsabilidades normalizadas no painel do cliente.

- `/clientes/:id` ganhou acao `Transferir` para responsabilidades `ACTIVE`;
- a acao chama `POST /api-v2/client-assignments/transfer` por `transferClientAssignments()`;
- a UI exige `client_assignments.transfer` e carrega responsaveis com `collaborators.read`;
- o formulario coleta novo responsavel operacional, gestor opcional, data efetiva e motivo;
- a UI bloqueia o mesmo responsavel atual como destino;
- `types/clients.ts` passou a tipar `ClientAssignmentTransferInput` e `ClientAssignmentsTransferResponse`;
- a seguranca real segue no backend com JWT, CSRF, RBAC, escopo e auditoria `client_assignments.transfer`.

Validacao local:

- `npx.cmd tsc --noEmit --pretty false` passou no Web;
- `npm.cmd run build` passou no Web;
- `npm.cmd run lint` passou no Web;
- `npm.cmd run test:e2e` passou no Web com 12 testes e 1 teste real opt-in pulado;
- `git diff --check` passou no Web.

Pendencia real remanescente: publicar o Web atualizado, validar no EasyPanel com responsabilidades reais e usuario autorizado, conferir auditoria persistida, validar bloqueios para perfis sem `client_assignments.transfer` e seguir com backfill e carteiras/filtros normalizados.

## Atualizacao operacional de 2026-06-02 scheduler Acessorias opt-in

Foi implementado localmente o scheduler seguro do Acessorias, desabilitado por padrao.

- criado `AcessoriasSchedulerService`;
- adicionadas envs `ACESSORIAS_SCHEDULER_ENABLED`, `ACESSORIAS_SCHEDULER_NOTIFICATIONS_ENABLED`, `ACESSORIAS_SCHEDULER_INITIAL_DELAY_SEC`, `ACESSORIAS_SCHEDULER_INTERVAL_MINUTES`, `ACESSORIAS_SCHEDULER_MAX_RUN_MINUTES` e `ACESSORIAS_SCHEDULER_NOTIFICATION_DAYS_AHEAD`;
- scheduler sincroniza entregas com `trigger=scheduler`;
- notificacoes automaticas ficam separadas e desligadas por padrao;
- execucao evita sobreposicao local e pula quando ha sync `RUNNING` recente;
- criado `GET /api-v2/integrations/acessorias/scheduler/status`;
- `/dev` ganhou o botao `Status scheduler`;
- cada execucao audita `integrations.acessorias.scheduler.run`.

Validacao local:

- `npm.cmd test -- acessorias-scheduler.service.spec.ts acessorias-deliveries.service.spec.ts acessorias-fiscal-application.service.spec.ts --runInBand` passou com 3 suites e 12 testes;
- `npm.cmd test -- acessorias-deliveries.service.spec.ts acessorias-registrations.service.spec.ts acessorias-home.service.spec.ts acessorias-scheduler.service.spec.ts departments.service.spec.ts --runInBand` passou com 5 suites e 23 testes;
- `npx.cmd tsc --noEmit --pretty false` passou no Web;
- `npm.cmd run build` e `npm.cmd run lint` passaram na API;
- `npm.cmd run build` e `npm.cmd run lint` passaram no Web.

Pendencia real remanescente: publicar API/Web no EasyPanel, validar status do scheduler com usuario autorizado, habilitar primeiro sem notificacoes automaticas, observar duracao/totais/auditoria e so depois habilitar notificacoes automaticas. Esta fatia nao esta homologada em producao.

## Atualizacao operacional de 2026-06-02 notificacoes Acessorias manuais

Foi implementada localmente a geracao manual de notificacoes operacionais do Acessorias.

- criado endpoint `POST /api-v2/integrations/acessorias/deliveries/notifications/generate`;
- a rotina cobre vencimento proximo, atraso, baixa confirmada e divergencia aberta;
- as notificacoes sao deduplicadas por `dedupeKey`;
- o alvo usa responsavel da entrega quando existir ou departamento operacional;
- `/dev` ganhou o botao protegido `Gerar notificacoes`;
- a acao exige JWT, CSRF, `integrations.acessorias.deliveries.manage` e `notifications.create`;
- a execucao registra auditoria.

Validacao local:

- `npm.cmd test -- acessorias-fiscal-application.service.spec.ts --runInBand` passou com 1 suite e 5 testes;
- `npm.cmd test -- acessorias-deliveries.service.spec.ts acessorias-registrations.service.spec.ts acessorias-home.service.spec.ts departments.service.spec.ts --runInBand` passou com 4 suites e 20 testes;
- `npx.cmd tsc --noEmit --pretty false` passou no Web;
- `npm.cmd run build` e `npm.cmd run lint` passaram na API;
- `npm.cmd run build` e `npm.cmd run lint` passaram no Web.

Pendencia real remanescente: publicar API/Web no EasyPanel, executar o botao com usuario autorizado, conferir notificacoes e auditoria no banco real, validar destinatarios reais e habilitar scheduler automatico apenas em rollout controlado. Esta fatia nao esta homologada em producao.

## Atualizacao operacional de 2026-06-01 paginacao Acessorias e Central de Vencimentos

Foi implementada localmente a correcao dos fluxos Acessorias que podiam retornar poucos registros ou falhar por timeout no Web.

- Home, preview/sync de entregas e preview/sync de cadastros paginam `ListAll` ate resposta vazia, com `ACESSORIAS_MAX_PAGES=1000` por padrao;
- as chamadas externas aguardam entre paginas conforme `ACESSORIAS_RATE_LIMIT_PER_MINUTE=95`, mantendo margem abaixo do limite de 100 requisicoes/minuto;
- respostas `429` do Acessorias entram em retry controlado, respeitando `Retry-After` quando informado;
- sync de entregas salva `incremental_since` no run e usa o ultimo run `SUCCESS` como marco para sincronizacoes seguintes;
- o Web passou a dar ate 10 minutos para chamadas Acessorias, e `/dev` mostra erro por acao;
- `Importar tudo` usa `entity=all` para cadastros e reaproveita empresas carregadas quando responsaveis forem extraidos dos departamentos;
- criada rota React `/departamentos/vencimentos`, consolidando vencimentos de calendario e Acessorias com filtros por departamento, mes, origem, status e busca.

Validacao local:

- `npm.cmd test -- acessorias-deliveries.service.spec.ts acessorias-registrations.service.spec.ts acessorias-home.service.spec.ts departments.service.spec.ts --runInBand` passou com 4 suites e 20 testes;
- `npx.cmd tsc --noEmit --pretty false` passou no Web;
- `npm.cmd run build` e `npm.cmd run lint` passaram na API;
- `npm.cmd run build` e `npm.cmd run lint` passaram no Web.

Pendencia real remanescente: publicar API/Web no EasyPanel, executar os botoes com token real, confirmar se os totais batem com a base real do Acessorias, observar duracao real da importacao e validar visualmente `/departamentos/vencimentos`. A Central esta implementada localmente, mas nao homologada.

## Atualizacao operacional de 2026-06-01 09:30 -03:00

Foi implementado no `portal-sama-api` o orquestrador local/container da Fase 1 da ordem operacional:

- criado script `scripts/run-operational-phase1.js`;
- criado comando `npm run ops:phase1`;
- a sequencia executa `prisma:migrate:deploy`, `prisma:seed`, `ops:readiness`, backup operacional, verificacao do backup e restore drill;
- o comando aceita evidencia JSON em `.ai-tests/phase1-operational`;
- restore real continua protegido por alvo isolado e confirmacao explicita `RESTORE_DRILL_TARGET_IS_ISOLATED`;
- tambem pode rodar `ops:backfill:report` com `--include-backfill-report`;
- o modo de teste local com todas as etapas puladas foi validado sem tocar banco, storage ou restore real.

Validacao local:

- `node --check scripts/run-operational-phase1.js` passou;
- `npm.cmd run ops:phase1 -- --help` passou;
- `npm.cmd run ops:phase1 -- --skip-migrate --skip-seed --skip-readiness --skip-backup --skip-restore --no-evidence --json` passou.

Pendencia real remanescente: executar `npm run ops:phase1` no container real da API no EasyPanel com `DATABASE_URL`, `STORAGE_PRIVATE_PATH`, alvo de banco isolado, alvo de storage isolado e backup fora do volume principal; depois registrar evidencia sanitizada e rodar `portal-sama-web` `npm run homologation:real` com matriz real de perfis.

## Atualizacao operacional de 2026-05-28 10:45 -03:00

Foi implementada a fundacao backend para importacao controlada de empresas e colaboradores do Acessorias.

- a API recebeu variaveis validadas `ACESSORIAS_CLIENTS_PATH` e `ACESSORIAS_COLLABORATORS_PATH`;
- criado endpoint protegido `GET /api-v2/integrations/acessorias/registrations/preview`;
- criado endpoint protegido `POST /api-v2/integrations/acessorias/registrations/sync`;
- o preview mostra acao prevista por registro: criar, vincular metadata, sobrescrever manualmente ou ignorar;
- empresas sao conciliadas por CNPJ e ID externo do Acessorias;
- colaboradores sao conciliados por usuario, e-mail e ID externo do Acessorias;
- registros importados recebem `metadata.acessorias` com `externalId`, `importedAt`, `editableInPortal=true` e payload bruto;
- por padrao, registros existentes mantem os dados locais editaveis e recebem apenas atualizacao de metadata;
- sobrescrita de dados locais exige `overwriteLocal=true` em sincronizacao manual com permissao `integrations.acessorias.registrations.sync`;
- a sincronizacao registra auditoria centralizada e nao expoe token do Acessorias ao frontend.

Validacao local:

- `npm.cmd run build` passou na API.

Pendencia real remanescente: validar no EasyPanel o contrato real dos endpoints de empresas/colaboradores do Acessorias, configurar `ACESSORIAS_CLIENTS_PATH`/`ACESSORIAS_COLLABORATORS_PATH`, rodar `prisma:seed` para disponibilizar as novas permissoes e criar UI seguindo `ux-ui-docs/`.

## Atualizacao operacional de 2026-05-28 11:20 -03:00

Foi implementada a primeira base backend persistente para entregas do Acessorias.

- criado modelo Prisma e migration `20260528110000_add_acessorias_deliveries`;
- criada tabela `acessorias_deliveries` para armazenar copia local das entregas;
- criada tabela `acessorias_delivery_sync_runs` para registrar execucoes de sincronizacao;
- a API recebeu variavel validada `ACESSORIAS_DELIVERIES_PATH`;
- criado endpoint protegido `POST /api-v2/integrations/acessorias/deliveries/sync`;
- criado endpoint protegido `GET /api-v2/integrations/acessorias/deliveries`;
- a sincronizacao manual busca entregas no Acessorias, normaliza campos comuns, salva/atualiza por `external_id` e registra auditoria;
- a listagem local aceita filtros por departamento, status, competencia, responsavel e documento do cliente;
- RBAC ganhou `integrations.acessorias.deliveries.read` e `integrations.acessorias.deliveries.sync`.

Validacao local:

- `npm.cmd run prisma:generate` passou;
- `npm.cmd run prisma:validate` passou;
- `npm.cmd test -- integrations/acessorias/acessorias-deliveries.service.spec.ts --runInBand` passou;
- `npm.cmd test -- integrations/acessorias/acessorias-registrations.service.spec.ts rbac/default-rbac.spec.ts --runInBand` passou;
- `npm.cmd run build` passou;
- `npm.cmd run lint` passou.

Pendencia real remanescente: aplicar migration no MySQL real, rodar seed RBAC, configurar `ACESSORIAS_DELIVERIES_PATH`, validar contrato real de entregas do Acessorias no EasyPanel e depois implementar mapeamento para planilha Fiscal, divergencias, Central de Vencimentos, scheduler e notificacoes.

## Atualizacao operacional de 2026-06-01 complementar

Foi ampliada a aplicacao operacional do Acessorias para os departamentos usados na planilha modelo.

- `/departamentos/modelo` agora alterna Fiscal, Contabil, Pessoal, Financeiro e Legalizacao;
- adicionadas rotas genericas `/api-v2/departments/workspace*`;
- criado endpoint protegido `POST /api-v2/integrations/acessorias/deliveries/apply-to-workspace`, mantendo `apply-to-fiscal` compativel;
- a Home do Acessorias trata `204 No Content` como retorno valido sem entregas e exibe diagnostico sanitizado em falhas.

Validacao parcial:

- testes focados de Acessorias/Departamentos passaram com 4 suites e 16 testes;
- `npx.cmd tsc --noEmit --pretty false` passou no Web;
- build/lint da API e do Web passaram.

Pendencia real remanescente: publicar API/Web, validar contrato real no EasyPanel, conferir mapeamentos reais por departamento e executar matriz de usuarios reais.

## Atualizacao operacional de 2026-06-01 11:30 -03:00

Foi implementada a primeira aplicacao segura de baixas do Acessorias na planilha Fiscal.

- criado modelo Prisma e migration `20260601113000_add_acessorias_fiscal_application`;
- criadas tabelas `acessorias_fiscal_apply_runs` e `acessorias_fiscal_divergences`;
- criado endpoint protegido `POST /api-v2/integrations/acessorias/deliveries/apply-to-fiscal`;
- o endpoint exige `integrations.acessorias.deliveries.manage` e `departments.workspace.write`;
- a aplicacao marca celula como `ACESSORIAS` apenas quando entrega local esta `DELIVERED`, cliente foi identificado, competencia esta aberta, coluna Fiscal e valida e mapeamento esta `CONFIRMED` com confianca minima;
- mapeamento ausente, baixa confianca, cliente nao encontrado, status conflitante, Acessorias pendente contra Portal concluido ou mes fechado geram divergencia aberta sem alterar a planilha;
- `/departamentos/modelo` passou a exibir contador de baixas Acessorias, divergencias abertas, marcador visual na celula e botao protegido `Aplicar Acessorias`.

Validacao local:

- `npm.cmd run prisma:generate` passou na API;
- `npm.cmd run prisma:format` passou na API;
- `npm.cmd run prisma:validate` passou na API;
- testes focados de Acessorias/Departamentos/RBAC passaram com 4 suites e 12 testes;
- `npm.cmd run build` passou na API;
- `npm.cmd run lint` passou na API;
- `npm.cmd run build` passou no Web;
- `npm.cmd run lint` passou no Web;
- `git diff --check` passou em API/Web.

Pendencia real remanescente: aplicar migrations no MySQL real, publicar API/Web, validar contrato real do Acessorias no EasyPanel, executar dry-run/execucao real do endpoint com usuario autorizado e conferir auditoria, celulas `ACESSORIAS` e divergencias abertas com dados reais.

## Atualizacao operacional de 2026-06-01 revisao manual de divergencias Acessorias

Foi implementada a revisao manual local das divergencias criadas pela aplicacao do Acessorias nas planilhas departamentais.

- criado endpoint protegido `PATCH /api-v2/integrations/acessorias/deliveries/divergences/:id`;
- o endpoint exige `integrations.acessorias.deliveries.manage` e `departments.workspace.write`, alem de CSRF;
- a revisao aceita status `RESOLVED`, `IGNORED` ou `OPEN`;
- a API grava `resolvedAt`, `resolvedById`, `metadata.manualReview`, `metadata.reviewHistory` e auditoria `integrations.acessorias.divergence.review`;
- `/departamentos/modelo` exibe botoes protegidos `Resolver` e `Ignorar` nos cards de divergencias abertas e recarrega o workspace apos a revisao.

Validacao local:

- `npm.cmd test -- acessorias-fiscal-application.service.spec.ts --runInBand` passou na API com 1 suite e 4 testes;
- `npm.cmd run build` passou na API;
- `npm.cmd run lint` passou na API;
- `npm.cmd run build` passou no Web;
- `npm.cmd run lint` passou no Web.

Pendencia real remanescente: publicar API/Web, validar no EasyPanel com usuario autorizado, CSRF real, dados reais e conferencia de auditoria/historico no banco. Central de Vencimentos, scheduler e notificacoes continuam pendentes.

## Atualizacao operacional de 2026-06-01 vencimentos Acessorias no workspace

Foi implementada a conexao local entre entregas/vencimentos do Acessorias e o workspace departamental.

- `GET /api-v2/departments/workspace` agora inclui entregas sincronizadas em `acessorias_deliveries` com `dueAt` dentro do mes selecionado e status pendente/atrasado;
- os vencimentos Acessorias entram no carrossel operacional da planilha;
- quando existe mapeamento `CONFIRMED` para uma coluna operacional valida, o vencimento tambem aparece na celula e participa do bloqueio por vencimento;
- sem mapeamento confirmado, o vencimento aparece apenas no carrossel, sem bloquear celula;
- `/departamentos/modelo` identifica a origem `Acessorias` no item do carrossel.

Validacao local:

- `npm.cmd test -- departments.service.spec.ts --runInBand` passou na API com 1 suite e 5 testes;
- `npm.cmd run build` passou na API;
- `npm.cmd run lint` passou na API;
- `npm.cmd run build` passou no Web;
- `npm.cmd run lint` passou no Web.

Pendencia real remanescente: validar no EasyPanel com entregas reais, mapeamentos reais, usuarios por departamento e conferencia visual do carrossel/celulas. Central de Vencimentos dedicada, scheduler e notificacoes continuam pendentes.

## Atualizacao operacional de 2026-06-01 10:00 -03:00

Foram implementadas duas pendencias desta rodada: correcao do auto-refresh de sessao no Web e base backend de mapeamento Acessorias -> planilha Fiscal.

Sessao/seguranca:

- `portal-sama-web/src/services/api.ts` passou a renovar access token antes de chamadas protegidas quando o token nao esta fresco;
- `api.ts` passou a tratar `401` com retry unico da requisicao original;
- o refresh usa promessa compartilhada para evitar corrida com rotacao de refresh token;
- endpoints `/auth/csrf`, `/auth/login`, `/auth/refresh`, `/auth/logout` e `/auth/forgot-password` ficam fora do auto-refresh;
- `withCredentials` e CSRF permanecem ativos.

Acessorias:

- criado modelo Prisma e migration `20260601100000_add_acessorias_delivery_column_mappings`;
- criada tabela `acessorias_delivery_column_mappings`;
- criado endpoint protegido `GET /api-v2/integrations/acessorias/delivery-column-mappings`;
- criado endpoint protegido `GET /api-v2/integrations/acessorias/delivery-column-mappings/suggestions`;
- criado endpoint protegido `POST /api-v2/integrations/acessorias/delivery-column-mappings`;
- criado endpoint protegido `PATCH /api-v2/integrations/acessorias/delivery-column-mappings/:id`;
- RBAC ganhou `integrations.acessorias.deliveries.manage`;
- criacao/alteracao de mapeamentos registra auditoria;
- sugestoes usam entregas locais sincronizadas, mas nao alteram celulas automaticamente.

Validacao local:

- `npm.cmd run build` passou no Web;
- `npm.cmd run lint` passou no Web;
- `npm.cmd run prisma:generate` passou na API;
- `npm.cmd run prisma:format` passou na API;
- `npm.cmd run prisma:validate` passou na API;
- `npm.cmd run build` passou na API;
- `npm.cmd run lint` passou na API;
- testes focados de Acessorias/RBAC passaram com 5 suites e 16 testes.

Pendencia real remanescente: aplicar migrations no MySQL real, rodar seed RBAC, validar no EasyPanel o auto-refresh aguardando vencimento real do access token, validar contrato real do Acessorias e publicar/validar a aplicacao de mapeamentos confirmados na planilha Fiscal com dados reais.

## Atualizacao operacional de 2026-05-28 10:06 -03:00

Foi implementada a fatia local da Home por perfil usando dados do Acessorias pela API v2.

- a API recebeu variaveis validadas `ACESSORIAS_BASE_URL`, `ACESSORIAS_TOKEN`, `ACESSORIAS_HOME_PATH`, `ACESSORIAS_AUTH_HEADER`, `ACESSORIAS_AUTH_SCHEME` e `ACESSORIAS_TIMEOUT_SEC`;
- criado modulo backend `AcessoriasHomeModule` com `GET /api-v2/integrations/acessorias/home-summary`;
- a consulta e read-only, autenticada no Portal Sama e usa o token do Acessorias somente no backend;
- o servico normaliza entregas/obrigacoes retornadas em formatos comuns (`data`, `items`, `entregas`, `deliveries`, `obrigacoes`, `results`) e calcula pendentes, proximas, atrasadas, baixadas e carteira por responsavel;
- a Home React deixou de exibir KPIs tecnicos genericos para colaborador/gestor e passou a mostrar pendencias, vencimentos, atrasos, entregas baixadas e diagnostico da integracao conforme perfil;
- quando a integracao nao estiver configurada ou estiver indisponivel, a Home retorna aviso controlado sem expor segredo nem quebrar a tela.

Validacao local:

- `npm.cmd test -- integrations/acessorias/acessorias-home.service.spec.ts --runInBand` passou;
- `npm.cmd run lint` passou na API;
- `npm.cmd run build` passou na API;
- `npm.cmd run prisma:validate` passou na API com `DATABASE_URL` dummy;
- `npm.cmd run lint` passou no Web;
- `npm.cmd run build` passou no Web;
- `npm.cmd run test:e2e -- -g "home"` passou no Web com 3 testes e 1 opt-in real skipped.

Pendencia real remanescente: publicar API/Web no EasyPanel, configurar/confirmar as variaveis do Acessorias no ambiente da API, validar o contrato real retornado pela API do Acessorias, repetir a Home com usuarios reais por perfil e decidir se a rota read-only permanece no corte base ou se entra como pre-MVP da trilha Acessorias.

## Atualizacao operacional de 2026-05-27 16:44 -03:00

Foi implementada a segunda fatia backend de responsabilidade normalizada de clientes: transferencia auditada entre responsaveis.

- criado DTO `TransferClientAssignmentDto`;
- criado endpoint protegido `POST /api-v2/client-assignments/transfer`;
- a permissao exigida e `client_assignments.transfer`;
- a transferencia aceita uma ou mais responsabilidades ativas;
- a responsabilidade atual e encerrada como `TRANSFERRED`, preservando historico e metadata da transferencia;
- a nova responsabilidade e criada como `ACTIVE`, mantendo cliente, departamento e tipo, mas apontando para o responsavel de destino;
- gestor pode ser preservado, trocado ou removido conforme payload;
- a API valida escopo, status ativo, departamento controlado, responsavel interno ativo e ausencia de duplicidade `PRIMARY` ativa;
- cada nova responsabilidade transferida registra auditoria `client_assignments.transfer`.

Validacao local:

- testes focados `client-assignments.service`, `default-rbac` e `department-catalog.service` passaram;
- `npm.cmd run prisma:validate` passou;
- `npm.cmd run lint` passou;
- `npm.cmd run build` passou.

Pendencia real remanescente: aplicar migrations/seeds no MySQL real, validar `POST /api-v2/client-assignments/transfer` e a UI local do painel no EasyPanel com usuario real e CSRF real, conectar carteira/transferencias em lote ao modelo normalizado e planejar backfill de `clients.metadata` para `client_department_assignments`.

## Atualizacao operacional de 2026-05-27 16:31 -03:00

Foi implementada a primeira fundacao backend para responsabilidade normalizada de clientes:

- criado modelo Prisma `ClientDepartmentAssignment` e migration `20260527162000_add_client_department_assignments`;
- criada tabela alvo `client_department_assignments`, vinculando cliente, departamento controlado, responsavel operacional, gestor opcional, tipo, status, periodo, criador/atualizador e metadata;
- criado `ClientAssignmentsModule` com endpoints protegidos:
  - `GET /api-v2/clients/:clientId/assignments`;
  - `POST /api-v2/clients/:clientId/assignments`;
  - `PATCH /api-v2/client-assignments/:id`;
  - `POST /api-v2/client-assignments/:id/end`;
- RBAC ganhou `client_assignments.read/create/update/transfer/end/audit`;
- o backend valida departamento pelo catalogo controlado, responsavel/gestor ativos, usuario interno e bloqueio de dois responsaveis `PRIMARY` ativos para o mesmo cliente/departamento;
- toda criacao, atualizacao e encerramento registra auditoria centralizada.

Validacao local:

- `npm.cmd run prisma:format` passou;
- `npm.cmd run prisma:generate` passou;
- testes focados `client-assignments.service`, `default-rbac` e `department-catalog.service` passaram;
- `npm.cmd run prisma:validate`, `npm.cmd run lint` e `npm.cmd run build` na API passaram.

Pendencia real remanescente: aplicar as migrations `20260527153000_add_controlled_departments` e `20260527162000_add_client_department_assignments` no MySQL real, rodar `prisma:seed`, validar os endpoints com usuarios reais e implementar a UI/migracao gradual que priorize a nova tabela antes do fallback em `clients.metadata`.

## Atualizacao operacional de 2026-05-27 16:11 -03:00

Foi implementada a segunda pendencia incremental de governanca operacional/UX:

- `CollaboratorsService` passou a validar `mainDepartment`/`departamento` contra o catalogo controlado em cadastro, edicao e filtro;
- departamento invalido/inativo em colaborador retorna `DEPARTMENT_NOT_FOUND` antes de gravar dados;
- `/dev/colaboradores`, `/dev/colaboradores/novo` e `/colaboradores/visao-geral` usam `select` carregado de `GET /api-v2/departments`;
- a validacao de colaborador foi alinhada a mesma regra ja aplicada a usuarios DEV/Admin.

Validacao local:

- a primeira execucao focada da API falhou por ordem de validacao em `CollaboratorsService.create`;
- a ordem foi corrigida para validar departamento antes de resolver roles;
- testes focados `collaborators.service`, `users.service`, `department-catalog.service` e `default-rbac` passaram;
- `npm.cmd run prisma:validate`, `npm.cmd run lint` e `npm.cmd run build` na API passaram;
- `npm.cmd run build` e `npm.cmd run lint` no Web passaram.

Percentuais operacionais permanecem inalterados: a mudanca reduz risco local de UX/governanca, mas ainda falta publicar, aplicar migration/seed e validar com usuarios reais no EasyPanel.

Pendencia real remanescente: aplicar migration/seed de departamentos no MySQL real, validar usuarios e colaboradores com usuario ADMIN/DEV real, e evoluir clientes/responsabilidades para entidade propria cliente x departamento x usuario/gestor.

## Atualizacao operacional de 2026-05-27 15:35 -03:00

Foi implementada a primeira pendencia de governanca operacional/UX:

- criado catalogo controlado de departamentos no backend (`Department`/`departments`);
- criada migration `20260527153000_add_controlled_departments`;
- `prisma:seed` passa a popular departamentos padrao;
- `GET /api-v2/departments` lista o catalogo protegido por `departments.read`;
- RBAC ganhou `departments.read` e `departments.manage`;
- `UsersService` valida `mainDepartment` contra o catalogo e rejeita valores livres inexistentes/inativos;
- a tela `/dev` usa `select` carregado da API para filtro, criacao e edicao de usuarios.

Validacao local:

- `npm.cmd run prisma:generate` na API passou;
- testes focados `users.service`, `department-catalog.service` e `default-rbac` passaram;
- `npm.cmd run prisma:validate`, `npm.cmd run lint` e `npm.cmd run build` na API passaram;
- `npm.cmd run build` e `npm.cmd run lint` no Web passaram.

Pendencia real remanescente: aplicar migration/seed no MySQL real do EasyPanel, validar `/api-v2/departments` e o formulario `/dev` com usuario ADMIN/DEV real, e expandir a mesma governanca para colaboradores, clientes/responsabilidades e demais telas com departamento.

## Atualizacao operacional de 2026-05-27 15:19 -03:00

Foi feita uma consolidacao documental das pendencias ainda abertas em `portal-sama-docs`, incluindo explicitamente:

- `integracao_acessorias_entregas_vencimentos.md`;
- `ux-ui-docs/README.md`;
- `ux-ui-docs/09-checklist-aceite.md`;
- `ux-ui-docs/10-departamentos-solicitacoes-permissoes.md`;
- `ux-ui-docs/11-responsabilidade-clientes-usuarios.md`.

Leitura consolidada:

- Acessorias/Entregas/Vencimentos esta planejado como fase futura de alta prioridade, posterior a migracao base NestJS/React, RBAC, auditoria, notificacoes, planilhas departamentais, modulo base de integracoes e testes minimos.
- UX/UI nao substitui QA funcional, mas e o ultimo gate de experiencia antes de liberar usuarios: navegacao, Home por perfil, mobile, documentos, configuracoes, seguranca visivel, solicitacoes, departamentos, roles/permissoes e responsabilidades precisam ser aceitos.
- Responsabilidade de clientes por usuario ainda nao deve depender exclusivamente de `clients.metadata`; a trilha recomendada e criar entidade propria cliente x departamento x usuario/gestor, com historico e auditoria.

Decisao operacional:

- Acessorias nao entra como requisito de corte da stack base se for formalmente mantido fora do escopo do deploy inicial.
- UX/UI entra como pendencia de aceite antes de liberar uso amplo aos usuarios.
- Departamentos, solicitacoes, permissoes e responsabilidade de clientes entram como trilha funcional/seguranca porque afetam escopo, carteira, visibilidade e auditoria.

## Atualizacao operacional de 2026-05-27 14:06 -03:00

Foi corrigida a decisao do leiaute Dominio do Integra-AI para o fluxo real do escritorio.

Evidencia:

- o arquivo esta sendo importado no conjunto correto `Lancamentos Contabeis em Lote`;
- nesse conjunto, `0451|...` e interpretado como registro fixo `04 - Rateios Gerenciais`;
- por isso os campos do arquivo com separador ficavam deslocados no Dominio.

Decisao corrigida:

- leiaute oficial do Portal Sama para o Integra-AI: `dominio_importador_lancamentos_lote_01_02_03_99`;
- label: `Dominio Importador 01/02/03/99`;
- nao usar `dominio_separador_0000_0451` no fluxo principal.

Implementacao local:

- backend gera TXT sem pipes, iniciando com `01`, contendo pares `02`/`03` e finalizando com `99`;
- registros possuem tamanhos fixos esperados: 54, 150, 664 e 100 caracteres;
- frontend exibe `Dominio Importador 01/02/03/99`;
- passaram testes focados da API, suite completa da API, lint/build/Prisma validate da API, lint/build do Web, Playwright focado de Integra-AI e Playwright local completo.

Pendencia para producao:

- importar TXT real no Dominio usando `Lancamentos Contabeis em Lote`;
- salvar evidencia sanitizada;
- congelar golden file aprovado.

## Atualizacao operacional de 2026-05-27 12:16 -03:00

O Integra-AI foi padronizado localmente para somente um leiaute Dominio oficial no fluxo principal: `dominio_separador_0000_0451` (`Dominio Sistemas com Separador - 0000/0451`).

Mudancas aplicadas:

- frontend removeu o seletor entre `Dominio 01/02/03/99` e `Dominio 0000/0451`;
- backend gera somente registros `0000` e `0451`;
- API rejeita `export_strategy` divergente;
- TXT e validado antes da exportacao;
- exportacao/download registram leiaute, status e hash em auditoria;
- download privado recebeu headers anti-cache adicionais.

Validacao local:

- testes focados e suite completa da API passaram;
- lint/build da API e do Web passaram;
- Prisma validate passou;
- Playwright focado do Integra-AI e Playwright local completo passaram.

Pendencia para producao:

- homologar TXT real no Dominio Contabilidade Fiscal com importador de separador `0000/0451`;
- salvar evidencia sanitizada;
- congelar golden file aprovado;
- publicar Web/API atualizados e repetir validacao com usuario contabil real.

## Atualizacao operacional de 2026-05-27 11:05 -03:00

Foi corrigido no `portal-sama-web` o erro de download TXT do Integra-AI relatado como `UNAUTHORIZED`/`Sessao expirada ou token invalido`.

- Causa tecnica provavel: a tela usava anchor/link direto para `/api-v2/accounting/integra-ai/jobs/:id/download`; como o access token da stack React fica em memoria, a navegacao direta nao enviava `Authorization: Bearer`.
- `portal-sama-web/src/services/integra-ai.service.ts` agora baixa o TXT via `api.get(..., { responseType: 'blob' })`, reaproveitando o interceptor autenticado.
- `portal-sama-web/src/pages/accounting/IntegraAiPage.tsx` trocou os links por botoes de download com estado de carregamento, erro controlado e filename a partir do `Content-Disposition` quando disponivel.
- A pagina recebeu classe `integra-ai-page` e CSS proprio para ficar alinhada ao lado da sidebar; o smoke Playwright valida que ela comeca apos a sidebar com folga controlada.
- `portal-sama-web/tests/e2e/smoke.spec.ts` passou a cobrir o download TXT autenticado, conferindo `Authorization: Bearer access-e2e` no request mockado.
- `eslint.config.js` passou a ignorar `test-results` e `playwright-report`, evitando falha local quando lint e Playwright manipulam artefatos temporarios.
- Validacao local no Web: `npm.cmd run build`, `npm.cmd run test:e2e -- -g "Integra-AI"`, `npm.cmd run lint` e `git diff --check` passaram.

Pendencia real remanescente: publicar o novo build do Web e validar em homologacao real com usuario contabil, permissao `accounting.integra_ai.download`, job real, TXT real e auditoria de download. Percentuais operacionais permanecem inalterados, porque a correcao ainda nao foi validada no ambiente publicado.

## Atualizacao operacional de 2026-05-27 10:40 -03:00

Foi reproduzida falha com PDF real do Banco Inter (`Extrato Inter dezembro 25.pdf` em unidade local). O parser detectava banco/periodo, mas retornava `rows: []` porque o layout do Inter usa data longa com `Saldo do dia` e lista transacoes abaixo sem repetir a data.

- `portal-sama-api/services/integra_ai_parser/parse_statement.py` ganhou parser `inter_pdf`.
- O parser reconhece datas como `1 de Dezembro de 2025`, carrega a data atual ate a proxima data, junta descricoes quebradas em multiplas linhas e interpreta valores com `R$`/`-R$`.
- `Saldo do dia` agora entra apenas como `row_kind=balance_snapshot` e `ignore_reason=daily_balance`.
- O fluxo contabil continua persistindo apenas `row_kind=transaction`; portanto saldo diario nao aparece para o usuario final, nao vira regra e nao entra no TXT Dominio.
- Validacao local no PDF real: parser retornou 31 transacoes e 15 saldos tecnicos. Passaram `python -m py_compile`, `npm.cmd test -- accounting.service.spec.ts --runInBand`, `npm.cmd run lint` e `npm.cmd run build` na API.

Decisao: saldo do dia deve ser guardado como dado tecnico para reconciliacao/diagnostico, nao como lancamento contabil. Pode servir futuramente para alertar divergencia entre saldo final do dia e ultimo saldo acumulado lido, mas nao deve ser exibido na etapa final nem exportado.

## Atualizacao operacional de 2026-05-27 10:03 -03:00

Foi analisada a documentacao externa `Downloads/docs/integra-ai/analise_semantica_integra_ai_portal_sama.md`. A recomendacao condiz com o estado do projeto: o parser Python ja suporta OFX, mas API/Web publicavam apenas PDF. A implementacao foi feita como rollout controlado:

- `portal-sama-api` ganhou `parseFile({ ext })` no parser, validacao/storage genericos para PDF/OFX e capabilities `pdf_import`/`ofx_import`.
- OFX fica desabilitado por padrao por `SAMA_INTEGRA_AI_OFX_IMPORT_ENABLED=false`; quando habilitado, a API valida extensao, MIME, assinatura OFX, tamanho, scanner/quarentena e audita `source_type=ofx`.
- `portal-sama-web` ajustou `/contabil/integra-ai` para aceitar dinamicamente PDF ou OFX conforme capability, mantendo compatibilidade com ambientes antigos que so retornam `import_mutation`.
- Nao houve migration de banco; o campo legado `source_type` passa a registrar `pdf` ou `ofx`.
- Validacao local: `npm.cmd test -- accounting.service.spec.ts --runInBand`, `npm.cmd run lint`, `npm.cmd run build` e `npm.cmd run prisma:validate` passaram na API; `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e -- -g "Integra-AI"` e `npm.cmd run test:e2e` passaram no Web.

Pendencia real remanescente: habilitar a flag somente em homologacao controlada, importar OFX real com usuario contabil, parser Python do container, MySQL/storage reais, ClamAV strict e evidencia sanitizada. Percentuais operacionais permanecem inalterados, porque a mudanca reduz risco tecnico mas nao substitui homologacao real.

## Atualizacao operacional de 2026-05-27 08:31 -03:00

Nesta rodada foi reduzida uma pendencia local/parcial do fluxo Integra-AI no frontend:

- `portal-sama-web/src/pages/accounting/IntegraAiPage.tsx` recebeu labels acessiveis nos campos customizados de contas.
- `portal-sama-web/tests/e2e/smoke.spec.ts` ganhou smoke Playwright para `/contabil/integra-ai`, com API mockada, cobrindo job aberto, pagina 2 da tabela de regras, busca por nome de conta, selecao de sugestao e autosave por blur/codigo numerico.
- `portal-sama-web/playwright.config.ts` passou a executar os smokes locais do mesmo arquivo em ordem para evitar concorrencia Vite/GSAP observada no Windows; `npm.cmd run test:e2e` voltou a passar no comando padrao.
- Validacao local: `npm.cmd run test:e2e -- -g "Integra-AI"`, `npm.cmd run test:e2e`, `npm.cmd run lint` e `npm.cmd run build` passaram no Web.

Pendencia real remanescente: validar o fluxo com job real do Integra-AI, usuario contabil, dados reais e MySQL real em homologacao apos deploy. Percentuais operacionais permanecem inalterados, porque esta rodada adiciona cobertura local e nao substitui a homologacao funcional real.

## Atualizacao operacional de 2026-05-26 16:50 -03:00

Nesta rodada foi criado o plano executavel de rollback/restore drill:

- `portal-sama-api` agora expoe `npm run ops:restore:drill`.
- O comando roda em dry-run por padrao, chama `ops:backup:verify`, valida o alvo de banco e storage, bloqueia alvo igual a `DATABASE_URL`/`STORAGE_PRIVATE_PATH` e exige `--confirm RESTORE_DRILL_TARGET_IS_ISOLATED` para aplicar restore.
- `PLANO_ROLLBACK_RESTORE_DRILL.md` foi criado com procedimento, evidencias esperadas e criterio para fechar a pendencia de backup/rollback.
- Validacao local: `node --check`, help do comando, backup descartavel sem banco com archive de storage, dry-run do restore, restore de storage em `.ai-tests`, lint/build/Prisma validate da API e `git diff --check` passaram.

Pendencia real remanescente: gerar backup real no EasyPanel, verificar artefatos reais, copiar para fora do container, criar banco/storage isolados e executar `ops:restore:drill` com evidencia sanitizada.

## Atualizacao operacional de 2026-05-26 09:38 -03:00

Depois que o `.env` real da API foi alocado localmente e confirmado como ignorado pelo Git:

- `DATABASE_URL` foi carregado sem imprimir segredo; ele aponta para `portal-sama_portal-sama-database:3306`, hostname interno do EasyPanel, portanto nao conecta a partir do Windows local.
- `npm.cmd run ops:backfill:report -- --soft --json` confirmou bloqueio local de conectividade com esse hostname interno.
- `npm.cmd run ops:readiness -- --skip-clamav --storage-path .ai-tests/readiness-storage --soft --json` confirmou o mesmo bloqueio de banco local e agora tambem falha se segredos locais (`JWT_*`, `CSRF_SECRET`, `CERTIFICATE_ENCRYPTION_KEY`) tiverem menos de 32 caracteres.
- Usando `SAMA_BOOTSTRAP_ADMIN_USERNAME`/`SAMA_BOOTSTRAP_ADMIN_PASSWORD` apenas no processo do shell, `npm.cmd run smoke:auth -- --soft --json` passou contra HTTPS: CSRF, login, `/auth/me`, refresh, novo `/auth/me` e logout.
- Uma matriz minima `BOOTSTRAP_ADMIN` passou em `npm.cmd run smoke:permissions -- --soft --json`: anonimos 401 em `/auth/me`, `/users`, `/documents`; bootstrap/DEV 200 em `/auth/me`, `/users`, `/roles`, `/permissions`.
- `npm.cmd run test:e2e:real` passou com usuario real bootstrap/DEV.
- `npm.cmd run homologation:real -- --soft --evidence-dir .ai-tests/homologation-real` passou a bateria consolidada real e gerou evidencia JSON sanitizada local.

Pendencia real remanescente: rodar backfill/backup/restore dentro do console do container `portal-sama-api` no EasyPanel, gerar matriz oficial por multiplos perfis reais, validar upload/download real com token/storage e executar QA final antes do desligamento seguro do legado.

## Atualizacao operacional de 2026-05-26 09:23 -03:00

Nesta rodada foi iniciada a execucao dos itens reais pendentes com o ambiente local disponivel:

- `npm.cmd run smoke:public` passou novamente contra `https://portal.samacontabil.com.br`, com frontend 200, health `database=up/storage=up`, CORS 204 e CSRF 200.
- `npm.cmd run smoke:auth -- --soft --json` foi executado e bloqueou por ausencia de `PORTAL_AUTH_USERNAME`/`PORTAL_AUTH_PASSWORD` no shell local.
- `npm.cmd run smoke:permissions -- --soft --json` validou os anonimos reais com 401 em `/auth/me`, `/users` e `/documents`, mas bloqueou a matriz autenticada por ausencia de `PORTAL_PERMISSION_MATRIX_FILE`/`PORTAL_PERMISSION_MATRIX_JSON`.
- `npm.cmd run test:e2e:real` foi executado e ficou `skipped`, como esperado, porque `PORTAL_REAL_E2E=1` e credenciais Playwright reais nao estavam definidos.
- `npm.cmd run ops:backfill:report -- --soft --json` foi executado e bloqueou por ausencia de `DATABASE_URL` no ambiente local.
- `npm.cmd run ops:readiness -- --soft --json` agora retorna JSON controlado com falhas de ambiente/banco/storage ausentes, em vez de abortar a coleta por erro Prisma.
- `portal-sama-web` agora expoe `npm.cmd run homologation:real`, que roda `smoke:public`, `smoke:auth`, `smoke:permissions` e `test:e2e:real` em sequencia, com preflight sanitizado de variaveis e evidencia JSON opcional em `.ai-tests/homologation-real`.

Pendencia real remanescente: carregar credenciais reais/matriz no shell, rodar `npm.cmd run homologation:real -- --evidence-dir .ai-tests/homologation-real`, executar backfill/backup/restore no container EasyPanel com `DATABASE_URL`/storage reais e anexar evidencias sanitizadas.

## Atualizacao operacional de 2026-05-26 09:10 -03:00

Nesta rodada foi disponibilizado o verificador operacional de artefatos de backup:

- `portal-sama-api` agora expoe `npm run ops:backup:verify`.
- O script verifica `metadata.json`, status dos passos, SHA-256/tamanho de `database.sql.gz`, integridade gzip, hash/consistencia de `storage-manifest.json` e, quando existir, SHA-256/listagem de `storage.tar.gz`.
- O readiness passou a orientar `ops:backup:create -> ops:backup:verify -> copiar artefatos para fora -> restore drill`.
- Validacoes locais: `node --check`, help do comando, backup local descartavel com `--skip-database`/manifesto/archive, verificacao JSON do backup gerado, readiness local com skips, build da API, `prisma:validate` com `DATABASE_URL` dummy e `git diff --check` passaram em `portal-sama-api`.

Pendencia real remanescente: rodar `npm run ops:backup:create` e `npm run ops:backup:verify` no EasyPanel com banco/storage reais, copiar artefatos para fora do container e executar restore drill em alvo isolado. O verificador nao substitui a restauracao real.

## Atualizacao operacional de 2026-05-26 08:49 -03:00

Nesta rodada foi disponibilizado o smoke operacional de matriz de permissoes por perfil:

- `portal-sama-web` agora expoe `npm.cmd run smoke:permissions`.
- O script valida anonimos esperando 401 em `/auth/me`, `/users` e `/documents`.
- A matriz autenticada e parametrizada por JSON/arquivo e aceita `usernameEnv`/`passwordEnv` para evitar credenciais em arquivos versionados.
- Cada perfil pode declarar checks por metodo, rota e status esperado, por exemplo ADMIN 200 em `/users`, CLIENT 403 em `/users` e CLIENT 200 em `/documents`.
- O script mantem cookies apenas em memoria, mascara usuarios e nao imprime senha, access token, refresh token ou CSRF token.
- Validacoes locais: `node --check`, help do comando, smoke `--soft --json` contra porta fechada, smoke positivo contra servidor fake local, lint, build e `git diff --check` passaram em `portal-sama-web`.

Pendencia real remanescente: rodar `npm.cmd run smoke:permissions` contra `https://portal.samacontabil.com.br/api-v2` com usuarios reais por perfil, sem registrar credenciais/tokens/cookies, e anexar evidencia sanitizada da matriz 401/403/200.

## Atualizacao operacional de 2026-05-25 17:54 -03:00

Nesta rodada foi disponibilizada a base Playwright para homologacao real de autenticacao via navegador:

- `portal-sama-web` agora expoe `npm.cmd run test:e2e:real`.
- A config `playwright.real.config.ts` aponta para `https://portal.samacontabil.com.br`, nao sobe servidor local e desliga trace, screenshot e video.
- A spec `tests/e2e/real-auth.spec.ts` fica bloqueada por `PORTAL_REAL_E2E=1` e credenciais em variaveis de ambiente.
- O teste cobre login pela tela React, Home autenticada, ausencia de chaves sensiveis no storage do navegador, refresh cookie fora de `document.cookie`, politica `HttpOnly`/`SameSite`/`Secure` e logout.
- Validacoes locais: `npm.cmd run test:e2e:real` sem variaveis passou com 1 skipped; `npm.cmd run lint`, `npm.cmd run build`, `npm.cmd run test:e2e` com 9 passados e 1 skipped, e `git diff --check` passaram em `portal-sama-web`.

Pendencia real remanescente: rodar `PORTAL_REAL_E2E=1 npm.cmd run test:e2e:real` contra `https://portal.samacontabil.com.br` com usuario real de homologacao, sem registrar credenciais/tokens/cookies, e repetir por perfis criticos.

## Atualizacao operacional de 2026-05-25 17:37 -03:00

Nesta rodada foi disponibilizada a ferramenta operacional para validar autenticacao real no EasyPanel:

- `portal-sama-web` agora expoe `npm.cmd run smoke:auth`.
- O script executa `csrf -> login -> me -> refresh -> me -> logout`.
- O smoke mantem cookies apenas em memoria, mascara o usuario e nao imprime senha, access token, refresh token ou CSRF token.
- A validacao local passou com servidor fake e o script checa `HttpOnly`, `SameSite` e `Secure` no refresh cookie quando a API esta em HTTPS.

Pendencia real remanescente: rodar `npm.cmd run smoke:auth` contra `https://portal.samacontabil.com.br/api-v2` com usuario real de homologacao, sem registrar credenciais ou tokens, e repetir por perfis criticos.

## Atualizacao operacional de 2026-05-25 17:27 -03:00

Nesta rodada foi concluida uma pendencia de aderencia entre documentacao, API e Web no fluxo publico de documentos de onboarding:

- `portal-sama-api` agora expoe os aliases documentados `GET /api-v2/public/onboarding/documents/:token` e `POST /api-v2/public/onboarding/documents/:token/upload`, mantendo os endpoints genericos antigos como compatibilidade.
- `portal-sama-web` agora consome esses aliases na pagina publica de documentos do cliente.
- Foi adicionado teste unitario focado na controller de documentos e o smoke Playwright local do Web passou mockando o alias novo.
- A documentacao viva foi atualizada em `STATUS_IMPLEMENTACAO.md`, `PENDENCIAS_TECNICAS.md`, `RELATORIO_TESTES.md`, `CHANGELOG_TECNICO.md`, matrizes de migracao, seguranca, inventario e pagina de onboarding.

Validacoes locais registradas nesta rodada: teste unitario da API, lint da API, build da API, `prisma:validate`, lint do Web, build do Web, smoke Playwright local do fluxo publico de documentos e `git diff --check` nos tres repositorios.

Pendencia real remanescente: validar no EasyPanel o fluxo publico completo com token real, MySQL real, storage privado real, ClamAV em modo strict e upload de arquivo real pelo navegador.

## Separacao em repositorios Bitbucket

### Estado local verificado em 2026-05-22 15:16 -03:00

As pastas ja foram separadas no workspace local:

```txt
C:\Users\Sama Contabilidade\Desktop\portal-sama-docs
C:\Users\Sama Contabilidade\Desktop\portal-sama-api
C:\Users\Sama Contabilidade\Desktop\portal-sama-web
```

Estado encontrado:

| Workspace | Estado local | Git | Observacao |
|---|---|---|---|
| `portal-sama-docs` | Separado | Git local inicializado em `main` | Contem os documentos na raiz do repo, com `paginas/` e `design/`; `.gitignore` e `.gitattributes` preparados. |
| `portal-sama-api` | Separado | Git local inicializado em `main` | Contem `.env`, `node_modules/` e `dist/` ignorados; `.gitignore` e `.gitattributes` preparados. |
| `portal-sama-web` | Separado | Git local inicializado em `main` | Contem `node_modules/`, `dist/`, `playwright-report/` e `test-results/` ignorados; `.gitignore` e `.gitattributes` preparados. |

Ponto importante: a documentacao esta na raiz de `portal-sama-docs`, nao dentro de uma subpasta `docs/`. Ao trabalhar nesse repositorio separado, leia os arquivos diretamente na raiz, por exemplo `INSTRUÇÕEDS_AI.MD` e `STATUS_IMPLEMENTACAO.md`.

Estado real informado em 2026-05-25 11:46 -03:00: os repositorios Bitbucket ja existem e o EasyPanel ja esta consumindo os repos tecnicos para deploy. A evidencia visual mostra:

```txt
portal-sama-api
portal-sama-web
portal-sama-doc
```

O workspace local de documentacao continua chamado `portal-sama-docs`. Se o remoto permanecer como `portal-sama-doc`, tratar esse nome como alias operacional do repositorio de documentacao.

Topologia-alvo de repositorios:

```txt
portal-sama-api
portal-sama-web
portal-sama-docs ou portal-sama-doc
```

Regra central: qualquer IA, pessoa desenvolvedora ou manutencao operacional deve ler primeiro o repositorio `portal-sama-docs` antes de alterar `portal-sama-api` ou `portal-sama-web`.

### Repositorio `portal-sama-api`

Conteudo esperado: apenas os arquivos de `portal-sama-api/` como raiz do repositorio.

Incluir:

```txt
Dockerfile
.dockerignore
package.json
package-lock.json
nest-cli.json
tsconfig.json
tsconfig.build.json
eslint.config.mjs
prisma/
src/
test/
scripts/
.env.example
```

Nao incluir:

```txt
.env
node_modules/
dist/
coverage/
.ai-tests/
logs/
arquivos privados
segredos reais
```

No EasyPanel, se o repo for realmente separado, o Dockerfile da API fica na raiz:

```txt
Dockerfile
```

Se o repo atual monolitico for usado temporariamente, configurar Root Directory como:

```txt
portal-sama-api
```

### Repositorio `portal-sama-web`

Conteudo esperado: apenas os arquivos de `portal-sama-web/` como raiz do repositorio.

Incluir:

```txt
Dockerfile
.dockerignore
nginx.conf
package.json
package-lock.json
vite.config.ts
index.html
public/
src/
tests/
playwright.config.ts
.env.example, se existir
```

Nao incluir:

```txt
.env
.env.local
node_modules/
dist/
coverage/
test-results/
playwright-report/
.ai-tests/
segredos reais
```

No EasyPanel, se o repo for realmente separado, o Dockerfile do Web fica na raiz:

```txt
Dockerfile
```

Se o repo atual monolitico for usado temporariamente, configurar Root Directory como:

```txt
portal-sama-web
```

### Repositorio `portal-sama-docs`

Conteudo esperado: toda a documentacao do projeto, incluindo os documentos de acompanhamento, arquitetura, seguranca, deploy, matrizes, inventarios, paginas mapeadas e design.

Incluir:

```txt
*.md
paginas/
design/
ux-ui-docs/
README.md especifico do repositorio de documentacao
.gitignore
```

O repositorio `portal-sama-docs` deve conter, no minimo:

```txt
INSTRUÇÕEDS_AI.MD
AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD
STATUS_IMPLEMENTACAO.md
CHANGELOG_TECNICO.md
DECISOES_ARQUITETURA.md
PENDENCIAS_TECNICAS.md
RELATORIO_TESTES.md
EASYPANEL_DEPLOY.md
README_DOCUMENTACAO.md
SEGURANCA.md
INVENTARIO_SEGURANCA.md
INVENTARIO_BANCO_DADOS.md
INVENTARIO_ENDPOINTS.md
MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md
MATRIZ_MIGRACAO_HTML_PARA_REACT.md
integracao_acessorias_entregas_vencimentos.md
ux-ui-docs/
paginas/
design/
```

Nao incluir:

```txt
.env
node_modules/
.ai-tests/
arquivos privados
logs com dados reais
documentos/certificados reais de clientes
segredos reais
```

### Regra de leitura entre repositorios

Depois da separacao:

1. Toda tarefa inicia em `portal-sama-docs`.
2. A IA deve ler `INSTRUÇÕEDS_AI.MD` e os documentos relacionados.
3. Somente depois disso deve abrir `portal-sama-api` ou `portal-sama-web`.
4. Ao alterar API ou Web, atualizar a documentacao correspondente em `portal-sama-docs`.
5. O commit de documentacao deve acompanhar ou anteceder o commit tecnico.
6. Quando a mudanca afetar deploy/producao, atualizar este documento e `docs/EASYPANEL_DEPLOY.md`.

Resolvido operacionalmente em 2026-05-25 11:09 -03:00: `portal-sama-api/README.md` foi criado e `portal-sama-web/README.md` foi reforcado com a regra minima de leitura do workspace irmao `portal-sama-docs`, incluindo os documentos obrigatorios antes de alterar API/Web.

## Configuracao necessaria no EasyPanel

### Servicos minimos

```txt
portal-sama-web
portal-sama-api
portal-sama-database
```

O banco ja informado no EasyPanel e:

```txt
Host interno observado: portal-sama-database
Porta: 3306
Banco: banco-sama
Versao vista no log EasyPanel: MySQL Community Server 9.7.0
```

### Variaveis essenciais da API

Configurar no EasyPanel, nunca em arquivo versionado:

```env
NODE_ENV=production
PORT=3000
API_PREFIX=api-v2
FRONTEND_URL=https://portal.samacontabil.com.br
CORS_ORIGIN=https://portal.samacontabil.com.br
DATABASE_URL=mysql://portal_user:SENHA_FORTE@HOST_INTERNO_MYSQL:3306/banco-sama
PRISMA_CONNECT_ON_BOOT=true
PRISMA_MIGRATE_ON_START=false

JWT_ACCESS_SECRET=gerar_chave_forte
JWT_REFRESH_SECRET=gerar_outra_chave_forte
CSRF_SECRET=gerar_chave_csrf_forte

COOKIE_NAME=sama_refresh_token
COOKIE_DOMAIN=.samacontabil.com.br
COOKIE_SECURE=true
COOKIE_SAME_SITE=lax

STORAGE_PRIVATE_PATH=/var/private/portal-sama
CERTIFICATE_STORAGE_PATH=/var/private/portal-sama/certificates
CONTRACT_PDF_STORAGE_PATH=/var/private/portal-sama/contracts

SAMA_UPLOAD_SCAN_MODE=strict
SAMA_UPLOAD_SCAN_TIMEOUT_SEC=45
SAMA_UPLOAD_SCAN_BIN=/usr/bin/clamscan
SAMA_UPLOAD_QUARANTINE_DIR=/var/private/portal-sama/uploads/_quarantine
SAMA_CLAMAV_UPDATE_ON_START=false
SAMA_BACKUP_DIR=/var/private/portal-sama/_ops-backups
SAMA_MYSQL_DUMP_BIN=
SAMA_MYSQL_CLIENT_BIN=
SAMA_RESTORE_BACKUP_DIR=
SAMA_RESTORE_TARGET_DATABASE_URL=mysql://portal_restore:SENHA_FORTE@portal-sama-database:3306/banco_sama_restore_drill
SAMA_RESTORE_TARGET_STORAGE_PATH=/tmp/portal-sama-restore-storage
SAMA_RESTORE_DRILL_CONFIRM=

WEB_PUSH_PUBLIC_KEY=
WEB_PUSH_PRIVATE_KEY=
WEB_PUSH_SUBJECT=mailto:suporte@samacontabil.com.br
```

Substituir `HOST_INTERNO_MYSQL` pelo host real exibido no EasyPanel/phpMyAdmin/log. Exemplo atualmente observado: `portal-sama-database`. Exemplos historicos ja observados em logs antigos: `portal-sama_database` e `portal-sama_portal-sama_database`.

Observacao 2026-05-22 16:08 -03:00: o erro Prisma `P1012` com `Environment variable not found: DATABASE_URL` indica que a variavel nao chegou ao build/runtime da API. O Dockerfile de `portal-sama-api` agora usa uma URL dummy somente para `prisma generate` no build; a `DATABASE_URL` real continua obrigatoria nas variaveis do servico `portal-sama-api` no EasyPanel antes de `prisma migrate deploy` e start da API.

Observacao 2026-05-22 16:38 -03:00: o erro Prisma `P3005` com `The database schema is not empty` indica banco existente sem historico `_prisma_migrations`. O start da API nao roda mais migrations por padrao (`PRISMA_MIGRATE_ON_START=false`), para o container ficar online. A preparacao do banco existente deve ser feita uma unica vez com backup e `SAMA_PRISMA_BASELINE_EXISTING_DATABASE=true npm run prisma:migrate:baseline-existing`.

### Usuario MySQL da aplicacao

Nao usar `root` na API.

Criar usuario proprio:

```sql
CREATE USER 'portal_user'@'%' IDENTIFIED BY 'SENHA_FORTE';
GRANT ALL PRIVILEGES ON `banco-sama`.* TO 'portal_user'@'%';
FLUSH PRIVILEGES;
```

### Comandos iniciais no container da API

Depois de publicar a API e configurar o `DATABASE_URL` real em banco limpo:

```bash
npx prisma migrate deploy
npm run prisma:seed
SAMA_BOOTSTRAP_ADMIN_USERNAME=admin \
SAMA_BOOTSTRAP_ADMIN_PASSWORD='senha_forte_temporaria' \
SAMA_BOOTSTRAP_ADMIN_NAME='Administrador Portal Sama' \
SAMA_BOOTSTRAP_ADMIN_ROLE=DEV \
npm run prisma:bootstrap-admin
```

Se o banco existente retornar `P3005`, usar o baseline controlado antes do seed/bootstrap:

```bash
SAMA_PRISMA_BASELINE_EXISTING_DATABASE=true npm run prisma:migrate:baseline-existing
```

Observacao 2026-05-25 09:25 -03:00: se o container atual ainda falhar em `npm run prisma:seed` com `Cannot find module '../src/modules/rbac/default-rbac'`, ele esta usando uma imagem anterior ao wrapper de runtime. Workaround imediato: `node dist/prisma/seed.js`. Correcao definitiva: redeployar `portal-sama-api` com o script atualizado.

Observacao 2026-05-25 09:48 -03:00, atualizada em 2026-05-25 11:46 -03:00: se o deploy falhar com Prisma `P1013` e `invalid port number in database URL`, o problema e o valor de `DATABASE_URL`, nao as migrations. Usar o formato `mysql://portal_user:SENHA_URL_ENCODED@portal-sama-database:3306/banco-sama`, sem porta duplicada e com caracteres especiais da senha codificados por `encodeURIComponent`.

Atualizacao operacional 2026-05-25 11:46 -03:00: o banco do EasyPanel ja esta conectado a API, usuarios iniciais da aplicacao ja foram criados pelo operador e o health publico retornou `database=up`. Nao registrar nomes, e-mails ou senhas reais nesta documentacao.

Depois trocar/gerenciar a senha por fluxo seguro e manter usuarios reais auditados.

## Ordem segura para homologacao

1. [x] Criar o repositorio de documentacao no Bitbucket (`portal-sama-doc` observado; alias local `portal-sama-docs`).
2. [x] Subir documentacao inicial para o repositorio de documentacao.
3. [x] Criar o repositorio `portal-sama-api` no Bitbucket.
4. [x] Subir `portal-sama-api` para o Bitbucket.
5. [x] Criar o repositorio `portal-sama-web` no Bitbucket.
6. [x] Subir `portal-sama-web` para o Bitbucket.
7. [x] Conferir que os tres repositorios apontam para a mesma decisao/documentacao vigente.
8. [x] Criar servico API no EasyPanel apontando para o repo da API.
9. [x] Criar servico Web no EasyPanel apontando para o repo do Web.
10. [x] Confirmar que `portal-sama-web` resolve/proxy `/api-v2` para `portal-sama-api`.
11. [x] Configurar `DATABASE_URL` para o banco real do EasyPanel.
12. Criar usuario MySQL nao-root.
13. [x] Disponibilizar comando operacional de backup no container da API (`npm run ops:backup:create`).
14. [x] Disponibilizar verificador operacional dos artefatos de backup (`npm run ops:backup:verify`).
14.1. [x] Disponibilizar restore drill operacional protegido (`npm run ops:restore:drill`) e plano documentado em `PLANO_ROLLBACK_RESTORE_DRILL.md`.
15. Rodar `npx prisma migrate deploy` em banco limpo ou `SAMA_PRISMA_BASELINE_EXISTING_DATABASE=true npm run prisma:migrate:baseline-existing` em banco existente com `P3005`.
16. Rodar `npm run prisma:seed`.
17. Rodar `npm run prisma:bootstrap-admin`.
18. [x] Rodar `npm run ops:readiness` no container da API sem `--skip` (passou com warning apenas de backup/rollback).
19. Rodar `npm run ops:backfill:report` no container da API e anexar JSON/evidencia ao plano de backfill.
20. [x] Validar `GET /api-v2/health` publicamente (`database=up`, `storage=up`).
21. [x] Validar `GET /api-v2/auth/csrf` publicamente com CORS/cookie CSRF.
22. [x] Validar acesso inicial ao frontend em HTTPS e criacao de usuarios iniciais.
23. Configurar volume privado e permissoes de storage.
24. [x] Ativar e validar ClamAV em modo `strict` com EICAR.
25. Executar backfills por modulo.
26. Criar/migrar usuarios reais e roles.
27. [x] Rodar smoke publico sem `--soft`.
28. [x] Implementar e testar localmente aliases publicos de documentos de onboarding na API/Web.
29. [x] Disponibilizar smoke operacional de autenticacao real (`npm.cmd run smoke:auth`).
30. [x] Rodar `npm.cmd run smoke:auth` com usuario real bootstrap/DEV.
31. [x] Disponibilizar suite Playwright opt-in contra homologacao real (`npm.cmd run test:e2e:real`).
32. [x] Disponibilizar smoke operacional de permissoes por perfil (`npm.cmd run smoke:permissions`).
33. [x] Disponibilizar bateria consolidada de homologacao real (`npm.cmd run homologation:real`).
34. [x] Rodar Playwright contra homologacao real com usuario real bootstrap/DEV.
35. [x] Rodar matriz minima `smoke:permissions` com usuario real bootstrap/DEV.
36. Rodar `npm.cmd run smoke:permissions` com usuarios reais por todos os perfis oficiais.
37. Classificar formalmente Acessorias/Entregas/Vencimentos como fase futura fora do corte base ou abrir backlog aprovado para implementacao posterior.
38. Executar aceite UX/UI com base em `ux-ui-docs/09-checklist-aceite.md`.
39. Fazer QA visual desktop/mobile.
40. Rodar `npm run ops:backup:create`, `npm run ops:backup:verify` e `npm run ops:restore:drill` no EasyPanel, copiar artefatos para fora do container e validar rollback/restore drill real.
41. Aprovar corte para producao.

## Checklist do que falta para producao

### Infraestrutura

- [x] Separar workspace local `portal-sama-api`.
- [x] Separar workspace local `portal-sama-web`.
- [x] Separar workspace local `portal-sama-docs`.
- [x] Criar `.gitignore` minimo em `portal-sama-api`.
- [x] Reforcar `.gitignore` em `portal-sama-web`.
- [x] Criar `.gitignore` e `README.md` inicial em `portal-sama-docs`.
- [x] Criar `.gitattributes` nos tres workspaces para normalizar quebras de linha.
- [x] Inicializar Git em `portal-sama-docs`.
- [x] Inicializar Git em `portal-sama-api`.
- [x] Inicializar Git em `portal-sama-web`.
- [x] Criar repo Bitbucket `portal-sama-api`.
- [x] Criar repo Bitbucket `portal-sama-web`.
- [x] Criar repo Bitbucket de documentacao (`portal-sama-doc` observado; alias local `portal-sama-docs`).
- [x] Subir documentacao inicial para o repo de documentacao.
- [x] Definir regra operacional para toda IA ler `portal-sama-docs` antes de API/Web.
- [x] Configurar builds do EasyPanel a partir dos repos.
- [x] Configurar dominio HTTPS.
- [x] Confirmar proxy `/api-v2`.
- [x] Confirmar CORS e cookie CSRF seguro em HTTPS.
- [x] Configurar logs basicos no EasyPanel.
- [x] Disponibilizar backup operacional da API (`npm run ops:backup:create`).
- [x] Disponibilizar verificador de artefatos de backup (`npm run ops:backup:verify`).
- [x] Disponibilizar restore drill operacional protegido (`npm run ops:restore:drill`) e plano de rollback.
- [ ] Rodar backup real no EasyPanel, verificar artefatos, copiar para fora do container e validar retencao.
- [x] Definir plano de rollback.

### Banco e dados

- [ ] Criar usuario MySQL nao-root.
- [x] Configurar e validar `DATABASE_URL` real no servico `portal-sama-api` do EasyPanel.
- [ ] Rodar/registrar `prisma migrate deploy` em banco limpo ou baseline controlado para `P3005` no MySQL 9.7.0 observado no EasyPanel.
- [ ] Rodar seed RBAC.
- [ ] Rodar/registrar bootstrap admin ou procedimento equivalente usado para usuarios iniciais.
- [ ] Comparar schema Prisma com schema legado.
- [x] Disponibilizar relatorio read-only de backfill/consistencia (`npm run ops:backfill:report`).
- [ ] Rodar relatorio read-only de backfill no MySQL real e anexar evidencia.
- [ ] Planejar backfills por modulo com base no relatorio real.
- [ ] Executar backfills com logs e conferencia.
- [ ] Validar contagens de registros.
- [ ] Validar vinculos cliente/usuario/gestor/departamento/carteira.

### Seguranca

- [ ] Validar login, refresh e logout em HTTPS.
- [x] Validar CSRF publico em HTTPS.
- [ ] Validar cookies `HttpOnly`, `Secure`, `SameSite`.
- [x] Disponibilizar smoke para validar login/refresh/logout e politica de refresh cookie em HTTPS.
- [x] Disponibilizar smoke para validar matriz 401/403/200 por perfil.
- [ ] Validar 401/403/200 por perfil.
- [ ] Validar IDOR em documentos, certificados, propostas e contratos.
- [ ] Validar storage privado sem acesso publico.
- [x] Disponibilizar validacao operacional de storage/ClamAV/RBAC (`npm run ops:readiness`).
- [x] Validar `npm run ops:readiness` no `portal-sama-api` sem `--skip`, com ClamAV/EICAR strict passando.
- [ ] Proteger phpMyAdmin.
- [ ] Confirmar Swagger desabilitado/protegido em producao.

### Aplicacao

- [x] Validar acesso inicial ao frontend publico em HTTPS.
- [ ] Validar login/Home com usuario real por perfil.
- [ ] Validar usuarios, roles e permissoes reais por matriz.
- [ ] Validar clientes e colaboradores reais.
- [ ] Validar documentos com upload/download reais.
- [ ] Validar certificados digitais com storage real.
- [ ] Validar propostas e contratos.
- [ ] Validar assinatura publica.
- [x] Conectar pagina publica de documentos de onboarding aos aliases documentados da API v2.
- [ ] Validar solicitacoes de acesso.
- [ ] Validar notificacoes/Web Push se forem liberadas.
- [ ] Validar gestor/calendario/transferencias.
- [ ] Migrar ou bloquear corretamente mutacoes ainda legadas do Integra-AI.

### Integra-AI, Dominio e arquivos contabeis

- [x] Publicar Web/API atualizados com download TXT autenticado e leiaute Dominio vigente.
- [ ] Validar fluxo completo com usuario contabil real, permissao `accounting.integra_ai.download`, job real, MySQL real e storage real.
- [x] Importar TXT real no Dominio usando `Lancamentos Contabeis em Lote`.
- [ ] Salvar evidencia sanitizada da importacao Dominio.
- [ ] Congelar golden file aprovado do TXT Dominio.
- [ ] Validar parser com PDFs reais/sanitizados de bancos prioritarios.
- [ ] Manter `SAMA_INTEGRA_AI_OFX_IMPORT_ENABLED=false` em producao ate haver homologacao controlada.
- [ ] Se OFX for liberado, validar OFX real em homologacao com parser Python no container, ClamAV strict, storage privado e auditoria.
- [ ] Confirmar que mutacoes legadas do Integra-AI foram migradas para API v2 ou formalmente bloqueadas antes do desligamento do PHP.

### Acessorias, entregas e vencimentos

Trilha consolidada a partir de `integracao_acessorias_entregas_vencimentos.md`. O MVP amplo continua como fase futura; excecoes locais ja implementadas incluem a consulta read-only do Acessorias para alimentar a Home por perfil, a aplicacao segura em planilhas departamentais, a revisao manual de divergencias e a exibicao de vencimentos oficiais no workspace.

- [x] Registrar Acessorias/Entregas/Vencimentos como backlog futuro pos-migracao base.
- [x] Implementar consumo read-only do Acessorias para a Home por perfil via backend, usando variaveis do ambiente da API.
- [x] Implementar preview/importacao controlada de empresas e colaboradores do Acessorias para registros locais editaveis no Portal Sama.
- [ ] Validar em homologacao o contrato real da API do Acessorias consumido por `GET /api-v2/integrations/acessorias/home-summary`.
- [ ] Configurar/confirmar `ACESSORIAS_BASE_URL` e `ACESSORIAS_TOKEN` no EasyPanel sem expor segredos.
- [ ] Configurar/confirmar `ACESSORIAS_CLIENTS_PATH` e `ACESSORIAS_COLLABORATORS_PATH` no ambiente da API.
- [ ] Validar em homologacao o contrato real de empresas/colaboradores do Acessorias para `registrations/preview` e `registrations/sync`.
- [ ] Configurar/confirmar `ACESSORIAS_DELIVERIES_PATH` no ambiente da API.
- [ ] Confirmar dependencias antes de iniciar: Auth, RBAC, auditoria, notificacoes, planilhas departamentais, modulo base de integracoes, Prisma/MySQL, frontend React, logs estruturados, tratamento global de erros e testes minimos.
- [ ] Validar na API do Acessorias campos de baixa, vencimento, competencia, CNPJ/CPF, departamento, responsavel, ID unico e entregas canceladas/dispensadas.
- [x] Criar base backend de entregas em `integrations/acessorias`, com persistencia local inicial.
- [x] Implementar sincronizacao manual de entregas do Acessorias com auditoria.
- [x] Salvar entregas localmente e registrar sync runs.
- [x] Implementar sincronizacao automatica agendada opt-in de entregas do Acessorias.
- [x] Salvar divergencias iniciais da aplicacao Fiscal em `acessorias_fiscal_divergences`.
- [x] Criar mapeamento entre entregas do Acessorias e colunas da planilha Fiscal.
- [x] Aplicar baixas confirmadas do Acessorias na planilha Fiscal existente com status visual `ACESSORIAS`.
- [x] Conectar vencimentos oficiais Acessorias ao carrossel/workspace departamental local.
- [ ] Validar vencimentos Acessorias no workspace com dados reais no EasyPanel.
- [ ] Gerar planilha Fiscal por competencia a partir das entregas sincronizadas quando a planilha ainda nao existir.
- [x] Criar Central de Vencimentos local em `/departamentos/vencimentos`.
- [x] Consolidar vencimentos oficiais do Acessorias na Central de Vencimentos e manter vencimentos previstos como fallback/local.
- [x] Implementar revisao manual local de divergencias Acessorias.
- [ ] Validar revisao manual de divergencias no EasyPanel com dados reais e auditoria.
- [x] Implementar geracao manual local de alertas/notificacoes para divergencias, vencimento proximo, atraso e baixa Acessorias.
- [ ] Validar notificacoes Acessorias no EasyPanel com dados reais, auditoria e destinatarios reais.
- [x] Implementar scheduler seguro opt-in para sincronizacao periodica.
- [ ] Validar scheduler Acessorias no EasyPanel com dados reais, auditoria e sem notificacoes automaticas inicialmente.
- [ ] Habilitar notificacoes recorrentes D-7, D-3, D-1, D0 e D+1 somente apos validar destinatarios/deduplicacao.
- [ ] Implementar motor de recorrencia inteligente sem substituir vencimento oficial.
- [ ] Criar permissoes, perfis, auditoria e testes unitarios/integracao/E2E/seguranca do MVP Fiscal.

### UX/UI e aceite de uso

Trilha consolidada a partir de `ux-ui-docs/`. O checklist `ux-ui-docs/09-checklist-aceite.md` e o gate visual/experiencia antes de liberar uso amplo aos usuarios.

- [ ] Rodar e marcar o checklist `ux-ui-docs/09-checklist-aceite.md`.
- [ ] Validar sidebar desktop com hover/foco, agrupamento de rotas e ocultacao de itens sem permissao.
- [ ] Validar mobile com botao hamburguer, drawer e ausencia de quebra/overflow.
- [ ] Validar Home por perfil no EasyPanel: colaborador com pendencias reais do Acessorias, gestor com carteira/atrasos/riscos e DEV/Admin com diagnostico tecnico.
- [ ] Validar notificacoes no header: sino, badge, popover, pagina completa e filtros.
- [ ] Validar clientes/documentos: Meus Clientes por vinculo, Todos os Clientes por permissao ampla, documentos no painel do cliente, status, auditoria e links publicos expiraveis.
- [ ] Validar propostas, contratos, assinatura publica e entrada de cliente no fluxo completo.
- [ ] Validar `/configuracoes`: minha conta, foto com validacao segura, troca de senha restrita quando aplicavel e auditoria.
- [ ] Validar UX de seguranca: confirmacao em acoes sensiveis, upload com tipo/tamanho, SVG bloqueado para foto, erros sem detalhes internos e sem dados sensiveis em storage do navegador.
- [x] Implementar catalogo controlado de departamentos no backend com chave tecnica unica.
- [x] Trocar departamento livre por select no cadastro/edicao de usuarios DEV/Admin.
- [x] Trocar departamento livre por select no cadastro/edicao de colaboradores nas telas React principais.
- [ ] Aplicar migration/seed de departamentos no MySQL real e validar `/api-v2/departments` no EasyPanel.
- [ ] Expandir departamentos controlados para clientes/responsabilidades, transferencias, gestor/workspace e demais telas com departamento.
- [x] Implementar endpoint backend de transferencia normalizada de responsabilidades de clientes.
- [ ] Validar solicitacoes de acesso disponiveis a todos os usuarios autenticados, com aprovacao/execucao conforme perfil e escopo.
- [ ] Validar roles/permissoes com nome amigavel, descricao, agrupamento por modulo/acao, risco, formulario guiado e confirmacao para permissoes criticas.
- [ ] Validar auditoria para alteracoes em usuarios, roles, permissoes, departamentos e solicitacoes.
- [x] Conectar painel do cliente a Equipe/Responsaveis usando `client_department_assignments` localmente.
- [x] Implementar atribuicao inicial local de responsavel operacional por departamento, com gestor opcional.
- [x] Implementar transferencia auditada local de responsabilidade pelo painel do cliente.
- [x] Implementar edicao local de responsabilidade pelo painel do cliente.
- [x] Implementar encerramento local de responsabilidade pelo painel do cliente.
- [x] Conectar consulta local de carteira do gestor a `client_department_assignments` via dashboard de transferencias.
- [ ] Validar painel do cliente com responsabilidades reais no EasyPanel.
- [ ] Migrar escrita da transferencia operacional em lote para `client_department_assignments`.
- [ ] Evoluir responsabilidade de clientes para uso operacional completo da entidade propria cliente x departamento x usuario/gestor, sem depender exclusivamente de `clients.metadata`.
- [ ] Validar cadastro/painel de cliente com responsavel operacional e gestor por departamento, atribuicao inicial, gestor editavel, encerramento e transferencia de carteira auditada.

### Testes e QA

- [x] Rodar lint/build da API no repo separado.
- [x] Rodar lint/build do Web no repo separado.
- [x] Rodar E2E do Web no repo separado.
- [x] Disponibilizar smoke publico automatizado no repo separado do Web.
- [x] Implementar health operacional da API para banco/storage.
- [x] Rodar smoke publico sem `--soft`.
- [x] Rodar smoke local da pagina publica de documentos usando alias `/api-v2/public/onboarding/documents/:token`.
- [x] Disponibilizar smoke operacional de autenticacao real (`npm.cmd run smoke:auth`).
- [x] Disponibilizar smoke operacional de permissoes por perfil (`npm.cmd run smoke:permissions`).
- [x] Disponibilizar bateria consolidada de homologacao real (`npm.cmd run homologation:real`).
- [x] Rodar smoke/auth, matriz minima DEV/bootstrap e Playwright real contra HTTPS com usuario real.
- [x] Disponibilizar validadores operacionais da API para RBAC, storage, ClamAV, usuarios privilegiados e backfill read-only.
- [x] Disponibilizar gerador de backup operacional da API (`npm run ops:backup:create`).
- [x] Disponibilizar verificador de artefatos de backup da API (`npm run ops:backup:verify`).
- [x] Disponibilizar restore drill operacional da API (`npm run ops:restore:drill`).
- [x] Criar Playwright contra homologacao real (`npm.cmd run test:e2e:real`).
- [ ] Rodar Playwright contra homologacao real com todos os perfis criticos.
- [ ] Testar fluxos por perfil real.
- [ ] Fazer QA visual desktop.
- [ ] Fazer QA visual mobile.
- [ ] Registrar evidencias em `.ai-tests/` e resumos em docs.

## Bloqueios atuais principais

1. Atualizacoes tecnicas e documentais desta rodada precisam ser enviadas aos remotos.
2. Confirmar historico de migrations/seed/bootstrap no banco real e registrar o procedimento usado.
3. Rodar `npm run ops:backfill:report -- --json` no container `portal-sama-api` e usar o resultado para planejar/executar backfills reais por modulo.
4. Rodar `npm run ops:backup:create`, `npm run ops:backup:verify` e `npm run ops:restore:drill` no container `portal-sama-api`, copiar os artefatos para fora do container e documentar restore drill real.
5. Storage privado retornou `storage=up` e readiness real confirmou escrita em `/var/private/portal-sama`; aliases publicos de onboarding ja estao implementados, mas persistencia, permissoes, backup externo e fluxos reais de upload/download ainda precisam ser homologados.
6. O smoke operacional de autenticacao passou com usuario bootstrap/DEV; ainda precisa ser repetido por perfis criticos quando as credenciais forem disponibilizadas.
7. Smoke de permissoes passou na matriz minima bootstrap/DEV; usuarios/permissoes reais ainda precisam ser validados por matriz oficial de roles/perfis no EasyPanel.
8. Suite Playwright real passou com usuario bootstrap/DEV, mas precisa ser ampliada por perfis/fluxos e o QA visual final ainda nao foi concluido.
9. Backup operacional foi instrumentalizado com geracao, verificacao e restore drill protegido, mas ainda precisa ser executado no EasyPanel, armazenado fora do container e testado em alvo isolado real; retencao de logs segue pendente.
10. Dependencias funcionais remanescentes do legado ainda precisam ser desligadas com seguranca.
11. UX/UI ainda precisa passar pelo aceite de `ux-ui-docs/09-checklist-aceite.md`, incluindo mobile, navegacao por permissao, Home por perfil, documentos, configuracoes e seguranca visivel.
12. Departamentos, solicitacoes, roles/permissoes e responsabilidade de clientes ainda precisam ser fechados como governanca operacional: usuarios e colaboradores ja usam catalogo controlado localmente, e o backend/painel do cliente de responsabilidades normalizadas existe localmente com atribuicao, edicao, encerramento, transferencia e leitura de carteira do gestor, mas ainda faltam validacao real, escrita normalizada de transferencias em lote, solicitacoes globais, permissoes guiadas/auditadas e migracao operacional fora de `clients.metadata`.
13. A Home agora possui uma leitura inicial do Acessorias e a planilha possui aplicacao/revisao manual local de divergencias, vencimentos oficiais no carrossel, Central de Vencimentos, geracao manual de notificacoes e scheduler opt-in, mas ainda falta validar o contrato real da API externa no EasyPanel, revisar com dados reais/auditoria/destinatarios e decidir o corte funcional; notificacoes automaticas devem ser habilitadas apenas depois dessa validacao.

## Criterio para dizer "pronto para producao"

So marcar como pronto quando todos estes pontos forem verdadeiros:

- API e Web publicados no EasyPanel a partir do Bitbucket.
- Documentacao publicada e atualizada em `portal-sama-docs`.
- Banco real com migrations aplicadas e backup validado.
- Seed RBAC e bootstrap admin executados.
- Usuarios reais criados/migrados.
- Backfills conferidos.
- Storage privado validado.
- ClamAV strict validado.
- HTTPS, CORS, CSRF e cookies seguros funcionando.
- Fluxos criticos testados com dados reais.
- Playwright real passando.
- QA visual aprovado.
- Checklist UX/UI aceito para liberar uso amplo aos usuarios.
- Acessorias/Entregas/Vencimentos formalmente classificado como fase futura fora do corte base ou implementado e homologado.
- Plano de rollback documentado.
- Dependencias do legado removidas ou formalmente bloqueadas.

## Proximo passo recomendado

Com Bitbucket, EasyPanel, dominio HTTPS, banco real e smoke publico ja funcionando, o proximo passo operacional e transformar a homologacao tecnica em homologacao funcional:

```txt
registrar migrations/seed/bootstrap -> configurar/validar Acessorias no ambiente da API -> carregar credenciais no shell -> rodar homologation:real por perfis -> rodar ops:backfill:report --json -> rodar ops:backup:create -> rodar ops:backup:verify -> rodar ops:restore:drill em alvo isolado -> copiar artefatos/evidencias -> testar fluxos criticos reais -> validar uploads/downloads/storage -> homologar TXT Dominio -> aceitar UX/UI/Home por perfil -> manter MVP amplo de Acessorias como backlog futuro -> desligamento seguro do legado
```

Tambem e necessario enviar este commit/documentacao atualizada ao remoto de documentacao para manter Bitbucket e workspace local sincronizados.
