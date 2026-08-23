---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-24
---

# Fórmula verificada só vale na escala em que foi verificada

> Uma fórmula conferida contra a realidade vira patrimônio: dá para reusá-la em
> qualquer tela. O que se esquece junto é a **procedência dos números** que ela
> recebeu na conferência. Alimentada por outra fonte, a mesma fórmula continua
> produzindo um resultado — e a aritmética não tem como reclamar de unidade.

## O que acontece

No [[piwdex2]] a calculadora inverte a fórmula de stat do jogo para descobrir o
IV de cada atributo, e foi conferida contra a tela de pokémon: um Electrode nível
54 com HP 113 devolve IV 29,9. Correto.

Reusei a mesma inversão numa ficha alimentada pelo WebSocket. Contra um pokémon
real, ela devolveu **IV 340 num teto de 32**. A fórmula estava certa; a escala,
não — a vida que o frame manda é cerca de dez vezes o stat de HP da tela.

O que torna esse erro caro é a apresentação. Ele não estoura: sai um número, com
duas casas, alinhado na coluna certa, ao lado de outros números que estão certos.
**Cálculo com entrada de escala errada não falha, ele mente com formatação.**

## O que fazer

- **Guarde a procedência junto da fórmula.** "Verificada contra X" é parte da
  definição, não do changelog. A anotação é o que faz a próxima pessoa perguntar
  se a fonte nova é o mesmo X.
- **Cheque contra um invariante que a própria fonte declara.** Se o servidor
  manda o total, confira a soma calculada contra ele. Divergiu, a leitura não
  serve — e é a leitura que sai, não o total.
- **Prefira não responder a responder torto.** Mostrar "não consegui abrir por
  atributo" preserva a confiança do resto da tela; um IV inventado contamina
  todos os números ao lado dele.
- **Teste com dado real antes de exibir.** Um caso concreto derruba em segundos o
  que a revisão de código não vê: as duas escalas passam pelo compilador.

## Conexões
- Irmã: [[Campo que a normalização não copia vira número errado, não erro]] ·
  [[Contador que conta sucesso de promessa afirma que deu certo]] ·
  [[Número de regra alheia se lê da fonte, não se congela em constante]]
- Visto em: [[piwdex2]]
- Mapa: [[Base]] · [[Dados]]
