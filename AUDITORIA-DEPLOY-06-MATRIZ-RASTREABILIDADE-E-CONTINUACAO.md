# Auditoria Deploy - Matriz de Rastreabilidade e Continuação

Data da continuação: 10/06/2026  
Documento base: `Testes-da-aplicação-DEPLOY.md`

## Objetivo desta continuação

Este arquivo complementa os relatórios `00` a `05` com uma matriz direta entre o que o roteiro de pré-produção pediu, o que foi executado, a evidência encontrada e o que ainda bloqueia uma homologação real.

## Matriz dos objetivos iniciais

| Objetivo do roteiro | Status | Evidência | Continuação necessária |
| --- | --- | --- | --- |
| Verificar viabilidade técnica | Atendido | API e web têm stacks coerentes, build/lint passam e health/login sobem localmente. | Repetir após correções críticas. |
| Verificar proximidade de uso real | Parcial | Sistema está próximo de homologação controlada, mas não de produção. | Corrigir escopo de clientes, E2E e infraestrutura real. |
| Identificar o que falta para produção | Atendido | Relatório `05` lista plano por prioridade. | Transformar plano em issues/tarefas e executar. |
| Classificar funcionalidades completas/incompletas/quebradas | Parcial | Relatório `01` traz matriz funcional por módulo. | Validar com usuários reais e banco real. |
| Identificar riscos de segurança, arquitetura, UX, integrações e operação | Atendido | Relatórios `02`, `03` e `04`. | Auditar novamente depois das correções. |
| Registrar testes executados e falhas | Atendido | Resumo executivo e relatórios técnicos registram comandos/resultados. | Guardar artefatos de Playwright/readiness em evidências de release. |
| Decidir se pode liberar usuários | Atendido | Decisão: não pronto para usuários reais. | Reavaliar após checklist Go/No-Go. |

## Matriz dos itens obrigatórios

| Exigência obrigatória | Status | Evidência/observação |
| --- | --- | --- |
| Subir back-end localmente | Atendido | API respondeu `GET /api-v2/health` com 200. |
| Subir front-end localmente | Atendido | Vite respondeu `/login` com 200. |
| Rodar build | Atendido | `npm.cmd run build` passou em API e web. |
| Rodar lint | Atendido | `npm.cmd run lint` passou em API e web. |
| Rodar testes existentes | Atendido com falhas | Unit/contract passaram; E2E API e web falharam. |
| Criar ou executar testes E2E | Atendido | `npm.cmd run test:e2e` executado em API e web. |
| Testar como usuário real | Bloqueado | Sem banco/credenciais reais; só sessões mockadas e testes API com tokens simulados. |
| Testar perfis DEV, Gestor e Colaborador | Parcial | Perfil Admin/colaborador em Playwright mockado; perfis reais não homologados. |
| Testar autenticação | Parcial | Código/testes cobrem auth; login real não executado. |
| Testar autorização/permissões | Parcial com reprovação | Foi encontrado vazamento de escopo em `clients.read`. |
| Testar rotas protegidas | Parcial | E2E API cobre 401/403 em várias rotas; homologação web real pendente. |
| Testar navegação | Parcial | Playwright percorreu rotas mockadas; fluxo autenticado real pendente. |
| Testar formulários | Parcial | Cobertura indireta; rodada manual real pendente. |
| Testar documentos | Parcial | API cobre regras e upload; ClamAV strict e navegador real pendentes. |
| Testar integrações | Parcial | Acessórias/Integra-AI têm testes; integrações reais pendentes. |
| Testar casos felizes e falhas | Parcial | E2E e unitários cobrem ambos; cenários reais ainda não. |
| Capturar screenshots/traces | Atendido | Playwright gerou screenshots/traces nas falhas web. |
| Capturar console/rede/API | Parcial | Logs de execução e erros de proxy/E2E foram observados; precisa pacote formal de evidências em homologação. |
| Registrar evidências no relatório final | Atendido | Relatórios `00` a `06`. |

## Rastreabilidade por etapa

| Etapa do roteiro | Status da análise | Evidência principal | Lacuna |
| --- | --- | --- | --- |
| Etapa 1 - Entendimento | Atendida | Docs ativos `00` a `07` lidos e consolidados. | Documentos históricos seguem fragmentados em `docs/_arquivo`. |
| Etapa 2 - Estrutura do projeto | Atendida | API, web e docs mapeados. | Gerar documentação final de arquitetura viva. |
| Etapa 3 - Execução técnica | Atendida com bloqueio externo | Build/lint/testes/servidores executados. | Banco MySQL alvo indisponível. |
| Etapa 4 - QA funcional | Parcial | Matriz funcional criada. | Homologação com usuários reais pendente. |
| Etapa 5 - Segurança | Atendida com reprovação | `.env` em ZIP e escopo de clientes. | Corrigir e repetir audit. |
| Etapa 6 - UI/UX | Parcial | Screenshots Playwright avaliadas; E2E web falhou em Home. | Mobile real/acessibilidade/dados longos pendentes. |
| Etapa 7 - API | Atendida com falha | E2E API falhou em 2 testes Web Push/CSRF. | Isolar `.env` dos testes e repetir. |
| Etapa 8 - Front-end | Atendida com falha | Build/lint/contract passaram; Playwright teve 2 falhas. | Decidir contrato textual da Home. |
| Etapa 9 - Back-end | Atendida | Unitários passaram; módulos analisados. | Corrigir escopo de clientes e validar DB real. |
| Etapa 10 - Build/produção | Parcial | Build passou; readiness falhou em DB/ClamAV. | EasyPanel/DB/HTTPS/ClamAV/backup reais. |
| Etapa 11 - Plano de testes | Atendida | Relatório `01` e checklist `05`. | Executar plano em homologação. |
| Etapa 12 - Relatório final | Atendida | Relatórios `00` a `06`. | Atualizar após correções. |

