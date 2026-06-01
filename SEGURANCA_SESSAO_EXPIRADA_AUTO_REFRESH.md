# [CONCLUIDO] Documentação técnica — Sessão expirada mesmo com o usuário usando o Portal Sama

## Status da correcao - 2026-06-01

Correcao implementada no front-end em `portal-sama-web/src/services/api.ts`.

- O interceptor de request agora faz auto-refresh antes de chamadas protegidas quando o access token nao esta fresco.
- O interceptor de response trata `401` com retry unico da requisicao original.
- O refresh usa promessa compartilhada (`refreshPromise`) para evitar chamadas paralelas durante a rotacao do refresh token.
- Endpoints de autenticacao sao ignorados pelo auto-refresh para evitar loop.
- A sessao local so e limpa quando o refresh tambem falha.
- `withCredentials` e envio de CSRF foram preservados.

Validacao local:

- `npm.cmd run build` passou no Web.
- `npm.cmd run lint` passou no Web.

## 1. Objetivo

Esta documentação explica de forma detalhada o problema em que o Portal Sama informa que a sessão foi expirada mesmo quando o usuário ainda está utilizando a aplicação.

O objetivo é documentar:

- o comportamento observado;
- a causa real identificada no código;
- os arquivos envolvidos;
- o fluxo atual de autenticação;
- o motivo técnico da expiração indevida;
- a solução recomendada;
- os cuidados de segurança que devem ser mantidos;
- um checklist de implementação e validação.

---

## 2. Resumo executivo

O problema ocorre porque o **access token** do usuário expira em um tempo curto, atualmente configurado para aproximadamente **15 minutos**, mas o front-end não renova esse token automaticamente durante o uso normal da aplicação.

O backend já possui suporte para renovação de sessão por meio do endpoint:

```txt
POST /auth/refresh
```

Porém, no front-end, a função de renovação existe, mas só é usada no carregamento inicial da aplicação, por meio do `bootstrapSession()`.

Na prática, o fluxo atual funciona assim:

```txt
Usuário faz login
↓
Front-end recebe access token válido por 15 minutos
↓
Usuário continua na mesma tela
↓
O token vence
↓
Front-end continua enviando o token vencido
↓
Backend rejeita a requisição
↓
Portal exibe erro de sessão expirada
```

Portanto, o erro não está no usuário ficar parado. O erro está na ausência de uma estratégia de **auto refresh** e **retry automático de requisições 401** no front-end.

---

## 3. Sintoma observado

O usuário consegue autenticar normalmente e usar o sistema. Depois de algum tempo na aplicação, mesmo interagindo com a tela, uma nova chamada autenticada pode falhar com a mensagem:

```txt
Sessao expirada ou token invalido.
```

Esse comportamento aparece principalmente quando:

- o usuário fica muito tempo em uma mesma página;
- o usuário usa partes da tela que não fazem chamadas frequentes para a API;
- o usuário volta a executar uma ação autenticada depois de alguns minutos;
- o token de acesso já venceu, mas a aplicação ainda mantém o estado visual como autenticado.

O usuário percebe o problema como se o sistema tivesse derrubado a sessão sem motivo, mesmo durante uso ativo.

---

## 4. Arquivos analisados

### Front-end

Foram identificados como relevantes:

```txt
portal-sama-web/src/services/api.ts
portal-sama-web/src/services/auth.service.ts
portal-sama-web/src/stores/auth.store.ts
portal-sama-web/src/components/layout/AppLayout.tsx
portal-sama-web/src/app/providers.tsx
```

### Backend/API

No pacote atual da API, o código-fonte TypeScript não veio em `src`, mas o build em `dist` permite confirmar o comportamento da autenticação.

Arquivos relevantes:

```txt
portal-sama-api/.env.example
portal-sama-api/dist/src/config/env.schema.js
portal-sama-api/dist/src/modules/auth/auth.controller.js
portal-sama-api/dist/src/modules/auth/auth.service.js
portal-sama-api/dist/src/modules/auth/guards/jwt-auth.guard.js
portal-sama-api/dist/src/modules/auth/csrf.service.js
```

---

## 5. Funcionamento atual da autenticação

## 5.1. Backend emite dois tipos de token

O backend trabalha com dois conceitos principais:

### Access token

É o token enviado no header:

```txt
Authorization: Bearer <access_token>
```

Ele é usado para autenticar as requisições protegidas.

No `.env.example`, a validade está definida como:

