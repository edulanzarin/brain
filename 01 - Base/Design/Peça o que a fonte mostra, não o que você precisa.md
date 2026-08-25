---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-08-21
---

# Peça o que a fonte mostra, não o que você precisa

> O formulário pedia IV. IV é o único número que a tela do jogo esconde — era pedir o
> resultado como se fosse a entrada.

## A regra

Antes de desenhar um campo, pergunte **de onde a pessoa vai copiar esse valor**. Se a
resposta é "de lugar nenhum, ela teria que calcular", o campo está errado: peça as
grandezas que a fonte **realmente exibe** e faça a conta você.

O erro é sutil porque o campo parece razoável — o sistema precisa daquele número, e
pedir o que se precisa é o caminho mais curto **do lado de quem constrói**. Do lado de
quem preenche, é um beco: a pessoa fecha a tela, ou pior, chuta e o resultado sai errado
com cara de certo.

Sintoma clássico: a mesma conta já existe noutra tela do próprio sistema. Se a
ferramenta A calcula X a partir de Y, a ferramenta B não pode pedir X — ela pede Y e
chama a mesma conta.

## O corolário caro: a incerteza viaja junto

Quando o valor é **derivado**, ele carrega a imprecisão da derivação, e essa imprecisão
tem que atravessar todas as telas seguintes. Se o IV do pai é "30 a 32", o IV do filho é
"30 a 32" — imprimir 31 no passo seguinte é inventar uma certeza que nunca existiu, e o
erro fica invisível justamente porque a conta intermediária pareceu limpa.

Vale também para o caso degenerado: quando **nenhum** valor de entrada explica o que foi
informado, a tela para. Travar no limite e seguir publicando número é o caminho em que o
sistema mente com cara de certeza — ver [[Zero na tela é afirmação, não valor de conforto]].

## O outro lado do mesmo erro: derivar o que a fonte já entrega

O caso do IV é pedir o que a fonte esconde. O espelho dele é **derivar o que a fonte
mostra**, e ele engana mais porque o formulário fica curto e limpo.

O Stadium do [[piwdex2]] nasceu pedindo espécie, nível e quality, e calculava os seis
stats supondo IV médio. Três campos em vez de nove, tudo aparentemente elegante. Só que a
tela do jogo mostra os seis stats, e o IV é justamente o que ela esconde: supô-lo fazia o
combate responder sobre um Charizard MÉDIO em vez do da pessoa, sem uma palavra na tela
dizendo isso. Quem usou reclamou na primeira olhada — "ele nem está pedindo os status do
meu pokémon".

Vale a pergunta invertida, então: além de "de onde a pessoa copia esse campo?", pergunte
**"o que mais a fonte mostra que eu estou calculando?"**. Campo a mais que se copia vale
mais que campo a menos que se estima.

## Regra de bolso

Abra a fonte (a tela do outro sistema, o extrato, o relatório) ao lado do seu formulário.
Todo campo tem que ter um correspondente visível ali. O que não tiver, você calcula.

## Conexões
- Aplica: [[Estimativa que inverte valor arredondado é faixa, não ponto]] ·
  [[Custo de processo aleatório se orça pela cauda, não pela média]]
- Irmã: [[A tela não afirma mais precisão do que a fonte tem]] ·
  [[Tela que abre vazia tem que ensinar, tela que abre cheia não]]
- Visto em: [[piwdex2]]
