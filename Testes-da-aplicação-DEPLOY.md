Você é um Arquiteto de Software Sênior, Engenheiro de QA, Especialista em Segurança AppSec, Especialista em Front-end, Back-end, UI/UX e Produto Digital.

Preciso que você analise uma aplicação chamada Portal Sama, composta por documentação, front-end e back-end. A aplicação lida com documentos empresariais, portanto segurança, controle de acesso, integridade dos dados, confiabilidade e usabilidade são pontos críticos.

Arquivos/projetos disponíveis para análise:

- portal-sama-docs
- portal-sama-web
- portal-sama-api

Seu objetivo é realizar uma auditoria completa da aplicação atual para verificar:

1. Se a aplicação é viável tecnicamente.
2. Se está próxima de ficar funcional para usuários reais.
3. O que falta para ela ser considerada pronta para uso em produção.
4. Quais funcionalidades estão completas, incompletas, quebradas ou ausentes.
5. Quais riscos existem em segurança, arquitetura, código, UX, UI, integrações e operação.
6. Quais testes devem ser realizados e quais falhas foram encontradas.
7. Se a aplicação pode ou não ser liberada para usuários começarem a utilizar.

A análise deve ser rigorosa, como se a aplicação estivesse prestes a entrar em produção.

A auditoria não deve ser apenas estática. Como você tem acesso à máquina e pode executar código, utilize Playwright preferencialmente, ou Cypress se for mais adequado ao projeto, para simular um navegador real e homologar a aplicação.

Obrigatório:

- Subir front-end e back-end localmente.
- Rodar build, lint e testes existentes.
- Criar ou executar testes E2E.
- Testar a aplicação como usuário real.
- Testar perfis DEV, Gestor e Colaborador.
- Testar autenticação, autorização, rotas protegidas, navegação, formulários, documentos e integrações.
- Testar casos felizes e casos de falha.
- Capturar screenshots, traces, logs de console, erros de rede e respostas da API.
- Registrar evidências no relatório final.

Use Playwright para:

- abrir a aplicação no navegador;
- simular cliques e preenchimento de campos;
- validar textos e elementos visíveis;
- verificar redirecionamentos;
- testar responsividade;
- capturar erros de console;
- capturar falhas de requests;
- gerar traces e screenshots.

A IA deve tratar cada falha encontrada como um achado de QA, classificando por severidade:

- Bloqueante
- Crítico
- Alto
- Médio
- Baixo

Ao final, informe se a aplicação está:

- pronta para produção;
- pronta apenas para homologação;
- não pronta para usuários reais;
- ou não testável por erro de configuração/execução.

---

# Regras obrigatórias da análise

Não faça suposições sem evidências. Sempre que possível, cite:

- Arquivo analisado.
- Pasta/módulo relacionado.
- Função, componente, endpoint ou tela afetada.
- Linha aproximada ou trecho relevante, caso disponível.
- Evidência do problema encontrado.
- Impacto prático para usuário, negócio ou segurança.

Caso não consiga executar alguma parte da aplicação, informe claramente:

- O que tentou fazer.
- Qual erro ocorreu.
- Qual dependência/configuração faltou.
- O impacto disso na avaliação.
- Como corrigir para permitir testes futuros.

Não apenas diga que algo “parece bom” ou “parece ruim”. Explique tecnicamente o motivo.

Classifique cada achado usando a seguinte escala:

- **Bloqueante**: impede a aplicação de funcionar ou impede uso seguro por usuários reais.
- **Crítico**: risco grave de segurança, perda de dados, vazamento de documentos ou falha grave de regra de negócio.
- **Alto**: problema importante que afeta estabilidade, autenticação, autorização, integridade, fluxo principal ou experiência essencial.
- **Médio**: problema relevante, mas com contorno temporário ou impacto limitado.
- **Baixo**: melhoria, ajuste visual, organização, documentação ou refinamento.

Ao final, dê uma decisão clara:

- **Pronto para produção**
- **Pronto apenas para ambiente de homologação**
- **Não pronto para usuários reais**
- **Não executável / análise limitada**

---

# Etapa 1 — Entendimento da aplicação

Primeiro, analise a documentação disponível.

Verifique:

- Qual é o objetivo do Portal Sama.
- Quem são os usuários esperados.
- Quais são os fluxos principais.
- Quais telas deveriam existir.
- Quais funcionalidades são esperadas.
- Quais regras de negócio aparecem na documentação.
- Quais integrações são mencionadas.
- Quais requisitos de segurança são necessários por lidar com documentos empresariais.

