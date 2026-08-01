---
tags: [tipo/projeto, projeto/navetalks]
criado: 2026-07-28
status: ativo
codigo_em: ~/Dev/navetalks
---

# navetalks

> Plataforma de **atendimento omnichannel** centrada em WhatsApp: vários atendentes e
> setores respondem os clientes por uma caixa compartilhada, e o cliente recebe tudo no
> WhatsApp dele. É a linhagem Evotalks/Whaticket. **Não é um CRM** — o Eduardo foi claro
> (ago/2026): o foco é atendimento; funil/campanhas e afins são CRM e ficam pra depois.

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
  Atendimento (inbox: lista/thread/composer/copiloto + painel perfil/histórico),
  Contatos, Funil (kanban), Campanhas, Chatbot (canvas de fluxo), Relatórios, Config.
- **Schema Prisma** (Org na raiz, hoje só a Navecon) + seed: equipe com papéis, 5 filas
  com número, 14 contatos-pessoa, 11 conversas com mensagens, funil, campanhas, fluxo do
  bot, snapshot de relatório.
- **Docker igual em dev e prod:** compose app/db/migrate, imagem standalone multi-stage,
  `docker compose up -d --build`. Migrate roda `prisma db push` + seed.
- Ações de escrita ainda em **modo demo** (feedback por toast, sem persistir).

## Modelo de domínio (linhagem B)

O modelo passou por idas e voltas com o Eduardo (ago/2026). O estado **atual** é este —
as versões anteriores (Empresa 1—* Contato; thread privada por atendente) foram
descartadas por ele, ficam só como histórico:

- **Contato = pessoa.** Nome livre (quem cadastra decide, tipo "Julia Luiza - RA
  Transportes"), telefone, etiquetas. **Sem entidade Empresa por ora** — religar contato a
  uma empresa é backend futuro. Não modelar empresa antes da hora.
- **Um único escritório.** Hoje só existe a **Navecon**. Tirei o "tenant rail" de trocar
  de conta: o app inteiro É a lógica do escritório. Vender o CRM pra outros escritórios
  (multi-tenant de verdade) é decisão de backend lá na frente, não UI agora.
- **Marca:** o produto/sistema é **Navetalks**; o escritório (org) é **Navecon**. Aba do
  browser = Navetalks.
- **Papéis:** Admin, Gestor, Funcionário.
- **Fila = setor com um número de WhatsApp.** Usuários pertencem a filas. A conversa entra
  por um número → fica na fila daquele número. **O escopo de acesso segue o número de
  entrada:** Funcionário só vê as conversas das suas filas; um do Contábil NÃO vê quem
  mandou pro número do Fiscal. Admin/Gestor veem todas. Aplicação de
  [[Escopo de dado se clampa no servidor, num funil só]] — a chave do escopo aqui é a fila
  (canal de entrada), não o dono do registro.

Sem auth ainda: o "usuário logado" vem de um cookie **"ver como" (demo)** que troca o
usuário pra exibir o escopo na prática. É o ponto onde a sessão real entra depois.

Reflexo na UI (estado atual):
- **É plataforma de atendimento, não CRM.** Funil e Campanhas (CRM) saíram do menu —
  viram `HIDDEN_MODULES` no `catalog.tsx` (rota viva, fora da navegação). Reativar é só
  mover de volta pra `MODULES`.
- **Atendimento** filtra por **estado** (Todas / Em atendimento / **Fila de espera** =
  conversas que ninguém puxou) — **sem "Resolvida"** (o Eduardo cortou o conceito). Mais
  filtros: busca na lista, por fila e por responsável (minhas / sem dono).
- **Contatos** é lista de pessoas com **linhas expansíveis** e filtro por etiqueta.
- **Etiquetas = rótulos operacionais** (Cliente ouro, VIP, Inadimplente, Lead) que a
  equipe cola pra segmentar/filtrar — não é empresa nem cargo.
- **Configurações** tem filas/números com membros + equipe com papéis; **Mensagens
  prontas** viraram um modal de respostas salvas (busca + categorias, preenche {{nome}}).

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
- [ ] Multi-escritório de verdade (se virar produto pra vender): separação por org no
      backend. Hoje é escritório único (Navecon), sem UI de troca.
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
