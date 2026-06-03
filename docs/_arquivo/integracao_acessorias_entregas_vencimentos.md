# [PARCIAL] Integração Acessórias — Entregas, Planilha Fiscal, Vencimentos e Notificações

## 1. Visão geral

Esta documentação descreve a funcionalidade futura de integração entre o **Portal Sama** e o módulo de **entregas/obrigações do Acessórias**.

A funcionalidade tem como objetivo transformar o Portal Sama em uma camada operacional inteligente sobre as obrigações do Acessórias, permitindo:

- gerar automaticamente a planilha padrão do departamento Fiscal por competência;
- criar linhas por empresa e colunas por obrigação;
- sincronizar automaticamente baixas realizadas no Acessórias;
- atualizar a célula correta da planilha para o status visual **Acessórias**;
- substituir gradualmente a página atual de vencimentos por uma **Central de Vencimentos** baseada nas entregas do Acessórias;
- notificar colaboradores a partir de 7 dias antes do vencimento;
- alertar gestores quando houver divergência entre o Portal Sama e o Acessórias;
- manter funcionamento local mesmo se o Acessórias estiver indisponível;
- aprender recorrências de obrigações para prever vencimentos futuros.

Esta funcionalidade **não deve ser implementada agora**. Ela deve ser planejada como uma fase futura, para ser executada somente após a conclusão do grande plano principal de migração, refatoração, segurança e estruturação do Portal Sama.

Atualizacao 2026-05-28: foi aberta uma excecao limitada para a Home por perfil. O Portal Sama passou a consultar um resumo read-only do Acessorias via backend (`GET /api-v2/integrations/acessorias/home-summary`), usando `ACESSORIAS_BASE_URL` e `ACESSORIAS_TOKEN` no ambiente da API. Essa entrega nao substitui o MVP futuro descrito neste documento: ainda nao havia, naquele momento, planilha Fiscal automatica, Central de Vencimentos, notificacoes por vencimento nem auditoria de sync runs.

Atualizacao 2026-05-28: foi adicionada tambem uma fundacao backend para cadastro inicial/importacao controlada de empresas e colaboradores vindos do Acessorias. A API passa a expor `GET /api-v2/integrations/acessorias/registrations/preview` e `POST /api-v2/integrations/acessorias/registrations/sync`, usando `ACESSORIAS_CLIENTS_PATH` e `ACESSORIAS_COLLABORATORS_PATH`. Essa importacao nao transforma o Acessorias em fonte editavel bidirecional: o Portal Sama salva o vinculo externo em `metadata.acessorias`, permite edicoes locais normalmente e, por padrao, atualiza apenas metadata de registros ja existentes. Sobrescrever dados locais exige `overwriteLocal=true` em sincronizacao manual autorizada.

Atualizacao 2026-05-28: foi criada a primeira base backend do MVP de entregas. A API passa a ter as tabelas `acessorias_deliveries` e `acessorias_delivery_sync_runs`, variavel `ACESSORIAS_DELIVERIES_PATH`, endpoint `POST /api-v2/integrations/acessorias/deliveries/sync` para sincronizacao manual auditada e endpoint `GET /api-v2/integrations/acessorias/deliveries` para listar entregas salvas localmente. Essa etapa ainda nao cria planilha Fiscal automatica, Central de Vencimentos, divergencias, alertas D-7/D-3/D-1/D0/D+1 ou scheduler automatico.

Atualizacao 2026-06-01: foi criada a camada backend de mapeamento entre entregas do Acessorias e colunas da planilha Fiscal. A API passa a ter a tabela `acessorias_delivery_column_mappings`, endpoint `GET /api-v2/integrations/acessorias/delivery-column-mappings`, endpoint `GET /api-v2/integrations/acessorias/delivery-column-mappings/suggestions`, `POST /api-v2/integrations/acessorias/delivery-column-mappings` e `PATCH /api-v2/integrations/acessorias/delivery-column-mappings/:id`. Essa etapa permite confirmar mapeamentos com auditoria, mas ainda nao atualiza automaticamente celulas da planilha Fiscal.

Atualizacao 2026-06-01 11:30: foi implementada a primeira aplicacao segura de mapeamentos confirmados na planilha Fiscal. A API passa a ter as tabelas `acessorias_fiscal_apply_runs` e `acessorias_fiscal_divergences`, alem do endpoint protegido `POST /api-v2/integrations/acessorias/deliveries/apply-to-fiscal`. A regra e conservadora: somente entrega `DELIVERED`, cliente identificado, competencia aberta, coluna Fiscal valida e mapeamento `CONFIRMED` com confianca minima geram status visual `ACESSORIAS` na celula. Qualquer caso inseguro cria divergencia aberta e nao altera a planilha. O Web em `/departamentos/modelo` exibe contadores, celulas `Acessorias` e divergencias por empresa/celula. Atualizacao 2026-06-01: a revisao manual local das divergencias foi implementada por `PATCH /api-v2/integrations/acessorias/deliveries/divergences/:id` e botoes `Resolver`/`Ignorar`. Ainda falta validar com contrato real no EasyPanel, Central de Vencimentos, scheduler e notificacoes.

Atualizacao 2026-06-01 complementar: a aplicacao deixou de ser apenas Fiscal no fluxo operacional. `/departamentos/modelo` agora permite selecionar Fiscal, Contabil, Pessoal, Financeiro e Legalizacao; cada departamento tem colunas proprias e metadados separados no cliente. A API adicionou `POST /api-v2/integrations/acessorias/deliveries/apply-to-workspace` e rotas genericas `/api-v2/departments/workspace*`, mantendo as rotas antigas como compatibilidade. A Home do Acessorias tambem foi ajustada para usar `deliveries` como fallback, tratar `204 No Content` como retorno valido sem entregas e exibir diagnostico sanitizado sem expor segredo.

Atualizacao 2026-06-01 contrato oficial: a validacao em `integracao_acessorias_paths_importacao_dev.md` confirmou que Home e entregas devem usar `deliveries/ListAll` com `DtInitial`, `DtFinal`, `DtLastDH`, `situation`, `config` e `Pagina`; empresas devem usar `companies/ListAll` com paginacao; e `departments/ListAll` nao deve ser usado como fonte de colaboradores. O backend local foi ajustado para achatar `empresa -> Entregas[]`, normalizar campos reais e extrair responsaveis a partir dos departamentos das empresas quando nao houver endpoint oficial de colaboradores.

Atualizacao 2026-06-01 importacao DEV: a rota React `/dev` agora possui painel `Integracao Acessorias` para testar conexao, pre-visualizar e importar clientes, responsaveis e entregas. O backend adicionou `GET /api-v2/integrations/acessorias/deliveries/preview` para consultar entregas externas sem persistir dados. A localizacao dos botoes e o contrato curto continuam centralizados em `integracao_acessorias_paths_importacao_dev.md`.

Atualizacao 2026-06-01 vencimentos no workspace: `GET /api-v2/departments/workspace` passou a incluir entregas Acessorias com vencimento no mes selecionado no carrossel operacional. Quando a entrega possui cliente visivel e mapeamento `CONFIRMED` para coluna valida, o vencimento tambem aparece na celula e participa do bloqueio por vencimento. Sem mapeamento confirmado, o item aparece apenas no carrossel, preservando a regra conservadora de nao alterar/bloquear celula por associacao insegura.

Atualizacao 2026-06-01 paginacao e Central: os servicos Acessorias de Home, entregas e cadastros passaram a paginar `ListAll` ate lista vazia, respeitar `ACESSORIAS_RATE_LIMIT_PER_MINUTE`, aceitar `ACESSORIAS_MAX_PAGES`, tentar novamente em `429` e usar timeout longo no Web para importacoes manuais. A sincronizacao de entregas registra `incremental_since` e usa o ultimo run bem-sucedido como marco incremental. Foi criada a rota React `/departamentos/vencimentos`, consolidando vencimentos de calendario e Acessorias do workspace com filtros operacionais. Esta Central esta implementada localmente, mas nao homologada ate validacao no EasyPanel com dados reais.

Atualizacao 2026-06-02 notificacoes manuais: a API passou a expor `POST /api-v2/integrations/acessorias/deliveries/notifications/generate`, que gera notificacoes deduplicadas para vencimento proximo, atraso, baixa confirmada e divergencia aberta. O endpoint e protegido por JWT, CSRF, `integrations.acessorias.deliveries.manage` e `notifications.create`, registra auditoria e mira o responsavel da entrega quando existir ou o departamento operacional. A rota `/dev` ganhou o botao `Gerar notificacoes`. Esta etapa e manual e local; scheduler e homologacao no EasyPanel seguem pendentes.

Atualizacao 2026-06-02 scheduler opt-in: foi criado `AcessoriasSchedulerService`, desabilitado por padrao. Quando `ACESSORIAS_SCHEDULER_ENABLED=true`, o backend agenda sincronizacao periodica de entregas com intervalo configuravel, evita sobreposicao com sync `RUNNING` recente, registra auditoria `integrations.acessorias.scheduler.run` e permite gerar notificacoes automaticas somente se `ACESSORIAS_SCHEDULER_NOTIFICATIONS_ENABLED=true`. O status fica disponivel em `GET /api-v2/integrations/acessorias/scheduler/status` e no botao `Status scheduler` da area `/dev`. Esta automacao ainda precisa ser validada no EasyPanel com dados reais antes de qualquer homologacao.

