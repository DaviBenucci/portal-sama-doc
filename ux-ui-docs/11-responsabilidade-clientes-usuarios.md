# Responsabilidade de clientes por usuario — diagnostico do codigo atual e melhoria proposta

## 1. Pergunta analisada

Como o Portal Sama define que um cliente esta sob responsabilidade de um usuario, seja colaborador ou gestor?

## 2. Conclusao tecnica

No codigo atual, a responsabilidade de um cliente **nao esta modelada como entidade propria no banco**.

A responsabilidade e inferida principalmente a partir de campos flexiveis em `clients.metadata`, especialmente por departamento.

Modelo pratico atual:

```txt
Cliente
  metadata.departamentos[]
    departamento
    responsavelId
```

Tambem existem formatos alternativos aceitos pelo codigo:

```txt
metadata.depts[]
metadata.carteira[]
metadata.portfolio[]
metadata.responsaveis[]
metadata.responsibles[]
metadata.assignments[]
metadata.departmentAssignments[]
metadata.dept_Fiscal = { responsavelId }
metadata.responsavelFiscal
metadata.responsavel_fiscal
metadata.fiscalResponsibleId
```

Isso significa que o sistema atual tenta ser compativel com varios formatos legados, mas nao possui ainda uma regra unica, forte e auditavel para atribuir clientes a colaboradores ou gestores.

---

## 3. Onde isso aparece no backend

### 3.1 Modelo `Client`

No Prisma, o model `Client` nao possui campos como:

```txt
responsibleUserId
managerUserId
departmentAssignments
```

O cliente possui apenas:

```txt
metadata Json?
```

Portanto, a carteira/responsabilidade nao esta normalizada em tabela relacional.

### 3.2 Departamentos / Fiscal workspace

Arquivo analisado:

```txt
src/modules/departments/departments.service.ts
```

A planilha Fiscal monta as empresas por meio de:

```txt
buildFiscalCompanies()
extractClientDepartments()
```

A funcao `extractClientDepartments()` le `client.metadata` e procura responsaveis em estruturas como:

```txt
departamentos
departments
depts
carteira
portfolio
responsaveis
responsibles
assignments
departmentAssignments
dept_Fiscal
responsavelFiscal
```

Depois, para o Fiscal, a empresa so entra no workspace se houver relacao com o departamento Fiscal:

```txt
Fiscal + responsavelId
```

A tela seleciona um colaborador e filtra empresas assim:

```txt
companies.filter(company => company.responsibleId === selectedCollaboratorId)
```

Conclusao: para a planilha Fiscal, a responsabilidade e resolvida por `metadata` do cliente, nao por tabela propria.

### 3.3 Transferencias

Arquivo analisado:

```txt
src/modules/transfers/transfers.service.ts
```

Este e o modulo que mais se aproxima de uma regra real de carteira.

Ele carrega usuarios e clientes, monta runtime interno e identifica clientes por responsavel usando:

```txt
getClientResponsible(client, department)
```

Quando uma transferencia e aplicada, ele chama:

```txt
setClientResponsible(runtime, clientId, department, responsibleId)
```

Essa funcao atualiza:

```txt
client.metadata.departamentos
client.metadata.dept_<Departamento>
user.metadata.clientes
```

Ou seja, a transferencia muda a responsabilidade do cliente, mas ainda usando JSON em `metadata`.

### 3.4 Gestor / historico

Arquivo analisado:

```txt
src/modules/managers/managers.service.ts
```

O modulo de gestor tambem le a responsabilidade a partir de `client.metadata`.

Ele monta relacoes assim:

```txt
companyRelations(client, department)
```

E retorna dados como:

```txt
responsavel_id
responsavel_nome
responsavel_username
```

Porem, o modulo de gestor nao possui um endpoint claro para atribuir cliente a usuario. Ele usa a responsabilidade existente para listar, filtrar e permitir historico operacional.

---

## 4. Onde isso aparece no frontend

### 4.1 Clientes do departamento

Arquivo analisado:

```txt
src/pages/departments/DepartmentClientsPage.tsx
```

A tela tenta descobrir responsavel lendo `client.metadata` no frontend.

Ela procura campos como:

```txt
responsibleName
responsavelNome
responsavel_nome
responsible
responsavel
username
```

E tambem procura dentro de:

```txt
departamentos
departments
carteira
portfolio
responsaveis
responsibles
assignments
departmentAssignments
```

