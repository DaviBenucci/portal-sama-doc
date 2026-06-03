# 02 — Plano de fechamento do MVP — Portal Sama

**Status:** fonte ativa  
**Data:** 2026-06-03  
**Objetivo:** fechar um MVP operacional, estável e seguro para centralizar o fluxo de trabalho da empresa.

---

## 1. Definição do MVP

O MVP do Portal Sama deve permitir que a empresa use a plataforma como centro de operação para:

- controlar obrigações e vencimentos;
- acompanhar entregas do Acessórias;
- consultar pendências por cliente, colaborador e competência;
- gerenciar documentos empresariais;
- controlar contratos, propostas e assinatura/aceite;
- controlar certificados digitais;
- acompanhar entrada de clientes/onboarding;
- controlar solicitações de acessos do departamento de T.I;
- centralizar extratos e Integra-AI;
- dar visão para gestor e operação;
- manter trilha de auditoria e segurança.

O MVP não precisa conter todos os recursos futuros. Ele precisa funcionar bem no fluxo operacional principal.

---

## 2. Princípio de fechamento

Cada área deve terminar com:

```txt
1. Tela operacional clara
2. Endpoint estável
3. Permissão/RBAC definido
4. Auditoria quando houver ação crítica
5. Loading/erro vazio bem tratado
6. Teste mínimo
7. Documentação curta de uso/decisão
```

---

## 3. Fase M0 — Congelamento documental e foco da IA

### Objetivo

Evitar que a IA se confunda com excesso de documentação.

### Tarefas

- Criar pasta `docs/_arquivo/`.
- Mover análises antigas, planos futuros e propostas duplicadas para `_arquivo`.
- Manter ativos apenas os documentos do pacote de fechamento MVP.
- Adicionar aviso no topo de documentos arquivados:

```md
> Documento arquivado. Não usar como requisito ativo do MVP.
> Mantido apenas como histórico de decisão.
```

### Critério de aceite

- IA/Codex consegue iniciar tarefa lendo no máximo 5 arquivos ativos.
- Não há documento antigo competindo com o plano atual.

---

## 4. Fase M1 — Estabilização técnica imediata

### Objetivo

Garantir que o projeto compile e erros externos não virem `500` genérico.

### Tarefas prioritárias

1. Corrigir build do frontend em `IntegraAiPage.tsx`.
2. Corrigir tratamento de erro de rede/timeout nos serviços Acessórias.
3. Garantir que falha externa retorne erro controlado:

```txt
503 Acessórias indisponível
504 timeout externo
429 rate limit externo
```

4. Garantir diagnóstico sanitizado sem token ou stack trace.
5. Revisar variáveis de produção/EasyPanel.
6. Remover/rotacionar segredos encontrados em `.env` enviados em ZIP.

### Critério de aceite

- `npm run build` do frontend passa.
- Build do backend passa.
- Erro de rede do Acessórias não retorna erro interno genérico.
- Nenhum token aparece no frontend/logs.

---

## 5. Fase M2 — Aquisição correta de dados do Acessórias

### Objetivo

Separar carga completa de sincronização incremental.

### Decisão técnica

```txt
deliveries/ListAll = incremental recente
companies/ListAll + deliveries/{Identificador} = backfill completo por empresa
```

### Tarefas

1. Criar método `syncIncrementalFromListAll()`.
2. Criar método `backfillDeliveriesByCompany()`.
3. Impedir uso de `DtLastDH` antigo em `deliveries/ListAll`.
4. Quando não houver sync recente, rodar backfill por empresa ou exigir ação manual autorizada.
5. Usar `companies/ListAll` paginado até lista vazia.
6. Para cada empresa, chamar `deliveries/{CNPJ/CPF}` no período desejado.
7. Salvar/atualizar entregas por `externalId` estável.
8. Registrar sync run com totais e erro resumido.

### Critério de aceite

- Carga inicial não depende de `deliveries/ListAll` com `DtLastDH` antigo.
- Incremental usa somente hoje/ontem quando `Identificador=ListAll`.
- Backfill pode ser executado manualmente por perfil autorizado.
- API externa fora do ar não quebra operação local.

---

## 6. Fase M3 — Central única de Vencimentos e Obrigações

### Objetivo

Transformar `/departamentos/vencimentos` na página única para vencimentos internos e obrigações Acessórias.

### Backend

Atualizar `GET /api-v2/departments/workspace` para aceitar:

```txt
status=all|open|pending|overdue|delivered|canceled|today|next7|future
source=all|calendar|acessorias
include_completed=true|false
colaborador_id=<id>
month_key=YYYY-MM
dept_key=<departamento>
```

O retorno deve incluir nos itens:

