---
tags: [tipo/projeto, projeto/idle-game]
criado: 2026-08-13
status: ativo
codigo_em: ~/Dev/idle-game
---

# Idle Game

> RPG idle de navegador, 2D top-down, estilo **Albion idle**: você não é uma classe,
> você é o que veste. O poder vem do gear (farmado e upado idle, offline incluso), a
> build vem das armas/peças que você combina, as skills saem do gear (trocáveis), a
> progressão é maestria por arma e os pets dão buff. A filosofia: "você não é uma
> classe, você é o que veste". (Começou como um idle de captura estilo Pokémon —
> resposta ao Poke Idle World — e pivotou pra Albion idle em ago/2026; design antigo
> arquivado em `docs/game-design-legacy-poke.md`.)

Código em: `~/Dev/idle-game`

## Estado atual

**BACKEND server-authoritative entregue (ago/2026) — a fatia decidida no GDD §11.**
O jogo deixou de ser mock no cliente: conta/personagem/mundo vivem no Postgres e o
cliente só renderiza e envia intenção. O que entrou, em quatro cortes na `main`:

- **Schema novo (Albion idle)** aposentando o Pokémon (espécie/indivíduo/DNA/
  atrator). Modelos: Account+Session, Character 1:1, ItemDef (catálogo data-driven)
  + InventoryItem unificado (stack de material e instância de gear com raridade/
  tier/atributos/enchant/slot), Mob + DropEntry (chance por valor) com variante
  **shiny** (Orc Negro -> item exclusivo), Zone pura (ZoneMob), HuntSession
  autoritativa, Pet (buff estruturado), MarketListing (NPC via flags do ItemDef).
  Migração dev-only (sem remote, dados só de seed): init nova, DB resetado, seed
  reescrito e idempotente (15 itens, 4 mobs, 4 zonas, conta+personagem demo).
- **Auth**: hash scrypt + sessão opaca em banco com cookie httpOnly; `/login` com
  abas Entrar/Criar conta; `/` e `/lobby` roteiam pela sessão.
- **Resolver autoritativo** como **função pura semeada** — virou o princípio
  [[Progresso idle é função pura do tempo semeada, não simulação tick a tick]].
  Resolve o Δt inteiro (offline = mesmo cálculo, teto 12h), porta o combate por
  stats do mock e aplica as regras da Hunt v2 (drop table por mob, shiny ~0,8% ->
  gear exclusivo escalado pelo nível da zona).
- **Store migrado pro servidor**: o reducer virou provider fino que hidrata o
  GameState do servidor; `dispatch` chama Server Actions que mutam o banco e
  devolvem o estado fresco; o idle sincroniza a hunt ativa a cada 4s.

Verificado (não suposto): build limpo; `/lobby` sem sessão -> 307 `/login`, com
sessão -> 200 renderizando herói+zonas do banco; integração resolve zona semeada
e persiste recompensa (ouro 250->831, xp, drops). Observação de balanceamento: com
o modelo fiel do mock, o herói inicial cai rápido (def baixa), então o offline
rende pouco até subir defesa/vida — ajuste fica pra uma fatia de balanceamento.
A UI/engine/cidade/hunt do cliente foram **mantidas** (só trocou a fonte de dados),
não redesenhadas — a UX plena da Hunt v2 (botão Casa, coletar-ao-voltar, seleção
de zonas vs. analisador) é a próxima fatia.

O histórico abaixo é do design ANTIGO (Pokémon), mantido como registro do caminho.

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

Fatia 4 também entregue — **DNA + treino + atratores → descoberta + Primordial**, o
coração do conceito virou mecânica. Treinar move o DNA de afinidade num espaço de
eixos; cruzar o "gate" de um atrator (limiar por eixo) evolui a criatura; se o
atrator era virgem, é descoberta: nasce a espécie no mundo (nó novo na árvore
global) e a criatura vira o Primordial (claim atômico, à prova de corrida). PE fica
separado — é qualidade, não decide a espécie. Verificado com a lógica real: um Lobo
treinado descobriu Lobo Espectral e depois Fenrir (Apex), Primordial nos dois.
Telas `/creature/[id]` (DNA, caminhos com progresso, treino) e `/pokepedia` (árvore +
descobridores). 7 atratores encadeados semeados (Lobo/Leão/Águia), nomes coerentes
pela gramática (base + domínio + tier).

