---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-07-20
---

# Migrations em container próprio no Docker Compose

> Um serviço que **roda, migra e encerra**, com o app dependendo do sucesso
> dele. Assim `docker compose up -d --build` continua sendo o único comando em
> máquina nova, sem inchar a imagem que fica rodando.

## O problema

Rodar `prisma migrate deploy` no entrypoint do app parece óbvio, mas a imagem de
produção do Next é o **standalone** — ela só tem o `node_modules` que o app
realmente importa. O CLI do Prisma não está lá.

Copiar o CLI peça por peça não funciona: `prisma/build/index.js` puxa
`@prisma/config`, que puxa `effect`, `c12`, `empathic`… Cada `COPY` revela a
próxima dependência faltando. É uma escada sem fim.

A saída preguiçosa — copiar o `node_modules` inteiro para o runner — joga fora
todo o ganho do standalone.

## A solução

Um **alvo separado** no mesmo Dockerfile, que tem o `node_modules` completo:

```dockerfile
FROM base AS migrator
COPY --from=deps /app/node_modules ./node_modules
COPY prisma ./prisma
CMD ["docker-migrate.sh"]     # migrate deploy + seed, depois sai
```

E no compose, o app espera esse container **terminar com sucesso**:

```yaml
  migrate:
    build: { context: ., target: migrator }
    restart: "no"
    depends_on: { db: { condition: service_healthy } }

  app:
    build: { context: ., target: runner }
    depends_on:
      migrate: { condition: service_completed_successfully }
```

Se a migration falhar, o app simplesmente não sobe — melhor do que subir contra
um banco em estado errado.

## Detalhe que morde

Não dá para esperar o banco com `prisma migrate status`: ele **sai com código 1
quando há migration pendente**, que é exatamente o caso do primeiro boot. O loop
de espera fica girando até estourar. Use um probe de TCP puro no host/porta do
`DATABASE_URL` (`net.connect` do Node basta) ou o healthcheck do compose.

## Bônus: gerar a migration sem banco

```bash
prisma migrate diff --from-empty --to-schema-datamodel prisma/schema.prisma --script \
  > prisma/migrations/0_init/migration.sql
```

Escreve o SQL inicial **offline**, sem precisar de Postgres no ar. Só não
esqueça do `migrations/migration_lock.toml` com o provider.

## Conexões
- Princípio: [[Ambiente de dev sobe igual ao de produção]]
- Outra armadilha do mesmo Dockerfile: [[Next.js standalone no Docker e o outputFileTracingRoot]]
- Visto em: [[Navedesk]]
- Mapa: [[Infra]]
