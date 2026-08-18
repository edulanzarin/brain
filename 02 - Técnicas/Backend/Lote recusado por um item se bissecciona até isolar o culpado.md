---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-18
---

# Lote recusado por um item se bissecciona até isolar o culpado

> API de lote tudo-ou-nada (um item inválido = 400 no lote inteiro) e o dado local não
> diz QUAL item é inválido: bisseccione o lote até isolar os recusados, processe o resto
> e ponha os isolados numa lista de bloqueio pra não travarem a próxima rodada.

## O problema

A venda automática mandava todos os pokémon vendáveis num `POST` só. O jogo recusa o
lote inteiro se UM for invendável (anunciado no mercado, na equipe, shiny) — e a lista
viva não expõe a flag "anunciado no mercado", então não dá pra filtrar antes de tentar.
Resultado: um único bicho anunciado congelava a venda inteira, varredura após varredura,
com o mesmo erro 400 pra sempre.

## A solução

Tentar o lote inteiro; num 400, dividir ao meio e recursar. Lote de tamanho 1 que falha
é o culpado: entra num `Set` de bloqueio e sai das próximas varreduras.

```ts
async function sellBatch(ids: string[]): Promise<Sold> {
  const w = await sell(ids);
  if (w.ok) return sold(w);
  if (w.status === 400) {
    if (ids.length === 1) { blocked.add(ids[0]); return none; }
    const mid = Math.ceil(ids.length / 2);
    return merge(await sellBatch(ids.slice(0, mid)), await sellBatch(ids.slice(mid)));
  }
  return none; // falha transitória (5xx/rede): a próxima varredura tenta de novo
}
```

- Custo máximo ~`2 * culpados * log2(lote)` chamadas — barato até com vários culpados.
- Só bisseccionar o erro DETERMINÍSTICO (400 de validação). 5xx/rede é transitório:
  aborta e deixa a próxima rodada tentar, senão a bissecção vira tempestade de retry.
- A lista de bloqueio é da sessão e se limpa quando a config muda ou a sessão reinicia:
  o estado que causou a recusa (anúncio no mercado) é externo e pode mudar — bloqueio
  eterno viraria item nunca mais processado.

## Conexões
- Princípio: [[Chamada externa tem timeout e erro tratado]]
- Irmã: [[Falha de automação recorrente vira alerta com throttle, não catch vazio]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
