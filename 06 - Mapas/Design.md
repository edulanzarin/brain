---
tags: [tipo/moc]
criado: 2026-07-20
---

# Design

**O que se reusa entre projetos é o princípio, não a estética.** Os princípios (por
quê) moram em [[Base]] e sobrevivem a qualquer redesign. As técnicas daqui (como) são
um *dialeto* concreto — o dashboard denso "Aurora Glass" — traduzido em CSS, Tailwind
e React, ramificado por assunto. É uma opção de cara, boa pra sistema de trabalho tipo
dashboard, não uma linguagem que todo projeto tenha que herdar:
[[Estética é por projeto, princípio de design é que se reusa]].

Projeto novo herda os princípios sempre; puxa este dialeto só se for da mesma forma
(admin/dashboard). Landing, app de cliente ou mobile escolhem a própria estética.

## Cor e tema

`02 - Técnicas/Design/Cor e tema`

- [[Sistema de cores e tema do dashboard]] — os tokens semânticos, claro/escuro e o
  script que evita o flash no carregamento.
- [[Vidro flutuante precisa de superfície mais opaca que a chrome]] — dois níveis de
  vidro: chrome arejada vs overlay opaco e legível.
- [[Sobre arte de fundo, a chrome também tem piso de opacidade]] — quando o fundo é
  foto ou wallpaper, a premissa da nota acima cai: a opacidade é multiplicativa entre
  scrim, painel e campo, e o que faz o vidro parecer vidro é blur + aresta de luz +
  sombra, não o alpha.
- [[Trocar a arte de fundo é refazer a calibração, e a régua não é a média]] — o scrim
  se mede pelo perfil vertical da arte (a mais clara das três pediu o scrim mais
  ABERTO), e o desfoque se decide pelo gênero dela: ilustração tem assunto e o olho
  tenta lê-la, geometria não.
- [[Validar paleta de gráficos antes de escolher cores]] — forma primeiro, cor por
  último; checagens objetivas antes de fechar a paleta.
- [[Cor de marca precisa de variante acessível por tema]] — a cor crua do cliente
  reprova contraste como texto; derive uma variante por tema e compute.
- [[Propriedade com prefixo escrita à mão pode perder a versão padrão no build]] —
  `backdrop-filter` escrito cru em `@layer components` sai do build só com
  `-webkit-`, e o computado vira `none`. O vidro continua translúcido e parece
  certo; some só o desfoque, que é o que faz parecer vidro.
- [[Acento da interface é um token separado da cor de dado]] — quando a cor tem
  significado de dado (azul=entrada), o acento de UI vira outro token; senão
  retematizar a interface contamina os gráficos.

Princípios: [[Token semântico em vez de valor literal]] ·
[[Hierarquia por superfície, não por borda]]

## Layout e espaço

`02 - Técnicas/Design/Layout e espaço`

- [[Classes de componente vão em @layer components no Tailwind]] — pra a classe de
  componente vencer a utilitária sem `!important`.
- [[A classe do chamador só vence a do primitivo com tailwind-merge]] — quando o
  default do primitivo é utilitário, concatenar não basta: precisa de merge.
- [[Trocar a fonte muda a largura, não só o desenho da letra]] — a família nova
  reescreve os slots fixos, a base em rem e os pesos disponíveis.
- [[Margem negativa em item de flex centralizado vale metade]] — `align-items: center`
  centraliza a caixa de margem, então `-64px` sobe 32 e o resto depende da altura do
  vizinho mais alto. Retrato sobre capa pede `align-self: flex-start`, o tamanho num
  token e nenhum gap entre as duas peças.
- [[Alvo de toque pergunta pelo apontador, não pela largura da janela]] — `pointer:
  coarse` e não breakpoint: janela estreita no desktop continua com mouse. Traz os
  dois pisos (24 da WCAG, 44 da Apple) e como medir a área sensível de verdade.
- [[Área de toque cresce por pseudo-elemento, não pela caixa]] — link dentro de
  frase não pode engordar sem empurrar o parágrafo; cresce a área, não o elemento.
