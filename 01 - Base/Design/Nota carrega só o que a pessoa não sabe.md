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

## Regra de bolso

Leia a nota tapando o resto da tela. Se ela continuar fazendo sentido sozinha, ela
carrega informação. Se só faz sentido junto do campo, ela é eco do campo.

## Conexões
- Irmã: [[Todo estado da tela tem visual]] · [[A tela não afirma mais precisão do que a fonte tem]]
- Aplica em: [[Tela que abre vazia tem que ensinar, tela que abre cheia não]]
- Visto em: [[piwdex2]]
