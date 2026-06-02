# [PARCIAL] 10 - Departamentos, Solicitacoes e Permissoes

## 1. Objetivo

Esta documentacao corrige pontos de UX/UI, governanca e seguranca identificados nas telas atuais de usuarios, roles, permissoes, departamentos e solicitacoes de acesso.

O foco e evitar ambiguidade operacional, reduzir erros de cadastro e tornar o modelo de permissao mais compreensivel para usuarios MASTER, gestores e colaboradores.

Atualizacao 2026-06-02: a criacao inicial de responsabilidade no painel `/clientes/:id` ja usa departamento controlado via `departments.read` e colaborador ativo via `collaborators.read`, gravando em `client_department_assignments` por `POST /api-v2/clients/:clientId/assignments`. A mesma tela agora transfere responsabilidade ativa por `POST /api-v2/client-assignments/transfer`, exigindo `client_assignments.transfer` e `collaborators.read` na experiencia. A migracao completa dos filtros/carteiras ainda depende de backfill e validacao real.

---

## 2. Decisoes principais

```text
1. Departamento nao deve ser campo livre.
2. Departamento deve ser entidade controlada ou lista cadastrada por MASTER.
3. Solicitacoes de Acesso devem estar disponiveis para todos os usuarios autenticados.
4. Roles e permissoes podem continuar configuraveis por MASTER, mas a interface deve ser guiada.
5. A chave tecnica da permissao pode continuar existindo, mas o usuario deve ver nome amigavel, modulo, acao e descricao clara.
6. Criacao de permissao deve ser restrita, validada e preferencialmente feita por fluxo assistido.
```

---

## 3. Departamento nao deve ser texto livre

### 3.1 Problema atual

Na tela de cadastro/edicao de usuario, o campo `Departamento` aparece como input de texto livre.

Isso permite erros como:

```text
Fiscal
FISCAL
fiscal
Fiscal 
FiscaI
Contabil
Contabil
Contábil
```

Mesmo quando visualmente parecem iguais, tecnicamente podem virar departamentos diferentes. Isso prejudica:

- filtros por departamento;
- regras de permissao;
- exibicao da sidebar;
- escopo de dados;
- relatorios;
- auditoria;
- vinculacao de usuarios a clientes;
- futura integracao com Acessorias;
- futuras regras por departamento.

### 3.2 Decisao recomendada

Departamento deve ser selecionado em um componente controlado:

```text
Select / Combobox / Autocomplete controlado
```

O usuario comum, gestor ou admin operacional nao deve digitar livremente o nome do departamento.

### 3.3 Modelo recomendado

Criar uma entidade de departamentos.

Campos sugeridos:

```text
id
key
name
description
status
order_index
created_at
updated_at
```

Exemplo:

```text
key: FISCAL
name: Fiscal

key: CONTABIL
name: Contabil

key: PESSOAL
name: Pessoal

key: LEGALIZACAO
name: Legalizacao

key: FINANCEIRO
name: Financeiro

key: COMERCIAL
name: Comercial

key: TI
name: T.I.

key: ADMINISTRATIVO
name: Administrativo
```

### 3.4 Regra de unicidade

O backend deve garantir unicidade por `key` normalizada.

```text
UNIQUE(department.key)
```

A `key` deve ser normalizada sem depender do texto visual.

Exemplo:

```text
Entrada: Contabil
Key: CONTABIL

Entrada: Contabil
Key: CONTABIL

Resultado: mesmo departamento
```

### 3.5 UI recomendada no cadastro de usuario

Em vez de input livre:

```text
Departamento
[ Selecione um departamento v ]
```

Com opcoes vindas do backend.

Exemplo visual:

```text
┌────────────────────────────────────────────┐
│ Departamento                               │
│ [ Fiscal                               v ] │
└────────────────────────────────────────────┘
```

### 3.6 Criacao de departamento

A criacao de departamentos deve existir apenas em area administrativa:

```text
Admin > Configuracoes > Departamentos
```

Ou:

```text
Admin > Usuarios e Permissoes > Departamentos
```

Somente MASTER ou permissao equivalente deve criar/editar departamentos.

