---
tags: [tipo/projeto, projeto/piwdex2]
criado: 2026-08-20
status: ativo
codigo_em: ~/Dev/piwdex
---

# piwdex2

> Reescrita do [[piwdex]] — dex e ferramentas para **Poke Idle World**. A premissa mudou:
> a dex não é uma galeria de sprites, é uma **ferramenta de consulta**. A pergunta que ela
> responde não é "como é o Bulbasaur", é "quem apanha de Fogo, dropa Bulb e dá pra encarar
> no nível 40".

Código em: **`~/Dev/piwdex`** · remote `git@github.com:edulanzarin/piwdex.git`
Par de portas: **4071** (app) / **5071** (banco, reservado).

**No ar desde 22/08/2026, em piwdex.com.br.**

> **Atenção ao caminho: ele inverteu.** O projeto se chama piwdex2 e o código dele mora
> no repositório do **piwdex**. Foi assim que a reescrita assumiu o deploy que já rodava,
> sem migrar domínio. `~/Dev/piwdex2` virou `~/Dev/piwdex2-arquivo` (com um
> `ARQUIVADO.md` dentro dizendo para onde ir) e o repo `edulanzarin/piwdex2` foi
> arquivado no GitHub. Editar lá e dar push não muda o site.
>
> O nome do projeto continua piwdex2 no Brain de propósito: renomear quebraria os links
> de todas as notas que apontam pra cá, e o nome nunca foi o do repositório — era o do
> sistema. O [[piwdex]] segue sendo outra coisa: a versão 1, com o robô, arquivada na
> tag `v1.0.0`.

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

**Camadas 1 e 2 entregues, verificadas em build de produção.**

- **Sistema de design**: tokens em `globals.css` (superfície/linha/texto/acento, raio
  pixel, brilho neon) e **26 primitivas** em `components/ui` — botão, campo, select,
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
- **Itens** com o índice reverso: 428 itens, 10 filtros, grid ⇄ tabela e estado na URL.
  A ficha de cada item responde de onde ele vem — quem dropa, com que chance real,
  quantos abates custa uma unidade, em que áreas do mapa e quanto ele soma de ouro por
  abate. Carta de shiny e disco de TM não caem de ninguém, então a ficha deles lê a
  resposta do próprio nome e aponta pra espécie ou pro golpe.

- **Calculadora de IV**: inverte a fórmula verificada pra achar o atributo que o jogo
  esconde, projeta em qualquer nível e mostra o quanto falta pro IV perfeito. Ela abre com
  o **"Como usar"** — resumo sempre visível, seis passos sob demanda.
- **Hunt**: mede TODO alvo do jogo contra o pokémon que você tem — não existe "a
  melhor hunt do jogo", existe a melhor hunt pra ele. Três vistas sobre o mesmo
  lutador: a **rota** (a subida em faixas até o nível alvo, com quantas horas cada
  faixa custa pela curva de XP), o **farm de ouro** (alvo em ouro, não em nível:
  quanto você quer juntar, quantas horas leva no melhor spot, e de onde o ouro vem)
  e a **tabela inteira** (342 alvos, ordenável por XP/h, ouro/h, abates/h,
  efetividade e segurança, com ficha por hunt). O diferencial está no motor: o
  rendimento é EFETIVO — se a hunt te derruba, o tempo parado na Joy já saiu do
  XP/h antes da lista ser ordenada.
- **Anúncios (preparado, desligado)**: o site nasce sem anúncio nenhum — sem
  `NEXT_PUBLIC_ADSENSE_CLIENT` não há script, `<ins>` nem espaço reservado.
  Ligado, entram um card intercalado a cada 12 na grade da dex e dos itens e uma
  faixa antes do rodapé; nunca dentro de painel de ferramenta. Vieram junto
  `robots`, `sitemap` com as ~900 fichas dinâmicas, `ads.txt` e a página de
  privacidade — os quatro que a revisão do AdSense cobra e o site não tinha.
  Mecânica em [[Slot de anúncio no App Router precisa de casca estável e filho keyado]],
  forma em [[Anúncio em feed não pode vestir a roupa do conteúdo]].
- **Apoio**: faixa no rodapé de todas as páginas (recado curto de um lado, botão
  do outro) e balão dispensável que espera a pessoa usar o site. O caminho de graça
  — o código de indicação no jogo — vem primeiro e sempre existe; o link de
  pagamento nasce vazio em `lib/apoio.ts` e, vazio, não desenha botão. Detalhe e
  porquê em [[Pedido de apoio entra depois do valor, e nunca ao lado de si mesmo]].
