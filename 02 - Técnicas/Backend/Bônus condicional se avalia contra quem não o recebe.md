---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-20
---

# Bônus condicional se avalia contra quem não o recebe

> Quando um bônus só vale para parte dos candidatos, a lista tem que continuar mostrando
> os outros. Filtrar pelos que recebem transforma "quem ganha o bônus" em "quem paga
> mais" — e essas duas perguntas costumam ter respostas diferentes.

## O problema

O sistema anuncia uma vantagem condicional: hoje o tipo Sombrio rende +20%, esta semana a
categoria X tem desconto, este cliente tem taxa reduzida. A tela óbvia é a lista dos que
se encaixam na condição, ordenada por rendimento.

Essa lista responde à pergunta errada. Ela pressupõe que o bônus decide, quando o bônus é
só uma parcela: um alvo que rende 3.000 sem bônus continua ganhando de um que rende 150
com +20%. No piwdex, com o bônus do dia ligado, **o topo do catálogo não mudou** — os
cinco alvos que mais pagavam por abate seguiram sendo os mesmos, nenhum deles do tipo
premiado. Uma lista filtrada teria mandado trocar de caçada com cara de otimização.

O erro é difícil de ver porque toda linha está certa. O que está errado é o recorte.

## A solução

**Ranqueie o conjunto inteiro com o bônus aplicado onde ele vale**, e marque quem
recebeu. A comparação passa a acontecer dentro da própria lista, e o leitor vê num
relance se o bônus mudou ou não a ordem.

Três colunas fecham a leitura:

| Coluna | Responde |
|---|---|
| rendimento com o bônus | quem eu escolho |
| ganho do dia (contra o mesmo candidato sem o bônus) | quanto o bônus adicionou aqui |
| aproveitamento | quanto do bônus este candidato consegue converter |

A segunda coluna é o **contrafactual** e é ela que impede a ilusão: calcular o mesmo
candidato duas vezes, com e sem a condição, custa quase nada e é a única evidência de que
a vantagem existe. A terceira aparece quando a grandeza satura — ver
[[Bônus multiplicativo só rende onde há folga até o teto]].

Ofereça também **simular outra condição** (outro tipo, outra categoria). Quem opera todo
dia quer saber se amanhã muda o plano, e o mesmo motor já responde.

## O que mais vale lembrar

- **Bônus condicional não pode entrar como percentual global.** Se ele multiplicar todo o
  catálogo, ninguém troca de posição e a conta infla tudo por igual — o sintoma some e o
  erro fica. O multiplicador é função do candidato.
- **Cobrir uma das grandezas e não a outra é o erro que não dá erro.** Quando o bônus
  paga em mais de uma moeda ("+20% de XP e +20% de loot"), implementar metade não estoura
  nada: a metade que falta simplesmente não acontece, e o número resultante continua
  plausível. No piwdex2 o Tipo do Dia entrava no ouro e não entrava no XP, e o que isso
  escondia numa rota 352→500 eram 17 horas. Ao ler o anúncio, liste as grandezas que ele
  toca antes de escrever a conta.
- **Deixe onde receber o bônus que você não conhece.** Evento de servidor, boost de loja,
  trilha de progressão: se não há campo pra informar, o número sai curto e ninguém
  descobre por quê. No piwdex2 o XP/h saía a 0,66x do real por falta de um campo, e o
  cálculo estava certo em tudo o mais.
- Vale para qualquer promoção segmentada: cupom por categoria, cashback por bandeira,
  frete grátis por região. A pergunta do usuário nunca é "o que tem desconto", é "o que
  sai mais barato".

## Conexões
- Princípio: [[Ordene pela grandeza que decide, não pela que impressiona]]
- Irmã: [[Bônus multiplicativo só rende onde há folga até o teto]]
- Parente: [[Número de regra alheia se lê da fonte, não se congela em constante]] · [[Num confronto, medir só o seu lado recomenda o alvo que te destrói]]
- Visto em: [[piwdex]] · [[piwdex2]]
- Mapa: [[Backend]]
