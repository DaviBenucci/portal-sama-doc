# AUDITORIA DEPLOY 18 - Plano de correcao e testes da homologacao EasyPanel

Data: 11/06/2026  
Escopo: plano executavel para corrigir os achados `HEP-01` a `HEP-06` descritos em `AUDITORIA-DEPLOY-17-HOMOLOGACAO-EASYPANEL-ACESSORIAS-POS-INTEGRACAO.md`.

## 1. Objetivo desta etapa

Levar o Portal Sama para uma homologacao controlada real, validando que:

- Acessorias funciona como fonte externa de origem;
- Portal Sama opera a partir do cache local persistido;
- Home DEV/Gestor/Colaborador mostra dados corretos por escopo;
- conciliacao de responsaveis e departamentos e segura;
- clientes e colaboradores sao navegaveis pelos perfis esperados;
- notificacoes internas possuem operacao em massa, retencao e preferencias persistidas;
- solicitacoes de acesso passam por fluxo correto de Gestor e DEV;
- Playwright e testes de API conseguem reproduzir e proteger os fluxos corrigidos.

## 2. Regras de execucao

1. Trabalhar em branch unica ou fases pequenas, por exemplo `fix/easypanel-homologation-acessorias`.
2. Ler `AUDITORIA-DEPLOY-17` antes de alterar codigo.
3. Nao remover protecoes de RBAC/CSRF para fazer testes passarem.
4. Nao usar dados reais em snapshots, traces ou logs.
5. Primeiro validar localmente com banco isolado; depois validar EasyPanel com smoke controlado.
6. A cada fase, rodar testes focados antes de avancar.
7. Ao fim de cada fase, atualizar um relatorio incremental, por exemplo:

```txt
portal-sama-docs/AUDITORIA-DEPLOY-19-EXECUCAO-HOMOLOGACAO-EASYPANEL.md
```

## 3. Fase 0 - Preparacao e reproducao dos bugs

Status alvo: obrigatoria antes de corrigir.

### O que fazer

1. Abrir os tres workspaces:

```txt
portal-sama-docs
portal-sama-api
portal-sama-web
```

2. Ler os documentos:

```txt
00-LEIA-ME-PARA-IA-MVP.md
03-CONTRATO-ACESSORIAS-OPERACIONAL.md
07-PROMPT-CODEX-PIPELINE-ACESSORIAS-PERSISTENCIA-PUSH-DEV.md
AUDITORIA-DEPLOY-13-HOMOLOGACAO-LOCAL-ACESSORIAS-DOCKER.md
AUDITORIA-DEPLOY-16-EXECUCAO-CORRECOES-GO-LIVE.md
AUDITORIA-DEPLOY-17-HOMOLOGACAO-EASYPANEL-ACESSORIAS-POS-INTEGRACAO.md
CONTEXTO-CODEX-ATUAL.md
```

3. Mapear no codigo os endpoints e telas atuais:

| Frente | Arquivos para mapear |
|---|---|
| Home Acessorias | `acessorias-home.service.ts`, `HomePage.tsx`, `acessorias.service.ts` |
| Operacao DEV | `DevAdminPage.tsx`, `acessorias.service.ts`, controllers de Acessorias |
| Responsaveis | `acessorias-responsible-resolver.service.ts`, `acessorias-reconciliation.service.ts` |
| Clientes | `clients.service.ts`, `ListClientsDto`, `ClientsOverviewPage.tsx`, `ClientsPage.tsx` |
| Colaboradores | `collaborators.service.ts`, `ListCollaboratorsDto`, `CollaboratorsOverviewPage.tsx` |
| Notificacoes | `notifications.service.ts`, `NotificationsPage.tsx`, `SettingsPage.tsx` |
| Solicitacoes | `access-requests.service.ts`, `AccessRequestPage.tsx`, `access-request.schema.ts` |

4. Executar validacoes de base.

API:

```powershell
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
npm.cmd run test:e2e
```

Web:

```powershell
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
npm.cmd run test:e2e -- --reporter=line
```

5. Se houver falhas preexistentes, separar:

