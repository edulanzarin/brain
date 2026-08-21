---
tags: [tipo/atomica, camada/tecnica, dados]
criado: 2026-08-21
---

# Distribuição exata sai de programação dinâmica, não de Monte Carlo

> "Quantos sorteios até somar N?" tem resposta fechada quando os ganhos são poucos e
> comensuráveis. Simular é mais lento, roda a cada tecla e ainda erra na terceira casa.

Forma concreta de [[Custo de processo aleatório se orça pela cauda, não pela média]]:
o princípio manda mostrar a cauda; esta nota é como se calcula a cauda sem simular.

## Quando dá

Quando o sorteio tem **poucos resultados** com probabilidade conhecida e todos são
**múltiplos de um mesmo passo**. É o caso de tabela de drop, de ganho por tier, de
custo por nível — dado de jogo e de sistema de regras costuma ser assim.

O truque é trocar a unidade pelo **mdc dos ganhos**. No breeding do Poke Idle World os
ganhos são 5/10/20/40 milésimos (mdc 5) num modo e 150/200/250/300 (mdc 50) no outro:
dividindo pelo mdc, o problema vira "somar N inteiros pequenos" e o espaço de estados
encolhe na mesma proporção.

## Como

Cadeia de Markov **absorvente**. O estado é quanto já se acumulou (`0..N-1`); chegar em
`N` é a absorção. Uma varredura por passo devolve tudo de uma vez:

```ts
let atual = new Float64Array(need); atual[0] = 1
while (acumulado < 1 - 1e-12) {
  const proximo = new Float64Array(need)
  let fechou = 0
  for (let u = 0; u < need; u++) {
    const p = atual[u]; if (p === 0) continue
    for (const s of passos) {
      const v = u + s.u
      if (v >= need) { fechou += p * s.p; sobra += p * s.p * (v - need) }  // absorveu
      else proximo[v] += p * s.p
    }
  }
  n++; acumulado += fechou; media += n * fechou    // P(N = n) é o que absorveu no passo n
  atual = proximo
}
```

Da mesma varredura saem **mediana, p90, média e a sobra** (o quanto o último sorteio passa
do alvo — que num sistema com teto é exatamente o desperdício). Monte Carlo precisaria de
centenas de milhares de rodadas pra chegar perto disso, e continuaria aproximado.

## Os dois cuidados

1. **Inteiro, nunca float.** Somar `0.005` duzentas vezes dá `1.0000000000000007`; somar
   `5` duzentas vezes dá `1000`. Converta na entrada, volte pra decimal só na saída —
   é o mesmo motivo de [[Zero na tela é afirmação, não valor de conforto]]: erro de
   representação vira número errado, não erro.
2. **Teto de estados, com a saída declarada.** O espaço cresce com o alvo, e campo de
   entrada aceita absurdo. Acima do teto, caia numa aproximação de renovação
   (`N ≈ need/μ`, desvio `√(need·σ²/μ³)`) e **marque o resultado como aproximado na tela**
   — nunca entregue aproximação com cara de conta exata.

## Como se confere

Escrevendo o Monte Carlo **uma vez**, como teste, e comparando com a DP. É a validação
certa: a simulação é fácil de escrever e óbvia de ler, e serve pra provar a versão rápida
que vai pra produção. No piwdex2, 400 mil rodadas bateram p50, p90 e sobra idênticos, com
a média dentro de 0,14% — o resíduo é ruído do Monte Carlo, não erro da DP.

## Conexões
- Princípio: [[Custo de processo aleatório se orça pela cauda, não pela média]]
- Irmã: [[O cálculo puro sai do módulo server-only para poder ser testado]] ·
  [[Estimativa que inverte valor arredondado é faixa, não ponto]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]] · [[Dados]]
