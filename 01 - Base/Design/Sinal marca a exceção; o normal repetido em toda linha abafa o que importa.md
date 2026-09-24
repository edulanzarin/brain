---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-09-24
---

# Sinal marca a exceção; o normal repetido em toda linha abafa o que importa

> Selo, cor e ícone existem para o olho achar alguma coisa numa lista. Quando o
> estado NORMAL também ganha selo, toda linha fala alto ao mesmo tempo, e a única
> que pedia atenção some no meio das outras.

## O que aconteceu

Numa lista de vendas, 20 das 27 linhas levavam o selo verde "Pago". O selo cinza
"Expirado" das outras sete passava despercebido, e a lista inteira parecia um mural
de etiquetas. Na lista de assinantes, cada ativo levava um selo violeta "Até 2 out"
e a meta "No grupo"; o âmbar de quem vencia em três dias era um selo a mais numa
coluna de selos.

A correção foi tirar o sinal do normal, não enfeitar a exceção:

- venda paga fica só com o valor em verde, sem selo; expirada ganha o valor riscado
  e o selo;
- ativo com prazo longe vira texto corrido ("Até 2 out"), e o selo fica para
  "Vence em 3 dias";
- "No grupo" some; o que aparece é "Ainda não entrou no grupo", que é a exceção.

A lista ficou mais calma e, ao mesmo tempo, o que importa passou a saltar sozinho.

## A regra

Antes de dar sinal a um estado, pergunte **qual é o estado da maioria das linhas
nesta tela**. Esse é o fundo, e o fundo não se anuncia. Sinal vai só no que foge
dele.

- O "normal" depende do recorte: na aba "Expiradas", todas expiraram, e o selo em
  cada linha volta a ser ruído. O filtro já disse.
- Estado normal ainda pode aparecer, mas em voz de texto (cor de apoio, sem
  contorno), e não na mesma voz da exceção.
- Vale igual para peso visual: [[Grade de iguais esconde o único item que funciona]]
  é o mesmo erro com cartões em vez de selos.

## Por que é princípio

Apareceu em contextos diferentes: numa grade de módulos em que tudo tinha o mesmo
peso ([[Grade de iguais esconde o único item que funciona]]), numa nota que repetia o
que a pessoa acabou de digitar ([[Nota carrega só o que a pessoa não sabe]]) e nos
selos de lista do telebot. Nos três, o sinal que carrega informação é o que difere do
resto; o que se repete em toda parte vira papel de parede.

## Conexões
- Irmã: [[Nota carrega só o que a pessoa não sabe]] · [[Hierarquia por superfície, não por borda]]
- Aplica em: [[Grade de iguais esconde o único item que funciona]] ·
  [[Fato vai em selo, estado vivo vai no retrato]]
- Visto em: [[telebot]] · [[Navetech Hub]]
- Mapa: [[Design]] · [[Base]]
