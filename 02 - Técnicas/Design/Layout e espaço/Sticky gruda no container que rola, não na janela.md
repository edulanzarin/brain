---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-08-20
---

# Sticky gruda no container que rola, não na janela

> `position: sticky; top: 3rem` num cabeçalho de tabela larga não gruda 3rem abaixo do topo
> da tela: gruda 3rem abaixo do topo da **div com `overflow-x-auto`** que envolve a tabela —
> ou seja, 3rem PRA DENTRO, cobrindo a primeira linha.

## O problema

Tabela densa quer duas coisas ao mesmo tempo: rolar na horizontal no celular
(`overflow-x-auto` no wrapper) e manter o cabeçalho visível na vertical (`sticky` no
`thead`). As duas parecem independentes. Não são.

Pela spec, quando um eixo do `overflow` deixa de ser `visible`, o outro é promovido de
`visible` pra `auto`. Então `overflow-x: auto` cria um **container de rolagem nos dois
eixos**. E `sticky` sempre se ancora no ancestral rolável mais próximo — que passou a ser
esse wrapper, não o documento.

O `top-12` que existia pra passar por baixo da barra fixa do site vira, dentro do wrapper,
um deslocamento de 48px pra baixo a partir do topo da tabela. O cabeçalho desce e come a
primeira linha. Nada no CSS parece errado; o número simplesmente passou a medir outra coisa.

## A solução

Escolher de propósito quem é o container de rolagem:

```tsx
// o wrapper rola nos DOIS eixos, e o cabeçalho gruda NELE
<div className="max-h-[calc(100dvh-11rem)] overflow-auto">
  <table className="min-w-[900px]">
    <thead className="sticky top-0 z-10 bg-surface/95 backdrop-blur"> ... </thead>
```

Com altura no wrapper, a rolagem vertical é dele, o `top-0` cola o cabeçalho exatamente na
borda e a tabela ganha o comportamento de planilha — cabeçalho preso, corpo correndo. Sem
altura, o wrapper não rola na vertical e o `sticky` vira posição normal: aí o certo é
`top-0` mesmo, e aceitar que não há cabeçalho preso.

O que **não** funciona é misturar: wrapper com `overflow-x-auto` sem altura + `top` medido
contra a janela.

## O que mais vale lembrar

Sempre que um `sticky` "gruda no lugar errado", a pergunta não é sobre o `top` — é
**"qual é o ancestral rolável?"**. Costuma ser um `overflow-hidden`/`overflow-x-auto`
colocado três divs acima por outro motivo (cortar sombra, permitir rolagem lateral).

O primo dessa armadilha: `overflow-hidden` num ancestral **mata** o sticky de vez (o
elemento não tem onde grudar) e também corta menus flutuantes — que é o motivo de popover
e dropdown irem pro `body` por portal.

## Visto em

No piwdex2 a tabela da Pokédex tinha `thead sticky top-12` (a altura da barra do site) e o
cabeçalho aparecia por cima da primeira linha, escondendo o Dragonite. O `top-12` estava
medindo a partir do wrapper `overflow-x-auto`, não da janela.

## Conexões
- Princípio: [[Container tem largura máxima e respiro constante]]
- Irmã: [[Modal com conteúdo que cresce tem teto de altura e área que rola]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]] · [[Frontend]]