---

## 2. Decisão de implementação

### 2.1 Decisão principal

A integração de entregas do Acessórias, geração automática da planilha Fiscal, Central de Vencimentos e notificações por vencimento será planejada como uma funcionalidade futura, a ser implementada.

### 2.2 Status da funcionalidade

```txt
Status: Planejada para fase futura, com excecao read-only ja criada para Home
Prioridade: Alta 
Momento de implementação: Pós-migração base TypeScript/NestJS/React
Departamentos operacionais iniciais: Fiscal, Contabil, Pessoal, Financeiro e Legalizacao
```

### 2.3 Motivo para não implementar agora

Esta integração depende de bases estruturais que ainda precisam ser finalizadas no plano principal:

- backend NestJS estruturado;
- frontend React estruturado;
- autenticação segura;
- RBAC e permissões;
- auditoria;
- módulo de notificações;
- módulo de planilhas departamentais;
- módulo base de integrações;
- Prisma/MySQL configurado;
- testes mínimos funcionando;
- configuração segura de variáveis de ambiente;
- logs estruturados;
- tratamento global de erros.

Implementar esta integração antes dessas bases aumentaria o risco de:

- exposição do token do Acessórias;
- sincronização incorreta de obrigações;
- atualização indevida de planilhas;
- ausência de auditoria;
- ausência de controle de permissões;
- dificuldade de teste;
- acoplamento com código legado;
- retrabalho após a migração principal.

---

## 3. Posicionamento no roadmap

Esta funcionalidade deve entrar como uma etapa posterior.

Ordem recomendada:

```txt
1. Migração base para NestJS + TypeScript
2. Migração base para React + TypeScript
3. Autenticação segura
4. RBAC/permissões
5. Auditoria
6. Banco MySQL + Prisma
7. Estrutura de documentos/storage seguro
8. Estrutura base de notificações
9. Estrutura base de planilhas/departamentos
10. Módulo base de integrações
11. Integração Acessórias — Entregas e Vencimentos
12. QA de UI/UX seguindo `ux-ui-docs/09-checklist-aceite.md`
```

Depois que a integração com o Acessórias estiver funcionalmente validada, a etapa de melhoria de interface deve seguir a documentação [`ux-ui-docs/`](ux-ui-docs/). Esse QA de UI/UX será a última verificação antes de liberar o sistema para os usuários utilizarem.

### 3.1 Etapa futura sugerida no roadmap

Adicionar ao `docs/ROADMAP_REFATORACAO.md` uma etapa futura:

```md
## Etapa 7 — Integrações operacionais e automações

### Integração Acessórias — Entregas

- Consultar entregas do Acessórias automaticamente.
- Gerar planilha Fiscal por competência.
- Criar colunas com base nas obrigações sincronizadas.
- Atualizar status `Acessórias` quando houver baixa externa.
- Refatorar página de vencimentos para Central de Vencimentos.
- Notificar colaboradores em D-7, D-3, D-1, D0 e D+1.
- Detectar recorrências de obrigações.
- Criar vencimentos previstos quando o Acessórias estiver indisponível.
- Registrar auditoria de sincronização e alterações.
```

---

## 4. Dependências obrigatórias antes da implementação

Antes de iniciar esta funcionalidade, deve-se validar se já existem:

```txt
AuthModule
PermissionsModule / RBAC
AuditModule
NotificationsModule
DepartmentSheetModule
IntegrationsModule
Configuração segura de .env
Banco modelado com Prisma
Frontend React base
Layout administrativo
Controle de perfis
Logs estruturados
Tratamento global de erros
Testes mínimos de backend
Testes mínimos de frontend
```

### 4.1 Regra para IA ou desenvolvedor

A funcionalidade de **Integração Acessórias — Entregas e Vencimentos** não deve ser implementada antes da conclusão das etapas principais de migração e segurança.

Antes de iniciar essa funcionalidade, validar se já existem:

- backend NestJS estruturado;
- frontend React estruturado;
- autenticação segura;
- RBAC;
- auditoria;
- módulo de notificações;
- módulo de planilhas departamentais;
- módulo base de integrações;
- Prisma/MySQL configurado;
- testes mínimos funcionando.

Se qualquer uma dessas dependências não existir, a IA ou desenvolvedor deve registrar a funcionalidade como pendente e continuar o plano principal.

---

## 5. Escopo aprovado

O escopo aprovado para o MVP operacional amplo continua sendo utilizar **principalmente o recurso de entregas do Acessórias**.

Como preparacao para as telas de `ux-ui-docs/`, fica aprovada uma excecao de cadastro inicial: o Portal Sama pode consultar empresas e colaboradores no Acessorias para criar ou vincular registros locais. Essa excecao existe para reduzir retrabalho de cadastro e melhorar a Home/carteiras, mas deve respeitar estas regras:

- o token do Acessorias permanece somente no backend;
- a importacao e manual/autorizada, com preview antes de aplicar;
- empresas sao conciliadas por CNPJ e, quando existir, por ID externo;
- colaboradores sao conciliados por usuario, e-mail e, quando existir, por ID externo;
- registros importados ficam editaveis no Portal Sama;
- alteracoes locais nao devem ser sobrescritas automaticamente;
- sincronizacoes posteriores atualizam apenas `metadata.acessorias`, salvo quando um operador usar explicitamente `overwriteLocal=true`;
- o Portal Sama nao altera dados no Acessorias.

Não faz parte deste primeiro escopo:

- alterar status de clientes no Acessórias;
- alterar responsáveis por departamento no Acessórias;
- integrar com Domínio;
- integrar com Active Directory;
- alterar dados no Acessórias a partir do Portal Sama.

Sincronizacao completa e bidirecional de cadastros permanece fora do escopo. A entrega atual cobre somente importacao/vinculo controlado e preserva o Portal Sama como camada local editavel.

---

## 6. Departamento inicial

O primeiro departamento de validação será o **Fiscal**.

Motivos:

- possui obrigações recorrentes;
- depende fortemente de prazos;
- a baixa no Acessórias tem impacto direto na operação;
- permite validar o modelo de planilha por competência;
- permite validar alertas para colaboradores e gestores;
- reduz retrabalho operacional;
- serve como prova de conceito antes de expandir para outros departamentos.

---

## 7. Fonte principal de vencimentos

A partir desta funcionalidade, para o departamento Fiscal:

```txt
Acessórias = fonte principal dos vencimentos oficiais das obrigações.
Portal Sama = camada de operação, visualização, histórico, alerta, previsão e continuidade.
```

A página atual de vencimentos deve ser reaproveitada como **Central de Vencimentos**, e não simplesmente removida.

---

## 8. Conceito principal da solução

O Acessórias possui entregas/obrigações com empresa, competência, vencimento e status. O Portal Sama deve consumir essas informações, salvar localmente e transformar esses dados em uma planilha operacional por departamento.

A estrutura conceitual é:

```txt
Departamento + Competência
        ↓
Planilha padrão
        ↓
Linha = empresa
Coluna = obrigação
Célula = status da obrigação naquela competência
```

Exemplo:

```txt
Departamento: Fiscal
Competência: 2026-04
```

Visual esperado:

| Empresa | DAS | DCTFWeb | EFD Contribuições | EFD ICMS/IPI | Status geral |
|---|---|---|---|---|---|
| Empresa A | Pendente | Acessórias | OK | Pendente | Parcial |
| Empresa B | Acessórias | Acessórias | Pendente | OK | Parcial |
| Empresa C | Pendente | Pendente | Pendente | Pendente | Pendente |

---

## 9. Fluxo geral de funcionamento

```txt
Job automático agenda sincronização
        ↓
Backend NestJS consulta entregas no Acessórias
        ↓
Dados retornados são normalizados
        ↓
Portal Sama salva entregas localmente
        ↓
Portal Sama identifica competência, empresa e obrigação
        ↓
Portal Sama gera ou atualiza a planilha Fiscal
        ↓
Portal Sama cria ou atualiza células por empresa + obrigação + competência
        ↓
Se entrega estiver baixada no Acessórias, célula recebe status Acessórias
        ↓
Portal Sama agenda notificações de vencimento
        ↓
Portal Sama gera alertas de divergência quando necessário
        ↓
Auditoria registra todo o processo
```

---

## 10. Sincronização automática

### 10.1 Regra principal

A sincronização com o Acessórias deve ocorrer automaticamente.

Motivo:

- colaboradores podem esquecer de sincronizar manualmente;
- a finalidade da integração é reduzir dependência de ações manuais;
- vencimentos e baixas precisam estar sempre atualizados;
- gestores precisam ser alertados sem depender de clique manual.

### 10.2 Frequência recomendada

Para o MVP Fiscal:

```txt
Horário comercial:
- sincronização incremental a cada 15 minutos

Fora do horário comercial:
- sincronização reduzida a cada 1 hora

Madrugada:
- sincronização completa diária
```