Depois, compare documentação versus código.

Informe:

- O que está documentado e implementado.
- O que está documentado, mas não implementado.
- O que está implementado, mas não documentado.
- O que está ambíguo ou sem regra de negócio clara.
- O que precisa ser melhor especificado antes de finalizar a aplicação.

---

# Etapa 2 — Análise da estrutura do projeto

Analise a arquitetura geral dos projetos:

## Front-end

Verifique:

- Framework utilizado.
- Organização de pastas.
- Componentização.
- Roteamento.
- Gerenciamento de estado.
- Chamadas para API.
- Tratamento de erros.
- Validação de formulários.
- Controle de autenticação no client.
- Proteção de rotas.
- Reutilização de componentes.
- Separação entre layout, regra de negócio e serviços.
- Responsividade.
- Acessibilidade.
- Consistência visual.
- Clareza da navegação.

## Back-end

Verifique:

- Framework utilizado.
- Organização de módulos.
- Estrutura de controllers, services, repositories, models e middlewares.
- Autenticação.
- Autorização.
- Validação de entrada.
- Regras de negócio.
- Tratamento de erros.
- Logs.
- Integração com banco de dados.
- Migrations ou schema.
- Controle de permissões.
- Upload/download/armazenamento de documentos.
- Segurança dos endpoints.
- Padronização de respostas da API.
- Testabilidade.

## Documentação

Verifique:

- Se existe documentação técnica.
- Se existe documentação de instalação.
- Se existe documentação de ambiente.
- Se existe documentação de API.
- Se existe documentação das telas.
- Se existe documentação de regras de negócio.
- Se existe documentação de segurança.
- Se existe documentação para deploy.
- Se existe documentação para usuários finais.
- Se a documentação está coerente com o código atual.

---

# Etapa 3 — Execução e validação técnica

Tente preparar e executar a aplicação localmente, se o ambiente permitir.

Execute ou avalie:

- Instalação de dependências.
- Configuração de variáveis de ambiente.
- Execução do front-end.
- Execução do back-end.
- Build de produção.
- Lint.
- Type-check, se aplicável.
- Testes automatizados existentes.
- Conexão com banco de dados.
- Execução de migrations/seeds.
- Integração front-end com back-end.
- Fluxos principais de usuário.

Informe todos os comandos utilizados e seus resultados.

Caso a aplicação não execute, identifique exatamente o motivo.

---

# Etapa 4 — QA funcional da aplicação

Monte uma matriz de funcionalidades com o seguinte formato:

| Área | Funcionalidade | Status | Evidência | Problema encontrado | Severidade | Recomendação |
|---|---|---|---|---|---|---|

Use os seguintes status:

- **Funcionando**
- **Funcionando parcialmente**
- **Não funcionando**
- **Não implementado**
- **Não testável**
- **Inconsistente com a documentação**

Teste ou avalie os seguintes pontos:

## Autenticação

- Login.
- Logout.
- Persistência de sessão.
- Expiração de sessão.
- Proteção contra acesso não autenticado.
- Redirecionamento correto.
- Mensagens de erro.
- Tratamento de token inválido.
- Recuperação de senha, se existir.

## Autorização e permissões

- Usuário comum.
- Administrador.
- Perfis ou papéis de acesso.
- Acesso indevido a rotas protegidas.
- Acesso indevido a documentos de outros usuários/empresas.
- Validação de permissão no back-end, não apenas no front-end.

## Documentos empresariais

- Upload de documentos.
- Listagem de documentos.
- Download de documentos.
- Visualização de documentos.
- Exclusão de documentos.
- Edição de metadados.
- Controle de proprietário/empresa.
- Permissões por documento.
- Validação de tipo de arquivo.
- Validação de tamanho de arquivo.
- Prevenção contra upload malicioso.
- Armazenamento seguro.
- Proteção contra acesso direto indevido.

## Usuários e empresas

- Cadastro.
- Edição.
- Listagem.
- Associação usuário-empresa.
- Controle de permissões por empresa.
- Isolamento de dados entre empresas.
- Tratamento de usuários inativos ou removidos.

## Dashboard e telas principais

- Dados exibidos corretamente.
- Estados vazios.
- Estados de carregamento.
- Estados de erro.
- Feedback visual para ações.
- Botões e menus funcionando.
- Navegação intuitiva.
- Responsividade.
- Consistência visual.

