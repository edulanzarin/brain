---
tags: [tipo/atomica, camada/base, contabil]
criado: 2026-07-21
---

# Balancete é movimento do período, saldo é consequência

> Um balancete não é uma foto de saldos — é a **movimentação de débito e crédito de cada conta no período**. O saldo (anterior, atual) é **derivado** disso: saldo atual = saldo anterior + débitos − créditos. Quem pensa "saldo" primeiro complica à toa (precisa de saldo anterior, encerramento de exercício, etc.); quem pensa "movimento" primeiro resolve com uma soma no período.

## Por que importa

Ao montar ou comparar balancetes, **compare movimento contra movimento** no mesmo período — não saldos acumulados. Isso:

- Dispensa saldo anterior e o tratamento de **encerramento de exercício** (contas de resultado que zeram no fim do ano) — que só aparecem se você insiste em saldo.
- Deixa a comparação "o que deveria ter movimentado × o que movimentou" limpa e local ao período.

Foi a correção que destravou o balancete fiscal do Questor: parei de perseguir saldo e passei a somar débito/crédito por conta no mês.

Não é só contabilidade. O mesmo formato reaparece na **folha**: o efetivo (headcount) é o estoque, admissões e desligamentos são o fluxo; o turnover se calcula do fluxo sobre o efetivo, sem tabela de saldo. Onde há data de início e fim, o estoque numa data é consequência — ver [[Estoque e fluxo numa série a partir de datas de início e fim]].

## O mesmo formato onde o dinheiro é devido a alguém

No [[Privello]] a casa fica com um percentual da assinatura de perfil e deve o
resto a quem publica. Não existe tabela de saldo: o disponível é a soma dos
repasses liberados menos a dos saques que não foram recusados. Três coisas caem
de graça com isso:

- **Recusar um saque não precisa de estorno.** O valor volta ao disponível
  sozinho, porque a linha simplesmente sai da soma. Com saldo guardado, seria
  uma segunda escrita — e é a que um dia alguém esquece.
- **O pedido ainda não pago já sai do disponível.** O movimento que a soma
  considera não é só o consumado: é o consumado mais o reservado. Sem isso a
  pessoa pede duas vezes o mesmo dinheiro, e a casa paga duas.
- **A retenção é uma condição na mesma soma**, não outra tabela: o que entrou
  nos últimos sete dias entra em "a liberar" em vez de "disponível", porque
  estorno e contestação chegam depois do pagamento.

O custo de derivar é somar a cada leitura, e ele só passa a doer quando o
extrato de uma pessoa tiver milhares de linhas. Aí a saída é materializar por
período fechado — que é, de novo, movimento.

## Conexões
- Aplicação em query: [[Estoque e fluxo numa série a partir de datas de início e fim]]
- Visto em: [[Navetech Hub]] (Balancete Fiscal, Folha/Rotatividade) · [[Privello]] (carteira do repasse)
- Dados/plano de contas do Questor: [[Vínculo nota fiscal e lançamento contábil no Questor]]
- Mapa: [[Base]]
