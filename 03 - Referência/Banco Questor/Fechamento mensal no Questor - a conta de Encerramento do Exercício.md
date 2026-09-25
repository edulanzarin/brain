---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor, contabil, sql]
criado: 2026-09-08
---

# Fechamento mensal no Questor - a conta de Encerramento do Exercício

> "A empresa fechou o mês" tem um marcador único no Questor: um lançamento na conta de **Encerramento do Exercício** (`classifconta` 7.1.01.001), a apuração do resultado contra Lucros ou Prejuízos do Exercício. Sem ele a empresa pode ter mil lançamentos e não estar fechada; com ele, alguém apurou.

## A conta

| campo | valor |
|---|---|
| `classifconta` | `7.1.01.001` (analítica, `tipoconta = 2`) |
| descrição | "Encerramento do Exercício" |
| conta reduzida | **4855** em 1.363 empresas — e mais 5 códigos noutras |
| natureza | credora (`natursaldo = -1`) |

A árvore em volta: `4852` classe 7 "RESULTADO" · `4853` `7.1` · `4854` `7.1.01`
"RESULTADO DO EXERCÍCIO" (sintética) · `4855` a analítica que recebe o lançamento.

**Resolver pela CLASSIFICAÇÃO, nunca pelo número.** Em uma empresa da base, a
reduzida 4855 é "PARTICIPAÇÕES NOS LUCROS" (`4.3.03.01`) — fixar o código contaria
fechamento onde não houve. A conta `5558` (classe 8, "RESULTADO DO PERÍODO") existe
em 1.362 empresas e **nunca é usada**: zero lançamentos.

## Como o lançamento se parece

Contrapartida `2538` "Lucros do Exercício" (`2.4.13.002.001`) ou `2539`
"(-) Prejuízos do Execício" (`2.4.13.002.002`), origem **`CB`** (digitado na
contabilidade), um por empresa por competência. O complemento é escrito à mão e
varia — "RESULTADO APURADO MES 07/2026", "LUCRO APURADO NO ENCERRAMENTO DO
EXERCICIO - 07/2026", "Encerramento de periodo", "PREJUÍZO DO PERÍODO". Não dá
para casar por texto; quem identifica é a conta.

A competência é a `datalctoctb` (o mês fechado); a `datahoralctoctb` é quando o
fechamento foi REGISTRADO, e as duas ficam longe — ver
[[Produtividade se mede pela hora do registro, não pela data do fato]].

## Volume medido (set/2026)

Empresas com fechamento por competência, escritório inteiro:

| set/25 | out | nov | dez/25 | jan/26 | fev | mar | abr | mai | jun | jul |
|---|---|---|---|---|---|---|---|---|---|---|
| 408 | 266 | 264 | 715 | 504 | 470 | 497 | 409 | 366 | 308 | 207 |

O dezembro alto é o encerramento anual; a queda de maio a julho é o normal do
mês recente ainda não fechado. **1.123 empresas já tiveram algum fechamento** em
toda a história da base — contra ~1.390 ativas.

## O saldo mensal responde rápido e mente

`saldoctbmensal` dá a mesma lista em 0,8 s (13 meses) e bate mês a mês com o razão.
Mas ele agrega movimento de SALDO, e saldo também entra por **implantação**
(`implsaldoctb`, ~12,9 mil linhas em 484 empresas): uma empresa apareceu com
movimento na conta de encerramento em maio/2026 sem nenhum lançamento no razão.
Ver [[Tabela agregada não distingue origem, então não serve de prova de trabalho]].

Para o marcador, use o razão. O saldo mensal serve bem para a pergunta vizinha —
"a empresa escriturou alguma coisa nesta competência?" —, aí descontando as
contas que vieram de `implsaldoctb` no mesmo mês.

**"Teve movimento" não prova que o mês começou.** Dois lançamentos entram no mês
seguinte sem trabalho nenhum dele (medido nas AD3, set/2026):

- **Estorno do saldo negativo no dia 1.** No fim do mês o banco negativo vai
  para o passivo ("Saldo devedor", `2.1.01`); em 01 do mês seguinte o escritório
  estorna: débito em `2.1.01`, crédito no banco `1.1.01`, origem `CB`. Em
  setembro, 66 empresas tinham movimento, quase todas só por isso.
- **Juros de empréstimo lançados adiante.** A mesma parcela já está lançada
  até dezembro (débito em `5.7.11`), então a regra "qualquer conta de resultado"
  também acende mês futuro.

Critérios medidos no escritório inteiro (movimento sem implantação):

| critério | jul/26 | ago/26 | set/26 |
|---|---|---|---|
| qualquer movimento (regra do NaveX hoje) | 529 | 386 | 66 |
| banco `1.1.01` com débito **e** crédito | 454 | 326 | 11 |
| alguma conta de resultado (4, 5 ou 6) | 502 | 345 | 24 |
| receita (4, crédito) e despesa (5, débito) | 372 | 176 | 2 |
| classe 6 com débito e crédito | 0 | 0 | 0 |

A equipe do contábil pediu "débito e crédito dentro do grupo 6" para o
amarelo; a classe 6 é Impostos sobre o Lucro e nunca tem os dois lados, então
o que eles chamam de grupo 6 ainda está por confirmar.

## Custo da consulta

Varrer `lctoctb` pelo par débito/crédito custa 58 s em doze meses (perto do
timeout de 60 s) e cai para **menos de 1 s** com a lista de empresas e o
predicado redundante de contas — ver
[[Predicado redundante é o que faz o índice entrar quando o join casa colunas pareadas]].
A última competência fechada de cada empresa (sem filtro de período) sai em 0,7 s
pelo mesmo caminho, e é o que responde "parada no fechamento desde quando".

## Conexões
- Depende de: [[Módulo contábil do Questor]] · [[Plano de contas padrão do Questor e leitura do balancete]]
- Técnica de consulta: [[Predicado redundante é o que faz o índice entrar quando o join casa colunas pareadas]]
- Armadilha da fonte: [[Tabela agregada não distingue origem, então não serve de prova de trabalho]]
- Quem é o responsável pela empresa: [[API do Acessórias]]
- Visto em: [[Navetech Hub]] (Contábil → Produtividade → Fechamento)
- Mapa: [[Banco Questor]]