Isso mostra que a tela esta fazendo leitura defensiva de varios formatos, mas nao existe uma estrutura unica garantida.

### 4.2 Cadastro de cliente

Arquivos analisados:

```txt
src/pages/clients/ClientsPage.tsx
src/pages/dev/DevNewClientPage.tsx
src/services/clients.service.ts
src/schemas/client.schema.ts
```

No formulario atual de cliente, nao existe campo para selecionar responsavel, carteira ou gestor.

O payload enviado pelo frontend tambem nao inclui `metadata`.

Campos enviados hoje:

```txt
razaoSocial
nomeFantasia
cnpj
email
status
rank
regimeTributario
idEmpresa
atividade
telefone
endereco
cidade
estado
grupo
```

Conclusao: pelo fluxo visual atual de cadastro de cliente, o usuario nao consegue atribuir o cliente a um colaborador ou gestor.

### 4.3 Carteira do colaborador

Arquivo analisado:

```txt
src/pages/collaborators/CollaboratorViewPage.tsx
```

A tela possui uma secao chamada `Carteira de clientes`, mas no momento ela exibe estado vazio:

```txt
Nenhum cliente vinculado foi retornado para este colaborador.
```

Ou seja, a UI ja sugere que havera carteira por colaborador, mas a integracao dessa carteira ainda nao esta concluida nessa tela.

---

## 5. Como a responsabilidade e atribuida hoje, na pratica

Pelo codigo atual, existem tres caminhos possiveis:

### 5.1 Por metadata preexistente no cliente

Um cliente pode vir com `metadata` contendo algo como:

```json
{
  "departamentos": [
    {
      "departamento": "Fiscal",
      "responsavelId": "user-id-ou-colaborador-id"
    }
  ]
}
```

Ou:

```json
{
  "dept_Fiscal": {
    "responsavelId": "user-id-ou-colaborador-id"
  }
}
```

Esse parece ser o formato mais usado pela logica atual.

### 5.2 Por transferencia de carteira

O modulo de transferencias consegue alterar responsabilidade, mas ele exige que o cliente ja esteja sob responsabilidade de um colaborador de origem.

Fluxo atual:

```txt
Cliente ja tem responsavel A
        ↓
Gestor cria transferencia para responsavel B
        ↓
Sistema valida origem
        ↓
Sistema atualiza metadata do cliente
        ↓
Sistema atualiza metadata dos usuarios envolvidos
```

Limite importante: esse fluxo nao resolve bem o primeiro vinculo de um cliente sem responsavel.

### 5.3 Por atualizacao direta de metadata via API

O backend aceita `metadata` no DTO de cliente, mas o frontend atual nao envia esse campo.

Tecnicamente, alguem com permissao `clients.update` poderia chamar a API diretamente com `metadata`, mas isso nao e uma experiencia segura, padronizada ou recomendada.

---

## 6. Pontos problematicos encontrados

### 6.1 Nao existe tabela propria de responsabilidade

Nao ha uma tabela como:

```txt
client_user_assignments
client_responsibilities
client_department_responsibles
```

Isso dificulta:

- auditoria clara;
- historico de troca de responsavel;
- validade por periodo;
- filtros performaticos;
- integridade referencial;
- controle por departamento;
- relatorios de carteira;
- bloqueio de duplicidade.

### 6.2 Responsabilidade esta escondida em JSON

Como a responsabilidade fica em `metadata`, o banco nao garante:

- se o usuario existe;
- se o departamento existe;
- se o responsavel pertence ao departamento;
- se existe mais de um responsavel ativo para o mesmo cliente/departamento;
- se o valor usa `user.id`, `username` ou `colaboradorId`.

### 6.3 Departamento ainda e string livre em varios pontos

O sistema usa normalizacao em codigo, mas isso nao substitui uma entidade controlada.

Exemplos de risco:

```txt
Contabil
Contábil
contabil
CONTABIL
Fiscal 
```

### 6.4 Primeiro vinculo do cliente nao esta claro

O modulo de transferencia funciona melhor para trocar responsavel, nao para definir a primeira responsabilidade.

Hoje nao foi encontrado um fluxo claro como:

```txt
Selecionar cliente sem responsavel
Selecionar departamento
Selecionar colaborador
Salvar vinculo inicial
```

### 6.5 Gestor e colaborador nao estao claramente separados na responsabilidade

