---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-09-24
---

# Degradê de SVG com id fixo some junto com a instância escondida que o define

> Um ícone em SVG com `<linearGradient id="marca">` e `fill="url(#marca)"`,
> renderizado duas vezes na página: uma na barra lateral do desktop, outra no topo
> do celular. No celular o símbolo sumia, sem erro nenhum.

## Por quê

O `id` é global no documento. Com duas instâncias, as duas definem `#marca` e o
navegador resolve `url(#marca)` para a **primeira** definição. A primeira estava na
barra lateral, que no celular fica em `display: none`, e o navegador não pinta um
degradê de dentro de um nó que não renderiza. O ícone visível apontava para um
degradê que não existia na tela.

No desktop tudo parecia certo, porque ali a primeira instância é a visível. O defeito
só aparece no tamanho de tela em que a instância que define é a escondida.

## A saída

Um id por instância. No React, `useId()` (vale em componente de servidor também):

```tsx
const id = `marca-${useId().replace(/:/g, "")}`;
// <linearGradient id={id}> ... <path fill={`url(#${id})`} />
```

O `replace` tira os dois-pontos que algumas versões do React põem no id, porque eles
atrapalham o seletor dentro de `url(#...)`.

Alternativas quando não há como gerar id: definir o degradê uma vez num `<svg>`
oculto por tamanho zero (não por `display: none`) no topo da página, ou trocar o
degradê por cor sólida.

## Conexões
- Princípio: folha isolada; nenhum princípio cobre ainda (o parente mais perto é conferir no tamanho de tela onde o defeito mora)
- Irmã: [[Componente de ícone não atravessa a fronteira server-client]]
- Visto em: [[telebot]]
- Mapa: [[Frontend]]
