---
tags: [tipo/atomica, design, dev/frontend, conceito, projeto/cofre-digital]
criado: 2026-07-20
---

# Esqueleto de carregamento imita a forma do conteúdo

> O esqueleto existe para a página **não se mexer** quando os dados chegam. Um
> retângulo cinza de altura arbitrária não faz isso — só preenche o tempo.

## O quê

Um primitivo `.vlt-skeleton` (brilho passando por cima, não opacidade piscando)
e, em cima dele, componentes com a forma real de cada conteúdo:
linha de lista, tabela com faixa de cabeçalho, grade de cards, cabeçalho de
página. Larguras alternadas entre linhas, senão a tabela vira um grid de barras
idênticas — que também não parece conteúdo.

O brilho é desligado em `prefers-reduced-motion`.

## Por que importa

No [[Cofre Digital]] o carregamento tinha dois problemas, e o segundo era o pior:

- 19 retângulos `h-64 animate-pulse` com as classes repetidas à mão em cada
  página — mesmo efeito escrito de 19 jeitos.
- Duas listas do dashboard **não mostravam nada**: um `<div>` vazio com padding.
  A página nascia curta e crescia quando os dados chegavam.

Trocar por esqueleto com a forma certa resolve o salto de layout e ainda dá uma
dica honesta do que vem: dá para ver que ali vai ter uma tabela de 7 colunas.

## Brilho vs pulso

`animate-pulse` altera a opacidade do bloco inteiro — o olho lê como "isso aqui
está desabilitado". O brilho atravessando lê como "isso está carregando". Custa
um `::after` com `transform: translateX`.

## Conexões
- Faz parte de: [[Design]]
- Aplicado em: [[Cofre Digital]]
- Ver também: [[Padrões de componentes de dashboard]] · [[Classes de componente vão em @layer components no Tailwind]]
