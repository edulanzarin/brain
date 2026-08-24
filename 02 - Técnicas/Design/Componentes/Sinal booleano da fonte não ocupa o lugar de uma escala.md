---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-24
---

# Sinal booleano da fonte não ocupa o lugar de uma escala

> Quando a fonte entrega `raro: true/false` e a tela precisa de uma escada, o
> caminho não é desenhar melhor o booleano. É **derivar a escada** de uma grandeza
> que a tela já mostra — e devolver *nenhuma faixa* onde a fonte não tem o que
> responder.

## Como se percebe que o interruptor não gradua

Antes de trocar, meça. O teste é simples: cruze o booleano com a grandeza que ele
deveria estar aproximando.

Num catálogo de 428 itens, `rare` estava ligado em 206 — quase metade. Cruzado
com a dificuldade real de conseguir uma unidade, **31 dos 85 itens mais fáceis do
jogo** carregavam o selo. Ele não ordena, não gradua e não concorda com o dado
mostrado ao lado dele. Um selo assim não é uma escala mal desenhada: é um fato de
outra natureza ocupando o slot errado.

## Derive do número que a tela já mostra

A tentação é inventar um eixo novo. O melhor eixo costuma já estar na tela: é
aquele que o card exibe logo abaixo do nome.

No caso, a página inteira girava em torno de "quantos abates custa uma unidade".
A escada saiu daí, em **ordens de grandeza** — cada degrau é uma década de
abates:

```
menos de 1     cai em quase todo abate       COMUM
1 a 10         alguns abates                 INCOMUM
10 a 100       dezenas de abates             RARO
100 a 1.000    centenas de abates            ÉPICO
1.000 a 10.000 milhares de abates            LENDÁRIO
10.000+        dezenas de milhares           MÍTICO
```

Deu 85 / 90 / 47 / 38 / 10 / 18. Uma cauda longa, que é a forma que uma raridade
tem que ter — a distribuição em si é parte da verificação.

**Década, e não percentil.** Percentil é a régua certa quando o eixo não tem
significado próprio (poder de golpe, soma de stat); "mil abates" tem. Além disso,
corte por percentil faz a faixa mudar a cada patch da fonte sem nada ter mudado
pra quem usa.

## Sem resposta na fonte, sem degrau

Um terço do catálogo não tinha chance publicada — vem de altar, evento, ou de
algo fora do catálogo público. Esses ficam **sem faixa**, e a tela diz por quê. As
duas alternativas eram piores: herdar o degrau mais baixo afirma "isto é comum"
sobre coisas que ninguém consegue, e deixar o espaço em branco faz a peça parecer
quebrada.

O filtro segue a mesma regra: item sem faixa não passa em filtro de faixa
nenhuma. `null` não é o primeiro degrau — é a ausência dele.

## E o booleano, o que acontece com ele

Não some: ele continua sendo um fato da fonte. Só sai da grade e da tabela, onde
competia com a escada e perdia, e volta na ficha **dizendo de quem é a
afirmação** — "o jogo marca como raro", não "raro". Fato onde cabe fato; veredito
onde cabe veredito.

## Conexões
- Princípio: [[A tela não afirma mais precisão do que a fonte tem]]
- Irmã: [[A mesma grandeza usa a mesma escada nas duas telas]] ·
  [[Limiar em grandeza contínua vira degrau, e o degrau decide a ordem]] ·
  [[Zero na tela é afirmação, não valor de conforto]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
