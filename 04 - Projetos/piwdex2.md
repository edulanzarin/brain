---
tags: [tipo/projeto, projeto/piwdex2]
criado: 2026-08-20
status: ativo
codigo_em: ~/Dev/piwdex2
---

# piwdex2

> Reescrita do [[piwdex]] — dex e ferramentas para **Poke Idle World**. A premissa mudou:
> a dex não é uma galeria de sprites, é uma **ferramenta de consulta**. A pergunta que ela
> responde não é "como é o Bulbasaur", é "quem apanha de Fogo, dropa Bulb e dá pra encarar
> no nível 40".

Código em: `~/Dev/piwdex2` · remote `git@github.com:edulanzarin/piwdex2.git`
Par de portas: **4071** (app) / **5071** (banco, reservado).

## Por que reescrever

O Eduardo rejeitou o visual do piwdex depois de cinco passadas de refino. O diagnóstico não
foi "a estética está feia" — foi **densidade**: a moldura do aparelho Pokédex comia a área
útil, e a barra horizontal de filtros só comportava três controles. Isso virou
[[Quantos filtros existem é decisão de layout, não de produto]].

Decisão de forma: mantém a identidade pixel/neon escura (é ferramenta de jogo, não
dashboard — ver [[Estética é por projeto, princípio de design é que se reusa]]), mas mata a
moldura e põe **trilho de filtros fixo**. A fonte pixel fica só em rótulo curto em caixa
alta; o dado vai no mono, senão a densidade morre de novo.

## Estado atual

**Camada 1 entregue, verificada em build de produção e no navegador.**

- **Sistema de design**: tokens em `globals.css` (superfície/linha/texto/acento, raio
  pixel, brilho neon) e **22 primitivas** em `components/ui` — botão, campo, select,
  multi-select com modo E/OU, combobox, modal, popover em portal, faixa de dois polegares,
  segmentado, abas, chip, checkbox, switch, paginação, esqueleto, vazio, tooltip, sprite,
  medidor, pokébola. Ícones em **pixel art de verdade**, desenhados num grid de texto 8x8,
  incluindo os 18 símbolos de tipo. Sem emoji.
- **Pokédex** com **17 filtros** (tipo com E/OU, raridade, origem, estágio, região, faixas
  de nível/valor/XP/stats/poder, pool natural vs TM, fraco contra, resiste a, índice
  reverso de drop, variantes), grid ⇄ tabela ordenável, e **estado inteiro na URL**.
- **Card** com espinha de stats — seis barrinhas que entregam o PERFIL da espécie de
  relance. A régua virou [[A régua de um medidor é percentil, não máximo]].
- **Ficha da espécie**: stats, defesa com multiplicador, cobertura ofensiva, linha
  evolutiva, pontos de caça, golpes (natural x TM separados) e drops com a chance real.
- **Home** que abre com os números do catálogo do momento e três recortes vivos.

Motores puros portados e vivos em `src/lib`: `stats`, `xp`, `typing`, `rarity`,
`catch-law`, `balls`, `combat`, `breeding`, `meta`, `boost`. Com eles no lugar,
Calculadora / Hunt / Breeding / Meta viram trabalho de interface, não de pesquisa.

## Decisões que valem lembrar

- **O `DexEntry` é flat.** Mandar a `Creature` inteira (40 golpes, 2.657 registros de
  loot) pro navegador custaria ~1MB de payload que o card nem usa. Derivar uma vez no
  servidor, filtrar barato no cliente.
- **`valueFromNpc` declara qual grandeza o "valor" carrega.** `sellValue` (o que o jogo
  paga por abate) e `priceNpc` (preço do cassino) não se comparam: sem a bandeira, um
  ranking de "paga mais por abate" coroava o Aerodactyl com 6,5 bilhões — que é o que ele
  CUSTA, e ele nem se caça. Segunda aparição da mesma armadilha, ver
  [[Chip que serve a duas grandezas declara qual delas mostra]].
- **Catálogo ao vivo com estado visível**: ETag a cada visita, download só quando o jogo
  mexeu, e o selo troca de `AO VIVO` pra `SNAPSHOT` quando a fonte cai — em vez de servir
  dado velho fingindo estar ao vivo.
- **Sem coluna de cooldown**, de propósito: o valor do catálogo é o cooldown BASE e a
  velocidade do bicho o encurta no jogo.

## O robô está parqueado, não portado

Pedido do Eduardo: trazer o bot pro piwdex2 pra pensar no substituto depois. Ele vive em
`parked/`, **fora de `src/` e no `exclude` do tsconfig** — não compila, não entra no build.
Religar direto arrastaria Auth.js, Postgres, Mercado Pago e a UI antiga inteira, ou seja,
justamente o design que motivou a reescrita, e ainda travaria a decisão em aberto.

O que não pode se perder está apontado no `parked/README.md`: o protocolo do WebSocket
cravado por engenharia reversa (único caminho pros pokémons individuais — ver
[[Quando a REST não expõe o dado, o WebSocket do mesmo sistema entrega]]), a sondagem
paralela de shard com early-exit, e o porquê do login ser por token e não por senha.

## Próximos passos

1. Itens com índice reverso (o motor já existe em `data.ts`).
2. Calculadora de IV/Quality/Poder sobre `stats.ts`.
3. Hunt, Breeding e Meta sobre os motores já portados.
4. Decidir o que substitui o robô. Até lá, `parked/` fica como está.
5. As páginas hoje são dinâmicas (`ƒ`) porque a fonte roda com `cache: "no-store"` — o
   frescor é gerido no `source.ts`. Se virar gargalo de CDN, é aqui que se mexe.

## Conexões
- Substitui: [[piwdex]]
- Usa: [[Design]] · [[Infra]] · [[Frontend]]
- Aprendizados: [[Quantos filtros existem é decisão de layout, não de produto]] ·
  [[A régua de um medidor é percentil, não máximo]] ·
  [[Conteúdo do servidor não pode nascer invisível esperando o cliente]] ·
  [[Sticky gruda no container que rola, não na janela]]
- Referência: [[Poke Idle World - endpoints publicos de dados]] · [[Poke Idle World - regras de breeding]]
- Mapa: [[Projetos]]
