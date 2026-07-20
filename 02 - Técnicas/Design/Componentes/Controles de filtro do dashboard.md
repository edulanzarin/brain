---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-07-20
---

# Controles de filtro do dashboard

> Os dois controles que aparecem em todo dashboard: alternar entre poucas opções e
> filtrar entre muitas.

## Toggle segmentado (pílula)

Fundo `surface-2`, item ativo vira `surface` com sombra — a hierarquia sai da
superfície, não de borda ([[Hierarquia por superfície, não por borda]]).

Serve pra Entradas|Saídas, Valor|Quantidade, Todas|Normais|Canceladas. **Um componente
só** para os três casos: o que muda é a lista de opções, não o componente. Vale até
umas 4 opções; acima disso o toggle vira paredão e o certo é dropdown.

## Dropdown de filtro

Botão com ícone e o rótulo do **estado atual** (não um "Filtrar" genérico — o botão tem
que dizer o que está filtrado agora sem abrir). Painel com checkboxes, ação de "Limpar"
e busca no topo quando a lista é grande (empresas, grupos).

## A regra que vale pros dois

Mostrar um controle **só onde ele muda alguma coisa**. No Questor BI o Valor|Quantidade
some na tela de dados, porque ali ele não faz nada. Controle inerte na tela ensina o
usuário a ignorar controles.

O que esses controles selecionam é estado compartilhável, então vai pra URL — ver
[[Estado compartilhável mora na URL]] e [[Filtro de lista mora na URL]].

## Conexões
- Princípios: [[Estado compartilhável mora na URL]] · [[Hierarquia por superfície, não por borda]]
- Índice: [[Padrões de componentes de dashboard]]
- Irmãs: [[Sidebar em acordeão e layout de módulo]] · [[Blocos de dado - card, KPI e gráfico]]
- Mapa: [[Design]]
