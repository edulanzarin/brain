---
tags: [tipo/referencia]
criado: 2026-08-25
---

# Poke Idle World - TM e os discos

> O TM e o maior salto de poder do jogo, e diferente da troca do Eevee ele esta
> INTEIRO em fonte publica: o `creatures.json` marca cada golpe de TM com o campo
> `tm`, entao quem aprende o que se deriva do catalogo. Ver
> [[Poke Idle World - endpoints publicos de dados]].

Visto em: [[piwdex2]]

## O tamanho do salto

| | Poder por segundo |
|---|---|
| Melhor golpe NATURAL das 482 especies | 43,3 |
| Todo golpe de TM | **60,0** (600 de poder / 10s) |

Nao existe golpe natural que chegue perto. O TM nao e um upgrade incremental: e
outra categoria.

## Os quinze golpes

Um por tipo, todos com `learnLevel: 1` (nao ha trava de nivel). O tipo do GOLPE
sempre bate com o tipo do DISCO, entao o campo `tm` do golpe e a identidade do
disco — nao precisa de tabela a mao.

| Disco | Golpe | Categoria | Especies jogaveis |
|---|---|---|---|
| Agua | Cyclonic Tide | especial | 24 |
| Voador | Massive Hurricane | fisico | 21 |
| Inseto | Hive Crush | fisico | 13 |
| Planta | Petal Bloom | especial | 9 |
| Terrestre | Crossing Fissure | fisico | 9 |
| Fogo | Ignition Point | especial | 8 |
| Psiquico | Mental Singularity | especial | 7 |
| Sombrio | Endless Hollow | fisico | 7 |
| Eletrico | Static Overload | especial | 6 |
| Venenoso | Toxic Deluge | fisico | 6 |
| Dragao | **Draconic Soul** | especial | 6 |
| Pedra | Crumbling Rain | fisico | 6 |
| Lutador | Meteor Fists | fisico | 5 |
| Gelo | Ice Time | especial | 5 |
| Fantasma | Untold Nightmare | especial | **1** (so o Gengar) |

**Draconic Soul tem 300 de poder, e nao 600.** E o unico dos quinze pela metade, e
quem troca o disco de Dragao esperando o salto dos outros leva metade dele.

## As tres armadilhas

1. **Normal, Aco e Fada tem DISCO e nao tem GOLPE.** Os tres itens existem em
   `items.json` (`Normal-Type TM Disk` e companhia) e nenhuma das 482 especies
   aprende um golpe de TM desses tipos. Hoje trocar pecas por eles nao tem em quem
   usar.
2. **O `AoE TM Disk` e outra coisa.** Ele nao ensina golpe: faz os golpes NORMAIS do
   pokemon acertarem em area (string do cliente: "all of the Pokémon's Normal moves
   start splashing"). Como o efeito nao esta no moveset publicado, nenhum motor
   sobre `creatures.json` consegue simula-lo. E decisao de farm, nao de dano num
   alvo so.
3. **O icone dos dezoito discos elementais e o MESMO arquivo**
   (`/assets/items/tm_disk_elemental.png`). Quem desenhar uma grade contando com
   arte distinta por tipo vai ter dezoito celulas iguais.

## Como se consegue

O NPC **TM Researcher** troca `{{n}}× TM Disk Piece` por **um disco a sua escolha**,
com o mesmo `n` pra qualquer tipo — as pecas sao consumidas. O `n` mora no servidor.

Isso repete o formato da [[Poke Idle World - evolucao e a troca do Eevee]]: **custo
igual em todas as opcoes**, entao o preco nao separa nada e a decisao inteira e
"quem aproveita". Ver
[[Recurso indivisível se aloca pelo salto, não pelo resultado]].

## Numeros de referencia (catalogo de ago/2026)

- 116 especies jogaveis aprendem ao menos um TM; **17 aprendem dois** (Charizard,
  Gyarados, Dragonite, Nidoking, Aerodactyl…). O Researcher entrega um disco por
  troca, entao a conta certa e por disco e nao pela soma dos dois.
- Maior salto do catalogo: **Jolteon, 7,62x** com o disco Eletrico (1.495 -> 11.395
  de poder por segundo), e ele sobe de tier B pra S.
- Menor salto: **Togetic, 1,39x** com Voador. Quem ja tem moveset natural bom ganha
  pouco.
- O salto NAO muda com nivel nem com quality: os dois lados usam o mesmo stat
  ofensivo multiplicado pelo mesmo fator `(nivel/100) * quality^0.8`, que cancela. O
  que nao cancela e o IV, quando o melhor golpe natural e o do TM usam stats
  diferentes (um fisico, outro especial).

## Conexoes
- Irma: [[Poke Idle World - endpoints publicos de dados]] ·
  [[Poke Idle World - evolucao e a troca do Eevee]] ·
  [[Poke Idle World - regras de breeding]]
- Tecnica: [[Recurso indivisível se aloca pelo salto, não pelo resultado]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
