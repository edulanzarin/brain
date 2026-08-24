---
tags: [tipo/atomica, camada/base, dados, contabil]
criado: 2026-07-22
---

# Espelhar por balde esconde item no lugar errado

> Numa reconciliação em que o que não tem regra **espelha o real** (fiscal = real, sem falso positivo), a granularidade do espelho decide o que você enxerga. Espelho **por balde** (conta, categoria) é cego a item no lugar errado: o balde errado é espelhado e mostra zero de diferença. E forçar o "lugar certo" a aparecer sem mexer no espelho **dobra o valor** (o certo pelo motor + o errado pelo espelho), quebrando o invariante do total. A saída é o espelho decidir **por item**: quando o motor reproduz um item, o real daquele item **sai do espelho** — a versão do motor o substitui. Aí o balde certo fica com o item a mais (+), o errado com ele a menos (−), os dois se anulam no total e o invariante se mantém.

## A segunda armadilha: substituição parcial vira fantasma

Excluir do espelho **tudo** de um item que o motor reproduziu só em parte cria diferença fantasma: os pedaços que o motor **não** reproduz (componentes irmãos, casos fora da regra) são varridos sem substituto. A exclusão tem que ser tão estreita quanto a substituição — só sai do espelho o que o motor **comprovadamente** repôs. Regra prática: exclusão cirúrgica no exato gatilho em que a reprodução aconteceu, nunca "excluir tudo do item porque ele foi tocado".

## Por que importa

Duas perguntas parecem a mesma e não são: "o total bate?" (lente agregada) e "cada item está no lugar certo?" (lente item a item). Espelho por balde responde só a primeira. Com o espelho por item, a lente agregada passa a mostrar também o deslocamento — em pares que se anulam — sem perder a honestidade (o que o motor não reproduz continua espelhado e não gera falso positivo).

A armadilha tem uma versão **temporal**: quando a "substituição" não é imediata (um
throttle, uma fila que roda de hora em hora), o item sai da visão A no instante em que é
_classificado_ como "vai ser tratado por B", mas B só o trata depois — na janela entre
os dois ele não está em lugar nenhum. Ou a exclusão espera a substituição de fato
acontecer, ou a substituição vira imediata. Não existe "vai ser tratado" invisível.

## Conexões
- Visto em: [[Navetech Hub]] — Balancete Fiscal: empresa com dezenas de notas na conta errada dava "tudo ok"; espelho por nota (com exclusão só no bypass) expôs os pares ±, mantendo dupla partida e reconciliação exata; excluir tudo da nota reproduzida criou fantasma de milhões (PIS/COFINS a recuperar sem substituto).
- Visto em: [[piwdex]] — robô: pokémon capturado que batia a trava de venda saía do acervo ("vai ser vendido") mas a venda era 1x/hora — ficava em limbo (nem vendido nem no acervo) até a varredura. Corrigido tornando a venda imediata (assim que coleta), fechando a janela.
- Visto em: [[piwdex2]] — robô: o piso de reposição somava todas as bolas (596) e o auto-catch consumia UMA (Ultra Ball, zero em estoque); a soma alta segurava a compra e a captura ficava parada. Ver [[Limiar conta a unidade que se consome, não o balde que a contém]].
- Parente: [[Balancete é movimento do período, saldo é consequência]]
- Mapa: [[Base]] · [[Dados]]
