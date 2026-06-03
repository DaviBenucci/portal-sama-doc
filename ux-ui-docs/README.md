# [PARCIAL] Portal Sama - Docs UX/UI e Configuracoes

Pacote de documentacao em Markdown para orientar a melhoria das telas do Portal Sama.

## Escopo
- UX/UI das telas.
- Organizacao da navegacao.
- Home como painel operacional.
- Wireframes textuais.
- Configuracoes da conta.
- Seguranca aplicada a interface.
- Governanca de departamentos, solicitacoes, roles e permissoes.
- Responsabilidade de clientes por usuario, departamento e gestor.
- Prompt para implementacao no Codex.

## Fora do escopo
- QA funcional.
- Teste do dominio.
- Alteracao de contratos da API sem documentacao previa.
- Remocao de permissoes existentes.
- Escrita externa em sistemas integrados sem fase aprovada.

## Uso no QA de UI/UX

Depois da integracao com o Acessorias, este pacote deve orientar a rodada de melhoria visual e experiencia de uso do Portal Sama. O arquivo `09-checklist-aceite.md` sera usado como ultima verificacao de UI/UX antes da liberacao do sistema para os usuarios.

## Status local - 2026-06-02

- Navegacao agrupada, drawer mobile, notificacoes no header e `/configuracoes` foram implementados localmente no React.
- Validacao local passou com lint/build Web e smoke Playwright.
- O aceite real ainda depende de EasyPanel, usuarios reais por perfil, notificacoes reais e validacao do avatar no backend/storage real.

## Status local - 2026-06-03

- Avatar persistido foi implementado localmente com backend/storage privado, validacao real de arquivo, remocao de metadados e leitura autenticada.
- Sessoes ativas e ultimos acessos foram implementados localmente em `/configuracoes` por `GET /api-v2/me/security`, usando `refresh_tokens` e `audit_logs` sem expor token/hash/cookie.
- Validacao local passou com teste focado, lint/build API e lint/build Web.
- O aceite real ainda depende de deploy no EasyPanel, storage real, auditoria persistida e usuarios reais.

## Arquivos
1. `01-visao-geral.md`
2. `02-navegacao-e-rotas.md`
3. `03-home-por-perfil.md`
4. `04-wireframes-textuais.md`
5. `05-modulos-operacao.md`
6. `06-gestao-entrada-ti-admin.md`
7. `07-configuracoes-e-seguranca.md`
8. `08-prompt-codex.md`
9. `09-checklist-aceite.md`
10. `10-departamentos-solicitacoes-permissoes.md`
11. `11-responsabilidade-clientes-usuarios.md`

## Decisoes importantes

- A sidebar desktop deve manter a expansao por hover/foco.
- No mobile, deve existir botao hamburguer.
- Departamentos nao devem ser campos livres.
- Solicitacoes de acesso devem estar disponiveis para todos os usuarios autenticados.
- Roles e permissoes devem ser configuraveis por MASTER com interface mais descritiva, agrupada e auditada.
- Responsabilidade de cliente nao deve depender exclusivamente de `clients.metadata`; deve evoluir para entidade propria cliente x departamento x usuario.
