---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-21
---

# Altura 100% em item de grid de linha automática volta ao tamanho intrínseco

> `height: 100%` dentro de uma linha de grid que se dimensiona pelo conteúdo é
> **dependência circular**: a linha depende do item e o item depende da linha. O
> navegador desiste do 100% e usa o tamanho intrínseco — e o conteúdo vaza pra
> fora de uma caixa que tinha tamanho fixo.

## O sintoma

A primitiva de sprite do [[piwdex2]] é uma caixa de tamanho fixo com a imagem
centralizada:

```tsx
<span className="relative grid place-items-center" style={{ width: 34, height: 34 }}>
  <img className="relative h-full w-full object-contain" src={...} />
</span>
```

Parece certo, e funciona em quase todo lugar — porque quase todo sprite do jogo é
quadrado. O Delibird é 32x64. Nele a caixa media 34px e a imagem media **68px**:
o pokémon descia e cruzava o divisor da linha da tabela. Medindo no DevTools, a
linha implícita do grid tinha `grid-template-rows: 68px` — a linha havia crescido
pra caber a altura intrínseca da imagem.

## Por que

`place-items-center` não estica o item (não é `stretch`), então a linha é
automática e se mede pelo conteúdo. Aí `height: 100%` no item pergunta o tamanho
da linha, que pergunta o tamanho do item: ciclo. A regra do CSS nesse caso é
tratar a altura como `auto` — o intrínseco (64px) — e a linha cresce pra ele.
A caixa continua com os 34px do `style`, e o excedente simplesmente transborda.

## A solução: tirar o item da negociação de tamanho

```tsx
<img className="absolute inset-0 h-full w-full object-contain" />
```

Fora do fluxo, o contêiner do item passa a ser o bloco posicionado (a caixa de
34px, que tem altura declarada), o 100% resolve e o `object-contain` centraliza o
que couber. Quem manda na caixa volta a ser o `size`.

## O que mais vale lembrar

- O sintoma só aparece em conteúdo **não-quadrado** ou de proporção inesperada —
  então ele passa por todo o desenvolvimento e estoura quando entra dado real
  variado (aqui, uma tabela com 342 sprites diferentes).
- Vale pra qualquer item de grid/flex de trilha automática, não só imagem.
- Diagnóstico rápido: comparar `getBoundingClientRect()` do item e do pai. Se o
  item é maior que a caixa que tem tamanho declarado, é isto.

## Conexões
- Princípio: [[Dado que chega preenche espaço reservado, não empurra a tela]]
- Irmã: [[Sticky gruda no container que rola, não na janela]]
- Visto em: [[piwdex2]]
- Mapa: [[Frontend]]