- **Meta**: três vistas sobre o catálogo — **tier list** com o corte por NOTA (não
  por posição na fila: num jogo que recebe patch, espécie buffada tem que poder
  subir sem alguém descer pra abrir vaga), **duelo** entre dois indivíduos com
  nível e quality (a tier list compara espécies; dois Gyarados diferentes são dois
  pokémon diferentes) e **tipos**, o panorama ofensivo pra montagem de time. O
  perfil de cada espécie é modal, com os dois eixos da nota, o percentil de cada
  stat e quem derruba quem — medindo os dois lados do duelo, não só quem tem o
  tipo certo.
- **Breeding**: valida o par enquanto se digita (mesma espécie, Quality a até 0.150),
  projeta o ovo — sorteio de Quality com as quatro probabilidades, IV herdado, stats
  reais e custo — e planeja quantos breeds faltam até a Quality alvo. Estado do par na
  URL, estante de pokémon salvos no `localStorage`.

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
  velocidade do pokémon o encurta no jogo.
- **"Cai de alguém" e "dá pra farmar" são perguntas diferentes.** 54 itens só caem de
  espécies sem ponto no mapa (quem só evolui ou só vem do cassino): a melhor fonte do
  item é a de maior chance ENTRE AS CAÇÁVEIS, e a tela diz quando não existe nenhuma.
  Sem isso a lista promete uma caçada que não existe.
- **Ouro por abate** (`chance × quantidade média × valor de NPC` na melhor fonte
  farmável) é o número que decide se vale parar pra pegar o item — um drop de 1.344 que
  cai 94% das vezes rende mais que um de 50.000 que cai 0,4%. A ficha mostra a fórmula
  junto, senão vira número de autoridade que ninguém confere.
- **Formulário que É a tarefa ocupa a largura; trilho estreito é pra filtro.** A
  calculadora nasceu com a entrada num trilho de 360px, copiando o da dex — e o Eduardo
  reprovou o design. O trilho serve input que ACOMPANHA um conteúdo (a lista filtrada é o
  que importa); numa calculadora o formulário é o conteúdo. Seis campos de stat espremidos
  em duas colunas viravam rolagem, e o resultado saía em barras de 4px que ninguém compara
  de relance. Vale como sinal: se o resultado da tela depende do que se digitou nela, o
  campo é figura, não moldura.
- **Nota carrega só o que a pessoa não sabe.** O Eduardo cortou um "está no 50" que
  aparecia ao lado do campo de nível onde ele mesmo tinha escrito 50. A regra que saiu:
  valor vindo de input, rótulo que o campo já tem e instrução que a label já dá são
  ruído — fica o que a tela sabe e ela não (de onde o número saiu, o que ele não conta,
  o que fazer quando não fecha). E aviso vai em **itálico**, que separa a voz da
  ferramenta do dado que ela apresenta. Virou a primitiva `Note` e o princípio
  [[Nota carrega só o que a pessoa não sabe]].
- **O IV estimado é FAIXA, não ponto.** O stat que o jogo mostra já veio arredondado, e
  o fator `(nível/100) × quality^exp` é tão pequeno em nível baixo que meia unidade de
  stat vale 8 pontos de IV — um Electrode nível 5 com IV 32 de verdade estima 34,3 no
  ponto. Isso também derruba a validação óbvia: entrada impossível é `piso da faixa > 32`,
  não `ponto > 32`. Virou
  [[Estimativa que inverte valor arredondado é faixa, não ponto]].
- **Ferramenta tem manual; catálogo não.** Dex e Itens abrem cheios e se explicam de
  olhar; a calculadora abre VAZIA, pedindo seis números que moram na tela do jogo — sem
  manual, a tela mais útil do site é a que mais parece inacabada. O `HowTo` é um
  `<details>` (abrir/fechar é chrome, não dado: sem estado de React, o bloco continua
  componente de servidor e o texto nasce no HTML), com o resumo fora do fechado pra não
  empurrar a ferramenta pra fora da primeira dobra. O texto mora em `lib/how-to.tsx` e
  importa `IV_MAX` em vez de digitar 32. Virou
  [[Tela que abre vazia tem que ensinar, tela que abre cheia não]] e
  [[Manual de ferramenta é resumo visível com passo a passo sob demanda]] ·
  [[Custo de processo aleatório se orça pela cauda, não pela média]] ·
  [[Distribuição exata sai de programação dinâmica, não de Monte Carlo]] ·
  [[Peça o que a fonte mostra, não o que você precisa]] ·
  [[Medidor de razão nomeia a grandeza e mostra os operandos]] — e Hunt,
  Breeding e Meta já nascem com uma constante reservada lá.
