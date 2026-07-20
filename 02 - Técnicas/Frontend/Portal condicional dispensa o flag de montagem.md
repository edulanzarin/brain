---
tags: [tipo/atomica, camada/padrao, dev/frontend]
criado: 2026-07-20
---

# Portal condicional dispensa o flag de montagem

> Se o portal só tem conteúdo depois de uma ação do usuário, ele já é seguro no
> SSR — o `mounted` que todo mundo escreve é redundante.

## O quê

`createPortal` precisa do `document`, que não existe no servidor. O reflexo é:

```tsx
const [mounted, setMounted] = useState(false);
useEffect(() => setMounted(true), []);   // <- lint reclama, com razão
if (!mounted) return null;
```

Mas quando o componente já tem um estado vazio natural, ele resolve sozinho:

```tsx
if (toasts.length === 0) return null;    // no servidor, sempre
return createPortal(...)
```

No servidor a pilha está sempre vazia, então o `createPortal` nunca roda lá. O
primeiro item só nasce de uma interação — ou seja, já hidratado.

## Por que importa

O `setState` dentro de `useEffect` é justamente o que a regra
`react-hooks/set-state-in-effect` marca: causa render em cascata e existe só
para contornar o SSR. Vale procurar o estado vazio antes de aceitar o flag.

Não serve para tudo: portal que precisa aparecer **já no primeiro paint** (um
layout fixo, um header em portal) não tem estado vazio natural e aí o flag —
ou um componente client-only — continua sendo a saída.

## Conexões
- Princípio: [[Todo estado da tela tem visual]]
- Ver também: [[Toast em vez de alert para o feedback do app]] · [[Componente de ícone não atravessa a fronteira server-client]]
- Visto em: [[Cofre Digital]], montando o toaster
- Mapa: [[Frontend]]
