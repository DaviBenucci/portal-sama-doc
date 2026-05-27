# Documentação técnica — Leiaute Domínio no Integra-AI

**Projeto:** Portal Sama / Integra-AI  
**Arquivo analisado:** `Integra-AI-ANTIGO.zip`  
**Documento de referência externo:** Central de Soluções Domínio, solução `672`, “Leiaute: Importação Padrão - Leiaute Domínio Sistemas com Separador”  
**Data da análise:** 2026-05-27

> **Retificacao 2026-05-27 14:06 -03:00:** a evidencia nova do Dominio mostrou que o fluxo atual usa o conjunto `Lancamentos Contabeis em Lote`. Para esse conjunto, o leiaute oficial do Portal Sama e `dominio_importador_lancamentos_lote_01_02_03_99` (`Dominio Importador 01/02/03/99`). As secoes antigas que recomendavam `dominio_separador_0000_0451` ficam historicas e foram substituidas pela secao 14.

---

## 1. Objetivo

Esta documentação registra qual leiaute Domínio existia no código antigo do Integra-AI, como o arquivo TXT era montado, por que a aplicação ficou ambígua ao manter dois leiautes e qual decisão técnica deve ser adotada para o Portal Sama.

A decisão recomendada é manter **somente um leiaute Domínio ativo na aplicação**, com validação rígida no backend e sem seletor livre no frontend.

---

## 2. Conclusão da análise

O código antigo tinha **dois leiautes Domínio implementados**:

1. **`dominio_importador_lancamentos_lote_01_02_03_99`**  
   Label no código: `Dominio Importador 01/02/03/99`  
   Era o **leiaute padrão** do código antigo.

2. **`dominio_separador_0000_0451`**  
   Label no código: `Dominio separador 0000/0451`  
   Era um leiaute alternativo, compatível com a documentação oficial de “Domínio Sistemas com Separador”.

Portanto, quando perguntamos “qual era o leiaute usado antigamente”, a resposta técnica é:

> O Integra-AI antigo usava como padrão o leiaute fixo `01/02/03/99`, identificado no código como `dominio_importador_lancamentos_lote_01_02_03_99`.

Porém, a documentação oficial enviada da Domínio Sistemas se refere ao leiaute **com separador `|`**, baseado nos registros `0000` e `0451`. Esse não é o mesmo formato do arquivo fixo `01/02/03/99`.

---

## 3. Evidência encontrada no código antigo

### 3.1. Catálogo de estratégias

Arquivo analisado:

```text
integra_ai_export_strategy.php
```

O catálogo antigo declarava duas estratégias:

```php
ia_export_importador_layout_key() => [
  'key' => ia_export_importador_layout_key(),
  'label' => 'Dominio Importador 01/02/03/99',
  'default' => true,
],
'dominio_separador_0000_0451' => [
  'key' => 'dominio_separador_0000_0451',
  'label' => 'Dominio separador 0000/0451',
  'default' => false,
],
```

A função de estratégia padrão retornava o leiaute importador:

```php
function ia_export_strategy_default_key(): string {
  return ia_export_importador_layout_key();
}
```

E a chave concreta era:

```php
function ia_export_importador_layout_key(): string {
  return 'dominio_importador_lancamentos_lote_01_02_03_99';
}
```

### 3.2. Ambiguidade no frontend antigo

O frontend antigo tinha uma mensagem estática informando que o preview representava o leiaute Domínio “com separador `|`”, mesmo quando o backend podia gerar o leiaute fixo `01/02/03/99`.

Esse comportamento é perigoso porque o usuário pode acreditar que está gerando o leiaute com separador oficial, quando na prática o arquivo pode estar sendo gerado em registros fixos `01`, `02`, `03` e `99`.

---

## 4. Leiaute padrão antigo: `01/02/03/99`

### 4.1. Característica geral

O leiaute `01/02/03/99` é um arquivo TXT de largura fixa. Ele não usa separador `|`.

Estrutura geral:

