---
tags: [tipo/atomica, camada/padrao, dev/frontend, nextjs, armadilha]
criado: 2026-09-25
---

# Regex do matcher do Next em string precisa de barra dupla

> O `matcher` do middleware (o `proxy` do Next 16) é uma string, e a string
> engole a barra invertida antes de a regex nascer: `"\."` vira `.` e `"\w"`
> vira `w`. O trecho clássico que pula arquivo com extensão,
> `.*\.[\w]+$`, escrito com barra simples, vira `.*.[w]+$`, que só pula caminho
> terminado em "w".

## O sintoma

Nada quebra para quem está logado: o middleware só confere se o cookie de
sessão existe, e com cookie tudo passa. Quem não tem sessão (a própria tela de
login, página pública aberta por link) recebe 307 para o login em todo arquivo
de `public/` e nas rotas de metadado: o ícone da aba (`/icon.svg`), o
`apple-icon`, a imagem da marca. No NaveX o defeito existia desde o primeiro dia
e só apareceu quando os ícones dos módulos viraram PNG em `public/` e alguém
testou a URL sem cookie.

## A regra

Escreva com barra dupla, como a documentação do Next faz:

```ts
export const config = {
  matcher: ["/((?!api|_next/static|_next/image|login|.*\\.[\\w]+$).*)"],
};
```

`String.raw` resolveria, mas o `config` do middleware é lido estaticamente no
build, e o literal simples é o que se garante que ele entende.

## Como conferir

Com o app no ar, sem cookie: `curl -o /dev/null -w "%{http_code}"` num arquivo
de `public/` e no `/icon.svg` tem que dar 200, e numa tela protegida, 307.
Conferir só logado não testa o matcher.

## Conexões
- Irmã: [[Formulário público por token opaco fica fora do gate de sessão]]
  (o mesmo matcher decide o que abre sem conta)
- Princípio: [[Verificar no build de produção, não só em dev]]
- Visto em: [[NaveX]]
- Mapa: [[Frontend]]
