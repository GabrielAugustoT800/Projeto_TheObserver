# Security.md — Email Organizer

Checklist e diretrizes de segurança para este projeto. Deve ser revisado
sempre que um novo módulo lidar com credenciais, dados pessoais ou rede.

## 1. Segredos e credenciais

- **Nunca versionar**: `data/credentials.json`, `data/token.json`, `.env`
  devem estar no `.gitignore` desde o primeiro commit.
- **Permissões de arquivo em produção**: restringir esses arquivos com
  `chmod 600` (leitura/escrita só pro dono do processo).
- **Nunca logar** o conteúdo desses arquivos, nem parcialmente, em stdout,
  arquivos de log ou serviços de monitoramento.
- Variáveis de ambiente em produção devem vir das *environment variables*
  nativas da plataforma de hospedagem, não de um `.env` copiado pro servidor.

## 2. Escopo de permissões (Gmail API)

- Usar exclusivamente o escopo `gmail.readonly`.
- Nunca solicitar escopos de envio (`gmail.send`) ou modificação
  (`gmail.modify`) — o app não precisa alterar nem enviar nada.
- Revisar o escopo sempre que a "Tela de consentimento OAuth" for
  reconfigurada.

## 3. Armazenamento local (SQLite)

- O banco (`data/*.sqlite`) pode conter trechos de email → tratar como dado
  sensível.
- Incluir no `.gitignore`.
- Aplicar as mesmas permissões restritas de arquivo (`chmod 600`).
- Definir uma política de retenção (ex: apagar histórico com mais de N
  semanas) para reduzir a superfície de dados acumulados.

## 4. Ollama (rede local)

- O serviço roda em `localhost:11434` sem autenticação própria.
- Garantir que essa porta **não seja exposta externamente** — bloquear via
  firewall/security group da hospedagem, permitindo acesso só via loopback.
- Nunca fazer bind em `0.0.0.0` para esse serviço.

## 5. Telegram

- O bot deve responder/notificar **apenas** o `chat_id` configurado nas
  variáveis de ambiente.
- Qualquer update recebido de um `chat_id` diferente deve ser ignorado
  silenciosamente (não processado, não logado com conteúdo).
- Tratar o token do bot como segredo (ver seção 1).

## 6. Logs e tratamento de erros

- Logs de aplicação não devem conter: conteúdo de emails, tokens, chaves,
  ou stack traces que exponham variáveis sensíveis.
- Preferir logar metadados (ex: "processado email com ID X") em vez de
  conteúdo.

## 7. Deploy

- Subir credenciais/token apenas via canais seguros da plataforma de
  hospedagem (secrets manager / environment variables), nunca por upload
  de arquivo público ou commit temporário.
- Confirmar que o serviço roda como *worker* isolado, sem porta HTTP pública
  desnecessária.

## Itens em aberto (revisar conforme o projeto evolui)

- [ ] Rotação de tokens (Google e Telegram)
- [ ] Política de retenção de dados no SQLite
- [ ] Necessidade (ou não) de criptografia em repouso para `data/`