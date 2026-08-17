---
tags: [tipo/atomica, camada/padrao, dev/backend, dev/frontend]
criado: 2026-08-17
---

# Um stream SSE substitui a constelação de pollings

> Uma conexão SSE com eventos nomeados empurra tudo que a área logada mostra ao vivo; os painéis assinam um provider, ninguém mais faz `setInterval`.

## O problema

Cada painel da área fazia seu próprio `fetch` em `setInterval` (2s a 10s, oito loops
diferentes), e três telas nem isso tinham — só atualizavam no F5. Números discordavam
entre painéis, o custo era por tela, e "tempo real" era mentira com atraso.

## A solução

Um endpoint SSE único (`ReadableStream` num route handler Node do Next) que empurra
**eventos nomeados** (`event: hunt`, `account`, `events`, `alerts`, `totals`, `ping`),
cada um com seu ritmo e sua fonte:

- O que vive **em memória no servidor** (a sessão do robô) é instantâneo: o singleton
  ganha um `EventEmitter`, o handler do stream assina `bus.on("change")` e empurra na
  hora — com um throttle curto (~300ms) pra rajada de eventos não virar rajada de frames.
- O que vive **no banco ou numa API externa** entra por timer dentro do próprio stream
  (5s pro banco, 15s pra API do jogo) — o polling não sumiu, mudou de lugar: é UM,
  server-side, servindo todos os consumidores de uma vez.
- No connect, o handler manda o **snapshot completo** — a UI nasce preenchida, sem
  cascata de fetches.

No client, um provider React abre o `EventSource` (que reconecta sozinho) e distribui
por Context; os painéis só leem. Mutações continuam POST, e a resposta do POST pode
ser aplicada otimisticamente no provider (`applyX`) — o stream confirma em seguida.

```ts
// servidor: rota SSE
const stream = new ReadableStream({ start(controller) {
  const send = (event: string, data: unknown) =>
    controller.enqueue(enc.encode(`event: ${event}\ndata: ${JSON.stringify(data)}\n\n`));
  gameSession.bus.on("change", onChange);        // memoria: push na hora (com throttle)
  timers.push(setInterval(pushDb, 5000));        // banco: UM poll server-side
  req.signal.addEventListener("abort", cleanup); // sem cleanup, o timer vaza
}});
return new Response(stream, { headers: { "Content-Type": "text/event-stream",
  "Cache-Control": "no-cache, no-transform", "X-Accel-Buffering": "no" } });
```

## O que mais vale lembrar

- `X-Accel-Buffering: no` e `no-transform`: sem isso, proxy reverso bufferiza o stream
  e o "tempo real" chega em lotes.
- Tipos do servidor (que importam `node:crypto`, `pg`) **não entram no client** — o
  provider declara espelhos client-side dos payloads.
- A montagem do payload (conta, totais) sai das rotas GET pra uma lib compartilhada:
  a rota pontual e o stream servem exatamente a mesma carga.
- O `EventSource` só faz GET — autentica pelo cookie de sessão, igual a qualquer rota.

## Conexões
- Princípio: [[Estado vivo se empurra, não se pergunta]]
- Irmã: [[Total ao vivo é o persistido fechado mais o em andamento ainda não gravado]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