```env
JWT_ACCESS_EXPIRES_IN=15m
```

Esse token é curto, o que é positivo para segurança.

### Refresh token

O refresh token é enviado em cookie HTTP-only:

```env
COOKIE_NAME=sama_refresh_token
JWT_REFRESH_EXPIRES_IN=7d
```

Esse token serve para renovar a sessão sem exigir novo login.

O backend rotaciona o refresh token a cada renovação. Ou seja: quando `/auth/refresh` é chamado com sucesso, o refresh token antigo é revogado e um novo refresh token é salvo.

Esse desenho é bom para segurança, porque reduz o impacto de vazamento de token antigo.

---

## 5.2. Login

No login, o backend cria uma sessão com:

- `accessToken`;
- `expiresIn`;
- dados do usuário;
- permissões;
- refresh token em cookie HTTP-only;
- token CSRF.

No front-end, o login chama:

```ts
export async function login(input: LoginInput) {
  await issueCsrfToken()
  const { data } = await api.post<AuthSessionResponse>('/auth/login', input, {
    headers: { 'Content-Type': 'application/json' },
  })
  useAuthStore.getState().setSession(data, 'login')
  return data
}
```

Depois disso, o estado de autenticação é salvo no Zustand.

---

## 5.3. Estado de autenticação no front-end

O store de autenticação salva o tempo de expiração do token assim:

```ts
expiresAt: Date.now() + session.expiresIn * 1000
```

E considera o token válido se ainda faltar mais de 30 segundos para expirar:

```ts
isTokenFresh: () => {
  const { accessToken, expiresAt } = get()
  return Boolean(accessToken && expiresAt > Date.now() + 30_000)
}
```

Esse cálculo está correto. O problema não está nesse trecho.

O problema é que essa informação é pouco usada para renovar o token durante o funcionamento normal da aplicação.

---

## 5.4. Bootstrap inicial da sessão

No `auth.service.ts`, existe a função:

```ts
export async function bootstrapSession() {
  const store = useAuthStore.getState()
  if (store.status === 'checking') return store.user

  store.setChecking()

  try {
    if (store.isTokenFresh()) {
      const data = await loadCurrentUser()
      return data.user
    }

    const session = await refreshSession()
    return session.user
  } catch {
    useAuthStore.getState().clearSession()
    return null
  }
}
```

Essa função tenta reaproveitar a sessão no carregamento inicial.

Se o token ainda estiver fresco, ela chama `/auth/me`.

Se o token não estiver fresco, ela chama `/auth/refresh`.

O problema é que essa função só roda no início do layout.

No `AppLayout.tsx`:

```tsx
useEffect(() => {
  if (status === 'idle') {
    bootstrapSession().catch(() => undefined)
  }
}, [status])
```

Ou seja, depois que o status vira `authenticated`, esse efeito não fica renovando a sessão.

---

## 5.5. Interceptor HTTP atual

O arquivo `api.ts` possui um interceptor de request:

```ts
api.interceptors.request.use((config) => {
  const { accessToken, csrfToken } = useAuthStore.getState()
  const headers = config.headers

  if (accessToken && !headers.get('Authorization')) {
    headers.set('Authorization', `Bearer ${accessToken}`)
  }

  if (csrfToken && !headers.get('X-CSRF-Token')) {
    headers.set('X-CSRF-Token', csrfToken)
  }

  return config
})
```

Esse interceptor apenas adiciona os headers de autenticação e CSRF.

Ele não faz nenhuma destas ações:

- verificar se o token está perto de vencer;
- chamar `/auth/refresh` antes da requisição;
- tratar resposta `401`;
- repetir a requisição original depois de renovar a sessão;
- limpar sessão somente quando o refresh também falhar.

Esse é o ponto principal do problema.

---

## 6. Causa raiz

A causa raiz é:

```txt
O front-end possui refresh de sessão implementado, mas não possui renovação automática acoplada ao ciclo normal de requisições da aplicação.
```

Em termos práticos:

```txt
JWT_ACCESS_EXPIRES_IN=15m
+
api.ts apenas injeta o token atual
+
bootstrapSession só roda no início
+
nenhum retry automático em 401
=
usuário recebe erro de sessão expirada durante o uso
```

O usuário pode estar usando a aplicação, mas isso não importa para o token se nenhuma rotina renova o access token antes ou depois da expiração.

---

## 7. Por que o erro aparece mesmo com o usuário mexendo na aplicação

É importante separar dois conceitos:

### Atividade visual no front-end

Exemplos:

- clicar em abas;
- expandir menus;
- navegar em conteúdo já carregado;
- digitar em campos locais;
- abrir e fechar componentes;
- interagir com elementos que não chamam a API.

Essas ações não renovam o token.

### Atividade autenticada na API

Exemplos:

- carregar dados;
- salvar formulário;
- baixar documento;
- criar notificação;
- atualizar cliente;
- consultar dashboard.

Essas ações enviam o access token para o backend.

Se o access token estiver vencido e não houver interceptor de refresh, a API responde `401`.

Por isso o usuário pode estar mexendo no Portal Sama, mas ainda assim a próxima chamada real para a API pode falhar por token vencido.

---

## 8. Por que não é recomendado apenas aumentar o tempo do access token

Uma solução rápida seria alterar:

```env
JWT_ACCESS_EXPIRES_IN=15m
```

para algo como:

```env
JWT_ACCESS_EXPIRES_IN=8h
```

Essa solução não é recomendada para o Portal Sama.

Motivo: a aplicação manipula documentos empresariais, dados de clientes, certificados, permissões internas e informações sensíveis. Um access token longo aumenta a janela de exploração caso o token seja capturado indevidamente.

O desenho mais seguro é:

```txt
Access token curto
+
Refresh token HTTP-only
+
CSRF
+
Renovação automática controlada
+
Retry seguro em 401
```

Portanto, o ideal é manter o access token curto e corrigir o mecanismo de renovação.

---

## 9. Solução recomendada

A correção deve ser feita principalmente no front-end, no arquivo:

```txt
portal-sama-web/src/services/api.ts
```

A solução recomendada possui três partes:

1. Antes de cada requisição protegida, verificar se o access token está vencido ou próximo de vencer.
2. Se estiver vencido, chamar `/auth/refresh` antes de enviar a requisição original.
3. Se mesmo assim a API retornar `401`, tentar renovar a sessão uma única vez e repetir a requisição original.

Além disso, como o backend rotaciona o refresh token, é obrigatório evitar múltiplos refresh simultâneos.

---

## 10. Por que é necessário controlar refresh simultâneo

O backend faz rotação do refresh token.

Fluxo do backend:

```txt
Recebe refresh token antigo
↓
Valida token antigo
↓
Revoga token antigo
↓
Cria novo refresh token
↓
Envia novo cookie
```

Se várias chamadas chamarem `/auth/refresh` ao mesmo tempo, pode ocorrer esta corrida:

```txt
Requisição A chama /auth/refresh com token antigo
Requisição B chama /auth/refresh com token antigo
↓
A valida e revoga o token antigo
↓
B tenta validar o mesmo token antigo
↓
B falha porque o token já foi revogado
↓
Front-end pode interpretar como sessão expirada
```

Por isso a solução precisa usar uma única promessa compartilhada, normalmente chamada de `refreshPromise` ou mecanismo de `single-flight`.

Enquanto um refresh estiver em andamento, outras requisições devem aguardar o mesmo refresh terminar, em vez de iniciar novos refreshes paralelos.

---

## 11. Implementação recomendada no `api.ts`

Substituir o conteúdo atual de `src/services/api.ts` por uma versão com:

- controle de endpoint de autenticação;
- refresh compartilhado;
- interceptor de request assíncrono;
- interceptor de response com retry único;
- limpeza de sessão somente quando o refresh falhar.

Exemplo de implementação:

```ts
import axios, { AxiosHeaders, type AxiosError, type InternalAxiosRequestConfig } from 'axios'
import { useAuthStore } from '../stores/auth.store'

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || '/api-v2',
  timeout: 20_000,
  withCredentials: true,
  headers: {
    Accept: 'application/json',
  },
})

type RetriableRequestConfig = InternalAxiosRequestConfig & {
  _retry?: boolean
}

let refreshPromise: Promise<unknown> | null = null

function isAuthEndpoint(url?: string) {
  if (!url) return false

  return (
    url.includes('/auth/csrf') ||
    url.includes('/auth/login') ||
    url.includes('/auth/refresh') ||
    url.includes('/auth/logout') ||
    url.includes('/auth/forgot-password')
  )
}

function ensureHeaders(config: InternalAxiosRequestConfig) {
  config.headers = AxiosHeaders.from(config.headers)
  return config.headers
}

function applyAuthHeaders(config: InternalAxiosRequestConfig) {
  const { accessToken, csrfToken } = useAuthStore.getState()
  const headers = ensureHeaders(config)

  if (accessToken) {
    headers.set('Authorization', `Bearer ${accessToken}`)
  }

  if (csrfToken) {
    headers.set('X-CSRF-Token', csrfToken)
  }
}

async function refreshOnce() {
  if (!refreshPromise) {
    refreshPromise = import('./auth.service')
      .then(({ refreshSession }) => refreshSession())
      .finally(() => {
        refreshPromise = null
      })
  }

  return refreshPromise
}

api.interceptors.request.use(async (config) => {
  if (!isAuthEndpoint(config.url)) {
    const store = useAuthStore.getState()

    if (store.status === 'authenticated' && store.accessToken && !store.isTokenFresh()) {
      try {
        await refreshOnce()
      } catch (error) {
        useAuthStore.getState().clearSession()
        throw error
      }
    }
  }

  applyAuthHeaders(config)
  return config
})

api.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    const originalRequest = error.config as RetriableRequestConfig | undefined

    if (
      error.response?.status === 401 &&
      originalRequest &&
      !originalRequest._retry &&
      !isAuthEndpoint(originalRequest.url)
    ) {
      originalRequest._retry = true

      try {
        await refreshOnce()
        applyAuthHeaders(originalRequest)
        return api(originalRequest)
      } catch (refreshError) {
        useAuthStore.getState().clearSession()
        return Promise.reject(refreshError)
      }
    }

    return Promise.reject(error)
  },
)

export function getErrorMessage(error: unknown, fallback = 'Nao foi possivel concluir a acao.') {
  if (axios.isAxiosError(error)) {
    const data = error.response?.data as { error?: { message?: string }; message?: string } | undefined
    return data?.error?.message || data?.message || fallback
  }

  if (error instanceof Error && error.message) return error.message
  return fallback
}
```

---

## 12. Explicação da correção

## 12.1. `isAuthEndpoint()`

Essa função evita que o interceptor tente renovar a sessão ao chamar os próprios endpoints de autenticação.

Sem isso, poderia acontecer loop infinito:

```txt
/auth/refresh falha
↓
interceptor tenta refresh novamente
↓
/auth/refresh falha novamente
↓
loop
```

Por isso os endpoints abaixo precisam ser ignorados pelo auto refresh:

```txt
/auth/csrf
/auth/login
/auth/refresh
/auth/logout
/auth/forgot-password
```

---

## 12.2. `refreshOnce()`

Essa função garante que apenas um refresh aconteça por vez.

Se várias requisições perceberem o token vencido ao mesmo tempo, todas aguardam a mesma promessa:

```ts
let refreshPromise: Promise<unknown> | null = null
```

Isso é essencial porque o backend rotaciona o refresh token. Sem esse controle, o próprio mecanismo de segurança do backend pode causar logout indevido por corrida entre requisições simultâneas.

---

## 12.3. Interceptor de request

Antes de enviar uma requisição protegida, o front-end verifica:

```ts
store.status === 'authenticated' && store.accessToken && !store.isTokenFresh()
```

Se o token não estiver fresco, ele chama:

```ts
await refreshOnce()
```

Depois aplica os headers atualizados:

```ts
applyAuthHeaders(config)
```

Assim, a requisição original já sai com o token novo.

---

## 12.4. Interceptor de response

Mesmo com refresh preventivo, ainda pode ocorrer uma resposta `401`, por exemplo:

- diferença de tempo entre cliente e servidor;
- token revogado no backend;
- token expirado entre a montagem e o envio da requisição;
- primeira requisição feita depois de muito tempo com estado antigo.

Nessa situação, o interceptor tenta uma única renovação:

```ts
if (error.response?.status === 401 && !originalRequest._retry) {
  originalRequest._retry = true
  await refreshOnce()
  applyAuthHeaders(originalRequest)
  return api(originalRequest)
}
```

Esse comportamento evita que o usuário veja erro de sessão expirada quando ainda existe refresh token válido.

---

## 13. Fluxo correto após a correção

## 13.1. Token ainda válido

```txt
Usuário executa ação
↓
Interceptor verifica token
↓
Token ainda está fresco
↓
Header Authorization é aplicado
↓
Requisição segue normalmente
↓
API responde sucesso
```

## 13.2. Token vencido antes da requisição

```txt
Usuário executa ação
↓
Interceptor verifica token
↓
Token não está fresco
↓
Front-end chama /auth/refresh
↓
Backend emite novo access token e novo refresh cookie
↓
Front-end atualiza Zustand
↓
Requisição original segue com token novo
↓
API responde sucesso
```

