---
tags: [tipo/atomica, camada/padrao, dev/frontend, design, armadilha]
criado: 2026-09-24
---

# Fundo no body cobre a camada de z-index negativo

> Uma textura de fundo fixa num `::before` com `z-index: -1` some quando o `html`
> e o `body` têm fundo os dois. O `body` pinta o dele DEPOIS das camadas de
> z-index negativo, por cima delas.

## O problema

O NaveX desenha o papel quadriculado e os brilhos da marca num `body::before`
fixo, atrás de tudo. Com `background` no `html` e no `body`, o print saiu com o
fundo liso: a grade existia no DOM, com tamanho e cor certos, e não aparecia.

A ordem de pintura de um contexto de empilhamento é: fundo do elemento raiz,
depois os filhos de z-index negativo, depois o fundo dos blocos comuns (o `body`
é um deles). Quando só o `body` tem fundo, o navegador o promove para a tela
inteira e ele vira o fundo da raiz; quando o `html` também tem, o `body` pinta o
seu como bloco comum, e aí cobre a camada negativa.

## A solução

Fundo num lugar só, no `html`. O `body` fica transparente:

```css
html { background: var(--fundo); }
body { min-height: 100dvh; } /* sem background */
body::before { content: ""; position: fixed; inset: 0; z-index: -1; /* textura */ }
```

## O que mais vale lembrar

- O sintoma é ausência, não erro: a camada está lá, com computado certo. Quem acha
  é comparar o print com a intenção, ou desligar o fundo do `body` no inspetor.
- Tema escuro esconde mais: a grade é tênue de propósito, e "está sutil demais"
  soa como gosto antes de soar como bug.

## Conexões
- Princípio: [[Propriedade escolhida pelo visual redefine a estrutura por baixo]]
- Irmã: [[Vidro cria contexto de empilhamento, e nenhum z-index atravessa isso]]
- Visto em: [[NaveX]]
- Mapa: [[Design]]
