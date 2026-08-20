---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-08-18
---

# Trocar a fonte muda a largura, não só o desenho da letra

> Fonte não é skin. Trocar a família muda **quanto espaço o mesmo texto ocupa** —
> e todo slot de largura fixa foi calibrado para a fonte antiga.

## O quê

Ao trocar a família tipográfica, três coisas mudam junto e precisam de uma passada
deliberada:

1. **Largura do texto.** Uma condensada (Jersey 10, Oswald) escreve o mesmo rótulo
   em muito menos espaço que uma de proporção normal (Quantico, Inter). Todo
   `w-20`, `min-w-[3.6rem]`, `max-w-[8rem]` e coluna de tabela vira candidato a
   truncar. Reavalie contra o **pior caso real**: nome longo, número de 7 dígitos,
   o rótulo traduzido para a língua mais verbosa.
2. **A base em `rem`.** Uma condensada costuma exigir subir a base (ex.: `112.5%`)
   para render no tamanho certo. Ao voltar para uma fonte normal, a base volta para
   16px — e **tudo** em `rem` encolhe junto: padding, gap, altura de controle,
   `min-h` de card. O texto melhora e o layout aperta.
3. **Os pesos disponíveis.** Fonte de peso único força hierarquia por cor e tamanho.
   Fonte com pesos reais destrava `font-bold` — mas só os pesos que ela **tem**.
   Pedir 600 de uma fonte que só tem 400/700 não falha: o navegador casa o mais
   próximo (500→400, 600→700). O código passa a mentir sobre o que a tela mostra.
   Normalize as classes para os pesos que existem.
4. **A ALTURA do bloco de texto.** Linha mais alta e rótulo que quebra em duas
   engordam o card inteiro. Onde isso mata é o layout **posicionado à mão**: card
   colocado em `left/top` de um container de altura fixa foi calibrado para a altura
   antiga do conteúdo e não tem como avisar que cresceu — os cards simplesmente
   passam a se sobrepor. Layout que se auto-mede (grid, flex) absorve a mudança
   sozinho; posição absoluta com altura fixa é dívida esperando a próxima fonte.

## Por que importa

No [[piwdex]] a troca da pixel condensada pela quadrada de proporção normal
compilou de primeira e passou no build — o estrago era invisível para o compilador:
slots truncando texto, cards apertados e `font-semibold` renderizando como bold sem
ninguém ter pedido. A regra que sobrou: **trocar fonte agenda uma auditoria de
largura**, não só um `git diff` no arquivo de layout.

E o estrago continuou aparecendo semanas depois, no canto que ninguém revisou: a
estrela de evolução da Eevee punha cada card em `left/top` de um container de 760px
fixos. Com os cards mais altos, o Espeon passou a **cobrir** a Eevee do centro. O
conserto não foi recalibrar as coordenadas — foi trocar por um grid 3x3, que não
pode colidir em fonte nenhuma. Coordenada à mão precisa ser recalibrada a cada
troca; layout que se auto-mede se conserta uma vez.

## Conexões
- Princípio: [[Escala fechada em vez de valor solto]]
- Irmã: [[Trocar arte por ícone de linha exige recalibrar tamanho, não só trocar o componente]] · [[Dado que chega preenche espaço reservado, não empurra a tela]]
- Visto em: [[piwdex]]
- Mapa: [[Design]]
