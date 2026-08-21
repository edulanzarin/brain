---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-08-21
---

# Tela que abre vazia tem que ensinar, tela que abre cheia não

> Catálogo se explica de olhar. Ferramenta abre esperando entrada — e ninguém aprende
> a preencher um formulário olhando pra ele.

## A regra

Antes de escrever qualquer instrução na tela, pergunte **como ela abre**:

- **Abre cheia** (lista, catálogo, ficha, relatório): o conteúdo já é a explicação. Um
  passo a passo aqui é o sistema ensinando a rolar uma lista — ruído, e ruído que
  empurra o conteúdo pra baixo.
- **Abre vazia** (calculadora, simulador, importador, busca avançada): ela pede números
  que moram **fora dela**. Sem manual, a tela mais útil do sistema é a que mais parece
  inacabada — porque o visitante não vê o que ela faz, vê seis campos em branco.

O que o manual de uma tela vazia carrega é justamente o que ela não pode mostrar: **de
onde vem cada entrada**, **o que o resultado significa** e **quando ele não fecha**.
Rótulo de campo, placeholder e ordem dos campos já dão o resto — repetir isso em prosa
é a mesma armadilha de [[Nota carrega só o que a pessoa não sabe]].

## Por que

Estado vazio bem desenhado resolve o caso de quem já está lá (ver
[[Todo estado da tela tem visual]]), mas ele fala tarde: aparece dentro do fluxo, depois
que a pessoa escolheu um campo. O manual responde antes — na altura em que a pessoa
ainda está decidindo se aquela tela serve pra ela.

E o custo é assimétrico. Manual em catálogo custa espaço e não compra nada; manual
ausente em ferramenta custa a ferramenta inteira.

## Regra de bolso

Se o primeiro print da tela é útil sozinho, ela não precisa de manual. Se o primeiro
print é um formulário em branco, ela precisa — e o manual tem que responder "de onde eu
tiro esses números".

## Conexões
- Aplica: [[Manual de ferramenta é resumo visível com passo a passo sob demanda]]
- Irmã: [[Todo estado da tela tem visual]] · [[Nota carrega só o que a pessoa não sabe]]
- Visto em: [[piwdex2]]
