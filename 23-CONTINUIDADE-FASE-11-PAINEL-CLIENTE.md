# Continuidade Fase 11 - Painel do cliente final

Atualizado em: 2026-06-30
Ambiente alvo: `https://portal.samacontabil.com.br`
Status formal: Fase 11 `CONCLUIDA`
Etapa atual: Fase 11 concluida; proximo passo Fase 12.1 - Refinar tela de Clientes

## 1. Entrada da fase

A Fase 11 foi iniciada depois da Fase 10 ficar formalmente `CONCLUIDA`.

Evidencia de entrada:

```txt
22-CONTINUIDADE-FASE-10-FRONTEND.md
Fase 10 CONCLUIDA
lint OK
21 web contract tests passed
build OK
test:e2e OK
git diff --check OK
```

Escopo da Fase 11 conforme o guia raiz:

```txt
Fase 11 - Painel do cliente final reaproveitando o legado
```

## 2. Etapa 11.1 - Criar ClientDashboardHeader

Status: `CONCLUIDA`

Arquivos alterados:

```txt
portal-sama-web/src/pages/clients/components/ClientDashboardHeader.tsx
portal-sama-web/src/pages/clients/ClientDashboardPage.tsx
portal-sama-web/src/pages/clients/ClientDocumentsPanel.tsx
portal-sama-web/src/pages/clients/ClientsPage.tsx
portal-sama-web/src/index.css
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- criado `ClientDashboardHeader` com voltar, nome fantasia/razao social, CNPJ formatado, ID da empresa, regime, atividade, contato, endereco, rank e status;
- criado `ClientDashboardQuickActions` com atalhos para `#client-assignments` e `#client-documents`;
- o seletor de status aparece somente para usuario com `clients.update` e usa `updateClient` com CSRF pelo service existente;
- a tela `/clientes/:clientId/painel` passou a usar o header extraido e preserva a origem de volta quando aberta pela lista de clientes;
- `ClientsPage` envia `state.from` com pathname e query string para manter filtros ao voltar;
- `ClientAssignmentsPanel` e `ClientDocumentsPanel` receberam ids estaveis para ancoras;
- CSS responsivo do header foi adicionado para desktop, tablet e mobile;
- contrato web novo cobre a existencia do header, dados principais, status, quick actions, estado de volta e ids de ancora;
- E2E focado abre o painel com dados mockados, valida identidade/rank/CNPJ/acoes e troca o status via `PATCH /clients/client-1`.

## 3. Validacoes executadas

Comandos executados em `portal-sama-web`:

```powershell
npm.cmd run lint
npm.cmd test -- --runInBand
npm.cmd run build
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "client dashboard header renders identity and updates status"
npm.cmd run test:e2e
git diff --check
```

Resultado:

```txt
lint OK
22 web contract tests passed
build OK
Playwright focado OK, 1 passed
test:e2e OK, 28 passed, 2 skipped
git diff --check OK no web e no docs
```

## 4. Etapa 11.2 - Criar ClientDashboardTabs

Status: `CONCLUIDA`

Arquivos alterados:

```txt
portal-sama-web/src/pages/clients/components/ClientDashboardTabs.tsx
portal-sama-web/src/pages/clients/components/ClientDashboardHeader.tsx
portal-sama-web/src/pages/clients/ClientDashboardPage.tsx
portal-sama-web/src/index.css
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- criado `ClientDashboardTabs` usando `QueryTabs` da Fase 10;
- abas criadas: `Geral`, `Responsaveis`, `Acessos`, `Documentos`, `Certificados` e `Vida da empresa`;
- query string `?tab=...` controla a aba ativa;
- aba `Geral` manteve os indicadores e o resumo operacional;
- aba `Responsaveis` renderiza o painel de equipe/responsaveis;
- aba `Documentos` renderiza o painel documental no contexto do cliente;
- abas `Acessos`, `Certificados` e `Vida da empresa` ficaram reservadas para as proximas etapas da Fase 11;
- quick actions do header passaram a apontar para `?tab=responsaveis#client-assignments` e `?tab=documentos#client-documents`;
- E2E focado valida abas, URL com `tab=documentos` e painel de documentos renderizado.

## 5. Validacoes executadas apos a Etapa 11.2

Comandos executados em `portal-sama-web`:

```powershell
npm.cmd run lint
npm.cmd test -- --runInBand
npm.cmd run build
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "client dashboard header renders identity and updates status"
npm.cmd run test:e2e
```

Resultado:

```txt
lint OK
22 web contract tests passed
build OK
Playwright focado OK, 1 passed
test:e2e OK, 28 passed, 2 skipped
```

## 6. Etapa 11.3 - Implementar aba Geral

Status: `CONCLUIDA`

Arquivos alterados:

```txt
portal-sama-web/src/pages/clients/ClientDashboardPage.tsx
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- aba `Geral` passou a exibir dados completos da empresa;
- leitura cobre razao social, nome fantasia, CNPJ, status, rank, ID da empresa, regime, atividade, contato, endereco, cidade/UF, grupo, criado em e atualizado em;
- edicao rapida usa `clientFormSchema`, `react-hook-form`, `zodResolver` e `updateClient`;
- botao `Editar dados` aparece somente quando a sessao tem `clients.update`;
- usuario sem `clients.update` ve aviso de consulta sem edicao e nao recebe campos editaveis;
- apos salvar, a tela invalida `client-dashboard` e `clients`;
- E2E cobre edicao de `nomeFantasia`, payload enviado para `PATCH /clients/client-1` e ausencia de edicao para usuario sem permissao.

## 7. Validacoes executadas apos a Etapa 11.3

Comandos executados em `portal-sama-web`:

```powershell
npm.cmd run lint
npm.cmd test -- --runInBand
npm.cmd run build
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "client dashboard"
npm.cmd run test:e2e
```

Resultado:

```txt
lint OK
22 web contract tests passed
build OK
Playwright focado OK, 2 passed
test:e2e OK, 29 passed, 2 skipped
```

## 8. Etapa 11.4 - Implementar aba Responsaveis

Status: `CONCLUIDA`

Arquivos alterados:

```txt
portal-sama-web/src/pages/clients/ClientDashboardPage.tsx
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- aba `Responsaveis` confirmada como painel operacional de equipe/responsaveis do cliente;
- fluxo cobre listagem de responsabilidades normalizadas, contador de vinculos e contador de responsabilidades ativas;
- criacao usa `createClientAssignment` com departamento, responsavel operacional, gestor, tipo e inicio;
- edicao usa `updateClientAssignment` com departamento, responsavel, gestor, tipo, status, inicio e fim;
- transferencia usa `transferClientAssignments` com novo responsavel, gestor, data efetiva, motivo e metadata de origem `client_dashboard`;
- encerramento usa `endClientAssignment` com data de encerramento e motivo;
- acoes respeitam permissoes `client_assignments.read`, `client_assignments.create`, `client_assignments.update`, `client_assignments.transfer`, `client_assignments.end`, alem de `departments.read` e `collaborators.read` para carregar opcoes;
- apos criar, editar, transferir ou encerrar, a tela invalida `client-assignments` e tambem `client-dashboard`, mantendo header/resumo sincronizados;
- contrato web passou a proteger os fluxos principais da aba;
- E2E focado cobre criar, editar, transferir e encerrar responsabilidade com payloads mockados.

## 9. Validacoes executadas apos a Etapa 11.4

Comandos executados em `portal-sama-web`:

```powershell
npm.cmd run lint
npm.cmd test -- --runInBand
npm.cmd run build
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "client dashboard"
npm.cmd run test:e2e
```

Resultado:

```txt
lint OK
22 web contract tests passed
build OK
Playwright focado OK, 3 passed
test:e2e OK, 30 passed, 2 skipped
```

## 10. Etapa 11.5 - Implementar aba Acessos

Status: `CONCLUIDA`

Arquivos alterados:

