---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-08-30
---

# O que responde pergunta rara não ocupa a rolagem de todo mundo

> A altura da página é orçamento, e quem paga é toda visita. Bloco que só
> interessa a uma minoria, aberto por padrão, cobra rolagem de quem não veio por
> ele — e empurra pra baixo o que a maioria veio ver.

## A regra

Antes de deixar um bloco aberto na página, pergunte **que fração de quem chega
tem essa dúvida**. Três faixas, três destinos:

- **Quase todo mundo** — fica aberto, e perto do topo.
- **Uma minoria, mas quando quer, quer inteiro** — vira modal, gaveta ou
  `<details>`. O gatilho fica visível, o conteúdo não.
- **Ninguém pergunta** — não é dobra, é corte. Esconder não conserta conteúdo
  que não devia existir.

O gatilho precisa carregar o estado que decide se vale abrir. "Horário de
expediente" ao lado de um selo "em expediente" já responde a pergunta comum sem
ninguém clicar; quem clica é quem tem a pergunta rara ("atende terça?").

## Por que

O custo é assimétrico e invisível. Quem abriu, achou e fechou não reclama; quem
rolou uma tela de tabela que não ia ler também não reclama — só sai um pouco
antes. O bloco parece inofensivo porque o prejuízo dele não chega como
reclamação, chega como atenção que acabou antes do fim da página.

E o padrão se acumula: cada bloco isolado parece pequeno, e nenhum deles é o
culpado pela página ter virado três telas.

## Dois casos

- **O expediente do perfil no [[Privello]].** Sete linhas de horário no meio da
  ficha, entre a descrição e as avaliações. Quem chega no meio da tarde já viu
  "em expediente" ao lado do nome e não precisa da tabela; virou modal atrás de
  um botão, e o quadro dentro dele mostra os sete dias — inclusive os fechados,
  porque a pergunta que sobra é "atende terça?".
- **O manual da ferramenta no [[piwdex2]]** —
  [[Manual de ferramenta é resumo visível com passo a passo sob demanda]]. Seis
  passos abertos empurravam a própria ferramenta pra fora da primeira dobra, que
  é o único lugar onde ela funciona.

Os dois têm a mesma forma: uma linha visível que diz o tamanho e o estado do que
está atrás, e o conteúdo sob demanda.

## O outro lado

Esconder não é de graça. Estado que só existe aberto precisa aparecer aberto em
algum lugar — no catálogo de componentes, no mínimo, senão a próxima pessoa
conclui que a peça não existe e reescreve —
[[Catálogo de componentes é contrato vivo, não documentação]]. E o que o
buscador precisa ler não pode nascer dentro de um modal que só o clique monta.

## Conexões
- Irmã: [[Tela que abre vazia tem que ensinar, tela que abre cheia não]] ·
  [[Nota carrega só o que a pessoa não sabe]]
- Técnica que aplica: [[Manual de ferramenta é resumo visível com passo a passo sob demanda]] ·
  [[Modal com conteúdo que cresce tem teto de altura e área que rola]]
- Visto em: [[Privello]] · [[piwdex2]]
- Mapa: [[Base]] · [[Design]]
