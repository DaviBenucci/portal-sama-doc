# 03 — Contrato Acessórias operacional para o Portal Sama

**Status:** fonte ativa  
**Data:** 2026-06-03  
**Base:** documentação pública da API Acessórias em `https://api.acessorias.com/documentation`

---

## 1. Autenticação

A API do Acessórias usa:

```txt
Base URL: https://api.acessorias.com
Header: Authorization: Bearer <token>
Rate limit: 100 requisições por minuto
```

O token deve ficar somente no backend.

---

## 2. Variáveis de ambiente recomendadas

```env
ACESSORIAS_BASE_URL="https://api.acessorias.com"
ACESSORIAS_TOKEN="..."
ACESSORIAS_HOME_PATH="deliveries/ListAll"
ACESSORIAS_DELIVERIES_PATH="deliveries/ListAll"
ACESSORIAS_CLIENTS_PATH="companies/ListAll"
ACESSORIAS_COLLABORATORS_PATH=""
ACESSORIAS_AUTH_HEADER="Authorization"
ACESSORIAS_AUTH_SCHEME="Bearer"
ACESSORIAS_TIMEOUT_SEC="30"
ACESSORIAS_RATE_LIMIT_PER_MINUTE="100"
ACESSORIAS_MAX_PAGES="200"
```

`ACESSORIAS_COLLABORATORS_PATH` deve ficar vazio por padrão, pois não há endpoint oficial confiável de “todos os colaboradores” no contrato validado.

---

## 3. Empresas

Endpoint:

```txt
GET /companies/{Identificador}
```

Uso para listar todas:

```txt
GET /companies/ListAll?departments&Pagina=1
```

Características:

- paginação por `Pagina`;
- 20 registros por página;
- continuar até retornar lista vazia;
- `departments` retorna responsáveis por departamento:

```txt
Departamentos[].RespNome
Departamentos[].RespEmail
```

Uso no Portal Sama:

- cadastro/importação controlada de empresas;
- extração auxiliar de responsáveis;
- conciliação por CNPJ/CPF;
- nunca alterar Acessórias a partir do Portal Sama no MVP.

---

## 4. Entregas

Endpoint:

```txt
GET /deliveries/{Identificador}
```

Uso por empresa:

```txt
GET /deliveries/{CNPJ_OU_CPF}?DtInitial=YYYY-MM-DD&DtFinal=YYYY-MM-DD&situation=pending,delivered,read&config=true&Pagina=1
```

Uso ListAll:

```txt
GET /deliveries/ListAll?DtInitial=YYYY-MM-DD&DtFinal=YYYY-MM-DD&DtLastDH=YYYY-MM-DD HH:mm:ss&situation=pending,delivered,read&config=true&Pagina=1
```

Regra crítica:

```txt
Quando Identificador = ListAll, DtLastDH é obrigatório e só aceita o dia atual ou o dia anterior.
```

Portanto:

```txt
deliveries/ListAll não é backfill completo.
deliveries/ListAll é incremental recente.
```

---

## 5. Estratégia correta no Portal Sama

### 5.1 Incremental recente

Usar:

```txt
GET /deliveries/ListAll
```

com `DtLastDH` somente hoje/ontem.

### 5.2 Backfill/carga completa

Usar:

```txt
GET /companies/ListAll?departments&Pagina=N
        ↓
GET /deliveries/{IdentificadorEmpresa}?DtInitial=...&DtFinal=...&config=true&Pagina=N
```

Não depender de `DtLastDH` antigo para carga completa.

---

## 6. Campos importantes de entregas

Campos úteis no payload:

```txt
Identificador
Razao
Fantasia
Entregas[].Nome
Entregas[].EntCompetencia
Entregas[].EntDtPrazo
Entregas[].EntDtEntrega
Entregas[].Status
Entregas[].EntLastDH
Entregas[].Config.EntID
Entregas[].Config.ID
Entregas[].Config.DptoID
Entregas[].Config.DptoNome
Entregas[].Config.RespPrazo
Entregas[].Config.RespEntrega
```

---

## 7. Normalização de status

Mapeamento sugerido:

```txt
Entregue / delivered         -> DELIVERED
Atrasada / atrasado/overdue  -> OVERDUE
Pendente / pending           -> PENDING
Lida / read                  -> DUE_SOON ou READ conforme regra visual
Cancelada/dispensada         -> CANCELED
Indefinido                   -> UNKNOWN
```

A Central deve exibir `DELIVERED` e `CANCELED`, mas esses status não devem bloquear célula por vencimento.

---

## 8. Responsáveis

Fontes possíveis:

```txt
companies/ListAll + departments:
- Departamentos[].RespNome
- Departamentos[].RespEmail

deliveries:
- Config.RespPrazo
- Config.RespEntrega
- RespPrazo
- RespEntrega
```

Regra:

- usar essas fontes para conciliar colaboradores locais;
- não criar usuário ativo automaticamente;
- criar alias pendente quando não houver match seguro.

---

## 9. Tratamento de erro obrigatório

O backend deve tratar:

```txt
401/403 -> credencial/token inválido
429 -> rate limit externo
5xx -> Acessórias instável
AbortError -> timeout
TypeError/fetch failed -> rede/DNS/conexão
JSON inválido -> resposta externa inválida
204 -> retorno válido sem conteúdo, quando aplicável
```

O frontend não deve receber stack trace nem token.

---

## 10. Segurança

- Token somente no backend.
- Nunca salvar token no React/localStorage.
- Não logar header de autenticação.
- Não expor URL com credencial.
- Auditoria em sincronização/importação/vínculo.
- RBAC para preview, sync, backfill, mapeamento e revisão.

---

## 11. Eventos Acessórias que devem gerar notificação

A integração Acessórias deve emitir `NotificationEvent` interno para eventos operacionais relevantes. O Web Push é enviado somente quando o destinatário possuir dispositivo inscrito e preferência ativa.

Eventos mínimos:

```txt
ACESSORIAS_DELIVERY_COMPLETED
ACESSORIAS_DELIVERY_DUE_SOON
ACESSORIAS_DELIVERY_OVERDUE
ACESSORIAS_SYNC_FAILED
ACESSORIAS_SYNC_COMPLETED
ACESSORIAS_BACKFILL_COMPLETED
ACESSORIAS_BACKFILL_FAILED
ACESSORIAS_RESPONSIBLE_PENDING_REVIEW
ACESSORIAS_DIVERGENCE_CREATED
```

Destinatários sugeridos:

```txt
Colaborador responsável -> obrigação próxima, vencida ou entregue
Gestor do departamento -> atrasos, divergências e pendências críticas
DEV/ADMIN/T.I. -> falha crítica de sync, token inválido, rate limit persistente, erro de backfill
```

Payload de Web Push deve ser sanitizado.

Permitido:

```txt
Título: Obrigação próxima do vencimento
Mensagem: Você possui uma obrigação pendente com prazo próximo.
```

Proibido:

```txt
Título ou mensagem com CNPJ/CPF completo, certificado, contrato, documento, token, stack trace ou detalhe fiscal sensível.
```

Os detalhes completos devem ser carregados apenas dentro do Portal Sama, com autenticação e RBAC.