```text
01...cabeçalho...
02...data do lançamento...
03...lançamento contábil...
02...data do lançamento...
03...lançamento contábil...
99...trailer...
```

Para cada lançamento exportado, o sistema gerava um par de registros:

```text
02 + 03
```

O arquivo terminava com o registro `99`.

### 4.2. Encoding e quebra de linha

O código antigo definia:

```php
function ia_export_txt_encoding(): string {
  return 'Windows-1252';
}
```

A geração final concatenava as linhas com `CRLF`:

```text
\r\n
```

Recomendação: manter um teste automatizado validando bytes, encoding e quebra de linha, porque arquivos de leiaute contábil costumam falhar quando espaços finais, acentuação ou quebra de linha são alterados por editor, navegador ou processo de download.

### 4.3. Registro `01` — cabeçalho

Montagem no código antigo:

```php
'01'
. $companyCode
. $cnpj
. $periodStart
. $periodEnd
. 'N'
. $headerSuffix;
```

Tamanho esperado pelo validador antigo: **54 caracteres**.

Campos:

| Ordem | Campo | Tamanho | Regra |
|---:|---|---:|---|
| 1 | Identificador | 2 | Fixo `01` |
| 2 | Código da empresa | 7 | Numérico, preenchido com zeros à esquerda |
| 3 | CNPJ | 14 | Numérico, preenchido com zeros à esquerda |
| 4 | Data inicial | 10 | `dd/mm/aaaa` |
| 5 | Data final | 10 | `dd/mm/aaaa` |
| 6 | Indicador | 1 | Fixo `N` |
| 7 | Sufixo do cabeçalho | 10 | Numérico, padrão antigo `0500000117` |

Exemplo estrutural:

```text
01CCCCCCCDDDDDDDDDDDDDDdd/mm/aaaadd/mm/aaaaNSSSSSSSSSS
```

### 4.4. Registro `02` — data do lançamento

Montagem no código antigo:

```php
'02'
. $seqText
. 'X'
. $txDate
. str_repeat(' ', 130);
```

Tamanho esperado pelo validador antigo: **150 caracteres**.

Campos:

| Ordem | Campo | Tamanho | Regra |
|---:|---|---:|---|
| 1 | Identificador | 2 | Fixo `02` |
| 2 | Sequencial | 7 | Numérico, zeros à esquerda |
| 3 | Literal | 1 | Fixo `X` |
| 4 | Data do lançamento | 10 | `dd/mm/aaaa` |
| 5 | Espaços | 130 | Preenchimento à direita |

### 4.5. Registro `03` — lançamento contábil

Montagem no código antigo:

```php
'03'
. $seqText
. $debit
. $credit
. $amountRaw
. $campo7
. $historyFixed
. $codHistorico;
```

Depois o sistema completava a linha com espaços até atingir **664 caracteres**.

Campos principais:

| Ordem | Campo | Tamanho | Regra |
|---:|---|---:|---|
| 1 | Identificador | 2 | Fixo `03` |
| 2 | Sequencial | 7 | Numérico, zeros à esquerda |
| 3 | Conta débito | 7 | Código reduzido, zeros à esquerda |
| 4 | Conta crédito | 7 | Código reduzido, zeros à esquerda |
| 5 | Valor | 15 | Valor em centavos, sem vírgula, zeros à esquerda |
| 6 | Campo auxiliar | 7 | Espaços quando vazio |
| 7 | Histórico | 512 | Texto normalizado e preenchido com espaços à direita |
| 8 | Código do histórico | 7 | Numérico, padrão `0000000` |
| 9 | Preenchimento final | variável | Espaços até 664 caracteres |

### 4.6. Registro `99` — trailer

Montagem no código antigo:

```php
str_repeat('9', 100)
```

Tamanho esperado: **100 caracteres**.

Conteúdo:

```text
9999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999
```

### 4.7. Regras de sequencial

O sequencial era incrementado linha a linha:

```text
01                         cabeçalho sem sequencial
02 0000001                 primeira linha do lançamento
03 0000002                 corpo do primeiro lançamento
02 0000003                 segunda linha do lançamento
03 0000004                 corpo do segundo lançamento
99                         trailer
```

