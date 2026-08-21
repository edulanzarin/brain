---
tags: [tipo/atomica, camada/principio, design, dados]
criado: 2026-08-21
---

# Custo de processo aleatório se orça pela cauda, não pela média

> "Em média são 53 tentativas" é o número que faz alguém começar com dinheiro pra 53
> e parar na 58. Média é o centro da distribuição, não a promessa dela.

## A regra

Quando a tela estima **quantas tentativas** ou **quanto custa** um processo com resultado
sorteado, ela mostra **três números**, nunca um:

- **melhor caso** — o piso, se tudo vier no máximo. Dá a escala do otimismo.
- **típico (mediana)** — onde metade das tentativas fecha. É a manchete.
- **azarado (p90)** — o orçamento. É o número pelo qual a pessoa decide se começa.

O melhor caso sozinho é propaganda; a média sozinha é uma armadilha, porque em processos
de soma-até-um-alvo ela fica perto da mediana e some com a cauda — que é justamente onde
o dinheiro acaba.

## Por que

O custo do erro é **assimétrico**. Terminar antes do previsto não devolve nada; ficar sem
recurso no meio de uma corrente irreversível perde tudo que já foi gasto. Uma estimativa
por ponto trata os dois lados como se doessem igual.

É a mesma família de [[Estimativa que inverte valor arredondado é faixa, não ponto]] e de
[[A tela não afirma mais precisão do que a fonte tem]]: quando a fonte é uma distribuição,
imprimir um ponto é inventar exatidão que não existe.

## E declare a suposição escondida

Toda conta dessas assume alguma coisa que a tela não simula — que sempre há insumo, que
as tentativas são independentes, que nada expira no meio. Essa frase vai na tela junto
do número, não na documentação: é a diferença entre uma estimativa e uma estimativa
utilizável.

## Regra de bolso

Se o número foi calculado dividindo alvo por média, ele não está pronto. Pergunte "e se
der azar?" — se a tela não responde, ela está orçando o caminho que quase ninguém pega.

## Conexões
- Aplica: [[Distribuição exata sai de programação dinâmica, não de Monte Carlo]]
- Irmã: [[Estimativa que inverte valor arredondado é faixa, não ponto]] ·
  [[A tela não afirma mais precisão do que a fonte tem]] ·
  [[A régua sai da distribuição, não dos extremos]]
- Visto em: [[piwdex2]]
