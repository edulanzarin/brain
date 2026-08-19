---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-19
---

# Zero num medidor é estado, não barra vazia

> Barra de vida, quota, bateria, progresso: quando o valor chega a zero, o preenchimento
> some e sobra o trilho — **exatamente a mesma imagem de "ainda não carregou"**. O momento
> mais importante do medidor é o único que ele não sabe mostrar.

## O problema

Um medidor comunica por comprimento. Comprimento zero não é um comprimento curto, é
ausência: o olho lê "sem dado", "não se aplica", "elemento decorativo". Quem projetou viu
a barra cheia o tempo todo e nunca olhou pro estado que importa.

O agravante é a barra sem número. Sem rótulo, não dá nem pra distinguir "0 de 24" de
"vazio porque não chegou". E é justo aí que uma ação é necessária.

## A solução

Zero ganha forma própria, e não só a falta de forma:

- **Número sempre visível** junto da barra. `0/24` não some.
- **O trilho muda**, já que o preenchimento não pode: borda e fundo na cor de alerta,
  com pulso. A caixa continua ocupando o mesmo espaço.
- **Rótulo do estado** (um chip "Desmaiado", "Sem saldo", "Bloqueado") — a palavra é o
  que o usuário repete quando pede ajuda.
- **A ação junto** quando existe uma: o botão que resolve aparece no mesmo card em que o
  problema apareceu, e some quando não há o que resolver.
- Se o dado ainda não chegou, o placeholder esmaecido é OUTRA coisa — ver
  [[Slot com placeholder esmaecido segura o lugar do dado vivo]].

Vale fechar isso num primitivo (barra + número + chip + `estaZerado()`), senão cada tela
redescobre metade da regra.

## Visto em

No piwdex a vida do time já vinha do jogo e já era desenhada — uma barra fina, sem número.
O pokémon líder morreu, a barra ficou vazia igual às caixas sem dado do card, o robô seguiu
"ligado" sem matar nada e o dono só descobriu abrindo o jogo. A barra virou primitivo com
número, trilho vermelho pulsando, chip DESMAIADO e o botão de curar ao lado.

## Conexões
- Princípio: [[Todo estado da tela tem visual]]
- Irmã: [[Slot com placeholder esmaecido segura o lugar do dado vivo]]
- Visto em: [[piwdex]]
- Mapa: [[Design]]
