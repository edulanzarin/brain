---
tags: [tipo/atomica, camada/principio, armadilha, dev/backend]
criado: 2026-08-25
---

# Diferença entre duas leituras só fala do mundo se o instrumento não mudou

> Comparar a foto de hoje com a de ontem parece medir o que mudou lá fora. Mede duas
> coisas somadas: o que mudou lá fora **e** o que mudou em quem fotografou. E como o
> segundo termo não deixa rastro na foto, o diff atribui ao mundo a mudança que foi sua.

## A regra

Todo dado guardado passou por um extrator — o normalizador, a query, o parser, a
fórmula que resumiu. Esse extrator é parte da medição, não moldura dela. Então:

1. **Carimbe a versão do extrator junto do dado**, no mesmo arquivo, como um número
   inteiro que sobe à mão a cada mudança que altere qualquer valor gravado.
2. **Recuse a comparação através de versões diferentes.** Não é para tentar corrigir,
   nem para comparar só os campos "que não mudaram": é para PULAR, dizendo o motivo.
3. E dentro da mesma versão, **trate mudança em massa como suspeita**, porque
   rebalanceamento geral do mundo é raro e mudança sua de forma é comum, e os dois se
   parecem exatamente igual num diff.

O carimbo tem que ser explícito. Derivar a versão do hash do código é elegante e
errado: mudança que não altera valor nenhum passaria a invalidar todo o histórico.

## Por que

O custo não é um erro visível. É uma **afirmação falsa, datada, com aparência de
apuração** — o pior formato possível, porque quem lê não tem por que duvidar.

Dois casos do mesmo jogo, com um mês de diferença:

- No [[piwdex]], o normalizador deixou de copiar três campos da fonte, e o app inteiro
  passou a calcular com o vazio no lugar deles. Ninguém viu, porque campo opcional
  ausente é o normal. Virou [[Campo que a normalização não copia vira número errado, não erro]].
- No [[piwdex2]], o diff entre dois snapshots de catálogo acusou **481 das 482 espécies
  "mudando de golpe"**. O jogo não tinha tocado em golpe nenhum: era um campo novo
  nascendo na minha ingestão entre uma foto e outra. Publicado, teria virado a manchete
  de um patch que nunca existiu.

A assimetria que decide o desenho: **um patch faltando é um buraco, um patch inventado
é uma mentira.** Os dois custam, e só o segundo destrói a confiança no resto da série —
quem descobre um número inventado tem que reler todos os outros.

## Na prática

- Mudou o extrator? Suba o número. É a única linha de disciplina que o mecanismo exige,
  e esquecê-la troca um erro barulhento (uma comparação pulada, dita no log) por um
  silencioso (uma diferença inventada, publicada com data).
- Guarde o carimbo **no artefato**, não numa tabela ao lado: o dado tem que viajar com
  a própria procedência, senão a cópia perde e o histórico antigo fica inútil.
- Dado sem carimbo (o que já estava lá antes da regra) vale como versão desconhecida, e
  desconhecido não compara — a menos que você confira à mão que aquele arquivo saiu do
  código de hoje, e aí carimbe.
- A mesma armadilha em outras roupas: painel que compara mês a mês depois de a query
  ter mudado; teste de performance contra baseline de outra máquina; A/B em que a
  instrumentação subiu no meio.

## Conexões
- Irmã: [[Fórmula verificada só vale na escala em que foi verificada]] (o mesmo defeito
  no eixo do valor, não no do tempo) · [[Auditar o registro, não só o agregado]]
- Técnica que aplica: [[Diff de catálogo externo carimba a versão do extrator]] ·
  [[Campo que a normalização não copia vira número errado, não erro]]
- Visto em: [[piwdex]] · [[piwdex2]]
- Mapa: [[Base]] · [[Backend]]
