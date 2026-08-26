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

- **Stadium** (24/08/2026): o time de seis contra um boss, com o combate inteiro
  simulado. É a sétima ferramenta, e a mecânica que a separa do Duelo do Meta é o **HP do
  boss atravessando a troca de lutador**: o segundo pokémon não entra contra um boss
  inteiro, entra contra o que sobrou — por isso um time de seis medianos derruba o que
  nenhum deles derruba sozinho, e nenhuma soma de duelos consegue dizer isso. A recarga
  do boss atravessa junto (ele estava lutando quando o seu caiu); quem entra chega com a
  própria recarga zerada. A medida que manda na tela é a **fatia**: quanto do boss cada
  um leva embora antes de cair, que é o único número que se soma de cabeça. Alvo, time e
  reforço na URL; times salvos no `localStorage`.

  O **catálogo de bosses** entrou na ingestão (`/game/bossCatalog.json`): 87 bosses com
  nome, categoria, nível oficial (300 a 625) e drops. O jogo não publica tipo nem stat de
  boss, então cada um é resolvido pra **espécie de que ele é feito** — fecha em 51 dos 87,
  e os 36 restantes (a categoria Terror inteira, os humanos da Rocket) continuam na lista
  dizendo que não dá pra simular.

- **Eevee** (25/08/2026): a oitava ferramenta, e a única de uma espécie só — porque o
  Eevee é o único caso de decisão irreversível com cinco saídas do mesmo preço. Mostra a
  estrela dos cinco destinos, o que a troca custa, **onde farmar as dez pedras no seu
  nível** e o que cada eeveelution vale em combate, nas duas colunas lado a lado. Detalhe
  na seção de 25/08 abaixo; a mecânica em
  [[Poke Idle World - evolucao e a troca do Eevee]].

  A primeira versão pedia espécie, nível e quality e **supunha IV médio** — e o Eduardo
  pegou na primeira olhada: "ele nem está pedindo os status do meu pokémon". Estava certo,
  e o defeito era de natureza, não de precisão: IV é justamente o número que o jogo
  esconde, então o combate respondia sobre um Charizard médio em vez do dele. Virou caso
  em [[Peça o que a fonte mostra, não o que você precisa]]. Agora o time entra com os
  **seis stats de verdade**; só o alvo é projetado, porque de boss o jogo não publica
  stat nenhum, e a tela diz isso.

- **A penalidade de grupo** (25/08/2026): o Eduardo testou no jogo e o Golem dele tomou IK
  enquanto a tela dizia "fatia 100%, aguenta infinito". Três erros ao mesmo tempo, e a
  ficha do boss NO JOGO entregou os três — ver
  [[A interface do sistema explica o que a API dele esconde]].

  A penalidade de grupo é `dano recebido × 3^(6 − força)`, com a força somando, nos seis
  lugares, o quanto cada um está no nível do boss. Um sozinho no nível toma **243x**. Era
  isso que matava. O elemento do boss é **Neutro**, e não o tipo da espécie de que ele é
  feito — o de-para prometia 2,5x numa luta que é 1x. E a vida do Ancient Aero é **72 mil**
  contra os 4,6 mil que a projeção sobre o Aerodactyl dava, então os seis stats do alvo
  viraram campos, guardados por boss.

  O piwtools resolve o mesmo problema cravando 130 em todos os seis stats de todo boss:
  número inventado com cara de dado, e com 130 de vida onde há 72 mil qualquer time vence
  em dois segundos.

- **Bolsa e decks** (25/08/2026): a estante de pokémon salvos saiu de dentro do Breeding e
  virou a **bolsa do site inteiro** — um pokémon cadastrado é da pessoa, não da
  ferramenta. Cada entrada é uma **carta** (apelido, nível, quality, os seis stats, e o IV
  lido de volta), e o **deck** aponta pras cartas em vez de copiar os números: corrigir o
  nível numa carta arruma todos os decks em que ela está. Sem login, os dois moram no
  `localStorage`, e a estante antiga é migrada **na leitura** —
  [[Sem servidor, a migração de dado local acontece na leitura]].

Motores puros portados e vivos em `src/lib`: `stats`, `xp`, `typing`, `rarity`,
`catch-law`, `balls`, `combat`, `breeding`, `meta`, `boost`, `sim`, `stadium`. Com eles no
lugar, Calculadora / Hunt / Breeding / Meta / Stadium viram trabalho de interface, não de
pesquisa.

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

## O robô voltou, em bot.piwdex.com.br (23/08/2026)

Ficou parqueado em `parked/` de 20 a 23/08. O que o desparqueou não foi só a decisão de
tê-lo de volta — foi decidir ONDE ele roda.

**Ele tem serviço próprio, e essa é a decisão central.** O robô segura um WebSocket por
usuário, e WebSocket morre inteiro a cada deploy; a dex é publicada várias vezes por dia
por causa de SEO. Enquanto dividiam um serviço, cada push derrubava a caçada de todo
mundo ([[Processo que guarda conexão viva não tolera deploy frequente, e o log não denuncia]]).
Dois serviços a partir da **mesma imagem**, papel decidido por `PIW_ROLE`, roteamento por
host no `proxy.ts` — [[Um processo serve dois hosts quando o papel vem do ambiente]].
Sem duplicar as 22 primitivas nem os motores puros, que os dois compartilham.

`PIW_ROLE` ausente em produção vale `site`: esquecer a variável deixa a dex intacta e o
robô apagado.

### O que entrou

Escopo escolhido pelo Eduardo: **núcleo** (conta, vínculo, caçada ao vivo), **pago desde
o começo**. Seis camadas, uma commit cada:

1. **Chassi** — `PIW_ROLE`, `proxy.ts`, grupos de rota `(site)`/`(robo)`. Nenhuma URL da
   dex mudou (grupo de rota não aparece em endereço).
2. **Banco** — quatro tabelas numa migration consolidada, não a replay das 20 do v1.
3. **Conta** — Auth.js v5 sobre SQL puro, com o freio de login e o `bcrypt.compare` em
   usuário inexistente que a revisão do v1 tinha deixado em aberto.
4. **Vínculo** — token do jogo cifrado (AES-256-GCM), recusa classificada (401/403/429
   pedem coisas opostas), shard por sondagem paralela.
5. **Assinatura** — Mercado Pago por PIX, avulso, com o `x-signature` do webhook que
   também estava na lista de pendências do v1.
6. **Motor e cockpit** — sessão por usuário, desmaio do líder, reconexão com backoff, e
   um stream SSE no lugar de cinco pollings.

O `parked/` continua onde está: o que sobrou lá (mercado, sniper, watchlist, alertas,
venda automática, fila de nivelamento, admin) é o que NÃO entrou no núcleo, e o
`ws-protocol.md` segue sendo a fonte do protocolo.

### O que não foi verificado

