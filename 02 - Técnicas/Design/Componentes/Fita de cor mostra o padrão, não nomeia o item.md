---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-09-09
---

# Fita de cor mostra o padrão, não nomeia o item

> Uma sequência de quadrados coloridos — fita de competências, sparkline,
> mapa de calor de dias — responde muito bem "como está indo". Ela não responde
> "qual". Se a pergunta que a pessoa faz começa com *qual*, o rótulo precisa
> estar escrito, não no title do quadrado.

## O que a fita faz bem, e o que ela não faz

A fita existe porque doze selos numa linha viram parede de texto. Ela troca
leitura por forma: **três verdes seguidos de dois amarelos** é um padrão que se
vê de longe, e essa é a força dela.

O custo aparece na hora de agir. Para dizer *qual* mês está aberto, quem lê tem
que contar quadrados a partir de uma ponta — e nem sabe qual ponta, porque a
fita não diz onde começa. O `title` resolve para quem sabe que ele existe, tem
mouse e já desconfia de qual quadrado olhar; não resolve para quem só bateu o
olho.

O sintoma é o pedido, não a reclamação. Na aba de fechamento contábil do Nexo a
fita estava lá, com legenda e tooltip, e a primeira pessoa a usar a tela pediu:
*"queria saber quais meses estão fechados"*. Não disse que estava errado — disse
que a resposta não estava ali.

## A correção

Escrever o rótulo debaixo de cada marca, em texto pequeno, e marcar a
referência:

- **o rótulo mora na fita**, não no title: 10 px de altura por coluna, e a
  pergunta passa a se responder de olhar;
- **a marca de referência** (borda mais forte, rótulo em peso maior) diz onde é
  "agora" — sem ela a sequência não tem âncora e o leitor conta de novo;
- **o rótulo encolhe até onde ainda identifica**: `set` basta dentro de um ano;
  `set/26` só quando o período atravessa a virada. Ano repetido em doze colunas
  é ruído, e ruído é o que fez a fita nascer sem rótulo.

Continua valendo o resto: cor sozinha não é estado, então situação e item vão
por escrito no rótulo acessível de cada marca, e a legenda diz o que cada cor
significa ([[Nota carrega só o que a pessoa não sabe]]).

## O teste

Antes de escolher fita, escreva a frase que a pessoa vai dizer ao ver a tela. Se
ela começa com "como" ou "quanto", a fita basta. Se começa com **"qual"** ou
**"quando"**, ela precisa de rótulo — ou não era fita, era tabela.

## Conexões
- Princípio: [[Nota carrega só o que a pessoa não sabe]]
- Irmã: [[A unidade se diz uma vez, não em cada rótulo do eixo]] · [[Escada ordinal empresta a forma entre domínios, nunca os cortes]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