- **O planejador de breeding responde com distribuição, não com média.** A versão do
  piwdex antigo fazia `ceil(delta / ganho médio)` — um número só, pra um processo
  sorteado. No Free o ganho é 0.005 em metade dos sorteios, então quem orça pela média
  começa uma corrente de 53 breeds (R$ 106 milhões) com dinheiro pra 53 e para na 58. O
  `breed-plan.ts` resolve **exato** por cadeia absorvente: os ganhos de cada modo são
  múltiplos de um mesmo passo (mdc 5 milésimos no Free, 50 no Pheromone), então a
  unidade vira o mdc, tudo soma em inteiro e da mesma varredura saem melhor caso,
  mediana, p90, média e a sobra que o teto 2.600 engole. Conferido contra Monte Carlo de
  400 mil rodadas: p50, p90 e sobra idênticos, média dentro de 0,14%; pior caso realista
  8 ms. Virou [[Custo de processo aleatório se orça pela cauda, não pela média]] e
  [[Distribuição exata sai de programação dinâmica, não de Monte Carlo]] ·
  [[Peça o que a fonte mostra, não o que você precisa]] ·
  [[Medidor de razão nomeia a grandeza e mostra os operandos]].
- **O breeding pede STATS, não IV — porque IV é o que o jogo esconde.** A primeira
  versão da tela tinha seis campos de IV, e o Eduardo cortou: o jogo mostra "Ataque
  1000", e saber se aquilo é IV 32 ou IV 27 é justamente a conta que a Calculadora já
  fazia. Era pedir o resultado como se fosse a entrada. A inversão virou
  `lib/iv-reading.ts` e as **duas ferramentas passam por ela** — duas leituras separadas
  podiam divergir sobre o mesmo pokémon, e ferramenta que se contradiz entre telas perde
  a única coisa que tem. O corolário é o que dá trabalho: **a dúvida viaja junto com o
  valor**. O filho herda o IV do pai, então herda a faixa daquele pai — leitura larga
  aparece como faixa no ovo, o total vira "106–144" em vez de "131", e leitura impossível
  (nenhum IV entre 0 e 32 explica os stats) PARA o ovo em vez de publicar o número travado
  no teto. Conferido por round-trip: 20.000 pokémon gerados pela fórmula e lidos de volta,
  IV real dentro da faixa em 100% dos casos e ponto exato em 100% das 94,6% de leituras
  cravadas. Virou [[Peça o que a fonte mostra, não o que você precisa]].
- **O medidor do par nasceu ilegível, e quem reprovou foi quem sabe a regra.** A faixa
  do veredito dizia `DIFERENÇA [barra] 0.000 / 0.150` e o Eduardo perguntou o que aquilo
  era — ele, que conhece o limite do jogo de cor. Faltavam três coisas: a grandeza
  ("diferença de Quality"), os operandos ("Quality 2.100 e 2.020") e o denominador em
  palavra ("de 0.150 que o jogo permite"). Virou
  [[Medidor de razão nomeia a grandeza e mostra os operandos]].
- **Três custos do breeding que nem o jogo nem a versão anterior mostravam.** (1) A
  corrente consome **N+1 pokémon** da espécie — cada breed come dois e devolve um, e esse
  é o custo que ninguém orça. (2) No Pheromone o ganho **mínimo** já é 0.150, que é o
  limite de diferença: metade dos sorteios deixa o filho **órfão**, sem par possível na
  estante de hoje. É número, não adjetivo — 50%. (3) O filho herda a distribuição de IV
  **inteira** do pai de maior Quality, então a tela avisa quando o descartado era o
  melhor, com quantos pontos vão pro lixo.
- **Cor de dado vinda de módulo portado é fronteira.** `RISK_COLOR` e `TIER_COLOR`
  chegaram com os tokens do piwdex 1 (`--green`, `--yellow`, `--cyan`), que aqui não
  existem — e `var()` desconhecida cai pro valor herdado em vez de falhar. "SEGURO"
  e "LETAL" saíam da mesma cor branca, sem um aviso sequer. Virou
  [[Token de cor que não existe vira cor herdada, sem erro]].
- **A Hunt tem dois tempos, e isso é a decisão de forma dela.** ENTRADA (o pokémon
  e o cenário) só vale depois do botão; RESULTADO (filtro, ordem, nível alvo)
  responde ao vivo. Não é custo de rede — a conta roda no navegador em dezenas de
  ms —, é que recalcular a cada tecla troca a resposta no meio de "1,8", e que o
  que exige comitar é o INSUMO, não a tela. O botão carrega o estado (calcular →
  calculado → recalcular) e a espera tem piso de 700ms, senão o loader pisca sem
  ser lido. Registrado em
  [[Consulta pesada executa por botão, não por mudança de filtro]].
- **A captura entra LÍQUIDA no ouro.** O jogo gasta uma bola por abate,
  capturando ou não. Somar só o que a captura rende (o que o piwdex 1 fazia)
  esconde a metade que importa: com Ultra Ball a 130 de ouro num alvo de chance
  0,17%, as bolas torram mais do que a venda devolve. O número fica negativo, e
  isso é resposta.
