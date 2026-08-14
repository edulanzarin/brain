---
tags: [tipo/atomica, camada/padrao, infra, dev/backend, armadilha]
criado: 2026-08-13
---

# Prisma 7 tira a URL do schema, vai pro config e pro adapter

No Prisma 7 o velho `datasource db { url = env("DATABASE_URL") }` **para de
compilar** (erro P1012: "the datasource property `url` is no longer supported in
schema files"). A conexão foi partida em dois lugares, por camada:

- **Migrate/CLI** lê a URL do `prisma.config.ts`:

  ```ts
  export default defineConfig({
    schema: path.join("prisma", "schema.prisma"),
    datasource: { url: process.env.DATABASE_URL },
    migrations: { seed: "tsx prisma/seed.ts" },
  });
  ```

- **Runtime** passa a URL por um **driver adapter** no construtor do client, não
  mais pela datasource:

  ```ts
  import { PrismaPg } from "@prisma/adapter-pg";
  const prisma = new PrismaClient({
    adapter: new PrismaPg({ connectionString: process.env.DATABASE_URL }),
  });
  ```

No schema, o bloco `datasource` fica só com o `provider`.

## O .env não é mais carregado sozinho

O Prisma 7 não injeta o `.env` automaticamente. Duas saídas: `dotenv`, ou —
melhor no Node 22+ — `process.loadEnvFile()` no topo do `prisma.config.ts` e de
qualquer script (`seed`, simuladores) que rode fora do Next. Dispensa dependência.

```ts
try { process.loadEnvFile(); } catch { /* já veio do ambiente */ }
```

## O client gerado mudou de forma

O generator novo (`provider = "prisma-client"`, com `output`) emite **`.ts`**
(não JS compilado) e **sem `index`**: o import é do `client`, não da pasta.

```ts
import { PrismaClient } from "@/generated/prisma/client";
```

## Conexões
- Irmã: [[Porta interna é constante, porta externa é configuração]]
- Visto em: [[Idle Game]]
- Mapa: [[Infra]] · [[Dados]]
