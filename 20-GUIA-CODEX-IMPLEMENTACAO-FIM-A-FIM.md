# Guia Codex — implementação fim a fim do Portal Sama

Atualizado em: 2026-06-19

## Regra rígida e inegociável

O Codex somente pode avançar para a próxima etapa quando a etapa anterior estiver **CONCLUÍDA**, com validação técnica e evidência registrada. Não é permitido pular etapa, compensar pendência em fase futura ou marcar como concluído algo que apenas compilou parcialmente.

Se surgir erro, incompatibilidade, falha de teste, ausência de endpoint, divergência de regra ou qualquer imprevisto, o Codex deve adicionar uma sub-etapa corretiva dentro da etapa atual, por exemplo:

```txt
Etapa 6.3.1 - Corrigir filtro de colaboradores antigos retornados pela API Acessórias
Status: PENDENTE -> EM_EXECUÇÃO -> CONCLUÍDA
Evidência: teste X criado, comando Y executado, resultado Z
```

A etapa principal só pode receber `CONCLUÍDA` depois que todas as sub-etapas corretivas estiverem concluídas.

## Status aceitos

| Status | Significado | Pode avançar? |
|---|---|---|
| `PENDENTE` | etapa ainda não iniciada | não |
| `EM_EXECUÇÃO` | etapa em alteração ou validação | não |
| `BLOQUEADA` | depende de correção, decisão ou dado ausente | não |
| `CONCLUÍDA` | implementada, validada e evidenciada | sim |

## Ordem estratégica

A ordem correta é: banco de dados e backend primeiro; frontend apenas como smoke simples para validar backend; frontend final somente depois do backend aprovado. Essa ordem evita criar telas bonitas sobre contratos instáveis ou endpoints incompletos.

## Regra específica Acessórias

A integração Acessórias deve buscar obrigações/listas de entregas a partir do ano atual e colaboradores/responsáveis a partir do mês atual. Dados antigos retornados pela API externa devem ser filtrados antes de persistir no workspace operacional.

## Evidência obrigatória por etapa

Para cada etapa, registrar:

```txt
Fase:
Etapa:
Status anterior:
Status final:
Arquivos alterados:
Comandos executados:
Resultado dos comandos:
Testes adicionados/alterados:
Pendências:
Observações de segurança:
```

## Fase 0 — Preparação, escopo e regra rígida de execução

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 0.1 — Criar checkpoint documental inicial

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Ler este guia, registrar data, branch atual, commit base e objetivo do ciclo.

**Como validar:** Checklist criado em `portal-sama-docs/docs/20-GUIA-CODEX-IMPLEMENTACAO-FIM-A-FIM.md` ou arquivo de acompanhamento.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 0.2 — Confirmar regra de avanço bloqueante

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Registrar que nenhuma etapa pode ser pulada e que cada imprevisto vira sub-etapa na etapa atual.

**Como validar:** Regra copiada para o checklist ativo com status `CONCLUÍDA`.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 0.3 — Inventariar repositórios

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Confirmar presença de `portal-sama-api`, `portal-sama-web`, `portal-sama-docs` e legado `portal-sama.zip` extraído para consulta.

**Como validar:** Lista de caminhos e commits registrada.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 0.4 — Bloquear edição fora do escopo

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Definir os arquivos esperados para a fase atual antes de alterar código.

**Como validar:** Arquivos impactados registrados antes do primeiro patch.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 0.5 — Executar leitura de documentação existente

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Ler arquitetura, RBAC, rotas públicas, runbooks, testes e docs novas 15 a 19.

**Como validar:** Resumo de decisões registrado sem criar solução nova não documentada.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 0.6 — Validar que não há secrets em pacote de trabalho

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Procurar `.env`, dumps, chaves, certificados e tokens antes de empacotar qualquer coisa.

**Como validar:** Lista de exclusões criada; se houver segredo, registrar sub-etapa de rotação.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 0.7 — Definir matriz de evidência

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Criar padrão para evidenciar cada etapa: comando, resultado, arquivos alterados, pendências.