### 3.7 Campos da tela de departamentos

```text
Nome exibido
Chave tecnica
Descricao
Status
Ordem de exibicao
Permite usuarios?
Permite clientes?
Permite vencimentos?
Permite documentos?
```

### 3.8 Regras de seguranca

- nao permitir exclusao fisica se houver usuarios vinculados;
- usar arquivamento/inativacao;
- auditar criacao, edicao e inativacao;
- impedir duplicidade por normalizacao;
- impedir alteracao de `key` se ja houver vinculos criticos;
- manter historico de alteracoes.

---

## 4. Solicitacoes de Acesso devem ser globais

### 4.1 Decisao funcional

Solicitacoes de Acesso nao devem ser restritas apenas ao T.I. ou a MASTER.

Todos os usuarios autenticados devem conseguir acessar a funcionalidade para:

- pedir acesso a um sistema;
- pedir permissao adicional;
- pedir liberacao temporaria;
- reportar problema de acesso;
- enviar feedback tecnico ou operacional;
- solicitar ajuste relacionado ao proprio usuario, equipe ou departamento.

### 4.2 Diferenca entre solicitar e gerenciar

```text
Solicitar acesso = todos os usuarios autenticados.
Aprovar solicitacao = gestores, conforme escopo.
Executar/liberar acesso = T.I. ou MASTER.
Auditar solicitacoes = T.I., MASTER ou Auditor.
```

### 4.3 Estrutura recomendada

A funcionalidade pode aparecer como item acessivel na sidebar ou na Home para todos, mas com conteudo adaptado ao perfil.

```text
Solicitacoes
├── Minhas solicitacoes
├── Nova solicitacao
├── Aprovacoes pendentes
├── Solicitacoes da equipe
├── Execucao T.I.
└── Auditoria
```

### 4.4 Visibilidade por perfil

| Guia | Colaborador | Gestor | T.I. | MASTER | Auditor |
|---|---:|---:|---:|---:|---:|
| Minhas solicitacoes | Sim | Sim | Sim | Sim | Consulta |
| Nova solicitacao | Sim | Sim | Sim | Sim | Nao |
| Aprovacoes pendentes | Nao | Sim | Opcional | Sim | Consulta |
| Solicitacoes da equipe | Nao | Sim | Sim, se permitido | Sim | Consulta |
| Execucao T.I. | Nao | Nao | Sim | Sim | Consulta |
| Auditoria | Nao | Nao | Restrito | Sim | Sim |

### 4.5 Tipos de solicitacao

```text
ACCESS_REQUEST
PERMISSION_CHANGE
TEMPORARY_ACCESS
ACCESS_PROBLEM
SYSTEM_FEEDBACK
BUG_REPORT
IMPROVEMENT_SUGGESTION
OTHER
```

Na interface, exibir nomes amigaveis:

```text
Solicitar novo acesso
Alterar permissao
Acesso temporario
Problema de acesso
Feedback do sistema
Reportar erro
Sugestao de melhoria
Outro
```

### 4.6 Fluxo recomendado

```text
Usuario cria solicitacao
        ↓
Sistema identifica tipo e escopo
        ↓
Se exigir aprovacao, envia ao gestor
        ↓
Gestor aprova ou rejeita
        ↓
Se aprovado, envia ao T.I./MASTER
        ↓
T.I./MASTER executa ou responde
        ↓
Usuario recebe retorno
        ↓
Auditoria registra tudo
```

### 4.7 Feedback nao deve virar permissao automatica

Feedback e sugestoes podem usar o mesmo hub de solicitacoes, mas nao devem entrar automaticamente no fluxo de permissao.

Exemplo:

```text
Feedback do sistema -> triagem MASTER/T.I.
Solicitacao de acesso -> fluxo de aprovacao e execucao
```

---

## 5. Roles e permissoes: problema de UX

### 5.1 Problema atual

A tela atual mostra permissoes por chave tecnica, como:

```text
clients.read
clients.update
audit.read
calendar.manage
certificates.download
```

Esse modelo e bom para o backend, mas ruim como interface principal para usuario MASTER, porque:

