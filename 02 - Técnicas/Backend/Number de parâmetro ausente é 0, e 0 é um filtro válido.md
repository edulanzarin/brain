---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-09-25
---

# Number de parâmetro ausente é 0, e 0 é um filtro válido

> `Number(searchParams.get("respId"))` com o parâmetro ausente dá `Number(null)`, que é `0`, e `Number.isInteger(0)` é verdadeiro. O filtro opcional que ninguém escolheu vira `resp_id = 0` e a lista some.

## O problema

A rota da fila de obrigações lia os filtros opcionais assim:

```ts
const inteiro = (k: string) => {
  const v = Number(q.get(k));
  return Number.isInteger(v) ? v : undefined;
};
```

A intenção era "número inteiro ou nada". Mas `get` devolve `null` quando o parâmetro não veio, e `Number(null)` é `0`. O `undefined` que desligaria o filtro nunca aparecia: sem responsável escolhido, a consulta ganhava `and resp_id = 0` e a fila vinha praticamente vazia, com cara de "escritório em dia". Esteve em produção sem ninguém notar, porque o sintoma (menos linhas) é indistinguível de "pouco trabalho".

`Number("")` também é `0`, então `?respId=` quebra igual.

## A solução

Decidir a ausência ANTES de converter:

```ts
const inteiro = (k: string) => {
  const bruto = q.get(k);
  if (bruto == null || bruto.trim() === "") return undefined;
  const v = Number(bruto);
  return Number.isInteger(v) ? v : undefined;
};
```

## O que mais vale lembrar

- A armadilha só morde quando o zero é um valor legal do domínio (id, quantidade, dias). Onde zero é impossível, ela vira um filtro que não acha nada, que é o pior jeito: silencioso.
- O mesmo vale para `parseInt` com fallback `|| 0` e para `Boolean("false")`. Coerção de parâmetro opcional sempre separa "não veio" de "veio vazio" de "veio o valor".
- Teste de filtro opcional precisa do caso "sem o parâmetro" conferindo que linhas com o campo preenchido aparecem, e não só o caso "com o parâmetro".

## Conexões
- Princípio: [[Ausência de leitura cai no valor que dispara a ação]]
- Irmã: [[Campo que a normalização não copia vira número errado, não erro]] · [[Filtro transversal só é honesto se todo o funil o honra]]
- Visto em: [[NaveX]] (a fila do Obrigações, herdada do nexo2)
- Mapa: [[Backend]]
