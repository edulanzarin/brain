---
tags: [tipo/atomica, camada/padrao, infra]
criado: 2026-08-17
---

# Railway não roda compose, cada serviço vira uma peça da plataforma

> O `docker-compose.yml` não viaja pro Railway: lá cada serviço é deployado
> separado, e cada container do compose precisa de um equivalente na plataforma —
> senão a peça simplesmente não existe em produção.

## O problema

O chassi padrão ([[Ambiente de dev sobe igual ao de produção]]) descreve a stack
inteira no compose: app, banco, container de migration e workers auxiliares. O
Railway ignora o compose — ele builda **um** Dockerfile por serviço. Quem sobe a
stack "só apontando o repo" ganha um app rodando contra um banco **sem schema** e
com os workers do compose silenciosamente inexistentes.

## A solução

Mapear peça por peça:

| No compose | No Railway |
|---|---|
| `<slug>-db` (Postgres) | plugin Postgres gerenciado; `DATABASE_URL` por referência (`${{Postgres.DATABASE_URL}}`, URL privada, sem SSL) |
| `<slug>-migrate` (container próprio) | **pre-deploy command** (`node db/setup.mjs`) rodando na própria imagem do app — a imagem final precisa **carregar `db/`** (no multi-stage, o runner não o copia por padrão) |
| worker auxiliar (loop curl etc.) | vira **loop interno no processo do app** (no Next: `instrumentation.ts`), o mesmo mecanismo em dev, compose e Railway |
| `ports:` externas | Railway injeta `PORT`; porta interna constante já resolve |

Config as code em `railway.json`: `preDeployCommand`, e **`numReplicas: 1` travado
quando o app tem estado singleton por processo** (sessão viva, keepalive) — réplica
dupla disputaria o estado.

## O que mais vale lembrar

- Mover o worker pra dentro do processo não é gambiarra de plataforma: **melhora a
  paridade** (um mecanismo só em todo ambiente) e o compose fica menor. O endpoint
  HTTP do job pode continuar existindo como gatilho manual autenticado.
- Env obrigatória se valida **no boot** (fail-fast com mensagem), não na primeira
  request — em PaaS a env esquecida é o modo de falha mais comum do primeiro deploy.

## Conexões
- Princípio: [[Ambiente de dev sobe igual ao de produção]]
- Irmã: [[Migrations em container próprio no Docker Compose]] · [[Agenda recorrente é um serviço do compose, não um crontab do host]]
- Visto em: [[piwdex]]
- Mapa: [[Infra]]
