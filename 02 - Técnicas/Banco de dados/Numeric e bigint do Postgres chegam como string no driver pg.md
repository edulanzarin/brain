---
tags: [tipo/atomica, camada/padrao, dev/backend, dados, armadilha, sql]
criado: 2026-08-04
---

# Numeric e bigint do Postgres chegam como string no driver pg

> O `node-pg` entrega `numeric`, `bigint` e afins como **string**, não number —
> pra não perder precisão. Se o código trata como number, quebra em silêncio.
> Quando quiser number de verdade, **casta pra `float8` (double precision)** na
> própria query.

## Onde morde

O caso concreto: converter timestamp pra epoch com `extract(epoch from col) * 1000`.
No Postgres 14+ o `extract` passou a retornar **`numeric`** (era `double precision`),
então o driver devolve uma **string** tipo `"1785867067699.984"`. `Date.now() - epochStr`
até funciona (o JS coage a string pra number), mas `new Date(epochStr)` vira **Invalid
Date** — o bug aparece só na formatação, longe da query que o causou. E o tipo
TypeScript dizia `number` e mentia: tipo estático não valida o que o driver faz em
runtime.

## A saída

Cast explícito pra um tipo que o pg parseia como number:

```sql
(extract(epoch from criado_em) * 1000)::float8 AS criado_em
```

`float8`/`double precision` (OID 700/701) o driver converte pra number; `numeric`
(1700) e `bigint` (20) ele deixa como string de propósito. Vale igual pra
`SUM(numeric)`, `money`, colunas `numeric` e qualquer `bigint`: ou casta na query, ou
converte no JS com `Number(...)` — castar na query mantém a intenção colada ao dado.

## Conexões
- Folha: comportamento do driver, sem princípio que o cubra.
- Visto em: [[Navehub]]
- Mapa: [[Dados]]
