---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-08-20
---

# Quantos filtros existem é decisão de layout, não de produto

> Uma tela de lista com três filtros raramente tem três filtros porque só três importam.
> Tem três porque **só três cabiam** na barra onde eles foram parar — e ninguém percebe que
> a decisão de produto foi tomada pelo layout.

## O problema

O filtro nasce numa barra horizontal acima da lista, junto da busca. É o lugar óbvio e
funciona bem pra dois ou três controles. O quarto aperta, o quinto quebra a linha, e a
partir daí toda ideia nova de filtro morre com "não tem onde por".

O sintoma é enganoso porque parece escopo: a lista "não precisa" de filtro por faixa de
valor, por fraqueza, por item que ela dropa. Na verdade a barra é que não tem 400px
sobrando. A forma escolheu a função, e a conversa sobre o que a tela deveria responder
nunca aconteceu.

A saída comum — esconder tudo atrás de um botão "Filtros" — troca um problema por outro:
**filtro que não está à vista é filtro que ninguém usa**, e ainda cria a lista que parece
quebrada porque sobrou um filtro ligado dentro da gaveta fechada.

## A solução

**Trilho vertical fixo** ao lado da lista quando a tela é de consulta, com:

- **Grupos colapsáveis**, e o grupo que tem filtro ligado **nasce aberto** — filtro ativo
  nunca fica escondido.
- **Contador de ativos** no cabeçalho do trilho e por grupo, com "limpar (N)".
- **Chips removíveis acima da lista**, um por filtro ligado, com o valor por extenso. É o
  que impede a lista de parecer quebrada, e é a única superfície do estado no celular.
- **Contagem por opção** dentro de cada menu ("FAIRY 12") — evita o clique que devolve
  lista vazia. A contagem sai do universo visível, não do resultado já filtrado, senão
  marcar um tipo zera todos os outros e o menu vira beco sem saída.
- No celular, o **mesmo** trilho vira modal. Duas implementações saem do ar uma da outra.

Com o espaço resolvido, a pergunta volta a ser de produto: *quais perguntas essa tela
deveria responder?* Aí aparecem os filtros que não existiam por falta de lugar.

## O que mais vale lembrar

Um filtro só vale se o estado dele mora na URL, senão o custo de montar oito filtros é
pago de novo a cada F5 — ver [[Filtro de lista mora na URL]].

O teste: **liste os filtros que você já quis e não pôs**. Se a lista existir, o layout
está decidindo o produto.

## Visto em

O piwdex tinha busca, tipo e origem. Não por escolha: a barra horizontal não comportava
mais. No piwdex2, com trilho fixo, a mesma tela passou a ter 17 filtros — incluindo dois
que nem o jogo nem o concorrente respondem: "quem apanha deste tipo" (a pergunta de quem
monta time) e o índice reverso de drop ("quem dropa este item").

## Conexões
- Princípio: [[Hierarquia por superfície, não por borda]]
- Irmã: [[Controles de filtro do dashboard]] · [[Filtro de lista mora na URL]] · [[Barra de topo contextual - o módulo injeta suas ferramentas via portal]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
