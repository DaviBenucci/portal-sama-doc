# Portal Sama — Documentação técnica da intro animada web

## 1. Objetivo

Criar uma intro animada premium para o Portal Sama, adequada para aplicação web, com alta qualidade visual, boa performance em navegador e manutenção simples no projeto.

A solução não deve depender de SVG extremamente detalhado para representar toda a logo. A logo premium e os brilhos complexos devem ser tratados como assets rasterizados em alta qualidade, enquanto SVGs simples, CSS, Motion for React e GSAP serão usados para controlar movimento, composição e efeitos.

A intro deve suportar pelo menos duas cenas:

```txt
loading  -> tela de carregamento com logo, halo, anel, partículas e texto CARREGANDO
welcome  -> tela de boas-vindas com logo, ondas laterais e texto Seja bem-vindo ao Portal Sama
```

O objetivo final é ter uma composição parecida com os exemplos visuais anteriores, mas com arquitetura viável para web.

---

## 2. Decisão técnica principal

A intro será construída com uma arquitetura híbrida:

```txt
React
+ Motion for React
+ GSAP
+ assets PNG/WebP premium
+ SVGs simples para linhas, órbitas e máscaras
+ CSS para layout e blend modes
```

Cada tecnologia terá um papel específico.

### 2.1. React

React será responsável por:

- montar o componente da intro;
- receber propriedades como `scene`, `mode`, `minDurationMs` e `onFinish`;
- controlar estado de exibição;
- decidir quando a intro aparece e quando termina;
- preservar integração com autenticação, carregamento e redirecionamento do Portal Sama;
- renderizar texto dinâmico quando necessário.

React não deve redesenhar a arte.

### 2.2. Motion for React

Motion for React será responsável por:

- entrada e saída do componente;
- fade in e fade out da tela;
- animações simples ligadas a estado;
- integração com `AnimatePresence`;
- suporte a `prefers-reduced-motion`;
- transições de interface.

Motion será usado para comportamento de UI.

### 2.3. GSAP

GSAP será responsável pela timeline cinematográfica da intro:

- sequência de entrada da logo;
- pulso do halo;
- revelação dos anéis orbitais;
- entrada dos glows azul e laranja;
- entrada do texto;
- passagem de linha luminosa;
- animações coordenadas com tempo exato.

GSAP será usado apenas dentro do componente da intro, com escopo controlado, para evitar interferência no restante da aplicação.

### 2.4. PNG/WebP

PNG/WebP será usado para elementos visualmente complexos:

- logo premium final;
- glow azul da logo;
- glow laranja da logo;
- halo difuso;
- sombra inferior;
- light streaks complexos;
- partículas suaves;
- fundo premium, se necessário.

Esses elementos não devem ser recriados como SVG detalhado, pois SVG com muitos detalhes vira pesado, difícil de animar e visualmente inconsistente.

### 2.5. SVG

SVG será usado para elementos simples e animáveis:

- anel orbital traseiro;
- anel orbital frontal;
- linhas finas de energia;
- masks simples;
- strokes reveláveis;
- elementos que precisam de `stroke-dashoffset`;
- contornos simples.

SVG não será usado para tentar reproduzir toda a textura/gloss da logo.

---

## 3. Princípio visual

A intro deve manter a estética:

```txt
tecnologia premium
corporativo sofisticado
fundo navy profundo
alto contraste
luz azul/ciano
luz laranja/dourada
partículas discretas
ondas luminosas laterais
logo central Sama
brilho controlado
sensação de portal digital
```

Paleta recomendada:

```txt
Navy base:       #020817
Navy médio:      #031B46
Azul profundo:   #062E82
Azul elétrico:   #006CFF
Azul vivo:       #0078FF
Ciano:           #00CFFF
Ciano claro:     #52EDFF
Ciano branco:    #9AFFFF
Laranja:         #FF7800
Laranja vivo:    #FF8A00
Laranja claro:   #FF9A00
Dourado:         #FFB000
Dourado claro:   #FFD45A
Texto branco:    #F4F7FF
```

O visual precisa ser escuro, limpo e luminoso. O glow deve valorizar a logo, não lavar a tela.

---

## 4. Estrutura final de assets

Os assets devem ficar em `public`, para serem servidos diretamente pelo servidor web.

Estrutura recomendada:

