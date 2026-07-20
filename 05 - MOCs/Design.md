---
tags: [tipo/moc]
criado: 2026-07-18
---

# Design

Mapa do sistema visual reutilizável entre projetos. A ideia é que cada projeto novo
(dashboard, app) já comece com uma linguagem visual coerente, sem redesenhar do zero.

Nasceu do [[Questor BI]], mas vale pra qualquer dashboard/app. Já foi reusado no
[[NaveDesk]] — primeira vez que o sistema saiu do projeto de origem, e onde
ganhou a camada de tokens de domínio.

## Notas

- [[Sistema de cores e tema do dashboard]] — tokens semânticos + claro/escuro.
- [[Padrões de componentes de dashboard]] — cards, toggles, filtros, sidebar, gráficos.
- [[Validar paleta de gráficos antes de escolher cores]] — como escolher a paleta categórica.
- [[Paleta categórica com estética por restrição]] — como computar a paleta sem
  ela sair feia: estética restringe, acessibilidade otimiza.
- [[Cor de gráfico e cor de texto pedem contrastes diferentes]] — por que a cor
  da série não serve como cor do badge.

## Princípios

1. **Cor com significado, não decoração** — cada cor é um token semântico (entrada, saída,
   espécie 1…5), nunca um hex solto no componente.
2. **Claro e escuro desde o início** — tudo via CSS vars que trocam por `data-theme`.
3. **Números legíveis** — fonte tabular (`tnum`), formato pt-BR, compacto nos eixos.
4. **Estado na URL** — filtros compartilháveis; recarregar não perde contexto.
5. **Cor se computa, não se chuta** — paleta e contraste passam por validador
   antes de entrar; o olho não estima ΔE nem 4,5:1.
6. **Três camadas de token** — primitiva (hex), semântica (papel na interface),
   domínio (papel no negócio). Componente consome a mais específica que couber.

---

Voltar para [[Início]]
