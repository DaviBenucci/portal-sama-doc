# [PENDENTE] Ordem de Implementacao para Concluir as Documentacoes

Ultima atualizacao: 2026-06-01
Responsavel/IA: Codex
Status do documento: guia operacional vivo

## Objetivo

Definir a ordem de continuidade das implementacoes para que todos os documentos marcados como `[PENDENTE]` ou `[PARCIAL]` avancem ate `[CONCLUIDO]` sem perda de escopo.

Este arquivo deve ser usado como trilho principal antes de iniciar novas rodadas na API, no Web ou na documentacao.

## Regra de uso

1. Comecar sempre pela primeira fase que ainda nao atingiu o criterio de saida.
2. Nao pular para uma fase posterior quando a anterior ainda bloqueia deploy, dados reais, seguranca, RBAC ou aceite.
3. Ao finalizar uma etapa, atualizar os documentos citados na propria fase.
4. Um documento so deve mudar para `[CONCLUIDO]` quando o codigo, a validacao e as evidencias correspondentes existirem.
5. Documentos ja marcados como `[CONCLUIDO]` nao devem ser reabertos, salvo regressao, mudanca de decisao ou descoberta de divergencia no codigo.

## Fase 0 - Base documental e navegacao

Estado: em andamento.

Objetivo: garantir que a documentacao esteja navegavel e que cada arquivo indique seu status real.

Ja feito:

- Titulos dos Markdown foram classificados com `[CONCLUIDO]`, `[PARCIAL]` ou `[PENDENTE]`.
- `README.md` recebeu legenda dos status.

Falta:

- Versionar esta ordem de implementacao.
- Manter este arquivo referenciado nos documentos de entrada.

Documentos principais:

- `README.md`
- `README_DOCUMENTACAO.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUCAO.MD`
- `STATUS_IMPLEMENTACAO.md`
- `PENDENCIAS_TECNICAS.md`

Criterio de saida:

- Todos os pontos de entrada citam este arquivo.
- Proximas rodadas usam esta ordem antes de escolher tarefas soltas.

## Fase 1 - Homologacao operacional obrigatoria

Estado: instrumentalizada localmente; execucao real no EasyPanel ainda pendente.

Objetivo: remover os bloqueios reais que impedem dizer que a stack esta pronta para continuar sem risco.

Ja feito:

- `portal-sama-api` possui `npm run ops:phase1` para orquestrar migrations, seed, readiness, backup, verificacao e restore drill.
- O comando gera evidencia JSON e mantem restore real protegido por alvo isolado e confirmacao explicita.

Implementar/validar em ordem:

1. Aplicar migrations pendentes no MySQL real do EasyPanel.
2. Rodar `prisma:seed` no container real para RBAC atualizado.
3. Executar readiness real sem warnings criticos.
4. Rodar backup real no EasyPanel.
5. Verificar artefatos de backup.
6. Copiar ou garantir retencao externa dos artefatos.
7. Executar restore drill real em banco/storage isolados.
8. Registrar evidencia sanitizada.
9. Repetir smoke/auth/permissoes com matriz minima de perfis reais.

Documentos que devem ser atualizados:

- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUCAO.MD`
- `EASYPANEL_DEPLOY.md`
- `PLANO_ROLLBACK_RESTORE_DRILL.md`
- `RELATORIO_TESTES.md`
- `PENDENCIAS_TECNICAS.md`
- `STATUS_IMPLEMENTACAO.md`
- `INVENTARIO_SEGURANCA.md`

Criterio de saida:

- Restore drill real aprovado.
- Migrations e seed aplicados no ambiente real.
- Matriz minima de autenticacao/RBAC validada com usuarios reais.
- Backups/rollback deixam de ser bloqueio de producao.

## Fase 2 - Contrato real do Acessorias

Objetivo: validar a integracao externa antes de expandir automacoes internas.

Implementar/validar em ordem:

1. Configurar `ACESSORIAS_BASE_URL`, `ACESSORIAS_TOKEN`, `ACESSORIAS_HOME_PATH`, `ACESSORIAS_CLIENTS_PATH`, `ACESSORIAS_COLLABORATORS_PATH` e `ACESSORIAS_DELIVERIES_PATH` no EasyPanel.
2. Validar payload real de Home, empresas, colaboradores e entregas.
3. Ajustar normalizadores se o contrato real divergir do previsto.
4. Rodar preview de empresas/colaboradores.
5. Sincronizar empresas e colaboradores preservando edicao local no Portal Sama.
6. Sincronizar entregas e conferir deduplicacao por `external_id`.
7. Validar auditoria e permissoes da integracao.

Documentos que devem ser atualizados:

- `integracao_acessorias_entregas_vencimentos.md`
- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUCAO.MD`
- `MAPEAMENTO_MIGRACAO_APIS.md`
- `INVENTARIO_ENDPOINTS.md`
- `INVENTARIO_BANCO_DADOS.md`
- `RELATORIO_TESTES.md`
- `STATUS_IMPLEMENTACAO.md`