- [[Sticky gruda no container que rola, não na janela]] — `overflow-x-auto` promove o
  outro eixo pra `auto` e vira o ancestral rolável; o `top` passa a medir de dentro.
- [[Faixa que sangra estoura pela barra de rolagem, e o corte é na raiz]] — `100vw`
  mede a janela com a canaleta junto, então toda seção de borda a borda nasce larga
  pela espessura da barra. O corte é `overflow-x: clip` na raiz: no ancestral mais
  próximo ele mataria o próprio sangramento, e `hidden` soltaria todo sticky.

- [[Peça de grade mostrada sozinha vai centrada num palco]] — cartão fora da grade
  deixa meia linha vazia ao lado, e a sobra é lida como defeito da prévia. Centrar
  num palco de fundo diferente resolve a sobra e ainda diz que aquilo é amostra.

Princípios: [[Escala fechada em vez de valor solto]] ·
[[Container tem largura máxima e respiro constante]]

## Componentes

`02 - Técnicas/Design/Componentes` — índice: [[Padrões de componentes de dashboard]]

Antes de decidir o que fica aberto na tela:
[[O que responde pergunta rara não ocupa a rolagem de todo mundo]] — a altura é
orçamento pago por toda visita, e o gatilho do que se esconde carrega o estado que
decide se vale abrir.

- [[Sidebar em acordeão e layout de módulo]] — a estrutura fixa da tela.
- [[Controles de filtro do dashboard]] — toggle segmentado e dropdown de filtro.
- [[Escolha única e múltipla não usam o mesmo controle]] — caixa em tudo deixa marcar
  cabelo loiro E ruivo. Eixo vira chip, acumulável fica caixa, e a trava mora numa
  coluna do cadastro para eixo novo não custar deploy.
- [[Palavra da interface é lida com o dicionário do usuário, não com o seu]] — "seu
  endereço" era exato e virou "onde você mora" num classificado de acompanhantes; e
  "nome" em dois campos vira "isto substitui aquilo?". Fale da coisa, não da categoria
  dela, e troque no produto inteiro.
- [[Peça de mentira que não se anuncia vira fundação de coisa real]] — o botão
  de denunciar fechava o modal e dizia "enviada" sem escrever linha nenhuma;
  peça faltando se vê, peça de mentira passa em toda inspeção e vira base do
  que se constrói depois.
- [[Fila de decisão agrupa pelo alvo, não pelo aviso]] — sete denúncias do mesmo
  perfil são uma decisão, não sete. E a fila de decisão ordena por gravidade,
  ao contrário da fila de conferência, que ordena pela mais antiga.
- [[Tela que manda comparar duas coisas mostra as duas]] — a fila mandava
  comparar com as fotos do anúncio e não mostrava as fotos do anúncio; quem
  modera fez a metade possível e aprovou. Instrução que a interface não deixa
  cumprir é pior que instrução nenhuma.
- [[Código que a pessoa copia à mão não pode ter caractere ambíguo]] — sem 0/O
  e 1/I/L no alfabeto, e em tamanho de título: o ambíguo não gera erro, gera
  recusa de quem fez tudo certo, e nenhum log registra isso.
- [[Último passo sem desfecho transforma a régua em beco]] — "Passo 9 de 9" promete
  um fim; o rodapé do último passo tem que entregá-lo, no mesmo canto onde os outros
  põem o avançar. Quando o fim é uma espera de terceiro, o desfecho é "Concluir".
- [[Campo que trava depois de escolhido não vai no meio do formulário]] — escolha que
  não se desfaz sai da fila e vira o primeiro bloco, sozinha, com o custo dito antes,
  prévia do resultado e conferência enquanto digita. No meio de quinze campos, quem
  preenche está em modo de despachar.
- [[Seletor cria e gerencia os próprios itens]] — combobox que cria ao digitar e
  renomeia/exclui no painel, dispensando tela de CRUD do auxiliar.
