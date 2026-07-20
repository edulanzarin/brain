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
| ~~Abacus `sendNotificationEmail`~~ | Todo e-mail (2FA, convites, cobranças) | **Feito** — SMTP/Gmail, fallback pra Abacus |
| Abacus HTML→PDF | Export de análises | Médio — Gotenberg/Puppeteer resolvem |
| Acessórias | Sincroniza obrigações contábeis | Só se a Navecon trocar de ferramenta |
| Autentique | Assinatura de ata | **Resolvido por polling** — ZapSign é código morto |
| AWS S3 | Upload de documentos | Vai quebrar até ter credencial explícita |

## Aprendizados (viram notas atômicas)

- [[Plataforma de IA hospedada prende o app pelo banco]]
- [[Next.js standalone no Docker e o outputFileTracingRoot]]
- [[Polling substitui webhook quando não há IP público]]

## Rodada 2 — saída da LAN sem perder funcionalidade

Objetivo: rodar só na rede interna mantendo tudo funcionando.

- **Assinatura**: webhook é chamada de entrada, não chega atrás de IP privado.
  Conciliação extraída para `lib/assinatura-sync.ts`, com dois gatilhos
  idempotentes — webhook e cron de polling. Ver [[Polling substitui webhook quando não há IP público]].
- **E-mail**: `enviarEmail()` virou ponto único e ganhou motor SMTP
  (nodemailer/Gmail) com fallback pra Abacus. Migrar = preencher `.env`.
- Os 11 pontos de envio duplicavam `fetch` **e** a lógica do remetente. Isso
  espalhava um bug: o remetente vinha do hostname do `NEXTAUTH_URL`, que em LAN
  é um IP (`noreply@192.168.1.34`), domínio inválido. Com 2FA obrigatório,
  trancaria todos fora. Centralizar matou 9 cópias do mesmo bug.

## Próximos passos

- [ ] `NEXTAUTH_URL` com o IP real do servidor (a Abacus injetava; não existia no `.env`)
- [ ] Credenciais AWS explícitas — o `S3Client({})` dependia de IAM role da máquina
- [ ] Backup fresco da Abacus antes do cutover (o dump em uso é de 05/07)
- [ ] Cron do host para `api/cron/*` (o scheduler era da plataforma)
- [ ] Centralizar as 7 chamadas de LLM num `lib/llm.ts` antes de trocar de provedor
- [ ] Credenciais SMTP reais (senha de app do Gmail) e primeiro login de validação

## Links

- Mapa: [[Projetos]]
- Verificação end-to-end antes de dar por pronto: [[Verificar no build de produção, não só em dev]]
