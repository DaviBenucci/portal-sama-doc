# Continuidade Fase 12 - Telas de modulos apos painel do cliente

Atualizado em: 2026-06-30
Ambiente alvo: `https://portal.samacontabil.com.br`
Status formal: Fase 12 `EM_EXECUCAO`
Etapa atual: 12.2 - Refinar tela de Documentos

## 1. Entrada da fase

A Fase 12 foi iniciada depois da Fase 11 ficar formalmente `CONCLUIDA`.

Evidencia de entrada:

```txt
23-CONTINUIDADE-FASE-11-PAINEL-CLIENTE.md
Fase 11 CONCLUIDA
lint OK
22 web contract tests passed
build OK
test:e2e OK, 36 passed, 2 skipped
git diff --check OK
```

Escopo da Fase 12 conforme o guia raiz:

```txt
Fase 12 - Telas de modulos apos painel do cliente
```

## 2. Etapa 12.1 - Refinar tela de Clientes

Status: `CONCLUIDA`

Arquivos alterados:

```txt
portal-sama-web/src/pages/clients/ClientsPage.tsx
portal-sama-web/scripts/web-contract-tests.mjs
portal-sama-web/tests/e2e/smoke.spec.ts
```

Resultado:

- filtros da tela `/clientes` passaram a ser refletidos na URL: `search`, `status`, `cnpj`, `cidade`, `grupo`, `archived`, `scope` e `skip`;
- a lista usa estado local sincronizado com a URL para preservar preenchimentos rapidos sem perder parametros;
- o link de painel do cliente ficou textual, com icone e rotulo visivel `Painel`, mantendo `aria-label` especifico por cliente;
- o retorno do painel preserva os filtros da lista via `state.from.search`;
- filtro `Arquivados` recebeu `id` e `label` explicitos;
- foi adicionado botao `Limpar` para zerar filtros operacionais sem remover o escopo selecionado;
- status da listagem passou a ter fallback de label quando o backend nao enviar `status_label`;
- contrato web novo cobre a Etapa 12.1;
- E2E novo valida busca, filtro de status, cidade, grupo, arquivados, escopo, chamada de API, abertura do painel em um clique, retorno preservando filtros e limpeza.

## 3. Validacoes executadas apos a Etapa 12.1

Comandos executados em `portal-sama-web`:

```powershell
npx.cmd playwright test tests/e2e/smoke.spec.ts -g "clients page keeps filters" --reporter=line
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
23 web contract tests passed
build OK
test:e2e OK, 37 passed, 2 skipped
git diff --check OK no web e no docs
```

## 4. Pendencias da Fase 12

Ainda pendentes conforme `20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md`:

1. Etapa 12.2 - Refinar tela de Documentos.
2. Etapa 12.3 - Refinar tela de Certificados.
3. Etapa 12.4 - Refinar Legalizacao.
4. Etapa 12.5 - Refinar tela de contratos ZapSign.
5. Etapa 12.6 - Refinar Onboarding.
6. Etapa 12.7 - Refinar T.I./Acessos.

## 5. Acao em producao

Nenhuma acao manual em producao e obrigatoria para concluir a Etapa 12.1 no repositorio local.

Quando a build desta etapa for publicada no EasyPanel, executar:

1. Conferir que o deploy do `portal-sama-web` usa o commit/build que contem este documento e os arquivos listados acima.
2. Rebuildar/redeployar apenas o servico web; nao ha migration, env nova ou alteracao obrigatoria na API para a Etapa 12.1.
3. Entrar com usuario que tenha `clients.read` e abrir `/clientes`.
4. Preencher busca, status, cidade, grupo e `Arquivados`, confirmando que a URL reflete os filtros.
5. Abrir um cliente pelo botao `Painel` em uma unica acao e confirmar `/clientes/{id}/painel`.
6. Usar o botao voltar `Clientes` no header do painel e confirmar que os filtros da lista continuam preenchidos.
7. Usar `Limpar` e confirmar que filtros sao removidos, preservando o escopo `Todos permitidos`/`Ver meus clientes` quando aplicavel.

## 6. Proximo passo recomendado

Continuar pela Etapa 12.2 - refinar tela de Documentos.
