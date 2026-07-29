---
tags: [tipo/atomica, camada/padrao, seguranca, dev/backend, sql]
criado: 2026-07-23
---

# Escopo de dado se clampa no servidor, num funil só

> Quando um usuário só pode ver parte dos dados (as empresas dele, os projetos
> dele), a lista de "o que ele pediu" chega do cliente e **não é confiável**. A
> restrição verdadeira se aplica no servidor, e de preferência num **funil único**
> por onde toda consulta passa — assim nenhuma rota escapa e é impossível esquecer.

## O erro que evita

Filtrar pelo que o cliente mandou (`?empresas=1,2,3`) confia no cliente. Ele edita
o parâmetro no DevTools e pede uma empresa que não é dele. Se o WHERE usa a lista
crua, o dado vaza. Pior: a checagem "certa" costuma ficar espalhada por N rotas, e
a de número N+1 nasce sem ela.

## A costura

O escopo permitido vem da sessão (server-only), e o funil que todas as consultas já
usam para montar o WHERE passa a ser **session-aware**: intersecta o pedido do
cliente com o permitido, e **lista vazia não casa nada** (não "casa tudo").

```ts
// buildWhere é o único lugar que monta o WHERE das consultas fiscais/contábeis.
// Lê o escopo da sessão e SEMPRE limita — o call site não passa nada.
const escopo = empresasPermitidas(sessao);        // number[] | "todas"
if (escopo !== "todas") {
  const efetivas = pedido.length ? pedido.filter(e => escopo.includes(e)) : escopo;
  // any('{}'::int[]) não casa NENHUMA linha — usuário sem empresa não vê nada
  conds.push(`codigoempresa = any($n::int[])`); params.push(efetivas);
}
```

A armadilha sutil é o caso vazio: um `if (lista.length) filtra` deixa "sem filtro =
todas as linhas". Escopo restrito tem que forçar a condição **sempre**, mesmo vazia.

## Por que o funil único

É a mesma lógica de [[Cravar o seam de permissão antes do login]]: derivar a
proteção de um ponto por onde tudo já passa (o `buildWhere`, o wrapper de rota) faz
a segurança ser **estrutural**, não uma linha que o autor precisa lembrar de colar.
Eixo diferente (Folha filtra por estabelecimento, não por empresa) = outro funil,
mesma regra.

## Conexões
- Princípio: [[Permissão se valida no servidor, não na interface]]
- Irmã: [[Cravar o seam de permissão antes do login]]
- Complemento: [[Drill-down por id foge do funil de escopo e precisa de gate próprio]] —
  o funil cobre a lista; a rota de detalhe por id escapa dele e precisa de gate próprio.
- Visto em: [[Navetech Hub]] · [[navetalks]]
- Mapa: [[Backend]]
