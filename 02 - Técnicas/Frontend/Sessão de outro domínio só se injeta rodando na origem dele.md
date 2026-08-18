---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-18
---

# Sessão de outro domínio só se injeta rodando na origem dele

> Um site A não consegue "logar" você num site B mexendo no storage de B: o navegador
> isola por origem. Pra injetar uma sessão em B, o código tem que rodar NA origem de B —
> console ou bookmarklet — não na página de A.

## O problema

Queria um botão no painel (origem `piwdex.com.br`) que abrisse a aba do jogo
(`poke.idleworld.online`) já logada numa conta, escrevendo o token no
`sessionStorage`/`localStorage` do jogo. Não dá. `sessionStorage`, `localStorage`,
cookies e IndexedDB são particionados por **origem** (esquema + host + porta). JS numa
página só lê/escreve o storage da própria origem. `window.open("outra-origem")` devolve
um handle, mas acessar `win.sessionStorage` dele estoura `SecurityError`. Não é bug, é a
fronteira de segurança do navegador — a mesma que impede um site qualquer de ler seus
cookies de outro.

## A solução

O código de injeção precisa **executar na origem de destino**. Dois jeitos, os dois
válidos, que é bom oferecer juntos:

- **Snippet no console**: o botão copia pro clipboard um comando pronto e abre a aba do
  destino. Você cola em F12 → Console → Enter. Sempre funciona, nada a instalar.
- **Bookmarklet arrastável**: um `javascript:` com o token embutido; arrasta pra barra de
  favoritos e, **na aba do destino**, clica. Roda na origem certa. Token que expira pede
  regerar o bookmarklet.

```js
// o comando (roda NA aba do jogo): põe o token na sessão e entra
sessionStorage.setItem('pokeweb:tokens', JSON.stringify({ accessToken, refreshToken }));
location.href = '/play';
```

Detalhe de React: `href="javascript:..."` é sanitizado (não renderiza). Pro bookmarklet,
setar o atributo por `ref` no `useEffect` (`el.setAttribute('href', ...)`), fora do JSX.

## O que mais vale lembrar

- O sentido natural é o inverso e é o que o app já faz: o token **sai** da origem do jogo
  (o usuário copia o `pokeweb:tokens`) e é colado no seu app. Injetar de volta é o mesmo
  caminho ao contrário — por isso só funciona rodando lá.
- Um-clique-de-verdade cross-origin só com **extensão de navegador** (tem permissão de
  origem) — peso que raramente compensa.
- REST com `Bearer` não tem essa fronteira: o servidor do seu app fala com a API do outro
  domínio à vontade (foi assim que o painel puxa gold/diamantes ao vivo). A trava é só do
  **storage do navegador**, não do HTTP server-side.

## Conexões
- Princípio: folha isolada por ora — candidato "isolamento de origem é fronteira dura do
  navegador"; promover a [[Base]] na 2a aparição (não inventar link falso antes disso).
- Irmã: [[Permissão se valida no servidor, não na interface]]
- Visto em: [[piwdex]]
- Mapa: [[Frontend]]
