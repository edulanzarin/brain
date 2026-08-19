---
tags: [tipo/referencia]
criado: 2026-08-15
---

# Poke Idle World - endpoints publicos de dados

> O jogo Poke Idle World (poke.idleworld.online) serve o catalogo inteiro como JSON
> publico, sem auth. Qualquer ferramenta/dex se constroi em cima disso. Puxar da
> fonte-mestra, nao de terceiros (piwtools e so uma copia).

Visto em: [[piwdex]]

## Fonte-mestra (JSON publico, sem auth)

Cloudflare bloqueia bot generico: mandar `User-Agent` de navegador.

| Endpoint | Conteudo |
|---|---|
| `poke.idleworld.online/game/creatures.json` | 482 pokemons: `pokeId, name, looktype, type1/2, rarity, base{Hp,Atk,Def,SpAtk,SpDef,Speed}, huntLevel, evolvesToId, evolveLevel, priceNpc, sellValue, experience, loot[], attacks[]` |
| `poke.idleworld.online/game/items.json` | 330 itens: `id, name, icon, category, rare, npcPrice` (+ `healAmount`/`revivePct` em consumiveis) |
| `poke.idleworld.online/api/game/map-markers` | 347 hunts: `slug, name, looktype, level, area, pixel[x,y], range[]` + `map{w,h}` |

`loot[]` = `{name, chance, minCount, maxCount}`. `attacks[]` =
`{name, type, category, power, cooldownMs, learnLevel}`.

## Duas chaves que destravam o dado

- **`chance` esta na escala 0..100000**: a porcentagem real e `chance / 1000`.
  Ex.: Bulbasaur dropa "Bulb" a `1675` = 1,675%. O piwtools nao mostra a % exata; a
  informacao esta na fonte.
- **`looktype` liga criatura -> localizacao no mapa**: o mesmo `looktype` aparece em
  `creatures` e em `map-markers`. 416/482 pokemons mapeiam pra ao menos um ponto de
  hunt. E o join pra "onde aparece o pokemon X".

Do `loot` sai o **indice reverso** "onde dropa o item X" (agrupar por nome do item).
Os 194 nomes de loot batem 100% com `items.json`.

- **Valor do pokemon = so venda.** No jogo NAO da pra comprar pokemon de NPC (o "NPC" e
  o cassino — Eevee etc. saem de la, dado ainda nao mapeado). Pokemon so se vende, ou se
  negocia no mercado de jogadores (camada 3, logada). Logo a UI mostra um unico campo
  **"Valor"**: usa `sellValue`, e cai pra `priceNpc` quando `sellValue` e 0 (18 exclusivos,
  ex. Aerodactyl). Nao rotular como "compra" — confunde (o Eduardo corrigiu).
- O `type` de um `attack` pode ser **`NEUTRAL`** (golpe sem tipo), fora dos 18 tipos
  canonicos de pokemon. Qualquer map por tipo (cor, icone) precisa de fallback.
- **`attacks[].cooldownMs` e o cooldown BASE, nao o tempo real de ataque.** Vem em ms
  (1200..30000) mas no jogo o intervalo real e bem menor — a velocidade/haste do
  pokemon reduz isso. Mostrar `cooldownMs/1000` como "10s" engana (o Eduardo pegou).
  Sem a formula de haste, e melhor NAO exibir CD. Os golpes em si (name/type/power/
  learnLevel) sao do jogo e corretos. STAB no jogo e o classico **1.5x** em golpe do
  mesmo tipo do pokemon (separado da efetividade x2/x0.5).

## API autenticada e realtime (JWT, dado da conta)

So pra companion logado. Base `poke.idleworld.online`, token em `pokeweb:tokens`
(localStorage):
`/api/game/{pokedex,all-pokes,balls,used-balls,profile,professions,offline,streak}`,
`/api/characters/me`, `POST /api/auth/refresh`. Realtime:
`wss://poke.idleworld.online/ws12?token=<JWT>` (eventos: field, field-kill,
catch-result, poke-xp, shiny-global, inventory, boosts...). Protocolo documentado no
repo de terceiros `AntonioFleck/poke-idle-launcher` (`docs/WS_SCHEMA.md`).

