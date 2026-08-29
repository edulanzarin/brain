---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-29
---

# Propriedade com prefixo escrita à mão pode perder a versão padrão no build

> Escrever `backdrop-filter` cru dentro de `@layer components` no Tailwind v4
> faz o Lightning CSS entregar só a `-webkit-`. O computado vira `none`, e o
> defeito não aparece: a superfície continua translúcida, some só o desfoque.

## O que aconteceu

A classe de vidro do [[Privello]] era assim:

```css
@layer components {
  .vidro {
    background-color: var(--color-vidro);
    backdrop-filter: blur(20px) saturate(180%);
    -webkit-backdrop-filter: blur(20px) saturate(180%);
  }
}
```

No CSS servido sobrava **uma linha só**, a `-webkit-`. E o Tailwind ainda emite,
na camada de utilitárias, um `backdrop-filter: var(--tw-backdrop-blur, ) …` que
resolve para vazio. Utilitárias vencem componentes, então o valor final é `none`.

## Por que passou despercebido

Porque o vidro **quase** funcionava. O `background-color` translúcido continuava
lá, a borda de luz também, e a peça parecia certa numa captura estática. O que
faltava era justamente o desfoque, que é o que separa vidro de retângulo
semitransparente. O sintoma só aparece quando passa conteúdo com detalhe por
baixo, e aí lê como sujeira, não como bug.

Quem denuncia é o computado, não o olho:

```js
getComputedStyle(el).backdropFilter; // "none"
```

## A correção

Deixe o Tailwind emitir a propriedade, via `@apply`:

```css
.vidro {
  @apply backdrop-blur-xl backdrop-saturate-150;
  background-color: var(--color-vidro);
}
```

Aí saem as duas versões, na camada certa, e o computado vira
`blur(40px) saturate(1.5)`.

## A regra maior

**Não escreva prefixo de fornecedor à mão em pipeline que já prefixa.** O
transformador assume que ele é o dono daquela propriedade; escrever as duas
versões vira ambiguidade, e a resolução dele pode não ser a sua. Se a ferramenta
tem utilitária para o efeito, use a utilitária.

E vale a checagem: **efeito visual só se confere no computado ou na captura com
conteúdo por baixo**. Efeito que depende de composição (desfoque, mistura,
máscara) falha calado, porque a peça continua desenhando.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
- Irmã: [[Classes de componente vão em @layer components no Tailwind]]
- Visto em: [[Privello]]
- Mapa: [[Design]]
