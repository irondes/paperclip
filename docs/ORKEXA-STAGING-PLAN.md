# Orkexa — Plano de desenvolvimento e staging

> Documento operacional. Este repositório contém **código**, não os dados, credenciais, chaves ou backups da instalação existente.

## Estado inicial

- Upstream: `paperclipai/paperclip`, branch `master`.
- Fork: `irondes/paperclip`.
- Versão instalada de referência: commit `49bd7a4ba62839a2eb219e126379e9080d1a2c4d`, preservado na branch `baseline/installed-49bd7a4`.
- Implementações novas partem da referência instalada, não diretamente do último `master`.
- O fork não substitui a imagem Docker, o banco nem os volumes da produção.

## Isolamento de ambientes

| Ambiente | Código | Banco e arquivos | Publicação |
| --- | --- | --- | --- |
| Produção existente | Imagem atual, sem alterações | Volumes atuais, intocados | Continua atendendo |
| Staging | Imagem construída de uma branch de teste | Volume e PostgreSQL **separados** | Subdomínio sob controle de acesso |
| Desenvolvimento | Branch `feat/*` | Dados sintéticos | Sem acesso a produção |

A branch `staging` acompanha a linha de desenvolvimento; **não autoriza deploy automático**. A branch `baseline/installed-49bd7a4` é uma referência imutável operacional.

## Pré-requisitos antes do staging

1. Registrar backup coerente do banco pelo mecanismo nativo do Paperclip; guardar uma cópia independente, com verificação de integridade, além de compose, configuração e arquivos persistentes.
2. Criar infraestrutura de staging no Coolify com projeto, nomes, portas, volume, credenciais e domínio próprios.
3. Conferir disco, memória e CPU antes de build: a imagem atual é grande. Se necessário, construir em outro ambiente e publicar imagem versionada.
4. Bloquear tarefas agendadas, agentes que executem ações reais, integrações externas, webhooks e mensagens até homologação.
5. Usar dados sintéticos inicialmente. Se for indispensável testar dados reais, restaurar somente em ambiente isolado e restringir os acessos.
6. Colocar Cloudflare Access ou proteção equivalente antes de expor staging.

## Recuperação de senha (primeira entrega)

- Integrar ao Better Auth existente via `emailAndPassword.sendResetPassword`.
- Transportar e-mail com SMTP do Stalwart usando autenticação e TLS/STARTTLS; configurar remetente, host, porta e segredo fora do Git.
- Implementar na UI “Esqueci minha senha”, pedido de envio e formulário para definir senha nova.
- Links temporários de uso único (30 minutos), sem registrar tokens em logs, sem revelar se uma conta existe.
- Aplicar rate limit, verificar a origem dos links e invalidar sessões anteriores após a redefinição.
- Testar envio com caixa controlada, caso de token expirado/usado, redirecionamentos e regressão de login.
- Não habilitar a recuperação pública se as credenciais SMTP ou a entrega não estiverem verificadas.

## Migração e rollback

A produção **não** será migrada automaticamente pelo fork. Após homologação: criar backup imediatamente antes de qualquer corte; testar restauração em ambiente isolado; validar dados, usuários, agentes e tarefas; planejar janela de mudança; solicitar aprovação explícita. Manter imagem e volume originais até aceitação completa. Não executar migração de banco contra a base ativa com versão não homologada.

## Próximas tarefas

1. Implementar recuperação em `feat/password-recovery-safe`.
2. Construir/testar imagem derivada do commit instalado.
3. Preparar staging isolado no Coolify e confirmar recursos disponíveis.
4. Homologar e documentar procedimento de restauração.
5. Só com aprovação, planejar atualização da produção.

Não adicionar a este repositório arquivos `.env`, SMTP passwords, dados dos agentes, backups, exports ou tokens.
