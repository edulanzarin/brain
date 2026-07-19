---
tags: [tipo/atomica, design, dev/frontend, conceito]
criado: 2026-07-18
---

# Padrões de componentes de dashboard

> Blocos de UI que se repetem em todo dashboard. Reusar esses padrões faz um projeto novo
> já parecer "da mesma família" do [[Questor BI]].

## Layout

- **Sidebar em acordeão**: módulos que expandem/colapsam; o módulo da rota atual abre por
  padrão. Escala pra muitos módulos sem virar um paredão. Config das seções num array
  (`{ id, rotulo, path, metrica, descricao }`) que dirige sidebar + header + regras.
- **Layout de módulo** segura o que é compartilhado (header + barra de filtros); cada seção
  é uma rota/página que busca só os seus dados.
- Card padrão: `rounded`, `border-hairline`, `bg-surface`, um header com ícone em
  quadradinho `bg-surface-2` + título + subtítulo `text-muted`.

## Controles

- **Toggle segmentado** (pílula): fundo `surface-2`, item ativo vira `surface` com sombra.
  Usado pra Entradas|Saídas, Valor|Quantidade, Todas|Normais|Canceladas. Um só componente.
- **Dropdown de filtro**: botão com ícone + rótulo do estado atual; painel com busca no topo
  quando a lista é grande (empresas, grupos), checkboxes, "Limpar".
- Mostrar um controle **só onde ele muda algo** (ex.: Valor|Quantidade some na tela de dados).

## Dados e feedback

- **KPI/stat tiles**: rótulo pequeno `muted`, número grande `tnum`, sub-linha de contexto
  (variação vs período anterior, % do total).
- **Gráficos**: wrapper único (`ChartCard`) com título/subtítulo/ação + skeleton + estado de
  recarga (dim/opacity). Recharts com `--grid`/`--baseline`/`--muted` nos eixos; tooltip próprio.
- **Skeleton no primeiro load**, **dim no refetch** (não pisca): React Query com
  `keepPreviousData`. Nada de spinner de tela cheia.
- **Tabela**: linhas clicáveis que expandem (drill-down), paginação simples, busca com debounce.

## Estado

- **Filtros na URL** (`history.replaceState`, não `router.replace` em prod — ver
  [[router.replace do Next falha no build de produção]]). Compartilhável e sobrevive a reload.
- Preferências locais (grupos, tema) no `localStorage`.

## Conexões
- Faz parte de: [[Design]]
- Origem: [[Questor BI]]
- Ver também: [[Sistema de cores e tema do dashboard]]