Contrato verificado (ago/2026, sondando a API): `GET /api/characters/me` com
`Authorization: Bearer <access>`; `POST /api/auth/refresh` body `{"refreshToken":"<str>"}`
-> novos tokens; `GET /api/auth/me`. O **mercado de jogadores** existe com preco em
**dollar E diamond** (confirmado), listing/premium. `balls`/`used-balls` podem destravar
dado real de captura (que o jogo nao publica).

**O login NAO da pra proxiar no servidor**: `POST /api/auth/login` exige uma confirmacao
anti-robo (captcha) amarrada ao navegador no dominio do jogo — o servidor do piwdex nao
gera esse token. Entao o companion loga **por TOKEN** (o jogador cola o `pokeweb:tokens`),
que por sorte e o modelo mais seguro (senha nunca sai do jogo). O `refresh` e o
`characters/me` funcionam so com o JWT, sem captcha. Implementado na camada 3 do piwdex.

**Onde o token realmente vive (verificado no bundle do app):** `sessionStorage["pokeweb:
tokens"]` = JSON `{accessToken, refreshToken}`. O jogo grava em sessionStorage e ATE
`localStorage.removeItem("pokeweb:tokens")` — por isso `localStorage.getItem` da null (o que
confundiu). sessionStorage e legivel por JS (nao e httpOnly), entao da pra automatizar a
conexao com um **bookmarklet**: le `sessionStorage["pokeweb:tokens"]` na aba do jogo e abre
`piwdex/conectar#<json>` (hash nao vai pro servidor); a pagina POSTa /api/connect na mesma
origem. Implementado no piwdex (`/conectar` + botao bookmarklet no /conta). Pega o refresh
token de brinde -> auto-renovacao.

**Sacada da extensao (futuro robo):** uma extensao rodando NO dominio do jogo usa a sessao
logada -> chama a API do jogo direto SEM captcha (o captcha so trava o /login, nao as chamadas
autenticadas). Ou seja a mesma extensao faz auto-conexao E automacao (farm/venda/compra).

Formatos verificados com token real (todos `Bearer`):
- `GET /api/characters/me` -> **so o TREINADOR** (`{character:{id,name,level,gold,diamonds,
  isVip,autoCatch,autoCatchBallId,autoPotion,selectedBallId,starterId,...}}`). NAO tem os
  pokemons individuais. `name`/`level` sao do treinador (confundiu a 1a versao da colecao).
- `GET /api/game/profile` -> resumo enxuto: `name,level,xp,gold,diamonds,rank,totalCatches,
  pokedexCount,pokedexTotal,vip,clan,...`.
- `GET /api/characters/me` -> `{character:{...}}` — o objeto MAIS RICO da conta. Alem do
  basico: `lookType` (skin do treinador, ~20029), `outfitHead/Body/Legs/Feet`, `gender`,
  `isVip`/`vipUntil`, `clan`/`clanRank`, `profession`/`professionRank`, `diamondsPurchased`,
  `referralCode`, `starterId`, `breedingSlots`, `streakExp/Loot/Shiny`, e a config de
  automacao NATIVA do jogo: `autoCatch`/`autoCatchBallId`/`autoCatchShiny`/
  `autoCatchShinyBallId`, `autoPotion`/`autoPotionThreshold`/`autoPotionItemId`,
  `autoRevive`, `selectedBallId`. Tambem `battlePass*` e `tasksClaimed`.
- `GET /api/game/all-pokes` -> `{total, entries:[{dexId,name,looktype,tier,count}]}` — pokedex
  AGREGADO por especie+tier (A..E), com quantos voce tem (167 especies distintas). Nao tem
  stats por individuo. **CUIDADO: o `looktype` daqui e outro numero (do individuo), NAO o da
  especie de creatures.json — usar `dexId` (== pokeId) pra sprite.**
