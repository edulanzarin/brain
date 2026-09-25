---
tags: [tipo/atomica, camada/principio]
criado: 2026-09-25
---

# Código sem cadastro se prova pelo comportamento do dado, não pelo rótulo herdado

> Quando um sistema guarda um código sem tabela que diga o que ele é, o nome que
> alguém escreveu no seu código é um palpite. O que o código é se descobre pelo que
> as linhas com ele FAZEM: valor, espécie, alíquota, presença de campo. Rótulo que
> não passou por isso vira número errado com cara de certo.

## A regra

Antes de dar nome a um código ou de contar por ele, cruze o código com duas ou três
colunas que só se comportam de um jeito para cada significado possível. Se a
impressão digital não bate com o rótulo, o rótulo está errado, mesmo que venha de um
sistema em produção há meses. Código que não se prova fica aparecendo como código na
tela, com a ressalva, até alguém que sabe nomear.

## Por que

O rótulo errado não quebra nada: a consulta roda, o indicador tem número, o gráfico
tem cor. O erro só aparece quando alguém olha o dado por outro ângulo, e até lá ele
já foi para relatório.

Dois casos no mesmo banco:

- **`tipoimposto` da apuração fiscal.** O candidato óbvio para o código 2 era IPI,
  pela ordem numérica. As alíquotas (2 e 3%) e a espécie (só NFS-e) provaram que é
  ISS. Ver [[Apuração fiscal no Questor - periodoapuradofis]].
- **`cdsituacao` da nota fiscal.** O sistema antigo chamava o 5 de denegada, o 6 de
  inutilizada e contava tudo que não fosse 0 como problema. O valor da nota
  desmentiu: o 5 tem valor zero em 100% das linhas e é quase todo NFC-e
  (inutilização de numeração), o 6 tem valor médio de R$ 388 e é CT-e e NF-e
  (complementar, documento regular). O indicador de "denegadas" saía quase quatro
  vezes maior, e o de "sem chave" era 99,7% inutilizada. Ver
  [[cdsituacao do Questor é o COD_SIT do SPED]].

## Na prática

- Procure a coluna que o significado obriga: documento inutilizado não tem valor nem
  chave; imposto de serviço só aparece em nota de serviço; cancelada tem flag
  própria. Uma consulta agrupada por código × essas colunas decide.
- Se o domínio tem um padrão público (SPED, tabela de CFOP, código de país), teste o
  padrão primeiro: sistema que fala com o fisco costuma guardar o código do fisco.
- Rótulo herdado de outro sistema é hipótese, não fonte. Portar a tela é a hora de
  conferir, porque é a hora em que alguém está lendo cada consulta.

## Conexões
- Irmã: [[Rótulo feito de chave técnica aponta para o registro errado quando os dois ids se parecem]] · [[Tirar o dado errado não põe a verdade no lugar]]
- Depende de: [[A tela não afirma mais precisão do que a fonte tem]]
- Visto em: [[NaveX]] · [[Navetech Hub]]
- Mapa: [[Base]]
