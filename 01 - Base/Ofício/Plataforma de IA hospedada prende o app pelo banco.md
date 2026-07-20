---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-07-19
---

# Plataforma de IA hospedada prende o app pelo banco

Plataformas que geram **e hospedam** o app (Abacus.AI, Replit, Lovable e afins)
entregam o código junto — o que dá a sensação de que sair é só rodar `docker
build`. Não é. O código é a parte portátil; o que prende é a **infra implícita**.

No [[Navecon Controller]] o bloqueio real foi o banco: o `DATABASE_URL` apontava
para `db-xxxx.hosteddb.reai.io`, que resolve para `172.21.254.219` — **IP
privado**. Só existe dentro da rede deles. Não é firewall que se pede pra abrir:
o endereço não é roteável na internet. Sem migrar o banco, o app não sobe em
lugar nenhum.

## O teste de 30 segundos

Antes de estimar qualquer porte, resolva o host do banco:

```bash
getent hosts <host-do-banco>
```

Caiu em `10.x`, `172.16-31.x` ou `192.168.x` → o banco é inalcançável de fora, e
migração de dados vira pré-requisito, não fase 2. Isso muda o tamanho do trabalho.

## O que mais fica implícito

Vale procurar cada um antes de começar:

- **Variáveis injetadas pela plataforma.** No Navecon, `NEXTAUTH_URL` não estava
  no `.env` mas era usada em 17 pontos — a plataforma injetava. Fora dela, login
  redireciona errado e link de e-mail sai quebrado. Grepar as env vars usadas no
  código e comparar com o `.env` acha essas.
- **Credencial por IAM role.** `new S3Client({})` sem credencial funcionava pela
  role da máquina. No container não há role: quebra em silêncio, só no upload.
- **Scheduler.** Rotas `api/cron/*` protegidas por segredo não têm quem as chame.
- **Caminhos absolutos da máquina deles.** `output = "/home/ubuntu/..."` no
  schema Prisma, script de backup com `/home/ubuntu` fixo.
- **Gerenciador de pacotes tolerante.** Yarn deixava passar conflito de peer dep
  que o npm barra (`--legacy-peer-deps` como saída).
- **Sem lockfile**, então o build não é reprodutível até você gerar um.

## A saída de dados

Se o banco é inalcançável, o `pg_dump` de fora também é. O caminho é exportar de
**dentro** da plataforma — no Navecon havia um `backup_db.sh` agendado lá,
deixando dumps na pasta do projeto. Foi o que salvou.

Dump restaurado quase sempre está **atrás do código**: aqui faltavam 4 colunas
que o schema Prisma já tinha. `prisma migrate diff` mostra a defasagem antes de
aplicar — se for só aditivo, `db push` resolve; se tiver DROP, é migração de
verdade.

## Conexões
- Visto em: [[Navecon Controller]]
- Empacotamento: [[Next.js standalone no Docker e o outputFileTracingRoot]]
- Mesma lição de fundo: [[Verificar no build de produção, não só em dev]]
- Mapa: [[Base]]