```txt
public/
  brand/
    sama/
      intro/
        manifest.json
        animation-map.json

        assets/
          shared/
            background-deep-navy.webp
            background-vignette.webp
            center-blue-glow.webp
            particles-soft.webp
            noise-overlay.webp

          logo/
            logo-core.webp
            logo-core@2x.webp
            logo-core.png
            logo-shadow.webp
            logo-halo-blue.webp
            logo-glow-orange.webp
            logo-glow-blue.webp
            logo-orbit-back.svg
            logo-orbit-front.svg

          loading/
            light-streak.webp
            text-loading.svg
            static-loading.webp

          welcome/
            orange-wave-left.webp
            blue-wave-right.webp
            orange-particles.webp
            blue-particles.webp
            text-welcome.svg
            static-welcome.webp
```

### 4.1. Sobre `logo-core.webp`

Este é o asset mais importante.

Ele deve ser uma imagem premium da logo completa, sem fundo, em alta resolução.

Requisitos:

```txt
formato: WebP para produção
fallback: PNG transparente
resolução fonte: 2048x2048 ou 1536x1536
fundo: transparente real
conteúdo: somente a logo premium
sem texto
sem fundo navy embutido
sem sombra exagerada embutida
```

A sombra e os glows devem preferencialmente ficar em arquivos separados, para permitir animação independente.

### 4.2. Sobre glows

Os glows devem ser separados:

```txt
logo-halo-blue.webp      -> halo atrás da logo
logo-glow-orange.webp    -> brilho na região laranja
logo-glow-blue.webp      -> brilho na região azul/ciano
center-blue-glow.webp    -> brilho ambiente central da cena
```

Isso permite animar intensidade, escala e opacidade sem alterar a logo principal.

### 4.3. Sobre órbitas SVG

As órbitas devem ser simples:

```txt
logo-orbit-back.svg
logo-orbit-front.svg
```

Características:

- poucas paths;
- stroke fino;
- gradiente simples;
- sem filtros pesados;
- bom para rotação e stroke reveal;
- alinhamento central com a logo.

---

## 5. Manifesto de assets

Criar um arquivo `manifest.json` para centralizar as camadas e evitar caminhos hardcoded espalhados pelo código.

Exemplo:

```json
{
  "project": "Portal Sama",
  "assetPack": "intro-web-hybrid",
  "version": "1.0.0",
  "basePath": "/brand/sama/intro",
  "scenes": {
    "loading": {
      "label": "Intro de carregamento",
      "fallback": "assets/loading/static-loading.webp",
      "layers": [
        {
          "id": "background",
          "file": "assets/shared/background-deep-navy.webp",
          "zIndex": 0,
          "role": "fundo navy profundo",
          "animationGroup": "background",
          "required": true,
          "fit": "cover"
        },
        {
          "id": "center-blue-glow",
          "file": "assets/shared/center-blue-glow.webp",
          "zIndex": 10,
          "role": "glow azul central",
          "animationGroup": "ambientGlow",
          "required": true,
          "blendMode": "screen",
          "opacity": 0.75,
          "fit": "contain"
        },
        {
          "id": "logo-halo-blue",
          "file": "assets/logo/logo-halo-blue.webp",
          "zIndex": 20,
          "role": "halo atrás da logo",
          "animationGroup": "logoHalo",
          "required": false,
          "blendMode": "screen",
          "opacity": 0.85,
          "fit": "contain"
        },
        {
          "id": "logo-shadow",
          "file": "assets/logo/logo-shadow.webp",
          "zIndex": 30,
          "role": "sombra sob a logo",
          "animationGroup": "logoShadow",
          "required": false,
          "opacity": 0.75,
          "fit": "contain"
        },
        {
          "id": "logo-core",
          "file": "assets/logo/logo-core.webp",
          "zIndex": 40,
          "role": "logo Sama premium completa",
          "animationGroup": "logoCore",
          "required": true,
          "fit": "contain"
        },
        {
          "id": "orbit-back",
          "file": "assets/logo/logo-orbit-back.svg",
          "zIndex": 35,
          "role": "anel orbital traseiro",
          "animationGroup": "orbitBack",
          "required": false,
          "fit": "contain"
        },
        {
          "id": "orbit-front",
          "file": "assets/logo/logo-orbit-front.svg",
          "zIndex": 50,
          "role": "anel orbital frontal",
          "animationGroup": "orbitFront",
          "required": false,
          "fit": "contain"
        },
        {
          "id": "light-streak",
          "file": "assets/loading/light-streak.webp",
          "zIndex": 60,
          "role": "linha luminosa inferior",
          "animationGroup": "lightStreak",
          "required": false,
          "blendMode": "screen",
          "fit": "contain"
        },
        {
          "id": "text-loading",
          "file": "assets/loading/text-loading.svg",
          "zIndex": 70,
          "role": "texto CARREGANDO",
          "animationGroup": "loadingText",
          "required": true,
          "fit": "contain"
        }
      ]
    },
    "welcome": {
      "label": "Intro de boas-vindas",
      "fallback": "assets/welcome/static-welcome.webp",
      "layers": []
    }
  }
}
```

