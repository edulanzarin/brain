---
tags: [tipo/atomica, camada/referencia, dev/backend, sql, banco/questor]
criado: 2026-07-18
---

# Receitas SQL do Questor

> Consultas prontas e testadas (jun/2026). Ajustar datas/empresas. Conexão read-only — ver [[Questor - conexão read-only e regras]].

## Faturamento e quantidade (saídas, sem canceladas)

```sql
select coalesce(sum(valorcontabil), 0) as valor, count(*) as qtd
  from lctofissai
 where datalctofis between '2026-06-01' and '2026-06-30'
   and cancelada <> '1';
```

Trocar `lctofissai`→`lctofisent` p/ entradas. Empresa: `and codigoempresa = any('{1200,57}'::int[])`.

## Todos os impostos do período (saídas)

Some cada fonte em paralelo (3 scans). Base = faturamento dos itens.

```sql
-- item (ICMS/IPI/ST/ISS)
select sum(valoricms) icms, sum(valoripi) ipi, sum(valorsubtribut) st,
       sum(valoriss) iss, sum(valortotal) base
  from lctofissaiproduto
 where datalctofis between '2026-06-01' and '2026-06-30';
-- PIS/COFINS
select sum(valorpis) pis, sum(valorcofins) cofins
  from lctofissaipiscofins
 where datalctofis between '2026-06-01' and '2026-06-30';
-- retenções (serviço)
select sum(valorirrf) irrf, sum(valorinss) inss, sum(valorcsll) csll, sum(valorissqn) issqn
  from lctofissairetido
 where datalctofis between '2026-06-01' and '2026-06-30';
```

## Top produtos (agrega-e-junta — padrão pra tabela gigante)

```sql
with topn as (
  select codigoempresa, codigoproduto, sum(valortotal) valor, sum(quantidade) qtd
    from lctofissaiproduto
   where datalctofis between '2026-06-01' and '2026-06-30'
   group by 1, 2 order by valor desc limit 10
)
select t.*, p.descrproduto, p.unidademedida, e.nomeempresa
  from topn t
  left join produto p using (codigoempresa, codigoproduto)
  left join empresa e using (codigoempresa)
 order by t.valor desc;
```

## Distribuição por UF da contraparte

Usa `pessoa.siglaestado` (UF do cabeçalho vem vazio):

```sql
select coalesce(p.siglaestado, '—') uf, sum(f.valorcontabil) valor, count(*) qtd
  from lctofissai f
  left join pessoa p on p.codigopessoa = f.codigopessoa
 where f.datalctofis between '2026-06-01' and '2026-06-30' and f.cancelada <> '1'
 group by 1 order by valor desc;
```

## Devoluções por CFOP

```sql
select f.codigocfop, max(cf.descrcfop) descr,
       count(distinct f.chavelctofissai) notas, sum(f.valortotal) valor
  from lctofissaiproduto f
  join cfop cf on cf.codigoempresa = f.codigoempresa
              and cf.codigoestab = f.codigoestab and cf.codigocfop = f.codigocfop
 where f.datalctofis between '2026-06-01' and '2026-06-30'
   and cf.descrcfop ilike '%devolu%'
 group by 1 order by valor desc;
```

## Impostos ao longo do tempo (série)

Mesmo scan do card, com `group by` no dia/mês. Sobre `lctofissaiproduto` (47M) é pesado — filtrar período curto ou empresa.

```sql
select to_char(datalctofis,'YYYY-MM-DD') dia,
       sum(valoricms) icms, sum(valorsubtribut) st, sum(valoripi) ipi, sum(valoriss) iss
  from lctofissaiproduto
 where datalctofis between '2026-06-01' and '2026-06-30'
 group by 1 order by 1;
```

## Empresas com movimento no período

```sql
select count(distinct codigoempresa) from (
  select codigoempresa from lctofisent where datalctofis between '2026-06-01' and '2026-06-30'
  union all
  select codigoempresa from lctofissai where datalctofis between '2026-06-01' and '2026-06-30'
) t;
```

## Listar notas brutas (explorador, paginado)

Contraparte via `pessoa`, empresa via `empresa`. `chavelctofissai` é `bigint` (mandar como texto pro front). Ordena por data desc, pagina com limit/offset. Rápido dentro de um mês (~0,5s até todas as empresas).

```sql
select f.codigoempresa, e.nomeempresa, f.chavelctofissai::text chave,
       f.numeronf, f.serienf, upper(btrim(f.especienf)) especie, f.cdmodelo,
       f.datalctofis, p.nomepessoa contraparte, p.inscrfederal doc, p.siglaestado uf,
       f.valorcontabil, (f.cancelada='1') cancelada, f.chavenfesai
  from lctofissai f
  left join pessoa p on p.codigopessoa = f.codigopessoa
  left join empresa e on e.codigoempresa = f.codigoempresa
 where f.datalctofis between '2026-06-01' and '2026-06-30'
 order by f.datalctofis desc, f.numeronf desc
 limit 50 offset 0;
```

Busca: número → `and f.numeronf = $n`; nome → `and p.nomepessoa ilike '%texto%'`.

## Itens de uma nota (drill-down)

```sql
select f.seq, f.codigoproduto, pr.descrproduto, f.codigocfop, cf.descrcfop,
       f.unidademedida, f.quantidade, f.valorunitario, f.valortotal, f.valoricms, f.valoripi
  from lctofissaiproduto f
  left join produto pr on pr.codigoempresa=f.codigoempresa and pr.codigoproduto=f.codigoproduto
  left join cfop cf on cf.codigoempresa=f.codigoempresa and cf.codigoestab=f.codigoestab and cf.codigocfop=f.codigocfop
 where f.codigoempresa = $1 and f.chavelctofissai = $2
 order by f.seq;
```

## Conexões
- Visto em: [[Questor BI]]
- Índice do banco: [[Banco Questor]]
- Contexto: [[Modelo de dados fiscais do Questor]] · [[Impostos no Questor - onde fica cada um]] · [[Canceladas e devoluções no Questor]]
- Mapa: [[Banco Questor]]
