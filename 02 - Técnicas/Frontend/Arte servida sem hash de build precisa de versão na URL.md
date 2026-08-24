---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-24
---

# Arte servida sem hash de build precisa de versão na URL

> O que o empacotador processa sai com hash no nome, então republicar invalida o
> cache sozinho. O que vive em `public/` **não** passa por isso: mesmo nome, mesma
> URL, e quem já visitou continua servindo a cópia velha do disco. A troca
> simplesmente "não acontece" — sem erro, sem 404, sem nada pra depurar.

## O formato do defeito é o que o torna caro

Ele só existe **para quem já visitou**. No computador de quem publicou, o arquivo
é novo e está certo. Então o relato chega como "sumiu o ícone" / "não mudou nada",
e a primeira reação é conferir o arquivo — que está perfeito. Nada no servidor,
nas ferramentas de rede ou no build acusa coisa alguma.

Pior quando a versão publicada por engano estava **quebrada**: aí o cache
congela o defeito. Aconteceu com um lote de ícones SVG publicado com um erro de
sintaxe e corrigido em minutos; quem abriu a página na janela errada ficou com o
arquivo inválido preso por dias, vendo o ícone de reserva.

## A saída

A URL sai de **uma função só**, com a versão dentro:

```ts
const VERSAO_ARTE = 2
export const arteUrl = (nome: string) => `/images/icons/${nome}.svg?v=${VERSAO_ARTE}`
```

Republicou, sobe o número. O ponto de a função existir não é a query — é que ela
tira do chamador a chance de escrever o caminho à mão e esquecer a versão. Caminho
escrito à mão é o formato em que essa regra é esquecida.

Sobre o cabeçalho: `Cache-Control` longo em `public/` continua certo, e é o que
mata a revalidação por navegação. O erro não é cachear muito — é cachear muito
**sem ter como invalidar**.

## Quando não precisa

Arquivo cujo NOME é o conteúdo — `/game-sprites/<looktype>.webp`, saído de um bake
em que looktype novo é arquivo novo — pode levar `immutable` e dispensa versão.
A pergunta é se o mesmo nome pode um dia ter outro conteúdo. Se pode, precisa.

## Conexões
- Princípio: [[Token semântico em vez de valor literal]]
- Irmã: [[Trocar a arte de fundo é refazer a calibração, e a régua não é a média]] ·
  [[Peça desenhada fora do DOM é uma segunda implementação do tema, e ela envelhece calada]]
- Visto em: [[piwdex2]]
- Mapa: [[Frontend]]