```txt
falha preexistente
falha causada por alteracao atual
falha de ambiente/configuracao
```

### Evidencias obrigatorias

- log sanitizado de build/lint/testes;
- screenshot atual da Home DEV;
- screenshot atual da Home Gestor;
- screenshot atual da tela DEV Acessorias;
- screenshot atual de solicitacao de acesso gestor;
- resposta sanitizada de `GET /integrations/acessorias/home-summary` para DEV e Gestor.

### Criterio de aceite

- Bugs reproduzidos ou, se nao reproduzidos, documentada a razao objetiva.
- Pontos de codigo confirmados antes de qualquer refatoracao.

## 4. Fase 1 - Consolidar botao unico `Integrar` no DEV

Status alvo: tela DEV mais segura e operacional.

### Decisao de produto

A tela DEV deve ter um unico botao manual que consulta a API externa Acessorias:

```txt
Integrar
```

Todos os outros botoes devem ser locais ou de diagnostico local.

### Backend recomendado

Criar ou consolidar um endpoint orquestrador:

```http
POST /integrations/acessorias/integrate
```

Permissao sugerida:

```txt
integrations.acessorias.sync
```

Servico sugerido:

```txt
AcessoriasIntegrationOrchestratorService
```

Fluxo interno:

1. validar CSRF;
2. validar permissao;
3. adquirir lock global;
4. registrar sync run `FULL_INTEGRATION`;
5. executar carga cadastral paginada;
6. executar persistencia de empresas/clientes;
7. executar normalizacao de departamentos;
8. executar conciliacao automatica segura de responsaveis;
9. executar sincronizacao de entregas/obrigacoes;
10. executar reconciliacao local de vencimentos/departamentos;
11. gerar resumo tecnico sanitizado;
12. notificar DEV em sucesso/falha parcial/falha total;
13. liberar lock.

Se reutilizar endpoints existentes em vez de criar endpoint novo, o frontend ainda deve chamar apenas uma funcao de servico para o botao `Integrar`, e essa funcao deve representar uma operacao atomica do ponto de vista do operador.

### Frontend recomendado

Arquivos:

```txt
portal-sama-web/src/pages/dev/DevAdminPage.tsx
portal-sama-web/src/services/acessorias.service.ts
portal-sama-web/src/types/acessorias.ts
```

Mudancas:

- substituir o bloco de muitos botoes por grupos claros:

```txt
Grupo 1 - Integracao externa
[Integrar]

Grupo 2 - Diagnostico local
[Status scheduler] [Status fila] [Sync runs] [Ver entregas locais]

Grupo 3 - Reconciliacao local
[Responsaveis pendentes] [Conciliar responsaveis] [Reconciliar vencimentos] [Reconciliar dados]

Grupo 4 - Operacao local
[Gerar notificacoes]
```

- remover ou recolher por padrao acoes de preview/backfill/importar/sincronizar separadas;
- cada botao deve mostrar claramente se e `Externo` ou `Local`;
- o botao `Integrar` deve ter confirmacao antes de iniciar;
- durante execucao, bloquear clique repetido e mostrar estado de progresso.

### Testes

API unitario/focado:

```powershell
npm.cmd test -- --runInBand src/modules/integrations/acessorias/acessorias-registrations.service.spec.ts src/modules/integrations/acessorias/acessorias-deliveries.service.spec.ts src/modules/integrations/acessorias/acessorias-rate-limiter.service.spec.ts
```

Adicionar testes para o orquestrador:

```txt
- executa etapas na ordem correta;
- persiste resultado parcial por etapa;
- falha externa nao apaga cache local;
- lock impede execucao concorrente;
- gera sync run e notificacao DEV;
- respeita dry-run se implementado.
```

Playwright:

```txt
- login DEV;
- abrir /dev;
- validar existencia de botao Integrar;
- validar que nao existem botoes externos dispersos na area principal;
- clicar Integrar em modo mock/local;
- validar feedback e resumo final;
- validar que status/sync runs/entregas locais nao fazem chamadas externas.
```

### Criterio de aceite

