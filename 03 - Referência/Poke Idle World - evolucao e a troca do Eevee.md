---
tags: [tipo/referencia]
criado: 2026-08-25
---

# Poke Idle World - evolucao e a troca do Eevee

> Como se evolui no jogo, e por que o Eevee nao entra nessa regra. A parte do Eevee
> NAO existe em fonte publica — o `creatures.json` afirma o contrario do que
> acontece (ver [[Poke Idle World - endpoints publicos de dados]]). A tabela abaixo
> veio da tela da Loja do Marlon, no jogo.

Visto em: [[piwdex2]]

## Evolucao comum (Pokepedia do jogo, `/pokepedia/systems/evolution`)

Nao envolve ouro. Sao dois caminhos:

| Caminho | Nivel depois | Custo |
|---|---|---|
| Com Evolution Stones | mantem o nivel atual | as stones da especie |
| Sem stones (gratis) | volta pro Lv.1 | zero |

- Botao "Evoluir" no HUD do time, ou pelo botao do proprio pokemon na tela Team.
- Destrava quando o pokemon chega ao **hunt level da especie de destino**.
- **So funciona em Cerulean**, e fica travado durante hunt.
- Especie de hunt **ate Lv.39**: 1 de cada stone da receita.
  Especie de hunt **Lv.40+**: **4 stones no total**, divididas igualmente em combos
  (Ivysaur = 4x Leaf Stone; Charmeleon = 2x Fire + 2x Feather).
- Apelido, posicao no time e Growth/Quality sao mantidos; os stats recalculam com a
  base da nova especie.
- **Shiny e Ditto nunca evoluem.**

A RECEITA por especie (quais stones, quantas) so aparece no modal de evolucao, que e
logado. Nao esta em `creatures.json` nem em lugar nenhum publico.

## O Eevee e caso a parte

A propria Pokepedia declara: *"Eevee e caso a parte (sistema proprio de stones)"* — e
nao publica qual. O que acontece:

**Ele nao evolui. Ele e TROCADO com o NPC Marlon**, em Cerulean. As strings do cliente
confirmam o mecanismo (`eeveeReq: "1 Eevee no time"`, `needStone: "Faltam {{stone}}"`,
`tradedVerb: "Trocou seu Eevee por {{name}}"`), mas a tabela mora no servidor.

Custo IGUAL nos cinco: **$65.000 + 10 pedras + 1 Eevee no time**. So a pedra muda.

| Destino | pokeId | Tipo | Pedra |
|---|---|---|---|
| Flareon | 136 | Fogo | 10x Fire Stone |
| Vaporeon | 134 | Agua | 10x Water Stone |
| Jolteon | 135 | Eletrico | 10x Thunder Stone |
| Umbreon | 197 | Sombrio | 10x Darkness Stone |
| Espeon | 196 | Psiquico | 10x Enigma Stone |

O Marlon tambem vende pokemon raro por ouro na mesma tela (Mr. Mime $50.000, Eevee
$65.000, Porygon $85.000).

## O que o catalogo publico diz, e esta errado

```
Eevee: { pokeId: 133, evolvesToId: 134, evolveLevel: 80 }
```

Falso nos dois pontos: nao e por nivel, e nao e um destino so. O campo esta
preenchido, bem tipado e plausivel — o tipo de dado errado que nenhuma validacao
pega. Toda derivacao de cadeia evolutiva que seguir `evolvesToId` cegamente vai
desenhar "Eevee -> nv 80 -> Vaporeon", inclusive nas fichas dos cinco destinos,
porque a cadeia tambem se caminha pra tras.

Os cinco destinos tambem tem ponto de caca proprio (Vaporeon/Jolteon/Flareon no 80;
Espeon/Umbreon no 60), entao dao pra pegar sem Eevee nenhum.

## O que NAO se sabe (marcar, nao inventar)

- **Se a troca devolve o pokemon no nivel em que ele entrou.** O botao diz "Trocar",
  nao "Evoluir", e a garantia de heranca de nivel/quality da Pokepedia e da evolucao
  comum, que o Eevee explicitamente nao usa. Sem isso, projetar stat do destino serve
  pra comparar as cinco especies na mesma regua, nao pra prometer o que chega.
- **Se o Eevee tambem evolui por nivel**, em paralelo a troca. O `evolveLevel: 80` do
  catalogo pode ser resto de outro sistema ou pode valer; a Pokepedia nao responde.
- **Leafeon, Glaceon e Sylveon**: as Bags dos tres existem em `items.json`, mas as
  especies nao existem em `creatures.json` e o Marlon nao os lista. Conteudo futuro,
  provavelmente.

## Onde farmar as cinco pedras

Nenhuma das cinco e vendida por NPC — todas caem de pokemon. Ordenar por
`chance x quantidade media`, nao por chance (ver
[[Ordene pela grandeza que decide, não pela que impressiona]]): Mightyena e Absol
soltam Darkness Stone de 1 a 5 por vez e rendem o triplo de um drop unitario de mesma
chance. Melhor fonte por faixa de nivel, no catalogo de ago/2026:

| Pedra | ate nv 60 | ate nv 100 | ate nv 150 | acima |
|---|---|---|---|---|
| Fire | Charmeleon (4.477) | Magmar (750) | Furious Magmar (567) | = |
| Water | Kingler (2.570) | Golduck (1.856) | Evil Cloyster (838) | = |
| Thunder | Pikachu (2.123) | Electabuzz (1.111) | Magnetic Electabuzz (463) | = |
| Darkness | Haunter (5.889) | Misdreavus (1.578) | Banshee Misdreavus (769) | Mightyena (125) |
| Enigma | Kadabra (2.979) | Alakazam (1.079) | Brave Alakazam (430) | = |

Entre parenteses, abates esperados ate as 10 pedras. A ordem MUDA com o nivel: a
Darkness e a pior de todas ate o 150 e disparado a melhor a partir do 470.

## Conexoes
- Irma: [[Poke Idle World - endpoints publicos de dados]] ·
  [[Poke Idle World - regras de breeding]]
- Tecnica: [[A interface do sistema explica o que a API dele esconde]]
- Principio: [[Tirar o dado errado não põe a verdade no lugar]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