O loop idle central agora está **fechado**: a hunt imprime a pressão da região no
DNA do líder ao longo do tempo (afinidade proporcional às horas), o líder ganha
nível pela xp e pode **evoluir/descobrir espécie OFFLINE** — verificado: um Leão
caçando 8h na Savana descobriu Leão Infernal (Primordial) sem treino manual.
HuntSession ganhou líder; UI com seletor de líder por região e bloco do líder no
relatório. É o pilar "o monstro é a história das batalhas dele" virando mecânica.

Primeira **prova visual** entregue: lobby jogável top-down no PixiJS (`/lobby`) —
herói andando por WASD/setas, câmera seguindo, e o **cenário inteiro em arte LPC
coerente com o personagem** (chão em tiles grama/terra/pedra + props carvalho/
pinheiro/barril/baú/pedra/placa). Os props do protótipo eram do PixelLab e não
casavam com o herói LPC — refeitos no próprio LPC (mesma família de assets) resolveu
de vez, e virou o caso novo do princípio [[Coerência em geração vem de âncora, não de liberdade]]
(coerência = restringir a fonte, mesmo só montando). `scripts/art/scenery.py`
recorta chão+props das folhas base do LPC (reproduzível, como `skins.py`). A tela
inicial virou menu do jogo (IdleRealm) com HUD-maquete de RPG.

**Engine própria + cidade jogável com UI de RPG idle.** O lobby monolítico virou
arquitetura em 3 camadas (ver [[Mundo imperativo e React se falam por eventos, não por referência]]):
`src/engine/*` (engine agnóstica ao jogo, dirigida por `MapDef`: núcleo Pixi
StrictMode-safe, TileMap, Actor direcional, Camera, Input, Collision AABB por
caixa-dos-pés, Interactables e EventBus tipado), `src/game/*` (a cidade como dados
+ CityScene + store em reducer PURO com tick idle) e `src/app/lobby/ui/*` (barra de
menu no topo, HUD do grupo herói+pet, chat, e painéis). A cidade é o **único lugar
andável** e virou uma cidade de verdade (56x42) em **grade de quarteirões**: uma
malha de ruas divide o mapa em blocos, o bloco central é a praça de pedra e cada
prédio ocupa um bloco com rua em volta (nada de cidade-quadrada; árvores só no anel
externo, com espaço de copa). Prédios LPC grandes e detalhados (parede 9-slice +
telhado tileado, texel 32px nativo, por `scripts/art/buildings.py`): taverna que
cura herói+pet, dois mercados e casas. **Mercado e taverna são LUGARES**, não botão
de menu — anda até o prédio e aperta E; a barra do topo só tem menus pessoais
(Personagem/Inventário/Caçadas). Dois mercados: **NPC** (compra/venda preço fixo) e
**de jogadores** (comprar ofertas de terceiros, anunciar itens seus — leilão
assíncrono, mock no cliente). **Hunts idle** sobem nível e dropam material mesmo com
o jogador parado. **A engine troca de cena** (`useGameEngine(location)`): cidade <->
mapa de caçada. **Hunt virou CENA, não menu**: cada zona tem uma arena onde o herói
(IA) vai sozinho até o monstro mais próximo, bate, o bicho morre (flash+fade) e
renasce, moeda flutua. Mapa da hunt é grande (32x22, câmera segue) e os mobs nascem
espalhados e renascem devagar. **Combate por STATS, sem ganho fixo por segundo**: os
mobs têm status (hp/atk/def/xp/loot em `game/mobs.ts`) e o dano do herói derruba o hp
— o ritmo EMERGE dos stats. O `store` simula isso num reducer PURO com PRNG semeado
no TICK (idle roda mesmo fora do mapa; o servidor herda o mesmo shape). Cada morte dá
xp, ouro e rola loot (vários "lixos" por mob pra vender no NPC); o herói toma dano por
status da zona (def importa; vida 0 = "caiu", cura na taverna). O painel virou
**Analisador de hunt** (derrotados/tempo/xp/ouro/loot/drops + coletar). Monstros LPC
(orc/esqueleto/slime/morcego, `scripts/art/mobs.py`). Props da cidade têm colisão. UI
ganhou moldura pixel (`.pixel-panel`), ícones pixel à mão (sem emoji no chrome) e
grama com textura. Decisão do Eduardo: engine "poderosa, reaproveitável,
modular"; mundo single-player (cada um no seu), mas com chat (e guerra no futuro).

