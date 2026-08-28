---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-08-24
---

# Produtividade se mede pela hora do registro, não pela data do fato

> Todo dado de trabalho carrega duas datas — quando o fato aconteceu e quando alguém o registrou. Medir gente é somar pela SEGUNDA; medir o negócio é somar pela primeira.

## A regra

Uma linha de sistema transacional quase sempre tem o par: `data do fato` (competência, data da nota, data de admissão, data do lançamento contábil) e `data/hora do registro` (o carimbo de auditoria de quando a linha nasceu). São perguntas diferentes:

- **Quanto o negócio movimentou em maio?** → data do fato.
- **Quanto a equipe trabalhou em agosto?** → hora do registro.

Trocar as duas parece detalhe e não é: lançamento de maio digitado em agosto é trabalho de agosto. Somado pela data do fato, ele aparece no mês que já foi entregue — e some do mês em que a pessoa efetivamente sentou e fez.

## Por que

O erro é silencioso: o número sai bonito, ninguém percebe. Quem fecha um mês atrasado (o normal em escritório contábil, onde julho fecha em agosto) some da própria produtividade; e o mês antigo "ganha" trabalho depois de já ter sido avaliado. Pior: o placar fica **instável**, porque a mesma consulta rodada duas vezes dá números diferentes para o mesmo mês passado.

O sinal de que a escolha está errada é esse: **produtividade de período fechado não pode mudar**. Se muda, você está somando pela data do fato.

## Na prática

- Verifique se a fonte tem o carimbo antes de prometer a tela: sem `datahora*`, não há placar de pessoa — só de negócio.
- O carimbo costuma ter índice próprio (é coluna de auditoria consultada por período), então filtrar por ele não é mais caro.
- Deixe explícito na interface qual data recorta o período. Quem olha assume competência por hábito.
- O mesmo par aparece fora da contabilidade: chamado (aberto × resolvido), pedido (emitido × faturado), tarefa (vencimento × conclusão).

## O nome da coluna não é prova; a consulta é

O caso mais forte deste princípio veio de onde ninguém procurava: uma coluna chamada
`datalctofis` ("data de lançamento fiscal"), que a documentação do schema — inclusive a
minha própria nota — descrevia como a data em que a nota foi lançada. Era a data do
DOCUMENTO. O carimbo de verdade morava ao lado, em `datahoralctofis`, e não era "a
mesma data com hora".

A pergunta que resolveu cabe numa linha e leva segundos:

```sql
select count(*) filter (where datahoralctofis::date = datalctofis) as iguais,
       count(*) as total
  from lctofissai
 where datalctofis between '2026-07-01' and '2026-07-31';
-- iguais: 0    total: 591.542
```

Zero em meio milhão. Não é "às vezes divergem": são grandezas diferentes com nomes
parecidos, e o escritório inteiro estava sendo medido pela errada havia meses. A tela
mostrava um placar plausível — que é exatamente como este erro sobrevive.

**Antes de construir qualquer coisa em cima do par, conte quantas linhas têm as duas
datas iguais.** Se a resposta for "todas", talvez sejam mesmo a mesma coluna em dois
formatos. Se for "nenhuma", você achou o par — e provavelmente achou também um relatório
antigo somando pelo lado errado. É o mesmo reflexo de
[[Config declarada envelhece; quem diz a regra é o comportamento observado]]: o que
descreve o dado envelhece, o dado não.

Corolário para a interface: a tela tem de **dizer por qual data ela recorta**, porque
quem lê assume competência por hábito. E, na mesma medida, quem escreve a consulta
assume que o nome da coluna diz a verdade.

## O mesmo par quando a fonte é um sistema de fora

Fora do transacional o par continua existindo, com outros nomes: `Last-Modified` do
arquivo é a data do FATO (quando o fornecedor publicou), e a hora em que a sua rotina
baixou é a data do REGISTRO. A regra vira: **carimbe o fato pelo relógio de quem o
produziu, e guarde o seu só como auditoria.**

No [[piwdex2]], o diário de patches do jogo datava pelo relógio da máquina que rodou a
ingestão. Com uma rotina de 6 em 6 horas, um patch publicado às 21h de terça sai datado
como quarta de manhã — e quem tenta cruzar isso com o que sentiu jogando erra por dois
dias. A fonte respondia o `Last-Modified` desde sempre; era só ler.

O sinal de erro é o mesmo de cima: **se a data de um fato passado muda quando você troca
a cadência da coleta, você está carimbando pelo observador.**

## Conexões
- Irmã: [[Auditar o registro, não só o agregado]] ·
  [[Diferença entre duas leituras só fala do mundo se o instrumento não mudou]]
- Irmã: [[Rendimento é vazão vezes tempo em pé, não vazão de pico]]
- Irmã: [[Config declarada envelhece; quem diz a regra é o comportamento observado]] —
  lá é a config que mente sobre a regra; aqui é o nome da coluna que mente sobre o dado.
- Técnica que aplica: [[Grão fino numa varredura só dispensa os count distinct]] ·
  [[Escada ordinal empresta a forma entre domínios, nunca os cortes]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Base]]
