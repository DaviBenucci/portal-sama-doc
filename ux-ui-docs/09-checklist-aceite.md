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

## Navegacao
- [ ] Sidebar desktop mantem hover/foco.
- [ ] Mobile tem botao hamburguer.
- [ ] Menu mobile abre em drawer.
- [ ] Rotas agrupadas corretamente.
- [ ] Itens sem permissao nao aparecem.
- [ ] Departamentos especificos respeitam regras.

## Home
- [ ] Home nao e apenas lista de atalhos.
- [ ] Colaborador ve pendencias reais.
- [ ] Gestor ve carteira, atrasos e riscos.
- [ ] DEV/Admin ve diagnostico tecnico.
- [ ] Cards respeitam permissoes.

## Notificacoes
- [ ] Sino no header.
- [ ] Badge de nao lidas.
- [ ] Popover com recentes.
- [ ] Botao para ver todas.
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
- [ ] Existe `/configuracoes`.
- [ ] Usuario pode alterar foto.
- [ ] Foto tem validacoes seguras.
- [ ] Troca de senha somente MASTER.
- [ ] Troca gera auditoria.

## Seguranca
- [ ] Acoes sensiveis tem confirmacao.
- [ ] Upload valida tipo e tamanho.
- [ ] SVG bloqueado para foto.
- [ ] Erros nao vazam detalhes internos.
- [ ] Dados sensiveis nao ficam no localStorage.
- [x] Auto-refresh de access token implementado no Web com retry unico em 401.
- [ ] Validar auto-refresh no EasyPanel apos vencimento real do access token.

## Build
- [ ] `npm run lint` passa.
- [ ] `npm run build` passa.


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
- [ ] Painel do Cliente possui guia Equipe/Responsaveis.
- [ ] Gestor consegue transferir carteira com auditoria.
- [ ] Colaborador visualiza apenas sua carteira conforme escopo.
- [ ] Vencimentos, documentos e planilhas usam a responsabilidade normalizada para filtros.
- [ ] Nao existe dependencia operacional exclusiva de `clients.metadata` para carteira.
- [ ] Toda alteracao de responsavel registra usuario, data, motivo e valor anterior/novo.