**Estilo visual travado: pixel art, escala inteira (pixel-perfect), fonte pixel na
UI toda.** Decisão do Eduardo: "vamos trabalhar em pixel". **Sem emoji no chrome** —
os ícones da UI (elmo, mochila, espadas, moeda, caneca, barraca) são pixel art
desenhados à mão por mapa de pixels (`scripts/art/icons.py` → `public/icons`,
componente `<PixelIcon>`); a barra do topo é SÓ ícone (nome no tooltip). Fonte da UI:
**DotGothic16** (quadrada e legível — o Eduardo achou a Pixelify Sans ruim de ler);
Press Start 2P só no título. (Fonte: rejeitou Pixelify Sans e DotGothic16 por finas/
apertadas; ficou **Silkscreen** encorpada + letter-spacing.) Auditoria de sprites em folha de contato é o portão de
QA (ver [[Personagem pixel direcional se desenha em código, não se gera por IA]]).

**Pipeline de arte, decisão final: assets abertos LPC (não PixelLab, não código à
mão).** A jornada: PixelLab dá arte boa mas depende de API/cota (Eduardo não paga) e
as direções saíam inconsistentes (resolvi com `/rotate`, mas...); então testei
**gerar em código** (achei/vetei/instalei a skill `pixel-art-gen` em `~/.claude/`,
desenhei um cavaleiro 32x32) — mas ficou **quadrado/feio** e não casou com o cenário.
O Eduardo cravou o insight certo: **não é "um guerreiro", é Albion — o ITEM decide o
visual**, com skins-base padrão. Isso apontou pro encaixe perfeito: o **LPC**
(Universal LPC Spritesheet), onde o personagem é feito de **camadas** (corpo, roupa,
armadura, arma) — é literalmente o "gear = aparência" do Albion, de graça, qualidade
profissional, virando PNG no repo. `scripts/art/skins.py` compõe as camadas e
exporta as skins **masculina e feminina** (idle + walk 8 frames nas 4 direções);
GameCanvas troca skin no C. Licença LPC (CC-BY-SA 3.0 / GPL 3.0) exige atribuição →
`docs/CREDITS.md`. Ver [[Personagem pixel direcional se desenha em código, não se gera por IA]]
(que agora aponta o LPC como a via de qualidade+volume).

Aprendizados transversais desta jornada de arte:
- **A ferramenta de gerar asset ≠ dependência do jogo.** O que roda é o PNG no repo;
  PixelLab/skill/LPC são só o "editor" na hora de criar.
- **Personagem em CAMADAS (LPC) é o encaixe de um jogo onde o gear muda o visual** —
  a mesma decisão de design (Albion) escolhe a tecnologia de arte.
- Código à mão só compensa pra asset simples/único; pra volume+qualidade, pack aberto.
- (Manhas do PixelLab, se um dia voltar: [[PixelLab só mantém o personagem ao animar com image guidance alto]].)

**Virada de design fechada nesta sessão: o jogo virou um Albion idle.** O GDD foi
reescrito (`docs/game-design.md`); o antigo (captura estilo Pokémon) está arquivado
em `docs/game-design-legacy-poke.md`. Pilares novos:

- **Sem classe — a arma é a classe.** A build vem do gear equipado; trocar de arma
  troca o papel. As "classes" que o jogador sente são as linhas de arma. (Curioso: o
  Eduardo pediu "quero classes" e, na hora de escolher, foi no modelo *sem classe* —
  o que faz sentido no Albion, onde a maestria por arma É a classe.)
- **Gear é o motor de poder, farmado e upado idle.** Farm **misto** (escolha dele):
  base craftada com material (previsível) + itens únicos/sets caindo raro de farm
  longo e boss. Tiers T1..T8 + raridade.
- **Skills vêm do gear e são trocáveis** (arma + peças de armadura dão o loadout).
- **Maestria por arma no lugar de XP de classe** — sobe por uso (idle), destrava
  skills e tiers. É o que dá o "upar idle" sem trava de classe.
- **Pets dão buff** (um ativo): o animal potencializa (dano/drop/farm/maestria), não
  é coleção.

Sobrevive do design antigo: server-authoritative, idle como função pura semeada,
boss como portão, e a coerência visual por âncora. O código atual (`hunt`,
`creature`, DNA/atratores) implementa o design ANTIGO — será migrado/aposentado na
fatia 2 do roadmap novo. A virada também alivia o "não gerar sprite por IA": pro
herói/gear/pets, geração única e cacheada (presa a âncora) é viável.

