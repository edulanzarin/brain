---
tags: [tipo/atomica, camada/padrao, infra, docker, armadilha]
criado: 2026-09-25
---

# Agendador em container conta as horas em UTC

> Um agendador em Node que dispara "às 8h" com `new Date().getHours() === 8` roda, dentro do container, em UTC. No Brasil ele dispara às 5h, três horas antes, e nada no log parece errado: a linha diz "ok" na hora em que rodou.

## O problema

O serviço de agendamento do compose (`node scripts/scheduler.mjs`) marcava os e-mails do RH e do DP para as 8h e a varredura do Acessórias para as 5h. A imagem `node:22-alpine` nasce sem fuso, o processo herda UTC, e o `getHours()` responde a hora de Greenwich. Os avisos das 8h saíam às 5h da manhã e a varredura das 5h às 2h.

Ninguém reclamou porque o efeito não parece defeito: o e-mail chega, só que de madrugada, e a varredura termina antes do expediente do mesmo jeito.

## A solução

Fixar o fuso no serviço, pelo ambiente:

```yaml
navex-scheduler:
  environment:
    TZ: America/Sao_Paulo
```

Na alpine isso basta **sem instalar `tzdata`**: o Node traz a base de fusos dentro do ICU dele. Conferido na própria imagem:

```bash
docker run --rm --entrypoint node navex-app -e "console.log(new Date().toString())"
# ... GMT+0000 (Coordinated Universal Time)
docker run --rm -e TZ=America/Sao_Paulo --entrypoint node navex-app -e "console.log(new Date().toString())"
# ... GMT-0300 (Brasilia Standard Time)
```

## O que mais vale lembrar

- Vale para qualquer processo que compare hora local: cron em Node, "fechar o dia à meia-noite", nome de arquivo com a hora. O Postgres do container tem a mesma origem e o mesmo sintoma, na nota irmã.
- O fuso é do **serviço que agenda**, não da máquina que hospeda. O host pode estar em Brasília e o container não herda nada dele.
- A conferência é uma linha. O log de partida deve dizer a hora local, ou `docker exec <agendador> date` a mostra.

## Conexões
- Princípio: [[Configuração vem do ambiente, não do código]]
- Irmã: [[Postgres de container nasce em UTC, e a hora formatada sem fuso mente]] · [[Agenda recorrente é um serviço do compose, não um crontab do host]]
- Visto em: [[NaveX]] (herdado do agendador do nexo2)
- Mapa: [[Infra]]