**Como validar:** Template de evidência anexado ao checklist.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 0.8 — Congelar decisões de produto reaproveitadas do legado

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Registrar que painel do cliente por abas e ZapSign legado são requisitos, não sugestões.

**Como validar:** Decisão registrada como obrigatória.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 0:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 1 — Infra local, Docker e banco de dados primeiro

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 1.1 — Validar versões de runtime

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Conferir Node >= 22, npm, Docker, MySQL e sistema operacional do ambiente.

**Como validar:** Comandos de versão registrados.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 1.2 — Criar ou revisar Docker Compose inicial

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Configurar MySQL, volume persistente e rede local para API; não incluir frontend completo ainda.

**Como validar:** `docker compose config` válido.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 1.3 — Criar `.env.example` da API

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Listar variáveis sem valores reais: DB, JWT, CSRF, storage, scanner, ZapSign, Acessórias, CORS.

**Como validar:** Arquivo existe e não contém segredo real.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 1.4 — Criar `.env.local.example` do web

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Incluir apenas `VITE_API_BASE_URL` e flags não sensíveis.

**Como validar:** Arquivo existe sem token/segredo.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 1.5 — Subir banco limpo

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Iniciar MySQL local e validar conexão.

**Como validar:** Conexão testada sem usar produção.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 1.6 — Configurar usuário de banco com menor privilégio

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Evitar root na aplicação; criar usuário local/homolog específico.

**Como validar:** Usuário e permissões documentados.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 1.7 — Validar charset/collation

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Garantir `utf8mb4` e collation compatível com português.

**Como validar:** Consulta de banco registrada.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 1.8 — Criar rotina de reset local seguro

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Definir como recriar banco local sem apagar produção por engano.

**Como validar:** Comando documentado com aviso de ambiente.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 1.9 — Validar health básico da API contra banco

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Subir API mínima e chamar health/readiness.

**Como validar:** Resposta de health registrada.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 1.10 — Marcar fase apenas se banco estiver operacional

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Checar que não há pendência de conexão, env ou compose.

**Como validar:** Todos os passos anteriores com status `CONCLUÍDA`.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 1:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 2 — Prisma, schema e modelagem de domínio

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 2.1 — Rodar validação Prisma atual

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Executar `npm run prisma:validate` na API.

**Como validar:** Saída sem erro registrada.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 2.2 — Revisar modelos existentes

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Mapear User, Client, Department, Assignment, Document, Certificate, Contract, Acessórias, Audit.

**Como validar:** Tabela de modelos atualizada na documentação.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 2.3 — Modelar provider de assinatura

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Adicionar enums/modelos de assinatura externa ZapSign conforme doc 18.

**Como validar:** Migration criada e revisada.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 2.4 — Modelar signatários de contrato

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Criar tabela ou relação para signers com campos normalizados.

**Como validar:** Campos indexáveis definidos fora de JSON genérico.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 2.5 — Modelar acessos do cliente

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Criar domínio para acessos/credenciais com segredo criptografado ou referência segura.

**Como validar:** Modelo aprovado com permissões e auditoria.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 2.6 — Revisar Vida da empresa

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Decidir se modelos legados `CompanyLifeEntry`/`CompanyHistoryEntry` serão mantidos ou normalizados.

**Como validar:** Decisão registrada com endpoint alvo.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 2.7 — Ajustar índices críticos

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Verificar índices de status, clientId, dueAt, externalId, docToken, deletedAt.

**Como validar:** Schema revisado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 2.8 — Criar migration incremental

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Gerar migration pequena e legível, sem alterações destrutivas inesperadas.

**Como validar:** Migration versionada e revisada.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 2.9 — Testar migration em banco limpo

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Aplicar migration do zero.

**Como validar:** Banco limpo sobe sem erro.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 2.10 — Testar migration em banco com dados fake

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Aplicar migration sobre seed local.

**Como validar:** Dados preservados.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 2.11 — Atualizar seed RBAC

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Adicionar permissões novas para ZapSign, acessos, vida da empresa e revelação sensível.

