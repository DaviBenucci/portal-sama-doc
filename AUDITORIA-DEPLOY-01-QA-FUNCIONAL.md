# Auditoria Deploy - QA Funcional

## Etapa 1 - Leitura e entendimento

Documentação ativa lida:

- `Testes-da-aplicação-DEPLOY.md`
- `00-LEIA-ME-PARA-IA-MVP.md`
- `01-ESTADO-ATUAL-CODIGO-DOCUMENTACAO.md`
- `02-PLANO-FECHAMENTO-MVP.md`
- `03-CONTRATO-ACESSORIAS-OPERACIONAL.md`
- `04-DIVERGENCIAS-DOCS-CODIGO.md`
- `05-PROMPT-CODEX-FECHAMENTO-MVP.md`
- `06-NOTIFICACOES-WEB-PUSH-MVP.md`
- `07-PROMPT-CODEX-PIPELINE-ACESSORIAS-PERSISTENCIA-PUSH-DEV.md`

Entendimento consolidado: o MVP atual cobre autenticação/RBAC/CSRF/auditoria/sessão, clientes, colaboradores, documentos, certificados, propostas, contratos, onboarding, solicitações de acesso, Integra-AI, Web Push e Acessórias com persistência local. A documentação antiga em `docs/_arquivo` foi tratada como referência histórica, não como contrato ativo.

## Etapa 2 - Estrutura da aplicação

Projetos encontrados:

| Projeto | Stack | Status estrutural |
| --- | --- | --- |
| `portal-sama-api` | NestJS, TypeScript, Prisma, MySQL | Estrutura modular, scripts de build/teste/prontidão, Prisma schema e módulos de domínio. |
| `portal-sama-web` | React, Vite, TypeScript, Playwright | Rotas protegidas, store de auth, serviços HTTP, testes contratuais e E2E. |
| `portal-sama-docs` | Markdown | Documentação ativa e histórica separadas. |

## Etapa 3 - Execução local

Resultado:

- API subiu localmente via `node dist/src/main.js`.
- `GET http://127.0.0.1:3000/api-v2/health` retornou 200 com `storage: up` e `database: not_checked`.
- Web subiu via Vite.
- `GET http://127.0.0.1:5173/login` retornou 200.

Observação: a validação local foi feita com `PRISMA_CONNECT_ON_BOOT=false`, porque o banco configurado não estava acessível.

## Etapa 4 - Fluxos funcionais avaliados

| Fluxo | Evidência | Resultado | Pendência |
| --- | --- | --- | --- |
| Login/sessão/CSRF | Código e testes automatizados; store web guarda token em memória | Parcialmente aprovado | Testar com usuários reais e HTTPS real. |
| Perfis DEV/ADMIN | E2E web com sessão mockada para Admin | Parcial | Falha em expectativa da Home. |
| Perfil colaborador/departamento | E2E web com sessão mockada para Ana Fiscal | Parcial | Falha em expectativa da Home/Acessórias. |
| Perfil cliente | API E2E cobre endpoints protegidos e permissões | Parcial | Homologar login real e escopo real. |
| Clientes | Código analisado | Reprovado | Corrigir vazamento de escopo em `clients.read`. |
| Responsabilidades/departamentos | Código analisado | Parcialmente aprovado | O módulo de atribuições tem checagens, mas depende do escopo de clientes corrigido. |
| Documentos/upload/download | Unit/e2e e código analisados | Parcialmente aprovado | ClamAV strict e fluxo real ainda pendentes. |
| Certificados digitais | Código analisado | Parcial | Exigir chave dedicada de criptografia em produção. |
| Propostas/contratos/onboarding | Cobertura unitária/e2e indireta | Parcial | Homologar fluxo ponta a ponta com usuários reais. |
| Solicitações de acesso | Código e rotas protegidas | Parcial | Homologar aprovação/reprovação real. |
| Integra-AI | Testes unitários e contratos | Parcial | Homologar arquivos reais e exportação Dominio. |
| Acessórias | Código e testes de rate limit/backfill | Parcial | Homologar token real, ListAll, incremental e backfill. |
| Web Push | API e web têm implementação | Reprovado no gate | E2E API falha por CSRF/cookie no ambiente atual; VAPID real pendente. |

## Etapa 5 - Matriz DEV/Gestor/Colaborador

| Perfil | Pode ser aprovado agora? | Observação |
| --- | --- | --- |
| DEV/ADMIN | Não para produção | Precisa corrigir segurança do ZIP, escopo de clientes e validar ambiente real. |
| Gestor/MANAGER | Não para produção | Tem permissão ampla, mas precisa validar restrições por escopo e ações críticas. |
| Colaborador/DEPARTMENT/ACCOUNTING/LEGALIZATION | Não para produção | Vazamento de leitura de clientes impacta diretamente este perfil. |
| Cliente | Não para produção | Fluxo real de login/documentos/contratos/assinaturas não foi executado em ambiente alvo. |

## Etapa 6 - Cenários manuais obrigatórios antes do deploy

1. Login real com DEV, Gestor, Colaborador e Cliente.
2. Renovação automática de sessão após expiração do access token.
3. Logout limpa sessão e refresh cookie.
4. CSRF bloqueia POST/PATCH/DELETE sem header/cookie.
5. Colaborador vê somente clientes do próprio escopo.
6. Colaborador não acessa detalhe/dashboard de cliente fora do escopo.
7. Gestor acessa somente escopo previsto.
8. Cliente acessa somente os próprios documentos/contratos/propostas.
9. Upload de PDF válido passa em modo `strict`.
10. Upload com arquivo malicioso de teste EICAR é bloqueado.
11. Upload público por token expirado/revogado é bloqueado.
12. Download exige permissão e escopo.
13. Cadastro de certificado `.p12/.pfx` válido e rejeição de arquivo inválido.
14. Proposta pública: visualização, aceite, ajuste e recusa.
15. Contrato público: assinatura, token inválido e expiração.
16. Onboarding com anexos, status e notificações.
17. Solicitação de acesso com aprovação/reprovação.
18. Integra-AI com OFX/CSV real e exportação validada.
19. Acessórias ListAll/backfill/incremental com token real.
20. Web Push com VAPID real em Chrome/Edge.
21. Notificações SSE sem backend indisponível.
22. Auditoria registra login, upload, alterações críticas e erros relevantes.
23. Backup criado, verificado e restore drill executado.
24. Migrações aplicadas sem drift.
25. RBAC seed confere permissões esperadas.

## Conclusão funcional

O QA funcional automatizado dá boa base, mas **não fecha release**. O sistema deve ir para uma rodada de correção e homologação controlada antes de qualquer usuário real.

