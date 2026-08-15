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

**Breeding Simulator (5a passada, ago/2026):** o Eduardo pediu a parte de breeding "tipo
no piwtools tem". Ferramenta `/breed` que faz o que o piwtools tem e integra ao app:
- **Tutorial** com as regras separadas em confirmadas / provisorias / nao suportado —
  nada de teto ou tier Shiny inventado (ver [[Poke Idle World - regras de breeding]]).
- **Colecao local** de Pokemon em `localStorage` (criar/editar/filtrar/ordenar), com
  seletor de especie por sprite. A simulacao NUNCA consome a colecao.
- **Simulador de par**: valida mesma especie + diff de Quality <= 0.150, projeta a
  heranca de IV (pai de maior Quality; empate -> Slot 1), a faixa/media de Quality por
  modo (Free EV 0.0096 / Pheromone EV 0.1875, batem exato), teto 2.600 no normal (Shiny
  bruto), Double Stones e o custo (R$ + Stones divididas por tipo + Pheromones).
Motor puro em `src/lib/breeding.ts`; UI no visual pixel/neon do app (icone de ovo na nav
e na home, i18n pt/en/es). Regras da especie sao **camada curada** — nao vem no JSON.

Verificado: `tsc` limpo, `next build` gera 822 paginas estaticas (incl. `/breed`
prerenderizada), e a calculadora e as medias de breeding batem exato com o jogo. Sem
remote ainda (so commit local; branch `feat/breeding` -> `master` fast-forward).

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
- [[Poke Idle World - regras de breeding]] — regras curadas do breeding (nao vem no
  JSON): mesma especie, diff Quality 0.150, heranca de IV, tabelas de ganho e custos.

## Proximos passos

- [x] Camada 1: ingestao + pokedex + ficha + itens + indice reverso de drop.
- [x] Redesign visual pixel/neon + calculadora de IV/qualidade/poder (formula do jogo).
- [ ] Camada 2 (resto): Hunt Planner (lucro/hora, XP/h, melhor spot por item) e
  comparador de dois pokemons; formulas de XP/poder raspadas de `/pokepedia/systems/*`.
- [x] Breeding como camada curada: simulador `/breed` com colecao local e projecao.
- [ ] Bosses como camada curada (nao vem no JSON do jogo).
- [ ] Camada 3: companion logado (proxy da API JWT + WebSocket). Aqui entra o chassi
  Postgres + Docker compose + auth.
- [ ] Deploy e dominio piwdex.com.br; job de ingestao recorrente (servico do compose).
- [ ] colorGroup no `.obsidian/graph.json` (pendente — regras de cor no CLAUDE.md).

## Conexoes
- Usa: [[Design]] · [[Infra]]
- Referencia: [[Poke Idle World - endpoints publicos de dados]]
- Mapa: [[Projetos]]
