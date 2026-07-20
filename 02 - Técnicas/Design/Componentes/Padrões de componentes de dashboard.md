---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-07-18
atualizado: 2026-07-20
---

# Padrões de componentes de dashboard

> Índice dos blocos de UI que se repetem em todo dashboard. Reusar esses padrões faz um
> projeto novo já nascer "da mesma família" dos que existem.

Esta nota era um bloco só com layout, controles, dados e estado misturados. Virou índice;
cada assunto tem nota própria, porque cada um evolui num ritmo diferente.

## As ramificações

- [[Sidebar em acordeão e layout de módulo]] — a estrutura fixa: navegação que escala e
  layout que segura header e filtros.
- [[Controles de filtro do dashboard]] — toggle segmentado e dropdown de filtro.
- [[Blocos de dado - card, KPI e gráfico]] — card, stat tile, gráfico, tabela e como
  carregar sem piscar.

## O que vale pros três

- **Cor e superfície saem de token**, nunca de hex — [[Token semântico em vez de valor literal]].
- **Espaço e raio saem da escala** — [[Escala fechada em vez de valor solto]].
- **Filtro vai pra URL** — [[Estado compartilhável mora na URL]]. Em produção use
  `history.replaceState`, não `router.replace`:
  [[router.replace do Next falha no build de produção]].
- **Preferência pessoal** (tema, grupos favoritos) fica no `localStorage`, não na URL —
  não é estado compartilhável.

## Conexões
- Princípios: [[Container tem largura máxima e respiro constante]] · [[Hierarquia por superfície, não por borda]]
- Ver também: [[Sistema de cores e tema do dashboard]]
- Mapa: [[Design]]