**Como validar:** Seed roda sem duplicar registros.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 2.12 — Gerar Prisma Client

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Executar `npm run prisma:generate`.

**Como validar:** Client gerado sem erro.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 2:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 3 — Fundação de segurança do backend

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 3.1 — Validar bootstrap da API

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Confirmar prefixo `api-v2`, Helmet, CORS, ValidationPipe, filtros e interceptors.

**Como validar:** Arquivo de bootstrap revisado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 3.2 — Revisar schema de ambiente

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Garantir validação de env obrigatória por ambiente.

**Como validar:** API falha cedo se secret obrigatório ausente.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 3.3 — Revisar JWT e refresh token

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Confirmar access curto, refresh rotativo e hash no banco.

**Como validar:** Testes de auth verdes.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 3.4 — Revisar CSRF

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Garantir assert em mutações autenticadas de todos os módulos.

**Como validar:** Rotas mutáveis auditadas.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 3.5 — Revisar guards de permissão

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Confirmar `JwtAuthGuard` + `PermissionsGuard` em controllers de domínio.

**Como validar:** Lista de rotas sem permissão justificada.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 3.6 — Adicionar rate limit em rotas sensíveis

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Login, refresh, upload, assinatura pública, ZapSign sync/webhook, Acessórias sync.

**Como validar:** Decorators/config testados.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 3.7 — Padronizar erros

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Retornar erro com code/message sem stack e sem segredo.

**Como validar:** Testes de erro atualizados.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 3.8 — Adicionar sanitização de logs

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Garantir que tokens, CPF completo, base64 e senha não sejam logados.

**Como validar:** Teste ou revisão registrada.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 3.9 — Revisar auditoria

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Confirmar eventos para mutações sensíveis e downloads.

**Como validar:** Eventos mínimos implementados.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 3.10 — Executar testes de segurança base

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar unitários de auth, csrf, permissions e env.

**Como validar:** Comandos verdes.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 3:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 4 — Clientes, colaboradores e responsabilidades

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 4.1 — Validar CRUD/listagem de clientes

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Conferir filtros, paginação, status, soft delete e escopo.

**Como validar:** Testes de clients verdes.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 4.2 — Validar dashboard de cliente

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Garantir endpoint consolidado para cabeçalho e KPIs do painel.

**Como validar:** Contrato do endpoint documentado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 4.3 — Completar dados da aba Geral

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Incluir campos necessários do legado: atividade, grupo, status timestamps quando disponíveis.

**Como validar:** DTO retorna dados esperados.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 4.4 — Validar colaboradores ativos

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Listar apenas usuários ativos para novas responsabilidades.

**Como validar:** Teste impede usuário inativo/desligado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 4.5 — Validar responsabilidades por departamento

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Criar, editar, transferir e encerrar assignment.

**Como validar:** Testes cobrem cada ação.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 4.6 — Exigir motivo em transferência/encerramento

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Garantir motivo quando a regra exigir rastreabilidade.

**Como validar:** DTO/serviço rejeitam motivo vazio se obrigatório.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 4.7 — Auditar alteração de responsabilidade

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Registrar before/after sanitizado.

**Como validar:** Audit log criado nos testes.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 4.8 — Criar endpoints da Vida da empresa

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Implementar GET/POST/PATCH/history por cliente se ainda ausentes.

**Como validar:** Endpoints protegidos por permissão.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 4.9 — Criar endpoints de acessos do cliente

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Implementar listagem segura e solicitação/gestão sem revelar segredo por padrão.

**Como validar:** Segredos mascarados em listagem.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 4.10 — Testar escopo mine/all

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Usuário comum não acessa cliente fora de responsabilidade.

**Como validar:** E2E/API confirma 403/404 seguro.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 4:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 5 — Documentos, certificados e storage privado

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 5.1 — Revisar configuração de storage

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Garantir diretório/bucket privado e permissões corretas.

**Como validar:** Arquivo não acessa por URL pública direta.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 5.2 — Validar upload de documentos

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Extensão, MIME, magic bytes, tamanho e hash.

