# [PARCIAL] 04 - Wireframes Textuais

## Layout desktop autenticado

```text
+------------+-----------------------------------------------------+
| Sidebar    | Header: Titulo da pagina     Sino   Perfil          |
| expansivel +-----------------------------------------------------+
|            | Breadcrumb / descricao curta                         |
| Operacao   |                                                     |
| Gestao     | +-----------+-----------+-----------+-------------+ |
| Entrada    | | Card      | Card      | Card      | Acao CTA    | |
| T.I        | +-----------+-----------+-----------+-------------+ |
| Admin      |                                                     |
|            | +-----------------------------------------------+   |
|            | | Conteudo principal                            |   |
|            | | Tabela, formulario, calendario ou painel       |   |
|            | +-----------------------------------------------+   |
+------------+-----------------------------------------------------+
```

## Layout mobile

```text
+------------------------------------+
| Menu  Portal Sama       Sino Perfil|
+------------------------------------+
| Titulo da pagina                   |
| Descricao curta                    |
+------------------------------------+
| Card em coluna                     |
| Card em coluna                     |
+------------------------------------+
| Conteudo principal                 |
+------------------------------------+
```

## Menu mobile aberto

```text
+----------------------+
| Portal Sama       X  |
+----------------------+
| Operacao          v  |
|   Inicio            |
|   Meus clientes     |
| Gestao            >  |
| Entrada de Cliente> |
| T.I               v  |
|   Solicitacoes      |
|   Acessos TI        |
| Admin             >  |
+----------------------+
```

## Login

```text
+-----------------------------+-----------------------------+
| Area institucional          | Formulario de acesso         |
| Logo Sama                   | Entrar no portal             |
| Ambiente seguro             | Usuario                      |
| Mensagem curta              | Senha                        |
| Gradiente/ilustracao        | [ Entrar ]                   |
|                             | Esqueci minha senha          |
|                             | Aviso de acesso restrito     |
+-----------------------------+-----------------------------+
```

## Notificacoes no header

```text
Header:
+----------------------------------------------------------+
| Titulo da pagina                       Sino(5)  Perfil   |
+----------------------------------------------------------+

Popover:
+--------------------------------------+
| Notificacoes recentes                |
+--------------------------------------+
| Documento pendente - Cliente A       |
| Vencimento amanha - Cliente B        |
| Certificado vence em 7 dias          |
+--------------------------------------+
| [ Ver todas as notificacoes ]        |
+--------------------------------------+
```

## Meus Clientes

```text
+-----------+--------------------------------------------------+
| Sidebar   | Meus Clientes                   Sino  Perfil     |
+-----------+--------------------------------------------------+
| Operacao  | Clientes sob sua responsabilidade                 |
|           | +----------+-------------+-------------+--------+ |
|           | | Ativos   | Pendencias  | Docs pend.  | Venc.  | |
|           | +----------+-------------+-------------+--------+ |
|           | Filtros: nome/CNPJ | Status | Regime | Depto     |
|           | +----------------------------------------------+ |
|           | | Cliente | CNPJ | Status | Proxima acao | Acoes | |
|           | +----------------------------------------------+ |
+-----------+--------------------------------------------------+
```

## Todos os Clientes

```text
+-----------+--------------------------------------------------+
| Sidebar   | Todos os Clientes               Sino  Perfil     |
+-----------+--------------------------------------------------+
| Operacao  | Visao geral dos clientes da empresa              |
|           | +----------+----------+------------+----------+  |
|           | | Total    | Ativos   | Arquivados | Em risco |  |
|           | +----------+----------+------------+----------+  |
|           | Filtros: Grupo | Cidade | UF | Regime | Rank      |
|           | +----------------------------------------------+ |
|           | | Cliente | Grupo | Depto | Status | Acoes      | |
|           | +----------------------------------------------+ |
+-----------+--------------------------------------------------+
```

## Painel do Cliente

```text
+-----------+--------------------------------------------------+
| Sidebar   | Cliente: Empresa Exemplo        Sino  Perfil     |
+-----------+--------------------------------------------------+
| Operacao  | CNPJ, regime, responsavel e status               |
|           |                                                  |
|           | Abas: Visao geral | Documentos | Certificados    |
|           |                                                  |
|           | +----------+----------+-------------+----------+ |
|           | | Status   | Docs     | Certificado | Prox venc| |
|           | +----------+----------+-------------+----------+ |
|           | +----------------------+-----------------------+ |
|           | | Dados e responsaveis | Atividades recentes   | |
|           | +----------------------+-----------------------+ |
+-----------+--------------------------------------------------+
```

## Documentos do Cliente

```text
+--------------------------------------------------------------+
| Cliente: Empresa Exemplo                                     |
+--------------------------------------------------------------+
| Abas: Visao geral | Documentos | Certificados | Historico     |
+--------------------------------------------------------------+
| Documentos                                                   |
| +-------------+-------------+------------+----------------+  |
| | Pendentes   | Aprovados   | Rejeitados | Links publicos |  |
| +-------------+-------------+------------+----------------+  |
| Filtros: Tipo | Status | Competencia | Enviado por | Busca   |
| +----------------------------------------------------------+ |
| | Documento | Tipo | Status | Confidencialidade | Acoes     | |
| +----------------------------------------------------------+ |
+--------------------------------------------------------------+
```

## Entrada de Cliente

```text
+-----------+--------------------------------------------------+
| Sidebar   | Entrada de Cliente              Sino  Perfil     |
+-----------+--------------------------------------------------+
| Entrada   | Processo de ativacao de novo cliente             |
| Cliente   | Etapas: Dados > Proposta > Contrato > Docs       |
|           | +----------------------+-----------------------+ |
|           | | Dados do processo    | Pendencias            | |
|           | +----------------------+-----------------------+ |
|           | +----------------------------------------------+ |
|           | | Proposta aceita e contrato assinado vinculados| |
|           | +----------------------------------------------+ |
+-----------+--------------------------------------------------+
```

## Configuracoes

```text
+-----------+--------------------------------------------------+
| Sidebar   | Configuracoes                   Sino  Perfil     |
+-----------+--------------------------------------------------+
| Admin     | Preferencias, seguranca e parametros             |
|           | Abas: Minha conta | Seguranca | Notificacoes     |
|           | +----------------------+-----------------------+ |
|           | | Foto e dados         | Preferencias/seguranca| |
|           | | Upload seguro        | Sessoes, senha, MFA   | |
|           | +----------------------+-----------------------+ |
+-----------+--------------------------------------------------+
```

## Pagina publica - Proposta

```text
+------------------------------------------------------------+
| Logo Sama                              Ambiente seguro      |
+------------------------------------------------------------+
| Proposta para Empresa Exemplo                              |
| Resumo, servicos, valores, condicoes e validade             |
+------------------------------------------------------------+
| [ Aceitar proposta ]       [ Recusar / Solicitar ajuste ]   |
+------------------------------------------------------------+
| Registro de seguranca: link pessoal, data, IP e validade    |
+------------------------------------------------------------+
```

## Estados padrao

```text
Estado vazio:
+------------------------------------------+
| Nenhum item encontrado                   |
| Use os filtros ou crie um novo registro. |
| [ Acao principal ]                       |
+------------------------------------------+

Erro:
+------------------------------------------+
| Nao foi possivel carregar esta informacao|
| Tente novamente ou contate o suporte.    |
| [ Tentar novamente ]                     |
+------------------------------------------+
```