- Somente `Integrar` consulta a API externa em operacao manual.
- Demais acoes principais usam banco local.
- Codex registra no relatorio quais botoes foram removidos, renomeados ou movidos para diagnostico avancado.

## 5. Fase 2 - Corrigir conciliacao rigorosa de responsaveis e departamentos

Status alvo: dados de Home, clientes e planilhas passam a ter responsaveis confiaveis.

### Regra de matching

Implementar uma politica defensiva de matching:

| Dado | Regra |
|---|---|
| Nome | normalizar acentos, caixa, espacos duplicados, pontuacao irrelevante. |
| Departamento | normalizar para chave canonica do catalogo de departamentos. |
| Usuario | somente `ACTIVE`. |
| Match automatico | permitido apenas quando nome normalizado + departamento canonico retornam exatamente 1 usuario. |
| Alias aprovado | permitido somente dentro do mesmo departamento canonico. |
| Ambiguidade | manter pendente para revisao. |
| Departamento diferente | bloquear conciliacao automatica. |
| Usuario inexistente | manter pendente. |

### Campos que devem ser atualizados

O Codex deve confirmar no schema quais campos ja existem. A expectativa funcional e que, apos a conciliacao, fiquem coerentes:

```txt
acessorias_responsible_aliases
acessorias_deliveries.responsibleUserId
acessorias_deliveries.responsibleUsername
acessorias_deliveries.responsibleName
client_department_assignments.responsibleUserId
client_department_assignments.managerUserId, quando aplicavel
```

Se algum campo nao existir, propor migration minima e segura.

### Testes unitarios obrigatorios

Adicionar/ajustar:

```txt
src/modules/integrations/acessorias/acessorias-responsible-resolver.service.spec.ts
src/modules/integrations/acessorias/acessorias-reconciliation.service.spec.ts
src/modules/clients/clients.service.spec.ts
src/modules/integrations/acessorias/acessorias-home.service.spec.ts
```

Casos:

| Caso | Resultado esperado |
|---|---|
| Nome e departamento batem com 1 usuario ativo | vincula automaticamente. |
| Nome bate, departamento nao bate | nao vincula. |
| Departamento bate, nome nao bate | nao vincula. |
| Dois usuarios com mesmo nome no mesmo departamento | pendente/ambiguidade. |
| Usuario inativo | nao vincula. |
| Alias aprovado do mesmo departamento | vincula. |
| Alias aprovado de departamento diferente | bloqueia. |
| Responsavel conciliado aparece no filtro `ver meus clientes` | OK. |

### Criterio de aceite

- Nenhum usuario de outro departamento e associado automaticamente.
- Pendencias ficam visiveis para revisao manual.
- Home, clientes e entregas passam a usar o responsavel conciliado.

## 6. Fase 3 - Corrigir Home DEV/Gestor com base local Acessorias

Status alvo: Home se torna confiavel para operacao diaria.

### Backend

Validar/ajustar:

```txt
AcessoriasHomeService.getHomeSummary()
findLocalDeliveries()
buildDeliveryReadScope()
applyActorScope()
toSummary()
```

Regras:

| Perfil | Escopo esperado |
|---|---|
| DEV/ADMIN | global. |
| Gestor | departamentos permitidos + responsabilidade direta. |
| Colaborador | responsabilidade direta + itens sem responsavel do proprio departamento, se mantida essa regra. |
| Cliente externo | nao deve receber dados internos Acessorias, salvo regra explicita futura. |

### Frontend

Validar/ajustar:

```txt
HomePage.tsx
acessorias.service.ts
```

Regras de exibicao:

- mostrar `ultima sincronizacao`;
- mostrar `fonte: banco local`;
- exibir warning se dados estao stale;
- diferenciar `0 real` de `erro de carregamento`;
- quando nao houver responsavel conciliado, mostrar mensagem de acao clara para DEV/Gestor.

### Testes

Unitarios:

```powershell
npm.cmd test -- --runInBand src/modules/integrations/acessorias/acessorias-home.service.spec.ts
```

Adicionar casos:

```txt
- DEV ve todos os itens locais.
- Gestor nao ve item de outro departamento.
- Gestor ve item do proprio departamento.
- Colaborador ve item do qual e responsavel.
- Usuario sem escopo recebe totais zerados e nao vaza dados.
- Totais da Home batem com entregas filtradas no banco local.
```

Playwright:

```txt
- login DEV -> Home mostra diagnostico local e totais esperados.
- login Gestor -> Home mostra somente escopo esperado.
- alterar usuario pela URL ou query nao amplia escopo.
```

### Criterio de aceite

- Home DEV e Home Gestor batem com dados locais sincronizados.
- Nao ha dependencia da API externa durante renderizacao da Home.
- E2E captura screenshot da Home corrigida.

## 7. Fase 4 - Consolidar Clientes e Colaboradores

Status alvo: navegacao clara, sem duplicidade conceitual, com escopo seguro.

### Clientes

Backend:

Adicionar ao DTO, se ainda nao existir:

```txt
myClientsOnly?: boolean
mine?: boolean
scope?: 'all' | 'mine'
```

Regra:

- `scope=all` nao significa acesso global. Significa todos os clientes permitidos pelo backend para aquele usuario.
- `scope=mine` restringe para clientes onde o usuario e responsavel ou gestor do departamento.
- DEV/ADMIN com `scope=all` veem tudo.
- Perfis nao privilegiados continuam limitados por backend.

Frontend:

- manter uma pagina principal `Clientes`;
- adicionar toggle `Ver meus clientes`;
- remover duplicidade visual entre `Todos os clientes` e `Meus clientes` se existir;
- explicar no empty state que o filtro depende da conciliacao de responsaveis.

Testes:

```txt
- DEV scope all ve clientes importados.
- Gestor scope all ve clientes do departamento permitido.
- Gestor scope mine ve clientes onde e responsavel/gestor.
- Colaborador scope mine ve clientes onde e responsavel.
- Usuario nao privilegiado nao acessa cliente fora de escopo por URL.
```

### Colaboradores

Backend:

Criar ou ajustar modo de listagem publica interna:

```http
GET /collaborators?visibility=company_public
```

Ou endpoint dedicado:

```http
GET /collaborators/public
```

Permissao sugerida:

```txt
collaborators.read_public
```

Se preferir reutilizar `collaborators.read`, garantir que o retorno publico nao exponha dados sensiveis.

Campos permitidos:

```txt
id, username, name, mainDepartment, position, email corporativo se aplicavel, extension.
```

Campos proibidos:

```txt
passwordHash, metadata sensivel, roles detalhadas com permissoes, tokens, auditoria interna, dados pessoais nao necessarios.
```

Testes:

```txt
- usuario interno ativo ve colaboradores ativos publicos.
- usuario CLIENT nao ve listagem interna, salvo decisao explicita.
- dados sensiveis nao aparecem na resposta.
- filtro por departamento continua funcionando.
```

### Criterio de aceite

- Clientes possuem toggle `Ver meus clientes` funcional.
- Colaboradores possuem visao publica interna segura.
- Backend impede vazamento de clientes/documentos por URL.

## 8. Fase 5 - Corrigir notificacoes e preferencias de perfil

Status alvo: notificacoes operaveis e preferências persistidas.

### Backend

Ajustar `NotificationsService`:

1. limitar listagem padrao a 100;
2. `MAX_TAKE` deve ser 100, salvo decisao explicita;
3. manter retencao por data;
4. adicionar retencao por quantidade por destinatario/escopo;
5. criar acao em massa para alertas.

Endpoints esperados:

```http
PATCH /notifications/read-all
PATCH /notifications/alert-all
PATCH /notifications/preferences
GET /notifications/preferences
```

Observacao: ja existem rotas para `read-all`/`clear-unread`; validar frontend e contrato. Se `alert-all` nao existir, implementar.

Retencao por quantidade:

```txt
Depois de criar notificacao, manter no maximo 100 por chave operacional.
Chave sugerida: username quando houver; senao dept; senao global.
Excluir sempre as mais antigas.
Nao excluir notificacoes ainda necessarias para auditoria critica se houver regra de compliance futura.
```

