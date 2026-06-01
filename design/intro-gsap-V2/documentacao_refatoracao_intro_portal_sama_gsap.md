# [CONCLUIDO] Portal Sama — Refatoração completa da intro com GSAP, manifesto e segurança

## 1. Decisão técnica

A intro atual deve ser tratada como **legado visual**. Ela não deve ser evoluída visualmente em cima dos componentes antigos, porque a implementação atual está muito acoplada a SVGs internos, `framer-motion`, componentes específicos de camadas e assets antigos em `/public/brand/sama/intro/svg`.

A nova decisão é seguir pela **Opção B**:

```txt
React + TypeScript + GSAP + @gsap/react + manifest.json + animation-map.json + assets WebP/PNG/SVG sanitizados
```

O objetivo é criar uma intro nova, cinematográfica, segura e modular, preservando apenas as partes de integração que já funcionam no Portal Sama.

---

## 2. O que deve ser reaproveitado

Mesmo refazendo a intro, nem tudo deve ser apagado. A base de autenticação e preferência da intro já está correta e deve ser preservada.

### 2.1. Reaproveitar

```txt
portal-sama-web/src/features/intro/components/IntroGate.tsx
portal-sama-web/src/features/intro/hooks/useReducedMotionPreference.ts
portal-sama-web/src/features/intro/hooks/useIntroTiming.ts
portal-sama-web/src/features/intro/services/introPreferences.service.ts
portal-sama-web/src/features/intro/types.ts
portal-sama-web/src/components/layout/AppLayout.tsx
```

Esses arquivos devem ser refatorados, não removidos sem substituição.

### 2.2. Motivo

Eles já resolvem pontos importantes:

- bloqueio da intro em rotas públicas;
- verificação se o usuário está autenticado;
- distinção entre usuário novo e usuário recorrente;
- uso de `welcomeAnimationSeen`;
- chamada segura para marcar intro como vista;
- integração com CSRF;
- integração com Zustand/AuthStore;
- integração com `PATCH /api-v2/me/preferences/intro`.

---

## 3. O que deve ser removido ou substituído

A camada visual atual deve ser substituída pela nova engine GSAP.

### 3.1. Remover ou aposentar

```txt
portal-sama-web/src/features/intro/components/MotionBackground.tsx
portal-sama-web/src/features/intro/components/MotionLogo.tsx
portal-sama-web/src/features/intro/components/MotionOrbitLines.tsx
portal-sama-web/src/features/intro/components/MotionParticles.tsx
portal-sama-web/src/features/intro/components/SamaLogoLayered.tsx
portal-sama-web/src/features/intro/components/LoadingCue.tsx
portal-sama-web/src/features/intro/components/StandardIntro.tsx
portal-sama-web/src/features/intro/components/WelcomeIntro.tsx
portal-sama-web/src/features/intro/components/PortalRevealTransition.tsx
portal-sama-web/src/features/intro/motion-tokens.ts
portal-sama-web/src/features/intro/styles/intro-motion.css
```

Esses arquivos foram feitos para a intro visual anterior. Como a intro será refeita com manifesto e GSAP, eles não devem continuar controlando a experiência visual.

### 3.2. Remover assets antigos da intro

```txt
portal-sama-web/public/brand/sama/intro/svg/
portal-sama-web/dist/brand/sama/intro/svg/
```

A pasta `dist` é saída de build e não deve ser editada manualmente. O foco deve ser `public`.

A pasta antiga `/public/brand/sama/intro/svg/` deve ser removida quando a nova intro estiver pronta e validada.

---

## 4. Estrutura nova recomendada

A nova intro deve continuar dentro de `src/features/intro`, pois essa já é a feature oficial no projeto.

Não criar uma nova pasta solta em `src/components/portal-sama-intro`, para evitar duplicidade arquitetural.

### 4.1. Estrutura de código

