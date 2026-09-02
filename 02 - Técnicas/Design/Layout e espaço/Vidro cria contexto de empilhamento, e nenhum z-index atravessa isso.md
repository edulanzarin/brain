---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-09-02
---

# Vidro cria contexto de empilhamento, e nenhum z-index atravessa isso

> Num sistema em que os painéis usam `backdrop-filter`, cada painel vira uma
> raiz de empilhamento — e a lista de um combo aberta lá dentro aparece ATRÁS do
> painel seguinte da página, por mais alto que seja o `z-index` dela.

## O problema

O arranjo é banal: um painel de vidro com um combo no topo da tela, outro painel
de vidro logo abaixo. Abrir o combo mostra a lista cortada pelo segundo painel,
como se o componente estivesse quebrado.

A causa não tem nada a ver com o combo. `backdrop-filter` **cria contexto de
empilhamento** — assim como `transform`, `filter`, `opacity` menor que 1 e
`will-change`. A partir daí, o `z-40` da lista compete só com os irmãos DELA,
dentro daquele painel. O painel inteiro, por sua vez, tem `z-index: auto` e
disputa com o painel de baixo pela regra mais antiga do CSS: em empate, ganha
quem vem depois no DOM.

Subir para `z-50`, `z-999` ou `z-[9999]` não muda nada, e é justamente essa
tentativa que faz o defeito parecer inexplicável.

## A solução

Sair da árvore. O conteúdo flutuante vai para um portal no `body` e a posição é
medida a partir do gatilho:

```tsx
const r = ancora.current.getBoundingClientRect();
// `fixed`, não `absolute`: o gatilho pode estar dentro de um container que rola,
// e a posição absoluta na página se descola do controle na primeira rolagem.
setCaixa({ left: r.left, top: r.bottom + 6, width: r.width });
```

Três detalhes que o portal cobra de volta, e sem os quais ele troca um defeito
por outro:

- **Remedir na rolagem e no redimensionamento**, com `capture: true` — a rolagem
  pode ser de qualquer ancestral, e o evento de rolagem não borbulha.
- **Virar para cima quando não cabe embaixo.** Combo no rodapé abre fora da
  janela e a lista fica inalcançável.
- **"Clicou fora" passa a mentir.** O painel não está mais dentro da raiz do
  controle, então o teste de `contains` fecha a lista no primeiro clique dentro
  dela. Marcar o portal (`data-flutuante`) e ignorar cliques que venham de
  dentro dele resolve sem acoplar os dois componentes.

## O que mais vale lembrar

- **Isto é sistêmico, não pontual.** Se o vidro é a linguagem dos painéis, todo
  flutuante do sistema tem o mesmo defeito latente: combo, combo múltiplo, menu,
  dica. Consertar só onde apareceu deixa os outros quebrados em silêncio.
- **A causa é a mesma de uma família de armadilhas.** Antes de brigar por
  camada: alguém me corta (`overflow`), alguém me cobre (contexto de
  empilhamento), quem responde ao Escape primeiro?

## Conexões
- Princípio: [[Propriedade escolhida pelo visual redefine a estrutura por baixo]]
- Irmã: [[Flutuante dentro de modal precisa vencer no z-index e no Escape]] ·
  [[Sticky gruda no container que rola, não na janela]] ·
  [[Vidro flutuante precisa de superfície mais opaca que a chrome]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]] · [[Frontend]]
