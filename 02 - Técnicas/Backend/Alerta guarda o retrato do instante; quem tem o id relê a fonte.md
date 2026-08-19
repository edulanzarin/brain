---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-19
---

# Alerta guarda o retrato do instante; quem tem o id relê a fonte

> Notificação de "achei o que você queria" é um **retrato** do objeto no instante em que
> a regra casou. Ele envelhece — e, pior, envelhece com o formato de quando foi tirado:
> campo que passou a ser gravado hoje não existe em nenhum retrato de ontem.

## O problema

O worker casa a busca do usuário contra a fonte e grava na notificação tudo o que a tela
vai precisar. Funciona no dia em que se escreve. Depois:

- **o campo novo não retroage.** Passou a gravar `stats` hoje? Todo alerta anterior segue
  sem, e a tela cai no fallback (no meu caso, os stats BASE da espécie — todos os
  indivíduos exibidos iguais, o que parece bug de cálculo e não de dado);
- **o preço, o estoque e a própria existência mudam.** O anúncio pode ter sido vendido, ou
  estar mais barato agora;
- **backfill não resolve**: o dado que faltava não estava em lugar nenhum seu — estava na
  fonte, naquele momento.

## A solução

Grave no retrato o **identificador da fonte** e trate o resto como cache:

1. a tela relê os itens **visíveis** pela fonte, em lote, pelo id (`?ids=a,b,c`);
2. o que a fonte devolve **substitui** o retrato — é o estado de agora;
3. o que ela não devolve **cai pro retrato** — sumiu da fonte, mas o usuário ainda merece
   ver o que foi que casou.

Releitura por página (o punhado que está na tela), não pela lista inteira, e por cima de
um cache curto do lado do servidor — a fonte costuma ser uma resposta grande e única para
todo mundo. Assim a leitura extra custa quase nada e o retrato deixa de ser a verdade.

O id é o que torna isso possível: **retrato sem id é beco sem saída**. Se a fonte não dá
identificador estável, guarde o suficiente para reencontrar o item (chave natural), e
aceite que a releitura será uma busca, não um lookup.

## Visto em

No piwdex, os "desejos" mostravam um pokémon do mercado com os stats reais e todos os
outros com os stats base: só o alerta criado depois da mudança de hoje tinha o campo. A
página do desejo passou a reler os anúncios vivos por `listingId` no mesmo cache de 60s
do consultor de mercado; o alerta virou fallback para o que já foi vendido.

## Conexões
- Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]]
- Irmã: [[Contador de terceiro conta no escopo dele, o seu recorte é delta sobre uma base]]
- Parente: [[Total ao vivo é o persistido fechado mais o em andamento ainda não gravado]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
