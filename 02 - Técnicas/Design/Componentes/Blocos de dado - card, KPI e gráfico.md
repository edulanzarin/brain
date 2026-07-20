---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-07-20
---

# Blocos de dado - card, KPI e gráfico

> Os recipientes que mostram número: card, stat tile, gráfico e tabela. Todos com o
> mesmo esqueleto, pra tela nova nascer parecida com as que já existem.

## Card padrão

`rounded`, `border-hairline`, `bg-surface`. Header com ícone em quadradinho
`bg-surface-2`, título e subtítulo em `text-muted`. É o recipiente de tudo — KPI,
gráfico e tabela são variações dele, não componentes independentes.

## KPI / stat tile

Rótulo pequeno em `muted`, número grande em fonte tabular (`tnum`, senão o número dança
ao atualizar), e uma sub-linha de contexto: variação contra o período anterior, ou
percentual do total. Número sem comparação não informa — 4.200 é bom ou ruim?

## Gráfico

Wrapper único (`ChartCard`) com título, subtítulo, ação, skeleton e estado de recarga.
Recharts usando `--grid` e `--baseline` nos eixos e `--muted` nos rótulos; tooltip
próprio, porque o padrão da lib ignora o tema.

A paleta categórica das séries tem ordem fixa e validada — ver
[[Validar paleta de gráficos antes de escolher cores]].

## Tabela

Linhas clicáveis que expandem (drill-down), paginação simples e busca com debounce.

## Carregar sem piscar

**Skeleton no primeiro load, dim no refetch.** React Query com `keepPreviousData`: o
dado antigo fica na tela com opacidade reduzida enquanto o novo chega, em vez de a tela
sumir e voltar. Nada de spinner de tela cheia. Ver
[[Esqueleto de carregamento imita a forma do conteúdo]].

## Conexões
- Princípios: [[Todo estado da tela tem visual]] · [[Token semântico em vez de valor literal]]
- Índice: [[Padrões de componentes de dashboard]]
- Irmãs: [[Sidebar em acordeão e layout de módulo]] · [[Controles de filtro do dashboard]]
- Mapa: [[Design]]