O laço de caçada de verdade — `enter-hunt`, analyzer acumulando, kill, captura, desmaio —
**não foi testado contra o jogo**. Testar exige um token real, e conectar TOMA a sessão de
jogo: chutaria a aba do Eduardo. Tudo o que dá pra provar sem isso foi provado, inclusive
uma ida e volta real ao jogo com JWT falso (o jogo respondeu 401 "Sessão inválida ou
expirada" e a classificação de recusa funcionou de ponta a ponta).

### O v1 estava desatualizado num ponto

A nota do [[piwdex]] listava "sessão por usuário (`Map<userId, GameSession>`)" como
pendência antes de vender pra 2+ pessoas. Ela já tinha sido feita — o `robot-boot.ts` do
v1 já religava todas. Isso encurtou o caminho pro modelo pago.

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

## A sessão medida corrige o motor (23/08/2026)

O Eduardo caçou 738 Furious Scyther com Ultra Ball e trouxe o painel do Hunt Analyzer.
Comparado por termo contra a previsão (e não pelo total), o diagnóstico saiu limpo.

**O que já estava certo:** abates/h 580 previstos contra 602 medidos, e o loot batendo
DROP A DROP — Straw 2.223 contra 2.214 esperados, Scythe 416 contra 408, Pot of Moss Bug
718 contra 738, Cocoon e Feather Stone 11 cada contra 9. Os 7% de diferença no ouro por
abate são quatro pedras a mais num drop de 1%. De quebra a sessão confirmou o teto de
chance: Straw sai em 95% base, o dia empurra pra 114% e o jogo entregou 100%.

**Erro 1: os bônus multiplicavam onde o jogo soma.** O `boost.ts` já documentava o
empilhamento somado do jogo para o loot, mas o XP fazia `e.xp * (vip ? 1.5 : 1)` e o Tipo
do Dia multiplicava por fora — 1,8 onde o jogo dá 1,7. E não havia campo nenhum para o que
a ferramenta não conhece (evento de servidor, boost de loja, streak): com um evento de XP
ligado, o XP/h saía a 0,66x do real sem nada na tela explicando. O cenário ganhou "XP
extra (%)" e "Loot extra (%)", o empilhamento virou soma, e o XP por abate previsto passou
de 0,46x para 1,03x do medido. VIP passou a entrar pela camada de economia junto do dia e
do evento: `estimateHunt` perdeu o parâmetro `vip` e o motor de combate deixou de conhecer
bônus de qualquer espécie.

**Erro 2: a captura, e ela sozinha estourava a conta.** A lei previa 1 captura a cada 129
abates; a sessão fez 1 em 738. Como a bola é cobrada em todo abate, o termo trocou de
sinal: +194k/h prometidos contra −30k/h reais. O ouro/h total previsto era 434k contra
236k medidos, e o buraco inteiro era captura. A captura saiu do `perKill`: o ouro que
ordena as três vistas voltou a ser só loot, e ela virou coluna marcada como estimativa com
o ponto de equilíbrio ao lado (1 em 462 para um alvo de 60.000 com bola de 130), que é
aritmética e se confere numa sessão. Virou
[[Estimativa fraca informa, número verificado ordena]].

O efeito visível: o topo do farm de ouro deixou de ser o Scyther, que só liderava pela
captura inflada, e passou a ser o Furious Scyther — onde o Eduardo já estava caçando por
conta própria.

**Fio solto:** o catálogo traz um campo `captureBase` em 120 das 482 espécies (Furious
Scyther 123, Psy Jynx 124, Brave Charizard 6), com 119 valores distintos e correlação
−0,06 com o valor de venda. O cabeçalho de `catch-law.ts` afirma que não há campo de
captura no catálogo. Independente do preço e específico por espécie é a cara de um
parâmetro de captura de verdade, e vale sondar contra `/api/game/used-balls`.

## Itens, meta e fundo entram na linguagem da dex (24/08/2026)

A dex e a ficha de espécie já tinham virado "console macio" — raio, elevação,
card com medalhão na costura, epíteto acima do nome. Itens e meta ficaram pra
trás, e o Eduardo apontou o sintoma exato: *"o raro nem é o mesmo de pokédex"*.

**A raridade dos itens era um interruptor, e ele mentia.** O jogo publica
`rare: true/false`, ligado em 206 dos 428 itens; cruzado com a dificuldade real,
31 dos 85 itens mais fáceis do catálogo o carregavam. A escada agora é derivada
de quantos abates custa uma unidade na melhor fonte farmável, em décadas, e reusa
o `Rarity` da dex — mesmos seis nomes, mesmas seis cores, mesma chave `?r=` na
URL. Deu 85/90/47/38/10/18. Os 140 itens sem chance publicada ficam **sem faixa** e
a ficha diz por quê. O selo do jogo volta só na ficha, dizendo de quem é a
afirmação. Virou [[Sinal booleano da fonte não ocupa o lugar de uma escala]] e
[[A mesma grandeza usa a mesma escada nas duas telas]].

**A ficha do item virou chegada de campeão**, no mesmo tratamento da ficha de
espécie — era o cabeçalho em linha que a dex já tinha abandonado. E o ouro por
abate saiu da tabela pro rodapé do card: era a resposta da tela, e só aparecia
pra quem trocava de modo de visualização.

**A tier list do meta era a última tela no dialeto antigo** — faixas de canto reto
com `border border-line` contra o `panel` de vidro do resto, e cada pokémon num
retângulo cinza. Além da forma, faltava informação: **o tipo não aparecia em
lugar nenhum** numa tela que existe pra decidir quem usar. Entrou como faixa de
cor de 3px na costura, e não como o medalhão do card grande — que reprovou por
escala e virou seção nova em
[[Trocar arte por ícone de linha exige recalibrar tamanho, não só trocar o componente]].

**O wallpaper trocou por uma ilustração clara** (média 77 na fonte contra 19 da
cidade anterior). Ela pediu desfoque assado no arquivo, o que derrubou a regra
antiga de "quem borra é o vidro dos painéis", e pediu o scrim mais **aberto** das
três calibrações — 18% no topo contra 40%. Em
[[Trocar a arte de fundo é refazer a calibração, e a régua não é a média]].

**O destaque da home voltou pra dentro de um card.** A passada anterior tinha
tirado o painel em favor de "luz em vez de caixa"; sem superfície ele não lia como
peça, ficava pousado sobre o wallpaper. A forma não foi inventada: é o `ArtCard`,
que já morava em `ui/` e só aparecia na página de estilo.

De quebra: o `image-rendering: pixelated` do fundo saiu (sobreviveu à remoção da
pixelização e serrilhava a arte suavizada), o `MultiSelect` ganhou slot de dica —
cor sozinha não ensina que rosa é caro — e os medidores do perfil de meta viraram
pílula.

### A home aparecia sem texto, e a calculadora ficou pra trás (24/08/2026)

O Eduardo mandou print da home com as artes das ferramentas boiando sobre o
wallpaper e **nenhuma palavra**. Duas causas somadas, nenhuma nova — o wallpaper
claro só tirou a maquiagem de um defeito que existia em janela baixa:

1. o gatilho da revelação segurava conteúdo já visível (`threshold: 0.15` com
   `rootMargin` de -12% embaixo, que ENCOLHE a raiz e dispara mais tarde — o
   comentário afirmava o contrário do que o valor fazia);
2. texto e arte eram duas revelações independentes de uma composição só.

Virou [[Nada que está na tela pode estar invisível esperando o scroll]]. A
densidade abriu a porta: empilhada a faixa mede ~810px contra ~470 de altura útil,
então duas colunas passaram a valer de `md` e não de `lg` — os 256px entre um e
outro eram exatamente a janela em que a home se desmontava.

**A calculadora entrou na linguagem das fichas.** Ela era a única tela que ainda
mostrava o sprite pixel de 96px (o `animatedSrc` do gif gen5 entra por cima do
render oficial), e o cabeçalho dela falava dois idiomas de raridade ao mesmo
tempo: halo na raridade da ESPÉCIE, chip na faixa do INDIVÍDUO. Tela de indivíduo
— quem manda é a quality digitada.

**E a marca de escala saiu do topo das seis ferramentas.** O "482 espécies" na
ponta direita da faixa respondia pergunta que ninguém faz na chegada, e a lista já
diz "428 itens" na barra dela três centímetros abaixo.

**O motor de IV foi auditado e a fórmula está certa** (a conferência declarada no
cabeçalho bate; ida e volta passou em 14.406 de 14.406). O defeito está em como o
inverso é RELATADO, e virou
[[Invertida a fórmula, o arredondamento é meia-aberto e o meio da faixa não é a resposta]]:
em 73% das leituras o stat fixa um único IV possível, e em 1% dessas a tela mostra
o inteiro errado — sempre uma unidade acima, concentrado em nível 50 com quality
1,0. **Não foi corrigido**: mexer nos números exibidos de uma fórmula verificada
contra o jogo é decisão do Eduardo. A saída é enumerar os 33 IVs em vez de
inverter.

### A espera, os ícones e o cartão (24/08/2026)

**A tela de espera vestia a cor errada.** Halo e barra eram `--color-t-dex`
cravado, então esperar a Calculadora abria uma cena vermelha — a cor da Pokédex.
A cor sai da rota agora (`usePathname` + `ferramentaDoCaminho`), e não de um
`tint` por chamada: são nove arquivos de uma linha e o décimo nasceria sem.

**O cartão de compartilhamento ficou órfão da virada pro console macio** — grade
de 20px, moldura dupla em fio neon, canto reto e os medidores em BLOCOS, que a
interface já tinha abandonado. Redesenhado nos tokens do grafite morno, com arte
oficial suavizada e medidor em linha contínua. Virou
[[Peça desenhada fora do DOM é uma segunda implementação do tema, e ela envelhece calada]].

### O motor de IV corrigido, e os ícones que não melhoraram (24/08/2026)

**O IV saiu da inversão e virou enumeração.** O defeito relatado na sessão
anterior foi confirmado e corrigido: em 73% das leituras o stat fixa um único IV
possível, e em 1% dessas a tela mostrava o inteiro errado, sempre uma unidade
acima — e o breeding lia `inteiros` como o IV do pai, então o erro entrava na
previsão do filho. Enumerar os 33 candidatos por stat dá resposta exata; depois da
troca, 0 erros nas mesmas 57 mil combinações. A inversão fica como reserva pra
quando a quality digitada estiver arredondada e nenhum inteiro fechar.

**Os ícones não melhoraram, e isso virou nota.** A tentativa foi dobrar a grade de
32×32 pra 48×48 com mais tons; piorou em três passadas seguidas. O mesmo desenho
em SVG ganhou dos dois logo de cara. Virou
[[Mais resolução não compra qualidade em ícone; trocar de meio compra]]. Os ícones
do site continuam os de 32×32 — a troca pra vetor está oferecida e não feita.

**Skill nova de ilustração SVG** em `~/.claude/skills/svg-illustration/`, com o
script de preview que obriga a olhar o render antes de entregar. Ela não depende
de API nenhuma: o desenho é autorado, não gerado.

### Ícones em vetor, espera sem gif e lendários fora do meta (24/08/2026)

**Os seis ícones viraram SVG**, autorados com a skill nova. Duas armadilhas que
falham caladas e custaram a achar: comentário XML não pode conter `--` (e nome de
token CSS começa com `--`, então documentar a paleta dentro do comentário deixava
o arquivo malformado e o navegador recusava a imagem inteira), e SVG só com
`viewBox` tem `naturalWidth` 0, o que fazia a verificação de carregamento do
`Sprite` dar a arte como falha.

**O gif de pokémon saiu da tela de espera** — era a última peça em pixel do site.
No lugar entra o ícone da própria ferramenta, então a espera passa a mostrar pra
onde se está indo e vira o primeiro quadro da tela que chega.

**Lendário saiu da tier list.** Os desenvolvedores disseram que não vão ser
jogáveis, e os 11 ocupavam o topo do S. Entrou como chave com padrão, não como
exclusão cravada — a mesma fonte disse "tudo pode mudar". Virou
[[Regra que veio de fora do sistema entra como chave declarada, não cravada no código]].
A home segue de graça, porque chama o mesmo motor: o destaque deixou de ser
Zapdos e passou a ser Gengar.

### Dois defeitos que só existiam pra quem já tinha visitado (24/08/2026)

**Os ícones sumiam.** Publicados com o erro do `--`, corrigidos em minutos — mas
`public/` não leva hash de build e `/images/` sai com cache de um dia mais 30 de
`stale-while-revalidate`, então quem abriu na janela errada ficou com o arquivo
inválido preso. No computador de quem consertou já estava certo. Virou
[[Arte servida sem hash de build precisa de versão na URL]]; a URL agora sai de
`arteUrl()`, com a versão dentro.

**O destaque da home mostrava o segundo colocado.** Havia um `pokeId < 1e4` que
pretendia pular variante de skin, mas sete espécies acima de 10000 têm
`captureBase` nulo e looktype próprio (os Megas, o Castform de Fogo) — são linha
própria. O Mega Alakazam lidera a tier list e a home mostrava Gengar. Quem separa
skin de espécie é o `captureBase`, e o `playableSet` já aplicava isso: a home
tinha uma segunda definição de "quem conta".

### O destaque da home vira "em alta" (24/08/2026)

O card mostrava o número 1 da tier list, e o Eduardo apontou o defeito: o mais
forte do jogo não muda, então a home nunca mudava. O desenho é dele — mostrar o
pokémon mais pesquisado, num reinado de três dias, contando uso nas ferramentas e
na ficha da dex, com o "preencher exemplo" **fora** (o pokémon dali não foi
escolhido por ninguém, e como é sempre o mesmo venceria toda eleição pra sempre).

Duas notas saíram daqui:
[[Contador de popularidade conta votante, não evento]] e
[[Rotação por período se apura na leitura, e dispensa agendador]].

**Pendência de infra:** o serviço `piwdex-app` não tem Postgres — só o
`piwdex-bot` tem. Enquanto o `DATABASE_URL` não for anexado no painel do Railway,
a home cai na semente (o topo da tier list, o comportamento antigo) e os
registros viram no-op. Verificado com a variável vazia: home 200, ping 204, seis
telas 200, zero erro. Falta também rodar `node db/migrate.mjs` no pré-deploy da
dex, que hoje não tem nenhum.

A página de privacidade dizia "Nada" sobre o que o site guarda; deixou de ser
verdade nesta mudança e foi reescrita no mesmo commit.

### A arte do site inteiro vira vetor (24/08/2026)

Migracao fechada em quatro levas, e o que a guiou foi uma fronteira que so
apareceu no meio do caminho: **glifo e ilustracao sao registros diferentes, e a
fronteira e o tamanho de uso.**

- **29 glifos de dominio** (stat, ouro, gema, TM, drop, nivel) e **18 de tipo**
  saem do lucide e passam a ser desenhados, CHEIOS. A 12-14px, onde 16 dos 21
  usos vivem, contorno de 2px perde metade da forma pro antisserrilhado e vira
  cinza. O chrome (chevron, busca, fechar) fica na biblioteca de proposito.
- **6 brasoes de raridade**, com a escada na FORMA e nao so na cor.
- **10 ilustracoes**: os 8 de categoria de item, a bola quebrada do 404 e o funil
  do vazio. Slot de 56 a 128px, entao levam sombra de contato e rampa de tres
  tons, como os icones das ferramentas.

O piso dos simbolos de tipo subiu pra 16 depois que o Eduardo apontou que estavam
"tortoes" no miudo — piso resolve melhor que corrigir 21 chamadas, porque o
proximo ponto de uso ja nasce certo.

Os 18 tipos foram desenhados à mão primeiro e depois TROCADOS pelos símbolos
oficiais (partywhale, MIT) — não por qualidade de traço, mas por reconhecimento:
tipo é cânone do jogo e o jogador já tem os dezoito na cabeça. Virou
[[Onde existe cânone visual, use o cânone; desenhe só onde a taxonomia é sua]].

`public/images/icons` agora tem 16 arquivos, todos SVG, servidos pelo `iconeUrl`
com versao. Os tipos e as raridades NAO sao arquivo: sao caminho inline nos
componentes, porque precisam de `currentColor` pra herdar a cor do tipo e da
faixa — arquivo em `public/` nao se tinge por CSS.

### A Hunt entra na linguagem, e a barra de rolagem some (24/08/2026)

Duas coisas na mesma passada, e a segunda saiu de uma queixa de um fiapo.

**O fiapo de barra horizontal na home.** Toda faixa de ferramenta sangra de borda
a borda com `width: 100vw`, e o site reserva a canaleta da barra sempre
(`scrollbar-gutter: stable`). Medido em 1920/1440/1280/768/390, o estouro era
exatamente 5px pra cada lado, sempre nas mesmas tres divs. O corte foi
`overflow-x: clip` na raiz — no ancestral mais proximo ele mataria o proprio
sangramento. Virou
[[Faixa que sangra estoura pela barra de rolagem, e o corte é na raiz]].

**A Hunt era a ultima tela no registro velho.** A faixa do lutador tinha sprite de
52px numa linha de 60: dizia o necessario, no vocabulario errado. Passou a ser a
mesma ficha da calculadora — halo pela quality, arte de 132, epiteto na cor da
faixa, discos de tipo e tres manchetes. O que muda e o CONTEUDO das manchetes,
porque a pergunta e outra: la e quanto o pokemon vale, aqui e melhor XP/h, melhor
ouro/h e quantos dos 342 alvos ele aguenta.

E as manchetes so puderam existir porque a conta subiu de nivel: `economyOf` e
`rankHunts` rodavam identicos dentro de DUAS abas, e a rota fazia o `withEconomy`
por conta propria. Foram pro `hunt-tool`, que calcula uma vez e distribui o array
pronto. O ganho de custo e o segundo motivo; o primeiro e que o resumo do topo sai
do mesmo array que as abas listam, em vez de virar a segunda definicao de "qual e
a melhor hunt" — o mesmo erro que a home ja tinha cometido contra a tier list.
Virou [[Número de resumo sai do mesmo cálculo que a tela detalha]].

A rota tambem saiu do aperto: o TEMPO da faixa lidera o degrau (era legenda de
11px), o tipo do alvo virou disco de 26px no lugar da pastilha com a palavra
escrita, e os tres numeros por hora viraram manchete com rotulo pixel.

As tres abas trocaram o lucide por glifo. A da rota virou **bandeira** depois de a
folha de contato reprovar as trilhas literais: escada colide com o `IconLevel` que
fica a centimetros dela, fita com dois marcos vira osso, trilha pontilhada com
mastro vira colcheia. Virou
[[Glifo miúdo é lido como o símbolo mais próximo que a pessoa já conhece]].

### O brasao vale no site inteiro, e o breed fecha a linguagem (24/08/2026)

**O brasao de raridade so aparecia nas seis paginas de `/dex/raridade`.** No card
de item era pior que ausencia: o medalhao desenhava a CATEGORIA (um cubo) com o
anel na cor da faixa, enquanto a palavra logo abaixo dizia "Raro" — glifo e
palavra a 20px um do outro falando de coisas diferentes. Agora o medalhao desenha
a faixa; sem faixa, desenha a origem, que e o que o epiteto escreve no lugar dela.
O brasao entrou tambem no card e na linha de especie, na linha de item, nos chips
de filtro ativo (onde uma gema generica servia os seis degraus), nos dois menus de
filtro, nas duas fichas, na calculadora, na hunt, na ficha do robo e no seletor de
qualidade minima da loja.

Pra isso a escada foi de seis degraus pra **nove**: a raridade da especie tem
seis, mas a quality do individuo vai de `WEAK` a `DIVINE`. Ficar nos seis
obrigaria a calculadora, a hunt e o robo a nao ter brasao nenhum — ou a emprestar
o de `MYTHIC` pra `ANCIENT`, que e afirmar que os dois degraus sao o mesmo. Virou
a condicao 2 registrada em
[[Chip que serve a duas grandezas declara qual delas mostra]].

**O breed era a ultima tela no dialeto antigo.** O bloco do filho ja era um
perfil, mas com `scanline`, nome em fonte de texto, pastilha de tipo com a palavra
escrita e o halo saindo da RARIDADE DA ESPECIE — o mesmo erro que a calculadora ja
tinha corrigido: a tela e de um individuo, entao quem manda e a faixa da quality
dele. Charizard e Raro; o filho de 1.830 e Lendario.

E dois defeitos de largura, os dois so no telefone de 390 e os dois SILENCIOSOS
depois que a raiz passou a aparar o eixo X: o cabecalho do slot nao podia quebrar
(340px de minimo, e slot e item de grid, que nasce com `min-width: auto`) e a
grade de IV abria em tres colunas. Sem barra, sem erro — a caixa simplesmente saia
cortada. Virou a secao nova de
[[Faixa que sangra estoura pela barra de rolagem, e o corte é na raiz]].

## O Eevee vira a oitava ferramenta (25/08/2026)

O Eduardo pediu "a parte do Eevee". A resposta que o piwdex tinha era uma mentira que
ninguém tinha percebido: **`creatures.json` afirma `Eevee: { evolvesToId: 134,
evolveLevel: 80 }`** — um caminho só, por nível, pro Vaporeon. Campo preenchido, bem
tipado, plausível, e falso nos dois pontos.

O que acontece de verdade está em [[Poke Idle World - evolucao e a troca do Eevee]]: o
Eevee **não evolui, é trocado** com o NPC Marlon por um de cinco destinos, e o custo é
igual nos cinco — $65.000, dez pedras, um Eevee no time. A tabela não existe em fonte
pública nenhuma; ela veio de um print da tela da loja, que o Eduardo mandou. Antes do
print eu tinha chutado Moon/Sun Stone pra Umbreon/Espeon, com dois argumentos bons e
errados. São Darkness e Enigma.

**O achado que definiu a ferramenta foi o preço igual.** Como o ouro não separa nada, a
decisão se muda pra duas perguntas que o piwdex já sabia responder separadas e nunca
tinha cruzado: de onde cai a pedra (índice reverso de drop + hunts) e o que a espécie
vale em combate (tier list do meta). A tela põe as duas colunas lado a lado, e elas
discordam — Flareon lidera o combate (S, 83,8) e pede a pior pedra até o nível 60. Virou
seção nova em [[Ordene pela grandeza que decide, não pela que impressiona]].

A estrela é radial de propósito: cinco destinos simultâneos, excludentes e de mesmo
custo não são uma fila, e numa fila o primeiro pareceria o padrão.

### O corte da aresta falsa abriu um buraco

`evolutionChainOf` seguia `evolvesToId` cegamente e desenhava "Eevee -> nv 80 ->
Vaporeon" em SEIS fichas (a do Eevee e a dos cinco destinos, porque a cadeia também se
caminha pra trás). A aresta foi recusada na derivação, num lugar só.

