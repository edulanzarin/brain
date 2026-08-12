---
tags: [tipo/atomica, camada/padrao, dev/backend, seguranca]
criado: 2026-08-12
---

# Sessão de painel interno é um cookie assinado, não uma tabela de sessões

> Um painel usado por uma ou duas pessoas não precisa de `express-session`, store
> em Redis nem tabela de sessões. O cookie **é** a sessão: um token `exp.hmac(exp)`
> que o servidor valida pela assinatura, sem guardar estado.

## O problema

O reflexo para "colocar login" é puxar uma lib de sessão com store. Isso traz
dependência nova, estado no servidor (memória que morre no restart, ou mais um
serviço para operar) e configuração — tudo para autenticar um punhado de acessos
internos.

## A solução

Emitir um cookie que se autovalida. O corpo é a expiração; a prova é o HMAC dela
com um segredo do ambiente:

```ts
issue()  => `${exp}.${hmac(exp)}`            // exp = agora + TTL
verify(t)=> safeEqual(sig, hmac(exp)) && Number(exp) > Date.now()
```

Login compara usuário/senha da env em tempo constante e, se baterem, grava o
cookie. Sem estado no servidor: qualquer instância valida o mesmo cookie, e o
restart não desloga ninguém. É uma aplicação de
[[A assinatura autentica o dado, não quem o trouxe]].

O cookie precisa das três defesas, ou não vale nada:

- `httpOnly` — JS da página não lê o token.
- `sameSite: 'lax'` — o navegador não manda o cookie num POST cross-site, o que
  **mata o CSRF** sem token anti-CSRF: a ação destrutiva só dispara a partir do
  próprio site.
- `secure` em produção — só trafega sob HTTPS.

E o login precisa de **rate limit** — a assinatura protege a sessão, não a senha
contra força-bruta.

## O que mais vale lembrar

É **stateless**: não dá para revogar uma sessão específica sem trocar o segredo
(o que derruba todas). Para um painel interno, isso é aceitável — logout apaga o
cookie local e o TTL fecha o resto. No dia em que precisar de vários usuários com
perfis, revogação seletiva ou auditoria de sessão, aí sim vale uma tabela. Antes
disso é peso sem retorno.

## Conexões
- Princípio: [[A assinatura autentica o dado, não quem o trouxe]]
- Irmã: [[Webhook de terceiro se valida pela assinatura antes de confiar no corpo]]
- Oposto por contexto: [[Sessão opaca no banco separa autenticação de permissão]]
- Visto em: [[Evento Navecon]]
- Mapa: [[Backend]]
