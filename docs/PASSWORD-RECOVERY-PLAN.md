# Recuperação de senha — contrato de implementação

Base de desenvolvimento: `baseline/installed-49bd7a4` (commit `49bd7a4ba62839a2eb219e126379e9080d1a2c4d`).

Objetivo: habilitar o fluxo de recuperação de senha de usuário do Better Auth com Stalwart, sem alterar o login atual nem depender de modificações manuais no banco.

## Backend

- Configurar `emailAndPassword.sendResetPassword` **apenas quando o transporte SMTP estiver configurado**; não registrar token nem URL contendo token.
- SMTP com TLS/STARTTLS, autenticação, timeouts e segredo por variáveis no ambiente de execução (Coolify secrets).
- Token Better Auth, 30 minutos, uso único. Resposta genérica para não revelar existência de contas.
- URL de retorno limitada à origem autenticada de Paperclip e rate limiting nos endpoints de recuperação.
- Invalidar sessões anteriores quando a senha for trocada e auditar apenas eventos não sensíveis.

## Frontend

- Adicionar link “Esqueci minha senha” na tela de login.
- Tela de solicitação por e-mail com resposta neutra.
- Tela de redefinição com senha nova, confirmação, validação de token, feedback de expiração e retorno ao login.

## Testes e critérios de entrega

- Unit: SMTP configurado/indisponível, validação de origem, tratamento de erros sem expor secretos.
- Integration: solicitação, link único, expiração, senha alterada, sessão anterior invalidada.
- UI: navegação login → solicitação → redefinição → login, responsiva.
- E-mail real somente em conta de testes autorizada. Sem enviar mensagens para usuários reais durante homologação.
- Build, typecheck, testes, revisão de segurança e rollback documentados.

**Restrições:** sem push/deploy na produção, sem reaproveitar o banco nem os volumes de produção no staging, sem credenciais no repositório.
