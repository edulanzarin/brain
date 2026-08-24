---
tags: [tipo/atomica, camada/padrao, dev/backend, sql]
criado: 2026-08-24
---

# Grão fino numa varredura só dispensa os count distinct

> Quando um painel precisa do mesmo fato quebrado por vários eixos, agrupe UMA vez pelo grão mais fino que a tela usa e faça todo o resto — rankings, séries, distintos, calendário — em memória.

## O problema

Painel de produtividade sobre `lctoctb` (32M linhas): a tela pede ranking por pessoa, quebra por origem, top de empresas, série por dia, distribuição por hora, empresas distintas atendidas, dias trabalhados. Escrito do jeito óbvio, vira uma consulta por bloco (seis varreduras do mesmo período) e, dentro de cada uma, `count(distinct)` de empresa e de dia.

Medido no escritório inteiro, um mês: a consulta com três `count(distinct)` levava ~5s. E cada bloco extra pagava a varredura de novo.

## A solução

Uma consulta, agrupada pelo produto das dimensões que a tela usa:

```sql
select codigousuario u, codigoempresa e, codigooriglctoctb o,
       to_char(datahoralctoctb, 'YYYY-MM-DD') d,
       extract(hour from datahoralctoctb)::int h,
       count(*)::int n, coalesce(sum(valorlctoctb), 0)::float v
  from lctoctb
 where datahoralctoctb >= $1::date and datahoralctoctb < ($2::date + 1)
 group by 1, 2, 3, 4, 5
```

O intermediário é minúsculo perto do fato: **2 milhões de lançamentos viram ~6 mil linhas de grão** (o produto real das dimensões é esparso — cada pessoa toca poucas empresas por dia). Um mês custa ~1,5s; o rollup em memória (um laço, alguns `Map`) é ruído.

Do grão saem, sem custo nenhum:

- ranking por pessoa, por origem, por empresa — somando o eixo que interessa;
- distintos (empresas atendidas, dias ativos) — `Map.size`, não `count(distinct)`;
- série por dia ou por mês, calendário e histograma por hora;
- "rodadas de trabalho" = a própria contagem de chaves do grão (empresa × dia × origem), que substituiu um agrupador que o ERP grava zerado.

## O que mais vale lembrar

- **O grão é escolhido pela dimensão mais fina que a tela mostra.** Botar hora no grão não encareceu (mesma varredura), mas cada dimensão a mais multiplica linhas — se o intermediário passar de dezenas de milhares, o eixo raro sai para uma segunda consulta.
- Cadastros (nome de pessoa, de empresa, descrição de origem) entram depois, em consultas separadas, só para os códigos que apareceram.
- O payload da tela é o rollup, não o grão. Guardar em cada pessoa os próprios recortes (origens, empresas, horas, dias) deixa a interface isolar alguém sem nova ida ao banco — e continua pequeno, porque cada pessoa tem poucos dias com movimento.
- Comparar com o período anterior é uma segunda consulta, em paralelo: só `count` e `sum`, sem grão.

## Conexões
- Princípio: [[Reduzir a cardinalidade vem antes de enriquecer]]
- Irmã: [[Agregar antes de juntar em tabelas gigantes no Postgres]]
- Relacionado: [[Módulo contábil do Questor]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Dados]]
