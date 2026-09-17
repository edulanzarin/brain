---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-09-17
---

# Série que tem cor no catálogo recebe a cor, não a posição na paleta

> O componente de gráfico pinta cada série pela ORDEM: primeira série, primeira
> cor da paleta. Quando a categoria já tem cor declarada num catálogo, e a
> legenda e a barra de composição leem essa cor, o gráfico precisa receber a
> mesma cor explícita. Senão a legenda mente a partir do primeiro item fora da
> ordem.

## O problema

Um catálogo de classes (digitado, importado, integrado, apuração, outras
origens) declara `cor` em cada uma. A legenda e a faixa de composição usam essa
`cor`. O gráfico de série recebia só `{ chave, rotulo }` e escolhia pela
posição.

Enquanto as classes coincidem com a paleta em ordem (`serie-1`, `serie-2`...),
nada aparece. O quinto item, "outras origens", é cinza no catálogo de propósito
(a cauda não compete com as classes) — e no gráfico saiu na quinta cor da
paleta. Num catálogo com duas classes cinzas, a segunda e a terceira da cauda
ficaram com cores que a legenda não mostra em lugar nenhum.

Não dá erro e passa em qualquer olhada rápida: o gráfico tem cor, a legenda tem
cor, só não são as mesmas.

## A regra

- **A paleta por posição é o padrão** para séries sem identidade própria (dois
  períodos, entradas e saídas).
- **A série aceita `cor` opcional**, e ela vence a posição. Quem tem catálogo
  passa a cor do catálogo; a legenda, a composição e o gráfico leem do mesmo
  lugar.
- **Vale para todo desenho que pinta por índice**: barras de ranking com cor por
  item, barras empilhadas, área. Uma barra de "ação" pintada pela classe a que a
  ação pertence é o mesmo caso.

## De quebra: descrição que promete forma

A mesma série vinha descrita como "empilhado por classe" e era área sobreposta
(cada classe do zero, com transparência). Empilhar virou opção do componente, e
com ela a opacidade muda: sobreposta, a área é véu sob a linha; empilhada, a
espessura da faixa é a leitura e precisa de corpo.

## Conexões
- Princípio: [[Dado escrito por dois caminhos precisa de uma regra só, fora dos dois]]
- Irmã: [[Componente que serve dois donos recebe o catálogo, não o campo renomeado]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