No pacote real, a cena `welcome` deve ser preenchida explicitamente, mesmo que reutilize várias camadas da cena `loading`.

---

## 6. Mapa de animação

Criar `animation-map.json` para separar regras de animação da implementação.

Exemplo conceitual:

```json
{
  "version": "1.0.0",
  "groups": {
    "background": {
      "type": "static"
    },
    "ambientGlow": {
      "type": "loop",
      "from": { "opacity": 0.45, "scale": 0.96 },
      "to": { "opacity": 0.8, "scale": 1.04 },
      "duration": 4.8,
      "repeat": -1,
      "yoyo": true,
      "ease": "sine.inOut"
    },
    "logoCore": {
      "type": "intro",
      "from": {
        "opacity": 0,
        "scale": 0.82,
        "rotation": -8,
        "filter": "blur(14px)"
      },
      "to": {
        "opacity": 1,
        "scale": 1,
        "rotation": 0,
        "filter": "blur(0px)"
      },
      "duration": 0.95,
      "ease": "power3.out"
    },
    "logoHalo": {
      "type": "intro-loop",
      "from": { "opacity": 0, "scale": 0.7 },
      "to": { "opacity": 0.9, "scale": 1 },
      "duration": 0.8,
      "loop": {
        "opacity": [0.65, 0.9, 0.65],
        "scale": [1, 1.025, 1],
        "duration": 4.2,
        "repeat": -1,
        "ease": "sine.inOut"
      }
    },
    "orbitBack": {
      "type": "loop",
      "from": { "opacity": 0, "rotation": -30 },
      "to": { "opacity": 0.45, "rotation": 0 },
      "duration": 1.1,
      "loop": {
        "rotation": 360,
        "duration": 28,
        "repeat": -1,
        "ease": "none"
      }
    },
    "orbitFront": {
      "type": "loop",
      "from": { "opacity": 0, "rotation": 35 },
      "to": { "opacity": 0.65, "rotation": 0 },
      "duration": 1.2,
      "loop": {
        "rotation": -360,
        "duration": 34,
        "repeat": -1,
        "ease": "none"
      }
    },
    "loadingText": {
      "type": "intro",
      "from": { "opacity": 0, "y": 12 },
      "to": { "opacity": 1, "y": 0 },
      "duration": 0.65,
      "delay": 0.75,
      "ease": "power2.out"
    },
    "welcomeText": {
      "type": "intro",
      "from": { "opacity": 0, "y": 18 },
      "to": { "opacity": 1, "y": 0 },
      "duration": 0.75,
      "delay": 0.9,
      "ease": "power2.out"
    }
  }
}
```

Esse arquivo pode ser simples no começo. O importante é que os nomes de `animationGroup` sejam estáveis.

---

## 7. Componentes React

Criar a seguinte estrutura:

```txt
src/components/portal-sama-intro/
  PortalSamaIntro.tsx
  PortalSamaSceneRenderer.tsx
  PortalSamaLayer.tsx
  PortalSamaStaticFallback.tsx
  portalSamaIntroTypes.ts
  portalSamaIntroAssets.ts
  portalSamaIntroAnimation.ts
  usePrefersReducedMotion.ts
```

---

## 8. Responsabilidades dos componentes

### 8.1. `PortalSamaIntro.tsx`

Responsável por coordenar a intro.

Props recomendadas:

```ts
type PortalSamaSceneName = "loading" | "welcome";
type PortalSamaMode = "auto" | "animated" | "static";

interface PortalSamaIntroProps {
  scene: PortalSamaSceneName;
  mode?: PortalSamaMode;
  minDurationMs?: number;
  fullScreen?: boolean;
  className?: string;
  ariaLabel?: string;
  onFinish?: () => void;
}
```

Responsabilidades:

- receber a cena;
- decidir modo de renderização;
- controlar tempo mínimo;
- chamar `onFinish`;
- montar/desmontar com Motion;
- não conhecer detalhes internos de cada camada;
- delegar a montagem para `PortalSamaSceneRenderer`.

### 8.2. `PortalSamaSceneRenderer.tsx`

Responsável por montar a cena visual.

Responsabilidades:

- carregar `manifest.json`;
- carregar `animation-map.json`;
- selecionar cena;
- ordenar layers por `zIndex`;
- renderizar cada layer;
- detectar falha de asset;
- decidir fallback;
- nunca renderizar fallback junto com camadas animadas.

### 8.3. `PortalSamaLayer.tsx`

Responsável por renderizar uma camada.

A camada deve ser renderizada como imagem:

```tsx
<motion.img />
```

Não deve importar SVG como componente React.

Não deve ter paths da logo em JSX.

Regras de estilo:

```css
position: absolute;
inset: 0;
width: 100%;
height: 100%;
pointer-events: none;
user-select: none;
```

Para cenas 16:9:

```css
object-fit: cover;
```

Para logo/elementos centrais:

```css
object-fit: contain;
```

### 8.4. `PortalSamaStaticFallback.tsx`

Responsável por renderizar fallback estático.

Usar quando:

- `mode="static"`;
- `prefers-reduced-motion` ativo e `mode="auto"`;
- manifesto falhar;
- animation map falhar;
- camada required falhar;
- erro inesperado na cena.

### 8.5. `usePrefersReducedMotion.ts`

Responsável por detectar:

```txt
(prefers-reduced-motion: reduce)
```

Quando ativo:

- evitar loops infinitos;
- evitar rotação contínua;
- evitar partículas flutuando;
- preferir fallback estático;
- no máximo usar fade simples.

---

## 9. Papel do Motion for React

Motion for React deve ser usado para controlar o wrapper e a presença da intro.

Exemplo conceitual:

```tsx
<AnimatePresence>
  {visible && (
    <motion.section
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      exit={{ opacity: 0 }}
      transition={{ duration: 0.45 }}
    >
      <PortalSamaSceneRenderer scene={scene} mode={mode} />
    </motion.section>
  )}
</AnimatePresence>
```

Motion não precisa controlar cada detalhe cinematográfico da logo. Isso será função do GSAP.

---

## 10. Papel do GSAP

GSAP deve controlar a timeline interna da cena.

O uso deve ser restrito ao escopo do componente para evitar vazamentos.

Exemplo conceitual:

```tsx
const root = useRef<HTMLDivElement>(null);

useGSAP(() => {
  const tl = gsap.timeline({ defaults: { ease: "power3.out" } });

  tl.from("[data-intro-layer='logo-core']", {
    opacity: 0,
    scale: 0.82,
    rotate: -8,
    filter: "blur(14px)",
    duration: 0.95,
  })
    .from("[data-intro-layer='logo-halo']", {
      opacity: 0,
      scale: 0.65,
      duration: 0.8,
    }, 0.1)
    .from("[data-intro-layer='orbit-front']", {
      opacity: 0,
      rotate: -60,
      duration: 1.2,
    }, 0.35)
    .from("[data-intro-layer='intro-text']", {
      opacity: 0,
      y: 18,
      duration: 0.7,
    }, 0.9);

  gsap.to("[data-intro-layer='logo-halo']", {
    opacity: 0.75,
    scale: 1.03,
    duration: 3.8,
    repeat: -1,
    yoyo: true,
    ease: "sine.inOut",
  });
}, { scope: root });
```

Em produção, a timeline deve respeitar `prefers-reduced-motion`.

Se reduced motion estiver ativo, não criar loops.

---

## 11. Sequência visual da intro loading

### 11.1. Estado inicial

```txt
fundo navy já visível
logo invisível
halo invisível
orbitais invisíveis
texto invisível
light streak invisível
```

### 11.2. Timeline sugerida

```txt
0.00s  fundo aparece com fade curto
0.10s  glow central aparece
0.20s  logo entra com escala, rotação leve e blur reduzindo
0.35s  halo aparece atrás da logo
0.45s  anel orbital traseiro surge
0.55s  anel orbital frontal surge
0.70s  light streak inferior aparece
0.85s  texto CARREGANDO entra
1.10s  partículas começam loop discreto
1.20s  halo entra em pulso lento
1.30s  orbitais começam rotação lenta
```

### 11.3. Duração mínima

Recomendação:

```txt
minDurationMs: 1200 a 1800
```

