# Portal Sama — Handoff para integração no Codex

## Objetivo

Integrar o pacote `portal-sama-intro-assets` na intro web do Portal Sama usando composição por camadas, preservando a fidelidade visual aprovada na entrega v1.0.

A intro deve usar os assets modulares para a experiência animada e os fallbacks estáticos quando movimento reduzido, erro de asset ou modo estático estiver ativo.

## Fontes operacionais

- `manifest.json`: fonte principal para ordem visual, `zIndex`, opacidade, blend mode, obrigatoriedade e grupos de animação.
- `asset-notes.md`: observações visuais e restrições de uso.
- `documentacao_assets_visuais_intro_portal_sama.md`: documentação completa de produção e decisão visual aprovada.

## Estrutura esperada de publicação

```txt
/public/brand/sama/intro/assets/
  shared/
  logo/
  loading/
  welcome/
  previews/
  manifest.json
  asset-notes.md
```

## Regras de renderização

### Camadas fullscreen

Usar para `shared/*`, ondas, partículas laterais e `loading/light-streak.webp`.

```css
position: absolute;
inset: 0;
width: 100%;
height: 100%;
object-fit: cover;
pointer-events: none;
```

### Camadas da logo

Usar para `logo/*` raster e SVG.

```css
position: absolute;
left: 50%;
top: var(--logo-y);
width: var(--logo-size);
height: var(--logo-size);
transform: translate(-50%, -50%);
object-fit: contain;
pointer-events: none;
```

Centros recomendados:

```txt
loading: 50% x 44%
welcome: 50% x 38%
```

## Ordem visual

A ordem deve ser derivada do `manifest.json`. Não reordenar manualmente, exceto por ajuste documentado.

### Loading

```txt
background-deep-navy
background-vignette
center-blue-glow
particles-soft
logo-halo-blue
logo-shadow
logo-orbit-back
logo-glow-orange
logo-glow-blue
logo-core
logo-orbit-front
light-streak
HTML text: CARREGANDO
```

### Welcome

```txt
background-deep-navy
background-vignette
center-blue-glow
orange-wave-left
blue-wave-right
orange-particles
blue-particles
logo-halo-blue
logo-shadow
logo-orbit-back
logo-glow-orange
logo-glow-blue
logo-core
logo-orbit-front
HTML text: Seja bem-vindo ao Portal Sama
```

## Estados da intro

### `loading`

Cena inicial, mais sóbria, com foco no centro.

- Logo menor.
- Orbitais discretos.
- Light streak abaixo da logo.
- Texto `CARREGANDO` pequeno, espaçado e institucional.

### `welcome`

Cena de entrada institucional.

- Logo no centro superior.
- Ondas laterais laranja e azul.
- Texto maior abaixo da logo.
- `Portal` em azul e `Sama` em laranja.

### `static`

Usar fallback fechado:

```txt
loading/static-loading.webp
welcome/static-welcome.webp
```

Acionar quando:

- `prefers-reduced-motion: reduce` estiver ativo;
- asset obrigatório falhar;
- modo estático for forçado por configuração;
- dispositivo não sustentar bem a composição animada.

## Regras de texto

Os textos da experiência animada devem ser HTML/CSS. Não rasterizar texto em camadas modulares.

### Loading

```txt
CARREGANDO
```

Estilo recomendado:

```css
font-size: clamp(12px, 1.1vw, 18px);
letter-spacing: 0.42em;
font-weight: 500;
color: #F4F7FF;
opacity: 0.92;
text-transform: uppercase;
```

### Welcome

```txt
Seja bem-vindo ao Portal Sama
```

Estilo recomendado:

```css
font-size: clamp(30px, 4.2vw, 72px);
line-height: 1.08;
font-weight: 400;
color: #F4F7FF;
```

Cores:

```css
.portal-word { color: #0078FF; }
.sama-word { color: #FF8A00; }
```

## Animações recomendadas

### `ambientPulse`

Aplicar em `center-blue-glow`.

```txt
opacity: 0.55 -> 0.85 -> 0.55
scale: 0.98 -> 1.03 -> 0.98
duration: 4s a 6s
ease: sine.inOut
repeat: infinite
```

