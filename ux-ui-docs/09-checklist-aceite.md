# [PENDENTE] 09 - Checklist de Aceite

## Status parcial - 2026-05-28

- Home por perfil foi implementada localmente com dados read-only do Acessorias via API v2.
- O aceite continua pendente ate validar no EasyPanel com dados reais do Acessorias e usuarios reais por perfil.

## Status parcial - 2026-06-01 11:30

- Planilha Fiscal passou a exibir status visual `Acessorias`, contadores de baixas sincronizadas e divergencias abertas por empresa/celula.
- O aceite continua pendente ate aplicar migration, publicar no EasyPanel e validar com entregas reais do Acessorias.

## Status parcial - 2026-06-01 complementar

- `/departamentos/modelo` passou a alternar Fiscal, Contabil, Pessoal, Financeiro e Legalizacao com cards de departamento e colunas proprias.
- A Home do Acessorias passou a diferenciar indisponibilidade real de retorno vazio (`204 No Content`), exibindo diagnostico sanitizado quando houver falha.
- O aceite continua pendente ate validar dados reais, responsaveis reais e mapeamentos reais de obrigacoes no EasyPanel.

## Status parcial - 2026-06-01 vencimentos Acessorias

- O carrossel do Modelo Dpto passou a exibir vencimentos oficiais vindos de entregas Acessorias sincronizadas.
- Vencimentos Acessorias so aparecem em celula/bloqueio quando ha mapeamento confirmado para coluna operacional.
- O aceite continua pendente ate validar com entregas reais, mapeamentos reais e usuarios reais por departamento no EasyPanel.

## Status parcial - 2026-06-01 Central de Vencimentos

- `/departamentos/vencimentos` foi criada localmente para consolidar vencimentos de calendario e Acessorias do workspace.
- A tela possui filtros por departamento, mes, origem, status e busca.
- O aceite continua pendente ate validar no EasyPanel com dados reais e confirmar totais contra o Acessorias da empresa.

## Status parcial - 2026-06-02 notificacoes Acessorias

- `/dev` ganhou a acao protegida `Gerar notificacoes` para vencimento proximo, atraso, baixa e divergencia Acessorias.
- A geracao retorna resumo operacional e amostra de destinatarios.
- O aceite continua pendente ate validar notificacoes reais, auditoria, destinatarios e ausencia de duplicidade no EasyPanel.

## Status parcial - 2026-06-02 scheduler Acessorias

- Scheduler Acessorias foi implementado localmente como opt-in e fica desabilitado por padrao.
- `/dev` ganhou `Status scheduler` para conferir configuracao, proxima execucao e ultimo sync.
- O aceite continua pendente ate habilitar no EasyPanel, observar execucoes reais e validar notificacoes automaticas apenas depois da deduplicacao/destinatarios.

## Status parcial - 2026-06-02 responsabilidades no cliente

- `/clientes/:id` passou a mostrar `Equipe e responsaveis` com dados normalizados de `client_department_assignments`.
- A secao respeita `client_assignments.read` e lista responsavel, gestor, departamento, tipo, status e periodo.
- A acao `Nova responsabilidade` permite atribuicao inicial local com departamento, responsavel operacional, tipo, inicio e gestor opcional.
- O aceite continua pendente ate validar com dados reais, backfill e UI de edicao/encerramento/transferencia.

## Status parcial - 2026-06-02 UX estrutural e configuracoes

- Sidebar local foi agrupada por Operacao, Gestao, Entrada, T.I e Admin.
- Mobile local usa botao hamburguer e drawer com backdrop, validado no smoke sem overflow horizontal.
- Header local possui sino de notificacoes, badge, popover e botao para `/notificacoes`.
- `/configuracoes` foi criada com abas de conta, seguranca, notificacoes, preferencias e administracao condicional.
- Foto de perfil tem validacao local de tipo/tamanho e bloqueio de SVG.
- Backend passou a restringir troca de senha a `MASTER`, com auditoria existente em `users.update`.
- O aceite continua pendente ate validar no EasyPanel com usuarios reais, notificacoes reais, auditoria persistida e avatar no backend real.

## Status parcial - 2026-06-03 avatar persistido

- `/configuracoes` passou a salvar foto de perfil por `PATCH /api-v2/me/avatar` e carregar a imagem por `GET /api-v2/me/avatar`.
- O backend valida tipo real, bloqueia SVG, limita 2 MB, remove metadados/chunks auxiliares e grava em storage privado.
- O aceite continua pendente ate publicar no EasyPanel e validar storage real, CSRF real, auditoria persistida e UX desktop/mobile com usuario real.

## Status parcial - 2026-06-03 sessoes e acessos em configuracoes

- `/configuracoes` passou a consultar `GET /api-v2/me/security` na aba Seguranca.
- A API retorna sessoes ativas de `refresh_tokens` e ultimos acessos de `audit_logs`, sem expor token/hash/cookie.
- O aceite continua pendente ate publicar no EasyPanel e validar com usuario real, refresh tokens reais, auditoria persistida e UX desktop/mobile.

