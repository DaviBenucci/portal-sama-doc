# [PARCIAL] 02 - Navegacao e Rotas

## Status local - 2026-06-02

- Sidebar React reorganizada por grupos Operacao, Gestao, Entrada, T.I e Admin.
- Desktop mantem expansao por hover/foco e itens filtrados por permissao.
- Mobile usa botao hamburguer no header e drawer com backdrop.
- Notificacoes foram movidas para o header com sino, badge, popover e botao para `/notificacoes`.
- Validacao local: Playwright smoke cobre sidebar desktop, drawer mobile e ausencia de overflow.
- Aceite real ainda depende de EasyPanel com usuarios reais por perfil e matriz de permissoes real.

## Status local - 2026-06-03

- Grupos da sidebar agora funcionam como submenus colapsaveis.
- A visibilidade visual das paginas usa politica central por permissao, departamento, cargo e papel tecnico.
- `DEV` e o unico papel de permissao total da aplicacao; `MASTER` deixa de ser usado como equivalente.
- Os atalhos da Home usam a mesma politica visual da sidebar.
- Aceite real ainda depende de EasyPanel com usuarios reais por perfil/departamento/cargo.

## Estrutura recomendada da sidebar

```text
⬇ {Nome do departamento do usuário}
- Modelo Dpto
- Meus Clientes
- Todos os Clientes
- Certificados Digitais
- Integra-AI
- Vencimentos
- Contratos
- Propostas
- Legalizacao

⬇Gestão
- Painel do Gestor
- Carteira de Colaboradores
- Transferencias
- Historico

Entrada de Cliente

⬇T.I
- Solicitacoes de Acesso
- Acessos de TI
- Auditoria

⬇Admin
- Usuarios e Permissoes
- Colaboradores
- Configuracoes
- DEV
```

## Regras de exibicao

| Item | Regra |
|---|---|
| Modelo Dpto | Todos os departamentos, conforme permissao |
| Meus Clientes | Clientes vinculados ao usuario logado |
| Todos os Clientes | Permissao ampla de visualizacao da empresa |
| Documentos | Nao deve ser item principal; acessar pelo painel do cliente |
| Certificados Digitais | Todos os departamentos, o certificado DEVE ser vinculado a um cliente, aonde as informações "cliente/id do cliente devem ser preenchidos automáticamente" e escolher departamento deve ser retirado pois todos os dptos terão acesso aos certificados digitais. |
| Integra-AI | Somente departamento Contabil |
| Vencimentos | Todos os departamentos, filtrando por escopo; rota local `/departamentos/vencimentos` pendente de aceite real |
| Contratos | Legalizacao, Financeiro e Comercial |
| Propostas | Financeiro e Comercial |
| Legalizacao | Legalizacao |
| Painel do Gestor | Gestores ou usuários com permissão mais elevada |
| Carteira de Colaboradores | Gestores ou usuários com permissão mais elevada |
| Transferencias | Gestores ou usuários com permissão mais elevada |
| Historico | Gestores ou usuários com perissão mais elevada |
| Entrada de Cliente | colaboradores dos departamentos Legalização, financeiro, comercial e gestores de qualquer departamento |
| Solicitacoes de Acesso | Todos os usuarios autenticados devem acessar para pedir acesso, reportar problema ou enviar feedback. O conteudo interno muda por perfil: colaborador cria e acompanha; gestor aprova no proprio escopo; T.I./DEV executa; auditor consulta. |
| Acessos de TI | T.I ou usuarios com permissao DEV |
| Auditoria | T.I, Auditor ou DEV |
| Usuarios e Permissoes | DEV |
| Colaboradores | Admin, RH, gestores e DEV |
| Configuracoes | Todos acessam pessoal; admin depende de permissao |
| DEV | DEV ou ambiente autorizado |

## Sidebar desktop

Manter:
- expansao por hover/foco;
- grupos clicaveis com submenus colapsaveis;
- icones no estado recolhido;
- transicao suave;
- acessibilidade por teclado.

## Mobile

Implementar:
- botao hamburguer no header;
- menu em drawer;
- cards em coluna;
- filtros colapsaveis.

## Notificacoes

As notificacoes devem sair da sidebar e ir para o header:
- icone de sino;
- badge de nao lidas;
- popover com recentes;
- botao "Ver todas as notificacoes";
- pagina completa para historico.


## Regras revisadas de navegacao por solicitacoes

Solicitacoes de Acesso nao devem ser tratadas como area exclusiva de T.I. Todos os usuarios autenticados precisam conseguir abrir uma solicitacao ou enviar feedback.

```text
Todos os usuarios autenticados:
- Minhas solicitacoes;
- Nova solicitacao;
- Feedback do sistema.

Gestores:
- Aprovacoes pendentes;
- Solicitacoes da equipe.

T.I./DEV:
- Execucao de solicitacoes;
- Controle tecnico de acessos;
- Auditoria de acessos.
```

A sidebar pode exibir `Solicitacoes` para todos. As guias internas devem ser filtradas por permissao.

## Regra revisada para departamentos

Departamento nao deve ser campo livre em nenhuma tela de cadastro de usuario, colaborador, cliente ou filtro critico. Deve vir de entidade controlada pelo backend.

```text
Correto: selecionar departamento em lista controlada.
Incorreto: digitar departamento em input livre.
```
