---
tags: [tipo/projeto, projeto/piwdex]
criado: 2026-08-15
status: ativo
codigo_em: ~/Dev/piwdex
---

# piwdex

> Dex e ferramentas completas para o jogo **Poke Idle World** (poke.idleworld.online).
> Faz o que o piwtools.com.br faz e vai alem: chance real de cada drop, indice reverso
> "onde dropa cada item", localizacao de hunt por pokemon, cadeia evolutiva e fraquezas.
> Dominio: piwdex.com.br. Escopo pedido pelo Eduardo: dex + calculadoras + companion
> logado — "o mais completo possivel".

Codigo em: `~/Dev/piwdex`

## Estado atual

**Camada 1 (base publica) entregue e verificada em build de producao.** Pipeline de
ingestao (`scripts/ingest.mjs`) baixa os tres JSON publicos do jogo direto da
fonte-mestra (ver [[Poke Idle World - endpoints publicos de dados]]), valida
integridade (todo loot bate com um item) e grava o snapshot versionado em
`src/data/piwdex.json`. As derivacoes NAO entram no snapshot — vivem em
`src/lib/data.ts` pra o snapshot ser diffavel contra o jogo: indice reverso de drop
("onde dropa X", ordenado por maior taxa), localizacao por `looktype`, cadeia
evolutiva. Efetividade de tipo (fraquezas) computada de tabela propria.

UI em Next 16 (App Router): home, Pokedex com busca/filtro por tipo, ficha completa
(stats, fraquezas, evolucao, drops com % real, onde cacar, moves), paginas de item com
indice reverso + preco NPC, e **Calculadora de IV/qualidade/poder** (formula do jogo
verificada — ver [[Poke Idle World - endpoints publicos de dados]]). Estima IVs dos
stats atuais e projeta stats/poder em qualquer nivel.

**Visual refeito (2a passada):** a 1a versao ficou "dashboard" e o Eduardo rejeitou.
Agora e pixel/neon dark estilo o jogo/piwtools: fontes Press Start 2P + JetBrains Mono,
ceu estrelado em CSS, cards com borda neon, pokebola SVG em pixel (logo + spinner de
loading), skeletons e sprite com loading. Sem emoji, selects estilizados. Bug corrigido:
sprite do SSR que terminava de carregar antes do onLoad travava no placeholder (checa
img.complete no mount).

**Raridade:** removida da especie — no jogo e por individuo (ver a formula na nota de
referencia). O lugar dela e a calculadora, nao um selo fixo.

**Catalogo ao vivo (ISR):** trocou o import estatico do snapshot por fetch do jogo com
`next: { revalidate: 3600 }` — o catalogo se atualiza sozinho de hora em hora, sem
redeploy. Se a fonte falhar, cai no snapshot versionado (fallback, site nao quebra).
`data.ts` virou `getData()` async memoizado por request (React `cache`). O **preco de
mercado de jogadores continua fora** — nao e questao de snapshot x live, so existe atras
do login (camada 3). Decisao do Eduardo (ago/2026): catalogo auto-atualizavel primeiro,
mercado logado depois. UI polida: moeda de ouro nos precos, combobox de pokemon com
sprites no lugar do select nativo, pokebola melhor + loaders animados (pikachu).

**Imersao pixel (3a passada, ago/2026):** o Eduardo pediu "nivel de jogo". Entregue:
- **Pokedex que abre na tela** — o /dex e envolto num aparelho Pokedex (device com
  lente/leds/telinha) e toca uma animacao de abertura em tela cheia (portas vermelhas
  se separam + boot + power-on) antes de revelar o grid dentro do frame. Peca central.
- **Icones de tipo em pixel** (8x8 SVG inline, currentColor) nos badges, no filtro e na
  tabela de fraquezas — que virou pills com **multiplicador** (x2, 1/2, 1/4), corrigindo
  o badge que quebrava desalinhado.
- **Ficha mais viva**: sprite anima no hover (gif gen5) + toggle shiny, papeis do bicho
  (atacante/tanque/veloz), mini-stats, golpe mais forte, e **Vende por vs Loja NPC**
  desambiguados (ver a nota de referencia — sao campos diferentes no schema).
- **Fundo pixel 4k**: cena de rota noturna gerada por script Pillow (`scripts/gen-bg.py`),
  fixa sob scrim. **Scroll reveal** (IntersectionObserver) nas secoes.
- Home enxuta: sprite do aparelho foi pro card, hero sem botao duplicado.
Principio herdado: aparencia pixel imersiva e por-projeto, nao dialeto de dashboard
(ver [[Estetica e por projeto, principio de design e que se reusa]]).

**Idioma BR/EN/ES + refinamentos (4a passada, ago/2026):** i18n **client-side com
toggle na mesma URL** (preferencia no localStorage) — escolha do Eduardo pra manter o
site 100% estatico (SSG), sem rota por idioma. Provider React + `<T>`/`<TB>` pra texto
em server component e `useT`/`useTypeLabel` nos clients; dicionario pt/en/es completo,
nomes de tipo traduzidos, mas **nome de pokemon/item fica em ingles** (o jogo e em
ingles). Bandeiras em pixel (SVG). Outros ajustes: barras de stat viraram gauge pixel;
ataque separa **STAB 1.5x** (tipos do proprio bicho) de "Forte contra" (cobertura x2);
coluna CD removida (cooldownMs base engana — ver nota de referencia); a ficha do pokemon
tambem abre dentro do aparelho Pokedex; loading = pokebola girando; itens clicaveis com
hover; scrollbars no tema.

**Planejador de Breeding (5a passada, ago/2026):** o Eduardo pediu a parte de breeding.
A 1a tentativa **clonou a pagina do piwtools** (cards Confirmadas/Provisorias/Futuras,
ordem das secoes, textos de estrategia) e ele rejeitou na hora — a regra e inspirar na
**mecanica/dado** do jogo, nunca na interface do concorrente (ver
[[Inspiração é na mecânica e no dado, não na interface do concorrente]]). Refeito do zero
como ferramenta propria (`/breed`):
- **Dois paineis de pais** editaveis inline, com salvar/carregar de uma colecao local
  (`localStorage`) — a simulacao nunca consome nada.
- **O ovo** (resultado reativo): IVs herdados, distribuicao de Quality e, o diferencial,
  os **stats reais do ovo** no nivel escolhido, ligando na engine `stats.ts`
  (`projectAll`) — o piwtools nao projeta stats de batalha no breeding.
