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
- [[Validar paleta de gráficos antes de escolher cores]] — forma primeiro, cor por
  último; checagens objetivas antes de fechar a paleta.
- [[Cor de marca precisa de variante acessível por tema]] — a cor crua do cliente
  reprova contraste como texto; derive uma variante por tema e compute.
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
- [[Alvo de toque pergunta pelo apontador, não pela largura da janela]] — `pointer:
  coarse` e não breakpoint: janela estreita no desktop continua com mouse. Traz os
  dois pisos (24 da WCAG, 44 da Apple) e como medir a área sensível de verdade.
- [[Área de toque cresce por pseudo-elemento, não pela caixa]] — link dentro de
  frase não pode engordar sem empurrar o parágrafo; cresce a área, não o elemento.
- [[Sticky gruda no container que rola, não na janela]] — `overflow-x-auto` promove o
  outro eixo pra `auto` e vira o ancestral rolável; o `top` passa a medir de dentro.

Princípios: [[Escala fechada em vez de valor solto]] ·
[[Container tem largura máxima e respiro constante]]

## Componentes

`02 - Técnicas/Design/Componentes` — índice: [[Padrões de componentes de dashboard]]

- [[Sidebar em acordeão e layout de módulo]] — a estrutura fixa da tela.
- [[Controles de filtro do dashboard]] — toggle segmentado e dropdown de filtro.
- [[Seletor cria e gerencia os próprios itens]] — combobox que cria ao digitar e
  renomeia/exclui no painel, dispensando tela de CRUD do auxiliar.
- [[Blocos de dado - card, KPI e gráfico]] — card, stat tile, gráfico e tabela.
- [[Chip que serve a duas grandezas declara qual delas mostra]] — dois eixos que
  compartilham vocabulário (espécie x indivíduo, previsto x realizado) precisam do
  eixo declarado no componente, senão a mesma entidade se contradiz entre telas.
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
- [[Arte de ícone se julga no tamanho de uso, e o acento é a massa]] — folha de
  contato no fundo real, silhueta antes de cor, e a fronteira entre arte de figura
  (24px pra cima) e ícone de traço no chrome miúdo.

Princípios: [[O primitivo só padroniza o que passa por dentro dele]] ·
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

---

Voltar para [[Início]]
