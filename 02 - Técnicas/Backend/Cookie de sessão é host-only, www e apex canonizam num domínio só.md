---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-17
---

# Cookie de sessão é host-only, www e apex canonizam num domínio só

> Cookie seguro de sessão (`__Host-`/`__Secure-` sem `domain`) vale só no host
> exato que o emitiu: login feito em `www.site.com.br` não existe em
> `site.com.br`. Publicar os dois hosts sem canonizar cria o bug "loguei e o site
> me deslogou" — e quebra callbacks de pagamento que apontam pra uma URL única.

## O problema

Apontar `www` e apex pro mesmo app parece cortesia. Mas a sessão é host-only por
design (o prefixo `__Host-` **proíbe** `domain=`), e integrações com URL única
(`APP_URL` para webhook e `back_urls` de gateway de pagamento) sempre devolvem o
usuário num host só — quem navegava no outro volta deslogado.

## A solução

Eleger UM host canônico (o apex) e fazer o outro **só redirecionar**, permanente:

```ts
// next.config.ts
async redirects() {
  return [{
    source: "/:path*",
    has: [{ type: "host", value: "www.site.com.br" }],
    destination: "https://site.com.br/:path*",
    permanent: true,
  }];
}
```

O DNS/plataforma pode registrar os dois hosts, mas todo tráfego do secundário cai
no canônico antes de qualquer cookie nascer. `APP_URL` aponta pro canônico.

## O que mais vale lembrar

- A alternativa (cookie com `domain=.site.com.br`) enfraquece o cookie (vale em
  todo subdomínio) e nem é permitida com `__Host-`. Canonizar é a saída certa,
  não a de contorno.

## Conexões
- Princípio: (folha isolada — nenhum princípio da Base cobre; candidata se aparecer segundo caso)
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