**Como validar:** Testes de upload verde.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 5.3 — Validar scanner de malware

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Configurar modo strict/best_effort e quarentena.

**Como validar:** Falha de scanner tem comportamento definido.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 5.4 — Validar download protegido

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Download deve exigir auth/permissão ou token público específico.

**Como validar:** Cabeçalhos `no-store` e `Content-Disposition` corretos.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 5.5 — Completar checklist de documentos do cliente

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Requisitos por cliente/departamento e badge de pendência.

**Como validar:** Endpoint retorna pendências.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 5.6 — Validar certificados digitais

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Upload, storage, metadata, validade e status.

**Como validar:** Testes de certificados verdes.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 5.7 — Validar criptografia de senha de certificado

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** AES-GCM/KMS, chave fora do banco e sem retorno em listagem.

**Como validar:** Teste garante senha ausente na listagem.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 5.8 — Implementar revelação auditada

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Endpoint específico para revelar senha quando autorizado.

**Como validar:** Audit log obrigatório.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 5.9 — Criar alertas de vencimento

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Certificados próximos do vencimento devem aparecer no dashboard.

**Como validar:** Regra testada.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 5.10 — Executar testes do módulo

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar unitários e integração de documentos/certificados.

**Como validar:** Comandos verdes.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 5:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 6 — Integração Acessórias com janela temporal correta

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 6.1 — Ler runbook Acessórias

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Revisar docs existentes e regras novas deste pacote.

**Como validar:** Resumo registrado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 6.2 — Corrigir janela padrão de obrigações

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Alterar fluxo operacional para buscar desde 01/01 do ano atual, não últimos 12 meses.

**Como validar:** Teste valida `DtInitial` no ano atual.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 6.3 — Corrigir janela de colaboradores/responsáveis

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Buscar colaboradores a partir do mês atual ou filtrar pós-consulta por mês atual.

**Como validar:** Teste impede importação de responsável desligado antigo.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 6.4 — Separar backfill histórico de sync operacional

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Backfill deve exigir datas explícitas e não alimentar workspace atual sem revisão.

**Como validar:** Endpoints e docs refletem diferença.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 6.5 — Validar filtros pós-consulta

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Quando a API externa não filtrar, descartar dados antigos antes de persistir.

**Como validar:** Teste com payload antigo não persiste dados obsoletos.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 6.6 — Validar rate limit e lock

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Evitar sync concorrente e respeitar limite externo.

**Como validar:** Teste de lock concorrente.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 6.7 — Auditar sync

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Registrar início, fim, contadores, janelas e falhas sanitizadas.

**Como validar:** Audit/sync run criado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 6.8 — Notificar falhas críticas

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Falha parcial deve avisar DEV/admin conforme serviço existente.

**Como validar:** Teste/mocking de notificação.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 6.9 — Atualizar docs do Acessórias

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Documentar ano atual para obrigações e mês atual para colaboradores.

**Como validar:** Runbook revisado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 6.10 — Executar testes Acessórias

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar specs do módulo e e2e se existir.

**Como validar:** Comandos verdes.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 6:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 7 — Contratos, Legalização e ZapSign

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 7.1 — Revisar módulo atual de contratos

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Entender create/generate/render-pdf/send-signature/public-signature.

**Como validar:** Mapa de endpoints registrado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 7.2 — Adicionar provider de assinatura

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Implementar `INTERNAL` e `ZAPSIGN` no domínio e DTOs.

**Como validar:** Contrato compila e migration aplicada.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 7.3 — Criar cliente ZapSign

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Implementar HTTP client isolado com base URL por ambiente e token sanitizado.

**Como validar:** Unitários do client verdes.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 7.4 — Implementar criação de documento ZapSign

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Ler PDF do storage, converter base64 e enviar signers.

**Como validar:** Mock de criação passa.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 7.5 — Implementar persistência do envelope

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Salvar doc token, sign_url, status, signer e payload seguro.