- **Planejador de Quality** (feature nossa): do Q atual ate o alvo, quantos breeds pela
  media, custo estimado (R$/Stones/Pheromone) e os dois riscos do jogo — passar de 0.150
  num passo (orfa o filho) e estourar o teto 2.600. Free vs Pheromone lado a lado.
- Regras do jogo num strip compacto colapsavel, na nossa voz, com o provisorio marcado.
Motor puro em `src/lib/breeding.ts` (`projectEgg` + `planQuality`); regras da especie sao
**camada curada** — nao vem no JSON (ver [[Poke Idle World - regras de breeding]]). Media
de ganho bate exato (Free 0.0096 / Pheromone 0.1875).

Verificado: `tsc` limpo, `next build` gera 822 paginas estaticas (incl. `/breed`
prerenderizada), e a calculadora e as medias de breeding batem exato com o jogo. Sem
remote ainda (so commit local; branch por tipo -> `master` fast-forward).

**Ajustes (ago/2026):**
- **Aviso de nivel baixo na calculadora/Eevee**: estimar IV so e confiavel em nivel
  ~20+ (ver [[Poke Idle World - endpoints publicos de dados]]). Abaixo disso os stats
  arredondados invertem pra IV errado (Charizard lvl1: jogo 138, inversao ~122).
- **Card seu-vs-perfeito no Projetar** da calculadora (o Lab da Eevee ja tinha na aba
  Comparar), com os stats e o Poder projetados no nivel alvo.
- **Engine da rota de hunt refeita**: estava sugerindo alvos 1x. Corrigido em
  `src/lib/combat.ts` — (a) usa o melhor golpe CONTRA o alvo (dano efetivo = poder *
  STAB * efetividade), nao o de maior nivel de aprendizado (que caia num golpe Normal
  1x); (b) sem janela de nivel: rankeia todos os alvos que voce encara (limitados pelo
  Power) por XP x efetividade, seguro antes de arriscado. Agora prioriza super-efetivos
  (Tyranitar/Aggron 5.5x) como o jogo espera.
- **Device frame por ferramenta**: o Eduardo quis tudo "dentro de um container" como o
  aparelho Pokedex, colorido. Generalizei o `.pkdx-*` num `.device-*` tematizavel (cor
  vem de `--dev` via color-mix) + `ToolFrame`; calc/itens/hunt/breed/eevee agora vivem
  numa casca neon com lente/label/icone pixel da cor da ferramenta e titulo no acento.
  O /dex mantem o aparelho Pokedex vermelho. Primeira passada do "mais cor/icone".

**Camada 3 iniciada — companion logado (6a passada, ago/2026):** conectar a conta do jogo
e ver a colecao. Descoberta que definiu tudo: o login do jogo exige **captcha anti-robo**,
entao proxiar email/senha no servidor NAO funciona — o companion loga **por token** (o
jogador cola o `pokeweb:tokens`), que e o modelo mais seguro (senha nunca sai do jogo). Ver
o contrato em [[Poke Idle World - endpoints publicos de dados]].
- `game-auth.ts`: sessao criptografada (AES-256-GCM) em cookie httpOnly, parse do token,
  `gameFetch` com refresh automatico em 401.
- Rotas `/api/{connect,disconnect,collection}` (Next route handlers, runtime node). A
  `collection` normaliza `characters/me` em pokemons com Power/IV pela `stats.ts`, e
  devolve o RAW pra finalizar o mapeamento de campos com um token real.
- `/conta`: conectar (instrucoes, sem senha) + colecao + visor de dado bruto.
- **Ainda sem Postgres**: roda so com route handlers + cookie (backend so onde precisa).

**Consultor de mercado + conta real (7a passada, ago/2026):** com um token real (o Eduardo
colou o dele), cravei os formatos e consertei tudo. `characters/me` e so o TREINADOR (por
isso a 1a colecao trazia o "Zashz"). O util esta em `/api/game/{profile,all-pokes,market,
balls}` — formatos na nota de referencia. Destaque: os anuncios de POKEMON do `/api/game/
market` ja vem com `ivTotal/quality/power/stats/price/currency` prontos, e `/api/game/balls`
tem `catchRate` real por bola.
- Conta agora mostra **perfil real** (level, ouro, diamantes, capturas) + **pokedex** (all-pokes).
- **Consultor de mercado** (`/api/market` + UI): "melhor <especie> ate X ouro / Y diamantes",
  ordena por Power / preco / custo-beneficio; cache de 60s do mercado (~6MB). Era o pedido
  central do Eduardo. Validado sobre o mercado real (10k pokemons a venda).
- Guardado pra depois: `catchRate` das bolas destrava a captura na rota de hunt.

Testado com token real (leitura). Proximo: captura na rota (usando catchRate) e realtime WS.

**Camada 4 — login do site + banco (8a passada, ago/2026):** o Eduardo apontou a
incoerencia: conectar a conta do jogo sem estar logado no piwdex nao faz sentido. Agora
o companion tem dono. Entregue:
- **Chassi Postgres** (slug `piwdex`, par 5070) em **SQL puro, sem ORM**, no padrao do
  Navehub: `docker-compose` + `Dockerfile` multi-stage + runner de migration em `db/`
  (ver [[Runner de migration em SQL puro dispensa o CLI do ORM]] e
  [[Migrations em container próprio no Docker Compose]]). Migration 001: `users`,
  `oauth_accounts`, `game_links`.
- **Login Auth.js v5** com sessao JWT por cima do `pg` cru (sem adapter) — **so
  email/senha** (bcryptjs); o Eduardo decidiu cortar o Google ("melhor sem"), migration
  002 dropou a tabela `oauth_accounts`. Padrao capturado em
  [[Auth.js sem adapter, a sessão JWT resolve o usuário no seu próprio SQL]]. Telas
  `/entrar` e `/criar-conta`, login/logout no header, gate em `/conta`. Cookies
  namespaced `piwdex.*` pra nao colidir com outros apps next-auth no `localhost`
  (o cookie ignora a porta — senao dava `no matching decryption secret`).
- **Vinculo da conta do jogo saiu do cookie AES e foi pro banco** (`game_links`), preso
  ao usuario logado: tokens cifrados, refresh persistido, `status='expired'` quando o
  refresh falha -> a UI pede reconexao ("tenta manter, senao reconecta", pedido do
  Eduardo). `connect`/`collection`/`disconnect`/`market` operam por usuario.
