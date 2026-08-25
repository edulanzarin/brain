---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-08-25
---

# Motivo de piso tem que tocar a borda do tile

> Um desenho centralizado dentro do tile, com folga até a borda, não vira padrão
> quando ladrilhado: vira o mesmo objeto repetido em grade. Padrão de piso nasce
> do **encontro entre tiles vizinhos**, então o motivo precisa cruzar a borda e
> se completar do outro lado.

## O problema

Desenhar um losango, uma flor ou um círculo no meio dos 32x32 e repetir parece
mosaico na cabeça e sai bolinha na tela. O olho lê a **grade**, não o desenho:
cada tile fica isolado no seu quadrado e o espaço vazio entre eles domina.

Foi exatamente o que aconteceu no mosaico de uma praça: círculo azul centrado no
tile, e a praça inteira virou poá.

## A solução

Faça a figura tocar as quatro bordas, de forma que o pedaço que sai por uma se
encaixe no que entra pela vizinha. Um losango medido por distância de Manhattan a
partir do centro resolve isso quase de graça — ele encosta nos quatro pontos
médios das bordas, e quatro tiles vizinhos fecham o desenho entre si:

```python
h = TILE // 2
d = abs(x - h + 0.5) + abs(y - h + 0.5)   # distância de losango
if   d < h - 5: claro          # miolo
elif d < h - 3: acento         # contorno
elif d > h + 3: escuro         # o canto, que fecha com o vizinho
```

O teste é sempre o mesmo: **renderize 4x4 do tile antes de aceitar**. Um tile
sozinho não diz nada sobre como ele ladrilha.

## O que mais vale lembrar

O contrário também vale: quando você QUER que o elemento leia como objeto solto
(uma pedra, uma poça), aí sim ele fica centrado e com folga. A folga é o que
separa "textura" de "coisa".

## Conexões
- Princípio: [[Escala fechada em vez de valor solto]]
- Irmã: [[Mais resolução não compra qualidade em ícone; trocar de meio compra]]
- Visto em: [[Vespéria]]
- Mapa: [[Design]]