```txt
competence
delivered_at
responsible_name
responsible_username
responsible_user_id
is_delivered
is_canceled
status_label
source
```

### Frontend

A tela deve exibir:

- Empresa;
- CNPJ;
- obrigação;
- competência;
- responsável;
- origem;
- vencimento;
- entregue em;
- situação;
- status Acessórias;
- status Portal Sama.

Filtros mínimos:

```txt
Todos
Pendentes
Vencidos
Entregues
Cancelados
Hoje
Próximos 7 dias
Futuros
Origem
Colaborador
Competência
Departamento
```

### Regra crítica

Entregas `DELIVERED` e `CANCELED` devem aparecer na Central, mas **não devem bloquear célula por vencimento**.

### Critério de aceite

- A Central mostra pendentes, vencidas, entregues e canceladas.
- A Central substitui a visão operacional separada de vencimentos.
- Configuração de calendário fica como configuração, não operação.
- Gestor consegue filtrar por colaborador.

---

## 7. Fase M4 — Conciliação de responsáveis Acessórias

### Objetivo

Vincular obrigações a colaboradores reais do Portal Sama.

### Regras

Resolver colaborador local por ordem de confiança:

```txt
1. e-mail exato
2. username/login exato
3. alias confirmado
4. nome normalizado + departamento, somente se único candidato
```

Se não houver match seguro:

```txt
criar alias pendente para revisão
não criar usuário ativo automaticamente
não alterar carteira oficial do cliente
```

### Banco recomendado

Adicionar em `AcessoriasDelivery`:

```txt
responsibleUserId
responsibleMatchStatus
responsibleMatchScore
responsibleMatchReason
```

Criar tabela:

```txt
acessorias_responsible_aliases
```

### Endpoints sugeridos

```txt
GET /api-v2/integrations/acessorias/responsibles
GET /api-v2/integrations/acessorias/responsibles/pending
PATCH /api-v2/integrations/acessorias/responsibles/:id/link-user
PATCH /api-v2/integrations/acessorias/responsibles/:id/ignore
```

### Critério de aceite

- Obrigações aparecem no painel do colaborador por `responsibleUserId`.
- Aliases ambíguos ficam pendentes.
- Nenhum usuário ativo é criado automaticamente.
- Vínculo confirmado gera auditoria.

---

## 8. Fase M5 — Painéis de operação e gestão

### Objetivo

Garantir que operação e gestão tenham visão clara.

### Painel do colaborador

Adicionar seção:

```txt
Obrigações e Competências
```

Com:

- total;
- pendentes;
- vencidas;
- entregues;
- canceladas;
- empresa;
- obrigação;
- competência;
- vencimento;
- status;
- origem.

### Painel do gestor

Deve permitir:

- ver carteira por colaborador;
- ver obrigações por colaborador;
- ver atrasos por departamento;
- ver entregas concluídas;
- revisar responsáveis pendentes;
- acessar divergências Acessórias.

### Papel MASTER

Definir uma decisão única:

- opção A: `MASTER` vira papel real de gestão/técnico e entra no RBAC;
- opção B: usar `DEV`/`ADMIN` como papel técnico total e não usar `MASTER` no código.

Não manter o estado atual ambíguo.

 Optaremos por utilizar a Opção B.

---

## 9. Fase M6 — Segurança, auditoria e homologação

### Objetivo

Fechar o MVP com segurança operacional.

### Checklist

- RBAC em endpoints críticos;
- CSRF em ações mutáveis;
- auditoria em sincronizações, importações, mapeamentos e vínculos;
- token Acessórias somente no backend;
- segredos fora de ZIP/commit;
- upload/documentos com controle de acesso;
- certificados digitais com criptografia e controle rígido;
- contratos/propostas com trilha de aceite;
- solicitações de acesso de T.I com responsável e histórico;
- tratamento global de erro sem stack trace em produção;
- logs sem dados sensíveis.

### Homologação

Executar no EasyPanel com dados reais:

1. preview Acessórias;
2. backfill por empresa;
3. incremental;
4. Central de Vencimentos;
5. painel colaborador;
6. mapeamento/divergências;
7. contratos/propostas;
8. certificados;
9. documentos;
10. onboarding;
11. solicitações T.I;
12. Integra-AI.

---

## 10. Entregáveis finais do MVP

O MVP só deve ser considerado fechado quando:

- backend builda;
- frontend builda;
- testes críticos passam;
- Acessórias não gera erro interno genérico;
- Central mostra todas as obrigações relevantes;
- gestor vê operação por colaborador;
- colaborador vê obrigações próprias;
- documentos/contratos/certificados/onboarding/acessos estão navegáveis;
- permissões e auditoria estão aplicadas;
- segredos estão protegidos;
- documentação ativa está reduzida e objetiva.
