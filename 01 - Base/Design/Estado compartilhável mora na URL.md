---
tags: [tipo/atomica, camada/principio, design, dev/frontend]
criado: 2026-07-20
---

# Estado compartilhável mora na URL

> Se recarregar a página perde a informação, ou se mandar o link pro colega não leva
> ele pra mesma tela, o estado está no lugar errado.

## A regra

Vai pra URL o que descreve **o que está sendo visto**: filtro, busca, aba ativa,
página, ordenação, intervalo de datas. Fica fora da URL o que é **como a interface
está**: dropdown aberto, campo em foco, item em hover, rascunho não salvo.

O teste é uma pergunta só: *isso faz sentido no link que eu mando pra outra pessoa?*
Filtro faz. Dropdown aberto não.

## Por que

A URL é o único estado que sobrevive a recarregar, a botão de voltar, a favorito e a
copiar-colar no chat. Estado equivalente guardado em `useState` perde tudo isso de
graça — e o usuário descobre da pior forma, recarregando por acidente e voltando à
estaca zero.

Também resolve o botão voltar sem código nenhum: cada mudança de filtro vira uma
entrada de histórico, e voltar faz o que o usuário espera.

## Onde não colocar

Cache de servidor (React Query e afins) não é lugar de estado de interface — ver
[[Cache do React Query não é lugar de estado de interface]]. E estado de tela que é de
uma seção não deve subir pra página inteira:
[[Estado de tela pertence à seção, não à página]].

## Conexões
- Padrão que aplica: [[Filtro de lista mora na URL]]
- Irmã: [[Todo estado da tela tem visual]]
- Mapa: [[Base]] · [[Design]]
