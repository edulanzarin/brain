---
tags: [tipo/atomica, camada/principio, infra, docker]
criado: 2026-07-20
---

# Porta interna é constante, porta externa é configuração

> Dentro do container a aplicação sempre escuta na mesma porta. Qual porta o mundo
> enxerga é decisão de quem sobe, não do código.

## A regra

**Dentro**, sempre 3000 (app) e 5432 (Postgres). Igual em todo projeto, igual em dev
e em produção. Nada no código, no Dockerfile ou no compose depende de qual porta é.

**Fora**, uma variável:

```yaml
services:
  app:
    container_name: prospects-app
    ports: ["${APP_PORT:-4011}:3000"]
  db:
    container_name: prospects-db
    ports: ["${DB_PORT:-5411}:5432"]
```

E no `package.json`, **um** script só, com a porta vindo do ambiente e um padrão:

```json
{ "scripts": { "dev": "next dev -p ${PORT:-4011}" } }
```

Aí `npm run dev` sobe em 4011 e `PORT=3000 npm run dev` sobe em 3000. Nada de
`dev:3000`, `dev:3001`, `dev:4011` — script por porta é a escala errada: um script novo
por porta que eu inventar, e nenhum deles diz qual é a porta de verdade do projeto.

## Por que

Foi o problema concreto: rodar vários projetos ao mesmo tempo obrigava a editar a porta
no código, e editar a porta no código mexe no que vai pro Docker. Separando as duas,
some o conflito — o container não sabe nem se importa em que porta o host publicou ele.

Isso é um caso particular de [[Configuração vem do ambiente, não do código]]: porta é
ambiente, não é lógica.

## A parte que confunde

Mudar a porta externa **não** exige mexer no Docker. `4011:3000` e `3000:3000` rodam a
mesma imagem, sem rebuild. O que precisa acompanhar é o que aponta pra ela de fora:
`DATABASE_URL` (que dentro da rede do compose usa `prospects-db:5432`, o nome do serviço
e a porta *interna*, nunca a externa), URL de callback de OAuth e proxy reverso.

Dentro da rede do compose, um container fala com o outro por nome de serviço e porta
interna. A porta publicada existe só pro host.

## Conexões
- Depende de: [[Configuração vem do ambiente, não do código]]
- Irmã: [[O nome do projeto governa o nome dos recursos]] · [[Uma faixa de portas por projeto]]
- Mapa: [[Base]] · [[Infra]]
