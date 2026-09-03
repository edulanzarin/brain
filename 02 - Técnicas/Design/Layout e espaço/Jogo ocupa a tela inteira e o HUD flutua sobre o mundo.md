---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-09-03
---

# Jogo ocupa a tela inteira e o HUD flutua sobre o mundo

> Painel ao lado do mundo não faz um jogo com painéis. Faz um painel de controle com
> um jogo dentro, e a leitura é imediata.

## O erro

Montei a tela do jogo com o hábito de dashboard: grade de duas colunas, o mundo numa
delas, painéis na outra, mais um cartão de ficha embaixo. Tudo alinhado, tudo com
respiro, e reprovado numa frase — "design horroroso". Nenhuma peça estava errada; a
**moldura** estava.

O que denuncia: o mundo tinha borda e largura máxima. Cenário com moldura é ilustração
numa página; cenário sem moldura é lugar onde se está.

## O desenho certo

O mundo ocupa a viewport inteira, sem borda e sem container. O HUD é **camada por
cima**, encostada nos cantos:

| Canto | O que vai | Por quê ali |
|---|---|---|
| superior esquerdo | quem eu sou: retrato, rank, nível, barras | é o que se consulta de relance, e o olho já mora ali |
| superior centro | ferramentas, só ícone | fila curta, alvo de 40 px, sem rótulo ocupando altura |
| superior direito | moeda e sair | o que é conta, não jogo |
| inferior esquerdo | onde eu estou e como andar | ancora o contexto sem competir com o centro |
| inferior direito | gaveta do que está aberto | o painel grande fica longe do personagem, que anda no meio |

O container da camada **não intercepta clique** (`pointer-events: none`); cada painel
reativa para si. Sem isso o HUD engole o mundo inteiro por baixo dele.

## O painel flutuante é quase opaco, não vidro

Aqui a regra do vidro se inverte: pixel art atrás de desfoque vira sujeira, e o texto
do HUD precisa ganhar do mapa em qualquer cenário — inclusive num deserto claro.
Superfície em 94% e sombra dura embaixo; sem `backdrop-filter`.

## A gaveta tem teto de altura

Overlay que cresce com o conteúdo empurra o mundo. Teto em `68vh`, cabeçalho fixo, e
a rolagem só na parte que cresce.

## Conexões
- Princípio: [[A casca se compartilha por público, não por marca]]
- Irmã: [[Modal com conteúdo que cresce tem teto de altura e área que rola]] ·
  [[Sobre arte de fundo, a chrome também tem piso de opacidade]]
- Visto em: [[naruto-idle]]
- Mapa: [[Design]]
