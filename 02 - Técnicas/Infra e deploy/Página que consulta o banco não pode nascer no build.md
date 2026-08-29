---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-29
---

# Página que consulta o banco não pode nascer no build

> O build roda dentro do Docker, onde não existe banco. Qualquer rota que o Next
> decida pré-renderizar e que faça uma query **derruba o build inteiro** — e o
> erro só aparece no container, nunca na máquina onde o banco está de pé.

## O problema

`npm run build` na máquina de dev passa. O mesmo build dentro do `docker
compose up --build` falha:

```
Error occurred prerendering page "/sitemap.xml"
Invalid `prisma.perfil.groupBy()` invocation:
  code: 'ECONNREFUSED'
Export encountered an error on /sitemap.xml/route, exiting the build.
```

A diferença não é o código, é a **companhia**: em dev o Postgres está publicado
no host e o build o alcança sem querer. Dentro da imagem não há ninguém na outra
ponta.

O que engana é que quase tudo passa. Página com `auth()`, `cookies()` ou
`searchParams` já é dinâmica por consequência, então nunca é pré-renderizada. As
que mordem são as que **não tocam em nada de request**: `sitemap.ts`,
`robots.ts`, uma home sem sessão, uma listagem estática. Justamente as mais
inocentes.

## A regra

Rota que lê do banco declara que é de request:

```ts
// src/app/sitemap.ts
export const dynamic = "force-dynamic";
```

`revalidate` **não** resolve: com ISR o Next ainda gera a primeira versão no
build, então ele continua precisando do banco lá.

## Por que force-dynamic e não um banco no build

Dá para subir um Postgres no builder, ou passar uma `DATABASE_URL` real. Os dois
são piores:

- **O conteúdo congela na data do deploy.** Um sitemap assado na imagem segue
  anunciando as cidades que existiam quando ela foi construída. O bug não é o
  build quebrado, é o build que passa e mente.
- Amarra o build a um serviço externo, e build tem que ser reprodutível numa
  máquina sem nada.

Sitemap é pedido por robô algumas vezes ao dia. A consulta por request não é o
gargalo de ninguém.

## O que mais vale lembrar

- O `DATABASE_URL` ainda precisa **existir** no build, mesmo apontando para o
  vazio: módulo que instancia o client no import exige a variável — ver
  [[Next.js standalone no Docker e o outputFileTracingRoot]].
- O sintoma "passa aqui, quebra no container" tem sempre a mesma raiz: o build é
  outro ambiente. Vale para banco, para arquivo em disco e para variável.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
- Irmã: [[Next.js standalone no Docker e o outputFileTracingRoot]] · [[Ambiente de dev sobe igual ao de produção]]
- Visto em: [[Privello]]
- Mapa: [[Infra]]
