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

## Regra temporal obrigatória da nova arquitetura

A sincronização operacional do Acessórias não deve trazer histórico antigo para o workspace atual.

Regras obrigatórias:

- obrigações/listas de entregas devem ser buscadas a partir de `01/01` do ano atual;
- colaboradores/responsáveis devem ser buscados a partir do primeiro dia do mês atual;
- se a API externa não aceitar filtro por data, o backend deve aplicar filtro pós-consulta antes de persistir;
- colaboradores antigos, desligados ou sem vínculo atual não podem ser importados como responsáveis ativos;
- obrigações vencidas de anos anteriores não podem alimentar Home, Central de Vencimentos ou painel operacional;
- backfill histórico deve ser tarefa manual, com janela explícita, auditoria e sem afetar o workspace operacional sem revisão.

Validações obrigatórias:

- teste unitário garantindo `DtInitial` no ano atual para entregas;
- teste unitário ou integração garantindo filtro do mês atual para colaboradores/responsáveis;
- teste com payload antigo confirmando que dados obsoletos são ignorados;
- registro dos parâmetros de janela em `AcessoriasSyncRun.metadata`.

## Implementação local - Fase 6

- O sync operacional de entregas em `deliveries/ListAll` envia `DtInitial` como `01/01` do ano atual, `DtFinal` como `31/12` do ano atual e `DtLastDH` limitado ao mesmo ano operacional.
- Entregas recebidas fora da janela operacional são descartadas antes de `upsert`, usando vencimento, entrega, última alteração externa ou competência como evidência.
- Backfill histórico de entregas exige `dt_initial` e `dt_final` explícitos; sem datas, o backend retorna `ACESSORIAS_BACKFILL_DATE_WINDOW_REQUIRED`.
- O sync direto de colaboradores envia `DtInitial`/`DtFinal` do mês atual, mesmo em caminho customizado sem paginação.
- Colaboradores inativos/desligados antigos são filtrados antes da criação/atualização local. Colaboradores ativos com admissão antiga não são descartados apenas por data de admissão.
- Responsáveis extraídos de departamentos carregam datas de alteração/desligamento quando o payload da empresa/departamento fornece essa evidência.
- `AcessoriasSyncRun.metadata` e auditoria registram política de janela, datas, contadores e parâmetros operacionais.
- Locks de sync pesado foram cobertos por teste unitário para concorrência, expiração por TTL e release.
