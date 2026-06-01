# [PARCIAL] 05 - Modulos de Operacao

## Modelo Dpto
Tela operacional padrao dos departamentos.

Status parcial 2026-06-01 11:30: a planilha Fiscal em React ja exibe baixas vindas do Acessorias com status visual `Acessorias`, mostra divergencias abertas e possui acao protegida para aplicar entregas sincronizadas.

Status parcial 2026-06-01 complementar: a tela passou a alternar Fiscal, Contabil, Pessoal, Financeiro e Legalizacao, com colunas proprias por departamento e aplicacao do Acessorias no departamento selecionado.

### Deve conter
- resumo do departamento;
- clientes;
- pendencias;
- obrigacoes;
- responsaveis;
- filtros.

### Segurança
O usuario visualiza apenas escopo permitido.

## Meus Clientes
Mostra clientes vinculados ao usuario logado.

### Deve conter
- busca por nome, CNPJ e nome fantasia;
- filtros;
- cards de pendencias;
- acesso ao painel do cliente.

## Todos os Clientes
Visao ampla da empresa.

### Deve conter
- total de clientes;
- ativos;
- arquivados;
- em risco;
- filtros avancados.

### Segurança
Somente usuarios com permissao ampla.

## Painel do Cliente
Ponto central de operacao do cliente.

### Abas
- Visao geral;
- Documentos;
- Certificados;
- Vencimentos;
- Historico.

## Documentos do Cliente
Documentos devem ficar dentro do painel do cliente.

### Funcionalidades
- upload;
- preview seguro;
- status;
- filtros;
- aprovacao;
- rejeicao;
- auditoria;
- links publicos com expiracao.

### Segurança
- validar MIME type no backend;
- limitar tamanho;
- bloquear arquivos perigosos;
- auditar download e upload;
- confirmar download sensivel.

## Certificados Digitais
Controle de certificados dos clientes.

### Deve conter
- vencidos;
- vencendo em 7 dias;
- vencendo em 30 dias;
- validos;
- responsavel;
- risco.

## Integra-AI
Somente departamento Contabil.

### Deve conter
- jobs ativos;
- processados;
- falhas;
- pendentes;
- busca por empresa/CNPJ/arquivo.

## Vencimentos
Central operacional de obrigacoes.

### Deve conter
- hoje;
- 7 dias;
- atrasados;
- concluidos;
- calendario;
- lista por cliente.

### Status
- previsto;
- confirmado;
- em andamento;
- concluido;
- atrasado.

## Propostas
Somente Financeiro e Comercial.

### Fluxo
```text
Criada -> Enviada -> Aceita/Recusada -> Concluida
```

## Contratos
Legalizacao, Financeiro e Comercial.

### Fluxo
```text
Criado -> Enviado para assinatura -> Assinado -> Concluido
```

## Legalizacao
Somente Legalizacao.

### Deve conter
- processos abertos;
- em analise;
- pendentes;
- concluidos;
- prazo;
- responsavel;
- historico.
