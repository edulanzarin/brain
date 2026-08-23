---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-23
---

# O código com que o socket fecha é a classificação que o retry precisa

> Conexão persistente que cai chega no seu código como um evento só: `close`. É
> tentador tratar o evento como o fato inteiro e reconectar. Mas o `close` vem com
> um número, e é ele que separa a queda que reconectar resolve da queda que
> reconectar nunca vai resolver. Jogar o número fora troca um diagnóstico por um
> laço.

## O que o número carrega

O WebSocket reserva a faixa **4000–4999 para a aplicação**. Quem escreveu o
servidor usa essa faixa justamente para dizer o que aconteceu, e costuma mandar
uma frase junto no `reason`. Não é enfeite de protocolo: é o canal por onde o
outro lado explica a recusa.

No [[piwdex]], sondando o servidor do jogo com um token inválido:

```
WS open
WS close code=4001 reason=unauthorized
```

e com o token bom no shard errado, `4003 wrong-shard`. Dois problemas com
tratamentos opostos, ambos anunciados, ambos descartados pelo motor — que lia só
"caiu" e agendava backoff. Resultado: reconexão infinita, e uma tela que só sabia
dizer "sessão perdida, tentando de novo em 3s".

## A tabela é o desenho

Escreva a classificação como dado, não como cadeia de `if`. A coluna que importa
não é o que o código significa: é **o que fazer com ele**.

```ts
const FECHAMENTOS: Record<number, { acao: "token" | "shard" | "parar" | "tentar"; frase: string }> = {
  4001: { acao: "token", frase: "o jogo recusou o token desta conexão" },
  4003: { acao: "shard", frase: "shard errado — a conta foi remanejada" },
  4004: { acao: "parar", frase: "o jogo recusou a conta" },
  1011: { acao: "tentar", frase: "erro interno do servidor" },
};
```

- **`shard`** — o valor cacheado ficou velho: redescobre e reconecta. Insistir no
  mesmo erra 100% das vezes.
- **`token`** — renova o par. Recusado de novo depois de renovar, o vínculo
  morreu: **pare** e peça a credencial nova, porque nenhuma tentativa sua produz
  uma.
- **`parar`** — terminal. Ver [[Recusa não é falha: contra o não do servidor, insistir é ruído]].
- **`tentar`** — a queda comum, a única que merece backoff.

## O que mais vale lembrar

- **Um contador por causa, e ele zera no `open`.** "Três shards errados seguidos"
  quer dizer outra coisa que "um shard errado" — mas só se a contagem for da
  sessão atual. Sem zerar na conexão que deu certo, um episódio ruim de ontem
  condena a de hoje.
- **Mostre o número cru na tela**, ao lado da frase traduzida. Traduzir sem
  mostrar apaga a evidência, e é ela que encerra a discussão sobre de quem é o
  problema.
- **Código desconhecido não vira palpite.** Caia no retry normal e registre o par
  número/frase. A tabela cresce com o caso real na mão.
- Quando o transporte de fato **não** classifica (o `1006` genérico é comum), aí
  sim vale perguntar a um canal que saiba — uma chamada REST responde com status
  HTTP. Mas essa é a saída de emergência, não o primeiro movimento.

## Conexões
- Princípio: [[Laço que trata toda falha igual apaga a causa da primeira]]
- Irmã: [[Recusa não é falha: contra o não do servidor, insistir é ruído]] ·
  [[Socket que não abre não emite evento, e só um temporizador percebe]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
