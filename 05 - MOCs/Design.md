---
tags: [tipo/moc]
criado: 2026-07-18
---

# Design

Mapa do sistema visual reutilizável entre projetos. A ideia é que cada projeto novo
(dashboard, app) já comece com uma linguagem visual coerente, sem redesenhar do zero.

Nasceu do [[Questor BI]], mas vale pra qualquer dashboard/app — já reusado no [[Navedesk]].

## Notas

- [[Sistema de cores e tema do dashboard]] — tokens semânticos + claro/escuro.
- [[Padrões de componentes de dashboard]] — cards, toggles, filtros, sidebar, gráficos.
- [[Validar paleta de gráficos antes de escolher cores]] — como escolher a paleta categórica.
- [[Classes de componente vão em @layer components no Tailwind]] — para as utilitárias vencerem.

## Princípios

1. **Cor com significado, não decoração** — cada cor é um token semântico (entrada, saída,
   espécie 1…5), nunca um hex solto no componente.
2. **Claro e escuro desde o início** — tudo via CSS vars que trocam por `data-theme`.
3. **Números legíveis** — fonte tabular (`tnum`), formato pt-BR, compacto nos eixos.
4. **Estado na URL** — filtros compartilháveis; recarregar não perde contexto.

---

Voltar para [[Início]]