- **Mercado saiu do /conta gratis** e virou `<MarketAdvisor>` reusavel, pronto pra area VIP.
Testado ponta a ponta (build de producao numa porta separada): cadastro/login por
credenciais emite sessao com id do banco, `/conta` abre logado e redireciona deslogado,
e `/api/collection` separa "nao logado" de "logado sem jogo vinculado".

**Time ativo via WebSocket (9a passada, ago/2026):** o unico pedaco da conta que a REST
nao da — os pokemons individuais ativos (time + colecao, com IV/quality/power por
individuo). Cravei o protocolo do WS conectando com token real: `wss://.../ws<SHARD>?
token=`, shard por conta (Zashz = ws47), shard errado fecha com `4003 wrong-shard`. No
connect o server empurra `{type:"pokes", list:[225 individuos]}`; time = `team:true` por
`slot`, `leader` marcado. Shard descoberto por sondagem paralela com early-exit (~300ms) e
cacheado em `game_links.shard` — padrao em [[Descobrir o shard por sondagem paralela com early-exit e cachear]].
`game-ws.ts` (WebSocket nativo do Node) + `/api/active-pokes` + card "Time ativo" na Conta.
Detalhe do protocolo em [[Poke Idle World - endpoints publicos de dados]]. Testado: 225 na
colecao, time de 3, ~0.3s.

**Pegadinha e a saida (ajuste ago/2026):** o WS E a sessao de jogo — conectar chuta a aba
do jogo aberta ("conta em uso"). Como conectar ja toma a sessao de qualquer jeito, o
piwdex puxa o time **no proprio connect** (do bookmark) e guarda um **snapshot** no banco
(`game_links.team_snapshot/total/at`). A Conta mostra o snapshot pelo `/api/collection`
SEM tocar o jogo; "atualizar" repuxa ao vivo (ai sim toma a sessao) e regrava. Mercado e
outros dados sao REST (Bearer) e NAO tocam a sessao — so o WS. (NOTA: a conclusao de que
"robo/realtime e territorio da extensao" foi REVISTA nas passadas seguintes — o robo virou
server-side e segura a sessao WS de proposito. Ver abaixo.)

