---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-17
---

# Config que a sessão cacheia no init não vê a escrita no backend, reaplique na mesma conexão

> Uma sessão de longa duração (WebSocket, worker que segura um handle) costuma **ler a
> config uma vez, ao inicializar** — e trabalhar com essa cópia em memória. Gravar a
> config nova na fonte da verdade (REST, banco) **não propaga** pra essa sessão: ela segue
> com a cópia velha até reinicializar. O sintoma é clássico: "mudei o valor e salvou, mas
> não pegou; só depois de reconectar".

A saída barata **não é reconectar** (derruba a sessão, perde o estado acumulado, e no
caso single-session ainda briga por quem segura a conexão). É **reenviar o comando de
init na MESMA conexão** — o que faz a sessão reler a config sem cair. Se o canal tem um
`init`/`subscribe`/`enter` que carrega a config, reemiti-lo é o reload.

E o gatilho do reload tem que morar **no próprio caminho de escrita**: quem grava a config
avisa a sessão viva a reaplicar. Deixar isso pro humano ("lembre de reconectar") é o
processo garantindo o invariante — que é exatamente o que não se sustenta.

## Visto em

No piwdex (robô server-side do Poke Idle World), a bola do auto-catch escrita via REST
(`auto-helper`) não pegava na hunt em andamento: a engine de campo do jogo **cacheia a
autohelper no `enter-hunt`**. Trocar a bola só valia depois de desligar/religar a caça
(reconexão). A correção: a rota que grava a config chama `refreshHunt()` na sessão viva,
que **reenvia `enter-hunt` no mesmo socket** — o jogo relê a config sem reconectar nem
zerar a caça. Mesmo padrão serve pra pokémon ativo/líder (`poke-summon` no socket vivo).

## Quando reenviar o init não basta: bounce

O reenvio só funciona se a sessão relê a config no comando de init. **Se a config vem no
SNAPSHOT da conexão** (o servidor manda a cópia uma vez, no connect), reemitir o init não
recarrega nada — só uma **conexão nova** relê. Foi o que regrediu no piwdex (ago/2026): o
jogo passou a mandar a `autohelper` no snapshot e o reenter deixou de religar a bola. A
saída virou o **bounce**: no caminho de escrita, persistir o acumulado do trecho (pra o
total não perder nada), invalidar os handlers do socket velho e reconectar NA HORA —
snapshot novo, config nova. O custo (zerar o acumulado vivo) se paga persistindo antes:
[[Total ao vivo é o persistido fechado mais o em andamento ainda não gravado]].

A escada, do mais barato pro mais caro: (1) a sessão relê sozinha → nada a fazer;
(2) relê no init → reenviar o init na mesma conexão; (3) só lê no connect → bounce
com persistência antes. Descobrir em que degrau o sistema está é metade do bug.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Quando a REST não expõe o dado, o WebSocket do mesmo sistema entrega]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
