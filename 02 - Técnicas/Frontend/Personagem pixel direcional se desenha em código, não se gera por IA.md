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
- **Código à mão só compensa pra asset simples/único.** Pra volume + qualidade
  (personagem com 4 direções, walk, muito gear), meu pixel à mão sai quadrado/feio.
  A via boa é **pack aberto**: CC0 (Kenney, OpenGameArt) ou, melhor ainda pra
  personagem, o **LPC (Universal LPC Spritesheet)** — corpo + roupa + armadura + arma
  em **camadas** que se compõem. Baixa as camadas, compõe (`alpha_composite`) e fatia
  as linhas de walk (up=8, left=9, down=10, right=11; 64x64/frame). Licença LPC:
  CC-BY-SA 3.0 / GPL 3.0 (exige atribuição — guarde um CREDITS).
  - **Armadilha LPC:** no gerador Universal (sanderfrenken) o **corpo NÃO tem cabeça**
    — `body/bodies/.../light.png` é só torso+membros; a cabeça é camada separada
    (`head/heads/human/.../light.png`). Esquecer a cabeça = personagem "sem rosto"
    (o cabelo flutua sobre o topo do torso). Ordem: corpo → cabeça → pernas → pés →
    torso → cabelo. Chibi via warp (ampliar a cabeça) cobre o rosto e fica pior —
    a proporção do LPC com a cabeça já lê fofa, não force.
  - **LPC não é só personagem — tem terreno e objetos** (mesma família → cenário
    coerente com o herói). Folhas base do LPC (grama, terra, pedra, água, árvores,
    barril, baú, pedra, placa) em `.../lpc_full_assets/base_assets/` (mirror
    `seveibar/liberated-pixel-cup`). Como usar: **tile de chão é 32px** — o interior
    sem costura das folhas 3x6 de terreno fica em `(32,96)`; ache-o programaticamente
    pelo tile de menor "custo de emenda" (borda direita vs esquerda + topo vs base) e
    exigindo alpha 100% (descarta bordas de transição). Objetos vêm em folhas com
    vários itens: recorta a sub-região e apara o alpha (`getbbox`). **Árvore = copa
    (`treetop.png`) sobre tronco (`trunk.png`)**, centralizados, copa sobrepondo o
    topo do tronco ~30px (a copa sozinha flutua como arbusto). Escale o tile 32px por
    um fator inteiro pra o texel casar com o do personagem (pixel-perfect).
- **Personagem em camadas é o encaixe de um jogo onde o gear muda o visual** (Albion):
  a decisão de design escolhe a tecnologia de arte. Se o item define a aparência, o
  personagem tem que ser montado por camadas, não um sprite fechado.

## Conexões
- Princípio: [[Coerência em geração vem de âncora, não de liberdade]]
- Irmã: [[PixelLab só mantém o personagem ao animar com image guidance alto]]
- Visto em: [[Idle Game]]
- Mapa: [[Frontend]]
