---
tags: [tipo/atomica, projeto/navecon-controller, dev/backend, conceito]
criado: 2026-07-19
---

# Next.js standalone no Docker e o outputFileTracingRoot

`output: 'standalone'` faz o Next emitir `.next/standalone/` com um `server.js` e
só o `node_modules` que o app realmente usa. É o jeito certo de containerizar:
imagem pequena, sem `next start`, sem dependência de dev.

A pegadinha é o **layout da saída**, que depende de `outputFileTracingRoot`.

## A regra

O standalone reproduz, a partir do tracing root, a árvore de diretórios até o
app. Se o root é a **pasta pai** do projeto:

```js
experimental: { outputFileTracingRoot: path.join(__dirname, '../') }
```

então o `server.js` **não** fica em `.next/standalone/server.js`, e sim em
`.next/standalone/<nome-da-pasta-do-app>/server.js`. Copiar o standalone e rodar
`node server.js` na raiz falha com "Cannot find module".

A solução é o Dockerfile espelhar a mesma estrutura de pastas do projeto:

```dockerfile
WORKDIR /app/nextjs_space      # subpasta, igual ao projeto real
...
COPY --from=builder /app/nextjs_space/.next/standalone ./
COPY --from=builder /app/nextjs_space/.next/static  ./nextjs_space/.next/static
COPY --from=builder /app/nextjs_space/public        ./nextjs_space/public
WORKDIR /app/nextjs_space
CMD ["node", "server.js"]
```

Mais simples do que reescrever o `next.config.js` — mexer no tracing root pode
deixar de incluir arquivos que o app usa.

## Outros detalhes que mordem

- **`static/` e `public/` não vão no standalone.** É comportamento esperado, não
  bug: são servidos por CDN em muitos deploys. Sem copiar à mão, o app sobe com
  CSS e imagens quebrados — 200 na página, visual destruído.
- **`HOSTNAME=0.0.0.0`** no container. O default escuta em localhost e o port
  mapping do Docker não alcança.
- **Prisma no debian-slim precisa de `openssl`** instalado, senão o engine não
  carrega. E `binaryTargets` tem que bater com a imagem — `native` resolve.
- **O build precisa do `DATABASE_URL` presente**, mesmo sem conectar: módulos que
  instanciam o `PrismaClient` no import exigem a variável. Um valor fake no
  `RUN` do build serve.
- `output` lido de env (`process.env.NEXT_OUTPUT_MODE`) deixa o mesmo config
  servir dev e imagem — só setar no builder.

## Links

- Usado em: [[Navecon Controller]]
- Contexto da migração: [[Plataforma de IA hospedada prende o app pelo banco]]
- Outra armadilha de build do Next: [[router.replace do Next falha no build de produção]]
