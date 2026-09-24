---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-09-24
---

# Truncar come o fim da linha, e o valor não pode morar no fim do título

> "Sabrina Tavares pagou R$ 29,90" numa linha de feed com reticências: no
> celular, o que sobrou foi "Sabrina Tavares pagou R$ 2…". O `truncate` corta do
> fim, e o fim era o número, justo a parte que alguém abre o feed para ver.

## O problema

Frase natural põe o valor por último ("fulano pagou X"), e o título da linha é o
lugar natural da frase. Em tela larga cabe; em tela estreita o navegador corta o
que passa, e corta pela direita. Nome e verbo sobrevivem, o dado some.

## A solução

Tirar o valor da frase e dar a ele uma coluna própria que não encolhe
(`shrink-0`), no lado direito, como num extrato:

- título: "Sabrina Tavares pagou" (este pode ser cortado sem perda grave);
- à direita: "R$ 29,90" em cor de dinheiro, com o tempo embaixo.

A frase inteira continua existindo onde ninguém corta: na torrada, que tem espaço
para ela. A descrição do evento devolve as duas formas (título curto + valor, e
a frase completa), e cada lugar escolhe a sua.

## O que mais vale lembrar

- A pergunta ao escrever uma linha de lista: se a largura cair pela metade, o
  que sobra? Tem de sobrar o dado que decide.
- Mesmo defeito, outro mecanismo: [[Rolagem horizontal que não se anuncia esconde a coluna que decide]].

## Conexões
- Princípio: nenhum cobre ainda, folha isolada. **Candidato**, junto da irmã de
  rolagem: "o dado que decide não pode ser o primeiro a sair da tela". As duas
  apareceram no mesmo projeto; promover quando aparecer em outro.
- Irmã: [[Rolagem horizontal que não se anuncia esconde a coluna que decide]] ·
  [[Título e metadados no mesmo flex-wrap deixam o dado decidir a quebra]]
- Visto em: [[telebot]]
- Mapa: [[Design]]
