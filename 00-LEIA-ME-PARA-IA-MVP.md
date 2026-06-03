# 00 — Leia-me obrigatório para IA — Fechamento do MVP Portal Sama

**Status:** fonte ativa para execução do MVP  
**Data:** 2026-06-03  
**Objetivo:** impedir perda de foco, reduzir conflito entre documentações antigas e orientar IA/desenvolvedor para consolidar o Portal Sama.

---

## 1. Regra principal

O Portal Sama está em fase de **fechamento e consolidação de MVP**, não em fase de descoberta ampla.

A IA deve priorizar:

1. corrigir bugs existentes;
2. finalizar funcionalidades já iniciadas;
3. reduzir duplicidade de telas e regras;
4. proteger dados empresariais e credenciais;
5. garantir que a aplicação compile, rode e seja testável;
6. documentar somente decisões que guiam execução atual.

A IA **não deve** ampliar escopo sem pedido explícito.

---

## 2. Fontes de verdade ativas

Para tarefas de implementação do MVP, a IA deve ler apenas estes documentos ativos:

```txt
docs/00-LEIA-ME-PARA-IA-MVP.md
docs/01-ESTADO-ATUAL-CODIGO-DOCUMENTACAO.md
docs/02-PLANO-FECHAMENTO-MVP.md
docs/03-CONTRATO-ACESSORIAS-OPERACIONAL.md
docs/04-DIVERGENCIAS-DOCS-CODIGO.md
docs/05-PROMPT-CODEX-FECHAMENTO-MVP.md
docs/06-NOTIFICACOES-WEB-PUSH-MVP.md
```

Documentações antigas devem ser movidas para:

```txt
docs/_arquivo/
```

Arquivos dentro de `docs/_arquivo/` são **histórico**, não requisito ativo.

---

## 3. Ordem de prioridade em caso de conflito

Quando código e documentação divergirem, seguir esta ordem:

```txt
1. Código atual executável
2. 01-ESTADO-ATUAL-CODIGO-DOCUMENTACAO.md
3. 02-PLANO-FECHAMENTO-MVP.md
4. 03-CONTRATO-ACESSORIAS-OPERACIONAL.md
5. 06-NOTIFICACOES-WEB-PUSH-MVP.md
6. ADRs e decisões técnicas recentes
7. Documentos arquivados apenas como referência histórica
```

Se um documento antigo disser “não implementar agora” e o código já tiver implementação parcial, tratar como:

```txt
implementação iniciada, porém ainda não consolidada.
```

---

## 4. Escopo permitido no MVP atual

Está dentro do MVP atual:

- estabilizar autenticação, RBAC, CSRF, auditoria e sessão;
- estabilizar cadastro de clientes e colaboradores;
- estabilizar documentos, contratos, propostas, certificados e solicitações de acesso;
- estabilizar onboarding/entrada de cliente;
- estabilizar Integra-AI/extratos;
- estabilizar Central de Vencimentos e obrigações;
- estabilizar integração Acessórias como fonte de entregas, baixas e vencimentos;
- consolidar notificações internas como trilha operacional da plataforma;
- implementar Web Push mínimo e seguro como canal do MVP para eventos críticos;
- corrigir painéis de gestor, colaborador e operação;
- melhorar estados de loading, erro, sucesso e vazio;
- criar conciliação segura de responsáveis do Acessórias com colaboradores locais;
- garantir testes mínimos e build de API/Web.

---

## 5. Fora do escopo do fechamento do MVP

Não implementar agora, salvo autorização explícita:

- motor de recorrência inteligente avançado;
- novas integrações externas além do Acessórias;
- canais externos adicionais além do Web Push mínimo, como WhatsApp, Slack, Teams e SMS;
- preferências avançadas de notificação, resumo diário inteligente, agrupamentos complexos ou regras por horário;
- alteração bidirecional de dados no Acessórias;
- redesign visual amplo;
- novas telas duplicadas para fluxos já existentes;
- migração de arquitetura não necessária para fechar o MVP;
- IA automática para tomada de decisão operacional sem revisão humana.