- `GET /api/game/pokedex` -> `{unlockKills, species:[{id,kills,unlocked,claimed,caught,
  canClaim,captureBonus}]}`. Os `caught:true` sao o **40** do "40/332" do perfil (especies
  registradas pro bonus). NAO confundir com o 167 do all-pokes (especies possuidas).
- `GET /api/game/depot` -> `{inventory:[...], depot:[...], maxSlots}`. `inventory` = itens
  carregados, `depot` = guardados; cada item `{id,name,icon,category,npcPrice,quantity}`.
  Icone via `assetIconUrl` (mesma regra do itemIconUrl).
- `GET /api/game/streak` -> `{available, totalKills, bonusPct:{exp,loot,shiny}, tracks,...}`.
  **`bonusPct` e OBJETO** (exp/loot/shiny), nao numero.
- `GET /api/game/breeding` -> `{unlocked, slots(desbloqueados), maxSlots(cap 6), usedSlots,
  pheromones, eggs:[...], shinyPartner}`.
- `GET /api/game/professions` -> `{profession, rankName, catchBonusPct, speciesCount,...}`.
- `GET /api/game/balls` -> `{catalog:[{id,name,catchRate,priceGold,iconUrl,buyable,infinite}],
  counts:{"<id>":qtd}, selected, gold}`. **catchRate real**: Poke 1, Great 2, Super 3, Ultra 4,
  Idle 5, Master 255. `counts` e um MAPA id->quantidade.
- Outros 200: `/api/game/tasks` (tasks,cities), `/api/game/offline`, `/api/game/professions`.
- `GET /api/game/market` (~6MB) -> `{charId,listings,mine,requests,history,catalog,...}`.
  `listings` tem `kind` in item|pokemon|diamonds|ball, `currency` GOLD|DIAMONDS. **Os de
  pokemon ja vem prontos**: `{speciesId,level,shiny,stats,ivTotal,quality,power,type1,price,
  currency,belowNpc}` — o consultor de mercado do piwdex filtra e ordena isso direto.

Nao ha endpoint REST pros pokemons INDIVIDUAIS ATIVOS (o time + a colecao, com IV por
individuo) — a REST so da o agregado (all-pokes), o pokedex (flags) e os do mercado. Os
individuos so vem pelo **WebSocket** (cravado em ago/2026, engenharia reversa conectando
com token real):

- URL: `wss://poke.idleworld.online/ws<SHARD>?token=<JWT>`. **O shard e por conta** (ex.:
  Zashz = `ws47`); conectar no errado fecha na hora com **code 4003 `wrong-shard`**. Nao
  ha campo de shard no token nem na REST — **descobre-se por sondagem**: abre `ws1..ws64`
  em paralelo, o shard certo responde e os outros fecham com 4003; early-exit no primeiro
  que mandar dado (~300ms). Cachear o shard (o resto e 1 conexao).
- No `open`, o server **empurra** varios frames JSON `{type, ...}` sem pedir nada:
  `events, history, mail-badge, inventory, boosts, balls, autohelper, pokes, chat`.
- **`{type:"pokes", list:[...]}`** e o filao: TODA a colecao individual. Cada poke tem
  `id, speciesId, name, level, shiny, finalStage, team, slot, leader, starter, sellValue,
  looktype, xp, hp, maxHp, type1, stats{hp,atk,def,spAtk,spDef,speed}, quality, ivTotal,
  power`. **Time ativo = `team:true`** (ordenar por `slot`; `leader:true` marca o lider).
- **`{type:"joy-heal"}`** (sem parametro) -> `joy-healed` + `pokes` com o HP cheio: e a
  enfermeira Joy. Cura o TIME; quem esta no BOX continua desmaiado (visto em HAR: Ledian
  0/24 no box seguiu 0/24 depois do healed). Pokemon em 0 de `hp` nao entra em campo, e
  `hp` so existe no frame `pokes` — `/api/characters/me` nao traz vida e `field` so vale
  dentro da hunt.
