# PROMPT CODEX - Homologacao EasyPanel Acessorias, Home, notificacoes e solicitacoes

Use este prompt dentro do Codex no workspace com os tres repositorios:

```md
Voce e um engenheiro senior full-stack, QA engineer e especialista AppSec trabalhando no Portal Sama.

Contexto:

- O usuario aplicou as documentacoes `AUDITORIA-DEPLOY-00` ate `AUDITORIA-DEPLOY-16` e os planos `00` ate `07`.
- A integracao Acessorias agora esta funcionando parcialmente em EasyPanel: empresas estao sendo puxadas e parte da tratativa ja acontece.
- Agora precisamos corrigir os bugs vistos na homologacao real e criar testes rigorosos, no mesmo estilo das auditorias anteriores.

Antes de alterar codigo, leia obrigatoriamente:

1. `portal-sama-docs/00-LEIA-ME-PARA-IA-MVP.md`
2. `portal-sama-docs/03-CONTRATO-ACESSORIAS-OPERACIONAL.md`
3. `portal-sama-docs/07-PROMPT-CODEX-PIPELINE-ACESSORIAS-PERSISTENCIA-PUSH-DEV.md`
4. `portal-sama-docs/AUDITORIA-DEPLOY-13-HOMOLOGACAO-LOCAL-ACESSORIAS-DOCKER.md`
5. `portal-sama-docs/AUDITORIA-DEPLOY-16-EXECUCAO-CORRECOES-GO-LIVE.md`
6. `portal-sama-docs/AUDITORIA-DEPLOY-17-HOMOLOGACAO-EASYPANEL-ACESSORIAS-POS-INTEGRACAO.md`
7. `portal-sama-docs/AUDITORIA-DEPLOY-18-PLANO-CORRECAO-HOMOLOGACAO-EASYPANEL.md`
8. `portal-sama-docs/CONTEXTO-CODEX-ATUAL.md`, se existir.

Tambem leia as evidencias desta rodada:

- `portal-sama-docs/evidencias/entrada/Homolacao-portal-sama-easypanel.txt`, se copiado para o repo.
- screenshots de homologacao na pasta de evidencias, se copiadas para o repo.

Objetivo:

Corrigir e homologar estas frentes:

1. Home DEV/Gestor inconsistente com dados Acessorias/local cache.
2. Conciliacao de responsaveis Acessorias, impedindo vinculos automaticos com departamento errado.
3. Tela DEV Acessorias com botao unico externo `Integrar` e demais acoes locais.
4. Clientes em pagina unica `Clientes`, com filtro `Ver meus clientes`.
5. Colaboradores com visao publica interna segura.
6. Notificacoes com `Ler todas`, `Confirmar todos os alertas`, retencao maxima de 100 e preferencias persistidas.
7. Solicitacoes de acesso com fluxo correto Colaborador -> Gestor -> DEV e Gestor -> DEV, sem aprovacao automatica indevida.
8. Calendario de solicitacao com selecao multipla e estilo Sama, sem depender do calendario nativo como UX principal.

Regras obrigatorias:

- Nao remova CSRF, guards ou RBAC para fazer teste passar.
- Nao coloque token, senha, JWT, CSRF, cookie, CNPJ completo, documento empresarial ou payload bruto Acessorias em logs/evidencias.
- Nao crie usuario ativo automaticamente a partir de responsavel externo do Acessorias.
- Toda regra de visibilidade precisa estar no backend, nao apenas no frontend.
- Em conciliacao de responsavel, duvida vira pendencia, nao vinculo automatico.
- Antes de testar EasyPanel com operacao que altera dados, rode localmente com banco isolado.

Trabalhe por fases conforme `AUDITORIA-DEPLOY-18`:

## Fase 0
Reproduza os problemas, mapeie arquivos e rode build/lint/testes atuais. Registre falhas preexistentes.

## Fase 1
Consolide o botao `Integrar` no DEV como unica acao manual que consulta API externa Acessorias. Demais botoes devem ser locais ou diagnosticos.

## Fase 2
Corrija conciliacao rigorosa de responsaveis por nome normalizado + departamento canonico + usuario ativo unico. Bloqueie cross-department e ambiguidade.

## Fase 3
Corrija Home DEV/Gestor para usar dados locais sincronizados, com escopo correto por perfil.

## Fase 4
Consolide Clientes e Colaboradores:

- pagina `Clientes` unica;
- filtro `Ver meus clientes`;
- listagem publica interna segura de colaboradores.

## Fase 5
Corrija notificacoes:

- `Ler todas` na pagina e no sininho;
- `Confirmar todos os alertas`;
- maximo 100 notificacoes por usuario/escopo;
- preferencias de notificacao/configuracao persistidas no backend.

## Fase 6
Corrija solicitacoes de acesso:

- colaborador cria `PENDING_MANAGER`;
- gestor aprova colaborador -> `PENDING_DEV`, nao `APPROVED`;
- gestor rejeita colaborador -> `DECLINED`;
- gestor para si/equipe -> `PENDING_DEV`;
- DEV aprova/rejeita `PENDING_DEV` -> `APPROVED`/`DECLINED`;
- notificacoes para solicitante, gestor e DEV conforme etapa;
- calendario multi-dia estilizado Sama.

## Fase 7
Crie/ajuste testes API e Playwright:

- unitarios de Acessorias, responsaveis, clientes, notificacoes e solicitacoes;
- E2E API quando aplicavel;
- Playwright para Home, DEV Integrar, Clientes, Colaboradores, Notificacoes e Solicitacoes.

## Fase 8
Execute smoke controlado no EasyPanel somente apos validacao local e com evidencias sanitizadas.

Comandos esperados:

API:

```powershell
npm.cmd run prisma:validate
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
npm.cmd run test:e2e
node scripts/validate-operational-readiness.js --soft
```

Web:

```powershell
npm.cmd run lint
npm.cmd run build
npm.cmd test -- --runInBand
npm.cmd run test:e2e -- --reporter=line
npx.cmd playwright test tests/e2e/smoke.spec.ts --reporter=line
```

Ao finalizar, crie ou atualize em `portal-sama-docs`:

```txt
AUDITORIA-DEPLOY-19-EXECUCAO-HOMOLOGACAO-EASYPANEL.md
```

O relatorio deve conter:

- arquivos alterados;
- comandos executados;
- testes que passaram/falharam;
- evidencias sanitizadas;
- bugs corrigidos;
- pendencias;
- veredito: pronto para homologacao controlada ou nao pronto.

Nao finalize sem executar testes focados. Se algum teste nao puder ser executado por ambiente, explique exatamente o motivo e como executar depois.
```
