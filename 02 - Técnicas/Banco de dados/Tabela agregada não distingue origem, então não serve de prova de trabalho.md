---
tags: [tipo/atomica, camada/padrao, dev/backend, sql, armadilha]
criado: 2026-09-08
---

# Tabela agregada não distingue origem, então não serve de prova de trabalho

> Uma tabela de saldo, total ou contador responde QUANTO. Ela não responde QUEM fez nem POR QUE apareceu, porque somou origens diferentes na mesma célula. Quando o relatório vai creditar (ou cobrar) uma pessoa, o marcador tem de sair do registro que carrega autor e carimbo — mesmo que ele custe mais caro.

## O caso

Medir quais empresas tiveram o mês contábil fechado. O fechamento deixa um
lançamento na conta de Encerramento do Exercício, e havia duas fontes:

| fonte | custo (13 meses) | o que é |
|---|---|---|
| `saldoctbmensal` (agregado) | 0,8 s | movimento por (empresa, conta, mês) |
| `lctoctb` (razão) | 0,9 s com [[Predicado redundante é o que faz o índice entrar quando o join casa colunas pareadas]] | o lançamento, com usuário e data/hora |

O agregado bateu com o razão mês a mês em oito competências seguidas — e mesmo
assim estava errado em um caso, que é o caso que importa: uma empresa aparecia
com movimento na conta de encerramento **sem ter um único lançamento**. O saldo
tinha vindo da IMPLANTAÇÃO de saldos (a função de saldo de abertura do ERP, que
grava direto na conta, sem partida dobrada). Contá-la como fechada creditaria a
um analista um fechamento que nunca houve.

A conferência que expôs isso foi simples: para cada par (empresa, mês) marcado
como fechado no agregado, exigir um lançamento correspondente no razão. Um em
882 não tinha — e um a cada 882 é exatamente o tipo de erro que passa despercebido
por meses e depois aparece numa reunião.

## A regra

- **Agregado responde quanto; registro responde quem.** Se a coluna do relatório
  é uma pessoa, a fonte tem de ter autoria.
- O agregado continua útil no que ele responde sem ambiguidade — ali era "esta
  empresa escriturou alguma coisa no mês?", pergunta em que a origem não muda a
  resposta.
- Ao trocar de fonte por causa de custo, **valide o cruzamento, não o total**:
  os totais batiam nas duas.

## Conexões
- Princípio: [[Uma pendência de prazo fecha por ato explícito, não por sinal inferido]]
- Irmã: [[Predicado redundante é o que faz o índice entrar quando o join casa colunas pareadas]]
- Relacionado: [[Fechamento mensal no Questor - a conta de Encerramento do Exercício]]
- Visto em: [[Navetech Hub]] (Contábil → Produtividade → Fechamento)
- Mapa: [[Dados]]
