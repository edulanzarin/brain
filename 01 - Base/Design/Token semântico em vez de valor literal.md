---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-07-20
---

# Token semântico em vez de valor literal

> Nenhum valor de design aparece cru no componente. Ele vira um nome que diz o
> **papel**, e o valor mora num lugar só.

## A regra

`background: var(--surface)`, nunca `background: #12161c`. O nome descreve a função
(superfície, tinta, entrada, saída, crítico), não a aparência (`--cinza-escuro`,
`--azul-2`). Token nomeado pela aparência quebra no primeiro tema novo: um
`--azul-claro` que precisa ficar escuro no dark mode é uma mentira no código.

Vale pra cor, mas também pra espaçamento, raio de borda, sombra, peso de fonte e
duração de animação — qualquer valor que se repete.

## Por que

Três ganhos que só aparecem com o tempo:

- **Trocar tema é trocar um bloco**, não caçar hex em 200 arquivos. É o que torna
  claro/escuro barato o suficiente pra existir desde o dia 1.
- **A decisão fica revisável.** Com o valor espalhado, ninguém sabe se `#2a78d6` em
  dois arquivos é a mesma intenção ou coincidência. Com token, é explícito.
- **O nome carrega a regra.** `--critical` obriga quem usa a pensar se aquilo é mesmo
  crítico. Hex não pergunta nada.

## Na prática

O token semântico costuma se apoiar numa escala fechada por baixo — ver
[[Escala fechada em vez de valor solto]]. A camada concreta (CSS vars, `@theme` do
Tailwind, claro/escuro sem flash) está em [[Sistema de cores e tema do dashboard]].

## Conexões
- Depende de: [[Escala fechada em vez de valor solto]]
- Padrão que aplica: [[Sistema de cores e tema do dashboard]] · [[Validar paleta de gráficos antes de escolher cores]]
- Mapa: [[Base]] · [[Design]]