E aí o painel caiu no texto padrão: *"Eevee não evolui e não vem de nenhuma evolução"* —
falso pelo outro lado, nas mesmas seis fichas. Virou o princípio
[[Tirar o dado errado não põe a verdade no lugar]], que é o aprendizado mais transferível
desta passada. A ficha agora diz o que acontece, com o custo, e leva pra `/eevee`.

### Duas coisas que o teste de navegador pegou

- **A linha da estrela atravessava o cartão.** Ia de centro a centro, e virava um espeto
  no meio do pokémon. Agora é um trecho do raio, de fora do disco central até a borda do
  cartão, em fração e não em pixel — a figura é a mesma em 320px e em 560px.
- **26px de rolagem lateral no telefone**, silenciosos. Item de grade nasce com
  `min-width: auto`, então a trilha cresceu até o min-content do painel de pedras (374px
  num aparelho de 360) e empurrou a página. `grid-cols-[minmax(0,1fr)]` na coluna base;
  quem cede passou a ser o nome da criatura, que já tinha `truncate`. Mesma família de
  [[Faixa que sangra estoura pela barra de rolagem, e o corte é na raiz]].

### O que ficou declarado como não sabido

Se a troca devolve o pokémon no nível em que ele entrou. O botão do jogo diz "Trocar",
não "Evoluir", e a Pokepedia só garante herança de nível/quality pra evolução comum — o
Eevee ela declara caso à parte sem dizer o que ele herda. Por isso os stats projetados
comparam as cinco espécies na mesma régua e a tela diz, colada nos números, que não é
promessa do que chega no time.