### `logoFloat`

Aplicar em `logo-core` e `logo-shadow`.

```txt
y: -4px -> 4px -> -4px
scale: 0.995 -> 1.005 -> 0.995
duration: 3.5s a 5s
ease: sine.inOut
repeat: infinite
```

### `logoPulse`

Aplicar em halo e glows.

```txt
opacity: base * 0.75 -> base -> base * 0.75
scale: 0.99 -> 1.04 -> 0.99
duration: 3s a 4.5s
ease: sine.inOut
repeat: infinite
```

### `orbitSlow`

Aplicar em `logo-orbit-back.svg`.

```txt
rotate: 0deg -> 360deg
duration: 18s a 28s
ease: linear
repeat: infinite
```

### `orbitReverse`

Aplicar em `logo-orbit-front.svg`.

```txt
rotate: 360deg -> 0deg
duration: 12s a 20s
ease: linear
repeat: infinite
```

### `particlesDrift`

Aplicar em partículas.

```txt
x/y drift pequeno: -8px a 8px
opacity sutil
duration: 8s a 14s
ease: sine.inOut
repeat: infinite
```

### `waveInLeft` e `waveInRight`

Aplicar somente na cena welcome.

```txt
entrada lateral curta
opacity: 0 -> base
x: -32px/32px -> 0
duration: 0.8s a 1.4s
ease: power2.out
```

## Performance

- Usar lazy/preload apenas conforme necessidade da intro.
- Precarregar `logo/logo-core.webp`, `shared/background-deep-navy.webp` e fallback da cena inicial.
- Se algum asset `required: true` falhar, alternar para fallback estático.
- Assets opcionais podem falhar sem bloquear a intro.
- Respeitar `prefers-reduced-motion`.

## Critérios de aceite

- A cena loading lembra `Logo-Sama.png`.
- A cena welcome lembra `Boas-vindas.png`.
- A logo mantém a direção aprovada: símbolo circular/orgânico em S, laranja à esquerda/inferior e azul à direita/superior.
- Assets modulares continuam sem texto.
- Fallbacks estáticos funcionam isoladamente.
- Camadas usam a ordem do `manifest.json`.
- Orbitais e glows permanecem independentes.
- Não há checkerboard visível.
- Texto animado é HTML/CSS.

## Próxima etapa recomendada

Implementar um componente React `PortalSamaIntro` que aceite:

```ts
type PortalSamaIntroMode = "loading" | "welcome" | "auto" | "static";
```

E exponha propriedades para:

```ts
assetBasePath
mode
reducedMotion
onReady
onComplete
className
```


---

# Portal Sama — Padrão visual das páginas do projeto

## Objetivo

Definir uma estilização padrão para as páginas internas do Portal Sama com base no painel operacional aprovado visualmente.

A direção visual deve manter a identidade premium/tech dos assets da intro, mas adaptada para uso diário em telas funcionais: leitura clara, hierarquia forte, contraste consistente, cards reutilizáveis e navegação estável.

## Direção visual principal

O layout base deve combinar:

```txt
sidebar escura institucional
área de conteúdo clara e ampla
cards glassmorphism suaves
bordas luminosas azul/laranja
ícones lineares brancos/azuis
fundos com ondas e partículas discretas
sombras profundas porém controladas
```

A página não deve parecer um dashboard genérico. Ela deve carregar a mesma linguagem visual da marca Sama: navy profundo, azul elétrico, laranja de acento, brilho controlado e acabamento premium.

## Estrutura padrão de página

```txt
<AppShell>
  <Sidebar />
  <MainArea>
    <Topbar />
    <PageHeader />
    <PageContent />
  </MainArea>
</AppShell>
```

### Sidebar

A sidebar deve ser o elemento mais marcante da interface.

Características:

```txt
largura desktop: 300px a 320px
fundo navy profundo com textura/partículas discretas
logo no topo
itens de menu com ícone + label
item ativo com gradiente azul, glow e borda luminosa
separador inferior antes de "Recolher menu"
```

Comportamento:

```txt
desktop: fixa à esquerda
tablet: recolhível
mobile: drawer/off-canvas
```