## 13.3. Token rejeitado pela API

```txt
Usuário executa ação
↓
Requisição chega com token inválido ou expirado
↓
API responde 401
↓
Interceptor captura o 401
↓
Front-end chama /auth/refresh
↓
Refresh funciona
↓
Front-end repete a requisição original
↓
Usuário não é derrubado
```

## 13.4. Refresh token também inválido ou expirado

```txt
Usuário executa ação
↓
Access token falha
↓
Front-end tenta /auth/refresh
↓
Refresh token também falha
↓
Front-end limpa a sessão
↓
Usuário é enviado para login
```

Esse é o único caso em que a aplicação deve realmente encerrar a sessão.

---

## 14. Ajuste opcional para controle de atividade do usuário

A correção principal deve ser feita no `api.ts`.

No entanto, para uma solução mais refinada, pode ser criado um controle de atividade do usuário.

Esse controle permitiria diferenciar:

- usuário ativo;
- usuário inativo;
- sessão expirada por inatividade;
- sessão encerrada por limite absoluto.

Exemplo de configuração futura:

```env
SESSION_IDLE_TIMEOUT=30m
SESSION_ABSOLUTE_TIMEOUT=8h
```

### Recomendação de segurança

Não é recomendado criar um timer cego que chame `/auth/refresh` continuamente sem verificar atividade do usuário.

Motivo: como o refresh token é renovado a cada refresh, um timer cego poderia manter uma sessão aberta indefinidamente em um computador abandonado.

Para o Portal Sama, que lida com documentos empresariais, o correto é:

```txt
renovar em requisições reais
+
renovar após 401 uma única vez
+
opcionalmente renovar por timer somente se houver atividade recente
+
encerrar por inatividade após limite configurado
```

---

## 15. Possível melhoria futura no store de autenticação

O store atual guarda:

```ts
accessToken
csrfToken
expiresAt
status
user
permissions
roles
```

Para evoluir a segurança, poderia guardar também:

```ts
lastActivityAt
sessionStartedAt
```

Com isso seria possível implementar regras como:

```txt
Se usuário ficou mais de 30 minutos sem atividade:
  limpar sessão local
  chamar logout quando possível
  redirecionar para login

Se sessão passou de 8 horas desde o login:
  exigir novo login, mesmo com refresh token válido
```

Essa melhoria não é obrigatória para corrigir o bug atual, mas é recomendada para maturidade de segurança.

---

## 16. Impacto esperado da correção

Após implementar o auto refresh no `api.ts`, o comportamento esperado é:

- usuário não será derrubado apenas porque o access token venceu;
- ações feitas depois de 15 minutos continuarão funcionando se o refresh token ainda estiver válido;
- erros `401` causados por access token vencido serão tratados automaticamente;
- a requisição original será repetida sem o usuário precisar recarregar a página;
- a sessão só será encerrada se o refresh token também estiver inválido, expirado ou revogado;
- a segurança do access token curto será preservada.

---

## 17. O que não deve ser feito

Não aplicar estas soluções como correção principal:

### 17.1. Não aumentar o access token para muitas horas

Evitar:

```env
JWT_ACCESS_EXPIRES_IN=8h
```

Isso reduz a segurança.

### 17.2. Não salvar refresh token no localStorage

O refresh token deve continuar em cookie HTTP-only.

Evitar qualquer solução como:

```ts
localStorage.setItem('refreshToken', token)
```

Isso aumentaria o risco em caso de XSS.

### 17.3. Não chamar refresh em paralelo

Evitar que cada requisição vencida chame `/auth/refresh` individualmente.

Isso pode quebrar a sessão por causa da rotação do refresh token.

### 17.4. Não fazer retry infinito em 401

O retry deve ocorrer apenas uma vez por requisição.

Caso contrário, pode ocorrer loop infinito em caso de refresh token inválido.

---

## 18. Checklist de implementação

Status 2026-06-01: a parte tecnica do front-end foi implementada em `portal-sama-web/src/services/api.ts` e validada localmente com build/lint. O aceite funcional em EasyPanel ainda deve ser feito aguardando o vencimento real do access token ou usando ambiente com validade reduzida.

## 18.1. Front-end

