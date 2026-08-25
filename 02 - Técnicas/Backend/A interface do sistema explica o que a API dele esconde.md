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

## O que mais vale lembrar

- **A tela existe pra ser entendida; a API, pra ser consumida.** Onde há um mecanismo
  punitivo, a interface quase sempre o explica, porque senão o usuário reclama. É
  justamente o mecanismo que você mais precisa modelar.
- **Peça o print.** Quando alguém usa o sistema todo dia, uma captura de tela vale mais
  que uma tarde de leitura de payload. No caso acima, um print fechou três perguntas
  abertas de uma vez.
- **Fonte pública sem consumidor apodrece.** Se um JSON público não alimenta a tela que
  você está vendo, desconfie dele antes de confiar.
- **Isto não substitui a medição.** A tela deu a fórmula; conferir a fórmula contra dois
  valores observados é o que a transformou em conta — ver
  [[Fator que domina o resultado não entra na conta por estimativa]].

Candidato a princípio na segunda aparição, fora deste sistema.

## Conexões
- Princípio: [[Peça o que a fonte mostra, não o que você precisa]]
- Irmã: [[Campo cujo nome você não sabe se lê do payload, nunca se chuta]] ·
  [[Fórmula verificada só vale na escala em que foi verificada]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