### O Eduardo pegou o mesmo defeito duas vezes, e a segunda foi minha

*"Não tá muito robusto não, ele tá usando stats base? Preciso informar os meus."*

A primeira versão da ferramenta pedia nível e quality e supunha IV 21 nos seis. É
**exatamente o defeito que o Stadium já tinha cometido e corrigido no dia anterior**,
depois da mesma reclamação — e eu reescrevi ele numa tela nova algumas horas depois.
Vale registrar sem enfeite: o padrão certo existia, estava no arquivo ao lado, e ainda
assim a tela nova nasceu supondo. Corrigir um defeito não o remove do repertório.

O conserto é o mesmo: os seis stats viram campo e o IV sai lido de volta por `lerIvs`.
A ordem da confiança ficou escrita no código — leitura dos stats, IV da carta guardada,
IV 21 por último — e leitura **impossível** não entra, porque `lerIvs` trava tudo no
teto e usar isso afirmaria um Eevee perfeito que ninguém digitou. Enquanto os stats não
vierem, a estrela avisa que os cinco ramos estão projetando um genérico:
[[A tela não afirma mais precisão do que a fonte tem]].

Verificado no navegador semeando IV conhecido (25/30/12/28/19/31 em nível 50, quality
1,8): a leitura devolveu os seis exatos, 145 de 192.

## A página de atualizações (25/08/2026)

O Eduardo pediu, e a razão de ela existir é mais específica do que "changelog": **um
conserto de CÁLCULO muda a resposta que alguém já tomou como boa**. Quem montou time
contra um boss antes de a penalidade de grupo entrar na conta saiu daqui com um "você
ganha" que o jogo desmentiu. Sem uma página que diga "isto mudou no dia tal", a única
leitura possível pra essa pessoa é que a ferramenta erra — e não que ela errava, foi
corrigida, e a correção tem data.

Duas decisões que valem lembrar:

- **Escrita à mão, não gerada do `git log`.** O histórico conta a mudança pelo que foi
  mexido no código, em granularidade de commit, incluindo dezenas de passadas que
  ninguém de fora percebe. Geraria uma lista longa, técnica e sem hierarquia — que é o
  que faz ninguém ler changelog. Aqui entra o que muda a RESPOSTA, e a frase diz o
  efeito, não o mecanismo.
- **Sem aba separando novidade de correção.** A correção é a parte que mais importa e
  não pode ficar numa segunda página que ninguém abre. O tipo é selo na linha.

E um defeito que o teste pegou: o selo de tipo estava herdando a cor da FERRAMENTA, e
aí "Conserto" e "Melhoria" do mesmo Stadium saíam os dois em verde-limão. Duas
perguntas, duas cores — a barra lateral veste a ferramenta ("onde mexeu"), o selo veste
o tipo ("o que aconteceu"). As três últimas aparecem na home, antes das cenas de
ferramenta, porque são a única informação da home que envelhece.

## TM, a nona ferramenta (25/08/2026)

Perguntado "que mais dá pra fazer", olhei o que o catálogo sustenta antes de sugerir,
e a resposta apareceu num número: **o melhor golpe natural das 482 espécies rende
43,3 de poder por segundo e TODO golpe de TM rende 60**. Não é upgrade incremental, é
outra categoria — e nenhuma tela do jogo nem do piwtools diz onde ele rende. Mecânica
e números em [[Poke Idle World - TM e os discos]].

O que fez a ferramenta valer é a segunda coincidência de desenho do mesmo jogo: **o
Researcher cobra o mesmo tanto de peças por qualquer disco**, exatamente como as cinco
trocas do Eevee custam o mesmo. Duas vezes seguidas o preço não separa nada. Virou
parágrafo em [[Ordene pela grandeza que decide, não pela que impressiona]] — onde o
custo é uniforme de propósito, o sistema está dizendo "não é aqui que você escolhe".

### O aprendizado que ficou: ordenar pelo salto

A lista ordena por quanto o disco TRANSFORMA e não por quem termina batendo mais, e as
duas respostas divergem de um jeito que decide errado:

| | Sem disco | Com disco | Salto |
|---|---|---|---|
| Scizor | 15.145 | 26.811 | 1,77x |
| Jolteon | 1.495 | 11.395 | **7,62x** |

Ordenado pelo resultado, o Scizor lidera — e ele é o que menos precisa. O Jolteon sobe
de tier **B pra S** com o mesmo disco. Virou
[[Recurso indivisível se aloca pelo salto, não pelo resultado]], irmã da nota do bônus
com teto: mesmo defeito por outra causa.

Um presente que caiu no colo e vale registrar: **a razão é invariante ao nível**. Os
dois lados usam o mesmo stat multiplicado pelo mesmo fator de nível e quality, então
ele cancela — a ordem vale pra todo mundo e se calcula uma vez. O que não cancela é o
IV, e só quando o golpe natural e o do TM usam stats diferentes.

### Três armadilhas do catálogo, todas na tela