- [[Blocos de dado - card, KPI e gráfico]] — card, stat tile, gráfico e tabela.
- [[Chip que serve a duas grandezas declara qual delas mostra]] — dois eixos que
  compartilham vocabulário (espécie x indivíduo, previsto x realizado) precisam do
  eixo declarado no componente, senão a mesma entidade se contradiz entre telas.
- [[A mesma grandeza usa a mesma escada nas duas telas]] — se duas telas respondem
  "quão raro é isto?", respondem com os mesmos nomes, cores e ordem; o que se
  compartilha é o TIPO, não umas cores parecidas. Traz o contrapeso: espelhar é da
  forma e da escada, nunca da contagem.
- [[Escada ordinal empresta a forma entre domínios, nunca os cortes]] — o contraponto
  da anterior: quando a escada mede uma grandeza CONTÍNUA (dias, reais), o que atravessa
  entre módulos é a forma — degraus, ordem, cores —, nunca os cortes. Unidade igual não
  é distribuição igual, e o primeiro degrau tem de ser o ciclo normal daquele trabalho.
- [[Invertida a fórmula, o arredondamento é meia-aberto e o meio da faixa não é a resposta]]
  — o intervalo é fechado embaixo e ABERTO em cima, e o ponto médio erra o inteiro quando a
  faixa mede exatamente 1. Com entrada inteira de faixa curta, enumerar é exato e mais barato
  que inverter.
- [[Onde existe cânone visual, use o cânone; desenhe só onde a taxonomia é sua]] — antes
  de desenhar um símbolo, pergunte se o público já o tem na cabeça; se tem, o seu cobra
  pedágio. Traz a tabela de decisão e como adaptar conjunto de terceiro ao seu tema.
- [[Mais resolução não compra qualidade em ícone; trocar de meio compra]] — dobrar a
  grade do pixel piorou o ícone três passadas seguidas; o mesmo desenho em SVG ganhou dos
  dois na primeira. Traz a tabela de escolha de meio e o teste de silhueta.
- [[Peça desenhada fora do DOM é uma segunda implementação do tema, e ela envelhece calada]]
  — cartão em canvas, imagem de OG, PDF: não leem token, não quebram, e continuam
  publicando o visual de duas viradas atrás. Traz junto as armadilhas de canvas (halo que
  vira disco, `transparent` que suja a borda, offset cravado).
- [[Regra que veio de fora do sistema entra como chave declarada, não cravada no código]]
  — "o jogo disse que X não vai valer" tem data de validade; vira chave com padrão certo e
  recorte declarado na tela, e o critério mora no campo da fonte, não numa lista de nomes.
- [[Sinal booleano da fonte não ocupa o lugar de uma escala]] — `raro: true` ligado em
  metade do catálogo não é escala mal desenhada, é outro fato no slot errado. Derive a
  escada da grandeza que a tela já mostra, em décadas, e devolva faixa NENHUMA onde a
  fonte se cala.
- [[Modal com conteúdo que cresce tem teto de altura e área que rola]] — overlay não
  pode crescer sem fim: teto no painel, scroll na parte que cresce.
- [[Primitiva de botão fecha o tamanho e abre só a variante]] — o botão vira
  componente que expõe cor, não tamanho; o `!h-7` por instância deixa de existir.
- [[Anúncio em feed não pode vestir a roupa do conteúdo]] — pode caber no layout, não
  pode passar por conteúdo: rótulo de palavra fechada, densidade mínima, sinal visual
  próprio e altura reservada.
- [[Fila de campos alinha por altura fixa de controle, não por items-end]] — todo
  controle veste a mesma casca e a célula reserva a linha do rótulo mesmo sem
  rótulo; `items-end` só disfarça altura diferente. Traz junto a armadilha de
  truncar, que esconde nos dois eixos e come o acento.
- [[Barra de topo contextual - o módulo injeta suas ferramentas via portal]] — o
  topo muda por tela: cada módulo manda busca/ações via slot, sem busca duplicada.
- [[Trocar arte por ícone de linha exige recalibrar tamanho, não só trocar o componente]]
  — glifo cheio e traço fino não vivem na mesma escala; piso de 14px.