## Integrações

Para cada integração encontrada, verifique:

- Onde é configurada.
- Se há variáveis de ambiente necessárias.
- Se há tratamento de erro.
- Se há fallback.
- Se dados sensíveis ficam expostos.
- Se a integração é chamada corretamente.
- Se existem logs úteis.
- Se existem timeouts.
- Se existe retry ou política de falha.
- Se a falha da integração quebra a aplicação inteira.

---

# Etapa 5 — Testes de segurança

Faça uma análise forte de segurança, considerando que a aplicação manipula documentos empresariais sensíveis.

Avalie, no mínimo:

## Autenticação e sessão

- Senhas armazenadas com hash seguro.
- Tokens JWT ou sessões configurados corretamente.
- Expiração de tokens.
- Refresh token, se existir.
- Proteção contra token reuse.
- Cookies seguros, se usados.
- Flags `HttpOnly`, `Secure` e `SameSite`, se aplicável.
- Logout realmente invalida a sessão.
- Não há dados sensíveis em localStorage sem necessidade.

## Autorização

- Verificação de permissão no back-end.
- Proteção contra IDOR.
- Usuário não pode acessar documento de outra empresa alterando ID na URL.
- Usuário não pode chamar endpoint administrativo sem permissão.
- Middleware de autorização aplicado corretamente.
- Rotas sensíveis protegidas.

## Validação de entrada

- Sanitização de inputs.
- Validação de payloads.
- Prevenção contra SQL Injection.
- Prevenção contra NoSQL Injection, se aplicável.
- Prevenção contra XSS.
- Prevenção contra path traversal.
- Validação de parâmetros de rota.
- Validação de query params.
- Validação de body requests.

## Upload e armazenamento de arquivos

- Bloqueio de extensões perigosas.
- Validação real de MIME type.
- Limite de tamanho.
- Nomes de arquivos sanitizados.
- Arquivos armazenados fora de pasta pública, se necessário.
- URLs assinadas ou controle de acesso para download.
- Proteção contra execução de arquivos enviados.
- Antivírus ou scanner de malware recomendado.
- Separação de arquivos por tenant/empresa.
- Logs de acesso a documentos.

## Configuração e segredos

- Não há senhas, tokens ou chaves no código.
- `.env` não está versionado.
- Existe `.env.example`.
- Variáveis obrigatórias documentadas.
- Configuração diferente para dev, homologação e produção.
- CORS configurado corretamente.
- Rate limiting.
- Helmet/security headers, se aplicável.
- CSRF protection, se aplicável.
- Logs não vazam dados sensíveis.

## Dependências

- Verificar bibliotecas vulneráveis.
- Verificar versões desatualizadas.
- Verificar pacotes sem manutenção.
- Verificar uso de dependências desnecessárias.
- Recomendar substituições quando necessário.

## OWASP

Avalie riscos relacionados ao OWASP Top 10:

- Broken Access Control.
- Cryptographic Failures.
- Injection.
- Insecure Design.
- Security Misconfiguration.
- Vulnerable and Outdated Components.
- Identification and Authentication Failures.
- Software and Data Integrity Failures.
- Security Logging and Monitoring Failures.
- Server-Side Request Forgery, se aplicável.

Ao final da seção de segurança, gere uma tabela:

| Risco | Severidade | Evidência | Impacto | Como explorar | Correção recomendada |
|---|---|---|---|---|---|

Não forneça instruções ofensivas detalhadas de exploração. Foque em análise defensiva, evidência e correção.

---

# Etapa 6 — Análise de UI, UX e usabilidade

Avalie a aplicação do ponto de vista de um usuário real.

Verifique:

- O usuário entende o objetivo de cada tela?
- A navegação é clara?
- Os menus são intuitivos?
- Os textos explicam bem as ações?
- Os botões têm rótulos claros?
- Os formulários indicam campos obrigatórios?
- Os erros são compreensíveis?
- O usuário recebe feedback após salvar, enviar, excluir ou falhar?
- Existe confirmação para ações destrutivas?
- Existem estados de loading?
- Existem estados vazios bem explicados?
- A aplicação é responsiva?
- A hierarquia visual ajuda o usuário?
- A interface parece consistente?
- Há excesso de informação?
- Há falta de informação?
- O usuário consegue completar os fluxos principais sem orientação externa?

Gere uma tabela:

| Tela/Fluxo | Problema de UX/UI | Impacto no usuário | Severidade | Recomendação |
|---|---|---|---|---|