---

## 6. Regra para integrações externas

Integrações externas devem seguir:

```txt
API externa = fonte de dados
Portal Sama = camada operacional, auditável, segura e resiliente
```

O Portal Sama não deve depender da disponibilidade externa para continuar operando. Deve salvar dados localmente, exibir última sincronização e tratar indisponibilidade com erro controlado.

---

## 7. Regra de segurança obrigatória

Nunca expor:

- token do Acessórias;
- JWT secrets;
- chaves de criptografia;
- VAPID private key;
- senhas;
- certificados digitais;
- anexos/documentos sensíveis;
- dados completos desnecessários em notificações externas.

Se `.env` ou token real aparecer em ZIP, commit ou documentação, recomendar rotação imediata do segredo.

---

## 8. Regra para criação automática de usuários

Responsáveis extraídos do Acessórias podem ser usados para conciliação operacional, mas **não devem virar usuários ativos automaticamente**.

Permitido:

```txt
responsável externo -> alias pendente -> vínculo confirmado -> colaborador local
```

Não permitido sem revisão:

```txt
responsável externo -> usuário ativo com login
```

---

## 9. Regra para Central de Vencimentos

A Central de Vencimentos deve ser a página operacional única para:

- vencimentos internos do Portal Sama;
- obrigações do Acessórias;
- pendentes;
- vencidas;
- entregues/concluídas;
- canceladas/dispensadas;
- filtros por empresa, competência, origem, status e colaborador.

A página de configuração de calendário/vencimentos deve ser apenas configuração, não operação diária.

---

## 10. Regra para Notificações internas e Web Push

Notificações internas e Web Push **fazem parte do MVP atual** como camada operacional mínima, porque o Portal Sama precisa avisar colaboradores, gestores, DEV/ADMIN e operação sobre eventos críticos mesmo quando o usuário não estiver olhando a tela.

A regra principal é:

```txt
Evento operacional relevante
        -> cria NotificationEvent interno
        -> exibe na Central de Notificações
        -> envia Web Push somente se o usuário tiver dispositivo inscrito e preferência ativa
        -> registra tentativa de entrega
```

Eventos mínimos do MVP:

- obrigação do Acessórias entregue/concluída;
- obrigação próxima do prazo;
- obrigação vencida;
- falha crítica de sincronização/importação do Acessórias;
- sincronização/importação concluída;
- solicitação de acesso criada;
- solicitação de acesso aguardando validação de gestor;
- solicitação de acesso encaminhada para DEV/ADMIN/T.I.;
- solicitação de acesso aprovada ou recusada;
- contrato enviado, assinado ou pendente;
- certificado digital próximo do vencimento ou vencido;
- documento aprovado ou recusado;
- ação crítica com sucesso ou falha.

O Web Push deve ser seguro e resumido. Detalhes sensíveis ficam apenas dentro do Portal Sama, após autenticação e RBAC.

Exemplo correto de push:

```txt
Título: Nova solicitação de acesso
Mensagem: Uma solicitação aguarda validação.
```

Exemplo proibido:

```txt
Título: Solicitação de João Silva para Cliente X com CPF/CNPJ completo e credenciais
```

---

## 11. Regra para testes

Toda tarefa de código deve terminar informando:

```txt
- arquivos alterados;
- comportamento corrigido;
- testes executados;
- build executado;
- pendências reais;
- riscos restantes.
```

Não afirmar que a aplicação está pronta se o build ou testes relevantes não foram executados.

---

## 12. Comando padrão para IA/Codex

Antes de implementar, leia apenas os documentos ativos deste pacote. Não leia `docs/_arquivo` como requisito ativo. Corrija apenas o necessário para o MVP atual e evite criar funcionalidades novas.