### Preferencias em configuracoes

`SettingsPage.tsx` hoje deve ser revisada para garantir que:

- abas de notificacoes carreguem `GET /notifications/preferences`;
- toggles salvem por `PATCH /notifications/preferences`;
- visual preferences ou startPage tenham persistencia real, ou sejam removidas/rotuladas como futuras;
- todo salvar tenha feedback de sucesso/erro.

### Testes

Unitarios:

```txt
- readAll marca readAt em todas as notificacoes do escopo.
- alertAll marca alertedAt em todas as notificacoes do escopo.
- usuario nao altera notificacao fora do escopo.
- ao exceder 100 notificacoes, as mais antigas sao removidas.
- preferencias sao upsertadas e retornadas.
```

Playwright:

```txt
- abrir sininho;
- clicar Ler todas;
- validar contador zerado;
- clicar Confirmar todos os alertas;
- validar alertas zerados;
- abrir Configuracoes > Notificacoes;
- alterar preferencia;
- salvar;
- recarregar pagina;
- validar persistencia.
```

### Criterio de aceite

- Contadores do sininho e pagina convergem.
- Notificacoes nao crescem indefinidamente.
- Preferencias nao sao apenas estado local.

## 9. Fase 6 - Refazer workflow de solicitacoes de acesso

Status alvo: fluxo de aprovacao correto por Gestor e DEV.

### Backend

Schema/enum:

Adicionar status, se ainda nao existir:

```txt
PENDING_DEV
```

Fluxo esperado:

| Origem | Status inicial | Proxima decisao | Status final possivel |
|---|---|---|---|
| Colaborador | `PENDING_MANAGER` | Gestor | `PENDING_DEV` ou `DECLINED` |
| Gestor para si | `PENDING_DEV` | DEV | `APPROVED` ou `DECLINED` |
| Gestor para equipe | `PENDING_DEV` | DEV | `APPROVED` ou `DECLINED` |

Alterar service:

```txt
AccessRequestsService.create()
AccessRequestsService.decide()
AccessRequestsService.assertCanDecide()
AccessRequestsService.getManagerApprovals()
AccessRequestsService.list()
AccessRequestsService.toStatusLabel()
AccessRequestsService.emitCreatedNotifications()
AccessRequestsService.emitDecisionNotifications()
```

Criar decisoes separadas se necessario:

```http
POST /access-requests/:id/manager-approve
POST /access-requests/:id/manager-reject
POST /access-requests/:id/dev-approve
POST /access-requests/:id/dev-reject
```

Ou manter `/approve` e `/reject`, desde que o service derive corretamente a etapa pelo status atual e role do decisor.

Regra de decisao:

- Gestor decide apenas `PENDING_MANAGER` de colaborador do seu escopo.
- DEV/ADMIN decide `PENDING_DEV`.
- Gestor nao aprova final.
- Solicitacao de gestor nao nasce `APPROVED`.
- Solicitacao recusada gera notificacao ao solicitante.
- Solicitacao aprovada pelo gestor gera notificacao ao DEV.
- Decisao DEV gera notificacao ao solicitante e, quando aplicavel, aos colaboradores envolvidos.

### Gestor selecionando a si mesmo

Na solicitacao de gestor, permitir:

```txt
- selecionar colaboradores do departamento;
- selecionar o proprio gestor;
- enviar apenas para si mesmo;
- enviar para si mesmo + colaboradores;
- enviar para equipe sem si mesmo.
```

Backend deve aceitar self apenas quando:

```txt
requester possui role MANAGER
self username == requesterUsername
profile == MANAGER
```

Nao permitir que gestor selecione outro gestor fora do escopo ou usuarios de outro departamento.

### Frontend - calendario Sama multi-data

Substituir o uso principal de `input type=date` por componente controlado:

```txt
MultiDatePickerSama
```

Comportamento:

- abre um calendario estilizado;
- permite selecionar varios dias;
- clicar em dia selecionado remove selecao;
- mostra chips de dias selecionados;
- ordena datas;
- permite limpar selecao;
- respeita teclado e acessibilidade minima;
- usa tokens visuais da Sama: azul, laranja, bordas arredondadas, estados hover/focus.

### Testes unitarios de backend

Adicionar casos em:

```txt
src/modules/access-requests/access-requests.service.spec.ts
```

Casos obrigatorios:

| Caso | Resultado esperado |
|---|---|
| Colaborador cria solicitacao | `PENDING_MANAGER`. |
| Gestor aprova colaborador | `PENDING_DEV`, notifica DEV. |
| Gestor rejeita colaborador | `DECLINED`, notifica colaborador. |
| Gestor cria para si mesmo | `PENDING_DEV`. |
| Gestor cria para equipe | `PENDING_DEV`. |
| DEV aprova `PENDING_DEV` | `APPROVED`, notifica solicitante. |
| DEV rejeita `PENDING_DEV` | `DECLINED`, notifica solicitante. |
| Gestor tenta aprovar final | Forbidden/BadRequest. |
| Usuario de outro departamento selecionado | BadRequest. |
| Datas multiplas sao normalizadas e ordenadas | OK. |

### Playwright

Criar/ajustar:

```txt
portal-sama-web/tests/e2e/access-requests.spec.ts
```

Cenarios:

```txt
- gestor abre solicitacao, seleciona varios dias no calendario Sama, seleciona si mesmo e envia;
- status exibido e Pendente DEV, nao Aprovada;
- DEV abre solicitacoes, aprova pedido do gestor;
- gestor recebe notificacao de aprovacao;
- colaborador cria solicitacao;
- gestor aprova;
- DEV aprova;
- colaborador recebe notificacao;
- gestor rejeita colaborador;
- colaborador recebe notificacao de recusa.
```

### Criterio de aceite

- Nenhuma solicitacao nasce aprovada indevidamente.
- Workflow de duas etapas funciona.
- Calendario permite multiplos dias sem depender do picker nativo.
- Testes API e Playwright protegem regressao.

## 10. Fase 7 - Bateria de homologacao local com Playwright/API

Status alvo: provar fluxo completo local antes de EasyPanel.

### Preparar ambiente

Usar banco isolado, por exemplo:

```txt
portal_sama_homolog_easypanel_phase17
```

Desligar schedulers durante testes manuais:

```env
ACESSORIAS_SCHEDULER_ENABLED=false
ACESSORIAS_INCREMENTAL_SYNC_ENABLED=false
ACESSORIAS_COMPANY_CATALOG_SYNC_ENABLED=false
ACESSORIAS_RECONCILIATION_ENABLED=false
```

Se a integracao externa for testada, limitar paginas e registrar resultado sanitizado:

```env
ACESSORIAS_MAX_PAGES=1
```

### Scripts de teste recomendados

API:

```powershell
npm.cmd run prisma:validate
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
npm.cmd run test:e2e
node scripts/validate-operational-readiness.js --soft
```

Web:

```powershell
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
npm.cmd run test:e2e -- --reporter=line
```

Playwright real/local:

```powershell
npx.cmd playwright test tests/e2e/smoke.spec.ts --reporter=line
npx.cmd playwright test tests/e2e/access-requests.spec.ts --reporter=line
```

### Cenarios minimos E2E

| ID | Cenario | Perfil | Resultado esperado |
|---|---|---|---|
| E2E-01 | Login DEV e abrir `/dev` | DEV | Botao `Integrar` visivel e botoes externos dispersos ausentes/recolhidos. |
| E2E-02 | Rodar integracao em modo controlado | DEV | Resultado com sync run e sem erro de console/rede. |
| E2E-03 | Abrir Home DEV | DEV | Totais batem com banco local. |
| E2E-04 | Abrir Home Gestor | MANAGER | Dados limitados ao escopo. |
| E2E-05 | Clientes com `Ver meus clientes` | Colaborador/Gestor | Filtro reduz para responsabilidade direta. |
| E2E-06 | Colaboradores publicos | Interno | Lista dados publicos sem campos sensiveis. |
| E2E-07 | Notificacoes `Ler todas` | Interno | Contador de nao lidas zera. |
| E2E-08 | Notificacoes `Confirmar alertas` | Interno | Contador de alertas zera. |
| E2E-09 | Configuracoes notificacoes | Interno | Preferencia persiste apos reload. |
| E2E-10 | Gestor solicita para si | MANAGER | Status `PENDING_DEV`. |
| E2E-11 | DEV aprova solicitacao de gestor | DEV | Status `APPROVED` e notificacao ao gestor. |
| E2E-12 | Colaborador -> Gestor -> DEV | Colaborador/Gestor/DEV | Fluxo completo sem aprovar indevidamente. |

