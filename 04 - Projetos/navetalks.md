---
tags: [tipo/projeto, projeto/navetalks]
criado: 2026-07-28
atualizado: 2026-08-08
status: ativo
codigo_em: ~/Dev/navetalks
---

# navetalks

> **Retomado (ago/2026), agora standalone.** A ideia de fundir atendimento + CRM
> num produto único (o Navehub) foi deixada de lado: o Eduardo voltou ao navetalks
> como **plataforma de conversas própria e NÃO CRM**. Começou de novo do zero
> (**linhagem C**) — o código da linhagem B foi apagado. A cara mudou: saiu o
> "Aurora Glass" roxo e o experimento neutro; entrou uma estética **macOS / Liquid
> Glass** com fonte **Geist**. Estado atual da linhagem C logo abaixo; A e B ficam
> como histórico da linhagem no fim.

> Plataforma de **atendimento omnichannel** centrada em WhatsApp: vários atendentes e
> setores respondem os clientes por uma caixa compartilhada, e o cliente recebe tudo no
> WhatsApp dele. É a linhagem Evotalks/Whaticket. **Não é um CRM** — o Eduardo foi claro
> (ago/2026): o foco é atendimento; funil/campanhas e afins são CRM e ficam pra depois.

Código em: `~/Dev/navetalks` · remote `git@github.com:edulanzarin/navetalks.git`
(remote ainda tem a linhagem A; push local sobrescreve — decisão do Eduardo, não automática).

## Estado atual (linhagem C — rebuild macOS/Liquid Glass, ago/2026)

Greenfield de novo, verificado (typecheck + build de produção limpos + as 6 rotas
respondendo 200 em SSR contra o Postgres seedado; escopo por papel conferido — o
Funcionário do Fiscal só enxerga as conversas das filas dele).

- **Estética macOS / Liquid Glass** (nova, decisão do Eduardo): janela flutuante com
  wallpaper mesh nas frestas pro vidro puxar cor. **Três materiais**: `.glass` (chrome:
  rail e topbar), `.glass-card` (card translúcido com curvatura/brilho no topo) e
  `.glass-over` (overlay flutuante). Fonte **Geist** (não mais Inter), raios grandes,
  scrollbar discreta. Aplica [[Sistema de cores e tema do dashboard]] com token por tema.
  - **Regra que o Eduardo cravou aqui (ago/2026):** container pode ser vidro, mas
    **onde há texto o fundo tem que ser sólido** — o overlay do dropdown estava deixando
    o texto da lista atrás vazar. Overlay ficou quase opaco (~0.99). É o 2º caso concreto
    de [[Vidro flutuante precisa de superfície mais opaca que a chrome]].
  - **Contraste verificado (WCAG 4.5:1)**: introduzido `--accent-solid` (violeta mais
    fundo) só pro **fill com texto branco** (balão OUT, botão) — o violeta de acento cru
    reprovava branco por cima. [[Cor de marca precisa de variante acessível por tema]].
- **Modelo de domínio herdado da linhagem B** (segue valendo, ver seção abaixo): Contato =
  pessoa; **Fila = a fila de espera de um número de WhatsApp**; papéis Admin/Gestor/
  Funcionário; conversa é **fio contínuo, não ticket**; transferência é pra colaborador
  com acesso à mesma fila. Escopo clampado no servidor —
  [[Escopo de dado se clampa no servidor, num funil só]].
- **Conversas (inbox) de pé**: lista com filtros na URL (estado/fila/responsável/busca),
  abas ícone+contagem, fio com balões e recibos de status, composer real. Escritas por
  **Server Action** escopada por papel/fila: enviar (assume sem roubar de quem já atende),
  assumir, devolver, finalizar. **Entrega passa por um seam** (`src/lib/whatsapp.ts`
  `deliverText`): sem conector a mensagem fica `pendente` e sai quando o número conectar —
  [[Persistir a mensagem não espera a entrega, a entrega é status]].
