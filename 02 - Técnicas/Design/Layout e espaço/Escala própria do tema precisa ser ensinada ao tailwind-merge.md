---
tags: [tipo/atomica, camada/padrao, dev/frontend, design, armadilha]
criado: 2026-09-24
---

# Escala própria do tema precisa ser ensinada ao tailwind-merge

> O `tailwind-merge` só conhece a escala padrão do Tailwind. Um tamanho de texto
> com nome próprio (`text-leitura`, `text-corpo`) é lido como COR, e diante de
> uma cor de verdade na mesma lista (`text-tinta`) ele descarta o tamanho
> achando que são duas cores brigando. O texto sai no tamanho herdado, sem erro
> nenhum.

## O que aconteceu

O indicador do NaveX juntava `cn("nx-leitura text-leitura", "text-tinta")`. No
print da tela, o número de 26px saiu com os 13px do corpo, idêntico ao rótulo
acima dele. Nada no build, no lint nem no tipo acusou: as duas classes existiam
no CSS gerado, só que uma delas não chegava no atributo `class`.

Quem denuncia é o atributo, não o CSS: inspecionar o elemento mostra só
`text-tinta`, e o `text-leitura` que está no código-fonte some no caminho.

## A correção

Ensinar a escala ao merge, no mesmo lugar onde o `cn` nasce:

```ts
import { extendTailwindMerge } from "tailwind-merge";

const mesclar = extendTailwindMerge({
  extend: {
    classGroups: {
      "font-size": [{ text: ["micro", "pequeno", "corpo", "medio", "titulo", "leitura"] }],
      rounded: [{ rounded: ["chip", "controle", "painel", "flutua"] }],
    },
  },
});

export const cn = (...i: ClassValue[]) => mesclar(clsx(i));
```

Cor com nome próprio (`text-tinta`, `bg-vidro`) não precisa: o merge já supõe que
o nome desconhecido depois de `text-`/`bg-` é cor. O risco é só da escala que
divide o prefixo com a cor, e o `text-` é o caso clássico.

## O que mais vale lembrar

- Toda escala nova no `@theme` (`--text-*`, `--radius-*`, `--spacing-*` com nome)
  entra no `extendTailwindMerge` no mesmo commit. Escala que o merge não conhece
  funciona até a primeira vez que alguém a combina com outra classe do mesmo
  prefixo.
- O defeito é silencioso e parece gosto: o número pequeno lê como "ficou discreto",
  não como bug. Foi o print que pegou, comparando com o catálogo.

## Conexões
- Princípio: [[Escala fechada em vez de valor solto]]
- Irmã: [[A classe do chamador só vence a do primitivo com tailwind-merge]]
- Visto em: [[NaveX]]
- Mapa: [[Design]]
