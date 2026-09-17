---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-09-17
---

# Filtro que depende de outro filtro vai na segunda linha da barra

> Setor, cargo e horário só têm opção depois que a empresa é escolhida. Na mesma
> grade da empresa, eles acabam AO LADO dela — e a pessoa vê, lado a lado, o
> campo que precisa preencher e três campos que ainda não sabem responder.

## A regra

A barra de recorte se divide em duas linhas, e a divisão não é estética:

- **Em cima, o recorte que toda tela tem** — empresa, período, filial — e o botão
  que executa. É o que existe antes de qualquer dado.
- **Embaixo, os filtros da tela** — os que dependem do recorte de cima para terem
  lista, e que refinam o resultado em vez de defini-lo.

O usuário disse isso em uma frase, olhando a tela: "setor e o resto só aparece
depois de escolher a empresa, então os quatro têm que ficar embaixo".

## Por que a grade sozinha não resolve

A grade de células iguais (`auto-fill`) preenche a linha na ordem de declaração.
Com sete campos, o quarto sobe para a primeira linha e o botão de executar cai
para o fim da segunda, longe do campo que ele executa. A quebra explícita corrige
a leitura e a ordem de operação: escolher, executar, refinar.

## Detalhe que fecha

O botão fica na PRIMEIRA linha, alinhado pela base dos controles. Ele pertence ao
recorte, não ao refino — e ficando embaixo, a barra termina com dois botões
pequenos ao lado de um campo, o que empurra o olho para o lugar errado.

## Conexões
- Princípio: [[Ordene pela grandeza que decide, não pela que impressiona]]
- Irmã: [[Barra de filtro é grade, não fila]] · [[Quantos filtros existem é decisão de layout, não de produto]] ·
  [[Consulta pesada executa por botão, não por mudança de filtro]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
