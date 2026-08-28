---
tags: [tipo/pensamento]
criado: 2026-08-04
---

# Primitivos, reaproveitamento e modularidade vêm antes da escala

> Quando construo, priorizo nesta ordem: **primitivo reaproveitável → modular →
> escalável**. Prefiro um kit de peças pequenas que se recombinam a telas grandes
> feitas sob medida. Escala é consequência de reaproveitar bem, não um alvo que se
> persegue direto.

## O pensamento

O Eduardo constrói de baixo pra cima: primeiro o **primitivo** (botão, campo, card,
token, uma função pura), depois compõe as telas a partir dele. A pergunta ao escrever
qualquer coisa é "isto vira uma peça reaproveitável?" antes de "isto resolve esta
tela?". Quando um conceito reutilizável aparece no meio da construção, ele extrai **na
hora** — o mesmo reflexo que aplica ao próprio cérebro ([[Manter o tooling enxuto e o conhecimento no cérebro]]): puxar pro nível mais primitivo em que ainda é verdade.

Modularidade é o par disso: cada parte com uma responsabilidade, trocável sem mexer no
resto — um token muda o app inteiro, um catálogo em dado dirige a navegação
([[A definição em dado dirige o comportamento, não um caso no código]]), um adapter
isola o provider. E a **escalabilidade vem de brinde**: um sistema feito de peças
pequenas e desacopladas cresce sem reescrever a casca.

## Por que importa

É régua de decisão, não enfeite. Diante de duas formas de resolver, ele escolhe a que
gera **peça reaproveitável e módulo isolado**, mesmo custando um pouco mais agora,
porque paga na próxima tela e no próximo projeto. É o oposto de otimizar pra entregar
só a tela de hoje e deixar o reaproveitamento pra depois — "depois" é como o
conhecimento (e o código) acaba preso. É a mesma ordem que ele usa pra escrever no
cérebro: **primitivo > reutilizável > organizado > robusto > escalável**.

## Conexões
- Irmã: [[Manter o tooling enxuto e o conhecimento no cérebro]]
- Visto em: [[Navehub]]
- Mapa: [[Pensamentos]]
