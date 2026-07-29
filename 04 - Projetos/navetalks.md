---
tags: [tipo/projeto, projeto/navetalks]
criado: 2026-07-28
status: ativo
codigo_em: ~/Dev/navetalks
---

# navetalks

> CRM de atendimento multicanal centrado em WhatsApp: vários atendentes e setores
> respondem os clientes por uma caixa compartilhada na plataforma, e o cliente recebe
> tudo no WhatsApp dele. SaaS multiempresa. É a linhagem Evotalks/Whaticket.

Código em: `~/Dev/navetalks` · remote `git@github.com:edulanzarin/navetalks.git`

## Estado atual

Base/chassi de pé e verificado (build de produção limpo e fluxo autenticado exercitado
ponta a ponta):

- Auth completa: sessão opaca no banco, scrypt, funil de escopo por org+setor, gate no
  middleware e no wrapper de API. Launcher só mostra o que a sessão libera.
- Inbox (caixa compartilhada) com lista/thread/composer, os quatro estados e a mecânica
  de assumir (primeiro assume, invariante estrutural) / resolver / enviar.
- Schema multiempresa (Org na raiz), migration inicial aplicada, seed com empresa demo.
- Adapter de canal + worker persistente em stub — sobe sem WhatsApp real ainda.
- Docker igual em dev e prod (app/db/migrate/worker): `docker compose up -d --build`.

Login demo: `admin@navetalks.dev / admin123` · `ana@navetalks.dev / atendente123`.

## Infra

Slug `navetalks` · app `navetalks-app` na `4050` · banco `navetalks-db` na `5050` ·
`navetalks-migrate` · `navetalks-worker` (processo vivo do WhatsApp). Chassi e mapa de
portas em [[Infra]].

## Stack

Next.js (App Router) · React · TypeScript · Tailwind v4 · Postgres · Prisma · worker em
tsx. WhatsApp por adapter (Baileys primeiro, Cloud API depois).

## Decisões importantes

- **WhatsApp por adapter, Baileys primeiro.** O inbox e o worker falam só com a interface
  de canal; Baileys (não-oficial) agora, Cloud API oficial depois, sem tocar no resto —
  [[Adapter de canal isola o app do provider de mensageria]]. Risco assumido: Baileys é
  contra o ToS e pode banir o número.
- **SaaS multiempresa desde o schema.** Tudo pendura em `orgId` e o funil clampa por ele —
  [[Escopo de dado se clampa no servidor, num funil só]].
- **Worker persistente separado do app.** Baileys precisa de processo vivo segurando a
  sessão; o app Next fica stateless. A ingestão de inbound é idempotente e compartilhada
  por worker/webhook/polling — mesma família de
  [[Polling substitui webhook quando não há IP público]].
- **Caixa compartilhada = recurso do setor.** A conversa é um recurso; o setor é o alvo, o
  primeiro atendente assume —
  [[Uma resposta canônica de um grupo é um token compartilhado]].

## Aprendizados (viraram notas)

- [[Adapter de canal isola o app do provider de mensageria]]

## Próximos passos

- [ ] Provider Baileys real atrás do adapter (QR, sessão persistida, enviar/receber).
- [ ] Realtime cross-process worker → app (Postgres LISTEN/NOTIFY) + SSE no inbox.
- [ ] Telas de config: canais (conectar/QR), setores, equipe, papéis.
- [ ] Marcar conversa como lida ao abrir; anexos/mídia; notas internas.
- [ ] Onboarding de empresa (SaaS): criar org + admin.
- [ ] Gotcha do worker: módulos que ele toca usam import relativo, não o alias `@/`
      (tsx não resolve paths do tsconfig). Candidato a virar técnica se reaparecer.

## Conexões
- Usa: [[Infra]] · [[Sistema de cores e tema do dashboard]] · [[Sidebar em acordeão e layout de módulo]] · [[Sessão opaca no banco separa autenticação de permissão]] · [[Cravar o seam de permissão antes do login]] · [[Permissão composta por papéis somados, não exceção por usuário]] · [[Escopo de dado se clampa no servidor, num funil só]] · [[Adapter de canal isola o app do provider de mensageria]] · [[Uma resposta canônica de um grupo é um token compartilhado]] · [[Polling substitui webhook quando não há IP público]] · [[Migrations em container próprio no Docker Compose]] · [[Next.js standalone no Docker e o outputFileTracingRoot]]
- Mapa: [[Projetos]]
