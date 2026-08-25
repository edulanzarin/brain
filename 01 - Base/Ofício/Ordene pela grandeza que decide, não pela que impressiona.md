---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-08-19
---

# Ordene pela grandeza que decide, não pela que impressiona

> Toda lista ordenada responde a uma pergunta. Se o número que ordena não for a
> grandeza da **decisão** que o leitor vai tomar, a lista aponta com confiança para a
> escolha errada — e continua parecendo certa, porque cada número nela está correto.

## A regra

Antes de ordenar, escreva a pergunta que o leitor faz. Depois confira se o número que
você ordenou responde **àquela** pergunta e não a uma vizinha mais fácil de calcular.
A vizinha fácil quase sempre é uma **parcela** da grandeza real:

| A pergunta | A parcela que engana | A grandeza que decide |
|---|---|---|
| qual golpe mata mais rápido | poder do golpe | poder ÷ recarga (dano por segundo) |
| quem aguenta mais dano | HP + defesa | HP × defesa (o que se absorve) |
| onde gastar o bônus | quem rende mais | quanto o bônus ainda cabe até o teto |
| qual opção rende mais na hora | vazão de pico | vazão × tempo em pé |
| esse item é bom | posição na fila | nota contra régua fixa |

O padrão comum: **a grandeza real tem duas dimensões e a métrica intuitiva mostra uma
só**. A esquecida costuma ser tempo (frequência, recarga, uptime), multiplicação (dois
fatores que se compõem em vez de se somar) ou folga (quanto ainda cabe antes do teto).

## Por que

Ordenar pela parcela não produz erro visível. Nenhum número está errado, nada estoura,
nada loga. O defeito aparece só como **decisão ruim tomada com confiança** — e como a
tela mostra um número plausível ao lado do nome, o leitor não tem motivo pra duvidar.

Pior: a parcela e o risco costumam crescer juntos. O golpe de maior poder é o de maior
recarga. O alvo de maior XP é o que te mata. O spot mais rico é o mais saturado. Quem
ordena pela parcela não erra por acaso — erra **sistematicamente para o pior lado**.

## O caso extremo: a grandeza que NÃO varia

Existe uma versão desta armadilha em que a métrica intuitiva não é uma parcela — é uma
constante. Aí a lista ordenada por ela não erra: ela empata, e o empate é lido como
"tanto faz", que costuma ser a conclusão mais errada disponível.

No [[piwdex2]], as cinco trocas do Eevee cobram exatamente o mesmo: $65.000, dez pedras
e um Eevee. Uma calculadora de custo responderia "os cinco custam igual, escolha o que
você gosta" — e estaria certa em cada número e errada no conselho, porque uma das cinco
pedras pede 5.889 abates no nível 60 e outra pede 2.123.

**Eixo constante é o sinal de que a decisão mudou de lugar.** O preço em ouro é igual
porque o jogo quis que ele não decidisse; o que decide é o que ele deixou variar — de
onde cai a pedra, e o que a espécie vale em combate. A ferramenta que aceita o empate
para de ser ferramenta.

A pergunta prática, quando as opções empatam na métrica óbvia: **o que o desenhista do
sistema deixou diferente?** Ali está a decisão, e quase sempre é uma coisa que a fonte
não publica em número — tem de ser derivada.

## Na prática

- Calcule as duas dimensões separadas e mostre as duas. Um número só ordena; dois
  discutem, e o leitor entende por que o segundo ganhou do primeiro.
- Desconfie da métrica que o próprio sistema exibe. Ela existe pra ser mostrada, não
  necessariamente pra decidir — o Power somado do Poke Idle World inclui Velocidade, um
  stat que nenhum sistema publicado do jogo usa em combate.
- **Peso em fator sem efeito é ruído com aparência de rigor.** Se você não sabe mostrar
  onde a variável muda o resultado, ela não entra no índice.
- Se a grandeza certa for cara de calcular, calcule mesmo assim para ordenar e deixe a
  barata para exibir. A ordem é o que decide.
- **Empate perfeito é suspeita, não resposta.** Se as opções não se separam pela métrica
  que você escolheu, o problema é a métrica.

## Conexões
- Irmã: [[Rendimento é vazão vezes tempo em pé, não vazão de pico]] (o caso em que a
  dimensão esquecida é o tempo de pé) · [[Auditar o registro, não só o agregado]] (o
  agregado é outra forma de olhar uma dimensão só)
- Técnica que aplica: [[Bônus multiplicativo só rende onde há folga até o teto]] ·
  [[Tier é nota com régua fixa, não posição na fila]]
- Visto em: [[piwdex]] · [[piwdex2]] · [[Idle Game]]
- Mapa: [[Base]] · [[Backend]]
