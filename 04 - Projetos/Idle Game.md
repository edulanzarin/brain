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

Chassi + **primeira fatia jogável (loop de hunt) rodando ponta a ponta**. O design
completo mora no GDD do próprio repositório (`docs/game-design.md`) — é a fonte da
verdade, não esta nota.

Fatia 2 do roadmap entregue: manda o personagem a uma região, acumula offline e
coleta encontros/vitórias/capturas/drops. O resolver (`src/lib/hunt.ts`) é função
pura com RNG semeado pelo id da sessão — server-authoritative, determinístico e à
prova de cliente (teste rodou idêntico duas vezes). Seed de 8 espécies-base
(animais) + 3 regiões com pressão/spawn; UI `/hunt` em Server Components/Actions com
relatório e inventário por PE. Verificado: typecheck limpo, GET /hunt 200, coleta
persiste indivíduos; dificuldade escala por região (Lv.12 ~28% vitória vs Lv.5 ~61%).
Já emergiu na prática o pilar "raridade ≠ PE" (Mítico PE 49 ao lado de Comum PE 158).

Branch `feat/hunt-loop` merjada na `main` (ff). Sem remote ainda.

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

Só links. O texto do aprendizado mora na nota, não aqui.

- [[Prisma 7 tira a URL do schema - vai pro config e pro adapter]]

Candidatos a virar nota quando reaparecerem em outro contexto:

- Conteúdo procedural coerente vem de compor peças curadas, não de gerar do zero.
- Determinismo no núcleo, IA só na borda criativa (relaciona-se com
  [[A definição em dado dirige o comportamento, não um caso no código]]).
- Resolver de progressão idle como função pura semeada = server-authoritative sem
  simular tick a tick (relaciona-se com [[A definição em dado dirige o comportamento, não um caso no código]]).

## Próximos passos

- [x] Fundação de dados: migration + seed de espécies-base e regiões.
- [x] Loop de hunt resolvido no servidor + UI de coleta (fatia 2).
- [ ] Captura por atrito de verdade (enfraquecer + janela garantida; hoje é chance direta).
- [ ] Time do jogador real: `partyPower` sai do líder + criaturas (hoje é constante 130).
- [ ] DNA + treino + atratores → primeira descoberta e Primordial (fatia 4).
- [ ] Compositor de sprites (um arquétipo) no PixiJS (fatia 5).
- [ ] Auth (hoje o player é o 'eduardo' fixo do seed).

## Conexões
- Usa: [[Infra]] · [[Design]]
- Mapa: [[Projetos]]
