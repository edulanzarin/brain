---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-26
---

# Config declarada envelhece; quem diz a regra é o comportamento observado

> Quando a configuração de um sistema diz uma coisa e o histórico dele mostra
> outra, sistematicamente, quem está desatualizado é a configuração. Cobrar pela
> config declarada não acusa erro: fabrica erro, todo mês, no mesmo lugar.

## A regra

Config é uma afirmação feita **uma vez**; comportamento é uma afirmação feita a
cada lançamento. A primeira envelhece calada — ninguém revisa cadastro que não
dá erro —, a segunda acompanha o mundo. Então, num conferidor, a config entra
como **hipótese**, e o histórico como **prova**.

O teste que separa um caso do outro é a **sistematicidade**: se todas as
ocorrências daquela chave desviam para o mesmo lugar, é a regra que mudou; se
uma desvia e as outras não, é aquela ocorrência que está errada. Erro real é
exceção — quando "o erro" é 100% dos casos, o errado é a régua.

## Por que

Dois casos no mesmo sistema, a um ano de distância:

1. **"Este CFOP contabiliza?"** — a config do Questor errava dos dois lados
   (CFOP com tabela que nunca lança, CFOP sem tabela que lança). O sinal que
   funcionou foi o histórico: em 12 meses, a maioria das notas do CFOP foi
   contabilizada?
2. **"Que conta esta natureza de serviço recebe?"** (ago/2026) — a tabela de
   contabilização apontava para uma conta que a empresa aposentou. Em 443
   naturezas, TODAS as notas caíam noutra conta, e em 79% delas a conta
   configurada não tinha um centavo de movimento no trimestre. O conferidor
   marcava cada NFSE como conta errada — dezenas por empresa, todo mês.

O custo não é o falso positivo isolado: é a **erosão da confiança**. Uma tela
que acusa quarenta erros iguais todo mês deixa de ser lida, e junto com o ruído
morrem os dois achados de verdade que estavam na mesma lista.

## Na prática

- Aprenda a regra do histórico e guarde o aprendizado **do seu lado**, com a
  evidência junto ("40 notas, 39 nesta conta"). A config vira reserva de quem
  ainda não tem histórico.
- Exija **dominância**, não maioria simples: abaixo de ~80% não há hábito, e
  fingir que há é trocar um erro falso por outro.
- **Sem hábito, não há regra** — e o que não é regra não se cobra. Ver
  [[Onde não há regra, espelhar é mais honesto que arbitrar]].
- Não corrija a fonte por conta própria: config alheia é dela. Mas **mostre** a
  divergência, para quem cuida do cadastro poder consertar.

## Conexões
- Irmã: [[Onde não há regra, espelhar é mais honesto que arbitrar]] · [[Estimativa desmentida pela realidade vira veto temporário do motor]]
- Técnica que aplica: [[Conta da natureza de serviço vem do hábito, não da tabela do ERP]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Base]]
