---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-08-19
---

# Estimativa desmentida pela realidade vira veto temporário do motor

> Motor que escolhe por estimativa vai errar. O conserto não é confiar mais no modelo nem
> desligá-lo: é deixar o **fato observado** vetar a recomendação — por um prazo medido na
> unidade do problema, não para sempre.

## O problema

O motor recomenda, o mundo executa, o mundo derruba. Se a recomendação nasce só do
modelo, o ciclo se fecha em loop: o robô cura o pokémon desmaiado e volta para o mesmo
alvo que acabou de matá-lo, morre de novo, cura de novo. Ninguém trava, ninguém reclama —
o sistema só para de render, e o dono descobre horas depois que o nível não subiu.

## A solução

Três decisões:

1. **O fato é o gatilho, não a estimativa.** Um desmaio pode ser azar; dois no mesmo alvo
   é o modelo errado. Conte por episódio (a transição para caído), não por leitura de
   estado — senão o mesmo desmaio conta a cada varredura.
2. **O veto entra na entrada do motor**, como lista de exclusão, e a rota é **recalculada
   inteira**. Trocar só a etapa atual devolve você ao alvo banido na etapa seguinte.
3. **O veto caduca na unidade do problema.** "Dez níveis à frente" é honesto — o que o
   tornava letal era a diferença de força, e ela mudou. Prazo em relógio não descreve nada.

Quando o veto esvazia as opções, **avise** em vez de insistir calado: sem alternativa no
alcance, o dono precisa trocar quem caça ou escolher na mão (ver
[[Falha de automação recorrente vira alerta com throttle, não catch vazio]]).

A memória do veto pertence ao **par** (quem executa, o que é executado): trocar o
executor zera a lista, porque a conta que a produziu era dele.

## Visto em

No piwdex, o Abra nível 14 ficou horas travado morrendo em Gastly: o cérebro corrigiu a
estimativa (passou a medir o dano recebido) e ganhou o veto por cima, para o caso de a
estimativa ainda errar. Dois desmaios no mesmo alvo banem a caçada, refazem o plano e
registram o motivo no feed de eventos.

## Conexões
- Princípio: [[Rendimento é vazão vezes tempo em pé, não vazão de pico]]
- Irmã: [[Num confronto, medir só o seu lado recomenda o alvo que te destrói]]
- Parente: [[Falha de automação recorrente vira alerta com throttle, não catch vazio]] · [[Fila de metas pula o item impossível com aviso, e nunca fica parada]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
