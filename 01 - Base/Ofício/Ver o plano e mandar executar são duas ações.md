---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-24
---

# Ver o plano e mandar executar são duas ações

> Sistema que decide por você precisa mostrar o que decidiu. A tentação é fazer
> isso no mesmo gesto: escolher o modo já liga o modo, e a tela passa a exibir o
> plano *enquanto ele roda*. Funciona para quem já confia. Para quem está
> decidindo se confia, obriga a **delegar para poder olhar** — e depois desfazer.

## O sintoma

No [[piwdex2]] o robô ganhou objetivos ("farmar dinheiro", "subir este até 200").
Clicar no objetivo já entregava o comando: a partir dali o robô escolhia o alvo,
e o seletor manual travava com um aviso mandando voltar para manual se você
quisesse escolher.

O Eduardo leu isso como confuso, e estava certo: **olhar é de graça, delegar
não**. Um gesto que faz as duas coisas cobra o preço da segunda para entregar a
primeira, e a saída ainda por cima é um passo a mais ("volte para manual").

## A separação

Dois estados, e não um:

- **o modo é do olho** — qual plano estou vendo. Local, descartável, não sai da
  tela.
- **o objetivo é da máquina** — o que ela está perseguindo. Persistido, com
  efeito no mundo.

Entre os dois, um botão explícito. E o inverso também precisa de saída óbvia: no
modo manual, começar uma caçada **desliga o piloto sozinho** em vez de recusar o
comando — quem pediu uma coisa específica já disse o que queria.

## O que mais vale lembrar

- **Prévia sem compromisso vale mais quanto mais autônomo for o sistema.** Se
  ele vai agir sozinho por horas, ver o plano antes é a única chance real de
  discordar.
- **Recusar comando é pior que absorver.** "Não posso porque o piloto está
  ligado" transfere ao usuário um trabalho que o código sabia fazer: desligar.
- O mesmo corte aparece em migração (`--dry-run` antes de aplicar), em deploy
  (plano antes de `apply`) e em qualquer coisa destrutiva. Onde já é natural
  separar, ninguém pensa nisso como princípio — e é justamente por isso que ele
  passa batido nas telas onde a ação parece pequena.

## Conexões
- Irmã: [[Freio de oscilação vale para a máquina, não para a ordem de quem manda]] ·
  [[Objetivo é exclusivo, interruptor é combinável]] ·
  [[Uma pendência de prazo fecha por ato explícito, não por sinal inferido]]
- Visto em: [[piwdex2]]
- Mapa: [[Base]] · [[Backend]]
