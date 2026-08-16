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