Ou seja, cada lançamento contábil consome dois sequenciais: um no registro `02` e outro no registro `03`.

---

## 5. Leiaute alternativo antigo: `0000/0451` com separador

### 5.1. Relação com a documentação oficial da Domínio

A documentação oficial informada no suporte da Domínio é para o leiaute:

```text
Leiaute Domínio Sistemas com Separador
```

Esse leiaute usa o caractere `|` como separador e possui registros como:

```text
0000|...
0451|...
```

### 5.2. Registro `0000` — identificação da empresa

Formato gerado pelo código antigo:

```text
0000|INSCRICAO_DA_EMPRESA
```

No código antigo:

```php
'content' => '0000|' . $companyInscricao
```

A inscrição deve ser CNPJ, CPF, CEI ou CAEPF, apenas números.

### 5.3. Registro `0451` — lançamento contábil

Formato gerado pelo código antigo:

```text
0451|CONTA_DEBITO|CONTA_CREDITO|VALOR|CODIGO_HISTORICO|DESCRICAO_HISTORICO
```

No código antigo:

```php
$fields = ['0451', $debit, $credit, $amountCents, $codHistorico, $history];
'content' => implode('|', $fields)
```

Campos:

| Ordem | Campo | Regra |
|---:|---|---|
| 1 | Identificador | Fixo `0451` |
| 2 | Conta débito | Código reduzido da conta contábil |
| 3 | Conta crédito | Código reduzido da conta contábil |
| 4 | Valor | Decimal com 2 casas, enviado sem vírgula no código antigo |
| 5 | Código do histórico | Código numérico do histórico |
| 6 | Descrição do histórico | Texto do histórico |

Exemplo:

```text
0000|12345678000199
0451|0010104|0011101|150000|1234567|PIX RECEBIDO CLIENTE ALFA
0451|0020001|0010104|42000|1234567|PAGAMENTO FORNECEDOR BETA
```

---

## 6. Diagnóstico do problema atual

Pelas imagens e pelo arquivo TXT analisado, o sistema está em uma situação de conflito:

- a tela oferece mais de um leiaute Domínio;
- o arquivo TXT enviado no exemplo está em formato fixo `01/02/03/99`;
- a documentação oficial indicada é do leiaute com separador `0000/0451`;
- a tela do Domínio acusa linhas menores do que o esperado para o leiaute selecionado.

Esse erro costuma acontecer quando o arquivo gerado não corresponde exatamente ao leiaute selecionado na importação do Domínio.

Exemplo prático:

- se o Domínio espera `0000|...` e `0451|...`, um arquivo iniciado por `01...` não será compatível;
- se o Domínio espera um importador customizado de largura fixa, a especificação exata desse importador precisa estar versionada junto ao projeto;
- se o Portal Sama permitir dois leiautes parecidos na UI, o usuário pode selecionar um formato e gerar outro.

---

## 7. Decisão recomendada para o Portal Sama

### 7.1. Leiaute canônico recomendado

Recomendação para produção:

```text
dominio_separador_0000_0451
```

Nome de exibição:

```text
Domínio Sistemas com Separador — 0000/0451
```

Motivo:

- é o leiaute diretamente alinhado com a documentação oficial enviada;
- usa registros documentados `0000` e `0451`;
- usa separador `|`, reduzindo risco de erro por tamanho fixo, espaços finais e padding;
- é mais simples de validar e auditar;
- evita depender de um importador customizado não documentado dentro do projeto.

### 7.2. Exceção

Se o escritório já tiver um **Conjunto de Dados / Importador customizado** homologado para `01/02/03/99`, então o Portal Sama pode manter esse formato, mas somente se forem cumpridas as seguintes condições:

1. anexar ao repositório a especificação oficial ou XML do conjunto de dados usado no Domínio;
2. criar testes automatizados com arquivo real aprovado pelo escritório;
3. remover o leiaute `0000/0451` da UI de produção;
4. exibir apenas “Domínio Importador 01/02/03/99” como leiaute ativo;
5. bloquear qualquer alteração de tamanho de linha, encoding ou campos sem passar por homologação.