Criterio de saida:

- Empresas e colaboradores podem vir do Acessorias e continuar editaveis no Portal Sama.
- Entregas reais ficam persistidas localmente.
- Home por perfil funciona com dados reais do Acessorias no EasyPanel.

## Fase 3 - Acessorias aplicado na planilha Fiscal e vencimentos

Estado: primeira aplicacao local implementada; validacao real e revisao manual ainda pendentes.

Objetivo: transformar a sincronizacao do Acessorias em operacao diaria visivel e auditavel.

Ja feito localmente:

- `POST /api-v2/integrations/acessorias/deliveries/apply-to-fiscal` aplica entregas `DELIVERED` com mapeamento confirmado na planilha Fiscal existente.
- Celulas aplicadas recebem status visual `ACESSORIAS`.
- Casos inseguros geram divergencias persistidas em `acessorias_fiscal_divergences`.
- `/departamentos/modelo` exibe contadores, marcador de celula e divergencias abertas.

Implementar em ordem:

1. [local concluido; real pendente] Aplicar mapeamentos confirmados na planilha Fiscal.
2. [local concluido; real pendente] Marcar celulas preenchidas pelo Acessorias com status visual proprio.
3. [local concluido; revisao pendente] Criar divergencias quando nao houver confianca suficiente para baixar automaticamente.
4. Permitir revisao manual das divergencias.
5. Conectar entregas/vencimentos ao calendario operacional.
6. Criar Central de Vencimentos baseada nas entregas sincronizadas.
7. Adicionar notificacoes para vencimento proximo, atraso, baixa e divergencia.
8. Criar scheduler seguro para sincronizacao periodica.

Documentos que devem ser atualizados:

- `integracao_acessorias_entregas_vencimentos.md`
- `paginas/depto-modelo.md`
- `paginas/manager-calendar-config.md`
- `paginas/manager-manager.md`
- `ux-ui-docs/03-home-por-perfil.md`
- `ux-ui-docs/05-modulos-operacao.md`
- `ux-ui-docs/09-checklist-aceite.md`
- `RELATORIO_TESTES.md`

Criterio de saida:

- Planilha Fiscal reflete entregas do Acessorias sem sobrescrever casos duvidosos.
- Divergencias existem e podem ser tratadas.
- Vencimentos e notificacoes deixam de depender somente do legado/manual.

## Fase 4 - Responsabilidade de clientes e carteiras

Objetivo: remover a dependencia operacional exclusiva de `clients.metadata` para carteira, responsavel e gestor.

Implementar em ordem:

1. Aplicar migrations/seeds de `client_department_assignments` no MySQL real.
2. Rodar relatorio de backfill.
3. Criar backfill seguro de responsabilidades a partir de `clients.metadata`.
4. Conectar painel do cliente a uma guia Equipe/Responsaveis.
5. Permitir atribuicao inicial de responsavel operacional por departamento.
6. Permitir gestor responsavel quando aplicavel.
7. Conectar telas de gestor/carteira ao modelo normalizado.
8. Garantir transferencia auditada pela UI.
9. Migrar filtros de planilha, documentos e vencimentos para responsabilidade normalizada.

Documentos que devem ser atualizados:

- `ux-ui-docs/11-responsabilidade-clientes-usuarios.md`
- `ux-ui-docs/10-departamentos-solicitacoes-permissoes.md`
- `paginas/client-painel.md`
- `paginas/dptclient-clientdpto.md`
- `paginas/manager-colaborador.md`
- `paginas/manager-transfers.md`
- `BANCO_DADOS_MYSQL_PRISMA.md`
- `INVENTARIO_BANCO_DADOS.md`

Criterio de saida:

- Carteira usa entidade propria.
- Transferencia de cliente entre responsaveis funciona pela UI com auditoria.
- `clients.metadata` deixa de ser fonte operacional principal para responsabilidade.

## Fase 5 - UX/UI estrutural e configuracoes

Objetivo: fechar a experiencia prevista em `ux-ui-docs` depois da base operacional estar segura.

Implementar em ordem:

1. Reorganizar sidebar por grupos: Operacao, Gestao, Entrada de Cliente, T.I e Admin.
2. Garantir drawer mobile.
3. Ajustar visibilidade por permissao/departamento.
4. Mover notificacoes para o header com sino, badge e popover.
5. Criar rota `/configuracoes`.
6. Implementar abas de Minha conta, Seguranca, Notificacoes e Preferencias.
7. Implementar foto de perfil com validacao segura.
8. Implementar troca/redefinicao de senha conforme regra MASTER/auditoria.
9. Exibir sessoes ativas, ultimos acessos e dispositivos recentes quando houver backend.

Documentos que devem ser atualizados:

- `ux-ui-docs/01-visao-geral.md`
- `ux-ui-docs/02-navegacao-e-rotas.md`
- `ux-ui-docs/04-wireframes-textuais.md`
- `ux-ui-docs/07-configuracoes-e-seguranca.md`
- `ux-ui-docs/09-checklist-aceite.md`
- `MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `SEGURANCA.md`

Criterio de saida:

- UX principal passa no checklist desktop/mobile.
- Configuracoes existem e respeitam as regras de seguranca.
- Navegacao deixa de ser apenas lista historica de rotas.

## Fase 6 - Fechamento funcional por modulo

Objetivo: transformar telas parciais em fluxos completos com dados reais.

Ordem recomendada:

1. Clientes, painel do cliente, documentos e certificados.
2. Entrada de cliente, onboarding publico e links publicos.
3. Propostas, contratos, assinatura e PDF real.
4. Integra-AI, Dominio real, PDF/OFX e golden file aprovado.
5. Gestor, transferencias, historico e calendario.
6. Solicitacoes de acesso, T.I e auditoria.
7. DEV/Admin, usuarios, roles, permissoes e departamentos guiados.

Documentos que devem ser atualizados:

- `paginas/*.md`, conforme modulo afetado.
- `MATRIZ_MIGRACAO_HTML_PARA_REACT.md`
- `MAPEAMENTO_MIGRACAO_APIS.md`
- `MATRIZ_MIGRACAO_PHP_PARA_NESTJS.md`
- `INVENTARIO_ENDPOINTS.md`
- `RELATORIO_TESTES.md`

Criterio de saida:

- Cada tela documentada tem rota React real, contrato API v2 real, validacao com dados reais e criterio de aceite fechado.
- Os documentos individuais de `paginas/` podem migrar gradualmente de `[PARCIAL]` para `[CONCLUIDO]`.

## Fase 7 - QA final, seguranca e corte do legado

Objetivo: provar que a nova stack pode operar sem depender do PHP legado.

Implementar/validar em ordem:

1. Playwright real por perfis criticos.
2. Smoke publico, auth e permissoes com matriz completa.
3. Upload/download real com storage privado e ClamAV strict.
4. Auditoria de acoes sensiveis.
5. Exportacao/download real do Integra-AI.
6. TXT real aprovado no Dominio.
7. Links publicos reais de documentos, propostas e assinatura.
8. Plano de corte do legado.
9. Plano de rollback do corte.
10. Evidencias sanitizadas anexadas aos documentos corretos.

Documentos que devem ser atualizados:

- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUCAO.MD`
- `RELATORIO_TESTES.md`
- `SEGURANCA.md`
- `EASYPANEL_DEPLOY.md`
- `PLANO_ROLLBACK_RESTORE_DRILL.md`
- `CHANGELOG_TECNICO.md`
- `STATUS_IMPLEMENTACAO.md`

Criterio de saida:

- `AINDA_FALTA_PARA_DEPLOY_EM_PRODUCAO.MD` deixa de ser `[PENDENTE]`.
- `ux-ui-docs/09-checklist-aceite.md` fica totalmente marcado.
- Dependencia do legado fica documentada como desligada ou restrita a fallback aprovado.

## Fase 8 - Fechamento documental

Objetivo: concluir a documentacao sem deixar rastros contraditorios.

Executar em ordem:

1. Revisar todos os arquivos `[PENDENTE]`.
2. Revisar todos os arquivos `[PARCIAL]`.
3. Confirmar que cada documento tem evidencia correspondente no codigo, nos testes ou no ambiente real.
4. Atualizar percentuais e bloqueios.
5. Remover ou marcar como historicas decisoes superadas.
6. Atualizar titulos para `[CONCLUIDO]` somente quando o criterio real estiver satisfeito.
7. Fazer commit final da documentacao.

Documentos que devem ser atualizados:

- Todos os Markdown ainda marcados como `[PENDENTE]` ou `[PARCIAL]`.

Criterio de saida:

- Todos os documentos ficam `[CONCLUIDO]`, ou permanecem `[PARCIAL]` apenas se houver escopo explicitamente futuro e fora do corte de producao.

## Ordem curta para a proxima rodada

1. Fase 1: migrations/seed/restore drill/matriz real.
2. Fase 2: contrato real do Acessorias.
3. Fase 3: Acessorias aplicado na planilha Fiscal.
4. Fase 4: responsabilidades/carteiras normalizadas na UI.
5. Fase 5: UX/UI estrutural e configuracoes.
6. Fase 6: fechamento modulo a modulo.
7. Fase 7: QA final e corte do legado.
8. Fase 8: fechamento documental.

## Regra para mudar status dos documentos

Use estes criterios:

- `[CONCLUIDO]`: codigo implementado, validacao local e/ou real executada conforme risco, documentacao atualizada e nenhuma pendencia aberta para o escopo daquele arquivo.
- `[PARCIAL]`: existe base implementada, mas falta validacao real, UI, backfill, aceite, teste E2E, dados reais ou corte do legado.
- `[PENDENTE]`: ainda e gate, backlog ou bloqueio principal de producao/homologacao.
