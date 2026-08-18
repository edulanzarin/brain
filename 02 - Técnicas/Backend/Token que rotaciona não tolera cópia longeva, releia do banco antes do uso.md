---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-18
---

# Token que rotaciona não tolera cópia longeva, releia do banco antes do uso

> Quando o refresh token é rotativo e mais de uma frente do servidor renova, cada frente
> relê o par de tokens do banco imediatamente antes de cada lote de chamadas REST.

## O problema

Um serviço long-lived com várias frentes falando com a mesma API externa (sessão
WebSocket, poller de snapshots, timer de compra), cada uma segurando a própria cópia dos
tokens em memória. O access expira em minutos; o refresh ROTACIONA a cada renovação.
Basta uma frente renovar para a cópia das outras apodrecer: a próxima chamada delas dá
401, o retry de refresh falha (refresh já consumido) e a operação morre — geralmente
dentro de um `catch {}` que esconde tudo. Sintoma: "às vezes funciona", degrada com o
tempo de sessão, conserta sozinho ao religar.

## A solução

- Toda frente que renova **persiste o par novo na hora** (uma função `updateTokens`
  compartilhada, gravando no banco).
- Toda frente que vai usar **relê do banco antes do lote** de chamadas — o banco tem
  sempre o último par persistido por qualquer frente, então todas convergem.

```ts
// antes de cada varredura REST (venda, compra):
const link = await getGameLink(userId);          // fonte persistida
if (link && link.status !== "expired") this.tokens = link.tokens;
```

## O que mais vale lembrar

- O WebSocket mascara o problema: a conexão viva não precisa de token depois do
  handshake, então o robô "parece saudável" enquanto todo o REST dele falha.
- Uma leitura por varredura basta; não é preciso ler por request.

## Conexões
- Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]]
- Irmã: [[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
