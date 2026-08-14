---
tags: [tipo/projeto, projeto/idle-game]
criado: 2026-08-13
status: ativo
codigo_em: ~/Dev/idle-game
---

# Idle Game

> RPG idle de navegador, 2D top-down. Cada jogador captura, treina, evolua e
> comercializa criaturas. As espécies não vêm prontas: emergem da trajetória que
> os jogadores dão às suas criaturas, formando uma árvore evolutiva global. A
> filosofia é "o jogo dá as regras; os jogadores criam as espécies, a economia e
> a história". Nasceu como resposta ao Poke Idle World (poke.idleworld.online),
> que tem o loop idle padrão sem nenhuma mecânica surpreendente.

Código em: `~/Dev/idle-game`

## Estado atual

Conceito + chassi. Nada jogável ainda. O design completo mora no GDD do próprio
repositório (`docs/game-design.md`) — é a fonte da verdade, não esta nota. Chassi
de infra montado (Next 16 + Prisma 7 + Postgres 16 + PixiJS, Docker standalone) e
schema Prisma inicial modelando os eixos centrais. Commit inicial na `main`, sem
remote ainda.

## Infra

Slug `idle-game` · app `idle-game-app` na `4060` · banco `idle-game-db` na `5060`.
Compose com app + db + rede `idle-game-net`, imagem standalone com entrypoint
(migrate + start). Chassi e mapa de portas em [[Infra]].

## Stack

Next.js 16 (App Router) · React 19 · Prisma 7 (driver adapter `@prisma/adapter-pg`)
· Postgres 16 · PixiJS (render do canvas 2D) · TypeScript · Tailwind 4.
**Server-authoritative**: todo o estado do mundo vive no Postgres; o cliente só
renderiza e envia intenções.

## Decisões importantes

- **Espécie global vs. indivíduo é o eixo de tudo.** Espécie é do mundo (nó da
  árvore evolutiva, de ninguém); indivíduo pertence ao jogador e é o que se vende.
  Toda a economia nasce dessa separação.
- **PE (Potencial Evolutivo) independente de raridade.** Raridade ≠ poder ≠
  potencial — cria decisões reais e vários eixos de valor no mercado.
- **Evolução determinística por atrator.** O DNA vive num espaço contínuo; regiões
  desse espaço ("atratores") cristalizam em espécies. Mesma trajetória → mesma
  espécie. Primeiro a entrar num atrator virgem descobre a espécie (vira
  Primordial). Resolve "como o sistema decide o que é espécie nova".
- **Coerência e viabilidade visual vêm de COMPOSIÇÃO, não de geração por IA.**
  Esqueletos de arquétipo desenhados à mão + biblioteca de peças/FX que o DNA
  seleciona; spritesheet cacheado por espécie. A IA só gera nome/lore/retrato
  dentro de uma gramática fixa (base + domínio + tier). A evolução herda o rig do
  pai, então descende visivelmente dele. É a decisão técnica que faz o projeto ser
  viável — e candidata a virar princípio na base quando reaparecer.
- **Descoberta é cara de propósito** (exposição de ambiente + catalisador de boss +
  PE mínimo), senão a árvore vira ruído de milhares de variantes sem valor.
- **Região = pressão evolutiva = objeto da guerra.** Cada região imprime DNA;
  controlar território na guerra é controlar o acesso a caminhos evolutivos. Amarra
  farm, evolução e PvP numa decisão só.
- **Server-authoritative por design.** O jogo de referência deixa farm no cliente e
  por isso é trivial de burlar; aqui o servidor decide loot/captura/evolução.

## Aprendizados (viraram notas)

Só links. Nada promovido à base ainda — o projeto é conceito. Candidatos a extrair
quando reaparecerem (o `[[Idle Game]]` linkaria como "Visto em"):

- Conteúdo procedural coerente vem de compor peças curadas, não de gerar do zero.
- Determinismo no núcleo, IA só na borda criativa (relaciona-se com
  [[A definição em dado dirige o comportamento, não um caso no código]]).

## Próximos passos

- [ ] `npm install` + primeira migration (fundação de dados: seed de espécies-base e regiões).
- [ ] Loop de hunt resolvido no servidor (sem render) — prova o núcleo idle.
- [ ] Indivíduo + PE + captura por atrito.
- [ ] DNA + treino + atratores → primeira descoberta e Primordial.
- [ ] Compositor de sprites (um arquétipo) no PixiJS.

## Conexões
- Usa: [[Infra]] · [[Design]]
- Mapa: [[Projetos]]
