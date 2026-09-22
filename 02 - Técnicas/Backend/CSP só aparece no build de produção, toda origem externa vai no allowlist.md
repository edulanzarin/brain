---
tags: [tipo/atomica, camada/padrao, dev/backend, seguranca, armadilha]
criado: 2026-07-28
---

# CSP só aparece no build de produção, toda origem externa vai no allowlist

> A Content-Security-Policy do servidor não roda no dev server. Um recurso
> externo (fonte, script, tile de mapa) passa em dev e é **bloqueado só em
> produção** — onde ninguém olhou.

## O problema

Em dev, o Vite (ou webpack-dev-server) serve o app **sem CSP**. Em produção,
quem serve é o Express com `helmet`, que manda uma CSP `default-src 'self'`. Aí
a fonte da marca vinha só do Google Fonts — `<link>` de `fonts.googleapis.com` +
os `.woff2` de `fonts.gstatic.com`. Em dev, Montserrat carregava; em produção a
CSP bloqueava os dois e a página caía pro `system-ui`. O bug não existe em dev,
por construção: é uma diferença de ambiente, não do código.

## A solução

CSP é allowlist: `default-src 'self'` nega tudo, e **cada origem externa que o
app usa de verdade precisa estar declarada na diretiva certa** — não basta uma
genérica.

```js
"style-src": ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"], // a folha .css
"font-src":  ["'self'", "data:",          "https://fonts.gstatic.com"],     // os .woff2
```

Fonte externa custa duas diretivas (a folha e o arquivo). Mapa, script de
terceiro, imagem remota — cada um na sua (`img-src`, `connect-src`,
`worker-src`, `script-src`). O jeito de não esquecer é [[Verificar no build de produção, não só em dev]]: suba o servidor de prod, abra o console e olhe os
erros de CSP; o dev server nunca vai te avisar.

## O que mais vale lembrar

- Alternativa mais robusta que liberar terceiros: **self-hostar** a fonte
  (`@font-face` local). Some a dependência de rede, o problema de privacidade e a
  CSP volta a ser só `'self'`.
- `preconnect`/`dns-prefetch` são dicas de recurso, não são governados pelas
  diretivas de fetch — liberar no allowlist é o `<link>` real e os arquivos.

## upgrade-insecure-requests deixa a tela em branco fora do https

A diretiva manda o navegador **refazer em https** toda requisição http da
página. Sob o domínio real ela é inócua, porque os assets são relativos e de
mesma origem. Mas ela inviabiliza qualquer acesso por http, e o sintoma não
parece de CSP:

- o HTML responde **200**;
- os assets, pedidos por `curl`, respondem **200**;
- no navegador a tela fica **em branco**, sem erro na página.

O navegador promoveu `/assets/…` para `https://host:4082`, onde ninguém fala
TLS. O `curl` não reproduz porque **ignora CSP** — e é por isso que a checagem
por linha de comando dá tudo certo enquanto a tela não abre.

Morde em prévia na LAN, em acesso por IP e em ambiente atrás de proxy que
termina o TLS e repassa http. Quem garante https no domínio é o HSTS, que não
tem esse efeito colateral; a diretiva pode sair.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
- Depende de: [[Ambiente de dev sobe igual ao de produção]]
- Visto em: [[Evento Navecon]] · [[Simulador Navecon]]
- Mapa: [[Backend]]