**Como validar:** Banco contém envelope único.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 7.6 — Implementar sincronização

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Consultar ZapSign e atualizar envelope/contrato.

**Como validar:** Mock assinado muda contrato para SIGNED.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 7.7 — Implementar webhook

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rota pública com secret/rate limit e confirmação ativa via GET doc.

**Como validar:** Webhook inválido rejeitado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 7.8 — Bloquear assinatura interna para ZapSign

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** POST público interno deve falhar se provider for ZapSign.

**Como validar:** Teste E2E cobre bloqueio.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 7.9 — Auditar eventos ZapSign

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Criar eventos started/succeeded/failed/signed/webhook.

**Como validar:** Audit log validado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 7.10 — Atualizar frontend mínimo de contrato

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Exibir provider, status e link retornado pelo backend.

**Como validar:** Smoke manual ou E2E.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 7.11 — Executar suite de contratos

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar unitários/e2e de contratos e ZapSign mock.

**Como validar:** Comandos verdes.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 7:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 8 — Backend completo, readiness e qualidade antes do frontend final

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 8.1 — Executar lint da API

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar `npm run lint`.

**Como validar:** Sem erros.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 8.2 — Executar build da API

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar `npm run build`.

**Como validar:** Build sem erro.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 8.3 — Executar testes unitários

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar `npm test -- --runInBand`.

**Como validar:** Suite verde.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 8.4 — Executar testes E2E

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar `npm run test:e2e`.

**Como validar:** Suite verde ou pendências registradas como sub-etapas.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 8.5 — Executar readiness

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar `npm run ops:readiness -- --soft --json`.

**Como validar:** Readiness aprovado ou correções registradas.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 8.6 — Executar verificação de schema

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar `npm run ops:schema:check` quando aplicável.

**Como validar:** Sem drift bloqueante.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 8.7 — Executar secret check

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar validação de secrets/rotação quando script existir.

**Como validar:** Nenhum segredo exposto.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 8.8 — Criar backup local/homolog

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Executar rotina de backup antes de qualquer dado real.

**Como validar:** Backup validado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 8.9 — Executar restore drill

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Restaurar backup em alvo isolado.

**Como validar:** Restore comprovado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 8.10 — Congelar contrato da API

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Atualizar Swagger/docs para o frontend consumir sem adivinhação.

**Como validar:** Docs de API atualizadas.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 8:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 9 — Frontend mínimo apenas para validar backend

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 9.1 — Criar tela simples de health

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Frontend deve chamar health/readiness e mostrar resultado.

**Como validar:** Tela funciona localmente.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 9.2 — Criar login simples

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Validar auth, CSRF, refresh e `/me`.

**Como validar:** Login/logout funcionam.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 9.3 — Criar tela simples de clientes

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Listar clientes sem layout final.

**Como validar:** Dados reais do backend aparecem.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 9.4 — Criar tela simples de contrato

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Criar/consultar contrato e testar provider interno/ZapSign com mock.

**Como validar:** Fluxo backend validado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 9.5 — Criar tela simples Acessórias

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Disparar sync controlado e exibir contadores.

**Como validar:** Janela temporal correta exibida.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 9.6 — Criar tela simples de upload

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Enviar documento válido/inválido para testar backend.

**Como validar:** Backend aceita/rejeita corretamente.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 9.7 — Não polir layout ainda

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Evitar gastar tempo em UX antes do backend estar fechado.

**Como validar:** Nenhum componente final grande criado nesta fase.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 9.8 — Registrar evidências

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Screenshots ou logs de smoke das telas simples.

**Como validar:** Evidências anexadas.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 9:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 10 — Base final do frontend e design system operacional

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 10.1 — Revisar roteador

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Confirmar rotas públicas, privadas, lazy loading e fallback.

**Como validar:** Rotas documentadas.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 10.2 — Reorganizar navegação

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Agrupar por Operação, Cliente, Legalização, Onboarding, T.I., Gestão, Administração e DEV.

**Como validar:** Menu fica mais próximo do legado e do fluxo real.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 10.3 — Criar componentes de abas

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Componente acessível com URL query string.

