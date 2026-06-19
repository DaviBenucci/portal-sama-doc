
# Análise semântica — legado vs nova arquitetura do Portal Sama

Atualizado em: 2026-06-19 12:00 -03:00

## 1. Objetivo

Este documento compara semanticamente a arquitetura antiga (`portal-sama.zip`) com a nova arquitetura (`portal-sama-api`, `portal-sama-web` e `portal-sama-docs`) e define como reaproveitar regras de negócio, fluxos e decisões de UX do legado sem comprometer a segurança e a manutenibilidade da nova base.

A análise não trata o legado como código descartável. Ele contém decisões funcionais já validadas pelo usuário final, principalmente nos fluxos de cliente, legalização, assinatura ZapSign, navegação e painéis operacionais. A nova arquitetura contém a fundação técnica correta, mas em alguns pontos perdeu coerência de jornada e concentração de contexto.

## 2. Resumo executivo

| Área | Legado | Nova arquitetura | Decisão recomendada |
|---|---|---|---|
| Backend | PHP procedural, endpoints por arquivo, regras espalhadas, mas fluxos completos | NestJS modular, DTOs, Prisma, RBAC, CSRF, testes, auditoria | Manter NestJS, migrar regras faltantes do PHP por domínio |
| Frontend | HTML/CSS/JS, navegação direta, painéis concentrados por contexto | React/Vite, roteamento moderno, componentes reutilizáveis, mas algumas jornadas estão fragmentadas | Manter React, recriar UX validada do legado com componentes modernos |
| Cliente | Painel único com abas: Geral, Responsáveis, Acessos, Documentos, Certificados, Vida da empresa | `/clientes/:clientId/painel` mostra identidade, estatísticas, resumo, responsabilidades e documentos; certificados/acessos/vida ficam fora ou ausentes | Reconstruir o painel do cliente em abas na nova UI |
| ZapSign | Fluxo real integrado: cria documento, envia PDF base64/server-side, guarda `doc_token`, `sign_url`, status remoto e redireciona assinatura | Contratos novos usam assinatura pública interna por token opaco; não há integração ZapSign real implementada no código novo | Migrar ZapSign como provider oficial no `ContractsModule` |
| Segurança | Melhorias pontuais, `.htaccess`, CSRF em alguns endpoints, upload scanner em evolução | Melhor fundação: Helmet, CSRF, JWT curto, refresh cookie, RBAC, scanner, storage privado, auditoria | Consolidar no backend novo e corrigir segredos empacotados |
| Acessórias | Menos estruturado, com caminhos legados e operação manual | Módulos dedicados, sync runs, rate limiter, reconciliação, workspace | Ajustar janela temporal: obrigações desde ano atual; colaboradores desde mês atual |
| Deploy | PHP/Apache/Railway/EasyPanel, serviço PDF separado | Docker separado API/Web, scripts de readiness/backup | Manter deploy separado e documentar gates de produção |

## 3. Arquitetura legada — leitura semântica

A aplicação antiga é composta por páginas estáticas e endpoints PHP organizados por domínio aparente:

```txt
portal-sama/
  api/
    auth.php
    storage.php
    db.php
    legalizacao.php
    zapsign.php
    onboarding.php
    client_documents.php
    certificados.php
    fiscal_workspace.php
    manager_workspace.php
    integra_ai.php
    notifications.php
    access_requests.php
  Client/
  Manager/
  Legalizacao/
  Onboarding/
  DEV/
  Depto/
  DptClient/
  HOME/
  TI/
  Contabil/
```

### 3.1 Pontos fortes do legado

