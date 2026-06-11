# AUDITORIA DEPLOY 19 - Execucao da homologacao EasyPanel

Data: 2026-06-11
Branch API: `main`
Branch Web: `main`
Branch Docs: `fix/deploy-readiness-go-live`
Ambiente local: Windows / PowerShell
Ambiente EasyPanel: nao executado nesta rodada
Responsavel pela execucao: Codex

## 1. Objetivo

Registrar a execucao das correcoes e testes definidos em:

- `AUDITORIA-DEPLOY-17-HOMOLOGACAO-EASYPANEL-ACESSORIAS-POS-INTEGRACAO.md`
- `AUDITORIA-DEPLOY-18-PLANO-CORRECAO-HOMOLOGACAO-EASYPANEL.md`

## 2. Resumo executivo

| Frente | Status | Observacao |
|---|---|---|
| Home DEV/Gestor | Pendente | Nao alterada nesta rodada. Precisa validacao/correcao de escopo e numeros reais. |
| Botao Integrar Acessorias | Implementado local | Painel DEV passou a expor um unico botao externo/API: `Integrar`. |
| Conciliacao de responsaveis | Parcial implementado local | Bloqueio por divergencia de departamento aplicado a matches exatos e alias confirmados. |
| Clientes / Ver meus clientes | Pendente | Ainda nao corrigido/validado nesta rodada. |
| Colaboradores publicos internos | Pendente | Ainda nao corrigido/validado nesta rodada. |
| Notificacoes | Pendente | Ainda nao corrigido/validado nesta rodada. |
| Preferencias/configuracoes | Pendente | Ainda nao corrigido/validado nesta rodada. |
| Solicitacoes de acesso | Implementado local | Fluxo passou para dois estagios: gestor encaminha para DEV, DEV decide final. |
| Playwright | Pendente | Testes unitarios/contratos passaram; E2E web nao foi executado nesta rodada. |
| Smoke EasyPanel | Pendente | Nao executado nesta rodada. |

## 3. Fase 0 - Preparacao e reproducao

Status: concluida

### Comandos executados

```powershell
git status --short
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
```

### Resultado

| Comando | Resultado | Observacao |
|---|---|---|
| `git status --short` em `portal-sama-docs` | OK | Havia somente novos documentos/evidencias da homologacao 17-19. |
| `git status --short` em `portal-sama-api` | OK | Worktree limpo antes das alteracoes desta rodada. |
| `git status --short` em `portal-sama-web` | OK | Worktree limpo antes das alteracoes desta rodada. |
| `npm.cmd run lint` API | OK | Validacao inicial. |
| `npm.cmd run build` API | OK | Validacao inicial. |
| `npm.cmd test -- --runInBand` API | OK | 44 suites, 276 testes antes das novas alteracoes. |
| `npm.cmd run lint` Web | OK | Validacao inicial. |
| `npm.cmd run build` Web | OK | Validacao inicial. |
| `npm.cmd test -- --runInBand` Web | OK | 9 testes de contrato. |

### Bugs reproduzidos

| ID | Bug | Evidencia | Severidade |
|---|---|---|---|
| HEP-01 | Home DEV/Gestor com numeros/escopo a validar | `evidencias/screenshots/02-home-dev-diagnostico-acessorias.png` | Alta |
| HEP-02 | Painel DEV Acessorias expunha muitas acoes de integracao | `evidencias/screenshots/01-operacao-dev-acessorias-botoes.png` | Critica |
| HEP-03 | Conciliacao podia confirmar responsavel de departamento divergente | Plano AUDITORIA-DEPLOY-18 e codigo do resolver | Media/Alta |
| HEP-04 | Clientes e `Ver meus clientes` ainda precisam consolidacao de escopo | AUDITORIA-DEPLOY-18 | Alta |
| HEP-05 | Notificacoes/preferencias ainda precisam correcao de massa/retencao/persistencia | AUDITORIA-DEPLOY-18 | Media/Alta |
| HEP-06 | Solicitacoes de acesso precisavam calendario multi-dia, gestor solicitar para si e aprovacao DEV | `evidencias/screenshots/03-solicitacao-acesso-calendario-nativo.png`, `evidencias/screenshots/04-solicitacao-acesso-formulario-gestor.png` | Critica |

## 4. Fase 1 - Botao Integrar

Status: implementado local

### Arquivos alterados

```txt
portal-sama-web/src/pages/dev/DevAdminPage.tsx
```

### Mudancas aplicadas

- O painel DEV Acessorias manteve apenas um botao operacional de integracao externa/API: `Integrar`.
- O botao `Integrar` usa o fluxo ja existente de `sync-all`, que integra catalogo e entregas.
- Acoes separadas de preview/import/sync/backfill deixaram de ficar expostas na interface DEV.
- Permaneceram visiveis botoes de diagnostico/local/interno, como status, filas, sync runs, entregas locais, responsaveis pendentes e reconciliacoes.

