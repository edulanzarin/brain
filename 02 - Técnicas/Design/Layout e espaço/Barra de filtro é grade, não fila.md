---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-09-02
---

# Barra de filtro é grade, não fila

> Numa fila `flex-wrap`, controle que nasce empurra o vizinho: escolher "período
> personalizado" abre dois campos de data, a filial pula de linha, e quem já
> estava mirando nela clica noutra coisa.

## O problema

A barra de filtro parece o caso perfeito para `flex flex-wrap`: itens de
larguras diferentes que quebram sozinhos. E funciona — enquanto o conjunto de
controles é fixo.

Ele nunca é. Filtro nasce e morre com a escolha anterior: o intervalo livre só
aparece no período personalizado, a filial só faz sentido com uma empresa, o
campo de senha só com PDF. Cada aparição reorganiza a barra inteira, e a
reorganização acontece **debaixo do cursor de quem já decidiu onde clicar**.

O segundo sintoma é mais silencioso: a fila termina onde o conteúdo acabou. Uma
barra que ocupa dois terços da largura deixa um vão que não é respiro, é sobra —
e ao lado dela cada controle tem uma largura decidida à mão (`w-72` aqui, `w-56`
ali), sem ninguém para arbitrar.

## A solução

Uma grade de trilha fixa. Cada controle é uma célula; controle que aparece ocupa
uma célula vazia em vez de empurrar a fila.

```css
grid-template-columns: repeat(auto-fill, minmax(13rem, 1fr));
```

`auto-fill` e **não** `auto-fit`: com `auto-fit` a última trilha estica para
comer a sobra, e o mesmo campo aparece com uma largura diferente em cada tela,
conforme quantos filtros existem.

Duas consequências que vêm junto e melhoram o resto:

- **O controle deixa de carregar largura própria.** Quem decide é a célula, e
  não cada tela por conta. Some a coleção de `w-72`/`w-56`/`w-40` espalhada.
- **Um componente que devolve vários campos devolve FRAGMENTO**, não um
  container. Agrupados num `div`, os três campos do seletor de período
  disputariam uma célula só, e o empurrão voltaria pela porta dos fundos.

Os botões de ação saem da fila e vão para uma linha própria, à direita: no
`flex` eles dependiam de `ml-auto` e mudavam de lugar conforme o número de
filtros.

## O que mais vale lembrar

- **Todo controle veste a mesma casca, com a linha do rótulo reservada.**
  Alinhar por `items-end` disfarça alturas diferentes e desaba no dia em que um
  campo do meio exibe erro ou texto de ajuda. O que não é campo (um segmentado,
  um botão que abre modal de escolha) entra na mesma casca para alinhar.
- **A grade também devolve a decisão ao produto.** Numa fila, é a largura da
  janela que decide quantos recortes a tela oferece — e ninguém escreveu isso
  em lugar nenhum.

## Conexões
- Princípio: [[Container tem largura máxima e respiro constante]]
- Irmã: [[Dado que chega preenche espaço reservado, não empurra a tela]] ·
  [[Peça de grade mostrada sozinha vai centrada num palco]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
