---
tags: [tipo/atomica, camada/padrao, dev/frontend]
criado: 2026-07-20
---

# NoInfer faz o genérico sair da lista, não do valor padrão

> Quando um hook recebe *valor padrão* e *lista de valores aceitos*, o TypeScript
> infere o tipo do lugar errado — do padrão, que é o menos informativo.

## O quê

Um hook de estado na URL com assinatura ingênua:

```ts
function useUrlState<T extends string>(key: string, fallback: T, allowed?: readonly T[])
```

Chamado como `useUrlState("q", "")`, o `T` fecha no literal `""`. O setter passa
a aceitar **só a string vazia** — qualquer busca digitada vira erro de tipo:

```
Argument of type 'string' is not assignable to parameter of type '""'.
```

`NoInfer` (TS 5.4+) tira o argumento da inferência:

```ts
function useUrlState<T extends string = string>(
  key: string,
  fallback: NoInfer<T>,
  allowed?: readonly T[],
)
```

Agora o `T` sai de `allowed` quando existe (`"all" | "expiring" | …`) e cai no
default `string` quando não existe. Os dois usos funcionam sem anotar nada.

## Por que importa

O sintoma engana: o erro aponta para a *chamada*, e a reação é anotar o tipo em
cada uso. O problema é a assinatura deixando um argumento fraco mandar na
inferência. `NoInfer` é a forma direta de dizer qual argumento manda.

O padrão aparece sempre que a mesma variável de tipo é servida por duas fontes
de qualidade diferente — valor inicial vs conjunto de opções, item vs array.

## Conexões
- Princípio: nenhum ainda — folha isolada de tipagem
- Ver também: [[Portal condicional dispensa o flag de montagem]]
- Visto em: [[Cofre Digital]], no hook de [[Filtro de lista mora na URL]]
- Mapa: [[Frontend]]
