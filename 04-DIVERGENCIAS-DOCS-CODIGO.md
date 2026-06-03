# 04 — Divergências entre documentação e código — Portal Sama

**Status:** fonte ativa  
**Data:** 2026-06-03

---

## 1. Tabela de divergências principais

| Tema | Documentação/conversa | Código atual | Impacto | Decisão para MVP |
|---|---|---|---|---|
| Escopo Acessórias | Documentos antigos dizem “não implementar agora”; atualizações indicam implementação parcial | Código já implementa Home, sync, preview, mapeamento, divergências e Central parcial | IA pode achar que deve parar ou recomeçar | Tratar como “implementado parcialmente, consolidar agora” |
| Central de Vencimentos | Deve ser única e listar pendentes, vencidas, entregues e canceladas | Backend filtra apenas `PENDING`, `DUE_SOON`, `OVERDUE`, `UNKNOWN` | Entregues/concluídas não aparecem | Adicionar filtros `status`, `source`, `include_completed` |
| `deliveries/ListAll` | Deve usar `DtLastDH` hoje/ontem quando `Identificador=ListAll` | Código pode usar último sync antigo como `DtLastDH` | Retorno incompleto ou erro externo | Separar incremental recente e backfill por empresa |
| Backfill Acessórias | Carga completa precisa consultar por empresa | Não há backfill robusto por `companies/ListAll -> deliveries/{Identificador}` | Primeira importação pode vir incompleta | Criar fluxo de backfill autorizado |
| Erro interno | Falha externa deve ser erro controlado | Alguns serviços não capturam `AbortError`/erro de rede | Usuário vê erro interno | Envolver `fetch` e retornar `503/504` sanitizado |
| Responsáveis | Usar responsáveis das obrigações para vincular colaborador | Código salva texto (`responsibleName`, `responsibleUsername`) | Não filtra corretamente por colaborador | Criar resolver + `responsibleUserId` + aliases |
| Colaboradores Acessórias | Não há endpoint oficial de colaboradores | Importação tem fallback por empresas; se path configurado pode criar usuários | Risco de fonte errada | Manter `ACESSORIAS_COLLABORATORS_PATH` vazio por padrão |
| MASTER | Usuário quer MASTER vendo painéis; doc recente fala DEV técnico total | Código privilegia `ADMIN`/`DEV`, não `MASTER` | Permissões inconsistentes | Decidir oficialmente: formalizar MASTER ou remover uso |
| Home loading | Deve mostrar carregamento claro | Cards exibem `...` | Parece bug/sem dados | Adicionar skeleton/loading textual |
| Frontend build | Deve compilar | `IntegraAiPage.tsx` falha por union type | Bloqueia deploy | Corrigir narrow de `IntegraAiExportResponse` |
| Web Push | Documentos anteriores tratavam como futuro; decisão atual inclui Web Push mínimo no MVP | Código possui indícios de base parcial (`Notification`, `BrowserPushSubscription`), mas fluxo transversal pode não estar consolidado | Sem push, operação pode perder alertas críticos fora da tela | Implementar notificações internas + Web Push mínimo, sem WhatsApp/Teams/recursos avançados |
| Recorrência inteligente | Documentada como futuro | Não precisa para MVP | Escopo cresce | Arquivar como futuro |
| Documentações | Muitas fontes ativas | IA pode se confundir | Retrabalho infinito | Criar pacote ativo, incluir contrato Web Push e mover histórico para `_arquivo` |

---

## 2. Divergência: Central de Vencimentos

### Problema

A documentação diz que a Central deve exibir vencimentos oficiais do Acessórias, previstos/manuais do Portal Sama, status de baixa, pendências, atrasos e histórico. O código atual restringe as entregas do Acessórias a status abertos.

### Correção

- Backend deve aceitar `status=all` e `include_completed=true`.
- Entregas `DELIVERED` e `CANCELED` devem entrar no retorno quando solicitado.
- A tela deve separar “status da obrigação” de “bloqueio por vencimento”.
- Entregue/cancelada não bloqueia célula.

