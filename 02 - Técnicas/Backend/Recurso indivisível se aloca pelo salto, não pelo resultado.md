---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-25
---

# Recurso indivisível se aloca pelo salto, não pelo resultado

> Quando você tem UM item pra dar a um de vários candidatos, a lista tem de ordenar
> pelo quanto ele MUDA cada um — não pelo estado em que cada um fica depois. Ordenar
> pelo resultado final entrega o item a quem menos precisava dele, e cada número da
> tela continua certo.

## O problema

A lista ordenada pelo valor final tem uma força de convencimento que a certa não
tem: o primeiro colocado é, de fato, o mais forte depois de receber o item. Parece a
resposta. Mas a pergunta era outra — "onde este item rende mais?" — e as duas listas
divergem sistematicamente, porque **quem termina no topo costuma ser quem já estava
perto dele**.

No [[piwdex2]], o disco de TM. Todo golpe de TM do jogo rende 60 de poder por
segundo e o melhor golpe natural das 482 espécies rende 43,3, então o disco é o maior
salto de poder disponível — e o Researcher entrega **um** por troca:

| Espécie | Sem disco | Com disco | Salto |
|---|---|---|---|
| Scizor | 15.145 | 26.811 | **1,77x** |
| Jolteon | 1.495 | 11.395 | **7,62x** |

Ordenado pelo resultado, o Scizor lidera e o conselho é "põe no Scizor". Ele é o que
menos precisa: o moveset natural dele já era o melhor da lista, e o disco só engrossa
o que já existia. O Jolteon tinha um moveset natural fraquíssimo, e o mesmo disco o
faz **subir de faixa** na tier list — de B pra S. Com um item na mão, transformar vale
mais que somar.

## A solução

Calcule os dois estados e ordene pela **razão** (ou pela diferença, quando a grandeza
é aditiva). E deixe as duas colunas na tela, porque as duas perguntas existem:

```
antes  = medida(candidato, sem o item)
depois = medida(candidato, com o item)
salto  = depois / antes        <- ORDENA por isto
```

- **Salto** responde "onde gasto o item que tenho".
- **Resultado** responde "e depois de tudo, quem é o melhor?" — pergunta de quem já
  tem o item em todo mundo, que é outro momento.

## O que mais vale lembrar

- **A razão costuma ser invariante a escala, e isso é um presente.** No caso do TM,
  os dois lados usam o mesmo stat multiplicado pelo mesmo fator de nível e quality,
  então ele cancela: o salto **não muda com o nível**. Uma ordem que não depende do
  estado de quem pergunta pode ser calculada uma vez e vale pra todo mundo. Vale
  procurar essa propriedade antes de assumir que a conta é por indivíduo.
- **Divisão por quase-zero mente.** Candidato sem nada antes tem salto infinito, e ele
  vai encabeçar a lista dizendo pouco. Trate o caso à parte, com palavra em vez de
  número ("passa a ter").
- **Quando o custo é igual em todas as opções, o salto é a decisão inteira.** Ver
  [[Ordene pela grandeza que decide, não pela que impressiona]], seção da grandeza que
  não varia — foi o que aconteceu duas vezes no mesmo jogo, nas trocas do Eevee e nos
  discos de TM.
- **Diferença ou razão?** Razão quando a grandeza tem zero natural e escala
  multiplicativa (dano, receita); diferença quando ela tem teto ou piso que importa
  (ver a irmã, que trata o caso do teto).

## Conexões
- Princípio: [[Ordene pela grandeza que decide, não pela que impressiona]]
- Irmã: [[Bônus multiplicativo só rende onde há folga até o teto]] (mesmo defeito por
  outra causa: lá é o teto que come o ganho, aqui é a escassez do item) ·
  [[Tier é nota com régua fixa, não posição na fila]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
