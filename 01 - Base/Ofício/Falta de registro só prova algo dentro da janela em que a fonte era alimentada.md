---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-09-25
---

# Falta de registro só prova algo dentro da janela em que a fonte era alimentada

> "Não tem recibo, então não gozou férias" e "não tem demissão, então está ativo"
> são a mesma inferência: a falta de uma linha lida como fato. Ela só vale no
> intervalo em que alguém alimentava a fonte com aquele tipo de linha. Antes de
> começar e depois de parar, o silêncio da tabela não diz nada sobre o mundo.

## A regra

Toda fonte tem uma janela por entidade: do primeiro ao último registro que alguém
fez sobre ela. Dentro da janela, a falta de uma linha esperada é indício. Fora
dela, é só a fonte calada.

Antes de concluir algo pela ausência, ache as duas pontas da janela daquela
entidade (a primeira e a última vez que a fonte registrou qualquer coisa sobre
ela) e só conclua dentro delas. As pontas quase nunca estão num campo: saem do
comportamento, da primeira e da última linha de atividade.

## Por que

Dois casos no mesmo cálculo de férias vencidas, um em cada ponta:

- **Antes da janela.** Funcionário admitido em 1988 aparecia com 33 períodos de
  férias vencidos: a regra derivava os períodos da admissão e chamava de vencido
  todo período sem recibo. Só que a empresa passou a ter a folha no sistema
  décadas depois; as férias dos anos 90 aconteceram fora dele. Com a primeira
  folha do contrato como ponto de partida, os casos de 5 ou mais períodos
  vencidos no escritório caíram de 65 para 3.
- **Depois da janela.** 3,3 mil dos 8 mil contratos "sem demissão" não tinham
  folha havia mais de 120 dias: empresa que deixou o escritório, contrato
  duplicado que nunca teve folha. Ninguém registra a demissão de quem já não é
  cliente. Eram quase todas as 1.469 "férias vencidas" do painel; exigindo folha
  recente, sobraram 126.

O erro não aparece como erro. A consulta roda, o número é grande e vermelho, e
parece o achado mais urgente da tela. É [[Tirar o dado errado não põe a verdade no lugar]]
pelo lado da inferência: a falta vira afirmação.

## Na prática

- Para cada "sem X, então Y" numa regra, pergunte desde quando e até quando a
  fonte registraria X para aquela entidade.
- A prova de vida é a atividade (folha, lançamento, nota), nunca o campo de
  situação cadastral: ele é escrito uma vez e envelhece
  ([[Config declarada envelhece; quem diz a regra é o comportamento observado]]).
- O que ficar fora da janela sai da conta e aparece num contador à parte ("1
  contrato sem folha há 120 dias ficou de fora"), como manda
  [[Ausência só aparece contra o universo, nunca contra a tabela de eventos]].

## Conexões
- Depende de: [[Config declarada envelhece; quem diz a regra é o comportamento observado]]
- Irmã: [[Ausência só aparece contra o universo, nunca contra a tabela de eventos]] · [[Código sem cadastro se prova pelo comportamento do dado, não pelo rótulo herdado]]
- Técnica que aplica: [[Contrato sem demissão não prova funcionário ativo no Questor]]
- Visto em: [[NaveX]] · [[Navetech Hub]]
- Mapa: [[Base]]
