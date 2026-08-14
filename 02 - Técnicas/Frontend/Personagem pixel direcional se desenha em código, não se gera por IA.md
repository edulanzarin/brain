---
tags: [tipo/atomica, camada/padrao, dev/frontend]
criado: 2026-08-14
---

# Personagem pixel direcional se desenha em código, não se gera por IA

> Pra um personagem 2D top-down com 4 direções + caminhada que seja **coerente**,
> desenhe em código (formas + paleta, escala inteira) em vez de gerar cada
> direção/frame por IA. Você controla cada pixel, então a identidade nunca deriva.

## O problema

Gerar sprite por IA (PixelLab etc.) dá arte rica, mas **cada chamada é uma tirada
nova**: as 4 direções saem como 4 personagens levemente diferentes, e os frames de
caminhada trocam a paleta/armadura. Dá pra mitigar (referência + guidance alto,
`/rotate` pra girar o mesmo personagem), mas some o controle fino e depende de uma
API com cota/custo.

## A solução

Desenhar o personagem **em código**, de forma modular:

- **Corpo por direção** (sul/norte/leste; oeste = flip do leste) — desenhado uma vez
  com formas (retângulos/elipses) numa grade baixa (ex.: 32x32) e paleta enxuta.
- **Pernas por fase de passo** (neutro / passo-esq / passo-dir) como camada separada,
  compostas sobre o corpo → gera idle + ciclo de walk sem redesenhar o corpo.
- **Escala inteira + nearest** na hora de renderizar (pixel-perfect).

O output são PNGs commitados no repo; o script gerador fica versionado (arte
reproduzível e editável). Existe skill do Claude pra isso sem API — `pixel-art-gen`
(desenha via JSON de pixels + Pillow). Trade-off honesto: fica mais flat/simples que
a IA, e cada asset novo é trabalho manual.

## O que mais vale lembrar

- **Ferramenta de gerar asset ≠ dependência do jogo.** IA ou código, o que roda é o
  PNG no repo — o "editor" (PixelLab, skill, Aseprite) só aparece na hora de criar.
- Alternativa de graça e sem limite pra volume: **packs CC0** (Kenney, OpenGameArt),
  que também viram PNG teu no repo.

## Conexões
- Princípio: [[Coerência em geração vem de âncora, não de liberdade]]
- Irmã: [[PixelLab só mantém o personagem ao animar com image guidance alto]]
- Visto em: [[Idle Game]]
- Mapa: [[Frontend]]