1. **Painel do cliente com alta densidade de contexto.** A tela `Client/painel.html` concentra os dados do cliente em abas funcionais. Isso reduz troca de tela e mantém o usuário no contexto da empresa.
2. **Fluxo ZapSign operacional.** O arquivo `api/zapsign.php` e o fluxo em `api/legalizacao.php` já cobrem criação de documento, autenticação via header, sandbox/prod, signatários, `sign_url`, status remoto e sincronização.
3. **Navegação por área mais previsível para usuários internos.** Mesmo com HTML/JS puro, a organização por `Client`, `Manager`, `Legalizacao`, `Onboarding`, `DEV`, `TI` e `Contabil` reflete o modelo mental da operação.
4. **Legalização mais conectada ao contrato.** O legado trata legalização, propostas, contratos e assinatura como uma jornada única.
5. **Acessos do cliente no mesmo contexto.** A aba `Acessos` do painel antigo deixa claro que credenciais de portais pertencem à empresa e não a uma tela genérica.
6. **Vida da empresa por departamento.** A aba `Vida da empresa` traz contexto operacional sensível e histórico de quem editou.
7. **Indicadores simples e úteis.** Badges de pendência documental, status do cliente, rank e resumo por departamentos ajudam na operação diária.

### 3.2 Pontos fracos do legado

1. **Baixa separação de responsabilidades.** Muitos endpoints concentram roteamento, validação, persistência, autorização e resposta no mesmo arquivo.
2. **Dificuldade de teste automatizado.** A arquitetura procedural torna testes unitários e e2e mais difíceis.
3. **Persistência heterogênea.** Há sinais de SQLite/KV, JSON e MySQL convivendo em camadas diferentes.
4. **Risco de exposição de arquivos sensíveis.** A presença de `.env`, dependências locais, `.git`, logs e artefatos em pacote exige disciplina de empacotamento seguro.
5. **Segurança inconsistente.** Existem mecanismos úteis, mas não há um padrão único e obrigatório para RBAC, DTOs, validação, logs, rate limit e auditoria.
6. **Credenciais de clientes em tela.** O legado permite copiar/revelar senhas de portais; isso deve ser redesenhado com criptografia, auditoria de leitura e política de expiração.

## 4. Nova arquitetura — leitura semântica

A nova arquitetura separa backend e frontend:

```txt
portal-sama-api/
  src/modules/*
  prisma/schema.prisma
  prisma/migrations/*

portal-sama-web/
  src/pages/*
  src/services/*
  src/types/*
  src/schemas/*
  src/components/*
```

### 4.1 Pontos fortes do backend novo

1. **NestJS modular.** Domínios como auth, clients, documents, certificates, contracts, proposals, onboarding, legalization, Acessórias, notifications, transfers e audit estão separados.
2. **Prisma e migrations.** O schema já cobre usuários, clientes, responsabilidades, documentos, certificados, contratos, propostas, onboarding, legalização, Acessórias, notificações, auditoria e Integra-AI.
3. **RBAC explícito.** Existe catálogo de permissões e guards por rota.
4. **Segurança HTTP base.** Helmet, CORS com credenciais, CSRF, JWT de access token, refresh token em cookie e rate limiting global já aparecem na fundação.
5. **Uploads mais seguros.** Documentos passam por allowlist de extensão/MIME, assinatura mágica, SHA-256, quarentena e scanner ClamAV/static rules.
6. **Certificados digitais mais protegidos.** Senhas de certificado são cifradas com AES-256-GCM usando chave derivada de segredo configurado.
7. **Auditoria de eventos.** A API registra ações relevantes em módulo de auditoria.
8. **Operação Acessórias mais madura.** Há sync runs, lock, rate limiter, reconciliação, alias de responsáveis, mapping de entregas e aplicação para workspace.

### 4.2 Pontos fracos do backend novo