Com a evidencia atual de importacao em `Lancamentos Contabeis em Lote`, o leiaute `01/02/03/99` deixa de ser excecao e passa a ser o leiaute oficial deste fluxo.

---

## 8. Mudanças obrigatórias no código

### 8.1. Remover seletor livre de leiaute no frontend

A UI não deve permitir escolher entre:

```text
Dominio 01/02/03/99
Dominio 0000/0451
```

Deve existir somente um leiaute ativo por ambiente.

Sugestão:

```text
Leiaute de exportação: Domínio Sistemas com Separador — 0000/0451
```

Esse valor deve ser apenas informativo, não um campo editável pelo usuário comum.

### 8.2. Definir constante única no backend

Criar uma constante única, por exemplo:

```php
const SAMA_DOMINIO_LAYOUT = 'dominio_separador_0000_0451';
```

Ou configurar por variável de ambiente apenas para implantação/homologação:

```text
SAMA_DOMINIO_LAYOUT=dominio_separador_0000_0451
```

A aplicação não deve aceitar um `layout_key` arbitrário vindo do frontend.

### 8.3. Rejeitar payload divergente

No backend, antes de gerar o TXT:

- ignorar `layout_key` recebido do frontend; ou
- validar que ele é exatamente igual ao leiaute canônico; ou
- retornar erro `layout_not_allowed`.

Exemplo de regra:

```php
if ($requestedLayout !== SAMA_DOMINIO_LAYOUT) {
  throw new RuntimeException('layout_not_allowed');
}
```

### 8.4. Atualizar preview

Se o leiaute canônico for `0000/0451`, o preview deve mostrar linhas como:

```text
0000|12345678000199
0451|0010118|0011100|37279|0000000|Credito domicilio cartao: CARTAO DE CREDITO - AMAZON
```

O preview não pode falar em “separador `|`” quando o arquivo real estiver sendo gerado em largura fixa, nem pode falar em `01/02/03/99` quando o arquivo real for `0000/0451`.

### 8.5. Atualizar validação automatizada

Para `0000/0451`, validar:

- primeira linha começa com `0000|`;
- existe exatamente um registro `0000`;
- todo lançamento começa com `0451|`;
- cada linha `0451` possui 6 campos;
- contas de débito e crédito são numéricas;
- valor é numérico e maior que zero;
- código histórico é numérico;
- histórico não contém `|`, `\r` ou `\n`;
- arquivo não contém registros `01`, `02`, `03` ou `99`.

Para `01/02/03/99`, caso seja mantido por homologação, validar:

- registro `01` com 54 caracteres;
- registro `02` com 150 caracteres;
- registro `03` com 664 caracteres;
- registro `99` com 100 caracteres;
- sequenciais crescentes;
- encoding e espaços finais preservados.

---

## 9. Segurança e governança

Como o Portal Sama manipula documentos e dados empresariais, a exportação Domínio deve seguir controles de segurança específicos.

### 9.1. Controle de acesso

Somente usuários autorizados devem gerar TXT contábil.

Papéis sugeridos:

| Papel | Permissão |
|---|---|
| Colaborador | Preparar classificação e revisar lançamentos próprios |
| Gestor | Aprovar e gerar TXT |
| DEV/Admin | Configurar leiaute, somente em ambiente controlado |

### 9.2. Auditoria

Registrar evento de auditoria para:

- geração de preview;
- geração de TXT final;
- download;
- alteração de conta contábil;
- alteração de regra contábil;
- alteração de código histórico;
- tentativa de usar leiaute não permitido.

O log deve conter:

- `user_id`;
- `company_id`;
- `job_id`;
- data/hora;
- hash do arquivo gerado;
- quantidade de lançamentos;
- leiaute usado;
- resultado: sucesso, bloqueado ou erro.

