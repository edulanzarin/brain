---
tags: [tipo/atomica, camada/padrao, armadilha, infra]
criado: 2026-08-30
---

# Nome de migration do Prisma é UTC, e é o nome que ordena

> A pasta da migration é carimbada em UTC. Criar uma à mão com a hora local
> produz um nome que ordena **antes** das que já existem — e o banco que nascer
> do zero roda na ordem errada.

## O problema

Escrevendo a migration à mão no fuso `-03`:

```
$ date +%Y%m%d%H%M%S   →  20260830030846      (local)
$ date -u +...         →  20260830060846      (UTC, é este)
```

As migrations anteriores iam até `20260830054210`. A pasta `...030846` foi
aplicada sem reclamar — o banco de dev só marca o que já rodou — e o defeito
fica dormindo: num banco novo, `migrate deploy` roda em ordem de nome, e a
migration que **renomeia** uma coluna vai antes da que a **cria**. Falha só na
primeira subida de um ambiente limpo, que é exatamente o que a produção faz.

## A solução

Carimbar em UTC. Se já aplicou com o nome errado, renomear a pasta e acertar a
linha do controle, senão o Prisma vê uma migration desconhecida no disco e uma
aplicada que sumiu:

```sql
UPDATE _prisma_migrations
   SET migration_name = '20260830060846_arroba_e_recado_de_24h'
 WHERE migration_name = '20260830030846_arroba_e_recado_de_24h';
```

`prisma migrate status` confirma que os dois lados voltaram a falar do mesmo.

## Conexões
- Princípio: [[Ambiente de dev sobe igual ao de produção]] — o erro é invisível
  no banco que já existe e só aparece no que nasce do zero.
- Irmã: [[Renomear coluna é migration à mão; a gerada derruba e recria]]
- Visto em: [[Privello]]
- Mapa: [[Infra]]
