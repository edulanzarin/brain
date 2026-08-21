---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-08-21
---

# A tela não afirma mais precisão do que a fonte tem

> Todo número impresso é uma afirmação, e a casa decimal é o tamanho dela. "0%" afirma
> que não acontece; "IV 17,3" afirma que se sabe até a primeira casa. Quando a fonte não
> sustenta a afirmação, quem inventou a precisão foi o formatador — e ninguém que lê a
> tela tem como desconfiar.

## A regra

Antes de escolher o formato de um número derivado, pergunte **o que a fonte realmente
sabe**:

- Se o valor real é menor que a resolução do formato, o formato está errado — aumente as
  casas ou use `<limiar`. Nunca deixe um valor existente virar zero.
- Se a fonte registra `0` num campo obrigatório, isso pode ser "não digo", não "nenhum".
  Derivar em cima disso produz `Infinity`, `NaN` ou — pior — um número plausível.
- Se o valor foi obtido **invertendo** uma observação já arredondada, o resultado é um
  INTERVALO. Publicar o ponto médio esconde que o intervalo às vezes é largo demais pra
  separar qualquer coisa.
- Quando a leitura é ampla ou impossível, isso é informação para a tela dar, não um
  detalhe a esconder atrás de um número redondo.

## Por que

O erro nunca aparece como erro. A conta roda, o número cabe na caixa, o alinhamento fica
bonito — e a tela passa a afirmar com a autoridade de um algarismo algo que ninguém
verificou. Quem lê não tem como distinguir "0%" medido de "0%" arredondado, nem "17,3"
calculado de "17,3" chutado.

Dois casos concretos, a mesma raiz:

- **Para baixo.** Uma tabela de chance de drop formatava com duas casas. Quarenta e
  quatro linhas caíam abaixo de 0,01% e viravam "0,00%" — o item que cai a 0,00001%
  passou a afirmar que não cai, e era justamente o número que a ferramenta existia pra
  mostrar. Na mesma tabela, a coluna vizinha dizia "10.000.000 abates por unidade": duas
  células se contradizendo.
- **Para o meio.** Uma calculadora estimava um atributo escondido invertendo a fórmula do
  jogo a partir do valor exibido. Como o jogo exibe arredondado, em nível baixo meia
  unidade de diferença valia 8 pontos do atributo: o mesmo número na tela era compatível
  com 4 e com 30 ao mesmo tempo. O ponto médio dizia "17,3" e a validação de faixa
  chegou a acusar de impossível um bicho perfeitamente normal.

## Na prática

A pergunta que revela o problema tem duas metades. **"Qual é o menor valor não-nulo do
conjunto, e como ele aparece na tela?"** — se aparece como zero, o formatador apaga dado.
E **"esse número foi medido ou reconstruído?"** — reconstruído a partir de observação
quantizada é intervalo, sempre.

Precisão honesta custa mais espaço na tela: `<0,0001%` é mais largo que `0%`, e `25–32`
é mais largo que `28,5`. É o preço, e é barato perto de uma tela que mente com cara de
exatidão.

## Conexões
- Irmã: [[Todo estado da tela tem visual]] · [[A régua sai da distribuição, não dos extremos]]
- Técnica que aplica: [[Zero na tela é afirmação, não valor de conforto]] ·
  [[Estimativa que inverte valor arredondado é faixa, não ponto]]
- Mapa: [[Base]]
