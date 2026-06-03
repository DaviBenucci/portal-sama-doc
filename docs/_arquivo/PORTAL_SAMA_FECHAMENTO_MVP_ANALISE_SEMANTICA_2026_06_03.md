# Portal Sama — Análise semântica e fechamento do MVP

**Data:** 2026-06-03  
**Escopo analisado:** documentação, backend e frontend atuais do Portal Sama  
**Objetivo:** consolidar o projeto, reduzir confusão documental e guiar a IA/desenvolvimento para fechar o MVP operacional.

---

## 1. Conclusão executiva

O Portal Sama já possui a base de um produto operacional amplo. A plataforma não deve mais ser tratada como uma ideia em documentação, mas como uma aplicação em consolidação.

O produto já cobre ou iniciou os principais fluxos necessários para centralizar o trabalho da empresa:

- obrigações e vencimentos;
- integração com Acessórias;
- clientes e carteiras;
- colaboradores;
- documentos empresariais;
- contratos e propostas;
- certificados digitais;
- entrada/onboarding de clientes;
- solicitações de acesso para T.I;
- extratos e Integra-AI;
- notificações;
- auditoria;
- permissões e papéis.

O problema principal não é ausência de código. O problema é **falta de fechamento de escopo** somada a **documentação excessiva e conflitante**.

A decisão recomendada é congelar o MVP e executar apenas as correções necessárias para a aplicação ficar estável, segura e utilizável pela operação e pela gestão.

---

## 2. Diagnóstico semântico do projeto

### 2.1 O projeto está amplo demais para continuar sendo guiado por documentos soltos

A documentação atual contém planos futuros, decisões antigas, análises parciais, escopos já implementados parcialmente e ideias de evolução. Isso faz uma IA interpretar tudo como pendência ativa.

Resultado provável:

```txt
mais documentação -> mais escopo -> mais alteração -> menos fechamento
```

A solução é criar um pacote de documentação ativa e arquivar o restante.

---

### 2.2 A integração Acessórias já saiu do “futuro”

Documentos antigos indicam que a integração Acessórias deveria ser futura. Porém, o código já possui Home, preview, sync de entregas, importação de empresas, mapeamentos, divergências e Central parcial.

A decisão correta para o MVP é:

```txt
Não recomeçar a integração.
Não tratá-la como ideia futura.
Consolidar a implementação parcial existente.
```

---

### 2.3 O maior erro técnico atual na integração Acessórias

O código usa `deliveries/ListAll` como parte central da busca de entregas. O contrato oficial do Acessórias permite `ListAll`, mas impõe uma restrição crítica: quando o identificador é `ListAll`, `DtLastDH` é obrigatório e aceita somente o dia atual ou o dia anterior.

Logo:

```txt
ListAll = incremental recente
ListAll != carga completa histórica
```

Se o Portal Sama usa o último sync bem-sucedido de vários dias atrás como `DtLastDH`, há risco de erro externo ou retorno incompleto.

A solução do MVP é separar:

```txt
incremental recente -> deliveries/ListAll
backfill completo -> companies/ListAll -> deliveries/{Identificador}
```

---

### 2.4 A Central de Vencimentos ainda não é a Central de Obrigações

A rota `/departamentos/vencimentos` existe, mas o backend ainda filtra apenas status abertos do Acessórias:

```txt
PENDING
DUE_SOON
OVERDUE
UNKNOWN
```

Isso exclui entregas concluídas e canceladas:

```txt
DELIVERED
CANCELED
```

Para a operação, isso é insuficiente. A gestão precisa ver:

- o que está pendente;
- o que venceu;
- o que foi entregue;
- o que foi cancelado/dispensado;
- qual competência foi concluída;
- qual colaborador estava responsável.

Portanto, a tela deve virar:

```txt
Central de Vencimentos e Obrigações
```

---

### 2.5 Responsáveis do Acessórias precisam virar vínculo operacional

O Acessórias não fornece, no contrato validado, um endpoint oficial de “todos os colaboradores”. Porém, as entregas e empresas trazem responsáveis.

O Portal Sama deve extrair esses responsáveis e conciliá-los com colaboradores locais.

Regra:

```txt
Nome externo não é usuário ativo.
Nome externo é candidato a vínculo.
```

Criar usuário ativo automaticamente seria risco de segurança.

A solução é criar:

```txt
AcessoriasResponsibleResolverService
acessorias_responsible_aliases
responsibleUserId em acessorias_deliveries
```

---

### 2.6 O papel MASTER precisa de decisão única

A conversa usa `MASTER` como perfil com visão ampla. O código, porém, privilegia principalmente `ADMIN` e `DEV` em pontos críticos.

Isso precisa ser:

- remover `MASTER` da semântica operacional e usar `DEV`/`ADMIN`/`MANAGER`.

---

### 2.7 O erro interno ao buscar informações do Acessórias

A causa mais provável é combinação de:

1. erro externo de rede/timeout não capturado corretamente em alguns serviços;
2. `DtLastDH` inválido/antigo em `deliveries/ListAll`;
3. variáveis de ambiente de produção divergentes;
4. dependência da Home em consulta externa direta em vez de leitura local resiliente.

A correção é tratar falhas externas como erro controlado e não como exceção inesperada.

---

## 3. Estado dos testes executados

### Backend

- Build backend executado com sucesso em base equivalente analisada.
- Testes direcionados de Acessórias/departamentos passaram:

```txt
5 test suites passed
27 tests passed
```

### Frontend

O build falhou por erro TypeScript em:

```txt
src/pages/accounting/IntegraAiPage.tsx
```

Erro:

```txt
Property 'error' does not exist on type 'IntegraAiExportResponse'
```

Esse erro deve ser corrigido antes de qualquer deploy.

---

## 4. Fechamento do MVP em uma frase

O MVP do Portal Sama deve entregar uma plataforma segura e centralizada para gestão operacional de clientes, obrigações, documentos, contratos, certificados, onboarding, acessos de T.I e extratos, com a integração Acessórias consolidada como fonte oficial de entregas/vencimentos e com a Central de Vencimentos como painel único da operação.

---

## 5. Pacote de documentação ativa gerado

Este pacote contém:

```txt
00-LEIA-ME-PARA-IA-MVP.md
01-ESTADO-ATUAL-CODIGO-DOCUMENTACAO.md
02-PLANO-FECHAMENTO-MVP.md
03-CONTRATO-ACESSORIAS-OPERACIONAL.md
04-DIVERGENCIAS-DOCS-CODIGO.md
05-PROMPT-CODEX-FECHAMENTO-MVP.md
```

Ele deve substituir a leitura indiscriminada de todas as documentações do projeto.

---

## 6. Próximos passos imediatos

1. Mover documentos antigos para `docs/_arquivo`.
2. Corrigir build do frontend.
3. Corrigir erro interno do Acessórias com tratamento de rede/timeout.
4. Separar incremental e backfill da integração Acessórias.
5. Atualizar Central para incluir entregues/cancelados.
6. Criar resolver de responsáveis.
7. Definir papel `MASTER`.
8. Executar build/testes.
9. Homologar no EasyPanel.

---

## 7. Critério final de pronto

O MVP estará pronto quando:

- API e Web compilarem;
- integração Acessórias não retornar erro interno genérico;
- Central mostrar todas as obrigações relevantes;
- gestor conseguir ver pendências por colaborador;
- colaborador conseguir ver suas obrigações;
- documentos, contratos, certificados, onboarding, acessos e Integra-AI estiverem navegáveis;
- permissões e auditoria estiverem aplicadas;
- segredos estiverem protegidos;
- documentação ativa estiver reduzida e alinhada ao código.
