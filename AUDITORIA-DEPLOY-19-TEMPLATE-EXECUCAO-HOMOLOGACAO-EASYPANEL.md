# AUDITORIA DEPLOY 19 - Execucao da homologacao EasyPanel

Data:  
Branch API:  
Branch Web:  
Branch Docs:  
Ambiente local:  
Ambiente EasyPanel:  
Responsavel pela execucao:  

## 1. Objetivo

Registrar a execucao das correcoes e testes definidos em:

- `AUDITORIA-DEPLOY-17-HOMOLOGACAO-EASYPANEL-ACESSORIAS-POS-INTEGRACAO.md`
- `AUDITORIA-DEPLOY-18-PLANO-CORRECAO-HOMOLOGACAO-EASYPANEL.md`

## 2. Resumo executivo

| Frente | Status | Observacao |
|---|---|---|
| Home DEV/Gestor |  |  |
| Botao Integrar Acessorias |  |  |
| Conciliacao de responsaveis |  |  |
| Clientes / Ver meus clientes |  |  |
| Colaboradores publicos internos |  |  |
| Notificacoes |  |  |
| Preferencias/configuracoes |  |  |
| Solicitacoes de acesso |  |  |
| Playwright |  |  |
| Smoke EasyPanel |  |  |

## 3. Fase 0 - Preparacao e reproducao

Status:  

### Comandos executados

```powershell

```

### Resultado

| Comando | Resultado | Observacao |
|---|---|---|
|  |  |  |

### Bugs reproduzidos

| ID | Bug | Evidencia | Severidade |
|---|---|---|---|
| HEP-01 |  |  | Alta |
| HEP-02 |  |  | Critica |
| HEP-03 |  |  | Media/Alta |
| HEP-04 |  |  | Alta |
| HEP-05 |  |  | Media/Alta |
| HEP-06 |  |  | Critica |

## 4. Fase 1 - Botao Integrar

Status:  

### Arquivos alterados

```txt

```

### Mudancas aplicadas

- 

### Testes

| Teste | Resultado | Evidencia |
|---|---|---|
|  |  |  |

### Pendencias

- 

## 5. Fase 2 - Conciliacao de responsaveis

Status:  

### Arquivos alterados

```txt

```

### Regras implementadas

| Regra | Status |
|---|---|
| Nome + departamento + usuario ativo unico |  |
| Bloqueio de departamento diferente |  |
| Ambiguidade vira pendencia |  |
| Alias dentro do mesmo departamento |  |
| Usuario inativo nao concilia |  |

### Testes

| Teste | Resultado | Evidencia |
|---|---|---|
|  |  |  |

## 6. Fase 3 - Home DEV/Gestor

Status:  

### Evidencias de escopo

| Perfil | Resultado esperado | Resultado obtido |
|---|---|---|
| DEV | Global |  |
| Gestor | Departamento/responsabilidade |  |
| Colaborador | Responsabilidade direta |  |

## 7. Fase 4 - Clientes e colaboradores

Status:  

### Clientes

| Cenario | Resultado |
|---|---|
| DEV ve todos permitidos |  |
| Gestor `Ver meus clientes` |  |
| Colaborador `Ver meus clientes` |  |
| URL fora de escopo bloqueada |  |

### Colaboradores

| Cenario | Resultado |
|---|---|
| Usuario interno ve dados publicos |  |
| Dados sensiveis ausentes |  |
| Cliente externo bloqueado |  |

## 8. Fase 5 - Notificacoes e preferencias

Status:  

| Cenario | Resultado |
|---|---|
| Ler todas |  |
| Confirmar todos os alertas |  |
| Retencao maxima 100 |  |
| Preferencias persistidas apos reload |  |

## 9. Fase 6 - Solicitacoes de acesso

Status:  

| Fluxo | Resultado esperado | Resultado obtido |
|---|---|---|
| Colaborador cria | `PENDING_MANAGER` |  |
| Gestor aprova colaborador | `PENDING_DEV` |  |
| Gestor rejeita colaborador | `DECLINED` |  |
| Gestor solicita para si | `PENDING_DEV` |  |
| DEV aprova | `APPROVED` |  |
| DEV rejeita | `DECLINED` |  |
| Calendario multi-dia | OK |  |

## 10. Fase 7 - Testes locais

### API

| Comando | Resultado |
|---|---|
| `npm.cmd run prisma:validate` |  |
| `npm.cmd run lint` |  |
| `npm.cmd run build` |  |
| `npm.cmd test -- --runInBand` |  |
| `npm.cmd run test:e2e` |  |
| `node scripts/validate-operational-readiness.js --soft` |  |

### Web

| Comando | Resultado |
|---|---|
| `npm.cmd run lint` |  |
| `npm.cmd run build` |  |
| `npm.cmd test -- --runInBand` |  |
| `npm.cmd run test:e2e -- --reporter=line` |  |
| `npx.cmd playwright test tests/e2e/smoke.spec.ts --reporter=line` |  |
| `npx.cmd playwright test tests/e2e/access-requests.spec.ts --reporter=line` |  |

## 11. Fase 8 - Smoke EasyPanel

Status:  

| Cenario | Resultado | Evidencia sanitizada |
|---|---|---|
| Health |  |  |
| Login DEV |  |  |
| Home DEV |  |  |
| Dev Integrar/status local |  |  |
| Login Gestor |  |  |
| Home Gestor |  |  |
| Solicitacao teste |  |  |
| Notificacao teste |  |  |

## 12. Arquivos alterados

### portal-sama-api

```txt

```

### portal-sama-web

```txt

```

### portal-sama-docs

```txt

```

## 13. Evidencias geradas

```txt

```

## 14. Pendencias restantes

| Pendencia | Severidade | Bloqueia homologacao? | Proxima acao |
|---|---|---|---|
|  |  |  |  |

## 15. Veredito

Escolher uma opcao:

- `Pronto para homologacao controlada`
- `Nao pronto para usuarios reais`
- `Nao testavel por falha de ambiente`

Justificativa:

```txt

```
