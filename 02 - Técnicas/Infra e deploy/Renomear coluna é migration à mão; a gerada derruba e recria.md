---
tags: [tipo/atomica, camada/padrao, dev/backend, infra]
criado: 2026-08-30
---

# Renomear coluna é migration à mão; a gerada derruba e recria

> O ORM compara dois schemas, não lê intenção. Coluna que troca de nome aparece
> pra ele como uma que sumiu e outra que nasceu — e a migration gerada apaga o
> conteúdo da primeira.

## O problema

No [[Privello]], o endereço público do anúncio deixou de sair do nome e passou a
ser um @ escolhido: a coluna `slug` virava `arroba`. O `prisma migrate dev`
entendeu duas operações independentes:

```
• Added the required column `arroba` ... There are 117 rows in this table
• You are about to drop the column `slug`, which still contains 117 non-null values
```

Ele estava certo sobre o schema e errado sobre o mundo: o conteúdo daquela
coluna era o endereço de 117 anúncios que já estavam no ar, e link publicado não
volta — [[Identificador que já circulou não é mais seu para mudar]].

## A solução

Gerar o arquivo vazio e escrever o SQL, dizendo o que o diff não consegue
adivinhar. `RENAME` preserva o dado, e o índice único precisa ser renomeado
junto, senão o ORM vê um nome de índice que ele não geraria e propõe recriá-lo
na próxima migration:

```sql
ALTER TABLE "anuncios" RENAME COLUMN "slug" TO "arroba";
ALTER INDEX "anuncios_slug_key" RENAME TO "anuncios_arroba_key";

-- transformação de conteúdo mora aqui também, não num script de fora
UPDATE "anuncios" SET "arroba" = replace("arroba", '-', '_');
```

Vale o mesmo para coluna nova com significado: ao acrescentar prazo a um campo
que já tinha texto, a migration preenche o prazo das linhas existentes. Sem
isso, a primeira leitura depois da virada trata todo registro antigo como
vencido e ele some calado.

## O que mais vale lembrar

Em ambiente não interativo o `migrate dev` recusa e não escreve nada: escrever a
pasta e o `migration.sql` na mão e aplicar com `migrate deploy` é o caminho.

## Conexões
- Princípio: [[Identificador que já circulou não é mais seu para mudar]]
- Irmã: [[Registro que muda de casa leva junto o token já distribuído]] ·
  [[Nome de migration do Prisma é UTC, e é o nome que ordena]]
- Ver também: [[Migração de dados mantém o antigo como reserva até a virada]]
- Visto em: [[Privello]]
- Mapa: [[Infra]] · [[Dados]]