```txt
portal-sama-web/src/types/clients.ts
portal-sama-web/src/services/clients.service.ts
portal-sama-web/src/pages/clients/ClientAccessesTab.tsx
portal-sama-web/src/pages/clients/ClientDashboardPage.tsx
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- criada `ClientAccessesTab` para a aba `Acessos`;
- adicionados tipos frontend para `ClientAccessCredentialItem`, listagem, resposta e payload;
- `clients.service.ts` passou a consumir `GET /clients/:id/access-credentials`, `POST /clients/:id/access-credentials`, `PATCH /client-access-credentials/:id` e `DELETE /client-access-credentials/:id`;
- listagem exibe rotulo, sistema, URL, login, status, departamento, dica, notas, criado em, atualizado em e ultima rotacao;
- segredo nunca e exibido na listagem; a tela mostra apenas estado `Protegido` ou `Nao cadastrado`;
- formulario de criacao/edicao aceita referencia segura (`secretRef`) e dica (`secretHint`), sem campo de senha em texto puro;
- edicoes comuns nao enviam `secretRef` vazio para evitar limpar/rotacionar segredo por acidente;
- acoes de novo acesso, edicao e arquivamento aparecem somente para usuario com `client_accesses.manage`;
- leitura depende de `client_accesses.read`;
- indicador de segredo auditado considera `client_accesses.reveal_sensitive` e `sensitive.reveal`, mas a UI nao revela segredo porque a API atual ainda nao expoe endpoint auditado de revelacao;
- apos criar, editar ou arquivar, a tela invalida `client-access-credentials` e `client-dashboard`;
- E2E cobre criacao, edicao e arquivamento e confirma que payloads nao enviam `secretCipher`/`secret_cipher`.

Observacao tecnica:

```txt
A Etapa 11.5 foi concluida para a superficie frontend disponivel hoje: listagem e gestao segura sem segredo em claro.
Se o negocio exigir botao de revelar/copiar segredo, a API precisa expor endpoint proprio com auditoria/rate limit antes de a UI oferecer essa acao.
```

## 11. Validacoes executadas apos a Etapa 11.5

Comandos executados em `portal-sama-web`:

```powershell
npm.cmd run lint
npm.cmd test -- --runInBand
npm.cmd run build
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "client dashboard"
npm.cmd run test:e2e
```

Resultado:

```txt
lint OK
22 web contract tests passed
build OK
Playwright focado OK, 4 passed
test:e2e OK, 31 passed, 2 skipped
```

## 12. Etapa 11.6 - Implementar aba Documentos

Status: `CONCLUIDA`

Arquivos alterados:

```txt
portal-sama-web/src/components/ui/QueryTabs.tsx
portal-sama-web/src/pages/clients/components/ClientDashboardTabs.tsx
portal-sama-web/src/pages/clients/ClientDashboardPage.tsx
portal-sama-web/src/pages/clients/ClientDocumentsPanel.tsx
portal-sama-web/src/services/documents.service.ts
portal-sama-web/src/types/documents.ts
portal-sama-web/src/index.css
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- `QueryTabs` passou a aceitar badge por aba, usado pela aba `Documentos`;
- `ClientDashboardPage` passou a reutilizar `GET /clients/:id/documents` para calcular pendencias obrigatorias do checklist e alimentar o badge da aba;
- a aba `Documentos` passou a destacar requisitos pendentes, separando obrigatorios, pendentes, enviados e aprovados;
- upload no contexto do cliente permanece usando `POST /documents/clients/:clientId/upload` com CSRF pelo service existente;
- criacao de requisito documental permanece usando `POST /documents/custom-requirements`;
- revisao documental permanece usando `PATCH /documents/:id/status`;
- download protegido permanece usando `GET /documents/:id/download`, sem expor storage key;
- adicionados tipos e service frontend para `GET /documents/:id/status-history`;
- a aba passou a exibir historico de status por documento com usuario, data, transicao de status e motivo;
- mutacoes de upload, requisito e revisao invalidam `client-documents`, `client-dashboard` e historico documental;
- E2E focado cobre badge de pendencia, lista de requisitos, upload, criacao de requisito, revisao e historico.

## 13. Validacoes executadas apos a Etapa 11.6

Comandos executados em `portal-sama-web`:

```powershell
npm.cmd run lint
npm.cmd test -- --runInBand
npm.cmd run build
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "client dashboard documentos" --reporter=line
npm.cmd run test:e2e
git diff --check
```

Resultado:

```txt
lint OK
22 web contract tests passed
build OK
Playwright focado OK, 1 passed
test:e2e OK, 32 passed, 2 skipped
git diff --check OK no web e no docs
```

## 14. Etapa 11.7 - Implementar aba Certificados

Status: `CONCLUIDA`

Arquivos alterados:

