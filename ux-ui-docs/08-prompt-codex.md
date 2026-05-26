# 08 - Prompt para Codex

```md
Voce esta trabalhando no projeto Portal Sama, composto por `portal-sama-web`, `portal-sama-api` e `portal-sama-docs`.

Objetivo: melhorar a UX/UI do frontend sem alterar regras de negocio sensiveis e sem remover controles de seguranca existentes.

Antes de alterar codigo:
1. Leia `portal-sama-docs`.
2. Leia `portal-sama-web/src/app/router.tsx`.
3. Leia `portal-sama-web/src/components/layout/navigation.tsx`.
4. Leia `portal-sama-web/src/components/layout/AppLayout.tsx`, `Sidebar.tsx`, `Header.tsx` e `PageShell.tsx`.
5. Leia `portal-sama-web/src/index.css`.
6. Identifique como permissoes, perfis e departamentos sao representados.

Escopo:
- Nao realizar QA funcional.
- Nao alterar contratos da API sem documentacao previa.
- Nao alterar permissoes existentes de forma insegura.
- Manter expansao da sidebar por hover/foco no desktop.
- Implementar menu hamburguer/drawer no mobile.

Implementar:
1. Refatorar navegacao lateral para:
   - Operacao;
   - Gestao;
   - Entrada de Cliente;
   - T.I;
   - Admin.
2. Manter expansao por hover/foco no desktop.
3. Implementar botao hamburguer no mobile.
4. Padronizar labels.
5. Aplicar regras de visibilidade:
   - Modelo Dpto: todos os departamentos;
   - Meus Clientes: clientes vinculados ao usuario;
   - Todos os Clientes: permissao ampla;
   - Documentos: acessar pelo painel do cliente;
   - Integra-AI: somente Contabil;
   - Vencimentos: todos os departamentos, filtrando escopo;
   - Contratos: Legalizacao, Financeiro e Comercial;
   - Propostas: Financeiro e Comercial;
   - Legalizacao: Legalizacao;
   - DEV: DEV/MASTER ou ambiente autorizado.
6. Mover notificacoes para o header:
   - sino;
   - badge;
   - popover;
   - botao "Ver todas as notificacoes".
7. Transformar Home em Painel do Dia por perfil:
   - Colaborador;
   - Gestor;
   - DEV/Admin.
8. Remover foco em dados tecnicos da Home comum.
9. Simplificar Propostas e Contratos.
10. Concentrar onboarding completo em Entrada de Cliente.
11. Criar base para `/configuracoes`.
12. Permitir foto de perfil com validacoes seguras.
13. Troca de senha somente para MASTER, com auditoria.
14. Reforcar UX de seguranca em documentos, links publicos e auditoria.
15. Trocar campos livres de departamento por select/combobox com dados controlados pelo backend.
16. Garantir que Solicitacoes de Acesso esteja disponivel para todos os usuarios autenticados, com guias e acoes filtradas por perfil/permissao.
17. Melhorar Usuarios e Permissoes:
   - Roles com nome amigavel, descricao, tipo e risco;
   - Permissoes agrupadas por modulo e acao;
   - Chave tecnica como detalhe secundario;
   - Criacao de permissao por formulario guiado;
   - Confirmacao extra para permissoes criticas.
18. Criar ou preparar guia `Departamentos` em Usuarios e Permissoes, restrita a MASTER/Admin autorizado.

Criterios de aceite:
- Sidebar desktop mantem hover/foco.
- Mobile tem menu hamburguer.
- Menu respeita permissoes/departamentos.
- Home varia por cargo.
- Notificacoes ficam no header.
- Documentos ficam no painel do cliente.
- Propostas e contratos têm fluxos simples.
- Entrada de Cliente concentra onboarding.
- Foto de perfil e segura.
- Troca de senha restrita a MASTER.
- Departamento nao e campo livre.
- Solicitacoes de Acesso acessivel a todos os usuarios autenticados.
- Roles e permissoes exibem titulos amigaveis, descricoes e risco.
- Criacao de permissoes e guiada e auditada.
- `npm run lint` e `npm run build` devem passar.
```
