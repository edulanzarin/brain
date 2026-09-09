---
tags: [tipo/atomica, camada/padrao, seguranca, dev/backend]
criado: 2026-07-20
---

# O que dois módulos compartilham é a query, não a rota

> Quando dois módulos precisam do mesmo dado, o compartilhado é a consulta — num
> lib que os dois chamam. Não é uma rota de um módulo que o outro consome: isso
> amarraria a permissão do dado ao módulo errado.

## O atrito

Com gate por módulo (ver [[Cravar o seam de permissão antes do login]]), a rota
`/api/fiscal/...` exige acesso ao Fiscal. Se a tela do Contábil chama
`/api/fiscal/nota-itens` para reusar o detalhamento, um usuário que só tem Contábil
leva 403 no que era pra ser dele. O reuso "óbvio" (chamar o endpoint que já existe)
fura a fronteira de permissão.

## O padrão

Separar as duas coisas que estavam grudadas na rota — a **consulta** e o **gate**:

- A consulta sai para um lib: `buscarNotaItens(...)`.
- Cada módulo tem sua **própria rota fina**, que só chama o lib e é gateada pelo
  seu módulo: `/api/contabil/nota-itens` e `/api/fiscal/nota-itens`, idênticas.
- No cliente, o hook recebe o módulo e monta o caminho: `/api/${modulo}/...`.

O dado é o mesmo; quem o serve é o módulo de quem está pedindo. A query não se
duplica, a permissão não vaza.

## Quando o compartilhado é a feature inteira

O caso acima é uma consulta em dois módulos. A forma se repete inteira quando o
que várias áreas compartilham não é um dado, é uma **tela** — o mesmo formulário
preenchido pelo Fiscal, pelo Contábil, pelo DP e pelo Societário.

A resposta é a mesma, e escala melhor do que parece: **os handlers, os
componentes e as páginas são escritos uma vez**, num lib e numa pasta de
componentes, e cada módulo tem só os arquivos que o roteador exige — reexports
de uma linha.

```ts
// src/app/api/fiscal/post-mortem/route.ts
export { GET, POST } from "@/lib/postmortem-rotas";
```

O handler descobre de quem é o pedido pelo próprio caminho (`/api/<modulo>/...`),
e é isso que fecha o buraco: com um endpoint único, o servidor teria que
descobrir a área por um **parâmetro**, e parâmetro é do cliente. Com o módulo no
caminho, quem pede o post mortem do Fiscal já passou pelo gate do Fiscal antes de
o handler rodar.

Doze arquivos de uma linha soam caros até a alternativa aparecer: quatro cópias
das mesmas telas, e a quinta — quando entrar um setor novo — saindo diferente das
outras.

## A exceção: dado genuinamente transversal

Nem tudo é de um módulo. Um recurso que os dois usam por igual e não pertence a
nenhum — a lista de empresas do filtro — pode ser uma rota compartilhada, liberada
a qualquer sessão. A regra é a dona: dado **de um módulo** vai pela rota do módulo;
dado **de ninguém** pode ser compartilhado. Detalhe de nota é dos dois, mas cada um
serve o seu.

## Conexões
- Princípio: [[Permissão se valida no servidor, não na interface]] · [[O recorte é a área que responde, não o assunto que a tela trata]]
- Depende de: [[Cravar o seam de permissão antes do login]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