Não gravar histórico completo, CNPJ completo ou dados bancários sensíveis em logs de aplicação sem mascaramento.

### 9.3. Sanitização

Para o leiaute com separador:

- remover `|` do histórico ou substituir por `/`;
- remover quebras de linha;
- normalizar múltiplos espaços;
- limitar tamanho de campos textuais;
- impedir caracteres de controle invisíveis.

Para o leiaute fixo:

- preservar espaços finais;
- bloquear truncamento silencioso de histórico;
- validar o tamanho real em bytes depois do encoding;
- impedir edição manual do TXT no navegador.

### 9.4. Retenção e criptografia

Arquivos TXT gerados devem ter retenção curta.

Recomendação:

- armazenar arquivo gerado por tempo limitado;
- criptografar em repouso;
- usar URL assinada com expiração curta para download;
- impedir indexação pública;
- não enviar TXT por e-mail sem criptografia;
- permitir revogação do arquivo gerado.

### 9.5. Integridade

Gerar hash SHA-256 do TXT final:

```text
sha256:<hash_do_arquivo>
```

Exibir no histórico do job e no log de auditoria.

Esse hash permite provar que o arquivo importado foi exatamente o arquivo gerado pelo Portal Sama.

---

## 10. Plano de implementação sugerido

### Fase 1 — Decisão de leiaute

- Definir oficialmente o leiaute canônico.
- Recomendado: `dominio_separador_0000_0451`.
- Remover a escolha de leiaute da interface de usuário comum.

### Fase 2 — Refatoração do backend

- Criar constante/enum única de leiaute Domínio.
- Remover fallback automático para `01/02/03/99`.
- Rejeitar payload divergente.
- Criar serviço isolado: `DominioLayoutExporter`.

### Fase 3 — Refatoração do frontend

- Exibir somente o leiaute ativo.
- Ajustar preview para o formato real.
- Adicionar mensagens claras de validação.
- Remover labels ambíguas.

### Fase 4 — Testes

Criar fixtures:

```text
fixtures/dominio/separador_0000_0451_valido.txt
fixtures/dominio/separador_0000_0451_invalido_pipe_no_historico.txt
fixtures/dominio/separador_0000_0451_invalido_conta_vazia.txt
fixtures/dominio/separador_0000_0451_invalido_valor_zero.txt
```

Testes obrigatórios:

- geração do TXT;
- validação de campos;
- encoding;
- quantidade de lançamentos;
- ausência de registros de outro leiaute;
- comparação com golden file homologado.

### Fase 5 — Homologação no Domínio

- Importar um arquivo real de teste no Domínio.
- Confirmar que não há advertências ou erros de estrutura.
- Salvar evidência da importação.
- Congelar o arquivo aprovado como golden file.

---

## 11. Prompt recomendado para o Codex

```text
Analise a implementação do Integra-AI no Portal Sama e refatore a exportação Domínio para trabalhar com somente um leiaute canônico.

Leiaute canônico definido: dominio_separador_0000_0451.

Objetivos:
1. Remover do frontend qualquer seletor entre Domínio 01/02/03/99 e Domínio 0000/0451.
2. Exibir apenas o texto informativo: "Domínio Sistemas com Separador — registros 0000/0451".
3. Remover ou isolar em legado a estratégia dominio_importador_lancamentos_lote_01_02_03_99.
4. Impedir que o frontend envie layout_key arbitrário para geração do TXT.
5. No backend, validar que o único leiaute permitido é dominio_separador_0000_0451.
6. Gerar o TXT no formato:
   - 0000|INSCRICAO_EMPRESA
   - 0451|CONTA_DEBITO|CONTA_CREDITO|VALOR|CODIGO_HISTORICO|DESCRICAO_HISTORICO
7. Sanitizar histórico removendo pipe, quebras de linha e caracteres de controle.
8. Criar testes automatizados com golden file.
9. Atualizar preview para mostrar exatamente o TXT final.
10. Criar auditoria de geração e download contendo user_id, company_id, job_id, layout, quantidade de lançamentos e hash SHA-256 do arquivo.
11. Não registrar CNPJ completo, histórico completo nem dados bancários sensíveis em logs comuns.

Antes de alterar o código, liste os arquivos impactados e explique a estratégia de migração.
```

