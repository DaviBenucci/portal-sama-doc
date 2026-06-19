# Runbook Acessórias

Objetivo: validar e operar a integração Acessórias como fonte externa sincronizada para dados locais do Portal Sama.

## Estado técnico

- Arquitetura local persistida implementada.
- Catálogo, entregas, sync runs, locks, rate limiter, retries e reconciliação existem no código.
- Homologação com payload real ainda pendente.

## Checklist de configuração

- [ ] `ACESSORIAS_BASE_URL` configurada no ambiente alvo.
- [ ] `ACESSORIAS_TOKEN` rotacionado e válido.
- [ ] Timeout configurado.
- [ ] Rate limit configurado conforme contrato externo.
- [ ] Scheduler configurado para produção ou manual-only.
- [ ] Push técnico DEV revisado para não expor dados sensíveis.

## Testes com API real

- [ ] `401/403` com token inválido ou sem autorização.
- [ ] `429` e comportamento de retry/backoff.
- [ ] Paginação real.
- [ ] Payload com campos nulos.
- [ ] Payload com campos ausentes.
- [ ] Volume alto.
- [ ] Incremental por `DtLastDH`.
- [ ] Backfill por empresa.
- [ ] Falha parcial.
- [ ] Reprocessamento.
- [ ] Divergência de responsável.

## Operação

1. Rodar sincronização controlada em horário de baixo risco.
2. Verificar `AcessoriasSyncRun`.
3. Verificar contadores de criados/atualizados/ignorados/falhos.
4. Revisar aliases pendentes e ambíguos.
5. Confirmar Central de Vencimentos com dados locais.
6. Registrar evidência de payload e contadores, sem dados sensíveis.

## Critérios de aceite

- [ ] Tela não depende da API externa em cada acesso.
- [ ] Dados locais ficam coerentes com amostra validada da Acessórias.
- [ ] Falhas externas não quebram a Home nem a Central de Vencimentos.
- [ ] Usuário vê stale warning quando dados ficam desatualizados.
- [ ] Reprocessamento não duplica entregas.