1. **ZapSign ausente como provider real.** O novo `ContractsModule` implementa assinatura pública interna por token, mas não replica o provider ZapSign do legado.
2. **Modelo de contrato ainda pequeno para provider externo.** `Contract` tem campos genéricos e `metadata`, mas não há tabela/estrutura própria para `provider`, `providerDocumentToken`, `providerSignUrl`, `providerStatus`, `providerSignedFile`, tentativas e webhooks.
3. **Acessórias com janela temporal ampla.** O código atual indica janela de entregas voltando meses para trás. A nova regra deve limitar obrigações ao ano atual e colaboradores ao mês atual.
4. **Backfill e migração precisam de gates fortes.** Há scripts operacionais, mas o Codex deve seguir ordem rígida e não misturar frontend avançado antes do backend estabilizado.
5. **Risco de segredo empacotado.** Foram encontrados `.env` em pacotes. Isso exige rotação e empacotamento seguro.

### 4.3 Pontos fortes do frontend novo

1. **React com lazy routes.** O roteamento já separa páginas por domínio e melhora performance com carregamento sob demanda.
2. **TanStack Query e Axios.** Há padrão de cache, invalidação e comunicação com a API.
3. **Zod e React Hook Form.** Formulários são tipados e validados.
4. **Zustand para sessão.** O estado de autenticação, token e permissões está centralizado.
5. **Design system inicial.** Há `AppPanel`, `KpiCard`, `ShortcutCard`, `StatusBadge`, `PageShell`, `Sidebar`, `Header` e `PermissionGate`.
6. **Rotas públicas isoladas.** Assinatura, proposta e documentos públicos ficam fora do layout autenticado.

### 4.4 Pontos fracos do frontend novo

1. **Painel do cliente fragmentado.** A tela nova não replica o painel antigo em abas e perdeu seções importantes: acessos, certificados detalhados e vida da empresa no mesmo contexto.
2. **Navegação com excesso de itens equivalentes.** O usuário precisa entender onde está cada parte do cliente, enquanto no legado o painel concentrava a operação por empresa.
3. **Legalização e contratos ainda sem ZapSign externo.** A UI mostra assinatura pública interna, mas não diferencia corretamente provider interno vs ZapSign real.
4. **Certificados digitais globais.** A tela de certificados existe, mas a operação por cliente deveria também aparecer no painel do cliente.
5. **Acessos do cliente ausentes ou não modelados com segurança.** Essa funcionalidade existia no legado e precisa voltar com criptografia e auditoria.

## 5. Reaproveitamento obrigatório do legado

### 5.1 Painel do cliente por abas

A nova tela `/clientes/:clientId/painel` deve virar o centro operacional da empresa, com abas equivalentes e evoluídas:

| Aba nova | Origem no legado | Conteúdo esperado | Backend esperado |
|---|---|---|---|
| Geral | `Client/painel.html#tab-geral` | Dados cadastrais, rank, regime, status, tempo no status, endereço, grupo, resumo por departamentos | `GET /clients/:id/dashboard`, `PATCH /clients/:id` |
| Responsáveis | `tab-departamentos` | Responsável operacional e gestor por departamento, transferência, encerramento, histórico | `GET /clients/:id/assignments`, `POST/PATCH/transfer/end` |
| Acessos | `tab-acessos` | Portais do cliente, login, segredo cifrado, revelação controlada, cópia auditada | Novo módulo `ClientAccessesModule` |
| Documentos | `tab-documentos` | Checklist, pendências, upload, status, revisão, histórico | `DocumentsModule` existente |
| Certificados | `tab-certificados` | Certificados do cliente, download, senha cifrada, rotação, auditoria de leitura | `CertificatesModule` existente com filtro por cliente |
| Vida da empresa | `tab-vida` | Particularidades por departamento, edição pelo responsável, histórico, contributors | `ManagersModule`/`CompanyLifeEntry` existente ou módulo próprio |
| Auditoria | evolução recomendada | Linha do tempo filtrada do cliente: documentos, acessos, certificado, contrato, responsável | `AuditModule` filtrado por entity/clientId |

### 5.2 Navegação

A navegação nova deve ser reduzida em quantidade de entradas visíveis e mais orientada por contexto.

Decisão recomendada:

