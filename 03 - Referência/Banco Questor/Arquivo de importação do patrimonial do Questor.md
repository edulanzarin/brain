---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor]
criado: 2026-09-17
---

# Arquivo de importação do patrimonial do Questor

> Bens do imobilizado entram no Questor por um CSV de nove colunas que o próprio Questor exporta como modelo ("MODELO ARQUIVO PATRIMONIAL"). O que cada coluna vira foi conferido em set/2026 contra uma implantação já importada (empresa 1383), lendo o que ficou em `patbem`, `patbemcontacontabil`, `patcfgbem` e `patenccontacontabil`.

## O formato

```
ITEM;Conta Contábil;Descrição;Quantidade;Valor do Bem;Data Aquisição;Encargo Acumulado;Percentual de Encargo;Data Final
1;1089;NF 285667 FACHINI SA;1;190000,00;19/01/2023;123500,13;20;
```

`;` entre campos, vírgula decimal sem milhar, data `DD/MM/AAAA`, CRLF, **Windows-1252 sem BOM** (o modelo vem assim; UTF-8 vira "DescriÃ§Ã£o").

## Coluna por coluna

| Coluna | O que vai | Onde o Questor grava |
|---|---|---|
| ITEM | `1` em toda linha | nada; `patbem.numerobem` sai em sequência sozinho |
| Conta Contábil | conta reduzida **do bem** (Veículos), nunca a "(-) Deprec. Veículos" | `patbemcontacontabil.contactb` |
| Descrição | texto do bem, até 300 | `patbem.descrisao` |
| Quantidade | `1` | `patbem.quantidade` |
| Valor do Bem | valor do bem | `patbem.valor` e `patbemcontacontabil.valor` |
| Data Aquisição | data do bem, dia inclusive | `patbem.dataaquisicao`, `dataincorporacao` e `patbemcontacontabil.datainicial` |
| Encargo Acumulado | depreciação **acumulada** até a posição, nunca o residual | `patcfgbem.encargoacumulado` |
| Percentual de Encargo | taxa anual como está, `0` inclusive | `patcfgbem.percentualencargo` |
| Data Final | vazio | `patcfgbem.datafinal` nulo |

O que não vem do arquivo:

- `patcfgbem.datainicial` saiu igual em todos os bens: o primeiro dia do mês seguinte à posição do relatório anterior (relatório até 30/04/2026 → 01/05/2026).
- `patenccontacontabil` ganha, por bem, uma linha `tipoencargo = 6` com o acumulado importado (a soma por conta bate com o total de depreciação do relatório) e as parcelas mensais `tipoencargo = 1` que o Questor projeta até zerar o residual. Somadas por conta, as parcelas dão o residual dos bens com taxa; bem com taxa 0 não ganha parcela.

## Os dois erros do arquivo feito à mão

Os dois apareceram no modelo que a contabilidade tinha montado com ajuda de IA: o residual no Encargo Acumulado (são colunas vizinhas no relatório de origem) e a conta de depreciação no lugar da conta do bem. Nenhum dos dois quebra a importação; os dois deixam o patrimônio errado.

## Conexões
- Módulo: [[Módulo patrimonial do Questor]]
- Irmã: [[Layouts de importação de lançamento contábil no Questor]]
- Técnica: [[Para alimentar o ERP, gere o arquivo de importação dele]]
- Visto em: [[Navetech Hub]] (Implantação do patrimonial)
- Mapa: [[Banco Questor]]
