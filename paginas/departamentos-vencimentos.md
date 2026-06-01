# [PARCIAL] Central de Vencimentos Departamental

## Status de migracao React/API v2

- **Atualizacao 2026-06-01:** criada rota React `/departamentos/vencimentos`, implementada em `portal-sama-web/src/pages/departments/DepartmentDueCenterPage.tsx`.
- **Fonte de dados:** reutiliza `GET /api-v2/departments/workspace`, que ja consolida vencimentos de calendario e entregas Acessorias sincronizadas.
- **Escopo:** leitura operacional por departamento/mes, com filtros por origem (`Calendario` ou `Acessorias`), status de prazo e busca por empresa, CNPJ, obrigacao ou coluna.
- **Navegacao:** adicionada na sidebar como `Central venc.`, nos atalhos da Home e como botao em `/departamentos/modelo`.
- **Seguranca aplicada:** rota protegida por `departments.workspace.read`; o backend mantem escopo por departamento/colaborador no workspace.
- **Status real:** implementada localmente; nao homologada ate validacao no EasyPanel com dados reais do Acessorias e regras reais de calendario.

## Objetivo da pagina

Concentrar vencimentos operacionais em uma tela dedicada, sem misturar configuracao de regras com acompanhamento diario.

A tela deve ajudar a responder:

- o que vence no mes selecionado;
- quais vencimentos estao atrasados;
- quais vencem nos proximos dias;
- quais itens vieram do Acessorias;
- quais itens vieram das regras manuais de calendario;
- em qual empresa/coluna cada vencimento impacta.

## Fluxo atual

1. Usuario acessa `/departamentos/vencimentos`.
2. A tela consulta `GET /api-v2/departments/workspace`.
3. O frontend deduplica vencimentos presentes nas celulas e no carrossel.
4. O usuario filtra por departamento, mes, origem, status ou busca textual.
5. A tabela exibe empresa, CNPJ, obrigacao, coluna, origem, prazo e situacao.

## Pendencias de homologacao

- Validar totais contra dados reais sincronizados do Acessorias.
- Validar regras reais de calendario manual no EasyPanel.
- Conferir escopo com usuarios de diferentes departamentos.
- Confirmar comportamento em telas mobile.
- Decidir se a Central precisara de endpoint proprio para volumes maiores apos dados reais.