- [[Quantos filtros existem é decisão de layout, não de produto]] — a barra horizontal
  cabe três controles e decide o escopo da tela sem ninguém notar; trilho fixo devolve
  a decisão pro produto.
- [[Faixa de cauda longa entra por número, não por slider]] — o trilho é linear em
  pixel e o dado quase nunca é; com p75 no primeiro décimo da barra, o slider não
  consegue expressar a pergunta.
- [[Medidor de razão nomeia a grandeza e mostra os operandos]] — "diferença 0.000 /
  0.150" é placar sem jogo: nomeie a grandeza, mostre os operandos e ponha o
  denominador em palavra. Reprova quando quem conhece o domínio pergunta o que é.
- [[Manual de ferramenta é resumo visível com passo a passo sob demanda]] — os passos
  num `<details>`; abrir/fechar é chrome, então o bloco continua sendo componente de
  servidor. A barra fechada anuncia o tamanho do manual, não repete a frase que o topo
  da tela já dá.
- [[Faixa de topo de ferramenta é chegada, não rótulo]] — o topo responde onde estou,
  o que isso faz e qual o tamanho disso, antes de dizer como se chama; toda a
  decoração sai de um `--tint` só, e a identidade da ferramenta vira registro que
  abastece home, navegação e topo.
- [[Ranking de opções não usa o verbo do estado ao vivo]] — lista de possibilidades e
  painel de status têm as mesmas linhas e os mesmos números; o que os separa é o verbo.
  "caçando X" em seis linhas afirma seis caçadas simultâneas onde só uma existe.
  Numere a ordem, troque o verbo, e tire o rótulo de "no ar" do ESTADO, nunca da posição.
  Princípio: [[Ver o plano e mandar executar são duas ações]].
- [[Arte de ícone se julga no tamanho de uso, e o acento é a massa]] — folha de
  contato no fundo real, silhueta antes de cor, e a fronteira entre arte de figura
  (24px pra cima) e ícone de traço no chrome miúdo.
- [[Glifo miúdo é lido como o símbolo mais próximo que a pessoa já conhece]] — em
  14px o olho fecha a figura no repertório dele: trilha pontilhada vira colcheia,
  fita com dois marcos vira osso. Pergunte "o que mais isso parece?" antes de
  "está bonito?", e cuide também da colisão interna, com os ícones do próprio set.

Princípios: [[Catálogo de componentes é contrato vivo, não documentação]] ·
[[O primitivo só padroniza o que passa por dentro dele]] ·
[[A variante de um controle muda a intenção, não o tamanho]] ·
[[Tela que abre vazia tem que ensinar, tela que abre cheia não]] ·
[[Texto de interface soa a IA pelo ritmo, não pelo assunto]]

## Estados e feedback

`02 - Técnicas/Design/Estados e feedback`

- [[Toast em vez de alert para o feedback do app]] — canal de mensagem próprio.
- [[Esqueleto de carregamento imita a forma do conteúdo]] — carregar sem saltar.
- [[Slot com placeholder esmaecido segura o lugar do dado vivo]] — tela de stream
  que não se mexe: o valor acende num slot que já existia.
- [[Filtro de lista mora na URL]] — estado que sobrevive ao reload.
- [[Consulta pesada executa por botão, não por mudança de filtro]] — rascunho e
  aplicado; o verbo do botão segue a ação real da tela.
- [[Zero num medidor é estado, não barra vazia]] — barra em 0 é igualzinha a "não
  carregou": número sempre visível, trilho de alerta, chip com a palavra e a ação
  que resolve no mesmo card.
- [[A régua de um medidor é percentil, não máximo]] — escalar pelo maior valor deixa
  99% das barras no primeiro terço; teto no p98 e marca em quem satura.
- [[Zero na tela é afirmação, não valor de conforto]] — arredondar valor minúsculo
  pra zero apaga dado, e zero herdado da fonte não pode alimentar derivação.