- **Os 4 módulos de pé** (ago/2026): além de Conversas — **Contatos** (diretório + ficha
  em modal com identidade, dados enxutos e timeline de anotações; a mesma ficha abre pela
  inbox no clique do contato), **Configurações** (seções deep-linkáveis por `?s=`: Filas &
  Números com membros, Equipe com papéis/convite, Mensagens prontas) e **Relatórios**
  (KPIs escopados por papel). Mensagens prontas viram picker no composer com `{{nome}}`.
- **Sem auth ainda**: usuário logado vem do cookie **"ver como"** (demo) em
  `current-user.ts`, que troca o ator pra demonstrar o escopo (as escritas de config são
  gated por papel no servidor).
- **Stack/infra**: Next 15 (App Router) · React 19 · TS · Tailwind v4 · Postgres · Prisma.
  Chassi `navetalks` app `4050` / banco `5050`, compose db+migrate+app, Dockerfile
  standalone Prisma-aware. 5 commits temáticos, sem remote configurado local.

### Próximos passos (linhagem C)
- [x] Contatos, Configurações (filas/números, equipe, mensagens prontas) e Relatórios —
      feitos (ago/2026).
- [ ] Auth/sessão real trocando o cookie "ver como".
- [ ] Adapter de WhatsApp: **entrada** (webhook/ingestão + recibos de status pelo
      `externalId`) e conexão real do número; realtime na inbox.
      [[Adapter de canal isola o app do provider de mensageria]].
- [ ] Anexo/áudio no composer e importar CSV (dependem do conector/armazenamento).

## Linhagens anteriores (histórico)

Existem três bases com este nome ao longo do tempo — não confundir:

- **Remote / GitHub (linhagem A):** o app anterior, backend-first — auth de sessão
  opaca, escopo por org+setor, inbox, worker persistente e integração Baileys real.
  Descrito na seção "Linhagem A" abaixo. Ainda vive no remote (`main`, ~10 commits).
- **Local (linhagem B):** rebuild greenfield de ago/2026 a partir de um esboço de
  design (design-first, 7 telas, sem auth). Chegou a ganhar persistência real e passou
  por um experimento de base **neutra**. **Foi apagado** e substituído pela linhagem C —
  fica aqui só como registro das decisões de modelagem, que a C herdou.

## Linhagem B (histórico — greenfield anterior, design-first)

Base de pé e verificada (build de produção limpo + as 7 rotas respondendo 200 em SSR
contra o Postgres seedado):

