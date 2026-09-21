---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-09-21
---

# Cabeçalho de seção não repete o que a navegação já diz

> A saída automática para o topo de uma tela de módulo é um cartão: fundo de
> vidro, halo da cor do módulo, chip de ícone tingido, título grande e uma linha
> de apoio. Fica bonito na primeira tela. Na trigésima, é a primeira dobra do
> sistema inteiro ocupada por informação que já está na tela.

## Conte os sinais antes de desenhar mais um

Na tela onde isso apareceu, o nome do módulo estava escrito em três lugares
visíveis ao mesmo tempo — o topo da barra lateral, a trilha do topo e a cor do
acento — e o nome da seção em três também: o item ativo da barra lateral, a
trilha (em negrito) e o título do cabeçalho.

O quarto sinal não reforça. Ocupa.

O teste é barato e se faz olhando um print: para cada elemento do cabeçalho,
perguntar **onde mais isto aparece nesta mesma tela**. O que aparece duas vezes
mais não é ênfase, é peso.

## Caixa que não contém nada não é caixa

Um cartão é um recipiente: ele existe para dizer que o que está dentro anda
junto e é separado do que está fora. Um cartão com um título e uma frase dentro
é uma borda em volta de uma linha de texto — a moldura promete um agrupamento
que não existe.

Some a moldura e o texto continua sendo o começo da tela, porque **posição já é
hierarquia**. O primeiro elemento da coluna é o título; não precisa de um quadro
avisando disso.

## O que sobrevive ao corte

- **O título**, porque a página precisa de um H1 e porque a trilha é cromo miúdo
  de 13px — serve para voltar, não para dar nome à tela.
- **A linha de apoio**, com largura máxima, porque é prosa: atravessando mil e
  quatrocentos pixels o olho não reencontra o começo da linha seguinte.
- **As ações**, na mesma altura do título em vez de numa fila própria.

O tamanho do título cai junto com a caixa. Ele estava em 1,75rem para preencher
um bloco que agora não existe; solto, 1,5rem já é claramente o título da página.

## O medo que não se confirmou

O argumento original para o chip tingido era real: numa plataforma de seis
módulos cujas telas têm a mesma forma — barra de filtro, painel, tabela —, sem
ele "a tela é cinza sobre cinza". Vale levar a sério antes de cortar.

Só que a cor do módulo não morava no chip. Mora na barra lateral (nome, ícone e
seção ativa), no fundo tingido da página e em cada botão de acento. Removido o
chip, o Contábil continua vermelho e o RH continua rosa à primeira vista. O que
se perdeu foi a repetição, não a identidade.

A lição geral: quando o argumento a favor de um elemento é "senão perde-se o
sinal X", confira **em quantos outros lugares X já está** antes de aceitar.

## Conexões
- Princípio: [[Nota carrega só o que a pessoa não sabe]] — a mesma economia,
  aplicada ao cromo: não gaste altura repetindo o que a superfície já diz.
- Irmã: [[Marca não repete o nome que está escrito ao lado dela]] ·
  [[Faixa de topo de ferramenta é chegada, não rótulo]] ·
  [[Barra de topo contextual - o módulo injeta suas ferramentas via portal]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
