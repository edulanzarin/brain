---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-08-24
---

# A mesma grandeza usa a mesma escada nas duas telas

> Se duas telas do mesmo sistema respondem "quão raro é isto?", elas têm que
> responder com **os mesmos nomes, as mesmas cores e a mesma ordem**. Duas
> escadas para uma grandeza é o que faz a segunda tela parecer de outro produto —
> e é sentido antes de ser diagnosticado.

## O sintoma chega como estética

A queixa não vem como "a escala está errada". Vem como *"o raro dos itens não é o
mesmo raro da Pokédex"* — alguém percebeu que duas telas irmãs falam línguas
diferentes, mas o que se vê é a superfície.

Vale ouvir esse tipo de queixa como técnica. Por trás dela costuma haver uma
grandeza modelada duas vezes, cada vez por quem estava construindo aquela tela.

## O que reusar é o TIPO, não o desenho

Reuso frouxo — "usei umas cores parecidas" — não resolve, porque a cor precisa
significar posição na escada. O que se compartilha é o tipo em si:

```ts
// a escada vive num lugar só; as duas telas importam DELE
export type Rarity = "COMMON" | "UNCOMMON" | "RARE" | "EPIC" | "LEGENDARY" | "MYTHIC"
export const RARITY_ORDER: Rarity[]
export const RARITY_COLOR: Record<Rarity, string>
export const RARITY_LABEL: Record<Rarity, string>
```

Com o tipo compartilhado, "Épico" na tela A ocupa exatamente a mesma posição que
"Épico" na tela B — que é a única condição em que a cor passa a querer dizer
alguma coisa. Sem ela, a cor é decoração que muda de sentido conforme a rota.

O reuso desce até a URL: se a dex filtra raridade em `?r=`, a outra tela filtra em
`?r=` também. Quem aprendeu a ler o link de uma lê o da outra.

## Espelhar vale até onde o vocabulário deixa

O contrapeso, e ele é concreto. Copiando o card de uma tela pra outra, a fila de
**três** números do original não coube: no card de 218px, um terço da largura não
segura o rótulo "OURO/ABATE" em caixa alta, e os três rótulos se atropelaram. O
original tinha três palavras de cinco letras; o domínio novo não tem.

Espelhar é da FORMA e da ESCADA, não da contagem. O número que não coube não foi
cortado — desceu pro rodapé, onde há linha inteira. Copiar a contagem junto teria
sido copiar a referência no ponto em que ela não se aplica.

## Conexões
- Princípio: [[Escala fechada em vez de valor solto]]
- Irmã: [[Sinal booleano da fonte não ocupa o lugar de uma escala]] ·
  [[Chip que serve a duas grandezas declara qual delas mostra]] ·
  [[Acento da interface é um token separado da cor de dado]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