- exige conhecimento tecnico;
- nao explica impacto operacional;
- mistura modulos diferentes sem agrupamento claro;
- dificulta revisar o que uma role realmente permite;
- aumenta risco de dar permissao indevida;
- torna a criacao/edicao de role cansativa;
- nao diferencia bem leitura, criacao, edicao, exclusao e acoes sensiveis.

### 5.2 Decisao recomendada

A chave tecnica deve continuar existindo, mas nao deve ser o texto principal da interface.

Cada permissao deve ter:

```text
Chave tecnica: clients.read
Nome amigavel: Visualizar clientes
Modulo: Clientes
Acao: Leitura
Descricao: Permite visualizar clientes dentro do escopo autorizado.
Risco: Baixo
Escopo: Proprio, departamento, todos
```

### 5.3 Exibicao recomendada

Em vez de mostrar somente:

```text
clients.read
```

Mostrar:

```text
Visualizar clientes
Permite consultar clientes dentro do escopo autorizado.
Chave: clients.read
Risco: Baixo
```

### 5.4 Agrupamento por modulo

As permissoes devem ser agrupadas por modulo:

```text
Clientes
Documentos
Certificados
Vencimentos
Propostas
Contratos
Entrada de Cliente
Solicitacoes
T.I.
Auditoria
Usuarios e Permissoes
Configuracoes
Integracoes
DEV
```

### 5.5 Agrupamento por acao

Dentro de cada modulo, organizar por tipo de acao:

```text
Leitura
Criacao
Atualizacao
Exclusao
Aprovacao
Download
Exportacao
Gerenciamento
Auditoria
Administracao sensivel
```

### 5.6 Niveis de risco

Adicionar classificacao visual:

```text
Baixo
Medio
Alto
Critico
```

Exemplos:

```text
clients.read = Baixo
clients.update = Medio
clients.delete = Alto
documents.download_sensitive = Alto
permissions.manage = Critico
dev.logs.read = Critico
```

Permissoes criticas devem exigir confirmacao extra ao salvar a role.

---

## 6. Tela recomendada de Roles

### 6.1 Lista de roles

A listagem atual de roles pode continuar, mas deve melhorar a leitura.

Adicionar:

- badge de tipo da role;
- badge de risco maximo;
- quantidade de usuarios;
- quantidade de permissoes criticas;
- ultima alteracao;
- botao para visualizar detalhes antes de editar.

Exemplo:

```text
┌────────────────────────────────────────────────────────────────────────────┐
│ Role        Tipo        Usuarios  Permissoes  Criticas  Risco  Atualizada │
├────────────────────────────────────────────────────────────────────────────┤
│ ADMIN       Sistema     0         69          12        Alto   26/05      │
│ MANAGER     Operacional 0         32          2         Medio  26/05      │
│ CLIENT      Externo     0         7           0         Baixo  26/05      │
└────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Criacao/edicao de role

A tela de edicao deve usar guias internas:

```text
Dados da role | Permissoes | Usuarios vinculados | Riscos | Auditoria
```

### 6.3 Guia Dados da role

Campos:

```text
Nome tecnico
Nome exibido
Descricao
Tipo
Status
Escopo padrao
```

Tipos sugeridos:

```text
SYSTEM
OPERATIONAL
DEPARTMENT
CLIENT
TECHNICAL
CUSTOM
```

### 6.4 Guia Permissoes

Layout recomendado:

```text
Buscar permissao: [________________]
Filtro de modulo: [Todos v]
Filtro de risco: [Todos v]
Filtro de acao: [Todos v]

[Clientes]
  [ ] Visualizar clientes          Baixo    clients.read
  [ ] Criar clientes               Medio    clients.create
  [ ] Editar clientes              Medio    clients.update
  [ ] Excluir clientes             Alto     clients.delete

[Documentos]
  [ ] Visualizar documentos        Medio    documents.read
  [ ] Baixar documentos            Alto     documents.download
  [ ] Gerar link publico           Critico  documents.public_link.create
