# AUDITORIA DEPLOY 17 - Homologacao EasyPanel Acessorias pos-integracao

Data: 11/06/2026  
Escopo: continuidade da homologacao apos aplicacao das documentacoes `AUDITORIA-DEPLOY-00` a `AUDITORIA-DEPLOY-16` e dos planos `00` a `07`, com foco no comportamento atual em EasyPanel, Home, Acessorias, responsaveis, clientes, colaboradores, notificacoes e solicitacoes de acesso.

## 1. Objetivo deste documento

Este documento transforma os problemas observados na homologacao atual em um roteiro tecnico para o Codex.

O objetivo nao e apenas implementar mudancas visuais. O Codex deve:

- identificar a causa tecnica dos problemas;
- reproduzir os comportamentos com testes;
- corrigir backend, frontend e contratos;
- validar a aplicacao como se estivesse em homologacao de producao;
- preservar a seguranca de documentos empresariais, dados de clientes e credenciais;
- registrar evidencias objetivas da correcao.

## 2. Estado de partida informado

O usuario informou que:

- as mudancas descritas em `AUDITORIA-DEPLOY-00` ate `AUDITORIA-DEPLOY-16` foram aplicadas;
- os planos `00` a `07` foram aplicados;
- a integracao Acessorias passou a funcionar parcialmente;
- a aplicacao agora puxa empresas do Acessorias e trata parte dos dados;
- ja e possivel homologar algumas frentes que antes estavam bloqueadas.

Porem, a homologacao visual e funcional revelou inconsistencias importantes nas seguintes areas:

| Area | Estado observado | Severidade inicial |
|---|---|---|
| Home DEV/Gestor | Cards e listas nao refletem corretamente o que existe no Acessorias/cache local. | Alta |
| Operacao DEV Acessorias | Tela tem muitos botoes, alguns consultam API externa de forma dispersa e dificultam o fluxo correto. | Media/Alta |
| Responsaveis Acessorias | Conciliacao nao esta confiavel; ha risco de vinculo com usuario de departamento errado. | Critica |
| Clientes | Apenas DEV enxerga corretamente todos os clientes importados; outros perfis nao conseguem validar o que deveriam. | Alta |
| Colaboradores | Usuarios conseguem ver apenas parte dos colaboradores; expectativa e visao publica interna mais ampla. | Media/Alta |
| Notificacoes | Faltam acoes em massa e limite operacional de retencao por usuario. Preferencias de perfil nao salvam. | Media/Alta |
| Solicitacoes de acesso | Fluxo de gestor e aprovacao DEV estao incorretos; calendario nativo e UX estao inadequados. | Critica |

## 3. Evidencias de entrada

As evidencias de homologacao desta rodada devem ser consultadas antes de qualquer alteracao:

```txt
evidencias/entrada/Homolacao-portal-sama-easypanel.txt
evidencias/screenshots/01-operacao-dev-acessorias-botoes.png
evidencias/screenshots/02-home-dev-diagnostico-acessorias.png
evidencias/screenshots/03-solicitacao-acesso-calendario-nativo.png
evidencias/screenshots/04-solicitacao-acesso-formulario-gestor.png
```

Leitura das imagens:

| Evidencia | Leitura tecnica |
|---|---|
| `01-operacao-dev-acessorias-botoes.png` | A tela DEV possui muitos botoes Acessorias, misturando chamadas externas, validacoes locais e operacoes tecnicas. Isso aumenta risco operacional e dificulta homologacao. |
| `02-home-dev-diagnostico-acessorias.png` | A Home DEV mostra dados de cache local, mas os numeros e itens exibidos precisam ser reconciliados contra entregas/clientes locais sincronizados e contra escopo por perfil. |
| `03-solicitacao-acesso-calendario-nativo.png` | O campo de data usa o calendario nativo do navegador, com UX inconsistente e sem selecao multipla direta por calendario. |
| `04-solicitacao-acesso-formulario-gestor.png` | O formulario de gestor exige colaborador e nao permite facilmente selecionar o proprio gestor para solicitar acesso. |

## 4. Fontes de verdade que o Codex deve ler antes de agir

Antes de editar codigo, ler no `portal-sama-docs`:

| Documento | Uso |
|---|---|
| `00-LEIA-ME-PARA-IA-MVP.md` | Regras do MVP, limites de escopo e seguranca. |
| `03-CONTRATO-ACESSORIAS-OPERACIONAL.md` | Contrato funcional da integracao Acessorias. |
| `07-PROMPT-CODEX-PIPELINE-ACESSORIAS-PERSISTENCIA-PUSH-DEV.md` | Pipeline, persistencia por pagina, fila, jobs e notificacoes DEV. |
| `AUDITORIA-DEPLOY-13-HOMOLOGACAO-LOCAL-ACESSORIAS-DOCKER.md` | O que ja foi provado localmente em MySQL Docker e Acessorias. |
| `AUDITORIA-DEPLOY-15-PREPARACAO-CORRECOES-GO-LIVE.md` | Estilo de plano por fases, criterios de aceite e comandos. |
| `AUDITORIA-DEPLOY-16-EXECUCAO-CORRECOES-GO-LIVE.md` | Registro incremental das correcoes ja aplicadas. |
| `CONTEXTO-CODEX-ATUAL.md` | Estado recente da branch e arquivos alterados. |

Regra: em caso de conflito, o codigo atual executavel e os documentos mais recentes de auditoria prevalecem sobre documentos arquivados.

## 5. Achados de homologacao

### HEP-01 - Home DEV e Home Gestor nao refletem corretamente o cache Acessorias

Severidade: Alta  
Areas provaveis:

- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.service.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.controller.ts`
- `portal-sama-web/src/pages/home/HomePage.tsx`
- `portal-sama-web/src/services/acessorias.service.ts`

Problema observado:

- A Home DEV e a Home Gestor exibem diagnosticos e itens que nao parecem bater com a sincronizacao real do Acessorias.
- A Home precisa usar a base local como fonte operacional, nao chamar API externa no carregamento normal.
- A visao DEV deve ser global.
- A visao Gestor deve ser por departamento/escopo/responsabilidade.

Risco:

- Usuario acredita que nao ha pendencias ou atrasos quando existem.
- Gestor pode nao ver obrigacoes do departamento.
- DEV pode homologar dados incorretos.

Criterio de aceite:

- `GET /integrations/acessorias/home-summary` deve retornar totais coerentes com `acessorias_deliveries` no banco local.
- DEV/ADMIN veem visao global.
- Gestor ve apenas escopo permitido.
- Colaborador ve apenas responsabilidades diretas e pendencias sem responsavel dentro do proprio departamento, se essa regra ja estiver ativa.
- Frontend nao envia `profile` para ampliar escopo.
- E2E Playwright valida Home DEV e Home Gestor com dados semanticos previsiveis.

### HEP-02 - Conciliacao de responsaveis Acessorias esta inconsistente

Severidade: Critica  
Areas provaveis:

- `portal-sama-api/src/modules/integrations/acessorias/acessorias-responsible-resolver.service.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-reconciliation.service.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-registrations.service.ts`
- `portal-sama-api/prisma/schema.prisma`
- `portal-sama-api/src/modules/clients/clients.service.ts`
- `portal-sama-api/src/modules/collaborators/collaborators.service.ts`

Problema observado:

- A conciliacao de responsaveis aparentemente nao esta associando corretamente responsaveis externos do Acessorias a usuarios internos.
- Ha indicio de associacao com usuarios de departamentos diferentes.
- Quando a conciliacao falha, filtros de clientes, planilhas por departamento e Home por perfil ficam vazios ou incorretos.

Regra de negocio obrigatoria:

```txt
responsavel externo Acessorias + departamento externo
  -> normalizacao rigorosa de nome e departamento
  -> usuario ativo local do mesmo departamento
  -> vinculo automatico somente se houver match unico e seguro
  -> caso ambiguo/inexistente fica pendente para revisao
```

Nao permitido:

```txt
responsavel externo -> usuario de outro departamento
responsavel externo ambiguo -> vinculo automatico
responsavel externo -> usuario ativo novo criado automaticamente
```

Criterio de aceite:

- Mesmo nome em departamento diferente nao concilia.
- Nome equivalente no mesmo departamento concilia quando o match e unico.
- Alias aprovado no mesmo departamento concilia.
- Alias de outro departamento nao concilia.
- Caso ambiguo gera pendencia local, nao vinculo automatico.
- A conciliacao atualiza os campos necessarios para que `ClientDepartmentAssignment`, entregas, Home e filtro "ver meus clientes" funcionem.

### HEP-03 - Operacao DEV Acessorias esta complexa demais e mistura chamadas externas com validacoes locais

Severidade: Media/Alta  
Areas provaveis:

- `portal-sama-web/src/pages/dev/DevAdminPage.tsx`
- `portal-sama-web/src/services/acessorias.service.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-pipeline.controller.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-registrations.controller.ts`
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-deliveries.controller.ts`