O codigo trabalha muito com `responsavelId` por departamento, mas nao diferencia bem:

```txt
responsavel operacional
responsavel gestor
responsavel substituto
auditor
```

### 6.6 Frontend exibe carteira, mas nao carrega carteira no detalhe do colaborador

A tela de colaborador ja tem secao de carteira, mas ainda nao retorna clientes vinculados.

---

## 7. Recomendacao de modelagem correta

Criar uma entidade propria para responsabilidade do cliente por departamento.

### 7.1 Tabela sugerida

```txt
client_department_assignments
```

Campos:

```txt
id
client_id
department_id
responsible_user_id
manager_user_id nullable
assignment_type
status
start_at
end_at nullable
created_by
updated_by
created_at
updated_at
```

### 7.2 Campos explicados

```txt
client_id
- cliente vinculado.

department_id
- departamento controlado, nao string livre.

responsible_user_id
- colaborador responsavel pela operacao.

manager_user_id
- gestor responsavel pela supervisao, quando necessario.

assignment_type
- PRIMARY, BACKUP, TEMPORARY, MANAGER_REVIEW.

status
- ACTIVE, INACTIVE, TRANSFERRED, PENDING_APPROVAL.

start_at / end_at
- permite historico e transferencias temporarias.
```

### 7.3 Restricao recomendada

Para evitar dois responsaveis principais ativos no mesmo departamento:

```txt
UNIQUE(client_id, department_id, assignment_type, status)
```

Ou, em regra de aplicacao:

```txt
Um cliente nao pode ter dois responsaveis PRIMARY ativos no mesmo departamento.
```

---

## 8. Fluxo de UI recomendado

### 8.1 No cadastro de cliente

Adicionar bloco:

```txt
Responsabilidades por departamento
```

Exemplo visual:

```txt
┌──────────────────────────────────────────────────────────────┐
│ Responsabilidades                                             │
├──────────────────────────────────────────────────────────────┤
│ Departamento     Responsavel operacional     Gestor           │
│ Fiscal           [Maria v]                   [Carlos v]       │
│ Contabil         [Joao v]                    [Carlos v]       │
│ Pessoal          [Nao atribuido v]           [Ana v]          │
├──────────────────────────────────────────────────────────────┤
│ [+ Adicionar departamento]                                    │
└──────────────────────────────────────────────────────────────┘
```

### 8.2 No painel do cliente

Adicionar guia:

```txt
Equipe / Responsaveis
```

Conteudo:

```txt
Departamento | Responsavel | Gestor | Status | Desde | Acoes
```

### 8.3 Na carteira do gestor

Mostrar clientes agrupados por colaborador:

```txt
Gestao → Carteira
├── Colaborador A
│   ├── Cliente 1
│   ├── Cliente 2
├── Colaborador B
│   ├── Cliente 3
```

### 8.4 Na tela de colaborador

A secao `Carteira de clientes` deve buscar os vinculos reais da tabela de responsabilidades.

---

## 9. Endpoints recomendados

### 9.1 Listar responsabilidades de um cliente

```txt
GET /api-v2/clients/:clientId/assignments
```

### 9.2 Atribuir responsavel inicial

```txt
POST /api-v2/clients/:clientId/assignments
```

Payload:

```json
{
  "departmentId": "fiscal-id",
  "responsibleUserId": "user-id",
  "managerUserId": "manager-id",
  "assignmentType": "PRIMARY"
}
```

### 9.3 Alterar responsavel

```txt
PATCH /api-v2/client-assignments/:id
```

### 9.4 Encerrar responsabilidade

```txt
POST /api-v2/client-assignments/:id/end
```

### 9.5 Transferir carteira

```txt
POST /api-v2/client-assignments/transfer
```

Esse endpoint pode substituir gradualmente a logica atual baseada em `TransferSession` + JSON.

---

## 10. Permissoes recomendadas

```txt
client_assignments.read
client_assignments.create
client_assignments.update
client_assignments.transfer
client_assignments.end
client_assignments.audit
```

Regras:

```txt
ADMIN/DEV
- acesso total.

MANAGER
- gerencia carteira dentro dos departamentos autorizados.

DEPARTMENT_MANAGER
- gerencia carteira do proprio departamento.

COLLABORATOR
- visualiza propria carteira.

AUDITOR
- visualiza historico sem alterar.
```

---

## 11. Regras de seguranca