```txt
Operação
  Início
  Clientes
    Lista
    Painel do cliente
  Departamento
    Modelo
    Vencimentos
  Legalização
    Processos
    Propostas
    Contratos
  Contábil
    Integra-AI
Gestão
  Painel gestor
  Carteira
  Transferências
  Histórico
Administração
  DEV/Admin
  Usuários
  Permissões
  Auditoria
  TI/Acessos
```

O item `Clientes` deve ser o ponto de entrada principal. Ao abrir um cliente, a operação fica nas abas do painel, e não em várias rotas desconectadas.

### 5.3 ZapSign

O fluxo antigo deve ser migrado como provider real:

```txt
Contrato gerado/importado
  -> renderização PDF server-side
  -> criação de documento ZapSign
  -> gravação de doc_token/sign_url/status
  -> retorno do link oficial da ZapSign
  -> sincronização periódica ou webhook
  -> atualização do contrato para SIGNED
  -> gravação de signed_file/evidências
  -> notificação/auditoria
```

Não substituir ZapSign por assinatura interna quando o contrato exigir validade/assinatura externa. A assinatura interna pode continuar como provider `internal`, mas `zapsign` precisa ser provider separado.

### 5.4 Acessos do cliente

A aba antiga de acessos é útil, mas deve ser redesenhada com segurança:

- nunca armazenar senha de portal em texto puro;
- cifrar segredo por registro com AES-256-GCM ou envelope encryption;
- auditar toda leitura/revelação/cópia;
- exigir permissão específica `client_accesses.secret.read` para revelar;
- auto-ocultar segredo no frontend;
- limitar taxa de revelações;
- permitir rotação/expiração de credencial;
- impedir exibição de segredo em logs, payloads genéricos e exportações.

### 5.5 Vida da empresa

O legado modela uma necessidade real: contexto operacional por cliente e departamento. Na nova arquitetura, isso deve ficar dentro da aba `Vida da empresa`, usando `CompanyLifeEntry` ou novo módulo, com:

- escopo por cliente e departamento;
- editor por tópicos;
- histórico imutável;
- permissão por responsável atual/gestor;
- leitura por colaboradores do departamento;
- auditoria de criação/edição.

## 6. Sequência de migração recomendada

1. Congelar inventário do legado e da nova arquitetura.
2. Corrigir empacotamento de segredos e rotacionar credenciais expostas.
3. Finalizar banco, migrations e seeds RBAC.
4. Finalizar backend e contratos de API.
5. Criar frontend mínimo apenas para validar backend e banco.
6. Migrar dados essenciais do legado.
7. Migrar ZapSign.
8. Migrar painel do cliente por abas.
9. Migrar acessos do cliente com criptografia e auditoria.
10. Consolidar frontend completo.
11. Rodar testes unitários, integração, e2e e homologação.
12. Preparar deploy, rollback e go-live.

## 7. Decisões que o Codex não deve inventar

O Codex não deve:

- criar novo fluxo de assinatura diferente de `internal` e `zapsign` sem documentação;
- persistir segredos de cliente/certificado/token em texto puro;
- buscar histórico antigo da Acessórias fora da janela definida;
- criar telas separadas para cada subfunção do cliente se o painel em abas atende melhor;
- alterar permissões sem atualizar matriz RBAC;
- alterar schema sem migration;
- avançar para frontend completo antes de backend e banco estarem validados;
- usar `prisma db push` em ambiente compartilhado ou produção;
- incluir `.env`, dumps, certificados, backups ou logs em pacotes versionáveis.

## 8. Critérios de aceite da análise

A análise será considerada aplicada quando:

- o painel do cliente novo estiver organizado por abas e cobrir as seções do legado;
- ZapSign estiver implementado como provider real no backend novo;
- a integração Acessórias respeitar janela de ano/mês atual;
- a documentação do Codex tiver status por etapa;
- todos os módulos críticos tiverem testes e evidências;
- segredos tiverem sido removidos dos pacotes e rotacionados;
- frontend e backend tiverem validação local e homologação registrada.
