---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-08-21
---

# Anúncio em feed não pode vestir a roupa do conteúdo

> Intercalar anúncio numa grade de cards é permitido e tem formato próprio
> (In-feed). O limite é preciso: ele pode **caber** no layout, não pode **passar
> por** conteúdo. Card de anúncio com a mesma moldura, o mesmo hover e o mesmo
> cursor do card de verdade é a violação que o Google chama de formatação
> mimética — e, antes de ser violação, é trapaça com quem está varrendo a grade
> procurando outra coisa.

## As regras que viram código

- **Rótulo:** só a família "Anúncio(s)", "Publicidade" e "Links patrocinados" é
  aceita. "Recomendados", "Veja também", "Recursos", "Links úteis" e "Parceiros"
  são citados nominalmente como título enganoso. Não é campo de criatividade.
- **Densidade:** o Google publica um piso para in-feed — no mínimo **3 blocos de
  conteúdo entre anúncios**. Um a cada 12 cards dá quatro vezes essa folga e
  ainda rende em página de 24.
- **Nunca na primeira célula** (a grade viraria mostruário) e **nunca no fim** (o
  anúncio cola na faixa do rodapé e vira faixa dupla).
- **Sinal visual próprio:** sem a moldura de vidro dos cards, sem hover, sem
  cursor de link, e sem nenhum marcador de identidade do conteúdo — no piwdex2,
  sem número da dex, sem badge de tipo, sem a espinha de stats.
- **Altura reservada** na casca, com `min-height` e não `height`: anúncio que
  chega e empurra a grade é a causa de clique acidental que o próprio Google
  lista. E colapsar o espaço quando não há preenchimento é outro salto — melhor
  deixar o espaço vazio.
- **Nada de anúncio dentro de painel de ferramenta.** Ao lado de um resultado de
  cálculo, o anúncio lê como se fosse dado — e é onde o clique sem querer mora.

## O que isso implica no resto da tela

Vale nos dois sentidos: o site não pode cobrir o anúncio. Balão flutuante,
tooltip, cabeçalho grudado e modal que passem por cima de uma unidade caem na
mesma regra ("conteúdo que encobre total ou parcialmente anúncios"), mesmo por um
instante. No piwdex2 o balão de apoio mede o próprio retângulo contra o de cada
anúncio na tela e sai da frente — ver
[[Pedido de apoio entra depois do valor, e nunca ao lado de si mesmo]].

## O risco que não é o que o dono teme

Para um site de fã que gera tabelas do catálogo público de outro jogo, a
reprovação provável **não é direitos autorais** — é *"conteúdo de baixo valor"* ou
*"conteúdo replicado"*: telas com conteúdo de terceiros sem comentário, curadoria
ou valor agregado. A defesa é conteúdo próprio de verdade (explicação de
mecânica, a lógica dos cálculos, guias escritos), não ajuste de layout.

## Conexões
- Princípio: [[Nota carrega só o que a pessoa não sabe]] · [[Hierarquia por superfície, não por borda]]
- Irmã: [[Slot de anúncio no App Router precisa de casca estável e filho keyado]] ·
  [[Pedido de apoio entra depois do valor, e nunca ao lado de si mesmo]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
