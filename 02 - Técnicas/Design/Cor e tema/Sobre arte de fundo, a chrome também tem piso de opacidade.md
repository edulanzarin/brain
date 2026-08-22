---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-08-22
---

# Sobre arte de fundo, a chrome também tem piso de opacidade

> [[Vidro flutuante precisa de superfície mais opaca que a chrome]] libera a chrome
> pra ser arejada porque "o que vaza atrás dela é o fundo da página, não conteúdo".
> Quando o fundo é **arte** — foto, wallpaper, pixel art —, essa premissa cai: o
> fundo tem contraste e variância de conteúdo, e a chrome herda o mesmo piso.

## O que se vê

A queixa chega como "está muito transparente o fundo". Ela não é sobre o wallpaper:
é sobre o neon do wallpaper chegando **atrás do número digitado**, dentro do painel.
Vidro que deixa ver a cena inteira não é superfície, é furo.

## A conta é multiplicativa, e nenhum alpha isolado denuncia isso

O erro é olhar o alpha do painel e achar 62% razoável. Sobre arte existem camadas
empilhadas, e o que sobra da arte é o **produto** do que cada uma deixa passar:

```
scrim 46% + painel 62% + campo 66%

arte dentro do painel:  (1-0,46) × (1-0,62)          = 20,5%
arte dentro do campo:   (1-0,46) × (1-0,62) × (1-0,66) =  7,0%
```

Um quinto do brilho da arte pousando dentro do painel. Num trecho de neon saturado,
isso é mais que suficiente pra brigar com o texto. Fechando as três:

```
scrim 70% + painel 90% + campo 92%

arte dentro do painel:  0,30 × 0,10 = 3,0%
arte dentro do campo:   0,30 × 0,10 × 0,08 = 0,24%
```

De 20,5% pra 3%. A arte continua visível **entre** os painéis, que é onde ela tem
função.

## O que faz o vidro parecer vidro não é o alpha

Esta é a parte que destrava a decisão. São três marcas, e nenhuma delas é
transparência:

1. `backdrop-filter: blur() saturate()` — o desfoque na borda;
2. a **aresta de luz** no topo (`inset 0 1px 0 rgb(255 255 255 / .07)`), que é o que
   dá impressão de espessura;
3. a sombra projetada, que faz o painel flutuar em vez de estar colado.

Com as três, o alpha pode ir a 90% e a superfície **continua lendo como vidro**. Não
se está trocando vidro por sólido; está-se trocando vidro de vitrine por vidro fosco.

## Subiu o alpha, desça o blur

Borrar 18px atrás de uma superfície 90% opaca é composição que ninguém vê. Cai pra
~12px no painel e ~8px no card de lista — e o card de lista é onde isso paga, porque
são 60 deles na tela ao mesmo tempo.

## O que a conta não pega

Superfície que fica **fora** de qualquer painel não herda os 90% de ninguém: rodapé,
faixa de anúncio, barra fixa do topo. Elas encostam direto na arte e precisam do
próprio alpha alto. É onde a auditoria por classe (`bg-surface/60` e afins) encontra
o que sobrou — as de dentro de painel podem continuar translúcidas, porque ali elas
são degrau de profundidade, não janela.

## Conexões
- Princípios: [[Hierarquia por superfície, não por borda]] ·
  [[Token semântico em vez de valor literal]]
- Irmã: [[Vidro flutuante precisa de superfície mais opaca que a chrome]] ·
  [[Sistema de cores e tema do dashboard]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
