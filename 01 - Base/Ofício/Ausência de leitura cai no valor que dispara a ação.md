---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-23
---

# Ausência de leitura cai no valor que dispara a ação

> "Não consegui ler" não é um valor do domínio. Quando vira um — zero, lista
> vazia, `null` que o chamador lê como falso —, ele não cai num ponto qualquer
> da escala: cai justamente na ponta que manda agir.

## A regra

Toda leitura que alimenta uma decisão automática tem **três** respostas, não
duas: o valor, o outro valor, e **não sei**. A terceira não escolhe por conta
própria — ela suspende a decisão e volta na próxima rodada.

Não saber quanto tem é razão para **não** agir.

## Por que

Porque a coerção não é neutra. O padrão de uma leitura que falhou é sempre o
mesmo punhado de valores — `0`, `[]`, `null`, `false` —, e num sistema que
decide por limiar esses são exatamente os valores do lado "está faltando, faça
alguma coisa". O bug não produz um número errado: produz **ação máxima**,
repetida a cada ciclo, sem exceção e sem log.

Três aparições, em camadas diferentes do mesmo sistema:

| Onde | A leitura sumiu | Virou | O que o sistema fez |
|---|---|---|---|
| cache de catálogo | `HEAD` da sonda falhou | "mudou" | baixou 1,6 MB a cada 10s, para sempre |
| ficha de item | a fonte publica `chance: 0` | "nunca cai" | derivou média e ranking em cima de um zero que ninguém mediu |
| compra automática | o frame de inventário ainda não chegou | "zero poções" | comprou o alvo inteiro a cada minuto, com a bolsa cheia |

O terceiro é o mais caro porque gasta dinheiro, e os três são o mesmo erro: um
ramo de falha que devolve um valor plausível em vez de devolver a própria falha.

Nos três o modo degradado é **silencioso**. Nada estoura, nada aparece na tela,
o dado continua com cara de certo — o sintoma mora na conta de banda de outra
pessoa, ou no saldo, ou num ranking que ninguém confere.

## Na prática

- **O ramo de erro devolve o desconhecido, não um padrão.** `catch { return
  null }` só presta se o chamador tratar `null` como terceiro estado. Quando o
  chamador faz `?? 0`, o `catch` virou um gerador de zeros.
- **Nomeie a terceira resposta.** Um tipo que só tem número obriga a inventar
  um. `number | null`, um tri-estado, um `Result` — qualquer coisa que force o
  chamador a decidir o que fazer quando não sabe.
- **Cópia viva não é fonte para decisão que custa.** Estado empurrado (frame de
  socket, evento, push) nasce vazio a cada conexão e é limpo quando ela cai:
  "vazio" ali quer dizer "ainda não chegou", nunca "não tem". Decisão que gasta
  relê da fonte autoritativa na hora.
- **A tela também precisa da terceira.** Desenhar `0` onde não se sabe leva a
  pessoa a mexer no parâmetro errado para resolver um problema que não existe.
- **Teste apagando a leitura**, não corrompendo. Derrube a fonte, mate o frame,
  devolva `404` — e veja o que o sistema faz. O caminho de erro raramente é
  exercitado, e é onde mora a ação máxima.

## Conexões
- Irmã: [[Estado mutável se lê da fonte no uso, não de cópia guardada]] ·
  [[Laço que trata toda falha igual apaga a causa da primeira]] ·
  [[Contador que conta sucesso de promessa afirma que deu certo]]
- Técnica que aplica: [[Sonda que falhou não é sinal de que mudou]] ·
  [[Zero na tela é afirmação, não valor de conforto]]
- Mapa: [[Base]]