Exemplo:

```txt
08:00 às 18:30 → sincronizar a cada 15 minutos
18:30 às 22:00 → sincronizar a cada 1 hora
02:00 → sincronização completa
```

### 10.3 Sincronização manual

Pode existir botão manual apenas para:

- administrador;
- gestor autorizado;
- suporte técnico;
- reprocessamento emergencial.

Permissão sugerida:

```txt
integrations.acessorias.deliveries.sync_manual
```

O botão manual não deve ser parte do fluxo operacional comum.

---

## 11. Identificação da linha e da coluna

### 11.1 Linha da planilha

A linha representa a empresa/cliente.

A identificação deve seguir esta ordem de prioridade:

```txt
1. ID externo do Acessórias vinculado ao cliente local
2. CNPJ/CPF
3. ID interno do cliente no Portal Sama
4. Nome da empresa apenas como fallback
```

O ideal é manter uma tabela de mapeamento entre o cliente local e a empresa no Acessórias.

### 11.2 Coluna da planilha

A coluna representa a obrigação.

A coluna não deve ser identificada apenas pelo texto visual. O sistema deve utilizar um identificador técnico.

Exemplo:

```txt
Nome exibido: DCTFWeb
Chave técnica: DCTFWEB
Aliases: DCTF WEB, DCTF-Web, DCTFWeb Mensal, Declaração DCTFWeb
```

### 11.3 Célula exata

A célula correta deve ser identificada por:

```txt
client_id + department_id + sheet_column_id + competence
```

Essa combinação deve ser única no banco.

Regra de unicidade recomendada:

```txt
UNIQUE(client_id, department_id, sheet_column_id, competence)
```

---

## 12. Mapeamento entre entregas do Acessórias e colunas

### 12.1 Necessidade do mapeamento

O sistema não deve tentar adivinhar a coluna de forma insegura.

Deve existir uma camada de mapeamento entre:

```txt
Entrega do Acessórias
        ↓
Coluna técnica da planilha padrão
```

### 12.2 Tipos de mapeamento

```txt
MANUAL
AUTO_CONFIRMED
AUTO_SUGGESTED
IGNORED
```

### 12.3 Regra de segurança

Somente mapeamentos confirmados devem atualizar automaticamente a planilha.

Se o sistema não conseguir identificar a coluna com segurança:

```txt
Não atualizar automaticamente.
Criar divergência para revisão do gestor.
```

### 12.4 Fluxo de configuração das colunas

Tela sugerida:

```txt
Configurações → Integrações → Acessórias → Mapeamento de Entregas
```

Exemplo:

| Entrega Acessórias | Departamento Acessórias | Coluna Portal Sama | Status |
|---|---|---|---|
| DCTF WEB | Fiscal | DCTFWeb | Confirmado |
| DAS | Fiscal | DAS | Confirmado |
| EFD CONTRIBUIÇÕES | Fiscal | EFD Contribuições | Confirmado |
| GIA | Fiscal | Não mapeado | Revisar |

Status implementado parcialmente em 2026-06-01: mapeamentos podem ser listados, sugeridos com base nas entregas locais, criados/atualizados com permissao `integrations.acessorias.deliveries.manage` e auditados. A etapa de 2026-06-01 11:30 passou a usar somente mapeamentos confirmados para aplicar baixas `DELIVERED` na planilha Fiscal com status `ACESSORIAS`; mapeamentos ausentes, baixa confianca, cliente nao encontrado, conflito de status ou mes fechado geram divergencia aberta.

---

## 13. Geração automática da planilha Fiscal por competência

### 13.1 Objetivo

Gerar a planilha padrão do departamento Fiscal com base nas entregas retornadas pelo Acessórias.

### 13.2 Entrada do processo

```txt
Departamento: Fiscal
Competência: 2026-04
Fonte: Acessórias
```

### 13.3 Processo

```txt
1. Buscar entregas do Acessórias para o departamento Fiscal.
2. Filtrar ou identificar a competência.
3. Identificar empresas.
4. Identificar obrigações.
5. Criar planilha Fiscal da competência, se não existir.
6. Criar colunas com base nas obrigações encontradas.
7. Criar linhas/células por empresa + obrigação.
8. Definir status inicial.
9. Salvar vencimentos oficiais.
10. Agendar notificações.
11. Registrar auditoria.
```

### 13.4 Criação de colunas

As colunas podem ser criadas com base nas obrigações encontradas no Acessórias.

No piloto, recomenda-se que novas colunas nasçam como:

```txt
PENDING_APPROVAL
```

Depois de confirmadas pelo gestor Fiscal, passam para:

```txt
ACTIVE
```

### 13.5 Obrigações não aplicáveis

Se uma obrigação existe para algumas empresas e não para outras, existem duas opções:

#### Opção A — Coluna global

A coluna aparece para todas as empresas, mas algumas células ficam:

```txt
Não se aplica
```

#### Opção B — Coluna apenas se existir na competência

A planilha cria apenas colunas encontradas na competência.

Para o MVP, recomenda-se a **Opção B**, por ser mais simples. Para maturidade operacional, a **Opção A** pode ser adotada futuramente.

---

## 14. Status da planilha

### 14.1 Status internos sugeridos

```txt
PENDING
IN_PROGRESS
OK
ACESSORIAS_BAIXADO
LATE
BLOCKED
CANCELED
NOT_APPLICABLE
```

### 14.2 Exibição visual

| Status técnico | Nome exibido | Cor sugerida | Significado |
|---|---|---|---|
| PENDING | Pendente | Amarelo | Ainda não tratado |
| IN_PROGRESS | Em andamento | Azul | Em execução |
| OK | OK | Verde | Concluído internamente |
| ACESSORIAS_BAIXADO | Acessórias | Roxo | Baixado no Acessórias |
| LATE | Atrasado | Vermelho | Vencido |
| BLOCKED | Bloqueado | Cinza/Preto | Impedido |
| CANCELED | Cancelado | Cinza | Cancelado |
| NOT_APPLICABLE | Não se aplica | Cinza claro | Não aplicável |

### 14.3 Diferença entre OK e Acessórias

**OK** significa que a obrigação foi concluída internamente no Portal Sama.

**Acessórias** significa que a obrigação foi baixada no sistema Acessórias e sincronizada para o Portal Sama.

A diferença deve ser preservada para permitir alertas de divergência.

---

## 15. Regras de atualização automática

### 15.1 Baixado no Acessórias e pendente no Portal Sama

```txt
Portal Sama: PENDING
Acessórias: DELIVERED
```

Ação:

```txt
Atualizar display_status para ACESSORIAS_BAIXADO.
Registrar histórico.
Registrar auditoria.
Cancelar notificações futuras.
```

### 15.2 OK no Portal Sama e pendente no Acessórias

```txt
Portal Sama: OK
Acessórias: PENDING
```

Ação:

```txt
Manter OK.
Gerar alerta para gestor.
Exibir badge: Baixa Acessórias pendente.
```

### 15.3 OK no Portal Sama e baixado no Acessórias

```txt
Portal Sama: OK
Acessórias: DELIVERED
```

Ação:

```txt
Atualizar display_status para ACESSORIAS_BAIXADO.
Registrar histórico.
Resolver alertas pendentes.
```

### 15.4 Pendente no Portal Sama e pendente no Acessórias

```txt
Portal Sama: PENDING
Acessórias: PENDING
```

Ação:

```txt
Não alterar status.
Atualizar última sincronização.
Manter notificações conforme vencimento.
```

### 15.5 Entrega não mapeada

Se a entrega existe no Acessórias, mas não há coluna/célula correspondente:

```txt
Não atualizar automaticamente.
Criar divergência UNMATCHED_ACESSORIAS_DELIVERY.
Enviar para revisão do gestor.
```

---

## 16. Central de Vencimentos

### 16.1 Substituição da página atual de vencimentos

A página atual de vencimentos não deve ser simplesmente removida. Ela deve ser refatorada para se tornar a:

```txt
Central de Vencimentos
```

A nova Central deve exibir:

- vencimentos oficiais vindos do Acessórias;
- vencimentos previstos pelo Portal Sama;
- vencimentos manuais excepcionais;
- status de baixa no Acessórias;
- status interno da planilha;
- obrigações pendentes;
- obrigações atrasadas;
- notificações enviadas;
- divergências.

### 16.2 Fonte dos vencimentos

Para o departamento Fiscal:

```txt
Acessórias = fonte principal dos vencimentos oficiais.
Portal Sama = camada local de operação, alerta, histórico e previsão.
```

### 16.3 Tipos de origem

```txt
ACESSORIAS
PORTAL_SAMA_PREDICTED
MANUAL
```

### 16.4 Prioridade das fontes

Quando houver conflito:

```txt
1. Acessórias oficial
2. Ajuste manual aprovado por gestor
3. Previsão inteligente do Portal Sama
```

Se o Acessórias trouxer data diferente da previsão local, a data oficial substitui a previsão.

---

## 17. Notificações de vencimento

### 17.1 Objetivo

Notificar colaboradores e gestores com base nos vencimentos das obrigações do Acessórias.

### 17.2 Regra de notificação

