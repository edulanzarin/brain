---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-08-14
---

# Progresso idle é função pura do tempo semeada, não simulação tick a tick

> O ganho de um período idle se resolve com uma função pura `(seed, Δt, poder, ambiente) -> relatório`, avaliada uma vez sobre o intervalo inteiro — nunca somando um tick de cada vez.

## A regra

Não simule o mundo segundo a segundo pra acumular recompensa. Modele o
resultado como uma **função determinística** que recebe o tempo decorrido (Δt),
uma **semente estável** (o id/started da sessão) e os parâmetros de gameplay
(poder do herói, zona, drop table), e devolve o relatório de uma vez. Offline de
6h é a **mesma chamada** que 1s de online — só muda o Δt. O cliente nunca calcula
recompensa: manda a intenção, o servidor resolve.

## Por que

- **À prova de trapaça.** Se o ganho emerge de um loop no navegador, é trivial
  burlar (o furo do jogo de referência do Idle Game). Resolvendo no servidor a
  partir da semente, o cliente não tem o que forjar.
- **Offline honesto e barato.** Sem função pura você teria que "recuperar" horas
  simulando tick a tick no login — lento e frágil. Com ela, offline é aritmética.
- **Idempotência por construção.** A mesma janela coletada duas vezes dá o mesmo
  resultado; a coleta materializa e reinicia a janela (`startedAt = agora`).
- **Reproduzível/testável.** Mesmo seed, mesmo relatório — dá pra verificar sem
  banco (rodei o resolver 2x com seed igual e comparei o JSON).

O combate ainda "emerge dos stats" (dps do herói vs hp/def do mob, dano sofrido
por status): a graça é derivar o total do intervalo (mortes = orçamento de dano /
hp; queda = quando a vida zera) em vez de iterar o relógio.

## Na prática

Apareceu duas vezes no [[Idle Game]], em designs diferentes: primeiro no resolver
de captura estilo Pokémon, depois reescrito pro combate Albion (drop table por
mob, shiny escalado pela zona, teto de acúmulo offline). O que sobrevive à
mudança de design é justamente esta forma — por isso é princípio, não técnica.

A semente deve **encodar a janela** (`session.seed + startedAt`) pra que coletas
sucessivas não fiquem correlacionadas. Os parâmetros de gameplay vêm de dado
(zona/mob/drop no banco), não de casos no código — ver
[[A definição em dado dirige o comportamento, não um caso no código]].

## Conexões
- Depende de: [[A definição em dado dirige o comportamento, não um caso no código]]
- Irmã: [[Permissão se valida no servidor, não na interface]]
- Visto em: [[Idle Game]]
- Mapa: [[Base]] · [[Backend]]
