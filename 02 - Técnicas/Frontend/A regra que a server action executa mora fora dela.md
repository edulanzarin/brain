---
tags: [tipo/atomica, camada/padrao, dev/frontend]
criado: 2026-08-28
---

# A regra que a server action executa mora fora dela

> Server action é porta de entrada: sessão, revalidate e redirect. A regra que ela
> dispara vive numa função comum — que roda num script, sem navegador.

## O problema

Server action é cômoda demais: dá para escrever a compra inteira dentro dela, com
`auth()` na primeira linha e `redirect()` na última. O preço aparece na hora de conferir
se funciona. A função só se alcança pelo protocolo do framework — POST com o id da ação
que o build gerou — então a única forma de exercitá-la é clicando na tela. E `redirect()`
funciona lançando exceção, o que quebra qualquer tentativa de chamar a função direto para
ver o que ela devolveu.

## A solução

Duas camadas. A regra é uma função comum que recebe ids e devolve um resultado
descrito por tipo:

```ts
// src/lib/compra.ts — sem "use server", sem sessão, sem redirect
export async function efetivarCompra(usuarioId, cursoId, email): Promise<ResultadoCompra>

// src/lib/actions/compra.ts
"use server";
export async function comprarAction(cursoId: string) {
  const usuario = await exigirUsuario();
  const r = await efetivarCompra(usuario.id, cursoId, usuario.email);
  if (r.tipo === "recusada") return falha(r.motivo);
  revalidatePath("/aprender");
  redirect(`/pedido/${r.pedidoId}`);
}
```

A action fica com o que só existe dentro de um request. A regra vira algo que um
`npx tsx` roda contra o banco de dev: cria usuário, compra, confere que o pedido virou
`PAGO`, que a matrícula nasceu ligada a ele, que a segunda matrícula esbarra na
unicidade — e apaga o usuário no fim.

## O que mais vale lembrar

- O resultado por união marcada (`{ tipo: "recusada", motivo }`) é o que permite a
  action decidir entre erro na tela e redirect. Devolver `boolean` empurra a regra de
  volta para dentro dela.
- O mesmo corte serve à segunda tela que precisar da regra: uma rota de webhook do
  provedor real chama `efetivarCompra` sem passar por formulário nenhum.

## O segundo caso, e a promoção

No [[Privello]] o mesmo aprendizado chegou pelo outro lado: a regra já estava
escrita dentro da ação, e foi **escrever o teste que forçou a separação**. Ele
morreu primeiro em `cookies` chamado fora de escopo de requisição; movida a
leitura da sessão para a ação, morreu de novo em `revalidatePath`. Duas mortes,
a mesma causa — a regra estava misturada com a porta.

Vale reparar que `revalidatePath` é da porta e não da regra, ainda que pareça
consequência da escrita: quem invalida cache é a requisição que respondeu.

Com dois sistemas e dois caminhos diferentes até a mesma conclusão, virou
princípio: [[A regra mora fora da porta que a chama]].

## Conexões
- Princípio: [[A regra mora fora da porta que a chama]]
- Irmã: [[React reseta o formulário ao fim de uma Server Action]]
- Visto em: [[monofire]] · [[Privello]]
- Mapa: [[Frontend]]