### Área principal

A área principal deve ser clara, limpa e espaçosa.

Características:

```txt
background base: azul muito claro / branco azulado
ondas suaves no topo direito
padding desktop: 40px a 56px
padding mobile: 20px a 24px
max-width opcional para conteúdos densos
```

### Topbar

A topbar deve ficar no topo da área principal, alinhada à direita.

Elementos:

```txt
avatar circular com iniciais
nome do usuário
menu/dropdown
botão sair
```

### PageHeader

Deve conter:

```txt
eyebrow azul: Portal Sama
h1 forte: título da página
subtítulo opcional
```

Exemplo:

```txt
Portal Sama
Painel operacional
```

## Tokens de design

### Cores

```css
:root {
  --sama-navy-950: #020817;
  --sama-navy-900: #031B46;
  --sama-navy-800: #062E82;

  --sama-blue-700: #005BEA;
  --sama-blue-600: #006CFF;
  --sama-blue-500: #0078FF;
  --sama-cyan-400: #00CFFF;
  --sama-cyan-200: #9AFFFF;

  --sama-orange-700: #B84A00;
  --sama-orange-600: #FF7800;
  --sama-orange-500: #FF8A00;
  --sama-gold-400: #FFB000;

  --sama-bg-page: #EEF5FF;
  --sama-bg-card: rgba(255, 255, 255, 0.68);
  --sama-bg-card-strong: rgba(255, 255, 255, 0.82);

  --sama-text-primary: #090B22;
  --sama-text-secondary: #153264;
  --sama-text-inverse: #F4F7FF;
  --sama-border-soft: rgba(0, 108, 255, 0.22);
  --sama-border-warm: rgba(255, 138, 0, 0.48);
}
```

### Gradientes

```css
--sama-gradient-sidebar:
  radial-gradient(circle at 80% 42%, rgba(0, 120, 255, 0.28), transparent 34%),
  radial-gradient(circle at 20% 86%, rgba(255, 138, 0, 0.20), transparent 28%),
  linear-gradient(180deg, #020817 0%, #031B46 48%, #020817 100%);

--sama-gradient-active:
  linear-gradient(135deg, rgba(0, 108, 255, 0.95), rgba(0, 207, 255, 0.55));

--sama-gradient-panel-dark:
  radial-gradient(circle at 94% 0%, rgba(0, 207, 255, 0.28), transparent 28%),
  linear-gradient(135deg, #020817 0%, #031B46 46%, #001A55 100%);
```

### Sombras

```css
--sama-shadow-card: 0 18px 42px rgba(3, 27, 70, 0.18);
--sama-shadow-card-hover: 0 24px 56px rgba(0, 108, 255, 0.22);
--sama-shadow-blue-glow: 0 0 0 1px rgba(0, 207, 255, 0.26), 0 0 34px rgba(0, 108, 255, 0.42);
--sama-shadow-orange-edge: -2px 0 0 rgba(255, 138, 0, 0.88);
```

### Raios e espaçamento

```css
--sama-radius-sm: 12px;
--sama-radius-md: 18px;
--sama-radius-lg: 24px;
--sama-radius-xl: 30px;

--sama-space-page-x: clamp(24px, 4vw, 56px);
--sama-space-section: 32px;
--sama-space-card: 24px;
```

## Componentes padrão

### Card KPI

Usar em métricas e resumo de página.

Características:

```txt
fundo branco glass
borda azul suave
filete laranja à esquerda
ícone circular com glow azul
valor grande em navy
label e descrição em azul escuro
hover com elevação leve
```

Estrutura:

```tsx
<MetricCard
  icon={ShieldCheck}
  label="Permissões"
  value="55"
  description="Total de permissões"
/>
```

### Painel escuro de destaque

Usar para atalhos, módulos principais ou seções de navegação.

Características:

```txt
fundo navy profundo
borda azul luminosa
glow no canto superior direito
partículas discretas no fundo
header branco
sublinhado/acento laranja
```

### Card de atalho

Características:

```txt
fundo navy translúcido
borda azul com acento laranja discreto
ícone em botão quadrado arredondado azul
label branco
chevron à direita
linha/glow inferior azul no hover
```

