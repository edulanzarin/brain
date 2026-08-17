---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-08-17
---

# Estado desejado persistido religa o robô depois do restart

> Uma tabela guarda o que o usuário QUER do robô (ligado, modo, alvo); a reconexão com backoff e o boot do container releem a tabela e religam sozinhos.

## O problema

O robô server-side vivia só em memória (singleton com a conexão WS). Duas mortes
silenciosas: a conexão caía e ficava caída até o usuário religar na mão; e um restart
do container matava todos os jobs sem deixar rastro — nada no banco dizia que eles
deviam estar rodando.

## A solução

Separar intenção de execução em três peças:

1. **Tabela de estado desejado** (`robot_sessions`): `enabled`, `mode`, alvo atual e
   configs, mais `last_status` observado (pro monitor mostrar algo com o processo
   recém-nascido). O motor grava fire-and-forget a cada decisão do usuário — banco
   fora do ar nunca derruba o robô.
2. **Reconexão com backoff**: no `close/error` do socket, se `enabled`, agenda retry
   com backoff exponencial (5s → 60s, pra sempre). Detalhe que morde: o WebSocket não
   tem o retry-em-401 do REST — token vencido é conexão recusada direto. Então
   **renova o access token ANTES de cada tentativa** de reabrir.
3. **Boot religa** (`instrumentation.ts` do Next, `register()` com delay pro banco
   subir): lê as sessões `enabled`, remonta o contexto (tokens, shard, plano) e chama
   `resume()` no motor. Restart do container deixou de ser evento — o robô volta só.

O comando explícito do usuário ("desligar") zera `enabled` e cancela o retry — só a
interferência do mundo dispara reconexão, nunca a decisão do usuário é desrespeitada.

## O que mais vale lembrar

- `getState()` expõe `reconnecting` e `nextRetryAt`: a UI mostra "religando em Xs" em
  vez de um "erro" seco — o monitor conta a verdade do que o motor está fazendo.
- Registrar o evento de reconexão no feed ("conexão restabelecida") dá confiança no
  robô que trabalha sozinho de madrugada.

## Conexões
- Princípio: [[Guarde a intenção e o processo se reconstrói dela]]
- Irmã: [[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
