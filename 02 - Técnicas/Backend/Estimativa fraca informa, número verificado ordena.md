---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-23
---

# Estimativa fraca informa, número verificado ordena

> Quando o número que ordena uma lista soma um termo medido com um termo estimado, a
> incerteza do estimado passa a mandar na ordem inteira. O estimado costuma ser o de maior
> erro E o de maior peso, então ele decide — e a lista continua com cara de precisa.

## O problema

O total parece a resposta completa: rendimento é loot mais venda, receita é assinatura
mais consumo, custo é infra mais suporte. Somar as parcelas num número só é natural, e
ordenar por ele é o passo seguinte.

Só que as parcelas não têm a mesma qualidade. Uma sai de contagem; a outra sai de um
modelo ajustado. Somadas, a lista herda o erro da pior sem avisar — e a soma esconde qual
metade está falando.

No [[piwdex2]] o ouro por abate era loot mais o líquido da captura, e uma sessão medida no
jogo (738 abates) separou as duas:

| parcela | medido | modelo | |
|---|---|---|---|
| loot por abate | 443,7 | 414,5 | 0,93x |
| chance de captura | 1 em 738 | 1 em 129 | **5,7x** |
| ouro/h total | 236k | 434k | 1,84x |

O loot batia drop a drop, item por item. A captura errou por 5,7, e como a bola é cobrada
em **todo** abate — capturando ou não — o termo inteiro trocou de sinal: a tela prometia
+194k/h onde a sessão entregou −30k/h. O buraco de 198k/h era 100% captura; o loot puxava
27k para o outro lado.

O efeito na recomendação: o topo da lista era um alvo que só liderava pela captura
inflada. Tirando ela da soma, o primeiro lugar passou a ser justamente onde o jogador já
estava caçando por conta própria.

## A solução

**O termo verificado ordena; o estimado vira coluna ao lado, marcada como estimativa, e
não entra no que decide.** Não é esconder: é tirar do número que manda.

E há um jeito de a coluna estimada ainda ser útil sem pedir fé: **mostrar o limiar que
decide, porque o limiar costuma ser aritmética pura.**

```
captura só paga a bola quando   chance * valorDeVenda > precoDaBola
ou seja                          chance > 130 / 60.000  =  1 em 462
```

Esse "1 em 462" não depende do modelo — sai de dois preços. O que depende do modelo é a
chance ao lado dele. Com os dois na tela, o leitor confere numa sessão o que nenhuma
revisão de código pegaria.

## Como se descobre qual parcela está errada

Comparando **por termo**, não pelo total. O total estava 1,84x alto: aplicar um fator de
correção global teria estragado o loot, que estava certo. A decomposição contra uma
medição real é o que diz onde mexer — e no caso do loot dá pra ir mais fundo ainda,
comparando a contagem esperada de CADA drop com a observada, o que de quebra confirmou o
teto de chance funcionando como o modelo supunha.

## O que mais vale lembrar

- **Termo que pode ficar negativo é o mais perigoso da soma.** Erro num termo positivo
  muda a magnitude; erro num termo que troca de sinal muda a recomendação.
- **A parcela estimada costuma ser a que mais pesa**, e não por acaso: ela é estimada
  justamente porque o sistema não publica o dado, e o que o sistema não publica costuma
  ser o que ele considera valioso.
- **Modelo que se declara aproximado tem que ser tratado como aproximado na fronteira,
  não só no comentário.** A lei de captura dizia no próprio cabeçalho que servia "pra
  ORDENAR alvos, não como número exato" — e mesmo assim estava somada dentro do número
  que ordenava tudo. Aviso em comentário não segura decisão de produto.
- Fora de jogo: LTV estimado somado a receita realizada num ranking de contas, custo de
  frete previsto somado a preço de tabela, tempo de fila estimado somado a tempo de
  processamento medido.

## Conexões
- Princípio: [[Ordene pela grandeza que decide, não pela que impressiona]]
- Irmã: [[Total acumulado premia a lentidão quando o tempo é livre]] ·
  [[Bônus multiplicativo só rende onde há folga até o teto]] ·
  [[A tela não afirma mais precisão do que a fonte tem]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
