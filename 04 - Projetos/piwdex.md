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

UI em Next 16 (App Router): home, Pokedex com busca/filtro por tipo e raridade, ficha
completa (stats, fraquezas, evolucao, drops com % real, onde cacar, moves) e paginas de
item com o indice reverso. Verificado: `tsc` limpo, `next build` gera 817 paginas
estaticas, server de producao responde 200 e as secoes-chave renderizam (drop com %,
"onde dropa" com fontes). Sprites vem da PokeAPI pelo `pokeId`; icones de item do host
do jogo — nao hospeda asset.

Sem remote ainda (so commit local).

## Infra

Slug `piwdex` · app `piwdex-app` na `4070` · banco `piwdex-db` na `5070` (reservado,
banco entra na camada 3). Chassi e mapa de portas em [[Infra]].

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

## Proximos passos

- [x] Camada 1: ingestao + pokedex + ficha + itens + indice reverso de drop.
- [ ] Camada 2: calculadoras (lucro/hora de farm, melhor spot por item, comparador de
  stats); formulas raspadas de `/pokepedia/systems/*`.
- [ ] Bosses e breeding como camada curada (nao vem no JSON do jogo).
- [ ] Camada 3: companion logado (proxy da API JWT + WebSocket). Aqui entra o chassi
  Postgres + Docker compose + auth.
- [ ] Deploy e dominio piwdex.com.br; job de ingestao recorrente (servico do compose).
- [ ] colorGroup no `.obsidian/graph.json` (pendente — regras de cor no CLAUDE.md).

## Conexoes
- Usa: [[Design]] · [[Infra]]
- Referencia: [[Poke Idle World - endpoints publicos de dados]]
- Mapa: [[Projetos]]
