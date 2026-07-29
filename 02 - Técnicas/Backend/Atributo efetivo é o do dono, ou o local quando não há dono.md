---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-07-29
---

# Atributo efetivo é o do dono, ou o local quando não há dono

> Quando um atributo mora numa entidade "dona" (o grupo na empresa) mas um
> registro filho às vezes não tem dono (um e-CPF sem empresa), dar ao filho um
> campo próprio OPCIONAL e resolver o efetivo como `dono?.attr ?? proprio`.
> Guardar o local só quando não há dono, e zerá-lo quando um dono aparece — uma
> fonte de verdade só.

## O problema

O atributo nasce colado numa entidade porque "faz sentido lá": grupo econômico é
da empresa. Some o caso em que o registro que deveria carregá-lo não tem essa
entidade. No Cofre Digital, grupo era só de `Company`; um certificado e-CPF
avulso (sem empresa) não tinha onde guardar o grupo. O formulário mostrava o
seletor de grupo e, ao salvar sem empresa, **descartava o valor em silêncio** —
a pessoa escolhia o grupo e ele não colava. Controle que descarta valor sem
avisar é desonesto, primo de [[Filtro transversal só é honesto se todo o funil o
honra]] ("um filtro que mente é pior que filtro nenhum").

## O padrão

Dar um lar ao valor em vez de esconder o controle: um `groupId` OPCIONAL no
próprio filho, usado como fallback.

- **Leitura** — o efetivo é `dono ?? proprio`, mas com o dono mandando quando
  existe (mesmo sem valor): `row.company ? row.company.group : row.group`. Filtro,
  coluna e contagem passam a ler esse efetivo, não o do dono direto.
- **Escrita** — com dono, atribui no dono e **zera o local** (`groupId = null`);
  sem dono, grava no local. Zerar o local quando há dono mantém **uma fonte só**:
  nunca há dúvida de qual vence — [[Um invariante se garante na estrutura, não no
  processo]].
- **Vazio não mexe** — seletor em branco/oculto preserva o que já havia (não
  desvincula), igual à atribuição no dono.

## O que mais vale lembrar

A pergunta que evita o bug: "e quando este registro NÃO tem o dono onde o atributo
mora?". Se a resposta é "o controle some sem efeito", ou dê um lar ao valor (campo
local + efetivo com fallback), ou esconda o controle nesse caso — nunca deixe
descartar em silêncio.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Filtro transversal só é honesto se todo o funil o honra]]
- Irmã: [[Criar e editar passam pelo mesmo funil de resolução]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Backend]]
