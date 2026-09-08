---
tags: [tipo/atomica, camada/padrao, dev/backend, sql, armadilha]
criado: 2026-09-08
---

# Predicado redundante é o que faz o índice entrar quando o join casa colunas pareadas

> Quando a condição que restringe está no `join` (ligando duas colunas de tabelas diferentes), o planner não tem constante nenhuma para procurar no índice — ele varre. Repetir a mesma restrição no `where` como lista de valores literais é o que transforma a varredura em busca indexada, sem mudar o resultado.

## O caso

Achar, no razão contábil (32M de linhas), os lançamentos que tocam a conta de
encerramento de cada empresa. A conta é diferente em cada uma, então o par certo
vem de uma tabela auxiliar:

```sql
from lctoctb l
join contas c on c.codigoempresa = l.codigoempresa
             and (l.contactbdeb = c.contactb or l.contactbcred = c.contactb)
```

Correto e lento: **58 s** para doze meses. O `join` garante o par, mas para o
planner `c.contactb` é uma coluna, não um valor — não há o que buscar em
`(codigoempresa, contactbdeb)`. Some a isso o `or` entre débito e crédito, e o
plano vira varredura de período.

Acrescentando ao `where` a mesma restrição em forma de LISTA — as contas
distintas, poucas dezenas — e a lista de empresas do recorte:

```sql
where l.codigoempresa = any($1::int[])
  and (l.contactbdeb = any(array(select contactb from distintas))
       or l.contactbcred = any(array(select contactb from distintas)))
```

**menos de 1 s**, mesmo resultado. Agora existe prefixo (`codigoempresa`) e valor
(`contactb`) para os dois índices, e o `or` vira um bitmap de duas buscas.

## A regra

O `join` continua ali e é ele que garante a correção — a lista é mais frouxa (a
conta 4855 de uma empresa pode ser outra coisa na vizinha), então filtrar só por
ela traria linhas demais. Os dois trabalham juntos:

- **lista no `where`** → o planner corta o volume por índice;
- **`join` pareado** → devolve só os pares que valem.

Sozinho, cada um falha de um jeito: a lista traz demais, o join lê tudo.

## Quando usar

- A restrição real é um PAR (empresa × conta, cliente × produto) guardado noutra tabela.
- O conjunto de valores distintos é pequeno (dezenas), mesmo que os pares sejam milhares.
- Existe índice composto começando pela coluna do recorte.

Se a lista distinta for grande, o ganho some — aí o caminho é reduzir a
cardinalidade antes, como em [[Agregar antes de juntar em tabelas gigantes no Postgres]].

## Conexões
- Princípio: [[Reduzir a cardinalidade vem antes de enriquecer]]
- Irmã: [[Agregar antes de juntar em tabelas gigantes no Postgres]]
- Relacionado: [[Módulo contábil do Questor]] · [[Fechamento mensal no Questor - a conta de Encerramento do Exercício]]
- Visto em: [[Navetech Hub]] (Contábil → Produtividade → Fechamento)
- Mapa: [[Dados]]
