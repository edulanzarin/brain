---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor, armadilha, sql]
criado: 2026-09-21
---

# Opção pelo Simples Nacional no Questor é vigência, não campo de regime

> Não existe coluna de regime tributário em `empresa` nem em `estab`. Quem responde é `opcaossimplesfederal`, um histórico de marcos por empresa — e a linha de saída marca o ÚLTIMO dia apurado, não o primeiro dia fora.

## Onde está

`opcaossimplesfederal` — 862 linhas cobrindo 663 empresas (set/2026), de 2018 até hoje.
Uma linha por marco, não por empresa:

- `codigoempresa`
- `datassimplesfederal` — a data em que o marco passa a valer
- `apurassimplesfederal` — `'1'` passa a apurar, `'0'` deixa de apurar

O resto da linha é configuração da apuração (anexo, ICMS/ISS fixo, fator R, código de
acesso do e-CAC). Quem só quer saber "é Simples?" usa as três colunas acima.

A tabela irmã `opcaosimplesfederal` (um `s` só) é o Simples **Federal** antigo e está
**vazia** — não confundir na hora de escrever o join.

## As duas armadilhas

### A saída cai no último dia apurado

Verificado no banco: das 708 linhas de entrada (`'1'`), **708 caem no dia 1** do mês;
das 154 de saída (`'0'`), **154 caem no último dia**. Ou seja, a data da saída ainda é
mês de Simples.

Exemplo real: a empresa 995 (SCR MIDIAS) tem saída em 30/06/2026 e **apurou junho** —
`totalssimplesfederal` tem a competência 2026-06-01 para ela. Tratar a data da saída
como "primeiro dia fora" perde a última competência de toda empresa que saiu.

### O corte é a competência, não a data do documento

Consequência da anterior: uma nota de 30/06/2026 é de empresa do Simples mesmo havendo
um marco de saída com essa mesma data. Por isso o teste compara sempre o **dia 1 do mês
da nota**, nunca `datalctofis` cru.

## A vigência em intervalos

Vira intervalo fechado com `lead()`. Detalhe que importa: a janela de um marco de
entrada seguido de OUTRA entrada (troca de configuração) tem de fechar na véspera,
senão os intervalos se sobrepõem num dia e a nota daquele mês é contada duas vezes num
join.

```sql
with marcos as (
  select codigoempresa,
         datassimplesfederal as inicio,
         apurassimplesfederal as apura,
         lead(datassimplesfederal) over w as prox_data,
         lead(apurassimplesfederal) over w as prox_apura
  from opcaossimplesfederal
  window w as (partition by codigoempresa order by datassimplesfederal)
)
select codigoempresa, inicio,
       case when prox_data is null then date '2100-12-31'
            when prox_apura = '0' then prox_data   -- saída: é o último dia apurado
            else prox_data - 1 end as fim          -- outra entrada: fecha na véspera
from marcos
where apura = '1'
```

Verificado: zero pares de intervalos sobrepostos na mesma empresa.

## Calibração contra a apuração real

`totalssimplesfederal` guarda o que o escritório efetivamente apurou (receita, base,
DAS), uma linha por empresa, competência e operação fiscal. É o gabarito natural: se a
empresa apurou o Simples naquele mês, ela era do Simples naquele mês.

Das **6.134** competências apuradas desde 2024, a regra acima casa **6.128**. As 6
restantes são 2 empresas que apuraram antes de ter marco cadastrado — buraco de
cadastro, não erro da regra. **Nenhuma competência apurada foi classificada como
"fora"**, que é o erro que importaria.

Ordem de grandeza, ago/2026: **464 empresas ativas no Simples** de 1.445 na carteira
(ativa = `estab.dataencerativ > current_date`, ver [[Cadastros centrais do Questor - empresa, estab, pessoa]]).

## Por que importa

É o recorte de qualquer relatório "só as empresas do Simples". E como a vigência varia
no meio do ano, o recorte não é uma lista fixa de empresas: é um par (empresa, mês) —
quem saiu em março entra com as notas até março e some depois, dentro do MESMO período
consultado.

## Conexões
- Ver também: [[Modelo de dados fiscais do Questor]] · [[Apuração fiscal no Questor - periodoapuradofis]]
- Cadastros: [[Cadastros centrais do Questor - empresa, estab, pessoa]]
- Impostos: [[Impostos no Questor - onde fica cada um]]
- Visto em: [[Navetech Hub]]
- Índice do banco: [[Banco Questor]]
- Mapa: [[Banco Questor]]
