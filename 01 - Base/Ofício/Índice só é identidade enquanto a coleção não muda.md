---
tags: [tipo/atomica, camada/principio, armadilha, dev/backend]
criado: 2026-08-24
---

# Índice só é identidade enquanto a coleção não muda

> Posição num array serve de identificador só enquanto o array é o mesmo. No instante em
> que alguém filtra, compacta ou reordena a coleção, o índice 2 passa a apontar pra outra
> coisa — e nada avisa, porque a leitura continua sendo válida.

## A regra

Toda vez que um índice **sobrevive** à coleção que o originou, ele vira um ponteiro solto.
Isso acontece em dois formatos, e os dois passam despercebidos:

1. **O índice atravessa iterações** — um relógio, um contador, um acumulado guardado por
   posição, enquanto a lista indexada é remontada a cada volta.
2. **O índice atravessa a fronteira** — uma função recebe a coleção já filtrada, devolve
   resultados numerados pela ordem em que recebeu, e quem chamou lê aquilo como se fosse a
   posição na coleção original.

Nos dois casos a correção é a mesma: **guarde pela chave estável, não pela posição.** Se a
coleção precisa ser filtrada, ou o estado se indexa pelo conjunto completo (com os
descartados presentes e neutralizados), ou cada item carrega a própria identidade e o
índice deixa de significar qualquer coisa.

## Por que

O erro não gera exceção. `arr[2]` existe, devolve um objeto do tipo certo, e o programa
segue — com o valor errado. É a classe de defeito mais cara de achar, porque o sintoma
aparece longe da causa e só em alguns dados.

Dois casos concretos, os dois no motor de arena do [[piwdex2]]:

- **O relógio que atravessa o laço.** O combate de time contra boss guarda a recarga de
  cada golpe do boss, e ela não reinicia quando o pokémon do jogador cai — o boss estava
  lutando. Só que a lista de golpes era filtrada por *quem atravessa a defesa de quem
  está na frente*, e isso muda a cada troca: um golpe Elétrico some contra um membro
  Terra e volta contra o próximo. Com a lista filtrada, o índice 2 deixava de ser o mesmo
  golpe entre um membro e outro, e um golpe de 30 segundos herdava o relógio de um de 5 —
  disparando seis vezes mais do que devia. A correção foi indexar pelo moveset inteiro e
  deixar o golpe que não atravessa com dano zero, em vez de tirá-lo da lista.
- **O número que vazou pra tela.** O motor recebia só os slots PREENCHIDOS do time e
  numerava as passagens pela ordem em que recebeu. A tela lia esse número como "#N do
  time". Com buraco no meio (slots 1, 3 e 6 ocupados), o mesmo pokémon aparecia como "#3"
  na carta e "#2" na fila do combate — dois números pro mesmo bicho na mesma página, e o
  de baixo sempre errado.

O segundo é o mais instrutivo: o motor estava **certo**. Ele numerou o que recebeu, que é
tudo que ele podia fazer. O defeito nasceu na fronteira, onde a posição foi tratada como
identidade sem ninguém decidir isso.

## Na prática

- Filtrou antes de passar? Devolva a identidade junto com o resultado, ou remapeie na
  volta. Nunca deixe o chamador adivinhar.
- Estado que persiste entre iterações se indexa pelo **conjunto estável**. Se um item
  precisa sair da conta, neutralize o efeito dele (peso zero) em vez de tirá-lo do array.
- Suspeite de todo `for (let i = 0; ...)` em que o `i` é guardado em algum lugar que vive
  mais que o laço.

## Conexões
- Depende de: [[Estado mutável se lê da fonte no uso, não de cópia guardada]]
- Irmã: [[Um invariante se garante na estrutura, não no processo]]
- Técnica que aplica: [[Campo cujo nome você não sabe se lê do payload, nunca se chuta]]
- Visto em: [[piwdex2]]
- Mapa: [[Base]] · [[Backend]]