**Como validar:** Testes de navegação por aba.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 10.4 — Criar estados padrão

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Loading, empty, error, permission denied, rate limit.

**Como validar:** Componentes reutilizáveis.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 10.5 — Padronizar cards e painéis

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** KPI, info card, tabela, timeline, drawer, modal.

**Como validar:** Componentes documentados.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 10.6 — Padronizar forms

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** React Hook Form/Zod quando aplicável e erros claros.

**Como validar:** Form com validação consistente.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 10.7 — Padronizar permissões no frontend

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Usar `PermissionGate` sem substituir validação backend.

**Como validar:** Ações sensíveis escondidas/bloqueadas.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 10.8 — Revisar API client

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Tratamento de 401, 403, 409, 422, 429 e refresh.

**Como validar:** Testes de contrato passam.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 10.9 — Revisar segurança visual

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Não logar payload, mascarar segredos, confirmar ações sensíveis.

**Como validar:** Checklist frontend seguro concluído.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 10:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 11 — Painel do cliente final reaproveitando o legado

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 11.1 — Criar `ClientDashboardHeader`

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Cabeçalho com voltar, nome, razão, CNPJ, rank, status e ações.

**Como validar:** Renderiza com dados reais.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 11.2 — Criar `ClientDashboardTabs`

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Abas Geral, Responsáveis, Acessos, Documentos, Certificados e Vida da empresa.

**Como validar:** Query string controla aba ativa.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 11.3 — Implementar aba Geral

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Dados completos da empresa e edição rápida com permissão.

**Como validar:** Usuário sem permissão não edita.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 11.4 — Implementar aba Responsáveis

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Tabela e ações de criar/editar/transferir/encerrar.

**Como validar:** Queries invalidam corretamente.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 11.5 — Implementar aba Acessos

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Listagem segura e solicitação/gestão de acessos, sem senha exposta por padrão.

**Como validar:** Revelação exige permissão/auditoria.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 11.6 — Implementar aba Documentos

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Checklist, upload, pendências e histórico no contexto do cliente.

**Como validar:** Badge de pendência funciona.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 11.7 — Implementar aba Certificados

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Lista, vencimento, upload, download e revelação segura quando permitida.

**Como validar:** Senha não aparece na listagem.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 11.8 — Implementar aba Vida da empresa

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Timeline por departamento e histórico conforme legado.

**Como validar:** Criar/consultar entrada funciona.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 11.9 — Ajustar responsividade

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Painel deve funcionar em desktop e notebook menor.

**Como validar:** Teste visual em tamanhos principais.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 11.10 — Criar E2E do painel

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Abrir cliente e percorrer todas as abas.

**Como validar:** Playwright verde.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 11:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 12 — Telas de módulos após painel do cliente

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 12.1 — Refinar tela de Clientes

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Busca, filtros, status, acesso rápido ao painel.

**Como validar:** Usuário chega ao painel em um clique.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 12.2 — Refinar tela de Documentos

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Visão global sem substituir aba do cliente.

**Como validar:** Filtros por cliente/departamento.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 12.3 — Refinar tela de Certificados

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Visão global de vencimentos e riscos.

**Como validar:** Alertas coerentes com aba do cliente.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 12.4 — Refinar Legalização

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Processos, propostas e contratos com status claros.

**Como validar:** Fluxo completo navegável.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 12.5 — Refinar tela de contratos ZapSign

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Exibir provider, ambiente, sign_url, sync e histórico.

**Como validar:** E2E de provider passa.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 12.6 — Refinar Onboarding

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Entrada de cliente e documentos públicos sem conflitar com operação recorrente.

**Como validar:** Fluxo público validado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 12.7 — Refinar T.I./Acessos

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Solicitações e gestão centralizada de acessos.

**Como validar:** Permissões respeitadas.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 12.8 — Refinar Gestão

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Transferências, calendário, histórico e painel gestor.

**Como validar:** Gestor vê apenas escopo permitido.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 12.9 — Refinar Administração

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Usuários, roles, permissões, auditoria e configurações.