## Continuação técnica recomendada

### Bloco A - Correções que destravam análise

1. Remover `.env` do ZIP e rotacionar segredos.
2. Corrigir escopo de `ClientsService`.
3. Ajustar ambiente dos testes E2E da API para não herdar `COOKIE_DOMAIN` do `.env`.
4. Atualizar contrato dos testes Playwright da Home.
5. Atualizar dependência vulnerável `qs`.

### Bloco B - Reexecução local

Comandos mínimos após correções:

```powershell
cd C:\Users\Sama Contabilidade\Desktop\portal-sama-api
npm.cmd audit --audit-level=moderate
npm.cmd run prisma:validate
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
npm.cmd run test:e2e

cd C:\Users\Sama Contabilidade\Desktop\portal-sama-web
npm.cmd audit --audit-level=moderate
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
npm.cmd run test:e2e
```

### Bloco C - Homologação real

Ambiente necessário:

- MySQL acessível.
- Variáveis reais de homologação sem segredos expostos em repositório ou ZIP.
- HTTPS/domínio real.
- Usuários reais DEV, Gestor, Colaborador e Cliente.
- VAPID real para Web Push.
- ClamAV instalado e `SAMA_UPLOAD_SCAN_MODE=strict`.
- Token Acessórias real.
- Rotina de backup/restore disponível.

Comandos de corte técnico:

```powershell
cd C:\Users\Sama Contabilidade\Desktop\portal-sama-api
npm.cmd run prisma:migrate:status
npm.cmd run prisma:migrate:deploy
npm.cmd run prisma:seed
npm.cmd run ops:readiness
npm.cmd run ops:backup:create
npm.cmd run ops:backup:verify
npm.cmd run ops:restore:drill
```

## Casos de teste que devem ser adicionados

| Área | Teste novo | Severidade que cobre |
| --- | --- | --- |
| Clientes/RBAC | `DEPARTMENT` não lista cliente sem atribuição/departamento permitido. | Crítica |
| Clientes/RBAC | `ACCOUNTING` não acessa dashboard de cliente fora do escopo contábil. | Crítica |
| Clientes/RBAC | `LEGALIZATION` não acessa detalhe de cliente fora do escopo. | Crítica |
| Clientes/RBAC | `CLIENT` não acessa outro cliente por `id`. | Crítica |
| API E2E | Teste força `COOKIE_DOMAIN=''` e confirma CSRF válido em ambiente local. | Alta |
| API E2E | Teste confirma que CSRF sem cookie/header continua 403. | Alta |
| Web E2E | Home Admin valida textos atuais de Acessórias ou contrato revisado. | Alta |
| Web E2E | Home colaborador valida `Baixas sincronizadas` se esse for o contrato final. | Alta |
| Upload | EICAR é bloqueado em modo strict. | Alta |
| Operação | Readiness falha se `CERTIFICATE_ENCRYPTION_KEY` estiver ausente em produção. | Média |
| Artefatos | Script de empacotamento falha se ZIP contiver `.env`. | Crítica |

## Critério para mudar a decisão

A decisão pode sair de **não pronto para usuários reais** para **pronto apenas para homologação** quando:

- SEC-01 e SEC-02 forem corrigidos.
- API E2E e web E2E ficarem verdes.
- `npm audit --audit-level=moderate` ficar verde na API.
- Readiness em homologação passar pelo menos em ambiente, banco, storage, migrations e RBAC.

A decisão pode sair para **pronto para produção** somente quando, além disso:

- ClamAV strict estiver validado.
- Backup/restore drill estiver documentado.
- Web Push real estiver validado.
- Acessórias real estiver validado.
- Homologação manual por perfil estiver aprovada.
- Não houver vazamento de escopo em dados de clientes/documentos.

## Conclusão da continuação

A análise do roteiro está coberta em nível de pré-produção, mas a aplicação ainda precisa de uma rodada de correção e nova execução dos gates. O próximo passo técnico mais eficiente é corrigir os dois bloqueios críticos: **segredos empacotados em ZIP** e **escopo de clientes**.

