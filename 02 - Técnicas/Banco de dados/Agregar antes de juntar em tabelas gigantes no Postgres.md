---
tags: [tipo/atomica, camada/padrao, dev/backend, sql]
criado: 2026-07-18
---

# Agregar antes de juntar em tabelas gigantes no Postgres

> Em ranking sobre tabela enorme, agregue primeiro num CTE e só depois junte as tabelas de apoio nos poucos vencedores — não junte linha a linha antes de agrupar.

## O quê

No [[Navetech Hub]], os "top 10 produtos" saem de `lctofissaiproduto` (~47M linhas). Juntar `produto` (~2,8M) e `empresa` a cada linha antes de agrupar seria caríssimo. O padrão que funcionou:

```sql
with topn as (
  select codigoempresa, codigoproduto,
         sum(valortotal) valor, sum(quantidade) qtd
    from lctofissaiproduto
   where datalctofis between $1 and $2   -- usa o índice (empresa, estab, data)
   group by 1, 2
   order by valor desc
   limit 10
)
select t.*, p.descrproduto, e.nomeempresa
  from topn t
  left join produto p using (codigoempresa, codigoproduto)
  left join empresa e using (codigoempresa)
 order by t.valor desc;
```

Agrega os milhões, corta pra 10, e só então faz join — 10 lookups por índice em vez de milhões. Medido: ~2,6s no mês inteiro de todas as empresas; com filtro de uma empresa, ~16ms.

## Por que importa

Regra geral de performance SQL: **reduza a cardinalidade antes de enriquecer**. Join é barato sobre 10 linhas, proibitivo sobre 47M. Vale pra qualquer "top N + nome bonito" sobre fato grande. Depende de conhecer o [[Modelo de dados fiscais do Questor]] e seus índices.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
- Relacionado: [[Modelo de dados fiscais do Questor]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Dados]]