```

### 6.5 Presets por perfil

Para facilitar configuracao, criar presets:

```text
Colaborador Fiscal
Colaborador Contabil
Colaborador Pessoal
Gestor de Departamento
T.I. Operacional
Auditor
Cliente Externo
MASTER
DEV
```

O MASTER pode usar um preset como base e ajustar permissoes.

---

## 7. Tela recomendada de Permissoes

### 7.1 Criacao de permissao nao deve ser livre demais

A criacao manual de permissao deve ser restrita porque uma permissao mal criada pode quebrar padrao de seguranca.

Em vez de apenas permitir digitar `modulo.acao`, usar formulario guiado:

```text
Modulo: [Clientes v]
Recurso: [Cliente v]
Acao: [Visualizar v]
Escopo: [Departamento v]
Risco: [Baixo v]
Descricao: [Permite visualizar clientes do escopo autorizado]
```

A chave tecnica pode ser gerada automaticamente:

```text
clients.read
```

### 7.2 Campo avancado

Permitir digitar a chave tecnica manualmente apenas em modo avancado:

```text
[ ] Modo avancado
```

Somente MASTER/DEV autorizado deve acessar esse modo.

### 7.3 Validacoes obrigatorias

- chave unica;
- formato padronizado;
- modulo existente;
- acao existente;
- descricao obrigatoria;
- risco obrigatorio;
- nao permitir permissao sem descricao;
- nao permitir chave com espaco, acento ou caracteres invalidos;
- auditar criacao e edicao.

### 7.4 Padrao de chave tecnica

Recomendado:

```text
module.resource.action
```

Ou, para simplicidade:

```text
module.action
```

Exemplos:

```text
clients.read
clients.create
clients.update
clients.delete
clients.export

documents.read
documents.upload
documents.download
documents.approve
documents.reject
documents.public_link.create

access_requests.read
access_requests.create
access_requests.approve
access_requests.reject
access_requests.execute
access_requests.audit

roles.read
roles.create
roles.update
roles.assign
roles.audit

