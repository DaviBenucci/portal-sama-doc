# Segurança e governança — Portal Sama

Atualizado em: 2026-06-19

## 1. Objetivo

O Portal Sama manipula documentos empresariais, certificados digitais, contratos, dados cadastrais, históricos de clientes, credenciais de acesso e integrações externas. Por isso, a segurança deve ser tratada como requisito central de arquitetura, não como ajuste final.

Este documento define controles mínimos para backend, frontend, banco, storage, integrações, logs, auditoria, backup, operação e resposta a incidentes.

## 2. Classificação de dados

| Classe | Exemplos | Tratamento |
|---|---|---|
| Público controlado | proposta pública, tela pública de assinatura | token opaco, expiração, rate limit. |
| Interno | lista de clientes, dashboard, responsáveis | autenticação, RBAC, auditoria em mutações. |
| Confidencial | documentos empresariais, contratos, histórico operacional | storage privado, download protegido, logs sem conteúdo. |
| Altamente sensível | certificados digitais, senhas de certificado, credenciais de cliente | criptografia, permissão específica, auditoria de revelação. |
| Segredo técnico | JWT secret, DB password, ZapSign token, VAPID keys | secret manager, rotação, nunca versionar. |

## 3. Ameaças principais

1. Vazamento de `.env`, tokens ou chaves.
2. Download indevido de documentos.
3. Upload de arquivo malicioso.
4. Exposição de senha/certificado digital.
5. Acesso de colaborador fora do escopo do cliente/departamento.
6. CSRF em mutações autenticadas.
7. XSS em HTML de contratos, histórico ou documentos.
8. Webhook externo falso.
9. Importação excessiva do Acessórias com dados antigos ou responsáveis desligados.
10. Falha de backup/restore antes de migration.
11. Logs contendo PII, token ou segredo.
12. Contrato assinado após alteração indevida de conteúdo.

## 4. Autenticação e sessão

Controles obrigatórios:

- senha armazenada com hash forte;
- access token curto;
- refresh token rotativo e armazenado como hash;
- logout revoga refresh token;
- cookies `HttpOnly`, `Secure` em produção e `SameSite` adequado;
- bloqueio/rate limit em login e refresh;
- detecção de refresh token reutilizado;
- endpoint `/me` sem dados sensíveis desnecessários;
- nunca salvar access token em localStorage.

Validações:

```bash
npm test -- auth
npm run test:e2e -- auth
```

## 5. CSRF

A nova API já possui `CsrfService`. Regras:

- toda mutação autenticada deve validar CSRF;
- rotas públicas por token não usam sessão, mas precisam de rate limit;
- o header CSRF deve ser enviado pelo frontend em POST/PATCH/PUT/DELETE;
- falha CSRF deve retornar 403 sem vazar detalhes internos.

Rotas que nunca podem ficar sem CSRF quando autenticadas:

- criação/edição de cliente;
- upload/importação de documentos;
- download/revelação de dados sensíveis quando for mutação de auditoria;
- certificados;
- contratos;
- ZapSign;
- Acessórias sync/backfill/apply;
- RBAC/admin.

## 6. RBAC e escopo por cliente

Regras:

- toda rota de domínio deve usar `@Permissions`;
- permissões devem ser granulares;
- frontend apenas esconde ações; backend decide;
- escopo `mine` deve limitar por responsabilidade/departamento quando aplicável;
- admin/manager precisa estar explicitamente autorizado;
- usuário inativo não deve autenticar nem aparecer como responsável ativo.

Permissões sensíveis sugeridas:

```txt
client_accesses.reveal
certificates.password.reveal
certificates.download
contracts.sign
contracts.zapsign.sync
acessorias.sync
acessorias.backfill
acessorias.apply
settings.secrets.check
audit.read
```

## 7. Uploads e documentos

Controles obrigatórios:

- validação por extensão;
- validação por MIME;
- validação por assinatura/magic bytes;
- limite de tamanho;
- hash SHA-256;
- scanner de malware;
- quarentena em falha quando configurado como `strict`;
- storage privado;
- download apenas por endpoint autenticado ou token público restrito;
- `Content-Disposition` seguro;
- `Cache-Control: no-store` para documentos sensíveis;
- auditoria de upload, download, arquivamento e alteração de status.

O backend deve rejeitar arquivo inválido mesmo que o frontend permita selecionar.

## 8. Certificados digitais

Certificados digitais exigem tratamento especial.

Regras:

- arquivo em storage privado;
- senha criptografada com AES-256-GCM ou KMS;
- IV/nonce único por segredo;
- tag de autenticação persistida;
- chave de criptografia fora do banco;
- revelação de senha apenas com permissão específica;
- auditoria de revelação com IP/user-agent;
- senha nunca deve ser devolvida em listagem;
- frontend mascara senha por padrão;
- rotação planejada da chave de criptografia.

## 9. Contratos e assinatura

Regras:

- contrato enviado para assinatura deve ter snapshot imutável;
- alterações após envio exigem cancelamento ou nova versão;
- assinatura interna deve registrar nome, documento, IP, user-agent e hash;
- ZapSign deve ser provider separado;
- provider ZapSign bloqueia assinatura interna;
- sync/webhook deve confirmar status na ZapSign antes de concluir;
- PDF assinado deve ser tratado como documento confidencial;
- evento de assinatura deve gerar auditoria e notificação.

