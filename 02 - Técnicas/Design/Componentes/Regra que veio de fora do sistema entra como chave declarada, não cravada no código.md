---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-24
---

# Regra que veio de fora do sistema entra como chave declarada, não cravada no código

> "O jogo disse que X não vai valer" não é a mesma coisa que "X não vale". A
> primeira é uma informação **de terceiro, com data**; a segunda é uma regra do
> seu sistema. Cravar a primeira como se fosse a segunda transforma a decisão de
> outra pessoa numa decisão sua — e quando ela mudar, o seu sistema mente sem que
> ninguém lembre por quê.

## O caso

Os desenvolvedores de um jogo disseram, num chat, que os pokémon lendários não
seriam jogáveis. A tier list media "quem eu uso", e onze lendários ocupavam o topo
— então tirá-los estava certo.

O detalhe que decide a forma da solução estava na mesma frase que trouxe a
notícia: *"pelo que falaram da última vez, **mas tudo pode mudar**"*. A fonte
entregou o fato e a validade dele junto.

## A forma que isso toma

Uma **chave com padrão**, e não um `filter` escondido:

- o comportamento certo é o padrão, então quem só abre a tela já vê o mundo certo;
- a tela **declara** o recorte por extenso, com o número ("os 11 estão fora") — e
  não com um interruptor mudo cujo efeito ninguém vê;
- o texto diz **de quem é a afirmação** ("os desenvolvedores disseram") em vez de
  apresentá-la como verdade do sistema;
- desfazer é um clique, e o estado vai pra URL como qualquer outro recorte.

O custo disso é uma chave a mais. O custo de cravar é uma lista errada que
continua parecendo certa.

## Onde o critério mora

No **campo da fonte**, nunca numa lista de nomes escrita à mão. `rarity ===
"LEGENDARY"` acompanha o catálogo; `["Mewtwo", "Mew", …]` envelhece no primeiro
patch e ninguém percebe, porque a lista continua funcionando — só que incompleta.

## O efeito colateral que se esquece: a régua

Se o sistema normaliza contra o maior do conjunto, mudar **quem está no conjunto**
muda a referência de todo mundo. Aqui isso é desejável — quem não entra em campo
não pode definir o que é "100 de ataque" —, mas é uma decisão a tomar de propósito
e a escrever, não um efeito a descobrir depois.

E vale medir antes de afirmar: eu escrevi no comentário que tirar os lendários
faria a nota de todos subir, medi, e não subia — o teto era de outro. **Comentário
que promete um efeito numérico é uma afirmação, e afirmação se confere.**

## Conexões
- Princípio: [[Estado compartilhável mora na URL]]
- Irmã: [[Sinal booleano da fonte não ocupa o lugar de uma escala]] ·
  [[A régua sai da distribuição, não dos extremos]] ·
  [[Nota carrega só o que a pessoa não sabe]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
