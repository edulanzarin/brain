---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-07-20
---

# Container tem largura máxima e respiro constante

> Conteúdo não encosta na borda e não estica sem limite. O container define até onde
> a tela cresce e quanto de ar sobra em volta.

## A regra

Todo container tem três medidas, e as três saem da escala
([[Escala fechada em vez de valor solto]]):

- **Largura máxima** — a partir dali o container centraliza e o espaço extra vira
  margem, não linha de texto de 2000px. Texto corrido para em ~70 caracteres; tela de
  dados (tabela, dashboard) pode ir bem mais longe, mas ainda tem teto.
- **Respiro interno (padding)** — constante dentro do mesmo nível. Card com 16 de um
  lado e 12 do outro é o erro mais comum e o mais visível.
- **Espaço entre irmãos (gap)** — um só valor por nível de agrupamento. Itens do mesmo
  grupo ficam mais perto entre si do que do grupo vizinho; é isso que faz o olho
  entender o agrupamento sem precisar de linha divisória.

O respiro entre blocos é maior que o respiro dentro do bloco. Quando os dois são
iguais, tudo vira uma massa só e nada parece agrupado.

## Por que

Densidade não se resolve encolhendo fonte — se resolve com espaço bem distribuído.
Layout apertado cansa, layout frouxo faz o olho viajar. Fixar as três medidas por nível
resolve os dois de uma vez e tira a discussão de cada tela nova.

## Na prática

Largura máxima do container geral, do conteúdo de leitura e do modal são três valores
diferentes, e cada um vira token. Os componentes concretos que seguem isso estão em
[[Padrões de componentes de dashboard]].

## Conexões
- Depende de: [[Escala fechada em vez de valor solto]]
- Padrão que aplica: [[Padrões de componentes de dashboard]]
- Irmã: [[Hierarquia por superfície, não por borda]]
- Mapa: [[Base]] · [[Design]]
