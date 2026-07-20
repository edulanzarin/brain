---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-07-20
---

# Escala fechada em vez de valor solto

> Espaçamento, tamanho de fonte, raio e sombra saem de um conjunto pequeno e fixo.
> Se o valor que eu quero não está na escala, o certo quase nunca é criar mais um.

## A regra

Uma escala por eixo, poucos degraus, passo previsível:

- **Espaço**: 4 8 12 16 24 32 48 64. Nada de 13, 18, 27.
- **Texto**: 12 14 16 20 24 32 — e cada degrau com peso e altura de linha já definidos.
- **Raio**: 6 (controle), 10 (card), 999 (pílula). Três, não sete.
- **Sombra**: repouso, elevado, flutuante. Nomeadas por altura, não por blur.

## Por que

Layout inconsistente quase nunca vem de erro grosseiro — vem de dezenas de escolhas
pontuais de 14px, 15px, 18px que ninguém consegue enxergar isoladas, mas que somadas
fazem a tela parecer amadora. A escala tira a decisão do calor do momento e transforma
"quanto de espaço aqui?" numa pergunta de *qual degrau*, que tem 8 respostas possíveis
em vez de infinitas.

O efeito colateral bom: ritmo visual. Espaços múltiplos do mesmo passo criam alinhamento
mesmo entre componentes que nunca se falaram.

## Quando a escala não serve

Se um caso legítimo não cabe, a resposta é revisar a escala inteira (e todos os usos),
não abrir exceção local. Exceção local é como a escala morre — uma de cada vez.

## Conexões
- Base de: [[Token semântico em vez de valor literal]]
- Irmã: [[Container tem largura máxima e respiro constante]]
- Mapa: [[Base]] · [[Design]]
