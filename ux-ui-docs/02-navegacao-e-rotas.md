# 02 - Navegacao e Rotas

## Estrutura recomendada da sidebar

```text
Operacao
- Modelo Dpto
- Meus Clientes
- Todos os Clientes
- Certificados Digitais
- Integra-AI
- Vencimentos
- Contratos
- Propostas
- Legalizacao

Gestao
- Painel do Gestor
- Carteira de Colaboradores
- Transferencias
- Historico

Entrada de Cliente

T.I
- Solicitacoes de Acesso
- Acessos de TI
- Auditoria

Admin
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
| Vencimentos | Todos os departamentos, filtrando por escopo |
| Contratos | Legalizacao, Financeiro e Comercial |
| Propostas | Financeiro e Comercial |
| Legalizacao | Legalizacao |
| Painel do Gestor | Gestores ou usuários com permissão mais elevada |
| Carteira de Colaboradores | Gestores ou usuários com permissão mais elevada |
| Transferencias | Gestores ou usuários com permissão mais elevada |
| Historico | Gestores ou usuários com perissão mais elevada |
| Entrada de Cliente | colaboradores dos departamentos Legalização, financeiro, comercial e gestores de qualquer departamento |
| Solicitacoes de Acesso | todos os usuários de acordo com o escopo. Pois colaboradores enviarão a petição para seus devidos gestores e com a aprovação dos gestores a solicitação chegará aos usuários com permissão MASTER que são do departamento T.I, já os gestores poderão enviar a solicitação por si mesmos, por toda a equipe ou por colaboradores selecionados. |
| Acessos de TI | T.I ou usuários com permissão MASTER |
| Auditoria | T.I e MASTER |
| Usuarios e Permissoes | Admin/MASTER |
| Colaboradores | Admin, RH, gestores e MASTER |
| Configuracoes | Todos acessam pessoal; admin depende de permissao |
| DEV | DEV/MASTER ou ambiente autorizado |

## Sidebar desktop

Manter:
- expansao por hover/foco;
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
