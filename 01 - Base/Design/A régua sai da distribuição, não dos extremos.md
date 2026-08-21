---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-08-21
---

# A régua sai da distribuição, não dos extremos

> Sempre que um valor vira comprimento, posição ou porcentagem de trilho, alguém escolhe
> uma régua. O reflexo é ancorá-la no mínimo e no máximo do conjunto, porque é o único
> par que garante que nada estoura. Mas a régua não existe pra caber — existe pra
> **separar**. Quem decide onde ela começa e termina é a distribuição do dado.

## A regra

Antes de mapear dado em pixel, olhe a **forma** do conjunto, não os seus limites:

- Ordene os valores e olhe a mediana, o p75 e o p95.
- Se o p75 cai no primeiro décimo da régua, a régua está errada — mesmo que o desenho
  esteja tecnicamente correto e nada estoure.
- Ancore no percentil (p95–p98), e **marque quem satura** em vez de reescrever a escala
  de todo mundo por causa de um outlier.
- Quando a cauda é longa demais pra qualquer trilho (ordens de grandeza), o certo não é
  outra régua: é **outro controle**.

Vale nos dois sentidos do fluxo. Na **saída** — barra, sparkline, tamanho de bolha,
intensidade de heatmap. E na **entrada** — slider de faixa, zoom de timeline, seletor de
intervalo.

## Por que

O erro de régua não dá sintoma. Nada quebra, nada avisa, o componente responde e anima.
Ele só para de informar — e informar era a única coisa que ele fazia.

Dois casos concretos, o mesmo erro nas duas pontas:

- **Saída.** Uma espinha de seis barras num card de catálogo, com teto no máximo (255).
  Mediana 65, p98 130, só 0,7% acima de 150: 99,3% das barras viviam abaixo da metade da
  altura e a diferença entre 45 e 65 — a única coisa que a espinha existia pra mostrar —
  virava dois pixels.
- **Entrada.** Um slider de preço de 1 a 1.000.000, com mediana 40 e p75 em 180: três
  quartos do catálogo espremidos nos primeiros 0,018% do trilho. Cada pixel arrastado
  pulava centenas de itens na região onde todos estavam.

Nos dois casos a tela parecia pronta. O que denunciou foi a pergunta sobre a
distribuição, não a inspeção do desenho.

## Na prática

A régua se calcula **uma vez, sobre o universo inteiro**, junto com as outras derivações
— nunca sobre a página filtrada. Régua que muda com o filtro faz o mesmo item desenhar
diferente em duas telas, e aí a comparação, que é o produto do gráfico, passa a mentir.

Outlier legítimo não se esconde: se marca como outlier.

## Conexões
- Irmã: [[Todo estado da tela tem visual]] · [[Escala fechada em vez de valor solto]]
- Técnica que aplica: [[A régua de um medidor é percentil, não máximo]] ·
  [[Faixa de cauda longa entra por número, não por slider]]
- Mapa: [[Base]]
