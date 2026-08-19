---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-19
---

# Fila de metas pula o item impossível com aviso, e nunca fica parada

> Motor autônomo que cumpre uma meta e para obriga o dono a ser o agendador: ele volta,
> monta a próxima e liga de novo. A fila resolve isso — mas só se ela souber lidar com o
> item que **deixou de fazer sentido** enquanto esperava.

## O problema

Entre enfileirar e executar passa tempo, e o mundo muda: o recurso saiu do time, o alvo já
foi atingido por outro caminho, o parâmetro não resolve mais. As duas reações comuns são
ruins — **travar** (a fila inteira morre num item quebrado, e o motor fica parado sem
ninguém saber) ou **pular calado** (o dono acha que rodou e descobre dias depois).

## A solução

Três regras, e a terceira é a que ninguém lembra:

1. **Valida na entrada E na hora de executar.** A validação do enfileiramento é cortesia; a
   que vale é a de quando o item vira o corrente, com o estado de agora.
2. **Item impossível é pulado com alerta** dizendo qual e por quê, e a fila anda pro
   próximo — no mesmo laço, não na próxima volta. Alerta é o preço de continuar sozinho;
   ver [[Falha de automação recorrente vira alerta com throttle, não catch vazio]].
3. **Fila vazia cai no comportamento contínuo**, não em ociosidade. O motor tinha um jeito
   de operar antes de existir fila; é pra lá que ele volta.

A fila guarda só o que ainda **não** começou — o item corrente vive no campo dele, com o
progresso. Misturar os dois faz o resume depois do restart ter que adivinhar onde parou.
E, como qualquer intenção do usuário, ela é persistida: ver
[[Estado desejado persistido religa o robô depois do restart]].

## O que mais vale lembrar

O limite pequeno (3, 5) não é técnico, é de confiança: fila longa é promessa que o motor
faz sobre um futuro que ele não consegue prever. Prefira poucos itens e um caminho fácil
de reenfileirar.

## Visto em

No piwdex, o plano de leveling terminava e o robô caía no modo auto — upar o próximo
pokémon era refazer tudo na mão. Virou fila de até 3: ao bater a meta, o robô troca o líder
(`poke-summon`), calcula a rota pelo nível real do bicho e entra na primeira faixa. Plano
com pokémon que saiu do time ou que já passou do alvo é pulado com alerta.

## Conexões
- Princípio: [[Guarde a intenção e o processo se reconstrói dela]]
- Irmã: [[Estado desejado persistido religa o robô depois do restart]]
- Parente: [[Falha de automação recorrente vira alerta com throttle, não catch vazio]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
