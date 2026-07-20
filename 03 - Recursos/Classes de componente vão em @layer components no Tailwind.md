---
tags: [tipo/atomica, projeto/navedesk, dev/frontend, design, conceito]
criado: 2026-07-20
---

# Classes de componente vão em @layer components no Tailwind

> Uma classe própria (`.campo`, `.btn`) escrita **fora** de uma `@layer` ganha
> das utilitárias do Tailwind quando a especificidade empata. Resultado: o
> `w-auto` do JSX não faz nada.

## O sintoma

No [[Navedesk]] a barra de filtros ficou com todos os `<select>` em largura
cheia, empilhados, e o texto do campo de busca por baixo do ícone de lupa. O
JSX estava certo:

```tsx
<select className="campo w-auto min-w-[9rem]" />
<input className="campo pl-9" />
```

## Por que acontece

`.campo` e `.w-auto` têm a **mesma especificidade** (0-1-0). Empate em CSS é
resolvido pela ordem no arquivo — e como o `globals.css` define `.campo` depois
do `@import "tailwindcss"`, ele vence sempre.

Pior com shorthand: `.campo { padding: 0.5rem 0.75rem }` sobrescreve o `pl-9`
inteiro, não só a parte que conflita.

## A regra

Envolva as classes de componente na camada certa:

```css
@layer components {
  .campo { … }
  .btn   { … }
  .tag   { … }
}
```

O Tailwind v4 ordena as camadas `theme → base → components → utilities`. Dentro
de `components`, a classe vira a **base** que as utilitárias podem sobrescrever
— que é exatamente o contrato mental de quem escreve `className="campo w-auto"`.

Vale para tudo que é "componente com utilitária por cima": card, badge, botão,
campo, pílula segmentada.

## Links

- Descoberto em: [[Navedesk]]
- Faz parte de: [[Design]] · [[Padrões de componentes de dashboard]]
- Ver também: [[Sistema de cores e tema do dashboard]]
