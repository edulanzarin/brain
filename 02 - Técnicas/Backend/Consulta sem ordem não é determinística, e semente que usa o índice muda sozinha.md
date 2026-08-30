---
tags: [tipo/atomica, camada/padrao, armadilha, dev/backend, sql]
criado: 2026-08-30
---

# Consulta sem ordem não é determinística, e semente que usa o índice muda sozinha

> O seed de exemplo rodava duas vezes e o banco dobrava. Nada no código dizia
> "crie de novo": o `upsert` simplesmente não reconhecia mais as linhas que ele
> mesmo tinha criado.

## O problema

`SELECT` sem `ORDER BY` pode devolver as linhas em qualquer ordem, e devolve
mesmo — muda com o plano do executor, com a atualização de estatísticas, com uma
escrita no meio. Isso é conhecido. O que morde é a **segunda ordem** de
consequência:

```ts
const cidades = await prisma.cidade.findMany({ where: { ... } }); // sem ordem
for (const [c, cidade] of cidades.entries()) {
    const semente = c * 100 + i;                  // o ÍNDICE virou dado
    const nome = NOMES[embaralho(semente, ...)];  // e o dado escolhe o conteúdo
    const arroba = `${nome}_${cidade.slug}_${i}`;
    const email = `${arroba}@exemplo`;            // que virou a CHAVE do upsert
    await prisma.usuario.upsert({ where: { email }, ... });
}
```

Trocando a ordem das cidades, a mesma posição gera outro nome, outro @, outro
e-mail — e o `upsert` cria um usuário novo em vez de reencontrar o antigo. O
resultado não é erro: é uma leva inteira duplicada, com dado plausível,
descoberta só quando alguém olha o total (117 anúncios viraram 208).

## A regra

**Se o índice de uma coleção entra em qualquer conta, a coleção precisa de
ordem explícita.** Não é sobre estética de saída, é sobre determinismo:
embaralho estável, cor derivada de posição, distribuição por fatia, nome
sorteado — tudo que usa `i` está usando a ordem como se fosse dado.

O conserto é uma linha, e o comentário ao lado vale mais que ela:

```ts
orderBy: { slug: "asc" }   // o índice entra na semente; sem ordem, o exemplo muda
```

## Como perceber

Rode duas vezes seguidas e compare o TOTAL, não o resultado da segunda rodada.
Seed idempotente que imprime o mesmo número duas vezes está bom; seed que
imprime "117 criados" nas duas mas o banco tem 208 está mentindo — o número que
ele imprime é o do laço, não o do banco.

## Conexões
- Princípio: [[Índice só é identidade enquanto a coleção não muda]] — aqui a
  coleção é a mesma, mas a ORDEM dela não é, e a posição virou identidade do
  mesmo jeito.
- Irmã: [[Semear teste cria linha nova, não muta linha real]]
- Visto em: [[Privello]]
- Mapa: [[Backend]] · [[Dados]]
