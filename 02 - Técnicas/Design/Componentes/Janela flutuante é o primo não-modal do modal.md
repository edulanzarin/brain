---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-08-12
---

# Janela flutuante é o primo não-modal do modal

> Querer "abrir várias telas e comparar lado a lado" não é estilizar o modal
> diferente: é um segundo primitivo, com a mesma casca e a intenção oposta.

## O problema

O modal bloqueia o fundo **de propósito** — backdrop, trava de scroll,
`aria-modal`, um de cada vez. É a UX certa pra confirmar, formulário, fluxo
linear. Mas comparar duas coisas, ou deixar um detalhe aberto enquanto se mexe
na tabela atrás, é o contrário disso. Reaproveitar o modal "só que arrastável"
quebraria justamente o que o faz modal.

## A solução

Dois primitivos com a **mesma casca** (o card opaco, cabeçalho com título e
fechar, corpo com teto de altura e área que rola) e **intenção oposta**:

- **Modal** — bloqueia, um por vez. Confirmar, formulário, decisão.
- **Janela** — coexistem, o fundo segue vivo, arrasta pela barra e vem pra
  frente ao foco. Consultar, comparar lado a lado, multitarefa.

A janela é gerida por um store externo (lista de janelas com `id`, posição e
`z`); a casca é desenhada num portal no `body`, montada uma vez alto, então
qualquer tela abre janela sem montar nada local. Focar = subir o `z`. O `id`
deduplica: reabrir a mesma coisa traz a janela existente pra frente em vez de
empilhar cópias.

## O que mais vale lembrar

- **Superfície opaca**: overlay que flutua sobre conteúdo tem que ser legível —
  ver [[Vidro flutuante precisa de superfície mais opaca que a chrome]]. Onde o
  card já é sólido, ele mesmo serve; só falta a elevação (sombra).
- **Arraste comita ao soltar, não a cada quadro**: durante o gesto aplica um
  `transform` local (barato, compositado) e só no `pointerup` grava a posição
  nova no store — um update em vez de um por frame.
- **O `z` mora entre o chrome e o modal**: acima do cabeçalho fixo da página,
  abaixo do modal verdadeiro. Assim um modal, quando abre, cobre as janelas — a
  hierarquia de bloqueio continua valendo.
- **No compacto não arrasta**: o gesto não cabe no telefone; a janela vira um
  painel que ocupa quase a tela.

## Conexões
- Princípio: [[A variante de um controle muda a intenção, não o tamanho]]
- Irmã: [[Modal com conteúdo que cresce tem teto de altura e área que rola]] · [[Vidro flutuante precisa de superfície mais opaca que a chrome]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