- Outros eventos ao vivo (nao usados ainda): `field/field-kill/poke-xp/catch-result/
  shiny-global`. Protocolo tambem em `AntonioFleck/poke-idle-launcher` (`docs/WS_SCHEMA.md`).
- **PEGADINHA CRITICA: o WS E a sessao de jogo (single-session).** Conectar com o token
  = assumir a sessao; se o jogador estiver com o jogo aberto em outra aba, o jogo o
  **desconecta** ("conta em uso, so um lugar por vez"). Nao ha WS read-only/observer, e
  **nao ha REST pros individuos** (`/api/game/offline` so da `{report}` de progresso
  offline). Consequencia no piwdex: ler o time e **opt-in com aviso** (idealmente com o
  jogo fechado — como e idle, o normal e a aba estar fechada e ninguem ser chutado), e a
  conexao e a mais breve possivel (open -> `pokes` -> close). Isso reforca que a
  **automacao de verdade e via extensao** rodando NA aba do jogo (reusa a sessao viva,
  sem disputa) — ver o plano da camada VIP/robo em [[piwdex]].

Implementado no piwdex: `/api/collection` junta profile+characters/me+depot+streak+breeding+
balls+professions -> painel da Conta (`normalizeAccount`). `/api/active-pokes` abre o WS
(`fetchActivePokes` em `game-ws.ts`, WebSocket nativo do Node), pega o `pokes`, devolve o
time (`normalizeActivePokes`) — o shard fica cacheado em `game_links.shard`.

## Sprites / arte do jogo (sistema de outfit, estilo Tibia)

O jogo NAO serve `<id>.png` simples. Cada criatura tem um `looktype` (termo de
Tibia/OTServer) e a arte e um **spritesheet animado** montado num canvas (componente
`OutfitSprite`: `looktype, direction, colors, size, fps`). Puxar a arte real do jogo
(em vez de sprite da PokeAPI) deixa a ferramenta identica ao que se ve jogando.

Pipeline de assets (`/game/asset-packs`, base `f="/game/asset-packs"` no bundle):

1. **Indice**: `GET /game/asset-packs/outfits-index.json?v=2` (~166KB). Mapa
   `outfits[id] -> {id, gender, category, manifest, colorizable, directions, frames,
   width, height, name, kind}`. **`id` === `looktype`** (id 25 = Bulbasaur). Tem 629
   outfits, **372 com `kind:"pokemon"`** (o resto e treinador/cosmetico). Ja traz o
   `name` por looktype de graca.
2. **Manifesto da categoria**: campo `manifest` vem como `/assets-packs/categories/
   outfits-<gender>-<id>-<hash>.json`. Resolver com
   `manifest.replace(/^\/assets-packs/, "/game/asset-packs")`. O JSON tem
   `categories` -> pega `Object.values(categories)[0]` -> `.pages[0].image` (mesma
   regra de replace do prefixo) + `geometry{directions, frames, layers, width, height}`
   e um array `assets` (`assets` fica no TOPO do manifesto, nao dentro da categoria)
   com os retangulos de cada frame no atlas.
3. **Spritesheet**: `pages[0].image` e um **.webp lossless** (ex. Bulbasaur 4094x34,
   `directions:4 x frames:3`). Cada asset tem `source` no formato
   **`<frame>_<layerX>_<layerY>_<direcao>.png`** — ATENCAO: **direcao e o ULTIMO campo
   (1..4), frame e o PRIMEIRO (1..3)** (facil inverter e recortar de costas). Direcoes:
   **1=costas, 2=direita, 3=FRENTE/sul, 4=esquerda**. Sprite estatico de frente = frame
   1, direcao 3 -> `source` termina em `..._3.png` comecando em `1_`. O rect
   `assets[].frames[0]{page,x,y,w,h}` ja e o sprite CHEIO (1x1=32px, 2x2=64px) — nao
   monta tile. Pokemon tem `colorizable:false` (1 layer) -> recorte direto; treinadores
   sao colorizaveis (precisam das cores + upscaler `xbr4x` do bundle).