```txt
portal-sama-web/src/features/intro/
  components/
    IntroGate.tsx
    PortalSamaIntro.tsx
    PortalSamaSceneRenderer.tsx
    PortalSamaLayer.tsx
    PortalSamaStaticFallback.tsx
    PortalSamaSkipButton.tsx
    PortalReveal.tsx
    IntroAssetPreview.tsx

  hooks/
    useIntroTiming.ts
    useReducedMotionPreference.ts
    useIntroAssetHealth.ts
    usePortalSamaGsapTimeline.ts

  services/
    introPreferences.service.ts

  styles/
    portal-sama-intro.css

  animation-map.ts
  intro-assets.ts
  intro-security.ts
  types.ts
  index.ts
```

### 4.2. Estrutura de assets públicos

```txt
portal-sama-web/public/brand/sama/intro/
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

---

## 5. Dependências

A intro nova deve usar GSAP oficialmente.

### 5.1. Instalar

```bash
npm install gsap @gsap/react
```

### 5.2. Remover Framer Motion?

Como o uso de `framer-motion` no projeto atual está concentrado na intro, existem duas opções.

#### Opção recomendada

Remover `framer-motion` após a migração completa:

```bash
npm uninstall framer-motion
```

Essa opção deixa o bundle mais limpo e evita duas engines de animação competindo pela mesma feature.

#### Opção transitória

Manter `framer-motion` temporariamente durante a refatoração, mas a nova intro não deve depender dele.

---

## 6. Tipagem correta

A tipagem atual deve ser atualizada para separar cena, modo e versão.

```ts
export type PortalSamaIntroScene = 'loading' | 'welcome'

export type PortalSamaIntroRenderMode = 'auto' | 'animated' | 'static'

export type PortalSamaIntroAssetFit = 'cover' | 'contain'

export type PortalSamaIntroLayerType = 'image' | 'html-text'

export type PortalSamaIntroAnimationGroup =
  | 'background'
  | 'ambientPulse'
  | 'particlesDrift'
  | 'logoFloat'
  | 'logoPulse'
  | 'orbitSlow'
  | 'orbitReverse'
  | 'loadingStreak'
  | 'loadingText'
  | 'welcomeText'
  | 'waveInLeft'
  | 'waveInRight'

export type PortalSamaIntroLayer = {
  type?: PortalSamaIntroLayerType
  path?: string
  content?: string
  zIndex: number
  fit?: PortalSamaIntroAssetFit
  opacity?: number
  blendMode?: React.CSSProperties['mixBlendMode']
  required?: boolean
  animationGroup: PortalSamaIntroAnimationGroup
}

export type PortalSamaIntroManifest = {
  name: string
  version: string
  assetBasePath: string
  scenes: Record<PortalSamaIntroScene, PortalSamaIntroLayer[]>
  fallbacks: Record<PortalSamaIntroScene, string>
}

