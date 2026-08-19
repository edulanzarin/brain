---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-19
---

# Num confronto, medir só o seu lado recomenda o alvo que te destrói

> Regra que vale para os dois lados (vantagem de tipo, custo, limite) e entra no modelo de
> um lado só não deixa a estimativa imprecisa — ela **inverte** a recomendação: o alvo em
> que você é mais forte é exatamente aquele em que ele também é.

## O problema

Um motor de escolha de alvo mede o quanto você bate: poder do golpe, razão de stats,
vantagem elemental, tempo até derrubar. Ordena por rendimento e devolve o primeiro. O
alvo campeão costuma ser o de vantagem máxima — e vantagem elemental é **simétrica**: o
Psíquico que bate x2,5 no Venenoso leva x2,5 do Fantasma que veio junto. Com pouca vida,
o "melhor alvo" mata você antes do segundo abate.

O sintoma não é um número torto na tela: é o motor recomendando, com convicção, a pior
opção disponível.

## A solução

**Mesma fórmula, lados invertidos.** O dano que você toma sai da mesma função de dano,
trocando atacante e defensor, com a mesma amplificação. Dali saem duas grandezas úteis:

- **quantas repetições a vida cheia aguenta** — o tempo até cair dividido pelo tempo de
  combate de cada abate;
- **quanto da hora sobra produzindo** — a queda custa detecção, cura e volta ao campo.

O segundo vira multiplicador do rendimento (ver
[[Rendimento é vazão vezes tempo em pé, não vazão de pico]]), e o primeiro vira o rótulo
que a interface mostra: seguro, arriscado, letal.

## Dado que faltava, sem engordar o payload

Para medir o outro lado é preciso o que o outro lado tem (golpes, ataque, ataque
especial). Os stats são números — cabem no objeto do alvo. Os golpes não: repetir a lista
de ataques em 416 alvos custava ~120KB no payload da página pública, e o catálogo de
espécies **já ia junto**. O motor passou a receber um resolvedor (`movesOf(id)`) em vez do
objeto inchado — quem chama já tem o catálogo na mão.

## Visto em

No piwdex, `src/lib/combat.ts`: `threatOf()` estima o dano recebido e devolve
`killsPerLife` + `uptime`; `estimateHunt()` já entrega KOs/h, XP/h e ouro/h efetivos.
O "melhor pokémon do seu time para esta hunt" tinha o mesmo viés (pontuava efetividade
vezes poder) e passou a pontuar pelo mesmo XP/h efetivo.

## Conexões
- Princípio: [[Rendimento é vazão vezes tempo em pé, não vazão de pico]]
- Irmã: [[Estimativa desmentida pela realidade vira veto temporário do motor]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
