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
- **Login Auth.js v5** com sessao JWT por cima do `pg` cru (sem adapter) — Google
  (condicional ao ambiente) + email/senha (bcryptjs). Padrao capturado em
  [[Auth.js sem adapter, a sessão JWT resolve o usuário no seu próprio SQL]]. Telas
  `/entrar` e `/criar-conta`, login/logout no header, gate em `/conta`.
- **Vinculo da conta do jogo saiu do cookie AES e foi pro banco** (`game_links`), preso
  ao usuario logado: tokens cifrados, refresh persistido, `status='expired'` quando o
  refresh falha -> a UI pede reconexao ("tenta manter, senao reconecta", pedido do
  Eduardo). `connect`/`collection`/`disconnect`/`market` operam por usuario.
- **Mercado saiu do /conta gratis** e virou `<MarketAdvisor>` reusavel, pronto pra area VIP.
Testado ponta a ponta (build de producao numa porta separada): cadastro/login por
credenciais emite sessao com id do banco, `/conta` abre logado e redireciona deslogado,
e `/api/collection` separa "nao logado" de "logado sem jogo vinculado".

## Infra

Slug `piwdex` · app `piwdex-app` na `4070` · banco `piwdex-db` na `5070` (Postgres 17,
entregue na camada 4: login + vinculo do jogo). Chassi e mapa de portas em [[Infra]].

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
  (Google + email/senha) convivendo com `pg` cru, sem adapter de ORM.

## Proximos passos

- [x] Camada 1: ingestao + pokedex + ficha + itens + indice reverso de drop.
- [x] Redesign visual pixel/neon + calculadora de IV/qualidade/poder (formula do jogo).
- [ ] Camada 2 (resto): Hunt Planner (lucro/hora, XP/h, melhor spot por item) e
  comparador de dois pokemons; formulas de XP/poder raspadas de `/pokepedia/systems/*`.
- [x] Breeding como camada curada: simulador `/breed` com colecao local e projecao.
- [ ] Bosses como camada curada (nao vem no JSON do jogo).
- [x] Camada 3 (leitura): companion logado por token; `/conta` completa (perfil, treinador,
  automacao, streak, breeding, inventario+deposito, bolas) + bookmarklet de conexao.
- [ ] Camada 3 (resto): pokemons ativos individuais via WebSocket (`ws12`/`ws47`).
- [x] Camada 4: login do site (Auth.js Google+email, Postgres) + vinculo do jogo por usuario.
- [ ] Camada VIP: gateway Mercado Pago (~R$15,90/mes) + area `/vip` (hospeda o mercado, gated).
- [ ] Extensao de navegador (auto-conexao + robo), rodando no dominio do jogo.
- [ ] Deploy e dominio piwdex.com.br; job de ingestao recorrente (servico do compose).
- [ ] colorGroup no `.obsidian/graph.json` (pendente — regras de cor no CLAUDE.md).

## Camada VIP + assinatura (decidido ago/2026, a construir)

- **Arte real do jogo** self-hostada (`npm run bake:sprites` -> `public/game-sprites/<looktype>.webp`,
  372 pokemons); `spriteUrl` prefere a arte do jogo. Detalhe em [[Poke Idle World - endpoints publicos de dados]].
- [x] **Login piwdex**: Auth.js com **Google + email/senha**, conta SEPARADA da do jogo
  (linka pelo token, agora presa ao usuario). Chassi Postgres na 5070. **Feito (camada 4).**
- [x] **Mercado saiu do /conta gratis** e virou componente reusavel `<MarketAdvisor>`.
  Falta a **area `/vip`** que o hospeda, gated por assinatura.
- **Assinatura ~R$15,90/mes** destrava o VIP. Gateway: **Mercado Pago** (Pix nativo, BR).
  Flag `vip` (+ `vip_ate`) ja existe no `users` e ja chega na sessao — falta o gateway.
- **Automacao/robo via extensao** de navegador: roda no dominio do jogo -> usa a sessao logada ->
  chama a API sem captcha. Mesma extensao faz auto-conexao E farm/venda/compra. Bookmarklet e o
  passo 0; extensao e o produto.

## Conexoes
- Usa: [[Design]] · [[Infra]]
- Referencia: [[Poke Idle World - endpoints publicos de dados]]
- Mapa: [[Projetos]]
