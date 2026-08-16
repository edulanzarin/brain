---
tags: [tipo/atomica, camada/padrao, dev/backend, infra]
criado: 2026-08-16
---

# Auth.js sem adapter, a sessão JWT resolve o usuário no seu próprio SQL

> Com Auth.js v5 e `session: { strategy: "jwt" }` **não existe tabela de sessão nem
> adapter de ORM**. Os providers só autenticam; quem resolve/cria o usuário é o SEU
> SQL, dentro dos callbacks. Assim o login convive com `pg` cru — sem arrastar o
> adapter do Prisma pra manter a transparência do SQL.

## O contexto

O caminho "oficial" do Auth.js é plugar um adapter (`@auth/pg-adapter`, Prisma…) que
cria e possui as tabelas `users`/`accounts`/`sessions`. Isso reintroduz justamente o
peso que [[Runner de migration em SQL puro dispensa o CLI do ORM]] tira: schema que
não é seu, e no caso do Prisma o CLI de volta na imagem
([[Prisma 7 tira a URL do schema - vai pro config e pro adapter]]). Num projeto que
escolheu `pg` direto, o adapter briga com o resto.

## A solução

Sessão **JWT** (stateless, sem store) + os callbacks fazendo o trabalho de banco:

- **Credentials**: `authorize` valida email/senha (`bcrypt.compare`) contra a sua
  tabela `users` e retorna `{ id }` — o id do SEU banco.
- **OAuth (Google)**: no callback `jwt`, quando vem `account.provider === "google"`,
  um `findOrCreateGoogleUser(sub, email)` numa transação faz upsert do usuário e do
  vínculo `oauth_accounts`, e devolve o id. Guarda em `token.uid`.
- **Revalidação por request**: o `jwt` relê o usuário pelo `uid` a cada chamada. Se
  sumiu (removido/banco recriado) retorna `null` e derruba a sessão; de quebra mantém
  campos voláteis (ex.: flag `vip`) sempre frescos, sem esperar o login expirar.
- `session` só copia `token.uid -> session.user.id`.

O provider OAuth entra condicional: só monta o `Google(...)` se
`AUTH_GOOGLE_ID/SECRET` existirem no ambiente — o app sobe e o email/senha funciona
mesmo sem OAuth configurado ([[Configuração vem do ambiente, não do código]]).

## Detalhe que morde

**Não gate por middleware.** O middleware do Next roda no edge, e `pg`/`bcrypt` não
são edge-safe — importar o auth completo lá quebra. Gate no **server component** com
`const s = await auth(); if (!s) redirect(...)`. Fica no runtime Node, onde o banco
vive, e evita o split `auth.config.ts` edge-safe que o adapter obrigaria.

**Cookie de `localhost` ignora a porta.** Dois apps next-auth em `localhost:4070` e
`localhost:5000` compartilham o mesmo `authjs.session-token` — o segundo tenta decifrar
o cookie do primeiro e estoura `no matching decryption secret` (ruído em toda request,
mesmo sem ninguém logar ali). Namespace os cookies por app
(`cookies.sessionToken.name = "<slug>.session-token"`, idem `csrfToken`/`callbackUrl`)
pra cada um só enxergar o seu.

## Conexões
- Irmã do mesmo ethos sem-ORM: [[Runner de migration em SQL puro dispensa o CLI do ORM]]
- Parente stateless: [[Sessão de painel interno é um cookie assinado, não uma tabela de sessões]]
- Contraponto (o caminho com adapter/ORM): [[Prisma 7 tira a URL do schema - vai pro config e pro adapter]]
- Princípio: [[Configuração vem do ambiente, não do código]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