Regras sugeridas:

```txt
D-7: avisar colaborador responsável
D-3: reforço para colaborador
D-1: alerta forte para colaborador e gestor
D0: alerta no dia do vencimento
D+1: alerta de atraso para gestor, se ainda pendente
```

### 17.3 Exemplo

```txt
Obrigação: DCTFWeb
Cliente: Empresa ABC
Competência: 2026-04
Vencimento: 15/05/2026
Responsável: João
```

Notificações:

```txt
08/05/2026 → colaborador
12/05/2026 → colaborador
14/05/2026 → colaborador + gestor
15/05/2026 → colaborador + gestor
16/05/2026 → gestor, se não baixada
```

### 17.4 Canais de notificação

MVP:

```txt
Notificação interna no Portal Sama
E-mail
```

Futuro:

```txt
WhatsApp
Microsoft Teams
Slack
```

### 17.5 Cancelamento automático de notificações

Quando a obrigação for baixada no Acessórias:

```txt
status = ACESSORIAS_BAIXADO
```

O sistema deve:

- cancelar notificações futuras;
- resolver alertas pendentes;
- registrar histórico;
- registrar auditoria.

---

## 18. Motor de recorrência inteligente

### 18.1 Objetivo

Reduzir dependência total do Acessórias por meio de previsões locais baseadas no histórico.

Se o Portal Sama identificar que a mesma obrigação ocorre no mesmo dia em diferentes meses, ele pode sugerir ou criar vencimentos previstos.

### 18.2 Exemplo

Histórico sincronizado:

```txt
DAS — 20/03/2026
DAS — 20/04/2026
DAS — 20/05/2026
```

Previsão:

```txt
DAS — 20/06/2026
DAS — 20/07/2026
```

### 18.3 Regras de recorrência

```txt
2 ocorrências consecutivas → sugerir recorrência.
3 ocorrências consecutivas → ativar previsão com confiança média.
6 ocorrências consecutivas → confiança alta.
```

### 18.4 Previsão não é vencimento oficial

Vencimento previsto deve ser identificado na interface como:

```txt
Previsto pelo Portal Sama
```

Vencimento oficial deve ser identificado como:

```txt
Oficial Acessórias
```

### 18.5 Confirmação pelo Acessórias

Quando o Acessórias sincronizar o vencimento oficial, a previsão deve ser substituída.

Exemplo:

```txt
Previsto pelo Portal Sama: 20/06/2026
Oficial Acessórias: 21/06/2026
Resultado: 21/06/2026
```

Registrar histórico:

```txt
Previsão ajustada por dado oficial do Acessórias.
```

---

## 19. Funcionamento quando o Acessórias estiver indisponível

Se o Acessórias estiver fora do ar, o Portal Sama deve continuar funcionando com:

- últimos vencimentos oficiais sincronizados;
- planilhas já geradas;
- status locais;
- previsões inteligentes;
- notificações locais;
- histórico local.

A interface deve exibir aviso:

```txt
Última sincronização Acessórias: 12/05/2026 08:15
Status: falha há 2 horas
Dados exibidos com base na última sincronização local.
```

Quando o Acessórias voltar:

```txt
Portal Sama reprocessa sincronizações.
Compara previsões com dados oficiais.
Atualiza vencimentos.
Atualiza status Acessórias.
Resolve divergências.
```

---

## 20. Arquitetura técnica sugerida

### 20.1 Módulos NestJS

```txt
src/modules/integrations/
  integrations.module.ts

src/modules/integrations/acessorias-deliveries/
  acessorias-deliveries.module.ts
  acessorias-deliveries.controller.ts
  acessorias-deliveries.service.ts
  acessorias-deliveries.client.ts
  acessorias-deliveries.mapper.ts
  acessorias-deliveries-sync.service.ts
  acessorias-deliveries-scheduler.ts
  dto/
  types/

src/modules/department-sheet/
  department-sheet.module.ts
  department-sheet.controller.ts
  department-sheet.service.ts
  department-sheet-generator.service.ts
  department-sheet-status.service.ts
  department-sheet-mapping.service.ts

src/modules/due-dates/
  due-dates.module.ts
  due-dates.service.ts
  due-date-recurrence.service.ts
  due-date-notification.service.ts

src/modules/notifications/
  notifications.module.ts
  notifications.service.ts

src/modules/audit/
  audit.module.ts
  audit.service.ts
```

### 20.2 Serviços principais

```txt
AcessoriasDeliveriesClient
- chama a API do Acessórias

AcessoriasDeliveriesMapper
- normaliza retorno externo

AcessoriasDeliveriesSyncService
- coordena sincronização

DepartmentSheetGeneratorService
- gera planilha por competência

DepartmentSheetMappingService
- resolve coluna correta

DepartmentSheetStatusService
- atualiza status e histórico

DueDateRecurrenceService
- detecta recorrência e cria previsões

DueDateNotificationService
- agenda e envia notificações

AuditService
- registra eventos críticos
```

---

## 21. Endpoints internos sugeridos

### 21.1 Sincronização

```txt
POST /api-v2/integrations/acessorias/deliveries/sync
```

Uso:

- manual emergencial;
- restrito a gestor/admin.

Status implementado parcialmente em 2026-05-28: executa sincronizacao manual, salva/atualiza entregas por `external_id`, registra `acessorias_delivery_sync_runs` e auditoria `integrations.acessorias.deliveries.sync`. Scheduler automatico e conciliacao com planilha Fiscal permanecem pendentes.

### 21.2 Listar entregas sincronizadas

```txt
GET /api-v2/integrations/acessorias/deliveries
```

Filtros:

```txt
clientId
departmentId
status
competence
dueDateStart
dueDateEnd
responsibleId
```

Status implementado parcialmente em 2026-05-28: lista entregas persistidas em `acessorias_deliveries`, com filtros basicos por departamento, status, competencia, responsavel e documento do cliente.

### 21.3 Divergências

```txt
GET /api-v2/integrations/acessorias/deliveries/divergences
```

Status implementado parcialmente em 2026-06-01 11:30: divergencias passaram a ser persistidas em `acessorias_fiscal_divergences` durante `POST /api-v2/integrations/acessorias/deliveries/apply-to-fiscal` e exibidas no workspace Fiscal. Atualizacao 2026-06-01: `PATCH /api-v2/integrations/acessorias/deliveries/divergences/:id` permite revisar manualmente uma divergencia como `RESOLVED`, `IGNORED` ou `OPEN`, com CSRF, RBAC, auditoria e historico em metadata; `/departamentos/modelo` oferece as acoes `Resolver` e `Ignorar`. Vencimentos Acessorias sincronizados tambem aparecem no carrossel/workspace quando possuem `dueAt`. Ainda faltam endpoint dedicado de listagem geral, validacao real no EasyPanel, alertas e Central de Vencimentos dedicada.

### 21.4 Resolver alerta

```txt
POST /api-v2/integrations/acessorias/delivery-alerts/:id/resolve
```

### 21.5 Vincular entrega à célula da planilha

```txt
POST /api-v2/integrations/acessorias/deliveries/:deliveryId/link-sheet-item
```

### 21.6 Gerar planilha por competência

```txt
POST /api-v2/department-sheets/fiscal/generate
```

Payload sugerido:

```json
{
  "competence": "2026-04",
  "source": "ACESSORIAS"
}
```

### 21.7 Listar planilha por competência

```txt
GET /api-v2/department-sheets/fiscal?competence=2026-04
```

### 21.8 Central de vencimentos

```txt
GET /api-v2/due-dates
```

Filtros:

```txt
competence
departmentId
responsibleId
clientId
source
status
dueDateStart
dueDateEnd
```

---

## 22. Banco de dados proposto

### 22.1 `acessorias_deliveries`

Armazena cópia local das entregas consultadas.

```txt
id
external_id
client_id
client_identifier
department_id
department_name
obligation_name
normalized_obligation_name
competence
due_date
delivery_status
delivered_at
responsible_name
responsible_email
raw_payload
last_synced_at
created_at
updated_at
```

### 22.2 `department_sheets`

Representa a planilha de um departamento em uma competência.

```txt
id
department_id
competence
status
generated_from
generated_at
created_by
created_at
updated_at
```

### 22.3 `department_sheet_columns`

Representa as colunas oficiais da planilha.

```txt
id
department_sheet_id
department_id
obligation_key
display_name
source
order_index
status
created_at
updated_at
```

### 22.4 `department_sheet_column_aliases`

Armazena variações de nomes de obrigação.

```txt
id
column_id
alias
normalized_alias
created_at
updated_at
```

### 22.5 `acessorias_delivery_column_mappings`

Mapeia entregas do Acessórias para colunas do Portal Sama.

```txt
id
acessorias_delivery_name
normalized_delivery_name
acessorias_department_id
acessorias_department_name
portal_department_id
sheet_column_id
confidence
mapping_type
status
created_at
updated_at
```

### 22.6 `department_sheet_items`

Representa cada célula da planilha.

```txt
id
department_sheet_id
client_id
department_id
sheet_column_id
competence
due_date
internal_status
acessorias_status
display_status
external_delivery_id
last_acessorias_sync_at
created_at
updated_at
```

