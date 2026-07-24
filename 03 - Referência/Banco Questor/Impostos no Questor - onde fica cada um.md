---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor]
criado: 2026-07-18
---

# Impostos no Questor - onde fica cada um

> No Questor não há uma coluna única de imposto — cada tributo mora numa tabela de detalhe diferente. Mapa verificado (jun/2026).

## Impostos de item — `lctofis{ent,sai}produto`

Uma linha por produto; some as colunas:

| Imposto | Coluna valor | Base |
|---|---|---|
| ICMS | `valoricms` | `basecalculoicms` |
| IPI | `valoripi` | `basecalculoipi` |
| ICMS-ST | `valorsubtribut` | `basecalculosubtribut` |
| ISS | `valoriss` | `basecalculoiss` |

Também `aliqicms`, `aliqipi` etc. Faturamento base = `sum(valortotal)`.

## PIS / COFINS — `lctofis{ent,sai}piscofins`

Uma linha por produto, **separado** do produto (scan próprio): `valorpis`, `valorcofins`,
`aliqpis`, `aliqcofins`, `basecalculopiscofins`, `cdsituatributpis`/`cdsituatributcofins` (CST).

## Retenções (serviço) — `lctofis{ent,sai}retido`

Tipicamente NFS-e, uma linha por nota:

| Retenção | Coluna |
|---|---|
| IRRF | `valorirrf` |
| INSS | `valorinss` |
| CSLL | `valorcsll` |
| ISSQN | `valorissqn` |
| IRPJ | `valorirpj` |
| PIS/COFINS retidos | `valorpis` / `valorcofins` |

Tem datas de previsão/pagamento (`datapgto...`) — útil pra automação de guias.

## DIFAL / FCP — `lctofissaidifal`

`vlricmsintufdest` (ICMS destino), `vlricmsintufrem` (remetente), `vlricmsfcpufdest` (FCP), `siglaestadodest`.

## FUNRURAL — `lctofissaifunrural`

`valorfunrural`, `basecalculofunrural` (existe, zerado na base atual).

## Genérica por CFOP — `lctofis{ent,sai}cfop`

Resumo por CFOP + `tipoimposto` (`valorimposto`, `basecalculoimposto`, `aliqimposto`).
`tipoimposto`: **1 = ICMS** (bate com `sum(valoricms)`), **2 = IPI**. Útil pra imposto por CFOP sem varrer itens.

## Apuração no banco (nota)

Não há um resultado de apuração pronto e simples de ler: existem dezenas de tabelas de
config/ajuste (`configsubapuracao*`, `infadicapuracaoicms<uf>`, `cfgimpspedfis*`, blocos
`sped*`), mas montar a apuração oficial (E100/E110, com ajustes/estornos) é trabalhoso. Uma
estimativa gerencial seria débito (impostos das saídas) − crédito (das entradas) — mas é só
estimativa; o Questor BI decidiu **não** exibir apuração pra não passar número que não é oficial.

## Outras (a documentar)

`lctofissaisubtribut` (ST detalhado: próprio, retido antecipado, FCP-ST), `lctofis*retif` (retificações), `sped*` (blocos do SPED).

## Conexões
- Visto em: [[Navetech Hub]]
- Índice do banco: [[Banco Questor]]
- Estrutura geral: [[Modelo de dados fiscais do Questor]]
- Consultas prontas: [[Receitas SQL do Questor]]
- Mapa: [[Banco Questor]]
