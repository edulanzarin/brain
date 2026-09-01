---
tags: [tipo/atomica, camada/padrao, sql, dev/backend, armadilha]
criado: 2026-09-01
---

# Uma conexão do pg não atende duas consultas ao mesmo tempo

> `Promise.all` sobre consultas que compartilham **o mesmo cliente** do `pg` não roda
> nada em paralelo: o driver enfileira internamente. O código promete concorrência,
> entrega fila, e o `pg` já avisa que vai deixar de aceitar isso.

## O erro que evita

O reflexo de juntar leituras independentes num `Promise.all` está certo — quando cada
uma pega a **sua** conexão do pool. Dentro de uma transação não é o caso: ali existe um
cliente só, reservado, e é ele que carrega o contexto que torna a transação o que é.

O sintoma é enganoso porque **funciona**. As consultas voltam, os dados estão certos, o
tempo é o mesmo que seria sequencial. O único vestígio é uma linha de aviso no log:

```
DeprecationWarning: Calling client.query() when the client is already executing
a query is deprecated and will be removed in pg@9.0
```

Quer dizer: hoje é ilusão de paralelismo, no `pg@9` é erro. E some numa atualização de
dependência, longe de onde foi escrito.

## A costura

Dentro da transação, `await` em sequência, e o comentário dizendo por quê — senão o
próximo leitor "corrige" de volta para `Promise.all`:

```ts
await comEscritorio(id, async (consulta) => {
  // Em sequência, não em Promise.all: é UMA conexão dentro de UMA transação, e
  // uma conexão do pg não executa duas consultas ao mesmo tempo.
  const lista = await conversas(consulta, status);
  const contagens = await consulta("SELECT status, count(*) FROM conversas GROUP BY status");
});
```

Paralelizar de verdade exige conexões diferentes — e aí não é mais a mesma transação,
nem o mesmo contexto. Quando a leitura é pesada o bastante para pagar isso, o caminho é
uma consulta só que devolva tudo, não várias conexões.

## Conexões
- Princípio: [[Ambiente de dev sobe igual ao de produção]]
- Irmã: [[Numeric e bigint do Postgres chegam como string no driver pg]] · [[Contexto de tenant tem que morrer no commit, senão o pool o carrega adiante]]
- Visto em: [[CRM Contábil]]
- Mapa: [[Dados]]
