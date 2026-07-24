---
tags: [tipo/atomica, camada/padrao, dev/backend, sql]
criado: 2026-07-22
---

# Estoque e fluxo numa série a partir de datas de início e fim

> Uma tabela onde cada linha tem uma data de início e (talvez) uma de fim — contrato, assinatura, locação, bem — responde tanto **fluxo** (quantos entraram/saíram num intervalo) quanto **estoque** (quantos estavam ativos numa data), sem tabela de saldo. Fluxo compara a data contra o intervalo; estoque testa "começou até a data e ainda não terminou".

## As duas contas

Com colunas `inicio` (sempre preenchida) e `fim` (nula enquanto ativo):

- **Fluxo num intervalo** — entradas = linhas com `inicio between $a and $b`; saídas = `fim between $a and $b`.
- **Estoque numa data D** — ativos em D = `inicio <= D and (fim is null or fim >= D)`. É consequência do fluxo, não um número guardado ([[Balancete é movimento do período, saldo é consequência]]).

## A série mensal (densa, sem buraco)

Para a série no tempo, gere os baldes com `generate_series` e cruze com os fatos; `count(*) filter (...)` faz cada balde escolher as linhas que lhe pertencem, e o `left join ... on true` garante que **mês sem nada ainda apareça** (com zero), em vez de sumir da série:

```sql
with m as (
  select gs::date as ini,
         (gs + interval '1 month' - interval '1 day')::date as fim
    from generate_series(date_trunc('month', $1::date),
                         date_trunc('month', $2::date),
                         interval '1 month') gs
),
f as ( select inicio, fim from fato where <recorte> )
select to_char(m.ini,'YYYY-MM-DD') as mes,
       count(*) filter (where f.inicio between m.ini and m.fim)                         as entrou,
       count(*) filter (where f.fim    between m.ini and m.fim)                         as saiu,
       count(*) filter (where f.inicio <= m.ini and (f.fim is null or f.fim >= m.ini))  as estoque_ini,
       count(*) filter (where f.inicio <= m.fim and (f.fim is null or f.fim >= m.fim))  as estoque_fim
  from m left join f on true
 group by m.ini order by m.ini;
```

O produto cartesiano `m × f` só compensa quando o fato é pequeno (dezenas de milhares) — aí é trivial e cabe numa consulta só. Em fato de milhões, agregue antes ([[Agregar antes de juntar em tabelas gigantes no Postgres]]).

## Por que importa

Vale para qualquer métrica de rotatividade/ocupação sobre registros com vigência. No [[Navetech Hub]] é o turnover da Folha: `funccontrato` tem `dataadm`/`datadem`, o efetivo numa data é "admitido e ainda não desligado", e a série mensal alimenta o índice ((admissões+desligamentos)/2 ÷ efetivo médio).

## Conexões
- Princípio: [[Balancete é movimento do período, saldo é consequência]]
- Relacionado: [[Agregar antes de juntar em tabelas gigantes no Postgres]]
- Visto em: [[Navetech Hub]] (Folha — Rotatividade)
- Mapa: [[Dados]]