export const INTRO_ANIMATION_VERSION = 'sama-gsap-layers-v2'
```

---

## 7. `animation-map.json`

Como a decisão foi pela Opção B, deve existir um arquivo `animation-map.json` real, separado do manifesto visual.

O `manifest.json` responde:

```txt
Quais camadas existem?
Em qual ordem aparecem?
Qual opacidade base?
Qual blend mode?
Qual camada é obrigatória?
```

O `animation-map.json` responde:

```txt
Como cada grupo anima?
Qual duração?
Qual ease?
Qual repeat?
Qual delay?
Qual transform?
```

### 7.1. Exemplo inicial

```json
{
  "version": "sama-gsap-layers-v2",
  "engine": "gsap",
  "defaults": {
    "ease": "power3.out",
    "duration": 0.8
  },
  "groups": {
    "ambientPulse": {
      "from": { "opacity": 0.35, "scale": 0.98 },
      "to": {
        "opacity": 0.85,
        "scale": 1.03,
        "duration": 4.8,
        "ease": "sine.inOut",
        "repeat": -1,
        "yoyo": true
      }
    },
    "logoFloat": {
      "from": { "opacity": 0, "scale": 0.82, "rotation": -8, "filter": "blur(14px)" },
      "to": { "opacity": 1, "scale": 1, "rotation": 0, "filter": "blur(0px)", "duration": 0.95, "ease": "power3.out" },
      "loop": {
        "y": 6,
        "scale": 1.005,
        "duration": 4.2,
        "ease": "sine.inOut",
        "repeat": -1,
        "yoyo": true
      }
    },
    "logoPulse": {
      "from": { "opacity": 0, "scale": 0.72 },
      "to": { "opacity": 1, "scale": 1, "duration": 0.8, "ease": "power2.out" },
      "loop": {
        "opacity": 0.72,
        "scale": 1.04,
        "duration": 3.6,
        "ease": "sine.inOut",
        "repeat": -1,
        "yoyo": true
      }
    },
    "orbitSlow": {
      "from": { "opacity": 0, "rotation": -30 },
      "to": { "opacity": 1, "rotation": 0, "duration": 0.7 },
      "loop": { "rotation": 360, "duration": 28, "ease": "none", "repeat": -1 }
    },
    "orbitReverse": {
      "from": { "opacity": 0, "rotation": 35 },
      "to": { "opacity": 1, "rotation": 0, "duration": 0.7 },
      "loop": { "rotation": -360, "duration": 34, "ease": "none", "repeat": -1 }
    },
    "particlesDrift": {
      "from": { "opacity": 0, "x": 0, "y": 0 },
      "to": { "opacity": 1, "duration": 1.1 },
      "loop": { "x": 8, "y": -6, "duration": 9, "ease": "sine.inOut", "repeat": -1, "yoyo": true }
    },
    "loadingStreak": {
      "from": { "opacity": 0, "x": -40 },
      "to": { "opacity": 1, "x": 0, "duration": 0.75, "ease": "power2.out" },
      "loop": { "x": 28, "opacity": 0.72, "duration": 2.8, "ease": "sine.inOut", "repeat": -1, "yoyo": true }
    },
    "loadingText": {
      "from": { "opacity": 0, "y": 12, "filter": "blur(6px)" },
      "to": { "opacity": 1, "y": 0, "filter": "blur(0px)", "duration": 0.65, "ease": "power2.out" }
    },
    "welcomeText": {
      "from": { "opacity": 0, "y": 24, "filter": "blur(10px)" },
      "to": { "opacity": 1, "y": 0, "filter": "blur(0px)", "duration": 0.9, "ease": "power3.out" }
    },
    "waveInLeft": {
      "from": { "opacity": 0, "x": -42 },
      "to": { "opacity": 1, "x": 0, "duration": 1.1, "ease": "power2.out" }
    },
    "waveInRight": {
      "from": { "opacity": 0, "x": 42 },
      "to": { "opacity": 1, "x": 0, "duration": 1.1, "ease": "power2.out" }
    }
  },
  "scenes": {
    "loading": {
      "durationMs": 2200,
      "sequence": [
        ["background", 0],
        ["ambientPulse", 0.05],
        ["logoFloat", 0.18],
        ["logoPulse", 0.28],
        ["orbitSlow", 0.45],
        ["orbitReverse", 0.55],
        ["loadingStreak", 0.7],
        ["loadingText", 0.85],
        ["particlesDrift", 1.1]
      ]
    },
    "welcome": {
      "durationMs": 4600,
      "sequence": [
        ["background", 0],
        ["ambientPulse", 0.05],
        ["waveInLeft", 0.18],
        ["waveInRight", 0.24],
        ["particlesDrift", 0.4],
        ["logoFloat", 0.55],
        ["logoPulse", 0.7],
        ["orbitSlow", 0.85],
        ["orbitReverse", 0.95],
        ["welcomeText", 1.65]
      ]
    }
  }
}
```

---

## 8. Novo fluxo de execução

### 8.1. Carregamento de sessão

Enquanto o Portal Sama valida sessão:

```tsx
<PortalSamaIntro scene="loading" mode="auto" />
```

Esse comportamento substitui o `StandardIntro` antigo usado em `AppLayout`.

### 8.2. Usuário autenticado recorrente

Usuário autenticado e com `welcomeAnimationSeen === true` pode ver somente a intro curta de carregamento, ou não ver intro nenhuma dependendo da decisão de produto.

Recomendação:

```txt
Usuário recorrente: intro loading curta.
Usuário novo: intro welcome completa.
```

### 8.3. Usuário novo

Se `user.welcomeAnimationSeen === false`:

```tsx
<PortalSamaIntro scene="welcome" mode="auto" />
```

Ao concluir, chamar:

```ts
markWelcomeIntroSeen()
```

Isso mantém a segurança atual: JWT + CSRF + auditoria no backend.

---

## 9. Segurança obrigatória

Como o Portal Sama lida com documentos empresariais, a intro não pode introduzir risco no front-end.

### 9.1. Assets somente locais

Todos os assets devem ser servidos do próprio domínio:

```txt
/brand/sama/intro/
```

Proibido:

```txt
CDN externa
URLs remotas em manifesto
base64 gigante em JSON
scripts inline vindos de assets
Lottie remoto
imagens carregadas de domínio de terceiros
```

### 9.2. Validação de caminho de asset

O renderer deve aceitar somente paths relativos e internos.

Proibido aceitar:

```txt
https://...
http://...
//cdn...
data:
javascript:
../
..\\
```

Função recomendada:

```ts
export function assertSafeIntroAssetPath(path: string) {
  const blocked = ['http:', 'https:', '//', 'data:', 'javascript:', '../', '..\\']

  if (blocked.some((token) => path.includes(token))) {
    throw new Error(`Intro asset path inseguro: ${path}`)
  }

  if (!/^[a-zA-Z0-9/_@.-]+$/.test(path)) {
    throw new Error(`Intro asset path invalido: ${path}`)
  }

  return path
}
```

### 9.3. Sanitização de SVG

Os arquivos SVG permitidos são apenas orbitais simples. Antes de commitar qualquer SVG, validar ausência de:

```txt
<script>
<foreignObject>
onload=
onerror=
onclick=
href="http
xlink:href="http
data:
javascript:
```

### 9.4. CSP recomendada

No Nginx ou camada de edge, aplicar CSP compatível com Vite/React em produção.

Base recomendada:

```nginx
add_header Content-Security-Policy "default-src 'self'; script-src 'self'; img-src 'self'; style-src 'self' 'unsafe-inline'; font-src 'self'; connect-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'; form-action 'self'; upgrade-insecure-requests" always;
```

Observação: `style-src 'unsafe-inline'` pode ser necessário por causa de estilos inline dinâmicos de React/GSAP. A meta futura deve ser reduzir esse uso, mas não bloquear a entrega inicial da intro.

### 9.5. A intro nunca deve bloquear acesso ao sistema

Se o manifesto falhar, se algum asset obrigatório falhar, ou se GSAP não inicializar, a aplicação deve seguir para o portal.

A regra é:

```txt
Falha visual da intro não pode virar falha de autenticação.
```

### 9.6. Rotas públicas sem intro autenticada

A intro autenticada deve continuar fora de:

```txt
/login
/assinatura/:token
/onboarding/publico/documentos/:token
/onboarding/publico/proposta/:token
```

Isso evita interferência em rotas públicas de assinatura, proposta e envio documental.

### 9.7. Não persistir estado sensível da intro no localStorage

Não salvar tokens, usuário, permissões ou estado sensível da sessão por causa da intro.

Permitido apenas estado visual não sensível, se necessário:

```txt
intro_render_mode
intro_debug_disabled
```

Mesmo assim, preferir não usar `localStorage` na primeira versão.

---

## 10. Performance

### 10.1. Preload mínimo

Para `loading`, precarregar somente:

```txt
/brand/sama/intro/assets/shared/background-deep-navy.webp
/brand/sama/intro/assets/logo/logo-core.webp
/brand/sama/intro/assets/loading/static-loading.webp
```

Para `welcome`, carregar os assets restantes sob demanda.

### 10.2. Fallback automático

Usar fallback estático quando:

```txt
prefers-reduced-motion: reduce
asset obrigatório falhar
modo static for forçado
navigator.hardwareConcurrency <= 2
conexão muito limitada, se navigator.connection estiver disponível
```

### 10.3. Propriedades animadas permitidas

Preferir:

```txt
transform
opacity
filter com moderação
```

Evitar animar:

```txt
width
height
top
left
box-shadow pesado
background-position em camadas grandes
```

---

## 11. Componente principal esperado

```tsx
import { useEffect, useRef, useState } from 'react'
import { gsap } from 'gsap'
import { useGSAP } from '@gsap/react'
import type { PortalSamaIntroRenderMode, PortalSamaIntroScene } from '../types'
import { PortalSamaSceneRenderer } from './PortalSamaSceneRenderer'
import { PortalSamaStaticFallback } from './PortalSamaStaticFallback'

