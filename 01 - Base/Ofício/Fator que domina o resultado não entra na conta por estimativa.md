---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-24
---

# Fator que domina o resultado não entra na conta por estimativa

> Quando um termo desconhecido é grande o bastante pra mandar sozinho no resultado,
> estimá-lo é pior do que deixá-lo de fora. Com ele chutado, o número que sai deixa de
> falar do que você modelou e passa a falar do seu chute.

## A regra

Antes de chutar um termo que falta, pergunte **de que tamanho ele é em relação ao resto**.
A resposta decide, e ela tem só dois lados:

- **Termo pequeno ou médio** — entra, mesmo aproximado, e a aproximação se declara. Um
  fator de 10% errado por metade continua deixando a ordenação de pé.
- **Termo que domina** — fica FORA da conta e DENTRO da tela, como aviso. O resultado
  passa a ser "o combate sem essa penalidade", que é uma afirmação verdadeira e útil, em
  vez de "o combate com a penalidade que eu inventei", que é uma afirmação falsa com cara
  de precisa.

Deixar de fora não é desistir: é mudar o que a conta afirma. O que sai continua ordenando
opções entre si, que costuma ser a pergunta real.

## Por que

Estimativa entra num modelo o tempo todo, e está certo que entre — todo motor deste tipo
é aproximação. O que separa a aproximação honesta do número inventado é o **peso**. Um
fator de dezenas de vezes não é mais um termo do modelo: ele vira o modelo. Ordenar por
um resultado assim é ordenar pelo chute.

Dois casos, os dois no [[piwdex2]]:

- **A penalidade de grupo do boss.** A API do jogo devolve, por boss, `members`,
  `strength`, `deficit` e `mult`, e a relação entre os dois últimos é exata:
  `mult = 3^deficit`. O que não estava publicado era como `strength` se calcula nem o que
  `mult` multiplica (HP do boss? dano recebido? recompensa?). Numa observação real, `mult`
  valia 48,7 — quase cinquenta vezes. Enfiar isso na conta faria o resultado ser a
  penalidade, não o time. Ficou fora, com o mecanismo escrito na tela.

  **E aí a regra fechou o ciclo, que é o melhor que podia acontecer com ela.** Dias
  depois, um print da ficha do boss no próprio jogo respondeu as duas perguntas de uma
  vez: "Seu time: 2.5/6 no nível · dano recebido ×48.51 — leve 6 Pokémon no nível do boss
  para tomar menos dano". `mult` multiplica o dano recebido, e `strength` soma o quanto
  cada um dos seis está no nível. Conferido contra duas observações independentes, a base
  implícita deu 2,9999 e 2,9969: é 3. Deixou de ser chute e entrou na conta no mesmo dia —
  e ela mudou o veredito de "aguenta infinito" para "morre no primeiro golpe", que era o
  que acontecia no jogo. Ver
  [[A interface do sistema explica o que a API dele esconde]].

  O ponto não é que esperar valeu a pena por sorte. É que **deixar de fora é reversível e
  chutar não é**: com o fator fora, a tela dizia o que não sabia e ninguém tomou decisão
  com base num 48,7 inventado; quando o número apareceu, entrou. Se tivesse entrado como
  chute, o erro estaria enterrado dentro de um resultado plausível.
- **A velocidade na nota da tier list.** Ela ficou de fora enquanto foi chute — a doc do
  jogo só usa Speed na soma do Power exibido, e nenhum sistema público dá a ela efeito em
  combate. Só entrou quando virou medida: uma varredura de 231 combinações contra 35
  posições observadas, que subiu a correlação de posto de 0,833 pra 0,920. Entrou com peso
  pequeno (10%) e com a ressalva de que aditivo é aproximação do multiplicativo.

O contraste entre os dois é o ponto inteiro. Um termo pequeno **medido** entra; um termo
enorme **não medido** não entra. O que decide não é o quanto você gostaria de ter a
resposta completa.

## Na prática

- Escreva na tela o que você sabe do mecanismo, mesmo sem usá-lo. "O jogo aplica uma
  penalidade que não está publicada, e a relação é `3^deficit`" vale mais que silêncio: é
  o que permite a próxima pessoa medir.
- Se der pra medir, meça e inclua. Chute e medição não são a mesma coisa com confiança
  diferente — são coisas diferentes.
- Diga no resultado o que ele não cobre, na mesma tela e não num rodapé de documentação.

## Conexões
- Depende de: [[A tela não afirma mais precisão do que a fonte tem]]
- Irmã: [[Fórmula verificada só vale na escala em que foi verificada]]
- Técnica que aplica: [[Campo cujo nome você não sabe se lê do payload, nunca se chuta]]
- Visto em: [[piwdex2]]
- Mapa: [[Base]]
