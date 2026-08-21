---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-21
---

# Estimativa que inverte valor arredondado é faixa, não ponto

> Toda calculadora que descobre um parâmetro escondido faz a mesma coisa: pega o valor
> que o sistema MOSTRA e inverte a fórmula. Só que o valor mostrado quase sempre já veio
> arredondado — e a inversão de um arredondamento não devolve um número, devolve um
> intervalo.

## O problema

A fórmula direta é `mostrado = round(f(escondido))`. Inverter dá `escondido = f⁻¹(mostrado)`,
que parece exato e não é: `mostrado` representa qualquer valor em `[mostrado−0,5,
mostrado+0,5)`, então o escondido cabe em toda a faixa `[f⁻¹(m−0,5), f⁻¹(m+0,5))`.

A largura dessa faixa **não é constante**. Ela é inversamente proporcional à derivada de
`f` — e quando `f` tem um multiplicador pequeno, meia unidade do mostrado vale dezenas do
escondido. Medido num caso real: com multiplicador alto a faixa era de 0,3; com
multiplicador baixo, de 9 — o mesmo número na tela era compatível com 4 e com 30.

Dois estragos, e o segundo é pior:

1. A tela imprime "17,3" e afirma duas casas que o dado não tem.
2. A **validação** passa a acusar entrada boa. Quem valida `ponto > máximo` reprova todo
   caso de multiplicador baixo, porque ali o ponto estoura o teto por arredondamento e
   não por erro do usuário.

## A solução

```ts
/** Intervalo compatível com o valor observado — o arredondamento é do sistema. */
function faixa(mostrado: number, f: number, base: number): [number, number] {
  return [((mostrado - 0.5) / f - base) / k, ((mostrado + 0.5) / f - base) / k];
}
```

- **A barra desenha a FAIXA**, não o ponto: começa em `lo`, termina em `hi`. É o que faz
  a leitura ruim parecer ruim.
- **O número escrito segue a largura**: faixa estreita mostra o ponto com uma casa;
  faixa larga mostra `25–32`. Uma regra só, sem o usuário escolher.
- **Validar pela faixa, nunca pelo ponto**: impossível é `lo > máximo || hi < mínimo` —
  ou seja, nenhum valor válido cabe na leitura.
- **Faixa larga é informação, não erro.** Diga o porquê e o que resolve: "no nível 5 o
  arredondamento cabe em 9 pontos; suba de nível e volte aqui".
- Quando a leitura é impossível, **suprima o que se derivou dela**. Uma nota presa no
  teto anunciando "100%" logo abaixo do aviso de que os números não fecham é pior que
  nenhuma nota.

## O que mais vale lembrar

Isso vale sempre que se reconstrói um parâmetro a partir de valor exibido: preço unitário
a partir de total arredondado, taxa a partir de percentual com uma casa, peso a partir de
balança que arredonda. A pergunta é **"o que eu estou invertendo já passou por um
round?"** — e a resposta é quase sempre sim.

## Visto em

No piwdex2 a calculadora estima o IV de um pokémon a partir dos stats que o jogo mostra.
Um Electrode nível 5 com IV 32 de verdade estima 34,3 no ponto — e a primeira versão da
validação acusou o pokémon de impossível por causa disso.

## Conexões
- Irmã: [[Peça o que a fonte mostra, não o que você precisa]] — quem deriva o valor herda a faixa dele, e a faixa tem que atravessar as telas seguintes.
- Princípio: [[A tela não afirma mais precisão do que a fonte tem]]
- Irmã: [[Zero na tela é afirmação, não valor de conforto]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