- **O Tipo do Dia é refeito drop a drop, não somado no total.** O bônus multiplica
  a CHANCE de cada drop e a chance tem teto: alvo cujo loot já sai em 95%
  aproveita 5% do bônus, não 20%. Por isso o payload leva o loot cru de quem tem
  ponto no mapa, em vez do ouro já somado.
- **A economia entra pelos campos `goldEV` e `xp` do alvo.** Tipo do Dia e captura
  são escolha do jogador; o motor de combate não conhece nenhum dos dois. Trocar os
  campos antes de chamar o motor faz o ouro/h e o XP/h já saírem completos — sem um
  segundo caminho de cálculo que um dia discordaria do primeiro.
- **Os golpes de TODO MUNDO viajam pro navegador, em tupla.** O combate tem dois
  lados: sem o moveset do selvagem não dá pra estimar o que você TOMA, e a tela
  promete XP/h de uma hunt que te mata. Com nome de campo em JSON são 133KB de
  chave repetida; em tupla, 77KB — 55KB no fio depois do gzip, o mesmo peso da dex.
- **O catálogo declara `chance: 0` em 346 linhas** (todas de Strange Pheromone, o item
  de breeding). Isso não é "nunca cai", é "a fonte não diz" — a ficha troca as
  derivações por uma frase em vez de imprimir zeros. Virou
  [[Zero na tela é afirmação, não valor de conforto]].

## O robô está parqueado, não portado

Pedido do Eduardo: trazer o bot pro piwdex2 pra pensar no substituto depois. Ele vive em
`parked/`, **fora de `src/` e no `exclude` do tsconfig** — não compila, não entra no build.
Religar direto arrastaria Auth.js, Postgres, Mercado Pago e a UI antiga inteira, ou seja,
justamente o design que motivou a reescrita, e ainda travaria a decisão em aberto.

O que não pode se perder está apontado no `parked/README.md`: o protocolo do WebSocket
cravado por engenharia reversa (único caminho pros pokémons individuais — ver
[[Quando a REST não expõe o dado, o WebSocket do mesmo sistema entrega]]), a sondagem
paralela de shard com early-exit, e o porquê do login ser por token e não por senha.

## Interface em português, e a fonte que levou cinco tentativas

O vocabulário do sistema (tipo, raridade, categoria, papel) é traduzido; nome de espécie
e de item ficam em inglês, porque são a chave de busca compartilhada com o jogo — a regra
virou [[Traduza o vocabulário do sistema, não o nome próprio]]. Cada grandeza da tela tem
símbolo próprio. Os de domínio e os 18 de tipo **não** são pixel art: nasceram 8x8 e o
Eduardo reprovou ("muito feios, quero moderno"), então viraram ícone de traço do
lucide, escolhido pela silhueta distinta e pareado com a cor canônica — quem não
distingue matiz separa pela forma. A fronteira entre os dois virou
[[Arte de ícone se julga no tamanho de uso, e o acento é a massa]].

Os ícones das ferramentas na home são **pixel art de 32x32 desenhada por código**
(`scripts/pixel-icons/`): um motor de formas em grid de caractere, porque 32x32 escrito
à mão em JSON de coordenada é erro de índice garantido. Cada um usa a cor da própria
ferramenta sobre o mesmo chassi escuro, com contorno preto e halo neon — a paleta saiu
do `pokedex.png` que o Eduardo desenhou, quantizado. Detalhe que só aparece depois de
montar: a arte tem de ter a **mesma proporção de margem** do arquivo de referência (73%
da altura), senão um ícone colado na borda aparece maior que os outros dentro da mesma
caixa de CSS. Por isso o grid é 32 dentro de uma moldura de 44.

A fonte de rótulo é **Quantico** (peso 700), e chegou lá por eliminação: Press Start 2P
(ilegível), Silkscreen (some no corpo pequeno), Jersey 10 (condensada e fina), Orbitron
(legível, mas larga e fria). O padrão que as três primeiras revelam: **fonte bitmap só
funciona com traço grosso E corpo grande**, e nessa combinação a densidade morre. A
Quantico resolve por outro caminho — é quadrada e tecno com formas de letra normais.
A escada inteira de tamanhos subiu um degrau junto (corpo em 15px, rótulo a partir de
10px): o Eduardo reprovou a versão anterior por ser pequena demais, e legibilidade ganha
de densidade.

## Passada de robustez, mobile e arte (ago/2026)

Auditoria multi-agente sobre as ~19,4k linhas, com refutação adversarial de cada
alegação de defeito — 118 achados, 25 alegações de defeito, **17 confirmadas** e 8
derrubadas. As mais graves eram todas do mesmo tipo: número errado com cara de certo.

