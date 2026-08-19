---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-19
---

# Bônus multiplicativo só rende onde há folga até o teto

> Quando um bônus multiplica uma grandeza que satura, o que ele rende não é
> proporcional ao tamanho da grandeza — é proporcional à **distância até o teto**.
> Ranquear alvos pelo valor bruto recomenda justamente onde o bônus se perde.

## O problema

Um bônus percentual parece render igual em todo lugar: +40% é +40%. Mas se ele
multiplica um valor **limitado** — uma probabilidade, uma barra que enche, uma cota —
o excedente que passa do teto é descartado em silêncio. O alvo mais gordo costuma ser
o mais saturado, então a lista ordenada por "quem rende mais" aponta exatamente o pior
lugar pra gastar o bônus.

Não há erro visível: o número bruto está certo, o multiplicador está certo, e o
resultado ainda sobe um pouco. Só o **ganho marginal** denuncia.

## A solução

Calcular o resultado duas vezes — sem bônus e com bônus, aplicando o teto nas duas — e
ordenar pela diferença. Melhor ainda: exibir a fração do bônus que o alvo consegue
converter, porque é ela que responde "vale a pena aqui?".

```
efetivo(v)     = min(TETO, v * mult)
base           = Σ efetivo(vᵢ, 1)
comBonus       = Σ efetivo(vᵢ, mult)
ganho          = comBonus - base
aproveitamento = ganho / (base * (mult - 1))   // 1 = captura tudo; 0,2 = perdeu 4/5
```

O denominador é o ganho **teórico**, o que o bônus renderia se nada saturasse. A razão
entre real e teórico é adimensional, então compara alvos de escalas diferentes sem
normalização extra.

## O que mais vale lembrar

- **O teto costuma estar escondido nos dados, não na documentação.** No piwdex ele
  apareceu como um valor repetido: 157 das 2.657 entradas de loot estavam em exatamente
  95.000 de 100.000. Constante repetida em massa é quase sempre um limite, não um acaso
  — vale contar a distribuição antes de confiar na fórmula.
- **A conclusão inverte a intuição.** O spot mais rico (Exploud, 3.914 de ouro por
  abate) aproveitava só 17% de um bônus de +41%, enquanto um alvo de metade do valor
  (Aggron) rendia +587 por abate com 45%, e um de nível baixo (Bastiodon) convertia
  quase 100%. Quem ordena pelo bruto queima o bônus no lugar errado.
- **Bônus que somam entre si, saturam juntos.** Quando várias fontes viram um
  multiplicador único, subir uma fonte reduz a folga das outras: o segundo bônus vale
  menos que o primeiro. O payback de "mais um ponto" tem que ser medido **sobre o
  cenário atual**, não sobre o cenário limpo.
- Vale fora de jogo: desconto sobre preço com piso, retry sobre uma cota já no limite,
  otimizar taxa de acerto de cache que já está em 98%, verba de anúncio num público
  saturado. Em todos, a pergunta útil é quanta folga sobrou.

## Conexões
- Princípio: [[Rendimento é vazão vezes tempo em pé, não vazão de pico]]
- Irmã: [[Num confronto, medir só o seu lado recomenda o alvo que te destrói]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
