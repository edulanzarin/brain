---
tags: [tipo/atomica, camada/principio]
criado: 2026-08-26
---

# Onde não há regra, espelhar é mais honesto que arbitrar

> Num conferidor, o que não tem regra não se cobra: copia-se o real para o lado
> esperado, e a linha fecha em zero. Arbitrar um valor onde não existe regra não
> produz conferência — produz uma diferença que fala do conferidor, não do mundo.

## A regra

Todo conferidor compara duas colunas: o que deveria ser e o que é. A tentação é
preencher a primeira coluna sempre, porque coluna vazia parece defeito. Mas
esperado inventado tem o mesmo peso visual do esperado calculado, e a diferença
que ele gera é indistinguível de um achado.

Então cada item se classifica antes de comparar: **tem regra** (compara) ou
**não tem** (espelha). E a classificação é por item, não por balde — espelhar a
conta inteira porque "a maioria ali não tem regra" esconde justamente o item
com regra que fugiu dela.

## Por que

No balancete fiscal, a granularidade errada do espelho custou duas vezes:

- espelhar **por conta**: a nota de natureza genérica caía numa conta que outras
  naturezas regram, então o gate por conta recusava o espelho e a nota ficava
  sem contrapartida no lado esperado — uma falta do tamanho exato dela, com cara
  de lançamento faltando;
- não espelhar **nada**: cada nota de serviço virava "conta errada", porque a
  régua cobrava uma conta que ninguém mais usava.

O critério que funcionou foi por NOTA: quem tem regra compara, quem não tem
espelha, esteja onde estiver.

## Na prática

- A pergunta é sempre "existe regra **para este item**?", não "esta conta costuma
  ter regra?".
- Espelho é `esperado := real`, o que zera a linha e mantém o total fechando —
  a reconciliação continua exata, e é isso que prova que não se perdeu nada.
- O que espelha some da lista de divergências. Deixar como "provável não-erro,
  confira manualmente" é meio caminho: a linha continua lá, continua sendo lida,
  continua custando atenção.

## Conexões
- Depende de: [[Espelhar por balde esconde item no lugar errado]]
- Irmã: [[Config declarada envelhece; quem diz a regra é o comportamento observado]]
- Técnica que aplica: [[Conta da natureza de serviço vem do hábito, não da tabela do ERP]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Base]]