- **Degrau no rendimento.** `uptime` trocava por 1 acima de 6 abates por vida; como ele
  multiplica XP/h, ouro/h e KOs/h, 0,08% de sobrevivência virava 2,5x na tela, e em 642
  combinações o alvo vencedor da rota mudava só por isso. Virou `smoothstep`, preservando
  o 1 na ponta pra não recalibrar a escala absoluta →
  [[Limiar em grandeza contínua vira degrau, e o degrau decide a ordem]].
- **Dois motores, dois critérios.** O Meta media só o melhor golpe enquanto a Hunt somava
  o moveset — mesma luta, respostas 3x diferentes, e 292 das 434 espécies mudavam mais de
  20 posições no ranking. Unificado em `poolDps`; `bestMove` sobrou só como o golpe que a
  tela **nomeia**.
- **Régua absoluta sobre escala móvel.** Os eixos normalizavam pelo maior do catálogo e
  os cortes de tier eram fixos: uma única espécie buffada movia 308 outras de tier. As
  referências viraram constantes versionadas — 308 → 0.
- **IV travado, tela ainda afirmando.** A trava resolveu o "99–32"; a projeção, a cor e o
  conselho continuavam saindo do valor travado →
  [[Travar o valor não impede a tela de afirmar a partir dele]].
- **Camada de fonte.** Selo preso em SNAPSHOT depois de um 502 passageiro, snapshot do
  build descartando dado do jogo na segunda falha, sonda cega virando 1,6 MB a cada 10s e
  validação que aceitava catálogo truncado como AO VIVO →
  [[Sonda que falhou não é sinal de que mudou]].
- **404 e erro** não existiam: um throw na derivação devolvia a tela padrão do Next, em
  inglês. Os três arquivos entraram no dialeto do site.

**Mobile, medido com Playwright e não por leitura** (iPhone 12 e Galaxy S9+). Sete telas
empurravam a página para fora da janela — era a causa da queixa "não dá pra clicar": a
página desliza de lado enquanto a pessoa mira. E o mesmo vazamento fazia o overlay
`fixed` do modal centrar na largura errada, abrindo a gaveta de filtros deslocada. Sete
telas → zero; alvos abaixo do piso duro da WCAG: 71 → zero. Detalhe: medir a **caixa** do
elemento não serve — vale `elementFromPoint` →
[[Alvo de toque pergunta pelo apontador, não pela largura da janela]] ·
[[Área de toque cresce por pseudo-elemento, não pela caixa]].

**Arte.** Doze peças novas em `scripts/pixel-icons/arte.py` (404, estado vazio, selo de
frescor, oito categorias de item) e as seis existentes passaram a abrir as seis
ferramentas, não só a home. A primeira leva saiu magra ao lado das antigas e teve de ser
refeita → [[Arte de ícone se julga no tamanho de uso, e o acento é a massa]].

**Movimento.** Grade sem quadro de transição ao filtrar, botão sem resposta ao toque,
barra de aba que teletransporta. E duas animações no eixo errado, que eram também custo:
`width` em laço infinito na tela de carregamento e caixa borrada crescendo em cada card
→ [[Animação de enfeite escolhe a propriedade pelo custo, não pelo efeito]].

## Passada de superfície, herói e voz (ago/2026)

Três queixas do Eduardo na mesma tela: "tá muito transparente o fundo", "esse herói
ainda está ruim, muita frase de IA", "cadê ícones, animações em tudo".

- **O vidro deixava passar a cena inteira.** Painel em 62%, campo em 66%, scrim
  abrindo 46% no topo: o neon do wallpaper chegava a 20% de brilho **dentro** do
  painel, atrás do número digitado. Painel foi a 90%, campo a 92%, scrim a 70%, e a
  arte dentro do painel caiu pra 3%. O vidro continua vidro porque quem faz vidro é
  blur + aresta de luz + sombra, não o alpha —
  [[Sobre arte de fundo, a chrome também tem piso de opacidade]].
- **O topo das seis ferramentas era rótulo.** Arte de 44px e a palavra: abrir a Hunt e
  a Meta davam a mesma imagem com uma palavra trocada. Virou faixa que responde onde
  estou, o que isso faz e qual o tamanho disso (a contagem real do catálogo), com toda
  a decoração saindo de um `--tint` só —
  [[Faixa de topo de ferramenta é chegada, não rótulo]].
- **Identidade de ferramenta virou dado.** Nome, cor, arte, glifo e frase moravam na
  home, na navegação e na chamada de cada página. `lib/ferramentas.ts` abastece os
  três, e a navegação ganhou de brinde o glifo e a cor que só a home tinha.
- **A frase saiu de dois lugares.** Com a faixa carregando a frase da ferramenta, a
  barra do "Como usar" repetia a mesma coisa dois blocos abaixo. Ela passa a anunciar
  o **tamanho** do manual ("7 passos, e 5 ressalvas"), que é o que decide se vale
  abrir — [[Manual de ferramenta é resumo visível com passo a passo sob demanda]]
  atualizada.
