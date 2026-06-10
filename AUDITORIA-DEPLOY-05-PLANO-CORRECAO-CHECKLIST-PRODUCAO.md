# Auditoria Deploy - Plano de Correção e Checklist de Produção

## Etapa 12 - Plano recomendado

## Decisão final

Status atual: **Não pronto para usuários reais**.

Pode seguir para homologação controlada somente depois de corrigir os bloqueios críticos de segurança e deixar os gates automatizados verdes.

## Ordem de correção

### 1. Segurança de segredos

Prioridade: crítica.

Ações:

1. Apagar ou isolar `C:\Users\Sama Contabilidade\Desktop\portal-sama-api.zip`.
2. Recriar o ZIP sem `.env`.
3. Garantir `.env`, `.env.local`, certificados, `.key`, `.pem`, `.p12`, `.pfx` e tokens fora de qualquer pacote.
4. Rotacionar segredos potencialmente expostos:
   - JWT access/refresh/CSRF.
   - Banco.
   - Acessórias.
   - VAPID/Web Push.
   - `CERTIFICATE_ENCRYPTION_KEY`.
   - Tokens de serviços legais/externos.
5. Registrar evidência da rotação.

Critério de aceite:

- Nenhum ZIP/artefato contém `.env`.
- `git ls-files` segue rastreando somente `.env.example`.
- Segredos antigos não funcionam mais.

### 2. Corrigir escopo de clientes

Prioridade: crítica.

Ações:

1. Ajustar `ClientsService.applyReadScope`.
2. Ajustar `ClientsService.assertCanReadClient`.
3. Garantir que `list`, `getById` e `getDashboard` usem a mesma regra.
4. Cobrir por testes unitários e E2E/API:
   - departamento não vê cliente fora do escopo;
   - contábil não vê cliente sem atribuição contábil;
   - legalização não vê cliente fora do escopo;
   - cliente vê somente o próprio cadastro;
   - admin/dev/gestor respeitam regra definida pelo negócio.

Critério de aceite:

- Testes negativos passam.
- QA manual confirma isolamento entre departamentos.

### 3. Fechar E2E API

Prioridade: alta.

Ações:

1. Isolar o ambiente de teste da `.env` local.
2. Definir `COOKIE_DOMAIN=''` e `COOKIE_SECURE=false` nos E2E locais.
3. Repetir `npm.cmd run test:e2e`.
4. Manter `COOKIE_SECURE=true` e domínio correto em produção.

Critério de aceite:

- E2E API 100% verde.
- Teste CSRF continua falhando quando header/cookie faltam.

### 4. Fechar E2E web

Prioridade: alta.

Ações:

1. Definir contrato correto da Home atual.
2. Atualizar testes ou UI para alinhar textos/atalhos:
   - `Integracao` vs `Diagnostico Acessorias`.
   - `Entregue pelo Acessorias` vs `Baixas sincronizadas`.
3. Mockar/estabilizar `/notifications/stream` e `/me/security`.
4. Repetir `npm.cmd run test:e2e`.

Critério de aceite:

- Playwright local verde.
- Screenshots sem sobreposição em desktop e mobile.

### 5. Dependências

Prioridade: média.

Ações:

1. Corrigir vulnerabilidade moderada `qs` na API.
2. Rodar:
   - `npm.cmd audit --audit-level=moderate`
   - `npm.cmd run build`
   - `npm.cmd test -- --runInBand`
   - `npm.cmd run test:e2e`

Critério de aceite:

- Audit sem vulnerabilidades moderadas ou superiores.

### 6. Infraestrutura e operação

Prioridade: alta.

Ações:

1. Subir ambiente de homologação com MySQL real.
2. Rodar `npm.cmd run prisma:migrate:status`.
3. Rodar `npm.cmd run prisma:migrate:deploy`.
4. Rodar seed/bootstrap RBAC.
5. Rodar `npm.cmd run ops:readiness`.
6. Validar ClamAV:
   - `npm.cmd run ops:clamav:update`
   - EICAR bloqueado.
   - `SAMA_UPLOAD_SCAN_MODE=strict`.
7. Validar backup/restore:
   - `npm.cmd run ops:backup:create`
   - `npm.cmd run ops:backup:verify`
   - `npm.cmd run ops:restore:drill`

Critério de aceite:

- Readiness sem failed.
- ClamAV strict validado.
- Restore drill documentado.

### 7. Homologação funcional real

Prioridade: alta.

Ações:

1. Criar usuários reais DEV, Gestor, Colaborador e Cliente.
2. Rodar roteiro manual do relatório de QA.
3. Rodar Web Push com VAPID real em Chrome/Edge.
4. Rodar Acessórias com token real e backfill controlado.
5. Validar documentos/contratos/propostas/onboarding de ponta a ponta.

Critério de aceite:

- Todos os perfis validam escopo e permissões.
- Nenhum dado de cliente aparece fora do escopo.
- Logs de auditoria registram ações críticas.

## Checklist de Go/No-Go

| Item | Status atual | Go quando |
| --- | --- | --- |
| ZIP sem `.env` | No-Go | Artefatos limpos e segredos rotacionados. |
| Escopo de clientes | No-Go | Testes negativos e QA manual verdes. |
| API build/lint/unit | Go | Já passou; repetir após correções. |
| API E2E | No-Go | 100% verde. |
| Web build/lint/contratos | Go | Já passou; repetir após correções. |
| Web E2E | No-Go | 100% verde. |
| DB/migrações | No-Go | `migrate status/deploy` verdes no alvo. |
| RBAC seed | No-Go | Permissões conferidas no banco real. |
| ClamAV strict | No-Go | EICAR bloqueado em produção/homologação. |
| Web Push | No-Go | VAPID real e navegador real validados. |
| Acessórias | No-Go | Token real, backfill e incremental validados. |
| Backup/restore | No-Go | Backup verificado e restore drill concluído. |
| UX mobile | Parcial | Rodada manual sem quebra visual. |

## Recomendação de release

1. Corrigir itens críticos.
2. Rodar gates locais até todos passarem.
3. Subir homologação com banco e HTTPS reais.
4. Executar checklist de Go/No-Go.
5. Liberar primeiro para grupo piloto interno.
6. Monitorar logs, auditoria, notificações, uploads, Acessórias e consumo de banco.
7. Só então liberar para clientes reais.

