---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-09-03
---

# Terreno de tile se desenha em duas camadas, base cobrindo e sobreposição com borda

> Tileset de terreno não é mosaico de peças justapostas. É pintura por camada: uma
> cobre tudo, a de cima recorta em cima dela.

## A armadilha

O blob 3×3 de um autotile tem o **canto externo transparente**: a borda foi desenhada
para ficar sobre outro terreno. Usada como mosaico, direto no fundo da página, o
canto vira o fundo — a trilha de areia ganhou contorno preto e pareceu adesivo
recortado sobre a grama.

## O desenho certo

```
camada 1   terreno base, no mapa INTEIRO, só tile de preenchimento
camada 2   sobreposição, só onde a célula declara, com o blob 3×3
```

No modelo de dados isso significa que a célula **não** carrega o terreno base: ela
carrega, quando tem, o terreno **sobreposto**. Célula sem terreno declarado é chão
puro. O mapa é que declara `terrenoBase`.

## O preenchimento precisa de variante, senão o chão vira planilha

O centro do blob é um tile só. Repetido em 600 células, ele denuncia a grade. Os
tilesets desse tipo trazem 2 a 4 variantes de preenchimento sem borda — escolha por
hash da coordenada, não por sorteio a cada quadro: `hash(x, y)` é estável entre
renders, então o tile não pisca ao rolar o mapa.

## Objeto alto ancora no tile de baixo

Consequência prática que morde na primeira fileira: uma árvore 2×3 posicionada em
`y = 0` desenha nas linhas −1 e 0, e o que sobra na tela é a raiz. A moldura superior
do mapa precisa de objeto de 1 tile de altura.

E a fila de desenho ordena por `y`: quem está mais embaixo pinta por cima, senão a
árvore da frente fica atrás da de trás.

## Conexões
- Princípio: [[Propriedade escolhida pelo visual redefine a estrutura por baixo]]
- Irmã: [[Motivo de piso tem que tocar a borda do tile]] · [[Eixo de folha de sprite se confere sprite a sprite, não se supõe]]
- Visto em: [[naruto-idle]]
- Mapa: [[Design]]
