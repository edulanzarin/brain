---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-24
---

# Número de resumo sai do mesmo cálculo que a tela detalha

> Cabeçalho que afirma "o melhor é X" e lista que mostra os itens são **a mesma
> pergunta em dois tamanhos**. Se cada um roda a própria conta, um dia eles
> discordam — e quem discorda em silêncio é sempre o de cima, porque ninguém
> revisa um número que já parecia certo.

## O que vai errado

Não é o resumo estar errado: é ele ter **critério próprio**. A lista filtra o que
não conta; o resumo, escrito depois e em outro arquivo, filtra quase igual. Os dois
respondem "quem é o melhor" com definições diferentes de "quem entra na disputa", e
o desencontro só aparece quando alguém repara que a capa aponta um e a tabela
aponta outro.

Foi assim duas vezes no mesmo produto:

- a home dizia que o pokémon mais forte era um, e a tier list dizia outro — porque
  a home tinha um `filter` próprio de quem era espécie jogável;
- o cabeçalho da Hunt ia afirmar o melhor XP/h enquanto três abas rodavam o mesmo
  ranking por conta própria, cada uma com o seu recorte de "hunt que conta".

## A forma que resolve

**Suba a conta pro pai e distribua o array pronto.** O resumo vira uma redução
sobre o mesmo array que as telas de baixo listam; discordar deixa de ser possível
porque não existem dois arrays ([[Um invariante se garante na estrutura, não no
processo]]).

O ganho de custo — parar de rodar o mesmo cálculo caro três vezes — é real e é o
segundo motivo, não o primeiro. Uma conta duplicada que sempre concorda só custa
tempo; duas definições da mesma coisa custam confiança.

Como saber que chegou a hora: se você está prestes a escrever um `filter` no
componente pai com uma condição que já existe num filho, pare. Ou a condição sobe
junto com os dados, ou você acabou de criar a segunda definição.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Estado de tela pertence à seção, não à página]]
- Visto em: [[piwdex2]]
- Mapa: [[Frontend]]