### Testes

| Teste | Resultado | Evidencia |
|---|---|---|
| `npm.cmd run lint` Web | OK | Sem erros. |
| `npm.cmd run build` Web | OK | Build Vite/TypeScript concluido. |
| `npm.cmd test -- --runInBand` Web | OK | 9 testes de contrato. |

### Pendencias

- Executar Playwright visual/funcional para confirmar que somente o botao `Integrar` ficou exposto no browser.
- Executar smoke EasyPanel apos deploy.

## 5. Fase 2 - Conciliacao de responsaveis

Status: parcialmente implementado local

### Arquivos alterados

```txt
portal-sama-api/src/modules/integrations/acessorias/acessorias-responsible-resolver.service.ts
portal-sama-api/src/modules/integrations/acessorias/acessorias-responsible-resolver.service.spec.ts
```

### Regras implementadas

| Regra | Status |
|---|---|
| Nome + departamento + usuario ativo unico | Parcial; a rodada focou nos matches exatos por email/username e alias confirmados. |
| Bloqueio de departamento diferente | Implementado para match exato e alias confirmado quando o departamento da Acessorias existe. |
| Ambiguidade vira pendencia | Mantido pelas regras existentes; nao expandido nesta rodada. |
| Alias dentro do mesmo departamento | Implementado para impedir alias confirmado fora do departamento fonte. |
| Usuario inativo nao concilia | Mantido pelas regras existentes; nao alterado nesta rodada. |

### Testes

| Teste | Resultado | Evidencia |
|---|---|---|
| `npm.cmd test -- --runInBand src/modules/access-requests/access-requests.service.spec.ts src/modules/integrations/acessorias/acessorias-responsible-resolver.service.spec.ts` | OK | 2 suites, 10 testes. |
| `npm.cmd test -- --runInBand` API | OK | 44 suites, 278 testes. |

## 6. Fase 3 - Home DEV/Gestor

Status: pendente

### Evidencias de escopo

| Perfil | Resultado esperado | Resultado obtido |
|---|---|---|
| DEV | Global | Nao validado/corrigido nesta rodada. |
| Gestor | Departamento/responsabilidade | Nao validado/corrigido nesta rodada. |
| Colaborador | Responsabilidade direta | Nao validado/corrigido nesta rodada. |

## 7. Fase 4 - Clientes e colaboradores

Status: pendente

### Clientes

| Cenario | Resultado |
|---|---|
| DEV ve todos permitidos | Nao validado/corrigido nesta rodada. |
| Gestor `Ver meus clientes` | Nao validado/corrigido nesta rodada. |
| Colaborador `Ver meus clientes` | Nao validado/corrigido nesta rodada. |
| URL fora de escopo bloqueada | Nao validado/corrigido nesta rodada. |

### Colaboradores

| Cenario | Resultado |
|---|---|
| Usuario interno ve dados publicos | Nao validado/corrigido nesta rodada. |
| Dados sensiveis ausentes | Nao validado/corrigido nesta rodada. |
| Cliente externo bloqueado | Nao validado/corrigido nesta rodada. |

## 8. Fase 5 - Notificacoes e preferencias

Status: pendente

| Cenario | Resultado |
|---|---|
| Ler todas | Nao validado/corrigido nesta rodada. |
| Confirmar todos os alertas | Nao validado/corrigido nesta rodada. |
| Retencao maxima 100 | Nao validado/corrigido nesta rodada. |
| Preferencias persistidas apos reload | Nao validado/corrigido nesta rodada. |

## 9. Fase 6 - Solicitacoes de acesso

Status: implementado local

| Fluxo | Resultado esperado | Resultado obtido |
|---|---|---|
| Colaborador cria | `PENDING_MANAGER` | Mantido. |
| Gestor aprova colaborador | `PENDING_DEV` | Implementado e coberto por teste. |
| Gestor rejeita colaborador | `DECLINED` | Mantido com escopo de decisao restrito ao gestor responsavel. |
| Gestor solicita para si | `PENDING_DEV` | Implementado: API inicia pedido de gestor como pendente DEV e web permite selecionar o proprio gestor. |
| DEV aprova | `APPROVED` | Implementado e coberto por teste. |
| DEV rejeita | `DECLINED` | Implementado pela mesma regra de decisao privilegiada; sem teste dedicado novo. |
| Calendario multi-dia | OK | Implementado no frontend sem calendario nativo; pendente Playwright visual. |

## 10. Fase 7 - Testes locais

### API