- **A voz.** O tique não era o assunto, era o ritmo: travessão emendando uma segunda
  oração, "não é X, é Y", e a explicação que se justifica sozinha. Reescritos os
  quatro manuais inteiros (resumo, passos e ressalvas), as notas de Meta e Hunt e o
  estado vazio. Nenhum dado do jogo mudou, e as constantes continuam importadas.
  Saiu um número a menos: "o botão simula os 342 alvos" estava digitado à mão
  enquanto a faixa de topo já mostra a contagem real —
  [[Texto de interface soa a IA pelo ritmo, não pelo assunto]].
- **Ícone onde ele paga aluguel.** Aba (o `icon` entrou no `TabItem`, não dentro do
  `label`, pra o vão ser o mesmo em toda aba), navegação, e as duas ações que toda
  ferramenta tem no cabeçalho — "preencher exemplo" e "limpar" eram texto puro numa
  fila onde todo vizinho tinha glifo, e liam como link solto.
- **Armadilha do reduced-motion.** O reset zerava a duração e deixava o atraso: com a
  entrada em cascata, meio segundo de tela em branco pra exatamente quem pediu pra não
  ver movimento — [[Reduzir movimento tem que zerar o atraso, não só a duração]].
  Conferido com Playwright em `reducedMotion: "reduce"` e **sem** `waitForTimeout`,
  que é o que escondia o defeito.

## A reescrita assume o deploy do v1 (22/08/2026)

O piwdex2 estava pronto e sem deploy; o [[piwdex]] estava no ar em piwdex.com.br, no
Railway, com o visual que motivou a reescrita. Em vez de subir um serviço novo e migrar
domínio, o **conteúdo do repositório do v1 passou a ser o deste projeto**. O deploy que
já existia publica a versão nova sozinho.

A commit da troca tem **dois pais**, de propósito: o histórico do v1 continua alcançável
e o do piwdex2 vai junto, em vez de virar um despejo inicial sem passado. A árvore é
byte a byte a do piwdex2 (conferido por `rev-parse HEAD^{tree}`).

**Uma cópia por escrito antes de qualquer coisa** — tag `v1.0.0`, branch
`reserva/v1-robo`, e um `git bundle` de 5,9 MB com as 38 branches. Detalhe na nota do
[[piwdex]].

### O que uma troca de árvore ingênua teria quebrado

Nenhum dos três aparece como erro de código, e dois deles falham **em silêncio** — o
deploy não promove e o site velho continua servindo:

1. **`preDeployCommand: node db/setup.mjs`** no `railway.json` do v1. A pasta `db/` não
   existe aqui, o pré-deploy falha, o deploy nunca sobe.
2. **`healthcheckPath: /api/health`.** O piwdex2 não tinha uma rota de API sequer.
   Healthcheck que responde 404 marca o deploy como falho.
3. **`www` → apex.** Sem o redirect o mesmo site responde em dois endereços, e isso é
   conteúdo duplicado na busca — o pior resultado para um site que vive de busca.

Os três foram portados antes da troca. A lição geral virou
[[Herdar um deploy é herdar o contrato dele, não só o domínio]].

### O robô saiu do ar

Sessão por WebSocket, fila de captura e de nivelamento, mercado, watchlist, alertas,
Auth.js, cockpit e o pagamento de VIP. Decisão do Eduardo: "pensaremos em algo depois".

`/vip` e `/bot-app` não viram 404 — caem na home por redirect **temporário** (307).
Essas rotas estão salvas no próprio jogo, no Discord e no favorito de quem usava, e
`permanent` ensinaria navegador e buscador a nunca mais pedi-las. O robô está parqueado,
não enterrado.

## Passada de SEO (23/08/2026)

Auditoria do que estava no ar, com evidência medida: **62,7/100, grau C** (perfil
publisher). O objetivo do Eduardo é aparecer para "poke idle world" e derivados.

### O erro central: o termo-alvo estava no lugar errado

As ~910 fichas traziam "Poke Idle World" no `og:title` e **não** no `<title>`.
OpenGraph pinta card de rede social; quem o buscador casa com a consulta é o
`<title>`. Quem digitava "bulbasaur poke idle world" batia numa página cujo título
não dizia isso em lugar nenhum — na maior superfície do site.

### As ~910 fichas viraram cacheáveis

Eram `force-dynamic`: todo rastreio renderizava do zero um conteúdo que só muda
quando o jogo publica patch. Isso queima orçamento de rastreio, que é o que decide
quantas páginas o buscador se dá ao trabalho de reler. `force-static` +
`revalidate`: de `ƒ` para `○` na tabela de rotas, e **1,9s → 13ms** na visita
cacheada. Saiu junto o piso de carregamento dessas telas — ele lê `headers()`, que
sozinho já impede o cache.

