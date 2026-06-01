# [CONCLUIDO] Portal Sama — Documentação para Codex implementar a intro web

## 1. Papel do Codex

O Codex será responsável exclusivamente pela implementação técnica da intro no projeto web.

O Codex não deve criar, redesenhar, vetorizar ou alterar a arte da logo. A arte visual será entregue separadamente em PNG/WebP/SVG simples. O papel do Codex é consumir esses assets, montar a composição por camadas, aplicar a timeline de animação e garantir funcionamento correto no navegador.

A implementação deve seguir a documentação principal da intro animada do Portal Sama e respeitar a arquitetura híbrida:

```txt
React
+ Motion for React
+ GSAP
+ assets PNG/WebP premium
+ SVGs simples para órbitas/linhas/máscaras
```

## 2. Objetivo técnico

Implementar uma intro web premium para o Portal Sama, com duas cenas principais:

```txt
loading  -> tela de carregamento com logo central, halo, órbita, brilho inferior e texto CARREGANDO
welcome  -> tela de boas-vindas com logo central, ondas laterais, partículas e texto Seja bem-vindo ao Portal Sama
```

A intro deve ser modular, performática, responsiva e segura para uso em produção em VPS.

## 3. Responsabilidades do Codex

O Codex deve implementar:

- componente React principal da intro;
- renderer de cenas baseado em manifesto;
- renderer de camadas;
- fallback estático;
- integração com Motion for React;
- integração com GSAP;
- suporte a `prefers-reduced-motion`;
- tratamento de erro por camada;
- tipagem TypeScript;
- organização de assets em `public`;
- documentação mínima de uso no README do projeto, se aplicável;
- validações de build, lint e typecheck.

O Codex não deve implementar:

- geração de imagens;
- vetorização da logo;
- criação de arte visual;
- edição manual de pixels;
- paths complexos da logo em JSX;
- canvas rendering;
- Lottie/Rive nesta fase;
- animações que dependam de vídeo renderizado.

## 4. Estrutura esperada de assets

O Codex deve assumir que os assets serão entregues nesta estrutura ou em estrutura equivalente:

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
            static-loading.webp

          welcome/
            orange-wave-left.webp
            blue-wave-right.webp
            orange-particles.webp
            blue-particles.webp
            static-welcome.webp
```

O texto pode ser renderizado em HTML/CSS, não precisa vir como imagem.

## 5. Estrutura de código recomendada

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
  portal-sama-intro.css ou module.css
```

Caso o projeto use Tailwind, o CSS pode ser reduzido, mas ainda é recomendável manter um arquivo ou módulo para classes específicas de composição e layers.

## 6. Componente principal

Criar `PortalSamaIntro.tsx`.

Interface esperada:

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

Comportamento esperado:

- `scene="loading"` mostra a intro de carregamento;
- `scene="welcome"` mostra a intro de boas-vindas;
- `mode="auto"` decide entre animação e fallback com base em reduced motion e disponibilidade de assets;
- `mode="animated"` força tentativa de animação;
- `mode="static"` usa imagem estática da cena;
- `minDurationMs` controla tempo mínimo antes de `onFinish`;
- `fullScreen` aplica `position: fixed; inset: 0;`;
- `onFinish` é chamado somente após tempo mínimo e fim da sequência principal.

## 7. Motion for React

Motion for React deve controlar montagem e desmontagem da intro.

Usar `AnimatePresence` para entrada e saída:

```tsx
<AnimatePresence>
  {visible && (
    <motion.section
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      exit={{ opacity: 0 }}
      transition={{ duration: 0.45, ease: "easeOut" }}
    >
      <PortalSamaSceneRenderer scene={scene} mode={mode} />
    </motion.section>
  )}
</AnimatePresence>
```

Motion deve ser usado para:

- fade in/out do wrapper;
- transições de interface;
- controle de presença;
- fallback simples quando GSAP estiver desativado.

Motion não deve ser usado para toda a timeline cinematográfica quando GSAP estiver ativo.

## 8. GSAP

GSAP deve controlar a timeline interna da cena.

Usar `@gsap/react` e `useGSAP` para garantir limpeza automática.

Regras:

- registrar plugin `useGSAP` uma única vez;
- usar `scope` com `rootRef`;
- usar seletores baseados em `data-intro-layer`;
- limpar timeline no unmount;
- não criar loops infinitos em reduced motion;
- não animar propriedades que forcem layout quando possível;
- preferir `transform`, `opacity` e `filter` com moderação.