```txt
portal-sama-web/src/types/certificates.ts
portal-sama-web/src/types/clients.ts
portal-sama-web/src/schemas/certificate.schema.ts
portal-sama-web/src/services/certificates.service.ts
portal-sama-web/src/pages/clients/ClientCertificatesTab.tsx
portal-sama-web/src/pages/clients/ClientDashboardPage.tsx
portal-sama-web/src/pages/clients/components/ClientDashboardTabs.tsx
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- criada `ClientCertificatesTab` para a aba `Certificados` no painel do cliente;
- a aba usa `GET /certificates` filtrado por `clientId` e respeita `certificates.read`;
- o painel mostra resumo de certificados validos, vencendo, vencidos e com senha;
- o badge da aba `Certificados` usa o resumo do dashboard para destacar certificados vencendo/vencidos;
- a lista exibe nome, validade/status, departamento, arquivo, tamanho, senha protegida e atualizacao;
- a senha nao aparece na listagem; revelacao exige clique explicito, `certificates.manage` e `sensitive.reveal`, usando `POST /certificates/:id/reveal-password`;
- download protegido usa `GET /certificates/:id/download` e depende de `certificates.download`;
- criacao/upload usa `POST /certificates` com `clientId`, `clientName`, departamento, validade, senha e arquivo `.p12/.pfx`;
- edicao usa `PATCH /certificates/:id` para metadados, validade e substituicao opcional do arquivo;
- rotacao de senha usa `POST /certificates/:id/rotate-password`;
- remocao usa `DELETE /certificates/:id`;
- mutacoes invalidam `client-certificates`, `certificates` e `client-dashboard`;
- tipos frontend passaram a refletir `validUntil`, status de validade, dias para vencer e resposta auditada de revelacao de senha;
- contrato web passou a proteger a existencia da aba, permissoes, validade e endpoint de revelacao;
- E2E focado cobre badge de certificados, alerta de vencimento, upload, edicao, rotacao, download e revelacao auditada sem senha exposta antes do clique.

## 15. Validacoes executadas apos a Etapa 11.7

Comandos executados em `portal-sama-web`:

```powershell
npm.cmd run lint
npm.cmd test -- --runInBand
npm.cmd run build
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "client dashboard certificados" --reporter=line
npm.cmd run test:e2e
```

Resultado:

```txt
lint OK
22 web contract tests passed
build OK
Playwright focado OK, 1 passed
test:e2e OK, 33 passed, 2 skipped
```

## 16. Etapa 11.8 - Implementar aba Vida da empresa

Status: `CONCLUIDA`

Arquivos alterados:

```txt
portal-sama-web/src/services/manager-history.service.ts
portal-sama-web/src/pages/clients/ClientCompanyLifeTab.tsx
portal-sama-web/src/pages/clients/ClientDashboardPage.tsx
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- criada `ClientCompanyLifeTab` para a aba `Vida da empresa` no painel do cliente;
- a aba usa o contrato backend real `GET /managers/company-life/:companyId` para consultar timeline por departamento;
- registro/atualizacao usa `PATCH /managers/company-life/:companyId` com CSRF pelo service existente;
- leitura depende de `company_life.read`;
- criacao de entrada depende de `company_life.write`;
- a tela exibe resumo de registros, departamento, contribuidores e ultima atualizacao;
- filtro de departamento e busca textual local foram adicionados;
- editor de topicos permite titulo, detalhe e marcador de concluido;
- timeline exibe autor, papel, data, resumo dos topicos e abre detalhe em `CompanyLifeHistoryDrawer`;
- o payload enviado contem apenas `company_id`, `dept` e `topics`, separando contexto operacional de segredo/senha;
- mutacoes invalidam `client-company-life` e `manager-history`;
- contrato web passou a proteger o componente, endpoint de leitura, endpoint de atualizacao e componentes internos da aba;
- E2E focado cobre consulta, busca, criacao de entrada, payload sem segredo e abertura do detalhe historico.

Observacao tecnica:

```txt
O guia sugeria endpoints /clients/:id/company-life, mas a API disponivel e validada nesta base expoe a vida da empresa pelo modulo Managers:
GET /managers/company-life/:companyId
POST /managers/company-life
PATCH /managers/company-life/:companyId

A aba do cliente reaproveita esse contrato real para evitar criar uma chamada inexistente no frontend.
```

## 17. Validacoes executadas apos a Etapa 11.8

Comandos executados em `portal-sama-web`:

```powershell
npm.cmd run lint
npm.cmd test -- --runInBand
npm.cmd run build
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "client dashboard vida" --reporter=line
npm.cmd run test:e2e
```

