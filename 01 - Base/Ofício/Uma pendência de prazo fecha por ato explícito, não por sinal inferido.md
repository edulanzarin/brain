---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-08-14
---

# Uma pendência de prazo fecha por ato explícito, não por sinal inferido

> Numa fila de coisas com prazo (pagar, entregar, responder), o item só sai da fila quando alguém DECLARA que resolveu. Um sinal derivado da fonte — uma data prevista, um cálculo feito, um registro criado — não pode fechar o item sozinho: ele informa, não conclui.

## A regra

O que dispara o alarme é o prazo; o que apaga o alarme é um **ato**. Modele a
resolução como um dado próprio, gravado por quem confirmou (com data e, se útil,
observação e autoria). Sinais da fonte read-only entram como **coluna de apoio**
("já calculado?", "pagamento previsto para tal dia") — visíveis ao lado do item,
nunca substituindo a confirmação.

## Por que

O sinal inferido quase sempre vem **antes** do fato e com semântica frouxa. No
Questor, a folha de rescisão guarda uma `datapgto` que é a data **prevista** de
pagamento, gravada no cálculo — resolver o item por ela apagaria o alarme dias
antes do dinheiro sair, que é exatamente quando ele mais precisa acender. "Folha
calculada" não é "verba paga"; "nota emitida" não é "recebida"; "evento gerado"
não é "aceito". Fechar por inferência troca um falso-negativo barato (o item fica
na fila mais tempo) por um falso-positivo caro (some da fila sem ter acontecido).

O ato explícito também dá o que a inferência não dá: **quem** confirmou e
**quando** — a trilha de responsabilidade que uma fila operacional exige.

## Na prática

- Situação = `resolvido ? 'feito' : urgência(prazo, hoje)`. A urgência é derivada;
  o `resolvido` é um registro que alguém criou.
- O override de resolução mora no banco do app (a fonte é read-only), chaveado
  pela identidade do item na fonte — [[Sobre fonte read-only, o editável mora no seu banco chaveado pela identidade dela]].
- Deixe o sinal da fonte à mostra para o humano decidir rápido, sem deixá-lo
  decidir sozinho — parente de [[Deixar o método da conferência visível quando o SQL não foi validado]].
- Reabrir tem de ser tão fácil quanto fechar (o ato é reversível).

## Conexões
- Onde o editável vive: [[Sobre fonte read-only, o editável mora no seu banco chaveado pela identidade dela]]
- Irmã: [[A definição em dado dirige o comportamento, não um caso no código]]
- Visto em: [[Navetech Hub]] (Folha → Rescisões a pagar)
- Mapa: [[Base]]