**Como validar:** Somente admin acessa.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 12:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 13 — Testes frontend, acessibilidade e segurança

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 13.1 — Executar lint do web

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar `npm run lint`.

**Como validar:** Sem erros.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 13.2 — Executar build do web

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar `npm run build`.

**Como validar:** Build sem erro.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 13.3 — Executar testes de contrato

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar `npm run test`.

**Como validar:** Contratos de API/client schemas verdes.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 13.4 — Executar smoke auth

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar `npm run smoke:auth` se configurado.

**Como validar:** Auth OK.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 13.5 — Executar smoke permissions

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar `npm run smoke:permissions` se configurado.

**Como validar:** Permissões OK.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 13.6 — Executar smoke público

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar `npm run smoke:public` se configurado.

**Como validar:** Rotas públicas OK.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 13.7 — Executar Playwright

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Rodar `npm run test:e2e`.

**Como validar:** E2E verde.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 13.8 — Testar usuário sem permissão

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Garantir ações sensíveis escondidas e backend retorna 403 se chamado manualmente.

**Como validar:** Cenário documentado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 13.9 — Testar XSS básico

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Inserir texto com HTML/script em campos permitidos e validar escape/sanitização.

**Como validar:** Não executa script.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 13.10 — Testar responsividade

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Validar layouts principais.

**Como validar:** Evidências visuais.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 13:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 14 — Migração de dados do legado

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 14.1 — Mapear dados legados

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Identificar clientes, documentos, certificados, contratos, ZapSign, vida da empresa e acessos.

**Como validar:** Inventário criado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 14.2 — Classificar dados sensíveis

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Separar segredo, documento, histórico e cadastro.

**Como validar:** Plano de migração seguro.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 14.3 — Criar scripts idempotentes

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Scripts devem poder rodar novamente sem duplicar dados.

**Como validar:** Chaves de dedupe definidas.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 14.4 — Migrar clientes em ambiente local

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Importar amostra e validar CNPJ/idEmpresa.

**Como validar:** Contadores batem.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 14.5 — Migrar responsabilidades

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Vincular departamentos e colaboradores ativos.

**Como validar:** Inativos não viram responsáveis ativos.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 14.6 — Migrar documentos

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Mover para storage privado com hash e metadata.

**Como validar:** Downloads protegidos funcionam.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 14.7 — Migrar certificados

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Criptografar senhas ou exigir redefinição se origem insegura.

**Como validar:** Nenhuma senha em texto puro.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 14.8 — Migrar contratos/ZapSign

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Preservar doc_token/status/sign_url quando existente.

**Como validar:** Sync funciona após migração.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 14.9 — Migrar vida da empresa

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Preservar histórico por departamento.

**Como validar:** Timeline exibe dados legados.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 14.10 — Migrar acessos

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Migrar somente com criptografia e política de revelação.

**Como validar:** Listagem mascarada.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 14.11 — Validar amostragem com usuário

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Comparar legado e novo para clientes críticos.

**Como validar:** Divergências registradas e corrigidas.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 14:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 15 — Homologação integrada

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 15.1 — Preparar ambiente de homologação

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Banco, storage, API, web, secrets sandbox e domínio.

**Como validar:** Readiness homolog OK.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 15.2 — Rodar migrations homolog

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Aplicar migrations com backup prévio.

**Como validar:** Migration sem erro.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 15.3 — Rodar seeds RBAC homolog

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Criar roles/permissões/admin inicial.

**Como validar:** Usuários de teste acessam conforme matriz.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 15.4 — Validar fluxo cliente

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Listar clientes, abrir painel, abas, documentos, certificados, vida da empresa.

**Como validar:** Checklist de UX aprovado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 15.5 — Validar fluxo Legalização

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Proposta, contrato, render PDF, assinatura interna e ZapSign sandbox.

**Como validar:** Contrato assina/sincroniza.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 15.6 — Validar Acessórias

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Sync operacional com ano atual/mês atual e contadores corretos.

