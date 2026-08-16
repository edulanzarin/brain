---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-16
---

# Quando a REST não expõe o dado, o WebSocket do mesmo sistema entrega

> A superfície real de um sistema é maior que a REST que ele documenta. O dado que
> falta num canal costuma existir noutro — porque o cliente oficial precisa dele.

## A técnica

Antes de concluir que um dado "não dá pra pegar", mapeie **todos** os canais do
sistema, não só o REST: WebSocket, SSE, gRPC, long-poll. O que o cliente oficial
mostra em tempo real, ele recebe de algum lugar — e esse lugar quase sempre carrega
mais do que a API pública admite.

No Poke Idle World isso foi o divisor de águas. O `characters/me` (REST) só devolve o
**treinador**; os pokémons ativos individuais (IV/quality/power por bicho), o time por
slot e o **analyzer da hunt** (kills, XP, loot, saldo/hora) não existem em endpoint REST
nenhum — só chegam pelo **WebSocket** do jogo, nos frames `pokes`, `field`, `field-kill`
e `analyzer`. Sem sondar o WS, esse dado ficaria "impossível". Ver o protocolo em
[[Poke Idle World - endpoints publicos de dados]].

## O custo do canal mais forte

O WS não é só leitura a mais — é a **sessão viva**. Costuma ser single-session (a
conexão nova derruba a antiga: conectar rouba a aba do jogo aberta) e costuma **mutar**
estado (no jogo, `poke-summon` troca o pokémon ativo). Então o WS é uma faca de dois
gumes: entrega o que a REST esconde, mas usá-lo custa a sessão e pode escrever. Quem só
quer ler tira um **snapshot** e solta; quem precisa de tempo real (um robô) é que segura
a conexão. Ver [[Descobrir o shard por sondagem paralela com early-exit e cachear]] pra
achar em qual nó o WS daquela conta vive.

## Conexões
- Irmã: [[Descobrir o shard por sondagem paralela com early-exit e cachear]]
- Depende de: [[Poke Idle World - endpoints publicos de dados]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
