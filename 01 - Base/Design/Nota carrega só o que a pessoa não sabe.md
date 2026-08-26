---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-08-21
---

# Nota carrega só o que a pessoa não sabe

> Repetir na legenda o que o campo já diz não é ajuda: é ruído com cara de ajuda.

## A regra

Uma nota, legenda ou aviso na tela só pode dizer o que a tela **sabe e a pessoa não**:

- de onde o número saiu (a fórmula, a fonte, o recorte),
- o que ele **não** conta (a ressalva, o limite da precisão),
- o que fazer quando ele não fecha (a saída).

Não entra: valor que veio de um input (a pessoa acabou de digitar), rótulo que o campo
já tem, e instrução que a label já dá.

E a nota tem **voz própria** — no piwdex2 isso virou itálico em todo recado. Sem a
separação, uma ressalva de três linhas no meio de uma tela de números parece mais um
campo, e o olho pula ela junto com o resto.

## Por que

Legenda redundante gasta o crédito de atenção que a legenda seguinte vai precisar. Se
metade das notas repete o óbvio, a pessoa aprende a não ler nota — e a que importava
(“esse valor é estimativa”) morre junto.

Sintoma que originou a regra: um “está no 50” ao lado do campo de nível onde o próprio
usuário tinha escrito 50.

## O mesmo desconto vale para a LINHA dentro de um grupo

A regra nasceu de legenda, e vale para qualquer texto que a tela repete de si
mesma. O caso caro é a lista agrupada: se o cabeçalho do bloco diz LEDIAN, cada
linha embaixo dele não pode começar por "Ledian".

No [[piwdex2]] a ficha de patch fazia isso seis vezes seguidas — "Ledian rende
37,5 de ouro...", "Ledian: Bug Gosme de 15% pra 0,82%..." — e o custo não é o
espaço. É a POSIÇÃO: a primeira palavra da linha é onde o olho procura o que
mudou, e ela estava ocupada por um dado que a pessoa acabou de ler no cabeçalho,
idêntico em todas as linhas. Tirando o sujeito, a mesma lista passa a abrir por
"Rende", "Bug Gosme", "Straw" — que é a informação.

O corolário é que a frase precisa saber viver dos dois jeitos: fora do grupo (um
cartão de resumo, um resultado de busca) a linha viaja sozinha e o sujeito volta
a ser obrigatório. Então o sujeito é PARÂMETRO da frase, não texto colado nela.

## Regra de bolso

Leia a nota tapando o resto da tela. Se ela continuar fazendo sentido sozinha, ela
carrega informação. Se só faz sentido junto do campo, ela é eco do campo.

## Conexões
- Irmã: [[Todo estado da tela tem visual]] · [[A tela não afirma mais precisão do que a fonte tem]]
- Aplica em: [[Tela que abre vazia tem que ensinar, tela que abre cheia não]]
- Visto em: [[piwdex2]]
