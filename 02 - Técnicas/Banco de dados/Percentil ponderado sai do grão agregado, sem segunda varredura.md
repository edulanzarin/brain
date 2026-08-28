---
tags: [tipo/atomica, camada/padrao, dev/backend, sql]
criado: 2026-08-27
---

# Percentil ponderado sai do grão agregado, sem segunda varredura

> Se a varredura já devolve o grão como (valor, quantas vezes), a mediana e o p90 saem dele em memória. Pedir `percentile_disc` ao banco significa varrer o fato de novo, ou carregar linha a linha só para ordenar.

## O problema

Tela de atraso contábil: para cada lançamento, a distância entre a data do fato e o
carimbo do registro. A tela quer a mediana e o p90 do escritório, e também os de cada
pessoa, de cada empresa e de cada dia do período.

`percentile_disc` é função de agregação ordenada: ele precisa das linhas, não do
agregado. Sobre 2 milhões de lançamentos por mês, pedir os percentis por pessoa **e**
por empresa **e** por dia é varrer o fato uma vez para cada eixo — e o grão do painel,
que já está pronto, fica sem uso.

## A solução

O grão já é uma distribuição: cada linha diz "este atraso aconteceu N vezes". Basta
somar as contagens até cruzar a fração.

```sql
select codigousuario u, codigoempresa e,
       (datahoralctoctb::date - datalctoctb) lag,
       count(*)::int n
  from lctoctb
 where datahoralctoctb >= $1::date and datahoralctoctb < ($2::date + 1)
 group by 1, 2, 3
```

```ts
/** Percentil de uma distribuição PONDERADA (valor → quantas vezes ocorre). */
function percentilPonderado(pesos: Map<number, number>, p: number): number | null {
  let total = 0;
  for (const n of pesos.values()) total += n;
  if (total === 0) return null;
  const alvo = total * p;
  let acumulado = 0;
  for (const valor of [...pesos.keys()].sort((a, b) => a - b)) {
    acumulado += pesos.get(valor) ?? 0;
    if (acumulado >= alvo) return valor;
  }
  return null;
}
```

Uma varredura (~2,8 s para o mês do escritório, 38 mil linhas de grão) responde a
mediana global, a de cada uma das 40 pessoas, a de cada empresa e a de cada dia — todas
exatas, não estimadas. Confere com `percentile_disc` rodado à parte sobre as mesmas
linhas.

## O que mais vale lembrar

- **A grandeza medida entra no grão, em unidade inteira e fina.** Guardar o mês da
  competência em vez do dia teria devolvido a mediana arredondada; o dia custou o mesmo
  e devolveu o número exato.
- **Sem amostra, devolva `null`, não zero** — dia sem lançamento não tem mediana, e um
  zero ali desenha uma linha no gráfico afirmando "atraso zero". A série liga os pontos
  com `connectNulls={false}` justamente por isso.
- A média não substitui: um lote de importação de dois anos atrás a leva embora. Ver
  [[A régua sai da distribuição, não dos extremos]].
- Só vale enquanto o valor for de baixa cardinalidade (dias, minutos, faixas). Para
  valor contínuo (moeda com centavos), o grão explode e o cálculo volta pro banco.

## Conexões
- Princípio: [[A régua sai da distribuição, não dos extremos]]
- Irmã: [[Grão fino numa varredura só dispensa os count distinct]]
- Irmã: [[Agregar antes de juntar em tabelas gigantes no Postgres]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Dados]]
