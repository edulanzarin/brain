---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-21
---

# Zero na tela é afirmação, não valor de conforto

> Imprimir `0` diz "nenhum". Duas coisas diferentes chegam a esse zero sem que ninguém
> tenha afirmado nada: um valor pequeno demais que o arredondamento comeu, e um campo
> que a fonte deixou zerado porque não publica o número. Nos dois casos a tela passa a
> afirmar, com a autoridade de um número, algo que ninguém verificou.

## O problema

Zero é o único número que também é a cara da ausência. Isso o torna o destino natural de
dois acidentes distintos:

**Arredondamento.** `toFixed(2)` numa porcentagem transforma 0,00001% em "0,00%". O
formatador foi calibrado nos valores típicos e o dado raro — que costuma ser exatamente
o que a ferramenta existe pra mostrar — sai da tela sem aviso. Num catálogo de drops,
44 das 2.657 linhas caem abaixo de 0,01%: a chance minúscula existe, é rastreável, e é o
número que ninguém mais publica. Arredondar apaga justamente a parte cara.

**Zero herdado da fonte.** Um campo numérico obrigatório é preenchido com `0` quando o
sistema de origem não tem o valor. Ali `0` não quer dizer "nunca acontece", quer dizer
"não digo". Derivar em cima disso (média, taxa, "quantos até sair um") produz `Infinity`,
`NaN` ou — pior — um número plausível, que ninguém desconfia.

O sintoma é o mesmo dos dois lados: nada quebra, nada avisa, e a tela mente com cara de
precisão.

## A solução

Separar as três situações no formatador, e não na cabeça de quem lê:

```ts
function pct(p: number): string {
  if (p <= 0) return "0%";            // zero LITERAL da fonte: fiel, e explicado à parte
  if (p < 0.0001) return "<0,0001%";  // existe, mas abaixo da resolução legível
  // casas crescem conforme o valor encolhe
  const s = p < 0.01 ? p.toFixed(4) : p < 1 ? p.toFixed(3) : p.toFixed(2);
  return `${s.replace(".", ",").replace(/,?0+$/, "")}%`;
}
```

- **Valor real nunca vira zero.** Ou ganha casas, ou vira `<limiar` — o operador `<` é
  honesto: diz que existe e que é menor do que dá pra escrever.
- **Zero da fonte permanece zero na tabela** (é o que a fonte diz), mas **nenhuma
  derivação sai dele**: onde apareceria "0 abates por unidade" ou "0 de retorno", vai um
  traço.
- **O caso ganha frase, não número.** Quando o zero é da fonte, alguma tela precisa
  dizer em português o que ele significa: "a fonte lista o drop e declara a chance como
  0 — o valor existe, mas não é publicado". Estado tem visual próprio; não se pede
  emprestado o visual de um valor.

## O que mais vale lembrar

A pergunta que revela o problema é **"qual é o menor valor não-nulo do conjunto, e como
ele aparece na tela?"**. Se aparece como zero, o formatador está apagando dado.

E a irmã dela: **"esse zero foi medido ou herdado?"**. Fonte externa raramente distingue
`0` de `null`, então a distinção tem de ser reconstruída na borda — antes que qualquer
média, taxa ou ranking a consuma.

## Visto em

No piwdex2, a página de itens é um índice reverso de drop: para cada item, quem dropa e
com que chance real. Os dois acidentes apareceram no mesmo dia. O Corsola dropa Air Tank
a 0,00001% e a tabela imprimia "0%" ao lado de "10.000.000 abates por unidade" — duas
células se contradizendo. E 346 linhas do catálogo (todas de Strange Pheromone, o item
de breeding) vêm com `chance: 0`, o que fazia a ficha do item mais dropado do jogo
oferecer uma "melhor fonte" com zero por cento e duas derivações vazias. A correção foi
formatar por faixa e trocar a derivação por uma frase que diz que o número não é
publicado.

## Conexões
- Princípio: [[Todo estado da tela tem visual]]
- Irmã: [[Zero num medidor é estado, não barra vazia]] · [[A régua de um medidor é percentil, não máximo]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
