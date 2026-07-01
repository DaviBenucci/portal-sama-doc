# Portal Sama — nova arquitetura, reaproveitamento do legado e guia para Codex

Atualizado em: 2026-06-19 12:00 -03:00

Este pacote documental foi criado para orientar a conclusão da nova arquitetura do Portal Sama separada em:

- `portal-sama-api`: backend NestJS, TypeScript, Prisma e MySQL.
- `portal-sama-web`: frontend React, TypeScript, Vite, TanStack Query e Zustand.
- `portal-sama-docs`: documentação operacional, arquitetura, segurança, homologação e guia de implementação.
- `portal-sama.zip`: aplicação legada em PHP, JavaScript, HTML e CSS, usada como fonte funcional e UX de referência.

## Ordem de leitura obrigatória

1. [`15-ANALISE-SEMANTICA-LEGADO-VS-NOVA-ARQUITETURA.md`](./15-ANALISE-SEMANTICA-LEGADO-VS-NOVA-ARQUITETURA.md)
2. [`16-BACKEND-API-BANCO-DETALHADO.md`](./16-BACKEND-API-BANCO-DETALHADO.md)
3. [`17-FRONTEND-UX-REAPROVEITAMENTO-LEGADO.md`](./17-FRONTEND-UX-REAPROVEITAMENTO-LEGADO.md)
4. [`18-ZAPSIGN-MIGRACAO-LEGADO-PARA-NESTJS.md`](./18-ZAPSIGN-MIGRACAO-LEGADO-PARA-NESTJS.md)
5. [`19-SEGURANCA-GOVERNANCA-DOCUMENTOS.md`](./19-SEGURANCA-GOVERNANCA-DOCUMENTOS.md)
6. [`20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md`](./20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md)
7. [`21-CONTINUIDADE-FASE-9-EASYPANEL.md`](./21-CONTINUIDADE-FASE-9-EASYPANEL.md), como evidencia formal da Fase 9 `CONCLUIDA`.
8. [`22-CONTINUIDADE-FASE-10-FRONTEND.md`](./22-CONTINUIDADE-FASE-10-FRONTEND.md), como evidencia formal da Fase 10 `CONCLUIDA`.
9. [`23-CONTINUIDADE-FASE-11-PAINEL-CLIENTE.md`](./23-CONTINUIDADE-FASE-11-PAINEL-CLIENTE.md), como evidencia formal da Fase 11 `CONCLUIDA`.
10. [`24-CONTINUIDADE-FASE-12-MODULOS.md`](./24-CONTINUIDADE-FASE-12-MODULOS.md), como acompanhamento formal da Fase 12 `EM_EXECUCAO`.
11. [`CONTEXTO-CODEX-ATUAL.md`](./CONTEXTO-CODEX-ATUAL.md), como estado operacional mais recente.

## Precedencia documental

Os documentos Markdown localizados na raiz deste workspace sao a fonte normativa principal para produto, arquitetura, seguranca, QA e continuidade do Codex. Em caso de conflito, eles prevalecem sobre documentos auxiliares em `docs/`, `evidencias/`, auditorias antigas ou registros de acompanhamento.

Precedencia pratica:

1. Documentos de decisao da raiz: `00-LEIA-ME-NOVA-ARQUITETURA-CODEX.md`, `12-RUNBOOK-ACESSORIAS.md`, `15-ANALISE-SEMANTICA-LEGADO-VS-NOVA-ARQUITETURA.md`, `16-BACKEND-API-BANCO-DETALHADO.md`, `17-FRONTEND-UX-REAPROVEITAMENTO-LEGADO.md`, `18-ZAPSIGN-MIGRACAO-LEGADO-PARA-NESTJS.md`, `19-SEGURANCA-GOVERNANCA-DOCUMENTOS.md`, `21-CONTINUIDADE-FASE-9-EASYPANEL.md`, `22-CONTINUIDADE-FASE-10-FRONTEND.md`, `23-CONTINUIDADE-FASE-11-PAINEL-CLIENTE.md`, `24-CONTINUIDADE-FASE-12-MODULOS.md`, `CONTEXTO-CODEX-ATUAL.md` e `Testes-da-aplicação-DEPLOY.md`.
   Durante a Fase 12, `24-CONTINUIDADE-FASE-12-MODULOS.md` e `CONTEXTO-CODEX-ATUAL.md` definem a proxima etapa operacional.
