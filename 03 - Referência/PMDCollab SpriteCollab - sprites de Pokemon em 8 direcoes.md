---
tags: [tipo/referencia, camada/referencia]
criado: 2026-08-25
---

# PMDCollab SpriteCollab - sprites de Pokemon em 8 direcoes

> Acervo comunitário de sprites no estilo Pokémon Mystery Dungeon, servido como
> arquivo bruto no GitHub. É a fonte que resolve o problema mais caro de
> qualquer jogo top-down com Pokémon: **arte andando pra todo lado**, sem
> desenhar nada.

Repositório: `github.com/PMDCollab/SpriteCollab`
Base bruta: `raw.githubusercontent.com/PMDCollab/SpriteCollab/master`

## O que tem, por espécie

Pasta por número da Pokédex, com **quatro dígitos e zero à esquerda**
(`sprite/0025/` = Pikachu):

| Arquivo | Conteúdo |
|---|---|
| `AnimData.xml` | geometria de cada animação: `FrameWidth`, `FrameHeight`, `Durations` |
| `Walk-Anim.png` | folha de caminhada |
| `Idle-Anim.png` | folha de parada |
| `Attack-Anim.png`, `Shoot-Anim.png`, ... | as demais |
| `portrait/0025/Normal.png` | retrato de interface, com variações de expressão |

Retrato fica em `portrait/`, não em `sprite/`.

## A chave: linha é direção, coluna é frame

**Linha 0=S · 1=SE · 2=L · 3=NE · 4=N · 5=NO · 6=O · 7=SO.** Verificado
visualmente, não presumido: recortando o frame 0 de cada linha do Charizard, o
focinho aponta pra direita na linha 2 e pra esquerda na linha 6.

O tamanho do frame **varia por espécie** (Bulbasaur 40x40; espécie grande, mais),
e o número de frames também — de 2 a 17, com a maioria em 4. Por isso a geometria
tem que sair do `AnimData.xml` de cada uma, nunca de uma constante.

`<CopyOf>` marca animação que reusa outra: se aparecer, não há folha própria.

`Durations` vem em frames de 60fps. `ms = duracao * 1000 / 60` preserva o ritmo
original da arte em vez de inventar uma velocidade.

## Cobertura e peso

**968 das 1025 espécies** têm caminhada (ago/2026). As 57 que faltam são de
geração recente, ainda não desenhadas. Baixar caminhada + parada + retrato de
todas dá **~22 MB** — cabe versionado no repositório, e vale: o build deixa de
depender da rede.

O baixador tem que tolerar 404 por espécie sem quebrar, porque a falta é normal.

## Licença

**CC BY-NC 4.0**: uso não comercial, com atribuição. Cada sprite tem autores
próprios, listados em `credit_names.txt` e `tracker.json` na raiz do repositório.
Projeto que for publicado precisa listar os autores das espécies que usa.

## Por que importa

A arte do Poke Idle World é sistema de outfit em canvas, proprietária e presa ao
jogo (ver [[Poke Idle World - endpoints publicos de dados]]). O SpriteCollab é
independente, tem licença declarada e entrega o que aquela não entrega:
**movimento em oito direções**. Para uma ferramenta que só mostra o Pokémon, a
arte do jogo é mais fiel; para um jogo em que ele anda, é esta.

## Conexões
- Irmã: [[Poke Idle World - endpoints publicos de dados]]
- Visto em: [[Vespéria]]
- Mapa: [[Projetos]]