### Dezoito hubs de tipo

`/dex/tipo/fogo` e irmãos. A dex só filtrava por parâmetro de busca, que o
buscador rastreia com má vontade, e as fichas estavam alcançáveis só pela lista
filtrada. O selo de tipo da ficha virou link pro hub, fechando a malha.

Cada hub afirma o que só vale pra aquele tipo (contra quem bate, de quem apanha,
faixa de nível), tudo saindo de `typing.ts` e do catálogo — mesma regra do
`prosa.ts`, sem molde com o nome trocado.

**Soft 404 que a primeira versão criou:** com `dynamicParams` no padrão,
`/dex/tipo/banana` respondia **200** com a tela de "não encontrado". Soft 404 é
pior que 404 — o buscador indexa página vazia. `dynamicParams = false` fecha o
conjunto nos dezoito.

### O achado que eu registrei errado

Marquei "/dex serve 1,14 MB" como falha ALTA. No fio são **68 KB** (Brotli, 94%).
Não havia o que consertar. Virou
[[Peso de página se mede no fio, não na saída do render]].

### O que ficou com o Eduardo

Search Console verificado por DNS (propriedade de Domínio, na Cloudflare) e
sitemap enviado — 936 URLs. Conferido que o Googlebot recebe 200 em tudo, sem
desafio da Cloudflare, e que a home passa em "É possível indexar a página".

Daqui o gargalo não é mais código: é a fila de rastreio do Google, que para site
novo leva de dias a semanas.

## O farm de ouro sai da rota (23/08/2026)

O Eduardo abriu a rota em modo "ganhar ouro" e viu a tela recomendar a caçada que paga
**menos** por hora. Reproduzido no motor contra o catálogo real (Golem nv352 → 500, IV
médio, VIP, captura com Ultra Ball): "subir rápido" dava Furious Scyther em 99h a 459k/h,
"ganhar ouro" dava Scyther em 176h a 447k/h. Os 78,7M de "ouro no caminho" que coroavam o
segundo eram 447k vezes 176 horas — o modo vencia por demorar. Virou
[[Total acumulado premia a lentidão quando o tempo é livre]].

O que fazia a promessa "cada faixa é a de maior ouro/h" quebrar era a histerese: no 352 o
Scyther paga mais mesmo (410k contra 396k), no 500 o Furious vira por 2,8% e a margem de
troca é 8%, então a faixa nunca troca. Como o card mostrava a estimativa do FIM da faixa,
a tela imprimia o número que desmentia o próprio botão.

Três mudanças, decididas com o Eduardo:

- **Aba "Farmar ouro"**, com alvo em ouro e não em nível. Você diz quanto quer juntar e a
  tela responde quantas horas no melhor spot; abre a origem do ouro (loot × captura
  líquida), o que rende parado em 1h/8h/24h e os nove seguintes com a fração do melhor.
  Spot letal fica fora: farm que precisa de você olhando rende zero.
- **A rota volta a perseguir nível e mais nada.** `RouteMode` saiu do motor junto com
  `pickHunt`, que era export sem chamador e só existia pra passar o modo.
- **O tempo da rota passou a ser integrado nível a nível.** Ele saía de "XP da faixa
  inteira ÷ XP/h do fim da faixa", e o ritmo sobe com o lutador: numa faixa 352→500 o
  alvo rende 11,7M de XP/h no começo e 13,6M no fim. Junto veio um nível cobrado a mais
  (`xpTotal(fim+1)`). A mesma rota passou de 99h28 pra 104h43, e a soma bate exatamente
  com a curva fechada em 7.628 faixas varridas. Virou
  [[Taxa que muda ao longo do trecho se integra, não se amostra na ponta]].

**O Tipo do Dia estava pela metade.** O jogo publica "+20% de XP e +20% de loot", e só a
metade do loot estava implementada — `xpH` saía de `e.xp * kosH * (vip ? 1.5 : 1)`, sem o
dia em lugar nenhum. Numa rota 352→500 com dia de Inseto isso escondia 17 horas. As duas
moedas têm regra diferente e agora o código diz isso: no XP o bônus entra inteiro, no loot
o teto de chance engole a maior parte, e o mesmo dia paga +20% de XP e +6,6% de ouro no
Scyther. Registrado em [[Bônus multiplicativo só rende onde há folga até o teto]] e
[[Bônus condicional se avalia contra quem não o recebe]].

Sobrou uma primitiva: `NumberField` ganhou `grouped`, que separa o milhar no próprio campo
(meta de ouro tem oito dígitos, e "10000000" não se lê, se conta com o dedo).

## Próximos passos