## 11. Fase 8 - Smoke controlado no EasyPanel

Status alvo: confirmar ambiente real sem operacao destrutiva indevida.

### Antes de testar EasyPanel

Verificar:

- backup recente;
- segredos rotacionados conforme auditorias anteriores;
- logs sem payload sensivel;
- usuario de teste DEV/Gestor/Colaborador;
- dados de teste ou clientes que possam ser usados sem risco;
- tempo da equipe para rollback se algo falhar.

### Smoke sem alterar dados sensiveis

1. `GET /health`.
2. Login DEV.
3. `GET /auth/me`.
4. Abrir Home DEV.
5. Abrir `/dev` e consultar apenas status locais.
6. Se for permitido, executar `Integrar` com limite controlado.
7. Abrir Sync runs.
8. Login Gestor.
9. Abrir Home Gestor.
10. Criar solicitacao de acesso de teste.
11. DEV aprovar/rejeitar solicitacao de teste.
12. Validar notificacao.

### Evidencias permitidas

Salvar apenas:

- status HTTP;
- nomes de telas;
- contadores agregados;
- screenshots sem CNPJ completo ou dados sensiveis;
- JSON sanitizado com campos sensiveis mascarados.

### Criterio de aceite

- Nenhum erro 500.
- Nenhum erro de console relevante.
- Nenhuma chamada CORS/CSRF falhando.
- Escopo por perfil preservado.
- Solicitações seguem workflow correto.
- Notificações em massa funcionam.

## 12. Checklist final Go/No-Go desta rodada

| Item | Status esperado |
|---|---|
| Build API | OK |
| Build Web | OK |
| Lint API | OK |
| Lint Web | OK |
| Unitarios API focados | OK |
| E2E API | OK ou falhas justificadas fora do escopo |
| Testes Web/contratos | OK |
| Playwright Home DEV/Gestor | OK |
| Playwright Solicitacoes | OK |
| Playwright Notificacoes | OK |
| Acessorias `Integrar` | OK em local e smoke controlado no EasyPanel |
| Responsaveis sem cross-department | OK |
| Clientes `Ver meus clientes` | OK |
| Colaboradores publicos sem dados sensiveis | OK |
| Retencao notificacoes max 100 | OK |
| Preferencias persistidas | OK |
| Solicitacao gestor nao nasce aprovada | OK |

## 13. Relatorio incremental esperado

Ao finalizar, criar novo documento no `portal-sama-docs`:

```txt
AUDITORIA-DEPLOY-19-EXECUCAO-HOMOLOGACAO-EASYPANEL.md
```

Estrutura minima:

```md
# AUDITORIA DEPLOY 19 - Execucao da homologacao EasyPanel

Data:
Branch:
Ambiente:

## Fase 0 - Preparacao
Status:
Comandos:
Resultado:
Evidencias:

## Fase 1 - Botao Integrar
Arquivos alterados:
Mudancas aplicadas:
Testes:
Pendencias:

...

## Veredito final
- Pronto para homologacao controlada
- Nao pronto para usuarios reais
- Bloqueios restantes
```

## 14. Commit sugerido

API:

```txt
fix(api): consolidar homologacao Acessorias e fluxo de solicitacoes
```

Web:

```txt
fix(web): ajustar homologacao Acessorias Home clientes notificacoes e acessos
```

Docs:

```txt
docs: adicionar plano de homologacao EasyPanel Acessorias
```
