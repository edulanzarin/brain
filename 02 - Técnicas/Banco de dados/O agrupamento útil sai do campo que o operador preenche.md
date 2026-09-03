---
tags: [tipo/atomica, camada/padrao, sql, dev/backend]
criado: 2026-09-03
---

# O agrupamento útil sai do campo que o operador preenche

> O schema nomeia um campo como "o certo" para aquela dimensão. Quem opera o
> sistema escreve a informação noutro, livre, ao lado. Para relatório, o campo
> livre ganha: ele carrega a taxonomia que as pessoas realmente usam.

## O problema

O ERP tinha os dois campos para a mesma coisa. Um **normalizado**, vindo de um
cadastro global — no Questor, `escala.descrescala`, a descrição do horário. Um
**livre**, digitado por contrato — `funcescala.descrtipojornada`, a jornada por
extenso, que existe porque o eSocial pede a descrição da jornada.

Quebrar o turnover por `descrescala` parecia óbvio: é o campo que o schema chama
de horário. O resultado foi uma tabela em que quase toda linha tinha uma pessoa,
porque cada empresa escreve o horário do seu jeito:

```
13:30 às 17:30 - 18:00 às 22:00/sab 09:00 às 13:00
07:30 às 12:00/13:00 às 17:18
07:30 12:00/13:00 17:18
05:00 AS 09:00 DAS 09:30 AS 13:00 SABADO 05:00 AS 09:00
```

Três dessas são o mesmo horário. Um agrupamento onde nada agrupa não é um
relatório: é uma lista de strings.

Enquanto isso, no campo livre ao lado, o Departamento Pessoal escrevia
`1º turno`, `2º turno`. Vocabulário curto, fechado, estável — uma taxonomia de
verdade, feita por quem convive com o dado, só que sem tabela de domínio que a
declare.

## A solução

Ler o campo do operador primeiro e deixar o normalizado de reserva, na mesma
expressão, para quem não preencheu:

```sql
coalesce(nullif(btrim(descrtipojornada), ''),
         nullif(btrim(descrescala), ''), '(sem horário)') as horario
```

O `coalesce` é o que torna a troca segura: onde o operador escreveu, agrupa pelo
vocabulário dele; onde não escreveu, ninguém perde o que já enxergava. Sem ele a
troca cria um balde gigante de "(sem preenchimento)" e some com metade do
detalhamento.

Duas condições para a expressão valer a pena:

- **Uma expressão só, num lugar só.** Se a dimensão nasce na CTE base, a troca
  vale de uma vez para a quebra, o filtro e o drill. Se estivesse copiada em três
  queries, essa correção seria três correções e uma esquecida.
- **`btrim` + `nullif` sempre.** Campo livre vem com espaço sobrando e com string
  vazia fingindo de nulo; sem normalizar, `"1º turno "` e `"1º turno"` viram dois
  grupos e o problema só muda de lugar.

## O que mais vale lembrar

- **O sinal de que se escolheu o campo errado é a cardinalidade**: se o número de
  grupos cresce junto com o número de linhas, aquele campo não é dimensão.
- **Perguntar a quem preenche vale mais que ler o dicionário de dados.** Este
  ajuste veio do próprio DP, que mandou um print da tela dizendo "puxa do
  `funcescala`, que lá eu escrevo 1º turno". Nenhuma leitura do schema ia contar
  isso: os dois campos são texto e o nome do "certo" é mais bonito.
- **Não normalize por conta própria.** Tentar mapear "13:30 às 17:30..." para
  turno com regex é inventar cadastro alheio; o operador já fez esse trabalho no
  campo do lado.
- Vale para qualquer base de sistema de terceiro — ERP, CRM, protocolo — em que
  exista campo de observação ao lado do campo domesticado.

## Conexões
- Princípio: [[Config declarada envelhece; quem diz a regra é o comportamento observado]]
- Irmã: [[Onde não há regra, espelhar é mais honesto que arbitrar]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Dados]] · [[Banco Questor]]
