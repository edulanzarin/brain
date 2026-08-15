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

- **`sellValue` != `priceNpc`** na criatura, apesar de baterem em 464/482 casos.
  `sellValue` = ouro que voce **recebe** vendendo o bicho pro NPC; `priceNpc` = ouro que
  voce **paga** pra comprar na loja NPC. Divergem nos 18 exclusivos de loja (ex.:
  Aerodactyl custa 6,5 bi pra comprar e tem `sellValue: 0` = nao vende). Na UI, rotular
  como "Vende por" vs "Loja NPC" e tratar `sellValue: 0` como "nao vende" — mostrar so um
  numero confunde (foi o que o Eduardo estranhou).
- O `type` de um `attack` pode ser **`NEUTRAL`** (golpe sem tipo), fora dos 18 tipos
  canonicos de pokemon. Qualquer map por tipo (cor, icone) precisa de fallback.

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

## O que NAO vem no JSON (curadoria/scraping)

- **Breeding** — nenhum endpoint.
- **Bosses** — sem endpoint proprio; catalogo so existe curado por terceiros (piwtools).
- **Formulas de mecanica** (offline, power, shiny odds, VIP, xp) — so em HTML de
  `poke.idleworld.online/pokepedia/systems/*`.

## Conexoes
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