Se o sistema carregar rápido demais, a intro ainda deve ficar tempo suficiente para não parecer piscada.

---

## 12. Sequência visual da intro welcome

### 12.1. Estado inicial

```txt
fundo navy visível
ondas laterais com opacidade baixa
logo invisível ou semi-invisível
texto invisível
```

### 12.2. Timeline sugerida

```txt
0.00s  fundo aparece
0.10s  ondas laterais surgem suavemente
0.20s  glow central aparece
0.30s  logo entra
0.45s  halo pulsa
0.60s  orbitais surgem
0.80s  partículas aparecem
0.95s  texto Seja bem-vindo ao Portal Sama entra
1.30s  ondas passam a flutuar levemente
1.40s  glows estabilizam
```

### 12.3. Duração mínima

Recomendação:

```txt
minDurationMs: 1600 a 2400
```

A cena welcome pode ser mais lenta e institucional que a loading.

---

## 13. Fallback e modos

O componente deve aceitar três modos.

### 13.1. `mode="animated"`

Sempre tenta renderizar camadas animadas.

Se algo crítico falhar, cai para fallback.

### 13.2. `mode="static"`

Sempre renderiza imagem estática.

Usado para:

- debug;
- dispositivos fracos;
- reduced motion manual;
- páginas em que a animação não é necessária.

### 13.3. `mode="auto"`

Modo recomendado.

Regras:

```txt
se prefers-reduced-motion: fallback estático
se assets críticos falharem: fallback estático
se tudo carregar corretamente: camadas animadas
```

---

## 14. Tratamento de erro

Cada camada tem `required`.

### 14.1. Camada opcional falhou

Exemplo:

```txt
partículas
orbitais
glow secundário
onda secundária
```

Comportamento:

```txt
não quebrar intro
registrar warning em development
continuar renderização
```

### 14.2. Camada obrigatória falhou

Exemplo:

```txt
background
logo-core
texto principal
```

Comportamento:

```txt
renderizar fallback estático da cena
não exibir composição quebrada
registrar warning em development
```

---

## 15. Performance web

### 15.1. Formatos

Usar:

```txt
WebP para produção
PNG transparente como fonte/fallback
SVG para strokes simples
```

### 15.2. Resolução

Sugestão:

```txt
logo-core.webp: 1024x1024 ou 1536x1536 para uso normal
logo-core@2x.webp: 2048x2048 para telas retina, se necessário
background.webp: 1920x1080
fallbacks: 1920x1080
```

Evitar imagens maiores que o necessário.

### 15.3. Preload

Pré-carregar assets críticos:

```txt
background-deep-navy.webp
logo-core.webp
logo-halo-blue.webp
static-loading.webp
```

Pode ser feito com `<link rel="preload">` ou via carregamento antecipado no React.

### 15.4. Cache na VPS

Na VPS, configurar Nginx para assets versionados:

```txt
Cache-Control: public, max-age=31536000, immutable
```

Usar nomes versionados:

```txt
logo-core.v1.webp
logo-halo-blue.v1.webp
static-loading.v1.webp
```

Quando trocar assets, trocar a versão do nome.

---

## 16. Responsividade

### 16.1. Desktop

Usar composição cheia em 16:9.

```txt
width: 100vw
height: 100vh
object-fit: cover para fundo
object-fit: contain para logo e assets centrais
```

### 16.2. Mobile

Problema esperado: assets 16:9 podem cortar.

Solução:

- fundo pode usar `cover`;
- logo e texto devem usar `contain`;
- ondas laterais podem usar `cover` com crop controlado;
- manter a logo central sempre visível;
- evitar texto como imagem se precisar responsividade extrema.

### 16.3. Texto

Preferencialmente, texto deve ser HTML/CSS quando for dinâmico.

Usar SVG para texto apenas se a cena precisar ser idêntica ao layout visual fixo.

Para o Portal Sama, recomenda-se:

```txt
CARREGANDO -> pode ser HTML/CSS ou SVG simples
Seja bem-vindo ao Portal Sama -> preferencialmente HTML/CSS
```

Motivo: texto HTML permite alterar nome, mensagem, idioma e responsividade.

---

## 17. Acessibilidade

A intro é decorativa, exceto quando informa carregamento.

Regras:

- imagens decorativas com `alt=""`;
- wrapper com `aria-label` se for necessário indicar estado;
- não prender foco dentro da intro;
- não bloquear navegação por teclado;
- reduced motion respeitado;
- texto de carregamento real pode existir em HTML para leitores de tela.

