---
tags: [tipo/atomica, camada/padrao, armadilha, dev/backend]
criado: 2026-08-30
---

# Registro com estado não se confere pela existência

> Quando uma tabela guarda um pedido que passa por uma fila — verificação,
> aprovação, pagamento —, a linha existe desde o primeiro envio e continua
> existindo depois de recusada. Conferir com `if (registro)` responde "ela
> mandou", não "está resolvido", e as duas frases se separam exatamente no caso
> que mais precisa de resposta.

## Como aparece

A tela de revisão do anúncio listava o que ainda travava a publicação:

```ts
!verificacao && {
    trava: true,
    texto: "Documento não enviado.",
}
```

Nunca enviou: trava, certo. Enviou e está na fila: não trava, certo. **Enviou e
a moderação recusou**: a linha existe, então não trava — e a tela dizia "nada
trava a publicação" para quem estava justamente com a bola no pé. A pessoa
ficava esperando uma publicação que não vinha.

O erro é silencioso porque a leitura mais barata (existe?) acerta a maioria dos
casos. Só o caminho de exceção discorda, e é o caminho que ninguém percorre
testando.

## A regra

A pergunta é pelo **estado**, e cada estado terminal precisa de resposta própria.
Recusado não é "entregue"; é uma trava diferente, com um texto diferente — e o
texto carrega o motivo, porque recusa sem explicação volta como o mesmo envio na
semana seguinte.

```ts
verificacao?.estado === "REJEITADA" && {
    trava: true,
    texto: `Documento recusado: ${verificacao.motivoRecusa}`,
}
```

Um sintoma de que o código caiu nisso: a consulta seleciona só as colunas de
identificação e não traz `estado`. Se o `select` não pediu o estado, a decisão
não podia estar olhando para ele.

## Conexões
- Princípio: [[Contador que conta sucesso de promessa afirma que deu certo]] — o
  mesmo buraco: "não falhou" e "fez o trabalho" são frases diferentes, e a
  leitura barata confunde as duas.
- Irmã: [[Uma pendência de prazo fecha por ato explícito, não por sinal inferido]]
- Visto em: [[Privello]]
- Mapa: [[Backend]]