Implementado em piwdex (`scripts/bake-sprites.mjs`, `npm run bake:sprites`): baixa os
372 spritesheets, recorta o frame frontal com `sharp` e salva
`public/game-sprites/<looktype>.webp` (~1.5MB) + mapa `pokeId->looktype` em
`src/data/game-sprites.json`. `spriteUrl()` prefere a arte do jogo (nao-shiny); shiny e
faltas caem na PokeAPI; os 482 pokeIds mapeiam (variantes reusam o looktype base). Arte
proprietaria do jogo (mesma zona cinza de rehospedar que todo o ecossistema fan-tool).
Detalhe do recorte: cada asset ja e o sprite CHEIO por direcao/frame (1x1=32px,
2x2=64px) — nao precisa montar tiles; `assets[].frames[0]` da o rect `{page,x,y,w,h}`.

**Skins de player tambem recortam** (ago/2026): alem dos 372 `kind:"pokemon"`, o
indice tem `kind` `trainer` (9), `premium` (14), `clan` (60) e `hunt` (1) — as skins do
personagem. A skin atual do jogador e `characters/me.lookType` (ex.: 20029 = "Gamer VIP
Outfit", `kind:premium`, nao-colorizavel -> arte exata). O baker recorta essas junto
(mesmo frame frontal) e guarda `skins:{looktype:nome}` no `game-sprites.json`;
`skinSpriteUrl(lookType)`/`skinName(lookType)` resolvem o avatar no perfil da Conta.

Join ja conhecido: `looktype` liga criatura -> mapa (map-markers) E arte, e liga o
jogador -> skin.

## Pokebolas (dado FIXO so na API logada)

As pokebolas reais (Poke/Great/Super/Ultra/Idle/Master) **NAO estao no `items.json`
publico** (os "*ball" de la sao loot-lixo). O catalogo com **catchRate** so vem de
`GET /api/game/balls` (logado): **Poke 1, Great 2, Super 3, Ultra 4, Idle 5,
Master 255** (+ `priceGold`, `id`, `iconUrl`, `buyable`, `infinite`). Precos reais
(ago/2026): Poke 5, Great 20, Super 50, Ultra 130, Idle 400 (nao-compravel), Master 0
(nao-compravel). Icones em `/assets/markitems/<ball>.png`. Poke Ball e a mais
CUSTO-EFICIENTE por ponto de captura (catchRate/preco); premium compra velocidade, nao
eficiencia. O catchRate e FIXO/igual pra todos ->
da pra salvar 1x e usar sem conta (piwdex: `src/data/balls.json` + sync opcional no
ingest com `PIW_TOKEN`). Mas a **formula de captura absoluta NAO e publicada**: o
catchRate e um multiplicador -> so da pra falar em **eficiencia relativa** entre bolas
(Ultra ~4x menos bolas que Poke), nunca "% de captura". % real so calibrando com
`used-balls` da conta.

## Formula de stat / IV / poder / qualidade (verificada)

O stat final de cada atributo, dado base, IV, nivel e qualidade:

    stat  = round( (base + 2*IV) * (nivel/100) * qualidade^exp )
    poder = round( soma(6 stats) * qualidade )
    IV    = (stat / ((nivel/100) * qualidade^exp) - base) / 2   (inverte)

`exp` por stat: **HP e Speed = 0.95**; ATK, DEF, SpAtk, SpDef = **0.8**. O
multiplicador de IV e **2**. A "qualidade" que o jogo mostra (ex.: 1,8) e esse
multiplicador direto, nao um score 0-100. A raridade/qualidade e por INDIVIDUO
capturado (rolada na captura), nao um traco da especie — a especie so tem base
stats.

Conferido exato contra o jogo (Electrode nv54, qualidade 1.8, base HP60/Atk50):
HP 113 -> IV 29.9; Atk 78 -> IV 20.1; ...; IV Total 132.8; soma 679*1.8 = 1222 de
poder. Fonte: engenharia reversa do calculo do piwtools (funcao `Rg`/`oi`/`ww` no
bundle) + conferencia com valores reais. Implementado em piwdex (`src/lib/stats.ts`).

**Inverter pra achar o IV so e confiavel em nivel alto.** A formula do stat tem um
`round()`; inverter (achar o IV a partir do stat mostrado) e mal-condicionado quando o
stat e um numero pequeno — em nivel baixo o fator `nivel/100 * qualidade^exp` e minusculo,
entao um mesmo stat inteiro cabe uma FAIXA larga de IVs e a estimativa vira chute. Ex.:
Charizard lvl 1, Q1.82, o jogo mostra IV total 138/192 mas os stats (2,2,2,2,2,3) invertem
pra ~122 com IVs fracionarios sem sentido (34.9). O poder ainda bate (round(soma*Q)=24),
so a decomposicao em IV que nao da. Regra pratica: **so estimar IV com nivel ~20+**; abaixo
disso, avisar. O IV verdadeiro so existe cheio na conta logada (nao nos stats arredondados).
Implementado como aviso na calculadora e no Lab da Eevee do piwdex.

## Sistemas do jogo (pokepedia/systems)

O jogo documenta as mecanicas em HTML em `poke.idleworld.online/pokepedia/systems/*`
(SSR Next; `curl` com UA de navegador funciona, WebFetch leva 403). O util pra ferramentas:

- **Combate na hunt** (`/combat`): wild REFORCADO — HP **×5**, dano **×1.8**/hit, vantagem
  elemental +50% pros DOIS lados (a amplificacao ja conhecida: x2->x2.5, x4->x5.5,
  x0.5->x0.33). Comeca com 100 Small Potions + 100 Poke Balls; Auto-Potion cura abaixo de
  um limiar, Auto-Revive levanta.
- **Evolucao** (`/evolution`): NAO gasta ouro. Com Evolution Stones **mantem o nivel**; de
  graca **reseta pro Lv.1**. Desbloqueia quando o pokemon atinge o hunt level da especie
  alvo (so em Cerulean). Ate Lv.39 de hunt: 1 de cada pedra; Lv.40+: 4 pedras no total
  (divididas). IV/Quality/apelido mantidos, stats recalculam. **Implicacao pra rota: a
  especie escolhida E o pokemon — nao existe "voltar" a forma anterior por nivel.**
- **XP** (`/xp`): XP total do nivel L = `round(50/3 × (L³ − 6L² + 17L − 12))`, sem cap.
  XP/kill fixo por especie (esta no dado). Pokedex +25% (apos 100 kills), VIP, XP Boost e
  Streak (+0.1%/ponto) multiplicam por cima.
- **Quality** (`/quality`): captura de wild vai de 0.8 a **1.8** (teto); 2.0+ (Mythic/
  Ancient/Divine) so shiny/breeding. Quality pesa ~2× o IV no Power.
- **Nap Mode** (`/offline`): 5 min de sample no servidor; coleta 50% (ou 100% com Turbo
  2💎), cap 24h. So Poke Balls nao sao jogadas offline.

**Dano por hit e taxa de captura NAO sao publicados.** O piwtools mostra numeros de
captura, mas sao engenharia reversa autoral deles — nao copiar nem inventar (ver
[[Inspiração é na mecânica e no dado, não na interface do concorrente]]). No piwdex,
KOs/h, XP/h e ouro/h da rota sao ESTIMATIVA (kill-speed por DPS estimado vs HP reforcado,
constantes calibradas pra bater a ordem de grandeza). Taxa de captura real so da conta
logada (camada 3).

## O que NAO vem no JSON (curadoria/scraping)

- **Breeding** — nenhum endpoint.
- **Bosses** — sem endpoint proprio; catalogo so existe curado por terceiros (piwtools).
- **Formulas de mecanica** (offline, power, shiny odds, VIP, xp) — so em HTML de
  `poke.idleworld.online/pokepedia/systems/*`.

## Conexoes
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
