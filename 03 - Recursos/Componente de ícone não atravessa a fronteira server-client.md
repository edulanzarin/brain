---
tags: [tipo/atomica, projeto/navedesk, dev/frontend, conceito]
criado: 2026-07-20
---

# Componente de ícone não atravessa a fronteira server-client

> Passar `icone={Ticket}` de um Server Component para um Client Component
> estoura em runtime: componente é função, e função não é serializável.

## O erro

```
Functions cannot be passed directly to Client Components unless you
explicitly expose it by marking it with "use server".
  {$$typeof: ..., render: function, displayName: ...}
```

O `render: function` no meio do objeto entrega o culpado: é um ícone do
lucide (um `forwardRef`), não uma função solta que dá para marcar com
`"use server"`.

## Como me pegou

No [[Navedesk]] havia um único `ui.tsx` com `"use client"` no topo, juntando
componentes interativos (botão de submit, campo de senha) e um `<Vazio>`
puramente visual. As páginas — Server Components — faziam:

```tsx
<Vazio icone={Ticket} titulo="Nenhum chamado encontrado" />
```

Três páginas passaram a dar **500** em produção. O typecheck não viu nada: é
erro de serialização em runtime, não de tipo.

## A regra

**Componente sem interatividade não deve morar num módulo `"use client"`.**
Separar `Vazio` num arquivo próprio, sem a diretiva, resolveu: ele voltou a ser
Server Component e o ícone é resolvido no servidor.

O caminho inverso é seguro: um Client Component pode importar um componente
puramente visual — ele só passa a fazer parte do bundle do cliente.

Quando o componente precisa mesmo ser client e receber um ícone, passe uma
**string** e resolva com um mapa `nome → ícone` do lado do cliente. Foi o que a
sidebar faz: a config de navegação guarda `icone: "Ticket"`, que é
serializável.

## Links

- Descoberto em: [[Navedesk]]
- Faz parte de: [[Desenvolvimento]]
- Ver também: [[Padrões de componentes de dashboard]]
