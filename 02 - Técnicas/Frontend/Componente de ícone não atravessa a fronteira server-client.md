---
tags: [tipo/atomica, camada/padrao, dev/frontend]
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


## Aconteceu de novo, e não era ícone

No [[Navetech Hub]] a armadilha voltou com outra roupa: a barra lateral (cliente)
passou a receber a ARTE do módulo como prop, vinda do layout (servidor) —
`arte={ArteAdmin}`. Mesmo erro, mesma mensagem, e a tela inteira da Administração
abriu em "This page couldn't load".

Dois aprendizados que a segunda vez deixou claros:

- **Não é sobre ícone, é sobre COMPONENTE.** Qualquer função cruzando a fronteira
  quebra — ícone, arte, renderizador, formatador passado como prop.
- **O build não avisa.** `next build` compilou e passou no type check; o defeito
  só existe em runtime, e só na rota que monta aquele componente. Rota que ninguém
  abriu no teste é rota que ninguém sabe que está quebrada.

O remédio é sempre o mesmo: **atravesse com uma CHAVE e resolva do lado do
cliente.** O registro por nome que já existe para o ícone serve igual para a arte
— `ARTE_POR_CHAVE[chave]` dentro do componente de cliente, e a prop vira string.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
- Ver também: [[Padrões de componentes de dashboard]]
- Visto em: [[Navedesk]] · [[Navetech Hub]]
- Mapa: [[Frontend]]
