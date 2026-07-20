---
tags: [tipo/moc]
criado: 2026-07-18
---

# Design

Mapa do sistema visual reutilizável entre projetos. A ideia é que cada projeto novo
(dashboard, app) já comece com uma linguagem visual coerente, sem redesenhar do zero.

Nasceu do [[Questor BI]], mas vale pra qualquer dashboard/app — já reusado no
[[Navedesk]] e no [[Cofre Digital]].

## Notas

- [[Sistema de cores e tema do dashboard]] — tokens semânticos + claro/escuro.
- [[Padrões de componentes de dashboard]] — cards, toggles, filtros, sidebar, gráficos.
- [[Validar paleta de gráficos antes de escolher cores]] — como escolher a paleta categórica.
- [[Classes de componente vão em @layer components no Tailwind]] — para as utilitárias vencerem.

### Estados da tela (carregando, erro, sucesso)

O que o app faz **enquanto** e **depois** da ação — a parte que costuma sobrar
pro fim e denunciar que o design parou no layout.

- [[Toast em vez de alert para o feedback do app]] — canal de mensagem próprio.
- [[Esqueleto de carregamento imita a forma do conteúdo]] — carregar sem saltar.
- [[Filtro de lista mora na URL]] — o princípio 4 na prática.

## Princípios

1. **Cor com significado, não decoração** — cada cor é um token semântico (entrada, saída,
   espécie 1…5), nunca um hex solto no componente.
2. **Claro e escuro desde o início** — tudo via CSS vars que trocam por `data-theme`.
3. **Números legíveis** — fonte tabular (`tnum`), formato pt-BR, compacto nos eixos.
4. **Estado na URL** — filtros compartilháveis; recarregar não perde contexto.
5. **Todo estado tem visual** — carregando, vazio, erro e sucesso são parte do
   design, não sobra. Nada de `alert()` nem de bloco cinza genérico.

---

Voltar para [[Início]]