- [[Estimativa que inverte valor arredondado é faixa, não ponto]] — inverter a
  fórmula sobre um número já arredondado devolve intervalo; validar pelo ponto
  reprova entrada boa.
- [[Conteúdo do servidor não pode nascer invisível esperando o cliente]] — `opacity-0`
  até o `onLoad` apaga a página que o servidor já mandou pronta.
- [[Nada que está na tela pode estar invisível esperando o scroll]] — `rootMargin`
  negativo embaixo dispara mais TARDE, e limiar por fração castiga o bloco alto; junto,
  deixam o rodapé da janela baixa permanentemente apagado. Traz também o recorte: revela-se
  por unidade de significado, não por metade de composição.
- [[Pedido de apoio entra depois do valor, e nunca ao lado de si mesmo]] — o gatilho
  é uso e não chegada, o "agora não" tem prazo, e o balão some enquanto o rodapé
  está em cena; link que ainda não existe não vira botão.
- [[Reduzir movimento tem que zerar o atraso, não só a duração]] — o reset padrão de
  `prefers-reduced-motion` deixa `animation-delay` intacto, e com entrada em cascata
  isso vira tela em branco pra quem pediu menos movimento.
- [[Animação de enfeite escolhe a propriedade pelo custo, não pelo efeito]] — mesmo
  desenho, ordens de grandeza de diferença: `width` faz layout a cada quadro e
  animar caixa borrada rasteriza o desfoque em cada card da grade.
- [[Ponto decimal em interface pt-BR afirma outro número]] — `toFixed` devolve ponto,
  e em português ponto é separador de milhar: a mesma chance saía "3,4%" no
  parágrafo e "3.400%" na tabela logo abaixo. Um formatador só, desde o começo.

Princípios: [[Todo estado da tela tem visual]] · [[Nota carrega só o que a pessoa não sabe]] ·
[[Travar o valor não impede a tela de afirmar a partir dele]] ·
[[Custo de processo aleatório se orça pela cauda, não pela média]] ·
[[Peça o que a fonte mostra, não o que você precisa]] ·
[[Estado compartilhável mora na URL]] ·
[[Dado que chega preenche espaço reservado, não empurra a tela]] ·
[[A régua sai da distribuição, não dos extremos]] ·
[[A tela não afirma mais precisão do que a fonte tem]]

## Checklist de tela nova

1. Cor e espaço saem de token e de escala — nada de hex ou `17px` solto.
2. Container com largura máxima, padding constante e gap por nível.
3. Hierarquia por superfície; borda só onde a superfície não resolveu.
4. Os quatro estados desenhados: carregando, vazio, erro, sucesso.
5. Se a tela abre VAZIA (calculadora, simulador, importador), ela tem manual —
   [[Manual de ferramenta é resumo visível com passo a passo sob demanda]] — e o topo
   dela é chegada, não rótulo:
   [[Faixa de topo de ferramenta é chegada, não rótulo]].
6. O texto da tela lido em voz alta: travessão emendando, "não é X, é Y" e explicação
   que ninguém pediu são os três tiques de
   [[Texto de interface soa a IA pelo ritmo, não pelo assunto]].
7. Filtro, busca e aba na URL; preferência pessoal no `localStorage`.
8. Testado no tema claro **e** no escuro, no build de produção
   ([[Verificar no build de produção, não só em dev]]) e com `prefers-reduced-motion`
   ligado ([[Reduzir movimento tem que zerar o atraso, não só a duração]]).

## Onde uma nota nova entra

Se fala de cor ou tema → **Cor e tema**. De medida, grade ou respiro → **Layout e
espaço**. De um bloco de UI concreto → **Componentes**. Do que a tela faz enquanto ou
depois de uma ação → **Estados e feedback**.

Se não couber em nenhuma e ainda assim for verdade sem CSS, é princípio: vai pra
[[Base]].

- [[Motivo de piso tem que tocar a borda do tile]] — desenho centrado com folga vira
  poá ao ladrilhar; padrão nasce do encontro entre vizinhos. Renderize 4x4 antes de aceitar.

---

Voltar para [[Início]]