permissions.read
permissions.create
permissions.update
permissions.manage
```

---

## 8. Escopo da permissao

### 8.1 Problema

Permissao sem escopo pode ser perigosa.

Exemplo:

```text
clients.read
```

Sozinha, essa permissao nao diz se o usuario pode ver:

- apenas clientes vinculados;
- clientes do departamento;
- todos os clientes;
- apenas clientes de uma unidade;
- clientes externos.

### 8.2 Decisao recomendada

Separar permissao de acao e escopo de dados.

```text
Permissao = o que pode fazer.
Escopo = sobre quais dados pode fazer.
```

Exemplo:

```text
Permissao: clients.read
Escopo: OWN_CLIENTS
```

Ou:

```text
Permissao: clients.read
Escopo: DEPARTMENT
```

### 8.3 Escopos sugeridos

```text
SELF
OWN_CLIENTS
TEAM
DEPARTMENT
ALL_COMPANY
EXTERNAL_CLIENT
SYSTEM
```

### 8.4 Aplicacao na UI

Na edicao de usuario ou role, exibir:

```text
Role: Colaborador Fiscal
Departamento: Fiscal
Escopo de clientes: Clientes vinculados ao usuario
Escopo de documentos: Clientes vinculados ao usuario
Escopo de vencimentos: Departamento Fiscal
```

---

## 9. Usuarios: formulario recomendado

### 9.1 Campos principais

```text
Nome
Email/login
Status
Departamento
Cargo/função
Roles
Escopo operacional
Gestor responsavel
```

### 9.2 Departamento

Deve ser select controlado.

```text
Departamento: [Fiscal v]
```

### 9.3 Roles

Deve permitir selecionar roles existentes com descricao clara.

```text
Roles
[ Colaborador Fiscal ]
Descricao: acesso operacional limitado ao departamento Fiscal.
```

### 9.4 Aviso de risco

Se o MASTER selecionar role critica:

```text
Atencao: esta role contem permissoes criticas.
Permissoes criticas: permissions.manage, users.update, audit.read.
[Confirmar atribuicao]
```

### 9.5 Auditoria

Auditar:

- alteracao de departamento;
- alteracao de role;
- ativacao/inativacao de usuario;
- alteracao de escopo;
- troca de senha por MASTER.

---

## 10. Nomenclatura recomendada

### 10.1 Roles atuais

| Role atual | Nome exibido recomendado | Observacao |
|---|---|---|
| ACCOUNTING | Contabil | Perfil operacional contabil |
| ADMIN | Administrador Operacional | Evitar confundir com MASTER absoluto |
| AUDITOR | Auditor | Consulta de trilhas e historicos |
| CLIENT | Cliente Externo | Acesso restrito ao proprio escopo |
| DEPARTMENT | Colaborador Departamental | Nome atual e generico demais |
| DEV | Desenvolvedor | Restrito a ambiente autorizado |
| LEGALIZATION | Legalizacao | Perfil do departamento de legalizacao |
| MANAGER | Gestor | Gestao de equipe/departamento |
| TI | T.I. | Suporte, acessos e operacao tecnica |

### 10.2 Permissoes

As permissoes devem ter titulo em portugues na UI.

Exemplos:

| Chave | Titulo amigavel | Descricao |
|---|---|---|
| clients.read | Visualizar clientes | Permite consultar clientes dentro do escopo autorizado. |
| clients.update | Editar clientes | Permite atualizar dados cadastrais permitidos. |
| documents.download | Baixar documentos | Permite baixar documentos autorizados. |
| access_requests.create | Criar solicitacao | Permite abrir solicitacoes de acesso, feedback ou suporte. |
| access_requests.approve | Aprovar solicitacoes | Permite aprovar solicitacoes dentro do escopo do gestor. |
| permissions.manage | Gerenciar permissoes | Permite alterar permissoes do sistema. Critico. |

---

## 11. Regras para o Codex implementar com seguranca

```text
1. Trocar campo livre de departamento por select carregado do backend.
2. Criar ou usar endpoint de departamentos controlados.
3. Impedir cadastro de departamento duplicado por normalizacao.
4. Manter Solicitacoes de Acesso visivel para todos os usuarios autenticados.
5. Controlar o conteudo de Solicitacoes por perfil e permissao.
6. Melhorar tela de Roles com agrupamento por modulo, acao e risco.
7. Exibir nome amigavel e descricao das permissoes.
8. Manter chave tecnica apenas como detalhe secundario.
9. Criar permissao por formulario guiado, nao por texto livre simples.
10. Auditar alteracoes de departamento, role e permissao.
11. Backend deve validar autorizacao mesmo que a guia ou botao esteja oculto no frontend.
```

---

## 12. Criterios de aceite

- [ ] Departamento no cadastro de usuario nao e campo livre.
- [ ] Departamento vem de lista controlada pelo backend.
- [ ] Departamento possui `key` tecnica unica.
- [ ] Criacao/edicao de departamento e restrita a MASTER/Admin autorizado.
- [ ] Solicitacoes de Acesso ficam disponiveis para todos os usuarios autenticados.
- [ ] Colaborador consegue criar solicitacao ou feedback.
- [ ] Gestor consegue aprovar solicitacoes do proprio escopo.
- [ ] T.I./MASTER consegue executar solicitacoes aprovadas.
- [ ] Roles exibem nome amigavel, descricao, risco e quantidade de permissoes criticas.
- [ ] Permissoes sao agrupadas por modulo.
- [ ] Permissoes exibem titulo descritivo em portugues.
- [ ] Chave tecnica fica visivel como informacao secundaria.
- [ ] Criacao de permissao usa formulario guiado.
- [ ] Permissoes criticas exigem confirmacao extra.
- [ ] Alteracoes em usuarios, roles, departamentos e permissoes geram auditoria.

---

## 13. Conclusao

A melhoria mais importante neste ponto e separar configuracao tecnica de experiencia operacional.

O sistema pode continuar tendo roles e permissoes tecnicas, mas a interface do MASTER precisa ser mais segura, explicativa e guiada.

A regra final recomendada e:

```text
Departamento deve ser dado controlado.
Permissao deve ser configuracao guiada.
Role deve ser pacote compreensivel de acesso.
Solicitacao deve ser canal aberto para todos os usuarios autenticados.
Acoes sensiveis devem ser restritas, confirmadas e auditadas.
```
