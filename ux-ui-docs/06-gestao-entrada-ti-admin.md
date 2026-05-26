# 06 - Gestao, Entrada de Cliente, T.I e Admin

## Painel do Gestor

### Objetivo
Acompanhamento da operacao.

### Deve conter
- carteira;
- vencimentos criticos;
- atrasos;
- riscos;
- aprovacoes;
- indicadores por colaborador.

## Carteira de Colaboradores

### Objetivo
Distribuicao de clientes por colaborador.

### Deve conter
- lista de colaboradores;
- clientes do colaborador selecionado;
- filtros por departamento/status/risco;
- indicadores de carga.

## Transferencias

### Objetivo
Transferir clientes entre colaboradores.

### Deve conter
- origem;
- destino;
- clientes selecionados;
- motivo;
- confirmacao;
- auditoria.

## Historico

### Objetivo
Timeline de eventos operacionais.

### Deve conter
- data;
- usuario;
- evento;
- cliente;
- modulo;
- detalhes.

## Entrada de Cliente

### Objetivo
Concentrar o onboarding completo.

### Fluxo
```text
Novo processo -> Dados iniciais -> Proposta vinculada -> Contrato vinculado -> Documentos -> Conferencia -> Ativacao -> Concluido
```

### Regras
- proposta aceita pode ser selecionada;
- contrato assinado pode ser selecionado;
- checklist documental;
- auditoria por etapa;
- link publico seguro.

## Solicitacoes de Acesso

### Objetivo
Solicitar acessos ou ajustes.

### Deve conter
- usuario;
- sistema/modulo;
- justificativa;
- periodo;
- aprovador;
- status.

## Acessos de TI

### Objetivo
Controlar acessos operacionais.

### Deve conter
- usuario;
- sistema;
- permissao;
- expiracao;
- status;
- acao.

## Auditoria

### Objetivo
Rastrear eventos sensiveis.

### Filtros
- usuario;
- acao;
- cliente;
- modulo;
- IP;
- periodo.

## Usuarios e Permissoes

### Objetivo
Administrar acessos.

### Abas
- Usuarios;
- Perfis;
- Permissoes;
- Auditoria.

## Colaboradores

### Objetivo
Gerenciar colaboradores internos.

### Deve conter
- ativos;
- restritos;
- arquivados;
- departamento;
- perfil;
- status.

## DEV

### Regra
Aparece apenas para DEV/MASTER ou ambiente autorizado.

### Deve conter
- diagnostico;
- logs;
- integracoes;
- filas;
- parametros;
- auditoria tecnica.