- `Draconic Soul`, o TM de Dragão, tem **300 de poder e não 600** — o único dos quinze
  pela metade.
- **Normal, Aço e Fada têm disco e nenhum golpe.** Eles ficam na grade, apagados, em
  vez de serem filtrados: virou a seção preventiva de
  [[Tirar o dado errado não põe a verdade no lugar]]. Ausência que não aparece parece
  decisão de quem fez a tela.
- O `AoE TM Disk` não ensina golpe — faz os Normais acertarem em área —, e o efeito
  não está no moveset publicado. A tela diz o que ele faz e para aí.

### Um defeito de desenho que o teste pegou

Os dezoito discos elementais compartilham **o mesmo arquivo de ícone** no jogo
(`tm_disk_elemental.png`). A grade nasceu mostrando ele em cada célula, ao lado do
símbolo de tipo: dezoito imagens idênticas roubando o lugar do único glifo que
separava uma célula da outra. Saiu da grade e ficou uma vez, no título do painel.
Repetição idêntica não é informação — é ruído com aparência de identidade.

## Próximos passos

1. **Subir o serviço do robô no Railway**: segundo serviço apontando pro mesmo repo com
   `PIW_ROLE=bot`, Postgres, DNS de `bot.piwdex.com.br`, e — o passo que faz a separação
   valer — **Watch Paths** restritos no serviço do robô. Sem eles ele redeploya junto com
   cada ajuste da dex, e volta o problema que motivou o subdomínio. Guia em `docs/railway.md`.
2. **`NEXT_PUBLIC_BOT_URL` no serviço da DEX é o interruptor** de `/vip` e `/bot-app`:
   enquanto estiver vazia, os dois seguem caindo na home. Preencher só depois do DNS
   responder — [[Ponte pro endereço novo só se levanta quando o outro lado responde]].
3. **Provar o laço de caçada** com token real (toma a sessão de jogo — é ato do Eduardo).
4. Decidir o que volta do `parked/` depois do núcleo: mercado e sniper são os candidatos
   com mais código pronto; alertas e venda automática, os de maior valor por linha.
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

## O robô: por que ele "não funcionava" (23/08/2026)

O painel mostrava a caçada alternando entre `sessão perdida` e `conectando`, com os seis
números em travessão. Nenhum log acusava nada, e do lado de fora só dava para dizer "não
funciona porra nenhuma" — que foi exatamente o relato.

A causa não era a conexão: era o motor **descartar o código com que o jogo fechava o
socket**. Sondando o servidor na mão, ele responde `4001 unauthorized` para token ruim e
`4003 wrong-shard` para shard remanejado. Os dois viravam `chutado`, e os dois recebiam o
mesmo tratamento — backoff e tenta de novo —, que é o que não resolve nem um nem outro. O
robô batia na porta para sempre e a tela não tinha o que dizer. Ver
[[O código com que o socket fecha é a classificação que o retry precisa]].

Junto saiu o segundo estado invisível: sem teto de abertura, um socket que não completa o
handshake não emite `open`, `close` nem `error`, e o agendador nunca era chamado. Era o
`conectando há 6s` do print. Ver
[[Socket que não abre não emite evento, e só um temporizador percebe]].

Verificado de ponta a ponta com um token propositalmente inválido: o motor agora para em
`vencido`, grava `expired` no vínculo e a tela manda colar o token novo, em vez de
reconectar indefinidamente.

### O que faltava para ser robô

Segurar uma sessão numa hunt não é jogar sozinho. Bola zerada trava a fila de captura do
jogo, e uma caçada boa queima centenas por hora — então o robô passava a maior parte do
tempo sem capturar nada. Entraram as automações que faltavam: reposição de bola, poção e
revive por piso/alvo com teto de gasto, venda de drop por lista branca, venda de pokémon
por vetos, e o Auto-Helper do jogo (captura, poção e revive automáticos rodam no servidor
deles). Compra e venda vão por REST de propósito: REST não disputa a sessão, então a
reposição acontece com a caçada correndo.

O painel virou três abas (Caçada, Automação, Registro), e o registro é gravado — o robô
trabalha quando ninguém está olhando, e o que aconteceu de madrugada só existe se estiver
no banco.

### Ligar deixou de significar caçar (24/08/2026)

O botão fazia duas coisas: tomava a sessão de jogo e entrava numa caçada. Com uma tarefa
só isso passa; com quatro (caçar, vender, repor, chat) ficou claro que a caçada tinha
virado pré-requisito das outras três, e que trocar de hunt passava por desligar o robô —
devolvendo a sessão para a aba do navegador no meio. Agora `/ligar` só adquire e segura, e
`/cacar` entra, troca ou sai do campo pelo socket que já está de pé. Ver
[[Adquirir o recurso exclusivo é uma ação, usá-lo é outra]].

O chat entrou junto, porque chega de graça no mesmo socket. Ele não tem estado para reler:
a confirmação do envio é o próprio jogo ecoar a mensagem de volta com o seu nome, e recusa
é um frame de sistema sem remetente. Só envio confirmado arma o cooldown de ~1 min — recusa
corrige e manda de novo na hora. Ver
[[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]].

Dois vazamentos fechados no caminho: a fila de captura ia inteira no stream uma vez por
segundo (e sem auto-catch ela só cresce), e a venda de pokémon não tinha trava de
concorrência — o `pokes-get` de confirmação chega a meio segundo do poll, e as duas rodadas
liam o mesmo box.

### A conta inteira, e a caçada que sobe sozinha (24/08/2026)

A Idle Ball não aparecia no seletor da captura automática, e a causa vale mais que o
sintoma: os dois seletores de bola liam o catálogo da LOJA, e a Idle Ball não está à
venda. Catálogo e posse respondem perguntas diferentes e divergem só na ausência — que
não dá erro. Ver
[[Duas listas parecidas respondem perguntas diferentes, e a errada some com o item]].

Qualidade virou trava de venda ao lado do IV: são grandezas diferentes, e IV médio com
qualidade DIVINA vale mais que IV alto com qualidade comum. A tela pergunta pela faixa
(`qualityTier`) porque é assim que a decisão se pensa; o piso de cada faixa passou a sair
de uma tabela só (`TIER_MIN`) em `rarity.ts`.

A caçada automática até um nível **não ganhou motor próprio**: chama o mesmo `buildRoute`
da ferramenta pública de rota. O que a camada do robô acrescenta é o slug — o motor
raciocina em espécie e o `enter-hunt` quer o ponto no mapa. Verificado ponta a ponta:
Bulbasaur nv20 → nv60 sai em três faixas (Kabuto, Larvitar, Graveler), e os três slugs
existem no catálogo do jogo. A prévia da rota não exige ligar nada, porque decidir se
concorda com o plano depois de ligá-lo é a ordem errada para um robô que joga sozinho por
horas.

Os IVs individuais o jogo não manda (só `ivTotal`), então a estimativa espalha o total
igualmente pelos seis: erra a distribuição, acerta o montante, e o montante é o que domina
o dano. Chutar distribuição enviesaria o ranking de alvos sem evidência.

### O F5 e a confirmação (24/08/2026)

Duas coisas se passavam por "o robô resetou no F5" e nenhuma era o F5. O painel abria em
`parado` com tudo em travessão até o primeiro frame do stream — mas a página roda no MESMO
processo do motor, então passou a entregar o estado vivo já no primeiro render. E o placar
das automações só existia em memória: um deploy apagava a conta do dia. Foi pro banco,
volta no boot, e zera só quando o robô é desligado de fato.

No caminho, o log do boot mentia: `1/1 sessao(oes) retomada(s)` para um alvo que tinha
saído na primeira linha por token vencido. Cheguei a investigar se o `globalThis` era
compartilhado entre o contexto da instrumentação e o das rotas (é — mesmo pid) porque a
única evidência afirmava que a retomada tinha acontecido. Ver
[[Contador que conta sucesso de promessa afirma que deu certo]].

A aba de automação virou rascunho com salvar/descartar. Antes, marcar "vender o que o box
acumula" já valia — e venda de pokémon não desfaz. A automação do jogo entrou no mesmo
rascunho apesar de ir por outra rota: para quem mexe é tudo "as configurações do robô".

Bolsa e mochila se separaram pelo mesmo critério que já tinha resolvido a Idle Ball: poção,
revive e bola são o que a caçada GASTA; drop é o que ela PRODUZ. Estavam na mesma lista de
venda, e um "marcar tudo" deixaria a conta sem cura no meio da noite.

### A ficha de pokémon, e o IV 340 (24/08/2026)

O modal de pokémon virou um componente só, aberto do topo, do time, da conta e do chat — e
já pronto para o mercado, que é a próxima fonte a fazer a mesma pergunta. Ele reusa
`estimateIvs`, a inversão de fórmula da calculadora pública.

Foi bom ter testado com dado real: contra um Lucario do chat, a inversão devolveu **IV 340
num teto de 32**. A fórmula estava certa; a escala dos stats que aquela fonte manda, não. A
ficha passou a só exibir o IV por atributo quando ele fecha com o `ivTotal` que o próprio
jogo declara. Ver [[Fórmula verificada só vale na escala em que foi verificada]].

