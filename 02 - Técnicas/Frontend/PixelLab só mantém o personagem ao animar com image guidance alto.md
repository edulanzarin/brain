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
- `animate-with-text` **exige ≥64x64** (rejeita 48 com 422); `generate-image-pixflux`
  aceita de 32 a 400. Resposta: `image.base64` (geração) e `images[].base64` (animação).
- **Chunkiness = tamanho do canvas.** Pixel "grandão"/retrô vem de gerar num canvas
  menor (48, 32), com `shading: flat` + `detail: low` + `outline: single color black`.
  64px com low detail ainda sai "renderizado".
- **A armadilha da resolução com animação:** se a arte é 48px (chunky) mas a animação
  de frames só existe ≥64, e 48 não é redutível por inteiro a partir de 64 (só 32 é
  — downscale nearest 2:1 limpo), então **48 crocante e frames de perna reais não
  cabem juntos**. Saídas: (a) animar a 64 e reduzir 2:1 pra 32; (b) ficar em 48 e
  fazer a caminhada **procedural** no motor (quicada/bob), mantendo a arte crocante;
  (c) `animate-with-skeleton` (aceita 16–256) com poses, mais trabalhoso. Gerar
  poses soltas de walk via `pixflux` + `init_image` NÃO mantém a identidade (testado:
  perde escudo/pose mesmo com strength 200).
- **Render pixel-perfect:** `scaleMode: "nearest"`, escala **inteira** (×2, ×3) e
  posições arredondadas; no CSS, `image-rendering: pixelated`. Escala fracionária
  (×2.5) borra o grid.

## Conexões
- Princípio: [[Coerência em geração vem de âncora, não de liberdade]]
- Visto em: [[Idle Game]]
- Mapa: [[Frontend]]
