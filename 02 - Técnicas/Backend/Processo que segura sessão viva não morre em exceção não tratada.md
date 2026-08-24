---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-24
---

# Processo que segura sessão viva não morre em exceção não tratada

> Deixar o processo cair na exceção é o comportamento certo para um servidor sem estado e
> errado para um que segura sessão viva. Quem decide não é o tamanho do erro: é **o que o
> processo tem dentro que não dá pra refazer**.

## A pergunta que decide

Antes de escolher entre morrer rápido e seguir vivo, responda o que se perde na queda:

| O processo segura | Morrer custa | O certo |
|---|---|---|
| nada além do request em curso | o cliente repete | **morrer** — estado sujo é pior que reinício |
| conexão viva por usuário, sessão que o outro lado limita a uma, trabalho de horas | tudo isso, de todo mundo | **sobreviver e registrar** |

Crash-only é um bom conselho **porque** pressupõe a primeira linha. Aplicá-lo na segunda
troca uma inconsistência possível numa perda certa.

## A armadilha do Node

Desde o Node 15 uma promessa rejeitada sem `catch` derruba o processo. Ou seja: todo
`void isso()` disparado e esquecido é um gatilho, e ele não parece um — a linha é curta,
o método é interno, e a rejeição pode vir de um `fetch` que hoje nunca falha.

Pior quando o processo é único: o motor cai por causa de **uma** conta e leva a de todas
as outras. E o sintoma engana — o boot religa sozinho e o log escreve "Ready" igual ao do
processo que rodava há horas.

A superfície mais exposta é sempre a que roda código seu sobre dado que **outro** escolheu:
o handler de mensagem de um socket de terceiro. O outro lado muda de forma quando quiser,
um campo vira `null`, e o `throw` sobe por dentro do emissor, onde não há nada pra pegá-lo.

## Três camadas, da origem pra fora

1. **Na origem**: um `protegido(oQue, fn)` que envolve cada disparo e cada handler.
   Recebe **função**, não promessa — `protegido(() => f())` ainda pega o `throw` imediato,
   `protegido(f())` já explodiu antes de entrar.
2. **No meio**: o erro vira evento no feed do dono, com throttle por operação
   (ver [[Falha de automação recorrente vira alerta com throttle, não catch vazio]]).
3. **Na borda**: `uncaughtException` e `unhandledRejection` que registram e **não** saem.
   É a última rede, para o que ninguém previu — não o lugar de tratar erro conhecido.

## O contador é obrigatório

Rede que apara em silêncio é um jeito mais lento de esconder defeito. O número de vezes
que ela aparou tem que sair em algum lugar que se lê (a sonda de saúde serve): sobreviver
a cinquenta exceções por hora não é o sistema estando bem, é o sistema mascarando algo que
precisa de conserto na origem.

## Conexões
- Princípio: [[Rendimento é vazão vezes tempo em pé, não vazão de pico]]
- Irmã: [[Falha de automação recorrente vira alerta com throttle, não catch vazio]] · [[Chamada externa tem timeout e erro tratado]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
