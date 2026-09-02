---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-22
---

# Ponto decimal em interface pt-BR afirma outro número

> `toFixed` devolve ponto. Numa interface em português o ponto é separador de
> **milhar** — então o mesmo número aparece na tela multiplicado por mil, sem erro,
> sem exceção e sem ninguém desconfiar.

## A regra

Todo número que vai pra tela passa por **um** formatador, e ele nasce junto com o
projeto — não no dia em que alguém percebe a divergência:

```ts
export function num(n: number, casas = 1, max = false): string {
  if (!Number.isFinite(n)) return "—";
  return n.toLocaleString("pt-BR", {
    minimumFractionDigits: max ? 0 : casas,
    maximumFractionDigits: casas,
  });
}
```

`toLocaleString` resolve as duas pontas de uma vez: vírgula decimal **e** ponto de
milhar (`1.234,5`). `toFixed` não faz nem uma nem outra.

O sintoma clássico de que isso está solto: **formatadores parciais**. Aparecem funções
que fazem `.replace(".", ",")` no fim — cada uma cobrindo o caso que doeu naquele dia.
Se existem dois desses no projeto, existe um terceiro lugar que ficou sem.

## Por que dói mais que inconsistência de estilo

Não é estética, é **afirmação errada**. Num caso real, a ficha de uma espécie imprimia a
mesma chance de drop duas vezes na mesma tela: o parágrafo derivado dizia `3,4%` e a
tabela logo abaixo dizia `3.400%`. Para um leitor brasileiro, a tabela não estava
"formatada diferente" — ela dizia **três mil e quatrocentos por cento**, num número que
a ferramenta existe pra informar.

E o erro escala com o valor: quanto mais casas decimais, mais convincente fica o número
errado.


## O compacto tem que copiar o espaço do Intl, não inventar o seu

Quando o formatador ganha um irmão compacto para eixo e cartão (`R$ 4,3 mi`), é
natural escrevê-lo com template: `` `R$ ${num(v/1e6,1)} mi` ``. Aí ele passa a
divergir do formatador principal num detalhe invisível: o `Intl` separa `R$` do
número com **espaço inseparável** (`\u00a0`), e o template usa espaço comum.

Não é preciosismo tipográfico — é o que impede a quebra de linha de deixar `R$`
pendurado no fim de uma linha e o valor na seguinte, dentro de um card estreito.
E a divergência é traiçoeira no teste: os dois lados saem **visualmente
idênticos** na mensagem de falha, e se perde um tempo absurdo procurando o que
não está lá. Declare a constante no teste (`const NB = "\u00a0"`) para a
expectativa dizer o que espera.

## Exceção, e ela é estreita

Campo que **copia o formato de um sistema externo** (a Quality que o jogo mostra com
ponto, um código, um identificador) fica como o sistema mostra — a pessoa vai comparar
caractere a caractere com a outra tela. Isso é exceção consciente, e vale comentar no
código; qualquer outro número passa pelo formatador.

## Conexões
- Princípio: [[A tela não afirma mais precisão do que a fonte tem]]
- Irmã: [[Traduza o vocabulário do sistema, não o nome próprio]] ·
  [[Zero na tela é afirmação, não valor de conforto]]
- Visto em: [[piwdex2]] · [[Navetech Hub]]
- Mapa: [[Design]]
