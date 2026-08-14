---
tags: [tipo/atomica, camada/padrao, dev/frontend]
criado: 2026-08-14
---

# Mundo imperativo e React se falam por eventos, não por referência

> Ao embutir um subsistema imperativo (canvas/Pixi, mapa, editor, viz) numa app
> React, mantenha os dois lados **cegos um pro outro**: o mundo imperativo emite
> fatos num barramento de eventos e a UI escuta; a UI manda intenção por métodos
> estáveis. Ninguém segura referência do outro nem lê o estado interno alheio.

## Por que

O mundo imperativo (loop de render, mutação a cada frame) e a árvore React
(declarativa, re-render sob demanda) têm relógios diferentes. Se a UI cutuca o
objeto do canvas direto — ou o canvas chama `setState` no meio do loop — você
acopla os dois ciclos de vida e leva corrida no StrictMode (mount duplo), memory
leak no unmount e re-render em cascata. A ponte de eventos corta isso: cada lado
evolui no próprio ritmo, e o contrato é uma lista pequena de eventos + comandos.

## Como (a forma que funcionou)

Três camadas, dependência só pra baixo:

- **Engine** (agnóstica ao app): dona do `Application`/loop, dirigida por **dados**
  (um `MapDef`: grade, terreno, props, prédios, spawn). Não sabe o que é
  "taverna" — só renderiza o que o dado manda. Trocar de cena é trocar de dado.
  Que o mundo venha de dado é [[A definição em dado dirige o comportamento, não um caso no código]].
- **Game**: os dados concretos (a cidade) + uma `Scene` que cola as peças da
  engine e **expõe um `EventBus` tipado** (`near`, `interact`, `skin`...).
- **UI (React)**: um hook monta a engine num `<div>`, assina o bus e traduz
  evento -> `setState`; devolve **controles estáveis** (`setPaused`, `setSkin`)
  guardados num ref, que a UI chama sem re-montar nada.

Detalhes que evitam dor:
- **StrictMode-safe**: no efeito, crie o app numa var local e só assuma como
  "o app" depois do `await init()` se um flag `disposed` não subiu no meio.
- **Estado do jogo num reducer PURO + store** (fora do canvas): a UI e o loop
  leem a mesma verdade; amanhã o servidor vira a fonte sem reescrever a UI.
- **Assets opcionais não derrubam a cena**: `tryLoad` devolve null e a cena pula
  o item (um prédio que faltou não quebra o mundo).

## Vale lembrar

- É a mesma ideia de fronteira de [[Estado de tela pertence à seção, não à página]]:
  cada parte dona do seu estado, contrato explícito no meio.
- **Candidato a princípio**: "imperativo e declarativo conversam por contrato de
  eventos, não por referência" deve reaparecer fora de jogo (mapa Leaflet, editor
  Monaco, gráfico D3). Na segunda aparição, promover pra [[Base]].

## Conexões
- Aplica: [[A definição em dado dirige o comportamento, não um caso no código]]
- Irmã: [[Personagem pixel direcional se desenha em código, não se gera por IA]]
- Visto em: [[Idle Game]]
- Mapa: [[Frontend]]