1. Nao permitir departamento livre na atribuicao.
2. Nao permitir responsavel inexistente.
3. Nao permitir responsavel inativo.
4. Nao permitir responsavel fora do departamento, salvo excecao aprovada.
5. Nao permitir dois responsaveis principais ativos para o mesmo cliente/departamento.
6. Toda alteracao deve gerar auditoria.
7. Transferencia temporaria deve ter data de inicio e fim.
8. Transferencia por tempo indeterminado deve registrar justificativa.
9. Historico antigo deve permanecer consultavel.
10. Alteracoes por API devem validar permissao no backend, nao apenas na UI.

---

## 12. Plano de migracao do modelo atual

### Fase 1 — Mapear dados atuais

Ler `clients.metadata` e extrair:

```txt
departamento
responsavelId
```

### Fase 2 — Resolver usuarios

Como o codigo aceita `user.id`, `username` e `colaboradorId`, criar processo de resolucao:

```txt
responsavelId atual
    ↓
procurar em user.id
    ↓
procurar em username
    ↓
procurar em metadata.colaboradorId
```

### Fase 3 — Criar assignments normalizados

Gerar registros em:

```txt
client_department_assignments
```

### Fase 4 — Manter compatibilidade temporaria

Durante transicao, o sistema pode continuar lendo `metadata`, mas deve priorizar a tabela nova.

```txt
1. Ler client_department_assignments
2. Se nao existir, fallback para clients.metadata
```

### Fase 5 — Remover dependencia operacional do metadata

Quando a migracao estiver completa:

```txt
metadata = somente dados complementares
assignments = fonte oficial da carteira
```

---

## 13. Decisao recomendada

A responsabilidade de clientes nao deve continuar escondida em `metadata`.

A decisao recomendada e:

```txt
Criar modulo ClientAssignmentsModule.
Criar tabela client_department_assignments.
Transformar Departamento em entidade controlada.
Permitir atribuicao inicial no cadastro do cliente.
Permitir alteracao/transferencia pelo gestor.
Exibir carteira real no colaborador e no gestor.
Auditar toda mudanca de responsabilidade.
```

---

## 14. Resumo executivo

Hoje, o Portal Sama possui leitura de responsabilidade por cliente, mas ainda nao possui um fluxo claro e seguro para atribuicao inicial.

O que existe:

```txt
- responsabilidade inferida por clients.metadata;
- transferencia de carteira que altera metadata;
- planilha Fiscal filtrando por responsavelId;
- gestor lendo relacoes por metadata;
- tela de clientes do departamento exibindo responsavel extraido do metadata.
```

O que falta:

```txt
- tabela propria de responsabilidade;
- endpoint direto de atribuicao inicial;
- UI para escolher responsavel por departamento no cadastro do cliente;
- carteira real no detalhe do colaborador;
- historico/auditoria detalhada por troca de responsavel;
- departamentos normalizados;
- separacao entre responsavel operacional e gestor.
```

Regra final recomendada:

```txt
Cliente nao deve pertencer genericamente a um usuario.
Cliente deve ter responsabilidades por departamento, com responsavel operacional, gestor, status, periodo e auditoria.
```

---

## 15. Implementacao parcial em 2026-05-27 16:31

Primeira fundacao backend implementada em `portal-sama-api`:

```txt
ClientAssignmentsModule
client_department_assignments
GET /api-v2/clients/:clientId/assignments
POST /api-v2/clients/:clientId/assignments
PATCH /api-v2/client-assignments/:id
POST /api-v2/client-assignments/:id/end
```

O que ja foi coberto:

```txt
- departamento vem do catalogo controlado;
- responsavel e gestor precisam ser usuarios internos ativos;
- responsavel operacional precisa pertencer ao departamento selecionado;
- dois PRIMARY ativos para o mesmo cliente/departamento sao bloqueados em regra de aplicacao;
- criacao, atualizacao e encerramento registram auditoria;
- RBAC inclui client_assignments.read/create/update/transfer/end/audit.
```

O que segue pendente:

```txt
- aplicar migration/seed no MySQL real;
- criar UI no cadastro/painel do cliente;
- carregar carteira real no detalhe do colaborador;
- migrar gestor, departamentos e transferencias para priorizar a tabela nova;
- criar endpoint especifico de transferencia normalizada;
- executar backfill de clients.metadata para client_department_assignments;
- manter fallback temporario para metadata ate a migracao ser conferida.
```
