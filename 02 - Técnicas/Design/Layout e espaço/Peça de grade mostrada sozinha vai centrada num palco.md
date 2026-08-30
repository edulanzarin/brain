---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-08-30
---

# Peça de grade mostrada sozinha vai centrada num palco

> Cartão, tile e card de produto nascem para viver numa grade, com vizinhos
> segurando a linha. Quando um deles aparece sozinho numa tela de prévia — "veja
> como seu anúncio vai ficar" —, encostá-lo na esquerda deixa meia linha vazia
> ao lado, e essa sobra não é lida como sobra de grade: é lida como defeito da
> prévia.

## A correção

A peça vai centrada dentro de um palco: uma área com fundo um nível diferente do
da página e o fio de 1px da casa em volta.

```tsx
<div className="flex justify-center rounded-2 border border-linha bg-fundo-2 p-4 sm:p-6">
    <div className="w-full max-w-xs">
        <CartaoAnuncio anuncio={...} />
    </div>
</div>
```

O `max-w` continua sendo o da peça na grade — o palco não a estica. Ele só
resolve o que sobra dos lados.

## Por que o palco e não só o `mx-auto`

Centralizar sozinho tira a sobra torta, mas não explica por que aquele cartão
está ali. O fundo diferente é o que diz **"isto é uma amostra, não mais um bloco
da página"** — a mesma distinção que separa o conteúdo do exemplo num catálogo
de componentes. É superfície fazendo o trabalho de hierarquia.

## Vale lembrar

A prévia mostra a peça REAL com os dados reais, nunca uma imitação desenhada à
mão para a ocasião: imitação envelhece calada, e a pessoa confere uma coisa e
publica outra.

## Conexões
- Princípio: [[Hierarquia por superfície, não por borda]] — o palco se separa da
  página pelo nível de fundo, e a borda é só o acabamento.
- Irmã: [[Peça desenhada fora do DOM é uma segunda implementação do tema, e ela envelhece calada]]
- Visto em: [[Privello]]
- Mapa: [[Design]]
