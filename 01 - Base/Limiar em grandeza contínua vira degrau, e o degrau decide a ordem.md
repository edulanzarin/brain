---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-22
---

# Limiar em grandeza contínua vira degrau, e o degrau decide a ordem

> Toda vez que um `if x >= K` responde um número em vez de um rótulo, nasce um degrau.
> Se esse número depois multiplica outros, o degrau para de ser detalhe de modelagem e
> passa a ser **quem decide a ordenação** — e ninguém vê, porque cada valor isolado
> parece plausível.

## A regra

Quando uma grandeza contínua entra e um **número** sai, a saída também tem que ser
contínua. O limiar é legítimo pra escolher **palavra** (seguro / arrisc**a**do / letal);
é quase sempre errado pra escolher **fator**.

O teste é mecânico e não depende do domínio:

- Varra a entrada em passos minúsculos e meça a maior razão entre valores vizinhos.
- Se dois pontos separados por 0,1% da entrada devolvem saídas com razão de 2x, o
  modelo tem degrau — independentemente de o número estar "certo" nas duas pontas.

Corrigir não é apagar o limiar: é interpolar entre os limiares que já existem. Uma rampa
linear tira o salto e deixa um joelho em cada extremo; um `smoothstep` (`t²(3−2t)`) zera
também a derivada nas pontas, então não sobra nem degrau nem quina.

**Interpole preservando as pontas.** Se o valor no topo da faixa era exatamente 1 e a
calibração absoluta do sistema foi feita em cima disso, a curva nova tem que continuar
chegando a 1 ali — senão a correção de ordenação vira recalibração de tudo, e um
conserto vira dois problemas.

## Por que

Um degrau não dá sintoma. Não há exceção, não há `NaN`, e cada tela isolada mostra um
número que passa em qualquer revisão. O que quebra é a **comparação**, que costuma ser
o produto real da ferramenta.

Caso concreto: um motor de rendimento estimava a fatia do tempo em que o jogador fica
de pé com `killsPerLife >= 6 ? 1 : vida/(vida + custoDaMorte)`. Como essa fatia
multiplica os três números que a tela mostra, dois alvos com 5,999 e 6,001 abates por
vida — 0,08% de diferença em sobrevivência — apareciam com **2,5x** de diferença em
XP/h, ouro/h e abates/h. No catálogo, 299.928 combinações caíam na vizinhança do 6, e em
642 delas o alvo **vencedor** mudava só pelo lado do limiar em que caiu.

O detalhe que fecha o argumento: o próprio arquivo afirmava, num comentário, que "a
ordenação é robusta a estas constantes". Era falso, e ninguém tinha como perceber lendo o
código — só medindo a razão entre vizinhos.

## Na prática

A margem de histerese que costuma existir rio abaixo ("só troca de recomendação se a
nova ganhar por 8%") **não salva**: ela foi dimensionada pra ruído, não pra um salto de
150%. Quando há degrau, a margem só desloca onde ele aparece.

## Conexões
- Irmã: [[A régua sai da distribuição, não dos extremos]] ·
  [[Custo de processo aleatório se orça pela cauda, não pela média]]
- Visto em: [[piwdex2]]
- Mapa: [[Base]]
