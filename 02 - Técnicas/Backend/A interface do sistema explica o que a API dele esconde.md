---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-25
---

# A interface do sistema explica o que a API dele esconde

> Ao fazer engenharia reversa de um sistema de terceiro, a TELA dele é fonte de primeira
> classe — não o último recurso. Ela precisa se explicar a um humano, então ela nomeia o
> mecanismo que o JSON só numera.

## O problema

O reflexo, ao integrar com um sistema fechado, é ir direto ao que parece dado: o endpoint,
o JSON público, o payload da requisição. É o caminho do programador, e ele tem dois furos
que só aparecem depois de horas gastas:

1. **A API entrega o número sem a semântica.** `{members: 6, strength: 2.46, deficit: 3.54,
   mult: 48.69}` diz que `mult = 3^deficit` — mas não diz o que `mult` multiplica, nem como
   `strength` se calcula. Dá pra passar muito tempo concluindo "isso não está publicado".
2. **O arquivo público pode estar velho ou ser genérico.** Ele não quebra nada quando
   envelhece, então ninguém percebe.
3. **E ele pode simplesmente MENTIR.** Este é o pior dos três, porque não parece falta:
   `Eevee: { evolvesToId: 134, evolveLevel: 80 }` é campo preenchido, bem tipado e
   plausível — e é falso nos dois pontos. O Eevee não evolui por nível e não tem um
   destino só; ele é trocado com um NPC por um de cinco, cada um pedindo a sua pedra.
   Nenhuma validação de schema pega isso, porque o schema está certo.

## A solução

Abra a tela do sistema e leia o que ela diz, com as palavras dela.

No [[piwdex2]], depois de concluir que a penalidade de grupo do boss era um mecanismo
não publicado, um print da ficha do boss no jogo resolveu tudo numa frase:

```
Seu time: 2.5/6 no nível · dano recebido ×48.51
leve 6 Pokémon no nível do boss (Nv. 300) para tomar menos dano
Elemento: Neutro
```

Três respostas que o JSON não dava:

- **o que `mult` multiplica** — "dano recebido";
- **como `strength` se calcula** — "no nível do boss", ou seja, quanto cada um dos seis
  está no nível dele;
- **o elemento do boss** — Neutro, e não o tipo da espécie de que ele é feito. Com esse
  campo faltando, a ferramenta prometia 2,5x de vantagem numa luta que é 1x.

E o mesmo boss expôs o segundo furo: o `bossCatalog.json` público lista as recompensas
como Boss Token / Rare Candy / Heart Scale, e a tela do jogo mostra TM Disk Piece /
Ancient Stone / Rock Stone. Não é discordância de formato — é o arquivo público sendo
uma lista genérica que ninguém atualiza.

O terceiro furo apareceu no mesmo jogo, semanas depois, e é o mais grave porque não se
parece com falta. A Pokepedia declarava o Eevee "caso à parte (sistema próprio de
stones)" sem publicar qual, e o `creatures.json` preenchia o campo com uma evolução por
nível que não existe. Um print da Loja do Marlon entregou a tabela inteira de uma vez:

```
Flareon  EVOLUÇÃO  $ 65.000   1 Eevee no time   28/10 Fire Stone
Vaporeon EVOLUÇÃO  $ 65.000   1 Eevee no time    0/10 Water Stone
Jolteon  EVOLUÇÃO  $ 65.000   1 Eevee no time    0/10 Thunder Stone
Umbreon  EVOLUÇÃO  $ 65.000   1 Eevee no time    0/10 Darkness Stone
Espeon   EVOLUÇÃO  $ 65.000   1 Eevee no time   50/10 Enigma Stone
```

E note o que a tela dá de graça além da tabela: o **contador `28/10`**. Ele diz quantas
a pessoa tem e quantas o NPC pede, ou seja, a interface publica o requisito ao lado do
estoque porque senão o jogador não entende o botão desligado. Requisito é justamente o
que a API esconde. Antes do print eu tinha CHUTADO Moon Stone e Sun Stone para
Umbreon/Espeon, com dois argumentos bons — são as únicas duas stones com `npcPrice: 0`
no catálogo inteiro, e a tradição da série manda lua e sol. Estava errado: são Darkness
e Enigma. Ver [[Fator que domina o resultado não entra na conta por estimativa]].

## O que mais vale lembrar

- **A tela existe pra ser entendida; a API, pra ser consumida.** Onde há um mecanismo
  punitivo, a interface quase sempre o explica, porque senão o usuário reclama. É
  justamente o mecanismo que você mais precisa modelar.
- **Peça o print.** Quando alguém usa o sistema todo dia, uma captura de tela vale mais
  que uma tarde de leitura de payload. Nos casos acima, dois prints fecharam três
  perguntas de mecânica e uma tabela de cinco linhas que não existe em lugar nenhum.
- **Onde há botão desligado, há requisito publicado.** Interface que bloqueia uma ação
  precisa dizer o que falta, senão vira suporte. `0/10 Water Stone`, "Faltam {{stone}}",
  "Você precisa de um Eevee no time" — é a regra de negócio escrita por obrigação de
  usabilidade. Procure a tela do estado IMPEDIDO, não a do estado feliz.
- **Fonte pública sem consumidor apodrece.** Se um JSON público não alimenta a tela que
  você está vendo, desconfie dele antes de confiar.
- **Isto não substitui a medição.** A tela deu a fórmula; conferir a fórmula contra dois
  valores observados é o que a transformou em conta — ver
  [[Fator que domina o resultado não entra na conta por estimativa]].

Candidato a princípio na segunda aparição, fora deste sistema.

## Conexões
- Princípio: [[Peça o que a fonte mostra, não o que você precisa]] ·
  [[Tirar o dado errado não põe a verdade no lugar]]
- Referência que saiu daqui: [[Poke Idle World - evolucao e a troca do Eevee]]
- Irmã: [[Campo cujo nome você não sabe se lê do payload, nunca se chuta]] ·
  [[Fórmula verificada só vale na escala em que foi verificada]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
