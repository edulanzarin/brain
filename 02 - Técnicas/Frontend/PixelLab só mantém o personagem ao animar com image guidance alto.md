---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-14
---

# PixelLab só mantém o personagem ao animar com image guidance alto

> Ao gerar animação a partir de um sprite de referência no PixelLab
> (`animate-with-text`), suba o `image_guidance_scale` (~4.0). No default (1.4) o
> modelo ignora a referência e desenha um personagem diferente a cada chamada.

## O problema

Pipeline top-down: gerar o herói parado com `generate-image-pixflux` (uma direção
por chamada, mesmo `seed` → as 4 direções saem consistentes entre si) e depois
animar a caminhada com `animate-with-text`, passando o idle como `reference_image`.

Com os defaults, o walk **não** era o mesmo personagem do idle: cada direção saía
com paleta, armadura e ângulo diferentes — um guerreiro marrom no idle, um laranja
no walk, um de cota de malha na outra direção. A referência estava sendo enviada,
mas com peso baixo demais pra prender a geração.

## A solução

`image_guidance_scale: 4.0` na chamada de animação. O peso alto força o modelo a
seguir a referência; a caminhada passa a ser o mesmo guerreiro do idle, com as
pernas alternando, coerente entre os 4 frames e as 4 direções.

```jsonc
// POST https://api.pixellab.ai/v1/animate-with-text
{
  "image_size": { "width": 64, "height": 64 }, // text-anim é travado em 64x64
  "description": "a brave human warrior hero, brown leather armor...",
  "action": "walking",
  "view": "high top-down",
  "direction": "south",
  "n_frames": 4,
  "image_guidance_scale": 4.0,          // <- a chave; default 1.4 deriva
  "reference_image": { "base64": "<idle_south.png>" },
  "seed": 1234
}
```

## O que mais vale lembrar

- Free tier é medido em **generations**, não em USD — `GET /balance` devolve
  `usd 0.0` mesmo funcionando; o custo aparece em `usage.generations` na resposta.
- Base do herói: `generate-image-pixflux`, uma chamada por direção, mesmo `seed`,
  `no_background: true`, `view: "high top-down"`. Consistente sem esforço.
- `animate-with-text` é travado em 64x64; `generate-image-pixflux` vai até 400x400.
- Resposta: `image.base64` (geração) e `images[].base64` (animação).

## Conexões
- Princípio: [[Coerência em geração vem de âncora, não de liberdade]]
- Visto em: [[Idle Game]]
- Mapa: [[Frontend]]
