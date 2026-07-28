---
tags: [tipo/atomica, camada/padrao, infra, dev/frontend]
criado: 2026-07-28
---

# App sob subcaminho fica na raiz e o proxy tira o prefixo

> Pra servir o app em `dominio.com/imersao` sem tocar no backend: o proxy
> **tira o prefixo** antes de repassar, e o frontend **deriva tudo de um único
> caminho base**.

## O problema

Já existe um site na raiz do domínio e o app novo tem que entrar num subcaminho
(`/imersao`). A tentação é remontar o app inteiro sob esse prefixo — rotas,
estáticos, endpoints — e espalhar `/imersao` por dezenas de lugares. Cada um que
escapar quebra: asset apontando pra raiz dá 404 (ou colide com o site que já
está lá), o POST do form vai pro caminho errado, a rota de retorno do pagamento
não casa.

## A solução

Duas metades de uma técnica só:

**Backend fica na raiz; o proxy adapta.** O container sobe só em loopback,
montado em `/` e `/api` como sempre. O servidor web da frente faz proxy do
subcaminho **removendo o prefixo** — no nginx, a barra final no `proxy_pass` é o
strip:

```nginx
location = /imersao { return 308 /imersao/; }
location /imersao/ { proxy_pass http://127.0.0.1:4099/; }  # a / final tira /imersao
```

É a mesma forma de [[Porta interna é constante, porta externa é configuração]]:
o interno é constante (raiz), o externo é configuração (o subcaminho), o proxy é
o adaptador. Zero mudança de código no backend.

**Frontend deriva de um caminho base só.** Um build-time `base` (Vite: assado em
`import.meta.env.BASE_URL`) governa três coisas de uma vez, sem `/imersao`
escrito à mão em lugar nenhum:

- assets: um `withBase(path)` prefixa o base em runtime;
- endpoint da API: `` `${base}/api/register` `` em vez de absoluto na raiz;
- roteamento de cliente: casa o pathname já tirando o base
  (`location.pathname.slice(base.length)`).

Trocar o subcaminho é trocar uma variável e rebuildar. As URLs que o MP/back_urls
precisam (`PUBLIC_BASE_URL=https://dominio.com/imersao`) o backend monta a partir
dessa mesma base.

## O que mais vale lembrar

- `VITE_*` (e o base) são **inlined no build**. Mudou o base → rebuild, não
  basta reiniciar. No Docker isso é um `--build-arg`.
- Passe `X-Forwarded-Proto`/`X-Forwarded-For` no proxy: o app atrás de
  `trust proxy` depende deles pro esquema https e o IP real do rate limit.
- Verifique no build de prod que o `index.html` referencia `/imersao/assets/...`
  e que o strip mapeia `/imersao/assets/x.js` → `/assets/x.js` no app.

## Conexões
- Princípio: [[Porta interna é constante, porta externa é configuração]]
- Depende de: [[Configuração vem do ambiente, não do código]]
- Visto em: [[Evento Navecon]]
- Mapa: [[Infra]]
