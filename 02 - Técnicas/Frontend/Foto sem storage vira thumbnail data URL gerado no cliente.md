---
tags: [tipo/atomica, camada/padrao, dev/frontend]
criado: 2026-08-08
---

# Foto sem storage vira thumbnail data URL gerado no cliente

> Precisa de avatar/foto mas o storage (S3, disco, CDN) ainda não existe? Não
> trave a feature. Reduz a imagem **no cliente** pra um thumbnail quadrado e
> salva o **data URL** numa coluna de texto. Troca por upload de verdade quando
> o storage entrar — a UI e o schema não mudam (é sempre um `src`).

## A técnica

No `<input type="file">`, ao escolher a imagem: desenha num `<canvas>` pequeno
(cover-crop pra quadrado), exporta como JPEG e guarda o data URL.

```ts
function fileToThumb(file: File, size = 160): Promise<string> {
  return new Promise((resolve, reject) => {
    const url = URL.createObjectURL(file);
    const img = new Image();
    img.onload = () => {
      const c = document.createElement("canvas");
      c.width = c.height = size;
      const ctx = c.getContext("2d")!;
      const scale = Math.max(size / img.width, size / img.height); // cover
      const w = img.width * scale, h = img.height * scale;
      ctx.drawImage(img, (size - w) / 2, (size - h) / 2, w, h);
      URL.revokeObjectURL(url);
      resolve(c.toDataURL("image/jpeg", 0.85));
    };
    img.onerror = () => { URL.revokeObjectURL(url); reject(new Error("imagem inválida")); };
    img.src = url;
  });
}
```

Um thumbnail 160×160 JPEG a 0.85 fica em ~6–12 KB de base64 — cabe numa coluna
`String` sem dor. O `<img src>` (ou o componente de avatar) renderiza data URL
igual a uma URL http, então **nada na renderização precisa saber a diferença**.

## O que não esquecer

- **Cap de tamanho no servidor** (ex.: ~400 KB) e **valida o prefixo**
  (`data:image/...;base64,` ou `https?://`) antes de gravar — senão a coluna
  vira depósito de imagem crua e o campo aceita lixo.
- **`URL.revokeObjectURL`** depois de carregar, pra não vazar o object URL.
- **Downscale é obrigatório**, não opcional: gravar a foto original em base64
  incha o banco e o payload da Server Action.
- **Trocar por upload depois é trivial**: a coluna já guarda um `src`; o upload
  real só passa a devolver uma URL http em vez do data URL. Nada a montar de novo.

## Conexões
- Visto em: [[navetalks]] (avatar de usuário no modal da Equipe, pré-storage)
- Mapa: [[Frontend]]