Exemplo de seletores esperados:

```tsx
<img data-intro-layer="logo-core" />
<img data-intro-layer="logo-halo-blue" />
<img data-intro-layer="orbit-back" />
<img data-intro-layer="orbit-front" />
<div data-intro-layer="intro-text" />
```

## 9. Timeline da cena loading

Sequência recomendada:

```txt
0.00s  wrapper aparece via Motion
0.05s  background navy já visível
0.10s  center-blue-glow aparece
0.18s  logo-core entra com opacity, scale, rotate e blur
0.28s  logo-halo-blue aparece atrás da logo
0.38s  logo-glow-orange e logo-glow-blue aparecem
0.45s  orbit-back surge e inicia rotação lenta
0.55s  orbit-front surge e inicia rotação inversa lenta
0.70s  light-streak aparece abaixo da logo
0.85s  texto CARREGANDO aparece
1.10s  partículas entram de forma discreta
1.20s  halo inicia pulso lento
```

Valores sugeridos:

```txt
logo-core:
  opacity: 0 -> 1
  scale: 0.82 -> 1
  rotate: -8deg -> 0deg
  filter: blur(14px) -> blur(0px)
  duration: 0.95s
  ease: power3.out

halo:
  opacity: 0 -> 0.85
  scale: 0.65 -> 1
  duration: 0.8s

orbit-back:
  opacity: 0 -> 0.45
  rotation inicial: -30deg -> 0deg
  loop: 360deg em 28s

orbit-front:
  opacity: 0 -> 0.65
  rotation inicial: 35deg -> 0deg
  loop: -360deg em 34s

text-loading:
  opacity: 0 -> 1
  y: 12px -> 0
  duration: 0.65s
```

## 10. Timeline da cena welcome

Sequência recomendada:

```txt
0.00s  wrapper aparece via Motion
0.10s  ondas laterais aparecem em baixa opacidade
0.20s  center-blue-glow aparece
0.30s  logo-core entra
0.45s  halo pulsa
0.60s  orbitais surgem
0.80s  partículas aparecem
0.95s  texto Seja bem-vindo ao Portal Sama entra
1.30s  ondas laterais entram em flutuação lenta
1.40s  glows estabilizam
```

Texto recomendado em HTML/CSS:

```tsx
<h1 data-intro-layer="intro-text">
  Seja bem-vindo ao <span className="portal-word">Portal</span>{" "}
  <span className="sama-word">Sama</span>
</h1>
```

Cores sugeridas:

```txt
texto-base: #F4F7FF
Portal:     #0078FF
Sama:       #FF8A00
```

## 11. Renderer de cena

`PortalSamaSceneRenderer.tsx` deve:

1. receber `scene`;
2. buscar dados da cena no manifesto;
3. ordenar camadas por `zIndex`;
4. renderizar cada camada;
5. aplicar `data-intro-layer` com `id` ou `animationGroup`;
6. aplicar `mix-blend-mode` quando definido;
7. aplicar `opacity` inicial quando definido;
8. renderizar texto HTML específico da cena;
9. ativar GSAP se modo animado estiver disponível;
10. cair para fallback quando necessário.

## 12. Renderer de camada

`PortalSamaLayer.tsx` deve renderizar imagens de forma segura:

```tsx
<img
  src={src}
  alt=""
  draggable={false}
  data-intro-layer={layer.id}
  className="portal-sama-intro-layer"
  style={{
    zIndex: layer.zIndex,
    mixBlendMode: layer.blendMode,
    opacity: layer.opacity,
    objectFit: layer.fit === "cover" ? "cover" : "contain",
  }}
/>
```

Regras:

- `alt=""` para camadas decorativas;
- `pointer-events: none`;
- `user-select: none`;
- `position: absolute`;
- `inset: 0`;
- não usar background-image para camadas principais, pois dificulta controle e debug.

## 13. Fallback estático

Cada cena deve ter fallback:

```txt
assets/loading/static-loading.webp
assets/welcome/static-welcome.webp
```

Usar fallback quando:

- `mode="static"`;
- reduced motion ativo em `mode="auto"`;
- manifesto não carregar;
- animation-map não carregar;
- camada obrigatória falhar;
- erro inesperado em runtime.

O fallback deve ser uma imagem estática 16:9 já aprovada visualmente.