Problema observado:

- A tela DEV possui muitos botoes Acessorias.
- O operador nao sabe qual botao consulta a API externa, qual apenas valida localmente e qual altera dados.
- A expectativa atual e ter um unico botao externo: `Integrar`.

Decisao de UX/operacao:

```txt
Botao Integrar = unica acao manual da tela DEV que pode consultar API externa Acessorias.
Demais botoes = validacoes, reconciliacoes ou sincronizacoes locais a partir do banco local.
```

Fluxo recomendado para `Integrar`:

1. obter lock global da integracao;
2. validar configuracao Acessorias;
3. consultar API externa de forma paginada e resiliente;
4. persistir cada pagina no banco local;
5. tratar empresas/clientes, colaboradores externos, departamentos, responsaveis, catalogo, entregas e vencimentos;
6. executar conciliacao automatica segura;
7. registrar sync runs;
8. gerar notificacao tecnica para DEV em caso de sucesso, falha parcial ou falha critica;
9. retornar resumo sanitizado.

Botoes que podem permanecer como locais:

| Botao/acao | Tipo permitido |
|---|---|
| Status scheduler | Local |
| Status fila | Local |
| Sync runs | Local |
| Ver entregas locais | Local |
| Ver responsaveis pendentes | Local |
| Conciliar responsaveis | Local/manual |
| Reconciliar dados | Local |
| Reconciliar vencimentos | Local |
| Gerar notificacoes | Local |

Botoes que devem ser removidos, renomeados ou movidos para debug:

- `Previa entregas recentes`
- `Sincronizar incremental`
- `Previa empresas`
- `Importar empresas`
- `Carga cadastral`
- `Previa backfill`
- `Executar backfill`
- `Sincronizar operacao`

Observacao: se alguma dessas acoes ainda for necessaria para desenvolvedor, mover para uma secao `Diagnostico tecnico avancado`, recolhida por padrao e claramente marcada como operacao tecnica.

### HEP-04 - Clientes e colaboradores precisam de escopo funcional mais claro

Severidade: Alta  
Areas provaveis:

- `portal-sama-api/src/modules/clients/clients.service.ts`
- `portal-sama-api/src/modules/clients/dto/list-clients.dto.ts`
- `portal-sama-api/src/modules/collaborators/collaborators.service.ts`
- `portal-sama-api/src/modules/collaborators/dto/list-collaborators.dto.ts`
- `portal-sama-web/src/pages/clients/ClientsOverviewPage.tsx`
- `portal-sama-web/src/pages/clients/ClientsPage.tsx`
- `portal-sama-web/src/pages/collaborators/CollaboratorsOverviewPage.tsx`

Problema observado:

- DEV ve todos os clientes importados.
- Outros perfis nao conseguem validar clientes conforme esperado.
- Colaboradores ficam restritos demais para consulta publica interna.
- Existem telas/abas de clientes que podem estar duplicando o conceito de `Todos os clientes` e `Meus clientes`.

Decisao funcional:

- A pagina principal deve se chamar apenas `Clientes`.
- Dentro dela deve existir filtro `Ver meus clientes`.
- Com o filtro desligado, mostrar os clientes permitidos para o perfil do usuario.
- Com o filtro ligado, mostrar apenas clientes em que o usuario e responsavel/gestor do departamento, usando dados consolidados da conciliacao.
- A visualizacao de colaboradores deve permitir consulta publica interna segura de todos os colaboradores ativos da empresa, sem expor dados sensiveis.

Dados publicos recomendados para colaborador:

```txt
id, nome, username/login corporativo, departamento, cargo, ramal/email corporativo se ja for dado publico interno.
```

Dados que nao devem entrar na listagem publica:

```txt
hash de senha, metadata sensivel, permissoes detalhadas, tokens, telefone pessoal, dados de auditoria sensiveis, status interno desnecessario.
```

### HEP-05 - Notificacoes precisam de acoes em massa, retencao por quantidade e preferencias persistidas

Severidade: Media/Alta  
Areas provaveis:

- `portal-sama-api/src/modules/notifications/notifications.controller.ts`
- `portal-sama-api/src/modules/notifications/notifications.service.ts`
- `portal-sama-api/src/modules/notifications/dto/update-notification-preferences.dto.ts`
- `portal-sama-web/src/pages/notifications/NotificationsPage.tsx`
- `portal-sama-web/src/services/notifications.service.ts`
- `portal-sama-web/src/pages/settings/SettingsPage.tsx`

Problemas observados:

- Falta acao clara `Ler todas` na pagina e no icone/sino.
- Falta acao `Confirmar todos os alertas`.
- A retencao atual precisa limitar volume por usuario/escopo para evitar crescimento indefinido.
- Configuracoes de perfil/preferencias aparentam ser apenas estado local e nao salvam.

Decisao funcional:

- Maximo operacional: 100 notificacoes por usuario/escopo na listagem padrao.
- Maximo no banco: manter no maximo 100 notificacoes recentes por destinatario/escopo operacional, apagando as mais antigas que excederem o limite.
- Manter tambem expiracao por data quando `expiresAt` existir.
- `Ler todas` deve marcar `readAt`.
- `Confirmar todos os alertas` deve marcar `alertedAt`.
- Preferencias devem carregar do backend e salvar no backend, com feedback de sucesso/erro.

### HEP-06 - Solicitacoes de acesso estao com workflow incorreto e UX de calendario inadequada

Severidade: Critica  
Areas provaveis:

- `portal-sama-api/prisma/schema.prisma`
- nova migration Prisma para status de solicitacao
- `portal-sama-api/src/modules/access-requests/access-requests.service.ts`
- `portal-sama-api/src/modules/access-requests/access-requests.controller.ts`
- `portal-sama-api/src/modules/access-requests/access-requests.service.spec.ts`
- `portal-sama-web/src/pages/access-requests/AccessRequestPage.tsx`
- `portal-sama-web/src/schemas/access-request.schema.ts`
- `portal-sama-web/src/services/access-requests.service.ts`

Problemas observados:

- Gestor so consegue enviar solicitacao se selecionar colaborador.
- Gestor precisa poder solicitar acesso para si mesmo.
- Selecionar dias por calendario nativo esta ruim e nao reflete a identidade visual da Sama.
- Solicitacao de gestor/equipe aparece diretamente como aceita/aprovada.
- Falta etapa DEV para aprovar ou recusar solicitacao de gestor.
- As notificacoes de aprovacao/recusa devem chegar para quem solicitou e para quem precisa agir.

Workflow esperado:

```txt
Colaborador solicita acesso
  -> PENDING_MANAGER
  -> Gestor aprova
      -> PENDING_DEV
      -> DEV aprova ou recusa
          -> APPROVED ou DECLINED
  -> Gestor recusa
      -> DECLINED

Gestor solicita acesso para si mesmo ou equipe
  -> PENDING_DEV
  -> DEV aprova ou recusa
      -> APPROVED ou DECLINED
```

Observacao: se o negocio preferir nomear a etapa final como TI, usar `PENDING_DEV` internamente apenas se a operacao real for de DEV. Nao manter solicitacao de gestor como `APPROVED` antes da decisao DEV.

## 6. Regras de seguranca obrigatorias

1. Nenhuma correcao pode ampliar escopo de cliente/documento sem validacao backend.
2. Nenhuma regra de visibilidade pode existir apenas no frontend.
3. Conciliacao de responsaveis deve ser defensiva: duvida vira pendencia, nao vinculo automatico.
4. Logs e evidencias nao devem salvar payload bruto da API Acessorias, CNPJ completo, token, senha, JWT, CSRF, cookies ou dados empresariais sensiveis.
5. Testes de EasyPanel devem usar usuarios e dados de homologacao ou dados sanitizados.
6. Antes de qualquer teste destrutivo em EasyPanel, confirmar backup/rollback ou executar apenas em ambiente local com MySQL Docker.

## 7. Resultado esperado ao final da correcao

O Codex deve entregar:

- codigo corrigido na API e no Web;
- testes unitarios e E2E atualizados;
- evidencias de build/lint/testes;
- relatorio Markdown incremental, no estilo `AUDITORIA-DEPLOY-16`, registrando comandos, resultados e pendencias;
- screenshots ou traces Playwright sanitizados dos fluxos principais;
- checklist final indicando se a aplicacao esta pronta para homologacao controlada ou se ainda ha bloqueios.
