# Matriz de Permissões RBAC

Status base: catálogo default validado em 2026-06-19 com 91 permissões e 9 papéis no MySQL descartável.

## Perfis principais

| Perfil | Uso esperado | Estado |
| --- | --- | --- |
| DEV | Administração técnica, usuários, auditoria, integrações e validação crítica | Smoke real mínimo aprovado |
| ADMIN | Administração operacional ampla | Pendente matriz completa |
| MANAGER | Gestão de equipe, transferências, histórico e carteira | Pendente matriz completa |
| DEPARTMENT | Operação por departamento e carteira atribuída | Pendente matriz completa |
| TI | Solicitações, acessos e auditoria técnica conforme escopo | Pendente matriz completa |
| LEGALIZATION | Legalização, propostas e contratos | Pendente matriz completa |
| ACCOUNTING | Integra-AI e rotinas contábeis | Pendente matriz completa |
| CLIENT | Área externa/cliente, se mantida no escopo | Pendente validação de escopo |

## Matriz mínima de aceite

| Perfil | Deve acessar | Deve negar |
| --- | --- | --- |
| DEV | `/users`, `/audit/logs`, `/integrations/acessorias/*`, `/notifications` | Dados fora de política de retenção |
| ADMIN | `/users`, `/clients`, `/documents`, `/contracts`, `/proposals` | Operações técnicas reservadas a DEV, se aplicável |
| MANAGER | `/managers`, `/transfers`, `/clients`, `/documents`, `/access-requests/manager/approvals` | Administração global de usuários sem permissão explícita |
| DEPARTMENT | `/home`, `/clients?scope=mine`, `/documents`, `/departments/vencimentos` | Clientes fora do escopo operacional |
| TI | `/access-requests`, `/audit/logs` conforme permissão | Módulos financeiros/contábeis sem permissão |
| LEGALIZATION | `/legalization/processes`, `/contracts`, `/proposals` | Administração técnica |
| ACCOUNTING | `/accounting/integra-ai`, dados necessários ao processamento | Auditoria/admin sem permissão |
| Anônimo | Rotas públicas por token válido | Qualquer rota autenticada |

## Testes obrigatórios

- [x] Anônimo recebe `401` em `/auth/me`, `/users` e `/documents`.
- [x] Usuário autorizado validado em smoke real mínimo.
- [ ] Matriz completa com usuário real por perfil.
- [ ] Testes `403` para usuário autenticado sem permissão.
- [ ] Testes de escopo por objeto: cliente, documento, certificado, contrato e proposta.
- [x] Escopo por objeto aplicado e testado para gestão de links públicos de documentos por MANAGER.
- [ ] Testes de token expirado/revogado em rotas públicas.

## Observações

RBAC por permissão não substitui autorização por objeto. Endpoints com `:id` precisam verificar se o usuário pode acessar aquele recurso específico, não apenas se possui uma permissão geral como `clients.read`.