- **Design "Aurora Glass" (roxo):** tema **roxo/violeta**, modo noturno mais preto (quase
  carvão), **fonte Inter**, muito **vidro** (blur + transparência + brilho superior) e
  animações de entrada. Tokens semânticos claro/escuro no Tailwind v4, anti-flash. Kit de
  primitivos em `src/components/ui`. Aplica [[Sistema de cores e tema do dashboard]].
  O vidro tem **dois níveis**: chrome arejada (`.glass`/`.glass-2`) e overlay opaco e
  legível (`.glass-over`, token `--surface-over`) pra tudo que flutua sobre o conteúdo —
  ver [[Vidro flutuante precisa de superfície mais opaca que a chrome]] (ago/2026, depois
  de o menu "ver como" ficar ilegível de tão translúcido).
  A **sidebar é sempre um rail de ícones com tooltips** (o Eduardo achou melhor, e "fica
  menos apertado"); na inbox dá pra ocultar a lista de conversas.
- **Dashboard é a landing** (`/` → `/dashboard`), **escopado por papel**: FUNCIONARIO vê
  sobre ele, GESTOR sobre suas filas, ADMIN sobre o escritório. KPIs (atendimentos, em
  atendimento, fila de espera, tempo de 1ª resposta, SLA, bot), volume/hora, distribuição
  por fila, ranking, SLA, CSAT e atividade recente.
- **As 8 telas** via camada de queries (`src/lib/queries.ts`): Dashboard, Atendimento
  (inbox), Contatos, Chatbot, Relatórios, Config (Funil e Campanhas existem mas ocultos —
  são CRM).
- **Schema Prisma** (Org na raiz, hoje só a Navecon). **Contact é objeto CORE + poucos campos
  próprios** (id, nome, telefone, email, **company** texto) — o CRM
  pesado (empresa como entidade, honorário, etiquetas) vira tabela satélite, sem inchar
  Contact: [[Entidade núcleo cresce por tabela satélite, não por coluna]]. Duas decisões de
  fronteira (ago/2026): (a) **email entrou direto na Contact**, não como satélite — a regra do
  satélite vale pra *atributo de CRM*, não pra **canal de contato** (email é irmão do telefone,
  dado core); (b) **anotações viraram satélite `ContactNote`** (log datado autor+data), que é o
  2º caso concreto do princípio do satélite. `company` é texto por ora, gancho pra virar
  relação no CRM. Seed: equipe (20 usuários), 5 filas com número, 14 contatos (com empresa e
  anotações), 11 conversas, fluxo do bot, snapshot de relatório.
- **Docker igual em dev e prod:** compose app/db/migrate, imagem standalone multi-stage,
  `docker compose up -d --build`. Migrate roda `prisma db push` + seed.
- **Escritas internas agora persistem** (ago/2026), não são mais demo: Server Actions em
  `src/lib/actions/` (um arquivo por domínio), escopadas por papel/fila e com
  `revalidatePath`. Inbox (assumir/devolver/resolver/reabrir/transferir + marcar lida),
  Contatos (criar/editar), Config (convidar usuário, registrar fila+número, gerenciar
  membros), e — nos módulos CRM ocultos — Funil (criar/editar/remover negócio) e
  Campanhas (criar rascunho/template). A escrita reusa o mesmo funil de escopo da leitura
  (`loadConv` espelha `filaScope`): [[Escopo de dado se clampa no servidor, num funil só]].
- **Envio de mensagem de texto agora é real** (ago/2026): o composer parou de dar toast
  "demo" — a Server Action `sendTextMessage` persiste a resposta (`OUT`), atualiza resumo
  e último-em, zera não lidas e **assume o atendimento** (tira da fila de espera) sem
  roubar de quem já atende; bloqueia se a conversa está finalizada; reusa o mesmo
  `loadConv` escopado por fila. A **entrega ao cliente** passa por um seam em
  `src/lib/whatsapp/transport.ts` (`deliverText`): com conector configurado (env
  `WHATSAPP_TOKEN`+`WHATSAPP_PHONE_ID`) e número `CONECTADO` envia pela Cloud API e grava
  o `wamid`; sem conector a mensagem fica **`pendente`** e sai quando o número conectar —
  nada fingido. É a aplicação de
  [[Persistir a mensagem não espera a entrega, a entrega é status]]. Campo novo
  `Message.externalId` (wamid) pra correlacionar os webhooks de status. UI otimista com
  `useOptimistic` (bolha "enviando" na hora, reconciliada no revalidate) e recibos por
  status no balão (relógio/1 tique/2 tiques/2 azuis).
- **Segue demo de propósito só o resto que depende do conector externo** (decisão do
  Eduardo, ago/2026): anexo e áudio no composer, importar CSV, e a **conexão real do
  número** (o registro interno entra como `DESCONECTADO`; falta o adapter que liga de
  fato). Anexo/áudio hoje avisam "chega com o conector do WhatsApp". Conector: pensar via
  [[Adapter de canal isola o app do provider de mensageria]].
- **Nuance CRM:** as escritas de Funil e Campanhas foram feitas sobre os `HIDDEN_MODULES`
  (rota viva, fora do menu — são CRM, parkados). Estão prontas e testadas; pra aparecerem,
  é só mover de `HIDDEN_MODULES` pra `MODULES` no `catalog.tsx`.

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
- **Fila = a fila de espera de um número de WhatsApp** (não "setor com número"). O Eduardo
  corrigiu (ago/2026): nem todo setor tem número próprio — pode haver **um número geral do
  escritório e um só do DP**, por exemplo. Quem messageia um número entra na fila daquele
  número. Usuários têm acesso a filas. **O escopo segue o número de entrada:** Funcionário
  só vê/atende as conversas das suas filas; quem só tem o número do DP não vê quem mandou
  pro número geral. Admin/Gestor veem todas. Aplicação de
  [[Escopo de dado se clampa no servidor, num funil só]] — a chave do escopo é a fila
  (canal de entrada), não o dono do registro.
- **Transferência é para colaborador, não para outra fila.** A conversa entrou por um
  número e **fica nele** — não faz sentido "mudar de fila/número" (o cliente chamou aquele
  número). Transferir = passar o **responsável** para outro colaborador que **tenha acesso
  à mesma fila**; não dá pra transferir pra quem não tem acesso. No código: `transferToUser`
  valida a elegibilidade do destino contra `conv.fila`. "Devolver à fila" é o inverso:
  solta o dono e a conversa volta pra espera daquele número, pra outro com acesso puxar.

Sem auth ainda: o "usuário logado" vem de um cookie **"ver como" (demo)** que troca o
usuário pra exibir o escopo na prática. É o ponto onde a sessão real entra depois.

Reflexo na UI (estado atual):
- **CORREÇÃO DE DIREÇÃO (ago/2026, Eduardo enfático): é CONVERSA, não "atendimento".** O
  modelo mental é **WhatsApp**, não ticket/chamado: um fio contínuo por contato onde os dois
  lados falam quando quiserem — "eu posso chamar o cliente a qualquer hora assim como ele
  pode". Nada de "atendimento iniciado/finalizado", nada de **protocolo**. Já aplicado no
  modal do contato e no cabeçalho da inbox. **Pendente (dps pensamos):** o resto da UI ainda
  fala "atendimento" — botão **Finalizar**, **fila de espera**, status `RESOLVIDA`, KPIs de
  atendimento. Esses termos abaixo refletem o estado ANTES da correção; a varredura ampla
  pra trocar a moldura toda ainda não foi feita. Não reintroduzir a moldura de ticket.
- **[[Interface enxuta e compacta, sem desperdício de espaço]]** (preferência dele, ago/2026):
  manter o glass bonito/moderno, mas **compacto** — "coisas ocupando muito espaço fica ruim
  pra um sistema desse". Densidade alta (tipos menores, paddings curtos) pra caber mais
  informação; o modal pode ser largo/alto, o que não pode é desperdiçar espaço.
- **É plataforma de atendimento, não CRM.** Funil e Campanhas (CRM) saíram do menu —
  viram `HIDDEN_MODULES` no `catalog.tsx` (rota viva, fora da navegação). Reativar é só
  mover de volta pra `MODULES`.
- **Atendimento** filtra por **estado** (Todas / Em atendimento / **Fila de espera** =
  conversas que ninguém puxou) — **sem aba "Resolvida"**. As abas são **ícone + contagem**
  (caixa/mensagem/relógio), não texto, pra não quebrar layout com muitos dígitos. Mais
  filtros: busca na lista, por fila e por responsável (minhas / sem dono). Os dropdowns são
  `Select` de **vidro custom** (o `<select>` nativo tinha popup feio) — ver
  [[Controles de filtro do dashboard]]. **Sem botão "limpar"**: limpar é reselecionar
  Todas/Qualquer um.
- **Finalizar (não "resolver"):** o botão do cabeçalho da conversa é **Finalizar** — o
  Eduardo achou "resolver" sem sentido (ago/2026). Finalizar **encerra o atendimento e tira
  a conversa da inbox**; o contato fica só no diretório de **Contatos**. No dado é o status
  terminal `RESOLVIDA` (zera não lidas e SLA), e a inbox passou a filtrar `status != RESOLVIDA`.
  Reabrir virá pelo contato. A **mensagem padrão de finalização** (enviada ao cliente) será
  **definida nas Configurações** — envio depende do conector de WhatsApp, então fica pro fim.
- **Configurações vai ser robusto** (aviso do Eduardo, ago/2026): não é a tela simples de
  hoje; vira o centro de definição do produto (mensagens padrão, filas/números, equipe,
  regras). Tratar como área que vai crescer, não como formulário fixo.
- **A lista de conversas recolhe pra um rail de ~72px** (só fotos + badge de não lidas +
  barra de acento rente à esquerda); o toggle mora na própria lista. Recolher **não some
  com tudo** — vira índice visual.
- **SLA é sinal por conversa, não contador agregado** (ago/2026): saiu a linha "X fora do
  SLA" e cada conversa mostra a **hora da última mensagem + relógio colorido** — vermelho
  (atrasado, `slaBreach`), âmbar (aguardando resposta = na fila ou com não lidas), neutro
  (em dia). SLA é **core de atendimento** (tempo de resposta, o mesmo do TMR no topo), não
  CRM. O âmbar é derivado do que existe hoje (`slaBreach`+status+unread) em `slaSignal()`;
  com campo de prazo real, é só trocar essa função.
- **Ficha do contato é UM modal só**, o mesmo na inbox (clique na foto) e no diretório de
  Contatos — a linha que expandia no diretório virou clique que abre a ficha (ago/2026). O
  Eduardo cortou a divergência: mesma casca em todo lugar. Componente
  `components/inbox/contact-modal.tsx`, `size lg`. Topo: **só foto, nome e número**.
- **A ficha é IDENTIDADE + DADOS ENXUTOS + TIMELINE DE ANOTAÇÕES** (ago/2026, muitas
  iterações — passou por "perfil+histórico de atendimento", "só contato", "2 colunas com
  caixa fixa" (o Eduardo achou feio e desproporcional) até chegar no minimalista). Aqui é a
  parte de conversa (WhatsApp): o "histórico de conversa" JÁ são as mensagens no fio, então
  nada de timeline de "conversa iniciada" na ficha. Forma final (coluna única, `size lg`):
  - **Topo**: foto, nome, número. O **número mora só aqui** — não repetir num campo embaixo
    (o Eduardo pegou essa redundância).
  - **Dados = uma linha de ícones**: email (`IconMail`) e empresa (`IconBuilding`), só quando
    existem. `Contact.company` é texto por ora — gancho pra ligar contato→entidade Company no
    CRM. **Características/`traits` foi removido** (adicionado e cortado no mesmo dia: o que é
    característica cabe numa anotação).
  - **Anotações = timeline minimalista** (ícone + risco vertical, sem cards pesados): autor ·
    data + texto, da mais recente à mais antiga. Adicionar é **expansível** — um "+ Adicionar"
    que abre a caixa só quando precisa (⌘/Ctrl+Enter envia, Esc cancela), nada de caixa fixa
    ocupando espaço. Log em tabela satélite **`ContactNote`** (contactId, author, body,
    createdAt) — 2º caso concreto de [[Entidade núcleo cresce por tabela satélite, não por coluna]].
    Actions `fetchContactNotes`/`addContactNote`; o antigo `Contact.note` único foi removido.
  - Padrão de forma que ele reforçou aqui (é [[Interface enxuta e compacta, sem desperdício de espaço]]):
    **ícones e elementos expansíveis no lugar de caixas fixas e cards grandes**.
  - Fronteira nítida: **este app = conversa** (contatos + mensagens + robustez na fila);
    o **CRM** (empresa como entidade, vários contatos por empresa, histórico rico) é
    produto/menu separado e futuro. Empresa e anotações aqui são a base mínima que já encaixa
    nesse CRM sem retrabalho. `Conversation.createdAt`/`resolvedAt` seguem no banco como
    metadata dormente.
- **"Protocolo" saiu da UI** (ago/2026): era o número do atendimento/ticket — exatamente a
  moldura de "atendimento" que o Eduardo rejeitou (ver a correção de direção acima). Tirado
  do modal e do cabeçalho da conversa na inbox. O campo `Conversation.protocol` segue no
  banco, sem uso na tela.
- **Thread mostra a conversa real do contato** (ago/2026): o seed reusava uma mensagem
  `generic` em vários contatos, o que dava sensação de "conversa fixa" — agora cada contato
  tem mensagens próprias. Também saiu o banner de data fixo ("01 ago 2026") da thread.
- **Copiloto (sugestão de IA) removido** do composer (ago/2026) — a prop `aiOn` saiu ponta
  a ponta. IA de resposta não faz parte do escopo agora.
- **Contatos** é só pessoa: nome + telefone + nota interna, em **linhas expansíveis**.
  **Sem etiquetas** — o Eduardo considerou etiqueta/tags coisa de CRM (ago/2026) e mandou
  tirar; "é pra ter só o contato e atendimento". Volta se/quando o CRM entrar.
- **Layout colapsável** (pedido dele: "fica sempre tudo apertado"): a sidebar recolhe pra
  rail de ícones (persiste no localStorage) e, na inbox, dá pra ocultar a lista de
  conversas pelo botão no cabeçalho. (O painel de contexto fixo saiu — virou modal da
  ficha do contato, ver acima.)
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
      org+setor no servidor). Hoje ainda é o cookie "ver como" em `current-user.ts`.
- [x] Mutações reais por Server Action trocando os toasts de demo — **feito** (ago/2026),
      menos o que depende do WhatsApp (enviar/anexo/áudio/conexão do número), que ficou de
      propósito pro fim. Ver seção de estado.
- [ ] Multi-escritório de verdade (se virar produto pra vender): separação por org no
      backend. Hoje é escritório único (Navecon), sem UI de troca.
- [ ] Integração WhatsApp por adapter (Baileys primeiro, Cloud API depois) + realtime.
      O seam de **saída** já existe (`deliverText`); falta a **entrada** (webhook/ingestão
      de mensagens recebidas + recibos de status que sobem `pendente→enviado→entregue→lida`
      pelo `externalId`) e a conexão real do número.
- [ ] Anexo/áudio no composer e importar CSV — dependem do conector/armazenamento, seguem
      de propósito pro fim.

## Aprendizados (viraram notas)

- [[Volume de dev sobrevive entre versões do projeto e traz schema velho]]
- [[Adapter de canal isola o app do provider de mensageria]] (linhagem A)
- [[Persistir a mensagem não espera a entrega, a entrega é status]] (envio real, ago/2026)

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
- Usa: [[Infra]] · [[Sistema de cores e tema do dashboard]] · [[Sidebar em acordeão e layout de módulo]] · [[Volume de dev sobrevive entre versões do projeto e traz schema velho]] · [[Sessão opaca no banco separa autenticação de permissão]] · [[Cravar o seam de permissão antes do login]] · [[Permissão composta por papéis somados, não exceção por usuário]] · [[Escopo de dado se clampa no servidor, num funil só]] · [[Adapter de canal isola o app do provider de mensageria]] · [[Persistir a mensagem não espera a entrega, a entrega é status]] · [[Uma resposta canônica de um grupo é um token compartilhado]] · [[Polling substitui webhook quando não há IP público]] · [[Migrations em container próprio no Docker Compose]] · [[Next.js standalone no Docker e o outputFileTracingRoot]]
- Mapa: [[Projetos]]
