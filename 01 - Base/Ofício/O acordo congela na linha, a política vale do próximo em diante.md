---
tags: [tipo/atomica, camada/principio, dev, dados]
criado: 2026-09-01
---

# O acordo congela na linha, a política vale do próximo em diante

> Todo sistema que cobra tem números que valem hoje: um preço, um percentual de
> comissão, uma alíquota, um desconto de plano. Eles moram no cadastro porque
> precisam mudar. E é exatamente por precisarem mudar que **o que já foi
> combinado não pode continuar apontando para eles**.

## A regra

No momento em que uma transação acontece, os termos dela viram **colunas da
própria linha**: o preço cobrado, o percentual aplicado, o valor de cada lado.
A tabela de política continua onde estava, e a partir dali ela só decide o
próximo.

O teste é uma pergunta: **mudar este número no cadastro muda algum relatório do
mês passado?** Se muda, o termo não foi congelado.

## Por que

Porque a consequência é a pior categoria de defeito: silenciosa, retroativa e
sobre dinheiro. No [[Privello]], a casa fica com um percentual da assinatura de
perfil, e o percentual mora no anúncio porque o acordo é negociado por pessoa.
Se a carteira somasse `preço × percentual de hoje`, baixar a taxa de 20% para
15% faria o saldo de todo mundo subir sozinho — inclusive o de quem já sacou.
Subir de volta faria descer. Ninguém teria feito nada, e o número seria outro.

O mesmo vale para o preço. Quem assinou por R$ 29,90 continua pagando R$ 29,90
até renovar; com o preço lido do cadastro na hora do extrato, a linha de três
meses atrás passaria a exibir o valor de hoje, e a soma do extrato deixaria de
bater com o que entrou na conta.

## Na prática

- **Congele o resultado, não só a entrada.** Guardar o percentual e recalcular
  ainda deixa o arredondamento livre: se a regra de arredondar mudar, o passado
  muda junto. Guarde o valor de cada lado, já calculado.
- **Arredonde a favor de quem sofre o erro.** Dividir R$ 29,90 em 80% dá
  R$ 23,92 exatos, mas nem toda divisão fecha: o centavo que sobra fica com a
  casa, porque para o outro lado a soma dos repasses de um mês ultrapassaria o
  que entrou.
- **A linha congelada é o comprovante.** É ela que responde "por que recebi
  isso?" meses depois, e é a única resposta que não depende de reconstruir qual
  era a política naquele dia.
- Vale além de dinheiro: qualquer termo que a pessoa aceitou olhando —
  a versão do contrato, o limite do plano no dia da compra, o prazo prometido.

## Conexões
- Irmã: [[Estado mutável se lê da fonte no uso, não de cópia guardada]] — o
  contraponto que delimita: dado que descreve o AGORA se lê da fonte, dado que
  descreve um ACORDO se copia. A pergunta que separa os dois é se o passado tem
  o direito de mudar.
- Irmã: [[A definição em dado dirige o comportamento, não um caso no código]] —
  a política mora em dado justamente para mudar; congelar é o que impede que
  mudá-la vire retroativo.
- Irmã: [[Balancete é movimento do período, saldo é consequência]]
- Visto em: [[Privello]]
- Mapa: [[Base]]
