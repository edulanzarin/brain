---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-09-03
---

# Eixo de folha de sprite se confere sprite a sprite, não se supõe

> A convenção "linha é direção, coluna é quadro" é a mais comum e estava errada no
> pack usado. O erro não quebra nada: o personagem só anda virado pro lado errado.

## A armadilha

Folha de 64×64 com 4 direções e 4 quadros é **ambígua**: os dois eixos têm 4. Chutar
qual é qual dá 50% de chance, e as duas versões renderizam sem erro — a diferença
aparece só na hora de andar, e um sprite de 16 px de costas é parecido o bastante com
um de frente para passar numa olhada rápida.

No Ninja Adventure Asset Pack: **coluna é a direção, linha é o quadro.**

## Como confirmar sem chutar

Não é a diferença numérica entre linhas e colunas — medi e deu 24454 contra 21083, o
que não decide nada. O que decide é uma **marca assimétrica** do desenho:

| Coluna | Pixel claro (olho) à esquerda | à direita | Logo |
|---|---|---|---|
| 0 | sim | sim | de frente = baixo |
| 1 | não | não | de costas = cima |
| 2 | sim | não | esquerda |
| 3 | não | sim | direita |

Conte o pixel do olho nos dois lados da metade de cima da célula. Duas linhas de
script respondem o que uma hora de suposição não responde.

## E depois gravar o achado onde o código lê

Confirmado o eixo, ele vai pro **manifesto gerado** (`direcoes: ["baixo", "cima",
"esquerda", "direita"]`, `quadros: 4`), não pra um comentário. Comentário não impede
o próximo a recalcular à mão; manifesto que o componente lê, sim.

## Vale para qualquer folha de terceiro

Mesmo raciocínio para autotile: descobri que o tileset de terreno é um blob 3×3 com
o **canto externo transparente** — as bordas foram desenhadas para ficar em cima de
outro terreno, não do fundo da página. Usadas soltas, a trilha ganhou contorno preto
e pareceu adesivo recortado. Ver
[[Terreno de tile se desenha em duas camadas, base cobrindo e sobreposição com borda]].

## Conexões
- Irmã: [[Mais resolução não compra qualidade em ícone; trocar de meio compra]]
- Visto em: [[naruto-idle]]
- Mapa: [[Frontend]]
