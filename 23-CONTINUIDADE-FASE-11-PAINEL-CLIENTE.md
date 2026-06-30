# Continuidade Fase 11 - Painel do cliente final

Atualizado em: 2026-06-30
Ambiente alvo: `https://portal.samacontabil.com.br`
Status formal: Fase 11 `EM_EXECUCAO`
Etapa atual: 11.3 - Implementar aba Geral

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

## 6. Pendencias da Fase 11

Ainda pendentes conforme `20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md`:

1. Etapa 11.3 - Implementar aba Geral.
2. Etapa 11.4 - Implementar aba Responsaveis.
3. Etapa 11.5 - Implementar aba Acessos.
4. Etapa 11.6 - Implementar aba Documentos.
5. Etapa 11.7 - Implementar aba Certificados.
6. Etapa 11.8 - Implementar aba Vida da empresa.
7. Etapa 11.9 - Ajustar responsividade final.
8. Etapa 11.10 - Criar E2E do painel completo.

## 7. Acao em producao

Nenhuma acao manual em producao e obrigatoria para concluir as etapas 11.1 e 11.2 no repositorio local.

Quando a build desta etapa for publicada no EasyPanel, executar:

1. Conferir que o deploy do `portal-sama-web` usa o commit/build que contem este documento e os arquivos listados acima.
2. Rebuildar/redeployar apenas o servico web; nao ha migration, env nova ou alteracao obrigatoria na API para as etapas 11.1 e 11.2.
3. Entrar com usuario que tenha `clients.read` e abrir `/clientes`.
4. Abrir um cliente pela acao de painel e confirmar `/clientes/{id}/painel`.
5. Validar no navegador: botao voltar, nome, razao social, CNPJ, rank, status, contato, endereco e botoes `Responsaveis`/`Documentos`.
6. Validar as abas `Geral`, `Responsaveis`, `Acessos`, `Documentos`, `Certificados` e `Vida da empresa`.
7. Confirmar que trocar de aba atualiza a URL com `?tab=...`, especialmente `?tab=documentos`.
8. Com usuario que tenha `clients.update`, trocar temporariamente o status de um cliente controlado e conferir mensagem de sucesso; depois restaurar o status original.
9. Com usuario sem `clients.update`, confirmar que o seletor de status nao aparece.

## 8. Proximo passo recomendado

Continuar pela Etapa 11.3 - implementar a aba Geral com dados completos da empresa e edicao rapida conforme permissao.
