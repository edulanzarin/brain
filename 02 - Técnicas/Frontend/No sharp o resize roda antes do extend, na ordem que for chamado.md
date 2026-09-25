---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-09-25
---

# No sharp o resize roda antes do extend, na ordem que for chamado

> O `sharp` monta uma fila de operações com ordem própria, e não a ordem das
> chamadas. `extend(...).resize(192, 192)` parece "completar o quadrado e depois
> reduzir", mas o resize roda primeiro, com o `fit: "cover"` padrão, e o extend
> acrescenta a margem depois: a imagem sai retangular e, desenhada num quadrado,
> achatada.

## O caso

Os cubos dos módulos do Nexo (1254 px, cubo mais alto que largo, margem
transparente) iam virar ícones de 192 px: recortar a margem (`trim`), completar
um quadrado com transparência (`extend`) e reduzir (`resize`). Numa corrente só,
os arquivos saíram com 338 a 379 × 192 px, e na tela o `<img>` quadrado espremeu
o cubo na horizontal.

## A regra

Operação que muda a geometria e depende de outra vem em outra instância:

```js
const recortado = await sharp(origem).ensureAlpha().trim().toBuffer();
const info = await sharp(recortado).metadata();
const lado = Math.max(info.width, info.height);
const quadrado = await sharp(recortado)
  .extend({ /* completa até lado × lado, fundo transparente */ })
  .png()
  .toBuffer();
await sharp(quadrado).resize(192, 192).png({ palette: true }).toFile(destino);
```

E confira o resultado pelo tamanho do arquivo, não pelo olho num mosaico que
redimensiona tudo para quadrado: o mosaico esconde exatamente esse erro.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
  (a prova é o artefato final, não a etapa intermediária)
- Visto em: [[NaveX]]
- Mapa: [[Frontend]]
