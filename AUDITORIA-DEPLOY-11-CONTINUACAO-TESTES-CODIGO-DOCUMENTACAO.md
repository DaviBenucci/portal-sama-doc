# Auditoria Deploy - Continuação de Testes, Código e Documentação

Data da continuação: 10/06/2026  
Escopo: `portal-sama-docs`, `portal-sama-api`, `portal-sama-web` e artefatos ZIP locais.  
Base: `Testes-da-aplicação-DEPLOY.md` e relatórios `AUDITORIA-DEPLOY-00` a `AUDITORIA-DEPLOY-10`.

## Veredito desta rodada

**Permanece No-Go para produção e para usuários reais.**

A aplicação continua com boa base técnica local: lint, build, testes unitários da API, testes contratuais do Web e schema Prisma passam. Porém os bloqueios centrais da auditoria anterior continuam presentes:

- ZIP da API ainda contém `.env`.
- `ClientsService` ainda não restringe adequadamente `clients.read` por escopo real de cliente/departamento/responsável.
- Home do Acessórias ainda aceita `profile` vindo do cliente e permite caminho de escalada para visão `admin`.
- E2E API e Web continuam vermelhos.
- Banco/migrations/RBAC/bootstrap ainda não foram comprovados em MySQL acessível.
- ClamAV strict, Web Push real, backup/restore drill e homologação com usuários reais continuam pendentes.

## Evidências executadas nesta continuação

| Área | Comando/validação | Resultado |
| --- | --- | --- |
| Ambiente | `node -v` | `v24.15.0` |
| Docs Git | `git status --short` em `portal-sama-docs` | Há relatórios alterados/não rastreados anteriores; esta rodada criou arquivo novo para não sobrescrever. |
| API Git | `git status --short` em `portal-sama-api` | Limpo antes da análise. |
| Web Git | `git status --short` em `portal-sama-web` | Limpo antes da análise. |
| Artefatos | `tar -tf portal-sama-api.zip` filtrando `.env` | **Falhou:** contém `portal-sama-api/.env` e `.env.example`. |
| Artefatos | `tar -tf portal-sama-web.zip` filtrando `.env` | OK: contém apenas `.env.example`. |
| Artefatos | `tar -tf portal-sama-docs.zip` filtrando `.env` | OK: não retornou `.env`. |
| API | `npm.cmd run prisma:validate` | Passou; schema Prisma válido. O Prisma carregou `.env`. |
| API | `npm.cmd run lint` | Passou. |
| API | `npm.cmd run build` | Passou. |
| API | `npm.cmd test -- --runInBand` | Passou: 44 suites, 264 testes. |
| API focado | `npm.cmd test -- --runInBand src/modules/clients/clients.service.spec.ts src/modules/integrations/acessorias/acessorias-home.service.spec.ts src/config/env.schema.spec.ts` | Passou: 3 suites, 13 testes. |
| API E2E | `npm.cmd run test:e2e` | **Falhou:** 134 passaram, 2 falharam em Web Push/CSRF com `403` onde esperava `200`. |
| API audit | `npm.cmd audit --audit-level=moderate` | **Falhou:** 1 vulnerabilidade moderada em `qs`. |
| API DB | `npm.cmd run prisma:migrate:status` | **Falhou:** schema engine; MySQL `portal-sama_portal-sama-database:3306` indisponível localmente. |
| API readiness | `node -r dotenv/config scripts/validate-operational-readiness.js --soft` | **Falhou:** banco não respondeu; avisos de scan mode, ClamAV e backup/rollback. |
| API readiness isolado | `npm.cmd run ops:readiness -- --skip-env --skip-database --skip-clamav --storage-path .ai-tests/readiness-storage --soft` | Passou com aviso de backup/rollback; valida apenas storage local isolado. |
| Web | `npm.cmd run lint` | Passou. |
| Web | `npm.cmd run build` | Passou. |
| Web | `npm.cmd test -- --runInBand` | Passou: 9 testes contratuais. |
| Web E2E | `npm.cmd run test:e2e` | **Falhou:** 11 passaram, 1 skip, 2 falharam na Home. |
| Web audit | `npm.cmd audit --audit-level=moderate` | Passou: 0 vulnerabilidades. |