gsap.registerPlugin(useGSAP)

type PortalSamaIntroProps = {
  scene: PortalSamaIntroScene
  mode?: PortalSamaIntroRenderMode
  durationMs?: number
  reducedMotion?: boolean
  canSkip?: boolean
  onComplete: () => void
}

export function PortalSamaIntro({
  scene,
  mode = 'auto',
  durationMs,
  reducedMotion = false,
  canSkip = false,
  onComplete,
}: PortalSamaIntroProps) {
  const rootRef = useRef<HTMLElement | null>(null)
  const [assetFailed, setAssetFailed] = useState(false)

  const shouldUseStatic = mode === 'static' || reducedMotion || assetFailed

  useEffect(() => {
    const timeout = window.setTimeout(onComplete, durationMs ?? (scene === 'welcome' ? 4600 : 2200))
    return () => window.clearTimeout(timeout)
  }, [durationMs, onComplete, scene])

  useGSAP(
    () => {
      if (shouldUseStatic) return

      const ctx = gsap.context(() => {
        const tl = gsap.timeline({ defaults: { ease: 'power3.out' } })

        tl.fromTo('[data-intro-layer]', { opacity: 0 }, { opacity: 1, duration: 0.35, stagger: 0.035 })

        gsap.to('[data-intro-group="ambientPulse"]', {
          scale: 1.03,
          opacity: 0.85,
          duration: 4.8,
          ease: 'sine.inOut',
          repeat: -1,
          yoyo: true,
        })

        gsap.to('[data-intro-group="orbitSlow"]', {
          rotation: 360,
          duration: 28,
          ease: 'none',
          repeat: -1,
        })

        gsap.to('[data-intro-group="orbitReverse"]', {
          rotation: -360,
          duration: 34,
          ease: 'none',
          repeat: -1,
        })
      }, rootRef)

      return () => ctx.revert()
    },
    { scope: rootRef, dependencies: [scene, shouldUseStatic] },
  )

  return (
    <section ref={rootRef} className="portal-sama-intro" aria-label={scene === 'welcome' ? 'Seja bem-vindo ao Portal Sama' : 'Carregando Portal Sama'}>
      {shouldUseStatic ? (
        <PortalSamaStaticFallback scene={scene} />
      ) : (
        <PortalSamaSceneRenderer scene={scene} onRequiredAssetError={() => setAssetFailed(true)} />
      )}

      {canSkip ? (
        <button type="button" className="portal-sama-intro__skip" onClick={onComplete}>
          Pular introdução
        </button>
      ) : null}
    </section>
  )
}
```

Esse exemplo é uma base. A implementação final deve usar `manifest.json` e `animation-map.json`, não seletores hardcoded espalhados.

---

## 12. Refatoração do `IntroGate`

O `IntroGate` deve continuar decidindo quando a intro aparece.

Novo comportamento esperado:

```txt
status !== authenticated -> não renderizar intro autenticada
rota pública -> não renderizar intro autenticada
user.welcomeAnimationSeen === false -> scene welcome
caso contrário -> scene loading ou nenhuma intro, conforme regra de produto
```

Recomendação inicial:

```tsx
{!introDone && (
  <PortalSamaIntro
    scene={isWelcomeIntro ? 'welcome' : 'loading'}
    mode="auto"
    reducedMotion={prefersReducedMotion}
    durationMs={isWelcomeIntro ? timing.welcomeDurationMs : timing.standardDurationMs}
    canSkip={isWelcomeIntro && canSkip}
    onComplete={async () => {
      if (isWelcomeIntro) {
        await markWelcomeIntroSeen()
      }
      completeIntro()
    }}
  />
)}
```

---

## 13. Atualização da versão da intro

A versão atual deve sair de:

```ts
export const INTRO_ANIMATION_VERSION = 'sama-svg-layers-v1'
```

Para:

```ts
export const INTRO_ANIMATION_VERSION = 'sama-gsap-layers-v2'
```

Isso permite que o backend registre qual versão da intro o usuário viu.

---

## 14. Ajustes no preview de desenvolvimento

A rota atual:

```txt
/dev/intro-preview
```

Deve continuar existindo somente em desenvolvimento.

Ela deve permitir testar:

```txt
scene=loading
scene=welcome
mode=animated
mode=static
reduced motion simulado
falha de asset obrigatório
```

Essa rota não deve existir em produção.

---

## 15. Critérios de aceite

A implementação só deve ser considerada pronta quando cumprir todos os pontos abaixo.

### 15.1. Visual

- Loading lembra a referência `Logo-Sama.png`.
- Welcome lembra a referência `Boas-vindas.png`.
- Logo premium aparece a partir de `logo-core.webp`, não de SVG reconstruído em JSX.
- Texto animado é HTML/CSS.
- Fallbacks estáticos funcionam.
- Não há checkerboard, fundo quebrado ou asset ausente visível.

### 15.2. Arquitetura

- A intro usa `manifest.json` para camadas.
- A intro usa `animation-map.json` para animações.
- GSAP está encapsulado na feature da intro.
- `@gsap/react` limpa timelines no unmount.
- Não há timeline global fora do escopo da intro.
- Não há dependência visual antiga da pasta `/svg`.

### 15.3. Segurança

- Paths de assets são validados.
- Não há asset remoto.
- SVGs são sanitizados.
- CSP está documentada.
- Intro não bloqueia autenticação nem navegação.
- Rotas públicas continuam fora da intro autenticada.
- `PATCH /me/preferences/intro` continua protegido por JWT + CSRF.

### 15.4. Performance

- Build passa.
- Typecheck passa.
- Lint passa.
- Não há carregamento antecipado desnecessário de todos os assets.
- Reduced motion usa fallback estático.
- Dispositivo fraco consegue acessar o portal sem travar.

---

## 16. Ordem recomendada de implementação para o Codex

1. Instalar `gsap` e `@gsap/react`.
2. Copiar assets novos para `/public/brand/sama/intro/assets`.
3. Criar `/public/brand/sama/intro/manifest.json` corrigido.
4. Criar `/public/brand/sama/intro/animation-map.json`.
5. Criar tipos novos em `src/features/intro/types.ts`.
6. Criar `intro-security.ts` para validação de paths.
7. Criar `PortalSamaLayer.tsx`.
8. Criar `PortalSamaSceneRenderer.tsx`.
9. Criar `PortalSamaStaticFallback.tsx`.
10. Criar `usePortalSamaGsapTimeline.ts`.
11. Criar `PortalSamaIntro.tsx`.
12. Refatorar `IntroGate.tsx`.
13. Refatorar `AppLayout.tsx` para usar a nova intro loading durante bootstrap.
14. Atualizar `IntroAssetPreview.tsx`.
15. Remover componentes visuais antigos.
16. Remover assets SVG antigos.
17. Rodar `npm run lint`.
18. Rodar `npm run build`.
19. Testar login, refresh, welcome intro, skip e rotas públicas.
20. Registrar evidências em `.ai-tests`.

---

## 17. Prompt recomendado para o Codex

```md
Você está trabalhando no projeto Portal Sama. Refaça completamente a intro atual do `portal-sama-web` usando GSAP, manifesto de assets e mapa de animações.

