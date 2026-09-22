---
tags: [tipo/atomica, camada/padrao, dev/frontend, design, armadilha]
criado: 2026-09-22
---

# Media query mede a janela; quem decide a quebra é a largura do contêiner

> `@media (min-width: 560px)` pergunta "a **tela** é larga?". A pergunta certa,
> quase sempre, é "o **bloco** é largo?". Quando o bloco tem largura fixa, as
> duas respostas divergem, e o layout quebra exatamente quando a tela cresce.

## O caso

No simulador, a pergunta de margem tinha cinco opções curtas e ganhou duas
colunas:

```css
@media (min-width: 560px) {
  .opcoes--duas { grid-template-columns: 1fr 1fr; }
}
```

Só que o contêiner era `.coluna`, travada em **440px**. Numa tela de 866px a
media query acendia e partia os 440px em dois cartões de ~215px. O texto passou
a quebrar dentro de cada um ("Margem apertada" em duas linhas, "Não sei dizer"
em duas), os cartões ficaram com alturas diferentes, e a grade apareceu
desalinhada. O defeito **não existia no celular** — só a partir de 560px, que é
onde ninguém procura por aperto.

O sintoma engana: parece problema de alinhamento de grid, e é problema de quem
foi consultado sobre a largura.

## A regra

- Layout **de página** (a página tem uma coluna ou duas?) pode usar `@media`: aí
  a janela é mesmo o assunto.
- Layout **de componente** (este bloco cabe em duas colunas?) usa
  `@container` — ou não usa condicional nenhuma.

```css
.opcoes { container-type: inline-size; }

@container (min-width: 520px) {
  .opcoes { grid-template-columns: 1fr 1fr; }
}
```

E vale perguntar antes se a condicional precisa existir. Aqui a resposta foi
não: uma coluna sempre, igual às outras oito perguntas. Duas colunas resolviam
um aperto de altura que, com cartão de 52px, já não havia. **Condicional de
layout que some é condicional que não pode quebrar.**

## O teste que pega

Não é olhar no celular e no desktop. É **varrer a largura** e procurar a faixa
em que o texto começa a quebrar dentro de uma célula. O defeito mora numa faixa,
não num extremo — mesma forma de
[[Calibre nas pontas, o meio esconde o defeito]], com o meio sendo a largura
intermediária que ninguém abre.

## Conexões
- Irmã: [[Centralizar na altura é margin auto, porque justify-content corta o topo]] ·
  [[Trocar a fonte muda a largura, não só o desenho da letra]]
- Princípio: [[Calibre nas pontas, o meio esconde o defeito]]
- Visto em: [[Simulador Navecon]]
- Mapa: [[Design]] · [[Frontend]]