Restrição recomendada:

```txt
UNIQUE(client_id, department_id, sheet_column_id, competence)
```

### 22.7 `department_sheet_status_history`

Histórico de alteração de status.

```txt
id
sheet_item_id
previous_internal_status
new_internal_status
previous_acessorias_status
new_acessorias_status
previous_display_status
new_display_status
source
changed_by
changed_at
metadata
```

### 22.8 `obligation_due_dates`

Tabela central de vencimentos.

```txt
id
client_id
department_id
sheet_column_id
obligation_name
competence
due_date
source
confidence
external_delivery_id
status
created_at
updated_at
```

Valores de `source`:

```txt
ACESSORIAS
PORTAL_SAMA_PREDICTED
MANUAL
```

### 22.9 `obligation_recurrence_rules`

Regras aprendidas de recorrência.

```txt
id
department_id
sheet_column_id
client_id nullable
frequency
day_of_month
weekday_rule
confidence
source
status
last_detected_at
created_at
updated_at
```

### 22.10 `obligation_due_date_history`

Histórico de alterações de vencimento.

```txt
id
obligation_due_date_id
previous_due_date
new_due_date
previous_source
new_source
reason
changed_at
metadata
```

### 22.11 `obligation_notifications`

Notificações programadas e enviadas.

```txt
id
obligation_due_date_id
recipient_user_id
notification_type
scheduled_for
sent_at
status
channel
created_at
updated_at
```

### 22.12 `acessorias_delivery_sync_runs`

Registro de cada sincronização.

```txt
id
started_at
finished_at
status
total_read
total_matched
total_updated
total_unmatched
total_errors
error_summary
created_at
```

### 22.13 `acessorias_delivery_alerts`

Alertas operacionais.

```txt
id
sheet_item_id
client_id
department_id
responsible_user_id
alert_type
message
status
created_at
resolved_at
```

Tipos de alerta:

```txt
INTERNAL_OK_EXTERNAL_PENDING
EXTERNAL_DELIVERED_INTERNAL_PENDING
DELIVERY_OVERDUE
UNMATCHED_ACESSORIAS_DELIVERY
SYNC_FAILED
```

---

## 23. Segurança

### 23.1 Token do Acessórias

O token da API do Acessórias deve ficar apenas no backend.

Regras:

- nunca expor token no React;
- nunca salvar token em localStorage;
- armazenar token em `.env` ou vault;
- não registrar token em logs;
- proteger endpoints de sincronização com RBAC.

### 23.2 Permissões sugeridas

```txt
integrations.acessorias.deliveries.read
integrations.acessorias.deliveries.sync
integrations.acessorias.deliveries.sync_manual
integrations.acessorias.deliveries.manage
integrations.acessorias.deliveries.resolve_alert

department_sheet.read
department_sheet.update_status
department_sheet.view_acessorias_status
department_sheet.manage_mapping

due_dates.read
due_dates.manage_manual
due_dates.view_predictions
notifications.read
notifications.manage
```

### 23.3 Perfis sugeridos

```txt
ADMIN: acesso total
MANAGER: leitura, alertas, divergências e reprocessamento controlado
DEPARTMENT_MANAGER: leitura e gestão do próprio departamento
COLLABORATOR: leitura dos próprios itens e alertas
AUDITOR: leitura de histórico e auditoria
```

### 23.4 Auditoria obrigatória

Registrar:

```txt
Sincronização iniciada
Sincronização concluída
Sincronização falhou
Planilha gerada
Coluna criada
Coluna aprovada
Status alterado por sincronização
Vencimento criado pelo Acessórias
Vencimento previsto pelo Portal Sama
Previsão substituída por vencimento oficial
Notificação enviada
Alerta gerado
Alerta resolvido
Vínculo manual criado entre entrega e item da planilha
Erro de autenticação com Acessórias
Erro de rate limit
```

---

## 24. Frontend necessário

### 24.1 Planilha Fiscal por competência

Funcionalidades:

- seletor de competência;
- linhas por empresa;
- colunas por obrigação;
- status visual por célula;
- badge **Acessórias** com cor roxa;
- indicação de última sincronização;
- tooltip com origem do status;
- filtros por status, empresa, responsável e obrigação.

### 24.2 Central de Vencimentos

Funcionalidades:

- listar vencimentos oficiais;
- listar vencimentos previstos;
- listar vencimentos manuais;
- filtrar por origem;
- filtrar por colaborador;
- filtrar por competência;
- cards de vencimentos próximos;
- cards de atrasados;
- status de baixa no Acessórias;
- histórico de notificações.

### 24.3 Tela de divergências

Funcionalidades:

- entregas sem mapeamento;
- OK interno sem baixa no Acessórias;
- baixado no Acessórias mas pendente no Portal Sama;
- falhas de sincronização;
- conflitos de vencimento;
- botão de resolver divergência;
- botão de criar mapeamento.

### 24.4 Tela de mapeamento de obrigações

Funcionalidades:

- listar obrigações vindas do Acessórias;
- mapear para coluna do Portal Sama;
- cadastrar aliases;
- aprovar colunas sugeridas;
- ignorar obrigação;
- visualizar confiança do matching.

---

## 25. Testes recomendados

### 25.1 Testes unitários

- normalização de nomes de obrigação;
- mapper Acessórias → Portal Sama;
- resolução de coluna por mapeamento;
- cálculo de status visual;
- geração de notificações D-7, D-3, D-1, D0 e D+1;
- detecção de recorrência;
- substituição de previsão por vencimento oficial.

### 25.2 Testes de integração

- sincronização de entregas com dados simulados;
- geração de planilha Fiscal por competência;
- criação de colunas por obrigação;
- criação de células por empresa + obrigação;
- atualização para status Acessórias;
- criação de vencimentos oficiais;
- criação de alertas de divergência;
- cancelamento de notificações após baixa.

### 25.3 Testes E2E

- gestor visualiza planilha Fiscal;
- colaborador visualiza status Acessórias;
- gestor visualiza Central de Vencimentos;
- colaborador recebe alerta D-7;
- gestor visualiza divergência OK interno sem baixa no Acessórias;
- admin cria mapeamento de coluna.

### 25.4 Testes de segurança

- token do Acessórias não aparece no frontend;
- token do Acessórias não aparece em logs;
- usuário sem permissão não sincroniza manualmente;
- colaborador não acessa dados de outro departamento sem permissão;
- endpoint de divergência exige autenticação;
- endpoint de mapeamento exige permissão administrativa;
- falha externa não expõe stack trace.

---

## 26. Fases de implementação futura

### Fase 1 — Validação das dependências

Antes de iniciar a integração, verificar se já existem:

- AuthModule;
- RBAC/PermissionsModule;
- AuditModule;
- NotificationsModule;
- DepartmentSheetModule;
- IntegrationsModule;
- Prisma/MySQL;
- testes mínimos;
- frontend React base.

Se qualquer item estiver ausente, a integração deve permanecer pendente.

### Fase 2 — Base da integração

- criar `IntegrationsModule` se ainda não existir;
- criar `AcessoriasDeliveriesModule`;
- configurar token via `.env`;
- criar client HTTP para Acessórias;
- criar mapper de entregas;
- salvar entregas localmente;
- registrar sync runs.

### Fase 3 — Planilha Fiscal por competência

- criar ou reaproveitar `DepartmentSheetModule`;
- criar tabela `department_sheets`;
- criar tabela `department_sheet_columns`;
- criar tabela `department_sheet_items`;
- gerar planilha Fiscal por competência;
- criar colunas com base nas obrigações;
- criar células por empresa + obrigação.

### Fase 4 — Status Acessórias

- mapear entregas para colunas;
- atualizar células para `ACESSORIAS_BAIXADO`;
- aplicar cor roxa no frontend;
- registrar histórico de status;
- gerar divergências quando não houver mapeamento seguro.

### Fase 5 — Central de Vencimentos

- criar ou reaproveitar `DueDatesModule`;
- salvar vencimentos oficiais;
- reaproveitar/refatorar página de vencimentos;
- exibir origem do vencimento;
- exibir filtros por competência, departamento, responsável e cliente.

### Fase 6 — Notificações

- criar notificações D-7, D-3, D-1, D0 e D+1;
- enviar notificação interna;
- enviar e-mail;
- cancelar notificações futuras quando houver baixa no Acessórias;
- escalar atrasos para gestor.

### Fase 7 — Recorrência inteligente

- detectar padrões mensais;
- criar sugestões de recorrência;
- gerar vencimentos previstos;
- substituir previsão por vencimento oficial;
- exibir confiabilidade da previsão.

### Fase 8 — Expansão futura

- validar comportamento no Fiscal;
- ajustar mapeamentos;
- expandir para outros departamentos;
- avaliar novas integrações.

---

## 27. Critérios de aceite do MVP Fiscal

A funcionalidade só deve ser considerada concluída no MVP se:

