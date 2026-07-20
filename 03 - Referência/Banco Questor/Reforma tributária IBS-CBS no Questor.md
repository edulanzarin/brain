---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor, fiscal]
criado: 2026-07-18
---

# Reforma tributária IBS-CBS no Questor

> A base do Questor já está aparelhada para a Reforma Tributária (IBS + CBS, que substituem ICMS/ISS/PIS/COFINS na transição): há tabelas de domínio de CST/classificação, o de-para NCM→tributação por empresa, e os valores de IVA já sendo lançados por item de nota. Em 2026 isso é ano de teste/apuração assistida.

## Domínio — `cstibscbs` (~161)

Tabela de referência do IBS/CBS. PK `(codigocst, codigoclasstrib)`:

- `codigocst` + `descricaocst` — CST do IBS/CBS.
- `codigoclasstrib` + `descricaoclasstrib` — a **classificação tributária** (`cClassTrib`) da nota técnica.
- `reducaoibs`, `reducaocbs`, `reducaobc` — percentuais de redução; `redacao` = texto legal.

## De-para por produto/serviço (por empresa)

- **`ncmrelacibscbs`** (~13,8M!) — relaciona **NCM → CST/`cClassTrib`** por empresa: PK `(codigoempresa, codigoncm, codigocst, codigoclasstrib, sequencia)`; `datafim`, `regraporproduto`, `possuiexcecao`. Enorme porque cruza cada NCM com cada empresa (é a maior tabela "de configuração" da reforma).
- **`nbsrelacibscbs`** (~1,2M) — o equivalente para **serviços** (NBS).
- Versões `*temp` são staging de importação dessas regras.

## Valores lançados por item — `*produtoiva`

- `lctofissaiprodutoiva` (~5,9M) / `lctofisentprodutoiva` (~1,7M) — os **valores de IVA (IBS/CBS)** por item de nota, espelhando `lctofis*produto` do fiscal ([[Impostos no Questor - onde fica cada um]]). É aqui que o imposto novo aparece nota a nota, à medida que as notas de 2026 vão trazendo os campos.

## Por que importa

Qualquer análise/automação fiscal daqui pra frente precisa considerar IBS/CBS. A estrutura já existe: para "imposto novo por nota", olhar `lctofis*produtoiva`; para "como tal NCM é tributado", `ncmrelacibscbs` + `cstibscbs`. Como é transição, tratar os valores como parciais/em teste em 2026 e confirmar preenchimento antes de reportar.

## Conexões
- Índice do banco: [[Banco Questor]] · Convenções: [[Panorama e convenções do banco Questor]]
- Espelha o fiscal: [[Modelo de dados fiscais do Questor]] · [[Impostos no Questor - onde fica cada um]]
- Mapa: [[Banco Questor]]
