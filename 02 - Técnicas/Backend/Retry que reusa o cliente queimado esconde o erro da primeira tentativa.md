---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-23
---

# Retry que reusa o cliente queimado esconde o erro da primeira tentativa

> Um `pg.Client` que falhou o `connect()` não pode ser reconectado: a segunda
> chamada estoura `Client has already been connected`. Um laço de espera que
> reusa a mesma instância e trata toda exceção como "banco ainda subindo" passa a
> reportar o próprio defeito, e não o do banco.

## O sintoma

O runner de migration do piwdex esperava o Postgres com quinze tentativas. A
saída:

```
banco indisponivel (tentativa 12/15), aguardando...
banco indisponivel (tentativa 13/15), aguardando...
banco indisponivel (tentativa 14/15), aguardando...
Error: Client has already been connected. You cannot reuse a client.
```

As três coisas que isso parecia dizer eram falsas. O banco estava de pé e
saudável; o erro final não tinha nada a ver com disponibilidade; e a causa real
— o usuário na `DATABASE_URL` era o do projeto anterior — só apareceu quando a
conexão foi tentada fora do laço.

## A forma certa

Cliente novo a cada volta, e o motivo impresso junto da contagem:

```js
export async function connectWithRetry(tries = 15, delayMs = 2000) {
  let ultimo;
  for (let i = 1; i <= tries; i++) {
    const client = makeClient();          // instância NOVA
    try { await client.connect(); return client; }
    catch (e) {
      ultimo = e;
      await client.end().catch(() => {}); // não deixa socket pendurado
      if (i === tries) break;
      console.log(`banco indisponivel (tentativa ${i}/${tries}): ${e.message}`);
      await new Promise((r) => setTimeout(r, delayMs));
    }
  }
  throw ultimo;
}
```

Duas mudanças, e a segunda é a que salva o tempo: a mensagem do erro sai em toda
volta. Com ela, `password authentication failed` aparece na primeira linha e a
espera de trinta segundos nem chega a acontecer.

## Vale além do Postgres

Qualquer recurso com handshake tem esse comportamento: socket, cliente de fila,
stream HTTP. A pergunta a fazer ao escrever o laço é "este objeto sobrevive a uma
falha?" — e a resposta segura é não.

## Conexões
- Princípio: [[Laço que trata toda falha igual apaga a causa da primeira]]
- Irmã: [[Runner de migration em SQL puro dispensa o CLI do ORM]] ·
  [[Migrations em container próprio no Docker Compose]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]] · [[Infra]]
