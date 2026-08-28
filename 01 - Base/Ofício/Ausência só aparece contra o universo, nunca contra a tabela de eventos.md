---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-27
---

# Ausência só aparece contra o universo, nunca contra a tabela de eventos

> Quem não trabalhou, não vendeu, não respondeu ou não foi atendido não tem linha nenhuma no registro de eventos. Para medi-lo, o denominador tem de vir do CADASTRO — e o cadastro raramente diz quem deveria estar lá, então quem diz é o comportamento.

## A regra

Toda tabela de evento responde uma pergunta só: **o que aconteceu**. Ranking, série,
top N, calendário — tudo sai dela e tudo fala dos presentes. A pergunta oposta ("quem
ficou de fora?") não tem resposta ali dentro, por construção: a ausência não gera linha.

Medir ausência exige juntar duas coisas que costumam morar em lugares diferentes:

1. **O universo** — a lista de quem deveria aparecer (cadastro de clientes, de
   colaboradores, de contas a conferir).
2. **O recorte de quem realmente deveria aparecer**, que quase nunca está declarado.

## Por que

Placar de produtividade de escritório contábil: cinco telas mediam quem lançou, quanto,
quando e com que atraso. Nenhuma delas conseguia dizer que **936 empresas ativas
passaram o mês inteiro sem um único lançamento** — porque nenhuma empresa parada aparece
no `lctoctb`. O buraco de atendimento, que é o achado mais caro do mês, era invisível
justamente para as telas feitas para medir atendimento.

O segundo passo é onde a conta erra feio. O cadastro tinha 1392 empresas ativas, mas 197
delas nunca tiveram um lançamento contábil na vida: são clientes de folha ou de fiscal,
que não deveriam entrar na conta de cobertura. Usar o cadastro cru como denominador
afundava o indicador (33%) com empresa que não pertence à pergunta. O ERP não tem campo
de "contrato de contabilidade" — quem separou foi o comportamento: **ativa com
lançamento nos últimos 12 meses** (997 empresas, cobertura real de 46%). É
[[Config declarada envelhece; quem diz a regra é o comportamento observado]] aplicado ao
denominador.

## Na prática

- Antes de escrever a tela, pergunte **quem some**. Se a resposta for "quem não tem
  linha", a tela precisa de uma segunda fonte, não de mais um `group by`.
- O universo custa pouco: é cadastro, tem milhares de linhas, não milhões. Caro é o
  fato — e o fato você já varreu.
- **Quem o recorte descarta continua na tela**, num contador à parte. No mesmo placar,
  as horas de quem usa o sistema mas não é do setor viraram uma linha de rodapé
  ("+64 pessoas de outras áreas, 1875 h fora da conta") em vez de sumirem caladas: o
  corte que ninguém vê é indistinguível de um bug.
- Conferência é o mesmo desenho pelo avesso: nota não contabilizada só existe cruzando
  o universo das notas com os lançamentos. Se o cruzamento não estiver lá, a tela está
  medindo presença e chamando de conferência.

## Conexões
- Depende de: [[Config declarada envelhece; quem diz a regra é o comportamento observado]]
- Irmã: [[Razão só afirma quando os dois lados vêm do mesmo trabalho]]
- Irmã: [[Auditar o registro, não só o agregado]]
- Técnica que aplica: [[Grão fino numa varredura só dispensa os count distinct]]
- Mapa: [[Base]]
