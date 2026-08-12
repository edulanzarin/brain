---
tags: [tipo/atomica, camada/padrao, infra, dev/backend]
criado: 2026-08-12
---

# Agenda recorrente é um serviço do compose, não um crontab do host

> Quem dispara os jobs periódicos do app é um serviço próprio dentro do `docker compose`, que bate as rotas de cron pela rede interna — não um `crontab` configurado à mão no servidor.

## O problema

O app tinha as rotas de job prontas (`/api/rh/cron/experiencia`,
`/api/rh/cron/envios`, protegidas por `RH_CRON_SECRET`), mas **nada as agendava**:
o `.env.example` documentava que era o `crontab do host` que devia batê-las. Isso é
frágil e invisível — depende de alguém lembrar de configurar o cron naquele servidor,
não sobe com o `docker compose up`, e some numa migração de máquina. Do lado do
código não dá nem para confirmar se está rodando.

## A solução

Um serviço a mais no compose, reusando a imagem do app, com entrypoint próprio que
**pula as migrations** (quem migra é o app) e roda um loop em Node que bate as rotas
pela rede interna (`app:3000`):

```yaml
scheduler:
  image: nexo-app
  container_name: nexo-scheduler
  restart: unless-stopped
  entrypoint: ["node", "scripts/scheduler.mjs"]
  env_file: [.env]
  depends_on:
    app: { condition: service_healthy }
```

```js
// scheduler.mjs — sem deps novas: fetch + setInterval
setInterval(() => bater("/api/rh/cron/envios"), 15 * 60_000);      // 15 min
// experiência 1x/dia na virada da hora configurada (checa de minuto em minuto)
```

O segredo (`RH_CRON_SECRET`) vem do mesmo `.env`; as rotas continuam recusando quem
não manda o header. Intervalos ajustáveis por env (`SCHEDULER_*`).

## O que mais vale lembrar

- O disparo automático passa a funcionar só com `docker compose up` — a receita do
  deploy carrega o agendador, não o host.
- Idempotência continua sendo responsabilidade da rota (o slot/ponteiro no servidor),
  não do scheduler — rodar duas vezes não pode duplicar.
- É irmã de [[Migrations em container próprio no Docker Compose]]: o que precisa
  rodar sozinho vira serviço do compose, não passo manual.

## Conexões
- Princípio: [[Ambiente de dev sobe igual ao de produção]]
- Irmã: [[Migrations em container próprio no Docker Compose]]
- Depende de: [[Configuração vem do ambiente, não do código]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Infra]]
