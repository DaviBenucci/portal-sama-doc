# Runbook Backup e Restore

Objetivo: provar que banco, storage privado e configuração operacional podem ser recuperados antes do go-live.

## Escopo

- Banco MySQL.
- Storage privado de documentos.
- Storage de certificados.
- PDFs de contratos.
- Evidências operacionais necessárias.

## Backup

- [ ] Executar backup do banco.
- [ ] Executar backup do storage privado.
- [ ] Gerar hashes dos artefatos.
- [ ] Validar tamanho e presença dos arquivos esperados.
- [ ] Armazenar evidência fora do servidor da aplicação.

## Restore drill

- [ ] Criar banco alvo isolado.
- [ ] Restaurar dump no banco isolado.
- [ ] Restaurar storage em diretório isolado.
- [ ] Apontar ambiente de teste para banco/storage restaurados.
- [ ] Rodar health/readiness.
- [ ] Validar login de usuário de teste.
- [ ] Validar listagem de clientes/documentos.
- [ ] Validar download autorizado de documento.
- [ ] Validar que storage não está em webroot público.

## Critérios de aceite

- [ ] Restore completa sem erro.
- [ ] Dados essenciais ficam consultáveis.
- [ ] Arquivos privados seguem acessíveis apenas por rota autorizada.
- [ ] Tempo de recuperação registrado.
- [ ] Responsável e data registrados.

## Pendências conhecidas

A validação Codex de 2026-06-19 não comprovou backup/restore real. Esse item continua bloqueador para produção com dados sensíveis.
