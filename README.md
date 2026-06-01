# [PARCIAL] Portal Sama Docs

Este repositorio e a fonte oficial de documentacao do Portal Sama.

## Legenda de status

- `[CONCLUIDO]`: decisao, incidente, bug ou pacote documentado ja refletido no codigo atual, ou documento historico sem acao tecnica aberta.
- `[PARCIAL]`: ha implementacao/base no codigo, mas ainda faltam validacao real, backfill, aceite, UI complementar ou corte do legado.
- `[PENDENTE]`: documento-gate ou backlog com bloqueio principal ainda aberto.

Antes de alterar `portal-sama-api` ou `portal-sama-web`, leia primeiro:

1. `INSTRUÇÕEDS_AI.MD`
2. `ORDEM_IMPLEMENTACAO_DOCUMENTACOES.md`
3. `AINDA_FALTA_PARA_DEPLOY_EM_PRODUÇÃO.MD`
4. `STATUS_IMPLEMENTACAO.md`
5. `PENDENCIAS_TECNICAS.md`
6. `EASYPANEL_DEPLOY.md`
7. `ux-ui-docs/README.md`

## Referencia obrigatoria de UX/UI

O pacote `ux-ui-docs/` e a referencia oficial para melhoria visual, organizacao de telas, navegacao e criterios de aceite de UI/UX do Portal Sama.

Depois da integracao com o Acessorias, toda rodada de melhoria de interface deve seguir `ux-ui-docs/09-checklist-aceite.md`. Esse QA de UI/UX sera a ultima verificacao antes de liberar o sistema para os usuarios utilizarem.

## Referencia da integracao Acessorias

Para Fase 2/Fase 3, use `integracao_acessorias_paths_importacao_dev.md` como contrato operacional curto de paths, `.env`, importacao DEV e regras de seguranca; o documento detalhado de negocio continua em `integracao_acessorias_entregas_vencimentos.md`.

Depois de qualquer mudanca tecnica em API ou Web, atualize aqui os documentos de status, changelog, testes, pendencias, decisoes e deploy quando aplicavel.
