# Auditoria Deploy - Frontend e UI/UX

## Etapas 7 e 8 - Frontend, navegação e experiência

## Stack e execução

| Item | Resultado |
| --- | --- |
| Framework | React 19, React Router 7, Vite 8, TypeScript |
| HTTP | Axios com `withCredentials`, Authorization e CSRF |
| Estado de auth | Zustand em memória |
| Ícones/UI | `lucide-react`, Tailwind 4 |
| Build | Passou |
| Lint | Passou |
| Testes contratuais | 9 passaram |
| E2E Playwright | 11 passaram, 1 skip, 2 falharam |
| Smoke local `/login` | 200 |

## Resultado dos testes web

Passou:

- `npm.cmd run lint`
- `npm.cmd run build`
- `npm.cmd test -- --runInBand`

Falhou:

- `npm.cmd run test:e2e`

Falhas:

| Teste | Local | Esperado | Observado |
| --- | --- | --- | --- |
| `home page renders authenticated shortcuts from session permissions` | `tests/e2e/smoke.spec.ts:1316` | Texto `Integracao` | A tela atual mostrou cards como `Base local`, `Diagnostico Acessorias` e `Banco local`. |
| `home page renders collaborator daily priorities from available data` | `tests/e2e/smoke.spec.ts:1332` | Heading `Entregue pelo Acessorias` | A tela atual mostrou `Baixas sincronizadas`. |

Evidências geradas pelo Playwright:

- `portal-sama-web/test-results/smoke-home-page-renders-au-4ab1d-ts-from-session-permissions-chromium/test-failed-1.png`
- `portal-sama-web/test-results/smoke-home-page-renders-au-4ab1d-ts-from-session-permissions-chromium/trace.zip`
- `portal-sama-web/test-results/smoke-home-page-renders-co-56cd2-orities-from-available-data-chromium/test-failed-1.png`
- `portal-sama-web/test-results/smoke-home-page-renders-co-56cd2-orities-from-available-data-chromium/trace.zip`

Leitura: as falhas parecem contrato de teste defasado contra a UI atual de Acessórias/Home, não quebra visual óbvia. Ainda assim, release não deve ignorar E2E vermelho.

## Revisão visual

As screenshots inspecionadas da Home Admin e Home colaborador estavam visualmente coerentes:

- Sem sobreposição evidente.
- Sidebar, cartões e áreas principais legíveis.
- Hierarquia de conteúdo clara.
- Não foi observado overflow horizontal nos testes de boas-vindas.

Pendências:

- Teste manual em mobile real.
- Teste com dados reais maiores: nomes longos, muitos clientes, muitas notificações, tabelas extensas.
- Teste de acessibilidade dedicado com teclado/leitor/contraste.

## UX operacional

Pontos bons:

- Rotas protegidas dependem de sessão autenticada.
- Home por perfil existe e usa permissões.
- API client renova sessão ao detectar token próximo da expiração ou 401.
- Mensagens e nomes estão em português operacional.

Riscos:

- No E2E web sem backend real apareceram erros de proxy para `/api-v2/notifications/stream` e `/api-v2/me/security`. Parte disso é ruído do ambiente de teste, mas a UI deve continuar resiliente quando SSE/segurança estiverem temporariamente indisponíveis.
- A Home/Acessórias parece ter mudado texto/estrutura sem atualizar o contrato E2E.
- Homologação real por perfil ainda não foi feita.

## Recomendações frontend

1. Decidir se os textos atuais da Home são o contrato correto.
2. Se forem corretos, atualizar os testes Playwright.
3. Se não forem corretos, ajustar UI para restaurar `Integracao` e `Entregue pelo Acessorias`.
4. Mockar ou estabilizar `/notifications/stream` e `/me/security` nos E2E.
5. Rodar `npm.cmd run test:e2e` até verde.
6. Rodar `npm.cmd run test:e2e:real` em homologação com credenciais reais.
7. Fazer rodada mobile com Chrome/Edge em viewport estreita e nomes/dados longos.

## Conclusão frontend

Frontend está compilável e visualmente próximo, mas **não aprovado** até fechar os dois E2E e homologar perfis reais.

