---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-08-22
---

# Animação de enfeite escolhe a propriedade pelo custo, não pelo efeito

> Duas propriedades produzem o mesmo desenho e custam ordens de grandeza diferentes.
> Como o resultado na tela é idêntico, a escolha some na revisão — e o preço aparece
> justamente onde a máquina já está apertada.

## A regra

`transform` e `opacity` andam na **composição**: a GPU reusa a textura já pronta.
Qualquer outra propriedade animada cobra mais caro:

| Anima | Custa | Troque por |
|---|---|---|
| `width` / `height` | layout a cada quadro | `transform: scaleX/scale` |
| `top` / `left` | layout | `transform: translate` |
| `filter` / `blur` | rasterização a cada quadro | `transform` sobre a caixa já borrada |
| `box-shadow` | pintura | opacidade de uma camada irmã |

Para trocar `width` por `scaleX` sem mudar o desenho: fixe a caixa no tamanho **maior**,
ponha `transform-origin` no lado certo e anime a escala **para baixo** no estado de
repouso. O quadro final é o mesmo; o custo muda de lugar.

## Dois lugares onde isso dói de verdade

**Barra indeterminada de carregamento.** Ela anima em laço infinito e roda exatamente
quando a máquina está montando a página — animar `width` ali é fazer layout a 60fps
concorrendo com o trabalho que a pessoa está esperando.

**Halo de card em grade.** Um brilho `blur` que cresce no hover parece barato porque é
um elemento só. Mas a grade tem dezenas deles: animar `width/height` de algo borrado
obriga o navegador a **recalcular o desfoque** a cada quadro, em cada card. É a animação
mais cara da página, e é enfeite de hover.

## Sinal de alerta

`transition-all` quase nunca é o que se quer: ele promete animar propriedades que ainda
nem existem no elemento e esconde qual é a cara. Liste as propriedades.

## Movimento reduzido

Quem indica **progresso** continua se mexendo com `prefers-reduced-motion`, sem
deslocamento — congelar o indicador junto com o resto deixa a tela com cara de travada,
e "travou" é informação pior que o movimento que a pessoa pediu pra evitar. O esqueleto
de carregamento entra nessa exceção; `float` e `glow` de enfeite, não.

## Conexões
- Princípio: [[Todo estado da tela tem visual]]
- Irmã: [[Esqueleto de carregamento imita a forma do conteúdo]] ·
  [[Dado que chega preenche espaço reservado, não empurra a tela]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