## Achados confirmados

### SEC-01 - ZIP da API ainda empacota `.env`

Severidade: **Crítico**.

Evidência:

- `C:\Users\Sama Contabilidade\Desktop\portal-sama-api.zip` lista `portal-sama-api/.env`.
- O arquivo local `portal-sama-api/.env` existe fora do Git.

Impacto: qualquer compartilhamento do ZIP pode vazar segredos de ambiente. Mesmo que as credenciais sejam temporárias, a ação segura é tratar como segredo exposto.

Correção obrigatória:

1. Remover ou substituir o ZIP atual.
2. Gerar novo artefato excluindo `.env`, `.env.local`, logs, backups, dumps e diretórios de teste.
3. Rotacionar segredos que possam ter sido incluídos no pacote.
4. Adicionar checagem automatizada de empacotamento que falhe se `.env` aparecer no ZIP.

### AUTHZ-01 - `clients.read` continua amplo demais

Severidade: **Crítico**.

Evidência:

- `portal-sama-api/src/modules/clients/clients.service.ts:33` chama `applyReadScope()`.
- `portal-sama-api/src/modules/clients/clients.service.ts:383` a `:402` aplica escopo apenas para role `CLIENT`; perfis internos não privilegiados retornam `where` sem restrição.
- `portal-sama-api/src/modules/clients/clients.service.ts:418` a `:439` permite `getById`/dashboard para qualquer usuário com `clients.read`.
- `portal-sama-api/src/modules/clients/clients.service.spec.ts` não contém teste negativo de `DEPARTMENT`, `ACCOUNTING`, `LEGALIZATION` ou colaborador fora do escopo tentando listar/acessar cliente alheio.

Impacto: perfis internos com `clients.read` podem listar ou acessar cadastro/dashboard de clientes fora do departamento, da responsabilidade ou do vínculo esperado. Para um sistema de documentos empresariais, isso sustenta risco de vazamento operacional e prepara caminho para IDOR em telas dependentes de cliente.

Correção obrigatória:

1. Definir a regra oficial de escopo por role/perfil.
2. Aplicar filtro em `list()` usando `ClientDepartmentAssignment`, departamento permitido, responsável ou vínculo de cliente.
3. Substituir o atalho `user.permissions.includes('clients.read')` em `assertCanReadClient()` por checagem de escopo real.
4. Adicionar testes negativos e positivos para `CLIENT`, `DEPARTMENT`, `ACCOUNTING`, `LEGALIZATION`, `MANAGER`, `ADMIN` e `DEV`.

### AUTHZ-03 - Home Acessórias ainda confia em `profile` da query

Severidade: **Crítico**.

Evidência:

- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.controller.ts:20` a `:23` expõe `GET /integrations/acessorias/home-summary` recebendo `@Query('profile')`.
- A rota não possui `@Permissions(...)`, então basta autenticação JWT para entrar.
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.service.ts:89` a `:95` não aplica filtro departamental quando `profile === 'admin'`.
- `portal-sama-api/src/modules/integrations/acessorias/acessorias-home.service.ts:200` a `:203` retorna todos os itens quando `profile === 'admin'`.
- `portal-sama-web/src/services/acessorias.service.ts:25` a `:29` envia `profile` como parâmetro.
- `portal-sama-web/src/pages/home/HomePage.tsx:49` a `:70` deriva o perfil no client, mas isso é UX, não autorização.

Impacto: um usuário autenticado pode tentar forçar manualmente `?profile=admin` para receber visão global de entregas Acessórias, com cliente, documento, obrigação, departamento, responsável e vencimento.

Correção obrigatória:

1. Remover confiança no `profile` vindo do cliente.
2. Derivar perfil no backend a partir de roles/permissões do usuário autenticado.
3. Exigir role `ADMIN`/`DEV` ou permissão explícita para visão global.
4. Adicionar E2E negativo: `CLIENT`, `DEPARTMENT`, `ACCOUNTING`, `LEGALIZATION` e `MANAGER` sem privilégio não podem obter visão `admin`.

### QA-01 - E2E API ainda falha em Web Push/CSRF

Severidade: **Alta**.

Evidência:

- `npm.cmd run test:e2e` na API falhou em:
  - `/api-v2/notifications/push/subscribe (POST) accepts valid CSRF...`
  - `/api-v2/notifications/push/test (POST) accepts valid CSRF...`
- Ambos esperavam `200`, mas receberam `403`.

Impacto: o gate automatizado da API não fecha. A causa provável continua sendo configuração de cookie/CSRF herdada do `.env` local, mas enquanto a suíte falha, o release não deve avançar.

Correção obrigatória:

1. Isolar `.env.test` dos E2E.
2. Forçar `COOKIE_DOMAIN=''` e `COOKIE_SECURE=false` no setup local de teste.
3. Manter produção com domínio real, `Secure`, `HttpOnly` e `SameSite` adequados.

### QA-02 - E2E Web ainda falha na Home

Severidade: **Alta**.

Evidência:

- `npm.cmd run test:e2e` no Web falhou em:
  - `home page renders authenticated shortcuts from session permissions`: não encontrou texto `Integracao`.
  - `home page renders collaborator daily priorities from available data`: não encontrou heading `Entregue pelo Acessorias`.
- O Playwright gerou screenshots, vídeos e traces em `portal-sama-web/test-results/...`.
- Houve avisos recorrentes de proxy para `/api-v2/notifications/stream?take=20` com `ECONNREFUSED`, pois a API real não estava ativa para essas chamadas não mockadas.

Impacto: há desalinhamento entre contrato E2E e UI atual da Home, além de ruído de rede em notificações/SSE durante os testes. O gate visual/funcional não está verde.

Correção obrigatória:

1. Decidir se os textos esperados são contrato de produto ou se os testes estão defasados.
2. Mockar ou neutralizar SSE/notificações nos testes de Home para evitar `ECONNREFUSED`.
3. Reexecutar Playwright até 100% verde.

### OPS-01 - Banco, migrations, RBAC e bootstrap seguem sem prova real

Severidade: **Alta**.

Evidência:

- `npm.cmd run prisma:migrate:status` não conseguiu validar o banco.
- `node -r dotenv/config scripts/validate-operational-readiness.js --soft` falhou em `database`.
- Checks de schema, migrations, RBAC e usuários privilegiados foram pulados porque o banco não respondeu.

Impacto: ainda não existe evidência de que o banco alvo tenha migrations aplicadas, seed RBAC, usuário `ADMIN`/`DEV` válido, login real e readiness completo.

Correção obrigatória:

1. Rodar `prisma:migrate:status`, `migrate:deploy`, `prisma:seed`, `prisma:bootstrap-admin` e `ops:readiness` no alvo de homologação/EasyPanel.
2. Registrar saídas sanitizadas no repositório de documentação.
3. Só considerar homologação com usuários reais depois dessa prova.

### DOC-01 - READMEs de API/Web apontam para caminho documental incompatível com o workspace atual

Severidade: **Média**.

Evidência:

- `portal-sama-api/README.md:7` aponta para `../portal-sama-doc`.
- `portal-sama-web/README.md:7` aponta para `../portal-sama-doc`.
- No Desktop atual, `C:\Users\Sama Contabilidade\Desktop\portal-sama-doc` não existe; `portal-sama-docs` existe.
- Os READMEs também citam `CORRECAO_MIGRATIONS_EASYPANEL_PORTAL_SAMA.md` e `ANALISE_DUPLICIDADES_UX_ACESSORIAS_PORTAL_SAMA_2026_06_05.md`, mas esses arquivos não estão na raiz atual de `portal-sama-docs`.
- A documentação histórica explica que `portal-sama-doc` pode ser alias remoto e `portal-sama-docs` o workspace local, mas os READMEs técnicos não deixam isso claro.

