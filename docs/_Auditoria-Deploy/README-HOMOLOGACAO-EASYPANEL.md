# README - Pacote de homologacao EasyPanel / Acessorias

Data: 11/06/2026  
Escopo: continuidade operacional apos aplicacao das documentacoes `AUDITORIA-DEPLOY-00` a `AUDITORIA-DEPLOY-16` e planos `00` a `07`.

## Ordem recomendada de leitura pelo Codex

1. `AUDITORIA-DEPLOY-17-HOMOLOGACAO-EASYPANEL-ACESSORIAS-POS-INTEGRACAO.md`
2. `AUDITORIA-DEPLOY-18-PLANO-CORRECAO-HOMOLOGACAO-EASYPANEL.md`
3. `PROMPT-CODEX-HOMOLOGACAO-EASYPANEL-ACESSORIAS.md`

## Evidencias de entrada

As evidencias recebidas nesta rodada foram copiadas para:

```txt
evidencias/entrada/Homolacao-portal-sama-easypanel.txt
evidencias/screenshots/01-operacao-dev-acessorias-botoes.png
evidencias/screenshots/02-home-dev-diagnostico-acessorias.png
evidencias/screenshots/03-solicitacao-acesso-calendario-nativo.png
evidencias/screenshots/04-solicitacao-acesso-formulario-gestor.png
```

## Regra de uso

Este pacote nao deve ser tratado como uma implementacao cega. Ele e um roteiro de auditoria, correcao e homologacao. O Codex deve:

- ler os documentos ativos do `portal-sama-docs` antes de alterar codigo;
- confirmar o comportamento atual no codigo;
- reproduzir os bugs com testes ou logs;
- corrigir por fases;
- executar testes focados a cada fase;
- registrar evidencias sanitizadas;
- nao gravar tokens, senhas, CNPJs reais completos, payloads brutos do Acessorias ou documentos empresariais em evidencias.
