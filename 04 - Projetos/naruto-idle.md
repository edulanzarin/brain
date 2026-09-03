---
tags: [tipo/projeto, projeto/naruto-idle]
criado: 2026-09-03
---

# naruto-idle

RPG idle de navegador em pixel art top-down. O jogador é **um ninja** e sobe de
Estudante a Kage: escolhe aldeia e natureza de chakra na criação da conta, caça no
mapa, aprende jutsu e enfrenta o boss do rank.

código em: `~/Dev/naruto-idle` · app `4080` · banco `5080`

## Estado

Núcleo jogável de pé, rodando em produção local. O que existe:

- conta e criação de ninja em quatro passos (nome, aldeia, chakra, confirmar)
- mundo top-down em tela cheia, 13 mapas, movimento com diagonal
- caçada idle com progresso offline e teto por rank
- mochila e jutsus funcionando (equipar, tirar, aprender pergaminho)
- catálogo de componentes em `/sistema`

Falta: missão, exame de rank, boss jogável, loja.

## Decisões que valem lembrar

**A arte não é desenhada aqui.** Vem do Ninja Adventure Asset Pack (Pixel-boy e AAA,
CC0), recortada por `npm run arte:importar` a partir de uma declaração — 107 entradas
de manifesto, com dimensão lida do PNG. Pixel art desenhada por assistente está
proibida no `GAME_RULES.md`: foi o que reprovou o poke-idle. O Eduardo escolheu "pack
pronto" sabendo que não existe pack de Naruto; a identidade dos personagens vem de
**nome + retrato + emblema de aldeia**, não de sprite exclusivo.

**Vida e XP de inimigo e boss não são escritos à mão.** `src/data/intencao.ts`
declara a intenção e `npm run calibrar` resolve os números —
[[Número que depende de outros números se declara em intenção e se deriva por comando]].
`npm run balanco` confere 12 regras e reprovou 26 coisas na primeira passada.

**O mundo ocupa a tela inteira e o HUD flutua.** A primeira versão era grade de duas
colunas com o mundo numa delas, e foi reprovada em uma frase —
[[Jogo ocupa a tela inteira e o HUD flutua sobre o mundo]].

**Mapa é ASCII mais legenda, gerado por script.** `npm run mapas:gerar` é
determinístico por semente e reprova sozinho se alguma zona ou saída não for
alcançável da entrada.

**O servidor mede o tempo da caçada.** O cliente nunca informa quanto tempo passou —
se informasse, informaria 24 horas. Um resolvedor só serve o tick ao vivo e a volta
do offline.

## Conhecimento que saiu daqui

- [[Número que depende de outros números se declara em intenção e se deriva por comando]]
- [[Jogo ocupa a tela inteira e o HUD flutua sobre o mundo]]
- [[Terreno de tile se desenha em duas camadas, base cobrindo e sobreposição com borda]]
- [[Eixo de folha de sprite se confere sprite a sprite, não se supõe]]
- [[Slug de asset ganha tipo derivado do manifesto gerado]]

## Comandos

```
npm run arte:importar    recorta o pack -> public/arte + manifesto
npm run mapas:gerar      gera os 13 mapas
npm run calibrar         resolve vida e XP a partir da intenção
npm run balanco          relatório de balanceamento (12 regras)
npm run teste:fluxo      teste ponta a ponta contra o banco
```

Skills do projeto em `.claude/skills/`: `game-assets`, `game-design`, `game-ui`,
`game-architecture`, `sprite-animation`, `world-generation`, `game-balancing`.

## Conexões
- Usa: [[Infra]] · [[Design]]
- Mapa: [[Projetos]]
