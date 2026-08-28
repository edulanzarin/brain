---
tags: [tipo/atomica, camada/padrao, sql]
criado: 2026-08-28
---

# Acesso comprado é linha própria, não status do pedido

> "Tem acesso" e "pagou" parecem a mesma pergunta e não são. Duas tabelas, ligadas
> mas independentes, respondem cada uma a sua sem apagar a outra.

## O problema

O atalho natural em qualquer venda de conteúdo é liberar o acesso lendo o pedido:
`SELECT ... FROM pedido WHERE usuario = ? AND curso = ? AND status = 'PAGO'`. Funciona
até o primeiro reembolso — e aí, para tirar o acesso, ou se apaga a venda (some do
histórico e do faturamento) ou se inventa um status que quer dizer duas coisas ao mesmo
tempo. Cortesia, acesso de equipe e curso gratuito também não têm pedido nenhum para
apontar.

## A solução

Duas tabelas: `pedido` guarda a venda (valor, provedor, referência externa, quando pagou)
e `matricula` guarda o acesso. A matrícula aponta para o pedido que a originou, mas o
ponteiro é opcional:

```prisma
model Matricula {
  usuarioId String
  cursoId   String
  pedidoId  String?  @unique

  @@unique([usuarioId, cursoId])
}
```

O que cada decisão compra:

- `pedidoId` **opcional** — matrícula sem venda é caso legítimo (gratuito, cortesia),
  não exceção a contornar.
- `@@unique([usuarioId, cursoId])` — comprar duas vezes o mesmo curso é impossível pela
  estrutura, não por um `if` no meio do código de compra.
- Reembolsar é apagar a matrícula e marcar o pedido; a venda continua no histórico.

As duas nascem juntas, na mesma transação, quando a cobrança aprova. Se o processo
morrer antes disso, sobra um pedido `PENDENTE` — que é a verdade — e nunca um acesso
liberado sem venda.

## O que mais vale lembrar

A ordem importa: pedido `PENDENTE` **antes** de chamar o provedor. Gravar depois da
cobrança é apostar que o processo não morre entre o dinheiro sair e a linha existir.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Provedor de pagamento entra por interface, e o simulado é a primeira implementação]]
- Visto em: [[monofire]]
- Mapa: [[Dados]]