Branches `feat/hunt-loop`, `feat/dna-attractors`, `feat/idle-imprint`, `feat/lobby`,
`fix/hero-face` e `feat/lobby-scenery-lpc` merjadas na `main` (ff, história linear).
Sem remote ainda.

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

> As decisões abaixo são do **design antigo (Pokémon)**, arquivado. Ficam como
> registro do que já se pensou; a maioria foi **substituída** pela direção Albion
> idle (ver "Estado atual" e o GDD novo). O que sobrevive está marcado.

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
  viável — virou o princípio [[Coerência em geração vem de âncora, não de liberdade]].
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
- [[PixelLab só mantém o personagem ao animar com image guidance alto]]
- [[Coerência em geração vem de âncora, não de liberdade]] — o "compor peças
  curadas em vez de gerar do zero" reapareceu (image guidance do PixelLab) e virou
  princípio na base.
- [[Progresso idle é função pura do tempo semeada, não simulação tick a tick]] —
  reapareceu no resolver Albion (2º contexto) e foi promovido à base.

Candidatos a virar nota quando reaparecerem em outro contexto:

- Determinismo no núcleo, IA só na borda criativa (relaciona-se com
  [[A definição em dado dirige o comportamento, não um caso no código]]).

## Próximos passos

- [x] Fundação de dados: migration + seed de espécies-base e regiões.
- [x] Loop de hunt resolvido no servidor + UI de coleta (fatia 2).
- [x] DNA + treino + atratores → descoberta e Primordial (fatia 4).
- [x] Ligar treino ao idle: a hunt imprime a pressão da região no DNA do líder (evolui/descobre offline).
- [x] Prova visual: lobby top-down no PixiJS com herói parado/andando (sprites LPC).
- [x] Unificar o cenário do lobby em LPC (chão + props da mesma família do personagem).
- [x] Reconciliar o GDD para a direção Albion idle (sem classe, gear é tudo, maestria, pets de buff).
- [x] Engine modular (engine/game/ui) + cidade maior andável com barra de menu, HUD, chat e painéis; taverna cura, hunts idle.
- [x] Hunt como cena (auto-combate) + combate por stats + analisador + loot de lixo; colisão de props; UI pixel (ícones, moldura, fonte Silkscreen, grama texturizada). Tudo **mock no cliente**.

**FATIA BACKEND = CONCLUÍDA** (ago/2026; detalhe no topo de "Estado atual" e no
GDD §9-11). Todos os quatro itens do escopo entraram na `main`:

- [x] **Contas + auth** (criação de conta, login, sessão em banco + cookie).
- [x] **Banco novo** (Postgres 16 + Prisma 7): Conta/Personagem/Item+Gear/
  Inventário/Mob+DropTable (raridade + shiny)/Zona/HuntSession/Mercado/Pet. Schema
  antigo (Pokémon) aposentado (`prisma/`, `src/lib/`, telas hunt/creature/pokepedia).
- [x] **Resolver autoritativo** (função pura semeada): combate, loot, shiny, nível
  de drop escalado pela zona, teto offline. Ver [[Progresso idle é função pura do tempo semeada, não simulação tick a tick]].
- [x] Store mock migrado pro servidor (cliente só renderiza + envia intenção).

Próximas fatias:

- [x] **Balanceamento de sobrevivência idle** — curva relativa ao nível (pressão
  por nível + mitigação % por defesa + regen idle): no seu nível o herói se
  sustenta (offline rende), acima do nível cai. Corrige o "cai em ~80s".
- [x] **Fluxo da Hunt v2 na UI** (GDD §10): botão Casa (coleta e volta), "Hunt"
  abre seleção de zonas, "Análise da Hunt" no mapa, trocar de zona coleta+migra,
  hunt não fica ativa na cidade.
- [ ] **Camada visual da Hunt v2**: no HuntScene, poucos mobs andando e batendo
  no herói (combate mútuo visível), respawn devagar. A lógica de combate mútuo já
  está no resolver autoritativo; falta a renderização.
- [ ] Depois: gear/equipar → skills por gear → craft/maestria → pets → bosses.

## Conexões
- Usa: [[Infra]] · [[Design]]
- Mapa: [[Projetos]]
