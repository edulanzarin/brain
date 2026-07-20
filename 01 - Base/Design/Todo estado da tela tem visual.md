---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-07-20
---

# Todo estado da tela tem visual

> Carregando, vazio, erro e sucesso são parte do design. Não são sobra pro fim.

## A regra

Toda tela que busca ou envia dado tem quatro estados desenhados, não um:

- **Carregando** — esqueleto com a forma do conteúdo, não spinner genérico nem bloco
  cinza. Ver [[Esqueleto de carregamento imita a forma do conteúdo]].
- **Vazio** — diz por que está vazio e qual é a próxima ação. "Nenhum resultado para
  *contrato*" com botão de limpar filtro, não uma área em branco.
- **Erro** — diz o que falhou e o que dá pra fazer. Nunca `alert()` do navegador:
  ver [[Toast em vez de alert para o feedback do app]].
- **Sucesso** — ação que deu certo confirma. Silêncio depois de salvar faz o usuário
  clicar de novo.

## Por que

É a parte que sempre sobra pro fim e é justamente a que denuncia que o design parou no
layout. O caminho feliz com dado pronto é fácil e é o que a gente desenha primeiro; o
usuário passa boa parte do tempo nos outros três.

Sintoma clássico, que já apareceu aqui: um app inteiro cujo único feedback eram 24
`alert()` nativos — janela do sistema operacional, fora da linguagem visual, e só para
erro. Sucesso não dizia nada.

## Regra de bolso

Se a tela só ficou bonita com dado pronto, ela não está pronta. Desenhe os quatro
estados antes de considerar a tela entregue.

## Conexões
- Padrões que aplicam: [[Toast em vez de alert para o feedback do app]] · [[Esqueleto de carregamento imita a forma do conteúdo]]
- Irmã: [[Estado compartilhável mora na URL]]
- Mapa: [[Base]] · [[Design]]
