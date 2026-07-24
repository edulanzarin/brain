---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor, sql]
criado: 2026-07-19
---

# Contas bancárias e layout de contabilização no Questor

> A conta contábil de um banco não tem tabela própria: é uma conta comum do `planoespec`, identificada pela classificação `1.1.01.002` (Disponibilidades → Bancos). E o formato do arquivo de importação de lançamentos está descrito em `layoutarqcontabilizacao`, que já vem com layouts cadastrados.

## Onde mora a conta do banco

Em `planoespec` (plano de contas por empresa), no ramo das disponibilidades:

- `1.1.01.001` — Caixa
- `1.1.01.002` — Bancos conta movimento
- `1.1.01.003` — aplicações e afins (também aparece com nome de banco)

Exemplo (empresa 1200): "Banco Viacredi" = `contactb` **16**. O número é **por empresa** — o mesmo banco tem conta diferente em cada uma, então nada de fixar no código.

Filtrar por `tipoconta = 2` (analítica): sintética é agrupadora e não recebe lançamento. Na 1200 são 4723 analíticas contra 367 sintéticas.

`contabancaria` (codigobanco, agência, conta) e `contabancariaempresa` existem mas estavam **quase vazias** (33 e 23 linhas no banco inteiro) e **não** ligam a conta contábil — não servem para descobrir "qual conta contábil é este banco". O vínculo banco↔conta contábil é conhecimento do escritório, não do ERP.

## Formato do arquivo de lançamentos — `layoutarqcontabilizacao`

Descreve arquivos de contabilização, com layout por campo. Colunas úteis: `descrlayout`, `caracterdelimitator`, `naturezalctodebito`/`credito`, `partidadobrada`, `layouttrailler` (a linha de detalhe).

Havia 2 layouts cadastrados. O da Navecon ("Navecon Consistem Ticket 469320"), delimitador `,`:

```
[TextoEmBranco] [pDataLacto dd/mm/yyyy] [ContaDebito] [ContaCredito]
[ValorLcto 2 casas] [CodigoHist] [pCompet mm/yyyy] [CodigoCentroCusto]
```

Ou seja, a ordem de campos é **data · conta débito · conta crédito · valor · histórico · competência · centro de custo**. O token é `[Campo;tipo;tamanho;…]`, com tipo 1=numérico, 2=texto, 3=data, 4=competência, 5=valor com decimais.

**Ressalva**: esse layout é o de *gerar* arquivo (contabilização por outra empresa). Se o objetivo é *importar* no Questor, confirmar com o setor se o importador espera exatamente esse conjunto — a tabela é uma pista forte do padrão usado, não prova.

## Partida dobrada num extrato bancário

- **Recebimento** (dinheiro entra): **débito** na conta do banco, **crédito** na contrapartida.
- **Pagamento** (dinheiro sai): **crédito** na conta do banco, **débito** na contrapartida.

## Não confundir

- `tabelaimportacao` / `tabelaimportacaoregra` **não** são regras de extrato: são regras de PIS/COFINS por CST na importação de nota fiscal (`regrareceita`, `regrabasecalculo`).
- `extratobancarioarquivo` estava vazia; `relacaoextratolcto` (171 linhas) liga extrato a lançamento na conciliação nativa do Questor.

## Conexões
- Índice: [[Banco Questor]] · Contábil: [[Módulo contábil do Questor]]
- Visto em: [[Navetech Hub]] (seção Extratos no módulo Contábil)
- Plano de contas: [[Cadastros centrais do Questor - empresa, estab, pessoa]]
- Contabilização de nota: [[Plano de contabilização por CFOP no Questor]]
- Mapa: [[Banco Questor]]
