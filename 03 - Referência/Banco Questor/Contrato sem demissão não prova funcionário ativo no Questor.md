---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor]
criado: 2026-09-25
---

# Contrato sem demissão não prova funcionário ativo no Questor

> `funccontrato.datadem` nulo quer dizer "ninguém registrou a demissão", e não
> "está trabalhando". A prova de que o contrato está vivo é ter folha calculada
> há pouco tempo. A primeira folha do contrato marca desde quando o Questor
> conhece a história dele.

## O tamanho

Set/2026, escritório inteiro, contratos CLT (`categoria = '01'`) sem demissão:
8.086. Destes, 3,3 mil não tinham nenhuma folha nos 120 dias anteriores.

- Empresa que deixou o escritório: os funcionários continuam sem `datadem`.
- Contrato duplicado: a mesma pessoa (`codigofuncpessoa`), mesma admissão, dois
  `codigofunccontr`, um deles sem folha nunca (visto na 1200).
- Contrato especial com admissão de enchimento (código 900000, admissão
  01/01/1980).

## A consulta

A primeira e a última folha de cada contrato saem de `funcpercalculo` (o cálculo
de um funcionário num período) junto de `periodocalculo`, sem provisão nem
transferência:

```sql
left join (
  select fpc.codigoempresa, fpc.codigofunccontr,
         min(pc.datafinalfolha) primeira, max(pc.datafinalfolha) ultima
    from funcpercalculo fpc
    join periodocalculo pc
      on pc.codigoempresa = fpc.codigoempresa and pc.codigopercalculo = fpc.codigopercalculo
   where pc.codigotipocalc not in (70, 71, 80)
   group by 1, 2
) fo on fo.codigoempresa = f.codigoempresa and fo.codigofunccontr = f.codigofunccontr
-- ativo na referência:
where (f.datadem is null or f.datadem > $ref)
  and (fo.ultima >= $ref::date - 120 or f.dataadm >= $ref::date - 120)
```

A admissão recente entra mesmo sem folha, porque a primeira ainda não foi
calculada. O escritório inteiro roda em cerca de 1 s.

## O horizonte

Período de férias cujo prazo de concessão venceu antes da primeira folha do
contrato no Questor aconteceu fora do banco: não ter `reciboferias` dele não
prova nada. Um funcionário admitido em 1988 aparecia com 33 períodos vencidos
pela regra que parte da admissão.

## Efeito no controle de férias

| | antes | depois |
|---|---|---|
| Férias vencidas (painel, escritório) | 1.469 | 126 |
| A vencer em 120 dias (painel) | 390 | 119 |
| Períodos vencidos (empresa 1200) | 270 | 12 |

## Conexões
- Princípio: [[Falta de registro só prova algo dentro da janela em que a fonte era alimentada]]
- Irmã: [[Módulo de folha e eSocial do Questor]]
- Visto em: [[NaveX]]
- Mapa: [[Banco Questor]]