**Robo server-side (10a passada, ago/2026):** a aposta anterior ("robo = extensao na aba
do jogo") foi abandonada — o robo virou **server-side de verdade**, segurando a sessao WS
de proposito. Isso so foi possivel cravando o protocolo do WS por engenharia reversa
(ferramentas em `scripts/`, protocolo inteiro em `scripts/ws-protocol.md`). Entregue na
area VIP (aba Robo, com sub-navegacao): **Hunt Analyzer ao vivo** (entra no campo via
`enter-hunt`, faz poll de `analyzer-get` a cada 2s e mostra kills/XP/loot/saldo por hora —
segurando a conexao, o que poe a aba do jogo em "conta em uso"); **venda de pokemon ao
vivo** (simular -> conferir -> vender) e **venda de drops**, com travas granulares (box por
raridade, IV 0-192, qualidade decimal); **poke-summon** (troca o pokemon ativo/lider —
comando que MUTA a conta, verificado); automacao nativa do jogo configurada server-side; e
auto-compra como placeholder. O aprendizado que destravou tudo virou nota:
[[Quando a REST não expõe o dado, o WebSocket do mesmo sistema entrega]] — o time
individual, o analyzer e o summon so existem no WS, nunca na REST.

**Redesign item 1 — sessao unificada + Conta read-only + Conectar VIP (11a passada,
ago/2026):** com o robo segurando a sessao, ficou incoerente ter DOIS lugares disputando a
sessao de jogo (a Conta com um "atualizar time" que roubava a sessao, e o robo). O Eduardo
decidiu unificar: **tudo que depende da sessao de jogo mora no `/vip`**.
- **Conectar virou 100% VIP**: vincular a conta do jogo = tomar a sessao, que so serve as
  features pagas. Gate `session.user.vip` no servidor em `connect`/`collection`/
  `active-pokes` — mais um caso de [[Permissão se valida no servidor, não na interface]]
  (gate de capacidade, nao so de linha).
- **Conta virou aba read-only do VIP**: `/conta` redireciona pro `/vip#conta`; a visao da
  conta e uma aba (o landing do VIP) que mostra so o **snapshot** capturado no connect,
  sem nunca tocar o jogo. O "atualizar time" que roubava a sessao saiu — atualizar ao vivo
  agora e territorio do Robo (que ja segura a sessao). A Conta jamais disputa a sessao.
- Icone Conta saiu do nav do topo (entra pelo botao VIP). i18n pt/en/es.
Verificado: `tsc` limpo, `next build` ok. Branch `refactor/sessao-unificada`.

**Seletor de hunt + venda automatica 24/7 (12a passada, ago/2026):**
- **Seletor de hunt**: o slug do `enter-hunt` E o `Hunt.slug` do catalogo (347 hunts;
  confirmado no JSON — "ledian"/"bulbasaur" sao entradas de hunt, nao so as areas
  "cerulean"/"pewter"). Trocou o input de texto por um MODAL com busca, agrupado por area
  (kanto/outland/orre), com o sprite do pokemon do ponto (resolvido por looktype/nivel mais
  proximo) e o nivel de cada hunt. Dados do server (`vip/page -> VipTabs -> RoboModule ->
  HuntAnalyzer`).
- **Venda automatica 24/7**: o Eduardo escolheu robo com sessao propria. O piwdex SEGURA a
  sessao WS e, a cada 60s, pede a lista viva (`pokes-get`), aplica as travas e vende o que
  bate via REST (`sellPokes`). Vender e REST; so a LISTA precisa do WS — por isso `pokes-get`
  na conexao segurada resolve. Dono unico da sessao: **mutuamente exclusivo com o Hunt
  Analyzer** (os dois seguram o WS — ligar um para o outro). A regra do que pode vender saiu
  pra `poke-sell.ts`, modulo unico que a simulacao, a venda manual e o robo reusam (venda e
  irreversivel — a regra tem que ser uma so). Guarda-costas duro: nunca time/lider/starter/
  shiny. Reforcou a nota [[Quando a REST não expõe o dado, o WebSocket do mesmo sistema entrega]].
  Semantica das travas de IV/qualidade (decisao do Eduardo, ago/2026): pra FICAR na conta o
  pokemon tem que passar das DUAS travas — falhou em qualquer eixo, vende. Trava e filtro de
  excelencia, nao de "bom em algum eixo" (a versao inicial mantinha com um eixo so e enchia
  o acervo de bicho mediano).

**Robo completo + area VIP polida (13a passada, ago/2026):** fechou o kit do robo e o
visual da area logada.
- **Hunt**: feed ao vivo de kills E capturas (catch-result), cada card com horario, sprite
  e icone do loot (o itemId do field-kill nao bate com os dados — resolve o icone por NOME),
  clicavel pra um modal de detalhe. Seletor de hunt em modal (347 hunts do catalogo). Stats
  com icone; a moeda do jogo e DOLAR, nao ouro.
- **Venda automatica de drops na Hunt**: ao ligar, um modal lista os drops vendaveis daquele
  ponto (loot da especie -> item real, so vendaveis) e o jogador marca o que vender; a sessao
  rastreia a mochila (frame `inventory` do WS) e vende via REST a cada 30s. "Vender drops"
  manual virou "Itens vendidos" (read-only).
- **Alertas do que rolou offline**: os robos server-side gravam eventos no Postgres enquanto
  rodam (migration 009 `robot_events`, tabela PROPRIA — o inbox de mercado e acoplado a
  watchlists). Shiny em destaque (individual), resumo de sessao de Hunt ao parar, vendas de
  pokemon/itens por leva; captura comum so entra no resumo (banco leve, teto 200/user). Isso e
  o valor do robo server-side: sobrevive a fechar o navegador (ver a pegadinha single-session).
- **UI**: marca **PIWdex** (logo + titulo fixo da aba), favicon pokebola, banner VIP (barra
  fixa que fecha + rodape que nao fecha) nas paginas publicas, header so com icone de Conta
  (sem VIP no topo), footer removido.

**Correções do robô — mesma sessão + venda ao coletar (14a passada, ago/2026):** o Eduardo
apontou os erros que sobraram. Consertados sem forçar reconexão:
- **Trocar a bola não pegava na hunt em andamento** (só reconectando): a engine de campo do
  jogo cacheia a autohelper no `enter-hunt`. Agora `/api/vip/auto`, ao gravar, chama
  `gameSession.refreshHunt()` que **reenvia `enter-hunt` no mesmo socket** — relê a config
  sem reconectar nem zerar a caça. Virou nota:
  [[Config que a sessão cacheia no init não vê a escrita no backend, reaplique na mesma conexão]].
- **Trocar o pokémon ativo AO VIVO** (`poke-summon`): antes só havia a _sugestão_ do melhor
  do time ("troque no jogo"). Agora tem botão "Usar este" — troca o líder na mesma sessão
  (socket vivo se a hunt roda, one-shot se não). Nova rota `/api/vip/summon`.
- **Venda de pokémon agora vende ASSIM QUE COLETA** (removido o throttle de 1h), igual à de
  drops. Isso fechou o **limbo** do capturado que batia a trava: saía do acervo ("vai
  vender") mas ficava até 1h sem ser vendido — nem no acervo nem vendido. Reforçou
  [[Espelhar por balde esconde item no lugar errado]] (versão temporal do fantasma).
- **Sem alerta por venda** (item/pokémon): poluía o feed de Atividade. O vendido já aparece
  nos painéis Itens/Pokémon vendidos e no totalizador de Estatísticas — o feed fica só pra
  shiny, resumo de hunt e compras.
- **Tempo real**: contagem de bolas na config atualiza sozinha sem atropelar a edição em
  andamento; acervo e estatísticas de 8s → 5s. Os painéis de dado vivo (hunt, vendidos,
  conta, alertas) já faziam poll.
Verificado: `tsc` limpo e `next build` ok. Branch `refactor/polish-ui-master` (sem remote).

**Dashboard de Estatísticas + tira "Vender agora" (15a passada, ago/2026):** o Eduardo pediu
que a aba Estatísticas virasse um dashboard de verdade, não só dois totais de venda.
- **Dashboard cumulativo (pra sempre)**: além das vendas, agora mostra o que a hunt rendeu
  (hunts concluídas, derrotados, capturados, itens coletados, XP), **dólar por fonte** (loot
  coletado vs itens vendidos vs pokémon vendidos, em gráfico de barras) e o **acervo mantido
  por raridade**. Migration 012 estende `robot_sales` com kills/captures/xp/loot/supply/hunts,
  preenchidos no fim de cada hunt (`logSummary`, mesmo gancho do resumo). `captured_pokes`
  ganhou `capturedStats` (total/shiny/por raridade). Gráficos em barra CSS pura (sem lib), na
  estética pixel — segue [[Estetica e por projeto, principio de design e que se reusa]].
- **Removido "Vender agora"**: o modal de ligar a hunt já pergunta se quer vender, e a venda
  é assim que coleta — botão manual só poluía. Tirado da UI, da rota e da sessão.
- Limitação anotada: "itens raros coletados" não dá (o drop do `field-kill` não carrega
  raridade); o dashboard usa raridade só onde ela existe (acervo de pokémon).
Verificado: `tsc` limpo, `next build` ok, migration aplicada no `piwdex-db`.

**Itens raros + tira venda 24/7 + conserta summon (16a passada, ago/2026):**
- **Itens raros coletados** no dashboard: o Eduardo apontou que a raridade EXISTE nos itens
  (a pagina de itens ja mostra "RARO"). O drop do `field-kill` nao carrega raridade, mas o
  **item dos dados carrega** (`item.rare`) — o join e por nome (`getItemByName`, cai pro
  itemId). Somado no fim de cada hunt a partir do breakdown do analyzer. Migration 013.
  (Revê a limitacao que eu tinha anotado na 15a passada — dava sim, era so cruzar o nome.)
- **Venda de pokemon virou atrelada a Hunt**: removido o card "Venda automatica (24/7)" das
  Configuracoes (o Eduardo: "deixe so a trave"). A venda so liga pela opcao "Vender pokemon
  junto" no modal de iniciar a Hunt, e `stopHunt` passa a parar a venda tambem (nao ha mais
  toggle standalone). So as travas ficam na config.
- **Conserta o 502 do poke-summon**: confirmar pelo echo `poke-summon` dava falso timeout (o
  echo nem sempre chega). Agora, apos o summon, pede `pokes-get` e confirma quando o alvo
  volta `leader:true`. Virou nota:
  [[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]].
Verificado: `tsc` limpo, `next build` ok, migrations 013 aplicada.

**Tempo real de verdade + summon reflete na Conta (17a passada, ago/2026):** o Eduardo notou
que a Caçada ficava zerada durante a hunt e que o summon "funcionava" mas a Conta nao mudava.
- **Estatisticas em tempo real**: a Caçada (derrotados/capturados/itens/XP) e o dolar de loot
  passaram a ser o persistido (hunts fechadas + vendas) + o analyzer AO VIVO da hunt em
  andamento, somado so quando ha hunt rodando. Antes so somava no fim. Virou nota:
  [[Total ao vivo é o persistido fechado mais o em andamento ainda não gravado]].
- **Time ativo ao vivo**: enquanto o robo segura a sessao, cada lista de `pokes` regrava o
  snapshot do time (`game_links`) — a Conta mostra o lider atual sem reconectar. (A Conta
  jamais abre conexao propria; reusa a que o robo ja segura.)
- **Summon reflete na Conta**: o motivo do "ficou ativo mas nao mudou aq" era o snapshot
  velho da Conta, nao o summon (que funcionou). `summonPoke` agora devolve a lista lida e a
  rota regrava o snapshot; na hunt viva, `summonActive` pede `pokes-get` e a sessao atualiza
  sozinha. Segue [[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]].
Verificado: `tsc` limpo, `next build` ok.

**Reforma da area VIP — cockpit em tempo real + robo autonomo (18a passada, ago/2026):**
o Eduardo rejeitou a area VIP como estava ("desorganizado e espalhado", tudo exigindo F5)
e pediu: tempo real em tudo, monitor de conexao fixo, robo que mantem a conexao sozinho,
auto-hunt sem escolher nada e "upar pokemon do lvl X ao Y" seguindo a melhor sequencia de
hunts. Entregue inteiro (branch `feat/vip-reforma` -> `master`):
- **Tempo real de verdade**: os 8 `setInterval` dos paineis (2s-10s) e o F5 de Desejos/
  Alertas viraram UM stream SSE (`/api/vip/stream`) — o motor emite eventos e o stream
  empurra na hora; banco/API do jogo entram por poll server-side unico. Provider React
  distribui; painel nenhum faz fetch de leitura. Virou nota:
  [[Um stream SSE substitui a constelação de pollings]].
- **Robo persistente**: tabela `robot_sessions` (migration 014) guarda o estado DESEJADO;
  reconexao automatica com backoff (renovando o access token antes de reabrir — WS nao tem
  retry-em-401) e `instrumentation.ts` religando no boot do container. Virou nota:
  [[Estado desejado persistido religa o robô depois do restart]].
- **Cerebro (`hunt-brain.ts`)**: reusa o motor de rota publico (`combat.ts`/`buildRoute`).
  Modo AUTO: escolhe a melhor hunt pro lider e re-escolhe a cada level-up (margem de 8%,
  a mesma da rota, pra nao pular de hunt a toa). Modo LEVELING: plano nivel X -> Y com as
  faixas de hunt, preview na UI (`/api/vip/plan`), execucao trocando de hunt sozinho pelo
  nivel AO VIVO (frames `field-kill`/`poke-xp`/`pokes` do WS, que antes nem eram parseados);
  meta batida vira evento `goal` e cai pro modo auto (segue rendendo). Sem curva XP->level
  publica: o plano decide por nivel observado, nao por tempo estimado — honesto e suficiente.
- **Cockpit**: HUD fixo em todas as telas (LED do robo, modo, hunt, XP/h, level+barra de XP,
  dolares, diamantes, bolas com alerta de estoque), navegacao LATERAL pixel com 9 secoes e
  acento por cor (matou o tab-dentro-de-tab), container proprio de ~103rem (breakout do
  container do site via `margin-inline: calc((100% - w)/2)`), painel "Visao geral" novo
  (monitor + hunt + plano + feed de kills com flash + eventos + acumulado), icones pixel
  novos (Signal/Target/Flag/Brain/Chart/Sword/Backpack) e animacoes (flash-in, radar).
Verificado: `tsc` limpo, `next build` ok, migration aplicada, smoke dos endpoints.

**Revisao bruta + pacote de deploy Railway (19a passada, ago/2026):** revisao completa
pre-deploy (5 frentes em paralelo: Railway, calculos, breeding, UI, seguranca) e o
pacote que deixa o projeto pronto pro Railway (dominio piwdex.com.br):
- **Deploy**: o runner do Dockerfile agora carrega `db/` e o `railway.json` roda as
  migrations no pre-deploy (`node db/setup.mjs`) com **1 replica travada** (robo e
  keepalive sao singleton por processo). O worker curl do sniper saiu do compose — a
  varredura virou **loop interno no boot** (`src/lib/sniper.ts` + instrumentation),
  mesmo mecanismo em dev/compose/Railway; `/api/cron/scan` ficou como gatilho manual
  (agora timing-safe). Boot **aborta em producao** sem DATABASE_URL/AUTH_SECRET/
  SESSION_SECRET/APP_URL (antes: AUTH_SECRET explodia no 1o login e APP_URL faltando
  deixava o webhook do MP em localhost — pagamento sem VIP). Redirect www -> apex por
  host + headers de seguranca no `next.config.ts`. Viraram notas:
  [[Railway não roda compose, cada serviço vira uma peça da plataforma]] e
  [[Cookie de sessão é host-only, www e apex canonizam num domínio só]].
- **Veredito da revisao**: nucleo matematico solido (tabela de tipos conferida linha a
  linha, stats/breeding batem com a referencia curada), SQL 100% parametrizado, tokens
  cifrados, sem IDOR, design system maduro e sem emoji. **Pendencias conhecidas** (por
  prioridade): (1) `gameSession` singleton — com 2+ VIPs simultaneos um ve/controla a
  sessao de jogo do outro; virar `Map<userId, GameSession>` antes de vender VIP pra
  mais gente; (2) especie fora do dex vira default COMMON = vendavel no robo (venda
  irreversivel — trocar pra default nao-vendavel); (3) sem rate limiting em login/
  registro; (4) webhook MP sem validar `x-signature` (mitigado: re-busca na fonte,
  idempotente); (5) UI: falta `not-found.tsx`/`error.tsx` tematicos, `loading.tsx` em
  /hunt /calc /breed /eevee, formatacao de numero presa em pt-BR (32 usos), golpe
  STATUS contando no DPS do combat, race do setTimeout na calculadora.

**Acervo sem venda (fix, ago/2026):** capturado com a venda de pokemon DESLIGADA nao
aparecia em lugar nenhum — o HUD contava (analyzer), nada era vendido, e o `recordKept`
exigia `pokeCfg` (o gravador do acervo estava acoplado a config de venda). Terceira
encarnacao do limbo: o registro de retencao nao pode depender da config de descarte.
Agora `recordKept` roda sempre — sem venda, toda captura nova (fora da linha de base) e
mantida e entra no acervo. Reforca [[Espelhar por balde esconde item no lugar errado]].

**Valor do chip do chat + regua por faixa (ago/2026):** o link `[poke!...]` do chat e
JSON em base64 sem preco; o modal agora mostra "Valeria ~X" via `/api/vip/value` — o
motor de preco justo do consultor, corrigido: a regua unica (mediana de preco-por-teto
do mercado inteiro) esmagava a elite (Haunter Q2.105 avaliado como media do lixao).
Virou mediana POR FAIXA de Quality (1.4/1.8/2.1) e shiny separado, cadeia
especie+faixa -> faixa -> geral, e teto de sanidade pelo anuncio real igual-ou-melhor
mais barato. Cache do mercado extraido pra `market-cache.ts` (compartilhado consultor/
value/sniper). Tambem: chat vira ring de 20 mensagens. Nota:
[[Avaliação por comparáveis segmenta a régua por faixa, não mediana única]].

## Infra

Slug `piwdex` · app `piwdex-app` na `4070` · banco `piwdex-db` na `5070` (Postgres 17,
entregue na camada 4: login + vinculo do jogo). Chassi e mapa de portas em [[Infra]].
**Producao**: Railway (app via Dockerfile + plugin Postgres, mapeamento em
[[Railway não roda compose, cada serviço vira uma peça da plataforma]]), dominio
canonico `piwdex.com.br` (www so redireciona).

## Stack

Next.js 16 (App Router) · React 19 · Tailwind 4 · TypeScript. Camada 1 e dado estatico
gerado em build (100% publico, read-only) — sem backend ainda. Postgres + Docker
compose entram na camada 3 (companion logado).

## Decisoes importantes

- **Puxar da fonte-mestra do jogo, nao do piwtools.** O piwtools e uma copia; a origem
  (`/game/*.json`) da o mesmo dado e pega patch de balanceamento antes.
- **Snapshot puro + derivacoes no codigo.** O JSON versionado espelha a fonte; indice
  reverso, localizacao e evolucao sao computados, pra o snapshot ficar diffavel.
- **Estatico primeiro, backend so onde precisa.** Camada 1 e reference data que muda
  pouco: static gen e a ferramenta certa. O chassi Postgres entra na camada 3, onde o
  companion logado (JWT + WebSocket) realmente exige servidor.
- **Estetica propria (tool de jogo), principios herdados.** O dialeto Aurora Glass e
  pra dashboard; aqui vale so o principio (token semantico, escala, 4 estados, claro+
  escuro). Acento de UI (vermelho pokebola) separado da cor de dado (cor por tipo).

## Aprendizados (viraram notas)

So links. O texto do aprendizado mora na nota, nunca aqui.

- [[Poke Idle World - endpoints publicos de dados]] — o schema e os dois joins que
  destravam o dado (chance/1000, looktype -> mapa).
- [[Poke Idle World - regras de breeding]] — regras curadas do breeding (nao vem no
  JSON): mesma especie, diff Quality 0.150, heranca de IV, tabelas de ganho e custos.
- [[Auth.js sem adapter, a sessão JWT resolve o usuário no seu próprio SQL]] — login
  (email/senha) convivendo com `pg` cru, sem adapter de ORM.
- [[Descobrir o shard por sondagem paralela com early-exit e cachear]] — achar o shard
  do WS quando o servidor nao diz qual e o seu.
- [[Quando a REST não expõe o dado, o WebSocket do mesmo sistema entrega]] — o dado que
  falta na REST (time individual, analyzer, summon) mora no WS, ao custo da sessao viva.
- [[Config que a sessão cacheia no init não vê a escrita no backend, reaplique na mesma conexão]] —
  a bola gravada via REST so pegava reconectando; reaplicar na mesma sessao (reenviar
  `enter-hunt`) valeu sem derrubar a caca.
- [[Espelhar por balde esconde item no lugar errado]] — a versao temporal: capturado que
  ia vender saia do acervo mas o throttle de 1h deixava em limbo; venda imediata fechou.
- [[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]] — o
  poke-summon dava 502 esperando o echo; confirmar pela releitura (`leader:true`) resolveu.
- [[Total ao vivo é o persistido fechado mais o em andamento ainda não gravado]] — o dashboard
  da Caçada ficava zerado durante a hunt; somar o analyzer vivo ao persistido deixou vivo.
- [[Permissão se valida no servidor, não na interface]] — o gate de VIP (conectar/conta)
  e negado no servidor, em cada rota que toca a sessao de jogo.
- [[Um stream SSE substitui a constelação de pollings]] — os 8 pollings da area VIP
  viraram um stream com eventos nomeados; push do que e memoria, poll unico do resto.
- [[Estado desejado persistido religa o robô depois do restart]] — intencao no banco,
  reconexao com backoff + refresh de token, boot religa via instrumentation.
- [[Railway não roda compose, cada serviço vira uma peça da plataforma]] — migration
  vira pre-deploy na propria imagem, worker vira loop no processo, 1 replica com singleton.
- [[Cookie de sessão é host-only, www e apex canonizam num domínio só]] — por que o
  www.piwdex.com.br so redireciona pro apex.

## Proximos passos

- [x] Camada 1: ingestao + pokedex + ficha + itens + indice reverso de drop.
- [x] Redesign visual pixel/neon + calculadora de IV/qualidade/poder (formula do jogo).
- [ ] Camada 2 (resto): Hunt Planner (lucro/hora, XP/h, melhor spot por item) e
  comparador de dois pokemons; formulas de XP/poder raspadas de `/pokepedia/systems/*`.
- [x] Breeding como camada curada: simulador `/breed` com colecao local e projecao.
- [ ] Bosses como camada curada (nao vem no JSON do jogo).
- [x] Camada 3 (leitura): companion logado por token; `/conta` completa (perfil, treinador,
  automacao, streak, breeding, inventario+deposito, bolas) + bookmarklet de conexao.
- [x] Camada 3 (resto): pokemons ativos individuais via WebSocket (time + colecao com IV).
- [x] Camada 4: login do site (Auth.js Google+email, Postgres) + vinculo do jogo por usuario.
- [ ] Camada VIP: gateway Mercado Pago (~R$15,90/mes) + area `/vip` (hospeda o mercado, gated).
- [x] Robo server-side (Hunt Analyzer ao vivo, venda de pokemon/drops, poke-summon,
  automacao nativa) na area VIP — segurando a sessao WS. Protocolo em `scripts/ws-protocol.md`.
  (Substituiu o plano de "robo via extensao de navegador".)
- [x] Redesign item 1: sessao unificada — Conectar 100% VIP, Conta read-only como aba do VIP.
- [ ] Redesign itens 2-4 (proximos: a definir na sessao).
- [x] Camada VIP: area `/vip` + assinatura Mercado Pago (avulso mensal) + mercado gated.
- [ ] Ligar o MP de verdade (token + APP_URL publica) e deploy pro webhook funcionar.
- [x] Pacote de deploy Railway: migrations na imagem + pre-deploy, sniper interno,
  fail-fast de env, canonizacao www -> apex. Falta so criar o projeto no Railway
  (Postgres + envs + dominio) e apontar o DNS.
- [ ] Antes de vender VIP pra 2+ pessoas: sessao por usuario (`Map<userId, GameSession>`).
- [ ] Correcoes da revisao: default nao-vendavel pra especie desconhecida, rate limit
  no login, x-signature do webhook MP, not-found/error/loadings, fmtNum por locale.
- [ ] colorGroup no `.obsidian/graph.json` (pendente — regras de cor no CLAUDE.md).

## Camada VIP + assinatura (ago/2026)

- **Arte real do jogo** self-hostada (`npm run bake:sprites` -> `public/game-sprites/<looktype>.webp`,
  372 pokemons + 84 skins de player); `spriteUrl`/`skinSpriteUrl` preferem a arte do jogo.
- [x] **Login piwdex** (camada 4): Auth.js email/senha, conta SEPARADA da do jogo, Postgres 5070.
- [x] **Area `/vip` entregue**: gated por login. Paywall (beneficios + preco + Assinar) pra
  nao-VIP; consultor de mercado (`<MarketAdvisor>`, agora **VIP-only**, gate tambem no
  `/api/market`) + placeholder do robo pra VIP. Link VIP no header.
- [x] **Pagamento: avulso mensal** (decisao do Eduardo, nao recorrente) via **Mercado Pago**,
  REST puro sem SDK. `/api/vip/checkout` cria a preference (init_point); `/api/vip/webhook`
  confirma na FONTE (nao no corpo), idempotente (tabela `vip_payments`), concede +30 dias
  (`grantVipDays` estende `users.vip_ate`). **Modo teste** (dev sem `MP_ACCESS_TOKEN`): libera
  direto; em prod fica indisponivel ate plugar. Flag `vip` ja chega fresca na sessao (jwt rele).
- **Falta pra ligar de verdade:** por o `MP_ACCESS_TOKEN` no `.env` + `APP_URL` publica (o
  webhook precisa de URL publica -> so apos deploy). Depois: alertas de mercado, e o robo.
- **Automacao/robo via extensao** de navegador: roda no dominio do jogo -> usa a sessao logada ->
  chama a API sem captcha, SEM disputar a sessao (o WS server-side chuta o player — ver a
  pegadinha em [[Poke Idle World - endpoints publicos de dados]]). Bookmarklet e o passo 0;
  extensao e o produto.

**Portal admin (11a passada, ago/2026):** o Eduardo quis um painel interno pra ver TODAS
as contas e "entrar" na de qualquer usuario. Entregue:
- **Flag `users.is_admin`** (migration 016). A conta admin cai em `/admin` em vez da area
  VIP/Conta (redirects em `/entrar` e `/conta`); a sessao carrega `session.user.admin` no
  jwt callback do Auth.js (mesmo padrao do `vip`). Gate duro server-side na rota.
- **Painel `/admin`**: tabela de todos os usuarios com VIP (+ validade), vinculo de jogo
  (player + status) e **moedas AO VIVO** (gold/diamantes/level/capturas) puxadas por REST
  `/api/game/profile` com o token de cada um — **sem abrir o WS**, entao nao rouba a sessao
  de jogo de ninguem (o bot segue conectado). REST Bearer nao tem fronteira de origem; o
  storage do navegador tem.
- **"Entrar como"**: dois jeitos de injetar o `pokeweb:tokens` na aba do jogo — copiar um
  snippet pro console + abrir a aba, e um bookmarklet arrastavel. Nao da pra um botao logar
  direto em outra aba: o navegador isola storage por origem. Aprendizado em
  [[Sessão de outro domínio só se injeta rodando na origem dele]].
- **Banco de producao exposto** por Public Access da Railway (`altaria.proxy.rlwy.net`) pra
  criar o admin e debugar via `pg` — pendente **rotacionar a senha** (a antiga vazou em
  chat) e, se quiser, remover o Public Access depois.

**Robo segura a sessao + auto-compra completa (12a passada, ago/2026):** duas correcoes
que o Eduardo pediu.
- **REVERSAO da politica de sessao:** a 10a passada fazia o robo CEDER a sessao quando a
  conta era tomada (usuario entrava no jogo pelo navegador) — pausava e avisava. Na pratica
  o robo desligava sozinho sem o usuario saber. Agora o robo SEGURA: reconecta na hora pra
  reclamar a sessao (o jogo da a conta pra conexao mais nova) e so larga quando o usuario
  DESLIGA. Um **aviso fixado** (`SessionHoldNotice`) explica: pra jogar no navegador,
  desligue o robo primeiro. Licao: num recurso single-owner disputado, a politica (segurar
  vs ceder) tem que ser explicita E visivel — ceder em silencio confunde tanto quanto o
  cabo-de-guerra silencioso.
- **Auto-compra repoe pocao e revive, nao so bola:** `restockBalls` virou `restockSupplies`.
  Pega a pocao (se Auto-Potion ligado) e o revive (se Auto-Revive ligado) com pisos/alvos
  menores (sao caros; revive ate 2500). Pegadinha do jogo: o Auto-Potion **nao tem campo de
  qual pocao** — o jogo usa a melhor sozinho. Entao a escolha de o-que-comprar vive do lado
  do piwdex (`robot_sessions.supply_cfg`, migration 017; null = "a melhor disponivel", a mais
  cara buyavel). Seletor na UI de Auto-compra. Pocoes = categoria `heal` (ids 200-204),
  revive = `revive` (205-206) nos dados estaticos.

**Venda/auto-compra confiaveis + Configuracoes com um Confirmar so (13a passada,
ago/2026):** o Eduardo reportou "vive falhando": nao vendia pokemon, nao comprava bola,
fila de captura travada. Tres causas-raiz, todas de fonte-da-verdade:
- **Tokens stale**: tres frentes renovavam o token rotativo do jogo (WS, poller SSE,
  auto-compra), cada uma com copia propria em memoria — a copia de uma apodrecia quando
  outra renovava e TODA venda/compra dela morria em 401 dentro de `catch {}`. Fix: toda
  frente rele o par do banco antes de cada lote REST. Ver
  [[Token que rotaciona não tolera cópia longeva, releia do banco antes do uso]].
- **Config de venda no localStorage**: so o start da aba Hunt anexava a config; hunt
  iniciada pelo Painel rodava sem venda (capturas vendaveis iam pro acervo), e parar a
  hunt APAGAVA a config do banco. Fix: config mora em `robot_sessions.poke_sell_cfg`
  com interruptor `on` embutido (desligar preserva as travas) e a rota aplica a config
  salva em start/auto/leveling/connect. Ver
  [[Config que o motor executa mora no servidor e se aplica em todo início de fluxo]].
- **Auto-compra 1x/h**: hunt queima ~700 bolas/h, o piso de 100 zerava em minutos e o
  auto-catch do jogo travava a fila ate a proxima hora. Fix: varredura de 10min + gatilho
  AO VIVO no frame `balls` do WS (bola da automacao abaixo do piso -> repoe ja, cooldown
  3min) — aplicacao de [[Estado vivo se empurra, não se pergunta]]. Piso 150, alvo 1000.
- Falhas de venda/compra agora viram Alerta (kind `error`, 1 por operacao/30min). Ver
  [[Falha de automação recorrente vira alerta com throttle, não catch vazio]].
- **UI**: secao Configuracoes unificada (`ConfigPanel`) — automacao nativa + travas de
  venda + auto-compra num draft so, barra sticky com contador de pendencias e UM botao
  Confirmar (substituiu tres cards, cada um com o seu). Seletor de bola mostra estoque e
  avisa em vermelho quando a bola da automacao esta zerada. Checkbox "Vender pokemon
  junto" da aba Hunt le/grava o servidor (mesmo interruptor das Configuracoes).

**Fonte legivel + primitivos de UI + Painel/Hunt reestruturados (14a passada, ago/2026):**
o Eduardo pediu fonte pixel mais legivel, painel mais bonito/dinamico e "sempre criar
componentes primitivos reutilizaveis".
- Fonte pixel: Press Start 2P -> Silkscreen -> **Jersey 10** (escolha final do Eduardo,
  peso unico 400, pixel de placar bem legivel); classes pixel do CSS subiram de tamanho.
- Painel DENSO (feedback: "sem espaco em branco, tudo encaixadinho"): barra de comando em
  uma linha, colunas de mesma altura, EmptyState compact (vazio vira linha), Panel ganhou
  modo collapsible. Checkbox "Vender pokemon junto" saiu da Hunt (duplicava Configuracoes).
- Copy da UI reescrita curta e neutra, sem apostrofo — ele rejeitou "textao de IA".
- **Primitivos em `src/components/ui/`**: Panel (card+cabecalho com icone/acento/slot),
  Led/LiveBadge/AlertBanner, ProgressBar, FeedRow/EmptyState — aplicacao direta de
  [[Primitiva de botão fecha o tamanho e abre só a variante]] em card/feed/status.
- Painel: barra de comando do robo em largura cheia + regua de rendimento com tick-glow
  (StatTile ganhou prop `live`: o valor pisca no acento quando muda — remonta o span por
  key). Hunt: hero da hunt viva com sprite/modo/status.
- Master Ball (id 5) escondida do Painel e da Conta — ninguem tem, nao e compravel.

**Mercado completo: compra direta, itens e desejos de item (15a passada, ago/2026):**
- O `/api/game/market` ja trazia TUDO (pokemon + pilhas de item + historico + catalogo);
  a compra e `POST /api/game/market/action` com `{action:"buy",id}` (pokemon) ou
  `{action:"buy-stack",kind,refId,price,currency,quantity,ids}` (itens) — contrato
  extraido do bundle publico do cliente do jogo, ver
  [[O bundle público do cliente entrega o contrato da API sem documentação]].
- Rota `/api/vip/market-buy` reconfere anuncio+preco no servidor antes de gastar
  (preco mudou = 409); ids da pilha vem do servidor; compra vira evento no feed.
- Mercado com abas Pokemon | Itens na mesma tela; desejos tambem (migration 018:
  `watchlists.kind`+`item_id`; sniper casa pilhas com dedup por item+preco unitario).

**Stats reais + raridade + secao Meus Pokemons (16a passada, ago/2026):**
- O anuncio do mercado JA trazia `stats` do individuo (a nota de referencia sabia); o
  normalizador descartava. Modal do mercado agora mostra os stats REAIS do bicho a
  venda, com barra vs o stat perfeito (IV 32 no mesmo nivel/qualidade) e IV estimado
  por stat — formula invertida de `stats.ts`. Base da especie virou fallback.
- `PokeStatsModal` + `MonStatsSection` (`mon-stats.tsx`) compartilhados: mesmo modal
  de stats no mercado, no time do painel, na Conta e no box.
- RarityBadge (raridade de especie do catalogo) em todo lugar que mostra pokemon:
  card/modal do mercado, achados dos Desejos, time da Conta, lider do painel e box.
- Secao nova **Meus Pokemons** no menu VIP (icone pokebola): gestao inteira do time
  (trocar lider, guardar no box) + box completo inline com busca/ordenacao. O card do
  painel virou LEITURA pura (clique = stats) com link pra secao — decisao do Eduardo,
  ver [[Cockpit mostra, aba própria gerencia]].

## Conexoes
- Usa: [[Design]] · [[Infra]]
- Aplica: [[Sessão de outro domínio só se injeta rodando na origem dele]] · [[Token que rotaciona não tolera cópia longeva, releia do banco antes do uso]] · [[Config que o motor executa mora no servidor e se aplica em todo início de fluxo]] · [[Falha de automação recorrente vira alerta com throttle, não catch vazio]] · [[Lote recusado por um item se bissecciona até isolar o culpado]]
- Referencia: [[Poke Idle World - endpoints publicos de dados]]
- Mapa: [[Projetos]]