- [ ] Atualizar `src/services/api.ts` com interceptor de request assíncrono.
- [ ] Adicionar `refreshOnce()` com controle de promessa única.
- [ ] Ignorar endpoints de autenticação no auto refresh.
- [ ] Aplicar headers atualizados depois do refresh.
- [ ] Adicionar interceptor de response para tratar `401`.
- [ ] Repetir a requisição original apenas uma vez.
- [ ] Limpar sessão somente se o refresh falhar.
- [ ] Garantir que `withCredentials: true` continue ativo.
- [ ] Garantir que o token CSRF continue sendo enviado.

## 18.2. Backend

A princípio, não é necessário alterar o backend para corrigir esse bug.

O backend já possui:

- endpoint `/auth/refresh`;
- cookie HTTP-only para refresh token;
- rotação de refresh token;
- validação de expiração;
- revogação de token antigo;
- CSRF em rotas sensíveis.

Mudanças futuras recomendadas:

- [ ] Adicionar política explícita de idle timeout.
- [ ] Adicionar política explícita de absolute session timeout.
- [ ] Auditar falhas de refresh com motivo técnico.
- [ ] Expor códigos de erro mais específicos para sessão expirada, refresh expirado e token revogado.

---

## 19. Plano de teste manual

## 19.1. Teste antes da correção

1. Fazer login no Portal Sama.
2. Permanecer na aplicação por mais de 15 minutos.
3. Executar uma ação que chama a API, como salvar, listar, baixar ou atualizar dados.
4. Verificar se aparece erro de sessão expirada.

Resultado atual esperado:

```txt
A ação falha com mensagem de sessão expirada ou token inválido.
```

## 19.2. Teste depois da correção

1. Fazer login no Portal Sama.
2. Permanecer na aplicação por mais de 15 minutos.
3. Executar uma ação que chama a API.
4. Verificar no DevTools/Network se ocorreu chamada para `/auth/refresh`.
5. Verificar se a ação original foi executada com sucesso.

Resultado esperado depois da correção:

```txt
A aplicação renova a sessão automaticamente e a ação do usuário funciona sem recarregar a página.
```

## 19.3. Teste de refresh token expirado

1. Fazer login.
2. Invalidar manualmente o refresh token no banco ou remover o cookie `sama_refresh_token`.
3. Executar ação autenticada após o access token vencer.

Resultado esperado:

```txt
O front-end tenta renovar a sessão, falha corretamente, limpa a sessão local e redireciona o usuário para o login.
```

## 19.4. Teste de concorrência

1. Fazer login.
2. Aguardar o access token vencer.
3. Abrir uma página que dispare várias consultas simultâneas.
4. Verificar no Network se somente uma chamada de `/auth/refresh` é feita.

Resultado esperado:

```txt
Apenas uma chamada de refresh deve ocorrer. As demais requisições devem aguardar o mesmo refresh e seguir com o novo access token.
```

---

## 20. Critérios de aceite

Status 2026-06-01: criterios tecnicos de implementacao foram atendidos localmente. O aceite de experiencia do usuario continua pendente de validacao real no EasyPanel apos vencimento do access token.

A correção pode ser considerada concluída quando:

- [ ] o usuário consegue continuar usando a aplicação após o vencimento do access token;
- [ ] a próxima chamada autenticada após o vencimento renova a sessão automaticamente;
- [ ] a requisição original é repetida sem intervenção manual;
- [ ] o usuário não precisa recarregar a página para voltar a usar o sistema;
- [ ] apenas uma chamada `/auth/refresh` ocorre quando várias requisições expiram ao mesmo tempo;
- [ ] se o refresh token estiver inválido, o usuário é redirecionado para login;
- [ ] não há retry infinito;
- [ ] não houve aumento inseguro do tempo do access token;
- [ ] o refresh token continua protegido em cookie HTTP-only;
- [ ] o CSRF continua sendo validado.

---

## 21. Conclusão

O comportamento de sessão expirada durante o uso da aplicação é confirmado pelo código atual.

O backend já possui uma arquitetura adequada para sessão segura:

```txt
access token curto
refresh token em cookie HTTP-only
rotação de refresh token
proteção CSRF
```

O problema está no front-end, especificamente na ausência de renovação automática no fluxo normal das requisições.

A correção recomendada é implementar no `src/services/api.ts`:

```txt
pre-refresh antes da requisição
+
retry único após 401
+
controle de refresh simultâneo
+
limpeza de sessão somente quando o refresh falhar
```

Essa solução corrige a experiência do usuário sem enfraquecer a segurança da aplicação.
