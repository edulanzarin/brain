---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-23
---

# Total acumulado premia a lentidão quando o tempo é livre

> Total é taxa vezes duração. Quando a duração não está fixada, oferecer o total como
> objetivo faz o sistema escolher a opção **mais lenta** — e nenhum número da tela fica
> errado, o que torna o defeito invisível.

## O problema

A tela mistura duas atividades numa só. Uma tem linha de chegada (subir até o nível 500,
processar até o fim da fila); a outra não tem (juntar ouro, faturar, acumular). Quando a
segunda vira um **modo** da primeira, ela herda a meta da primeira e o resultado passa a
ser "quanto acumulei até chegar lá".

Esse número é taxa vezes horas, e as horas são justamente o que a outra atividade quer
minimizar. Maximizar o total vira maximizar o tempo gasto.

No [[piwdex2]], a rota de treino tinha um botão "ganhar ouro" ao lado de "subir rápido".
Com o mesmo pokémon e o mesmo alvo de nível:

| Modo | Caçada | Tempo | Ouro/h | "Ouro no caminho" |
|---|---|---|---|---|
| subir rápido | Furious Scyther | 99h | **459k** | 45,7M |
| ganhar ouro | Scyther | 176h | 447k | **78,7M** |

O modo chamado "ganhar ouro" apontava a caçada que paga **menos por hora**. Os 78,7M não
vinham de render mais, vinham de demorar 77 horas a mais. E como o número grande da tela
era o acumulado, ele confirmava a escolha errada.

## A solução

**A taxa é o objetivo; o total é consequência.** Quem quer decidir onde caçar decide por
ouro/h, e o acumulado só aparece como leitura secundária do que já foi decidido.

Se a atividade sem linha de chegada precisa de uma meta própria, **inverta a pergunta em
vez de emprestar a meta da outra**:

| Atividade | Pergunta | Entrada | Resposta |
|---|---|---|---|
| com linha de chegada | quanto falta pra chegar lá | nível alvo | horas |
| sem linha de chegada | quando eu junto o que quero | quantidade alvo | horas |

Nos dois casos a resposta é tempo, e nos dois casos o mesmo número que ordena a lista
**encurta** a espera. Demorar deixa de poder virar vitória.

## O que mais vale lembrar

- **O teste é uma pergunta só: essa atividade tem linha de chegada?** Se não tem, ela não
  pode receber a meta de outra. Duas perguntas de forma diferente pedem dois lugares, e
  espremer as duas num seletor de dois botões é o que produz a contradição.
- **O sintoma é dois modos da mesma tela se desmentindo.** Se o modo A exibe, num campo
  qualquer, um valor melhor do que o modo B naquilo que B promete otimizar, o objetivo de
  B está errado ou a tela está mostrando a coisa errada. Vale como conferência barata:
  rode os dois modos e compare todas as colunas, não só a que cada um persegue.
- Fora de jogo é o mesmo desenho: "total faturado no período" como meta quando quem
  escolhe o período é o usuário; "linhas processadas" como nota de qualidade de um job;
  "horas registradas" como medida de entrega. Em todos, o denominador sumiu.
- Cuidado com o inverso, que é igualmente comum: **taxa sem escala não decide compra**.
  Ouro/h não se compara com preço de loja, então a tela ainda deve traduzir a taxa em
  janelas concretas (1h, 8h, 24h). O que não pode é a janela virar o critério de ordem.

## Conexões
- Princípio: [[Ordene pela grandeza que decide, não pela que impressiona]]
- Irmã: [[Taxa que muda ao longo do trecho se integra, não se amostra na ponta]] ·
  [[Bônus condicional se avalia contra quem não o recebe]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