Também informe quais melhorias são necessárias para deixar a aplicação mais profissional e fácil de usar.

---

# Etapa 7 — Testes de API

Analise todos os endpoints encontrados.

Para cada grupo de endpoints, avalie:

- Método HTTP correto.
- Nome da rota.
- Autenticação exigida.
- Autorização exigida.
- Validação de entrada.
- Resposta de sucesso.
- Resposta de erro.
- Status codes corretos.
- Tratamento de exceções.
- Paginação, se necessário.
- Filtros, se necessário.
- Ordenação, se necessário.
- Proteção contra acesso indevido.
- Consistência do contrato da API.

Gere uma tabela:

| Endpoint | Método | Protegido? | Validação | Status | Problemas | Severidade | Recomendação |
|---|---|---|---|---|---|---|---|

---

# Etapa 8 — Testes de front-end

Analise as telas e componentes.

Verifique:

- Rotas quebradas.
- Links quebrados.
- Componentes sem tratamento de erro.
- Componentes duplicados.
- Código morto.
- Chamadas de API hardcoded.
- Falta de loading.
- Falta de feedback.
- Falta de validação.
- Falta de proteção de rota.
- Estados inconsistentes.
- Problemas de responsividade.
- Problemas de acessibilidade.
- Problemas de performance.
- Exposição de dados sensíveis no client.

Gere uma tabela:

| Tela/Componente | Problema | Evidência | Impacto | Severidade | Correção |
|---|---|---|---|---|---|

---

# Etapa 9 — Testes de back-end

Analise serviços, controllers, models, middlewares e integrações.

Verifique:

- Regras de negócio incompletas.
- Falta de validação.
- Falta de autorização.
- Queries inseguras.
- Falta de transações.
- Falta de tratamento de erro.
- Falta de logs.
- Respostas inconsistentes.
- Código duplicado.
- Acoplamento excessivo.
- Falta de testes.
- Falta de documentação de endpoints.
- Problemas de escalabilidade.
- Problemas no acesso a documentos.
- Problemas de isolamento entre empresas/tenants.

Gere uma tabela:

| Módulo | Problema | Evidência | Impacto | Severidade | Correção |
|---|---|---|---|---|---|

---

# Etapa 10 — Testes de qualidade, build e produção

Avalie se a aplicação está pronta para rodar em ambiente real.

Verifique:

- Build de produção.
- Variáveis de ambiente.
- Logs.
- Observabilidade.
- Tratamento de erros.
- Health check.
- Docker, se existir.
- Deploy, se documentado.
- Banco de dados.
- Migrações.
- Backup.
- Restauração.
- Rate limit.
- CORS.
- Segurança de headers.
- HTTPS.
- Política de arquivos.
- Configuração por ambiente.
- Scripts de inicialização.
- Scripts de teste.
- Seeds.
- CI/CD, se existir.
- Testes automatizados.
- Monitoramento.
- Alertas.

Classifique a prontidão operacional:

| Item | Status | Observação | Prioridade |
|---|---|---|---|

---

# Etapa 11 — Plano de testes recomendado

Além dos testes realizados, crie um plano de QA recomendado para validar a aplicação antes da entrega.

Inclua:

## Testes manuais

- Fluxos felizes.
- Fluxos de erro.
- Permissões.
- Upload/download de documentos.
- Tentativas de acesso indevido.
- Responsividade.
- Usabilidade.
- Navegação.

## Testes automatizados

- Unitários.
- Integração.
- End-to-end.
- API.
- Segurança básica.
- Contrato da API.
- Regressão.

## Testes não funcionais

- Performance.
- Carga.
- Segurança.
- Acessibilidade.
- Compatibilidade de navegadores.
- Recuperação de falhas.

Gere casos de teste no formato:

| ID | Caso de teste | Pré-condição | Passos | Resultado esperado | Prioridade |
|---|---|---|---|---|---|

---

# Etapa 12 — Relatório final obrigatório

Ao final, gere um relatório em Markdown com as seguintes seções:

## 1. Resumo executivo

Explique em linguagem clara:

- Estado geral da aplicação.
- Principais riscos.
- Principais problemas.
- Se usuários reais já poderiam usar.
- O que impede a entrada em produção.
- Estimativa qualitativa de maturidade: baixa, média ou alta.

## 2. Decisão de prontidão

Escolha uma das opções:

- Pronto para produção.
- Pronto para homologação.
- Não pronto para usuários reais.
- Não foi possível validar por falhas de execução/configuração.

Justifique a decisão.

## 3. Funcionalidades avaliadas

Tabela com todas as funcionalidades encontradas e seus status.

## 4. Problemas bloqueantes

Liste apenas problemas que impedem o uso real ou seguro da aplicação.

Para cada item:

- Descrição.
- Evidência.
- Impacto.
- Correção recomendada.
- Arquivos afetados.
- Prioridade.

## 5. Problemas críticos de segurança

Liste os riscos mais graves de segurança.

Priorize especialmente:

- Vazamento de documentos.
- Acesso indevido.
- Falha de autorização.
- Upload inseguro.
- Segredos expostos.
- Falhas de autenticação.
- Falhas de isolamento entre empresas.

## 6. Bugs funcionais

Liste bugs de front-end, back-end, API e integração.

## 7. Problemas de UI/UX

Liste problemas que dificultam o entendimento ou uso da aplicação.

## 8. Problemas de documentação

Liste lacunas, inconsistências ou documentos ausentes.

## 9. Problemas de arquitetura e código

Liste problemas estruturais, técnicos e de manutenção.

## 10. Integrações

Informe:

- Quais integrações existem.
- Quais funcionam.
- Quais não funcionam.
- Quais não puderam ser testadas.
- Quais precisam de configuração.
- Quais têm risco de segurança.

## 11. Testes executados

Liste:

- Comandos executados.
- Testes realizados.
- Resultados.
- Falhas encontradas.
- Partes não testadas.

## 12. Plano de correção

Monte um plano por prioridade:

### Fase 1 — Correções bloqueantes

Itens indispensáveis antes de qualquer usuário real acessar.

### Fase 2 — Segurança e estabilidade

Itens necessários antes de homologação séria ou produção.

### Fase 3 — UX, documentação e refinamento

Itens para melhorar a experiência e maturidade.

### Fase 4 — Evolução técnica

Melhorias de arquitetura, frameworks, testes, CI/CD e escalabilidade.

## 13. Checklist para produção

Crie um checklist final com status:

| Item | Status | Observação |
|---|---|---|

Use os status:

- OK
- Pendente
- Crítico
- Não avaliado

## 14. Conclusão final

Responda claramente:

- A aplicação está pronta?
- O que falta para fechar?
- Quais são os riscos de liberar agora?
- Qual seria o próximo passo recomendado?
- Qual é a ordem ideal de correção?

---

# Formato de resposta

Entregue a resposta em Markdown.

Se possível, divida o resultado em arquivos lógicos:

- `AUDITORIA_GERAL.md`
- `QA_FUNCIONAL.md`
- `SEGURANCA.md`
- `UI_UX.md`
- `FRONTEND.md`
- `BACKEND.md`
- `INTEGRACOES.md`
- `PLANO_DE_CORRECAO.md`
- `CHECKLIST_PRODUCAO.md`

Caso não consiga criar arquivos, entregue tudo em uma única resposta bem organizada em Markdown.

A análise precisa ser objetiva, técnica e acionável. O foco principal é descobrir se a aplicação está realmente pronta para usuários reais e o que precisa ser corrigido antes disso.

#####

Importante: não faça apenas uma revisão visual ou teórica. Tente executar a aplicação, instalar dependências, rodar build, rodar testes, validar endpoints, revisar permissões, simular fluxos de usuário e procurar falhas reais. 

Quando algo não funcionar, registre como achado. Quando algo não puder ser testado por falta de configuração, registre como limitação da auditoria e explique o que precisa ser fornecido para testar corretamente.

O resultado esperado não é uma opinião geral, mas sim um diagnóstico técnico de pré-produção com evidências, severidade, impacto e plano de correção.

#####

Dê peso máximo para segurança de documentos empresariais. Qualquer possibilidade de um usuário acessar, baixar, visualizar, alterar ou excluir documentos de outra empresa/usuário deve ser tratada como problema crítico ou bloqueante.

Verifique especialmente:

- IDOR.
- Falta de autorização no back-end.
- Proteção apenas no front-end.
- Links públicos para documentos.
- Falhas no isolamento entre empresas.
- Upload de arquivos perigosos.
- Ausência de logs de acesso a documentos.
- Ausência de trilha de auditoria.
- Falta de criptografia ou controle seguro de armazenamento.
- Falta de expiração em links de download.
- Falta de política de retenção/exclusão.