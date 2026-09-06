---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-09-05
---

# Sticky só anda dentro do pai, e o pai precisa ser a coluna que rola

> `position: sticky` não gruda na janela: ele anda dentro da caixa do **pai**. Se o
> pai tem a altura do próprio conteúdo — uma `div` de 40px com a barra dentro —, o
> elemento gruda por 40px e vai embora junto com ela. A classe está lá, o `top` está
> certo, e o comportamento é o de um elemento comum.

## O problema

A barra de filtros da vitrine era um componente com raiz própria:

```tsx
<div className="flex flex-col gap-4">
  <AbasGrandes … />          {/* gênero */}
  <div className="pv-trilho">…</div>   {/* chips */}
  <div className="sticky top-16 …">…</div>  {/* filtro e ordem */}
</div>
```

A raiz agrupa e espaça — motivo legítimo. Só que ela termina logo abaixo da linha
que deveria grudar, e o alcance do sticky é a altura do pai menos a altura do
elemento. Aqui isso dá quase zero: a linha "gruda" por alguns pixels e sai da tela
com o resto do bloco.

O engano é achar que quem manda é o ancestral de **rolagem**. Ele manda em *onde*
o elemento cola (é a armadilha da nota irmã); quem manda em *por quanto tempo* é o
pai imediato.

## A forma

O sticky precisa ser filho da caixa que tem a altura da lista inteira — a coluna
da página. Duas saídas:

**1. `display: contents` na raiz do componente.** A caixa some do layout e os
filhos passam a ser filhos da coluna da página, sem mexer em quem chama:

```tsx
<div className="contents">
  <div className="flex flex-col gap-4">{/* abas e chips, ainda juntos */}</div>
  <div className="sticky top-16 …">{/* filtro e ordem */}</div>
</div>
```

O preço: com a caixa some também o `gap`, o `padding` e o fundo dela — o
espaçamento passa a ser o do avô. Por isso o que precisa continuar junto ganha uma
caixa interna, como as abas e os chips acima.

**2. O componente recebe a lista como filho.** A raiz volta a existir e passa a ser
alta de verdade. Custa mudar a API em todas as chamadas, e faz uma barra de filtro
virar dona do feed — o que nem sempre é verdade.

## Como distinguir as duas armadilhas

- **Nunca gruda em lugar nenhum** → pai curto demais (esta nota), ou um
  `overflow: hidden` num ancestral, que mata o sticky de vez.
- **Gruda, mas no lugar errado** → ancestral de rolagem inesperado:
  [[Sticky gruda no container que rola, não na janela]].

Nos dois casos a pergunta não é sobre o `top`. É sobre qual caixa está mandando.

## Visto em

No Privello, a linha de Filtros/Ordenar da vitrine. O feed é rolagem infinita, e
sem ela grudada trocar um filtro depois de trinta cartões custava a rolagem
inteira de volta.

## Conexões
- Princípio: [[Propriedade escolhida pelo visual redefine a estrutura por baixo]] —
  a coluna flex foi escolhida para espaçar e, de quebra, definiu o alcance do
  sticky; o `contents` é a mesma família de propriedade, usada de propósito.
- Irmã: [[Sticky gruda no container que rola, não na janela]]
- Visto em: [[Privello]]
- Mapa: [[Design]] · [[Frontend]]
