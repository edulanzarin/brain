---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-08-19
---

# Tier é nota com régua fixa, não posição na fila

> Classificar em faixas cortando por **percentil de posição** ("os 10% melhores são S")
> parece tier list e não é: mede fila, não mérito. A faixa tem que sair de um corte de
> **score**, calibrado uma vez sobre a distribuição real e escrito como número.

## O problema

O corte por posição tem tamanho fixo por construção. Sempre existem exatamente 10% em S
— mesmo que o conjunto inteiro seja ruim, mesmo que metade dele seja excelente. Disso
saem dois defeitos:

- **Mérito não sobe.** Um patch que melhora trinta itens não promove nenhum: pra alguém
  subir de faixa, outro tem que descer e abrir vaga. A classificação fica cega
  justamente para a mudança que ela deveria registrar.
- **A faixa não quer dizer nada.** "É tier B" só informa onde o item está na fila
  daquela consulta. Filtrou a lista, mudou a fila; a nota do item deveria ser a mesma.

É a mesma armadilha da curva forçada de avaliação de pessoas, e do benchmark que se
recalcula a cada rodada: régua que se move junto com o medido não mede progresso.

## A solução

Calibrar o corte **uma vez**, com o dado na mão, e deixá-lo escrito:

1. Calcule o score do conjunto todo e monte o **histograma**. Distribuição real quase
   nunca é uniforme — costuma ter vales, e vale é onde o corte deve cair, porque ali a
   fronteira separa dois grupos que já existem em vez de partir um grupo ao meio.
2. Se um **modo de uso** muda a forma da distribuição, calibre um jogo de cortes por
   modo. No piwdex a distribuição com TM é bimodal (o golpe de poder 600 abre o vale) e
   sem TM é um morro só; um corte único deixaria uma das duas telas inútil, com 60% do
   catálogo em duas faixas.
3. Escreva os cortes como constante no código, com o comentário do porquê. A calibração
   é uma decisão datada, não uma função que roda a cada render.

```ts
const TIER_CUTS: Record<Pool, [Tier, number][]> = {
  natural: [["S", 66], ["A", 52], ["B", 45], ["C", 39], ["D", 31], ["E", -1]],
  tm:      [["S", 74], ["A", 65], ["B", 44], ["C", 35], ["D", 27], ["E", -1]],
};
```

## O que mais vale lembrar

- **Recalibrar é manutenção honesta, recalcular é esconder mudança.** Se um patch mexe
  no jogo inteiro, refazer o histograma e mover os cortes é trabalho consciente e
  datado — diferente de deixar a régua deslizar sozinha a cada consulta.
- Mostre o corte na tela ("S = score 66+"). Sem ele o leitor não distingue nota de fila,
  e é a diferença que dá sentido à faixa.
- Sanidade se checa pelos extremos: se o pior conhecido não está na última faixa e o
  melhor conhecido não está na primeira, o problema é o score, não o corte.

## Conexões
- Princípio: [[Ordene pela grandeza que decide, não pela que impressiona]]
- Irmã: [[Bônus multiplicativo só rende onde há folga até o teto]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