Resultado:

```txt
lint OK
22 web contract tests passed
build OK
Playwright focado OK, 1 passed
test:e2e OK, 34 passed, 2 skipped
```

## 18. Etapa 11.9 - Ajustar responsividade final

Status: `CONCLUIDA`

Arquivos alterados:

```txt
portal-sama-web/src/pages/clients/ClientDashboardPage.tsx
portal-sama-web/src/index.css
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- `ClientDashboardPage` recebeu a classe raiz `client-dashboard-page` para conter largura e evitar propagacao de overflow horizontal no painel;
- CSS global passou a limitar largura de `client-dashboard-page`, paineis, `QueryTabs`, `query-tab-panel`, tabelas e areas `overflow-x-auto`;
- `QueryTabs` manteve comportamento flexivel em desktop/notebook e passou a organizar as seis abas em duas colunas no mobile;
- `panel-pad` ficou mais compacto em telas ate 560px, preservando espaco util para formularios, cards e tabelas;
- contrato web da Fase 11 passou a verificar os marcadores de responsividade do painel;
- E2E novo valida o painel em 1180x760 e 390x844, navegando por `Geral`, `Documentos`, `Certificados` e `Vida da empresa` sem overflow horizontal.

## 19. Validacoes executadas apos a Etapa 11.9

Comandos executados em `portal-sama-web`:

```powershell
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "client dashboard remains responsive" --reporter=line
npm.cmd run lint
npm.cmd test -- --runInBand
npm.cmd run build
npm.cmd run test:e2e
```

Resultado:

```txt
Playwright focado OK, 1 passed
lint OK
22 web contract tests passed
build OK
test:e2e OK, 35 passed, 2 skipped
```

## 20. Etapa 11.10 - Criar E2E do painel completo

Status: `CONCLUIDA`

Arquivos alterados:

```txt
portal-sama-web/tests/e2e/smoke.spec.ts
portal-sama-web/scripts/web-contract-tests.mjs
```

Resultado:

- criado usuario E2E `completeDashboardUser` com permissoes amplas para o painel do cliente;
- criado teste `client dashboard complete panel walks through every tab`;
- o teste abre `/clientes/client-1/painel`, valida header, CNPJ, status e aba `Geral`;
- o teste navega pelo atalho rapido `Responsaveis` e confirma `?tab=responsaveis#client-assignments`;
- o teste percorre as abas `Acessos`, `Documentos`, `Certificados` e `Vida da empresa`;
- cada aba recebe mock do contrato backend correspondente e valida conteudo operacional principal;
- o teste confirma badges de documentos/certificados, ausencia de senha revelada antes de acao explicita e ausencia de overflow horizontal;
- contrato web passou a verificar a existencia do E2E completo e dos marcadores principais do fluxo.

## 21. Validacoes executadas apos a Etapa 11.10

Comandos executados em `portal-sama-web`:

```powershell
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "client dashboard complete panel" --reporter=line
npm.cmd run lint
npm.cmd test -- --runInBand
npm.cmd run build
npm.cmd run test:e2e
git diff --check
```

Resultado:

```txt
Playwright focado OK, 1 passed
lint OK
22 web contract tests passed
build OK
test:e2e OK, 36 passed, 2 skipped
git diff --check OK no web e no docs
```

## 22. Conclusao formal da Fase 11

Status final: `CONCLUIDA`

Todas as etapas da Fase 11 foram concluidas em 2026-06-30:

```txt
11.1 ClientDashboardHeader
11.2 ClientDashboardTabs
11.3 Aba Geral
11.4 Aba Responsaveis
11.5 Aba Acessos
11.6 Aba Documentos
11.7 Aba Certificados
11.8 Aba Vida da empresa
11.9 Responsividade final
11.10 E2E do painel completo
```

## 23. Pendencias da Fase 11

Nenhuma pendencia funcional restante conforme `20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md`.

## 24. Acao em producao

Nenhuma acao manual em producao e obrigatoria para concluir as etapas 11.1 a 11.10 no repositorio local.

Quando a build desta etapa for publicada no EasyPanel, executar:

1. Conferir que o deploy do `portal-sama-web` usa o commit/build que contem este documento e os arquivos listados acima.
2. Rebuildar/redeployar apenas o servico web; nao ha migration, env nova ou alteracao obrigatoria na API para as etapas 11.1 a 11.10.
3. Entrar com usuario que tenha `clients.read` e abrir `/clientes`.
4. Abrir um cliente pela acao de painel e confirmar `/clientes/{id}/painel`.
5. Validar no navegador: botao voltar, nome, razao social, CNPJ, rank, status, contato, endereco e botoes `Responsaveis`/`Documentos`.
6. Validar as abas `Geral`, `Responsaveis`, `Acessos`, `Documentos`, `Certificados` e `Vida da empresa`.
7. Confirmar que trocar de aba atualiza a URL com `?tab=...`, especialmente `?tab=documentos`.
8. Na aba `Geral`, com usuario que tenha `clients.update`, editar temporariamente um campo controlado e conferir mensagem de sucesso; depois restaurar o valor original.
9. Com usuario que tenha `clients.update`, trocar temporariamente o status de um cliente controlado e conferir mensagem de sucesso; depois restaurar o status original.
10. Com usuario sem `clients.update`, confirmar que o seletor de status e o botao `Editar dados` nao aparecem.
11. Na aba `Responsaveis`, entrar com usuario que tenha `client_assignments.read`, `client_assignments.create`, `client_assignments.update`, `client_assignments.transfer`, `client_assignments.end`, `departments.read` e `collaborators.read`.
12. Em um cliente controlado, validar criar uma responsabilidade temporaria, editar o tipo/status, transferir para outro responsavel e encerrar; ao final, restaurar ou remover o dado temporario conforme regra operacional.
13. Repetir a aba `Responsaveis` com usuario apenas de leitura e confirmar que as acoes mutaveis nao aparecem.
14. Na aba `Acessos`, entrar com usuario que tenha `client_accesses.read` e confirmar que a lista carrega sem expor segredo/senha.
15. Com usuario que tambem tenha `client_accesses.manage`, criar um acesso temporario usando uma `Referencia segura` controlada, editar status/notas e arquivar com motivo.
16. Confirmar que a tela nao possui campo de senha em texto puro nem botao de revelacao/copia de segredo nesta build.
17. Se houver necessidade operacional de revelar segredo no futuro, implementar primeiro endpoint backend auditado/rate-limited antes de habilitar qualquer botao no frontend.
18. Na aba `Documentos`, entrar com usuario que tenha `documents.read` e confirmar checklist, badge de pendencia e extensoes permitidas.
19. Com usuario que tenha `documents.upload`, enviar um PDF controlado em requisito documental temporario e conferir retorno visual.
20. Com usuario que tenha `documents.requirements`, criar requisito documental temporario e confirmar que ele aparece como pendente.
21. Com usuario que tenha `documents.review`, aprovar/rejeitar um documento temporario e conferir atualizacao de status.
22. Com usuario que tenha `documents.download`, baixar documento controlado e confirmar que o download passa pelo endpoint protegido.
23. Abrir o historico de um documento revisado e confirmar usuario/data/motivo sem expor storage key.
24. Na aba `Certificados`, entrar com usuario que tenha `certificates.read` e confirmar lista, validade/status, badge de vencimento e extensoes permitidas.
25. Com usuario que tenha `certificates.manage`, cadastrar certificado temporario `.pfx`/`.p12`, editar validade/departamento e rotacionar senha.
26. Com usuario que tenha `certificates.download`, baixar certificado controlado pelo endpoint protegido.
27. Com usuario que tenha `certificates.manage` e `sensitive.reveal`, revelar senha apenas por clique explicito e confirmar que a senha nao aparece na listagem antes da acao.
28. Na aba `Vida da empresa`, entrar com usuario que tenha `company_life.read` e confirmar timeline por departamento, busca e contribuidores.
29. Com usuario que tenha `company_life.write`, criar entrada temporaria com topico/detalhe e confirmar retorno visual.
30. Abrir o detalhe da entrada criada e confirmar autor/data/topicos sem expor segredo/senha.
31. Validar o painel em notebook menor e mobile, conferindo que header, abas, paineis, cards e tabelas nao geram rolagem horizontal da pagina.
32. Percorrer o painel completo de um cliente controlado passando por todas as abas em uma unica sessao autenticada.
33. Restaurar/remover dados temporarios criados no teste operacional conforme regra interna.

## 25. Proximo passo recomendado

Iniciar a Fase 12 pela Etapa 12.1 - refinar tela de Clientes.
