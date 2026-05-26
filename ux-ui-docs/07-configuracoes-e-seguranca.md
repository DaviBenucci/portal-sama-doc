# 07 - Configuracoes e Seguranca

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