---

## 3. Divergência: Acessórias ListAll x carga completa

### Problema

`deliveries/ListAll` não deve ser usado como carga completa permanente, porque `DtLastDH` em `ListAll` só aceita hoje ou ontem.

### Correção

- `ListAll` apenas para incremental recente.
- Backfill completo por empresa.
- Guard para impedir `DtLastDH` antigo.
- Registrar no sync run quando o sistema exigir backfill.

---

## 4. Divergência: responsáveis e colaboradores

### Problema

O usuário precisa que o DEV/gestor veja obrigações por colaborador. O código ainda não possui vínculo seguro entre entrega do Acessórias e usuário local.

### Correção

Adicionar:

```txt
responsibleUserId
responsibleMatchStatus
responsibleMatchScore
responsibleMatchReason
acessorias_responsible_aliases
```

Implementar resolver por:

```txt
e-mail -> username -> alias confirmado -> nome + departamento único
```

---

## 5. Divergência: documentação excessiva

### Problema

Documentações antigas e atuais competem. Isso dificulta o fechamento do MVP.

### Correção

- mover histórico para `_arquivo`;
- manter apenas documentos ativos do fechamento;
- criar ADRs curtos para decisões fixas;
- impedir que IA use documentos arquivados como requisito.

---

## 6. Divergência: papel MASTER

### Estado atual

O código trata privilégios principalmente por:

```txt
ADMIN
DEV
MANAGER
DEPARTMENT
```

`MASTER` aparece na conversa e em algumas intenções, mas não está consolidado no RBAC dos serviços críticos.

### Decisão necessária

Escolher uma das opções:

#### Opção A — Formalizar MASTER

`MASTER` entra como papel real:

```txt
PRIVILEGED_ROLES = ADMIN, DEV, MASTER
MANAGER_ROLES = ADMIN, DEV, MASTER, MANAGER
```

Com permissões explícitas.

#### Opção B — Não usar MASTER

Padronizar comunicação e código para usar:

```txt
DEV = técnico total
ADMIN = administração
MANAGER = gestão operacional
```

A opção escolhida deve ser aplicada no código, RBAC default e documentação.

Utilizaremos a Opção B e não vamos utilizar o MASTER

---

## 7. Divergência: Web Push agora entra no MVP

### Estado anterior

Os documentos anteriores colocavam Web Push como funcionalidade futura, para depois da integração Acessórias e da Central de Vencimentos.

### Decisão atual

Web Push mínimo entra no MVP porque o Portal Sama será usado como centro operacional da empresa. A plataforma precisa avisar sobre obrigações, acessos, contratos, certificados, documentos, erros e ações críticas sem depender de o usuário estar olhando a tela.

### Correção

Adicionar como requisito ativo:

```txt
NotificationEvent interno
Central de Notificações
sino/contador no layout
Browser/Web Push subscription por usuário autenticado
NotificationDispatcherService
registro de tentativas de entrega
preferências básicas
payload de push sanitizado
```

### Limite do escopo

A inclusão de Web Push **não autoriza**:

```txt
WhatsApp
Slack
Teams
SMS
resumo diário inteligente
agrupamentos avançados
regras complexas por horário
payload com dados sensíveis
```

### Eventos mínimos

```txt
ACESSORIAS_DELIVERY_COMPLETED
ACESSORIAS_DELIVERY_DUE_SOON
ACESSORIAS_DELIVERY_OVERDUE
ACESSORIAS_SYNC_FAILED
ACCESS_REQUEST_MANAGER_REVIEW
ACCESS_REQUEST_DEV_REVIEW
CONTRACT_SENT
CONTRACT_SIGNED
CERTIFICATE_EXPIRING
DOCUMENT_APPROVED
DOCUMENT_REJECTED
SYSTEM_ACTION_SUCCESS
SYSTEM_ACTION_FAILED
```
