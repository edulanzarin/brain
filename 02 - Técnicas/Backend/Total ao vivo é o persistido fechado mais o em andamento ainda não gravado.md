---
tags: [tipo/atomica, camada/padrao, dev/backend, dados]
criado: 2026-08-17
---

# Total ao vivo é o persistido fechado mais o em andamento ainda não gravado

> Um dashboard cumulativo que só grava o total no FIM de cada unidade de trabalho (a hunt,
> o mês, a corrida) fica **zerado durante** a unidade — o usuário está gerando número e o
> painel mostra 0 até fechar. Somar a cada evento resolve o "durante", mas espalha escrita e
> convida a inconsistência.

O padrão: o total exibido é `persistido (unidades JÁ fechadas) + em andamento (o valor
vivo, ainda não gravado)`. O persistido é a fonte durável; o vivo vem do estado em memória
da unidade corrente (o analyzer, o contador da sessão). Some os dois **na leitura**.

O perigo é dupla-contagem na virada. A regra que evita: o valor vivo só entra enquanto a
unidade **não foi persistida**. No fim dela, um passo persiste o acumulado E o marca como
fechada (limpa o estado vivo / muda o status) — atômico o suficiente pra a próxima leitura
ver `persistido + 0`, nunca `persistido + vivo` de novo. A leitura checa esse status antes
de somar o vivo.

## Visto em

No piwdex, as Estatísticas do robô mostravam Caçada (derrotados, capturados, itens, XP)
zerada durante a hunt — só somavam no `logSummary` (fim da hunt). Passou a exibir o
persistido (`robot_sales`) + o analyzer AO VIVO, somado só quando `status=running` e há
hunt ativa. Ao parar, o `logSummary` persiste e a hunt deixa de ser "live" — sem
dupla-contagem.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Parente: [[Balancete é movimento do período, saldo é consequência]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]] · [[Dados]]