- a implementação ocorrer depois da conclusão das dependências principais;
- sincronização automática estiver ativa;
- entregas do Fiscal forem salvas localmente;
- planilha Fiscal for gerada por competência;
- linhas forem criadas por empresa;
- colunas forem criadas por obrigação;
- células forem identificadas por empresa + departamento + obrigação + competência;
- baixas do Acessórias atualizarem status para **Acessórias**;
- status **Acessórias** tiver cor diferenciada;
- vencimentos oficiais forem criados a partir do Acessórias;
- notificações D-7 forem geradas;
- gestores forem alertados em caso de divergência;
- falhas da API externa não quebrarem a operação local;
- token do Acessórias não for exposto;
- auditoria registrar sincronizações e alterações de status;
- testes principais estiverem implementados e executados;
- documentação do status da implementação estiver atualizada.

---

## 28. Pontos a validar na API do Acessórias

Antes da implementação completa, validar:

```txt
1. Qual campo indica que a entrega recebeu baixa.
2. Qual campo traz data/hora da baixa.
3. Se existe ID único estável da entrega.
4. Se existe ID ou código estável do tipo de obrigação.
5. Se a entrega retorna competência.
6. Se a entrega retorna vencimento.
7. Se a entrega retorna CNPJ/CPF da empresa.
8. Se a entrega retorna departamento.
9. Se a entrega retorna responsável.
10. Se existe filtro por departamento.
11. Se existe filtro por empresa/CNPJ.
12. Se existe filtro por status.
13. Se existe filtro por última alteração.
14. Qual o limite de paginação.
15. Qual o limite de requisições.
16. Se entregas canceladas/dispensadas aparecem no retorno.
17. Se existe diferença entre entregue, baixado, protocolado e concluído.
```

---

## 29. Backlog consolidado

### 29.1 Integração de Entregas Acessórias

- **Nome:** Integração Acessórias — Entregas e baixa automática na planilha padrão
- **Prioridade:** Alta após conclusão do plano principal
- **Departamento inicial:** Fiscal
- **Status:** Planejada para fase futura
- **Motivo para não implementar agora:** depende da conclusão da base NestJS/React, autenticação, RBAC, auditoria, notificações e planilhas departamentais.

### 29.2 Planilha Fiscal automática por competência

- **Nome:** Planilha Fiscal automática por competência via entregas Acessórias
- **Prioridade:** Alta após conclusão do plano principal
- **Departamento inicial:** Fiscal
- **Status:** Planejada para fase futura

### 29.3 Central de Vencimentos Acessórias

- **Nome:** Central de Vencimentos baseada nas entregas do Acessórias
- **Prioridade:** Alta após conclusão do plano principal
- **Departamento inicial:** Fiscal
- **Status:** Planejada para fase futura

### 29.4 Motor de recorrência inteligente

- **Nome:** Previsão inteligente de vencimentos recorrentes
- **Prioridade:** Média/Alta
- **Status:** Planejar após sincronização e vencimentos oficiais

---

## 30. Registro sugerido no `docs/BACKLOG_FUNCIONALIDADES.md`

```md
## Integração Acessórias — Entregas, Planilha Fiscal e Central de Vencimentos

- **Status:** Planejada para fase futura
- **Prioridade:** Alta após conclusão do plano principal
- **Momento de implementação:** Após estabilização da nova arquitetura TypeScript/NestJS/React
- **Departamento inicial:** Fiscal
- **Motivo para não implementar agora:** depende de autenticação, RBAC, auditoria, notificações, planilhas departamentais, estrutura de integração e banco Prisma já consolidados.
- **Descrição:** Integrar entregas do Acessórias ao Portal Sama para gerar planilhas fiscais por competência, atualizar status `Acessórias`, substituir a página de vencimentos por uma Central de Vencimentos e notificar colaboradores automaticamente antes dos vencimentos.
- **Dependências:** AuthModule, PermissionsModule, AuditModule, NotificationsModule, DepartmentSheetModule, IntegrationsModule, Prisma/MySQL, frontend React base, testes mínimos.
- **Critério de início:** somente iniciar após conclusão validada das dependências principais.
```

---

## 31. Registro sugerido no prompt mestre do projeto

Adicionar a seguinte regra ao prompt mestre usado pela IA:

```md
A funcionalidade de Integração Acessórias — Entregas e Vencimentos não deve ser implementada antes da conclusão das etapas principais de migração e segurança.

Antes de iniciar essa funcionalidade, validar se já existem:

- backend NestJS estruturado;
- frontend React estruturado;
- autenticação segura;
- RBAC;
- auditoria;
- módulo de notificações;
- módulo de planilhas departamentais;
- módulo base de integrações;
- Prisma/MySQL configurado;
- testes mínimos funcionando.

Se qualquer uma dessas dependências não existir, a IA deve registrar a funcionalidade como pendente e continuar o plano principal.
```

---

## 32. Funcionalidade futura pós-Acessórias — Web Push Notifications

### 32.1 Visão geral

Após a implementação da integração com o Acessórias — Entregas, Planilha Fiscal, Central de Vencimentos e Notificações — deve ser planejada uma camada de **Web Push Notifications** para que os usuários recebam alertas do Portal Sama mesmo quando não estiverem com a tela principal aberta no computador.

Essa funcionalidade deve permitir que notificações importantes da plataforma também sejam enviadas como notificações nativas do navegador, desde que o usuário tenha concedido permissão.

Exemplos de notificações que podem usar Web Push:

```txt
Obrigação vencendo em 7 dias
Obrigação vencendo hoje
Obrigação atrasada
Baixa pendente no Acessórias
Divergência entre Portal Sama e Acessórias
Documento recusado
Nova solicitação de acesso
Contrato aguardando assinatura
Certificado digital próximo do vencimento
Tarefa atribuída ao colaborador
Alerta para gestor
```

### 32.2 Momento de implementação

Esta funcionalidade deve ser implementada **somente após**:

```txt
1. Conclusão do grande plano principal de migração/refatoração.
2. Implementação da integração Acessórias — Entregas e Vencimentos.
3. Existência de um NotificationsModule funcional.
4. Existência de RBAC e auditoria.
5. Existência de preferências de notificação por usuário.
```

Status recomendado:

```txt
Status: Planejada para fase futura pós-Acessórias
Prioridade: Média/Alta
Momento de implementação: Depois da Central de Vencimentos e notificações internas/e-mail
```

### 32.3 Decisão funcional

A Web Push Notification não substitui as notificações internas do Portal Sama.

Ela deve ser uma extensão dos canais de notificação.

Modelo recomendado:

```txt
Notificação interna = canal obrigatório principal
E-mail = canal complementar inicial
Web Push = canal complementar em tempo real
```

Ou seja:

```txt
Portal Sama gera uma notificação
        ↓
Salva no banco
        ↓
Exibe na central interna de notificações
        ↓
Envia e-mail, se configurado
        ↓
Envia Web Push, se o usuário autorizou
```

### 32.4 Objetivo

Permitir que o usuário receba alertas importantes mesmo quando:

- está com o Portal Sama fechado;
- está em outra aba do navegador;
- está usando outro sistema;
- não está na tela principal do Portal Sama;
- precisa ser alertado rapidamente sobre uma obrigação, divergência ou vencimento.

### 32.5 Requisitos funcionais

A funcionalidade deve permitir:

- solicitar permissão de notificação ao usuário;
- registrar o dispositivo/navegador autorizado;
- enviar notificações push para usuários específicos;
- enviar notificações push para grupos/perfis;
- respeitar preferências individuais;
- permitir ativar/desativar Web Push;
- permitir configurar tipos de notificação recebidos por push;
- expirar ou remover assinaturas inválidas;
- registrar auditoria de envio;
- registrar falhas de entrega;
- evitar envio duplicado;
- não enviar dados sensíveis no conteúdo da notificação.

### 32.6 Requisitos de segurança

As notificações Web Push devem seguir regras rígidas de segurança, principalmente porque o Portal Sama lida com documentos empresariais, certificados, contratos e informações sensíveis.

Regras obrigatórias:

```txt
1. Não enviar dados sensíveis no texto da notificação.
2. Não enviar CNPJ completo, CPF, senha, token, certificado, contrato ou documento no push.
3. A notificação deve conter apenas resumo seguro.
4. O clique na notificação deve abrir o Portal Sama e exigir sessão válida.
5. Se o usuário não estiver autenticado, redirecionar para login.
6. A autorização real deve continuar no backend.
7. O endpoint de inscrição push deve exigir autenticação.
8. Cada subscription deve pertencer a um usuário autenticado.
9. O usuário deve poder revogar dispositivos.
10. Falhas de push não devem expor detalhes internos.
```

Exemplo de notificação segura:

```txt
Título: Obrigação próxima do vencimento
Mensagem: Você possui uma obrigação fiscal vencendo em 7 dias.
Ação: Abrir Portal Sama
```

Evitar:

```txt
Título: DCTFWeb Empresa ABC Ltda CNPJ 12.345.678/0001-99
Mensagem: Documento fiscal X está atrasado...
```

### 32.7 Permissões sugeridas

```txt
notifications.read
notifications.manage_preferences
notifications.web_push.subscribe
notifications.web_push.unsubscribe
notifications.web_push.send
notifications.web_push.manage_devices
notifications.web_push.audit
```

Perfis:

```txt
ADMIN: gerenciar configuração global
MANAGER: receber alertas gerenciais
DEPARTMENT_MANAGER: receber alertas do departamento
COLLABORATOR: receber alertas próprios
AUDITOR: visualizar auditoria de notificações
```

### 32.8 Preferências por usuário

O usuário deve poder configurar quais notificações deseja receber por Web Push.

Exemplo:

```txt
Obrigação vencendo em 7 dias: ativado
Obrigação vencendo hoje: ativado
Obrigação atrasada: ativado
Baixa pendente no Acessórias: ativado
Documento recusado: ativado
Contrato aguardando assinatura: desativado
Solicitação de acesso: ativado
```

Também deve ser possível configurar canais:

```txt
Interna: sempre ativa
E-mail: ativável/desativável
Web Push: ativável/desativável
```

### 32.9 Arquitetura técnica sugerida

Adicionar ao backend NestJS:

```txt
src/modules/notifications/
  notifications.module.ts
  notifications.service.ts
  notification-preferences.service.ts
  notification-dispatcher.service.ts

src/modules/notifications/web-push/
  web-push.module.ts
  web-push.controller.ts
  web-push.service.ts
  web-push-subscription.service.ts
  web-push-dispatcher.service.ts
  web-push-cleanup.service.ts
  dto/
  types/
```

Adicionar ao frontend React:

```txt
src/services/web-push-service.ts
src/hooks/useWebPush.ts
src/components/notifications/EnablePushNotificationsButton.tsx
src/components/notifications/NotificationPreferencesPanel.tsx
src/components/notifications/NotificationCenter.tsx
public/service-worker.js
```

### 32.10 Service Worker

Para Web Push funcionar no navegador, será necessário registrar um **Service Worker** no frontend.

Responsabilidades do Service Worker:

- receber push em background;
- exibir notificação nativa;
- tratar clique na notificação;
- abrir ou focar a aba do Portal Sama;
- redirecionar para a rota correta, quando permitido.

Exemplo de comportamento:

```txt
Usuário recebe push
        ↓
Clica na notificação
        ↓
Service Worker abre /notifications ou rota específica
        ↓
Frontend valida sessão
        ↓
Backend valida permissão ao carregar dados
```

### 32.11 VAPID keys

A implementação de Web Push normalmente exige chaves VAPID.

Regras:

```txt
VAPID_PUBLIC_KEY pode ir para o frontend.
VAPID_PRIVATE_KEY deve ficar somente no backend.
VAPID_PRIVATE_KEY deve ficar em variável de ambiente segura.
Nunca commitar chaves no repositório.
```

Variáveis sugeridas:

```env
WEB_PUSH_VAPID_PUBLIC_KEY="..."
WEB_PUSH_VAPID_PRIVATE_KEY="..."
WEB_PUSH_CONTACT="mailto:suporte@empresa.com"
```

### 32.12 Banco de dados proposto

#### `notification_preferences`

```txt
id
user_id
notification_type
channel
enabled
created_at
updated_at
```

Exemplo de `channel`:

```txt
IN_APP
EMAIL
WEB_PUSH
```

#### `web_push_subscriptions`

```txt
id
user_id
endpoint
p256dh_key
auth_key
user_agent
device_name
last_used_at
revoked_at
created_at
updated_at
```

Observação:

- `endpoint`, `p256dh_key` e `auth_key` devem ser tratados como dados sensíveis operacionais.
- avaliar criptografia em repouso para esses campos.

#### `notification_events`

```txt
id
user_id
notification_type
title
message
safe_payload
status
created_at
read_at
```

#### `notification_delivery_attempts`

```txt
id
notification_event_id
channel
provider
status
attempted_at
error_message
metadata
```

### 32.13 Endpoints internos sugeridos

#### Inscrever navegador/dispositivo

```txt
POST /api-v2/notifications/web-push/subscribe
```

Payload esperado:

```json
{
  "endpoint": "https://...",
  "keys": {
    "p256dh": "...",
    "auth": "..."
  },
  "deviceName": "Chrome - Notebook"
}
```

#### Remover inscrição

```txt
POST /api-v2/notifications/web-push/unsubscribe
```

#### Listar dispositivos inscritos

```txt
GET /api-v2/notifications/web-push/devices
```

#### Revogar dispositivo

```txt
DELETE /api-v2/notifications/web-push/devices/:id
```

#### Preferências de notificação

```txt
GET /api-v2/notifications/preferences
PATCH /api-v2/notifications/preferences
```

#### Listar notificações internas

```txt
GET /api-v2/notifications
```

#### Marcar como lida

```txt
PATCH /api-v2/notifications/:id/read
```

### 32.14 Fluxo de inscrição Web Push

```txt
Usuário acessa Portal Sama
        ↓
Frontend verifica suporte a notificações
        ↓
Usuário clica em Ativar notificações
        ↓
Navegador solicita permissão
        ↓
Usuário permite
        ↓
Frontend registra Service Worker
        ↓
Frontend cria Push Subscription
        ↓
Frontend envia subscription para API NestJS
        ↓
Backend salva subscription vinculada ao usuário
        ↓
Portal Sama passa a poder enviar Web Push para aquele navegador
```

### 32.15 Fluxo de envio

```txt
Evento ocorre no Portal Sama
        ↓
NotificationsService cria notification_event
        ↓
NotificationDispatcher verifica preferências
        ↓
Se Web Push estiver ativo, WebPushDispatcher busca subscriptions do usuário
        ↓
Backend envia Web Push
        ↓
Registra tentativa de entrega
        ↓
Se subscription inválida, marca como revogada/inativa
```

### 32.16 Eventos que devem disparar Web Push

Após a integração com Acessórias:

```txt
D-7 obrigação vencendo
D-3 obrigação vencendo
D-1 obrigação vencendo
D0 vencimento hoje
D+1 obrigação atrasada
Baixa pendente no Acessórias
Divergência Acessórias
Falha crítica de sincronização
```

Eventos gerais do Portal Sama:

```txt
Documento recusado
Documento aprovado
Nova tarefa atribuída
Solicitação de acesso aprovada
Solicitação de acesso recusada
Contrato aguardando assinatura
Proposta aguardando aprovação
Certificado digital próximo do vencimento
```

### 32.17 Regras anti-spam

Para evitar excesso de notificações:

```txt
Não enviar a mesma notificação mais de uma vez no mesmo nível de alerta.
Agrupar notificações quando houver muitas obrigações.
Permitir resumo diário para notificações menos críticas.
Não enviar push fora do horário configurado, exceto alertas críticos.
Permitir silenciar tipos específicos de notificação.
```

Exemplo de agrupamento:

```txt
Você possui 8 obrigações fiscais vencendo nos próximos 7 dias.
```

### 32.18 Auditoria

Registrar:

```txt
Usuário ativou Web Push
Usuário desativou Web Push
Dispositivo inscrito
Dispositivo revogado
Preferência alterada
Notificação criada
Web Push enviado
Web Push falhou
Subscription inválida removida
```

Nunca registrar conteúdo sensível da notificação em logs.

### 32.19 Frontend esperado

#### Botão de ativação

Adicionar componente:

```txt
Ativar notificações neste dispositivo
```

Estados:

```txt
Não suportado pelo navegador
Permissão ainda não solicitada
Permissão concedida
Permissão negada
Dispositivo já inscrito
Erro ao inscrever
```

#### Painel de preferências

Permitir ao usuário configurar:

```txt
Receber alertas de vencimento por Web Push
Receber alertas de atraso por Web Push
Receber alertas de documentos por Web Push
Receber alertas de contratos/propostas por Web Push
Receber alertas de solicitações de acesso por Web Push
```

#### Central de notificações

A Web Push deve sempre apontar para a Central de Notificações ou para a rota segura relacionada ao evento.

### 32.20 Testes recomendados

#### Testes unitários

- criação de preferência de notificação;
- validação de subscription;
- dispatcher escolhendo canais corretos;
- sanitização do conteúdo enviado no push;
- remoção de subscription inválida.

#### Testes de integração

- inscrição de dispositivo autenticado;
- revogação de dispositivo;
- envio de push para usuário com preferência ativa;
- não envio para usuário com preferência desativada;
- registro de tentativa de entrega;
- falha de provider sem quebrar fluxo principal.

#### Testes E2E

- usuário ativa notificações;
- usuário recebe aviso visual de permissão concedida;
- usuário altera preferências;
- usuário visualiza notificação na Central;
- clique na notificação abre rota segura.

#### Testes de segurança

- endpoint de subscribe exige autenticação;
- usuário não lista dispositivos de outro usuário;
- usuário não revoga dispositivo de outro usuário;
- payload do push não contém dados sensíveis;
- VAPID private key não aparece no frontend;
- falhas não expõem stack trace.

### 32.21 Critérios de aceite

A funcionalidade só deve ser considerada concluída se:

- Service Worker estiver registrado corretamente;
- usuário conseguir ativar/desativar Web Push;
- subscription for salva vinculada ao usuário autenticado;
- VAPID private key não for exposta;
- preferências por usuário funcionarem;
- notificações internas continuarem funcionando independentemente do Web Push;
- Web Push for enviado somente quando permitido;
- payload não contiver dados sensíveis;
- tentativas de entrega forem registradas;
- subscriptions inválidas forem tratadas;
- auditoria for registrada;
- testes principais passarem;
- documentação e backlog forem atualizados.

