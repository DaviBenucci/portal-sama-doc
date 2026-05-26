# 03 - Home por Perfil

## Objetivo

A Home deve ser um Painel do Dia. Ela precisa mostrar o resumo do que o usuario deve fazer.

## Home - Colaborador

### Prioridade
Execucao diaria.

### Cards
- Meus clientes;
- Documentos pendentes;
- Vencimentos proximos;
- Tarefas abertas.

### Blocos
- minhas pendencias;
- vencimentos do departamento;
- ultimos clientes acessados;
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
- Colaboradores;
- Clientes;
- Atrasos;
- Aprovacoes.

### Blocos
- gargalos operacionais;
- transferencias pendentes;
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
- Erros;
- Auditoria;
- Usuarios.

### Blocos
- logs recentes;
- alertas de seguranca;
- status tecnico;
- parametros criticos.

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
Agora o MASTER verá somente as específicações mais técnicar sobre a plataforma/aplicação

## Conteudos que nao devem ser foco da Home comum
- quantidade de permissoes;
- quantidade de perfis;
- username tecnico;
- diagnostico interno.

Essas informacoes devem ficar em Configuracoes ou DEV/Admin.