O chat ganhou os cartões: `[poke!<b64>]` e `[item!<b64>]` são JSON em base64
(`{k,n,lv,sh,q,iv,pw,t1,t2,st}` e `{k,ic,cat,npc,d}`) — antes apareciam como trezentos
caracteres de lixo no meio da conversa. Bloco que não decodifica volta como texto.

O topo do painel mudou de assunto: como ele fica em toda aba, passou a mostrar só o que
vale em todas (sessão, pokémon ativo com vida e XP, treinador, ouro, diamante, bolsa). O
seletor de caçada desceu para a aba de caçada.

### O deploy travava a caçada (24/08/2026)

`enter-hunt` é frame sem resposta. A conexão que nasce logo depois de um deploy pega o
servidor do jogo no meio da troca e às vezes perde esse frame — e o que sobra é o pior
estado possível: socket aberto, sessão válida, analyzer zerado, tela dizendo "a conexão
está aberta e o jogo não está mandando combate", indefinidamente. O robô segurava a conta
do dono a noite inteira sem caçar, e nada tinha falhado. Ver
[[Comando sem resposta precisa de vigia, não de fé]].

Duas correções de padrão vieram junto, e as duas eram erro meu de não reusar o que já
existia: a moeda do jogo é **dólar**, não ouro (o rótulo errado atravessou quatro telas com
um ícone de moeda genérica sustentando a leitura), e eu tinha escrito uma barra lisa
própria no painel do robô em vez de usar `Segments` — o medidor em blocos que a dex e a
calculadora já usam. `Medidor` passou a embrulhar `Segments`: um arquivo mudou, todas as
barras acompanharam.

O chat guarda 300 mensagens e conta só as não lidas. As 300 só cabem porque ele saiu do
estado e ganhou evento próprio no stream: dentro do estado, que sai uma vez por segundo,
seriam ~45KB/s reenviando a mesma conversa.

### Objetivo, e o chat que dizia ter falhado (24/08/2026)

"Quero mais dinheiro" virou conta: `motor/objetivo.ts` cruza o time com o catálogo de
alvos pelo mesmo `rankHunts` da ferramenta pública e devolve os pares (meu pokémon ×
caçada) ordenados por dólares/h. Alvo letal fica fora do ranking — ele às vezes lidera o
ouro por hora, e lidera até o primeiro desmaio. Verificado com um time de seis: Golem nv400
→ Brave Charizard a 296k/h por dinheiro, e → Furious Scyther a 10M xp/h por XP, que é a
mesma hunt que o Eduardo tinha escolhido na mão.

Os dois objetivos (dinheiro, nível) viraram uma ESCOLHA e não duas chaves: disputam o mesmo
par de valores (quem é o líder, em que campo). Ver
[[Objetivo é exclusivo, interruptor é combinável]].

O chat dizia "o jogo não confirmou o envio" numa mensagem que tinha entrado — o eco levava
mais que os 6s de prazo. Sem eco não é sem envio: o prazo subiu, e no silêncio o cooldown
arma mesmo assim, porque reenviar num canal com anti-flood de um minuto ou duplica a
mensagem ou queima a janela.

Mensagem privada ainda não dá: o jogo tem, e o formato não está em captura nenhuma que eu
tenha. Em vez de adivinhar o nome do frame, o motor passou a gravar a forma dos frames
desconhecidos (`/api/robo/frames`) — implementar contra evidência, não contra suposição.

### Três reclamações, uma causa (24/08/2026)

Barra fora do padrão, selo de raridade mais baixo que os badges de tipo, e a chave
liga/desliga ilegível. As três eram primitivo escrito de novo em vez de reusado — e a
segunda rodada de reclamação sobre a barra foi o que deixou a lição clara: eu tinha
corrigido o wrapper e deixado uma cópia local dentro do modal, que é onde há mais barras na
tela. Corrigir uma cópia não encontra as outras. Ver
[[O primitivo só padroniza o que passa por dentro dele]].

A chave ganhou desenho novo (40x20, miolo de 12, fundo e rótulo acendendo juntos) e uma
prop `block`: numa grade de cartões cada rótulo tem tamanho diferente, e encolher até o
texto desenhava quatro larguras. Na fila de filtros da dex o padrão continua encolhendo.
De quebra, o realce do rótulo nunca tinha funcionado — usava `group-has-[:checked]` sem
`group` na casca.

### O objetivo mostrava e não ia (24/08/2026)

Escolher um objetivo caía na mesma trava anti-oscilação das decisões automáticas: leitura a
cada 20s mais 60s de freio, então o clique podia levar um minuto e meio para virar alguma
coisa na tela. Ver
[[Freio de oscilação vale para a máquina, não para a ordem de quem manda]].

Junto: a rota montava um plano por tecla (digitar "500" pedia rota para 5, 50 e 500), e o
seletor de caçada continuava mostrando o que a pessoa tinha digitado enquanto o robô caçava
em outro lugar. O nível virou rascunho com botão, e o seletor passou a acompanhar a caçada
que está no ar — travado enquanto o objetivo comanda, porque aceitar uma ordem que a
próxima reavaliação desfaz em segundos é pior que recusar.

### "Volte para manual" (24/08/2026)

Escolher um objetivo já entregava o comando ao robô, e o seletor manual travava com um aviso
mandando voltar. Era eu obrigando a desfazer para poder olhar. O modo virou estado local (o
que estou vendo) e o objetivo continuou no banco (o que a máquina persegue), com um botão
explícito entre os dois; e começar uma caçada no manual desliga o piloto sozinho, em vez de
recusar. Ver [[Ver o plano e mandar executar são duas ações]].

Terceira vez na mesma pedra do primitivo: os painéis do robô estavam lisos ao lado das
fichas da dex porque eu desenhava um `<h2>` solto no corpo em vez de usar o `title`/`actions`
do `Panel` — que é o que dá a barra com divisória e o vidro. Onze painéis convertidos.

### O balcão virou aba, e a reposição parou de comprar às cegas (23/08/2026)

**A aba Automação guardava duas coisas que não se parecem.** De um lado o Auto-Helper —
capturar, beber poção e reviver sozinho —, que roda no servidor do JOGO: o robô só liga o
interruptor, e quando não pega o motivo é o VIP de lá. Do outro, repor consumível e vender:
chamada REST nossa, que gasta ouro e destrói pokémon. O Eduardo cortou pelo **uso**:
automação é o que se faz com o que a conta já tem, e comprar/vender é balcão, então virou
aba **Loja**. O ganho não é arrumação — juntas, a decisão de quanto gastar ficava a três
rolagens da de quanto se recebe, que é a única comparação que essa tela precisa permitir.

A Loja abre no caixa da sessão (dólares, comprou, vendeu, saldo), traz a reposição inteira
e as duas vendas lado a lado. O "rodar agora" veio junto e agora **relê a bolsa depois de
rodar**: um botão que não mostra o resultado é um botão que pede fé.

