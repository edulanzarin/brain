---
tags: [tipo/atomica, camada/padrao, seguranca, sql, armadilha]
criado: 2026-09-01
---

# Contexto de tenant tem que morrer no commit, senão o pool o carrega adiante

> A variável de sessão que diz de qual cliente é a requisição precisa ser escrita com
> escopo de **transação**. Escrita com escopo de sessão, ela fica gravada na conexão,
> a conexão volta para o pool, e o próximo pedido — de outro cliente — lê os dados do
> anterior.

## O erro que evita

É a falha que a política de isolamento não pega, porque do ponto de vista do banco
está tudo certo: existe contexto, a política foi obedecida, as linhas devolvidas
pertencem mesmo ao cliente do contexto. O contexto é que é do cliente errado.

Pior: o defeito é **intermitente por construção**. Depende de qual conexão do pool o
pedido pegou, e sob carga baixa — um desenvolvedor sozinho testando — quase nunca
acontece. Ele aparece em produção, com dois clientes usando ao mesmo tempo.

## A costura

No Postgres, o terceiro argumento de `set_config` é o que decide, e ele é `local`:

```ts
await cliente.query("BEGIN");
// o `true` faz o valor evaporar no COMMIT. Sem ele, fica na conexão.
await cliente.query("SELECT set_config('app.escritorio_id', $1, true)", [id]);
// ...toda a operação acontece aqui dentro
await cliente.query("COMMIT");
```

O equivalente em SQL escrito à mão é `SET LOCAL`, nunca `SET`.

Como consequência, **toda leitura passa a acontecer dentro de uma transação**, mesmo a
que só lê. Não é desperdício: é o que dá à leitura o mesmo escopo que já se exige da
escrita.

## Como provar que está certo

A prova não é ler o código, é atravessar:

1. sem contexto, a consulta devolve zero linhas;
2. com contexto, devolve só as do próprio cliente;
3. gravar no cliente vizinho é recusado;
4. **depois do commit, a mesma conexão volta a devolver zero** — este é o teste desta
   nota, e é o que reprova o `SET` sem `LOCAL`;
5. o papel da aplicação não consegue desligar a política.

Um script que roda essas cinco em minutos vale mais que a leitura atenta, porque a
quarta passa despercebida em revisão.

## Conexões
- Princípio: [[Permissão se valida no servidor, não na interface]]
- Irmã: [[Isolamento entre clientes é política do banco, não filtro na query]] · [[Uma conexão do pg não atende duas consultas ao mesmo tempo]]
- Depende de: [[Configuração vem do ambiente, não do código]]
- Visto em: [[navecrm]]
- Mapa: [[Dados]]
