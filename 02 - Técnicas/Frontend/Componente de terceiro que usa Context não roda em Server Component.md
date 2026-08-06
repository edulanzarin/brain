---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-06
---

# Componente de terceiro que usa Context não roda em Server Component

> Server Component não tem hook. Um componente de biblioteca que lê Context por
> dentro (useContext) quebra ao renderizar no servidor — procure a variante
> SSR/context-free.

## Como me pegou

No [[Navehub]] troquei os ícones (lucide -> Phosphor) e o build quebrou no painel,
que é Server Component. Causa: os ícones Phosphor "normais" (CSR) leem um
`IconContext` via `useContext` pra herdar peso/tamanho — e hook não existe em
Server Component.

É diferente de [[Componente de ícone não atravessa a fronteira server-client]]:
aquilo é passar o componente como **prop** através da fronteira (serialização);
aqui é o componente **usar Context por dentro** ao ser renderizado no servidor.

## A saída

A lib expõe `@phosphor-icons/react/ssr`: os mesmos ícones, sem Context, seguros em
server e client. Importar por `/ssr` resolve — sem precisar marcar o consumidor
como `"use client"` (o que mataria o SSR do painel).

Regra geral: **antes de usar um componente de terceiro dentro de um Server
Component, veja se ele usa hook/Context.** Se usa, ou existe variante
SSR/context-free, ou o consumidor vira client (perdendo o SSR).

Desacoplar de quebra: um **módulo central de ícones** que re-exporta com aliases
(`export { Buildings as Building2 } from ".../ssr"`) deixa o resto do app importar
nomes estáveis — trocar de biblioteca vira mudar um arquivo só.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
- Irmã: [[Componente de ícone não atravessa a fronteira server-client]]
- Visto em: [[Navehub]]
- Mapa: [[Frontend]]