Estados:

```txt
default: contido e legível
hover: sobe 2px, aumenta glow azul
active/focus: outline ciano acessível
pressed: reduz escala para 0.99
```

### Botões

#### Primário

```txt
fundo gradiente azul
texto branco
borda ciano suave
shadow blue glow
```

#### Secundário

```txt
fundo glass claro
texto navy
borda azul suave
```

#### Destrutivo / sair

```txt
fundo glass claro
ícone navy
borda azul suave
hover com borda laranja discreta
```

### Tabelas

Tabelas devem seguir a estética clara da área principal.

```txt
container glass branco
header com fundo azul muito claro
linhas com hover azul claro
bordas suaves
ações em ícones circulares
status badges com cor sem exagero
```

### Formulários

```txt
label navy médio
input branco/glass
borda azul suave
focus ring azul/ciano
mensagem de erro com laranja/vermelho controlado
```

## Classes utilitárias recomendadas

```css
.sama-glass-card {
  background: var(--sama-bg-card);
  border: 1px solid var(--sama-border-soft);
  border-left: 2px solid var(--sama-border-warm);
  border-radius: var(--sama-radius-lg);
  box-shadow: var(--sama-shadow-card);
  backdrop-filter: blur(18px);
}

.sama-dark-panel {
  background: var(--sama-gradient-panel-dark);
  border: 1px solid rgba(0, 207, 255, 0.42);
  border-radius: var(--sama-radius-xl);
  box-shadow: var(--sama-shadow-blue-glow), 0 28px 54px rgba(2, 8, 23, 0.32);
  position: relative;
  overflow: hidden;
}

.sama-active-nav-item {
  background: var(--sama-gradient-active);
  color: var(--sama-text-inverse);
  border: 1px solid rgba(82, 237, 255, 0.74);
  box-shadow: 0 0 30px rgba(0, 108, 255, 0.72);
}
```

## Aplicação em páginas internas

Todas as páginas devem reutilizar o mesmo `AppShell`.

### Página simples

```txt
Header
Card ou tabela principal
Ações no topo direito
```

### Página de listagem

```txt
Header
Barra de filtros
Tabela glass
Paginação
```

### Página de detalhe

```txt
Header com status
Grid de cards informativos
Tabs ou seções internas
Histórico/auditoria em painel separado
```

### Página de formulário

```txt
Header
Card principal com formulário
Ações fixadas no rodapé do card
Validação clara
```

## Responsividade

### Desktop

```txt
sidebar fixa
cards KPI em 4 colunas
atalhos em 4 colunas
padding amplo
```

### Tablet

```txt
sidebar recolhida ou overlay
cards KPI em 2 colunas
atalhos em 2 colunas
```

### Mobile

```txt
sidebar como drawer
cards em 1 coluna
atalhos em 1 coluna
header mais compacto
avatar/menu agrupado
```

## Acessibilidade

- Manter contraste mínimo entre texto e fundo.
- Todo item interativo deve ter estado de foco visível.
- Não depender apenas de cor para status.
- Animações devem respeitar `prefers-reduced-motion`.
- Textos em cards escuros devem usar branco ou cinza muito claro.
- Evitar glow atrás de textos pequenos.

## O que evitar

```txt
não usar fundo claro puro sem textura/ondas
não exagerar partículas em páginas funcionais
não aplicar glow pesado em todo componente
não transformar todos os cards em painéis escuros
não misturar muitos tons fora da paleta Sama
não rasterizar textos da interface
não criar páginas com estilos divergentes do AppShell
```

## Critério de aceite visual

Uma página nova está aderente ao padrão se:

```txt
[ ] usa AppShell com sidebar Sama
[ ] preserva navy/azul/laranja como linguagem principal
[ ] usa fundo claro azulado na área de conteúdo
[ ] usa cards glass para informações
[ ] usa painel escuro apenas para destaque/seções especiais
[ ] mantém espaçamento amplo
[ ] tem hover/focus consistente
[ ] respeita reduced motion
[ ] parece parte do mesmo produto da intro Portal Sama
```
