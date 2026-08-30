---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-30
---

# Tela que manda comparar duas coisas mostra as duas

> Uma fila de moderação dizia, no cabeçalho: "compare o rosto da selfie com o do
> documento, **e os dois com as fotos do anúncio**". As fotos do anúncio não
> estavam na página. Quem modera não para para reclamar da instrução impossível:
> faz a parte que a tela permite e aprova.

## O que acontece de verdade

A metade que sobrou era selfie contra documento — a comparação fácil, e a que
ninguém tenta fraudar. A que importava (as fotos publicadas são desta pessoa?)
simplesmente não acontecia, e nada na tela denunciava isso. A fila esvaziava, os
números iam bem, e a verificação era teatro.

Instrução que a interface não deixa cumprir é pior que instrução nenhuma:
sem ela, quem modera percebe que falta algo; com ela, sai da tela achando que
fez o trabalho inteiro.

## A regra

Se a tela pede uma comparação, os dois lados aparecem **na mesma tela**, no
tamanho em que dá para comparar. Um link "abrir o perfil" não substitui: ele
resolve o caso raro em que a fila não cabe, não o trabalho de toda linha.

E o caso vazio precisa de texto próprio. Anúncio sem foto nenhuma não é "sem
fotos para mostrar" — é "não há o que comparar, e a recusa tem que pedir as
fotos antes".

## Como achar os outros

Leia os textos de instrução da sua interface e pergunte, para cada verbo, se a
tela oferece com o que fazer aquilo. "Compare", "confira", "verifique se bate
com", "veja se corresponde" — todo verbo de comparação exige dois objetos
visíveis, e é comum um deles ter ficado para trás.

## Conexões
- Princípio: [[Nota carrega só o que a pessoa não sabe]] — a irmã pelo avesso:
  lá o texto sobra, aqui ele pede o que a tela não dá. Nos dois casos o problema
  é a distância entre o que está escrito e o que existe.
- Irmã: [[Palavra da interface é lida com o dicionário do usuário, não com o seu]]
- Visto em: [[Privello]]
- Mapa: [[Design]]
