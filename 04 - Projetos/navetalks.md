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

## Duas linhagens (atenção)

Existem duas bases distintas com este nome — não confundir:

- **Remote / GitHub (linhagem A):** o app anterior, backend-first — auth de sessão
  opaca, escopo por org+setor, inbox, worker persistente e integração Baileys real.
  Descrito na seção "Linhagem A" abaixo. Ainda vive no remote (`main`, ~10 commits).
- **Local / working copy (linhagem B):** rebuild **greenfield** feito em ago/2026 a
  partir de um esboço de design. Design-first, as 7 telas de pé, sem auth ainda. É o
  que está hoje em `~/Dev/navetalks` (repo git novo, sem remote configurado). O Eduardo
  optou por recomeçar do zero aqui em vez de restaurar a linhagem A.

Push do local sobrescreve a linhagem A no remote — decisão do Eduardo, não automática.

## Estado atual (linhagem B — local)

Base de pé e verificada (build de produção limpo + as 7 rotas respondendo 200 em SSR
contra o Postgres seedado):

- **Design "Aurora Glass":** tokens semânticos claro/escuro (vidro, raio, gradiente
  aurora) expostos ao Tailwind v4, script anti-flash. Kit de primitivos reutilizáveis
  em `src/components/ui` (button, card, badge, avatar, input, segmented, stat-tile,
  progress, toast, skeleton, empty/error-state, ícones). Aplica
  [[Sistema de cores e tema do dashboard]] com estética arredondada/glass.
- **As 7 telas** renderizando o seed via camada de queries (`src/lib/queries.ts`):
  Atendimento (inbox: lista/thread/composer/copiloto + painel perfil/histórico/negócios),
  Contatos, Funil (kanban), Campanhas, Chatbot (canvas de fluxo), Relatórios, Config.
- **Schema Prisma multiempresa** (Org na raiz) + seed com a org demo *Nexo Contábil*
  (equipe, filas, canais, 10 contatos, conversas com mensagens, funil, campanhas,
  fluxo do bot, snapshot de relatório).
- **Docker igual em dev e prod:** compose app/db/migrate, imagem standalone multi-stage,
  `docker compose up -d --build`. Migrate roda `prisma db push` + seed.
- Ações de escrita ainda em **modo demo** (feedback por toast, sem persistir).

## Infra

Slug `navetalks` · app `navetalks-app` na `4050` · banco `navetalks-db` na `5050` ·
`navetalks-migrate`. Chassi e mapa de portas em [[Infra]]. Gotcha de rebuild no mesmo
slug: [[Volume de dev sobrevive entre versões do projeto e traz schema velho]].

## Stack

Next.js (App Router) · React 19 · TypeScript · Tailwind v4 · Postgres · Prisma. Telas
como Server Components lendo o Postgres; ilhas client (inbox, filtros, tema) por cima.

## Próximos passos (linhagem B)

- [ ] Auth/sessão (pode reaproveitar o desenho da linhagem A: sessão opaca, escopo por
      org+setor no servidor).
- [ ] Mutações reais por Server Action (assumir/resolver/enviar, novo contato, mover
      card do funil) trocando os toasts de demo.
- [ ] Multi-tenant real: o rail troca de org de verdade (hoje é visual).
- [ ] Integração WhatsApp por adapter (Baileys primeiro, Cloud API depois) + realtime.

## Aprendizados (viraram notas)

- [[Volume de dev sobrevive entre versões do projeto e traz schema velho]]
- [[Adapter de canal isola o app do provider de mensageria]] (linhagem A)

---

## Linhagem A (remote — app anterior)

Backend-first, verificado ponta a ponta na época:

- Auth completa: sessão opaca no banco, scrypt, funil de escopo por org+setor, gate no
  middleware e no wrapper de API.
- Inbox (caixa compartilhada) com a mecânica de assumir/resolver/enviar.
- Schema multiempresa, migration inicial, seed com empresa demo
  (`admin@navetalks.dev` / `ana@navetalks.dev`).
- Adapter de canal + worker persistente; Baileys real atrás do adapter.
- Docker app/db/migrate/worker.

Decisões que valem revisitar ao evoluir a linhagem B:

- **WhatsApp por adapter, Baileys primeiro** —
  [[Adapter de canal isola o app do provider de mensageria]]. Risco: Baileys é contra o
  ToS e pode banir o número.
- **SaaS multiempresa desde o schema**, escopo clampado no servidor —
  [[Escopo de dado se clampa no servidor, num funil só]].
- **Worker persistente separado do app** (Baileys precisa de processo vivo) —
  [[Polling substitui webhook quando não há IP público]].
- **Caixa compartilhada = recurso do setor**, primeiro atendente assume —
  [[Uma resposta canônica de um grupo é um token compartilhado]].

## Conexões
- Usa: [[Infra]] · [[Sistema de cores e tema do dashboard]] · [[Sidebar em acordeão e layout de módulo]] · [[Volume de dev sobrevive entre versões do projeto e traz schema velho]] · [[Sessão opaca no banco separa autenticação de permissão]] · [[Cravar o seam de permissão antes do login]] · [[Permissão composta por papéis somados, não exceção por usuário]] · [[Escopo de dado se clampa no servidor, num funil só]] · [[Adapter de canal isola o app do provider de mensageria]] · [[Uma resposta canônica de um grupo é um token compartilhado]] · [[Polling substitui webhook quando não há IP público]] · [[Migrations em container próprio no Docker Compose]] · [[Next.js standalone no Docker e o outputFileTracingRoot]]
- Mapa: [[Projetos]]
