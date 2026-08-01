---
tags: [tipo/atomica, camada/padrao, infra, dev/infra]
criado: 2026-08-01
---

# Volume de dev sobrevive entre versões do projeto e traz schema velho

> O volume Docker nomeado pela slug (`<slug>-db-data`) **persiste entre reescritas
> do projeto**. Um rebuild greenfield sobe apontando pro mesmo volume e encontra as
> tabelas da versão anterior — o `prisma db push` recusa porque as mudanças de schema
> destruiriam linhas existentes.

## O sintoma

`prisma db push` num banco que "devia estar vazio" reclama de colunas obrigatórias
sem default "porque há N linhas na tabela". Essas linhas são da versão antiga do
mesmo projeto: o `docker compose down` não apaga volumes, e o nome do volume é
estável ([[O nome do projeto governa o nome dos recursos]]), então o banco de dev
de ontem reaparece.

## A saída certa

Recriar o volume, não forçar reset no lugar:

```bash
docker compose stop <slug>-db && docker compose rm -f <slug>-db
docker volume rm <slug>-db-data
docker compose up -d <slug>-db      # banco vazio → db push passa liso
```

Num banco vazio o `db push` não tem linha pra destruir e roda sem cerimônia. É mais
honesto que `--force-reset`: recriar o volume deixa explícito que se está jogando
fora o banco de dev, em vez de mascarar a destruição num flag.

## Bônus: o guard de consentimento do Prisma

`prisma db push --force-reset` agora **trava pedindo consentimento humano explícito**
quando detecta que quem chama é uma IA (exige a env
`PRISMA_USER_CONSENT_FOR_DANGEROUS_AI_ACTION`). Recriar o volume por fora contorna
isso e é o caminho limpo mesmo sem o guard — o reset destrutivo fica sendo uma ação
de infra deliberada, não um efeito colateral de um comando de schema.

Regra de fundo: banco de dev é descartável e o volume é que segura o estado — quando
o schema muda de forma incompatível, **a unidade a recriar é o volume, não a tabela**.

## Conexões
- Visto em: [[navetalks]] (rebuild greenfield jul/2026, mesmo slug do app anterior)
- Relacionado: [[O nome do projeto governa o nome dos recursos]] ·
  [[Migrations em container próprio no Docker Compose]] ·
  [[Ambiente de dev sobe igual ao de produção]]
- Mapa: [[Infra]]