Contexto:
- A intro visual atual em `src/features/intro` está ruim e deve ser substituída.
- Reaproveite somente a integração de sessão, `IntroGate`, hooks úteis, serviço `markWelcomeIntroSeen`, estado `welcomeAnimationSeen`, endpoint `PATCH /api-v2/me/preferences/intro`, CSRF e auditoria.
- Remova/substitua a camada visual antiga baseada em `framer-motion` e assets SVG antigos.
- Use `gsap` e `@gsap/react` como engine oficial da nova intro.
- Use os assets aprovados em `/public/brand/sama/intro/assets`.
- Crie e use `manifest.json` para ordem/camadas e `animation-map.json` para timelines.
- A intro deve ter as cenas `loading` e `welcome`.
- A intro deve ter modo `auto`, `animated` e `static`.
- A intro deve respeitar `prefers-reduced-motion`.
- A intro deve usar fallback estático se asset obrigatório falhar.
- A intro não pode bloquear autenticação, sessão ou navegação.
- A intro não deve aparecer em rotas públicas como `/login`, `/assinatura/:token`, `/onboarding/publico/documentos/:token` e `/onboarding/publico/proposta/:token`.

Regras de segurança obrigatórias:
- Não usar CDN externa.
- Não aceitar asset remoto no manifesto.
- Validar paths de assets contra `http:`, `https:`, `//`, `data:`, `javascript:`, `../` e caracteres inválidos.
- Sanitizar SVGs e proibir `<script>`, `<foreignObject>`, handlers `on*`, `href` externo e `javascript:`.
- Manter `PATCH /me/preferences/intro` protegido por JWT e CSRF.
- Não salvar dados sensíveis da sessão em `localStorage`.
- A falha da intro deve cair para fallback ou concluir a intro, nunca travar o portal.

Arquitetura desejada:
- Manter a feature em `src/features/intro`.
- Criar `PortalSamaIntro.tsx`, `PortalSamaSceneRenderer.tsx`, `PortalSamaLayer.tsx`, `PortalSamaStaticFallback.tsx`, `PortalReveal.tsx`, `usePortalSamaGsapTimeline.ts`, `intro-security.ts`, `intro-assets.ts`, `animation-map.ts` e `portal-sama-intro.css`.
- Atualizar `types.ts` com `PortalSamaIntroScene`, `PortalSamaIntroRenderMode`, `PortalSamaIntroLayer`, `PortalSamaIntroManifest` e `INTRO_ANIMATION_VERSION = 'sama-gsap-layers-v2'`.
- Atualizar `IntroGate.tsx` para usar `PortalSamaIntro`.
- Atualizar `AppLayout.tsx` para usar a nova cena `loading` durante bootstrap.
- Atualizar `/dev/intro-preview` para testar loading, welcome, static, animated, reduced motion e falha de asset.

Ao finalizar:
- Remova arquivos visuais antigos não usados.
- Remova `/public/brand/sama/intro/svg` se não houver mais dependência.
- Rode `npm run lint` e `npm run build`.
- Documente evidências e decisões em `.ai-tests`.
```