**Como validar:** Sem dados antigos indevidos.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 15.7 — Validar segurança

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** CSRF, RBAC, uploads, scanner, segredos, auditoria.

**Como validar:** Checklist segurança aprovado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 15.8 — Validar backups

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Criar backup homolog e fazer restore drill.

**Como validar:** Restore comprovado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 15.9 — Validar performance básica

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Listagens, painel e uploads dentro de tempo aceitável.

**Como validar:** Métricas registradas.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 15.10 — Registrar aceite de homologação

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Listar pendências não bloqueantes e bloqueantes.

**Como validar:** Bloqueantes zerados antes de produção.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 15:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Fase 16 — Preparação e go-live de produção

**Critério de entrada:** todas as fases anteriores devem estar `CONCLUÍDA`.

### Etapa 16.1 — Congelar janela de deploy

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Definir responsável, horário e plano de rollback.

**Como validar:** Plano registrado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 16.2 — Rotacionar secrets

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Garantir secrets finais em produção e remover expostos.

**Como validar:** Checklist de rotação concluído.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 16.3 — Criar backup pré-go-live

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Backup do banco/storage antes de migration.

**Como validar:** Backup e restore drill recentes.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 16.4 — Aplicar migrations produção

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Executar com logs e monitoramento.

**Como validar:** Migration concluída.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 16.5 — Deploy backend

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Subir API, validar health/readiness e logs.

**Como validar:** API saudável.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 16.6 — Deploy frontend

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Subir web apontando para API correta.

**Como validar:** Web carrega e login funciona.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 16.7 — Validar smoke produção

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Login, painel cliente, documentos, contratos, ZapSign, Acessórias sem ações destrutivas.

**Como validar:** Smoke aprovado.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 16.8 — Monitorar erros

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Observar 5xx, auth, sync, storage, scanner e logs.

**Como validar:** Sem alerta crítico.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 16.9 — Comunicar go-live

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Registrar versão e notas internas.

**Como validar:** Comunicação feita.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

### Etapa 16.10 — Fechar documentação

**Status inicial:** `PENDENTE`  
**Status permitido ao finalizar:** `CONCLUÍDA`  
**Se houver imprevisto:** criar sub-etapa `Etapa X.Y - Correção de imprevisto`, resolver, validar e só então concluir a etapa principal.

**O que fazer:** Atualizar status final das fases e pendências.

**Como validar:** Guia marcado como concluído somente se tudo anterior estiver verde.

**Evidência mínima:** registrar arquivos alterados, comando executado, resultado e pendências. Se algum comando falhar, criar sub-etapa corretiva antes de continuar.

**Critério de saída da Fase 16:** todas as etapas desta fase estão `CONCLUÍDA`, sem falha bloqueante, com evidências registradas.

## Comandos de validação finais

Executar no final, sem pular testes anteriores:

```bash
cd portal-sama-api
npm run prisma:validate
npm run prisma:generate
npm run lint
npm run build
npm test -- --runInBand
npm run test:e2e
npm run ops:readiness -- --soft --json

cd ../portal-sama-web
npm run lint
npm run build
npm run test
npm run test:e2e
```

## Critério final de aceite

O projeto só pode ser considerado concluído quando:

- banco e migrations estiverem validados em banco limpo e homologação;
- backend estiver completo e com testes verdes;
- Acessórias respeitar ano atual para obrigações e mês atual para colaboradores;
- ZapSign estiver migrado, testado, auditado e sem secrets expostos;
- painel do cliente tiver abas Geral, Responsáveis, Acessos, Documentos, Certificados e Vida da empresa;
- frontend consumir endpoints reais, sem mocks permanentes;
- documentos e certificados usarem storage privado e controles sensíveis;
- RBAC/CSRF/rate limit/auditoria estiverem ativos;
- backup e restore drill estiverem comprovados;
- pacote final não contiver `.env`, tokens, dumps, storage, certificados, logs ou `.git`;
- homologação integrada estiver aprovada;
- go-live tiver smoke test aprovado e plano de rollback registrado.