1. As seis ferramentas estão no ar; o "em breve" saiu da navegação.
2. Decidir o que substitui o robô. Até lá, `parked/` fica como está.
3. O `pokedex.png` tem 584 KB (arte gerada, com ruído) contra ~10 KB dos ícones
   desenhados por código. Quantizar em 64 cores derruba pra 57 KB sem diferença visível
   no tamanho em que ele aparece — decisão do Eduardo, é arte dele.
4. Sobraram 101 alvos de toque entre 24 e 44px. Passam na norma, ficam abaixo do conforto
   da Apple — subir todos mexe na densidade escolhida, então é decisão de produto.
5. O piso de 1,2s do `pacing.ts` (loading sempre visível, pedido do Eduardo) é pago em
   TODA renderização, e as rotas são dinâmicas: um rastreamento completo do sitemap custa
   ~18 min de CPU parada em `setTimeout`. O interruptor está a vista (`MINIMO_MS = 0`).
6. A auditoria deixou 16 achados de design de tela, 22 de primitiva e 17 de texto de
   interface ainda não aplicados.
4. As páginas hoje são dinâmicas (`ƒ`) porque a fonte roda com `cache: "no-store"` — o
   frescor é gerido no `source.ts`. Se virar gargalo de CDN, é aqui que se mexe.
5. O script `lint` do `package.json` ainda chama `next lint`, que saiu no Next 16 e hoje
   quebra. Trocar por ESLint direto quando incomodar.

## Conexões
- Substitui: [[piwdex]]
- Usa: [[Design]] · [[Infra]] · [[Frontend]]
- Aprendizados: [[Traduza o vocabulário do sistema, não o nome próprio]] ·
  [[A tela não afirma mais precisão do que a fonte tem]] ·
  [[Estimativa que inverte valor arredondado é faixa, não ponto]] ·
  [[Quantos filtros existem é decisão de layout, não de produto]] ·
  [[A régua sai da distribuição, não dos extremos]] ·
  [[A régua de um medidor é percentil, não máximo]] ·
  [[Faixa de cauda longa entra por número, não por slider]] ·
  [[Zero na tela é afirmação, não valor de conforto]] ·
  [[Chip que serve a duas grandezas declara qual delas mostra]] ·
  [[Conteúdo do servidor não pode nascer invisível esperando o cliente]] ·
  [[Sticky gruda no container que rola, não na janela]] ·
  [[Tela que abre vazia tem que ensinar, tela que abre cheia não]] ·
  [[Nota carrega só o que a pessoa não sabe]] ·
  [[Manual de ferramenta é resumo visível com passo a passo sob demanda]] ·
  [[Custo de processo aleatório se orça pela cauda, não pela média]] ·
  [[Distribuição exata sai de programação dinâmica, não de Monte Carlo]] ·
  [[Peça o que a fonte mostra, não o que você precisa]] ·
  [[Medidor de razão nomeia a grandeza e mostra os operandos]] ·
  [[Fila de campos alinha por altura fixa de controle, não por items-end]] ·
  [[Altura 100% em item de grid de linha automática volta ao tamanho intrínseco]] ·
  [[Consulta pesada executa por botão, não por mudança de filtro]] ·
  [[Token de cor que não existe vira cor herdada, sem erro]] ·
  [[Pedido de apoio entra depois do valor, e nunca ao lado de si mesmo]] ·
  [[Slot de anúncio no App Router precisa de casca estável e filho keyado]] ·
  [[Anúncio em feed não pode vestir a roupa do conteúdo]] ·
  [[Limiar em grandeza contínua vira degrau, e o degrau decide a ordem]] ·
  [[Travar o valor não impede a tela de afirmar a partir dele]] ·
  [[Alvo de toque pergunta pelo apontador, não pela largura da janela]] ·
  [[Área de toque cresce por pseudo-elemento, não pela caixa]] ·
  [[Animação de enfeite escolhe a propriedade pelo custo, não pelo efeito]] ·
  [[Sonda que falhou não é sinal de que mudou]] ·
  [[Arte de ícone se julga no tamanho de uso, e o acento é a massa]] ·
  [[Sobre arte de fundo, a chrome também tem piso de opacidade]] ·
  [[Faixa de topo de ferramenta é chegada, não rótulo]] ·
  [[Reduzir movimento tem que zerar o atraso, não só a duração]] ·
  [[Texto de interface soa a IA pelo ritmo, não pelo assunto]] ·
  [[Herdar um deploy é herdar o contrato dele, não só o domínio]] ·
  [[Total acumulado premia a lentidão quando o tempo é livre]] ·
  [[Taxa que muda ao longo do trecho se integra, não se amostra na ponta]] ·
  [[Bônus multiplicativo só rende onde há folga até o teto]] ·
  [[Bônus condicional se avalia contra quem não o recebe]]
- Referência: [[Poke Idle World - endpoints publicos de dados]] · [[Poke Idle World - regras de breeding]]
- Mapa: [[Projetos]]
