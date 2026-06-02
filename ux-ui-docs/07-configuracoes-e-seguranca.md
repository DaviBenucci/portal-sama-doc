# [PARCIAL] 07 - Configuracoes e Seguranca

## Status local - 2026-06-02

- `/configuracoes` foi criada no React.
- Abas locais implementadas: Minha conta, Seguranca, Notificacoes, Preferencias e Administracao condicional.
- Minha conta exibe nome, login, email, departamento, status e perfis em modo leitura.
- Foto de perfil possui preview local e validacoes client-side: PNG/JPG/WebP, limite de 2 MB e bloqueio de SVG.
- Seguranca exibe expiracao da sessao, politica de senha, MFA futuro e formulario de troca de senha.
- Troca de senha foi reforcada no backend: `UsersService.update()` rejeita alteracao de senha quando o ator nao possui papel `MASTER`; quando permitido, a auditoria existente registra `passwordChanged` sem segredo bruto.
- Preferencias de notificacoes/visual sao locais ate existir endpoint dedicado.
- Pendencias: persistir avatar em backend/storage seguro, validar MIME real no servidor, remover metadados, gerar versao otimizada, listar sessoes/dispositivos reais e homologar com usuarios reais no EasyPanel.

## Rota
Criar base para `/configuracoes`.

## Abas
- Minha conta;
- Seguranca;
- Notificacoes;
- Preferencias;
- Administracao.

## Minha conta

### Deve conter
- nome;
- foto de perfil;
- usuario/login;
- email;
- departamento;
- perfis em modo leitura;
- preferencias visuais.

## Foto de perfil

O usuario pode alterar a foto de perfil.

### Validacoes obrigatorias
- aceitar PNG, JPG ou WebP;
- bloquear SVG;
- limitar tamanho;
- validar MIME type real no backend;
- remover metadados;
- gerar versao otimizada;
- armazenar em local controlado.

## Seguranca da conta

### Deve conter
- sessoes ativas;
- ultimos acessos;
- dispositivos recentes;
- expiracao de sessao;
- MFA futuro;
- politica de senha em leitura.

### Sessao expirada e auto-refresh

Implementado em 2026-06-01 no Web:

- renovar access token automaticamente antes de chamadas protegidas quando o token nao estiver fresco;
- repetir uma unica vez a requisicao original apos `401`, se o refresh token ainda estiver valido;
- impedir refresh paralelo durante rotacao do cookie HTTP-only;
- manter CSRF e `withCredentials`;
- nao aumentar a duracao do access token como solucao principal.

## Troca de senha

Regra definida:
- somente usuarios MASTER podem realizar troca de senha;
- usuarios comuns podem solicitar redefinicao, se existir fluxo;
- troca por MASTER deve gerar auditoria;
- troca por MASTER deve exigir confirmacao.

## Notificacoes

### Preferencias
- portal;
- web push;
- modulo;
- frequencia;
- criticidade minima;
- resumo diario futuro.

## UX de seguranca

### Exibir
- selo de ambiente seguro;
- badge de confidencialidade;
- expiracao de links;
- confirmacao para acoes sensiveis;
- trilha de auditoria.

### Acoes que exigem confirmacao
- excluir/arquivar cliente;
- baixar documento sensivel;
- gerar link publico;
- alterar permissao;
- transferir carteira;
- concluir vencimento critico;
- alterar certificado;
- trocar senha por MASTER.

## Dados sensiveis

Nunca salvar no localStorage:
- tokens;
- refresh tokens;
- documentos;
- dados de clientes;
- certificados;
- segredos;
- credenciais.


## Governanca de departamentos, roles e permissoes

### Departamento

Departamento e dado de seguranca operacional. Nao deve ser texto livre.

Regras:
- selecionar departamento por lista controlada;
- bloquear duplicidade por chave normalizada;
- auditar criacao, edicao e inativacao;
- nao excluir departamento com usuarios, clientes ou historico vinculado;
- restringir criacao de departamento a MASTER/Admin autorizado.

### Roles

Roles devem ser apresentadas com nome amigavel, descricao, tipo e risco.

A interface deve mostrar:
- quantidade de usuarios vinculados;
- quantidade de permissoes;
- quantidade de permissoes criticas;
- ultima atualizacao;
- historico de alteracoes.

### Permissoes

Permissoes devem ter titulo descritivo, modulo, acao, descricao e risco.

A chave tecnica deve continuar existindo, mas como informacao secundaria.

Exemplo:

```text
Titulo: Visualizar clientes
Descricao: Permite consultar clientes dentro do escopo autorizado.
Chave tecnica: clients.read
Risco: Baixo
```

### Acoes sensiveis adicionais

Tambem exigem confirmacao e auditoria:
- criar departamento;
- inativar departamento;
- alterar departamento de usuario;
- atribuir role critica;
- criar permissao;
- editar permissao;
- alterar risco de permissao;
- executar solicitacao de acesso aprovada.
