# [PARCIAL] 03 - Home por Perfil

## Objetivo

A Home deve ser um Painel do Dia. Ela precisa mostrar o resumo do que o usuario deve fazer.

## Status de implementacao local - 2026-05-28

- A Home React foi alterada para consumir `GET /api-v2/integrations/acessorias/home-summary`.
- A API consulta o Acessorias usando `ACESSORIAS_BASE_URL` e `ACESSORIAS_TOKEN` do ambiente do backend.
- O token do Acessorias nao vai para o navegador.
- Colaborador, gestor e DEV/Admin possuem blocos diferentes, mantendo os atalhos conforme permissoes.
- Atualizacao complementar 2026-06-01: retorno externo `204 No Content`/corpo vazio e tratado como resumo disponivel sem entregas; falhas retornam diagnostico sanitizado para o painel admin.
- Atualizacao 2026-06-01: o workspace departamental passou a receber vencimentos oficiais de entregas Acessorias sincronizadas; a Home continua pendente de validacao real para consolidar esses vencimentos por perfil.
- Atualizacao 2026-06-01 Central: a Home/atalhos locais podem apontar para `/departamentos/vencimentos`; aceite segue pendente ate validar com dados reais.
- Atualizacao 2026-06-03: os atalhos inteligentes passaram a usar a mesma politica visual da sidebar, filtrando por permissao, departamento, cargo e papel `DEV`.
- Ainda falta validar no EasyPanel com a API real do Acessorias e usuarios reais por perfil.

## Home - Colaborador

### Prioridade
Execucao diaria.

### Cards
- Meus clientes;
- Entregas pendentes;
- Vencimentos proximos;
- Atrasos.

### Blocos
- minhas pendencias;
- entregue pelo Acessorias;
- atalhos inteligentes.

### Wireframe

```text
+-----------+--------------------------------------------------+
| Sidebar   | Inicio                         Sino  Perfil      |
+-----------+--------------------------------------------------+
| Operacao  | Ola, colaborador. Prioridades de hoje.           |
| Gestao    | +-------------+-------------+----------+-------+ |
| Entrada   | | Clientes    | Docs pend.  | Venc.    | Taref | |
| T.I       | | 42          | 8           | 5        | 3     | |
| Admin     | +-------------+-------------+----------+-------+ |
|           | +----------------------+-----------------------+ |
|           | |      Entregue        |   Minhas pendencias   | |
|           | +----------------------+-----------------------+ |
|           | +----------------------+-----------------------+ |
|           | | Obrigações atrasadas |  Vencimentos proximos | |
|           | +----------------------+-----------------------+
|           | +----------------------------------------------+ |
|           | | Atalhos: Meus Clientes | Modelo Dpto | Venc. | |
|           | +----------------------------------------------+ |
+-----------+--------------------------------------------------+
```
Com essas informações sendo obtidas pelo sistema do acessórias

## Home - Gestor

### Prioridade
Acompanhamento e decisao.

### Cards
- Responsaveis;
- Clientes;
- Atrasos;
- Riscos de vencimento.

### Blocos
- gargalos operacionais;
- carteira por colaborador;
- ranking de criticidade.

### Wireframe

```text
+-----------+--------------------------------------------------+
| Sidebar   | Inicio do Gestor                Sino  Perfil     |
+-----------+--------------------------------------------------+
| Gestao    | Visao do departamento e carteira.                |
|           | +-------------+-------------+----------+-------+ |
|           | | Colabs      | Clientes    | Atrasos  | Aprov | |
|           | | 12          | 318         | 9        | 4     | |
|           | +-------------+-------------+----------+-------+ |
|           | +----------------------+-----------------------+ |
|           | | Gargalos             | Transferencias        | |
|           | +----------------------+-----------------------+ |
|           | +----------------------+-----------------------+ |
|           | |Obrigações entregues| Obrigações pendentes/atrasadas| |
|           | +----------------------+-----------------------+ |
|           | +----------------------+-----------------------+ |
|           | |Obrigações entregues dpto.| Obrigações pendentes/atrasadas dpto.| |
|           | +----------------------+-----------------------+ |
|           | +----------------------------------------------+ |
|           | | Carteira por colaborador /    criticidade    | |
|           | +----------------------------------------------+ |
+-----------+--------------------------------------------------+
```
O gestor poderá ver quais obrigações acessórias estão atrasadas e de quais colaboradores as obrigações atrasadas ou entregadas são, e se eles entregaram no prazo ou não.
Aonde ele terá um resumo das obrigações dele também.

## Home - DEV/Admin

### Prioridade
Diagnostico e manutencao.

### Cards
- Integracoes;
- Atrasos;
- Pendencias;
- Clientes.

### Blocos
- diagnostico da integracao Acessorias;
- atrasos e riscos;
- status tecnico.

### Wireframe

```text
+-----------+--------------------------------------------------+
| Sidebar   | Administracao Tecnica           Sino  Perfil     |
+-----------+--------------------------------------------------+
| Admin     | Diagnostico, seguranca e manutencao.             |
|           | +-------------+-------------+----------+-------+ |
|           | | Integracoes | Erros       | Audit.   | Users | |
|           | | OK          | 2           | 18       | 4     | |
|           | +-------------+-------------+----------+-------+ |
|           | +----------------------+-----------------------+ |
|           | | Logs recentes        | Alertas de seguranca  | |
|           | +----------------------+-----------------------+ |
+-----------+--------------------------------------------------+
```
Agora o DEV vera somente as especificacoes mais tecnicas sobre a plataforma/aplicacao.

## Conteudos que nao devem ser foco da Home comum
- quantidade de permissoes;
- quantidade de perfis;
- username tecnico;
- diagnostico interno.

Essas informacoes devem ficar em Configuracoes ou DEV/Admin.
