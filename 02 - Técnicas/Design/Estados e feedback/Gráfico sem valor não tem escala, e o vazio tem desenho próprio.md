---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-09-24
---

# Gráfico sem valor não tem escala, e o vazio tem desenho próprio

> Com todos os valores em zero, o gráfico calcula uma escala de 0 a quase nada, e
> as três linhas de grade saem com o mesmo rótulo: "R$ 0,00", "R$ 0,00",
> "R$ 0,00". A tela afirma uma escala que não existe, e o que devia dizer "nada
> vendido ainda" parece defeito.

## O problema

A regra que arredonda o teto da escala para cima (1, 2, 2,5, 5, 10) funciona para
qualquer máximo positivo. Com máximo zero, ela devolve 1 (centavo), e o
formatador de eixo arredonda 0,5 e 1 centavo para "R$ 0,00". A conta está certa;
o desenho mente.

É exatamente o gráfico que uma conta nova vê primeiro.

## A solução

Tratar o vazio como estado, antes de montar a escala: se todos os pontos são
zero, desenhar só a linha de base, os rótulos das pontas do período e uma frase
que diz o que é ("Nenhuma venda nos últimos 30 dias"). A frase vem por prop,
porque só quem usa o gráfico sabe o nome do que falta.

## O que mais vale lembrar

- **A margem do eixo se mede pelo maior rótulo.** Com margem fixa, "R$ 250,00"
  perdeu o "R" cortado na borda do SVG; e com rótulo curto sobra espaço morto.
  Calcular a largura pelo número de caracteres do maior rótulo formatado basta.
- Tela de conta nova sem nenhuma história não precisa nem do gráfico vazio:
  enquanto a configuração não terminou, o início mostra só o que ensina o
  próximo passo. Ver [[Tela que abre vazia tem que ensinar, tela que abre cheia não]].

## Conexões
- Princípio: [[Todo estado da tela tem visual]] · [[A tela não afirma mais precisão do que a fonte tem]]
- Irmã: [[Zero num medidor é estado, não barra vazia]] ·
  [[A unidade se diz uma vez, não em cada rótulo do eixo]]
- Visto em: [[telebot]]
- Mapa: [[Design]]
