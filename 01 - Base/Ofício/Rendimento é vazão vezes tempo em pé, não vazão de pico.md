---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-08-19
---

# Rendimento é vazão vezes tempo em pé, não vazão de pico

> Ao comparar opções por rendimento, o número que ordena tem que ser o **efetivo**: a
> vazão bruta multiplicada pela fração do tempo em que o sistema fica de pé. Ranquear
> por vazão de pico escolhe justamente a opção que derruba mais rápido.

## A regra

O custo de falhar não é um filtro ao lado do cálculo — é **tempo parado dentro do mesmo
número**. Estime quanto tempo o sistema aguenta antes de cair (quantas repetições cabem
numa vida, numa cota, num limite) e converta isso em fração de hora útil:

```
repetições por vida = tempo até cair / tempo de cada repetição
tempo em pé         = (repetições por vida * ciclo) / (repetições por vida * ciclo + custo da queda)
rendimento efetivo  = rendimento bruto * tempo em pé
```

Um número só ordena; dois números discutem. Quando o risco vira multiplicador, a opção
brilhante que cai a cada duas repetições desaba sozinha no ranking — sem limiar chutado,
sem "regra de segurança" separada que ora é frouxa demais ora proíbe opção boa.

## Por que

- **A opção mais atraente costuma ser a mais arriscada**, e não por acaso: ela é atraente
  porque leva o sistema ao limite. Vazão de pico e fragilidade crescem juntas.
- **Limiar arbitrário envelhece mal.** "Não pegue alvo 5 níveis acima" erra dos dois
  lados; "desconte o tempo caído" continua certo em qualquer escala.
- **O mesmo número explica a decisão.** Quem lê a lista entende por que a segunda ganhou
  da primeira, e a interface pode mostrar as duas metades sem inventar outra métrica.

Reserve a trava dura para o caso **degenerado** — aquele que cai tão rápido que o modelo
perde sentido — e guarde um último recurso pra nunca devolver lista vazia.

## Onde apareceu

- **[[Idle Game]]**: o resolver do período idle desconta a queda (herói de defesa baixa
  "cai" e o offline rende pouco). O rendimento do intervalo já nasce descontado.
- **[[piwdex]]**: o motor da rota de hunt ordenava por XP/h **bruto** e mandava um Abra de
  9 de HP caçar Gastly — matava rápido e morria mais rápido ainda, rendendo zero. Passou a
  ordenar por XP/h efetivo e o alvo mortal saiu da rota sozinho.

O mesmo raciocínio serve fora de jogo: o worker que satura e reinicia, o scraper que toma
bloqueio, a query paralela que estoura conexão. Vazão medida no minuto bom mente sobre a
hora inteira.

## Conexões
- Depende de: [[Progresso idle é função pura do tempo semeada, não simulação tick a tick]]
- Irmã: [[Um invariante se garante na estrutura, não no processo]]
- Visto em: [[Idle Game]] · [[piwdex]]
- Mapa: [[Base]] · [[Backend]]
