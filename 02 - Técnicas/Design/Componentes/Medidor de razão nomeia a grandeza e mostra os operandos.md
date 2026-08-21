---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-21
---

# Medidor de razão nomeia a grandeza e mostra os operandos

> `DIFERENÇA [▓▓░░░░] 0.000 / 0.150` é um placar sem jogo. Diferença de quê? Entre o quê?
> Quem escreveu sabia as duas respostas e por isso não viu que a barra não as diz.

## A armadilha

Um medidor de razão — `X de Y` com barra — parece autoexplicativo porque **quem o
desenhou tem o contexto inteiro na cabeça**. Na tela sobram três buracos:

1. **A grandeza não é nomeada.** "Diferença", "Uso", "Progresso" são categorias, não
   grandezas. Diferença *de Quality*, uso *de armazenamento*.
2. **Os operandos não aparecem.** Quando o número é derivado de dois valores que estão
   ali perto, mostrar só o resultado obriga a pessoa a adivinhar de onde ele saiu.
3. **A barra não diz o que é bom.** Cheia é sucesso ou é alerta? Só o rótulo resolve.

O teste que reprova: **quem conhece o domínio pergunta o que aquilo é**. No piwdex2 quem
perguntou foi o dono do projeto, que sabe a regra do jogo de cor.

## A forma que se lê

```
par válido · Quality 2.100 e 2.020            ← estado + os operandos
diferença de Quality [▓▓░░░░] 0.080           ← a grandeza, nomeada
   de 0.150 que o jogo permite                ← o denominador em palavra, não em "/"
Ainda cabem 0.070 antes do par reprovar.      ← a pergunta seguinte, respondida
```

Quatro trocas concretas:

- **Grandeza no rótulo**, não categoria.
- **Operandos visíveis** quando o número é derivado de valores da mesma tela.
- **Denominador em palavra.** `/ 0.150` obriga a inferir que aquilo é um limite; "de
  0.150 que o jogo permite" diz de quem é a regra.
- **A folga por extenso.** Depois de "quanto já gastei" vem sempre "quanto ainda cabe" —
  responder as duas custa uma frase.

Isso NÃO contradiz [[Nota carrega só o que a pessoa não sabe]]: aquela regra corta o eco
do campo; esta cobre o buraco oposto, o rótulo que economizou a informação que só existia
na cabeça de quem construiu.

## E o zero continua valendo

Diferença `0.000` esvazia a barra, que é a mesma imagem de "não carregou" — só que aqui
zero é a **melhor** notícia possível, não a pior. O trilho ganha um preenchimento fraco de
folga por baixo e a frase diz "Quality idêntica", pra o medidor continuar dizendo alguma
coisa quando não há o que medir. Mesma raiz de
[[Zero num medidor é estado, não barra vazia]], com o sinal invertido: o estado extremo
precisa de palavra, seja ele bom ou ruim.

## Conexões
- Princípio: [[Todo estado da tela tem visual]] · [[Nota carrega só o que a pessoa não sabe]]
- Irmã: [[Zero num medidor é estado, não barra vazia]] ·
  [[Blocos de dado - card, KPI e gráfico]] ·
  [[Chip que serve a duas grandezas declara qual delas mostra]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
