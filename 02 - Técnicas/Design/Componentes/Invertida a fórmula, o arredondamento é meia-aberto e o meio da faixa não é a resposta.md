---
tags: [tipo/atomica, camada/padrao, armadilha, dev/frontend]
criado: 2026-08-24
---

# Invertida a fórmula, o arredondamento é meia-aberto e o meio da faixa não é a resposta

> Quando a tela mostra um valor **arredondado** e você inverte a fórmula pra
> descobrir a entrada, o resultado é um intervalo. Duas coisas que parecem
> detalhe e não são: esse intervalo é **meio-aberto**, e o **ponto médio dele não
> é a melhor resposta** quando a entrada só admite inteiros.

## De onde vem o meio-aberto

`Math.round` arredonda meio pra CIMA. Então `stat = round(v)` quer dizer
`v ∈ [stat − 0,5 ; stat + 0,5)` — fechado embaixo, **aberto em cima**. Inverter
devolvendo um par fechado `[lo, hi]` inclui um valor a mais na ponta de cima.

Medido num caso real, a ponta sobrando colocava um inteiro que **não reproduz o
stat observado** dentro da faixa em 3% das leituras. Não quebra nada e não dá
sintoma — só alarga a dúvida que a tela declara.

## O ponto médio erra quando a faixa mede exatamente 1

Esta é a metade cara. O ponto vindo da inversão crua é o meio do intervalo; se o
intervalo tem largura 1, o meio cai em `x,5` — e arredondar `x,5` sobe pra `x+1`,
que é justamente o valor que a fórmula direta **não** reproduz.

Não é canto raro: a largura vale `1 / (2·fator)`, e ela dá exatamente 1 quando o
fator dá 0,5 — um pokémon de nível 50 com quality 1,0, o caso mais banal que
existe. Medido em 57 mil combinações: em 73% delas o stat observado fixa **um
único** IV possível, e em 1% dessas a tela mostrava o inteiro errado, sempre
uma unidade acima.

## A saída não é corrigir o arredondamento — é parar de inverter

Quando a entrada é **inteira e de faixa curta**, inverter é o caminho difícil de
um problema fácil. Enumere:

```ts
const compativeis = []
for (let iv = 0; iv <= IV_MAX; iv++)          // 33 candidatos
  if (statProjetado(base, iv, nivel, q) === statObservado) compativeis.push(iv)
```

Trinta e três avaliações por stat, cento e noventa e oito pela leitura inteira —
irrelevante em qualquer máquina. E o resultado é **exato**: não há ponta aberta
pra errar, o "não fecha" vira `compativeis.length === 0` sem precisar de teste
separado, e a largura da dúvida é a contagem, não uma subtração de floats.

A inversão continua útil pra sugerir o valor quando o espaço de entrada é grande
ou contínuo. Para espaço pequeno e discreto, ela só acrescenta uma classe de erro.

## O que mais vale lembrar

**Verificar ida e volta não pega isto.** Projetar com um IV e conferir que ele cai
dentro da faixa devolvida passou em 14.406 de 14.406 casos — a faixa é generosa,
então o teste sempre passa. O que reprova é o teste ao contrário: *todo inteiro
dentro da faixa reproduz o valor observado, e nenhum de fora reproduz*. Faixa se
testa pelos dois lados, senão só se prova que ela é grande o bastante.

## Conexões
- Princípio: [[A tela não afirma mais precisão do que a fonte tem]]
- Irmã: [[Estimativa que inverte valor arredondado é faixa, não ponto]] ·
  [[Zero na tela é afirmação, não valor de conforto]] ·
  [[Ponto decimal em interface pt-BR afirma outro número]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
