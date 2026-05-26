# Portal Sama - Docs UX/UI e Configuracoes

Pacote de documentação em Markdown para orientar a melhoria das telas do Portal Sama.

## Escopo
- UX/UI das telas.
- Organizacao da navegacao.
- Home como painel operacional.
- Wireframes textuais.
- Configuracoes da conta.
- Seguranca aplicada a interface.
- Governanca de departamentos, solicitacoes, roles e permissoes.
- Prompt para implementacao no Codex.

## Fora do escopo
- QA funcional.
- Teste do dominio.
- Alteracao de contratos da API.
- Remocao de permissoes existentes.

## Uso no QA de UI/UX

Depois da integracao com o Acessorias, este pacote deve orientar a rodada de melhoria visual e experiencia de uso do Portal Sama. O arquivo `09-checklist-aceite.md` sera usado como ultima verificacao de UI/UX antes da liberacao do sistema para os usuarios.

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

Decisao importante: a sidebar desktop deve manter a expansao por hover/foco. No mobile, deve existir botao hamburguer.


Decisao adicional: departamentos nao devem ser campos livres; solicitacoes de acesso devem estar disponiveis para todos os usuarios autenticados; roles e permissoes devem ser configuraveis por MASTER com interface mais descritiva, agrupada e auditada.
