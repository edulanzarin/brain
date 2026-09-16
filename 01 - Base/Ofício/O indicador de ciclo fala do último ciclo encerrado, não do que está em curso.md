---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-09-16
---

# O indicador de ciclo fala do último ciclo encerrado, não do que está em curso

> Quando o trabalho medido só termina depois de o ciclo fechar, medir o ciclo
> corrente é cobrar trabalho que ainda não podia existir. A unidade é o ciclo, e
> o período escolhido só diz quais ciclos entram.

## A regra

Há duas famílias de indicador. Uma mede **fluxo**: quantos lançamentos, quantas
horas, quantas notas — e o dia de hoje conta, porque cada linha já aconteceu. A
outra mede **conclusão de ciclo**: o mês fechou, o período aquisitivo completou,
a apuração saiu. Nessa, o evento que prova a conclusão chega **depois** de o
ciclo terminar.

Para a segunda, a referência é o **último ciclo já encerrado**. O ciclo em curso
aparece na tela — ele existe —, mas fora da conta.

## Por que

O erro não dá erro: ele mostra zero. E zero é um número plausível, que parece
diagnóstico e é só calendário.

**O caso que ensinou.** Na aba de Fechamento do Nexo, "fechada" quer dizer que
alguém lançou o encerramento do exercício na competência
([[Fechamento mensal no Questor - a conta de Encerramento do Exercício]]). Esse
lançamento entra depois do mês virar, em geral no mês seguinte. A aba media a
última competência **do período**, e o período padrão da barra é "do dia 1º até
hoje": em 16/09 ela media set/26 e mostrava o escritório inteiro em aberto, 0%
fechado no ranking por analista. O relatório estava certo e inútil — quem abria
tinha que recuar o período à mão para ler o número que importa. O aviso na tela
("set/26 ainda está em curso, recue o período") era a confissão de que o padrão
estava errado: se a tela sabe qual é a leitura certa, é ela que deve fazê-la.

**O mesmo erro noutro domínio.** No controle de férias, o período aquisitivo em
curso não entra na lista de períodos a gozar: só o que já completou gera
direito. Contar o que está em andamento inventaria férias que o funcionário
ainda não tem — a mesma troca, com prejuízo invertido.

## Na prática

- **Puxe o último ciclo encerrado para dentro, em vez de avisar.** Se o período
  pedido não contém ciclo encerrado nenhum, estenda-o até o último que fechou.
  Mandar a pessoa recuar o filtro é transferir a regra para quem lê.
- **Mostre o ciclo em curso sem medi-lo.** Ele some da conta, não da tela: some
  da tela, alguém vai perguntar se o sistema esqueceu o mês.
- **Diga de que ciclo são os números**, no cabeçalho do quadro e do ranking. Sem
  isso, ninguém distingue "ninguém fechou" de "não era para ter fechado ainda".
- **A janela da consulta é o ciclo inteiro, não o intervalo pedido.** Intervalo
  de dias serve para escolher ciclos; depois de escolhidos, cada um conta do
  primeiro ao último dia. Senão o período que termina no dia 9 perde o
  fechamento lançado no dia 28, e a empresa consta em aberto sem estar.
- **O teste que revela:** abra a tela no dia 2 do mês. Se ela mostra o
  escritório todo em falta, o indicador está medindo o ciclo em curso.
- **Regra de calendário vai para função pura com teste** — mês anterior, virada
  de ano, fevereiro bissexto. Ela erra calado e cara.

## Conexões
- Irmã: [[Produtividade se mede pela hora do registro, não pela data do fato]] —
  lá a escolha é entre as duas datas da linha; aqui é entre os dois ciclos, o que
  fechou e o que está correndo.
- Irmã: [[Uma pendência de prazo fecha por ato explícito, não por sinal inferido]]
- Depende de: [[Ausência só aparece contra o universo, nunca contra a tabela de eventos]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Base]]