---

## 12. Decisão final documentada

A aplicação não deve manter dois leiautes Domínio ativos para o mesmo fluxo de exportação contábil.

Decisão recomendada:

```text
Usar somente o leiaute Domínio Sistemas com Separador — registros 0000/0451.
```

Motivo:

- está alinhado com a documentação oficial indicada;
- reduz erro de importação;
- simplifica preview e validação;
- melhora segurança e auditoria;
- evita que o usuário gere um formato e selecione outro no Domínio.

O leiaute antigo `01/02/03/99` deve ser tratado como legado técnico e removido da produção, salvo se houver homologação formal do importador customizado correspondente.

---

## 13. Implementacao aplicada no Portal Sama em 2026-05-27 12:16 -03:00

O Portal Sama passou a adotar no fluxo principal somente o leiaute:

```text
dominio_separador_0000_0451
Dominio Sistemas com Separador - 0000/0451
```

### 13.1. Comportamento do backend

- A constante oficial fica centralizada no motor do Integra-AI.
- `sanitizeStep3` normaliza configuracoes antigas para o leiaute oficial.
- A API rejeita atualizacao explicita de `export_strategy` divergente com `INTEGRA_AI_LAYOUT_NOT_ALLOWED`.
- O preview e a exportacao geram exclusivamente:

```text
0000|INSCRICAO_EMPRESA
0451|CONTA_DEBITO|CONTA_CREDITO|VALOR|CODIGO_HISTORICO|DESCRICAO_HISTORICO
```

- Retificacao 14:06: esta decisao foi substituida. O leiaute `01/02/03/99` e o leiaute oficial do fluxo `Lancamentos Contabeis em Lote`.

### 13.2. Validacoes antes da exportacao

O backend valida:

- primeira linha iniciando com `0000|`;
- exatamente um registro `0000`;
- somente registros `0000` e `0451`;
- cada `0451` com 6 campos;
- conta debito, conta credito, valor e codigo de historico numericos;
- valor maior que zero;
- historico obrigatorio, sem `|`, sem CR/LF e com espacos normalizados;
- ausencia de registros legados `01`, `02`, `03` e `99`.

Se a geracao nao puder ocorrer, a API retorna erro seguro de prontidao/exportacao, sem expor conteudo financeiro em logs comuns.

### 13.3. Comportamento do frontend

- A tela `/contabil/integra-ai` nao apresenta mais seletor entre leiautes Dominio.
- A etapa de plano exibe somente o texto do leiaute oficial.
- O payload de configuracoes envia sempre `dominio_separador_0000_0451`.

### 13.4. Seguranca aplicada

- Geracao exige `accounting.integra_ai.export`.
- Download exige `accounting.integra_ai.download`.
- Exportacao/download verificam escopo contabil e escopo da empresa quando a empresa possui departamentos no cadastro.
- TXT continua em storage privado e o download passa pela API protegida.
- `Content-Disposition` usa nome sanitizado; a chave de storage e resolvida dentro de `STORAGE_PRIVATE_PATH`, bloqueando path traversal.
- Download retorna `Cache-Control: no-store`, `Pragma: no-cache`, `Expires: 0` e `X-Content-Type-Options: nosniff`.
- A geracao registra SHA-256 e tamanho do arquivo no metadata do job.
- Auditoria de exportacao/download registra usuario, job, empresa, leiaute, status da operacao e hash quando disponivel.

### 13.5. Testes automatizados

Foram adicionados/ajustados testes para:

- geracao valida do arquivo oficial `0000/0451`;
- normalizacao de configuracao legada para leiaute oficial;
- rejeicao de registros legados ou linha `0451` incompleta;
- bloqueio por campos contabeis obrigatorios ausentes;
- rejeicao de update com leiaute legado;
- bloqueio por escopo de empresa/departamento;
- ausencia do seletor de leiaute conflitante no frontend;
- preservacao do download autenticado por blob.