Exemplo:

```tsx
<section
  role="status"
  aria-label="Carregando Portal Sama"
>
  <span className="sr-only">Carregando Portal Sama</span>
</section>
```

Para welcome:

```tsx
<section aria-label="Seja bem-vindo ao Portal Sama">
```

---

## 18. Exemplo de uso

### Loading

```tsx
<PortalSamaIntro
  scene="loading"
  mode="auto"
  minDurationMs={1400}
  fullScreen
/>
```

### Welcome

```tsx
<PortalSamaIntro
  scene="welcome"
  mode="auto"
  minDurationMs={1800}
  fullScreen
  onFinish={() => setIntroDone(true)}
/>
```

### Fallback estático forçado

```tsx
<PortalSamaIntro
  scene="loading"
  mode="static"
  fullScreen
/>
```

---

## 19. Fluxo de implementação

### Fase 1 — Preparar assets

1. Criar `logo-core.webp` premium com transparência.
2. Criar glows separados.
3. Criar halo separado.
4. Criar shadow separado.
5. Criar orbitais SVG simples.
6. Criar backgrounds e fallbacks.
7. Criar manifest.
8. Criar animation-map.

### Fase 2 — Implementar renderer

1. Criar tipos TypeScript.
2. Criar centralização de paths.
3. Criar loader de manifest.
4. Criar renderer de cena.
5. Criar layer renderer.
6. Criar fallback renderer.
7. Criar hook de reduced motion.

### Fase 3 — Integrar Motion

1. Envolver intro com `AnimatePresence`.
2. Criar entrada e saída do wrapper.
3. Garantir desmontagem limpa.

### Fase 4 — Integrar GSAP

1. Instalar GSAP e `@gsap/react` se necessário.
2. Criar timeline dentro do renderer.
3. Usar `data-intro-layer` em cada layer.
4. Escopar animação no root.
5. Desabilitar loops em reduced motion.
6. Limpar timeline no unmount.

### Fase 5 — Validar

1. Rodar lint.
2. Rodar typecheck.
3. Rodar build.
4. Testar desktop.
5. Testar mobile.
6. Testar reduced motion.
7. Testar falha de asset opcional.
8. Testar falha de asset obrigatório.
9. Testar VPS/produção.

---

## 20. Comportamento esperado em produção

Ao abrir o Portal Sama:

1. A aplicação começa a carregar.
2. A intro `loading` aparece.
3. Assets críticos são carregados.
4. A logo entra com movimento suave.
5. Halo e orbitais aparecem.
6. O texto `CARREGANDO` surge.
7. Quando o app estiver pronto e o tempo mínimo terminar, a intro some com fade.
8. Opcionalmente, a cena `welcome` aparece após login ou em primeiro acesso.
9. A tela principal do portal é liberada.

---

## 21. Critérios de aceite

A implementação será considerada correta quando:

- a logo não for redesenhada em React;
- não houver paths complexos da marca dentro do JSX;
- a composição usar assets externos aprovados;
- Motion controlar entrada/saída da tela;
- GSAP controlar a timeline interna;
- reduced motion for respeitado;
- fallback estático funcionar;
- falha de camada opcional não quebrar a intro;
- falha de camada obrigatória cair para fallback;
- build de produção passar;
- assets funcionarem em VPS;
- texto puder ser ajustado sem recriar a logo;
- visual ficar próximo das referências premium do Portal Sama.

---

## 22. O que não fazer

Não fazer:

```txt
logo hiper-detalhada inteira em SVG
SVG gigante com milhares de paths
logo desenhada em JSX
canvas
Lottie neste fluxo
Rive neste fluxo
texto fixo dentro da imagem quando precisa ser dinâmico
assets sem transparência real
checkerboard embutido no PNG
paths hardcoded no componente
animações infinitas para usuário com reduced motion
```

---

## 23. Resumo executivo

A melhor solução para o Portal Sama é híbrida.

A arte premium deve ser asset visual aprovado, preferencialmente WebP/PNG transparente. Os efeitos e camadas devem ser separados para permitir animação. SVG deve ser usado apenas onde ele é forte: linhas, órbitas, strokes e máscaras simples.

Motion for React deve cuidar da presença da intro na interface. GSAP deve cuidar da sequência cinematográfica interna.

A aplicação deve apenas montar, animar e controlar os assets. Ela não deve redesenhar a marca.
