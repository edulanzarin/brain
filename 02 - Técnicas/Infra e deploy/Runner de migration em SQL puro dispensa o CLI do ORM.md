---
tags: [tipo/atomica, camada/padrao, dev/backend, infra]
criado: 2026-08-04
---

# Runner de migration em SQL puro dispensa o CLI do ORM

> Sem ORM (SQL direto com `pg`), a migration não precisa de um CLI pesado no
> container: um **script de ~30 linhas** aplica os `.sql` em ordem e anota o que já
> rodou. Some o problema de encaixar o CLI do Prisma na imagem standalone.

## O contexto

Com Prisma, migrar no boot exige o CLI do ORM, que não vive na imagem standalone do
Next — daí a escada de dependências e o container de migrate só pra ter o
`node_modules` inteiro ([[Migrations em container próprio no Docker Compose]]). Quando o
projeto usa `pg` cru, esse peso some: `pg` já está no `node_modules` do app.

## O runner

Uma tabela de controle e um loop idempotente:

```
CREATE TABLE IF NOT EXISTS schema_migrations (
  id text PRIMARY KEY, aplicado_em timestamptz DEFAULT now()
);
```

Lê `db/migrations/*.sql` em ordem, pula os já registrados e aplica cada um **numa
transação** (BEGIN → SQL → INSERT no controle → COMMIT; ROLLBACK no erro). O seed é
outro script idempotente (pula se a org já existe). O container de migrate ainda é
próprio e o app depende do sucesso dele — a **forma** de
[[Migrations em container próprio no Docker Compose]] continua; o que muda é que o
"migrator" é trivial e não arrasta CLI nenhum.

## A troca

Você escreve o SQL da migration à mão (e o rollback, se quiser) em vez de gerar do
schema. É o custo de não ter ORM — que é justamente a intenção quando se escolhe `pg`
direto pela transparência do SQL.

## Conexões
- Princípio: [[Ambiente de dev sobe igual ao de produção]]
- Irmã: [[Migrations em container próprio no Docker Compose]]
- Visto em: [[Navehub]]
- Mapa: [[Infra]]
