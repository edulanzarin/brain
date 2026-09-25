---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-09-25
---

# Filtro de tela não reusa o nome de parâmetro que o contexto já usa

> Quando o app tem um contexto de trabalho que vai na URL (empresa, filial,
> período) e uma tela ganha filtro próprio, o filtro da tela precisa de nome de
> parâmetro próprio. Mesmo nome com outro significado faz o parser do contexto
> ler o valor da tela e recusar ou, pior, aceitar errado.

## O caso

No DP, a Rotatividade filtrava por estabelecimento pelo nome ("Filial 2") e
mandava `estabs=Filial 2`. O contexto do topo já usava `estabs` para a filial,
pelo código numérico, e o parser comum do servidor lê `estabs` como número.
Escolher qualquer estabelecimento dava 400 "Filial inválida". O defeito veio do
sistema em produção e passou meses sem aparecer, porque ninguém usava o filtro
junto do resto.

O parâmetro virou `estabelecimentos`. A tela não oferece a filial do topo, então
os dois não se cruzam mais.

## A regra

- O contexto é dono dos nomes dele. Liste-os num lugar só (o serializador do
  contexto) e confira contra eles ao criar o parâmetro de uma tela.
- Nome de parâmetro diz o tipo do valor: `estabs` carrega código;
  `estabelecimentos` carrega o texto que a tela mostra.
- Se o servidor tem um parser comum que lê a querystring inteira, o parâmetro da
  tela passa por ele também. Um teste que monta a URL da tela e a entrega ao
  parser comum pega a colisão antes da produção.

## Conexões
- Princípio: [[Dado escrito por dois caminhos precisa de uma regra só, fora dos dois]]
- Irmã: [[Dois setters de URL no mesmo gesto, e o segundo desfaz o primeiro]]
- Visto em: [[NaveX]] · [[Navetech Hub]]
- Mapa: [[Frontend]]