### 32.22 Registro sugerido no backlog

```md
## Web Push Notifications

- **Status:** Planejada para fase futura pós-Acessórias
- **Prioridade:** Média/Alta
- **Momento de implementação:** Após NotificationsModule, integração Acessórias e Central de Vencimentos
- **Descrição:** Permitir que notificações importantes do Portal Sama sejam enviadas como Web Push Notifications para o navegador do usuário, mesmo quando ele não estiver com a tela principal aberta.
- **Módulos afetados:** NotificationsModule, WebPushModule, AuditModule, AuthModule, UsersModule, DueDatesModule, AcessoriasDeliveriesModule.
- **Frontend necessário:** Service Worker, botão de ativação, painel de preferências, central de notificações, tratamento de permissão do navegador.
- **Backend necessário:** endpoints de subscribe/unsubscribe, dispatcher de notificações, armazenamento de subscriptions, envio Web Push com VAPID, auditoria e tratamento de falhas.
- **Banco de dados necessário:** notification_preferences, web_push_subscriptions, notification_events, notification_delivery_attempts.
- **Riscos de segurança:** exposição de dados sensíveis no push, exposição da VAPID private key, envio para dispositivo indevido, ausência de revogação, spam de notificações.
- **Dependências:** AuthModule, RBAC, AuditModule, NotificationsModule, frontend React base, configuração segura de .env.
- **Critério de início:** somente iniciar após a integração Acessórias e a Central de Vencimentos estarem estabilizadas.
```

---

## 33. Conclusão

A integração de entregas do Acessórias deve ser tratada como uma funcionalidade futura de alto valor operacional, mas não deve concorrer com o grande plano principal de migração e refatoração do Portal Sama.

O modelo aprovado é:

```txt
Acessórias = fonte oficial de entregas, baixas e vencimentos.
Portal Sama = camada operacional, visual, inteligente, auditável e resiliente.
```

Para o MVP futuro, o foco deve ser o departamento Fiscal, com geração automática de planilha por competência, sincronização automática de baixas, status visual **Acessórias**, Central de Vencimentos baseada no Acessórias e notificações iniciando 7 dias antes do vencimento.

Após essa etapa, deve ser planejada a implementação de **Web Push Notifications**, permitindo que usuários recebam notificações importantes do Portal Sama mesmo fora da tela principal da aplicação.

A principal decisão de planejamento é:

```txt
Não implementar agora.
Documentar como funcionalidade futura prioritária.
Executar somente depois da base principal estar finalizada.
Executar Web Push somente depois da integração Acessórias e da Central de Vencimentos estarem estabilizadas.
```

A principal regra de segurança operacional é:

```txt
O sistema só deve atualizar automaticamente uma célula quando conseguir identificar com segurança:
cliente + departamento + obrigação + competência.
```

Se essa identificação não for segura, o Portal Sama deve gerar divergência para análise do gestor, e não alterar a planilha automaticamente.

A principal regra de segurança para Web Push é:

```txt
Notificações Web Push nunca devem expor dados sensíveis; devem apenas avisar o usuário e direcioná-lo ao Portal Sama, onde autenticação e autorização serão validadas pelo backend.
```

# Acessórias verificação do código e se o uso dos endpoints está correto [Verificar]

Sim — **em geral vocês estão usando os endpoints certos para o escopo atual da aplicação**. O foco em **empresas/clientes + entregas/obrigações** está alinhado com o que a aplicação faz hoje e com o propósito do Acessórias como plataforma de gestão de prazos/processos e automação de entregas. ([Acessórias][1])

## O que está correto

| Item                                                                                                     | Avaliação                                                                                                      |
| -------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `Authorization: Bearer {token}`                                                                          | Correto. O código monta o header via `ACESSORIAS_AUTH_HEADER=Authorization` e `ACESSORIAS_AUTH_SCHEME=Bearer`. |
| `ACESSORIAS_BASE_URL + path`                                                                             | Correto, desde que `ACESSORIAS_BASE_URL` seja `https://api.acessorias.com`.                                    |
| `companies/ListAll`                                                                                      | Correto para sincronizar empresas/clientes.                                                                    |
| `companies/ListAll?obligations&departments&stateRegistrations&registrationData&contacts&Pagina={pagina}` | Correto para trazer dados completos de empresas, incluindo departamentos e contatos.                           |
| `deliveries/ListAll`                                                                                     | Correto para sincronizar entregas/obrigações.                                                                  |
| Paginação com `Pagina`                                                                                   | Correto. O código pagina até a API retornar lista vazia.                                                       |
| Rate limit `95/min`                                                                                      | Boa decisão. Fica abaixo do limite que havíamos considerado de `100/min`.                                      |
| Não usar escrita no Acessórias                                                                           | Correto para o escopo atual. O código consulta dados e grava apenas localmente.                                |
| Não usar `/requests`, `/invoices`, `/processes`, `/econtinuo`                                            | Correto. Esses endpoints fugiriam do escopo atual da aplicação.                                                |

## Endpoints que o código usa corretamente

```http
GET /companies/ListAll?obligations&departments&stateRegistrations&registrationData&contacts&Pagina={pagina}
```

Usado em `AcessoriasRegistrationsService` para importar clientes e extrair responsáveis/departamentos.

```http
GET /deliveries/ListAll?DtInitial={data}&DtFinal={data}&DtLastDH={data_hora}&situation=pending,delivered,read&config&Pagina={pagina}
```

Usado em `AcessoriasDeliveriesService` e `AcessoriasHomeService` para buscar entregas e obrigações.

## Pontos que eu ajustaria

### 1. Primeiro sync pode estar vindo incompleto

Hoje, quando **não existe sincronização anterior**, o código define:

```text
DtLastDH = ontem 00:00:00
```

Isso significa que o primeiro sync pode trazer **somente entregas alteradas desde ontem**, e não todas as entregas do período configurado.

O código também envia:

```text
DtInitial
DtFinal
DtLastDH
```

Se a API aplicar os filtros em conjunto, o resultado será:

```text
entregas dentro do período
E alteradas após DtLastDH
```

Para carga inicial, isso pode deixar muita coisa de fora.

**Ajuste recomendado:**

Na primeira sincronização, quando `incrementalSince` for `null`, eu removeria o `DtLastDH`.

Ficaria assim:

```http
GET /deliveries/ListAll?DtInitial={data}&DtFinal={data}&situation=pending,delivered,read&config&Pagina={pagina}
```

Depois da primeira sincronização, aí sim usar:

```http
GET /deliveries/ListAll?DtInitial={data}&DtFinal={data}&DtLastDH={ultima_sincronizacao}&situation=pending,delivered,read&config&Pagina={pagina}
```

### 2. Validar se `situation=read` é aceito

O código usa:

```text
situation=pending,delivered,read
```

Eu manteria `pending` e `delivered`. Mas validaria se `read` é oficialmente aceito pela API.

Se `read` não estiver documentado, melhor usar:

```text
situation=pending,delivered
```

ou remover `situation` e tratar status localmente.

### 3. Cuidado com marcação incremental por `finishedAt`

O código pega a última sincronização de sucesso usando `finishedAt` e manda isso como `DtLastDH`.

Isso funciona, mas tem um risco: se uma entrega for alterada **durante a execução da sincronização**, antes do `finishedAt`, ela pode não entrar na execução atual e também pode ser ignorada na próxima.

**Ajuste mais seguro:**

Usar uma janela com sobreposição:

```text
DtLastDH = ultima_sincronizacao - 5 minutos
```

ou salvar o horário de início da execução como referência incremental.

### 4. `config=true` vira `config=`

O código transforma booleano `true` em query vazia:

```http
config=
```

Normalmente isso funciona como flag de query string. Mas se a API exigir valor explícito, o ideal seria enviar:

```http
config=true
```

Eu só mudaria isso se a API retornar erro ou não trouxer a configuração da entrega.

## Veredito final

**Sim, vocês estão usando os endpoints corretos para a aplicação atual.**

O conjunto certo é:

```http
GET /companies/ListAll?obligations&departments&stateRegistrations&registrationData&contacts&Pagina={pagina}

GET /deliveries/ListAll?DtInitial={data}&DtFinal={data}&DtLastDH={data_hora}&situation=pending,delivered,read&config&Pagina={pagina}
```

Mas eu faria estes ajustes antes de considerar a integração “fechada”:

1. **Não enviar `DtLastDH` na primeira sincronização.**
2. **Validar se `situation=read` é aceito.**
3. **Usar sobreposição no incremental para evitar perda de alterações.**
4. **Confirmar se `config=` é aceito ou se precisa ser `config=true`.**

Fora isso, o código está bem alinhado ao escopo: **ler empresas e entregas do Acessórias, sincronizar localmente e aplicar essas informações no workspace operacional/fiscal da aplicação.**

[1]: "Acessórias - Automação e gestão online de prazos e processos para sua ..."