**E o bug que a tela nova expôs.** Cada cartão de consumível passou a mostrar o ESTOQUE ao
lado do piso — configurar "abaixo de 25 poções" sem o número que os 25 comparam era
escolher no escuro. Ao ligar os dois, apareceu que o motor contava poção e revive pelo
frame `inventory` do WebSocket. O frame é mais fresco que qualquer REST, e era essa a razão
de usá-lo, mas ele **nasce vazio a cada conexão e é limpo quando ela cai**. Bolsa vazia lia
como "zero poções", zero fura qualquer piso, e piso furado é compra: uma conta com 400
poções comprava 100 a cada minuto enquanto o socket não mandasse o primeiro frame. Agora a
decisão lê a bolsa por REST na hora — como a própria doc do arquivo já mandava ("nada roda
às cegas") e como a compra de bola sempre fez — e quando o catálogo não responde a função
devolve `null` em vez de zero, porque não saber quanto tem é razão pra NÃO comprar. Virou
[[Ausência de leitura cai no valor que dispara a ação]], que é a terceira vez desta pedra
no projeto (as outras duas: [[Sonda que falhou não é sinal de que mudou]] e
[[Zero na tela é afirmação, não valor de conforto]]).

A mesma função que decide a compra é a que serve o estoque pra tela, pela rota da loja.
Uma segunda contagem lá daria uma tela dizendo 30 e um robô comprando como se fossem 12 —
e a que ninguém revisa seria a de lá.

**E "o piso por item" saiu de pendência para bug, no dia seguinte.** Com o estoque na
tela, o Eduardo mostrou o robô com a reposição ligada e sem comprar nada: 555 Poké Ball e
41 Idle Ball na bolsa, piso em 150 — e o auto-catch configurado em **Ultra Ball**, da qual
havia **zero**. O piso somava a categoria inteira, então 596 passava folgado, enquanto a
bola que o jogo de fato joga estava zerada e a captura ficava parada. Bolsa cheia da bola
errada não é bolsa cheia, e o sintoma foi inércia: nada quebrou, nada foi logado, os dois
lados estavam certos isoladamente. Virou
[[Limiar conta a unidade que se consome, não o balde que a contém]] — segunda aparição de
[[Espelhar por balde esconde item no lugar errado]] no vault, agora do lado de quem decide
por limiar.

`estoqueDoAlvo` mora em `motor/tipos.ts` e não no motor porque a TELA compara o mesmo
número: o cartão mostra o estoque ao lado do piso, e duas contagens dariam uma tela
dizendo 596 e um robô decidindo por 0. Pela mesma razão a rota da loja parou de mandar
total pronto e manda a bolsa item a item.

O caso ganhou aviso na tela: bola do auto-catch zerada agora é uma frase dizendo que o
jogo não vai capturar nada e que ter outra bola não resolve. Na mesma passada, os quatro
cartões da automação viraram grade de verdade (slot de ajuste com altura fixa, presente
até no cartão que não tem ajuste; limiar da poção em linha em vez de rótulo empilhado), e
o hint passou a dizer **o que não dá para escolher** — só a bola se escolhe, porque para
poção e revive o jogo usa o que estiver na bolsa e não expõe a escolha. A pergunta "onde
escolho a poção?" não tinha resposta dentro da tela.

**E eu tinha respondido errado sobre a poção.** O Eduardo perguntou por que não dava para
escolher revive e poção na aba de automação; eu respondi que o jogo não expõe a escolha,
baseado só em `CAMPOS_AUTO`, que é a nossa lista branca — quer dizer, no que a gente já
tinha mapeado, e não no que o jogo tem. Ele insistiu, e insistir estava certo: o bundle
público do jogo traz o bloco i18n `autoHelper` inteiro, com **três** seletores
(`Pokébola:`, `Pokébola (shiny):`, `Potion:`), o padrão `Automático (melhor)`, e mais um
filtro de captura por nome que ninguém tinha visto. Só o revive não tem seletor mesmo — a
dica do próprio jogo é "usa Revive ao desmaiar", o item, no singular.

O que o bundle NÃO deu foi o nome do campo: a i18n vem num chunk público, o componente que
chama a API só carrega depois do login. Chutar `autoPotionId` seria pior que não ter — API
tolerante ignora chave desconhecida, responde 200, e a tela mostraria uma escolha que nunca
aconteceu. Então o campo se descobre no payload da conta, pela forma, e a escrita volta na
mesma chave que leu. Virou
[[Campo cujo nome você não sabe se lê do payload, nunca se chuta]], e a nota do bundle
ganhou o capítulo do chunk atrás do login —
[[O bundle público do cliente entrega o contrato da API sem documentação]].

A lição de ofício é mais curta que a técnica: **"a nossa lista branca" não é resposta para
"o que o jogo tem"**. Eu li o nosso mapeamento e falei como se fosse o território.

Ficou de fora, de propósito: **compra manual** — a Loja hoje só configura a automática — e
o **filtro de captura por nome**, que o jogo tem e o painel ainda não expõe.

### AdSense: a conta entrou, e o ads.txt virou rota (24/08/2026)

Conta aprovada (`ca-pub-5164828819712988`), e o passo era a verificação — o Google pede o
script no `<head>` e nada mais. Toda a infraestrutura já estava pronta e desligada desde a
camada de anúncios, então "ligar" é uma variável de build no Railway:
`NEXT_PUBLIC_ADSENSE_CLIENT`. Com a conta e **sem nenhum slot**, o site carrega o script,
emite a meta `google-adsense-account` e não desenha caixa de anúncio nenhuma — que é
exatamente o que a verificação precisa.

O que mudou no código foi o `ads.txt`. Ele era arquivo em `public/` com um aviso escrito
pra si mesmo: "o id também entra em `lib/ads.ts`, os dois precisam bater". Virou rota
(`src/app/ads.txt/route.ts`), derivando o publisher id do MESMO `ADSENSE_CLIENT` que monta
o script, sem o prefixo `ca-`. `robots.ts` e `sitemap.ts` já eram gerados; o `ads.txt` era
o arquivo público que ainda repetia um valor do código.

A razão não é elegância: **o modo de falha da divergência é silencioso e caro**. O site
serve anúncio normalmente enquanto o AdSense trata o inventário como não autorizado, e
isso aparece semanas depois como receita que não chega. Agora preencher a variável liga o
script e a autorização juntos, e esquecê-la desliga os dois juntos — em vez de produzir
meia configuração. O arquivo estático foi removido junto, senão `public/` venceria a rota
e ela seria código morto que passa em qualquer teste.

Conferido servindo o build de produção nos dois estados: com a variável, `ads.txt` traz a
linha e a home traz script e meta, com zero caixas de anúncio na dex; sem ela, só
comentário. `.env.example` passou a dizer por que a variável fica vazia em
desenvolvimento — preenchida, ela carrega o script real do Google em localhost, que é
impressão inválida na própria conta.

### O registro sem valor, e a lista que afirmava seis caçadas (24/08/2026)

Duas leituras do Eduardo sobre a mesma tela, e a segunda achou um erro de fato.

**O registro dizia "191 itens vendidos" e não dizia por quanto.** O número sempre
esteve gravado — `aplicarRecados` grava `{ ouro, quantidade }` em `data` desde o primeiro
dia — e a tela nunca leu. Faltava justamente a metade que responde "valeu a pena deixar
isso ligado". Agora cada linha traz o movimento com sinal e cor, e zero não vira "+0"
(recusa grava `ouro: 0`, e um zero ao lado de "não comprou" é ruído com cara de dado).

Junto veio um caixa que acompanha o **filtro**, não o total: filtrar por "essence of fire"
e ver quanto aquele drop rendeu em quatorze dias é uma conta que não tinha como ser feita.
Total fixo no topo responde sempre a mesma coisa.

**A lista de "mais dinheiro" dizia "caçando X" nas seis linhas.** O Eduardo perguntou
"como que cada um tá caçando um? dá pra fazer isso?" — e não dá: o jogo entrega uma
sessão, com um líder, em um campo. Era um ranking de pares lido como seis caçadas
simultâneas. Nada estava errado ali além do verbo, e o verbo mentia seis vezes. Virou
[[Ranking de opções não usa o verbo do estado ao vivo]].

O detalhe que quase passou: o realce de "está rodando" saía da POSIÇÃO (`i === 0`). Num
sistema que reavalia, o ranking muda antes da troca acontecer — o topo vira o candidato
novo enquanto o motor ainda executa o antigo, e o destaque aponta pra linha errada
exatamente durante a mudança. Agora casa com `estado.slug`.

### Várias contas de jogo por assinante (24/08/2026)

O modelo dizia "uma conta por pessoa" da forma mais dura possível: `user_id` era a CHAVE
PRIMÁRIA de `game_links` e de `robot_sessions`. Não era regra de código — era a estrutura
do banco, e por isso não havia como duplicar por engano.

A chave passou a ser o VÍNCULO. Tudo que parecia ser "do usuário" era na verdade "da
conta": estado desejado, config das automações, placar, shard, snapshot de time. Um
WebSocket por conta, todas rodando ao mesmo tempo. O que continua do usuário é o que ele
paga e o que ele pode ver — `robot_events` guarda os dois ids, porque "o que aconteceu
comigo" é a pergunta sobre todas as contas.

O refactor foi MUDO: 27 arquivos, e nenhum tipo mudou (os dois ids são `string`). O que
achou as chamadas foi mudar a FORMA — aridade em `registrarEvento`, objeto `DonoDaSessao`
em `segurar`. Virou
[[Re-chavear um sistema é refactor mudo, force o compilador a achar as chamadas]].

Teto de 5 contas por assinatura, numa constante só: cada conta é um socket aberto o tempo
todo mais o poll do analyzer, e o processo é um só (`numReplicas: 1`).

### Coleta e o Flint: o dinheiro que estava parado (24/08/2026)

Uma captura completa do jogo — todas as ações feitas à mão — entregou o contrato de tudo
que ficava de fora do robô por não passar pela loja nem pelo campo. Três eram dinheiro
parado: **diária**, **passe de batalha** (missão concluída e tier alcançado esperando
clique) e o **Flint**, o NPC de Pewter que compra PEDRA por um preço por unidade que a
loja comum não paga — 413 Cocoon Stone a 5.000 são dois milhões que estavam na mochila.

Duas decisões: a coleta é a única automação do robô **sem lista branca e sem teto**
(recusar presente não protege ninguém de nada), e tem **relógio próprio** de 10 minutos —
no ritmo do minuto seriam mil chamadas por dia para ouvir "já coletou". Pedra vai por
lista branca com reserva, porque é material de evolução e vender não desfaz.

O mapa inteiro da API, inclusive o que ficou de fora e por quê, está em
`docs/api-do-jogo.md` no repositório. Mercado, tasks, streak, breeding e gifts aparecem na
captura só como LEITURA: o POST de cada um não foi capturado, e implementar contra
suposição ali mandaria um corpo que o jogo ignora e responde 200 — ver
[[Campo cujo nome você não sabe se lê do payload, nunca se chuta]].

### Uma conta que quebra parava a caçada de todas (24/08/2026)

Não havia `uncaughtException` nem `unhandledRejection` em lugar nenhum do projeto, e havia
**16 disparos `void this.algumaCoisa()`** espalhados pelo motor. No Node moderno, promessa
rejeitada sem `catch` derruba o processo — e o processo é um só, com o WebSocket de todas
as contas dentro. Uma venda que falhasse numa conta levava a caçada das outras dezenove.

O sintoma era o pior possível de diagnosticar: o boot religa sozinho e o log escreve
"Ready" igual ao do processo que rodava havia horas. Mesma pegadinha do deploy que derrubava
a caçada, e mesma lição da nota de log — processo que renasce parece processo que nasceu.

Três camadas, da origem pra fora: `protegido(oQue, fn)` na sessão envolvendo os disparos e
os handlers do socket; o erro virando evento `falha` no feed do dono com throttle de 10
minutos por trabalho; e `motor/guarda.ts` segurando o que ninguém previu **sem** matar o
processo. A guarda contraria o conselho padrão de propósito, e é essa a decisão que virou
[[Processo que segura sessão viva não morre em exceção não tratada]]: crash-only pressupõe
um servidor sem estado, onde o request perdido o cliente repete. Aqui não — morrer troca
uma inconsistência possível por uma perda certa.

O `message` do socket ganhou proteção antes de todos: é o único ponto que roda código
nosso sobre dado que o JOGO escolheu, e o jogo muda de forma sem avisar.

O contador da guarda sai no `/api/health`, e isso não é enfeite: rede que apara em silêncio
é um jeito mais lento de esconder defeito. Número subindo é conserto na origem, não robô
saudável.

## O diário do catálogo (25/08/2026)

O jogo não publica changelog, e patch de balanceamento troca a resposta de TODAS as
ferramentas de uma vez. O sinal já existia e ninguém usava: o `source.ts` confere o ETag
do `creatures.json` quase a cada visita e sabe o segundo em que o catálogo mudou — só
que usava isso pra recarregar e jogava fora.

Agora a ingestão compara o catálogo baixado com o snapshot em disco **antes** de
sobrescrever, grava a diferença em `src/data/patches.json` e isso vira a rota
`/patches`, com ficha por patch. Uma rotina do GitHub Actions roda de 6 em 6h e commita
quando o jogo mexeu; de brinde, o snapshot de fallback parou de envelhecer.

A manchete de cada espécie é o **ouro por abate**, derivado, e não a lista de chances que
o causou. No patch de 20/08 o Ledian caiu de 492,8 para 37,5 (13,1x) — que é exatamente
o "493 pra 38, 13x" escrito à mão no cabeçalho do `source.ts` meses antes, e o motor
chegou lá sozinho, sem receber o número. Serviu de conferência do próprio motor.

O diário nasceu com esse patch, reconstruído de duas revisões do snapshot no git. Os
pares anteriores ficaram de fora porque misturam mudança do jogo com mudança da própria
ingestão: entre 16/08 e 20/08 o diff acusa 481 das 482 espécies "mudando de golpe", e
era o campo `tm` nascendo na normalização. Virou
[[Diferença entre duas leituras só fala do mundo se o instrumento não mudou]] e
[[Diff de catálogo externo carimba a versão do extrator]].

**Disciplina que o mecanismo exige:** mexeu na normalização do `ingest.mjs`? Suba o
`PIPELINE` em `src/lib/patches.ts`. Esquecer troca um erro barulhento (passada pulada,
dita no log) por um silencioso (patch inventado, publicado com data).

## A área logada ganhou a passada visual (25/08/2026)

O robô estava com os glifos todos certos e nenhuma arte: abria com um `h1` de 17px
em cima de um formulário enquanto a dex abre com faixa, ilustração e uma frase.
Dez artes novas (`robo*.svg`), um registro de telas (`src/lib/telas-robo.ts`, o
espelho do `FERRAMENTAS`) e dois pesos de chegada — faixa nas telas que se abrem,
cabeçalho de 44px nas seis abas do cockpit.

Duas decisões que valem lembrar:

- **A cor é uma só, e quem separa é a silhueta.** Na dex cada ferramenta tem a sua
  cor porque lá ela responde "onde estou" num site de dez destinos. Aqui são as
  telas de um produto, e dez matizes inventariariam dez identidades pra um painel.
- **A marca da barra deixou de ser a pokébola.** Ela é a marca do site inteiro, e
  a única tela que precisa se anunciar como OUTRO endereço estava se anunciando
  com o símbolo de onde a pessoa acabou de sair.

Duas artes voltaram pra prancheta depois de rasterizadas (a caçada lia como riscos
soltos, a loja lia como balde) — o teste de [[Arte de ícone se julga no tamanho de
uso, e o acento é a massa]] pagando de novo. E o par ilustração/glifo virou
[[Trocar arte por ícone de linha exige recalibrar tamanho, não só trocar o componente]].

## Conexões
- Substitui: [[piwdex]]
- Usa: [[Design]] · [[Infra]] · [[Frontend]] · [[Backend]]
- Aprendizados: [[Índice só é identidade enquanto a coleção não muda]] ·
  [[Sem servidor, a migração de dado local acontece na leitura]] ·
  [[Flutuante dentro de modal precisa vencer no z-index e no Escape]] ·
  [[A interface do sistema explica o que a API dele esconde]] ·
  [[Fator que domina o resultado não entra na conta por estimativa]] ·
  [[Fator que domina o resultado não entra na conta por estimativa]] ·
  [[Traduza o vocabulário do sistema, não o nome próprio]] ·
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
  [[Bônus condicional se avalia contra quem não o recebe]] ·
  [[Estimativa fraca informa, número verificado ordena]] ·
  [[Um processo serve dois hosts quando o papel vem do ambiente]] ·
  [[Ponte pro endereço novo só se levanta quando o outro lado responde]] ·
  [[Laço que trata toda falha igual apaga a causa da primeira]] ·
  [[Retry que reusa o cliente queimado esconde o erro da primeira tentativa]] ·
  [[O empacotador segue o valor importado, não o tipo]] ·
  [[O código com que o socket fecha é a classificação que o retry precisa]] ·
  [[Socket que não abre não emite evento, e só um temporizador percebe]] ·
  [[Adquirir o recurso exclusivo é uma ação, usá-lo é outra]] ·
  [[Duas listas parecidas respondem perguntas diferentes, e a errada some com o item]] ·
  [[Contador que conta sucesso de promessa afirma que deu certo]] ·
  [[Fórmula verificada só vale na escala em que foi verificada]] ·
  [[Comando sem resposta precisa de vigia, não de fé]] ·
  [[Objetivo é exclusivo, interruptor é combinável]] ·
  [[O primitivo só padroniza o que passa por dentro dele]] ·
  [[Freio de oscilação vale para a máquina, não para a ordem de quem manda]] ·
  [[Ver o plano e mandar executar são duas ações]] ·
  [[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]] ·
  [[Ausência de leitura cai no valor que dispara a ação]] ·
  [[Limiar conta a unidade que se consome, não o balde que a contém]] ·
  [[Campo cujo nome você não sabe se lê do payload, nunca se chuta]] ·
  [[Ranking de opções não usa o verbo do estado ao vivo]] ·
  [[Re-chavear um sistema é refactor mudo, force o compilador a achar as chamadas]] ·
  [[403 do escudo não é 403 do dono da API]] ·
  [[Trocar de sujeito na mesma rota não remonta, e o estado do anterior fica]] ·
  [[Processo que segura sessão viva não morre em exceção não tratada]] ·
  [[Nada que está na tela pode estar invisível esperando o scroll]] ·
  [[Peça desenhada fora do DOM é uma segunda implementação do tema, e ela envelhece calada]] ·
  [[Mais resolução não compra qualidade em ícone; trocar de meio compra]] ·
  [[Regra que veio de fora do sistema entra como chave declarada, não cravada no código]] ·
  [[Contador de popularidade conta votante, não evento]] ·
  [[Rotação por período se apura na leitura, e dispensa agendador]] ·
  [[Invertida a fórmula, o arredondamento é meia-aberto e o meio da faixa não é a resposta]] ·
  [[Invertida a fórmula, o arredondamento é meia-aberto e o meio da faixa não é a resposta]] ·
  [[Sinal booleano da fonte não ocupa o lugar de uma escala]] ·
  [[A mesma grandeza usa a mesma escada nas duas telas]] ·
  [[Trocar a arte de fundo é refazer a calibração, e a régua não é a média]] ·
  [[Trocar arte por ícone de linha exige recalibrar tamanho, não só trocar o componente]] ·
  [[Tirar o dado errado não põe a verdade no lugar]] ·
  [[Recurso indivisível se aloca pelo salto, não pelo resultado]] ·
  [[Diferença entre duas leituras só fala do mundo se o instrumento não mudou]] ·
  [[Diff de catálogo externo carimba a versão do extrator]] ·
  [[Produtividade se mede pela hora do registro, não pela data do fato]]
- Referência: [[Poke Idle World - endpoints publicos de dados]] · [[Poke Idle World - regras de breeding]] · [[Poke Idle World - evolucao e a troca do Eevee]] · [[Poke Idle World - TM e os discos]]
- Mapa: [[Projetos]]