| Comando | Resultado |
|---|---|
| `npm.cmd run prisma:validate` | OK; schema valido. O Prisma carregou `.env`, sem imprimir valores. |
| `npx.cmd prisma generate` | OK; client gerado apos novo enum. O Prisma carregou `.env`, sem imprimir valores. |
| `npm.cmd run lint` | OK |
| `npm.cmd run build` | OK |
| `npm.cmd test -- --runInBand` | OK; 44 suites, 278 testes. |
| `npm.cmd run test:e2e` | Nao executado nesta rodada. |
| `node scripts/validate-operational-readiness.js --soft` | Nao executado nesta rodada. |

### Web

| Comando | Resultado |
|---|---|
| `npm.cmd run lint` | OK |
| `npm.cmd run build` | OK |
| `npm.cmd test -- --runInBand` | OK; 9 testes de contrato. |
| `npm.cmd run test:e2e -- --reporter=line` | Nao executado nesta rodada. |
| `npx.cmd playwright test tests/e2e/smoke.spec.ts --reporter=line` | Nao executado nesta rodada. |
| `npx.cmd playwright test tests/e2e/access-requests.spec.ts --reporter=line` | Nao executado nesta rodada. |

Checagens adicionais:

```txt
portal-sama-api: git diff --check OK
portal-sama-web: git diff --check OK
```

## 11. Fase 8 - Smoke EasyPanel

Status: pendente

| Cenario | Resultado | Evidencia sanitizada |
|---|---|---|
| Health | Nao executado nesta rodada. | - |
| Login DEV | Nao executado nesta rodada. | - |
| Home DEV | Nao executado nesta rodada. | - |
| Dev Integrar/status local | Nao executado nesta rodada. | - |
| Login Gestor | Nao executado nesta rodada. | - |
| Home Gestor | Nao executado nesta rodada. | - |
| Solicitacao teste | Nao executado nesta rodada. | - |
| Notificacao teste | Nao executado nesta rodada. | - |

## 12. Arquivos alterados

### portal-sama-api

```txt
prisma/schema.prisma
prisma/migrations/20260611130000_add_access_request_pending_dev/migration.sql
src/modules/access-requests/access-requests.service.ts
src/modules/access-requests/access-requests.service.spec.ts
src/modules/integrations/acessorias/acessorias-responsible-resolver.service.ts
src/modules/integrations/acessorias/acessorias-responsible-resolver.service.spec.ts
```

### portal-sama-web

```txt
src/pages/access-requests/AccessRequestPage.tsx
src/pages/dev/DevAdminPage.tsx
src/types/access-requests.ts
```

### portal-sama-docs

```txt
AUDITORIA-DEPLOY-19-EXECUCAO-HOMOLOGACAO-EASYPANEL.md
CONTEXTO-CODEX-ATUAL.md
```

## 13. Evidencias geradas

```txt
Nenhuma evidencia externa nova foi gerada nesta rodada.
Foram usadas as evidencias fornecidas em evidencias/entrada/ e evidencias/screenshots/.
Os resultados de validacao local estao registrados neste documento.
```

## 14. Pendencias restantes

| Pendencia | Severidade | Bloqueia homologacao? | Proxima acao |
|---|---|---|---|
| Aplicar migration `20260611130000_add_access_request_pending_dev` no banco alvo | Alta | Sim | Rodar `prisma migrate deploy` no ambiente controlado antes de testar fluxo novo. |
| Executar API E2E apos migration | Alta | Sim | Rodar `npm.cmd run test:e2e` em ambiente local/controlado. |
| Executar Playwright Web, especialmente solicitacoes de acesso e painel DEV | Alta | Sim | Rodar smoke/access-requests com browser e revisar screenshot. |
| Smoke EasyPanel apos deploy | Critica | Sim | Validar health, login, home, Integrar, solicitacao e notificacoes no dominio real. |
| Home DEV/Gestor | Alta | Sim | Corrigir/validar escopo e numeros por perfil. |
| Clientes / Ver meus clientes / colaboradores publicos internos | Alta | Sim | Implementar fase 4 do plano. |
| Notificacoes e preferencias | Media/Alta | Sim | Implementar fase 5 do plano. |
| Conciliacao por nome + departamento | Media/Alta | Parcial | Revisar se a regra nominal precisa ficar explicita alem dos matches exatos/alias. |

## 15. Veredito

Escolher uma opcao:

- `Pronto para homologacao controlada`
- `Nao pronto para usuarios reais`
- `Nao testavel por falha de ambiente`

Veredito selecionado: `Nao pronto para usuarios reais`

Justificativa:

```txt
A primeira leva de correcoes esta implementada e verde em lint/build/test locais.
Ainda faltam migration aplicada em ambiente alvo, E2E API, Playwright Web,
smoke EasyPanel e as fases pendentes de Home, Clientes/Colaboradores,
Notificacoes/Preferencias.
```
