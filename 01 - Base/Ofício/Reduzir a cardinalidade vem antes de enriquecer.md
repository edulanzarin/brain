---
tags: [tipo/atomica, camada/principio, dev/backend, sql]
criado: 2026-08-24
---

# Reduzir a cardinalidade vem antes de enriquecer

> Sobre fato grande, primeiro se corta o volume — depois se junta nome, se calcula distinto, se fatia. Trabalho caro feito antes do corte é trabalho feito milhões de vezes à toa.

## A regra

Toda consulta sobre tabela enorme tem duas metades: a que **reduz** (filtro por índice + `group by`) e a que **enriquece** (join de cadastro, `count(distinct)`, ordenação, recorte por várias dimensões). A ordem não é estética — é a diferença entre segundos e minutos:

1. Reduza uma vez, no banco, com o filtro que usa índice.
2. Só então enriqueça, sobre o que sobrou.

O "o que sobrou" quase sempre cabe na aplicação: milhões de linhas viram milhares, e milhares de linhas em memória se fatiam de graça, em qualquer eixo, quantas vezes quiser.

## Por que

Os dois casos que ensinaram isso saíram do mesmo banco e da mesma armadilha — pagar caro por linha antes de saber quais linhas importam:

- Ranking de produtos sobre 47M de itens: juntar `produto` e `empresa` linha a linha antes de agrupar torna a consulta inviável; agregar primeiro e juntar nos dez vencedores resolve em segundos ([[Agregar antes de juntar em tabelas gigantes no Postgres]]).
- Painel de produtividade sobre 32M de lançamentos: cada `count(distinct)` no `select` é uma passada a mais sobre o mesmo conjunto; trazer o grão e contar na aplicação corta o custo por três ([[Grão fino numa varredura só dispensa os count distinct]]).

## Na prática

- Ao ver `count(distinct)`, join de cadastro ou `order by` no mesmo `select` que varre o fato, pergunte o que dá para adiar.
- Meça antes e depois na escala real (todas as empresas, período inteiro), não numa empresa — ver [[Fórmula verificada só vale na escala em que foi verificada]].
- Enriquecer na aplicação só vale enquanto o intermediário for pequeno; se o corte não reduz, o problema é o filtro, não o enriquecimento.

## Conexões
- Técnica que aplica: [[Agregar antes de juntar em tabelas gigantes no Postgres]]
- Técnica que aplica: [[Grão fino numa varredura só dispensa os count distinct]]
- Irmã: [[Fórmula verificada só vale na escala em que foi verificada]]
- Mapa: [[Base]]