### 13.6. Homologacao manual ainda obrigatoria

Antes de considerar a exportacao pronta para producao, ainda e necessario importar um TXT real gerado pelo Portal Sama no Dominio Contabilidade Fiscal, confirmar que o importador selecionado e o de separador `0000/0451`, salvar evidencia do aceite e congelar um arquivo aprovado como golden file.

---

## 14. Retificacao operacional em 2026-05-27 14:06 -03:00

Esta secao substitui a decisao anterior que apontava `dominio_separador_0000_0451` como leiaute principal.

### 14.1. Evidencia nova

O erro atual do Dominio mostra que o arquivo foi importado no conjunto correto:

```text
Lancamentos Contabeis em Lote
```

Nesse conjunto, o Dominio interpreta os dois primeiros caracteres da linha como tipo de registro fixo. Por isso uma linha com separador:

```text
0451|0010118|0011100|37279|0000000|Credito...
```

foi lida como registro `04 - Rateios Gerenciais`, deslocando os campos:

- `51|0010` entrou como sequencial;
- `118|001` entrou como conta debito;
- `1100|37` entrou como conta credito;
- `279|0000000|Cre` entrou como valor.

Portanto, `0000/0451` nao serve para o fluxo atual de importacao usado pelo escritorio.

### 14.2. Leiaute oficial corrigido

Para o fluxo atual do Portal Sama / Integra-AI, o leiaute oficial passa a ser:

```text
dominio_importador_lancamentos_lote_01_02_03_99
Dominio Importador 01/02/03/99
```

O leiaute `01/02/03/99` nao e deprecated neste fluxo. Ele e o leiaute oficial para importacao em `Lancamentos Contabeis em Lote`.

O leiaute `dominio_separador_0000_0451` deve ficar fora do fluxo principal e nao deve ser aceito em `export_strategy` para a exportacao atual.

### 14.3. Estrutura obrigatoria do TXT

O TXT deve ser gerado sem pipes, com `CRLF`, em `Windows-1252`:

```text
01...cabecalho...
02...data do lancamento...
03...lancamento contabil...
02...data do lancamento...
03...lancamento contabil...
9999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999
```

Tamanhos obrigatorios:

- registro `01`: 54 caracteres;
- registro `02`: 150 caracteres;
- registro `03`: 664 caracteres;
- registro `99`: 100 caracteres preenchidos com `9`.

Cada lancamento exportado gera um par `02`/`03`. O registro `02` contem sequencial, literal `X`, data `dd/mm/aaaa` e preenchimento. O registro `03` contem sequencial, conta debito, conta credito, valor em centavos, campo auxiliar de 7 caracteres, historico fixo de 512 caracteres e codigo historico de 7 caracteres.

### 14.4. Como importar no Dominio

No Dominio Contabilidade Fiscal, usar o conjunto:

```text
Lancamentos Contabeis em Lote
```

Nao selecionar importador de leiaute com separador para este arquivo. O arquivo correto deve iniciar com `01`, nao deve conter `|`, deve conter pares `02`/`03` e deve terminar com a linha `99` de 100 caracteres.

### 14.5. Validacoes automatizadas obrigatorias

O Portal Sama deve bloquear:

- arquivo contendo `|`;
- arquivo iniciando com `0000` ou contendo `0451`;
- registro `01` diferente de 54 caracteres;
- registro `02` diferente de 150 caracteres;
- registro `03` diferente de 664 caracteres;
- registro `99` diferente de 100 caracteres preenchidos com `9`;
- pares `02`/`03` fora de ordem;
- contas, valor ou codigo historico nao numericos;
- valor zerado;
- historico vazio.

### 14.6. Homologacao ainda necessaria

A correcao automatizada valida a estrutura do arquivo, mas ainda e necessario importar um TXT real no Dominio usando `Lancamentos Contabeis em Lote`, salvar evidencia sanitizada e congelar um golden file aprovado.
