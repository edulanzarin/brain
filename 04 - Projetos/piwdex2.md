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
  esconde, projeta em qualquer nível e mostra o quanto falta pro IV perfeito.

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
  ferramenta do dado que ela apresenta. Virou a primitiva `Note`.
- **O IV estimado é FAIXA, não ponto.** O stat que o jogo mostra já veio arredondado, e
  o fator `(nível/100) × quality^exp` é tão pequeno em nível baixo que meia unidade de
  stat vale 8 pontos de IV — um Electrode nível 5 com IV 32 de verdade estima 34,3 no
  ponto. Isso também derruba a validação óbvia: entrada impossível é `piso da faixa > 32`,
  não `ponto > 32`. Virou
  [[Estimativa que inverte valor arredondado é faixa, não ponto]].
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
símbolo próprio: 20 ícones de domínio em pixel art 8x8, além dos 18 de tipo.

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

## Próximos passos

1. Hunt, Breeding e Meta sobre os motores já portados. A rota de hunt tem agora um
   segundo eixo pronto: o `items.ts` já sabe o que cada caçada rende de loot por abate.
2. Decidir o que substitui o robô. Até lá, `parked/` fica como está.
3. O `pokedex.png` tem 584 KB (arte gerada, com ruído) contra ~10 KB dos ícones
   desenhados por código. Quantizar em 64 cores derruba pra 57 KB sem diferença visível
   no tamanho em que ele aparece — decisão do Eduardo, é arte dele.
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
  [[Sticky gruda no container que rola, não na janela]]
- Referência: [[Poke Idle World - endpoints publicos de dados]] · [[Poke Idle World - regras de breeding]]
- Mapa: [[Projetos]]
