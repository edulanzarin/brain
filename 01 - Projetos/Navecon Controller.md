---
tags: [tipo/projeto, projeto/navecon-controller]
criado: 2026-07-19
status: ativo
codigo_em: ~/Dev/Navecon_Controller_Assistant_Chatbot_Creation
---

# Navecon Controller

> Sistema interno de controladoria da Navecon: gestão de clientes, reuniões com
> ata, análises tributárias/financeiras, cobrança de documentos e assinatura
> eletrônica. Gerado na **Abacus.AI** (plataforma de IA que escreve e hospeda o
> app) e sendo portado para servidor próprio.

Código em: `~/Dev/Navecon_Controller_Assistant_Chatbot_Creation`

## Estado atual

Rodando em Docker, `http://IP:4088`, com Postgres 17 local. Login, leitura e
escrita no banco validados end-to-end. Falta o cutover de produção.

## Stack e contexto técnico

Next.js 14 (App Router, ~37k linhas) · Prisma 6 + PostgreSQL 17 · NextAuth com
2FA por e-mail · Tailwind + shadcn/ui · 110 rotas de API · 37 tabelas.

Segurança acima da média para app interno: 2FA, bloqueio geográfico (só Brasil),
checagem de senha vazada no HaveIBeenPwned, limite de sessões, auditoria de tudo,
janela de horário permitido.

## Decisões importantes

- **Banco migrou junto, sem escolha.** O `DATABASE_URL` da Abacus resolvia para
  um IP privado (`172.21.254.219`) — inalcançável fora da infra deles. Postgres
  passou a ser serviço do compose, restaurado do dump de 05/07. Ver
  [[Plataforma de IA hospedada prende o app pelo banco]].
- **Manter a Abacus como provedor de LLM/e-mail/PDF por enquanto.** Porta o app
  primeiro, troca as APIs depois — evita mudar duas variáveis ao mesmo tempo.
- **Postgres só em loopback** (`127.0.0.1:5432`); o app fala por rede interna do
  compose. Só a 4088 fica exposta.
- **Repo git criado do zero** (veio como zip, sem versionamento), com
  `backups/`, `Uploads/` e `logs/` fora do Git — têm dados de cliente e hashes.

## Dependências externas (e o custo de trocar)

| API | Uso | Trocar é |
|---|---|---|
| Abacus LLM (`/v1/chat/completions`) | 7 rotas: extração de balancete, transcrição de ata, resumos | Fácil — é OpenAI-compatible |
| Abacus `sendNotificationEmail` | Todo e-mail (2FA, convites, cobranças) | Trabalhoso — templates `NOTIF_ID_*` vivem lá |
| Abacus HTML→PDF | Export de análises | Médio — Gotenberg/Puppeteer resolvem |
| Acessórias | Sincroniza obrigações contábeis | Só se a Navecon trocar de ferramenta |
| ZapSign + Autentique | Assinatura de ata (redundantes) | Webhooks exigem IP alcançável de fora |
| AWS S3 | Upload de documentos | Vai quebrar até ter credencial explícita |

## Aprendizados (viram notas atômicas)

- [[Plataforma de IA hospedada prende o app pelo banco]]
- [[Next.js standalone no Docker e o outputFileTracingRoot]]

## Próximos passos

- [ ] `NEXTAUTH_URL` com o IP real do servidor (a Abacus injetava; não existia no `.env`)
- [ ] Credenciais AWS explícitas — o `S3Client({})` dependia de IAM role da máquina
- [ ] Backup fresco da Abacus antes do cutover (o dump em uso é de 05/07)
- [ ] Cron do host para `api/cron/*` (o scheduler era da plataforma)
- [ ] Centralizar as 7 chamadas de LLM num `lib/llm.ts` antes de trocar de provedor
- [ ] Remover `appllm-lib.js` do `layout.tsx` — script da Abacus, inútil fora de lá

## Links

- Mapa: [[Desenvolvimento]]
- Verificação end-to-end antes de dar por pronto: [[Verificar no build de produção, não só em dev]]
