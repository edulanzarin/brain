---
tags: [tipo/atomica, camada/principio, dev/backend, dados]
criado: 2026-08-28
---

# O que tem ciclo de vida próprio é entidade própria, não modo de outra

> Se uma coisa nasce, muda e morre num ritmo diferente do da entidade que a
> hospeda, ela não é um modo dessa entidade — é entidade própria. Modo escondido
> economiza uma tabela e cobra o preço no dia em que o ciclo diverge.

## A regra

Antes de acomodar uma novidade como flag, status ou coluna opcional de uma tabela
que já existe, pergunte três coisas sobre ela:

1. **Quantas vezes acontece?** Uma resposta por linha ou várias?
2. **Repete no tempo?** Acontece de novo com o mesmo par de atores?
3. **Quando termina?** O fim é o mesmo evento que termina a hospedeira?

Se qualquer resposta divergir da entidade hospedeira, o modo é disfarce: crie a
tabela, com a chave e o fechamento que **ela** pede.

## Por que

O modo escondido não falha na hora — falha no primeiro caso que o ciclo da
hospedeira não previa, e aí o conserto é caro porque já tem dado dentro.

- **Venda de curso**: liberar acesso lendo `pedido.status = 'PAGO'` funciona até o
  primeiro reembolso, e não tem onde encaixar cortesia (acesso sem venda). Acesso
  e venda têm fins diferentes — viraram duas tabelas em
  [[Acesso comprado é linha própria, não status do pedido]].
- **Avaliação de desempenho**: nasceu como uma linha de destinatário de campanha
  com um colaborador anexado. Herdou o ciclo da campanha — **uma** resposta, que
  fecha o link — quando a avaliação queria **várias** (um gestor cada) e queria
  repetir na mesma pessoa a cada rodada. Enquanto foi modo, também não tinha tela:
  aparecia dentro da lista de campanhas, sem filtro e sem histórico por pessoa.

O sintoma comum é a linguagem: quando a tela precisa dizer "campanha" para falar
de "avaliação", o nome já denunciava que eram duas coisas.

## Na prática

Entidade própria é tabela própria, mas o que a define é o resto:

- **Chave que permite repetir.** Único por `(rodada, colaborador)` — não por
  `(colaborador)`, que proibiria a segunda avaliação e mataria o histórico.
- **Filho para o que é N.** Resposta vira tabela filha, não coluna sobrescrita;
  é o que deixa dois gestores responderem sem um apagar o outro.
- **Fim explícito.** O que fecha é um ato registrado, não o primeiro sinal que
  chega — [[Uma pendência de prazo fecha por ato explícito, não por sinal inferido]].
- **Uma porta só.** Ao promover o modo a entidade, o caminho antigo sai junto;
  duas portas para a mesma coisa dividem o dado e o histórico.

Ser entidade própria não é ser ilha: ela referencia a hospedeira quando a origem
importa, como a matrícula aponta o pedido opcionalmente.

## Conexões
- Irmã: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Entidade núcleo cresce por tabela satélite, não por coluna]]
- Técnica que aplica: [[Acesso comprado é linha própria, não status do pedido]]
- Técnica que aplica: [[Registro que muda de casa leva junto o token já distribuído]]
- Mapa: [[Base]]
