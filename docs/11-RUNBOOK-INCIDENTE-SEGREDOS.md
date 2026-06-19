# Runbook de Incidente de Segredos

Use este runbook quando `.env`, token, senha, chave privada, cookie secret, VAPID private key, API key ou credencial de banco puder ter sido exposto.

## Escopo confirmado

- `.env` real existe em diretórios locais do legado e da API.
- Os ZIPs atuais visíveis não confirmaram `.env`, `.git`, `node_modules` ou `dist`.
- Mesmo assim, qualquer histórico de pacote/backup compartilhado deve ser tratado como risco até revisão.

## Ação imediata

1. Identificar todos os artefatos que podem conter segredos.
2. Remover artefatos inseguros de compartilhamentos e backups acessíveis.
3. Criar pacote seguro sem `.env`, `.git`, `node_modules`, `dist`, logs e `.ai-tests`.
4. Rotacionar credenciais por categoria.
5. Invalidar sessões e tokens antigos quando aplicável.
6. Registrar data, responsável e evidência externa de rotação.

## Ordem de rotação

1. Banco de dados.
2. JWT access/refresh secrets.
3. CSRF secret.
4. Bootstrap/admin.
5. Chaves de criptografia de certificado.
6. Acessórias.
7. ZapSign, se continuar no escopo.
8. SMTP/IMAP.
9. Web Push VAPID.
10. OpenAI e demais integrações.

## Validações depois da rotação

- [ ] Login real.
- [ ] Refresh real.
- [ ] Logout real.
- [ ] Smoke público.
- [ ] Smoke autenticado.
- [ ] `ops:secrets:check`.
- [ ] `ops:readiness`.
- [ ] Web Push, se habilitado.
- [ ] Acessórias, se habilitada.

## Regras permanentes

- Nunca versionar `.env`.
- Nunca colocar `.env` em ZIP.
- Nunca publicar logs com token, senha, cookie ou segredo.
- Manter `.env.example` sem valor real.
- Preferir secret manager ou variáveis protegidas do ambiente.
