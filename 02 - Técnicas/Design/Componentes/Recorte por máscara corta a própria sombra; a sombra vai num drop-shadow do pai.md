---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-09-24
---

# Recorte por máscara corta a própria sombra; a sombra vai num drop-shadow do pai

> Forma que não é retângulo (ingresso com furos de picote, etiqueta com canto
> cortado) sai de uma `mask` no CSS. A máscara corta tudo o que o elemento pinta,
> inclusive o `box-shadow`: a peça fica recortada e chapada, sem sombra nenhuma.

## A receita do ingresso

Dois furos, um em cima e um embaixo, exatamente onde o canhoto começa. Cada gradiente
radial cobre metade da altura (51%, para as metades se sobreporem sem fresta) e abre o
furo na borda da sua metade:

```css
.ingresso {
  --canhoto: 7.25rem;
  --furo: 9px;
  --x: calc(100% - var(--canhoto));
  mask:
    radial-gradient(circle at var(--x) 0, #0000 var(--furo), #000 calc(var(--furo) + .5px)) top / 100% 51% no-repeat,
    radial-gradient(circle at var(--x) 100%, #0000 var(--furo), #000 calc(var(--furo) + .5px)) bottom / 100% 51% no-repeat;
}
```

O meio pixel entre transparente e preto é o antisserrilhado do furo. A picotagem é
um `border-left: dashed` num pseudo-elemento do canhoto, entre os dois furos.

## A sombra

`box-shadow` é pintado pelo próprio elemento, então a máscara o come. A sombra vai
num `filter: drop-shadow()` do **pai**: o filtro roda depois da máscara do filho e
segue o contorno recortado, com os furos incluídos. De brinde, a sombra que vaza
para dentro do furo faz ele parecer um buraco de verdade.

Custo: `filter` cria camada de composição. Em duas ou três peças na tela não pesa;
numa grade de cem, vale medir ([[Animação de enfeite escolhe a propriedade pelo custo, não pelo efeito]]).

## Conexões
- Princípio: [[Propriedade escolhida pelo visual redefine a estrutura por baixo]]
- Irmã: [[Vidro cria contexto de empilhamento, e nenhum z-index atravessa isso]]
- Visto em: [[telebot]]
- Mapa: [[Design]]
