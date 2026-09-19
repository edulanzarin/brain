---
tags: [tipo/atomica, camada/padrao, infra, docker]
criado: 2026-09-19
---

# Sem Docker na máquina, a imagem se confere montando o standalone numa pasta limpa

> Quando a máquina de desenvolvimento não roda Docker, o `Dockerfile` e o compose
> ficam sem execução até o primeiro deploy. Dá para conferir quase tudo antes:
> montar numa pasta vazia exatamente o que o estágio final da imagem copia, e
> subir de lá do jeito que o container sobe.

## O problema

Sem virtualização na BIOS não há Docker
([[Sem virtualização na BIOS não há Docker no Windows; o banco de dev vira Postgres portátil]]),
e o compose do projeto vira texto não executado. O risco não está no YAML — está
no que a imagem **deixa de carregar**. Os defeitos típicos são todos de ausência:
`static` e `public` fora do standalone (página 200 sem CSS), script que importa um
pacote que o tracing não levou, arquivo que só existia porque o `.env` estava no
disco.

Rodar `npm run start` na pasta do projeto não pega nenhum desses: lá está tudo, o
`node_modules` inteiro e o `.env` junto.

## A solução

Reproduzir o último estágio do `Dockerfile` à mão, numa pasta vazia, e rodar de
lá **sem `.env`**:

```bash
IMG=<pasta temporária vazia>
cp -r .next/standalone/. "$IMG/"
cp -r .next/static "$IMG/.next/static"
cp -r public "$IMG/public"
cp -r migrations "$IMG/migrations"
cp scripts/migrate.mjs scripts/agenda.mjs "$IMG/scripts/"
rm -f "$IMG"/.env*                       # a imagem não leva .env

cd "$IMG"
APP_DB_URL=... node scripts/migrate.mjs  # o pg resolve do node_modules do standalone?
PORT=... HOSTNAME=0.0.0.0 NODE_ENV=production <variáveis> node server.js
```

E então: a sonda de saúde responde, a página vem com CSS (buscar o `.css` do HTML
e conferir o 200), a rota protegida redireciona, o agendador faz uma rodada. Cada
um desses é uma cópia do `Dockerfile` sendo posta à prova.

A lista de `cp` **é** o `Dockerfile`. Se ela diverge do arquivo, o teste mente —
por isso a ordem certa é escrever o `Dockerfile` primeiro e copiar dele.

## O que isto não cobre

A camada do Docker propriamente dita: rede do compose, resolução de nome entre
serviços, `depends_on` com healthcheck, permissão do usuário não-root, `wget`
existir na imagem alpine para o healthcheck. A primeira subida num servidor ainda
é a primeira execução do compose, e isso tem que ser dito, não escondido.

## A armadilha que o teste revela: o .env de dev dentro do container

O `env_file: .env` do compose entrega ao container as variáveis escritas para o
desenvolvimento local — `PORT=4081` e o banco em `localhost:5081`. Lido cru, o
Next escuta na porta errada dentro do container e o app procura o banco em si
mesmo. O bloco `environment:` do serviço **vence** o `env_file`, e é nele que
`PORT`, `HOSTNAME` e o endereço do banco (`<slug>-db:5432`) precisam ser
sobrescritos. Subir o standalone "só com variáveis" é o que obriga a listar quais
são essas.

## Conexões
- Princípio: [[Ambiente de dev sobe igual ao de produção]] · [[Verificar no build de produção, não só em dev]]
- Irmã: [[Next.js standalone no Docker e o outputFileTracingRoot]] · [[Agenda recorrente é um serviço do compose, não um crontab do host]] · [[Porta interna é constante, porta externa é configuração]]
- Visto em: [[telebot]]
- Mapa: [[Infra]]