## Navegacao
- [x] Sidebar desktop mantem hover/foco.
- [x] Mobile tem botao hamburguer.
- [x] Menu mobile abre em drawer.
- [x] Rotas agrupadas corretamente.
- [x] Itens sem permissao nao aparecem.
- [ ] Departamentos especificos respeitam regras.

## Home
- [ ] Home nao e apenas lista de atalhos.
- [ ] Colaborador ve pendencias reais.
- [ ] Gestor ve carteira, atrasos e riscos.
- [ ] DEV/Admin ve diagnostico tecnico.
- [ ] Cards respeitam permissoes.

## Notificacoes
- [x] Sino no header.
- [x] Badge de nao lidas.
- [x] Popover com recentes.
- [x] Botao para ver todas.
- [ ] Pagina completa com filtros.

## Clientes e documentos
- [ ] Meus Clientes mostra somente vinculados.
- [ ] Todos os Clientes exige permissao ampla.
- [ ] Painel do Cliente centraliza informacoes.
- [ ] Documentos ficam no painel do cliente.
- [ ] Documentos tem status e auditoria.
- [ ] Links publicos expiram.

## Propostas, contratos e entrada
- [ ] Proposta: criada, enviada, aceita/recusada, concluida.
- [ ] Contrato: criado, enviado, assinado, concluido.
- [ ] Entrada de Cliente concentra onboarding.
- [ ] Proposta aceita pode ser vinculada.
- [ ] Contrato assinado pode ser vinculado.

## Configuracoes
- [x] Existe `/configuracoes`.
- [x] Usuario pode alterar foto persistida localmente.
- [x] Foto tem validacoes seguras locais.
- [x] Troca de senha somente MASTER.
- [x] Troca gera auditoria via `users.update`.
- [x] Sessoes ativas e ultimos acessos possuem base local em `/me/security`.
- [ ] Validar sessoes ativas, ultimos acessos e dispositivos recentes no EasyPanel.

## Seguranca
- [ ] Acoes sensiveis tem confirmacao.
- [x] Upload de foto valida tipo e tamanho localmente.
- [x] SVG bloqueado para foto.
- [ ] Erros nao vazam detalhes internos.
- [ ] Dados sensiveis nao ficam no localStorage.
- [x] Auto-refresh de access token implementado no Web com retry unico em 401.
- [ ] Validar auto-refresh no EasyPanel apos vencimento real do access token.

## Build
- [x] `npm run lint` passa localmente.
- [x] `npm run build` passa localmente.


## Departamentos, solicitacoes e permissoes
- [ ] Departamento no cadastro de usuario nao e campo livre.
- [ ] Departamento vem de lista controlada pelo backend.
- [ ] Departamento possui chave tecnica unica.
- [ ] Criacao/edicao de departamento e restrita a MASTER/Admin autorizado.
- [ ] Departamento no cadastro/edicao de colaboradores usa lista controlada nas telas React principais.
- [ ] Solicitacoes de Acesso aparecem para todos os usuarios autenticados.
- [ ] Colaborador consegue criar solicitacao e enviar feedback.
- [ ] Gestor consegue aprovar solicitacoes do proprio escopo.
- [ ] T.I./MASTER consegue executar solicitacoes aprovadas.
- [ ] Roles exibem nome amigavel, descricao, risco e permissoes criticas.
- [ ] Permissoes sao agrupadas por modulo e acao.
- [ ] Chave tecnica da permissao aparece como informacao secundaria.
- [ ] Criacao de permissao usa formulario guiado.
- [ ] Atribuicao de permissao critica exige confirmacao.
- [ ] Alteracoes em usuarios, roles, permissoes e departamentos geram auditoria.

## Responsabilidade de clientes por usuario

- [ ] Departamento deixou de ser campo livre na atribuicao de responsabilidade.
- [ ] Existe entidade/tabela propria para responsabilidade cliente x departamento x usuario.
- [ ] Cadastro de cliente permite escolher responsavel operacional por departamento.
- [ ] Cadastro de cliente permite escolher gestor responsavel por departamento, quando aplicavel.
- [x] Painel do Cliente possui guia/secao Equipe/Responsaveis local.
- [x] Painel do Cliente permite atribuicao inicial local de responsavel por departamento, com gestor opcional.
- [ ] Gestor consegue transferir carteira com auditoria.
- [ ] Colaborador visualiza apenas sua carteira conforme escopo.
- [ ] Vencimentos, documentos e planilhas usam a responsabilidade normalizada para filtros.
- [ ] Nao existe dependencia operacional exclusiva de `clients.metadata` para carteira.
- [ ] Toda alteracao de responsavel registra usuario, data, motivo e valor anterior/novo.