## 14. Reduced motion

Implementar `usePrefersReducedMotion`.

Quando `prefers-reduced-motion: reduce` estiver ativo:

- não iniciar loops de rotação;
- não iniciar partículas flutuantes;
- não usar blur animado forte;
- renderizar fallback estático em `mode="auto"`;
- permitir apenas fade simples se necessário.

## 15. Tratamento de erro

Cada camada no manifesto deve ter:

```ts
required: boolean;
```

Se camada opcional falhar:

```txt
continuar renderização
console.warn apenas em development
```

Se camada obrigatória falhar:

```txt
acionar fallback estático da cena
console.warn em development
```

Não exibir intro visualmente quebrada em produção.

## 16. CSS base

CSS conceitual:

```css
.portal-sama-intro {
  position: fixed;
  inset: 0;
  overflow: hidden;
  background: #020817;
  display: grid;
  place-items: center;
  isolation: isolate;
}

.portal-sama-intro-stage {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

.portal-sama-intro-layer {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  user-select: none;
  -webkit-user-drag: none;
}

.portal-sama-intro-text {
  position: absolute;
  left: 50%;
  bottom: 14%;
  transform: translateX(-50%);
  z-index: 100;
  color: #f4f7ff;
  font-weight: 400;
  letter-spacing: 0.01em;
  text-align: center;
  white-space: nowrap;
}

.portal-sama-intro-text .portal-word {
  color: #0078ff;
}

.portal-sama-intro-text .sama-word {
  color: #ff8a00;
}
```

Ajustar tamanhos por media query.

## 17. Performance em VPS

A VPS não executará a animação. A animação roda no navegador do usuário.

A VPS deve apenas servir assets de forma eficiente.

Recomendações:

- usar WebP para produção;
- usar nomes versionados nos assets;
- configurar cache longo no Nginx para assets versionados;
- comprimir JS/CSS;
- evitar imagens muito maiores que necessário;
- carregar assets críticos antes de iniciar a intro;
- usar fallback se asset crítico falhar.

Cache recomendado para assets versionados:

```txt
Cache-Control: public, max-age=31536000, immutable
```

## 18. Checklist de implementação

Antes de considerar finalizado, verificar:

```txt
[ ] componente PortalSamaIntro criado
[ ] renderer de cena criado
[ ] renderer de camada criado
[ ] fallback estático criado
[ ] Motion usado para entrada/saída
[ ] GSAP usado para timeline interna
[ ] useGSAP com scope implementado
[ ] reduced motion respeitado
[ ] assets obrigatórios tratados
[ ] assets opcionais não quebram a tela
[ ] texto welcome em HTML/CSS
[ ] texto loading acessível
[ ] responsivo desktop/mobile
[ ] build passa
[ ] typecheck passa
[ ] lint passa
[ ] funciona em produção/VPS
```

## 19. Critérios de aceite

A entrega do Codex será aprovada se:

- a intro reproduzir a estética das referências fornecidas;
- o código não contiver paths complexos da logo;
- a logo vier de asset externo;
- as camadas forem renderizadas por manifesto;
- `loading` e `welcome` funcionarem;
- `mode="static"` funcionar;
- `mode="auto"` respeitar reduced motion;
- GSAP não vazar animações após unmount;
- Motion controlar entrada e saída do wrapper;
- falhas de assets forem tratadas;
- a implementação for limpa, tipada e fácil de manter.

## 20. Proibição explícita

Não fazer:

```txt
não desenhar a logo em JSX
não transformar a logo inteira em SVG complexo dentro do componente
não usar canvas
não usar vídeo como primeira solução
não embutir base64 gigante no código
não depender de CDN externa para assets internos
não ignorar prefers-reduced-motion
não criar timeline global fora do escopo da intro
não misturar fallback estático com camadas animadas ao mesmo tempo
```

## 21. Resultado esperado

A implementação deve permitir que o projeto use:

```tsx
<PortalSamaIntro
  scene="loading"
  mode="auto"
  minDurationMs={1400}
  fullScreen
/>
```

E:

```tsx
<PortalSamaIntro
  scene="welcome"
  mode="auto"
  minDurationMs={1800}
  fullScreen
  onFinish={() => setIntroDone(true)}
/>
```

Com isso, o Codex entrega o motor técnico da intro, enquanto os assets visuais serão entregues separadamente.
