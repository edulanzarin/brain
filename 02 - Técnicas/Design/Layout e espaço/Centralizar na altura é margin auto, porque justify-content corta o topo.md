---
tags: [tipo/atomica, camada/padrao, dev/frontend, design, armadilha]
criado: 2026-09-22
---

# Centralizar na altura é margin auto, porque justify-content corta o topo

> `justify-content: center` num contêiner de altura fixa centraliza enquanto o
> conteúdo cabe. Quando ele passa da altura, o excesso **transborda para os dois
> lados**, e o lado de cima não tem como ser rolado: some. `margin-block: auto`
> no filho centraliza igual e empurra o excesso só para baixo, que é onde a
> rolagem alcança.

## A regra

Etapa de tela cheia (um passo de formulário, uma pergunta de quiz, uma tela de
espera) quer duas coisas ao mesmo tempo:

- **centrada na altura** quando o conteúdo é curto;
- **rolável a partir do topo** quando é longo.

```css
.palco {
  min-height: 100dvh;      /* dvh, não vh: a barra do navegador móvel some e volta */
  display: flex;
  flex-direction: column;
}

.palco__miolo {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: safe center;   /* `safe` já evita o corte nos navegadores que o têm */
}
```

A palavra-chave `safe` é a correção padronizada: ela desliga o alinhamento
quando ele causaria perda de conteúdo. Onde ela não chegou, o equivalente é
`margin-block: auto` no filho, que nunca produz margem negativa e por isso nunca
corta.

## Por que o corte acontece

Alinhamento central distribui a sobra **igualmente** nas duas pontas. Com sobra
negativa (conteúdo maior que o contêiner), "metade da sobra" vira deslocamento
negativo no início: o começo do conteúdo vai para antes da borda de rolagem. O
navegador só rola do início do contêiner para a frente, então o que ficou antes
é inalcançável. É um corte silencioso: nada avisa, e no desktop, onde sobra
altura, ele nem aparece.

## Duas consequências que vêm junto

- **Teste sempre com o conteúdo mais alto que existe**, não com o representativo.
  No simulador era a pergunta de nove opções; as outras oito perguntas cabiam
  folgadas e teriam passado na revisão.
- **`100vh` mente no celular.** A barra do navegador entra e sai, e `vh` fica
  congelado no tamanho maior: o rodapé some atrás da barra. `100dvh` acompanha,
  com `100vh` na linha anterior como reserva para quem não conhece `dvh`.

## Conexões
- Irmã: [[Margem negativa em item de flex centralizado vale metade]] ·
  [[Estado de tela pertence à seção, não à página]]
- Visto em: [[Simulador Navecon]]
- Mapa: [[Design]] · [[Frontend]]