Impacto: uma pessoa ou IA que abrir API/Web primeiro pode seguir caminho inexistente, ignorar a documentação ativa ou concluir falsamente que docs obrigatórios estão ausentes.

Correção recomendada:

1. Atualizar READMEs para usar `../portal-sama-docs` como caminho local principal.
2. Explicar que `portal-sama-doc` é alias remoto/Bitbucket, quando aplicável.
3. Trocar referências a documentos ausentes por arquivos ativos equivalentes, por exemplo `AUDITORIA-DEPLOY-05-PLANO-CORRECAO-CHECKLIST-PRODUCAO.md` e relatórios `AUDITORIA-DEPLOY-*`, ou restaurar os arquivos citados se ainda forem oficiais.

## O que permanece positivo

| Área | Evidência | Leitura |
| --- | --- | --- |
| API unitária | 44 suites e 264 testes passaram. | Boa base de regressão interna. |
| API lint/build | Passaram. | Código compila e padrão está estável. |
| Prisma schema | `prisma validate` passou. | Schema versionado é sintaticamente válido. |
| Web lint/build | Passaram. | Front compila para produção. |
| Web contratos | 9 testes passaram. | Contratos de notificações/Web Push/client bundle continuam protegidos. |
| Web audit | 0 vulnerabilidades. | Sem achados npm moderados ou superiores nesta rodada. |
| Storage local isolado | Readiness com `--skip-database` validou escrita/leitura. | Storage local funciona, mas não substitui storage real. |

## Matriz de decisão atualizada

| Critério | Status | Motivo |
| --- | --- | --- |
| Build API/Web | OK | Ambos passam. |
| Lint API/Web | OK | Ambos passam. |
| Unit/contract | OK | API e Web passam nas suítes não E2E. |
| E2E API | Crítico | 2 falhas Web Push/CSRF. |
| E2E Web | Crítico | 2 falhas na Home. |
| Dependências API | Pendente | `qs` vulnerável em nível moderado. |
| Segredos/artefatos | Crítico | `.env` dentro de ZIP da API. |
| Autorização de clientes | Crítico | `clients.read` sem escopo suficiente. |
| Acessórias Home | Crítico | `profile=admin` controlável pelo cliente. |
| Banco/migrations/RBAC | Crítico | Não comprovado em DB real. |
| ClamAV strict | Pendente | Scanner não disponível localmente. |
| Web Push real | Pendente | Sem VAPID/navegador real autenticado. |
| Homologação por perfil | Pendente | Sem usuários reais DEV/Gestor/Colaborador/Cliente. |

## Próxima sequência recomendada

1. Eliminar o ZIP contaminado, recriar artefatos sem `.env` e rotacionar segredos potencialmente expostos.
2. Corrigir `ClientsService` para aplicar escopo real em listagem, detalhe e dashboard.
3. Corrigir `home-summary` para derivar perfil no backend e bloquear visão `admin` para perfis não autorizados.
4. Adicionar testes negativos para os dois pontos de autorização.
5. Isolar configuração E2E da API para fechar Web Push/CSRF.
6. Ajustar contrato ou UI da Home e neutralizar SSE não mockado no Playwright.
7. Atualizar READMEs de API/Web para `portal-sama-docs` e documentos realmente existentes.
8. Reexecutar `audit`, `lint`, `build`, unitários, contratos, E2E API e E2E Web.
9. Só então executar homologação real com banco, migrations, seed, bootstrap admin, ClamAV strict, Web Push real e usuários por perfil.

## Conclusão

A continuação confirma a avaliação dos relatórios anteriores: o Portal Sama está tecnicamente próximo de uma homologação controlada, mas ainda não está pronto para usuários reais. O risco mais importante segue sendo autorização e exposição de dados/documentos entre escopos, agravado pelo artefato `.env` no ZIP da API.

