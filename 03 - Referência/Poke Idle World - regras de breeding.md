---
tags: [tipo/referencia]
criado: 2026-08-15
---

# Poke Idle World - regras de breeding

> O breeding do jogo NAO tem endpoint (ver [[Poke Idle World - endpoints publicos de
> dados]]): as regras da especie sao curadas, nao vem no JSON. Aqui ficam as regras
> confirmadas do jogo, o que ainda e provisorio e o que nao se sabe — pra qualquer
> ferramenta simular sem redescobrir nem inventar numero.

Visto em: [[piwdex]]

## Regras confirmadas

- Os pais devem ser da **mesma especie** e a diferenca de **Quality no maximo 0.150**.
- Quality usa **tres casas decimais**. O filho parte da **maior Quality entre os pais**
  mais um ganho (tabela abaixo).
- Os dois pais sao **consumidos**; todo par valido gera **um ovo**.
- O filho **herda a distribuicao completa de IVs do pai de maior Quality**; em empate
  exato, herda do **Slot 1**. IVs nunca sao misturados/medios — vem inteiros de um pai.
- Custo base: **R$ 2.000.000 + 20 Evolution Stones**. Pokemon de dois tipos divide as
  Stones igualmente (10/10). Cada IV vai ate **32**, total maximo **192**.
- **Double Stones**: usa **40 Stones** e da **5% de chance de +1 IV** num stat aleatorio
  **abaixo de 32**. Com todos os stats em 32, nao ha ganho possivel.
- **Shiny por heranca**: se ao menos um pai for Shiny, o filho e **sempre Shiny**, sem
  mudar a regra de heranca de IV. Pokemon **normal tem Quality maxima 2.600**.

## Ganho de Quality por modo

O filho recebe UM ganho sorteado, somado a maior Quality dos pais (teto 2.600 no normal;
Shiny fica com a Quality bruta, sem teto).

| Free Breeding | prob | Strange Pheromone | prob |
|---|---|---|---|
| +0.005 | 50% | +0.150 | 50% |
| +0.010 | 35% | +0.200 | 30% |
| +0.020 | 12% | +0.250 | 15% |
| +0.040 | 3%  | +0.300 | 5%  |

Ganho esperado (media ponderada): **Free 0.0096**, **Pheromone 0.1875**. Free custa so
R$ + 20 Stones; o modo Pheromone normal usa **9 Strange Pheromones**.

## Provisorio (marcar, nao inventar)

- Dois pais **normais** tem **5% de chance de gerar um filho Shiny espontaneo**.
- Custos de Pheromone por tier Shiny (Tier C 50, Tier B 150, Tier A 1500) e as
  combinacoes normal+Shiny ainda podem mudar com doc oficial.

## Ainda desconhecido

- Os **limites exatos de Quality por tier Shiny** nao sao conhecidos — nao inventar.
- A **incubacao do ovo** (tempo/progresso) nao esta descrita.

## Estrategia

- **Free** pra avancos pequenos e baratos. Acima de Quality ~1.950, planejar um parceiro
  compativel antes de investir: um resultado muito distante pode passar do limite 0.150
  e deixar o parceiro inutilizavel.
- **Pheromone** acelera (bonus 0.150–0.300) mas arrisca bater o teto 2.600 e desperdicar
  Quality. Comparar os quatro resultados antes de consumir os pais.
- Colocar no **Slot 1** o Pokemon cujos IVs se quer preservar (doa a distribuicao em
  empate de Quality).

Implementado em piwdex (`src/lib/breeding.ts` + ferramenta `/breed`): colecao local,
validacao de par, projecao de Quality/IV/custo. As medias batem exato com o jogo.

## Conexoes
- Irma: [[Poke Idle World - endpoints publicos de dados]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
