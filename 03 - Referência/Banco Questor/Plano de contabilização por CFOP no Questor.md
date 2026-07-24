---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor, sql]
criado: 2026-07-19
---

# Plano de contabilização por CFOP no Questor

> O Questor já sabe, para cada CFOP de cada empresa, em quais contas contábeis a nota deve cair — débito, crédito, uma linha por tributo. A configuração mora na tabela `cfop`, que **é por empresa + estabelecimento** (não é tabela global de CFOP nacional). Isso permite conferir se a nota foi para a conta CERTA, não só se foi contabilizada.

## A cadeia (verificada jul/2026, empresa 1200)

```
cfop (codigoempresa, codigoestab, codigocfop)
  └─ codigotabctbfis<TRIBUTO>  ─→  tabelactbfis (codigoempresa, codigotabctbfis)
                                     └─ tabelactbfislctoctb (seq)
                                          → naturlctoctb, contactb, regravalorlctoctb
```

1. **`cfop`** — uma linha por (empresa, estab, CFOP). Tem um slot de contabilização por tributo: `codigotabctbfisvlrcontabil` (a mercadoria / valor contábil), `...icms`, `...ipi`, `...iss`, `...pis`, `...cofins`, `...subtribut`, `...funrural`, `...difalfcp`, `...monofasico`, `...credpressn`, e as **retenções** `...irrfret`, `...irpjret`, `...inssret`, `...issqnret`, `...pisret`, `...cofinsret`, `...csllret`. Cada slot aponta uma tabela de contabilização.
2. **`apura<TRIBUTO>`** (`apuraicms`, `apuraipi`, `apurapiscofinsoutros`…) liga/desliga o tributo. Slot com tabela mas `apura='0'` **não gera lançamento** — ignorar, senão vira falso positivo.
3. **`tabelactbfis`** — nome da tabela (ex.: "Compra para Industrialização - a vista", "ICMS Sobre Compras", "14603 RESIDUO DE MADEIRA (CAVACO)"). `formatabctbfis`: 1 = mercadoria, 3 = imposto.
4. **`tabelactbfislctoctb`** — os lançamentos que a tabela gera, um por `seq`:
   - `naturlctoctb`: **1 = débito, -1 = crédito**.
   - `origemcontactb`: **0 = conta fixa** (está em `contactb`); **1 e 2 = conta variável**, resolvida no lançamento (fornecedor/cliente). Variável tem `contactb` nulo — é o "lançamento aberto".
   - `regravalorlctoctb`: fórmula só com `+`/`-` sobre tokens `vlr*` — ex.: `vlrICMS`, `vlrIPI`, `vlrContabil-vlrPISOutros-vlrCOFINSOutros-vlrIPI-vlrICMS`, `vlrContICMS-vlrFunRural`.

`planoespec (codigoempresa, contactb)` dá `descrconta` para mostrar nome em vez de número.

## Validação que fecha

Empresa 1200, CFOP `1556064` ("Compra de material para uso ou consumo - 25204 RESÍDUO DE MADEIRA"), tabela 6054. Config: `D 25204` mercadoria, `C variável` fornecedor, `D 384` PIS, `D 385` COFINS, `D 381` IPI, `D 382` ICMS. Os lançamentos reais no `lctoctb` bateram exatamente — inclusive a ausência de IPI/ICMS nas notas que não tinham esses tributos (lançamento de valor zero simplesmente não é gerado).

## `cfoptabctbfis` NÃO é o caminho

A tabela `cfoptabctbfis` (empresa, estab, cfop → tabela) parece ser a configuração oficial, mas **estava praticamente vazia: 7 linhas no banco inteiro**. `cfgintegractbfiscal` idem (0 linhas). Quem manda de verdade são as colunas `codigotabctbfis*` da própria `cfop`. Ver [[Vínculo nota fiscal e lançamento contábil no Questor]], que antes apostava em `cfoptabctbfis`.

`tabelactbfisopefis` (24k linhas) liga tabela → **operação fiscal**, mas as `operacaofis` são operações de **apuração/EFD** (receita bruta do Simples etc.), não a operação de uma nota. Não serve para conferir nota.

## Nem todo componente vira lançamento na nota

Armadilha que gera falso positivo em massa: **ICMS e IPI de saída são contabilizados na apuração mensal**, não nota a nota. Na empresa 1200 (jun/2026) a conta 2827 ("(-) ICMS") teve **zero** lançamentos com `chaveorigem 'MS…'` e só 2 no mês inteiro — os da apuração, que entram com `codigooriglctoctb='FI'` e `chaveorigem` prefixo **`IM`**. Cobrar esses por nota apontava 1170 de 1186 saídas como erradas.

Regra que resolve: só cobrar conta que **comprovadamente recebe lançamento nota a nota** no período (calibrar pelo que se observa em `lctoctb`). Ver [[Impostos no Questor - onde fica cada um]].

## Conferir valor só em nota de CFOP único

Nota com vários CFOPs não permite atribuir a cada um a sua parcela do tributo — e o `lctofisent.valoripi` do cabeçalho pode não bater com a soma lançada (visto 2x numa nota de 2 CFOPs). Conferir valor só quando a nota tem um CFOP só; nos demais, conferir apenas conta/natureza/ausência.

## "Este CFOP contabiliza?" — aprender do histórico, NÃO da config

Ter slot preenchido (`codigotabctbfis*`) **não** é sinal confiável de que o CFOP gera lançamento — **corrige** o que [[Vínculo nota fiscal e lançamento contábil no Questor]] dizia ("config é exata, CFOP sem slot = não contabiliza. Não precisa de heurística"). Validado na **empresa 103** (mês fechado): CFOP `8000001` **tem** slot mas lançou em só 2 de 29 notas (não contabiliza); `8002022` **não tem** slot e lançou (contabiliza). A config erra dos dois lados.

O sinal confiável é o **histórico**: nos últimos 12 meses, em que fração das notas do CFOP houve lançamento FI. Contabiliza se **≥50%** — separa limpo o compra (85–97%) do retorno/exceção (<10%). O `≥1` não serve (pegaria os 2 lançamentos-exceção do 8000001 e viraria pendência falsa).

Não dá pra olhar **só o mês da tela**: num mês ainda não fechado (0 contabilizações) tudo cai em "não exige", escondendo as pendências reais (era o bug). Por isso o BI **cadastra** esse aprendizado por empresa+estab+cfop (tabela própria `conf_cfop_contabiliza`) e a Conferência lê de lá. **Precedência: override manual (`conf_regra`) > aprendido (12m) > config do Questor.** Semeado na 1ª conferência da empresa; reaprendível pelo botão "Atualizar do histórico".

## Conexões
- Índice: [[Banco Questor]] · Contábil: [[Módulo contábil do Questor]]
- Visto em: [[Navetech Hub]] (abas Configuração e Conferência de Contas)
- Elo nota↔lançamento: [[Vínculo nota fiscal e lançamento contábil no Questor]]
- Fiscal: [[Modelo de dados fiscais do Questor]] · Tributos: [[Impostos no Questor - onde fica cada um]]
- Mapa: [[Banco Questor]]
