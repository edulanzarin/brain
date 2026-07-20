---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor]
criado: 2026-07-18
---

# Canceladas e devoluções no Questor

> Canceladas se identificam por `cancelada='1'` no cabeçalho; devoluções, pelo CFOP (descrição contém "devolu").

## Notas canceladas

- Indicador confiável no cabeçalho: **`cancelada = '1'`**. Excluir dos totais/faturamento.
- `cdsituacao` complementa: canceladas costumam ser `2`. Observado (jun/26, entradas): `0` normal (maioria), `2` cancelada, `8` outra situação (denegada/inutilizada?), `1`/`6` raros.
- **Nota cancelada tem `valorcontabil` ZERADO no cabeçalho.** Então análise de cancelamento é por **contagem/taxa** (canceladas ÷ total de notas), não por valor. Taxa típica < 0,5%.
- Tabelas de **item** não têm `cancelada`; pra excluir canceladas de somas de item, juntar ao cabeçalho por `(codigoempresa, chavelctofis*)` e filtrar lá.

## Devoluções

Notas normais identificadas pelo **CFOP**. Pragmático: juntar em `cfop` e filtrar `descrcfop ilike '%devolu%'`.

- Saída (`lctofissaiproduto`) = devolução de compra: `5202002`/`6202002` (compra p/ comerc.), `5201002` (compra p/ indústria), `5921002`/`6921002` (vasilhame), `5918002` (consignação).
- Entrada (`lctofisentproduto`) = devolução de venda: `1201007`/`2201007`, `2202002` (muito volumosa), `1202002`.
- Ou seja: **devolução de venda entra como nota de entrada**; **de compra sai como saída**. Dependendo da análise, olhar os dois lados.
- `cfop.finalidade` também classifica a operação.

## Espécies (`especienf`)

Vem com espaço/caixa variável — normalizar `upper(btrim(especienf))`. Principais: NFE, CTE, NFSE, NFCE, NF, NF3E, NFCOM, NFST. CTE é volumoso em quantidade, baixo em valor.

## emitentenf

`'P'` = a própria empresa emitiu; `'T'` = terceiro. Em entradas, `'P'` normalmente indica devolução de venda (empresa reemitindo). Ver [[Modelo de dados fiscais do Questor]].

## Conexões
- Visto em: [[Questor BI]]
- Índice do banco: [[Banco Questor]]
- Contexto: [[Modelo de dados fiscais do Questor]] · [[Receitas SQL do Questor]]
- Mapa: [[Banco Questor]]