## 10. Integração Acessórias

Regra de negócio obrigatória solicitada para a nova arquitetura:

- obrigações/listas de entregas devem ser buscadas a partir do **ano atual**;
- colaboradores/responsáveis devem ser buscados a partir do **mês atual**;
- se a API externa não aceitar filtro por data, aplicar filtro pós-consulta antes de persistir;
- não importar colaboradores antigos/desligados como responsáveis ativos;
- não alimentar a operação com obrigações expiradas de anos anteriores;
- backfill histórico só pode existir como tarefa controlada, documentada, manual, com janela explícita e sem contaminar o workspace operacional.

A janela padrão atual do backend deve ser corrigida para não buscar meses do ano anterior no fluxo operacional.

## 11. Secrets e empacotamento

Foram encontrados arquivos `.env` nos pacotes analisados. Não incluir em nenhum pacote final.

Regras:

- `.env` real nunca versionado;
- `.env.example` sem valores reais;
- tokens expostos devem ser rotacionados;
- zip seguro deve excluir `.git`, `.env*`, `node_modules`, dumps, backups, certificados, storage, logs e evidências sensíveis;
- scripts de package devem validar exclusões;
- secret scanning antes de deploy.

Comandos esperados:

```bash
npm run ops:secrets:check
npm run ops:readiness -- --soft --json
```

## 12. Banco de dados

Regras:

- migration revisada antes de produção;
- backup antes de migration;
- restore drill em ambiente isolado;
- índices para campos de busca/status;
- constraints para unicidade crítica;
- soft delete quando houver histórico/auditoria;
- evitar JSON para campos que precisam filtro/índice;
- dados sensíveis criptografados ou separados.

## 13. Logs e auditoria

Logs técnicos não podem conter:

- senha;
- token;
- refresh token;
- JWT completo;
- CPF completo quando não necessário;
- conteúdo de documento;
- PDF/base64;
- chave de certificado.

Auditoria deve conter:

- ator;
- ação;
- entidade;
- antes/depois sanitizado quando aplicável;
- IP/user-agent;
- request id;
- status;
- motivo para ação sensível.

Eventos mínimos:

```txt
auth.login
auth.logout
auth.refresh.reuse_detected
clients.created
clients.updated
documents.uploaded
documents.downloaded
certificates.uploaded
certificates.password_revealed
contracts.sent_signature
contracts.signed
contracts.zapsign.synced
acessorias.sync.started
acessorias.sync.finished
rbac.changed
secrets.rotation.checked
```

## 14. Frontend seguro

Regras:

- não usar localStorage para tokens;
- não renderizar HTML não confiável sem sanitização;
- não fazer log de payload sensível;
- mascarar dados sensíveis;
- tratar 401/403/429 corretamente;
- botões bloqueados por permissão;
- rotas públicas com payload mínimo;
- CSP deve ser planejada para produção;
- links externos de ZapSign devem usar `rel="noreferrer"` quando abrirem nova aba.

## 15. Ambientes

| Ambiente | Regras |
|---|---|
| Local | dados fake, `.env` local fora do Git, sandbox. |
| Homologação | banco próprio, ZapSign sandbox, dados anonimizados quando possível. |
| Produção | secrets reais, HTTPS obrigatório, backups, logs, monitoring. |

Swagger:

- permitido em local/homologação controlada;
- desabilitado ou protegido em produção.

## 16. Backup e restore

Obrigatório antes de:

- migrations;
- backfills;
- importação de legado;
- rotação de criptografia;
- alterações de storage;
- deploy com mudança de schema.

Critério de segurança: backup não testado por restore drill não deve ser considerado backup válido.

## 17. Monitoramento e alertas

Alertas mínimos:

- falha de login elevada;
- erro 5xx elevado;
- upload rejeitado por malware;
- falha no scanner;
- falha ZapSign;
- webhook rejeitado;
- sync Acessórias falhando;
- fila/backfill preso;
- backup falhou;
- disco/storage próximo do limite;
- readiness falhou.

## 18. Política de incidentes

Quando houver suspeita de vazamento:

1. Isolar ambiente afetado.
2. Revogar/rotacionar secrets.
3. Invalidar sessões se necessário.
4. Preservar logs e auditoria.
5. Identificar janela de exposição.
6. Verificar acessos indevidos.
7. Comunicar responsáveis internos.
8. Corrigir causa raiz.
9. Fazer pós-incidente com ações preventivas.

## 19. Checklist de go-live de segurança

- `.env` ausente em pacote final;
- secrets rotacionados;
- migrations testadas;
- backup criado e restore validado;
- RBAC revisado;
- scanner de upload ativo;
- storage privado;
- CSRF ativo;
- CORS restrito;
- rate limit ativo;
- ZapSign em produção controlada;
- Acessórias com janela temporal correta;
- logs sem segredo;
- auditoria habilitada;
- testes automatizados verdes;
- readiness aprovado.