2. Guia operacional da raiz: `20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md`, usado para ordenar fases, status permitidos e evidencias.
3. Documentos em `docs/`, usados como acompanhamento, matriz, checklist e evidencias; eles nao podem liberar uma fase se conflitarem com os documentos de maior precedencia.

## Decisão central

A nova arquitetura não deve reescrever o Portal Sama ignorando o legado. O legado deve ser tratado como **fonte de regra de negócio, fluxo operacional e UX validada**, enquanto a nova arquitetura deve ser tratada como **fundação técnica segura, testável e escalável**.

O objetivo é unir os dois pontos fortes:

- reaproveitar do legado: painel do cliente por abas, navegação mais direta, fluxo ZapSign, rotina de legalização, visão operacional por cliente, histórico/vida da empresa e decisões de layout já compreensíveis para o usuário;
- preservar/evoluir na nova arquitetura: NestJS modular, Prisma, MySQL, RBAC, CSRF, tokens opacos, storage privado, scanner de upload, auditoria, Docker, testes automatizados e deploy separado.

## Regra rígida para o Codex

O Codex somente pode avançar para a próxima etapa quando a etapa anterior estiver **CONCLUÍDA**, validada por evidência técnica e com status atualizado na documentação.

Cada etapa deve seguir este ciclo:

```txt
1. Ler a etapa atual.
2. Confirmar arquivos e contratos impactados.
3. Implementar somente o escopo da etapa.
4. Executar validações exigidas da etapa.
5. Registrar evidência: comando, resultado, arquivos alterados e pendências.
6. Atualizar status da etapa para CONCLUÍDA somente se não houver erro bloqueante.
7. Avançar para a próxima etapa.
```

Se surgir imprevisto, o Codex deve criar uma **sub-etapa corretiva dentro da etapa atual**, resolver, validar e somente depois continuar. Não é permitido pular para outra fase para compensar uma etapa incompleta.

Urgencia ativa em 2026-07-01: antes de continuar a Fase 12 pela Etapa 12.2, executar a sub-etapa 12.1.1 para adicionar a entrada do Integra-AI com os botoes `Extrato` e `Faturamento`. `Extrato` preserva o fluxo atual. `Faturamento` deve criar frontend para o backend Python existente em `C:\Users\Sama Contabilidade\Downloads\Faturamento`, com escolha entre `Um mes` e `Todos os meses` e `codigo_dominio` opcional quando o codigo do Dominio for diferente do Acessorias.

## Regras específicas da integração Acessórias

A integração Acessórias deve ser ajustada para evitar excesso de histórico e dados obsoletos:

- obrigações/listas de entregas devem ser buscadas a partir do **ano atual**;
- colaboradores/responsáveis devem ser buscados a partir do **mês atual**;
- se a API externa não aceitar filtro por data, o backend deve aplicar filtro pós-consulta antes de persistir;
- colaboradores antigos que já saíram da empresa não devem ser importados como responsáveis ativos;
- obrigações antigas, expiradas ou acumuladas por anos no Acessórias não devem alimentar workspace operacional atual.

## Observação crítica de segurança

Foram encontrados arquivos `.env` nos pacotes analisados. A documentação e os pacotes compartilháveis não devem incluir `.env`, tokens, chaves, dumps, certificados, backups ou segredos reais. Qualquer segredo que já tenha sido empacotado deve ser considerado comprometido e rotacionado antes de produção.
