# [CONCLUIDO] Portal Sama — Asset notes v1.0

Pacote produzido seguindo a documentação original e a direção visual aprovada após ajuste da logo.

## Decisão visual aprovada

A logo central foi tratada como elemento de maior fidelidade: esfera/S orgânico com laranja no lado esquerdo/inferior, azul no lado direito/superior, volume glossy/tech, brilho interno e bordas luminosas. A direção evita o símbolo genérico abstrato e aproxima o visual das cenas de referência `Logo-Sama.png` e `Boas-vindas.png`.

## Estrutura entregue

- `shared/`: fundos, vignette, glow central, partículas gerais e ruído sutil.
- `logo/`: logo isolada, halo, glows por cor, sombra inferior e orbitais SVG.
- `loading/`: light streak e fallback estático da cena de carregamento.
- `welcome/`: ondas laterais, partículas laterais e fallback estático da cena de boas-vindas.
- `previews/`: previews de loading, welcome, logo isolada e contato visual dos assets.
- `manifest.json`: ordem visual, `zIndex`, blend modes, opacidades, obrigatoriedade e grupos de animação.

## Regras preservadas

- Assets modulares não têm texto embutido.
- Fallbacks estáticos têm texto embutido, conforme especificação.
- Camadas não-background usam transparência real.
- Orbitais foram mantidos como SVG simples, com poucos paths e sem base64.
- A composição mantém paleta navy/azul/laranja da documentação.
- `logo-core` não carrega halo grande, sombra de chão ou texto.

## Observações de implementação para o Codex

1. Renderizar camadas fullscreen com `object-fit: cover`.
2. Renderizar camadas de logo com `object-fit: contain` e centro visual comum.
3. Aplicar `mix-blend-mode: screen` nas camadas de glow, partículas, ondas e light streak.
4. Usar `loading/static-loading.webp` e `welcome/static-welcome.webp` quando `prefers-reduced-motion`, falha de asset obrigatório ou modo estático estiver ativo.
5. O texto dinâmico deve ser HTML/CSS no runtime; somente os fallbacks já possuem texto rasterizado.

## Validação de produção

- [x] logo-core sem texto
- [x] logo-core com transparência
- [x] halo separado
- [x] glows separados
- [x] sombra inferior separada
- [x] orbitais simples em SVG
- [x] ondas welcome em camadas próprias
- [x] partículas discretas
- [x] fundo sem logo embutida
- [x] fallbacks 16:9
- [x] previews 16:9
- [x] documentação atualizada no pacote

Data da geração: 2026-05-21

## Atualizacao v1.0.1 - Handoff para Codex

Foi adicionado `docs/codex-handoff.md` com orientacoes objetivas para integracao no Codex:

- ordem visual por cena;
- regras CSS para camadas fullscreen e camadas da logo;
- estados `loading`, `welcome` e `static`;
- textos em HTML/CSS;
- recomendacoes de animacao;
- criterio de fallback por `prefers-reduced-motion` ou falha de asset obrigatorio;
- criterios de aceite para QA visual.

O handoff nao altera os assets visuais aprovados. Ele formaliza a proxima etapa de implementacao.
