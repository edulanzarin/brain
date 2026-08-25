---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-25
---

# Tirar o dado errado não põe a verdade no lugar

> Remover um valor falso não deixa a tela em silêncio: deixa ela no caminho PADRÃO,
> e o padrão também afirma. Metade das vezes ele afirma o oposto do falso, que é
> igualmente mentira — e agora sem nada errado no código pra encontrar.

## A regra

Quando você apagar, cortar ou deixar de fora um dado que estava errado, faça a
pergunta seguinte antes de fechar: **o que a ausência dele passa a dizer?**

São três respostas, e só uma é aceitável:

- **A ausência não diz nada** — o campo some, a linha some, o painel some. Raro.
- **A ausência diz "não sei"** — é o certo, e quase sempre precisa ser escrito à
  mão, porque nenhum padrão de linguagem significa isso.
- **A ausência diz outra coisa** — é o caso comum e o perigoso. O ramo `else` do
  componente, o zero da soma, a lista vazia: todos são frases completas.

Cortar dado ruim é meio trabalho. O outro meio é escrever o que fica no lugar.

## Por que

Porque o padrão foi escrito para o caso comum, não para o caso que você acabou de
esvaziar. Ele é uma afirmação sobre um mundo que não é mais o seu, e é feita com a
mesma confiança visual do dado de verdade — mesma fonte, mesma cor, mesma caixa.

Pior que o erro original: o erro original tinha um valor errado em algum lugar,
que dá pra procurar. Depois do corte não há valor nenhum errado — há um `else`
correto respondendo à pergunta errada. Nada loga, nada estoura, e o texto lido é
gramaticalmente impecável.

Duas aparições, as duas no [[piwdex2]] e por caminhos opostos:

- **O que sobrou depois do corte.** O catálogo do jogo afirma que o Eevee evolui
  pro Vaporeon no nível 80. É falso duas vezes: não é por nível e não é um destino
  só (são cinco trocas com um NPC, uma pedra cada). Cortada a aresta falsa, a ficha
  passou a cair no texto padrão do painel de linha evolutiva:

  > "Eevee não evolui e não vem de nenhuma evolução — é uma linha de um estágio só."

  Que é falso pelo outro lado, e em seis fichas. O Eevee VIRA cinco coisas; o que
  não existe é a seta de nível. A correção só ficou pronta quando o painel ganhou o
  texto próprio do que acontece de verdade, com o caminho pra tela que explica.

- **O que sobrou depois da omissão.** A penalidade de grupo do boss ficou de fora
  da conta enquanto era palpite, e isso estava certo — ver
  [[Fator que domina o resultado não entra na conta por estimativa]]. Só que a tela
  não ficou calada por isso: ela dizia "fatia 100%, aguenta infinito" sobre um Golem
  que no jogo morria no primeiro golpe. Omitir o fator produziu uma afirmação, e a
  afirmação era a mais otimista possível.

Os dois casos têm a mesma forma: **o vazio caiu no ramo mais afirmativo que
existia**. Não é coincidência — o ramo padrão costuma ser o do caso feliz, porque é
o que se escreve primeiro.

## Na prática

- Depois de cortar, LEIA a tela. Não o diff: a tela. O defeito só existe em texto
  renderizado, então revisão de código não pega.
- Se o dado saiu porque estava errado, o lugar dele fica com o que você SABE, mesmo
  que seja pouco: "isto acontece de outro jeito, veja ali". Um ponteiro vale mais
  que um silêncio, e muito mais que um padrão.
- Desconfie de todo `else`, `?? 0`, `|| []` e "estado vazio" que herdou um caso que
  não existia quando ele foi escrito.
- Corte a origem, não o sintoma: aqui a aresta falsa foi recusada na derivação
  (`evolutionChainOf`), num lugar só, e não em cada tela que a desenhava.

## Conexões
- Irmã: [[Ausência de leitura cai no valor que dispara a ação]] ·
  [[Fator que domina o resultado não entra na conta por estimativa]]
- Depende de: [[A tela não afirma mais precisão do que a fonte tem]]
- Técnica que aplica: [[A interface do sistema explica o que a API dele esconde]]
- Visto em: [[piwdex2]]
- Mapa: [[Base]]
